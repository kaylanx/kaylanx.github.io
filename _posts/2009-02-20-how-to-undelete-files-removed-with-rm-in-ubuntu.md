---
title: 'How to Undelete files removed with &#8220;rm&#8221; in Ubuntu'
layout: post
---

Recently I found myself in a situation where I needed to recover some files I accidentally deleted. I had written a program in Java to read off a JBoss Messaging Dead Letter Queue, and save them in a format that could be re-inputted back into the system.

So my program was writing files into a directory within my home directory, and while I was testing the program I was not committing back to JMS so ensure the messages stayed there. When I was confident that the program was correct, I added the commit, I then removed the files that my tests had produce by using `rm \*`.

I then ran my program for real. In the directory I was expecting files to appear I typed the usual `ls -ltr` to see a list of files. I pressed the up key to run the last command i.e. `ls -ltr` to see some more files created. I repeated this a few times, until I accidentally pressed the up key twice…..yep you guessed it….`rm \*`, the files were deleted and the commit to JMS means they were no longer on the queue. My heart sank.

Now I rememebered on the old days of `MS-DOS` there was an `UNDELETE` command, so I was looking around the web to see if there was something similar for Linux. I was in luck, especially as my program was still running.

Basically thanks to [this information](http://www.oreillynet.com/pub/h/363){:target="_blank"}, I recovered my deleted files with relative ease. 🙂

Ok so to do this, I believe you need to be as fortunate as I was and still have the program that created the deleted files still running, so let's create a Java program that will write to a file and then keep running…

{% highlight java %}
package name.kayley.undeletefiletest;

import java.io.FileWriter;
import java.io.PrintWriter;

public class UndeleteFileTest {
 public static void main(String[] args) throws Exception {  
  FileWriter writer = new FileWriter("/tmp/testFile.txt");
  
  PrintWriter printer = new PrintWriter(writer);
  
  printer.append("hello this file is going to be deleted.");
  printer.flush();
  Object lock = new Object();
  
  synchronized(lock) {
   lock.wait();
  }
 }
}
{% endhighlight %}

Ok, now run that program and it will create a file in `/tmp` called `testFile.txt`, so lets see it..

{% highlight shell %}
andy@andy-laptop:/tmp$ ls -ltr *.txt
-rw-r--r-- 1 andy andy 39 2009-02-13 18:54 testFile.txt
{% endhighlight %}

So we can see the file is there, let's delete it to see if we can recover it…

{% highlight shell %}
andy@andy-laptop:/tmp$ rm testFile.txt
{% endhighlight %}

Now the file is gone. So how do we get it back? Well first we need to find the PID of the process that created the file that was deleted. We know that it was a java program that created it so we'll use the lsof and search for java..

{% highlight shell %}
andy@andy-laptop:/tmp$ lsof | grep java
.....[lots of things listed]
java      9048       andy    4r      REG        8,1       39 1622029 /tmp/testFile.txt (deleted)
{% endhighlight %}

No now we know the PID is 9048 lets find the file… so there should be a directory /proc/\[PID\]/fd…

{% highlight shell %}
andy@andy-laptop:/tmp$ cd /proc/9048/fd
andy@andy-laptop:/proc/9048/fd$ ls -ltr
total 0
lr-x------ 1 andy andy 64 2009-02-13 18:54 4 -> /tmp/testFile.txt (deleted)
l-wx------ 1 andy andy 64 2009-02-13 18:54 3 -> /usr/local/jdk1.6.0_07/jre/lib/rt.jar
l-wx------ 1 andy andy 64 2009-02-13 18:54 2 -> pipe:[43665]
l-wx------ 1 andy andy 64 2009-02-13 18:54 1 -> pipe:[43664]
lr-x------ 1 andy andy 64 2009-02-13 18:54 0 -> pipe:[43663]
{% endhighlight %}

Bingo! We have found the deleted file, so now all we need to do is cat the link to a new file, so here I put it into my home directory…

{% highlight shell %}
andy@andy-laptop:/proc/9048/fd$ cat 4 > ~/testFile.txt
{% endhighlight %}

Next, I go to my home folder and have a look at the file, and hey presto… it's restored…

{% highlight shell %}
andy@andy-laptop:/proc/9048/fd$ cd
andy@andy-laptop:~$ less testFile.txt 
hello this file is going to be deleted.
testFile.txt (END) 
{% endhighlight %}

Phew!

