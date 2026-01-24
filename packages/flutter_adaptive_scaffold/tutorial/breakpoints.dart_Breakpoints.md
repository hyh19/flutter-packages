# Breakpoints 类详解

## 引言

`Breakpoints` 类是 `flutter_adaptive_scaffold` 包中提供的一组标准断点常量集合。它基于 Material Design 3 规范，预定义了适用于不同屏幕尺寸的断点，为开发者提供了开箱即用的响应式布局解决方案。通过使用 `Breakpoints` 类中预定义的断点，开发者可以快速构建符合 Material Design 3 规范的响应式应用，而无需手动创建和配置断点。

## 类概述

### 设计目的

`Breakpoints` 类的主要目的是：

1. **提供标准断点集合**：基于 Material Design 3 规范，预定义常用的屏幕尺寸断点
2. **简化开发流程**：开发者无需手动创建断点，直接使用预定义常量即可
3. **确保一致性**：所有使用 `Breakpoints` 的应用都遵循相同的断点标准
4. **支持平台特定优化**：提供桌面和移动平台的特定版本，优化不同平台的用户体验

### 在响应式布局中的作用

`Breakpoints` 类作为标准断点集合，在响应式布局系统中起到以下作用：

- **与 `AdaptiveScaffold` 集成**：`AdaptiveScaffold` 使用 `Breakpoints` 中的部分断点作为默认配置
- **与 `SlotLayout` 配合**：开发者可以在 `SlotLayout.config` 中使用 `Breakpoints` 中的断点作为键
- **快速响应式开发**：通过使用预定义断点，快速实现不同屏幕尺寸的布局适配

### 与 Material Design 3 规范的关系

`Breakpoints` 类严格遵循 Material Design 3 的设计规范：

- **断点宽度**：使用标准断点值（600、840、1200、1600 dp）
- **平台区分**：区分桌面和移动平台，提供平台特定的断点版本
- **尺寸范围**：每个断点都有明确的宽度范围定义
- **向上兼容**：提供 `andUp` 版本，支持包含更大尺寸的屏幕

### 静态类设计

`Breakpoints` 是一个静态类，所有成员都是静态常量：

- 所有断点都是 `static const`，可以在编译时确定
- 无需实例化，直接通过类名访问（如 `Breakpoints.small`）
- 所有断点都是不可变的，确保一致性

## 标准断点详解

`Breakpoints` 类提供了多个系列的预定义断点，每个系列都有基础版本、`andUp` 版本和平台特定版本。下面按系列详细介绍：

### standard - 标准/回退断点

```dart
static const Breakpoint standard = Breakpoint.standard();
```

**特点**：

- **宽度范围**：从 -1 dp 到无穷大（`beginWidth = -1`，`endWidth = null`）
- **作用**：作为回退断点，当没有其他断点激活时使用
- **`andUp` 属性**：`true`，表示包含所有大于等于 -1 的宽度
- **使用场景**：作为断点列表的最后一个元素，确保总是有一个断点激活

**示例**：

```dart
SlotLayout(
  config: <Breakpoint, SlotLayoutConfig>{
    Breakpoints.small: SlotLayoutConfig(...),
    Breakpoints.medium: SlotLayoutConfig(...),
    Breakpoints.standard: SlotLayoutConfig(...),  // 回退配置
  },
)
```

### small 系列 - 小屏幕断点

小屏幕断点适用于宽度小于 600 dp 的屏幕，主要是手机设备。

#### small - 基础小屏幕断点

```dart
static const Breakpoint small = Breakpoint.small();
```

**特点**：

- **宽度范围**：0-600 dp（`beginWidth = 0`，`endWidth = 600`）
- **高度限制**：高度上限 480 dp
- **间距**：紧凑间距（`spacing = 0`）
- **边距**：紧凑边距（`margin = 16`）
- **面板数**：1 个面板（`recommendedPanes = 1`，`maxPanes = 1`）
- **平台**：所有平台

**使用场景**：手机竖屏和横屏（宽度小于 600 dp）。

#### smallAndUp - 小屏幕及以上

```dart
static const Breakpoint smallAndUp = Breakpoint.small(andUp: true);
```

**特点**：

