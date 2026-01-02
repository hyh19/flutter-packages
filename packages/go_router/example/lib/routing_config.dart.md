# routing_config.dart 代码解析

## 概述

这个示例应用展示了如何使用 `go_router` 包实现**动态路由配置**。核心功能是在运行时动态地向路由配置中添加新路由，而无需重启应用或重新构建整个路由树。

## 核心概念

### 动态路由配置

传统的路由配置通常在应用启动时静态定义，一旦创建就无法修改。而这个示例使用了 `GoRouter.routingConfig` 构造函数配合 `ValueNotifier<RoutingConfig>`，实现了路由配置的**动态更新**。

## 代码结构分析

### 应用入口

```dart 8:9:example/lib/routing_config.dart
/// This app shows how to dynamically add more route into routing config
void main() => runApp(const MyApp());
```

应用入口非常简单，直接启动 `MyApp` 组件。

### 主应用组件

```dart 11:18:example/lib/routing_config.dart
/// The main app.
class MyApp extends StatefulWidget {
  /// Constructs a [MyApp]
  const MyApp({super.key});

  @override
  State<MyApp> createState() => _MyAppState();
}
```

`MyApp` 是一个 `StatefulWidget`，因为需要在运行时修改路由配置，所以需要状态管理。

### 状态管理

```dart 20:24:example/lib/routing_config.dart
class _MyAppState extends State<MyApp> {
  bool isNewRouteAdded = false;

  late final ValueNotifier<RoutingConfig> myConfig =
      ValueNotifier<RoutingConfig>(_generateRoutingConfig());
```

状态类包含两个关键成员：

1. **`isNewRouteAdded`**：布尔标志，用于跟踪是否已添加新路由
2. **`myConfig`**：`ValueNotifier<RoutingConfig>`，这是实现动态路由的关键。它持有一个可观察的路由配置对象，当配置发生变化时，`GoRouter` 会自动响应并更新路由树

### GoRouter 初始化

```dart 26:43:example/lib/routing_config.dart
  late final GoRouter router = GoRouter.routingConfig(
    routingConfig: myConfig,
    errorBuilder: (_, GoRouterState state) => Scaffold(
      appBar: AppBar(title: const Text('Page not found')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text('${state.uri} does not exist'),
            ElevatedButton(
              onPressed: () => router.go('/'),
              child: const Text('Go to home'),
            ),
          ],
        ),
      ),
    ),
  );
```

这里使用了 `GoRouter.routingConfig` 构造函数，这是与传统的 `GoRouter` 构造函数不同的方式：

- **`routingConfig`**：传入 `ValueNotifier<RoutingConfig>`，`GoRouter` 会监听这个 `ValueNotifier` 的变化
- **`errorBuilder`**：自定义错误页面构建器，当路由不存在时显示友好的错误信息，并提供返回首页的按钮

### 路由配置生成方法

```dart 45:105:example/lib/routing_config.dart
  RoutingConfig _generateRoutingConfig() {
    return RoutingConfig(
      routes: <RouteBase>[
        GoRoute(
          path: '/',
          builder: (_, __) {
            return Scaffold(
              appBar: AppBar(title: const Text('Home')),
              body: Center(
                child: Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: <Widget>[
                    ElevatedButton(
                      onPressed: isNewRouteAdded
                          ? null
                          : () {
                              setState(() {
                                isNewRouteAdded = true;
                                // Modify the routing config.
                                myConfig.value = _generateRoutingConfig();
                              });
                            },
                      child: isNewRouteAdded
                          ? const Text('A route has been added')
                          : const Text('Add a new route'),
                    ),
                    ElevatedButton(
                      onPressed: () {
                        router.go('/new-route');
                      },
                      child: const Text('Try going to /new-route'),
                    ),
                  ],
                ),
              ),
            );
          },
        ),
        if (isNewRouteAdded)
          GoRoute(
            path: '/new-route',
            builder: (_, __) {
              return Scaffold(
                appBar: AppBar(title: const Text('A new Route')),
                body: Center(
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: <Widget>[
                      ElevatedButton(
                        onPressed: () => router.go('/'),
                        child: const Text('Go to home'),
                      ),
                    ],
                  ),
                ),
              );
            },
          ),
      ],
    );
  }
```

