# C02 性能调优 (Performance Tuning)

## 1. 主题定位

### 1.2 核心价值
- **用户体验**: 降低延迟，提升响应速度
- **资源效率**: 优化资源利用率，降低成本
- **系统容量**: 支持更高的并发和负载
- **稳定性**: 避免性能问题导致的故障

### 1.3 性能指标体系

| 指标类别 | 指标名称 | 说明 | 优秀范围 |
|----------|----------|------|----------|
| **延迟** | Response Time | 响应时间 | < 200ms |
| **延迟** | P50/P95/P99 | 分位延迟 | P99 < 500ms |
| **吞吐** | QPS/RPS | 每秒请求数 | 视业务而定 |
| **吞吐** | TPS | 每秒事务数 | 视业务而定 |
| **资源** | CPU % | CPU 使用率 | 40%-70% |
| **资源** | Memory % | 内存使用率 | < 80% |
| **资源** | Disk I/O | 磁盘 I/O | 视硬件而定 |
| **错误** | Error Rate | 错误率 | < 0.1% |

---

## 2. 核心概念

### 2.1 性能分析方法

```
性能分析层次模型：

┌──────────────────────────────────────────────────────────────┐
│ Layer 7: Application (应用层)                                 │
│          - 代码优化、算法改进、SQL 优化                        │
├──────────────────────────────────────────────────────────────┤
│ Layer 6: Framework (框架层)                                   │
│          - 连接池、线程池、缓存配置                            │
├──────────────────────────────────────────────────────────────┤
│ Layer 5: Runtime (运行时层)                                   │
│          - JVM、GC、内存模型                                  │
├──────────────────────────────────────────────────────────────┤
│ Layer 4: Middleware (中间件层)                                │
│          - 数据库、消息队列、Redis                            │
├──────────────────────────────────────────────────────────────┤
│ Layer 3: OS (操作系统层)                                      │
│          - 内核参数、文件系统、网络栈                          │
├──────────────────────────────────────────────────────────────┤
│ Layer 2: Virtualization (虚拟化层)                            │
│          - CPU 调度、内存超分、I/O 虚拟化                     │
├──────────────────────────────────────────────────────────────┤
│ Layer 1: Hardware (硬件层)                                    │
│          - CPU、内存、磁盘、网络                               │
└──────────────────────────────────────────────────────────────┘
```

