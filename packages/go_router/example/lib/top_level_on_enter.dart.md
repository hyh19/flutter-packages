# top_level_on_enter.dart 代码解析

## 概述

这是一个 Flutter 应用示例，演示了如何使用 `go_router` 包的顶级 `onEnter` 守卫功能。该示例展示了如何在路由导航之前进行拦截、处理深度链接、实现路由守卫、处理 OAuth 回调等高级路由场景。

## 核心功能

该示例主要演示了以下功能：

1. **顶级路由守卫**：使用 `onEnter` 在路由级别拦截导航
2. **深度链接处理**：处理带查询参数和片段（fragment）的深度链接
3. **推荐码处理**：处理推荐链接但不显示页面
4. **OAuth 回调处理**：处理 OAuth 认证回调
5. **路由保护**：未登录用户访问受保护路由时重定向到登录页
6. **异常处理**：统一的导航异常处理机制
7. **URL 片段支持**：支持使用 URL 片段（hash）进行页面内定位

## 代码结构

### 1. ReferralService - 推荐服务类

```dart 10:25:example/lib/top_level_on_enter.dart
/// Simulated service for handling referrals and deep links
class ReferralService {
  /// processReferralCode
  static Future<bool> processReferralCode(String code) async {
    // Simulate network delay
    await Future<dynamic>.delayed(const Duration(seconds: 1));
    return true;
  }

  /// trackDeepLink
  static Future<void> trackDeepLink(Uri uri) async {
    // Simulate analytics tracking
    await Future<dynamic>.delayed(const Duration(milliseconds: 300));
    debugPrint('Deep link tracked: $uri');
  }
}
```

这是一个模拟服务类，用于处理推荐码和深度链接追踪：

- `processReferralCode`：模拟处理推荐码的网络请求（延迟 1 秒）
- `trackDeepLink`：模拟追踪深度链接的分析服务（延迟 300 毫秒）

### 2. App - 主应用类

主应用类负责配置 `GoRouter` 和定义路由规则。

#### 2.1 路由配置入口

```dart 29:42:example/lib/top_level_on_enter.dart
/// The main application widget.
class App extends StatelessWidget {
  /// The main application widget.
  const App({super.key});

  @override
  Widget build(BuildContext context) {
    final key = GlobalKey<NavigatorState>();

    return MaterialApp.router(
      routerConfig: _router(key),
      title: 'Top-level onEnter',
    );
  }
```

使用 `MaterialApp.router` 配置路由，通过 `_router` 方法创建 `GoRouter` 实例。

#### 2.2 GoRouter 配置

