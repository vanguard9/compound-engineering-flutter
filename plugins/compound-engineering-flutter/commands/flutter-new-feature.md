---
name: flutter-new-feature
description: 快速搭建 Flutter 新功能的文件脚手架
argument-hint: "[功能名称] [--bloc|--riverpod|--provider]"
disable-model-invocation: true
---

根据指定的功能名称和状态管理方案，生成完整的功能模块脚手架。

## 参数解析

从 `$ARGUMENTS` 中提取：

- **功能名称**：第一个参数，将用于生成目录和文件名
- **状态管理方案**：`--bloc`（默认）、`--riverpod` 或 `--provider`

## 脚手架结构

假设功能名为 `product`：

### Bloc 方案

```
lib/features/product/
├── data/
│   ├── models/
│   │   └── product_model.dart
│   ├── repositories/
│   │   └── product_repository.dart
│   └── datasources/
│       └── product_remote_datasource.dart
├── domain/
│   ├── entities/
│   │   └── product.dart
│   └── repositories/
│       └── product_repository.dart    (抽象接口)
├── presentation/
│   ├── bloc/
│   │   ├── product_bloc.dart
│   │   ├── product_event.dart
│   │   └── product_state.dart
│   ├── pages/
│   │   └── product_page.dart
│   └── widgets/
│       └── product_card.dart
test/features/product/
├── data/
│   └── repositories/
│       └── product_repository_test.dart
├── presentation/
│   ├── bloc/
│   │   └── product_bloc_test.dart
│   └── pages/
│       └── product_page_test.dart
```

### Riverpod 方案

```
lib/features/product/
├── data/
│   ├── models/
│   │   └── product_model.dart
│   └── repositories/
│       └── product_repository.dart
├── domain/
│   └── entities/
│       └── product.dart
├── presentation/
│   ├── providers/
│   │   └── product_provider.dart
│   ├── pages/
│   │   └── product_page.dart
│   └── widgets/
│       └── product_card.dart
```

## 执行步骤

1. 解析参数，确定功能名和状态管理方案
2. 检查 `lib/features/` 目录是否存在，不存在则创建
3. 按照对应方案生成所有文件，包含基础代码模板
4. 生成对应的测试文件骨架
5. 输出生成的文件列表和后续步骤提示

## 生成的文件模板要求

- 所有文件包含正确的 import 和基本类定义
- Entity 类使用 `Equatable` 或 `freezed`
- State 类包含 loading/loaded/error 三态
- Page 和 Widget 使用 const 构造函数
- 测试文件包含基本的 group 和 test 骨架
- 所有公开 API 包含中文文档注释
