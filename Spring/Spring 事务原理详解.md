# 1 Spring 事务配置

spring支持编程式事务管理和声明式事务管理两种方式：

- ``编程式事务`` 使用TransactionTemplate或者直接使用底层的PlatformTransactionManager。对于编程式事务管理，spring推荐使用TransactionTemplate。
- ``声明式事务`` 建立在AOP之上的。其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。声明式事务最大的优点就是不需要通过编程的方式管理事务，这样就不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中做相关的事务规则声明(或通过基于@Transactional注解的方式)，便可以将事务规则应用到业务逻辑中。

显然声明式事务管理要优于编程式事务管理，这正是spring倡导的非侵入式的开发方式。声明式事务管理使业务代码不受污染，一个普通的POJO对象，只要加上注解就可以获得完全的事务支持。和编程式事务相比，声明式事务唯一不足地方是，它的最细粒度只能作用到方法级别，无法做到像编程式事务那样可以作用到代码块级别。但是即便有这样的需求，也存在很多变通的方法，比如，可以将需要进行事务管理的代码块独立为方法等等。

声明式事务管理也有两种常用的方式，一种是基于 tx 和 aop 名字空间的xml配置文件，另一种就是基于@Transactional注解。显然基于注解的方式更简单易用更清爽。

## 1.1 编程式事务管理

```xml
<!-- 配置业务层类 -->
<bean id="accountService" class="cn.muke.spring.demol.AccountServiceImpl">
	<property name="accountDao" ref="accountDao" />
	<!-- 注入事务管理的模版 -->
	<property name="transactionTemplate" ref="transactionTemplate"></property>
</bean>
<!-- 配置DAO类(简化，会自动配置JdbcTemplate) -->                                       
<bean id="accountDao" class="cn.muke.spring.demol.AccountDaoImpl"/>
<!-- 配置事物管理器 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager"/>
<!-- 配置事物管理的模版：Spring为了简化事务管理的代码 -->
<bean id="transactionTemplate" class="org.springframework.transaction.support.TransactionTemplate">
	<property name="transactionManager" ref="transactionManager"></property>
</bean>
```

```java
public interface AccountDao {
	public void outMoney(String out,Double money);
	public void inMoney(String in,Double money);
}
 
public class AccountDaoImpl extends JdbcDaoSupport implementsAccountDao { 
	public void outMoney(String out, Double money) {
		String sql = "update account set money = money - ? where name= ?";
		this.getJdbcTemplate().update(sql, money,out);
	}
 
	public void inMoney(String in, Double money) {
		String sql = "update account set money = money + ? where name= ?";
		this.getJdbcTemplate().update(sql, money,in);
	}
}
 
public interface AccountService {
	public void transfer(String out,String in,Double money);
}
 
public class AccountServiceImpl implements AccountService{
 
	public void setTransactionTemplate(TransactionTemplatetransactionTemplate) {
		this.transactionTemplate = transactionTemplate;
	}
	public void setAccountDao(AccountDao accountDao) {
		this.accountDao = accountDao;
	}
 
	// 注入转账Dao的类
	private AccountDao accountDao;
	//注入事务管理的模版
	private TransactionTemplate transactionTemplate;
	@Override
	public void transfer(final String out,final String in,finalDouble money) {
		transactionTemplate.execute(newTransactionCallbackWithoutResult() {
			@Override
			protected void doInTransactionWithoutResult(TransactionStatustransactionStatus) {
				accountDao.outMoney(out, money);
				//int i = 1/0;
				accountDao.inMoney(in, money);
			}
		});
	}
}

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class SpringDemol1 {
	@Resource(name="accountService")
	private AccountService accountService;
	@Test
	public void demol(){
		accountService.transfer("aaa", "bbb", 200d);
	}
}
```

## 1.2 声明式事务

### 基于.xml文件的声明式事务配置

