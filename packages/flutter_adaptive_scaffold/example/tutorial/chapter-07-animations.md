# 第 7 章 动画与过渡效果

## 引言

流畅的动画过渡是响应式布局体验的关键。当屏幕尺寸变化或布局切换时，合适的动画能让用户感受到应用的连贯性和专业性。本章将深入讲解 `flutter_adaptive_scaffold` 中的动画系统，包括内置动画、自定义动画的创建，以及动画性能优化。

## 动画系统架构

### AnimatedSwitcher 基础

`SlotLayout` 内部使用 `AnimatedSwitcher` 实现动画过渡。`AnimatedSwitcher` 的工作原理是：

1. **检测子 Widget 变化**：通过 `Key` 识别 Widget 的变化
2. **执行退出动画**：旧 Widget 执行 `outAnimation`
3. **执行进入动画**：新 Widget 执行 `inAnimation`
4. **移除旧 Widget**：动画完成后移除

### 动画函数签名

动画函数的标准签名是：

```dart
Widget Function(Widget child, Animation<double> animation)
```

- **child**：要应用动画的 Widget
- **animation**：动画对象，值从 0.0 到 1.0
- **返回值**：应用了动画的 Widget

## 内置动画详解

### leftOutIn（从左侧滑入/滑出）

**用途**：通常用于 `primaryNavigation` 的显示和隐藏

**实现**：

```dart
static AnimatedWidget leftOutIn(
  Widget child,
  Animation<double> animation,
) {
  return SlideTransition(
    position: Tween<Offset>(
      begin: const Offset(-1.0, 0.0),  // 从左侧开始
      end: Offset.zero,                // 到正常位置
    ).animate(CurvedAnimation(
      parent: animation,
      curve: Curves.easeInOutCubic,
    )),
    child: child,
  );
}
```

**使用示例**：

```dart
primaryNavigation: SlotLayout(
  config: {
    Breakpoints.medium: SlotLayout.from(
      key: const Key('Nav'),
      inAnimation: AdaptiveScaffold.leftOutIn,
      builder: (_) => NavigationRail(...),
    ),
  },
)
```

**视觉效果**：导航栏从左侧滑入，隐藏时向左侧滑出

### rightOutIn（从右侧滑入/滑出）

**用途**：通常用于 `secondaryNavigation` 或 `secondaryBody`

**实现**：

```dart
static AnimatedWidget rightOutIn(
  Widget child,
  Animation<double> animation,
) {
  return SlideTransition(
    position: Tween<Offset>(
      begin: const Offset(1.0, 0.0),   // 从右侧开始
      end: Offset.zero,                // 到正常位置
    ).animate(CurvedAnimation(
      parent: animation,
      curve: Curves.easeInOutCubic,
    )),
    child: child,
  );
}
```

**使用示例**：

```dart
secondaryBody: SlotLayout(
  config: {
    Breakpoints.mediumAndUp: SlotLayout.from(
      key: const Key('Detail'),
      inAnimation: AdaptiveScaffold.rightOutIn,
      builder: (_) => DetailView(...),
    ),
  },
)
```

**视觉效果**：详情视图从右侧滑入，隐藏时向右侧滑出

### topToBottom（从顶部滑入/滑出）

**用途**：通常用于 `topNavigation` 或内容区域的切换

**实现**：

```dart
static AnimatedWidget topToBottom(
  Widget child,
  Animation<double> animation,
) {
  return SlideTransition(
    position: Tween<Offset>(
      begin: const Offset(0.0, -1.0),  // 从顶部开始
      end: Offset.zero,                 // 到正常位置
    ).animate(CurvedAnimation(
      parent: animation,
      curve: Curves.easeInOutCubic,
    )),
    child: child,
  );
}
```

**使用示例**：

```dart
body: SlotLayout(
  config: {
    Breakpoints.standard: SlotLayout.from(
      key: const Key('Body'),
      inAnimation: AdaptiveScaffold.topToBottom,
      builder: (_) => ContentView(...),
    ),
  },
)
```

**视觉效果**：内容从顶部滑入，隐藏时向顶部滑出

### bottomToTop（从底部滑入/滑出）

