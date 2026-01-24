# 第 3 章 断点系统详解

## 引言

断点（Breakpoint）是响应式布局的核心概念。它定义了在不同屏幕尺寸下应该使用哪种布局。本章将深入解析 `Breakpoint` 类的设计原理、标准断点的定义、如何创建自定义断点，以及断点的匹配逻辑。

## Breakpoint 类的设计原理

### 核心属性

`Breakpoint` 类通过以下属性定义屏幕条件：

```dart
class Breakpoint {
  final double? beginWidth;      // 起始宽度（dp）
  final double? endWidth;         // 结束宽度（dp）
  final double? beginHeight;      // 起始高度（dp）
  final double? endHeight;         // 结束高度（dp）
  final bool andUp;               // 是否包含更大尺寸
  final Set<TargetPlatform>? platform; // 目标平台
  final double spacing;           // 推荐间距
  final double margin;            // 推荐边距
  final double padding;           // 推荐内边距
  final int recommendedPanes;     // 推荐面板数
  final int maxPanes;             // 最大面板数
}
```

### isActive 方法

`isActive(BuildContext context)` 是断点的核心方法，用于判断当前屏幕是否匹配该断点：

```dart
bool isActive(BuildContext context) {
  // 1. 检查平台
  final TargetPlatform host = Theme.of(context).platform;
  final bool isRightPlatform = platform?.contains(host) ?? true;
  
  // 2. 获取屏幕尺寸
  final double width = MediaQuery.sizeOf(context).width;
  final double height = MediaQuery.sizeOf(context).height;
  final Orientation orientation = MediaQuery.orientationOf(context);
  
  // 3. 检查宽度范围
  final bool isWidthActive = andUp
      ? width >= lowerBoundWidth
      : width >= lowerBoundWidth && width < upperBoundWidth;
  
  // 4. 检查高度范围（考虑方向）
  final bool isHeightActive = isPortrait || isWidthActive || andUp
      ? height >= lowerBoundHeight && height < upperBoundHeight
      : true;
  
  // 5. 综合判断
  return isRightPlatform && isWidthActive && isHeightActive;
}
```

### 设计特点

1. **非互斥性**：多个断点可以同时激活，最后一个匹配的断点优先
2. **平台感知**：可以限制断点只在特定平台生效
3. **方向感知**：高度检查会考虑屏幕方向
4. **Material Design 3 规范**：包含推荐的间距、边距等设计参数

## 标准断点详解

### Breakpoints 类

`Breakpoints` 类提供了 Material Design 3 定义的标准断点：

```dart
class Breakpoints {
  static const Breakpoint standard = Breakpoint.standard();
  static const Breakpoint small = Breakpoint.small();
  static const Breakpoint medium = Breakpoint.medium();
  static const Breakpoint mediumLarge = Breakpoint.mediumLarge();
  static const Breakpoint large = Breakpoint.large();
  static const Breakpoint extraLarge = Breakpoint.extraLarge();
  
  // 带 "AndUp" 后缀的变体
  static const Breakpoint smallAndUp = Breakpoint.small(andUp: true);
  static const Breakpoint mediumAndUp = Breakpoint.medium(andUp: true);
  // ...
  
  // 平台特定变体
  static const Breakpoint smallDesktop = Breakpoint.small(platform: Breakpoint.desktop);
  static const Breakpoint smallMobile = Breakpoint.small(platform: Breakpoint.mobile);
  // ...
}
```

### 各断点详解

#### 1. Small（0-600 dp）

**定义**：

```dart
Breakpoint.small()
// beginWidth: 0
// endWidth: 600
// beginHeight: null
// endHeight: 480
// recommendedPanes: 1
// maxPanes: 1
```

**特点**：

- 手机竖屏的标准尺寸
- 推荐单面板布局
- 紧凑的间距和边距
- 通常使用 `BottomNavigationBar`

**使用场景**：

```dart
SlotLayout(
  config: {
    Breakpoints.small: SlotLayout.from(
      key: const Key('Small Layout'),
      builder: (_) => MobileLayout(),
    ),
  },
)
```

#### 2. Medium（600-840 dp）

**定义**：

```dart
Breakpoint.medium()
// beginWidth: 600
// endWidth: 840
// beginHeight: 480
// endHeight: 900
// recommendedPanes: 1
// maxPanes: 2
```

**特点**：

- 手机横屏或小平板
- 可以支持双面板
- 中等间距
- 开始使用 `NavigationRail`

**使用场景**：

```dart
SlotLayout(
  config: {
    Breakpoints.medium: SlotLayout.from(
      key: const Key('Medium Layout'),
      builder: (_) => TabletLayout(),
    ),
  },
)
```

#### 3. Medium Large（840-1200 dp）

**定义**：

```dart
Breakpoint.mediumLarge()
// beginWidth: 840
// endWidth: 1200
// beginHeight: 900
// endHeight: null
// recommendedPanes: 2
// maxPanes: 2
```

**特点**：

- 平板标准尺寸
- 推荐双面板布局
- 更大的间距
- 扩展的 `NavigationRail`

**使用场景**：

