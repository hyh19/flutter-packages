# AdaptiveLayout 类详解

## 引言

`AdaptiveLayout` 是 `flutter_adaptive_scaffold` 包中响应式布局系统的核心组件，负责将应用窗口划分为预定义的槽位（slots），并根据不同屏幕条件自动调整布局。它使用 `SlotLayout` 来管理每个槽位在不同断点下的内容，通过 `CustomMultiChildLayout` 和 `MultiChildLayoutDelegate` 实现精确的布局控制，支持折叠屏适配、文本方向（LTR/RTL）切换以及流畅的布局过渡动画。

`AdaptiveLayout` 提供了完全自定义的布局能力，是构建复杂响应式应用的基础。它通过槽位系统将复杂的布局问题分解为多个独立的子问题，每个槽位都可以独立配置，最终组合成完整的响应式界面。

## 类概述

### 设计目的

`AdaptiveLayout` 类的主要目的是：

1. **槽位管理**：将应用窗口划分为六个预定义的槽位，每个槽位可以独立配置
2. **响应式布局**：根据屏幕尺寸、方向、平台等条件自动调整槽位布局
3. **折叠屏适配**：自动检测并适配折叠屏设备的铰链（hinge）区域
4. **布局动画**：提供流畅的布局切换动画，提升用户体验
5. **文本方向支持**：支持从左到右（LTR）和从右到左（RTL）的文本方向

### 在响应式布局中的作用

`AdaptiveLayout` 在响应式布局系统中起到顶层协调者的作用：

- **槽位编排**：管理六个槽位的布局和位置关系
- **空间分配**：根据槽位尺寸和屏幕空间，智能分配剩余空间
- **动画协调**：协调多个槽位的尺寸变化动画
- **折叠屏处理**：检测折叠屏铰链，调整布局以适应铰链位置

### 与 SlotLayout 的关系

`AdaptiveLayout` 与 `SlotLayout` 是组合关系：

- **槽位配置**：每个槽位都是一个 `SlotLayout`，负责该槽位在不同断点下的内容选择
- **布局分离**：`AdaptiveLayout` 负责位置和尺寸，`SlotLayout` 负责内容选择
- **协同工作**：`AdaptiveLayout` 从 `SlotLayout` 获取当前选择的配置，然后进行布局

**示例**：

```dart
AdaptiveLayout(
  body: SlotLayout(
    config: {
      Breakpoints.small: SlotLayoutConfig(...),
      Breakpoints.medium: SlotLayoutConfig(...),
    },
  ),
  primaryNavigation: SlotLayout(
    config: {
      Breakpoints.medium: SlotLayoutConfig(...),
    },
  ),
)
```

### 与 Breakpoint 的关系

`AdaptiveLayout` 通过 `SlotLayout` 间接使用 `Breakpoint`：

- **断点检测**：每个 `SlotLayout` 根据 `Breakpoint` 选择配置
- **响应式切换**：当屏幕尺寸变化导致断点切换时，`AdaptiveLayout` 会重新布局
- **布局适配**：不同断点下，槽位的布局策略可能不同

### StatefulWidget 设计

`AdaptiveLayout` 继承自 `StatefulWidget`，这是因为：

- **动画管理**：需要 `AnimationController` 来管理布局动画
- **状态跟踪**：需要跟踪槽位尺寸、动画状态等信息
- **响应式更新**：当屏幕尺寸或断点变化时，需要重新计算布局

## 核心组件详解

### AdaptiveLayout 类

#### AdaptiveLayout 构造函数

```dart
const AdaptiveLayout({
  super.key,
  this.topNavigation,
  this.primaryNavigation,
  this.secondaryNavigation,
  this.bottomNavigation,
  this.body,
  this.secondaryBody,
  this.bodyRatio,
  this.transitionDuration = const Duration(seconds: 1),
  this.internalAnimations = true,
  this.bodyOrientation = Axis.horizontal,
});
```

**参数说明**：

- `key`（可选）：Widget 的键，用于 Widget 树中的识别
- `topNavigation`（可选）：顶部导航槽位
- `primaryNavigation`（可选）：主导航槽位（左侧/右侧）
- `secondaryNavigation`（可选）：次导航槽位
- `bottomNavigation`（可选）：底部导航槽位
- `body`（可选）：主体内容槽位
- `secondaryBody`（可选）：次要主体内容槽位
- `bodyRatio`（可选）：body 和 secondaryBody 的比例
- `transitionDuration`（可选）：过渡动画时长，默认 1 秒
- `internalAnimations`（可选）：是否启用内部动画，默认 `true`
- `bodyOrientation`（可选）：body 的排列方向，默认水平

**特点**：

- 使用 `const` 构造函数，支持编译时常量
- 所有槽位参数都是可选的，可以根据需要配置
- 提供了合理的默认值，简化使用

#### 槽位属性详解

##### topNavigation - 顶部导航

```dart
final SlotLayout? topNavigation;
```

**位置**：应用窗口的顶部，全宽显示

**特点**：

- 必须具有明确的尺寸（不能是 `Container` 等弹性 Widget）
- 从屏幕顶部开始，占据整个宽度``
- 通常用于应用栏（AppBar）、工具栏等

**布局规则**：

- 位置：`Offset(0, 0)`
- 宽度：`size.width`（全宽）
- 高度：由内容决定
- 影响：增加 `topMargin`，影响下方所有槽位

**示例**：

```dart
AdaptiveLayout(
  topNavigation: SlotLayout(
    config: {
      Breakpoints.standard: SlotLayoutConfig.from(
        key: const Key('top_nav'),
        builder: (_) => AppBar(
          title: const Text('我的应用'),
        ),
      ),
    },
  ),
)
```

