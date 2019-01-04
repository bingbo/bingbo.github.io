```properties
##日志级别为debug级别，输出方式为console和catalina两种
log4j.rootLogger = DEBUG, CONSOLE, CATALINA, stdout


## catalina输出方式定义
log4j.appender.CATALINA = org.apache.log4j.DailyRollingFileAppender
log4j.appender.CATALINA.File = ${catalina.base}/catalina.out
log4j.appender.CATALINA.Append = true
log4j.appender.CATALINA.Encoding = UTF-8
# Roll-over the log once per day
log4j.appender.CATALINA.DatePattern = '.'yyyy-MM-dd'.log'
log4j.appender.CATALINA.layout = org.apache.log4j.PatternLayout
log4j.appender.CATALINA.layout.ConversionPattern = %d [%t] %-5p %c- %m%n

## console控制台输出方式定义
log4j.appender.CONSOLE = org.apache.log4j.ConsoleAppender
log4j.appender.CONSOLE.Encoding = UTF-8
log4j.appender.CONSOLE.layout = org.apache.log4j.PatternLayout
log4j.appender.CONSOLE.layout.ConversionPattern = %d [%t] %-5p %c- %m%n


##自定义日志输出方式样例
log4j.logger.CUSTOM = INFO, CUSTOMLOG
log4j.additivity.CUSTOM = false
log4j.appender.CUSTOMLOG = org.apache.log4j.DailyRollingFileAppender
log4j.appender.CUSTOMLOG.File = ${catalina.base}/custom.out
log4j.appender.CUSTOMLOG.Append = true
log4j.appender.CUSTOMLOG.Encoding = UTF-8
# Roll-over the log once per day
log4j.appender.CUSTOMLOG.DatePattern = '.'yyyy-MM-dd'.log'
log4j.appender.CUSTOMLOG.layout = org.apache.log4j.PatternLayout
log4j.appender.CUSTOMLOG.layout.ConversionPattern = %d [%t] %-5p %c- %m%n


##标准输出方式定义
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d{ABSOLUTE} %5p %t %c{2}:%L - %m%n

##以trace级别记录输出com.ibingbo.dao和com.ibingbo.mapper包下类执行日志,可以输出执行sql
log4j.logger.com.ibingbo.dao=TRACE,stdout,CATALINA
log4j.logger.com.ibingbo.mapper=TRACE,stdout,CATALINA
##以指定级别记录相应包下执行日志
log4j.logger.org.apache.http=ERROR
##通过log4j.category.xxx指定warn等高级别的日志记录等级，则可屏蔽该xxx包的执行日志
log4j.category.org.springframework.beans.factory=WARN
```

## Appender输出方式

### org.apache.log4j.ConsoleAppender

> 控制台输出 

_参数_

* Threshold=WARN:指定日志消息的输出最低层次。 
* ImmediateFlush=true:默认值是true,意谓着所有的消息都会被立即输出。 
* Target=System.err：默认情况下是：System.out,指定输出控制台 
* encoding 指定日志编码

### org.apache.log4j.FileAppender

> 文件输出 

_参数_

* Threshold=WARN:指定日志消息的输出最低层次。 
* ImmediateFlush=true:默认值是true,意谓着所有的消息都会被立即输出。 
* File=mylog.txt:指定消息输出到mylog.txt文件。 
* Append=false:默认值是true,即将消息增加到指定文件中，false指将消息覆盖指定的文件内容。 


### org.apache.log4j.DailyRollingFileAppender

> 每天产生一个日志文件

_参数_

* Threshold=WARN:指定日志消息的输出最低层次。 
* ImmediateFlush=true:默认值是true,意谓着所有的消息都会被立即输出。 
* File=mylog.txt:指定消息输出到mylog.txt文件。 
* Append=false:默认值是true,即将消息增加到指定文件中，false指将消息覆盖指定的文件内容。 
* DatePattern='.'yyyy-ww:每周滚动一次文件，即每周产生一个新的文件。当然也可以指定按月、周、天、时和分。即对应的格式如下： 

    * '.'yyyy-MM: 每月 
    * '.'yyyy-ww: 每周 
    * '.'yyyy-MM-dd: 每天 
    * '.'yyyy-MM-dd-a: 每天两次 
    * '.'yyyy-MM-dd-HH: 每小时 
    * '.'yyyy-MM-dd-HH-mm: 每分钟 


### org.apache.log4j.RollingFileAppender

> 文件大小到达指定尺寸的时候产生一个新的文件

_参数_