**用途**：通常用于 `bottomNavigation` 的显示和隐藏

**实现**：

```dart
static AnimatedWidget bottomToTop(
  Widget child,
  Animation<double> animation,
) {
  return SlideTransition(
    position: Tween<Offset>(
      begin: const Offset(0.0, 1.0),   // 从底部开始
      end: Offset.zero,                 // 到正常位置
    ).animate(CurvedAnimation(
      parent: animation,
      curve: Curves.easeInOutCubic,
    )),
    child: child,
  );
}
```

**使用示例**：

```dart
bottomNavigation: SlotLayout(
  config: {
    Breakpoints.small: SlotLayout.from(
      key: const Key('Bottom Nav'),
      inAnimation: AdaptiveScaffold.bottomToTop,
      outAnimation: AdaptiveScaffold.topToBottom,
      builder: (_) => BottomNavigationBar(...),
    ),
  },
)
```

**视觉效果**：底部导航栏从底部滑入，隐藏时向底部滑出

### stayOnScreen（保持在屏幕上）

**用途**：当 Widget 需要保持在屏幕上，不执行退出动画时使用

**实现**：

```dart
static Widget stayOnScreen(
  Widget child,
  Animation<double> animation,
) {
  return child;  // 直接返回，不应用动画
}
```

**使用场景**：

在 `main.dart`（邮件应用）中，当 `secondaryBody` 需要保持在屏幕上时：

```dart
secondaryBody: SlotLayout(
  config: {
    Breakpoints.mediumAndUp: SlotLayout.from(
      outAnimation: AdaptiveScaffold.stayOnScreen,  // 退出时保持显示
      key: const Key('Secondary Body'),
      builder: (_) => DetailTile(...),
    ),
  },
)
```

**视觉效果**：Widget 保持在原位置，不执行退出动画

## 自定义动画创建

### 基础自定义动画

创建一个简单的淡入淡出动画：

```dart
Widget fadeAnimation(Widget child, Animation<double> animation) {
  return FadeTransition(
    opacity: animation,
    child: child,
  );
}

// 使用
SlotLayout.from(
  key: const Key('Body'),
  inAnimation: fadeAnimation,
  builder: (_) => Widget(),
)
```

### 组合动画

组合多个动画效果：

```dart
Widget fadeSlideAnimation(Widget child, Animation<double> animation) {
  return FadeTransition(
    opacity: animation,
    child: SlideTransition(
      position: Tween<Offset>(
        begin: const Offset(0.0, 0.3),
        end: Offset.zero,
      ).animate(CurvedAnimation(
        parent: animation,
        curve: Curves.easeOut,
      )),
      child: child,
    ),
  );
}
```

### 缩放动画

创建缩放效果：

```dart
Widget scaleAnimation(Widget child, Animation<double> animation) {
  return ScaleTransition(
    scale: Tween<double>(
      begin: 0.8,
      end: 1.0,
    ).animate(CurvedAnimation(
      parent: animation,
      curve: Curves.easeOutBack,
    )),
    child: child,
  );
}
```

### 旋转动画

创建旋转效果（较少使用）：

```dart
Widget rotateAnimation(Widget child, Animation<double> animation) {
  return RotationTransition(
    turns: Tween<double>(
      begin: 0.0,
      end: 1.0,
    ).animate(CurvedAnimation(
      parent: animation,
      curve: Curves.easeInOut,
    )),
    child: child,
  );
}
```

### 复杂组合动画

结合淡入、滑动和缩放：

```dart
Widget complexAnimation(Widget child, Animation<double> animation) {
  final CurvedAnimation curvedAnimation = CurvedAnimation(
    parent: animation,
    curve: Curves.easeInOutCubic,
  );

  return FadeTransition(
    opacity: curvedAnimation,
    child: SlideTransition(
      position: Tween<Offset>(
        begin: const Offset(0.0, 0.2),
        end: Offset.zero,
      ).animate(curvedAnimation),
      child: ScaleTransition(
        scale: Tween<double>(
          begin: 0.95,
          end: 1.0,
        ).animate(curvedAnimation),
        child: child,
      ),
    ),
  );
}
```

