# 第 8 章 与路由系统集成

## 引言

在实际应用中，自适应布局通常需要与路由系统配合使用。`go_router_demo` 示例展示了如何将 `AdaptiveScaffold` 与 GoRouter 集成，实现完整的导航和路由管理。本章将深入分析这个集成方案，包括路由配置、状态管理、认证流程等。

## GoRouter 基础

### 为什么选择 GoRouter

GoRouter 是 Flutter 推荐的声明式路由解决方案，具有以下优势：

1. **声明式路由**：使用声明式 API 定义路由
2. **深度链接支持**：自动处理 URL 和深度链接
3. **状态管理**：与状态管理方案良好集成
4. **嵌套路由**：支持复杂的路由结构

### 基本结构

```dart
final GoRouter router = GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => HomePage(),
    ),
    GoRoute(
      path: '/details',
      builder: (context, state) => DetailsPage(),
    ),
  ],
);
```

## ScaffoldShell 实现

### 核心设计

`ScaffoldShell` 是连接 `AdaptiveScaffold` 和 GoRouter 的桥梁：

```dart
class ScaffoldShell extends StatelessWidget {
  const ScaffoldShell({
    required this.navigationShell,
    super.key,
  });

  final StatefulNavigationShell navigationShell;

  @override
  Widget build(BuildContext context) {
    return AdaptiveScaffold(
      useDrawer: false,
      body: (BuildContext context) => navigationShell,
      selectedIndex: navigationShell.currentIndex,
      onSelectedIndexChange: (int index) {
        navigationShell.goBranch(
          index,
          initialLocation: index == navigationShell.currentIndex,
        );
      },
      destinations: navigationShell.route.branches.map(
        (StatefulShellBranch e) {
          return switch (e.defaultRoute?.name) {
            HomePage.name => const NavigationDestination(
                icon: Icon(Icons.home), label: 'Home'),
            CounterPage.name => const NavigationDestination(
                icon: Icon(Icons.add), label: 'Counter'),
            MorePage.name => const NavigationDestination(
                icon: Icon(Icons.account_circle), label: 'More'),
            _ => throw UnimplementedError(
                'The route ${e.defaultRoute?.name} is not implemented.',
              ),
          };
        },
      ).toList(),
    );
  }
}
```

### 关键点解析

#### 1. StatefulNavigationShell

`StatefulNavigationShell` 是 GoRouter 提供的组件，用于管理分支导航：

- **currentIndex**：当前选中的分支索引
- **goBranch**：切换到指定分支
- **route.branches**：所有分支路由

#### 2. 状态同步

`AdaptiveScaffold` 的 `selectedIndex` 与 `navigationShell.currentIndex` 同步：

```dart
selectedIndex: navigationShell.currentIndex,
onSelectedIndexChange: (int index) {
  navigationShell.goBranch(index, ...);
}
```

#### 3. Destinations 映射

从 `navigationShell.route.branches` 生成 `NavigationDestination` 列表：

```dart
destinations: navigationShell.route.branches.map((branch) {
  return switch (branch.defaultRoute?.name) {
    HomePage.name => NavigationDestination(...),
    CounterPage.name => NavigationDestination(...),
    // ...
  };
}).toList(),
```

## 路由配置

### StatefulShellRoute

使用 `StatefulShellRoute.indexedStack` 创建分支路由：

```dart
static final StatefulShellRoute _authenticatedRoutes =
    StatefulShellRoute.indexedStack(
  parentNavigatorKey: rootNavigatorKey,
  builder: (
    BuildContext context,
    GoRouterState state,
    StatefulNavigationShell navigationShell,
  ) {
    return ScaffoldShell(navigationShell: navigationShell);
  },
  redirect: (BuildContext context, GoRouterState state) {
    if (!authenticatedNotifier.value) {
      return LoginPage.path;
    }
    return null;
  },
  branches: <StatefulShellBranch>[
    // 分支定义
  ],
);
```

### 分支定义

每个分支对应一个导航项：

