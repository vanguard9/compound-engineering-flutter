---
name: widget-patterns
description: Flutter Widget 设计模式和最佳实践。在构建 UI 组件、讨论 Widget 设计或需要常用 Widget 模式参考时使用。涵盖布局模式、列表优化、响应式设计、自适应 Widget 和动画。
---

# Flutter Widget 设计模式

> 用组合构建复杂 UI，用模式保持一致性。

## 常用布局模式

### 1. 安全区域 + 脚手架

```dart
/// 标准页面骨架
class StandardPage extends StatelessWidget {
  const StandardPage({
    super.key,
    required this.title,
    required this.body,
    this.floatingActionButton,
    this.bottomNavigationBar,
  });

  final String title;
  final Widget body;
  final Widget? floatingActionButton;
  final Widget? bottomNavigationBar;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(title)),
      body: SafeArea(child: body),
      floatingActionButton: floatingActionButton,
      bottomNavigationBar: bottomNavigationBar,
    );
  }
}
```

### 2. 加载-错误-数据三态

```dart
/// 异步数据展示的通用模式
class AsyncDataView<T> extends StatelessWidget {
  const AsyncDataView({
    super.key,
    required this.isLoading,
    required this.error,
    required this.data,
    required this.builder,
    this.onRetry,
  });

  final bool isLoading;
  final String? error;
  final T? data;
  final Widget Function(T data) builder;
  final VoidCallback? onRetry;

  @override
  Widget build(BuildContext context) {
    if (isLoading) {
      return const Center(child: CircularProgressIndicator());
    }
    if (error != null) {
      return Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Text(error!, style: const TextStyle(color: Colors.red)),
            if (onRetry != null) ...[
              const SizedBox(height: 16),
              ElevatedButton(onPressed: onRetry, child: const Text('重试')),
            ],
          ],
        ),
      );
    }
    if (data != null) {
      return builder(data as T);
    }
    return const SizedBox.shrink();
  }
}
```

### 3. 空状态

```dart
class EmptyStateView extends StatelessWidget {
  const EmptyStateView({
    super.key,
    required this.icon,
    required this.title,
    this.subtitle,
    this.action,
    this.onAction,
  });

  final IconData icon;
  final String title;
  final String? subtitle;
  final String? action;
  final VoidCallback? onAction;

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Padding(
        padding: const EdgeInsets.all(32),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Icon(icon, size: 64, color: Colors.grey),
            const SizedBox(height: 16),
            Text(title, style: Theme.of(context).textTheme.titleMedium),
            if (subtitle != null) ...[
              const SizedBox(height: 8),
              Text(subtitle!, style: Theme.of(context).textTheme.bodyMedium),
            ],
            if (action != null && onAction != null) ...[
              const SizedBox(height: 24),
              ElevatedButton(onPressed: onAction, child: Text(action!)),
            ],
          ],
        ),
      ),
    );
  }
}
```

## 列表模式

### 高性能长列表

```dart
/// 分页加载列表
class PaginatedListView<T> extends StatefulWidget {
  const PaginatedListView({
    super.key,
    required this.items,
    required this.itemBuilder,
    required this.onLoadMore,
    this.isLoading = false,
    this.hasMore = true,
  });

  final List<T> items;
  final Widget Function(BuildContext, T) itemBuilder;
  final VoidCallback onLoadMore;
  final bool isLoading;
  final bool hasMore;

  @override
  State<PaginatedListView<T>> createState() => _PaginatedListViewState<T>();
}

class _PaginatedListViewState<T> extends State<PaginatedListView<T>> {
  final _scrollController = ScrollController();

  @override
  void initState() {
    super.initState();
    _scrollController.addListener(_onScroll);
  }

  void _onScroll() {
    if (_scrollController.position.pixels >=
        _scrollController.position.maxScrollExtent - 200) {
      if (!widget.isLoading && widget.hasMore) {
        widget.onLoadMore();
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      controller: _scrollController,
      itemCount: widget.items.length + (widget.hasMore ? 1 : 0),
      itemBuilder: (context, index) {
        if (index == widget.items.length) {
          return const Center(
            child: Padding(
              padding: EdgeInsets.all(16),
              child: CircularProgressIndicator(),
            ),
          );
        }
        return widget.itemBuilder(context, widget.items[index]);
      },
    );
  }

  @override
  void dispose() {
    _scrollController.dispose();
    super.dispose();
  }
}
```

### Sliver 组合列表

