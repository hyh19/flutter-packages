# GoRouterDelegate 类说明文档

## 概述

`GoRouterDelegate` 是 GoRouter 包中 `RouterDelegate` 接口的具体实现类，负责管理路由导航、处理路由配置变更以及构建 Navigator 组件。它是 GoRouter 路由系统的核心组件之一，实现了 Flutter 路由架构中的委托模式。

## 类定义

```dart 18:20:lib/src/delegate.dart
/// GoRouter implementation of [RouterDelegate].
class GoRouterDelegate extends RouterDelegate<RouteMatchList>
    with ChangeNotifier {
```

该类：

- 继承自 `RouterDelegate<RouteMatchList>`，实现了 Flutter 路由委托接口
- 混入了 `ChangeNotifier`，用于通知路由配置的变更
- 泛型参数 `RouteMatchList` 表示路由配置的类型

## 构造函数

```dart 23:43:lib/src/delegate.dart
  GoRouterDelegate({
    required RouteConfiguration configuration,
    required GoRouterBuilderWithNav builderWithNav,
    required GoRouterPageBuilder? errorPageBuilder,
    required GoRouterWidgetBuilder? errorBuilder,
    required List<NavigatorObserver> observers,
    required this.routerNeglect,
    String? restorationScopeId,
    bool requestFocus = true,
  }) : _configuration = configuration {
    builder = RouteBuilder(
      configuration: configuration,
      builderWithNav: builderWithNav,
      errorPageBuilder: errorPageBuilder,
      errorBuilder: errorBuilder,
      restorationScopeId: restorationScopeId,
      observers: observers,
      onPopPageWithRouteMatch: _handlePopPageWithRouteMatch,
      requestFocus: requestFocus,
    );
  }
```

### 参数说明

- **`configuration`**: 路由配置对象，包含路由规则和导航键
- **`builderWithNav`**: 用于构建带导航器的路由页面的构建器
- **`errorPageBuilder`**: 错误页面的页面构建器（可选）
- **`errorBuilder`**: 错误页面的 Widget 构建器（可选）
- **`observers`**: Navigator 观察者列表，用于监听路由变化
- **`routerNeglect`**: 是否禁用 Web 平台的历史记录创建（已过时，将在下个大版本中移除）
- **`restorationScopeId`**: 路由恢复范围 ID（可选）
- **`requestFocus`**: 是否请求焦点（默认为 true）

构造函数内部创建了一个 `RouteBuilder` 实例，该实例负责实际的 Navigator 构建工作。

## 核心属性

### builder

```dart 45:47:lib/src/delegate.dart
  /// Builds the top-level Navigator given a configuration and location.
  @visibleForTesting
  late final RouteBuilder builder;
```

用于构建顶级 Navigator 的构建器，在测试中可见。

### routerNeglect

```dart 49:52:lib/src/delegate.dart
  /// Set to true to disable creating history entries on the web.
  // TODO(tolo): This field is obsolete and should be removed in the next major
  // version.
  final bool routerNeglect;
```

用于禁用 Web 平台历史记录创建的标志，已标记为过时。

### currentConfiguration

```dart 209:211:lib/src/delegate.dart
  /// For use by the Router architecture as part of the RouterDelegate.
  @override
  RouteMatchList currentConfiguration = RouteMatchList.empty;
```

当前的路由配置，表示当前活跃的路由匹配列表。

### navigatorKey

```dart 206:207:lib/src/delegate.dart
  /// For use by the Router architecture as part of the RouterDelegate.
  GlobalKey<NavigatorState> get navigatorKey => _configuration.navigatorKey;
```

Navigator 的全局键，从路由配置中获取。

### state

```dart 199:204:lib/src/delegate.dart
  /// The top [GoRouterState], the state of the route that was
  /// last used in either [GoRouter.go] or [GoRouter.push].
  GoRouterState get state => currentConfiguration.last.buildState(
    _configuration,
    currentConfiguration,
  );
```

获取当前顶层路由的状态对象，即最后一个通过 `GoRouter.go` 或 `GoRouter.push` 使用的路由状态。

## 核心方法

### popRoute()

```dart 56:79:lib/src/delegate.dart
  @override
  Future<bool> popRoute() async {
    final Iterable<NavigatorState> states = _findCurrentNavigators();
    for (final state in states) {
      final bool didPop = await state.maybePop(); // Call maybePop() directly
      if (didPop) {
        return true; // Return true if maybePop handled the pop
      }
    }

    // Fallback to onExit if maybePop did not handle the pop
    final GoRoute lastRoute = currentConfiguration.last.route;
    if (lastRoute.onExit != null && navigatorKey.currentContext != null) {
      return !(await lastRoute.onExit!(
        navigatorKey.currentContext!,
        currentConfiguration.last.buildState(
          _configuration,
          currentConfiguration,
        ),
      ));
    }

    return false;
  }
```

