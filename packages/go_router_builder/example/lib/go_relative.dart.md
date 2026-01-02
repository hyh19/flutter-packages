# go_relative.dart 代码解析

## 概述

这个文件演示了 `go_router_builder` 包中**相对路由（Relative Routes）**的使用方法。相对路由允许你定义一个可以在多个父路由下复用的路由，通过 `goRelative` 方法进行相对导航，而不需要知道完整的绝对路径。

## 核心概念

### 相对路由 vs 绝对路由

- **绝对路由**：路径从根路径 `/` 开始，如 `/dashboard/details/123`
- **相对路由**：路径相对于当前路由，如 `./details/123`，会根据当前所在的路由自动解析为完整路径

### 关键类型

- `TypedRelativeGoRoute`：定义相对路由的注解
- `RelativeGoRouteData`：相对路由数据类，继承自 `GoRouteData`
- `goRelative()`：相对导航方法，会根据当前路由上下文自动解析路径

## 代码结构分析

### 应用入口和路由配置

```dart 12:26:example/lib/go_relative.dart
void main() => runApp(const MyApp());

/// The main app.
class MyApp extends StatelessWidget {
  /// Constructs a [MyApp]
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(routerConfig: _router);
  }
}

/// The route configuration.
final GoRouter _router = GoRouter(routes: $appRoutes);
```

应用使用 `MaterialApp.router` 配置路由，`$appRoutes` 是由代码生成器自动生成的根路由列表。

### 相对路由定义

```dart 27:33:example/lib/go_relative.dart
const TypedRelativeGoRoute<DetailsRoute> detailRoute =
    TypedRelativeGoRoute<DetailsRoute>(
      path: 'details/:detailId',
      routes: <TypedRoute<RouteData>>[
        TypedRelativeGoRoute<SettingsRoute>(path: 'settings/:settingId'),
      ],
    );
```

这里定义了一个**可复用的相对路由** `detailRoute`：

- 路径为 `details/:detailId`（注意没有前导斜杠，表示相对路径）
- 包含一个嵌套的相对路由 `SettingsRoute`
- 这个路由可以在多个父路由下使用

### 路由层次结构

```dart 35:50:example/lib/go_relative.dart
@TypedGoRoute<HomeRoute>(
  path: '/',
  routes: <TypedRoute<RouteData>>[
    TypedGoRoute<DashboardRoute>(
      path: '/dashboard',
      routes: <TypedRoute<RouteData>>[detailRoute],
    ),
    detailRoute,
  ],
)
class HomeRoute extends GoRouteData with $HomeRoute {
  @override
  Widget build(BuildContext context, GoRouterState state) {
    return const HomeScreen();
  }
}
```

路由结构如下：

```text
/ (HomeRoute)
├── /dashboard (DashboardRoute)
│   └── details/:detailId (DetailsRoute)  ← 相对路由
│       └── settings/:settingId (SettingsRoute)  ← 相对路由
└── details/:detailId (DetailsRoute)  ← 相对路由（复用）
    └── settings/:settingId (SettingsRoute)  ← 相对路由
```

**关键点**：`detailRoute` 被使用了两次：

1. 作为 `DashboardRoute` 的子路由：`/dashboard/details/:detailId`
2. 作为 `HomeRoute` 的直接子路由：`/details/:detailId`

### 路由数据类

#### DetailsRoute - 相对路由示例

```dart 59:67:example/lib/go_relative.dart
class DetailsRoute extends RelativeGoRouteData with $DetailsRoute {
  const DetailsRoute({required this.detailId});
  final String detailId;

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return DetailsScreen(id: detailId);
  }
}
```

`DetailsRoute` 继承自 `RelativeGoRouteData`（而不是 `GoRouteData`），这使得它可以：

- 使用 `goRelative(context)` 进行相对导航
- 自动根据当前路由上下文解析路径
- 在多个父路由下复用

#### SettingsRoute - 嵌套相对路由

```dart 69:77:example/lib/go_relative.dart
class SettingsRoute extends RelativeGoRouteData with $SettingsRoute {
  const SettingsRoute({required this.settingId});
  final String settingId;

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return SettingsScreen(id: settingId);
  }
}
```

`SettingsRoute` 是 `DetailsRoute` 的子路由，也是一个相对路由。

## 导航方式对比

### 相对导航（推荐）

```dart 92:94:example/lib/go_relative.dart
            onPressed: () {
              const DetailsRoute(detailId: 'DetailsId').goRelative(context);
            },
```

使用 `goRelative(context)` 的优势：

- **上下文感知**：自动根据当前路由解析完整路径
- **代码复用**：同一个 `DetailsRoute` 可以从不同父路由导航
- **维护简单**：如果路由结构改变，不需要修改导航代码

**实际路径解析**：

- 从 `/` 导航：`./details/DetailsId` → `/details/DetailsId`
- 从 `/dashboard` 导航：`./details/DetailsId` → `/dashboard/details/DetailsId`

### 绝对导航

```dart 98:100:example/lib/go_relative.dart
            onPressed: () {
              DashboardRoute().go(context);
            },
```

使用 `go(context)` 进行绝对导航，路径固定为 `/dashboard`。

## UI 组件分析

### HomeScreen - 首页

```dart 80:107:example/lib/go_relative.dart
/// The home screen
class HomeScreen extends StatelessWidget {
  /// Constructs a [HomeScreen]
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Home Screen')),
      body: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: <Widget>[
          ElevatedButton(
            onPressed: () {
              const DetailsRoute(detailId: 'DetailsId').goRelative(context);
            },
            child: const Text('Go to the Details screen'),
          ),
          ElevatedButton(
            onPressed: () {
              DashboardRoute().go(context);
            },
            child: const Text('Go to the Dashboard screen'),
          ),
        ],
      ),
    );
  }
}
```

