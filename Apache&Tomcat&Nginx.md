# Apache、Tomcat、Nginx

这几天在搭建阿里云服务器，看着镜像市场一大堆镜像眼花缭乱。之前直接用Tomcat运行web程序，由于vue刷新之后报404错误。所以需要迁移到Nginx。趁这个机会把几个服务器的特点和区别总结一下。

#### 1、首先要明确两个概念：HTTP Server和Application Server。

- **HTTP Server** 是静态服务器，关心的是 HTTP 协议层面的传输和访问控制。

  - 所以在 Apache/Nginx 上你可以看到代理、负载均衡等功能。
  - 客户端通过 HTTP Server 访问服务器上存储的资源（HTML 文件、图片文件等等）。
  - 通过 CGI 技术，也可以将处理过的内容通过 HTTP Server 分发，但是一个 HTTP Server 始终只是把服务器上的文件如实的通过 HTTP 协议传输给客户端。

- **Application Server**是动态服务器。

  - 是一个应用执行的容器。它首先需要支持开发语言的 Runtime（对于 Tomcat 来说，就是 Java），保证应用能够在应用服务器上正常运行。**比如Tomcat和Jetty。**

  - 其次，需要支持应用相关的规范，例如类库、安全方面的特性。对于 Tomcat 来说，就是需要提供 JSP/Sevlet 运行需要的标准类库、Interface 等。

  - 为了方便，应用服务器往往也会集成 HTTP Server 的功能，但是不如专业的 HTTP Server 那么强大，所以应用服务器往往是运行在 HTTP Server 的背后，执行应用，将动态的内容转化为静态的内容之后，通过 HTTP Server 分发到客户端。 

    ​

#### 2、也就是说，Apache、Nginx属于HTTP Server，Tomcat属于Application Server。



#### 3、接下来看他们详细的区别：

* Apache：Apache软件基金会下的一个项目—[Apache HTTP Server Project](https://httpd.apache.org/)。相对于Tomcat服务器来说处理静态文件是它的优势，速度快。Apache是静态解析，适合静态HTML、图片等。

* Nginx：

  * 负载均衡、反向代理、处理静态文件优势。nginx处理静态请求的速度高于apache。
  * 轻量级，同样起web 服务，比apache占用更少的内存及资源 

* Apache Tomcat：与Apache HTTP Server相比，Tomcat能够**动态**的生成资源并返回到客户端。属于**Application Server**，是一个servlet/JSP的应用容器。

  * Tomcat本身也内含了一个HTTP服务器，也可以**直接提供http服务**，通常用在内网/本地，和不需要流控等小型服务的场景。
  * Tomcat运行在JVM之上，它和HTTP服务器一样，绑定IP地址并监听TCP端口，同时还包含以下职责：
    * 管理Servlet程序的生命周期
    * 将URL映射到指定的Servlet进行处理
    * 与Servlet程序合作处理HTTP请求——根据HTTP请求生成HttpServletResponse对象并传递给Servlet进行处理，将Servlet中的HttpServletResponse对象生成的内容返回给浏览器

  ​

#### 4、虽然Tomcat也可以认为是HTTP服务器，但通常它仍然会和Nginx/Apache配合在一起 使用：

- 动静态资源分离——运用Nginx的**反向代理**功能分发请求：所有动态资源的请求交给Tomcat，而静态资源的请求（例如图片、视频、CSS、JavaScript文件等）则直接由Nginx返回到浏览器，这样能大大减轻Tomcat的压力。
  - **——反向代理（Reverse Proxy）**方式是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器。
- 负载均衡——当业务压力增大时，可能一个Tomcat的实例不足以处理，那么这时可以启动多个Tomcat实例进行水平扩展，而Nginx的负载均衡功能可以把请求通过算法分发到各个不同的实例进行处理
  - ——**什么是Servlet容器：**即实现HttpServletRequest、HttpServletResponse、HttpSession等等接口，解析http请求，通过类加载器加载对应的servlet实现类并调用。