# GoRouter 类概述

## 简介

`GoRouter` 是 go_router 包中的核心类，用于配置和管理应用的路由系统。它实现了 `RouterConfig<RouteMatchList>` 接口，提供了声明式的路由配置方式。

## 类文档注释

```dart 124:172:lib/src/router.dart
/// The route configuration for the app.
///
/// The `routes` list specifies the top-level routes for the app. It must not be
/// empty and must contain an [GoRoute] to match `/`.
///
/// See the [Get
/// started](https://github.com/flutter/packages/blob/main/packages/go_router/example/lib/main.dart)
/// example, which shows an app with a simple route configuration.
///
/// The [onEnter] callback allows intercepting navigation before routes are
/// processed. Return [Allow] to proceed or [Block] to prevent navigation.
/// Order of operations:
/// 1) `onEnter` (your guard) - can block navigation
/// 2) If allowed: legacy top-level `redirect` - runs in same navigation cycle
/// 3) route-level `GoRoute.redirect`
///
/// The [redirect] callback allows the app to redirect to a new location.
/// Alternatively, you can specify a redirect for an individual route using
/// [GoRoute.redirect]. If [BuildContext.dependOnInheritedWidgetOfExactType] is
/// used during the redirection (which is how `of` methods are usually
/// implemented), a re-evaluation will be triggered when the [InheritedWidget]
/// changes.
///
/// To handle exceptions, use one of `onException`, `errorBuilder`, or
/// `errorPageBuilder`. The `onException` is called when an exception is thrown.
/// If `onException` is not provided, the exception is passed to
/// `errorPageBuilder` to build a page for the Router if it is not null;
/// otherwise, it is passed to `errorBuilder` instead. If none of them are
/// provided, go_router builds a default error screen to show the exception.
/// See [Error handling](https://pub.dev/documentation/go_router/latest/topics/Error%20handling-topic.html)
/// for more details.
///
/// To disable automatically requesting focus when new routes are pushed to the navigator, set `requestFocus` to false.
///
/// See also:
/// * [Configuration](https://pub.dev/documentation/go_router/latest/topics/Configuration-topic.html)
/// * [GoRoute], which provides APIs to define the routing table.
/// * [examples](https://github.com/flutter/packages/tree/main/packages/go_router/example),
///    which contains examples for different routing scenarios.
/// {@category Get started}
/// {@category Upgrading}
/// {@category Configuration}
/// {@category Navigation}
/// {@category Redirection}
/// {@category Web}
/// {@category Deep linking}
/// {@category Error handling}
/// {@category Named routes}
/// {@category State restoration}
```

## 核心概念

### 路由配置

`GoRouter` 通过 `routes` 列表来定义应用的路由表。这个列表：

- **不能为空**：必须至少包含一个路由定义
- **必须包含根路由**：必须包含一个能匹配 `/` 路径的 `GoRoute`
- **定义顶层路由**：列表中的路由是应用的顶层路由，可以包含子路由

### 导航守卫：onEnter 和 redirect 的执行顺序

`GoRouter` 提供了多个拦截和重定向导航的机制，它们的执行顺序如下：

1. **`onEnter` 守卫**：在路由处理之前被调用，可以阻止导航（返回 `Block`）或允许导航（返回 `Allow`）
2. **顶层 `redirect` 回调**：如果 `onEnter` 允许导航，则在同一个导航周期内执行（这是为了向后兼容而保留的旧版 API）
3. **路由级 `GoRoute.redirect`**：针对特定路由的重定向逻辑

这种顺序确保了导航守卫在重定向之前执行，为应用提供了细粒度的导航控制。

### 错误处理机制

`GoRouter` 提供了三种错误处理方式，按优先级如下：

1. **`onException`**：如果提供，当异常抛出时会被调用，用于自定义异常处理逻辑
2. **`errorPageBuilder`**：如果 `onException` 未提供，异常会被传递给 `errorPageBuilder` 来构建一个页面（如果它不为 null）
3. **`errorBuilder`**：如果前两者都未提供，异常会被传递给 `errorBuilder`
4. **默认错误屏幕**：如果以上三种方式都未提供，go_router 会构建一个默认的错误屏幕来显示异常

这种分层设计允许开发者根据需要选择合适的错误处理方式。

### 焦点管理

`GoRouter` 默认会在新路由推入导航器时自动请求焦点。如果不需要这个行为，可以将 `requestFocus` 参数设置为 `false`。

### 文档分类标签

类文档注释中使用了多个 `{@category}` 标签，这些标签用于文档分类：

- `Get started`：入门指南
- `Upgrading`：升级指南
- `Configuration`：配置相关
- `Navigation`：导航相关
- `Redirection`：重定向相关
- `Web`：Web 平台相关
- `Deep linking`：深层链接相关
- `Error handling`：错误处理相关
- `Named routes`：命名路由相关
- `State restoration`：状态恢复相关

这些标签帮助用户在文档中找到相关的主题内容。

## 相关文档链接

- [Get started 示例](https://github.com/flutter/packages/blob/main/packages/go_router/example/lib/main.dart)
- [Configuration 文档](https://pub.dev/documentation/go_router/latest/topics/Configuration-topic.html)
- [Error handling 文档](https://pub.dev/documentation/go_router/latest/topics/Error%20handling-topic.html)
- [示例集合](https://github.com/flutter/packages/tree/main/packages/go_router/example)
