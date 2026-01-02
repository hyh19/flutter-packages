# all_extension_types.dart - Extension Types 路由详解

本文档详细讲解 `all_extension_types.dart` 文件中各种 Extension Types 路由的实现，包括基础数据类型和枚举类型的 Extension Types 路由处理。

## 概述

Extension Types 路由展示了如何在 `go_router_builder` 中使用 Extension Types 作为路由参数类型。与直接使用基础类型不同，Extension Types 提供了更强的类型安全性和语义化封装，同时保持零运行时开销。

## Extension Types 路由特点

1. **类型封装**：通过 Extension Types 封装底层类型，提供更清晰的语义
2. **值访问**：通过 `.value` 属性访问底层类型的值
3. **类型安全**：编译时类型检查，防止类型混淆
4. **零成本**：Extension Types 在编译时被擦除，不产生运行时开销

## 基础数据类型 Extension Types 路由

### BigIntExtensionRoute

`BigIntExtensionRoute` 展示了如何处理 `BigIntExtension` 类型。

```dart 62:83:example/lib/all_extension_types.dart
class BigIntExtensionRoute extends GoRouteData with $BigIntExtensionRoute {
  const BigIntExtensionRoute({
    required this.requiredBigIntField,
    this.bigIntField,
  });

  final BigIntExtension requiredBigIntField;
  final BigIntExtension? bigIntField;

  @override
  Widget build(BuildContext context, GoRouterState state) => BasePage<BigInt>(
    dataTitle: 'BigIntExtensionRoute',
    param: requiredBigIntField.value,
    queryParam: bigIntField?.value,
  );

  Widget drawerTile(BuildContext context) => ListTile(
    title: const Text('BigIntExtensionRoute'),
    onTap: () => go(context),
    selected: GoRouterState.of(context).uri.path == location,
  );
}
```

#### BigIntExtensionRoute 特点

- **路径参数**：`requiredBigIntField` 是必需的 `BigIntExtension` 类型参数
- **查询参数**：`bigIntField` 是可选的 `BigIntExtension?` 类型
- **值访问**：通过 `.value` 访问底层 `BigInt` 值
- **URL 路径**：`/big-int-route/:requiredBigIntField`

#### BigIntExtension 定义

```dart 51:51:example/lib/all_extension_types.dart
extension type const BigIntExtension(BigInt value) {}
```

#### BigIntExtensionRoute 使用示例

```dart 337:340:example/lib/all_extension_types.dart
          BigIntExtensionRoute(
            requiredBigIntField: BigIntExtension(BigInt.two),
            bigIntField: BigIntExtension(BigInt.zero),
          ).drawerTile(context),
```

### BoolExtensionRoute

`BoolExtensionRoute` 展示了如何处理 `BoolExtension` 类型，包括带默认值的参数。

```dart 85:109:example/lib/all_extension_types.dart
class BoolExtensionRoute extends GoRouteData with $BoolExtensionRoute {
  const BoolExtensionRoute({
    required this.requiredBoolField,
    this.boolField,
    this.boolFieldWithDefaultValue = const BoolExtension(true),
  });

  final BoolExtension requiredBoolField;
  final BoolExtension? boolField;
  final BoolExtension boolFieldWithDefaultValue;

  @override
  Widget build(BuildContext context, GoRouterState state) => BasePage<bool>(
    dataTitle: 'BoolExtensionRoute',
    param: requiredBoolField.value,
    queryParam: boolField?.value,
    queryParamWithDefaultValue: boolFieldWithDefaultValue.value,
  );

  Widget drawerTile(BuildContext context) => ListTile(
    title: const Text('BoolExtensionRoute'),
    onTap: () => go(context),
    selected: GoRouterState.of(context).uri.path == location,
  );
}
```

#### BoolExtensionRoute 特点

- **必需参数**：`requiredBoolField` 是必需的 `BoolExtension` 类型
- **可选参数**：`boolField` 是可空的 `BoolExtension?` 类型
- **默认值参数**：`boolFieldWithDefaultValue` 有默认值 `const BoolExtension(true)`
- **URL 路径**：`/bool-route/:requiredBoolField`

