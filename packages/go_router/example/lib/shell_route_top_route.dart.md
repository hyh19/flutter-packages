# ShellRoute TopRoute 示例代码解析

## 概述

这个示例演示了如何在 Flutter 应用中使用 `go_router` 包的 `ShellRoute` 实现嵌套导航，并利用 `topRoute` 属性在应用外壳（Shell）中根据不同的路由动态设置 AppBar 标题。这是一个典型的底部导航栏（BottomNavigationBar）应用场景，其中每个标签页都有对应的详情页面。

## 核心概念

### ShellRoute 的作用

`ShellRoute` 允许在 widget 树中放置一个额外的 `Navigator`，用于替代根导航器。这使得深度链接（deep-links）能够同时显示页面和其他 UI 组件（如底部导航栏），而不会在导航时丢失这些组件。

### topRoute 的使用

`topRoute` 是 `GoRouterState` 的一个属性，用于获取当前路由栈中最顶层的路由信息。在这个示例中，它被用来根据当前路由名称动态设置 AppBar 的标题。

## 代码结构分析

### 1. 导航器键（Navigator Keys）

```dart 8:13:example/lib/shell_route_top_route.dart
final GlobalKey<NavigatorState> _rootNavigatorKey = GlobalKey<NavigatorState>(
  debugLabel: 'root',
);
final GlobalKey<NavigatorState> _shellNavigatorKey = GlobalKey<NavigatorState>(
  debugLabel: 'shell',
);
```

代码定义了两个全局导航器键：

- **`_rootNavigatorKey`**：用于根导航器，管理整个应用的导航栈
- **`_shellNavigatorKey`**：用于 Shell 导航器，管理 ShellRoute 内部的导航栈

使用不同的导航器键可以实现嵌套导航，使得详情页面可以覆盖主屏幕，但不会影响应用外壳（如底部导航栏）。

### 2. 应用入口和路由配置

```dart 27:134:example/lib/shell_route_top_route.dart
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
          final String? routeName = GoRouterState.of(context).topRoute?.name;
          // This title could also be created using a route's path parameters in GoRouterState
          final String title = switch (routeName) {
            'a' => 'A Screen',
            'a.details' => 'A Details',
            'b' => 'B Screen',
            'b.details' => 'B Details',
            'c' => 'C Screen',
            'c.details' => 'C Details',
            _ => 'Unknown',
          };
          return ScaffoldWithNavBar(title: title, child: child);
        },
        routes: <RouteBase>[
          /// The first screen to display in the bottom navigation bar.
          GoRoute(
            // The name of this route used to determine the title in the ShellRoute.
            name: 'a',
            path: '/a',
            builder: (BuildContext context, GoRouterState state) {
              return const ScreenA();
            },
            routes: <RouteBase>[
              // The details screen to display stacked on the inner Navigator.
              // This will cover screen A but not the application shell.
              GoRoute(
                // The name of this route used to determine the title in the ShellRoute.
                name: 'a.details',
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
            // The name of this route used to determine the title in the ShellRoute.
            name: 'b',
            path: '/b',
            builder: (BuildContext context, GoRouterState state) {
              return const ScreenB();
            },
            routes: <RouteBase>[
              // The details screen to display stacked on the inner Navigator.
              // This will cover screen B but not the application shell.
              GoRoute(
                // The name of this route used to determine the title in the ShellRoute.
                name: 'b.details',
                path: 'details',
                builder: (BuildContext context, GoRouterState state) {
                  return const DetailsScreen(label: 'B');
                },
              ),
            ],
          ),

          /// The third screen to display in the bottom navigation bar.
          GoRoute(
            // The name of this route used to determine the title in the ShellRoute.
            name: 'c',
            path: '/c',
            builder: (BuildContext context, GoRouterState state) {
              return const ScreenC();
            },
            routes: <RouteBase>[
              // The details screen to display stacked on the inner Navigator.
              // This will cover screen C but not the application shell.
              GoRoute(
                // The name of this route used to determine the title in the ShellRoute.
                name: 'c.details',
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

#### 关键点解析

**路由配置结构**：

1. **根路由**：使用 `ShellRoute` 作为应用的根路由，它包装了整个应用的导航结构
2. **ShellRoute 的 builder**：
   - 接收三个参数：`BuildContext`、`GoRouterState` 和 `Widget child`
   - `child` 是当前匹配的路由对应的 widget
   - 通过 `GoRouterState.of(context).topRoute?.name` 获取当前顶层路由的名称

**动态标题设置**：

```dart 41:51:example/lib/shell_route_top_route.dart
final String? routeName = GoRouterState.of(context).topRoute?.name;
// This title could also be created using a route's path parameters in GoRouterState
final String title = switch (routeName) {
  'a' => 'A Screen',
  'a.details' => 'A Details',
  'b' => 'B Screen',
  'b.details' => 'B Details',
  'c' => 'C Screen',
  'c.details' => 'C Details',
  _ => 'Unknown',
};
```

使用 `switch` 表达式根据路由名称（`routeName`）动态设置标题。每个路由都有唯一的名称（如 `'a'`、`'a.details'`），这使得可以根据当前路由显示不同的标题。

**嵌套路由结构**：

- 每个主路由（`/a`、`/b`、`/c`）都有一个子路由 `details`
- 子路由使用相对路径 `'details'`，完整路径为 `/a/details`、`/b/details`、`/c/details`
- 子路由的名称使用点号分隔（如 `'a.details'`），表示层级关系

### 3. 应用外壳组件（ScaffoldWithNavBar）

```dart 136:216:example/lib/shell_route_top_route.dart
/// Builds the "shell" for the app by building a Scaffold with a
/// BottomNavigationBar, where [child] is placed in the body of the Scaffold.
class ScaffoldWithNavBar extends StatelessWidget {
  /// Constructs an [ScaffoldWithNavBar].
  const ScaffoldWithNavBar({
    super.key,
    required this.title,
    required this.child,
  });