```python
#!/usr/bin/env python3
"""
性能分析工具集
"""

import time
import psutil
import statistics
from typing import Dict, List, Callable
from dataclasses import dataclass
from functools import wraps
from contextlib import contextmanager

@dataclass
class PerformanceMetrics:
    """性能指标"""
    operation: str
    duration_ms: float
    cpu_percent: float
    memory_mb: float
    io_read_mb: float
    io_write_mb: float
    timestamp: float


class PerformanceProfiler:
    """性能分析器"""
    
    def __init__(self):
        self.metrics: List[PerformanceMetrics] = []
        self.baseline: Dict[str, float] = {}
    
    @contextmanager
    def profile(self, operation: str):
        """性能分析上下文管理器"""
        start_time = time.time()
        process = psutil.Process()
        
        # 记录开始状态
        cpu_start = process.cpu_percent()
        mem_start = process.memory_info().rss / 1024 / 1024
        io_start = process.io_counters()
        
        try:
            yield
        finally:
            # 计算指标
            duration = (time.time() - start_time) * 1000
            cpu_percent = process.cpu_percent()
            mem_mb = process.memory_info().rss / 1024 / 1024
            io_counters = process.io_counters()
            
            metric = PerformanceMetrics(
                operation=operation,
                duration_ms=duration,
                cpu_percent=cpu_percent,
                memory_mb=mem_mb,
                io_read_mb=(io_counters.read_bytes - io_start.read_bytes) / 1024 / 1024,
                io_write_mb=(io_counters.write_bytes - io_start.write_bytes) / 1024 / 1024,
                timestamp=start_time
            )
            
            self.metrics.append(metric)
    
    def profile_decorator(self, func: Callable) -> Callable:
        """性能分析装饰器"""
        @wraps(func)
        def wrapper(*args, **kwargs):
            with self.profile(func.__name__):
                return func(*args, **kwargs)
        return wrapper
    
    def get_summary(self) -> Dict:
        """获取性能摘要"""
        if not self.metrics:
            return {}
        
        by_operation = {}
        for m in self.metrics:
            if m.operation not in by_operation:
                by_operation[m.operation] = []
            by_operation[m.operation].append(m)
        
        summary = {}
        for op, metrics in by_operation.items():
            durations = [m.duration_ms for m in metrics]
            summary[op] = {
                'count': len(metrics),
                'avg_duration_ms': statistics.mean(durations),
                'min_duration_ms': min(durations),
                'max_duration_ms': max(durations),
                'p95_duration_ms': self._percentile(durations, 95),
                'p99_duration_ms': self._percentile(durations, 99),
                'avg_cpu_percent': statistics.mean([m.cpu_percent for m in metrics]),
                'avg_memory_mb': statistics.mean([m.memory_mb for m in metrics]),
            }
        
        return summary
    
    @staticmethod
    def _percentile(data: List[float], percentile: int) -> float:
        """计算百分位数"""
        sorted_data = sorted(data)
        index = int(len(sorted_data) * percentile / 100)
        return sorted_data[min(index, len(sorted_data) - 1)]
    
    def print_report(self):
        """打印性能报告"""
        summary = self.get_summary()
        
        print("\n" + "=" * 80)
        print("性能分析报告")
        print("=" * 80)
        
        for op, stats in summary.items():
            print(f"\n操作: {op}")
            print(f"  执行次数: {stats['count']}")
            print(f"  平均耗时: {stats['avg_duration_ms']:.2f} ms")
            print(f"  最小/最大: {stats['min_duration_ms']:.2f} / {stats['max_duration_ms']:.2f} ms")
            print(f"  P95/P99: {stats['p95_duration_ms']:.2f} / {stats['p99_duration_ms']:.2f} ms")
            print(f"  平均 CPU: {stats['avg_cpu_percent']:.2f}%")
            print(f"  平均内存: {stats['avg_memory_mb']:.2f} MB")


# 代码级性能分析示例
class CodeOptimizer:
    """代码优化示例"""
    
    @staticmethod
    def inefficient_list_operation(n: int) -> List[int]:
        """低效的列表操作 - O(n^2)"""
        result = []
        for i in range(n):
            # 每次都在开头插入，需要移动所有元素
            result.insert(0, i)
        return result
    
    @staticmethod
    def optimized_list_operation(n: int) -> List[int]:
        """优化的列表操作 - O(n)"""
        # 使用 append 然后反转
        result = []
        for i in range(n):
            result.append(i)
        result.reverse()
        return result
    
    @staticmethod
    def inefficient_string_concat(strings: List[str]) -> str:
        """低效的字符串拼接"""
        result = ""
        for s in strings:
            result += s  # 每次都会创建新字符串
        return result
    
    @staticmethod
    def optimized_string_concat(strings: List[str]) -> str:
        """优化的字符串拼接"""
        return "".join(strings)  # 使用 join 更高效
    
    @staticmethod
    def inefficient_lookup(data: List[Dict], key: str, value) -> List[Dict]:
        """低效的查找操作"""
        results = []
        for item in data:
            if item.get(key) == value:
                results.append(item)
        return results
    
    @staticmethod
    def optimized_lookup(data: List[Dict], key: str) -> Dict:
        """优化的查找操作 - 使用索引"""
        # 预构建索引
        index = {}
        for item in data:
            index[item.get(key)] = item
        return index


def demo_profiling():
    """演示性能分析"""
    print("=" * 60)
    print("性能分析演示")
    print("=" * 60)
    
    profiler = PerformanceProfiler()
    optimizer = CodeOptimizer()
    
    # 测试低效实现
    print("\n--- 测试低效实现 ---")
    with profiler.profile("inefficient_list"):
        result = optimizer.inefficient_list_operation(10000)
    print(f"列表长度: {len(result)}")
    
    # 测试优化实现
    print("\n--- 测试优化实现 ---")
    with profiler.profile("optimized_list"):
        result = optimizer.optimized_list_operation(10000)
    print(f"列表长度: {len(result)}")
    
    # 测试字符串拼接
    strings = ["item" + str(i) for i in range(10000)]
    
    print("\n--- 测试字符串拼接 ---")
    with profiler.profile("inefficient_concat"):
        result = optimizer.inefficient_string_concat(strings)
    print(f"结果长度: {len(result)}")
    
    with profiler.profile("optimized_concat"):
        result = optimizer.optimized_string_concat(strings)
    print(f"结果长度: {len(result)}")
    
    # 打印报告
    profiler.print_report()


if __name__ == '__main__':
    demo_profiling()
```

