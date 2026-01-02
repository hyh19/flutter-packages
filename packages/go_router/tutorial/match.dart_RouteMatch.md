# RouteMatch 类说明

## 概述

`RouteMatch` 是 go_router 包中用于表示路由匹配结果的核心类。它表示一个 `GoRoute` 与某个位置（location）匹配成功后的结果。这个类是不可变的（immutable），继承自抽象基类 `RouteMatchBase`。

## 类定义

```dart 305:360:lib/src/match.dart
/// An matched result by matching a [GoRoute] against a location.
///
/// This is typically created by calling [RouteMatchBase.match].
@immutable
class RouteMatch extends RouteMatchBase {
  /// Constructor for [RouteMatch].
  const RouteMatch({
    required this.route,
    required this.matchedLocation,
    required this.pageKey,
  });

  /// The matched route.
  @override
  final GoRoute route;

  @override
  final String matchedLocation;

  @override
  final ValueKey<String> pageKey;

  @override
  bool operator ==(Object other) {
    if (other.runtimeType != runtimeType) {
      return false;
    }
    return other is RouteMatch &&
        route == other.route &&
        matchedLocation == other.matchedLocation &&
        pageKey == other.pageKey;
  }

  @override
  int get hashCode => Object.hash(route, matchedLocation, pageKey);

  @override
  GoRouterState buildState(
    RouteConfiguration configuration,
    RouteMatchList matches,
  ) {
    return GoRouterState(
      configuration,
      uri: matches.uri,
      matchedLocation: matchedLocation,
      fullPath: matches.fullPath,
      pathParameters: matches.pathParameters,
      pageKey: pageKey,
      name: route.name,
      path: route.path,
      extra: matches.extra,
      topRoute: matches.lastOrNull?.route,
    );
  }
}
```

## 主要特性

### 1. 不可变性

`RouteMatch` 使用 `@immutable` 注解标记，表示这是一个不可变类。所有字段都是 `final` 的，一旦创建就不能修改。这种设计有助于：

- 线程安全
- 简化状态管理
- 支持值相等性比较

### 2. 继承关系

`RouteMatch` 继承自 `RouteMatchBase`，这是所有路由匹配结果的基类。`RouteMatchBase` 定义了路由匹配的通用接口，包括：

- `route`：匹配的路由（抽象属性）
- `matchedLocation`：匹配的位置字符串（抽象属性）
- `pageKey`：页面键（抽象属性）
- `buildState`：构建路由状态的方法（抽象方法）

## 字段说明

### route

```dart 317:319:lib/src/match.dart
/// The matched route.
@override
final GoRoute route;
```

表示当前匹配到的 `GoRoute` 对象。这是路由配置中定义的原始路由实例，包含了路由的路径、名称、构建器等信息。

### matchedLocation

```dart 321:322:lib/src/match.dart
@override
final String matchedLocation;
```

表示匹配到的位置字符串。这是实际匹配的 URL 路径部分，可能与完整的 URI 不同。

**示例**：

假设 URI 是 `/family/f2/person/p2`，而路由是 `GoRoute('/family/:id')`，那么 `matchedLocation` 就是 `/family/f2`。

### pageKey

```dart 324:325:lib/src/match.dart
@override
final ValueKey<String> pageKey;
```

用于标识页面的唯一键。这是一个 `ValueKey<String>` 对象，Flutter 使用它来区分不同的页面实例。通常由匹配的完整路径模式生成，例如 `ValueKey('/family/:fid')`。

## 方法说明

### 构造函数

```dart 311:315:lib/src/match.dart
/// Constructor for [RouteMatch].
const RouteMatch({
  required this.route,
  required this.matchedLocation,
  required this.pageKey,
});
```

使用 `const` 构造函数创建 `RouteMatch` 实例。所有三个参数都是必需的：

- `route`：匹配的 `GoRoute`
- `matchedLocation`：匹配的位置字符串
- `pageKey`：页面键

### 相等性比较

```dart 327:336:lib/src/match.dart
@override
bool operator ==(Object other) {
  if (other.runtimeType != runtimeType) {
    return false;
  }
  return other is RouteMatch &&
      route == other.route &&
      matchedLocation == other.matchedLocation &&
      pageKey == other.pageKey;
}
```

`RouteMatch` 实现了值相等性比较。两个 `RouteMatch` 实例被认为是相等的，当且仅当：

1. 它们的运行时类型相同
2. `route`、`matchedLocation` 和 `pageKey` 都相等

