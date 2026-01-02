# GoRouterStateRegistry 类解析

## 概述

`GoRouterStateRegistry` 是一个用于记录 `GoRouterState` 与 `Page` 之间关系的注册表类。它是 go_router 内部实现的核心组件，负责维护路由状态与页面之间的映射关系，并管理这些关系的生命周期。

```dart 217:222:lib/src/state.dart
/// A registry to record [GoRouterState] to [Page] relation.
///
/// Should not be used directly, consider using [GoRouterState.of] to access
/// [GoRouterState] from the context.
@internal
class GoRouterStateRegistry extends ChangeNotifier {
```

这个类被标记为 `@internal`，表明它是内部实现细节，不应该被外部代码直接使用。开发者应该通过 `GoRouterState.of(context)` 方法来访问路由状态。

## 核心数据结构

### registry 映射表

```dart 226:229:lib/src/state.dart
/// A [Map] that maps a [Page] to a [GoRouterState].
@visibleForTesting
final Map<Page<Object?>, GoRouterState> registry =
    <Page<Object?>, GoRouterState>{};
```

`registry` 是核心的映射表，用于存储 `Page` 对象到 `GoRouterState` 对象的映射关系。这个映射表是公开的（标记为 `@visibleForTesting`），以便在测试中访问。

### _routePageAssociation 关联表

```dart 231:232:lib/src/state.dart
final Map<Route<Object?>, Page<Object?>> _routePageAssociation =
    <ModalRoute<Object?>, Page<Object?>>{};
```

`_routePageAssociation` 是一个私有映射表，用于跟踪 `Route` 对象与 `Page` 对象之间的关联关系。这个映射主要用于生命周期管理，确保在路由完成时正确清理资源。

## 核心方法

### _createPageRouteAssociation

```dart 234:266:lib/src/state.dart
GoRouterState? _createPageRouteAssociation(
  Page<Object?> page,
  ModalRoute<Object?> route,
) {
  assert(route.settings == page);
  if (!registry.containsKey(page)) {
    return null;
  }
  final Page<Object?>? oldPage = _routePageAssociation[route];
  if (oldPage == null) {
    // This is a new association.
    _routePageAssociation[route] = page;
    // If there is an association, the registry relies on the route to remove
    // entry from registry because it wants to preserve the GoRouterState
    // until the route finishes the popping animations.
    route.completed.then<void>((Object? result) {
      // Can't use `page` directly because Route.settings may have changed during
      // the lifetime of this route.
      final Page<Object?> associatedPage = _routePageAssociation.remove(
        route,
      )!;
      assert(registry.containsKey(associatedPage));
      registry.remove(associatedPage);
    });
  } else if (oldPage != page) {
    // Need to update the association to avoid memory leak.
    _routePageAssociation[route] = page;
    assert(registry.containsKey(oldPage));
    registry.remove(oldPage);
  }
  assert(_routePageAssociation[route] == page);
  return registry[page]!;
}
```

这个方法负责创建 `Page` 和 `Route` 之间的关联关系，并返回对应的 `GoRouterState`。它的工作流程如下：

1. **验证关联**：首先检查 `route.settings` 是否等于传入的 `page`，这是一个断言检查。

2. **检查注册表**：如果 `registry` 中不包含该 `page`，直接返回 `null`。这通常发生在页面还没有被注册到注册表时。

3. **处理新关联**：如果这是 `route` 的一个新关联（`oldPage == null`）：
   - 将关联关系存入 `_routePageAssociation`
   - 注册路由完成后的清理回调，当路由完成时（`route.completed`），会从 `_routePageAssociation` 和 `registry` 中移除对应的条目
   - 这种设计确保 `GoRouterState` 在路由弹出动画完成之前不会被清理，这对于支持路由转场动画非常重要

4. **更新现有关联**：如果 `route` 已经有关联的 `page`，但现在是不同的 `page`（`oldPage != page`）：
   - 更新关联关系
   - 移除旧 `page` 的注册项，避免内存泄漏

5. **返回状态**：最后返回 `registry` 中对应的 `GoRouterState`

**关键设计点**：在清理回调中，不能直接使用传入的 `page` 参数，而是要从 `_routePageAssociation` 中获取关联的 `page`。这是因为在路由的生命周期内，`Route.settings` 可能会发生变化。

### updateRegistry

```dart 268:305:lib/src/state.dart
/// Updates this registry with new records.
void updateRegistry(Map<Page<Object?>, GoRouterState> newRegistry) {
  var shouldNotify = false;
  final Set<Page<Object?>> pagesWithAssociation = _routePageAssociation.values
      .toSet();
  for (final MapEntry<Page<Object?>, GoRouterState> entry
      in newRegistry.entries) {
    final GoRouterState? existingState = registry[entry.key];
    if (existingState != null) {
      if (existingState != entry.value) {
        shouldNotify =
            shouldNotify || pagesWithAssociation.contains(entry.key);
        registry[entry.key] = entry.value;
      }
      continue;
    }
    // Not in the _registry.
    registry[entry.key] = entry.value;
    // Adding or removing registry does not need to notify the listen since
    // no one should be depending on them.
  }
  registry.removeWhere((Page<Object?> key, GoRouterState value) {
    if (newRegistry.containsKey(key)) {
      return false;
    }
    // For those that have page route association, it will be removed by the
    // route future. Need to notify the listener so they can update the page
    // route association if its page has changed.
    if (pagesWithAssociation.contains(key)) {
      shouldNotify = true;
      return false;
    }
    return true;
  });
  if (shouldNotify) {
    notifyListeners();
  }
}
```

