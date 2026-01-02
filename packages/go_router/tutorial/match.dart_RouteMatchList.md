# RouteMatchList 类详解

## 概述

`RouteMatchList` 是 `go_router` 包中用于表示路由匹配列表的核心类。它维护了一个 `RouteMatchBase` 对象的列表，这些对象代表了当前路由导航栈中的所有路由匹配结果。

```dart 512:526:lib/src/match.dart
/// The list of [RouteMatchBase] objects.
///
/// This can contains tree structure if there are [ShellRouteMatch] in the list.
///
/// This corresponds to the GoRouter's history.
@immutable
class RouteMatchList with Diagnosticable {
  /// RouteMatchList constructor.
  RouteMatchList({
    required this.matches,
    required this.uri,
    this.extra,
    this.error,
    required this.pathParameters,
  }) : fullPath = _generateFullPath(matches);
```

这个类具有以下关键特性：

- **不可变性**：使用 `@immutable` 注解，确保实例的不可变性
- **树形结构支持**：当列表中包含 `ShellRouteMatch` 时，可以形成树形结构
- **路由历史**：对应 `GoRouter` 的导航历史
- **路径参数管理**：维护当前路由的路径参数
- **错误处理**：可以包含路由匹配过程中产生的错误

## 核心成员变量

### matches

```dart 535:536:lib/src/match.dart
  /// The route matches.
  final List<RouteMatchBase> matches;
```

`matches` 是 `RouteMatchList` 的核心，包含了当前导航栈中的所有路由匹配对象。这个列表可能包含三种类型的匹配：

- **`RouteMatch`**：普通路由的匹配结果
- **`ShellRouteMatch`**：Shell 路由的匹配结果，内部包含嵌套的 `matches` 列表
- **`ImperativeRouteMatch`**：通过 `GoRouter.push` 推送的路由匹配

### pathParameters

```dart 538:542:lib/src/match.dart
  /// Parameters for the matched route, URI-encoded.
  ///
  /// The parameters only reflects [RouteMatch]s that are not
  /// [ImperativeRouteMatch].
  final Map<String, String> pathParameters;
```

路径参数映射表，用于存储路由路径中提取的参数（如 `/user/:id` 中的 `id`）。注意这个映射只反映非 `ImperativeRouteMatch` 的 `RouteMatch`。

### uri

```dart 544:547:lib/src/match.dart
  /// The uri of the current match.
  ///
  /// This uri only reflects [RouteMatch]s that are not [ImperativeRouteMatch].
  final Uri uri;
```

当前匹配对应的 URI，同样只反映非 `ImperativeRouteMatch` 的路由。

### fullPath

```dart 555:562:lib/src/match.dart
  /// the full path pattern that matches the uri.
  ///
  /// For example:
  ///
  /// ```dart
  /// '/family/:fid/person/:pid'
  /// ```
  final String fullPath;
```

完整的路径模式，由所有路由匹配组合而成。例如，如果路由结构是 `/family/:fid` 和 `/person/:pid`，那么 `fullPath` 就是 `/family/:fid/person/:pid`。

### extra

```dart 549:550:lib/src/match.dart
  /// An extra object to pass along with the navigation.
  final Object? extra;
```

导航时传递的额外对象，可以用于在路由之间传递自定义数据。

### error

```dart 552:553:lib/src/match.dart
  /// An exception if there was an error during matching.
  final GoException? error;
```

如果在路由匹配过程中发生错误，此字段会包含相应的异常对象。

## 静态工厂方法

### empty

```dart 528:533:lib/src/match.dart
  /// Constructs an empty matches object.
  static RouteMatchList empty = RouteMatchList(
    matches: const <RouteMatchBase>[],
    uri: Uri(),
    pathParameters: const <String, String>{},
  );