#### BoolExtension 定义

```dart 52:52:example/lib/all_extension_types.dart
extension type const BoolExtension(bool value) {}
```

#### 三种参数模式

`BoolExtensionRoute` 展示了三种不同的参数处理模式：

1. **必需路径参数**：`required BoolExtension requiredBoolField`
2. **可选查询参数**：`BoolExtension? boolField`
3. **带默认值的查询参数**：`BoolExtension boolFieldWithDefaultValue = const BoolExtension(true)`

### DateTimeExtensionRoute

`DateTimeExtensionRoute` 展示了如何处理 `DateTimeExtension` 类型。

```dart 111:132:example/lib/all_extension_types.dart
class DateTimeExtensionRoute extends GoRouteData with $DateTimeExtensionRoute {
  const DateTimeExtensionRoute({
    required this.requiredDateTimeField,
    this.dateTimeField,
  });

  final DateTimeExtension requiredDateTimeField;
  final DateTimeExtension? dateTimeField;

  @override
  Widget build(BuildContext context, GoRouterState state) => BasePage<DateTime>(
    dataTitle: 'DateTimeExtensionRoute',
    param: requiredDateTimeField.value,
    queryParam: dateTimeField?.value,
  );

  Widget drawerTile(BuildContext context) => ListTile(
    title: const Text('DateTimeExtensionRoute'),
    onTap: () => go(context),
    selected: GoRouterState.of(context).uri.path == location,
  );
}
```

#### DateTimeExtensionRoute 特点

- **路径参数**：`requiredDateTimeField` 是必需的 `DateTimeExtension` 类型
- **查询参数**：`dateTimeField` 是可选的 `DateTimeExtension?` 类型
- **URL 路径**：`/date-time-route/:requiredDateTimeField`
- **序列化**：`DateTime` 在 URL 中通常被序列化为 ISO 8601 格式字符串

#### DateTimeExtension 定义

```dart 53:53:example/lib/all_extension_types.dart
extension type const DateTimeExtension(DateTime value) {}
```

### DoubleExtensionRoute

`DoubleExtensionRoute` 展示了如何处理 `DoubleExtension` 类型，包含默认值示例。

```dart 134:158:example/lib/all_extension_types.dart
class DoubleExtensionRoute extends GoRouteData with $DoubleExtensionRoute {
  const DoubleExtensionRoute({
    required this.requiredDoubleField,
    this.doubleField,
    this.doubleFieldWithDefaultValue = const DoubleExtension(1.0),
  });

  final DoubleExtension requiredDoubleField;
  final DoubleExtension? doubleField;
  final DoubleExtension doubleFieldWithDefaultValue;

  @override
  Widget build(BuildContext context, GoRouterState state) => BasePage<double>(
    dataTitle: 'DoubleExtensionRoute',
    param: requiredDoubleField.value,
    queryParam: doubleField?.value,
    queryParamWithDefaultValue: doubleFieldWithDefaultValue.value,
  );

  Widget drawerTile(BuildContext context) => ListTile(
    title: const Text('DoubleExtensionRoute'),
    onTap: () => go(context),
    selected: GoRouterState.of(context).uri.path == location,
  );
}
```

#### DoubleExtensionRoute 特点

- **路径参数**：`requiredDoubleField` 是必需的 `DoubleExtension` 类型
- **可选参数**：`doubleField` 是可空的 `DoubleExtension?` 类型
- **默认值**：`doubleFieldWithDefaultValue` 默认值为 `const DoubleExtension(1.0)`
- **URL 路径**：`/double-route/:requiredDoubleField`

#### DoubleExtension 定义

```dart 54:54:example/lib/all_extension_types.dart
extension type const DoubleExtension(double value) {}
```

### IntExtensionRoute

`IntExtensionRoute` 展示了如何处理 `IntExtension` 类型。