- **宽度范围**：>= 0 dp（`beginWidth = 0`，`andUp = true`）
- **作用**：匹配所有宽度大于等于 0 的屏幕
- **使用场景**：作为通用断点，适用于所有屏幕尺寸

#### smallDesktop - 桌面平台小屏幕

```dart
static const Breakpoint smallDesktop =
    Breakpoint.small(platform: Breakpoint.desktop);
```

**特点**：

- **宽度范围**：0-600 dp
- **平台限制**：仅限桌面平台（Linux、macOS、Windows）
- **使用场景**：小尺寸桌面窗口、小屏幕桌面应用

#### smallMobile - 移动平台小屏幕

```dart
static const Breakpoint smallMobile =
    Breakpoint.small(platform: Breakpoint.mobile);
```

**特点**：

- **宽度范围**：0-600 dp
- **平台限制**：仅限移动平台（Android、Fuchsia、iOS）
- **使用场景**：手机应用、移动设备

### medium 系列 - 中等屏幕断点

中等屏幕断点适用于宽度在 600-840 dp 之间的屏幕，主要是大屏手机和小平板。

#### medium - 基础中等屏幕断点

```dart
static const Breakpoint medium = Breakpoint.medium();
```

**特点**：

- **宽度范围**：600-840 dp（`beginWidth = 600`，`endWidth = 840`）
- **高度范围**：480-900 dp
- **间距**：中等间距（`spacing = 24`）
- **边距**：中等边距（`margin = 24`）
- **内边距**：8（2 倍基础内边距）
- **面板数**：推荐 1 个，最多 2 个面板

**使用场景**：大屏手机、小平板、小桌面窗口。

#### mediumAndUp - 中等屏幕及以上

```dart
static const Breakpoint mediumAndUp = Breakpoint.medium(andUp: true);
```

**特点**：

- **宽度范围**：>= 600 dp（`beginWidth = 600`，`andUp = true`）
- **作用**：匹配所有宽度大于等于 600 dp 的屏幕
- **使用场景**：适用于平板和桌面设备

#### mediumDesktop - 桌面平台中等屏幕

```dart
static const Breakpoint mediumDesktop =
    Breakpoint.medium(platform: Breakpoint.desktop);
```

**特点**：

- **宽度范围**：600-840 dp
- **平台限制**：仅限桌面平台
- **使用场景**：中等尺寸桌面窗口

#### mediumMobile - 移动平台中等屏幕

```dart
static const Breakpoint mediumMobile =
    Breakpoint.medium(platform: Breakpoint.mobile);
```

**特点**：

- **宽度范围**：600-840 dp
- **平台限制**：仅限移动平台
- **使用场景**：大屏手机、小平板

### mediumLarge 系列 - 中大屏幕断点

中大屏幕断点适用于宽度在 840-1200 dp 之间的屏幕，主要是平板和中等桌面窗口。

#### mediumLarge - 基础中大屏幕断点

```dart
static const Breakpoint mediumLarge = Breakpoint.mediumLarge();
```

**特点**：

- **宽度范围**：840-1200 dp（`beginWidth = 840`，`endWidth = 1200`）
- **高度下限**：900 dp（无上限）
- **间距**：中等间距（`spacing = 24`）
- **边距**：中等边距（`margin = 24`）
- **内边距**：12（3 倍基础内边距）
- **面板数**：推荐 2 个面板（`recommendedPanes = 2`，`maxPanes = 2`）

**使用场景**：平板、中等桌面窗口。

#### mediumLargeAndUp - 中大屏幕及以上

```dart
static const Breakpoint mediumLargeAndUp =
    Breakpoint.mediumLarge(andUp: true);
```

**特点**：

- **宽度范围**：>= 840 dp（`beginWidth = 840`，`andUp = true`）
- **作用**：匹配所有宽度大于等于 840 dp 的屏幕
- **使用场景**：适用于平板和桌面设备

#### mediumLargeDesktop - 桌面平台中大屏幕

```dart
static const Breakpoint mediumLargeDesktop =
    Breakpoint.mediumLarge(platform: Breakpoint.desktop);
```

**特点**：

- **宽度范围**：840-1200 dp
- **平台限制**：仅限桌面平台
- **使用场景**：中等尺寸桌面窗口

