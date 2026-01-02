# StatefulNavigationShell 类详解

## 概述

`StatefulNavigationShell` 是 go_router 包中用于管理 `StatefulShellRoute` 状态的 Widget 类。它继承自 `StatefulWidget`，主要负责管理和维护 `StatefulShellRoute` 中各个分支（branch）的导航器（Navigator）状态。

通常情况下，这个 Widget 不会直接使用，而是由 `StatefulShellRoute` 内部创建。但是，如果需要自定义分支导航器的容器，可以在 `StatefulShellRoute` 的 `builder` 或 `pageBuilder` 方法中使用 `StatefulNavigationShell`。

## 核心作用

`StatefulNavigationShell` 的主要作用是：

1. **状态管理**：管理 `StatefulShellRoute` 中各个分支的导航状态
2. **分支切换**：提供切换活动分支的能力
3. **容器构建**：通过 `containerBuilder` 自定义分支导航器的容器布局
4. **上下文提供**：在 Widget 树中提供导航状态，供子 Widget 访问

## 类定义

```dart 1234:1355:lib/src/route.dart
class StatefulNavigationShell extends StatefulWidget {
  /// Constructs an [StatefulNavigationShell].
  StatefulNavigationShell({
    required this.shellRouteContext,
    required GoRouter router,
    required this.containerBuilder,
  }) : assert(shellRouteContext.route is StatefulShellRoute),
       _router = router,
       currentIndex = _indexOfBranchNavigatorKey(
         shellRouteContext.route as StatefulShellRoute,
         shellRouteContext.navigatorKey,
       ),
       super(
         key: (shellRouteContext.route as StatefulShellRoute)._shellStateKey,
       );

  /// The ShellRouteContext responsible for building the Navigator for the
  /// current [StatefulShellBranch].
  final ShellRouteContext shellRouteContext;

  final GoRouter _router;

  /// The builder for a custom container for shell route Navigators.
  final ShellNavigationContainerBuilder containerBuilder;

  /// The index of the currently active [StatefulShellBranch].
  ///
  /// Corresponds to the index in the branches field of [StatefulShellRoute].
  final int currentIndex;

  /// The associated [StatefulShellRoute].
  StatefulShellRoute get route => shellRouteContext.route as StatefulShellRoute;

  /// Navigate to the last location of the [StatefulShellBranch] at the provided
  /// index in the associated [StatefulShellBranch].
  ///
  /// This method will switch the currently active branch [Navigator] for the
  /// [StatefulShellRoute]. If the branch has not been visited before, or if
  /// initialLocation is true, this method will navigate to initial location of
  /// the branch (see [StatefulShellBranch.initialLocation]).
  // TODO(chunhtai): figure out a way to avoid putting navigation API in widget
  // class.
  void goBranch(int index, {bool initialLocation = false}) {
    final route = shellRouteContext.route as StatefulShellRoute;
    final StatefulNavigationShellState? shellState =
        route._shellStateKey.currentState;
    if (shellState != null) {
      shellState.goBranch(index, initialLocation: initialLocation);
    } else {
      _router.go(_effectiveInitialBranchLocation(index));
    }
  }

  /// Checks if the provided branch is loaded (i.e. has navigation state
  /// associated with it).
  @visibleForTesting
  List<StatefulShellBranch> get debugLoadedBranches =>
      route._shellStateKey.currentState?._loadedBranches ??
      <StatefulShellBranch>[];

  /// Gets the effective initial location for the branch at the provided index
  /// in the associated [StatefulShellRoute].
  ///
  /// The effective initial location is either the
  /// [StatefulShellBranch.initialLocation], if specified, or the location of the
  /// [StatefulShellBranch.defaultRoute].
  String _effectiveInitialBranchLocation(int index) {
    final route = shellRouteContext.route as StatefulShellRoute;
    final StatefulShellBranch branch = route.branches[index];
    final String? initialLocation = branch.initialLocation;
    if (initialLocation != null) {
      return initialLocation;
    } else {
      /// Recursively traverses the routes of the provided StackedShellBranch to
      /// find the first GoRoute, from which a full path will be derived.
      final GoRoute route = branch.defaultRoute!;
      final parameters = <String>[];
      patternToRegExp(
        route.path,
        parameters,
        caseSensitive: route.caseSensitive,
      );
      assert(parameters.isEmpty);
      final String fullPath = _router.configuration.locationForRoute(route)!;
      return patternToPath(
        fullPath,
        shellRouteContext.routerState.pathParameters,
      );
    }
  }

  @override
  State<StatefulWidget> createState() => StatefulNavigationShellState();

  /// Gets the state for the nearest stateful shell route in the Widget tree.
  static StatefulNavigationShellState of(BuildContext context) {
    final StatefulNavigationShellState? shellState = context
        .findAncestorStateOfType<StatefulNavigationShellState>();
    assert(shellState != null);
    return shellState!;
  }

  /// Gets the state for the nearest stateful shell route in the Widget tree.
  ///
  /// Returns null if no stateful shell route is found.
  static StatefulNavigationShellState? maybeOf(BuildContext context) {
    final StatefulNavigationShellState? shellState = context
        .findAncestorStateOfType<StatefulNavigationShellState>();
    return shellState;
  }

  static int _indexOfBranchNavigatorKey(
    StatefulShellRoute route,
    GlobalKey<NavigatorState> navigatorKey,
  ) {
    final int index = route.branches.indexWhere(
      (StatefulShellBranch branch) => branch.navigatorKey == navigatorKey,
    );
    assert(index >= 0);
    return index;
  }
}
```

