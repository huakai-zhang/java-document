Spring从Spring2.5开始支持注解注入，注解在省去了很多繁琐的XML配置的同时，也失去了灵活性。context:component-scan可以让我们要根据需要来选择是否启用注解注入。但是在配置过context:component-scan之后就不用配置context:annotation-config标签，因为前者已经包含后者。这个配置文件中必须声明xmlns:context 这个xml命名空间，在schemaLocation中需要指定schema：
>http://www.springframework.org/schema/context
  http://www.springframework.org/schema/context/spring-context-3.0.xsd
 
### 使用过滤器进行自定义扫描
默认情况下，类被自动发现并注册bean的条件是：使用@Component，@Repository，@Service，@Controller注解或者使用@Component的自定义注解，可以通过过滤器修改上面的行为，如：下面例子的xml配置忽略所有的@repository注解并用“Stub”替代：
```xml
<bean>
    <context:component-scan base-package="com.spring">
        <context:include-filter type="regex" expression=".*Stur.*Repository"/>
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Repository"/>
    </context:component-scan>
</bean>
```
**context:component-scan**还提供了两个子标签 **context:include-filter**和**context:exclude-filter**。filter标签的type和表达式说明如下：
| Filter Type | Examples Expression    | Description   |
|--| :-------------| -- |
|annotation	|org.example.SomeAnnotation	|符合SomeAnnoation的target class|
|assignable|	org.example.SomeClass	|指定class或interface的全名|
|aspectj	|org.example..*Service+	|AspectJ语法|
|regex|	org\.example\.Default.*|	Regelar Expression|
|custom	|org.example.MyTypeFilter	|Spring3新增自订Type，实作|org.springframework.core.type.TypeFilter|

```xml
<!--扫包 扫描@service等其他注解 而不扫描controller-->
<context:component-scan base-package="com.web.controller">  
	<context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>  
<!--配置springmvc扫包 只扫controller注解-->
<context:component-scan base-package="com.web" use-default-filters="false">
	<context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
<!--配置springmvc扫包 只扫dao和service文件夹下注解-->
<context:component-scan base-package="com.web" use-default-filters="false">
		<context:include-filter type="regex" expression="com\.web\.[^.]+(Dao|Service)"/>
</context:component-scan>
```
### 禁止自动发现与注册
还可以使用use-default-filters="false"禁止自动发现与注册。
context:component-scan具有一个属性**use-default-filters**，默认值为true，表示使用默认的过滤器，此处的默认过滤器，会扫描包含Service,Component,Repository,Controller注解修饰的类。
### 自定义 Bean 名称
扫描过程中组件被自动检测，那么Bean名称是由 ``org.springframework.beans.factory.support.BeanNameGenerator``生成的(@Component，@Repository，@Service，@Controllor都会有个name属性用于显式设置BeanName)，例如：@Service("myUserService") 如果我们显式指定括号里的名称即为当该类被自动注册到配置文件中是的id，如果没有就是userService

可自定义bean命名策略，实现BeanNameGenerator接口，并一定要包含一个无参数构造器
```xml
<bean>
	<context:component-scan base-package="org.example" name-generator="org.example.MyNameGeberator"/>
</bean>
```
```java
// 定义一个类
@Component
public class BeanAnnotation {
	public void say(String arg) {
		System.out.println("BeanAnnotation : " + arg);
	}
}

// 单元测试
@RunWith(BlockJUnit4ClassRunner.class)
public class TestBeanAnnotation extends UnitTestBase {
	public TestBeanAnnotation() {
		super("classpath*:spring-beanannotation.xml");
	}
	@Test
	public void testSay() {
		BeanAnnotation bean = super.getBean("beanAnnotation");
		bean.say("This is test.");
	}
}
```
运行结果：
BeanAnnotation : This is test.
修改类的注解@Component("bean")以及测试里的getBean("bean")，结果一致
### 作用域(Scope)
.通常情况下自动查找的Spring组件，其scope是 ``singleton``，Spring2.5提供了一个标示scope的注解@Scope：
```java
@Scope("prototype")
@Repository
public class MovoeFinderImpl inplements MovieFinder{

}
```
也可以自定义scope策略，实现 ```org.springframework.context.annotation.ScopeMetadataResolver`接口并提供一个无参构造器
```xml
<bean>
	<context:compontent-scan base-package="org.example" scope-resolver="org.example.MyScopeResolver"/>
</bean>
```

```java
// 为类BeanAnnotation添加注解@Scope("prototype")
// 添加方法
public void myHashCode() {
	System.out.println("BeanAnnotation : " +this.hashCode());
}

// 修改单元测试@test
@Test
public void testScpoe() {
	BeanAnnotation bean = super.getBean("beanAnnotation");
	bean.myHashCode();
	bean = super.getBean("beanAnnotation");
	bean.myHashCode();
}
```
测试结果：
BeanAnnotation : 1524951007
BeanAnnotation : 1781731351 不是同一个对象
如果是@Scope：
BeanAnnotation : 1114656084
BeanAnnotation : 1114656084  同一个对象

### 代理方式
可以使用scoped-proxy属性指定代理，有三个值可选：no,interfaces,targetClass
```xml
<bean>
	<context:compontent-scan base-package="org.example" scoped-proxy="interfaces"/>
</bean>
```
