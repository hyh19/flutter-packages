# DetailOverviewPage 自适应布局实现方案调研

## 需求概述

实现 `DetailOverviewPage` 的自适应布局：

- **small/medium 尺寸**：只显示列表（body）
- **mediumLarge 及以上尺寸**：body 显示列表，secondaryBody 显示 `DetailPage`
- **默认行为**：首次进入且未选择 item 时，自动显示第一个 item 的详情

## 实现方案

### 方案1：使用 AdaptiveLayout（最灵活，推荐）

**优点**：

- 最大的灵活性和控制能力
- 可以精确控制每个 breakpoint 的行为
- 可以自定义动画效果

**缺点**：

- 代码量较多
- 需要手动配置 SlotLayout 和 Breakpoint 映射
- 学习曲线较陡

**实现要点**：

- 在 `DetailOverviewPage` 中使用 `AdaptiveLayout` 替代 `Scaffold`
- 使用 `StatefulWidget` 管理选中的 item 索引
- `body` slot：small/medium 显示列表，mediumLarge+ 也显示列表
- `secondaryBody` slot：只在 mediumLarge+ 显示，根据选中索引渲染 `DetailPage`
- 使用 `Breakpoints.mediumLarge` 作为 secondaryBody 的激活条件

**关键文件修改**：

- `example/lib/go_router_demo/pages/detail_overview_page.dart`：重写为使用 `AdaptiveLayout`

**参考示例**：`example/lib/main.dart` 中的 `_AdaptiveLayoutExample`（第 384-398 行）

**完整代码实现**：

```dart
// example/lib/go_router_demo/pages/detail_overview_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_adaptive_scaffold/flutter_adaptive_scaffold.dart';
import 'package:go_router/go_router.dart';

import 'detail_page.dart';

/// The detail overview page.
class DetailOverviewPage extends StatefulWidget {
  /// Construct the detail overview page.
  const DetailOverviewPage({super.key});

  /// The path for the detail page.
  static const String path = 'detail-overview';

  /// The name for the detail page.
  static const String name = 'DetailOverview';

  @override
  State<DetailOverviewPage> createState() => _DetailOverviewPageState();
}

class _DetailOverviewPageState extends State<DetailOverviewPage> {
  int _selectedIndex = 0; // 默认选中第一个 item
  static const int _itemCount = 10;

  Widget _buildItemList(BuildContext context) {
    final bool isLargeScreen = Breakpoints.mediumLarge.isActive(context);
    
    return ListView.builder(
      itemCount: _itemCount,
      itemBuilder: (BuildContext context, int index) {
        return ListTile(
          title: Text('Item $index'),
          selected: isLargeScreen && index == _selectedIndex,
          onTap: () {
            if (isLargeScreen) {
              // 大屏幕：更新选中状态
              setState(() {
                _selectedIndex = index;
              });
            } else {
              // 小屏幕：导航到 DetailPage
              context.goNamed(
                DetailPage.name,
                queryParameters: <String, String>{'itemName': '$index'},
              );
            }
          },
        );
      },
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Detail Overview Page'),
      ),
      body: AdaptiveLayout(
        body: SlotLayout(
          config: <Breakpoint, SlotLayoutConfig?>{
            Breakpoints.standard: SlotLayout.from(
              key: const Key('body'),
              builder: (_) => _buildItemList(context),
            ),
          },
        ),
        secondaryBody: SlotLayout(
          config: <Breakpoint, SlotLayoutConfig?>{
            Breakpoints.mediumLarge: SlotLayout.from(
              key: const Key('secondaryBody'),
              builder: (_) => DetailPage(itemName: '$_selectedIndex'),
            ),
          },
        ),
      ),
    );
  }
}
```

**关键代码说明**：

