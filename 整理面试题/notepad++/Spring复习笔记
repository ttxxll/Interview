BeanFactory工厂通过反射来实例化对象进行解耦：多例

用IOC容器实例化对象 -> 默认是单例 ：有线程安全问题：因为每次操作的都是一个对象，所以属性会变。

但是在service和dao层中一般不会有成员属性和成员方法，所以可以忽略单例的线程安全问题。所以我们还是选择单例来创建对象。

一般如果想要单例实例化对象可以将对象放到一个类似于Map的容器中；工厂一般都是通过new多例实例化。

Spring IOC

	IOC：	控制反转，将对象的创建、销毁、初始化等一系列的生命周期的过程交给spring容器来处理。
	
			所有的类都会在Spring容器中登录，告诉Spring自己是什么，需要哪些对象。然后Spring会在系统
			运行到适当的时候，把你需要的东西主动给你，同时也把你交给其他需要你的对象。
			
			所有类的创建，销毁等生命周期过程都由Spring来控制。因为以前是控制对象的生存周期是由
			引用他的对象来控制，而现在全部由Spring来控制，所以叫做控制反转。
	
	注入DI：在系统运行时，向某个对象提供他所需要的其他对象。
	
			类似业务层和持久层的依赖关系，在使用spring之后，就让spring来维护了，依赖关系的维护，就称为依赖注入。
	
			通过反射实现，允许程序在运行的时候动态的生成对象，执行对象的方法，改变对象的属性，
			Spring就是通过反射来实现注入的。
			
	
Spring的两个核心容器：	
		1.	ApplicationContext：单例模式适用
		
            它在创建核心容器是，创建对象采用的策略是立即加载方式。
			也就是说，只要一读取配置文件bean.xml，马上就创建配置文件中的配置的对象
			
		2.	BeanFactory：多例模式适用
			
			BeanFactory是一个顶层的接口，显然功能不是那么完善。
			
            它在构建核心容器时，创建对象采取的是延迟加载的方式。
			也就是说，什么时候根据id获取对象了，什么时候才真正的创建对象。
			
			1.	简单工厂
				<bean id="instanceFactory" class="com.itheima.factory.InstanceFactory"></bean>
				<bean id="accountService" factory-bean="instanceFactory" factory-method="getAccountService"></bean>

			
			2.	静态工厂：不需要实例化工厂对象
				<bean id="accountService" class="com.itheima.factory.StaticFactory"
				factory-method="getAccountService"></bean>
				
bean对象的生命周期
 
		1. 单例对象
			出生：当容器创建时，对象出生。
			活着：只要容器还在，对象一直活着
			死亡：容器销毁，对象消亡
			
		2.	多例对象
            出生：	当使用对象时，spring框架才为我们创建对象
            活着：	对象在使用过程中一直活着。
            死亡：	spring虽然很强大，但是它仍然无法知道我们这个对象要用到什么时候，什么时候销毁。
					所以它不会轻易的将这个对象销毁的，当对象长时间不用且没有别的对象引用时，由java的垃圾回收器回收
				  
				  
Spring的常用注解：
		
		这是创建bean
		@Component: 一般不属于三层的会用这个

        作用：	反射创建一个当前类对象，并把当前类对象存入spring容器中
        属性：	value：用于指定bean的id。默认值是当前类名且首字母小写，
                如果连着首字母2个或多个大写那么id是当前类名。

		
		这三个是注入bean
		@Autowired：
		作用：	自动按照类型注入。
				只要容器中有唯一的一个bean对象类型和要注入的变量类型匹配就可以注入成功。
				注意这里是唯一。如果没有匹配的就会报错。

				如果有多个能匹配的bean对象类型：
				@Autowired
				private IAccountDao accountDao1， = null;
				
				@Repository("accountDao2")
				public class IAccountDaoImpl2 implements IAccountDao {
				}
				
				@Repository("accountDao1")
				public class IAccountDaoImpl implements IAccountDao {
				}
				
				就会让变量名accountDao1再去匹配key--->匹配到@Repository("accountDao1")。就可以成功注入了。
				
		小结：	先根据类型找，再根据变量名找IOC中的key。
		
		缺点：	无法指定Bean的id，当几个bean对象的在容器中的key也无法和变量名对应上，那么该注解就无法注入了。
		
		
		@Qualifier：
		作用：	在按照类型注入的基础上，再按照名称注入。它在给成员注入时不能单独使用(搭配@Autowired)。
		
				@Autowired
				@Qualifier("accountDao1")
				private IAccountDao accountDao1 = null;
				
		@Resource：
		作用：	直接按照bean的id注入。它可以独立使用。是J2EE的注解，可以减少和Spring的耦合，
				并且相较于@Autowired和@Qualifier结合使用，较简洁。
		
				@Resource(name="accountDao1")
				private IAccountDao accountDao1 = null;
				
		以上三个注入都只能注入其他bean类型的数据，而基本类型和String类型无法使用上述注解实现，
		另外集合类型的注入只能通过XML来实现。
    
