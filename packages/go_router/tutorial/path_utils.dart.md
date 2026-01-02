# path_utils.dart 文件详解

## 概述

`path_utils.dart` 是 `go_router` 包中用于处理路由路径的工具文件。它提供了路径模式转换、参数提取、路径拼接和 URI 规范化等核心功能。这些工具函数是路由匹配、路径构建和 URI 处理的基础。

## 核心正则表达式

文件开头定义了一个用于匹配路径参数的正则表达式：

```dart 8:8:lib/src/path_utils.dart
final RegExp _parameterRegExp = RegExp(r':(\w+)(\((?:\\.|[^\\()])+\))?');
```

这个正则表达式用于识别路径模式中的参数，例如 `/user/:id` 中的 `:id`。表达式包含两个捕获组：

1. **`(\w+)`**：匹配参数名称（如 `id`、`userId` 等）
2. **`(\((?:\\.|[^\\()])+\))?`**：可选的正则表达式模式（用于自定义参数匹配规则）

例如，路径 `/user/:id(\d+)` 中，`id` 是参数名，`\d+` 是自定义匹配模式（只匹配数字）。

## 路径模式转正则表达式

### patternToRegExp 函数

这是文件中最核心的函数，用于将路径模式（如 `/user/:id`）转换为可用的正则表达式。

```dart 26:55:lib/src/path_utils.dart
RegExp patternToRegExp(
  String pattern,
  List<String> parameters, {
  required bool caseSensitive,
}) {
  final buffer = StringBuffer('^');
  var start = 0;
  for (final RegExpMatch match in _parameterRegExp.allMatches(pattern)) {
    if (match.start > start) {
      buffer.write(RegExp.escape(pattern.substring(start, match.start)));
    }
    final String name = match[1]!;
    final String? optionalPattern = match[2];
    final String regex = optionalPattern != null
        ? _escapeGroup(optionalPattern, name)
        : '(?<$name>[^/]+)';
    buffer.write(regex);
    parameters.add(name);
    start = match.end;
  }

  if (start < pattern.length) {
    buffer.write(RegExp.escape(pattern.substring(start)));
  }

  if (!pattern.endsWith('/')) {
    buffer.write(r'(?=/|$)');
  }
  return RegExp(buffer.toString(), caseSensitive: caseSensitive);
}
```

#### 功能说明

1. **构建正则表达式**：使用 `StringBuffer` 从 `^`（字符串开始）开始构建正则表达式
2. **处理路径参数**：
   - 遍历模式中的所有参数（使用 `_parameterRegExp`）
   - 对于每个参数前的普通文本，使用 `RegExp.escape` 进行转义
   - 将参数转换为命名捕获组，默认匹配一个或多个非斜杠字符 `[^/]+`
   - 如果参数有自定义模式（如 `:id(\d+)`），则使用 `_escapeGroup` 处理
3. **处理剩余文本**：将参数后的剩余路径部分添加到正则表达式中
4. **处理路径结尾**：如果路径不以 `/` 结尾，添加 `(?=/|$)` 以确保匹配在路径段边界或字符串结尾处结束

#### 使用示例

```dart
List<String> params = [];
RegExp regex = patternToRegExp('/user/:id/book/:bookId', params, caseSensitive: true);
// regex: ^/user/(?<id>[^/]+)/book/(?<bookId>[^/]+)(?=/|$)
// params: ['id', 'bookId']
```

### _escapeGroup 辅助函数

这是一个私有辅助函数，用于转义参数的自定义正则表达式模式：

```dart 57:66:lib/src/path_utils.dart
String _escapeGroup(String group, [String? name]) {
  final String escapedGroup = group.replaceFirstMapped(
    RegExp(r'[:=!]'),
    (Match match) => '\\${match[0]}',
  );
  if (name != null) {
    return '(?<$name>$escapedGroup)';
  }
  return escapedGroup;
}
```

#### 功能说明

1. **转义特殊字符**：转义模式中的 `:`、`=` 和 `!` 字符（这些字符在路由系统中有特殊含义）
2. **创建命名捕获组**：如果提供了参数名，将模式包装为命名捕获组

#### 使用示例

```dart
// 输入: '(\d+)', 'id'
// 输出: '(?<id>\\d+)'
```

