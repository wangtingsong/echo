分享一下关于Java网络通讯方面的内容.
    下载地址：https://github.com/mldn/echo

Java基础知识：BIO、NIO、AIO三者的技术实现，以及彼此之间的区别

Netty：TCP 程序实现为主


    代码的核心：Echo程序模型，通过网络实现一个基础的Echo。


一、BIO模型：同步阻塞IO处理
    在程序的开发之中Java里面最小的处理单元就是线程，也就是说每一个线程可以进行IO的处理，在处理之中，该线程无法进行任何的其他操作。
    多线程是不可能无限制进行创造的，所以需要去考虑堆线程进行有效的个数控制。如果产生的线程过多，那么直接的问题在于，处理性能降低 ，响应的速度变慢。
    需要去区分操作系统的内核线程，以及用户线程的区别，所以最好与内核线程有直接联系，需要使用到固定线程池。
    【BIO】现在烧水，意味着你现在需要一直盯着水壶去看，一直看它已经烧为止，在这之中你什么都干不了。


二、NIO模型：同步非阻塞IO处理
   在传统的Java环境里面，最初的程序需要依赖于JVM虚拟机技术。最早的时候由于虚拟机的性能很差，所以很少有人去关注通讯的速度问题，大部分的问题都出现在了CPU处理上。
   但是随着硬件的性能提升，实际上CPU的处理速度加强了。所以从JDK 1.4开始就引入NIO的开发包，可以带来底层数据的传输性能。
   在NIO之中采用了一个Reactor事件模型，注册的汇集点Selector
  【NIO】烧水，不会一直傻站着看，你采用轮询的方式来观察水是否烧开。
  
三、AIO模型：异步非阻塞IO、是在JDK 1.7的时候才推出的模型。
    是利用了本地操作系统的IO操作处理模式，当有IO操作产生之后，会启动有一个单独的线程，它将所有的IO操作全部交由系统完成，只需要知道返回结果即可。
    主要的模式是基于操作回调的方式来完成处理的，如果以烧水为例：在烧水的过程之中你不需要去关注这个水的状态，在烧水完成后才进行通知。


之所以在整个的开发之中去使用系统实现的程序类，核心的意义在于：它可以帮助开发者隐藏协议的实现细节，但是在开发中依然会发现有如下几点：
1、程序的实现方式上的差异，因为代码的执行会有底层的实现琐碎问题；
2、在现在给定的通讯里面并没有处理长连接的问题，也就是说按照当前编写的网络通讯，一旦要发送的是稍微大一些的文件，则很大可能是无法传送完成；
3、在数据量较大的时候需要考虑粘包与拆包问题；
4、实现的协议细节操作不好控制；
5、在很多项目开发中（RCP底层实现）需要提供有大量的IO通讯的问题，如果直接使用原始的程序类，开发难度过高；

Netty的产生也是符合时代的要求，可以简化大量的繁琐的程序代码，官网：https://netty.io/

    Dubbo使用了Netty作为底层的通讯实现，Netty是基于NIO实现的，也采用了线程池的概念。Netty的最新版本为"4.1.31"，
    一开始为了进一步提高Netty处理性能，所以研发了5.0版本，但是研发之后（修改了一些类名称和方法名称、通讯算法）发现性能没有比4.x提升多少。
    所以Netty 5.x是暂时不会被更新的版本了，而现在主要更新的就是Netty 4.x。

在AIO的模型里面，会发现都采用了回调的处理形式，所以在Netty里面是基于状态的处理形式，例如：连接成功、信息的读取、失败等操作都会由一系列的状态方法来进行定义的。

Netty可以实现HTTP处理机制，但是Tomcat本身也是基于NIO的实现。

如果在实现Netty的过程之中只是进行这些简单的NIO包装处理实际上是没有任何优势的。Netty的出现是为了解决传输问题，最为重要的情况就是粘包与拆包。
1、如果现在数据稍微有点大（又不是很大的时候），那么如果要进行多条数据的发送（缓冲区有大小限制）；
2、粘包和拆包的问题解决方案有如下几种：
    A、设置消息的边界内容，例如：每一个消息使用"\n"结尾操作；
    B、使用特定消息头，在真实信息之前传入一个长度的信息。
    C、使用定长信息；
3、Netty解决拆包与粘包问题的关键在于使用了分割器的模式来进行数据的拆分。
4、Netty默认分隔符是系统提供的分隔符常亮，需要考虑分隔符的定义问题。
5、序列化管理操作：Java原生实现（性能比较差）、JSON（Restful）、MessagePack、Marshalling、AVRO、....
    netty本身直接支持有原生的 Java序列化操作，直接配置已有的程序类即可
    MessagePack：类似于JSON，但是要比JSON传输的更加小巧同时速度也快。它定义一个自己的压缩算法，例如：boolean只有true和false，但是有了MP就可以通过0和1描述了；
    Marshalling：使用JBoss实现的序列化处理操作，是基于传统的Java序列化的形式的一种升级。
    JSON：是一种标准做法，但是JSON需要清楚的问题是传输体积要大，但是传输的信息明确，本次使用FastJSON操作

HTTP程序开发：
    在进行WEB开发过程之中，HTTP是主要的通讯协议 ，但是你千万要记住一个问题，HTTP都是基于TCP协议的一种应用，HTTP是在TCP的基础上完善出来的。
    TCP是一种可靠的连接协议，所以TCP的执行性能未必会高。据说google正在开发HTTP 3.0技术标准，并且这个技术里面将使用UDP协议作为HTTP基础协议 。

HTTP里面存在有请求模式：
    A、HTTP 1.0：GET、POST、HEAD
    B、HTTP 1.1：OPTIONS、PUT、DELETE、TRACE、CONNECT
    像Restful架构里面就认为这样的请求模式的区分操作非常的明智，但是从另外一个角度来讲，这样的操作实际上会非常麻烦，所以到了Spring实现的Restful架构的时候就不推荐这样的设计了。

    HTTP在进行处理的时候操作分为两类数据内容：真实请求数据、头部信息（语言、浏览器内核、Cookie）。
    
在进行HTTP处理的时候还需要考虑到回应的操作问题：response里面给定的状态码。

Netty可以实现HTTP协议开发，但是需要注意的是，在Netty之中服务器的开发需要考虑两种不同的状态：
    处理请求：HttpRequest
    处理数据：HttpContent

处理Session与Cookie
    服务器端应该进行session的统一管理，应该建立HttpSession的操作标准
    需要设置保存在客户端的Sesson的Cookie名称。


