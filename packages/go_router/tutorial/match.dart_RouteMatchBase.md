# RouteMatchBase 类详解

## 概述

`RouteMatchBase` 是 go_router 包中路由匹配系统的核心抽象基类。它定义了路由匹配结果的基本结构，并提供了静态方法来执行路由匹配算法。这个类负责将 URI 路径与路由配置进行匹配，生成对应的路由匹配对象。

## 类定义

```dart 28:31:lib/src/match.dart
/// The base class for various route matches.
abstract class RouteMatchBase with Diagnosticable {
  /// An abstract route match base
  const RouteMatchBase();
```

`RouteMatchBase` 是一个抽象类，混入了 `Diagnosticable` mixin，这使得它可以在 Flutter 的调试工具中显示诊断信息。

## 核心属性

### route

```dart 33:34:lib/src/match.dart
  /// The matched route.
  RouteBase get route;
```

返回匹配到的路由对象。这是一个抽象属性，由子类实现。

### pageKey

```dart 36:37:lib/src/match.dart
  /// The page key.
  ValueKey<String> get pageKey;
```

返回页面的唯一标识键，用于 Flutter 的页面识别和状态管理。

### matchedLocation

```dart 39:47:lib/src/match.dart
  /// The location string that matches the [route].
  ///
  /// for example:
  ///
  /// uri = '/family/f2/person/p2'
  /// route = GoRoute('/family/:id')
  ///
  /// matchedLocation = '/family/f2'
  String get matchedLocation;
```

返回与当前路由匹配的位置字符串。例如，如果 URI 是 `/family/f2/person/p2`，而路由是 `GoRoute('/family/:id')`，则 `matchedLocation` 为 `/family/f2`。

## 核心方法

### buildState

```dart 49:53:lib/src/match.dart
  /// Gets the state that represent this route match.
  GoRouterState buildState(
    RouteConfiguration configuration,
    RouteMatchList matches,
  );
```

根据路由匹配结果构建 `GoRouterState` 对象，该对象包含了路由状态的所有信息，供页面构建器使用。

### match（静态方法）

```dart 55:82:lib/src/match.dart
  /// Generates a list of [RouteMatchBase] objects by matching the `route` and
  /// its sub-routes with `uri`.
  ///
  /// This method returns empty list if it can't find a complete match in the
  /// `route`.
  ///
  /// The `rootNavigatorKey` is required to match routes with
  /// parentNavigatorKey.
  ///
  /// The extracted path parameters, as the result of the matching, are stored
  /// into `pathParameters`.
  static List<RouteMatchBase> match({
    required RouteBase route,
    required Map<String, String> pathParameters,
    required GlobalKey<NavigatorState> rootNavigatorKey,
    required Uri uri,
  }) {
    return _matchByNavigatorKey(
          route: route,
          matchedPath: '',
          remainingLocation: uri.path,
          matchedLocation: '',
          pathParameters: pathParameters,
          scopedNavigatorKey: rootNavigatorKey,
          uri: uri,
        )[null] ??
        const <RouteMatchBase>[];
  }
```

这是路由匹配的入口方法。它调用 `_matchByNavigatorKey` 进行实际的匹配工作，并返回匹配结果列表。如果无法找到完整匹配，则返回空列表。

**参数说明**：

- `route`: 要匹配的路由对象
- `pathParameters`: 用于存储提取的路径参数的 Map
- `rootNavigatorKey`: 根导航器的键，用于匹配具有 `parentNavigatorKey` 的路由
- `uri`: 要匹配的 URI 对象

**返回值**：匹配到的路由列表，如果未找到匹配则返回空列表。

## 内部匹配方法

### _matchByNavigatorKey

