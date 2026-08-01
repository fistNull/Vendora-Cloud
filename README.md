
# 多租户商城平台商业矩阵

## 1. 项目定位

本项目定位为一套商业级的多租户商城 SaaS 平台，基于 `Spring Cloud + DDD + 微服务 + 事件驱动 + 配置化平台` 设计，支持租户快速搭建属于自己的商城，并支持租户在不改代码的前提下对店铺、页面、商品模型、营销玩法、渠道能力进行自由定义和修改。

平台核心目标如下：

- 支持多租户 SaaS 模式，兼容标准租户、小租户、大客户独立部署
- 支持完整电商交易闭环：商品、库存、订单、支付、履约、售后、结算
- 支持营销能力：促销、优惠券、积分、秒杀、会员权益
- 支持店铺快速搭建：模板化建店、页面装修、组件拖拽、商品模型扩展
- 支持渠道对接：第三方渠道商品同步、订单同步、库存同步、价格同步
- 支持开放平台：向第三方提供授权后的 Open API 调用能力
- 支持商业级稳定性：多租户隔离、限流熔断、幂等、审计、风控、监控、补偿

该平台本质上不是一个单纯商城系统，而是：

`交易中台 + 营销中台 + 店铺建站平台 + 多租户底座 + 渠道开放平台`

---

## 2. 总体架构原则

### 2.1 架构原则

- 采用 DDD 进行限界上下文划分，避免按表和接口做粗暴拆分
- 采用微服务架构，但保持领域边界先行，不为了拆服务而拆服务
- 单服务内部强一致，跨服务之间通过事件驱动实现最终一致
- 多租户隔离必须贯穿请求、权限、数据、缓存、消息、搜索、文件、监控全链路
- 营销规则、页面能力、商品模型、渠道能力采用配置化和元数据驱动设计
- 高并发活动如秒杀采用专门的并发模型，不混入普通交易模型
- 对外开放能力必须经过统一授权、审计、限流和签名校验

### 2.2 技术路线

- `Spring Boot 3.x`
- `Spring Cloud 2023`
- `Spring Cloud Alibaba`
- `Nacos`：服务注册、配置中心
- `Spring Cloud Gateway`：API 网关
- `OpenFeign`：同步 RPC
- `Sentinel`：限流、熔断、降级
- `RocketMQ`：领域事件、异步削峰、补偿驱动
- `MyBatis`
- `MySQL 8.x`
- `Redis`
- `Elasticsearch / OpenSearch`
- `MinIO / OSS / S3`
- `XXL-JOB`
- `Prometheus + Grafana + Loki + Tempo/Jaeger`

说明：

- `Seata` 不作为核心依赖，优先使用本地事务 + Outbox + 最终一致
- 秒杀等热点活动优先使用 `Redis + Lua + MQ + 限流`

---

## 3. 整体系统分层

平台整体分为五层。

### 3.1 接入层

- `gateway-service`
- `bff-admin-service`
- `bff-merchant-service`
- `bff-shop-service`
- `openapi-gateway-service`

职责：

- 多终端统一接入
- 鉴权与签名校验
- 租户解析
- 请求路由
- 灰度和限流
- Open API 访问控制

### 3.2 平台基础能力层

- `tenant-service`
- `iam-service`
- `config-service`
- `file-service`
- `message-service`
- `audit-service`
- `risk-service`
- `notification-service`

职责：

- 租户中心
- 身份权限
- 配置中心
- 文件管理
- 审计留痕
- 风控能力
- 短信、邮件、站内信等通知

### 3.3 核心业务域服务层

- `store-service`
- `product-service`
- `inventory-service`
- `pricing-service`
- `promotion-service`
- `coupon-service`
- `point-service`
- `member-service`
- `flashsale-service`
- `cart-service`
- `trade-service`
- `order-service`
- `payment-service`
- `fulfillment-service`
- `aftersale-service`
- `settlement-service`
- `search-service`
- `cms-service`

### 3.4 平台配置与扩展层

- `page-designer-service`
- `component-service`
- `meta-model-service`
- `rule-engine-service`
- `workflow-service`
- `channel-service`
- `open-platform-service`

职责：

- 店铺装修
- 页面模板与组件体系
- 自定义模型
- 规则引擎
- 审批流程
- 渠道适配
- 开放平台授权与 API 管理

### 3.5 基础设施层

- MySQL
- Redis
- RocketMQ
- Elasticsearch
- Object Storage
- Monitoring

---

## 4. DDD 限界上下文与微服务清单

