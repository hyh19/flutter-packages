# ShellRouteExampleApp 类详解

## 概述

`ShellRouteExampleApp` 是一个演示如何使用 `ShellRoute` 实现嵌套导航的示例应用。该类展示了如何在 Flutter 中使用 `go_router` 包创建一个带有底部导航栏的应用，并演示了不同导航器层级的页面跳转行为。

## 类定义

```dart 28:114:example/lib/shell_route.dart
/// An example demonstrating how to use [ShellRoute]
class ShellRouteExampleApp extends StatelessWidget {
  /// Creates a [ShellRouteExampleApp]
  ShellRouteExampleApp({super.key});

  final GoRouter _router = GoRouter(
    navigatorKey: _rootNavigatorKey,
    initialLocation: '/a',
    debugLogDiagnostics: true,
    routes: <RouteBase>[
      /// Application shell
      ShellRoute(
        navigatorKey: _shellNavigatorKey,
        builder: (BuildContext context, GoRouterState state, Widget child) {
          return ScaffoldWithNavBar(child: child);
        },
        routes: <RouteBase>[
          /// The first screen to display in the bottom navigation bar.
          GoRoute(
            path: '/a',
            builder: (BuildContext context, GoRouterState state) {
              return const ScreenA();
            },
            routes: <RouteBase>[
              // The details screen to display stacked on the inner Navigator.
              // This will cover screen A but not the application shell.
              GoRoute(
                path: 'details',
                builder: (BuildContext context, GoRouterState state) {
                  return const DetailsScreen(label: 'A');
                },
              ),
            ],
          ),

          /// Displayed when the second item in the the bottom navigation bar is
          /// selected.
          GoRoute(
            path: '/b',
            builder: (BuildContext context, GoRouterState state) {
              return const ScreenB();
            },
            routes: <RouteBase>[
              /// Same as "/a/details", but displayed on the root Navigator by
              /// specifying [parentNavigatorKey]. This will cover both screen B
              /// and the application shell.
              GoRoute(
                path: 'details',
                parentNavigatorKey: _rootNavigatorKey,
                builder: (BuildContext context, GoRouterState state) {
                  return const DetailsScreen(label: 'B');
                },
              ),
            ],
          ),

          /// The third screen to display in the bottom navigation bar.
          GoRoute(
            path: '/c',
            builder: (BuildContext context, GoRouterState state) {
              return const ScreenC();
            },
            routes: <RouteBase>[
              // The details screen to display stacked on the inner Navigator.
              // This will cover screen C but not the application shell.
              GoRoute(
                path: 'details',
                builder: (BuildContext context, GoRouterState state) {
                  return const DetailsScreen(label: 'C');
                },
              ),
            ],
          ),
        ],
      ),
    ],
  );

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'Flutter Demo',
      theme: ThemeData(primarySwatch: Colors.blue),
      routerConfig: _router,
    );
  }
}
```

## 核心概念

### ShellRoute 的作用

`ShellRoute` 是 `go_router` 提供的一种特殊路由类型，它允许在应用的路由树中插入一个额外的 `Navigator`。这个 `Navigator` 可以用来显示一些持久性的 UI 组件（如底部导航栏、侧边栏等），而子路由则在这个 `Navigator` 内部进行切换。

### 导航器层级

该示例使用了两个导航器：

1. **根导航器**（`_rootNavigatorKey`）：应用的最顶层导航器
2. **Shell 导航器**（`_shellNavigatorKey`）：位于 `ShellRoute` 内部的导航器，用于管理底部导航栏内的页面切换

## 代码结构分析

### 构造函数

```dart 30:31:example/lib/shell_route.dart
  /// Creates a [ShellRouteExampleApp]
  ShellRouteExampleApp({super.key});
```

这是一个标准的 `StatelessWidget` 构造函数，使用 `super.key` 将 key 传递给父类。

### GoRouter 配置

```dart 33:104:example/lib/shell_route.dart
  final GoRouter _router = GoRouter(
    navigatorKey: _rootNavigatorKey,
    initialLocation: '/a',
    debugLogDiagnostics: true,
    routes: <RouteBase>[
      /// Application shell
      ShellRoute(
        navigatorKey: _shellNavigatorKey,
        builder: (BuildContext context, GoRouterState state, Widget child) {
          return ScaffoldWithNavBar(child: child);
        },
        routes: <RouteBase>[
          // ... 子路由
        ],
      ),
    ],
  );
```

#### 配置参数说明

- **`navigatorKey: _rootNavigatorKey`**：指定根导航器的 key，用于控制整个应用的导航栈
- **`initialLocation: '/a'`**：设置应用启动时的初始路由路径
- **`debugLogDiagnostics: true`**：启用调试日志，方便开发时查看路由信息
- **`routes`**：定义应用的路由配置列表

### ShellRoute 配置

```dart 39:43:example/lib/shell_route.dart
      ShellRoute(
        navigatorKey: _shellNavigatorKey,
        builder: (BuildContext context, GoRouterState state, Widget child) {
          return ScaffoldWithNavBar(child: child);
        },
        routes: <RouteBase>[
```

`ShellRoute` 的关键特性：

