# on_enter.dart 代码解析

## 概述

`on_enter.dart` 文件定义了 `go_router` 包中用于导航拦截的核心类型系统。该模块提供了在路由导航发生前进行拦截和控制的机制，允许开发者根据业务逻辑决定是否允许导航继续进行。

## 核心组件

### OnEnterThenCallback

```dart 7:8:lib/src/on_enter.dart
/// Signature for callbacks invoked after an [OnEnterResult] is resolved.
typedef OnEnterThenCallback = FutureOr<void> Function();
```

这是一个类型别名，定义了在 `OnEnterResult` 决策完成后执行的回调函数签名。

**特点**：

- 支持同步和异步操作（`FutureOr<void>`）
- 无参数，无返回值
- 在导航决策提交后执行

**使用场景**：

- 导航被允许后执行清理工作
- 导航被阻止后执行重定向操作
- 记录导航日志或分析数据

### OnEnterResult

```dart 10:25:lib/src/on_enter.dart
/// The result of an onEnter callback.
///
/// This sealed class represents the possible outcomes of navigation interception.
/// This class can't be extended. One must use one of its subtypes, [Allow] or
/// [Block], to indicate the result.
sealed class OnEnterResult {
  /// Creates an [OnEnterResult].
  const OnEnterResult({this.then});

  /// Executed after the decision is committed.
  /// Errors are reported and do not revert navigation.
  final OnEnterThenCallback? then;

  /// Whether this block represents a hard stop without a follow-up callback.
  bool get isStop => this is Block && then == null;
}
```

`OnEnterResult` 是一个密封类（sealed class），表示导航拦截的结果。它不能被直接实例化，必须使用其子类 `Allow` 或 `Block`。

**设计特点**：

1. **密封类设计**：使用 `sealed` 关键字确保类型安全，编译器可以检查所有可能的情况
2. **可选回调**：通过 `then` 字段支持在决策后执行额外操作
3. **错误处理**：`then` 回调中的错误会被报告但不会撤销已提交的导航

**isStop 属性**：

用于判断是否为"硬停止"（hard stop），即阻止导航且不执行后续回调的情况。这在处理重定向历史时很重要。

### Allow

```dart 27:36:lib/src/on_enter.dart
/// Allows the navigation to proceed.
///
/// The [then] callback runs **after** the navigation is committed. Errors
/// thrown by this callback are reported via `FlutterError.reportError` and
/// do **not** undo the already-committed navigation.
final class Allow extends OnEnterResult {
  /// Creates an [Allow] result with an optional [then] callback executed after
  /// navigation completes.
  const Allow({super.then});
}
```

`Allow` 类表示允许导航继续进行。

**关键特性**：

- **导航继续**：返回 `Allow` 后，导航会正常进行
- **回调时机**：`then` 回调在导航**提交后**执行，此时导航已经完成
- **错误隔离**：`then` 回调中的错误不会影响已完成的导航，只会通过 `FlutterError.reportError` 报告

**使用示例**：

```dart
// 允许导航，并在导航完成后记录日志
return Allow(
  then: () {
    Analytics.trackPageView(nextState.uri);
  },
);

// 简单允许，无后续操作
return const Allow();
```

### Block

```dart 38:63:lib/src/on_enter.dart
/// Blocks the navigation from proceeding.
///
/// Returning an object of this class from an `onEnter` callback halts the
/// navigation completely.
///
/// Use [Block.stop] for a "hard stop" that resets the redirection history, or
/// [Block.then] to chain a callback after the block (commonly to redirect
/// elsewhere, e.g. `router.go('/login')`).
///
/// Note: We don't introspect callback bodies. Even an empty closure still
/// counts as chaining, so prefer [Block.stop] when you want the hard stop
/// behavior.
final class Block extends OnEnterResult {
  /// Creates a [Block] that stops navigation without running a follow-up
  /// callback.
  ///
  /// Returning an object created by this constructor from an `onEnter`
  /// callback halts the navigation completely and resets the redirection
  /// history so the next attempt is evaluated fresh.
  const Block.stop() : super();

  /// Creates a [Block] that runs [then] after the navigation is blocked.
  ///
  /// Keeps the redirection history to detect loops during chained redirects.
  const Block.then(OnEnterThenCallback then) : super(then: then);
}
```

`Block` 类表示阻止导航继续进行。

**两种构造方式**：

#### 1. Block.stop()

```dart 57:57:lib/src/on_enter.dart
const Block.stop() : super();
```

创建一个"硬停止"，特点：

- **完全阻止导航**：导航请求被完全取消
- **重置重定向历史**：清空重定向历史记录，下次尝试时会重新评估
- **无后续回调**：不执行任何 `then` 回调

