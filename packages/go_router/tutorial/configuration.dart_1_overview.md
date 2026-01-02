# RouteConfiguration 类概述

## 概述

`RouteConfiguration` 是 go_router 包中的核心配置类，负责管理和维护应用的路由配置。它封装了路由表的验证、路由匹配、重定向处理等核心功能，是 `GoRouter` 和路由系统之间的桥梁。

```dart 26:27:lib/src/configuration.dart
/// The route configuration for GoRouter configured by the app.
class RouteConfiguration {
```

该类的主要职责包括：

- 管理路由配置表的生命周期和变更监听
- 验证路由配置的正确性（路径格式、参数唯一性等）
- 提供路由匹配功能（根据 URI 查找对应的路由）
- 处理路由重定向逻辑
- 管理命名路由的映射关系
- 构建路由状态对象（`GoRouterState`）

## 构造函数

```dart 28:37:lib/src/configuration.dart
/// Constructs a [RouteConfiguration].
RouteConfiguration(
  this._routingConfig, {
  required this.navigatorKey,
  this.extraCodec,
  this.router,
}) {
  _onRoutingTableChanged();
  _routingConfig.addListener(_onRoutingTableChanged);
}
```

构造函数接收以下参数：

- **`_routingConfig`**（位置参数）：`ValueListenable<RoutingConfig>` 类型，表示可监听的路由配置。通过 `ValueListenable`，可以在运行时动态更新路由配置，而不需要重新创建 `RouteConfiguration` 实例。
- **`navigatorKey`**（必需命名参数）：`GlobalKey<NavigatorState>` 类型，根导航器的全局键。用于标识和管理应用的根导航器。
- **`extraCodec`**（可选命名参数）：`Codec<Object?, Object?>?` 类型，用于序列化和反序列化导航时传递的 `extra` 数据。当需要在导航时传递复杂对象（如自定义类实例）时，可以通过提供 codec 来实现序列化支持。
- **`router`**（可选命名参数）：`GoRouter?` 类型，拥有此配置的 `GoRouter` 实例。主要用于在重定向等场景中提供路由器的访问。

构造函数执行以下初始化操作：

1. **立即执行路由表变更处理**：调用 `_onRoutingTableChanged()` 进行初始的路由表验证和缓存构建。
2. **监听路由配置变更**：通过 `_routingConfig.addListener(_onRoutingTableChanged)` 注册监听器，当路由配置发生变化时自动重新验证和更新。

这种设计使得 `RouteConfiguration` 能够响应动态路由配置的变更，实现热更新路由表的功能。

## 核心属性

### 路由配置

```dart 244:245:lib/src/configuration.dart
/// The routing table.
final ValueListenable<RoutingConfig> _routingConfig;
```

`_routingConfig` 是私有的路由配置对象，类型为 `ValueListenable<RoutingConfig>`。它提供了响应式的路由配置管理能力，当配置发生变化时，会自动触发 `_onRoutingTableChanged` 回调。

### 导航器键

```dart 271:272:lib/src/configuration.dart
/// The global key for top level navigator.
final GlobalKey<NavigatorState> navigatorKey;
```

`navigatorKey` 是根导航器的全局键，用于标识和管理应用的顶层 `Navigator`。在 Flutter 中，`GlobalKey` 提供了一种跨 widget 树访问特定 widget 状态的方式，这里用于访问根导航器实例。

### 额外数据编解码器

```dart 274:287:lib/src/configuration.dart
/// The codec used to encode and decode extra into a serializable format.
///
/// When navigating using [GoRouter.go] or [GoRouter.push], one can provide
/// an `extra` parameter along with it. If the extra contains complex data,
/// consider provide a codec for serializing and deserializing the extra data.
///
/// See also:
///  * [Navigation](https://pub.dev/documentation/go_router/latest/topics/Navigation-topic.html)
///    topic.
///  * [extra_codec](https://github.com/flutter/packages/blob/main/packages/go_router/example/lib/extra_codec.dart)
///    example.
///  * [topOnEnter] for navigation interception.
///  * [topRedirect] for legacy redirections.
final Codec<Object?, Object?>? extraCodec;
```