```xml
<aop:aspectj-autoproxy proxy-target-class="true"/>
<!--
 1.数据源：实际上就是包含了Connection对象，不管是哪个厂商都需要实现DataSource接口
 就可以拿到Connection对象
 2.使用Spring给我们提供的工具类，TransactionManager事务管理器来管理所有的事务操作（肯定要拿到连接对象）
 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
   <property name="dataSource" ref="dataSource"/>
</bean>

<tx:annotation-driven transaction-manager="transactionManager"/>

<!-- 3.利用切面编程来实现对某一类方法进行事务管理（声明式事务） -->
<aop:config>
   <aop:pointcut expression="execution(public * com.spring..*.service..*Service.*(..))" id="transactionPointcut"/>
      <aop:advisor pointcut-ref="transactionPointcut" advice-ref="transactionAdvice"/>
 </aop:config>

<!-- 4、配置通知规则 -->
<!-- Transaction  tx :NameSpace -->
<tx:advice id="transactionAdvice" transaction-manager="transactionManager">
    <tx:attributes>
      <tx:method name="add*" propagation="REQUIRED" rollback-for="Exception,RuntimeException" timeout="-1"/>
      <tx:method name="remove*" propagation="REQUIRED" rollback-for="Exception,RuntimeException"/>
      <tx:method name="modify*" propagation="REQUIRED" rollback-for="Exception,RuntimeException"/>
      <tx:method name="transfer*" propagation="REQUIRED" rollback-for="Exception"/>
      <tx:method name="login" propagation="MANDATORY"/>
      <tx:method name="query*" read-only="true"/>
    </tx:attributes>
</tx:advice>
```

`Spring 事务管理基于 AOP 来实现`，主要是统一封装非功能性需求。

### 基于注解的声明式事务配置

@Transactional 可以作用于接口、接口方法、类以及类方法上。当作用于类上时，该类的所有 public 方法将都具有该类型的事务属性，同时，我们也可以在方法级别使用该注解来覆盖类级别的定义。

虽然 @Transactional 注解可以作用于接口、接口方法、类以及类方法上，但是 Spring 建议不要在接口或者接口方法上使用该注解，因为这只有在使用基于接口的代理时它才会生效。另外， @Transactional 注解应该只被应用到 public 方法上，这是由 Spring AOP 的本质决定的。如果你在 protected、private 或者默认可见性的方法上使用 @Transactional 注解，这将被忽略，也不会抛出任何异常。

# 2 事务原理详解

## 2.1 什么是事务（Transaction）

事务（Transaction）是访问并可能更新数据库中各种数据项的一个程序执行单元（unit）。 

特点：``事务是恢复和并发控制的基本单元``。 

事务应具备4个属性：原子性，一致性，隔离性，持久性。这四种属性通常称为ACID特性：

- `原子性（atomicity）` 一个事务是一个不可分割的工作单位，事务中包括的操作要么都做，要么都不做 
- `一致性（consistency）` 事务必须是使数据库从一个一致性状态变到另一个一致性状态 
- `隔离性（isolation）` 一个事务的执行不能被其他事务干扰。即一个事务内部的操作及使用的数据对并发的其他事务是隔离的，并发执行的各个事务之间不能相互干扰 
- `持久性（durability）` 持久性也成永久性，指一个事务一旦提交，他对数据库的改变就应该是永久性的。接下来的其他操作或故障不应该对其有任何影响

>数据库操作原理：
>
>插入
>
> 1、先把要插入的数据放入临时表
>
> 2、将临时表中数据插入实际表中去
>
> 3、如果没问题，就复制一份到实际表中，并将临时表中的数据删除
>
> 4、如果有问题，返回错误信息，临时表清空
>
> 删除，修改
>
> 1、先根据条件从原始表中查询来满足条件的数据行
>
> 2、将这些数据行复制一份到临时表
>
> 3、执行删除，如果出现错误，原来的数据原封不动，清空临时表中满足本次条件的记录，返回
>
> 4、如果执行成功，真正的干掉原始表中的记录。返回影响行数

## 2.2 事务的基本原理


Spring事务的本质其实就是数据库对事务的支持，没有数据库的事务支持，spring是无法提供事务功能的。对于纯JDBC操作数据库，想要用到事务，可以按照以下步骤进行：

1. 获取连接 Connection con = DriverManager.getConnection() 
2. 开启事务con.setAutoCommit(true/false) 
3. 执行 CRUD 
4. 提交事务/回滚事务con.commit()/con.rollback() 
5. 关闭连接conn.close

