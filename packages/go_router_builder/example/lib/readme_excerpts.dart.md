# readme_excerpts.dart 代码解释

## 文件概述

`readme_excerpts.dart` 是一个示例文件，用于展示 `go_router_builder` 包的各种用法。该文件包含了大量使用 `// #docregion` 和 `// #enddocregion` 标记的代码片段，这些标记用于从源代码中提取文档示例。

文件的主要目的是：

- 演示如何使用类型安全的路由定义
- 展示各种路由参数的使用方法
- 提供导航、重定向、错误处理等功能的示例
- 作为文档生成的源代码

## 文件结构

文件主要包含以下几个部分：

1. **导入和配置**（第 7-13 行）
2. **其他示例代码片段**（第 15-82 行）
3. **路由定义类**（第 84-424 行）
4. **UI 组件类**（散落在各个路由定义之间）

## 导入和配置

```dart 7:13:example/lib/readme_excerpts.dart
import 'package:flutter/material.dart';
import 'shared/data.dart';
// #docregion import
import 'package:go_router/go_router.dart';

part 'readme_excerpts.g.dart';
// #enddocregion import
```

文件导入了 Flutter Material 库和 `go_router` 包，并声明了一个 part 文件 `readme_excerpts.g.dart`。这个 `.g.dart` 文件是由代码生成器自动生成的，包含了路由相关的辅助代码。

## 基础路由示例

### 传统 GoRoute 定义

```dart 16:25:example/lib/readme_excerpts.dart
  // #docregion GoRoute
  GoRoute(
    path: ':familyId',
    builder: (BuildContext context, GoRouterState state) {
      // Require the familyId to be present and be an integer.
      final int familyId = int.parse(state.pathParameters['familyId']!);
      return FamilyScreen(familyId);
    },
  );
  // #enddocregion GoRoute
```

这个示例展示了传统的 `GoRoute` 定义方式，需要手动从 `state.pathParameters` 中提取参数并进行类型转换。这种方式容易出现类型错误，如下面的错误示例所示：

```dart 27:30:example/lib/readme_excerpts.dart
  // #docregion GoWrong
  void tap() =>
      context.go('/familyId/a42'); // This is an error: `a42` is not an `int`.
  // #enddocregion GoWrong
```

### GoRouter 初始化

```dart 32:34:example/lib/readme_excerpts.dart
  // #docregion GoRouter
  final router = GoRouter(routes: $appRoutes);
  // #enddocregion GoRouter
```

这里使用了 `$appRoutes`，这是由代码生成器生成的变量，包含了所有类型安全的路由定义。

### 错误处理配置

```dart 36:43:example/lib/readme_excerpts.dart
  // #docregion routerWithErrorBuilder
  final routerWithErrorBuilder = GoRouter(
    routes: $appRoutes,
    errorBuilder: (BuildContext context, GoRouterState state) {
      return ErrorRoute(error: state.error!).build(context, state);
    },
  );
  // #enddocregion routerWithErrorBuilder
```

通过 `errorBuilder` 可以自定义错误页面的构建逻辑。

## 类型安全的路由定义

### 基础路由类

```dart 84:98:example/lib/readme_excerpts.dart
// #docregion TypedGoRouteHomeRoute
@TypedGoRoute<HomeRoute>(
  path: '/',
  routes: <TypedGoRoute<GoRouteData>>[
    TypedGoRoute<FamilyRoute>(path: 'family/:fid'),
  ],
)
// #docregion HomeRoute
class HomeRoute extends GoRouteData with $HomeRoute {
  const HomeRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) => const HomeScreen();
}
// #enddocregion HomeRoute
```

这是类型安全路由的核心用法：

- `@TypedGoRoute` 注解用于标记路由类
- `path` 指定路由路径
- `routes` 可以定义子路由
- 路由类需要继承 `GoRouteData` 并混入生成的 mixin（如 `$HomeRoute`）
- 实现 `build` 方法返回对应的 Widget

### 带查询参数的路由

```dart 212:223:example/lib/readme_excerpts.dart
// #docregion MyRoute
@TypedGoRoute<MyRoute>(path: '/my-route')
class MyRoute extends GoRouteData with $MyRoute {
  MyRoute({this.queryParameter = 'defaultValue'});
  final String queryParameter;

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return MyScreen(queryParameter: queryParameter);
  }
}
// #enddocregion MyRoute
```

`MyRoute` 展示了如何使用查询参数，参数可以有默认值。

### 登录路由示例

```dart 110:121:example/lib/readme_excerpts.dart
// #docregion login
@TypedGoRoute<LoginRoute>(path: '/login')
class LoginRoute extends GoRouteData with $LoginRoute {
  LoginRoute({this.from});
  final String? from;

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return LoginScreen(from: from);
  }
}
// #enddocregion login
```

`LoginRoute` 包含一个可选的 `from` 参数，用于记录用户从哪个页面跳转到登录页。

## 导航方法

### go 方法