1. **状态管理**：使用 `StatefulWidget` 和 `_selectedIndex` 管理选中项
2. **列表构建**：`_buildItemList` 方法根据屏幕尺寸决定点击行为
3. **Breakpoint 检测**：使用 `Breakpoints.mediumLarge.isActive(context)` 判断是否为大屏幕
4. **SlotLayout 配置**：
   - `body`：使用 `Breakpoints.standard` 确保在所有尺寸都显示
   - `secondaryBody`：只在 `Breakpoints.mediumLarge` 及以上显示
5. **选中状态可视化**：使用 `ListTile.selected` 高亮选中的项

### 方案2：使用 AdaptiveScaffold（不推荐，仅作参考）

**注意**：`AdaptiveScaffold` 主要用于有导航栏的场景，需要至少 2 个 destinations。对于 `DetailOverviewPage` 这种单个页面场景，**实际推荐使用方案1（AdaptiveLayout）**。此方案仅作为参考展示。

**优点**：

- API 更简洁，代码量少
- 自动处理导航栏和布局
- 内置了常用的 breakpoint 配置
- 与现有 `ScaffoldShell` 的设计风格一致

**缺点**：

- **需要 destinations**：`AdaptiveScaffold` 要求至少 2 个 destinations，不适合单个页面场景
- 灵活性略低于 AdaptiveLayout
- 需要移除原有的 AppBar（因为 AdaptiveScaffold 内部处理）

**实现要点**：

- 在 `DetailOverviewPage` 中使用 `AdaptiveScaffold` 替代 `Scaffold`
- 使用 `StatefulWidget` 管理选中的 item 索引
- `smallBody` 和 `body`：显示列表（覆盖 small/medium）
- `secondaryBody`：在 mediumLarge+ 显示 `DetailPage`（使用 `mediumLargeSecondaryBody` 属性）
- `smallSecondaryBody`：设置为 `AdaptiveScaffold.emptyBuilder` 以在小屏幕隐藏

**关键文件修改**：

- `example/lib/go_router_demo/pages/detail_overview_page.dart`：重写为使用 `AdaptiveScaffold`（不推荐，实际应使用方案1）

**参考示例**：`example/lib/adaptive_scaffold_demo.dart`（第 142-152 行）

**完整代码实现**：

```dart
// example/lib/go_router_demo/pages/detail_overview_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_adaptive_scaffold/flutter_adaptive_scaffold.dart';
import 'package:go_router/go_router.dart';

import 'detail_page.dart';

/// The detail overview page.
class DetailOverviewPage extends StatefulWidget {
  /// Construct the detail overview page.
  const DetailOverviewPage({super.key});

  /// The path for the detail page.
  static const String path = 'detail-overview';

  /// The name for the detail page.
  static const String name = 'DetailOverview';

  @override
  State<DetailOverviewPage> createState() => _DetailOverviewPageState();
}

class _DetailOverviewPageState extends State<DetailOverviewPage> {
  int _selectedIndex = 0; // 默认选中第一个 item
  static const int _itemCount = 10;

  Widget _buildItemList(BuildContext context) {
    final bool isLargeScreen = Breakpoints.mediumLarge.isActive(context);
    
    return ListView.builder(
      itemCount: _itemCount,
      itemBuilder: (BuildContext context, int index) {
        return ListTile(
          title: Text('Item $index'),
          selected: isLargeScreen && index == _selectedIndex,
          onTap: () {
            if (isLargeScreen) {
              // 大屏幕：更新选中状态
              setState(() {
                _selectedIndex = index;
              });
            } else {
              // 小屏幕：导航到 DetailPage
              context.goNamed(
                DetailPage.name,
                queryParameters: <String, String>{'itemName': '$index'},
              );
            }
          },
        );
      },
    );
  }

  @override
  Widget build(BuildContext context) {
    return AdaptiveScaffold(
      useDrawer: false,
      // 移除 AppBar，如果需要可以在 DetailPage 中添加
      smallBody: _buildItemList,
      body: _buildItemList,
      // 小屏幕隐藏 secondaryBody
      smallSecondaryBody: AdaptiveScaffold.emptyBuilder,
      // mediumLarge 及以上显示 DetailPage
      mediumLargeSecondaryBody: (_) => DetailPage(itemName: '$_selectedIndex'),
      largeSecondaryBody: (_) => DetailPage(itemName: '$_selectedIndex'),
      extraLargeSecondaryBody: (_) => DetailPage(itemName: '$_selectedIndex'),
      // 如果没有 destinations，需要提供空列表（但 AdaptiveScaffold 要求至少 2 个）
      // 这里可以提供一个隐藏的导航栏，或者使用 AdaptiveLayout 替代
      destinations: const [
        NavigationDestination(icon: Icon(Icons.list), label: ''),
      ],
      selectedIndex: 0,
      onSelectedIndexChange: (_) {},
    );
  }
}
```