##### bottomNavigation - 底部导航

```dart
final SlotLayout? bottomNavigation;
```

**位置**：应用窗口的底部，全宽显示

**特点**：

- 必须具有明确的尺寸
- 从屏幕底部开始，占据整个宽度
- 通常用于底部导航栏（BottomNavigationBar）

**布局规则**：

- 位置：`Offset(0, size.height - currentSize.height)`
- 宽度：`size.width`（全宽）
- 高度：由内容决定
- 影响：增加 `bottomMargin`，影响上方所有槽位

**示例**：

```dart
AdaptiveLayout(
  bottomNavigation: SlotLayout(
    config: {
      Breakpoints.small: SlotLayoutConfig.from(
        key: const Key('bottom_nav'),
        builder: (_) => BottomNavigationBar(
          items: destinations,
        ),
      ),
    },
  ),
)
```

##### primaryNavigation - 主导航

```dart
final SlotLayout? primaryNavigation;
```

**位置**：应用窗口的"开始"侧

**方向说明**：

- **LTR（从左到右）**：左侧
- **RTL（从右到左）**：右侧

**特点**：

- 必须具有明确的尺寸
- 从 `topNavigation` 下方延伸到 `bottomNavigation` 上方
- 通常用于导航栏（NavigationRail）、侧边栏等

**布局规则**：

- **LTR 模式**：
  - 位置：`Offset(leftMargin, topMargin)`
  - 宽度：由内容决定
  - 高度：`size.height - topMargin - bottomMargin`
  - 影响：增加 `leftMargin`
- **RTL 模式**：
  - 位置：`Offset(size.width - currentSize.width, topMargin)`
  - 宽度：由内容决定
  - 高度：`size.height - topMargin - bottomMargin`
  - 影响：增加 `rightMargin`

**示例**：

```dart
AdaptiveLayout(
  primaryNavigation: SlotLayout(
    config: {
      Breakpoints.medium: SlotLayoutConfig.from(
        key: const Key('primary_nav'),
        builder: (_) => NavigationRail(
          destinations: destinations,
          selectedIndex: selectedIndex,
        ),
      ),
    },
  ),
)
```

##### secondaryNavigation - 次导航

```dart
final SlotLayout? secondaryNavigation;
```

**位置**：应用窗口的"结束"侧

**方向说明**：

- **LTR（从左到右）**：右侧
- **RTL（从右到左）**：左侧

**特点**：

- 必须具有明确的尺寸
- 从 `topNavigation` 下方延伸到 `bottomNavigation` 上方
- 较少使用，可用于辅助导航、工具栏等

**布局规则**：

- **LTR 模式**：
  - 位置：`Offset(size.width - currentSize.width, topMargin)`
  - 宽度：由内容决定
  - 高度：`size.height - topMargin - bottomMargin`
  - 影响：增加 `rightMargin`
- **RTL 模式**：
  - 位置：`Offset(0, topMargin)`
  - 宽度：由内容决定
  - 高度：`size.height - topMargin - bottomMargin`
  - 影响：增加 `leftMargin`

##### body - 主体内容

```dart
final SlotLayout? body;
```

**位置**：填充剩余空间的主要区域

**特点**：

- 可以具有弹性尺寸（如 `Container`）
- 从"开始"侧填充剩余空间
- 通常用于应用的主要内容区域

**布局规则**：

- 位置：`Offset(leftMargin, topMargin)`
- 宽度：`remainingWidth`（剩余宽度）
- 高度：`remainingHeight`（剩余高度）
- 如果存在 `secondaryBody`，会根据 `bodyRatio` 和 `bodyOrientation` 进行分割

**示例**：

```dart
AdaptiveLayout(
  body: SlotLayout(
    config: {
      Breakpoints.small: SlotLayoutConfig.from(
        key: const Key('body_small'),
        builder: (_) => ListView(
          children: items,
        ),
      ),
      Breakpoints.medium: SlotLayoutConfig.from(
        key: const Key('body_medium'),
        builder: (_) => GridView.count(
          crossAxisCount: 2,
          children: items,
        ),
      ),
    },
  ),
)
```

##### secondaryBody - 次要主体内容

```dart
final SlotLayout? secondaryBody;
```

**位置**：填充剩余空间的次要区域

**特点**：

- 可以具有弹性尺寸
- 从"结束"侧填充剩余空间
- 通常用于详情视图、辅助面板等
- 默认具有滑动进入动画

**布局规则**：

- 与 `body` 共享剩余空间
- 分割方式由 `bodyRatio` 和 `bodyOrientation` 决定
- 支持折叠屏铰链适配

**使用场景**：

- 主从视图（Master-Detail）：主列表和详情视图
- 双面板布局：编辑器和预览
- 折叠屏应用：利用铰链两侧的屏幕

**示例**：

```dart
AdaptiveLayout(
  body: SlotLayout(...),  // 主列表
  secondaryBody: SlotLayout(
    config: {
      Breakpoints.mediumLarge: SlotLayoutConfig.from(
        key: const Key('detail'),
        builder: (_) => DetailView(item: selectedItem),
      ),
    },
  ),
)
```

#### 其他重要属性

##### bodyRatio - 主体比例

```dart
final double? bodyRatio;
```

**作用**：定义 `body` 和 `secondaryBody` 之间的空间分配比例

**取值范围**：`0.0` 到 `1.0`

**说明**：

- `0.3` 表示 `body` 占据 30% 的空间，`secondaryBody` 占据 70%
- `null` 表示使用默认比例：
  - 无铰链：各占 50%（居中分割）
  - 有铰链：围绕铰链分割

**示例**：

