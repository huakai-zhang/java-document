---
layout:  post
title:   JSP&Servlet 统计在线人数及信息
date:   2017-07-28 08:59:27
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-JAVA

---


```java
@WebListener
    public class MyHttpSessionListener implements HttpSessionListener {
        private int userNumber = 0;

        @Override
        public void sessionCreated(HttpSessionEvent arg0) {
            userNumber++;
            arg0.getSession().getServletContext().setAttribute("userNumber", userNumber);
        }

        @Override
        public void sessionDestroyed(HttpSessionEvent arg0) {
            userNumber--;
            arg0.getSession().getServletContext().setAttribute("userNumber", userNumber);
            ArrayList<User> userList = null;//在线用户List
            userList = (ArrayList<User>) arg0.getSession().getServletContext().getAttribute("userList");
            if (SessionUtil.getUserBySessionId(userList, arg0.getSession().getId()) != null) {
                userList.remove(SessionUtil.getUserBySessionId(userList, arg0.getSession().getId()));
            }
        }
    }
```

```java
@WebListener
    public class MyServletRequestListener implements ServletRequestListener {
        private ArrayList<User> userList;//在线用户List

        @Override
        public void requestDestroyed(ServletRequestEvent arg0) {
        }

        @Override
        public void requestInitialized(ServletRequestEvent arg0) {
            userList = (ArrayList<User>) arg0.getServletContext().getAttribute("userList");
            if (userList == null)
                userList = new ArrayList<User>();
            HttpServletRequest request = (HttpServletRequest) arg0.getServletRequest();
            String sessionIdString = request.getSession().getId();
            if (SessionUtil.getUserBySessionId(userList, sessionIdString) == null) {
                User user = new User();
                user.setSessionIdString(sessionIdString);
                user.setFirstTimeString(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date()));
                user.setIpString(request.getRemoteAddr());
                userList.add(user);
            }
            arg0.getServletContext().setAttribute("userList", userList);
        }
    }
```

实现请求监听中的getUserBySessionId方法，创建单独的类SessionUtil实现：



```java
public class SessionUtil {
        public static Object getUserBySessionId(ArrayList<User> userList, String sessionIdString) {
            for (int i = 0; i < userList.size(); i++) {
                User user = userList.get(i);
                if (user.getSessionIdString().equals(sessionIdString)) {
                    return user;
                }
            }
            return null;
        }
    }
```



```java
<body>
    当前在线用户人数:${userNumber }<br/>
    <% 
   ArrayList<com.imooc.entity.User>  userList =  (ArrayList<com.imooc.entity.User>)request.getServletContext().getAttribute("userList"); 
   if(userList!=null){
       for(int i = 0 ; i < userList.size() ; i++){
      com.imooc.entity.User user = userList.get(i);
   %>
    IP:<%=user.getIpString() %>,FirstTime:<%=user.getFirstTimeString() %>,SessionId:<%=user.getSessionIdString() %> <br/>
    <%}} %>
  </body>
```