##########################################################################################################
	1.
		@Value：
		作用：用于注入基本类型和String类型的数据。一般是配置文件中的属性。
		
	2.	
		@Configuration
		@ComponentScan(value={"com.itheima"})
		public class SpringConfiguration {
		}
		
		作用：代替xml文件，来告诉Spring在创建容器时要扫描的包。
		1.Configuration：标注这是一个配置类，
		2.ComponentScan：通过该注解指定spring在创建容器时要扫描的包，那么就能扫描到类上的注解（@Component,@Service...）
						 那么就能实例化Bean了。
		
		
	3.
		继续完善这个配置类：
		@Configuration
		@ComponentScan(value={"com.itheima"})
		public class SpringConfiguration {
			
			@Bean(name="queryRunner")
			@Scope(value="prototype")   //注意QueryRunner我们需要多例创建
			public QueryRunner creatQueryRunner(DataSource dataSource){
			return new QueryRunner(dataSource);
			}
			
		}
		
		@Bean：	标注在方法上，方法的将返回值放到容器中
				name:用于指定bean的id。默认值：当不指定时：当前方法的名称。
				
				那么当方法中有参数时：就要考虑参数来自哪里？
				
					参数也会去容器中匹配相应的bean对象，匹配方式同@AutoWerid的引用类型注入。
				
				那么如果参数去容器中匹配相应的bean对象时，id也匹配不上呢？
				
					@Qualifier("id")：它是可以标注在方法参数中
					

###############################################################################

Spring中的事务：

	1.	因为考虑到单例的线程安全问题，所有操作数据库的对象是多例创建的。
	
		那么每个每个connection都有一套自身独立的默认事务管理：setAutoCommit(true)
	
		如果业务方法一次需要执行多条sql语句，每次执行sql都会从线程池中拿一个新的connection，此时这种方式就会出现事务问题。
		
		ThreadLocal对象把Connection和当前线程绑定，从而使一个线程中只有一个能控制事务的对象。执行完方法后在解绑。
		
#######################################################################################

