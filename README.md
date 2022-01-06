# Java基础

## equals和hashcode

​	hashcode 是通过散列方法进行计算获取唯一的hash值。在hashMap中将通过hash确认对象在桶的位置。即便是通过桶索引(bucketIndex)找到了对象也要调用一个或者多个equals方法去一个一个比较，成功了才属于真正的对象相等。因为一个桶里面只能装一个entry对象，但是entry对象是可以包含一个索引值去指向另一个entry，最终可以形成entry链；

​	equal是源于Object类，默认方法是去对比两个对象的引用，可以通过重写equal方法去自定义验证对象是否相等的新规则。

> hashcode详解  https://www.cnblogs.com/whgk/p/6071617.html 



## StringBuffer、StringBuilder和String

1）StringBuffer和StringBuilder初始化对象的时候是创建一个默认长度为16的char数组，修改都是针对char[]进行操作；String的底层是创建一个final 的char数组，他的操作是将引用对象指向新的对象。

2）StringBuffer是线程安全的，StringBuilder是线程不安全的。



## Java的重写和重载

1）重写是子类继承父类的方法进行重写，重载是同一个类下相同方法名，可以根据传参类型、传参个数的不同进行方法重载。

2）子类重写的方法名、返回类型、传参必须和父类完全相同，修饰符、异常范围必须小于等于父类；重载只要求方法名相同，传参数量和类型必须不同；

![重写与重载](image/重写与重载.png)

## HashMap的扩容机制

​	初始化HashMap时如果不传容量大小，默认16，加载因子为0.75；当容器容量乘上加载因子小于当前哈希条目时，进行扩容操作，每次扩容都是以2的指数倍进行扩容；

​	Java1.7在扩容时需要对每个元素进行rehash计算扩容后的hash地址，而1.8之后通过hash值和oldCap(旧容器大小)进行&位计算，得出新的hash地址；因为数组是以2的指数倍扩容，就相当于二进制中在多一个高位，如果高位是1，则将其调整到原索引加上oldCap的位置上，如果是0则无需调整。



## 为什么HashMap扩容的时候是二的指数倍

1）因为在二进制中2的指数减去1，这个结果n的二进制全部为1，再拿n去与hash值进行&位运算可以充分的散列，避免必要的hash冲突。

2）由于扩容后最高位是否为1可以判断元素是否移位，优化性能。



# Spring框架

## 对Spring 的理解

1）Spring是一个开源框架

2）Spring是开源的，拥有庞大的且活跃的社区

3）Spring有两大核心：AOP(面向切面编程)和IOC(控制反转)

4）Spring的ORM和DAO提供了与第三方持久层框架的的良好整合，并简化了底层的数据库访问



## 对AOP的理解

​		AOP的意思是面向切面编程，任何一个系统都是由不同的组件组成的，每个组件负责一块特定的功能，当然会存在很多组件是跟业务无关的，例如日志、事务、权限等核心服务组件，这些核心服务组件经常融入到具体的业务逻辑中，如果我们为每一个具体业务逻辑操作都添加这样的代码，很明显代码冗余太多，因此我们需要将这些公共的代码逻辑抽象出来变成一个切面，然后注入到目标对象（具体业务）中去，AOP正是基于这样的一个思路实现的，通过动态代理的方式，将需要注入切面的对象进行代理，在进行调用的时候，将公共的逻辑直接添加进去，而不需要修改原有业务的逻辑代码，只需要在原来的业务逻辑基础之上做一些增强功能即可。

### proxy 

​	proxy是java内置的动态代理，其特点是无需第三方依赖；

​	proxy代理的底层逻辑是通过反射的原理，它是可以对多个不同的接口实现类中定义的方法进行代理；

### cglib

​	cglib的特点是需要导入第三方依赖；

​	cglib代理的底层逻辑是通过子类继承父类去重写方法的方式进行动态代理；



## 对IOC的理解

​	IOC（invest of controll） ，意思是控制反转，它是一种设计思想，并不是一种技术框架或者结构。在传统的创建对象中，是通过对象内部的new方法去实例一个对象，而IOC则是通过特定的容器去扫描注解，自动去创建依赖对象。

​	正转的意思是通过对象主动控制去直接获取依赖对象，而反转则是通过容器来帮忙创建和注入依赖对象。

​	IOC容器使用map结构来存储，在Spring中一般存在三级缓存，singletonObjects存放完整的bean对象，整个生命周期中，从创建到使用到销毁的过程都是由IOC容器来控制的。

### IOC底层逻辑

1.  容器创建过程设计的关键类（beanFactory，DefaultListableBeanFactory），向Bean工厂中设置参数涉及的关键类（BeanPostProcessor，Aware接口的子类）。
2. 加载解析bean对象，准备要创建的bean对象的定义对象beanDefintion（xml或者注解的解析过程）
3. beanFactoryPostProcessor的处理，此处是扩展点，PlaceHolderConfigurSupport，ConfigurationClassPostProcessor
4. BeanPostProcessor的注册功能，方便后续对bean对象完成具体的扩展功能（占位符的扩展）
5. BeanPostProcessor通过反射的方式实例化bean对象
6. bean对象的初始化过程（填充属性、调用aware子类的方法，调用BeanPostProcessor前置处理方法、调用init-method、调用BeanPostProcessor后置处理方法）
7. bean销毁过程

Bean生命周期流程中调用的关键方法：

getBean->doGetBean->createBean->doCreateBean->createBeanInstantce->populateBean->initializingBEan

## BeanFactory和ApplicationContext

相同:

- 它们两个都是spring下的IOC容器，都是Interface接口，ApplicationContext继承于BeanFactory（ApplicationContext集成于ListableBeanFactory，ListableBeanFactory继承于BeanFactory）
- 它们都可以用来配置XML属性，也支持属性的自动注入。

不同：

- BeanFactory在调用getBean()的时候才实例化Bean（懒加载），而ApplicationContext是在启动容器的时候实例化Bean（非懒加载）。
- 因为ApplicationContext是BeanFactory的扩展，提供了更多的功能:支持国际化、事件传递、Bean自动装配、各种不同应用层的Context实现。

## Spring扫描底层的原理

1. 首先，通过ResourcePatternResolver获得指定包路径下的所有 .class 文件（Spring源码中将此文件包装成了Resource对象)
2. 遍历每个Resource对象
3.  利用MetadataReaderFactory解析Resource对象得到MetadataReader（在Spring源码中MetadataReaderFactory具体的实现类为CachingMetadataReaderFactory，MetadataReader的具体实现类为SimpleMetadataReader）
4. 利用MetadataReader进行excludeFilters和includeFilters，以及条件注解@Conditional的筛选（条件注解并不能理解:某个类上是否存在@Conditional注解，如果存在则调用注解中所指定的类的match方法进行匹配，匹配成功则通过筛选，匹配失败则pass掉）
5. 筛选通过后，基于metadataReader生成ScannedGenericBeanDefinition
6. 再基于metadataReader判断是不是对应的类是不是接口或抽象类
7. 如果筛选通过，那么就表示扫描到了一个Bean，将ScannedGenericBeanDefinition加入结果集

