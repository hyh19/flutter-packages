# StatefulShellRoute 状态恢复配置示例解析

## 概述

这个示例展示了如何在 `go_router` 中为 `StatefulShellRoute` 配置状态恢复功能。`StatefulShellRoute` 是 `ShellRoute` 的增强版本，它能够保持每个分支（branch）的独立状态，非常适合实现底部导航栏、侧边栏导航等多标签页应用。

## StatefulShellRoute 的核心特性

与 `ShellRoute` 相比，`StatefulShellRoute` 具有以下特点：

- **状态保持**：每个分支（branch）的状态在切换时完全保持，不会重建
- **独立导航栈**：每个分支可以有自己的导航栈
- **索引管理**：通过 `currentIndex` 管理当前激活的分支
- **状态恢复**：可以为每个分支设置独立的恢复作用域

## 代码结构分析

### 应用入口

```dart 8:8:example/lib/state_restoration/stateful_shell_route_state_restoration.dart
void main() => runApp(const App());
```

### App 组件配置

```dart 12:69:example/lib/state_restoration/stateful_shell_route_state_restoration.dart
class App extends StatefulWidget {
  /// Creates an [App].
  const App({super.key});

  @override
  State<App> createState() => _AppState();
}

class _AppState extends State<App> {
  final GoRouter _router = GoRouter(
    restorationScopeId: 'router',
    routes: <RouteBase>[
      StatefulShellRoute.indexedStack(
        restorationScopeId: 'appShell',
        pageBuilder:
            (
              BuildContext context,
              GoRouterState state,
              StatefulNavigationShell navigationShell,
            ) {
              return MaterialPage<void>(
                restorationId: 'appShellPage',
                child: AppShell(navigationShell: navigationShell),
              );
            },
        branches: <StatefulShellBranch>[
          StatefulShellBranch(
            restorationScopeId: 'homeBranch',
            routes: <GoRoute>[
              GoRoute(
                path: '/',
                builder: (BuildContext context, GoRouterState state) {
                  return const HomeBody();
                },
              ),
            ],
          ),
          StatefulShellBranch(
            restorationScopeId: 'profileBranch',
            routes: <GoRoute>[
              GoRoute(
                path: '/profile',
                builder: (BuildContext context, GoRouterState state) {
                  return const ProfileBody();
                },
              ),
            ],
          ),
        ],
      ),
    ],
  );

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(restorationScopeId: 'app', routerConfig: _router);
  }
}
```

#### 关键配置点

1. **StatefulShellRoute.indexedStack**：

   ```dart 24:36:example/lib/state_restoration/stateful_shell_route_state_restoration.dart
   StatefulShellRoute.indexedStack(
     restorationScopeId: 'appShell',
     pageBuilder:
         (
           BuildContext context,
           GoRouterState state,
           StatefulNavigationShell navigationShell,
         ) {
           return MaterialPage<void>(
             restorationId: 'appShellPage',
             child: AppShell(navigationShell: navigationShell),
           );
         },
   ```

   - `indexedStack` 表示使用索引栈布局，适合底部导航栏场景
   - `restorationScopeId: 'appShell'` 为整个 Shell 创建恢复作用域
   - `pageBuilder` 接收 `StatefulNavigationShell` 参数，这是管理分支导航的核心对象
   - `restorationId: 'appShellPage'` 为 Shell 页面指定恢复标识

2. **StatefulShellBranch 配置**：

   ```dart 37:59:example/lib/state_restoration/stateful_shell_route_state_restoration.dart
   branches: <StatefulShellBranch>[
     StatefulShellBranch(
       restorationScopeId: 'homeBranch',
       routes: <GoRoute>[
         GoRoute(
           path: '/',
           builder: (BuildContext context, GoRouterState state) {
             return const HomeBody();
           },
         ),
       ],
     ),
     StatefulShellBranch(
       restorationScopeId: 'profileBranch',
       routes: <GoRoute>[
         GoRoute(
           path: '/profile',
           builder: (BuildContext context, GoRouterState state) {
             return const ProfileBody();
           },
         ),
       ],
     ),
   ],
   ```

   - 每个分支都有独立的 `restorationScopeId`，确保状态恢复的隔离性
   - `homeBranch` 对应首页分支，路径为 `/`
   - `profileBranch` 对应个人资料分支，路径为 `/profile`
   - 每个分支可以包含多个路由，形成独立的导航栈