```dart 44:200:example/lib/top_level_on_enter.dart
  /// Configures the router with navigation handling and deep link support.
  GoRouter _router(GlobalKey<NavigatorState> key) {
    return GoRouter(
      navigatorKey: key,
      initialLocation: '/home',
      debugLogDiagnostics: true,

      // If anything goes sideways during parsing/guards/redirects,
      // surface a friendly message and offer a one-tap "Go Home".
      onException:
          (BuildContext context, GoRouterState state, GoRouter router) {
            // Show a user-friendly error message
            if (context.mounted) {
              ScaffoldMessenger.of(context).showSnackBar(
                SnackBar(
                  content: Text('Navigation error: ${state.error}'),
                  backgroundColor: Colors.red,
                  duration: const Duration(seconds: 5),
                  action: SnackBarAction(
                    label: 'Go Home',
                    onPressed: () => router.go('/home'),
                  ),
                ),
              );
            }
            // Log the error for debugging
            debugPrint('Router exception: ${state.error}');

            // Navigate to error screen if needed
            if (state.uri.path == '/crash-test') {
              router.go('/error');
            }
          },

      /// Top-level guard runs BEFORE legacy top-level redirects and route-level redirects.
      /// Return:
      ///  - `Allow()` to proceed (optionally with `then:` side-effects)
      ///  - `Block.stop()` to cancel navigation immediately
      ///  - `Block.then(() => ...)` to cancel navigation and run follow-up work
      onEnter:
          (
            BuildContext context,
            GoRouterState current,
            GoRouterState next,
            GoRouter router,
          ) async {
            // Example: fire-and-forget analytics for deep links; never block the nav
            if (next.uri.hasQuery || next.uri.hasFragment) {
              // Don't await: keep the guard non-blocking for best UX.
              unawaited(ReferralService.trackDeepLink(next.uri));
            }

            switch (next.uri.path) {
              // Block deep-link routes that should never render a page
              // (we stay on the current page and show a lightweight UI instead).
              case '/referral':
                {
                  final String? code = next.uri.queryParameters['code'];
                  if (code != null) {
                    if (context.mounted) {
                      ScaffoldMessenger.of(context).showSnackBar(
                        const SnackBar(
                          content: Text('Processing referral code...'),
                          duration: Duration(seconds: 2),
                        ),
                      );
                    }
                    // Do the real work in the background; don't keep the user waiting.
                    await _processReferralCodeInBackground(context, code);
                  }
                  return const Block.stop(); // keep user where they are
                }

              // Simulate an OAuth callback: do background work + toast; never show a page at /auth
              case '/auth':
                {
                  final String? token = next.uri.queryParameters['token'];
                  if (token != null) {
                    _handleAuthToken(context, token);
                    return const Block.stop(); // cancel showing any /auth page
                  }
                  return const Allow();
                }

              // Demonstrate error reporting path
              case '/crash-test':
                throw Exception('Simulated error in onEnter callback!');

              case '/protected':
                {
                  // ignore: prefer_final_locals
                  var isLoggedIn = false; // pretend we're not authenticated
                  if (!isLoggedIn) {
                    // Chaining block: cancel the original nav, then redirect to /login.
                    // This preserves redirection history to detect loops.
                    final String from = Uri.encodeComponent(
                      next.uri.toString(),
                    );
                    return Block.then(() => router.go('/login?from=$from'));
                  }
                  // ignore: dead_code
                  return const Allow();
                }

              default:
                return const Allow();
            }
          },

      routes: <RouteBase>[
        // Simple "root → home"
        GoRoute(
          path: '/',
          redirect: (BuildContext _, GoRouterState __) => '/home',
        ),

        // Auth + simple pages
        GoRoute(path: '/login', builder: (_, __) => const LoginScreen()),
        GoRoute(path: '/home', builder: (_, __) => const HomeScreen()),
        GoRoute(path: '/settings', builder: (_, __) => const SettingsScreen()),

        // The following routes will never render (we always Block in onEnter),
        // but they exist so deep-links resolve safely.
        GoRoute(path: '/referral', builder: (_, __) => const SizedBox.shrink()),
        GoRoute(path: '/auth', builder: (_, __) => const SizedBox.shrink()),
        GoRoute(
          path: '/crash-test',
          builder: (_, __) => const SizedBox.shrink(),
        ),

        // Route-level redirect happens AFTER top-level onEnter allows.
        GoRoute(
          path: '/old',
          builder: (_, __) => const SizedBox.shrink(),
          redirect: (_, __) => '/home?from=old',
        ),

        // A page that shows fragments (#hash) via state.uri.fragment
        GoRoute(
          path: '/article/:id',
          name: 'article',
          builder: (_, GoRouterState state) {
            return Scaffold(
              appBar: AppBar(title: const Text('Article')),
              body: Center(
                child: Text(
                  'id=${state.pathParameters['id']}; fragment=${state.uri.fragment}',
                ),
              ),
            );
          },
        ),

        GoRoute(path: '/error', builder: (_, __) => const ErrorScreen()),
      ],
    );
  }
```

#### 2.3 onException - 异常处理

`onException` 回调用于处理路由导航过程中发生的异常：

- 显示用户友好的错误提示（红色 SnackBar）
- 提供"返回首页"的快捷操作
- 记录错误日志用于调试
- 特殊路由（如 `/crash-test`）会导航到错误页面

#### 2.4 onEnter - 顶级路由守卫

`onEnter` 是核心功能，它在路由导航之前执行，可以：

1. **允许导航**：返回 `Allow()` 继续导航
2. **阻止导航**：返回 `Block.stop()` 取消导航
3. **阻止并重定向**：返回 `Block.then(() => ...)` 取消导航并执行后续操作

**执行时机**：`onEnter` 在传统的顶级重定向和路由级重定向**之前**执行。

