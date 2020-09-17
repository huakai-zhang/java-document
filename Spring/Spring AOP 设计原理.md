## 简介


AOP是OOP的延续，是Aspect Oriented Programming的缩写，意思是面向切面编程。可以通过预编译方式和运行期动态代理实现在不修改源代码的情况下给程序动态统一添加功能的一种技术。AOP实际是GoF设计模式的延续，设计模式孜孜不倦追求的是调用者和被调用者之间的解耦，AOP可以说也是这种目标的一种实现。 现在做的一些非业务，如：日志、事务、安全等都会写在业务代码中(也即是说，这些非业务类横切于业务类)，但这些代码往往是重复，复制——粘贴式的代码会给程序的维护带来不便，AOP就实现了把这些业务需求与系统需求分开来做。这种解决的方式也称代理机制。AOP的核心构造是切面，它将那些影响多个类的行为(有一定的规则，可以单独把一定规律的规则单独分离出来)封装到可重用的模块中。

## AOP实现方式

预编译

- AspectJ 运行期动态代理(JDK动态代理，CGLib动态代理) 
- SpringAOP，JbossAOP


- 切面(Aspect) :官方的抽象定义为“一个关注点的模块化，这个关注点可能会横切多个对象”，例：“切面”就是类TestAspect所关注的具体行为,AServiceImpl.barA()的调用就是切面TestAspect所关注的行为之一。“切面"在ApplicationContext中<aop:aspect>来配置。 
- 连接点(Joinpoint) :程序执行过程中的某一行为，例如，UserService .get的调用或者UserService.delete抛出异常等行为。 
- 通知(Advice) :“切面对于某个“连接点"所产生的动作，例如，TestAspect 中对com spring.service包下所有类的方法进行日志记录的动作就是一个 Advice。其中，一个“切面”可以包含多个“Advice",例如ServiceAspect 
- 切入点(Pointcut) : 匹配连接点的断言，在AOP中通知和一个切入点表达式关联。例如，TestAspect中的所有通知所关注的连接点，都由切入点表达式execution(* com spring service. *.* (. ))来诀定。 
- 目标对象(Target Object) :被一个或者多个切面所通知的对象。例如，AServcieImpl 和BServiceImpl，当然在实际运行时，Spring AOP采用代理实现，实际AOP操作的是TargetObject的代理对象。 
- AOP代理(AOP Proxy) :在Spring AOP中有两种代理方式,JDK动态代理和CGLIB代理。默认情祝下, TargetObject实现了接口时，则采用JDK动态代理，例如，AServiceImpl; 反之，采用CGLIB代理，例如，BServiceImplo 强制使用CGLIB代理需要将<aop.config>的proxy-target-class 属性设为true。 
- 织入：把切面连接到其他的应用程序或对象上，并创建一个被通知的对象，分为：编译时织入，类加载时织入，执行时织入(Spring AOP利用的是运行时织入)。

## 通知(Advice) 类型

- 前置通知(Beforeadvice):在某连接点(JoinPoint)之前执行的通知，但这个通知不能阻止连接点前的执行。ApplicationContext中在<aop:aspect>里面使用<aop:beforex>元素进行声明。例如，TestAspect 中的doBefore方法。 
- 后置通知(After advice) :当某连接点退出的时候执行的通知(不论是正常返回还是异常退出)。ApplicationContext中在<aop:aspect>里面使用<aop:affter>元素进行声明。例如，ServiceAspect 中的returnAfter方法，所以Teser.中调用UserService. delete抛出异常时，returnAfter 方法仍然执行。. 
- 返回后通知(Afterreturnadvice):在某连接点正常完成后执行的通知，不包括抛出异常的情祝。ApplicationContext中在<aop:aspect>里面使用<after-returning>元素进行声明。 
- 环绕通知(Around advice) :包围-个连接点的通知，类似Web中Servlet规范中的Filter的doFilter 方法。可以在方法的调用前后完成自定义的行为，也可以选择不执行。ApplicationContext 中在<aop:aspect>里面使用<aop:around>元素进行声明。例如，ServiceAspect 中的around方法。 
- 抛出异常后通知(After throwing advice) :在方法抛出异常退出时执行的通知。ApplicationContext中在<aop:aspect>里面使用<aop:after-throwing>元素进行声明。例如，ServiceAspect 中的returnThrow方法。 注:可以将多个通知应用到一个目标对象上，即可以将多个切面织入到同一目标对象。