### 2.2 数据库性能优化

```python
#!/usr/bin/env python3
"""
数据库性能优化指南
"""

from dataclasses import dataclass
from typing import List, Dict, Optional
import re

@dataclass
class QueryAnalysis:
    """查询分析结果"""
    query: str
    execution_time_ms: float
    rows_examined: int
    rows_returned: int
    index_used: Optional[str]
    warnings: List[str]
    suggestions: List[str]


class SQLQueryOptimizer:
    """SQL 查询优化器"""
    
    COMMON_PATTERNS = {
        'select_star': re.compile(r'SELECT\s+\*', re.IGNORECASE),
        'missing_where': re.compile(r'SELECT.*FROM\s+\w+\s*(?:LEFT\s+)?JOIN', re.IGNORECASE),
        'implicit_conversion': re.compile(r'\w+\s*=\s*["\']\d+["\']', re.IGNORECASE),
        'offset_pagination': re.compile(r'LIMIT\s+\d+\s+OFFSET\s+\d+', re.IGNORECASE),
        'n_plus_one': re.compile(r'FOR\s+.*IN.*SELECT', re.IGNORECASE),
    }
    
    def analyze_query(self, query: str) -> QueryAnalysis:
        """分析 SQL 查询"""
        warnings = []
        suggestions = []
        
        # 检查 SELECT *
        if self.COMMON_PATTERNS['select_star'].search(query):
            warnings.append("使用 SELECT * 可能导致不必要的列传输")
            suggestions.append("明确指定需要的列名")
        
        # 检查隐式类型转换
        if self.COMMON_PATTERNS['implicit_conversion'].search(query):
            warnings.append("可能存在隐式类型转换，导致索引失效")
            suggestions.append("确保比较操作符两边类型一致")
        
        # 检查 OFFSET 分页
        if self.COMMON_PATTERNS['offset_pagination'].search(query):
            warnings.append("大 OFFSET 分页性能较差")
            suggestions.append("考虑使用游标分页或覆盖索引")
        
        # 检查 N+1 查询模式
        if self.COMMON_PATTERNS['n_plus_one'].search(query):
            warnings.append("可能存在 N+1 查询问题")
            suggestions.append("使用 JOIN 或批量查询替代")
        
        return QueryAnalysis(
            query=query,
            execution_time_ms=0.0,
            rows_examined=0,
            rows_returned=0,
            index_used=None,
            warnings=warnings,
            suggestions=suggestions
        )
    
    @staticmethod
    def generate_index_suggestion(query: str, table: str, 
                                   where_columns: List[str],
                                   order_columns: List[str] = None) -> str:
        """生成索引建议"""
        columns = where_columns.copy()
        if order_columns:
            columns.extend([c for c in order_columns if c not in columns])
        
        column_list = ', '.join(columns)
        return f"CREATE INDEX idx_{table}_{'_'.join(where_columns)} ON {table}({column_list});"
    
    @staticmethod
    def optimize_pagination(table: str, order_column: str, 
                           last_value: any = None, page_size: int = 20) -> str:
        """生成优化的分页查询"""
        if last_value:
            return f"""
                SELECT * FROM {table}
                WHERE {order_column} > %s
                ORDER BY {order_column}
                LIMIT {page_size}
            """.strip()
        else:
            return f"""
                SELECT * FROM {table}
                ORDER BY {order_column}
                LIMIT {page_size}
            """.strip()


class DatabaseConnectionPool:
    """数据库连接池配置建议"""
    
    @staticmethod
    def calculate_pool_size(server_cores: int, 
                           db_max_connections: int,
                           app_instances: int) -> Dict:
        """计算连接池大小"""
        # 基础公式: connections = ((core_count * 2) + effective_spindle_count)
        base_size = (server_cores * 2) + 1
        
        # 考虑应用实例数
        per_instance = min(
            base_size,
            db_max_connections // app_instances - 5  # 保留余量
        )
        
        return {
            'minimum_idle': max(per_instance // 4, 2),
            'maximum_pool_size': per_instance,
            'connection_timeout_ms': 30000,
            'idle_timeout_ms': 600000,
            'max_lifetime_ms': 1800000,
            'recommended': {
                'hikari_cp': f"maximumPoolSize={per_instance}, minimumIdle={max(per_instance // 4, 2)}",
                'c3p0': f"maxPoolSize={per_instance}, minPoolSize={max(per_instance // 4, 2)}",
            }
        }


# 缓存策略
class CacheStrategy:
    """缓存策略建议"""
    
    @staticmethod
    def cache_aside(key: str, loader: callable, ttl_seconds: int = 300):
        """Cache-Aside 模式实现"""
        # 实际实现中需要 Redis 等缓存客户端
        print(f"[Cache-Aside] Key: {key}, TTL: {ttl_seconds}s")
        # 1. 尝试从缓存读取
        # 2. 缓存未命中，从数据库加载
        # 3. 写入缓存
        return loader()
    
    @staticmethod
    def write_through(key: str, value: any, writer: callable, 
                     ttl_seconds: int = 300):
        """Write-Through 模式"""
        print(f"[Write-Through] Key: {key}, TTL: {ttl_seconds}s")
        # 1. 先写入缓存
        # 2. 同步写入数据库
        return writer(value)
    
    @staticmethod
    def write_behind(key: str, value: any, writer: callable):
        """Write-Behind 模式"""
        print(f"[Write-Behind] Key: {key}")
        # 1. 只写入缓存
        # 2. 异步批量写入数据库
        return True


def demo_sql_optimization():
    """演示 SQL 优化"""
    print("=" * 60)
    print("SQL 查询优化分析")
    print("=" * 60)
    
    optimizer = SQLQueryOptimizer()
    
    queries = [
        "SELECT * FROM users WHERE status = 'active'",
        "SELECT id, name FROM orders WHERE user_id = '123' AND created_at > '2024-01-01'",
        "SELECT * FROM products LIMIT 20 OFFSET 10000",
    ]
    
    for query in queries:
        print(f"\n查询: {query[:50]}...")
        analysis = optimizer.analyze_query(query)
        
        if analysis.warnings:
            print("  警告:")
            for w in analysis.warnings:
                print(f"    - {w}")
        
        if analysis.suggestions:
            print("  建议:")
            for s in analysis.suggestions:
                print(f"    - {s}")
        
        if not analysis.warnings:
            print("  ✓ 无明显问题")
    
    # 连接池建议
    print("\n" + "=" * 60)
    print("连接池配置建议")
    print("=" * 60)
    
    pool_config = DatabaseConnectionPool.calculate_pool_size(
        server_cores=8,
        db_max_connections=200,
        app_instances=4
    )
    
    print(f"最小空闲连接: {pool_config['minimum_idle']}")
    print(f"最大连接数: {pool_config['maximum_pool_size']}")
    print(f"HikariCP 配置: {pool_config['recommended']['hikari_cp']}")


if __name__ == '__main__':
    demo_sql_optimization()
```