## 路径重建

### patternToPath 函数

该函数根据路径模式和路径参数值重建完整路径：

```dart 80:96:lib/src/path_utils.dart
String patternToPath(String pattern, Map<String, String> pathParameters) {
  final buffer = StringBuffer();
  var start = 0;
  for (final RegExpMatch match in _parameterRegExp.allMatches(pattern)) {
    if (match.start > start) {
      buffer.write(pattern.substring(start, match.start));
    }
    final String name = match[1]!;
    buffer.write(pathParameters[name]);
    start = match.end;
  }

  if (start < pattern.length) {
    buffer.write(pattern.substring(start));
  }
  return buffer.toString();
}
```

#### 功能说明

1. **遍历模式中的参数**：使用 `_parameterRegExp` 找到所有参数位置
2. **填充路径**：
   - 添加参数前的普通文本
   - 用参数映射中的值替换参数占位符
   - 添加参数后的剩余文本

#### 使用示例

```dart
String path = patternToPath('/user/:id/book/:bookId', {'id': '123', 'bookId': '456'});
// 结果: '/user/123/book/456'
```

### extractPathParameters 函数

从正则表达式匹配结果中提取路径参数：

```dart 102:110:lib/src/path_utils.dart
Map<String, String> extractPathParameters(
  List<String> parameters,
  RegExpMatch match,
) {
  return <String, String>{
    for (int i = 0; i < parameters.length; ++i)
      parameters[i]: match.namedGroup(parameters[i])!,
  };
}
```

#### 功能说明

1. **提取命名组**：遍历参数名称列表，从 `RegExpMatch` 中提取对应的命名组值
2. **构建参数映射**：返回一个 `Map<String, String>`，键为参数名，值为匹配到的参数值

#### 使用示例

```dart
// 假设 patternToRegExp 返回的 regex 匹配了 '/user/123/book/456'
// parameters = ['id', 'bookId']
// match 是匹配结果

Map<String, String> params = extractPathParameters(parameters, match);
// 结果: {'id': '123', 'bookId': '456'}
```

## 路径拼接

### concatenatePaths 函数

拼接两个路径，自动处理斜杠和空段：

```dart 116:122:lib/src/path_utils.dart
String concatenatePaths(String parentPath, String childPath) {
  final Iterable<String> segments = <String>[
    ...parentPath.split('/'),
    ...childPath.split('/'),
  ].where((String segment) => segment.isNotEmpty);
  return '/${segments.join('/')}';
}
```

#### 功能说明

1. **分割路径**：将父路径和子路径都按 `/` 分割成段
2. **过滤空段**：移除所有空字符串段（处理多个连续斜杠的情况）
3. **重新组合**：用 `/` 连接所有段，并在开头添加 `/` 确保是绝对路径

#### 使用示例

```dart
concatenatePaths('/a', '/c/d');  // 结果: '/a/c/d'
concatenatePaths('a', 'c/d');    // 结果: '/a/c/d'
concatenatePaths('/a/', '/b/');  // 结果: '/a/b'
```

### concatenateUris 函数

拼接两个 URI，合并路径但只保留子 URI 的查询参数：

```dart 127:135:lib/src/path_utils.dart
Uri concatenateUris(Uri parentUri, Uri childUri) {
  Uri newUri = childUri.replace(
    path: concatenatePaths(parentUri.path, childUri.path),
  );

  // Parse the new normalized uri to remove unnecessary parts, like the trailing '?'.
  newUri = Uri.parse(canonicalUri(newUri.toString()));
  return newUri;
}
```

#### 功能说明

1. **合并路径**：使用 `concatenatePaths` 合并父 URI 和子 URI 的路径部分
2. **保留子 URI 参数**：使用 `replace` 方法只更改路径，保留子 URI 的查询参数和片段
3. **规范化结果**：使用 `canonicalUri` 规范化最终的 URI 字符串

#### 使用示例

```dart
// pathA = /a?fid=f1, pathB = c/d?pid=p2
// 结果: /a/c/d?pid=p2（父 URI 的查询参数被丢弃）
```

## URI 规范化

### canonicalUri 函数

规范化 URI 字符串，处理尾随斜杠、查询参数格式等问题：

