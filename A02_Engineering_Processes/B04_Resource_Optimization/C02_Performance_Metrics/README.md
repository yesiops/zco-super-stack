# C02_Performance_Metrics - 性能指标

## 1. 主题定位

### 1.1 定义与范围

性能指标（Performance Metrics）是量化系统性能特征的可测量标准，用于评估、监控和优化系统表现。本知识单元涵盖四大黄金信号、RED方法、USE方法等性能分析框架，以及APM工具实践。

### 1.2 业务价值

- 及时发现性能问题
- 指导容量决策
- 优化用户体验
- 支撑SLA管理
- 驱动持续改进

### 1.3 适用场景

| 场景 | 适用性 |
|-----|-------|
| 生产监控 | ⭐⭐⭐⭐⭐ |
| 性能测试 | ⭐⭐⭐⭐⭐ |
| 容量规划 | ⭐⭐⭐⭐⭐ |
| 故障排查 | ⭐⭐⭐⭐⭐ |

## 2. 核心概念

### 2.1 四大黄金信号

```
四大黄金信号 (Google SRE)：

┌─────────────────────────────────────────────────────────────┐
│  Latency    │  延迟    │  服务处理请求所需时间              │
│  Traffic    │  流量    │  系统承受的请求量                  │
│  Errors     │  错误    │  请求失败率                        │
│  Saturation │  饱和度  │  资源接近满载的程度                │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 RED方法 vs USE方法

| 方法 | 适用对象 | 指标 |
|-----|---------|-----|
| RED | 微服务 | Rate(请求率), Errors(错误率), Duration(持续时间) |
| USE | 资源 | Utilization(使用率), Saturation(饱和度), Errors(错误) |

## 3. 技术实践

### 3.1 Prometheus指标采集

```python
#!/usr/bin/env python3
"""
性能指标采集与导出
使用Prometheus客户端库
"""

from prometheus_client import Counter, Histogram, Gauge, generate_latest, CONTENT_TYPE_LATEST
from flask import Flask, Response
import random
import time

app = Flask(__name__)

# 定义指标
REQUEST_COUNT = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status_code']
)

REQUEST_DURATION = Histogram(
    'http_request_duration_seconds',
    'HTTP request duration',
    ['method', 'endpoint'],
    buckets=[0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1.0, 2.5, 5.0, 10.0]
)

ACTIVE_CONNECTIONS = Gauge(
    'http_active_connections',
    'Number of active HTTP connections'
)

QUEUE_SIZE = Gauge(
    'request_queue_size',
    'Current request queue size'
)

BUSINESS_METRICS = {
    'orders_created': Counter('orders_created_total', 'Total orders created'),
    'order_value': Histogram('order_value_dollars', 'Order value distribution'),
    'active_users': Gauge('active_users', 'Number of active users')
}


@app.before_request
def before_request():
    """请求开始处理"""
    ACTIVE_CONNECTIONS.inc()
    REQUEST_DURATION.labels(
        method=request.method,
        endpoint=request.endpoint or 'unknown'
    ).observe(time.time())


@app.after_request
def after_request(response):
    """请求处理完成"""
    ACTIVE_CONNECTIONS.dec()
    REQUEST_COUNT.labels(
        method=request.method,
        endpoint=request.endpoint or 'unknown',
        status_code=response.status_code
    ).inc()
    return response


@app.route('/metrics')
def metrics():
    """Prometheus指标端点"""
    return Response(
        generate_latest(),
        mimetype=CONTENT_TYPE_LATEST
    )


# 性能测试脚本
import requests
import statistics
from concurrent.futures import ThreadPoolExecutor, as_completed


