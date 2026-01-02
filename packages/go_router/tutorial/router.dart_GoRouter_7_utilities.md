# GoRouter 工具方法和生命周期

本文档介绍 `GoRouter` 中的工具方法（`of()` 和 `maybeOf()`）、生命周期方法（`dispose()`）以及内部辅助方法（`_effectiveInitialLocation()`）。

## of() 和 maybeOf()

这两个静态方法用于从 widget 树中获取 `GoRouter` 实例。它们是 Flutter 中常见的 `of` 模式实现。

### of()

```dart 599:606:lib/src/router.dart
  /// Find the current GoRouter in the widget tree.
  static GoRouter of(BuildContext context) {
    final GoRouter? router = maybeOf(context);
    if (router == null) {
      throw FlutterError('No GoRouter found in context');
    }
    return router;
  }
```

### 说明

查找 widget 树中的当前 `GoRouter` 实例。如果找不到，会抛出 `FlutterError`。

### 参数

- **`context`**（必需）：BuildContext，用于在 widget 树中查找 `GoRouter`

### 返回值

返回 `GoRouter` 实例。

### 异常

如果 widget 树中不存在 `GoRouter`，会抛出 `FlutterError`，错误消息为 "No GoRouter found in context"。

### 使用示例

```dart
// 在 widget 的 build 方法中使用
Widget build(BuildContext context) {
  final router = GoRouter.of(context);
  return ElevatedButton(
    onPressed: () => router.go('/home'),
    child: Text('返回首页'),
  );
}

// 在回调中使用
void onButtonTap(BuildContext context) {
  final router = GoRouter.of(context);
  router.push('/details');
}
```

### maybeOf()

```dart 608:621:lib/src/router.dart
  /// The current GoRouter in the widget tree, if any.
  static GoRouter? maybeOf(BuildContext context) {
    final inherited =
        context
                .getElementForInheritedWidgetOfExactType<InheritedGoRouter>()
                ?.widget
            as InheritedGoRouter?;
    if (inherited != null) {
      return inherited.goRouter;
    }

    // Check if we're in a redirect context
    return Zone.current[currentRouterKey] as GoRouter?;
  }
```

### 说明

查找 widget 树中的当前 `GoRouter` 实例，如果找不到则返回 `null`。这是 `of()` 的安全版本。

### 参数

- **`context`**（必需）：BuildContext，用于在 widget 树中查找 `GoRouter`

### 返回值

返回 `GoRouter?`，如果找到则返回实例，否则返回 `null`。

### 查找机制

`maybeOf()` 使用两种方式查找 `GoRouter`：

1. **通过 InheritedWidget**：首先尝试从 widget 树中查找 `InheritedGoRouter`，这是 `GoRouter` 通过 `GoRouterDelegate` 的 `builderWithNav` 参数包装到 widget 树中的
2. **通过 Zone**：如果在 widget 树中找不到，会尝试从当前 Zone 中获取（用于重定向上下文中）

### 使用场景

- 不确定 `GoRouter` 是否存在时
- 需要可选地访问 `GoRouter` 时
- 在可能没有 `GoRouter` 的环境中（例如，某些测试场景）

### 使用示例

```dart
// 可选地访问 GoRouter
Widget build(BuildContext context) {
  final router = GoRouter.maybeOf(context);
  if (router != null) {
    return ElevatedButton(
      onPressed: () => router.go('/home'),
      child: Text('返回首页'),
    );
  } else {
    return Text('GoRouter 不可用');
  }
}
```

### 实现细节

#### InheritedGoRouter

`InheritedGoRouter` 是一个 `InheritedWidget` 的子类，用于在 widget 树中提供 `GoRouter` 实例。它通过 `GoRouterDelegate` 的 `builderWithNav` 参数包装到导航器中：

```dart
builderWithNav: (BuildContext context, Widget child) =>
    InheritedGoRouter(goRouter: this, child: child),
```

这使得任何在导航器下方的 widget 都可以通过 `of()` 或 `maybeOf()` 访问 `GoRouter`。

#### Zone 上下文

在某些情况下（例如，在重定向回调中），可能无法通过 widget 树访问 `GoRouter`。此时，`GoRouter` 会被存储在 Zone 中，使用 `currentRouterKey` 作为键。这允许在重定向上下文中访问 `GoRouter`。

### of() vs maybeOf()

| 特性 | `of()` | `maybeOf()` |
| --- | --- | --- |
| 返回值类型 | `GoRouter` | `GoRouter?` |
| 找不到时行为 | 抛出异常 | 返回 `null` |
| 使用场景 | 确定存在时使用 | 不确定是否存在时使用 |

## dispose()

```dart 623:628:lib/src/router.dart
  /// Disposes resource created by this object.
  void dispose() {
    _routingConfig.removeListener(_handleRoutingConfigChanged);
    routeInformationProvider.dispose();
    routerDelegate.dispose();
  }
```

### 说明

