# ShellRoute 状态恢复配置示例解析

## 概述

这个示例展示了如何在 `go_router` 中为 `ShellRoute` 配置状态恢复功能。`ShellRoute` 是一种特殊的路由类型，它允许在多个子路由之间共享一个公共的 UI 容器（Shell），常用于实现引导流程、设置向导等场景。

## ShellRoute 的特点

`ShellRoute` 与普通 `GoRoute` 的主要区别在于：

- **共享容器**：所有子路由共享同一个 Shell 容器（如 `OnboardingScaffold`）
- **状态保持**：Shell 容器的状态在子路由切换时保持不变
- **独立作用域**：可以为 Shell 设置独立的 `restorationScopeId`

## 代码结构分析

### 应用入口

```dart 8:8:example/lib/state_restoration/shell_route_state_restoration.dart
void main() => runApp(const App());
```

### App 组件配置

```dart 12:63:example/lib/state_restoration/shell_route_state_restoration.dart
class App extends StatefulWidget {
  /// Creates an [App].
  const App({super.key});

  @override
  State<App> createState() => _AppState();
}

class _AppState extends State<App> {
  final GoRouter _router = GoRouter(
    restorationScopeId: 'router',
    routes: <RouteBase>[
      GoRoute(
        path: '/',
        builder: (BuildContext context, GoRouterState state) {
          return const HomePage();
        },
        routes: <RouteBase>[
          ShellRoute(
            restorationScopeId: 'onboardingShell',
            pageBuilder:
                (BuildContext context, GoRouterState state, Widget child) {
                  return MaterialPage<void>(
                    restorationId: 'onboardingPage',
                    child: OnboardingScaffold(child: child),
                  );
                },
            routes: <GoRoute>[
              GoRoute(
                path: 'welcome',
                builder: (BuildContext context, GoRouterState state) {
                  return const WelcomeBody();
                },
              ),
              GoRoute(
                path: 'setup',
                builder: (BuildContext context, GoRouterState state) {
                  return const SetupBody();
                },
              ),
            ],
          ),
        ],
      ),
    ],
  );

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(restorationScopeId: 'app', routerConfig: _router);
  }
}
```

#### 关键配置点

1. **路由结构**：
   - 根路由 `/` 指向 `HomePage`
   - 在根路由下定义了一个 `ShellRoute`，包含两个子路由：`welcome` 和 `setup`

2. **ShellRoute 的状态恢复配置**：

   ```dart 30:38:example/lib/state_restoration/shell_route_state_restoration.dart
   ShellRoute(
     restorationScopeId: 'onboardingShell',
     pageBuilder:
         (BuildContext context, GoRouterState state, Widget child) {
           return MaterialPage<void>(
             restorationId: 'onboardingPage',
             child: OnboardingScaffold(child: child),
           );
         },
   ```

   - `restorationScopeId: 'onboardingShell'`：为 Shell 创建独立的状态恢复作用域
   - `restorationId: 'onboardingPage'`：为 Shell 页面指定恢复标识
   - `pageBuilder` 接收 `child` 参数，这是当前激活的子路由内容

3. **子路由配置**：
   - `welcome` 和 `setup` 是引导流程的两个步骤
   - 它们都共享 `OnboardingScaffold` 作为容器
   - 子路由使用 `builder`，所以 `restorationId` 会自动设置

### HomePage 组件

```dart 65:87:example/lib/state_restoration/shell_route_state_restoration.dart
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
              context.go('/welcome');
            },
            child: const Text('Go to Welcome'),
          ),
        ],
      ),
    );
  }
}
```

- 首页包含一个可恢复的文本输入框
- 点击按钮导航到 `/welcome`，这会进入 ShellRoute 的第一个子路由

### OnboardingScaffold 组件

```dart 89:107:example/lib/state_restoration/shell_route_state_restoration.dart
/// A [Scaffold] for the onboarding flow.
class OnboardingScaffold extends StatelessWidget {
  /// Creates an [OnboardingScaffold].
  const OnboardingScaffold({required this.child, super.key});

  /// The widget displayed in the body of the [Scaffold].
  final Widget child;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Onboarding'),
        automaticallyImplyLeading: false,
      ),
      body: child,
    );
  }
}
```

