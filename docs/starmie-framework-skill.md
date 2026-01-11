# Starmie Framework Development Skill

## 触发条件
当用户输入包含关键字 `starmie` 或涉及以下场景时,优先使用 starmie-framework 提供的接口/工具/API 进行编码:
- 创建实体类、DTO、Result类
- 实现 Mapper/Service/Manager/Controller 层代码
- 数据库查询条件构建
- 分页查询
- Excel/PDF 导入导出
- 线程池配置
- 代码生成

---

## 框架概述

**Starmie Framework** 是一个模块化的 Java 框架,基于 Spring Boot 4.x + MyBatis 3.5.x 构建,提供通用的基础功能、工具类和代码生成能力。

### 核心依赖
```xml
<parent>
    <groupId>io.github.starmoon1617</groupId>
    <artifactId>starmie-bom</artifactId>
    <version>2.0.0</version>
</parent>
```

### 技术栈版本
- Java: 25
- Spring Boot: 4.0.1
- Spring Framework: 7.0.2
- MyBatis: 3.5.19
- Apache POI: 5.5.1
- Apache PDFBox: 3.0.6
- Jackson: 3.0.3

---

## 模块结构与使用指南

### 1. starmie-core-base - 基础实体定义

**Maven 依赖:**
```xml
<dependency>
    <groupId>io.github.starmoon1617</groupId>
    <artifactId>starmie-core-base</artifactId>
    <version>2.0.0</version>
</dependency>
```

**核心类:**

#### BaseEntity - 实体基类
```java
import io.github.starmoon1617.starmie.core.base.BaseEntity;

// E = 实体类, ID = 主键类型, U = 用户标识类型
public class User extends BaseEntity<Long, Long> {
    private String username;
    private String email;
    // getters/setters
}
```

#### BaseDto - 统一返回结果
```java
import io.github.starmoon1617.starmie.core.base.BaseDto;

// 接口返回示例
public BaseDto<User> getUser(Long id) {
    BaseDto<User> result = new BaseDto<>();
    result.setData(user);
    result.setCode(0);
    result.setMessage("success");
    return result;
}
```

#### BaseResult - 基础结果类
```java
import io.github.starmoon1617.starmie.core.base.BaseResult;
// 包含 code, message 字段
```

---

### 2. starmie-core-common - 通用工具与查询条件

**Maven 依赖:**
```xml
<dependency>
    <groupId>io.github.starmoon1617</groupId>
    <artifactId>starmie-core-common</artifactId>
    <version>2.0.0</version>
</dependency>
```

#### BaseCriteria - 查询条件构建器
```java
import io.github.starmoon1617.starmie.core.criterion.BaseCriteria;
import io.github.starmoon1617.starmie.core.criterion.enums.OperatorType;
import io.github.starmoon1617.starmie.core.criterion.enums.SortType;
import io.github.starmoon1617.starmie.core.criterion.enums.LimitationType;

// 创建查询条件
BaseCriteria criteria = BaseCriteria.getInstance();

// 添加等于条件 (单值为 EQ, 多值为 IN)
criteria.addEqual("status", 1);
criteria.addEqual("type", 1, 2, 3);  // IN 条件

// 添加 LIKE 条件
criteria.addLike("username", "john");

// 添加其他操作符条件
criteria.addCriterion(OperatorType.GT, "age", 18);        // 大于
criteria.addCriterion(OperatorType.LT, "age", 60);        // 小于
criteria.addCriterion(OperatorType.GTE, "score", 80);     // 大于等于
criteria.addCriterion(OperatorType.LTE, "score", 100);    // 小于等于
criteria.addCriterion(OperatorType.NE, "status", 0);      // 不等于
criteria.addCriterion(OperatorType.BTW, "createTime", startDate, endDate);  // BETWEEN
criteria.addCriterion(OperatorType.ISN, "deletedAt");     // IS NULL
criteria.addCriterion(OperatorType.ISNN, "updatedAt");    // IS NOT NULL

// 添加排序
criteria.addSortCriterion("createTime", SortType.DESC);
criteria.addSortCriterion(1, "id", SortType.ASC);  // 带优先级的排序

// 添加分页限制
criteria.addLimitation(LimitationType.OFFSET, 0);
criteria.addLimitation(LimitationType.LIMIT, 10);
```

**OperatorType 枚举值:**
| 枚举值 | SQL 操作 | 说明 |
|--------|----------|------|
| `EQ` | `=` | 等于 |
| `NE` | `<>` | 不等于 |
| `GT` | `>` | 大于 |
| `GTE` | `>=` | 大于等于 |
| `LT` | `<` | 小于 |
| `LTE` | `<=` | 小于等于 |
| `LK` | `LIKE '%value%'` | 模糊匹配 |
| `RLKM` | `LIKE 'value%'` | 右模糊 |
| `LLKM` | `LIKE '%value'` | 左模糊 |
| `IN` | `IN (...)` | 包含 |
| `NIN` | `NOT IN (...)` | 不包含 |
| `BTW` | `BETWEEN ... AND ...` | 区间 |
| `ISN` | `IS NULL` | 为空 |
| `ISNN` | `IS NOT NULL` | 不为空 |

