# GoRouter replace、pop 和其他导航方法

本文档介绍 `GoRouter` 中的 `replace` 系列方法、`pop()` 方法和 `refresh()` 方法。这些方法用于替换路由（保留状态）、弹出路由和刷新路由。

## replace()

```dart 528:545:lib/src/router.dart
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
  Future<T?> replace<T>(String location, {Object? extra}) {
    log('replace $location');
    return routeInformationProvider.replace<T>(
      location,
      base: routerDelegate.currentConfiguration,
      extra: extra,
    );
  }
```

### 说明

替换导航栈顶部的页面，但将其视为同一个页面。这是 `replace()` 方法的关键特性。

### 参数

- **`location`**（必需）：要替换为的 URI 位置字符串
- **`extra`**：可选的额外数据对象

### 返回值

返回 `Future<T?>`，可以获取从新路由返回的数据。

### 关键特性

- **重用页面 Key**：页面 Key 会被重用，这意味着新页面被视为原页面的更新
- **保留状态**：由于 Key 相同，页面状态会被保留
- **无页面动画**：不会运行页面切换动画，因为被视为同一页面
- **替换栈顶**：移除当前栈顶页面，用新页面替换

### 使用场景

适用于以下场景：

- 更新同一页面的不同内容（例如，从用户详情页的某个标签切换到另一个标签）
- 需要保留页面状态但更新内容的情况
- 避免页面动画，提供无缝的视觉体验

### 使用示例

```dart
// 替换栈顶页面（保留状态）
await router.replace('/user/profile?tab=settings');

// 替换并获取返回结果
final result = await router.replace<String>('/user/profile?tab=posts');
```

### 与 pushReplacement() 的对比

| 特性 | `replace()` | `pushReplacement()` |
| --- | --- | --- |
| 页面 Key | 重用 | 新建 |
| 页面状态 | 保留 | 不保留 |
| 页面动画 | 无 | 有 |
| 使用场景 | 更新同一页面 | 替换为新页面 |

## replaceNamed()