```dart
AdaptiveLayout(
  bodyRatio: 0.3,  // body 占 30%，secondaryBody 占 70%
  body: SlotLayout(...),
  secondaryBody: SlotLayout(...),
)
```

##### transitionDuration - 过渡时长

```dart
final Duration transitionDuration = const Duration(seconds: 1);
```

**作用**：定义布局切换动画的持续时间

**默认值**：`Duration(seconds: 1)`

**说明**：

- 影响槽位尺寸变化的动画时长
- 影响 `secondaryBody` 的进入动画时长
- 可以通过设置较短的时长来加快响应速度

**示例**：

```dart
AdaptiveLayout(
  transitionDuration: const Duration(milliseconds: 300),  // 300 毫秒
  // ...
)
```

##### internalAnimations - 内部动画

```dart
final bool internalAnimations = true;
```

**作用**：控制是否启用内部布局动画

**默认值**：`true`

**说明**：

- `true`：启用 `secondaryBody` 的滑动进入动画
- `false`：禁用内部动画，布局立即切换

**使用场景**：

- 性能优化：在低端设备上禁用动画
- 测试：快速验证布局逻辑
- 特殊需求：需要立即切换的场景

**示例**：

```dart
AdaptiveLayout(
  internalAnimations: false,  // 禁用内部动画
  // ...
)
```

##### bodyOrientation - 主体方向

```dart
final Axis bodyOrientation = Axis.horizontal;
```

**作用**：定义 `body` 和 `secondaryBody` 的排列方向

**可选值**：

- `Axis.horizontal`：水平排列（左右并排），默认值
- `Axis.vertical`：垂直排列（上下堆叠）

**说明**：

- 水平排列：适用于宽屏设备，如平板、桌面
- 垂直排列：适用于竖屏设备，或需要上下布局的场景

**示例**：

```dart
AdaptiveLayout(
  bodyOrientation: Axis.vertical,  // 垂直排列
  body: SlotLayout(...),
  secondaryBody: SlotLayout(...),
)
```

### _AdaptiveLayoutState 状态管理

`_AdaptiveLayoutState` 是 `AdaptiveLayout` 的状态类，负责管理动画、槽位尺寸和布局状态。

#### 动画控制器

```dart
late AnimationController _controller;
late final CurvedAnimation _sizeAnimation = CurvedAnimation(
  parent: _controller,
  curve: Curves.easeInOutCubic,
);
```

**作用**：

- `_controller`：控制布局动画的播放
- `_sizeAnimation`：提供平滑的尺寸变化动画，使用 `Curves.easeInOutCubic` 曲线

**初始化逻辑**：

```dart
if (widget.internalAnimations) {
  _controller = AnimationController(
    duration: widget.transitionDuration,
    vsync: this,
  )..forward();
} else {
  _controller = AnimationController(
    duration: Duration.zero,
    vsync: this,
  );
}
```

- 如果启用内部动画，使用配置的 `transitionDuration`
- 如果禁用内部动画，使用零时长（立即完成）

#### 槽位尺寸管理

```dart
Map<String, Size?> slotSizes = <String, Size?>{};
```

**作用**：存储每个槽位的尺寸信息

**用途**：

- 记录槽位的当前尺寸
- 在动画过程中，从旧尺寸过渡到新尺寸
- 用于计算边距和剩余空间

**更新机制**：

- 在 `_AdaptiveLayoutDelegate.updateSize()` 中更新
- 当槽位尺寸变化时，触发动画
- 动画完成后，更新存储的尺寸

#### 动画状态跟踪

```dart
Set<String> isAnimating = <String>{};
Map<String, ValueNotifier<Key?>> notifiers = <String, ValueNotifier<Key?>>{};
```

**作用**：

- `isAnimating`：跟踪哪些槽位正在动画中
- `notifiers`：为每个槽位提供 `ValueNotifier`，监听配置变化

**工作流程**：

1. 为每个槽位创建 `ValueNotifier<Key?>`
2. 当槽位的配置 Key 变化时，触发监听器
3. 将槽位添加到 `isAnimating` 集合
4. 重置并启动动画控制器
5. 动画完成后，清空 `isAnimating` 集合

**初始化代码**：

```dart
for (final _SlotIds item in _SlotIds.values) {
  notifiers[item.name] = ValueNotifier<Key?>(null)
    ..addListener(() {
      isAnimating.add(item.name);
      _controller.reset();
      _controller.forward();
    });
}
```

#### 选择的 Widget 配置

```dart
late Map<String, SlotLayoutConfig?> chosenWidgets = <String, SlotLayoutConfig?>{};
```

**作用**：存储每个槽位当前选择的 `SlotLayoutConfig`

**更新时机**：在 `build` 方法中，通过 `SlotLayout.pickWidget()` 选择

**用途**：

- 传递给 `_AdaptiveLayoutDelegate`，用于布局计算
- 判断 `secondaryBody` 是否存在内容
- 更新 `notifiers` 的值，触发动画

### _AdaptiveLayoutDelegate 布局委托

`_AdaptiveLayoutDelegate` 继承自 `MultiChildLayoutDelegate`，负责实际计算每个槽位的位置和尺寸。

#### _AdaptiveLayoutDelegate 构造函数

```dart
_AdaptiveLayoutDelegate({
  required this.slots,
  required this.chosenWidgets,
  required this.slotSizes,
  required this.controller,
  required this.bodyRatio,
  required this.isAnimating,
  required this.internalAnimations,
  required this.bodyOrientation,
  required this.textDirection,
  required this.sizeAnimation,
  this.hinge,
}) : super(relayout: controller);
```

**参数说明**：

