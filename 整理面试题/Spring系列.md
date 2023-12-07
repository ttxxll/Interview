---
typora-copy-images-to: images
---

### 0.待整理的一些题目

Spring的一些扩展点：比如Bean生命周期的各个扩展点，@Import等

介绍一下Spring

Spring的IOC，DI，AOP介绍，应用，实现，场景



### 1.推断构造方法

在Spring Bean生命周期的过程中，需要根据Java类来创建一个Spring Bean对象，具体的手段就是通过构造方法来实例化得到。

但是在一个Java类里面可能会有多个构造方法，每个构造方法的参数个数，参数类型也可能不一样。所以具体选择哪个构造方法，选择的构造方法中的参数从哪里获取，这就需要一个具体的规则逻辑，而Spring有一个自己的选择逻辑，就是推断构造方法。

在选择构造方法来实例化Bean的时候，具体的逻辑是：

- 如果该Java类只有一个构造方法，无论这个方法是有参还是无参的构造，那么都会选这个构造方法来实例化Bean。
- 如果该Java类有多个构造方法：
  - 那么会默认选择无参的构造方法来实例化Bean。
  - 如果多个构造方法中，没有一个是无参构造，那么会直接报错，提示实例化该Bean失败，没有发现默认构造方法。
  - 报错信息：Error creating bean with name... Instantiation of bean faild. Faild to instantiate [...]: No default constructor found. 创建....bean错误，实例化bean失败：没有发现默认的构造方法。
- @Autowired注解：如果在构造方法上加了@Autowired注解，那么就选择加了@Autowired注解的构造方法，不会选择默认的构造方法。
- 参数推断：如果最终选择的是一个有参的构造方法，那么在实例化时，或者说Spring在调用这个构造方法时，具体的参数值来自哪里？会根据参数类型和参数名字到Spring容器中进行匹配，匹配到的bean对象作为入参传给构造方法，具体的参数匹配逻辑：
  - 先根据参数类型匹配，如果只能匹配到一个，就选择它作为入参（即使这个找到的对象的变量名和入参的变量名不匹配）。
  - 如果根据参数类型匹配到了多个bean，那么接下来会再根据参数名称进一步筛选。如果选择出唯一的一个对象，那么就作为入参。如果还没有匹配到，那么就会报错。

综上所述，在Spring实例化Bean的时候，确定用哪个构造方法，确定哪些Bean对象作为构造方法中的入参，这样一个过程称为推断构造方法。



### 2.Spring的几种Bean

1. 通过@Componnet，@Service，@Controller这些注解生成的Spring Bean默认都是非懒加载的单例Bean。非懒加载的单例Bean，是随着Spring容器初始化的时候就创建生成了，并且因为是单例bean，后面每次获取使用时都是同一样bean对象。
2. 如果加上@Lazy注解，那么这个bean时一个懒加载的单例bean。懒加载的单例bean是在第一次获取使用时才会真正创建出来。
3. 如果加了@Scope("prototype")注解，那么这个bean是一个多例bean，每次获取使用bean对象时都会创建一个新的bean对象给使用者。



### 3.@Conditional注解

1. 是Spring框架提供的一个注解，@Conditional注解的作用是来动态的控制是否注册创建被标注的Spring Bean，一般加在配置类或者@Bean方法上。Spring在创建Bean时，如果目标上加了@Conditional注解，那么会判断该注解中value属性中的Condition接口的matches方法，如果方法返回true，那么会顺利的注册创建SpringBean，否则会忽略。

2. 通过@Conditional源码可以发现，它是一个纯功能的注解，该注解上只有三个元注解：

   - @Target：ElementType.Type，ElementType.Method，指定了@Conditional注解的使用范围是类，接口，注解，枚举，方法，所以它也可以加到我们的自定义注解中，集合我们自己的逻辑，进行额外扩展。
   - @Retention：RetentionPolicy.RUNTIME，该注解不仅在编译后的class文件存在，在jvm加载class文件到内存后也存在，即运行时一直存在。所以可以在运行时通过反射来访问获取该注解的信息。
   - @Documented：该注解的使用访问是注解（@Target(ElementType.ANNOTATION_TYPE)），加了该注解意味着，在生成Java文档时，会包含被@Documented注解的注解信息。

