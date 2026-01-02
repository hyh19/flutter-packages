# custom_stateful_shell_route.dart 代码解析

## 概述

这个示例演示了如何使用 `StatefulShellRoute` 创建嵌套导航结构，其中每个底部导航栏项使用独立的持久化导航器。该示例还展示了如何为分支导航器构建自定义容器（如 `TabBarView`），实现复杂的导航层次结构。

## 核心概念

### StatefulShellRoute

`StatefulShellRoute` 允许创建具有多个分支的路由，每个分支维护自己的导航状态。当切换分支时，之前分支的导航状态会被保留。

### 导航层次结构

这个示例实现了三层导航结构：

1. **顶层**：底部导航栏（`BottomNavigationBar`），包含两个主要部分（Section A 和 Section B）
2. **中间层**：Section B 内部使用顶部标签栏（`TabBar`），包含两个标签（B1 和 B2）
3. **底层**：每个标签内部可以有自己的子路由（如详情页）

## 全局导航器键

```dart 10:24:example/lib/others/custom_stateful_shell_route.dart
final GlobalKey<NavigatorState> _rootNavigatorKey = GlobalKey<NavigatorState>(
  debugLabel: 'root',
);
final GlobalKey<NavigatorState> _tabANavigatorKey = GlobalKey<NavigatorState>(
  debugLabel: 'tabANav',
);
final GlobalKey<NavigatorState> _tabBNavigatorKey = GlobalKey<NavigatorState>(
  debugLabel: 'tabBNav',
);
final GlobalKey<NavigatorState> _tabB1NavigatorKey = GlobalKey<NavigatorState>(
  debugLabel: 'tabB1Nav',
);
final GlobalKey<NavigatorState> _tabB2NavigatorKey = GlobalKey<NavigatorState>(
  debugLabel: 'tabB2Nav',
);
```

为每个导航层级创建独立的 `GlobalKey<NavigatorState>`，用于管理各自的导航栈。

## 路由配置

### 顶层 StatefulShellRoute

```dart 52:84:example/lib/others/custom_stateful_shell_route.dart
      StatefulShellRoute(
        builder:
            (
              BuildContext context,
              GoRouterState state,
              StatefulNavigationShell navigationShell,
            ) {
              // This nested StatefulShellRoute demonstrates the use of a
              // custom container for the branch Navigators. In this implementation,
              // no customization is done in the builder function (navigationShell
              // itself is simply used as the Widget for the route). Instead, the
              // navigatorContainerBuilder function below is provided to
              // customize the container for the branch Navigators.
              return navigationShell;
            },
        navigatorContainerBuilder:
            (
              BuildContext context,
              StatefulNavigationShell navigationShell,
              List<Widget> children,
            ) {
              // Returning a customized container for the branch
              // Navigators (i.e. the `List<Widget> children` argument).
              //
              // See ScaffoldWithNavBar for more details on how the children
              // are managed (using AnimatedBranchContainer).
              return ScaffoldWithNavBar(
                navigationShell: navigationShell,
                children: children,
              );
              // NOTE: To use a Cupertino version of ScaffoldWithNavBar, replace
              // ScaffoldWithNavBar above with CupertinoScaffoldWithNavBar.
            },
```

关键点：

1. **builder**：直接返回 `navigationShell`，不做额外定制
2. **navigatorContainerBuilder**：自定义容器，用于包装分支导航器，这里使用 `ScaffoldWithNavBar` 来添加底部导航栏

### Section A 分支

```dart 85:108:example/lib/others/custom_stateful_shell_route.dart
          // The route branch for the first tab of the bottom navigation bar.
          StatefulShellBranch(
            navigatorKey: _tabANavigatorKey,
            routes: <RouteBase>[
              GoRoute(
                // The screen to display as the root in the first tab of the
                // bottom navigation bar.
                path: '/a',
                builder: (BuildContext context, GoRouterState state) =>
                    const RootScreenA(),
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
          ),
```

Section A 包含：

- 根路由 `/a`：显示 `RootScreenA`
- 子路由 `details`：显示详情页，会覆盖 Section A 的内容，但不会覆盖底部导航栏

### Section B 分支（嵌套 StatefulShellRoute）

