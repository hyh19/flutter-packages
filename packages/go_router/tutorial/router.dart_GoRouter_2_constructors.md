# GoRouter 构造函数

`GoRouter` 类提供了两种构造函数：一个是工厂构造函数 `GoRouter()`，用于创建静态路由配置的路由器；另一个是命名构造函数 `GoRouter.routingConfig()`，用于创建支持动态路由配置的路由器。

## 工厂构造函数 GoRouter()

这是 `GoRouter` 的默认构造函数，适用于大多数静态路由配置的场景。

```dart 174:222:lib/src/router.dart
  /// Default constructor to configure a GoRouter with a routes builder
  /// and an error page builder.
  ///
  /// The `routes` must not be null and must contain an [GoRouter] to match `/`.
  factory GoRouter({
    required List<RouteBase> routes,
    OnEnter? onEnter,
    Codec<Object?, Object?>? extraCodec,
    GoExceptionHandler? onException,
    GoRouterPageBuilder? errorPageBuilder,
    GoRouterWidgetBuilder? errorBuilder,
    GoRouterRedirect? redirect,
    int redirectLimit = 5,
    Listenable? refreshListenable,
    bool routerNeglect = false,
    String? initialLocation,
    bool overridePlatformDefaultLocation = false,
    Object? initialExtra,
    List<NavigatorObserver>? observers,
    bool debugLogDiagnostics = false,
    GlobalKey<NavigatorState>? navigatorKey,
    String? restorationScopeId,
    bool requestFocus = true,
  }) {
    return GoRouter.routingConfig(
      routingConfig: _ConstantRoutingConfig(
        RoutingConfig(
          routes: routes,
          redirect: redirect ?? RoutingConfig._defaultRedirect,
          onEnter: onEnter,
          redirectLimit: redirectLimit,
        ),
      ),
      extraCodec: extraCodec,
      onException: onException,
      errorPageBuilder: errorPageBuilder,
      errorBuilder: errorBuilder,
      refreshListenable: refreshListenable,
      routerNeglect: routerNeglect,
      initialLocation: initialLocation,
      overridePlatformDefaultLocation: overridePlatformDefaultLocation,
      initialExtra: initialExtra,
      observers: observers,
      debugLogDiagnostics: debugLogDiagnostics,
      navigatorKey: navigatorKey,
      restorationScopeId: restorationScopeId,
      requestFocus: requestFocus,
    );
  }
```

### 实现机制

工厂构造函数 `GoRouter()` 实际上是一个便捷方法，它：

1. **创建静态路由配置**：将传入的路由参数包装成 `RoutingConfig` 对象
2. **使用 `_ConstantRoutingConfig`**：将 `RoutingConfig` 包装成不可变的 `ValueListenable`，因为静态配置不会改变
3. **委托给 `GoRouter.routingConfig()`**：调用命名构造函数来完成实际的初始化工作

这种设计允许两种构造函数共享相同的初始化逻辑，同时为静态配置提供更简洁的 API。

### 参数说明

- **`routes`**（必需）：顶层路由列表，必须包含能匹配 `/` 的路由
- **`onEnter`**：导航守卫回调
- **`extraCodec`**：用于序列化/反序列化 `extra` 参数的编解码器
- **`onException`**：异常处理回调
- **`errorPageBuilder`**：错误页面构建器
- **`errorBuilder`**：错误 widget 构建器
- **`redirect`**：顶层重定向回调（旧版 API，保留用于向后兼容）
- **`redirectLimit`**：最大重定向次数（默认 5）
- **`refreshListenable`**：用于触发路由刷新的 Listenable 对象
- **`routerNeglect`**：是否忽略路由变化（用于 Web 平台）
- **`initialLocation`**：初始路由位置
- **`overridePlatformDefaultLocation`**：是否覆盖平台默认位置
- **`initialExtra`**：初始路由的额外数据
- **`observers`**：导航观察者列表
- **`debugLogDiagnostics`**：是否启用调试日志
- **`navigatorKey`**：根导航器的 GlobalKey
- **`restorationScopeId`**：状态恢复作用域 ID
- **`requestFocus`**：是否自动请求焦点（默认 true）