- **`navigatorKey: _shellNavigatorKey`**：为 Shell 导航器指定一个独立的 key
- **`builder`**：接收三个参数：
  - `context`：构建上下文
  - `state`：当前路由状态
  - `child`：子路由对应的 widget（在这里是 `ScreenA`、`ScreenB` 或 `ScreenC`）
- **`routes`**：定义在 Shell 内部显示的子路由

### 路由配置详解

#### 路由 A：使用 Shell 导航器

```dart 46:61:example/lib/shell_route.dart
          GoRoute(
            path: '/a',
            builder: (BuildContext context, GoRouterState state) {
              return const ScreenA();
            },
            routes: <RouteBase>[
              // The details screen to display stacked on the inner Navigator.
              // This will cover screen A but not the application shell.
              GoRoute(
                path: 'details',
                builder: (BuildContext context, GoRouterState state) {
                  return const DetailsScreen(label: 'A');
                },
              ),
            ],
          ),
```

**特点**：

- 路径为 `/a`，对应 `ScreenA` 页面
- 子路由 `details` 的完整路径是 `/a/details`
- **没有指定 `parentNavigatorKey`**，因此 `details` 页面会使用默认的 Shell 导航器
- 当跳转到 `/a/details` 时，详情页会覆盖 `ScreenA`，但**不会覆盖底部导航栏**（因为底部导航栏在 Shell 导航器之外）

#### 路由 B：使用根导航器

```dart 65:82:example/lib/shell_route.dart
          GoRoute(
            path: '/b',
            builder: (BuildContext context, GoRouterState state) {
              return const ScreenB();
            },
            routes: <RouteBase>[
              /// Same as "/a/details", but displayed on the root Navigator by
              /// specifying [parentNavigatorKey]. This will cover both screen B
              /// and the application shell.
              GoRoute(
                path: 'details',
                parentNavigatorKey: _rootNavigatorKey,
                builder: (BuildContext context, GoRouterState state) {
                  return const DetailsScreen(label: 'B');
                },
              ),
            ],
          ),
```

**特点**：

- 路径为 `/b`，对应 `ScreenB` 页面
- 子路由 `details` 的完整路径是 `/b/details`
- **指定了 `parentNavigatorKey: _rootNavigatorKey`**，因此 `details` 页面会使用根导航器
- 当跳转到 `/b/details` 时，详情页会覆盖 `ScreenB`**以及整个应用 Shell**（包括底部导航栏）

#### 路由 C：使用 Shell 导航器

```dart 85:100:example/lib/shell_route.dart
          GoRoute(
            path: '/c',
            builder: (BuildContext context, GoRouterState state) {
              return const ScreenC();
            },
            routes: <RouteBase>[
              // The details screen to display stacked on the inner Navigator.
              // This will cover screen C but not the application shell.
              GoRoute(
                path: 'details',
                builder: (BuildContext context, GoRouterState state) {
                  return const DetailsScreen(label: 'C');
                },
              ),
            ],
          ),
```

**特点**：

- 与路由 A 类似，使用 Shell 导航器
- 详情页不会覆盖底部导航栏

### build 方法

```dart 106:113:example/lib/shell_route.dart
  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'Flutter Demo',
      theme: ThemeData(primarySwatch: Colors.blue),
      routerConfig: _router,
    );
  }
```

使用 `MaterialApp.router` 构造函数，这是使用 `go_router` 的标准方式：

- **`routerConfig: _router`**：将配置好的 `GoRouter` 实例传递给 `MaterialApp`
- **`title`**：应用标题
- **`theme`**：应用主题配置

## 导航行为对比

### 场景 1：路由 A 的详情页（`/a/details`）

- **使用的导航器**：Shell 导航器
- **视觉效果**：
  - 详情页覆盖 `ScreenA`
  - **底部导航栏仍然可见**
  - 用户可以通过返回按钮或手势返回到 `ScreenA`

### 场景 2：路由 B 的详情页（`/b/details`）

- **使用的导航器**：根导航器
- **视觉效果**：
  - 详情页覆盖 `ScreenB`**以及整个应用 Shell**
  - **底部导航栏被完全覆盖，不可见**
  - 用户可以通过返回按钮或手势返回到 `ScreenB`

### 场景 3：路由 C 的详情页（`/c/details`）

- **使用的导航器**：Shell 导航器
- **视觉效果**：与场景 1 相同

## 使用场景

### 何时使用 Shell 导航器（默认行为）

适用于需要在详情页中仍然保持底部导航栏可见的场景，例如：

- 商品详情页
- 文章详情页
- 用户资料页

### 何时使用根导航器（指定 `parentNavigatorKey`）

适用于需要全屏显示详情页的场景，例如：

- 全屏图片查看
- 视频播放页
- 登录/注册页面

## 关键要点总结

1. **`ShellRoute` 创建了一个嵌套的导航器**，用于管理持久性 UI 组件（如底部导航栏）和子页面

2. **`parentNavigatorKey` 参数**决定了子路由使用哪个导航器：
   - 不指定：使用 Shell 导航器（默认）
   - 指定为 `_rootNavigatorKey`：使用根导航器

3. **不同的导航器选择会产生不同的视觉效果**：
   - Shell 导航器：详情页不会覆盖 Shell UI
   - 根导航器：详情页会覆盖整个应用 Shell

4. **这种设计模式**允许开发者灵活控制页面层级和导航行为，满足不同业务场景的需求
