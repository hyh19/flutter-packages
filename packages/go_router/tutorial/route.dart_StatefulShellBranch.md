# StatefulShellBranch 类详解

## 概述

`StatefulShellBranch` 是 go_router 包中用于表示状态化导航树中独立分支的类，主要用于配置 `StatefulShellRoute`。该类代表一个有状态的导航分支，每个分支拥有自己独立的 `Navigator`，可以维护各自的导航栈状态。

```dart 1120:1204:lib/src/route.dart
/// Representation of a separate branch in a stateful navigation tree, used to
/// configure [StatefulShellRoute].
///
/// The only required argument when creating a StatefulShellBranch is the
/// sub-routes ([routes]), however sometimes it may be convenient to also
/// provide a [initialLocation]. The value of this parameter is used when
/// loading the branch for the first time (for instance when switching branch
/// using the goBranch method in [StatefulNavigationShell]).
///
/// A separate [Navigator] will be built for each StatefulShellBranch in a
/// [StatefulShellRoute], and the routes of this branch will be placed onto that
/// Navigator instead of the root Navigator. A custom [navigatorKey] can be
/// provided when creating a StatefulShellBranch, which can be useful when the
/// Navigator needs to be accessed elsewhere. If no key is provided, a default
/// one will be created.
@immutable
class StatefulShellBranch {
  /// Constructs a [StatefulShellBranch].
  StatefulShellBranch({
    required this.routes,
    GlobalKey<NavigatorState>? navigatorKey,
    this.initialLocation,
    this.restorationScopeId,
    this.observers,
    this.preload = false,
  }) : navigatorKey = navigatorKey ?? GlobalKey<NavigatorState>() {
    assert(() {
      ShellRouteBase._debugCheckSubRouteParentNavigatorKeys(
        routes,
        this.navigatorKey,
      );
      return true;
    }());
  }

  /// The [GlobalKey] to be used by the [Navigator] built for this branch.
  ///
  /// A separate Navigator will be built for each StatefulShellBranch in a
  /// [StatefulShellRoute] and this key will be used to identify the Navigator.
  /// The routes associated with this branch will be placed o onto that
  /// Navigator instead of the root Navigator.
  final GlobalKey<NavigatorState> navigatorKey;

  /// The list of child routes associated with this route branch.
  final List<RouteBase> routes;

  /// The initial location for this route branch.
  ///
  /// If none is specified, the location of the first descendant [GoRoute] will
  /// be used (i.e. [defaultRoute]). The initial location is used when loading
  /// the branch for the first time (for instance when switching branch using
  /// the goBranch method).
  final String? initialLocation;

  /// Restoration ID to save and restore the state of the navigator, including
  /// its history.
  final String? restorationScopeId;

  /// The observers for this branch.
  ///
  /// The observers parameter is used by the [Navigator] built for this branch.
  final List<NavigatorObserver>? observers;

  /// Whether this route branch should be eagerly loaded when navigating to the
  /// associated StatefulShellRoute for the first time.
  ///
  /// If this property is `false` (the default), the branch will only be loaded
  /// when needed. Set the value to `true` to force the branch to be loaded
  /// immediately when the associated [StatefulShellRoute] is visited for the
  /// first time. In that case, the branch will be preloaded by navigating to
  /// the initial location (see [initialLocation]).
  ///
  /// *Note:* The primary purpose of branch preloading is to enhance the user
  /// experience when switching branches. As with all preloading, there is a
  /// cost in terms of resource use. **Use sparingly** and only after a thorough
  /// trade-off analysis.
  final bool preload;

  /// The default route of this branch, i.e. the first descendant [GoRoute].
  ///
  /// This route will be used when loading the branch for the first time, if
  /// an [initialLocation] has not been provided.
  GoRoute? get defaultRoute =>
      RouteBase.routesRecursively(routes).whereType<GoRoute>().firstOrNull;
}
```

## 核心概念

### 状态化导航分支

在 Flutter 应用中，特别是在使用底部导航栏（BottomNavigationBar）或标签页（TabBar）的场景中，通常希望每个标签页都能保持自己的导航状态。例如，用户在"首页"标签中浏览到某个详情页，然后切换到"消息"标签，之后再切回"首页"时，应该还能看到之前浏览的详情页，而不是回到首页的初始状态。

`StatefulShellBranch` 就是为了实现这种需求而设计的。每个分支都拥有自己独立的 `Navigator`，因此可以独立维护各自的导航栈。

## 构造函数

```dart 1137:1153:lib/src/route.dart
  /// Constructs a [StatefulShellBranch].
  StatefulShellBranch({
    required this.routes,
    GlobalKey<NavigatorState>? navigatorKey,
    this.initialLocation,
    this.restorationScopeId,
    this.observers,
    this.preload = false,
  }) : navigatorKey = navigatorKey ?? GlobalKey<NavigatorState>() {
    assert(() {
      ShellRouteBase._debugCheckSubRouteParentNavigatorKeys(
        routes,
        this.navigatorKey,
      );
      return true;
    }());
  }
```

