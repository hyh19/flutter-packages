# 错误处理

go_router 中有几种错误或异常类型。

## GoError 和 AssertionError

这类错误在 go_router 使用不当时会抛出，例如，如果根路由的
[GoRoute.path](https://pub.dev/documentation/go_router/latest/go_router/GoRoute/path.html) 不是以 `/` 开头，
或者 GoRoute 中的 builder 未提供。这些错误不应被捕获，必须修复代码才能使用 go_router。

## GoException

当 go_router 的配置无法处理来自用户或代码其他部分的请求时，会抛出此类异常。
例如，当用户输入的 URL 无法根据 `GoRouter.routes` 中指定的模式解析时，会抛出 GoException。
这些异常可以在各种回调中处理。

可以通过提供回调到 `GoRouter.onException` 来处理此类异常。在此回调中，
可以根据情况选择忽略、重定向或推送不同的页面。
有关可运行的示例，请参阅 [异常处理](https://github.com/flutter/packages/blob/main/packages/go_router/example/lib/exception_handling.dart)。

`GoRouter.errorBuilder` 和 `GoRouter.errorPageBuilder` 也可用于处理异常。

```dart
GoRouter(
  /* ... */
  errorBuilder: (context, state) => ErrorScreen(state.error),
);
```

默认情况下，go_router 为 `MaterialApp` 和 `CupertinoApp` 提供了默认的错误页面，
以及在没有使用这些应用类型时的默认错误页面。

**注意**：`GoRouter.onException` 会优先于其他异常处理 API。
