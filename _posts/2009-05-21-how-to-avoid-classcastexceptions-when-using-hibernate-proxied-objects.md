---
id: 55
title: 'How to Avoid ClassCastExceptions when using Hibernate Proxied Objects'
date: '2009-05-21T23:54:00+01:00'
author: andykayley
layout: post
guid: 'http://54.194.124.185/2009/05/21/how-to-avoid-classcastexceptions-when-using-hibernate-proxied-objects/'
permalink: /2009/05/21/how-to-avoid-classcastexceptions-when-using-hibernate-proxied-objects/
blogger_blog:
    - andykayley.blogspot.com
blogger_author:
    - 'Andy Kayley'
blogger_permalink:
    - /2009/05/how-to-avoid-classcastexceptions-when.html
blogger_internal:
    - /feeds/6447184396655674320/posts/default/990719597758264895
restapi_import_id:
    - 58ebf9b1175ef
original_post_id:
    - '55'
categories:
    - ClassCastException
    - EnhancerByCGLIB
    - hibernate
    - java
    - patterns
    - spring
    - Uncategorized
---

Recently at work I was faced with an issue on the live system. A certain page within the application would only show the system's standard error message. After looking in the logs I discovered there was a `ClassCastException` being thrown, this was due to the fact that a lazily loaded List of objects retrieved by hibernate contained proxied objects. The entities that were proxied were using a single table inheritance strategy, so there is a base class that is subclassed several times. The entities once loaded were being sorted into separate lists that are typed according to the subclasses, they were being cast into the appropriate subclass but this cast was failing because instead of being an instance of say `Animal` it was an instance of `Animal$$EnhancerByCGLIB$$10db1483` i.e. A Hibernate Proxy. Let me show you an example.

So lets say we have a base class that has an inheritance strategy set…

{% highlight java %}
package name.kayley.cglibexample;

import javax.persistence.*;

