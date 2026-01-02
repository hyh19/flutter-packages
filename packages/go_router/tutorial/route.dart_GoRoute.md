# GoRoute 类详解

## 概述

`GoRoute` 是 go_router 包中用于定义路由的核心类。它继承自 `RouteBase`，用于创建一个在视觉上显示在匹配的父路由之上的路由，通过 Flutter 的 `Navigator` 进行管理。

```dart 255:271:lib/src/route.dart
/// A route that is displayed visually above the matching parent route using the
/// [Navigator].
///
/// The widget returned by [builder] is wrapped in [Page] and provided to the
/// root Navigator, the nearest ShellRoute ancestor's Navigator, or the
/// Navigator with a matching [parentNavigatorKey].
///
/// The Page depends on the application type: [MaterialPage] for
/// [MaterialApp], [CupertinoPage] for [CupertinoApp], or
/// [NoTransitionPage] for [WidgetsApp].
///
/// {@category Get started}
/// {@category Configuration}
/// {@category Transition animations}
/// {@category Named routes}
/// {@category Redirection}
class GoRoute extends RouteBase {
```

`GoRoute` 返回的 widget 会被包装成 `Page` 对象，并传递给：

- 根 Navigator
- 最近的 ShellRoute 祖先的 Navigator
- 匹配 `parentNavigatorKey` 的 Navigator

根据应用类型，会使用不同的 Page 类型：

- `MaterialPage` 用于 `MaterialApp`
- `CupertinoPage` 用于 `CupertinoApp`
- `NoTransitionPage` 用于 `WidgetsApp`

## 构造函数

```dart 272:302:lib/src/route.dart
  /// Constructs a [GoRoute].
  /// - [path] and [name] cannot be empty strings.
  /// - One of either [builder] or [pageBuilder] must be provided.
  GoRoute({
    required this.path,
    this.name,
    this.builder,
    this.pageBuilder,
    super.parentNavigatorKey,
    super.redirect,
    this.onExit,
    this.caseSensitive = true,
    super.routes = const <RouteBase>[],
  }) : assert(path.isNotEmpty, 'GoRoute path cannot be empty'),
       assert(name == null || name.isNotEmpty, 'GoRoute name cannot be empty'),
       assert(
         pageBuilder != null || builder != null || redirect != null,
         'builder, pageBuilder, or redirect must be provided',
       ),
       assert(
         onExit == null || pageBuilder != null || builder != null,
         'if onExit is provided, one of pageBuilder or builder must be provided',
       ),
       super._() {
    // cache the path regexp and parameters
    _pathRE = patternToRegExp(
      path,
      pathParameters,
      caseSensitive: caseSensitive,
    );
  }
```

### 构造参数说明

- **`path`**（必需）：路由的路径字符串，不能为空
- **`name`**（可选）：路由的唯一名称，用于命名路由导航
- **`builder`**（可选）：用于构建 widget 的函数
- **`pageBuilder`**（可选）：用于构建 `Page` 对象的函数
- **`parentNavigatorKey`**（继承自父类）：父 Navigator 的 GlobalKey
- **`redirect`**（继承自父类）：重定向函数
- **`onExit`**（可选）：路由退出时的回调函数
- **`caseSensitive`**（默认 `true`）：路径匹配是否区分大小写
- **`routes`**（继承自父类）：子路由列表

### 构造验证

构造函数包含以下断言：

1. **路径非空验证**：`path` 不能为空字符串
2. **名称非空验证**：如果提供了 `name`，则不能为空字符串
3. **构建器验证**：必须提供 `builder`、`pageBuilder` 或 `redirect` 中的至少一个
4. **退出回调验证**：如果提供了 `onExit`，则必须同时提供 `builder` 或 `pageBuilder`

### 初始化逻辑

在构造函数体中，会调用 `patternToRegExp` 函数将路径模式转换为正则表达式，并缓存到 `_pathRE` 中。这个过程会：

- 解析路径参数（如 `:id`）
- 将参数名存储到 `pathParameters` 列表中
- 根据 `caseSensitive` 参数生成相应的正则表达式

## 核心属性

### redirectOnly

```dart 304:307:lib/src/route.dart
  /// Whether this [GoRoute] only redirects to another route.
  ///
  /// If this is true, this route must redirect location other than itself.
  bool get redirectOnly => pageBuilder == null && builder == null;
```

