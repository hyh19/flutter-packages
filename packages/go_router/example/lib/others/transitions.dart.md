# transitions.dart 代码解析

这是一个展示如何在 GoRouter 中使用自定义页面过渡动画的完整示例应用。该示例演示了多种过渡效果，包括淡入淡出、缩放、滑动、旋转以及无过渡效果。

## 文件概述

该文件包含一个 Flutter 应用的完整实现，展示了如何使用 `CustomTransitionPage` 和 `NoTransitionPage` 来创建自定义的路由过渡动画。应用提供了多个不同的路由，每个路由都使用不同的过渡动画效果。

## 主要组件

### App 类

```dart 11:118:example/lib/others/transitions.dart
/// The main app.
class App extends StatelessWidget {
  /// Creates an [App].
  App({super.key});

  /// The title of the app.
  static const String title = 'GoRouter Example: Custom Transitions';

  @override
  Widget build(BuildContext context) =>
      MaterialApp.router(routerConfig: _router, title: title);

  final GoRouter _router = GoRouter(
    routes: <GoRoute>[
      GoRoute(path: '/', redirect: (_, __) => '/none'),
      GoRoute(
        path: '/fade',
        pageBuilder: (BuildContext context, GoRouterState state) =>
            CustomTransitionPage<void>(
              key: state.pageKey,
              child: const ExampleTransitionsScreen(
                kind: 'fade',
                color: Colors.red,
              ),
              transitionsBuilder:
                  (
                    BuildContext context,
                    Animation<double> animation,
                    Animation<double> secondaryAnimation,
                    Widget child,
                  ) => FadeTransition(opacity: animation, child: child),
            ),
      ),
      GoRoute(
        path: '/scale',
        pageBuilder: (BuildContext context, GoRouterState state) =>
            CustomTransitionPage<void>(
              key: state.pageKey,
              child: const ExampleTransitionsScreen(
                kind: 'scale',
                color: Colors.green,
              ),
              transitionsBuilder:
                  (
                    BuildContext context,
                    Animation<double> animation,
                    Animation<double> secondaryAnimation,
                    Widget child,
                  ) => ScaleTransition(scale: animation, child: child),
            ),
      ),
      GoRoute(
        path: '/slide',
        pageBuilder: (BuildContext context, GoRouterState state) =>
            CustomTransitionPage<void>(
              key: state.pageKey,
              child: const ExampleTransitionsScreen(
                kind: 'slide',
                color: Colors.yellow,
              ),
              transitionsBuilder:
                  (
                    BuildContext context,
                    Animation<double> animation,
                    Animation<double> secondaryAnimation,
                    Widget child,
                  ) => SlideTransition(
                    position: animation.drive(
                      Tween<Offset>(
                        begin: const Offset(0.25, 0.25),
                        end: Offset.zero,
                      ).chain(CurveTween(curve: Curves.easeIn)),
                    ),
                    child: child,
                  ),
            ),
      ),
      GoRoute(
        path: '/rotation',
        pageBuilder: (BuildContext context, GoRouterState state) =>
            CustomTransitionPage<void>(
              key: state.pageKey,
              child: const ExampleTransitionsScreen(
                kind: 'rotation',
                color: Colors.purple,
              ),
              transitionsBuilder:
                  (
                    BuildContext context,
                    Animation<double> animation,
                    Animation<double> secondaryAnimation,
                    Widget child,
                  ) => RotationTransition(turns: animation, child: child),
            ),
      ),
      GoRoute(
        path: '/none',
        pageBuilder: (BuildContext context, GoRouterState state) =>
            NoTransitionPage<void>(
              key: state.pageKey,
              child: const ExampleTransitionsScreen(
                kind: 'none',
                color: Colors.white,
              ),
            ),
      ),
    ],
  );
}
```

`App` 类是应用的主入口，负责配置路由系统。它包含以下关键特性：