- `slots`：所有槽位的 `SlotLayout` 映射
- `chosenWidgets`：每个槽位当前选择的配置
- `slotSizes`：每个槽位的尺寸记录
- `controller`：动画控制器，用于触发重布局
- `bodyRatio`：body 和 secondaryBody 的比例
- `isAnimating`：正在动画的槽位集合
- `internalAnimations`：是否启用内部动画
- `bodyOrientation`：body 的排列方向
- `textDirection`：文本方向（`true` 表示 LTR）
- `sizeAnimation`：尺寸动画
- `hinge`：折叠屏铰链区域（可选）

**关键点**：

- `super(relayout: controller)`：当动画控制器变化时，触发重布局
- 所有参数都是必需的，确保布局计算的准确性

#### performLayout 方法

`performLayout` 是布局委托的核心方法，负责计算每个槽位的位置和尺寸。

##### 布局流程概述

布局按照以下顺序进行：

1. **初始化边距**：`leftMargin`、`topMargin`、`rightMargin`、`bottomMargin`
2. **布局顶部导航**：`topNavigation`（如果存在）
3. **布局底部导航**：`bottomNavigation`（如果存在）
4. **布局主导航**：`primaryNavigation`（如果存在）
5. **布局次导航**：`secondaryNavigation`（如果存在）
6. **计算剩余空间**：`remainingWidth`、`remainingHeight`
7. **布局主体内容**：`body` 和 `secondaryBody`（如果存在）

##### performLayout 中的边距累积

边距是逐步累积的：

```dart
double leftMargin = 0;
double topMargin = 0;
double rightMargin = 0;
double bottomMargin = 0;
```

**累积规则**：

- `topNavigation` 增加 `topMargin`
- `bottomNavigation` 增加 `bottomMargin`
- `primaryNavigation` 根据文本方向增加 `leftMargin` 或 `rightMargin`
- `secondaryNavigation` 根据文本方向增加 `rightMargin` 或 `leftMargin`

**示例**：

```dart
// 1. topNavigation 高度 56，topMargin = 56
// 2. primaryNavigation 宽度 72（LTR），leftMargin = 72
// 3. 剩余宽度 = size.width - 72
// 4. 剩余高度 = size.height - 56
```

##### 顶部导航布局

```dart
if (hasChild(_SlotIds.topNavigation.name)) {
  final Size childSize = layoutChild(
    _SlotIds.topNavigation.name,
    BoxConstraints.loose(size),
  );
  updateSize(_SlotIds.topNavigation.name, childSize);
  final Size currentSize = Tween<Size>(
    begin: slotSizes[_SlotIds.topNavigation.name] ?? Size.zero,
    end: childSize,
  ).animate(controller).value;
  positionChild(_SlotIds.topNavigation.name, Offset.zero);
  topMargin += currentSize.height;
}
```

**步骤说明**：

1. 使用 `BoxConstraints.loose(size)` 布局子 Widget，允许子 Widget 决定自己的尺寸
2. 调用 `updateSize()` 更新尺寸记录，如果尺寸变化则触发动画
3. 使用 `Tween<Size>` 创建尺寸动画，从旧尺寸过渡到新尺寸
4. 将 Widget 定位到 `Offset(0, 0)`（左上角）
5. 将当前高度添加到 `topMargin`

**动画处理**：

- 使用 `Tween<Size>` 实现平滑的尺寸过渡
- 动画值用于计算边距，确保其他槽位平滑移动

##### 底部导航布局

```dart
if (hasChild(_SlotIds.bottomNavigation.name)) {
  final Size childSize = layoutChild(
    _SlotIds.bottomNavigation.name,
    BoxConstraints.loose(size),
  );
  updateSize(_SlotIds.bottomNavigation.name, childSize);
  final Size currentSize = Tween<Size>(
    begin: slotSizes[_SlotIds.bottomNavigation.name] ?? Size.zero,
    end: childSize,
  ).animate(controller).value;
  positionChild(
    _SlotIds.bottomNavigation.name,
    Offset(0, size.height - currentSize.height),
  );
  bottomMargin += currentSize.height;
}
```

**关键点**：

- 位置计算：`Offset(0, size.height - currentSize.height)`，从底部向上定位
- 增加 `bottomMargin`，影响上方槽位的可用高度

##### 主导航布局

```dart
if (hasChild(_SlotIds.primaryNavigation.name)) {
  final Size childSize = layoutChild(...);
  updateSize(_SlotIds.primaryNavigation.name, childSize);
  final Size currentSize = Tween<Size>(...).animate(controller).value;
  if (textDirection) {
    // LTR 模式
    positionChild(
      _SlotIds.primaryNavigation.name,
      Offset(leftMargin, topMargin),
    );
    leftMargin += currentSize.width;
  } else {
    // RTL 模式
    positionChild(
      _SlotIds.primaryNavigation.name,
      Offset(size.width - currentSize.width, topMargin),
    );
    rightMargin += currentSize.width;
  }
}
```

**文本方向处理**：

- **LTR**：放置在左侧，增加 `leftMargin`
- **RTL**：放置在右侧，增加 `rightMargin`

##### 次导航布局

```dart
if (hasChild(_SlotIds.secondaryNavigation.name)) {
  // ... 布局逻辑
  if (textDirection) {
    // LTR 模式：右侧
    positionChild(
      _SlotIds.secondaryNavigation.name,
      Offset(size.width - currentSize.width, topMargin),
    );
    rightMargin += currentSize.width;
  } else {
    // RTL 模式：左侧
    positionChild(_SlotIds.secondaryNavigation.name, Offset(0, topMargin));
    leftMargin += currentSize.width;
  }
}
```

##### Body 和 SecondaryBody 布局

