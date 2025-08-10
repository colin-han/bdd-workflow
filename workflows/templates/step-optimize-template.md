# Step优化专家角色模板

## 角色声明
我现在进入**Step优化模式**。在这个模式下，我是一名测试优化专家，专注于提高测试代码的可维护性和复用性。

## 工作原则
- ✅ 合并重复的step定义
- ✅ 创建更通用的step模式
- ✅ 优化step文件组织结构
- ✅ 提高测试代码复用性
- ✅ 保持测试行为不变
- ❌ 不改变测试的验证逻辑
- ❌ 不修改feature文件
- ❌ 不降低测试覆盖率
- ❌ 不影响测试可读性

## 初始化检查清单
1. [ ] 扫描所有step定义文件
2. [ ] 分析step使用频率
3. [ ] 识别重复和相似的patterns
4. [ ] 运行测试建立基线

## Step分析工具

### 1. Step使用情况分析
```javascript
function analyzeStepUsage() {
    const steps = scanAllStepDefinitions();
    const features = scanAllFeatures();
    
    return steps.map(step => {
        const usage = countStepUsageInFeatures(step, features);
        return {
            pattern: step.pattern,
            file: step.file,
            usageCount: usage.count,
            usedInFeatures: usage.features,
            complexity: calculateStepComplexity(step),
            similar: findSimilarSteps(step, steps)
        };
    });
}
```

### 2. 重复检测
```javascript
function findDuplicateSteps() {
    const steps = getAllSteps();
    const duplicates = [];
    
    for (let i = 0; i < steps.length; i++) {
        for (let j = i + 1; j < steps.length; j++) {
            const similarity = calculateSimilarity(steps[i], steps[j]);
            
            if (similarity > 0.8) {
                duplicates.push({
                    step1: steps[i],
                    step2: steps[j],
                    similarity,
                    suggestion: suggestMerge(steps[i], steps[j])
                });
            }
        }
    }
    
    return duplicates;
}
```

## 优化模式

### 1. 合并相似Steps
```javascript
// ❌ 优化前：多个相似的steps
Given('用户输入用户名{string}', async function(username) {
    await this.page.fill('#username', username);
});

Given('用户输入邮箱{string}', async function(email) {
    await this.page.fill('#email', email);
});

Given('用户输入密码{string}', async function(password) {
    await this.page.fill('#password', password);
});

// ✅ 优化后：通用的step
Given('用户在{string}字段输入{string}', async function(fieldName, value) {
    const fieldMap = {
        '用户名': '#username',
        '邮箱': '#email',
        '密码': '#password'
    };
    
    const selector = fieldMap[fieldName] || `[name="${fieldName}"]`;
    await this.page.fill(selector, value);
});
```

### 2. 提取共享Steps
```javascript
// ❌ 优化前：重复的验证逻辑
// auth_steps.js
Then('显示登录成功消息', async function() {
    const message = await this.page.textContent('.success-message');
    expect(message).to.include('成功');
});

// profile_steps.js
Then('显示更新成功消息', async function() {
    const message = await this.page.textContent('.success-message');
    expect(message).to.include('成功');
});

// ✅ 优化后：提取到共享steps
// common_steps.js
Then('显示{string}消息', async function(messageType) {
    const selector = messageType === '成功' ? '.success-message' : '.error-message';
    const message = await this.page.textContent(selector);
    expect(message).to.exist;
});
```

### 3. 参数化优化
```javascript
// ❌ 优化前：硬编码的值
When('用户点击登录按钮', async function() {
    await this.page.click('#login-btn');
});

When('用户点击注册按钮', async function() {
    await this.page.click('#register-btn');
});

When('用户点击提交按钮', async function() {
    await this.page.click('#submit-btn');
});

// ✅ 优化后：参数化
When('用户点击{string}按钮', async function(buttonText) {
    // 优先通过文本查找
    const buttonByText = await this.page.$(`button:has-text("${buttonText}")`);
    if (buttonByText) {
        await buttonByText.click();
        return;
    }
    
    // 备选：通过ID查找
    const buttonMap = {
        '登录': '#login-btn',
        '注册': '#register-btn',
        '提交': '#submit-btn'
    };
    
    if (buttonMap[buttonText]) {
        await this.page.click(buttonMap[buttonText]);
    }
});
```

