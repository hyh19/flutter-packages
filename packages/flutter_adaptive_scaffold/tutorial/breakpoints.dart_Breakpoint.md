# Breakpoint 类详解

## 引言

`Breakpoint` 类是 `flutter_adaptive_scaffold` 包中响应式布局系统的核心组件。它定义了在不同屏幕尺寸、方向和平台下应该使用的布局配置。通过 `Breakpoint`，开发者可以创建能够自适应各种设备的 Flutter 应用，遵循 Material Design 3 的设计规范。

## 类概述

### 设计目的

`Breakpoint` 类的主要目的是：

1. **定义屏幕条件**：通过宽度、高度、平台等条件来区分不同类型的屏幕
2. **提供布局建议**：包含 Material Design 3 推荐的间距、边距、内边距和面板数量
3. **支持响应式布局**：与 `SlotLayout` 配合使用，根据当前屏幕条件动态调整布局

### 在响应式布局中的作用

自适应应用通常根据屏幕类型显示不同的布局：

- **紧凑布局**：适用于较小的屏幕（如手机）
- **宽松布局**：适用于较大的屏幕（如平板、桌面）

`Breakpoint` 通过 `isActive()` 方法判断当前屏幕是否匹配特定条件，从而决定使用哪种布局。

### 与 Material Design 3 规范的关系

`Breakpoint` 类严格遵循 Material Design 3 的设计规范：

- **间距值**：使用 `kMaterialCompactSpacing`（0）和 `kMaterialMediumAndUpSpacing`（24）
- **边距值**：使用 `kMaterialCompactMargin`（16）和 `kMaterialMediumAndUpMargin`（24）
- **内边距值**：使用 `kMaterialPadding`（4）作为基础单位
- **断点宽度**：遵循 Material Design 3 的标准断点（600、840、1200、1600 dp）

### 非互斥性设计

`Breakpoint` 的一个重要特性是**非互斥性**：多个断点可以同时激活，系统会按照测试顺序，让最后一个匹配的断点优先。这种设计允许开发者创建重叠的断点范围，并通过顺序控制优先级。

## 构造函数详解

### 1. Breakpoint() - 通用构造函数

这是最灵活的构造函数，允许完全自定义所有参数。

```dart
const Breakpoint({
  this.beginWidth,           // 起始宽度（dp），null 表示无下限
  this.endWidth,             // 结束宽度（dp），null 表示无上限
  this.beginHeight,          // 起始高度（dp），null 表示无下限
  this.endHeight,            // 结束高度（dp），null 表示无上限
  this.andUp = false,        // 是否包含更大尺寸
  this.platform,             // 目标平台集合，null 表示所有平台
  this.spacing = kMaterialMediumAndUpSpacing,  // 默认间距：24
  this.margin = kMaterialMediumAndUpMargin,    // 默认边距：24
  this.padding = kMaterialPadding,              // 默认内边距：4
  this.recommendedPanes = 1, // 推荐面板数：1
  this.maxPanes = 1,         // 最大面板数：1
});
```

**使用场景**：当需要创建完全自定义的断点时使用。

**示例**：

```dart
// 创建一个自定义断点：宽度在 500-700 dp 之间，仅限桌面平台
final customBreakpoint = Breakpoint(
  beginWidth: 500,
  endWidth: 700,
  platform: Breakpoint.desktop,
  spacing: 32,
  recommendedPanes: 2,
);
```

### 2. Breakpoint.standard() - 标准/回退断点

这是一个通用的回退断点，当没有其他断点激活时使用。

```dart
const Breakpoint.standard({this.platform})
    : beginWidth = -1,              // 从 -1 dp 开始
      endWidth = null,              // 无上限
      beginHeight = null,           // 无高度限制
      endHeight = null,             // 无高度上限
      spacing = kMaterialMediumAndUpSpacing,  // 24
      margin = kMaterialMediumAndUpMargin,    // 24
      padding = kMaterialPadding,              // 4
      recommendedPanes = 1,         // 1 个面板
      maxPanes = 1,                 // 最多 1 个面板
      andUp = true;                 // 包含所有更大尺寸
```

**特点**：

