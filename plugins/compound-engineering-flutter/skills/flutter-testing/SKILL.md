---
name: flutter-testing
description: Flutter 测试最佳实践。在编写测试、设计测试策略或进行测试覆盖率分析时使用。涵盖单元测试、Widget 测试、集成测试、Mock 策略和 Golden 测试。
---

# Flutter 测试指南

> 好的测试让重构无所畏惧，让 bug 无处藏身。

## 测试分层

### 测试金字塔

```
         △
        / \
       / 集 \         ← 少量端到端测试
      / 成测 \           验证关键用户路径
     /  试    \
    /----------\
   / Widget 测 \      ← 中等数量
  /   试        \        验证组件交互行为
 /--------------\
/ 单元测试       \    ← 大量快速测试
/________________\       验证业务逻辑
```

| 层级        | 速度   | 依赖            | 覆盖目标           | 比例 |
| ----------- | ------ | --------------- | ------------------ | ---- |
| 单元测试    | 毫秒级 | 无              | 业务逻辑、工具函数 | 70%  |
| Widget 测试 | 秒级   | Flutter 框架    | UI 组件行为        | 20%  |
| 集成测试    | 分钟级 | 真实设备/模拟器 | 端到端流程         | 10%  |

## 单元测试

### 基础结构

```dart
// test/services/cart_service_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';

// Mock 类定义
class MockProductRepository extends Mock implements ProductRepository {}

void main() {
  // 测试组
  group('CartService', () {
    // 共享变量
    late CartService sut;  // system under test
    late MockProductRepository mockRepo;

    // 每个测试前执行
    setUp(() {
      mockRepo = MockProductRepository();
      sut = CartService(repository: mockRepo);
    });

    // 清理
    tearDown(() {
      // 如果需要清理资源
    });

    group('addItem', () {
      test('首次添加商品，购物车数量为 1', () {
        // Arrange（准备）
        final product = Product(id: '1', name: '测试', price: 10);

        // Act（执行）
        sut.addItem(product);

        // Assert（断言）
        expect(sut.itemCount, equals(1));
        expect(sut.totalPrice, equals(10.0));
      });

      test('添加相同商品，数量累加', () {
        final product = Product(id: '1', name: '测试', price: 10);

        sut.addItem(product);
        sut.addItem(product);

        expect(sut.itemCount, equals(1));
        expect(sut.items.first.quantity, equals(2));
      });
    });
  });
}
```

### Bloc 测试

```dart
import 'package:bloc_test/bloc_test.dart';

void main() {
  group('ProductBloc', () {
    late MockProductRepository mockRepo;
    final testProducts = [
      Product(id: '1', name: '商品A', price: 100),
      Product(id: '2', name: '商品B', price: 200),
    ];

    setUp(() {
      mockRepo = MockProductRepository();
    });

    blocTest<ProductBloc, ProductState>(
      '发出 LoadProducts 后，依次 emit loading 和 loaded 状态',
      setUp: () {
        when(() => mockRepo.getProducts())
            .thenAnswer((_) async => testProducts);
      },
      build: () => ProductBloc(repository: mockRepo),
      act: (bloc) => bloc.add(const LoadProducts()),
      expect: () => [
        const ProductState(status: ProductStatus.loading),
        ProductState(status: ProductStatus.loaded, products: testProducts),
      ],
      verify: (_) {
        verify(() => mockRepo.getProducts()).called(1);
      },
    );

    blocTest<ProductBloc, ProductState>(
      '加载失败时 emit error 状态',
      setUp: () {
        when(() => mockRepo.getProducts()).thenThrow(Exception('网络错误'));
      },
      build: () => ProductBloc(repository: mockRepo),
      act: (bloc) => bloc.add(const LoadProducts()),
      expect: () => [
        const ProductState(status: ProductStatus.loading),
        isA<ProductState>()
            .having((s) => s.status, 'status', ProductStatus.error)
            .having((s) => s.errorMessage, 'error', contains('网络错误')),
      ],
    );
  });
}
```

## Widget 测试

### 基础 Widget 测试

