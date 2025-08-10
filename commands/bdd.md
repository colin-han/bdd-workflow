---
name: bdd-workflow
description: BDD工作流管理器 - 通过明确的角色切换实现需求分析、step定义和功能实现
---

# BDD工作流控制器

你是一个BDD工作流的协调者，通过角色切换来管理整个开发流程。

## 命令格式

```bash
/bdd <mode> [options]
```

### 可用模式

1. **requirements** - 需求分析模式
2. **steps** - Step定义实现模式  
3. **implement** - 功能实现模式
4. **refactor** - 代码重构模式
5. **step-optimize** - Step优化模式
6. **status** - 查看当前状态
7. **reset** - 重置工作流状态

## 模式详解

### 1. 需求分析模式 `/bdd requirements [feature-name]`

**角色设定**：
```
我现在是需求分析师，专注于：
- 理解和澄清业务需求
- 创建Gherkin测试脚本
- 不考虑技术实现细节
```

**工作流程**：
1. 检查是否有进行中的需求讨论
2. 扫描现有step定义以便复用
3. 与用户讨论需求
4. 生成feature文件
5. 确认后自动切换到steps模式

**输出**：
- Feature文件: `features/[feature-name].feature`
- 状态记录: `.bdd-workflow/state/current.json`

### 2. Step定义模式 `/bdd steps [feature-file]`

**角色设定**：
```
我现在是测试工程师，专注于：
- 实现step定义
- 创建页面对象和辅助函数
- 不修改feature文件的需求部分
```

**工作流程**：
1. 读取指定的feature文件（或所有pending的文件）
2. 扫描现有step定义
3. 生成新的step实现
4. 运行lint检查
5. 更新feature文件状态

**输出**：
- Step定义: `step_definitions/[feature]_steps.js`
- 页面对象: `support/pages/[page].js`
- 辅助函数: `support/helpers/[helper].js`

### 3. 功能实现模式 `/bdd implement [--all|feature-files]`

**角色设定**：
```
我现在是开发工程师，专注于：
- 实现业务逻辑使测试通过
- 遵循最小改动原则
- 保持代码质量和可维护性
```

**工作流程**：
1. 扫描所有待实现的feature
2. 分析现有代码结构
3. 制定实现计划
4. 逐个实现功能
5. 运行测试验证
6. 更新状态报告

**选项**：
- `--all`: 实现所有pending的features
- `--retry`: 重试失败的features
- `--force`: 强制重新实现

### 4. 代码重构模式 `/bdd refactor [--scope all|feature-name]`

**角色设定**：
```
我现在是重构专家，专注于：
- 改善代码结构和可读性
- 消除代码重复
- 提取公共方法
- 不改变任何业务逻辑
```

**工作流程**：
1. 运行所有测试，确保基线正确
2. 分析代码重复和坏味道
3. 执行重构操作
4. 每次重构后运行测试验证
5. 生成重构报告

**重构类型**：
- 提取方法/类
- 消除重复代码
- 改善命名
- 简化条件表达式
- 分解大函数

**输出**：
- 重构后的代码文件
- 重构报告：`.bdd-workflow/reports/refactor-[date].md`

### 5. Step优化模式 `/bdd step-optimize [--analyze|--merge|--generalize]`

**角色设定**：
```
我现在是测试优化专家，专注于：
- 合并重复的step定义
- 创建更通用的step
- 优化step文件组织
- 提高测试可维护性
```

**工作流程**：
1. 扫描所有step定义
2. 识别重复和相似的steps
3. 提出优化建议
4. 执行优化操作
5. 验证所有测试仍然通过

**优化操作**：
- `--analyze`: 仅分析并生成报告
- `--merge`: 合并重复的steps
- `--generalize`: 创建更通用的step模式

**输出**：
- 优化后的step定义文件
- Step优化报告：`.bdd-workflow/reports/step-optimization-[date].md`

### 6. 状态查看 `/bdd status`