3. Condition接口和matches方法

   ```java
   public class TaoOnClassCondition implements Condition {
       @Override
       public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
   
           // 通过元数据获取到TaoConditionalOnClass注解元数据信息
           Map<String, Object> annotationAttributes = metadata.getAnnotationAttributes(TaoConditionalOnClass.class.getName());
   
           // 获取TaoConditionalOnClass注解的value属性
           String className = (String) annotationAttributes.get("value");
   
           try {
               // 如果能加载到该类，说明引入了对应的依赖
               context.getClassLoader().loadClass(className);
               return true;
           } catch (ClassNotFoundException e) {
               // 没有该类的依赖
               return false;
           }
       }
   }
   ```

   matches方法中的参数：

   - 程序上下文环境参数ConditionContext context：这个参数提供了很多有关当前应用程序上下文和环境的信息，可以获得以下环境变量，
   - 元数据参数AnnotatedTypeMetadata metadata：该参数是要创建的SpringBean对应的类或方法上的元数据信息。通过该参数能拿到该类或方法的相关信息，从而能拿到条件注解的信息，也能获取该Bean上定义的其他注解信息。

4. 示例

   TaoOnClassCondition：判断是否能存在该依赖，有的话就返回true，否则false

   ```java
   public class TaoOnClassCondition implements Condition {
       @Override
       public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
   
           // 通过元数据获取到TaoConditionalOnClass注解元数据信息
           Map<String, Object> annotationAttributes = metadata.getAnnotationAttributes(TaoConditionalOnClass.class.getName());
   
           // 获取TaoConditionalOnClass注解的value属性
           String className = (String) annotationAttributes.get("value");
   
           try {
               // 如果能加载到该类，说明引入了对应的依赖
               context.getClassLoader().loadClass(className);
               return true;
           } catch (ClassNotFoundException e) {
               // 没有该类的依赖
               return false;
           }
       }
   }
   ```

   @TaoConditionalOnClass注解：加了该注解的类或者@Bean方法在创建SpringBean时，会判断value中指定的类是否存在，存在的话才会注册创建SpringBean。

   ```java
   // 使用范围：类，接口，注解，枚举，方法
   @Target({ElementType.TYPE, ElementType.METHOD})
   // 运行时一直存在：能在运行时通过反射获取到该自定义注解
   @Retention(RetentionPolicy.RUNTIME)
   // 加了@TaoConditionalOnClass注解的类，也相当于加了@Conditional注解，所以matches方法中的metadata参数拿到的是加了当前@Conditional注解对象的类的元数据
   @Conditional(TaoOnClassCondition.class)
   public @interface TaoConditionalOnClass {
   
       String value();
   }
   ```

   MyWebServerAutoConfiguration：自动WebServer配置类，根据项目中引入的依赖动态的注册对应的WebServer。

   ```java
   @Configuration
   @TaoConditionalOnClass("com.taoxl.abc.WebServerAutoConfiguration")
   public class MyWebServerAutoConfiguration {
   
       @Bean
       @TaoConditionalOnClass("com.taoxl.abc.web-server.auto.AutoTomcat")
       public TomcatWebServer tomcatWebServer() {
           return new TomcatWebServer();
       }
   
       @Bean
       @TaoConditionalOnClass("com.taoxl.abc.web-server.auto.AutoJetty")
       public JettyWebServer jettyWebServer() {
           return new JettyWebServer();
       }
   }
   ```

   

5. 除了@Conditional注解外，springboot通过@Conditional注解又扩展了很多类似的注解，比如：

   @ConditionalOnProperty
   @ConditionalOnMissingBean
   @ConditionalOnBean
   @ConditionalOnClass

   这四个注解都是SpringBoot框架中的条件注解，他们用于基于特定条件来控制SpringBoot自动配置的行为。

   - @ConditionalOnProperty：这个注解用于在指定的Environment中存在某个属性且具有特定的值时，启用特定的自动配置。它可以用来根据应用程序的属性配置来决定是否启用某个自动配置类，你可以指定一个或多个属性名和期望的属性值，只有这些属性存在并且值为具体指定的值时，才会生效对应的自动配置。
   - @ConditionalOnMissingBean：该注解用于判断在Spring上下文中如果不存在指定类型的Bean，那么才会启用响应的自动配置，这个注解允许你根据Bean是否缺失来控制自动配置的生效。这个注解在SpringBoot的自动配置中很重要，利用这个注解SpringBoot来决定到底是用用户自定定义的配置Bean，还是用SpringBoot自动配置的Bean。厂商会在提供给我们服务时一般会给我们做好默认的配置，但是厂商考虑到默认配置无法满足用户的需求，用户可能需要做一些自定义的配置 ，所以还会留下来一些接口给用户做扩展或者做自定义的配置。所以通过@ConditionalOnMissingBean注解我们来判断是否不存在这些接口的Spring Bean，如果有不存在用户做的自定义配置，那么SpringBoot就用自己事先写好的通用配置。
   - @ConditionalOnBean：这个注解用于判断在Spring上下文中是否已经存在特定类型的Bean，只有存在指定类型的Bean时，才会启用相应的自动配置。这个注解允许你根据Bean的存在来控制自动配置的生效。
   - @ConditionalOnClass：这个注解用以判断指定的类是否在类路径上存在，只有指定的类存在于类路径上时，才会启用相应的自动配置。这个注解允许你根据类是否存在来控制自动配置的应用。

   这些配置通常与SpringBoot的自动配置一起使用，帮助开发者根据特定的条件来自动加载和配置SpringBean，以适应不同的运行时环境或配置需求。




