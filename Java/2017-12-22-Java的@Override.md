---
layout:  post
title:   Java的@Override
date:   2017-12-22 14:25:18
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-Java基础
-注释
-java
-Override

---




@Override表示重写(当然不写也可以)，是覆盖的意思，这是jdk自带的一个注解。表示该方法是继承过来或者实现的方法，如果加了该注解，它的父类或者实现的接口中没有该方法，则ide会报错。不加也可以，但是加了增强了可读性，并且是一种强制性的覆盖。 这种机制其实是将运行期的错误放到编译期进行处理了。  好处:  1、可以当注释用,方便阅读；  2、编译器可以给你验证@Override下面的方法名是否是你父类中所有的，如果没有则报错。例如，你如果没写@Override，而你下面的方法名又写错了，这时你的编译器是可以编译通过的，因为编译器以为这个方法是你的子类中自己增加的方法。  例如：  ![img](https://img-blog.csdn.net/20171222142336975?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3N6Y3kxOTk1MDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


![img](https://img-blog.csdn.net/20171222142421609?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3N6Y3kxOTk1MDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

