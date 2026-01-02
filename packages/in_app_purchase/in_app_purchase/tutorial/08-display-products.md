# 第 8 章：展示订阅产品

## 本章目标

- 设计美观的订阅产品展示界面
- 突出显示价格和优惠信息
- 展示试用期和优惠
- 实现响应式布局

## 订阅产品界面设计原则

### 关键信息层级

订阅产品展示应该清晰传达以下信息（按重要性排序）：

1. **价格**：最显眼的元素
2. **订阅周期**：每月/每年
3. **产品名称**：标识订阅等级
4. **功能说明**：订阅包含的功能
5. **优惠信息**：试用期、折扣等
6. **购买按钮**：清晰的行动号召

### UI/UX 最佳实践

- ✅ 使用卡片式设计突出每个订阅选项
- ✅ 高亮推荐的订阅计划
- ✅ 显示年度订阅的节省金额
- ✅ 使用清晰的颜色和图标
- ✅ 提供横向或纵向滚动（产品较多时）

## 步骤 1：创建基础产品卡片

### 简单卡片设计

```dart
import 'package:flutter/material.dart';
import '../../../../models/subscription_product.dart';

class ProductCard extends StatelessWidget {
  final SubscriptionProduct product;
  final VoidCallback onTap;
  final bool isRecommended;

  const ProductCard({
    super.key,
    required this.product,
    required this.onTap,
    this.isRecommended = false,
  });

  @override
  Widget build(BuildContext context) {
    return Card(
      elevation: isRecommended ? 8 : 2,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(16),
        side: isRecommended
            ? const BorderSide(color: Colors.blue, width: 2)
            : BorderSide.none,
      ),
      child: InkWell(
        onTap: onTap,
        borderRadius: BorderRadius.circular(16),
        child: Padding(
          padding: const EdgeInsets.all(20),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              _buildHeader(),
              const SizedBox(height: 16),
              _buildPrice(),
              const SizedBox(height: 12),
              _buildDescription(),
              const SizedBox(height: 16),
              _buildButton(),
            ],
          ),
        ),
      ),
    );
  }

  Widget _buildHeader() {
    return Row(
      mainAxisAlignment: MainAxisAlignment.spaceBetween,
      children: [
        Text(
          product.shortTitle,
          style: const TextStyle(
            fontSize: 20,
            fontWeight: FontWeight.bold,
          ),
        ),
        if (isRecommended)
          Container(
            padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 6),
            decoration: BoxDecoration(
              color: Colors.blue,
              borderRadius: BorderRadius.circular(12),
            ),
            child: const Text(
              '推荐',
              style: TextStyle(
                color: Colors.white,
                fontSize: 12,
                fontWeight: FontWeight.bold,
              ),
            ),
          ),
      ],
    );
  }

  Widget _buildPrice() {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text(
          product.price,
          style: const TextStyle(
            fontSize: 32,
            fontWeight: FontWeight.bold,
            color: Colors.blue,
          ),
        ),
        Text(
          product.periodDescription,
          style: TextStyle(
            fontSize: 14,
            color: Colors.grey[600],
          ),
        ),
      ],
    );
  }

  Widget _buildDescription() {
    return Text(
      product.description,
      style: TextStyle(
        fontSize: 14,
        color: Colors.grey[700],
      ),
      maxLines: 3,
      overflow: TextOverflow.ellipsis,
    );
  }

  Widget _buildButton() {
    return SizedBox(
      width: double.infinity,
      child: ElevatedButton(
        onPressed: onTap,
        style: ElevatedButton.styleFrom(
          padding: const EdgeInsets.symmetric(vertical: 16),
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(12),
          ),
        ),
        child: const Text(
          '立即订阅',
          style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
        ),
      ),
    );
  }
}
```

## 步骤 2：添加试用期和优惠显示

### 扩展产品模型

首先扩展 `SubscriptionProduct` 类以支持优惠信息：

