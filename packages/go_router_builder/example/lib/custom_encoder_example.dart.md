# Custom Encoder Example 代码解析

## 概述

这个示例展示了如何在 `go_router_builder` 中使用 `@CustomParameterCodec` 注解来自定义路由参数的编码和解码方式。通过自定义编码器，你可以控制路由参数在 URL 中的表示形式，这对于处理特殊字符、加密数据或需要特定格式的参数非常有用。

在本示例中，我们使用 Base64 编码来处理路由参数，确保参数值在 URL 中能够安全传输。

## 核心概念

### 自定义参数编码器

`@CustomParameterCodec` 注解允许你为路由参数指定自定义的编码和解码函数：

- **encode**：将参数值编码为 URL 中使用的字符串格式
- **decode**：从 URL 字符串中解码出原始参数值

## 代码结构解析

### 应用入口

```dart 14:24:example/lib/custom_encoder_example.dart
void main() => runApp(App());

class App extends StatelessWidget {
  App({super.key});

  @override
  Widget build(BuildContext context) =>
      MaterialApp.router(routerConfig: _router, title: _appTitle);

  final GoRouter _router = GoRouter(routes: $appRoutes);
}
```

应用入口非常简单，创建了一个 `App` 组件，使用 `MaterialApp.router` 配置路由。`$appRoutes` 是由代码生成器自动生成的路由列表。

### 主路由定义

```dart 26:38:example/lib/custom_encoder_example.dart
@TypedGoRoute<HomeRoute>(
  path: '/',
  name: 'Home',
  routes: <TypedGoRoute<GoRouteData>>[
    TypedGoRoute<EncodedRoute>(path: 'encoded'),
  ],
)
class HomeRoute extends GoRouteData with $HomeRoute {
  const HomeRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) => const HomeScreen();
}
```

`HomeRoute` 是应用的根路由，路径为 `/`。它包含一个子路由 `EncodedRoute`，路径为 `encoded`，完整路径为 `/encoded`。

### 使用自定义编码器的路由

```dart 40:49:example/lib/custom_encoder_example.dart
class EncodedRoute extends GoRouteData with $EncodedRoute {
  const EncodedRoute(this.token);

  @CustomParameterCodec(encode: toBase64, decode: fromBase64)
  final String token;

  @override
  Widget build(BuildContext context, GoRouterState state) =>
      EncodedScreen(token: token);
}
```

这是示例的核心部分。`EncodedRoute` 定义了一个带有 `token` 参数的路由，并使用 `@CustomParameterCodec` 注解指定了编码和解码函数：

- **encode: toBase64**：将 `token` 值编码为 Base64 字符串
- **decode: fromBase64**：从 Base64 字符串解码出原始的 `token` 值

这意味着：

- 当你使用 `EncodedRoute('Base64Token').go(context)` 导航时，代码生成器会自动将 `'Base64Token'` 编码为 Base64 格式放入 URL
- 当从 URL（如深度链接）解析路由时，代码生成器会自动将 URL 中的 Base64 字符串解码为原始值

### 编码和解码函数

```dart 84:92:example/lib/custom_encoder_example.dart
String fromBase64(String value) {
  return const Utf8Decoder().convert(
    base64Url.decode(base64Url.normalize(value)),
  );
}

String toBase64(String value) {
  return base64Url.encode(const Utf8Encoder().convert(value));
}
```

这两个函数实现了 Base64 URL 安全的编码和解码：

- **toBase64**：将字符串转换为 UTF-8 字节，然后进行 Base64 URL 安全编码
- **fromBase64**：将 Base64 URL 安全编码的字符串解码，然后转换为 UTF-8 字符串

使用 `base64Url` 而不是 `base64` 是因为 URL 中不能包含某些字符（如 `+`、`/`），而 `base64Url` 使用 `-` 和 `_` 替代这些字符，更适合在 URL 中使用。

### 主屏幕组件

```dart 51:71:example/lib/custom_encoder_example.dart
class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) => Scaffold(
    appBar: AppBar(title: const Text(_appTitle)),
    body: ListView(
      children: <Widget>[
        ListTile(
          title: const Text('Base64Token'),
          onTap: () => const EncodedRoute('Base64Token').go(context),
        ),
        ListTile(
          title: const Text('from url only'),
          // like in deep links
          onTap: () => context.go('/encoded?token=ZW5jb2RlZCBpbmZvIQ'),
        ),
      ],
    ),
  );
}
```

