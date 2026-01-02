# go_relative.dart 代码解释

## 概述

这个示例文件展示了如何在 Flutter 中使用 `go_router` 包进行相对路径导航。它演示了 `GoRouter.go('./$path')` 的用法，这是一种基于当前路由位置进行相对导航的方式。

## 文件结构

文件包含以下主要部分：

- 路由配置：定义了嵌套的路由结构
- 三个屏幕组件：`HomeScreen`、`DetailsScreen`、`SettingsScreen`
- 相对路径导航：使用 `context.go('./$path')` 进行导航

## 详细说明

### 路由配置

```dart 23:48:example/lib/go_relative.dart
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
          routes: <RouteBase>[
            GoRoute(
              path: 'settings',
              builder: (BuildContext context, GoRouterState state) {
                return const SettingsScreen();
              },
            ),
          ],
        ),
      ],
    ),
  ],
);
```

这个路由配置创建了一个三层嵌套的路由结构：

1. **根路由** (`/`): 对应 `HomeScreen`
2. **详情路由** (`/details`): 作为根路由的子路由，对应 `DetailsScreen`
3. **设置路由** (`/details/settings`): 作为详情路由的子路由，对应 `SettingsScreen`

**要点**：

- 子路由的 `path` 不需要以 `/` 开头，GoRouter 会自动拼接父路由路径
- `details` 会被解析为 `/details`
- `settings` 会被解析为 `/details/settings`

### 主应用入口

```dart 12:20:example/lib/go_relative.dart
class MyApp extends StatelessWidget {
  /// Constructs a [MyApp]
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(routerConfig: _router);
  }
}
```

这是一个标准的 Flutter 应用入口，使用 `MaterialApp.router` 并配置了上面定义的路由。

### HomeScreen - 首页

```dart 51:72:example/lib/go_relative.dart
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
              onPressed: () => context.go('./details'),
              child: const Text('Go to the Details screen'),
            ),
          ],
        ),
      ),
    );
  }
}
```

**关键导航代码**：

```dart 64:64:example/lib/go_relative.dart
onPressed: () => context.go('./details'),
```

这里使用了相对路径 `./details`：

- `./` 表示当前路由位置（即 `/`）
- `details` 是相对于当前位置的子路由
- 最终导航到 `/details`

### DetailsScreen - 详情页

```dart 75:103:example/lib/go_relative.dart
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
              child: const Text('Go back'),
            ),
            TextButton(
              onPressed: () {
                context.go('./settings');
              },
              child: const Text('Go to the Settings screen'),
            ),
          ],
        ),
      ),
    );
  }
}
```

这个屏幕包含两个导航操作：

1. **返回上一页**：

```dart 87:90:example/lib/go_relative.dart
onPressed: () {
  context.pop();
},
```

使用 `context.pop()` 返回导航栈中的上一个路由。

1. **导航到设置页面**：

```dart 93:96:example/lib/go_relative.dart
onPressed: () {
  context.go('./settings');
},
```

使用相对路径 `./settings` 导航：

- 当前路由是 `/details`
- `./settings` 表示相对于 `/details` 的子路由
- 最终导航到 `/details/settings`

### SettingsScreen - 设置页

```dart 106:126:example/lib/go_relative.dart
class SettingsScreen extends StatelessWidget {
  /// Constructs a [SettingsScreen]
  const SettingsScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Settings Screen')),
      body: Column(
        children: <Widget>[
          TextButton(
            onPressed: () {
              context.pop();
            },
            child: const Text('Go back'),
          ),
        ],
      ),
    );
  }
}
```

设置页面只有一个返回按钮，使用 `context.pop()` 返回到 `/details` 页面。

## 相对路径导航的优势

### 1. 简化路径管理

使用相对路径可以避免硬编码完整路径，让代码更加灵活：

```dart
// 绝对路径（需要知道完整路径）
context.go('/details/settings');

// 相对路径（基于当前位置）
context.go('./settings');
```

### 2. 路由重构更安全

如果路由结构发生变化，相对路径更容易维护，因为你不需要修改所有使用完整路径的地方。

### 3. 代码可读性更好

相对路径能够清晰地表达"相对于当前路由"的导航意图。

## 相对路径语法说明

- `./path`: 相对于当前路由的子路由
- `../path`: 相对于父路由的路径
- `./`: 当前路由本身

## 完整导航流程示例

1. 应用启动 → `/` (HomeScreen)
2. 点击 "Go to the Details screen" → `/details` (DetailsScreen)
3. 点击 "Go to the Settings screen" → `/details/settings` (SettingsScreen)
4. 点击 "Go back" → `/details` (DetailsScreen)
5. 点击 "Go back" → `/` (HomeScreen)

## 总结

这个示例展示了 `go_router` 中相对路径导航的核心用法，通过 `context.go('./$path')` 实现基于当前路由位置的相对导航。这种方式让路由管理更加灵活和可维护，特别适合嵌套路由的场景。