* Threshold=WARN:指定日志消息的输出最低层次。 
* ImmediateFlush=true:默认值是true,意谓着所有的消息都会被立即输出。 
* File=mylog.txt:指定消息输出到mylog.txt文件。 
* Append=false:默认值是true,即将消息增加到指定文件中，false指将消息覆盖指定的文件内容。 
* MaxFileSize=100KB: 后缀可以是KB, MB 或者是 GB. 在日志文件到达该大小时，将会自动滚动，即将原来的内容移到mylog.log.1文件。 
* MaxBackupIndex=2:指定可以产生的滚动文件的最大数。
* layout 日志格式布局
* name appender的名字
* filter 过滤器，决定一个事件是否由该appender处理，多个过滤器可以使用CompositeFilter
* immediateFlush 默认为true，表示数据立即写入disk，可能会影响性能
* encoding 指定日志编码
* bufferedIO 默认为true,记录将先写到缓存buffer中，当buffer满时或immediateFlush设置时,记录写入disk
* bufferSize 缓冲区buffer的大小，默认为8192字节


### org.apache.log4j.WriterAppender

> 将日志信息以流格式发送到任意指定的地方

_参数_

* layout
* name
* threshold 优先级
* filter 过滤器，决定一个事件是否由该appender处理，多个过滤器可以使用CompositeFilter
* immediateFlush 默认为true，表示数据立即写入disk，可能会影响性能
* encoding 指定日志编码

### 各种Appender关系类图