## 注解

使用注解配置SpringAOP总体分为两步，第一步是在xml文件中声明激活自动扫描组件功能，同时激活自动代理功能（来测试AOP的注解功能）：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xmlns:aop="http://www.springframework.org/schema/aop"
   xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
	<!-- 激活组件扫描功能,在包com.spring.aop及其子包下面自动扫描通过注解配置的组件 -->
	<context:component-scan base-package="com.spring"/>
	<!-- 激活自动代理功能 -->
	<aop:aspectj-autoproxy proxy-target-class="true"/>
</beans>
```

第二步是为Aspect切面类型添加注解：

```java
// 声明这个类是被SpringIOC容器来管理的，如果不声明就无法做到自动织入
@Component
// 这个类被声明为是一个需要动态织入到我们的虚拟切面中的类
@Aspect
public class AnnotationAspect {

    private final static Logger logger = Logger.getLogger(AnnotationAspect.class);

    // 声明切点
    // 因为要利用反射机制去读取这个切面中的所有的注解信息
    @Pointcut("execution(* com.spring.aop.service..*(..))")
    public void pointcutConfig(){}

    @Before("pointcutConfig()")
    public void before(JoinPoint joinPoint) {
        logger.info("调用方法之前执行" + joinPoint);
    }

    @After("pointcutConfig()")
    public void after(JoinPoint joinPoint) {
        logger.info("调用方法之后执行" + joinPoint);
    }

    @AfterReturning("pointcutConfig()")
    public void afterReturn(JoinPoint joinPoint) {
        logger.info("调用获得返回值之后执行" + joinPoint);
    }

    @AfterThrowing("pointcutConfig()", throwing="ex")
    public void afterThrow(JoinPoint joinPoint) {
        System.out.println("切点的参数" + Arrays.toString(joinPoint.getArgs()));
        System.out.println("切点的方法" + joinPoint.getKind());
        System.out.println(joinPoint.getSignature());
        // 生成以后的代理对象
        System.out.println(joinPoint.getTarget());
        // 当前类的本身（通过反射机制去调用）
        System.out.println(joinPoint.getThis());

        logger.info("抛出异常之后执行" + joinPoint);
    }
}
@ContextConfiguration(locations = {"classpath*:application-context.xml"})
@RunWith(SpringJUnit4ClassRunner.class)
public class MemberManagerServiceTest {

    @Autowired
    MemberManagerService memberManagerService;

    @Test
    @Ignore
    public void testAdd() {
        memberManagerService.add(null);
    }

    // 做事务代理的时候
    // TransactionManage来管理事务操作（切面）
    // DataSource,SessionFactory
    // DataSource包含了链接信息，事物的提交或回滚一些基础功能
    // 通过连接点是可以获得到方法（切点）具体操作哪个DataSource
    // 通过切面通知类型，去执行DataSource的功能方法
    @Test
    public void testRemove() {
        try {
            memberManagerService.remove(0);
        } catch (Exception e) {
            //e.printStackTrace();
        }
    }

    public void testModify() {
        memberManagerService.modify(null);
    }

    public void testQuery() {
        memberManagerService.query("");
    }

}
```

可以看到，虽然并没有对MemberService类包括其调用方式做任何改变，但是 Spring 仍然拦截到了其中方法的调用。

## XML配置方式

```java
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:tx="http://www.springframework.org/schema/tx"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx-3.0.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop-3.0.xsd">
    <!-- 声明一个需要织入到虚拟切面的逻辑 -->
    <bean id="logAspect" class="com.spring.aop.aspect.LogAspect">

    </bean>

    <aop:config>
        <aop:aspect ref="logAspect">
            <!-- 切点，具体的方法声明 -->
            <aop:pointcut id="logPointcut" expression="execution(* com.spring.aop.service..*(..))"/>
            <aop:before method="before" pointcut-ref="logPointcut"/>
            <aop:after-returning method="afterReturn" pointcut-ref="logPointcut"/>
            <aop:after method="after" pointcut-ref="logPointcut"/>
            <aop:after-throwing method="afterThrow" pointcut-ref="logPointcut"/>
        </aop:aspect>
    </aop:config>