### AppShell 组件

```dart 71:100:example/lib/state_restoration/stateful_shell_route_state_restoration.dart
/// The shell of the app.
class AppShell extends StatelessWidget {
  /// Creates an [AppShell].
  const AppShell({required this.navigationShell, super.key});

  /// The [StatefulNavigationShell] displayed in the body
  /// of the [Scaffold].
  final StatefulNavigationShell navigationShell;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('App')),
      body: navigationShell,
      bottomNavigationBar: NavigationBar(
        selectedIndex: navigationShell.currentIndex,
        onDestinationSelected: (int index) {
          navigationShell.goBranch(index);
        },
        destinations: const <NavigationDestination>[
          NavigationDestination(icon: Icon(Icons.home), label: 'Home'),
          NavigationDestination(
            icon: Icon(Icons.account_circle),
            label: 'Profile',
          ),
        ],
      ),
    );
  }
}
```

**核心功能解析**：

1. **StatefulNavigationShell**：
   - 这是管理分支导航的核心对象
   - 通过 `navigationShell` 可以访问当前激活的分支索引和切换分支

2. **底部导航栏配置**：

   ```dart 85:89:example/lib/state_restoration/stateful_shell_route_state_restoration.dart
   bottomNavigationBar: NavigationBar(
     selectedIndex: navigationShell.currentIndex,
     onDestinationSelected: (int index) {
       navigationShell.goBranch(index);
     },
   ```

   - `selectedIndex` 绑定到 `navigationShell.currentIndex`，自动反映当前激活的分支
   - `onDestinationSelected` 调用 `navigationShell.goBranch(index)` 切换分支
   - 切换分支时，之前分支的状态会完全保持

3. **Body 渲染**：
   - `body: navigationShell` 直接渲染导航 Shell
   - Shell 会根据 `currentIndex` 显示对应分支的内容

### HomeBody 组件

```dart 102:118:example/lib/state_restoration/stateful_shell_route_state_restoration.dart
/// The home body of the app.
class HomeBody extends StatelessWidget {
  /// Creates a [HomeBody].
  const HomeBody({super.key});

  @override
  Widget build(BuildContext context) {
    return const Column(
      children: <Widget>[
        TextField(
          restorationId: 'homeTextField',
          decoration: InputDecoration(labelText: 'Home'),
        ),
      ],
    );
  }
}
```

- 首页内容，包含一个可恢复的文本输入框
- `restorationId: 'homeTextField'` 在 `homeBranch` 作用域下恢复

### ProfileBody 组件

```dart 120:136:example/lib/state_restoration/stateful_shell_route_state_restoration.dart
/// The profile body of the app.
class ProfileBody extends StatelessWidget {
  /// Creates a [ProfileBody].
  const ProfileBody({super.key});

  @override
  Widget build(BuildContext context) {
    return const Column(
      children: <Widget>[
        TextField(
          restorationId: 'profileTextField',
          decoration: InputDecoration(labelText: 'Profile'),
        ),
      ],
    );
  }
}
```

- 个人资料页面内容，也包含一个可恢复的文本输入框
- `restorationId: 'profileTextField'` 在 `profileBranch` 作用域下恢复

## 状态恢复层级结构

