# Flutter Adaptive Scaffold 教程系列

## 简介

本教程系列深入讲解 `flutter_adaptive_scaffold` 库的使用方法、实现原理和最佳实践。通过分析官方示例项目，帮助高级开发者全面掌握 Flutter 自适应布局的核心技术。

## 目标受众

本教程面向**高级 Flutter 开发者**，假设读者已经：

- 熟悉 Flutter 基础 Widget 和布局系统
- 理解 Material Design 设计规范
- 具备响应式布局的基础知识
- 了解状态管理和路由系统

## 学习目标

完成本教程后，你将能够：

- 深入理解 Material Design 3 自适应设计理念
- 掌握 `AdaptiveScaffold` 和 `AdaptiveLayout` 的使用方法
- 理解断点系统和槽位布局的工作原理
- 实现复杂的响应式布局和动画效果
- 将自适应布局与路由系统（如 GoRouter）集成
- 在实际项目中应用最佳实践

## 教程结构

本教程共分为 10 章，从基础概念到实战应用，循序渐进：

1. [第 1 章：介绍与概述](chapter-01-introduction.md)
   - Material Design 3 自适应设计理念
   - 库的定位和优势
   - 示例项目结构概览

2. [第 2 章：核心概念解析](chapter-02-core-concepts.md)
   - AdaptiveScaffold vs AdaptiveLayout
   - 槽位系统详解
   - 响应式布局原理

3. [第 3 章：断点系统详解](chapter-03-breakpoints.md)
   - Breakpoint 类设计原理
   - 标准断点与自定义断点
   - 断点匹配逻辑

4. [第 4 章：AdaptiveScaffold 基础与进阶](chapter-04-adaptive-scaffold.md)
   - 基础用法和配置
   - 导航元素自定义
   - 静态辅助方法

5. [第 5 章：AdaptiveLayout 高级用法](chapter-05-adaptive-layout.md)
   - 完整 API 解析
   - 槽位配置方法
   - 性能优化技巧

6. [第 6 章：SlotLayout 槽位布局系统](chapter-06-slot-layout.md)
   - SlotLayout 工作原理
   - 多断点配置策略
   - 槽位协调机制

7. [第 7 章：动画与过渡效果](chapter-07-animations.md)
   - 内置动画类型
   - 自定义动画创建
   - 性能优化

8. [第 8 章：与路由系统集成](chapter-08-routing-integration.md)
   - GoRouter 集成方法
   - 路由状态管理
   - 认证流程处理

9. [第 9 章：实战案例 - 邮件应用](chapter-09-real-world-example.md)
   - 完整应用架构分析
   - 状态管理策略
   - 响应式导航实现

10. [第 10 章：最佳实践与常见问题](chapter-10-best-practices.md)
    - 性能优化建议
    - 常见陷阱和解决方案
    - 测试策略

## 示例项目

本教程基于 `flutter_adaptive_scaffold` 的官方示例项目，包含以下示例：

- **adaptive_scaffold_demo.dart**：`AdaptiveScaffold` 基础用法示例
- **adaptive_layout_demo.dart**：`AdaptiveLayout` 高级用法示例
- **main.dart**：完整的邮件应用示例
- **go_router_demo/**：与 GoRouter 集成的完整示例

## 前置要求

在开始学习之前，请确保：

1. 已安装 Flutter SDK（建议 3.0+）
2. 已克隆 `flutter_adaptive_scaffold` 仓库
3. 能够运行示例项目
4. 熟悉 Dart 语言和 Flutter 开发环境

## 如何使用本教程

1. **按顺序阅读**：建议按照章节顺序阅读，每章都建立在前一章的基础上
2. **运行示例**：每章都包含代码示例，建议在阅读时运行相关示例代码
3. **动手实践**：完成每章的练习，加深理解
4. **查阅源码**：结合官方源码理解实现细节

## 相关资源

- [Flutter 官方文档](https://flutter.dev/docs)
- [Material Design 3 规范](https://m3.material.io/foundations/adaptive-design/overview)
- [flutter_adaptive_scaffold GitHub](https://github.com/flutter/packages/tree/main/packages/flutter_adaptive_scaffold)
- [设计文档](https://flutter.dev/go/adaptive-layout-foldables)

## 贡献

如果你发现教程中的错误或有改进建议，欢迎提出 Issue 或 Pull Request。

---

**开始学习**：[第 1 章：介绍与概述](chapter-01-introduction.md)