```dart
import 'dart:io';
import 'package:in_app_purchase/in_app_purchase.dart';
import 'package:in_app_purchase_storekit/in_app_purchase_storekit.dart';
import 'package:in_app_purchase_android/in_app_purchase_android.dart';

class SubscriptionProduct {
  final ProductDetails productDetails;

  const SubscriptionProduct(this.productDetails);

  // ... 基本属性 ...

  // ==================== 优惠信息 ====================
  
  /// 是否有试用期
  bool get hasFreeTrial {
    if (Platform.isIOS) {
      return _getIOSFreeTrial() != null;
    } else if (Platform.isAndroid) {
      return _getAndroidFreeTrial() != null;
    }
    return false;
  }

  /// 获取试用期描述
  String? get freeTrialDescription {
    if (Platform.isIOS) {
      return _getIOSFreeTrial();
    } else if (Platform.isAndroid) {
      return _getAndroidFreeTrial();
    }
    return null;
  }

  /// 是否有介绍性优惠
  bool get hasIntroductoryOffer {
    if (Platform.isIOS) {
      return _getIOSIntroPrice() != null;
    } else if (Platform.isAndroid) {
      return _getAndroidIntroPrice() != null;
    }
    return false;
  }

  /// 获取介绍性优惠描述
  String? get introductoryOfferDescription {
    if (Platform.isIOS) {
      return _getIOSIntroPrice();
    } else if (Platform.isAndroid) {
      return _getAndroidIntroPrice();
    }
    return null;
  }

  // ==================== iOS 特定 ====================
  
  String? _getIOSFreeTrial() {
    if (productDetails is AppStoreProductDetails) {
      final skProduct = (productDetails as AppStoreProductDetails).skProduct;
      final intro = skProduct.introductoryPrice;
      
      if (intro != null && intro.price == '0') {
        final period = _formatIOSPeriod(intro.subscriptionPeriod);
        return '$period 免费试用';
      }
    }
    return null;
  }

  String? _getIOSIntroPrice() {
    if (productDetails is AppStoreProductDetails) {
      final skProduct = (productDetails as AppStoreProductDetails).skProduct;
      final intro = skProduct.introductoryPrice;
      
      if (intro != null && intro.price != '0') {
        final period = _formatIOSPeriod(intro.subscriptionPeriod);
        return '首${period}仅需 ${intro.price}';
      }
    }
    return null;
  }

  String _formatIOSPeriod(SKSubscriptionPeriodWrapper? period) {
    if (period == null) return '';
    
    final unit = period.unit;
    final num = period.numberOfUnits;
    
    switch (unit) {
      case SKSubscriptionPeriodUnit.day:
        return '$num 天';
      case SKSubscriptionPeriodUnit.week:
        return '$num 周';
      case SKSubscriptionPeriodUnit.month:
        return '$num 个月';
      case SKSubscriptionPeriodUnit.year:
        return '$num 年';
      default:
        return '';
    }
  }

  // ==================== Android 特定 ====================
  
  String? _getAndroidFreeTrial() {
    if (productDetails is GooglePlayProductDetails) {
      final details = (productDetails as GooglePlayProductDetails).productDetails;
      
      if (details.subscriptionOfferDetails != null) {
        for (final offer in details.subscriptionOfferDetails!) {
          final phases = offer.pricingPhases;
          if (phases.isNotEmpty) {
            final firstPhase = phases.first;
            if (firstPhase.priceAmountMicros == 0) {
              final period = _formatAndroidPeriod(firstPhase.billingPeriod);
              return '$period 免费试用';
            }
          }
        }
      }
    }
    return null;
  }

  String? _getAndroidIntroPrice() {
    if (productDetails is GooglePlayProductDetails) {
      final details = (productDetails as GooglePlayProductDetails).productDetails;
      
      if (details.subscriptionOfferDetails != null) {
        for (final offer in details.subscriptionOfferDetails!) {
          final phases = offer.pricingPhases;
          if (phases.length > 1) {
            final firstPhase = phases.first;
            if (firstPhase.priceAmountMicros > 0) {
              final period = _formatAndroidPeriod(firstPhase.billingPeriod);
              return '首$period仅需 ${firstPhase.formattedPrice}';
            }
          }
        }
      }
    }
    return null;
  }

  String _formatAndroidPeriod(String period) {
    // period 格式：P1M (1 month), P1Y (1 year), P7D (7 days)
    if (period.startsWith('P')) {
      final value = period.substring(1, period.length - 1);
      final unit = period.substring(period.length - 1);
      
      switch (unit) {
        case 'D':
          return '$value 天';
        case 'W':
          return '$value 周';
        case 'M':
          return '$value 个月';
        case 'Y':
          return '$value 年';
      }
    }
    return period;
  }
}
```