## Spring是如何简化开发的

- 基于POJO的轻量级和最小侵入性编程
- 通过依赖注入和面向接口实现松耦合
- 基于切面和惯例进行声明式编程
- 通过切面和模板减少样板式代码



## Spring bean的生命周期

1）实例化（CreateBeanInstance）

​	在Java堆内存开辟空间，通过反射的方式生成对象

2）设置属性

1. 用户自定义属性赋值/注入依赖，如果出现循环依赖，需要解决循环依赖（populateBean()）
2. 容器对象赋值/调用Aware接口（invokeAwareMethods），完成BeanName，BeanFactroy，BeanClassLoader对象的属性设置

3）初始化 （Initialization）

1. 调用BeanPostProccessor的前置接口 postProcessBeforeInitialization() （ApplicationContextPostProcessor、ApplicationContext、Environment、ResourceLoader等对象）
2. InvokeInitMethod(),判断当前bean对象是否设置了InitializingBean接口，然后进行属性的设置等基本工作
3. 如果当前bean对象定义了初始化方法(afterPropertiesSet)，那么在此处调用初始化方法
4. 调用BeanPostProccessor的后置接口-postProcessAfterInitialization(), Spring中的Aop就是在这实现的(AbstractAutoProxyCreator)

4）调用对象(getBean)

5）销毁（Destruction）

1. DisposableBean的destory()方法
2. 自定义的destoryMethod指定方法



> 请别再问**Spring Bean**的生命周期了！ https://www.jianshu.com/p/1dec08d290c1
>
> **Bean的生命周期**流程图  https://www.processon.com/view/61a80d96e401fd49ed415987



## BeanFactory和FactoryBean

1. BeanFactory是负责生产和管理Bean的一个工厂接口，它是最基础的IOC容器，为其他IOC容器提供了规范。
2. FactoryBean是SpringIOC容器是创建Bean的一种方式，用户可以通过重写getObject()方法来自定义Bean的创建过程。



## Spring支持的Bean作用域

① singleton(默认)

​	使用该属性定义Bean时，IOC容器仅创建一个Bean实例，IOC容器每次返回的是同一个Bean实例。

② prototype

​	使用该属性定义Bean时，IOC容器可以创建多个Bean实例，每次返回的都是一个新的实例。

③ request

​	该属性仅对HTTP请求产生作用，使用该属性定义Bean时，每次HTTP请求都会创建一个新的Bean，适用于WebApplicationContext环境。

④ session

​	该属性仅用于HTTP Session，同一个Session共享一个Bean实例。不同Session使用不同的实例。

⑤ global-session

​	该属性仅用于HTTP Session，和session作用域不同的是，所有的Session共享一个Bean实例。



## Spring中Bean的线程安全

Spring中bean对象的默认是单例的，Spring中没有对bean进行多线程的封装，所以面对多线程问题最直接的方式就是把作用域改为prototype，这样在每次调用bean都是创建新的对象避免线程安全问题。

## Spring框架中使用的设计模式及应用场景

1. 工厂模式，在各种BeanFactory以及ApplicationContext创建中都用到了
2. 模版模式，在各种BeanFactory以及ApplicationContext实现中也都用到了
3. 代理模式，Spring AOP 利用了 AspectJ AOP实现的! AspectJ AOP 的底层用了动态代理
4. 策略模式，加载资源文件的方式，使用了不同的方法，比如：ClassPathResourece，FileSystemResource，ServletContextResource，UrlResource但他们都有共同的借口Resource；在Aop的实现中，采用了两种不同的方式，JDK动态代理和CGLIB代理
5. 单例模式，比如在创建bean的时候。
6. 观察者模式，spring中的ApplicationEvent，ApplicationListener,ApplicationEventPublisher
7. 适配器模式，MethodBeforeAdviceAdapter,ThrowsAdviceAdapter,AfterReturningAdapter
8. 装饰者模式，源码中类型带Wrapper或者Decorator的都使用了



## Spring事务的实现方式原理

​	Spring事务的本质其实就是数据库对事务的支持，没有数据库的事务支持，Spring是无法提供事务功能的。

​	在使用Spring框架的时候，可以有两种事务的实现方式，一种是编程式事务，有用户自己通过代码来控制事务的处理逻辑，还有一种是声明式事务，通过@Transactional注解来实现。

​	其实事务的操作本来应该是由数据库来进行控制，但是为了方便用户进行业务逻辑的操作，spring对事务功能进行了扩展实现，一般我们很少会用编程式事务，更多的是通过添加@Transactional注解来进行实现，当添加此注解之后事务的自动功能就会关闭，有spring框架来帮助进行控制。

​	其实事务操作是AOP的一个核心体现，当一个方法添加@Transactional注解之后，spring会基于这个类生成一个代理对象，会将这个代理对象作为bean，当使用这个代理对象的方法的时候，如果有事务处理，那么会先把事务的自动提交给关系，然后去执行具体的业务逻辑，如果执行逻辑没有出现异常，那么代理逻辑就会直接提交，如果出现任何异常情况，那么直接进行回滚操作，当然用户可以控制对哪些异常进行回滚操作。

## Spring事务的隔离级别

spring中的事务隔离级别就是数据库的隔离级别，有以下几种：

- read uncommitted （未提交读）      脏读、幻读、不可重复读
- read committed      （已提交读）      幻读、不可重复读
- repeatable read     （可重复读）       幻读
- serializable             （可序列化）       效率较低

在进行配置的时候，如果数据库和spring代码中的隔离级别不同，那么以spring的配置为主。

## Spring的事务传播机制

多个事务方法相互调用时，事务如何在这些方法之间进行传播,spring中提供了7中不同的传播特性，来保证事务的正常执行：

​	REQUIRED：默认的传播特性，如果当前没有事务，则新建一个事务，如果当前存在事务，则加入这个事务

​	SUPPORTS：当前存在事务，则加入当前事务，如果当前没有事务，则以非事务的方式执行

​	MANDATORY：当前存在事务，则加入当前事务，如果当前事务不存在，则抛出异常

​	REQUIRED_NEW：创建新事务，无论当前存不存在事务，都创建新事务。

​	NOT_SUPPORTED：以非事务方式执行，如果存在当前事务，则挂起当前事务

​	NEVER：不使用事务，如果当前事务存在，则抛出异常

​	NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则按REQUIRED属性执行。

​	NESTED和REQUIRED_NEW的区别：

​	REQUIRED_NEW是新建一个事务并且新开始的这个事务与原有事务无关，而NESTED则是当前存在事务时会开启一个嵌套事务，在NESTED情况下，父事务回滚时，子事务也会回滚，而REQUIRED_NEW情况下，原有事务回滚，不会影响新开启的事务

​	NESTED和REQUIRED的区别：

