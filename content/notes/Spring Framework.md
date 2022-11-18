---
title: "Spring Framework"
---

# Spring framework

![[notes/images/Spring's basic functional area.png]]

### The Core Spring Container

### Aspect-oriented programming (AOP)

### Data Access/Integration

### DI
https://martinfowler.com/articles/injection.html#ConstructorVersusSetterInjection

### POJO
#pojo Plain Old Java Object
https://www.geeksforgeeks.org/pojo-vs-java-beans/


### Wiring beans using XML

```java
public AccountService(AccountDao accountDao) {
  this.accountDao = accountDao;
}
```

In Spring, you resolve the dependency like this:

```xml
<bean id="accountService" class="com.springinpractice.ch01.service.AccountService">
  <constructor-arg ref="accountDao"/>
</bean>
```

#PropertyPlaceholderConfigurer


### Bean scopes
- #singleton default scope for beans in Spring, 
- prototype
- request
- session
- global session

### JDBC

