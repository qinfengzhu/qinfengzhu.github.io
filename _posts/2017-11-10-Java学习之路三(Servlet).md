---
layout: post
title: "Java学习之路三(Servlet)"
date: 2017-10-18   
tag: java学习之路 
---

### 什么是Servlet 

Servlet是链接Web服务器与服务端Java程序的协议，是一种通信规范。
![Servlet所处的位置](/images/java/servlet.png)

### Servlet 生命周期

包括四部分:Servlet对象的创建、Servlet对象的初始化、Servlet对象服务的执行、Servlet对象被销毁。
![Servlet的生命周期](/images/java/servlet-life.png)

### Servlet 特征

1. Servlet是单例多线程
2. 一个Servlet实例只会执行一次无参构造器与init()方法，并且是在第一次访问时执行
3. 用户每提交一次对当前Servlet的请求，就会执行一次service()方法
4. 一个Servlet实例只会执行一次destroy()方法，在应用停止时执行
5. 由于Servlet是单例多线程的，所以为了保证其多线程安全性，一般情况下是不为Servlet类定义可修改的成员变量的。因为每个线程均可修改这个成员变量，会出现线程安全问题。
6. 默认情况下,Servlet在Web容器启动时是不会被实例化的。

### 什么是ServletConfig?

在Servlet接口的init()方法中具有唯一的一个参数ServletConfig。ServletConfig是个接口，就是Servlet配置,在Web.xml中对当前Servlet类的配置信息。
servlet规范将Servlet的配置信息全部封装到ServletConfig接口对象中。在Web容器调用init()方法时，Web容器首先会将Web.xml中当前Servlet类的配置信息封装为一个对象。
这个对象的类型实现了ServletConfig接口，Web容器会将这个对象传递给init()方法中的ServletConfig参数。

### 获取ServletConfig对象

源码查看需要下载tomcat的源码文件