这是一个计算属性，用于判断当前路由是否仅用于重定向。当 `pageBuilder` 和 `builder` 都为 `null` 时，返回 `true`，表示这是一个纯重定向路由。

### name

```dart 309:344:lib/src/route.dart
  /// Optional name of the route.
  ///
  /// If used, a unique string name must be provided and it can not be empty.
  ///
  /// This is used in [GoRouter.namedLocation] and its related API. This
  /// property can be used to navigate to this route without knowing exact the
  /// URI of it.
  ///
  /// Typical usage is as follows:
  ///
  /// ```dart
  /// GoRoute(
  ///   name: 'home',
  ///   path: '/',
  ///   builder: (BuildContext context, GoRouterState state) =>
  ///       HomeScreen(),
  ///   routes: <GoRoute>[
  ///     GoRoute(
  ///       name: 'family',
  ///       path: 'family/:fid',
  ///       builder: (BuildContext context, GoRouterState state) =>
  ///           FamilyScreen(),
  ///     ),
  ///   ],
  /// );
  ///
  /// context.go(
  ///   context.namedLocation('family'),
  ///   pathParameters: <String, String>{'fid': 123},
  ///   queryParameters: <String, String>{'qid': 'quid'},
  /// );
  /// ```
  ///
  /// See the [named routes example](https://github.com/flutter/packages/blob/main/packages/go_router/example/lib/named_routes.dart)
  /// for a complete runnable app.
  final String? name;
```

路由的可选名称，用于命名路由导航。通过名称可以导航到路由，而无需知道确切的 URI。这在路径结构复杂或需要动态生成路径时特别有用。

### path

```dart 346:369:lib/src/route.dart
  /// The path of this go route.
  ///
  /// For example:
  /// ```dart
  /// GoRoute(
  ///   path: '/',
  ///   pageBuilder: (BuildContext context, GoRouterState state) => MaterialPage<void>(
  ///     key: state.pageKey,
  ///     child: HomePage(families: Families.data),
  ///   ),
  /// ),
  /// ```
  ///
  /// The path also support path parameters. For a path: `/family/:fid`, it
  /// matches all URIs start with `/family/...`, e.g. `/family/123`,
  /// `/family/456` and etc. The parameter values are stored in [GoRouterState]
  /// that are passed into [pageBuilder] and [builder].
  ///
  /// The query parameter are also capture during the route parsing and stored
  /// in [GoRouterState].
  ///
  /// See [Query parameters and path parameters](https://github.com/flutter/packages/blob/main/packages/go_router/example/lib/path_and_query_parameters.dart)
  /// to learn more about parameters.
  final String path;
```

路由的路径字符串，支持路径参数。例如 `/family/:fid` 可以匹配 `/family/123`、`/family/456` 等。参数值会存储在 `GoRouterState` 中，并传递给 `pageBuilder` 和 `builder`。

查询参数也会在路由解析时被捕获并存储在 `GoRouterState` 中。

### pageBuilder

```dart 371:386:lib/src/route.dart
  /// A page builder for this route.
  ///
  /// Typically a MaterialPage, as in:
  /// ```dart
  /// GoRoute(
  ///   path: '/',
  ///   pageBuilder: (BuildContext context, GoRouterState state) => MaterialPage<void>(
  ///     key: state.pageKey,
  ///     child: HomePage(families: Families.data),
  ///   ),
  /// ),
  /// ```
  ///
  /// You can also use CupertinoPage, and for a custom page builder to use
  /// custom page transitions, you can use [CustomTransitionPage].
  final GoRouterPageBuilder? pageBuilder;
```

用于构建 `Page` 对象的函数。通常使用 `MaterialPage`，也可以使用 `CupertinoPage`。如果需要自定义页面转场动画，可以使用 `CustomTransitionPage`。

### builder

```dart 388:402:lib/src/route.dart
  /// A custom builder for this route.
  ///
  /// For example:
  /// ```dart
  /// GoRoute(
  ///   path: '/',
  ///   builder: (BuildContext context, GoRouterState state) => FamilyPage(
  ///     families: Families.family(
  ///       state.pathParameters['id'],
  ///     ),
  ///   ),
  /// ),
  /// ```
  ///
  final GoRouterWidgetBuilder? builder;
