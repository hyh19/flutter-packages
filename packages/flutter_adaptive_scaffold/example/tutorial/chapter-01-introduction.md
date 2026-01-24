# 第 1 章 介绍与概述

## 引言

在现代应用开发中，响应式布局已成为不可或缺的能力。用户可能在不同尺寸的设备上使用你的应用：从手机到平板，从桌面到折叠屏设备。`flutter_adaptive_scaffold` 库正是为了解决这一挑战而设计的，它基于 Material Design 3 规范，提供了一套完整的自适应布局解决方案。

## Material Design 3 自适应设计理念

Material Design 3（M3）是 Google 最新的设计系统，其中自适应设计（Adaptive Design）是其核心特性之一。自适应设计的核心思想是：

### 设计原则

1. **内容优先**：布局应该根据可用空间智能调整，而不是简单地缩放
2. **渐进增强**：在小屏幕上提供核心功能，在大屏幕上展示更多细节
3. **一致性**：在不同设备上保持视觉和交互的一致性
4. **性能优化**：确保布局切换流畅，不影响用户体验

### 断点系统

Material Design 3 定义了标准的屏幕宽度断点：

- **Small**：0-600 dp（手机竖屏）
- **Medium**：600-840 dp（手机横屏、小平板）
- **Medium Large**：840-1200 dp（平板）
- **Large**：1200-1600 dp（桌面）
- **Extra Large**：1600+ dp（大桌面）

这些断点决定了应用在不同屏幕尺寸下的布局策略。

## flutter_adaptive_scaffold 库的定位

### 库的使命

`flutter_adaptive_scaffold` 库旨在简化 Flutter 应用中自适应布局的实现。它提供了两个主要组件：

1. **AdaptiveScaffold**：高级 API，开箱即用，适合大多数场景
2. **AdaptiveLayout**：底层 API，提供完全的控制权，适合复杂需求

### 核心优势

#### 1. 符合 Material Design 3 规范

库完全遵循 Material Design 3 的设计规范，包括：

- 标准的断点定义
- 推荐的间距和边距
- 导航元素的自动适配
- 动画过渡效果

#### 2. 简化开发流程

使用 `AdaptiveScaffold`，你只需要：

```dart
AdaptiveScaffold(
  destinations: destinations,
  body: (_) => YourContent(),
  smallBody: (_) => YourMobileLayout(),
  largeBody: (_) => YourDesktopLayout(),
)
```

库会自动处理：

- 导航栏在不同断点下的切换（BottomNavigationBar ↔ NavigationRail）
- 布局的平滑过渡动画
- 平台特定的适配

#### 3. 灵活的定制能力

虽然 `AdaptiveScaffold` 提供了便捷的默认实现，但通过 `AdaptiveLayout` 和 `SlotLayout`，你可以：

- 自定义任意断点的布局
- 控制每个槽位的显示和隐藏
- 定义自定义的动画效果
- 实现复杂的多面板布局

### 适用场景

#### 理想场景

- **跨平台应用**：需要在手机、平板、桌面等多个平台运行
- **响应式 Web 应用**：需要适配不同浏览器窗口尺寸
- **折叠屏设备**：需要处理屏幕展开和折叠的状态
- **复杂导航结构**：需要主从视图、多级导航等

#### 局限性

需要注意的是，该库目前已经**停止维护**（Discontinued）。虽然功能完整且稳定，但不会再有新功能更新。对于新项目，建议：

1. 评估是否真的需要如此复杂的自适应布局
2. 考虑使用社区维护的替代方案
3. 如果使用，需要自己维护和扩展

## 示例项目结构概览

让我们来看看示例项目的结构，了解库的实际应用：

```text
example/
├── lib/
│   ├── main.dart                    # 完整的邮件应用示例
│   ├── adaptive_scaffold_demo.dart  # AdaptiveScaffold 基础示例
│   ├── adaptive_layout_demo.dart    # AdaptiveLayout 高级示例
│   └── go_router_demo/              # 与 GoRouter 集成的示例
│       ├── app_router.dart
│       ├── scaffold_shell.dart
│       └── pages/
└── images/                          # 示例图片资源
```

### 主要示例文件

#### 1. adaptive_scaffold_demo.dart

这是最简单的示例，展示了 `AdaptiveScaffold` 的基本用法：