​	REQUIRED情况下，调用方存在事务时，则被调用方和调用方使用同一个事务，那么被调用方出现异常时，由于共用一个事务，所以无论是否catch异常，事务都会回滚，而在NESTED情况下，被调用方发生异常时，调用方可以catch其异常，这样只有子事务回滚，父事务不会回滚。

## Spring事务失效问题

1. bean对象没有被spring容器管理
2. 方法的访问修饰符不是public
3. 自身调用问题
4. 数据源没有配置事务管理器
5. 数据库不支持事务
6. 异常被捕获
7. 异常类型错误或者配置错误



## Spring中bean的自动装配的方式

bean的自动装配指的是bean的属性值在进行注入的时候通过某种特定的规则和方式去容器中查找，并设置到具体的对象属性中，主要有五种方式：

- no – 缺省情况下，自动配置是通过“ref”属性手动设定，在项目中最常用
- byName – 根据属性名称自动装配。如果一个bean的名称和其他bean属性的名称是一样的，将会自装配它。
- byType – 按数据类型自动装配，如果bean的数据类型是用其它bean属性的数据类型，兼容并自动装配它。
- constructor – 在构造函数参数的byType方式。
- autodetect – 如果找到默认的构造函数，使用“自动装配用构造”; 否则，使用“按类型自动装配”。




## 对SpringBoot的理解

- 用来简化Spring应用的初始搭建以及开发过程，使用特定的方式来进行配置
- 嵌入的tomcat无需部署war文件
- 简化maven配置
- 自动配置Spring添加对应的功能starter自动化配置



## 对SpringBoot中starter的理解

​	使用Spring+SpringMVC框架进行开发的时候，如果需要引入mybatis框架，那么需要在xml中定义需要的bean对象，这个过程很明显是很麻烦的，如果需要引入额外的其他组件，那么也需要进行复杂的配置，因此在SpringBoot中引入了Starter

​	Starter就是一个jar包，写一个@Configuration的配置类，将这些bean定义在其中，然后再Starter包的META-INF/spring.factories中写入配置类，那么SpringBoot程序在启动的时候就会按照约定来加载该配置类

​	开发人员只需要将相应的Starter包依赖进应用中，进行相关的属性配置，就可以进行代码开发，而不需要单独进行bean对象的配置

## bean的自动装配

​	bean的自动装配指的是bean的属性值在进行注入的时候通过某种特定的规则和方式去容器中查找，并设置到具体的对象属性中，主要有五种方式：

- no – 缺省情况下，自动配置是通过“ref”属性手动设定，在项目中最常用
- byName – 根据属性名称自动装配。如果一个bean的名称和其他bean属性的名称是一样的，将会自装配它。
- byType – 按数据类型自动装配，如果bean的数据类型是用其它bean属性的数据类型，兼容并自动装配它。
- constructor – 在构造函数参数的byType方式。
- autodetect – 如果找到默认的构造函数，使用“自动装配用构造”; 否则，使用“按类型自动装配”。
  ​		

## 对SpringMVC的理解

- SpringMVC是一种基于Spring实现了MVC设计模式的Web框架
- 将模型、视图、控制方法分离，实现职责解耦

## SpringMVC的工作流程

当发起请求时被前置的控制器拦截到请求，根据请求参数生成代理请求，找到请求对应的实际控制器，控制器处理请求，创建数据模型，访问数据库，将模型响应给中心控制器，控制器使用模型与视图渲染视图结果，将结果返回给中心控制器，再将结果返回给请求者。

![springmvc运行流程](image/springmvc运行流程.png)

1、DispatcherServlet表示前置控制器，是整个SpringMVC的控制中心。用户发出请求，DispatcherServlet接收请求并拦截请求。
2、HandlerMapping为处理器映射。DispatcherServlet调用HandlerMapping,HandlerMapping根据请求url查找Handler。
3、返回处理器执行链，根据url查找控制器，并且将解析后的信息传递给DispatcherServlet
4、HandlerAdapter表示处理器适配器，其按照特定的规则去执行Handler。
5、执行handler找到具体的处理器
6、Controller将具体的执行信息返回给HandlerAdapter,如ModelAndView。
7、HandlerAdapter将视图逻辑名或模型传递给DispatcherServlet。
8、DispatcherServlet调用视图解析器(ViewResolver)来解析HandlerAdapter传递的逻辑视图名。
9、视图解析器将解析的逻辑视图名传给DispatcherServlet。
10、DispatcherServlet根据视图解析器解析的视图结果，调用具体的视图，进行试图渲染
11、将响应数据返回给客户端

## SpringMVC的主要组件

1.**HandlerMapping(处理器映射器)**
根据request找到相应的处理器。因为Handler（Controller）有两种形式，一种是基于类的Handler，另一种是基于Method的Handler（也就是我们常用的）

2.**HandlerAdapter(处理器适配器)**
调用Handler的适配器。如果把Handler（Controller）当做工具的话，那么HandlerAdapter就相当于干活的工人

3.**HandlerExceptionResolver(处理器异常解析器)**
对异常的处理

4.**ViewResolver (视图解析器)**
用来将String类型的视图名和Locale解析为View类型的视图

5.**RequestToViewNameTranslator**
有的Handler（Controller）处理完后没有设置返回类型，比如是void方法，这是就需要从request中获取viewName

6.**LocaleResolver**
从request中解析出Locale。Locale表示一个区域，比如zh-cn，对不同的区域的用户，显示不同的结果，这就是i18n（SpringMVC中有具体的拦截器LocaleChangeInterceptor）

7.**ThemeResolver**
主题解析，这种类似于我们手机更换主题，不同的UI，css等

8.**MultipartResolver**
处理上传请求，将普通的request封装成MultipartHttpServletRequest

9.**FlashMapManager**
用于管理FlashMap，FlashMap用于在redirect重定向中传递参数



# 设计模式

## 设计模式七大原则

### 单一职责原则(Single responsibility principle)

​	对类来说，一个类只负责一个职责；如果一个类有两个职责A和B，当A职责的需求改变是可能造成B职责的代码执行错误，那么就需要将A的代码块分为两部分，所以不如一开始就将类的职责统一；

### 接口隔离原则(Interface Segregation Principle)

​	客户端不应该依赖(实现)它不需要的接口，即一个类对另一个类的依赖(实现)应该建立在最小的接口上，即将接口尽可能按需拆分为最小接口。

### 依赖倒转原则(Dependence Inversion Principle)

​	高层模块不应该依赖(实现)低层模块的代码，二者都应该依赖其接口/抽象类。  

### 里氏替换原则(Liskov Substitution Principle)

1. 所有引用到父类的地方都能使用子类（使用子类中父类方法结果不变）
2. 在子类中尽量不要重写父类的方法
3. 在适当情况下，可以通过聚合、组合、依赖来解决问题

### 开闭原则(Open Close Principle)

​	开闭原则就是说对**提供方**提供扩展开放，对**接收方**修改关闭。用抽象构建框架，用实现扩展细节；

​	大致意思就是，**接受方**使用了**提供方**提供的方法B，而**提供方**将方法B进行了扩展，**接收方**继续调用方法B时并不需要修改代码，程序依旧正常执行；