#### mediumLargeMobile - 移动平台中大屏幕

```dart
static const Breakpoint mediumLargeMobile =
    Breakpoint.mediumLarge(platform: Breakpoint.mobile);
```

**特点**：

- **宽度范围**：840-1200 dp
- **平台限制**：仅限移动平台
- **使用场景**：大平板设备

### large 系列 - 大屏幕断点

大屏幕断点适用于宽度在 1200-1600 dp 之间的屏幕，主要是大桌面窗口。

#### large - 基础大屏幕断点

```dart
static const Breakpoint large = Breakpoint.large();
```

**特点**：

- **宽度范围**：1200-1600 dp（`beginWidth = 1200`，`endWidth = 1600`）
- **高度下限**：900 dp（无上限）
- **间距**：中等间距（`spacing = 24`）
- **边距**：中等边距（`margin = 24`）
- **内边距**：16（4 倍基础内边距）
- **面板数**：推荐 2 个面板（`recommendedPanes = 2`，`maxPanes = 2`）

**使用场景**：大桌面窗口、大屏显示器。

#### largeAndUp - 大屏幕及以上

```dart
static const Breakpoint largeAndUp = Breakpoint.large(andUp: true);
```

**特点**：

- **宽度范围**：>= 1200 dp（`beginWidth = 1200`，`andUp = true`）
- **作用**：匹配所有宽度大于等于 1200 dp 的屏幕
- **使用场景**：适用于大桌面窗口和超大屏幕

#### largeDesktop - 桌面平台大屏幕

```dart
static const Breakpoint largeDesktop =
    Breakpoint.large(platform: Breakpoint.desktop);
```

**特点**：

- **宽度范围**：1200-1600 dp
- **平台限制**：仅限桌面平台
- **使用场景**：大尺寸桌面窗口

#### largeMobile - 移动平台大屏幕

```dart
static const Breakpoint largeMobile =
    Breakpoint.large(platform: Breakpoint.mobile);
```

**特点**：

- **宽度范围**：1200-1600 dp
- **平台限制**：仅限移动平台
- **使用场景**：超大平板设备（较少见）

### extraLarge 系列 - 超大屏幕断点

超大屏幕断点适用于宽度大于 1600 dp 的屏幕，主要是超大桌面窗口和超宽屏。

#### extraLarge - 基础超大屏幕断点

```dart
static const Breakpoint extraLarge = Breakpoint.extraLarge();
```

**特点**：

- **宽度范围**：>= 1600 dp（`beginWidth = 1600`，`endWidth = null`）
- **高度下限**：900 dp（无上限）
- **间距**：中等间距（`spacing = 24`）
- **边距**：中等边距（`margin = 24`）
- **内边距**：20（5 倍基础内边距，最大内边距）
- **面板数**：推荐 2 个面板，最多 3 个面板（`recommendedPanes = 2`，`maxPanes = 3`）

**使用场景**：超大桌面窗口、超宽屏显示器。

#### extraLargeDesktop - 桌面平台超大屏幕

```dart
static const Breakpoint extraLargeDesktop =
    Breakpoint.extraLarge(platform: Breakpoint.desktop);
```

**特点**：

- **宽度范围**：>= 1600 dp
- **平台限制**：仅限桌面平台
- **使用场景**：超大尺寸桌面窗口、超宽屏显示器

#### extraLargeMobile - 移动平台超大屏幕

```dart
static const Breakpoint extraLargeMobile =
    Breakpoint.extraLarge(platform: Breakpoint.mobile);
```

**特点**：

- **宽度范围**：>= 1600 dp
- **平台限制**：仅限移动平台
- **使用场景**：超大平板设备（非常少见）

## all 列表

`Breakpoints.all` 是一个包含所有标准断点的列表，按照特定的优先级顺序排列。

```dart
static const List<Breakpoint> all = <Breakpoint>[
  smallDesktop,
  smallMobile,
  small,
  mediumDesktop,
  mediumMobile,
  medium,
  mediumLargeDesktop,
  mediumLargeMobile,
  mediumLarge,
  largeDesktop,
  largeMobile,
  large,
  extraLargeDesktop,
  extraLargeMobile,
  extraLarge,
  smallAndUp,
  mediumAndUp,
  mediumLargeAndUp,
  largeAndUp,
  standard,
];
```