**注意**：由于 `AdaptiveScaffold` 要求至少 2 个 destinations，上述代码可能不太合适。更好的做法是使用方案1的 AdaptiveLayout。

**关键代码说明**：

1. **AdaptiveScaffold 限制**：`AdaptiveScaffold` 主要用于有导航栏的场景，需要 destinations
2. **实际建议**：如果不需要导航栏，使用 `AdaptiveLayout` 更合适（即方案1）
3. **状态管理**：使用 `StatefulWidget` 管理选中索引
4. **条件渲染**：根据屏幕尺寸决定点击行为

### 方案3：基于路由和 URL 查询参数（无需状态管理）

**优点**：

- 不需要额外的状态管理（`StatefulWidget`）
- URL 状态即数据源，状态持久化
- 可以直接使用路由系统进行导航
- 更符合声明式编程范式

**缺点**：

- 需要监听路由变化（使用 `GoRouter.of(context).routerDelegate` 或 `GoRouterState.of(context)`）
- 在小屏幕时仍需要处理路由导航（跳转到 DetailPage）
- 代码逻辑可能更复杂（需要判断屏幕尺寸和路由状态）

**实现要点**：

- `DetailOverviewPage` 保持 `StatelessWidget`
- 使用 `GoRouterState` 获取当前路由和查询参数
- 检测屏幕尺寸（使用 `Breakpoints.mediumLarge.isActive(context)`）
- 在大屏幕：根据 URL 查询参数 `itemName` 决定显示的 item，如果没有则默认第一个
- 在小屏幕：点击 item 时使用 `context.goNamed` 跳转到 `DetailPage`
- 使用 `AdaptiveLayout` 或 `AdaptiveScaffold`，根据路由状态决定 secondaryBody 内容
- 在 `app_router.dart` 中可能需要调整路由配置，支持在 DetailOverviewPage 上直接显示查询参数

**关键文件修改**：

- `example/lib/go_router_demo/pages/detail_overview_page.dart`：使用路由状态而非本地状态
- 可能需要修改 `example/lib/go_router_demo/app_router.dart`：支持查询参数传递

**参考示例**：无直接示例，需要结合路由状态和自适应布局

**完整代码实现**：

