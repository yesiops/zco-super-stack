# C03_Team_Topologies - 团队拓扑

## 1. 主题定位

### 1.1 定义与范围

团队拓扑（Team Topologies）是由Matthew Skelton和Manuel Pais提出的组织设计框架，提供四种基本团队类型和三种交互模式，帮助组织优化团队结构和协作方式。

### 1.2 业务价值

- 明确团队职责边界
- 减少跨团队依赖
- 提升交付效率
- 优化认知负载
- 支持规模化敏捷

### 1.3 适用场景

| 场景 | 适用性 |
|-----|-------|
| 团队扩张 | ⭐⭐⭐⭐⭐ |
| 组织变革 | ⭐⭐⭐⭐⭐ |
| 平台战略 | ⭐⭐⭐⭐⭐ |
| 微服务架构 | ⭐⭐⭐⭐⭐ |

## 2. 核心概念

### 2.1 四种团队类型

```
团队拓扑：

┌─────────────────────────────────────────────────────────────┐
│  Stream-aligned Team                                        │
│  面向流团队 - 核心业务价值交付                               │
├─────────────────────────────────────────────────────────────┤
│  Platform Team                                              │
│  平台团队 - 提供自助服务基础设施                             │
├─────────────────────────────────────────────────────────────┤
│  Complicated Subsystem Team                                 │
│  复杂子系统团队 - 处理专业领域复杂性                         │
├─────────────────────────────────────────────────────────────┤
│  Enabling Team                                              │
│  赋能团队 - 帮助其他团队提升能力                             │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 三种交互模式

| 模式 | 说明 | 使用场景 |
|-----|-----|---------|
| **协作** | 团队紧密合作 | 探索新领域 |
| **服务** | 一方提供服务 | 清晰的依赖关系 |
| **促进** | 赋能团队帮助 | 能力转移 |

## 3. 技术实践

### 3.1 平台团队API设计

```yaml
# platform-api.yaml - 平台团队服务定义
openapi: 3.0.0
info:
  title: 内部开发者平台API
  version: 1.0.0
  description: 为面向流团队提供自助服务接口

paths:
  /v1/environments:
    post:
      summary: 创建新环境
      tags:
        - 环境管理
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/EnvironmentRequest'
      responses:
        '201':
          description: 环境创建成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Environment'
    
    get:
      summary: 列出所有环境
      tags:
        - 环境管理
      responses:
        '200':
          description: 环境列表
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Environment'

  /v1/databases:
    post:
      summary: 创建数据库实例
      tags:
        - 数据服务
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/DatabaseRequest'
      responses:
        '201':
          description: 数据库创建成功

components:
  schemas:
    EnvironmentRequest:
      type: object
      required:
        - name
        - team
        - type
      properties:
        name:
          type: string
          pattern: '^[a-z][a-z0-9-]*$'
        team:
          type: string
          description: 所属团队
        type:
          type: string
          enum: [development, staging, production]
    
    Environment:
      allOf:
        - $ref: '#/components/schemas/EnvironmentRequest'
        - type: object
          properties:
            id:
              type: string
            status:
              type: string
              enum: [provisioning, ready, failed]
            endpoints:
              type: object
              properties:
                api:
                  type: string
                console:
                  type: string
```

### 3.2 团队认知负载评估

```python
#!/usr/bin/env python3
"""
团队认知负载评估工具
基于Team Topologies的认知负载理论
"""

from dataclasses import dataclass, field
from typing import List, Dict
from enum import Enum


class LoadType(Enum):
    """认知负载类型"""
    INTRINSIC = "内在负载"  # 领域本身的复杂性
    EXTRANEOUS = "外在负载"  # 不必要的复杂性
    GERMANE = "相关负载"    # 有助于学习的负载


@dataclass
class CognitiveLoadItem:
    """认知负载项"""
    name: str
    load_type: LoadType
    complexity: int  # 1-10
    frequency: str  # daily, weekly, monthly, rare
    ownership: str  # owned, shared, external


