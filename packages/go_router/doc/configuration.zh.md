# 配置

通过调用 [GoRouter][] 构造函数并提供 [GoRoute][] 对象列表来创建 GoRouter 配置：

```dart
GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const Page1Screen(),
    ),
    GoRoute(
      path: '/page2',
      builder: (context, state) => const Page2Screen(),
    ),
  ],
);
```

## GoRoute

要配置 GoRoute，必须提供路径模板和构建器。通过提供 `path` 参数来指定要处理的路径模板，通过提供 `builder` 或 `pageBuilder` 参数来指定构建器：

```dart
GoRoute(
  path: '/users/:userId',
  builder: (context, state) => const UserScreen(),
),
```

要导航到此路由，请使用 [go()](https://pub.dev/documentation/go_router/latest/go_router/GoRouter/go.html)。要了解更多关于导航工作原理的信息，请访问 [Navigation](https://pub.dev/documentation/go_router/latest/topics/Navigation-topic.html) 主题。

## Parameters

要指定路径参数，请在路径段前加上 `:` 字符，后跟唯一名称，例如 `:userId`。您可以通过提供给构建器回调的 [GoRouterState][] 对象访问参数值：

```dart
GoRoute(
  path: '/users/:userId',
  builder: (context, state) => const UserScreen(id: state.pathParameters['userId']),
),
```

类似地，要访问[查询字符串](https://en.wikipedia.org/wiki/Query_string)参数（URL 中 `?` 之后的部分），请使用 [GoRouterState][]。例如，像 `/users?filter=admins` 这样的 URL 路径可以读取 `filter` 参数：

```dart
GoRoute(
  path: '/users',
  builder: (context, state) => const UsersScreen(filter: state.uri.queryParameters['filter']),
),
```

## Child routes

匹配的路由可能导致在 Navigator 上显示多个屏幕。这相当于调用 `push()`，新屏幕会显示在先前屏幕的上方，带有过渡动画，如果使用了 `AppBar` 组件，还会有一个应用内返回按钮。

要在另一个屏幕之上显示屏幕，请通过将其添加到父路由的 `routes` 列表来添加子路由：

```dart
GoRoute(
  path: '/',
  builder: (context, state) {
    return HomeScreen();
  },
  routes: [
    GoRoute(
      path: 'details',
      builder: (context, state) {
        return DetailsScreen();
      },
    ),
  ],
)
```

## Dynamic RoutingConfig

[RoutingConfig][] 提供了一种在 [GoRouter][] 已创建后更新 GoRoute\[s\] 的方法。这可以通过使用特殊构造函数 [GoRouter.routingConfig][] 创建 GoRouter 来完成：

```dart
final ValueNotifier<RoutingConfig> myRoutingConfig = ValueNotifier<RoutingConfig>(
  RoutingConfig(
    routes: <RouteBase>[GoRoute(path: '/', builder: (_, __) => HomeScreen())],
  ),
);
final GoRouter router = GoRouter.routingConfig(routingConfig: myRoutingConfig);
```

要稍后更改 GoRoute，请直接修改 [ValueNotifier][] 的值：

```dart
myRoutingConfig.value = RoutingConfig(
  routes: <RouteBase>[
    GoRoute(path: '/', builder: (_, __) => AlternativeHomeScreen()),
    GoRoute(path: '/a-new-route', builder: (_, __) => SomeScreen()),
  ],
);
```

值更改会被 GoRouter 自动捕获，并导致它重新解析存储在 GoRouter 中的当前路由，即 RouteMatchList。RouteMatchList 将反映 `RoutingConfig` 的最新更改。

## Nested navigation

某些应用在屏幕的某个子部分显示目标，例如，使用 BottomNavigationBar 的应用，在目标之间导航时该组件保持在屏幕上。

要添加额外的 Navigator，请使用 [ShellRoute][] 并提供一个返回 widget 的构建器：

```dart
ShellRoute(
  builder:
      (BuildContext context, GoRouterState state, Widget child) {
    return Scaffold(
      body: child,
      /* ... */
      bottomNavigationBar: BottomNavigationBar(
      /* ... */
      ),
    );
  },
  routes: <RouteBase>[
    GoRoute(
      path: 'details',
      builder: (BuildContext context, GoRouterState state) {
        return const DetailsScreen();
      },
    ),
  ],
),
```

`child` widget 是一个配置为显示匹配子路由的 Navigator。

有关更多详细信息，请参阅 [ShellRoute API 文档](https://pub.dev/documentation/go_router/latest/go_router/ShellRoute-class.html)。有关完整示例，请参阅 example/ 目录中的 [ShellRoute 示例](https://github.com/flutter/packages/tree/main/packages/go_router/example/lib/shell_route.dart)。

## Stateful nested navigation

除了使用嵌套导航（例如使用 BottomNavigationBar）之外，许多应用还要求在目标之间导航时保持状态。要实现此目的，请使用 [StatefulShellRoute][] 而不是 `ShellRoute`。

StatefulShellRoute 为其每个嵌套[分支](https://pub.dev/documentation/go_router/latest/go_router/StatefulShellBranch-class.html)（即并行导航树）创建单独的 `Navigator`，从而可以构建具有有状态嵌套导航的应用。构造函数 [StatefulShellRoute.indexedStack](https://pub.dev/documentation/go_router/latest/go_router/StatefulShellRoute/StatefulShellRoute.indexedStack.html) 提供了使用 `IndexedStack` 管理分支导航器的默认实现。

使用 StatefulShellRoute 时，路由不会在 shell 路由本身上配置。相反，它们为每个分支配置。示例：

<?code-excerpt "../example/lib/stateful_shell_route.dart (configuration-branches)"?>

```dart
branches: <StatefulShellBranch>[
  // 底部导航栏第一个标签页的路由分支。
  StatefulShellBranch(
    navigatorKey: _sectionANavigatorKey,
    routes: <RouteBase>[
      GoRoute(
        // 在底部导航栏第一个标签页中显示为根屏幕的屏幕。
        path: '/a',
        builder: (BuildContext context, GoRouterState state) =>
            const RootScreen(label: 'A', detailsPath: '/a/details'),
        routes: <RouteBase>[
          // 在第一个标签页的导航器上堆叠显示的详细信息屏幕。
          // 这将覆盖屏幕 A，但不会覆盖应用程序 shell（底部导航栏）。
          GoRoute(
            path: 'details',
            builder: (BuildContext context, GoRouterState state) =>
                const DetailsScreen(label: 'A'),
          ),
        ],
      ),
    ],
    // 要启用分支初始位置的预加载，请为参数 `preload` 传递
    // 'true'（默认为 false）。
  ),
```

与 ShellRoute 类似，必须提供一个构建器来构建封装分支导航容器的实际 shell Widget。后者由类 [StatefulNavigationShell](https://pub.dev/documentation/go_router/latest/go_router/StatefulNavigationShell-class.html) 实现，它作为构建器函数的最后一个参数传递。示例：

<?code-excerpt "../example/lib/stateful_shell_route.dart (configuration-builder)"?>

```dart
StatefulShellRoute.indexedStack(
  builder:
      (
        BuildContext context,
        GoRouterState state,
        StatefulNavigationShell navigationShell,
      ) {
        // 返回实现自定义 shell 的 widget（在这种情况下
        // 使用 BottomNavigationBar）。传递 StatefulNavigationShell
        // 以便能够访问 shell 的状态并以有状态的方式导航到其他分支。
        return ScaffoldWithNavBar(navigationShell: navigationShell);
      },
```

在自定义 shell widget 中，StatefulNavigationShell 首先用作 shell 的子组件或主体。其次，它还用于处理分支之间的有状态切换，以及提供当前活动的分支索引。示例：

<?code-excerpt "../example/lib/stateful_shell_route.dart (configuration-custom-shell)"?>

```dart
@override
Widget build(BuildContext context) {
  return Scaffold(
    // 来自关联 StatefulShellRoute 的 StatefulNavigationShell
    // 直接作为 Scaffold 的主体传递。
    body: navigationShell,
    bottomNavigationBar: BottomNavigationBar(
      // 这里，BottomNavigationBar 的项是硬编码的。在真实
      // 场景中，这些项很可能是从 shell 路由的分支生成的，
      // 可以使用 `navigationShell.route.branches` 获取。
      items: const <BottomNavigationBarItem>[
        BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Section A'),
        BottomNavigationBarItem(icon: Icon(Icons.work), label: 'Section B'),
        BottomNavigationBarItem(icon: Icon(Icons.tab), label: 'Section C'),
      ],
      currentIndex: navigationShell.currentIndex,
      // 当点击 BottomNavigationBar 中的项时，导航到
      // 提供索引处分支的当前位置。
      onTap: (int index) => navigationShell.goBranch(index),
    ),
  );
}
```

有关完整示例，请参阅 example/ 目录中的[有状态嵌套导航](https://github.com/flutter/packages/blob/main/packages/go_router/example/lib/stateful_shell_route.dart)。有关更多详细信息，请参阅 [StatefulShellRoute API 文档](https://pub.dev/documentation/go_router/latest/go_router/StatefulShellRoute-class.html)。

## Initial location

初始位置是在应用首次打开且平台未提供深度链接时显示的位置。要指定初始位置，请向 GoRouter 构造函数提供 `initialLocation` 参数：

```dart
GoRouter(
  initialLocation: '/details',
  /* ... */
);
```

## Logging

要启用日志输出，请启用 `debugLogDiagnostics` 参数：

```dart
final _router = GoRouter(
  routes: [/* ... */],
  debugLogDiagnostics: true,
);
```

[GoRouter]: https://pub.dev/documentation/go_router/latest/go_router/GoRouter-class.html
[GoRoute]: https://pub.dev/documentation/go_router/latest/go_router/GoRoute-class.html
[GoRouterState]: https://pub.dev/documentation/go_router/latest/go_router/GoRouterState-class.html
[ShellRoute]: https://pub.dev/documentation/go_router/latest/go_router/ShellRoute-class.html
[StatefulShellRoute]: https://pub.dev/documentation/go_router/latest/go_router/StatefulShellRoute-class.html
[RoutingConfig]: https://pub.dev/documentation/go_router/latest/go_router/RoutingConfig-class.html
[GoRouter.routingConfig]: https://pub.dev/documentation/go_router/latest/go_router/GoRouter/GoRouter.routingConfig.html
[ValueNotifier]: https://api.flutter.dev/flutter/foundation/ValueNotifier-class.html
