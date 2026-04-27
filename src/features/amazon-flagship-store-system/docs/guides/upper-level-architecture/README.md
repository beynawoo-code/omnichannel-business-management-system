---
title: 亚马逊旗舰店子系统 · 上层架构 README（边界宣言）
owner: "@beynawoo-code"
date: 2026-04-22
status: v0.1 · Draft
audience: 业务方（旗舰店渠道推广 / 旗舰店运营 / 数据分析师 / 业务负责人）、架构师、研发 leader
applies_to:
  - src/features/amazon-flagship-store-system/**
related:
  - ./flagship-store_capability_architecture.md
  - ../../../../../../docs/knowledge-base/business-research-materials/amazon-flagship-store/
  - ../../../../../../docs/knowledge-base/business-research-meeting/亚马逊旗舰店需求讨论会20251223/
  - ../../../../../../docs/knowledge-base/methodology/上层架构与竞品调研-工作范式.md
  - ../../../../../../docs/knowledge-base/methodology/数字化项目全生命周期-软件产品经理工作范式.md
---

# 亚马逊旗舰店子系统 · 上层架构 README

> **文档日期**：2026-04-22

本目录承载亚马逊旗舰店（Brand Store）数字化子系统的**上层架构三件套**：

```
upper-level-architecture/
├── flagship-store_capability_architecture.md   ← L1 业务能力地图（本批已交付）
├── flagship-store_capability_architecture.html ← L1 可视化（HTML，待 MD 稳定后导出）
├── flagship-store_menu_v1.md                   ← L2 菜单 + 角色矩阵（下一批交付）
├── flagship-store_menu_v1.html                 ← L2 可视化
├── flagship-store_system_architecture.html     ← L3 系统架构图
└── README.md                                   ← 本文件（边界宣言 + 索引）
```

> 三件套关系遵循 `docs/knowledge-base/methodology/上层架构与竞品调研-工作范式.md` §1：**上一层是下一层的「宪法」**。L2 不能出现 L1 没有的能力；L3 不能解决 L2 不存在的菜单。

---

## 1. 边界宣言（Boundary Declaration）

> **本子系统只做亚马逊 Brand Store 引流与转化的多维分析、Tag 与联盟的全生命周期管理；不做广告投放执行、不做订单履约、不做品牌注册。**

### 1.1 做什么（In-Scope）

- **Brand Store 旗舰店运营**：站点 / 场景 / 页面三级结构维护，重点品（Hero SKU）标记，目标管理。
- **引流归因（Attribution）**：在 ERP 内自助创建 Amazon Attribution Tag，按统一命名规范生成；归因数据回流分析。
- **Amazon Live 直播**：直播排期登记，亚马逊官方直播数据接入与分析（API 接入排期 P1）。
- **联盟（Affiliate）**：多平台源数据导入（CSV/Excel）、ASIN→SKU 映射、佣金 / ROAS / 预算进度自动计算。
- **多维数据看板**：站点总览、场景维度、场景-页面、流量组成、品线维度、重点品、Top SKU、旗舰店占比。
- **复盘资产沉淀**：周/月报，命名规范资产，方法论文档。

### 1.2 不做什么（Out-of-Scope）

| 业务诉求 | 归口子系统 | 与本系统的关系 |
|---|---|---|
| SP/SB/SD 广告投放与托管 | `amazon-ads-system` | 本系统消费其投放结果作为「SB 渠道」流量来源 |
| 站外付费媒体投放（Paid social/Google ads/DSP） | `off-site-ads-system` | 本系统消费其归因 Tag 作为「Other 渠道」来源 |
| 订单履约、库存、退换货 | `amazon-sales-system` / `delivery-system` | 占比指标依赖其销售明细 |
| 全渠道经营管理与 GTM 战略 | `omnichannel-business-management-system` | 本系统是 OC 的执行层数据源之一 |
| 品牌注册、ASIN 上架、Listing 维护 | 现有亚马逊运营流程 | 不在本系统范围 |
| 客服、合规、认证 | `customer-service-system` / `certification-system` | 不在本系统范围 |

### 1.3 解耦原则（与 §`.globalrules` §1 对齐）

- 与其他子系统**严禁深层引用**；跨域数据（ASIN-SKU 主数据、销售明细、广告投放结果）必须经 `src/domain/` 交换。
- 本系统对外暴露的能力，仅通过 `index.ts` 与 `shared/api/` 显式导出。

---

## 2. 用户角色（User Personas）

| 角色 | 代号 | 在本系统的位置 | 核心使用场景 |
|---|:---:|---|---|
| 旗舰店渠道推广 | **CP** (Channel Promotion) | 主用户 | Tag 自助创建、联盟管理、看板分析、复盘 |
| 旗舰店运营 | **SO** (Store Ops) | 主用户 | 页面/场景维护、重点品标记、目标维护 |
| 数据分析师 | **DA** | 协同用户 | 看板查阅、指标口径维护、自定义透视 |
| 业务负责人 | **BO** (Business Owner) | 汇报视角 | 站点总览、目标达成、跨站点占比 |

> 角色与 50+ 职责点的映射详见 `docs/knowledge-base/business-research-materials/amazon-flagship-store/渠道推广角色价值点识别.xlsx`，将在 L2 菜单设计时落到角色矩阵（⭐️/🔵/⚪/—）。

---

## 3. 数据源边界（4 类）

| 数据源 | 来源方式 | 归属流量类型 | MVP 是否纳入 |
|---|---|---|---|
| Brand Store 旗舰店本身 | Amazon SP-API / 后台导出（待评估） | 站内 | ✅ P0 |
| Attribution Tag | Amazon Attribution（自助创建 + 数据回流） | 站外（Other） | ✅ P0 |
| Amazon Live 直播 | 亚马逊后台手工拉表 → API（P1） | 站内 | ⚠️ P1（API 评估后） |
| 联盟（Affiliate） | 多平台 CSV/Excel 导入（26 年新增 1 家至 3 家） | 站外（Other） | ✅ P0 |

---

## 4. MVP 范围（P0 / P1）

### P0（首期上线）

1. 旗舰店多维数据看板（站点 / 场景 / 页面 / 流量组成 / 品线 / 重点品 / Top SKU 共 7 大看板）
2. Attribution Tag 同比补全 + Tag 自助创建（强约束命名规范）
3. 联盟数据导入 + 佣金 / ROAS / 预算进度看板（支持插件化新增平台）
4. 后台维护：站点-场景-页面三级映射、佣金比例配置、重点品标记、目标维护

### P1（二期）

5. 旗舰店占比指标（依赖跨子系统销售明细打通）
6. Amazon Live 直播数据 API 接入与分析

---

## 5. 关键风险登记（Risk Register）

| # | 风险 | 影响 | 缓解策略 |
|---|---|---|---|
| R1 | Live API 可行性未知 | Live 模块 MVP 不可达 | 与 SP-API 团队拉可行性评估；MVP 先做手工数据兜底 |
| R2 | 跨系统占比指标 | P1 能否落地依赖销售域数据 | 把 ASIN-SKU 主数据 + 全店销售明细规划到 `src/domain/data/` |
| R3 | 页面变更场景导致历史数据归属错乱 | 环比/同比口径混乱 | 设计「映射版本快照」机制，按月固化映射 |
| R4 | Tag 命名规范执行不到位 | 数据脏，分析失真 | 创建即规范化（系统强约束），不允许 Excel 自起名 |
| R5 | 联盟平台异构 | 26 年新增平台 → 字段差异 | 导入器插件化：每平台一份字段映射配置 |
| R6 | 比率指标聚合错误 | 看板算错（CVR/ASP/UPT/NTS%） | 数据层定义统一指标计算服务，强制 `sum/sum` 重算 |

---

## 6. 文档版本

| 版本 | 日期 | 说明 |
|---|---|---|
| v0.1 | 2026-04-22 | 首版：边界宣言 + 用户角色 + 数据源 + MVP 范围 + 风险登记，配套 L1 能力地图 |

---

作者：@beynawoo-code
