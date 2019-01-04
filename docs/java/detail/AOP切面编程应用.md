
## 定义自定义注释

> 创建一个自定义注解，用来打日志

```java
//Target表示可以在类，属性，方法中应用注解
@Target({ElementType.TYPE, ElementType.FIELD, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Log {

    String remark() default "";
}
```

## 切面AOP

### springContext.xml中配置，开房切面注解

```xml
<!--在spring-context.xml中开启切面注解-->
<!--切面应用-->
<aop:aspectj-autoproxy proxy-target-class="true"/>
```

### java程序中运用@Aspect注解标注一个切面

```java
@Aspect
@Component
public class LogAspect {


    //定义切点
    @Pointcut("@annotation(com.ibingbo.annotation.Log)")
    public void log() {
        System.out.println("log this controller ....");
    }

    //定义advise，在切点的哪里（前后）拦截
    @Before("log()")
    public void doLog() {

        System.out.println("this is aspect before dolog ....");
    }

    //直接定义切点，advise并在连接点之后运行
    @After("@annotation(com.ibingbo.annotation.Log)")
    public void afterDoLog(JoinPoint joinPoint) throws ClassNotFoundException {
        System.out.println("this is aspect after dolog ....");
        System.out.println(joinPoint.getTarget());
        String targetName = joinPoint.getTarget().getClass().getName();
        String methodName = joinPoint.getSignature().getName();
        Object[] args = joinPoint.getArgs();
        System.out.println(args);
        Class targetClass = Class.forName(targetName);
        java.lang.reflect.Method[] methods = targetClass.getMethods();
        String value = "";
        for (java.lang.reflect.Method method : methods) {
            if (method.getName().equals(methodName)) {
                Class[] classes = method.getParameterTypes();
                if (classes.length == args.length) {
                    value = method.getAnnotation(Log.class).remark();
                }
            }
        }
        System.out.println("annotation parameter is " + value);
    }

    //在连接点前后运行，通过ProceedingJoinPoint控制
    @Around("log()")
    public Object aroundDoLog(ProceedingJoinPoint point) {
        System.out.println("this is aspect around dolog ...");
        System.out.println(point.getArgs().toString());
        Object o = null;
        try {
             o = point.proceed();
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
        System.out.println(o);
        System.out.println("this is aspect around dolog ...");
        return o;
    }

    @AfterThrowing("log()")
    public void afterThrowingDoLog() {
        System.out.println("this is aspect after throwing dolog ...");
    }
}
```

### 运用@Log注解通过切面打日志

```java
//在controller中的应用
@Controller
@RequestMapping("/student")
@Scope("prototype")
public class StudentController {

	private static final Logger logger = LoggerFactory.getLogger(StudentController.class);

	@Autowired
	private StudentService service;
	
	@RequestMapping("/list")
	@ResponseBody
	@Log(remark = "list api") //这里运用注解打日志
	public Map<String, Object> list(){
		Map<String, Object> map=new HashMap<String, Object>();
		map.put("errno", 0);
		map.put("errmsg", "success");
		try{
			map.put("data", this.service.getStudents());
			System.out.println("student_controller: " + this);
		}catch(Exception exception){
			map.put("errno", -1);
			map.put("errmsg", exception.getMessage());
		}
		return map;
	}
}
```