- `beginWidth = -1` 确保几乎总是匹配
- `andUp = true` 表示包含所有大于等于 -1 的宽度
- 作为默认回退选项

**使用场景**：作为断点列表的最后一个元素，确保总是有一个断点激活。

### 3. Breakpoint.small() - 小屏幕断点

适用于小屏幕设备（主要是手机）。

```dart
const Breakpoint.small({this.andUp = false, this.platform})
    : beginWidth = 0,                    // 从 0 dp 开始
      endWidth = 600,                     // 到 600 dp 结束
      beginHeight = null,                 // 无高度下限
      endHeight = 480,                    // 高度上限 480 dp
      spacing = kMaterialCompactSpacing,  // 0（紧凑间距）
      margin = kMaterialCompactMargin,    // 16（紧凑边距）
      padding = kMaterialPadding,         // 4
      recommendedPanes = 1,               // 1 个面板
      maxPanes = 1;                       // 最多 1 个面板
```

**特点**：

- 宽度范围：0-600 dp
- 使用紧凑的间距和边距（符合小屏幕设计）
- 只支持单面板布局

**使用场景**：手机竖屏和横屏（宽度小于 600 dp）。

**示例**：

```dart
// 仅限移动平台的小屏幕断点
final smallMobile = Breakpoint.small(platform: Breakpoint.mobile);

// 包含所有大于等于 0 dp 的屏幕
final smallAndUp = Breakpoint.small(andUp: true);
```

### 4. Breakpoint.medium() - 中等屏幕断点

适用于中等屏幕设备（大手机、小平板）。

```dart
const Breakpoint.medium({this.andUp = false, this.platform})
    : beginWidth = 600,                     // 从 600 dp 开始
      endWidth = 840,                       // 到 840 dp 结束
      beginHeight = 480,                    // 高度下限 480 dp
      endHeight = 900,                      // 高度上限 900 dp
      spacing = kMaterialMediumAndUpSpacing, // 24
      margin = kMaterialMediumAndUpMargin,   // 24
      padding = kMaterialPadding * 2,        // 8（2 倍基础内边距）
      recommendedPanes = 1,                  // 1 个面板
      maxPanes = 2;                          // 最多 2 个面板
```

**特点**：

- 宽度范围：600-840 dp
- 高度范围：480-900 dp
- 内边距增加到 8（2 倍基础值）
- 可以支持最多 2 个面板

**使用场景**：大屏手机、小平板、小桌面窗口。

### 5. Breakpoint.mediumLarge() - 中大屏幕断点

适用于中大屏幕设备（平板、中等桌面窗口）。

```dart
const Breakpoint.mediumLarge({this.andUp = false, this.platform})
    : beginWidth = 840,                     // 从 840 dp 开始
      endWidth = 1200,                      // 到 1200 dp 结束
      beginHeight = 900,                    // 高度下限 900 dp
      endHeight = null,                     // 无高度上限
      spacing = kMaterialMediumAndUpSpacing, // 24
      margin = kMaterialMediumAndUpMargin,   // 24
      padding = kMaterialPadding * 3,        // 12（3 倍基础内边距）
      recommendedPanes = 2,                  // 推荐 2 个面板
      maxPanes = 2;                          // 最多 2 个面板
```

**特点**：

- 宽度范围：840-1200 dp
- 高度下限：900 dp（无上限）
- 内边距增加到 12
- 推荐使用 2 个面板

**使用场景**：平板、中等桌面窗口。

### 6. Breakpoint.large() - 大屏幕断点

适用于大屏幕设备（大桌面窗口）。

```dart
const Breakpoint.large({this.andUp = false, this.platform})
    : beginWidth = 1200,                    // 从 1200 dp 开始
      endWidth = 1600,                      // 到 1600 dp 结束
      beginHeight = 900,                    // 高度下限 900 dp
      endHeight = null,                     // 无高度上限
      spacing = kMaterialMediumAndUpSpacing, // 24
      margin = kMaterialMediumAndUpMargin,   // 24
      padding = kMaterialPadding * 4,        // 16（4 倍基础内边距）
      recommendedPanes = 2,                  // 推荐 2 个面板
      maxPanes = 2;                          // 最多 2 个面板
```