## 构造函数

### 参数说明

```dart 1236:1248:lib/src/route.dart
StatefulNavigationShell({
  required this.shellRouteContext,
  required GoRouter router,
  required this.containerBuilder,
}) : assert(shellRouteContext.route is StatefulShellRoute),
     _router = router,
     currentIndex = _indexOfBranchNavigatorKey(
       shellRouteContext.route as StatefulShellRoute,
       shellRouteContext.navigatorKey,
     ),
     super(
       key: (shellRouteContext.route as StatefulShellRoute)._shellStateKey,
     );
```

构造函数接收三个必需参数：

1. **`shellRouteContext`**：`ShellRouteContext` 类型，负责为当前的 `StatefulShellBranch` 构建 Navigator。这个上下文包含了路由状态、导航器键等关键信息。

2. **`router`**：`GoRouter` 类型，路由器的实例，用于执行导航操作。

3. **`containerBuilder`**：`ShellNavigationContainerBuilder` 类型，用于构建自定义容器的构建器函数。

### 初始化逻辑

在构造函数中，会执行以下初始化：

- **类型断言**：确保 `shellRouteContext.route` 是 `StatefulShellRoute` 类型
- **计算当前索引**：通过 `_indexOfBranchNavigatorKey` 方法，根据导航器键找到对应的分支索引
- **设置 Widget Key**：使用 `StatefulShellRoute` 的 `_shellStateKey` 作为 Widget 的 key，这样可以确保状态能够正确关联

## 属性详解

### shellRouteContext

```dart 1250:1252:lib/src/route.dart
/// The ShellRouteContext responsible for building the Navigator for the
/// current [StatefulShellBranch].
final ShellRouteContext shellRouteContext;
```

`ShellRouteContext` 包含了构建当前分支 Navigator 所需的所有信息，包括路由状态、导航器键、路由匹配列表等。

### containerBuilder

```dart 1256:1257:lib/src/route.dart
/// The builder for a custom container for shell route Navigators.
final ShellNavigationContainerBuilder containerBuilder;
```

`containerBuilder` 是一个函数类型，用于构建自定义容器。其类型定义为：

```dart 1208:1213:lib/src/route.dart
typedef ShellNavigationContainerBuilder =
    Widget Function(
      BuildContext context,
      StatefulNavigationShell navigationShell,
      List<Widget> children,
    );
```

这个构建器接收三个参数：

- `context`：构建上下文
- `navigationShell`：当前的 `StatefulNavigationShell` 实例
- `children`：表示各个分支 Navigator 的 Widget 列表

### currentIndex

