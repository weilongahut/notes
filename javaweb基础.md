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
- 4.创建一个用于封装请求的ServletRequest对象和一个代表响应消息的ServletResponse对象，然后调用Servlet的service方法并将请求和相应对象作为参数传递进去；
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

---

## JSP概述

Java Server Pages，为了简化Servlet进行的一种扩展，在HTML页面中直接写Java代码，本质上JSP也是一个servlet，会被翻译成java文件并编译成Class文件。
hello.jsp会被编译成如下代码：
	
	//omitted...
	public final class hello_jsp extends org.apache.jasper.runtime.HttpJspBase implements org.apache.jasper.runtime.JspSourceDependent{
	//omitted
	}

从上面的代码可以看出JSP默认继承了HttpJspBase并扩展JspSourceDependent接口

	<%= 在这里写Java代码%>
	
#### JSP的9个隐含对象：

**Response，Resquest， pageContext， session
, application, config, out, page, exception**

已经在JSP中声明，直接可以使用

- request， HttpServletRequest；
- response, HttpServletResponse;
- pageContext, PageContext对象，可以获取到当前页面的几乎所有信息，包含其他8个隐含对象，在设计自定义标签时会用到这个对象；
- session, 浏览器和服务器的一次会话，HttpSession对象，这个很重要
- application，代表当前WEB应用，是ServletContext对象；
- config， 当前JSP对应的Servlet的ServletConfig对象,如果需要访问JSP的初始化参数，需要通过映射的地址才可以
		
		//映射jsp文件的方式
		<servlet>
			<servlet-name>hellojsp</servlet-name>
			<jsp-file>/hello.jsp</jsp-file>
			<init-param>
				<param-name> test</param-name>
				<param-value> testValue</param-value>
			</init-param>
		</servlet>
		
		<servlet-mapping>
			<servlet-name>hellojsp</servlet-name>
			<url-pattern>/hellojsp</url-pattern>
		</	servlet-mapping>	
- out，JSPWriter，out.print（）方法直接将字符打印到浏览器上，注意此处的println()方法不会换行，需要在jsp内插入<br>换行
- page, 当前JSP对象的servlet对象的引用，为Object对象
- exception，异常对象，Exception,在error页面上使用
	<%@ page isErrorPage=“true” %>

 
#### JSP的基本语法


- JSP模板元素：JSP页面中的静态HTML代码
- JSP表达式，
	
		<!-- JSP表达式 -->
		<%= date %>

- 脚本片段：在HTML中嵌入的Java代码，<% %>
- JSP声明：<%! %>，用于声明JSP对象的方法，JSP代码默认会在service（）方法体中，是用声明则可以声明独立的方法
- JSP注解：
	- <%-- JSP注释 --%>
	- <! -- HTML注释 -->

#### 域对象属性操作

request，pageContext，session， application属性相关的方法

- Object getAttribute(String name): 获取指定的属性
- Enumeration getAttributeNames(): 获取所有的属性的名字组成的Enumeration对象
- void removeAttribute(String name):移除指定的属性
- void setAttribute(String name, Object o): 设置属性


request(HttpRequest)对象的属性作用范围仅限一次请求；
pageContext(PageContext)对象的属性作用范围在当前页面；
Session（HttpSession）对象属性的所用范围在一次会话；
application（ServletContext）对象属性作用范围在整个WEB应用中。

#### 请求重定向和转发

RequestDispatcher是用于服务器端请求转发的接口，其实例对象是由Servlet引擎创建，用于包装一个要被其他资源调用的资源（例如，Servlet、HTML文件、JSP文件等）并可以通过其中的方法将客户端的请求转发给所包装的资源。

RequestDispatcher有两个方法，分别是forward和include。

	//获取RequestDispatcher对象， path为要跳转的路径
	RequestDispatcher reqeustDispatcher = request.getResquestDispatcher("/" + path);
	//执行转发
	requestDispatcher.forward(request, response);
	
重定向：
response.sendRedirect(locaiton);		

**重定向和转发的区别：**

- foward方法是服务器方法，浏览器不会感知到这种变化，地址栏路径不会发生变化浏览器只一次请求，转发在服务端执行，foward是RequestDispatcher的方法，reqeust.getRequestDisptcher().forwar(path);请求转发只能转发到当前应用的资源，forward的路径的“/”表示当前WEB应用的根目录
- sendRedirect重新定向是服务器返回给浏览器的指示，浏览器再发一次到新路径的请求, sendRedirect是HttpServletResponse的方法，response.sendRedirect(location)，重定向是浏览器端的行为，所以可以转发到任意地址，redirect的location中的“/”是WEB站点的根目录

### JSP指令

JSP指令（direction）是为JSP引擎设计的，它们并不直接产生任何可见的输出，而是**告诉引擎如何处理JSP页面中的相关部分**