```dart 110:209:example/lib/others/custom_stateful_shell_route.dart
          // The route branch for the second tab of the bottom navigation bar.
          StatefulShellBranch(
            navigatorKey: _tabBNavigatorKey,
            // To enable preloading of the initial locations of branches, pass
            // `true` for the parameter `preload` (`false` is default).
            preload: true,
            // StatefulShellBranch will automatically use the first descendant
            // GoRoute as the initial location of the branch. If another route
            // is desired, specify the location of it using the defaultLocation
            // parameter.
            // defaultLocation: '/b1',
            routes: <RouteBase>[
              StatefulShellRoute(
                builder:
                    (
                      BuildContext context,
                      GoRouterState state,
                      StatefulNavigationShell navigationShell,
                    ) {
                      // Just like with the top level StatefulShellRoute, no
                      // customization is done in the builder function.
                      return navigationShell;
                    },
                navigatorContainerBuilder:
                    (
                      BuildContext context,
                      StatefulNavigationShell navigationShell,
                      List<Widget> children,
                    ) {
                      // Returning a customized container for the branch
                      // Navigators (i.e. the `List<Widget> children` argument).
                      //
                      // See TabbedRootScreen for more details on how the children
                      // are managed (in a TabBarView).
                      return TabbedRootScreen(
                        navigationShell: navigationShell,
                        key: tabbedRootScreenKey,
                        children: children,
                      );
                      // NOTE: To use a PageView version of TabbedRootScreen,
                      // replace TabbedRootScreen above with PagedRootScreen.
                    },
                // This bottom tab uses a nested shell, wrapping sub routes in a
                // top TabBar.
                branches: <StatefulShellBranch>[
                  StatefulShellBranch(
                    navigatorKey: _tabB1NavigatorKey,
                    routes: <GoRoute>[
                      GoRoute(
                        path: '/b1',
                        builder: (BuildContext context, GoRouterState state) =>
                            const TabScreen(
                              label: 'B1',
                              detailsPath: '/b1/details',
                            ),
                        routes: <RouteBase>[
                          GoRoute(
                            path: 'details',
                            builder:
                                (BuildContext context, GoRouterState state) =>
                                    const DetailsScreen(
                                      label: 'B1',
                                      withScaffold: false,
                                    ),
                          ),
                        ],
                      ),
                    ],
                  ),
                  StatefulShellBranch(
                    navigatorKey: _tabB2NavigatorKey,
                    // To enable preloading for all nested branches, set
                    // `preload` to `true` (`false` is default).
                    preload: true,
                    routes: <GoRoute>[
                      GoRoute(
                        path: '/b2',
                        builder: (BuildContext context, GoRouterState state) =>
                            const TabScreen(
                              label: 'B2',
                              detailsPath: '/b2/details',
                            ),
                        routes: <RouteBase>[
                          GoRoute(
                            path: 'details',
                            builder:
                                (BuildContext context, GoRouterState state) =>
                                    const DetailsScreen(
                                      label: 'B2',
                                      withScaffold: false,
                                    ),
                          ),
                        ],
                      ),
                    ],
                  ),
                ],
              ),
            ],
          ),
```

Section B 的特点：

1. **嵌套 StatefulShellRoute**：在 Section B 内部又创建了一个 `StatefulShellRoute`，用于实现顶部标签栏
2. **preload 参数**：设置为 `true` 时，会在应用启动时预加载分支的初始位置
3. **两个子分支**：B1 和 B2，每个都有自己的导航器和路由

## 自定义容器组件

### ScaffoldWithNavBar

```dart 225:279:example/lib/others/custom_stateful_shell_route.dart
/// Builds the "shell" for the app by building a Scaffold with a
/// BottomNavigationBar, where [child] is placed in the body of the Scaffold.
class ScaffoldWithNavBar extends StatelessWidget {
  /// Constructs an [ScaffoldWithNavBar].
  const ScaffoldWithNavBar({
    required this.navigationShell,
    required this.children,
    Key? key,
  }) : super(key: key ?? const ValueKey<String>('ScaffoldWithNavBar'));

  /// The navigation shell and container for the branch Navigators.
  final StatefulNavigationShell navigationShell;

  /// The children (branch Navigators) to display in a custom container
  /// ([AnimatedBranchContainer]).
  final List<Widget> children;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: AnimatedBranchContainer(
        currentIndex: navigationShell.currentIndex,
        children: children,
      ),
      bottomNavigationBar: BottomNavigationBar(
        // Here, the items of BottomNavigationBar are hard coded. In a real
        // world scenario, the items would most likely be generated from the
        // branches of the shell route, which can be fetched using
        // `navigationShell.route.branches`.
        items: const <BottomNavigationBarItem>[
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Section A'),
          BottomNavigationBarItem(icon: Icon(Icons.work), label: 'Section B'),
        ],
        currentIndex: navigationShell.currentIndex,
        onTap: (int index) => _onTap(context, index),
      ),
    );
  }

  /// Navigate to the current location of the branch at the provided index when
  /// tapping an item in the BottomNavigationBar.
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

功能说明：

1. **底部导航栏**：使用 `BottomNavigationBar` 显示两个主要部分
2. **AnimatedBranchContainer**：使用自定义容器来管理分支导航器的显示和动画
3. **goBranch 方法**：点击导航栏项时，使用 `goBranch` 切换分支，这会恢复该分支的最后导航状态
4. **initialLocation 参数**：当点击当前激活的项时，导航到该分支的初始位置

### AnimatedBranchContainer

```dart 346:383:example/lib/others/custom_stateful_shell_route.dart
/// Custom branch Navigator container that provides animated transitions
/// when switching branches.
class AnimatedBranchContainer extends StatelessWidget {
  /// Creates a AnimatedBranchContainer
  const AnimatedBranchContainer({
    super.key,
    required this.currentIndex,
    required this.children,
  });