释放 `GoRouter` 创建的资源。应该在不再使用 `GoRouter` 时调用，以避免内存泄漏。

### 行为

`dispose()` 方法执行以下操作：

1. **移除路由配置监听器**：从 `_routingConfig` 中移除 `_handleRoutingConfigChanged` 监听器
2. **释放路由信息提供者**：调用 `routeInformationProvider.dispose()`
3. **释放路由委托**：调用 `routerDelegate.dispose()`

### 使用场景

通常在以下场景中调用 `dispose()`：

- 应用退出时
- 需要替换 `GoRouter` 实例时
- 在测试中清理资源时

### 使用示例

```dart
class MyApp extends StatefulWidget {
  @override
  State<MyApp> createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  late final GoRouter _router;

  @override
  void initState() {
    super.initState();
    _router = GoRouter(/* 配置参数 */);
  }

  @override
  void dispose() {
    _router.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: _router,
    );
  }
}
```

### 注意事项

- 调用 `dispose()` 后，不应再使用 `GoRouter` 实例
- 确保所有使用该 `GoRouter` 的 widget 都已经卸载
- 如果 `GoRouter` 是通过 `MaterialApp.router` 或其他方式管理的，通常不需要手动调用 `dispose()`，Flutter 框架会自动处理

## _effectiveInitialLocation()

```dart 630:650:lib/src/router.dart
  String _effectiveInitialLocation(String? initialLocation) {
    if (overridePlatformDefaultLocation) {
      // The initialLocation must not be null as it's already
      // verified by assert() during the initialization.
      return initialLocation!;
    }
    Uri platformDefaultUri = Uri.parse(
      WidgetsBinding.instance.platformDispatcher.defaultRouteName,
    );
    if (platformDefaultUri.hasEmptyPath) {
      platformDefaultUri = platformDefaultUri.replace(path: '/');
    }
    final platformDefault = platformDefaultUri.toString();
    if (initialLocation == null) {
      return platformDefault;
    } else if (platformDefault == '/') {
      return initialLocation;
    } else {
      return platformDefault;
    }
  }
```

### 说明

这是一个私有方法，用于计算有效的初始位置。它根据 `overridePlatformDefaultLocation` 和 `initialLocation` 参数决定使用哪个初始位置。

### 参数

- **`initialLocation`**：可选的初始位置字符串

### 返回值

返回有效的初始位置字符串。

### 逻辑

方法按以下逻辑决定初始位置：

1. **如果 `overridePlatformDefaultLocation` 为 `true`**：
   - 直接返回 `initialLocation!`（此时 `initialLocation` 必须不为 null，已在构造函数中通过断言验证）

2. **如果 `overridePlatformDefaultLocation` 为 `false`**（默认）：
   - 获取平台的默认路由名称（通常是深层链接的路径）
   - 如果平台默认路径为空，将其替换为 `/`
   - 如果 `initialLocation` 为 `null`，返回平台默认位置
   - 如果平台默认位置为 `/`，返回 `initialLocation`
   - 否则，返回平台默认位置（忽略 `initialLocation`）

### 设计意图

这个方法的逻辑体现了以下设计原则：

- **平台优先**：默认情况下，平台的默认位置（通常是深层链接）优先于手动设置的 `initialLocation`
- **灵活性**：通过 `overridePlatformDefaultLocation` 参数，允许开发者覆盖平台行为
- **向后兼容**：当平台默认位置为 `/` 时，使用 `initialLocation`（如果提供），保持向后兼容

### 使用场景

这个方法在构造函数中被调用，用于初始化 `routeInformationProvider`：

```dart
routeInformationProvider = GoRouteInformationProvider(
  initialLocation: _effectiveInitialLocation(initialLocation),
  // ...
);
```

### 示例

假设应用通过深层链接 `/user/123` 打开：

- **默认情况**（`overridePlatformDefaultLocation = false`，`initialLocation = '/home'`）：
  - 平台默认位置：`/user/123`
  - 返回：`/user/123`（使用平台默认位置）

- **覆盖平台默认位置**（`overridePlatformDefaultLocation = true`，`initialLocation = '/home'`）：
  - 返回：`/home`（使用 `initialLocation`）

- **平台默认位置为根路径**（`overridePlatformDefaultLocation = false`，`initialLocation = '/home'`）：
  - 平台默认位置：`/`
  - 返回：`/home`（使用 `initialLocation`）

## 最佳实践

1. **使用 `of()` 还是 `maybeOf()`**：
   - 确定 `GoRouter` 存在时使用 `of()`
   - 不确定时使用 `maybeOf()` 并检查 null

2. **资源清理**：
   - 如果手动创建了 `GoRouter`，记得在适当的时机调用 `dispose()`
   - 如果通过框架管理（如 `MaterialApp.router`），通常不需要手动调用

3. **初始位置设置**：
   - 大多数情况下使用默认行为（平台优先）
   - 仅在确实需要覆盖平台行为时设置 `overridePlatformDefaultLocation = true`
