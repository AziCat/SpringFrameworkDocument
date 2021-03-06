# IoC容器
## 介绍Spring IoC容器和beans

本章节涵盖了spring framework的控制反转(IoC)的实现原理。IoC也被称为依赖注入(Dependence Injection)。
这是一个过程，对象通过构造参数，工厂方法的参数或者通过给对象初始化后的实例设置属性来定义它们的依赖关系。
当创建bean的时间容器就会注入这些依赖。此过程从根本上说是反转的，因此命名为控制反转(IoC)，bean本身根据依赖关系来通过直接使用类的构造方法
来控制本身的实例化或者位置，例如服务定位模式的机制。

`org.springframework.beans`和`org.springframework.context`包是SpringFramework控制反转容器的基础。
[BeanFactory](https://docs.spring.io/spring-framework/docs/5.0.2.RELEASE/javadoc-api/org/springframework/beans/factory/BeanFactory.html)
接口提供一个高级配置机制可以管理所有类型的对象。[ApplicationContext](https://docs.spring.io/spring-framework/docs/5.0.2.RELEASE/javadoc-api/org/springframework/context/ApplicationContext.html)是`BeanFactory`的子接口。
通过它能更容易进行集成Spring AOP的特性；消息资源处理（国际化处理），事情发布；应用层特定的上下文对象例如
使用在Web应用的`WebApplicationContext`。

简单来说，`BeanFactory`提供了配置框架和基础功能，`ApplicationContext`增加了很多企业专用的功能。
`ApplicationContext`是`BeanFactory`的一个完整超集，仅在此章节用来描述Spring IoC容器。

在Spring，应用程序的主干和由Spring IoC容器管理的对象称为`beans`。一个bean是实例化、组装并由Spring IoC容器管理的对象。
否则，bean则是你的应用程序中多个对象中的其中的简单一个而已。Beans和它们之间的依赖关系，反映在容器使用的配置元数据中。

## 容器概述
接口`org.springframework.context.ApplicationContext`代表了Spring IoC容器，而且负责实例化、配置和组装上述的beans。
容器会通过阅读配置元数据来获取bean的说明以此来决定对象何时被实例化、配置和组装。配置元数据以
XML，java注解或者java代码来表示。这允许你表达组成应用程序的对象和丰富这些对象之间的相互依赖关系。

Spring提供了几个开箱即用的`ApplicationContext`的接口实现。在一个标准的应用中通常会创建
`ClassPathXmlApplicationContext`或者`FileSystemXmlApplicationContext`的实例。虽然XML是定义配置元数据的传统格式，
但也你可以通过提供少量的XML配置来声明允许使用java注解或者java代码这些额外的元数据格式来定义你的容器。

在很多应用场景，实现化Spring IoC容器一个或者多个实例是不需要明确的用户代码。例如，在一个
web应用场景下，应用中的`web.xml`里简单的8行（或者比8行更多）XML描述就已经足够了。

下图从高层级上描述了Spring是怎样工作的。你的应用程序中的类通过配置元数据组合而成，所以当`ApplicationContext`创建和初始化之后，
你就拥有一个完整配置的、可执行的系统或者应用。

![](https://docs.spring.io/spring/docs/current/spring-framework-reference/images/container-magic.png)

***1. The Spring IoC container***

### 配置元数据
就如上述的图片所示，Spring IoC容器消费了一组配置元数据；这些配置元数据向你展示了作为一个应用开发者
如何去告诉Spring容器去实例化，配置和组合你应用中的对象。

配置元数据传统来说一般由一个简单而直观的XML文件来提供，这就是为什么很多章节用XML来传达Spring IoC容器的关键概念和特性。

```
基于XML的元数据不是唯一允许的配置元数据的形式。配置元数据的写入格式实际上是与Spring IoC容器分离开来的。
许多开发者选择了基于java来配置他们的应用。
```

关于使用Spring IoC容器其它配置形式，请查看：

* [基于注解的配置](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-annotation-config)：Spring2.5介绍了支持基于注册的元数据配置。
* [基于java的配置](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-java)：从Spring3.0开始，Spring JavaConfig项目成为了Spring Framework的一部分并提供了新功能。
所以你可以使用java而不是XML文件去定义你的应用类的外部Beans。使用这些新特性，请查看`@Configuration`、`@Bean`、`@Import`和`@DependsOn`这些注解。

容器管理的Spring配置必须至少有一个，通常是多个定义好的bean来组成。基于XMl的元数据配置是通过顶层的`<beans/>`元素和内部的`<bean/>`
来表示这些beans配置。基于java的配置一般是在一个使用`@Configuration`注解的类里面，对方法使用`@Bean`注解。

这些bean的定义与组成应用的实际对象是对应起来的。通常你会定义服务层，数据访问对象（Data Access Objects-DAOs），表现对象如Struts Action实例，基础对象如Hibernate SessionFactories，
JMS Queues等等。通常不会在容器配置细粒度的域对象，因为创建和加载域对象通常是DAOs和业务逻辑所负责。不过，你可以使用Spring中集成好的AspectJ来在IoC容器的控制之外来配置你的对象。
查看[在Spring中使用AspectJ去依赖注入域对象](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aop-atconfigurable)。

下面例子展示基于XML配置元数据的基础结构：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```

`id`属性是一个字符串用来标志一个单独的bean定义。`class`属性使用了全类名用来定义这个bean的类型。
id属性的值是作为协作对象来使用，上述的XML例子没有体现到这一点；查看[Dependencies](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-dependencies)获取更多信息。

### 实例化容器
实例化一个Spring IoC容器是很直截了当的。提供给ApplicationContext构造函数的位置路径实际上是字符串资源，它允许容器从外部资源例如本地文件系统、java classpath等等来加载配置元数据。
```java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```

下面例子展示了服务层对象`services.xml`配置文件：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->

    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for services go here -->

</beans>
```

下面的例子展示了数据访问对象`daos.xml`配置文件：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for data access objects go here -->

</beans>
```

在上述的例子中，服务层由`PetStoreServiceImpl`类组成，还有两个数据访问对象`JpaAccountDao`和`JpaItemDao`。
name元素引用了JavaBean的name属性,ref元素引用了其它bean定义的name。id和ref之间的联系表现了两个相互依赖的对象之间的关系。
更多关于对象之间依赖配置细节，参考[Dependencies](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-dependencies)。

#### 基于XML构成配置元数据
通过多个XML文件来定义bean是很有用的。在你的架构中通常单个XML配置文件表示了一个逻辑层或者模块。

你可以通过application context的构造函数来从这些XML片段中加载bean的定义。如上面章节所术，构造函数需要获取多个资源定位。
另外，使用一个或者多个`<import/>`元素可以从另外的XML文件中加载bean的定义，例如：
```xml
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```

在上面的例子中，从3个文件：`services.xml`，`messageSource.xml`和`themeSource.xml`中加载了外部bean的定义。
所有的定位路径都是相对于当前使用import元素的XML文件的所在位置，所以`services.xml`文件一定跟上述例子中的XML文件
在同一个目录或者在同一个编译路径，而`messageSource.xml`和`themeSource.xml`则在当前文件位置的`resources`文件夹中。
就如你所见，第一个`/`符被忽略了，鉴于路径都是相对的，所以最好还是不要使用`/`。那些被导入的内容，包括顶层的`<beans>`
元素，都必须是根据Spring Schema并是合法的XML bean定义。

```
使用相对路径"../"来引用父目录中的文件是可以的，但这并不推荐。这样做会使当前应用对外部文件造成依赖。
特别是"classpath:" URLs (for example, "classpath:../services.xml")这种情况是不推荐的，因为运行解析
过程时会选择“最近”的Classpath然后就会导致它的父目录会被添加进Classpath。Classpath配置的改变会导致目录
选择错误。

你可以使用全资源路径来代替使用相对路径，例如："file:C:/config/services.xml" 或者 "classpath:/config/services.xml"。
不过你要注意你正在把一个特定的绝对路径耦合到你的应用配置中。通常最好保持这些绝对路径的间接性，例如，使用在运行时针对
JVM系统属性解析的"${}"占位符。
```

导入指令这个功能是由beans的命名空间本身提供的。你可以通过"context"和"util"之类的Spring命名空间来使用其它指令来定义你的bean。

####  Groovy Bean定义DSL
略略略

### 使用容器
`ApplicationContext`是高级工厂的接口，能够注册和维护不同的beans和它们的依赖。使用方法`T getBean(String name, Class<T> requiredType)`
可以获取你的一个bean的实例。

`ApplicationContext`允许你通过下面的方法来读取bean的定义和访问它们：
```java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

其中比较灵活的`GenericApplicationContext`可以使用配置阅读代理进行组合：

eg:`XmlBeanDefinitionReader`用来获取XML配置
```java
GenericApplicationContext context = new GenericApplicationContext();
new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
context.refresh();
```

不同的配置阅读代理可以混合匹配使用在相同的`ApplicationContext`上，实现从不同的配置源中获取你想要的beans定义。

你可以使用`getBean`来获取bean的实例，`ApplicationContext`还有很多方法可以去获取beans，但一般来说你的应用代码不
会使用到它们，进一步来讲，你的应用代码就不应该使用`getBean`方法，因为这完全不依赖于Spring API。举个例子，Spring集成
的web framework提供了许多web组件的依赖注入例如 controllers 和 JSF-managed 的beans，允许你通过元数据来描述bean的依赖
关系（eg:使用`autowiring`注解）

## Bean概念
一个Spring IoC容器管理着一个或者多个bean，这些bean是用你所提供的配置元数据来进行创建的，例如XML中`<beans/>`节点中的定义。

在容器自身内部，`BeanDefinition`对象代表了这些bean的定义，包含了以下相关的元数据：
* 一个包含包签名的类名：通常是被定义的bean的实现类。
* Bean的行为配置元素，它声明了bean在容器中的行为（范围，生命周期回调等等）。
* Bean工作时所需要的其它的bean的引用，这些引用也被称为*collaborators*或者*dependencies*.
* 在创建新对象时生效的配置设置，例如：在管理连接池的bean中的连接数量或者池的大小限制。

这些元数据会转换成用于构成每个bean的属性。

*Table 1. The bean definition*

属性|解析
---|---
class|[Instantiating beans](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-class)
name|[Naming beans](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-beanname)
scope|[Bean scopes](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-scopes)
constructor arguments|[Dependency Injection](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-collaborators)
properties|[Dependency Injection](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-collaborators)
autowiring mode|[Autowiring collaborators](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-autowire)
lazy-initialization mode|[Lazy-initialized beans](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-lazy-init)
initialization method|[Initialization callbacks](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle-initializingbean)
destruction method|[Destruction callbacks](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle-disposablebean)

除了包含有关如何创建特定bean的信息的bean定义之外，ApplicationContext的实现还允许用户在容器外创建现有对象。
这是通过访问ApplicationContext’s BeanFactory中的`getBeanFactory()`方法，用来获取BeanFactory的实现类
`DefaultListableBeanFactory`。`DefaultListableBeanFactory`通过`registerSingleton(..)`
和`registerBeanDefinition(..)`方法来支持这种注册。不过，一般应用都是通过元数据bean定义来定义bean。

```
Bean元数据和手动提供的单例实现需要尽早的去注册，以便容器在自省步骤和自动写入时能正确的获取它们。虽然在一定程度上可以支持
覆盖已经存在的元数据和单例实例，但是在运行时（与当前访问工厂同时进行）创建新的bean是不支持的，这会导致bean容器并发访问异
常或者使得状态不一致。
```

### beans命名
每个bean都有一个或者多个身份标识，这些标识在托管bean的容器必须是唯一的。一个bean通常只一个身份标识，但如果它需要多个，
多余的标识则会被认为是别名。

在基于XML配置元数据中，你使用`id`或者`name`属性来指定bean的标识。`id`属性只允许你指定一个id。通常这些名称由字母组成
如('myBean', 'fooService'等)，但也可以包含特殊字符。如果你想给bean声明其它别名，你可以使用`name`属性，通过`,`,`;`
或者空格分隔的字符串来指定它们。历史版本中，在Spring 3.1之前，`id`属性被定义为`xsd:ID`类型，这限制了可选的字符。截
至3.1版本，`id`属性被定义为`xsd:string`类型。请注意，bean的`id`属性的唯一性由容器来强制执行而不是由XML解析器来控制。

你不需要为bean提供一个name或者id。如果没有明确的id或者name，容器会为这个bean生成一个唯一的name。不过，如果你想通过
name来引用这个bean，你必须提供一个name，通过使用`ref`元素或者[Service Locator](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-servicelocator)来引用。
如果没有提供name，可以使用[inner beans](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-inner-beans)和[autowiring collaborators](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-autowire)。

#### beans命名约定
简单来说就是请使用java字段的命名标准-驼峰命名法。

```
在classpath上扫描组件时，spring会给没有命名的组件按规范生成一个名字，本质上，会以简单类名的小写来作为名字。
```

##### 在bean定义外给bean起别名
在bean的定义中，通过组合使用定义单个名字的特定属性`id`和定义多个名字的属性`name`，你可以给bean提供一个或者多个名字。
这些名字可以等同于bean的别名，而且在一些场景下很有用，例如在一个应用中允许每个组件使用特定于该组件的bean name来引用
公共依赖项。

在bean定义的时候指定所有的别名是不够的，有时候也需要在别的地方定义bean的别名。通常，在一个大型系统中，每个子系统的配置
都是分隔开的，每个子系统都有属于自己的配置。在基于XML的配置元数据中，你可以使用`<alias/>`元素来完成这个操作。

```xml
<alias name="fromName" alias="toName"/>
```

在这个例子中，在同个容器中这个bean命名为`fromName`，然后通过使用alias定义添加别名`toName`。

举个例子，子系统A的配置元数据也许会通过名称`subsystemA-dataSource`来引用DataSource。子系统B
也许会通过名称`subsystemB-dataSource`来引用DataSource。当构成这两个子系统的主应用时，主应用
通过名称`myApp-dataSource`来引用DataSource。你可以给你的应用添加以下配置元数据来引用这3个名称。

```xml
<alias name="subsystemA-dataSource" alias="subsystemB-dataSource"/>
<alias name="subsystemA-dataSource" alias="myApp-dataSource" />
```

这样每个组件和主应用都可以通过一个唯一而且不跟别的定义冲突的名称来引用一个相同的bean。

如果你在使用`Java-configuration`，`@Bean`注解提供别名的方式请参考[这里](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-java-bean-annotation)。

### 实例化Bean
如果把创建对象比喻为做菜的话，一个bean的定义实质上是创建一个或者多个对象的菜谱。当一个bean被
引用时容器就会去菜谱找这个bean的定义，然后再使用定义中封装的配置元数据去创建或者获取一个实际
的对象。

如果你使用基于xml的配置元数据，你需要在`<bean/>`元素内的`class`定义你想初始化的对象的类型（或者class）
这个`class`属性，是`BeanDefinition`实例内部的一个`class` 属性，而且通常是强制的。你可以通过以下两种
之一的方法来使用`class`属性：

* 通常，要定义一个被构造的bean class时容器本身会直接的通过反射来调用它的构造函数来创建bean，相当于java
里的`new`操作。
* 定义一个实际的类，其包含用于调用创建对象的`static`工厂方法。这种容器通过类的静态工厂方法来创建bean不是很常见。
通过调用静态工厂方法返回的类可以是同一个class或者不同的class。

```
内部类名称
如果你配置一个静态的内部类的bean定义，你要使用这个内部类的二进制名称。
举个例子，有一个Foo类在com.example这个包中，Foo类有一个静态内部类叫Bar，那么定义bean时class属性会是这样的
com.example.Foo$Bar
注意使用$符号来分隔外部类和内部类
```

#### 通过构造函数初始化

