# ShellRoute 类详解

## 概述

`ShellRoute` 是 go_router 包中用于创建嵌套导航结构的路由类。它允许开发者在匹配的子路由周围显示一个 UI 外壳（shell），并为其子路由创建一个独立的 `Navigator`，而不是将它们放在根 `Navigator` 上。

```dart 710:710:lib/src/route.dart
class ShellRoute extends ShellRouteBase {
```

`ShellRoute` 继承自 `ShellRouteBase`，这是一个抽象基类，为所有 shell 路由类型提供了通用功能。

## 核心概念

### 什么是 Shell Route？

Shell Route 是一种特殊的路由类型，它：

1. **创建独立的 Navigator**：当 `ShellRoute` 被添加到 `GoRouter` 或 `GoRoute` 的路由列表时，会创建一个新的 `Navigator` 来显示匹配的子路由，而不是将它们放在根 `Navigator` 上。

2. **提供 UI 外壳**：通过 `builder` 或 `pageBuilder` 参数，可以构建一个 UI 外壳（如带有导航栏的 `Scaffold`）来包裹子路由的内容。

3. **支持嵌套导航**：子路由的导航操作（如 `push`、`pop`）发生在 Shell Route 的 `Navigator` 中，而不是根 `Navigator`。

### 使用场景

Shell Route 特别适用于以下场景：

- **底部导航栏应用**：需要为每个标签页保持独立的导航状态
- **侧边栏导航**：需要在侧边栏存在的情况下进行页面导航
- **多级导航结构**：需要在不同层级使用不同的导航器

## 构造函数

```dart 712:732:lib/src/route.dart
  ShellRoute({
    super.redirect,
    this.builder,
    this.pageBuilder,
    super.notifyRootObserver,
    this.observers,
    required super.routes,
    super.parentNavigatorKey,
    GlobalKey<NavigatorState>? navigatorKey,
    this.restorationScopeId,
  }) : assert(routes.isNotEmpty),
       navigatorKey = navigatorKey ?? GlobalKey<NavigatorState>(),
       super._() {
    assert(() {
      ShellRouteBase._debugCheckSubRouteParentNavigatorKeys(
        routes,
        this.navigatorKey,
      );
      return true;
    }());
  }
```

### 参数说明

- **`redirect`**：可选的重定向函数，用于在路由匹配前进行重定向
- **`builder`**：Widget 构建器，用于构建 Shell Route 的 UI 外壳（与 `pageBuilder` 二选一）
- **`pageBuilder`**：Page 构建器，用于构建 Shell Route 的页面（与 `builder` 二选一）
- **`notifyRootObserver`**：是否通知根路由的观察者（默认为 `true`）
- **`observers`**：用于 Shell Route 的 `Navigator` 的观察者列表
- **`routes`**：子路由列表（必需，且不能为空）
- **`parentNavigatorKey`**：父导航器的键，用于指定在哪个 `Navigator` 上显示此路由
- **`navigatorKey`**：Shell Route 自己的 `Navigator` 键（如果未提供，会自动创建）
- **`restorationScopeId`**：用于保存和恢复导航器状态的恢复 ID

### 构造函数行为

1. **断言检查**：确保 `routes` 不为空
2. **自动创建 Navigator Key**：如果未提供 `navigatorKey`，会自动创建一个新的 `GlobalKey<NavigatorState>`
3. **调试检查**：在调试模式下，验证子路由的 `parentNavigatorKey` 是否正确设置

## 核心属性

### builder 和 pageBuilder

```dart 734:748:lib/src/route.dart
  /// The widget builder for a shell route.
  ///
  /// Similar to [GoRoute.builder], but with an additional child parameter. This
  /// child parameter is the Widget managing the nested navigation for the
  /// matching sub-routes. Typically, a shell route builds its shell around this
  /// Widget.
  final ShellRouteBuilder? builder;

  /// The page builder for a shell route.
  ///
  /// Similar to [GoRoute.pageBuilder], but with an additional child parameter.
  /// This child parameter is the Widget managing the nested navigation for the
  /// matching sub-routes. Typically, a shell route builds its shell around this
  /// Widget.
  final ShellRoutePageBuilder? pageBuilder;
```

这两个属性用于构建 Shell Route 的 UI。它们都接收一个额外的 `child` 参数，这个参数是管理嵌套导航的 `Widget`（实际上是一个 `Navigator`）。

**区别**：

- `builder`：直接返回 `Widget`，适用于简单的 UI 外壳
- `pageBuilder`：返回 `Page<dynamic>`，适用于需要页面转换动画的场景

