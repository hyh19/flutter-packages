# on_exit_example.dart 代码详解

## 概述

这个示例文件展示了如何在 `go_router_builder` 中使用 `onExit` 功能。`onExit` 是 `GoRouteData` 类的一个可选方法，允许在用户尝试离开某个路由页面时进行拦截，通常用于显示确认对话框，防止用户意外离开（例如，表单有未保存的数据时）。

## 文件结构

文件主要包含以下组件：

1. **应用入口**：`main` 函数和 `App` 类
2. **路由定义**：`HomeRoute` 和 `SubRoute` 类
3. **UI 组件**：`HomeScreen` 和 `SubScreen` 类
4. **代码生成**：通过 `part` 指令引用生成的代码

## 代码详解

### 导入和代码生成

```dart 7:10:example/lib/on_exit_example.dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

part 'on_exit_example.g.dart';
```

- 导入了 Flutter 的 Material 组件库和 `go_router` 包
- `part` 指令用于引用代码生成器生成的 `on_exit_example.g.dart` 文件，该文件包含路由配置的生成代码

### 应用入口

```dart 12:22:example/lib/on_exit_example.dart
void main() => runApp(App());

class App extends StatelessWidget {
  App({super.key});

  @override
  Widget build(BuildContext context) =>
      MaterialApp.router(routerConfig: _router, title: _appTitle);

  final GoRouter _router = GoRouter(routes: $appRoutes);
}
```

`App` 类负责：

- 创建应用的根组件
- 使用 `MaterialApp.router` 配置路由（`routerConfig` 参数）
- 通过 `$appRoutes`（由代码生成器生成）初始化 `GoRouter` 实例

### 路由定义

#### HomeRoute（主路由）

```dart 24:35:example/lib/on_exit_example.dart
@TypedGoRoute<HomeRoute>(
  path: '/',
  routes: <TypedGoRoute<GoRouteData>>[
    TypedGoRoute<SubRoute>(path: 'sub-route'),
  ],
)
class HomeRoute extends GoRouteData with $HomeRoute {
  const HomeRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) => const HomeScreen();
}
```

`HomeRoute` 的特点：

- 使用 `@TypedGoRoute` 注解定义路由，路径为 `/`
- 在 `routes` 参数中定义子路由 `SubRoute`（相对路径 `sub-route`，完整路径为 `/sub-route`）
- 继承 `GoRouteData` 并混入生成的 `$HomeRoute` mixin
- 实现 `build` 方法返回 `HomeScreen` 组件

#### SubRoute（子路由）- 核心功能

```dart 37:63:example/lib/on_exit_example.dart
class SubRoute extends GoRouteData with $SubRoute {
  const SubRoute();

  @override
  Future<bool> onExit(BuildContext context, GoRouterState state) async {
    final bool? confirmed = await showDialog<bool>(
      context: context,
      builder: (_) => AlertDialog(
        content: const Text('Are you sure to leave this page?'),
        actions: <Widget>[
          TextButton(
            onPressed: () => Navigator.of(context).pop(false),
            child: const Text('Cancel'),
          ),
          ElevatedButton(
            onPressed: () => Navigator.of(context).pop(true),
            child: const Text('Confirm'),
          ),
        ],
      ),
    );
    return confirmed ?? false;
  }

  @override
  Widget build(BuildContext context, GoRouterState state) => const SubScreen();
}
```

`SubRoute` 实现了 `onExit` 方法，这是本示例的核心功能：

**方法签名**：

- 返回类型：`Future<bool>`，异步方法
- 参数：`BuildContext context` 和 `GoRouterState state`
- 返回值：
  - `true`：允许离开页面
  - `false`：阻止离开页面

**实现逻辑**：

1. 显示确认对话框：使用 `showDialog` 显示一个 `AlertDialog`
2. 对话框内容：
   - 提示文本："Are you sure to leave this page?"
   - 两个按钮：
     - **Cancel**：点击时通过 `Navigator.of(context).pop(false)` 关闭对话框并返回 `false`
     - **Confirm**：点击时通过 `Navigator.of(context).pop(true)` 关闭对话框并返回 `true`
3. 返回值处理：使用 `confirmed ?? false` 确保如果对话框被取消（返回 `null`），则返回 `false` 阻止离开