#### Pagination - 分页对象
```java
import io.github.starmoon1617.starmie.core.page.Pagination;

Pagination<User> pagination = new Pagination<>();
pagination.setPageNo(0);    // 页码从0开始
pagination.setPageSize(10);
// 查询后填充:
// pagination.setTotal(100);
// pagination.setTotalPage(10);
// pagination.setElms(userList);
```

#### EntityUtils - 实体工具类
```java
import io.github.starmoon1617.starmie.core.util.EntityUtils;

// 属性复制 (排除ID)
EntityUtils.copyProperties(target, source);

// 反射获取/设置属性值
Object value = EntityUtils.getValue(entity, "fieldName");
EntityUtils.setValue(entity, "fieldName", value);

// JSON 转换
String json = EntityUtils.toJson(object);
String jsonNonNull = EntityUtils.toNonNJson(object);  // 排除 null 字段
User user = EntityUtils.fromJson(jsonStr, User.class);
List<User> users = EntityUtils.fromJsonToList(jsonStr, User.class);
```

#### DateUtils - 日期工具类
```java
import io.github.starmoon1617.starmie.core.util.DateUtils;

// 获取当前日期
Date now = DateUtils.getCurrentDate();

// 日期解析 (自动识别格式)
Date date = DateUtils.parse("2024-01-01");
Date dateTime = DateUtils.parse("2024-01-01 12:00:00");
Date dateTimeMs = DateUtils.parse("2024-01-01 12:00:00.123");

// 指定格式解析
Date custom = DateUtils.parse("2024/01/01", "yyyy/MM/dd");

// 日期格式化
String dateStr = DateUtils.formatCurrentDate();         // yyyy-MM-dd
String dateTimeStr = DateUtils.formatCurrentDateTime(); // yyyy-MM-dd HH:mm:ss
String yearStr = DateUtils.formatCurrentYear();         // yyyy
String customStr = DateUtils.format("yyyyMMdd", date);
```

#### CommonUtils - 通用工具类
```java
import io.github.starmoon1617.starmie.core.util.CommonUtils;

boolean empty = CommonUtils.isEmpty(collection);
boolean notBlank = CommonUtils.isNotBlank(string);
```

---

### 3. starmie-core-service - 服务层定义

**Maven 依赖:**
```xml
<dependency>
    <groupId>io.github.starmoon1617</groupId>
    <artifactId>starmie-core-service</artifactId>
    <version>2.0.0</version>
</dependency>
```

#### BaseMapper - Mapper 接口
```java
import io.github.starmoon1617.starmie.core.mapper.BaseMapper;

public interface UserMapper extends BaseMapper<User, Long, Long> {
    // 已继承方法:
    // E select(E e)           - 根据实体查询单条
    // int insert(E e)         - 插入
    // int update(E e)         - 更新
    // int delete(E e)         - 删除
    // List<E> selectList(BaseCriteria criteria)  - 条件查询列表
    // int count(BaseCriteria criteria)           - 条件计数
}
```

#### BaseService - Service 接口
```java
import io.github.starmoon1617.starmie.core.service.BaseService;

public interface UserService extends BaseService<User, Long, Long> {
    // 已继承方法:
    // E find(E e)             - 查找单条
    // int save(E e)           - 保存
    // int delete(E e)         - 删除
    // int update(E e)         - 更新
    // List<E> find(BaseCriteria criteria)  - 条件查询
    // int count(BaseCriteria criteria)     - 条件计数
}
```

#### BaseServiceImpl - Service 实现
```java
import io.github.starmoon1617.starmie.core.service.BaseServiceImpl;

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

#### BaseManager - Manager 接口
```java
import io.github.starmoon1617.starmie.core.manager.BaseManager;

public interface UserManager extends BaseManager<User, Long, Long> {
    // 已继承方法:
    // E find(E e)             - 查找单条
    // int save(E e)           - 保存单条
    // int save(Collection<E> es)  - 批量保存
    // int delete(E e)         - 删除
    // int update(E e)         - 更新
    // void find(Pagination<E> pagination, BaseCriteria criteria)  - 分页查询
    // List<E> find(BaseCriteria criteria)  - 条件查询
    // int count(BaseCriteria criteria)     - 条件计数
}
```

#### BaseManagerImpl - Manager 实现
```java
import io.github.starmoon1617.starmie.core.manager.BaseManagerImpl;

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