## 动画时长和曲线

### 控制动画时长

可以为进入和退出动画设置不同的时长：

```dart
SlotLayout.from(
  key: const Key('Body'),
  inDuration: Duration(milliseconds: 500),   // 进入动画 500ms
  outDuration: Duration(milliseconds: 300), // 退出动画 300ms
  inAnimation: AdaptiveScaffold.leftOutIn,
  builder: (_) => Widget(),
)
```

**最佳实践**：

- **快速切换**：200-300 ms（适合频繁切换的内容）
- **标准切换**：300-500 ms（大多数场景）
- **平滑切换**：500-1000 ms（重要内容切换）

### 动画曲线

Flutter 提供了多种动画曲线：

```dart
Curves.linear          // 线性
Curves.easeIn          // 缓入
Curves.easeOut         // 缓出
Curves.easeInOut       // 缓入缓出
Curves.easeInOutCubic  // 三次缓入缓出（内置动画使用）
Curves.easeOutBack     // 缓出回弹
Curves.bounceIn        // 弹跳进入
```

**使用示例**：

```dart
SlotLayout.from(
  key: const Key('Body'),
  inCurve: Curves.easeOutBack,    // 进入时使用回弹效果
  outCurve: Curves.easeIn,       // 退出时使用缓入效果
  inAnimation: scaleAnimation,
  builder: (_) => Widget(),
)
```

## 实际应用示例

### 邮件应用中的动画

在 `main.dart`（邮件应用）中，使用了多种动画：

```dart
// 1. 主导航：从左侧滑入
primaryNavigation: SlotLayout(
  config: {
    Breakpoints.medium: SlotLayout.from(
      key: const Key('primaryNavigation'),
      builder: (_) => AdaptiveScaffold.standardNavigationRail(...),
      // 使用默认动画（leftOutIn）
    ),
  },
),

// 2. 底部导航：从底部滑入，向顶部滑出
bottomNavigation: SlotLayout(
  config: {
    Breakpoints.small: SlotLayout.from(
      key: const Key('bottomNavigation'),
      outAnimation: AdaptiveScaffold.topToBottom,  // 退出动画
      builder: (_) => AdaptiveScaffold.standardBottomNavigationBar(...),
    ),
  },
),

// 3. 详情视图：保持在屏幕上
secondaryBody: SlotLayout(
  config: {
    Breakpoints.mediumAndUp: SlotLayout.from(
      outAnimation: AdaptiveScaffold.stayOnScreen,  // 退出时保持
      key: const Key('Secondary Body'),
      builder: (_) => DetailTile(...),
    ),
  },
),
```

### 导航项的交错动画

在 `main.dart` 中，导航项使用了交错动画（staggered animation）：

```dart
NavigationRailDestination slideInNavigationItem({
  required double begin,
  required AnimationController controller,
  required IconData icon,
  required String label,
}) {
  return NavigationRailDestination(
    icon: SlideTransition(
      position: Tween<Offset>(
        begin: Offset(begin, 0),  // 不同的起始位置
        end: Offset.zero,
      ).animate(
        CurvedAnimation(
          parent: controller,
          curve: Curves.easeInOutCubic,
        ),
      ),
      child: Icon(icon),
    ),
    label: Text(label),
  );
}

// 使用不同的延迟
slideInNavigationItem(
  begin: -1,
  controller: _inboxIconSlideController,
  icon: Icons.inbox,
  label: 'Inbox',
),
slideInNavigationItem(
  begin: -2,
  controller: _articleIconSlideController,
  icon: Icons.article_outlined,
  label: 'Articles',
),
```

**效果**：导航项依次从左侧滑入，创造流畅的视觉体验

## 动画性能优化

### 1. 避免过度动画

过多的动画会影响性能，特别是低端设备：

```dart
// ✅ 推荐：只在必要时使用动画
SlotLayout.from(
  key: const Key('Body'),
  inAnimation: AdaptiveScaffold.leftOutIn,  // 简单动画
  builder: (_) => Widget(),
)

// ❌ 避免：复杂的组合动画
SlotLayout.from(
  key: const Key('Body'),
  inAnimation: (child, animation) {
    return FadeTransition(
      opacity: animation,
      child: SlideTransition(
        position: ...,
        child: ScaleTransition(
          scale: ...,
          child: RotationTransition(
            turns: ...,
            child: child,
          ),
        ),
      ),
    );
  },
  builder: (_) => Widget(),
)
```

