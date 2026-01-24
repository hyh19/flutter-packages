# SlotLayout 类详解

## 引言

`SlotLayout` 是 `flutter_adaptive_scaffold` 包中响应式布局系统的核心组件之一。它根据当前屏幕的断点条件，从预定义的配置映射中选择并显示合适的 Widget，同时提供流畅的动画过渡效果。`SlotLayout` 将断点（`Breakpoint`）与布局配置（`SlotLayoutConfig`）关联起来，实现了声明式的响应式布局方案，让开发者能够轻松创建适配不同屏幕尺寸的应用界面。

## 类概述

### 设计目的

`SlotLayout` 类的主要目的是：

1. **响应式布局选择**：根据当前屏幕条件（宽度、高度、平台等）自动选择最合适的布局配置
2. **平滑过渡动画**：在布局切换时提供流畅的动画效果，提升用户体验
3. **声明式配置**：通过映射表的方式声明不同断点下的布局，代码清晰易维护
4. **与 AdaptiveLayout 集成**：作为 `AdaptiveLayout` 的基础组件，为不同的槽位（slot）提供布局能力

### 在响应式布局中的作用

`SlotLayout` 在响应式布局系统中起到承上启下的作用：

- **接收断点信息**：通过 `Breakpoint.isActive()` 方法检测当前屏幕匹配的断点
- **选择布局配置**：从 `config` 映射表中选择对应的 `SlotLayoutConfig`
- **渲染 Widget**：使用 `SlotLayoutConfig.builder` 构建并显示 Widget
- **处理动画**：在布局切换时执行进入和退出动画

### 与 Breakpoint 的关系

`SlotLayout` 与 `Breakpoint` 紧密配合：

- **配置键**：`SlotLayout.config` 使用 `Breakpoint` 作为键
- **断点检测**：通过 `Breakpoint.activeBreakpointIn()` 查找激活的断点
- **优先级处理**：当多个断点同时激活时，按照映射表中的顺序，后面的断点配置会覆盖前面的

**示例**：

```dart
SlotLayout(
  config: <Breakpoint, SlotLayoutConfig>{
    Breakpoints.small: SlotLayoutConfig(...),      // 小屏幕配置
    Breakpoints.medium: SlotLayoutConfig(...),     // 中等屏幕配置
    Breakpoints.large: SlotLayoutConfig(...),      // 大屏幕配置
  },
)
```

### 与 AdaptiveLayout 的关系

`SlotLayout` 是 `AdaptiveLayout` 的基础组件：

- **槽位布局**：`AdaptiveLayout` 使用多个 `SlotLayout` 来管理不同的槽位（如 body、navigationRail、endDrawer 等）
- **统一配置**：每个槽位都有独立的 `SlotLayout`，可以独立配置不同断点下的布局
- **协调工作**：多个 `SlotLayout` 协同工作，共同构建完整的响应式界面

**示例**：

```dart
AdaptiveLayout(
  body: SlotLayout(...),              // body 槽位的布局
  navigationRail: SlotLayout(...),    // navigationRail 槽位的布局
  endDrawer: SlotLayout(...),         // endDrawer 槽位的布局
)
```

### StatefulWidget 设计

`SlotLayout` 继承自 `StatefulWidget`，这是因为：

- **状态管理**：需要在 `build` 方法中动态选择配置，需要状态来跟踪当前选择的 Widget
- **动画支持**：内部使用 `AnimatedSwitcher`，需要 `SingleTickerProviderStateMixin` 来提供动画控制器
- **响应式更新**：当屏幕尺寸变化时，需要重新构建并选择新的配置

## 核心组件详解

### SlotLayout 类

#### 构造函数

```dart
const SlotLayout({required this.config, super.key});
```

**参数**：

- `config`（必需）：`Map<Breakpoint, SlotLayoutConfig?>` 类型，将断点映射到布局配置
- `key`（可选）：Widget 的键，用于 Widget 树中的识别

**特点**：

- 使用 `const` 构造函数，支持编译时常量
- `config` 中的值可以为 `null`，用于覆盖更宽范围的断点配置

**示例**：

```dart
SlotLayout(
  key: const Key('body_slot'),
  config: <Breakpoint, SlotLayoutConfig>{
    Breakpoints.small: SlotLayoutConfig(...),
    Breakpoints.medium: SlotLayoutConfig(...),
    Breakpoints.standard: null,  // 覆盖标准断点的配置
  },
)
```

#### config 属性

```dart
final Map<Breakpoint, SlotLayoutConfig?> config;
```