- **路由配置**：定义了 6 个路由（包括根路径重定向）
- **默认路由**：根路径 `/` 重定向到 `/none`
- **自定义过渡**：每个路由使用 `CustomTransitionPage` 或 `NoTransitionPage` 来定义过渡效果

#### 路由类型详解

##### 1. 淡入淡出过渡（Fade Transition）

```dart 25:42:example/lib/others/transitions.dart
      GoRoute(
        path: '/fade',
        pageBuilder: (BuildContext context, GoRouterState state) =>
            CustomTransitionPage<void>(
              key: state.pageKey,
              child: const ExampleTransitionsScreen(
                kind: 'fade',
                color: Colors.red,
              ),
              transitionsBuilder:
                  (
                    BuildContext context,
                    Animation<double> animation,
                    Animation<double> secondaryAnimation,
                    Widget child,
                  ) => FadeTransition(opacity: animation, child: child),
            ),
      ),
```

使用 `FadeTransition` 实现淡入淡出效果。`animation` 参数控制透明度从 0.0（完全透明）到 1.0（完全不透明）的变化。

##### 2. 缩放过渡（Scale Transition）

```dart 43:60:example/lib/others/transitions.dart
      GoRoute(
        path: '/scale',
        pageBuilder: (BuildContext context, GoRouterState state) =>
            CustomTransitionPage<void>(
              key: state.pageKey,
              child: const ExampleTransitionsScreen(
                kind: 'scale',
                color: Colors.green,
              ),
              transitionsBuilder:
                  (
                    BuildContext context,
                    Animation<double> animation,
                    Animation<double> secondaryAnimation,
                    Widget child,
                  ) => ScaleTransition(scale: animation, child: child),
            ),
      ),
```

使用 `ScaleTransition` 实现缩放效果。子组件从缩放因子 0.0（完全缩小）过渡到 1.0（原始大小）。

##### 3. 滑动过渡（Slide Transition）

```dart 61:86:example/lib/others/transitions.dart
      GoRoute(
        path: '/slide',
        pageBuilder: (BuildContext context, GoRouterState state) =>
            CustomTransitionPage<void>(
              key: state.pageKey,
              child: const ExampleTransitionsScreen(
                kind: 'slide',
                color: Colors.yellow,
              ),
              transitionsBuilder:
                  (
                    BuildContext context,
                    Animation<double> animation,
                    Animation<double> secondaryAnimation,
                    Widget child,
                  ) => SlideTransition(
                    position: animation.drive(
                      Tween<Offset>(
                        begin: const Offset(0.25, 0.25),
                        end: Offset.zero,
                      ).chain(CurveTween(curve: Curves.easeIn)),
                    ),
                    child: child,
                  ),
            ),
      ),
```

使用 `SlideTransition` 实现滑动效果，并配合 `Tween` 和 `CurveTween` 来定制动画：

- **起始位置**：`Offset(0.25, 0.25)` 表示从屏幕的 25% 位置开始
- **结束位置**：`Offset.zero` 表示最终位置在屏幕原点（正常位置）
- **动画曲线**：`Curves.easeIn` 提供缓入效果

##### 4. 旋转过渡（Rotation Transition）

```dart 87:104:example/lib/others/transitions.dart
      GoRoute(
        path: '/rotation',
        pageBuilder: (BuildContext context, GoRouterState state) =>
            CustomTransitionPage<void>(
              key: state.pageKey,
              child: const ExampleTransitionsScreen(
                kind: 'rotation',
                color: Colors.purple,
              ),
              transitionsBuilder:
                  (
                    BuildContext context,
                    Animation<double> animation,
                    Animation<double> secondaryAnimation,
                    Widget child,
                  ) => RotationTransition(turns: animation, child: child),
            ),
      ),
```

使用 `RotationTransition` 实现旋转效果。`turns` 参数控制旋转圈数，从 0.0 到 1.0（完整旋转一圈）。

##### 5. 无过渡效果（No Transition）