### 迪米特法则(Demeter Principle)

迪米特法则又叫最少知道原则，即一个类对自己依赖的类知道的越少越好。也就是说，对于被依赖的类不管多么复杂，都尽量将逻辑封装在类的内部。对外除了提供public方法，不对外泄漏任何信息。

陌生的类最好不要以局部变量的形式出现在类的内部。	

### 合成服用原则(Composite Reuse Principle)

原则是尽量使用合成/聚合的方式，而不是使用继承。



## 行为型模式

### 访问者模式

封装一些作用于某种数据结构中的各元素的操作，它可以在不改变数据结构的前提下定义作用于这些元素的新的操作。

**使用场景:**

1. 一个对象结构包含很多类对象，它们有不同的接口，而你想对这些对象实施一些依赖于其具体类的操作，也就说是用迭代器模式已经不能胜任的情景。

2. 需要对一个对象结构中的对象进行很多不同并且不相关的操作，而你想避免让这些操作“污染”这些对象的类。

### 模板模式

定义一个操作中的算法的框架，而将一些步骤延迟到子类中。使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。

**使用场景:**

1. 多个子类有公有的方法，并且逻辑基本相同时。

2. 重要、复杂的算法，可以把核心算法设计为模板方法，周边的相关细节功能则由各个子类实现。

3. 重构时，模板方法模式是一个经常使用的模式，把相同的代码抽取到父类中，然后通过钩子函数（见“模板方法模式的扩展”）约束其行为。

### 策略模式

定义一组算法，将每个算法都封装起来，并且使它们之间可以互换。

**使用场景:**

1. 多个类只有在算法或行为上稍有不同的场景。

2. 算法需要自由切换的场景。

3. 需要屏蔽算法规则的场景。

### 状态模式

当一个对象内在状态改变时允许其改变行为，这个对象看起来像改变了其类。

**使用场景：**

1. 行为随状态改变而改变的场景这也是状态模式的根本出发点，例如权限设计，人员的状态不同即使执行相同的行为结果也会不同，在这种情况下需要考虑使用状态模式。

2. 条件、分支判断语句的替代者

### 观察者模式

定义对象间一种一对多的依赖关系，使得每当一个对象改变状态，则所有依赖于它的对象都会得到通知并被自动更新。

1、观察者接口

```java
public interface Observer {
    void response();
}
```

2、实现类

```java
public class ConcreteObserver2 implements Observer {
    @Override
    public void response() {
        System.out.println("1号观察者作出反应！");
    }
}
public class ConcreteObserver2 implements Observer {
    @Override
    public void response() {
        System.out.println("2号观察者作出反应！");
    }
}
```

3、抽象目标

```java
public abstract class Subject {
    //定义一个观察者数组
    protected Vector<Observer> obsVector = new Vector();

    //增加一个观察者
    public void addObserver(Observer o) {
        this.obsVector.add(o);
    }

    //删除一个观察者
    public void delObserver(Observer o) {
        this.obsVector.remove(o);
    }

    //通知观察者方法
    public abstract void notifyObserver();

}
```

4、真实目标

```java
public class ConcreteSubject extends Subject{
    @Override
    public void notifyObserver() {
        for (Observer obs : obsVector) {
            obs.response();
        }
    }

		//测试方法
    public static void main(String[] args) {
        Subject subject=new ConcreteSubject();
        Observer concreteSubject1=new ConcreteObserver1();
        Observer concreteSubject2=new ConcreteObserver2();
        subject.addObserver(concreteSubject1);
        subject.addObserver(concreteSubject2);

        subject.notifyObserver();
    }
}

```

**使用场景:**

1. 关联行为场景。需要注意的是，关联行为是可拆分的，而不是“组合”关系。

2. 事件多级触发场景。

3. 跨系统的消息交换场景，如消息队列的处理机制

### 备忘录模式

### 中介者模式

### 迭代器模式

### 解释器模式

### 命令模式

### 责任链模式

## 创建型模式

### 单例模式

所谓类的单例设计模式，就是采用一定的方法保证在整个的软件系统中，对某个类只能存在一个对象实例，并且该类只提供一个获取该实例的方法(静态方法)。

单例模式保证了系统内部中该类只存在一个对象，节省了系统资源，对于一些需要频繁创建销毁的对象，使用单例模式可以提高系统性能。

------

**单例设计模式八种方式：**

**1、饿汉式(静态常量)**

```java
public class Singleton {

    //构造器私有化，防止外部new
    private  Singleton() {
    }

    //本类内部创建对象实例
    private final static Singleton instantce = new Singleton();

    //提供一个公有静态方法，返回实例对象
    public static Singleton getInstance() {
        return instantce;
    }
}

```

优点：在类加载的时候就完成了实例化，避免了线程同步问题。

缺点：在类加载的时候就完成了实例化，如果从始至终没有使用这个实例，则会造成内存的浪费。

------

**2、饿汉式(静态代码块)**

```java
public class Singleton {

    //构造器私有化，防止外部new
    private  Singleton() {
    }

    //本类内部创建对象实例
    private static Singleton instantce;

    //在静态方法块中，创建单例对象
    static {
        instantce = new Singleton();
    }

    //提供一个公有静态方法，返回实例对象
    public static Singleton getInstance() {
        return instantce;
    }
}
```

优缺点与静态常量写法一致

------

**3、懒汉式(线程不安全)**

```java
public class Singleton {

    private static Singleton instantce;

    //构造器私有化，防止外部new
    private Singleton() {
    }

    //提供一个静态的公有方法，当使用到该方法时，才去进行实例化
    public static Singleton getInstance() {
        if (instantce == null) {
            instantce = new Singleton();
        }
        return instantce;
    }
}
```

优点：避免内存的浪费

缺点：当两个线程同时走进了if语句块时，便会产生多个实例，多线程下破坏单例模式，线程不安全。

------

**4、懒汉式(线程安全，同步方法)**

```java
public class Singleton {

    private static Singleton instantce;

    private Singleton() {
    }

    //加入同步代码，解决线程不安全问题
    public static  synchronized Singleton getInstance() {
        if (instantce == null) {
            instantce = new Singleton();
        }
        return instantce;
    }
}
```

虽然解决了线程不安全问题，由于加了synchronized效率过低。

------

**5、懒汉式(线程不安全，同步方法块)**

```java
public class Singleton {

    private static Singleton instantce;

    private Singleton() {
    }

    public static  Singleton getInstance() {
        if (instantce == null) {
            //加入同步代码块，解决线程不安全问题
            synchronized (Singleton.class){
                instantce = new Singleton();
            }
        }
        return instantce;
    }
}
```

效率问题虽然解决了，但是线程安全问题又出现了

------

**6、双重检查(DCL)**

```java
public class Singleton {

    private  static volatile Singleton instantce;

    private Singleton() {
    }
    //加入双重检查解决线程安全问题
    public static  Singleton getInstance() {
        if (instantce == null) {
            synchronized (Singleton.class){
                if (instantce ==null){
                    instantce = new Singleton();
                }
            }
        }
        return instantce;
    }
}
```

