---
id: 59
title: 'How to Inject Spring Beans into Servlets Revisited.'
date: '2008-06-29T15:37:00+01:00'
author: andykayley
layout: post
guid: 'http://54.194.124.185/2008/06/29/how-to-inject-spring-beans-into-servlets-revisited/'
permalink: /2008/06/29/how-to-inject-spring-beans-into-servlets-revisited/
blogger_blog:
    - andykayley.blogspot.com
blogger_author:
    - 'Andy Kayley'
blogger_permalink:
    - /2008/06/how-to-inject-spring-beans-into.html
blogger_internal:
    - /feeds/6447184396655674320/posts/default/5954556380060780250
restapi_import_id:
    - 58ebf9b1175ef
original_post_id:
    - '59'
categories:
    - Uncategorised
    - Uncategorized
---

Back in November I wrote a post about [How to Inject Spring Beans into Servlets](/2007/11/20/deprecated-how-to-inject-spring-beans-into-servlets/){:target="_blank"}, since then one of the comments made on the post by [angel](http://www.blogger.com/profile/09435493279526225197){:target="_blank"}, mentioned an interface and Servlet that comes with spring that does this the proper spring way. Thank you very much angel, this is what I was looking for before I wrote the last post 🙂

The proper way how this should be achieved is to write a class that implements [HttpRequestHandler](http://static.springframework.org/spring/docs/2.5.x/api/org/springframework/web/HttpRequestHandler.html){:target="_blank"} and define this class in your `applicationContext.xml`. Then you define a servlet in `web.xml` with a class of [HttpRequestHandlerServlet](http://static.springframework.org/spring/docs/2.5.x/api/org/springframework/web/context/support/HttpRequestHandlerServlet.html){:target="_blank"} and make sure the name of the servlet is the same as the bean you defined in `applicationContext.xml`.

First lets write a class that we want to inject into a servlet…

{% highlight java %}
package name.kayley.httprequesthandlertest;

public interface ServiceToInject {
  public String someMethod();
}
{% endhighlight %}

{% highlight java %}
package name.kayley.httprequesthandlertest;

public class ServiceToInjectImpl implements ServiceToInject {
  public String someMethod() {
      return "someMethod called";
  }
}
{% endhighlight %}

Next we'll write a class that implements the `HttpRequestHandler` this is basically our servlet.

{% highlight java %}
package name.kayley.httprequesthandlertest;

public class MyHttpRequestHandler implements HttpRequestHandler {

   private ServiceToInject serviceToInject;

   public void setServiceToInject(ServiceToInject serviceToInject) {
       this.serviceToInject = serviceToInject;
   }

   public void handleRequest(HttpServletRequest request, HttpServletResponse httpServletResponse) throws ServletException, IOException {

       System.out.println("******* HELLO *******");
       System.out.println(serviceToInject.someMethod());
   }
}
{% endhighlight %}

Next all we need to do is configure our `web.xml` and `applicationContext.xml`…

First `applicationContext.xml`…

{% highlight xml %}
<xml version="1.0" encoding="UTF-8">
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.0.xsd">

    <bean id="serviceToInject" class="name.kayley.httprequesthandlertest.ServiceToInjectImpl"/>

    <bean id="myhttprequesthandler" class="name.kayley.httprequesthandlertest.MyHttpRequestHandler">
        <property name="serviceToInject" ref="serviceToInject"/>
    </bean>

</beans>
{% endhighlight %}

And finally configure your `web.xml` …

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
    http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
           version="2.5">

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/applicationContext.xml</param-value>
    </context-param>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <servlet>
        <servlet-name>myhttprequesthandler</servlet-name>
        <servlet-class>org.springframework.web.context.support.HttpRequestHandlerServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>myhttprequesthandler</servlet-name>
        <url-pattern>/MyServlet</url-pattern>
    </servlet-mapping>
</web-app>
{% endhighlight %}

Now run your web app and go to `http://localhost:8080/MyServlet`

You should see in your console log the output of your injected servlet…

{% highlight console %}
\*\*\*\*\*\*\* HELLO \*\*\*\*\*\*\*  
someMethod called
{% endhighlight %}

Brilliant 🙂 Thanks again angel.

{% include archive.html %}