title: Spring开启手动事务
date: 2015-01-29
tags:
---
用Spring的项目一般在方法切面上控制事务比较方便：
```java
@Transactional(rollbackFor = Exception.class)
public void methodName(){
    ......
}
```
Spring的切面只能切入到本Service方法的第一级调用，内层本Service方法调用是不能切入的，即便加了事务标记也不可以。
把内层相关代码挪到其他Service是一个解决办法，不过略烦。还有一个解决方法，就是手动开启事务支持，把想支持事务的代码块包裹起来。
<!-- more -->
第一步，配置：
```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager"
	p:dataSource-ref="dataSource" />
<bean id="transactionTemplate" class="org.springframework.transaction.support.TransactionTemplate">  
    <property name="transactionManager" ref="transactionManager" />  
</bean>
```
`dataSource`可以是动态数据源。

第二步，在需要的地方注入：
```java
@Autowired
private DataSourceTransactionManager transactionManager;

@Autowired
private TransactionOperations        transactionTemplate;
```

第三步，开始写手动事务：
如果代码块有返回值
```java
T ret=this.transactionTemplate.execute(new TransactionCallback<T>() {
    @Override
    public T doInTransaction(TransactionStatus status) {
        // your code goes here
        // return T type
    }
}
```
`T`是返回值类型。
如果代码块没有返回值
```java
this.transactionTemplate.execute(new TransactionCallbackWithoutResult() {
    @Override
    protected void doInTransactionWithoutResult(TransactionStatus tstatus) {
        // your code goes here
    }
}
```

就是这样~