**作用**：定义不同断点下的布局配置映射表。

**特点**：

1. **可空值支持**：`SlotLayoutConfig?` 可以为 `null`，用于显式覆盖更宽范围断点的配置
2. **优先级规则**：`SlotLayout` 选择最后一个激活的断点对应的配置
3. **非互斥性**：多个断点可以同时激活，后面的配置会覆盖前面的

**选择逻辑**：

```dart
// SlotLayout 内部的选择逻辑（简化版）
SlotLayoutConfig? pickWidget(BuildContext context, Map<Breakpoint, SlotLayoutConfig?> config) {
  // 1. 从所有断点中找到激活的断点（最后一个匹配的）
  final Breakpoint? breakpoint = Breakpoint.activeBreakpointIn(
    context,
    config.keys.toList(),
  );
  
  // 2. 如果找到激活的断点且配置存在，返回配置
  return breakpoint != null && config.containsKey(breakpoint)
      ? config[breakpoint]
      : null;
}
```

**示例**：

```dart
SlotLayout(
  config: <Breakpoint, SlotLayoutConfig>{
    // 如果 small 和 smallAndUp 都激活，smallAndUp 的配置会生效
    Breakpoints.small: SlotLayoutConfig(...),           // 可能被覆盖
    Breakpoints.smallAndUp: SlotLayoutConfig(...),      // 优先级更高
    Breakpoints.standard: SlotLayoutConfig(...),        // 回退配置
  },
)
```

### SlotLayoutConfig 类

`SlotLayoutConfig` 是一个 `StatelessWidget`，用于封装单个断点下的布局配置和动画参数。

#### SlotLayoutConfig 构造函数

```dart
const SlotLayoutConfig._({
  super.key,
  required this.builder,
  this.inAnimation,
  this.outAnimation,
  this.inDuration,
  this.outDuration,
  this.inCurve,
  this.outCurve,
});
```

**注意**：这是一个私有构造函数（`_`），应该通过 `SlotLayout.from()` 工厂方法创建实例。

#### builder 属性

```dart
final WidgetBuilder? builder;
```

**作用**：构建要显示的 Widget。

**类型**：`WidgetBuilder` 是 `typedef WidgetBuilder = Widget Function(BuildContext context);`

**使用场景**：

- 返回该断点下应该显示的 Widget
- 可以为 `null`，此时会显示空 Widget（`SizedBox.shrink()`）

**示例**：

```dart
SlotLayoutConfig(
  builder: (context) => Column(
    children: [
      Text('小屏幕布局'),
      // ... 其他 Widget
    ],
  ),
)
```

#### 动画属性

`SlotLayoutConfig` 提供了丰富的动画配置选项：

##### inAnimation

```dart
final Widget Function(Widget, Animation<double>)? inAnimation;
```

**作用**：定义 Widget 进入时的动画效果。

**参数**：

- `Widget`：要应用动画的子 Widget
- `Animation<double>`：动画对象，值从 0.0 到 1.0

**返回值**：应用了动画效果的 Widget

**示例**：

```dart
SlotLayoutConfig(
  inAnimation: (child, animation) => FadeTransition(
    opacity: animation,
    child: child,
  ),
)
```

##### outAnimation

```dart
final Widget Function(Widget, Animation<double>)? outAnimation;
```

**作用**：定义 Widget 退出时的动画效果。

**注意**：在 `SlotLayout` 内部，退出动画会使用 `ReverseAnimation(animation)`，所以动画函数应该处理从 1.0 到 0.0 的动画。

**示例**：

```dart
SlotLayoutConfig(
  outAnimation: (child, animation) => SlideTransition(
    position: Tween<Offset>(
      begin: Offset.zero,
      end: const Offset(-1.0, 0.0),
    ).animate(animation),
    child: child,
  ),
)
```

##### inDuration / outDuration

```dart
final Duration? inDuration;   // 进入动画时长
final Duration? outDuration;   // 退出动画时长
```

**作用**：控制动画的持续时间。

**默认值**：

- `inDuration`：如果为 `null`，`AnimatedSwitcher` 使用默认的 1000 毫秒
- `outDuration`：如果为 `null`，使用 `inDuration` 的值

**示例**：

```dart
SlotLayoutConfig(
  inDuration: const Duration(milliseconds: 500),
  outDuration: const Duration(milliseconds: 300),
)
```

##### inCurve / outCurve

```dart
final Curve? inCurve;   // 进入动画曲线
final Curve? outCurve;  // 退出动画曲线
```

**作用**：控制动画的缓动曲线。