---

## 3. 技术实践

### 3.1 系统级性能调优

```python
#!/usr/bin/env python3
"""
Linux 系统性能调优
"""

import os
import subprocess
from typing import Dict, List

class SystemTuning:
    """系统调优工具"""
    
    @staticmethod
    def get_kernel_params() -> Dict[str, str]:
        """获取当前内核参数"""
        params = {}
        try:
            with open('/proc/sys/net/core/somaxconn', 'r') as f:
                params['net.core.somaxconn'] = f.read().strip()
            with open('/proc/sys/net/ipv4/tcp_max_syn_backlog', 'r') as f:
                params['net.ipv4.tcp_max_syn_backlog'] = f.read().strip()
            with open('/proc/sys/vm/swappiness', 'r') as f:
                params['vm.swappiness'] = f.read().strip()
        except:
            pass
        return params
    
    @staticmethod
    def get_network_tuning_recommendations() -> Dict[str, Dict]:
        """网络调优建议"""
        return {
            'net.core.somaxconn': {
                'description': 'TCP 连接队列长度',
                'default': '128',
                'recommended': '65535',
                'reason': '支持高并发连接'
            },
            'net.ipv4.tcp_max_syn_backlog': {
                'description': 'SYN 队列长度',
                'default': '1024',
                'recommended': '65535',
                'reason': '防止 SYN Flood'
            },
            'net.core.netdev_max_backlog': {
                'description': '网卡队列长度',
                'default': '1000',
                'recommended': '65535',
                'reason': '提高网络吞吐量'
            },
            'net.ipv4.tcp_tw_reuse': {
                'description': '重用 TIME_WAIT 连接',
                'default': '0',
                'recommended': '1',
                'reason': '提高端口利用率'
            },
            'net.ipv4.ip_local_port_range': {
                'description': '本地端口范围',
                'default': '32768 60999',
                'recommended': '1024 65535',
                'reason': '增加可用端口数'
            }
        }
    
    @staticmethod
    def get_memory_tuning_recommendations() -> Dict[str, Dict]:
        """内存调优建议"""
        return {
            'vm.swappiness': {
                'description': 'swap 使用倾向',
                'default': '60',
                'recommended': '10',
                'reason': '减少 swap 使用，提高性能'
            },
            'vm.dirty_ratio': {
                'description': '脏页比例上限',
                'default': '20',
                'recommended': '40',
                'reason': '提高写入性能'
            },
            'vm.dirty_background_ratio': {
                'description': '后台刷新脏页比例',
                'default': '10',
                'recommended': '10',
                'reason': '保持默认值即可'
            }
        }
    
    @staticmethod
    def generate_sysctl_config() -> str:
        """生成 sysctl 配置文件"""
        lines = ['# 高性能服务器内核参数配置', '']
        
        lines.append('# 网络优化')
        for param, info in SystemTuning.get_network_tuning_recommendations().items():
            lines.append(f'# {info["description"]}')
            lines.append(f'{param} = {info["recommended"]}')
            lines.append('')
        
        lines.append('# 内存优化')
        for param, info in SystemTuning.get_memory_tuning_recommendations().items():
            lines.append(f'# {info["description"]}')
            lines.append(f'{param} = {info["recommended"]}')
            lines.append('')
        
        return '\n'.join(lines)


class JVMPerformanceTuning:
    """JVM 性能调优"""
    
    @staticmethod
    def get_g1_gc_recommendations(heap_size_gb: int) -> Dict[str, str]:
        """G1 垃圾收集器调优建议"""
        return {
            '-XX:+UseG1GC': '使用 G1 垃圾收集器',
            f'-Xms{heap_size_gb}g': '初始堆大小',
            f'-Xmx{heap_size_gb}g': '最大堆大小',
            '-XX:MaxGCPauseMillis=200': '目标最大 GC 停顿时间',
            f'-XX:InitiatingHeapOccupancyPercent={min(45, 35 + heap_size_gb)}': 
                '触发并发 GC 的堆占用率',
            '-XX:+ParallelRefProcEnabled': '并行处理引用',
            '-XX:+AlwaysPreTouch': '启动时预分配内存',
        }
    
    @staticmethod
    def get_zgc_recommendations(heap_size_gb: int) -> Dict[str, str]:
        """ZGC 垃圾收集器调优建议 (JDK 11+)"""
        return {
            '-XX:+UseZGC': '使用 ZGC 垃圾收集器',
            f'-Xms{heap_size_gb}g': '初始堆大小',
            f'-Xmx{heap_size_gb}g': '最大堆大小',
            '-XX:+ZGenerational': '启用分代 ZGC (JDK 21+)',
        }
    
    @staticmethod
    def generate_startup_script(app_name: str, heap_size_gb: int, 
                                use_zgc: bool = False) -> str:
        """生成启动脚本"""
        lines = ['#!/bin/bash', '']
        lines.append(f'# {app_name} JVM 启动脚本')
        lines.append('')
        
        # GC 选择
        if use_zgc and heap_size_gb >= 8:
            gc_opts = JVMPerformanceTuning.get_zgc_recommendations(heap_size_gb)
            lines.append('# 使用 ZGC (低延迟，适合大堆)')
        else:
            gc_opts = JVMPerformanceTuning.get_g1_gc_recommendations(heap_size_gb)
            lines.append('# 使用 G1GC (平衡吞吐量和延迟)')
        
        lines.append('JAVA_OPTS="')
        for opt, desc in gc_opts.items():
            lines.append(f'  {opt} \\')
        
        # 通用优化
        lines.append('  -XX:+DisableExplicitGC \\')
        lines.append('  -XX:+UseStringDeduplication \\')
        lines.append('  -XX:+OptimizeStringConcat \\')
        lines.append('  -XX:+UseCompressedOops \\')
        lines.append('"')
        lines.append('')
        lines.append(f'java $JAVA_OPTS -jar {app_name}.jar')
        
        return '\n'.join(lines)


def demo_system_tuning():
    """演示系统调优"""
    print("=" * 60)
    print("系统性能调优建议")
    print("=" * 60)
    
    # 内核参数
    print("\n--- 当前内核参数 ---")
    current = SystemTuning.get_kernel_params()
    for param, value in current.items():
        print(f"  {param} = {value}")
    
    print("\n--- 网络调优建议 ---")
    for param, info in SystemTuning.get_network_tuning_recommendations().items():
        print(f"\n{param}")
        print(f"  描述: {info['description']}")
        print(f"  默认值: {info['default']}")
        print(f"  建议值: {info['recommended']}")
        print(f"  原因: {info['reason']}")
    
    print("\n--- sysctl.conf 配置 ---")
    print(SystemTuning.generate_sysctl_config())
    
    # JVM 调优
    print("\n" + "=" * 60)
    print("JVM 性能调优")
    print("=" * 60)
    
    print("\n--- G1GC 配置 (4GB 堆) ---")
    for opt, desc in JVMPerformanceTuning.get_g1_gc_recommendations(4).items():
        print(f"{opt:<50} # {desc}")
    
    print("\n--- 启动脚本示例 ---")
    print(JVMPerformanceTuning.generate_startup_script("myapp", 8))


if __name__ == '__main__':
    demo_system_tuning()
```

---

## 4. 资源索引

### 4.1 性能分析工具

| 工具 | 用途 | 层次 |
|------|------|------|
| perf | Linux 性能分析 | 系统 |
| eBPF/BCC | 动态追踪 | 系统 |
| pprof | Go/Python 性能分析 | 应用 |
| JProfiler | Java 性能分析 | 应用 |
| Arthas | Java 诊断 | 应用 |
| MySQL Performance Schema | 数据库性能 | 数据库 |
| Redis slowlog | Redis 慢查询 | 数据库 |

### 4.2 学习资源

- **《性能之巅》**: Brendan Gregg 著，系统性能分析圣经
- **《高性能 MySQL》**: 数据库优化经典
- **《Java 性能优化实践》**: JVM 调优指南

---

## 5. 关联知识

```
A04_Security_Quality/B03_Maintenance_Ops/
├── C02_Performance_Tuning (本模块)
│   ├── → C01_Incident_Response: 性能事件处理
│   └── → C03_Tech_Debt_Management: 性能债务
│
├── → B02_Reliability_Evaluation: 性能与可靠性
└── → B04_Quality_Assurance: 性能测试
```

---

*最后更新: 2024-01-30*
