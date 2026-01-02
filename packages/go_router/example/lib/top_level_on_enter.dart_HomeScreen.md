# HomeScreen 类说明

## 概述

`HomeScreen` 是一个演示页面，用于展示 `go_router` 包中各种导航场景和深度链接处理功能。它提供了多个交互式按钮，用于测试不同的路由场景，包括基础导航、深度链接处理、路由守卫、重定向以及 URL 片段（hash）等功能。

## 类定义

```dart 251:254:example/lib/top_level_on_enter.dart
/// Demonstrates various navigation scenarios and deep link handling.
class HomeScreen extends StatelessWidget {
  /// Demonstrates various navigation scenarios and deep link handling.
  const HomeScreen({super.key});
```

`HomeScreen` 继承自 `StatelessWidget`，是一个无状态的 Flutter 组件。它使用 `const` 构造函数，表明这是一个不可变的组件。

## 核心功能

### 1. 基础导航

```dart 281:285:example/lib/top_level_on_enter.dart
          ElevatedButton.icon(
            onPressed: () => context.go('/login'),
            icon: const Icon(Icons.login),
            label: const Text('Go to Login'),
          ),
```

这个按钮演示了最基本的导航功能，使用 `context.go()` 方法导航到登录页面。`go()` 方法会替换当前路由栈，直接跳转到目标路由。

### 2. 深度链接测试

#### 处理推荐码（Referral Code）

```dart 293:297:example/lib/top_level_on_enter.dart
          const _DeepLinkButton(
            label: 'Process Referral',
            path: '/referral?code=TEST123',
            description: 'Processes code without navigation',
          ),
```

这个按钮演示了如何处理深度链接中的推荐码。当用户点击时，会导航到 `/referral?code=TEST123`。根据路由配置中的 `onEnter` 守卫，这个路由会被拦截（返回 `Block.stop()`），不会实际显示页面，而是在后台处理推荐码并显示提示信息。

#### OAuth 回调处理

```dart 299:303:example/lib/top_level_on_enter.dart
          const _DeepLinkButton(
            label: 'Auth Callback',
            path: '/auth?token=abc123',
            description: 'Simulates OAuth callback',
          ),
```

这个按钮模拟了 OAuth 认证回调场景。当导航到 `/auth?token=abc123` 时，`onEnter` 守卫会提取 token 参数并在后台处理，同时阻止显示 `/auth` 页面（返回 `Block.stop()`）。

### 3. 路由守卫和重定向

#### 受保护路由

```dart 311:315:example/lib/top_level_on_enter.dart
          const _DeepLinkButton(
            label: 'Protected Route (redirects to login)',
            path: '/protected',
            description: 'Top-level onEnter returns Block.then(() => go(...))',
          ),
```

这个按钮演示了路由守卫功能。当用户尝试访问 `/protected` 时，`onEnter` 守卫会检查用户是否已登录。如果未登录，会使用 `Block.then()` 取消原始导航并重定向到登录页面，同时保留重定向历史以检测循环重定向。

#### 路由级重定向

```dart 317:321:example/lib/top_level_on_enter.dart
          const _DeepLinkButton(
            label: 'Legacy Route-level Redirect',
            path: '/old',
            description: 'Route-level redirect to /home?from=old',
          ),
```

这个按钮演示了路由级别的重定向功能。当访问 `/old` 时，路由配置中的 `redirect` 回调会在 `onEnter` 守卫允许后执行，将用户重定向到 `/home?from=old`。

### 4. URL 片段（Fragment/Hash）支持

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

这个本地函数演示了如何使用命名路由和 URL 片段。它使用 `context.goNamed()` 方法导航到名为 `'article'` 的路由，传递路径参数 `id: '42'`，并设置片段为 `'section-2'`。最终生成的 URL 类似于 `/article/42#section-2`。

```dart 329:337:example/lib/top_level_on_enter.dart
          OutlinedButton(
            onPressed: goArticleWithFragment,
            child: const Text('Open Article #section-2'),
          ),
          Text(
            "Uses goNamed(..., fragment: 'section-2') and reads state.uri.fragment",
            style: Theme.of(context).textTheme.bodySmall,
            textAlign: TextAlign.center,
          ),
```

按钮触发上述函数，下方的说明文字解释了该功能的工作原理。

## UI 结构

### AppBar

```dart 268:276:example/lib/top_level_on_enter.dart
      appBar: AppBar(
        title: const Text('Top-level onEnter'),
        actions: <Widget>[
          IconButton(
            icon: const Icon(Icons.settings),
            onPressed: () => context.go('/settings'),
          ),
        ],
      ),
```

AppBar 包含一个设置按钮，点击后会导航到设置页面。

### Body 布局

```dart 277:339:example/lib/top_level_on_enter.dart
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
```

Body 使用 `ListView` 布局，包含以下内容：

1. **基础导航按钮**：跳转到登录页面
2. **深度链接测试区域**：包含两个 `_DeepLinkButton`，用于测试推荐码和 OAuth 回调
3. **路由守卫和重定向区域**：包含两个 `_DeepLinkButton`，用于测试受保护路由和路由级重定向
4. **URL 片段区域**：包含一个按钮和说明文字，演示片段功能

每个区域之间使用 `SizedBox` 添加间距，并使用 `Text` 组件作为区域标题。

## 关键概念

### 1. 深度链接处理

深度链接允许应用响应外部链接（如来自邮件、短信或网页的链接）。`HomeScreen` 演示了如何处理这些链接：

- **推荐码处理**：从 URL 查询参数中提取推荐码，在后台处理而不显示页面
- **OAuth 回调**：从 URL 中提取认证 token，完成认证流程

### 2. 路由守卫

路由守卫（`onEnter`）在导航发生前执行，可以：

- **允许导航**：返回 `Allow()` 继续导航
- **阻止导航**：返回 `Block.stop()` 取消导航
- **阻止并重定向**：返回 `Block.then(() => router.go(...))` 取消导航并执行重定向

### 3. 命名路由

使用 `goNamed()` 方法可以通过路由名称进行导航，而不是硬编码路径。这提供了更好的类型安全性和可维护性。

### 4. URL 片段

URL 片段（hash）用于页面内锚点定位。`go_router` 支持通过 `fragment` 参数设置片段，可以通过 `state.uri.fragment` 读取。

## 使用场景

`HomeScreen` 适用于以下场景：

1. **学习和演示**：作为 `go_router` 功能的演示页面
2. **测试深度链接**：测试应用处理外部链接的能力
3. **验证路由守卫**：测试路由守卫和重定向逻辑
4. **开发调试**：在开发过程中快速测试各种导航场景

## 相关组件

- `_DeepLinkButton`：用于创建深度链接测试按钮的私有组件
- `LoginScreen`：登录页面
- `SettingsScreen`：设置页面
- `App` 类中的路由配置：定义了所有路由和 `onEnter` 守卫逻辑

## 总结

`HomeScreen` 是一个功能丰富的演示页面，展示了 `go_router` 包的核心功能，包括基础导航、深度链接处理、路由守卫、重定向和 URL 片段支持。它通过交互式按钮让开发者可以直观地测试和理解这些功能的工作原理。
