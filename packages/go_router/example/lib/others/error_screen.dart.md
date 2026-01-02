# error_screen.dart 代码解析

## 概述

这是一个展示如何在 GoRouter 中实现自定义错误页面的完整示例。当用户访问不存在的路由或发生路由错误时，应用会显示一个自定义的错误页面，而不是使用默认的错误处理。

## 文件结构

该文件包含以下主要组件：

1. **主应用入口** (`main` 函数)
2. **应用主类** (`App`)
3. **页面组件** (`Page1Screen`、`Page2Screen`)
4. **错误页面组件** (`ErrorScreen`)

## 代码详解

### 应用入口

```dart 8:8:example/lib/others/error_screen.dart
void main() => runApp(App());
```

标准的 Flutter 应用入口点，启动 `App` 组件。

### 主应用类

```dart 11:38:example/lib/others/error_screen.dart
/// The main app.
class App extends StatelessWidget {
  /// Creates an [App].
  App({super.key});

  /// The title of the app.
  static const String title = 'GoRouter Example: Custom Error Screen';

  @override
  Widget build(BuildContext context) =>
      MaterialApp.router(routerConfig: _router, title: title);

  final GoRouter _router = GoRouter(
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
    ],
    errorBuilder: (BuildContext context, GoRouterState state) =>
        ErrorScreen(state.error!),
  );
}
```

`App` 类是应用的核心，负责配置路由系统。关键点：

- **路由配置**：定义了两个路由
  - `/`：首页，显示 `Page1Screen`
  - `/page2`：第二页，显示 `Page2Screen`
- **错误处理**：通过 `errorBuilder` 自定义错误页面
  - 当路由匹配失败或发生错误时触发
  - 接收 `BuildContext` 和 `GoRouterState`
  - 使用 `state.error!` 获取错误信息并传递给 `ErrorScreen`

### 首页组件

```dart 41:60:example/lib/others/error_screen.dart
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

首页包含：

- 应用栏显示应用标题
- 居中按钮，点击后使用 `context.go('/page2')` 导航到第二页
- 使用 `context.go()` 进行声明式导航

### 第二页组件

```dart 63:82:example/lib/others/error_screen.dart
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

第二页结构与首页类似：

- 应用栏显示标题
- 按钮用于返回首页（`context.go('/')`）

### 错误页面组件

```dart 85:108:example/lib/others/error_screen.dart
/// The screen of the error page.
class ErrorScreen extends StatelessWidget {
  /// Creates an [ErrorScreen].
  const ErrorScreen(this.error, {super.key});

  /// The error to display.
  final Exception error;

  @override
  Widget build(BuildContext context) => Scaffold(
    appBar: AppBar(title: const Text('My "Page Not Found" Screen')),
    body: Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: <Widget>[
          SelectableText(error.toString()),
          TextButton(
            onPressed: () => context.go('/'),
            child: const Text('Home'),
          ),
        ],
      ),
    ),
  );
}
```

错误页面特点：

- **错误信息展示**：通过 `SelectableText` 显示错误信息（可选中复制）
- **错误对象**：接收 `Exception` 类型，通过构造函数传入
- **返回首页**：提供按钮返回首页
- **自定义标题**：应用栏标题为 `'My "Page Not Found" Screen'`

## 关键特性

### 1. 自定义错误处理

通过 `errorBuilder` 实现自定义错误页面：

```dart 35:36:example/lib/others/error_screen.dart
    errorBuilder: (BuildContext context, GoRouterState state) =>
        ErrorScreen(state.error!),
```

触发场景：

- 访问未定义的路由（如 `/unknown`）
- 路由匹配失败
- 路由构建过程中抛出异常

### 2. 错误信息传递

`GoRouterState` 的 `error` 属性包含错误信息，通过 `state.error!` 传递给错误页面组件。

### 3. 导航一致性

错误页面使用 `context.go('/')` 返回首页，与正常页面导航方式一致。

## 使用场景

1. **404 页面**：访问不存在的路由时显示友好提示
2. **错误展示**：显示路由错误详情，便于调试
3. **用户体验**：提供返回首页的快捷方式
4. **错误处理**：统一处理路由相关异常

## 测试建议

可以通过以下方式测试错误页面：

1. 访问未定义的路由，如 `/unknown`
2. 在路由构建器中抛出异常
3. 访问需要参数但未提供参数的路由

## 扩展建议

- 根据错误类型显示不同内容
- 添加错误日志记录
- 提供更多导航选项（如返回上一页）
- 美化错误页面 UI
- 添加错误重试机制
