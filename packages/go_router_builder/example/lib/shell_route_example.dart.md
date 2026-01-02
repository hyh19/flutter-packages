# Shell Route 示例代码解析

本文档详细解析 `shell_route_example.dart` 文件，该文件展示了如何使用 `go_router_builder` 实现 Shell Route 功能。Shell Route 是 go_router 提供的一种特殊路由类型，允许在多个子路由之间共享一个公共的 UI 容器（如底部导航栏）。

## 概述

这个示例演示了以下核心概念：

1. **Shell Route 的定义**：使用 `@TypedShellRoute` 注解创建 Shell Route
2. **嵌套路由**：在 Shell Route 中定义多个子路由
3. **共享 UI 容器**：通过 Shell Route 的 `builder` 方法创建共享的 UI 结构
4. **底部导航栏实现**：在 Shell Route 中实现底部导航栏，实现不同路由之间的切换
5. **混合路由**：同时使用 Shell Route 和普通 Route

## 应用入口与路由配置

应用入口非常简单，创建了一个 `App` widget，使用 `MaterialApp.router` 配置路由：

```dart 12:25:example/lib/shell_route_example.dart
void main() => runApp(App());

class App extends StatelessWidget {
  App({super.key});

  @override
  Widget build(BuildContext context) =>
      MaterialApp.router(routerConfig: _router);

  final GoRouter _router = GoRouter(
    routes: $appRoutes,
    initialLocation: '/foo',
  );
}
```

关键点：

- `$appRoutes` 是由代码生成器自动生成的，包含了所有通过注解定义的路由
- `initialLocation: '/foo'` 设置应用启动时的初始路由为 `/foo`，这个路由位于 Shell Route 内部

## Shell Route 定义

Shell Route 是示例的核心部分，它定义了一个包含两个子路由的 Shell 容器：

```dart 35:48:example/lib/shell_route_example.dart
@TypedShellRoute<MyShellRouteData>(
  routes: <TypedRoute<RouteData>>[
    TypedGoRoute<FooRouteData>(path: '/foo'),
    TypedGoRoute<BarRouteData>(path: '/bar'),
  ],
)
class MyShellRouteData extends ShellRouteData {
  const MyShellRouteData();

  @override
  Widget builder(BuildContext context, GoRouterState state, Widget navigator) {
    return MyShellRouteScreen(child: navigator);
  }
}
```

### 注解说明

- `@TypedShellRoute<MyShellRouteData>`：标记这是一个 Shell Route，类型参数是路由数据类
- `routes`：定义 Shell Route 包含的子路由列表
  - `TypedGoRoute<FooRouteData>(path: '/foo')`：定义路径为 `/foo` 的子路由
  - `TypedGoRoute<BarRouteData>(path: '/bar')`：定义路径为 `/bar` 的子路由

### ShellRouteData 类

`MyShellRouteData` 继承自 `ShellRouteData`，需要实现 `builder` 方法：

- `context`：BuildContext，用于访问 Flutter 上下文
- `state`：GoRouterState，包含当前路由状态信息
- `navigator`：Widget，这是子路由渲染的内容，需要将其嵌入到 Shell 的 UI 结构中

在这个示例中，`builder` 方法将 `navigator` 传递给 `MyShellRouteScreen`，由它来构建包含底部导航栏的完整 UI。

## 子路由定义

Shell Route 包含两个子路由：`FooRouteData` 和 `BarRouteData`。

### FooRouteData

```dart 50:57:example/lib/shell_route_example.dart
class FooRouteData extends GoRouteData with $FooRouteData {
  const FooRouteData();

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return const FooScreen();
  }
}
```

### BarRouteData

```dart 59:66:example/lib/shell_route_example.dart
class BarRouteData extends GoRouteData with $BarRouteData {
  const BarRouteData();

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return const BarScreen();
  }
}
```

这两个路由类都：

- 继承自 `GoRouteData` 并混入对应的生成 mixin（`$FooRouteData` 或 `$BarRouteData`）
- 实现 `build` 方法返回对应的屏幕组件
- 使用 `const` 构造函数，因为它们是纯数据类

## Shell Route 屏幕实现

`MyShellRouteScreen` 是 Shell Route 的 UI 容器，它实现了底部导航栏功能：

```dart 68:103:example/lib/shell_route_example.dart
class MyShellRouteScreen extends StatelessWidget {
  const MyShellRouteScreen({required this.child, super.key});

  final Widget child;

  int getCurrentIndex(BuildContext context) {
    final String location = GoRouterState.of(context).uri.path;
    if (location == '/bar') {
      return 1;
    }
    return 0;
  }

  @override
  Widget build(BuildContext context) {
    final int currentIndex = getCurrentIndex(context);
    return Scaffold(
      body: child,
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: currentIndex,
        items: const <BottomNavigationBarItem>[
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Foo'),
          BottomNavigationBarItem(icon: Icon(Icons.business), label: 'Bar'),
        ],
        onTap: (int index) {
          switch (index) {
            case 0:
              const FooRouteData().go(context);
            case 1:
              const BarRouteData().go(context);
          }
        },
      ),
    );
  }
}
```