```dart
SlotLayout(
  config: {
    Breakpoints.mediumLarge: SlotLayout.from(
      key: const Key('MediumLarge Layout'),
      builder: (_) => TabletExtendedLayout(),
    ),
  },
)
```

#### 4. Large（1200-1600 dp）

**定义**：

```dart
Breakpoint.large()
// beginWidth: 1200
// endWidth: 1600
// beginHeight: 900
// endHeight: null
// recommendedPanes: 2
// maxPanes: 2
```

**特点**：

- 桌面标准尺寸
- 推荐双面板布局
- 更大的内边距
- 完整的导航和详情视图

**使用场景**：

```dart
SlotLayout(
  config: {
    Breakpoints.large: SlotLayout.from(
      key: const Key('Large Layout'),
      builder: (_) => DesktopLayout(),
    ),
  },
)
```

#### 5. Extra Large（1600+ dp）

**定义**：

```dart
Breakpoint.extraLarge()
// beginWidth: 1600
// endWidth: null
// beginHeight: 900
// endHeight: null
// recommendedPanes: 2
// maxPanes: 3
```

**特点**：

- 大桌面显示器
- 可以支持三面板布局
- 最大的间距和内边距
- 充分利用屏幕空间

**使用场景**：

```dart
SlotLayout(
  config: {
    Breakpoints.extraLarge: SlotLayout.from(
      key: const Key('ExtraLarge Layout'),
      builder: (_) => LargeDesktopLayout(),
    ),
  },
)
```

#### 6. Standard（通用断点）

**定义**：

```dart
Breakpoint.standard()
// beginWidth: -1
// endWidth: null
// andUp: true
```

**特点**：

- 作为后备（fallback）断点
- 匹配所有尺寸
- 当没有其他断点匹配时使用

**使用场景**：

```dart
SlotLayout(
  config: {
    Breakpoints.standard: SlotLayout.from(
      key: const Key('Default Layout'),
      builder: (_) => DefaultLayout(),
    ),
  },
)
```

### "AndUp" 变体

带 `andUp: true` 的断点会匹配所有大于等于起始宽度的屏幕：

```dart
Breakpoints.mediumAndUp  // 匹配 600+ dp 的所有屏幕
Breakpoints.largeAndUp   // 匹配 1200+ dp 的所有屏幕
```

**使用场景**：

```dart
// 在中等及以上屏幕显示详情视图
secondaryBody: SlotLayout(
  config: {
    Breakpoints.mediumAndUp: SlotLayout.from(
      key: const Key('Secondary Body'),
      builder: (_) => DetailView(),
    ),
  },
)
```

### 平台特定断点

可以限制断点只在特定平台生效：

```dart
Breakpoints.smallDesktop  // 仅桌面平台的小屏幕
Breakpoints.smallMobile  // 仅移动平台的小屏幕
```

**使用场景**：

```dart
// 桌面平台使用不同的布局
SlotLayout(
  config: {
    Breakpoints.smallDesktop: SlotLayout.from(
      key: const Key('Desktop Small'),
      builder: (_) => DesktopSmallLayout(),
    ),
    Breakpoints.smallMobile: SlotLayout.from(
      key: const Key('Mobile Small'),
      builder: (_) => MobileSmallLayout(),
    ),
  },
)
```

## 自定义断点

### 创建自定义断点

你可以创建完全自定义的断点：

```dart
// 自定义断点：超宽屏（2000+ dp）
const Breakpoint ultraWide = Breakpoint(
  beginWidth: 2000,
  endWidth: null,
  andUp: true,
  recommendedPanes: 3,
  maxPanes: 4,
);

// 自定义断点：特定范围（800-1000 dp）
const Breakpoint customRange = Breakpoint(
  beginWidth: 800,
  endWidth: 1000,
  spacing: 32,
  margin: 32,
  padding: 8,
);
```

### 基于标准断点扩展

更常见的做法是基于标准断点进行扩展：

```dart
// 扩展 medium 断点，添加平台限制
const Breakpoint mediumTablet = Breakpoint.medium(
  platform: Breakpoint.mobile,
);

// 扩展 large 断点，修改推荐面板数
const Breakpoint largeCustom = Breakpoint(
  beginWidth: 1200,
  endWidth: 1600,
  recommendedPanes: 3,  // 自定义推荐面板数
  maxPanes: 3,
);
```

### 实际应用示例

```dart
// 为折叠屏设备创建特殊断点
const Breakpoint foldable = Breakpoint(
  beginWidth: 600,
  endWidth: 840,
  beginHeight: 900,
  endHeight: null,
  recommendedPanes: 2,
  maxPanes: 2,
);

// 使用自定义断点
SlotLayout(
  config: {
    foldable: SlotLayout.from(
      key: const Key('Foldable Layout'),
      builder: (_) => FoldableLayout(),
    ),
  },
)
```

## 断点匹配逻辑

### 匹配顺序

`SlotLayout` 使用 `pickWidget` 方法选择匹配的断点：