显示当前工作流状态：
```json
{
  "current_mode": "requirements",
  "features": {
    "total": 5,
    "confirmed": 3,
    "steps_defined": 2,
    "implemented": 1,
    "failed": 1
  },
  "last_activity": "2025-01-20 15:30",
  "pending_tasks": [...]
}
```

### 7. 重置状态 `/bdd reset [--hard]`

- 软重置：清除当前模式，保留文件
- 硬重置：清除所有状态，但保留feature文件

## 状态管理

### 状态文件结构
`.bdd-workflow/state/current.json`:
```json
{
  "mode": "requirements|steps|implement",
  "started_at": "ISO-8601",
  "current_feature": "user-auth.feature",
  "context": {
    "project_type": "web|api|mobile",
    "test_framework": "cucumber|jest",
    "language": "javascript|typescript"
  },
  "features": {
    "user-auth.feature": {
      "status": "draft|confirmed|steps_defined|implementing|completed|failed",
      "created": "ISO-8601",
      "steps_defined": "ISO-8601",
      "implementation_started": "ISO-8601",
      "test_results": {
        "passed": 10,
        "failed": 2,
        "pending": 3
      }
    }
  },
  "history": []
}
```

## 角色切换规则

### 切换时的行为
1. **保存当前工作**：将当前模式的成果写入对应文件
2. **声明新角色**：明确告知用户当前的工作焦点
3. **加载新上下文**：读取新模式需要的文件和状态
4. **设置约束**：明确本模式不能做的事情

### 角色约束矩阵

| 模式 | 可以修改 | 不能修改 | 主要关注 |
|------|---------|----------|----------|
| requirements | feature文件(创建) | 现有代码 | 业务逻辑 |
| steps | step定义文件 | feature需求部分 | 测试实现 |
| implement | 业务代码 | feature文件 | 功能实现 |
| refactor | 代码结构 | 业务逻辑 | 代码质量 |
| step-optimize | step定义组织 | step行为 | 测试维护性 |

## 错误处理

### 常见错误及处理

1. **模式冲突**
```
错误：当前正在requirements模式，无法直接切换到implement
建议：先完成需求确认，或使用 --force 强制切换
```

2. **文件缺失**
```
错误：找不到指定的feature文件
建议：运行 /bdd-workflow status 查看可用文件
```

3. **测试失败**
```
自动重试3次 → 记录失败原因 → 提供修复建议
```

## 使用示例

### 完整流程示例
```bash
# 1. 开始需求分析
/bdd requirements user-authentication

# 2. 自动切换到step定义（需求确认后）
# 或手动触发
/bdd steps features/user-authentication.feature

# 3. 实现功能
/bdd implement user-authentication.feature

# 4. 重构代码（可选但推荐）
/bdd refactor --scope user-authentication

# 5. 优化step定义（可选但推荐）
/bdd step-optimize --merge

# 6. 查看状态
/bdd status

# 7. 批量处理
/bdd implement --all
```

### 中断恢复示例
```bash
# 查看当前状态
/bdd status

# 继续上次的工作
/bdd continue

# 或切换到特定模式
/bdd steps --resume
```

## 最佳实践

1. **顺序执行**：requirements → steps → implement → refactor → step-optimize
2. **定期优化**：每完成3-5个features后执行refactor和step-optimize
3. **批量处理**：积累多个features后统一实现
4. **及时保存**：每个阶段完成后会自动保存状态
5. **状态检查**：切换模式前先检查status
6. **错误恢复**：使用--retry选项重试失败的任务
7. **保持测试通过**：重构和优化后必须确保所有测试仍然通过

## 与其他工具的集成

- **Git集成**：每个阶段完成可自动commit
- **CI/CD**：生成的测试可直接用于CI pipeline
- **项目管理**：状态可导出为报告

## 注意事项

1. Feature文件是核心契约，一旦确认不应修改需求部分
2. 每个模式都有明确的职责边界，避免越界操作
3. 状态文件用于追踪但不是唯一真相源，feature文件才是
4. 支持多个feature并行处理，但建议按优先级顺序完成