`updateRegistry` 方法用于批量更新注册表。这个方法通常由 go_router 的路由构建器调用，用于同步最新的路由状态。

**更新逻辑**：

1. **准备阶段**：
   - 初始化 `shouldNotify` 标志，用于判断是否需要通知监听者
   - 获取所有已关联路由的 `Page` 集合（`pagesWithAssociation`）

2. **处理新注册表条目**：
   - 遍历 `newRegistry` 中的每个条目
   - 如果 `page` 已存在于 `registry` 中：
     - 检查 `GoRouterState` 是否发生变化
     - 如果状态发生变化，且该 `page` 已经关联了路由，则标记需要通知监听者
     - 更新 `registry` 中的状态
   - 如果 `page` 不在 `registry` 中：
     - 直接添加新条目
     - 添加新条目不需要通知监听者，因为没有代码依赖于这些新条目

3. **清理过时条目**：
   - 使用 `removeWhere` 移除 `registry` 中不在 `newRegistry` 中的条目
   - 如果某个 `page` 已经关联了路由（在 `pagesWithAssociation` 中），则不能直接移除，因为清理工作将由路由完成回调处理
   - 对于这些有路由关联的 `page`，标记需要通知监听者，以便监听者可以更新关联关系

4. **通知监听者**：
   - 如果有需要通知的情况（状态变化或需要更新的关联），调用 `notifyListeners()`

**设计要点**：

- 添加或移除未关联路由的注册项不需要通知，因为这些条目还没有被使用
- 只有已经关联了路由的 `page` 的状态变化才需要通知，因为可能有代码正在使用这些状态
- 有路由关联的 `page` 不能直接移除，必须等待路由完成回调来处理，这样可以保证在路由动画期间状态仍然可用

## 与 GoRouterState.of 的配合

`GoRouterStateRegistry` 主要通过 `GoRouterState.of(context)` 方法被使用：

```dart 118:148:lib/src/state.dart
static GoRouterState of(BuildContext context) {
  ModalRoute<Object?>? route;
  GoRouterStateRegistryScope? scope;
  while (true) {
    route = ModalRoute.of(context);
    if (route == null) {
      throw _noGoRouterStateError;
    }
    final RouteSettings settings = route.settings;
    if (settings is Page<Object?>) {
      scope = context
          .dependOnInheritedWidgetOfExactType<GoRouterStateRegistryScope>();
      if (scope == null) {
        throw _noGoRouterStateError;
      }
      final GoRouterState? state = scope.notifier!
          ._createPageRouteAssociation(
            route.settings as Page<Object?>,
            route,
          );
      if (state != null) {
        return state;
      }
    }
    final NavigatorState? state = Navigator.maybeOf(context);
    if (state == null) {
      throw _noGoRouterStateError;
    }
    context = state.context;
  }
}
```

`GoRouterState.of` 方法会：

1. 通过 `GoRouterStateRegistryScope` 获取 `GoRouterStateRegistry` 实例
2. 调用 `_createPageRouteAssociation` 方法建立关联并获取状态
3. 如果找不到状态，会向上遍历 Navigator 层级继续查找

## 生命周期管理

`GoRouterStateRegistry` 继承自 `ChangeNotifier`，这意味着它可以通知监听者状态变化。这种设计允许依赖于路由状态的组件在状态更新时得到通知。

生命周期管理的关键点：

1. **延迟清理**：`GoRouterState` 在路由弹出动画完成之前不会被清理，这通过 `route.completed` 回调实现
2. **内存泄漏防护**：当路由关联的 `page` 发生变化时，及时清理旧的关联关系
3. **状态同步**：通过 `updateRegistry` 方法批量更新状态，确保注册表与当前路由配置保持一致

## 使用场景

这个类主要在以下场景中使用：

1. **路由状态查询**：当组件需要获取当前路由状态时，通过 `GoRouterState.of(context)` 间接使用
2. **路由构建**：在路由构建过程中，通过 `updateRegistry` 方法更新注册表
3. **状态同步**：当路由配置发生变化时，同步更新注册表中的状态

## 总结

`GoRouterStateRegistry` 是 go_router 实现路由状态管理的核心组件。它通过维护 `Page` 到 `GoRouterState` 的映射关系，以及与 `Route` 的关联关系，实现了路由状态的查找、更新和生命周期管理。其设计充分考虑了路由动画、内存管理和状态同步的需求，确保路由系统能够稳定高效地工作。
