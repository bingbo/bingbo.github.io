多线程中单例模式和几种应用：

### 饿汉模式

```java
public class MyService implements Serializable {
    /**
     * 饿汉模式，使用类时已经创建完对象
     * 线程安全
     */
    private static MyService myService = new MyService();

    private MyService() {

    }

    public static MyService getInstance() {
        return myService;
    }
}
```

### 延迟加载、懒汉模式

```java
public class MyService implements Serializable {
    /**
     * 延迟加载、懒汉模式，即调用get方法时才被创建
     * 通过synchronized保证线程安全，但方法同步，性能较差
     */
    private static MyService myService;
    private MyService(){}

    public static synchronized MyService getInstance() {
        try {
            if (null == myService) {
                myService = new MyService();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return myService;
    }
}
```

### 双检查机制，分别在获取锁前后检查

```java
public class MyService implements Serializable {
    /**
     * 双检查机制，分别在获取锁前后检查
     * 通过volatile获取最新myService变量状态
     * 通过synchronized获取该类锁，实现并发线程安全
     */
    private volatile static MyService myService;
    private MyService(){}

    public static MyService getInstance() {
        try {
            if (null == myService) {
                synchronized (MyService.class) {
                    if (null == myService) {
                        myService = new MyService();
                    }
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return myService;
    }
}
```

### 使用静态内置类实现单例模式

```java
public class MyService implements Serializable {
    /**
     * 使用静态内置类实现单例模式
     * 静态内置类可以达到线程安全问题
     * 但要注意序列化对象时，也要保持单例模式
     */
    private static class MyServiceHolder {
        private static MyService myService = new MyService();
    }
    private MyService() {

    }

    public static MyService getInstance() {
        return MyServiceHolder.myService;
    }

    protected Object readResolve() throws ObjectStreamException {
        System.out.println("call readResolve method ...");
        return MyServiceHolder.myService;
    }
```

### 使用静态代码块

```java
public class MyService implements Serializable {
    /**
     * 使用静态代码块
     * 静态代码块在使用类时就已经执行了，也能保证线程安全
     */
    private static MyService myService = null;
    static {
        myService = new MyService();
    }

    private MyService() {

    }

    public static MyService getInstance() {
        return myService;
    }
}
```

### 在使用枚举类时，构造方法会被自动调用，可以应用其特性实现线程安全的单例模式

```java
public class MyService implements Serializable {
    /**
     * 在使用枚举类时，构造方法会被自动调用，可以应用其特性实现线程安全的单例模式
     */
    public enum MyServiceEnum{
        myServiceFactory;
        private MyService myService;

        MyServiceEnum() {
            try {
                myService = new MyService();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }

        public MyService getMyService() {
            return myService;
        }
    }

    public static MyService getInstance() {
        return MyServiceEnum.myServiceFactory.getMyService();
    }
}
```