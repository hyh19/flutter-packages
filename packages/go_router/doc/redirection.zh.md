# 重定向

重定向根据应用状态将位置更改为新位置。例如，如果用户未登录，可以使用重定向来显示登录屏幕。

重定向是类型为 [GoRouterRedirect](https://pub.dev/documentation/go_router/latest/go_router/GoRouterRedirect.html) 的回调。要根据某些应用状态更改传入位置，请向 GoRouter 或 GoRoute 构造函数添加回调：

```dart
redirect: (BuildContext context, GoRouterState state) {
  if (!AuthState.of(context).isSignedIn) {
    return '/signin';
  } else {
    return null;
  }   
},
```

要显示预期路由而不进行重定向，请返回 `null` 或原始路由路径。

## 顶级重定向与路由级重定向

有两种类型的重定向：

- 顶级重定向：在 `GoRouter` 构造函数上定义。在任何导航事件之前调用。
- 路由级重定向：在 `GoRoute` 构造函数上定义。当导航事件即将显示路由时调用。

## 命名路由

你也可以使用[命名路由]进行重定向。

## 注意事项

- 你可以指定 `redirectLimit` 来配置应用中预期发生的最大重定向次数。默认情况下，此值设置为 5。如果超过此重定向限制，GoRouter 将显示错误屏幕（有关错误屏幕的更多信息，请参阅[错误处理][]主题。）

[命名路由]: https://pub.dev/documentation/go_router/latest/topics/Named%20routes-topic.html
[错误处理]: https://pub.dev/documentation/go_router/topics/Error%20handling-topic.html