```dart 138:171:lib/src/path_utils.dart
String canonicalUri(String loc) {
  if (loc.isEmpty) {
    throw GoException('Location cannot be empty.');
  }
  var canon = Uri.parse(loc).toString();
  canon = canon.endsWith('?') ? canon.substring(0, canon.length - 1) : canon;
  final Uri uri = Uri.parse(canon);

  // remove trailing slash except for when you shouldn't, e.g.
  // /profile/ => /profile
  // / => /
  // /login?from=/ => /login?from=/
  canon =
      uri.path.endsWith('/') &&
          uri.path != '/' &&
          !uri.hasQuery &&
          !uri.hasFragment
      ? canon.substring(0, canon.length - 1)
      : canon;

  // replace '/?', except for first occurrence, from path only
  // /login/?from=/ => /login?from=/
  // /?from=/ => /?from=/
  final int pathStartIndex = uri.host.isNotEmpty
      ? uri.toString().indexOf(uri.host) + uri.host.length
      : uri.hasScheme
      ? uri.toString().indexOf(uri.scheme) + uri.scheme.length
      : 0;
  if (pathStartIndex < canon.length) {
    canon = canon.replaceFirst('/?', '?', pathStartIndex + 1);
  }

  return canon;
}
```

#### 功能说明

该函数执行以下规范化步骤：

1. **验证输入**：如果位置字符串为空，抛出 `GoException`
2. **移除尾随问号**：移除 URI 末尾的单独 `?` 字符
3. **处理尾随斜杠**：
   - 移除路径末尾的 `/`，但保留根路径 `/`
   - 如果有查询参数或片段，不移除尾随斜杠（因为斜杠可能是查询参数值的一部分）
4. **修正路径中的 `/?`**：
   - 在路径部分（而非查询参数部分）将 `/?` 替换为 `?`
   - 但保留第一个出现的 `/?`（作为查询参数的开始）

#### 规范化规则示例

```dart
canonicalUri('/profile/');      // => '/profile'
canonicalUri('/');               // => '/'
canonicalUri('/login?from=/');   // => '/login?from=/'（保留斜杠，因为它在查询参数中）
canonicalUri('/login/?from=/');  // => '/login?from=/'（移除路径中的尾随斜杠）
canonicalUri('/?query=value');   // => '/?query=value'（保留第一个 /?）
```

## 路由路径构建

### fullPathForRoute 函数

递归查找并构建指定路由的完整绝对路径：

```dart 174:198:lib/src/path_utils.dart
String? fullPathForRoute(
  RouteBase targetRoute,
  String parentFullpath,
  List<RouteBase> routes,
) {
  for (final route in routes) {
    final String fullPath = (route is GoRoute)
        ? concatenatePaths(parentFullpath, route.path)
        : parentFullpath;

    if (route == targetRoute) {
      return fullPath;
    } else {
      final String? subRoutePath = fullPathForRoute(
        targetRoute,
        fullPath,
        route.routes,
      );
      if (subRoutePath != null) {
        return subRoutePath;
      }
    }
  }
  return null;
}
```

#### 功能说明

1. **遍历路由列表**：检查当前级别的每个路由
2. **构建当前路径**：
   - 如果是 `GoRoute`，使用 `concatenatePaths` 将父路径与当前路由路径拼接
   - 如果是其他类型的路由（如 `ShellRoute`），保持父路径不变
3. **匹配检查**：如果当前路由是目标路由，返回构建的完整路径
4. **递归查找**：如果不是目标路由，递归查找其子路由
5. **返回结果**：找到返回路径，未找到返回 `null`

#### 使用场景

这个函数主要用于根据路由配置树查找某个路由的完整路径，例如在命名路由导航时，需要知道路由的完整路径模式。

## 总结

`path_utils.dart` 提供了 `go_router` 包中路径处理的核心功能：

1. **模式转换**：将用户友好的路径模式转换为正则表达式
2. **参数处理**：提取和重建路径参数
3. **路径操作**：拼接和规范化路径
4. **URI 处理**：规范化 URI 字符串，确保一致性
5. **路由查找**：在路由树中查找并构建完整路径

这些工具函数协同工作，为 `go_router` 提供了强大而灵活的路径匹配和构建能力。
