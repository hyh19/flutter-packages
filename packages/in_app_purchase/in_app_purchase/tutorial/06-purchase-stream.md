# 第 6 章：监听购买更新流

## 本章目标

- 理解购买更新流的工作原理
- 实现购买流的监听
- 处理不同的购买状态
- 实现错误处理和边缘情况

## 购买更新流概述

### 什么是购买更新流？

购买更新流（Purchase Stream）是 `in_app_purchase` 插件提供的核心机制，用于接收购买状态的实时更新。

```dart
Stream<List<PurchaseDetails>> purchaseStream = InAppPurchase.instance.purchaseStream;
```

### 为什么需要监听购买流？

1. **捕获购买状态变化**
   - 用户发起购买
   - 购买成功或失败
   - 订阅续订
   - 购买恢复

2. **处理应用重启后的购买**
   - 应用在购买过程中被关闭
   - 系统在后台完成了购买
   - 重新打开应用时接收购买信息

3. **处理自动续订**
   - 订阅自动续订时会触发购买更新
   - 无需用户操作即可接收更新

### 何时开始监听？

⚠️ **关键**：必须在应用启动时尽早开始监听，甚至在用户发起购买前。

原因：

- 可能有待处理的购买
- 可能有之前未完成的交易
- 订阅可能在后台自动续订

## 购买状态详解

### PurchaseDetails 对象

每个购买更新都包含一个 `PurchaseDetails` 对象，主要属性：

```dart
class PurchaseDetails {
  final String productID;                  // 产品 ID
  final String purchaseID;                 // 购买 ID（交易 ID）
  final String? transactionDate;           // 交易日期
  final PurchaseStatus status;             // 购买状态
  final IAPError? error;                   // 错误信息（如果有）
  final bool pendingCompletePurchase;      // 是否需要完成购买
  final String? verificationData.localVerificationData;  // 验证数据
  final String? verificationData.serverVerificationData; // 服务器验证数据
}
```

### 购买状态类型

```dart
enum PurchaseStatus {
  pending,      // 待处理（等待用户操作）
  purchased,    // 购买成功
  error,        // 购买失败
  restored,     // 恢复的购买
  canceled,     // 用户取消（Android only）
}
```

#### 状态说明

1. **pending（待处理）**
   - 用户正在处理购买（如输入密码）
   - 等待家长批准（家庭共享）
   - 等待支付确认

2. **purchased（已购买）**
   - 购买成功完成
   - 需要验证购买凭证
   - 需要解锁内容
   - 必须调用 `completePurchase()`

3. **error（错误）**
   - 购买过程中发生错误
   - 检查 `error` 属性获取详情
   - 可能是网络问题、支付失败等

4. **restored（已恢复）**
   - 用户恢复了之前的购买
   - 与 purchased 类似，需要验证
   - 也需要调用 `completePurchase()`

5. **canceled（已取消）**
   - 仅 Android 支持
   - 用户主动取消了购买

## 步骤 1：扩展 SubscriptionService

更新 `lib/services/subscription_service.dart`，添加购买流监听：

