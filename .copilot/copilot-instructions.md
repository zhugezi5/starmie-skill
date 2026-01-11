# Starmie Framework Development Instructions

## 触发条件
当用户输入包含关键字 `starmie` 或涉及以下场景时,**优先使用 starmie-framework 提供的接口/工具/API 进行编码**:
- 创建实体类、DTO、Result类
- 实现 Mapper/Service/Manager/Controller 层代码
- 数据库查询条件构建
- 分页查询
- Excel/PDF 导入导出
- 线程池配置
- 代码生成

---

## 框架概述

**Starmie Framework** (`io.github.starmoon1617:starmie:2.0.0`) 是一个模块化的 Java 框架,基于 Spring Boot 4.x + MyBatis 3.5.x 构建。

### 核心依赖 BOM
```xml
<parent>
    <groupId>io.github.starmoon1617</groupId>
    <artifactId>starmie-bom</artifactId>
    <version>2.0.0</version>
</parent>
```

### 技术栈
- Java 25 | Spring Boot 4.0.1 | MyBatis 3.5.19 | POI 5.5.1 | PDFBox 3.0.6

---

## 核心模块 API

### 1. 实体类 - starmie-core-base

```java
import io.github.starmoon1617.starmie.core.base.BaseEntity;
import io.github.starmoon1617.starmie.core.base.BaseDto;
import io.github.starmoon1617.starmie.core.base.BaseResult;

// 实体继承 BaseEntity<ID类型, 用户标识类型>
public class User extends BaseEntity<Long, Long> {
    private String username;
    // ...
}

// 统一返回 BaseDto<数据类型>
public BaseDto<User> getUser() { ... }
```

### 2. 查询条件 - starmie-core-common

```java
import io.github.starmoon1617.starmie.core.criterion.BaseCriteria;
import io.github.starmoon1617.starmie.core.criterion.enums.OperatorType;
import io.github.starmoon1617.starmie.core.criterion.enums.SortType;
import io.github.starmoon1617.starmie.core.criterion.enums.LimitationType;
import io.github.starmoon1617.starmie.core.page.Pagination;

// 查询条件构建
BaseCriteria criteria = BaseCriteria.getInstance();
criteria.addEqual("status", 1);              // 等于
criteria.addEqual("type", 1, 2, 3);          // IN
criteria.addLike("name", "john");            // LIKE %value%
criteria.addCriterion(OperatorType.GT, "age", 18);    // 大于
criteria.addCriterion(OperatorType.BTW, "time", start, end);  // BETWEEN
criteria.addSortCriterion("createTime", SortType.DESC);
criteria.addLimitation(LimitationType.OFFSET, 0);
criteria.addLimitation(LimitationType.LIMIT, 10);

// 分页 (pageNo 从0开始)
Pagination<User> page = new Pagination<>();
```

**OperatorType**: `EQ, NE, GT, GTE, LT, LTE, LK, RLKM, LLKM, IN, NIN, BTW, ISN, ISNN`

### 3. 工具类 - starmie-core-common

```java
import io.github.starmoon1617.starmie.core.util.EntityUtils;
import io.github.starmoon1617.starmie.core.util.DateUtils;
import io.github.starmoon1617.starmie.core.util.CommonUtils;

// EntityUtils
EntityUtils.copyProperties(target, source);  // 复制属性(排除ID)
String json = EntityUtils.toJson(obj);
User user = EntityUtils.fromJson(json, User.class);
List<User> list = EntityUtils.fromJsonToList(json, User.class);

// DateUtils
Date now = DateUtils.getCurrentDate();
Date date = DateUtils.parse("2024-01-01 12:00:00");  // 自动识别格式
String str = DateUtils.formatCurrentDateTime();      // yyyy-MM-dd HH:mm:ss

// CommonUtils
boolean empty = CommonUtils.isEmpty(collection);
boolean notBlank = CommonUtils.isNotBlank(string);
```

### 4. 数据层 - starmie-core-service

