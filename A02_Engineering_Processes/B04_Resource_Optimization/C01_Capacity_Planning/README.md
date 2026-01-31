# C01_Capacity_Planning - 容量规划

## 1. 主题定位

### 1.1 定义与范围

容量规划（Capacity Planning）是预测和确保IT资源满足当前和未来业务需求的过程。本知识单元涵盖工作负载预测、资源建模、自动扩缩容、成本优化等内容，确保系统在性能与成本之间取得平衡。

### 1.2 业务价值

- 避免服务中断
- 优化资源成本
- 支持业务增长
- 提升用户体验
- 提高资源利用率

### 1.3 适用场景

| 场景 | 适用性 |
|-----|-------|
| 云服务成本优化 | ⭐⭐⭐⭐⭐ |
| 峰值流量应对 | ⭐⭐⭐⭐⭐ |
| 长期资源投资 | ⭐⭐⭐⭐⭐ |
| 多云架构 | ⭐⭐⭐⭐⭐ |

## 2. 核心概念

### 2.1 容量规划层次

```
容量规划层次：

战略层（1-3年）
├── 业务增长预测
├── 基础设施投资
└── 技术路线规划

战术层（3-12月）
├── 季度容量审查
├── 预算分配
└── 供应商管理

操作层（1-4周）
├── Sprint容量
├── 自动扩缩容
└── 资源调度
```

### 2.2 关键指标

| 指标 | 说明 | 目标值 |
|-----|-----|-------|
| CPU利用率 | 平均CPU使用 | 40-70% |
| 内存利用率 | 平均内存使用 | 60-80% |
| 磁盘I/O | 读写吞吐量 | <80% |
| 网络带宽 | 网络使用率 | <70% |
| 响应时间 | P99延迟 | <500ms |
| 错误率 | 5xx错误比例 | <0.1% |

## 3. 技术实践

### 3.1 容量预测模型