</beans>
```

个人觉得不如注解灵活和强大，你可以不同意这个观点，但是不知道如下的代码会不会让你的想法有所改善：

```java
//配置切入点,该方法无方法体,主要为方便同类中其他方法使用此处配置的切入点
@Pointcut("execution(* com.spring.aop.service..*(..))")
public void aspect(){ }

//配置前置通知,拦截返回值为com.spring.aop.bean.User的方法
@Before("execution(com.spring.aop.bean.User com.spring.aop.service..*(..))")
public void beforeReturnUser(JoinPoint joinPoint){
    log.info("beforeReturnUser " + joinPoint);
}

//配置前置通知,拦截参数为com.spring.aop.bean.User的方法
@Before("execution(* com.spring.aop.service..*(com.spring.aop.bean.User))")
public void beforeArgUser(JoinPoint joinPoint){
    log.info("beforeArgUser " + joinPoint);
}

//配置前置通知,拦截含有long类型参数的方法,并将参数值注入到当前方法的形参id中
@Before("aspect()&&args(id)")
public void beforeArgId(JoinPoint joinPoint, long id){
    log.info("beforeArgId " + joinPoint + "\tID:" + id);
}
@Service
public class MemberManagerService {

    private final static Logger logger = Logger.getLogger(MemberManagerService.class);

    public void add(Member member) {
        logger.info("增加用户");
    }

    public boolean remove(long id) throws Exception {
        logger.info("删除用户");
        throw new Exception("这是自己抛出来的异常");
    }

    public boolean modify(Member member) {
        logger.info("修改用户");
        return true;
    }

    public boolean query(String loginName) {
        logger.info("查询用户");
        return true;
    }
}
```

## 切入点表达式的配置规则

通常情况下，表达式中使用”execution“就可以满足大部分的要求。表达式格式如下：

```java
execution(modifiers-pattern? ret-type-pattern declaring-type-pattern? name-pattern(param-pattern) throws-pattern?
```

- modifiers-pattern：方法的操作权限 
- ret-type-pattern：返回值 
- declaring-type-pattern：方法所在的包 
- name-pattern：方法名 
- parm-pattern：参数名 
- throws-pattern：异常

其中，除 ret-type-pattern 和 name-pattern 之外，其他都是可选的。上例中，execution(* com.spring.aop.service…*(…))表示 com.spring.aop.service 包下，返回值为任意类型；方法名任意；参数不作限制的所有方法。

## 通知参数

可以通过args来绑定参数，这样就可以在通知（Advice）中访问具体参数了。例如，<aop:aspect>配置如下：

```java
<aop:config>
    <aop:aspect ref="xmlAspect">
        <aop:pointcut id="simplePointcut" expression="execution(* com.spring.aop.service..*(..)) and args(msg,..)"/>
        <aop:after pointcut-ref=" simplePointcut" Method="after"/>
    </aop:aspect>
</aop:config>
```

上面的代码args(msg,…)是指将切入点方法上的第一个String类型参数添加到参数名为msg的通知的入参上，这样就可以直接使用该参数啦。

## 访问当前的连接点

在上面的Aspect切面Bean中已经看到了，每个通知方法第一个参数都是JoinPoint。其实，在Spring中，任何通知（Advice）方法都可以将第一个参数定义为org.aspectj.lang.JoinPoint类型用以接受当前连接点对象。JoinPoint接口提供了一系列有用的方法，比如getArgs()（返回方法参数）、getThis()（返回代理对象）、getTarget()（返回目标）、getSignature()（返回正在被通知的方法相关信息）和toString()（打印出正在被通知的方法的有用信息）。