### 4.@Autowired，@Resource，@Qualifier，@Primary注解

1. @Autowired 依赖注入

   - 加在属性上：先根据属性类型找Bean，如果找到多个会再根据属性名去匹配。

     ```java
     @RestController
     @RequestMapping("/novel-chapter")
     public class NovelChapterController {
     
         @Autowired
         private TestService testService;
     }
     ```

     ```java
     @Configuration
     public class TestConfig {
     	// 注册一个TestService类型，且beanName=testService的bean
         @Bean
         public TestService testService() {
             return new TestService();
         }
     
         @Bean
         public TestService getService() {
             return new TestService();
         }
     
         @Bean
         public TestService getService1() {
             return new TestService();
         }
     }
     ```

     

   - 加在构造方法时：Spring容器在进行构造方法推断时，会选择该方法来实例化得到bean。如果构造方法有参数，那么入参会根据参数类型到Spring容器中寻找。如果找到多个，会再根据参数名去匹配。

   - 加在set方法上：会给对象的属性进行注入赋值，并且对象的set方法的入参会根据参数类型到Spring容器中匹配，如果匹配到多个，那么会再根据参数名去匹配。

   - required属性：默认是true，在进行注入时，如果没有找到匹配的Bean，会报错，必须有。

   - 如果找到多个：

     - 那么可以结合@Qualifier注解指定beanName，

       ```java
       @Autowired
       @Qualifier("birth")
       private Date birthday ;
       ```

       

     - 在多个Bean中选择一个加上@Primary，该Bean会被注入进去

       ```java
       @Configuration
       public class TestConfig {
       
       //    @Bean
       //    public TestService testService() {
       //        return new TestService();
       //    }
       
           @Bean
           @Primary
           public TestService getService() {
               return new TestService();
           }
       
           @Bean
           public TestService getService1() {
               return new TestService();
           }
       }
       ```

       

     - 在多个Bean中选择一个加上@Priority，让其优先级最高。

       

2. @Resource注解

   Resource是根据属性名进行注入，如果没有会再按照类型匹配。

   如果我们指定了name属性，那么如果根据name没有找到，就会直接报错，不会再按照类型匹配。

   是Java提供的注解，Spring支持的，和Spring解耦。

3. @Qualifier

   搭配@Autowired注解，在Spring容器中有多个相同类型的Bean，可以用来指定Bean的名称。

   ```java
   @Autowired
   @Qualifier("birth")
   private Date birthday ;
   ```

   也可以加在方法参数上，根据@Qualifier指定的名字到Spring容器中匹配，然后注入到参数中。

   ```java
   @Service
   public class SomeService {
   
       private final UserService userService;
   
       @Autowired
       public SomeService(@Qualifier("userServiceImplA") UserService userService) {
           this.userService = userService;
       }
   }
   ```

   

   筛选限定的Bean注入：只收集带有@Qualifier的候选者，比如我们想要注入A类型的List集合，那么我们可以通过@Qualifier控制注入哪些A类型的Bean。

   ```java
   @Autowired
   @Qualifier
   private List<TestService> testServices;
   
   @Configuration
   public class TestConfig {
       @Bean
       public TestService getService() {
           return new TestService();
       }
   	
       // 只有getService1会被注入到testServices中
       @Bean
       @Qualifier
       public TestService getService1() {
           return new TestService();
       }
   }
   
   ```

   

   在Ribbon的@LoadBanlance注解中就是用到了这个功能，@LoadBalanced注解中就有@Qualifier注解，所以@LoadBalanced也有@Qualifier注解的功能。

   ```java
   @Target({ ElementType.FIELD, ElementType.PARAMETER, ElementType.METHOD })
   @Retention(RetentionPolicy.RUNTIME)
   @Documented
   @Inherited
   @Qualifier
   public @interface LoadBalanced {
   
   }
   ```

   我们可以将@LoadBalanced加在RestTemplate类型的Bean上。

   ```java
   @Configuration
   public class RestConfig {
   
       /**
        * RestTemplate：进行API的调用
        * LoadBalanced：
        *  1.开启负载均衡的功能，只需要在RestTemplate的bean定义上添加@LoadBalanced注解即可
        *  2.由于加了@LoadBalanced注解，底层会使用RestTemplateCustomizer对所有标注了@LoadBalanced的RestTemplate的Bean添加一个
        *      LoadBalancedInterceptor拦截器。利用该拦截器，Spring可以对RestTemplate这个Bean进行定制，比如进行服务名和ip端口的映射，
        *      这样我们在进行服务间调用时，就可以直接用服务名即可。
        	3.Ribbon的自动配置类只会收集带有@LoadBalanced注解的RestTemplate类型的Bean
        * @return
        */
       @Bean
       @LoadBalanced  // 微服务名替换为具体的ip:port
       public RestTemplate restTemplate() {
           return new RestTemplate();
       }
   
   }
   
   ```

   在Ribbon的自动配置类中，会注入一个RestTemplate类型的List，并且该List也被@LoadBalanced注解标注。说明只收集带有@LoadBalanced注解的RestTemplate对象。

   ![image-20230821101743166](D:\笔记\整理面试相关\整理面试题\images\image-20230821101743166.png)