[tomcat9.0.1](http://111.23.5.172:81/mirrors.hust.edu.cn/apache/tomcat/tomcat-9/v9.0.1/src/apache-tomcat-9.0.1-src.zip)
[tomcat8.0.47](http://mirrors.shuosc.org/apache/tomcat/tomcat-8/v8.0.47/src/apache-tomcat-8.0.47-src.zip)

ServletConfig 对象 可以获取Web.xml中配置节点servlet中的的信息
```  
   <servlet>
  	<servlet-name>demo-servlet</servlet-name>
  	<servlet-class>com.bestkf.servlets.DemoServlet</servlet-class>
  	<load-on-startup>1</load-on-startup>
  </servlet>
```

### url-pattern

* 全路径模式
```
/* 是真正的全路径模式，可以拦截所有请求，无论是动态资源请求还是静态资源请求均会被拦截
/ 只会拦截静态资源请求，对于动态资源请求是不进行拦截的
```

* 后缀模式
```
*.do  
*.action 
*.xxxx
这种方式叫做后缀模式
```
* 全路径模式不能够与后缀模式混合使用
```
/xxx/*.action 这种写法是错误的
```

* 匹配原则
```
1.路径优先后缀匹配原则
2.精确路径优先匹配原则
3.最长路径优先匹配原则
```

### Servlet 接口实现繁琐,所以定义一个简洁的

```java
public abstract class AbstractServlet implements Servlet
{
	@Override
	public void init(ServletConfig config) throws ServletException{

	}
	@Override
	public ServletConfig getServletConfig(){
		return null;
	}
	@Override
	public abstract void service(ServletRequest req,ServletResponse res) throws ServletException,IOException;
	@Override
	public String getServletInfo(){
		return null
	}
	@Override
	public void destroy(){

	}
}

```

### HttpServlet 

HttpServlet 继承自GenericServlet 继承自Servlet 

### HttpServletRequest 

Javax.Servlet.Http.HttpServletRequest 是SUN制定的Servlet规范，是一个接口，表示请求，其父接口是 Javax.Servlet.ServletRequest。
Http 请求协议 的完整内容都被封装到request对象中。

```
1. 请求参数是存放在Map中的
2. 这个Map的Key为请求参数的名称，为String类型,这个Map的Value为请求参数的所有值，为String[] 类型
3. 使用最多的是getParameter()方法，其等价于getParameterValues()[0]
```

URL : http://www.bestkf.com/product/get  : HttpServletRequest.getRequestURL()

URI : /product/get 						 : HttpServletRequest.getRequestURI()

ContextPath:  /                          : HttpServletRequest.getContextPath() Web项目的根

远程Ip ：                                ：HttpServletRequest.getRemoteAddr()  获取远程Ip


### 中文乱码问题

Http协议规定，数据的传输采用字节编码方式，即无论浏览器提交的数据包含的中文是什么字符编码格式，一旦浏览器经过Http协议传输，则这些数据均将以字节的形式
上传给服务器。因为Http协议的底层使用的是TCP传输协议。Tcp是一种面向连接的、可靠的、基于字节流的、端对端的通信协议。请求中，这些字节均以%开头，并以十六进制形式出现。
如%5A%3D等。

乱码的产生： 用户通过浏览器提交一个包含UTF-8编码格式的两个字的中文请求时，浏览器会将这两个中文字符变为六个字节(一般一个UTF-8汉字占用三个字节)，即形成六个类似%8E的字节
表示形式，并将这六个字节上传至Tomcat服务器。Tomcat服务器在收到这六个字节后，并不知道他们原始采用的是什么字符编码。而Tomcat默认的编码格式为ISO-8859-1，所以会将这六个
字节按照ISO-8859-1的格式进行编码，编码后在控制台显示，所以控制台会显示为乱码。

Post乱码解决方案通过HttpServletRequest.setCharacterEncoding("UTF-8")

Get乱码解决方案Server.xml 配置节点
```
<Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" URIEncoding="UTF-8" />
```

Get乱码另一种解决方案:
```
String productName=request.getParameter("productName");
byte[] bytes = name.getBytes("ISO8859-1");
productName=new String(bytes,"UTF-8");
```

### 请求转发与重定向

通过HttpServletRequest获取到的RequestDispatcher对象的forward()方法，可以完成请求转发功能。而通过HttpServletResponse的sendRedirect()方法，可以完成重定向功能。

请求转发与重定向区别：
![请求转发与重定向区别](/images/java/servlet-forward-redirect.png)

```
asp.net也有转发与重定向:
转发:Server.Transfer;Server.Execute
重定向:Response.Redirect
```

Url编码,主要解决 sendRedirect中参数出现空字符的情况
```
//下面两种方法是成对出现的

URLEncoder.encode(参数，“UTF-8”); //参数可能为空字符，参数编码

URLDecoder.decode(参数,"UTF-8"); //参数解码
```

### 路径

路径 构成由 资源路径与资源构成，比如: http://www.bestkf.com/js/mini.js ; 资源路径 http://www.bestkf.com/js/  ;资源 mini.js

绝对路径 = 参照路径 + 相对路径 

相对路径两种写法：一种是以斜杠开头的相对路径，一种是以路径名称开头的相对路径。

根据相对路径是否以斜杠开头，且路径出现的文件的不同，其默认的参照路径是不同的。

前台路径是查找，后台路径是标识。

* 以斜杠开头的相对路径

1.前台路径，前台路径的参照路径是Web服务器的根路径，即 http://www.bestkf.com ;将前台路径转换为绝对路径的工作，是由浏览器来完成的。该路径的作用是要求用户提交对某种资源的请求，是要查找并定位服务器中的某资源。

2.后台路径,由服务器解析执行的代码及文件中所包含的路径。例如,Java代码中的路径、Jsp文件动态部分(Java代码块)
中的路径、Xml等配置文件中的路径（配置文件是要被Java代码解析后加载到内存的）。后台路径的参照路径是Web应用的根路径。

* 以路径名称开头的相对路径
	
以路径名称开头的相对路径，无论是出现在前台页面，还是出现在后台java代码或配置文件中，其参照路径都是当前访问路径的资源路径。
即使是Response的sendRedirect()方法的参数路径，若不以斜杠开头，其也属于"以路径名称开头的相对路径"类，参照路径为当前访问路径的资源路径。

* sendRedirect(“/productServlet”)特例

所以一般sendRedirect会这么写,response.sendRedirect(request.getContextPath()+"/produjctServlet")


### JVM中可能存在线程安全问题的数据分析

* 栈内存数据分析

栈内存是多例的，即JVM会为每一个线程创建一个栈，所以其中的数据不是共享的。另外，方法中的局部变量存放在Stack的栈帧中，方法执行完毕，栈帧弹栈，局部变量消失。
局部变量是局部的，不是共享的。所以栈内存中的数据不存在线程安全问题。

* 堆内存数据分析

一个JVM 中只存在一个堆内存，堆内存是共享的。被创建出的对象是存放在堆内存的，二存放在堆内存中的对象，实际就是对象成员变量的值的集合。即成员变量是存放在堆内存的。
堆内存中的数据是多线程共享的，也就是说，堆内存中的数据是存在线程安全问题的。

* 方法区数据分析

一个JVM中只存在一个方法区。静态变量与常量存放在方法区，方法区是多线程共享的。常量是不能被修改的量，所以常量不存在线程安全问题。静态变量是多线程共享的，所以静态变量存在线程安全问题。

### 线程安全问题的解决方案

1.对于一般性的类，不要定义为单例的。除非项目有特殊需求，或者该类对象属于重量级对象。所谓重量级对象是指，创建该类对象时需要占用较大的系统资源。

2.无论类似否为单例类，尽量不适用静态变量。

3.若需要定义为单例类，则单例类中尽量不适用成员变量

4.若单例类中必须要适用成员变量。则对成员变量的操作，可以添加串行化锁synchronized,实现线程同步。不过，最好不要适用线程同步机制。因为一旦操作进入串行化的排队状态，将大大降低程序的执行效率。


