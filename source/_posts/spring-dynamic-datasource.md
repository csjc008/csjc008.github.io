title: Spring添加动态数据源支持（标注）
date: 2015-01-15
---
本文目的在于用注解支持多数据源切换（使用MyBatis，其他ORM可能类似）。
公司原有根据Service方法名前缀决定读数据源还是写数据源的spring配置，但不适用于某些除了读写分离还有更多数据源的情况。
用注解感觉还是比较方便的。
<!-- more -->

写一个常量类，记录项目中用到的数据源名称
```java
package your.package.constant;

public class DataSourceConstant {
    public final static String DEFAULT = "read";
    public final static String READ    = "read";
    public final static String WRITE   = "write";
    public final static String ETL     = "etl";
}
```
 
写一个注解
```java
package your.package.annotations;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import your.package.constant.DataSourceConstant;

@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface DataSourceName {
    String dataSource() default DataSourceConstant.DEFAULT;
}
```
注意这个注解的RetentionPolicy，并且指定注解可以添加到类和方法上。

写一个类记录记录当前请求的数据源名称，使用ThreadLocal
```java
public class DatabaseContextHolder {

    private static final ThreadLocal<String> contextHolder = new ThreadLocal<String>();

    public static void setDataSourceName(String dataSourceName) {
        if (StringUtils.isEmpty(dataSourceName)) {
            contextHolder.set(DataSourceConstant.DEFAULT);
        } else {
            contextHolder.set(dataSourceName);
        }
    }

    public static String getDataSourceName() {
        String dataSourceName = contextHolder.get();
        if (StringUtils.isEmpty(dataSourceName)) {
            contextHolder.set(DataSourceConstant.DEFAULT);
            return DataSourceConstant.DEFAULT;
        }
        return dataSourceName;
    }

    public static void clearDataSourceName() {
        contextHolder.remove();
    }
}
```

继承`AbstractRoutingDataSource`
```java
public class MultiDynamicDataSource extends AbstractRoutingDataSource {

    @Override
    protected Object determineCurrentLookupKey() {
        String dataSourceName = DatabaseContextHolder.getDataSourceName();
        if (StringUtils.isEmpty(dataSourceName)) {
            dataSourceName = DataSourceConstant.DEFAULT;
        }
        return dataSourceName;
    }

}
```
这个类重写`determineCurrentLookupKey`方法，动态确定当前使用的数据源。

写一个拦截器
```java
@Aspect
@Order(Ordered.HIGHEST_PRECEDENCE)
@Component
public class DataSourceAdvice {
    @Around("execution(public * *(..)) && ( @annotation(your.package.annotations.DataSourceName) || @within(your.package.annotations.DataSourceName) )")
    public Object invoke(ProceedingJoinPoint pjp) throws Throwable {
        if (pjp.getSignature() instanceof MethodSignature) {
            String dataSource = null;
            MethodSignature msig = (MethodSignature) pjp.getSignature();
            DataSourceName dsnMtd = msig.getMethod().getAnnotation(DataSourceName.class);
            if (dsnMtd != null) {
                dataSource = dsnMtd.dataSource();
            } else {
                Class<?> clz = msig.getDeclaringType();
                DataSourceName dsnClz = clz.getAnnotation(DataSourceName.class);
                if (dsnClz != null) {
                    dataSource = dsnClz.dataSource();
                }
            }
            if (!StringUtils.isEmpty(dataSource)) {
                DatabaseContextHolder.setDataSourceName(dataSource);
            }
        }
        Object obj = pjp.proceed();
        return obj;
    }
}
```
这里也可以不用`@Around`改用其他，切入点含义是：在声明了`@DataSourceName`的public方法或者类名上声明了`@DataSourceName`的public方法中切入。
如果该方法上声明了数据源，则用该方法指定的数据源，否则用该方法所在类声明的数据源。
设置`@Order`是因为在MyBatis中，MyBatis使用Java的Proxy对我们的业务代码生成代理类，这个切入的优先级更高，我们需要将设置数据源的切入置于最外层，先设置数据源，然后`determineCurrentLookupKey`再执行。
详见MyBatis代码，在`org.apache.ibatis.session`下面
```java
  private class SqlSessionInterceptor implements InvocationHandler {
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
      final SqlSession sqlSession = SqlSessionManager.this.localSqlSession.get();
      if (sqlSession != null) {
        try {
          return method.invoke(sqlSession, args);
        } catch (Throwable t) {
          throw ExceptionUtil.unwrapThrowable(t);
        }
      } else {
        final SqlSession autoSqlSession = openSession();
        try {
          final Object result = method.invoke(autoSqlSession, args);
          autoSqlSession.commit();
          return result;
        } catch (Throwable t) {
          autoSqlSession.rollback();
          throw ExceptionUtil.unwrapThrowable(t);
        } finally {
          autoSqlSession.close();
        }
      }
    }
  }
```
`SqlSessionManager`在创建的时候：
```java
this.sqlSessionProxy = (SqlSession) Proxy.newProxyInstance(
        SqlSessionFactory.class.getClassLoader(),
        new Class[]{SqlSession.class},
        new SqlSessionInterceptor());
```

