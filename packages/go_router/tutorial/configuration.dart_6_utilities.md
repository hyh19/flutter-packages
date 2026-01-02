# RouteConfiguration 工具方法和调试方法

## 概述

`RouteConfiguration` 类提供了一些工具方法和调试辅助方法，用于获取路由位置、字符串表示、调试信息输出等功能。这些方法虽然不是核心的路由处理逻辑，但对于开发调试和工具支持非常重要。

本文档介绍 `RouteConfiguration` 中的工具方法和调试方法：`locationForRoute`、`toString`、`debugKnownRoutes`、`_debugFullPathsFor` 和 `_getDecoration`。

## 获取路由位置：locationForRoute

```dart 671:676:lib/src/configuration.dart
/// Get the location for the provided route.
///
/// Builds the absolute path for the route, by concatenating the paths of the
/// route and all its ancestors.
String? locationForRoute(RouteBase route) =>
    fullPathForRoute(route, '', _routingConfig.value.routes);
```

`locationForRoute` 方法用于获取指定路由的完整路径位置。

### 功能说明

该方法是一个简单的包装方法，调用 `fullPathForRoute` 函数（定义在 `path_utils.dart` 中）来构建路由的完整路径。

### 参数说明

- **`route`**：`RouteBase` 类型，要查找位置的路由对象。

### 返回值

返回 `String?` 类型：

- 如果找到了路由，返回其完整路径（如 `/users/:id`）
- 如果路由不在当前配置中，返回 `null`

### 实现细节

方法调用 `fullPathForRoute`，传入：

- 目标路由：`route`
- 父路径：空字符串 `''`（从根路径开始）
- 路由列表：`_routingConfig.value.routes`（顶层路由列表）

`fullPathForRoute` 会递归搜索路由树，找到目标路由后返回从根路径到该路由的完整路径。

### 使用场景

这个方法主要用于：

- 在调试时查找特定路由的路径
- 在重定向逻辑中获取路由的位置
- 在路由配置验证中检查路由路径

## 字符串表示：toString

```dart 678:681:lib/src/configuration.dart
@override
String toString() {
  return 'RouterConfiguration: ${_routingConfig.value.routes}';
}
```

`toString` 方法提供了 `RouteConfiguration` 对象的字符串表示。

### 功能说明

该方法重写了 `Object.toString()` 方法，返回一个包含路由配置信息的字符串。主要用于调试输出和日志记录。

### 返回值

返回格式为 `'RouterConfiguration: [路由列表]'` 的字符串，其中路由列表是 `_routingConfig.value.routes` 的字符串表示。

### 使用场景

- 调试时查看路由配置对象
- 日志记录
- 错误消息中的对象表示

## 调试已知路由：debugKnownRoutes

```dart 683:708:lib/src/configuration.dart
/// Returns the full path of [routes].
///
/// Each path is indented based depth of the hierarchy, and its `name`
/// is also appended if not null
@visibleForTesting
String debugKnownRoutes() {
  final sb = StringBuffer();
  sb.writeln('Full paths for routes:');
  _debugFullPathsFor(
    _routingConfig.value.routes,
    '',
    const <_DecorationType>[],
    sb,
  );

  if (_nameToPath.isNotEmpty) {
    sb.writeln('known full paths for route names:');
    for (final MapEntry<String, _NamedPath> e in _nameToPath.entries) {
      sb.writeln(
        '  ${e.key} => ${e.value.path}${e.value.caseSensitive ? '' : ' (case-insensitive)'}',
      );
    }
  }

  return sb.toString();
}
```

`debugKnownRoutes` 方法生成一个格式化的字符串，包含所有已知路由的完整路径和命名路由的映射关系。

### 功能说明

该方法执行以下操作：

1. **构建路由树表示**：调用 `_debugFullPathsFor` 方法，生成格式化的路由树结构，使用树形字符（`├─`、`└─`、`│`）展示层级关系。

2. **输出命名路由映射**：如果 `_nameToPath` 映射不为空，输出所有命名路由的映射关系，包括：
   - 路由名称
   - 对应的完整路径
   - 是否大小写敏感的标志

