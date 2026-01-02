# Navigator Observer 示例代码解析

## 概述

这个示例展示了如何在 Flutter 应用中使用 `NavigatorObserver` 来监听和记录路由导航事件。通过自定义的 `MyNavObserver` 类，我们可以追踪应用中的所有路由变化，包括页面推送、弹出、替换等操作。

## 文件结构

该文件包含以下主要组件：

1. **App 类**：主应用入口，配置了 GoRouter 和导航观察者
2. **MyNavObserver 类**：自定义的导航观察者，用于监听路由事件
3. **Route 扩展**：为 Route 对象添加便捷的字符串表示方法
4. **三个页面组件**：Page1Screen、Page2Screen、Page3Screen

## 核心组件详解

### 应用入口和路由配置

```dart 9:49:example/lib/others/nav_observer.dart
void main() => runApp(App());

/// The main app.
class App extends StatelessWidget {
  /// Creates an [App].
  App({super.key});

  /// The title of the app.
  static const String title = 'GoRouter Example: Navigator Observer';

  @override
  Widget build(BuildContext context) =>
      MaterialApp.router(routerConfig: _router, title: title);

  final GoRouter _router = GoRouter(
    observers: <NavigatorObserver>[MyNavObserver()],
    routes: <GoRoute>[
      GoRoute(
        // if there's no name, path will be used as name for observers
        path: '/',
        builder: (BuildContext context, GoRouterState state) =>
            const Page1Screen(),
        routes: <GoRoute>[
          GoRoute(
            name: 'page2',
            path: 'page2/:p1',
            builder: (BuildContext context, GoRouterState state) =>
                const Page2Screen(),
            routes: <GoRoute>[
              GoRoute(
                name: 'page3',
                path: 'page3',
                builder: (BuildContext context, GoRouterState state) =>
                    const Page3Screen(),
              ),
            ],
          ),
        ],
      ),
    ],
  );
}
```

**关键点**：

- **observers 参数**：在 `GoRouter` 构造函数中传入 `MyNavObserver` 实例，这样所有路由变化都会被观察者捕获
- **路由结构**：定义了嵌套路由结构，从首页 `/` 到 `page2/:p1`，再到 `page3`
- **路由命名**：`page2` 和 `page3` 使用了命名路由，而首页使用路径作为名称（如注释所示）

### 自定义导航观察者

```dart 52:89:example/lib/others/nav_observer.dart
/// The Navigator observer.
class MyNavObserver extends NavigatorObserver {
  /// Creates a [MyNavObserver].
  MyNavObserver() {
    log.onRecord.listen((LogRecord e) => debugPrint('$e'));
  }

  /// The logged message.
  final Logger log = Logger('MyNavObserver');

  @override
  void didPush(Route<dynamic> route, Route<dynamic>? previousRoute) =>
      log.info('didPush: ${route.str}, previousRoute= ${previousRoute?.str}');

  @override
  void didPop(Route<dynamic> route, Route<dynamic>? previousRoute) =>
      log.info('didPop: ${route.str}, previousRoute= ${previousRoute?.str}');

  @override
  void didRemove(Route<dynamic> route, Route<dynamic>? previousRoute) =>
      log.info('didRemove: ${route.str}, previousRoute= ${previousRoute?.str}');

  @override
  void didReplace({Route<dynamic>? newRoute, Route<dynamic>? oldRoute}) =>
      log.info('didReplace: new= ${newRoute?.str}, old= ${oldRoute?.str}');

  @override
  void didStartUserGesture(
    Route<dynamic> route,
    Route<dynamic>? previousRoute,
  ) => log.info(
    'didStartUserGesture: ${route.str}, '
    'previousRoute= ${previousRoute?.str}',
  );

  @override
  void didStopUserGesture() => log.info('didStopUserGesture');
}
```

**功能说明**：

`MyNavObserver` 继承自 `NavigatorObserver`，重写了所有导航生命周期方法：

1. **didPush**：当新路由被推入导航栈时触发（如使用 `context.go()` 或 `context.push()`）
2. **didPop**：当路由从导航栈弹出时触发（如用户点击返回按钮）
3. **didRemove**：当路由被移除时触发（不常见，通常用于清理）
4. **didReplace**：当路由被替换时触发（如使用 `context.go()` 替换当前路由）
5. **didStartUserGesture**：当用户开始手势导航时触发（如开始滑动返回）
6. **didStopUserGesture**：当用户停止手势导航时触发

**日志记录**：

- 使用 `logging` 包的 `Logger` 来记录事件
- 在构造函数中设置监听器，将所有日志记录输出到 `debugPrint`
- 每个方法都记录相应的路由信息，包括当前路由和上一个路由

### Route 扩展方法

```dart 91:93:example/lib/others/nav_observer.dart
extension on Route<dynamic> {
  String get str => 'route(${settings.name}: ${settings.arguments})';
}
```

这个扩展为 `Route` 对象添加了一个 `str` getter，用于获取路由的字符串表示。它包含：