```dart
/// 多区段组合列表
CustomScrollView(
  slivers: [
    // 吸顶头部
    const SliverAppBar(
      floating: true,
      title: Text('商城'),
    ),
    // 轮播图
    SliverToBoxAdapter(
      child: BannerCarousel(banners: banners),
    ),
    // 分类网格
    SliverGrid(
      gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 4,
        mainAxisSpacing: 8,
        crossAxisSpacing: 8,
      ),
      delegate: SliverChildBuilderDelegate(
        (context, index) => CategoryItem(category: categories[index]),
        childCount: categories.length,
      ),
    ),
    // 区域标题
    const SliverToBoxAdapter(
      child: SectionHeader(title: '热门商品'),
    ),
    // 商品列表
    SliverList(
      delegate: SliverChildBuilderDelegate(
        (context, index) => ProductCard(product: products[index]),
        childCount: products.length,
      ),
    ),
  ],
)
```

## 响应式设计

### 自适应布局

```dart
class ResponsiveBuilder extends StatelessWidget {
  const ResponsiveBuilder({
    super.key,
    required this.mobile,
    this.tablet,
    this.desktop,
  });

  final Widget mobile;
  final Widget? tablet;
  final Widget? desktop;

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        if (constraints.maxWidth >= 1200 && desktop != null) {
          return desktop!;
        }
        if (constraints.maxWidth >= 600 && tablet != null) {
          return tablet!;
        }
        return mobile;
      },
    );
  }
}

// 使用
ResponsiveBuilder(
  mobile: const ProductListMobile(),
  tablet: const ProductGridTablet(),
  desktop: const ProductGridDesktop(),
)
```

## 表单模式

### 带验证的表单

```dart
class LoginForm extends StatefulWidget {
  const LoginForm({super.key, required this.onSubmit});

  final void Function(String email, String password) onSubmit;

  @override
  State<LoginForm> createState() => _LoginFormState();
}

class _LoginFormState extends State<LoginForm> {
  final _formKey = GlobalKey<FormState>();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Form(
      key: _formKey,
      child: Column(
        children: [
          TextFormField(
            controller: _emailController,
            decoration: const InputDecoration(labelText: '邮箱'),
            keyboardType: TextInputType.emailAddress,
            validator: (value) {
              if (value == null || value.isEmpty) return '请输入邮箱';
              if (!value.contains('@')) return '邮箱格式不正确';
              return null;
            },
          ),
          const SizedBox(height: 16),
          TextFormField(
            controller: _passwordController,
            decoration: const InputDecoration(labelText: '密码'),
            obscureText: true,
            validator: (value) {
              if (value == null || value.isEmpty) return '请输入密码';
              if (value.length < 6) return '密码至少 6 位';
              return null;
            },
          ),
          const SizedBox(height: 24),
          ElevatedButton(
            onPressed: () {
              if (_formKey.currentState!.validate()) {
                widget.onSubmit(
                  _emailController.text,
                  _passwordController.text,
                );
              }
            },
            child: const Text('登录'),
          ),
        ],
      ),
    );
  }

  @override
  void dispose() {
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }
}
```

## 动画模式

### 隐式动画

```dart
// 简单的属性变化动画
AnimatedContainer(
  duration: const Duration(milliseconds: 300),
  curve: Curves.easeInOut,
  width: isExpanded ? 200 : 100,
  height: isExpanded ? 200 : 100,
  color: isSelected ? Colors.blue : Colors.grey,
  child: child,
)

// 渐显渐隐
AnimatedOpacity(
  duration: const Duration(milliseconds: 200),
  opacity: isVisible ? 1.0 : 0.0,
  child: child,
)

// 交叉渐变切换
AnimatedSwitcher(
  duration: const Duration(milliseconds: 300),
  child: isLoading
      ? const CircularProgressIndicator(key: ValueKey('loading'))
      : ContentWidget(key: const ValueKey('content')),
)
```

### 显式动画（Hero）

```dart
// 列表页
Hero(
  tag: 'product-image-${product.id}',
  child: Image.network(product.imageUrl),
)

// 详情页
Hero(
  tag: 'product-image-${product.id}',
  child: Image.network(product.imageUrl),
)
```

## 设计要点总结

1. **能 const 就 const** — 减少不必要的 rebuild
2. **能 Stateless 就 Stateless** — 避免不必要的状态复杂性
3. **组合优于继承** — 用小 Widget 拼装大 Widget
4. **Key 用在正确的地方** — 列表项和动态切换的场景
5. **尾随逗号** — 保持代码格式化后的可读性
