# _CustomNavigator 实现解析

## 概述

`_CustomNavigator` 是 GoRouter 中用于构建自定义导航器的核心组件。它是一个私有 `StatefulWidget`，负责将路由匹配结果（`RouteMatchBase`）转换为 Flutter 的 `Page` 对象，并构建出完整的 `Navigator` 组件树。

## 类结构

### _CustomNavigator (StatefulWidget)

```dart 127:161:packages/go_router/lib/src/builder.dart
class _CustomNavigator extends StatefulWidget {
  const _CustomNavigator({
    super.key,
    required this.navigatorKey,
    required this.observers,
    required this.navigatorRestorationId,
    required this.onPopPageWithRouteMatch,
    required this.matchList,
    required this.matches,
    required this.configuration,
    required this.errorBuilder,
    required this.errorPageBuilder,
    required this.requestFocus,
  });

  final GlobalKey<NavigatorState> navigatorKey;
  final List<NavigatorObserver> observers;

  /// The actual [RouteMatchBase]s to be built.
  ///
  /// This can be different from matches in [matchList] if this widget is used
  /// to build navigator in shell route. In this case, these matches come from
  /// the [ShellRouteMatch.matches].
  final List<RouteMatchBase> matches;
  final RouteMatchList matchList;
  final RouteConfiguration configuration;
  final PopPageWithRouteMatchCallback onPopPageWithRouteMatch;
  final String? navigatorRestorationId;
  final GoRouterWidgetBuilder? errorBuilder;
  final GoRouterPageBuilder? errorPageBuilder;
  final bool requestFocus;

  @override
  State<StatefulWidget> createState() => _CustomNavigatorState();
}
```

#### 关键属性说明

- **`matches`**: 需要构建的实际路由匹配列表。注意：在 ShellRoute 中使用时，这个列表可能不同于 `matchList.matches`，而是来自 `ShellRouteMatch.matches`，用于构建 ShellRoute 内部的子导航器
- **`matchList`**: 完整的路由匹配列表，包含所有匹配信息和错误状态
- **`navigatorKey`**: Navigator 的全局键，用于控制导航栈
- **`onPopPageWithRouteMatch`**: 处理页面弹出时的回调函数
- **`errorBuilder` 和 `errorPageBuilder`**: 用于构建错误页面的构建器
- **`requestFocus`**: 控制新路由是否请求焦点

### _CustomNavigatorState

```dart 163:200:packages/go_router/lib/src/builder.dart
class _CustomNavigatorState extends State<_CustomNavigator> {
  HeroController? _controller;
  late Map<Page<Object?>, RouteMatchBase> _pageToRouteMatchBase;
  final GoRouterStateRegistry _registry = GoRouterStateRegistry();
  List<Page<Object?>>? _pages;

  @override
  void didUpdateWidget(_CustomNavigator oldWidget) {
    super.didUpdateWidget(oldWidget);
    if (widget.matchList != oldWidget.matchList) {
      _pages = null;
    }
  }

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    // Create a HeroController based on the app type.
    if (_controller == null) {
      if (isMaterialApp(context)) {
        _controller = createMaterialHeroController();
      } else if (isCupertinoApp(context)) {
        _controller = createCupertinoHeroController();
      } else {
        _controller = HeroController();
      }
    }
    // This method can also be called if any of the page builders depend on
    // the context. In this case, make sure _pages are rebuilt.
    _pages = null;
  }

  @override
  void dispose() {
    _controller?.dispose();
    _registry.dispose();
    super.dispose();
  }
```

#### 关键状态变量

- **`_controller`**: HeroController，根据应用类型（MaterialApp/CupertinoApp）创建，用于处理页面间的 Hero 动画
- **`_pageToRouteMatchBase`**: Page 到 RouteMatchBase 的映射，用于在页面弹出时找到对应的路由匹配
- **`_registry`**: GoRouterStateRegistry，用于注册和管理每个页面的 GoRouterState
- **`_pages`**: 缓存构建好的 Page 列表，使用懒加载策略

## 核心方法解析

### _updatePages：构建页面列表

