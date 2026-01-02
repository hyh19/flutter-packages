# GoRouter push 系列导航方法

`GoRouter` 提供了 `push` 系列的导航方法，用于将新路由推入导航栈，而不是替换整个栈。这些方法返回 `Future<T?>`，可以用于获取从新路由返回的结果。

## push()

```dart 450:466:lib/src/router.dart
  /// Push a URI location onto the page stack w/ optional query parameters, e.g.
  /// `/family/f2/person/p1?color=blue`.
  ///
  /// See also:
  /// * [pushReplacement] which replaces the top-most page of the page stack and
  ///   always use a new page key.
  /// * [replace] which replaces the top-most page of the page stack but treats
  ///   it as the same page. The page key will be reused. This will preserve the
  ///   state and not run any page animation.
  Future<T?> push<T extends Object?>(String location, {Object? extra}) async {
    log('pushing $location');
    return routeInformationProvider.push<T>(
      location,
      base: routerDelegate.currentConfiguration,
      extra: extra,
    );
  }
```

### 说明

将指定的 URI 位置推入页面栈。与 `go()` 不同，`push()` 不会替换整个导航栈，而是在当前栈的顶部添加新路由。

### 参数

- **`location`**（必需）：要推入的 URI 位置字符串，例如 `/family/f2/person/p1?color=blue`
- **`extra`**：可选的额外数据对象

### 返回值

返回 `Future<T?>`，其中 `T` 是从新路由返回的数据类型。当新路由通过 `pop()` 返回数据时，这个 Future 会完成。

### 行为

- **推入栈顶**：新路由被推入导航栈的顶部
- **保留历史**：之前的路由仍然保留在栈中
- **返回结果**：可以通过 Future 获取从新路由返回的数据
- **页面动画**：会显示页面推入的动画效果
- **新页面 Key**：每次调用都会创建新的页面 Key

### 使用示例

```dart
// 推入新路由
await router.push('/user/profile');

// 推入带查询参数的路由
await router.push('/family/f2/person/p1?color=blue');

// 获取返回结果
final result = await router.push<String>('/edit-page');
if (result != null) {
  print('返回的数据: $result');
}
```

### 实现

委托给 `routeInformationProvider.push()` 方法，传入当前配置作为基础配置。

### 相关方法对比

- **`pushReplacement()`**：替换栈顶页面，总是使用新的页面 Key
- **`replace()`**：替换栈顶页面，但保留页面 Key（不会运行页面动画）
- **`go()`**：替换整个导航栈

## pushNamed()

```dart 468:482:lib/src/router.dart
  /// Push a named route onto the page stack w/ optional parameters, e.g.
  /// `name='person', pathParameters={'fid': 'f2', 'pid': 'p1'}`
  Future<T?> pushNamed<T extends Object?>(
    String name, {
    Map<String, String> pathParameters = const <String, String>{},
    Map<String, dynamic> queryParameters = const <String, dynamic>{},
    Object? extra,
  }) => push<T>(
    namedLocation(
      name,
      pathParameters: pathParameters,
      queryParameters: queryParameters,
    ),
    extra: extra,
  );
```

### 说明

通过路由名称将路由推入页面栈。这是 `push()` 方法的命名路由版本。

### 参数

- **`name`**（必需）：路由的名称
- **`pathParameters`**：路径参数映射（默认空 map）
- **`queryParameters`**：查询参数映射（默认空 map）
- **`extra`**：可选的额外数据对象

### 返回值

返回 `Future<T?>`，与 `push()` 相同。

### 实现

1. 使用 `namedLocation()` 方法根据路由名称和参数生成 URI 位置
2. 调用 `push()` 方法进行导航

### 使用示例

```dart
// 推入命名路由
await router.pushNamed('profile');

// 带路径参数
await router.pushNamed('person',
  pathParameters: {'fid': 'f2', 'pid': 'p1'}
);

// 带查询参数和获取返回结果
final result = await router.pushNamed<bool>('settings',
  queryParameters: {'section': 'privacy'}
);
```

## pushReplacement()