### 2. 使用合适的动画时长

过长的动画会让用户感到延迟：

```dart
// ✅ 推荐：快速切换
inDuration: Duration(milliseconds: 300),

// ❌ 避免：过长的动画
inDuration: Duration(milliseconds: 2000),  // 太慢了
```

### 3. 避免不必要的动画

对于不重要的内容，可以省略动画：

```dart
// ✅ 推荐：重要内容使用动画
primaryNavigation: SlotLayout(
  config: {
    Breakpoints.medium: SlotLayout.from(
      key: const Key('Nav'),
      inAnimation: AdaptiveScaffold.leftOutIn,
      builder: (_) => NavigationRail(...),
    ),
  },
),

// ✅ 也可以：不重要的内容省略动画
body: SlotLayout(
  config: {
    Breakpoints.standard: SlotLayout.from(
      key: const Key('Body'),
      // 不设置 inAnimation，使用默认行为
      builder: (_) => Widget(),
    ),
  },
)
```

### 4. 使用 RepaintBoundary

对于复杂的 Widget，使用 `RepaintBoundary` 隔离重绘：

```dart
SlotLayout.from(
  key: const Key('Body'),
  builder: (_) => RepaintBoundary(
    child: ComplexWidget(),  // 复杂 Widget
  ),
)
```

## 动画最佳实践

### 1. 保持一致性

在整个应用中使用一致的动画风格：

```dart
// 定义统一的动画函数
final Widget Function(Widget, Animation<double>) standardInAnimation =
    AdaptiveScaffold.leftOutIn;

final Widget Function(Widget, Animation<double>) standardOutAnimation =
    AdaptiveScaffold.rightOutIn;

// 在所有槽位中使用
primaryNavigation: SlotLayout(
  config: {
    Breakpoints.medium: SlotLayout.from(
      key: const Key('Nav'),
      inAnimation: standardInAnimation,
      builder: (_) => NavigationRail(...),
    ),
  },
)
```

### 2. 考虑用户体验

动画应该增强而不是干扰用户体验：

- **快速响应**：动画不应该让用户等待
- **自然流畅**：使用符合物理直觉的动画
- **适度使用**：不要过度使用动画

### 3. 测试不同设备

在不同性能的设备上测试动画：

- 高端设备：可以承受更复杂的动画
- 低端设备：应该使用简单的动画或减少动画

### 4. 提供动画开关

为需要高性能的用户提供关闭动画的选项：

```dart
final bool enableAnimations = Preferences.getBool('enable_animations', true);

SlotLayout.from(
  key: const Key('Body'),
  inAnimation: enableAnimations
      ? AdaptiveScaffold.leftOutIn
      : (child, _) => child,  // 无动画
  builder: (_) => Widget(),
)
```

## 总结

本章我们深入学习了：

- **动画系统架构**：`AnimatedSwitcher` 的工作原理
- **内置动画**：leftOutIn、rightOutIn、topToBottom、bottomToTop、stayOnScreen
- **自定义动画**：淡入淡出、滑动、缩放、旋转等
- **动画控制**：时长和曲线的设置
- **实际应用**：邮件应用中的动画使用
- **性能优化**：避免过度动画、使用合适的时长
- **最佳实践**：一致性、用户体验、测试

在下一章中，我们将学习如何将自适应布局与路由系统（GoRouter）集成。

## 练习

1. 创建一个自定义动画，结合淡入、滑动和缩放效果
2. 为不同的槽位配置不同的动画，观察效果
3. 尝试调整动画时长和曲线，找到最佳体验

## 检查清单

- [ ] 理解动画系统的工作原理
- [ ] 掌握所有内置动画的使用
- [ ] 能够创建自定义动画
- [ ] 了解如何控制动画时长和曲线
- [ ] 理解动画性能优化技巧
- [ ] 了解动画最佳实践
