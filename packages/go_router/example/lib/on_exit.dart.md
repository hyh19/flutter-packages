# on_exit.dart 代码解析

## 概述

这是一个 Flutter 示例应用，演示了如何使用 `go_router` 包中 `GoRoute` 的 `onExit` 回调功能。`onExit` 允许在用户离开某个路由页面时执行自定义逻辑，比如显示确认对话框，防止用户意外离开。

## 应用结构

应用包含三个主要屏幕：

1. **HomeScreen**：首页，包含导航到详情页的按钮
2. **DetailsScreen**：详情页，配置了 `onExit` 回调，离开时会弹出确认对话框
3. **SettingsScreen**：设置页，用于测试从详情页导航到其他页面

## 路由配置

应用使用 `GoRouter` 进行路由管理，路由配置如下：

```dart 12:56:example/lib/on_exit.dart
final GoRouter _router = GoRouter(
  routes: <RouteBase>[
    GoRoute(
      path: '/',
      builder: (BuildContext context, GoRouterState state) {
        return const HomeScreen();
      },
      routes: <RouteBase>[
        GoRoute(
          path: 'details',
          builder: (BuildContext context, GoRouterState state) {
            return const DetailsScreen();
          },
          onExit: (BuildContext context, GoRouterState state) async {
            final bool? confirmed = await showDialog<bool>(
              context: context,
              builder: (_) {
                return AlertDialog(
                  content: const Text('Are you sure to leave this page?'),
                  actions: <Widget>[
                    TextButton(
                      onPressed: () => Navigator.of(context).pop(false),
                      child: const Text('Cancel'),
                    ),
                    TextButton(
                      onPressed: () => Navigator.of(context).pop(true),
                      child: const Text('Confirm'),
                    ),
                  ],
                );
              },
            );
            return confirmed ?? false;
          },
        ),
        GoRoute(
          path: 'settings',
          builder: (BuildContext context, GoRouterState state) {
            return const SettingsScreen();
          },
        ),
      ],
    ),
  ],
);
```

### 路由层级

- **根路由** (`/`)：对应 `HomeScreen`
  - **详情路由** (`details`)：对应 `DetailsScreen`，配置了 `onExit` 回调
  - **设置路由** (`settings`)：对应 `SettingsScreen`

### onExit 回调详解

`onExit` 是 `GoRoute` 的一个可选回调函数，在用户尝试离开该路由时会被调用。回调函数的签名如下：

```dart
Future<bool?> Function(BuildContext context, GoRouterState state)
```

**关键特性**：

1. **返回值决定导航行为**：
   - 返回 `true`：允许离开当前页面，导航继续进行
   - 返回 `false`：阻止导航，用户停留在当前页面
   - 返回 `null`：等同于返回 `false`，阻止导航

2. **异步支持**：回调函数是异步的，可以执行异步操作（如显示对话框、保存数据等）

3. **触发时机**：当用户尝试通过以下方式离开页面时会触发：
   - 调用 `context.go()` 导航到其他路由
   - 调用 `context.pop()` 返回上一页
   - 使用系统返回按钮（Android）或返回手势（iOS）

在本示例中，`onExit` 回调的实现如下：

```dart 25:45:example/lib/on_exit.dart
onExit: (BuildContext context, GoRouterState state) async {
  final bool? confirmed = await showDialog<bool>(
    context: context,
    builder: (_) {
      return AlertDialog(
        content: const Text('Are you sure to leave this page?'),
        actions: <Widget>[
          TextButton(
            onPressed: () => Navigator.of(context).pop(false),
            child: const Text('Cancel'),
          ),
          TextButton(
            onPressed: () => Navigator.of(context).pop(true),
            child: const Text('Confirm'),
          ),
        ],
      );
    },
  );
  return confirmed ?? false;
},
```

**实现逻辑**：

1. 显示一个确认对话框，询问用户是否确定要离开页面
2. 用户点击 "Cancel" 时，对话框返回 `false`，导航被阻止
3. 用户点击 "Confirm" 时，对话框返回 `true`，导航继续进行
4. 如果用户直接关闭对话框（返回 `null`），使用 `??` 运算符将其转换为 `false`，同样阻止导航

## 应用入口

应用的主入口非常简单：

```dart 8:9:example/lib/on_exit.dart
/// This sample app demonstrates how to use GoRoute.onExit.
void main() => runApp(const MyApp());
```

```dart 58:67:example/lib/on_exit.dart
/// The main app.
class MyApp extends StatelessWidget {
  /// Constructs a [MyApp]
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(routerConfig: _router);
  }
}
```

`MyApp` 使用 `MaterialApp.router` 并传入配置好的 `_router`，这是使用 `go_router` 的标准方式。

## 屏幕组件

### HomeScreen

首页提供了一个按钮，用于导航到详情页：

```dart 69:91:example/lib/on_exit.dart
/// The home screen
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
          ],
        ),
      ),
    );
  }
}
```

点击按钮会调用 `context.go('/details')` 导航到详情页。

### DetailsScreen

详情页提供了两个按钮，用于测试不同的导航场景：

```dart 93:122:example/lib/on_exit.dart
/// The details screen
class DetailsScreen extends StatelessWidget {
  /// Constructs a [DetailsScreen]
  const DetailsScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Details Screen')),
      body: Center(
        child: Column(
          children: <Widget>[
            TextButton(
              onPressed: () {
                context.pop();
              },
              child: const Text('go back'),
            ),
            TextButton(
              onPressed: () {
                context.go('/settings');
              },
              child: const Text('go to settings'),
            ),
          ],
        ),
      ),
    );
  }
}
```

**两个导航按钮的作用**：

1. **"go back" 按钮**：调用 `context.pop()` 返回上一页（首页）
2. **"go to settings" 按钮**：调用 `context.go('/settings')` 导航到设置页

无论用户点击哪个按钮，或者使用系统返回按钮，都会触发 `onExit` 回调，显示确认对话框。

### SettingsScreen

设置页是一个简单的展示页面：

```dart 124:136:example/lib/on_exit.dart
/// The settings screen
class SettingsScreen extends StatelessWidget {
  /// Constructs a [SettingsScreen]
  const SettingsScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Settings Screen')),
      body: const Center(child: Text('Settings')),
    );
  }
}
```

## 使用场景

`onExit` 回调适用于以下场景：

1. **表单数据保护**：用户在表单中输入了数据但未保存，离开时提示保存
2. **未完成操作提醒**：提醒用户有未完成的操作
3. **权限确认**：某些敏感页面需要确认权限才能离开
4. **数据同步**：离开前同步数据到服务器

## 注意事项

1. **异步操作**：`onExit` 是异步的，可以执行耗时操作，但要注意用户体验，避免长时间阻塞导航

2. **返回值处理**：确保正确处理返回值，`null` 值会被视为 `false`，会阻止导航

3. **对话框上下文**：在 `onExit` 中显示对话框时，确保使用正确的 `context`，通常使用回调函数提供的 `context` 参数

4. **性能考虑**：如果 `onExit` 中包含耗时操作，考虑添加加载指示器，提升用户体验

5. **系统返回按钮**：在 Android 上，系统返回按钮也会触发 `onExit`，这是 `go_router` 提供的便利功能

## 总结

这个示例展示了如何使用 `go_router` 的 `onExit` 功能来实现页面离开确认。通过配置 `onExit` 回调，开发者可以在用户离开页面时执行自定义逻辑，比如显示确认对话框，从而提升用户体验并防止意外操作。
