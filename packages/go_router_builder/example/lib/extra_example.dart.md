# extra_example.dart 代码解析

## 概述

`extra_example.dart` 是一个演示 go_router_builder 中 `$extra` 参数功能的示例文件。该示例展示了如何在类型安全的路由系统中传递非 URL 参数的数据对象，包括必需和可选两种使用场景。

## 核心概念：$extra 参数

在 go_router 中，`extra` 参数允许你传递无法通过 URL 编码的复杂对象。go_router_builder 通过 `$extra` 字段提供了类型安全的支持，让你可以在路由类中声明这些额外数据。

### 为什么需要 $extra？

- **复杂对象传递**：某些数据（如自定义类实例）无法序列化到 URL 中
- **类型安全**：通过类型系统确保传递的数据类型正确
- **可选性支持**：可以声明必需或可选的 extra 参数

## 代码结构分析

### Extra 数据类

```dart 28:32:example/lib/extra_example.dart
class Extra {
  const Extra(this.value);

  final int value;
}
```

这是一个简单的数据类，用于演示如何传递自定义对象。它包含一个 `int` 类型的 `value` 字段。在实际应用中，你可以使用任何可序列化的 Dart 类。

### 必需 Extra 参数的路由

```dart 34:43:example/lib/extra_example.dart
@TypedGoRoute<RequiredExtraRoute>(path: '/requiredExtra')
class RequiredExtraRoute extends GoRouteData with $RequiredExtraRoute {
  const RequiredExtraRoute({required this.$extra});

  final Extra $extra;

  @override
  Widget build(BuildContext context, GoRouterState state) =>
      RequiredExtraScreen(extra: $extra);
}
```

**关键特性**：

1. **`$extra` 字段**：使用 `$` 前缀标识这是一个特殊的 extra 参数
2. **必需参数**：通过 `required` 关键字声明，创建路由实例时必须提供
3. **类型安全**：`final Extra $extra` 确保只能传递 `Extra` 类型的对象
4. **自动传递**：生成的代码会自动将 `$extra` 传递给 go_router 的 `extra` 参数

**对应的 UI 组件**：

```dart 45:57:example/lib/extra_example.dart
class RequiredExtraScreen extends StatelessWidget {
  const RequiredExtraScreen({super.key, required this.extra});

  final Extra extra;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Required Extra')),
      body: Center(child: Text('Extra: ${extra.value}')),
    );
  }
}
```

### 可选 Extra 参数的路由

```dart 59:68:example/lib/extra_example.dart
@TypedGoRoute<OptionalExtraRoute>(path: '/optionalExtra')
class OptionalExtraRoute extends GoRouteData with $OptionalExtraRoute {
  const OptionalExtraRoute({this.$extra});

  final Extra? $extra;

  @override
  Widget build(BuildContext context, GoRouterState state) =>
      OptionalExtraScreen(extra: $extra);
}
```

**关键特性**：

1. **可选参数**：不使用 `required`，且类型为 `Extra?`（可空）
2. **灵活使用**：可以在创建路由时提供 extra，也可以不提供（为 `null`）
3. **空值处理**：UI 组件需要处理 `extra` 可能为 `null` 的情况

**对应的 UI 组件**：

```dart 70:82:example/lib/extra_example.dart
class OptionalExtraScreen extends StatelessWidget {
  const OptionalExtraScreen({super.key, this.extra});

  final Extra? extra;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Optional Extra')),
      body: Center(child: Text('Extra: ${extra?.value}')),
    );
  }
}
```

注意使用了 `extra?.value` 来安全地访问可能为 `null` 的值。

### 应用入口和导航

```dart 12:26:example/lib/extra_example.dart
void main() => runApp(const App());

final GoRouter _router = GoRouter(
  routes: $appRoutes,
  initialLocation: '/splash',
);

class App extends StatelessWidget {
  const App({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(routerConfig: _router);
  }
}
```

应用使用 `$appRoutes`（由代码生成器生成）来配置路由，初始位置设置为 `/splash`。

**导航示例页面**：

```dart 84:121:example/lib/extra_example.dart
@TypedGoRoute<SplashRoute>(path: '/splash')
class SplashRoute extends GoRouteData with $SplashRoute {
  const SplashRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) => const Splash();
}

class Splash extends StatelessWidget {
  const Splash({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Splash')),
      body: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: <Widget>[
          const Placeholder(),
          ElevatedButton(
            onPressed: () =>
                const RequiredExtraRoute($extra: Extra(1)).go(context),
            child: const Text('Required Extra'),
          ),
          ElevatedButton(
            onPressed: () =>
                const OptionalExtraRoute($extra: Extra(2)).go(context),
            child: const Text('Optional Extra'),
          ),
          ElevatedButton(
            onPressed: () => const OptionalExtraRoute().go(context),
            child: const Text('Optional Extra (null)'),
          ),
        ],
      ),
    );
  }
}
```

这个页面展示了三种导航方式：

