# JavaWeb基础

Java Web应由一组Servlet、HTML页、类、以及其他可以被绑定的资源构成。它可以在各种供应商提供的实现Servlet规范的Servlet容器中运行。

## Servlet容器
Servlet为Java Web应用提供**运行时环境**，他负责管理Servlet和JSP的生命周期，以及管理他们的共享数据。

### Tomcat
修改端口号：

	vi conf/server.xml
	<Connector port="8080" protocal="HTTP/1.1”>
	
startup.sh脚本实际调用catalina.sh启动服务

conf/tomcat-users.xml中配置tomcat管理员

	<tomcat-users>
		<role rolename="manager" />
		<user username="tomcat" password="zhang123" roles="manager" />
	
	</tomcat-users>

配置任意路径的Web应用

	
	vi conf/server.xml
	<Content path="/" docBase="" reloadable="true" />
	
从Tomcat5开始，不建议上面的做法，因为server.xml作为Tomcat的主要配置文件，在Tomcat启动后，将不会在读取这个文件，因此无法在Tomcat服务启动时发布Web应用程序。建议如下做法：

	mkdir /conf/Catalina/localhost
	cd /conf/Catalina/localhost
	touch test.xml
	vi text.xml
	<Content path="/test" docBase="/test" reloadable="true" />
	
	
### Servlet

Java Servlet是和平台无关的服务器端组件，它运行在Servlet容器中。Servlet容器负责Servlet和客户的通信以及调用Servlet的方法，Servlet和客户端的通信采用“请求/响应”的模式。
Servlet可以完成如下功能：

- 创建并返回基于客户请求的动态页面
- 创建可嵌入到现有HTML页面中的部分HTML页面
- 与其他服务器资源通信

Servlet是一个接口最关键的四个方法：

- init()
- service()
- getServletInfo()
- destroy()

在web.xml中配置Servlet的映射，向容器注册Servlet

	<servlet>
		<servlet-name>servletName</servlet-name>
		<servlet-class>com.wilson.servlet.TestServlet</servelt-class>
		<!--Servlet创建的时机-->
		<load-on-startup>1</load-on-startup>
	</servlet>
	<servlet-mapping>
		<servlet-name>servletName</servlet-naem>
		<url-pattern>/test</url-pattern>
	</servlet-mapping>
	
Note:

一个servlet可以有多个servlet-mapping映射<br>
load-on-startup可以指定Servlet被创建的时机，若为负数，则在第一次请求时被创建，若为0或正数，则在当前Web应用被容器加载时创建实例，而且数值越小，越早创建

Servlet的生命周期

- 1.构造：正在第一次请求时调用构造器，构造实例，所以Servlet是单实例
- 2.init()：只调用一次，在创建实例后，立即被调用，用于初始化当前Servlet对象
- 3.service()：服务方法，每次请求被调用
- 4.destroy()：销毁Servlet对象，在Web应用卸载前调用，释放Servlet占用的资源

Servlet容器处理客户请求的过程

- 1.Servlet引擎检查是否已经加载并创建了该Servlet实例对象，如果是直接到4；
- 2.加载并创建该servlet的对象，调用该Servlet的constructor；
- 3.调用servlet对象的init方法；
- 4.创建爱你一个用于封装请求的ServletRequest对象和一个代表响应消息的ServletResponse对象，然后调用Servlet的service方法并将请求和相应对象作为参数传递进去；
- 5.Web应用被停止或者重新启动之前，Servlet引擎将卸载Servlet并在卸载前调用destroy方法

ServletConfig对象：

封装了Servlet的配置信息，并且可以获取ServletContext对象

- 1.配置servlet的初始化参数

		<servlet>
			<init-param>
				<param-name>test</param-name>
				<param-value>123</param-value>
			</init-param>
		</servlet>
		
- 2.获得初始化参数，getInitParameter（String name）,获取指定参数名的初始化参数，getInitParameterNames():获取参数名组成的Enumeration对象


ServletContext对象 <br>

- 1.Servlet引擎为每个Web应用都创建了一个ServletContext对象，ServletContext对象被包含在ServletConfig对象中，调用ServletConfig.getServletContext方法可以返回该对象的引用

- 2.application对象，代表当前Web应用，可以获取到当前应用的各个信息
- 3.获取当前Web应用的初始化参数 getInitParamter（String name）,getInitParameterNames()

		<!--在web.xml中配置Web的初始化参数-->
		<context-param>
			<param-name></param-name>
			<param-value></param-value>
		</context-param>

- 4.获取当前Web应用的某一个文件的绝对路径：getRealPath（String path）
- 5.获取当前Web应用的某一个文件对应的输入流：getResourceAsStream（String path），path参数为相对于当前Web应用的相对路径
- 6.和Attribute相关的几个方法比较常用

在Servlet中获取请求信息

- 1.Servlet的service方法用于处理客户端请求，在这个方法中获取请求参数并进行处理

		public void service(ServletReqeust request, ServletResponse response) throws ServletException, IOException
		
- 2.请求信息被封装在ServletRequest对象中
	String getParameter(String name):可以获取参数名的值
	String [] getParameter(String name):参数有多个值的时候用这个方法
	Enumeration getParameterNames():参数的Enumeration对象 
	Map getParameterMap():获取参数的键值对
	
	HttpServletRequest对象是ServletRequest的子接口，在这个接口中会有HTTP相关的请求信息，如URL等
	
ServletResponse响应对象

- 1.getWriter()返回一个PrintWriter对象，调用该对象的print方法可以把内容打印到客户的浏览器上
- 2.setContentType("application/json"):设置响应内容格式
- 3.HTTP相关的信息在其子接口HttpServletResponse对象中

**Servlet的一个实现是GenericServlet，而开发时一般直接继承HttpServlet，该类是GenericServlet的一个子类，通过封装，使得开发更为便利，同样的ServletRequest和ServletResponse也有相应的实现HttpServletRequest和HttpServletResponse**



### HTTP协议的GET和POST
HTTP的请求头：

	GET /books/java.html HTTP 1.1       --请求行
	Accept: */*
	Accept-Language:en-us
	Connection:Keep-Alive
	Host:localhost
	Refer:http://localhost/links.asp
	User-agent:Mozilla/4.0              --多个消息头
	Accept-Encoding:gzip,defalte
										--一个空行
										
GET和POST的主要区别在于GET请求的参数会直接附在URL后面，POST请求则会封装在formData或者payload中





										








	
	 