主屏幕展示了两种导航方式：

1. **类型安全的导航**（第 61 行）：
   - 使用 `EncodedRoute('Base64Token').go(context)`
   - 代码生成器会自动将 `'Base64Token'` 编码为 Base64 并构建正确的 URL
   - 这是推荐的方式，提供类型安全和编译时检查

2. **直接 URL 导航**（第 66 行）：
   - 使用 `context.go('/encoded?token=ZW5jb2RlZCBpbmZvIQ')`
   - 直接提供已编码的 URL（`ZW5jb2RlZCBpbmZvIQ` 是 `encoded info!` 的 Base64 编码）
   - 这种方式通常用于处理深度链接，从外部来源接收 URL

### 编码屏幕组件

```dart 73:82:example/lib/custom_encoder_example.dart
class EncodedScreen extends StatelessWidget {
  const EncodedScreen({super.key, required this.token});
  final String token;

  @override
  Widget build(BuildContext context) => Scaffold(
    appBar: AppBar(title: const Text('Base64Token')),
    body: Center(child: Text(token)),
  );
}
```

这个屏幕简单地显示解码后的 `token` 值。无论 URL 中的 token 是 Base64 编码的，最终显示的都是解码后的原始值。

## 生成的代码分析

查看生成的代码可以帮助理解编码器是如何工作的：

```dart 42:52:example/lib/custom_encoder_example.g.dart
mixin $EncodedRoute on GoRouteData {
  static EncodedRoute _fromState(GoRouterState state) =>
      EncodedRoute(fromBase64(state.uri.queryParameters['token']!));

  EncodedRoute get _self => this as EncodedRoute;

  @override
  String get location => GoRouteData.$location(
    '/encoded',
    queryParams: {'token': toBase64(_self.token)},
  );
```

关键点：

1. **从 URL 解析路由**（第 43-44 行）：
   - `_fromState` 方法从 URL 查询参数中获取 `token`
   - 使用 `fromBase64` 函数解码 Base64 字符串
   - 创建 `EncodedRoute` 实例时传入解码后的值

2. **生成 URL**（第 49-51 行）：
   - `location` getter 使用 `toBase64` 函数将 `token` 编码为 Base64
   - 将编码后的值放入查询参数中

## 使用场景

自定义编码器适用于以下场景：

1. **特殊字符处理**：当参数值包含 URL 不安全的字符时
2. **数据加密**：需要在 URL 中传输敏感但需要加密的数据
3. **格式转换**：需要将复杂数据类型（如 JSON）编码为 URL 安全的字符串
4. **兼容性**：需要与现有 API 或系统保持 URL 格式兼容

## 工作流程

### 类型安全导航流程

1. 调用 `EncodedRoute('Base64Token').go(context)`
2. 代码生成器调用 `toBase64('Base64Token')` 进行编码
3. 生成 URL：`/encoded?token=QmFzZTY0VG9rZW4=`
4. 执行导航

### 深度链接解析流程

1. 接收到 URL：`/encoded?token=ZW5jb2RlZCBpbmZvIQ`
2. 代码生成器调用 `_fromState` 方法
3. 从查询参数中提取 `token`：`ZW5jb2RlZCBpbmZvIQ`
4. 调用 `fromBase64('ZW5jb2RlZCBpbmZvIQ')` 进行解码
5. 得到原始值：`encoded info!`
6. 创建 `EncodedRoute('encoded info!')` 实例
7. 渲染 `EncodedScreen` 并显示解码后的值

## 注意事项

1. **编码和解码必须匹配**：`encode` 和 `decode` 函数必须是对应的，否则会导致数据损坏
2. **URL 安全性**：确保编码后的字符串是 URL 安全的（不包含特殊字符）
3. **错误处理**：在实际应用中，应该处理解码失败的情况（如无效的 Base64 字符串）
4. **性能考虑**：编码和解码操作会在每次导航时执行，对于频繁导航的场景需要考虑性能影响

## 总结

这个示例展示了 `go_router_builder` 中自定义参数编码器的强大功能。通过 `@CustomParameterCodec` 注解，你可以完全控制路由参数在 URL 中的表示形式，同时保持类型安全的导航体验。无论是处理特殊字符、加密数据，还是需要特定格式，自定义编码器都能满足你的需求。