```dart 87:139:lib/src/match.dart
  /// Returns a navigator key to route matches maps.
  ///
  /// The null key corresponds to the route matches of `scopedNavigatorKey`.
  /// The scopedNavigatorKey must not be part of the returned map; otherwise,
  /// it is impossible to order the matches.
  static Map<GlobalKey<NavigatorState>?, List<RouteMatchBase>>
  _matchByNavigatorKey({
    required RouteBase route,
    required String matchedPath, // e.g. /family/:fid
    required String remainingLocation, // e.g. person/p1
    required String matchedLocation, // e.g. /family/f2
    required Map<String, String> pathParameters,
    required GlobalKey<NavigatorState> scopedNavigatorKey,
    required Uri uri,
  }) {
    final Map<GlobalKey<NavigatorState>?, List<RouteMatchBase>> result;
    if (route is ShellRouteBase) {
      result = _matchByNavigatorKeyForShellRoute(
        route: route,
        matchedPath: matchedPath,
        remainingLocation: remainingLocation,
        matchedLocation: matchedLocation,
        pathParameters: pathParameters,
        scopedNavigatorKey: scopedNavigatorKey,
        uri: uri,
      );
    } else if (route is GoRoute) {
      result = _matchByNavigatorKeyForGoRoute(
        route: route,
        matchedPath: matchedPath,
        remainingLocation: remainingLocation,
        matchedLocation: matchedLocation,
        pathParameters: pathParameters,
        scopedNavigatorKey: scopedNavigatorKey,
        uri: uri,
      );
    } else {
      assert(false, 'Unexpected route type: $route');
      return _empty;
    }
    // Grab the route matches for the scope navigator key and put it into the
    // matches for `null`.
    if (result.containsKey(scopedNavigatorKey)) {
      final List<RouteMatchBase> matchesForScopedNavigator = result.remove(
        scopedNavigatorKey,
      )!;
      assert(matchesForScopedNavigator.isNotEmpty);
      result
          .putIfAbsent(null, () => <RouteMatchBase>[])
          .addAll(matchesForScopedNavigator);
    }
    return result;
  }
```

这是路由匹配的核心分发方法。它根据路由类型（`ShellRouteBase` 或 `GoRoute`）调用相应的匹配方法。

**关键逻辑**：

1. **路由类型判断**：根据路由类型选择匹配策略
   - `ShellRouteBase`：调用 `_matchByNavigatorKeyForShellRoute`
   - `GoRoute`：调用 `_matchByNavigatorKeyForGoRoute`

2. **Navigator Key 处理**：返回的 Map 以 `GlobalKey<NavigatorState>?` 为键，对应的路由匹配列表为值
   - `null` 键对应 `scopedNavigatorKey` 的路由匹配
   - `scopedNavigatorKey` 本身不能出现在返回的 Map 中，否则无法正确排序匹配结果

3. **结果整理**：如果结果中包含 `scopedNavigatorKey`，将其移除并添加到 `null` 键下

### _matchByNavigatorKeyForShellRoute

```dart 141:192:lib/src/match.dart
  static Map<GlobalKey<NavigatorState>?, List<RouteMatchBase>>
  _matchByNavigatorKeyForShellRoute({
    required ShellRouteBase route,
    required String matchedPath, // e.g. /family/:fid
    required String remainingLocation, // e.g. person/p1
    required String matchedLocation, // e.g. /family/f2
    required Map<String, String> pathParameters,
    required GlobalKey<NavigatorState> scopedNavigatorKey,
    required Uri uri,
  }) {
    final GlobalKey<NavigatorState>? parentKey =
        route.parentNavigatorKey == scopedNavigatorKey
        ? null
        : route.parentNavigatorKey;
    Map<GlobalKey<NavigatorState>?, List<RouteMatchBase>>? subRouteMatches;
    late GlobalKey<NavigatorState> navigatorKeyUsed;
    for (final RouteBase subRoute in route.routes) {
      navigatorKeyUsed = route.navigatorKeyForSubRoute(subRoute);
      subRouteMatches = _matchByNavigatorKey(
        route: subRoute,
        matchedPath: matchedPath,
        remainingLocation: remainingLocation,
        matchedLocation: matchedLocation,
        pathParameters: pathParameters,
        uri: uri,
        scopedNavigatorKey: navigatorKeyUsed,
      );
      assert(
        !subRouteMatches.containsKey(route.navigatorKeyForSubRoute(subRoute)),
      );
      if (subRouteMatches.isNotEmpty) {
        break;
      }
    }
    if (subRouteMatches?.isEmpty ?? true) {
      return _empty;
    }
    final RouteMatchBase result = ShellRouteMatch(
      route: route,
      // The RouteConfiguration should have asserted the subRouteMatches must
      // have at least one match for this ShellRouteBase.
      matches: subRouteMatches!.remove(null)!,
      matchedLocation: remainingLocation,
      pageKey: ValueKey<String>(route.hashCode.toString()),
      navigatorKey: navigatorKeyUsed,
    );
    subRouteMatches
        .putIfAbsent(parentKey, () => <RouteMatchBase>[])
        .insert(0, result);

    return subRouteMatches;
  }
```