这个方法负责生成路由配置，包含以下关键点：

#### 首页路由 (`/`)

首页包含两个按钮：

1. **"Add a new route" 按钮**：
   - 当 `isNewRouteAdded` 为 `false` 时显示，点击后会：
     - 设置 `isNewRouteAdded = true`
     - 更新 `myConfig.value`，触发路由配置重新生成
     - 按钮变为禁用状态，文本变为 "A route has been added"
   - 这是**动态添加路由的核心逻辑**

2. **"Try going to /new-route" 按钮**：
   - 始终可用，用于测试导航到新路由
   - 如果路由尚未添加，会触发错误页面

#### 条件路由 (`/new-route`)

```dart 83:102:example/lib/routing_config.dart
        if (isNewRouteAdded)
          GoRoute(
            path: '/new-route',
            builder: (_, __) {
              return Scaffold(
                appBar: AppBar(title: const Text('A new Route')),
                body: Center(
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: <Widget>[
                      ElevatedButton(
                        onPressed: () => router.go('/'),
                        child: const Text('Go to home'),
                      ),
                    ],
                  ),
                ),
              );
            },
          ),
```

使用 Dart 的 `if` 条件表达式，只有当 `isNewRouteAdded` 为 `true` 时，这个路由才会被包含在配置中。这是实现**动态路由添加**的关键技术。

### 构建方法

```dart 107:110:example/lib/routing_config.dart
  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(routerConfig: router);
  }
```

标准的 `MaterialApp.router` 配置，传入之前创建的 `GoRouter` 实例。

## 工作流程

### 1. 初始状态

- `isNewRouteAdded = false`
- 路由配置只包含首页 `/`
- "Add a new route" 按钮可用
- 点击 "Try going to /new-route" 会显示错误页面

### 2. 添加新路由

当用户点击 "Add a new route" 按钮时：

1. `setState` 被调用，`isNewRouteAdded` 变为 `true`
2. `myConfig.value` 被更新为新的 `RoutingConfig`（包含 `/new-route` 路由）
3. `GoRouter` 监听到 `ValueNotifier` 的变化，自动更新路由树
4. UI 重新构建，"Add a new route" 按钮变为禁用状态

### 3. 导航到新路由

此时点击 "Try going to /new-route" 按钮，可以成功导航到新添加的路由页面。

## 技术要点

### ValueNotifier 的作用

`ValueNotifier<RoutingConfig>` 是响应式编程的核心：

- 当 `myConfig.value` 发生变化时，`GoRouter` 会自动收到通知
- 无需手动调用任何刷新方法
- 实现了路由配置的**声明式更新**

### 条件路由

使用 Dart 的集合 `if` 语法实现条件路由：

```dart
routes: <RouteBase>[
  // 基础路由
  GoRoute(...),
  // 条件路由
  if (condition) GoRoute(...),
]
```

这种方式比在运行时动态修改路由列表更加清晰和安全。

### 状态同步

注意 `_generateRoutingConfig()` 方法中直接访问了 `isNewRouteAdded` 状态。这确保了：

- 路由配置始终与当前状态保持一致
- 每次调用 `_generateRoutingConfig()` 都会生成最新的配置
- 避免了状态不同步的问题

## 使用场景

这种动态路由配置适用于以下场景：

1. **按需加载路由**：根据用户权限或功能开关动态添加路由
2. **插件化架构**：插件可以动态注册自己的路由
3. **A/B 测试**：根据实验配置动态启用/禁用某些路由
4. **渐进式功能发布**：逐步向用户开放新功能的路由

## 注意事项

1. **性能考虑**：频繁更新路由配置可能会影响性能，应谨慎使用
2. **状态管理**：确保路由配置生成方法中使用的状态变量是同步的
3. **错误处理**：在动态添加路由时，要处理好导航到不存在路由的情况
4. **路由参数**：动态路由同样支持路径参数和查询参数

## 总结

这个示例展示了 `go_router` 包的高级功能——动态路由配置。通过 `ValueNotifier<RoutingConfig>` 和 `GoRouter.routingConfig` 构造函数，我们可以在运行时灵活地修改路由配置，为应用提供更强大的路由管理能力。
