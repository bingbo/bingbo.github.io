### 定义数据源

```xml
<!-- 数据源配置 -->
<bean
    class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="locations">
        <value>classpath:jdbc.properties</value>
    </property>
</bean>

<bean id="dataSource" destroy-method="close"
    class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="${jdbc.driverClassName}" />
    <property name="url" value="${jdbc.url}" />
    <property name="username" value="${jdbc.username}" />
    <property name="password" value="${jdbc.password}" />
</bean>
<!-- 数据源配置 -->
```

### 声明事务配置

```xml
<!-- 添加配置事务管理，并指定数据源 -->
<bean id="transactionManager"
    class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource" />
</bean>
<!-- 添加配置事务管理，并指定数据源 -->
```

### 通过AOP进行配置并应用事务

```xml
<!--通过aop使用事务在user service 上-->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <tx:method name="get*" read-only="true" rollback-for="UserException" propagation="REQUIRED" isolation="READ_COMMITTED" timeout="-1"/>
        <tx:method name="*"/>
    </tx:attributes>
</tx:advice>
<aop:config>
    <aop:pointcut id="userServiceOperation" expression="execution(* com.ibingbo.services.UserService.*(..))"/>
    <aop:advisor advice-ref="txAdvice" pointcut-ref="userServiceOperation"/>
</aop:config>
<!--通过aop使用事务在user service 上-->

### 通过注释方式应用事务

```xml
<!-- 启用具有注释的事务划分 然后在类或方法里用@Transactional配置注释使用事务-->
<tx:annotation-driven transaction-manager="transactionManager" mode="proxy" proxy-target-class="true" />
<!-- 启用具有注释的事务划分 -->
```

```java
@Repository
public class StudentDaoImpl implements StudentDao {

	@Autowired
	private StudentMapper studentMapper;
	

    //这里通过@Transaction定义声明应用事务
	@Transactional(propagation = Propagation.REQUIRES_NEW)
	@Override
	public List<StudentDO> getStudents() {
		// TODO Auto-generated method stub
		return this.studentMapper.getStudents();
	}
}
```