处理系统返回按钮或路由弹出操作。该方法：

1. 首先尝试找到所有当前的 Navigator 状态
2. 依次调用每个 Navigator 的 `maybePop()` 方法
3. 如果 `maybePop()` 成功处理弹出，返回 `true`
4. 如果 `maybePop()` 无法处理，回退到使用路由的 `onExit` 回调
5. 如果 `onExit` 返回 `true`（允许退出），则返回 `false`（表示未处理，需要其他方式处理）

### canPop()

```dart 81:97:lib/src/delegate.dart
  /// Returns `true` if the active Navigator can pop.
  bool canPop() {
    if (navigatorKey.currentState?.canPop() ?? false) {
      return true;
    }
    if (currentConfiguration.matches.isEmpty) {
      return false;
    }
    RouteMatchBase walker = currentConfiguration.matches.last;
    while (walker is ShellRouteMatch) {
      if (walker.navigatorKey.currentState?.canPop() ?? false) {
        return true;
      }
      walker = walker.matches.last;
    }
    return false;
  }
```

检查当前活跃的 Navigator 是否可以弹出路由。该方法：

1. 首先检查顶级 Navigator 是否可以弹出
2. 如果当前配置为空，返回 `false`
3. 遍历 Shell 路由匹配，检查每个 Shell 路由的 Navigator 是否可以弹出
4. 如果任何一个 Navigator 可以弹出，返回 `true`

### pop()

```dart 99:108:lib/src/delegate.dart
  /// Pops the top-most route.
  void pop<T extends Object?>([T? result]) {
    final Iterable<NavigatorState> states = _findCurrentNavigators().where(
      (NavigatorState element) => element.canPop(),
    );
    if (states.isEmpty) {
      throw GoError('There is nothing to pop');
    }
    states.first.pop(result);
  }
```

弹出顶层路由。该方法：

1. 找到所有可以弹出的 Navigator 状态
2. 如果没有可弹出的 Navigator，抛出 `GoError`
3. 弹出第一个可弹出的 Navigator 的顶层路由

### _findCurrentNavigators()

```dart 110:139:lib/src/delegate.dart
  /// Get a prioritized list of NavigatorStates,
  /// which either can pop or are exit routes.
  ///
  /// 1. Sub route within branches of shell navigation
  /// 2. Branch route
  /// 3. Parent route
  Iterable<NavigatorState> _findCurrentNavigators() {
    final states = <NavigatorState>[];
    if (navigatorKey.currentState != null) {
      // Set state directly without canPop check
      states.add(navigatorKey.currentState!);
    }

    RouteMatchBase walker = currentConfiguration.matches.last;
    while (walker is ShellRouteMatch) {
      final NavigatorState potentialCandidate =
          walker.navigatorKey.currentState!;

      final ModalRoute<dynamic>? modalRoute = ModalRoute.of(
        potentialCandidate.context,
      );
      if (modalRoute == null || !modalRoute.isCurrent) {
        // Stop if there is a pageless route on top of the shell route.
        break;
      }
      states.add(potentialCandidate);
      walker = walker.matches.last;
    }
    return states.reversed;
  }
```

获取当前 Navigator 状态的优先列表。该方法：

1. 添加顶级 Navigator 状态（如果存在）
2. 遍历 Shell 路由匹配，从最深层开始向上查找
3. 检查每个 Shell 路由的 Navigator 是否处于当前状态
4. 如果遇到无页面的路由（pageless route），停止查找
5. 返回反转后的列表（从子路由到父路由的顺序）

注释说明优先级顺序为：

1. Shell 导航分支内的子路由
2. 分支路由
3. 父路由

### setNewRoutePath()

```dart 219:278:lib/src/delegate.dart
  /// For use by the Router architecture as part of the RouterDelegate.
  // This class avoids using async to make sure the route is processed
  // synchronously if possible.
  @override
  Future<void> setNewRoutePath(RouteMatchList configuration) {
    if (currentConfiguration == configuration) {
      return SynchronousFuture<void>(null);
    }

    assert(configuration.isNotEmpty || configuration.isError);

    final BuildContext? navigatorContext = navigatorKey.currentContext;
    // If navigator is not built or disposed, the GoRoute.onExit is irrelevant.
    if (navigatorContext != null) {
      final currentGoRouteMatches = <RouteMatch>[];
      currentConfiguration.visitRouteMatches((RouteMatchBase match) {
        if (match is RouteMatch) {
          currentGoRouteMatches.add(match);
        }
        return true;
      });
      final newGoRouteMatches = <RouteMatch>[];
      configuration.visitRouteMatches((RouteMatchBase match) {
        if (match is RouteMatch) {
          newGoRouteMatches.add(match);
        }
        return true;
      });

      final int compareUntil = math.min(
        currentGoRouteMatches.length,
        newGoRouteMatches.length,
      );
      var indexOfFirstDiff = 0;
      for (; indexOfFirstDiff < compareUntil; indexOfFirstDiff++) {
        if (currentGoRouteMatches[indexOfFirstDiff] !=
            newGoRouteMatches[indexOfFirstDiff]) {
          break;
        }
      }

      if (indexOfFirstDiff < currentGoRouteMatches.length) {
        final List<RouteMatch> exitingMatches = currentGoRouteMatches
            .sublist(indexOfFirstDiff)
            .toList();
        return _callOnExitStartsAt(
          exitingMatches.length - 1,
          context: navigatorContext,
          matches: exitingMatches,
        ).then<void>((bool exit) {
          if (!exit) {
            return SynchronousFuture<void>(null);
          }
          return _setCurrentConfiguration(configuration);
        });
      }
    }

    return _setCurrentConfiguration(configuration);
  }
```