解决了线程安全问题，也大大提高了效率。

------

**7、静态内部类**

```java
public class Singleton {

    private Singleton() {
    }

    //写一个静态内部类，该类中有一个私有属性Singleton
    //在Singleton进行类加载时，并不会对静态内部类进行加载。
    private static class SingletonInstance{
        private static final Singleton INSTANCE = new Singleton();
    }

    //提供一个静态公有方法，直接返回SingletonInstance.INSTANCE
    public static Singleton getInstance(){
        return SingletonInstance.INSTANCE;
    }
}

```

静态内部类的特点：在类加载(Singleton)时不会对静态内部类进行加载(SingletonInstance)

优点：

- 采用类加载机制来保证初始化实例只有一个线程
- 静态内部类方式在Singleton类被装载时并不会立即实例化，只有在调用getInstance方法时，去装在静态内部类SingletonInstance，从而完成Singleton的实例化。
- 类的静态属性只会在第一次加载类的时候进行初始化，所以在这里JVM，帮助我们保证了线程的安全性，在类进行初始化时，别的线程是无法进入的

------

**8、枚举**

```java
public class SingletonTest {
    enum Singleton {
        INSTANCE;
    }
    public static void main(String[] args) {
        Singleton s1=Singleton.INSTANCE;
        Singleton s2=Singleton.INSTANCE;
        System.out.println(s1==s2); //true
        System.out.println(s1.hashCode()==s2.hashCode());//true

    }
   
}
```

借助了jdk1.5中添加的枚举来实现单例模式。不仅解决了多线程同步问题，而且还能防止反序列化重写创建对象。

### 工厂方法模式

工厂方法模式分为两种种：

**1、简单工厂方法模式**

我们把被创建的对象称为“产品”，把创建产品的对象称为“工厂”。如果要创建的产品不多，只要一个工厂类就可以完成，这种模式叫“简单工厂模式”。

在简单工厂模式中创建实例的方法通常为静态方法，因此简单工厂模式又叫作静态工厂方法模式。

简单来说，简单工厂模式有一个具体的工厂类，可以生成多个不同的产品，属于创建型设计模式。

简单工厂模式每增加一个产品就要增加一个具体产品类和一个对应的具体工厂类，这增加了系统的复杂度，违背了“开闭原则”。

1、定义发送消息接口

```java
public interface Sender {
	public void Send();
}
```

2、创建实现类

```java
//邮箱发送
public class MailSender implements Sender {
	@Override
	public void Send() {
		System.out.println("这是邮箱发送!");
	}
}
//手机短信
public class PhoneSender implements Sender {
 
	@Override
	public void Send() {
		System.out.println("这是手机短信发送!");
	}
}
```

3、创建工厂类

```java
public class SendFactory {
 
	public Sender produce(String type) {
		if ("mail".equals(type)) {
			return new MailSender();
		} else if ("phone".equals(type)) {
			return new PhoneSender();
		} else {
			System.out.println("请输入正确的类型!");
			return null;
		}
	}
}
```

4、测试结果

```java
public class FactoryTest {
 
	public static void main(String[] args) {
		SendFactory factory = new SendFactory();
		Sender sender = factory.produce("phone");
		sender.Send();
    //输出：这是手机短信发送!
	}
}
```



**2、普通工厂方法模式**

普通工厂方法模式是对简单工厂模式的进一步抽象化，其好处是可以使系统在不修改原来代码的情况下引进新的产品。



将上面的代码做下修改，改动下SendFactory类就行，如下：

```java
public class SendFactory {
 
	public Sender produceMail(){
		return new MailSender();
	}
	
	public Sender producePhone(){
		return new PhoneSender();
	}
}
```

测试类:

```java
public class FactoryTest {
 
	public static void main(String[] args) {
		SendFactory factory = new SendFactory();
		Sender sender = factory.produceMail();
		sender.Send();
    //输出: 这是邮箱发送!
	}
}
```

使用场景：jdbc连接数据库，硬件访问，降低对象的产生和销毁

### 抽象工厂模式

定义了一个interface用于创建相关或有依赖关系的对象簇，而无需指明具体的类。他是由简单工厂和工厂方法模式的整合，更利于代码的维护和扩展。

1、定义两个接口

```java
//发送接口
public interface Sender {
	public void Send();
}
//提供具体工厂接口
public interface Provider {
	public Sender produce();
}
```

2、实现类

```java
public class MailSender implements Sender {
	@Override
	public void Send() {
		System.out.println("this is mailsender!");
	}
}

public class SmsSender implements Sender {
 
	@Override
	public void Send() {
		System.out.println("this is sms sender!");
	}
}
```

3、工厂类

```java
public class SendMailFactory implements Provider {
	@Override
	public Sender produce(){
		return new MailSender();
	}
}

public class SendSmsFactory implements Provider{
 
	@Override
	public Sender produce() {
		return new SmsSender();
	}
}
```

4、测试类

```java
public class Test {
 
	public static void main(String[] args) {
		Provider provider = new SendMailFactory();
		Sender sender = provider.produce();
		sender.Send();
	}
}
```

这个模式的好处就是，如果你现在想增加一个功能：发及时信息，则只需做一个实现类，实现Sender接口，同时做一个工厂类，实现Provider接口，就OK了，无需去改动现成的代码。这样做，拓展性较好！



使用场景：一个对象族（或是一组没有任何关系的对象）都有相同的约束。 涉及不同操作系统的时候，都可以考虑使用抽象工厂模式

### 建造者模式

将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。

```java
package com.example.test.DesignMode;

public class Build {
    static class Student {
        String name = null;
        int number = -1;
        String sex = null;

        public Student(Builder builder) {
            this.name = builder.name;
            this.number = builder.number;
            this.sex = builder.sex;
        }

        static class Builder {
            String name = null;
            int number = -1;
            String sex = null;

            public Builder setName(String name) {
                this.name = name;
                return this;
            }

            public Builder setNumber(int number) {
                this.number = number;
                return this;
            }

            public Builder setSex(String sex) {
                this.sex = sex;
                return this;
            }
						//在此方法构建
            public Student build() {
                return new Student(this);
            }
        }
    }

    public static void main(String[] args) {
        Student A = new Student.Builder().setName("张 三").setNumber(1).build();
        Student B = new Student.Builder().setSex("男").setName("李 四").build();
        System.out.println(A.name + " " + A.number + " " + A.sex);//张 三 1 男
        System.out.println(B.name + " " + B.number + " " + B.sex);//李 四 -1 男
    }
}
```

在JDK中，StringBulider用到了建造者模式。

### 原型模式

用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。

```java
package com.example.test.DesignMode;

//原型模式
public class Prototype implements Cloneable {
    private String name;

    Prototype(){
        this.name="初始化";
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    protected Object clone() {
        try {
            return super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return null;
    }

    public static void main(String[] args) {
        Prototype pro = new Prototype();
        pro.setName("pro");
        Prototype pro1 = (Prototype) pro.clone();
        System.out.println(pro==pro1);//false
        System.out.println(pro.hashCode()==pro1.hashCode());//false
    }
}
```

