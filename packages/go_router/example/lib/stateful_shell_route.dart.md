# stateful_shell_route.dart 代码解析

## 概述

这个文件演示了如何使用 `StatefulShellRoute` 实现带有 `BottomNavigationBar` 的嵌套导航结构。核心特点是每个底部导航栏的标签页都拥有独立的 `Navigator`，能够保持各自的导航状态。这意味着用户在不同标签页之间切换时，每个标签页的导航栈会被独立维护，不会丢失之前的导航历史。

此外，这种配置还支持深度链接（deep linking），可以直接导航到嵌套在某个标签页内的子页面。

## 核心概念

### StatefulShellRoute

`StatefulShellRoute` 是 go_router 提供的一种特殊路由类型，用于创建有状态的 shell 路由。与普通的 `ShellRoute` 不同，`StatefulShellRoute` 能够为每个分支（branch）维护独立的导航状态。

### indexedStack 构造方法

示例中使用了 `StatefulShellRoute.indexedStack` 构造方法，这个方法提供了一个基于 `IndexedStack` 的默认实现，用于管理分支 `Navigator` 的容器。`IndexedStack` 会保持所有子组件的状态，但只显示当前选中的那个，这正是底部导航栏所需要的特性。

### StatefulNavigationShell

`StatefulNavigationShell` 是一个 Widget，负责管理嵌套导航的状态。它提供了 `goBranch` 方法来切换活动分支，并维护每个分支的导航栈状态。

## 代码结构解析

### 全局 Navigator Keys

```dart 8:12:example/lib/stateful_shell_route.dart
final GlobalKey<NavigatorState> _rootNavigatorKey = GlobalKey<NavigatorState>(
  debugLabel: 'root',
);
final GlobalKey<NavigatorState> _sectionANavigatorKey =
    GlobalKey<NavigatorState>(debugLabel: 'sectionANav');
```

这里定义了两个 `Navigator` 的全局键：

- `_rootNavigatorKey`：根导航器的键，用于整个应用的路由管理
- `_sectionANavigatorKey`：Section A 分支的导航器键，用于标识特定的分支导航器

为分支提供自定义的 `navigatorKey` 是可选的。如果不提供，系统会自动生成一个默认的键。但如果你需要从其他地方访问特定的分支导航器（例如，需要在某个页面中弹出到分支的根页面），提供自定义键会很有用。

### 主应用类：NestedTabNavigationExampleApp

