# Stateful Shell Route 初始位置示例详解

## 概述

本示例展示了如何在 `StatefulShellRoute` 的各个分支（branch）中设置**初始位置**（initial location）。这是 `go_router_builder` 的一个高级特性，允许你为每个分支指定首次激活时应该导航到的具体路径，特别是当分支包含带参数的路由时非常有用。

### 核心概念

- **应用级初始位置**：通过 `GoRouter` 的 `initialLocation` 参数设置应用启动时的默认路由
- **分支级初始位置**：通过 `StatefulShellBranchData` 子类中的 `$initialLocation` 静态属性设置分支首次激活时的默认路由
- **带参数路由的默认值**：当分支包含带路径参数的路由时，可以指定默认参数值

## 应用入口和全局初始位置

```dart 16:27:example/lib/stateful_shell_route_initial_location_example.dart
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

应用入口使用 `MaterialApp.router` 配置路由。`initialLocation: '/home'` 设置应用启动时的初始路由为 `/home`，这会激活 Home 分支（索引 0）。

**注意**：应用级的 `initialLocation` 决定应用启动时显示哪个分支和路由。如果设置为 `/notifications/old`，应用启动时会直接显示 Notifications 分支，并导航到 `/notifications/old` 路径。

## StatefulShellRoute 定义

示例定义了一个包含三个分支的 Shell 路由：

```dart 37:67:example/lib/stateful_shell_route_initial_location_example.dart
@TypedStatefulShellRoute<MainShellRouteData>(
  branches: <TypedStatefulShellBranch<StatefulShellBranchData>>[
    TypedStatefulShellBranch<HomeShellBranchData>(
      routes: <TypedRoute<RouteData>>[
        TypedGoRoute<HomeRouteData>(path: '/home'),
      ],
    ),
    TypedStatefulShellBranch<NotificationsShellBranchData>(
      routes: <TypedRoute<RouteData>>[
        TypedGoRoute<NotificationsRouteData>(path: '/notifications/:section'),
      ],
    ),
    TypedStatefulShellBranch<OrdersShellBranchData>(
      routes: <TypedRoute<RouteData>>[
        TypedGoRoute<OrdersRouteData>(path: '/orders'),
      ],
    ),
  ],
)
class MainShellRouteData extends StatefulShellRouteData {
  const MainShellRouteData();

  @override
  Widget builder(
    BuildContext context,
    GoRouterState state,
    StatefulNavigationShell navigationShell,
  ) {
    return MainPageView(navigationShell: navigationShell);
  }
}
```

### 分支结构

1. **Home 分支**（索引 0）：包含 `/home` 路由
2. **Notifications 分支**（索引 1）：包含 `/notifications/:section` 路由，这是一个带路径参数的路由
3. **Orders 分支**（索引 2）：包含 `/orders` 路由

## 分支数据类定义

每个分支都有对应的数据类，继承自 `StatefulShellBranchData`：

```dart 69:81:example/lib/stateful_shell_route_initial_location_example.dart
class HomeShellBranchData extends StatefulShellBranchData {
  const HomeShellBranchData();
}

class NotificationsShellBranchData extends StatefulShellBranchData {
  const NotificationsShellBranchData();

  static String $initialLocation = '/notifications/old';
}

class OrdersShellBranchData extends StatefulShellBranchData {
  const OrdersShellBranchData();
}
```

### 关键特性：`$initialLocation` 静态属性

`NotificationsShellBranchData` 中定义了 `static String $initialLocation = '/notifications/old'`。这个静态属性告诉代码生成器：当 Notifications 分支首次被激活时（比如用户点击底部导航栏的 Notifications 标签），应该导航到 `/notifications/old` 而不是 `/notifications/latest`。

**工作原理**：

1. 代码生成器会检查每个 `StatefulShellBranchData` 子类是否有 `$initialLocation` 静态属性
2. 如果存在，生成的代码会将其传递给 `StatefulShellBranchData.$branch()` 的 `initialLocation` 参数
3. 当用户首次切换到该分支时，路由系统会导航到指定的初始位置

查看生成的代码可以确认这一点：

```dart 21:29:example/lib/stateful_shell_route_initial_location_example.g.dart
    StatefulShellBranchData.$branch(
      initialLocation: NotificationsShellBranchData.$initialLocation,
      routes: [
        GoRouteData.$route(
          path: '/notifications/:section',
          factory: $NotificationsRouteData._fromState,
        ),
      ],
    ),
```

可以看到，`NotificationsShellBranchData.$initialLocation` 被传递给了 `$branch()` 方法。

## 路由数据类

### HomeRouteData

```dart 83:90:example/lib/stateful_shell_route_initial_location_example.dart
class HomeRouteData extends GoRouteData with $HomeRouteData {
  const HomeRouteData();

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return const HomePageView(label: 'Home page');
  }
}
```

Home 路由是一个简单的无参数路由，直接显示 `HomePageView`。

### NotificationsRouteData

```dart 92:103:example/lib/stateful_shell_route_initial_location_example.dart
enum NotificationsPageSection { latest, old, archive }

class NotificationsRouteData extends GoRouteData with $NotificationsRouteData {
  const NotificationsRouteData({required this.section});