使用Spring的事务管理功能后，可以不再写步骤2和4的代码，而是由Spring自动完成。那么Spring是如何在CRUD之前和之后开启事务和关闭事务的呢？解决这个问题，也就可以从整体上理解Spring的事务管理实现原理来。注解方式的例子：

1. 配置文件开始注解驱动，在相关的类和方法上通过注解@Transactional标识 
2. spring在启动的时候会去解析生成相关的bean，这时候会查看拥有注解的类和方法，并且为这些类和方法生成代理，并根据@Transactional的相关参数进行相关配置注入，这样就在代理中为我们把相关的事务处理掉了（开启正常提交事务，异常回滚事务） 
3. 真正的数据库层的事务提交和回滚是通过binlog或者redo log实现的

## 2.3 事务管理器

Spring为不同的持久层框架提供了不同PlatformTransactionManager接口实现：

| 事务                                                         | 说明                                                        |
| ------------------------------------------------------------ | ----------------------------------------------------------- |
| org.springframework.jdbc.datasource.DataSourceTransactionManager | 使用Spring JDBC或iBatis 进行持久化数据时使用                |
| org.springframework.orm.hibernate3.HibernateTransactionManager | 使用Hibernate3.0 版本进行持久化数据时使用                   |
| org.springframework.orm.jpa.JpaTransactionManager            | 使用JPA进行持久化时使用                                     |
| org.springframework.orm.jdo.JdoTransactionManager            | 当持久化机制是Jdo时使用                                     |
| org.springframework.transaction.jta.JtaTransactionManager    | 使用一个JTA实现来管理事务，在一个事务跨越多个资源时必须使用 |

## 2.4 Spring 事务的传播属性

所谓Spring事务的传播属性，就是定义在存在多个事务同时存在的时候，Spring应该如何处理这些事务的行为。这些属性在``TransactionDefinition``中定义，具体常量的解释见下表：

| 常量名称      | 常量解释                                                     |
| ------------- | ------------------------------------------------------------ |
| REQUIRED      | 如果有事务, 那么加入事务, 没有的话新建一个(默认情况下)       |
| REQUIRES_NEW  | 不管是否存在事务,都创建一个新的事务,原来的挂起,新的执行完毕,继续执行老的事务 |
| SUPPORTS      | 如果其他bean调用这个方法,在其他bean中声明事务,那就用事务.如果其他bean没有声明事务,那就不用事务 |
| NOT_SUPPORTED | 容器不为这个方法开启事务                                     |
| MANDATORY     | 必须在一个已有的事务中执行,否则抛出异常                      |
| NEVER         | 必须在一个没有的事务中执行,否则抛出异常                      |
| NESTED        | 如果一个活动的事务存在，则运行在一个潜逃的事务中如果没有活动事务，则按REQUIRED属性执行。它使用了一个单独的事务，这个事务拥有多个可以回滚的保存点。内部事务的回滚不会对外部事务造成影响。它只对Dat aSourceTransactionManager事务管理器起效。 |

## 2.5 Spring 中的隔离级别

| 常量名称         | 常量解释                                                     |
| ---------------- | ------------------------------------------------------------ |
| DEFAULT          | 这是个PlatfromTransactionManager默认的隔离级别，使用数据库默认的事务隔离级别。另外四个与JDBC的隔离界别相对应 |
| READ_UNCOMMITTED | 这是事务最低的隔离级别，它允许另一个事务剋看到这个事务未提交的数据，这个隔离级别会产生脏读，不可重复读和幻读 |
| READ_COMMITED    | 保证一个事务修改的数据提交后才能被另一个事务读取。另一个事务不能读取该事务未提交的数据 |
| REPEARABLE_READ  | 这种事务隔离级可以防止脏读，不可重复读，但是可能出现幻读     |
| SERIALIZABLE     | 这是话费最高代价但是最可靠的事务隔离级别。事务被处理为顺序执行 |

## 2.6 事务的嵌套

