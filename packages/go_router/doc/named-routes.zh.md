# 命名路由

除了基于 URL 进行导航外，`GoRoute` 还可以被赋予一个唯一的名称。要配置命名路由，请使用 `name` 参数：

```dart
GoRoute(
   name: 'song',
   path: 'songs/:songId',
   builder: /* ... */,
 ),
```

要使用路由名称进行导航，请调用 [`goNamed`](https://pub.dev/documentation/go_router/latest/go_router/GoRouter/goNamed.html)：

```dart
TextButton(
  onPressed: () {
    context.goNamed('song', pathParameters: {'songId': 123});
  },
  child: const Text('Go to song 2'),
),
```

或者，你可以使用 `namedLocation` 查找名称对应的位置：

```dart
TextButton(
  onPressed: () {
    final String location = context.namedLocation('song', pathParameters: {'songId': 123});
    context.go(location);
  },
  child: const Text('Go to song 2'),
),
```

要了解更多关于导航的信息，请参阅 [Navigation][] 主题。

## 重定向到命名路由

要重定向到命名路由，请使用 `namedLocation` API：

```dart
redirect: (BuildContext context, GoRouterState state) {
  if (AuthState.of(context).isSignedIn) {
    return context.namedLocation('signIn');
  } else {
    return null;
  }   
},
```

要了解更多关于重定向的信息，请参阅 [Redirection][] 主题。

[Navigation]: https://pub.dev/documentation/go_router/latest/topics/Navigation-topic.html
[Redirection]: https://pub.dev/documentation/go_router/latest/topics/Redirection-topic.html