```

创建一个空的 `RouteMatchList` 实例，用于表示没有任何路由匹配的状态。

## 路径生成方法

### _generateFullPath

```dart 564:611:lib/src/match.dart
  /// Generates the full path (ex: `'/family/:fid/person/:pid'`) of a list of
  /// [RouteMatch].
  ///
  /// This method ignores [ImperativeRouteMatch]s in the `matches`, as they
  /// don't contribute to the path.
  ///
  /// This methods considers that [matches]'s elements verify the go route
  /// structure given to `GoRouter`. For example, if the routes structure is
  ///
  /// ```dart
  /// GoRoute(
  ///   path: '/a',
  ///   routes: [
  ///     GoRoute(
  ///       path: 'b',
  ///       routes: [
  ///         GoRoute(
  ///           path: 'c',
  ///         ),
  ///       ],
  ///     ),
  ///   ],
  /// ),
  /// ```
  ///
  /// The [matches] must be the in same order of how GoRoutes are matched.
  ///
  /// ```dart
  /// [RouteMatchA(), RouteMatchB(), RouteMatchC()]
  /// ```
  static String _generateFullPath(Iterable<RouteMatchBase> matches) {
    var fullPath = '';
    for (final RouteMatchBase match in matches.where(
      (RouteMatchBase match) => match is! ImperativeRouteMatch,
    )) {
      final String pathSegment;
      if (match is RouteMatch) {
        pathSegment = match.route.path;
      } else if (match is ShellRouteMatch) {
        pathSegment = _generateFullPath(match.matches);
      } else {
        assert(false, 'Unexpected match type: $match');
        continue;
      }
      fullPath = concatenatePaths(fullPath, pathSegment);
    }
    return fullPath;
  }
```

这是一个静态私有方法，用于生成完整的路径模式。它的工作原理：

1. **过滤 `ImperativeRouteMatch`**：忽略 `ImperativeRouteMatch`，因为它们不贡献路径
2. **递归处理**：如果遇到 `ShellRouteMatch`，递归处理其内部的 `matches`
3. **路径拼接**：使用 `concatenatePaths` 函数将所有路径段拼接起来

例如，如果有以下路由匹配：

- `RouteMatch` with path `/family/:fid`
- `ShellRouteMatch` containing `RouteMatch` with path `/person/:pid`

那么 `fullPath` 将是 `/family/:fid/person/:pid`。

## 查询方法

### isEmpty / isNotEmpty

```dart 613:617:lib/src/match.dart
  /// Returns true if there are no matches.
  bool get isEmpty => matches.isEmpty;

  /// Returns true if there are matches.
  bool get isNotEmpty => matches.isNotEmpty;
```

简单的便捷方法，用于检查匹配列表是否为空。

### last / lastOrNull

```dart 777:801:lib/src/match.dart
  /// The last leaf route.
  ///
  /// If the last RouteMatchBase from [matches] is a ShellRouteMatch, it
  /// recursively goes into its [ShellRouteMatch.matches] until it reach the leaf
  /// [RouteMatch].
  ///
  /// Throws a [StateError] if [matches] is empty.
  RouteMatch get last {
    if (matches.last is RouteMatch) {
      return matches.last as RouteMatch;
    }
    return (matches.last as ShellRouteMatch)._lastLeaf;
  }

  /// The last leaf route or null if [matches] is empty
  ///
  /// If the last RouteMatchBase from [matches] is a ShellRouteMatch, it
  /// recursively goes into its [ShellRouteMatch.matches] until it reach the leaf
  /// [RouteMatch].
  RouteMatch? get lastOrNull {
    if (matches.isEmpty) {
      return null;
    }
    return last;
  }
```

获取最后一个叶子路由匹配。由于 `matches` 可能包含 `ShellRouteMatch`（具有嵌套结构），这个方法会递归深入到最底层的叶子节点。

- **`last`**：如果列表为空会抛出 `StateError`
- **`lastOrNull`**：如果列表为空返回 `null`

### isError

```dart 803:804:lib/src/match.dart
  /// Returns true if the current match intends to display an error screen.
  bool get isError => error != null;
```

检查当前匹配是否表示错误状态。

### routes

```dart 806:814:lib/src/match.dart
  /// The routes for each of the matches.
  List<RouteBase> get routes {
    final result = <RouteBase>[];
    visitRouteMatches((RouteMatchBase match) {
      result.add(match.route);
      return true;
    });
    return result;
  }
```

获取所有匹配对象对应的路由列表。使用 `visitRouteMatches` 方法遍历所有匹配（包括嵌套在 `ShellRouteMatch` 中的）。

## 遍历方法

### visitRouteMatches

```dart 816:841:lib/src/match.dart
  /// Traverse route matches in this match list in preorder until visitor
  /// returns false.
  ///
  /// This method visit recursively into shell route matches.
  // TODO(loic-sharma): Remove meta library prefix.
  // https://github.com/flutter/flutter/issues/171410
  @meta.internal
  void visitRouteMatches(RouteMatchVisitor visitor) {
    _visitRouteMatches(matches, visitor);
  }

  static bool _visitRouteMatches(
    List<RouteMatchBase> matches,
    RouteMatchVisitor visitor,
  ) {
    for (final routeMatch in matches) {
      if (!visitor(routeMatch)) {
        return false;
      }
      if (routeMatch is ShellRouteMatch &&
          !_visitRouteMatches(routeMatch.matches, visitor)) {
        return false;
      }
    }
    return true;
  }
