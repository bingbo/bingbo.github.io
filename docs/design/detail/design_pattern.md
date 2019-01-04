---
layout: default
---

## 设计模式种类

### 创建型


1. 工厂方法（Factory Method）

	_**工厂方法模式**，定义一个用于创建对象的接口，让子类决定实例化哪一个类。工厂方法使一个类的实例化延迟到其子类。_	

	> 简单工厂模式的最大优点在于工厂类中包含了必要的逻辑判断，根据客户端的选择条件动态实例化相关的类，对于客户端来说，去除了与具体产品的依赖。

    ![](https://github.com/bingbo/blog/blob/master/images/gof/Factory%20Method.jpg)

	**示例代码**

	```java
	//工厂接口
	public interface Factory {
	    Product createProduct();
	}

	//产品接口
	public interface Product {

	    void update();
	}

	//具体工厂
	public class FactoryA implements Factory{
	    public Product createProduct() {
		return new ProductA();
	    }
	}

	public class FactoryB implements Factory{
	    public Product createProduct() {
		return new ProductB();
	    }
	}

	//具体产品
	public class ProductA implements Product{
	    public void update() {
		System.out.println("product A update ....");
	    }
	}

	public class ProductB implements Product{
	    public void update() {
		System.out.println("product B update ...");
	    }
	}

	//客户端
	public class Client {
	    public static void main(String[] args) {
		Factory factory = new FactoryA();
		Product product = factory.createProduct();
		product.update();

		Factory factory1 = new FactoryB();
		Product product1 = factory1.createProduct();
		product1.update();
	    }
	}
	```


1. 抽象工厂（Abstract Factory）

	_**抽象工厂模式**，提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。_


	![](https://github.com/bingbo/blog/blob/master/images/design_patterns/abstract_factory.jpg)

	**优点**

	* 易于交换产品系列，盱具体工厂类，在一个应用中只需要在初始化的时候出现一次，这就使得改变一个应用的具体工厂变得非常容易，它只需要改变具体工厂即可使用不同的产品配置。
	* 它让具体的创建实例过程与客户端分离，客户端是通过它们的抽象接口操纵实例，产品的具体类名也被具体工厂的实现分离，不会出现在客户代码中。

	**代码示例**

    ```java
    //接口工厂
    public interface IDatabase {

        IUserTable createUser();

        IDepartmentTable createDepartment();

    }

    //工厂A
    public class OracleDataBase implements IDatabase{
        public IUserTable createUser() {
            return new OracleUserTable();
        }

        public IDepartmentTable createDepartment() {
            return new OracleDepartmentTable();
        }
    }

    //工厂B
    public class SqlServerDataBase implements IDatabase{
        public IUserTable createUser() {
            return new SqlServerUserTable();
        }

        public IDepartmentTable createDepartment() {
            return new SqlServerDepartmentTable();
        }
    }

    //产品接口A
    public interface IUserTable {

        void insert(User user);

        User getUser(int id);
    }

    //产品接口B
    public interface IDepartmentTable {

        void insert(Department department);

        Department getDepartment(int id);
    }

    /**
    具体不同的产品
    **/
    public class OracleDepartmentTable implements IDepartmentTable{
        public void insert(Department department) {
            System.out.println("oracle insert department");
        }

        public Department getDepartment(int id) {
            System.out.println("oracle get department");
            return null;
        }
    }

    public class OracleUserTable implements IUserTable{
        public void insert(User user) {
            System.out.println("oracle insert user");
        }

        public User getUser(int id) {
            System.out.println("oracle get user");
            return null;
        }
    }


    public class SqlServerDepartmentTable implements IDepartmentTable{
        public void insert(Department department) {
            System.out.println("sql server insert department");
        }

        public Department getDepartment(int id) {
            System.out.println("sql server get department");
            return null;
        }
    }


    public class SqlServerUserTable implements IUserTable
    {
        public void insert(User user) {
            System.out.println("sql server insert user");
        }

        public User getUser(int id) {
            System.out.println("sql server get user");
            return null;
        }
    }

    //客户端
    public class Client {

        public static void main(String[] args) {
            IDatabase sqlDataBase = new SqlServerDataBase();
            IDepartmentTable departmentTable = sqlDataBase.createDepartment();
            IUserTable userTable = sqlDataBase.createUser();

            departmentTable.getDepartment(0);
            departmentTable.insert(null);
            userTable.getUser(0);
            userTable.insert(null);


            IDatabase oracleDataBase = new OracleDataBase();
            IDepartmentTable departmentTable1 = oracleDataBase.createDepartment();
            IUserTable userTable1 = oracleDataBase.createUser();
            departmentTable1.getDepartment(0);
            departmentTable1.insert(null);
            userTable1.getUser(0);
            userTable1.insert(null);
        }
    }
    ```


1. 建造者（Builder）


	_**建造者模式**，将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示，或将复杂的对象进行分步构建。_

    ![](https://github.com/bingbo/blog/blob/master/images/design_patterns/builder.png)

    示例代码

    ```java
    public abstract class AbstractBuilder<T> {

        protected T data;

        public AbstractBuilder(T data) {
            this.data = data;
        }

        public T build() {
            return this.data;
        }

    }
    ```

    > 具体实例构建器

    ```java
    public class MyLogBuilder extends AbstractBuilder<MyLog> {

        public MyLogBuilder() {
            super(new MyLog());
        }

        public MyLogBuilder id(Integer id) {
            this.data.setId(id);
            return this;
        }

        public MyLogBuilder key(String key) {
            this.data.setKey(key);
            return this;
        }

        public MyLogBuilder value(String value) {
            this.data.setValue(value);
            return this;
        }

        public MyLogBuilder level(Integer level) {
            this.data.setLevel(level);
            return this;
        }

        public MyLogBuilder type(Integer type) {
            this.data.setType(type);
            return this;
        }

        public MyLogBuilder time(Date time) {
            this.data.setTime(time);
            return this;
        }

    }
    ```

    ```java
    public class MyLog {
        private Integer id;
        private String key;
        private String value;
        private Integer type;
        private Integer level;
        private Date time;

        public MyLog() {
        }

        public Integer getId() {
            return id;
        }

        public void setId(Integer id) {
            this.id = id;
        }

        public String getKey() {
            return key;
        }

        public void setKey(String key) {
            this.key = key;
        }

        public String getValue() {
            return value;
        }

        public void setValue(String value) {
            this.value = value;
        }

        public Integer getType() {
            return type;
        }

        public void setType(Integer type) {
            this.type = type;
        }

        public Integer getLevel() {
            return level;
        }

        public void setLevel(Integer level) {
            this.level = level;
        }

        public Date getTime() {
            return time;
        }

        public void setTime(Date time) {
            this.time = time;
        }
    }
    ```

1. 原型模式（Prototype）

    
	_**原型模式**，用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。_
1. 单例（Singleton）


    ![单例](https://github.com/bingbo/blog/blob/master/images/gof/Singleton.jpg)

    在多线程情况下的多种实现方式

    > 饿汉模式

    ```java
    // 立即加载方式==饿汉模式
    public static class AObject {
        // 立即加载方式==饿汉模式
        private static AObject instance = new AObject();

        private AObject() {

        }

        public static AObject getInstance() {
            return instance;
        }
    }
    ```

    > 懒汉模式

    ```java
    /**
     * 懒汉模式
     */
    public static class BObject {
        private static BObject instance;

        private BObject() {

        }

        /**
         * 在整个方法上加锁
         *
         * @return
         */
        public static synchronized BObject getInstance() {
            try {
                if (instance == null) {
                    // 模拟在创建对象之前做一些准备工作
                    Thread.sleep(1000);
                    instance = new BObject();
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
            return instance;
        }
    }
    ```

    > 使用静态内置类实现单例模式

    ```java
    /**
     * 使用静态内置类实现单例模式
     */
    public static class CObject {
        // 内部类方式
        private static class ObjectHandler {
            private static CObject instance = new CObject();
        }

        private CObject() {
        }

        public static CObject getInstance() {
            return ObjectHandler.instance;
        }
    }
    ```

    > 序列化与反序列化的单例模式实现

    ```java
    /**
     * 序列化与反序列化的单例模式实现
     * 静态内置类可以达到线程安全问题，但如果遇到序列化对象时，使用默认的方式运行得到的结棍还是多例的
     */
    public static class DObject implements Serializable {
        private static final long serialVersionUID = 888L;

        private static class ObjectHandler {
            private static final DObject instance = new DObject();
        }

        private DObject() {

        }

        public static DObject getInstance() {
            return ObjectHandler.instance;
        }

        // 在反序列化中使用该方法
        protected Object readResolve() throws ObjectStreamException {
            return ObjectHandler.instance;
        }
    }
    ```

    >  使用static代码块实现单例模式

    ```java
    /**
     * 使用static代码块实现单例模式
     * 静态代码块中的代码在使用类的时候就已经执行了，所以可以应用静态代码块的这个特性来实现单例模式
     */
    public static class EObject {
        private static EObject instance = null;

        private EObject() {

        }
        static {
            instance = new EObject();
        }

        public static EObject getInstance() {
            return instance;
        }
    }
    ```

    > 使用enum枚举数据类型实现单例模式

    ```java
    /**
     * 使用enum枚举数据类型实现单例模式
     * 枚举enum和静态代码块的特性相似，在使用枚举类时，构造方法会被自动调用，也可以应用这个特性实现单例模式
     */
    public static class MyObject{
        public enum MyEnumSingleton{
            INSTANCE;
            private MyObject instance;

            MyEnumSingleton() {
                instance = new MyObject();
            }

            public MyObject getInstance() {
                return instance;
            }
        }

        public static MyObject getInstance() {
            return MyEnumSingleton.INSTANCE.getInstance();
        }
    }
    ```

### 结构型

1. 适配器（Adapter Class/Object）

    _**适配器模式(Adapter)**，将一个类的接口转换成客户希望的另外一个接口。Adpater模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。_

    ![](https://github.com/bingbo/blog/blob/master/images/design_patterns/adapter.png)

    示例代码

    ```java
    public class LogObject {
    }
    ```

    > 适配器接口

    ```java
    public interface LogHandlerAdapter {
        boolean supports(Object handler);

        Object handle(Object handler);

    }
    ```

    > 复杂适配器抽象类

    ```java
    public abstract class AbstractLogHandlerAdapter implements LogHandlerAdapter {

        public List<Object> handle(Object handler) {
            return this.handleInternal((LogObject) handler);
        }

        public boolean supports(Object handler) {
            return handler instanceof LogObject && this.supportsInternal((LogObject) handler);
        }

        protected abstract boolean supportsInternal(LogObject optLogEvent);

        protected abstract List<Object> handleInternal(LogObject handler);

    }
    ```

    > 具体适配器

    ```java
    public class ALogHandlerAdapter extends AbstractLogHandlerAdapter {

        protected List<Object> handleInternal(LogObject handler) {
            LogObject event = (LogObject) handler;
            return null;
        }

        protected boolean supportsInternal(LogObject optLogEvent) {
            return true;
        }

    }
    ```

    ```java
    public class BLogHandlerAdapter extends AbstractLogHandlerAdapter {

        protected List<Object> handleInternal(LogObject handler) {
            LogObject event = (LogObject) handler;
            return null;
        }

        protected boolean supportsInternal(LogObject optLogEvent) {
            return true;
        }

    }
    ```

    > 适配器模式在spring中的应用

    ![](https://github.com/bingbo/blog/blob/master/images/design_patterns/spring%E4%B8%AD%E7%9A%84%E9%80%82%E9%85%8D%E5%99%A8%E6%A8%A1%E5%BC%8F.jpg)

1. 桥接（Bridge）

    _**桥接模式**,将抽象部分与它的实现部分分离，使它们都可以独立地变化。它是一种对象结构型模式。_

    ![](https://github.com/bingbo/blog/blob/master/images/design_patterns/bridge.jpg)

    ```java
    public interface ImageImpl {

        void doPaint();
    }

    public class LinuxImpl implements ImageImpl{
        public void doPaint() {
            System.out.println("in linux ");

        }
    }

    public class WindowsImpl implements ImageImpl{
        public void doPaint() {
            System.out.println("in windows ");
        }
    }

    public abstract class Image {

        protected ImageImpl imageImpl;

        public void setImageImpl(ImageImpl impl) {
            this.imageImpl = impl;
        }

        public abstract void parseFile(String fileName);
    }

    public class JPGImage extends Image{
        public void parseFile(String fileName) {
            this.imageImpl.doPaint();
            System.out.println(" jpg image");
        }
    }

    public class PNGImage extends Image{
        public void parseFile(String fileName) {
            this.imageImpl.doPaint();
            System.out.println(" png image");
        }
    }

    public class App {

        private static Image image;

        public static void main(String[] args) {
            paint();
        }

        public static void paint() {
            image = new JPGImage();
            image.setImageImpl(new LinuxImpl());
            image.parseFile("aa");
        }
    }
    ```

1. 组合（Composite）

    _**组合模式(Composite)**，将对象组合成树形结构以表示“部分-整体”的层次结构。组合模式使得用户对单个对象和组合对象的使用具有一致性。_

    ![](https://github.com/bingbo/blog/blob/master/images/design_patterns/composite.jpg)


    ```java
    public abstract class Company {
        protected String name;

        public Company(String name) {
            this.name = name;
        }

        public abstract void add(Company company);

        public abstract void remove(Company company);

        public abstract void display(int depth);

        public abstract void doDuty();

    }

    public class ConcreteCompany extends Company {
        private List<Company> companies = new ArrayList<Company>();

        public ConcreteCompany(String name) {
            super(name);
        }

        public void add(Company company) {
            this.companies.add(company);
        }

        public void remove(Company company) {
            this.companies.remove(company);
        }

        public void display(int depth) {
            StringBuilder sb = new StringBuilder();
            for (int i = 0; i < depth; i++) {
                sb.append("-");
            }
            System.out.println(sb.append(this.name).toString());
            for (Company company : companies) {
                company.display(depth + 2);
            }
        }

        public void doDuty() {
            for (Company company : companies) {
                company.doDuty();
            }
        }
    }

    public class FinanceDepartment extends Company{
        public FinanceDepartment(String name) {
            super(name);
        }

        public void add(Company company) {

        }

        public void remove(Company company) {

        }

        public void display(int depth) {
            StringBuilder sb = new StringBuilder();
            for (int i = 0; i < depth; i++) {
                sb.append("-");
            }
            System.out.println(sb.append(this.name).toString());
        }

        public void doDuty() {
            System.out.println(this.name);
        }
    }

    public class HRDepartment extends Company{
        public HRDepartment(String name) {
            super(name);
        }

        public void add(Company company) {

        }

        public void remove(Company company) {

        }

        public void display(int depth) {
            StringBuilder sb = new StringBuilder();
            for (int i = 0; i < depth; i++) {
                sb.append("-");
            }
            System.out.println(sb.append(this.name).toString());
        }

        public void doDuty() {
            System.out.println(this.name);
        }
    }
    ```


1. 装饰（Decorator）

    _**装饰模式(Decorator)**，动态地给一个对象添加一些额外的职责，就增加功能来说，装饰模式比生成了类更为灵活。这样在不必改变原类文件和使用继承的情况下，动态地扩展一个对象的功能，通过创建一个包装对象，也就是装饰来包裹真实的对象。_

    ![](https://github.com/bingbo/blog/blob/master/images/design_patterns/zhuangshiqi_moshi_1.png)

    **示例代码**

	```java
    // 组件接口
    public interface Component {
        void operation();
    }

    // 具体组件A
    public class ComponentA implements Component{
       public void operation() {
           System.out.println("component a");
       }
    }

    // 装饰接口
    public abstract class Decorator implements Component{
       protected Component component;

       public Decorator(Component component) {
           this.component = component;
       }

       public void operation() {
           if (this.component != null) {
               this.component.operation();
           }
       }
    }

    // 组件A对应的装饰器A
    public class DecoratorA extends Decorator {

       public DecoratorA(Component component) {
           super(component);
       }

       @Override
       public void operation() {
           super.operation();
           this.operationInternal();
       }

       private void operationInternal() {
           System.out.println("my self operation");
       }
    }
     
    // 简单应用
    public class App {

      public static void main(String[] args) {
          Component component = new ComponentA();
          Decorator decorator = new DecoratorA(component);
          decorator.operation();
      }
    }
    ```
    
    > 装饰器模式在java io中的应用
    
    ![](https://github.com/bingbo/blog/blob/master/images/design_patterns/zhuangshiqi_moshi.png)
	
1. 外观（Facade）

    ![](https://github.com/bingbo/blog/blob/master/images/gof/Facade.jpg)
1. 享元（Flyweight）
1. 代理（Proxy）

    _**代理模式(Proxy)**，为其他对象提供一种代理以控制对这个对象的访问。_

    ![](https://github.com/bingbo/blog/blob/master/images/design_patterns/proxy.jpg)

    **示例代码**

	```java
	public interface Subject {
	    void request();
	}
	```

	```java
	public class RealSubject implements Subject {

	    public void request() {
		System.out.println("real request ...");
	    }
	}
	```

	```java
	public class Proxy implements Subject{


	    private Subject target;

	    public Proxy(Subject target) {
		this.target = target;
	    }

	    public void request() {
		this.target.request();
	    }
	}
	```

	```java
	public class Client {
	    public static void main(String[] args) {
		Subject target = new RealSubject();
		Proxy proxy = new Proxy(target);
		proxy.request();
	    }
	}

	```
    

    **动态代理示例**


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

	//处理器
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

	//client
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


    **cglib动态代理示例**


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


### 行为型

1. 解释器（Interpreter）

    _**解释器模式**，给定一个语言，定义它的文法的一种表示，并定义一个解释器，这个解释器使用该表示来解释语言中的句子。_


    > 解释器模式需要解决的是，如果一种特定类型的问题发生的频率足够高，那么可能就值得将该问题的各个实例表述为一个简单语言中的句子。这样就可以构建一个解释器，该解释器通过解释这些句子来解决该问题。

    ![](https://github.com/bingbo/blog/blob/master/images/design_patterns/interpreter.jpg)

    **示例代码**

    ```java
    public class Context {
        private String input;
        private String output;

        public String getInput() {
            return input;
        }

        public void setInput(String input) {
            this.input = input;
        }

        public String getOutput() {
            return output;
        }

        public void setOutput(String output) {
            this.output = output;
        }
    }

    public interface Expression {

        void interpret(Context context);
    }

    public class NonterminalExpression implements Expression{
        public void interpret(Context context) {
            System.out.println("nonterminal expression");
        }
    }

    public class TerminalExpression implements Expression{
        public void interpret(Context context) {
            System.out.println("terminal expression");
        }
    }
    ```

1. 模板方法（Template Method）

    _**模板方法模式**，定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。_

    ![](https://github.com/bingbo/blog/blob/master/images/design_patterns/template_method.jpg)

    **示例代码**

    ```java
    public abstract class Template {

        public void execute() {
            this.executeA();
            this.executeB();
        }

        protected abstract void executeB();

        protected abstract void executeA();

    }

    public class ConcreteTemplateOne extends Template{
        protected void executeB() {
            System.out.println("execute template one b method");
        }

        protected void executeA() {
            System.out.println("execute template one a method");
        }
    }

    public class ConcreteTemplateTwo extends Template{
        protected void executeB() {
            System.out.println("execute template two b method");
        }

        protected void executeA() {
            System.out.println("execute template two a method");
        }
    }

    public class App {
        private static Template template = null;

        public static void main(String[] args) {
            template = new ConcreteTemplateOne();
            template.execute();

            template = new ConcreteTemplateTwo();
            template.execute();
        }
    }
    ```


1. 责任链（Chain of Responsibility）

    _**职责链模式**(Chain of Responsibility),使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系。将这个对象连成一条链，并沿着这条链传递该请求，直到有一个对象处理它为止。_

    ![](https://github.com/bingbo/blog/blob/master/images/design_patterns/chain_of_responsibility.jpg)

    **代码示例**

    ```java
    public abstract class Manager {
        protected String name;
        protected Manager superior;

        public Manager(String name) {
            this.name = name;
        }

        public abstract void approve(int request);

        public Manager getSuperior() {
            return superior;
        }

        public void setSuperior(Manager superior) {
            this.superior = superior;
        }
    }
    ```

    ```java
    public class OneLevelManager extends Manager{

        public OneLevelManager(String name) {
            super(name);
        }

        public void approve(int request) {
            System.out.println("manager " + this.name + " approve request " + request);
            if (this.superior != null) {
                this.superior.approve(request);
            }
        }
    }
    ```

    ```java
    public class ThreeLevelManager extends Manager{
        public ThreeLevelManager(String name) {
            super(name);
        }

        public void approve(int request) {
            System.out.println("manager " + this.name + " approve request " + request);
            if (this.superior != null) {
                this.superior.approve(request);
            }
        }
    }
    ```

    ```java
    public class TwoLevelManager extends Manager{
        public TwoLevelManager(String name) {
            super(name);
        }

        public void approve(int request) {
            System.out.println("manager " + this.name + " approve request " + request);
            if (this.superior != null) {
                this.superior.approve(request);
            }
        }
    }
    ```

    ```java
    public class Client {
        public static void main(String[] args) {
            Manager one = new OneLevelManager("A");
            Manager two = new TwoLevelManager("B");
            Manager three = new ThreeLevelManager("C");
            one.setSuperior(two);
            two.setSuperior(three);

            one.approve(1);
            one.approve(2);
            one.approve(3);

        }
    }
    ```
    
    ![](https://github.com/bingbo/blog/blob/master/images/design_patterns/zerenlianmoshi.png)

    > 示例代码2
 
    ```java
    public interface CheckerChain {

        void doCheck(Map param) throws Exception;
        void addChecker(Checker checker);
    }
    ```

    ```java
    public interface Checker {
        void doCheck(Map params, CheckerChain chain);
    }
    ```

    > 检查器链

    ```java
    public class DefaultCheckerChain implements CheckerChain {

        private Checker[] checkers = new Checker[0];

        /**
         * checker链现在执行到了哪个位子
         */
        private int pos = 0;

        /**
         * 当前checker链中有checker数量
         */
        private int n = 0;
        public static final int INCREMENT = 10;

        public void doCheck(Map params) throws Exception {
            internalDoCheck(params);
        }

        private void internalDoCheck(Map params) throws Exception {
            if (pos < n) {
                try {
                    Checker checker = checkers[pos++];
                    checker.doCheck(params, this);
                } catch (Exception e) {
                    throw e;
                }
                return;
            }
        }

        public void addChecker(Checker checker) {
            for (Checker chk : checkers) {
                if (chk == checker) {
                    return;
                }
            }
            if (n == checkers.length) {
                Checker[] newCheckers = new Checker[n + INCREMENT];
                System.arraycopy(checkers, 0, newCheckers, 0, n);
                checkers = newCheckers;
            }
            checkers[n++] = checker;
        }
    }
    ```

    > 检查器工厂

    ```java
    public final class DefaultCheckerFactory {

        private DefaultCheckerFactory() {
        }

        public static CheckerChain createCheckerChain(ApplicationContext context) {
            CheckerChain checkerChain = new DefaultCheckerChain();
            Map<String, Checker> checkerMap = context.getBeansOfType(Checker.class);
            if (checkerMap != null && checkerMap.size() > 0) {
                for (Checker checker : checkerMap.values()) {
                    checkerChain.addChecker(checker);
                }
            }
            return checkerChain;
        }

    }
    ```

    > 检查器A

    ```java
    public class AChecker implements Checker{
        public void doCheck(Map params, CheckerChain chain) {

        }
    }
    ```

    > 检查器B

    ```java
    public class BChecker implements Checker{
        public void doCheck(Map params, CheckerChain chain) {

        }
    }
    ```

    **责任链模式在tomcat中的应用**

    ![](https://github.com/bingbo/blog/blob/master/images/design_patterns/tomcat%E8%B4%A3%E4%BB%BB%E9%93%BE%E6%A8%A1%E5%BC%8F.jpg)

    **责任链模式在tomcat中的交互流程**

    ![](https://github.com/bingbo/blog/blob/master/images/design_patterns/tomcat%E8%B4%A3%E4%BB%BB%E9%93%BE%E6%A8%A1%E5%BC%8F%E6%B5%81%E7%A8%8B%E5%9B%BE.jpg)

1. 命令（Command）

    _**命令模式**_，将一个请求封装为一个对象，从而使你可以用不同的请求对客户进行参数化；对请求排队或记录请求日志，以及支持可撤销的操作
    
    > 优点
    
    * 它能比较容易地设计一个命令队列；
    * 在需要的情况下，可以较容易地将命令记入日志；
    * 允许 接收请求的一方决定是否要否决请求；
    * 可以容易地实现对请求的撤销和重做；
    * 由于加进新的具体命令类不影响其他的类；
    * 它把请求一个操作的对象与知道怎么招待一个操作的对象分割开；
    * 使用命令模式作为"CallBack"在面向对象系统中的替代。"CallBack"讲的便是先将一个函数登记上，然后在以后调用此函数。
    
    ![](https://github.com/bingbo/blog/blob/master/images/design_patterns/command.jpg)
    
    **示例代码**
    
    ```java
    /**
     * Command
     * 类似于Callback,定义相应的回调函数
     */
    public interface Command {
    
        void execute();
    }
 
    public class CommandA implements Command{

         private Receiver receiver;

         public void setReceiver(Receiver receiver) {
             this.receiver = receiver;
         }

         public void execute() {
             this.receiver.action();
         }
    }
    
    public class CommandB implements Command{
        private Receiver receiver;
    
        public void setReceiver(Receiver receiver) {
            this.receiver = receiver;
        }
    
        public void execute() {
            this.receiver.action();
        }
    }
 
    public class Receiver {
    
        public void action() {
            System.out.println("do action");
        }
    }
 
    public class Sender {
        private Command command;
    
        public void execute() {
            this.command.execute();
        }
    
        /**
         * 或是相当于直接传一个命令callback类
         *
         * @param command
         */
        public void execute(Command command) {
            command.execute();
        }
    }
    ```
    
1. 迭代器（Iterator）
1. 中介者（Mediator）
1. 备忘录（Memento）
1. 观察者（Observer）

    _**观察者模式**定义了一种一对多的依赖关系，让多个观察者对象同时监听某一个主题对象。这个主题对象在状态发生变化时，会通知所有观察者对象，使它们能够自己更新自己。_

    ![](https://github.com/bingbo/blog/blob/master/images/gof/Observer.jpg)

    **相关代码示例**

    ```java
    public abstract class Subject {
        private List<Observer> observers = new ArrayList<Observer>();

        //增加观察者
        public void attach(Observer observer) {
            this.observers.add(observer);
        }

        //移除观察者
        public void detach(Observer observer) {
            this.observers.remove(observer);
        }

        //通知
        public void doNotify() {
            for (Observer o : this.observers) {
                o.update();
            }
        }
    }
    ```

    ```java
    public class ConcreteSubject extends Subject{
        //具体被观察者状态
        private String subjectState;

        public String getSubjectState() {
            return subjectState;
        }

        public void setSubjectState(String subjectState) {
            this.subjectState = subjectState;
        }
    }
    ```

    ```java
    public interface Observer {

        void update();
    }
    ```

    ```java
    public class ConcreteObserver implements Observer{
        private String name;
        private String observerState;
        private ConcreteSubject subject;

        public ConcreteObserver(String name, ConcreteSubject subject) {
            this.name = name;
            this.subject = subject;
        }

        public void update() {
            this.observerState=this.subject.getSubjectState();
            System.out.println("观察者" + this.name + "的新状态是" + this.observerState);
        }

        public ConcreteSubject getSubject() {
            return subject;
        }

        public void setSubject(ConcreteSubject subject) {
            this.subject = subject;
        }
    }
    ```

    ```java
    public class Client {
        public static void main(String[] args) {
            ConcreteSubject s = new ConcreteSubject();
            s.attach(new ConcreteObserver("X", s));
            s.attach(new ConcreteObserver("Y", s));
            s.attach(new ConcreteObserver("Z", s));

            s.setSubjectState("abc");
            s.doNotify();
        }
    }
    ```

    **观察者模式在spring中的应用**

    ![](https://github.com/bingbo/blog/blob/master/images/design_patterns/spring%E4%B8%AD%E7%9A%84%E8%A7%82%E5%AF%9F%E8%80%85%E6%A8%A1%E5%BC%8F.jpg)

    **spring中观察者模式交互流程**

    ![](https://github.com/bingbo/blog/blob/master/images/design_patterns/spring%E8%A7%82%E5%AF%9F%E8%80%85%E6%A8%A1%E5%BC%8F%E4%BA%A4%E4%BA%92%E5%9B%BE.jpg)

1. 状态（State）

    _**状态模式**，当一个对象的内存状态改变时允许改变其行为，这个对象看起来像是改变了其类。状态模式主要解决的是当控制一个对象状态转换的条件表达式过于复杂的情况。把状态的判断逻辑转移到表示不同状态的一系列类当中，可以把复杂的判断逻辑简化。_
    
    ![](https://github.com/bingbo/blog/blob/master/images/design_patterns/state.png)
    
    __代码示例__
    
    ```java
        // 上下文
    public class Context {
        private State state;

        public Context(State state) {
            this.state = state;
        }

        public void request() {
            state.handle(this);
        }

        public State getState() {
            return state;
        }

        public void setState(State state) {
            this.state = state;
        }
    }
    ```

    ```java
    // 抽象状态类接口
    public interface State {
        void handle(Context context);
    }
    ```
    
    ```java
    public class StateA implements State{
        public void handle(Context context) {
            // 下一状态是StateB
            context.setState(new StateB());
            System.out.println("do state A operation");
        }
    }
    ```
    
    ```java
    public class StateB implements State{
        public void handle(Context context) {
    
            // 下一状态是StateC
            context.setState(new StateC());
            System.out.println("do state B operation");
        }
    }
    ```
    
    ```java
    public class StateC implements State{
        public void handle(Context context) {
            // 下一状态是StateA
            context.setState(new StateA());
            System.out.println("do state C operation");
        }
    }
    ```
    
    ```java
    public class App {
        public static void main(String[] args) {
            Context context = new Context(new StateA());
            context.request();
            context.request();
            context.request();
            context.request();
            context.request();
        }
    }
    ```

1. 策略（Strategy）

    _**策略模式**，它定义了一些算法，分别封装起来，让它们之间可以互相替换，此模式让算法的变化，不会影响到使用算法的客户。_

    ![策略](https://github.com/bingbo/blog/blob/master/images/gof/Strategy.jpg)

1. 访问者（Visitor）

    _**访问者模式**，表示一个作用于某对象结构中的各元素的操作。它使你可以在不改变各元素的类的前提下定义作用于这些元素的新操作。_

    * 该模式把数据结构和作用于结构上的操作之间的耦合解脱开，使得操作集合可以相对自由地演化
    * 增加新的操作很容易，因为增加新的操作就意味着增加一个新的访问者，并将有关行为集中到一个访问者对象中

    ![](https://github.com/bingbo/blog/blob/master/images/gof/Visitor.jpg)

    ![](https://raw.githubusercontent.com/bingbo/mybatis-spring-webapp/master/webapp-util/src/test/java/com/ibingbo/design/pattern/visitor/visitor_design.png)

    _**示例代码**_

    ```java
    //元素

    public interface Node {
        public void accept(Visitor visitor);
    }

    public class NodeA implements Node{
        public void accept(Visitor visitor) {
            visitor.visit(this);
        }

        public String operation() {
            return "NodeA";
        }
    }

    public class NodeB implements Node{
        public void accept(Visitor visitor) {
            visitor.visit(this);
        }

        public String operation() {
            return "NodeB";
        }
    }

    //访问者
    public interface Visitor {

        public void visit(NodeA node);

        public void visit(NodeB node);
    }

    public class VisitorA implements Visitor{
        public void visit(NodeA node) {
            System.out.println(node.operation());
        }

        public void visit(NodeB node) {
            System.out.println(node.operation());
        }
    }

    public class VisitorB implements Visitor{
        public void visit(NodeA node) {
            System.out.println(node.operation());
        }

        public void visit(NodeB node) {
            System.out.println(node.operation());
        }
    }

    //元素上下文
    public class NodeContext {

        private List<Node> nodes = new ArrayList<Node>();

        public void accept(Visitor visitor) {
            for (Node node : this.nodes) {
                node.accept(visitor);
            }
        }

        public void add(Node node) {
            this.nodes.add(node);
        }
    }

    public class App {

        public static void main(String[] args) {
            NodeContext context = new NodeContext();
            context.add(new NodeA());
            context.add(new NodeB());
            Visitor visitor = new VisitorA();
            context.accept(visitor);

            visitor = new VisitorB();
            context.accept(visitor);

        }
    }
    ```