- 定义导航目标（destinations）
- 为不同断点配置不同的 body
- 配置 secondaryBody（详情视图）
- 自定义过渡动画时长

**关键特性**：

- 小屏幕使用 `ListView`
- 大屏幕使用 `GridView`，列数随屏幕增大而增加
- 自动在 `BottomNavigationBar` 和 `NavigationRail` 之间切换

#### 2. adaptive_layout_demo.dart

这个示例展示了 `AdaptiveLayout` 的完整能力：

- 手动配置 `primaryNavigation` 槽位
- 在不同断点下显示不同样式的 `NavigationRail`
- 控制 `bottomNavigation` 的显示时机
- 自定义动画效果

**关键特性**：

- 更细粒度的控制
- 可以添加自定义的 leading 和 trailing 组件
- 支持条件渲染

#### 3. main.dart（邮件应用）

这是一个完整的、功能丰富的邮件应用示例，展示了：

- 复杂的多面板布局
- 主列表和详情视图的协调
- 响应式导航（小屏幕用模态，大屏幕用侧边栏）
- 自定义动画效果
- 状态管理

**关键特性**：

- 使用 `AdaptiveLayout` 实现完全自定义的布局
- 实现了 Material Design 3 邮件应用的布局模式
- 包含搜索、文件夹等完整功能

#### 4. go_router_demo/

这个示例展示了如何将自适应布局与 GoRouter 路由系统集成：

- 使用 `StatefulShellRoute` 实现分支导航
- 认证流程的处理
- 模态对话框的显示
- 错误页面的处理

**关键特性**：

- 路由状态与导航状态的同步
- 支持深度链接
- 处理认证重定向

## 核心概念预览

在深入细节之前，让我们先了解几个核心概念：

### 1. 槽位（Slots）

`AdaptiveLayout` 将屏幕划分为多个槽位：

- **primaryNavigation**：主导航（通常是左侧的 NavigationRail）
- **secondaryNavigation**：次导航（右侧导航，较少使用）
- **topNavigation**：顶部导航栏
- **bottomNavigation**：底部导航栏
- **body**：主内容区域
- **secondaryBody**：次内容区域（常用于详情视图）

### 2. 断点（Breakpoints）

断点定义了屏幕尺寸的阈值，用于决定显示哪个布局：

```dart
Breakpoints.small      // 0-600 dp
Breakpoints.medium     // 600-840 dp
Breakpoints.mediumLarge // 840-1200 dp
Breakpoints.large      // 1200-1600 dp
Breakpoints.extraLarge // 1600+ dp
```

### 3. SlotLayout

`SlotLayout` 将断点映射到具体的布局配置：

```dart
SlotLayout(
  config: {
    Breakpoints.small: SlotLayout.from(
      key: const Key('Small Layout'),
      builder: (_) => SmallScreenWidget(),
    ),
    Breakpoints.large: SlotLayout.from(
      key: const Key('Large Layout'),
      builder: (_) => LargeScreenWidget(),
    ),
  },
)
```

## 学习路径建议

基于示例项目的复杂度，建议的学习路径：

1. **入门**：从 `adaptive_scaffold_demo.dart` 开始，理解基本概念
2. **进阶**：学习 `adaptive_layout_demo.dart`，掌握底层 API
3. **实战**：分析 `main.dart`，理解复杂应用的设计
4. **集成**：研究 `go_router_demo`，学习与路由系统的配合

## 总结

本章我们了解了：

- Material Design 3 的自适应设计理念和断点系统
- `flutter_adaptive_scaffold` 库的定位、优势和局限性
- 示例项目的结构和各个示例的特点
- 核心概念的初步介绍

在下一章中，我们将深入探讨这些核心概念，理解 `AdaptiveScaffold` 和 `AdaptiveLayout` 的设计原理。

## 练习

1. 运行 `adaptive_scaffold_demo.dart`，尝试调整窗口大小，观察布局变化
2. 查看 `adaptive_scaffold_demo.dart` 的代码，理解 `destinations` 和不同断点的 `body` 配置
3. 思考你的项目中哪些场景适合使用自适应布局

## 检查清单

- [ ] 理解 Material Design 3 的自适应设计理念
- [ ] 了解 `flutter_adaptive_scaffold` 库的定位和优势
- [ ] 熟悉示例项目的结构
- [ ] 理解槽位、断点、SlotLayout 等核心概念
- [ ] 能够运行和观察示例应用的行为
