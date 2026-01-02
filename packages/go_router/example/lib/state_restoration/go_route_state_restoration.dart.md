# GoRoute 状态恢复配置示例解析

## 概述

这个示例展示了如何在 `go_router` 中为 `GoRoute` 配置状态恢复（State Restoration）功能。状态恢复允许应用在系统终止后（如内存不足时）重新启动时，自动恢复用户界面的状态，包括路由位置和表单输入内容。

## 核心概念

状态恢复在 Flutter 中通过 `restorationScopeId` 和 `restorationId` 来实现：

- **restorationScopeId**：定义状态恢复的作用域，用于组织和管理恢复数据
- **restorationId**：为特定的 Widget 或 Page 提供唯一标识，用于恢复其状态

## 代码结构分析

### 应用入口

```dart 8:8:example/lib/state_restoration/go_route_state_restoration.dart
void main() => runApp(const App());
```

应用入口非常简单，直接启动 `App` 组件。

### App 组件配置

```dart 12:56:example/lib/state_restoration/go_route_state_restoration.dart
class App extends StatefulWidget {
  /// Creates an [App].
  const App({super.key});

  @override
  State<App> createState() => _AppState();
}

class _AppState extends State<App> {
  final GoRouter _router = GoRouter(
    restorationScopeId: 'router',
    routes: <GoRoute>[
      GoRoute(
        path: '/',
        // restorationId is set for the route automatically
        // since builder is used.
        builder: (BuildContext context, GoRouterState state) {
          return const HomePage();
        },
        routes: <GoRoute>[
          GoRoute(
            path: 'login',
            // restorationId must be supplied to the MaterialPage
            // since pageBuilder is used.
            pageBuilder: (BuildContext context, GoRouterState state) {
              return const MaterialPage<void>(
                restorationId: 'loginPage',
                fullscreenDialog: true,
                child: LoginPage(),
              );
            },
          ),
        ],
      ),
    ],
  );

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      restorationScopeId: 'mainApp',
      routerConfig: _router,
    );
  }
}
```

#### 关键配置点

1. **GoRouter 的 restorationScopeId**：
   - 设置为 `'router'`，为整个路由系统创建恢复作用域
   - 这确保了路由状态可以被正确保存和恢复

2. **根路由（`/`）的状态恢复**：
   - 使用 `builder` 方式创建页面时，`restorationId` 会自动设置
   - 这是 `go_router` 的便利特性，无需手动指定

3. **子路由（`/login`）的状态恢复**：
   - 使用 `pageBuilder` 方式时，必须手动在 `MaterialPage` 中指定 `restorationId`
   - 这里设置为 `'loginPage'`，确保登录页面状态可以被恢复
   - `fullscreenDialog: true` 表示这是一个全屏对话框样式的页面

4. **MaterialApp.router 的 restorationScopeId**：
   - 设置为 `'mainApp'`，这是应用最顶层的恢复作用域
   - 与 GoRouter 的 `restorationScopeId` 形成层级关系：`mainApp` > `router`

### HomePage 组件

```dart 58:80:example/lib/state_restoration/go_route_state_restoration.dart
/// The root page of the app.
class HomePage extends StatelessWidget {
  /// Creates a [HomePage].
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Home')),
      body: Column(
        children: <Widget>[
          const TextField(restorationId: 'homeTextField'),
          FilledButton(
            onPressed: () {
              context.go('/login');
            },
            child: const Text('Go to Login'),
          ),
        ],
      ),
    );
  }
}
```

**状态恢复要点**：

- `TextField` 设置了 `restorationId: 'homeTextField'`
- 当应用恢复时，用户在文本框中输入的内容会自动恢复
- 使用 `context.go('/login')` 进行导航

### LoginPage 组件

```dart 82:94:example/lib/state_restoration/go_route_state_restoration.dart
/// A [LoginPage] with a restorable [TextField].
class LoginPage extends StatelessWidget {
  /// Creates a [LoginPage].
  const LoginPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Login')),
      body: const TextField(restorationId: 'loginTextField'),
    );
  }
}
```

**状态恢复要点**：

- 登录页面的 `TextField` 也设置了 `restorationId: 'loginTextField'`
- 即使用户在登录页面输入内容后应用被终止，恢复时输入内容也会保留

## 状态恢复层级结构

这个示例展示了完整的状态恢复层级：

```text
mainApp (MaterialApp.router)
  └── router (GoRouter)
      ├── / (自动设置 restorationId)
      │   └── homeTextField (TextField)
      └── /login (restorationId: 'loginPage')
          └── loginTextField (TextField)
```

## 使用场景

这个配置适用于以下场景：

1. **简单路由结构**：使用 `GoRoute` 的普通路由场景
2. **表单数据保护**：需要保护用户在表单中输入的数据
3. **导航状态恢复**：需要恢复用户离开时的路由位置

## 注意事项

1. **builder vs pageBuilder**：
   - 使用 `builder` 时，`restorationId` 会自动设置，更简单
   - 使用 `pageBuilder` 时，必须手动在 `MaterialPage` 中指定 `restorationId`

2. **restorationId 的唯一性**：
   - 在同一作用域内，`restorationId` 必须唯一
   - 不同作用域可以有相同的 `restorationId`

3. **作用域层级**：
   - 作用域形成树形结构，子作用域可以访问父作用域的数据
   - 合理设置作用域层级有助于组织和管理恢复数据

## 测试状态恢复

要测试状态恢复功能：

1. 在模拟器或设备上运行应用
2. 导航到登录页面并输入一些文本
3. 在设备设置中启用"不保留活动"选项（Android）或使用系统终止应用
4. 重新启动应用，应该能看到：
   - 自动导航到登录页面
   - 文本框中之前输入的内容已恢复