设置新的路由路径。这是 `RouterDelegate` 接口的核心方法，当路由配置需要更新时会被调用。该方法：

1. **快速返回**：如果新配置与当前配置相同，同步返回
2. **断言检查**：确保新配置不为空或者是错误配置
3. **收集路由匹配**：遍历当前和新配置，收集所有的 `RouteMatch` 对象
4. **比较差异**：找到第一个不同的路由匹配位置
5. **调用 onExit**：如果有需要退出的路由，从最后一个开始依次调用 `onExit` 回调
6. **更新配置**：如果所有 `onExit` 都返回 `true`（允许退出），更新当前配置

该方法注释说明尽量使用同步处理以避免异步操作。

### _callOnExitStartsAt()

```dart 280:317:lib/src/delegate.dart
  /// Calls [GoRoute.onExit] starting from the index
  ///
  /// The returned future resolves to true if all routes below the index all
  /// return true. Otherwise, the returned future resolves to false.
  Future<bool> _callOnExitStartsAt(
    int index, {
    required BuildContext context,
    required List<RouteMatch> matches,
  }) {
    if (index < 0) {
      return SynchronousFuture<bool>(true);
    }
    final RouteMatch match = matches[index];
    final GoRoute goRoute = match.route;
    if (goRoute.onExit == null) {
      return _callOnExitStartsAt(index - 1, context: context, matches: matches);
    }

    Future<bool> handleOnExitResult(bool exit) {
      if (exit) {
        return _callOnExitStartsAt(
          index - 1,
          context: context,
          matches: matches,
        );
      }
      return SynchronousFuture<bool>(false);
    }

    final FutureOr<bool> exitFuture = goRoute.onExit!(
      context,
      match.buildState(_configuration, currentConfiguration),
    );
    if (exitFuture is bool) {
      return handleOnExitResult(exitFuture);
    }
    return exitFuture.then<bool>(handleOnExitResult);
  }
```

从指定索引开始调用 `GoRoute.onExit` 回调。该方法：

1. **边界检查**：如果索引小于 0，返回 `true`（所有路由都已处理）
2. **跳过无回调的路由**：如果路由没有 `onExit` 回调，递归处理前一个路由
3. **处理回调结果**：调用 `onExit` 回调，处理返回结果（可能是 `bool` 或 `Future<bool>`）
4. **递归调用**：如果回调返回 `true`（允许退出），递归处理前一个路由；否则返回 `false`（阻止退出）

该方法从最后一个需要退出的路由开始，依次向前调用 `onExit`，确保所有路由都同意退出后才允许路由变更。

### build()

```dart 213:217:lib/src/delegate.dart
  /// For use by the Router architecture as part of the RouterDelegate.
  @override
  Widget build(BuildContext context) {
    return builder.build(context, currentConfiguration, routerNeglect);
  }
```

构建 Widget 树。这是 `RouterDelegate` 接口要求实现的方法，用于构建路由系统的 UI。该方法委托给 `RouteBuilder` 来构建实际的 Navigator。

### _handlePopPageWithRouteMatch()

```dart 141:171:lib/src/delegate.dart
  bool _handlePopPageWithRouteMatch(
    Route<Object?> route,
    Object? result,
    RouteMatchBase match,
  ) {
    if (route.willHandlePopInternally) {
      final bool popped = route.didPop(result);
      assert(!popped);
      return popped;
    }
    final RouteBase routeBase = match.route;
    if (routeBase is! GoRoute || routeBase.onExit == null) {
      route.didPop(result);
      _completeRouteMatch(result, match);
      return true;
    }

    // The _handlePopPageWithRouteMatch is called during draw frame, schedule
    // a microtask in case the onExit callback want to launch dialog or other
    // navigator operations.
    scheduleMicrotask(() async {
      final bool onExitResult = await routeBase.onExit!(
        navigatorKey.currentContext!,
        match.buildState(_configuration, currentConfiguration),
      );
      if (onExitResult) {
        _completeRouteMatch(result, match);
      }
    });
    return false;
  }
```