**注意**：`builder` 和 `pageBuilder` 只能使用其中一个，不能同时使用。

### navigatorKey

```dart 792:795:lib/src/route.dart
  /// The [GlobalKey] to be used by the [Navigator] built for this route.
  /// All ShellRoutes build a Navigator by default. Child GoRoutes
  /// are placed onto this Navigator instead of the root Navigator.
  final GlobalKey<NavigatorState> navigatorKey;
```

这是 Shell Route 使用的 `Navigator` 的键。所有 `ShellRoute` 默认都会构建一个 `Navigator`，子 `GoRoute` 会被放置在这个 `Navigator` 上，而不是根 `Navigator`。

### observers

```dart 786:790:lib/src/route.dart
  /// The observers for a shell route.
  ///
  /// The observers parameter is used by the [Navigator] built for this route.
  /// sub-route's observers.
  final List<NavigatorObserver>? observers;
```

用于 Shell Route 的 `Navigator` 的观察者列表。这些观察者可以监听导航事件（如路由推送、弹出等）。

### restorationScopeId

```dart 797:799:lib/src/route.dart
  /// Restoration ID to save and restore the state of the navigator, including
  /// its history.
  final String? restorationScopeId;
```

用于保存和恢复导航器状态的恢复 ID，包括导航历史。这在应用恢复时非常有用，可以恢复之前的导航状态。

## 核心方法

### buildWidget

```dart 750:766:lib/src/route.dart
  @override
  Widget? buildWidget(
    BuildContext context,
    GoRouterState state,
    ShellRouteContext shellRouteContext,
  ) {
    if (builder != null) {
      final Widget navigator = shellRouteContext._buildNavigatorForCurrentRoute(
        context,
        observers,
        notifyRootObserver,
        restorationScopeId,
      );
      return builder!(context, state, navigator);
    }
    return null;
  }
```

此方法用于构建 Shell Route 的 Widget。如果提供了 `builder`，它会：

1. 通过 `shellRouteContext._buildNavigatorForCurrentRoute` 构建嵌套的 `Navigator`
2. 将构建好的 `Navigator` 作为 `child` 参数传递给 `builder`
3. 返回 `builder` 构建的 Widget

如果没有提供 `builder`，返回 `null`（此时应该使用 `buildPage`）。

### buildPage

```dart 768:784:lib/src/route.dart
  @override
  Page<dynamic>? buildPage(
    BuildContext context,
    GoRouterState state,
    ShellRouteContext shellRouteContext,
  ) {
    if (pageBuilder != null) {
      final Widget navigator = shellRouteContext._buildNavigatorForCurrentRoute(
        context,
        observers,
        notifyRootObserver,
        restorationScopeId,
      );
      return pageBuilder!(context, state, navigator);
    }
    return null;
  }
```

此方法用于构建 Shell Route 的 Page。逻辑与 `buildWidget` 类似，但返回的是 `Page<dynamic>` 而不是 `Widget`。

### navigatorKeyForSubRoute

```dart 801:805:lib/src/route.dart
  @override
  GlobalKey<NavigatorState> navigatorKeyForSubRoute(RouteBase subRoute) {
    assert(routes.contains(subRoute));
    return navigatorKey;
  }
```

此方法返回指定子路由应该使用的 `Navigator` 键。对于 `ShellRoute`，所有子路由都使用同一个 `navigatorKey`（即 Shell Route 自己的 `Navigator` 键）。

**注意**：方法中有一个断言，确保传入的 `subRoute` 确实是此 Shell Route 的子路由。

### debugFillProperties

```dart 807:816:lib/src/route.dart
  @override
  void debugFillProperties(DiagnosticPropertiesBuilder properties) {
    super.debugFillProperties(properties);
    properties.add(
      DiagnosticsProperty<GlobalKey<NavigatorState>>(
        'navigatorKey',
        navigatorKey,
      ),
    );
  }
```

此方法用于调试目的，将 `navigatorKey` 添加到诊断属性中，方便在调试工具中查看。

## 使用示例

### 基本用法

文档中提供了一个基本的使用示例：

