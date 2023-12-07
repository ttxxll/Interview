#### 1. 什么是SpringBoot

​		先从Spring大概说一下，Spring框架为开发Java应用程序提供了全面的基础架构支持。它包含了一些很友好的功能，如依赖注入和开箱即用模块。比如：SpringJDBC，SpringMVC，SpringAOP等，这些模块缩短了应用程序的开发时间，提高了应用开发的效率。

​		SpringBoot基本上是Spring框架的扩展，它消除了设置Spring应用程序所需的XML配置，为更快，更高效的开发生态系统铺平了道路。

​		比如SpringBoot只需要一个依赖就可以启动和运行Web应用程序。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId> spring-boot-starter-web</artifactId>
    <version>2.0.6.RELEASE</version>
</dependency>
```

​	优点：

​		能够快速独立的运行Spring项目以及主流框架集成。

​		简化部署：使用嵌入式的Servlet容器，应用无需打成WAR包。可以达成jar包，使用java-jar独立运行jar包。不需要配置tomcat等服务器，直接注入一个插件，可以将程序打成一个可执行的jar包

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```



​		大量的自动配置，简化开发，也可修改默认值。

​		无需配置XML，无代码生成，开箱即用。



#### 2.	分析一下SpringBoot的自动配置原理

1.先从pom.xml分析一下：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.1.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>  
```

​		spring-boot-starter-parent的父项目spring-boot-dependencies作为springboot的版本仲裁中心。

​		spring-boot-dependencies里面的<properties/>锁定了很多jar包的版本。所以父项目spring-boot-starter-parent的父项目spring-boot-dependencies是来真正管理springboot应用里面所有的依赖

​		spring-boot-dependencies又称版本仲裁中心，所以我们导入默认依赖是不需要写版本的。当然如果我们要导入不在spring-boot-dependencies里面管理的依赖就要自己声明版本号了。

2.这个简单的web项目运行所需的jar包是由谁导进来的呢？

​		在pom.xml文件中，除了导入了一个父项目，还有一个依赖：spring-boot-starter-web。这是一个springboot-web应用程序的场景启动器。这里面依赖了若干jar包，导入了web模块正常运行所依赖的组件，而且依赖的版本都受父项目控制。

​		点进去可以看到里面有tomcat和springmvc等依赖。

​		springboot将所有功能都抽取成了一个个场景，每个场景都对应着一个启动器starter。那么显然除了web还有很多适用其他场景的启动器，只需要在项目里面引这些starter，相关场景的所有依赖都会导入进来。依赖版本由springboot自动控制。要用什么功能就导入什么场景启动器。

3.分析一下主程序

```java
    @SpringBootApplication
    public class HelloWorldMainApplication {
        public static void main(String[] args) {
    
            //spring应用启动起来
            SpringApplication.run(HelloWorldMainApplication.class, args);
        }
    }
