# extensions.dart 文件说明

## 概述

`extensions.dart` 文件定义了一个 Dart 扩展（extension），为 Flutter 的 `BuildContext` 对象添加了便捷的导航方法。这个扩展使得开发者可以直接通过 `context` 对象调用 GoRouter 的导航功能，而无需手动获取 `GoRouter` 实例。

## 扩展定义

```dart 11:11:lib/src/misc/extensions.dart
extension GoRouterHelper on BuildContext {
```

这个扩展名为 `GoRouterHelper`，它扩展了 `BuildContext` 类型，为所有 `BuildContext` 实例添加了导航相关的方法。

## 核心功能

### 1. 命名路由位置获取

```dart 13:23:lib/src/misc/extensions.dart
  /// Get a location from route name and parameters.
  String namedLocation(
    String name, {
    Map<String, String> pathParameters = const <String, String>{},
    Map<String, dynamic> queryParameters = const <String, dynamic>{},
    String? fragment,
  }) => GoRouter.of(this).namedLocation(
    name,
    pathParameters: pathParameters,
    queryParameters: queryParameters,
    fragment: fragment,
  );
```

**功能说明**：根据路由名称和参数生成对应的 URL 位置字符串。

**参数**：

- `name`：路由名称
- `pathParameters`：路径参数（如 `/user/:id` 中的 `id`）
- `queryParameters`：查询参数（URL 中的 `?key=value` 部分）
- `fragment`：URL 片段（`#` 后面的部分）

**使用示例**：

```dart
String location = context.namedLocation(
  'user',
  pathParameters: {'id': '123'},
  queryParameters: {'tab': 'profile'},
  fragment: 'settings',
);
```

### 2. 导航到指定位置

```dart 25:27:lib/src/misc/extensions.dart
  /// Navigate to a location.
  void go(String location, {Object? extra}) =>
      GoRouter.of(this).go(location, extra: extra);
```

**功能说明**：导航到指定的 URL 位置。这是最常用的导航方法，会替换当前的导航栈。

**参数**：

- `location`：目标 URL 路径
- `extra`：可选的额外数据对象

**使用示例**：

```dart
context.go('/home');
context.go('/user/123', extra: {'from': 'login'});
```

### 3. 导航到命名路由

```dart 29:42:lib/src/misc/extensions.dart
  /// Navigate to a named route.
  void goNamed(
    String name, {
    Map<String, String> pathParameters = const <String, String>{},
    Map<String, dynamic> queryParameters = const <String, dynamic>{},
    Object? extra,
    String? fragment,
  }) => GoRouter.of(this).goNamed(
    name,
    pathParameters: pathParameters,
    queryParameters: queryParameters,
    extra: extra,
    fragment: fragment,
  );
```

**功能说明**：通过路由名称进行导航，支持路径参数、查询参数和片段。

**使用示例**：

```dart
context.goNamed(
  'user',
  pathParameters: {'id': '123'},
  queryParameters: {'tab': 'profile'},
  extra: {'source': 'dashboard'},
  fragment: 'settings',
);
```

### 4. 推入新页面到导航栈

```dart 44:53:lib/src/misc/extensions.dart
  /// Push a location onto the page stack.
  ///
  /// See also:
  /// * [pushReplacement] which replaces the top-most page of the page stack and
  ///   always uses a new page key.
  /// * [replace] which replaces the top-most page of the page stack but treats
  ///   it as the same page. The page key will be reused. This will preserve the
  ///   state and not run any page animation.
  Future<T?> push<T extends Object?>(String location, {Object? extra}) =>
      GoRouter.of(this).push<T>(location, extra: extra);
```

**功能说明**：将新页面推入导航栈，保留当前页面在栈中。用户可以返回上一页。

**返回值**：返回一个 `Future<T?>`，当新页面被关闭时，可以通过 `pop` 方法传递返回值。

**与 `go` 的区别**：

- `go`：替换整个导航栈，无法返回
- `push`：推入新页面，可以返回

**使用示例**：

```dart
final result = await context.push<String>('/details');
if (result != null) {
  print('返回结果: $result');
}
```

