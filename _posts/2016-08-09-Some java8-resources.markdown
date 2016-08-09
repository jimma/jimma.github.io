---
layout:     post
title:      "java8 is faster"
subtitle:   ""
date:       2016-08-08 12:00:00
author:     "Jim Ma"
header-img: "img/post-bg-06.jpg"
---
Java8 is faster?
===================

lambda and stream's performance
----------

java8 has a lot of cool features to help write less code to get some complex function. From [java8 performance test](http://blog.takipi.com/benchmark-how-java-8-lambdas-and-streams-can-make-your-code-5-times-slower/?utm_source=twitter&utm_medium=maintweet&utm_content=functionalbenchmark&utm_campaign=java), if you write it correctly it will improve the performance. If you write it not well, it slows 5x times performance.  From [improvement change](https://github.com/takipi/loops-jmh-playground/commit/d91c55aee2d480c31fa7fad72f6f0b94fb59612e) , we can see we need to take care of  auto-boxing. It is better to use the out-of-box apis like parallel stream of intStream etc in JDK8. And the performance is much more better than your own composing lambda expression:
![from blog.takipi.com](http://384uqqh5pka2ma24ild282mv.wpengine.netdna-cdn.com/wp-content/uploads/2015/11/remake.png)
This blog is really nice performance comparison between old java7 style and java8 stream lambda features.  With java8 is more and more adopted, we'll get more and more experience of using these features. I am looking forward to read the new edition of this book :
![enter image description here](https://images-na.ssl-images-amazon.com/images/I/51sRluR-aOL._SX359_BO1,204,203,200_.jpg)
   
java8 resources
----------
As learn other technology , the in action series book is recommended : 
![enter image description here](https://images-na.ssl-images-amazon.com/images/I/51JNZZSTmFL._SX397_BO1,204,203,200_.jpg)

As said in this book's review: 

> This is a concise and well-written introduction to Java 8.
> Blockquote

it gives more detailed and complete explanation about java8 new added apis. 
Beside this book, I recommend  [the oracle's lambda expression tutorial](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html) .  This is very easy to read. The examples is great to explain why lambda. If you haven't get this book, read this first.
