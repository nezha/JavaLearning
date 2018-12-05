# Java 中的三种代理模式

## 第一种：静态代理

### 程序实现

`代码实现`

`1.设计接口`

```java
public interface IUserDao {
    void save();
}
```

`2.实现接口`

```java
public class UserDao implements IUserDao {
    @Override
    public void save() {
        System.out.println("----已经保存数据!----");
    }
}
```

`3.代理类` 

> 必须实现接口

```java
public class UserDaoProxy implements IUserDao {
    //接收保存目标对象
    private IUserDao target;
    public UserDaoProxy(IUserDao target){
        this.target=target;
    }
    @Override
    public void save() {
        System.out.println("开始事务...");
        target.save();//执行目标对象的方法
        System.out.println("提交事务...");
    }
}
```

`4.测试类`

```java
public class App {
    public static void main(String[] args) {
        //目标对象
        UserDao target = new UserDao();
        //代理对象,把目标对象传给代理对象,建立代理关系
        UserDaoProxy proxy = new UserDaoProxy(target);
        proxy.save();//执行的是代理的方法
    }
}
```


## 第二种：JDK动态代理



### `代码实现`

`1.设计接口`

```java
public interface IUserDao {
    void save();
}
```

`2.实现接口`

```java
public class UserDao implements IUserDao {
    @Override
    public void save() {
        System.out.println("----已经保存数据!----");
    }
}
```

`3.代理类`

> 不需要实现接口

```java
public class ProxyFactory {
    //维护一个目标对象
    private Object target;
    public ProxyFactory(Object target){
        this.target=target;
    }

    //这是Java8语法
    public Object getProxyInstanceLambda(){
        return Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                (proxy,method,args) -> {
                    System.out.println("开始事务2");
                    //执行目标对象方法
                    Object returnValue = method.invoke(target, args);
                    System.out.println("提交事务2");
                    return returnValue;
                });
    }
    
    //传统语法
    public Object getProxyInstance(){
        return Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        System.out.println("开始事务2");
                        //执行目标对象方法
                        Object returnValue = method.invoke(target, args);
                        System.out.println("提交事务2");
                        return returnValue;
                    }
                });
    }
}
```


`4.测试类`

```java
public class App {
    public static void main(String[] args) {
        // 目标对象
        IUserDao target = new UserDao();
        // 【原始的类型 class UserDao】
        System.out.println(target.getClass());

        // 给目标对象，创建代理对象
        IUserDao proxy = (IUserDao) new ProxyFactory(target).getProxyInstanceLambda();
        // class $Proxy0   内存中动态生成的代理对象
        System.out.println(proxy.getClass());

        // 执行方法   【代理对象】
        proxy.save();
    }
}
```

### JDK中生成代理对象的API

> 代理类所在包:`java.lang.reflect.Proxy`
> `JDK`实现代理只需要使用`newProxyInstance`方法,但是该方法需要接收三个参数,完整的写法是:

```java
static Object newProxyInstance(ClassLoader loader, Class[] interfaces,InvocationHandler h )
```

**注意该方法是在`Proxy`类中是静态方法,且接收的三个参数依次为:**

1. `ClassLoader loader`,:指定当前目标对象使用类加载器,获取加载器的方法是固定的

2. `Class[] interfaces`,:目标对象实现的接口的类型,使用泛型方式确认类型

3. `InvocationHandler h`:事件处理,执行目标对象的方法时,会触发事件处理器的方法,会把当前执行目标对象的方法作为参数传入


## 第三种：Cglib代理

### `代码实现`

`1.目标对象类`

```java
public class UserDao {
    public void save() {
        System.out.println("----已经保存数据!----");
    }
}
```

`2.Cglib代理工厂`

```java
public class ProxyFactory implements MethodInterceptor {
    //维护目标对象
    private Object target;

    public ProxyFactory(Object target) {
        this.target = target;
    }

    //给目标对象创建一个代理对象
    public Object getProxyInstance(){
        //1.工具类
        Enhancer en = new Enhancer();
        //2.设置父类
        en.setSuperclass(target.getClass());
        //3.设置回调函数
        en.setCallback(this);
        //4.创建子类(代理对象)
        return en.create();

    }

    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("开始事务...");
        //执行目标对象的方法
        Object returnValue = method.invoke(target, objects);
        System.out.println("提交事务...");
        return returnValue;
    }
}
```

`3.测试类`

```java
public class App {

    public static void main(String[] args) {
        //目标对象
        UserDao target = new UserDao();
        //代理对象
        UserDao proxy = (UserDao)new ProxyFactory(target).getProxyInstance();
        //执行代理对象的方法
        proxy.save();
    }
}
```

## 三者的区别

`1.静态代理`

> 目标对象要实现接口,而且代理对象需要实现接口。
> 这样的代理类只能代理特定接口的对象，灵活度不够。

`2.JDK动态代理`

> 代理对象不需要实现接口,但是目标对象一定要实现接口,否则不能用动态代理

`3.CGLib动态代理`

> 代理对象不需要实现接口，目标对象也可以是单纯的一个对象。

--- 

> 在Spring的AOP编程中:
> `如果加入容器的目标对象有实现接口,用JDK代理.`
> `如果目标对象没有实现接口,用Cglib代理.`

## 参考文献

详解 Java 中的三种代理模式：<https://mp.weixin.qq.com/s/nBmbNP2mR7ei-lDsuOxjWg>

你真的完全了解Java动态代理吗？看这篇就够了：<https://mp.weixin.qq.com/s/P-nrfyyWfRUurKgF4dnugA>