3. **返回完整字符串**：将所有信息组合成字符串并返回。

### 返回值

返回一个多行字符串，包含：

- 格式化的路由树（使用树形字符表示层级）
- 命名路由的映射表（如果存在）

### 使用场景

这个方法主要用于：

- 调试时查看完整的路由配置
- 在日志中输出路由信息
- 在 `_onRoutingTableChanged` 中记录路由变更（通过 `log(debugKnownRoutes())`）

### 输出示例

```
Full paths for routes:
├─/
├─/users
│ ├─/users/:id
│ └─/users/:id/posts
└─/settings
known full paths for route names:
  home => /
  userDetail => /users/:id
  userPosts => /users/:id/posts
  settings => /settings
```

## 调试完整路径：_debugFullPathsFor

```dart 710:741:lib/src/configuration.dart
void _debugFullPathsFor(
  List<RouteBase> routes,
  String parentFullpath,
  List<_DecorationType> parentDecoration,
  StringBuffer sb,
) {
  for (final (int index, RouteBase route) in routes.indexed) {
    final List<_DecorationType> decoration = _getDecoration(
      parentDecoration,
      index,
      routes.length,
    );
    final String decorationString = decoration
        .map((_DecorationType e) => e.toString())
        .join();
    var path = parentFullpath;
    if (route is GoRoute) {
      path = concatenatePaths(parentFullpath, route.path);
      final String? screenName = route.builder?.runtimeType
          .toString()
          .split('=> ')
          .last;
      sb.writeln(
        '$decorationString$path '
        '${screenName == null ? '' : '($screenName)'}',
      );
    } else if (route is ShellRouteBase) {
      sb.writeln('$decorationString (ShellRoute)');
    }
    _debugFullPathsFor(route.routes, path, decoration, sb);
  }
}
```

`_debugFullPathsFor` 是一个私有的递归方法，用于生成格式化的路由树字符串表示。

### 功能说明

该方法递归遍历路由树，为每个路由生成格式化的输出：

1. **遍历路由列表**：使用 `indexed` 扩展方法获取每个路由的索引。

2. **计算装饰符**：调用 `_getDecoration` 方法，根据路由在列表中的位置（是否为最后一个）和父装饰符，计算当前路由的装饰符列表。

3. **构建装饰符字符串**：将装饰符列表转换为字符串（如 `├─`、`└─`、`│` 等）。

4. **处理 GoRoute**：
   - 构建完整路径：使用 `concatenatePaths` 拼接父路径和当前路由路径
   - 提取屏幕名称：从 `route.builder` 的运行时类型中提取屏幕名称（用于调试信息）
   - 输出路径：将装饰符、路径和屏幕名称写入 `StringBuffer`

5. **处理 ShellRouteBase**：
   - 输出 `(ShellRoute)` 标记，表示这是一个 Shell 路由

6. **递归处理子路由**：对每个路由的子路由列表递归调用 `_debugFullPathsFor`。

### 参数说明

- **`routes`**：`List<RouteBase>` 类型，要处理的路由列表
- **`parentFullpath`**：`String` 类型，父路由的完整路径
- **`parentDecoration`**：`List<_DecorationType>` 类型，父路由的装饰符列表
- **`sb`**：`StringBuffer` 类型，用于构建输出字符串的缓冲区

### 树形字符说明

生成的输出使用以下字符表示路由层级：

- `├─`：表示有后续兄弟节点的节点
- `└─`：表示最后一个兄弟节点
- `│`：表示父节点有后续兄弟节点（用于子节点的对齐）
- `  `（两个空格）：表示父节点是最后一个兄弟节点（用于子节点的对齐）

### 输出示例

对于以下路由配置：

```dart
GoRoute(path: '/', ...),
GoRoute(
  path: '/users',
  routes: [
    GoRoute(path: '/:id', ...),
    GoRoute(path: '/:id/posts', ...),
  ],
),
GoRoute(path: '/settings', ...),
```

输出可能是：

```
├─/
├─/users
│ ├─/users/:id
│ └─/users/:id/posts
└─/settings
```

## 获取装饰符：_getDecoration