**默认值**：如果为 `null`，使用 `Curves.linear`

**常用曲线**：

- `Curves.easeInOut`：先加速后减速
- `Curves.fastOutSlowIn`：快速开始，缓慢结束
- `Curves.bounceOut`：弹跳效果

**示例**：

```dart
SlotLayoutConfig(
  inCurve: Curves.easeInOut,
  outCurve: Curves.fastOutSlowIn,
)
```

#### empty() 静态方法

```dart
static SlotLayoutConfig empty() {
  return const SlotLayoutConfig._(key: Key(''), builder: null);
}
```

**作用**：创建一个空的 `SlotLayoutConfig`，用于表示该槽位不应该显示任何内容。

**使用场景**：

- 在某些断点下隐藏某个槽位
- 作为 `SlotLayout` 的回退配置（当没有匹配的断点时）

**示例**：

```dart
SlotLayout(
  config: <Breakpoint, SlotLayoutConfig>{
    Breakpoints.small: SlotLayoutConfig.empty(),  // 小屏幕下隐藏
    Breakpoints.medium: SlotLayoutConfig(...),  // 中等屏幕显示
  },
)
```

#### build() 方法

```dart
@override
Widget build(BuildContext context) {
  return (builder != null) ? builder!(context) : const SizedBox.shrink();
}
```

**作用**：构建 Widget，如果 `builder` 为 `null`，返回空的 `SizedBox.shrink()`。

## 静态方法详解

### pickWidget() 方法

```dart
static SlotLayoutConfig? pickWidget(
    BuildContext context, Map<Breakpoint, SlotLayoutConfig?> config)
```

**作用**：根据当前屏幕条件，从配置映射表中选择合适的 `SlotLayoutConfig`。

**参数**：

- `context`：构建上下文，用于获取屏幕信息和查找断点
- `config`：断点到配置的映射表

**返回值**：匹配的 `SlotLayoutConfig`，如果没有匹配的断点则返回 `null`

**实现逻辑**：

```dart
static SlotLayoutConfig? pickWidget(
    BuildContext context, Map<Breakpoint, SlotLayoutConfig?> config) {
  // 1. 从配置的键（断点列表）中找到激活的断点
  final Breakpoint? breakpoint =
      Breakpoint.activeBreakpointIn(context, config.keys.toList());
  
  // 2. 如果找到激活的断点且配置存在，返回配置
  return breakpoint != null && config.containsKey(breakpoint)
      ? config[breakpoint]
      : null;
}
```

**关键点**：

1. **使用 `activeBreakpointIn()`**：这个方法会返回最后一个匹配的断点（考虑优先级）
2. **检查配置存在**：即使断点激活，也要检查配置映射表中是否有对应的值
3. **返回 null**：如果没有匹配的断点或配置，返回 `null`，`SlotLayout` 会使用 `SlotLayoutConfig.empty()`

**使用场景**：

- 在 `SlotLayout` 内部使用（`_SlotLayoutState.build()` 中调用）
- 可以独立使用，用于在其他地方预先判断应该使用哪个配置

**示例**：

```dart
// 在 SlotLayout 外部使用
final config = <Breakpoint, SlotLayoutConfig>{
  Breakpoints.small: SlotLayoutConfig(...),
  Breakpoints.medium: SlotLayoutConfig(...),
};

final chosenConfig = SlotLayout.pickWidget(context, config);
if (chosenConfig != null) {
  // 使用选中的配置
}
```

### from() 工厂方法

```dart
static SlotLayoutConfig from({
  WidgetBuilder? builder,
  Widget Function(Widget, Animation<double>)? inAnimation,
  Widget Function(Widget, Animation<double>)? outAnimation,
  Duration? inDuration,
  Duration? outDuration,
  Curve? inCurve,
  Curve? outCurve,
  required Key key,
})
```

**作用**：创建 `SlotLayoutConfig` 实例的便捷工厂方法。

**参数**：

- `builder`（可选）：Widget 构建器
- `inAnimation`（可选）：进入动画函数
- `outAnimation`（可选）：退出动画函数
- `inDuration`（可选）：进入动画时长
- `outDuration`（可选）：退出动画时长
- `inCurve`（可选）：进入动画曲线
- `outCurve`（可选）：退出动画曲线
- `key`（必需）：Widget 的键

**返回值**：`SlotLayoutConfig` 实例

**为什么需要这个方法**：

- `SlotLayoutConfig` 的构造函数是私有的（`_`），无法直接创建
- 提供更友好的 API，参数名称清晰
- 统一创建方式，便于维护