以下服务按领域边界划分。

### 4.1 TenantContext - `tenant-service`

职责：

- 租户注册
- 套餐开通
- 功能开关
- 域名绑定
- 店铺初始化
- 租户状态管理

核心聚合：

- `Tenant`
- `TenantPlan`
- `TenantFeature`
- `TenantDomainBinding`

### 4.2 IdentityContext - `iam-service`

职责：

- 平台用户、租户管理员、门店员工、消费者账号
- RBAC 权限
- 组织和岗位
- Token 签发与校验

核心聚合：

- `Account`
- `User`
- `Role`
- `Permission`
- `OrgUnit`

### 4.3 StoreContext - `store-service`

职责：

- 店铺基础资料
- 店铺主题
- 店铺渠道配置
- 店铺模板初始化
- 商城站点绑定

核心聚合：

- `Store`
- `StoreTheme`
- `StoreChannelConfig`
- `StoreTemplate`

### 4.4 ProductContext - `product-service`

职责：

- SPU/SKU 管理
- 类目、品牌、属性
- 商品发布、上下架
- 商品审核

核心聚合：

- `Product`
- `Sku`
- `Category`
- `Brand`

### 4.5 MetaModelContext - `meta-model-service`

职责：

- 商品模型定义
- 自定义字段
- 动态表单模型
- 模型版本管理

核心聚合：

- `MetaModel`
- `MetaField`
- `MetaFieldGroup`
- `MetaSchemaVersion`

说明：

这是租户自由定义商品、店铺、内容、表单字段的核心服务。

### 4.6 InventoryContext - `inventory-service`

职责：

- 实时库存
- 锁库存
- 扣库存
- 回补库存
- 多仓库存

核心聚合：

- `InventoryItem`
- `StockReservation`
- `Warehouse`

### 4.7 PricingContext - `pricing-service`

职责：

- 基础价
- 渠道价
- 会员价
- 阶梯价
- 区域价

核心聚合：

- `PriceBook`
- `PriceRule`
- `PriceSnapshot`

### 4.8 PromotionContext - `promotion-service`

职责：

- 满减
- 满赠
- 折扣
- 套餐促销
- 优惠叠加规则
- 营销互斥规则

核心聚合：

- `PromotionActivity`
- `PromotionRule`
- `PromotionScope`
- `PromotionStackPolicy`

### 4.9 CouponContext - `coupon-service`

职责：

- 优惠券模板
- 发券
- 领券
- 券库存
- 核销
- 失效处理

核心聚合：

- `CouponTemplate`
- `CouponBatch`
- `UserCoupon`

### 4.10 PointContext - `point-service`

职责：

- 积分账户
- 积分发放
- 积分抵扣
- 积分冻结
- 积分流水
- 过期处理

核心聚合：

- `PointAccount`
- `PointLedger`
- `PointRule`

### 4.11 MemberContext - `member-service`

职责：

- 会员档案
- 会员等级
- 成长值
- 标签画像
- 会员权益

核心聚合：

- `Member`
- `MemberLevel`
- `GrowthAccount`
- `MemberTag`

### 4.12 FlashSaleContext - `flashsale-service`

职责：

- 秒杀活动
- 秒杀场次
- 秒杀商品与配额
- 排队令牌
- 限购控制
- 抢购资格

核心聚合：

- `FlashSaleActivity`
- `FlashSaleSession`
- `FlashSaleSku`
- `FlashSaleQuota`

说明：

秒杀服务必须独立，不建议混入通用促销服务。

### 4.13 CartContext - `cart-service`

职责：

- 购物车项管理
- 购物车预览
- 购物车试算上下文

核心聚合：

- `Cart`
- `CartItem`

### 4.14 TradeContext - `trade-service`

职责：

- 下单聚合编排
- 统一试算
- 汇总价格、促销、券、积分、库存结果
- 普通交易和活动交易下单入口编排

说明：

`trade-service` 不保存最终订单主数据，它是交易应用服务域，用于协调其他领域能力。

### 4.15 OrderContext - `order-service`

职责：

- 订单创建
- 订单状态机
- 订单拆单
- 订单取消
- 价格快照
- 商品快照

核心聚合：

- `Order`
- `OrderItem`
- `OrderPriceSnapshot`
- `OrderFulfillmentPlan`

### 4.16 PaymentContext - `payment-service`

职责：

- 支付单
- 渠道支付
- 回调处理
- 退款
- 对账

核心聚合：