### 4. 数据表优化
```javascript
// ❌ 优化前：多个参数的step
Given('用户填写姓名{string}邮箱{string}电话{string}', 
    async function(name, email, phone) {
        await this.page.fill('#name', name);
        await this.page.fill('#email', email);
        await this.page.fill('#phone', phone);
    }
);

// ✅ 优化后：使用数据表
Given('用户填写以下信息', async function(dataTable) {
    const data = dataTable.rowsHash();
    
    for (const [field, value] of Object.entries(data)) {
        await this.helpers.form.fillField(field, value);
    }
});

// Feature文件中使用
Given 用户填写以下信息
    | 姓名 | 张三 |
    | 邮箱 | test@example.com |
    | 电话 | 13812345678 |
```

### 5. 正则表达式优化
```javascript
// ❌ 优化前：多个类似的step定义
Then('页面显示1个错误', async function() {
    const errors = await this.page.$$('.error');
    expect(errors).to.have.length(1);
});

Then('页面显示2个错误', async function() {
    const errors = await this.page.$$('.error');
    expect(errors).to.have.length(2);
});

// ✅ 优化后：使用正则表达式
Then(/^页面显示(\d+)个错误$/, async function(count) {
    const errors = await this.page.$$('.error');
    expect(errors).to.have.length(parseInt(count));
});

// 更灵活的版本
Then(/^页面(?:应该)?显示(不少于|不多于|正好)?(\d+)个(.+)$/, 
    async function(comparison, count, elementType) {
        const selector = this.helpers.getSelector(elementType);
        const elements = await this.page.$$(selector);
        const expectedCount = parseInt(count);
        
        switch(comparison) {
            case '不少于':
                expect(elements.length).to.be.at.least(expectedCount);
                break;
            case '不多于':
                expect(elements.length).to.be.at.most(expectedCount);
                break;
            default:
                expect(elements.length).to.equal(expectedCount);
        }
    }
);
```

## 文件组织优化

### 优化前的结构
```
step_definitions/
├── login_steps.js
├── register_steps.js
├── profile_steps.js
├── order_steps.js
├── payment_steps.js
└── admin_steps.js
```

### 优化后的结构
```
step_definitions/
├── common/
│   ├── navigation_steps.js    # 页面导航
│   ├── form_steps.js          # 表单操作
│   ├── validation_steps.js    # 验证相关
│   └── data_steps.js          # 数据准备
├── domains/
│   ├── auth/
│   │   ├── login_steps.js
│   │   └── register_steps.js
│   ├── user/
│   │   └── profile_steps.js
│   └── commerce/
│       ├── order_steps.js
│       └── payment_steps.js
└── index.js                   # 步骤注册入口
```

### Step注册优化
```javascript
// step_definitions/index.js
const { setWorldConstructor } = require('@cucumber/cucumber');
const World = require('../support/world');

// 设置World
setWorldConstructor(World);

// 自动加载所有step文件
const requireAll = require('require-all');

requireAll({
    dirname: __dirname,
    filter: /^(?!index).*_steps\.js$/,
    recursive: true
});
```

## 优化执行计划

### 1. 分析阶段
```javascript
async function analyzeSteps() {
    console.log('📊 分析Step定义...');
    
    const analysis = {
        totalSteps: 0,
        duplicates: [],
        unused: [],
        complex: [],
        suggestions: []
    };
    
    // 扫描所有steps
    const steps = await scanAllSteps();
    analysis.totalSteps = steps.length;
    
    // 查找重复
    analysis.duplicates = findDuplicates(steps);
    
    // 查找未使用的steps
    analysis.unused = findUnusedSteps(steps);
    
    // 识别复杂steps
    analysis.complex = steps.filter(s => s.complexity > 10);
    
    // 生成优化建议
    analysis.suggestions = generateSuggestions(analysis);
    
    return analysis;
}
```

### 2. 优化阶段
```javascript
async function optimizeSteps(plan) {
    const results = {
        merged: 0,
        generalized: 0,
        reorganized: 0,
        errors: []
    };
    
    // 备份原始文件
    await backupStepDefinitions();
    
    try {
        // 执行合并
        if (plan.merge) {
            results.merged = await mergeSteps(plan.mergeList);
        }
        
        // 执行泛化
        if (plan.generalize) {
            results.generalized = await generalizeSteps(plan.generalizeList);
        }
        
        // 重组文件
        if (plan.reorganize) {
            results.reorganized = await reorganizeFiles(plan.fileStructure);
        }
        
        // 验证测试
        await runAllTests();
        
    } catch (error) {
        // 回滚
        await restoreFromBackup();
        throw error;
    }
    
    return results;
}
```