这种比较方式确保具有相同路由、匹配位置和页面键的 `RouteMatch` 实例被认为是同一个匹配。

### hashCode

```dart 338:339:lib/src/match.dart
@override
int get hashCode => Object.hash(route, matchedLocation, pageKey);
```

与 `==` 操作符保持一致，`hashCode` 基于 `route`、`matchedLocation` 和 `pageKey` 计算。这确保了相等的对象具有相同的哈希值，可以在哈希集合（如 `Set`、`Map`）中正确使用。

### buildState

```dart 341:358:lib/src/match.dart
@override
GoRouterState buildState(
  RouteConfiguration configuration,
  RouteMatchList matches,
) {
  return GoRouterState(
    configuration,
    uri: matches.uri,
    matchedLocation: matchedLocation,
    fullPath: matches.fullPath,
    pathParameters: matches.pathParameters,
    pageKey: pageKey,
    name: route.name,
    path: route.path,
    extra: matches.extra,
    topRoute: matches.lastOrNull?.route,
  );
}
```

`buildState` 方法用于构建 `GoRouterState` 对象，这是路由状态的核心表示。该方法：

1. 接收 `RouteConfiguration`（路由配置）和 `RouteMatchList`（匹配列表）作为参数
2. 创建一个新的 `GoRouterState` 实例，包含：
   - `uri`：完整的 URI（来自 `matches.uri`）
   - `matchedLocation`：当前匹配的位置（来自 `this.matchedLocation`）
   - `fullPath`：完整路径模式（来自 `matches.fullPath`）
   - `pathParameters`：路径参数（来自 `matches.pathParameters`）
   - `pageKey`：页面键
   - `name`：路由名称（来自 `route.name`）
   - `path`：路由路径（来自 `route.path`）
   - `extra`：额外数据（来自 `matches.extra`）
   - `topRoute`：顶层路由（来自 `matches.lastOrNull?.route`）

这个方法将匹配结果转换为可以在路由构建器中使用的状态对象。

## 创建方式

`RouteMatch` 通常不是直接创建的，而是通过调用 `RouteMatchBase.match` 静态方法生成。该方法会：

1. 根据给定的 URI 和路由配置进行匹配
2. 如果匹配成功，返回包含 `RouteMatch` 实例的列表
3. 如果匹配失败，返回空列表

在 `_matchByNavigatorKeyForGoRoute` 方法中可以看到 `RouteMatch` 的创建：

```dart 246:254:lib/src/match.dart
return <GlobalKey<NavigatorState>?, List<RouteMatchBase>>{
  parentKey: <RouteMatchBase>[
    RouteMatch(
      route: route,
      matchedLocation: newMatchedLocation,
      pageKey: ValueKey<String>(newMatchedPath),
    ),
  ],
};
```

## 使用场景

`RouteMatch` 主要用于：

1. **路由匹配结果存储**：作为路由匹配过程的输出，存储在 `RouteMatchList` 中
2. **状态构建**：通过 `buildState` 方法生成 `GoRouterState`，供路由构建器使用
3. **路由历史管理**：作为路由历史的一部分，用于导航和状态恢复

## 与其他类的关系

### RouteMatchBase

`RouteMatch` 继承自 `RouteMatchBase`，实现了基类定义的抽象接口。

### ShellRouteMatch

`ShellRouteMatch` 是 `RouteMatchBase` 的另一个子类，用于表示 `ShellRoute` 的匹配结果。与 `RouteMatch` 不同的是，`ShellRouteMatch` 包含嵌套的子匹配列表。

### ImperativeRouteMatch

`ImperativeRouteMatch` 继承自 `RouteMatch`，用于表示通过 `GoRouter.push` 推送的路由。它包含额外的信息，如 `Completer` 用于处理路由返回结果。

### RouteMatchList

`RouteMatchList` 包含多个 `RouteMatchBase` 对象（包括 `RouteMatch`），表示完整的路由匹配链。`buildState` 方法接收 `RouteMatchList` 来获取完整的匹配上下文。

## 总结

`RouteMatch` 是 go_router 中路由匹配的核心数据结构，它：

- 封装了单个 `GoRoute` 的匹配结果
- 提供了不可变的值语义
- 能够转换为可用的路由状态（`GoRouterState`）
- 支持值相等性比较，便于状态管理和去重

理解 `RouteMatch` 有助于深入理解 go_router 的路由匹配机制和状态管理方式。