### 显示优惠信息的卡片

```dart
class ProductCardWithOffer extends StatelessWidget {
  final SubscriptionProduct product;
  final VoidCallback onTap;
  final bool isRecommended;

  const ProductCardWithOffer({
    super.key,
    required this.product,
    required this.onTap,
    this.isRecommended = false,
  });

  @override
  Widget build(BuildContext context) {
    return Card(
      elevation: isRecommended ? 8 : 2,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(16),
        side: isRecommended
            ? const BorderSide(color: Colors.blue, width: 2)
            : BorderSide.none,
      ),
      child: InkWell(
        onTap: onTap,
        borderRadius: BorderRadius.circular(16),
        child: Stack(
          children: [
            Padding(
              padding: const EdgeInsets.all(20),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  _buildHeader(),
                  const SizedBox(height: 8),
                  if (product.hasFreeTrial || product.hasIntroductoryOffer)
                    _buildOfferBadge(),
                  const SizedBox(height: 16),
                  _buildPrice(),
                  const SizedBox(height: 12),
                  _buildDescription(),
                  const SizedBox(height: 16),
                  _buildFeaturesList(),
                  const SizedBox(height: 16),
                  _buildButton(),
                ],
              ),
            ),
            if (isRecommended)
              Positioned(
                top: 0,
                right: 0,
                child: _buildRecommendedBadge(),
              ),
          ],
        ),
      ),
    );
  }

  Widget _buildHeader() {
    return Text(
      product.shortTitle,
      style: const TextStyle(
        fontSize: 22,
        fontWeight: FontWeight.bold,
      ),
    );
  }

  Widget _buildOfferBadge() {
    String offerText = '';
    
    if (product.hasFreeTrial) {
      offerText = product.freeTrialDescription!;
    } else if (product.hasIntroductoryOffer) {
      offerText = product.introductoryOfferDescription!;
    }

    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 6),
      decoration: BoxDecoration(
        color: Colors.orange[100],
        borderRadius: BorderRadius.circular(8),
        border: Border.all(color: Colors.orange, width: 1),
      ),
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: [
          const Icon(Icons.local_offer, size: 16, color: Colors.orange),
          const SizedBox(width: 6),
          Text(
            offerText,
            style: const TextStyle(
              color: Colors.orange,
              fontSize: 13,
              fontWeight: FontWeight.bold,
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildPrice() {
    return Row(
      crossAxisAlignment: CrossAxisAlignment.end,
      children: [
        Text(
          product.price,
          style: const TextStyle(
            fontSize: 36,
            fontWeight: FontWeight.bold,
            color: Colors.blue,
          ),
        ),
        const SizedBox(width: 8),
        Padding(
          padding: const EdgeInsets.only(bottom: 6),
          child: Text(
            '/ ${product.periodDescription}',
            style: TextStyle(
              fontSize: 16,
              color: Colors.grey[600],
            ),
          ),
        ),
      ],
    );
  }

  Widget _buildDescription() {
    return Text(
      product.description,
      style: TextStyle(
        fontSize: 14,
        color: Colors.grey[700],
      ),
    );
  }

  Widget _buildFeaturesList() {
    final features = _getFeatures();
    
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: features.map((feature) {
        return Padding(
          padding: const EdgeInsets.only(bottom: 8),
          child: Row(
            children: [
              const Icon(Icons.check_circle, size: 20, color: Colors.green),
              const SizedBox(width: 8),
              Expanded(
                child: Text(
                  feature,
                  style: const TextStyle(fontSize: 14),
                ),
              ),
            ],
          ),
        );
      }).toList(),
    );
  }

  List<String> _getFeatures() {
    // 根据产品 ID 返回不同的功能列表
    if (product.id.contains('basic')) {
      return ['基础功能', '标准支持', '移除广告'];
    } else if (product.id.contains('premium')) {
      return ['所有基础功能', '高级功能解锁', '优先支持', '云同步'];
    } else if (product.id.contains('vip')) {
      return ['所有高级功能', 'VIP 专属功能', '24/7 专属支持', '无限云存储'];
    }
    return [];
  }

  Widget _buildButton() {
    return SizedBox(
      width: double.infinity,
      child: ElevatedButton(
        onPressed: onTap,
        style: ElevatedButton.styleFrom(
          padding: const EdgeInsets.symmetric(vertical: 16),
          backgroundColor: isRecommended ? Colors.blue : null,
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(12),
          ),
        ),
        child: Text(
          product.hasFreeTrial ? '开始免费试用' : '立即订阅',
          style: const TextStyle(
            fontSize: 16,
            fontWeight: FontWeight.bold,
          ),
        ),
      ),
    );
  }

  Widget _buildRecommendedBadge() {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
      decoration: const BoxDecoration(
        color: Colors.blue,
        borderRadius: BorderRadius.only(
          topRight: Radius.circular(16),
          bottomLeft: Radius.circular(16),
        ),
      ),
      child: const Text(
        '最受欢迎',
        style: TextStyle(
          color: Colors.white,
          fontSize: 12,
          fontWeight: FontWeight.bold,
        ),
      ),
    );
  }
}
```