处理页面弹出时的路由匹配处理。该方法：

1. **内部处理检查**：如果路由内部处理弹出，调用 `didPop` 并返回
2. **无 onExit 回调**：如果路由不是 `GoRoute` 或没有 `onExit` 回调，直接完成路由匹配并返回 `true`
3. **异步处理 onExit**：如果有 `onExit` 回调，使用 `scheduleMicrotask` 异步调用，因为该方法在绘制帧期间被调用，异步执行可以避免在回调中启动对话框或其他导航操作时出现问题
4. **完成路由匹配**：如果 `onExit` 返回 `true`，完成路由匹配

### _completeRouteMatch()

```dart 181:197:lib/src/delegate.dart
  void _completeRouteMatch(Object? result, RouteMatchBase match) {
    var walker = match;
    while (walker is ShellRouteMatch) {
      walker = walker.matches.last;
    }
    if (walker is ImperativeRouteMatch) {
      walker.complete(result);
    }

    // Unconditionally remove the match from the current configuration
    currentConfiguration = currentConfiguration.remove(match);

    notifyListeners();

    // Ensure the configuration is not empty
    _debugAssertMatchListNotEmpty();
  }
```

完成路由匹配的清理工作。该方法：

1. **查找底层匹配**：如果是 `ShellRouteMatch`，向下遍历到最底层的匹配
2. **完成命令式路由**：如果是 `ImperativeRouteMatch`（通过命令式 API 创建的路由），调用 `complete` 方法
3. **移除匹配**：从当前配置中移除该匹配
4. **通知监听者**：调用 `notifyListeners()` 通知路由配置已变更
5. **调试断言**：确保配置不为空

### _setCurrentConfiguration()

```dart 319:323:lib/src/delegate.dart
  Future<void> _setCurrentConfiguration(RouteMatchList configuration) {
    currentConfiguration = configuration;
    notifyListeners();
    return SynchronousFuture<void>(null);
  }
```

设置当前配置的内部方法。该方法：

1. 更新 `currentConfiguration`
2. 通知监听者配置已变更
3. 同步返回（使用 `SynchronousFuture`）

## 工作流程

### 路由导航流程

1. 外部调用 `setNewRoutePath()` 设置新路由
2. 比较新旧配置，找出需要退出的路由
3. 依次调用需要退出路由的 `onExit` 回调
4. 如果所有 `onExit` 都返回 `true`，更新配置并通知监听者
5. `build()` 方法被调用，使用新的配置构建 Navigator

### 路由弹出流程

1. 系统或用户触发弹出操作（如返回按钮）
2. `popRoute()` 被调用
3. 找到所有当前 Navigator 状态
4. 依次尝试 `maybePop()`
5. 如果 `maybePop()` 无法处理，调用路由的 `onExit` 回调
6. 根据 `onExit` 的结果决定是否允许弹出

### 路由完成流程

1. 页面被弹出时，`_handlePopPageWithRouteMatch()` 被调用
2. 检查路由是否有 `onExit` 回调
3. 如果有，异步调用 `onExit`
4. 如果 `onExit` 返回 `true`，调用 `_completeRouteMatch()` 完成清理
5. 从配置中移除匹配，通知监听者

## 设计模式

### 委托模式

`GoRouterDelegate` 实现了 `RouterDelegate` 接口，这是典型的委托模式，将路由管理的职责委托给 `GoRouterDelegate`。

### 观察者模式

通过混入 `ChangeNotifier`，`GoRouterDelegate` 可以在配置变更时通知所有监听者（如 `Router` Widget）。

### 策略模式

实际的 Navigator 构建策略由 `RouteBuilder` 负责，`GoRouterDelegate` 只是委托调用。

## 注意事项

1. **异步处理**：`onExit` 回调是异步的，需要正确处理 `FutureOr<bool>` 类型
2. **Shell 路由**：Shell 路由具有嵌套的 Navigator，需要特殊处理
3. **配置不可为空**：除了错误配置外，路由配置不能为空
4. **同步优先**：尽可能使用同步处理以提高性能
5. **线程安全**：`notifyListeners()` 的调用需要确保在正确的上下文中

## 总结

`GoRouterDelegate` 是 GoRouter 路由系统的核心组件，负责：

- 管理路由配置的变更
- 处理路由的弹出操作
- 协调 `onExit` 回调的执行
- 构建 Navigator Widget 树
- 维护路由状态和通知机制

它通过委托模式和观察者模式，实现了 Flutter 路由架构的要求，为上层 API（如 `GoRouter`）提供了稳定的路由管理能力。