### 排序逻辑

`all` 列表的排序遵循以下逻辑：

1. **平台特定断点优先**：每个尺寸的桌面和移动平台特定断点放在基础断点之前
2. **尺寸从小到大**：按照屏幕尺寸从小到大排列（small → medium → mediumLarge → large → extraLarge）
3. **`andUp` 版本在后**：所有 `andUp` 版本放在对应尺寸的基础断点之后
4. **回退断点最后**：`standard` 作为回退断点，放在列表最后

### 使用场景

`all` 列表主要用于：

1. **默认断点查找**：`Breakpoint.defaultBreakpointOf()` 方法使用此列表查找激活的断点
2. **快速遍历**：需要遍历所有标准断点时，可以使用此列表
3. **优先级参考**：了解断点的优先级顺序，用于自定义断点列表的排序

**示例**：

```dart
// 使用 all 列表查找激活的断点
final activeBreakpoint = Breakpoint.defaultBreakpointOf(context);

// 遍历所有断点
for (final breakpoint in Breakpoints.all) {
  if (breakpoint.isActive(context)) {
    print('激活的断点: $breakpoint');
  }
}
```

### 优先级说明

由于 `Breakpoint` 的非互斥性设计，多个断点可能同时激活。`all` 列表的顺序决定了优先级：

- **更具体的断点优先**：平台特定的断点比通用断点优先
- **更小的断点优先**：小屏幕断点比大屏幕断点优先（在相同平台条件下）
- **范围断点优先**：范围断点（如 `small`）比 `andUp` 断点优先
- **回退最后**：`standard` 作为最后的回退选项

## 使用示例

### 示例 1：在 SlotLayout 中使用标准断点

```dart
SlotLayout(
  config: <Breakpoint, SlotLayoutConfig>{
    // 小屏幕：单面板布局
    Breakpoints.small: SlotLayoutConfig(
      inSecondaryPane: false,
      body: (context) => MobileLayout(),
    ),
    // 中等屏幕：单面板布局
    Breakpoints.medium: SlotLayoutConfig(
      inSecondaryPane: false,
      body: (context) => TabletLayout(),
    ),
    // 中大屏幕及以上：双面板布局
    Breakpoints.mediumLargeAndUp: SlotLayoutConfig(
      inSecondaryPane: true,
      body: (context) => DesktopLayout(),
    ),
    // 回退断点
    Breakpoints.standard: SlotLayoutConfig(
      inSecondaryPane: false,
      body: (context) => DefaultLayout(),
    ),
  },
)
```

### 示例 2：平台特定布局

```dart
SlotLayout(
  config: <Breakpoint, SlotLayoutConfig>{
    // 桌面平台小屏幕
    Breakpoints.smallDesktop: SlotLayoutConfig(
      inSecondaryPane: false,
      body: (context) => CompactDesktopLayout(),
    ),
    // 移动平台小屏幕
    Breakpoints.smallMobile: SlotLayoutConfig(
      inSecondaryPane: false,
      body: (context) => MobileLayout(),
    ),
    // 通用小屏幕（回退）
    Breakpoints.small: SlotLayoutConfig(
      inSecondaryPane: false,
      body: (context) => DefaultSmallLayout(),
    ),
  },
)
```

### 示例 3：使用 andUp 版本简化配置

```dart
SlotLayout(
  config: <Breakpoint, SlotLayoutConfig>{
    // 小屏幕：单面板
    Breakpoints.small: SlotLayoutConfig(
      inSecondaryPane: false,
      body: (context) => SinglePaneLayout(),
    ),
    // 中等屏幕及以上：双面板
    Breakpoints.mediumAndUp: SlotLayoutConfig(
      inSecondaryPane: true,
      body: (context) => MultiPaneLayout(),
    ),
  },
)
```

### 示例 4：根据断点调整 UI 元素