## 命名构造函数 GoRouter.routingConfig()

这个构造函数用于创建支持动态路由配置的 `GoRouter` 实例，适用于需要在运行时改变路由配置的场景。

```dart 224:320:lib/src/router.dart
  /// Creates a [GoRouter] with a dynamic [RoutingConfig].
  ///
  /// See [routing_config.dart](https://github.com/flutter/packages/blob/main/packages/go_router/example/lib/routing_config.dart).
  GoRouter.routingConfig({
    required ValueListenable<RoutingConfig> routingConfig,
    Codec<Object?, Object?>? extraCodec,
    GoExceptionHandler? onException,
    GoRouterPageBuilder? errorPageBuilder,
    GoRouterWidgetBuilder? errorBuilder,
    Listenable? refreshListenable,
    bool routerNeglect = false,
    String? initialLocation,
    this.overridePlatformDefaultLocation = false,
    Object? initialExtra,
    this.observers,
    bool debugLogDiagnostics = false,
    GlobalKey<NavigatorState>? navigatorKey,
    String? restorationScopeId,
    bool requestFocus = true,
  }) : _routingConfig = routingConfig,
       backButtonDispatcher = RootBackButtonDispatcher(),
       assert(
         initialExtra == null || initialLocation != null,
         'initialLocation must be set in order to use initialExtra',
       ),
       assert(
         !overridePlatformDefaultLocation || initialLocation != null,
         'Initial location must be set to override platform default',
       ),
       assert(
         (onException == null ? 0 : 1) +
                 (errorPageBuilder == null ? 0 : 1) +
                 (errorBuilder == null ? 0 : 1) <
             2,
         'Only one of onException, errorPageBuilder, or errorBuilder can be provided.',
       ) {
    setLogging(enabled: debugLogDiagnostics);
    WidgetsFlutterBinding.ensureInitialized();

    navigatorKey ??= GlobalKey<NavigatorState>(debugLabel: 'root');

    _routingConfig.addListener(_handleRoutingConfigChanged);
    configuration = RouteConfiguration(
      _routingConfig,
      navigatorKey: navigatorKey,
      extraCodec: extraCodec,
      router: this,
    );

    final ParserExceptionHandler? parserExceptionHandler;
    if (onException != null) {
      parserExceptionHandler =
          (BuildContext context, RouteMatchList routeMatchList) {
            onException(
              context,
              configuration.buildTopLevelGoRouterState(routeMatchList),
              this,
            );
            // Avoid updating GoRouterDelegate if onException is provided.
            return routerDelegate.currentConfiguration;
          };
    } else {
      parserExceptionHandler = null;
    }

    routeInformationParser = GoRouteInformationParser(
      onParserException: parserExceptionHandler,
      configuration: configuration,
      router: this,
    );

    routeInformationProvider = GoRouteInformationProvider(
      initialLocation: _effectiveInitialLocation(initialLocation),
      initialExtra: initialExtra,
      refreshListenable: refreshListenable,
      routerNeglect: routerNeglect,
    );

    routerDelegate = GoRouterDelegate(
      configuration: configuration,
      errorPageBuilder: errorPageBuilder,
      errorBuilder: errorBuilder,
      routerNeglect: routerNeglect,
      observers: <NavigatorObserver>[...observers ?? <NavigatorObserver>[]],
      restorationScopeId: restorationScopeId,
      requestFocus: requestFocus,
      // wrap the returned Navigator to enable GoRouter.of(context).go() et al,
      // allowing the caller to wrap the navigator themselves
      builderWithNav: (BuildContext context, Widget child) =>
          InheritedGoRouter(goRouter: this, child: child),
    );

    assert(() {
      log('setting initial location $initialLocation');
      return true;
    }());
  }
```

### 关键区别

与工厂构造函数的主要区别：

1. **`routingConfig` 参数**：接受 `ValueListenable<RoutingConfig>` 而不是静态的路由列表，允许路由配置在运行时动态变化
2. **监听路由配置变化**：通过 `_routingConfig.addListener(_handleRoutingConfigChanged)` 监听配置变化并做出响应
3. **不包含 `redirect` 和 `redirectLimit` 参数**：这些参数已经在 `RoutingConfig` 中定义