class PerformanceTester:
    """性能测试工具"""
    
    def __init__(self, base_url: str):
        self.base_url = base_url
        self.results = []
    
    def send_request(self, endpoint: str) -> dict:
        """发送单个请求"""
        start = time.time()
        try:
            response = requests.get(
                f"{self.base_url}{endpoint}",
                timeout=30
            )
            duration = time.time() - start
            return {
                'success': response.status_code < 500,
                'status_code': response.status_code,
                'duration': duration,
                'endpoint': endpoint
            }
        except Exception as e:
            return {
                'success': False,
                'error': str(e),
                'duration': time.time() - start,
                'endpoint': endpoint
            }
    
    def load_test(self, endpoint: str, 
                  concurrent_users: int = 10,
                  requests_per_user: int = 100) -> dict:
        """负载测试"""
        total_requests = concurrent_users * requests_per_user
        self.results = []
        
        print(f"Starting load test: {concurrent_users} users, "
              f"{requests_per_user} requests each")
        
        with ThreadPoolExecutor(max_workers=concurrent_users) as executor:
            futures = [
                executor.submit(self.send_request, endpoint)
                for _ in range(total_requests)
            ]
            
            for future in as_completed(futures):
                self.results.append(future.result())
        
        return self.generate_report()
    
    def generate_report(self) -> dict:
        """生成测试报告"""
        successful = [r for r in self.results if r.get('success')]
        failed = [r for r in self.results if not r.get('success')]
        
        durations = [r['duration'] for r in successful]
        
        if not durations:
            return {'error': 'All requests failed'}
        
        durations.sort()
        
        return {
            'total_requests': len(self.results),
            'successful': len(successful),
            'failed': len(failed),
            'error_rate': len(failed) / len(self.results) * 100,
            'latency': {
                'min_ms': round(min(durations) * 1000, 2),
                'max_ms': round(max(durations) * 1000, 2),
                'mean_ms': round(statistics.mean(durations) * 1000, 2),
                'median_ms': round(statistics.median(durations) * 1000, 2),
                'p95_ms': round(durations[int(len(durations) * 0.95)] * 1000, 2),
                'p99_ms': round(durations[int(len(durations) * 0.99)] * 1000, 2)
            },
            'throughput_rps': len(self.results) / sum(durations)
        }


if __name__ == "__main__":
    # 运行性能测试
    tester = PerformanceTester("http://localhost:8000")
    report = tester.load_test("/api/data", concurrent_users=50, requests_per_user=20)
    print(report)
```

### 3.2 Grafana仪表板配置

```json
{
  "dashboard": {
    "title": "应用性能监控",
    "panels": [
      {
        "title": "请求速率",
        "type": "graph",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total[5m])) by (endpoint)",
            "legendFormat": "{{endpoint}}"
          }
        ],
        "yAxes": [
          {"label": "req/s", "min": 0}
        ]
      },
      {
        "title": "P99延迟",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, endpoint))",
            "legendFormat": "{{endpoint}}"
          }
        ],
        "yAxes": [
          {"label": "seconds", "min": 0}
        ],
        "alert": {
          "name": "高延迟告警",
          "conditions": [
            {
              "evaluator": {"params": [1], "type": "gt"},
              "operator": {"type": "and"},
              "query": {"params": ["A", "5m", "now"]},
              "reducer": {"params": [], "type": "avg"},
              "type": "query"
            }
          ],
          "executionErrorState": "alerting",
          "frequency": "1m",
          "handler": 1,
          "message": "P99延迟超过1秒",
          "severity": "critical"
        }
      },
      {
        "title": "错误率",
        "type": "singlestat",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total{status_code=~\"5..\"}[5m])) / sum(rate(http_requests_total[5m])) * 100"
          }
        ],
        "format": "percent",
        "thresholds": "1,5",
        "colorBackground": true,
        "colors": ["green", "yellow", "red"]
      }
    ]
  }
}
```

## 4. 资源索引

| 工具 | 用途 | 官网 |
|-----|-----|------|
| Prometheus | 指标采集 | https://prometheus.io |
| Grafana | 可视化 | https://grafana.com |
| Jaeger | 分布式追踪 | https://jaegertracing.io |
| Datadog | APM | https://datadoghq.com |
| New Relic | APM | https://newrelic.com |

## 5. 关联知识

- C01_Capacity_Planning
- C03_Cost_Management
- B02_Technical_Practices

## 6. 学习建议

1. 掌握四大黄金信号
2. 学习Prometheus查询语言
3. 实践分布式追踪
4. 建立SLI/SLO体系

---
*最后更新：2024年*