`extraCodec` 是一个可选的编解码器，用于将导航时传递的 `extra` 对象序列化为可持久化的格式（如 JSON）。这对于需要在应用重启后恢复导航状态，或者需要在不同页面间传递复杂数据结构的场景很有用。

### 路由器引用

```dart 289:292:lib/src/configuration.dart
/// The GoRouter instance that owns this configuration.
///
/// This is used to provide access to the router during redirects.
final GoRouter? router;
```

`router` 是对拥有此配置的 `GoRouter` 实例的可选引用。主要用于在重定向回调中提供对路由器的访问，使得重定向逻辑可以执行一些需要路由器上下文的操作。

### 命名路由缓存

```dart 294:294:lib/src/configuration.dart
final Map<String, _NamedPath> _nameToPath = <String, _NamedPath>{};
```

`_nameToPath` 是一个私有映射，用于缓存路由名称到路径的映射关系。`_NamedPath` 是一个记录类型（record type），包含路径字符串和是否大小写敏感的标志。这个缓存提高了通过名称查找路由路径的性能。

## 属性访问器

### 路由列表

```dart 247:248:lib/src/configuration.dart
/// The list of top level routes used by [GoRouterDelegate].
List<RouteBase> get routes => _routingConfig.value.routes;
```

`routes` getter 返回顶层路由列表。它直接访问 `_routingConfig.value.routes`，提供对当前路由配置中定义的所有顶层路由的访问。

### 顶层重定向

```dart 250:253:lib/src/configuration.dart
/// Legacy top level page redirect.
///
/// This is handled via [applyTopLegacyRedirect] and runs at most once per navigation.
GoRouterRedirect get topRedirect => _routingConfig.value.redirect;
```

`topRedirect` getter 返回顶层重定向回调函数。这是一个遗留的重定向机制，在每次导航时最多执行一次，在路由级重定向之前运行。详细的处理逻辑在 `applyTopLegacyRedirect` 方法中实现。

### 顶层进入回调

```dart 255:256:lib/src/configuration.dart
/// Top level page on enter.
OnEnter? get topOnEnter => _routingConfig.value.onEnter;
```

`topOnEnter` getter 返回顶层进入回调。这个回调在路由处理之前执行，可以用来拦截导航、进行权限检查等操作。

### 重定向限制

```dart 258:259:lib/src/configuration.dart
/// The limit for the number of consecutive redirects.
int get redirectLimit => _routingConfig.value.redirectLimit;
```

`redirectLimit` getter 返回允许的最大连续重定向次数。这是为了防止重定向循环而设置的安全限制，默认值通常是 5 次。

## URI 规范化

```dart 261:269:lib/src/configuration.dart
/// Normalizes a URI by ensuring it has a valid path and removing trailing slashes.
static Uri normalizeUri(Uri uri) {
  if (uri.hasEmptyPath) {
    return uri.replace(path: '/');
  } else if (uri.path.length > 1 && uri.path.endsWith('/')) {
    return uri.replace(path: uri.path.substring(0, uri.path.length - 1));
  }
  return uri;
}
```

`normalizeUri` 是一个静态方法，用于规范化 URI 路径。它执行以下操作：

1. **处理空路径**：如果 URI 的路径为空，将其替换为 `'/'`。
2. **移除尾随斜杠**：如果路径长度大于 1 且以 `/` 结尾（例如 `/users/`），则移除尾随的斜杠（变为 `/users`）。注意，根路径 `/` 不会被处理，因为它本身就是单个字符。

这个方法确保了路由匹配时路径的一致性，避免了因为路径格式不同（如 `/users` 和 `/users/`）而导致的匹配失败。

## 路由表变更处理

