# BDD Workflow 工作流管理系统

一个强大的BDD（行为驱动开发）工作流管理系统，通过明确的角色切换实现需求分析、step定义和功能实现的全流程管理。

## 🎯 核心特性

- **角色驱动**：通过清晰的角色切换（需求分析师、测试工程师、开发工程师）确保各阶段专注度
- **状态追踪**：自动管理工作流状态，支持断点续传和进度查看
- **模板化**：预置标准的Gherkin模板和工作流模板，提高效率
- **智能复用**：自动扫描和复用已有step定义，避免重复工作

## 📁 项目结构

```
BDD Workflow/
├── commands/
│   └── bdd.md                 # BDD工作流命令文档
├── workflows/
│   └── templates/             # 工作流模板目录
│       ├── requirements-template.md    # 需求分析师模板
│       ├── steps-template.md          # 测试工程师模板
│       ├── implement-template.md      # 开发工程师模板
│       ├── refactor-template.md       # 重构专家模板
│       └── step-optimize-template.md  # Step优化模板
├── install-claude-bdd         # Claude Code命令安装脚本
└── README.md                  # 项目说明文档
```

## 🚀 快速开始

### 1. 安装到Claude Code

```bash
# 克隆或下载本项目
git clone <repository-url>

# 安装到目标项目
./install-claude-bdd /path/to/your/project

# 或在目标项目目录中执行
./install-claude-bdd
```

### 2. 基本使用流程

```bash
# 1. 开始需求分析
/bdd requirements user-authentication

# 2. 定义测试步骤
/bdd steps features/user-authentication.feature

# 3. 实现功能
/bdd implement user-authentication.feature

# 4. 代码重构
/bdd refactor --scope user-authentication

# 5. 优化步骤定义
/bdd step-optimize --merge

# 6. 查看状态
/bdd status
```

## 📋 工作流模式

### 🔍 需求分析模式 (`/bdd requirements`)
- **角色**：需求分析师
- **专注**：业务需求理解、Gherkin脚本创建
- **输出**：Feature文件和需求文档

### 🧪 Step定义模式 (`/bdd steps`)
- **角色**：测试工程师
- **专注**：测试步骤实现、页面对象创建
- **输出**：Step定义文件和辅助函数

### ⚙️ 功能实现模式 (`/bdd implement`)
- **角色**：开发工程师
- **专注**：业务逻辑实现，测试驱动开发
- **输出**：功能代码和通过的测试

### 🔧 代码重构模式 (`/bdd refactor`)
- **角色**：重构专家
- **专注**：代码优化、结构改善
- **输出**：重构后的代码和报告

### 📊 Step优化模式 (`/bdd step-optimize`)
- **角色**：测试优化专家
- **专注**：Step定义合并、通用化
- **输出**：优化的Step定义和报告

## 🎭 角色切换机制

每个模式都有明确的职责边界：

| 模式 | 可修改 | 不可修改 | 主要关注点 |
|------|--------|----------|-----------|
| requirements | Feature文件(创建) | 现有代码 | 业务逻辑 |
| steps | Step定义文件 | Feature需求部分 | 测试实现 |
| implement | 业务代码 | Feature文件 | 功能实现 |
| refactor | 代码结构 | 业务逻辑 | 代码质量 |
| step-optimize | Step定义组织 | Step行为 | 测试维护性 |

## 📁 生成的文件结构

系统运行后会在目标项目中生成以下结构：

```
your-project/
├── features/                  # Feature文件目录
│   └── *.feature
├── step_definitions/          # Step定义目录
│   └── *_steps.js
├── support/                   # 支持文件目录
│   ├── pages/                # 页面对象
│   └── helpers/              # 辅助函数
└── .bdd-workflow/            # 工作流状态目录
    ├── state/
    │   └── current.json      # 当前状态文件
    └── reports/              # 报告目录
        ├── refactor-*.md
        └── step-optimization-*.md
```

## 📊 状态管理

系统使用JSON文件追踪工作流状态：

```json
{
  "mode": "requirements|steps|implement",
  "started_at": "2025-01-20T15:30:00Z",
  "current_feature": "user-auth.feature",
  "features": {
    "user-auth.feature": {
      "status": "completed",
      "created": "2025-01-20T14:00:00Z",
      "test_results": {
        "passed": 10,
        "failed": 0,
        "pending": 0
      }
    }
  }
}
```

## 🛠️ 高级功能

### 批量处理
```bash
# 实现所有待处理的features
/bdd implement --all

# 重试失败的features
/bdd implement --retry
```

### 状态查看和管理
```bash
# 查看当前状态
/bdd status

# 重置工作流状态
/bdd reset

# 硬重置（清除所有状态）
/bdd reset --hard
```

### 中断恢复
```bash
# 继续上次的工作
/bdd continue

# 恢复到特定模式
/bdd steps --resume
```

## 🎯 最佳实践

1. **顺序执行**：按 requirements → steps → implement → refactor → step-optimize 顺序进行
2. **定期优化**：每完成3-5个features后执行重构和step优化
3. **批量处理**：积累多个features后统一实现，提高效率
4. **状态检查**：切换模式前先检查当前状态
5. **测试保证**：重构后确保所有测试仍然通过

## 📈 与其他工具集成

- **Git集成**：每个阶段完成可自动提交
- **CI/CD**：生成的测试直接用于持续集成
- **项目管理**：状态可导出为进度报告

## 🔧 技术要求

- Node.js (用于JavaScript/TypeScript项目)
- Cucumber.js 或其他BDD测试框架
- Git (用于版本控制)
- Claude Code (用于运行工作流命令)

## 📝 许可证

本项目采用MIT许可证。

## 🤝 贡献

欢迎提交Issue和Pull Request来改进这个工作流系统。

---

*通过BDD Workflow，让行为驱动开发变得更加系统化和高效！*