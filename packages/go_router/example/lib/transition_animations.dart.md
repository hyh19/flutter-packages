# transition_animations.dart 代码解析

## 概述

这个文件展示了如何在 Flutter 应用中使用 `go_router` 包实现自定义页面过渡动画。该示例演示了三种不同的过渡动画场景：

1. **基础淡入淡出动画**：使用自定义曲线和时长的淡入淡出效果
2. **可关闭的模态页面**：支持点击遮罩层关闭的对话框式页面
3. **自定义反向过渡时长**：前进和后退使用不同时长的动画

## 文件结构

文件包含以下主要组件：

- 路由配置（`_router`）
- 主应用组件（`MyApp`）
- 首页组件（`HomeScreen`）
- 详情页组件（`DetailsScreen`）
- 可关闭详情页组件（`DismissibleDetails`）

## 路由配置

### 主路由结构

```dart 17:91:example/lib/transition_animations.dart
final GoRouter _router = GoRouter(
  routes: <RouteBase>[
    GoRoute(
      path: '/',
      builder: (BuildContext context, GoRouterState state) {
        return const HomeScreen();
      },
      routes: <RouteBase>[
        // 子路由定义...
      ],
    ),
  ],
);
```

路由配置使用嵌套结构，根路径 `/` 对应首页，三个子路由分别展示不同的过渡动画效果。

### 场景一：基础淡入淡出动画

```dart 25:50:example/lib/transition_animations.dart
GoRoute(
  path: 'details',
  pageBuilder: (BuildContext context, GoRouterState state) {
    return CustomTransitionPage<void>(
      key: state.pageKey,
      child: const DetailsScreen(),
      transitionDuration: const Duration(milliseconds: 150),
      transitionsBuilder:
          (
            BuildContext context,
            Animation<double> animation,
            Animation<double> secondaryAnimation,
            Widget child,
          ) {
            // Change the opacity of the screen using a Curve based on the the animation's
            // value
            return FadeTransition(
              opacity: CurveTween(
                curve: Curves.easeInOut,
              ).animate(animation),
              child: child,
            );
          },
    );
  },
),
```

**关键特性**：

- **`pageBuilder`**：使用 `pageBuilder` 而非 `builder`，这是实现自定义过渡动画的关键
- **`CustomTransitionPage`**：go_router 提供的自定义过渡页面类
- **`state.pageKey`**：使用路由状态中的页面键，确保页面正确管理
- **`transitionDuration`**：设置过渡动画时长为 150 毫秒
- **`transitionsBuilder`**：自定义过渡动画构建器
  - `animation`：主动画对象，从 0.0 到 1.0
  - `secondaryAnimation`：次要动画对象（通常用于前一个页面的动画）
  - `child`：要应用动画的子组件
- **`CurveTween`**：使用 `Curves.easeInOut` 曲线，使动画更自然

### 场景二：可关闭的模态页面

```dart 51:64:example/lib/transition_animations.dart
GoRoute(
  path: 'dismissible-details',
  pageBuilder: (BuildContext context, GoRouterState state) {
    return CustomTransitionPage<void>(
      key: state.pageKey,
      child: const DismissibleDetails(),
      barrierDismissible: true,
      barrierColor: Colors.black38,
      opaque: false,
      transitionDuration: Duration.zero,
      transitionsBuilder: (_, __, ___, Widget child) => child,
    );
  },
),
```

**关键特性**：

- **`barrierDismissible: true`**：允许点击遮罩层关闭页面
- **`barrierColor: Colors.black38`**：设置半透明黑色遮罩
- **`opaque: false`**：页面不完全不透明，允许看到背景
- **`transitionDuration: Duration.zero`**：无过渡动画，立即显示
- **`transitionsBuilder`**：直接返回子组件，不应用任何动画

这种配置常用于实现对话框、底部表单或模态页面。

### 场景三：自定义反向过渡时长

```dart 65:87:example/lib/transition_animations.dart
GoRoute(
  path: 'custom-reverse-transition-duration',
  pageBuilder: (BuildContext context, GoRouterState state) {
    return CustomTransitionPage<void>(
      key: state.pageKey,
      child: const DetailsScreen(),
      barrierDismissible: true,
      barrierColor: Colors.black38,
      opaque: false,
      transitionDuration: const Duration(milliseconds: 500),
      reverseTransitionDuration: const Duration(milliseconds: 200),
      transitionsBuilder:
          (
            BuildContext context,
            Animation<double> animation,
            Animation<double> secondaryAnimation,
            Widget child,
          ) {
            return FadeTransition(opacity: animation, child: child);
          },
    );
  },
),
```