Spring配置（部分）
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:tx="http://www.springframework.org/schema/tx" xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.1.xsd
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-3.1.xsd
        http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.1.xsd">

	<context:property-placeholder location="classpath:props/jdbc.properties"  ignore-unresolvable="true"/>
	<aop:aspectj-autoproxy/>
	<context:component-scan base-package="your.package.interceptor" />

	<bean id="parentDataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource"
		destroy-method="close">
		<property name="driverClass">
			<value>${jdbc.driverClassName}</value>
		</property>
		<property name="jdbcUrl">
			<value>${jdbc.url}</value>
		</property>
		<property name="user">
			<value>${jdbc.username}</value>
		</property>
		<property name="password">
			<value>${jdbc.password}</value>
		</property>
		<property name="maxPoolSize">
			<value>${jdbc.maxPoolSize}</value>
		</property>
		<property name="minPoolSize">
			<value>${jdbc.minPoolSize}</value>
		</property>
		<property name="idleConnectionTestPeriod">
			<value>${jdbc.idleConnectionTestPeriod}
			</value>
		</property>
		<property name="maxIdleTime">
			<value>${jdbc.maxIdleTime}</value>
		</property>
		<property name="testConnectionOnCheckin">
			<value>${jdbc.testConnectionOnCheckin}</value>
		</property>
		<property name="checkoutTimeout">
			<value>${jdbc.checkoutTimeout}</value>
		</property>
	</bean>

	<bean id="read" destroy-method="close" parent="parentDataSource">
		<property name="jdbcUrl">
			<value>${jdbc.read.url}</value>
		</property>
		<property name="user">
			<value>${jdbc.read.username}</value>
		</property>
		<property name="password">
			<value>${jdbc.read.password}</value>
		</property>
	</bean>

	<bean id="write" parent="parentDataSource"></bean>

	<bean id="etl" parent="parentDataSource"><!-- 各种配置 --></bean>

	<!-- 动态数据源 -->
	<bean id="dynamicDataSource" class="your.package.MultiDynamicDataSource">  
        <property name="targetDataSources">  
            <map key-type="java.lang.String">  
                <entry value-ref="read" key="read"></entry>  
                <entry value-ref="write" key="write"></entry>
                <entry value-ref="etl" key="etl"></entry>    
            </map>  
        </property>  
        <property name="defaultTargetDataSource" ref="read">  
        </property>  
    </bean>
	
	<!-- datasource transactionManager -->
	<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager"
		p:dataSource-ref="dynamicDataSource" />
	<tx:annotation-driven transaction-manager="transactionManager"  proxy-target-class="true"/>
</beans>
```
MyBatis配置也相应需要更改。

这样，在各个Service的实现类和方法名上可以使用类似`@DataSourceName(dataSource = DataSourceConstant.READ)`的配置来设置数据源。

PS. 调用同一个Service内部的方法，是不会切换数据源的。