```

按前序遍历方式遍历所有路由匹配。这是一个深度优先遍历：

1. 访问当前匹配
2. 如果访问器返回 `false`，停止遍历
3. 如果当前匹配是 `ShellRouteMatch`，递归遍历其内部的 `matches`
4. 继续下一个匹配

这个方法对于需要访问所有路由匹配（包括嵌套的）的场景非常有用。

## 修改方法

### push

```dart 619:632:lib/src/match.dart
  /// Returns a new instance of RouteMatchList with the input `match` pushed
  /// onto the current instance.
  RouteMatchList push(ImperativeRouteMatch match) {
    if (match.matches.isError) {
      return copyWith(matches: <RouteMatchBase>[...matches, match]);
    }
    return copyWith(
      matches: _createNewMatchUntilIncompatible(
        matches,
        match.matches.matches,
        match,
      ),
    );
  }
```

将一个新的 `ImperativeRouteMatch` 推入到当前列表。这个方法的核心逻辑：

1. **错误处理**：如果新匹配表示错误，直接追加到列表末尾
2. **兼容性检查**：否则调用 `_createNewMatchUntilIncompatible` 来处理兼容的路由匹配

#### _createNewMatchUntilIncompatible

```dart 634:661:lib/src/match.dart
  static List<RouteMatchBase> _createNewMatchUntilIncompatible(
    List<RouteMatchBase> currentMatches,
    List<RouteMatchBase> otherMatches,
    ImperativeRouteMatch match,
  ) {
    final List<RouteMatchBase> newMatches = currentMatches.toList();
    if (otherMatches.last is ShellRouteMatch &&
        newMatches.isNotEmpty &&
        otherMatches.last.route == newMatches.last.route) {
      assert(newMatches.last is ShellRouteMatch);
      final lastShellRouteMatch = newMatches.removeLast() as ShellRouteMatch;
      newMatches.add(
        // Create a new copy of the `lastShellRouteMatch`.
        lastShellRouteMatch.copyWith(
          matches: _createNewMatchUntilIncompatible(
            lastShellRouteMatch.matches,
            (otherMatches.last as ShellRouteMatch).matches,
            match,
          ),
        ),
      );
      return newMatches;
    }
    newMatches.add(
      _cloneBranchAndInsertImperativeMatch(otherMatches.last, match),
    );
    return newMatches;
  }
```

这个方法用于创建新的匹配列表，直到遇到不兼容的路由。它的逻辑：

1. **Shell 路由兼容性检查**：如果最后一个匹配是 `ShellRouteMatch` 且与新的匹配使用相同的路由，则递归更新这个 `ShellRouteMatch` 的内部 `matches`
2. **克隆分支**：否则，克隆新匹配的分支并插入 `ImperativeRouteMatch`

#### _cloneBranchAndInsertImperativeMatch

```dart 663:678:lib/src/match.dart
  static RouteMatchBase _cloneBranchAndInsertImperativeMatch(
    RouteMatchBase branch,
    ImperativeRouteMatch match,
  ) {
    if (branch is ShellRouteMatch) {
      return branch.copyWith(
        matches: <RouteMatchBase>[
          _cloneBranchAndInsertImperativeMatch(branch.matches.last, match),
        ],
      );
    }
    // Add the input `match` instead of the incompatibleMatch since it contains
    // page key and push future.
    assert(branch.route == match.route);
    return match;
  }