```dart 23:137:example/lib/stateful_shell_route.dart
/// An example demonstrating how to use nested navigators
class NestedTabNavigationExampleApp extends StatelessWidget {
  /// Creates a NestedTabNavigationExampleApp
  NestedTabNavigationExampleApp({super.key});

  final GoRouter _router = GoRouter(
    navigatorKey: _rootNavigatorKey,
    initialLocation: '/a',
    routes: <RouteBase>[
      // #docregion configuration-builder
      StatefulShellRoute.indexedStack(
        builder:
            (
              BuildContext context,
              GoRouterState state,
              StatefulNavigationShell navigationShell,
            ) {
              // Return the widget that implements the custom shell (in this case
              // using a BottomNavigationBar). The StatefulNavigationShell is passed
              // to be able access the state of the shell and to navigate to other
              // branches in a stateful way.
              return ScaffoldWithNavBar(navigationShell: navigationShell);
            },
        // #enddocregion configuration-builder
        // #docregion configuration-branches
        branches: <StatefulShellBranch>[
          // The route branch for the first tab of the bottom navigation bar.
          StatefulShellBranch(
            navigatorKey: _sectionANavigatorKey,
            routes: <RouteBase>[
              GoRoute(
                // The screen to display as the root in the first tab of the
                // bottom navigation bar.
                path: '/a',
                builder: (BuildContext context, GoRouterState state) =>
                    const RootScreen(label: 'A', detailsPath: '/a/details'),
                routes: <RouteBase>[
                  // The details screen to display stacked on navigator of the
                  // first tab. This will cover screen A but not the application
                  // shell (bottom navigation bar).
                  GoRoute(
                    path: 'details',
                    builder: (BuildContext context, GoRouterState state) =>
                        const DetailsScreen(label: 'A'),
                  ),
                ],
              ),
            ],
            // To enable preloading of the initial locations of branches, pass
            // 'true' for the parameter `preload` (false is default).
          ),
          // #enddocregion configuration-branches

          // The route branch for the second tab of the bottom navigation bar.
          StatefulShellBranch(
            // It's not necessary to provide a navigatorKey if it isn't also
            // needed elsewhere. If not provided, a default key will be used.
            routes: <RouteBase>[
              GoRoute(
                // The screen to display as the root in the second tab of the
                // bottom navigation bar.
                path: '/b',
                builder: (BuildContext context, GoRouterState state) =>
                    const RootScreen(
                      label: 'B',
                      detailsPath: '/b/details/1',
                      secondDetailsPath: '/b/details/2',
                    ),
                routes: <RouteBase>[
                  GoRoute(
                    path: 'details/:param',
                    builder: (BuildContext context, GoRouterState state) =>
                        DetailsScreen(
                          label: 'B',
                          param: state.pathParameters['param'],
                        ),
                  ),
                ],
              ),
            ],
          ),

          // The route branch for the third tab of the bottom navigation bar.
          StatefulShellBranch(
            routes: <RouteBase>[
              GoRoute(
                // The screen to display as the root in the third tab of the
                // bottom navigation bar.
                path: '/c',
                builder: (BuildContext context, GoRouterState state) =>
                    const RootScreen(label: 'C', detailsPath: '/c/details'),
                routes: <RouteBase>[
                  GoRoute(
                    path: 'details',
                    builder: (BuildContext context, GoRouterState state) =>
                        DetailsScreen(label: 'C', extra: state.extra),
                  ),
                ],
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

这是整个应用的核心配置类。主要包含：

1. **GoRouter 配置**：
   - `navigatorKey`：使用根导航器键
   - `initialLocation: '/a'`：应用启动时初始显示的路径
   - `routes`：路由配置列表，只包含一个 `StatefulShellRoute.indexedStack`

2. **StatefulShellRoute.indexedStack 配置**：
   - `builder`：构建自定义 shell 的函数，接收 `StatefulNavigationShell` 参数
   - `branches`：定义三个 `StatefulShellBranch`，对应三个底部导航栏标签

3. **StatefulShellBranch 配置**：
   每个分支代表底部导航栏的一个标签页：
   - **分支 A**（`/a`）：根路径是 `/a`，有一个子路由 `/a/details`
   - **分支 B**（`/b`）：根路径是 `/b`，有一个带参数的子路由 `/b/details/:param`
   - **分支 C**（`/c`）：根路径是 `/c`，有一个子路由 `/c/details`

注意分支 B 的路由使用了路径参数（`:param`），分支 C 的详情页面使用了 `extra` 参数来传递额外数据。

### 自定义 Shell：ScaffoldWithNavBar

```dart 139:192:example/lib/stateful_shell_route.dart
/// Builds the "shell" for the app by building a Scaffold with a
/// BottomNavigationBar, where [child] is placed in the body of the Scaffold.
class ScaffoldWithNavBar extends StatelessWidget {
  /// Constructs an [ScaffoldWithNavBar].
  const ScaffoldWithNavBar({required this.navigationShell, Key? key})
    : super(key: key ?? const ValueKey<String>('ScaffoldWithNavBar'));

  /// The navigation shell and container for the branch Navigators.
  final StatefulNavigationShell navigationShell;

