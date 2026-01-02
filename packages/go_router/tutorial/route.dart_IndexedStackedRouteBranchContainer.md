# _IndexedStackedRouteBranchContainer 类解析

## 概述

`_IndexedStackedRouteBranchContainer` 是 go_router 包中用于管理路由分支 Navigator 容器的默认实现类。它使用 `IndexedStack` 作为底层容器，用于在 `StatefulShellRoute` 中管理和展示多个路由分支的导航状态。

## 类定义

```dart 1668:1676:lib/src/route.dart
class _IndexedStackedRouteBranchContainer extends StatelessWidget {
  const _IndexedStackedRouteBranchContainer({
    required this.currentIndex,
    required this.children,
  });

  final int currentIndex;

  final List<Widget> children;
```

这是一个私有类（以下划线 `_` 开头），继承自 `StatelessWidget`，包含两个必需的属性：

- **`currentIndex`**：当前激活的路由分支索引
- **`children`**：所有路由分支的 Widget 列表（每个分支对应一个 Navigator）

## 使用场景

该类主要用于 `StatefulShellRoute.indexedStack` 构造函数的默认实现中。当开发者使用 `StatefulShellRoute.indexedStack` 创建带状态的路由时，会自动使用这个容器来管理分支 Navigator。

```dart 1054:1063:lib/src/route.dart
  static Widget _indexedStackContainerBuilder(
    BuildContext context,
    StatefulNavigationShell navigationShell,
    List<Widget> children,
  ) {
    return _IndexedStackedRouteBranchContainer(
      currentIndex: navigationShell.currentIndex,
      children: children,
    );
  }
```

## 核心实现

### build 方法

```dart 1678:1688:lib/src/route.dart
  @override
  Widget build(BuildContext context) {
    final List<Widget> stackItems = children
        .mapIndexed(
          (int index, Widget child) =>
              _buildRouteBranchContainer(context, currentIndex == index, child),
        )
        .toList();

    return IndexedStack(index: currentIndex, children: stackItems);
  }
```

`build` 方法的执行流程：

1. **遍历所有子 Widget**：使用 `mapIndexed`（来自 `package:collection/collection.dart`）遍历 `children` 列表，为每个分支创建一个包装容器
2. **判断激活状态**：通过比较 `index` 和 `currentIndex` 来确定当前分支是否激活
3. **构建容器列表**：调用 `_buildRouteBranchContainer` 为每个子 Widget 创建包装容器
4. **返回 IndexedStack**：使用处理后的列表创建 `IndexedStack`，并指定当前激活的索引

### _buildRouteBranchContainer 方法

```dart 1690:1699:lib/src/route.dart
  Widget _buildRouteBranchContainer(
    BuildContext context,
    bool isActive,
    Widget child,
  ) {
    return Offstage(
      offstage: !isActive,
      child: TickerMode(enabled: isActive, child: child),
    );
  }
```

该方法为每个路由分支创建一个优化包装器，使用两个关键 Widget：

- **`Offstage`**：控制 Widget 的显示/隐藏
  - `offstage: !isActive`：当分支未激活时，将 Widget 移出渲染树（但仍保持状态）
- **`TickerMode`**：控制动画 ticker 的启用/禁用
  - `enabled: isActive`：只有激活的分支才会运行动画，节省资源

## 设计优势

### 1. 状态保持

使用 `IndexedStack` 配合 `Offstage` 的策略，确保所有分支的 Navigator 状态都能被保持。即使某个分支暂时不可见（`offstage: true`），其导航堆栈仍然存在，用户切换回来时可以立即恢复到之前的状态。

### 2. 性能优化

通过 `TickerMode` 禁用非激活分支的动画 ticker，可以显著减少不必要的资源消耗。这对于拥有多个分支的应用（如底部导航栏应用）尤其重要。

### 3. 简单高效

使用 `IndexedStack` 作为底层容器，实现简洁且高效。`IndexedStack` 只渲染当前激活的子 Widget，但会保持所有子 Widget 的状态，完美契合 `StatefulShellRoute` 的需求。

## 工作原理示意

```text
用户界面（当前显示分支 0）
    ↓
IndexedStack(index: 0)
    ├─ stackItems[0] (isActive: true)
    │   └─ Offstage(offstage: false)
    │       └─ TickerMode(enabled: true) → Navigator for Branch 0 ✅
    ├─ stackItems[1] (isActive: false)
    │   └─ Offstage(offstage: true)
    │       └─ TickerMode(enabled: false) → Navigator for Branch 1 (保持状态但不渲染)
    └─ stackItems[2] (isActive: false)
        └─ Offstage(offstage: true)
            └─ TickerMode(enabled: false) → Navigator for Branch 2 (保持状态但不渲染)
```

## 与自定义容器的对比

开发者也可以提供自定义的 `navigatorContainerBuilder` 来替代这个默认实现，例如：

- 使用 `TabBarView` 实现滑动切换效果
- 使用自定义动画实现淡入淡出等过渡效果
- 使用不同的布局策略

但对于大多数使用场景，`_IndexedStackedRouteBranchContainer` 提供的基于 `IndexedStack` 的实现已经足够且高效。

## 总结

`_IndexedStackedRouteBranchContainer` 是 go_router 中 `StatefulShellRoute` 的默认分支容器实现，它通过 `IndexedStack`、`Offstage` 和 `TickerMode` 的组合，实现了高效的路由分支管理和状态保持机制。这种设计既保证了用户体验（状态保持），又优化了性能（禁用非激活分支的动画），是一个典型的 Flutter 性能优化实践。