**深度链接追踪**：

```dart 90:94:example/lib/top_level_on_enter.dart
            // Example: fire-and-forget analytics for deep links; never block the nav
            if (next.uri.hasQuery || next.uri.hasFragment) {
              // Don't await: keep the guard non-blocking for best UX.
              unawaited(ReferralService.trackDeepLink(next.uri));
            }
```

对于所有带查询参数或片段的 URL，异步追踪深度链接（不阻塞导航）。

**推荐码处理**（`/referral`）：

```dart 99:115:example/lib/top_level_on_enter.dart
              case '/referral':
                {
                  final String? code = next.uri.queryParameters['code'];
                  if (code != null) {
                    if (context.mounted) {
                      ScaffoldMessenger.of(context).showSnackBar(
                        const SnackBar(
                          content: Text('Processing referral code...'),
                          duration: Duration(seconds: 2),
                        ),
                      );
                    }
                    // Do the real work in the background; don't keep the user waiting.
                    await _processReferralCodeInBackground(context, code);
                  }
                  return const Block.stop(); // keep user where they are
                }
```

- 从查询参数中提取推荐码
- 显示处理提示
- 后台处理推荐码（不阻塞用户）
- 返回 `Block.stop()` 阻止显示 `/referral` 页面

**OAuth 回调处理**（`/auth`）：

```dart 118:126:example/lib/top_level_on_enter.dart
              case '/auth':
                {
                  final String? token = next.uri.queryParameters['token'];
                  if (token != null) {
                    _handleAuthToken(context, token);
                    return const Block.stop(); // cancel showing any /auth page
                  }
                  return const Allow();
                }
```

- 提取 OAuth token
- 处理 token（显示提示，后台处理）
- 阻止显示 `/auth` 页面

**错误演示**（`/crash-test`）：

```dart 129:130:example/lib/top_level_on_enter.dart
              case '/crash-test':
                throw Exception('Simulated error in onEnter callback!');
```

故意抛出异常，用于测试异常处理机制。

**路由保护**（`/protected`）：

```dart 132:146:example/lib/top_level_on_enter.dart
              case '/protected':
                {
                  // ignore: prefer_final_locals
                  var isLoggedIn = false; // pretend we're not authenticated
                  if (!isLoggedIn) {
                    // Chaining block: cancel the original nav, then redirect to /login.
                    // This preserves redirection history to detect loops.
                    final String from = Uri.encodeComponent(
                      next.uri.toString(),
                    );
                    return Block.then(() => router.go('/login?from=$from'));
                  }
                  // ignore: dead_code
                  return const Allow();
                }
```

- 检查用户是否已登录
- 未登录时，取消原导航并重定向到登录页
- 将原始 URL 编码后作为 `from` 参数传递，用于登录后返回

#### 2.5 路由定义

路由定义包括：

1. **基础路由**：`/`、`/home`、`/login`、`/settings`
2. **深度链接路由**：`/referral`、`/auth`、`/crash-test`（这些路由永远不会显示页面，因为 `onEnter` 会阻止）
3. **重定向路由**：`/old` 重定向到 `/home?from=old`
4. **动态路由**：`/article/:id` 支持路径参数和 URL 片段
5. **错误页面**：`/error`

**注意**：深度链接路由虽然定义了 `builder`，但返回 `SizedBox.shrink()`，因为 `onEnter` 会阻止这些页面显示。

#### 2.6 辅助方法

**处理推荐码**：

```dart 202:222:example/lib/top_level_on_enter.dart
  /// Processes referral code in the background without blocking navigation
  Future<void> _processReferralCodeInBackground(
    BuildContext context,
    String code,
  ) async {
    final bool ok = await ReferralService.processReferralCode(code);
    if (!context.mounted) {
      return;
    }

    // Show result with a simple SnackBar
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text(
          ok
              ? 'Referral code $code applied successfully!'
              : 'Failed to apply referral code',
        ),
      ),
    );
  }
```

异步处理推荐码，完成后显示结果提示。

**处理 OAuth Token**：