```

​		(1) 运行main方法，在SpringApplication.run()时，需要传入一个参数。这个参数是一定要是一个被@SpringBootApplication注解标注的类。

​			@SpringBootApplication这个注解标注在哪个类上就说明这个类就是SpringBoot的主配置类，SpringBoot就会运行这个类的main方法来启动SpringBoot应用。

​		(2)分析一下@SpringBootApplication注解：

​			它是一个组合注解，里面有两个比较重要的子注解：

​			1.@SpringBootConfiguration

​			2.@EnableAutoConfiguration

​		(3)子注解1：@SpringBootConfiguration

​			这个子注解也是个组合注解，里面有@Configuration和@Component

​			所以第一个子注解的功能，就是将标注了@SpringBootApplication的主入口类指定为一个配置类，并且注册到IOC容器中，所以可以将这个类称为主入口类或主配置类。

​		(4)子注解2：@EnableAutoConfiguration

​			这个子注解主要开启自动配置功能。就拿这个简单的web应用来说，我们整个springboot项目中没有进行任何配置，但是SpringMVC启动了，@Controller也扫进来了，整个应用也能用了，那这些功能是怎么做的呢？

​			就是靠这个注解@EnableAutoConfiguration，开启自动配置。也就是SpringBoot帮我们配置了场景需要的依赖，但是还要靠这个注解来开启自动配置功能，这样自动配置的东西才能生效。

4.自动配置原理：@EnableAutoConfiguration

​	(1)子注解1：@AutoConfigurationPackage

​		1.1自动配置包。

​		自动配置包@AutoConfigurationPackage里面还有一个@Import注解，@Import注解指定了Registrar.class。这个注解来完成@AutoConfigurationPackage自动配置包的功能。

​		@Import是Spring的底层注解：给容器中导入组件。由Registrar.class来指定具体的组件。

​		到这一步，可以理解，@AutoConfigurationPackage标注的类个给我们IOC容器中注册了一些组件。

​		1.2导入了哪些组件呢？

```	java
	static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {

		@Override
		public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
			register(registry, new PackageImports(metadata).getPackageNames().toArray(new String[0]));
		}

		@Override
		public Set<Object> determineImports(AnnotationMetadata metadata) {
			return Collections.singleton(new PackageImports(metadata));
		}

	}
```

​		点进Registrar类中可以看到一个静态方法registerBeanDefinitions()方法，参数是AnnotationMetadata，这是注解的原信息，通过注解原信息来拿到@SpringBootApplication注解和SpringBoot程序的主入口类。然后就可以将@SpringBootApplication标注的类所在包和下面的所有子包中的组件都扫描到IOC容器中。

​		这就是为什么我们没有配置文件仍然能将Controller扫描进去，因为它在主配置类所在包的子包中。

​	(2)子注解2：@Import({EnableAutoConfigurationImportSelector.class})

​		该注解的意思还是：给容器中导入一些组件。导入哪些组件由AutoConfigurationImportSelect.class类来指定。--- 开启自动配置类的导包的选择器

​		2.1 点进来看看想给我们导入哪些组件：

​				EnableAutoConfigurationImportSelect有一个父类：AutoConfigurationImportSelector

​		2.2 在这个父类AutoConfigurationImportSelector中有一个selectImports()方法

```java
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        if (!this.isEnabled(annotationMetadata)) {
            return NO_IMPORTS;
        } else {
            try {
                AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader.loadMetadata(this.beanClassLoader);
                AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
                List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
                configurations = this.removeDuplicates(configurations);
                configurations = this.sort(configurations, autoConfigurationMetadata);
                Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
                this.checkExcludedClasses(configurations, exclusions);
                configurations.removeAll(exclusions);
                configurations = this.filter(configurations, autoConfigurationMetadata);
                this.fireAutoConfigurationImportEvents(configurations, exclusions);
                return (String[])configurations.toArray(new String[configurations.size()]);
            } catch (IOException var6) {
                throw new IllegalStateException(var6);
            }
        }
    }
```

​				告诉我们导入哪些选择器组件，将所有需要导入的组件的选择器以全类名的方式返回，这些组件会以反		射的方式放到IOC容器中。

​		2.3 selectImports方法到底添加了哪些组件呢？

​				发现selectImports()方法和自动配置包注解中的registerBeanDefinitions()方法一样也传入了AnnotationMetadata，注解原信息参数。这个参数包含了@SpringBootApplication注解和其所标注的SpringBoot程序的主入口类相关信息。

​				1.首先通过getCandidateConfigurations()方法，参数是注解原信息annotationMetadata。得到一个List<String>，它的里面都是所有的候选的配置文件。

​				2.点进这个方法，里面有这么一个方法：SpringFactoriesLoader.loadFactoryNames()

```java
SpringFactoriesLoader.loadFactoryNames(
    this.getSpringFactoriesLoaderFactoryClass(), 
    this.getBeanClassLoader());
