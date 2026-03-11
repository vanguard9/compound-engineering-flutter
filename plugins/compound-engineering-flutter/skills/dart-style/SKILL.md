---
name: dart-style
description: Dart 语言编码规范速查。在编写 Dart/Flutter 代码、进行代码审查或讨论编码标准时使用。基于 Effective Dart 官方指南，涵盖命名、类型、文档、用法和设计。
---

# Dart 编码规范

> 基于 Effective Dart 官方指南的中文实践手册

## 命名规范

### 标识符命名

| 类别              | 风格                       | 示例                                       |
| ----------------- | -------------------------- | ------------------------------------------ |
| 类、枚举、typedef | UpperCamelCase             | `class HttpClient`、`enum Color`           |
| 变量、参数、函数  | lowerCamelCase             | `var itemCount`、`void getData()`          |
| 常量              | lowerCamelCase             | `const defaultTimeout = 30`                |
| 文件名            | lowercase_with_underscores | `user_profile_page.dart`                   |
| 库前缀            | lowercase_with_underscores | `import 'package:foo/foo.dart' as foo_bar` |
| 私有成员          | 前缀 \_                    | `String _name`、`void _init()`             |

### 命名最佳实践

```dart
// 布尔值用肯定的描述
bool isEnabled;      // 好
bool isNotDisabled;  // 差

// 集合用复数名
List<User> users;     // 好
List<User> userList;  // 差

// 回调用 on + 动词
VoidCallback? onPressed;
ValueChanged<String>? onChanged;

// 异步方法不需要 async 后缀
Future<User> getUser();     // 好
Future<User> getUserAsync(); // 差
```

## 类型系统

### 类型标注原则

```dart
// 公开 API 必须标注类型
String getGreeting(String name) => '你好，$name';

// 私有局部变量可以省略
final users = <User>[];        // 类型明显，可省略
var count = 0;                  // 类型明显
final result = compute(input);  // 类型可推断

// 避免冗余类型标注
final List<String> items = <String>[];  // 差：冗余
final items = <String>[];               // 好：简洁
```

### Null Safety

```dart
// 用 ? 标注可空类型
String? nullableName;

// 用 ! 需要确保非空（谨慎使用）
final length = nullableName!.length;  // 确认非空才用

// 优先使用空值运算符
final name = user?.name ?? '匿名';
final items = list ?? [];

// 使用 late 延迟初始化（确保使用前初始化）
late final TextEditingController _controller;
```

## 导入排序

```dart
// 1. dart 标准库
import 'dart:async';
import 'dart:convert';

// 2. package 第三方库
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

// 3. 项目内导入（相对路径）
import '../models/user.dart';
import 'widgets/header.dart';
```

排序规则：

- 各组之间空一行
- 每组内按字母排序
- 优先使用相对路径导入项目内文件

## 类设计

### 构造函数

```dart
class UserProfile extends StatelessWidget {
  // 使用 super. 参数语法
  const UserProfile({
    super.key,
    required this.name,
    required this.avatarUrl,
    this.subtitle,
    this.onTap,
  });

  final String name;
  final String avatarUrl;
  final String? subtitle;
  final VoidCallback? onTap;

  @override
  Widget build(BuildContext context) { ... }
}
```

### 集合操作

```dart
// 使用字面量创建集合
final items = <String>[];           // 好
final items = List<String>.empty(); // 差

// 使用集合操作符
final names = users.map((u) => u.name).toList();
final adults = users.where((u) => u.age >= 18);
final total = prices.fold<double>(0, (sum, p) => sum + p);

// 使用 spread 和 collection if/for
final widgets = [
  const Header(),
  if (showBanner) const Banner(),
  for (final item in items) ItemCard(item: item),
  const Footer(),
];
```

## 异步编程

```dart
// 使用 async/await 而非 .then()
// 好
Future<User> getUser() async {
  final response = await api.get('/user');
  return User.fromJson(response.data);
}

// 差
Future<User> getUser() {
  return api.get('/user').then((response) {
    return User.fromJson(response.data);
  });
}

// 多个不相互依赖的异步操作并行执行
final results = await Future.wait([
  fetchUsers(),
  fetchProducts(),
  fetchSettings(),
]);
```

## 错误处理

```dart
// 定义具体的异常类型
class NetworkException implements Exception {
  final String message;
  final int? statusCode;

  const NetworkException(this.message, {this.statusCode});

  @override
  String toString() => 'NetworkException: $message (status: $statusCode)';
}

// 捕获特定异常
try {
  await api.fetchData();
} on NetworkException catch (e) {
  showError('网络错误：${e.message}');
} on FormatException catch (e) {
  showError('数据格式错误：${e.message}');
}
```

## 文档注释

```dart
/// 用户资料管理服务。
///
/// 提供用户信息的查询、更新和缓存功能。
/// 所有方法在网络异常时抛出 [NetworkException]。
class UserService {
  /// 根据 [userId] 获取用户资料。
  ///
  /// 优先返回缓存数据，缓存过期后从服务端获取。
  /// 如果用户不存在，返回 null。
  Future<User?> getUser(String userId) async { ... }
}
```

## 代码格式

- 行宽 80 字符
- 使用尾随逗号触发自动格式化
- 运行 `dart format` 自动格式化
- 在 CI 中使用 `dart format --set-exit-if-changed` 检查