```dart
import 'dart:async';
import 'dart:io';
import 'package:in_app_purchase/in_app_purchase.dart';

class SubscriptionService {
  // ==================== 单例模式 ====================
  static final SubscriptionService _instance = SubscriptionService._internal();
  factory SubscriptionService() => _instance;
  SubscriptionService._internal();

  // ==================== 私有属性 ====================
  final InAppPurchase _inAppPurchase = InAppPurchase.instance;
  StreamSubscription<List<PurchaseDetails>>? _subscription;
  bool _isInitialized = false;
  bool _isAvailable = false;

  // ==================== 公共属性 ====================
  bool get isInitialized => _isInitialized;
  bool get isAvailable => _isAvailable;

  // ==================== 初始化方法 ====================
  
  Future<bool> initialize() async {
    if (_isInitialized) {
      print('IAP 服务已经初始化');
      return _isAvailable;
    }

    print('开始初始化 IAP 服务...');

    // 检查平台支持
    if (!Platform.isIOS && !Platform.isAndroid) {
      print('当前平台不支持应用内购买');
      _isInitialized = true;
      _isAvailable = false;
      return false;
    }

    try {
      // 检查商店可用性
      _isAvailable = await _inAppPurchase.isAvailable();

      if (_isAvailable) {
        // 开始监听购买更新流
        _listenToPurchaseUpdates();
        print('IAP 服务初始化成功');
      } else {
        print('应用内购买服务不可用');
      }

      _isInitialized = true;
      return _isAvailable;
    } catch (e, stackTrace) {
      print('初始化 IAP 服务时出错: $e');
      print('堆栈跟踪: $stackTrace');
      _isInitialized = true;
      _isAvailable = false;
      return false;
    }
  }

  // ==================== 购买流监听 ====================
  
  /// 监听购买更新流
  void _listenToPurchaseUpdates() {
    // 取消之前的订阅（如果有）
    _subscription?.cancel();

    // 订阅购买更新流
    _subscription = _inAppPurchase.purchaseStream.listen(
      _handlePurchaseUpdates,
      onDone: _onPurchaseStreamDone,
      onError: _onPurchaseStreamError,
    );

    print('开始监听购买更新流');
  }

  /// 处理购买更新
  Future<void> _handlePurchaseUpdates(List<PurchaseDetails> purchaseDetailsList) async {
    print('收到 ${purchaseDetailsList.length} 个购买更新');

    for (final purchaseDetails in purchaseDetailsList) {
      await _handlePurchaseUpdate(purchaseDetails);
    }
  }

  /// 处理单个购买更新
  Future<void> _handlePurchaseUpdate(PurchaseDetails purchaseDetails) async {
    print('处理购买更新: ${purchaseDetails.productID}, 状态: ${purchaseDetails.status}');

    switch (purchaseDetails.status) {
      case PurchaseStatus.pending:
        await _handlePendingPurchase(purchaseDetails);
        break;

      case PurchaseStatus.purchased:
        await _handleSuccessfulPurchase(purchaseDetails);
        break;

      case PurchaseStatus.error:
        await _handleFailedPurchase(purchaseDetails);
        break;

      case PurchaseStatus.restored:
        await _handleRestoredPurchase(purchaseDetails);
        break;

      case PurchaseStatus.canceled:
        await _handleCanceledPurchase(purchaseDetails);
        break;
    }

    // 完成购买（如果需要）
    if (purchaseDetails.pendingCompletePurchase) {
      await _completePurchase(purchaseDetails);
    }
  }

  /// 处理待处理的购买
  Future<void> _handlePendingPurchase(PurchaseDetails purchaseDetails) async {
    print('购买待处理: ${purchaseDetails.productID}');
    // 显示加载指示器或等待提示
    // 将在第 9 章详细实现
  }

  /// 处理成功的购买
  Future<void> _handleSuccessfulPurchase(PurchaseDetails purchaseDetails) async {
    print('购买成功: ${purchaseDetails.productID}');

    // 1. 验证购买（将在第 10 章详细实现）
    final bool valid = await _verifyPurchase(purchaseDetails);

    if (valid) {
      // 2. 解锁订阅内容
      await _unlockSubscription(purchaseDetails);
      print('订阅已解锁: ${purchaseDetails.productID}');
    } else {
      print('购买验证失败: ${purchaseDetails.productID}');
      // 处理无效购买
    }
  }

  /// 处理失败的购买
  Future<void> _handleFailedPurchase(PurchaseDetails purchaseDetails) async {
    print('购买失败: ${purchaseDetails.productID}');
    
    if (purchaseDetails.error != null) {
      final error = purchaseDetails.error!;
      print('错误代码: ${error.code}');
      print('错误消息: ${error.message}');
      print('错误详情: ${error.details}');
      
      // 显示错误消息给用户
      _showErrorToUser(error);
    }
  }

  /// 处理恢复的购买
  Future<void> _handleRestoredPurchase(PurchaseDetails purchaseDetails) async {
    print('恢复购买: ${purchaseDetails.productID}');

    // 恢复的购买与新购买的处理类似
    final bool valid = await _verifyPurchase(purchaseDetails);

    if (valid) {
      await _unlockSubscription(purchaseDetails);
      print('订阅已恢复: ${purchaseDetails.productID}');
    } else {
      print('恢复验证失败: ${purchaseDetails.productID}');
    }
  }

  /// 处理取消的购买
  Future<void> _handleCanceledPurchase(PurchaseDetails purchaseDetails) async {
    print('购买已取消: ${purchaseDetails.productID}');
    // 更新 UI，移除加载指示器
  }

  /// 完成购买
  Future<void> _completePurchase(PurchaseDetails purchaseDetails) async {
    try {
      await _inAppPurchase.completePurchase(purchaseDetails);
      print('购买已完成: ${purchaseDetails.productID}');
    } catch (e) {
      print('完成购买时出错: $e');
    }
  }

  // ==================== 辅助方法（占位符） ====================
  
  /// 验证购买（将在第 10 章实现）
  Future<bool> _verifyPurchase(PurchaseDetails purchaseDetails) async {
    // 占位符实现
    print('验证购买: ${purchaseDetails.productID}');
    return true;
  }

  /// 解锁订阅内容
  Future<void> _unlockSubscription(PurchaseDetails purchaseDetails) async {
    // 占位符实现
    print('解锁订阅: ${purchaseDetails.productID}');
    // 实际实现：
    // - 更新本地状态
    // - 通知服务器
    // - 更新 UI
  }

  /// 显示错误给用户
  void _showErrorToUser(IAPError error) {
    // 占位符实现
    print('向用户显示错误: ${error.message}');
    // 实际实现：使用 SnackBar、Dialog 等
  }

  /// 购买流完成回调
  void _onPurchaseStreamDone() {
    print('购买更新流已关闭');
  }

  /// 购买流错误回调
  void _onPurchaseStreamError(error) {
    print('购买更新流错误: $error');
  }

  // ==================== 资源清理 ====================
  
  void dispose() {
    _subscription?.cancel();
    _subscription = null;
    print('订阅服务已释放');
  }
}
```

