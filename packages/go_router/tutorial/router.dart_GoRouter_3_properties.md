# GoRouter 属性

`GoRouter` 类包含多个属性和字段，用于管理路由状态、配置和核心组件。本文档详细说明这些属性的用途和使用方式。

## state 属性

```dart 322:328:lib/src/router.dart
  /// The top [GoRouterState], the state of the route that was
  /// last used in either [GoRouter.go] or [GoRouter.push].
  ///
  /// Accessing this property via GoRouter.of(context).state will not
  /// cause rebuild if the state has changed, consider using
  /// GoRouterState.of(context) instead.
  GoRouterState get state => routerDelegate.state;
```

### 说明

`state` 是一个 getter，返回当前的路由状态。这个状态是在最后一次调用 `GoRouter.go()` 或 `GoRouter.push()` 时使用的路由状态。

### 重要提示

**不会触发重建**：通过 `GoRouter.of(context).state` 访问此属性时，即使状态发生变化，也不会触发 widget 重建。如果需要响应式地监听状态变化，应该使用 `GoRouterState.of(context)` 代替。

## optionURLReflectsImperativeAPIs

```dart 330:348:lib/src/router.dart
  /// Whether the imperative API affects browser URL bar.
  ///
  /// The Imperative APIs refer to [push], [pushReplacement], or [replace].
  ///
  /// If this option is set to true. The URL bar reflects the top-most [GoRoute]
  /// regardless the [RouteBase]s underneath.
  ///
  /// If this option is set to false. The URL bar reflects the [RouteBase]s
  /// in the current state but ignores any [RouteBase]s that are results of
  /// imperative API calls.
  ///
  /// Defaults to false.
  ///
  /// This option is for backward compatibility. It is strongly suggested
  /// against setting this value to true, as the URL of the top-most [GoRoute]
  /// is not always deeplink-able.
  ///
  /// This option only affects web platform.
  static bool optionURLReflectsImperativeAPIs = false;
```

### 说明

这是一个静态布尔属性，控制命令式 API（`push`、`pushReplacement` 或 `replace`）是否影响浏览器地址栏中的 URL。

### 行为说明

- **`true`**：URL 栏反映最顶层的 `GoRoute`，忽略其下的所有 `RouteBase`
- **`false`**（默认）：URL 栏反映当前状态中的 `RouteBase`，但忽略由命令式 API 调用产生的 `RouteBase`

### 重要提示

- **默认值为 `false`**
- **仅影响 Web 平台**：此选项只在 Web 平台上有效
- **向后兼容性**：此选项是为了向后兼容而保留的，**强烈建议不要将其设置为 `true`**，因为最顶层 `GoRoute` 的 URL 并不总是可以作为深层链接使用

## 核心组件属性

### configuration

```dart 350:351:lib/src/router.dart
  /// The route configuration used in go_router.
  late final RouteConfiguration configuration;
```

路由配置对象，管理路由表、名称到路径的映射等核心配置信息。在构造函数中初始化。

### backButtonDispatcher

```dart 353:354:lib/src/router.dart
  @override
  final BackButtonDispatcher backButtonDispatcher;
```

返回按钮分发器，用于处理系统返回按钮事件。实现 `RouterConfig` 接口所需。在构造函数中初始化为 `RootBackButtonDispatcher()`。

### routerDelegate

```dart 356:359:lib/src/router.dart
  /// The router delegate. Provide this to the MaterialApp or CupertinoApp's
  /// `.router()` constructor
  @override
  late final GoRouterDelegate routerDelegate;
```

路由委托对象，这是 `GoRouter` 的核心组件之一。需要将其提供给 `MaterialApp` 或 `CupertinoApp` 的 `.router()` 构造函数。负责构建导航器和处理路由状态变更。

### routeInformationProvider

```dart 361:363:lib/src/router.dart
  /// The route information provider used by [GoRouter].
  @override
  late final GoRouteInformationProvider routeInformationProvider;
```

路由信息提供者，用于管理路由信息的初始化和变更。实现 `RouterConfig` 接口所需。

### routeInformationParser

```dart 365:367:lib/src/router.dart
  /// The route information parser used by [GoRouter].
  @override
  late final GoRouteInformationParser routeInformationParser;
```

路由信息解析器，负责将 URI 转换为路由匹配列表。实现 `RouterConfig` 接口所需。

### observers

```dart 369:370:lib/src/router.dart
  /// The navigator observers used by [GoRouter].
  final List<NavigatorObserver>? observers;
```

导航观察者列表，用于监听导航事件（如路由推送、弹出等）。可以为 null，在构造函数中通过参数传入。

## overridePlatformDefaultLocation

```dart 377:392:lib/src/router.dart
  /// Whether to ignore platform's default initial location when
  /// `initialLocation` is set.
  ///
  /// When set to [true], the [initialLocation] will take
  /// precedence over the platform's default initial location.
  /// This allows developers to control the starting route of the application
  /// independently of the platform.
  ///
  /// Platform's initial location is set when the app opens via a deeplink.
  /// Use [overridePlatformDefaultLocation] only if one wants to override
  /// platform implemented initial location.
  ///
  /// Setting this parameter to [false] (default) will allow the platform's
  /// default initial location to be used even if the `initialLocation` is set.
  /// It's advisable to only set this to [true] if one explicitly wants to.
  final bool overridePlatformDefaultLocation;
```

### 说明

控制当设置了 `initialLocation` 时，是否忽略平台的默认初始位置。

### 行为说明

- **`true`**：`initialLocation` 将优先于平台的默认初始位置，允许开发者独立于平台控制应用的起始路由
- **`false`**（默认）：即使设置了 `initialLocation`，也允许使用平台的默认初始位置

### 使用场景

平台的初始位置通常在应用通过深层链接打开时设置。只有在明确想要覆盖平台实现的初始位置时，才应将此参数设置为 `true`。建议仅在确实需要时使用。

## _routingConfig

```dart 394:394:lib/src/router.dart
  final ValueListenable<RoutingConfig> _routingConfig;
```

### 说明

这是一个私有字段，保存路由配置的可监听对象。在构造函数中初始化，用于支持动态路由配置。

### 路由配置变更处理

```dart 372:375:lib/src/router.dart
  void _handleRoutingConfigChanged() {
    // Reparse is needed to update its builder
    restore(configuration.reparse(routerDelegate.currentConfiguration));
  }
```

当路由配置发生变化时，`_handleRoutingConfigChanged` 方法会被调用。这个方法会重新解析当前配置并恢复路由状态，确保路由配置的变更能够正确应用。

## 属性使用建议

1. **访问路由状态**：优先使用 `GoRouterState.of(context)` 而不是 `GoRouter.of(context).state`，以获得响应式重建
2. **Web 平台 URL 行为**：除非有特殊需求，否则不要修改 `optionURLReflectsImperativeAPIs` 的默认值
3. **初始位置控制**：仅在确实需要覆盖平台默认行为时，才将 `overridePlatformDefaultLocation` 设置为 `true`
4. **核心组件**：`routerDelegate`、`routeInformationProvider` 和 `routeInformationParser` 是 `RouterConfig` 接口的实现，通常不需要直接访问，除非有特殊需求