**关键特性**：

- **`transitionDuration`**：前进动画时长为 500 毫秒（较慢）
- **`reverseTransitionDuration`**：后退动画时长为 200 毫秒（较快）
- **`FadeTransition`**：直接使用 `animation`，不使用曲线，实现简单的淡入淡出

这种配置让用户进入页面时看到较慢的动画，退出时快速关闭，提供更好的用户体验。

## 应用组件

### MyApp

```dart 94:102:example/lib/transition_animations.dart
class MyApp extends StatelessWidget {
  /// Constructs a [MyApp]
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(routerConfig: _router);
  }
}
```

标准的 Flutter 应用入口，使用 `MaterialApp.router` 并传入路由配置。

### HomeScreen

```dart 105:139:example/lib/transition_animations.dart
class HomeScreen extends StatelessWidget {
  /// Constructs a [HomeScreen]
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Home Screen')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            ElevatedButton(
              onPressed: () => context.go('/details'),
              child: const Text('Go to the Details screen'),
            ),
            const SizedBox(height: 48),
            ElevatedButton(
              onPressed: () => context.go('/dismissible-details'),
              child: const Text('Go to the Dismissible Details screen'),
            ),
            const SizedBox(height: 48),
            ElevatedButton(
              onPressed: () =>
                  context.go('/custom-reverse-transition-duration'),
              child: const Text(
                'Go to the Custom Reverse Transition Duration Screen',
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

首页包含三个按钮，分别导航到三个不同的详情页面，展示不同的过渡动画效果。使用 `context.go()` 进行导航。

### DetailsScreen

```dart 142:163:example/lib/transition_animations.dart
class DetailsScreen extends StatelessWidget {
  /// Constructs a [DetailsScreen]
  const DetailsScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Details Screen')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            ElevatedButton(
              onPressed: () => context.go('/'),
              child: const Text('Go back to the Home screen'),
            ),
          ],
        ),
      ),
    );
  }
}
```

标准的详情页面，包含返回首页的按钮。

### DismissibleDetails

```dart 166:177:example/lib/transition_animations.dart
class DismissibleDetails extends StatelessWidget {
  /// Constructs a [DismissibleDetails]
  const DismissibleDetails({super.key});

  @override
  Widget build(BuildContext context) {
    return const Padding(
      padding: EdgeInsets.all(48),
      child: ColoredBox(color: Colors.red),
    );
  }
}
```

一个简单的红色方块组件，用于演示可关闭的模态页面效果。由于设置了 `opaque: false` 和遮罩层，这个组件会显示为一个浮动的红色方块。

## 核心概念

### CustomTransitionPage 参数说明

- **`key`**：必须使用 `state.pageKey`，确保页面正确管理
- **`child`**：要显示的页面内容
- **`transitionDuration`**：前进动画时长
- **`reverseTransitionDuration`**：后退动画时长（可选）
- **`transitionsBuilder`**：自定义动画构建器函数
- **`barrierDismissible`**：是否允许点击遮罩层关闭（默认 `false`）
- **`barrierColor`**：遮罩层颜色（默认 `null`）
- **`opaque`**：页面是否不透明（默认 `true`）

### 动画参数

`transitionsBuilder` 函数接收四个参数：

1. **`BuildContext context`**：构建上下文
2. **`Animation<double> animation`**：主动画，值从 0.0 到 1.0
3. **`Animation<double> secondaryAnimation`**：次要动画，通常用于前一个页面
4. **`Widget child`**：要应用动画的子组件

### 常用过渡效果

除了示例中的 `FadeTransition`，还可以使用：

- **`SlideTransition`**：滑动过渡
- **`ScaleTransition`**：缩放过渡
- **`RotationTransition`**：旋转过渡
- **`SizeTransition`**：大小过渡
- 或者组合多个过渡效果

## 使用建议

1. **性能考虑**：过渡动画时长不宜过长，通常 200-500 毫秒为宜
2. **用户体验**：前进和后退可以使用不同的动画时长，后退通常更快
3. **曲线选择**：根据场景选择合适的动画曲线（`Curves.easeInOut`、`Curves.easeOut` 等）
4. **模态页面**：对于对话框式页面，使用 `barrierDismissible` 和 `barrierColor` 提供更好的交互体验

## 相关资源

- [Flutter 动画介绍](https://docs.flutter.dev/development/ui/animations)
- [go_router 文档](https://pub.dev/documentation/go_router/latest/)
- [CustomTransitionPage API 文档](https://pub.dev/documentation/go_router/latest/go_router/CustomTransitionPage-class.html)