```

这个方法用于克隆路由分支并插入 `ImperativeRouteMatch`：

1. **Shell 路由处理**：如果分支是 `ShellRouteMatch`，递归处理其最后一个匹配
2. **替换匹配**：最终用 `ImperativeRouteMatch` 替换叶子路由匹配（因为它包含页面 key 和 push future）

### remove

```dart 680:726:lib/src/match.dart
  /// Returns a new instance of RouteMatchList with the input `match` removed
  /// from the current instance.
  RouteMatchList remove(RouteMatchBase match) {
    final List<RouteMatchBase> newMatches = _removeRouteMatchFromList(
      matches,
      match,
    );
    if (newMatches == matches) {
      return this;
    }

    final String fullPath = _generateFullPath(newMatches);
    if (this.fullPath == fullPath) {
      return copyWith(matches: newMatches);
    }

    if (newMatches.isEmpty) {
      return RouteMatchList.empty;
    }

    RouteBase newRoute = newMatches.last.route;
    while (newRoute is ShellRouteBase) {
      newRoute = newRoute.routes.last;
    }
    newRoute as GoRoute;
    // Need to remove path parameters that are no longer in the fullPath.
    final newParameters = <String>[];
    patternToRegExp(
      fullPath,
      newParameters,
      caseSensitive: newRoute.caseSensitive,
    );
    final Set<String> validParameters = newParameters.toSet();
    final newPathParameters = Map<String, String>.fromEntries(
      pathParameters.entries.where(
        (MapEntry<String, String> value) => validParameters.contains(value.key),
      ),
    );
    final Uri newUri = uri.replace(
      path: patternToPath(fullPath, newPathParameters),
    );
    return copyWith(
      matches: newMatches,
      uri: newUri,
      pathParameters: newPathParameters,
    );
  }
```

从列表中移除指定的匹配。这个方法比较复杂，包含以下步骤：

1. **移除匹配**：调用 `_removeRouteMatchFromList` 移除匹配
2. **快速返回**：如果没有变化，直接返回当前实例
3. **路径更新**：生成新的 `fullPath`
4. **空列表处理**：如果移除后列表为空，返回 `RouteMatchList.empty`
5. **参数清理**：移除不再有效的路径参数
6. **URI 更新**：根据新的路径和参数更新 URI

#### _removeRouteMatchFromList

```dart 728:775:lib/src/match.dart
  /// Returns a new List from the input matches with target removed.
  ///
  /// This method recursively looks into any ShellRouteMatch in matches and
  /// removes target if it found a match in the match list nested in
  /// ShellRouteMatch.
  ///
  /// This method returns a new list as long as the target is found in the
  /// matches' subtree.
  ///
  /// If a target is found, the target and every node after the target in tree
  /// order is removed.
  static List<RouteMatchBase> _removeRouteMatchFromList(
    List<RouteMatchBase> matches,
    RouteMatchBase target,
  ) {
    // Remove is caused by pop; therefore, start searching from the end.
    for (int index = matches.length - 1; index >= 0; index -= 1) {
      final RouteMatchBase match = matches[index];
      if (match == target) {
        // Remove any redirect only route immediately before the target.
        while (index > 0) {
          final RouteMatchBase lookBefore = matches[index - 1];
          if (lookBefore is! RouteMatch || !lookBefore.route.redirectOnly) {
            break;
          }
          index -= 1;
        }
        return matches.sublist(0, index);
      }
      if (match is ShellRouteMatch) {
        final List<RouteMatchBase> newSubMatches = _removeRouteMatchFromList(
          match.matches,
          target,
        );
        if (newSubMatches == match.matches) {
          // Didn't find target in the newSubMatches.
          continue;
        }
        // Removes `match` if its sub match list become empty after the remove.
        return <RouteMatchBase>[
          ...matches.sublist(0, index),
          if (newSubMatches.isNotEmpty) match.copyWith(matches: newSubMatches),
        ];
      }
    }
    // Target is not in the match subtree.
    return matches;
  }
```

这个静态方法递归地从匹配列表中移除目标匹配：

1. **从后往前搜索**：因为移除通常是由 pop 操作引起的，所以从列表末尾开始搜索
2. **直接匹配**：如果找到目标，移除它及其之后的所有节点
3. **重定向路由处理**：同时移除目标之前的所有仅用于重定向的路由
4. **递归处理 Shell 路由**：如果当前匹配是 `ShellRouteMatch`，递归处理其内部的 `matches`
5. **空列表处理**：如果移除后 `ShellRouteMatch` 的内部列表为空，则移除该 `ShellRouteMatch`

## 辅助方法

### copyWith

```dart 843:859:lib/src/match.dart
  /// Create a new [RouteMatchList] with given parameter replaced.
  // TODO(loic-sharma): Remove meta library prefix.
  // https://github.com/flutter/flutter/issues/171410
  @meta.internal
  RouteMatchList copyWith({
    List<RouteMatchBase>? matches,
    Uri? uri,
    Map<String, String>? pathParameters,
  }) {
    return RouteMatchList(
      matches: matches ?? this.matches,
      uri: uri ?? this.uri,
      extra: extra,
      error: error,
      pathParameters: pathParameters ?? this.pathParameters,
    );
  }