### 4. starmie-app-web - Web 层控制器

**Maven 依赖:**
```xml
<dependency>
    <groupId>io.github.starmoon1617</groupId>
    <artifactId>starmie-app-web</artifactId>
    <version>2.0.0</version>
</dependency>
```

#### AbstractBaseController - 通用 Controller
```java
import io.github.starmoon1617.starmie.core.app.web.AbstractBaseController;
import io.github.starmoon1617.starmie.core.base.BaseDto;
import io.github.starmoon1617.starmie.core.page.Pagination;

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
    
    // 已继承的端点:
    // GET  /toList    - 跳转列表页
    // GET  /toAdd     - 跳转新增页
    // GET  /toEdit    - 跳转编辑页
    // GET  /toDelete  - 跳转删除页
    // POST /list      - 分页列表数据
    // POST /save      - 保存
    // POST /update    - 更新
    // POST /delete    - 删除
}
```

#### 返回结果快捷方法
```java
// 在 Controller 中使用
return getSuccess(data);                    // 成功返回
return getFailure(-1, "Error message");     // 失败返回
```

---

### 5. starmie-utils-poi - Excel 工具

**Maven 依赖:**
```xml
<dependency>
    <groupId>io.github.starmoon1617</groupId>
    <artifactId>starmie-utils-poi</artifactId>
    <version>2.0.0</version>
</dependency>
```

#### Excel 导出
```java
import io.github.starmoon1617.starmie.utils.poi.write.ExcelWriter;
import io.github.starmoon1617.starmie.utils.doc.head.DocHead;
import io.github.starmoon1617.starmie.utils.doc.enums.DateMode;

// 定义表头
List<DocHead> heads = Arrays.asList(
    new DocHead("id", "ID"),
    new DocHead("username", "用户名"),
    new DocHead("email", "邮箱"),
    new DocHead("createTime", "创建时间")
);

// 导出到 OutputStream
List<User> users = userManager.find(criteria);
ExcelWriter.WriteToExcel(outputStream, "用户列表", heads, users, DateMode.DATE_TIME);
```

#### Excel 导入
```java
import io.github.starmoon1617.starmie.utils.poi.read.ExcelReader;
import io.github.starmoon1617.starmie.utils.poi.read.RowReadListener;

// 实现 RowReadListener 处理每行数据
```

---

### 6. starmie-utils-pdf - PDF 工具

**Maven 依赖:**
```xml
<dependency>
    <groupId>io.github.starmoon1617</groupId>
    <artifactId>starmie-utils-pdf</artifactId>
    <version>2.0.0</version>
</dependency>
```

```java
import io.github.starmoon1617.starmie.utils.pdf.write.PdfWriter;
import io.github.starmoon1617.starmie.utils.pdf.config.PageConf;
import io.github.starmoon1617.starmie.utils.pdf.enums.PageType;
import io.github.starmoon1617.starmie.utils.pdf.enums.PageOrientation;
```

---

### 7. starmie-boot-executor - 线程池自动配置

**Maven 依赖:**
```xml
<dependency>
    <groupId>io.github.starmoon1617</groupId>
    <artifactId>starmie-boot-executor</artifactId>
    <version>2.0.0</version>
</dependency>
```

```java
import io.github.starmoon1617.starmie.boot.executor.util.ExecutorUtils;
// 线程池自动配置,通过 Spring Boot auto-configuration 生效
```

#### 虚拟线程支持 (Java 21+)
```xml
<dependency>
    <groupId>io.github.starmoon1617</groupId>
    <artifactId>starmie-boot-executor-vt</artifactId>
    <version>2.0.0</version>
</dependency>
```

```java
import io.github.starmoon1617.starmie.boot.executor.vt.util.VirtualThreadExecutorUtils;
```

---

### 8. starmie-generator - 代码生成器

**Maven 依赖:**
```xml
<dependency>
    <groupId>io.github.starmoon1617</groupId>
    <artifactId>starmie-generator-freemarker</artifactId>
    <version>2.0.0</version>
</dependency>
<!-- 或 -->
<dependency>
    <groupId>io.github.starmoon1617</groupId>
    <artifactId>starmie-generator-thymeleaf</artifactId>
    <version>2.0.0</version>
</dependency>
```

基于 MyBatis Generator + 模板引擎,可生成:
- Model/Entity
- Mapper 接口 + XML
- Service 接口 + 实现
- Manager 接口 + 实现
- Controller
- 前端页面 (JS/HTML)

---

## 完整项目配置示例

### pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
    https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>io.github.starmoon1617</groupId>
        <artifactId>starmie-bom</artifactId>
        <version>2.0.0</version>
    </parent>
    
    <groupId>com.example</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0.0</version>
    
    <dependencies>
        <!-- Starmie Framework -->
        <dependency>
            <groupId>io.github.starmoon1617</groupId>
            <artifactId>starmie-core-base</artifactId>
            <version>2.0.0</version>
        </dependency>
        <dependency>
            <groupId>io.github.starmoon1617</groupId>
            <artifactId>starmie-core-common</artifactId>
            <version>2.0.0</version>
        </dependency>
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
        <dependency>
            <groupId>io.github.starmoon1617</groupId>
            <artifactId>starmie-utils-poi</artifactId>
            <version>2.0.0</version>
        </dependency>
        <dependency>
            <groupId>io.github.starmoon1617</groupId>
            <artifactId>starmie-boot-executor</artifactId>
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
        <dependency>
            <groupId>com.zaxxer</groupId>
            <artifactId>HikariCP</artifactId>
        </dependency>
    </dependencies>
</project>
```

---

## 代码生成模板

### 快速生成实体类
当需要创建新实体时,使用以下模板:
```java
package com.example.entity;

import io.github.starmoon1617.starmie.core.base.BaseEntity;
import java.util.Date;

public class ${EntityName} extends BaseEntity<Long, Long> {
    
    private static final long serialVersionUID = 1L;
    
    // 属性定义
    
    // getters/setters
}
```

### 快速生成 CRUD 层
```java
// Mapper
package com.example.mapper;
import io.github.starmoon1617.starmie.core.mapper.BaseMapper;
import com.example.entity.${EntityName};

public interface ${EntityName}Mapper extends BaseMapper<${EntityName}, Long, Long> {
}

// Service
package com.example.service;
import io.github.starmoon1617.starmie.core.service.BaseService;
import com.example.entity.${EntityName};

public interface ${EntityName}Service extends BaseService<${EntityName}, Long, Long> {
}

// ServiceImpl
package com.example.service.impl;
import io.github.starmoon1617.starmie.core.service.BaseServiceImpl;
import io.github.starmoon1617.starmie.core.mapper.BaseMapper;
import com.example.entity.${EntityName};
import com.example.mapper.${EntityName}Mapper;
import com.example.service.${EntityName}Service;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class ${EntityName}ServiceImpl extends BaseServiceImpl<${EntityName}, Long, Long> implements ${EntityName}Service {
    
    @Autowired
    private ${EntityName}Mapper mapper;
    
    @Override
    protected BaseMapper<${EntityName}, Long, Long> getMapper() {
        return mapper;
    }
}

// Manager
package com.example.manager;
import io.github.starmoon1617.starmie.core.manager.BaseManager;
import com.example.entity.${EntityName};

public interface ${EntityName}Manager extends BaseManager<${EntityName}, Long, Long> {
}

// ManagerImpl
package com.example.manager.impl;
import io.github.starmoon1617.starmie.core.manager.BaseManagerImpl;
import io.github.starmoon1617.starmie.core.service.BaseService;
import com.example.entity.${EntityName};
import com.example.manager.${EntityName}Manager;
import com.example.service.${EntityName}Service;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class ${EntityName}ManagerImpl extends BaseManagerImpl<${EntityName}, Long, Long> implements ${EntityName}Manager {
    
    @Autowired
    private ${EntityName}Service service;
    
    @Override
    protected BaseService<${EntityName}, Long, Long> getService() {
        return service;
    }
}

// Controller
package com.example.controller;
import io.github.starmoon1617.starmie.core.app.web.AbstractBaseController;
import io.github.starmoon1617.starmie.core.manager.BaseManager;
import com.example.entity.${EntityName};
import com.example.manager.${EntityName}Manager;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/${entityName}")
public class ${EntityName}Controller extends AbstractBaseController<${EntityName}, Long, Long> {
    
    @Autowired
    private ${EntityName}Manager manager;
    
    @Override
    protected BaseManager<${EntityName}, Long, Long> getManager() {
        return manager;
    }
    
    @Override
    protected String getPath() {
        return "${entityName}";
    }
}
```

---

## 注意事项

1. **包名**: 所有 starmie 类位于 `io.github.starmoon1617.starmie.*` 包下
2. **泛型**: 基类使用三个泛型参数 `<E, ID, U>` - 实体、主键类型、用户标识类型
3. **分页**: `Pagination.pageNo` 从 0 开始计数
4. **查询条件**: `BaseCriteria` 会自动将驼峰字段名转换为下划线格式
5. **JSON**: 使用 Jackson 进行 JSON 序列化,日期默认格式为 `yyyy-MM-dd HH:mm:ss`

---

## 相关链接

- GitHub: https://github.com/starmoon1617/starmie-framework
- BOM: https://github.com/starmoon1617/starmie-bom
