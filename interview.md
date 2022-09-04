# 简历详解

## 自我介绍

面试官您好，我叫张辉，是一名23届的应届生，专业是软件工程。

目前是在一家公司做Java实习生的工作，主要负责的内容是数据的清洗和转换。

自己也做过一些项目，有Java的也有Go的，在开发中都是选择项目中的难点来开发并以此锻炼自己。

自己本人是非常热爱技术，对软件开发和底层实现都有着浓厚的兴趣，热衷于阅读各种技术书籍，喜欢同其他人探讨。

我大学的专业是软件工程，在校期间对计算机基础知识有一定的掌握，然后自学的java开发，基本上所有的知识点都是自己学习然后思考总结的。
我注重实践和理论相结合，学习基础的同时也没有落下开发的技能。在学校期间跟随老师做过一些商业的项目，对开发技术有一定的掌握。还有就是自己对热爱技术，软件开发有自己的理解，热爱阅读技术书籍，喜欢探索知识，不满足与网上的博客，对知识点有自己的理解。

以上就是我的自我介绍，谢谢面试官。



## 项目

### 极简版抖音

#### 项目介绍

一句话介绍，实现极简版抖音。每一个应用都是从基础版本逐渐发展迭代过来的，希望同学们能通过实现一个极简版的抖音，来切实实践课程中学到的知识点，如Go语言编程，常用框架、数据库、对象存储等内容，同时对开发工作有更多的深入了解与认识，长远讲能对大家的个人技术成长或视野有启发。

#### 涉及技术

Go、Gin、Gorm、OSS、FFmpeg、Token、MySql、Redis（不是我做的不太了解）

#### 我所做的

1 使用OSS和FFmpeg完成用户投稿的功能，通过使用协程的异步优化此流程使得响应时间从原来的20+s 降低到了5~7s。 

2 使用Viper做反序列化，实现配置文件的读取,完成配置的初始化工作。

3 使用Token作为拦截器，完成对非法请求的拦截和过滤的功能。

4  完成了用户模块、评论模块、视频投稿模块和基本信息模块的代码的编写和功能的实现。 

#### 原理分析

##### 一  优化问题

###### OSS

首先就是OSS，OSS全程是Object Storage Service（对象存储服务），具有易扩展、易用（支持RestFul，而传统的HDFS是支持TCP私有协议的，如果想使用HDFS还需要再代码中内嵌一个客户端才行）、便宜等特点，适合存储海量数据。

所以我们就选用了OSS对象存储服务。

###### FFmpeg

FFmpeg是用来做视频处理的一个工具，我这里就是使用ffmpeg实现了视频截取封面的功能。

###### 流程优化

下面来聊一下我是怎么做的优化。

原来的流程是这样的 

1 用户在客户端调用接口上传视频

2 服务端接收到用户上传的视频数据调用oss上传文件的接口完成视频的上传、视频上传完成，服务端得到视频的url。

3 以url为参数，调用ffmpeg完成封面图的截取并存储在本地、然后调用oss文件上传接口完成封面图的上传，服务端得到封面图的url。

4 至此，用户所有数据都已经上传到oss，服务端调用数据库存储接口完成将用户信息、视频和封面图存入MySql。

5 返回响应的数据告知客户端上传完成

使用这一套流程在测试的时候接口的响应时间平均在15s+以上，客户端时常会收到请求响应超时的错误，后来下定决心优化流程，优化过程中使用了Go的协程，优化后的效果非常好，接口的响应时间平均在5s+。下面看下优化后的流程

1 用户在客户端调用接口上传视频

2 提前定义好封面图的url，用于后面的异步操作

3 服务端接收到用户上传的视频数据调用oss上传文件的接口完成视频的上传、视频上传完成，服务端得到视频的url。

4 以url为参数，开启一个协程调用ffmpeg完成封面图的截取并存储在本地、然后调用oss文件上传接口完成封面图的上传，服务端得到封面图的url。

5 不需要等步骤4完成，直接调用数据库存储接口完成将用户信息、视频和封面图存入MySql。

6 返回响应的数据告知客户端上传完成。

##### 二 序列化问题