```dart
void main() {
  group('ProductCard', () {
    final product = Product(id: '1', name: '测试商品', price: 99.9);

    testWidgets('显示商品名称和价格', (tester) async {
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: ProductCard(product: product),
          ),
        ),
      );

      expect(find.text('测试商品'), findsOneWidget);
      expect(find.text('¥99.90'), findsOneWidget);
    });

    testWidgets('点击触发 onTap 回调', (tester) async {
      var tapped = false;

      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: ProductCard(
              product: product,
              onTap: () => tapped = true,
            ),
          ),
        ),
      );

      await tester.tap(find.byType(ProductCard));
      expect(tapped, isTrue);
    });

    testWidgets('长按显示删除选项', (tester) async {
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: ProductCard(product: product, canDelete: true),
          ),
        ),
      );

      await tester.longPress(find.byType(ProductCard));
      await tester.pumpAndSettle();

      expect(find.text('删除'), findsOneWidget);
    });
  });
}
```

### 带状态管理的 Widget 测试

```dart
testWidgets('加载状态显示进度条', (tester) async {
  final bloc = MockProductBloc();
  when(() => bloc.state).thenReturn(
    const ProductState(status: ProductStatus.loading),
  );
  whenListen(bloc, Stream<ProductState>.empty());

  await tester.pumpWidget(
    MaterialApp(
      home: BlocProvider<ProductBloc>.value(
        value: bloc,
        child: const ProductListPage(),
      ),
    ),
  );

  expect(find.byType(CircularProgressIndicator), findsOneWidget);
});
```

## Golden 测试

```dart
testWidgets('ProductCard 快照测试', (tester) async {
  await tester.pumpWidget(
    MaterialApp(
      home: Scaffold(
        body: ProductCard(
          product: Product(id: '1', name: '测试商品', price: 99.9),
        ),
      ),
    ),
  );

  await expectLater(
    find.byType(ProductCard),
    matchesGoldenFile('goldens/product_card.png'),
  );
});
```

```bash
# 生成 golden 基准图
flutter test --update-goldens
```

## 集成测试

```dart
// integration_test/checkout_flow_test.dart
import 'package:integration_test/integration_test.dart';

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  group('结算流程', () {
    testWidgets('从商品列表到下单成功', (tester) async {
      await tester.pumpWidget(const MyApp());
      await tester.pumpAndSettle();

      // 点击第一个商品
      await tester.tap(find.byType(ProductCard).first);
      await tester.pumpAndSettle();

      // 点击加入购物车
      await tester.tap(find.text('加入购物车'));
      await tester.pumpAndSettle();

      // 进入购物车
      await tester.tap(find.byIcon(Icons.shopping_cart));
      await tester.pumpAndSettle();

      // 点击结算
      await tester.tap(find.text('去结算'));
      await tester.pumpAndSettle();

      // 确认订单
      await tester.tap(find.text('提交订单'));
      await tester.pumpAndSettle();

      // 验证跳转到订单成功页
      expect(find.text('下单成功'), findsOneWidget);
    });
  });
}
```

## Mock 策略

### 使用 mocktail

```dart
// 定义 Mock
class MockApiClient extends Mock implements ApiClient {}

// 设置行为
when(() => mockApi.get('/users')).thenAnswer(
  (_) async => Response(data: [...], statusCode: 200),
);

// 验证调用
verify(() => mockApi.get('/users')).called(1);
verifyNever(() => mockApi.delete(any()));
```

### Fake 对象

```dart
class FakeUserRepository implements UserRepository {
  final List<User> _users = [];

  @override
  Future<List<User>> getAll() async => _users;

  @override
  Future<void> add(User user) async => _users.add(user);
}
```

## 测试命名规范

```dart
test('当 [条件] 时，应该 [预期结果]', () { ... });
test('添加已存在的商品时，数量应该累加', () { ... });
test('网络异常时，应该显示错误提示', () { ... });
```

## 运行测试

```bash
# 运行所有测试
flutter test

# 运行单个文件
flutter test test/services/cart_service_test.dart

# 运行带覆盖率
flutter test --coverage

# 查看覆盖率报告
genhtml coverage/lcov.info -o coverage/html
open coverage/html/index.html
```
