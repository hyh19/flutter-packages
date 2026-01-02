# GoRouter 初始位置示例代码解析

## 概述

这个示例展示了如何在 Flutter 应用中使用 GoRouter 的 `initialLocation` 功能来设置应用的初始路由位置。默认情况下，应用启动时会导航到根路径 `/`，但通过 `initialLocation` 参数，可以指定应用启动时直接导航到其他路由。

## 代码结构

### 应用入口

```dart 8:8:example/lib/others/init_loc.dart
void main() => runApp(App());
```

应用入口函数非常简单，直接运行 `App` 组件。

### 主应用组件

```dart 11:42:example/lib/others/init_loc.dart
/// The main app.
class App extends StatelessWidget {
  /// Creates an [App].
  App({super.key});

  /// The title of the app.
  static const String title = 'GoRouter Example: Initial Location';

  @override
  Widget build(BuildContext context) =>
      MaterialApp.router(routerConfig: _router, title: title);

  final GoRouter _router = GoRouter(
    initialLocation: '/page3',
    routes: <GoRoute>[
      GoRoute(
        path: '/',
        builder: (BuildContext context, GoRouterState state) =>
            const Page1Screen(),
      ),
      GoRoute(
        path: '/page2',
        builder: (BuildContext context, GoRouterState state) =>
            const Page2Screen(),
      ),
      GoRoute(
        path: '/page3',
        builder: (BuildContext context, GoRouterState state) =>
            const Page3Screen(),
      ),
    ],
  );
}
```

`App` 类是整个应用的核心组件，主要特点：

1. **使用 MaterialApp.router**：通过 `MaterialApp.router` 构造函数配置路由，这是使用 GoRouter 的标准方式。

2. **initialLocation 参数**：这是本示例的关键特性。`initialLocation: '/page3'` 指定应用启动时直接导航到 `/page3` 路由，而不是默认的根路径 `/`。

3. **路由配置**：定义了三个路由：
   - `/`：对应 `Page1Screen`
   - `/page2`：对应 `Page2Screen`
   - `/page3`：对应 `Page3Screen`

## 页面组件

示例中定义了三个页面组件，它们的结构非常相似，都遵循相同的模式。

### Page1Screen

```dart 45:64:example/lib/others/init_loc.dart
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
            onPressed: () => context.go('/page2'),
            child: const Text('Go to page 2'),
          ),
        ],
      ),
    ),
  );
}
```

第一个页面包含一个按钮，点击后导航到 `/page2`。

### Page2Screen

```dart 67:86:example/lib/others/init_loc.dart
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
            onPressed: () => context.go('/'),
            child: const Text('Go to home page'),
          ),
        ],
      ),
    ),
  );
}
```

第二个页面包含一个按钮，点击后导航回根路径 `/`。

### Page3Screen

```dart 89:108:example/lib/others/init_loc.dart
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
            onPressed: () => context.go('/page2'),
            child: const Text('Go to page 2'),
          ),
        ],
      ),
    ),
  );
}
```

第三个页面是应用的初始页面（由于 `initialLocation: '/page3'` 的设置），包含一个按钮，点击后导航到 `/page2`。

## 核心功能说明

### initialLocation 的作用

`initialLocation` 参数允许你指定应用启动时的初始路由位置。这在以下场景中非常有用：

1. **深度链接**：当用户通过外部链接打开应用时，可以导航到特定页面
2. **状态恢复**：恢复应用之前的状态，导航到用户上次访问的页面
3. **条件导航**：根据某些条件（如用户登录状态）决定初始页面
4. **开发调试**：在开发过程中快速跳转到特定页面进行测试

### 路由导航

示例中使用了 `context.go()` 方法进行路由导航：

```dart 57:57:example/lib/others/init_loc.dart
            onPressed: () => context.go('/page2'),
```

`context.go()` 是 GoRouter 提供的扩展方法，用于导航到指定路径。它会替换当前的路由栈，而不是在栈上添加新路由。

## 运行效果

当应用启动时：

1. 由于 `initialLocation: '/page3'` 的设置，应用会直接显示 `Page3Screen`
2. 用户可以通过按钮在不同页面之间导航
3. 每个页面都有清晰的导航路径，形成简单的页面流转

## 使用场景

这个示例适用于以下场景：

- 需要设置非默认首页的应用
- 实现深度链接功能
- 根据用户状态动态设置初始页面
- 在开发过程中快速测试特定页面

## 总结

这个示例简洁地展示了 GoRouter 的 `initialLocation` 功能，通过设置初始位置，可以让应用在启动时直接导航到指定的路由，而不是默认的根路径。这对于实现深度链接、状态恢复等功能非常有用。