```dart 686:707:lib/src/route.dart
/// ```dart
/// ShellRoute(
///   builder: (BuildContext context, GoRouterState state, Widget child) {
///     return Scaffold(
///       appBar: AppBar(
///         title: Text('App Shell')
///       ),
///       body: Center(
///         child: child,
///       ),
///     );
///   },
///   routes: [
///     GoRoute(
///       path: 'a'
///       builder: (BuildContext context, GoRouterState state) {
///         return Text('Child Route "/a"');
///       }
///     ),
///   ],
/// ),
/// ```
```

这个示例展示了如何创建一个简单的 Shell Route，它使用 `Scaffold` 作为 UI 外壳，并在 `body` 中显示子路由的内容。

### 高级用法：指定父导航器

文档中还展示了一个更复杂的示例，演示如何让某些子路由显示在根 `Navigator` 上：

```dart 627:679:lib/src/route.dart
/// ```dart
/// final GlobalKey<NavigatorState> _rootNavigatorKey =
///     GlobalKey<NavigatorState>();
///
///   final GoRouter _router = GoRouter(
///     navigatorKey: _rootNavigatorKey,
///     initialLocation: '/a',
///     routes: [
///       ShellRoute(
///         navigatorKey: _shellNavigatorKey,
///         builder: (context, state, child) {
///           return ScaffoldWithNavBar(child: child);
///         },
///         routes: [
///           // This screen is displayed on the ShellRoute's Navigator.
///           GoRoute(
///             path: '/a',
///             builder: (context, state) {
///               return const ScreenA();
///             },
///             routes: <RouteBase>[
///               // This screen is displayed on the ShellRoute's Navigator.
///               GoRoute(
///                 path: 'details',
///                 builder: (BuildContext context, GoRouterState state) {
///                   return const DetailsScreen(label: 'A');
///                 },
///               ),
///             ],
///           ),
///           // Displayed ShellRoute's Navigator.
///           GoRoute(
///             path: '/b',
///             builder: (BuildContext context, GoRouterState state) {
///               return const ScreenB();
///             },
///             routes: <RouteBase>[
///               // Displayed on the root Navigator by specifying the
///               // [parentNavigatorKey].
///               GoRoute(
///                 path: 'details',
///                 parentNavigatorKey: _rootNavigatorKey,
///                 builder: (BuildContext context, GoRouterState state) {
///                   return const DetailsScreen(label: 'B');
///                 },
///               ),
///             ],
///           ),
///         ],
///       ),
///     ],
///   );
/// ```
```

在这个示例中：

- `/a/details` 路由会显示在 Shell Route 的 `Navigator` 上（默认行为）
- `/b/details` 路由通过指定 `parentNavigatorKey: _rootNavigatorKey`，会显示在根 `Navigator` 上

这种设计允许某些子路由（如详情页）突破 Shell Route 的导航限制，直接显示在根 `Navigator` 上，这在某些 UI 设计中非常有用。

## 工作原理

### Navigator 的创建

当 `ShellRoute` 被匹配时：

1. `buildWidget` 或 `buildPage` 方法被调用
2. 通过 `shellRouteContext._buildNavigatorForCurrentRoute` 创建一个新的 `Navigator`
3. 这个 `Navigator` 使用 `navigatorKey` 作为键
4. 子路由会被放置在这个 `Navigator` 的栈中

### 导航流程

1. **路由匹配**：GoRouter 匹配当前路径，找到对应的 `ShellRoute`
2. **构建 Navigator**：为 Shell Route 创建独立的 `Navigator`
3. **构建 Shell UI**：调用 `builder` 或 `pageBuilder`，将 `Navigator` 作为 `child` 传入
4. **显示子路由**：子路由的内容显示在 `Navigator` 中

### 与根 Navigator 的关系

- **默认行为**：子路由显示在 Shell Route 的 `Navigator` 上
- **覆盖行为**：通过设置子路由的 `parentNavigatorKey`，可以让子路由显示在根 `Navigator` 或其他 `Navigator` 上

## 注意事项

1. **routes 不能为空**：`ShellRoute` 必须至少有一个子路由
2. **builder 和 pageBuilder 互斥**：只能使用其中一个，不能同时使用
3. **parentNavigatorKey 验证**：在调试模式下，会验证子路由的 `parentNavigatorKey` 是否正确
4. **Navigator 键的唯一性**：每个 `ShellRoute` 应该使用唯一的 `navigatorKey`，避免冲突

## 总结

`ShellRoute` 是 go_router 中用于创建嵌套导航结构的重要组件。它通过创建独立的 `Navigator` 和提供 UI 外壳，使得开发者可以构建复杂的导航层次结构，同时保持代码的清晰和可维护性。无论是简单的底部导航栏应用，还是复杂的多级导航结构，`ShellRoute` 都能提供强大的支持。
