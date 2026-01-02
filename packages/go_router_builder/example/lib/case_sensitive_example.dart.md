# case_sensitive_example.dart 代码解析

## 概述

这个示例文件展示了如何在 `go_router_builder` 中使用路由的大小写敏感性控制功能。通过对比两个路由配置，演示了 `caseSensitive` 参数对路由匹配行为的影响。

## 文件结构

文件包含以下主要组件：

1. **应用入口**：`CaseSensitivityApp` 类
2. **大小写敏感路由**：`CaseSensitiveRoute` 类
3. **大小写不敏感路由**：`NotCaseSensitiveRoute` 类
4. **通用屏幕组件**：`Screen` 类

## 应用入口

```dart 12:25:example/lib/case_sensitive_example.dart
void main() => runApp(CaseSensitivityApp());

class CaseSensitivityApp extends StatelessWidget {
  CaseSensitivityApp({super.key});

  @override
  Widget build(BuildContext context) =>
      MaterialApp.router(routerConfig: _router);

  final GoRouter _router = GoRouter(
    initialLocation: '/case-sensitive',
    routes: $appRoutes,
  );
}
```

`CaseSensitivityApp` 是应用的根组件，它：

- 使用 `MaterialApp.router` 配置路由
- 通过 `$appRoutes` 获取所有生成的路由（由代码生成器自动生成）
- 设置初始位置为 `/case-sensitive`

## 大小写敏感路由

```dart 27:34:example/lib/case_sensitive_example.dart
@TypedGoRoute<CaseSensitiveRoute>(path: '/case-sensitive')
class CaseSensitiveRoute extends GoRouteData with $CaseSensitiveRoute {
  const CaseSensitiveRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) =>
      const Screen(title: 'Case Sensitive');
}
```

`CaseSensitiveRoute` 是一个**大小写敏感**的路由：

- 使用 `@TypedGoRoute` 注解，路径为 `/case-sensitive`
- **默认情况下**，路由是大小写敏感的（`caseSensitive` 默认为 `true`）
- 这意味着只有完全匹配 `/case-sensitive` 的路径才能匹配此路由
- 例如：`/Case-Sensitive`、`/CASE-SENSITIVE` 等都不会匹配

## 大小写不敏感路由

```dart 36:46:example/lib/case_sensitive_example.dart
@TypedGoRoute<NotCaseSensitiveRoute>(
  path: '/not-case-sensitive',
  caseSensitive: false,
)
class NotCaseSensitiveRoute extends GoRouteData with $NotCaseSensitiveRoute {
  const NotCaseSensitiveRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) =>
      const Screen(title: 'Not Case Sensitive');
}
```

`NotCaseSensitiveRoute` 是一个**大小写不敏感**的路由：

- 通过设置 `caseSensitive: false` 来禁用大小写敏感性
- 这意味着路径匹配时不区分大小写
- 例如：`/not-case-sensitive`、`/Not-Case-Sensitive`、`/NOT-CASE-SENSITIVE` 等都能匹配此路由

## 生成的代码

代码生成器会根据注解生成相应的路由配置。从生成的 `case_sensitive_example.g.dart` 文件中可以看到：

### 大小写敏感路由的生成代码

```dart 13:16:example/lib/case_sensitive_example.g.dart
RouteBase get $caseSensitiveRoute => GoRouteData.$route(
  path: '/case-sensitive',
  factory: $CaseSensitiveRoute._fromState,
);
```

注意：这里**没有** `caseSensitive` 参数，因为默认值就是 `true`。

### 大小写不敏感路由的生成代码

```dart 39:43:example/lib/case_sensitive_example.g.dart
RouteBase get $notCaseSensitiveRoute => GoRouteData.$route(
  path: '/not-case-sensitive',
  caseSensitive: false,
  factory: $NotCaseSensitiveRoute._fromState,
);
```

这里**显式**设置了 `caseSensitive: false`。

## UI 组件

```dart 48:68:example/lib/case_sensitive_example.dart
class Screen extends StatelessWidget {
  const Screen({required this.title, super.key});

  final String title;
  @override
  Widget build(BuildContext context) => Scaffold(
    appBar: AppBar(title: Text(title)),
    body: ListView(
      children: <Widget>[
        ListTile(
          title: const Text('Case Sensitive'),
          onTap: () => context.go('/case-sensitive'),
        ),
        ListTile(
          title: const Text('Not Case Sensitive'),
          onTap: () => context.go('/not-case-sensitive'),
        ),
      ],
    ),
  );
}
```

`Screen` 是一个通用的屏幕组件，用于显示：

- 一个带有标题的 `AppBar`
- 一个包含两个导航选项的 `ListView`
- 每个选项点击后会导航到对应的路由

## 使用场景

### 何时使用大小写敏感路由

- 需要精确匹配 URL 路径
- 路径大小写具有语义意义
- 需要区分不同大小写的路径（如 RESTful API 路由）

### 何时使用大小写不敏感路由

- 提高用户体验，避免因大小写输入错误导致路由匹配失败
- 处理用户输入的 URL（如深度链接）
- 兼容不同来源的链接（可能大小写不一致）

## 实际行为示例

假设用户访问以下路径：

| 路径 | 大小写敏感路由 | 大小写不敏感路由 |
| --- | --- | --- |
| `/case-sensitive` | ✅ 匹配 | ❌ 不匹配 |
| `/Case-Sensitive` | ❌ 不匹配 | ❌ 不匹配 |
| `/not-case-sensitive` | ❌ 不匹配 | ✅ 匹配 |
| `/Not-Case-Sensitive` | ❌ 不匹配 | ✅ 匹配 |
| `/NOT-CASE-SENSITIVE` | ❌ 不匹配 | ✅ 匹配 |

## 技术细节

### Mixin 的使用

每个路由类都使用了对应的 mixin（如 `$CaseSensitiveRoute`），这些 mixin 由代码生成器自动生成，提供了：

- `location` getter：获取路由位置
- `go()` 方法：导航到该路由
- `push()` 方法：推入该路由
- `pushReplacement()` 方法：替换当前路由
- `replace()` 方法：替换路由

### 路由工厂方法

每个 mixin 都包含一个 `_fromState` 静态方法，用于从 `GoRouterState` 创建路由实例：

```dart 19:20:example/lib/case_sensitive_example.g.dart
  static CaseSensitiveRoute _fromState(GoRouterState state) =>
      const CaseSensitiveRoute();
```

这个方法在路由匹配时被调用，用于创建路由数据对象。

## 总结

这个示例清晰地展示了：

1. **默认行为**：路由默认是大小写敏感的
2. **配置方式**：通过 `caseSensitive: false` 可以禁用大小写敏感性
3. **实际应用**：根据需求选择合适的配置，平衡精确性和用户体验

通过这个示例，开发者可以理解如何在 `go_router_builder` 中控制路由的大小写匹配行为，从而创建更灵活和用户友好的路由系统。
