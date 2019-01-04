## 根据不同环境构建差异化

### maven profile

profile能够在构建的时候修改pom的一个子集，或者添加额外的配置元素。用户可以使用很多方式激活profile，以实现构建在不同环境下的移植

```xml
<profiles>
    <profile>
        <id>dev</id>
        <properties>
            <db.driver>com.mysql.jdbc.Driver</db.driver>
            <db.url>jdbc:mysql://localhost:4406/test</db.url>
            <db.username>aaa</db.username>
            <db.password>aaa</db.password>
        </properties>
    </profile>
    <profile>
        <id>test</id>
        <properties>
            <db.driver>com.oracle.jdbc.Driver</db.driver>
            <db.url>jdbc:oracle://localhost:4406/test</db.url>
            <db.username>bbb</db.username>
            <db.password>bbb</db.password>
        </properties>
    </profile>
</profiles>
```

__激活profile__

1. 命令行激活

   > 用户可以使用mvn命令行参数-P加上profile的id来激活profile,多个id之间以逗号分隔

   `mvn clean install -P dev-x,dev-y`

1. settings文件显式激活

    > 如果用户希望某个profile默认一直处于激活状态，就可以配置settings.xml文件的activeProfiles元素，表示其配置的profile对于所有项目都牌激活状态

    ```xml
    <settings>
    ...
        <activeProfiles>
            <activeProfile>dev-x</activeProfile>
        </activeProfiles>
    ...
    </settings>
    ```

1. 系统属性激活

    > 用户可以配置当某系统属性存在的时候，自动激活profile

    ```xml
    <profiles>
        <profile>
            <activation>
                <property>
                    <!--当系统属性test存在时激活-->
                    <name>test</name>
                    <!--当系统属性test存在，且值为x时激活-->
                    <value>x</value>
                </property>
            </activation>
        ...
        </profile>
    </profiles>
    ```

    > 用户也可以通过命令行声明系统属性，如`mvn clean install -Dtest=x`

1. 操作系统环境激活

    > profile还可以自动根据操作系统环境激活，如果构建在不同的操作系统有差异，用户完全可以将这些差异写进profile,然后配置它们 自动基于操作系统环境激活

    ```xml
    <profiles>
        <profile>
            <activation>
                <os>
                    <!--对应系统属性中的os.name-->
                   <name>windows xp</name>
                    <!--值包括windows,unix,mac-->
                   <family>windows</family>
                    <!--对应系统属性中的os.arch-->
                   <arch>x86</arch>
                    <!--对应系统属性中的os.version-->
                   <version>5.1.2600</version>
                </os>
            </activation>
            ...
        </profile>
    </profiles>
    ```

1. 文件存在与否激活

    > maven能够根据项目中某个文件存在与否来决定是否激活profile

    ```xml
    <profiles>
        <profile>
            <activation>
                <file>
                    <!--不存在该文件-->
                    <missing>x.properties</missing>
                    <!--存在该文件时-->
                    <exists>y.properties</exists>
                </file>
            </activation>
        </profile>
    </profiles>
    ```

1. 默认激活

    > 用户可以在定义profile的时候指定其默认激活

    ```xml
    <profiles>
        <profile>
            <id>dev</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            ...
        </profile>
    </profiles>
    ```

    > 通过activeByDefault元素用户可以指定profile自动激活，不过需要注意的是，如果pom中有任何一个profile通过以上其他任意一种方式被激活了，所有的默认激活配置都会失效

    `mvn help:active-profiles`可以查看当前激活的profile,`mvn help:all-profiles`列出来当前所有的profile
    
__profile的种类__

* pom.xml：在该文件中声明的只对当前项目有效
* 用户settings.xml：用户目录下.m2/settings.xml中的profile对本机上该用户所有的项目有效
* 全局settings.xml：maven安装目录下conf/settings.xml中的profile对本机上所有的项目有效

### web资源过滤

```xml
<profiles>
    <profile>
        <id>dev</id>
        <properties>
            <db.driver>com.mysql.jdbc.Driver</db.driver>
            <db.url>jdbc:mysql://localhost:4406/test</db.url>
            <db.username>aaa</db.username>
            <db.password>aaa</db.password>
        </properties>
    </profile>
    <profile>
        <id>test</id>
        <properties>
            <db.driver>com.oracle.jdbc.Driver</db.driver>
            <db.url>jdbc:oracle://localhost:4406/test</db.url>
            <db.username>bbb</db.username>
            <db.password>bbb</db.password>
        </properties>
    </profile>
</profiles>

<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-war-plugin</artifactId>
    <version>2.1-beta-1</version>
    <configuration>
        <webResources>
            <resource>
                <!--针对这一目录开启过滤-->
                <filtering>true</filtering>
                <directory>src/main/webapp</directory>
                <!--指定过滤的文件-->
                <includes>
                    <include>* * /*.css</include>
                    <include>* * /*.js</include>
                </includes>
            </resource>
        </webResources>
    </configuration>
</plugin>
```

或者是在filter里过滤

```xml
<build>
    <filters>
        <!--过滤数据-->
        <filter>
            ${project.basedir}/src/main/resources/config.properties.${app-env}
        </filter>
    </filters>
    <resources>
        <!--需要过滤的目录文件-->
        <resource>
            <directory>${project.basedir}/src/main/resources</directory>
            <filtering>true</filtering>
        </resource>
    </resources>
    ...
</build>
```
