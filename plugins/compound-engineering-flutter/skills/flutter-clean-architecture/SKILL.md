---
name: flutter-clean-architecture
description: Flutter Clean Architecture 实践指南。在设计项目架构、创建新功能模块、讨论分层策略或重构代码结构时使用。涵盖分层架构、依赖反转、数据流设计和模块化。
---

# Flutter Clean Architecture

> 清晰的分层，明确的职责，可测试的代码。

## 架构概览

```
┌─────────────────────────────────────────┐
│             Presentation 层              │
│  Pages / Widgets / Bloc / Controller    │
├─────────────────────────────────────────┤
│              Domain 层                   │
│  Entities / Use Cases / Repositories    │
├─────────────────────────────────────────┤
│               Data 层                    │
│  Models / Repositories / DataSources    │
└─────────────────────────────────────────┘
```

**依赖方向**：Presentation → Domain ← Data

Domain 层不依赖任何外部层，是整个架构的核心。

## 项目结构

```
lib/
├── core/                      # 全局共享
│   ├── error/                 # 错误和异常定义
│   │   ├── exceptions.dart
│   │   └── failures.dart
│   ├── network/               # 网络客户端
│   │   ├── api_client.dart
│   │   └── interceptors.dart
│   ├── theme/                 # 主题和样式
│   │   ├── app_theme.dart
│   │   ├── app_colors.dart
│   │   └── app_text_styles.dart
│   ├── utils/                 # 工具函数
│   ├── widgets/               # 全局共享 Widget
│   └── constants/             # 常量定义
├── features/                  # 按功能模块组织
│   ├── auth/
│   │   ├── data/
│   │   │   ├── datasources/
│   │   │   │   ├── auth_remote_datasource.dart
│   │   │   │   └── auth_local_datasource.dart
│   │   │   ├── models/
│   │   │   │   └── user_model.dart
│   │   │   └── repositories/
│   │   │       └── auth_repository_impl.dart
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   │   └── user.dart
│   │   │   ├── repositories/
│   │   │   │   └── auth_repository.dart   (抽象类)
│   │   │   └── usecases/
│   │   │       ├── login.dart
│   │   │       └── register.dart
│   │   └── presentation/
│   │       ├── bloc/
│   │       │   ├── auth_bloc.dart
│   │       │   ├── auth_event.dart
│   │       │   └── auth_state.dart
│   │       ├── pages/
│   │       │   ├── login_page.dart
│   │       │   └── register_page.dart
│   │       └── widgets/
│   │           └── auth_form.dart
│   ├── product/
│   │   └── ...                # 同样的三层结构
│   └── cart/
│       └── ...
├── di/                        # 依赖注入
│   └── injection_container.dart
├── router/                    # 路由配置
│   └── app_router.dart
└── main.dart
```

## 各层详解

### Domain 层 - 业务核心

#### Entity（实体）

```dart
/// 领域实体，不依赖任何框架
class User {
  final String id;
  final String name;
  final String email;
  final UserRole role;

  const User({
    required this.id,
    required this.name,
    required this.email,
    required this.role,
  });
}

enum UserRole { admin, member, guest }
```

#### Repository 接口

```dart
/// 仓库抽象接口，定义数据操作契约
abstract class AuthRepository {
  /// 使用邮箱和密码登录
  Future<Either<Failure, User>> login(String email, String password);

  /// 注册新用户
  Future<Either<Failure, User>> register(RegisterParams params);

  /// 获取当前登录用户
  Future<Either<Failure, User>> getCurrentUser();

  /// 退出登录
  Future<Either<Failure, void>> logout();
}
```

#### Use Case（用例）

```dart
/// 用例基类
abstract class UseCase<Type, Params> {
  Future<Either<Failure, Type>> call(Params params);
}

/// 无参数用例
class NoParams {}

/// 登录用例
class LoginUseCase extends UseCase<User, LoginParams> {
  final AuthRepository repository;

  LoginUseCase(this.repository);

  @override
  Future<Either<Failure, User>> call(LoginParams params) {
    return repository.login(params.email, params.password);
  }
}

class LoginParams {
  final String email;
  final String password;

  const LoginParams({required this.email, required this.password});
}
```

