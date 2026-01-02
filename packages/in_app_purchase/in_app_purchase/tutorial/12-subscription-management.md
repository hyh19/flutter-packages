# 第 12 章：订阅升级和降级

## 本章目标

- 理解订阅升级和降级的概念
- 实现 iOS 订阅组机制
- 实现 Android 订阅升级和降级
- 处理升降级后的订阅状态

## 订阅升降级概述

### 什么是订阅升降级？

- **升级（Upgrade）**：从低等级订阅切换到高等级订阅
  - 例如：基础月度会员 → 高级月度会员
  - 例如：月度订阅 → 年度订阅

- **降级（Downgrade）**：从高等级订阅切换到低等级订阅
  - 例如：高级会员 → 基础会员
  - 例如：年度订阅 → 月度订阅

### iOS vs Android 的区别

| 特性 | iOS | Android |
|------|-----|---------|
| 机制 | 订阅组自动处理 | 手动指定旧订阅 |
| 升级生效时间 | 立即 | 立即或周期结束 |
| 降级生效时间 | 当前周期结束 | 当前周期结束 |
| 退款处理 | 按比例退款 | 按策略处理 |
| 代码复杂度 | 简单 | 较复杂 |

## iOS 订阅升降级

### 订阅组机制

iOS 使用订阅组（Subscription Group）管理升降级：

1. 同一订阅组内的产品被视为同一服务的不同等级
2. 用户同时只能订阅组内的一个产品
3. 切换订阅时，iOS 自动处理升降级

### 配置订阅组

在 App Store Connect 中：

1. 创建订阅组（例如："VIP 会员"）
2. 在组内添加多个订阅产品
3. 设置产品等级（1 = 最低，数字越大等级越高）

```
订阅组：VIP 会员
├── 基础月度会员（等级 1）
├── 高级月度会员（等级 2）
├── 高级年度会员（等级 2）
└── 旗舰月度会员（等级 3）
```

### iOS 代码实现

iOS 的订阅升降级非常简单，只需像正常购买一样调用 `buyNonConsumable()`：

```dart
import 'package:in_app_purchase/in_app_purchase.dart';

class SubscriptionService {
  // ...

  /// iOS 升级或降级订阅
  /// 
  /// 只需购买新的订阅产品，iOS 会自动处理升降级
  Future<bool> changeSubscriptionIOS(ProductDetails newProduct) async {
    print('iOS 切换订阅: ${newProduct.id}');

    // 直接购买新订阅
    final purchaseParam = PurchaseParam(productDetails: newProduct);
    
    return await _inAppPurchase.buyNonConsumable(
      purchaseParam: purchaseParam,
    );
  }
}
```

### iOS 升降级规则

#### 升级（Upgrade）

- **生效时间**：立即生效
- **计费**：立即扣除新订阅费用
- **退款**：旧订阅按比例退款
- **示例**：
  - 用户订阅了月度基础会员（¥12/月）
  - 15 天后升级到月度高级会员（¥18/月）
  - 立即支付 ¥18，旧订阅的剩余 15 天按比例退款

#### 降级（Downgrade）

- **生效时间**：当前周期结束后生效
- **计费**：当前周期结束后扣费
- **示例**：
  - 用户订阅了月度高级会员（¥18/月）
  - 15 天后降级到月度基础会员（¥12/月）
  - 继续享受高级会员直到当前周期结束
  - 下个周期开始扣 ¥12

#### 跨周期切换

- **月度 → 年度**：通常视为升级（立即生效）
- **年度 → 月度**：通常视为降级（周期结束后生效）

## Android 订阅升降级

### Android 机制

Android 需要在代码中明确指定：

1. 旧订阅的购买详情
2. 替换模式（ProrationMode）
3. 新订阅的产品详情

### 替换模式说明

```dart
enum ReplacementMode {
  // 立即替换，按比例计费
  withTimeProration,
  
  // 立即替换，不按比例计费（使用剩余时间）
  chargeFullPrice,
  
  // 当前周期结束后替换
  deferredProration,
  
  // 立即替换并延长周期
  withoutProration,
  
  // 充值模式（未使用的时间保留）
  chargeProrated Price,
}
```

### 实现 Android 升降级