```dart 160:184:example/lib/all_extension_types.dart
class IntExtensionRoute extends GoRouteData with $IntExtensionRoute {
  const IntExtensionRoute({
    required this.requiredIntField,
    this.intField,
    this.intFieldWithDefaultValue = const IntExtension(1),
  });

  final IntExtension requiredIntField;
  final IntExtension? intField;
  final IntExtension intFieldWithDefaultValue;

  @override
  Widget build(BuildContext context, GoRouterState state) => BasePage<int>(
    dataTitle: 'IntExtensionRoute',
    param: requiredIntField.value,
    queryParam: intField?.value,
    queryParamWithDefaultValue: intFieldWithDefaultValue.value,
  );

  Widget drawerTile(BuildContext context) => ListTile(
    title: const Text('IntExtensionRoute'),
    onTap: () => go(context),
    selected: GoRouterState.of(context).uri.path == location,
  );
}
```

#### IntExtensionRoute 特点

- **路径参数**：`requiredIntField` 是必需的 `IntExtension` 类型
- **可选参数**：`intField` 是可空的 `IntExtension?` 类型
- **默认值**：`intFieldWithDefaultValue` 默认值为 `const IntExtension(1)`
- **URL 路径**：`/int-route/:requiredIntField`

#### IntExtension 定义

```dart 55:55:example/lib/all_extension_types.dart
extension type const IntExtension(int value) {}
```

### NumExtensionRoute

`NumExtensionRoute` 展示了如何处理 `NumExtension` 类型（`int` 和 `double` 的父类）。

```dart 186:210:example/lib/all_extension_types.dart
class NumExtensionRoute extends GoRouteData with $NumExtensionRoute {
  const NumExtensionRoute({
    required this.requiredNumField,
    this.numField,
    this.numFieldWithDefaultValue = const NumExtension(1),
  });

  final NumExtension requiredNumField;
  final NumExtension? numField;
  final NumExtension numFieldWithDefaultValue;

  @override
  Widget build(BuildContext context, GoRouterState state) => BasePage<num>(
    dataTitle: 'NumExtensionRoute',
    param: requiredNumField.value,
    queryParam: numField?.value,
    queryParamWithDefaultValue: numFieldWithDefaultValue.value,
  );

  Widget drawerTile(BuildContext context) => ListTile(
    title: const Text('NumExtensionRoute'),
    onTap: () => go(context),
    selected: GoRouterState.of(context).uri.path == location,
  );
}
```

#### NumExtensionRoute 特点

- **类型灵活性**：`NumExtension` 可以接受 `int` 或 `double` 值
- **路径参数**：`requiredNumField` 是必需的 `NumExtension` 类型
- **可选参数**：`numField` 是可空的 `NumExtension?` 类型
- **默认值**：`numFieldWithDefaultValue` 默认值为 `const NumExtension(1)`
- **URL 路径**：`/num-route/:requiredNumField`

#### NumExtension 定义

```dart 56:56:example/lib/all_extension_types.dart
extension type const NumExtension(num value) {}
```

### StringExtensionRoute

`StringExtensionRoute` 展示了如何处理 `StringExtension` 类型，包括特殊字符的处理。

```dart 271:295:example/lib/all_extension_types.dart
class StringExtensionRoute extends GoRouteData with $StringExtensionRoute {
  const StringExtensionRoute({
    required this.requiredStringField,
    this.stringField,
    this.stringFieldWithDefaultValue = const StringExtension('defaultValue'),
  });

  final StringExtension requiredStringField;
  final StringExtension? stringField;
  final StringExtension stringFieldWithDefaultValue;

  @override
  Widget build(BuildContext context, GoRouterState state) => BasePage<String>(
    dataTitle: 'StringExtensionRoute',
    param: requiredStringField.value,
    queryParam: stringField?.value,
    queryParamWithDefaultValue: stringFieldWithDefaultValue.value,
  );

  Widget drawerTile(BuildContext context) => ListTile(
    title: const Text('StringExtensionRoute'),
    onTap: () => go(context),
    selected: GoRouterState.of(context).uri.path == location,
  );
}
```

#### StringExtensionRoute 特点