处理 `ShellRoute` 类型的路由匹配。

**匹配流程**：

1. **确定父导航器键**：如果路由的 `parentNavigatorKey` 等于 `scopedNavigatorKey`，则设为 `null`，否则使用路由的 `parentNavigatorKey`

2. **遍历子路由**：遍历 `ShellRoute` 的所有子路由，尝试匹配：
   - 为每个子路由获取对应的导航器键
   - 递归调用 `_matchByNavigatorKey` 进行匹配
   - 如果找到匹配，立即停止遍历（`break`）

3. **创建 ShellRouteMatch**：如果找到匹配的子路由：
   - 从子路由匹配结果中提取 `null` 键对应的匹配列表
   - 创建 `ShellRouteMatch` 对象
   - 将 `ShellRouteMatch` 插入到父导航器键对应的列表中

4. **返回结果**：返回按导航器键组织的匹配结果 Map

### _matchByNavigatorKeyForGoRoute

```dart 194:296:lib/src/match.dart
  static Map<GlobalKey<NavigatorState>?, List<RouteMatchBase>>
  _matchByNavigatorKeyForGoRoute({
    required GoRoute route,
    required String matchedPath, // e.g. /family/:fid
    required String remainingLocation, // e.g. person/p1
    required String matchedLocation, // e.g. /family/f2
    required Map<String, String> pathParameters,
    required GlobalKey<NavigatorState> scopedNavigatorKey,
    required Uri uri,
  }) {
    final GlobalKey<NavigatorState>? parentKey =
        route.parentNavigatorKey == scopedNavigatorKey
        ? null
        : route.parentNavigatorKey;

    final RegExpMatch? regExpMatch = route.matchPatternAsPrefix(
      remainingLocation,
    );

    if (regExpMatch == null) {
      return _empty;
    }
    final Map<String, String> encodedParams = route.extractPathParams(
      regExpMatch,
    );
    // A temporary map to hold path parameters. This map is merged into
    // pathParameters only when this route is part of the returned result.
    final Map<String, String> currentPathParameter = encodedParams
        .map<String, String>(
          (String key, String value) =>
              MapEntry<String, String>(key, Uri.decodeComponent(value)),
        );
    final String pathLoc = patternToPath(route.path, encodedParams);
    final String newMatchedLocation = concatenatePaths(
      matchedLocation,
      pathLoc,
    );
    final String newMatchedPath = concatenatePaths(matchedPath, route.path);

    final String newMatchedLocationToCompare;
    final String uriPathToCompare;
    if (route.caseSensitive) {
      newMatchedLocationToCompare = newMatchedLocation;
      uriPathToCompare = uri.path;
    } else {
      newMatchedLocationToCompare = newMatchedLocation.toLowerCase();
      uriPathToCompare = uri.path.toLowerCase();
    }
    if (newMatchedLocationToCompare == uriPathToCompare) {
      // A complete match.
      pathParameters.addAll(currentPathParameter);

      return <GlobalKey<NavigatorState>?, List<RouteMatchBase>>{
        parentKey: <RouteMatchBase>[
          RouteMatch(
            route: route,
            matchedLocation: newMatchedLocation,
            pageKey: ValueKey<String>(newMatchedPath),
          ),
        ],
      };
    }
    assert(uriPathToCompare.startsWith(newMatchedLocationToCompare));
    assert(remainingLocation.isNotEmpty);

    final String childRestLoc = uri.path.substring(
      newMatchedLocation.length + (newMatchedLocation == '/' ? 0 : 1),
    );

    Map<GlobalKey<NavigatorState>?, List<RouteMatchBase>>? subRouteMatches;
    for (final RouteBase subRoute in route.routes) {
      subRouteMatches = _matchByNavigatorKey(
        route: subRoute,
        matchedPath: newMatchedPath,
        remainingLocation: childRestLoc,
        matchedLocation: newMatchedLocation,
        pathParameters: pathParameters,
        uri: uri,
        scopedNavigatorKey: scopedNavigatorKey,
      );
      if (subRouteMatches.isNotEmpty) {
        break;
      }
    }
    if (subRouteMatches?.isEmpty ?? true) {
      // If not finding a sub route match, it is considered not matched for this
      // route even if this route match part of the `remainingLocation`.
      return _empty;
    }

    pathParameters.addAll(currentPathParameter);
    subRouteMatches!
        .putIfAbsent(parentKey, () => <RouteMatchBase>[])
        .insert(
          0,
          RouteMatch(
            route: route,
            matchedLocation: newMatchedLocation,
            pageKey: ValueKey<String>(newMatchedPath),
          ),
        );
    return subRouteMatches;
  }
```

