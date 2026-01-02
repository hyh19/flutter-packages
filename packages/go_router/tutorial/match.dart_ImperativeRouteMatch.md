# ImperativeRouteMatch 类详解

## 概述

`ImperativeRouteMatch` 是 go_router 中用于表示通过命令式导航（imperative navigation）推送的路由匹配结果的类。它继承自 `RouteMatch`，专门用于处理通过 `GoRouter.push`、`GoRouter.pushReplacement` 或 `GoRouter.replace` 等命令式 API 创建的路由。

与声明式路由（declarative routes）不同，命令式路由：

- 不会被添加到浏览器历史记录中（在 Web 平台）
- 不是深度链接的一部分
- 可以通过返回值向调用者传递结果
- 可以通过 `Navigator.pop` 返回结果并关闭

```dart 451:511:lib/src/match.dart
/// The route match that represent route pushed through [GoRouter.push].
class ImperativeRouteMatch extends RouteMatch {
  /// Constructor for [ImperativeRouteMatch].
  ImperativeRouteMatch({
    required super.pageKey,
    required this.matches,
    required this.completer,
  }) : super(
         route: _getsLastRouteFromMatches(matches),
         matchedLocation: _getsMatchedLocationFromMatches(matches),
       );

  static GoRoute _getsLastRouteFromMatches(RouteMatchList matchList) {
    if (matchList.isError) {
      return GoRoute(
        path: 'error',
        builder: (_, __) => throw UnimplementedError(),
      );
    }
    return matchList.last.route;
  }

  static String _getsMatchedLocationFromMatches(RouteMatchList matchList) {
    if (matchList.isError) {
      return matchList.uri.toString();
    }
    return matchList.last.matchedLocation;
  }

  /// The matches that produces this route match.
  final RouteMatchList matches;

  /// The completer for the future returned by [GoRouter.push].
  final Completer<Object?> completer;

  /// Called when the corresponding [Route] associated with this route match is
  /// completed.
  void complete([dynamic value]) {
    completer.complete(value);
  }

  @override
  GoRouterState buildState(
    RouteConfiguration configuration,
    RouteMatchList matches,
  ) {
    return super.buildState(configuration, this.matches);
  }

  @override
  bool operator ==(Object other) {
    return other is ImperativeRouteMatch &&
        completer == other.completer &&
        matches == other.matches &&
        super == other;
  }

  @override
  int get hashCode => Object.hash(super.hashCode, completer, matches.hashCode);
}
```

## 类层次结构

`ImperativeRouteMatch` 的继承关系：

```text
RouteMatchBase (抽象基类)
  └── RouteMatch (不可变的声明式路由匹配)
      └── ImperativeRouteMatch (命令式路由匹配)
```

## 核心属性

### matches

```dart 481:481:lib/src/match.dart
  final RouteMatchList matches;
```

`matches` 是一个 `RouteMatchList` 对象，包含了产生此路由匹配的完整匹配列表。这个列表记录了从根路由到目标路由的完整路径，包括所有中间的路由匹配。

这个属性在以下场景中很重要：

- 构建路由状态时需要完整的匹配上下文
- 错误处理时需要访问完整的 URI 信息
- 路由历史管理需要知道完整的匹配链

### completer

```dart 483:484:lib/src/match.dart
  /// The completer for the future returned by [GoRouter.push].
  final Completer<Object?> completer;
```

`completer` 是一个 `Completer<Object?>` 对象，用于处理异步导航的结果。当调用 `GoRouter.push` 时，会返回一个 `Future`，这个 `Future` 由 `completer` 控制完成。

使用场景：

- 当用户通过 `Navigator.pop(context, result)` 返回时，`result` 值会通过 `complete()` 方法传递给 `completer`
- 调用者可以等待路由完成并获得返回值

示例：

```dart
// 在路由 A 中
final result = await context.push('/route-b');
print('返回的值: $result'); // 如果 route-b 通过 pop(context, 'hello') 返回，这里会打印 'hello'

// 在路由 B 中
Navigator.of(context).pop('hello'); // 返回值给调用者
```

