# RouteConfiguration 调试检查方法

## 概述

`RouteConfiguration` 类包含多个静态和实例方法，用于在开发模式下验证路由配置的正确性。这些方法通过 `assert` 语句执行，只在 debug 模式下生效，确保路由配置符合 go_router 的规范要求。

所有的调试检查都在 `_onRoutingTableChanged` 方法中被调用，当路由配置发生变化时自动执行验证。

## 路径格式检查

```dart 39:56:lib/src/configuration.dart
static bool _debugCheckPath(List<RouteBase> routes, bool isTopLevel) {
  for (final route in routes) {
    late bool subRouteIsTopLevel;
    if (route is GoRoute) {
      if (route.path != '/') {
        assert(
          !route.path.endsWith('/'),
          'route path may not end with "/" except for the top "/" route. Found: $route',
        );
      }
      subRouteIsTopLevel = false;
    } else if (route is ShellRouteBase) {
      subRouteIsTopLevel = isTopLevel;
    }
    _debugCheckPath(route.routes, subRouteIsTopLevel);
  }
  return true;
}
```

`_debugCheckPath` 是一个静态递归方法，用于检查路由路径的格式是否正确。

### 检查规则

1. **尾随斜杠检查**：除了根路径 `/` 之外，所有 `GoRoute` 的路径不能以 `/` 结尾。
   - 允许：`/users`, `/users/:id`
   - 不允许：`/users/`, `/users/:id/`

2. **递归检查**：方法会递归检查所有子路由，确保整个路由树都符合规范。

### 参数说明

- **`routes`**：要检查的路由列表
- **`isTopLevel`**：布尔值，表示当前路由是否为顶层路由

### 逻辑流程

1. 遍历路由列表中的每个路由
2. 对于 `GoRoute` 类型：
   - 如果路径不是 `/`，检查是否以 `/` 结尾（不允许）
   - 将 `subRouteIsTopLevel` 设置为 `false`（子路由不再是顶层）
3. 对于 `ShellRouteBase` 类型：
   - 保持 `subRouteIsTopLevel` 的值与 `isTopLevel` 相同（Shell 路由的子路由保持顶层状态）
4. 递归检查每个路由的子路由列表

这个检查确保了路由路径格式的一致性，避免了因路径格式问题导致的路由匹配失败。

## 父导航器键验证

```dart 58:112:lib/src/configuration.dart
// Check that each parentNavigatorKey refers to either a ShellRoute's
// navigatorKey or the root navigator key.
static bool _debugCheckParentNavigatorKeys(
  List<RouteBase> routes,
  List<GlobalKey<NavigatorState>> allowedKeys,
) {
  for (final route in routes) {
    if (route is GoRoute) {
      final GlobalKey<NavigatorState>? parentKey = route.parentNavigatorKey;
      if (parentKey != null) {
        // Verify that the root navigator or a ShellRoute ancestor has a
        // matching navigator key.
        assert(
          allowedKeys.contains(parentKey),
          'parentNavigatorKey $parentKey must refer to'
          " an ancestor ShellRoute's navigatorKey or GoRouter's"
          ' navigatorKey',
        );

        _debugCheckParentNavigatorKeys(
          route.routes,
          <GlobalKey<NavigatorState>>[
            // Once a parentNavigatorKey is used, only that navigator key
            // or keys above it can be used.
            ...allowedKeys.sublist(0, allowedKeys.indexOf(parentKey) + 1),
          ],
        );
      } else {
        _debugCheckParentNavigatorKeys(
          route.routes,
          <GlobalKey<NavigatorState>>[...allowedKeys],
        );
      }
    } else if (route is ShellRoute) {
      _debugCheckParentNavigatorKeys(
        route.routes,
        <GlobalKey<NavigatorState>>[...allowedKeys, route.navigatorKey],
      );
    } else if (route is StatefulShellRoute) {
      for (final StatefulShellBranch branch in route.branches) {
        assert(
          !allowedKeys.contains(branch.navigatorKey),
          'StatefulShellBranch must not reuse an ancestor navigatorKey '
          '(${branch.navigatorKey})',
        );

        _debugCheckParentNavigatorKeys(
          branch.routes,
          <GlobalKey<NavigatorState>>[...allowedKeys, branch.navigatorKey],
        );
      }
    }
  }
  return true;
}
```

`_debugCheckParentNavigatorKeys` 是一个静态递归方法，用于验证 `GoRoute` 的 `parentNavigatorKey` 配置是否正确。

### 检查规则

