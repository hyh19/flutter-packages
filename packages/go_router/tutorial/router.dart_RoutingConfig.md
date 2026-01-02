# RoutingConfig 类详解

`RoutingConfig` 是 GoRouter 中用于定义路由配置的参数集合。这个类主要用于通过 `GoRouter.routingConfig` 构造函数创建具有动态路由配置的 go router 实例。

## 类定义

```dart 43:123:lib/src/router.dart
/// A set of parameters that defines routing in GoRouter.
///
/// This is typically used with [GoRouter.routingConfig] to create a go router
/// with dynamic routing config.
///
/// See [routing_config.dart](https://github.com/flutter/packages/blob/main/packages/go_router/example/lib/routing_config.dart).
///
/// {@category Configuration}
class RoutingConfig {
  /// Creates a routing config.
  ///
  /// The [routes] must not be empty.
  const RoutingConfig({
    required this.routes,
    this.onEnter,
    this.redirect = _defaultRedirect,
    this.redirectLimit = 5,
  });

  static FutureOr<String?> _defaultRedirect(
    BuildContext context,
    GoRouterState state,
  ) => null;

  /// The supported routes.
  ///
  /// The `routes` list specifies the top-level routes for the app. It must not be
  /// empty and must contain an [GoRoute] to match `/`.
  ///
  /// See [GoRouter].
  final List<RouteBase> routes;

  /// The top-level callback allows the app to redirect to a new location.
  ///
  /// Alternatively, you can specify a redirect for an individual route using
  /// [GoRoute.redirect]. If [BuildContext.dependOnInheritedWidgetOfExactType] is
  /// used during the redirection (which is how `of` methods are usually
  /// implemented), a re-evaluation will be triggered when the [InheritedWidget]
  /// changes.
  ///
  /// This legacy callback remains supported alongside [onEnter]. If both are
  /// provided, [onEnter] executes first and may block the navigation. When
  /// allowed, this callback runs once per navigation cycle before any
  /// route-level redirects.
  final GoRouterRedirect redirect;

  /// The maximum number of redirection allowed.
  ///
  /// See [GoRouter].
  final int redirectLimit;

  /// A callback invoked for every incoming route before it is processed.
  ///
  /// This callback allows you to control navigation by inspecting the incoming
  /// route and conditionally preventing the navigation. Return [Allow] to proceed
  /// with navigation or [Block] to cancel it. Both can optionally include an
  /// `then` callback for deferred actions.
  ///
  /// When a deep link opens the app and `onEnter` returns [Block], GoRouter
  /// will stay on the current route or redirect to the initial route.
  ///
  /// Example:
  /// ```dart
  /// final GoRouter router = GoRouter(
  ///   routes: [...],
  ///   onEnter: (BuildContext context, GoRouterState current,
  ///             GoRouterState next, GoRouter router) async {
  ///     if (next.uri.path == '/login' && isUserLoggedIn()) {
  ///       return const Block.stop(); // Prevent navigation to /login
  ///     }
  ///     if (next.uri.path == '/protected' && !isUserLoggedIn()) {
  ///       // Block and redirect to login
  ///       return Block.then(() => router.go('/login?from=${next.uri}'));
  ///     }
  ///     return const Allow(); // Allow navigation
  ///   },
  /// );
  /// ```
  final OnEnter? onEnter;
}
```

## 主要用途

`RoutingConfig` 类封装了 GoRouter 路由系统的核心配置参数，主要用于：

1. **动态路由配置**：与 `GoRouter.routingConfig` 构造函数配合使用，支持在运行时动态更改路由配置
2. **路由定义**：通过 `routes` 属性定义应用的顶级路由列表
3. **导航控制**：通过 `onEnter` 回调实现路由守卫功能，控制导航行为
4. **重定向逻辑**：通过 `redirect` 回调实现路由重定向

## 属性详解

### routes（必需）

```dart 67:73:lib/src/router.dart
  /// The supported routes.
  ///
  /// The `routes` list specifies the top-level routes for the app. It must not be
  /// empty and must contain an [GoRoute] to match `/`.
  ///
  /// See [GoRouter].
  final List<RouteBase> routes;
