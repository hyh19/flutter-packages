# ShellRoute 示例代码解析

## 概述

这个示例文件展示了如何使用 `ShellRoute` 在 Flutter 应用中实现嵌套导航模式。`ShellRoute` 允许在应用的小部件树中放置一个额外的 `Navigator`，用于替代根导航器。这种模式特别适合需要同时显示页面和其他 UI 组件（如底部导航栏）的场景，同时支持深度链接。

## 核心概念

### Navigator Keys

代码中定义了两个 `Navigator` 的全局键：

```dart 8:13:example/lib/shell_route.dart
final GlobalKey<NavigatorState> _rootNavigatorKey = GlobalKey<NavigatorState>(
  debugLabel: 'root',
);
final GlobalKey<NavigatorState> _shellNavigatorKey = GlobalKey<NavigatorState>(
  debugLabel: 'shell',
);
```

- **`_rootNavigatorKey`**：根导航器的键，用于整个应用的最顶层导航
- **`_shellNavigatorKey`**：Shell 导航器的键，用于 `ShellRoute` 内部的嵌套导航

这两个导航器形成了两层导航结构，允许在不同层级进行页面切换。

### ShellRoute 的作用

`ShellRoute` 是 go_router 提供的一种特殊路由类型，它允许：

1. 创建一个持久的 UI 外壳（如底部导航栏）
2. 在保持外壳可见的同时切换内部页面
3. 支持深度链接直接跳转到嵌套页面
4. 通过 `parentNavigatorKey` 控制页面在哪个导航器上显示

## 应用结构

### 主应用类

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

**关键配置说明**：

- **`navigatorKey: _rootNavigatorKey`**：GoRouter 使用根导航器作为顶层导航
- **`initialLocation: '/a'`**：应用启动时默认显示 Screen A
- **`ShellRoute`**：创建应用外壳，包含底部导航栏
- **`navigatorKey: _shellNavigatorKey`**：ShellRoute 使用独立的导航器
- **`builder`**：接收 `child` 参数，这是当前匹配的路由页面

### 路由结构

应用包含三个主要路由：

1. **`/a`**：第一个底部导航项，包含子路由 `/a/details`
2. **`/b`**：第二个底部导航项，包含子路由 `/b/details`（使用根导航器）
3. **`/c`**：第三个底部导航项，包含子路由 `/c/details`

## 关键组件

### ScaffoldWithNavBar

```dart 116:172:example/lib/shell_route.dart
/// Builds the "shell" for the app by building a Scaffold with a
/// BottomNavigationBar, where [child] is placed in the body of the Scaffold.
class ScaffoldWithNavBar extends StatelessWidget {
  /// Constructs an [ScaffoldWithNavBar].
  const ScaffoldWithNavBar({required this.child, super.key});

  /// The widget to display in the body of the Scaffold.
  /// In this sample, it is a Navigator.
  final Widget child;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: child,
      bottomNavigationBar: BottomNavigationBar(
        items: const <BottomNavigationBarItem>[
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'A Screen'),
          BottomNavigationBarItem(
            icon: Icon(Icons.business),
            label: 'B Screen',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.notification_important_rounded),
            label: 'C Screen',
          ),
        ],
        currentIndex: _calculateSelectedIndex(context),
        onTap: (int idx) => _onItemTapped(idx, context),
      ),
    );
  }

  static int _calculateSelectedIndex(BuildContext context) {
    final String location = GoRouterState.of(context).uri.path;
    if (location.startsWith('/a')) {
      return 0;
    }
    if (location.startsWith('/b')) {
      return 1;
    }
    if (location.startsWith('/c')) {
      return 2;
    }
    return 0;
  }

  void _onItemTapped(int index, BuildContext context) {
    switch (index) {
      case 0:
        GoRouter.of(context).go('/a');
      case 1:
        GoRouter.of(context).go('/b');
      case 2:
        GoRouter.of(context).go('/c');
    }
  }
}
```

这个组件是应用的外壳，负责：

1. **显示底部导航栏**：提供三个导航选项
2. **包裹子页面**：通过 `child` 参数接收并显示当前路由页面
3. **同步导航状态**：根据当前路径计算并高亮对应的导航项
4. **处理导航点击**：点击底部导航栏时切换到对应路由

**关键方法**：

- **`_calculateSelectedIndex`**：根据当前路径（`GoRouterState.of(context).uri.path`）计算应该高亮哪个导航项
- **`_onItemTapped`**：处理底部导航栏点击事件，使用 `GoRouter.of(context).go()` 进行导航

### 屏幕组件

应用包含三个主屏幕和一个详情屏幕：

#### ScreenA

```dart 174:199:example/lib/shell_route.dart
/// The first screen in the bottom navigation bar.
class ScreenA extends StatelessWidget {
  /// Constructs a [ScreenA] widget.
  const ScreenA({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(),
      body: Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: <Widget>[
            const Text('Screen A'),
            TextButton(
              onPressed: () {
                GoRouter.of(context).go('/a/details');
              },
              child: const Text('View A details'),
            ),
          ],
        ),
      ),
    );
  }
}
```

