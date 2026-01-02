# named_routes.dart 代码解析

## 概述

这个文件展示了如何在 Flutter 应用中使用 GoRouter 的命名路由功能。与直接硬编码 URL 路径不同，命名路由允许开发者通过路由名称来导航，GoRouter 会自动将名称转换为实际的 URL 路径。这种方式提高了代码的可维护性和可读性。

## 核心概念

### 命名路由的优势

- **解耦路径与代码**：当路由路径发生变化时，只需修改路由配置，无需修改所有使用该路径的代码
- **类型安全**：通过名称引用路由，减少拼写错误
- **易于重构**：路径变更不会影响业务逻辑代码

## 数据结构

### Family 类

```dart 16:26:example/lib/named_routes.dart
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

- `name`：家庭的姓氏（String 类型）
- `people`：家庭成员映射（Map<String, Person>），键为人员 ID，值为 Person 对象

### Person 类

```dart 28:38:example/lib/named_routes.dart
/// Person data class.
class Person {
  /// Creates a person.
  const Person({required this.name, required this.age});

  /// The first name of the person.
  final String name;

  /// The age of the person.
  final int age;
}
```

`Person` 类表示一个人，包含：

- `name`：名字（String 类型）
- `age`：年龄（int 类型）

### 示例数据

```dart 40:55:example/lib/named_routes.dart
const Map<String, Family> _families = <String, Family>{
  'f1': Family(
    name: 'Doe',
    people: <String, Person>{
      'p1': Person(name: 'Jane', age: 23),
      'p2': Person(name: 'John', age: 6),
    },
  ),
  'f2': Family(
    name: 'Wong',
    people: <String, Person>{
      'p1': Person(name: 'June', age: 51),
      'p2': Person(name: 'Xin', age: 44),
    },
  ),
};
```

示例数据包含两个家庭：

- `f1`：Doe 家庭，包含 Jane（23 岁）和 John（6 岁）
- `f2`：Wong 家庭，包含 June（51 岁）和 Xin（44 岁）

## 路由配置

### GoRouter 配置

```dart 74:104:example/lib/named_routes.dart
  late final GoRouter _router = GoRouter(
    debugLogDiagnostics: true,
    routes: <GoRoute>[
      GoRoute(
        name: 'home',
        path: '/',
        builder: (BuildContext context, GoRouterState state) =>
            const HomeScreen(),
        routes: <GoRoute>[
          GoRoute(
            name: 'family',
            path: 'family/:fid',
            builder: (BuildContext context, GoRouterState state) =>
                FamilyScreen(fid: state.pathParameters['fid']!),
            routes: <GoRoute>[
              GoRoute(
                name: 'person',
                path: 'person/:pid',
                builder: (BuildContext context, GoRouterState state) {
                  return PersonScreen(
                    fid: state.pathParameters['fid']!,
                    pid: state.pathParameters['pid']!,
                  );
                },
              ),
            ],
          ),
        ],
      ),
    ],
  );
```

路由结构采用嵌套设计：

1. **home 路由**（名称：`'home'`）
   - 路径：`/`
   - 对应屏幕：`HomeScreen`
   - 子路由：family

2. **family 路由**（名称：`'family'`）
   - 路径：`family/:fid`（`:fid` 是路径参数，表示家庭 ID）
   - 对应屏幕：`FamilyScreen`
   - 从 `state.pathParameters['fid']` 获取家庭 ID
   - 子路由：person

3. **person 路由**（名称：`'person'`）
   - 路径：`person/:pid`（`:pid` 是路径参数，表示人员 ID）
   - 对应屏幕：`PersonScreen`
   - 需要同时获取 `fid` 和 `pid` 两个路径参数

### 路由命名的重要性

每个 `GoRoute` 都设置了 `name` 属性，这是使用命名路由 API 的关键。通过名称，可以在代码中引用路由，而不需要知道具体的路径结构。

## 屏幕组件

### HomeScreen - 首页

```dart 107:132:example/lib/named_routes.dart
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
              onTap: () => context.go(
                context.namedLocation(
                  'family',
                  pathParameters: <String, String>{'fid': entry.key},
                ),
              ),
            ),
        ],
      ),
    );
  }
}
```

**功能说明**：

- 显示所有家庭的列表
- 每个列表项显示家庭姓氏
- 点击列表项时，使用 `context.namedLocation()` 方法生成路由地址
- 传入路由名称 `'family'` 和路径参数 `{'fid': entry.key}`
- 使用 `context.go()` 进行导航

**关键 API**：

- `context.namedLocation('family', pathParameters: {...})`：根据路由名称和参数生成 URL
- `context.go(...)`：导航到指定路径

### FamilyScreen - 家庭成员列表页

```dart 134:167:example/lib/named_routes.dart
/// The screen that shows a list of persons in a family.
class FamilyScreen extends StatelessWidget {
  /// Creates a [FamilyScreen].
  const FamilyScreen({required this.fid, super.key});