```

- **类型**：`List<RouteBase>`
- **必需性**：必需（构造函数中使用 `required` 标记）
- **说明**：
  - 定义了应用的顶级路由列表
  - 列表不能为空
  - 必须包含一个匹配 `/` 路径的 `GoRoute`
  - 这是整个路由系统的基础配置

### onEnter（可选）

```dart 94:121:lib/src/router.dart
  /// A callback invoked for every incoming route before it is processed.
  ///
  /// This callback allows you to control navigation by inspecting the incoming
  /// route and conditionally preventing the navigation. Return [Allow] to proceed
  /// with navigation or [Block] to cancel it. Both can optionally include an
  /// `then` callback for deferred actions.
  ///
  /// When a deep link opens the app and `onEnter` returns [Block], GoRouter
  /// will stay on the current route or redirect to the initial route.
  ///
  /// Example:
  /// ```dart
  /// final GoRouter router = GoRouter(
  ///   routes: [...],
  ///   onEnter: (BuildContext context, GoRouterState current,
  ///             GoRouterState next, GoRouter router) async {
  ///     if (next.uri.path == '/login' && isUserLoggedIn()) {
  ///       return const Block.stop(); // Prevent navigation to /login
  ///     }
  ///     if (next.uri.path == '/protected' && !isUserLoggedIn()) {
  ///       // Block and redirect to login
  ///       return Block.then(() => router.go('/login?from=${next.uri}'));
  ///     }
  ///     return const Allow(); // Allow navigation
  ///   },
  /// );
  /// ```
  final OnEnter? onEnter;
```

- **类型**：`OnEnter?`（可空类型）
- **必需性**：可选
- **说明**：
  - 在每次导航处理之前调用的回调函数
  - 可以检查即将导航的路由状态，并决定是否允许导航
  - 返回 `Allow` 允许导航继续，返回 `Block` 阻止导航
  - 支持使用 `Block.then()` 在阻止导航后执行延迟操作（如重定向）
  - 当通过深度链接打开应用且 `onEnter` 返回 `Block` 时，GoRouter 会停留在当前路由或重定向到初始路由

`OnEnter` 的类型签名定义如下：

```dart 35:41:lib/src/router.dart
typedef OnEnter =
    FutureOr<OnEnterResult> Function(
      BuildContext context,
      GoRouterState currentState,
      GoRouterState nextState,
      GoRouter goRouter,
    );
```

### redirect（可选）

```dart 75:87:lib/src/router.dart
  /// The top-level callback allows the app to redirect to a new location.
  ///
  /// Alternatively, you can specify a redirect for an individual route using
  /// [GoRoute.redirect]. If [BuildContext.dependOnInheritedWidgetOfExactType] is
  /// used during the redirection (which is how `of` methods are usually
  /// implemented), a re-evaluation will be triggered when the [InheritedWidget]
  /// changes.
  ///
  /// This legacy callback remains supported alongside [onEnter]. If both are
  /// provided, [onEnter] executes first and may block the navigation. When
  /// allowed, this callback runs once per navigation cycle before any
  /// route-level redirects.
  final GoRouterRedirect redirect;
```

- **类型**：`GoRouterRedirect`
- **必需性**：可选（默认值为 `_defaultRedirect`，直接返回 `null`，表示不重定向）
- **说明**：
  - 顶级重定向回调，允许应用重定向到新位置
  - 这是一个遗留的回调，与 `onEnter` 一起使用时仍然被支持
  - **执行顺序**：如果同时提供了 `onEnter` 和 `redirect`，`onEnter` 会首先执行并可能阻止导航；当导航被允许时，`redirect` 回调会在每个导航周期中运行一次，在路由级别的重定向之前执行
  - 如果在重定向过程中使用了 `BuildContext.dependOnInheritedWidgetOfExactType`（通常通过 `of` 方法实现），当 `InheritedWidget` 改变时会触发重新评估
  - 也可以在单个路由上使用 `GoRoute.redirect` 来指定重定向逻辑

默认的 `_defaultRedirect` 实现：

```dart 62:65:lib/src/router.dart
  static FutureOr<String?> _defaultRedirect(
    BuildContext context,
    GoRouterState state,
  ) => null;
```

### redirectLimit（可选）

```dart 89:92:lib/src/router.dart
  /// The maximum number of redirection allowed.
  ///
  /// See [GoRouter].
  final int redirectLimit;