**示例**：

```dart
SlotLayoutConfig.from(
  key: const Key('body_small'),
  builder: (context) => MobileBody(),
  inAnimation: (child, animation) => FadeTransition(
    opacity: animation,
    child: child,
  ),
  inDuration: const Duration(milliseconds: 300),
  inCurve: Curves.easeInOut,
)
```

**Key 的重要性**：

`key` 是必需参数，因为：

1. **Widget 识别**：`AnimatedSwitcher` 使用 key 来识别 Widget 的变化
2. **动画触发**：当 key 改变时，`AnimatedSwitcher` 会触发切换动画
3. **状态保持**：正确的 key 可以帮助 Flutter 保持 Widget 的状态

**最佳实践**：

```dart
// ✅ 推荐：使用描述性的、唯一的 Key
SlotLayoutConfig.from(
  key: const Key('body_small_screen'),
  builder: (context) => MobileBody(),
)

// ❌ 避免：使用相同的 Key 或空 Key
SlotLayoutConfig.from(
  key: const Key('body'),  // 多个配置使用相同 Key 会导致问题
  builder: (context) => MobileBody(),
)
```

## 内部实现详解

### _SlotLayoutState 类

`_SlotLayoutState` 是 `SlotLayout` 的状态类，负责实际的布局选择和动画处理。

#### 类定义

```dart
class _SlotLayoutState extends State<SlotLayout>
    with SingleTickerProviderStateMixin {
  SlotLayoutConfig? chosenWidget;
  // ...
}
```

**特点**：

- 混入 `SingleTickerProviderStateMixin`：为 `AnimatedSwitcher` 提供动画控制器
- `chosenWidget`：存储当前选择的配置

#### build() 方法详解

`build()` 方法是 `SlotLayout` 的核心，负责选择配置并构建动画 Widget。

```dart
@override
Widget build(BuildContext context) {
  // 1. 选择匹配的配置
  chosenWidget = SlotLayout.pickWidget(context, widget.config);
  
  // 2. 标记是否有退出动画
  bool hasAnimation = false;
  
  // 3. 使用 AnimatedSwitcher 实现切换动画
  return AnimatedSwitcher(
    // 动画时长配置
    duration: chosenWidget?.inDuration ?? const Duration(milliseconds: 1000),
    reverseDuration: chosenWidget?.outDuration,
    switchInCurve: chosenWidget?.inCurve ?? Curves.linear,
    switchOutCurve: chosenWidget?.outCurve ?? Curves.linear,
    
    // 布局构建器：处理多个 Widget 的堆叠
    layoutBuilder: (Widget? currentChild, List<Widget> previousChildren) {
      final Stack elements = Stack(
        children: <Widget>[
          // 如果有退出动画且存在之前的子 Widget，显示之前的子 Widget
          if (hasAnimation && previousChildren.isNotEmpty)
            previousChildren.first,
          // 显示当前的子 Widget
          if (currentChild != null) currentChild,
        ],
      );
      return elements;
    },
    
    // 过渡构建器：应用进入/退出动画
    transitionBuilder: (Widget child, Animation<double> animation) {
      final SlotLayoutConfig configChild = child as SlotLayoutConfig;
      
      // 判断是进入还是退出
      if (child.key == chosenWidget?.key) {
        // 进入动画：当前选择的 Widget
        return (configChild.inAnimation != null)
            ? configChild.inAnimation!(child, animation)
            : child;
      } else {
        // 退出动画：之前的 Widget
        if (configChild.outAnimation != null) {
          hasAnimation = true;  // 标记有退出动画
        }
        return (configChild.outAnimation != null)
            ? configChild.outAnimation!(child, ReverseAnimation(animation))
            : child;
      }
    },
    
    // 子 Widget：当前选择的配置或空配置
    child: chosenWidget ?? SlotLayoutConfig.empty(),
  );
}
```

#### 执行流程

1. **选择配置**：调用 `SlotLayout.pickWidget()` 选择匹配的配置
2. **构建 AnimatedSwitcher**：使用选中的配置创建 `AnimatedSwitcher`
3. **处理布局**：`layoutBuilder` 负责将当前和之前的 Widget 堆叠在一起
4. **应用动画**：`transitionBuilder` 根据 Widget 的 key 判断是进入还是退出，应用相应的动画
5. **显示结果**：最终显示应用了动画的 Widget

#### 动画机制详解

**进入动画**：

- 当 `child.key == chosenWidget?.key` 时，说明这是新选择的 Widget
- 如果配置了 `inAnimation`，应用进入动画
- 动画值从 0.0 到 1.0

