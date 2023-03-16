---
layout:  post
title:   JSP&Servlet servlet生成报表
date:   2017-07-28 09:06:31
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-JAVA
-sql
-xml
-jsp
-serlvet

---









商品信息表


![img](https://img-blog.csdn.net/20170728090355000?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3N6Y3kxOTk1MDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

​买家信息表


![img](https://img-blog.csdn.net/20170728090424928?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3N6Y3kxOTk1MDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

​卖家信息表


![img](https://img-blog.csdn.net/20170728090437194?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3N6Y3kxOTk1MDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)




```java
public class Service {
        private Connection conn;
        private Statement st, st1, st2;
        private ResultSet re, re1, re2;
        private String sql;
        private List list;
        private Profit pf;

        public List getProfit() {
            list = new ArrayList();
            conn = DBOperator.getConnection();
            try {
                st = (Statement) conn.createStatement();
                st1 = (Statement) conn.createStatement();
                st2 = (Statement) conn.createStatement();
                sql = "SELECT g.GOOD_NAME goodsName,g.SELLING_PRICE selling,g.COST_PRICE costPrice,g.GOOD_ID goodsId " +
                        "FROM goods g,trading t " +
                        "WHERE t.TRADING_GOODS_ID=g.GOOD_ID " +
                        "GROUP BY g.GOOD_NAME,g.SELLING_PRICE,g.COST_PRICE,g.GOOD_ID";
                re = st.executeQuery(sql);
                int temp;
                while (re.next()) {
                    pf = new Profit();
                    pf.setGoodName(re.getString("goodsName"));
                    pf.setSellingPrice(re.getInt("selling"));
                    pf.setCostPrice(re.getInt("costPrice"));
                    pf.setGoodsId(re.getString("goodsId"));
                    temp = 0;
                    temp = pf.getSellingPrice() - pf.getCostPrice();
                    sql = "SELECT SUM(t.TRADING_NUMBER) sumNum " +
                            "FROM trading t " +
                            "WHERE t.TRADING_GOODS_ID=" + pf.getGoodsId();
                    re1 = st1.executeQuery(sql);
                    while (re1.next()) {
                        pf.setTradingNum(re1.getInt("sumNum"));
                    }
                    pf.setPorfit(temp * pf.getTradingNum());
                    sql = "SELECT COUNT(t.TRADING_ID) times " +
                            "FROM trading t " +
                            "WHERE t.TRADING_GOODS_ID=" + pf.getGoodsId();
                    re2 = st2.executeQuery(sql);
                    while (re2.next()) {
                        pf.setTimes(re2.getInt("times"));
                    }
                    list.add(pf);
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
            return list;
        }
    }
```

```java
public void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        List list;
        Service service = new Service();
        list = service.getProfit();
        request.getSession().setAttribute("PROFIT", list);
        response.sendRedirect("index.jsp");
    }
```

```java
<style type="text/css">
table.hovertable{
font-family: verdana,arial,sans-serif;
font-size: 13px;
color: #333333;
border-width: 1px;
border-color: #999999;
border-collapse: collapse;
}
table.hovertable th{
background-color: #c3dde0;
border-width: 1px;
padding: 8px;
border-style: solid;
border-color: #a9c6c9;
}
table.hovertable td{
border-style:solid;
border-width: 1px;
padding: 8px;
border-color: #a9c6c9;
}
table.hovertable tr{
background-color: #d4e3e5;
}
</style>
  </head><body>


    <input type="submit" value="生成报表">


    <table class="hovertable"><tr>
    <th colspan="5" style="text-align: center;">利润表</th>
    </tr><tr>
    <th>序号</th>
    <th>商品名称</th>
    <th>卖出数量</th>
    <th>交易笔数</th>
    <th>盈利额</th>
    </tr>
    <%
    List list = null;
    if(session.getAttribute("PROFIT")!=null){
    list = (List)session.getAttribute("PROFIT");
    if(list.size()>0){
    int temp，temp1，temp2,temp3= 0;
    Profit pf;
    for(int i=0;i
    pf = new Profit();
    pf=(Profit)list.get(i);
    temp1+=pf.getTradingNum();
    temp2+=pf.getTimes();
    temp3+=pf.getPorfit();
    %>
     <tr οnmοuseοver="this.style.backgroundColor='#ffff66';"
     οnmοuseοut="this.style.backgroundColor='#d4e3e5';">
    <td><%= temp+=1 %></td>
    <td><%= pf.getGoodName() %></td>
    <td><%= pf.getTradingNum() %></td>
    <td><%= pf.getTimes() %></td>
    <td><%= pf.getPorfit() %></td>
    </tr>
    <%}%>
    <tr οnmοuseοver="this.style.backgroundColor='#ffff66';"
     οnmοuseοut="this.style.backgroundColor='#d4e3e5';">
    <td colspan="2">合计</td>
    <td><%= temp1%></td>
    <td><%= temp2 %></td>
    <td><%= temp3 %></td>
    </tr>
    <%}  }
     %></table></body>
```

效果：


![img](https://img-blog.csdn.net/20170728090615808?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3N6Y3kxOTk1MDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