```dart 45:47:example/lib/readme_excerpts.dart
  // #docregion go
  void onTap() => const FamilyRoute(fid: 'f2').go(context);
  // #enddocregion go
```

类型安全的路由可以使用 `.go(context)` 方法进行导航，参数在编译时就会进行类型检查，避免了运行时错误：

```dart 49:52:example/lib/readme_excerpts.dart
  // #docregion goError
  // This is an error: missing required parameter 'fid'.
  void errorTap() => const FamilyRoute().go(context);
  // #enddocregion goError
```

### push 方法和返回值

```dart 133:137:example/lib/readme_excerpts.dart
          // #docregion awaitPush
          final bool? result = await const FamilyRoute(
            fid: 'John',
          ).push<bool>(context);
          // #enddocregion awaitPush
```

`push` 方法可以导航到新页面并等待返回结果。类型参数 `<bool>` 指定了返回值的类型。

### goRelative 方法

```dart 60:62:example/lib/readme_excerpts.dart
  // #docregion goRelative
  void onTapRelative() => const DetailsRoute().goRelative(context);
  // #enddocregion goRelative
```

`goRelative` 方法用于相对路径导航，适用于嵌套路由场景。

## 路由参数类型

### 路径参数

路径参数通过路径模式定义，例如 `FamilyRoute`：

```dart 146:155:example/lib/readme_excerpts.dart
class FamilyRoute extends GoRouteData with $FamilyRoute {
  const FamilyRoute({this.fid});

  final String? fid;

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return FamilyScreen(int.parse(fid!));
  }
}
```

路径中定义了 `:fid` 参数，在路由类中作为字段使用。

### 查询参数

查询参数通过路由类的可选参数定义，如 `MyRoute` 中的 `queryParameter`。

### $extra 参数

```dart 235:246:example/lib/readme_excerpts.dart
@TypedGoRoute<PersonRouteWithExtra>(path: '/person')
// #docregion PersonRouteWithExtra
class PersonRouteWithExtra extends GoRouteData with $PersonRouteWithExtra {
  PersonRouteWithExtra(this.$extra);
  final Person? $extra;

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return PersonScreen($extra);
  }
}
// #enddocregion PersonRouteWithExtra
```

`$extra` 是一个特殊的参数，用于传递复杂对象。使用时：

```dart 54:58:example/lib/readme_excerpts.dart
  // #docregion tapWithExtra
  void tapWithExtra() {
    PersonRouteWithExtra(Person(id: 1, name: 'Marvin', age: 42)).go(context);
  }
  // #enddocregion tapWithExtra
```

### 混合参数示例

```dart 258:272:example/lib/readme_excerpts.dart
// #docregion HotdogRouteWithEverything
@TypedGoRoute<HotdogRouteWithEverything>(path: '/:ketchup')
class HotdogRouteWithEverything extends GoRouteData
    with $HotdogRouteWithEverything {
  HotdogRouteWithEverything(this.ketchup, this.mustard, this.$extra);
  final bool ketchup; // A required path parameter.
  final String? mustard; // An optional query parameter.
  final Sauce $extra; // A special $extra parameter.

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return HotdogScreen(ketchup, mustard, $extra);
  }
}
// #enddocregion HotdogRouteWithEverything
```

`HotdogRouteWithEverything` 展示了如何同时使用路径参数（`ketchup`）、查询参数（`mustard`）和 `$extra` 参数。

### 枚举参数

```dart 288:301:example/lib/readme_excerpts.dart
// #docregion BookKind
enum BookKind { all, popular, recent }

@TypedGoRoute<BooksRoute>(path: '/books')
class BooksRoute extends GoRouteData with $BooksRoute {
  BooksRoute({this.kind = BookKind.popular});
  final BookKind kind;

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return BooksScreen(kind: kind);
  }
}
// #enddocregion BookKind
```

路由参数也可以使用枚举类型，并设置默认值。

## 重定向

### 路由级别的重定向

```dart 100:108:example/lib/readme_excerpts.dart
// #docregion RedirectRoute
class RedirectRoute extends GoRouteData {
  // There is no need to implement [build] when this [redirect] is unconditional.
  @override
  String? redirect(BuildContext context, GoRouterState state) {
    return const HomeRoute().location;
  }
}
// #enddocregion RedirectRoute
```

当一个路由只用于重定向时，可以不实现 `build` 方法，只实现 `redirect` 方法。

### GoRouter 级别的重定向

```dart 66:81:example/lib/readme_excerpts.dart
  final routerWithRedirect = GoRouter(
    routes: $appRoutes,
    // #docregion redirect
    redirect: (BuildContext context, GoRouterState state) {
      final bool loggedIn = loginInfo.loggedIn;
      final loggingIn = state.matchedLocation == LoginRoute().location;
      if (!loggedIn && !loggingIn) {
        return LoginRoute(from: state.matchedLocation).location;
      }
      if (loggedIn && loggingIn) {
        return const HomeRoute().location;
      }
      return null;
    },
    // #enddocregion redirect
  );
```