从首页可以：

1. 导航到 Details 页面（相对导航，路径为 `/details/DetailsId`）
2. 导航到 Dashboard 页面（绝对导航，路径为 `/dashboard`）

### DashboardScreen - 仪表板

```dart 110:134:example/lib/go_relative.dart
/// The home screen
class DashboardScreen extends StatelessWidget {
  /// Constructs a [DashboardScreen]
  const DashboardScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Dashboard Screen')),
      body: Column(
        children: <Widget>[
          ElevatedButton(
            onPressed: () {
              const DetailsRoute(detailId: 'DetailsId').goRelative(context);
            },
            child: const Text('Go to the Details screen'),
          ),
          ElevatedButton(
            onPressed: () => context.pop(),
            child: const Text('Go back'),
          ),
        ],
      ),
    );
  }
}
```

从仪表板可以：

1. 导航到 Details 页面（相对导航，路径为 `/dashboard/details/DetailsId`）
2. 返回上一页

**注意**：虽然使用的是同一个 `DetailsRoute` 实例，但由于使用了 `goRelative`，实际导航的路径会根据当前路由自动调整。

### DetailsScreen - 详情页

```dart 137:165:example/lib/go_relative.dart
/// The details screen
class DetailsScreen extends StatelessWidget {
  /// Constructs a [DetailsScreen]
  const DetailsScreen({super.key, required this.id});

  final String id;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Details Screen $id')),
      body: Center(
        child: Column(
          children: <Widget>[
            ElevatedButton(
              onPressed: () => context.pop(),
              child: const Text('Go back'),
            ),
            ElevatedButton(
              onPressed: () => const SettingsRoute(
                settingId: 'SettingsId',
              ).goRelative(context),
              child: const Text('Go to the Settings screen'),
            ),
          ],
        ),
      ),
    );
  }
}
```

从详情页可以：

1. 返回上一页
2. 导航到 Settings 页面（相对导航）

**路径解析示例**：

- 如果当前在 `/details/DetailsId`，导航到 Settings：`./settings/SettingsId` → `/details/DetailsId/settings/SettingsId`
- 如果当前在 `/dashboard/details/DetailsId`，导航到 Settings：`./settings/SettingsId` → `/dashboard/details/DetailsId/settings/SettingsId`

### SettingsScreen - 设置页

```dart 168:186:example/lib/go_relative.dart
/// The details screen
class SettingsScreen extends StatelessWidget {
  /// Constructs a [SettingsScreen]
  const SettingsScreen({super.key, required this.id});

  final String id;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Settings Screen $id')),
      body: Center(
        child: TextButton(
          onPressed: () => context.pop(),
          child: const Text('Go back'),
        ),
      ),
    );
  }
}
```

设置页只有一个返回按钮，用于返回上一页。

## 相对路由的优势

### 1. 代码复用

同一个相对路由定义可以在多个父路由下使用，避免重复定义：

```dart
// 定义一次
const TypedRelativeGoRoute<DetailsRoute> detailRoute = ...;

// 在多个地方使用
routes: [
  TypedGoRoute<DashboardRoute>(
    routes: [detailRoute],  // 使用
  ),
  detailRoute,  // 复用
]
```

### 2. 上下文感知导航

使用 `goRelative` 时，路由会根据当前上下文自动解析：

```dart
// 从不同位置导航，使用相同的代码
DetailsRoute(detailId: '123').goRelative(context);

// 从 / 导航 → /details/123
// 从 /dashboard 导航 → /dashboard/details/123
```

### 3. 维护性更好

如果路由结构发生变化，相对路由的导航代码不需要修改，因为路径是相对于当前路由解析的。

## 生成的代码分析

查看 `go_relative.g.dart` 可以看到生成的代码实现了：

1. **相对路径计算**：`relativeLocation` 属性使用 `./` 前缀
2. **相对导航方法**：`goRelative`、`pushRelative`、`pushReplacementRelative`、`replaceRelative`
3. **路径解析**：根据当前路由上下文自动构建完整路径

```dart 86:114:example/lib/go_relative.g.dart
mixin $DetailsRoute on RelativeGoRouteData {
  static DetailsRoute _fromState(GoRouterState state) =>
      DetailsRoute(detailId: state.pathParameters['detailId']!);

  DetailsRoute get _self => this as DetailsRoute;

  @override
  String get subLocation => RelativeGoRouteData.$location(
    'details/${Uri.encodeComponent(_self.detailId)}',
  );

  @override
  String get relativeLocation => './$subLocation';

  @override
  void goRelative(BuildContext context) => context.go(relativeLocation);

  @override
  Future<T?> pushRelative<T>(BuildContext context) =>
      context.push<T>(relativeLocation);

  @override
  void pushReplacementRelative(BuildContext context) =>
      context.pushReplacement(relativeLocation);

  @override
  void replaceRelative(BuildContext context) =>
      context.replace(relativeLocation);
}
```

## 使用场景

相对路由特别适用于以下场景：

1. **可复用的子路由**：如详情页、设置页等，可能在多个父路由下出现
2. **模块化路由**：将路由定义模块化，便于在不同模块间复用
3. **动态路由结构**：路由结构可能根据用户权限或配置动态变化

## 总结

这个示例展示了 `go_router_builder` 中相对路由的强大功能：

- **定义**：使用 `TypedRelativeGoRoute` 定义相对路由
- **实现**：路由数据类继承 `RelativeGoRouteData`
- **导航**：使用 `goRelative()` 进行上下文感知的相对导航
- **复用**：同一个相对路由可以在多个父路由下使用
- **优势**：代码更简洁、维护性更好、支持动态路由结构

通过相对路由，你可以构建更加灵活和可维护的路由系统。