1. **有效性检查**：每个 `GoRoute` 的 `parentNavigatorKey`（如果设置）必须引用以下之一：
   - 根导航器的键（`GoRouter` 的 `navigatorKey`）
   - 祖先 `ShellRoute` 的 `navigatorKey`

2. **作用域限制**：一旦某个路由使用了 `parentNavigatorKey`，其子路由只能使用该键或更上层的键，不能使用下层的键。

3. **StatefulShellBranch 唯一性**：`StatefulShellBranch` 的 `navigatorKey` 不能与任何祖先的导航器键重复。

### 参数说明

- **`routes`**：要检查的路由列表
- **`allowedKeys`**：允许使用的导航器键列表（按层级顺序排列）

### 逻辑流程

1. **GoRoute 处理**：
   - 如果路由设置了 `parentNavigatorKey`：
     - 验证该键在 `allowedKeys` 列表中
     - 递归检查子路由，但只允许使用该键及其上层的键（通过 `sublist` 限制）
   - 如果未设置 `parentNavigatorKey`：
     - 递归检查子路由，允许使用所有当前的 `allowedKeys`

2. **ShellRoute 处理**：
   - 将 `ShellRoute` 的 `navigatorKey` 添加到 `allowedKeys` 列表
   - 递归检查子路由，传递更新后的 `allowedKeys`

3. **StatefulShellRoute 处理**：
   - 遍历所有分支（`branches`）
   - 验证分支的 `navigatorKey` 不与 `allowedKeys` 中的任何键重复
   - 将分支的 `navigatorKey` 添加到 `allowedKeys`，递归检查分支的子路由

这个检查确保了导航器层级关系的正确性，防止了无效的导航器键引用导致的导航错误。

## 路径参数重复检查

```dart 114:135:lib/src/configuration.dart
static bool _debugVerifyNoDuplicatePathParameter(
  List<RouteBase> routes,
  Map<String, GoRoute> usedPathParams,
) {
  for (final route in routes) {
    if (route is! GoRoute) {
      continue;
    }
    for (final String pathParam in route.pathParameters) {
      if (usedPathParams.containsKey(pathParam)) {
        final sameRoute = usedPathParams[pathParam] == route;
        throw GoError(
          "duplicate path parameter, '$pathParam' found in ${sameRoute ? '$route' : '${usedPathParams[pathParam]}, and $route'}",
        );
    }
      usedPathParams[pathParam] = route;
    }
    _debugVerifyNoDuplicatePathParameter(route.routes, usedPathParams);
    route.pathParameters.forEach(usedPathParams.remove);
  }
  return true;
}
```

`_debugVerifyNoDuplicatePathParameter` 是一个静态递归方法，用于检查路由树中是否存在重复的路径参数名称。

### 检查规则

在同一个路由层级中，不同的路由不能使用相同的路径参数名称。例如：

- 允许：父路由 `/users/:id` 和子路由 `/users/:id/posts/:postId`（不同层级）
- 不允许：兄弟路由 `/users/:id` 和 `/users/:name`（如果 `id` 和 `name` 在同一层级，虽然名称不同但会被检查）

实际上，这个检查主要防止在同一个父路由下，不同的子路由使用相同的参数名。

### 参数说明

- **`routes`**：要检查的路由列表
- **`usedPathParams`**：已使用的路径参数映射（参数名 -> 路由对象）

### 逻辑流程

1. 遍历路由列表，只处理 `GoRoute` 类型
2. 对于每个路径参数：
   - 检查该参数名是否已在 `usedPathParams` 中使用
   - 如果已使用，抛出 `GoError` 异常，指出重复的参数名和涉及的路由
   - 将参数名和当前路由添加到 `usedPathParams` 中
3. 递归检查子路由
4. **关键步骤**：在递归返回后，从 `usedPathParams` 中移除当前路由的所有路径参数

第 4 步是关键：通过移除当前路由的参数，确保参数名的作用域仅限于当前路由及其子路由，允许不同分支的路由使用相同的参数名（只要它们不在同一层级）。

这个检查确保了路径参数名称的唯一性，避免了参数名冲突导致的解析错误。

## StatefulShellBranch 默认位置检查