## 步骤 3：创建产品列表页面

### 垂直列表布局

```dart
import 'package:flutter/material.dart';
import '../../../../services/subscription_service.dart';
import '../../../../models/subscription_product.dart';
import '../../../../widgets/product_card.dart';

class SubscriptionListScreen extends StatefulWidget {
  const SubscriptionListScreen({super.key});

  @override
  State<SubscriptionListScreen> createState() => _SubscriptionListScreenState();
}

class _SubscriptionListScreenState extends State<SubscriptionListScreen> {
  final SubscriptionService _service = SubscriptionService();
  bool _isLoading = true;

  @override
  void initState() {
    super.initState();
    _loadProducts();
  }

  Future<void> _loadProducts() async {
    setState(() => _isLoading = true);
    await _service.loadProducts();
    setState(() => _isLoading = false);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('选择订阅计划'),
        elevation: 0,
      ),
      body: _isLoading
          ? const Center(child: CircularProgressIndicator())
          : _buildProductList(),
    );
  }

  Widget _buildProductList() {
    final products = _service.getProductsSortedByPrice(ascending: true);

    if (products.isEmpty) {
      return const Center(child: Text('暂无可用的订阅产品'));
    }

    return ListView.builder(
      padding: const EdgeInsets.all(16),
      itemCount: products.length,
      itemBuilder: (context, index) {
        final product = products[index];
        final isRecommended = index == 1; // 推荐中间价位的产品

        return Padding(
          padding: const EdgeInsets.only(bottom: 16),
          child: ProductCardWithOffer(
            product: product,
            isRecommended: isRecommended,
            onTap: () => _handlePurchase(product),
          ),
        );
      },
    );
  }

  void _handlePurchase(SubscriptionProduct product) {
    // 将在第 9 章实现
    print('购买: ${product.id}');
  }
}
```

### 横向滚动布局