```

用于构建 widget 的自定义构建器。与 `pageBuilder` 不同，`builder` 直接返回 widget，go_router 会自动将其包装成 `Page` 对象。

### onExit

```dart 404:450:lib/src/route.dart
  /// Called when this route is removed from GoRouter's route history.
  ///
  /// Some example this callback may be called:
  ///  * This route is removed as the result of [GoRouter.pop].
  ///  * This route is no longer in the route history after a [GoRouter.go].
  ///
  /// This method can be useful it one wants to launch a dialog for user to
  /// confirm if they want to exit the screen.
  ///
  /// ```dart
  /// final GoRouter _router = GoRouter(
  ///   routes: <GoRoute>[
  ///     GoRoute(
  ///       path: '/',
  ///       onExit: (BuildContext context) => showDialog<bool>(
  ///         context: context,
  ///         builder: (BuildContext context) {
  ///           return AlertDialog(
  ///             title: const Text('Do you want to exit this page?'),
  ///             actions: <Widget>[
  ///               TextButton(
  ///                 style: TextButton.styleFrom(
  ///                   textStyle: Theme.of(context).textTheme.labelLarge,
  ///                 ),
  ///                 child: const Text('Go Back'),
  ///                 onPressed: () {
  ///                   Navigator.of(context).pop(false);
  ///                 },
  ///               ),
  ///               TextButton(
  ///                 style: TextButton.styleFrom(
  ///                   textStyle: Theme.of(context).textTheme.labelLarge,
  ///                 ),
  ///                 child: const Text('Confirm'),
  ///                 onPressed: () {
  ///                   Navigator.of(context).pop(true);
  ///                 },
  ///               ),
  ///             ],
  ///           );
  ///         },
  ///       ),
  ///     ),
  ///   ],
  /// );
  /// ```
  final ExitCallback? onExit;
```

当路由从 GoRouter 的路由历史中移除时调用的回调函数。可能触发的情况包括：

- 通过 `GoRouter.pop` 移除路由
- 通过 `GoRouter.go` 导航后，路由不再在路由历史中

这个回调非常有用，可以用于显示确认对话框，询问用户是否真的要退出当前页面。如果回调返回 `false` 或返回的 Future 解析为 `false`，则退出操作会被中止。

### caseSensitive

```dart 452:461:lib/src/route.dart
  /// Determines whether the route matching is case sensitive.
  ///
  /// When `true`, the path must match the specified case. For example,
  /// a [GoRoute] with `path: '/family/:fid'` will not match `/FaMiLy/f2`.
  ///
  /// When `false`, the path matching is case insensitive.  The route
  /// with `path: '/family/:fid'` will match `/FaMiLy/f2`.
  ///
  /// Defaults to `true`.
  final bool caseSensitive;
```

决定路由匹配是否区分大小写。默认为 `true`。

- 当为 `true` 时：路径必须完全匹配大小写，例如 `/family/:fid` 不会匹配 `/FaMiLy/f2`
- 当为 `false` 时：路径匹配不区分大小写，例如 `/family/:fid` 可以匹配 `/FaMiLy/f2`

## 核心方法

### matchPatternAsPrefix

```dart 463:468:lib/src/route.dart
  // TODO(chunhtai): move all regex related help methods to path_utils.dart.
  /// Match this route against a location.
  RegExpMatch? matchPatternAsPrefix(String loc) {
    return _pathRE.matchAsPrefix('/$loc') as RegExpMatch? ??
        _pathRE.matchAsPrefix(loc) as RegExpMatch?;
  }
```

将路由模式与给定的位置进行匹配。这个方法使用缓存的 `_pathRE` 正则表达式进行匹配。

匹配逻辑：

1. 首先尝试在位置前添加 `/` 进行匹配（`'/$loc'`）
2. 如果失败，则直接匹配原始位置（`loc`）

这种双重匹配策略可以处理带或不带前导斜杠的路径。

### extractPathParams

```dart 470:472:lib/src/route.dart
  /// Extract the path parameters from a match.
  Map<String, String> extractPathParams(RegExpMatch match) =>
      extractPathParameters(pathParameters, match);