```dart 224:248:example/lib/top_level_on_enter.dart
  /// Handles OAuth tokens with minimal UI interaction
  void _handleAuthToken(BuildContext context, String token) {
    if (!context.mounted) {
      return;
    }

    // Just show feedback, avoid complex UI
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text('Processing auth token: $token'),
        duration: const Duration(seconds: 2),
      ),
    );
    // background processing — keeps UI responsive and avoids re-entrancy
    Future<void>(() async {
      await Future<dynamic>.delayed(const Duration(seconds: 1));
      if (!context.mounted) {
        return;
      }

      ScaffoldMessenger.of(
        context,
      ).showSnackBar(SnackBar(content: Text('Auth token processed: $token')));
    });
  }
```

处理 OAuth token，显示处理提示，后台异步处理完成后再次提示。

### 3. HomeScreen - 首页

```dart 251:342:example/lib/top_level_on_enter.dart
/// Demonstrates various navigation scenarios and deep link handling.
class HomeScreen extends StatelessWidget {
  /// Demonstrates various navigation scenarios and deep link handling.
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    void goArticleWithFragment() {
      context.goNamed(
        'article',
        pathParameters: <String, String>{'id': '42'},
        // demonstrate fragment support (e.g., for in-page anchors)
        fragment: 'section-2',
      );
    }

    return Scaffold(
      appBar: AppBar(
        title: const Text('Top-level onEnter'),
        actions: <Widget>[
          IconButton(
            icon: const Icon(Icons.settings),
            onPressed: () => context.go('/settings'),
          ),
        ],
      ),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: <Widget>[
          // Navigation examples
          ElevatedButton.icon(
            onPressed: () => context.go('/login'),
            icon: const Icon(Icons.login),
            label: const Text('Go to Login'),
          ),
          const SizedBox(height: 16),

          Text(
            'Deep Link Tests',
            style: Theme.of(context).textTheme.titleMedium,
          ),
          const SizedBox(height: 8),
          const _DeepLinkButton(
            label: 'Process Referral',
            path: '/referral?code=TEST123',
            description: 'Processes code without navigation',
          ),
          const SizedBox(height: 8),
          const _DeepLinkButton(
            label: 'Auth Callback',
            path: '/auth?token=abc123',
            description: 'Simulates OAuth callback',
          ),

          const SizedBox(height: 24),
          Text(
            'Guards & Redirects',
            style: Theme.of(context).textTheme.titleMedium,
          ),
          const SizedBox(height: 8),
          const _DeepLinkButton(
            label: 'Protected Route (redirects to login)',
            path: '/protected',
            description: 'Top-level onEnter returns Block.then(() => go(...))',
          ),
          const SizedBox(height: 8),
          const _DeepLinkButton(
            label: 'Legacy Route-level Redirect',
            path: '/old',
            description: 'Route-level redirect to /home?from=old',
          ),

          const SizedBox(height: 24),
          Text(
            'Fragments (hash)',
            style: Theme.of(context).textTheme.titleMedium,
          ),
          const SizedBox(height: 8),
          OutlinedButton(
            onPressed: goArticleWithFragment,
            child: const Text('Open Article #section-2'),
          ),
          Text(
            "Uses goNamed(..., fragment: 'section-2') and reads state.uri.fragment",
            style: Theme.of(context).textTheme.bodySmall,
            textAlign: TextAlign.center,
          ),
        ],
      ),
    );
  }
}
```

首页提供了多个测试按钮，用于演示不同的路由场景：

- 基础导航：跳转到登录页
- 深度链接测试：推荐码处理、OAuth 回调
- 路由守卫和重定向：受保护路由、旧路由重定向
- URL 片段支持：使用 `goNamed` 的 `fragment` 参数

**URL 片段使用示例**：

```dart 258:265:example/lib/top_level_on_enter.dart
    void goArticleWithFragment() {
      context.goNamed(
        'article',
        pathParameters: <String, String>{'id': '42'},
        // demonstrate fragment support (e.g., for in-page anchors)
        fragment: 'section-2',
      );
    }
```

使用 `goNamed` 方法并指定 `fragment` 参数，可以在目标页面中通过 `state.uri.fragment` 读取。

### 4. _DeepLinkButton - 深度链接按钮组件