- `PaymentOrder`
- `PaymentTransaction`
- `RefundOrder`

### 4.17 FulfillmentContext - `fulfillment-service`

职责：

- 发货
- 仓配
- 包裹
- 运费模板
- 物流轨迹

核心聚合：

- `Shipment`
- `Package`
- `DeliveryRule`

### 4.18 AfterSaleContext - `aftersale-service`

职责：

- 退款
- 退货退款
- 换货
- 售后审批
- 逆向物流

核心聚合：

- `AfterSaleOrder`
- `RefundRequest`
- `ReturnRequest`

### 4.19 SettlementContext - `settlement-service`

职责：

- 平台抽佣
- 商家分账
- 财务流水
- 结算单
- 对账单

核心聚合：

- `SettlementAccount`
- `SettlementBill`
- `LedgerEntry`

### 4.20 SearchContext - `search-service`

职责：

- 商品搜索
- 条件聚合
- 搜索索引同步

说明：

这是读模型服务，不作为交易事实来源。

### 4.21 CMS / DesignContext - `page-designer-service` + `component-service` + `cms-service`

职责：

- 店铺装修
- 页面模板
- 页面组件
- 专题页
- 首页和频道页配置

核心聚合：

- `PageDefinition`
- `PageTemplate`
- `PageComponent`
- `UiComponentDefinition`

### 4.22 RuleEngineContext - `rule-engine-service`

职责：

- 条件表达式
- 规则 DSL
- 活动规则执行
- 显示规则
- 审批规则

核心聚合：

- `RuleDefinition`
- `RuleScript`
- `RuleContext`

### 4.23 ChannelContext - `channel-service`

职责：

- 外部渠道对接
- 第三方平台店铺绑定
- 商品同步
- 库存同步
- 价格同步
- 订单同步
- 渠道映射关系

核心聚合：

- `ChannelAccount`
- `ChannelBinding`
- `ChannelProductMapping`
- `ChannelOrderMapping`
- `ChannelSyncTask`

说明：

该服务负责对接外部销售渠道，例如第三方电商平台、分销渠道、线下渠道系统、企业采购平台、内容电商平台等。

### 4.24 OpenPlatformContext - `open-platform-service`

职责：

- 第三方开发者接入
- 应用授权
- AccessKey / Secret 管理
- OAuth2 / 授权码模式
- API 权限范围控制
- 签名校验
- API 调用审计
- API 订阅和回调

核心聚合：

- `OpenApp`
- `OpenAuthorization`
- `ApiCredential`
- `ApiScope`
- `WebhookSubscription`

说明：

该服务用于向外部合作方提供授权后的 Open API 调用能力，是商城开放生态的核心。

---

## 5. 多租户设计方案

### 5.1 租户模式

采用混合多租户模式。

#### 模式 A：共享应用 + 独立 Schema / 独立库

适用：

- 标准 SaaS 商家
- 付费中型客户

优点：

- 隔离较强
- 运维可控
- 可按租户迁移

#### 模式 B：共享库共享表 + `tenant_id`

适用：

- 低成本套餐
- 小型租户

优点：

- 成本最低
- 资源利用率高

风险：

- 查询、索引、隔离难度更高

#### 模式 C：独立部署

适用：

- 大客户
- 私有化客户
- 高合规要求客户

### 5.2 多租户隔离要求

必须覆盖以下层面：

- 请求上下文隔离
- 权限隔离
- 数据隔离
- 缓存隔离
- 消息隔离
- 搜索隔离
- 文件隔离
- 配置隔离
- 日志隔离
- 监控隔离

### 5.3 租户上下文

统一上下文：

```java
public class TenantContext {
    private Long tenantId;
    private Long storeId;
    private Long userId;
    private String clientType;
    private String channelCode;
    private String traceId;
}
```

Gateway 负责解析租户：

- 域名
- Header
- 应用标识
- Open API 授权凭证

所有下游服务统一接收 `tenantId`。

### 5.4 存储隔离规范

数据库：

- 所有共享表必须有 `tenant_id`
- Repository 层强制注入租户条件

Redis Key：

- `tenant:{tenantId}:cart:{userId}`
- `tenant:{tenantId}:coupon:{couponId}`
- `tenant:{tenantId}:flashsale:{activityId}:stock`

MQ Header：

- `tenantId`
- `traceId`
- `eventId`
- `bizType`

ES：

- 共享索引时文档必须带 `tenantId`
- 大租户可独立索引

---

## 6. 支持租户快速搭建商城的核心设计