```dart
import 'dart:io';
import 'package:in_app_purchase/in_app_purchase.dart';
import 'package:in_app_purchase_android/in_app_purchase_android.dart';

class SubscriptionService {
  // ...

  /// Android 升级或降级订阅
  Future<bool> changeSubscriptionAndroid(
    ProductDetails newProduct,
    PurchaseDetails oldPurchase, {
    ProrationMode? prorationMode,
  }) async {
    if (!Platform.isAndroid) {
      throw UnsupportedError('此方法仅适用于 Android');
    }

    print('Android 切换订阅: ${oldPurchase.productID} -> ${newProduct.id}');

    // 创建替换参数
    final changeParam = ChangeSubscriptionParam(
      oldPurchaseDetails: oldPurchase,
      prorationMode: prorationMode ?? ProrationMode.immediateWithTimeProration,
    );

    // 创建购买参数
    final purchaseParam = GooglePlayPurchaseParam(
      productDetails: newProduct as GooglePlayProductDetails,
      changeSubscriptionParam: changeParam,
    );

    // 执行购买
    return await _inAppPurchase.buyNonConsumable(
      purchaseParam: purchaseParam,
    );
  }

  /// 根据升级或降级选择合适的替换模式
  ProrationMode getProrationMode({
    required bool isUpgrade,
    required bool immediate,
  }) {
    if (isUpgrade) {
      // 升级：立即生效，按比例计费
      return ProrationMode.immediateWithTimeProration;
    } else {
      if (immediate) {
        // 降级但立即生效
        return ProrationMode.immediateWithoutProration;
      } else {
        // 降级：当前周期结束后生效
        return ProrationMode.deferred;
      }
    }
  }
}
```

### Android 替换模式详解

#### immediateWithTimeProration

- **用途**：升级订阅
- **效果**：立即切换，按比例退款
- **示例**：
  - 月度基础（¥12）剩余 15 天
  - 升级到月度高级（¥18）
  - 扣费：¥18 - (¥12 × 15/30) = ¥12

#### immediateWithoutProration

- **用途**：立即替换但不退款
- **效果**：立即切换，剩余时间浪费
- **示例**：
  - 月度高级（¥18）剩余 15 天
  - 切换到月度基础（¥12）
  - 立即扣 ¥12，浪费旧订阅的 15 天

#### deferred

- **用途**：降级订阅
- **效果**：当前周期结束后切换
- **示例**：
  - 月度高级（¥18）剩余 15 天
  - 切换到月度基础（¥12）
  - 继续享受高级 15 天，之后按 ¥12 扣费

## 统一的跨平台实现

### 自动选择平台

```dart
import 'dart:io';
import 'package:in_app_purchase/in_app_purchase.dart';

class SubscriptionService {
  // ...

  /// 跨平台订阅升降级
  Future<bool> changeSubscription({
    required ProductDetails newProduct,
    PurchaseDetails? currentPurchase,
    bool immediate = true,
  }) async {
    print('切换订阅: ${currentPurchase?.productID} -> ${newProduct.id}');

    if (Platform.isIOS) {
      // iOS：直接购买新订阅
      return _changeSubscriptionIOS(newProduct);
    } else if (Platform.isAndroid) {
      // Android：需要旧购买信息
      if (currentPurchase == null) {
        print('Android 升降级需要提供当前订阅信息');
        return false;
      }
      
      // 判断是升级还是降级
      final isUpgrade = _isUpgrade(currentPurchase.productID, newProduct.id);
      
      // 选择替换模式
      final prorationMode = getProrationMode(
        isUpgrade: isUpgrade,
        immediate: immediate || isUpgrade,
      );
      
      return _changeSubscriptionAndroid(
        newProduct,
        currentPurchase,
        prorationMode: prorationMode,
      );
    }

    return false;
  }

  /// 判断是否为升级
  bool _isUpgrade(String oldProductId, String newProductId) {
    // 根据产品等级判断
    final oldLevel = _getProductLevel(oldProductId);
    final newLevel = _getProductLevel(newProductId);
    
    return newLevel > oldLevel;
  }

  /// 获取产品等级
  int _getProductLevel(String productId) {
    // 定义产品等级
    const levels = {
      'monthly_basic': 1,
      'monthly_premium': 2,
      'yearly_premium': 2,
      'monthly_vip': 3,
    };
    
    return levels[productId] ?? 0;
  }

  Future<bool> _changeSubscriptionIOS(ProductDetails newProduct) async {
    final purchaseParam = PurchaseParam(productDetails: newProduct);
    return await _inAppPurchase.buyNonConsumable(
      purchaseParam: purchaseParam,
    );
  }

  Future<bool> _changeSubscriptionAndroid(
    ProductDetails newProduct,
    PurchaseDetails oldPurchase, {
    ProrationMode? prorationMode,
  }) async {
    final changeParam = ChangeSubscriptionParam(
      oldPurchaseDetails: oldPurchase,
      prorationMode: prorationMode,
    );

    final purchaseParam = GooglePlayPurchaseParam(
      productDetails: newProduct as GooglePlayProductDetails,
      changeSubscriptionParam: changeParam,
    );

    return await _inAppPurchase.buyNonConsumable(
      purchaseParam: purchaseParam,
    );
  }
}
```