  /// The index (in [children]) of the branch Navigator to display.
  final int currentIndex;

  /// The children (branch Navigators) to display in this container.
  final List<Widget> children;

  @override
  Widget build(BuildContext context) {
    return Stack(
      children: children.mapIndexed((int index, Widget navigator) {
        return AnimatedScale(
          scale: index == currentIndex ? 1 : 1.5,
          duration: const Duration(milliseconds: 400),
          child: AnimatedOpacity(
            opacity: index == currentIndex ? 1 : 0,
            opacity: index == currentIndex ? 1 : 0,
            duration: const Duration(milliseconds: 400),
            child: _branchNavigatorWrapper(index, navigator),
          ),
        );
      }).toList(),
    );
  }

  Widget _branchNavigatorWrapper(int index, Widget navigator) => IgnorePointer(
    ignoring: index != currentIndex,
    child: TickerMode(enabled: index == currentIndex, child: navigator),
  );
}
```

功能说明：

1. **Stack 布局**：使用 `Stack` 将所有分支导航器叠加在一起
2. **动画效果**：
   - `AnimatedScale`：非当前分支放大到 1.5 倍
   - `AnimatedOpacity`：非当前分支透明度为 0
3. **性能优化**：
   - `IgnorePointer`：非当前分支忽略触摸事件
   - `TickerMode`：非当前分支禁用动画 ticker，节省资源

### TabbedRootScreen

```dart 498:578:example/lib/others/custom_stateful_shell_route.dart
/// Builds a nested shell using a [TabBar] and [TabBarView].
class TabbedRootScreen extends StatefulWidget {
  /// Constructs a TabbedRootScreen
  const TabbedRootScreen({
    required this.navigationShell,
    required this.children,
    super.key,
  });

  /// The current state of the parent StatefulShellRoute.
  final StatefulNavigationShell navigationShell;

  /// The children (branch Navigators) to display in the [TabBarView].
  final List<Widget> children;

  @override
  State<StatefulWidget> createState() => TabbedRootScreenState();
}