```dart 547:572:lib/src/router.dart
  /// Replaces the top-most page with the named route and optional parameters,
  /// preserving the page key.
  ///
  /// This will preserve the state and not run any page animation. Optional
  /// parameters can be providded to the named route, e.g. `name='person',
  /// pathParameters={'fid': 'f2', 'pid': 'p1'}`.
  ///
  /// See also:
  /// * [pushNamed] which pushes the given location onto the page stack.
  /// * [pushReplacementNamed] which replaces the top-most page of the page
  ///   stack but always uses a new page key.
  Future<T?> replaceNamed<T>(
    String name, {
    Map<String, String> pathParameters = const <String, String>{},
    Map<String, dynamic> queryParameters = const <String, dynamic>{},
    Object? extra,
  }) {
    return replace(
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

通过路由名称替换导航栈顶部的页面，保留页面 Key。这是 `replace()` 方法的命名路由版本。

### 参数

- **`name`**（必需）：路由的名称
- **`pathParameters`**：路径参数映射（默认空 map）
- **`queryParameters`**：查询参数映射（默认空 map）
- **`extra`**：可选的额外数据对象

### 返回值

返回 `Future<T?>`，与 `replace()` 相同。

### 特性

与 `replace()` 相同的特性：

- 重用页面 Key
- 保留页面状态
- 无页面动画

### 实现

1. 使用 `namedLocation()` 方法根据路由名称和参数生成 URI 位置
2. 调用 `replace()` 方法进行导航

### 使用示例

```dart
// 替换为命名路由（保留状态）
await router.replaceNamed('profile',
  queryParameters: {'tab': 'settings'}
);

// 带路径参数
await router.replaceNamed('user',
  pathParameters: {'id': '123'},
  queryParameters: {'view': 'details'}
);
```

## pop()

```dart 574:588:lib/src/router.dart
  /// Pop the top-most route off the current screen.
  ///
  /// If the top-most route is a pop up or dialog, this method pops it instead
  /// of any GoRoute under it.
  ///
  /// Ensure that the `value` of `routeInformationProvider` is synced
  ///  with `routerDelegate.currentConfiguration`.
  void pop<T extends Object?>([T? result]) {
    assert(() {
      log('popping ${routerDelegate.currentConfiguration.uri}');
      return true;
    }());
    routerDelegate.pop<T>(result);
    restore(routerDelegate.currentConfiguration);
  }
```

### 说明

弹出导航栈顶部的路由。这是用于返回上一页的核心方法。

### 参数

- **`result`**（可选）：要返回给上一个路由的数据。类型由泛型参数 `T` 指定

### 返回值

无返回值（`void`）。

### 行为

- **弹出栈顶**：移除导航栈顶部的路由
- **处理弹窗优先**：如果栈顶是弹窗（pop up）或对话框（dialog），会先弹出它们，而不是弹出下方的 `GoRoute`
- **同步状态**：确保 `routeInformationProvider` 的值与 `routerDelegate.currentConfiguration` 同步
- **恢复路由状态**：调用 `restore()` 方法恢复当前配置

### 使用示例

```dart
// 简单弹出
router.pop();

// 弹出并返回数据
router.pop<String>('保存成功');

// 弹出并返回布尔值
router.pop<bool>(true);

// 弹出并返回对象
router.pop<Map<String, dynamic>>({'status': 'completed'});
```

### 与 push() 的配合使用

`pop()` 通常与 `push()` 系列方法配合使用：

```dart
// 推入新路由并等待结果
final result = await router.push<String>('/edit-page');
if (result != null) {
  print('返回结果: $result');
}

// 在新路由中返回数据
router.pop('编辑完成');
```

### 注意事项

- 如果栈中只有一个路由，调用 `pop()` 可能不会有任何效果（取决于平台行为）
- 使用 `canPop()` 方法可以检查是否可以弹出路由

## refresh()

```dart 590:597:lib/src/router.dart
  /// Refresh the route.
  void refresh() {
    assert(() {
      log('refreshing ${routerDelegate.currentConfiguration.uri}');
      return true;
    }());
    routeInformationProvider.notifyListeners();
  }
```

### 说明

刷新当前路由。这会通知所有监听路由信息的监听器，触发路由信息的重新评估。

### 返回值

无返回值（`void`）。

### 行为

- **通知监听器**：调用 `routeInformationProvider.notifyListeners()` 通知所有监听器
- **触发重定向**：如果路由配置中使用了 `BuildContext.dependOnInheritedWidgetOfExactType`（通常通过 `of` 方法），会触发重定向的重新评估
- **不改变位置**：不会改变当前的路由位置，只是刷新路由信息

### 使用场景

适用于以下场景：

- 当依赖的外部状态发生变化时（例如，用户登录状态改变），需要重新评估重定向逻辑
- 需要手动触发路由刷新以响应状态变化

### 使用示例

```dart
// 刷新路由（例如，在用户登录状态改变后）
void onLoginStateChanged() {
  // 登录状态改变
  isLoggedIn = true;
  // 刷新路由以触发重定向
  router.refresh();
}
```

### 与 refreshListenable 的关系

如果在构造函数中提供了 `refreshListenable` 参数，当该 Listenable 发生变化时，会自动触发路由刷新。`refresh()` 方法提供了手动触发刷新的能力。

## 方法对比总结

| 方法 | 行为 | 页面 Key | 状态 | 动画 | 返回值 |
|------|------|----------|------|------|--------|
| `replace()` | 替换栈顶 | 重用 | 保留 | 无 | `Future<T?>` |
| `replaceNamed()` | 替换栈顶（命名） | 重用 | 保留 | 无 | `Future<T?>` |
| `pushReplacement()` | 替换栈顶 | 新建 | 不保留 | 有 | `Future<T?>` |
| `pop()` | 弹出栈顶 | - | - | 有 | `void` |
| `refresh()` | 刷新路由 | - | - | - | `void` |

## 最佳实践

1. **选择合适的方法**：
   - 需要更新同一页面内容时使用 `replace()`
   - 需要替换为新页面时使用 `pushReplacement()`
   - 需要返回上一页时使用 `pop()`
   - 需要手动触发重定向重新评估时使用 `refresh()`

2. **返回值处理**：使用 `pop()` 返回数据时，在上层使用 `await` 等待结果

3. **状态保留**：理解 `replace()` 和 `pushReplacement()` 在状态保留方面的区别

4. **路由刷新**：优先使用 `refreshListenable` 参数进行自动刷新，必要时使用 `refresh()` 方法手动触发
