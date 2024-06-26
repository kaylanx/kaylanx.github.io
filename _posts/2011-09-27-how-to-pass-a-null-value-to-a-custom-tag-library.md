---
title: 'How To Pass A Null Value To A Custom Tag Library'
layout: post
---

Today I found something very interesting. I wanted to pass a `java.math.BigDecimal` to a custom JSP Tag Library and format it as a percentage, however if the `BigDecimal` was `null` I didn't want to show it.

Easy, I thought. So I created a class similar to the following…

{% highlight java %}
package name.kayley.blog.taglib;

import javax.servlet.jsp.JspException;
import javax.servlet.jsp.tagext.TagSupport;
import java.io.IOException;
import java.math.BigDecimal;

public class BigDecimalTagSupport extends TagSupport {

    private BigDecimal value;

    public void setValue(BigDecimal value) {
        this.value = value;
    }

    public int doEndTag() throws JspException {
        try {
            if (value != null) {
                pageContext
                        .getOut()
                        .print(formatAsPercentage(value));
            }
        } catch (IOException e) {
            throw new JspException(e);
        }
        return EVAL_PAGE;
    }
// ...
}
{% endhighlight %}

However, when I called this taglib with a `null` value I still got a value of 0.

{% highlight xml %}
<mytag:bigDecimal value="${aBigDecimal}" />
{% endhighlight %}

The output:  
`0`

After a bit of trial and error I discovered that the JSP tab library framework does a little bit of magic on known classes, such as `String`s and `Numeric` objects. Classes of `Numeric` types that are `null` get converted to `0` and `String`s get converted into the empty string `""`

I discovered that if you change your internal value to an `java.lang.Object` instead of a `java.math.BigDecimal` the magic doesn't know what to do and just passes `null` to your class.

So the new class looks like the following…

{% highlight java %}
package name.kayley.blog.taglib;

import javax.servlet.jsp.JspException;
import javax.servlet.jsp.tagext.TagSupport;
import java.io.IOException;

public class BigDecimalTagSupport extends TagSupport {

    private Object value;

    public void setValue(Object value) {
        this.value = value;
    }

    public int doEndTag() throws JspException {
        try {
            if (value != null) {
                // Cast value to a BigDecimal 
                // if you need to.
                pageContext
                        .getOut()
                        .print(formatAsPercentage(value));
            }
        } catch (IOException e) {
            throw new JspException(e);
        }
        return EVAL_PAGE;
    }
//...
}
{% endhighlight %}

So now when I use the tag and pass a `null BigDecimal` to it I get the required result of no output.

WIN

I have an example web app demonstrating this, if anyone wants a copy just let me know.