  // #docregion configuration-custom-shell
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      // The StatefulNavigationShell from the associated StatefulShellRoute is
      // directly passed as the body of the Scaffold.
      body: navigationShell,
      bottomNavigationBar: BottomNavigationBar(
        // Here, the items of BottomNavigationBar are hard coded. In a real
        // world scenario, the items would most likely be generated from the
        // branches of the shell route, which can be fetched using
        // `navigationShell.route.branches`.
        items: const <BottomNavigationBarItem>[
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Section A'),
          BottomNavigationBarItem(icon: Icon(Icons.work), label: 'Section B'),
          BottomNavigationBarItem(icon: Icon(Icons.tab), label: 'Section C'),
        ],
        currentIndex: navigationShell.currentIndex,
        // Navigate to the current location of the branch at the provided index
        // when tapping an item in the BottomNavigationBar.
        onTap: (int index) => navigationShell.goBranch(index),
      ),
    );
  }
  // #enddocregion configuration-custom-shell

  /// NOTE: For a slightly more sophisticated branch switching, change the onTap
  /// handler on the BottomNavigationBar above to the following:
  /// `onTap: (int index) => _onTap(context, index),`
  // ignore: unused_element
  void _onTap(BuildContext context, int index) {
    // When navigating to a new branch, it's recommended to use the goBranch
    // method, as doing so makes sure the last navigation state of the
    // Navigator for the branch is restored.
    navigationShell.goBranch(
      index,
      // A common pattern when using bottom navigation bars is to support
      // navigating to the initial location when tapping the item that is
      // already active. This example demonstrates how to support this behavior,
      // using the initialLocation parameter of goBranch.
      initialLocation: index == navigationShell.currentIndex,
    );
  }
}
```

这是实现自定义 shell 的 Widget，主要职责是：

1. **接收 StatefulNavigationShell**：这个 Widget 包含了所有分支的导航器，直接作为 `Scaffold` 的 body
2. **底部导航栏配置**：
   - `currentIndex`：使用 `navigationShell.currentIndex` 获取当前活动的分支索引
   - `onTap`：调用 `navigationShell.goBranch(index)` 切换到对应的分支

**简单切换 vs 高级切换**：

代码中提供了两种切换方式：

- **简单方式**（当前使用的）：`onTap: (int index) => navigationShell.goBranch(index)`
  - 直接切换到指定分支，如果分支之前访问过，会恢复到上次的导航状态

- **高级方式**（注释中的 `_onTap` 方法）：

  ```dart 179:191:example/lib/stateful_shell_route.dart
  void _onTap(BuildContext context, int index) {
    // When navigating to a new branch, it's recommended to use the goBranch
    // method, as doing so makes sure the last navigation state of the
    // Navigator for the branch is restored.
    navigationShell.goBranch(
      index,
      // A common pattern when using bottom navigation bars is to support
      // navigating to the initial location when tapping the item that is
      // already active. This example demonstrates how to support this behavior,
      // using the initialLocation parameter of goBranch.
      initialLocation: index == navigationShell.currentIndex,
    );
  }
  ```

  - 使用 `initialLocation` 参数实现：当点击已激活的标签时，导航到该分支的初始位置
  - 这是一种常见的 UX 模式，允许用户快速返回到标签页的根页面

**关于 BottomNavigationBar items**：

注释中提到，在实际项目中，`BottomNavigationBar` 的 items 应该从 `navigationShell.route.branches` 动态生成，而不是硬编码。这样可以保持路由配置和 UI 的一致性。

### 根屏幕组件：RootScreen

```dart 194:245:example/lib/stateful_shell_route.dart
/// Widget for the root/initial pages in the bottom navigation bar.
class RootScreen extends StatelessWidget {
  /// Creates a RootScreen
  const RootScreen({
    required this.label,
    required this.detailsPath,
    this.secondDetailsPath,
    super.key,
  });

  /// The label
  final String label;

  /// The path to the detail page
  final String detailsPath;

  /// The path to another detail page
  final String? secondDetailsPath;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Root of section $label')),
      body: Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: <Widget>[
            Text(
              'Screen $label',
              style: Theme.of(context).textTheme.titleLarge,
            ),
            const Padding(padding: EdgeInsets.all(4)),
            TextButton(
              onPressed: () {
                GoRouter.of(context).go(detailsPath, extra: '$label-XYZ');
              },
              child: const Text('View details'),
            ),
            const Padding(padding: EdgeInsets.all(4)),
            if (secondDetailsPath != null)
              TextButton(
                onPressed: () {
                  GoRouter.of(context).go(secondDetailsPath!);
                },
                child: const Text('View more details'),
              ),
          ],
        ),
      ),
    );
  }
}
```

这是每个标签页的根屏幕，显示：

1. **标签标识**：显示当前标签的名称（A、B 或 C）
2. **导航按钮**：
   - "View details" 按钮：导航到详情页面，并通过 `extra` 参数传递数据（`'$label-XYZ'`）
   - "View more details" 按钮（可选）：仅在提供了 `secondDetailsPath` 时显示（只有分支 B 使用）

注意这里使用了 `GoRouter.of(context).go()` 进行导航，这会导航到同一分支内的子路由。由于这是在分支的根页面，导航到详情页面会在同一个分支的 Navigator 上堆叠，所以底部导航栏会保持可见。

### 详情屏幕组件：DetailsScreen

```dart 247:339:example/lib/stateful_shell_route.dart
/// The details screen for either the A or B screen.
class DetailsScreen extends StatefulWidget {
  /// Constructs a [DetailsScreen].
  const DetailsScreen({
    required this.label,
    this.param,
    this.extra,
    this.withScaffold = true,
    super.key,
  });

  /// The label to display in the center of the screen.
  final String label;

  /// Optional param
  final String? param;

  /// Optional extra object
  final Object? extra;

  /// Wrap in scaffold
  final bool withScaffold;

  @override
  State<StatefulWidget> createState() => DetailsScreenState();
}