## UI 实现

### 升降级按钮

```dart
import 'package:flutter/material.dart';
import '../../../../services/subscription_service.dart';
import '../../../../models/subscription_product.dart';

class ChangeSubscriptionButton extends StatefulWidget {
  final SubscriptionProduct currentSubscription;
  final SubscriptionProduct newSubscription;

  const ChangeSubscriptionButton({
    super.key,
    required this.currentSubscription,
    required this.newSubscription,
  });

  @override
  State<ChangeSubscriptionButton> createState() =>
      _ChangeSubscriptionButtonState();
}

class _ChangeSubscriptionButtonState extends State<ChangeSubscriptionButton> {
  final SubscriptionService _service = SubscriptionService();
  bool _isChanging = false;

  Future<void> _handleChange() async {
    // 显示确认对话框
    final confirmed = await _showConfirmDialog();
    if (confirmed != true) return;

    setState(() => _isChanging = true);

    try {
      // 获取当前购买详情（Android 需要）
      final currentPurchase = await _getCurrentPurchaseDetails();

      // 执行升降级
      final success = await _service.changeSubscription(
        newProduct: widget.newSubscription.productDetails,
        currentPurchase: currentPurchase,
      );

      if (!success) {
        _showError('切换失败，请重试');
      }
      // 成功消息在购买流监听器中显示
    } finally {
      if (mounted) {
        setState(() => _isChanging = false);
      }
    }
  }

  Future<PurchaseDetails?> _getCurrentPurchaseDetails() async {
    // 从已购买的列表中获取当前订阅
    // 实际实现需要保存购买详情
    return null; // 占位符
  }

  Future<bool?> _showConfirmDialog() {
    final isUpgrade = _isUpgrade();
    
    return showDialog<bool>(
      context: context,
      builder: (context) => AlertDialog(
        title: Text(isUpgrade ? '升级订阅' : '更改订阅'),
        content: Column(
          mainAxisSize: MainAxisSize.min,
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text('当前：${widget.currentSubscription.shortTitle}'),
            Text('切换到：${widget.newSubscription.shortTitle}'),
            const SizedBox(height: 16),
            Text(
              isUpgrade
                  ? '升级将立即生效，旧订阅的剩余时间会按比例退款。'
                  : '降级将在当前订阅周期结束后生效。',
              style: const TextStyle(fontSize: 12, color: Colors.grey),
            ),
          ],
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.of(context).pop(false),
            child: const Text('取消'),
          ),
          ElevatedButton(
            onPressed: () => Navigator.of(context).pop(true),
            child: Text(isUpgrade ? '升级' : '更改'),
          ),
        ],
      ),
    );
  }

  bool _isUpgrade() {
    return widget.newSubscription.rawPrice >
        widget.currentSubscription.rawPrice;
  }

  void _showError(String message) {
    if (!mounted) return;
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text(message), backgroundColor: Colors.red),
    );
  }

  @override
  Widget build(BuildContext context) {
    final isUpgrade = _isUpgrade();
    
    return ElevatedButton(
      onPressed: _isChanging ? null : _handleChange,
      style: ElevatedButton.styleFrom(
        backgroundColor: isUpgrade ? Colors.blue : Colors.grey,
      ),
      child: _isChanging
          ? const SizedBox(
              width: 20,
              height: 20,
              child: CircularProgressIndicator(strokeWidth: 2),
            )
          : Text(isUpgrade ? '升级' : '更改计划'),
    );
  }
}
```

### 订阅管理页面