```

​				其中参数1：this.getSpringFactoriesLoaderFactoryClass()是EnableAutoConfiguration.class;

​				参数2：this.getBeanClassLoader()是一个类加载器。

​				进入这个loadFactoryNames()方法中

```java
    public static List<String> loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader) {
        String factoryClassName = factoryClass.getName();

        try {
            Enumeration<URL> urls = classLoader != null ? classLoader.getResources("META-INF/spring.factories") : ClassLoader.getSystemResources("META-INF/spring.factories");
            ArrayList result = new ArrayList();

            while(urls.hasMoreElements()) {
                URL url = (URL)urls.nextElement();
                Properties properties = PropertiesLoaderUtils.loadProperties(new UrlResource(url));
                String factoryClassNames = properties.getProperty(factoryClassName);
                result.addAll(Arrays.asList(StringUtils.commaDelimitedListToStringArray(factoryClassNames)));
            }

            return result;
        } catch (IOException var8) {
            throw new IllegalArgumentException("Unable to load [" + factoryClass.getName() + "] factories from location [" + "META-INF/spring.factories" + "]", var8);
        }
    }
```

​		其中有一句代码：

```
Enumeration<URL> urls = classLoader.getResources("META-INF/spring.factories")
```

​		会从类路径下加载一个资源：扫描所有jar包下类路径下的这个文件META-INF/spring.factories，得到这些文件的url，放到枚举中。

​		接着遍历枚举：将其中的url通过loadProperties加载成properties对象。意味着扫描到的这些文件的内容会被包装成properties对象。

```java
Properties properties = PropertiesLoaderUtils.loadProperties(new UrlResource(url));
```

​		再从每个propertie取出它的factoryClassName字符串，将其放入我们要返回的List<String>中，返回给getCandidateConfigurations()方法。

​		getCandidateConfigurations()再将包含了properties信息的List<String>返回给selectImports()方法

​		再通过AutoConfigurationImportSelector类中的方法，从properties中获取到EnableAutoConfiguration.class类名对应的值，然后将他们添加到容器中。

```
    protected Class<?> getSpringFactoriesLoaderFactoryClass() {
        return EnableAutoConfiguration.class;
    }
