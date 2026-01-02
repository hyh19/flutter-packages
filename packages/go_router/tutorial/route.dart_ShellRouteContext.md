# ShellRouteContext 类详解

## 概述

`ShellRouteContext` 是 go_router 包中用于构建 shell 路由和 Navigator 的上下文对象。它封装了构建 shell 路由所需的所有关键信息，包括路由配置、状态、导航键以及匹配信息等。

## 类定义

```dart 555:613:lib/src/route.dart
/// Context object used when building the shell and Navigator for a shell route.
class ShellRouteContext {
  /// Constructs a [ShellRouteContext].
  ShellRouteContext({
    required this.route,
    required this.routerState,
    required this.navigatorKey,
    required this.match,
    required this.routeMatchList,
    required this.navigatorBuilder,
  });

  /// The associated shell route.
  final ShellRouteBase route;

  /// The current route state associated with [route].
  final GoRouterState routerState;

  /// The [Navigator] key to be used for the nested navigation associated with
  /// [route].
  final GlobalKey<NavigatorState> navigatorKey;

  /// The `ShellRouteMatch` in [routeMatchList] that corresponds to the
  /// associated shell route.
  final ShellRouteMatch match;

  /// The route match list representing the current location within the
  /// associated shell route.
  final RouteMatchList routeMatchList;

  /// Function used to build the [Navigator] for the current route.
  final NavigatorBuilder navigatorBuilder;

  Widget _buildNavigatorForCurrentRoute(
    BuildContext context,
    List<NavigatorObserver>? observers,
    bool notifyRootObserver,
    String? restorationScopeId,
  ) {
    final effectiveObservers = <NavigatorObserver>[...?observers];

    if (notifyRootObserver) {
      final List<NavigatorObserver>? rootObservers = GoRouter.maybeOf(
        context,
      )?.observers;
      if (rootObservers != null) {
        effectiveObservers.add(_MergedNavigatorObserver(rootObservers));
      }
    }

    return navigatorBuilder(
      navigatorKey,
      match,
      routeMatchList,
      effectiveObservers,
      restorationScopeId,
    );
  }
}
```

## 用途

`ShellRouteContext` 主要用于以下场景：

1. **构建 Shell 路由的 Widget 或 Page**：当 `ShellRouteBase` 的子类（如 `ShellRoute` 或 `StatefulShellRoute`）需要构建其 UI 时，会使用这个上下文对象。

2. **管理嵌套导航**：它提供了构建嵌套 Navigator 所需的所有信息，使得 shell 路由可以管理其子路由的导航。

3. **传递路由状态和匹配信息**：将当前路由的状态、匹配信息等传递给构建函数。

## 属性详解

### route

```dart 567:568:lib/src/route.dart
  /// The associated shell route.
  final ShellRouteBase route;
```

关联的 shell 路由对象。这是 `ShellRouteBase` 的实例，可以是 `ShellRoute` 或 `StatefulShellRoute`。

### routerState

```dart 570:571:lib/src/route.dart
  /// The current route state associated with [route].
  final GoRouterState routerState;
```

与当前 shell 路由关联的路由状态。包含当前路由的路径、参数、查询参数等信息。

### navigatorKey

```dart 573:575:lib/src/route.dart
  /// The [Navigator] key to be used for the nested navigation associated with
  /// [route].
  final GlobalKey<NavigatorState> navigatorKey;
```

用于嵌套导航的 Navigator 键。每个 shell 路由都有自己独立的 Navigator，这个键用于标识和管理该 Navigator。

### match

```dart 577:579:lib/src/route.dart
  /// The `ShellRouteMatch` in [routeMatchList] that corresponds to the
  /// associated shell route.
  final ShellRouteMatch match;
```

在 `routeMatchList` 中对应关联 shell 路由的 `ShellRouteMatch`。包含路由匹配的详细信息。

### routeMatchList

```dart 581:583:lib/src/route.dart
  /// The route match list representing the current location within the
  /// associated shell route.
  final RouteMatchList routeMatchList;
```

表示当前 shell 路由内位置的完整路由匹配列表。包含从根路由到当前路由的所有匹配信息。

### navigatorBuilder

```dart 585:586:lib/src/route.dart
  /// Function used to build the [Navigator] for the current route.
  final NavigatorBuilder navigatorBuilder;
```

用于构建当前路由 Navigator 的函数。`NavigatorBuilder` 的类型定义如下：

```dart 56:63:lib/src/route.dart
/// Signature for functions used to build Navigators
typedef NavigatorBuilder =
    Widget Function(
      GlobalKey<NavigatorState> navigatorKey,
      ShellRouteMatch match,
      RouteMatchList matchList,
      List<NavigatorObserver>? observers,
      String? restorationScopeId,
    );
```

## 内部方法

### _buildNavigatorForCurrentRoute