在 `GoRouter` 级别设置 `redirect` 可以处理全局重定向逻辑，比如登录验证。

## 错误处理

```dart 176:186:example/lib/readme_excerpts.dart
// #docregion ErrorRoute
class ErrorRoute extends GoRouteData {
  ErrorRoute({required this.error});
  final Exception error;

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return ErrorScreen(error: error);
  }
}
// #enddocregion ErrorRoute
```

`ErrorRoute` 用于处理路由错误，接收一个 `Exception` 对象并显示错误信息。

## 自定义页面构建

### MaterialPage 与 Key

```dart 313:323:example/lib/readme_excerpts.dart
@TypedGoRoute<MyMaterialRouteWithKey>(path: '/my-material-route-with-key')
// #docregion MyMaterialRouteWithKey
class MyMaterialRouteWithKey extends GoRouteData with $MyMaterialRouteWithKey {
  const MyMaterialRouteWithKey();
  static const LocalKey _key = ValueKey<String>('my-route-with-key');
  @override
  MaterialPage<void> buildPage(BuildContext context, GoRouterState state) {
    return const MaterialPage<void>(key: _key, child: MyPage());
  }
}
// #enddocregion MyMaterialRouteWithKey
```

通过重写 `buildPage` 方法，可以自定义页面构建方式，比如指定 `MaterialPage` 的 `key`。

### 自定义转场动画

```dart 347:371:example/lib/readme_excerpts.dart
@TypedGoRoute<FancyRoute>(path: '/fancy')
// #docregion FancyRoute
class FancyRoute extends GoRouteData with $FancyRoute {
  const FancyRoute();
  @override
  CustomTransitionPage<void> buildPage(
    BuildContext context,
    GoRouterState state,
  ) {
    return CustomTransitionPage<void>(
      key: state.pageKey,
      child: const MyPage(),
      transitionsBuilder:
          (
            BuildContext context,
            Animation<double> animation,
            Animation<double> secondaryAnimation,
            Widget child,
          ) {
            return RotationTransition(turns: animation, child: child);
          },
    );
  }
}
// #enddocregion FancyRoute
```

`FancyRoute` 展示了如何实现自定义的页面转场动画，使用 `CustomTransitionPage` 和 `transitionsBuilder`。

## Shell Route（壳路由）

```dart 373:403:example/lib/readme_excerpts.dart
// #docregion MyShellRouteData
final GlobalKey<NavigatorState> shellNavigatorKey = GlobalKey<NavigatorState>();
final GlobalKey<NavigatorState> rootNavigatorKey = GlobalKey<NavigatorState>();

@TypedShellRoute<MyShellRouteData>(
  routes: <TypedRoute<RouteData>>[
    TypedGoRoute<MyGoRouteData>(path: 'my-go-route'),
  ],
)
class MyShellRouteData extends ShellRouteData {
  const MyShellRouteData();

  static final GlobalKey<NavigatorState> $navigatorKey = shellNavigatorKey;

  @override
  Widget builder(BuildContext context, GoRouterState state, Widget navigator) {
    return MyShellRoutePage(navigator);
  }
}

// For GoRoutes:
class MyGoRouteData extends GoRouteData with $MyGoRouteData {
  const MyGoRouteData();

  static final GlobalKey<NavigatorState> $parentNavigatorKey = rootNavigatorKey;

  @override
  Widget build(BuildContext context, GoRouterState state) => const MyPage();
}

// #enddocregion MyShellRouteData
```

Shell Route 用于创建一个包含多个子路由的公共布局：

- `@TypedShellRoute` 注解标记壳路由
- `ShellRouteData` 需要实现 `builder` 方法，接收 `navigator` 参数
- 通过 `$navigatorKey` 可以指定导航器的 key
- 子路由可以通过 `$parentNavigatorKey` 指定父导航器

## 相对路由

```dart 405:414:example/lib/readme_excerpts.dart
// #docregion relativeRoute
@TypedRelativeGoRoute<DetailsRoute>(path: 'details')
class DetailsRoute extends RelativeGoRouteData with $DetailsRoute {
  const DetailsRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) =>
      const DetailsScreen();
}
// #enddocregion relativeRoute
```

`@TypedRelativeGoRoute` 用于定义相对路由，路径是相对于父路由的。配合 `goRelative` 方法使用。

## 总结

这个文件全面展示了 `go_router_builder` 包的各种功能：

1. **类型安全**：通过代码生成实现编译时的类型检查
2. **参数处理**：支持路径参数、查询参数、枚举参数和 `$extra` 参数
3. **导航方法**：提供 `go`、`push`、`goRelative` 等方法
4. **高级特性**：重定向、错误处理、自定义页面构建、转场动画
5. **嵌套路由**：Shell Route 和相对路由

使用类型安全的路由定义可以大大减少路由相关的运行时错误，提高代码的可维护性和可读性。