**使用原型模式的优点**：

1.性能优良

原型模式是在内存二进制流的拷贝，性能要比直接new一个对象快，在一个循环体内new大量对象时，原型模式可以更好地体现其优点。

2.不会执行对象的构造函数

**使用场景**：

资源优化场景类初始化需要消化非常多的资源，这个资源包括数据、硬件资源等。

性能和安全要求的场景通过new产生一个对象需要非常繁琐的数据准备或访问权限，则可以使用原型模式。

一个对象多个修改者的场景 一个对象需要提供给其他对象访问，而且各个调用者可能都需要修改其值时，可以考虑使用原型模式拷贝多个对象供调用者使用。



## 结构型模式

### 适配器模式

将一个类的接口变换成客户端所期待的另一种接口，从而使原本因接口不匹配而无法在一起工作的两个类能够在一起工作。

主要可分为3种：

- 类适配：创建新类，继承源类，并实现新接口，例如 class adapter extends oldClass implements newFunc{}
- 对象适配：创建新类持源类的实例，并实现新接口，例如 class adapter implements newFunc { private oldClass oldInstance;}
- 接口适配：创建新的抽象类实现旧接口方法。例如 abstract class adapter implements oldClassFunc { void newFunc();}

**使用场景**：

你有动机修改一个已经投产中的接口时，适配器模式可能是适合你的模式。比如系统扩展了，需要使用一个已有或新建立的类，但这个类又不符合系统的接口，怎么办？使用适配器模式，这也是我们例子中提到的。

### 桥接模式(桥梁模式)

将抽象和实现解耦，使得两者可以独立地变化

Circle类将DrwaApi与Shape类进行了桥接

1、定义接口

```java
//绘画接口
public interface DrawAPI {
     void drawCircle(int radius, int x, int y);
}
```

2、实现类

```java
//实现类-画红色的圆
public class RedCircle implements DrawAPI {
    @Override
    public void drawCircle(int radius, int x, int y) {
        System.out.println("Drawing Circle[ color: red, radius: " + radius +", x: " +x+", "+ y +"]");
    }
}
//实现类-画绿色的圆
public class GreenCircle implements DrawAPI {
    @Override
    public void drawCircle(int radius, int x, int y) {
        System.out.println("Drawing Circle[ color: green, radius: " + radius +", x: " +x+", "+ y +"]");
    }
}

```

3、抽象类

```java
abstract class Shape {
    protected DrawAPI drawAPI;

    protected Shape(DrawAPI drawAPI) {
        this.drawAPI = drawAPI;
    }

    public abstract void draw();
}
```

4、形状类以及测试类

```java
class Circle extends Shape {
    private int x, y, radius;

    public Circle(int x, int y, int radius, DrawAPI drawAPI) {
        super(drawAPI);
        this.x = x;
        this.y = y;
        this.radius = radius;
    }

    public void draw() {
        drawAPI.drawCircle(radius, x, y);
    }

    public static void main(String[] args) {
        Shape redCircle = new Circle(100, 100, 10, new RedCircle());
        Shape greenCircle = new Circle(100, 100, 10, new GreenCircle());
        redCircle.draw();
        greenCircle.draw();
    }
}

```

**使用场景:**

1. 不希望或不适用使用继承的场景

2. 接口或抽象类不稳定的场景

3. 重用性要求较高的场景

### 组合模式

将对象组合成树形结构以表示“部分-整体”的层次结构，使得用户对单个对象和组合对象的使用具有一致性。

```java
public class Composite extends Component {
    //构件容器
    private ArrayList componentArrayList = new ArrayList();

    //增加一个叶子构件或树枝构件
    public void add(Component component) {
        this.componentArrayList.add(component);
    }

    //删除一个叶子构件或树枝构件
    public void remove(Component component) {
        this.componentArrayList.remove(component);
    }

    //获得分支下的所有叶子构件和树枝构件
    public ArrayList getChildren() {
        return this.componentArrayList;
    }
}
```

**使用场景:**

1. 维护和展示部分-整体关系的场景，如树形菜单、文件和文件夹管理。

2. 从一个整体中能够独立出部分模块或功能的场景。

### 装饰器模式

动态地给一个对象添加一些额外的职责。就增加功能来说，装饰器模式相比生成子类更为灵活 。

```java
public interface Source
{
    void method();
}

public class Decorator implements Source {
    private Source source ;

    public void decorate() {
        System.out.println("decorate");
    }

    @Override
    public void method() {
      	//在方法调用前进行装饰
        decorate();
        source.method();
    }
}
```

**使用场景:**

1. 需要扩展一个类的功能，或给一个类增加附加功能。

2. 需要动态地给一个对象增加功能，这些功能可以再动态地撤销。

3. 需要为一批的兄弟类进行改装或加装功能，当然是首选装饰模式。



### 外观模式(门面模式)

要求一个子系统的外部与其内部的通信必须通过一个统一的对象进行。外观模式提供一个高层次的接口，使得子系统更易于使用。

```java
public class Facade {
    private SubSystem1 subSystem1 = new SubSystem1();
    private SubSystem2 subSystem2 = new SubSystem2();
    private SubSystem3 subSystem3 = new SubSystem3();

    public void startSystem() {
        subSystem1.start();
        subSystem2.start();
        subSystem3.start();
    }

    public void stopSystem() {
        subSystem1.stop();
        subSystem2.stop();
        subSystem3.stop();
    }
}
```

**使用场景：**

1. 为一个复杂的模块或子系统提供一个供外界访问的接口

2. 子系统相对独立——外界对子系统的访问只要黑箱操作即可

3. 预防低水平人员带来的风险扩散

### 亨元模式

使用共享对象的方法，用来尽可能减少内存使用量以及分享资讯。

1、抽象类-flywei

```java
abstract class flywei {}
```

2、对象-Flyweight

```java
public class Flyweight extends flywei {
    Object obj;

    public Flyweight(Object obj) {
        this.obj = obj;
    }
}
```

3、工厂类-FlyweightFactory

```java
public class FlyweightFactory {
    private HashMap data;

    public FlyweightFactory() {
        data = new HashMap<>();
    }

    public Flyweight getFlyweight(Object object) {
        if (data.containsKey(object)) {
            return (Flyweight) data.get(object);
        } else {
            Flyweight flyweight = new Flyweight(object);
            data.put(object, flyweight);
            return flyweight;
        }
    }
}
```

**使用场景：**

1. 系统中存在大量的相似对象。

2. 细粒度的对象都具备较接近的外部状态，而且内部状态与环境无关，也就是说对象没有特定身份。

3. 需要缓冲池的场景。

### 代理模式

为其他对象提供一种代理以控制对这个对象的访问。

1、创建接口-Source

```java
public interface Source {
    void method();
}
```

2、继承类-OldClass