```dart 344:373:example/lib/top_level_on_enter.dart
/// A button that demonstrates a deep link scenario.
class _DeepLinkButton extends StatelessWidget {
  const _DeepLinkButton({
    required this.label,
    required this.path,
    required this.description,
  });

  final String label;
  final String path;
  final String description;

  @override
  Widget build(BuildContext context) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.stretch,
      children: <Widget>[
        OutlinedButton(onPressed: () => context.go(path), child: Text(label)),
        Padding(
          padding: const EdgeInsets.only(top: 4, bottom: 12),
          child: Text(
            description,
            style: Theme.of(context).textTheme.bodySmall,
            textAlign: TextAlign.center,
          ),
        ),
      ],
    );
  }
}
```

可复用的深度链接测试按钮组件，包含按钮和描述文字。

### 5. 其他屏幕组件

#### LoginScreen

```dart 375:396:example/lib/top_level_on_enter.dart
/// Login screen implementation
class LoginScreen extends StatelessWidget {
  /// Login screen implementation
  const LoginScreen({super.key});

  @override
  Widget build(BuildContext context) => Scaffold(
    appBar: AppBar(title: const Text('Login')),
    body: Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: <Widget>[
          ElevatedButton.icon(
            onPressed: () => context.go('/home'),
            icon: const Icon(Icons.home),
            label: const Text('Go to Home'),
          ),
        ],
      ),
    ),
  );
}
```

简单的登录页面，提供返回首页的按钮。

#### SettingsScreen

```dart 398:422:example/lib/top_level_on_enter.dart
/// Settings screen implementation
class SettingsScreen extends StatelessWidget {
  /// Settings screen implementation
  const SettingsScreen({super.key});

  @override
  Widget build(BuildContext context) => Scaffold(
    appBar: AppBar(title: const Text('Settings')),
    body: ListView(
      padding: const EdgeInsets.all(16),
      children: <Widget>[
        ListTile(
          title: const Text('Home'),
          leading: const Icon(Icons.home),
          onTap: () => context.go('/home'),
        ),
        ListTile(
          title: const Text('Login'),
          leading: const Icon(Icons.login),
          onTap: () => context.go('/login'),
        ),
      ],
    ),
  );
}
```

设置页面，提供导航到首页和登录页的选项。

#### ErrorScreen

```dart 424:452:example/lib/top_level_on_enter.dart
/// Error screen implementation
class ErrorScreen extends StatelessWidget {
  /// Error screen implementation
  const ErrorScreen({super.key});

  @override
  Widget build(BuildContext context) => Scaffold(
    appBar: AppBar(title: const Text('Error'), backgroundColor: Colors.red),
    body: Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: <Widget>[
          const Icon(Icons.error_outline, color: Colors.red, size: 60),
          const SizedBox(height: 16),
          const Text(
            'An error occurred during navigation',
            style: TextStyle(fontSize: 18),
          ),
          const SizedBox(height: 24),
          ElevatedButton.icon(
            onPressed: () => context.go('/home'),
            icon: const Icon(Icons.home),
            label: const Text('Return to Home'),
          ),
        ],
      ),
    ),
  );
}
```

错误页面，显示错误信息并提供返回首页的按钮。

## 关键概念

### onEnter 的执行顺序

`onEnter` 在以下操作**之前**执行：

1. 传统的顶级重定向
2. 路由级别的重定向（`GoRoute.redirect`）

这意味着 `onEnter` 可以拦截所有导航，包括那些原本会被重定向的导航。

### 路由守卫的返回值

`onEnter` 可以返回三种类型的值：

1. **`Allow()`**：允许导航继续
   - 可以带可选的 `then:` 参数执行副作用操作
   - 导航会正常进行

2. **`Block.stop()`**：立即取消导航
   - 用户停留在当前页面
   - 不会显示目标页面

3. **`Block.then(() => ...)`**：取消导航并执行后续操作
   - 通常用于重定向到其他页面
   - 保留重定向历史以检测循环

### 深度链接处理策略

对于不需要显示页面的深度链接（如推荐码、OAuth 回调），采用以下策略：

1. 在 `onEnter` 中处理业务逻辑
2. 返回 `Block.stop()` 阻止显示页面
3. 仍然定义路由（返回 `SizedBox.shrink()`），确保深度链接可以正确解析

