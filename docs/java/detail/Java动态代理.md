## Jdk动态代理 

通过InvocationHandler和Proxy针对实现了接口的类进行动态代理，即必须有相应的接口

> 应用

```java
public class TestProxy {
    public static void main(String[] args) {
        CTest cTest=new CTest();
        MyInvokationHandler handler=new MyInvokationHandler();
        handler.setTarget(cTest);
        ITest test = (ITest)Proxy.newProxyInstance(cTest.getClass().getClassLoader(), cTest.getClass().getInterfaces(), handler);
        test.say();
        test.sayHi("bill");
    }
}
```

> 接口及实现类

```java
//接口
public interface ITest {
    public void say();
    public void sayHi(String name);
}

//实现类
public class CTest implements ITest{
    public void say() {
        System.out.println("just say ...");
    }

    public void sayHi(String name) {
        System.out.println("just say to " + name);

    }
}
```

> 反射处理器

```java
public class MyInvokationHandler implements InvocationHandler{
    private Object target;

    public void setTarget(Object target) {
        this.target = target;
    }
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("starting call " + method);
        method.invoke(target, args);
        System.out.println("ending call " + method);
        return null;
    }
}
```

## Cglib动态代理
cglib动态代理是通过Enhancer创建代理类的子类，并重写相应的方法加入方法拦截器后再调用父类即代理类的方法

> 应用

```java
/**
 * 通过Enhancer创建代理类的子类，并重写相应的方法加入方法拦截器后再调用父类即代理类的方法
 * Created by bing on 2016/12/30.
 */
public class MyApp {

    public static void main(String[] args) {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(MyService.class);
        enhancer.setCallback(new MyMethodInterceptor());
        MyService service = (MyService)enhancer.create();
        service.add();
        service.delete();
    }
}
```

> 定义方法拦截器

```java
/**
 * 定义方法拦截器
 * Created by bing on 2016/12/30.
 */
public class MyMethodInterceptor implements MethodInterceptor{
    @Override
    public Object intercept(Object target, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        System.out.println("before: " + method);
        Object object = methodProxy.invokeSuper(target, args);
        System.out.println("after: " + method);
        return object;
    }
}
```

> 业务逻辑实现类

```java
/**
 * 业务逻辑实现类
 * Created by bing on 2016/12/30.
 */
public class MyService {

    public void add(){
        System.out.println("test service add method ..");
    }

    public void delete(){
        System.out.println("test service delete method ..");
    }
}
```