如果希望用户快速拥有自己的商城，并且能自由定义和修改，必须采用配置化与元数据驱动。

### 6.1 店铺模板化

租户可以直接选择行业模板：

- 标准零售商城
- 生鲜商城
- 本地生活商城
- 会员积分商城
- B2B 订货商城
- 内容电商商城

模板初始化内容：

- 店铺结构
- 首页模板
- 默认导航
- 默认商品模型
- 默认营销规则
- 默认支付与物流配置

### 6.2 页面装修化

页面使用 JSON DSL 驱动，后台提供拖拽式装修器。

示例：

```json
{
  "pageCode": "home",
  "title": "首页",
  "components": [
    {
      "type": "banner",
      "props": {
        "autoplay": true,
        "images": ["a.jpg", "b.jpg"]
      }
    },
    {
      "type": "flashSale",
      "props": {
        "activityId": 10001,
        "showCount": 4
      }
    },
    {
      "type": "couponCenter",
      "props": {
        "showCount": 6
      }
    }
  ]
}
```

前端根据 DSL 渲染页面，实现租户对页面自由修改。

### 6.3 商品模型化

不同租户可定义不同商品属性模型。

例如：

- 手机：CPU、内存、屏幕尺寸
- 生鲜：产地、保质期、储存条件
- 课程：课时、讲师、授课形式
- 服务：服务周期、预约规则、服务范围

这些能力由 `meta-model-service` 实现，不通过频繁改表和改代码完成。

### 6.4 营销规则配置化

营销中心支持规则配置，而不是完全写死：

- 满减条件
- 优惠叠加优先级
- 新会员首单优惠
- 指定商品可用券/不可用券
- 积分抵扣比例
- 秒杀与会员价是否互斥

通过规则引擎或者规则 DSL 实现。

### 6.5 组件中心

可复用组件包括：

- 轮播图
- 分类导航
- 商品列表
- 推荐商品
- 秒杀专区
- 优惠券中心
- 积分商城入口
- 会员权益卡片
- 专题活动页

这样租户可以组合自己的商城页面，而不需要单独开发前台。

---

## 7. 渠道对接架构

平台必须支持将商城能力向多渠道输出，也支持从外部渠道接入交易数据。

### 7.1 渠道对接目标

支持以下场景：

- 商城商品同步到第三方销售渠道
- 第三方渠道订单同步回商城
- 渠道库存、价格、上下架同步
- 渠道售后单回流
- 渠道营销活动映射
- 渠道账号统一管理

### 7.2 渠道对接设计

建议引入 `channel-service` 统一处理。

职责划分：

- 渠道账户管理
- 渠道路由
- 渠道适配器
- 映射关系维护
- 同步任务调度
- 对账与异常补偿

适配层模式：

```text
channel-service
 ├─ channel-core
 ├─ channel-adapter-jd
 ├─ channel-adapter-tiktok
 ├─ channel-adapter-wechat
 ├─ channel-adapter-distributor
 └─ channel-adapter-custom
```

采用适配器模式：

```java
public interface ChannelAdapter {
    void syncProduct(ChannelProductSyncCommand command);
    void syncInventory(ChannelInventorySyncCommand command);
    void syncPrice(ChannelPriceSyncCommand command);
    ChannelOrderDTO queryOrder(String channelOrderNo);
    void pushOrder(ChannelOrderPushCommand command);
}
```

### 7.3 渠道映射模型

必须维护映射关系：

- 平台商品 ID <-> 渠道商品 ID
- 平台 SKU ID <-> 渠道 SKU ID
- 平台订单号 <-> 渠道订单号
- 平台仓库 <-> 渠道仓库
- 平台店铺 <-> 渠道店铺

### 7.4 渠道同步方式

- 实时同步：库存、价格、上下架
- 异步批量同步：商品资料、类目映射
- 定时拉取：订单、售后、物流状态
- Webhook 推送：渠道事件回调

### 7.5 渠道集成要求

- 渠道调用必须可重试
- 渠道异常必须可补偿
- 渠道数据必须有同步状态
- 每个渠道必须单独限流
- 渠道凭证必须租户隔离存储

---

## 8. 开放平台与授权 API 设计

平台要支持向第三方合作伙伴、ISV、ERP、WMS、供应链系统、分销商等提供授权后的 API 调用能力。

### 8.1 Open API 目标

允许第三方在获得租户授权后调用商城 API，例如：