## 步骤 2：处理购买错误

### 常见错误类型

创建 `lib/utils/iap_error_handler.dart`：

```dart
import 'package:in_app_purchase/in_app_purchase.dart';

class IAPErrorHandler {
  /// 获取用户友好的错误消息
  static String getUserFriendlyMessage(IAPError error) {
    // iOS 错误代码
    if (error.code == 'storekit_duplicate_product_object') {
      return '重复的购买请求，请稍后再试';
    }

    // Android 错误代码
    switch (error.code) {
      case 'BillingResponse.itemAlreadyOwned':
        return '您已拥有此订阅';
      
      case 'BillingResponse.userCanceled':
        return '购买已取消';
      
      case 'BillingResponse.itemUnavailable':
        return '此商品暂时不可用';
      
      case 'BillingResponse.networkError':
        return '网络连接错误，请检查网络后重试';
      
      case 'BillingResponse.error':
        return '购买过程中发生错误，请稍后再试';
      
      default:
        return error.message ?? '未知错误，请稍后再试';
    }
  }

  /// 判断错误是否可重试
  static bool isRetryable(IAPError error) {
    final retryableCodes = [
      'BillingResponse.networkError',
      'BillingResponse.serviceTimeout',
      'BillingResponse.serviceUnavailable',
    ];

    return retryableCodes.contains(error.code);
  }

  /// 判断是否为用户取消
  static bool isUserCanceled(IAPError error) {
    return error.code == 'BillingResponse.userCanceled' ||
           error.code == 'storekit_user_cancelled';
  }
}
```

## 步骤 3：创建购买状态管理

创建 `lib/models/purchase_state.dart`：

```dart
import 'package:in_app_purchase/in_app_purchase.dart';

/// 购买状态管理类
class PurchaseState {
  final bool isPending;
  final bool isPurchased;
  final bool hasError;
  final String? errorMessage;
  final PurchaseDetails? purchaseDetails;

  const PurchaseState({
    this.isPending = false,
    this.isPurchased = false,
    this.hasError = false,
    this.errorMessage,
    this.purchaseDetails,
  });

  /// 初始状态
  factory PurchaseState.initial() {
    return const PurchaseState();
  }

  /// 待处理状态
  factory PurchaseState.pending(PurchaseDetails details) {
    return PurchaseState(
      isPending: true,
      purchaseDetails: details,
    );
  }

  /// 购买成功状态
  factory PurchaseState.purchased(PurchaseDetails details) {
    return PurchaseState(
      isPurchased: true,
      purchaseDetails: details,
    );
  }

  /// 错误状态
  factory PurchaseState.error(String message, PurchaseDetails details) {
    return PurchaseState(
      hasError: true,
      errorMessage: message,
      purchaseDetails: details,
    );
  }

  /// 复制并修改
  PurchaseState copyWith({
    bool? isPending,
    bool? isPurchased,
    bool? hasError,
    String? errorMessage,
    PurchaseDetails? purchaseDetails,
  }) {
    return PurchaseState(
      isPending: isPending ?? this.isPending,
      isPurchased: isPurchased ?? this.isPurchased,
      hasError: hasError ?? this.hasError,
      errorMessage: errorMessage ?? this.errorMessage,
      purchaseDetails: purchaseDetails ?? this.purchaseDetails,
    );
  }
}
```

## 步骤 4：在 UI 中响应购买状态

### 使用 StreamBuilder 监听

