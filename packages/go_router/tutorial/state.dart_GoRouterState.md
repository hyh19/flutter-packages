# GoRouterState 类详解

## 概述

`GoRouterState` 是 go_router 包中用于表示路由状态的核心类。它包含了当前 URI 的解析结果，提供了路由匹配后的所有相关信息，使得在路由构建过程中可以访问路径参数、查询参数、错误信息等。

```dart 12:16:lib/src/state.dart
/// The route state during routing.
///
/// The state contains parsed artifacts of the current URI.
@immutable
class GoRouterState {
```

该类被标记为 `@immutable`，表明其实例是不可变的，这对于 Flutter 的响应式系统很重要，可以安全地用于状态比较和重建优化。

## 构造函数与属性

### 构造函数

```dart 17:30:lib/src/state.dart
/// Default constructor for creating route state during routing.
const GoRouterState(
  this._configuration, {
  required this.uri,
  required this.matchedLocation,
  this.name,
  this.path,
  required this.fullPath,
  required this.pathParameters,
  this.extra,
  this.error,
  required this.pageKey,
  this.topRoute,
});
```

构造函数使用了 `const` 关键字，这意味着所有的属性都必须在编译时确定，进一步保证了不可变性。构造函数接收以下参数：

- `_configuration`：路由配置对象（`RouteConfiguration`），用于内部操作
- `uri`：完整的 URI 对象（必需）
- `matchedLocation`：已匹配的位置路径（必需）
- `name`：路由名称（可选）
- `path`：路由路径模板（可选）
- `fullPath`：完整的路径（必需）
- `pathParameters`：路径参数映射（必需）
- `extra`：额外的对象数据（可选）
- `error`：错误信息（可选）
- `pageKey`：页面唯一键（必需）
- `topRoute`：顶层路由（可选）

### 核心属性详解

#### URI 相关属性

```dart 33:34:lib/src/state.dart
/// The full uri of the route, e.g. /family/f2/person/p1?filter=name#fragment
final Uri uri;
```

`uri` 属性包含了完整的 URI 信息，包括路径、查询参数和片段标识符。例如，`/family/f2/person/p1?filter=name#fragment` 这样的 URI 会被完整解析。

```dart 36:44:lib/src/state.dart
/// The matched location until this point.
///
/// For example:
///
/// location = /family/f2/person/p1
/// route = GoRoute('/family/:id')
///
/// matchedLocation = /family/f2
final String matchedLocation;
```

`matchedLocation` 表示到当前路由为止已经匹配的位置。这在嵌套路由中特别有用，可以知道当前路由在整个路由栈中的位置。

#### 路由信息属性

```dart 46:49:lib/src/state.dart
/// The optional name of the route associated with this app.
///
/// This can be null for GoRouterState pass into top level redirect.
final String? name;
```

`name` 是路由的可选名称。当路由通过名称定义时，可以通过这个属性访问。在顶级重定向中，此值可能为 `null`。

```dart 51:54:lib/src/state.dart
/// The path of the route associated with this app. e.g. family/:fid
///
/// This can be null for GoRouterState pass into top level redirect.
final String? path;
```

`path` 是路由的路径模板，例如 `family/:fid`。这表示路由的模式定义，其中 `:fid` 是路径参数的占位符。

```dart 56:61:lib/src/state.dart
/// The full path to this sub-route, e.g. /family/:fid
///
/// For top level redirect, this is the entire path that matches the location.
/// It can be empty if go router can't find a match. In that case, the [error]
/// contains more information.
final String? fullPath;
```

`fullPath` 是到当前子路由的完整路径。对于顶级重定向，这是匹配位置的整个路径。如果找不到匹配，此值可能为空，此时 `error` 属性会包含更多信息。

#### 参数与数据属性

```dart 63:64:lib/src/state.dart
/// The parameters for this match, e.g. {'fid': 'f2'}
final Map<String, String> pathParameters;
```

`pathParameters` 是一个映射，包含了从路径中提取的参数。例如，如果路径模板是 `/family/:fid`，实际路径是 `/family/f2`，那么 `pathParameters` 就是 `{'fid': 'f2'}`。

```dart 66:67:lib/src/state.dart
/// An extra object to pass along with the navigation.
final Object? extra;
```

`extra` 允许传递任意对象数据，这在需要传递复杂数据而不仅仅是字符串参数时非常有用。

```dart 69:70:lib/src/state.dart
/// The error associated with this sub-route.
final GoException? error;
```