这是最复杂的部分，需要处理多种情况：

1. **只有 body**：填充所有剩余空间
2. **body + secondaryBody（secondaryBody 为空）**：body 填充所有空间
3. **body + secondaryBody（都有内容）**：根据 `bodyRatio` 和 `bodyOrientation` 分割

###### 情况 1：只有 body

```dart
else if (hasChild(_SlotIds.body.name)) {
  layoutChild(
    _SlotIds.body.name,
    BoxConstraints.tight(
      Size(remainingWidth, remainingHeight),
    ),
  );
  positionChild(_SlotIds.body.name, Offset(leftMargin, topMargin));
}
```

**说明**：使用 `BoxConstraints.tight()` 强制 body 使用所有剩余空间。

###### 情况 2：secondaryBody 为空

```dart
if (chosenWidgets[_SlotIds.secondaryBody.name] == null ||
    chosenWidgets[_SlotIds.secondaryBody.name]!.builder == null) {
  // body 填充所有剩余空间
  if (!textDirection) {
    // RTL 模式
    currentBodySize = layoutChild(
      _SlotIds.body.name,
      BoxConstraints.tight(
        Size(remainingWidth, remainingHeight),
      ),
    );
  } else if (bodyOrientation == Axis.horizontal) {
    // 水平布局，使用动画从部分宽度过渡到全宽
    double beginWidth = bodyRatio == null
        ? halfWidth - leftMargin
        : remainingWidth * bodyRatio!;
    currentBodySize = layoutChild(
      _SlotIds.body.name,
      BoxConstraints.tight(
        Size(animatedSize(beginWidth, remainingWidth), remainingHeight),
      ),
    );
  } else {
    // 垂直布局，使用动画从部分高度过渡到全高
    // ...
  }
  layoutChild(_SlotIds.secondaryBody.name, BoxConstraints.loose(size));
}
```

**说明**：

- 当 `secondaryBody` 为空时，`body` 应该填充所有空间
- 使用 `animatedSize()` 实现从分割状态到全屏状态的平滑过渡
- `secondaryBody` 使用 `BoxConstraints.loose()` 布局，但不显示

###### 情况 3：body + secondaryBody（都有内容）

这是最复杂的情况，需要处理：

- 水平布局 vs 垂直布局
- LTR vs RTL
- 有铰链 vs 无铰链
- 自定义比例 vs 默认比例

**水平布局（LTR）**：

```dart
if (bodyOrientation == Axis.horizontal) {
  if (textDirection) {
    // LTR 模式
    double finalBodySize;
    double finalSBodySize;
    if (hinge != null) {
      // 有铰链：围绕铰链分割
      finalBodySize = hinge!.left - leftMargin;
      finalSBodySize = size.width - (hinge!.left + hingeWidth) - rightMargin;
    } else if (bodyRatio != null) {
      // 自定义比例
      finalBodySize = remainingWidth * bodyRatio!;
      finalSBodySize = remainingWidth * (1 - bodyRatio!);
    } else {
      // 默认：各占 50%
      finalBodySize = halfWidth - leftMargin;
      finalSBodySize = halfWidth - rightMargin;
    }
    
    // 布局 body：从全宽动画到目标宽度
    currentBodySize = layoutChild(
      _SlotIds.body.name,
      BoxConstraints.tight(
        Size(animatedSize(remainingWidth, finalBodySize), remainingHeight),
      ),
    );
    // 布局 secondaryBody：固定宽度
    layoutChild(
      _SlotIds.secondaryBody.name,
      BoxConstraints.tight(
        Size(finalSBodySize, remainingHeight),
      ),
    );
  }
}
```

**水平布局（RTL）**：

```dart
else {
  // RTL 模式
  double finalBodySize;
  double finalSBodySize;
  if (hinge != null) {
    finalBodySize = size.width - (hinge!.left + hingeWidth) - rightMargin;
    finalSBodySize = hinge!.left - leftMargin;
  } else if (bodyRatio != null) {
    finalBodySize = remainingWidth * bodyRatio!;
    finalSBodySize = remainingWidth * (1 - bodyRatio!);
  } else {
    finalBodySize = halfWidth - rightMargin;
    finalSBodySize = halfWidth - leftMargin;
  }
  
  // secondaryBody 在左侧，从 0 宽度动画到目标宽度
  currentSBodySize = layoutChild(
    _SlotIds.secondaryBody.name,
    BoxConstraints.tight(
      Size(animatedSize(0, finalSBodySize), remainingHeight),
    ),
  );
  // body 在右侧，固定宽度
  layoutChild(
    _SlotIds.body.name,
    BoxConstraints.tight(
      Size(finalBodySize, remainingHeight),
    ),
  );
}
```

**垂直布局**：

```dart
else {
  // 垂直布局
  currentBodySize = layoutChild(
    _SlotIds.body.name,
    BoxConstraints.tight(
      Size(
        remainingWidth,
        animatedSize(
          remainingHeight,
          bodyRatio == null
              ? halfHeight - topMargin
              : remainingHeight * bodyRatio!,
        ),
      ),
    ),
  );
  layoutChild(
    _SlotIds.secondaryBody.name,
    BoxConstraints.tight(
      Size(
        remainingWidth,
        bodyRatio == null
            ? halfHeight - bottomMargin
            : remainingHeight * (1 - bodyRatio!),
      ),
    ),
  );
}
```

**定位逻辑**：