```



5.小结：

​	@SpringBootApplication的两个子注解：

1. @SpringBootConfiguration标注在某个类上，表示这是一个SpringBoot的配置类，并且将标注的类加载到IOC中。

指明主启动类是一个SpringBoot主配置类，而且这个类的对象要放到IOC中。



2. @EnableAutoConfiguration：其中有两个子注解

   1.@AutoConfigrationPackage：

 这个注解将主启动类所在的包下面的所有子包里面的组件都扫描到IOC中了。

​		2.@Import({EnableAutoConfigrationImportSelector.class})

 给容器中导入一些组建，导入哪些组件由EnableAutoConfigrationImportSelector.class来指定。会将类路径下的META-INF/spring.factories的配置进行导入，通过 SpringFactoriesLoader加载 到容器中。

 每一个指定的xxxAutoConfiguration类都是容器中的一个组件，都加入到容器中，用他们来做自动配置。

 只有这些自动配置类进到容器中，自动配置类注解@EnableAutoConfiguration才能起作用，实现自动配置的功能。

​	其实就是Springboot根据配置文件自动装配需要依赖的类，通过动态代理注入到Spring容器中。



#### 3.SpringBoot的大概启动过程



![1599225852147](images\springboot启动原理.png)

1. SpringFactoriesLoader加载META-INF/spring.factories文件，创建并获取SpringApplicationRunListener对象
2. 然后通过SpringApplicationRunListener来发出starting消息。
3. 创建参数，并配置当前SpringBoot应用将要使用的Environment
4. 完成之后，依然由SpringApplicationRunListener来发出enviromentPrepared消息
5. 创建SpringBoot应用上下文：ApplicationContext
6. 初始化Application，并设置Environment，加载相关配置等。
7. 由SpringApplicationRunListener来发出contextPrepared消息，告知SpringBoot应用使用的ApplicationContext已准备OK。
8. 将各种beans装载入ApplicationContext，继续由SpringApplicationRunListener来发出contextLoaded消息，告知SpringBoot应用使用的ApplicationContext已装填OK。
9. refresh ApplicationContext，完成IOC容器可用的最后一步。
10. 由SpringApplicationRunListener来发出started消息
11. 完成最终的程序的启动
12. 有SpringApplicationRunListener来发出running消息，告知SpringBoot程序已经运行起来了。



#### 4. SpringBoot的配置文件

​	1.SpringBoot使用一个全局的配置文件：application.properties/application.yml

​	配置文件放在：src/main.resources目录，或者类路径下/config目录，在这两个目录下的该文件SpringBoot都会把它当成全局配置文件。

​	2.配置文件的作用

​	由于SpringBoot在底层，在启动的时候给我们做了很多大量的自动配置。如果我们想要自己定制化一些内存，一些配置，就可以在全局配置文件中进行修改。

​	3.yml的基本语法

​		K: V表示一对键值对（必须有空格）

​		yml格式是以空格和缩进来控制层级关系的，只要是左对齐的一列数据表示是同一层级；

​		注意不要用tab键来缩进，缩进要使用空格来缩进。

#### 5. @ConfigurationProperties和@Value

​	1.@ConfigurationProperties(prefix = "")

​	只需要一个@ConfigurationProperties注解，可以为我们批量注入属性。比如我们专门写了一个JavaBean来和配置文件中的信息进行映射，我们就可以直接使用@ConfigurationProperties。

```y&#39;m
        person:
          lastName: zhangsan
          age: 18
          boss: false
          birth: 2019/12/12
          maps: {k1: v1, k2: 12}
          list:
            - lisi
            - zhaoliu
            - zhangsan
          dog:
            name: 小狗
            age: 2
```

```java
@Component
@ConfigurationProperties(prefix = "person")
public class Person {

    private String name;
    private Integer age;
    private Boolean boss;
    private Date birth;

    private Map<String,Object> maps;
    private List<Object> list;
    private Dog dog;

    public String getName() {
        return name;
    }
}
```

​	2.@Value注解为单个属性注入

```java
//从配置文件中取值
@Value="${key}"
private String lastName;

//写法：字面值
@Value("true")
private Boolean boss;

//写法：SpeL语言
@Value("#{11*2}")
private Integer age;
```

@ConfigurationProperties和@Value的区别

​	1.	@ConfigurationProperties可以批量注入属性

​		@Value只能为单个属性注入

​	2.	@ConfigurationProperties不支持SPEL表达式

​		@Value支持SPEL表达式	

​	3.	@Configuration支持JSR303数据校验

​		@Value不支持

```java
@Email
private String lastname;
```

​	4.	@Configuration支持复杂类型注入

​		@Value不支持

```java	
person.maps.k1=v1
person.maps.k2=v2

//会报错：不支持复杂类型注入
@Value("${person.maps}")
```

#### 6. @ConfigurationProperties结合@PropertySource读取指定配置文件

​	@ConfigurationProperties能将实体类中的属性和配置文件中的值进行绑定，但是绑定的前提是要求你的属性都写在了application.yml全局配置文件中。即@ConfigurationProperties默认是从全局配置文件中获取值的。

​	@PropertySource：配合注入的，加在要注入的类上指定要加载的配置文件，将配置文件中的值注入到属性中。

​	但是我们不会将所有的配置都放到一个配置文件中，那么一些和SpringBoot不相关的配置，我们就希望放到一个单独的配置文件中。

​	1.person.properties

```properties
            person.lastname=李四
            person.age=15
            person.birth=2019/12/01
            person.boss=false
            person.maps.k1=v1
            person.maps.k2=14
            person.list=a,b,c
            person.dog.name=dog
            person.dog.age=15