```text
app (MaterialApp.router)
  └── router (GoRouter)
      └── appShell (StatefulShellRoute)
          └── appShellPage (MaterialPage)
              ├── homeBranch (StatefulShellBranch)
              │   └── / (自动设置 restorationId)
              │       └── homeTextField (TextField)
              └── profileBranch (StatefulShellBranch)
                  └── /profile (自动设置 restorationId)
                      └── profileTextField (TextField)
```

## StatefulShellRoute 的优势

1. **完整状态保持**：
   - 切换分支时，之前分支的 Widget 树完全保持
   - 滚动位置、输入内容、动画状态等都会保留

2. **独立导航栈**：
   - 每个分支可以有自己的导航栈
   - 在分支内使用 `context.push()` 导航，不会影响其他分支

3. **性能优化**：
   - 分支切换时不会重建 Widget，性能更好
   - 适合需要频繁切换的场景

4. **状态恢复隔离**：
   - 每个分支有独立的作用域，状态恢复互不干扰

## 使用场景

这个配置适用于以下场景：

1. **底部导航栏应用**：最常见的多标签页应用
2. **侧边栏导航**：桌面端或平板端的侧边栏导航
3. **标签页切换**：需要保持每个标签页状态的场景
4. **主从视图**：主视图切换，从视图保持的场景

## 分支切换机制

### goBranch 方法

```dart 87:89:example/lib/state_restoration/stateful_shell_route_state_restoration.dart
onDestinationSelected: (int index) {
  navigationShell.goBranch(index);
},
```

`goBranch(index)` 方法的作用：

- 切换到指定索引的分支
- 保持之前分支的完整状态
- 更新 `currentIndex`
- 触发 UI 更新（底部导航栏高亮）

### 状态保持原理

`indexedStack` 使用 `IndexedStack` Widget 实现：

- `IndexedStack` 会同时保持所有子 Widget 的状态
- 只显示 `currentIndex` 对应的 Widget
- 其他 Widget 虽然不可见，但状态完全保持

## 与其他路由类型的对比

| 特性 | GoRoute | ShellRoute | StatefulShellRoute |
| ------ | --------- | ------------ | ------------------- |
| 状态保持 | 否 | 部分（Shell 容器） | 是（完整分支） |
| 分支切换 | 不适用 | 重建子路由 | 保持子路由 |
| 适用场景 | 独立页面 | 相关页面组 | 多标签页应用 |
| 导航栈 | 单一 | 单一 | 每个分支独立 |

## 注意事项

1. **分支索引**：
   - 分支索引从 0 开始
   - 必须与 `NavigationBar` 的 `destinations` 顺序一致

2. **路径配置**：
   - 每个分支的第一个路由路径会作为分支的标识
   - 使用 `navigationShell.goBranch(index)` 切换时，会自动导航到对应分支的第一个路由

3. **状态恢复作用域**：
   - Shell 和每个分支都有独立的作用域
   - 合理设置作用域有助于组织恢复数据

4. **性能考虑**：
   - `indexedStack` 会同时保持所有分支的状态
   - 如果分支数量很多或内容很重，可能影响内存使用
   - 考虑使用 `StatefulShellRoute` 的其他变体（如 `scaffold`）来优化

5. **导航行为**：
   - 在分支内使用 `context.push()` 进行导航
   - 使用 `context.go('/')` 会导航到根路由，可能退出 Shell

## 扩展场景

### 在分支内添加更多路由

每个分支可以包含多个路由：

```dart
StatefulShellBranch(
  restorationScopeId: 'homeBranch',
  routes: <GoRoute>[
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeBody(),
    ),
    GoRoute(
      path: '/details',
      builder: (context, state) => const DetailsPage(),
    ),
  ],
),
```

这样在首页分支内可以使用 `context.push('/details')` 导航到详情页，而不会影响其他分支。

### 动态分支切换

除了通过底部导航栏，还可以通过代码动态切换：

```dart
// 在某个事件处理中
navigationShell.goBranch(1); // 切换到第二个分支
```

这在响应特定业务逻辑时很有用。