### 5. 推入命名路由到导航栈

```dart 55:66:lib/src/misc/extensions.dart
  /// Navigate to a named route onto the page stack.
  Future<T?> pushNamed<T extends Object?>(
    String name, {
    Map<String, String> pathParameters = const <String, String>{},
    Map<String, dynamic> queryParameters = const <String, dynamic>{},
    Object? extra,
  }) => GoRouter.of(this).pushNamed<T>(
    name,
    pathParameters: pathParameters,
    queryParameters: queryParameters,
    extra: extra,
  );
```

**功能说明**：通过路由名称推入新页面到导航栈。

**使用示例**：

```dart
final result = await context.pushNamed<String>(
  'details',
  pathParameters: {'id': '123'},
  queryParameters: {'mode': 'edit'},
);
```

### 6. 检查是否可以返回

```dart 68:69:lib/src/misc/extensions.dart
  /// Returns `true` if there is more than 1 page on the stack.
  bool canPop() => GoRouter.of(this).canPop();
```

**功能说明**：检查导航栈中是否有多于一个页面，即是否可以执行返回操作。

**使用示例**：

```dart
if (context.canPop()) {
  context.pop();
} else {
  // 处理无法返回的情况
}
```

### 7. 返回上一页

```dart 71:73:lib/src/misc/extensions.dart
  /// Pop the top page off the Navigator's page stack by calling
  /// [Navigator.pop].
  void pop<T extends Object?>([T? result]) => GoRouter.of(this).pop(result);
```

**功能说明**：关闭当前页面并返回上一页，可以传递一个返回值给调用者。

**参数**：

- `result`：可选的返回值，会传递给 `push` 或 `pushNamed` 返回的 `Future`

**使用示例**：

```dart
// 返回上一页，不传递数据
context.pop();

// 返回上一页，传递数据
context.pop('操作成功');
```

### 8. 替换栈顶页面（新页面键）

```dart 75:85:lib/src/misc/extensions.dart
  /// Replaces the top-most page of the page stack with the given URL location
  /// w/ optional query parameters, e.g. `/family/f2/person/p1?color=blue`.
  ///
  /// See also:
  /// * [go] which navigates to the location.
  /// * [push] which pushes the given location onto the page stack.
  /// * [replace] which replaces the top-most page of the page stack but treats
  ///   it as the same page. The page key will be reused. This will preserve the
  ///   state and not run any page animation.
  void pushReplacement(String location, {Object? extra}) =>
      GoRouter.of(this).pushReplacement(location, extra: extra);
```

**功能说明**：替换导航栈顶部的页面，使用新的页面键。这意味着页面状态会被重置，并且会播放页面动画。

**与 `replace` 的区别**：

- `pushReplacement`：使用新页面键，重置状态，有动画
- `replace`：重用页面键，保留状态，无动画

**使用示例**：

```dart
context.pushReplacement('/login');
```

### 9. 替换栈顶页面（命名路由，新页面键）

