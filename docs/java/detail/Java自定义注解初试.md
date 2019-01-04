### 创建自定义注解

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Testable {
}
```

> @Retention 注解保留时间

* RetentionPolicy.SOURCE - 只在源码保留，编译阶段被丢弃
* RetentionPolicy.CLASS - 被编译器保留在编译后的类文件中，但在运行时不被虚拟机保留
* RetentionPolicy.RUNTIME - 编译阶段和运行阶段虚拟机都一直存在

> @Target 注解可被应用到哪些地方

* ElementType.TYPE - 可用于类，接口或枚举类声明时
* ElementType.FIELD - 可用于属性声明时
* ElementType.METHOD - 可用于方法声明时
* ElementType.PARAMETER - 可用于参数声明时
* ElementType.CONSTRUCTOR - 可用于构造函数声明时
* ElementType.LOCAL_VARIABLE - 可用于本地亦是声明时
* ElementType.ANNOTATION_TYPE - 可用于注解声明时，即定义注解时也可用
* ElementType.PACKAGE - 可用于包声明时
* ElementType.TYPE_PARAMETER - 可用于参数类型的声明时
* ElementType.TYPE_USE - 一种类型使用时


### 运用注解在其他地方

```java
public class MyTestable {

    @Testable
    public static void m1() {
        System.out.println("@m1");
    }

    public static void m2() {
        System.out.println("m2");
    }

    @Testable
    public static void m3() {
        System.out.println("@m3");
    }

    public static void m4() {
        System.out.println("m4");
    }
}
```

### 应用测试注解

```java
public class TestableProcessor {

    public static void process(String clazz) throws ClassNotFoundException {
        int passed=0;
        int failed=0;
        for(Method m:Class.forName(clazz).getMethods()){
            //判断该方法是否运用了该注解
            if(m.isAnnotationPresent(Testable.class)){
                try {
                    m.invoke(null);
                    passed++;
                }catch (Exception e){
                    failed++;
                }
            }
        }
        System.out.println("run some methods"+(passed+failed)+" pass:"+passed+" fail: "+failed);
    }

    public static void main(String[] args) throws ClassNotFoundException {
        process("com.ibingbo.annotation.example.MyTestable");
    }
}
```