**特点**：

- 宽度范围：1200-1600 dp
- 内边距增加到 16
- 推荐使用 2 个面板

**使用场景**：大桌面窗口、大屏显示器。

### 7. Breakpoint.extraLarge() - 超大屏幕断点

适用于超大屏幕设备（超大桌面窗口、超宽屏）。

```dart
const Breakpoint.extraLarge({this.andUp = false, this.platform})
    : beginWidth = 1600,                    // 从 1600 dp 开始
      endWidth = null,                      // 无上限
      beginHeight = 900,                    // 高度下限 900 dp
      endHeight = null,                     // 无高度上限
      spacing = kMaterialMediumAndUpSpacing, // 24
      margin = kMaterialMediumAndUpMargin,   // 24
      padding = kMaterialPadding * 5,        // 20（5 倍基础内边距）
      recommendedPanes = 2,                  // 推荐 2 个面板
      maxPanes = 3;                          // 最多 3 个面板
```

**特点**：

- 宽度从 1600 dp 开始，无上限
- 内边距增加到 20（最大）
- 可以支持最多 3 个面板

**使用场景**：超大桌面窗口、超宽屏显示器。

## 属性详解

### 尺寸相关属性

#### beginWidth / endWidth

```dart
final double? beginWidth;  // 起始宽度（dp）
final double? endWidth;    // 结束宽度（dp）
```

- **作用**：定义断点激活的宽度范围
- **null 值处理**：
  - `beginWidth = null`：无下限，相当于 `double.negativeInfinity`
  - `endWidth = null`：无上限，相当于 `double.infinity`
- **范围判断**：在 `isActive()` 方法中，如果 `andUp = false`，则使用 `[beginWidth, endWidth)`（左闭右开区间）

**示例**：

```dart
// 宽度在 600-840 dp 之间
Breakpoint(beginWidth: 600, endWidth: 840)

// 宽度大于等于 1200 dp
Breakpoint(beginWidth: 1200, endWidth: null, andUp: true)
```

#### beginHeight / endHeight

```dart
final double? beginHeight;  // 起始高度（dp）
final double? endHeight;    // 结束高度（dp）
```

- **作用**：定义断点激活的高度范围
- **null 值处理**：与宽度属性相同
- **方向感知**：在 `isActive()` 方法中，高度检查会考虑屏幕方向（横屏/竖屏）

**示例**：

```dart
// 高度在 480-900 dp 之间
Breakpoint(beginHeight: 480, endHeight: 900)

// 高度大于等于 900 dp
Breakpoint(beginHeight: 900, endHeight: null)
```

### 行为控制属性

#### andUp

```dart
final bool andUp;  // 是否包含更大尺寸
```

- **作用**：控制断点是否包含所有大于等于设定尺寸的屏幕
- **默认值**：`false`
- **影响**：
  - `andUp = false`：使用范围判断 `[beginWidth, endWidth)`
  - `andUp = true`：使用下限判断 `width >= beginWidth`

**示例**：

```dart
// 仅匹配 600-840 dp
Breakpoint.medium(andUp: false)

// 匹配所有 >= 600 dp 的屏幕
Breakpoint.medium(andUp: true)
```

#### platform

```dart
final Set<TargetPlatform>? platform;  // 目标平台集合
```

- **作用**：限制断点只在特定平台生效
- **null 值**：`null` 表示在所有平台都生效
- **平台常量**：
  - `Breakpoint.desktop`：桌面平台（Linux、macOS、Windows）
  - `Breakpoint.mobile`：移动平台（Android、Fuchsia、iOS）

**示例**：

```dart
// 仅限桌面平台
Breakpoint.small(platform: Breakpoint.desktop)

// 仅限移动平台
Breakpoint.medium(platform: Breakpoint.mobile)

// 所有平台
Breakpoint.large(platform: null)  // 或省略 platform 参数
```

### Material Design 3 间距属性

#### spacing

```dart
final double spacing;  // 默认间距
```