  /// The title to display in the AppBar.
  final String title;

  /// The widget to display in the body of the Scaffold.
  /// In this sample, it is a Navigator.
  final Widget child;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: child,
      appBar: AppBar(title: Text(title), leading: _buildLeadingButton(context)),
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

  /// Builds the app bar leading button using the current location [Uri].
  ///
  /// The [Scaffold]'s default back button cannot be used because it doesn't
  /// have the context of the current child.
  Widget? _buildLeadingButton(BuildContext context) {
    final RouteMatchList currentConfiguration = GoRouter.of(
      context,
    ).routerDelegate.currentConfiguration;
    final RouteMatch lastMatch = currentConfiguration.last;
    final Uri location = lastMatch is ImperativeRouteMatch
        ? lastMatch.matches.uri
        : currentConfiguration.uri;
    final bool canPop = location.pathSegments.length > 1;
    return canPop ? BackButton(onPressed: GoRouter.of(context).pop) : null;
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

#### 组件功能说明

**Scaffold 结构**：

- **`body: child`**：将 ShellRoute 传入的 `child`（当前路由对应的页面）放置在 Scaffold 的 body 中
- **`appBar`**：显示动态标题，并根据导航栈状态显示返回按钮
- **`bottomNavigationBar`**：底部导航栏，包含三个标签页

**返回按钮逻辑**：

```dart 180:190:example/lib/shell_route_top_route.dart
Widget? _buildLeadingButton(BuildContext context) {
  final RouteMatchList currentConfiguration = GoRouter.of(
    context,
  ).routerDelegate.currentConfiguration;
  final RouteMatch lastMatch = currentConfiguration.last;
  final Uri location = lastMatch is ImperativeRouteMatch
      ? lastMatch.matches.uri
      : currentConfiguration.uri;
  final bool canPop = location.pathSegments.length > 1;
  return canPop ? BackButton(onPressed: GoRouter.of(context).pop) : null;
}
```

这个方法实现了自定义返回按钮：

1. 获取当前路由配置和最后一个匹配的路由
2. 根据路由类型（`ImperativeRouteMatch` 或普通路由）获取正确的 URI
3. 判断路径段数量：如果大于 1（如 `/a/details`），说明可以返回，显示返回按钮；否则不显示

**底部导航栏索引计算**：

```dart 192:204:example/lib/shell_route_top_route.dart
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
```

根据当前路径计算底部导航栏的选中索引。注意这里使用 `startsWith`，所以 `/a/details` 也会返回索引 0，保持底部导航栏的正确高亮状态。

**导航栏点击处理**：

```dart 206:215:example/lib/shell_route_top_route.dart
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
```

当用户点击底部导航栏时，使用 `go()` 方法导航到对应的主路由。注意这里使用 `go()` 而不是 `push()`，因为底部导航栏的切换应该是替换当前路由，而不是在栈中叠加。

### 4. 主屏幕组件

代码包含三个主屏幕组件：`ScreenA`、`ScreenB` 和 `ScreenC`。它们的结构完全相同，只是导航路径不同。

```dart 218:236:example/lib/shell_route_top_route.dart
/// The first screen in the bottom navigation bar.
class ScreenA extends StatelessWidget {
  /// Constructs a [ScreenA] widget.
  const ScreenA({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: TextButton(
          onPressed: () {
            GoRouter.of(context).go('/a/details');
          },
          child: const Text('View A details'),
        ),
      ),
    );
  }
}
```

每个主屏幕都包含一个按钮，点击后导航到对应的详情页面。注意这里也使用 `go()` 方法，因为详情页面是主屏幕的子路由，会在同一个导航栈中叠加。

### 5. 详情页面组件

```dart 278:297:example/lib/shell_route_top_route.dart
/// The details screen for either the A, B or C screen.
class DetailsScreen extends StatelessWidget {
  /// Constructs a [DetailsScreen].
  const DetailsScreen({required this.label, super.key});

  /// The label to display in the center of the screen.
  final String label;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
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

详情页面是一个简单的展示组件，接收一个 `label` 参数来显示不同的内容。当从不同主屏幕导航到详情页面时，会传入不同的标签（'A'、'B' 或 'C'）。

## 导航流程示例

### 场景 1：从主屏幕到详情页面

1. 用户在主屏幕 A（`/a`）点击 "View A details" 按钮
2. 调用 `GoRouter.of(context).go('/a/details')`
3. 路由系统匹配到 `'a.details'` 路由
4. `topRoute` 变为 `'a.details'`
5. AppBar 标题更新为 "A Details"
6. 详情页面显示在主屏幕 A 之上，但底部导航栏仍然可见

### 场景 2：切换底部导航栏标签

1. 用户在屏幕 A 的详情页面（`/a/details`）
2. 点击底部导航栏的第二个标签（B Screen）
3. 调用 `GoRouter.of(context).go('/b')`
4. 路由系统导航到屏幕 B
5. `topRoute` 变为 `'b'`
6. AppBar 标题更新为 "B Screen"
7. 详情页面被移除，显示屏幕 B

### 场景 3：返回操作

1. 用户在详情页面（`/a/details`）
2. 点击 AppBar 的返回按钮
3. 调用 `GoRouter.of(context).pop()`
4. 返回到主屏幕 A（`/a`）
5. `topRoute` 变为 `'a'`
6. AppBar 标题更新为 "A Screen"

## 技术要点总结

### 1. 嵌套导航的优势

- **保持 UI 组件**：详情页面不会覆盖底部导航栏
- **深度链接支持**：可以直接链接到详情页面，同时保持应用外壳
- **导航栈隔离**：Shell 导航器和根导航器各自管理自己的栈

### 2. topRoute 的使用场景

- **动态 UI 更新**：根据当前路由更新 AppBar、侧边栏等组件
- **路由名称匹配**：使用路由名称而非路径，更加语义化和可维护
- **层级感知**：可以区分主路由和子路由，实现不同的 UI 表现

### 3. 路由命名规范

- 使用点号分隔的命名方式（如 `'a.details'`）表示路由层级关系
- 命名应该清晰、语义化，便于维护和理解
- 可以通过路由名称而非路径来判断当前路由状态

### 4. 导航方法的选择

- **`go()`**：用于替换当前路由或导航到新路由，适合底部导航栏切换和主屏幕到详情页面的导航
- **`pop()`**：用于返回上一级路由，适合详情页面返回主屏幕

## 扩展建议

### 1. 使用路径参数

注释中提到可以使用路径参数来创建标题：

```dart
// This title could also be created using a route's path parameters in GoRouterState
```

例如，可以定义路由为 `/user/:id`，然后通过 `state.pathParameters['id']` 获取用户 ID 并动态设置标题。

### 2. 添加路由守卫

可以在 `ShellRoute` 的 `builder` 中添加认证检查、权限验证等逻辑，实现路由守卫功能。

### 3. 优化返回按钮逻辑

当前的返回按钮逻辑可以进一步优化，例如：

- 检查是否可以真正返回（使用 `GoRouter.of(context).canPop()`）
- 处理特殊情况（如首次加载、深层嵌套等）

### 4. 添加过渡动画

可以为不同路由之间的切换添加自定义过渡动画，提升用户体验。

## 总结

这个示例展示了 `go_router` 中 `ShellRoute` 和 `topRoute` 的强大功能，实现了一个典型的底部导航栏应用场景。通过嵌套导航和动态标题设置，创建了一个既美观又功能完整的导航结构。理解这个示例有助于在实际项目中实现类似的导航模式。