### 5.@EnableConfigurationProperties和@ConfigurationProperties

1. @ConfigurationProperties是Spring中的一个注解，用于将配置文件中的属性值绑定到Java类的字段上。它的作用是将应用程序的配置属性和Java对象关联起来，以便于统一管理和访问配置信息。

   - 属性绑定：通过在Java类的字段上添加@ConfigurationProperties注解，并指定prefix属性，可以将外部配置文件中的特定的前缀的属性自动绑定到这些字段上。这样，你就不需要手动配置文件，解析属性值，然后再将其赋值给对象属性。
   - 提高可读性和维护性：将配置属性集中在一个类中，可以提高代码的可读性和维护性。开发人员可以更加直观地查看那行属性被使用，以及他们的含义和默认值。
   - 结合SpringBoot的自动配置：在SpringBoot应用程序中，@ConfigurationProperties经常与@EnableConfigurationProperties一起使用，@EnableConfigurationProperties加载自动配置类上，以便支持自动配置类的属性绑定。

2. @EnableConfigurationProperties

   @EnableConfigurationProperties的主要作用是用来启用对@ConfigurationProperties注解类的支持，它的value属性中指定若干属性配置类（即加了@ConfigurationProperties注解的）。后续在SpringBoot应用启动时，Spring会开启对这些属性配置类的支持，将他们扫描注册成Bean，然后根据他们指定的prefix，将配置文件中的内容绑定到他们的属性中。

   - 启用属性绑定：通过在配置类上添加@EnableConfigurationProperties注解，SpringBoot应用程序会解析注册注解value属性中指定的属性配置类，这些属性配置类一般都标有@ConfigurationProperties注解。然后将配置文件中的符合prefix的属性值绑定到带有@ConfigurationProperties注解的类的字段上。
   - 创建配置属性类的Bean：当使用@EnableConfigurationProperties注解时，可以在value中指定若干带有@ConfigurationProperties注解的属性配置类，后续SpringBoot应用在启动时会自动扫描创建这些属性配置Bean，并且将配置文件在的属性值绑定到属性配置Bean的字段中。

3. 结合SpringBoot的自动配置

   在SpringBoot应用程序中，@ConfigurationProperties经常与@EnableConfigurationProperties一起使用，@EnableConfigurationProperties加载自动配置类上，以便支持自动配置类的属性绑定。

   在SpringBoot整合Feign的自动配置类上，通过@EnableConfigurationProperties指定了3个属性配置Bean

   ```java
   @EnableConfigurationProperties({FeignClientProperties.class, FeignHttpClientProperties.class, FeignEncoderProperties.class})
   ```

   ![image-20230822151010148](D:\笔记\整理面试相关\整理面试题\images\image-20230822151010148.png)

   

   打开FeignClientProperties发现它用@ConfigurationProperties指定了配置文件中的feign.client前缀，所以配置文件中的该前缀对应的属性会被加到FeignClientProperties类的4个字段中。

   ![image-20230822152022514](D:\笔记\整理面试相关\整理面试题\images\image-20230822152022514.png)

   ![image-20230822152151496](D:\笔记\整理面试相关\整理面试题\images\image-20230822152151496.png)

4. @ConfigurationProperties也可以不搭配@EnableConfigurationProperties使用

   不搭配@EnableConfigurationProperties使用，那就要通过@Configuration或者@Bean将其注册到Spring容器中。

   ```java
   @Configuration
   @ConfigurationProperties(prefix = "app")
   public class AppConfig {
       private String name;
       private int maxConnections;
       
       // Getters and setters for properties
       
       // Other bean definitions
   }
   ```

5. 所以通过属性配置类，我们基本不需要去记ymal中的配置了，到@ConfigurationProperties标注的属性配置类中查看相应的字段即可。