```

- **类型**：`int`
- **必需性**：可选（默认值为 `5`）
- **说明**：
  - 允许的最大重定向次数
  - 用于防止无限重定向循环
  - 当重定向次数超过此限制时，GoRouter 会停止重定向并显示错误

## 构造函数

```dart 52:60:lib/src/router.dart
  /// Creates a routing config.
  ///
  /// The [routes] must not be empty.
  const RoutingConfig({
    required this.routes,
    this.onEnter,
    this.redirect = _defaultRedirect,
    this.redirectLimit = 5,
  });
```

构造函数特点：

- 使用 `const` 关键字，支持编译时常量
- `routes` 是必需参数
- `onEnter` 是可选的，默认为 `null`
- `redirect` 是可选的，默认为 `_defaultRedirect`（返回 `null`，不进行重定向）
- `redirectLimit` 是可选的，默认为 `5`

## 执行顺序

当同时提供 `onEnter` 和 `redirect` 时，执行顺序如下：

1. **`onEnter`**：首先执行，可以阻止导航
2. **`redirect`**（如果导航被允许）：在同一个导航周期中运行，在路由级别的重定向之前执行
3. **路由级别的 `GoRoute.redirect`**：最后执行

## 使用示例

### 基本用法

```dart
final routingConfig = RoutingConfig(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => HomeScreen(),
    ),
    GoRoute(
      path: '/login',
      builder: (context, state) => LoginScreen(),
    ),
  ],
);
```

### 带导航守卫的配置

```dart
final routingConfig = RoutingConfig(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => HomeScreen(),
    ),
    GoRoute(
      path: '/login',
      builder: (context, state) => LoginScreen(),
    ),
    GoRoute(
      path: '/protected',
      builder: (context, state) => ProtectedScreen(),
    ),
  ],
  onEnter: (context, current, next, router) async {
    // 如果用户已登录，阻止访问登录页面
    if (next.uri.path == '/login' && isUserLoggedIn()) {
      return const Block.stop();
    }
    
    // 如果用户未登录，阻止访问受保护页面并重定向到登录页
    if (next.uri.path == '/protected' && !isUserLoggedIn()) {
      return Block.then(() => router.go('/login?from=${next.uri}'));
    }
    
    // 允许导航
    return const Allow();
  },
);
```

### 带重定向的配置

```dart
final routingConfig = RoutingConfig(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => HomeScreen(),
    ),
  ],
  redirect: (context, state) {
    // 如果未登录，重定向到登录页
    if (!isUserLoggedIn() && state.uri.path != '/login') {
      return '/login';
    }
    return null; // 不重定向
  },
);
```

### 动态路由配置

`RoutingConfig` 主要用于与 `GoRouter.routingConfig` 构造函数配合使用，实现动态路由配置：

```dart
final routingConfigNotifier = ValueNotifier<RoutingConfig>(
  RoutingConfig(
    routes: initialRoutes,
  ),
);

final router = GoRouter.routingConfig(
  routingConfig: routingConfigNotifier,
  // ... 其他配置
);

// 运行时更新路由配置
routingConfigNotifier.value = RoutingConfig(
  routes: updatedRoutes,
  onEnter: newOnEnter,
);
```

## 相关类型

- **`RouteBase`**：路由基类，`routes` 列表中元素的类型
- **`GoRouterRedirect`**：重定向回调的类型别名
- **`OnEnter`**：导航守卫回调的类型别名，返回 `OnEnterResult`（`Allow` 或 `Block`）
- **`GoRouterState`**：路由状态对象，包含当前路由和即将导航到的路由信息
- **`GoRouter`**：GoRouter 主类，使用 `RoutingConfig` 创建路由器实例

## 注意事项

1. **`routes` 不能为空**：必须至少提供一个路由定义
2. **必须包含根路由**：`routes` 列表中必须包含一个匹配 `/` 路径的 `GoRoute`
3. **`onEnter` 和 `redirect` 的配合**：如果同时提供两者，`onEnter` 会先执行，只有导航被允许时才会执行 `redirect`
4. **重定向限制**：注意 `redirectLimit` 的设置，避免意外的无限重定向循环
5. **性能考虑**：`onEnter` 回调在每个导航之前都会执行，应该保持逻辑简单高效