```dart
class ManageSubscriptionScreen extends StatelessWidget {
  final SubscriptionProduct currentSubscription;
  final List<SubscriptionProduct> availableSubscriptions;

  const ManageSubscriptionScreen({
    super.key,
    required this.currentSubscription,
    required this.availableSubscriptions,
  });

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('管理订阅'),
      ),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          _buildCurrentSubscription(),
          const SizedBox(height: 24),
          const Text(
            '可切换的订阅',
            style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
          ),
          const SizedBox(height: 16),
          ...availableSubscriptions
              .where((p) => p.id != currentSubscription.id)
              .map((product) => _buildSubscriptionOption(context, product)),
        ],
      ),
    );
  }

  Widget _buildCurrentSubscription() {
    return Card(
      color: Colors.blue[50],
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text(
              '当前订阅',
              style: TextStyle(fontSize: 12, color: Colors.grey),
            ),
            const SizedBox(height: 8),
            Text(
              currentSubscription.shortTitle,
              style: const TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 4),
            Text('${currentSubscription.price}/${currentSubscription.periodDescription}'),
          ],
        ),
      ),
    );
  }

  Widget _buildSubscriptionOption(
    BuildContext context,
    SubscriptionProduct product,
  ) {
    final isUpgrade = product.rawPrice > currentSubscription.rawPrice;
    
    return Card(
      margin: const EdgeInsets.only(bottom: 12),
      child: ListTile(
        title: Text(product.shortTitle),
        subtitle: Text(
          '${product.price}/${product.periodDescription}',
        ),
        trailing: Chip(
          label: Text(
            isUpgrade ? '升级' : '降级',
            style: const TextStyle(fontSize: 12),
          ),
          backgroundColor: isUpgrade ? Colors.green[100] : Colors.grey[300],
        ),
        onTap: () {
          // 显示详情或直接切换
          _showChangeDialog(context, product);
        },
      ),
    );
  }

  void _showChangeDialog(BuildContext context, SubscriptionProduct newProduct) {
    // 实现切换对话框
  }
}
```

## 处理升降级后的状态

### 更新本地状态

```dart
class SubscriptionService {
  // ...

  Future<void> _handleSuccessfulPurchase(PurchaseDetails purchaseDetails) async {
    // 验证购买
    final valid = await _verifyPurchase(purchaseDetails);
    
    if (valid) {
      // 检查是否为升降级
      final isChange = await _isSubscriptionChange(purchaseDetails);
      
      if (isChange) {
        print('检测到订阅升降级');
        // 更新订阅状态
        await _updateSubscriptionChange(purchaseDetails);
      } else {
        print('新订阅');
        await _unlockSubscription(purchaseDetails);
      }
    }

    if (purchaseDetails.pendingCompletePurchase) {
      await _completePurchase(purchaseDetails);
    }
  }

  Future<bool> _isSubscriptionChange(PurchaseDetails purchaseDetails) async {
    // 检查是否已有活跃订阅
    final current = await _subscriptionManager.getCurrentSubscription();
    return current != null && current != purchaseDetails.productID;
  }

  Future<void> _updateSubscriptionChange(PurchaseDetails purchaseDetails) async {
    print('更新订阅: ${purchaseDetails.productID}');
    
    // 更新到新订阅
    await _subscriptionManager.activateSubscription(purchaseDetails.productID);
    
    // 通知 UI
    _notifySubscriptionChanged(purchaseDetails.productID);
  }
}
```

## 常见问题

### Q1：iOS 需要手动处理升降级吗？

**不需要**。只要产品在同一订阅组，iOS 会自动处理。

### Q2：Android 必须提供旧购买信息吗？

**是的**。Android 需要 `PurchaseDetails` 来标识要替换的订阅。

### Q3：降级会立即生效吗？

- **iOS**：否，当前周期结束后生效
- **Android**：取决于 `ProrationMode`，通常是周期结束后

### Q4：升级会退款吗？

**会**。升级时，旧订阅的剩余时间会按比例退款或抵扣。

### Q5：可以跨订阅组升降级吗？

**不可以**。不同订阅组的产品是独立的服务，不能升降级。

## 小结

在本章中，我们学习了：

- 订阅升降级的概念和重要性
- iOS 订阅组的自动处理机制
- Android 手动升降级的实现
- 不同替换模式的使用场景
- 如何实现跨平台的升降级功能

完成订阅管理功能后，下一步是处理订阅价格变更。

## 下一步

[第 13 章：处理订阅价格变更](13-price-changes.md) - 学习如何处理订阅价格调整时的用户确认流程。