```dart 202:226:packages/go_router/lib/src/builder.dart
  void _updatePages(BuildContext context) {
    assert(_pages == null);
    final pages = <Page<Object?>>[];
    final pageToRouteMatchBase = <Page<Object?>, RouteMatchBase>{};
    final registry = <Page<Object?>, GoRouterState>{};
    if (widget.matchList.isError) {
      pages.add(_buildErrorPage(context, widget.matchList));
    } else {
      for (final RouteMatchBase match in widget.matches) {
        final Page<Object?>? page = _buildPage(context, match);
        if (page == null) {
          continue;
        }
        pages.add(page);
        pageToRouteMatchBase[page] = match;
        registry[page] = match.buildState(
          widget.configuration,
          widget.matchList,
        );
      }
    }
    _pages = pages;
    _registry.updateRegistry(registry);
    _pageToRouteMatchBase = pageToRouteMatchBase;
  }
```

这是核心的页面构建方法：

1. **错误处理**: 如果 `matchList.isError` 为 true，构建错误页面
2. **遍历匹配**: 遍历所有 `RouteMatchBase`，为每个匹配构建对应的 Page
3. **状态注册**: 为每个 Page 构建 `GoRouterState` 并注册到 `_registry` 中
4. **映射建立**: 建立 Page 到 RouteMatchBase 的映射，用于后续的页面弹出处理

### _buildPage：页面构建分发器

```dart 228:239:packages/go_router/lib/src/builder.dart
  Page<Object?>? _buildPage(BuildContext context, RouteMatchBase match) {
    if (match is RouteMatch) {
      if (match is ImperativeRouteMatch && match.matches.isError) {
        return _buildErrorPage(context, match.matches);
      }
      return _buildPageForGoRoute(context, match);
    }
    if (match is ShellRouteMatch) {
      return _buildPageForShellRoute(context, match);
    }
    throw GoError('unknown match type ${match.runtimeType}');
  }
```

根据路由匹配类型进行分发：

- **`RouteMatch`**: 普通路由匹配，调用 `_buildPageForGoRoute`
- **`ImperativeRouteMatch`**: 命令式路由匹配，如果存在错误则构建错误页面
- **`ShellRouteMatch`**: Shell 路由匹配，调用 `_buildPageForShellRoute`

### _buildPageForGoRoute：构建普通路由页面

```dart 241:269:packages/go_router/lib/src/builder.dart
  /// Builds a [Page] for a [RouteMatch]
  Page<Object?>? _buildPageForGoRoute(BuildContext context, RouteMatch match) {
    final GoRouterPageBuilder? pageBuilder = match.route.pageBuilder;
    final GoRouterState state = match.buildState(
      widget.configuration,
      widget.matchList,
    );
    if (pageBuilder != null) {
      final Page<Object?> page = pageBuilder(context, state);
      if (page is! NoOpPage) {
        return page;
      }
    }

    final GoRouterWidgetBuilder? builder = match.route.builder;

    if (builder == null) {
      return null;
    }
    return _buildPlatformAdapterPage(
      context,
      state,
      Builder(
        builder: (BuildContext context) {
          return builder(context, state);
        },
      ),
    );
  }
```

构建流程：

1. **构建状态**: 调用 `match.buildState` 构建 `GoRouterState`
2. **优先使用 pageBuilder**: 如果路由定义了 `pageBuilder`，使用它构建 Page（如果结果不是 `NoOpPage`）
3. **回退到 builder**: 如果没有 `pageBuilder` 或返回 `NoOpPage`，使用 `builder` 并通过 `_buildPlatformAdapterPage` 包装为平台适配的 Page

### _buildPageForShellRoute：构建 Shell 路由页面

```dart 271:338:packages/go_router/lib/src/builder.dart
  /// Builds a [Page] for a [ShellRouteMatch]
  Page<Object?> _buildPageForShellRoute(
    BuildContext context,
    ShellRouteMatch match,
  ) {
    final GoRouterState state = match.buildState(
      widget.configuration,
      widget.matchList,
    );
    final GlobalKey<NavigatorState> navigatorKey = match.navigatorKey;
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
    final Page<Object?>? page = match.route.buildPage(
      context,
      state,
      shellRouteContext,
    );
    if (page != null && page is! NoOpPage) {
      return page;
    }

    // Return the result of the route's builder() or pageBuilder()
    return _buildPlatformAdapterPage(
      context,
      state,
      Builder(
        builder: (BuildContext context) {
          return match.route.buildWidget(context, state, shellRouteContext)!;
        },
      ),
    );
  }
```

ShellRoute 的构建特点：

1. **嵌套导航器**: 通过 `navigatorBuilder` 创建一个新的 `_CustomNavigator` 来构建 ShellRoute 内部的子路由，实现递归构建
2. **PopScope 保护**: 使用 `PopScope` 包裹，当 ShellRoute 有活跃子路由时（`match.matches.length > 1`），防止 ShellRoute 被弹出
3. **ShellRouteContext**: 创建 `ShellRouteContext` 提供 ShellRoute 的上下文信息
4. **构建策略**: 优先使用 `route.buildPage`，否则使用 `route.buildWidget` 并通过 `_buildPlatformAdapterPage` 包装

