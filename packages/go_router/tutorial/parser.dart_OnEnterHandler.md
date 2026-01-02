# _OnEnterHandler 代码解析

## 概述

`_OnEnterHandler` 是一个私有类，负责处理顶级 `onEnter` 回调逻辑并管理重定向历史。该类封装了执行顶级 `onEnter` 回调、强制执行路由器配置中定义的重定向限制，以及当超出限制时生成错误匹配列表的逻辑。它由 `GoRouter` 在路由解析过程中内部使用。

```dart 376:382:lib/src/parser.dart
/// Handles the top-level [onEnter] callback logic and manages redirection history.
///
/// This class encapsulates the logic to execute the top-level [onEnter] callback,
/// enforce the redirection limit defined in the router configuration, and generate
/// an error match list when the limit is exceeded. It is used internally by [GoRouter]
/// during route parsing.
class _OnEnterHandler {
```

## 核心职责

1. **执行顶级 onEnter 回调**：在导航发生前执行用户定义的 `onEnter` 回调函数
2. **管理重定向历史**：跟踪 `onEnter` 触发重定向的 URI 序列
3. **强制重定向限制**：确保重定向次数不超过配置的限制（默认 5 次）
4. **错误处理**：捕获异常并生成适当的错误 `RouteMatchList`
5. **状态构建**：从 `RouteMatchList` 构建 `GoRouterState` 供 `onEnter` 回调使用

## 类结构

### 字段

```dart 396:418:lib/src/parser.dart
  /// The current route configuration.
  ///
  /// Contains all route definitions, redirection logic, and navigation settings.
  final RouteConfiguration _configuration;

  /// Optional exception handler for route parsing errors.
  ///
  /// This handler is invoked when errors occur during route parsing (for example,
  /// when the [onEnter] redirection limit is exceeded) to return a fallback [RouteMatchList].
  final ParserExceptionHandler? _onParserException;

  /// The [GoRouter] instance used to perform navigation actions.
  ///
  /// This provides access to the imperative navigation methods (like [go], [push],
  /// [replace], etc.) and serves as a fallback reference in case the [BuildContext]
  /// does not include a [GoRouter].
  final GoRouter _router;

  /// A history of URIs encountered during [onEnter] redirections.
  ///
  /// This list tracks every URI that triggers an [onEnter] redirection, ensuring that
  /// the number of redirections does not exceed the limit defined in the router's configuration.
  final List<Uri> _redirectionHistory = <Uri>[];
```

**字段说明**：

- **`_configuration`**：当前路由配置，包含所有路由定义、重定向逻辑和导航设置
- **`_onParserException`**：可选的解析异常处理器，在路由解析错误时被调用
- **`_router`**：`GoRouter` 实例，用于执行导航操作（如 `go`、`push`、`replace` 等）
- **`_redirectionHistory`**：重定向历史列表，记录每次触发 `onEnter` 重定向的 URI，用于检测循环重定向

### 构造函数

```dart 383:394:lib/src/parser.dart
  /// Creates an [_OnEnterHandler] instance.
  ///
  /// * [configuration] is the current route configuration containing all route definitions.
  /// * [router] is the [GoRouter] instance used for navigation actions.
  /// * [onParserException] is an optional exception handler invoked on route parsing errors.
  _OnEnterHandler({
    required RouteConfiguration configuration,
    required GoRouter router,
    required ParserExceptionHandler? onParserException,
  }) : _onParserException = onParserException,
       _configuration = configuration,
       _router = router;
```

构造函数接收三个参数：

- `configuration`：必需的路由配置
- `router`：必需的 `GoRouter` 实例
- `onParserException`：可选的异常处理器

## 核心方法

### handleTopOnEnter

这是类的主要公共方法，执行顶级 `onEnter` 回调并决定是否允许导航继续。

```dart 420:566:lib/src/parser.dart
  /// Executes the top-level [onEnter] callback and determines whether navigation should proceed.
  ///
  /// It checks for redirection errors by verifying if the redirection history exceeds the
  /// configured limit. If everything is within limits, this method builds the current and
  /// next navigation states, then executes the [onEnter] callback.
  ///
  /// * If [onEnter] returns [Allow], the [onCanEnter] callback is invoked to allow navigation.
  /// * If [onEnter] returns [Block], the [onCanNotEnter] callback is invoked to block navigation.
  ///
  /// Exceptions thrown synchronously or asynchronously by [onEnter] are caught and processed
  /// via the [_onParserException] handler if available.
  ///
  /// Returns a [Future<RouteMatchList>] representing the final navigation state.
  Future<RouteMatchList> handleTopOnEnter({
    required BuildContext context,
    required RouteInformation routeInformation,
    required RouteInfoState infoState,
    required NavigationCallback onCanEnter,
    required NavigationCallback onCanNotEnter,
  }) {
```

