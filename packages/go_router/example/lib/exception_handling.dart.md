# exception_handling.dart 代码解析

本文档详细解析了 `exception_handling.dart` 文件，这是一个展示如何使用 `GoRouter.onException` 来处理路由异常和重定向的示例应用。

## 概述

这个示例应用演示了如何在 `go_router` 中使用 `onException` 回调来处理路由异常。当用户访问未知路由或发生异常时，应用会自动重定向到 404 页面。

## 主要组件

### 1. 路由配置

```dart 18:36:example/lib/exception_handling.dart
final GoRouter _router = GoRouter(
  onException: (_, GoRouterState state, GoRouter router) {
    router.go('/404', extra: state.uri.toString());
  },
  routes: <RouteBase>[
    GoRoute(
      path: '/',
      builder: (BuildContext context, GoRouterState state) {
        return const HomeScreen();
      },
    ),
    GoRoute(
      path: '/404',
      builder: (BuildContext context, GoRouterState state) {
        return NotFoundScreen(uri: state.extra as String? ?? '');
      },
    ),
  ],
);
```

**关键特性**：

- **`onException` 回调**：当路由配置无法处理用户请求时（例如访问未知 URL），会触发此回调。回调接收三个参数：
  - 第一个参数（`_`）：异常对象，在此示例中被忽略
  - `GoRouterState state`：当前的路由状态，包含请求的 URI 等信息
  - `GoRouter router`：路由器实例，用于执行导航操作

- **异常处理逻辑**：当发生异常时，使用 `router.go('/404', extra: state.uri.toString())` 重定向到 404 页面，并将原始 URI 作为额外数据传递。

- **路由定义**：
  - `/`：首页路由，映射到 `HomeScreen`
  - `/404`：404 错误页面路由，映射到 `NotFoundScreen`

### 2. 应用入口

```dart 15:15:example/lib/exception_handling.dart
void main() => runApp(const MyApp());
```

应用入口函数，启动 `MyApp` 组件。

### 3. 主应用组件

```dart 39:47:example/lib/exception_handling.dart
class MyApp extends StatelessWidget {
  /// Constructs a [MyApp]
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(routerConfig: _router);
  }
}
```

`MyApp` 是一个无状态的 Widget，使用 `MaterialApp.router` 并传入配置好的 `_router` 作为路由配置。

### 4. 首页组件

```dart 50:66:example/lib/exception_handling.dart
class HomeScreen extends StatelessWidget {
  /// Constructs a [HomeScreen]
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Home Screen')),
      body: Center(
        child: ElevatedButton(
          onPressed: () => context.go('/some-unknown-route'),
          child: const Text('Simulates user entering unknown url'),
        ),
      ),
    );
  }
}
```

`HomeScreen` 是应用的首页，包含一个按钮用于模拟用户访问未知路由的场景。点击按钮会调用 `context.go('/some-unknown-route')`，这会触发 `onException` 回调，因为 `/some-unknown-route` 不在路由配置中。

### 5. 404 错误页面组件

```dart 69:83:example/lib/exception_handling.dart
class NotFoundScreen extends StatelessWidget {
  /// Constructs a [HomeScreen]
  const NotFoundScreen({super.key, required this.uri});

  /// The uri that can not be found.
  final String uri;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Page Not Found')),
      body: Center(child: Text("Can't find a page for: $uri")),
    );
  }
}
```

`NotFoundScreen` 是 404 错误页面，接收一个必需的 `uri` 参数，用于显示无法找到的页面 URI。这个 URI 是通过 `onException` 回调中的 `extra` 参数传递的。

## 工作流程

1. **正常访问**：用户访问 `/` 时，显示 `HomeScreen`。

2. **异常触发**：当用户访问未知路由（如 `/some-unknown-route`）时：
   - `go_router` 无法在路由配置中找到匹配的路由
   - 抛出 `GoException` 异常
   - 触发 `onException` 回调

3. **异常处理**：在 `onException` 回调中：
   - 获取当前请求的 URI（`state.uri.toString()`）
   - 使用 `router.go('/404', extra: ...)` 重定向到 404 页面
   - 将原始 URI 作为 `extra` 参数传递

4. **显示错误页面**：`NotFoundScreen` 接收传递的 URI 并显示错误信息。

## 使用场景

这个示例适用于以下场景：

- **处理未知路由**：当用户输入错误的 URL 或访问不存在的页面时
- **统一错误处理**：集中处理路由相关的异常
- **用户友好的错误提示**：向用户显示清晰的错误信息，而不是默认的错误页面

## 注意事项

1. **`onException` 的优先级**：根据 `go_router` 文档，`GoRouter.onException` 会覆盖其他异常处理 API（如 `errorBuilder` 和 `errorPageBuilder`）。

2. **异常类型**：`onException` 主要用于处理 `GoException`，这是当路由配置无法处理请求时抛出的异常。对于 `GoError` 和 `AssertionError`（代码使用错误导致的），不应该捕获，而应该修复代码。

3. **数据传递**：使用 `extra` 参数可以在页面间传递额外的数据，如示例中传递原始 URI。

4. **类型安全**：在 `NotFoundScreen` 中，使用 `state.extra as String? ?? ''` 进行安全的类型转换，如果 `extra` 为 `null` 则使用空字符串作为默认值。

## 扩展建议

在实际应用中，你可以：

- 根据不同的异常类型执行不同的处理逻辑
- 记录异常信息用于调试或分析
- 根据用户权限重定向到不同的页面
- 显示更丰富的错误信息，包括返回首页的按钮