  /// The id family to display.
  final String fid;

  @override
  Widget build(BuildContext context) {
    final Map<String, Person> people = _families[fid]!.people;
    return Scaffold(
      appBar: AppBar(title: Text(_families[fid]!.name)),
      body: ListView(
        children: <Widget>[
          for (final MapEntry<String, Person> entry in people.entries)
            ListTile(
              title: Text(entry.value.name),
              onTap: () => context.go(
                context.namedLocation(
                  'person',
                  pathParameters: <String, String>{
                    'fid': fid,
                    'pid': entry.key,
                  },
                  queryParameters: <String, String>{'qid': 'quid'},
                ),
              ),
            ),
        ],
      ),
    );
  }
}
```

**功能说明**：

- 显示指定家庭的所有成员列表
- 接收 `fid`（家庭 ID）作为构造参数
- 从 `_families` 中获取对应家庭的数据
- 点击成员列表项时，导航到 `person` 路由
- 传递多个路径参数：`fid` 和 `pid`
- 还演示了查询参数的使用：`queryParameters: {'qid': 'quid'}`

**关键点**：

- 嵌套路由需要传递父路由的参数（`fid`）
- 可以同时传递路径参数和查询参数

### PersonScreen - 个人详情页

```dart 169:189:example/lib/named_routes.dart
/// The person screen.
class PersonScreen extends StatelessWidget {
  /// Creates a [PersonScreen].
  const PersonScreen({required this.fid, required this.pid, super.key});

  /// The id of family this person belong to.
  final String fid;

  /// The id of the person to be displayed.
  final String pid;

  @override
  Widget build(BuildContext context) {
    final Family family = _families[fid]!;
    final Person person = family.people[pid]!;
    return Scaffold(
      appBar: AppBar(title: Text(person.name)),
      body: Text('${person.name} ${family.name} is ${person.age} years old'),
    );
  }
}
```

**功能说明**：

- 显示个人的详细信息
- 接收 `fid` 和 `pid` 两个参数
- 从数据源中查找并显示对应的人员信息
- 显示格式：`{名字} {姓氏} is {年龄} years old`

## 命名路由 API 详解

### context.namedLocation() 方法

这是使用命名路由的核心方法，语法如下：

```dart
context.namedLocation(
  'routeName',                    // 路由名称
  pathParameters: {...},          // 路径参数（可选）
  queryParameters: {...},         // 查询参数（可选）
)
```

**参数说明**：

- `routeName`：在 `GoRoute` 中定义的 `name` 属性值
- `pathParameters`：路径参数映射，对应路径中的 `:paramName`
- `queryParameters`：查询参数映射，会附加到 URL 后面（如 `?key=value`）

### 使用示例对比

**使用命名路由**（推荐）：

```dart
context.go(
  context.namedLocation(
    'person',
    pathParameters: {'fid': 'f1', 'pid': 'p1'},
  ),
);
```

**直接使用路径**（不推荐）：

```dart
context.go('/family/f1/person/p1');
```

命名路由的优势在于：

- 路径变更时只需修改路由配置
- 代码更易读，意图更明确
- 减少硬编码路径导致的错误

## 路由层级关系

```text
home (/)
  └── family (/family/:fid)
      └── person (/family/:fid/person/:pid)
```

这是一个三层嵌套的路由结构：

1. 首页显示家庭列表
2. 点击家庭进入家庭成员列表页
3. 点击成员进入个人详情页

## 应用入口

```dart 57:72:example/lib/named_routes.dart
void main() => runApp(App());

/// The main app.
class App extends StatelessWidget {
  /// Creates an [App].
  App({super.key});

  /// The title of the app.
  static const String title = 'GoRouter Example: Named Routes';

  @override
  Widget build(BuildContext context) => MaterialApp.router(
    routerConfig: _router,
    title: title,
    debugShowCheckedModeBanner: false,
  );
```

应用使用 `MaterialApp.router` 并配置 `routerConfig` 来使用 GoRouter。

## 总结

这个示例完整展示了：

1. **如何定义命名路由**：在 `GoRoute` 中设置 `name` 属性
2. **如何使用命名路由导航**：通过 `context.namedLocation()` 生成路径
3. **如何处理路径参数**：在 `namedLocation` 中传递 `pathParameters`
4. **如何处理查询参数**：在 `namedLocation` 中传递 `queryParameters`
5. **嵌套路由的使用**：子路由需要传递父路由的参数

命名路由是 GoRouter 提供的一个强大功能，它让路由管理更加灵活和可维护，特别适合大型应用的路由管理需求。