**方法流程**：

1. **检查是否有 onEnter 回调**：

```dart 440:445:lib/src/parser.dart
    // Get the user-provided onEnter callback (legacy redirect is handled separately)
    final OnEnter? topOnEnter = _configuration.topOnEnter;
    // If no onEnter guard, allow navigation immediately.
    if (topOnEnter == null) {
      return onCanEnter();
    }
```

如果没有配置 `onEnter` 回调，直接允许导航。

1. **检查重定向限制**：

```dart 447:455:lib/src/parser.dart
    // Check if the redirection history exceeds the configured limit.
    // `routeInformation` has already been normalized by the parser entrypoint.
    final RouteMatchList? redirectionErrorMatchList =
        _redirectionErrorMatchList(context, routeInformation.uri, infoState);

    if (redirectionErrorMatchList != null) {
      // Return immediately if the redirection limit is exceeded.
      return SynchronousFuture<RouteMatchList>(redirectionErrorMatchList);
    }
```

如果重定向历史超过配置的限制（默认 5 次），立即返回错误。

1. **构建导航状态**：

```dart 457:473:lib/src/parser.dart
    // Find route matches for the normalized URI.
    final RouteMatchList incomingMatches = _configuration.findMatch(
      routeInformation.uri,
      extra: infoState.extra,
    );

    // Build the next navigation state.
    final GoRouterState nextState = _buildTopLevelGoRouterState(
      incomingMatches,
    );

    // Get the current state from the router delegate.
    final RouteMatchList currentMatchList =
        _router.routerDelegate.currentConfiguration;
    final GoRouterState currentState = currentMatchList.isNotEmpty
        ? _buildTopLevelGoRouterState(currentMatchList)
        : nextState;
```

构建当前状态和下一个状态的 `GoRouterState`，供 `onEnter` 回调使用。

1. **执行 onEnter 回调**：

```dart 475:504:lib/src/parser.dart
    // Execute the onEnter callback in a try-catch to capture synchronous exceptions.
    Future<OnEnterResult> onEnterResultFuture;
    try {
      final FutureOr<OnEnterResult> result = topOnEnter(
        context,
        currentState,
        nextState,
        _router,
      );
      // Convert FutureOr to Future
      onEnterResultFuture = result is OnEnterResult
          ? SynchronousFuture<OnEnterResult>(result)
          : result;
    } catch (error) {
      final RouteMatchList errorMatchList = _errorRouteMatchList(
        routeInformation.uri,
        error is GoException ? error : GoException(error.toString()),
        extra: infoState.extra,
      );

      _resetRedirectionHistory();

      final bool canHandleException =
          _onParserException != null && context.mounted;
      final RouteMatchList handledMatchList = canHandleException
          ? _onParserException(context, errorMatchList)
          : errorMatchList;

      return SynchronousFuture<RouteMatchList>(handledMatchList);
    }
```

使用 try-catch 捕获同步异常，并将 `FutureOr<OnEnterResult>` 转换为 `Future<OnEnterResult>`。

1. **处理 onEnter 结果**：

```dart 506:549:lib/src/parser.dart
    // Handle asynchronous completion and catch any errors.
    return onEnterResultFuture.then<RouteMatchList>(
      (OnEnterResult result) async {
        RouteMatchList matchList;
        final OnEnterThenCallback? callback = result.then;

        if (result is Allow) {
          matchList = await onCanEnter();
          _resetRedirectionHistory(); // reset after committed navigation
        } else {
          // Block: check if this is a hard stop or chaining block
          log(
            'onEnter blocked navigation from ${currentState.uri} to ${nextState.uri}',
          );
          matchList = await onCanNotEnter();

          // Treat `Block.stop()` as the explicit hard stop.
          // We intentionally don't try to detect "no-op" callbacks; any
          // Block with `then` keeps history so chained guards can detect loops.
          if (result.isStop) {
            _resetRedirectionHistory();
          }
          // For chaining blocks (with then), keep history to detect loops.
        }

        if (callback != null) {
          try {
            await Future<void>.sync(callback);
          } catch (error, stack) {
            // Log error but don't crash - navigation already committed
            log('Error in then callback: $error');
            FlutterError.reportError(
              FlutterErrorDetails(
                exception: error,
                stack: stack,
                library: 'go_router',
                context: ErrorDescription('while executing then callback'),
              ),
            );
          }
        }

        return matchList;
      },
```