```dart
branches: <StatefulShellBranch>[
  // Home 分支
  StatefulShellBranch(
    navigatorKey: _homeNavigatorKey,
    routes: <RouteBase>[
      GoRoute(
        name: HomePage.name,
        path: HomePage.path,
        pageBuilder: (context, state) {
          return const NoTransitionPage<void>(
            child: HomePage(),
          );
        },
        routes: <RouteBase>[
          // 子路由
        ],
      ),
    ],
  ),
  
  // Counter 分支
  StatefulShellBranch(
    navigatorKey: _counterNavigatorKey,
    routes: <RouteBase>[
      GoRoute(
        name: CounterPage.name,
        path: CounterPage.path,
        pageBuilder: (context, state) {
          return const NoTransitionPage<void>(
            child: CounterPage(),
          );
        },
      ),
    ],
  ),
  
  // More 分支
  StatefulShellBranch(
    navigatorKey: _moreNavigatorKey,
    routes: <RouteBase>[
      GoRoute(
        name: MorePage.name,
        path: MorePage.path,
        pageBuilder: (context, state) {
          return const NoTransitionPage<void>(
            child: MorePage(),
          );
        },
        routes: <RouteBase>[
          // 子路由
        ],
      ),
    ],
  ),
],
```

### NavigatorKey 的作用

每个分支使用独立的 `NavigatorKey`：

```dart
final GlobalKey<NavigatorState> _homeNavigatorKey =
    GlobalKey<NavigatorState>(debugLabel: 'home');
final GlobalKey<NavigatorState> _counterNavigatorKey =
    GlobalKey<NavigatorState>(debugLabel: 'counter');
final GlobalKey<NavigatorState> _moreNavigatorKey =
    GlobalKey<NavigatorState>(debugLabel: 'more');
```

**作用**：

- 每个分支维护独立的导航栈
- 切换分支时保持各自的状态
- 支持深度链接到特定分支

## 认证流程处理

### 认证状态管理

使用 `ValueNotifier` 管理认证状态：

```dart
class AppRouter {
  static ValueNotifier<bool> authenticatedNotifier =
      ValueNotifier<bool>(false);
  
  static final GoRouter router = GoRouter(
    refreshListenable: authenticatedNotifier,  // 监听认证状态变化
    routes: [
      _unauthenticatedRoutes,
      _authenticatedRoutes,
      ..._openRoutes,
    ],
  );
}
```

### 未认证路由

未认证用户只能访问登录相关页面：

```dart
static final GoRoute _unauthenticatedRoutes = GoRoute(
  name: LoginPage.name,
  path: LoginPage.path,
  pageBuilder: (context, state) {
    return const MaterialPage<void>(child: LoginPage());
  },
  redirect: (context, state) {
    if (authenticatedNotifier.value) {
      return HomePage.path;  // 已登录，重定向到首页
    }
    return null;
  },
  routes: <RouteBase>[
    GoRoute(
      name: ForgotPasswordPage.name,
      path: ForgotPasswordPage.path,
      pageBuilder: (context, state) {
        return const MaterialPage<void>(
          child: ForgotPasswordPage(),
        );
      },
    ),
  ],
);
```

### 认证路由重定向

认证路由检查登录状态：

```dart
static final StatefulShellRoute _authenticatedRoutes =
    StatefulShellRoute.indexedStack(
  redirect: (context, state) {
    if (!authenticatedNotifier.value) {
      return LoginPage.path;  // 未登录，重定向到登录页
    }
    return null;
  },
  // ...
);
```

### 登录流程

登录成功后更新认证状态：

```dart
// 在 LoginPage 中
void _handleLogin() {
  // 执行登录逻辑
  AppRouter.authenticatedNotifier.value = true;
  // GoRouter 会自动重定向到 HomePage
}
```

## 子路由处理

### 嵌套路由

每个分支可以有自己的子路由：

```dart
GoRoute(
  name: HomePage.name,
  path: HomePage.path,
  pageBuilder: (context, state) {
    return const NoTransitionPage<void>(child: HomePage());
  },
  routes: <RouteBase>[
    GoRoute(
      name: DetailOverviewPage.name,
      path: DetailOverviewPage.path,
      pageBuilder: (context, state) {
        return const MaterialPage<void>(
          child: DetailOverviewPage(),
        );
      },
      routes: <RouteBase>[
        GoRoute(
          name: DetailPage.name,
          path: DetailPage.path,
          pageBuilder: (context, state) {
            return MaterialPage<void>(
              child: DetailPage(
                itemName: state.uri.queryParameters['itemName']!,
              ),
            );
          },
        ),
      ],
    ),
  ],
)
```