```dart
static SlotLayoutConfig? pickWidget(
  BuildContext context,
  Map<Breakpoint, SlotLayoutConfig?> config,
) {
  final Breakpoint? breakpoint = Breakpoint.activeBreakpointIn(
    context,
    config.keys.toList(),
  );
  return breakpoint != null && config.containsKey(breakpoint)
      ? config[breakpoint]
      : null;
}
```

### activeBreakpointIn 方法

该方法会：

1. **遍历所有断点**：按配置顺序检查每个断点
2. **调用 isActive**：判断当前屏幕是否匹配
3. **选择最后一个匹配的断点**：如果多个断点匹配，选择最后一个

**重要**：这意味着配置顺序很重要！

```dart
SlotLayout(
  config: {
    Breakpoints.standard: SlotLayout.from(...),  // 这个会匹配所有屏幕
    Breakpoints.small: SlotLayout.from(...),     // 但小屏幕会优先匹配这个
  },
)
```

### 匹配优先级示例

```dart
SlotLayout(
  config: {
    // 1. 标准断点（匹配所有，但优先级最低）
    Breakpoints.standard: SlotLayout.from(
      key: const Key('Default'),
      builder: (_) => DefaultWidget(),
    ),
    
    // 2. 小屏幕断点（600 dp 以下会匹配这个）
    Breakpoints.small: SlotLayout.from(
      key: const Key('Small'),
      builder: (_) => SmallWidget(),
    ),
    
    // 3. 中等屏幕断点（600-840 dp 会匹配这个）
    Breakpoints.medium: SlotLayout.from(
      key: const Key('Medium'),
      builder: (_) => MediumWidget(),
    ),
    
    // 4. 大屏幕及以上（1200+ dp 会匹配这个，覆盖 standard）
    Breakpoints.largeAndUp: SlotLayout.from(
      key: const Key('Large'),
      builder: (_) => LargeWidget(),
    ),
  },
)
```

**匹配结果**：

- 400 dp：匹配 `Breakpoints.small`
- 700 dp：匹配 `Breakpoints.medium`
- 1000 dp：匹配 `Breakpoints.standard`（因为没有更具体的匹配）
- 1500 dp：匹配 `Breakpoints.largeAndUp`

## 断点比较操作

`Breakpoint` 类提供了比较操作符：

```dart
// 比较断点的大小
bool operator >(Breakpoint breakpoint)
bool operator <(Breakpoint breakpoint)
bool operator >=(Breakpoint breakpoint)
bool operator <=(Breakpoint breakpoint)

// 检查断点是否在范围内
bool between(Breakpoint lower, Breakpoint upper)
```

**使用示例**：

```dart
if (Breakpoints.large > Breakpoints.medium) {
  // large 断点的起始宽度大于 medium
}

if (currentBreakpoint.between(Breakpoints.medium, Breakpoints.large)) {
  // 当前断点在 medium 和 large 之间
}
```

## 最佳实践

### 1. 使用标准断点优先

除非有特殊需求，否则优先使用标准断点：

```dart
// ✅ 推荐
Breakpoints.small
Breakpoints.medium
Breakpoints.large

// ❌ 避免（除非必要）
const Breakpoint custom = Breakpoint(beginWidth: 650, endWidth: 850);
```

### 2. 合理使用 "AndUp" 变体

使用 `andUp` 变体可以简化配置：

```dart
// ✅ 推荐：使用 mediumAndUp 而不是分别配置 medium、mediumLarge、large
Breakpoints.mediumAndUp

// ❌ 避免：重复配置
Breakpoints.medium
Breakpoints.mediumLarge
Breakpoints.large
```

### 3. 注意配置顺序

配置顺序影响匹配结果，将更具体的断点放在后面：

```dart
SlotLayout(
  config: {
    Breakpoints.standard: ...,  // 通用配置在前
    Breakpoints.small: ...,     // 具体配置在后
    Breakpoints.large: ...,     // 更具体的配置
  },
)
```

### 4. 使用平台特定断点

当移动端和桌面端需要不同布局时：

```dart
SlotLayout(
  config: {
    Breakpoints.smallMobile: ...,   // 移动端小屏幕
    Breakpoints.smallDesktop: ...,  // 桌面端小屏幕
  },
)
```

## 总结

本章我们深入了解了：

- **Breakpoint 类的设计**：核心属性和 `isActive` 方法
- **标准断点**：Small、Medium、MediumLarge、Large、ExtraLarge 的定义和特点
- **自定义断点**：如何创建和使用自定义断点
- **匹配逻辑**：断点的匹配顺序和优先级
- **最佳实践**：如何合理使用断点

在下一章中，我们将学习如何使用 `AdaptiveScaffold` 实现响应式布局。

## 练习

1. 创建一个自定义断点，用于超宽屏（1800+ dp）
2. 分析 `adaptive_scaffold_demo.dart` 中使用的断点
3. 尝试修改断点配置，观察布局变化

## 检查清单

- [ ] 理解 Breakpoint 类的核心属性和方法
- [ ] 掌握所有标准断点的定义和特点
- [ ] 能够创建自定义断点
- [ ] 理解断点的匹配逻辑和优先级
- [ ] 了解断点比较操作的使用