```dart 205:224:lib/src/configuration.dart
void _onRoutingTableChanged() {
  final RoutingConfig routingTable = _routingConfig.value;
  assert(_debugCheckPath(routingTable.routes, true));
  assert(
    _debugVerifyNoDuplicatePathParameter(
      routingTable.routes,
      <String, GoRoute>{},
    ),
  );
  assert(
    _debugCheckParentNavigatorKeys(
      routingTable.routes,
      <GlobalKey<NavigatorState>>[navigatorKey],
    ),
  );
  assert(_debugCheckStatefulShellBranchDefaultLocations(routingTable.routes));
  _nameToPath.clear();
  _cacheNameToPath('', routingTable.routes);
  log(debugKnownRoutes());
}
```

`_onRoutingTableChanged` 是一个私有方法，在路由配置发生变化时被调用。它执行以下操作：

1. **获取当前路由配置**：从 `_routingConfig.value` 获取最新的路由配置。
2. **执行调试检查**（仅在 debug 模式下）：
   - `_debugCheckPath`：检查路径格式（如不能以 `/` 结尾，除非是根路径）
   - `_debugVerifyNoDuplicatePathParameter`：验证路径参数没有重复
   - `_debugCheckParentNavigatorKeys`：验证父导航器键的有效性
   - `_debugCheckStatefulShellBranchDefaultLocations`：验证 StatefulShellBranch 的默认位置配置
3. **清空并重建命名路由缓存**：清空 `_nameToPath` 映射，然后调用 `_cacheNameToPath` 重新构建缓存。
4. **记录已知路由日志**：调用 `debugKnownRoutes()` 获取路由列表的字符串表示，并通过 `log` 记录日志（仅在启用调试日志时输出）。

这个方法确保了路由配置的一致性和正确性，并在配置变更时及时更新内部缓存。

## GoRouterState 构建

```dart 226:242:lib/src/configuration.dart
/// Builds a [GoRouterState] suitable for top level callback such as
/// `GoRouter.redirect` or `GoRouter.onException`.
GoRouterState buildTopLevelGoRouterState(RouteMatchList matchList) {
  return GoRouterState(
    this,
    uri: matchList.uri,
    // No name available at the top level trim the query params off the
    // sub-location to match route.redirect
    fullPath: matchList.fullPath,
    pathParameters: matchList.pathParameters,
    matchedLocation: matchList.uri.path,
    extra: matchList.extra,
    pageKey: const ValueKey<String>('topLevel'),
    topRoute: matchList.lastOrNull?.route,
    error: matchList.error,
  );
}
```

`buildTopLevelGoRouterState` 方法用于构建适用于顶层回调（如 `GoRouter.redirect` 或 `GoRouter.onException`）的 `GoRouterState` 对象。

方法参数：

- **`matchList`**：`RouteMatchList` 类型，包含路由匹配结果的列表。

返回的 `GoRouterState` 对象包含以下信息：

- **`uri`**：完整的 URI 对象（包含路径、查询参数、片段等）
- **`fullPath`**：完整的路径（从路由匹配列表中获取）
- **`pathParameters`**：路径参数的映射（如 `/users/:id` 中的 `id`）
- **`matchedLocation`**：匹配的位置路径（从 URI 的 path 属性获取）
- **`extra`**：额外的对象数据
- **`pageKey`**：固定为 `ValueKey<String>('topLevel')`，标识这是顶层状态
- **`topRoute`**：匹配列表中的最后一个路由（如果有）
- **`error`**：错误信息（如果有）

注意，在顶层状态下，`name` 参数为 `null`，因为顶层回调不需要具体的路由名称信息。

## 总结

`RouteConfiguration` 类作为 go_router 的核心配置类，提供了：

- **响应式配置管理**：通过 `ValueListenable` 支持动态路由配置更新
- **配置验证**：在配置变更时自动执行多项验证检查
- **路由匹配基础**：为路由匹配功能提供配置数据
- **状态构建支持**：为顶层回调构建路由状态对象
- **性能优化**：通过缓存机制提高命名路由查找效率

这些功能共同构成了 go_router 路由系统的配置和管理基础。