@dataclass
class TeamAssessment:
    """团队评估"""
    team_name: str
    team_type: str  # stream-aligned, platform, etc.
    items: List[CognitiveLoadItem] = field(default_factory=list)
    
    def calculate_total_load(self) -> Dict:
        """计算总负载"""
        scores = {
            LoadType.INTRINSIC: 0,
            LoadType.EXTRANEOUS: 0,
            LoadType.GERMANE: 0
        }
        
        freq_multiplier = {
            'daily': 3,
            'weekly': 2,
            'monthly': 1,
            'rare': 0.5
        }
        
        ownership_multiplier = {
            'owned': 1.0,
            'shared': 0.7,
            'external': 0.3
        }
        
        for item in self.items:
            score = (item.complexity * 
                    freq_multiplier.get(item.frequency, 1) *
                    ownership_multiplier.get(item.ownership, 1))
            scores[item.load_type] += score
        
        return scores
    
    def get_recommendations(self) -> List[str]:
        """生成优化建议"""
        scores = self.calculate_total_load()
        recommendations = []
        
        total = sum(scores.values())
        
        # 检查外在负载
        extraneous_ratio = scores[LoadType.EXTRANEOUS] / total if total else 0
        if extraneous_ratio > 0.3:
            recommendations.append(
                "外在负载过高（{:.0%}），建议简化工具链或流程".format(extraneous_ratio)
            )
        
        # 检查负载分布
        owned_items = [i for i in self.items if i.ownership == 'owned']
        if len(owned_items) > 10:
            recommendations.append(
                f"团队拥有{len(owned_items)}个领域，建议考虑拆分团队"
            )
        
        return recommendations
    
    def generate_report(self) -> str:
        """生成评估报告"""
        scores = self.calculate_total_load()
        total = sum(scores.values())
        
        report = f"""
# 团队认知负载评估报告

## 团队信息
- 名称: {self.team_name}
- 类型: {self.team_type}
- 评估项数: {len(self.items)}

## 负载分析
| 类型 | 分数 | 占比 |
|-----|-----|-----|
| 内在负载 | {scores[LoadType.INTRINSIC]:.1f} | {scores[LoadType.INTRINSIC]/total:.1%} |
| 外在负载 | {scores[LoadType.EXTRANEOUS]:.1f} | {scores[LoadType.EXTRANEOUS]/total:.1%} |
| 相关负载 | {scores[LoadType.GERMANE]:.1f} | {scores[LoadType.GERMANE]/total:.1%} |
| **总计** | **{total:.1f}** | 100% |

## 优化建议
"""
        for rec in self.get_recommendations():
            report += f"- {rec}\\n"
        
        return report


# 使用示例
if __name__ == "__main__":
    assessment = TeamAssessment(
        team_name="订单服务团队",
        team_type="stream-aligned"
    )
    
    # 添加评估项
    assessment.items = [
        CognitiveLoadItem("订单处理", LoadType.INTRINSIC, 8, "daily", "owned"),
        CognitiveLoadItem("支付集成", LoadType.INTRINSIC, 7, "daily", "owned"),
        CognitiveLoadItem("CI/CD维护", LoadType.EXTRANEOUS, 5, "weekly", "owned"),
        CognitiveLoadItem("监控配置", LoadType.EXTRANEOUS, 4, "weekly", "shared"),
        CognitiveLoadItem("领域知识学习", LoadType.GERMANE, 6, "weekly", "owned"),
    ]
    
    print(assessment.generate_report())
```

## 4. 资源索引

| 资源 | 链接 |
|-----|------|
| Team Topologies书籍 | https://teamtopologies.com |
| 官方培训 | https://teamtopologies.com/academy |
| 社区资源 | https://github.com/TeamTopologies |
| 相关论文 | https://queue.acm.org/detail.cfm?id=3380777 |

## 5. 关联知识

- C01_Remote_Workflows
- C02_Knowledge_Sharing
- B01_SDLC_Frameworks

## 6. 学习建议

1. 阅读《Team Topologies》
2. 评估团队认知负载
3. 识别团队交互反模式
4. 实践Thinnest Viable Platform

---
*最后更新：2024年*
