---
name: flutter-analyze
description: 运行全面的 Flutter 项目健康检查
argument-hint: "[可选：--fix 自动修复]"
disable-model-invocation: true
---

对 Flutter 项目进行全面的健康检查，涵盖静态分析、依赖检查、代码格式和项目配置。

## 检查流程

### 1. 静态分析

```bash
flutter analyze
```

如果有问题并且指定了 `--fix`：

```bash
dart fix --apply
flutter analyze
```

### 2. 代码格式检查

```bash
dart format --set-exit-if-changed lib/ test/
```

如果有格式问题并且指定了 `--fix`：

```bash
dart format lib/ test/
```

### 3. 依赖健康检查

```bash
# 检查过期依赖
flutter pub outdated

# 检查未使用的依赖
flutter pub deps
```

分析结果，标记：

- 🔴 有重大安全更新的依赖
- 🟡 有新 major 版本的依赖
- 🟢 有 minor/patch 更新的依赖

### 4. 项目配置检查

检查以下文件的配置是否合理：

- `pubspec.yaml`：SDK 约束、依赖版本范围
- `analysis_options.yaml`：是否启用推荐的 lint 规则
- `android/app/build.gradle`：minSdkVersion、targetSdkVersion
- `ios/Podfile`：iOS 最低版本

### 5. 测试覆盖率

```bash
flutter test --coverage
```

分析覆盖率报告，找出覆盖率不足的模块。

### 6. 输出健康报告

```markdown
# Flutter 项目健康报告

## 总体评分：A/B/C/D

### 静态分析

- 错误：X 个
- 警告：X 个
- 信息：X 个

### 代码格式

- [通过/未通过]，X 个文件需要格式化

### 依赖状态

- 总依赖数：X
- 需要更新：X
- 安全更新：X

### 测试覆盖率

- 总覆盖率：XX%
- 覆盖率不足的模块：...

### 建议操作

1. [按优先级排列的改进建议]
```
