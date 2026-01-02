# path_and_query_parameters.dart 代码解析

## 概述

这是一个演示如何在 GoRouter 中使用**路径参数（Path Parameters）**和**查询参数（Query Parameters）**的完整示例。该示例展示了一个简单的家庭管理系统，用户可以在家庭列表和家庭成员列表之间导航，并通过查询参数控制排序方式。

## 核心概念

### 路径参数

路径参数是 URL 路径中的动态部分，在路由定义中使用 `:` 前缀标识。例如，路径 `'family/:fid'` 中的 `:fid` 就是一个路径参数，可以从 `GoRouterState.pathParameters` 中获取。

### 查询参数

查询参数是 URL 中 `?` 后面的键值对，例如 `?sort=asc`。GoRouter 会自动解析并存储在 `GoRouterState.queryParameters` 中。

## 数据模型

### Family 类

```dart 17:26:example/lib/path_and_query_parameters.dart
/// Family data class.
class Family {
  /// Create a family.
  const Family({required this.name, required this.people});

  /// The last name of the family.
  final String name;

  /// The people in the family.
  final Map<String, Person> people;
}
```

`Family` 类表示一个家庭，包含：

- `name`：家庭的姓氏
- `people`：家庭成员映射，键为人员 ID，值为 `Person` 对象

### Person 类

```dart 29:35:example/lib/path_and_query_parameters.dart
/// Person data class.
class Person {
  /// Creates a person.
  const Person({required this.name});

  /// The first name of the person.
  final String name;
}
```

`Person` 类表示一个人员，包含：

- `name`：人员的名字

### 示例数据

```dart 37:52:example/lib/path_and_query_parameters.dart
const Map<String, Family> _families = <String, Family>{
  'f1': Family(
    name: 'Doe',
    people: <String, Person>{
      'p1': Person(name: 'Jane'),
      'p2': Person(name: 'John'),
    },
  ),
  'f2': Family(
    name: 'Wong',
    people: <String, Person>{
      'p1': Person(name: 'June'),
      'p2': Person(name: 'Xin'),
    },
  ),
};
```

示例数据定义了两个家庭：

- `f1`：Doe 家庭，包含 Jane 和 John
- `f2`：Wong 家庭，包含 June 和 Xin

## 路由配置

### App 类

```dart 57:93:example/lib/path_and_query_parameters.dart
/// The main app.
class App extends StatelessWidget {
  /// Creates an [App].
  App({super.key});

  /// The title of the app.
  static const String title = 'GoRouter Example: Query Parameters';

  // add the login info into the tree as app state that can change over time
  @override
  Widget build(BuildContext context) => MaterialApp.router(
    routerConfig: _router,
    title: title,
    debugShowCheckedModeBanner: false,
  );

  late final GoRouter _router = GoRouter(
    routes: <GoRoute>[
      GoRoute(
        path: '/',
        builder: (BuildContext context, GoRouterState state) =>
            const HomeScreen(),
        routes: <GoRoute>[
          GoRoute(
            name: 'family',
            path: 'family/:fid',
            builder: (BuildContext context, GoRouterState state) {
              return FamilyScreen(
                fid: state.pathParameters['fid']!,
                asc: state.uri.queryParameters['sort'] == 'asc',
              );
            },
          ),
        ],
      ),
    ],
  );
}
```

路由配置包含两个路由：

1. **根路由** (`'/'`)：显示 `HomeScreen`，展示所有家庭的列表
2. **家庭路由** (`'family/:fid'`)：
   - 路径中包含路径参数 `:fid`（家庭 ID）
   - 从 `state.pathParameters['fid']` 获取路径参数值
   - 从 `state.uri.queryParameters['sort']` 获取查询参数，判断是否为升序排序
   - 使用命名路由 `name: 'family'`，方便后续使用 `context.goNamed()` 导航

### 路径参数的使用

在路由定义中，`path: 'family/:fid'` 中的 `:fid` 表示这是一个动态路径参数。当访问 `/family/f1` 时，`fid` 的值就是 `'f1'`。

### 查询参数的获取

查询参数通过 `state.uri.queryParameters` 访问。例如，URL `/family/f1?sort=asc` 中，`queryParameters['sort']` 的值为 `'asc'`。

## 屏幕组件

### HomeScreen

```dart 96:115:example/lib/path_and_query_parameters.dart
/// The home screen that shows a list of families.
class HomeScreen extends StatelessWidget {
  /// Creates a [HomeScreen].
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text(App.title)),
      body: ListView(
        children: <Widget>[
          for (final MapEntry<String, Family> entry in _families.entries)
            ListTile(
              title: Text(entry.value.name),
              onTap: () => context.go('/family/${entry.key}'),
            ),
        ],
      ),
    );
  }
}
```

`HomeScreen` 的功能：