  final NotificationsPageSection section;

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return NotificationsPageView(section: section);
  }
}
```

Notifications 路由接受一个 `section` 参数，类型为 `NotificationsPageSection` 枚举。这个参数从路径 `/notifications/:section` 中解析而来。由于 `$initialLocation` 设置为 `/notifications/old`，当用户首次切换到 Notifications 分支时，会自动导航到 `old` 部分。

### OrdersRouteData

`OrdersRouteData` 定义在单独的 `separate_file_route.dart` 文件中，展示了如何将路由定义模块化到独立文件中：

```dart 12:19:example/lib/separate_file_route.dart
class OrdersRouteData extends GoRouteData with $OrdersRouteData {
  const OrdersRouteData();

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return const OrdersPageView(label: 'Orders page');
  }
}
```

**代码组织说明**：

- 主文件通过 `import 'separate_file_route.dart'` 导入该路由定义（见主文件第 10 行）
- 这种组织方式有助于保持代码结构清晰，特别是当项目包含多个路由时
- 路由数据类和对应的页面组件可以放在同一个文件中，便于维护

## UI 组件实现

### MainPageView - Shell 容器

```dart 105:136:example/lib/stateful_shell_route_initial_location_example.dart
class MainPageView extends StatelessWidget {
  const MainPageView({required this.navigationShell, super.key});

  final StatefulNavigationShell navigationShell;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(),
      body: navigationShell,
      bottomNavigationBar: BottomNavigationBar(
        items: const <BottomNavigationBarItem>[
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Home'),
          BottomNavigationBarItem(
            icon: Icon(Icons.favorite),
            label: 'Notifications',
          ),
          BottomNavigationBarItem(icon: Icon(Icons.list), label: 'Orders'),
        ],
        currentIndex: navigationShell.currentIndex,
        onTap: (int index) => _onTap(context, index),
      ),
    );
  }

  void _onTap(BuildContext context, int index) {
    navigationShell.goBranch(
      index,
      initialLocation: index == navigationShell.currentIndex,
    );
  }
}
```

`MainPageView` 是 Shell 路由的容器组件，包含：

- **底部导航栏**：三个标签页对应三个分支
- **当前索引**：通过 `navigationShell.currentIndex` 获取当前激活的分支索引
- **切换分支**：`_onTap` 方法使用 `navigationShell.goBranch()` 切换分支

**关键点**：`goBranch()` 方法的 `initialLocation` 参数：

- 当 `index == navigationShell.currentIndex` 时（点击当前分支），`initialLocation` 为 `true`，会导航到分支的初始位置
- 当切换到其他分支时，`initialLocation` 为 `false`，会保持该分支的当前导航状态（如果之前访问过）

### HomePageView

```dart 138:147:example/lib/stateful_shell_route_initial_location_example.dart
class HomePageView extends StatelessWidget {
  const HomePageView({required this.label, super.key});

  final String label;

  @override
  Widget build(BuildContext context) {
    return Center(child: Text(label));
  }
}
```

简单的页面组件，显示传入的标签文本。

### NotificationsPageView

```dart 149:187:example/lib/stateful_shell_route_initial_location_example.dart
class NotificationsPageView extends StatelessWidget {
  const NotificationsPageView({super.key, required this.section});

  final NotificationsPageSection section;

  @override
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: 3,
      initialIndex: NotificationsPageSection.values.indexOf(section),
      child: const Column(
        children: <Widget>[
          TabBar(
            tabs: <Tab>[
              Tab(
                child: Text('Latest', style: TextStyle(color: Colors.black87)),
              ),
              Tab(
                child: Text('Old', style: TextStyle(color: Colors.black87)),
              ),
              Tab(
                child: Text('Archive', style: TextStyle(color: Colors.black87)),
              ),
            ],
          ),
          Expanded(
            child: TabBarView(
              children: <Widget>[
                NotificationsSubPageView(label: 'Latest notifications'),
                NotificationsSubPageView(label: 'Old notifications'),
                NotificationsSubPageView(label: 'Archived notifications'),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

`NotificationsPageView` 使用 `TabBar` 和 `TabBarView` 实现三个子页面的切换。关键点是 `initialIndex` 参数：它根据传入的 `section` 枚举值设置初始选中的标签页。

由于 `$initialLocation` 设置为 `/notifications/old`，当用户首次切换到 Notifications 分支时，`section` 参数会是 `NotificationsPageSection.old`，因此会默认选中 "Old" 标签页。

### OrdersPageView

```dart 21:30:example/lib/separate_file_route.dart
class OrdersPageView extends StatelessWidget {
  const OrdersPageView({required this.label, super.key});

  final String label;

  @override
  Widget build(BuildContext context) {
    return Center(child: Text(label));
  }
}
```

`OrdersPageView` 是一个简单的页面组件，与 `HomePageView` 结构类似，显示传入的标签文本。它定义在 `separate_file_route.dart` 文件中，与 `OrdersRouteData` 放在一起，体现了良好的代码组织实践。

## 使用场景

这个示例展示了 `$initialLocation` 的典型使用场景：

1. **带参数路由的默认值**：当分支包含带路径参数的路由时，可以指定合理的默认值
2. **用户体验优化**：例如，通知页面可能希望默认显示"旧通知"而不是"最新通知"
3. **业务逻辑需求**：根据应用需求，为不同分支设置不同的默认页面

## 总结

- **应用级初始位置**：通过 `GoRouter.initialLocation` 设置，决定应用启动时显示的路由
- **分支级初始位置**：通过 `StatefulShellBranchData` 子类的 `$initialLocation` 静态属性设置，决定分支首次激活时的默认路由
- **代码生成**：代码生成器会自动识别 `$initialLocation` 属性，并将其传递给路由配置
- **实际应用**：特别适用于带参数路由需要默认值的场景，提升用户体验

通过合理使用这两个初始位置配置，可以精确控制应用的导航行为，提供更好的用户体验。