动态代理：

	1.	基于接口的动态代理：被代理的是实现类，代理对象和被代理对象有相同的接口。
	
		作用：不修改源码的基础上，对方法增强
		涉及的类：Proxy
		
		被代理类最少实现一个接口，如果没有则不能使用
	
	
	public class Client {

		public static void main(String[] args) {
			final Producer producer = new Producer();
		
				IProducer proxyProducer = (IProducer) Proxy.newProxyInstance(producer.getClass().getClassLoader(),
				producer.getClass().getInterfaces(),
				new InvocationHandler() {
					public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
						Object returnValue = null;
						//1.获取方法执行的参数
						Float money = (Float)args[0];
						//2.判断当前方法是不是销售
						if("saleProduct".equals(method.getName())){
							//注意一下：参数是money:float，这里money*0.8之后就不是float了，所以要money*0.8f
							//匿名内部类访问外部成员变量时，外部成员要求时最终的final修饰
							returnValue = method.invoke(producer, money*0.8f);
							}
		
							return returnValue;
						}
					});
		
			producer.saleProduct(10000f);
			//用代理对象调用一下被代理对象的方法
			proxyProducer.saleProduct(10000f);
		}
	}

	参数1.	类加载器，和被代理对象的类加载器是同一个，用于加载代理对象字节码。
	参数2.	字节码数组，被代理对象的接口的字节码数组。
			保证代理对象和被代理对象实现同一个接口，都有同样的方法，就可以增强响应的方法。
	参数3.	new InvocationHandler() {}
			里面写如何代理，具体的增强逻辑。通常情况下都是一个匿名内部类。
			
			匿名内部类实现了InvocationHandler接口，要重写invoke()方法：当代理对象执行被代理对象的任何接口和方法都会经过该方法
			
			其中的参数：
				
                     * @param proxy     代理对象的引用
                     * @param method    当前执行的方法
                     * @param args      当前方法所需的参数
					 
	
	2.	基于子类的动态代理：被代理的是父类，得到一个被代理对象的子类。
	
		涉及的类：Enhancer，使用Enhancer类中的create方法
		
		创建代理对象的要求：被代理类不能是最终类，如果是最终类，就不能在创建子类了，也就没法创建代理对象了
		
		
		public class Client {

			public static void main(String[] args) {
				final Producer producer = new Producer();

				Producer cglibProducer = (Producer)Enhancer.create(producer.getClass(), new MethodInterceptor() {
					public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
							Object returnValue = null;
							//1.获取方法执行的参数
							Float money = (Float)args[0];
							//2.判断当前方法是不是销售
							if("saleProduct".equals(method.getName())){
								returnValue = method.invoke(producer, money*0.8f);
							}
							return returnValue;
						}
					});
			
					cglibProducer.saleProduct(12000f);
			}
		}
		
		create()方法的参数：
        *  	Class : 字节码，它是用于指定被代理对象的字节码，想代理谁就写谁的.getClass()
			
        *  	Callback：用于提供增强的代码，它是让我们写如何代理。我们一般都是写一个该接口的实现类：
         
				通常情况下都是匿名内部类，但不是必须的。
				我们一般写的都是该接口的子接口实现类：MethodInterceptor（方法拦截）
				
				public interface MethodInterceptor extends Callback：
				
				其中重写了intercept方法，代理对象执行的所有被代理对象的方法都会在此处被拦截。
				参数1： proxy               代理对象的引用
				参数2： method              当前执行的方法
				参数3： args                当前方法所需的参数
				参数4： methodProxy         当前执行方法的代理对象，一般用不到
				返回值：                    和被代理对象方法中的放回值是一样的
				
				
###########################################################################


Spring AOP：

	它就是把我们程序重复的代码抽取出来，在需要执行的时候，使用动态代理的技术，在不修改源码的基础上，对我们已有的方法进行增强。
	
	作用：	在程序运行期间，不修改代码对已有方法进行增强。
	
	AOP的实现方式：	使用动态代理技术。
	
	我们学习spring的AOP，就是通过配置的方式，实现上一部分动态代理抽取事务的功能
	
	1.	一些术语：
		
		连接点Joinpoint：test方法是可以被增强的但是没有增强，所以是连接点，但不是切入点。
		
		切入点Pointout：就是那些被增强的方法
		
		通知Advice：	所谓通知是指拦截到的Joinpoint之后要做的事情（增强逻辑代码）就是通知。
						就是那些提供了公共代码的类。（为每个service中的方法提供事务支持的类。）
		
		    通知的类型：根据切入点的位置（和通知的相对顺序顺序）
				前置通知
				后置通知
				异常通知
				最终通知
				环绕通知
				
		切面Aspect：	是切入点和通知的结合。
			
			建立切入点方法和通知方法在执行调用的对应关系：这整个过程配出来就是切面。
		
			通知：就是那些提供了公共代码的类。（为每个service中的方法提供事务支持的类。）
			
			那这些公共代码什么时候执行呢？怎么定义这些通知和切点的相对顺序呢？
			
	2.	要明确：学习SpringAop我们用来做什么？
	
			编写核心业务代码（开发主线）：大部分程序员来做。
			
			将核心业务代码中的公共代码抽取出来，制作成通知。
			
			在配置文件中，声明切入点与通知间的关系，即切面。
			
	3.	Spring框架能为我们做哪些？
	
			Spring框架监控切入点方法的执行。
			
			一旦监控到切入点方法被运行，就会使用代理机制，动态创建目标对象的代理对象。
			根据通知类别（前，后，异常，最终...）在代理对象，切入点相应的位置将通知织入，组装成切面。
			
	4.	spring的事务控制都是基于AOP的，它既可以使用编程的方式实现，也可以使用配置的方式实现。
			
    重复的代码：
        先抽取，再在方法执行时通过动态代理给它加进去，不改变源码增强了原有的方法。
        
        最后才由Spring来实现AOP。
		
			
		
		


		
	
		
		
		










			