```dart
// example/lib/go_router_demo/pages/detail_overview_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_adaptive_scaffold/flutter_adaptive_scaffold.dart';
import 'package:go_router/go_router.dart';

import 'detail_page.dart';

/// The detail overview page.
class DetailOverviewPage extends StatelessWidget {
  /// Construct the detail overview page.
  const DetailOverviewPage({super.key});

  /// The path for the detail page.
  static const String path = 'detail-overview';

  /// The name for the detail page.
  static const String name = 'DetailOverview';

  static const int _itemCount = 10;

  Widget _buildItemList(BuildContext context) {
    final GoRouter router = GoRouter.of(context);
    final String? currentPath = router.routerDelegate.currentConfiguration.uri.path;
    final Map<String, String> queryParams = 
        router.routerDelegate.currentConfiguration.uri.queryParameters;
    final bool isLargeScreen = Breakpoints.mediumLarge.isActive(context);
    
    // 从 URL 查询参数获取选中的 item，如果没有则默认 0
    final int selectedIndex = isLargeScreen 
        ? int.tryParse(queryParams['selectedItem'] ?? '0') ?? 0
        : -1;

    return ListView.builder(
      itemCount: _itemCount,
      itemBuilder: (BuildContext context, int index) {
        return ListTile(
          title: Text('Item $index'),
          selected: isLargeScreen && index == selectedIndex,
          onTap: () {
            if (isLargeScreen) {
              // 大屏幕：更新 URL 查询参数（使用 replace 避免历史记录）
              final Uri currentUri = router.routerDelegate.currentConfiguration.uri;
              final Uri newUri = currentUri.replace(
                queryParameters: <String, String>{'selectedItem': '$index'},
              );
              context.go(newUri.path + (newUri.hasQuery ? '?${newUri.query}' : ''));
            } else {
              // 小屏幕：导航到 DetailPage
              context.goNamed(
                DetailPage.name,
                queryParameters: <String, String>{'itemName': '$index'},
              );
            }
          },
        );
      },
    );
  }

  Widget? _buildSecondaryBody(BuildContext context) {
    final GoRouter router = GoRouter.of(context);
    final Map<String, String> queryParams = 
        router.routerDelegate.currentConfiguration.uri.queryParameters;
    
    // 获取选中的 item，如果没有则默认 0
    final int selectedIndex = int.tryParse(queryParams['selectedItem'] ?? '0') ?? 0;
    
    return DetailPage(itemName: '$selectedIndex');
  }

  @override
  Widget build(BuildContext context) {
    final bool isLargeScreen = Breakpoints.mediumLarge.isActive(context);
    
    return Scaffold(
      appBar: AppBar(
        title: const Text('Detail Overview Page'),
      ),
      body: isLargeScreen
          ? AdaptiveLayout(
              body: SlotLayout(
                config: <Breakpoint, SlotLayoutConfig?>{
                  Breakpoints.standard: SlotLayout.from(
                    key: const Key('body'),
                    builder: (_) => _buildItemList(context),
                  ),
                },
              ),
              secondaryBody: SlotLayout(
                config: <Breakpoint, SlotLayoutConfig?>{
                  Breakpoints.mediumLarge: SlotLayout.from(
                    key: const Key('secondaryBody'),
                    builder: (_) => _buildSecondaryBody(context)!,
                  ),
                },
              ),
            )
          : _buildItemList(context),
    );
  }
}
```

**路由配置修改（可选，支持查询参数）**：如果需要支持在 URL 中传递 `selectedItem` 查询参数，可以修改 `app_router.dart`：

```dart
// 在 app_router.dart 的 DetailOverviewPage 路由中
GoRoute(
  name: DetailOverviewPage.name,
  path: DetailOverviewPage.path,
  pageBuilder: (BuildContext context, GoRouterState state) {
    return MaterialPage<void>(
      key: state.pageKey, // 确保查询参数变化时重建页面
      child: DetailOverviewPage(),
    );
  },
  routes: <RouteBase>[
    // ... DetailPage 路由
  ],
),
```

**关键代码说明**：

1. **无状态组件**：使用 `StatelessWidget`，状态存储在 URL 中
2. **路由状态获取**：使用 `GoRouter.of(context).routerDelegate.currentConfiguration.uri` 获取当前 URI 和查询参数
3. **查询参数解析**：从 URL 查询参数 `selectedItem` 获取选中索引，默认为 0
4. **URL 更新**：大屏幕点击时使用 `context.go` 更新 URL 查询参数
5. **条件布局**：根据屏幕尺寸决定是否使用 `AdaptiveLayout`
6. **状态持久化**：选中状态存储在 URL 中，刷新页面后状态保持

**注意**：这种方案在实际应用中，由于需要监听路由变化来更新 UI，可能仍需要 `StatefulWidget` 或使用 `GoRouter` 的监听机制。

