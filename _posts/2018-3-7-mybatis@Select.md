---
layout: default
title: "@Select注解中当参数为空则不添加该参数的判断"
tags: mybatis
---

## mybatis @Select注解中当参数为空则不添加该参数的判断


``
public interface OrderMapper extends SqlMapper{
@Select("select * from tbl_order where room like #{room} and mydate like #{mydate}")
public List<Order> getbyroom(OrderPara op);
}
``
这样整个语句是写死的，必须有2个参数，在这种模式下，如何能实现根据room和mydate是否为空来动态的拼写sql语句
比如当
````
mydate=""
Select("select * from tbl_order where room like #{room} ")
public List<Order> getbyroom(OrderPara op);
````
如果用xml来配置语句的话，可以用<when test="title != null">
and mydate= #{mydate}
</when>
如果是用@Select 这种 改如何做呢？



个人所知有2种解决方法：
1. 用script标签包围，然后像xml语法一样书写

````
@Select({"<script>",
"SELECT * FROM tbl_order",
"WHERE 1=1",
"<when test='title!=null'>",
"AND mydate = #{mydate}",
"</when>",
"</script>"})
````
2. 用Provider去实现SQL拼接，例如：

````
public class OrderProvider {
    private final String TBL_ORDER = "tbl_order";
    public String queryOrderByParam(OrderPara param) {
        SQL sql = new SQL().SELECT("*").FROM(TBL_ORDER);
        String room = param.getRoom();
        if (StringUtils.hasText(room)) {
          sql.WHERE("room LIKE #{room}");
        }
        Date myDate = param.getMyDate();
        if (myDate != null) {
            sql.WHERE("mydate LIKE #{mydate}");
        }
        return sql.toString();
    }
}    

public interface OrderDAO {
    @SelectProvider(type = OrderProvider.class, method = "queryOrderByParam")
    List<Order> queryOrderByParam(OrderParam param);
}
````


注意：方式1有个隐患就是当传入参数为空的时候，可能会造成全表查询。
复杂SQL用方式2会比较灵活（当然，并不建议写复杂SQL），而且可以抽象成通用的基类，使每个DAO都可以通过这个基类实现基本的通用查询，原理类似Spring JDBC Template。