```dart
// 处理定位
if (bodyOrientation == Axis.horizontal &&
    !textDirection &&
    chosenWidgets[_SlotIds.secondaryBody.name] != null) {
  // RTL 水平布局：secondaryBody 在左，body 在右
  if (hinge != null) {
    positionChild(
      _SlotIds.body.name,
      Offset(currentSBodySize.width + leftMargin + hingeWidth, topMargin),
    );
    positionChild(
      _SlotIds.secondaryBody.name,
      Offset(leftMargin, topMargin),
    );
  } else {
    positionChild(
      _SlotIds.body.name,
      Offset(currentSBodySize.width + leftMargin, topMargin),
    );
    positionChild(
      _SlotIds.secondaryBody.name,
      Offset(leftMargin, topMargin),
    );
  }
} else {
  // LTR 水平布局或垂直布局
  positionChild(_SlotIds.body.name, Offset(leftMargin, topMargin));
  if (bodyOrientation == Axis.horizontal) {
    if (hinge != null) {
      positionChild(
        _SlotIds.secondaryBody.name,
        Offset(
          currentBodySize.width + leftMargin + hingeWidth,
          topMargin,
        ),
      );
    } else {
      positionChild(
        _SlotIds.secondaryBody.name,
        Offset(currentBodySize.width + leftMargin, topMargin),
      );
    }
  } else {
    positionChild(
      _SlotIds.secondaryBody.name,
      Offset(leftMargin, topMargin + currentBodySize.height),
    );
  }
}
```

##### 折叠屏铰链处理

```dart
Rect? hinge;
for (final DisplayFeature e in MediaQuery.displayFeaturesOf(context)) {
  if (e.type == DisplayFeatureType.hinge ||
      e.type == DisplayFeatureType.fold) {
    if (e.bounds.left != 0) {
      hinge = e.bounds;
    }
  }
}
```

**检测逻辑**：

- 遍历所有 `DisplayFeature`
- 查找类型为 `hinge` 或 `fold` 的特征
- 只处理左侧不为 0 的铰链（排除屏幕边缘的铰链）

**布局影响**：

- 当存在铰链时，`body` 和 `secondaryBody` 围绕铰链分割
- 铰链宽度（`hingeWidth`）在定位时需要考虑
- 确保内容不会被铰链遮挡

**示例**：

```dart
if (hinge != null) {
  // body 在铰链左侧
  finalBodySize = hinge!.left - leftMargin;
  // secondaryBody 在铰链右侧
  finalSBodySize = size.width - (hinge!.left + hingeWidth) - rightMargin;
}
```

#### updateSize 方法

```dart
void updateSize(String id, Size childSize) {
  if (slotSizes[id] == null || slotSizes[id] != childSize) {
    void listener(AnimationStatus status) {
      if ((status == AnimationStatus.completed ||
              status == AnimationStatus.dismissed) &&
          (slotSizes[id] == null || slotSizes[id] != childSize)) {
        slotSizes[id] = childSize;
      }
      controller.removeStatusListener(listener);
    }
    controller.addStatusListener(listener);
  }
}
```

**作用**：更新槽位尺寸，并在尺寸变化时触发动画

**工作流程**：

1. 检查尺寸是否变化
2. 如果变化，添加动画状态监听器
3. 动画完成后，更新 `slotSizes` 并移除监听器

**关键点**：

- 只在尺寸真正变化时更新
- 使用一次性监听器，避免内存泄漏
- 在动画完成后才更新尺寸，确保动画平滑

#### shouldRelayout 方法

```dart
@override
bool shouldRelayout(_AdaptiveLayoutDelegate oldDelegate) {
  return oldDelegate.slots != slots;
}
```

**作用**：判断是否需要重新布局

**逻辑**：当槽位配置变化时，需要重新布局

**优化**：只比较 `slots`，因为其他参数变化会通过 `relayout` 参数触发

## 布局算法详解

### 槽位布局顺序

布局按照以下固定顺序进行：

1. **topNavigation**：顶部，全宽
2. **bottomNavigation**：底部，全宽
3. **primaryNavigation**：开始侧，固定宽度
4. **secondaryNavigation**：结束侧，固定宽度
5. **body** 和 **secondaryBody**：填充剩余空间

**为什么是这个顺序？**

- 顶部和底部导航需要先布局，确定垂直边距
- 侧边导航需要先布局，确定水平边距
- 主体内容最后布局，使用剩余空间

### 边距累积机制

边距是逐步累积的，每个槽位都会影响后续槽位的可用空间：

```dart
// 初始状态
topMargin = 0
bottomMargin = 0
leftMargin = 0
rightMargin = 0

// 布局 topNavigation（高度 56）
topMargin = 56

// 布局 primaryNavigation（宽度 72，LTR）
leftMargin = 72

// 布局 secondaryNavigation（宽度 200，LTR）
rightMargin = 200

// 计算剩余空间
remainingWidth = size.width - 72 - 200
remainingHeight = size.height - 56 - bottomMargin
```

### 折叠屏适配逻辑

折叠屏适配的核心是检测铰链位置，并围绕铰链分割 `body` 和 `secondaryBody`：

```dart
// 检测铰链
Rect? hinge = ...;

// 计算铰链宽度
double hingeWidth = hinge != null ? hinge!.right - hinge!.left : 0;

// 分割空间（LTR 水平布局）
if (hinge != null) {
  // body 占据铰链左侧
  finalBodySize = hinge!.left - leftMargin;
  // secondaryBody 占据铰链右侧
  finalSBodySize = size.width - (hinge!.left + hingeWidth) - rightMargin;
}
```

**优势**：

- 自动适配，无需手动计算
- 利用折叠屏的双屏特性
- 提供更好的用户体验

### 文本方向适配

`AdaptiveLayout` 完全支持 LTR 和 RTL 文本方向：

**LTR（从左到右）**：

