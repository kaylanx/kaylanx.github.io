---
id: 57
title: 'How to Print Out Bind Variables in Java Prepared Statements.'
date: '2008-07-29T23:52:00+01:00'
author: andykayley
layout: post
guid: 'http://54.194.124.185/2008/07/29/how-to-print-out-bind-variables-in-java-prepared-statements/'
permalink: /2008/07/29/how-to-print-out-bind-variables-in-java-prepared-statements/
blogger_blog:
    - andykayley.blogspot.com
blogger_author:
    - 'Andy Kayley'
blogger_permalink:
    - /2008/07/how-to-print-out-bind-variables-in-java.html
blogger_internal:
    - /feeds/6447184396655674320/posts/default/2909241200946650425
restapi_import_id:
    - 58ebf9b1175ef
original_post_id:
    - '57'
categories:
    - 'bind variables'
    - hibernate
    - java
    - jdbc
    - mysql
    - p6spy
    - PreparedStatement
    - Uncategorized
---

Ever been frustrated by the fact that when you use (as you should do!) bind variables within a `Java` prepared statement you cannot see the actual values that get put into those bind variables? I know I have. For example if you had a class that had some `JDBC` calls that did the following….

{% highlight java %}
Class.forName("com.mysql.jdbc.Driver");

connection = DriverManager.getConnection("jdbc:mysql://localhost/message_board", "root", "");

statement = connection.prepareStatement("select * from messages where id = ?");
statement.setLong(1,1L);

resultSet = statement.executeQuery();
{% endhighlight %}

There is no nice out the box way to find out what the ?'s will be resolved to. I've always wanted a method on `PreparedStatement` that would address this, but alas there still is none. Traditionally I would have done something like the following…

{% highlight java %}
long id = 1L;
String sql = "select * from messages where id = ?"
logger.info(sql + "; ?1=" + id);

statement = connection.prepareStatement(sql);
statement.setLong(1,id);
{% endhighlight %}

Or write your own convoluted method for replacing each `?` with the value which is prone to bugs and not being database agnostic.

When using an old `JDBC` based in-house framework this really wasn't an issue for me, as we had such a method that was prone to bugs but for the most part it worked OK. Since then the work I have been doing has been much more hibernate based. This is where the issue came for me, you can turn on `SQL` logging either by configuring your favourite logging framework to do so, or by turning on the `hibernate.show_sql` property, which will give you the following output…

{% highlight sql %}
select message0_.id as id2_, message0_.email_address as email2_2_, message0_.inserted_at as inserted3_2_, message0_.remote_host as remote4_2_, message0_.text as text2_ from messages message0_ where message0_.id=?
{% endhighlight %}

This `SQL` was generated from the following `HQL`

{% highlight sql %}
from Message where id = ?
{% endhighlight %}

Now apart from this sql being particularly hideous to look at, the question mark is still there, we have no way of possibly knowing what those bind variables were without printing them out at the time the `HQL` is executed.

Enter `p6spy`.

I did a bit of googling about and found other people had the same issue. And there is an answer to this problem, and its called `p6spy`. `P6spy` is a `JDBC` driver that is a proxy for your real driver. I've only just started to use it, I believe it is capable of many cool things, but the one that I am interested in allows you to print out `SQL` from prepared statements with – yep, you guessed it – the bind variables substituted for the real values.

You can download it from [https://github.com/p6spy/p6spy](https://github.com/p6spy/p6spy), it must be fairly stable as the last release was in 2003 (I'm so annoyed that I didn't know about this earlier). There seems to be a little problem with their website as when you click download you get an error while trying to fill in or ignore their survey. Don't panic, once you've ignored the survey go back to the main page and click download again and the files will be there waiting for you to download.

So how do we configure this little beauty? Well the download is **`p6spy-install.zip`**, unzip this somewhere safe (and keep in mind that all the files are in the root of the zip file, so you might want to create a directory to unzip it to first). The files we're interested in are **`p6spy.jar`** and **`spy.properties`**. Copy the jar file into the lib folder of your project and copy the **`spy.properties`** file into somewhere that will be on the **`classpath`**.

If you look inside spy.properties you will see that it's really very well documented and tells you all you really need to know on how to configure it. Although one thing to note in there is that they have said that the real driver you configure in your application should be set to **`com.p6spy.engine.P6SpyDriver`** which is actually incorrect, it should be **`com.p6spy.engine.spy.P6SpyDriver`**, the first one doesn't exist and you will get a ClassNotFoundException.

So what I did… In my spy.properties I set the realdriver…

{% highlight properties %}
# mysql Connector/J driver
realdriver=com.mysql.jdbc.Driver
{% endhighlight %}

(note by default the open source `MySQl` driver is uncommented which was a gotcha for me at first, so I commented it out, and uncommented the above)

Next I altered my code in the first example to the following…

{% highlight java %}
Class.forName("com.p6spy.engine.spy.P6SpyDriver");

connection = DriverManager.getConnection("jdbc:mysql://localhost/message_board", "root", "");

statement = connection.prepareStatement("select * from messages where id = ?");
statement.setLong(1,1L);

resultSet = statement.executeQuery();
{% endhighlight %}

And lo and behold in the root of my `Eclipse` project on the file system a file called `spy.log` has appeared, and in it the following…

{% highlight sql %}
1217374504582|6|0|statement|select * from messages where id = ?|select * from messages where id = 1
1217374504595|-1||resultset|select * from messages where id = 1|email_address = myemail@nospam.com, id = 1, inserted_at = 2008-07-29 00:00:00.0, remote_host = localhost, text = Thank you for showing me the wonders of p6spy
{% endhighlight %}

There is lots of other stuff in the log file as well but as you can see its very handy, as it shows you the statement with question marks and with the real values substituted, and also shows you the results set which is passed back 🙂 Beautiful lol.

And the hql of

{% highlight sql %}
from Message where id = ?
{% endhighlight %}

is output as:

{% highlight sql %}

1217374913350|5|2|statement|select message0_.id as id2_, message0_.email_address as email2_2_, message0_.inserted_at as inserted3_2_, message0_.remote_host as remote4_2_, message0_.text as text2_ from messages message0_ where message0_.id=?|select message0_.id as id2_, message0_.email_address as email2_2_, message0_.inserted_at as inserted3_2_, message0_.remote_host as remote4_2_, message0_.text as text2_ from messages message0_ where message0_.id=1
1217374913369|-1||resultset|select message0_.id as id2_, message0_.email_address as email2_2_, message0_.inserted_at as inserted3_2_, message0_.remote_host as remote4_2_, message0_.text as text2_ from messages message0_ where message0_.id=1|email2_2_ = myemail@nospam.com, remote4_2_ = localhost
{% endhighlight %}

Still ugly but at least I can now see what the bind variables are going to be 🙂

You can also configure it so it comes out in your normal logging file, the `spy.properties` file details how you do that.

Oh how I wish I'd have known this 4 years ago.

{% include archive.html %}