```dart
class AdaptiveWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final breakpoint = Breakpoint.activeBreakpointOf(context);

    // 根据断点调整间距
    final spacing = breakpoint.spacing;
    final margin = breakpoint.margin;
    final padding = breakpoint.padding;

    return Padding(
      padding: EdgeInsets.all(margin),
      child: Column(
        children: [
          Widget1(),
          SizedBox(height: spacing),
          Widget2(),
          SizedBox(height: spacing),
          Padding(
            padding: EdgeInsets.all(padding),
            child: Widget3(),
          ),
        ],
      ),
    );
  }
}
```

### 示例 5：使用 all 列表进行断点检测

```dart
class BreakpointIndicator extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // 查找当前激活的断点
    Breakpoint? activeBreakpoint;
    for (final breakpoint in Breakpoints.all) {
      if (breakpoint.isActive(context)) {
        activeBreakpoint = breakpoint;
        // 平台特定断点优先，找到后立即返回
        if (breakpoint.platform != null) {
          break;
        }
      }
    }

    return Text(
      '当前断点: ${activeBreakpoint?.runtimeType ?? "未知"}',
      style: Theme.of(context).textTheme.bodyMedium,
    );
  }
}
```

### 示例 6：结合 AdaptiveScaffold 使用

```dart
AdaptiveScaffold(
  // AdaptiveScaffold 内部使用 Breakpoints 作为默认断点
  body: SlotLayout(
    config: <Breakpoint, SlotLayoutConfig>{
      Breakpoints.small: SlotLayoutConfig(
        inSecondaryPane: false,
        body: (context) => MobileBody(),
      ),
      Breakpoints.mediumAndUp: SlotLayoutConfig(
        inSecondaryPane: true,
        body: (context) => DesktopBody(),
      ),
    },
  ),
)
```

## 最佳实践

### 1. 优先使用标准断点

**建议**：优先使用 `Breakpoints` 类中预定义的断点，而不是创建自定义断点。

**原因**：

- 符合 Material Design 3 规范
- 与其他使用 `Breakpoints` 的应用保持一致
- 经过充分测试和验证
- 与 `AdaptiveScaffold` 等组件完美集成

**示例**：

```dart
// ✅ 推荐：使用标准断点
SlotLayout(
  config: <Breakpoint, SlotLayoutConfig>{
    Breakpoints.small: ...,
    Breakpoints.medium: ...,
  },
)

// ❌ 不推荐：创建自定义断点（除非有特殊需求）
SlotLayout(
  config: <Breakpoint, SlotLayoutConfig>{
    Breakpoint(beginWidth: 500, endWidth: 700): ...,
  },
)
```

### 2. 合理使用平台特定断点

**建议**：当需要在不同平台上提供不同的用户体验时，使用平台特定断点。

**场景**：

- 桌面平台需要不同的导航方式
- 移动平台需要触摸优化的 UI
- 不同平台的交互模式差异较大

**示例**：

```dart
SlotLayout(
  config: <Breakpoint, SlotLayoutConfig>{
    // 桌面平台小屏幕：可能使用侧边栏导航
    Breakpoints.smallDesktop: SlotLayoutConfig(...),
    // 移动平台小屏幕：使用底部导航栏
    Breakpoints.smallMobile: SlotLayoutConfig(...),
    // 通用小屏幕：回退配置
    Breakpoints.small: SlotLayoutConfig(...),
  },
)
```

### 3. 使用 andUp 版本简化配置

**建议**：当多个连续尺寸使用相同布局时，使用 `andUp` 版本简化配置。

**优势**：

- 减少配置代码量
- 更容易维护
- 自动支持未来更大的屏幕尺寸

**示例**：

```dart
// ✅ 推荐：使用 andUp 版本
SlotLayout(
  config: <Breakpoint, SlotLayoutConfig>{
    Breakpoints.small: SinglePaneConfig(),
    Breakpoints.mediumAndUp: MultiPaneConfig(),  // 覆盖所有 >= 600 dp 的屏幕
  },
)

// ❌ 不推荐：逐个列出所有尺寸
SlotLayout(
  config: <Breakpoint, SlotLayoutConfig>{
    Breakpoints.small: SinglePaneConfig(),
    Breakpoints.medium: MultiPaneConfig(),
    Breakpoints.mediumLarge: MultiPaneConfig(),
    Breakpoints.large: MultiPaneConfig(),
    Breakpoints.extraLarge: MultiPaneConfig(),
  },
)
```