**退出动画**：

- 当 `child.key != chosenWidget?.key` 时，说明这是之前的 Widget
- 如果配置了 `outAnimation`，应用退出动画
- 使用 `ReverseAnimation(animation)`，动画值从 1.0 到 0.0
- 设置 `hasAnimation = true`，确保在 `layoutBuilder` 中显示之前的 Widget

**布局堆叠**：

- `layoutBuilder` 使用 `Stack` 将当前和之前的 Widget 堆叠
- 只有在有退出动画时才显示之前的 Widget
- 这样可以实现两个 Widget 同时显示并执行动画的效果

## 动画系统

### AnimatedSwitcher 集成

`SlotLayout` 内部使用 `AnimatedSwitcher` 来实现布局切换的动画效果。

**为什么使用 AnimatedSwitcher**：

- **自动切换**：当子 Widget 的 key 改变时，自动触发切换动画
- **灵活配置**：支持自定义过渡效果和时长
- **性能优化**：Flutter 框架优化了动画性能

### 动画参数传递

`SlotLayout` 将 `SlotLayoutConfig` 中的动画参数传递给 `AnimatedSwitcher`：

```dart
AnimatedSwitcher(
  duration: chosenWidget?.inDuration ?? const Duration(milliseconds: 1000),
  reverseDuration: chosenWidget?.outDuration,
  switchInCurve: chosenWidget?.inCurve ?? Curves.linear,
  switchOutCurve: chosenWidget?.outCurve ?? Curves.linear,
  // ...
)
```

**默认值**：

- `duration`：如果未指定，使用 1000 毫秒
- `reverseDuration`：如果未指定，使用 `duration` 的值
- `switchInCurve` / `switchOutCurve`：如果未指定，使用 `Curves.linear`

### 常见动画效果

#### 淡入淡出（Fade）

```dart
SlotLayoutConfig.from(
  key: const Key('fade_example'),
  builder: (context) => YourWidget(),
  inAnimation: (child, animation) => FadeTransition(
    opacity: animation,
    child: child,
  ),
  outAnimation: (child, animation) => FadeTransition(
    opacity: animation,
    child: child,
  ),
)
```

#### 滑动（Slide）

```dart
SlotLayoutConfig.from(
  key: const Key('slide_example'),
  builder: (context) => YourWidget(),
  inAnimation: (child, animation) => SlideTransition(
    position: Tween<Offset>(
      begin: const Offset(1.0, 0.0),  // 从右侧滑入
      end: Offset.zero,
    ).animate(animation),
    child: child,
  ),
  outAnimation: (child, animation) => SlideTransition(
    position: Tween<Offset>(
      begin: Offset.zero,
      end: const Offset(-1.0, 0.0),  // 向左侧滑出
    ).animate(animation),
    child: child,
  ),
)
```

#### 缩放（Scale）

```dart
SlotLayoutConfig.from(
  key: const Key('scale_example'),
  builder: (context) => YourWidget(),
  inAnimation: (child, animation) => ScaleTransition(
    scale: animation,
    child: child,
  ),
  outAnimation: (child, animation) => ScaleTransition(
    scale: animation,
    child: child,
  ),
)
```

#### 组合动画

```dart
SlotLayoutConfig.from(
  key: const Key('combined_example'),
  builder: (context) => YourWidget(),
  inAnimation: (child, animation) {
    return FadeTransition(
      opacity: animation,
      child: ScaleTransition(
        scale: animation,
        child: child,
      ),
    );
  },
  outAnimation: (child, animation) {
    return FadeTransition(
      opacity: animation,
      child: SlideTransition(
        position: Tween<Offset>(
          begin: Offset.zero,
          end: const Offset(0.0, 1.0),
        ).animate(animation),
        child: child,
      ),
    );
  },
)
```

## 使用示例

### 示例 1：基础使用

最简单的 `SlotLayout` 使用方式：

```dart
SlotLayout(
  config: <Breakpoint, SlotLayoutConfig>{
    Breakpoints.small: SlotLayoutConfig.from(
      key: const Key('body_small'),
      builder: (context) => MobileBody(),
    ),
    Breakpoints.medium: SlotLayoutConfig.from(
      key: const Key('body_medium'),
      builder: (context) => TabletBody(),
    ),
    Breakpoints.large: SlotLayoutConfig.from(
      key: const Key('body_large'),
      builder: (context) => DesktopBody(),
    ),
    Breakpoints.standard: SlotLayoutConfig.from(
      key: const Key('body_standard'),
      builder: (context) => DefaultBody(),
    ),
  },
)
```