- **作用**：Material Design 3 推荐的组件间距
- **标准值**：
  - `kMaterialCompactSpacing = 0`（紧凑布局）
  - `kMaterialMediumAndUpSpacing = 24`（中等及以上布局）
- **使用场景**：组件之间的间距、列表项间距等

#### margin

```dart
final double margin;  // 默认边距
```

- **作用**：Material Design 3 推荐的页面边距
- **标准值**：
  - `kMaterialCompactMargin = 16`（紧凑布局）
  - `kMaterialMediumAndUpMargin = 24`（中等及以上布局）
- **使用场景**：页面内容与屏幕边缘的距离

#### padding

```dart
final double padding;  // 默认内边距
```

- **作用**：Material Design 3 推荐的组件内边距
- **基础值**：`kMaterialPadding = 4`
- **缩放规则**：
  - Small：`padding = 4`（1 倍）
  - Medium：`padding = 8`（2 倍）
  - MediumLarge：`padding = 12`（3 倍）
  - Large：`padding = 16`（4 倍）
  - ExtraLarge：`padding = 20`（5 倍）
- **使用场景**：组件内部的内边距、卡片内边距等

### 布局建议属性

#### recommendedPanes

```dart
final int recommendedPanes;  // 推荐面板数
```

- **作用**：Material Design 3 推荐的该断点下的面板数量
- **取值范围**：1-3
- **标准值**：
  - Small/Medium：1 个面板
  - MediumLarge/Large：2 个面板
  - ExtraLarge：2 个面板（但可以支持 3 个）

#### maxPanes

```dart
final int maxPanes;  // 最大面板数
```

- **作用**：该断点下可以显示的最大面板数量
- **取值范围**：1-3
- **标准值**：
  - Small/Medium：1 个面板
  - MediumLarge/Large：2 个面板
  - ExtraLarge：3 个面板

## 核心方法

### isActive(BuildContext context)

这是 `Breakpoint` 类的核心方法，用于判断当前屏幕是否匹配该断点。

```dart
bool isActive(BuildContext context) {
  // 1. 平台检查
  final TargetPlatform host = Theme.of(context).platform;
  final bool isRightPlatform = platform?.contains(host) ?? true;

  // 2. 获取屏幕信息
  final double width = MediaQuery.sizeOf(context).width;
  final double height = MediaQuery.sizeOf(context).height;
  final Orientation orientation = MediaQuery.orientationOf(context);
  final bool isPortrait = orientation == Orientation.portrait;

  // 3. 处理 null 值，设置边界
  final double lowerBoundWidth = beginWidth ?? double.negativeInfinity;
  final double upperBoundWidth = endWidth ?? double.infinity;
  final double lowerBoundHeight = beginHeight ?? double.negativeInfinity;
  final double upperBoundHeight = endHeight ?? double.infinity;

  // 4. 宽度检查
  final bool isWidthActive = andUp
      ? width >= lowerBoundWidth
      : width >= lowerBoundWidth && width < upperBoundWidth;

  // 5. 高度检查（考虑方向）
  final bool isHeightActive = isPortrait || isWidthActive || andUp
      ? isWidthActive || height >= lowerBoundHeight
      : height >= lowerBoundHeight && height < upperBoundHeight;

  // 6. 综合判断
  return isWidthActive && isHeightActive && isRightPlatform;
}
```

#### 执行流程详解

1. **平台检查**：
   - 从 `Theme.of(context).platform` 获取当前平台
   - 如果 `platform` 为 `null`，则所有平台都匹配
   - 如果 `platform` 不为 `null`，则检查当前平台是否在集合中

2. **获取屏幕信息**：
   - 使用 `MediaQuery.sizeOf(context)` 获取屏幕宽度和高度
   - 使用 `MediaQuery.orientationOf(context)` 获取屏幕方向

3. **边界处理**：
   - `beginWidth = null` → `lowerBoundWidth = double.negativeInfinity`
   - `endWidth = null` → `upperBoundWidth = double.infinity`
   - 高度边界处理相同

4. **宽度检查**：
   - 如果 `andUp = true`：只检查 `width >= lowerBoundWidth`
   - 如果 `andUp = false`：检查 `width >= lowerBoundWidth && width < upperBoundWidth`（左闭右开区间）