- 查询商品
- 创建商品
- 查询库存
- 更新库存
- 查询订单
- 发货
- 创建优惠券
- 查询会员信息
- 创建售后单

### 8.2 开放平台架构

设计：

- `openapi-gateway-service`
- `open-platform-service`

`openapi-gateway-service` 负责：

- 请求签名校验
- Access Token 校验
- API 限流
- API 路由
- API 版本控制

`open-platform-service` 负责：

- 应用管理
- 授权管理
- API Scope 管理
- 秘钥管理
- 回调订阅
- API 调用日志

### 8.3 授权模型

支持两类模式：

#### 模式 A：应用授权

适用于第三方系统接入。

流程：

1. 第三方注册应用
2. 平台为应用分配 `appKey/appSecret`
3. 租户在后台授权该应用访问其商城数据
4. 平台签发 Access Token
5. 第三方按 Scope 调用 API

#### 模式 B：OAuth2 授权码模式

适用于第三方 SaaS 插件市场、生态合作伙伴。

流程：

1. 第三方跳转授权页
2. 租户确认授权范围
3. 平台返回授权码
4. 第三方换取 Access Token
5. 按权限范围调用 API



### 8.6 Open API 安全要求

- 必须按应用和租户双维度授权
- Token 必须有失效时间
- 高风险 API 必须审计
- 高风险 API 支持 IP 白名单
- API 调用必须限流
- 回调地址必须校验
- 回调必须支持重试和签名验签

### 8.7 Webhook 设计

支持事件回调：

- `order.created`
- `order.paid`
- `order.shipped`
- `order.completed`
- `inventory.changed`
- `coupon.used`
- `aftersale.created`

Webhook 必须支持：

- 重试
- 签名
- 事件去重
- 失败告警

---

## 9. 促销、积分、优惠券、秒杀设计

营销能力必须拆分建模，不建议做成一个大营销服务。

### 9.1 职责划分

- `pricing-service`：基础价、渠道价、会员价
- `promotion-service`：满减、折扣、套餐等活动
- `coupon-service`：券模板、领券、用券、核销
- `point-service`：积分账务、积分抵扣、积分发放
- `flashsale-service`：秒杀活动、资格、配额、令牌
- `trade-service`：统一试算和交易编排

### 9.2 试算顺序建议

1. 计算基础价格
2. 校验秒杀资格与秒杀价
3. 计算普通促销优惠
4. 计算优惠券优惠
5. 计算积分抵扣
6. 计算运费
7. 输出最终应付金额

### 9.3 优惠叠加策略

必须支持：

- 互斥关系
- 优先级关系
- 可叠加上限
- 指定商品排除
- 指定用户群排除

统一输出价格明细：

- `TradePriceDetail`
- `DiscountLine`
- `PromotionAdjustment`
- `CouponAdjustment`
- `PointAdjustment`

订单域必须保存完整优惠快照。

### 9.4 秒杀专项设计

秒杀采用专门并发模型：

- 活动库存独立
- Redis 原子扣减
- 资格校验
- 令牌发放
- MQ 异步削峰
- 超时回补
- 限购校验

秒杀服务与普通促销服务隔离，避免复杂促销逻辑拖垮高并发链路。

---

## 10. 核心业务链路

### 10.1 普通下单链路

```text
用户提交下单
 -> trade-service 接收交易请求
 -> pricing-service 计算基础价
 -> promotion-service 计算活动优惠
 -> coupon-service 校验和试算优惠券
 -> point-service 计算积分抵扣
 -> inventory-service 锁库存
 -> order-service 创建订单与快照
 -> payment-service 创建支付单
 -> 发布 OrderCreated 事件
```

### 10.2 秒杀下单链路

```text
用户进入秒杀页
 -> flashsale-service 校验资格
 -> Redis 预扣活动库存
 -> 发放秒杀令牌
 -> 用户提交订单
 -> trade-service 校验令牌
 -> order-service 创建秒杀订单
 -> payment-service 创建支付单
 -> 超时未支付自动回补库存
```

### 10.3 支付成功链路

```text
支付渠道回调
 -> payment-service 幂等更新支付状态
 -> 发布 PaymentSucceeded 事件
 -> order-service 更新订单为已支付
 -> fulfillment-service 创建发货任务
 -> point-service 发放积分/成长值
 -> settlement-service 记录待结算流水
```

### 10.4 渠道订单回流链路

```text
第三方渠道产生订单
 -> channel-service 拉取或接收回调
 -> 渠道订单转换为平台统一订单模型
 -> trade-service / order-service 落单
 -> payment-service 或结算服务登记渠道支付信息
 -> fulfillment-service 继续履约
```