```dart 484:503:lib/src/router.dart
  /// Replaces the top-most page of the page stack with the given URL location
  /// w/ optional query parameters, e.g. `/family/f2/person/p1?color=blue`.
  ///
  /// See also:
  /// * [go] which navigates to the location.
  /// * [push] which pushes the given location onto the page stack.
  /// * [replace] which replaces the top-most page of the page stack but treats
  ///   it as the same page. The page key will be reused. This will preserve the
  ///   state and not run any page animation.
  Future<T?> pushReplacement<T extends Object?>(
    String location, {
    Object? extra,
  }) {
    log('pushReplacement $location');
    return routeInformationProvider.pushReplacement<T>(
      location,
      base: routerDelegate.currentConfiguration,
      extra: extra,
    );
  }
```

### 说明

替换导航栈顶部的页面。与 `push()` 不同，`pushReplacement()` 会移除栈顶的当前页面，然后用新页面替换它。

### 参数

- **`location`**（必需）：要替换为的 URI 位置字符串
- **`extra`**：可选的额外数据对象

### 返回值

返回 `Future<T?>`，可以获取从新路由返回的数据。

### 行为

- **替换栈顶**：移除当前栈顶页面，用新页面替换
- **新页面 Key**：总是使用新的页面 Key
- **页面动画**：会显示页面替换的动画效果
- **保留历史**：栈中其他页面仍然保留

### 使用场景

适用于以下场景：

- 登录后替换登录页面为首页
- 完成某个流程后替换当前页面为结果页面
- 需要移除当前页面但保留历史的情况

### 使用示例

```dart
// 替换栈顶页面
await router.pushReplacement('/home');

// 替换并获取返回结果
final result = await router.pushReplacement<String>('/success');
```

### 相关方法对比

- **`push()`**：推入新页面到栈顶
- **`replace()`**：替换栈顶页面，但保留页面 Key（无动画）
- **`go()`**：替换整个导航栈

## pushReplacementNamed()

```dart 505:526:lib/src/router.dart
  /// Replaces the top-most page of the page stack with the named route w/
  /// optional parameters, e.g. `name='person', pathParameters={'fid': 'f2', 'pid':
  /// 'p1'}`.
  ///
  /// See also:
  /// * [goNamed] which navigates a named route.
  /// * [pushNamed] which pushes a named route onto the page stack.
  Future<T?> pushReplacementNamed<T extends Object?>(
    String name, {
    Map<String, String> pathParameters = const <String, String>{},
    Map<String, dynamic> queryParameters = const <String, dynamic>{},
    Object? extra,
  }) {
    return pushReplacement<T>(
      namedLocation(
        name,
        pathParameters: pathParameters,
        queryParameters: queryParameters,
      ),
      extra: extra,
    );
  }
```

### 说明

通过路由名称替换导航栈顶部的页面。这是 `pushReplacement()` 方法的命名路由版本。

### 参数

- **`name`**（必需）：路由的名称
- **`pathParameters`**：路径参数映射（默认空 map）
- **`queryParameters`**：查询参数映射（默认空 map）
- **`extra`**：可选的额外数据对象

### 返回值

返回 `Future<T?>`，与 `pushReplacement()` 相同。

### 实现

1. 使用 `namedLocation()` 方法根据路由名称和参数生成 URI 位置
2. 调用 `pushReplacement()` 方法进行导航

### 使用示例

```dart
// 替换为命名路由
await router.pushReplacementNamed('home');

// 带路径参数
await router.pushReplacementNamed('result',
  pathParameters: {'id': '123'}
);

// 带查询参数和获取返回结果
final data = await router.pushReplacementNamed<Map>('details',
  queryParameters: {'type': 'success'}
);
```

## push 系列方法对比

| 方法 | 行为 | 页面 Key | 动画 | 返回值 |
| --- | --- | --- | --- | --- |
| `push()` | 推入栈顶 | 新 Key | 是 | `Future<T?>` |
| `pushNamed()` | 推入栈顶（命名） | 新 Key | 是 | `Future<T?>` |
| `pushReplacement()` | 替换栈顶 | 新 Key | 是 | `Future<T?>` |
| `pushReplacementNamed()` | 替换栈顶（命名） | 新 Key | 是 | `Future<T?>` |

## 最佳实践

1. **使用命名路由**：优先使用 `pushNamed()` 和 `pushReplacementNamed()` 以获得更好的类型安全性
2. **处理返回结果**：使用 `await` 等待 Future 完成，处理可能返回的数据
3. **选择合适的方法**：
   - 需要保留历史时使用 `push()`
   - 需要移除当前页面时使用 `pushReplacement()`
   - 需要替换整个栈时使用 `go()`
4. **类型参数**：指定泛型类型参数 `<T>` 以获得类型安全的返回值
