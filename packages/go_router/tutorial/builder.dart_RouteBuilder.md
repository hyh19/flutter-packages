# RouteBuilder 类详解

## 概述

`RouteBuilder` 是 GoRouter 中负责构建顶层 Navigator 的核心类。它接收路由配置和匹配结果，然后构建出包含所有路由页面的 Navigator 组件，这是 GoRouter 路由系统的基础构建块。

## 类定义

```dart 46:125:lib/src/builder.dart
/// Builds the top-level Navigator for GoRouter.
class RouteBuilder {
  /// [RouteBuilder] constructor.
  RouteBuilder({
    required this.configuration,
    required this.builderWithNav,
    required this.errorPageBuilder,
    required this.errorBuilder,
    required this.restorationScopeId,
    required this.observers,
    required this.onPopPageWithRouteMatch,
    this.requestFocus = true,
  });

  /// Builder function for a go router with Navigator.
  final GoRouterBuilderWithNav builderWithNav;

  /// Error page builder for the go router delegate.
  final GoRouterPageBuilder? errorPageBuilder;

  /// Error widget builder for the go router delegate.
  final GoRouterWidgetBuilder? errorBuilder;

  /// The route configuration for the app.
  final RouteConfiguration configuration;

  /// Restoration ID to save and restore the state of the navigator, including
  /// its history.
  final String? restorationScopeId;

  /// Whether or not the navigator created by this builder and it's new topmost route should request focus
  /// when the new route is pushed onto the navigator.
  ///
  /// Defaults to true.
  final bool requestFocus;

  /// NavigatorObserver used to receive notifications when navigating in between routes.
  /// changes.
  final List<NavigatorObserver> observers;

  /// A callback called when a `route` produced by `match` is about to be popped
  /// with the `result`.
  ///
  /// If this method returns true, this builder pops the `route` and `match`.
  ///
  /// If this method returns false, this builder aborts the pop.
  final PopPageWithRouteMatchCallback onPopPageWithRouteMatch;

  /// Builds the top-level Navigator for the given [RouteMatchList].
  Widget build(
    BuildContext context,
    RouteMatchList matchList,
    bool routerNeglect, // TODO(tolo): This parameter is not used and should be
    // removed in the next major version.
  ) {
    if (matchList.isEmpty && !matchList.isError) {
      // The build method can be called before async redirect finishes. Build a
      // empty box until then.
      return const SizedBox.shrink();
    }
    assert(matchList.isError || !matchList.last.route.redirectOnly);
    return builderWithNav(
      context,
      _CustomNavigator(
        // The state needs to persist across rebuild.
        key: GlobalObjectKey(configuration.navigatorKey.hashCode),
        navigatorKey: configuration.navigatorKey,
        observers: observers,
        navigatorRestorationId: restorationScopeId,
        onPopPageWithRouteMatch: onPopPageWithRouteMatch,
        matchList: matchList,
        matches: matchList.matches,
        configuration: configuration,
        errorBuilder: errorBuilder,
        errorPageBuilder: errorPageBuilder,
        requestFocus: requestFocus,
      ),
    );
  }
}
```

## 核心职责

`RouteBuilder` 的主要职责是：

1. **接收路由匹配结果**：从路由解析器接收 `RouteMatchList`，其中包含了当前 URL 对应的所有路由匹配
2. **构建 Navigator**：根据匹配结果构建包含所有页面的 Navigator 组件
3. **处理边界情况**：处理空匹配、错误状态等特殊情况
4. **委托构建**：将实际的 Navigator 构建工作委托给 `_CustomNavigator` 组件

## 属性详解

### builderWithNav

```dart 60:61:lib/src/builder.dart
  /// Builder function for a go router with Navigator.
  final GoRouterBuilderWithNav builderWithNav;
```

这是一个构建函数，类型定义为：

```dart 19:21:lib/src/builder.dart
/// Signature of a go router builder function with navigator.
typedef GoRouterBuilderWithNav =
    Widget Function(BuildContext context, Widget child);
```

它接收一个 `Widget child`（通常是 `_CustomNavigator`），然后将其包装在应用特定的容器中（如 `MaterialApp` 或 `CupertinoApp`）。

### errorPageBuilder 和 errorBuilder

```dart 63:67:lib/src/builder.dart
  /// Error page builder for the go router delegate.
  final GoRouterPageBuilder? errorPageBuilder;

  /// Error widget builder for the go router delegate.
  final GoRouterWidgetBuilder? errorBuilder;
```

这两个属性用于处理路由错误：

- **errorPageBuilder**：构建一个完整的 `Page` 对象用于错误页面
- **errorBuilder**：只构建错误页面的 Widget，会被包装在平台特定的 Page 中

如果两者都提供，优先使用 `errorPageBuilder`。

### configuration

```dart 69:70:lib/src/builder.dart
  /// The route configuration for the app.
  final RouteConfiguration configuration;
```

包含应用的所有路由配置信息，包括路由定义、导航键等。

### restorationScopeId

```dart 72:74:lib/src/builder.dart
  /// Restoration ID to save and restore the state of the navigator, including
  /// its history.
  final String? restorationScopeId;
```

用于状态恢复的标识符。当应用被系统杀死后重新启动时，Flutter 可以使用这个 ID 来恢复 Navigator 的状态，包括导航历史。

### requestFocus

```dart 76:80:lib/src/builder.dart
  /// Whether or not the navigator created by this builder and it's new topmost route should request focus
  /// when the new route is pushed onto the navigator.
  ///
  /// Defaults to true.
  final bool requestFocus;
```