```dart 588:612:lib/src/route.dart
  Widget _buildNavigatorForCurrentRoute(
    BuildContext context,
    List<NavigatorObserver>? observers,
    bool notifyRootObserver,
    String? restorationScopeId,
  ) {
    final effectiveObservers = <NavigatorObserver>[...?observers];

    if (notifyRootObserver) {
      final List<NavigatorObserver>? rootObservers = GoRouter.maybeOf(
        context,
      )?.observers;
      if (rootObservers != null) {
        effectiveObservers.add(_MergedNavigatorObserver(rootObservers));
      }
    }

    return navigatorBuilder(
      navigatorKey,
      match,
      routeMatchList,
      effectiveObservers,
      restorationScopeId,
    );
  }
```

这是一个私有方法，用于构建当前路由的 Navigator Widget。它的工作流程如下：

1. **收集观察者**：首先复制传入的 `observers` 列表。

2. **合并根观察者**：如果 `notifyRootObserver` 为 `true`，会从 `GoRouter` 获取根级别的观察者，并通过 `_MergedNavigatorObserver` 合并到有效观察者列表中。这样，嵌套 Navigator 的导航事件也会通知到根路由的观察者。

3. **构建 Navigator**：调用 `navigatorBuilder` 函数，传入所有必要的参数来构建 Navigator Widget。

## 使用场景

### 在 ShellRouteBase 中使用

`ShellRouteContext` 被传递给 `ShellRouteBase` 的 `buildWidget` 和 `buildPage` 方法：

```dart 534:538:lib/src/route.dart
  Widget? buildWidget(
    BuildContext context,
    GoRouterState state,
    ShellRouteContext shellRouteContext,
  );
```

```dart 544:548:lib/src/route.dart
  Page<dynamic>? buildPage(
    BuildContext context,
    GoRouterState state,
    ShellRouteContext shellRouteContext,
  );
```

### 在 ShellRoute 中的实际使用

在 `ShellRoute` 的 `buildWidget` 方法中，可以看到如何使用 `ShellRouteContext`：

```dart 751:766:lib/src/route.dart
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

这里，`ShellRoute` 使用 `shellRouteContext._buildNavigatorForCurrentRoute` 来构建嵌套的 Navigator，然后将其作为 `child` 参数传递给 `builder` 函数。

### 在 builder.dart 中的创建

`ShellRouteContext` 在 `builder.dart` 中被创建并传递给路由的构建方法：

```dart 281:318:lib/src/builder.dart
    final shellRouteContext = ShellRouteContext(
      route: match.route,
      routerState: state,
      navigatorKey: navigatorKey,
      match: match,
      routeMatchList: widget.matchList,
      navigatorBuilder:
          (
            GlobalKey<NavigatorState> navigatorKey,
            ShellRouteMatch match,
            RouteMatchList matchList,
            List<NavigatorObserver>? observers,
            String? restorationScopeId,
          ) {
            return PopScope(
              // Prevent ShellRoute from being popped, for example
              // by an iOS back gesture, when the route has active sub-routes.
              // TODO(LukasMirbt): Remove when minimum flutter version includes
              // https://github.com/flutter/flutter/pull/152330.
              canPop: match.matches.length == 1,
              child: _CustomNavigator(
                // The state needs to persist across rebuild.
                key: GlobalObjectKey(navigatorKey.hashCode),
                navigatorRestorationId: restorationScopeId,
                navigatorKey: navigatorKey,
                matches: match.matches,
                matchList: matchList,
                configuration: widget.configuration,
                observers: observers ?? const <NavigatorObserver>[],
                onPopPageWithRouteMatch: widget.onPopPageWithRouteMatch,
                // This is used to recursively build pages under this shell route.
                errorBuilder: widget.errorBuilder,
                errorPageBuilder: widget.errorPageBuilder,
                requestFocus: widget.requestFocus,
              ),
            );
          },
    );
```

这里可以看到，`navigatorBuilder` 函数返回一个 `PopScope` 包裹的 `_CustomNavigator`，用于管理嵌套导航。

## 设计模式

`ShellRouteContext` 采用了**上下文对象模式（Context Object Pattern）**，将构建 shell 路由所需的所有相关信息封装在一个对象中，避免了方法签名过长的问题，同时提供了清晰的职责分离。

## 关键特性

1. **不可变性**：所有属性都是 `final` 的，确保上下文对象在创建后不会被修改。

2. **信息聚合**：将分散的路由信息（路由对象、状态、匹配信息等）聚合在一起，方便传递和使用。

3. **观察者管理**：通过 `_buildNavigatorForCurrentRoute` 方法智能地管理 Navigator 观察者，支持将嵌套导航事件通知到根路由观察者。

4. **状态恢复支持**：支持通过 `restorationScopeId` 进行导航状态恢复。

## 总结

`ShellRouteContext` 是 go_router 中 shell 路由系统的核心组件之一，它提供了构建和管理嵌套导航所需的所有上下文信息。通过封装路由状态、匹配信息和 Navigator 构建逻辑，它简化了 shell 路由的实现，使得开发者可以轻松创建具有嵌套导航能力的路由结构。