---

## 11. 一致性设计

### 11.1 原则

- 单服务内本地事务强一致
- 跨服务使用最终一致

### 11.2 推荐方案

- Outbox 本地消息表
- 可靠事件投递
- 消费幂等
- 死信队列
- 重试补偿
- 定时校正任务

### 11.3 必须幂等的场景

- 下单
- 支付回调
- 优惠券核销
- 积分扣减
- 库存锁定与回补
- 秒杀资格发放
- 渠道订单同步
- Open API 写操作

---

## 12. 商业级非功能要求

### 12.1 幂等

每个关键写操作必须支持幂等键。

### 12.2 审计

必须记录：

- 价格变更
- 活动变更
- 券模板变更
- 积分规则变更
- 库存调整
- 售后审批
- API 调用日志
- 渠道同步操作

### 12.3 风控

- 防刷券
- 防刷积分
- 防恶意秒杀
- IP / 设备 / 用户限流
- API 滥用防护

### 12.4 可观测性

- traceId 全链路追踪
- tenant 维度监控
- 渠道维度监控
- Open API 维度监控
- 秒杀活动维度监控

### 12.5 灰度发布

- 按租户灰度
- 按渠道灰度
- 按功能开关灰度

### 12.6 安全

- 敏感数据脱敏
- 凭证加密存储
- API 签名校验
- Token 过期管理
- 接口白名单和黑名单

---

## 13. Spring Cloud 项目结构



```text
mall-platform
├─ mall-dependencies
├─ mall-common
│  ├─ common-core
│  ├─ common-web
│  ├─ common-tenant
│  ├─ common-security
│  ├─ common-redis
│  ├─ common-mq
│  ├─ common-ddd
│  └─ common-openapi
├─ mall-gateway
│  ├─ gateway-service
│  └─ openapi-gateway-service
├─ mall-platform-services
│  ├─ tenant-service
│  ├─ iam-service
│  ├─ config-service
│  ├─ audit-service
│  ├─ risk-service
│  └─ file-service
├─ mall-business-services
│  ├─ store-service
│  ├─ product-service
│  ├─ inventory-service
│  ├─ pricing-service
│  ├─ promotion-service
│  ├─ coupon-service
│  ├─ point-service
│  ├─ member-service
│  ├─ flashsale-service
│  ├─ cart-service
│  ├─ trade-service
│  ├─ order-service
│  ├─ payment-service
│  ├─ fulfillment-service
│  ├─ aftersale-service
│  ├─ settlement-service
│  ├─ search-service
│  └─ cms-service
├─ mall-ext-services
│  ├─ page-designer-service
│  ├─ component-service
│  ├─ meta-model-service
│  ├─ rule-engine-service
│  ├─ workflow-service
│  ├─ channel-service
│  └─ open-platform-service
└─ docs
```

---

## 14. 服务内部 DDD 分层规范

每个服务内部采用统一结构：

```text
order-service
├─ interfaces
│  ├─ controller
│  ├─ dto
│  └─ assembler
├─ application
│  ├─ command
│  ├─ query
│  ├─ service
│  └─ handler
├─ domain
│  ├─ model
│  │  ├─ aggregate
│  │  ├─ entity
│  │  └─ valueobject
│  ├─ service
│  ├─ repository
│  ├─ event
│  └─ policy
├─ infrastructure
│  ├─ persistence
│  ├─ mq
│  ├─ rpc
│  ├─ cache
│  └─ config
└─ starter
```

职责划分：

- `interfaces`：接口入参、鉴权、DTO 转换
- `application`：用例编排
- `domain`：领域规则和不变量
- `infrastructure`：技术实现
- 
总结

- 架构采用 `Spring Cloud + DDD + 微服务 + 事件驱动`
- 多租户采用混合模式，标准租户独立库/Schema，大客户独立部署
- 电商交易主链路由 `trade-service` 聚合编排，`order-service` 专注订单域
- 促销、优惠券、积分、秒杀分域治理，避免营销服务臃肿失控
- 通过 `store-service + page-designer-service + component-service + meta-model-service + rule-engine-service` 实现租户快速搭建和自由定义商城
- 通过 `channel-service` 实现渠道对接和多渠道同步
- 通过 `open-platform-service + openapi-gateway-service` 实现授权后的开放 API 调用能力
- 整个平台必须内建多租户隔离、幂等、审计、风控、可观测性和补偿机制