```
public class OldClass  implements Source {
    @Override
    public void method() {

    }
}
```

3、代理类

```java
//代理模式
public class Proxy implements Source {
    private Source source = new OldClass();

    void doSomething() {
    }

    @Override
    public void method() {
        preRequest();
        source.method();
        postRequest();
        doSomething();
    }

    public void preRequest() {
        System.out.println("访问具体方法前操作");
    }

    public void postRequest() {
        System.out.println("访问具体方法后操作");
    }
}
```



> Java之美[从菜鸟到高手演变]之设计模式   https://blog.csdn.net/zhangerqing/article/details/8194653



# Mybatis

## Mybatis的优缺点

**优点**：

- JDBC相比，减少了50%以上的代码量，消除了JDBC大量冗余的代码，不需要手动开关连接；
- 很好的与各种数据库兼容（因为MyBatis使用JDBC来连接数据库，所以只要JDBC支持的数据库MyBatis都支持，而JDBC提供了可扩展性，所以只要这个数据库有针对Java的jar包就可以就可以与MyBatis兼容），开发人员不需要考虑数据库的差异性。
- 提供了很多第三方插件（分页插件 / 逆向工程）
- 能够与Spring很好的集成
- MyBatis相当灵活，不会对应用程序或者数据库的现有设计强加任何影响，SQL写在XML里，从程序代码中彻底分离，解除sql与程序代码的耦合，便于统一管理和优化，并可重用
- 提供XML标签，支持编写动态SQL语句
- 提供映射标签，支持对象与数据库的ORM字段关系映射
- 提供对象关系映射标签，支持对象关系组建维护

**缺点**：

- SQL语句的编写工作量较大，尤其是字段多、关联表多时，更是如此，对开发人员编写SQL语句的功底有一定要求
- SQL语句依赖于数据库，导致数据库移植性差，不能随意更换数据库

## Mybatis中#{}和${}的区别

- #{}是预编译处理，${}是字符串替换
- Mybatis 在处理#{}时，会将 sql 中的#{}替换为?号，调用 PreparedStatement 的 set 方法来赋值；
- Mybatis 在处理${}时，就是把${}替换成变量的值
- 使用#{}可以有效的防止 SQL 注入，提高系统安全性

## Mybatis插件运行原理及开发流程

mybatis只支持针对ParameterHandler、ResultSetHandler、StatementHandler、Executor这四种接口的插件，mybatis使用jdk的动态代理，为需要拦截的接口生成代理对象以实现接口方法拦截功能，每当执行这四种接口对象的方法时，就会进入拦截方法，具体就是InvocationHandler的invoke方法，拦截那些你指定需要拦截的方法。

编写插件：实现Mybatis的Interceptor接口并复写intercept方法啊，然后给插件编写注解，指定要拦截哪一个接口的哪些方法，在配置文件中配置编写的插件即可。

```java
@Intercepts({@Signature(type = StatementHandler.class,method = "parameterize",args = Statement.class)})
```



# JVM

## 1、结构概览

​	class文件 -> 类加载器 -> 运行时数据区（Java栈、本地方法栈、方法区、堆、程序计数器）-> 执行引擎（windows、linux、mac）->本地方法接口（native方法）<- 本地方法库

![JVM体系结构概览](image/JVM体系结构概览.png)





# 数据库

## MySQL



# 中间件

## Redis

## RocketMQ

# 算法

## 八大排序算法

![排序算法复杂度对比图](image/排序算法对比图.png)

### 冒泡排序

### 快速排序

### 归并排序

### 希尔排序

### 直接插入排序

### 基数排序

### 堆排序

### 简单选择排序

# 软件环境

## Linux下安装Java环境

1.执行以下命令，查看yum源中JDK版本。

```shell
yum list java*
```

2.执行以下命令，使用yum安装JDK1.8。

```shell
yum -y install java-1.8.0-openjdk*
```

3.执行以下命令，查看是否安装成功。

```shell
java -version
```

如果显示如下图内容，则表示JDK安装成功。

![Linux下校验Java环境](image/Linux下校验Java环境.png)



------

## Linux下安装Mysql

1.如果Centos7以上的系统，由于系统自带mysql会导致版本冲突，请先禁用mysql

```shell
yum module disable mysql
```

2.执行以下命令，下载并安装MySQL官方的Yum Repository。

```shell
wget http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
yum -y install mysql57-community-release-el7-10.noarch.rpm
yum -y install mysql-community-server
```

3.执行以下命令，启动 MySQL 数据库。

```shell
systemctl start mysqld.service
```

4.执行以下命令，查看MySQL初始密码。

```shell
grep "password" /var/log/mysqld.log
```

5.执行以下命令，登录数据库。

```shell
mysql -uroot -p
```

6. 执行以下命令，修改MySQL默认密码。

```shell
set global validate_password_policy=0;  #修改密码安全策略为低（只校验密码长度，至少8位）。
ALTER USER 'root'@'localhost' IDENTIFIED BY '12345678';
```

7.执行以下命令，授予root用户远程管理权限。

```shell
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '12345678';
```

8. 输入exit退出数据库。

------

## Linux下安装Tomcat

1.执行以下命令，下载Tomcat压缩包。

说明:该资源包可能会失效，请自行去清华镜像官网下载。

```shell
wget https://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-8/v8.5.73/bin/apache-tomcat-8.5.73.tar.gz
```

2.执行以下命令，解压刚刚下载Tomcat包。

```shell
tar -zxvf apache-tomcat-8.5.73.tar.gz
```

3.执行以下命令，修改Tomcat名字。

```shell
mv apache-tomcat-8.5.73 /usr/local/Tomcat8.5
```

4.执行以下命令，为Tomcat授权。

```shell
chmod +x /usr/local/Tomcat8.5/bin/*.sh
```

5.执行以下命令，修改Tomcat默认端口号为80。

说明： Tomcat默认端口号为8080。

```shell
sed -i 's/Connector port="8080"/Connector port="80"/' /usr/local/Tomcat8.5/conf/server.xml
```

![Tomcat端口配置](image/Tomcat端口配置.png)

6. 启动Tomcat。

```shell
/usr/local/Tomcat8.5/bin/./startup.sh
```

7.访问Tomcat。

直接访问你的公网IP即可，如果你是阿里云ECS需添加安全组策略。

![ECS安全组策略](image/ECS安全组策略.png)

## Linux下安装Nginx

1、从nginx官网下载解压包

```shell
wget http://nginx.org/download/nginx-1.18.0.tar.gz
```

2、安装Nginx依赖

```shell
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel
```

3、移动安装包

```shell
mv nginx-1.18.0.tar.gz /usr/local/
```

4、解压压缩包

```shell
tar -zxvf nginx-1.18.0.tar.gz
```

5、进入安装包目录并编译安装

```shell
cd nginx-1.18.0.tar.gz/  ## 进入安装包
./configure							 ## 编译
make && make install     ## 安装
```

shell6、验证是否成功

![Nginx安装验证](image/Nginx安装验证.png)



## Linux下安装Vue运行环境