5. **高度检查**（方向感知逻辑）：
   - 如果是竖屏（`isPortrait = true`）或宽度已激活或 `andUp = true`：
     - 使用简化逻辑：`isWidthActive || height >= lowerBoundHeight`
   - 否则（横屏且宽度未激活）：
     - 使用范围检查：`height >= lowerBoundHeight && height < upperBoundHeight`

6. **综合判断**：
   - 所有条件必须同时满足：平台匹配、宽度匹配、高度匹配

#### 方向感知逻辑说明

高度检查的方向感知逻辑是为了处理横屏和竖屏的不同情况：

- **竖屏**：通常高度较大，主要依赖宽度判断
- **横屏**：如果宽度未激活，则严格检查高度范围；如果宽度已激活，则放宽高度要求

## 静态方法

### maybeActiveBreakpointFromSlotLayout()

```dart
static Breakpoint? maybeActiveBreakpointFromSlotLayout(BuildContext context)
```

- **作用**：从 `SlotLayout` 中获取当前激活的断点
- **返回值**：如果找到 `SlotLayout` 且存在激活的断点，返回该断点；否则返回 `null`
- **实现逻辑**：
  1. 查找最近的 `SlotLayout` 祖先组件
  2. 如果找到，从 `SlotLayout.config` 的键（断点列表）中查找激活的断点
  3. 如果未找到 `SlotLayout`，返回 `null`

**使用场景**：在 `SlotLayout` 的子组件中获取当前使用的断点。

### defaultBreakpointOf()

```dart
static Breakpoint defaultBreakpointOf(BuildContext context)
```

- **作用**：获取基于 `BuildContext` 的默认断点
- **返回值**：从 `Breakpoints.all` 中查找激活的断点，如果未找到则返回 `Breakpoints.standard`
- **实现逻辑**：
  1. 在 `Breakpoints.all` 列表中查找激活的断点
  2. 如果找到，返回该断点
  3. 如果未找到，返回 `Breakpoints.standard` 作为回退

**使用场景**：当没有 `SlotLayout` 时，获取系统默认的断点。

### activeBreakpointOf()

```dart
static Breakpoint activeBreakpointOf(BuildContext context)
```

- **作用**：获取当前激活的断点（最常用的方法）
- **返回值**：总是返回一个有效的 `Breakpoint`（不会为 `null`）
- **实现逻辑**：
  1. 首先尝试从 `SlotLayout` 获取断点
  2. 如果未找到，使用默认断点

**使用场景**：在任何地方获取当前激活的断点，这是最常用的方法。

**示例**：

```dart
// 在 Widget 中获取当前断点
final currentBreakpoint = Breakpoint.activeBreakpointOf(context);

// 根据断点调整布局
if (currentBreakpoint.recommendedPanes >= 2) {
  // 显示多面板布局
} else {
  // 显示单面板布局
}
```

### activeBreakpointIn()

```dart
static Breakpoint? activeBreakpointIn(
    BuildContext context, List<Breakpoint> breakpoints)
```

- **作用**：从给定的断点列表中查找当前激活的断点
- **参数**：
  - `context`：构建上下文
  - `breakpoints`：要检查的断点列表
- **返回值**：找到的激活断点，如果未找到则返回 `null`
- **优先级逻辑**：
  1. 遍历断点列表
  2. 如果断点激活且指定了平台（`platform != null`），立即返回（平台特定断点优先）
  3. 如果断点激活但未指定平台，保存为候选
  4. 返回最后一个匹配的非平台特定断点，或平台特定断点

**使用场景**：在自定义断点列表中查找激活的断点。

**示例**：

```dart
final customBreakpoints = [
  Breakpoint.small(),
  Breakpoint.medium(),
  Breakpoint.large(),
];

final active = Breakpoint.activeBreakpointIn(context, customBreakpoints);
```

### isDesktop() / isMobile()

```dart
static bool isDesktop(BuildContext context)
static bool isMobile(BuildContext context)
```

- **作用**：判断当前平台是否为桌面或移动平台
- **实现**：检查 `Theme.of(context).platform` 是否在对应的平台集合中