**结果处理逻辑**：

- **Allow**：允许导航，调用 `onCanEnter()`，导航提交后重置重定向历史
- **Block**：
  - 如果是 `Block.stop()`（硬停止），重置重定向历史
  - 如果是 `Block.then()`（链式阻止），保留历史以检测循环重定向
- **then 回调**：如果 `OnEnterResult` 包含 `then` 回调，在导航决策提交后执行，错误不会撤销导航

1. **错误处理**：

```dart 550:565:lib/src/parser.dart
      onError: (Object error, StackTrace stackTrace) {
        // Reset history on error to prevent stale state
        _resetRedirectionHistory();

        final RouteMatchList errorMatchList = _errorRouteMatchList(
          routeInformation.uri,
          error is GoException ? error : GoException(error.toString()),
          extra: infoState.extra,
        );

        if (_onParserException != null && context.mounted) {
          return _onParserException(context, errorMatchList);
        }
        return errorMatchList;
      },
```

捕获异步错误，重置历史，并通过异常处理器处理错误（如果可用）。

### _buildTopLevelGoRouterState

构建 `GoRouterState` 的方法，从 `RouteMatchList` 中提取有效状态信息。

```dart 568:619:lib/src/parser.dart
  /// Builds a [GoRouterState] based on the given [matchList].
  ///
  /// This method derives the effective URI, full path, path parameters, and extra data from
  /// the topmost route match, drilling down through nested shells if necessary.
  ///
  /// Returns a constructed [GoRouterState] reflecting the current or next navigation state.
  GoRouterState _buildTopLevelGoRouterState(RouteMatchList matchList) {
    // Determine effective navigation state from the match list.
    Uri effectiveUri = matchList.uri;
    String? effectiveFullPath = matchList.fullPath;
    Map<String, String> effectivePathParams = matchList.pathParameters;
    String effectiveMatchedLocation = matchList.uri.path;
    Object? effectiveExtra = matchList.extra; // Base extra

    if (matchList.matches.isNotEmpty) {
      RouteMatchBase lastMatch = matchList.matches.last;
      // Drill down to the actual leaf match even inside shell routes.
      while (lastMatch is ShellRouteMatch) {
        if (lastMatch.matches.isEmpty) {
          break;
        }
        lastMatch = lastMatch.matches.last;
      }

      if (lastMatch is ImperativeRouteMatch) {
        // Use state from the imperative match.
        effectiveUri = lastMatch.matches.uri;
        effectiveFullPath = lastMatch.matches.fullPath;
        effectivePathParams = lastMatch.matches.pathParameters;
        effectiveMatchedLocation = lastMatch.matches.uri.path;
        effectiveExtra = lastMatch.matches.extra;
      } else {
        // For non-imperative matches, use the matched location and extra from the match list.
        effectiveMatchedLocation = lastMatch.matchedLocation;
        effectiveExtra = matchList.extra;
      }
    }

    return GoRouterState(
      _configuration,
      uri: effectiveUri,
      matchedLocation: effectiveMatchedLocation,
      name: matchList.lastOrNull?.route.name,
      path: matchList.lastOrNull?.route.path,
      fullPath: effectiveFullPath,
      pathParameters: effectivePathParams,
      extra: effectiveExtra,
      pageKey: const ValueKey<String>('topLevel'),
      topRoute: matchList.lastOrNull?.route,
      error: matchList.error,
    );
  }
```

**关键逻辑**：

1. **初始值**：从 `matchList` 获取基础值（URI、完整路径、路径参数等）
2. **向下遍历**：如果存在匹配，从最后一个匹配开始，向下遍历嵌套的 `ShellRouteMatch` 直到找到叶子节点
3. **命令式路由处理**：如果最后匹配是 `ImperativeRouteMatch`（通过 `GoRouter.push` 等命令式 API 推入的路由），使用其内部的 `matches` 中的状态
4. **构建状态**：使用提取的信息构建 `GoRouterState`

### _redirectionErrorMatchList

检查重定向历史是否超过限制。