```

​	2.那么此时@ConfigurationProperties(prefix = "person")该注解从全局配置文件中去匹配前缀是person的值，显然取不到了。

​	3.结合@PropertySource(value = {"classpath:person.properties"})

​	告诉SpringBoot来加载类路径下的person.properties的内容，并把它绑定到JavaBean属性中。

```java
        @Component
        @PropertySource(value = {"classpath:person.properties"})
        @ConfigurationProperties(prefix = "person")
        public class Person {
            private String lastname;
            private Integer age;
            private Boolean boss;
            private Date birth;
        }
```

#### 7. @ImportResource

​	加载配置类上，指定要加载的其他配置文件的路径locations

​	我想向IOC里面注册一个JavaBean有两种方式：

一.方式1：不推荐

​	1.写一个spring的配置文件：bean.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--仅仅是写了配置文件，但是还没有加载-->
    <bean id="helloService" class="com.atguigu.day0625_springboot.service.HelloService"></bean>
</beans>

```

​	这个配置文件向IOC中注册了一个JavaBean ：helloService。

​	但是SpringBoot不会自动加载这个spring的配置文件

​	2.@ImportResource标注在一个配置类上

​	我们可以标注在主配置类上

```java
//加载beans.xml配置文件让其生效。
@ImportResource(locations = {"classpath:beans.xml"})
@SpringBootApplication
public class SpringBoot01ConfigApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBoot01ConfigApplication.class, args);
    }

}
```

​	不推荐这种方式，本来SpringBoot就是为了简化Spring繁杂的配置文件的，这里又写了一个配置文件。

二.方式2：@Configuration+@Bean

```java
@Configuration
public class MyConfig {
    //将方法的返回值添加到容器中：容器中这个组件默认的ID就是方法名
    @Bean
    public HelloService helloService(){

        System.out.println("配置类给容器添加组件了");
        return new HelloService();
    }
}
```

​	专门写一个配置类，将需要的JavaBean注册到IOC中。

​	这样就不用再指定加载添加组件的配置类了。

#### 8. 多Profile配置文件：进行环境切换

​	1.主配置文件application.yum

​	2.再配置两个不同环境的配置文件

​		a.application-dev.properties：开发环境的配置文件

​		b.application-prod.properties：生产环境的配置文件

​	SpringBoot会默认加载主配置文件，如何切换到其他环境，使用其他环境中的配置文件呢？

​		在主配置文件中进行如下配置：spring.profiles.active=dev

​		切换到开发环境，加载开发环境配置。



​	注意：

​	yml文件还有一种文档块的语法，可以进行开发环境的切换。

​	命令行方式来切换环境。



#### 9. @EnableConfigurationProperties(XxxProperties.class)和@ConfigurationProperties

```java
@Configuration(proxyBeanMethods = false)
@EnableConfigurationProperties(ServerProperties.class)
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)
@ConditionalOnClass(CharacterEncodingFilter.class)
@ConditionalOnProperty(prefix = "server.servlet.encoding", value = "enabled", matchIfMissing = true)
public class HttpEncodingAutoConfiguration {

	@Bean
	@ConditionalOnMissingBean
	public CharacterEncodingFilter characterEncodingFilter() {
		return filter;
	}
	@Bean
	public LocaleCharsetMappingsCustomizer localeCharsetMappingsCustomizer() {
		return new LocaleCharsetMappingsCustomizer(this.properties);
	}
}
```

1.@EnableConfigurationProperties注解的作用

​	让使用@ConfigurationProperties注解的类生效

```java
@ConfigurationProperties(prefix = "server", ignoreUnknownFields = true)
public class ServerProperties {

    private Integer port;
    private InetAddress address;
    ...
}
```

​	我来分析：

​	1.1@EnableConfigurationProperties会搭配着@Configuration和@Bean使用，一般用在自动配置类上。对了，一般还会搭配@ConditionalOnXxxx的一起使用，这个自动配置类根据若干个@ConditionalOnXxxx的注解来判断这个自动配置类是否生效。

