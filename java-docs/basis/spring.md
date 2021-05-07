## Spring

### 1. 简介
> Spring是一个开发应用框架
> <br>轻量级、非侵入式、一站式、模块化，其目的是用于简化企业级应用程序开发。
> <br>应用程序是由一组相互协作的对象组成，而在传统应用程序开发中，一个完整的应用是由一组相互协作的对象组成。
> <br>开发一个应用除了要开发业务逻辑之外，最多的是关注如何使这些对象协作来完成所需功能，而且要低耦合、高内聚。业务逻辑开发是不可避免的，那如果有个框架出来帮我们来创建对象及管理这些对象之间的依赖关系。

### 2. 核心组件
> #### 2.1 核心组件关系
>> Bean：包装的是Object数据   
>> Context：发现每个 Bean 之间的关系，为它们建立这种关系并且要维护好这种关系。所以 Context 就是一个 Bean 关系的集合，这个关系集合又叫 Ioc 容器
>> <br>Core：发现、建立和维护每个 Bean 之间的关系所需要的一些列的工具，Core 这个组件叫 Util 更容易理解
>> ![](https://i.loli.net/2021/05/06/8sGHf4pUZ1brhKT.png)
> #### 2.2 Bean
>>     包装的是Object数据
>>     Bean 组件在 Spring 的 org.springframework.beans 包下, 这个包下的所有类主要解决了三件事：Bean 的定义、Bean 的创建以及对 Bean 的解析
>>     Spring Bean 的创建时典型的工厂模式，他的顶级接口是 BeanFactory
>>     BeanFactory 有三个子类：ListableBeanFactory、HierarchicalBeanFactory 和 AutowireCapableBeanFactory
>>     ListableBeanFactory：表示这些 Bean 是可列表的
>>     HierarchicalBeanFactory： 表示的是这些 Bean 是有继承关系的，也就是每个 Bean 有可能有父 Bean
>>     AutowireCapableBeanFactory：定义 Bean 的自动装配规则
>>     DefaultListableBeanFactory：为最终的默认实现类 ，他实现了所有的接口。它主要是为了区分在 Spring 内部在操作过程中对象的传递和转化过程中，对对象的数据访问所做的限制
>>     这四个接口共同定义了 Bean 的集合、Bean 之间的关系、以及 Bean 行为
>>     下图是这个工厂的继承层次关系图和bean的生命周期图
>> ![](https://i.loli.net/2021/05/06/PrWGJtI1oS47lze.png)
>> ![](https://i.loli.net/2021/05/07/3VQWcitzbdy8GgI.png)
> #### 2.3 Context
>>     Context 在 Spring 的 org.springframework.context 包下，他实际上就是给 Spring 提供一个运行时的环境，用以保存各个对象的状态  
>>     ApplicationContext 是 Context 的顶级父类，他除了能标识一个应用环境的基本信息外，他还继承了五个接口，这五个接口主要是扩展了 Context 的功能
>>     ApplicationContext 继承了 BeanFactory，这也说明了 Spring 容器中运行的主体对象是 Bean
>>     ApplicationContext 继承了 ResourceLoader 接口，使得 ApplicationContext 可以访问到任何外部资源
>>     ConfigurableApplicationContext 表示该 Context 是可修改的，也就是在构建 Context 中用户可以动态添加或修改已有的配置信息
>>     WebApplicationContext 顾名思义，就是为 web 准备的 Context 他可以直接访问到 ServletContext
>>     ApplicationContext 必须要完成以下几件事: 标识一个应用环境, 利用 BeanFactory 创建 Bean 对象, 保存对象关系表, 能够捕获各种事件
>>     Context 作为 Spring 的 Ioc 容器，基本上整合了 Spring 的大部分功能，或者说是大部分功能的基础
>>     下面是 Context 的类结构图
> ![](https://i.loli.net/2021/05/07/othGbigkAI2ScsU.png)
> #### 2.4 Core
>>     Core 组件作为 Spring 的核心组件，其中一个重要组成部分就是定义了资源的访问方式。这种把所有资源都抽象成一个接口的方式很值得在以后的设计中拿来学习  
>>     Resource 接口封装了各种可能的资源类型，也就是对使用者来说屏蔽了文件类型的不同    
>>     Resource 接口继承了 InputStreamSource 接口，这个接口中有个 getInputStream 方法，返回的是 InputStream 类。这样所有的资源都被可以通过 InputStream 这个类来获取，所以也屏蔽了资源的提供者       
>>     ResourceLoader 接口屏蔽了所有的资源加载者的差异，只需要实现这个接口就可以加载所有的资源，他的默认实现是 DefaultResourceLoader       
>>     Context 是把资源的加载、解析和描述工作委托给了 ResourcePatternResolver 类来完成，他相当于一个接头人，他把资源的加载、解析和资源的定义整合在一起便于其他组件使用       
> ![](https://i.loli.net/2021/05/07/BthMiDlcR9woVH2.png)     