```dart 621:647:lib/src/parser.dart
  /// Processes the redirection history and checks against the configured redirection limit.
  ///
  /// Adds [redirectedUri] to the history and, if the limit is exceeded, returns an error
  /// match list. Otherwise, returns null.
  RouteMatchList? _redirectionErrorMatchList(
    BuildContext context,
    Uri redirectedUri,
    RouteInfoState infoState,
  ) {
    _redirectionHistory.add(redirectedUri);
    if (_redirectionHistory.length > _configuration.redirectLimit) {
      final String formattedHistory = _formatOnEnterRedirectionHistory(
        _redirectionHistory,
      );
      final RouteMatchList errorMatchList = _errorRouteMatchList(
        redirectedUri,
        GoException('Too many onEnter calls detected: $formattedHistory'),
        extra: infoState.extra,
      );
      _resetRedirectionHistory();
      if (_onParserException != null && context.mounted) {
        return _onParserException(context, errorMatchList);
      }
      return errorMatchList;
    }
    return null;
  }
```

**工作流程**：

1. 将当前 URI 添加到重定向历史
2. 检查历史长度是否超过配置的限制（默认 5 次）
3. 如果超过限制：
   - 格式化历史为字符串用于错误报告
   - 创建错误 `RouteMatchList`
   - 重置历史
   - 通过异常处理器处理错误（如果可用）
4. 如果未超过限制，返回 `null`

### _resetRedirectionHistory

清空重定向历史。

```dart 649:652:lib/src/parser.dart
  /// Clears the redirection history.
  void _resetRedirectionHistory() {
    _redirectionHistory.clear();
  }
```

在以下情况下调用：

- 导航被允许并提交后（`Allow` 结果）
- 硬停止阻止（`Block.stop()`）
- 发生错误时
- 重定向限制被触发时

### _formatOnEnterRedirectionHistory

将重定向历史格式化为字符串，用于错误报告。

```dart 654:657:lib/src/parser.dart
  /// Formats the redirection history into a string for error reporting.
  String _formatOnEnterRedirectionHistory(List<Uri> history) {
    return history.map((Uri uri) => uri.toString()).join(' => ');
  }
```

**输出格式**：`/path1 => /path2 => /path3 => ...`

### _errorRouteMatchList

创建错误的 `RouteMatchList` 的静态方法。

```dart 659:674:lib/src/parser.dart
  /// Creates an error [RouteMatchList] for the given [uri] and [exception].
  ///
  /// This is used to encapsulate errors encountered during redirection or parsing.
  static RouteMatchList _errorRouteMatchList(
    Uri uri,
    GoException exception, {
    Object? extra,
  }) {
    return RouteMatchList(
      matches: const <RouteMatch>[],
      extra: extra,
      error: exception,
      uri: uri,
      pathParameters: const <String, String>{},
    );
  }
```

创建一个空的匹配列表，包含错误信息和原始 URI，用于错误处理流程。

## 重定向历史管理策略

重定向历史的管理策略对于防止循环重定向至关重要：

### 何时重置历史

1. **导航被允许**：`Allow` 结果后，导航提交时重置
2. **硬停止**：`Block.stop()` 后重置，表示明确的停止，不需要检测循环
3. **错误发生**：任何错误发生时重置，防止状态污染

### 何时保留历史

1. **链式阻止**：`Block.then()` 后保留历史，因为 `then` 回调可能会触发新的导航，需要检测是否形成循环

**示例场景**：

```dart
// 场景 1：硬停止，重置历史
onEnter: (context, current, next, router) {
  if (!isAuthenticated) {
    return const Block.stop(); // 重置历史
  }
  return const Allow();
}

// 场景 2：链式阻止，保留历史
onEnter: (context, current, next, router) {
  if (!isAuthenticated) {
    return Block.then(() {
      router.go('/login'); // 可能触发新的导航，需要检测循环
    });
  }
  return const Allow();
}
```

## 错误处理机制

### 同步错误处理

使用 try-catch 捕获 `onEnter` 回调中同步抛出的异常：

```dart 475:504:lib/src/parser.dart
    // Execute the onEnter callback in a try-catch to capture synchronous exceptions.
    Future<OnEnterResult> onEnterResultFuture;
    try {
      final FutureOr<OnEnterResult> result = topOnEnter(
        context,
        currentState,
        nextState,
        _router,
      );
      // Convert FutureOr to Future
      onEnterResultFuture = result is OnEnterResult
          ? SynchronousFuture<OnEnterResult>(result)
          : result;
    } catch (error) {
      final RouteMatchList errorMatchList = _errorRouteMatchList(
        routeInformation.uri,
        error is GoException ? error : GoException(error.toString()),
        extra: infoState.extra,
      );

      _resetRedirectionHistory();

      final bool canHandleException =
          _onParserException != null && context.mounted;
      final RouteMatchList handledMatchList = canHandleException
          ? _onParserException(context, errorMatchList)
          : errorMatchList;

      return SynchronousFuture<RouteMatchList>(handledMatchList);
    }
```