- 显示所有家庭的列表
- 使用 `ListView` 和 `ListTile` 展示每个家庭
- 点击列表项时，使用 `context.go()` 导航到对应的家庭详情页
- 导航时通过字符串插值将家庭 ID 嵌入路径：`'/family/${entry.key}'`

### FamilyScreen

```dart 118:163:example/lib/path_and_query_parameters.dart
/// The screen that shows a list of persons in a family.
class FamilyScreen extends StatelessWidget {
  /// Creates a [FamilyScreen].
  const FamilyScreen({required this.fid, required this.asc, super.key});

  /// The family to display.
  final String fid;

  /// Whether to sort the name in ascending order.
  final bool asc;

  @override
  Widget build(BuildContext context) {
    final Map<String, String> newQueries;
    final List<String> names = _families[fid]!.people.values
        .map<String>((Person p) => p.name)
        .toList();
    names.sort();
    if (asc) {
      newQueries = const <String, String>{'sort': 'desc'};
    } else {
      newQueries = const <String, String>{'sort': 'asc'};
    }
    return Scaffold(
      appBar: AppBar(
        title: Text(_families[fid]!.name),
        actions: <Widget>[
          IconButton(
            onPressed: () => context.goNamed(
              'family',
              pathParameters: <String, String>{'fid': fid},
              queryParameters: newQueries,
            ),
            tooltip: 'sort ascending or descending',
            icon: const Icon(Icons.sort),
          ),
        ],
      ),
      body: ListView(
        children: <Widget>[
          for (final String name in asc ? names : names.reversed)
            ListTile(title: Text(name)),
        ],
      ),
    );
  }
}
```

`FamilyScreen` 的功能：

1. **接收参数**：
   - `fid`：家庭 ID（来自路径参数）
   - `asc`：是否升序排序（来自查询参数）

2. **数据处理**：
   - 从 `_families` 中获取对应家庭的所有成员
   - 提取成员名字并排序
   - 根据 `asc` 参数决定显示顺序（升序或降序）

3. **排序切换**：
   - AppBar 中包含一个排序按钮
   - 点击按钮时，使用 `context.goNamed()` 导航到同一路由，但切换查询参数
   - 如果当前是升序（`asc=true`），则切换到降序（`sort=desc`）
   - 如果当前是降序（`asc=false`），则切换到升序（`sort=asc`）

4. **使用命名路由的优势**：
   - 使用 `context.goNamed()` 而不是 `context.go()`，可以更清晰地传递参数
   - 通过 `pathParameters` 和 `queryParameters` 参数明确指定路径参数和查询参数

## 关键代码片段解析

### 路径参数的提取

```dart 84:84:example/lib/path_and_query_parameters.dart
fid: state.pathParameters['fid']!,
```

从 `GoRouterState` 的 `pathParameters` 映射中获取路径参数值。使用 `!` 断言是因为在路由匹配时，路径参数一定存在。

### 查询参数的解析

```dart 85:85:example/lib/path_and_query_parameters.dart
asc: state.uri.queryParameters['sort'] == 'asc',
```

从 `GoRouterState` 的 `uri.queryParameters` 中获取查询参数，并判断其值是否为 `'asc'`。

### 使用命名路由导航

```dart 145:149:example/lib/path_and_query_parameters.dart
onPressed: () => context.goNamed(
  'family',
  pathParameters: <String, String>{'fid': fid},
  queryParameters: newQueries,
),
```

使用 `context.goNamed()` 的好处：

- 使用路由名称而不是硬编码路径，提高可维护性
- 明确指定路径参数和查询参数，代码更清晰
- 类型安全，编译器可以检查参数是否正确

### 字符串路径导航

```dart 109:109:example/lib/path_and_query_parameters.dart
onTap: () => context.go('/family/${entry.key}'),
```

在 `HomeScreen` 中使用 `context.go()` 进行导航，通过字符串插值构建路径。这种方式简单直接，适合简单的导航场景。

## 使用场景总结

这个示例展示了以下 GoRouter 的核心功能：

1. **路径参数**：用于传递 URL 路径中的动态值（如资源 ID）
2. **查询参数**：用于传递可选的状态信息（如排序方式、筛选条件）
3. **命名路由**：提高代码可维护性和类型安全性
4. **参数访问**：通过 `GoRouterState` 访问路径参数和查询参数
5. **动态导航**：根据当前状态切换查询参数，实现状态切换

## 运行效果

1. 启动应用后，显示家庭列表（Doe 和 Wong）
2. 点击某个家庭，导航到 `/family/f1` 或 `/family/f2`
3. 显示该家庭的所有成员，默认按升序排列
4. 点击排序按钮，切换排序方式（升序/降序），URL 会相应更新为 `/family/f1?sort=desc` 或 `/family/f1?sort=asc`