控制当新路由被推送到 Navigator 时，是否自动请求焦点。默认为 `true`，这对于可访问性和键盘导航很重要。

### observers

```dart 82:84:lib/src/builder.dart
  /// NavigatorObserver used to receive notifications when navigating in between routes.
  /// changes.
  final List<NavigatorObserver> observers;
```

Navigator 观察者列表，用于监听路由变化事件（如路由推送、弹出等）。常用于实现分析追踪、日志记录等功能。

### onPopPageWithRouteMatch

```dart 86:92:lib/src/builder.dart
  /// A callback called when a `route` produced by `match` is about to be popped
  /// with the `result`.
  ///
  /// If this method returns true, this builder pops the `route` and `match`.
  ///
  /// If this method returns false, this builder aborts the pop.
  final PopPageWithRouteMatchCallback onPopPageWithRouteMatch;
```

当路由即将被弹出时调用的回调函数。类型定义为：

```dart 35:44:lib/src/builder.dart
/// Signature for a function that takes in a `route` to be popped with
/// the `result` and returns a boolean decision on whether the pop
/// is successful.
///
/// The `match` is the corresponding [RouteMatch] the `route`
/// associates with.
///
/// Used by of [RouteBuilder.onPopPageWithRouteMatch].
typedef PopPageWithRouteMatchCallback =
    bool Function(Route<dynamic> route, dynamic result, RouteMatchBase match);
```

返回 `true` 表示允许弹出，返回 `false` 表示阻止弹出。这可以用于实现"确认退出"等功能。

## build 方法详解

`build` 方法是 `RouteBuilder` 的核心方法，负责根据路由匹配结果构建 Navigator。

### 方法签名

```dart 94:100:lib/src/builder.dart
  /// Builds the top-level Navigator for the given [RouteMatchList].
  Widget build(
    BuildContext context,
    RouteMatchList matchList,
    bool routerNeglect, // TODO(tolo): This parameter is not used and should be
    // removed in the next major version.
  ) {
```

**参数说明**：

- **context**：BuildContext，用于访问 Flutter 的构建上下文
- **matchList**：路由匹配列表，包含当前 URL 对应的所有路由匹配
- **routerNeglect**：已废弃的参数，将在下一个主版本中移除

### 空匹配处理

```dart 101:105:lib/src/builder.dart
    if (matchList.isEmpty && !matchList.isError) {
      // The build method can be called before async redirect finishes. Build a
      // empty box until then.
      return const SizedBox.shrink();
    }
```

当匹配列表为空且不是错误状态时，返回一个空的 `SizedBox`。这通常发生在异步重定向尚未完成时，此时还没有有效的路由匹配结果。

### 断言检查

```dart 106:106:lib/src/builder.dart
    assert(matchList.isError || !matchList.last.route.redirectOnly);
```

这个断言确保：

- 如果是错误状态，可以继续构建错误页面
- 如果不是错误状态，最后一个路由不能是仅重定向路由（`redirectOnly`）

仅重定向路由不应该出现在匹配列表的末尾，因为它们不应该渲染页面。

### Navigator 构建

```dart 107:123:lib/src/builder.dart
    return builderWithNav(
      context,
      _CustomNavigator(
        // The state needs to persist across rebuild.
        key: GlobalObjectKey(configuration.navigatorKey.hashCode),
        navigatorKey: configuration.navigatorKey,
        observers: observers,
        navigatorRestorationId: restorationScopeId,
        onPopPageWithRouteMatch: onPopPageWithRouteMatch,
        matchList: matchList,
        matches: matchList.matches,
        configuration: configuration,
        errorBuilder: errorBuilder,
        errorPageBuilder: errorPageBuilder,
        requestFocus: requestFocus,
      ),
    );
```

这里创建了一个 `_CustomNavigator` 组件，它是 GoRouter 自定义的 Navigator 实现。关键点：

1. **GlobalObjectKey**：使用 `configuration.navigatorKey.hashCode` 作为 key，确保在重建时 Navigator 状态能够持久化
2. **传递所有必要参数**：将配置、观察者、错误处理器等所有必要信息传递给 `_CustomNavigator`
3. **委托给 builderWithNav**：最后通过 `builderWithNav` 将 `_CustomNavigator` 包装在应用特定的容器中

## 使用场景

`RouteBuilder` 主要在 `GoRouterDelegate` 中使用：

```dart
builder = RouteBuilder(
  configuration: configuration,
  builderWithNav: builderWithNav,
  errorPageBuilder: errorPageBuilder,
  errorBuilder: errorBuilder,
  restorationScopeId: restorationScopeId,
  observers: observers,
  onPopPageWithRouteMatch: _handlePopPageWithRouteMatch,
  requestFocus: requestFocus,
);
```

当路由状态发生变化时，`GoRouterDelegate` 会调用 `builder.build()` 来重新构建 Navigator。

## 设计模式

`RouteBuilder` 采用了以下设计模式：

1. **建造者模式（Builder Pattern）**：将复杂的 Navigator 构建过程封装在 `RouteBuilder` 中
2. **委托模式（Delegation Pattern）**：将实际的 Navigator 构建委托给 `_CustomNavigator`，将应用包装委托给 `builderWithNav`
3. **策略模式（Strategy Pattern）**：通过 `errorPageBuilder` 和 `errorBuilder` 允许自定义错误处理策略

## 总结

`RouteBuilder` 是 GoRouter 路由系统的核心组件之一，它负责将路由匹配结果转换为实际的 Navigator 组件。通过合理的职责分离和委托，它实现了灵活且可扩展的路由构建机制。