```dart 743:769:lib/src/configuration.dart
List<_DecorationType> _getDecoration(
  List<_DecorationType> parentDecoration,
  int index,
  int length,
) {
  final Iterable<_DecorationType> newDecoration = parentDecoration.map((
    _DecorationType e,
  ) {
    switch (e) {
      // swap
      case _DecorationType.branch:
        return _DecorationType.parentBranch;
      case _DecorationType.leaf:
        return _DecorationType.none;
      // no swap
      case _DecorationType.parentBranch:
        return _DecorationType.parentBranch;
      case _DecorationType.none:
        return _DecorationType.none;
    }
  });
  if (index == length - 1) {
    return <_DecorationType>[...newDecoration, _DecorationType.leaf];
  } else {
    return <_DecorationType>[...newDecoration, _DecorationType.branch];
  }
}
```

`_getDecoration` 是一个私有辅助方法，用于计算路由在树形输出中的装饰符。

### 功能说明

该方法根据路由在列表中的位置和父装饰符，计算当前路由的装饰符列表：

1. **转换父装饰符**：
   - `branch`（`├─`）→ `parentBranch`（`│`）：如果父节点有后续兄弟节点，子节点使用竖线对齐
   - `leaf`（`└─`）→ `none`（`  `）：如果父节点是最后一个兄弟节点，子节点使用空格对齐
   - `parentBranch`（`│`）→ `parentBranch`（`│`）：保持不变
   - `none`（`  `）→ `none`（`  `）：保持不变

2. **添加当前装饰符**：
   - 如果是最后一个元素（`index == length - 1`），添加 `leaf`（`└─`）
   - 否则，添加 `branch`（`├─`）

### 参数说明

- **`parentDecoration`**：`List<_DecorationType>` 类型，父路由的装饰符列表
- **`index`**：`int` 类型，当前路由在列表中的索引
- **`length`**：`int` 类型，路由列表的总长度

### 返回值

返回 `List<_DecorationType>`，包含当前路由的完整装饰符列表。

### 装饰符类型

`_DecorationType` 枚举定义了四种装饰符类型：

```dart 801:813:lib/src/configuration.dart
enum _DecorationType {
  parentBranch('│ '),
  branch('├─'),
  leaf('└─'),
  none('  ');

  const _DecorationType(this.value);

  final String value;

  @override
  String toString() => value;
}
```

- **`parentBranch`**：`│`，表示父节点有后续兄弟节点（用于子节点对齐）
- **`branch`**：`├─`，表示当前节点有后续兄弟节点
- **`leaf`**：`└─`，表示当前节点是最后一个兄弟节点
- **`none`**：`  `，两个空格，表示父节点是最后一个兄弟节点（用于子节点对齐）

### 算法说明

装饰符的计算遵循以下规则：

1. **父装饰符转换**：将父节点的装饰符转换为对齐字符（`branch` → `parentBranch`，`leaf` → `none`）

2. **当前装饰符添加**：根据当前节点在列表中的位置，添加相应的装饰符

3. **层级累积**：装饰符列表会随着递归深度累积，形成完整的树形结构

例如，对于一个三层深度的路由树：

```
├─/users          [branch]
│ ├─/users/:id    [parentBranch, branch]
│ └─/users/:id/posts [parentBranch, leaf]
```

在第二层：

- 第一个路由使用 `[parentBranch, branch]`（父节点有后续兄弟，当前节点也有后续兄弟）
- 第二个路由使用 `[parentBranch, leaf]`（父节点有后续兄弟，当前节点是最后一个）

## 总结

`RouteConfiguration` 的工具方法和调试方法提供了：

- **路由位置查询**：`locationForRoute` 方法用于查找路由的完整路径
- **对象表示**：`toString` 方法提供对象的基本字符串表示
- **调试信息**：`debugKnownRoutes` 方法生成格式化的路由树和命名路由映射
- **树形格式化**：`_debugFullPathsFor` 和 `_getDecoration` 方法共同实现美观的树形输出

这些方法对于开发和调试 go_router 应用非常有用，特别是在理解路由配置结构和排查路由问题时。
