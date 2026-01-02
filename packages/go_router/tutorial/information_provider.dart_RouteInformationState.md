# RouteInformationState 类详解

## 概述

`RouteInformationState<T>` 是 go_router 包中的一个数据类，用于存储在 `RouteInformation.state` 中的状态信息。该类主要用于 `GoRouteInformationParser` 处理路由导航时携带必要的导航元数据。

**重要特性**：

- 这是一个内部类，只在 go_router 包内部使用，**不会被发送到 Flutter 引擎**
- 使用泛型 `<T>` 支持类型化的导航返回值
- 通过 `NavigatingType` 枚举区分不同的导航类型

## 类定义

```dart 39:44:lib/src/information_provider.dart
/// The data class to be stored in [RouteInformation.state] to be used by
/// [GoRouteInformationParser].
///
/// This state class is used internally in go_router and will not be sent to
/// the engine.
class RouteInformationState<T> {
```

该类是一个泛型类，类型参数 `T` 表示导航操作可能返回的数据类型（通常用于 `push`、`pushReplacement`、`replace` 等操作的返回值）。

## 构造函数

### 主构造函数

```dart 45:56:lib/src/information_provider.dart
  /// Creates an InternalRouteInformationState.
  @visibleForTesting
  RouteInformationState({
    this.extra,
    this.completer,
    this.baseRouteMatchList,
    required this.type,
  }) : assert(
         (type == NavigatingType.go || type == NavigatingType.restore) ==
             (completer == null),
       ),
       assert((type != NavigatingType.go) == (baseRouteMatchList != null));
```

构造函数带有 `@visibleForTesting` 注解，表示主要用于测试。它包含以下参数：

- **`extra`**：可选的额外对象，用于在导航时传递额外数据
- **`completer`**：可选的 `Completer<T?>`，用于异步导航操作的完成回调
- **`baseRouteMatchList`**：可选的 `RouteMatchList`，表示基础路由匹配列表
- **`type`**：必需的 `NavigatingType`，表示导航类型

**断言约束**：

1. 第一个断言：当 `type` 为 `NavigatingType.go` 或 `NavigatingType.restore` 时，`completer` 必须为 `null`（因为这两种导航类型不需要返回值）
2. 第二个断言：当 `type` 不是 `NavigatingType.go` 时，`baseRouteMatchList` 不能为 `null`（因为其他导航类型都需要基于现有的路由栈进行操作）

### 工厂构造函数

类提供了两个静态工厂构造函数，用于创建特定类型的 `RouteInformationState` 实例：

#### go 工厂构造函数

```dart 76:78:lib/src/information_provider.dart
  /// Factory constructor for 'go' navigation type.
  static RouteInformationState<void> go({Object? extra}) =>
      RouteInformationState<void>(extra: extra, type: NavigatingType.go);
```

用于创建 `go` 类型的导航状态。`go` 导航会替换整个路由栈，因此：

- 返回类型固定为 `RouteInformationState<void>`（因为不需要返回值）
- 只需要可选的 `extra` 参数
- `baseRouteMatchList` 和 `completer` 自动设为 `null`

#### restore 工厂构造函数

```dart 80:88:lib/src/information_provider.dart
  /// Factory constructor for 'restore' navigation type.
  static RouteInformationState<void> restore({
    required RouteMatchList base,
    Object? extra,
  }) => RouteInformationState<void>(
    extra: extra ?? base.extra,
    baseRouteMatchList: base,
    type: NavigatingType.restore,
  );
```

用于创建 `restore` 类型的导航状态。`restore` 导航用于恢复之前的路由状态，通常用于浏览器前进/后退操作：

- 返回类型固定为 `RouteInformationState<void>`
- 需要 `base` 参数（`RouteMatchList`），表示要恢复的基础路由匹配列表
- `extra` 参数可选，如果未提供则使用 `base.extra`
- `completer` 自动设为 `null`

## 字段说明

### extra

```dart 58:59:lib/src/information_provider.dart
  /// The extra object used when navigating with [GoRouter].
  final Object? extra;
```

用于在导航时传递额外数据的对象。可以是任何类型的对象，通常用于传递路由参数以外的数据。

### completer

```dart 61:66:lib/src/information_provider.dart
  /// The completer that needs to be completed when the newly added route is
  /// popped off the screen.
  ///
  /// This is only null if [type] is [NavigatingType.go] or
  /// [NavigatingType.restore].
  final Completer<T?>? completer;
```