1、下载node.js解压包

```shell
wget https://nodejs.org/download/release/v9.11.2/node-v9.11.2-linux-x64.tar.xz
```

2、解压

```shell
tar -xf node-v9.11.2-linux-x64.tar.xz
```

3、配置全局变量

```shell
ln -s /usr/local/nodejs/node-v9.11.2-linux-x64/bin/npm /usr/local/bin/ 	##建立软连接，变为全局
rm -f /usr/local/bin/node
ln -s /usr/local/nodejs/node-v9.11.2-linux-x64/bin/node /usr/local/bin/ ##建立软连接，变为全局
node -v ##检验nodejs是否已变为全局
```

<img src="image/检验nodejs全局配置是否生效.png" alt="检验nodejs全局配置是否生效" style="zoom:150%;" />



4、安装cnpm

```shell
npm install -g cnpm --registry=https://registry.npm.taobao.org

##建立软连接
rm -f  /usr/local/bin/cnpm
ln -s /usr/local/nodejs/node-v9.11.2-linux-x64/bin/cnpm  /usr/local/bin/cnpm
```



## Linux下安装Redis

1、下载安装包

```shell
wget http://download.redis.io/releases/redis-3.0.7.tar.gz
```

2、安装[gcc](https://so.csdn.net/so/search?q=gcc)，编译的时候需要依赖gcc环境

```shell
 yum install gcc-c++
```

3、解压安装包

```shell
tar -zxvf redis-3.0.7.tar.gz
```

4、移动文件夹

```shell
mv redis-3.0.7 /usr/local/
```

5、编译 且安装

```shell
make #编译
make PREFIX=/usr/local/redis install #安装
cp redis.conf /usr/local/redis #复制配置文件
```

6、将redis启动模式改为后端启动

```shell
vi /usr/local/redis/redis.conf
```

<img src="image/redis安装.png" alt="redis安装" style="zoom:150%;" />

7、启动redis

```shell
./bin/redis-server ./redis.conf
```

8、登陆redis（阿里云先设置安全组策略）

```shell
./bin/redis-cli -h {ip} -p 6379 #{ip} 是你服务器的ip
./bin/redis-cli -h {ip} -p 6379 -a 123456 #有密码时的指令
```

9、设置密码

```shell
#修改redis.conf配置文件　　

# requirepass foobared
requirepass 123456   指定密码123456
```

10、设置日志输出地址(解决Redis关闭报错问题)

```shell
#修改redis.conf配置文件　　

# output for logging but daemonize, logs will be sent to /dev/null
logfile "/usr/local/redis/redis_log.log " #注意logfile 后面一定要留空格 保存并退出
```

## Mac下安装Maven

1、下载资源包

下载地址：https://maven.apache.org/download.cgi

![Maven资源包](image/Maven资源包.png)

2、将下载的压缩包解压到/usr/local文件夹下

​	通过指令

```shell
cd /usr/local
sudo mv /Users/chenjiaofu/Downloads/apache-maven-3.8.4-bin.tar.gz ./
tar zxvf apache-maven-3.8.4-bin.tar.gz
```

​	通过finder访问/usr/local，进行移动文件并解压；（**访问文件夹快捷键 command+shift+G**）

3、修改配置文件

打开终端，输入vim ~/.bash_profile 按i进入编辑模式

或者直接 open ~/.bash_profile 打开配置文件



将下面两句拷贝到文件内

```tex
export MAVEN_HOME=/usr/local/apache-maven-3.8.4
export PATH=$MAVEN_HOME/bin:$PATH
```

添加之后 执行 source ~/.bash_profile 为配置环境变量生效

生效后 测试 mvn -v 查看是否出现maven版本

![校验Maven环境变量](image/校验Maven环境变量.png)



4、配置国内镜像

编辑`MAVEN_HOME/conf/setting.xml`文件找到`<mirrors>`节点，往里面添加配置代码

```xml
<mirror>
        <id>alimaven</id>
        <name>aliyun maven</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
        <mirrorOf>central</mirrorOf>
    </mirror>
    <mirror>
        <id>central</id>
        <name>Maven Repository Switchboard</name>
        <url>http://repo1.maven.org/maven2/</url>
        <mirrorOf>central</mirrorOf>
    </mirror>
    <mirror>
        <id>repo2</id>
        <mirrorOf>central</mirrorOf>
        <name>Human Readable Name for this Mirror.</name>
        <url>http://repo2.maven.org/maven2/</url>
    </mirror>
    <mirror>
        <id>ibiblio</id>
        <mirrorOf>central</mirrorOf>
        <name>Human Readable Name for this Mirror.</name>
        <url>http://mirrors.ibiblio.org/pub/mirrors/maven2/</url>
    </mirror>
    <mirror>
        <id>jboss-public-repository-group</id>
        <mirrorOf>central</mirrorOf>
        <name>JBoss Public Repository Group</name>
        <url>http://repository.jboss.org/nexus/content/groups/public</url>
    </mirror>
    <!-- 中央仓库在中国的镜像 -->
    <mirror>
        <id>maven.net.cn</id>
        <name>oneof the central mirrors in china</name>
        <url>http://maven.net.cn/content/groups/public/</url>
        <mirrorOf>central</mirrorOf>
    </mirror>
```

5、配置本地仓库

```xml
<localRepository>/Users/chenjiaofu/Documents/apache-maven-3.8.4/maven-repo</localRepository>
```





# Github

## MAC系统下绑定github,ssh key

参考文章 https://www.jianshu.com/p/f008d9fe3f34/



## GitHub常用指令

```
git config --global --edit //全局打开配置文件

git pull orgin main //更新
git clone xxx  //从服务器上将代码给拉下来
git commit -am "提交的注释" //提交代码
git add [file1] [file2] [directory1] //添加文件或文件夹（包括文件夹下所有文件）
git rm [file1] [directory1] //删除文件或文件夹
git push orgin main//将本地文件推送到云端，20年底分支从默认的master改为main，所以这里是main
git branch -m br_rename_old br_rename_new //将本地仓库的br_rename_old的名称修改为br_rename_new

```



# 平时遇到的BUG

## Springboot项目打包失败

![Springboot项目打包失败](image/Springboot项目打包失败.png)

解决方法：

​	在pom文件下的plugins下添加代码块

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-resources-plugin</artifactId>
    <version>2.4.3</version>
</plugin>
```

## Mac系统下DNS污染

```shell
sudo killall -HUP mDNSResponder ##清除DNS缓存
```

## SpringBoot下跨域问题

通过实现WebMvcConfigurer接口然后重写addCorsMappings方法解决跨域问题。

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .allowCredentials(true)
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                .maxAge(3600);
    }

}	
```



# 常用指令

```shell

nohup java -jar epidemic-0.0.1-SNAPSHOT.jar >	 /dev/null 2>&1 &  ##Jar包后台启动指令

nc -vz -w 2 192.168.1.1 8080 #mac下ping端口指令
telnet 192.168.1.1 8080 #windows下ping端口指令
```