```dart 105:115:example/lib/others/transitions.dart
      GoRoute(
        path: '/none',
        pageBuilder: (BuildContext context, GoRouterState state) =>
            NoTransitionPage<void>(
              key: state.pageKey,
              child: const ExampleTransitionsScreen(
                kind: 'none',
                color: Colors.white,
              ),
            ),
      ),
```

使用 `NoTransitionPage` 实现无过渡效果，页面切换会立即完成，没有任何动画。这对于某些需要即时响应的场景很有用。

### ExampleTransitionsScreen 类

```dart 120:166:example/lib/others/transitions.dart
/// An Example transitions screen.
class ExampleTransitionsScreen extends StatelessWidget {
  /// Creates an [ExampleTransitionsScreen].
  const ExampleTransitionsScreen({
    required this.color,
    required this.kind,
    super.key,
  });

  /// The available transition kinds.
  static final List<String> kinds = <String>[
    'fade',
    'scale',
    'slide',
    'rotation',
    'none',
  ];

  /// The color of the container.
  final Color color;

  /// The transition kind.
  final String kind;

  @override
  Widget build(BuildContext context) => Scaffold(
    appBar: AppBar(title: Text('${App.title}: $kind')),
    body: ColoredBox(
      color: color,
      child: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            for (final String kind in kinds)
              Padding(
                padding: const EdgeInsets.all(8),
                child: ElevatedButton(
                  onPressed: () => context.go('/$kind'),
                  child: Text('$kind transition'),
                ),
              ),
          ],
        ),
      ),
    ),
  );
}
```

`ExampleTransitionsScreen` 是一个展示屏幕，提供以下功能：

- **动态标题**：显示当前使用的过渡类型
- **颜色标识**：使用不同颜色区分不同的过渡类型
- **导航按钮**：提供所有可用的过渡类型按钮，用户可以点击切换

#### 关键特性

1. **静态过渡类型列表**：定义了所有可用的过渡类型
2. **颜色属性**：每个屏幕使用不同的背景颜色来区分
3. **循环生成按钮**：使用 `for` 循环生成所有过渡类型的导航按钮
4. **导航方法**：使用 `context.go()` 进行路由导航

## transitionsBuilder 参数详解

`transitionsBuilder` 是一个函数，接受以下参数：

1. **`BuildContext context`**：构建上下文
2. **`Animation<double> animation`**：主动画对象，值从 0.0 到 1.0
3. **`Animation<double> secondaryAnimation`**：次要动画对象，用于处理返回动画
4. **`Widget child`**：要进行过渡的子组件

### 动画生命周期

- **进入时**：`animation` 从 0.0 到 1.0
- **退出时**：`animation` 从 1.0 到 0.0（反向动画）

## 使用场景

这个示例展示了以下使用场景：

1. **自定义过渡效果**：替换默认的 Material 或 Cupertino 过渡动画
2. **多种过渡类型**：在同一应用中使用多种不同的过渡效果
3. **组合动画**：可以通过组合多个 `Transition` 组件来创建复杂的动画效果
4. **性能优化**：使用 `NoTransitionPage` 在需要时禁用动画以提升性能

## 扩展建议

基于这个示例，你可以：

1. **组合多个过渡**：将多个 `Transition` 组件嵌套，创建更复杂的动画效果
2. **自定义动画曲线**：使用不同的 `Curve` 来调整动画的节奏感
3. **添加动画持续时间**：通过 `CustomTransitionPage` 的 `transitionDuration` 参数自定义动画时长
4. **响应式设计**：根据屏幕大小或设备类型选择不同的过渡效果

## 注意事项

1. **页面键值**：使用 `state.pageKey` 确保每个页面都有唯一的键值，这对于正确的页面生命周期管理很重要
2. **性能考虑**：复杂的过渡动画可能会影响性能，特别是在低端设备上
3. **用户体验**：过渡动画应该增强用户体验，而不是分散注意力
4. **一致性**：在整个应用中保持过渡动画的一致性，有助于提升用户体验
