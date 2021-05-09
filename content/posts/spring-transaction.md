---
title: "Spring中的@Transactional探索 "
date: 2019-10-08T10:39:26+08:00
draft: false
categories: ["Java相关"]
tags: ["Java","Spring","事务管理","AOP"]
author: "Jason·Gan"
---

* AOP是Spring的两大核心之一，可用于事务管理、安全、日志等系统中重要的功能，把他们从切面中抽离出来，实现解耦。Spring 为事务管理提供了丰富的功能支持。Spring 事务管理分为编码式和声明式的两种方式。编程式事务指的是通过Java编码方式实现事务，一般是自己写事务管理器，实现PlatformTransactionManager接口；声明式事务基于 AOP,将具体业务逻辑与事务处理解耦。声明式事务管理使业务代码逻辑不受污染, 因此在实际使用中声明式事务用的比较多。声明式事务有两种方式，一种是在配置文件（xml）中做相关的事务规则声明，另一种是基于@Transactional 注解的方式。注释配置是目前流行的使用方式，因此本文将着重介绍基于@Transactional 注解的事务管理。  

## @Transactional 注解管理事务的实现步骤  
* 使用@Transactional 注解管理事务的实现步骤分为两步。第一步，在 xml 配置文件中添加如下代码所有的事务配置信息。除了用配置文件的方式，@EnableTransactionManagement注解也可以启用事务管理功能。这里以简单的 DataSourceTransactionManager为例。  
```

<tx:annotation-driven />
<bean id="transactionManager"
        class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource" /> <!--这里要注入数据源-->
</bean>  
```  
* 上面表明先向Spring容器中注入了事务管理器  
* 第二步，将@Transactional 注解添加到合适的方法上，并设置合适的属性信息。@Transactional 注解的属性信息如下表展示。  
![](/image/transaction/1.jpg)  
* 除此以外，@Transactional 注解也可以添加到类级别上。当把@Transactional 注解放在类级别时，表示所有该类的公共方法都配置相同的事务属性信息。见下代码块，EmployeeService的所有方法都支持事务并且是只读。当类级别配置了@Transactional，方法级别也配置了@Transactional，应用程序会以方法级别的事务属性信息来管理事务，换言之，方法级别的事务属性信息会覆盖类级别的相关配置信息。  
```

@Transactional(propagation= Propagation.SUPPORTS,readOnly=true)
@Service(value ="employeeService")
public class EmployeeService  
```  
## Spring的注解方式的事务实现机制  
* 在应用系统调用声明@Transactional的目标方法时，Spring Framework默认使用AOP代理，在代码运行时生成一个动态代理对象，根据@Transactional 的属性配置信息，这个代理对象决定该声明@Transactional 的目标方法是否由拦截器TransactionInterceptor来使用拦截，在 TransactionInterceptor拦截时，会在在目标方法开始执行之前创建并加入事务，并执行目标方法的逻辑, 最后根据执行情况是否出现异常，利用抽象事务管理器(图2有相关介绍)AbstractPlatformTransactionManager操作数据源 DataSource提交或回滚事务,如下图所示。  
![图1](/image/transaction/2.jpg)  

* Spring AOP实现的代理技术有CglibAopProxy 和 JdkDynamicAopProxy两种，图 1是以 CglibAopProxy 为例，对于CglibAopProxy，需要调用其内部类的 DynamicAdvisedInterceptor的intercept方法。对于 JdkDynamicAopProxy，需要调用其invoke方法。正如上文提到的，事务管理的框架是由抽象事务管理器AbstractPlatformTransactionManager来提供的，而具体的底层事务处理实现，由PlatformTransactionManager的具体实现类来实现，如事务管理器DataSourceTransactionManager。不同的事务管理器管理不同的数据资源DataSource，比如 DataSourceTransactionManager管理JDBC的Connection。
PlatformTransactionManager，AbstractPlatformTransactionManager及具体实现类关系如下图所示。  
![](/image/transaction/3.jpg)  

## 注解方式的事务使用注意事项  
* 当对Spring的基于注解方式的实现步骤和事务内在实现机制有较好的理解之后，就会更好的使用注解方式的事务管理，避免当系统抛出异常，数据不能回滚的问题。  
* 正确的设置@Transactional的propagation属性  
* 需要注意下面三种propagation可以不启动事务。本来期望目标方法进行事务管理，但若是错误的配置这三种 propagation，事务将不会发生回滚。  
  1. TransactionDefinition.PROPAGATION_SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
  2. TransactionDefinition.PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
  3. TransactionDefinition.PROPAGATION_NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。  
