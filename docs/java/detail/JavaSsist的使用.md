
详情见[测试项目](https://github.com/bingbo/simple-springmvc-app)

> 添加依赖

```xml
<!--Javassist是一个开源的分析、编辑和创建Java字节码的类库,
    jdk1.8必须用javassist3.18以上版本
-->
<dependency>
    <groupId>org.javassist</groupId>
    <artifactId>javassist</artifactId>
    <version>3.21.0-GA</version>
</dependency>
```

> 应用示例

```java
package com.ibingbo;

import javassist.ClassPool;
import javassist.CtClass;
import javassist.CtConstructor;
import javassist.CtField;
import javassist.CtMethod;
import javassist.CtNewMethod;
import javassist.Modifier;

/**
 * jdk1.8必须用javassist3.18以上版本
 *
 * @author zhangbingbing
 * @title JavaSsistTest
 * @date 17/10/16
 */
public class JavaSsistTest {
    public static void main(String[] args)
            throws Exception {
        createClass();
        getClassInfo();
        addAndEidtMethod();
    }

    /**
     * 添加修改方法
     */
    public static void addAndEidtMethod() throws Exception {

        ClassPool classPool = ClassPool.getDefault();
        CtClass ctClass = classPool.get("com.ibingbo.People");

        CtClass[] paramTypes = {classPool.get(String.class.getName())};
        CtMethod method = ctClass.getDeclaredMethod("show", paramTypes);
        // 复制方法
        CtMethod newMethod = CtNewMethod.copy(method, ctClass, null);

        String oldName = method.getName() + "$Impl";
        // 修改原方法名
        method.setName(oldName);

        // 修改新方法体
        newMethod.setBody("{System.out.println(\"执行前\");" + oldName + "($$);System.out.println(\"执行后\");System.out"
                + ".println(\"hi,\"+$1);}");
        ctClass.addMethod(newMethod);

        Class<?> cls = ctClass.toClass();
        cls.getMethod("show", String.class).invoke(cls.newInstance(), "world");
        ctClass.defrost();
    }

    /**
     * 获取类信息
     *
     * @throws Exception
     */
    public static void getClassInfo() throws Exception {
        ClassPool classPool = ClassPool.getDefault();
        CtClass ctClass = classPool.get("com.ibingbo.User");
        System.out.println(ctClass.getName());
        System.out.println("package: " + ctClass.getPackageName());
        System.out.println(Modifier.toString(ctClass.getModifiers()) + " class " + ctClass.getSimpleName());
        System.out.println("extends " + ctClass.getSuperclass().getName());
        if (ctClass.getInterfaces() != null && ctClass.getInterfaces().length > 0) {
            System.out.println("implements ");
            boolean first = true;
            for (CtClass c : ctClass.getInterfaces()) {
                if (first) {
                    first = false;
                } else {
                    System.out.print(", ");

                }
                System.out.print(c.getName());
            }
        }
        System.out.println();

    }

    /**
     * 动态创建类
     *
     * @throws Exception
     */
    public static void createClass() throws Exception {
        ClassPool classPool = ClassPool.getDefault();
        // 创建一个类
        CtClass ctClass = classPool.makeClass("com.ibingbo.Student");
        // 新建一个属性
        CtField field = new CtField(classPool.get(Integer.class.getName()), "id", ctClass);
        field.setModifiers(Modifier.PRIVATE);

        // 添加getter,setter方法
        ctClass.addMethod(CtNewMethod.setter("setId", field));
        ctClass.addMethod(CtNewMethod.getter("getId", field));
        ctClass.addField(field);

        // 添加默认构造函数
        CtConstructor constructor = new CtConstructor(null, ctClass);
        constructor.setModifiers(Modifier.PUBLIC);
        constructor.setBody("{}");
        ctClass.addConstructor(constructor);

        // 添加有参构造函数
        constructor = new CtConstructor(new CtClass[] {classPool.get(Integer.class.getName())}, ctClass);
        constructor.setModifiers(Modifier.PUBLIC);
        constructor.setBody("{this.id=$1;}");
        ctClass.addConstructor(constructor);

        // 设置方法
        CtMethod method = new CtMethod(CtClass.voidType, "run", null, ctClass);
        method.setModifiers(Modifier.PUBLIC);
        method.setBody("{System.out.println(\"执行结果\" + this.id);}");
        ctClass.addMethod(method);

        // 加载和执行生成的类
        Class<?> clazz = ctClass.toClass();
        Object object = clazz.newInstance();
        clazz.getMethod("setId", Integer.class).invoke(object, 11);
        clazz.getMethod("run").invoke(object);

        object = clazz.getConstructor(Integer.class).newInstance(22);
        clazz.getMethod("run").invoke(object);

    }
}
```
