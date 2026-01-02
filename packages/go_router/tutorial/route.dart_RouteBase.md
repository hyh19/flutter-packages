# RouteBase 类详解

## 概述

`RouteBase` 是 `go_router` 包中的核心抽象基类，它为 `GoRoute` 和 `ShellRoute` 提供了共同的基础结构。这个类定义了路由系统的基本属性和行为，包括路由树结构、重定向机制和导航器管理。

## 类定义

```dart 73:160:lib/src/route.dart
/// The base class for [GoRoute] and [ShellRoute].
///
/// Routes are defined in a tree such that parent routes must match the
/// current location for their child route to be considered a match. For
/// example the location "/home/user/12" matches with parent route "/home" and
/// child route "user/:userId".
///
/// To create sub-routes for a route, provide them as a [GoRoute] list
/// with the sub routes.
///
/// For example these routes:
/// ```none
/// /         => HomePage()
///   family/f1 => FamilyPage('f1')
///     person/p2 => PersonPage('f1', 'p2') ← showing this page, Back pops ↑
/// ```
///
/// Can be represented as:
///
/// ```dart
/// final GoRouter _router = GoRouter(
///   routes: <GoRoute>[
///     GoRoute(
///       path: '/',
///       pageBuilder: (BuildContext context, GoRouterState state) => MaterialPage<void>(
///         key: state.pageKey,
///         child: HomePage(families: Families.data),
///       ),
///       routes: <GoRoute>[
///         GoRoute(
///           path: 'family/:fid',
///           pageBuilder: (BuildContext context, GoRouterState state) {
///             final Family family = Families.family(state.pathParameters['fid']!);
///             return MaterialPage<void>(
///               key: state.pageKey,
///               child: FamilyPage(family: family),
///             );
///           },
///           routes: <GoRoute>[
///             GoRoute(
///               path: 'person/:pid',
///               pageBuilder: (BuildContext context, GoRouterState state) {
///                 final Family family = Families.family(state.pathParameters['fid']!);
///                 final Person person = family.person(state.pathParameters['pid']!);
///                 return MaterialPage<void>(
///                   key: state.pageKey,
///                   child: PersonPage(family: family, person: person),
///                 );
///               },
///             ),
///           ],
///         ),
///       ],
///     ),
///   ],
/// );
/// ```
///
/// If there are multiple routes that match the location, the first match is used.
/// To make predefined routes to take precedence over dynamic routes eg. '/:id'
/// consider adding the dynamic route at the end of the routes.
///
/// For example:
/// ```dart
/// final GoRouter _router = GoRouter(
///   routes: <GoRoute>[
///     GoRoute(
///       path: '/',
///       redirect: (_, __) => '/family/${Families.data[0].id}',
///     ),
///     GoRoute(
///       path: '/family',
///       pageBuilder: (BuildContext context, GoRouterState state) => ...,
///     ),
///     GoRoute(
///       path: '/:username',
///       pageBuilder: (BuildContext context, GoRouterState state) => ...,
///     ),
///   ],
/// );
/// ```
///
/// In the above example, if `/family` route is matched, it will be used.
/// else `/:username` route will be used.
///
/// See [main.dart](https://github.com/flutter/packages/blob/main/packages/go_router/example/lib/main.dart)
@immutable
abstract class RouteBase with Diagnosticable {
  const RouteBase._({
    this.redirect,
    required this.routes,
    required this.parentNavigatorKey,
  });
```

## 核心特性

### 1. 路由树结构

`RouteBase` 实现了树形路由结构，其中：

- **父路由必须匹配**：子路由只有在父路由匹配当前位置时才会被考虑匹配
- **路径继承**：子路由的路径是相对于父路由的
- **层级导航**：返回操作会按照路由树的层级向上弹出

例如，对于路径 `/home/user/12`：

- 父路由 `/home` 必须匹配
- 子路由 `user/:userId` 才会被匹配
- 参数 `userId` 的值是 `12`

### 2. 路由匹配优先级

当多个路由匹配同一位置时，遵循以下规则：

- **第一个匹配的路由会被使用**
- **静态路由优先于动态路由**：例如 `/family` 会优先于 `/:username` 匹配
- **建议将动态路由放在路由列表的末尾**，以确保静态路由优先匹配

## 属性详解

### redirect

```dart 167:221:lib/src/route.dart
  /// An optional redirect function for this route.
  ///
  /// In the case that you like to make a redirection decision for a specific
  /// route (or sub-route), consider doing so by passing a redirect function to
  /// the GoRoute constructor.
  ///
  /// For example:
  /// ```dart
  /// final GoRouter _router = GoRouter(
  ///   routes: <GoRoute>[
  ///     GoRoute(
  ///       path: '/',
  ///       redirect: (_, __) => '/family/${Families.data[0].id}',
  ///     ),
  ///     GoRoute(
  ///       path: '/family/:fid',
  ///       pageBuilder: (BuildContext context, GoRouterState state) => ...,
  ///     ),
  ///   ],
  /// );
  /// ```
  ///
  /// If there are multiple redirects in the matched routes, the parent route's
  /// redirect takes priority over sub-route's.
  ///
  /// For example:
  /// ```dart
  /// final GoRouter _router = GoRouter(
  ///   routes: <GoRoute>[
  ///     GoRoute(
  ///       path: '/',
  ///       redirect: (_, __) => '/page1', // this takes priority over the sub-route.
  ///       routes: <GoRoute>[
  ///         GoRoute(
  ///           path: 'child',
  ///           redirect: (_, __) => '/page2',
  ///         ),
  ///       ],
  ///     ),
  ///   ],
  /// );
  /// ```
  ///
  /// The `context.go('/child')` will be redirected to `/page1` instead of
  /// `/page2`.
  ///
  /// Redirect can also be used for conditionally preventing users from visiting
  /// routes, also known as route guards. One canonical example is user
  /// authentication. See [Redirection](https://github.com/flutter/packages/blob/main/packages/go_router/example/lib/redirection.dart)
  /// for a complete runnable example.
  ///
  /// If [BuildContext.dependOnInheritedWidgetOfExactType] is used during the
  /// redirection (which is how `of` method is usually implemented), a
  /// re-evaluation will be triggered if the [InheritedWidget] changes.
  final GoRouterRedirect? redirect;
```

**功能说明**：

- **可选的重定向函数**：用于在路由匹配时决定是否重定向到其他路由
- **路由守卫**：可以用于实现路由保护，例如用户认证检查
- **优先级规则**：如果路由树中有多个重定向，**父路由的重定向优先于子路由**
- **响应式重定向**：如果在重定向函数中使用了 `BuildContext.dependOnInheritedWidgetOfExactType`（通常通过 `of` 方法实现），当 `InheritedWidget` 发生变化时会触发重新评估

**使用场景**：

1. **默认路由重定向**：将根路径重定向到默认页面
2. **认证检查**：未登录用户访问受保护页面时重定向到登录页
3. **条件导航**：根据应用状态决定导航目标

### routes

```dart 223:224:lib/src/route.dart
  /// The list of child routes associated with this route.
  final List<RouteBase> routes;
```

**功能说明**：

- **子路由列表**：定义当前路由的所有子路由
- **树形结构**：通过嵌套的路由列表构建路由树
- **类型灵活**：子路由可以是 `GoRoute` 或 `ShellRoute` 等任何 `RouteBase` 的子类

### parentNavigatorKey

```dart 226:231:lib/src/route.dart
  /// An optional key specifying which Navigator to display this route's screen
  /// onto.
  ///
  /// Specifying the root Navigator will stack this route onto that
  /// Navigator instead of the nearest ShellRoute ancestor.
  final GlobalKey<NavigatorState>? parentNavigatorKey;
```

**功能说明**：

- **导航器选择**：指定路由应该显示在哪个 `Navigator` 上
- **默认行为**：如果不指定，路由会显示在最近的 `ShellRoute` 祖先的 `Navigator` 上
- **根导航器**：可以指定根 `Navigator`，使路由显示在根导航器上而不是 `ShellRoute` 的导航器上

**使用场景**：

- 在 `ShellRoute` 中，某些子路由需要显示在根导航器上（例如全屏对话框）
- 实现多层级导航结构

## 静态方法

### routesRecursively

```dart 233:239:lib/src/route.dart
  /// Builds a lists containing the provided routes along with all their
  /// descendant [routes].
  static Iterable<RouteBase> routesRecursively(Iterable<RouteBase> routes) {
    return routes.expand(
      (RouteBase e) => <RouteBase>[e, ...routesRecursively(e.routes)],
    );
  }
```

**功能说明**：

- **递归收集**：递归地收集所有提供的路由及其所有后代路由
- **扁平化处理**：将树形路由结构扁平化为一个可迭代的列表
- **用途**：用于需要遍历所有路由的场景，例如路由匹配、路由验证等

**工作原理**：

1. 对每个路由，先添加路由本身
2. 然后递归处理该路由的所有子路由
3. 使用 `expand` 方法将嵌套结构展开为扁平列表

## 调试支持

### debugFillProperties

```dart 241:252:lib/src/route.dart
  @override
  void debugFillProperties(DiagnosticPropertiesBuilder properties) {
    super.debugFillProperties(properties);
    if (parentNavigatorKey != null) {
      properties.add(
        DiagnosticsProperty<GlobalKey<NavigatorState>>(
          'parentNavKey',
          parentNavigatorKey,
        ),
      );
    }
  }
```

**功能说明**：

- **调试信息**：为 Flutter 的调试工具提供路由的调试信息
- **条件添加**：只有当 `parentNavigatorKey` 不为空时才添加该属性
- **诊断支持**：继承自 `Diagnosticable`，支持 Flutter 的诊断工具链

## 设计模式

### 抽象基类模式

`RouteBase` 使用了抽象基类模式：

- **统一接口**：为所有路由类型提供统一的接口
- **代码复用**：共享的属性和方法在基类中定义
- **多态支持**：允许以统一的方式处理不同类型的路由

### 不可变设计

类被标记为 `@immutable`，意味着：

- **线程安全**：不可变对象在多线程环境下是安全的
- **值语义**：路由配置可以作为值传递和比较
- **性能优化**：不可变对象可以进行更好的优化

## 使用示例

### 基本路由树

```dart
final GoRouter router = GoRouter(
  routes: <GoRoute>[
    GoRoute(
      path: '/',
      builder: (context, state) => HomePage(),
      routes: <GoRoute>[
        GoRoute(
          path: 'profile',
          builder: (context, state) => ProfilePage(),
          routes: <GoRoute>[
            GoRoute(
              path: 'settings',
              builder: (context, state) => SettingsPage(),
            ),
          ],
        ),
      ],
    ),
  ],
);
```

### 带重定向的路由

```dart
final GoRouter router = GoRouter(
  routes: <GoRoute>[
    GoRoute(
      path: '/',
      redirect: (context, state) => '/home',
    ),
    GoRoute(
      path: '/home',
      builder: (context, state) => HomePage(),
    ),
  ],
);
```

### 路由守卫示例

```dart
final GoRouter router = GoRouter(
  routes: <GoRoute>[
    GoRoute(
      path: '/login',
      builder: (context, state) => LoginPage(),
    ),
    GoRoute(
      path: '/dashboard',
      redirect: (context, state) {
        final isAuthenticated = AuthService.of(context).isAuthenticated;
        return isAuthenticated ? null : '/login';
      },
      builder: (context, state) => DashboardPage(),
    ),
  ],
);
```

## 总结

`RouteBase` 是 `go_router` 路由系统的核心抽象类，它提供了：

1. **树形路由结构**：支持嵌套路由和层级导航
2. **重定向机制**：灵活的路由重定向和路由守卫
3. **导航器管理**：支持多层级导航器结构
4. **类型安全**：通过抽象类确保类型一致性
5. **调试支持**：完整的调试信息支持

通过继承 `RouteBase`，`GoRoute` 和 `ShellRoute` 可以共享这些核心功能，同时实现各自特定的行为。
