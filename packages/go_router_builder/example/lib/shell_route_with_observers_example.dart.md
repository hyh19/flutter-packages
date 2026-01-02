# Shell Route 与 NavigatorObserver 使用示例解析

## 概述

本示例展示了如何在 `go_router_builder` 中使用 `TypedShellRoute` 并配置 `NavigatorObserver` 来监听路由导航事件。这是 Shell Route 的高级用法，适用于需要追踪路由生命周期、实现路由分析、日志记录等场景。

## 核心概念

### Shell Route

Shell Route 是 GoRouter 中的一种特殊路由类型，它允许在多个子路由之间共享一个公共的 UI 容器（Shell）。在这个容器中，可以放置导航栏、侧边栏等共享 UI 元素，而子路由的内容会在容器内切换。

### NavigatorObserver

`NavigatorObserver` 是 Flutter 导航系统的观察者，可以监听路由的各种生命周期事件，如路由入栈（push）、出栈（pop）等。通过配置 `$observers` 静态属性，可以为 Shell Route 的导航器添加观察者。

## 代码结构解析

### 应用入口

```dart 12:25:example/lib/shell_route_with_observers_example.dart
void main() => runApp(App());

class App extends StatelessWidget {
  App({super.key});

  @override
  Widget build(BuildContext context) =>
      MaterialApp.router(routerConfig: _router);

  final GoRouter _router = GoRouter(
    routes: $appRoutes,
    initialLocation: '/home',
  );
}
```

应用入口使用 `MaterialApp.router` 配置路由，通过 `$appRoutes`（由代码生成器生成）提供路由配置，初始位置设置为 `/home`。

### Shell Route 定义

```dart 27:49:example/lib/shell_route_with_observers_example.dart
@TypedShellRoute<MyShellRouteData>(
  routes: <TypedRoute<RouteData>>[
    TypedGoRoute<HomeRouteData>(path: '/home'),
    TypedGoRoute<UsersRouteData>(
      path: '/users',
      routes: <TypedGoRoute<UserRouteData>>[
        TypedGoRoute<UserRouteData>(path: ':id'),
      ],
    ),
  ],
)
class MyShellRouteData extends ShellRouteData {
  const MyShellRouteData();

  static final List<NavigatorObserver> $observers = <NavigatorObserver>[
    MyNavigatorObserver(),
  ];

  @override
  Widget builder(BuildContext context, GoRouterState state, Widget navigator) {
    return MyShellRouteScreen(child: navigator);
  }
}
```

这是示例的核心部分：

1. **`@TypedShellRoute` 注解**：定义了一个 Shell Route，包含两个子路由：
   - `/home`：首页路由
   - `/users`：用户列表路由，包含一个动态子路由 `:id` 用于显示用户详情

2. **`$observers` 静态属性**：这是关键特性，通过定义这个静态属性，可以为 Shell Route 的导航器添加观察者列表。代码生成器会自动识别这个属性并将其传递给 Shell Route 的配置。

3. **`builder` 方法**：构建 Shell Route 的 UI 容器，接收 `navigator` 参数（子路由的内容），并将其包装在 `MyShellRouteScreen` 中。

### NavigatorObserver 实现

```dart 51:54:example/lib/shell_route_with_observers_example.dart
class MyNavigatorObserver extends NavigatorObserver {
  @override
  void didPush(Route<dynamic> route, Route<dynamic>? previousRoute) {}
}
```

这是一个自定义的 `NavigatorObserver`，重写了 `didPush` 方法来监听路由入栈事件。在实际应用中，你可以在这里实现：

- 路由分析统计
- 日志记录
- 性能监控
- 用户行为追踪

`NavigatorObserver` 还提供了其他生命周期方法：

- `didPop`：路由出栈时调用
- `didRemove`：路由被移除时调用
- `didReplace`：路由被替换时调用

### Shell Route 容器 UI