### 示例 2：自定义动画

使用自定义的进入和退出动画：

```dart
SlotLayout(
  config: <Breakpoint, SlotLayoutConfig>{
    Breakpoints.small: SlotLayoutConfig.from(
      key: const Key('animated_small'),
      builder: (context) => MobileBody(),
      inAnimation: (child, animation) => FadeTransition(
        opacity: CurvedAnimation(
          parent: animation,
          curve: Curves.easeInOut,
        ),
        child: child,
      ),
      inDuration: const Duration(milliseconds: 500),
      inCurve: Curves.easeInOut,
    ),
    Breakpoints.medium: SlotLayoutConfig.from(
      key: const Key('animated_medium'),
      builder: (context) => TabletBody(),
      inAnimation: (child, animation) => SlideTransition(
        position: Tween<Offset>(
          begin: const Offset(1.0, 0.0),
          end: Offset.zero,
        ).animate(CurvedAnimation(
          parent: animation,
          curve: Curves.fastOutSlowIn,
        )),
        child: child,
      ),
      outAnimation: (child, animation) => FadeTransition(
        opacity: animation,
        child: child,
      ),
      inDuration: const Duration(milliseconds: 300),
      outDuration: const Duration(milliseconds: 200),
    ),
  },
)
```

### 示例 3：使用 andUp 版本简化配置

使用 `andUp` 版本的断点简化配置：

```dart
SlotLayout(
  config: <Breakpoint, SlotLayoutConfig>{
    // 小屏幕：单面板布局
    Breakpoints.small: SlotLayoutConfig.from(
      key: const Key('single_pane'),
      builder: (context) => SinglePaneLayout(),
    ),
    // 中等屏幕及以上：双面板布局
    Breakpoints.mediumAndUp: SlotLayoutConfig.from(
      key: const Key('multi_pane'),
      builder: (context) => MultiPaneLayout(),
    ),
  },
)
```

### 示例 4：平台特定布局

为不同平台提供不同的布局：

```dart
SlotLayout(
  config: <Breakpoint, SlotLayoutConfig>{
    // 桌面平台小屏幕
    Breakpoints.smallDesktop: SlotLayoutConfig.from(
      key: const Key('desktop_small'),
      builder: (context) => CompactDesktopLayout(),
    ),
    // 移动平台小屏幕
    Breakpoints.smallMobile: SlotLayoutConfig.from(
      key: const Key('mobile_small'),
      builder: (context) => MobileLayout(),
    ),
    // 通用小屏幕（回退）
    Breakpoints.small: SlotLayoutConfig.from(
      key: const Key('small_fallback'),
      builder: (context) => DefaultSmallLayout(),
    ),
  },
)
```

### 示例 5：隐藏某些断点的内容

使用 `null` 或 `empty()` 来隐藏某些断点下的内容：

```dart
SlotLayout(
  config: <Breakpoint, SlotLayoutConfig>{
    // 小屏幕：显示侧边栏
    Breakpoints.small: SlotLayoutConfig.from(
      key: const Key('sidebar_small'),
      builder: (context) => DrawerSidebar(),
    ),
    // 中等屏幕及以上：隐藏侧边栏（使用 null 覆盖）
    Breakpoints.mediumAndUp: null,  // 显式设置为 null
    // 或者使用 empty()
    // Breakpoints.mediumAndUp: SlotLayoutConfig.empty(),
  },
)
```

### 示例 6：与 AdaptiveScaffold 集成

在 `AdaptiveScaffold` 中使用 `SlotLayout`：

```dart
AdaptiveScaffold(
  body: SlotLayout(
    config: <Breakpoint, SlotLayoutConfig>{
      Breakpoints.small: SlotLayoutConfig.from(
        key: const Key('body_small'),
        builder: (context) => MobileBody(),
      ),
      Breakpoints.mediumAndUp: SlotLayoutConfig.from(
        key: const Key('body_large'),
        builder: (context) => DesktopBody(),
      ),
    },
  ),
  navigationRail: SlotLayout(
    config: <Breakpoint, SlotLayoutConfig>{
      Breakpoints.small: SlotLayoutConfig.empty(),  // 小屏幕隐藏导航栏
      Breakpoints.mediumAndUp: SlotLayoutConfig.from(
        key: const Key('nav_rail'),
        builder: (context) => NavigationRail(),
      ),
    },
  ),
)
```

### 示例 7：条件渲染复杂布局

根据断点渲染不同的复杂布局：