```dart 137:188:lib/src/configuration.dart
// Check to see that the configured initialLocation of StatefulShellBranches
// points to a descendant route of the route branch.
bool _debugCheckStatefulShellBranchDefaultLocations(List<RouteBase> routes) {
  for (final route in routes) {
    if (route is StatefulShellRoute) {
      for (final StatefulShellBranch branch in route.branches) {
        if (branch.initialLocation == null) {
          // Recursively search for the first GoRoute descendant. Will
          // throw assertion error if not found.
          final GoRoute? defaultGoRoute = branch.defaultRoute;
          final String? initialLocation = defaultGoRoute != null
              ? locationForRoute(defaultGoRoute)
              : null;
          assert(
            initialLocation != null,
            'The default location of a StatefulShellBranch must be '
            'derivable from GoRoute descendant',
          );
          assert(
            defaultGoRoute!.pathParameters.isEmpty,
            'The default location of a StatefulShellBranch cannot be '
            'a parameterized route',
          );
        } else {
          final RouteMatchList matchList = findMatch(
            Uri.parse(branch.initialLocation!),
          );
          assert(
            !matchList.isError,
            'initialLocation (${matchList.uri}) of StatefulShellBranch must '
            'be a valid location',
          );
          final List<RouteBase> matchRoutes = matchList.routes;
          final int shellIndex = matchRoutes.indexOf(route);
          var matchFound = false;
          if (shellIndex >= 0 && (shellIndex + 1) < matchRoutes.length) {
            final RouteBase branchRoot = matchRoutes[shellIndex + 1];
            matchFound = branch.routes.contains(branchRoot);
          }
          assert(
            matchFound,
            'The initialLocation (${branch.initialLocation}) of '
            'StatefulShellBranch must match a descendant route of the '
            'branch',
          );
        }
      }
    }
    _debugCheckStatefulShellBranchDefaultLocations(route.routes);
  }
  return true;
}
```

`_debugCheckStatefulShellBranchDefaultLocations` 是一个实例方法，用于验证 `StatefulShellBranch` 的 `initialLocation` 配置是否正确。

### 检查规则

1. **未设置 initialLocation 的情况**：
   - 分支必须能够找到一个默认的 `GoRoute` 子路由
   - 默认路由不能包含路径参数（必须是具体的路径）

2. **设置了 initialLocation 的情况**：
   - `initialLocation` 必须是一个有效的路由位置（能够成功匹配）
   - `initialLocation` 必须匹配该分支下的某个子路由

### 逻辑流程

#### 情况 1：initialLocation 为 null

1. 获取分支的默认路由（`branch.defaultRoute`）
2. 如果默认路由存在，使用 `locationForRoute` 获取其位置
3. 验证默认位置不为 null
4. 验证默认路由不包含路径参数（因为参数化路由无法作为默认位置）

#### 情况 2：initialLocation 已设置

1. 使用 `findMatch` 尝试匹配 `initialLocation`
2. 验证匹配结果不是错误（`!matchList.isError`）
3. 在匹配的路由列表中找到 `StatefulShellRoute` 的位置（`shellIndex`）
4. 获取 `StatefulShellRoute` 之后的第一个路由（应该是分支的根路由）
5. 验证该路由是否在分支的 `routes` 列表中

这个检查确保了 `StatefulShellBranch` 的初始位置配置正确，避免了导航到无效位置的问题。

## 错误路由匹配列表

```dart 190:203:lib/src/configuration.dart
/// The match used when there is an error during parsing.
static RouteMatchList _errorRouteMatchList(
  Uri uri,
  GoException exception, {
  Object? extra,
}) {
  return RouteMatchList(
    matches: const <RouteMatch>[],
    extra: extra,
    error: exception,
    uri: uri,
    pathParameters: const <String, String>{},
  );
}
```

`_errorRouteMatchList` 是一个静态辅助方法，用于创建一个表示错误的 `RouteMatchList` 对象。

### 用途

当路由解析过程中发生错误时（如找不到匹配的路由、重定向异常等），可以使用这个方法创建一个包含错误信息的 `RouteMatchList`，用于后续的错误处理。

### 参数说明

- **`uri`**：发生错误的 URI
- **`exception`**：`GoException` 类型的异常对象
- **`extra`**：可选的额外数据

### 返回的 RouteMatchList 特性

- `matches`：空列表（因为没有成功匹配的路由）
- `error`：包含异常信息
- `uri`：原始的 URI
- `pathParameters`：空映射
- `extra`：传递的额外数据

这个方法提供了一个统一的方式来创建错误状态的路由匹配结果，简化了错误处理逻辑。

## 总结

这些调试检查方法确保了路由配置的正确性和一致性：

- **路径格式检查**：确保路径格式符合规范
- **导航器键验证**：确保导航器层级关系正确
- **路径参数检查**：防止参数名冲突
- **StatefulShellBranch 验证**：确保分支初始位置有效
- **错误处理辅助**：提供统一的错误状态创建方式

这些检查在开发阶段帮助开发者及时发现配置错误，提高了开发体验和代码质量。
