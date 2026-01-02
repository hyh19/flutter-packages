# Stateful Shell Route 示例详解

## 概述

本示例展示了如何使用 `go_router_builder` 创建**有状态的 Shell 路由**（Stateful Shell Route）。这种路由模式特别适用于需要保持多个独立导航栈的应用场景，比如带有底部导航栏的应用，每个标签页都维护自己的导航历史。

### 核心特性

- **独立导航栈**：每个分支（branch）拥有独立的 `Navigator`，可以维护各自的导航历史
- **状态保持**：切换分支时，每个分支的状态（包括滚动位置、表单数据等）会被保留
- **自定义容器**：可以自定义分支导航器的容器，实现独特的 UI 布局和动画效果
- **动画过渡**：示例中实现了自定义的动画容器，在切换分支时提供平滑的过渡效果

## 应用入口和路由配置

```dart 16:29:example/lib/stateful_shell_route_example.dart
void main() => runApp(App());

class App extends StatelessWidget {
  App({super.key});

  @override
  Widget build(BuildContext context) =>
      MaterialApp.router(routerConfig: _router);

  final GoRouter _router = GoRouter(
    routes: $appRoutes,
    initialLocation: '/detailsA',
  );
}
```

应用入口非常简单，使用 `MaterialApp.router` 配置路由。`$appRoutes` 是由代码生成器自动生成的，包含了所有通过注解定义的路由。初始位置设置为 `/detailsA`，即应用启动时显示分支 A 的详情页面。

## 有状态 Shell 路由定义

这是整个示例的核心部分，定义了包含两个分支的 Shell 路由：

```dart 39:77:example/lib/stateful_shell_route_example.dart
@TypedStatefulShellRoute<MyShellRouteData>(
  branches: <TypedStatefulShellBranch<StatefulShellBranchData>>[
    TypedStatefulShellBranch<BranchAData>(
      routes: <TypedRoute<RouteData>>[
        TypedGoRoute<DetailsARouteData>(path: '/detailsA'),
      ],
    ),
    TypedStatefulShellBranch<BranchBData>(
      routes: <TypedRoute<RouteData>>[
        TypedGoRoute<DetailsBRouteData>(path: '/detailsB'),
      ],
    ),
  ],
)
class MyShellRouteData extends StatefulShellRouteData {
  const MyShellRouteData();

  @override
  Widget builder(
    BuildContext context,
    GoRouterState state,
    StatefulNavigationShell navigationShell,
  ) {
    return navigationShell;
  }

  static const String $restorationScopeId = 'restorationScopeId';

  static Widget $navigatorContainerBuilder(
    BuildContext context,
    StatefulNavigationShell navigationShell,
    List<Widget> children,
  ) {
    return ScaffoldWithNavBar(
      navigationShell: navigationShell,
      children: children,
    );
  }
}
```

### 注解说明

`@TypedStatefulShellRoute` 注解用于定义有状态的 Shell 路由：

- **`branches`**：定义 Shell 路由下的所有分支。每个分支都是一个独立的导航栈
- **`MyShellRouteData`**：路由数据类，继承自 `StatefulShellRouteData`

### 路由数据类方法

`MyShellRouteData` 类实现了几个关键方法：

1. **`builder` 方法**：直接返回 `navigationShell`，这是最简单的实现方式。如果需要自定义包装，可以在这里添加额外的 Widget

2. **`$restorationScopeId`**：用于状态恢复的标识符。当应用被系统回收后恢复时，可以使用这个 ID 来恢复路由状态

3. **`$navigatorContainerBuilder`**：这是自定义容器构建器，用于包装所有分支的导航器。在这个示例中，它返回 `ScaffoldWithNavBar`，该组件提供了底部导航栏和自定义的动画容器

## 分支数据类

每个分支都需要一个对应的数据类，继承自 `StatefulShellBranchData`：

```dart 79:88:example/lib/stateful_shell_route_example.dart
class BranchAData extends StatefulShellBranchData {
  const BranchAData();
}

class BranchBData extends StatefulShellBranchData {
  const BranchBData();

  static final GlobalKey<NavigatorState> $navigatorKey = _sectionANavigatorKey;
  static const String $restorationScopeId = 'restorationScopeId';
}
```

### 分支配置选项

- **`BranchAData`**：使用默认配置，系统会自动为它创建 `Navigator` 和 `GlobalKey`

- **`BranchBData`**：展示了如何自定义分支配置：
  - **`$navigatorKey`**：为分支指定自定义的 `Navigator` 的 `GlobalKey`。这在需要从外部访问特定分支的导航器时很有用
  - **`$restorationScopeId`**：为分支指定独立的状态恢复标识符

注意：示例中 `BranchBData` 的 `$navigatorKey` 实际上引用了 `_sectionANavigatorKey`（定义在文件顶部），这个命名可能有些误导，在实际应用中应该为每个分支使用不同的 key。

## 路由数据类

每个具体的路由都有自己的路由数据类：

```dart 90:106:example/lib/stateful_shell_route_example.dart
class DetailsARouteData extends GoRouteData with $DetailsARouteData {
  const DetailsARouteData();

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return const DetailsScreen(label: 'A');
  }
}

class DetailsBRouteData extends GoRouteData with $DetailsBRouteData {
  const DetailsBRouteData();

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return const DetailsScreen(label: 'B');
  }
}
```

这两个路由数据类都继承自 `GoRouteData`，并混入了对应的生成类（`$DetailsARouteData` 和 `$DetailsBRouteData`）。它们都渲染相同的 `DetailsScreen` 组件，只是传入不同的 `label` 参数来区分。

