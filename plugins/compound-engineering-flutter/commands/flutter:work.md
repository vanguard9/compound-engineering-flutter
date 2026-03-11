---
name: flutter:work
description: 按照计划文件执行 Flutter 开发任务
argument-hint: "[计划文件路径或任务描述]"
---

# Flutter 开发执行

<command_purpose>按照计划文件逐步执行 Flutter 开发任务，每完成一个任务就运行测试并标记完成。</command_purpose>

## 角色

<role>高效的 Flutter 开发工程师，严格按照计划执行任务，保证代码质量</role>

## 执行流程

### 1. 加载计划

```bash
# 查找最新的计划文件
ls -la docs/plans/*flutter* docs/plans/*feat*
```

如果指定了 `$ARGUMENTS`，加载对应的计划文件。否则加载最新的计划文件。

### 2. 环境检查

```bash
# 确保项目可以正常编译
flutter pub get
flutter analyze
flutter test --no-pub
```

### 3. 逐步执行任务

对于计划中的每个未完成的任务：

#### 3.1 开始任务

- 读取任务描述和验收标准
- 理解任务与其他任务的依赖关系
- 确认前置任务已完成

#### 3.2 编写代码

- 按照项目现有风格编写代码
- 遵循 Effective Dart 规范
- 添加必要的类型标注和文档注释
- 使用 const 构造函数
- 保持导入排序

#### 3.3 编写测试

- 为新增的业务逻辑编写单元测试
- 为新增的 Widget 编写 Widget 测试
- 确保测试覆盖正常路径和异常路径

#### 3.4 验证任务

```bash
# 静态分析
flutter analyze

# 运行测试
flutter test

# 格式化代码
dart format lib/ test/
```

#### 3.5 标记完成

在计划文件中将已完成的任务标记为 `[x]`。

### 4. 完成报告

所有任务执行完成后，输出：

```markdown
## 开发完成报告

### 完成的任务

- [x] 任务 1
- [x] 任务 2
      ...

### 新增文件

- lib/features/xxx/...
- test/features/xxx/...

### 修改文件

- lib/router.dart
- pubspec.yaml

### 测试结果

- 单元测试：XX 通过
- Widget 测试：XX 通过
- 静态分析：无问题

### 后续建议

- [需要手动测试的场景]
- [需要用户确认的设计决策]
```