### _buildPlatformAdapterPage：平台适配页面构建

```dart 387:405:packages/go_router/lib/src/builder.dart
  /// builds the page based on app type, i.e. MaterialApp vs. CupertinoApp
  Page<Object?> _buildPlatformAdapterPage(
    BuildContext context,
    GoRouterState state,
    Widget child,
  ) {
    // build the page based on app type
    _cacheAppType(context);
    return _pageBuilderForAppType!(
      key: state.pageKey,
      name: state.name ?? state.path,
      arguments: <String, String>{
        ...state.pathParameters,
        ...state.uri.queryParameters,
      },
      restorationId: state.pageKey.value,
      child: child,
    );
  }
```

根据应用类型构建对应的 Page：

- **MaterialApp**: 使用 `MaterialPage`
- **CupertinoApp**: 使用 `CupertinoPage`
- **WidgetsApp**: 使用 `NoTransitionPage`

### _cacheAppType：缓存应用类型

```dart 344:385:packages/go_router/lib/src/builder.dart
  void _cacheAppType(BuildContext context) {
    // cache app type-specific page and error builders
    if (_pageBuilderForAppType == null) {
      assert(_errorBuilderForAppType == null);

      // can be null during testing
      final Element? elem = context is Element ? context : null;

      if (elem != null && isMaterialApp(elem)) {
        log('Using MaterialApp configuration');
        _pageBuilderForAppType = pageBuilderForMaterialApp;
        _errorBuilderForAppType = (BuildContext c, GoRouterState s) =>
            MaterialErrorScreen(s.error);
      } else if (elem != null && isCupertinoApp(elem)) {
        log('Using CupertinoApp configuration');
        _pageBuilderForAppType = pageBuilderForCupertinoApp;
        _errorBuilderForAppType = (BuildContext c, GoRouterState s) =>
            CupertinoErrorScreen(s.error);
      } else {
        log('Using WidgetsApp configuration');
        _pageBuilderForAppType =
            ({
              required LocalKey key,
              required String? name,
              required Object? arguments,
              required String restorationId,
              required Widget child,
            }) => NoTransitionPage<void>(
              name: name,
              arguments: arguments,
              key: key,
              restorationId: restorationId,
              child: child,
            );
        _errorBuilderForAppType = (BuildContext c, GoRouterState s) =>
            ErrorScreen(s.error);
      }
    }

    assert(_pageBuilderForAppType != null);
    assert(_errorBuilderForAppType != null);
  }
```

根据应用类型缓存对应的页面构建器和错误页面构建器，避免重复检测。

### _buildErrorPage：构建错误页面

```dart 421:441:packages/go_router/lib/src/builder.dart
  /// Builds a an error page.
  Page<void> _buildErrorPage(BuildContext context, RouteMatchList matchList) {
    final GoRouterState state = _buildErrorState(matchList);
    assert(state.error != null);

    // If the error page builder is provided, use that, otherwise, if the error
    // builder is provided, wrap that in an app-specific page (for example,
    // MaterialPage). Finally, if nothing is provided, use a default error page
    // wrapped in the app-specific page.
    _cacheAppType(context);
    final GoRouterWidgetBuilder? errorBuilder = widget.errorBuilder;
    return widget.errorPageBuilder != null
        ? widget.errorPageBuilder!(context, state)
        : _buildPlatformAdapterPage(
            context,
            state,
            errorBuilder != null
                ? errorBuilder(context, state)
                : _errorBuilderForAppType!(context, state),
          );
  }
```

错误页面构建优先级：

1. **`errorPageBuilder`**: 如果提供了 `errorPageBuilder`，直接使用
2. **`errorBuilder`**: 如果提供了 `errorBuilder`，使用 `_buildPlatformAdapterPage` 包装
3. **默认错误页面**: 使用应用类型对应的默认错误页面（`MaterialErrorScreen`、`CupertinoErrorScreen` 或 `ErrorScreen`）

### build：构建 Navigator

