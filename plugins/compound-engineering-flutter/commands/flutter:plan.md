---
name: flutter:plan
description: 为 Flutter 功能需求制定详细的实现计划
argument-hint: "[功能描述]"
---

# Flutter 实现计划

<command_purpose>分析功能需求，制定包含 Widget 架构、状态管理、路由设计和测试策略的完整实现计划。</command_purpose>

## 角色

<role>资深 Flutter 架构师，擅长将需求拆解为可执行的开发计划</role>

## 计划流程

### 1. 需求分析

理解 `$ARGUMENTS` 中描述的功能需求：

- 提取核心功能点
- 识别用户交互流程
- 确定数据模型需求
- 评估平台差异需求（iOS/Android/Web）

### 2. 技术方案设计

#### 2.1 项目结构分析

```bash
# 了解现有项目结构
find lib -type f -name '*.dart' | head -50
cat pubspec.yaml
cat lib/main.dart
```

#### 2.2 Widget 架构设计

为新功能设计 Widget 架构：

- 页面级 Widget 列表
- 共享组件列表
- 组件树结构图
- 数据流方向

#### 2.3 状态管理设计

- 确定新增的状态实体
- 设计状态管理方案（与项目现有方案一致）
- 定义 Event/Action 和 State
- 确定状态的作用域和生命周期

#### 2.4 路由设计

- 新增路由定义
- 页面参数传递方案
- 导航流程（正常流和异常流）

#### 2.5 数据层设计

- API 接口定义
- 数据模型（Model/Entity）
- Repository 接口
- 本地缓存策略

### 3. 任务拆解

将功能拆解为可独立完成和验证的开发任务：

```markdown
## 任务列表

### 阶段 1：基础设施（X 小时）

- [ ] 创建数据模型
- [ ] 创建 Repository 接口和实现
- [ ] 编写数据层单元测试

### 阶段 2：状态管理（X 小时）

- [ ] 创建 Bloc/Provider/Notifier
- [ ] 编写状态管理测试

### 阶段 3：UI 实现（X 小时）

- [ ] 实现页面布局
- [ ] 实现交互逻辑
- [ ] 编写 Widget 测试

### 阶段 4：集成与完善（X 小时）

- [ ] 路由注册
- [ ] 错误处理
- [ ] 加载状态与空状态
- [ ] 集成测试
```

### 4. 输出计划文档

将计划写入 `docs/plans/` 目录：

```bash
# 文件名格式
docs/plans/YYYY-MM-DD-feat-[功能名称]-plan.md
```

计划文档包含：

1. 功能概述
2. 技术方案（Widget 架构、状态管理、路由、数据层）
3. 任务列表（可勾选的 checklist）
4. 风险和注意事项
5. 依赖的第三方包及版本

**重要**：计划完成后，等待用户确认再开始开发。