- **路由名称**：`settings.name`（如果路由有名称）
- **路由参数**：`settings.arguments`（传递给路由的参数）

### 页面组件

#### Page1Screen

```dart 95:119:example/lib/others/nav_observer.dart
/// The screen of the first page.
class Page1Screen extends StatelessWidget {
  /// Creates a [Page1Screen].
  const Page1Screen({super.key});

  @override
  Widget build(BuildContext context) => Scaffold(
    appBar: AppBar(title: const Text(App.title)),
    body: Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: <Widget>[
          ElevatedButton(
            onPressed: () => context.goNamed(
              'page2',
              pathParameters: <String, String>{'p1': 'pv1'},
              queryParameters: <String, String>{'q1': 'qv1'},
            ),
            child: const Text('Go to page 2'),
          ),
        ],
      ),
    ),
  );
}
```

**导航方式**：使用 `context.goNamed()` 进行命名路由导航，并传递：

- **路径参数**：`p1: 'pv1'`（对应路由定义中的 `:p1`）
- **查询参数**：`q1: 'qv1'`（会附加到 URL 中，如 `?q1=qv1`）

#### Page2Screen

```dart 121:144:example/lib/others/nav_observer.dart
/// The screen of the second page.
class Page2Screen extends StatelessWidget {
  /// Creates a [Page2Screen].
  const Page2Screen({super.key});

  @override
  Widget build(BuildContext context) => Scaffold(
    appBar: AppBar(title: const Text(App.title)),
    body: Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: <Widget>[
          ElevatedButton(
            onPressed: () => context.goNamed(
              'page3',
              pathParameters: <String, String>{'p1': 'pv2'},
            ),
            child: const Text('Go to page 3'),
          ),
        ],
      ),
    ),
  );
}
```

**导航方式**：同样使用 `context.goNamed()` 导航到 `page3`，传递路径参数 `p1: 'pv2'`

#### Page3Screen

```dart 146:166:example/lib/others/nav_observer.dart
/// The screen of the third page.
class Page3Screen extends StatelessWidget {
  /// Creates a [Page3Screen].
  const Page3Screen({super.key});

  @override
  Widget build(BuildContext context) => Scaffold(
    appBar: AppBar(title: const Text(App.title)),
    body: Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: <Widget>[
          ElevatedButton(
            onPressed: () => context.go('/'),
            child: const Text('Go to home page'),
          ),
        ],
      ),
    ),
  );
}
```

**导航方式**：使用 `context.go('/')` 直接通过路径导航回首页

## 路由结构

应用的路由结构如下：

```text
/ (首页)
  └── page2/:p1 (第二页，带路径参数)
      └── page3 (第三页)
```

**路由路径示例**：

- 首页：`/`
- 第二页：`/page2/pv1`（`pv1` 是路径参数 `p1` 的值）
- 第三页：`/page2/pv2/page3`（嵌套在 `page2` 下）

## 导航事件监听流程

当用户进行导航操作时，`MyNavObserver` 会捕获并记录以下事件：

1. **从首页到第二页**：
   - 触发 `didPush`：新路由是 `page2`，上一个路由是首页

2. **从第二页到第三页**：
   - 触发 `didPush`：新路由是 `page3`，上一个路由是 `page2`

3. **从第三页返回**：
   - 触发 `didPop`：当前路由是 `page3`，上一个路由是 `page2`

4. **从第三页直接跳转回首页**：
   - 触发 `didReplace`：新路由是首页，旧路由是 `page3`（因为使用了 `context.go()` 替换整个路由栈）

## 使用场景

这个示例展示了 `NavigatorObserver` 的典型应用场景：

1. **调试和日志记录**：追踪应用中的路由变化，帮助调试导航问题
2. **分析统计**：记录用户导航路径，用于分析用户行为
3. **状态管理**：在路由变化时更新应用状态（如更新当前页面标题）
4. **权限控制**：在路由变化时检查权限，决定是否允许导航

## 注意事项

1. **路由命名**：如果路由没有指定 `name`，GoRouter 会使用路径作为名称（如注释所示）
2. **日志输出**：所有日志通过 `debugPrint` 输出，在 Release 模式下不会显示
3. **路由参数**：路径参数（`pathParameters`）和查询参数（`queryParameters`）的区别：
   - 路径参数是路由路径的一部分（如 `/page2/:p1`）
   - 查询参数是 URL 查询字符串（如 `?q1=qv1`）
4. **导航方法**：
   - `context.go()`：替换当前路由栈
   - `context.goNamed()`：使用命名路由导航
   - `context.push()`：推入新路由（保留当前路由）

## 扩展建议

基于这个示例，你可以进一步扩展：

1. **添加更多日志信息**：记录路由的完整 URL、时间戳等
2. **集成分析服务**：将导航事件发送到分析平台（如 Firebase Analytics）
3. **添加权限检查**：在 `didPush` 中检查用户权限，决定是否允许导航
4. **状态同步**：在路由变化时同步应用状态（如更新全局导航栏）