![](https://github.com/bingbo/blog/blob/master/images/log4j-appender.jpg)

## Layout日志格式

### org.apache.log4j.HTMLLayout

> 以HTML表格形式布局 

_参数_

* LocationInfo=true:默认值是false,输出java文件名称和行号 
* Title=my app file: 默认值是 Log4J Log Messages. 

### org.apache.log4j.PatternLayout

> 可以灵活地指定布局模式

_参数_

ConversionPattern=%m%n :指定怎样格式化指定的消息。 

_这里需要说明的就是日志信息格式中几个符号所代表的含义： _

* －X号: X信息输出时左对齐； 
* %p: 输出日志信息优先级，即DEBUG，INFO，WARN，ERROR，FATAL, 
* %d: 输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，比如：%d{yyy MMM dd HH:mm:ss,SSS}，输出类似：2002年10月18日 22：10：28，921 
* %r: 输出自应用启动到输出该log信息耗费的毫秒数 
* %c: 输出日志信息所属的类目，通常就是所在类的全名 
* %t: 输出产生该日志事件的线程名 
* %l: 输出日志事件的发生位置，相当于%C.%M(%F:%L)的组合,包括类目名、发生的线程，以及在代码中的行数。举例：Testlog4.main(TestLog4.java:10) 
* %x: 输出和当前线程相关联的NDC(嵌套诊断环境),尤其用到像java servlets这样的多客户多线程的应用中。 
* %%: 输出一个"%"字符 
* %F: 输出日志消息产生时所在的文件名称 
* %L: 输出代码中的行号 
* %m: 输出代码中指定的消息,产生的日志具体信息 
* %n: 输出一个回车换行符，Windows平台为"\r\n"，Unix平台为"\n"输出日志信息换行 

_可以在%与模式字符之间加上修饰符来控制其最小宽度、最大宽度、和文本的对齐方式。如：_ 

* %20c：指定输出category的名称，最小的宽度是20，如果category的名称小于20的话，默认的情况下右对齐。 
* %-20c:指定输出category的名称，最小的宽度是20，如果category的名称小于20的话，"-"号指定左对齐。 
* %.30c:指定输出category的名称，最大的宽度是30，如果category的名称大于30的话，就会将左边多出的字符截掉，但小于30的话也不会有空格。 
* %20.30c:如果category的名称小于20就补空格，并且右对齐，如果其名称长于30字符，就从左边交远销出的字符截掉。

### org.apache.log4j.SimpleLayout

> 包含日志信息的级别和信息字符串

_参数_

* LocationInfo=true:默认值是false,输出java文件和行号 

### org.apache.log4j.TTCCLayout

> 包含日志产生的时间、线程、类别等等信息_

### 各种Layout格式关系类图

![](https://github.com/bingbo/blog/blob/master/images/log4j-layout.jpg)

## mybatis日志记录


Mybatis内置的日志工厂提供日志功能，具体的日志实现有以下几种工具：

* SLF4J
* Apache Commons Logging
* Log4j 2
* Log4j
* JDK logging

具体选择哪个日志实现工具由MyBatis的内置日志工厂确定。它会使用最先找到的（按上文列举的顺序查找）。 如果一个都未找到，日志功能就会被禁用

不少应用服务器的classpath中已经包含Commons Logging，如Tomcat和WebShpere， 所以MyBatis会把它作为具体的日志实现。记住这点非常重要。这将意味着，在诸如 WebSphere的环境中——WebSphere提供了Commons Logging的私有实现，你的Log4J配置将被忽略

你可以通过在MyBatis的配置文件mybatis-config.xml里面添加一项setting（配置）来选择一个不同的日志实现

```xml
<configuration>
<settings>
  ...
      <setting name="logImpl" value="LOG4J"/>
          ...
</settings>
</configuration>
```

> logImpl可选的值有：SLF4J、LOG4J、LOG4J2、JDK_LOGGING、COMMONS_LOGGING、STDOUT_LOGGING、NO_LOGGING 或者是实现了接口org.apache.ibatis.logging.Log的类的完全限定类名

## 程序中运用

### 直接使用log4j打日志

```xml
<dependency>
  <groupId>log4j</groupId>
  <artifactId>log4j</artifactId>
  <version>1.2.17</version>
</dependency>
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
  <version>1.7.22</version>
</dependency>
```

```java
...
private static final org.apache.log4j.Logger logger = LogManager.getLogger(UserController.class);
...
logger.info("user_log_test_log4j");
```

### 使用slf4j打日志

> 首先得添加slf4j-log4j12依赖,slf4j的使用是通过适配器间接调用log4j的api进行打日志的

```xml
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
  <version>1.7.22</version>
</dependency>
```

> 其次程序调用

```java
...
private final static org.slf4j.Logger LOGGER = org.slf4j.LoggerFactory.getLogger(UserController.class);
...
LOGGER.debug("user_log_test_slf4j");
```

### slf4j的优点

* 使用SLF4J，可以使它独立于任何的日志实现，这就意味着不需要管理多个库和多个日志文件。你的客户端将会体会到这一点。
* SLF4J提供了占位日志记录，通过移除对isDebugEnabled(), isInfoEnabled()等等的检查提高了代码的可读性。
* 通过使用日志记录方法，直到你使用到的时候，才会去构造日志信息（字符串），这就同时提高了内存和CPU的使用率。
* 做一个侧面的说明，越少的临时字符串，垃圾回收器就意味着越少的工作，这就意味着为你的应用程序提供更好的吞吐量和性能。

### log4j默认配置

log4j默认是在LogManager中加载指定的配置文件，根据配置文件解析生成相应格式的Appender进行日志打印的。如下代码所示

```java
public class LogManager {

  /**
   * @deprecated This variable is for internal use only. It will
   * become package protected in future versions.
   * */
  static public final String DEFAULT_CONFIGURATION_FILE = "log4j.properties";
  
  static final String DEFAULT_XML_CONFIGURATION_FILE = "log4j.xml";  
   
  /**
   * @deprecated This variable is for internal use only. It will
   * become private in future versions.
   * */
  static final public String DEFAULT_CONFIGURATION_KEY="log4j.configuration";

  /**
   * @deprecated This variable is for internal use only. It will
   * become private in future versions.
   * */
  static final public String CONFIGURATOR_CLASS_KEY="log4j.configuratorClass";

  /**
  * @deprecated This variable is for internal use only. It will
  * become private in future versions.
  */
  public static final String DEFAULT_INIT_OVERRIDE_KEY = 
                                                 "log4j.defaultInitOverride";


  static private Object guard = null;
  static private RepositorySelector repositorySelector;

  static {
    // By default we use a DefaultRepositorySelector which always returns 'h'.
    Hierarchy h = new Hierarchy(new RootLogger((Level) Level.DEBUG));
    repositorySelector = new DefaultRepositorySelector(h);

    /** Search for the properties file log4j.properties in the CLASSPATH.  */
    String override =OptionConverter.getSystemProperty(DEFAULT_INIT_OVERRIDE_KEY,
						       null);

    // if there is no default init override, then get the resource
    // specified by the user or the default config file.
    if(override == null || "false".equalsIgnoreCase(override)) {

      String configurationOptionStr = OptionConverter.getSystemProperty(
							  DEFAULT_CONFIGURATION_KEY, 
							  null);

      String configuratorClassName = OptionConverter.getSystemProperty(
                                                   CONFIGURATOR_CLASS_KEY, 
						   null);

      URL url = null;

      // if the user has not specified the log4j.configuration
      // property, we search first for the file "log4j.xml" and then
      // "log4j.properties"
      if(configurationOptionStr == null) {	
	url = Loader.getResource(DEFAULT_XML_CONFIGURATION_FILE);
	if(url == null) {
	  url = Loader.getResource(DEFAULT_CONFIGURATION_FILE);
	}
      } else {
	try {
	  url = new URL(configurationOptionStr);
	} catch (MalformedURLException ex) {
	  // so, resource is not a URL:
	  // attempt to get the resource from the class path
	  url = Loader.getResource(configurationOptionStr); 
	}	
      }
      
      // If we have a non-null url, then delegate the rest of the
      // configuration to the OptionConverter.selectAndConfigure
      // method.
      if(url != null) {
	    LogLog.debug("Using URL ["+url+"] for automatic log4j configuration.");
        try {
            OptionConverter.selectAndConfigure(url, configuratorClassName,
					   LogManager.getLoggerRepository());
        } catch (NoClassDefFoundError e) {
            LogLog.warn("Error during default initialization", e);
        }
      } else {
	    LogLog.debug("Could not find resource: ["+configurationOptionStr+"].");
      }
    } else {
        LogLog.debug("Default initialization of overridden by " + 
            DEFAULT_INIT_OVERRIDE_KEY + "property."); 
    }  
  }
...
} 
```

### log4j初始化调用流程

![](https://github.com/bingbo/blog/blob/master/images/log4j_seq.jpg)