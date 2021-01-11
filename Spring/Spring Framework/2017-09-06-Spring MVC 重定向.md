---
layout:  post
title:   Spring MVC 重定向
date:   2017-09-06 10:52:33
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-# Spring

---

```java
@RequestMapping(value = "test",method = RequestMethod.GET)
    public String test(){
         return "redirect:/list";
    }
    @RequestMapping(value = "list", method = RequestMethod.GET)
    public ModelAndView list() {
        ModelAndView mv = new ModelAndView("list");
        return mv;
    }
```


```java
@RequestMapping(value="/testa", method=RequestMethod.POST)
public String outputData(HttpServletRequest request){
    String userName = request.getParameter("name");
    String password = request.getParameter("pwd");
    request.setAttribute("name", userName);
    request.setAttribute("pwd", password);
    //转发到 /testb 的Controller方法(即outputDataX)上
    return "forward:/testb"; 
}


@RequestMapping(value="/testb", method=RequestMethod.POST)
public String outputDataX(HttpServletRequest request){
    return "testb";
}
```
