## 正确的设置@Transactional的rollbackFor属性  
* 默认情况下，如果在事务中抛出了未检查异常（继承自 RuntimeException 的异常）或者 Error，则Spring将回滚事务；除此之外，Spring不会回滚事务。如果在事务中抛出其他类型的异常，并期望Spring能够回滚事务，可以指定 rollbackFor。例：```@Transactional(propagation=Propagation.REQUIRED,rollbackFor=MyException.class)```
通过分析Spring源码可以知道，若在目标方法中抛出的异常是rollbackFor指定的异常的子类，事务同样会回滚。  
* Spring中RollbackRuleAttribute的getDepth方法:  
```

private int getDepth(Class<?> exceptionClass, int depth) {
        if (exceptionClass.getName().contains(this.exceptionName)) {
            // Found it!
            return depth;
}
        // If we've gone as far as we can go and haven't found it...
        if (exceptionClass == Throwable.class) {
            return -1;
}
return getDepth(exceptionClass.getSuperclass(), depth + 1);
}
```

## @Transactional 只能应用到 public 方法才有效  
* 只有@Transactional 注解应用到public方法，才能进行事务管理。这是因为在使用 Spring AOP代理时，Spring在调用在图1中的TransactionInterceptor在目标方法执行前后进行拦截之前，DynamicAdvisedInterceptor（CglibAopProxy 的内部类）的intercept方法或 JdkDynamicAopProxy的invoke方法会间接调用 AbstractFallbackTransactionAttributeSource（Spring通过这个类获取@Transactional注解中的事务属性配置属性信息）的computeTransactionAttribute方法。  
```

protected TransactionAttribute computeTransactionAttribute(Method method,
    Class<?> targetClass) {
        // Don't allow no-public methods as required.
        if (allowPublicMethodsOnly() && !Modifier.isPublic(method.getModifiers())) {
return null;}
```  
* 这个方法会检查目标方法的修饰符是不是public，若不是 public，就不会获取@Transactional的属性配置信息，最终会造成不会用TransactionInterceptor来拦截该目标方法进行事务管理。  
## 避免 Spring 的 AOP 的自调用问题  
* 在Spring的AOP代理下，只有目标方法由外部调用，目标方法才由Spring生成的代理对象来管理，这会造成自调用问题。若同一类中的其他没有@Transactional注解的方法内部调用有@Transactional 注解的方法，有@Transactional 注解的方法的事务被忽略，不会发生回滚。
``` 

@Service
public class OrderService {
    private void insert() {
        insertOrder();
    }
@Transactional
    public void insertOrder() {
        //insert log info
        //insertOrder
        //updateAccount
       }
}
```  
* insertOrder 尽管有@Transactional 注解，但它被内部方法 insert 调用，事务被忽略，出现异常事务不会发生回滚。
* 上面的两个问题@Transactional注解只应用到 public 方法和自调用问题，是由于使用Spring AOP代理造成的。为解决这两个问题，使用AspectJ取代Spring AOP代理。需要将下面的AspectJ信息添加到xml配置信息中。  
```  
<!--AspectJ的xml配置-->
<tx:annotation-driven mode="aspectj" />
<bean id="transactionManager"
class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource" />
</bean>
<bean
class="org.springframework.transaction.aspectj.AnnotationTransactionAspect"
factory-method="aspectOf">
    <property name="transactionManager" ref="transactionManager" />
</bean>

<!--同时在Maven的pom文件中加入spring-aspects和 aspectjrt的dependency以及aspectj-maven-plugin。-->
```  
*  AspectJ的pom配置信息:  
```

<dependency>
<groupId>org.springframework</groupId>
<artifactId>spring-aspects</artifactId>
<version>4.3.2.RELEASE</version>
</dependency>
<dependency>
<groupId>org.aspectj</groupId>
<artifactId>aspectjrt</artifactId>
<version>1.8.9</version>
</dependency>
<plugin>
<groupId>org.codehaus.mojo</groupId>
<artifactId>aspectj-maven-plugin</artifactId>
<version>1.9</version>
<configuration>
<showWeaveInfo>true</showWeaveInfo>
<aspectLibraries>
<aspectLibrary>
<groupId>org.springframework</groupId>
<artifactId>spring-aspects</artifactId>
</aspectLibrary>
</aspectLibraries>
</configuration>
<executions>
<execution>
<goals>
<goal>compile</goal>
<goal>test-compile</goal>
</goals>
</execution>
</executions>
</plugin>
```