```dart 1259:1262:lib/src/route.dart
/// The index of the currently active [StatefulShellBranch].
///
/// Corresponds to the index in the branches field of [StatefulShellRoute].
final int currentIndex;
```

当前活动分支的索引，对应 `StatefulShellRoute.branches` 列表中的索引位置。

### route

```dart 1264:1265:lib/src/route.dart
/// The associated [StatefulShellRoute].
StatefulShellRoute get route => shellRouteContext.route as StatefulShellRoute;
```

获取关联的 `StatefulShellRoute` 实例，通过类型转换从 `shellRouteContext.route` 获取。

## 方法详解

### goBranch

```dart 1267:1285:lib/src/route.dart
/// Navigate to the last location of the [StatefulShellBranch] at the provided
/// index in the associated [StatefulShellBranch].
///
/// This method will switch the currently active branch [Navigator] for the
/// [StatefulShellRoute]. If the branch has not been visited before, or if
/// initialLocation is true, this method will navigate to initial location of
/// the branch (see [StatefulShellBranch.initialLocation]).
// TODO(chunhtai): figure out a way to avoid putting navigation API in widget
// class.
void goBranch(int index, {bool initialLocation = false}) {
  final route = shellRouteContext.route as StatefulShellRoute;
  final StatefulNavigationShellState? shellState =
      route._shellStateKey.currentState;
  if (shellState != null) {
    shellState.goBranch(index, initialLocation: initialLocation);
  } else {
    _router.go(_effectiveInitialBranchLocation(index));
  }
}
```

`goBranch` 方法用于切换到指定索引的分支。

**参数**：

- `index`：目标分支的索引
- `initialLocation`：如果为 `true`，则导航到分支的初始位置；如果为 `false`，则导航到分支上次访问的位置

**工作原理**：

1. 首先尝试获取 `StatefulNavigationShellState`
2. 如果状态存在，调用状态的 `goBranch` 方法
3. 如果状态不存在（Widget 尚未构建），则通过路由器直接导航到分支的有效初始位置

**使用场景**：
在底部导航栏的点击事件中调用，例如：

```dart
void _onItemTapped(int index) {
  navigationShell.goBranch(index);
}
```

### _effectiveInitialBranchLocation

```dart 1294:1323:lib/src/route.dart
/// Gets the effective initial location for the branch at the provided index
/// in the associated [StatefulShellRoute].
///
/// The effective initial location is either the
/// [StatefulShellBranch.initialLocation], if specified, or the location of the
/// [StatefulShellBranch.defaultRoute].
String _effectiveInitialBranchLocation(int index) {
  final route = shellRouteContext.route as StatefulShellRoute;
  final StatefulShellBranch branch = route.branches[index];
  final String? initialLocation = branch.initialLocation;
  if (initialLocation != null) {
    return initialLocation;
  } else {
    /// Recursively traverses the routes of the provided StackedShellBranch to
    /// find the first GoRoute, from which a full path will be derived.
    final GoRoute route = branch.defaultRoute!;
    final parameters = <String>[];
    patternToRegExp(
      route.path,
      parameters,
      caseSensitive: route.caseSensitive,
    );
    assert(parameters.isEmpty);
    final String fullPath = _router.configuration.locationForRoute(route)!;
    return patternToPath(
      fullPath,
      shellRouteContext.routerState.pathParameters,
    );
  }
}
```

这是一个私有方法，用于获取分支的有效初始位置。

**逻辑**：

1. 如果分支指定了 `initialLocation`，直接返回该位置
2. 否则，找到分支的 `defaultRoute`（第一个 `GoRoute`）
3. 通过路由配置将路由路径转换为完整的位置字符串
4. 使用路径参数填充路径模板

### of 和 maybeOf

```dart 1328:1343:lib/src/route.dart
/// Gets the state for the nearest stateful shell route in the Widget tree.
static StatefulNavigationShellState of(BuildContext context) {
  final StatefulNavigationShellState? shellState = context
      .findAncestorStateOfType<StatefulNavigationShellState>();
  assert(shellState != null);
  return shellState!;
}

/// Gets the state for the nearest stateful shell route in the Widget tree.
///
/// Returns null if no stateful shell route is found.
static StatefulNavigationShellState? maybeOf(BuildContext context) {
  final StatefulNavigationShellState? shellState = context
      .findAncestorStateOfType<StatefulNavigationShellState>();
  return shellState;
}
```

