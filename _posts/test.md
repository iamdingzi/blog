---
layout: default
title: "test"
tags: mybatis
---

## test

自己踩过的坑，test，终于发现解决方法。
步骤有三：
1. 写SQL，但不要自己插入主键值
2. 配置@Options(useGeneratedKeys=true, keyProperty="对象.属性") 这个的作用是设置是否使用JDBC的getGenereatedKeys()方法获取主键并赋值到keyProperty设置的对象的属性中，说白了就是把自增长的主键值赋值给对象相应的属性
3. 在插入后，使用对象.主键属性的getXXId()方法 获取主键值
示例：
``
@Insert("INSERT INTO api(component, clientApi, scribeDescribe, clientApiVersion, invokeTimeout, authType, sessionRequire, openType) VALUES(#{api.component}, #{api.clientApi}, #{api.scribeDescribe}, #{api.clientApiVersion}, #{api.invokeTimeout}, #{api.authType}, #{api.sessionRequire}, #{api.openType})")
@Options(useGeneratedKeys = true, keyProperty = "api.apiId")
void add(@Param("api") Api api);
``
然后调用api.getApiId()即可