```python
#!/usr/bin/env python3
"""
容量预测与规划工具
"""

import numpy as np
import pandas as pd
from datetime import datetime, timedelta
from typing import List, Dict, Tuple
from dataclasses import dataclass


@dataclass
class CapacityMetrics:
    timestamp: datetime
    cpu_percent: float
    memory_percent: float
    request_count: int
    response_time_ms: float


class CapacityPlanner:
    """容量规划器"""
    
    def __init__(self, historical_data: List[CapacityMetrics]):
        self.data = pd.DataFrame([
            {
                'timestamp': m.timestamp,
                'cpu': m.cpu_percent,
                'memory': m.memory_percent,
                'requests': m.request_count,
                'latency': m.response_time_ms
            }
            for m in historical_data
        ])
        self.data.set_index('timestamp', inplace=True)
    
    def predict_growth(self, days_ahead: int = 90) -> Dict:
        """预测未来增长"""
        # 简单线性回归预测
        from sklearn.linear_model import LinearRegression
        
        # 准备数据
        daily_avg = self.data.resample('D').mean()
        X = np.arange(len(daily_avg)).reshape(-1, 1)
        
        predictions = {}
        for metric in ['cpu', 'memory', 'requests']:
            y = daily_avg[metric].values
            model = LinearRegression()
            model.fit(X, y)
            
            # 预测未来
            future_X = np.arange(len(daily_avg), 
                               len(daily_avg) + days_ahead).reshape(-1, 1)
            future_y = model.predict(future_X)
            
            predictions[metric] = {
                'current': y[-1],
                'predicted_30d': future_y[29],
                'predicted_90d': future_y[-1],
                'growth_rate': model.coef_[0]
            }
        
        return predictions
    
    def calculate_headroom(self, target_utilization: float = 70) -> Dict:
        """计算容量余量"""
        current = {
            'cpu': self.data['cpu'].mean(),
            'memory': self.data['memory'].mean()
        }
        
        headroom = {}
        for resource, usage in current.items():
            headroom[resource] = {
                'current_usage': usage,
                'target_usage': target_utilization,
                'headroom_percent': max(0, target_utilization - usage),
                'scale_up_needed': usage > target_utilization
            }
        
        return headroom
    
    def recommend_instance_count(self, 
                                 current_instances: int,
                                 peak_rps: float,
                                 target_rps_per_instance: float = 1000) -> Dict:
        """推荐实例数量"""
        # 考虑20%缓冲
        required_instances = int(np.ceil(
            peak_rps / target_rps_per_instance * 1.2
        ))
        
        return {
            'current': current_instances,
            'recommended': max(required_instances, 2),  # 最少2个
            'change': max(required_instances, 2) - current_instances,
            'peak_rps': peak_rps,
            'capacity_per_instance': target_rps_per_instance
        }


class AutoScalingConfig:
    """自动扩缩容配置"""
    
    def __init__(self):
        self.min_instances = 2
        self.max_instances = 20
        self.target_cpu = 60
        self.scale_up_threshold = 70
        self.scale_down_threshold = 30
        self.scale_up_cooldown = 300  # 5分钟
        self.scale_down_cooldown = 600  # 10分钟
    
    def to_terraform(self) -> str:
        """生成Terraform配置"""
        return f"""
resource "aws_autoscaling_group" "app" {{
  name                = "app-asg"
  min_size            = {self.min_instances}
  max_size            = {self.max_instances}
  desired_capacity    = {self.min_instances}
  
  tag {{
    key                 = "Name"
    value               = "app-instance"
    propagate_at_launch = true
  }}
}}

resource "aws_autoscaling_policy" "scale_up" {{
  name                   = "scale-up"
  scaling_adjustment     = 1
  adjustment_type        = "ChangeInCapacity"
  cooldown               = {self.scale_up_cooldown}
  autoscaling_group_name = aws_autoscaling_group.app.name
}}

resource "aws_cloudwatch_metric_alarm" "high_cpu" {{
  alarm_name          = "high-cpu"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = "2"
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = "60"
  statistic           = "Average"
  threshold           = "{self.scale_up_threshold}"
  
  dimensions = {{
    AutoScalingGroupName = aws_autoscaling_group.app.name
  }}
  
  alarm_actions = [aws_autoscaling_policy.scale_up.arn]
}}
"""


# 使用示例
if __name__ == "__main__":
    # 生成模拟数据
    base_time = datetime.now() - timedelta(days=30)
    metrics = []
    
    for i in range(30 * 24):  # 30天，每小时
        metrics.append(CapacityMetrics(
            timestamp=base_time + timedelta(hours=i),
            cpu_percent=40 + (i / 720) * 20 + np.random.normal(0, 5),
            memory_percent=50 + (i / 720) * 15 + np.random.normal(0, 3),
            request_count=int(1000 + (i / 720) * 500),
            response_time_ms=100 + np.random.normal(0, 10)
        ))
    
    planner = CapacityPlanner(metrics)
    
    # 生成预测
    predictions = planner.predict_growth(days_ahead=90)
    print("增长预测:", predictions)
    
    # 计算余量
    headroom = planner.calculate_headroom()
    print("容量余量:", headroom)
    
    # 生成扩缩容配置
    config = AutoScalingConfig()
    print(config.to_terraform())
```

### 3.2 Kubernetes HPA配置

```yaml
# hpa.yaml - 水平Pod自动扩缩容
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app-deployment
  minReplicas: 3
  maxReplicas: 50
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 60
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 70
    - type: Pods
      pods:
        metric:
          name: http_requests_per_second
        target:
          type: AverageValue
          averageValue: "1000"
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
        - type: Percent
          value: 100
          periodSeconds: 60
        - type: Pods
          value: 4
          periodSeconds: 60
      selectPolicy: Max
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 10
          periodSeconds: 60
      selectPolicy: Min

---
# VPA - 垂直Pod自动扩缩容
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app-deployment
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
      - containerName: '*'
        minAllowed:
          cpu: 50m
          memory: 100Mi
        maxAllowed:
          cpu: 4
          memory: 8Gi
        controlledResources: ["cpu", "memory"]
```

## 4. 资源索引

| 资源 | 链接 |
|-----|------|
| AWS Well-Architected | https://aws.amazon.com/architecture/well-architected |
| Azure Advisor | https://azure.microsoft.com/services/advisor |
| GCP Recommender | https://cloud.google.com/recommender |
| Kubernetes Autoscaling | https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale |

## 5. 关联知识

- C02_Performance_Metrics
- C03_Cost_Management
- B02_Technical_Practices

## 6. 学习建议

1. 掌握监控数据分析
2. 学习容量建模方法
3. 了解云服务定价模型
4. 实践自动扩缩容

---
*最后更新：2024年*