@Entity
@Table(name="animals")
@Inheritance(strategy=InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(
    name="animal_type_id",
    discriminatorType=DiscriminatorType.INTEGER
)
public class Animal {

    @Id
    @GeneratedValue
    private Long id;
    
    @Column
    private String name;
    
    @Column
    private Integer age;
    
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
{% endhighlight %}

And then we had 2 subclasses of this `Animal` class, lets say, `Cat` and `Dog`…

{% highlight java %}
package name.kayley.cglibexample;

import javax.persistence.DiscriminatorValue;
import javax.persistence.Entity;

@Entity
@DiscriminatorValue("1")
public class Cat extends Animal {
   // some cat specific attributes
}
{% endhighlight %}


{% highlight java %}
package name.kayley.cglibexample;

import javax.persistence.DiscriminatorValue;
import javax.persistence.Entity;

@Entity
@DiscriminatorValue("2")
public class Dog extends Animal {
   // some dog specific attributes
}
{% endhighlight %}

As you can see the discriminator column is `animal_type_id`, which I have defined 2 to be `Dog` and 1 to be `Cat`. So now we should make sure we have some data in a database table called animals…

```
SELECT id,animal_type_id,name,age FROM animals a LIMIT 0,1000

'id', 'animal_type_id', 'name', 'age'
1, 2, 'Scooby', 3
2, 1, 'Felix', 4
3, 1, 'Garfield', 4
```

Right so to recreate a similar situation I will show you a unit test that will pass! What I am trying to do is to retrieve the list of animals and sort them in the java into separate lists, a list of dogs and a list of cats..

{% highlight java %}
public void testCGLibInstanceOfError() throws Exception {

    List dogs = new ArrayList();
    List cats = new ArrayList();

    Iterator it = hibernateDao.createQuery("from Animal").iterate();

    while (it.hasNext()) {

        Animal animal = it.next();
        assertTrue(animal.getClass().getName().contains("Animal$$EnhancerByCGLIB$$"));
                    
        if (animal instanceof Cat) {
            cats.add((Cat) animal);
        } 
        else if (animal instanceof Dog) {
            dogs.add((Dog) animal);
        }
    }
    assertEquals(0, dogs.size());
    assertEquals(0, cats.size());
}
{% endhighlight %}

Now using the data in the database we want a list of 1 dog and a list of 2 cats. Here I'm being particularly evil and using instanceof to differentiate these. The `assertEquals` at the end of the test pass, even though the code looks like it should be adding items to these lists based on what is in the database. This is because animal is not an `instanceof Cat` or `Dog` but an `instanceof Animal$$EnhancerByCGLIB$$10db1483`.

So how do we solve this issue?

I first thought about using `Hibernate.initialize(animal);` which will force the animal class to become the actual object, not the proxy. I felt this approach to be dirty and it is really polluting code that shouldn't really know anything about hibernate.

So I did some googling and after reading [the hibernate documentation](https://www.hibernate.org/280.html){:target="_blank"} and then [this](http://en.wikipedia.org/wiki/Visitor_pattern){:target="_blank"} I set about using the Gang Of Four Visitor Pattern.

First we need a visitor interface that we can implement in your test…

{% highlight java %}
package name.kayley.cglibexample;

public interface AnimalEntityVisitor {
    void visit(Dog dog);
    void visit(Cat cat);
}
{% endhighlight %}
Next we need is an interface for our `Animal` class to implement…

{% highlight java %}
package name.kayley.cglibexample;

public interface AnimalEntity {
    void accept(AnimalEntityVisitor visitor);
}
{% endhighlight %}

Now we make our `Animal` class I showed earlier implement the `AnimalEntity`…

{% highlight java %}
package name.kayley.cglibexample;

import javax.persistence.*;

@Entity
@Table(name="animals")
@Inheritance(strategy=InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(
    name="animal_type_id",
    discriminatorType=DiscriminatorType.INTEGER
)
public abstract class Animal implements AnimalEntity {
   ...
   // see top of post
   ...
}
{% endhighlight %}

And next implement the method in the subclasses..

{% highlight java %}
package name.kayley.cglibexample;

import javax.persistence.DiscriminatorValue;
import javax.persistence.Entity;

@Entity
@DiscriminatorValue("1")
public class Cat extends Animal {
   // some cat specific attributes

   public void accept(AnimalEntityVisitor visitor) {
      visitor.visit(this);
   }
}
{% endhighlight %}

{% highlight java %}
package name.kayley.cglibexample;

import javax.persistence.DiscriminatorValue;
import javax.persistence.Entity;

@Entity
@DiscriminatorValue("2")
public class Dog extends Animal {
   // some dog specific attributes

   public void accept(AnimalEntityVisitor visitor) {
      visitor.visit(this);
   }
}
{% endhighlight %}

All we need to do is implement the visitor in our test…

{% highlight java %}
public class MyTest 
    extends AbstractTransactionalDataSourceSpringContextTests 
    implements AnimalEntityVisitor {

        ...
    // some spring setup     
        ...

    private List dogs = new ArrayList();
    private List cats = new ArrayList();
    
    public void testCGLibUsingVisitor() throws Exception {

        Iterator it = hibernateDao.createQuery("from Animal").iterate();

        while (it.hasNext()) {

            Animal animal = it.next();

            assertTrue(animal.getClass().getName().contains("Animal$$EnhancerByCGLIB$$"));

            animal.accept(this);
            
        }
        assertEquals(1, dogs.size());
        assertEquals(2, cats.size());
    }

    public void visit(Dog dog) {
        dogs.add(dog);
    }

    public void visit(Cat cat) {
        cats.add(cat);
    }
}
{% endhighlight %}

So instead of using `instanceof` or having to cast anything (which is what was happening in the live code I talked about) which can cause weird side effects and ClassCastExceptions, you can just use this visitor pattern. What happens is the test is implementing the `AnimalEntityVisitor`, so when `animal.accept(this);` is called, it calls the appropriate visit method on `this`. In our case the visit method is just adding the object to a list. So I've updated the `assertEquals` calls to be what I now expect and the test passes.  

Happy days, enjoy!  
