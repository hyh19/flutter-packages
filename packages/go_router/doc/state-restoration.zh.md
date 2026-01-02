# 状态恢复

## 什么是状态恢复？

状态恢复是指在应用被操作系统在后台终止后，持久化和恢复序列化状态的过程。

更多信息，请参阅 [Android 上的状态恢复](https://docs.flutter.dev/platform-integration/android/restore-state-android)
和 [iOS 上的状态恢复](https://docs.flutter.dev/platform-integration/ios/restore-state-ios)。

> [!NOTE]
> 状态恢复不是指通用的状态持久化。
> 如需在内存中同时保持多个导航分支，请参阅 [StatefulShellRoute](https://pub.dev/documentation/go_router/latest/go_router/StatefulShellRoute-class.html)。

## 支持

GoRouter 完全支持状态恢复。

要启用状态恢复，需要顶层配置以及根据使用的路由类型进行额外配置。

## 顶层配置

为 `GoRouter` 和 `MaterialApp.router` 添加 `restorationScopeId`：

```dart
final _router = GoRouter(
  restorationScopeId: 'router',
  routes: [
    ...
  ],
);
```

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      restorationScopeId: 'app',
      routerConfig: _router,
    );
  }
}
```

## 路由特定配置

### GoRoute

对于使用 `pageBuilder` 的 `GoRoute`，为页面提供 `restorationId`：

```dart
GoRoute(
  pageBuilder: (context, state) {
    return MaterialPage(
      restorationId: 'detailsPage',
      path: '/details',
      child: DetailsPage(),
    );
  },
)
```

对于不使用 `pageBuilder` 的 `GoRoute`，无需额外配置。

有关可运行的示例和测试，请参阅 [GoRoute 状态恢复示例](https://github.com/flutter/packages/tree/main/packages/go_router/example/lib/state_restoration/go_route_state_restoration.dart)。

### ShellRoute

为 `ShellRoute` 添加唯一的 `restorationScopeId`。
此外，添加 `pageBuilder` 并为页面提供 `restorationId`。

> [!IMPORTANT]
> 必须为 `ShellRoute` 提供返回带有 `restorationId` 的页面的 `pageBuilder`，状态恢复才能正常工作。

有关可运行的示例和测试，请参阅 [ShellRoute 状态恢复示例](https://github.com/flutter/packages/tree/main/packages/go_router/example/lib/state_restoration/shell_route_state_restoration.dart)。

```dart
ShellRoute(
  restorationScopeId: 'onboardingShell',
  pageBuilder: (context, state, child) {
    return MaterialPage(
      restorationId: 'onboardingPage',
      child: OnboardingScaffold(child: child),
    );
  },
  routes: [
    ...
  ],
)
```

### StatefulShellRoute

为 `StatefulShellRoute` 添加 `restorationScopeId` 和返回带有 `restorationId` 的页面的 `pageBuilder`。

此外，为每个 `StatefulShellBranch` 添加 `restorationScopeId`。

> [!IMPORTANT]
> 必须为 `StatefulShellRoute` 提供返回带有 `restorationId` 的页面的 `pageBuilder`，状态恢复才能正常工作。

有关可运行的示例和测试，请参阅 [StatefulShellRoute 状态恢复示例](https://github.com/flutter/packages/tree/main/packages/go_router/example/lib/state_restoration/stateful_shell_route_state_restoration.dart)。

```dart
StatefulShellRoute.indexedStack(
  restorationScopeId: 'appShell',
  pageBuilder: (context, state, navigationShell) {
    return MaterialPage(
      restorationId: 'appShellPage',
      child: AppShell(navigationShell: navigationShell),
    );
  },
  branches: [
    StatefulShellBranch(
      restorationScopeId: 'homeBranch',
      routes: [
        ...
      ],
    ),
    StatefulShellBranch(
      restorationScopeId: 'profileBranch',
      routes: [
        ...
      ],
    ),
  ],
)
```