## 构造函数

```dart 454:461:lib/src/match.dart
  ImperativeRouteMatch({
    required super.pageKey,
    required this.matches,
    required this.completer,
  }) : super(
         route: _getsLastRouteFromMatches(matches),
         matchedLocation: _getsMatchedLocationFromMatches(matches),
       );
```

构造函数接收三个必需参数：

1. **`pageKey`**：通过 `super.pageKey` 传递给父类，用于唯一标识路由页面
2. **`matches`**：产生此路由匹配的完整匹配列表
3. **`completer`**：用于完成异步导航的 completer

在初始化列表中，通过调用私有静态方法从 `matches` 中提取路由和匹配位置，然后传递给父类构造函数。

## 辅助方法

### _getsLastRouteFromMatches

```dart 463:471:lib/src/match.dart
  static GoRoute _getsLastRouteFromMatches(RouteMatchList matchList) {
    if (matchList.isError) {
      return GoRoute(
        path: 'error',
        builder: (_, __) => throw UnimplementedError(),
      );
    }
    return matchList.last.route;
  }
```

这个私有静态方法从匹配列表中提取最后一个路由（即目标路由）。如果匹配过程中出现错误（`matchList.isError` 为 `true`），则返回一个占位符错误路由。

**错误处理逻辑**：

- 当路由匹配失败时，创建一个临时的错误路由
- 这个错误路由的 `builder` 会抛出 `UnimplementedError`，在实际构建时会触发错误处理流程

### _getsMatchedLocationFromMatches

```dart 473:478:lib/src/match.dart
  static String _getsMatchedLocationFromMatches(RouteMatchList matchList) {
    if (matchList.isError) {
      return matchList.uri.toString();
    }
    return matchList.last.matchedLocation;
  }
```

这个私有静态方法从匹配列表中提取匹配位置字符串。如果匹配失败，返回完整的 URI 字符串作为匹配位置。

**匹配位置的作用**：

- 匹配位置是路由匹配后得到的实际路径（可能包含路径参数的值）
- 例如，如果路由定义为 `/user/:id`，URI 为 `/user/123`，则匹配位置为 `/user/123`
- 这个位置用于构建路由状态和页面导航

## 核心方法

### complete

```dart 486:490:lib/src/match.dart
  /// Called when the corresponding [Route] associated with this route match is
  /// completed.
  void complete([dynamic value]) {
    completer.complete(value);
  }
```

`complete` 方法在对应的路由完成时被调用，用于完成异步导航的 `Future`。当用户通过 `Navigator.pop` 返回时，go_router 会调用此方法，并将返回值传递给调用者。

**参数**：

- `value`：可选的值，作为路由的返回结果。如果路由通过 `Navigator.pop(context, result)` 返回，`result` 会被传递到这里。

**使用流程**：

1. 用户调用 `context.push('/some-route')`，返回一个 `Future`
2. go_router 创建一个 `ImperativeRouteMatch`，其中包含一个 `Completer`
3. 当目标路由通过 `Navigator.pop(context, result)` 返回时
4. go_router 调用 `imperativeRouteMatch.complete(result)`
5. `Future` 完成，调用者收到 `result` 值

### buildState

```dart 492:498:lib/src/match.dart
  @override
  GoRouterState buildState(
    RouteConfiguration configuration,
    RouteMatchList matches,
  ) {
    return super.buildState(configuration, this.matches);
  }
```

`buildState` 方法用于构建路由状态对象。它重写了父类的方法，但有一个关键区别：**它使用自身的 `this.matches` 而不是方法参数中的 `matches`**。

**设计原因**：

- `ImperativeRouteMatch` 需要保持自己的完整匹配上下文（`this.matches`）
- 传入的 `matches` 参数可能不包含命令式路由的完整信息
- 使用 `this.matches` 确保状态构建时使用的是正确的匹配链

这个设计确保了：

- 命令式路由能够正确构建状态
- 路径参数、查询参数等信息能够正确传递
- 路由历史记录的完整性

## 相等性和哈希码

