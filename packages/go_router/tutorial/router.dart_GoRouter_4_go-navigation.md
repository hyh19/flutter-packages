# GoRouter go 系列导航方法

`GoRouter` 提供了一系列用于导航的方法。本文档介绍 `go` 系列的导航方法，这些方法用于直接导航到新的路由位置，会替换当前的路由栈。

## canPop()

```dart 396:397:lib/src/router.dart
  /// Returns `true` if there is at least two or more route can be pop.
  bool canPop() => routerDelegate.canPop();
```

### 说明

检查是否至少有两个或更多路由可以被弹出。如果有至少两个路由在栈中，返回 `true`；否则返回 `false`。

### 用途

通常用于判断是否可以执行 `pop()` 操作，例如在 AppBar 中控制返回按钮的显示。

### 实现

直接委托给 `routerDelegate.canPop()` 方法。

## namedLocation()

```dart 399:411:lib/src/router.dart
  /// Get a location from route name and parameters.
  /// This is useful for redirecting to a named location.
  String namedLocation(
    String name, {
    Map<String, String> pathParameters = const <String, String>{},
    Map<String, dynamic> queryParameters = const <String, dynamic>{},
    String? fragment,
  }) => configuration.namedLocation(
    name,
    pathParameters: pathParameters,
    queryParameters: queryParameters,
    fragment: fragment,
  );
```

### 说明

根据路由名称和参数生成对应的 URI 位置字符串。这对于重定向到命名路由非常有用。

### 参数

- **`name`**（必需）：路由的名称
- **`pathParameters`**：路径参数映射（默认空 map）
- **`queryParameters`**：查询参数映射（默认空 map）
- **`fragment`**：URI 片段（可选）

### 返回值

返回生成的 URI 位置字符串。

### 用途

主要用于：

- 在重定向逻辑中生成命名路由的位置
- 在其他导航方法中构造 URI

### 实现

委托给 `configuration.namedLocation()` 方法。

## go()

```dart 413:418:lib/src/router.dart
  /// Navigate to a URI location w/ optional query parameters, e.g.
  /// `/family/f2/person/p1?color=blue`
  void go(String location, {Object? extra}) {
    log('going to $location');
    routeInformationProvider.go(location, extra: extra);
  }
```

### 说明

导航到指定的 URI 位置。这是 `GoRouter` 中最常用的导航方法之一，会替换当前的路由栈。

### 参数

- **`location`**（必需）：要导航到的 URI 位置字符串，例如 `/family/f2/person/p1?color=blue`
- **`extra`**：可选的额外数据对象，会传递给目标路由

### 行为

- **替换导航栈**：使用 `go()` 导航会替换整个导航栈，而不是推入新路由
- **记录日志**：在调试模式下会记录导航日志
- **URL 同步**：在 Web 平台上，浏览器地址栏会同步更新

### 使用示例

```dart
// 导航到简单路径
router.go('/home');

// 带查询参数
router.go('/family/f2/person/p1?color=blue');

// 带额外数据
router.go('/user/profile', extra: {'userId': 123});
```

### 实现

委托给 `routeInformationProvider.go()` 方法。

## restore()

```dart 420:427:lib/src/router.dart
  /// Restore the RouteMatchList
  void restore(RouteMatchList matchList) {
    log('restoring ${matchList.uri}');
    routeInformationProvider.restore(
      matchList.uri.toString(),
      matchList: matchList,
    );
  }
```

### 说明

恢复路由匹配列表。用于恢复之前保存的路由状态，通常在路由配置变更后使用。

### 参数

- **`matchList`**（必需）：要恢复的路由匹配列表

### 用途

主要用于：

- 路由配置动态变更后恢复当前路由状态
- 状态恢复场景中恢复之前的路由状态

### 实现

将 `matchList` 转换为 URI 字符串，然后调用 `routeInformationProvider.restore()` 方法。在调试模式下会记录恢复日志。

### 内部使用

在 `_handleRoutingConfigChanged()` 方法中使用，当路由配置发生变化时恢复当前路由状态。

## goNamed()

```dart 429:448:lib/src/router.dart
  /// Navigate to a named route w/ optional parameters, e.g.
  /// `name='person', pathParameters={'fid': 'f2', 'pid': 'p1'}`
  /// Navigate to the named route.
  void goNamed(
    String name, {
    Map<String, String> pathParameters = const <String, String>{},
    Map<String, dynamic> queryParameters = const <String, dynamic>{},
    Object? extra,
    String? fragment,
  }) =>
      // Construct location with optional fragment
      go(
        namedLocation(
          name,
          pathParameters: pathParameters,
          queryParameters: queryParameters,
          fragment: fragment,
        ),
        extra: extra,
      );
```

### 说明

通过路由名称导航到指定的路由。这是 `go()` 方法的命名路由版本，使用路由名称而不是 URI 路径。

### 参数

- **`name`**（必需）：路由的名称
- **`pathParameters`**：路径参数映射（默认空 map），例如 `{'fid': 'f2', 'pid': 'p1'}`
- **`queryParameters`**：查询参数映射（默认空 map）
- **`extra`**：可选的额外数据对象
- **`fragment`**：URI 片段（可选）

### 行为

1. 使用 `namedLocation()` 方法根据路由名称和参数生成 URI 位置
2. 调用 `go()` 方法进行导航

### 优势

使用命名路由的优势：

- **类型安全**：如果路由名称不存在，会在生成位置时出错
- **路径解耦**：不需要知道具体的路径格式，只需知道路由名称
- **易于重构**：路径变更时不需要修改所有调用处

### 使用示例

```dart
// 简单命名路由导航
router.goNamed('home');

// 带路径参数
router.goNamed('person', 
  pathParameters: {'fid': 'f2', 'pid': 'p1'}
);

// 带查询参数和片段
router.goNamed('search',
  queryParameters: {'q': 'flutter'},
  fragment: 'results'
);

// 带额外数据
router.goNamed('profile',
  extra: {'userId': 123}
);
```

## 导航方法对比

| 方法 | 用途 | 参数类型 | 是否替换栈 |
| --- | --- | --- | --- |
| `go()` | 通过 URI 导航 | URI 字符串 | 是 |
| `goNamed()` | 通过名称导航 | 路由名称 | 是 |
| `namedLocation()` | 生成 URI | 路由名称 | 不导航 |

## 最佳实践

1. **优先使用命名路由**：使用 `goNamed()` 而不是 `go()`，以获得更好的类型安全性和可维护性
2. **路径参数验证**：确保传入的路径参数与路由定义匹配
3. **查询参数类型**：查询参数的值可以是任意类型（`dynamic`），但通常使用字符串或数字
4. **额外数据使用**：`extra` 参数用于传递复杂对象，但要注意序列化的限制（特别是在深层链接场景中）