### 模态对话框路由

使用 `parentNavigatorKey` 在根导航器上显示模态：

```dart
GoRoute(
  name: DetailModalPage.name,
  path: DetailModalPage.path,
  parentNavigatorKey: rootNavigatorKey,  // 使用根导航器
  pageBuilder: (context, state) {
    return const MaterialPage<void>(
      fullscreenDialog: true,  // 全屏对话框
      child: DetailModalPage(),
    );
  },
)
```

## 路由状态与导航状态同步

### 问题

`AdaptiveScaffold` 的导航状态和 GoRouter 的路由状态需要保持同步：

- 用户点击导航项 → 更新 `selectedIndex` → 切换路由
- 用户通过其他方式导航 → 更新路由 → 更新 `selectedIndex`

### 解决方案

在 `ScaffoldShell` 中实现双向同步：

```dart
class ScaffoldShell extends StatelessWidget {
  const ScaffoldShell({
    required this.navigationShell,
    super.key,
  });

  final StatefulNavigationShell navigationShell;

  @override
  Widget build(BuildContext context) {
    return AdaptiveScaffold(
      selectedIndex: navigationShell.currentIndex,  // 从路由状态读取
      onSelectedIndexChange: (int index) {
        navigationShell.goBranch(  // 更新路由状态
          index,
          initialLocation: index == navigationShell.currentIndex,
        );
      },
      // ...
    );
  }
}
```

## 错误处理

### 错误页面

定义错误页面处理未匹配的路由：

```dart
static final GoRouter router = GoRouter(
  errorPageBuilder: (context, state) {
    return const MaterialPage<void>(
      child: NavigationErrorPage(),
    );
  },
  // ...
);
```

### 重定向处理

使用 `redirect` 处理路由重定向：

```dart
static final GoRouter router = GoRouter(
  redirect: (context, state) {
    if (state.uri.path == '/') {
      return HomePage.path;  // 根路径重定向到首页
    }
    return null;
  },
  // ...
);
```

## 完整示例分析

让我们完整分析 `app_router.dart` 的实现：