```dart 500:509:lib/src/match.dart
  @override
  bool operator ==(Object other) {
    return other is ImperativeRouteMatch &&
        completer == other.completer &&
        matches == other.matches &&
        super == other;
  }

  @override
  int get hashCode => Object.hash(super.hashCode, completer, matches.hashCode);
}
```

`ImperativeRouteMatch` 重写了 `==` 运算符和 `hashCode` 方法，确保：

- 相同的 `completer`、`matches` 和父类属性被视为相等
- 哈希码计算包含所有相关属性
- 支持在集合中使用（如 `Set`、`Map`）

**注意**：由于 `Completer` 的相等性比较，两个不同的 `ImperativeRouteMatch` 实例即使其他属性相同，也不会被视为相等（除非是同一个 `completer` 实例）。这是预期的行为，因为每个命令式导航都应该有唯一的 completer。

## 使用场景

### 1. GoRouter.push

当调用 `GoRouter.push` 时，会创建一个 `ImperativeRouteMatch`：

```dart
// 在 parser.dart 中的实现
case NavigatingType.push:
  return baseRouteMatchList!.push(
    ImperativeRouteMatch(
      pageKey: _getUniqueValueKey(),
      completer: completer!,
      matches: newMatchList,
    ),
  );
```

### 2. GoRouter.pushReplacement

`pushReplacement` 也会创建 `ImperativeRouteMatch`，但会移除当前最后一个路由匹配：

```dart
case NavigatingType.pushReplacement:
  final RouteMatch routeMatch = baseRouteMatchList!.last;
  baseRouteMatchList = baseRouteMatchList.remove(routeMatch);
  // ... 然后创建新的 ImperativeRouteMatch
```

### 3. GoRouter.replace

`replace` 类似于 `pushReplacement`，但保持相同的 `pageKey`：

```dart
case NavigatingType.replace:
  final RouteMatch routeMatch = baseRouteMatchList!.last;
  baseRouteMatchList = baseRouteMatchList.remove(routeMatch);
  return baseRouteMatchList.push(
    ImperativeRouteMatch(
      pageKey: routeMatch.pageKey, // 保持相同的 pageKey
      completer: completer!,
      matches: newMatchList,
    ),
  );
```

## 与其他路由类型的区别

### ImperativeRouteMatch vs RouteMatch

| 特性 | RouteMatch | ImperativeRouteMatch |
| --- | ---------- | -------------------- |
| 创建方式 | 声明式（通过 `go()` 或 URL 匹配） | 命令式（通过 `push()`） |
| 浏览器历史 | 是（在 Web 平台） | 否 |
| 深度链接 | 是 | 否 |
| 返回值 | 否 | 是（通过 Future） |
| matches 属性 | 无（依赖传入的参数） | 有（保持自己的完整匹配链） |
| completer | 无 | 有（用于异步结果） |

### 在 RouteMatchList 中的特殊处理

在 `RouteMatchList` 的 `_generateFullPath` 方法中，`ImperativeRouteMatch` 被特殊处理：

```dart
static String _generateFullPath(Iterable<RouteMatchBase> matches) {
  // ...
  for (final RouteMatchBase match in matches.where(
    (RouteMatchBase match) => match is! ImperativeRouteMatch,
  )) {
    // 忽略 ImperativeRouteMatch，因为它们不贡献路径
  }
}
```

这是因为命令式路由不参与 URL 路径的构建，它们存在于导航栈中但不影响深度链接。

## 总结

`ImperativeRouteMatch` 是 go_router 中处理命令式导航的关键类，它：

1. **继承自 `RouteMatch`**：复用声明式路由的基础功能
2. **包含完整的匹配链**：通过 `matches` 属性保存完整的路由匹配上下文
3. **支持异步结果**：通过 `completer` 实现路由返回值的传递
4. **特殊的状态构建**：重写 `buildState` 以使用自身的匹配链
5. **不参与深度链接**：命令式路由不会出现在 URL 中，也不影响浏览器历史

理解 `ImperativeRouteMatch` 有助于深入理解 go_router 中命令式导航的实现机制，以及如何处理路由返回值和异步导航流程。