处理 `GoRoute` 类型的路由匹配。这是最复杂的匹配逻辑。

**匹配流程详解**：

#### 1. 路径模式匹配

```dart 209:215:lib/src/match.dart
    final RegExpMatch? regExpMatch = route.matchPatternAsPrefix(
      remainingLocation,
    );

    if (regExpMatch == null) {
      return _empty;
    }
```

使用正则表达式匹配路径模式。如果无法匹配，直接返回空结果。

#### 2. 提取路径参数

```dart 216:225:lib/src/match.dart
    final Map<String, String> encodedParams = route.extractPathParams(
      regExpMatch,
    );
    // A temporary map to hold path parameters. This map is merged into
    // pathParameters only when this route is part of the returned result.
    final Map<String, String> currentPathParameter = encodedParams
        .map<String, String>(
          (String key, String value) =>
              MapEntry<String, String>(key, Uri.decodeComponent(value)),
        );
```

从匹配结果中提取路径参数（如 `:id`、`:name` 等），并对参数值进行 URI 解码。

#### 3. 构建匹配位置和路径

```dart 226:231:lib/src/match.dart
    final String pathLoc = patternToPath(route.path, encodedParams);
    final String newMatchedLocation = concatenatePaths(
      matchedLocation,
      pathLoc,
    );
    final String newMatchedPath = concatenatePaths(matchedPath, route.path);
```

- `pathLoc`：将路径模式转换为实际路径（替换参数）
- `newMatchedLocation`：累积的匹配位置
- `newMatchedPath`：累积的匹配路径模式

#### 4. 大小写敏感性处理

```dart 233:241:lib/src/match.dart
    final String newMatchedLocationToCompare;
    final String uriPathToCompare;
    if (route.caseSensitive) {
      newMatchedLocationToCompare = newMatchedLocation;
      uriPathToCompare = uri.path;
    } else {
      newMatchedLocationToCompare = newMatchedLocation.toLowerCase();
      uriPathToCompare = uri.path.toLowerCase();
    }
```

根据路由的 `caseSensitive` 属性决定是否进行大小写转换。

#### 5. 完全匹配检查

```dart 242:254:lib/src/match.dart
    if (newMatchedLocationToCompare == uriPathToCompare) {
      // A complete match.
      pathParameters.addAll(currentPathParameter);

      return <GlobalKey<NavigatorState>?, List<RouteMatchBase>>{
        parentKey: <RouteMatchBase>[
          RouteMatch(
            route: route,
            matchedLocation: newMatchedLocation,
            pageKey: ValueKey<String>(newMatchedPath),
          ),
        ],
      };
    }
```

如果匹配位置与 URI 路径完全一致，说明这是一个完整的匹配（没有子路由需要继续匹配）。此时：

- 将路径参数添加到全局参数 Map
- 创建 `RouteMatch` 对象并返回

#### 6. 子路由匹配

```dart 256:277:lib/src/match.dart
    assert(uriPathToCompare.startsWith(newMatchedLocationToCompare));
    assert(remainingLocation.isNotEmpty);

    final String childRestLoc = uri.path.substring(
      newMatchedLocation.length + (newMatchedLocation == '/' ? 0 : 1),
    );

    Map<GlobalKey<NavigatorState>?, List<RouteMatchBase>>? subRouteMatches;
    for (final RouteBase subRoute in route.routes) {
      subRouteMatches = _matchByNavigatorKey(
        route: subRoute,
        matchedPath: newMatchedPath,
        remainingLocation: childRestLoc,
        matchedLocation: newMatchedLocation,
        pathParameters: pathParameters,
        uri: uri,
        scopedNavigatorKey: scopedNavigatorKey,
      );
      if (subRouteMatches.isNotEmpty) {
        break;
      }
    }
```