### 关键实现细节

1. **接收子路由内容**：通过 `child` 参数接收 Shell Route 的 `navigator`，这是当前激活的子路由渲染的内容

2. **获取当前路由索引**：`getCurrentIndex` 方法通过 `GoRouterState.of(context).uri.path` 获取当前路径，判断应该高亮哪个底部导航项
   - `/bar` 路径对应索引 1
   - 其他路径（主要是 `/foo`）对应索引 0

3. **底部导航栏**：
   - `currentIndex`：根据当前路径动态设置
   - `items`：定义两个导航项（Foo 和 Bar）
   - `onTap`：点击导航项时，使用类型安全的路由数据类进行导航
     - 点击索引 0：调用 `FooRouteData().go(context)` 导航到 `/foo`
     - 点击索引 1：调用 `BarRouteData().go(context)` 导航到 `/bar`

### 导航方式

示例中使用了 `go` 方法进行导航，这是 go_router 提供的导航方法之一：

- `go(context)`：替换当前路由栈，导航到新路由
- 其他可用方法（由代码生成器自动生成）：
  - `push(context)`：将新路由推入导航栈
  - `pushReplacement(context)`：替换当前路由
  - `replace(context)`：替换路由

## 屏幕组件

示例中定义了三个简单的屏幕组件：

### FooScreen

```dart 105:112:example/lib/shell_route_example.dart
class FooScreen extends StatelessWidget {
  const FooScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return const Text('Foo');
  }
}
```

### BarScreen

```dart 114:121:example/lib/shell_route_example.dart
class BarScreen extends StatelessWidget {
  const BarScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return const Text('Bar');
  }
}
```

这两个屏幕组件非常简单，仅显示文本。在实际应用中，它们可以包含完整的页面内容。

## 独立路由示例

除了 Shell Route，示例还展示了一个独立的普通路由：

```dart 123:130:example/lib/shell_route_example.dart
@TypedGoRoute<LoginRoute>(path: '/login')
class LoginRoute extends GoRouteData with $LoginRoute {
  const LoginRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) =>
      const LoginScreen();
}
```

这个路由：

- 使用 `@TypedGoRoute` 注解（不是 `@TypedShellRoute`）
- 路径为 `/login`，不在任何 Shell Route 内部
- 访问 `/login` 时不会显示底部导航栏，因为它不在 Shell Route 的容器内

### LoginScreen

```dart 132:139:example/lib/shell_route_example.dart
class LoginScreen extends StatelessWidget {
  const LoginScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return const Text('Login');
  }
}
```

## 代码生成

文件开头引用了生成的代码：

```dart 10:10:example/lib/shell_route_example.dart
part 'shell_route_example.g.dart';
```

代码生成器会根据注解自动生成：

1. **路由列表**：`$appRoutes` 包含所有定义的路由
2. **路由工厂方法**：每个路由数据类都有对应的 `_fromState` 工厂方法
3. **导航方法 mixin**：为每个路由数据类生成包含 `go`、`push` 等导航方法的 mixin

生成的代码结构大致如下：

- `$appRoutes`：包含 `$myShellRouteData` 和 `$loginRoute`
- `$myShellRouteData`：Shell Route 定义，包含两个子路由
- `$FooRouteData`、`$BarRouteData`、`$LoginRoute`：各自的导航方法 mixin

## 使用场景

Shell Route 特别适用于以下场景：

1. **底部导航栏应用**：多个主要功能模块共享底部导航栏
2. **侧边栏导航**：桌面应用中的侧边栏导航结构
3. **持久化 UI 元素**：需要在多个页面间保持可见的 UI 元素（如播放器控件、购物车图标等）
4. **嵌套导航结构**：复杂的导航层级结构

## 关键要点总结

1. **Shell Route 定义**：使用 `@TypedShellRoute` 注解，在 `routes` 参数中定义子路由
2. **ShellRouteData 类**：继承 `ShellRouteData`，实现 `builder` 方法，接收 `navigator` 参数
3. **子路由**：子路由使用 `TypedGoRoute` 定义，继承 `GoRouteData`
4. **UI 容器**：在 `builder` 方法中创建包含 `navigator` 的 UI 结构
5. **类型安全导航**：使用路由数据类的 `go`、`push` 等方法进行类型安全的导航
6. **混合使用**：可以在同一个应用中同时使用 Shell Route 和普通 Route

## 未使用的代码

文件中定义了一个 `HomeScreen` 类，但在示例中并未使用：

```dart 27:33:example/lib/shell_route_example.dart
class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) =>
      Scaffold(appBar: AppBar(title: const Text('foo')));
}
```

这可能是为了演示目的保留的，或者是从其他示例中复制过来的代码。