构造函数接受以下参数：

- **`routes`**（必需）：该分支关联的子路由列表，类型为 `List<RouteBase>`
- **`navigatorKey`**（可选）：用于标识该分支 `Navigator` 的全局键。如果不提供，系统会自动创建一个默认的 `GlobalKey<NavigatorState>`
- **`initialLocation`**（可选）：该分支的初始位置。如果未指定，将使用第一个后代 `GoRoute` 的位置
- **`restorationScopeId`**（可选）：用于保存和恢复导航器状态（包括历史记录）的恢复 ID
- **`observers`**（可选）：该分支 `Navigator` 使用的观察者列表
- **`preload`**（可选）：是否在首次访问关联的 `StatefulShellRoute` 时立即加载该分支，默认为 `false`

构造函数中包含了调试断言，用于检查子路由的父 `Navigator` 键是否正确配置。

## 属性详解

### navigatorKey

```dart 1155:1161:lib/src/route.dart
  /// The [GlobalKey] to be used by the [Navigator] built for this branch.
  ///
  /// A separate Navigator will be built for each StatefulShellBranch in a
  /// [StatefulShellRoute] and this key will be used to identify the Navigator.
  /// The routes associated with this branch will be placed o onto that
  /// Navigator instead of the root Navigator.
  final GlobalKey<NavigatorState> navigatorKey;
```

每个 `StatefulShellBranch` 都会创建一个独立的 `Navigator`，`navigatorKey` 用于标识这个 `Navigator`。该分支下的所有路由都会被放置在这个独立的 `Navigator` 上，而不是根 `Navigator` 上。

如果你需要从代码的其他地方访问某个分支的 `Navigator`（例如，需要在某个页面中通过编程方式弹出到分支的根页面），可以在创建分支时提供自定义的 `navigatorKey`。

### routes

```dart 1163:1164:lib/src/route.dart
  /// The list of child routes associated with this route branch.
  final List<RouteBase> routes;
```

这是该分支关联的子路由列表，是创建 `StatefulShellBranch` 时唯一必需的参数。这些路由构成了该分支的导航树。

### initialLocation

```dart 1166:1172:lib/src/route.dart
  /// The initial location for this route branch.
  ///
  /// If none is specified, the location of the first descendant [GoRoute] will
  /// be used (i.e. [defaultRoute]). The initial location is used when loading
  /// the branch for the first time (for instance when switching branch using
  /// the goBranch method).
  final String? initialLocation;
```

该分支的初始位置。当分支首次加载时（例如，使用 `StatefulNavigationShell.goBranch` 方法切换分支时），会导航到这个初始位置。

如果未指定 `initialLocation`，将使用 `defaultRoute` 的位置（即第一个后代 `GoRoute` 的位置）。

### restorationScopeId

```dart 1174:1176:lib/src/route.dart
  /// Restoration ID to save and restore the state of the navigator, including
  /// its history.
  final String? restorationScopeId;
```

用于状态恢复的 ID。通过设置 `restorationScopeId`，可以让该分支的 `Navigator` 状态（包括导航历史）在应用重启后得到恢复。这对于需要在应用重新启动后恢复用户导航历史的场景非常有用。

### observers

```dart 1178:1181:lib/src/route.dart
  /// The observers for this branch.
  ///
  /// The observers parameter is used by the [Navigator] built for this branch.
  final List<NavigatorObserver>? observers;
```

该分支 `Navigator` 使用的观察者列表。`NavigatorObserver` 可以监听导航事件（如路由的推送、弹出等），常用于实现分析追踪、日志记录等功能。

### preload

```dart 1183:1196:lib/src/route.dart
  /// Whether this route branch should be eagerly loaded when navigating to the
  /// associated StatefulShellRoute for the first time.
  ///
  /// If this property is `false` (the default), the branch will only be loaded
  /// when needed. Set the value to `true` to force the branch to be loaded
  /// immediately when the associated [StatefulShellRoute] is visited for the
  /// first time. In that case, the branch will be preloaded by navigating to
  /// the initial location (see [initialLocation]).
  ///
  /// *Note:* The primary purpose of branch preloading is to enhance the user
  /// experience when switching branches. As with all preloading, there is a
  /// cost in terms of resource use. **Use sparingly** and only after a thorough
  /// trade-off analysis.
  final bool preload;
```

是否在首次访问关联的 `StatefulShellRoute` 时立即加载该分支。默认值为 `false`，意味着分支只会在需要时（即用户切换到该分支时）才被加载。

如果将 `preload` 设置为 `true`，分支会在 `StatefulShellRoute` 首次被访问时立即加载，通过导航到初始位置（`initialLocation`）来实现预加载。