### 断言检查

构造函数包含三个断言来确保参数的有效性：

1. **`initialExtra` 检查**：如果提供了 `initialExtra`，必须同时提供 `initialLocation`
2. **`overridePlatformDefaultLocation` 检查**：如果要覆盖平台默认位置，必须提供 `initialLocation`
3. **错误处理互斥检查**：`onException`、`errorPageBuilder` 和 `errorBuilder` 三者只能提供一个

### 初始化流程

构造函数的初始化过程按以下顺序进行：

1. **初始化日志系统**：根据 `debugLogDiagnostics` 参数启用或禁用日志
2. **确保 Flutter 绑定已初始化**：调用 `WidgetsFlutterBinding.ensureInitialized()`
3. **创建或使用导航器 Key**：如果未提供，创建一个默认的根导航器 Key
4. **监听路由配置变化**：注册 `_handleRoutingConfigChanged` 监听器
5. **创建 RouteConfiguration**：使用路由配置和导航器 Key 创建路由配置对象
6. **创建异常处理器**：如果提供了 `onException`，创建一个包装器用于路由信息解析器
7. **创建 GoRouteInformationParser**：用于解析路由信息
8. **创建 GoRouteInformationProvider**：用于提供路由信息（处理初始位置、刷新等）
9. **创建 GoRouterDelegate**：核心委托对象，负责构建导航器和处理路由变更
10. **调试日志**：在调试模式下记录初始位置

### 核心组件创建

构造函数创建了 `GoRouter` 的核心组件：

#### RouteConfiguration

路由配置对象，管理路由表和名称到路径的映射：

```dart 266:271:lib/src/router.dart
    configuration = RouteConfiguration(
      _routingConfig,
      navigatorKey: navigatorKey,
      extraCodec: extraCodec,
      router: this,
    );
```

#### GoRouteInformationParser

路由信息解析器，负责将 URI 转换为路由匹配列表：

```dart 289:293:lib/src/router.dart
    routeInformationParser = GoRouteInformationParser(
      onParserException: parserExceptionHandler,
      configuration: configuration,
      router: this,
    );
```

如果提供了 `onException`，会创建一个异常处理器包装器，在解析异常时调用自定义异常处理逻辑。

#### GoRouteInformationProvider

路由信息提供者，管理路由信息的初始化和变更：

```dart 295:300:lib/src/router.dart
    routeInformationProvider = GoRouteInformationProvider(
      initialLocation: _effectiveInitialLocation(initialLocation),
      initialExtra: initialExtra,
      refreshListenable: refreshListenable,
      routerNeglect: routerNeglect,
    );
```

#### GoRouterDelegate

路由委托，负责构建导航器和处理路由状态变化：

```dart 302:314:lib/src/router.dart
    routerDelegate = GoRouterDelegate(
      configuration: configuration,
      errorPageBuilder: errorPageBuilder,
      errorBuilder: errorBuilder,
      routerNeglect: routerNeglect,
      observers: <NavigatorObserver>[...observers ?? <NavigatorObserver>[]],
      restorationScopeId: restorationScopeId,
      requestFocus: requestFocus,
      // wrap the returned Navigator to enable GoRouter.of(context).go() et al,
      // allowing the caller to wrap the navigator themselves
      builderWithNav: (BuildContext context, Widget child) =>
          InheritedGoRouter(goRouter: this, child: child),
    );
```

`builderWithNav` 参数用于包装返回的 Navigator，使其能够通过 `InheritedGoRouter` 提供 `GoRouter` 实例到 widget 树中，从而支持 `GoRouter.of(context)` 等 API。

## 使用场景

- **静态路由配置**：使用工厂构造函数 `GoRouter()`，适合大多数应用场景
- **动态路由配置**：使用 `GoRouter.routingConfig()`，适合需要根据应用状态动态改变路由表的场景（例如，根据用户登录状态显示不同的路由）

## 相关文档

- [routing_config.dart 示例](https://github.com/flutter/packages/blob/main/packages/go_router/example/lib/routing_config.dart)