```

从匹配结果中提取路径参数。这个方法调用了 `extractPathParameters` 工具函数，将 `RegExpMatch` 中的命名组提取为 `Map<String, String>` 格式。

### pathParameters

```dart 474:478:lib/src/route.dart
  /// The path parameters in this route.
  // TODO(loic-sharma): Remove meta library prefix.
  // https://github.com/flutter/flutter/issues/171410
  @meta.internal
  final List<String> pathParameters = <String>[];
```

路由中的路径参数名称列表。这是一个内部属性（标记为 `@meta.internal`），在构造函数初始化时通过 `patternToRegExp` 函数填充。

例如，对于路径 `/user/:id/book/:bookId`，`pathParameters` 会包含 `['id', 'bookId']`。

### debugFillProperties

```dart 480:488:lib/src/route.dart
  @override
  void debugFillProperties(DiagnosticPropertiesBuilder properties) {
    super.debugFillProperties(properties);
    properties.add(StringProperty('name', name));
    properties.add(StringProperty('path', path));
    properties.add(
      FlagProperty('redirect', value: redirectOnly, ifTrue: 'Redirect Only'),
    );
  }
```

用于调试的属性填充方法。在 Flutter 的调试工具中，这个方法会显示路由的名称、路径以及是否为仅重定向路由。

### _pathRE

```dart 490:490:lib/src/route.dart
  late final RegExp _pathRE;
```

缓存的路径正则表达式。这是一个延迟初始化的最终字段，在构造函数中通过 `patternToRegExp` 函数初始化。使用 `late final` 确保它只被初始化一次，并且是线程安全的。

## 路径匹配机制

`GoRoute` 使用正则表达式进行路径匹配。在构造函数中，路径模式（如 `/user/:id`）会被转换为正则表达式：

1. **参数提取**：路径中的 `:paramName` 会被识别为路径参数
2. **正则生成**：参数会被替换为正则表达式组，例如 `:id` 会被替换为 `(?<id>[^/]+)`
3. **缓存存储**：生成的正则表达式存储在 `_pathRE` 中，参数名存储在 `pathParameters` 中

匹配时，`matchPatternAsPrefix` 方法使用这个正则表达式来检查给定的位置是否匹配当前路由。

## 使用场景

### 基本路由定义

```dart
GoRoute(
  path: '/',
  builder: (context, state) => HomeScreen(),
)
```

### 带参数的路由

```dart
GoRoute(
  path: '/user/:id',
  builder: (context, state) {
    final userId = state.pathParameters['id'];
    return UserScreen(userId: userId);
  },
)
```

### 命名路由

```dart
GoRoute(
  name: 'profile',
  path: '/user/:id/profile',
  builder: (context, state) => ProfileScreen(),
)
```

### 使用 pageBuilder 自定义页面

```dart
GoRoute(
  path: '/details',
  pageBuilder: (context, state) => MaterialPage<void>(
    key: state.pageKey,
    child: DetailsScreen(),
  ),
)
```

### 带退出确认的路由

```dart
GoRoute(
  path: '/edit',
  builder: (context, state) => EditScreen(),
  onExit: (context, state) async {
    final shouldExit = await showDialog<bool>(
      context: context,
      builder: (context) => ExitConfirmationDialog(),
    );
    return shouldExit ?? false;
  },
)
```

## 继承关系

`GoRoute` 继承自 `RouteBase`，这意味着它继承了以下功能：

- **`redirect`**：路由级别的重定向函数
- **`routes`**：子路由列表
- **`parentNavigatorKey`**：父 Navigator 的 GlobalKey

这些功能使得 `GoRoute` 可以构建复杂的路由树结构，支持嵌套路由和路由重定向。

## 总结

`GoRoute` 是 go_router 包中定义路由的核心类，提供了：

1. **路径匹配**：支持路径参数和查询参数
2. **灵活构建**：支持 `builder` 和 `pageBuilder` 两种构建方式
3. **命名路由**：支持通过名称导航，无需知道确切路径
4. **退出控制**：通过 `onExit` 回调控制路由退出行为
5. **大小写控制**：可配置路径匹配是否区分大小写
6. **性能优化**：通过缓存正则表达式提高匹配性能

通过合理使用这些特性，可以构建出功能强大、易于维护的路由系统。