通过上面的理论知识的铺垫，我们大致知道了数据库事务和spring事务的一些属性和特点，接下来我们通过分析一些嵌套事务的场景，来深入理解spring事务传播的机制。

假设外层事务 Service A 的 Method A() 调用 内层Service B 的 Method B()

**PROPAGATION_REQUIRED(spring 默认) **

如果ServiceB.methodB() 的事务级别定义为 PROPAGATION_REQUIRED，那么执行 ServiceA.methodA() 的时候spring已经起了事务，这时调用 ServiceB.methodB()，ServiceB.methodB() 看到自己已经运行在 ServiceA.methodA() 的事务内部，就不再起新的事务。

假如 ServiceB.methodB() 运行的时候发现自己没有在事务中，他就会为自己分配一个事务。

这样，在 ServiceA.methodA() 或者在 ServiceB.methodB() 内的任何地方出现异常，事务都会被回滚。 

**PROPAGATION_REQUIRES_NEW**

比如我们设计 ServiceA.methodA() 的事务级别为 PROPAGATION_REQUIRED，ServiceB.methodB() 的事务级别为 PROPAGATION_REQUIRES_NEW。

那么当执行到 ServiceB.methodB() 的时候，ServiceA.methodA() 所在的事务就会挂起，ServiceB.methodB() 会起一个新的事务，等待 ServiceB.methodB() 的事务完成以后，它才继续执行。

他与 PROPAGATION_REQUIRED 的事务区别在于事务的回滚程度了。因为 ServiceB.methodB() 是新起一个事务，那么就是存在两个不同的事务。如果 ServiceB.methodB() 已经提交，那么 ServiceA.methodA() 失败回滚，ServiceB.methodB() 是不会回滚的。如果 ServiceB.methodB() 失败回滚，如果他抛出的异常被 ServiceA.methodA() 捕获，ServiceA.methodA() 事务仍然可能提交(主要看B抛出的异常是不是A会回滚的异常)。

**PROPAGATION_SUPPORTS**

假设ServiceB.methodB() 的事务级别为 PROPAGATION_SUPPORTS，那么当执行到ServiceB.methodB()时，如果发现ServiceA.methodA()已经开启了一个事务，则加入当前的事务，如果发现ServiceA.methodA()没有开启事务，则自己也不开启事务。这种时候，内部方法的事务性完全依赖于最外层的事务。

**PROPAGATION_NESTED**

现在的情况就变得比较复杂了, ServiceB.methodB() 的事务属性被配置为 PROPAGATION_NESTED, 此时两者之间又将如何协作呢? ServiceB#methodB 如果 rollback, 那么内部事务(即 ServiceB#methodB) 将回滚到它执行前的 SavePoint 而外部事务(即 ServiceA#methodA) 可以有以下两种处理方式:

```java
void methodA() { 
    try { 
        ServiceB.methodB(); 
    } catch (SomeException) { 
        // 执行其他业务, 如 ServiceC.methodC(); 
    } 
}
```

这种方式也是嵌套事务最有价值的地方, 它起到了分支执行的效果, 如果 ServiceB.methodB 失败, 那么执行 ServiceC.methodC(), 而 ServiceB.methodB 已经回滚到它执行之前的 SavePoint, 所以不会产生脏数据(相当于此方法从未执行过), 这种特性可以用在某些特殊的业务中, 而 PROPAGATION_REQUIRED 和 PROPAGATION_REQUIRES_NEW 都没有办法做到这一点。 