### 方案4：使用嵌套的 StatefulShellRoute（Shell Route 方案）

#### 可行性评估

**技术可行性**：✅ 可行

GoRouter 支持嵌套的 `StatefulShellRoute`，可以在 Home 分支内部创建一个嵌套的 shell route 来管理列表和详情的关系。

**实现思路**：

1. 在 Home 分支内创建嵌套的 `StatefulShellRoute`
2. 两个分支：列表分支（DetailOverviewPage）和详情分支（DetailPage）
3. 创建 `DetailShellBuilder`（类似于 `ScaffoldShell`），使用 `AdaptiveLayout` 实现响应式布局
4. 小屏幕：只显示列表分支，隐藏详情分支
5. 大屏幕：使用 `AdaptiveLayout` 的 `body` 和 `secondaryBody` 同时显示两个分支

#### 优点

- ✅ **路由状态管理清晰**：列表和详情有独立的导航栈
- ✅ **状态持久化**：切换分支时状态自动保持
- ✅ **支持深度链接**：可以直接链接到特定的列表项和详情
- ✅ **符合路由架构**：利用 GoRouter 的路由系统管理状态
- ✅ **导航栈独立**：列表和详情可以各自维护导航历史

#### 缺点和挑战

- ❌ **路由结构复杂**：需要嵌套的 `StatefulShellRoute`，增加了路由配置的复杂度
- ❌ **需要更多 NavigatorKey**：每个分支都需要独立的 `NavigatorKey`
- ❌ **需要创建新的 Shell Builder**：需要创建 `DetailShellBuilder` 组件
- ❌ **可能过度设计**：`StatefulShellRoute` 通常用于应用级别的导航（如底部导航栏），用于页面级别的 master-detail 布局可能显得过重
- ❌ **小屏幕处理复杂**：需要在小屏幕时隐藏一个分支，但仍然保持路由状态

#### 实现架构

```javascript
Home 分支 (StatefulShellRoute)
└── DetailShellRoute (嵌套的 StatefulShellRoute)
    ├── ListBranch (列表分支)
    │   └── DetailOverviewPage
    └── DetailBranch (详情分支)
        └── DetailPage
```

#### 代码实现概览

**1. 路由配置（app_router.dart）**：

```dart
// 需要添加新的 NavigatorKey
final GlobalKey<NavigatorState> _detailListNavigatorKey =
    GlobalKey<NavigatorState>(debugLabel: 'detailList');
final GlobalKey<NavigatorState> _detailDetailNavigatorKey =
    GlobalKey<NavigatorState>(debugLabel: 'detailDetail');

// 在 Home 分支的 routes 中
StatefulShellRoute(
  parentNavigatorKey: _homeNavigatorKey,
  builder: (context, state, navigationShell) {
    return DetailShellBuilder(navigationShell: navigationShell);
  },
  branches: <StatefulShellBranch>[
    StatefulShellBranch(
      navigatorKey: _detailListNavigatorKey,
      routes: <RouteBase>[
        GoRoute(
          name: DetailOverviewPage.name,
          path: DetailOverviewPage.path,
          pageBuilder: (context, state) {
            return const NoTransitionPage<void>(
              child: DetailOverviewPage(),
            );
          },
        ),
      ],
    ),
    StatefulShellBranch(
      navigatorKey: _detailDetailNavigatorKey,
      routes: <RouteBase>[
        GoRoute(
          name: DetailPage.name,
          path: DetailPage.path,
          pageBuilder: (context, state) {
            return NoTransitionPage<void>(
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

**2. DetailShellBuilder 实现**：

```dart
class DetailShellBuilder extends StatelessWidget {
  const DetailShellBuilder({
    required this.navigationShell,
    super.key,
  });

  final StatefulNavigationShell navigationShell;

