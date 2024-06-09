---
id: 61
title: 'How to configure Tomcat 5.0.x to use Java Logging'
date: '2008-01-05T09:08:00+00:00'
author: andykayley
layout: post
guid: 'http://54.194.124.185/2008/01/05/how-to-configure-tomcat-5-0-x-to-use-java-logging/'
permalink: /2008/01/05/how-to-configure-tomcat-5-0-x-to-use-java-logging/
blogger_blog:
    - andykayley.blogspot.com
blogger_author:
    - 'Andy Kayley'
blogger_permalink:
    - /2007/12/how-to-configure-tomcat-50x-to-use-java.html
blogger_internal:
    - /feeds/6447184396655674320/posts/default/5384621694977728032
restapi_import_id:
    - 58ebf9b1175ef
original_post_id:
    - '61'
categories:
    - java
    - juli
    - logging
    - tomcat
    - Uncategorized
---

If you are using `Java Logging` and a version of `Tomcat` previous to `Tomcat 5.5`, you have to use the deprecated `Logger` declarations in your `server.xml` or `context.xml` files. The are deprecated in `Tomcat 5.0` and have actually been removed in `Tomcat 5.5`.

This is used to achieve separate log files for separate virtual hosts/web applications for example in your `server.xml` you may have…

{% highlight xml %}
<Host name="myapp.mydomain.com" debug="0" appBase="webapps/myapp"
      unpackWARs="true" autoDeploy="true"
      xmlValidation="false" xmlNamespaceAware="false">
      <Logger className="org.apache.catalina.logger.FileLogger"
              directory="logs"  prefix="myapp." suffix=".log"
              timestamp="true"/>
{% endhighlight %}

This will log the `myapp.timestamp.log` file with all the contents of the webapp `myapp`.

If you are using `log4j` you can simply omit this and use the `log4j.properties` (or `log4j.xml` file) within your webapp to specify what the logging levels are and where to log to etc.

`Java Logging` lacks this functionality. The `Java Logging` configuration file `logging.properies` is a JVM-wide properties file, and therefore traditionally you would have to use the Logger feature to get separate log file per webapp.

In `Tomcat 5.5` Apache introduced `juli` which is an implementation of the java logger that addresses the shortcomings of the `logging.properties` file.  i.e. with `Tomcat 5.5` and above you can have a `logging.properties` file within your webapp.

Now if you are tied to `Tomcat 5.0` for your deploys you can actually add `juli` logging to `Tomcat 5.0` quite easily, here's what you do…

Firstly download the [`Tomcat 5.5`](https://archive.apache.org/dist/tomcat/tomcat-5/v5.5.36/bin/){:target="_blank"} core binary distribution.

From this `apache-tomcat-5.5.36.zip` file extract the following two files…  
  
* `apache-tomcat-5.5.36/bin/tomcat-juli.jar` into `jakarta-tomcat-5.0.28/bin`  
and  
* `apache-tomcat-5.5.36/conf/logging.properties` into `jakarta-tomcat-5.0.28/conf`  
  
Next we need to edit our catalina startup script to tell `Java Logging` about the juli logging jar and properties file.

If you are using a flavour of linux/unix edit `jakarta-tomcat-5.0.28/bin/catalina.sh` and under the cygwin section add the following…

{% highlight shell %}
# Set juli LogManager if it is present
if [ -r "$CATALINA_BASE"/conf/logging.properties ]; then
 JAVA_OPTS="$JAVA_OPTS "-Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager" "-Djava.util.logging.config.file="$CATALINA_BASE/conf/logging.properties"
fi
{% endhighlight %}

and then after the classpath section add…

{% highlight shell %}
if [ -r "$CATALINA_HOME"/bin/tomcat-juli.jar ]; then
 CLASSPATH="$CLASSPATH":"$CATALINA_HOME"/bin/tomcat-juli.jar
fi
{% endhighlight %}

If you are using windows edit `jakarta-tomcat-5.0.28/bin/catalina.bat` and after the setenv.bat section add…

{% highlight shell %}
if not exist "%CATALINA_BASE%conflogging.properties" goto noJuliProps
set JAVA_OPTS=%JAVA_OPTS% -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djava.util.logging.config.file="%CATALINA_BASE%conflogging.properties"
:noJuliProps
{% endhighlight %}

and then after the classpath section add…

{% highlight shell %}
if not exist "%CATALINA_HOME%bintomcat-juli.jar" goto noJuli
set CLASSPATH=%CLASSPATH%;%CATALINA_HOME%bintomcat-juli.jar
:noJuli
{% endhighlight %}

Et voilà that's it!

All you need now in your webapp is add your own specific `logging.properties` file to the `WEB-INF/classes` directory of the war, an example of which…

{% highlight properties %}
####### --------- logging.properties --------- #############
handlers = org.apache.juli.FileHandler, java.util.logging.ConsoleHandler

############################################################
# Handler specific properties.
# Describes specific configuration info for Handlers.
############################################################

org.apache.juli.FileHandler.level=FINEST
org.apache.juli.FileHandler.directory=/var/log/tomcat
org.apache.juli.FileHandler.prefix=myapp.

java.util.logging.ConsoleHandler.level=FINEST
java.util.logging.ConsoleHandler.formatter=java.util.logging.SimpleFormatter
{% endhighlight %}