**设计要点**：

- `OnboardingScaffold` 是 Shell 容器，为所有子路由提供统一的 UI 框架
- `child` 参数是当前激活的子路由内容（`WelcomeBody` 或 `SetupBody`）
- `automaticallyImplyLeading: false` 禁用了返回按钮，因为这是引导流程，不应该有返回操作
- 这个 Scaffold 的状态会在子路由切换时保持，但不会自动恢复（如果需要恢复，需要将其改为 `StatefulWidget` 并添加 `restorationId`）

### WelcomeBody 组件

```dart 109:128:example/lib/state_restoration/shell_route_state_restoration.dart
/// The body for the Welcome step of the onboarding flow.
class WelcomeBody extends StatelessWidget {
  /// Creates a [WelcomeBody].
  const WelcomeBody({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: <Widget>[
        const Text('Welcome'),
        FilledButton(
          onPressed: () {
            context.go('/setup');
          },
          child: const Text('Go to Setup'),
        ),
      ],
    );
  }
}
```

- 欢迎页面的内容，点击按钮导航到设置步骤

### SetupBody 组件

```dart 130:150:example/lib/state_restoration/shell_route_state_restoration.dart
/// The body for the Setup step of the onboarding flow.
class SetupBody extends StatelessWidget {
  /// Creates a [SetupBody].
  const SetupBody({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: <Widget>[
        const Text('Setup'),
        const TextField(restorationId: 'setupTextField'),
        FilledButton(
          onPressed: () {
            context.go('/');
          },
          child: const Text('Go to Home'),
        ),
      ],
    );
  }
}
```

**状态恢复要点**：

- `TextField` 设置了 `restorationId: 'setupTextField'`
- 用户在设置步骤输入的内容会在应用恢复时自动恢复
- 这个文本框的状态恢复作用域是 `onboardingShell`

## 状态恢复层级结构

```text
app (MaterialApp.router)
  └── router (GoRouter)
      └── / (自动设置 restorationId)
          ├── homeTextField (TextField)
          └── onboardingShell (ShellRoute)
              └── onboardingPage (MaterialPage)
                  ├── welcome (自动设置 restorationId)
                  └── setup (自动设置 restorationId)
                      └── setupTextField (TextField)
```

## ShellRoute 的优势

1. **UI 一致性**：所有子路由共享相同的 Shell 容器，保持 UI 一致性
2. **状态隔离**：Shell 有独立的作用域，便于管理状态恢复
3. **导航流畅**：子路由切换时，Shell 容器保持不变，提供流畅的导航体验
4. **代码复用**：公共的 UI 元素（如 AppBar）只需定义一次

## 使用场景

这个配置适用于以下场景：

1. **引导流程**：新用户引导、功能介绍等
2. **设置向导**：多步骤的设置流程
3. **表单流程**：需要分步骤填写的复杂表单
4. **认证流程**：登录、注册等多步骤流程

## 注意事项

1. **Shell 状态恢复**：
   - 当前示例中，`OnboardingScaffold` 是 `StatelessWidget`，不会自动恢复状态
   - 如果 Shell 容器需要恢复状态（如滚动位置、展开状态等），应改为 `StatefulWidget` 并添加 `restorationId`

2. **子路由切换**：
   - 子路由切换时，Shell 容器不会重建
   - 只有 `child` 参数会变化，指向新的子路由内容

3. **导航行为**：
   - 在 Shell 内部导航使用 `context.go()` 或 `context.push()`
   - 退出 Shell 回到首页也使用 `context.go('/')`

4. **作用域管理**：
   - Shell 的 `restorationScopeId` 创建了独立的作用域
   - 子路由中的可恢复组件会自动使用这个作用域

## 与 GoRoute 的对比

| 特性 | GoRoute | ShellRoute |
| ------ | --------- | ------------ |
| 容器共享 | 否 | 是 |
| 状态保持 | 否 | 是（Shell 容器） |
| 适用场景 | 独立页面 | 相关页面组 |
| 恢复作用域 | 路由级别 | Shell 级别 |