  @override
  Widget build(BuildContext context) {
    final bool isLargeScreen = Breakpoints.mediumLarge.isActive(context);

    if (!isLargeScreen) {
      // 小屏幕：只显示列表分支（索引 0）
      return navigationShell.route.branches[0].navigatorKey.currentContext != null
          ? navigationShell
          : const SizedBox.shrink();
    }

    // 大屏幕：使用 AdaptiveLayout 显示两个分支
    return Scaffold(
      appBar: AppBar(
        title: const Text('Detail Overview Page'),
      ),
      body: AdaptiveLayout(
        body: SlotLayout(
          config: <Breakpoint, SlotLayoutConfig?>{
            Breakpoints.standard: SlotLayout.from(
              key: const Key('listBody'),
              builder: (_) => _buildListBranch(),
            ),
          },
        ),
        secondaryBody: SlotLayout(
          config: <Breakpoint, SlotLayoutConfig?>{
            Breakpoints.mediumLarge: SlotLayout.from(
              key: const Key('detailBody'),
              builder: (_) => _buildDetailBranch(),
            ),
          },
        ),
      ),
    );
  }

  Widget _buildListBranch() {
    // 只渲染列表分支的内容
    return IndexedStack(
      index: 0,
      children: navigationShell.route.branches.map((branch) {
        // 这里需要手动构建分支内容，比较复杂
      }).toList(),
    );
  }

  Widget _buildDetailBranch() {
    // 只渲染详情分支的内容
    return IndexedStack(
      index: 1,
      children: navigationShell.route.branches.map((branch) {
        // 这里需要手动构建分支内容，比较复杂
      }).toList(),
    );
  }
}
```

**关键问题**：

- `StatefulNavigationShell` 是一个整体，不能简单地分割成两部分
- 需要自定义实现分支内容的分离渲染，这非常复杂
- GoRouter 的 `StatefulShellRoute` 设计用于显示一个 `navigationShell`，而不是同时显示多个分支

#### 可行性结论

❌ 不推荐使用此方案

**原因**：

1. **架构不匹配**：`StatefulShellRoute` 的设计初衷是应用级别的多分支导航（如底部导航栏），而不是页面级别的 master-detail 布局
2. **实现复杂**：`StatefulNavigationShell` 是一个整体 widget，无法简单分割为两部分同时显示
3. **过度设计**：对于页面级别的布局需求，使用 `AdaptiveLayout` 在页面内部实现更简单直接
4. **维护成本高**：嵌套的 `StatefulShellRoute` 增加了路由配置的复杂度，不利于维护

**替代建议**：

- **推荐方案1（AdaptiveLayout）**：在 `DetailOverviewPage` 内部使用 `AdaptiveLayout`，这是最直接和简单的方案
- 如果确实需要路由级别的状态管理，可以考虑**方案3（路由状态）**，它结合了路由状态和 `AdaptiveLayout`

## 方案对比总结

| 特性 | 方案1: AdaptiveLayout | 方案2: AdaptiveScaffold | 方案3: 路由状态 | 方案4: Shell Route |
| --- | --- | --- | --- | --- |
| 代码复杂度 | 中 | 中 | 中-高 | 高（嵌套路由） |
| 灵活性 | 最高 | 高 | 中 | 低（架构限制） |
| 状态管理 | 需要 StatefulWidget | 需要 StatefulWidget | 不需要（使用路由状态） | 路由状态（自动） |
| 学习曲线 | 中 | 平缓 | 中 | 陡（嵌套复杂） |
| 与现有代码一致性 | 中 | 高（与 ScaffoldShell 一致） | 中 | 中 |
| 架构匹配度 | ✅ 高（适合页面级布局） | ✅ 高 | ✅ 高 | ❌ 低（适合应用级导航） |
| 推荐度 | ⭐⭐⭐⭐⭐（实际推荐） | ⭐⭐（不推荐，仅作参考） | ⭐⭐⭐⭐ | ❌（不推荐，架构不匹配） |

## 实现细节说明

### 所有方案的共同点

1. **Breakpoint 选择**：
   - 使用 `Breakpoints.mediumLarge`（840 dp）作为分界点
   - small/medium (< 840 dp)：单面板布局
   - mediumLarge+ (>= 840 dp)：双面板布局

2. **默认 item 处理**：
   - 使用 `useState` 或路由状态初始化选中索引为 0
   - 在大屏幕自动显示第一个 item 的详情

3. **列表点击行为**：
   - 小屏幕：导航到 `DetailPage` 路由
   - 大屏幕：更新选中状态，在 secondaryBody 显示详情

4. **DetailPage 集成**：
   - 在所有方案中，secondaryBody 都直接渲染 `DetailPage` widget
   - 传入 `itemName` 参数（字符串形式，如 '0', '1' 等）

### 方案选择建议

- **推荐方案1（AdaptiveLayout）**：最适合当前场景，灵活且无需导航栏，代码清晰
- **方案3（路由状态）**：如果希望状态持久化到 URL，或偏好无状态组件，适合需要深度链接的场景
- **方案2（AdaptiveScaffold）**：不推荐，因为需要 destinations，不适合单个页面场景
- **方案4（Shell Route）**：❌ 不推荐，`StatefulShellRoute` 设计用于应用级别的多分支导航，不适合页面级别的 master-detail 布局

### 实际推荐

**方案1（AdaptiveLayout）是最佳选择**，因为：

1. 不需要导航栏（destinations），适合单个页面场景
2. 代码清晰，易于理解和维护
3. 灵活性高，可以精确控制布局和动画
4. 与现有代码集成简单（只需修改 DetailOverviewPage）

## 文件结构

```text
example/lib/go_router_demo/
├── app_router.dart (方案3可能需要修改)
└── pages/
    └── detail_overview_page.dart (所有方案都需要修改)
