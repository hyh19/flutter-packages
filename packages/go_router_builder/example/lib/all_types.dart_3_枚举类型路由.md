# all_types.dart - 枚举类型路由

本文档详细讲解 `all_types.dart` 文件中涉及枚举类型的路由实现，包括普通枚举（`PersonDetails`）和增强枚举（Enhanced Enum，`SportDetails`）的处理方式。

## 概述

枚举类型路由展示了 `go_router_builder` 如何处理 Dart 中的枚举类型。Dart 2.17 引入了增强枚举（Enhanced Enum）特性，允许枚举值包含额外的数据，`go_router_builder` 对两种类型的枚举都提供了支持。

## 枚举类型定义

在 `shared/data.dart` 文件中定义了两种枚举类型：

### PersonDetails（普通枚举）

```dart
enum PersonDetails { hobbies, favoriteFood, favoriteSport }
```

这是一个标准的枚举类型，包含三个枚举值：`hobbies`、`favoriteFood`、`favoriteSport`。

### SportDetails（增强枚举）

增强枚举允许每个枚举值包含额外的属性和数据：

```dart
enum SportDetails {
  volleyball(
    imageUrl: '/sportdetails/url/volleyball.jpg',
    playerPerTeam: 6,
    accessory: null,
    hasNet: true,
  ),
  football(
    imageUrl: '/sportdetails/url/Football.jpg',
    playerPerTeam: 11,
    accessory: null,
    hasNet: true,
  ),
  tennis(
    imageUrl: '/sportdetails/url/tennis.jpg',
    playerPerTeam: 2,
    accessory: 'Rackets',
    hasNet: true,
  ),
  hockey(
    imageUrl: '/sportdetails/url/hockey.jpg',
    playerPerTeam: 6,
    accessory: 'Hockey sticks',
    hasNet: true,
  );

  const SportDetails({
    required this.accessory,
    required this.hasNet,
    required this.imageUrl,
    required this.playerPerTeam,
  });

  final String imageUrl;
  final int playerPerTeam;
  final String? accessory;
  final bool hasNet;
}
```

增强枚举的特点：

- 每个枚举值可以有自己的构造函数参数
- 枚举类可以定义 final 字段来存储这些数据
- 提供了更丰富的枚举值信息

## EnumRoute

`EnumRoute` 展示了如何处理普通枚举类型（`PersonDetails`）。

```dart 189:214:example/lib/all_types.dart
class EnumRoute extends GoRouteData with $EnumRoute {
  EnumRoute({
    required this.requiredEnumField,
    this.enumField,
    this.enumFieldWithDefaultValue = PersonDetails.favoriteFood,
  });

  final PersonDetails requiredEnumField;
  final PersonDetails? enumField;
  final PersonDetails enumFieldWithDefaultValue;

  @override
  Widget build(BuildContext context, GoRouterState state) =>
      BasePage<PersonDetails>(
        dataTitle: 'EnumRoute',
        param: requiredEnumField,
        queryParam: enumField,
        queryParamWithDefaultValue: enumFieldWithDefaultValue,
      );

  Widget drawerTile(BuildContext context) => ListTile(
    title: const Text('EnumRoute'),
    onTap: () => go(context),
    selected: GoRouterState.of(context).uri.path == location,
  );
}
```

### 特点

- **路径参数**：`requiredEnumField` 是必需的 `PersonDetails` 类型
- **可选参数**：`enumField` 是可空的 `PersonDetails?` 类型
- **默认值**：`enumFieldWithDefaultValue` 默认值为 `PersonDetails.favoriteFood`
- **URL 路径**：`/enum-route/:requiredEnumField`

### 路由定义

```dart 24:24:example/lib/all_types.dart
    TypedGoRoute<EnumRoute>(path: 'enum-route/:requiredEnumField'),
```

### 枚举序列化

普通枚举在 URL 中的序列化方式：

- 枚举值被转换为字符串形式
- 通常使用枚举值的名称（如 `hobbies`、`favoriteFood`）
- 某些情况下可能使用 kebab-case（如 `favorite-food`）

### 使用示例

```dart 477:480:example/lib/all_types.dart
          EnumRoute(
            requiredEnumField: PersonDetails.favoriteSport,
            enumField: PersonDetails.favoriteFood,
          ).drawerTile(context),
```

## EnhancedEnumRoute

`EnhancedEnumRoute` 展示了如何处理增强枚举类型（`SportDetails`）。

```dart 216:241:example/lib/all_types.dart
class EnhancedEnumRoute extends GoRouteData with $EnhancedEnumRoute {
  EnhancedEnumRoute({
    required this.requiredEnumField,
    this.enumField,
    this.enumFieldWithDefaultValue = SportDetails.football,
  });

  final SportDetails requiredEnumField;
  final SportDetails? enumField;
  final SportDetails enumFieldWithDefaultValue;

  @override
  Widget build(BuildContext context, GoRouterState state) =>
      BasePage<SportDetails>(
        dataTitle: 'EnhancedEnumRoute',
        param: requiredEnumField,
        queryParam: enumField,
        queryParamWithDefaultValue: enumFieldWithDefaultValue,
      );

  Widget drawerTile(BuildContext context) => ListTile(
    title: const Text('EnhancedEnumRoute'),
    onTap: () => go(context),
    selected: GoRouterState.of(context).uri.path == location,
  );
}
```

### 特点