**工作原理**：

当用户尝试离开 `SubRoute` 页面时（例如，点击返回按钮或导航到其他路由），`go_router` 会调用 `onExit` 方法。如果方法返回 `false`，导航操作将被阻止；如果返回 `true`，导航操作正常进行。

### UI 组件

#### HomeScreen

```dart 65:78:example/lib/on_exit_example.dart
class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) => Scaffold(
    appBar: AppBar(title: const Text(_appTitle)),
    body: Center(
      child: ElevatedButton(
        onPressed: () => const SubRoute().go(context),
        child: const Text('Go to sub screen'),
      ),
    ),
  );
}
```

`HomeScreen` 是主页面：

- 包含一个标题栏显示应用标题
- 中心位置有一个按钮，点击时调用 `const SubRoute().go(context)` 导航到子路由页面
- 使用类型安全的路由导航方式（`.go()` 方法由生成的 mixin 提供）

#### SubScreen

```dart 80:93:example/lib/on_exit_example.dart
class SubScreen extends StatelessWidget {
  const SubScreen({super.key});

  @override
  Widget build(BuildContext context) => Scaffold(
    appBar: AppBar(title: const Text('$_appTitle Sub screen')),
    body: Center(
      child: ElevatedButton(
        onPressed: () => Navigator.of(context).pop(),
        child: const Text('Go back'),
      ),
    ),
  );
}
```

`SubScreen` 是子页面：

- 显示带有 "Sub screen" 后缀的标题
- 有一个返回按钮，使用 `Navigator.of(context).pop()` 返回上一页
- 当用户点击返回按钮时，会触发 `SubRoute` 的 `onExit` 方法

### 应用标题常量

```dart 95:95:example/lib/on_exit_example.dart
const String _appTitle = 'GoRouter Example: builder';
```

定义了一个私有常量用于存储应用标题，在多个地方使用以保持一致性。

## 工作流程

1. **应用启动**：
   - `main` 函数启动 `App` 组件
   - `App` 创建 `GoRouter` 实例，使用生成的 `$appRoutes` 配置路由

2. **显示主页面**：
   - 初始路由为 `/`，显示 `HomeScreen`

3. **导航到子页面**：
   - 用户点击 "Go to sub screen" 按钮
   - 调用 `const SubRoute().go(context)` 导航到 `/sub-route`
   - 显示 `SubScreen`

4. **尝试离开子页面**：
   - 用户点击返回按钮或进行其他导航操作
   - `go_router` 检测到即将离开 `SubRoute`
   - 调用 `SubRoute.onExit` 方法

5. **显示确认对话框**：
   - `onExit` 方法显示确认对话框
   - 用户可以选择：
     - **Cancel**：返回 `false`，阻止导航，留在当前页面
     - **Confirm**：返回 `true`，允许导航，返回到上一页

## 关键概念

### onExit 的使用场景

`onExit` 方法适用于以下场景：

- **表单未保存**：用户填写了表单但未保存，离开前需要确认
- **正在处理中**：某个操作正在进行，离开可能导致数据丢失
- **重要操作确认**：需要用户明确确认才能离开的敏感页面

### onExit 的返回值

- `true`：允许导航继续进行
- `false`：阻止导航，用户留在当前页面
- 如果对话框被取消（返回 `null`），代码使用 `?? false` 默认返回 `false`，这是一种安全的处理方式

### 路由嵌套

本示例展示了路由嵌套的使用：

- `HomeRoute` 是父路由（路径 `/`）
- `SubRoute` 是子路由（相对路径 `sub-route`，完整路径 `/sub-route`）
- 子路由在父路由的 `routes` 参数中定义
- 这种嵌套结构使得路由组织更加清晰和模块化

### 类型安全的路由导航

通过 `go_router_builder` 代码生成：

- 路由类提供了类型安全的导航方法（如 `.go()`、`.push()` 等）
- 编译时检查路由参数和路径的正确性
- 提供更好的 IDE 自动完成支持

## 总结

这个示例完整展示了 `onExit` 功能的实现和使用方式。通过实现 `onExit` 方法，开发者可以在用户离开页面时进行拦截，显示确认对话框，从而防止用户意外离开重要的页面或丢失未保存的数据。这是 Flutter 应用中常见的用户体验优化手段。