**使用场景**：快速判断平台类型，进行平台特定的 UI 调整。

**示例**：

```dart
if (Breakpoint.isDesktop(context)) {
  // 桌面平台特定的 UI
} else if (Breakpoint.isMobile(context)) {
  // 移动平台特定的 UI
}
```

## 比较运算符

`Breakpoint` 类实现了比较运算符，用于比较两个断点的大小关系。比较基于宽度和高度的边界值。

### operator >

```dart
bool operator >(Breakpoint breakpoint)
```

- **作用**：判断当前断点是否大于给定断点
- **逻辑**：比较所有边界值（宽度和高度），所有边界都更大时才返回 `true`
- **null 值处理**：
  - `beginWidth/beginHeight = null` → 使用 `double.negativeInfinity`
  - `endWidth/endHeight = null` → 使用 `double.infinity`

**示例**：

```dart
final large = Breakpoint.large();
final medium = Breakpoint.medium();

print(large > medium);  // true（large 的所有边界都大于 medium）
```

### operator <

```dart
bool operator <(Breakpoint breakpoint)
```

- **作用**：判断当前断点是否小于给定断点
- **逻辑**：比较所有边界值，所有边界都更小时才返回 `true`

### operator >=

```dart
bool operator >=(Breakpoint breakpoint)
```

- **作用**：判断当前断点是否大于等于给定断点
- **逻辑**：所有边界都大于等于时才返回 `true`

### operator <=

```dart
bool operator <=(Breakpoint breakpoint)
```

- **作用**：判断当前断点是否小于等于给定断点
- **逻辑**：所有边界都小于等于时才返回 `true`

### between()

```dart
bool between(Breakpoint lower, Breakpoint upper)
```

- **作用**：判断当前断点是否在给定的两个断点之间
- **逻辑**：`this >= lower && this < upper`
- **使用场景**：检查断点是否在某个范围内

**示例**：

```dart
final medium = Breakpoint.medium();
final small = Breakpoint.small();
final large = Breakpoint.large();

print(medium.between(small, large));  // true
```

## 平台常量

### Breakpoint.desktop

```dart
static const Set<TargetPlatform> desktop = <TargetPlatform>{
  TargetPlatform.linux,
  TargetPlatform.macOS,
  TargetPlatform.windows
};
```

- **作用**：桌面平台集合
- **包含平台**：Linux、macOS、Windows
- **使用场景**：创建仅限桌面平台的断点

### Breakpoint.mobile

```dart
static const Set<TargetPlatform> mobile = <TargetPlatform>{
  TargetPlatform.android,
  TargetPlatform.fuchsia,
  TargetPlatform.iOS,
};
```

- **作用**：移动平台集合
- **包含平台**：Android、Fuchsia、iOS
- **使用场景**：创建仅限移动平台的断点

## 使用示例

### 示例 1：创建自定义断点

```dart
// 创建一个仅限桌面平台的中等屏幕断点
final desktopMedium = Breakpoint(
  beginWidth: 600,
  endWidth: 840,
  platform: Breakpoint.desktop,
  spacing: 32,  // 自定义间距
  recommendedPanes: 2,
  maxPanes: 2,
);
```

### 示例 2：在 SlotLayout 中使用

```dart
SlotLayout(
  config: <Breakpoint, SlotLayoutConfig>{
    Breakpoints.small: SlotLayoutConfig(
      inSecondaryPane: false,
      body: (context) => MobileLayout(),
    ),
    Breakpoints.medium: SlotLayoutConfig(
      inSecondaryPane: false,
      body: (context) => TabletLayout(),
    ),
    Breakpoints.large: SlotLayoutConfig(
      inSecondaryPane: true,
      body: (context) => DesktopLayout(),
    ),
  },
)
```

### 示例 3：条件渲染

```dart
class AdaptiveWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final breakpoint = Breakpoint.activeBreakpointOf(context);

    // 根据断点调整 UI
    if (breakpoint.recommendedPanes >= 2) {
      return Row(
        children: [
          Expanded(child: PrimaryPane()),
          Expanded(child: SecondaryPane()),
        ],
      );
    } else {
      return SinglePane();
    }
  }
}
```

