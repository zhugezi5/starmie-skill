# Starmie Skill

AI 编码助手的 Skill 文件集合，用于在使用 GitHub Copilot 和 Claude Code 时优先使用 [starmie-framework](https://github.com/starmoon1617/starmie-framework) 的 API 进行开发。

## 📁 项目结构

```
starmie-skill/
├── README.md                    # 本文件 - 项目说明和使用指南
├── .copilot/                     # GitHub Copilot 配置文件
│   └── copilot-instructions.md  # Copilot 指令文件
├── .claude/                      # Claude Code 配置文件
│   └── starmie.md              # Claude 规则文件
└── docs/                        # 详细文档
    ├── starmie-framework-skill.md   # 完整 API 参考文档
    └── quick-start.md               # 快速入门指南
```

## 🚀 快速开始

### GitHub Copilot 配置

1. 将 `.copilot/copilot-instructions.md` 复制到你的项目根目录的 `.github/` 文件夹下：

```bash
# 在你的项目根目录执行
mkdir -p .github
cp path/to/starmie-skill/copilot/copilot-instructions.md .github/
```

2. 重启 VS Code 或 IDE，Copilot 将自动加载这些指令。

### Claude Code 配置

1. 将 `.claude/starmie.md` 复制到你的项目根目录的 `.claude/` 文件夹下：

```bash
# 在你的项目根目录执行
mkdir -p .claude
cp path/to/starmie-skill/claude/starmie.md .claude/
```

2. Claude Code 将在会话中自动应用这些规则。

### 全局配置 (可选)

如果希望在所有项目中使用 starmie 规则：

**GitHub Copilot:**
```bash
# Windows
copy .copilot\copilot-instructions.md %USERPROFILE%\.github\copilot-instructions.md

# macOS/Linux
cp .copilot/copilot-instructions.md ~/.github/copilot-instructions.md
```

**Claude Code:**
```bash
# Windows
copy .claude\starmie.md %USERPROFILE%\.claude\starmie.md

# macOS/Linux
cp .claude/starmie.md ~/.claude/starmie.md
```

## ⚡ 触发条件

当你的输入包含以下关键字或场景时，AI 会优先使用 starmie-framework 的 API：

| 关键字/场景 | 说明 |
|------------|------|
| `starmie` | 直接触发 |
| 创建实体类 | 使用 `BaseEntity` |
| 查询条件 | 使用 `BaseCriteria` |
| 分页查询 | 使用 `Pagination` |
| CRUD 层 | 使用 `BaseMapper/Service/Manager/Controller` |
| Excel 导出 | 使用 `ExcelWriter` |
| PDF 导出 | 使用 `PdfWriter` |

## 📖 示例

### 示例 1: 创建实体
```
输入: "用 starmie 创建一个 User 实体，包含 username 和 email 字段"

输出:
import io.github.starmoon1617.starmie.core.base.BaseEntity;

public class User extends BaseEntity<Long, Long> {
    private String username;
    private String email;
    // getters/setters
}
```

### 示例 2: 查询条件
```
输入: "starmie 查询 status=1 且 age>18 的用户，按创建时间倒序"

输出:
BaseCriteria criteria = BaseCriteria.getInstance();
criteria.addEqual("status", 1);
criteria.addCriterion(OperatorType.GT, "age", 18);
criteria.addSortCriterion("createTime", SortType.DESC);
```

### 示例 3: 完整 CRUD
```
输入: "用 starmie 生成 Product 的完整 CRUD 层代码"

输出: 生成 ProductMapper, ProductService, ProductServiceImpl, 
      ProductManager, ProductManagerImpl, ProductController
```

## 📚 文档

- [完整 API 参考](docs/starmie-framework-skill.md) - 详细的 API 文档和代码示例
- [快速入门指南](docs/quick-start.md) - 5 分钟上手 starmie-framework

## 🔗 相关链接

- [starmie-framework](https://github.com/starmoon1617/starmie-framework) - 框架源码
- [starmie-bom](https://github.com/starmoon1617/starmie-bom) - 依赖管理 BOM

## 📋 版本兼容

| Starmie Skill | Starmie Framework | Java | Spring Boot |
|---------------|-------------------|------|-------------|
| 1.0.0 | 2.0.0 | 25 | 4.0.x |

## 📄 License

MIT License