外部事务回滚/提交 代码不做任何修改, 那么如果内部事务(ServiceB#methodB) rollback, 那么首先 ServiceB.methodB 回滚到它执行之前的 SavePoint(在任何情况下都会如此), 外部事务(即 ServiceA#methodA) 将根据具体的配置决定自己是 commit 还是 rollback。

另外三种事务传播属性基本用不到，在此不做分析。

## 2.7 Spring 事务 API 架构图

![img](Spring 事务原理详解.assets/20200401132248973.png)

# 3 Spring 事务源码分析

先看一看Spring事务是如何配置的，找到程序入口，通常情况下都是使用 xml 来配置Spring的声明式事务，先来分析`<tx:advice>…</tx:advice>`，tx是TransactionNameSpace，对应的是Hanler是TxNamespaceHandler（约定大于配置，Spring的默认套路），这个类一个init方法：

```java
public void init() {
   registerBeanDefinitionParser("advice", new TxAdviceBeanDefinitionParser());
   registerBeanDefinitionParser("annotation-driven", new AnnotationDrivenBeanDefinitionParser());
   registerBeanDefinitionParser("jta-transaction-manager", new JtaTransactionManagerBeanDefinitionParser());
}
```

这个方法是在 `DefaultNamespaceHandlerResolver.resolve()` 中调用的。在为对应的标签寻找 namespacehandler 的时候，调用这个 resolve() 方法。resolve() 方法先寻找 namespaceUri 对应的 namespacehandler，如果找到了就先调用init方法。 `<tx:advice>`对应的解析器也注册了，那就是上面代码里面的。现在这个 parser 的 parse() 方法在 `NamespaceHandlerSupport` 的parse方法中被调用了。下面来看看 TxAdviceBeanDefinitionParser 的 parse() 方法，这方法在 TxAdviceBeanDefinitionParser 的祖父类AbstractBeanDefinitionParser中：

```java
public final BeanDefinition parse(Element element, ParserContext parserContext) {
   // 关键方法
   AbstractBeanDefinition definition = parseInternal(element, parserContext);
   if (definition != null && !parserContext.isNested()) {
      try {
         String id = resolveId(element, definition, parserContext);
         if (!StringUtils.hasText(id)) {
            parserContext.getReaderContext().error(
                  "Id is required for element '" + parserContext.getDelegate().getLocalName(element)
                        + "' when used as a top-level tag", element);
         }
         String[] aliases = new String[0];
         String name = element.getAttribute(NAME_ATTRIBUTE);
         if (StringUtils.hasLength(name)) {
            aliases = StringUtils.trimArrayElements(StringUtils.commaDelimitedListToStringArray(name));
         }
         BeanDefinitionHolder holder = new BeanDefinitionHolder(definition, id, aliases);
         registerBeanDefinition(holder, parserContext.getRegistry());
         if (shouldFireEvents()) {
            BeanComponentDefinition componentDefinition = new BeanComponentDefinition(holder);
            postProcessComponentDefinition(componentDefinition);
            parserContext.registerComponent(componentDefinition);
         }
      }
      catch (BeanDefinitionStoreException ex) {
         parserContext.getReaderContext().error(ex.getMessage(), element);
         return null;
      }
   }
   return definition;
}
```

注意 parseInternal() 方法是在 TxAdviceBeanDefinitionParser 的父类 AdstractSingleBeanDefinitionParser 中实现的，代码如下：

```java
protected final AbstractBeanDefinition parseInternal(Element element, ParserContext parserContext) {
   BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition();
   String parentName = getParentName(element);
   if (parentName != null) {
      builder.getRawBeanDefinition().setParentName(parentName);
   }
   // 获取被代理对象
   Class<?> beanClass = getBeanClass(element);
   if (beanClass != null) {
      builder.getRawBeanDefinition().setBeanClass(beanClass);
   }
   else {
      String beanClassName = getBeanClassName(element);
      if (beanClassName != null) {
         builder.getRawBeanDefinition().setBeanClassName(beanClassName);
      }
   }
   builder.getRawBeanDefinition().setSource(parserContext.extractSource(element));
   if (parserContext.isNested()) {
      // Inner bean definition must receive same scope as containing bean.
      builder.setScope(parserContext.getContainingBeanDefinition().getScope());
   }
   if (parserContext.isDefaultLazyInit()) {
      // Default-lazy-init applies to custom bean definitions as well.
      builder.setLazyInit(true);
   }
   doParse(element, parserContext, builder);
   return builder.getBeanDefinition();
}
```

getBeanClass() 方法是在 TxAdviceBeanDefinitionParser 中实现的，很简单：

```java
protected Class<?> getBeanClass(Element element) {
   return TransactionInterceptor.class;
}
```

至此，这个标签解析的流程已经基本清晰了。那就是：解析出了一个以 `TransactionInterceptor` 为classname的beandefinition并且注册这个bean。剩下来要看的，就是这个TranscationInterceptor到底是什么？ 看看这个类的接口定义，就明白了：

```java
public class TransactionInterceptor extends TransactionAspectSupport implements MethodInterceptor, Serializable
```

这根本就是一个Spring AOP的 advice 嘛！现在明白为什么事务的配置能通过aop产生作用了吧？

```java
public Object invoke(final MethodInvocation invocation) throws Throwable {
   // Work out the target class: may be {@code null}.
   // The TransactionAttributeSource should be passed the target class
   // as well as the method, which may be from an interface.
   Class<?> targetClass = (invocation.getThis() != null ? AopUtils.getTargetClass(invocation.getThis()) : null);

   // Adapt to TransactionAspectSupport's invokeWithinTransaction...
   return invokeWithinTransaction(invocation.getMethod(), targetClass, new InvocationCallback() {
      public Object proceedWithInvocation() throws Throwable {
         return invocation.proceed();
      }
   });
}
```

接下来看一看 DataSourceTransactionManager 是如何工作的，其父类 AbstractPlatformTransactionManager 的 getTransaction 方法：

```java
public final TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException {
   Object transaction = doGetTransaction();

   // Cache debug flag to avoid repeated checks.
   boolean debugEnabled = logger.isDebugEnabled();

   if (definition == null) {
      // Use defaults if no transaction definition given.
      definition = new DefaultTransactionDefinition();
   }

   if (isExistingTransaction(transaction)) {
      // Existing transaction found -> check propagation behavior to find out how to behave.
      return handleExistingTransaction(definition, transaction, debugEnabled);
   }

   // Check definition settings for new transaction.
   if (definition.getTimeout() < TransactionDefinition.TIMEOUT_DEFAULT) {
      throw new InvalidTimeoutException("Invalid transaction timeout", definition.getTimeout());
   }

   // No existing transaction found -> check propagation behavior to find out how to proceed.
   if (definition.getPropagationBehavior() == TransactionDefinition.PROPAGATION_MANDATORY) {
      throw new IllegalTransactionStateException(
            "No existing transaction found for transaction marked with propagation 'mandatory'");
   }
   else if (definition.getPropagationBehavior() == TransactionDefinition.PROPAGATION_REQUIRED ||
         definition.getPropagationBehavior() == TransactionDefinition.PROPAGATION_REQUIRES_NEW ||
      definition.getPropagationBehavior() == TransactionDefinition.PROPAGATION_NESTED) {
      SuspendedResourcesHolder suspendedResources = suspend(null);
      if (debugEnabled) {
         logger.debug("Creating new transaction with name [" + definition.getName() + "]: " + definition);
      }
      try {
         boolean newSynchronization = (getTransactionSynchronization() != SYNCHRONIZATION_NEVER);
         DefaultTransactionStatus status = newTransactionStatus(
               definition, transaction, true, newSynchronization, debugEnabled, suspendedResources);
         doBegin(transaction, definition);
         prepareSynchronization(status, definition);
         return status;
      }
      catch (RuntimeException ex) {
         resume(null, suspendedResources);
         throw ex;
      }
      catch (Error err) {
         resume(null, suspendedResources);
         throw err;
      }
   }
   else {
      // Create "empty" transaction: no actual transaction, but potentially synchronization.
      boolean newSynchronization = (getTransactionSynchronization() == SYNCHRONIZATION_ALWAYS);
      return prepareTransactionStatus(definition, null, true, newSynchronization, debugEnabled, null);
   }
}
```

doGetTransaction()为DataSourceTransactionManager中，从ThreadLocal中获取一个connection(相互独立的)

```java
protected Object doGetTransaction() {
   DataSourceTransactionObject txObject = new DataSourceTransactionObject();
   txObject.setSavepointAllowed(isNestedTransactionAllowed());
   ConnectionHolder conHolder =
      (ConnectionHolder) TransactionSynchronizationManager.getResource(this.dataSource);
   txObject.setConnectionHolder(conHolder, false);
   return txObject;
}
```

```java
public abstract class TransactionSynchronizationManager {
	private static final ThreadLocal<Map<Object, Object>> resources =
      new NamedThreadLocal<Map<Object, Object>>("Transactional resources");
    ...
	private static Object doGetResource(Object actualKey) {
   		Map<Object, Object> map = resources.get();
    	if (map == null) {
      		return null;
   		}
   		Object value = map.get(actualKey);
   		// Transparently remove ResourceHolder that was marked as void...
   		if (value instanceof ResourceHolder && ((ResourceHolder) value).isVoid()) {
      		map.remove(actualKey);
      		// Remove entire ThreadLocal if empty...
      		if (map.isEmpty()) {
         		resources.remove();
     		}
      		value = null;
   		}
   		return value;
	}
	...
}
```

AbstractPlatformTransactionManager 的 getTransaction 方法调用 doBegin() 开启事务

```java
protected void doBegin(Object transaction, TransactionDefinition definition) {
   DataSourceTransactionObject txObject = (DataSourceTransactionObject) transaction;
   Connection con = null;

   try {
      if (txObject.getConnectionHolder() == null ||
            txObject.getConnectionHolder().isSynchronizedWithTransaction()) {
         Connection newCon = this.dataSource.getConnection();
         if (logger.isDebugEnabled()) {
            logger.debug("Acquired Connection [" + newCon + "] for JDBC transaction");
         }
         txObject.setConnectionHolder(new ConnectionHolder(newCon), true);
      }

      txObject.getConnectionHolder().setSynchronizedWithTransaction(true);
      con = txObject.getConnectionHolder().getConnection();

      Integer previousIsolationLevel = DataSourceUtils.prepareConnectionForTransaction(con, definition);
      txObject.setPreviousIsolationLevel(previousIsolationLevel);

      // Switch to manual commit if necessary. This is very expensive in some JDBC drivers,
      // so we don't want to do it unnecessarily (for example if we've explicitly
      // configured the connection pool to set it already).
      // 判断是否是自动提交，如果不是设置为手动提交
      if (con.getAutoCommit()) {
         txObject.setMustRestoreAutoCommit(true);
         if (logger.isDebugEnabled()) {
            logger.debug("Switching JDBC Connection [" + con + "] to manual commit");
         }
         con.setAutoCommit(false);
      }
      txObject.getConnectionHolder().setTransactionActive(true);

      int timeout = determineTimeout(definition);
      if (timeout != TransactionDefinition.TIMEOUT_DEFAULT) {
         txObject.getConnectionHolder().setTimeoutInSeconds(timeout);
      }

      // Bind the session holder to the thread.
      if (txObject.isNewConnectionHolder()) {
         TransactionSynchronizationManager.bindResource(getDataSource(), txObject.getConnectionHolder());
      }
   }

   catch (Throwable ex) {
      DataSourceUtils.releaseConnection(con, this.dataSource);
      throw new CannotCreateTransactionException("Could not open JDBC Connection for transaction", ex);
   }
}
```

执行业务逻辑 根据业务逻辑的执行结果来判断是否要提交还是要回滚（实质上都是操作JDBC的Connection） doCommit() 提交 doRollback() 回滚

```java
protected void doCommit(DefaultTransactionStatus status) {
   DataSourceTransactionObject txObject = (DataSourceTransactionObject) status.getTransaction();
   Connection con = txObject.getConnectionHolder().getConnection();
   if (status.isDebug()) {
      logger.debug("Committing JDBC transaction on Connection [" + con + "]");
   }
   try {
      con.commit();
   }
   catch (SQLException ex) {
      throw new TransactionSystemException("Could not commit JDBC transaction", ex);
   }
}

protected void doRollback(DefaultTransactionStatus status) {
   DataSourceTransactionObject txObject = (DataSourceTransactionObject) status.getTransaction();
   Connection con = txObject.getConnectionHolder().getConnection();
   if (status.isDebug()) {
      logger.debug("Rolling back JDBC transaction on Connection [" + con + "]");
   }
   try {
      con.rollback();
   }
   catch (SQLException ex) {
      throw new TransactionSystemException("Could not roll back JDBC transaction", ex);
   }
}
```

------

