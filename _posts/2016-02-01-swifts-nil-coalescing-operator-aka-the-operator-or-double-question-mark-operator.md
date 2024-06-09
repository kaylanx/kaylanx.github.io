---
id: 48
title: 'Swift''s Nil Coalescing Operator AKA the ''?? Operator'' or ''Double Question Mark Operator'''
date: '2016-02-01T22:44:00+00:00'
author: andykayley
layout: post
guid: 'http://54.194.124.185/2016/02/01/swifts-nil-coalescing-operator-aka-the-operator-or-double-question-mark-operator/'
permalink: /2016/02/01/swifts-nil-coalescing-operator-aka-the-operator-or-double-question-mark-operator/
blogger_blog:
    - andykayley.blogspot.com
blogger_author:
    - 'Andy Kayley'
blogger_permalink:
    - /2016/02/swifts-nil-coalescing-operator-aka.html
blogger_internal:
    - /feeds/6447184396655674320/posts/default/3459606977197448211
restapi_import_id:
    - 58ebf9b1175ef
original_post_id:
    - '48'
categories:
    - 'andy kayley'
    - 'double question mark'
    - iOS
    - nil
    - 'nil coalescing'
    - 'null'
    - Objective-C
    - Swift
    - Uncategorized
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

{% include archive.html %}