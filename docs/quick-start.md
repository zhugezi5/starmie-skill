# Starmie Framework 快速入门

5 分钟上手 starmie-framework，快速构建 Spring Boot + MyBatis 应用。

---

## 1. 添加依赖

在 `pom.xml` 中配置：

```xml
<parent>
    <groupId>io.github.starmoon1617</groupId>
    <artifactId>starmie-bom</artifactId>
    <version>2.0.0</version>
</parent>

<dependencies>
    <!-- 核心依赖 -->
    <dependency>
        <groupId>io.github.starmoon1617</groupId>
        <artifactId>starmie-core-service</artifactId>
        <version>2.0.0</version>
    </dependency>
    <dependency>
        <groupId>io.github.starmoon1617</groupId>
        <artifactId>starmie-app-web</artifactId>
        <version>2.0.0</version>
    </dependency>
    
    <!-- Spring Boot -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    
    <!-- MyBatis -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
    </dependency>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
    </dependency>
    
    <!-- MySQL -->
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
    </dependency>
</dependencies>
```

---

## 2. 创建实体类

```java
package com.example.entity;

import io.github.starmoon1617.starmie.core.base.BaseEntity;
import java.util.Date;

public class User extends BaseEntity<Long, Long> {
    
    private static final long serialVersionUID = 1L;
    
    private String username;
    private String email;
    private Integer status;
    private Date createTime;
    
    // getters and setters
    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
    
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    
    public Integer getStatus() { return status; }
    public void setStatus(Integer status) { this.status = status; }
    
    public Date getCreateTime() { return createTime; }
    public void setCreateTime(Date createTime) { this.createTime = createTime; }
}
```

---

## 3. 创建 Mapper

```java
package com.example.mapper;

import io.github.starmoon1617.starmie.core.mapper.BaseMapper;
import com.example.entity.User;

public interface UserMapper extends BaseMapper<User, Long, Long> {
    // 基础 CRUD 方法已由 BaseMapper 提供
}
```

对应的 MyBatis XML：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mapper.UserMapper">
    
    <resultMap id="BaseResultMap" type="com.example.entity.User">
        <id column="id" property="id"/>
        <result column="username" property="username"/>
        <result column="email" property="email"/>
        <result column="status" property="status"/>
        <result column="create_time" property="createTime"/>
    </resultMap>
    
    <sql id="Base_Column_List">
        id, username, email, status, create_time
    </sql>
    
    <select id="select" resultMap="BaseResultMap">
        SELECT <include refid="Base_Column_List"/>
        FROM user WHERE id = #{id}
    </select>
    
    <insert id="insert" useGeneratedKeys="true" keyProperty="id">
        INSERT INTO user (username, email, status, create_time)
        VALUES (#{username}, #{email}, #{status}, #{createTime})
    </insert>
    
    <update id="update">
        UPDATE user SET 
            username = #{username},
            email = #{email},
            status = #{status}
        WHERE id = #{id}
    </update>
    
    <delete id="delete">
        DELETE FROM user WHERE id = #{id}
    </delete>
    
    <!-- 条件查询需要配合 BaseCriteria 使用 -->
    <select id="selectList" resultMap="BaseResultMap">
        SELECT <include refid="Base_Column_List"/>
        FROM user
        <!-- 此处添加动态条件 -->
    </select>
    
    <select id="count" resultType="int">
        SELECT COUNT(*) FROM user
        <!-- 此处添加动态条件 -->
    </select>
    
</mapper>
```

---

## 4. 创建 Service

```java
// 接口
package com.example.service;

import io.github.starmoon1617.starmie.core.service.BaseService;
import com.example.entity.User;

public interface UserService extends BaseService<User, Long, Long> {
}

// 实现
package com.example.service.impl;

import io.github.starmoon1617.starmie.core.service.BaseServiceImpl;
import io.github.starmoon1617.starmie.core.mapper.BaseMapper;
import com.example.entity.User;
import com.example.mapper.UserMapper;
import com.example.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class UserServiceImpl extends BaseServiceImpl<User, Long, Long> implements UserService {
    
    @Autowired
    private UserMapper userMapper;
    
    @Override
    protected BaseMapper<User, Long, Long> getMapper() {
        return userMapper;
    }
}
```

---

## 5. 创建 Manager

```java
// 接口
package com.example.manager;

import io.github.starmoon1617.starmie.core.manager.BaseManager;
import com.example.entity.User;

public interface UserManager extends BaseManager<User, Long, Long> {
}

// 实现
package com.example.manager.impl;

import io.github.starmoon1617.starmie.core.manager.BaseManagerImpl;
import io.github.starmoon1617.starmie.core.service.BaseService;
import com.example.entity.User;
import com.example.manager.UserManager;
import com.example.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class UserManagerImpl extends BaseManagerImpl<User, Long, Long> implements UserManager {
    
    @Autowired
    private UserService userService;
    
    @Override
    protected BaseService<User, Long, Long> getService() {
        return userService;
    }
}
```

---

## 6. 创建 Controller

```java
package com.example.controller;

import io.github.starmoon1617.starmie.core.app.web.AbstractBaseController;
import io.github.starmoon1617.starmie.core.manager.BaseManager;
import com.example.entity.User;
import com.example.manager.UserManager;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/user")
public class UserController extends AbstractBaseController<User, Long, Long> {
    
    @Autowired
    private UserManager userManager;
    
    @Override
    protected BaseManager<User, Long, Long> getManager() {
        return userManager;
    }
    
    @Override
    protected String getPath() {
        return "user";
    }
}
```

---

## 7. 使用查询条件

```java
import io.github.starmoon1617.starmie.core.criterion.BaseCriteria;
import io.github.starmoon1617.starmie.core.criterion.enums.OperatorType;
import io.github.starmoon1617.starmie.core.criterion.enums.SortType;
import io.github.starmoon1617.starmie.core.page.Pagination;

// 构建查询条件
BaseCriteria criteria = BaseCriteria.getInstance();
criteria.addEqual("status", 1);                          // status = 1
criteria.addLike("username", "john");                    // username LIKE '%john%'
criteria.addCriterion(OperatorType.GT, "createTime", startDate);  // create_time > ?
criteria.addSortCriterion("createTime", SortType.DESC);  // ORDER BY create_time DESC

// 列表查询
List<User> users = userManager.find(criteria);

// 分页查询
Pagination<User> page = new Pagination<>();
page.setPageNo(0);
page.setPageSize(10);
userManager.find(page, criteria);
// 结果在 page.getElms() 中
```

---

## 8. 可用的 API 端点

继承 `AbstractBaseController` 后，自动获得以下端点：

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | `/user/list` | 分页列表 |
| POST | `/user/save` | 新增 |
| POST | `/user/update` | 更新 |
| POST | `/user/delete` | 删除 |
| GET | `/user/toList` | 跳转列表页 |
| GET | `/user/toAdd` | 跳转新增页 |
| GET | `/user/toEdit` | 跳转编辑页 |
| GET | `/user/toDelete` | 跳转删除页 |

---

## 下一步

- 查看 [完整 API 参考](starmie-framework-skill.md)
- 了解 [Excel 导出功能](starmie-framework-skill.md#5-starmie-utils-poi---excel-工具)
- 配置 [线程池](starmie-framework-skill.md#7-starmie-boot-executor---线程池自动配置)
