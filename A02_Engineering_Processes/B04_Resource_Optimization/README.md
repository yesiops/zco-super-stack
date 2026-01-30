# B04 Resource Optimization

**所属领域**: [A02_Engineering_Processes](../readme.md)
**创建日期**: 2026-01-30
**最后更新**: 2026-01-30

## 📋 子领域定位

资源优化关注如何高效利用人力、技术和财务资源，以最大化工程团队的产出价值。在云计算时代，资源优化不仅涉及团队容量规划，还包括云成本控制、基础设施利用率和开发效能度量。

本领域涵盖容量规划（团队规模、技能矩阵、工作量预测）、效能度量（DORA 指标、SPACE 框架、流指标）和成本管理（云成本优化、FinOps 实践、预算管理）三大方向。

## 🗂️ 专项列表

### [C01. Capacity_Planning](C01_Capacity_Planning/README.md)

容量规划确保团队具备交付价值所需的能力。本专项详解团队能力评估、技能矩阵建设、工作量估算技术（故事点、理想天数）、以及基于速率的迭代规划。涵盖团队拓扑设计、康威定律应用和组织扩展策略。

### [C02. Performance_Metrics](C02_Performance_Metrics/README.md)

效能度量帮助团队理解并改进开发流程。本专项覆盖 DORA 四项关键指标（部署频率、变更前置时间、变更失败率、恢复时间）、SPACE 框架（满意度与福祉、绩效、活动、沟通与协作、效率与流程）、以及流效率和价值流分析。

### [C03. Cost_Management](C03_Cost_Management/README.md)

成本管理确保技术投入产生业务价值。本专项详解云成本优化策略（预留实例、Spot 实例、自动伸缩）、FinOps 实践（成本分摊、预算告警、单位经济学）、以及技术 ROI 分析。涵盖成本可视化工具和成本优化自动化。

## 🛠️ 技术栈概览

### 效能度量工具

| 工具 | 功能 | 官网 |
|------|------|------|
| **Apache DevLake** | 开源效能平台 | https://devlake.apache.org |
| **LinearB** | 工程智能 | https://linearb.io |
| **Swarmia** | 团队效能 | https://swarmia.com |
| **Allstacks** | 价值流分析 | https://allstacks.com |
| **Pluralsight Flow** | 工程分析 | https://www.pluralsight.com/product/flow |

### 成本管理工具

| 工具 | 功能 | 官网 |
|------|------|------|
| **Kubecost** | K8s 成本分析 | https://www.kubecost.com |
| **CloudHealth** | 多云成本管理 | https://www.cloudhealthtech.com |
| **Infracost** | IaC 成本估算 | https://www.infracost.io |
| **Ternary** | FinOps 平台 | https://ternary.app |

## 💼 实践案例

### DORA 指标基准

| 指标 | 精英表现 | 高表现 | 中等表现 | 低表现 |
|------|----------|--------|----------|--------|
| 部署频率 | 按需（每日多次） | 每日一次至每周一次 | 每周一次至每月一次 | 每月一次至每半年一次 |
| 变更前置时间 | < 1小时 | < 1周 | 1周至1个月 | 1至6个月 |
| 变更失败率 | < 5% | < 15% | < 30% | > 30% |
| 恢复时间 | < 1小时 | < 1天 | < 1周 | 1周至1个月 |

## 🔗 知识关联

- 与 [B01 SDLC Frameworks](../B01_SDLC_Frameworks/README.md) 关联：流程效率影响资源利用
- 与 [B05 Collaboration Models](../B05_Collaboration_Models/README.md) 关联：协作模式影响团队产能

## 📖 学习资源

- 《Accelerate》- Nicole Forsgren (DORA 研究)
- 《Team Topologies》- Matthew Skelton
- 《Flow Framework》- Mik Kersten
- DevLake: https://devlake.apache.org
- DORA: https://dora.dev

## 🔄 维护说明

- 每季度更新工具链和基准数据
- 跟踪 DORA 年度报告更新