```java
import io.github.starmoon1617.starmie.core.mapper.BaseMapper;
import io.github.starmoon1617.starmie.core.service.BaseService;
import io.github.starmoon1617.starmie.core.service.BaseServiceImpl;
import io.github.starmoon1617.starmie.core.manager.BaseManager;
import io.github.starmoon1617.starmie.core.manager.BaseManagerImpl;

// Mapper 接口
public interface UserMapper extends BaseMapper<User, Long, Long> { }

// Service 接口
public interface UserService extends BaseService<User, Long, Long> { }

// Service 实现
@Service
public class UserServiceImpl extends BaseServiceImpl<User, Long, Long> implements UserService {
    @Autowired private UserMapper mapper;
    @Override protected BaseMapper<User, Long, Long> getMapper() { return mapper; }
}

// Manager 接口 (支持批量和分页)
public interface UserManager extends BaseManager<User, Long, Long> { }

// Manager 实现
@Component
public class UserManagerImpl extends BaseManagerImpl<User, Long, Long> implements UserManager {
    @Autowired private UserService service;
    @Override protected BaseService<User, Long, Long> getService() { return service; }
}
```

**BaseMapper 方法**: `select, insert, update, delete, selectList, count`
**BaseService 方法**: `find, save, delete, update, find(criteria), count`
**BaseManager 方法**: 以上 + `save(Collection), find(Pagination, criteria)`

### 5. 控制器 - starmie-app-web

```java
import io.github.starmoon1617.starmie.core.app.web.AbstractBaseController;

@RestController
@RequestMapping("/user")
public class UserController extends AbstractBaseController<User, Long, Long> {
    @Autowired private UserManager manager;
    @Override protected BaseManager<User, Long, Long> getManager() { return manager; }
    @Override protected String getPath() { return "user"; }
    
    // 已继承端点: POST /list, /save, /update, /delete
    // GET /toList, /toAdd, /toEdit, /toDelete
    
    // 返回方法
    // return getSuccess(data);
    // return getFailure(-1, "message");
}
```

### 6. Excel 导出 - starmie-utils-poi

```java
import io.github.starmoon1617.starmie.utils.poi.write.ExcelWriter;
import io.github.starmoon1617.starmie.utils.doc.head.DocHead;
import io.github.starmoon1617.starmie.utils.doc.enums.DateMode;

List<DocHead> heads = Arrays.asList(
    new DocHead("id", "ID"),
    new DocHead("username", "用户名")
);
ExcelWriter.WriteToExcel(outputStream, "Sheet名", heads, dataList, DateMode.DATE_TIME);
```

### 7. 线程池 - starmie-boot-executor

```java
import io.github.starmoon1617.starmie.boot.executor.util.ExecutorUtils;
// 虚拟线程 (Java 21+)
import io.github.starmoon1617.starmie.boot.executor.vt.util.VirtualThreadExecutorUtils;
```

---

## 快速代码模板

### 完整 CRUD 层生成
对于实体 `${EntityName}`,生成以下文件:
1. `${EntityName}Mapper` extends `BaseMapper<${EntityName}, Long, Long>`
2. `${EntityName}Service` extends `BaseService<${EntityName}, Long, Long>`
3. `${EntityName}ServiceImpl` extends `BaseServiceImpl<...>` 实现 `${EntityName}Service`
4. `${EntityName}Manager` extends `BaseManager<${EntityName}, Long, Long>`
5. `${EntityName}ManagerImpl` extends `BaseManagerImpl<...>` 实现 `${EntityName}Manager`
6. `${EntityName}Controller` extends `AbstractBaseController<${EntityName}, Long, Long>`

---

## 注意事项

1. **包名前缀**: `io.github.starmoon1617.starmie.*`
2. **泛型顺序**: `<Entity, ID类型, 用户标识类型>`
3. **分页起始**: `Pagination.pageNo` 从 **0** 开始
4. **字段转换**: `BaseCriteria` 自动将驼峰转下划线
5. **日期格式**: Jackson 默认 `yyyy-MM-dd HH:mm:ss`

## 相关资源
- Framework: https://github.com/starmoon1617/starmie-framework
- BOM: https://github.com/starmoon1617/starmie-bom
