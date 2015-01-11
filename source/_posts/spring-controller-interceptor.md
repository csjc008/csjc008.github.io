title: SpringMVC的Controller拦截日志配置
date: 2015-01-11
tags:
---

在项目中一直使用SpringMVC做Java后台服务，返回的基本都是JSON。
要求请求参数、返回结果都要打印到log里，方便追踪。
如果处理除了问题，也要返回显示错误的JSON。
<!--more-->
如果不用AOP拦截，每个Controller方法都要写成类似如下的样子：
```java
@RequestMapping(value = "/someUrl", produces = { "application/json;charset=UTF-8" })
@ResponseBody
public String someUrl(@ModelAttribute ReqForm reqParams) {
	Logger.info(..., "someUrl request: "+JsonUtil.toJson(reqParams));
    // form validation ...
    try {
        ResultObject result = someService.serviceMethod(reqParams);
        Logger.info(..., "someUrl response: "+JsonUtil.toJson(result));
        return JsonResult.success(result);
    } catch (Exception e) {
        Logger.error(..., "someUrl: "
                + JsonUtil.toJson(reqParams) + " || " + ExceptionUtil.printStackTraceToString(e));
        return JsonResult.fail();
    }
}
```
很烦，都是重复的东西。用上AOP就方便多了，加一个拦截类，加一点配置，就可以搞定所有。

首先，添加Maven依赖：
```xml
<dependency>
  <groupId>org.aspectj</groupId>
  <artifactId>aspectjweaver</artifactId>
  <version>1.8.0</version>
</dependency>
```

然后，实现一个请求拦截处理类(Logger的方法被简化了)
```java
import java.io.PrintWriter;
import java.io.StringWriter;
import java.lang.reflect.Method;
import java.util.Enumeration;
import java.util.HashMap;
import java.util.Map;

import javax.servlet.http.HttpServletRequest;

import org.aopalliance.intercept.MethodInterceptor;
import org.aopalliance.intercept.MethodInvocation;
import org.springframework.ui.Model;
import org.springframework.ui.ModelMap;
import org.springframework.validation.BindingResult;

/**
 * 请求拦截处理类
 * 
 * 
 */
public class ControllerInterceptor implements MethodInterceptor {

    @Override
    public String invoke(MethodInvocation invocation) {
        String result = "";
        String paramsStr = "";
        Object value = null;
        Method md = invocation.getMethod();
        try {
            Object[] args = invocation.getArguments();
            paramsStr = this.logRequest(args);
            value = invocation.proceed();
        } catch (Throwable e) {
            if (e instanceof ServiceException) {
                Logger.error(..., ((ServiceException) e).getAlarmId(), md
                        .getDeclaringClass().getSimpleName()
                        + "."
                        + md.getName()
                        + " || "
                        + paramsStr
                        + " || "
                        + printStackTraceToString(e));
            } else {
                Logger.error(..., "some_alarm_id", md
                        .getDeclaringClass().getSimpleName()
                        + "."
                        + md.getName()
                        + " || "
                        + paramsStr
                        + " || "
                        + printStackTraceToString(e));
            }
        }
        if (value != null) {
            result = value.toString();
        } else {
            result = JsonResult.SYSERROR;
        }

        this.logRequestResponse(md, paramsStr, result);
        return result;
    }

    private String logRequest(Object[] args) {
        if (args == null) {
            return "";
        }
        // 请求参数日志信息
        Map<String, Object> params = new HashMap<String, Object>();
        int i = 1;
        for (Object arg : args) {
            if (!(arg instanceof BindingResult) && !(arg instanceof ModelMap) && !(arg instanceof Model)) {
                if (arg instanceof HttpServletRequest) {
                    HttpServletRequest httpRequest = (HttpServletRequest) arg;
                    Enumeration<?> enume = httpRequest.getParameterNames();
                    if (null != enume) {
                        Map<String, String> hpMap = new HashMap<String, String>();
                        while (enume.hasMoreElements()) {
                            Object element = enume.nextElement();
                            if (null != element) {
                                String paramName = (String) element;
                                String paramValue = httpRequest.getParameter(paramName);
                                hpMap.put(paramName, paramValue);
                            }
                        }
                        params.put("HttpServletRequest", hpMap);
                    }
                } else {
                    try {
                        params.put("arg" + i, JsonUtil.toJson(arg));
                        i++;
                    } catch (Throwable e) {
                        Logger.warn(..., "CANNOT trasform to json string:"
                                + arg.getClass().getName());
                    }
                }
            }
        }
        String paramsStr = JsonUtil.toJson(params);
        return paramsStr;
    }

    private void logRequestResponse(Method md, String paramsStr, String re) {
        Map<String, String> logMap = new HashMap<String, String>();
        logMap.put("controller.method", md.getDeclaringClass().getSimpleName() + "." + md.getName());
        logMap.put("logReq", paramsStr);
        logMap.put("logRes", re);
        Logger.info(..., logMap);
    }

    private String printStackTraceToString(Throwable e) {
        StringWriter sw = new StringWriter();
        PrintWriter pw = new PrintWriter(sw);
        e.printStackTrace(pw);
        return sw.toString().replace("\n", " ").replace("\r", " ").replace("\t", " ");
    }

}
```
这里用一个类实现了`org.aopalliance.intercept.MethodInterceptor`接口，实现`invoke`方法，达到对Controller方法的进入和返回拦截的效果。
注意要在适当的时机执行`invocation.proceed()`方法，并返回它的返回值。否则所有Controller方法都没返回了。
还有一点，由于我用的SpringMVC，controller方法的一些参数并不需要打印出来，比如`BingdingResult` `ModelMap` `Model`等等。
如果有HttpServletRequest传进来，也不能直接被JSON序列化，需要特殊处理一下。还有其他不能序列化的东西，直接略掉(代码中的try...catch)。

