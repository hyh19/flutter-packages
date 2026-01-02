# RouteMatchListCodec 类说明

## 概述

`RouteMatchListCodec` 是一个用于处理 `RouteMatchList` 对象编码和解码的类。它实现了 Dart 的 `Codec` 接口，将 `RouteMatchList` 对象转换为适合使用 `StandardMessageCodec` 的 `Map<Object?, Object?>` 格式。

该类的主要用途是**状态恢复**和**浏览器历史记录**。通过序列化和反序列化路由匹配信息，go_router 可以在应用重启后恢复之前的导航状态，或者在 Web 平台上与浏览器的历史记录 API 集成。

## 类定义

```dart 903:930:lib/src/match.dart
/// Handles encoding and decoding of [RouteMatchList] objects to a format
/// suitable for using with [StandardMessageCodec].
///
/// The primary use of this class is for state restoration and browser history.
// TODO(loic-sharma): Remove meta library prefix.
// https://github.com/flutter/flutter/issues/171410
@meta.internal
class RouteMatchListCodec extends Codec<RouteMatchList, Map<Object?, Object?>> {
  /// Creates a new [RouteMatchListCodec] object.
  RouteMatchListCodec(RouteConfiguration configuration)
    : decoder = _RouteMatchListDecoder(configuration),
      encoder = _RouteMatchListEncoder(configuration);

  static const String _locationKey = 'location';
  static const String _extraKey = 'state';
  static const String _imperativeMatchesKey = 'imperativeMatches';
  static const String _pageKey = 'pageKey';
  static const String _codecKey = 'codec';
  static const String _jsonCodecName = 'json';
  static const String _customCodecName = 'custom';
  static const String _encodedKey = 'encoded';

  @override
  final Converter<RouteMatchList, Map<Object?, Object?>> encoder;

  @override
  final Converter<Map<Object?, Object?>, RouteMatchList> decoder;
}
```

## 继承关系

该类继承自 `Codec<RouteMatchList, Map<Object?, Object?>>`，这意味着：

- **输入类型**：`RouteMatchList` - 需要编码的路由匹配列表对象
- **输出类型**：`Map<Object?, Object?>` - 编码后的 Map 格式，可以与 `StandardMessageCodec` 配合使用

## 核心组件

### 构造函数

```dart 912:914:lib/src/match.dart
RouteMatchListCodec(RouteConfiguration configuration)
  : decoder = _RouteMatchListDecoder(configuration),
    encoder = _RouteMatchListEncoder(configuration);
```

构造函数接收一个 `RouteConfiguration` 参数，用于：

1. 创建 `_RouteMatchListDecoder` 实例（解码器）
2. 创建 `_RouteMatchListEncoder` 实例（编码器）

两个转换器都需要配置信息来正确执行编码和解码操作，因为它们需要了解路由配置结构才能重建 `RouteMatchList` 对象。

### 编码器和解码器

```dart 925:929:lib/src/match.dart
@override
final Converter<RouteMatchList, Map<Object?, Object?>> encoder;

@override
final Converter<Map<Object?, Object?>, RouteMatchList> decoder;
```

这两个属性实现了 `Codec` 接口的要求：

- **encoder**：将 `RouteMatchList` 编码为 `Map<Object?, Object?>`
- **decoder**：将 `Map<Object?, Object?>` 解码为 `RouteMatchList`

### 序列化键常量

类中定义了一系列静态常量，用于标识序列化 Map 中的各个字段：

#### `_locationKey = 'location'`

存储路由位置的 URI 字符串。

#### `_extraKey = 'state'`

存储路由的额外状态信息（extra data）。注意这里的键名是 `'state'`，但变量名是 `_extraKey`，这是为了与序列化格式保持一致。

#### `_imperativeMatchesKey = 'imperativeMatches'`

存储命令式路由匹配列表。命令式路由匹配是通过 `go()`、`push()` 等方法动态创建的路由匹配。

#### `_pageKey = 'pageKey'`

存储页面的唯一标识键。

#### `_codecKey = 'codec'`

标识用于编码 `extra` 数据的编码器类型。可能的值：

- `_jsonCodecName = 'json'`：使用 JSON 编码
- `_customCodecName = 'custom'`：使用自定义编码器

#### `_encodedKey = 'encoded'`

存储实际编码后的 `extra` 数据。

## 使用场景

### 1. 状态恢复（State Restoration）

在 Flutter 应用中，当应用被系统终止后重新启动时，可以使用 `RouteMatchListCodec` 来恢复之前的导航状态。编码后的数据可以保存到持久化存储中，应用重启后解码恢复路由状态。

### 2. 浏览器历史记录（Browser History）

在 Web 平台上，go_router 需要与浏览器的历史记录 API 集成。编码后的 `RouteMatchList` 可以存储在浏览器的历史状态中，当用户使用前进/后退按钮时，可以从历史状态中解码并恢复路由。

### 3. 平台通道（Platform Channels）

由于 `StandardMessageCodec` 只能处理基本数据类型（primitive types），复杂的 `RouteMatchList` 对象需要先转换为 `Map<Object?, Object?>` 格式，才能通过平台通道传递。

## 注解说明

### `@meta.internal`

```dart 909:909:lib/src/match.dart
@meta.internal
```

这个注解标记该类为内部 API，意味着：

- 该类不应该被外部代码直接使用
- API 可能会在未来版本中更改
- 主要用于 go_router 包内部的实现细节

## 实现细节

虽然该类本身的代码很简单，但它依赖于两个私有类来实现实际的编码和解码逻辑：

1. **`_RouteMatchListEncoder`**：负责将 `RouteMatchList` 编码为 Map
2. **`_RouteMatchListDecoder`**：负责将 Map 解码为 `RouteMatchList`

这两个类的实现处理了：

- URI 的序列化
- `extra` 数据的编码（支持 JSON 和自定义编码器）
- 命令式路由匹配的递归编码/解码
- 错误处理（当 `extra` 数据无法序列化时）

## 注意事项

1. **编码器选择**：如果 `RouteConfiguration` 中配置了 `extraCodec`，则使用自定义编码器；否则使用 JSON 编码。JSON 编码可能无法处理复杂的对象类型，此时会发出警告并返回 `null`。

2. **内部 API**：由于使用了 `@meta.internal` 注解，外部代码不应该直接实例化或使用这个类。它主要用于 go_router 的内部实现。

3. **递归解码**：解码器需要递归处理命令式路由匹配，因为它们可能包含嵌套的路由匹配信息。

4. **配置依赖**：编码器和解码器都需要 `RouteConfiguration` 来正确工作，因为它们需要了解路由结构才能重建路由匹配。

## 相关类

- `RouteMatchList`：需要编码/解码的主要数据类型
- `RouteConfiguration`：路由配置，提供路由结构和编码器信息
- `ImperativeRouteMatch`：命令式路由匹配，需要在序列化中特殊处理
- `StandardMessageCodec`：Flutter 的标准消息编解码器，用于平台通道通信
