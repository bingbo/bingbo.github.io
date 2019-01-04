## maven仓库

### 远程仓库

```xml
<project>
...
<repositories>
    <repository>
        <!--id必须是唯一的-->
        <id>jboss</id>
        <!--仓库名称-->
        <name>jboss repository</name>
        <!--仓库地址,一般基于http协议，可直接浏览器打开-->
        <url>http://repository.jboss.com/maven2/</url>
        <!--发布版本构件配置-->
        <releases>
            <!--true表示可以下载发布版本-->
            <enabled>true</enabled>
        </relaeses>
        <!--快照版本构件配置-->
        <snapshots>
            <!--false表示不能下载快照版本-->
            <enabled>false</enabled>
            <!--配置仓库检查更新频率，默认为daily表示每天检查一次，never从不更新，always每次构件都更新,interval每隔x分钟检查一次更新-->
            <updatePolicy>daily</updatePolicy>
            <!--配置maven检查检验和文件的策略，当构件被部署到仓库时，会同时部署对应的检验和文件，下载构件时会验证检验和文件，如果失败，warn会输出警告信息，fail会构件失败,ignore忽略校验和错误-->
            <checksumPolicy>ignore</checksumPolicy>
        </snapshots>
        <!--layout:default表示仓库的布局是maven2,3的默认布局-->
        <layout>default</layout>
    </repository>
</repositories>
```

> repositories元素下可以用repository子元素声明一个或多个远程仓库

### 仓库认证

```xml
<settings>
...
<servers>
    <server>
        <!--针对my-proj仓库访问的用户名及密码-->
        <id>my-proj</id>
        <username>repo-user</username>
        <password>repo-pwd</password>
    </server>
</servers>
...
</settings>
```

### 部署到远程仓库

```xml
<project>
...
    <distributionManagement>
        <!--发布版本构件仓库-->
        <repository>
            <!--id为远程仓库唯一标识-->
            <id>repo-releases</id>
            <!--name仓库名称,为了方便人阅读-->
            <name>release repository</name>
            <!--仓库地址-->
            <url>http://192.168.1.1/content/</url>
        </repository>
        <!--快照版本仓库-->
        <snapshotRepository>
            <id>repo-snapshot</id>
            <name>snapshot repository</name>
            <url>http://192.168.1.2/content</url>
        </snapshotRepository>
    </distributionManagement>
...
</project>
```

> `mvn clean deploy`命令会将项目构建输出的构件部署到配置对应的远程仓库

### 从仓库解析依赖的机制

当本地仓库没有依赖构件的时候，maven会自动从远程仓库下载；当依赖版本为快照版本的时候，maven会自动找到最新的快照。

1. 当依赖的范围是system的时候，maven直接从本地文件系统解析构件
1. 是v日内瓦一个坑坐标计算仓库路径后，尝试直接从本地仓库寻找构件，如果发现相应构件，则解析成功
1. 在本地仓库不存在相应构件的情况下，如果依赖的版本是显式的发布版本构件，如1.2、2.1-beta-1等，则遍历所有的远程仓库，发现后，下载并解析使用。
1. 如果依赖的版本是release或者latest，则基于更新策略读取所有远程仓库的元数据groupId/artifactId/maven-metadata.xml，将其与本地仓库的对应元数据合并后，计算出relaese或者latest真实的值，然后基于这个真实的值检查本地和远程仓库，如步骤2）和3）
1. 如果依赖的版本是snapshot，则基于更新策略读取所有远程仓库的元数据groupId/artifactId/version/maven-metadata.xml，将其与本地仓库的对应数据合并后，得到最新快照版本的值，然后基于该值检查本地仓库，或者从远程仓库下载。
1. 如果最后解析得到的构件版本是时间戳格式的快照，如1.4-20091104.121450-121，则复制其时间缘戳格式的文件至非时间缀格式，如snapshot，并使用该非时间缀格式的构件。

> 可以用命令行参数mvn -U 强制更新

### 仓库镜像

```xml
<settings>
...
    <mirrors>
        <mirror>
            <!--镜像仓库的唯一标识符、名称、地址-->
            <id>maven.net.cn</id>
            <name> one of the central mirrors in China</name>
            <url>http://maven.net.cn/content</url>
            <!--表示该配置是中央仓库的镜像-->
            <mirrorOf>central</mirrorOf>
        </mirror>
    </mirrors>
...
</settings>
```

> 更高级的一些镜像配置：

* `<mirrorOf>*</mirrorOf>`  _匹配所有远程仓库_
* `<mirrorOf>external:*</mirrorOf>` _匹配所有远程仓库，使用localhost的除外，使用file://协议的除外。也就是说，匹配所有不在本机上的远程仓库_
* `<mirrorOf>repo1,repo2</mirrorOf>` _匹配仓库repo1和repo2,使用逗号分隔多个远程仓库_
* `<mirrorOf>*,!repo1</mirrorOf>` _匹配所有远程仓库，repo1除外，使用感叹号将仓库从匹配中排除_

> 需要注意的是，由于镜像仓库完全屏蔽了被镜像仓库，当镜像我不要两个i稳定或停止服务的时候，maven仍将无法访问被镜像仓库，因而将无法下载构件

### 插件仓库

```xml
<pluginRepositories>
    <pluginRepository>
        <id>central</id>
        <name>maven plugin repository</name>
        <url>http://repo1.maven.org/maven2</url>
        <layout>default</layout>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
        <releases>
            <updatePolicy>never</updatePolicy>
        </releases>
    </pluginRepository>
</pluginRepositories>
```

> 除了pluginRepository和pluginRepositories不同之外，其余所有子元素表达的含义与依赖远程仓库配置完全一样，默认插件仓库的地址就是中央仓库，关闭了对snapshot的支持，以防止引入snapshot版本的插件而导致的不稳定问题，一般只有在很少情况下才会配置自己的插件仓库。

> maven官方插件，在使用时可以省略groupId(org.apache.maven.plugins),默认会自动补齐


