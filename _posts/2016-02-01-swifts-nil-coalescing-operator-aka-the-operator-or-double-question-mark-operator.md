---
title: 'Swift''s Nil Coalescing Operator AKA the ''?? Operator'' or ''Double Question Mark Operator'''
layout: post
---

This last week I was introduced to quite a funky little operator in Swift called the [Nil Coalescing operator](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/basicoperators/#Nil-Coalescing-Operator){:target="_blank"}. It is basically a shorthand for the ternary conditional operator when used to test for nil.

For example traditionally in many programming languages you could do something like this to check for nil/null…

{% highlight swift %}
stringVar != nil ? stringVar! : "a default string"
{% endhighlight %}

Here if the condition is not nil the value of stringVar will be returned otherwise the value “a default string” is returned.

This can be shortened in Swift by using the [Nil Coalescing operator](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/basicoperators/#Nil-Coalescing-Operator){:target="_blank"}.

{% highlight swift %}
stringVar ?? "a default string"
{% endhighlight %}

This does exactly the same thing, if stringVar **is not** nil then it is returned, however if it **is** nil the value “a default string” is returned.

Love it.