```dart
import 'package:flutter/material.dart';
import 'package:in_app_purchase/in_app_purchase.dart';

class PurchaseStatusWidget extends StatelessWidget {
  const PurchaseStatusWidget({super.key});

  @override
  Widget build(BuildContext context) {
    final InAppPurchase inAppPurchase = InAppPurchase.instance;

    return StreamBuilder<List<PurchaseDetails>>(
      stream: inAppPurchase.purchaseStream,
      builder: (context, snapshot) {
        if (!snapshot.hasData) {
          return const SizedBox.shrink();
        }

        final purchases = snapshot.data!;
        if (purchases.isEmpty) {
          return const SizedBox.shrink();
        }

        // 获取最新的购买
        final latestPurchase = purchases.last;

        return _buildStatusWidget(context, latestPurchase);
      },
    );
  }

  Widget _buildStatusWidget(BuildContext context, PurchaseDetails purchase) {
    switch (purchase.status) {
      case PurchaseStatus.pending:
        return _buildPendingWidget();
      
      case PurchaseStatus.purchased:
        return _buildSuccessWidget();
      
      case PurchaseStatus.error:
        return _buildErrorWidget(purchase.error);
      
      default:
        return const SizedBox.shrink();
    }
  }

  Widget _buildPendingWidget() {
    return const Card(
      child: Padding(
        padding: EdgeInsets.all(16.0),
        child: Row(
          children: [
            CircularProgressIndicator(),
            SizedBox(width: 16),
            Text('正在处理购买...'),
          ],
        ),
      ),
    );
  }

  Widget _buildSuccessWidget() {
    return Card(
      color: Colors.green[50],
      child: const Padding(
        padding: EdgeInsets.all(16.0),
        child: Row(
          children: [
            Icon(Icons.check_circle, color: Colors.green),
            SizedBox(width: 16),
            Text('购买成功！'),
          ],
        ),
      ),
    );
  }

  Widget _buildErrorWidget(IAPError? error) {
    return Card(
      color: Colors.red[50],
      child: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Row(
          children: [
            const Icon(Icons.error, color: Colors.red),
            const SizedBox(width: 16),
            Expanded(
              child: Text(
                error?.message ?? '购买失败',
                style: const TextStyle(color: Colors.red),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

## 测试购买流监听

### 测试场景

1. **应用启动测试**
   - 启动应用
   - 检查日志确认开始监听
   - 验证没有错误

2. **购买流程测试**
   - 发起一个测试购买
   - 观察日志中的状态变化
   - 验证状态处理正确

3. **应用重启测试**
   - 在购买过程中关闭应用
   - 重新打开应用
   - 验证未完成的购买被正确处理

### 调试日志示例

正常流程的日志：

```
[IAP] 开始初始化 IAP 服务...
[IAP] IAP 服务初始化成功
[IAP] 开始监听购买更新流
[IAP] 收到 1 个购买更新
[IAP] 处理购买更新: monthly_premium, 状态: PurchaseStatus.purchased
[IAP] 购买成功: monthly_premium
[IAP] 验证购买: monthly_premium
[IAP] 订阅已解锁: monthly_premium
[IAP] 购买已完成: monthly_premium
```

## 常见问题

### Q1：为什么 purchaseStream 会发送多次相同的购买？

这是正常行为。在以下情况会发生：

- 购买未完成（未调用 `completePurchase()`）
- 应用重启时
- 购买验证失败时

解决方法：使用购买 ID 去重，或正确调用 `completePurchase()`。

### Q2：何时调用 completePurchase()？

**必须在以下操作完成后调用**：

1. 验证购买凭证
2. 向服务器报告购买
3. 解锁内容给用户

⚠️ **警告**：过早调用可能导致用户付费但未获得内容。

### Q3：如果忘记调用 completePurchase() 会怎样？

- 购买会一直处于待完成状态
- 每次应用启动都会收到该购买更新
- 超过 3 天未完成会导致退款（iOS）

### Q4：可以在不同页面监听购买流吗？

**不推荐**。应该在全局（如服务类）监听一次，然后通过状态管理（如 Provider、Bloc）分发状态。

## 小结

在本章中，我们学习了：

- 购买更新流的工作原理和重要性
- 如何监听购买更新流
- 如何处理不同的购买状态
- 如何处理购买错误
- 如何在 UI 中响应购买状态

掌握购买流监听后，下一步是加载订阅产品。

## 下一步

[第 7 章：加载订阅产品](07-load-products.md) - 学习如何从商店加载订阅产品信息。