```dart 449:469:packages/go_router/lib/src/builder.dart
  @override
  Widget build(BuildContext context) {
    if (_pages == null) {
      _updatePages(context);
    }
    assert(_pages != null);
    return GoRouterStateRegistryScope(
      registry: _registry,
      child: HeroControllerScope(
        controller: _controller!,
        child: Navigator(
          key: widget.navigatorKey,
          requestFocus: widget.requestFocus,
          restorationScopeId: widget.navigatorRestorationId,
          pages: _pages!,
          observers: widget.observers,
          onPopPage: _handlePopPage,
        ),
      ),
    );
  }
```

构建完整的 Widget 树：

1. **懒加载页面**: 如果 `_pages` 为 null，调用 `_updatePages` 构建页面列表
2. **GoRouterStateRegistryScope**: 提供 GoRouterState 注册表，使得子组件可以通过 `GoRouterState.of(context)` 访问状态
3. **HeroControllerScope**: 提供 HeroController，支持页面间的 Hero 动画
4. **Navigator**: 实际的导航器组件，使用构建好的页面列表

### _handlePopPage：处理页面弹出

```dart 443:447:packages/go_router/lib/src/builder.dart
  bool _handlePopPage(Route<Object?> route, Object? result) {
    final page = route.settings as Page<Object?>;
    final RouteMatchBase match = _pageToRouteMatchBase[page]!;
    return widget.onPopPageWithRouteMatch(route, result, match);
  }
```

当 Navigator 尝试弹出页面时，通过 `_pageToRouteMatchBase` 映射找到对应的 `RouteMatchBase`，然后调用 `onPopPageWithRouteMatch` 回调。

## 生命周期管理

### didUpdateWidget

```dart 169:175:packages/go_router/lib/src/builder.dart
  @override
  void didUpdateWidget(_CustomNavigator oldWidget) {
    super.didUpdateWidget(oldWidget);
    if (widget.matchList != oldWidget.matchList) {
      _pages = null;
    }
  }
```

当 `matchList` 发生变化时，清空 `_pages` 缓存，强制在下次 `build` 时重新构建页面。

### didChangeDependencies

```dart 177:193:packages/go_router/lib/src/builder.dart
  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    // Create a HeroController based on the app type.
    if (_controller == null) {
      if (isMaterialApp(context)) {
        _controller = createMaterialHeroController();
      } else if (isCupertinoApp(context)) {
        _controller = createCupertinoHeroController();
      } else {
        _controller = HeroController();
      }
    }
    // This method can also be called if any of the page builders depend on
    // the context. In this case, make sure _pages are rebuilt.
    _pages = null;
  }
```

1. **初始化 HeroController**: 根据应用类型创建对应的 HeroController
2. **重建页面**: 由于页面构建器可能依赖 context，清空 `_pages` 缓存以确保使用最新的 context

### dispose

```dart 195:200:packages/go_router/lib/src/builder.dart
  @override
  void dispose() {
    _controller?.dispose();
    _registry.dispose();
    super.dispose();
  }
```

清理资源：释放 HeroController 和 GoRouterStateRegistry。

## 设计模式与特点

### 1. 递归构建

ShellRoute 的构建使用了递归模式：`_buildPageForShellRoute` 中创建的 `navigatorBuilder` 会创建新的 `_CustomNavigator` 实例来构建子路由，形成嵌套的导航器结构。

### 2. 懒加载缓存

使用 `_pages` 为 null 表示需要构建，实现懒加载策略，避免不必要的重建。

### 3. 平台适配

通过 `_cacheAppType` 和 `_buildPlatformAdapterPage` 实现跨平台支持，自动适配 MaterialApp、CupertinoApp 和 WidgetsApp。

### 4. 状态管理

通过 `GoRouterStateRegistry` 管理每个页面的 `GoRouterState`，使得组件可以通过 context 访问路由状态。

### 5. 错误处理

提供多层次的错误处理机制：错误页面构建器、错误构建器、默认错误页面。

## 使用场景

- **顶层导航器**: 由 `RouteBuilder.build` 创建，构建应用的顶层 Navigator
- **ShellRoute 嵌套导航器**: 在 ShellRoute 内部递归创建，构建 ShellRoute 的子路由导航器

## 总结

`_CustomNavigator` 是 GoRouter 路由系统与 Flutter Navigator 系统之间的桥梁，它负责：

1. 将路由匹配结果转换为 Flutter 的 Page 对象
2. 处理不同应用类型的平台适配
3. 支持 ShellRoute 的嵌套导航
4. 管理路由状态注册
5. 提供错误页面构建机制
6. 处理页面弹出逻辑

通过这个组件，GoRouter 实现了声明式路由系统与 Flutter 导航系统的无缝集成。
