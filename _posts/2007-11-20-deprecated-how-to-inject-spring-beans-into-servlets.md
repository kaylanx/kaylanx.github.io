---
id: 63
title: '@Deprecated: How to Inject Spring Beans into Servlets.'
date: '2007-11-20T23:58:00+00:00'
author: andykayley
layout: post
guid: 'http://54.194.124.185/2007/11/20/deprecated-how-to-inject-spring-beans-into-servlets/'
permalink: /2007/11/20/deprecated-how-to-inject-spring-beans-into-servlets/
blogger_blog:
    - andykayley.blogspot.com
blogger_author:
    - 'Andy Kayley'
blogger_permalink:
    - /2007/11/how-to-inject-spring-beans-into.html
blogger_internal:
    - /feeds/6447184396655674320/posts/default/1971773936813609356
restapi_import_id:
    - 58ebf9b1175ef
original_post_id:
    - '63'
categories:
    - Uncategorised
    - Uncategorized
---

**This post has been Deprecated, see [this post instead](/2008/06/29/how-to-inject-spring-beans-into-servlets-revisited/)**

A couple of months ago I was working on an application that used Servlets as entry points to an application, sort of like a web service. This was a new application and I decided to use the `Spring` framework for dependancy injection and to make using `Hibernate` easier.

I am fairly new to `Spring` and never needed to inject any spring beans into `Servlet` before, and I thought there must be a way to do it. However after browsing through a number of websites, blog posts and forum posts, it appeared that there wasn't a clean spring way to do this.

## Solution 1

In the end I read somewhere that you could inject spring beans into the `ServletContext`, so I took this route.

With this you have to declare this little piece in your `applicationContext.xml`

{% highlight xml %}
<bean class="org.springframework.web.context.support.ServletContextAttributeExporter">
 <property name="attributes">
     <map>
         <!-- inject the following beans into the servlet
context so the servlets can access them. -->
         <entry key="myBeanFromSpring">
             <ref bean="myBeanFromSpring"/>
         </entry>
     </map>
 </property>
</bean>
{% endhighlight %}

As you can see this puts the spring bean `myBeanFromSpring` into the servlet context. Therefore in your servlet code you can do the following…

{% highlight java %}
protected void doGet(HttpServletRequest reqest, HttpServletResponse response)
    throws ServletException, IOException {

   MyBeanFromSpring myBean = MyBeanFromSpring)getServletContext().getAttribute("myBeanFromSpring");
   myBean.someOperation();
   ...
}
{% endhighlight %}

Although this works it still doesn't feel very spring like.

## Solution 2

There is another way to achieve the same thing. You can use `WebApplicationContext` and get the beans directly from Spring without having to inject anything into the servlet context.

{% highlight java %}
void doGet(HttpServletRequest reqest, HttpServletResponse response) 
    throws ServletException, IOException {

   WebApplicationContext springContext = WebApplicationContextUtils.getWebApplicationContext(getServletContext());
   MyBeanFromSpring myBean =(MyBeanFromSpring)springContext.getBean("myBeanFromSpring");
   myBean.someOperation();
   ...
}
{% endhighlight %}

Although this achieves the same thing and is probably more concise than **Solution 1**, it still is not achieving what I initially wanted, which was dependancy injection into the servlet.

## Solution 3

Although I stayed with **Solution 1** for the application it got me thinking. So I set out to write a sub class of `HttpServlet` that would use the `servletContext` solution and use reflection to figure out if the servlet had any setters on that spring should call.

My original servlet that had to do all the stuff with getting the servlet context suddenly looks a lot more spring-like…

{% highlight java %}
import name.kayley.springutils.SpringDependencyInjectionServlet;

public class MyServlet extends SpringDependencyInjectionServlet {
    private MyBeanFromSpring myBean;
    
    public void setMyBeanFromSpring(MyBeanFromSpring myBean) {
        this.myBean = myBean;
    }

    protected void doGet(HttpServletRequest reqest, HttpServletResponse response)
        throws ServletException, IOException {
        myBean.someOperation();
    }
}
{% endhighlight %}

And here is the source of the `SpringDependencyInjectionServlet`, go easy on it, It's the first version and it passes the unit tests I have written for it, use it at your own risk or as a starting point, but like I said it appears to work ok so far.

I have attempted to keep in with the `Spring` autowiring of giving 2 options, autowire by type and by name, so you can override `autowireByType` if you want to autowire by name. I think they need some work as I believe that autowire by name should go off the name of the parameter to the setter method and not the setter method name itself… keep coming back for updates.

{% highlight java %}
package name.kayley.springutils;

import java.io.IOException;
import java.lang.reflect.Method;
import java.util.Enumeration;
import java.util.logging.Level;
import java.util.logging.Logger;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.commons.lang.StringUtils;

public class SpringDependencyInjectionServlet extends HttpServlet {

   private static final Logger logger =
Logger.getLogger(SpringDependencyInjectionServlet.class.getName());

   protected void service(HttpServletRequest request,
HttpServletResponse response) throws ServletException, IOException {
       Enumeration attributeNames = getServletContext().getAttributeNames();

       while (attributeNames.hasMoreElements()) {

           String name = (String) attributeNames.nextElement();
           logger.log(Level.INFO, "attempting to autowire " + name);

           autowire(name);
       }

       super.service(request, response);
   }

   protected boolean autowireByType() {
       return true;
   }

   private void autowire(String name) {
       if (name != null) {
           Object attribute = getServletContext().getAttribute(name);
           Class c = getClass();
           while (c != null && c != c.getSuperclass()) {
               try {
                   if (autowireByType()) {
                       if (byType(c, attribute)) {
                           break;
                       }
                       else {
                           c = c.getSuperclass();
                       }
                   }
                   else {
                       if (byName(c, name, attribute)) {
                           break;
                       }
                       else {
                           c = c.getSuperclass();
                       }
                   }
               }
               catch (NoSuchMethodException e) {
                   c = c.getSuperclass();
               }
           }
       }
   }

   private boolean byName(Class c, String name, Object attribute)
       throws NoSuchMethodException {
       boolean success = false;

       if (attribute != null) {

           Method[] methods = c.getDeclaredMethods();
           for (Method method : methods) {
               if (method.getName().equals(getMethodName(name))) {
                   Class[] paramTypes = method.getParameterTypes();

                   if (paramTypes.length == 1) {
                       success = invokeSpringBeanSetter(method, attribute);
                   }
               }
           }
       }
       return success;
   }

   private boolean byType(Class c, Object attribute) {
       boolean success = false;

       if (attribute != null) {
           Method[] methods = c.getDeclaredMethods();

           for (Method method : methods) {
               Class[] paramTypes = method.getParameterTypes();

               Class attributeClass = attribute.getClass();
               if (paramTypes.length == 1 &&
          paramTypes[0].equals(attributeClass)) {
                   boolean succeeded = invokeSpringBeanSetter(method,attribute);
                   if (!success && succeeded) {
                       success = succeeded;
                   }
               }
           }
       }
       return success;
   }

   private boolean invokeSpringBeanSetter(Method method, Object attribute) {
       boolean success = false;
       try {
           method.invoke(this, attribute);
           success = true;
       }
       catch (Exception e) {
           // TODO do we care?
       }
       return success;
   }

   private String getMethodName(String contextName) {
       return "set" + StringUtils.capitalize(contextName);
   }
}
{% endhighlight %}
