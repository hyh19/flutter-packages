# 导航

在应用中有多种方式可以在不同目标页面之间导航。

## 直接导航到目标页面

在 GoRouter 中导航到目标页面时，会将当前的屏幕堆栈替换为为目标路由配置显示的屏幕。要切换到新屏幕，使用 URL 调用 `context.go()`：

```dart
build(BuildContext context) {
  return TextButton(
    onPressed: () => context.go('/users/123'),
  );
}
```

这是调用 `GoRouter.of(context).go('/users/123)` 的简写形式。

要构建包含查询参数的 URI，可以使用 Dart 标准库中的 `Uri` 类：

```dart
context.go(Uri(path: '/users/123', queryParameters: {'filter': 'abc'}).toString());
```

## 命令式导航

GoRouter 可以使用 `context.push()` 将屏幕推入 Navigator 的历史堆栈，也可以通过 `context.pop()` 弹出当前屏幕。但是，命令式导航已知会导致浏览器历史记录问题。

要了解更多信息，请参阅 [issue #99112](https://github.com/flutter/flutter/issues/99112)。

## 使用 Link 组件

你可以使用 url_launcher 包中的 Link 组件来创建指向应用中目标页面的链接。这等同于调用 `context.go()`，但在 Web 上会渲染为真实的链接。

要在应用中添加 Link，请遵循 url_launcher 包中的 [Link API 文档](https://pub.dev/documentation/url_launcher/latest/link/Link-class.html)。

## 使用命名路由

你也可以使用[命名路由]来导航，而不是使用 URL。

## 阻止导航

GoRouter 和其他基于 Router 的 API 与 [WillPopScope](https://api.flutter.dev/flutter/widgets/WillPopScope-class.html) 组件不兼容。

有关此类 API 在 go_router 中的可能实现方式，请参阅 [issue #102408](https://github.com/flutter/flutter/issues/102408)。

## 导航时禁用浏览器历史记录跟踪

要在导航时禁用浏览器历史记录跟踪，请使用 `Router` 类的 `neglect` 方法：

```dart
ElevatedButton(
  onPressed: () => Router.neglect(
    context,
    () => context.go('/destination'),
  ),
  child: ...
),
```

要为**整个**应用禁用浏览器历史记录跟踪，请将 `GoRouter` 组件的 `routerNeglect` 属性设置为 `true`：

```dart
final _router = GoRouter(
  routerNeglect: true,
  routes: [
    ...
  ],
);
```

## 使用 Navigator 进行命令式导航

你可以继续使用 Navigator 来推送和弹出页面。以这种方式显示的页面不支持深度链接，并且如果任何与 GoRoute 关联的父页面被移除（例如，当发生新的 `go()` 调用时），这些页面将被替换。

要使用命令式 Navigator API 推送屏幕，请调用 [`NavigatorState.push()`](https://api.flutter.dev/flutter/widgets/NavigatorState/push.html)：

```dart
Navigator.of(context).push(
  MaterialPageRoute(
    builder: (BuildContext context) {
      return const DetailsScreen();
    },
  ),
);
```

行为可能会根据当前屏幕和新屏幕的 shell 路由而改变。

如果将一个没有任何 shell 路由的新屏幕推送到具有 shell 路由的当前屏幕上，新屏幕将完全放置在当前屏幕的顶部。

![动画展示了在当前屏幕顶部推送新屏幕](https://flutter.github.io/assets-for-api-docs/assets/go_router/push_regular_route.gif)

如果推送一个与当前屏幕具有相同 shell 路由的新屏幕，新屏幕将放置在 shell 内部。

![动画展示了推送一个与当前屏幕具有相同 shell 的新屏幕](https://flutter.github.io/assets-for-api-docs/assets/go_router/push_same_shell.gif)

如果推送一个与当前屏幕具有不同 shell 路由的新屏幕，新屏幕及其 shell 将完全放置在当前屏幕的顶部。

![动画展示了推送一个与当前屏幕具有不同 shell 的新屏幕](https://flutter.github.io/assets-for-api-docs/assets/go_router/push_different_shell.gif)

要亲自尝试此行为，请参阅 [push_with_shell_route.dart](https://github.com/flutter/packages/blob/main/packages/go_router/example/lib/push_with_shell_route.dart)。

## 返回值

等待返回值：

```dart
onTap: () async {
  final bool? result = await context.push<bool>('/page2');
  if(result ?? false)...
}
```

返回值：

```dart
onTap: () => context.pop(true)
```

## 使用 extra

你可以在导航时提供额外的数据。

```dart
context.go('/123', extra: 'abc');
```

并从 GoRouterState 中检索数据：

```dart
final String extraString = GoRouterState.of(context).extra! as String;
```

额外数据在存储到浏览器时会经过序列化。如果你计划使用复杂数据作为 extra，请考虑同时为 GoRouter 提供一个编解码器，这样在序列化过程中数据就不会丢失。

有关如何在 extra 中使用复杂数据和编解码器的示例，请参阅 [extra_codec.dart](https://github.com/flutter/packages/blob/main/packages/go_router/example/lib/extra_codec.dart)。

[命名路由]: https://pub.dev/documentation/go_router/latest/topics/Named%20routes-topic.html