```dart
class AppRouter {
  // 认证状态
  static ValueNotifier<bool> authenticatedNotifier = ValueNotifier<bool>(false);

  // 导航器 Keys
  final GlobalKey<NavigatorState> rootNavigatorKey =
      GlobalKey<NavigatorState>(debugLabel: 'root');
  final GlobalKey<NavigatorState> _homeNavigatorKey =
      GlobalKey<NavigatorState>(debugLabel: 'home');
  final GlobalKey<NavigatorState> _counterNavigatorKey =
      GlobalKey<NavigatorState>(debugLabel: 'counter');
  final GlobalKey<NavigatorState> _moreNavigatorKey =
      GlobalKey<NavigatorState>(debugLabel: 'more');

  // 路由器配置
  static final GoRouter router = GoRouter(
    navigatorKey: rootNavigatorKey,
    debugLogDiagnostics: true,
    errorPageBuilder: (context, state) {
      return const MaterialPage<void>(child: NavigationErrorPage());
    },
    redirect: (context, state) {
      if (state.uri.path == '/') {
        return HomePage.path;
      }
      return null;
    },
    refreshListenable: authenticatedNotifier,
    routes: <RouteBase>[
      _unauthenticatedRoutes,  // 未认证路由
      _authenticatedRoutes,    // 认证路由
      ..._openRoutes,         // 开放路由
    ],
  );

  // 未认证路由
  static final GoRoute _unauthenticatedRoutes = GoRoute(
    name: LoginPage.name,
    path: LoginPage.path,
    pageBuilder: (context, state) {
      return const MaterialPage<void>(child: LoginPage());
    },
    redirect: (context, state) {
      if (authenticatedNotifier.value) {
        return HomePage.path;
      }
      return null;
    },
    routes: <RouteBase>[
      GoRoute(
        name: ForgotPasswordPage.name,
        path: ForgotPasswordPage.path,
        pageBuilder: (context, state) {
          return const MaterialPage<void>(child: ForgotPasswordPage());
        },
      ),
    ],
  );

  // 认证路由（使用 StatefulShellRoute）
  static final StatefulShellRoute _authenticatedRoutes =
      StatefulShellRoute.indexedStack(
    parentNavigatorKey: rootNavigatorKey,
    builder: (context, state, navigationShell) {
      return ScaffoldShell(navigationShell: navigationShell);
    },
    redirect: (context, state) {
      if (!authenticatedNotifier.value) {
        return LoginPage.path;
      }
      return null;
    },
    branches: <StatefulShellBranch>[
      // Home 分支
      StatefulShellBranch(
        navigatorKey: _homeNavigatorKey,
        routes: <RouteBase>[
          GoRoute(
            name: HomePage.name,
            path: HomePage.path,
            pageBuilder: (context, state) {
              return const NoTransitionPage<void>(child: HomePage());
            },
            routes: <RouteBase>[
              GoRoute(
                name: DetailOverviewPage.name,
                path: DetailOverviewPage.path,
                pageBuilder: (context, state) {
                  return const MaterialPage<void>(
                    child: DetailOverviewPage(),
                  );
                },
                routes: <RouteBase>[
                  GoRoute(
                    name: DetailPage.name,
                    path: DetailPage.path,
                    pageBuilder: (context, state) {
                      return MaterialPage<void>(
                        child: DetailPage(
                          itemName: state.uri.queryParameters['itemName']!,
                        ),
                      );
                    },
                  ),
                ],
              ),
              GoRoute(
                name: DetailModalPage.name,
                path: DetailModalPage.path,
                parentNavigatorKey: rootNavigatorKey,
                pageBuilder: (context, state) {
                  return const MaterialPage<void>(
                    fullscreenDialog: true,
                    child: DetailModalPage(),
                  );
                },
              ),
            ],
          ),
        ],
      ),
      // Counter 和 More 分支类似
    ],
  );

  // 开放路由（不需要认证）
  static final List<GoRoute> _openRoutes = <GoRoute>[
    GoRoute(
      name: LanguagePage.name,
      path: LanguagePage.path,
      pageBuilder: (context, state) {
        return const MaterialPage<void>(child: LanguagePage());
      },
    ),
  ];
}
```

## 最佳实践

### 1. 使用命名路由

使用命名路由提高可维护性：

```dart
class HomePage {
  static const String name = 'home';
  static const String path = '/home';
}
```

### 2. 分离路由配置

将路由配置分离到独立文件，保持代码清晰。

### 3. 使用 NoTransitionPage

对于同一分支内的页面切换，使用 `NoTransitionPage` 避免不必要的动画：

```dart
pageBuilder: (context, state) {
  return const NoTransitionPage<void>(child: HomePage());
}
```

### 4. 处理深度链接

GoRouter 自动处理深度链接，确保路由配置正确：

```dart
GoRoute(
  path: '/details/:id',
  builder: (context, state) {
    final id = state.pathParameters['id'];
    return DetailsPage(id: id);
  },
)
```

## 总结

本章我们深入学习了：

- **GoRouter 基础**：为什么选择 GoRouter 和基本结构
- **ScaffoldShell 实现**：连接 AdaptiveScaffold 和 GoRouter 的桥梁
- **路由配置**：StatefulShellRoute 和分支定义
- **认证流程**：认证状态管理和路由重定向
- **子路由处理**：嵌套路由和模态对话框
- **状态同步**：路由状态与导航状态的同步
- **错误处理**：错误页面和重定向
- **最佳实践**：命名路由、代码组织等

在下一章中，我们将分析完整的邮件应用示例，理解复杂应用的设计。

## 练习

1. 创建一个新的分支路由，添加新的导航项
2. 实现认证流程，包括登录和登出
3. 添加深度链接支持，处理 URL 参数

## 检查清单

- [ ] 理解 GoRouter 与 AdaptiveScaffold 的集成方式
- [ ] 掌握 ScaffoldShell 的实现
- [ ] 了解 StatefulShellRoute 的配置
- [ ] 能够实现认证流程
- [ ] 理解路由状态同步机制
- [ ] 了解错误处理和最佳实践