```dart
class AdaptiveContent extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return SlotLayout(
      config: <Breakpoint, SlotLayoutConfig>{
        Breakpoints.small: SlotLayoutConfig.from(
          key: const Key('content_small'),
          builder: (context) => _buildMobileLayout(context),
        ),
        Breakpoints.medium: SlotLayoutConfig.from(
          key: const Key('content_medium'),
          builder: (context) => _buildTabletLayout(context),
        ),
        Breakpoints.large: SlotLayoutConfig.from(
          key: const Key('content_large'),
          builder: (context) => _buildDesktopLayout(context),
        ),
      },
    );
  }

  Widget _buildMobileLayout(BuildContext context) {
    return Column(
      children: [
        AppBar(title: Text('Mobile')),
        Expanded(child: ContentList()),
      ],
    );
  }

  Widget _buildTabletLayout(BuildContext context) {
    return Row(
      children: [
        Expanded(flex: 1, child: Sidebar()),
        Expanded(flex: 2, child: ContentList()),
      ],
    );
  }

  Widget _buildDesktopLayout(BuildContext context) {
    return Row(
      children: [
        Expanded(flex: 1, child: Sidebar()),
        Expanded(flex: 3, child: ContentList()),
        Expanded(flex: 1, child: DetailsPanel()),
      ],
    );
  }
}
```

## 最佳实践

### 1. 使用唯一且描述性的 Key

**建议**：为每个 `SlotLayoutConfig` 使用唯一且描述性的 key。

**原因**：

- `AnimatedSwitcher` 使用 key 来识别 Widget 的变化
- 相同的 key 会导致动画不触发
- 描述性的 key 有助于调试和维护

**示例**：

```dart
// ✅ 推荐：使用描述性的、唯一的 Key
SlotLayoutConfig.from(
  key: const Key('body_small_screen'),
  builder: (context) => MobileBody(),
)

SlotLayoutConfig.from(
  key: const Key('body_medium_screen'),
  builder: (context) => TabletBody(),
)

// ❌ 避免：使用相同的 Key
SlotLayoutConfig.from(
  key: const Key('body'),  // 多个配置使用相同 Key
  builder: (context) => MobileBody(),
)
SlotLayoutConfig.from(
  key: const Key('body'),  // 相同 Key 会导致问题
  builder: (context) => TabletBody(),
)
```

### 2. 总是包含回退配置

**建议**：在 `config` 映射表中总是包含 `Breakpoints.standard` 作为回退配置。

**原因**：

- 确保在任何情况下都有一个激活的断点
- 处理边缘情况（如非常规屏幕尺寸）
- 提供默认的布局配置

**示例**：

```dart
SlotLayout(
  config: <Breakpoint, SlotLayoutConfig>{
    Breakpoints.small: SlotLayoutConfig.from(...),
    Breakpoints.medium: SlotLayoutConfig.from(...),
    Breakpoints.standard: SlotLayoutConfig.from(...),  // ✅ 总是包含回退
  },
)
```

### 3. 合理使用 andUp 版本

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

### 4. 优化动画性能

**建议**：合理配置动画参数，避免过度复杂的动画。

**性能考虑**：

- 使用简单的动画效果（如 Fade、Slide）而不是复杂的组合动画
- 设置合理的动画时长（通常 200-500 毫秒）
- 避免在动画中使用 `GlobalKey`，可能导致性能问题

**示例**：

```dart
// ✅ 推荐：简单高效的动画
SlotLayoutConfig.from(
  key: const Key('simple_animation'),
  builder: (context) => YourWidget(),
  inAnimation: (child, animation) => FadeTransition(
    opacity: animation,
    child: child,
  ),
  inDuration: const Duration(milliseconds: 300),  // 合理的时长
)

// ⚠️ 谨慎使用：复杂的组合动画可能影响性能
SlotLayoutConfig.from(
  key: const Key('complex_animation'),
  builder: (context) => YourWidget(),
  inAnimation: (child, animation) {
    // 多个嵌套的动画可能影响性能
    return FadeTransition(
      opacity: animation,
      child: ScaleTransition(
        scale: animation,
        child: RotationTransition(
          turns: animation,
          child: child,
        ),
      ),
    );
  },
)
```

### 5. 组织配置代码

**建议**：将复杂的配置提取到单独的方法或类中，提高代码可读性。

**示例**：

