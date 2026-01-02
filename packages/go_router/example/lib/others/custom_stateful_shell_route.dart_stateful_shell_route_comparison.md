# StatefulShellRoute 两种配置方式对比

本文档对比了 `StatefulShellRoute` 的两种不同配置方式：使用便捷构造函数 `indexedStack()` 与使用标准构造函数配合 `navigatorContainerBuilder`。

## 代码片段对比

### 方式一：使用 `StatefulShellRoute.indexedStack()`（标准方式）

```dart 33:45:example/lib/stateful_shell_route.dart
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
```

### 方式二：使用 `StatefulShellRoute()` 配合 `navigatorContainerBuilder`（自定义容器）

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

## 核心区别

### 1. 构造函数选择

**方式一**：使用便捷构造函数 `StatefulShellRoute.indexedStack()`

- 内部使用 `IndexedStack` 作为默认容器来管理分支导航器
- 适合大多数标准场景，无需自定义容器行为

**方式二**：使用标准构造函数 `StatefulShellRoute()`

- 需要显式提供 `navigatorContainerBuilder` 来定义容器
- 提供完全的控制权，可以自定义容器实现

### 2. Builder 函数的职责

**方式一**：

- `builder` 函数直接返回完整的 shell widget（如 `ScaffoldWithNavBar`）
- `StatefulNavigationShell` 被直接作为 body 使用

**方式二**：

- `builder` 函数只返回 `navigationShell` 本身
- 实际的容器定制在 `navigatorContainerBuilder` 中完成

### 3. 容器管理方式

**方式一**：

- 使用默认的 `IndexedStack` 容器
- 所有分支导航器都在 `IndexedStack` 中，只有当前索引的导航器可见
- 切换分支时没有动画效果

**方式二**：

- 使用自定义容器（如 `AnimatedBranchContainer`）
- 可以实现自定义的切换动画和视觉效果
- 可以控制分支导航器的显示方式（如透明度、缩放等）

### 4. ScaffoldWithNavBar 的实现差异

**方式一中的 ScaffoldWithNavBar**：

```dart 150:172:example/lib/stateful_shell_route.dart
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
```

- 构造函数只需要 `navigationShell` 参数
- `body` 直接使用 `navigationShell`

**方式二中的 ScaffoldWithNavBar**：

```dart 227:262:example/lib/others/custom_stateful_shell_route.dart
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
```

- 构造函数需要 `navigationShell` 和 `children` 两个参数
- `body` 使用 `AnimatedBranchContainer` 来管理 `children`（分支导航器列表）

## 使用场景建议

### 使用 `StatefulShellRoute.indexedStack()` 当

- 需要标准的底部导航栏功能
- 不需要自定义的切换动画
- 希望使用最简单的配置方式
- 分支切换时只需要简单的显示/隐藏效果

### 使用 `StatefulShellRoute()` 配合 `navigatorContainerBuilder` 当

- 需要自定义的分支切换动画（如淡入淡出、缩放等）
- 需要更精细的控制分支导航器的显示方式
- 需要实现复杂的容器布局（如 `TabBarView`、`PageView` 等）
- 需要嵌套的 shell 结构（如底部导航栏 + 顶部标签栏）

## 技术细节

### IndexedStack 的工作原理

`IndexedStack` 是 Flutter 的一个 widget，它维护一个子 widget 列表，但只显示当前索引对应的子 widget。其他子 widget 虽然不可见，但仍然保持在 widget 树中，因此它们的状态会被保留。

### AnimatedBranchContainer 的实现

在方式二中，`AnimatedBranchContainer` 使用 `Stack` 和动画效果来管理分支导航器：

```dart 348:377:example/lib/others/custom_stateful_shell_route.dart
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
```

这种方式允许在分支切换时添加动画效果，提供更好的用户体验。

## 总结

两种方式都能实现状态保持的底部导航栏功能，主要区别在于：

1. **便捷性**：`indexedStack()` 更简单，适合标准场景
2. **灵活性**：标准构造函数配合 `navigatorContainerBuilder` 提供更多自定义选项
3. **视觉效果**：自定义容器可以实现更丰富的切换动画
4. **复杂度**：自定义方式需要更多的代码，但提供了更大的控制权

选择哪种方式取决于你的具体需求。对于大多数应用，`indexedStack()` 已经足够；如果需要特殊的视觉效果或复杂的布局，则应该使用自定义容器的方式。