```

创建一个新的 `RouteMatchList` 实例，可选地替换部分字段。这是不可变对象模式的典型实现。

## 相等性和哈希码

### operator ==

```dart 861:875:lib/src/match.dart
  @override
  bool operator ==(Object other) {
    if (other.runtimeType != runtimeType) {
      return false;
    }
    return other is RouteMatchList &&
        uri == other.uri &&
        extra == other.extra &&
        error == other.error &&
        const ListEquality<RouteMatchBase>().equals(matches, other.matches) &&
        const MapEquality<String, String>().equals(
          pathParameters,
          other.pathParameters,
        );
  }
```

两个 `RouteMatchList` 相等当且仅当：

- 类型相同
- `uri` 相等
- `extra` 相等
- `error` 相等
- `matches` 列表相等（使用 `ListEquality` 进行深度比较）
- `pathParameters` 映射相等（使用 `MapEquality` 进行比较）

### hashCode

```dart 877:891:lib/src/match.dart
  @override
  int get hashCode {
    return Object.hash(
      Object.hashAll(matches),
      uri,
      extra,
      error,
      Object.hashAllUnordered(
        pathParameters.entries.map<int>(
          (MapEntry<String, String> entry) =>
              Object.hash(entry.key, entry.value),
        ),
      ),
    );
  }
```

哈希码计算考虑了所有相关字段：

- 使用 `Object.hashAll` 计算 `matches` 的哈希
- 使用 `Object.hashAllUnordered` 计算 `pathParameters` 的哈希（因为 Map 是无序的）
- 其他字段直接参与哈希计算

## 调试支持

### debugFillProperties

```dart 893:900:lib/src/match.dart
  @override
  void debugFillProperties(DiagnosticPropertiesBuilder properties) {
    super.debugFillProperties(properties);
    properties.add(DiagnosticsProperty<Uri>('uri', uri));
    properties.add(
      DiagnosticsProperty<List<RouteMatchBase>>('matches', matches),
    );
  }
```

为 Flutter 的调试工具提供属性信息，用于在调试时显示 `uri` 和 `matches`。

## 使用场景

### 1. 路由导航历史管理

`RouteMatchList` 维护了 `GoRouter` 的导航历史，记录了用户访问的所有路由：

```dart
// 获取当前路由
final currentRoute = routeMatchList.last.route;

// 检查是否有历史记录
if (routeMatchList.matches.length > 1) {
  // 可以执行返回操作
}
```

### 2. 路径参数访问

通过 `pathParameters` 可以访问当前路由的路径参数：

```dart
// 假设当前路径是 /user/123/profile
final userId = routeMatchList.pathParameters['id']; // "123"
```

### 3. 路由匹配遍历

使用 `visitRouteMatches` 可以遍历所有路由匹配：

```dart
routeMatchList.visitRouteMatches((match) {
  print('Route: ${match.route.path}');
  return true; // 继续遍历
});
```

### 4. 路由推送和弹出

`push` 和 `remove` 方法用于管理导航栈：

```dart
// 推送新路由
final newList = routeMatchList.push(imperativeRouteMatch);

// 移除路由（弹出）
final poppedList = routeMatchList.remove(targetMatch);
```

## 设计模式

`RouteMatchList` 采用了以下设计模式：

1. **不可变对象模式**：所有修改操作都返回新实例
2. **值对象模式**：通过 `==` 和 `hashCode` 实现值语义
3. **访问者模式**：`visitRouteMatches` 方法允许外部访问匹配列表
4. **组合模式**：通过 `ShellRouteMatch` 支持嵌套的路由匹配结构

## 注意事项

1. **不可变性**：`RouteMatchList` 是不可变的，所有修改操作都会创建新实例
2. **`ImperativeRouteMatch` 的特殊性**：这种匹配不贡献路径，在生成 `fullPath` 和 `uri` 时会被忽略
3. **递归结构**：`ShellRouteMatch` 内部的 `matches` 形成了递归结构，相关方法需要递归处理
4. **路径参数同步**：当移除路由时，需要同步更新 `pathParameters` 和 `uri`，确保它们与 `fullPath` 一致

## 总结

`RouteMatchList` 是 `go_router` 中管理路由匹配的核心数据结构，它：

- 维护了路由导航的完整历史
- 支持嵌套的路由结构（通过 `ShellRouteMatch`）
- 提供了丰富的查询和修改方法
- 确保了路径参数和 URI 的一致性
- 实现了不可变对象模式，提高了安全性和可预测性

理解这个类对于深入理解 `go_router` 的工作机制非常重要。