```dart 56:103:example/lib/shell_route_with_observers_example.dart
class MyShellRouteScreen extends StatelessWidget {
  const MyShellRouteScreen({required this.child, super.key});

  final Widget child;

  int getCurrentIndex(BuildContext context) {
    final String location = GoRouterState.of(context).uri.path;
    if (location.startsWith('/users')) {
      return 1;
    }
    return 0;
  }

  @override
  Widget build(BuildContext context) {
    final int selectedIndex = getCurrentIndex(context);

    return Scaffold(
      body: Row(
        children: <Widget>[
          NavigationRail(
            destinations: const <NavigationRailDestination>[
              NavigationRailDestination(
                icon: Icon(Icons.home),
                label: Text('Home'),
              ),
              NavigationRailDestination(
                icon: Icon(Icons.group),
                label: Text('Users'),
              ),
            ],
            selectedIndex: selectedIndex,
            onDestinationSelected: (int index) {
              switch (index) {
                case 0:
                  const HomeRouteData().go(context);
                case 1:
                  const UsersRouteData().go(context);
              }
            },
          ),
          const VerticalDivider(thickness: 1, width: 1),
          Expanded(child: child),
        ],
      ),
    );
  }
}
```

这个 Widget 实现了 Shell Route 的容器 UI：

1. **`getCurrentIndex` 方法**：根据当前路由路径确定导航栏的选中索引。如果路径以 `/users` 开头，返回 1，否则返回 0。

2. **`NavigationRail`**：使用侧边导航栏（适合桌面端），包含两个目的地：
   - Home：首页
   - Users：用户列表

3. **`child` 参数**：这是 Shell Route 传入的子路由内容，通过 `Expanded` 包裹以填充剩余空间。

4. **导航处理**：点击导航栏时，使用类型安全的路由数据类（`HomeRouteData`、`UsersRouteData`）进行导航。

### 路由数据类

#### HomeRouteData

```dart 105:112:example/lib/shell_route_with_observers_example.dart
class HomeRouteData extends GoRouteData with $HomeRouteData {
  const HomeRouteData();

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return const Center(child: Text('The home page'));
  }
}
```

首页路由数据类，显示简单的文本内容。

#### UsersRouteData

```dart 114:129:example/lib/shell_route_with_observers_example.dart
class UsersRouteData extends GoRouteData with $UsersRouteData {
  const UsersRouteData();

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return ListView(
      children: <Widget>[
        for (int userID = 1; userID <= 3; userID++)
          ListTile(
            title: Text('User $userID'),
            onTap: () => UserRouteData(id: userID).go(context),
          ),
      ],
    );
  }
}
```

用户列表路由，显示三个用户项，点击后导航到用户详情页面。

#### UserRouteData（对话框实现）

```dart 148:167:example/lib/shell_route_with_observers_example.dart
class UserRouteData extends GoRouteData with $UserRouteData {
  const UserRouteData({required this.id});

  // Without this static key, the dialog will not cover the navigation rail.
  final int id;

  @override
  Page<void> buildPage(BuildContext context, GoRouterState state) {
    return DialogPage(
      key: state.pageKey,
      child: Center(
        child: SizedBox(
          width: 300,
          height: 300,
          child: Card(child: Center(child: Text('User $id'))),
        ),
      ),
    );
  }
}
```

用户详情路由使用对话框形式显示。注意：

1. **`buildPage` 方法**：重写此方法以返回自定义的 `Page` 对象，而不是使用默认的 `MaterialPage`。

2. **`DialogPage`**：自定义的页面类，将内容显示为对话框。

3. **`state.pageKey`**：使用路由状态的页面键，确保对话框能够正确覆盖导航栏。注释中说明了如果没有这个 key，对话框将无法覆盖导航栏。

### DialogPage 实现

```dart 131:146:example/lib/shell_route_with_observers_example.dart
class DialogPage extends Page<void> {
  /// A page to display a dialog.
  const DialogPage({required this.child, super.key});

  /// The widget to be displayed which is usually a [Dialog] widget.
  final Widget child;

  @override
  Route<void> createRoute(BuildContext context) {
    return DialogRoute<void>(
      context: context,
      settings: this,
      builder: (BuildContext context) => child,
    );
  }
}
```