如果当前路由只匹配了 URI 的一部分，需要继续匹配子路由：

1. **计算剩余位置**：从 URI 路径中提取未匹配的部分
2. **递归匹配子路由**：遍历所有子路由，递归调用 `_matchByNavigatorKey`
3. **找到匹配即停止**：一旦找到匹配的子路由，立即停止遍历

#### 7. 合并匹配结果

```dart 278:295:lib/src/match.dart
    if (subRouteMatches?.isEmpty ?? true) {
      // If not finding a sub route match, it is considered not matched for this
      // route even if this route match part of the `remainingLocation`.
      return _empty;
    }

    pathParameters.addAll(currentPathParameter);
    subRouteMatches!
        .putIfAbsent(parentKey, () => <RouteMatchBase>[])
        .insert(
          0,
          RouteMatch(
            route: route,
            matchedLocation: newMatchedLocation,
            pageKey: ValueKey<String>(newMatchedPath),
          ),
        );
    return subRouteMatches;
```

如果找到子路由匹配：

- 将当前路由的路径参数添加到全局参数 Map
- 将当前路由的 `RouteMatch` 插入到父导航器键对应的列表开头
- 返回合并后的匹配结果

**重要设计**：即使当前路由匹配了部分路径，如果没有找到匹配的子路由，整个匹配仍然失败。这确保了只有完整的路由链才能被匹配。

## 调试支持

```dart 298:302:lib/src/match.dart
  @override
  void debugFillProperties(DiagnosticPropertiesBuilder properties) {
    super.debugFillProperties(properties);
    properties.add(DiagnosticsProperty<RouteBase>('route', route));
  }
```

重写了 `debugFillProperties` 方法，在 Flutter 的调试工具中显示路由信息。

## 设计模式与算法

### 递归匹配算法

路由匹配采用深度优先搜索（DFS）的递归算法：

1. **从根路由开始**：从路由树的根节点开始匹配
2. **路径前缀匹配**：使用正则表达式检查当前路由是否匹配 URI 的前缀
3. **参数提取**：从匹配结果中提取路径参数
4. **递归子路由**：如果 URI 还有剩余部分，递归匹配子路由
5. **结果合并**：将匹配结果按导航器键组织并返回

### Navigator Key 的作用

`Navigator Key` 用于管理不同层级的导航栈：

- **根导航器**：使用 `rootNavigatorKey`
- **Shell 路由导航器**：每个 `ShellRoute` 可能有自己的导航器
- **父导航器**：路由可以指定 `parentNavigatorKey` 来指定父级导航器

匹配结果按导航器键分组，这样可以：

- 正确构建多层级导航栈
- 支持嵌套路由的独立导航历史
- 实现复杂的导航场景（如底部导航栏 + 页面内导航）

## 使用示例

假设有以下路由配置：

```dart
GoRoute(
  path: '/family/:fid',
  routes: [
    GoRoute(
      path: 'person/:pid',
    ),
  ],
)
```

当 URI 为 `/family/f2/person/p1` 时，匹配过程如下：

1. **第一层匹配**：`/family/:fid` 匹配 `/family/f2`
   - 提取参数：`fid = 'f2'`
   - 剩余位置：`person/p1`

2. **第二层匹配**：`person/:pid` 匹配 `person/p1`
   - 提取参数：`pid = 'p1'`
   - 完全匹配

3. **返回结果**：包含两个 `RouteMatch` 对象的列表

## 总结

`RouteMatchBase` 是 go_router 路由匹配系统的核心，它：

- 提供了路由匹配的抽象接口
- 实现了递归的路由匹配算法
- 支持路径参数提取
- 处理大小写敏感性
- 管理多层级导航器
- 支持 `GoRoute` 和 `ShellRoute` 两种路由类型

通过这个类，go_router 能够将 URI 路径转换为对应的路由匹配对象，为页面构建和导航提供基础。