```dart
class SubscriptionHorizontalScreen extends StatefulWidget {
  const SubscriptionHorizontalScreen({super.key});

  @override
  State<SubscriptionHorizontalScreen> createState() =>
      _SubscriptionHorizontalScreenState();
}

class _SubscriptionHorizontalScreenState
    extends State<SubscriptionHorizontalScreen> {
  final SubscriptionService _service = SubscriptionService();
  bool _isLoading = true;

  @override
  void initState() {
    super.initState();
    _loadProducts();
  }

  Future<void> _loadProducts() async {
    setState(() => _isLoading = true);
    await _service.loadProducts();
    setState(() => _isLoading = false);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('选择订阅计划'),
        elevation: 0,
      ),
      body: _isLoading
          ? const Center(child: CircularProgressIndicator())
          : _buildHorizontalList(),
    );
  }

  Widget _buildHorizontalList() {
    final products = _service.getProductsSortedByPrice();

    if (products.isEmpty) {
      return const Center(child: Text('暂无可用的订阅产品'));
    }

    return Column(
      children: [
        const Padding(
          padding: EdgeInsets.all(16),
          child: Text(
            '选择最适合您的订阅计划',
            style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
          ),
        ),
        Expanded(
          child: PageView.builder(
            itemCount: products.length,
            controller: PageController(viewportFraction: 0.85),
            itemBuilder: (context, index) {
              final product = products[index];
              final isRecommended = index == 1;

              return Padding(
                padding: const EdgeInsets.symmetric(horizontal: 8),
                child: ProductCardWithOffer(
                  product: product,
                  isRecommended: isRecommended,
                  onTap: () => _handlePurchase(product),
                ),
              );
            },
          ),
        ),
        const SizedBox(height: 16),
      ],
    );
  }

  void _handlePurchase(SubscriptionProduct product) {
    print('购买: ${product.id}');
  }
}
```

## 步骤 4：添加对比表格

对于多个订阅选项，可以提供功能对比表：

```dart
class SubscriptionComparisonScreen extends StatelessWidget {
  final List<SubscriptionProduct> products;

  const SubscriptionComparisonScreen({
    super.key,
    required this.products,
  });

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('对比订阅计划'),
      ),
      body: SingleChildScrollView(
        scrollDirection: Axis.horizontal,
        child: SingleChildScrollView(
          child: _buildComparisonTable(),
        ),
      ),
    );
  }

  Widget _buildComparisonTable() {
    return DataTable(
      columns: [
        const DataColumn(label: Text('功能')),
        ...products.map((p) => DataColumn(
          label: Text(
            p.shortTitle,
            style: const TextStyle(fontWeight: FontWeight.bold),
          ),
        )),
      ],
      rows: [
        _buildPriceRow(),
        _buildFeatureRow('基础功能', [true, true, true]),
        _buildFeatureRow('高级功能', [false, true, true]),
        _buildFeatureRow('VIP 功能', [false, false, true]),
        _buildFeatureRow('优先支持', [false, true, true]),
        _buildFeatureRow('云同步', [false, true, true]),
        _buildFeatureRow('无限存储', [false, false, true]),
      ],
    );
  }

  DataRow _buildPriceRow() {
    return DataRow(
      cells: [
        const DataCell(Text('价格', style: TextStyle(fontWeight: FontWeight.bold))),
        ...products.map((p) => DataCell(
          Text(
            p.price,
            style: const TextStyle(
              color: Colors.blue,
              fontWeight: FontWeight.bold,
            ),
          ),
        )),
      ],
    );
  }

  DataRow _buildFeatureRow(String feature, List<bool> availability) {
    return DataRow(
      cells: [
        DataCell(Text(feature)),
        ...availability.map((available) => DataCell(
          available
              ? const Icon(Icons.check, color: Colors.green)
              : const Icon(Icons.close, color: Colors.grey),
        )),
      ],
    );
  }
}
```

## 步骤 5：添加动画和交互

### 选中效果