这是一个自定义的 `Page` 实现，用于将路由内容显示为对话框：

1. **继承 `Page<void>`**：实现自定义页面类型。

2. **`createRoute` 方法**：返回 `DialogRoute`，这是 Flutter 提供的对话框路由类型，会在当前页面上方显示对话框。

3. **`child` 参数**：要显示在对话框中的 Widget。

## 工作流程

1. **应用启动**：应用启动时，GoRouter 使用 `$appRoutes` 配置路由，初始位置为 `/home`。

2. **Shell Route 初始化**：`MyShellRouteData` 被创建，`$observers` 中的 `MyNavigatorObserver` 被添加到 Shell Route 的导航器中。

3. **UI 渲染**：`MyShellRouteScreen` 被构建，显示侧边导航栏和当前路由的内容。

4. **导航事件**：
   - 用户点击导航栏时，使用类型安全的路由数据类进行导航
   - 导航事件会被 `MyNavigatorObserver` 捕获（如果实现了相应的方法）

5. **对话框显示**：当导航到用户详情时，`DialogPage` 会创建一个对话框覆盖整个界面，包括导航栏。

## 关键特性

### 1. 类型安全的路由

使用 `go_router_builder` 生成的路由数据类提供了类型安全的导航方式：

```dart
const HomeRouteData().go(context);
UserRouteData(id: userID).go(context);
```

### 2. 观察者模式

通过 `$observers` 静态属性，可以轻松地为 Shell Route 添加多个观察者：

```dart
static final List<NavigatorObserver> $observers = <NavigatorObserver>[
  MyNavigatorObserver(),
  AnalyticsObserver(),
  LoggingObserver(),
];
```

### 3. 对话框覆盖

通过自定义 `Page` 类型和 `buildPage` 方法，可以实现对话框覆盖整个 Shell Route 容器的效果。

## 使用场景

1. **路由分析**：通过 `NavigatorObserver` 追踪用户导航路径，进行数据分析。

2. **日志记录**：记录路由切换事件，便于调试和问题排查。

3. **性能监控**：监控路由切换的性能指标。

4. **用户行为追踪**：追踪用户在应用中的导航行为。

5. **复杂 UI 布局**：需要共享导航栏、侧边栏等 UI 元素的多页面应用。

## 注意事项

1. **`$observers` 命名**：必须使用 `$observers` 这个确切的名称，代码生成器才能识别。

2. **静态属性**：`$observers` 必须是静态的 `final` 属性。

3. **类型要求**：`$observers` 的类型必须是 `List<NavigatorObserver>`。

4. **对话框 Key**：使用对话框时，必须使用 `state.pageKey` 作为 `Page` 的 key，否则对话框可能无法正确覆盖导航栏。

5. **观察者生命周期**：观察者会在 Shell Route 的导航器生命周期内保持活跃，直到 Shell Route 被销毁。

## 扩展建议

1. **实现完整的观察者**：在 `MyNavigatorObserver` 中实现所有生命周期方法，以全面追踪路由事件。

2. **添加更多观察者**：可以添加多个观察者，分别处理不同的功能（分析、日志、监控等）。

3. **错误处理**：在观察者中添加错误处理逻辑，避免观察者异常影响路由导航。

4. **性能优化**：如果观察者中有耗时操作，考虑使用异步处理或队列机制。

## 相关示例

- `shell_route_example.dart`：基础的 Shell Route 示例，不包含观察者
- `shell_route_with_keys_example.dart`：使用 Navigator Key 的 Shell Route 示例

## 总结

本示例展示了 `go_router_builder` 中 Shell Route 与 NavigatorObserver 的集成使用。通过 `$observers` 静态属性，可以轻松地为 Shell Route 添加路由观察者，实现路由生命周期监听、分析统计等功能。同时，示例还展示了如何在 Shell Route 中使用对话框覆盖整个容器，这是一个常见的 UI 需求。
