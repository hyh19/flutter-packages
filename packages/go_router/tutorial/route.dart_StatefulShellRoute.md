# StatefulShellRoute 类详解

## 概述

`StatefulShellRoute` 是 go_router 包中用于实现有状态嵌套导航的路由类。它继承自 `ShellRouteBase`，与 `ShellRoute` 相似，都是将子路由放置在独立的 `Navigator` 上，但 `StatefulShellRoute` 的核心区别在于它为每个分支（branch）创建了独立的 `Navigator`，从而实现了并行的导航树。

这种设计使得应用可以维护多个独立的导航栈，特别适合实现带底部导航栏（`BottomNavigationBar`）的应用，其中每个标签页都能保持自己的导航状态。

```dart 819:886:lib/src/route.dart
/// A route that displays a UI shell with separate [Navigator]s for its
/// sub-routes.
///
/// Similar to [ShellRoute], this route class places its sub-route on a
/// different Navigator than the root [Navigator]. However, this route class
/// differs in that it creates separate [Navigator]s for each of its nested
/// branches (i.e. parallel navigation trees), making it possible to build an
/// app with stateful nested navigation. This is convenient when for instance
/// implementing a UI with a [BottomNavigationBar], with a persistent navigation
/// state for each tab.
///
/// A StatefulShellRoute is created by specifying a List of
/// [StatefulShellBranch] items, each representing a separate stateful branch
/// in the route tree. StatefulShellBranch provides the root routes and the
/// Navigator key ([GlobalKey]) for the branch, as well as an optional initial
/// location.
///
/// Like [ShellRoute], either a [builder] or a [pageBuilder] must be provided
/// when creating a StatefulShellRoute. However, these builders differ slightly
/// in that they accept a [StatefulNavigationShell] parameter instead of a
/// child Widget. The StatefulNavigationShell can be used to access information
/// about the state of the route, as well as to switch the active branch (i.e.
/// restoring the navigation stack of another branch). The latter is
/// accomplished by using the method [StatefulNavigationShell.goBranch], for
/// example:
///
/// ```dart
/// void _onItemTapped(int index) {
///   navigationShell.goBranch(index: index);
/// }
/// ```
///
/// The StatefulNavigationShell is also responsible for managing and maintaining
/// the state of the branch Navigators. Typically, a shell is built around this
/// Widget, for example by using it as the body of [Scaffold] with a
/// [BottomNavigationBar].
///
/// When creating a StatefulShellRoute, a [navigatorContainerBuilder] function
/// must be provided. This function is responsible for building the actual
/// container for the Widgets representing the branch Navigators. Typically,
/// the Widget returned by this function handles the layout (including
/// [Offstage] handling etc) of the branch Navigators and any animations needed
/// when switching active branch.
///
/// For a default implementation of [navigatorContainerBuilder] that is
/// appropriate for most use cases, consider using the constructor
/// [StatefulShellRoute.indexedStack].
///
/// With StatefulShellRoute (and any route below it), animated transitions
/// between routes in the same navigation stack works the same way as with other
/// route classes, and can be customized using pageBuilder. However, since
/// StatefulShellRoute maintains a set of parallel navigation stacks,
/// any transitions when switching between branches is the responsibility of the
/// branch Navigator container (i.e. [navigatorContainerBuilder]). The default
/// [IndexedStack] implementation ([StatefulShellRoute.indexedStack]) does not
/// use animated transitions, but an example is provided on how to accomplish
/// this (see link to custom StatefulShellRoute example below).
///
/// See also:
/// * [StatefulShellRoute.indexedStack] which provides a default
/// StatefulShellRoute implementation suitable for most use cases.
/// * [Stateful Nested Navigation example](https://github.com/flutter/packages/blob/main/packages/go_router/example/lib/stateful_shell_route.dart)
/// for a complete runnable example using StatefulShellRoute.
/// * [Custom StatefulShellRoute example](https://github.com/flutter/packages/blob/main/packages/go_router/example/lib/others/custom_stateful_shell_route.dart)
/// which demonstrates how to customize the container for the branch Navigators
/// and how to implement animated transitions when switching branches.
///
/// {@category Configuration}
class StatefulShellRoute extends ShellRouteBase {
```

## 核心概念

### 并行导航树

与普通的 `ShellRoute` 不同，`StatefulShellRoute` 为每个分支创建独立的 `Navigator`，形成并行的导航树。这意味着：

- 每个分支维护自己的导航栈
- 切换分支时，之前分支的导航状态会被保留
- 再次切换到之前访问过的分支时，会恢复到之前的导航状态

### 状态化导航

"状态化"指的是导航状态能够在分支切换时被保留。例如，在一个带有底部导航栏的应用中：

1. 用户在"首页"标签中导航到 `/home/details`
2. 用户切换到"消息"标签，导航到 `/messages/list`
3. 用户再次切换回"首页"标签时，仍然停留在 `/home/details`，而不是回到 `/home`

这种体验对于提升用户体验非常重要，特别是在复杂的移动应用中。

## 构造函数

### 主构造函数

```dart 895:917:lib/src/route.dart
  StatefulShellRoute({
    required this.branches,
    super.redirect,
    this.builder,
    this.pageBuilder,
    super.notifyRootObserver,
    required this.navigatorContainerBuilder,
    super.parentNavigatorKey,
    this.restorationScopeId,
    GlobalKey<StatefulNavigationShellState>? key,
  }) : assert(branches.isNotEmpty),
       assert(
         (pageBuilder != null) || (builder != null),
         'One of builder or pageBuilder must be provided',
       ),
       assert(
         _debugUniqueNavigatorKeys(branches).length == branches.length,
         'Navigator keys must be unique',
       ),
       assert(_debugValidateParentNavigatorKeys(branches)),
       assert(_debugValidateRestorationScopeIds(restorationScopeId, branches)),
       _shellStateKey = key ?? GlobalKey<StatefulNavigationShellState>(),
       super._(routes: _routes(branches));
```

主构造函数接受以下参数：

- **`branches`**（必需）：`List<StatefulShellBranch>` 类型，表示该 shell 路由管理的所有分支
- **`builder`** 或 **`pageBuilder`**（必需其一）：用于构建 shell UI 的构建器函数
- **`navigatorContainerBuilder`**（必需）：用于构建分支 `Navigator` 容器的函数
- **`redirect`**：可选的重定向函数
- **`notifyRootObserver`**：是否通知根观察者，默认为 `true`
- **`parentNavigatorKey`**：父 `Navigator` 的键
- **`restorationScopeId`**：状态恢复的 ID
- **`key`**：`StatefulNavigationShellState` 的全局键

构造函数包含多个断言，用于验证：

1. `branches` 不能为空
2. `builder` 或 `pageBuilder` 必须提供其中一个
3. 所有分支的 `Navigator` 键必须唯一
4. 父 `Navigator` 键必须正确设置
5. 如果分支使用了 `restorationScopeId`，则 `StatefulShellRoute` 也必须设置

### indexedStack 构造函数

```dart 929:948:lib/src/route.dart
  StatefulShellRoute.indexedStack({
    required List<StatefulShellBranch> branches,
    bool notifyRootObserver = true,
    GoRouterRedirect? redirect,
    StatefulShellRouteBuilder? builder,
    GlobalKey<NavigatorState>? parentNavigatorKey,
    StatefulShellRoutePageBuilder? pageBuilder,
    String? restorationScopeId,
    GlobalKey<StatefulNavigationShellState>? key,
  }) : this(
         branches: branches,
         redirect: redirect,
         builder: builder,
         pageBuilder: pageBuilder,
         notifyRootObserver: notifyRootObserver,
         parentNavigatorKey: parentNavigatorKey,
         restorationScopeId: restorationScopeId,
         navigatorContainerBuilder: _indexedStackContainerBuilder,
         key: key,
       );
```

`indexedStack` 构造函数是一个便捷构造函数，它提供了基于 `IndexedStack` 的默认 `navigatorContainerBuilder` 实现。`IndexedStack` 会保持所有子组件的状态，但只显示当前选中的那个，这正是底部导航栏所需要的特性。

使用这个构造函数可以简化大多数常见场景的配置，无需自己实现 `navigatorContainerBuilder`。

## 关键属性

### branches

```dart 996:1001:lib/src/route.dart
  /// Representations of the different stateful route branches that this
  /// shell route will manage.
  ///
  /// Each branch uses a separate [Navigator], identified
  /// [StatefulShellBranch.navigatorKey].
  final List<StatefulShellBranch> branches;
```

`branches` 属性存储了该 `StatefulShellRoute` 管理的所有分支。每个分支代表一个独立的导航树，使用各自的 `Navigator`。

### builder 和 pageBuilder

```dart 954:982:lib/src/route.dart
  /// The widget builder for a stateful shell route.
  ///
  /// Similar to [GoRoute.builder], but with an additional
  /// [StatefulNavigationShell] parameter. StatefulNavigationShell is a Widget
  /// responsible for managing the nested navigation for the
  /// matching sub-routes. Typically, a shell route builds its shell around this
  /// Widget. StatefulNavigationShell can also be used to access information
  /// about which branch is active, and also to navigate to a different branch
  /// (using [StatefulNavigationShell.goBranch]).
  ///
  /// Custom implementations may choose to ignore the child parameter passed to
  /// the builder function, and instead use [StatefulNavigationShell] to
  /// create a custom container for the branch Navigators.
  final StatefulShellRouteBuilder? builder;