```dart 87:104:lib/src/misc/extensions.dart
  /// Replaces the top-most page of the page stack with the named route w/
  /// optional parameters, e.g. `name='person', pathParameters={'fid': 'f2', 'pid':
  /// 'p1'}`.
  ///
  /// See also:
  /// * [goNamed] which navigates a named route.
  /// * [pushNamed] which pushes a named route onto the page stack.
  void pushReplacementNamed(
    String name, {
    Map<String, String> pathParameters = const <String, String>{},
    Map<String, dynamic> queryParameters = const <String, dynamic>{},
    Object? extra,
  }) => GoRouter.of(this).pushReplacementNamed(
    name,
    pathParameters: pathParameters,
    queryParameters: queryParameters,
    extra: extra,
  );
```

**功能说明**：通过路由名称替换栈顶页面，使用新的页面键。

**使用示例**：

```dart
context.pushReplacementNamed(
  'home',
  queryParameters: {'tab': 'dashboard'},
);
```

### 10. 替换栈顶页面（保留页面键）

```dart 106:117:lib/src/misc/extensions.dart
  /// Replaces the top-most page of the page stack with the given one but treats
  /// it as the same page.
  ///
  /// The page key will be reused. This will preserve the state and not run any
  /// page animation.
  ///
  /// See also:
  /// * [push] which pushes the given location onto the page stack.
  /// * [pushReplacement] which replaces the top-most page of the page stack but
  ///   always uses a new page key.
  void replace(String location, {Object? extra}) =>
      GoRouter.of(this).replace<Object?>(location, extra: extra);
```

**功能说明**：替换栈顶页面，但将其视为同一页面。页面键会被重用，这意味着页面状态会被保留，并且不会播放页面动画。

**适用场景**：

- 需要更新 URL 但保持页面状态
- 需要避免页面动画
- 需要保持滚动位置等 UI 状态

**使用示例**：

```dart
context.replace('/user/123?tab=profile');
```

### 11. 替换栈顶页面（命名路由，保留页面键）

```dart 119:140:lib/src/misc/extensions.dart
  /// Replaces the top-most page with the named route and optional parameters,
  /// preserving the page key.
  ///
  /// This will preserve the state and not run any page animation. Optional
  /// parameters can be provided to the named route, e.g. `name='person',
  /// pathParameters={'fid': 'f2', 'pid': 'p1'}`.
  ///
  /// See also:
  /// * [pushNamed] which pushes the given location onto the page stack.
  /// * [pushReplacementNamed] which replaces the top-most page of the page
  ///   stack but always uses a new page key.
  void replaceNamed(
    String name, {
    Map<String, String> pathParameters = const <String, String>{},
    Map<String, dynamic> queryParameters = const <String, dynamic>{},
    Object? extra,
  }) => GoRouter.of(this).replaceNamed<Object?>(
    name,
    pathParameters: pathParameters,
    queryParameters: queryParameters,
    extra: extra,
  );
```

**功能说明**：通过路由名称替换栈顶页面，保留页面键和状态。

**使用示例**：

```dart
context.replaceNamed(
  'user',
  pathParameters: {'id': '123'},
  queryParameters: {'tab': 'settings'},
);
```

## 导航方法对比

### 导航方式对比

| 方法 | 是否替换栈 | 是否保留状态 | 是否有动画 | 是否可返回 |
| --- | --- | --- | --- | --- |
| `go` | 是 | 否 | 是 | 否 |
| `push` | 否 | 是 | 是 | 是 |
| `pushReplacement` | 否（替换栈顶） | 否 | 是 | 是 |
| `replace` | 否（替换栈顶） | 是 | 否 | 是 |

### 命名路由 vs 路径导航

- **路径导航**（`go`、`push`、`replace`）：直接使用 URL 路径，如 `/user/123`
- **命名路由**（`goNamed`、`pushNamed`、`replaceNamed`）：使用路由名称，更灵活，支持参数化

## 实现原理

所有扩展方法都是通过 `GoRouter.of(this)` 获取当前 `BuildContext` 对应的 `GoRouter` 实例，然后调用相应的方法。这种设计模式：

1. **简化 API**：开发者无需手动获取 `GoRouter` 实例
2. **类型安全**：通过 `BuildContext` 确保在正确的上下文中使用
3. **一致性**：所有导航方法都通过相同的扩展访问

## 使用建议

1. **简单导航**：使用 `context.go()` 或 `context.goNamed()`
2. **需要返回**：使用 `context.push()` 或 `context.pushNamed()`
3. **需要保留状态**：使用 `context.replace()` 或 `context.replaceNamed()`
4. **需要重置状态**：使用 `context.pushReplacement()` 或 `context.pushReplacementNamed()`
5. **检查返回能力**：在调用 `pop()` 前使用 `canPop()` 检查

## 注意事项

1. 所有方法都需要在有效的 `BuildContext` 中使用，且该上下文必须在 `GoRouter` 的 widget 树中
2. `push` 和 `pushNamed` 返回 `Future`，可以用于等待页面返回的结果
3. `replace` 和 `replaceNamed` 会保留页面状态，适合用于更新 URL 但保持 UI 状态的场景
4. `pushReplacement` 和 `pushReplacementNamed` 会重置页面状态，适合用于登录后跳转等场景
