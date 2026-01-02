# StatefulShellRoute 的 builder 和 navigatorContainerBuilder 使用说明

本文档解释 `StatefulShellRoute` 中 `builder` 和 `navigatorContainerBuilder` 两个关键参数的作用和使用方式。

## 概述

`StatefulShellRoute` 是 go_router 中用于创建有状态导航壳（shell）的路由类型。它允许每个分支（branch）维护独立的导航状态，非常适合实现底部导航栏、标签页等需要保持多个导航栈的场景。

在 `StatefulShellRoute` 中，有两个重要的构建函数：

1. **`builder`**：构建路由的主要 Widget
2. **`navigatorContainerBuilder`**：构建包含分支导航器的自定义容器

## 第一段代码：顶层 StatefulShellRoute

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

### builder 函数

**作用**：构建路由的主要 Widget，通常直接返回 `navigationShell`。

**参数说明**：

- `BuildContext context`：当前构建上下文
- `GoRouterState state`：当前路由状态，包含路径、参数等信息
- `StatefulNavigationShell navigationShell`：导航壳对象，管理分支导航器的状态

**返回值**：Widget，通常是 `navigationShell` 本身

**设计意图**：在这个实现中，`builder` 函数不做任何自定义处理，直接返回 `navigationShell`。真正的自定义逻辑放在 `navigatorContainerBuilder` 中。

### navigatorContainerBuilder 函数

**作用**：构建一个自定义容器来包裹分支导航器（branch Navigators）。这是实现自定义导航 UI 的关键位置。

**参数说明**：

- `BuildContext context`：当前构建上下文
- `StatefulNavigationShell navigationShell`：导航壳对象，用于获取当前激活的分支索引等信息
- `List<Widget> children`：所有分支的导航器 Widget 列表，每个元素对应一个分支的 Navigator

**返回值**：Widget，通常是包含底部导航栏的 Scaffold

**实现细节**：

在这个例子中，`navigatorContainerBuilder` 返回 `ScaffoldWithNavBar`，它：

1. 使用 `AnimatedBranchContainer` 来管理子导航器的显示和切换动画
2. 提供 `BottomNavigationBar` 用于在分支之间切换
3. 通过 `navigationShell.currentIndex` 获取当前激活的分支索引
4. 通过 `navigationShell.goBranch()` 方法切换分支

### 使用场景

这段代码适用于需要底部导航栏的应用，例如：

- 主应用有多个主要功能模块（如首页、工作、个人中心等）
- 每个模块需要维护独立的导航栈
- 切换模块时，之前的导航状态需要保持

## 第二段代码：嵌套 StatefulShellRoute

```dart 122:151:example/lib/others/custom_stateful_shell_route.dart
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
```

### 结构特点

这段代码展示了**嵌套的 `StatefulShellRoute`**，即在一个 `StatefulShellRoute` 的分支中，又包含另一个 `StatefulShellRoute`。

### builder 函数

与顶层 `StatefulShellRoute` 相同，直接返回 `navigationShell`，不做自定义处理。

### navigatorContainerBuilder 函数

**关键区别**：这里返回的是 `TabbedRootScreen` 而不是 `ScaffoldWithNavBar`。

**`TabbedRootScreen` 的特点**：

1. 使用 `TabBar` 和 `TabBarView` 来管理子导航器
2. 提供顶部标签栏导航，而不是底部导航栏
3. 通过 `TabController` 同步标签切换和导航状态
4. 支持滑动切换标签页

**使用场景**：

这段代码适用于需要嵌套导航的场景，例如：

- 应用的主界面使用底部导航栏（顶层 `StatefulShellRoute`）
- 某个底部导航项内部需要进一步细分，使用顶部标签栏（嵌套 `StatefulShellRoute`）
- 例如：主界面有"首页"和"工作"两个底部导航项，"工作"项内部又有"任务"和"项目"两个标签页

## 设计模式：职责分离

这两段代码展示了一个重要的设计模式：**职责分离**。

### builder 的职责

- 返回路由的主要 Widget（通常是 `navigationShell`）
- 不处理 UI 自定义逻辑
- 保持简单和一致

### navigatorContainerBuilder 的职责

- 提供自定义的容器 Widget
- 管理分支导航器的显示方式
- 实现导航 UI（如底部导航栏、标签栏等）
- 处理分支切换逻辑

### 为什么这样设计？

1. **灵活性**：可以在不修改 `builder` 的情况下，通过 `navigatorContainerBuilder` 实现各种不同的导航 UI
2. **可维护性**：职责清晰，代码更容易理解和维护
3. **可复用性**：`builder` 的逻辑可以复用，只需替换 `navigatorContainerBuilder` 即可实现不同的 UI 风格

## 关键概念

### StatefulNavigationShell

`StatefulNavigationShell` 是导航壳的核心对象，提供以下功能：

- `currentIndex`：当前激活的分支索引
- `goBranch(int index)`：切换到指定分支
- `route.branches`：获取所有分支的路由配置

### 分支导航器（Branch Navigators）

每个分支都有自己独立的 `Navigator`，这意味着：

- 每个分支维护独立的导航栈
- 在分支 A 中导航到详情页，切换到分支 B 再切回分支 A 时，详情页仍然存在
- 支持深度链接到任意分支的任意页面

### children 参数

`navigatorContainerBuilder` 的 `children` 参数是一个 `List<Widget>`，包含所有分支的 Navigator Widget。容器需要：

1. 根据 `currentIndex` 显示对应的 Navigator
2. 处理分支切换时的动画效果
3. 确保非激活的 Navigator 不会接收用户输入（使用 `IgnorePointer`）

## 实际应用建议

### 选择容器类型

- **底部导航栏**：使用 `ScaffoldWithNavBar`（Material 风格）或 `CupertinoScaffoldWithNavBar`（iOS 风格）
- **顶部标签栏**：使用 `TabbedRootScreen`（TabBarView）或 `PagedRootScreen`（PageView）
- **自定义布局**：创建自己的容器 Widget，接收 `navigationShell` 和 `children` 参数

### 性能优化

- 使用 `preload: true` 可以预加载分支的初始位置，提升用户体验
- 对于不常用的分支，可以延迟加载其内容

### 状态管理

- 每个分支的导航状态由 go_router 自动管理
- 不需要手动管理多个 Navigator 的状态
- 使用 `navigationShell.goBranch()` 切换分支时，会自动恢复该分支的最后导航状态

## 总结

`StatefulShellRoute` 的 `builder` 和 `navigatorContainerBuilder` 提供了灵活的导航架构：

- `builder` 保持简单，直接返回 `navigationShell`
- `navigatorContainerBuilder` 实现自定义的导航 UI 容器
- 支持嵌套使用，可以创建复杂的导航层次结构
- 每个分支维护独立的导航栈，支持深度链接和状态保持

这种设计使得 go_router 能够支持从简单的底部导航栏到复杂的嵌套导航结构等各种导航需求。