`error` 包含了与当前子路由相关的错误信息。当路由匹配失败或发生其他问题时，此属性会包含 `GoException` 对象。

#### 页面标识属性

```dart 72:77:lib/src/state.dart
/// A unique string key for this sub-route.
/// E.g.
/// ```dart
/// ValueKey('/family/:fid')
/// ```
final ValueKey<String> pageKey;
```

`pageKey` 是一个唯一字符串键，用于标识当前子路由。Flutter 使用这个键来决定是否需要重建页面，以及如何管理页面状态。

```dart 79:84:lib/src/state.dart
/// The current matched top route associated with this state.
///
/// If this state represents a [ShellRoute], the top [GoRoute] will be the current
/// matched location associated with the [ShellRoute]. This allows the [ShellRoute]'s
/// associated GoRouterState to be uniquely identified using [GoRoute.name]
final GoRoute? topRoute;
```

`topRoute` 表示与当前状态关联的顶层路由。如果当前状态代表一个 `ShellRoute`，这个属性会指向与 `ShellRoute` 关联的当前匹配位置对应的 `GoRoute`。

## 静态方法：从上下文获取状态

`GoRouterState.of(BuildContext context)` 是一个非常重要的静态方法，它允许从 `BuildContext` 中获取当前的 `GoRouterState`。

```dart 86:148:lib/src/state.dart
/// Gets the [GoRouterState] from context.
///
/// The returned [GoRouterState] will depends on which [GoRoute] or
/// [ShellRoute] the input `context` is in.
///
/// This method only supports [GoRoute] and [ShellRoute] that generate
/// [ModalRoute]s. This is typically the case if one uses [GoRoute.builder],
/// [ShellRoute.builder], [CupertinoPage], [MaterialPage],
/// [CustomTransitionPage], or [NoTransitionPage].
///
/// This method is fine to be called during [GoRoute.builder] or
/// [ShellRoute.builder].
///
/// This method cannot be called during [GoRoute.pageBuilder] or
/// [ShellRoute.pageBuilder] since there is no [GoRouterState] to be
/// associated with yet.
///
/// To access GoRouterState from a widget.
///
/// ```dart
/// GoRoute(
///   path: '/:id'
///   builder: (_, __) => MyWidget(),
/// );
///
/// class MyWidget extends StatelessWidget {
///   @override
///   Widget build(BuildContext context) {
///     return Text('${GoRouterState.of(context).pathParameters['id']}');
///   }
/// }
/// ```
static GoRouterState of(BuildContext context) {
  ModalRoute<Object?>? route;
  GoRouterStateRegistryScope? scope;
  while (true) {
    route = ModalRoute.of(context);
    if (route == null) {
      throw _noGoRouterStateError;
    }
    final RouteSettings settings = route.settings;
    if (settings is Page<Object?>) {
      scope = context
          .dependOnInheritedWidgetOfExactType<GoRouterStateRegistryScope>();
      if (scope == null) {
        throw _noGoRouterStateError;
      }
      final GoRouterState? state = scope.notifier!
          ._createPageRouteAssociation(
            route.settings as Page<Object?>,
            route,
          );
      if (state != null) {
        return state;
      }
    }
    final NavigatorState? state = Navigator.maybeOf(context);
    if (state == null) {
      throw _noGoRouterStateError;
    }
    context = state.context;
  }
}
```

### 工作原理

该方法通过向上遍历 widget 树来查找 `GoRouterState`：

1. **查找 ModalRoute**：首先尝试从当前上下文获取 `ModalRoute`
2. **检查 Page 对象**：如果路由设置是 `Page` 对象，则查找 `GoRouterStateRegistryScope`
3. **创建关联**：通过注册表创建页面路由关联，获取对应的 `GoRouterState`
4. **向上遍历**：如果当前层级找不到，则向上查找父级 Navigator 的上下文，继续搜索

### 使用限制

- **只支持生成 ModalRoute 的路由**：仅在使用 `GoRoute.builder`、`ShellRoute.builder` 或特定页面类型（`CupertinoPage`、`MaterialPage` 等）时可用
- **不能在 pageBuilder 中调用**：在 `GoRoute.pageBuilder` 或 `ShellRoute.pageBuilder` 中无法使用，因为此时还没有关联的 `GoRouterState`
- **可以在 builder 中调用**：在 `GoRoute.builder` 或 `ShellRoute.builder` 中可以安全调用

### 错误处理

```dart 150:154:lib/src/state.dart
static GoError get _noGoRouterStateError => GoError(
  'There is no GoRouterState above the current context. '
  'This method should only be called under the sub tree of a '
  'RouteBase.builder.',
);
```

如果在 widget 树中找不到 `GoRouterState`，会抛出 `GoError`，提示应该在 `RouteBase.builder` 的子树下调用此方法。

## 命名路由定位方法

```dart 156:172:lib/src/state.dart
/// Get a location from route name and parameters.
/// This is useful for redirecting to a named location.
String namedLocation(
  String name, {
  Map<String, String> pathParameters = const <String, String>{},
  Map<String, String> queryParameters = const <String, String>{},
  String? fragment,
}) {
  // Generate base location using configuration, with optional path and query parameters
  // Then conditionally append fragment if it exists and is not empty
  return _configuration.namedLocation(
    name,
    pathParameters: pathParameters,
    queryParameters: queryParameters,
    fragment: fragment,
  );
}
```

`namedLocation` 方法根据路由名称和参数生成完整的定位字符串。这对于重定向到命名路由非常有用。方法支持：

- `name`：路由名称
- `pathParameters`：路径参数（默认为空映射）
- `queryParameters`：查询参数（默认为空映射）
- `fragment`：URI 片段（可选）

该方法委托给内部的 `_configuration.namedLocation` 方法来完成实际的位置生成工作。

## 相等性与哈希码

由于 `GoRouterState` 是不可变的，它重写了 `==` 操作符和 `hashCode` getter，使得可以正确地进行状态比较。

```dart 174:186:lib/src/state.dart
@override
bool operator ==(Object other) {
  return other is GoRouterState &&
      other.uri == uri &&
      other.matchedLocation == matchedLocation &&
      other.name == name &&
      other.path == path &&
      other.fullPath == fullPath &&
      other.pathParameters == pathParameters &&
      other.extra == extra &&
      other.error == error &&
      other.pageKey == pageKey;
}
```

相等性比较检查了除 `_configuration` 和 `topRoute` 之外的所有公共属性。`_configuration` 是内部实现细节，而 `topRoute` 可能是动态的，因此不参与比较。

```dart 188:199:lib/src/state.dart
@override
int get hashCode => Object.hash(
  uri,
  matchedLocation,
  name,
  path,
  fullPath,
  pathParameters,
  extra,
  error,
  pageKey,
);
```

哈希码使用了 `Object.hash` 方法，包含了参与相等性比较的所有属性。这确保了相等的对象具有相同的哈希码，满足了 `Object.hashCode` 的约定。

## 使用场景

### 1. 在路由构建器中访问状态

```dart
GoRoute(
  path: '/users/:userId',
  builder: (context, state) {
    final userId = state.pathParameters['userId']!;
    final filter = state.uri.queryParameters['filter'];
    return UserScreen(userId: userId, filter: filter);
  },
)
```

### 2. 在 Widget 中获取路由状态

```dart
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final state = GoRouterState.of(context);
    return Text('Current path: ${state.fullPath}');
  }
}
```

### 3. 处理路由错误

```dart
GoRoute(
  path: '/users/:userId',
  builder: (context, state) {
    if (state.error != null) {
      return ErrorScreen(error: state.error!);
    }
    return UserScreen(userId: state.pathParameters['userId']!);
  },
)
```

### 4. 重定向到命名路由

```dart
GoRoute(
  path: '/home',
  redirect: (context, state) {
    if (user.isLoggedIn) {
      return state.namedLocation('dashboard');
    }
    return null;
  },
)
```

## 设计模式与架构

`GoRouterState` 体现了以下设计模式和架构思想：

1. **不可变对象模式**：通过 `@immutable` 和 `const` 构造函数确保状态不可变，有利于状态管理和性能优化
2. **值对象模式**：通过重写 `==` 和 `hashCode`，使其可以作为值对象使用
3. **上下文依赖注入**：通过 `GoRouterState.of(context)` 实现从 widget 树中获取状态，符合 Flutter 的设计哲学
4. **委托模式**：`namedLocation` 方法委托给 `RouteConfiguration` 完成实际工作

## 总结

`GoRouterState` 是 go_router 包中的核心状态类，它封装了路由匹配后的所有信息，提供了访问路径参数、查询参数、错误信息等的统一接口。通过 `GoRouterState.of(context)` 方法，开发者可以在任何 widget 中轻松获取当前路由状态，这使得基于状态的路由处理变得简单而优雅。