### 4. 总是包含回退断点

**建议**：在断点配置中总是包含 `Breakpoints.standard` 作为最后的回退选项。

**原因**：

- 确保在任何情况下都有一个激活的断点
- 处理边缘情况（如非常规屏幕尺寸）
- 提供默认的布局配置

**示例**：

```dart
SlotLayout(
  config: <Breakpoint, SlotLayoutConfig>{
    Breakpoints.small: ...,
    Breakpoints.medium: ...,
    Breakpoints.standard: ...,  // ✅ 总是包含回退
  },
)
```

### 5. 遵循 all 列表的优先级顺序

**建议**：在自定义断点列表时，遵循 `Breakpoints.all` 的优先级顺序。

**顺序规则**：

1. 平台特定断点在前
2. 基础断点在后
3. `andUp` 版本在对应基础断点之后
4. 回退断点最后

**示例**：

```dart
final customBreakpoints = [
  Breakpoints.smallDesktop,      // 平台特定
  Breakpoints.smallMobile,       // 平台特定
  Breakpoints.small,              // 基础断点
  Breakpoints.mediumDesktop,     // 平台特定
  Breakpoints.mediumMobile,      // 平台特定
  Breakpoints.medium,             // 基础断点
  Breakpoints.mediumAndUp,        // andUp 版本
  Breakpoints.standard,          // 回退
];
```

### 6. 利用断点的 Material Design 3 属性

**建议**：使用断点提供的 Material Design 3 属性（`spacing`、`margin`、`padding`）来保持设计一致性。

**优势**：

- 自动适配不同屏幕尺寸的间距
- 符合 Material Design 3 规范
- 减少手动计算和硬编码

**示例**：

```dart
class AdaptiveSpacing extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final breakpoint = Breakpoint.activeBreakpointOf(context);

    return Padding(
      padding: EdgeInsets.all(breakpoint.margin),  // 使用断点的边距
      child: Column(
        children: [
          Widget1(),
          SizedBox(height: breakpoint.spacing),   // 使用断点的间距
          Widget2(),
        ],
      ),
    );
  }
}
```

### 7. 测试不同断点

**建议**：在开发过程中测试所有相关的断点，确保布局在不同屏幕尺寸下都能正常工作。

**测试方法**：

- 使用 Flutter 的设备预览功能
- 调整窗口大小（桌面应用）
- 使用不同尺寸的模拟器
- 测试横屏和竖屏方向

## 总结

`Breakpoints` 类是 `flutter_adaptive_scaffold` 包中提供的标准断点集合，为开发者提供了基于 Material Design 3 规范的响应式布局解决方案。

### 关键要点

1. **标准断点集合**：提供预定义的断点常量，符合 Material Design 3 规范
2. **平台特定支持**：为桌面和移动平台提供特定版本的断点
3. **向上兼容**：提供 `andUp` 版本，支持包含更大尺寸的屏幕
4. **优先级排序**：`all` 列表按照特定逻辑排序，确保正确的优先级
5. **回退机制**：`standard` 断点作为回退选项，确保总是有断点激活

### 常用断点

- `Breakpoints.small`：小屏幕（0-600 dp）
- `Breakpoints.medium`：中等屏幕（600-840 dp）
- `Breakpoints.mediumLarge`：中大屏幕（840-1200 dp）
- `Breakpoints.large`：大屏幕（1200-1600 dp）
- `Breakpoints.extraLarge`：超大屏幕（>= 1600 dp）
- `Breakpoints.standard`：回退断点

### 使用建议

1. 优先使用标准断点，保持与 Material Design 3 规范一致
2. 合理使用平台特定断点，优化不同平台的用户体验
3. 使用 `andUp` 版本简化配置，减少代码量
4. 总是包含 `Breakpoints.standard` 作为回退选项
5. 利用断点的 Material Design 3 属性（`spacing`、`margin`、`padding`）保持设计一致性

通过合理使用 `Breakpoints` 类，开发者可以快速构建响应式、自适应的 Flutter 应用，提供优秀的用户体验，同时确保符合 Material Design 3 的设计规范。