## 自定义导航容器

`ScaffoldWithNavBar` 是自定义的导航容器，它提供了底部导航栏和自定义的动画容器：

```dart 110:162:example/lib/stateful_shell_route_example.dart
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

### 关键组件

1. **`navigationShell`**：`StatefulNavigationShell` 对象，提供了当前活跃分支的索引和导航方法

2. **`children`**：所有分支的 `Navigator` Widget 列表，每个分支对应一个

3. **`AnimatedBranchContainer`**：自定义的动画容器，用于显示当前活跃的分支并处理切换动画

4. **`BottomNavigationBar`**：底部导航栏，显示两个标签页

### 导航逻辑

`_onTap` 方法展示了如何响应底部导航栏的点击：

- 使用 `navigationShell.goBranch()` 方法切换到指定分支
- `initialLocation` 参数：当点击当前已激活的标签时，设置为 `true` 会导航到该分支的初始路由。这是一个常见的 UX 模式，允许用户通过再次点击当前标签返回到该分支的首页

## 动画容器实现

`AnimatedBranchContainer` 实现了分支切换时的动画效果：

```dart 166:201:example/lib/stateful_shell_route_example.dart
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

### 动画机制

1. **`Stack` 布局**：所有分支的导航器都叠加在一起，通过透明度控制显示

2. **`AnimatedOpacity`**：当前活跃的分支透明度为 1（完全可见），其他分支透明度为 0（完全透明）

3. **`AnimatedScale`**：非活跃分支的缩放比例为 1.5，活跃分支为 1.0，创造出缩放动画效果

4. **性能优化**：
   - **`IgnorePointer`**：非活跃分支忽略触摸事件，避免用户与隐藏的 Widget 交互
   - **`TickerMode`**：非活跃分支禁用动画 ticker，节省性能。这对于包含大量动画的页面特别重要

### 动画效果

当切换分支时，会看到：

- 当前分支淡出并放大（scale 1.0 → 1.5，opacity 1 → 0）
- 新分支淡入并缩小（scale 1.5 → 1.0，opacity 0 → 1）
- 整个过程持续 400 毫秒

## 详情页面

`DetailsScreen` 是一个简单的状态化 Widget，用于展示分支内容：

```dart 204:266:example/lib/stateful_shell_route_example.dart
class DetailsScreen extends StatefulWidget {
  /// Constructs a [DetailsScreen].
  const DetailsScreen({required this.label, this.param, this.extra, super.key});

  /// The label to display in the center of the screen.
  final String label;

  /// Optional param
  final String? param;

  /// Optional extra object
  final Object? extra;
  @override
  State<StatefulWidget> createState() => DetailsScreenState();
}

/// The state for DetailsScreen
class DetailsScreenState extends State<DetailsScreen> {
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Details Screen - ${widget.label}')),
      body: _build(context),
    );
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
        ],
      ),
    );
  }
}
```

### 状态保持演示

这个页面包含一个计数器（`_counter`），用于演示 Stateful Shell Route 的状态保持特性：

1. 在分支 A 中增加计数器到某个值（比如 5）
2. 切换到分支 B
3. 再切换回分支 A
4. **计数器仍然保持为 5**，而不是重置为 0

这是因为每个分支维护独立的 `Navigator` 和 Widget 树，切换分支时不会销毁和重建 Widget，从而保持了状态。

## 关键特性说明

### 1. 独立导航栈

每个分支都有自己独立的 `Navigator`，这意味着：

- 分支 A 可以有自己的导航历史（比如：首页 → 列表页 → 详情页）
- 分支 B 也可以有自己的导航历史（比如：首页 → 设置页 → 关于页）
- 在分支之间切换时，各自的导航历史都会被保留

### 2. 状态保持

由于每个分支的 Widget 树不会被销毁，所以：

- 滚动位置会被保留
- 表单输入会被保留
- 状态变量（如计数器）会被保留
- 任何 Widget 状态都会被保留

### 3. 自定义容器

通过 `$navigatorContainerBuilder`，可以完全自定义如何展示分支：

- 可以使用 `BottomNavigationBar`（如本示例）
- 可以使用 `NavigationRail`（侧边导航栏）
- 可以使用 `TabBar`（顶部标签栏）
- 可以实现任何自定义的导航 UI

### 4. 性能优化

`AnimatedBranchContainer` 中的性能优化技巧：

- **`IgnorePointer`**：防止用户与隐藏的 Widget 交互
- **`TickerMode`**：禁用非活跃分支的动画，节省 CPU 和电池

## 使用场景

Stateful Shell Route 特别适用于以下场景：

1. **底部导航栏应用**：如微信、淘宝等应用，每个标签页都有独立的导航栈
2. **侧边导航应用**：如管理后台，左侧导航栏切换不同的功能模块
3. **标签页应用**：如浏览器标签页，每个标签都有独立的导航历史
4. **需要保持状态的复杂应用**：当需要在不同模块间切换且不希望丢失状态时

## 总结

这个示例完整展示了如何使用 `go_router_builder` 创建有状态的 Shell 路由：

1. 使用 `@TypedStatefulShellRoute` 注解定义 Shell 路由
2. 为每个分支创建对应的 `StatefulShellBranchData` 子类
3. 实现 `$navigatorContainerBuilder` 来自定义容器
4. 使用 `StatefulNavigationShell.goBranch()` 进行分支切换
5. 通过自定义容器实现独特的 UI 和动画效果

通过这种方式，可以构建出功能强大、用户体验良好的多分支导航应用。