```

## 实现注意事项

### 1. DetailPage 的 AppBar 处理

在所有方案中，`DetailPage` 作为 `secondaryBody` 的一部分被嵌入到布局中。如果 `DetailPage` 包含 `AppBar`，可能需要：

- **选项A**：移除 `DetailPage` 的 `Scaffold` 和 `AppBar`，只保留内容部分
- **选项B**：保留 `AppBar`，但注意可能与父页面的布局产生冲突
- **选项C**：创建一个新的 `DetailContent` widget，不包含 `Scaffold`，专门用于 secondaryBody

**推荐选项C**：创建 `DetailContent` widget

```dart
// detail_page.dart
class DetailPage extends StatelessWidget {
  // ... 现有代码
}

// 新增：用于 secondaryBody 的内容组件
class DetailContent extends StatelessWidget {
  const DetailContent({super.key, required this.itemName});
  final String itemName;
  
  @override
  Widget build(BuildContext context) {
    return Center(
      child: Text('Detail Page: $itemName'),
    );
  }
}
```

### 2. 路由导航行为

- **小屏幕**：点击 item 时使用 `context.goNamed(DetailPage.name, ...)` 导航到新页面
- **大屏幕**：点击 item 时只更新状态，不进行路由导航
- **路由返回**：在小屏幕从 `DetailPage` 返回时，使用浏览器的返回按钮或 `context.pop()`

### 3. 状态同步

- **方案1和方案2**：状态存储在 widget 的 state 中，页面重建后状态会丢失（除非使用状态管理方案）
- **方案3**：状态存储在 URL 中，页面刷新后状态保持，支持深度链接

### 4. 性能考虑

- `SlotLayout` 的 `builder` 函数会在 breakpoint 变化时重建
- 使用 `key` 参数可以帮助 Flutter 优化 widget 重建
- `DetailPage` 的创建应该尽可能轻量，避免在 builder 中进行重计算

### 5. 测试建议

- 测试不同屏幕尺寸下的布局行为
- 测试小屏幕到大屏幕的切换（窗口调整）
- 测试路由导航的正确性
- 测试状态保持（方案3）
- 测试默认选中第一个 item 的行为