1. **必需 extra 导航**：`RequiredExtraRoute($extra: Extra(1)).go(context)` - 必须提供 extra 参数
2. **可选 extra 导航（有值）**：`OptionalExtraRoute($extra: Extra(2)).go(context)` - 提供 extra 参数
3. **可选 extra 导航（无值）**：`OptionalExtraRoute().go(context)` - 不提供 extra 参数，值为 `null`

## 生成的代码分析

代码生成器会为每个路由生成相应的 mixin 和工厂方法。让我们看看关键部分：

### RequiredExtraRoute 的生成代码

```dart 22:45:example/lib/extra_example.g.dart
mixin $RequiredExtraRoute on GoRouteData {
  static RequiredExtraRoute _fromState(GoRouterState state) =>
      RequiredExtraRoute($extra: state.extra as Extra);

  RequiredExtraRoute get _self => this as RequiredExtraRoute;

  @override
  String get location => GoRouteData.$location('/requiredExtra');

  @override
  void go(BuildContext context) => context.go(location, extra: _self.$extra);

  @override
  Future<T?> push<T>(BuildContext context) =>
      context.push<T>(location, extra: _self.$extra);

  @override
  void pushReplacement(BuildContext context) =>
      context.pushReplacement(location, extra: _self.$extra);

  @override
  void replace(BuildContext context) =>
      context.replace(location, extra: _self.$extra);
}
```

**关键点**：

1. **`_fromState` 方法**：从 `GoRouterState` 中提取 `extra` 并转换为 `Extra` 类型
2. **导航方法**：所有导航方法（`go`、`push`、`pushReplacement`、`replace`）都会自动传递 `$extra` 参数
3. **类型转换**：使用 `as Extra` 进行类型断言（在实际应用中可能需要更安全的类型检查）

### OptionalExtraRoute 的生成代码

```dart 52:75:example/lib/extra_example.g.dart
mixin $OptionalExtraRoute on GoRouteData {
  static OptionalExtraRoute _fromState(GoRouterState state) =>
      OptionalExtraRoute($extra: state.extra as Extra?);

  OptionalExtraRoute get _self => this as OptionalExtraRoute;

  @override
  String get location => GoRouteData.$location('/optionalExtra');

  @override
  void go(BuildContext context) => context.go(location, extra: _self.$extra);

  @override
  Future<T?> push<T>(BuildContext context) =>
      context.push<T>(location, extra: _self.$extra);

  @override
  void pushReplacement(BuildContext context) =>
      context.pushReplacement(location, extra: _self.$extra);

  @override
  void replace(BuildContext context) =>
      context.replace(location, extra: _self.$extra);
}
```

与必需版本的主要区别是类型转换使用 `as Extra?`，允许 `null` 值。

## 使用场景

### 1. 传递复杂对象

当你需要传递无法序列化到 URL 的对象时，使用 `$extra`：

```dart
// 传递用户对象
final user = User(id: 1, name: 'John');
UserDetailRoute($extra: user).go(context);
```

### 2. 传递临时数据

对于不需要在 URL 中体现的临时数据：

```dart
// 传递编辑模式标志
EditRoute($extra: EditMode.create).go(context);
```

### 3. 向后兼容

可选 `$extra` 允许在不破坏现有代码的情况下添加新功能。

## 最佳实践

### 1. 选择必需还是可选

- **必需 `$extra`**：当数据对页面功能至关重要时使用
- **可选 `$extra`**：当数据只是增强功能或提供额外上下文时使用

### 2. 类型安全

始终使用具体的类型而不是 `dynamic`：

```dart
// ✅ 好
final Extra $extra;

// ❌ 避免
final dynamic $extra;
```

### 3. 空值处理

对于可选的 `$extra`，始终在 UI 组件中处理 `null` 情况：

```dart
Widget build(BuildContext context) {
  if (extra == null) {
    return const Text('No extra data');
  }
  return Text('Extra: ${extra.value}');
}
```

### 4. 数据类设计

确保 `$extra` 使用的类：

- 是不可变的（使用 `const` 构造函数）
- 实现了 `==` 和 `hashCode`（如果需要比较）
- 是可序列化的（如果需要持久化）

## 注意事项

1. **类型转换风险**：生成的代码使用 `as` 进行类型断言，如果类型不匹配会抛出异常。在生产环境中，你可能需要添加类型检查。

2. **URL 不可见**：`$extra` 数据不会出现在 URL 中，因此：
   - 无法通过 URL 直接访问带 extra 数据的页面
   - 浏览器前进/后退不会保留 extra 数据
   - 深度链接无法包含 extra 数据

3. **内存考虑**：extra 数据存储在内存中，对于大型对象需要注意内存使用。

## 总结

`extra_example.dart` 展示了 go_router_builder 中 `$extra` 参数的强大功能，它允许你在类型安全的路由系统中传递复杂对象。通过必需和可选两种模式，你可以根据实际需求灵活选择。记住，`$extra` 适用于无法或不应该出现在 URL 中的数据，为你的路由系统提供了额外的灵活性。