@visibleForTesting
// ignore: public_member_api_docs
class TabbedRootScreenState extends State<TabbedRootScreen>
    with SingleTickerProviderStateMixin {
  @visibleForTesting
  // ignore: public_member_api_docs
  late final TabController tabController = TabController(
    length: widget.children.length,
    vsync: this,
    initialIndex: widget.navigationShell.currentIndex,
  );

  void _switchedTab() {
    if (tabController.index != widget.navigationShell.currentIndex) {
      widget.navigationShell.goBranch(tabController.index);
    }
  }

  @override
  void initState() {
    super.initState();
    tabController.addListener(_switchedTab);
  }

  @override
  void dispose() {
    tabController.removeListener(_switchedTab);
    tabController.dispose();
    super.dispose();
  }

  @override
  void didUpdateWidget(covariant TabbedRootScreen oldWidget) {
    super.didUpdateWidget(oldWidget);
    tabController.index = widget.navigationShell.currentIndex;
  }

  @override
  Widget build(BuildContext context) {
    final List<Tab> tabs = widget.children
        .mapIndexed((int i, _) => Tab(text: 'Tab ${i + 1}'))
        .toList();

    return Scaffold(
      appBar: AppBar(
        title: Text(
          'Section B root (tab: ${widget.navigationShell.currentIndex + 1})',
        ),
        bottom: TabBar(
          controller: tabController,
          tabs: tabs,
          onTap: (int tappedIndex) => _onTabTap(context, tappedIndex),
        ),
      ),
      body: TabBarView(controller: tabController, children: widget.children),
    );
  }

  void _onTabTap(BuildContext context, int index) {
    widget.navigationShell.goBranch(index);
  }
}
```

功能说明：

1. **TabController 同步**：`TabController` 与 `navigationShell.currentIndex` 保持同步
2. **双向绑定**：
   - 用户滑动 `TabBarView` 时，通过监听器调用 `goBranch`
   - 用户点击 `TabBar` 时，通过 `onTap` 调用 `goBranch`
   - 程序调用 `goBranch` 时，通过 `didUpdateWidget` 更新 `TabController.index`
3. **生命周期管理**：在 `dispose` 中清理监听器和控制器

## 页面组件

### RootScreenA

```dart 385:411:example/lib/others/custom_stateful_shell_route.dart
/// Widget for the root page for the first section of the bottom navigation bar.
class RootScreenA extends StatelessWidget {
  /// Creates a RootScreenA
  const RootScreenA({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Section A root')),
      body: Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: <Widget>[
            Text('Screen A', style: Theme.of(context).textTheme.titleLarge),
            const Padding(padding: EdgeInsets.all(4)),
            TextButton(
              onPressed: () {
                GoRouter.of(context).go('/a/details');
              },
              child: const Text('View details'),
            ),
          ],
        ),
      ),
    );
  }
}
```

Section A 的根页面，包含一个按钮可以导航到详情页。

### TabScreen

```dart 663:697:example/lib/others/custom_stateful_shell_route.dart
/// Widget for the pages in the top tab bar.
class TabScreen extends StatelessWidget {
  /// Creates a RootScreen
  const TabScreen({required this.label, required this.detailsPath, super.key});

  /// The label
  final String label;

  /// The path to the detail page
  final String detailsPath;

  @override
  Widget build(BuildContext context) {
    /// If preloading is enabled on the top StatefulShellRoute, this will be
    /// printed directly after the app has been started, but only for the route
    /// that is the initial location ('/b1')
    debugPrint('Building TabScreen - $label');

    return Center(
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: <Widget>[
          Text('Screen $label', style: Theme.of(context).textTheme.titleLarge),
          const Padding(padding: EdgeInsets.all(4)),
          TextButton(
            onPressed: () {
              GoRouter.of(context).go(detailsPath);
            },
            child: const Text('View details'),
            ),
          ),
        ],
      ),
    );
  }
}
```

Section B 中每个标签的页面，包含一个按钮可以导航到对应的详情页。

### DetailsScreen

```dart 413:496:example/lib/others/custom_stateful_shell_route.dart
/// The details screen for either the A or B screen.
class DetailsScreen extends StatefulWidget {
  /// Constructs a [DetailsScreen].
  const DetailsScreen({
    required this.label,
    this.param,
    this.withScaffold = true,
    super.key,
  });

  /// The label to display in the center of the screen.
  final String label;

  /// Optional param
  final String? param;

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

详情页的特点：

1. **可选 Scaffold**：通过 `withScaffold` 参数控制是否使用 `Scaffold`
2. **状态管理**：维护一个计数器，演示状态保持
3. **返回按钮**：当不使用 `Scaffold` 时，显示自定义返回按钮

## 预加载机制

### preload 参数

```dart 113:115:example/lib/others/custom_stateful_shell_route.dart
            // To enable preloading of the initial locations of branches, pass
            // `true` for the parameter `preload` (`false` is default).
            preload: true,
```

当 `preload` 设置为 `true` 时：

- 分支的初始位置会在应用启动时预加载
- 这意味着即使当前不在该分支，其初始页面也会被构建
- 可以提高切换分支时的响应速度

## 使用场景

### 适用场景

1. **复杂的导航结构**：需要多层嵌套导航的应用
2. **状态保持**：需要在切换标签时保持每个标签的导航状态
3. **自定义 UI**：需要自定义导航容器（如动画效果、特殊布局）
4. **深链接支持**：需要支持直接链接到嵌套路由

### 设计模式

这个示例展示了以下设计模式：

1. **组合模式**：通过组合多个 `StatefulShellRoute` 创建复杂导航结构
2. **策略模式**：通过 `navigatorContainerBuilder` 自定义容器实现策略
3. **观察者模式**：`TabController` 监听器同步导航状态

## 注意事项

1. **导航器键管理**：确保为每个分支创建独立的导航器键
2. **状态同步**：当使用自定义控制器（如 `TabController`）时，需要与 `navigationShell.currentIndex` 保持同步
3. **性能考虑**：预加载会消耗更多资源，需要权衡
4. **内存管理**：及时清理监听器和控制器，避免内存泄漏