- **路径参数**：`requiredStringField` 是必需的 `StringExtension` 类型
- **可选参数**：`stringField` 是可空的 `StringExtension?` 类型
- **默认值**：`stringFieldWithDefaultValue` 默认值为 `const StringExtension('defaultValue')`
- **URL 路径**：`/string-route/:requiredStringField`
- **URL 编码**：字符串中的特殊字符会自动进行 URL 编码

#### StringExtension 定义

```dart 57:57:example/lib/all_extension_types.dart
extension type const StringExtension(String value) {}
```

#### StringExtensionRoute 使用示例

```dart 361:364:example/lib/all_extension_types.dart
          const StringExtensionRoute(
            requiredStringField: StringExtension(r'$!/#bob%%20'),
            stringField: StringExtension(r'$!/#bob%%20'),
          ).drawerTile(context),
```

这里使用了原始字符串（`r''`），包含特殊字符，演示了 URL 编码的处理。

### UriExtensionRoute

`UriExtensionRoute` 展示了如何处理 `UriExtension` 类型。

```dart 297:315:example/lib/all_extension_types.dart
class UriExtensionRoute extends GoRouteData with $UriExtensionRoute {
  const UriExtensionRoute({required this.requiredUriField, this.uriField});

  final UriExtension requiredUriField;
  final UriExtension? uriField;

  @override
  Widget build(BuildContext context, GoRouterState state) => BasePage<Uri>(
    dataTitle: 'UriExtensionRoute',
    param: requiredUriField.value,
    queryParam: uriField?.value,
  );

  Widget drawerTile(BuildContext context) => ListTile(
    title: const Text('UriExtensionRoute'),
    onTap: () => go(context),
    selected: GoRouterState.of(context).uri.path == location,
  );
}
```

#### UriExtensionRoute 特点

- **路径参数**：`requiredUriField` 是必需的 `UriExtension` 类型
- **查询参数**：`uriField` 是可选的 `UriExtension?` 类型
- **URL 路径**：`/uri-route/:requiredUriField`
- **序列化**：`Uri` 对象会被序列化为字符串形式存储在 URL 中

#### UriExtension 定义

```dart 58:58:example/lib/all_extension_types.dart
extension type const UriExtension(Uri value) {}
```

#### UriExtensionRoute 使用示例

```dart 375:378:example/lib/all_extension_types.dart
          UriExtensionRoute(
            requiredUriField: UriExtension(Uri.parse('https://dart.dev')),
            uriField: UriExtension(Uri.parse('https://dart.dev')),
          ).drawerTile(context),
```

## 枚举类型 Extension Types 路由

### EnumExtensionRoute

`EnumExtensionRoute` 展示了如何处理普通枚举类型的 Extension Type（`PersonDetailsExtension`）。

```dart 212:239:example/lib/all_extension_types.dart
class EnumExtensionRoute extends GoRouteData with $EnumExtensionRoute {
  const EnumExtensionRoute({
    required this.requiredEnumField,
    this.enumField,
    this.enumFieldWithDefaultValue = const PersonDetailsExtension(
      PersonDetails.favoriteFood,
    ),
  });

  final PersonDetailsExtension requiredEnumField;
  final PersonDetailsExtension? enumField;
  final PersonDetailsExtension enumFieldWithDefaultValue;

  @override
  Widget build(BuildContext context, GoRouterState state) =>
      BasePage<PersonDetails>(
        dataTitle: 'EnumExtensionRoute',
        param: requiredEnumField.value,
        queryParam: enumField?.value,
        queryParamWithDefaultValue: enumFieldWithDefaultValue.value,
      );

  Widget drawerTile(BuildContext context) => ListTile(
    title: const Text('EnumExtensionRoute'),
    onTap: () => go(context),
    selected: GoRouterState.of(context).uri.path == location,
  );
}
```

#### EnumExtensionRoute 特点

- **路径参数**：`requiredEnumField` 是必需的 `PersonDetailsExtension` 类型
- **可选参数**：`enumField` 是可空的 `PersonDetailsExtension?` 类型
- **默认值**：`enumFieldWithDefaultValue` 默认值为 `const PersonDetailsExtension(PersonDetails.favoriteFood)`
- **URL 路径**：`/enum-route/:requiredEnumField`