​	1.2如果@ConditionalOnXxx条件成立，这个自动配置类生效，那么@Configuration和@Bean这个两个注解就会起作用，指定当前这个类是一个配置类，并且向IOC中注册一些组件。将@Bean标注的方法的返回值，注册到IOC中，也就是当前这个自动配置类生效了。

​	1.3继续分析：自动配置类通过@Bean注册到IOC的组件经常会和@EnableConfigurationProperties(XxxxProperties.class)中的指定的XxxxProperties有很大的联系。一般都是从这个Properties对象中获取到相关的信息，相关的配置信息。通过@Bean注册到IOC的组件一般都包含这个Properties对象中包含的配置信息。

​	1.4那么是怎么从这个Properties对象中获取到里面的信息呢？这个Properties对象是从哪里来的呢？此时@EnableConfigurationProperties(XxxProperties.class)这个注解就起作用了。这个注解里面有一个子注解@Import({EnableConfigurationPropertiesRegister.class})，它的作用就是将XxxProperties注册到IOC里面，那么自动配置类HttpEncodingAutoConfiguration就可以拿到这个XxxxProperties对象，自动配置类再向IOC注册一些包含XxxProperties对象中的信息的组件，这些组件一般都是实现自动配置功能的。

​	1.5为什么这个XxxProperties中有这些自动配置相关的信息呢？一般这个XxxProperties会被@ConfigurationProperties(prefix = "")注解标注。那么这个类就和配置文件中prefix指定的前缀内容绑定，绑定到Properties对象里面的属性上面。

​	1.6但是这个@ConfigurationProperties要起作用，这个XxxProperties对象必须要被扫描到，注册到IOC中。注册到IOC中之后，Spring容器会自动的使该类上的@ConfigurationProperties注解生效，将相关的属性和配置信息绑定，注册进IOC中。

​	1.7实现将@ConfigurationProperties标注的类注册到IOC中，这一步的方法一般有两个。比如@Conponent主角，或者配置扫描该类所在的包。如果该类是使用了@ConfigurationProperties注解，并且该类没有在扫描路径下或者没有@Componet注解，导致无法被扫描为Bean。那么就必须在一个配置类上使用@EnableConfigurationProperties注解去指定这个类，将这个类扫描并注册到IOC中，同时使@ConfigurationProperties注解生效，绑定相关的配置信息。

​	1.7XxxProperties类一般都通过@ConfigurationProperties(prefix="")用来封装配置文件中的相关配置信息，而且这些prefix前缀的配置内容一般都会有默认值，如果这些默认的配置信息满足了我的需求，就是用默认的配置信息。如果不满足，我可以在application.yml中配置相应的信息，覆盖掉默认信息。



#### 11.SpringBoot配置过滤器，监听器，拦截器

首先实现相应的接口定义，然后重写方法，实现你要实现的逻辑，通过配置类将其加入到spring容器中，实现我们自定义的过滤器，监听器，拦截器。

1.过滤器：impements Filter  

重写init()，doFilter()，destroy()方法。在doFilter()中写你要实现的过滤器逻辑。

2.监听器类：implements HttpSessionListener/ServletRequestListener等等

监听器有监听ServletContext对象的创建以及销毁，监听Session对象的创建以及销毁，监听Request对象的创建以及销毁....

2.1监听Session对象的创建以及销毁：implements  HttpSessionListener

重写 sessionCreated(HttpSessionEvent se)， sessionDestroyed(HttpSessionEvent se)

2.2监听Request对象的创建以及销毁：implements  ServletRequestListener

重写 requestInitialized(ServletRequestEvent sre)， requestDestroyed(ServletRequestEvent src)

3.拦截器类：implements HandlerInterceptor

重写 preHandle(request, response, obj)，postHandler(request, response, obj)，afterCompletion(request, response, obj)

在preHandler()方法里面写拦截逻辑：在请求被处理之前，会走preHandler()方法。

