## maven依赖范围

```xml
<dependency> <groupId>javax.servlet</groupId>
<artifactId>servlet-api</artifactId>
<version>${servlet-api-version}</version>
<scope>provided</scope>
</dependency>
```

### compile

> 编译依赖范围。如果没有指定，就会默认使用该依赖范围。使用此依赖范围的maven依赖，对于编译、测试、运行三种classpath都有效。如spring-core，在编译、测试和运行的时候都需要使用该依赖

### test

> 测试依赖范围。使用此依赖范围的依赖，只对于测试classpath有效，在编译主代码或者运行项目的使用时将无法使用此类依赖。如junit，它只在编译测试代码及运行测试的时候才需要。

### provided

> 已提供依赖范围。使用此依赖范围的依赖，对于编译和测试classpath有效，但在运行时无效，如servlet-api，编译和测试项目的时候需要该依赖，但在运行项目的时候，由于容器已经提供，就不需要重复地引入一遍。

### runtime

> 运行时依赖范围。使用此依赖范围的依赖，对于测试和运行classpath有效，但在编译主代码时无效。如jdbc驱动实现，项目主代码的编译只需要jdk提供的jdbc接口，只有在执行测试或者运行项目的时候才需要实现上述接口的具体jdbc驱动。

### system

> 系统 依赖范围。该依赖与三种classpath的关系，和provided依赖范围完全一致。但是，使用system范围的依赖时必须通过systemPath元素显式的指定依赖文件的路径。由于此类依赖不是通过maven仓库解析 的，而且往往与本机系统绑定，可能造成构建的不可移植。因此应该谨慎使用。

### import(Maven 2.0.9及以上)

> 导入依赖范围。该依赖范围不会对三种classpath产生实际的影响。

### 依赖范围与classpath关系

|依赖范围(scope)|对于编译classpath有效|对于测试classpath有效|对于运行时classpath有效|例子|
|-----|-----|----|----|----|
|compile|Y|Y|Y|spring-core|
|test|--|Y|--|Y|junit|
|provided|Y|Y|-|servlet-api|
|runtime|-|Y|Y|jdbc驱动实现|
|system|Y|Y|--|本地的，maven仓库之外的类库文件|