JSP指令的基本语法：
	
	<%@ page contentType="text/html;charset=utf8" %>
JSP2.0中定义了page、include和taglib三种指令

####  page指令
	
page指令用于定义JSP页面的各种属性，无论page指令出现在JSP页面中的什么地方，它作用的都是整个页面，为了保持程序的可读性和遵循良好的编程习惯，page指令最好是放在整个page页面的起始位置

用法：

	<%@ page
		[language="java"] 
		[contentType=""]
		[extends="package.class"]
		[import="{package.class | package}, ..."]
		[session="true | false"]
		[error="error.jsp"]
		[isErrorPage="true"]
		[isELIgnored="true"]
	%>
	
	<%@ page language="java" contentType="text/html;charset=utf-8" pageEncoding="utf-8"%>


#### include指令
include指令用于通知JSP引擎在翻译当前JSP页面时将其他文件中的内容合并到当前的JSP页面转换成的Servlet源文件中，这种在源文件级别进行引入的方式成为**静态引入**

用法：
	
	<%@ include file="relativeURL"%>
	
	**注意：这里的URL是相对路径 **
	
细节：

- 被引入的文件必须遵循JSP语法，其中的内容可以包含静态HTML/JSP脚本元素、JSP指令和JSP行为元素等普通JSP页面所具有的一切内容
- 被引入文件会被JSP引擎当做JSP页面来处理，不管是各种文件扩展名，JSP规范建议使用.jspf（JSP fragment）来作为静态引入文件的扩展名
- page指令的相关属性除了import和pageEncoding之外，其他属性不能有两个不同值

#### JSP标签
JSP提供了一种称之为Action的元素，在JSP页面中使用Action元素可以完成各种通用的JSP页面功能，也可以实现一些处理复杂业务逻辑的专用功能；
Action元素采用XML元素的语法格式，即每个Action元素在JSP页面中都以XML标签的形式出现。JSP规范中定义了一些标准的Action元素，这些元素的标签名都以jsp为前缀，并且全部采用小写，如<jsp:include>、<jsp:forward>

**<jsp:include>标签和<%@ include file="URL"%>**的区别：
include指令是源码级引入，在编译期引入文件，两部分会在一个Servlet文件中，include标签是运行期引入，会分别编译成两个Servlet文件并通过方法调用的方式实现引入

#### 中文乱码问题

- 1 设置page属性

	<!-- charset和pageEncoding的属性值要一致，一般为utf-8 -->
	<%@ page contentType="text/html;charset=utf-8" pageEncoding="utf-8"%>

- 2 request.setCharacterEncoding("utf-8")

- 3 先解码再编码

		String val = request.getParameter("userName");
		String userName = new String(val.getBytes("iso-8859-1"), "utf-8");
		out.print(userName);
		
- 4 URI请求编码

设置Tomcat的useBodyEncodingForURI属性，为Connector节点添加useBodyEncodingForURI属性为true

### MVC设计模式


**组件开发**

JavaEE常见组件：commons-beanutils、commons-dbcp、commons-dbutils、commons-fileupload、commons-logging、hibernate-release、jbmp


Model-View-Controller

- Model，是应用程序的主体部分，表示业务数据和业务逻辑
- View，是展示给用户与之交互的页面，
- Controller，接收用户输入，并调用模型和视图完成处理任务



### Cookie & Session 会话跟踪

HTTP协议是一个无状态的协议，WEB服务器本身不能识别出哪些请求是同一个浏览器发出的，即使HTTP1.1的长连接，在用户有一段时间没有发起请求之后连接也会被关闭，Web服务器需要一种机制来识别用户浏览器的请求，维护会话状态。

在Servlet规范中有两种机制来实现会话跟踪：

- Cookie
- Session



#### Cookie
cookie机制是在客户端保持HTTP状态信息的方案，Cookie是在浏览器访问WEB服务器资源时，由WEB服务器在HTTP相应消息头中附带传送给浏览器的一个小文本文件。一旦WEB浏览器保存了某个Cookie，那么它以后每次访问该WEB服务器时都会在HTTP请求头中将这个Cookie回传给WEB服务器。

WEB服务器在通过HTTP相应消息中增加**Set-Cookie响应头**字段来讲Cookie信息发送给浏览器，浏览器则通过在HTTP请求消息中增加Cookie请求头字段将Cookie回传给WEB服务器

一个Cookie只能标识一种信息，它至少包含一个标识该信息的名称（NAME）和设置值（VALUE）

一般浏览器允许存放300个左右Cookie，每个站点最多20个，每个Cookie最大4K

	Cookie cookie = new Cookie(name, value);
	request.addCookie(cookie);


**Cookie的path（作用路径）问题**
JSP中，cookie的作用范围在当前目录及其子目录中



	
	 
