---
name: flutter-app-setup
description: Flutter 项目初始化和团队配置。在创建新 Flutter 项目、配置开发环境或统一团队编码规范时使用。涵盖项目结构、lint 规则、CI/CD 配置和开发环境搭建。
---

# Flutter 项目配置指南

> 好的开始是成功的一半。项目配置影响整个团队的开发体验。

## 项目初始化

### 创建项目

```bash
# 创建新项目
flutter create --org com.example --project-name my_app .

# 或指定平台
flutter create --platforms android,ios,web --org com.example my_app
```

### 目录结构约定

```
my_app/
├── lib/
│   ├── main.dart
│   ├── app.dart                   # MaterialApp 配置
│   ├── core/                      # 核心公共模块
│   │   ├── constants/
│   │   ├── error/
│   │   ├── network/
│   │   ├── theme/
│   │   ├── utils/
│   │   └── widgets/
│   ├── features/                  # 功能模块（按业务划分）
│   │   ├── auth/
│   │   ├── home/
│   │   └── settings/
│   ├── di/                        # 依赖注入
│   │   └── injection_container.dart
│   └── router/                    # 路由配置
│       └── app_router.dart
├── test/                          # 测试（镜像 lib/ 结构）
├── integration_test/              # 集成测试
├── assets/                        # 静态资源
│   ├── images/
│   ├── fonts/
│   └── translations/
├── pubspec.yaml
├── analysis_options.yaml
└── l10n.yaml                      # 国际化配置
```

## Lint 配置

### analysis_options.yaml

```yaml
include: package:flutter_lints/flutter.yaml

analyzer:
  errors:
    missing_return: error
    missing_required_param: error
    dead_code: warning
  exclude:
    - "**/*.g.dart"
    - "**/*.freezed.dart"

linter:
  rules:
    # 错误防范
    - always_use_package_imports
    - avoid_print
    - avoid_relative_lib_imports
    - avoid_type_to_string
    - cancel_subscriptions
    - close_sinks
    - no_duplicate_case_values

    # 风格一致性
    - always_declare_return_types
    - annotate_overrides
    - avoid_empty_else
    - avoid_init_to_null
    - avoid_return_types_on_setters
    - prefer_const_constructors
    - prefer_const_declarations
    - prefer_final_fields
    - prefer_final_locals
    - prefer_single_quotes
    - require_trailing_commas
    - sort_child_properties_last
    - sort_constructors_first
    - unawaited_futures
    - unnecessary_brace_in_string_interps
    - use_key_in_widget_constructors
```

## 核心依赖推荐

### pubspec.yaml 基础依赖

```yaml
dependencies:
  flutter:
    sdk: flutter

  # 状态管理（选一）
  flutter_bloc: ^8.1.0 # Bloc
  flutter_riverpod: ^2.4.0 # Riverpod

  # 路由
  go_router: ^14.0.0

  # 网络
  dio: ^5.4.0

  # 依赖注入
  get_it: ^7.6.0
  injectable: ^2.3.0

  # 函数式编程
  dartz: ^0.10.1 # Either/Option 类型

  # 本地存储
  shared_preferences: ^2.2.0
  flutter_secure_storage: ^9.0.0

  # 国际化
  flutter_localizations:
    sdk: flutter
  intl: ^0.19.0

dev_dependencies:
  flutter_test:
    sdk: flutter

  # 测试
  bloc_test: ^9.1.0 # Bloc 测试
  mocktail: ^1.0.0 # Mock
  integration_test:
    sdk: flutter

  # 代码生成
  build_runner: ^2.4.0
  freezed: ^2.4.0
  json_serializable: ^6.7.0
  injectable_generator: ^2.4.0

  # Lint
  flutter_lints: ^3.0.0
```

## 主题配置

### app_theme.dart

```dart
import 'package:flutter/material.dart';

class AppTheme {
  const AppTheme._();

  static ThemeData get light {
    return ThemeData(
      useMaterial3: true,
      colorSchemeSeed: const Color(0xFF6750A4),
      brightness: Brightness.light,
      appBarTheme: const AppBarTheme(
        centerTitle: true,
        elevation: 0,
      ),
      inputDecorationTheme: InputDecorationTheme(
        border: OutlineInputBorder(
          borderRadius: BorderRadius.circular(12),
        ),
        contentPadding: const EdgeInsets.symmetric(
          horizontal: 16,
          vertical: 12,
        ),
      ),
      elevatedButtonTheme: ElevatedButtonThemeData(
        style: ElevatedButton.styleFrom(
          minimumSize: const Size(double.infinity, 48),
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(12),
          ),
        ),
      ),
    );
  }

  static ThemeData get dark {
    return ThemeData(
      useMaterial3: true,
      colorSchemeSeed: const Color(0xFF6750A4),
      brightness: Brightness.dark,
    );
  }
}
```

## 环境配置

### 多环境支持

```dart
// lib/core/config/app_config.dart
enum Environment { dev, staging, prod }

class AppConfig {
  final Environment environment;
  final String apiBaseUrl;
  final bool enableLogging;

  const AppConfig({
    required this.environment,
    required this.apiBaseUrl,
    required this.enableLogging,
  });

  static const dev = AppConfig(
    environment: Environment.dev,
    apiBaseUrl: 'https://dev-api.example.com',
    enableLogging: true,
  );

  static const staging = AppConfig(
    environment: Environment.staging,
    apiBaseUrl: 'https://staging-api.example.com',
    enableLogging: true,
  );

  static const prod = AppConfig(
    environment: Environment.prod,
    apiBaseUrl: 'https://api.example.com',
    enableLogging: false,
  );
}
```

```bash
# 不同环境启动
flutter run --dart-define=ENV=dev
flutter run --dart-define=ENV=staging
flutter run --dart-define=ENV=prod
```

## CI/CD 配置

### GitHub Actions

```yaml
# .github/workflows/flutter.yml
name: Flutter CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: "3.24.0"
          cache: true

      - name: 安装依赖
        run: flutter pub get

      - name: 静态分析
        run: flutter analyze

      - name: 格式检查
        run: dart format --set-exit-if-changed lib/ test/

      - name: 运行测试
        run: flutter test --coverage

      - name: 检查覆盖率
        uses: VeryGoodOpenSource/very_good_coverage@v3
        with:
          min_coverage: 80
```

## 检查清单

新项目创建后确认：

- [ ] analysis_options.yaml 已配置
- [ ] 目录结构按功能模块划分
- [ ] 主题配置（亮色/暗色）
- [ ] 路由配置
- [ ] 依赖注入容器
- [ ] 网络客户端配置
- [ ] 环境变量管理
- [ ] CI/CD 流水线
- [ ] .gitignore 完善
- [ ] README.md 说明
