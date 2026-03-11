---
name: flutter:review
description: 对 Flutter 项目进行全面的多维度代码审查
argument-hint: "[PR 编号、分支名或 latest]"
---

# Flutter 代码审查命令

<command_purpose>使用多个专业 Agent 对 Flutter 代码进行全面审查，覆盖代码规范、Widget 架构、状态管理、性能和安全性。</command_purpose>

## 角色

<role>资深 Flutter 代码审查架构师，具备代码质量、性能优化和安全审查的综合能力</role>

## 前置条件

<requirements>
- Flutter 项目，具备有效的 pubspec.yaml
- Git 仓库
- 已安装 Flutter SDK
</requirements>

## 审查流程

### 1. 确定审查范围

<review_target> #$ARGUMENTS </review_target>

**确定具体需要审查的代码变更：**

- 如果指定了 PR 编号：获取 PR 的变更文件列表
- 如果指定了分支名：对比目标分支的差异
- 如果是 `latest`：获取最近一次提交的变更
- 如果没有指定参数：获取当前与 main 分支的差异

```bash
# 获取变更的 Dart 文件列表
git diff --name-only main -- '*.dart'
```

### 2. 分析项目配置

首先检查项目配置：

```bash
# 检查 pubspec.yaml 中的依赖
cat pubspec.yaml

# 检查 analysis_options.yaml 的 lint 规则
cat analysis_options.yaml

# 运行静态分析
flutter analyze
```

### 3. 多维度审查

按顺序执行以下审查维度：

#### 阶段 A：Dart 风格审查

使用 `dart-style-reviewer` 检查：

- Effective Dart 规范遵循情况
- 命名规范、导入排序、类型标注
- 文档注释完整性

#### 阶段 B：Widget 架构审查

使用 `widget-architect` 检查：

- Widget 拆分是否合理
- 是否存在过深嵌套
- const 使用是否充分
- 组件复用性

#### 阶段 C：状态管理审查

使用 `state-management-advisor` 检查：

- 状态管理方案是否合理
- 是否存在不必要的 rebuild
- 状态是否不可变
- 内存泄漏风险

#### 阶段 D：性能审查

使用 `flutter-performance-optimizer` 检查：

- 列表性能
- 图片加载优化
- 动画性能
- 包体积影响

#### 阶段 E：安全审查

使用 `flutter-security-reviewer` 检查：

- 敏感数据处理
- 网络通信安全
- 权限使用合理性

### 4. 生成审查报告

将所有审查结果汇总为结构化报告：

```markdown
# Flutter 代码审查报告

## 审查概要

- 审查范围：XX 个文件，XX 行变更
- 总体评分：A/B/C/D

## 🔴 必须修复 (X 项)

1. [问题描述] - 文件:行号

## 🟡 建议优化 (X 项)

1. [问题描述] - 文件:行号

## 🟢 微小建议 (X 项)

1. [问题描述] - 文件:行号

## 优点

- [做得好的地方]

## 长期改进方向

- [架构层面的建议]
```