```dart
class SelectableProductCard extends StatefulWidget {
  final SubscriptionProduct product;
  final bool isSelected;
  final VoidCallback onTap;

  const SelectableProductCard({
    super.key,
    required this.product,
    required this.isSelected,
    required this.onTap,
  });

  @override
  State<SelectableProductCard> createState() => _SelectableProductCardState();
}

class _SelectableProductCardState extends State<SelectableProductCard>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _scaleAnimation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(milliseconds: 200),
      vsync: this,
    );
    _scaleAnimation = Tween<double>(begin: 1.0, end: 0.95).animate(
      CurvedAnimation(parent: _controller, curve: Curves.easeInOut),
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTapDown: (_) => _controller.forward(),
      onTapUp: (_) {
        _controller.reverse();
        widget.onTap();
      },
      onTapCancel: () => _controller.reverse(),
      child: ScaleTransition(
        scale: _scaleAnimation,
        child: AnimatedContainer(
          duration: const Duration(milliseconds: 200),
          decoration: BoxDecoration(
            borderRadius: BorderRadius.circular(16),
            border: Border.all(
              color: widget.isSelected ? Colors.blue : Colors.grey[300]!,
              width: widget.isSelected ? 3 : 1,
            ),
            boxShadow: widget.isSelected
                ? [
                    BoxShadow(
                      color: Colors.blue.withOpacity(0.3),
                      blurRadius: 10,
                      spreadRadius: 2,
                    ),
                  ]
                : [],
          ),
          child: _buildContent(),
        ),
      ),
    );
  }

  Widget _buildContent() {
    // ... 产品卡片内容 ...
    return Container();
  }
}
```

## 步骤 6：计算和显示节省金额

对于年度订阅，显示相比月度订阅能节省多少：

```dart
class SubscriptionProduct {
  // ... 现有代码 ...

  /// 计算相对于月度订阅的节省金额（仅年度订阅）
  String? calculateSavings(List<SubscriptionProduct> allProducts) {
    if (!isYearly) return null;

    // 查找对应的月度订阅
    final monthlyVersion = allProducts.firstWhere(
      (p) => p.id.replaceAll('yearly', 'monthly') == id.replaceAll('yearly', 'monthly'),
      orElse: () => this,
    );

    if (monthlyVersion.isYearly) return null;

    // 计算年度总价 vs 月度 x 12
    final yearlyTotal = rawPrice;
    final monthlyTotal = monthlyVersion.rawPrice * 12;
    final savings = monthlyTotal - yearlyTotal;

    if (savings <= 0) return null;

    final percentage = ((savings / monthlyTotal) * 100).toInt();
    return '相比月度订阅节省 $currencySymbol${savings.toStringAsFixed(0)} ($percentage%)';
  }
}
```

在 UI 中显示：

```dart
if (product.isYearly) {
  final savings = product.calculateSavings(_service.products);
  if (savings != null) {
    Container(
      padding: const EdgeInsets.all(8),
      decoration: BoxDecoration(
        color: Colors.green[100],
        borderRadius: BorderRadius.circular(8),
      ),
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: [
          const Icon(Icons.savings, size: 16, color: Colors.green),
          const SizedBox(width: 6),
          Text(
            savings,
            style: const TextStyle(
              color: Colors.green,
              fontSize: 12,
              fontWeight: FontWeight.bold,
            ),
          ),
        ],
      ),
    );
  }
}
```

## 常见问题

### Q1：如何处理产品名称过长？

使用 `Text` 的 `overflow` 和 `maxLines` 属性：

```dart
Text(
  product.title,
  maxLines: 2,
  overflow: TextOverflow.ellipsis,
)
```

### Q2：如何适配不同屏幕尺寸？

使用 `MediaQuery` 和响应式设计：

```dart
Widget build(BuildContext context) {
  final screenWidth = MediaQuery.of(context).size.width;
  final isLargeScreen = screenWidth > 600;
  
  return isLargeScreen
      ? _buildGridLayout()
      : _buildListLayout();
}
```

### Q3：产品排序的最佳实践？

一般按照：低价 → 推荐 → 高价，或者将推荐的放在最前面。

## 小结

在本章中，我们学习了：

- 订阅产品界面的设计原则
- 如何创建美观的产品卡片
- 如何展示试用期和优惠信息
- 如何实现横向和纵向布局
- 如何添加功能对比和节省计算

完成产品展示后，下一步是实现购买功能。

## 下一步

[第 9 章：发起订阅购买](09-purchase-subscription.md) - 学习如何实现订阅购买流程。