#### ScreenB

```dart 201:226:example/lib/shell_route.dart
/// The second screen in the bottom navigation bar.
class ScreenB extends StatelessWidget {
  /// Constructs a [ScreenB] widget.
  const ScreenB({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(),
      body: Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: <Widget>[
            const Text('Screen B'),
            TextButton(
              onPressed: () {
                GoRouter.of(context).go('/b/details');
              },
              child: const Text('View B details'),
            ),
          ],
        ),
      ),
    );
  }
}
```

#### ScreenC

```dart 228:253:example/lib/shell_route.dart
/// The third screen in the bottom navigation bar.
class ScreenC extends StatelessWidget {
  /// Constructs a [ScreenC] widget.
  const ScreenC({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(),
      body: Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: <Widget>[
            const Text('Screen C'),
            TextButton(
              onPressed: () {
                GoRouter.of(context).go('/c/details');
              },
              child: const Text('View C details'),
            ),
          ],
        ),
      ),
    );
  }
}
```

这三个屏幕结构相同，都包含一个按钮用于跳转到对应的详情页面。

#### DetailsScreen

```dart 255:275:example/lib/shell_route.dart
/// The details screen for either the A, B or C screen.
class DetailsScreen extends StatelessWidget {
  /// Constructs a [DetailsScreen].
  const DetailsScreen({required this.label, super.key});

  /// The label to display in the center of the screen.
  final String label;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Details Screen')),
      body: Center(
        child: Text(
          'Details for $label',
          style: Theme.of(context).textTheme.headlineMedium,
        ),
      ),
    );
  }
}
```

详情屏幕接收一个 `label` 参数，用于显示不同屏幕的详情信息。

## 导航行为差异

这个示例的关键在于展示了两种不同的导航行为：

### 使用 Shell Navigator（默认行为）

对于 `/a/details` 和 `/c/details` 路由：

```dart 54:59:example/lib/shell_route.dart
              GoRoute(
                path: 'details',
                builder: (BuildContext context, GoRouterState state) {
                  return const DetailsScreen(label: 'A');
                },
              ),
```

这些路由**没有指定 `parentNavigatorKey`**，因此使用默认的 Shell Navigator。这意味着：

- 详情页面会覆盖对应的主屏幕（Screen A 或 Screen C）
- **底部导航栏仍然可见**（因为它在 Shell 层级）
- 用户可以通过返回按钮返回到主屏幕

### 使用 Root Navigator

对于 `/b/details` 路由：

```dart 74:80:example/lib/shell_route.dart
              GoRoute(
                path: 'details',
                parentNavigatorKey: _rootNavigatorKey,
                builder: (BuildContext context, GoRouterState state) {
                  return const DetailsScreen(label: 'B');
                },
              ),
```

这个路由**明确指定了 `parentNavigatorKey: _rootNavigatorKey`**，因此使用根导航器。这意味着：

- 详情页面会覆盖整个应用，包括底部导航栏
- **底部导航栏被隐藏**（因为整个 Shell 都被覆盖）
- 这适合需要全屏显示的页面（如登录页、全屏对话框等）

## 导航流程示例

### 场景 1：从 Screen A 跳转到详情

1. 用户在 Screen A 点击 "View A details" 按钮
2. 调用 `GoRouter.of(context).go('/a/details')`
3. 由于 `/a/details` 使用 Shell Navigator，详情页面覆盖 Screen A
4. 底部导航栏仍然可见
5. 用户点击返回按钮，返回到 Screen A

### 场景 2：从 Screen B 跳转到详情

1. 用户在 Screen B 点击 "View B details" 按钮
2. 调用 `GoRouter.of(context).go('/b/details')`
3. 由于 `/b/details` 使用 Root Navigator，详情页面覆盖整个应用
4. 底部导航栏被隐藏
5. 用户点击返回按钮，返回到 Screen B（底部导航栏重新出现）

## 使用场景

`ShellRoute` 特别适合以下场景：

1. **底部导航栏应用**：需要在切换页面时保持底部导航栏可见
2. **侧边栏导航**：需要在切换内容时保持侧边栏可见
3. **持久化 UI 元素**：需要在页面切换时保持某些 UI 组件（如应用栏、抽屉菜单等）
4. **深度链接**：需要支持直接跳转到嵌套页面，同时保持外壳 UI

## 总结

这个示例展示了 `ShellRoute` 的核心用法：

- **嵌套导航**：通过 Shell Navigator 实现页面切换而不影响外壳 UI
- **灵活控制**：通过 `parentNavigatorKey` 可以选择在哪个导航器上显示页面
- **深度链接支持**：可以直接跳转到嵌套路由，同时保持外壳可见
- **状态同步**：底部导航栏能够根据当前路径自动高亮对应项

理解这些概念对于构建具有复杂导航结构的 Flutter 应用非常重要。