**重要提示**：预加载的目的是提升用户切换分支时的体验，但所有预加载都会带来资源消耗的成本。**请谨慎使用**，只有在经过充分的权衡分析后再考虑启用。

### defaultRoute（getter）

```dart 1198:1203:lib/src/route.dart
  /// The default route of this branch, i.e. the first descendant [GoRoute].
  ///
  /// This route will be used when loading the branch for the first time, if
  /// an [initialLocation] has not been provided.
  GoRoute? get defaultRoute =>
      RouteBase.routesRecursively(routes).whereType<GoRoute>().firstOrNull;
```

这是一个 getter 属性，返回该分支的默认路由，即第一个后代 `GoRoute`。如果未提供 `initialLocation`，则在首次加载分支时使用这个默认路由。

实现上，它通过递归遍历 `routes` 列表，找到第一个 `GoRoute` 类型的路由。如果找不到，则返回 `null`。

## 使用场景

`StatefulShellBranch` 主要用于以下场景：

1. **底部导航栏应用**：每个底部导航栏的标签页对应一个分支，用户可以在不同标签页之间切换，每个标签页保持自己的导航状态。

2. **标签页应用**：类似底部导航栏，每个标签页对应一个独立的分支。

3. **侧边栏导航**：桌面应用或 Web 应用中，侧边栏的每个主要导航项可以对应一个分支。

## 与其他类的关系

### StatefulShellRoute

`StatefulShellBranch` 是 `StatefulShellRoute` 的配置组件。一个 `StatefulShellRoute` 包含多个 `StatefulShellBranch`，每个分支代表一个有状态的导航分支。

### StatefulNavigationShell

`StatefulNavigationShell` 是用于管理 `StatefulShellRoute` 状态的 Widget。它提供了 `goBranch` 方法来切换活动分支，并维护每个分支的导航栈状态。

### Navigator

每个 `StatefulShellBranch` 都会创建一个独立的 `Navigator`。该分支下的所有路由都使用这个独立的 `Navigator`，而不是应用的根 `Navigator`。

## 使用示例

以下是一个使用 `StatefulShellBranch` 的典型示例：

```dart
final GlobalKey<NavigatorState> _homeNavigatorKey = 
    GlobalKey<NavigatorState>(debugLabel: 'homeNav');
final GlobalKey<NavigatorState> _profileNavigatorKey = 
    GlobalKey<NavigatorState>(debugLabel: 'profileNav');

final router = GoRouter(
  navigatorKey: _rootNavigatorKey,
  routes: [
    StatefulShellRoute.indexedStack(
      branches: [
        StatefulShellBranch(
          routes: [
            GoRoute(
              path: '/home',
              builder: (context, state) => HomeScreen(),
              routes: [
                GoRoute(
                  path: 'details/:id',
                  builder: (context, state) => DetailsScreen(
                    id: state.pathParameters['id']!,
                  ),
                ),
              ],
            ),
          ],
          navigatorKey: _homeNavigatorKey,
          initialLocation: '/home',
        ),
        StatefulShellBranch(
          routes: [
            GoRoute(
              path: '/profile',
              builder: (context, state) => ProfileScreen(),
            ),
          ],
          navigatorKey: _profileNavigatorKey,
        ),
      ],
      builder: (context, state, navigationShell) {
        return Scaffold(
          body: navigationShell,
          bottomNavigationBar: BottomNavigationBar(
            currentIndex: navigationShell.currentIndex,
            onTap: (index) {
              navigationShell.goBranch(index);
            },
            items: const [
              BottomNavigationBarItem(
                icon: Icon(Icons.home),
                label: 'Home',
              ),
              BottomNavigationBarItem(
                icon: Icon(Icons.person),
                label: 'Profile',
              ),
            ],
          ),
        );
      },
    ),
  ],
);
```

在这个示例中：

1. 创建了两个 `StatefulShellBranch`，分别对应"首页"和"个人资料"两个标签页。
2. 为每个分支提供了自定义的 `navigatorKey`，以便在需要时访问对应的 `Navigator`。
3. 首页分支设置了 `initialLocation` 为 `'/home'`，而个人资料分支则使用默认的初始位置。
4. 使用 `StatefulNavigationShell.goBranch` 方法在用户点击底部导航栏时切换分支。

## 注意事项

1. **不可变类**：`StatefulShellBranch` 是一个 `@immutable` 类，创建后不可修改。

2. **Navigator 键的唯一性**：在同一个 `StatefulShellRoute` 中，所有分支的 `navigatorKey` 必须是唯一的。

3. **状态恢复**：如果使用了 `restorationScopeId`，需要确保在 `StatefulShellRoute` 级别也设置了相应的 `restorationScopeId`。

4. **预加载权衡**：谨慎使用 `preload` 属性。虽然预加载可以提升用户体验，但会增加资源消耗。只有在经过充分的性能测试和权衡分析后才应该启用。

5. **路由配置**：确保分支中的路由路径配置正确，避免路径冲突。