### 示例 4：使用间距属性

```dart
class SpacedContent extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final breakpoint = Breakpoint.activeBreakpointOf(context);

    return Padding(
      padding: EdgeInsets.all(breakpoint.padding),
      child: Column(
        children: [
          Widget1(),
          SizedBox(height: breakpoint.spacing),
          Widget2(),
        ],
      ),
    );
  }
}
```

### 示例 5：平台特定布局

```dart
class PlatformAwareLayout extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    if (Breakpoint.isDesktop(context)) {
      return DesktopNavigation();
    } else if (Breakpoint.isMobile(context)) {
      return MobileNavigation();
    }
    return DefaultNavigation();
  }
}
```

## 设计模式和最佳实践

### 1. 断点的非互斥性设计

`Breakpoint` 采用非互斥设计，多个断点可以同时激活。系统按照测试顺序，让最后一个匹配的断点优先。这种设计的好处：

- **灵活性**：可以创建重叠的断点范围
- **优先级控制**：通过顺序控制哪个断点最终生效
- **回退机制**：可以设置默认断点作为回退

**最佳实践**：

```dart
// 将更具体的断点放在前面，通用断点放在后面
final breakpoints = [
  Breakpoint.small(platform: Breakpoint.mobile),  // 最具体
  Breakpoint.small(platform: Breakpoint.desktop),  // 次具体
  Breakpoint.small(),                              // 通用
  Breakpoints.standard,                           // 回退
];
```

### 2. 平台特定断点的优先级

在 `activeBreakpointIn()` 方法中，平台特定的断点（`platform != null`）具有更高的优先级。一旦找到平台特定的激活断点，立即返回，不再继续查找。

**最佳实践**：

```dart
// 优先使用平台特定的断点
final breakpoints = [
  Breakpoint.medium(platform: Breakpoint.desktop),  // 优先匹配
  Breakpoint.medium(),                               // 回退
];
```

### 3. Material Design 3 规范遵循

`Breakpoint` 严格遵循 Material Design 3 的设计规范：

- **断点宽度**：600、840、1200、1600 dp
- **间距值**：紧凑布局使用 0，中等及以上使用 24
- **边距值**：紧凑布局使用 16，中等及以上使用 24
- **内边距**：基于 4 的倍数递增
- **面板数量**：根据屏幕大小推荐 1-3 个面板

**最佳实践**：使用预定义的断点构造函数（`small`、`medium` 等），而不是完全自定义，以确保符合 Material Design 3 规范。

### 4. 断点选择策略

1. **优先使用标准断点**：使用 `Breakpoints` 类中预定义的断点
2. **平台特定优化**：为不同平台创建特定断点以优化体验
3. **提供回退**：总是包含 `Breakpoints.standard` 作为回退
4. **测试顺序**：将更具体的断点放在列表前面

### 5. 性能考虑

- `isActive()` 方法会被频繁调用，但实现高效（主要是数值比较）
- 使用 `const` 构造函数创建断点，避免重复创建
- 在 `SlotLayout.config` 中使用 `const` 断点作为键

## 总结

`Breakpoint` 类是 `flutter_adaptive_scaffold` 包中响应式布局系统的核心。它通过定义屏幕条件（宽度、高度、平台）和提供 Material Design 3 布局建议，帮助开发者创建能够自适应各种设备的 Flutter 应用。

### 关键要点

1. **非互斥性**：多个断点可以同时激活，最后一个匹配的断点优先
2. **平台感知**：支持创建平台特定的断点
3. **方向感知**：高度检查会考虑屏幕方向
4. **Material Design 3**：严格遵循 Material Design 3 的设计规范
5. **灵活扩展**：支持完全自定义的断点创建

### 常用方法

- `Breakpoint.activeBreakpointOf(context)`：获取当前激活的断点（最常用）
- `breakpoint.isActive(context)`：判断断点是否激活
- `Breakpoint.isDesktop(context)` / `Breakpoint.isMobile(context)`：平台判断

通过合理使用 `Breakpoint` 类，开发者可以轻松创建响应式、自适应的 Flutter 应用，提供优秀的用户体验。