#### PersonDetailsExtension 定义

```dart 59:59:example/lib/all_extension_types.dart
extension type const PersonDetailsExtension(PersonDetails value) {}
```

#### EnumExtensionRoute 使用示例

```dart 365:370:example/lib/all_extension_types.dart
          const EnumExtensionRoute(
            requiredEnumField: PersonDetailsExtension(
              PersonDetails.favoriteSport,
            ),
            enumField: PersonDetailsExtension(PersonDetails.favoriteFood),
          ).drawerTile(context),
```

### EnhancedEnumExtensionRoute

`EnhancedEnumExtensionRoute` 展示了如何处理增强枚举类型的 Extension Type（`SportDetailsExtension`）。

```dart 241:269:example/lib/all_extension_types.dart
class EnhancedEnumExtensionRoute extends GoRouteData
    with $EnhancedEnumExtensionRoute {
  const EnhancedEnumExtensionRoute({
    required this.requiredEnumField,
    this.enumField,
    this.enumFieldWithDefaultValue = const SportDetailsExtension(
      SportDetails.football,
    ),
  });

  final SportDetailsExtension requiredEnumField;
  final SportDetailsExtension? enumField;
  final SportDetailsExtension enumFieldWithDefaultValue;

  @override
  Widget build(BuildContext context, GoRouterState state) =>
      BasePage<SportDetails>(
        dataTitle: 'EnhancedEnumExtensionRoute',
        param: requiredEnumField.value,
        queryParam: enumField?.value,
        queryParamWithDefaultValue: enumFieldWithDefaultValue.value,
      );

  Widget drawerTile(BuildContext context) => ListTile(
    title: const Text('EnhancedEnumExtensionRoute'),
    onTap: () => go(context),
    selected: GoRouterState.of(context).uri.path == location,
  );
}
```

#### EnhancedEnumExtensionRoute 特点

- **路径参数**：`requiredEnumField` 是必需的 `SportDetailsExtension` 类型
- **可选参数**：`enumField` 是可空的 `SportDetailsExtension?` 类型
- **默认值**：`enumFieldWithDefaultValue` 默认值为 `const SportDetailsExtension(SportDetails.football)`
- **URL 路径**：`/enhanced-enum-route/:requiredEnumField`

#### SportDetailsExtension 定义

```dart 60:60:example/lib/all_extension_types.dart
extension type const SportDetailsExtension(SportDetails value) {}
```

#### EnhancedEnumExtensionRoute 使用示例

```dart 371:374:example/lib/all_extension_types.dart
          const EnhancedEnumExtensionRoute(
            requiredEnumField: SportDetailsExtension(SportDetails.football),
            enumField: SportDetailsExtension(SportDetails.volleyball),
          ).drawerTile(context),
```

## Extension Types 使用模式

### 创建 Extension Types 实例

创建 Extension Types 实例时，传入底层类型的值：

```dart
// 基础类型
const boolExtension = BoolExtension(true);
const intExtension = IntExtension(42);
const stringExtension = StringExtension('hello');

// 枚举类型
const enumExtension = PersonDetailsExtension(PersonDetails.favoriteFood);
const enhancedEnumExtension = SportDetailsExtension(SportDetails.football);
```

### 访问底层值

通过 `.value` 属性访问 Extension Types 的底层值：

```dart
final boolExtension = BoolExtension(true);
final boolValue = boolExtension.value; // 获取 bool 值

final intExtension = IntExtension(42);
final intValue = intExtension.value; // 获取 int 值
```

### 默认值处理

Extension Types 支持 `const` 构造函数，可以在默认值中使用：

```dart
this.boolFieldWithDefaultValue = const BoolExtension(true),
this.intFieldWithDefaultValue = const IntExtension(1),
this.stringFieldWithDefaultValue = const StringExtension('defaultValue'),
```

### 可空 Extension Types

Extension Types 也可以是可空类型：

```dart
final BoolExtension? boolField; // 可空的 BoolExtension
final IntExtension? intField; // 可空的 IntExtension
```

访问可空 Extension Types 的值时，使用空安全操作符：

