---
layout: post
title: 'Clob Handling Made Easy in Java with Oracle 10g'
---

I found out a pretty neat thing last week about the JDBC Driver for Oracle 10g.

Previously when I've had to use a `CLOB` (**C**haracter **L**arge **OB**ject) in Java using Oracle I've had to use the Oracle specific classes, rather than use the `java.sql` classes.

i.e.

{% highlight java %}
String sql = "SELECT DATA FROM MY_TABLE WHERE ID = ? FOR UPDATE";

PreparedStatement stmt = con.prepareStatement(sql);
stmt.setString(1, primaryKey);
ResultSet res = stmt.executeQuery();

oracle.sql.CLOB clob = (oracle.sql.CLOB)res.getClob(1);
{% endhighlight %}

Then you had to use methods on the `CLOB` class that are not available on `javax.sql.Clob` to stream the bytes to / from the database.  
If you tried to use the methods that should have been used on the `javax.sql.Clob` class you got some sort of error, an Unsupported Driver Operation or the like (Haven't done this for a while and my memory's a bit rusty on it)

Since the 10g driver the oracle `CLOB` and `BLOB` classes both implement the jdbc interface properly, so you can just use the jdbc classes/interfaces  
instead of putting oracle specific classes in your code.

This allows you to use

{% highlight java %}
javax.sql.Clob clob = res.getClob(1);
{% endhighlight %}

and for writing using the jdbc methods…

{% highlight java %}
out = clob.setAsciiStream(0);
{% endhighlight %}

and for reading …

{% highlight java %}
in = new BufferedInputStream(clob.getAsciiStream());
{% endhighlight %}

Recently I've had to use a clob again, and I have always wondered why can't I just go `stmt.setClob()` and `rs.getClob()` without having to use a load of byte streaming code.  Well the nice guys n gals at Oracle have done something similar for the 10g driver which I think is fantastic.

No more selecting for update, all you do is use the `setString` and `getString` methods on `PreparedStatement` and `ResultSet`.

You will need to set some oracle specific settings in the driver properties if you want to put data in that's bigger than 32k.

I found this information out [here](https://docs.oracle.com/javadb/10.10.1.2/ref/rrefclob.html){:target="_blank"} 

Nice one oracle, its about time 🙂 I wonder if they've done something similar for Blob handling!!