用于异步导航操作的 `Completer`。当新添加的路由被弹出时，这个 `Completer` 会被完成，返回值会传递给调用者。

**使用场景**：

- 仅在 `push`、`pushReplacement`、`replace` 类型的导航中使用
- `go` 和 `restore` 类型导航中为 `null`（因为这两种导航不需要返回值）

### baseRouteMatchList

```dart 68:71:lib/src/information_provider.dart
  /// The base route match list to push on top to.
  ///
  /// This is only null if [type] is [NavigatingType.go].
  final RouteMatchList? baseRouteMatchList;
```

基础路由匹配列表，新路由会被推送到这个列表之上。

**使用场景**：

- `go` 类型导航中为 `null`（因为 `go` 会替换整个路由栈）
- 其他导航类型（`push`、`pushReplacement`、`replace`、`restore`）都需要这个字段

### type

```dart 73:74:lib/src/information_provider.dart
  /// The type of navigation.
  final NavigatingType type;
```

导航类型，用于区分不同的导航操作。`NavigatingType` 枚举包含以下值：

- `push`：在基础路由栈顶部推送新路由
- `pushReplacement`：移除基础路由栈最顶部的路由，然后推送新路由
- `replace`：替换基础路由栈最顶部的路由
- `go`：替换整个路由栈
- `restore`：恢复之前的路由状态

## 使用示例

### 在 GoRouteInformationParser 中的使用

`RouteInformationState` 主要在 `GoRouteInformationParser` 中使用，用于处理不同的导航场景：

```dart 90:105:lib/src/parser.dart
    if (raw == null) {
      // Framework/browser provided no state — synthesize a standard "go" nav.
      // This happens on initial app load and some framework calls.
      infoState = RouteInformationState.go();
      incomingUri = routeInformation.uri;
    } else if (raw is! RouteInformationState) {
      // Restoration/back-forward: decode the stored match list and treat as restore.
      final RouteMatchList decoded = _routeMatchListCodec.decode(
        raw as Map<Object?, Object?>,
      );
      infoState = RouteInformationState.restore(base: decoded);
      incomingUri = decoded.uri;
    } else {
      infoState = raw;
      incomingUri = routeInformation.uri;
    }
```

### 在 GoRouteInformationProvider 中的使用

`GoRouteInformationProvider` 使用 `RouteInformationState` 来创建不同类型的导航状态：

```dart 182:198:lib/src/information_provider.dart
  /// Pushes the `location` as a new route on top of `base`.
  Future<T?> push<T>(
    String location, {
    required RouteMatchList base,
    Object? extra,
  }) {
    final completer = Completer<T?>();
    _setValue(
      location,
      RouteInformationState<T>(
        extra: extra,
        baseRouteMatchList: base,
        completer: completer,
        type: NavigatingType.push,
      ),
    );
    return completer.future;
  }
```

```dart 200:206:lib/src/information_provider.dart
  /// Replace the current route matches with the `location`.
  void go(String location, {Object? extra}) {
    _setValue(
      location,
      RouteInformationState<void>(extra: extra, type: NavigatingType.go),
    );
  }
```

## 字段关系总结

不同导航类型下的字段值关系：

| 导航类型 | `completer` | `baseRouteMatchList` | `extra` |
| --- | --- | --- | --- |
| `go` | `null` | `null` | 可选 |
| `restore` | `null` | 必需 | 可选（默认使用 `base.extra`） |
| `push` | 必需 | 必需 | 可选 |
| `pushReplacement` | 必需 | 必需 | 可选 |
| `replace` | 必需 | 必需 | 可选 |

## 设计要点

1. **内部使用**：这个类只在 go_router 包内部使用，不会暴露给外部用户，也不会发送到 Flutter 引擎
2. **类型安全**：通过泛型 `T` 提供类型安全的导航返回值
3. **状态约束**：通过构造函数断言确保不同导航类型下字段值的正确性
4. **工厂方法**：提供便捷的工厂构造函数，简化常用场景的创建过程
5. **异步支持**：通过 `Completer` 支持异步导航操作和返回值传递

## 相关类型

- **`NavigatingType`**：导航类型枚举，定义在同一个文件中
- **`RouteMatchList`**：路由匹配列表，用于表示路由栈
- **`GoRouteInformationParser`**：路由解析器，使用 `RouteInformationState` 进行路由解析
- **`GoRouteInformationProvider`**：路由信息提供者，创建和管理 `RouteInformationState` 实例