```dart
class BodySlotLayout extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return SlotLayout(
      config: _buildBodyConfig(),
    );
  }

  Map<Breakpoint, SlotLayoutConfig> _buildBodyConfig() {
    return <Breakpoint, SlotLayoutConfig>{
      Breakpoints.small: _buildSmallConfig(),
      Breakpoints.medium: _buildMediumConfig(),
      Breakpoints.large: _buildLargeConfig(),
      Breakpoints.standard: _buildStandardConfig(),
    };
  }

  SlotLayoutConfig _buildSmallConfig() {
    return SlotLayoutConfig.from(
      key: const Key('body_small'),
      builder: (context) => MobileBody(),
      inAnimation: (child, animation) => FadeTransition(
        opacity: animation,
        child: child,
      ),
    );
  }

  // ... 其他配置方法
}
```

### 6. 测试不同断点

**建议**：在开发过程中测试所有相关的断点，确保布局在不同屏幕尺寸下都能正常工作。

**测试方法**：

- 使用 Flutter 的设备预览功能
- 调整窗口大小（桌面应用）
- 使用不同尺寸的模拟器
- 测试横屏和竖屏方向

### 7. 避免在动画中使用 GlobalKey

**建议**：避免在 `SlotLayoutConfig.builder` 返回的 Widget 中使用 `GlobalKey`。

**原因**：

- `AnimatedSwitcher` 在动画过程中可能同时显示多个 Widget
- `GlobalKey` 在多个 Widget 实例中可能导致状态冲突
- 文档中也明确提到："If you are using GlobalKeys, this may cause issues with the AnimatedSwitcher."

**示例**：

```dart
// ⚠️ 避免：在 builder 中使用 GlobalKey
SlotLayoutConfig.from(
  key: const Key('with_global_key'),
  builder: (context) => MyWidget(
    key: GlobalKey(),  // 可能导致问题
  ),
)

// ✅ 推荐：使用普通的 Key 或 ValueKey
SlotLayoutConfig.from(
  key: const Key('with_local_key'),
  builder: (context) => MyWidget(
    key: ValueKey('unique_id'),  // 使用局部 Key
  ),
)
```

### 8. 利用断点的 Material Design 3 属性

**建议**：在 `builder` 中使用断点的 Material Design 3 属性（`spacing`、`margin`、`padding`）来保持设计一致性。

**示例**：

```dart
SlotLayoutConfig.from(
  key: const Key('adaptive_spacing'),
  builder: (context) {
    final breakpoint = Breakpoint.activeBreakpointOf(context);
    return Padding(
      padding: EdgeInsets.all(breakpoint.margin),
      child: Column(
        children: [
          Widget1(),
          SizedBox(height: breakpoint.spacing),
          Widget2(),
          Padding(
            padding: EdgeInsets.all(breakpoint.padding),
            child: Widget3(),
          ),
        ],
      ),
    );
  },
)
```

## 总结

`SlotLayout` 是 `flutter_adaptive_scaffold` 包中响应式布局系统的核心组件，它通过将断点与布局配置关联，实现了声明式的响应式布局方案。

### 关键要点

1. **响应式选择**：根据当前屏幕条件自动选择最合适的布局配置
2. **动画支持**：内置 `AnimatedSwitcher`，支持自定义进入和退出动画
3. **优先级机制**：多个断点同时激活时，后面的配置会覆盖前面的
4. **灵活配置**：支持可空的配置值，可以显式覆盖更宽范围的断点
5. **与 AdaptiveLayout 集成**：作为 `AdaptiveLayout` 的基础组件，为不同槽位提供布局能力

### 核心 API

- `SlotLayout(config: Map<Breakpoint, SlotLayoutConfig?>)`：创建 `SlotLayout` 实例
- `SlotLayout.pickWidget(context, config)`：静态方法，选择匹配的配置
- `SlotLayout.from(...)`：工厂方法，创建 `SlotLayoutConfig` 实例
- `SlotLayoutConfig.empty()`：创建空的配置，用于隐藏内容

### 使用建议

1. 使用唯一且描述性的 key，确保动画正常工作
2. 总是包含 `Breakpoints.standard` 作为回退配置
3. 合理使用 `andUp` 版本简化配置
4. 优化动画性能，避免过度复杂的动画
5. 组织配置代码，提高可读性和可维护性
6. 测试不同断点，确保布局在各种屏幕尺寸下正常工作
7. 避免在动画中使用 `GlobalKey`
8. 利用断点的 Material Design 3 属性保持设计一致性

通过合理使用 `SlotLayout`，开发者可以轻松创建响应式、自适应的 Flutter 应用，提供优秀的用户体验，同时确保符合 Material Design 3 的设计规范。
