## maven常用命令

### mvn help:system

> 该命令会打印出所有的java系统属性和环境变量，这些信息对我们日常的编程工作很有帮助

### mvn dependency:list

> 查看当前项目的已解析依赖

### mvn dependency:tree

> 查看当前项目的依赖树

### mvn dependency:analyze

> 分析当前项目依赖信息，哪些是项目中未使用但显式声明的依赖，哪些是项目中使用到但没有显式声明的依赖

### mvn dependency:sources

> 尝试下载在pom.xml中依赖的文件的源代码

### mvn dependency:resolve -Dclassifier=javadoc

> 尝试下载对应的javadocs

### mvn help:describe -Dplugin=compiler -Ddetail

> 查看某个插件的详细信息

### mvn test -Dtest= xxxTest

> 运行指定的测试用例,test参数的值必须匹配一个或者多个测试类，如果找不到任何匹配的测试类，就会报错并导致构建失败

```
mvn test -Dtest=aaaTest
mvn test -Dtest=aaa*Test
mvn test -Dtest=aaaTest,bbbTest
#即使没有任何测试用例也不要报错
mvn test -Dtest -DfailIfNoTests = false
```

### mvn source:jar|mvn source:test-jar

> 单独打包源码或测试源码命令，需要安装打包源码插件

```pom
<build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-source-plugin</artifactId>
        <version>3.0.1</version>
        <executions>
          <execution>
            <id>attach-sources</id>
            <phase>verify</phase>
            <goals>
              <goal>jar-no-fork</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
</build>
```

## maven命令行

```
usage: mvn [options] [<goal(s)>] [<phase(s)>]
```

### 选项参数

> -am, --also-make 同时构建所列模块的依赖模块

```
#同时构建所列模块的依赖模块
mvn clean install -pl module1 -am
```

> -amd, -also-make-dependents同时构建依赖于所列模块的模块

```
#同时构建依赖于module1的模块
mvn clean install -pl module1 -amd
```

> -pl, --projects <arg> 构建指定的模块，模块间用逗号分隔

```
#构建指定某几个模块
mvn clean install -pl module1,module2
```

> -rf -resume-from <arg> 从指定的模块回复反应堆

```
#在完整的瓜堆构建顺序基础上指定从哪个模块开始构建，如从module1开始构建
mvn clean install -rf module1
```

## maven属性

```xml
<properties>
    <spring-version>4.3.2</spring-version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>${spring-version}</version>
    </dependency>
    ...
</dependencies>
```

> 通过<properties>元素用户可以自定义一个或多个maven属性，然后在pom的其他地方使用${属性名}的方式引用该属性，maven有6类属性：

* 内置属性

    主要有两个常用的内置属性，${basedir}表示项目根目录，即包含pom.xml文件的目录；${version}表示项目版本

* pom属性

    用户可以使用该类属性引用pom文件中对应元素的值。常用的pom属性包括：

    * ${project.build.sourceDirectory}：项目的主源码目录，默认为src/main/java
    * ${project.build.testSourceDirectory}：项目的测试源码目录，默认为src/test/java
    * ${project.build.directory}：项目构建输出目录，默认为target/
    * ${project.outputDirectory}：项目主代码编译输出目录，默认为target/classes/
    * ${project.testOutputDirectory}：项目测试代码编译输出目录，默认主target/test-classes/
    * ${project.groupId}：项目的groupId
    * ${project.artifactId}：项目的artifactId
    * ${project.version}：项目的version,与${version}等价
    * ${project.build.finalName}：项目打包输出文件的名称，默认为${project.artifactId}-${project.version}

    这些属性都对应了一个pom元素，它们中的一些属性的默认值都在超级pom中定义的

* 自定义属性

    用户可以在pom的<properties>元素下自定义maven属性

* settings属性

    与pom属性同理，用户使用以settings.开头的属性引用settings.xml文件中的xml元素的值，如${settings.localRepository}指向用户本地仓库的地址

* java系统属性

    据肝java系统属性都可以使用maven改改引用，如${user.home}指向了用户目录，可以使用mvn help:system查看所有的java系统属性

* 环境变量属性

    所有环境变量都可以使用以env.开头的maven属性引用，如${env.JAVA_HOME}指定了java_home环境变量的值，可以使用mvn help:system查看所有的环境变量