这两个静态方法用于在 Widget 树中查找最近的 `StatefulNavigationShellState`。

- **`of`**：如果找不到会抛出断言错误，适用于确定状态一定存在的场景
- **`maybeOf`**：如果找不到返回 `null`，适用于状态可能不存在的场景

**使用场景**：
在 `StatefulShellRoute` 的子 Widget 中访问导航状态，例如在自定义容器 Widget 中：

```dart
class MyCustomShell extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final shellState = StatefulNavigationShell.maybeOf(context);
    if (shellState != null) {
      // 使用 shellState 进行自定义操作
    }
    return Container(/* ... */);
  }
}
```

### _indexOfBranchNavigatorKey

```dart 1345:1354:lib/src/route.dart
static int _indexOfBranchNavigatorKey(
  StatefulShellRoute route,
  GlobalKey<NavigatorState> navigatorKey,
) {
  final int index = route.branches.indexWhere(
    (StatefulShellBranch branch) => branch.navigatorKey == navigatorKey,
  );
  assert(index >= 0);
  return index;
}
```

这是一个静态辅助方法，用于根据导航器键查找对应的分支索引。

**工作原理**：

1. 在 `route.branches` 中查找 `navigatorKey` 匹配的分支
2. 返回找到的索引
3. 如果找不到（返回 -1），断言会失败

这个方法在构造函数中被调用，用于初始化 `currentIndex` 属性。

## 使用示例

### 基本用法

在 `StatefulShellRoute` 的 `builder` 中使用：

```dart 1224:1233:lib/src/route.dart
/// Example:
/// ```dart
/// builder: (BuildContext context, GoRouterState state,
///     StatefulNavigationShell navigationShell) {
///   return StatefulNavigationShell(
///     shellRouteState: state,
///     containerBuilder: (_, __, List<Widget> children) => MyCustomShell(shellState: state, children: children),
///   );
/// }
/// ```
```

### 自定义容器示例

如果需要自定义分支导航器的容器，可以这样使用：

```dart
StatefulShellRoute(
  branches: branches,
  builder: (BuildContext context, GoRouterState state,
      StatefulNavigationShell navigationShell) {
    return StatefulNavigationShell(
      shellRouteContext: /* ... */,
      router: router,
      containerBuilder: (context, shell, children) {
        return MyCustomContainer(
          currentIndex: shell.currentIndex,
          children: children,
          onBranchChanged: (index) {
            shell.goBranch(index);
          },
        );
      },
    );
  },
)
```

## 与 StatefulShellRoute 的关系

`StatefulNavigationShell` 是 `StatefulShellRoute` 的核心组件：

1. **内部创建**：通常由 `StatefulShellRoute` 内部创建和管理
2. **状态管理**：负责维护各个分支的导航状态
3. **容器构建**：通过 `containerBuilder` 构建分支导航器的容器
4. **分支切换**：提供 `goBranch` 方法实现分支间的切换

## 注意事项

1. **通常不直接使用**：这个 Widget 主要是内部使用，除非需要自定义容器布局，否则不需要直接使用

2. **状态键的重要性**：Widget 的 key 使用了 `StatefulShellRoute` 的 `_shellStateKey`，这确保了状态的正确关联和持久化

3. **分支索引**：`currentIndex` 是在构造函数中计算的，基于 `navigatorKey` 匹配

4. **导航方法的位置**：代码中有 TODO 注释提到，考虑将导航 API 从 Widget 类中移出，这反映了设计上的考虑

## 总结

`StatefulNavigationShell` 是 `StatefulShellRoute` 实现状态化嵌套导航的关键组件。它封装了分支切换、状态管理等复杂逻辑，为开发者提供了简洁的 API。虽然通常不需要直接使用，但了解其工作原理有助于更好地理解和使用 `StatefulShellRoute`。