```dart
queryParam: boolField?.value, // 如果 boolField 为 null，则返回 null
```

## 通用模式总结

### 参数类型总结

所有 Extension Types 路由都遵循相同的参数模式：

1. **必需路径参数**：使用 `required` 关键字，类型为非空 Extension Type（如 `BigIntExtension`、`BoolExtension` 等）
2. **可选查询参数**：使用可空 Extension Type（如 `BigIntExtension?`、`BoolExtension?` 等）
3. **带默认值的查询参数**：使用非空 Extension Type + 默认值（如 `BoolExtension boolFieldWithDefaultValue = const BoolExtension(true)`）

### 值访问模式

所有路由类在 `build` 方法中都通过 `.value` 访问 Extension Types 的底层值：

```dart
param: requiredBigIntField.value,
queryParam: bigIntField?.value,
queryParamWithDefaultValue: boolFieldWithDefaultValue.value,
```

### drawerTile 方法

所有路由类都实现了 `drawerTile` 方法，用于在导航抽屉中显示列表项：

```dart 78:82:example/lib/all_extension_types.dart
  Widget drawerTile(BuildContext context) => ListTile(
    title: const Text('BigIntExtensionRoute'),
    onTap: () => go(context),
    selected: GoRouterState.of(context).uri.path == location,
  );
```

**功能说明**：

- **标题**：显示路由名称
- **点击事件**：调用 `go(context)` 进行导航
- **选中状态**：根据当前路径判断是否选中（高亮显示）

## URL 格式示例

以下是各种 Extension Types 路由的 URL 格式示例：

- **BigIntExtensionRoute**：`/big-int-route/12345678901234567890?bigIntField=987654321`
- **BoolExtensionRoute**：`/bool-route/true?boolField=false&boolFieldWithDefaultValue=false`
- **DateTimeExtensionRoute**：`/date-time-route/2023-05-20T10:30:00Z?dateTimeField=2023-05-21T12:00:00Z`
- **DoubleExtensionRoute**：`/double-route/3.14?doubleField=-3.14`
- **IntExtensionRoute**：`/int-route/42?intField=-42`
- **NumExtensionRoute**：`/num-route/2.71828?numField=-2.71828`
- **StringExtensionRoute**：`/string-route/hello?stringField=world`
- **UriExtensionRoute**：`/uri-route/https%3A%2F%2Fdart.dev?uriField=https%3A%2F%2Fflutter.dev`
- **EnumExtensionRoute**：`/enum-route/favorite-sport?enumField=favorite-food`
- **EnhancedEnumExtensionRoute**：`/enhanced-enum-route/football?enumField=volleyball`

## Extension Types vs 直接类型

### 类型安全性对比

**直接类型路由**：

```dart
class BigIntRoute {
  final BigInt requiredBigIntField;
  final BigInt? bigIntField;
}
```

**Extension Types 路由**：

```dart
class BigIntExtensionRoute {
  final BigIntExtension requiredBigIntField;
  final BigIntExtension? bigIntField;
}
```

Extension Types 提供了更强的类型区分能力，即使底层类型相同，不同的 Extension Types 也是不同的类型。

### 使用场景

使用 Extension Types 的场景：

- 需要为相同底层类型提供不同的语义
- 需要防止类型混淆
- 需要更清晰的 API 设计
- 需要零成本的类型抽象

## 总结

Extension Types 路由展示了 `go_router_builder` 对 Extension Types 的完整支持：

1. **类型封装**：通过 Extension Types 封装底层类型，提供更清晰的语义
2. **类型安全**：编译时类型检查，防止类型混淆
3. **灵活的参数处理**：支持必需参数、可选参数和带默认值的参数
4. **零成本抽象**：Extension Types 在编译时被擦除，不产生运行时开销
5. **值访问**：通过 `.value` 属性访问底层类型的值
6. **URL 序列化**：自动处理 Extension Types 在 URL 中的表示和解析

Extension Types 路由为类型安全的路由系统提供了更强的类型抽象能力，在保持性能的同时提供了更好的代码可读性和类型安全性。