- **路径参数**：`requiredEnumField` 是必需的 `SportDetails` 类型
- **可选参数**：`enumField` 是可空的 `SportDetails?` 类型
- **默认值**：`enumFieldWithDefaultValue` 默认值为 `SportDetails.football`
- **URL 路径**：`/enhanced-enum-route/:requiredEnumField`

### 路由定义

```dart 25:27:example/lib/all_types.dart
    TypedGoRoute<EnhancedEnumRoute>(
      path: 'enhanced-enum-route/:requiredEnumField',
    ),
```

### 增强枚举序列化

增强枚举在 URL 中的序列化方式：

- 与普通枚举类似，使用枚举值的名称
- 枚举值包含的额外数据（如 `imageUrl`、`playerPerTeam` 等）不会存储在 URL 中
- 这些额外数据在创建枚举实例时通过构造函数设置，而不是从 URL 中解析

### 使用示例

```dart 481:484:example/lib/all_types.dart
          EnhancedEnumRoute(
            requiredEnumField: SportDetails.football,
            enumField: SportDetails.volleyball,
          ).drawerTile(context),
```

## 普通枚举 vs 增强枚举

### 相同点

两种枚举类型在路由处理上的相同之处：

1. **参数模式相同**：都支持必需参数、可选参数和带默认值的参数
2. **URL 序列化**：在 URL 中都使用枚举值的名称（字符串形式）
3. **类型安全**：都提供编译时类型检查
4. **代码结构**：路由类的代码结构完全相同

### 不同点

主要区别在于枚举本身的定义和使用：

| 特性 | 普通枚举 | 增强枚举 |
| --- | --- | --- |
| **额外数据** | 不支持 | 支持，每个枚举值可以有属性 |
| **构造函数** | 不支持 | 支持自定义构造函数 |
| **字段定义** | 不支持 | 可以定义 final 字段 |
| **复杂性** | 简单 | 更复杂，但功能更强 |

### 路由处理差异

在路由处理层面，两种枚举类型的使用方式完全相同：

```dart
// 普通枚举
final PersonDetails personDetails = PersonDetails.favoriteFood;

// 增强枚举
final SportDetails sportDetails = SportDetails.football;
```

两者都可以：

- 作为路径参数
- 作为查询参数
- 设置默认值
- 在 URL 中序列化/反序列化

## 枚举在 URL 中的表示

### 序列化规则

枚举值在 URL 中的表示遵循以下规则：

1. **名称转换**：枚举值的名称（如 `favoriteFood`）可能需要转换为 URL 友好的格式（如 `favorite-food`）
2. **大小写处理**：通常使用小写或 kebab-case
3. **特殊字符**：可能需要 URL 编码

### 示例 URL

- **EnumRoute**：`/enum-route/favorite-sport?enumField=favorite-food`
- **EnhancedEnumRoute**：`/enhanced-enum-route/football?enumField=volleyball`

### 反序列化

从 URL 解析枚举值时：

1. 读取 URL 中的字符串（如 `favorite-food`）
2. 查找对应的枚举值（如 `PersonDetails.favoriteFood`）
3. 对于增强枚举，使用枚举值名称创建对应的枚举实例（附带的所有属性值都会被正确设置）

## 代码生成

代码生成器会为枚举类型创建映射表：

### PersonDetails 映射

生成的代码会创建类似以下的映射：

```dart
const _$PersonDetailsEnumMap = {
  PersonDetails.hobbies: 'hobbies',
  PersonDetails.favoriteFood: 'favorite-food',
  PersonDetails.favoriteSport: 'favorite-sport',
};
```

这个映射用于：

- **序列化**：将枚举值转换为 URL 字符串
- **反序列化**：将 URL 字符串转换回枚举值

### SportDetails 映射

增强枚举的映射方式相同：

```dart
const _$SportDetailsEnumMap = {
  SportDetails.volleyball: 'volleyball',
  SportDetails.football: 'football',
  SportDetails.tennis: 'tennis',
  SportDetails.hockey: 'hockey',
};
```

## 使用建议

### 何时使用普通枚举

使用普通枚举的场景：

- 只需要简单的命名常量
- 不需要为每个值存储额外数据
- 枚举值之间没有复杂的关系

示例：状态枚举、类型枚举、选项枚举

### 何时使用增强枚举

使用增强枚举的场景：

- 需要为每个枚举值存储额外信息
- 枚举值有属性需要访问
- 需要更丰富的枚举值表示

示例：错误码（包含错误消息）、配置选项（包含默认值）、UI 主题（包含颜色配置）

### 路由中的选择

在路由参数中使用枚举时：

- 两种枚举类型都可以使用
- 选择取决于业务需求，而非路由功能限制
- 增强枚举的额外数据不会影响路由的 URL 格式
- 路由只关心枚举值的标识（名称），不关心额外数据

## 总结

枚举类型路由展示了 `go_router_builder` 对枚举类型的完整支持：

1. **类型安全**：枚举值在编译时确定，避免字符串拼写错误
2. **灵活的参数处理**：支持必需参数、可选参数和带默认值的参数
3. **两种枚举支持**：普通枚举和增强枚举都可以使用
4. **URL 序列化**：自动处理枚举值在 URL 中的表示
5. **代码生成**：自动生成枚举值映射表，简化序列化/反序列化

无论是简单的状态枚举还是复杂的配置枚举，都可以在类型安全的路由系统中使用，提供了良好的开发体验和代码可维护性。
