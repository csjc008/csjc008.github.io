title: Spring实现注解式配置
date: 2015-01-11
tags:
---
一般我们对于给Spring的bean进行使用properties文件的配置，使用如下的方式：
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:p="http://www.springframework.org/schema/p"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.1.xsd 
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.1.xsd">
	
	<context:property-placeholder location= "classpath:props/myPropFile.properties"  ignore-unresolvable= "true" />

	<!-- sample bean -->
	<bean id= "sampleBean" class= "your.package.SampleClass" >
	     <property name ="myProp" value= "${prop.in.propfile}" />
	</bean>
</beans>
```
<!-- more -->
然后在`SampleClass`中实现`myProp`的定义和setter。
这样在两个地方写来写去觉得略有麻烦，容易产生问题。
在Spring 3.0以上可以用注解的方式略掉spring配置文件中相关的配置了。
配置中加入
```xml
<beans xmlns:util="http://www.springframework.org/schema/util"  
    xsi:schemaLocation="http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-3.1.xsd">  
</beans>
```

再加入一个配置bean
```xml
<util:properties id="propsConfig" location="classpath:props/config.properties" />
```
这样，在用到的地方可以直接
```java
@Value("#{propsConfig['my.config.string']}")
private String needToConfig;
```

配置文件`config.properties`中可以写成
```ini
my.config.string=configuredString
```
类似的样子。这样省掉了setter，在一处配置与使用，要方便点。