  /// The page builder for a stateful shell route.
  ///
  /// Similar to [GoRoute.pageBuilder], but with an additional
  /// [StatefulNavigationShell] parameter. StatefulNavigationShell is a Widget
  /// responsible for managing the nested navigation for the
  /// matching sub-routes. Typically, a shell route builds its shell around this
  /// Widget. StatefulNavigationShell can also be used to access information
  /// about which branch is active, and also to navigate to a different branch
  /// (using [StatefulNavigationShell.goBranch]).
  ///
  /// Custom implementations may choose to ignore the child parameter passed to
  /// the builder function, and instead use [StatefulNavigationShell] to
  /// create a custom container for the branch Navigators.
  final StatefulShellRoutePageBuilder? pageBuilder;
```

`builder` 和 `pageBuilder` 用于构建 shell UI，它们与 `GoRoute` 中的对应属性类似，但有一个关键区别：它们接收 `StatefulNavigationShell` 作为参数，而不是 `child` Widget。`StatefulNavigationShell` 提供了访问路由状态和切换活动分支的能力。

### navigatorContainerBuilder

```dart 984:994:lib/src/route.dart
  /// The builder for the branch Navigator container.
  ///
  /// The function responsible for building the container for the branch
  /// Navigators. When this function is invoked, access is provided to a List of
  /// Widgets representing the branch Navigators, where the the index
  /// corresponds to the index of in [branches].
  ///
  /// The builder function is expected to return a Widget that ensures that the
  /// state of the branch Widgets is maintained, for instance by inducting them
  /// in the Widget tree.
  final ShellNavigationContainerBuilder navigatorContainerBuilder;
```

`navigatorContainerBuilder` 负责构建分支 `Navigator` 的容器。当函数被调用时，会接收到一个 Widget 列表，每个 Widget 代表一个分支的 `Navigator`，索引对应 `branches` 中的索引。这个函数应该返回一个 Widget，确保分支 Widget 的状态得到维护（例如，将它们保持在 Widget 树中）。

### restorationScopeId

```dart 950:952:lib/src/route.dart
  /// Restoration ID to save and restore the state of the navigator, including
  /// its history.
  final String? restorationScopeId;
```

`restorationScopeId` 用于保存和恢复 `Navigator` 的状态，包括其导航历史。这对于实现状态恢复功能非常重要，特别是在应用被系统终止后重新启动时。

## 关键方法

### buildWidget 和 buildPage

```dart 1005:1031:lib/src/route.dart
  @override
  Widget? buildWidget(
    BuildContext context,
    GoRouterState state,
    ShellRouteContext shellRouteContext,
  ) {
    if (builder != null) {
      return builder!(context, state, _createShell(context, shellRouteContext));
    }
    return null;
  }