然后，配置下spring。
在`spring-mvc.xml`文件的`<beans>`标签中加入aop相应的schema，并且添加刚才写的拦截类bean。
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	... ...
	xmlns:aop="http://www.springframework.org/schema/aop"

	xsi:schemaLocation="
		... ...
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.1.xsd">

	<bean id="controllerInterceptor" class="some.package.interceptor.ControllerInterceptor"/>
    <aop:config> 
         <aop:pointcut id="allPoint" expression="execution(public * your.controller.package.*.*(..)) "/>  
         <aop:advisor pointcut-ref="allPoint" advice-ref="controllerInterceptor"/>
    </aop:config>

</beans>
```
`<beans>`上主要添加`xmlns:aop`和`xsi:schemaLocation`两端aop的配置。其他配置根据情况添加。
切面配置的含义是：在`your.controller.package`下面所有类所有public的，任何参数，任何返回类型的方法进行拦截。

这样的话，开头的代码就可以简化为：
```java
@RequestMapping(value = "/someUrl", produces = { "application/json;charset=UTF-8" })
@ResponseBody
public String someUrl(@ModelAttribute ReqForm reqParams) {
    // form validation ...
    ResultObject result = someService.serviceMethod(reqParams);
    return JsonResult.success(result);
}
```
只管业务就可以了，代码清晰了很多。

最后，要注意在`web.xml`中springMVC的配置，类似于：
```xml
<servlet>
	<servlet-name>springmvc</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	<init-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:/spring/spring-mvc.xml</param-value>
	</init-param>
	<load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
	<servlet-name>springmvc</servlet-name>
	<url-pattern>/*</url-pattern>
</servlet-mapping>
```
注意跟业务的bean定义分离开。

采用这种方法，实现了对所有controller方法进行统一的日志记录。
包括进入记录：类名方法名，参数。返回记录：返回值。异常处理返回并记录StackTrace。
同时，使用一个简单的`ServiceException`可以做到在业务代码中写入alarmId，统一扔给Interceptor进行处理。