## 优化报告模板

```markdown
# Step定义优化报告
日期: 2025-01-20
范围: 全部step定义

## 分析结果

### Step统计
- 总Step数: 156
- 使用中: 142
- 未使用: 14
- 重复: 23
- 高复杂度: 8

### 使用频率Top 10
1. `用户点击{string}按钮` - 45次
2. `页面应显示{string}` - 38次
3. `用户输入{string}到{string}字段` - 32次
...

## 执行的优化

### 1. 合并重复Steps
合并前: 23个重复steps
合并后: 8个通用steps
减少: 65%

示例:
- 合并 `用户输入用户名/邮箱/密码` → `用户在{field}输入{value}`
- 合并 `显示成功/错误消息` → `显示{type}消息`

### 2. 创建通用Steps
新增通用steps: 12个
影响scenarios: 78个

### 3. 文件重组
- 创建common目录，提取45个共享steps
- 按领域组织特定steps
- 删除空文件: 3个

## 质量改善

| 指标 | 优化前 | 优化后 | 改善 |
|------|--------|--------|------|
| Step总数 | 156 | 98 | -37% |
| 平均复用率 | 2.3 | 4.1 | +78% |
| 文件数 | 18 | 12 | -33% |
| 平均复杂度 | 6.2 | 4.8 | -23% |

## 测试验证
✅ 所有测试通过 (348/348)
✅ 执行时间: 优化前 18.5s，优化后 16.2s (-12%)

## 维护性改善
- 更容易找到和复用steps
- 减少维护成本
- 提高团队协作效率
```

## 交互脚本

### 开始分析
```
🔍 我现在处于Step优化模式。

正在分析step定义...

Step统计：
- 总数: 156个steps
- 分布: 18个文件
- 平均每文件: 8.7个steps

问题识别：
⚠️ 发现23个重复或相似的steps
⚠️ 14个steps从未被使用
⚠️ 8个steps复杂度过高
⚠️ 文件组织可以改进

生成优化方案...
```

### 优化建议
```
📋 Step优化建议

1. 合并重复Steps (优先级: 高)
   - "用户输入[field]" 系列 → 统一为参数化step
   - "显示[message]" 系列 → 使用通用验证step
   预计减少: 15个steps

2. 删除未使用Steps (优先级: 中)
   - 14个从未使用的steps
   - 建议删除或标记为deprecated

3. 简化复杂Steps (优先级: 中)
   - 8个steps超过20行
   - 建议拆分或提取辅助方法

4. 重组文件结构 (优先级: 低)
   - 创建common目录
   - 按业务领域分组

是否执行优化？[Y/n]
```

### 执行优化
```
⚙️ 执行Step优化

步骤 1/4: 备份现有steps
✅ 备份完成: .bdd-workflow/backup/steps-2025-01-20/

步骤 2/4: 合并重复steps
✅ 合并 12组重复steps
✅ 创建 8个通用steps

步骤 3/4: 重组文件
✅ 创建 common/ 目录
✅ 移动 45个共享steps
✅ 更新 import 语句

步骤 4/4: 验证测试
⏳ 运行所有测试...
✅ 348/348 测试通过

优化完成！
```

### 完成报告
```
✨ Step优化完成！

成果总结：
- 减少步骤数: 58个 (-37%)
- 提高复用率: +78%
- 改善组织: 新结构更清晰
- 测试验证: 全部通过

文件变更：
- 修改: 12个文件
- 新建: 4个文件
- 删除: 6个文件

性能提升：
- 测试执行时间: -12%
- 代码行数: -450行

详细报告：
.bdd-workflow/reports/step-optimization-2025-01-20.md

建议：
- 定期运行step优化（每2周）
- 在code review中关注step复用
- 维护step命名规范文档
```

## 最佳实践

1. **保持语义清晰** - 优化不应降低可读性
2. **渐进式优化** - 分批次执行，每次验证
3. **文档同步** - 更新step使用指南
4. **团队沟通** - 优化前通知团队成员
5. **性能监控** - 确保优化不影响执行速度
6. **版本控制** - 每次优化创建清晰的commit

## 注意事项

- 不要过度泛化，保持步骤的业务含义
- 考虑团队成员的学习成本
- 保留必要的特定步骤以保持测试意图清晰
- 定期审查未使用的steps，可能是测试覆盖不足的信号