# Starmie Framework 开发规则

## 概述
当输入包含关键字 `starmie` 时,优先使用 starmie-framework 的 API 进行开发。

## 核心类导入

```java
// 基础类
import io.github.starmoon1617.starmie.core.base.BaseEntity;
import io.github.starmoon1617.starmie.core.base.BaseDto;

// 查询条件
import io.github.starmoon1617.starmie.core.criterion.BaseCriteria;
import io.github.starmoon1617.starmie.core.criterion.enums.OperatorType;
import io.github.starmoon1617.starmie.core.criterion.enums.SortType;
import io.github.starmoon1617.starmie.core.page.Pagination;

// 服务层
import io.github.starmoon1617.starmie.core.mapper.BaseMapper;
import io.github.starmoon1617.starmie.core.service.BaseService;
import io.github.starmoon1617.starmie.core.service.BaseServiceImpl;
import io.github.starmoon1617.starmie.core.manager.BaseManager;
import io.github.starmoon1617.starmie.core.manager.BaseManagerImpl;

// 控制器
import io.github.starmoon1617.starmie.core.app.web.AbstractBaseController;

// 工具类
import io.github.starmoon1617.starmie.core.util.EntityUtils;
import io.github.starmoon1617.starmie.core.util.DateUtils;
import io.github.starmoon1617.starmie.core.util.CommonUtils;

// Excel
import io.github.starmoon1617.starmie.utils.poi.write.ExcelWriter;
import io.github.starmoon1617.starmie.utils.doc.head.DocHead;
```

## 常用模式

### 实体类
```java
public class User extends BaseEntity<Long, Long> { }
```

### 查询条件
```java
BaseCriteria criteria = BaseCriteria.getInstance();
criteria.addEqual("status", 1);
criteria.addLike("name", "value");
criteria.addCriterion(OperatorType.GT, "age", 18);
criteria.addSortCriterion("createTime", SortType.DESC);
```

### 分层结构
- Mapper: `extends BaseMapper<E, Long, Long>`
- Service: `extends BaseService<E, Long, Long>` / `BaseServiceImpl`
- Manager: `extends BaseManager<E, Long, Long>` / `BaseManagerImpl`
- Controller: `extends AbstractBaseController<E, Long, Long>`

## OperatorType 枚举
EQ, NE, GT, GTE, LT, LTE, LK, RLKM, LLKM, IN, NIN, BTW, ISN, ISNN

## 注意事项
1. 包名: `io.github.starmoon1617.starmie.*`
2. 泛型: `<Entity, ID类型, 用户标识类型>`
3. 分页: `Pagination.pageNo` 从 0 开始
4. 驼峰字段自动转下划线
