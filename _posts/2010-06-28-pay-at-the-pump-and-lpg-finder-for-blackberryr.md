---
id: 54
title: 'Pay At The Pump and LPG Finder for BlackBerry(R)'
date: '2010-06-28T18:26:00+01:00'
author: andykayley
layout: post
guid: 'http://54.194.124.185/2010/06/28/pay-at-the-pump-and-lpg-finder-for-blackberryr/'
permalink: /2010/06/28/pay-at-the-pump-and-lpg-finder-for-blackberryr/
blogger_blog:
    - andykayley.blogspot.com
blogger_author:
    - 'Andy Kayley'
blogger_permalink:
    - /2010/06/pay-at-pump-and-lpg-finder-for.html
blogger_internal:
    - /feeds/6447184396655674320/posts/default/1018040448320718904
restapi_import_id:
    - 58ebf9b1175ef
original_post_id:
    - '54'
categories:
    - Abstractec
    - BlackBerry
    - iPhone
    - J2ME
    - java
    - 'Pay At The Pump'
    - Uncategorized
---

For the past few months on and off I've been working on a project for my friend and former colleague [John Haselden](http://happydevilsnackshop.blogspot.com/){:target="_blank"} of [Abstractec](http://www.abstractec.co.uk/){:target="_blank"}. He has made a website and an iPhone phone application which utilises web services of the website. The project is called Fuelista, and the applications are [Pay at the Pump](http://www.payatthepump.co.uk/){:target="_blank"} and [iPayAtThePump](http://www.whatsoniphone.com/reviews/ipayatthepump){:target="_blank"} for iPhone, (with variants for [LPG](http://www.lpgfinder.co.uk/){:target="_blank"} etc).

The idea of the Pay at the Pump and LPG Finder applications is to provide a location based service to allow you to find petrol stations that provide a “pay at the pump” or “LPG” service. The application does a GPS lookup on your location, and returns a list of garages that provide that service. You can then navigate through and get more details of the garage, and if you require you can a route from your current location to the garage.

Abstractec are trying out more platforms than just the iPhone. They are investigating Android, BlackBerry and PalmPre. I was asked if I would like a go at the BlackBerry port, so I thought “why not?”

I wanted to get into writing iPhone applications for fun and maybe a bit of extra cash, I bought myself a Macbook Pro and an iPhone in mid 2009, but there is a lot to learn to get up and running with iPhone development, the applications are written in Objective-C and the development environment is quite difficult to get your head around when you're starting off, especially coming from a Java background.  
BlackBerry applications however are written using Java.

Research In Motion (RIM) provide a layer on top of J2ME providing their own classes and UI components specifically for the BlackBerry handsets.

The application is complete and also I have created the LPG equivalent. Check out the video below for a demo of the application. The GPS lookup on the handset that I have to test with is quite poor…Hence the timeout when the first lookup occurs.

Enjoy…

<iframe width="480" height="360" src="https://youtube.com/embed/qRBemxmqxPk" frameborder="0" allowfullscreen> </iframe>


Update 9th December 2018 – I dug out the code for this little app and it's available [here](https://github.com/kaylanx/bb_patp){:target="_blank"} if anyone is interested.

{% include archive.html %}