[序列化相关文章](https://zhuanlan.zhihu.com/p/40462507)

**序列化：**把对象转化为可传输的字节序列过程称为序列化。

**反序列化：**把字节序列还原为对象的过程称为反序列化。

其实序列化最终的目的是为了对象可以**跨平台存储，和进行网络传输**。而我们进行跨平台存储和网络传输的方式就是IO，而我们的IO支持的数据格式就是字节数组。

因为我们单方面的只把对象转成字节数组还不行，因为没有规则的字节数组我们是没办法把对象的本来面目还原回来的，所以我们必须在把对象转成字节数组的时候就制定一种规则**（序列化）**，那么我们从IO流里面读出数据的时候再以这种规则把对象还原回来**（反序列化）。**

默认的序列化ID和声明的ID是有区别的，如果使用默认的ID，当类属性发送变化的时候，反序列化就会失败，而有时候我们想做到兼容，所以推荐使用声明的ID，在这种情况下，可以做到版本的兼容，即使类文件发送的改变，序列化ID是显示指定的，从而不会导致反序列化失败。

##### 三 JWT问题

[Cookie、Session、Token 视频](https://www.bilibili.com/video/BV1ob4y1Y7Ep?spm_id_from=333.337.search-card.all.click&vd_source=ed7e751a69384a4f5cb20556664b84d4)

[腾讯云社区文章 Cookie Session Token](https://cloud.tencent.com/developer/article/1542456)

[Session的创建过程](https://www.cnblogs.com/woshimrf/p/5317776.html)

token和jwt是区别的 

token实际上是一种加强过的SessionId

为什么说是加强呢？是因为token不需要存储在服务器端，具有这个优势，所以说是加强

但是还有一个问题，就是token里面的数据比较少，可能需要查库从而获取数据

那为什么不在token里面放上数据呢？ 这里是安全性的考虑，如果直接放明文数据的话，会造成数据安全的问题。

那么有没有一种办法解决呢？

当然，加密就行了，这就是jwt所作的事情。

[jwt和token的区别案例](https://www.icode9.com/content-4-886449.html)

jwt比Session好在哪里？ 好在了不需要存储，但是多了解密这一步，也就是常说的时间换空间

jwt比Cookie好在哪里？一个是安全性，一个是简洁性（即没有跨域限制）

[跨域相关问题](https://www.cnblogs.com/kgwei520blog/p/13667378.html)



### 智慧路灯系统

#### 项目介绍

是⼀个完整的前后端分离的以路灯为基础的物联⽹系统，路灯上可以安装包括但不限于摄像头、报警器、⼴播、⼤屏等设备。

主要功能是通过智慧路灯物联⽹平台完成在⽹⻚端对路灯的控制,进⽽完成对以上设备的控制。

此系统通过技术⼿段解决了⽤户管理多个设备复杂性，通过⼀个系统就可以完成对所有设备的控制。

#### 涉及技术

Java springboot mybatis MySql  Go Redis webrtc

#### 我所做的

1 使⽤Go语⾔完成⽹⻚端⽆插件直播功能的开发，采⽤了webrtc技术优化视频的播放,使⽤户可以在⽹⻚ 端以极低延迟的感受观看直播和视频的回放。 

2 完成云台控制后台的开发，实现在⽹⻚端完成对摄像头的控制，使⽤Java语⾔完成基于sdk的二次开发。 

3 使用redis作为缓存系统,使得首页的响应速度提高了大约30%。

#### 原理分析

##### 一 RTSP

rtsp，英文全称 Real Time Streaming Protocol，RFC2326，实时流传输协议，是TCP/IP协议体系中的一个应用层协议！协议主要规定定了一对多应用程序如何有效地通过IP网络传送多媒体数据。RTSP体系结位于RTP和RTCP之上（RTCP用于控制传输，RTP用于数据传输），使用TCP或UDP完成数据传输！
[RTSP交互流程](https://blog.csdn.net/sinat_36002055/article/details/123801585)

[RTSP操作大华摄像头](https://blog.csdn.net/lbjbs/article/details/117455605)

流程如下

前端js与Go的服务器通过http交互，完成RTSP的交互流程

这里Go的服务器中内置了一个RTSP的Client，使用RTSP协议完成流媒体数据的获取

这里实际上和webrtc没有半毛钱关系，使用的其实是rtsp协议去做的视频消息的传输。

但还是要比ffmpeg去做这个效果来的快得多，而且延迟也大大的小于ffmpeg。

##### 二 云台控制相关问题

使用的websocket长连接去做的。 [websocket](https://zh.m.wikipedia.org/zh-hans/WebSocket)

这里考虑用websoket主要是考虑到服务端主动给客户端发消息。

服务端收到前端传递过来的参数后调用sdk完成对摄像头的控制，没有考虑并发问题。

##### 三 Redis提速

将一些复杂的数据存入到redis中，不用每次都去数据库查询，极大的提高了首页的打开的速度。



## 技术问题

### 金康赛力斯（北京）



#### Spring IOC讲一讲，跟DI有什么区别

```java
IOC是控制反转，把对象的创建和对象之间的调用过程，交给Spring来管理，而不是使用new的方式写死在代码中，降低了系统的耦合度。
    
IOC的底层原理由三部分组成  xml，工厂模式和反射
我们一步步的来说，凡事都是从易到难的，spring最初的设计者恐怕也不会一开始就想到要使用上面的三部分去解决耦合度的问题。我们下面举一个例子去说明这个问题。
假设我们现在有一个UserService和一个UserDao，这两个类分别有doService()和doDao() 两个方法。
如果我们需要在UserService中的doService()方法中调用UserDao中的doDao()方法。我们可以怎么做呢？
传统的方法是在doService()中创建一个UserDao的对象，然后调用doDao()方法。
这样做当然可以实现需求，但是耦合度很高。为什么呢？
因为我们doService()中的代码是强依赖于UserDao的构造方法的，一旦UserDao的构造方法改变，那么我所有强依赖它的地方的代码都需要修改。
为了消除这个强依赖，我们不得以给代码加一层，优化代码首选的就是设计模式了。
我们先来看我们的问题在哪里，在我们原来的代码中，doService()方法是创建一个UserDao对象，然后使用对象的方法，这两步是连在一起的，我们能不能找到一种方法帮助我们创建对象，不需要我们再关心对象的创建呢？我们只需要拿着创建好的对象去调用方法就可以了。
相信你心中应该有答案了，这不就是工厂模式的特点嘛，创建和使用分离。
我们写一个工厂，让工厂负责我们对象的创建，我们在doService()中使用工厂类的方法帮助我们完成对象的创建。
我们默认工厂的方法是不会轻易的改变的，就算UserDao的构造方法发生了改变，我们只需要改变工厂中创建对象的方法，其他的就都不需要改变了。
    
上面是我们自己实现的IOC，因为我们已知了UserDao对象，所以我们可以写一个UserDao的工厂，然后在工厂中去new UserDao的对象。但是我们站在Spring的角度，这个工厂应该怎么去写，Spring并不知道一个应用中到底需要什么对象，它需要使用者去告诉它，怎么告知它呢？当然是我们的配置文件啦。在Spring启动的时候读取配置文件，然后通过运行时创建对象（也就是反射）完成控制反转的实现。

IOC思想基于IOC容器完成，IOC容器底层就是对象工厂
Spring提供了两种IOC容器的实现方式
1 BeanFactory:IOC容器基本实现，是Spring的内部使用的接口，不提倡开发人员使用，加载配置文件的时候不会创建对象，在获取对象的时候才会创建对象。
2 ApplicationContext:BeanFactory接口的子接口，提供更强大的功能，提倡开发人员使用，加载配置文件的时候就完成了对象的创建

为什么不提倡使用BeanFactroy,虽然它懒加载对象节约内存，但是对于服务器来说，我们更希望对象在服务启动的时候就完成对象的创建，相比于提高用户的响应速度来说，我们并不缺少那些内存。
    
DI的意思是依赖注入，直白点也就是对象成员变量的注入。DI需要依赖于IOC，只有一个对象交给Spring管理，并且完成对象的创建后，这个对象才可以进行DI。


```

#### Spring有几种类型的bean，有什么区别？

```java
Spring有两种类型的bean，一种是普通类型的bean，另一种是工厂类型的bean（FactoryBean）
1 普通bean:也就是我们平时使用的bean，配置文件中定义的bean类型就是返回类型
2 FactoryBean:在配置文件中定义的类型可以和返回类型不一样。
实现方法如下
第一步创建类，让这个类作为工厂bean，实现接口FactoryBean
第二步 实现接口的方法，在实现的方法中定义返回bean的类型

```

#### Spring中bean的作用域了解吗？

```java
1 在Spring中可以设置bean实例是单实例还是多实例 
2 在默认情况下是单实例，也就是多次创建的对象都是同一个对象
3 可以使用scope属性设置 
    第一个值singleton 默认就是，表示单实例
    第二个值prototype 表示多实例
    第三个值request 表示每次请求创建一个对象，相同的请求不会创建对象
    第四个值session 表示每次会话过程创建一个对象，相同的会话不会创建对象
  
```

#### Spring中bean的生命周期讲一下

```java
Spring中bean的生命周期粗略的分有五步，详细的分有七步
第一步 默认执行无参数的构造方法创建对象
第二步 通过set方法为bean的属性设置值
第三步 把bean实例和bean的名称传给后置处理器类的方法（需要自己创建后置处理器类并交给Spring管理）
第四步 调用bean的初始化方法（需要自己创建方法并进行配置）
第五步 把bean实例和bean的名称传给后置处理器类的方法（需要自己创建后置处理器类并交给Spring管理）
第六步 可以使用bean了
第七步 当容器关闭时，调用bean销毁的方法（需要自己创建并配置）

```

#### Java中的注解了解吗

```java
java中的注解是jdk1.5后引入的一个新特性，也成为元数据（即描述数据的数据），是一种代码级别的说明，与类，接口，枚举是在同一个层次。它可以声明在包，类，字段，局部变量，方法参数等前面。
jdk中预定义的一些注解有@Override @Deprecated @SuppressWarnings 
注解主要分为两部分，元注解和public @interface 注解名称(){里面可以写一些无参数的方法，用于接收注解的一些参数} 本质上是一个接口，继承于java.lang.annontion.Annontion{} 接口
元注解是一些jdk内置的一些注解，用于生成文档或者标注过期等。

```



#### Spring中的注解有哪些？

```java
创建对象的注解
@Component  作用于通用组件层
@Controller 作用于web层
@Service	作用于业务层
@Repository 作用于dao层
在Spring配置文件中开启注解扫描后，Spring在启动的时候就会扫描指定包下的类上有没有上述的注解，如果有的话就解析注解，完成对象的创建。

    
属性注入的注解
@Autowired:根据对象的类型进行注入，如果一个接口有多个实现类，就需要下面的注解。
@Qualifier:根据对象的名称进行注入，需要配合上面的注解进行使用，否则会报错。
@Resource:既可以根据对象的类型进行注入，也可以根据对象的名称进行注入 有一个name属性可以进行设置需要注入对象的名称，不设置默认按照类型注入。这里要提一点这个注解不是Spring官方提供的注解，是javax中的，Spring不建议使用。
@Value:注入基本类型的属性，字符串，Integer等。
使用上述这些注解不需要再手写set方法，这些注解本质上就可以帮助我们生成set方法，并完成属性的注入

    
    
```



#### java反射了解吗，详细说一下反射，反射有什么用途？

```java
反射机制允许程序在执行期间借助于反射的API取得任何类的内部信息（比如成员变量，构造器，方法等信息），并能操作对象的属性及方法，反射在框架的底层都会用到
在加载完类之后，在堆中就产生了一个Class类型的对象，一个类只有一个Class对象，这个对象包括了类的完整结构信息，通过这个对象得到类的结构。
java反射可以做到
1 在运行时判断任何一个对象所属的类
2 在运行时构造任何一个类的对象
3 在运行时得到任意一个类的方法和成员变量
4 生成动态代理
   	
```

#### aop了解吗，详细说一下

```java
aop是面向切面编程，通过预编译和运行时动态代理的方式实现程序功能的一种技术。利用aop可以对业务的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性。
通俗来讲就是在不修改源代码的情况下，在主干中添加新的功能。

aop底层使用的是动态代理，有两种情况的动态代理，对于要增强的类来说，分为有接口和无接口两种情况。有接口使用的是jdk的动态代理，无接口使用的是cglib的动态代理。
aop相关术语
1 连接点 类中可以被增强的方法
2 切入点 实际上类被增强的方法
3 通知   实际增强的逻辑部分称为通知，通知有多种类型。
4 切面   是一个动作，把通知应用到切入点的过程称为切面

在Spring中一般使用aspectj来完成aop的操作，并不是Spring的一部分，而是独立的aop框架。

```

#### 事务了解吗，Spring中的事务有什么特点，是怎么做的

```java
事务是数据库操作的基本单元，逻辑上的一组操作，要么全部成功，要么全部失败。
事务有四大特性，如下
原子性:原子性是指事务是一个不可分割的工作单位，事务中的操作要么全部成功，要么全部失败。比如在同一个事务中的SQL语句，要么全部执行成功，要么全部执行失败。
一致性:事务必须使数据库从一个一致性状态变换到另外一个一致性状态。
隔离性:事务的隔离性是多个用户并发访问数据库时，数据库为每一个用户开启的事务，不能被其他事务的操作数据所干扰，多个并发事务之间要相互隔离。
持久性:持久性是指一个事务一旦被提交，它对数据库中数据的改变就是永久性的，接下来即使数据库发生故障也不应该对其有任何影响。

 Spring的事务管理有两种
 1 编程式事务（采用硬编码的方式，不推荐）
 2 声明式事务（可以使用xml或者注解实现，推荐注解）
 
 在Spring使用声明式事务管理的时候，底层使用的就是aop原理
 
 使用Spring声明式事务的步骤
 1 在配置文件中配置事务管理器并引入数据源
 2 在配置文件中开启事务注解
 3 推荐在service层的类上加上@Transactional注解即可
 
 事务的传播行为
 当一个事务的方法被另一个事务的方法调用时，这个事务的方法应该如何执行。
 https://blog.csdn.net/edward0830ly/article/details/7569954（传播行为）
```

#### SpringMVC包括什么

<img src="C:\Users\86198\AppData\Roaming\Typora\typora-user-images\image-20220904111652397.png" alt="image-20220904111652397" style="zoom:67%;" />



#### 什么是MVC，什么是SpringMVC？

```java
MVC是一种软件架构的思想，将软件按照模型，视图和控制器来划分。
M:模型 V:视图  C:控制层
MVC工作流程如下
用户通过视图层发送请求到服务器，在服务器中的C层接收请求，C层调用Service层进行处理并封装到M层，然后C层根据M的数据渲染V层，最后返回给浏览器。

```



#### 如何配置SpringMVC的前端控制器DispatcherServlet

```java
a>默认配置方式

此配置作用下，SpringMVC的配置文件默认位于WEB-INF下，默认名称为\<servlet-name>-servlet.xml，例如，以下配置所对应SpringMVC的配置文件位于WEB-INF下，文件名为springMVC-servlet.xml
 
<!-- 配置SpringMVC的前端控制器，对浏览器发送的请求统一进行处理 -->
<servlet>
    <servlet-name>springMVC</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>springMVC</servlet-name>
    <!--
        设置springMVC的核心控制器所能处理的请求的请求路径
        /所匹配的请求可以是/login或.html或.js或.css方式的请求路径
        但是/不能匹配.jsp请求路径的请求
    -->
    <url-pattern>/</url-pattern>
</servlet-mapping>
 
b>扩展配置方式

可通过init-param标签设置SpringMVC配置文件的位置和名称，通过load-on-startup标签设置SpringMVC前端控制器DispatcherServlet的初始化时间
<!-- 配置SpringMVC的前端控制器，对浏览器发送的请求统一进行处理 -->
<servlet>
    <servlet-name>springMVC</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!-- 通过初始化参数指定SpringMVC配置文件的位置和名称 -->
    <init-param>
        <!-- contextConfigLocation为固定值 -->
        <param-name>contextConfigLocation</param-name>
        <!-- 使用classpath:表示从类路径查找配置文件，例如maven工程中的src/main/resources -->
        <param-value>classpath:springMVC.xml</param-value>
    </init-param>
    <!-- 
 		作为框架的核心组件，在启动过程中有大量的初始化操作要做
		而这些操作放在第一次请求时才执行会严重影响访问速度
		因此需要通过此标签将启动控制DispatcherServlet的初始化时间提前到服务器启动时
	-->
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>springMVC</servlet-name>
    <!--
        设置springMVC的核心控制器所能处理的请求的请求路径
        /所匹配的请求可以是/login或.html或.js或.css方式的请求路径
        但是/不能匹配.jsp请求路径的请求
    -->
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

#### RequestMapping注解了解吗

```java
### 1、@RequestMapping注解的功能

从注解名称上我们可以看到，@RequestMapping注解的作用就是将请求和处理请求的控制器方法关联起来，建立映射关系。

SpringMVC 接收到指定的请求，就会来找到在映射关系中对应的控制器方法来处理这个请求。
    

### 2、@RequestMapping注解的位置

@RequestMapping标识一个类：设置映射请求的请求路径的初始信息

@RequestMapping标识一个方法：设置映射请求请求路径的具体信息
    
### 3、@RequestMapping注解的value属性

@RequestMapping注解的value属性通过请求的请求地址匹配请求映射

@RequestMapping注解的value属性是一个字符串类型的数组，表示该请求映射能够匹配多个请求地址所对应的请求

@RequestMapping注解的value属性必须设置，至少通过请求地址匹配请求映射
### 4、@RequestMapping注解的method属性

@RequestMapping注解的method属性通过请求的请求方式（get或post）匹配请求映射

@RequestMapping注解的method属性是一个RequestMethod类型的数组，表示该请求映射能够匹配多种请求方式的请求

若当前请求的请求地址满足请求映射的value属性，但是请求方式不满足method属性，则浏览器报错405：Request method 'POST' not supported
    
注：

1、对于处理指定请求方式的控制器方法，SpringMVC中提供了@RequestMapping的派生注解

处理get请求的映射-->@GetMapping

处理post请求的映射-->@PostMapping

处理put请求的映射-->@PutMapping

处理delete请求的映射-->@DeleteMapping

2、常用的请求方式有get，post，put，delete

但是目前浏览器只支持get和post，若在form表单提交时，为method设置了其他请求方式的字符串（put或delete），则按照默认的请求方式get处理

若要发送put和delete请求，则需要通过spring提供的过滤器HiddenHttpMethodFilter，在RESTful部分会讲到

### 5、@RequestMapping注解的params属性（了解）

@RequestMapping注解的params属性通过请求的请求参数匹配请求映射

@RequestMapping注解的params属性是一个字符串类型的数组，可以通过四种表达式设置请求参数和请求映射的匹配关系

"param"：要求请求映射所匹配的请求必须携带param请求参数

"!param"：要求请求映射所匹配的请求必须不能携带param请求参数

"param=value"：要求请求映射所匹配的请求必须携带param请求参数且param=value

"param!=value"：要求请求映射所匹配的请求必须携带param请求参数但是param!=value

 ### 6、SpringMVC支持路径中的占位符（重点）

原始方式：/deleteUser?id=1

rest方式：/deleteUser/1

SpringMVC路径中的占位符常用于RESTful风格中，当请求路径中将某些数据通过路径的方式传输到服务器中，就可以在相应的@RequestMapping注解的value属性中通过占位符{xxx}表示传输的数据，在通过@PathVariable注解，将占位符所表示的数据赋值给控制器方法的形参
案例
@RequestMapping("/testRest/{id}/{username}")
public String testRest(@PathVariable("id") String id, @PathVariable("username") String username){
    System.out.println("id:"+id+",username:"+username);
    return "success";
}
//最终输出的内容为-->id:1,username:admin
```





mysql事务隔离级别了解吗

tcp为什么要三次握手，两次不行吗

linux下查询指定端口用什么命令

linux下查询日志的命令呢

聊一聊你的项目主要做了什么

集合用的多吗，项目中集合用来做什么





### 星辰物联网（北京）

java多线程实现方式有几种

java的lambda表达式是什么

java涉及到io的类有哪些

springboot启动类是什么，用的哪一个注解

java多线程传递消息怎么做

jvm垃圾回收算法有哪些，区别是什么

怎么区别新生代和老年代

策略模式怎么去写

mysql分页查询怎么用

mysql索引分类有哪些

redis常见的数据类型有哪些

选择排序和冒泡有什么不一样吗

消息队列kafka怎样保证数据不丢的

git常用操作



### 火树科技（杭州）

实习工作详解

es和lussen内存布局有什么区别

单例线程安全的操作

jvm内存区域

工厂的几种写法

nio对比io有什么优势



### 奇安信

Java线程池详解

tcp握手的序号问题  ack的计算规则

子网划分问题 C类地址

统计log文件中某一个字符出现的次数

hashmap的问题 是否有序

mysql的s锁和x锁

函数中的final字段在哪一个区  final字段的含义有什么

粘包问题怎么解决

进程让出cpu后处于什么状态

mysql联合索引的特性

排序算法 是否稳定

lru算法

合并区间问题

全排列



### 京东

final的用法

mysql增加多个字段

String.format的用法

ls列出隐藏文件

osi数据链路层的作用

String的spilt和indexof的性能

线程池 submit和execute有什么区别

linux中系统库文件存放的目录

java权限修饰符

lsd和msd

二叉树的遍历问题

short的比较 参考Integer的比较

大字符串的创建 StringBuf和StringBuilder有什么区别

### 巨峰科技（杭州）

反射

GC线程

重写和重载

异常体系

abstract关键字

string和int转换

8大基本数据类型

字符流和字节流

java有内存泄漏吗



### 搜狐

中断

sleep和yield

指令周期

排序算法

二叉树遍历问题

int num= 999 int num =9_9_9 int num = _999

finalize()用法

new String（） 和 直接赋值的String比较

数组初始化

sleep和wait

instanc of

Threadlocal

类加载器

ArrayList扩容

函数式接口

jvm参数

G1和cms

jvm内存屏障

异或左移运算符

跨域

ipc类地址

类型占字节数

DNS

死锁

linux负载怎么查看

斐波那契数列