### 异步错误处理

使用 `Future.then` 的 `onError` 回调处理异步错误：

```dart 550:565:lib/src/parser.dart
      onError: (Object error, StackTrace stackTrace) {
        // Reset history on error to prevent stale state
        _resetRedirectionHistory();

        final RouteMatchList errorMatchList = _errorRouteMatchList(
          routeInformation.uri,
          error is GoException ? error : GoException(error.toString()),
          extra: infoState.extra,
        );

        if (_onParserException != null && context.mounted) {
          return _onParserException(context, errorMatchList);
        }
        return errorMatchList;
      },
```

### then 回调错误处理

`then` 回调中的错误不会撤销导航，只会被记录：

```dart 531:546:lib/src/parser.dart
        if (callback != null) {
          try {
            await Future<void>.sync(callback);
          } catch (error, stack) {
            // Log error but don't crash - navigation already committed
            log('Error in then callback: $error');
            FlutterError.reportError(
              FlutterErrorDetails(
                exception: error,
                stack: stack,
                library: 'go_router',
                context: ErrorDescription('while executing then callback'),
              ),
            );
          }
        }
```

## 使用场景

### 场景 1：权限检查

```dart
GoRouter(
  onEnter: (context, current, next, router) {
    if (next.uri.path.startsWith('/admin') && !isAdmin()) {
      return Block.then(() {
        router.go('/unauthorized');
      });
    }
    return const Allow();
  },
  // ...
)
```

### 场景 2：认证检查

```dart
GoRouter(
  onEnter: (context, current, next, router) {
    if (requiresAuth(next.uri.path) && !isAuthenticated()) {
      return Block.then(() {
        router.go('/login?redirect=${next.uri}');
      });
    }
    return const Allow();
  },
  // ...
)
```

### 场景 3：硬停止阻止

```dart
GoRouter(
  onEnter: (context, current, next, router) {
    if (isMaintenanceMode()) {
      // 硬停止，不执行任何后续操作
      return const Block.stop();
    }
    return const Allow();
  },
  // ...
)
```

## 设计模式

### 策略模式

`OnEnterResult` 及其子类（`Allow`、`Block`）实现了策略模式，不同的结果对应不同的处理策略。

### 责任链模式

重定向历史检查、`onEnter` 执行、错误处理构成了一个责任链，每个环节处理特定职责。

### 模板方法模式

`handleTopOnEnter` 定义了处理 `onEnter` 的标准流程，具体步骤通过回调函数（`onCanEnter`、`onCanNotEnter`）自定义。

## 注意事项

1. **重定向限制**：默认重定向限制为 5 次，超过限制会触发错误
2. **历史管理**：理解 `Block.stop()` 和 `Block.then()` 对重定向历史的不同影响
3. **错误隔离**：`then` 回调中的错误不会撤销导航，只会被报告
4. **异步支持**：`onEnter` 回调支持异步操作，返回 `Future<OnEnterResult>`
5. **状态构建**：`_buildTopLevelGoRouterState` 会向下遍历嵌套的 shell route 以找到真正的叶子节点
6. **上下文检查**：在调用异常处理器前检查 `context.mounted`，确保上下文仍然有效

## 相关类型

- **`OnEnter`**：`onEnter` 回调的函数签名
- **`OnEnterResult`**：`onEnter` 回调的返回类型（`Allow` 或 `Block`）
- **`GoRouterState`**：导航状态，包含当前和目标路由信息
- **`RouteMatchList`**：路由匹配列表，表示导航的最终状态
- **`ParserExceptionHandler`**：解析异常处理器类型

## 总结

`_OnEnterHandler` 是 `go_router` 中处理导航拦截的核心类，它通过执行 `onEnter` 回调、管理重定向历史、强制重定向限制和错误处理，确保了路由导航的安全性和可靠性。通过区分硬停止和链式阻止，它能够在阻止导航的同时支持重定向操作，并通过历史管理机制防止循环重定向问题。