/// The state for DetailsScreen
class DetailsScreenState extends State<DetailsScreen> {
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    if (widget.withScaffold) {
      return Scaffold(
        appBar: AppBar(title: Text('Details Screen - ${widget.label}')),
        body: _build(context),
      );
    } else {
      return ColoredBox(
        color: Theme.of(context).scaffoldBackgroundColor,
        child: _build(context),
      );
    }
  }

  Widget _build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: <Widget>[
          Text(
            'Details for ${widget.label} - Counter: $_counter',
            style: Theme.of(context).textTheme.titleLarge,
          ),
          const Padding(padding: EdgeInsets.all(4)),
          TextButton(
            onPressed: () {
              setState(() {
                _counter++;
              });
            },
            child: const Text('Increment counter'),
          ),
          const Padding(padding: EdgeInsets.all(8)),
          if (widget.param != null)
            Text(
              'Parameter: ${widget.param!}',
              style: Theme.of(context).textTheme.titleMedium,
            ),
          const Padding(padding: EdgeInsets.all(8)),
          if (widget.extra != null)
            Text(
              'Extra: ${widget.extra!}',
              style: Theme.of(context).textTheme.titleMedium,
            ),
          if (!widget.withScaffold) ...<Widget>[
            const Padding(padding: EdgeInsets.all(16)),
            TextButton(
              onPressed: () {
                GoRouter.of(context).pop();
              },
              child: const Text(
                '< Back',
                style: TextStyle(fontWeight: FontWeight.bold, fontSize: 18),
              ),
            ),
          ],
        ],
      ),
    );
  }
}
```

这是一个状态化的详情屏幕组件，展示了多种使用模式：

1. **参数接收方式**：
   - `param`：路径参数（用于分支 B 的 `/b/details/:param`）
   - `extra`：额外对象（用于分支 C，传递任意数据）

2. **状态管理**：使用 `StatefulWidget` 维护一个计数器，演示在标签切换时状态如何保持

3. **灵活的布局**：
   - `withScaffold` 参数控制是否包装 `Scaffold`
   - 当 `withScaffold` 为 `false` 时，显示自定义的返回按钮

4. **状态保持**：由于使用了 `StatefulShellRoute`，当用户在不同标签页之间切换时，每个详情页面的计数器状态都会被保留

## 关键实现细节

### 导航状态保持机制

`StatefulShellRoute.indexedStack` 使用 `IndexedStack` 来管理分支 Navigator。`IndexedStack` 的关键特性是：

- 它维护所有子 Widget 的状态，即使子 Widget 当前不可见
- 只渲染当前索引对应的子 Widget，其他保持不可见但仍保持状态
- 这确保了每个分支的导航栈在切换时不会丢失

### 深度链接支持

由于使用了标准的路由配置，这个示例天然支持深度链接。例如：

- `/a`：直接导航到标签 A 的根页面
- `/a/details`：直接导航到标签 A 的详情页面
- `/b/details/123`：直接导航到标签 B 的详情页面，并传递参数 "123"
- `/c/details`：直接导航到标签 C 的详情页面

### 分支间的导航隔离

每个分支拥有独立的 `Navigator`，这意味着：

- 分支内的导航（如从根页面到详情页面）不会影响其他分支
- 使用 `GoRouter.of(context).go()` 在分支内导航时，导航发生在该分支的 Navigator 上
- 如果需要跨分支导航或弹出到根导航器，需要使用相应的 API（如 `context.go()` 配合完整路径，或使用 `context.push()` 在根导航器上推入）

## 使用场景

这种模式特别适合以下场景：

1. **底部导航栏应用**：需要多个主要功能模块，每个模块有独立的导航栈
2. **标签页导航**：类似移动应用中的标签页设计
3. **需要保持状态的嵌套导航**：用户在不同模块间切换时，希望每个模块的导航状态被保留
4. **复杂的导航结构**：需要在应用的不同部分维护独立的导航历史

## 与 ShellRoute 的区别

- **ShellRoute**：每次切换分支时，导航状态会重置，不适合需要保持状态的场景
- **StatefulShellRoute**：每个分支维护独立的导航栈，切换时状态得以保留

## 总结

这个示例展示了如何使用 go_router 的 `StatefulShellRoute` 创建一个功能完整的底部导航栏应用。通过为每个标签页提供独立的 Navigator，实现了导航状态的独立维护，同时保持了代码的清晰和可维护性。这种模式是现代移动应用开发中的常见需求，go_router 提供了简洁而强大的 API 来支持这种场景。