- `primaryNavigation` 在左侧
- `secondaryNavigation` 在右侧
- `body` 在左侧，`secondaryBody` 在右侧

**RTL（从右到左）**：

- `primaryNavigation` 在右侧
- `secondaryNavigation` 在左侧
- `body` 在右侧，`secondaryBody` 在左侧

**实现方式**：

```dart
final bool textDirection = Directionality.of(context) == TextDirection.ltr;
```

通过 `Directionality.of(context)` 获取文本方向，然后在布局逻辑中分支处理。

### Body 和 SecondaryBody 分割逻辑

分割逻辑需要考虑多个因素：

1. **方向**：水平（`Axis.horizontal`）或垂直（`Axis.vertical`）
2. **比例**：自定义比例（`bodyRatio`）或默认比例（`null`）
3. **铰链**：有铰链或无铰链
4. **文本方向**：LTR 或 RTL

**水平布局默认比例**：

```dart
if (bodyRatio == null) {
  // 无铰链：各占 50%
  finalBodySize = halfWidth - leftMargin;
  finalSBodySize = halfWidth - rightMargin;
} else {
  // 自定义比例
  finalBodySize = remainingWidth * bodyRatio!;
  finalSBodySize = remainingWidth * (1 - bodyRatio!);
}
```

**垂直布局默认比例**：

```dart
if (bodyRatio == null) {
  finalBodySize = halfHeight - topMargin;
  finalSBodySize = halfHeight - bottomMargin;
} else {
  finalBodySize = remainingHeight * bodyRatio!;
  finalSBodySize = remainingHeight * (1 - bodyRatio!);
}
```

## 动画机制

### 尺寸动画（sizeAnimation）

`sizeAnimation` 是一个 `CurvedAnimation`，使用 `Curves.easeInOutCubic` 曲线：

```dart
late final CurvedAnimation _sizeAnimation = CurvedAnimation(
  parent: _controller,
  curve: Curves.easeInOutCubic,
);
```

**用途**：

- 在 `performLayout` 中，用于 `body` 和 `secondaryBody` 的尺寸过渡
- 提供平滑的动画效果

**使用示例**：

```dart
double animatedSize(double begin, double end) {
  if (isAnimating.contains(_SlotIds.secondaryBody.name)) {
    return internalAnimations
        ? Tween<double>(begin: begin, end: end).animate(sizeAnimation).value
        : end;
  }
  return end;
}
```

### 槽位切换动画

当槽位的配置 Key 变化时，会触发动画：

```dart
notifiers.forEach((String key, ValueNotifier<Key?> notifier) {
  notifier.value = chosenWidgets[key]?.key;
});
```

**流程**：

1. 更新 `notifier.value` 为新的 Key
2. 触发 `notifier` 的监听器
3. 将槽位添加到 `isAnimating` 集合
4. 重置并启动动画控制器
5. 动画完成后，清空 `isAnimating` 集合

### 内部动画控制

`internalAnimations` 参数控制是否启用内部动画：

**启用时**（`true`）：

- `secondaryBody` 的进入使用滑动动画
- 槽位尺寸变化使用平滑过渡
- 动画时长为 `transitionDuration`

**禁用时**（`false`）：

- 布局立即切换，无动画
- 动画控制器时长为 `Duration.zero`
- 适用于性能优化或特殊需求

## 代码示例

### 示例 1：基本使用

```dart
AdaptiveLayout(
  topNavigation: SlotLayout(
    config: {
      Breakpoints.standard: SlotLayoutConfig.from(
        key: const Key('app_bar'),
        builder: (_) => AppBar(
          title: const Text('我的应用'),
        ),
      ),
    },
  ),
  body: SlotLayout(
    config: {
      Breakpoints.standard: SlotLayoutConfig.from(
        key: const Key('body'),
        builder: (_) => ListView(
          children: [
            ListTile(title: Text('项目 1')),
            ListTile(title: Text('项目 2')),
            ListTile(title: Text('项目 3')),
          ],
        ),
      ),
    },
  ),
)
```

### 示例 2：多槽位配置

```dart
AdaptiveLayout(
  topNavigation: SlotLayout(
    config: {
      Breakpoints.standard: SlotLayoutConfig.from(
        key: const Key('app_bar'),
        builder: (_) => AppBar(title: const Text('应用')),
      ),
    },
  ),
  primaryNavigation: SlotLayout(
    config: {
      Breakpoints.small: SlotLayoutConfig.empty(),  // 小屏幕隐藏
      Breakpoints.medium: SlotLayoutConfig.from(
        key: const Key('nav_rail'),
        builder: (_) => NavigationRail(
          destinations: destinations,
          selectedIndex: selectedIndex,
        ),
      ),
    },
  ),
  body: SlotLayout(
    config: {
      Breakpoints.small: SlotLayoutConfig.from(
        key: const Key('body_small'),
        builder: (_) => ListView(children: items),
      ),
      Breakpoints.medium: SlotLayoutConfig.from(
        key: const Key('body_medium'),
        builder: (_) => GridView.count(
          crossAxisCount: 2,
          children: items,
        ),
      ),
    },
  ),
  bottomNavigation: SlotLayout(
    config: {
      Breakpoints.small: SlotLayoutConfig.from(
        key: const Key('bottom_nav'),
        builder: (_) => BottomNavigationBar(items: destinations),
      ),
      Breakpoints.medium: SlotLayoutConfig.empty(),  // 中等屏幕隐藏
    },
  ),
)
```

### 示例 3：主从视图（Master-Detail）