### Data 层 - 数据实现

#### Model（数据模型）

```dart
/// 数据模型，负责序列化/反序列化
class UserModel extends User {
  const UserModel({
    required super.id,
    required super.name,
    required super.email,
    required super.role,
  });

  factory UserModel.fromJson(Map<String, dynamic> json) {
    return UserModel(
      id: json['id'] as String,
      name: json['name'] as String,
      email: json['email'] as String,
      role: UserRole.values.byName(json['role'] as String),
    );
  }

  Map<String, dynamic> toJson() => {
    'id': id,
    'name': name,
    'email': email,
    'role': role.name,
  };
}
```

#### DataSource（数据源）

```dart
/// 远程数据源
abstract class AuthRemoteDataSource {
  Future<UserModel> login(String email, String password);
  Future<UserModel> register(RegisterParams params);
}

class AuthRemoteDataSourceImpl implements AuthRemoteDataSource {
  final ApiClient client;

  AuthRemoteDataSourceImpl(this.client);

  @override
  Future<UserModel> login(String email, String password) async {
    final response = await client.post('/auth/login', data: {
      'email': email,
      'password': password,
    });
    return UserModel.fromJson(response.data);
  }
}
```

#### Repository 实现

```dart
class AuthRepositoryImpl implements AuthRepository {
  final AuthRemoteDataSource remoteDataSource;
  final AuthLocalDataSource localDataSource;

  AuthRepositoryImpl({
    required this.remoteDataSource,
    required this.localDataSource,
  });

  @override
  Future<Either<Failure, User>> login(String email, String password) async {
    try {
      final user = await remoteDataSource.login(email, password);
      await localDataSource.cacheUser(user);
      return Right(user);
    } on ServerException catch (e) {
      return Left(ServerFailure(e.message));
    } on NetworkException {
      return Left(const NetworkFailure('网络连接失败'));
    }
  }
}
```

### Presentation 层 - UI 和状态

```dart
/// Bloc 处理 UI 状态
class AuthBloc extends Bloc<AuthEvent, AuthState> {
  final LoginUseCase loginUseCase;

  AuthBloc({required this.loginUseCase}) : super(AuthInitial()) {
    on<LoginRequested>(_onLoginRequested);
  }

  Future<void> _onLoginRequested(
    LoginRequested event,
    Emitter<AuthState> emit,
  ) async {
    emit(AuthLoading());
    final result = await loginUseCase(
      LoginParams(email: event.email, password: event.password),
    );
    result.fold(
      (failure) => emit(AuthError(failure.message)),
      (user) => emit(AuthAuthenticated(user)),
    );
  }
}
```

## 依赖注入

```dart
// 使用 get_it 进行依赖注入
final sl = GetIt.instance;

void setupDependencies() {
  // Bloc
  sl.registerFactory(() => AuthBloc(loginUseCase: sl()));

  // Use Cases
  sl.registerLazySingleton(() => LoginUseCase(sl()));

  // Repositories
  sl.registerLazySingleton<AuthRepository>(
    () => AuthRepositoryImpl(
      remoteDataSource: sl(),
      localDataSource: sl(),
    ),
  );

  // Data Sources
  sl.registerLazySingleton<AuthRemoteDataSource>(
    () => AuthRemoteDataSourceImpl(sl()),
  );

  // External
  sl.registerLazySingleton(() => ApiClient());
}
```

## 关键原则

1. **依赖反转**：高层模块不依赖低层模块，两者都依赖抽象
2. **单一职责**：每个类只有一个变化的理由
3. **数据流单向**：UI → Event → Bloc → UseCase → Repository → DataSource
4. **错误处理用 Either**：避免 try-catch 穿透多层
5. **Entity 与 Model 分离**：领域实体 ≠ API 数据结构
