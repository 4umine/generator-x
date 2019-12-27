MyBatis Generator (MBG)
=======================

[![Build Status](https://travis-ci.org/mybatis/generator.svg?branch=master)](https://travis-ci.org/mybatis/generator)
[![Coverage Status](https://coveralls.io/repos/mybatis/generator/badge.svg?branch=master&service=github)](https://coveralls.io/github/mybatis/generator?branch=master)
[![Dependency Status](https://www.versioneye.com/user/projects/561964c6a193340f2800033c/badge.svg?style=flat)](https://www.versioneye.com/user/projects/561964c6a193340f2800033c)
[![Maven central](https://maven-badges.herokuapp.com/maven-central/org.mybatis.generator/mybatis-generator/badge.svg)](https://maven-badges.herokuapp.com/maven-central/org.mybatis.generator/mybatis-generator)
[![License](http://img.shields.io/:license-apache-brightgreen.svg)](http://www.apache.org/licenses/LICENSE-2.0.html)

![mybatis-generator](http://mybatis.github.io/images/mybatis-logo.png)

本项目对 MBG 源码进行了一些改动, 增加了如下功能：

在默认生成的 xml 中添加了 `<sql id="Base_Column_Prefix_List>"` 内容，如下

```xml
<!-- 默认生成的字段列表 -->
<sql id="Base_Column_List">
id, create_time, update_time ...
</sql>

<!-- 增强后带有表前缀的字段列表 -->
<sql id="Base_Column_Prefix_List">
t.id, t.create_time, t.update_time ...
</sql>
```

作用是啥呢，举个栗子，假如用户表 `t_user`，查询所有用户

```xml
<sql id="base_column">
    <includerefid="com.xxx.UserMBGMapper.Base_Column_List" />
</sql>

<select id="listAllUser" resultMap="com.xxx.UserMBGMapper.BaseResultMap">
select 
    <include refid="base_column" />
from t_user
where delete_flag = 0
</select>
```

但实际业务中查询一般比这复杂，比如查询下过订单的用户

```xml
<select id="listAllUser" resultMap="com.xxx.UserMBGMapper.BaseResultMap">
select 
    u.id,
    u.username,
    ...
from t_user u 
inner join t_order o on o.user_id = u.id
where u.delete_flag = 0
and u.delete_flag = 0
</select>
```

可以看到，如果关联表查询就用不到 `base_column`，查询返回字段只能手动指定。

针对这个痛点，于是修改了 MBG 的源码，使得在生成 xml 多生成**带表前缀的字段** sql，如下

```xml
<!-- 增强后带有表前缀的字段列表 -->
<sql id="Base_Column_Prefix_List">
t.id, t.create_time, t.update_time ...
</sql>
```

那上面关联查询就可以这么写了

```xml
<sql id="base_column_prefix">
    <includerefid="com.xxx.UserMBGMapper.Base_Column_Prefix_List" />
</sql>

<select id="listAllUser" resultMap="com.xxx.UserMBGMapper.BaseResultMap">
select 
    <include refid="base_column_prefix" />
from t_user u 
inner join t_order o on o.user_id = u.id
where u.delete_flag = 0
and u.delete_flag = 0
</select>
```

注意：表前缀是根据表名最后一个单词的首字母定的，比如 t_user，那生成的就是 u.id, u.username 等，如果是 t_order，那生成的是 o.id，o.user_id 等。