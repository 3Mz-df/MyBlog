---
title: MybatisPlus
description: MybatisPlus  
published: 2026-10-29
image: ''
tags: []
category: '技术学习'
draft: false
lang: ''
slug: MybatisPlus
---
基于Mybatis的简单增删改查的优化
  pom里加个配置的事
  让Mapper继承BaseMapper<User>
  工作原理，常用注解：
  ![约定大于配置](image-3.png)
  不符合约定的实体类就得自己定义表名字段名了
  @TableName("实际表名") 指定表名
  @TableField("实际字段名") 指定表中普通字段信息  
  @TableId("实际主键字段名") 指定表中主键字段信息