  @override
  Page<dynamic>? buildPage(
    BuildContext context,
    GoRouterState state,
    ShellRouteContext shellRouteContext,
  ) {
    if (pageBuilder != null) {
      return pageBuilder!(
        context,
        state,
        _createShell(context, shellRouteContext),
      );
    }
    return null;
  }
```

这两个方法实现了 `ShellRouteBase` 的抽象方法，用于构建 Widget 或 Page。它们都会调用 `_createShell` 方法创建 `StatefulNavigationShell` 实例，然后将其传递给相应的构建器函数。

### navigatorKeyForSubRoute

```dart 1033:1040:lib/src/route.dart
  @override
  GlobalKey<NavigatorState> navigatorKeyForSubRoute(RouteBase subRoute) {
    final StatefulShellBranch? branch = branches.firstWhereOrNull(
      (StatefulShellBranch e) => e.routes.contains(subRoute),
    );
    assert(branch != null);
    return branch!.navigatorKey;
  }
```

这个方法用于确定给定子路由应该使用哪个 `Navigator` 的键。它会查找包含该子路由的分支，然后返回该分支的 `navigatorKey`。

### _createShell

```dart 1045:1052:lib/src/route.dart
  StatefulNavigationShell _createShell(
    BuildContext context,
    ShellRouteContext shellRouteContext,
  ) => StatefulNavigationShell(
    shellRouteContext: shellRouteContext,
    router: GoRouter.of(context),
    containerBuilder: navigatorContainerBuilder,
  );
```

这个方法用于创建 `StatefulNavigationShell` 实例。`StatefulNavigationShell` 是一个 Widget，负责管理嵌套导航的状态。

## 实现细节

### IndexedStack 容器构建器

```dart 1054:1063:lib/src/route.dart
  static Widget _indexedStackContainerBuilder(
    BuildContext context,
    StatefulNavigationShell navigationShell,
    List<Widget> children,
  ) {
    return _IndexedStackedRouteBranchContainer(
      currentIndex: navigationShell.currentIndex,
      children: children,
    );
  }
```

这是 `indexedStack` 构造函数使用的默认容器构建器。它创建了一个 `_IndexedStackedRouteBranchContainer`，该容器使用 `IndexedStack` 来管理分支 `Navigator` 的显示和状态保持。

### 路由提取

```dart 1065:1066:lib/src/route.dart
  static List<RouteBase> _routes(List<StatefulShellBranch> branches) =>
      branches.expand((StatefulShellBranch e) => e.routes).toList();
```

这个方法将所有分支的路由展平成一个列表，用于传递给父类的构造函数。

### 调试验证方法

#### _debugUniqueNavigatorKeys

```dart 1068:1072:lib/src/route.dart
  static Set<GlobalKey<NavigatorState>> _debugUniqueNavigatorKeys(
    List<StatefulShellBranch> branches,
  ) => Set<GlobalKey<NavigatorState>>.from(
    branches.map((StatefulShellBranch e) => e.navigatorKey),
  );
```

这个方法用于在调试模式下验证所有分支的 `Navigator` 键是否唯一。它创建一个包含所有 `navigatorKey` 的 `Set`，如果 `Set` 的大小与 `branches` 的长度相同，则说明所有键都是唯一的。

#### _debugValidateParentNavigatorKeys

```dart 1074:1088:lib/src/route.dart
  static bool _debugValidateParentNavigatorKeys(
    List<StatefulShellBranch> branches,
  ) {
    for (final branch in branches) {
      for (final RouteBase route in branch.routes) {
        if (route is GoRoute) {
          assert(
            route.parentNavigatorKey == null ||
                route.parentNavigatorKey == branch.navigatorKey,
          );
        }
      }
    }
    return true;
  }
```

这个方法验证每个分支中的子路由的 `parentNavigatorKey` 是否正确设置。如果子路由的 `parentNavigatorKey` 不为 `null`，则它必须与所属分支的 `navigatorKey` 相同。

#### _debugValidateRestorationScopeIds

```dart 1090:1106:lib/src/route.dart
  static bool _debugValidateRestorationScopeIds(
    String? restorationScopeId,
    List<StatefulShellBranch> branches,
  ) {
    if (branches
        .map((StatefulShellBranch e) => e.restorationScopeId)
        .nonNulls
        .isNotEmpty) {
      assert(
        restorationScopeId != null,
        'A restorationScopeId must be set for '
        'the StatefulShellRoute when using restorationScopeIds on one or more '
        'of the branches',
      );
    }
    return true;
  }
```

这个方法验证状态恢复 ID 的配置是否正确。如果任何一个分支设置了 `restorationScopeId`，则 `StatefulShellRoute` 也必须设置 `restorationScopeId`。

### 调试信息

```dart 1108:1117:lib/src/route.dart
  @override
  void debugFillProperties(DiagnosticPropertiesBuilder properties) {
    super.debugFillProperties(properties);
    properties.add(
      DiagnosticsProperty<Iterable<GlobalKey<NavigatorState>>>(
        'navigatorKeys',
        _navigatorKeys,
      ),
    );
  }
```

这个方法用于在调试模式下填充诊断属性，帮助开发者了解路由的配置信息。

## 使用场景

### 底部导航栏应用

最常见的用例是实现带有 `BottomNavigationBar` 的应用，其中每个标签页都维护自己的导航栈。

### 标签页导航

类似地，也可以用于实现 `TabBar` 导航，每个标签页保持独立的状态。

### 多级导航结构

对于需要复杂嵌套导航结构的应用，`StatefulShellRoute` 提供了清晰的架构来管理多个并行的导航树。

## 动画过渡

`StatefulShellRoute` 中的动画过渡分为两种情况：

1. **同一导航栈内的路由过渡**：与普通路由相同，可以使用 `pageBuilder` 自定义过渡动画
2. **分支之间的切换过渡**：由 `navigatorContainerBuilder` 负责实现。默认的 `IndexedStack` 实现不提供过渡动画，但可以通过自定义实现来添加

## 相关类型

- **`StatefulShellBranch`**：表示状态化导航树中的一个分支
- **`StatefulNavigationShell`**：管理嵌套导航状态的 Widget
- **`ShellRoute`**：类似的 shell 路由，但不维护多个独立的导航栈
- **`ShellRouteBase`**：`StatefulShellRoute` 和 `ShellRoute` 的基类

## 总结

`StatefulShellRoute` 是 go_router 包中用于实现有状态嵌套导航的强大工具。通过为每个分支创建独立的 `Navigator`，它使得应用能够维护多个并行的导航栈，从而实现了在分支切换时保持导航状态的能力。这对于构建带有底部导航栏或标签页的现代移动应用至关重要。
