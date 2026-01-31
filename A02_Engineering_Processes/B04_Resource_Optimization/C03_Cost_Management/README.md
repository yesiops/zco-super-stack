# C03_Cost_Management - 成本管理

## 1. 主题定位

### 1.1 定义与范围

成本管理（Cost Management）是优化云资源支出、提高IT投资回报率的过程。本知识单元涵盖云成本可见性、优化策略、预算管理、成本分摊等内容，帮助组织实现FinOps实践。

### 1.2 业务价值

- 降低云支出
- 提高资源效率
- 预算可控
- 成本透明
- 业务价值最大化

### 1.3 适用场景

| 场景 | 适用性 |
|-----|-------|
| 多云环境 | ⭐⭐⭐⭐⭐ |
| 云成本优化 | ⭐⭐⭐⭐⭐ |
| 预算规划 | ⭐⭐⭐⭐⭐ |
| 成本分摊 | ⭐⭐⭐⭐⭐ |

## 2. 核心概念

### 2.1 FinOps生命周期

```
FinOps生命周期：

通知 ──► 优化 ──► 运营
 │        │        │
 │        ▼        │
 │    自动优化     │
 │        │        │
 └────────┴────────┘
```

### 2.2 成本优化策略

| 策略 | 节省潜力 | 实施难度 |
|-----|---------|---------|
| 预留实例 | 30-70% | 低 |
| Spot实例 | 60-90% | 中 |
| 自动扩缩容 | 20-40% | 低 |
| 存储分层 | 40-60% | 低 |
| 无服务器 | 20-50% | 中 |
| 资源标记 | 5-15% | 低 |

## 3. 技术实践

### 3.1 云成本分析工具

```python
#!/usr/bin/env python3
"""
云成本分析与优化工具
"""

import boto3
from datetime import datetime, timedelta
from typing import Dict, List
from dataclasses import dataclass
import json


@dataclass
class CostBreakdown:
    service: str
    amount: float
    percentage: float
    trend: float  # 环比变化


class AWSCostAnalyzer:
    """AWS成本分析器"""
    
    def __init__(self):
        self.ce_client = boto3.client('ce')
    
    def get_monthly_cost(self, months: int = 3) -> Dict:
        """获取月度成本"""
        end_date = datetime.now()
        start_date = end_date - timedelta(days=30*months)
        
        response = self.ce_client.get_cost_and_usage(
            TimePeriod={
                'Start': start_date.strftime('%Y-%m-%d'),
                'End': end_date.strftime('%Y-%m-%d')
            },
            Granularity='MONTHLY',
            Metrics=['BlendedCost', 'UnblendedCost', 'UsageQuantity'],
            GroupBy=[
                {'Type': 'DIMENSION', 'Key': 'SERVICE'}
            ]
        )
        
        results = {}
        for result in response['ResultsByTime']:
            month = result['TimePeriod']['Start'][:7]
            results[month] = {
                'total': float(result['Total']['BlendedCost']['Amount']),
                'services': {
                    group['Keys'][0]: float(group['Metrics']['BlendedCost']['Amount'])
                    for group in result['Groups']
                }
            }
        
        return results
    
    def get_cost_by_tag(self, tag_key: str) -> Dict:
        """按标签分析成本"""
        end_date = datetime.now()
        start_date = end_date - timedelta(days=30)
        
        response = self.ce_client.get_cost_and_usage(
            TimePeriod={
                'Start': start_date.strftime('%Y-%m-%d'),
                'End': end_date.strftime('%Y-%m-%d')
            },
            Granularity='MONTHLY',
            Metrics=['BlendedCost'],
            GroupBy=[
                {'Type': 'TAG', 'Key': tag_key}
            ]
        )
        
        return {
            group['Keys'][0]: float(group['Metrics']['BlendedCost']['Amount'])
            for group in response['ResultsByTime'][0]['Groups']
        }
    
    def get_reservation_recommendations(self) -> List[Dict]:
        """获取预留实例建议"""
        response = self.ce_client.get_reservation_purchase_recommendation(
            Service='Amazon Elastic Compute Cloud - Compute',
            AccountScope='PAYER',
            LookbackPeriodInDays=7,
            TermInYears='ONE_YEAR',
            PaymentOption='PARTIAL_UPFRONT'
        )
        
        recommendations = []
        for rec in response.get('Recommendations', []):
            for detail in rec.get('RecommendationDetails', []):
                recommendations.append({
                    'instance_type': detail['InstanceDetails']['EC2InstanceDetails']['InstanceType'],
                    'region': detail['InstanceDetails']['EC2InstanceDetails']['Region'],
                    'estimated_savings': float(detail['EstimatedMonthlySavingsAmount']),
                    'upfront_cost': float(detail['UpfrontCost']),
                    'recurring_cost': float(detail['RecurringStandardMonthlyCost'])
                })
        
        return recommendations
    
    def identify_unused_resources(self) -> List[Dict]:
        """识别未使用资源"""
        unused = []
        
        # 检查未关联的EBS卷
        ec2 = boto3.client('ec2')
        volumes = ec2.describe_volumes(
            Filters=[{'Name': 'status', 'Values': ['available']}]
        )
        
        for vol in volumes['Volumes']:
            unused.append({
                'type': 'EBS Volume',
                'id': vol['VolumeId'],
                'size_gb': vol['Size'],
                'monthly_cost': vol['Size'] * 0.1,  # gp2约$0.1/GB/月
                'reason': '未挂载'
            })
        
        # 检查未使用的弹性IP
        addresses = ec2.describe_addresses()
        for addr in addresses['Addresses']:
            if 'InstanceId' not in addr:
                unused.append({
                    'type': 'Elastic IP',
                    'id': addr['AllocationId'],
                    'monthly_cost': 3.6,  # 未使用EIP约$3.6/月
                    'reason': '未关联实例'
                })
        
        return unused


class CostOptimizer:
    """成本优化建议生成器"""
    
    def __init__(self, analyzer: AWSCostAnalyzer):
        self.analyzer = analyzer
    
    def generate_recommendations(self) -> List[Dict]:
        """生成优化建议"""
        recommendations = []
        
        # RI建议
        ri_recs = self.analyzer.get_reservation_recommendations()
        for rec in ri_recs[:5]:  # 前5个
            recommendations.append({
                'category': '预留实例',
                'priority': 'high',
                'description': f"购买 {rec['instance_type']} 预留实例",
                'estimated_monthly_savings': rec['estimated_savings'],
                'implementation': 'low'
            })
        
        # 未使用资源
        unused = self.analyzer.identify_unused_resources()
        total_unused_cost = sum(r['monthly_cost'] for r in unused)
        if total_unused_cost > 0:
            recommendations.append({
                'category': '资源清理',
                'priority': 'high',
                'description': f"清理 {len(unused)} 个未使用资源",
                'estimated_monthly_savings': total_unused_cost,
                'implementation': 'low',
                'details': unused
            })
        
        return recommendations
    
    def generate_budget_alert(self, budget_amount: float) -> Dict:
        """生成预算告警配置"""
        return {
            "BudgetName": "Monthly-Budget",
            "BudgetLimit": {
                "Amount": str(budget_amount),
                "Unit": "USD"
            },
            "TimeUnit": "MONTHLY",
            "BudgetType": "COST",
            "CostTypes": {
                "IncludeTax": True,
                "IncludeSubscription": True
            },
            "NotificationsWithSubscribers": [
                {
                    "Notification": {
                        "NotificationType": "ACTUAL",
                        "ComparisonOperator": "GREATER_THAN",
                        "Threshold": 80
                    },
                    "Subscribers": [
                        {
                            "SubscriptionType": "SNS",
                            "Address": "arn:aws:sns:us-east-1:123456789:alerts"
                        }
                    ]
                }
            ]
        }


# 使用示例
if __name__ == "__main__":
    analyzer = AWSCostAnalyzer()
    
    # 获取成本分析
    costs = analyzer.get_monthly_cost(months=3)
    print("月度成本:", json.dumps(costs, indent=2))
    
    # 生成优化建议
    optimizer = CostOptimizer(analyzer)
    recommendations = optimizer.generate_recommendations()
    print("\n优化建议:")
    for rec in recommendations:
        print(f"- {rec['description']}: 节省 ${rec['estimated_monthly_savings']:.2f}/月")
```