这样可以：

- 确保深度链接不会导致 404 错误
- 避免显示不必要的页面
- 提供良好的用户体验

### 非阻塞操作

在 `onEnter` 中，对于不需要阻塞导航的操作（如分析追踪），使用 `unawaited()` 确保异步操作不会阻塞导航：

```dart 91:94:example/lib/top_level_on_enter.dart
            if (next.uri.hasQuery || next.uri.hasFragment) {
              // Don't await: keep the guard non-blocking for best UX.
              unawaited(ReferralService.trackDeepLink(next.uri));
            }
```

## 使用场景

### 1. 推荐码处理

当用户点击推荐链接（如 `app://referral?code=ABC123`）时：

1. `onEnter` 拦截导航到 `/referral`
2. 提取推荐码
3. 显示处理提示
4. 后台处理推荐码
5. 阻止显示 `/referral` 页面，用户停留在当前页面

### 2. OAuth 回调处理

当 OAuth 提供者回调到应用（如 `app://auth?token=xyz`）时：

1. `onEnter` 拦截导航到 `/auth`
2. 提取 token
3. 处理认证逻辑
4. 阻止显示 `/auth` 页面

### 3. 路由保护

当未登录用户尝试访问受保护的路由时：

1. `onEnter` 检查认证状态
2. 如果未登录，取消原导航
3. 重定向到登录页，并传递原始 URL
4. 登录后可以返回到原始页面

### 4. 深度链接追踪

对于所有带查询参数或片段的 URL：

1. 异步追踪深度链接（不阻塞）
2. 导航正常进行
3. 不影响用户体验

## 最佳实践

### 1. 检查 Context 是否已挂载

在执行异步操作后使用 `context` 之前，始终检查 `context.mounted`：

```dart 208:210:example/lib/top_level_on_enter.dart
    if (!context.mounted) {
      return;
    }
```

### 2. 非阻塞操作使用 unawaited

对于不需要等待的操作，使用 `unawaited()` 避免阻塞导航：

```dart 93:93:example/lib/top_level_on_enter.dart
              unawaited(ReferralService.trackDeepLink(next.uri));
```

### 3. 后台处理长时间操作

对于需要时间的操作（如网络请求），在后台处理，不要阻塞用户：

```dart 111:112:example/lib/top_level_on_enter.dart
                    // Do the real work in the background; don't keep the user waiting.
                    await _processReferralCodeInBackground(context, code);
```

### 4. 提供用户反馈

在处理异步操作时，提供适当的用户反馈（如 SnackBar）：

```dart 103:109:example/lib/top_level_on_enter.dart
                    if (context.mounted) {
                      ScaffoldMessenger.of(context).showSnackBar(
                        const SnackBar(
                          content: Text('Processing referral code...'),
                          duration: Duration(seconds: 2),
                        ),
                      );
                    }
```

### 5. 统一异常处理

使用 `onException` 统一处理导航异常，提供友好的错误提示：

```dart 53:76:example/lib/top_level_on_enter.dart
      onException:
          (BuildContext context, GoRouterState state, GoRouter router) {
            // Show a user-friendly error message
            if (context.mounted) {
              ScaffoldMessenger.of(context).showSnackBar(
                SnackBar(
                  content: Text('Navigation error: ${state.error}'),
                  backgroundColor: Colors.red,
                  duration: const Duration(seconds: 5),
                  action: SnackBarAction(
                    label: 'Go Home',
                    onPressed: () => router.go('/home'),
                  ),
                ),
              );
            }
            // Log the error for debugging
            debugPrint('Router exception: ${state.error}');

            // Navigate to error screen if needed
            if (state.uri.path == '/crash-test') {
              router.go('/error');
            }
          },
```

## 总结

这个示例展示了 `go_router` 的 `onEnter` 功能的强大之处：

1. **灵活性**：可以在导航前拦截、处理、重定向
2. **用户体验**：支持非阻塞操作，保持界面响应
3. **深度链接**：优雅处理各种深度链接场景
4. **安全性**：实现路由保护，控制访问权限
5. **可维护性**：统一的异常处理和错误提示

通过合理使用 `onEnter`，可以构建出功能强大、用户体验良好的路由系统。
