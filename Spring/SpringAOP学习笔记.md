# Spring AOP学习笔记


## 引言

> `Advice`，通知增强，主要包括五个注解`Before`,`After`,`AfterReturning`,`AfterThrowing`,`Around`

`@Before`  在切点方法之前执行

`@After`  在切点方法之后执行

`@AfterReturning` 切点方法返回后执行

`@AfterThrowing` 切点方法抛异常执行

`@Around` 属于环绕增强，能控制切点执行前，执行后，，用这个注解后，程序抛异常，会影响@AfterThrowing这个注解

> **环绕通知非常强大，`可以决定目标方法是否执行，什么时候执行，执行时是否需要替换方法参数，执行完毕是否需要替换返回值`。**



## 代码实现

`1.引入POM`

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

`2.AOP拦截类`

```java
package com.nezha.learn.demo.aop;

import com.nezha.learn.demo.User;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class HelloAspect {

    @Pointcut("execution(* com.nezha.learn.demo.api.Hello.login(..))")
    private void pointCutLogin(){ }


    @Before(value = "pointCutLogin()")
    public void before(JoinPoint joinPoint){
        Object[] args = joinPoint.getArgs();
        String className = joinPoint.getSignature().getDeclaringTypeName();
        String methodName = joinPoint.getSignature().getName();
        System.out.println("before开始执行--->>>"+className+"/"+methodName+",参数是："+String.valueOf(args[0]));
    }

    @After(value = "pointCutLogin()")
    public void after(JoinPoint joinPoint){
        Object[] args = joinPoint.getArgs();
        String className = joinPoint.getSignature().getDeclaringTypeName();
        String methodName = joinPoint.getSignature().getName();
        System.out.println("after开始执行--->>>"+className+"/"+methodName+",参数是："+String.valueOf(args[0]));
    }

    @Around(value = "pointCutLogin()")
    public Object around(ProceedingJoinPoint proceedingJoinPoint){
        Object[] args = proceedingJoinPoint.getArgs();
        String className = proceedingJoinPoint.getSignature().getDeclaringTypeName();
        String methodName = proceedingJoinPoint.getSignature().getName();
        System.out.println("Around开始执行--->>>"+className+"/"+methodName+",参数是："+String.valueOf(args[0]));
        Object[] input = {"nezha"};
        Object proceed = null;
        try {
            //这里是执行方法的
            proceed = proceedingJoinPoint.proceed(input);
        }catch (Throwable t){
            t.printStackTrace();
        }
        System.out.println("===获得的proceed结果是："+(User)proceed);
        //！！！这边如果不把proceed返回出去，拦截的方法也不会进行结果返回的！
        return proceed;
    }
}
```

`3.被拦截的方法`

```java
package com.nezha.learn.demo.api;

import com.nezha.learn.demo.User;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/Hello")
public class Hello {

    @RequestMapping("/user/{name}")
    public User login(@PathVariable("name") String name){
        User user = new User(name,26);
        return user;
    }
}
```


`4.User类`

```java
package com.nezha.learn.demo;

import java.io.Serializable;

public class User implements Serializable {

    private static final long serialVersionUID = 1L;

    private String name;
    private Integer age;
    public User(){

    }
    public User(String username, Integer age) {
        this.name = username;
        this.age = age;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public Integer getAge() {
        return age;
    }
    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User String is:"+name+",age:"+age;
    }
}
```
## 参考文献

Spring AOP的实现原理：<http://www.importnew.com/24305.html>

从源码入手，带你一文读懂Spring AOP面向切面编程：<https://zackku.com/spring-aop/>