```dart
AdaptiveLayout(
  primaryNavigation: SlotLayout(
    config: {
      Breakpoints.medium: SlotLayoutConfig.from(
        key: const Key('nav'),
        builder: (_) => NavigationRail(...),
      ),
    },
  ),
  body: SlotLayout(
    config: {
      Breakpoints.standard: SlotLayoutConfig.from(
        key: const Key('master'),
        builder: (_) => MasterListView(
          items: items,
          onItemSelected: (item) {
            setState(() => selectedItem = item);
          },
        ),
      ),
    },
  ),
  secondaryBody: SlotLayout(
    config: {
      Breakpoints.small: SlotLayoutConfig.empty(),  // 小屏幕隐藏详情
      Breakpoints.mediumLarge: SlotLayoutConfig.from(
        key: const Key('detail'),
        builder: (_) => DetailView(item: selectedItem),
      ),
    },
  ),
  bodyRatio: 0.4,  // master 占 40%，detail 占 60%
)
```

### 示例 4：折叠屏适配

```dart
AdaptiveLayout(
  body: SlotLayout(
    config: {
      Breakpoints.standard: SlotLayoutConfig.from(
        key: const Key('left_panel'),
        builder: (_) => LeftPanel(),
      ),
    },
  ),
  secondaryBody: SlotLayout(
    config: {
      Breakpoints.standard: SlotLayoutConfig.from(
        key: const Key('right_panel'),
        builder: (_) => RightPanel(),
      ),
    },
  ),
  // bodyRatio 为 null，自动围绕铰链分割
)
```

**说明**：

- 当检测到铰链时，`body` 和 `secondaryBody` 会自动围绕铰链分割
- 无需手动计算铰链位置
- 提供原生的折叠屏体验

### 示例 5：自定义动画

```dart
AdaptiveLayout(
  transitionDuration: const Duration(milliseconds: 500),
  internalAnimations: true,
  body: SlotLayout(
    config: {
      Breakpoints.medium: SlotLayoutConfig.from(
        key: const Key('body'),
        inAnimation: (child, animation) => FadeTransition(
          opacity: animation,
          child: child,
        ),
        builder: (_) => ContentWidget(),
      ),
    },
  ),
)
```

**说明**：

- `transitionDuration` 控制布局动画时长
- `SlotLayoutConfig` 的 `inAnimation` 控制内容切换动画
- 两者可以组合使用，提供丰富的动画效果

### 示例 6：垂直布局

```dart
AdaptiveLayout(
  bodyOrientation: Axis.vertical,
  body: SlotLayout(
    config: {
      Breakpoints.standard: SlotLayoutConfig.from(
        key: const Key('top_content'),
        builder: (_) => TopContent(),
      ),
    },
  ),
  secondaryBody: SlotLayout(
    config: {
      Breakpoints.standard: SlotLayoutConfig.from(
        key: const Key('bottom_content'),
        builder: (_) => BottomContent(),
      ),
    },
  ),
  bodyRatio: 0.6,  // top 占 60%，bottom 占 40%
)
```

## 最佳实践

### 槽位配置建议

1. **明确尺寸**：固定尺寸的槽位（如导航栏）应该使用明确尺寸的 Widget，避免使用 `Container` 等弹性 Widget

2. **合理使用 empty()**：使用 `SlotLayoutConfig.empty()` 或 `null` 来隐藏某些断点下的槽位

3. **键的唯一性**：确保每个 `SlotLayoutConfig` 的 `key` 是唯一且稳定的

4. **渐进增强**：从 `standard` 断点开始配置，然后逐步添加更具体的断点

### 性能优化建议

1. **禁用动画**：在低端设备上，考虑设置 `internalAnimations: false`

2. **减少重布局**：避免频繁改变槽位配置，使用状态管理来稳定配置

3. **合理使用 const**：尽可能使用 `const` 构造函数，减少 Widget 重建

4. **优化 builder**：`builder` 函数应该尽可能轻量，避免复杂计算

### 常见问题解决

#### 问题 1：槽位内容不显示

**原因**：可能是 `SlotLayoutConfig.builder` 返回了 `null` 或空 Widget

**解决**：

```dart
SlotLayoutConfig.from(
  key: const Key('body'),
  builder: (_) => YourWidget(),  // 确保返回有效的 Widget
)
```

#### 问题 2：布局动画不流畅

**原因**：可能是 `transitionDuration` 设置过长，或 `internalAnimations` 被禁用

**解决**：

```dart
AdaptiveLayout(
  transitionDuration: const Duration(milliseconds: 300),  // 调整时长
  internalAnimations: true,  // 确保启用
  // ...
)
```

#### 问题 3：折叠屏适配不正确

**原因**：可能是铰链检测失败，或布局逻辑有误

**解决**：

- 确保使用最新版本的 Flutter
- 检查 `MediaQuery.displayFeaturesOf(context)` 是否返回正确的数据
- 使用 `bodyRatio: null` 让系统自动适配

#### 问题 4：RTL 布局不正确

**原因**：可能是没有正确处理文本方向

**解决**：

- 确保应用正确设置了 `Directionality`
- 测试时使用 `Directionality` Widget 切换方向：

```dart
Directionality(
  textDirection: TextDirection.rtl,
  child: AdaptiveLayout(...),
)
```

## 总结

`AdaptiveLayout` 是一个强大的响应式布局组件，通过槽位系统将复杂的布局问题分解为多个独立的子问题。它支持：

- **六个预定义槽位**：灵活配置不同区域的内容
- **响应式布局**：根据断点自动调整布局
- **折叠屏适配**：自动检测并适配铰链
- **文本方向支持**：完整的 LTR/RTL 支持
- **流畅动画**：平滑的布局过渡效果

通过合理使用 `AdaptiveLayout`，可以构建出适配各种屏幕尺寸和设备的现代化应用界面。
