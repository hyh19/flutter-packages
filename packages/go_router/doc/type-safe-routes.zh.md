# 类型安全路由

除了使用 URL 字符串进行导航外，go_router 还支持使用 go_router_builder 包实现类型安全路由。

要开始使用，请在 pubspec.yaml 的 dev_dependencies 部分添加 [go_router_builder][]、[build_runner][] 和 [build_verify][]：

```yaml
dev_dependencies:
  go_router_builder: any
  build_runner: any
  build_verify: any
```

然后为应用中的每个路由扩展 [GoRouteData](https://pub.dev/documentation/go_router/latest/go_router/GoRouteData-class.html) 类并添加 TypedGoRoute 注解：

```dart
import 'package:go_router/go_router.dart';

part 'go_router_builder.g.dart';

@TypedGoRoute<HomeScreenRoute>(
    path: '/',
    routes: [
      TypedGoRoute<SongRoute>(
        path: 'song/:id',
      )
    ]
)
@immutable
class HomeScreenRoute extends GoRouteData with _$HomeScreenRoute {
  @override
  Widget build(BuildContext context, GoRouterState state) {
    return const HomeScreen();
  }
}

@immutable
class SongRoute extends GoRouteData with _$SongRoute {
  final int id;

  const SongRoute({
    required this.id,
  });

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return SongScreen(songId: id.toString());
  }
}
```

要构建生成的文件（以 .g.dart 结尾），请使用 build_runner 命令：

```bash
flutter pub global activate build_runner
flutter pub run build_runner build
```

要进行导航，请使用所需参数构造一个 GoRouteData 对象并调用 go()：

```dart
TextButton(
  onPressed: () {
    const SongRoute(id: 2).go(context);
  },
  child: const Text('Go to song 2'),
),
```

更多信息，请访问 [go_router_builder 包文档](https://pub.dev/documentation/go_router_builder/latest/)。

[go_router_builder]: https://pub.dev/packages/go_router_builder
[build_runner]: https://pub.dev/packages/build_runner
[build_verify]: https://pub.dev/packages/build_verify