### 3.2 Kubernetes成本分配

```yaml
# kubecost配置
apiVersion: v1
kind: Namespace
metadata:
  name: kubecost
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kubecost
  namespace: kubecost
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubecost-cost-analyzer
  namespace: kubecost
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cost-analyzer
  template:
    metadata:
      labels:
        app: cost-analyzer
    spec:
      serviceAccountName: kubecost
      containers:
        - name: cost-model
          image: kubecost/cost-model:latest
          env:
            - name: PROMETHEUS_SERVER_ENDPOINT
              value: "http://prometheus.monitoring.svc:9090"
            - name: CLUSTER_ID
              value: "production-cluster"
          resources:
            requests:
              cpu: 200m
              memory: 512Mi
            limits:
              cpu: 1000m
              memory: 1Gi
---
apiVersion: v1
kind: Service
metadata:
  name: kubecost-cost-analyzer
  namespace: kubecost
spec:
  type: ClusterIP
  ports:
    - port: 9090
      targetPort: 9090
  selector:
    app: cost-analyzer
```

## 4. 资源索引

| 工具 | 用途 | 官网 |
|-----|-----|------|
| Kubecost | K8s成本 | https://kubecost.com |
| CloudHealth | 多云成本 | https://cloudhealthtech.com |
| ProsperOps | AWS优化 | https://prosperops.com |
| Infracost | IaC成本 | https://infracost.io |
| FinOps Foundation | 最佳实践 | https://finops.org |

## 5. 关联知识

- C01_Capacity_Planning
- C02_Performance_Metrics
- B02_Technical_Practices

## 6. 学习建议

1. 学习FinOps框架
2. 掌握成本分配方法
3. 了解云定价模型
4. 建立成本意识

---
*最后更新：2024年*