**适用场景**：

- 用户未登录时阻止访问受保护页面
- 权限不足时阻止访问
- 需要完全停止导航的场景

#### 2. Block.then()

```dart 62:62:lib/src/on_enter.dart
const Block.then(OnEnterThenCallback then) : super(then: then);
```

创建一个带回调的阻止，特点：

- **阻止导航**：当前导航被阻止
- **保留重定向历史**：保持重定向历史，用于检测循环重定向
- **执行回调**：在阻止后执行 `then` 回调

**适用场景**：

- 阻止导航并重定向到登录页：`Block.then(() => router.go('/login'))`
- 阻止导航并显示错误提示
- 需要链式重定向的场景

**重要提示**：

代码注释明确指出，框架不会检查回调函数体是否为空。即使是一个空的闭包，也会被视为"链式调用"，因此：

- 如果只需要硬停止，使用 `Block.stop()`
- 如果需要执行后续操作，使用 `Block.then()`

## 工作流程

### 导航拦截流程

1. **触发 onEnter 回调**：当导航请求发生时，`onEnter` 回调被调用
2. **返回 OnEnterResult**：回调必须返回 `Allow` 或 `Block`
3. **处理结果**：
   - 如果返回 `Allow`：导航继续进行，完成后执行 `then` 回调（如果有）
   - 如果返回 `Block`：导航被阻止
     - `Block.stop()`：重置重定向历史
     - `Block.then()`：保留历史并执行回调
4. **错误处理**：`then` 回调中的错误会被捕获并报告，但不会影响已提交的导航

### 重定向历史管理

重定向历史用于检测循环重定向：

- **Block.stop()**：清空历史，下次尝试重新开始
- **Block.then()**：保留历史，用于检测重定向循环

这种设计允许在链式重定向中检测无限循环，同时为简单场景提供重置机制。

## 使用示例

### 示例 1：简单的权限检查

```dart
onEnter: (context, current, next, router) {
  if (!UserService.isLoggedIn) {
    return const Block.stop(); // 阻止未登录用户访问
  }
  return const Allow();
}
```

### 示例 2：阻止导航并重定向

```dart
onEnter: (context, current, next, router) {
  if (!UserService.isLoggedIn) {
    return Block.then(() {
      router.go('/login'); // 重定向到登录页
    });
  }
  return const Allow();
}
```

### 示例 3：允许导航并执行后续操作

```dart
onEnter: (context, current, next, router) {
  return Allow(
    then: () {
      // 导航完成后记录分析数据
      Analytics.trackPageView(next.uri);
      // 发送通知
      NotificationService.notifyPageEntered(next.uri);
    },
  );
}
```

### 示例 4：处理深度链接

```dart
onEnter: (context, current, next, router) async {
  if (next.uri.path == '/referral') {
    final code = next.uri.queryParameters['code'];
    if (code != null) {
      // 在后台处理推荐码，不显示页面
      await _processReferralCode(code);
      return const Block.stop(); // 阻止显示 /referral 页面
    }
  }
  return const Allow();
}
```

## 设计模式

### 密封类模式

使用 `sealed class` 确保类型安全：

- 编译时检查：确保所有可能的情况都被处理
- 模式匹配：可以与 `switch` 表达式配合使用
- 防止扩展：确保只有 `Allow` 和 `Block` 两种结果

### 策略模式

`OnEnterResult` 及其子类实现了策略模式：

- **Allow**：允许策略
- **Block.stop()**：硬停止策略
- **Block.then()**：链式阻止策略

### 回调模式

通过 `then` 字段实现回调模式，允许在决策后执行额外操作。

## 注意事项

1. **错误处理**：`then` 回调中的错误不会撤销导航，只会被报告
2. **异步支持**：`OnEnterThenCallback` 支持异步操作
3. **历史管理**：理解 `Block.stop()` 和 `Block.then()` 对重定向历史的不同影响
4. **性能考虑**：`then` 回调不应阻塞 UI，长时间操作应使用 `unawaited` 或后台处理
5. **类型安全**：使用密封类确保编译时类型检查

## 相关类型

- **OnEnter**：`onEnter` 回调的函数签名，定义在 `router.dart` 中
- **GoRouterState**：导航状态，包含当前和目标路由信息
- **GoRouter**：路由器实例，用于执行重定向等操作

## 总结

`on_enter.dart` 提供了一个类型安全、灵活的导航拦截系统。通过 `Allow` 和 `Block` 两种结果类型，以及可选的 `then` 回调机制，开发者可以精确控制路由导航行为，实现权限检查、重定向、日志记录等功能。密封类的设计确保了类型安全，而清晰的重定向历史管理机制则防止了循环重定向问题。
