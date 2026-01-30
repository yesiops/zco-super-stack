# C01 混沌工程 (Chaos Engineering)

## 1. 主题定位

### 1.1 定义
混沌工程是一门在分布式系统上进行实验的学科，目的是建立对系统抵御生产环境中失控条件的能力的信心。通过主动注入故障，验证系统的韧性，发现潜在的系统弱点。

### 1.2 核心价值
- **故障提前发现**: 在真实故障发生前识别系统弱点
- **韧性验证**: 验证系统的自愈能力和降级策略
- **减少 MTTR**: 缩短平均故障恢复时间
- **提升信心**: 增强团队对系统稳定性的信心

### 1.3 适用场景
- 微服务架构的可靠性验证
- 云原生应用的故障演练
- 分布式数据库的容灾测试
- 容器编排平台的调度测试

---

## 2. 核心概念

### 2.1 混沌工程原则

```
┌─────────────────────────────────────────────────────────────┐
│                    混沌工程核心原则                          │
├─────────────────────────────────────────────────────────────┤
│ 1. 建立稳定状态的假设                                        │
│    - 定义系统正常行为的可测量指标                            │
│    - 建立基线性能数据                                        │
├─────────────────────────────────────────────────────────────┤
│ 2. 多样化真实世界的事件                                      │
│    - 模拟各种故障类型                                        │
│    - 基于历史事故设计实验                                    │
├─────────────────────────────────────────────────────────────┤
│ 3. 生产环境实验                                              │
│    - 在真实环境验证系统行为                                  │
│    - 最小化对客户的影响                                      │
├─────────────────────────────────────────────────────────────┤
│ 4. 持续自动化运行                                            │
│    - 集成到 CI/CD 流水线                                     │
│    - 定期执行混沌实验                                        │
├─────────────────────────────────────────────────────────────┤
│ 5. 最小化爆炸半径                                            │
│    - 控制实验影响范围                                        │
│    - 具备快速回滚能力                                        │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 故障类型分类

```python
#!/usr/bin/env python3
"""
混沌工程故障类型定义与分类
"""

from dataclasses import dataclass
from typing import List, Dict, Optional, Callable
from enum import Enum
import random

class FaultCategory(Enum):
    """故障类别"""
    INFRASTRUCTURE = "基础设施"
    NETWORK = "网络"
    APPLICATION = "应用"
    DATA = "数据"
    DEPENDENCY = "依赖"

class FaultSeverity(Enum):
    """故障严重程度"""
    CRITICAL = "致命"
    HIGH = "高危"
    MEDIUM = "中危"
    LOW = "低危"

@dataclass
class Fault:
    """故障定义"""
    name: str
    category: FaultCategory
    severity: FaultSeverity
    description: str
    impact: str
    rollback_procedure: str
    detection_methods: List[str]


class ChaosFaultLibrary:
    """混沌故障库"""
    
    FAULTS = {
        # 基础设施故障
        "instance_termination": Fault(
            name="实例终止",
            category=FaultCategory.INFRASTRUCTURE,
            severity=FaultSeverity.HIGH,
            description="随机终止生产环境中的实例",
            impact="服务实例减少，可能触发自动扩缩容",
            rollback_procedure="等待自动扩容或手动启动新实例",
            detection_methods=["监控告警", "健康检查失败"]
        ),
        "cpu_stress": Fault(
            name="CPU 压力测试",
            category=FaultCategory.INFRASTRUCTURE,
            severity=FaultSeverity.MEDIUM,
            description="对目标实例施加高 CPU 负载",
            impact="响应延迟增加，可能导致超时",
            rollback_procedure="停止压力测试进程",
            detection_methods=["CPU 使用率告警", "响应时间增加"]
        ),
        "memory_stress": Fault(
            name="内存压力测试",
            category=FaultCategory.INFRASTRUCTURE,
            severity=FaultSeverity.HIGH,
            description="消耗目标实例的可用内存",
            impact="可能导致 OOM，服务重启",
            rollback_procedure="释放内存压力",
            detection_methods=["内存使用率告警", "OOM Killer 日志"]
        ),
        "disk_stress": Fault(
            name="磁盘压力测试",
            category=FaultCategory.INFRASTRUCTURE,
            severity=FaultSeverity.MEDIUM,
            description="填充磁盘空间或施加高 I/O 负载",
            impact="写入失败，性能下降",
            rollback_procedure="清理磁盘空间",
            detection_methods=["磁盘使用率告警", "I/O 等待增加"]
        ),
        
        # 网络故障
        "network_latency": Fault(
            name="网络延迟",
            category=FaultCategory.NETWORK,
            severity=FaultSeverity.MEDIUM,
            description="向网络连接注入延迟",
            impact="请求响应时间增加",
            rollback_procedure="移除延迟注入规则",
            detection_methods=["网络延迟指标", "请求超时"]
        ),
        "network_loss": Fault(
            name="网络丢包",
            category=FaultCategory.NETWORK,
            severity=FaultSeverity.HIGH,
            description="随机丢弃网络数据包",
            impact="重传增加，吞吐量下降",
            rollback_procedure="停止丢包注入",
            detection_methods=["丢包率指标", "连接超时"]
        ),
        "network_partition": Fault(
            name="网络分区",
            category=FaultCategory.NETWORK,
            severity=FaultSeverity.CRITICAL,
            description="隔离部分实例的网络连接",
            impact="脑裂问题，数据不一致",
            rollback_procedure="恢复网络连接",
            detection_methods=["连接失败", "集群状态异常"]
        ),
        "dns_failure": Fault(
            name="DNS 故障",
            category=FaultCategory.NETWORK,
            severity=FaultSeverity.HIGH,
            description="使 DNS 解析失败或返回错误结果",
            impact="服务发现失败",
            rollback_procedure="恢复 DNS 服务",
            detection_methods=["DNS 查询失败", "服务发现错误"]
        ),
        
        # 应用故障
        "service_crash": Fault(
            name="服务崩溃",
            category=FaultCategory.APPLICATION,
            severity=FaultSeverity.HIGH,
            description="模拟服务进程崩溃",
            impact="服务不可用，直到重启完成",
            rollback_procedure="自动或手动重启服务",
            detection_methods=["进程不存在", "健康检查失败"]
        ),
        "exception_injection": Fault(
            name="异常注入",
            category=FaultCategory.APPLICATION,
            severity=FaultSeverity.MEDIUM,
            description="向应用注入特定异常",
            impact="错误率增加，部分功能不可用",
            rollback_procedure="停止异常注入",
            detection_methods=["错误率增加", "异常日志"]
        ),
        "latency_injection": Fault(
            name="延迟注入",
            category=FaultCategory.APPLICATION,
            severity=FaultSeverity.LOW,
            description="在应用层增加处理延迟",
            impact="响应时间增加",
            rollback_procedure="停止延迟注入",
            detection_methods=["P99 延迟增加"]
        ),
        
        # 数据故障
        "database_slowdown": Fault(
            name="数据库慢查询",
            category=FaultCategory.DATA,
            severity=FaultSeverity.HIGH,
            description="使数据库查询变慢",
            impact="整体响应延迟，连接池耗尽",
            rollback_procedure="优化查询或停止注入",
            detection_methods=["查询时间增加", "连接池告警"]
        ),
        "cache_failure": Fault(
            name="缓存故障",
            category=FaultCategory.DATA,
            severity=FaultSeverity.MEDIUM,
            description="使缓存服务不可用",
            impact="请求直达数据库，负载增加",
            rollback_procedure="恢复缓存服务",
            detection_methods=["缓存命中率下降", "数据库负载增加"]
        ),
        "data_corruption": Fault(
            name="数据损坏",
            category=FaultCategory.DATA,
            severity=FaultSeverity.CRITICAL,
            description="模拟数据损坏场景",
            impact="数据不一致，业务错误",
            rollback_procedure="从备份恢复",
            detection_methods=["数据校验失败", "业务异常"]
        ),
        
        # 依赖故障
        "dependency_failure": Fault(
            name="依赖服务故障",
            category=FaultCategory.DEPENDENCY,
            severity=FaultSeverity.HIGH,
            description="模拟下游依赖服务不可用",
            impact="级联故障，服务降级",
            rollback_procedure="恢复依赖服务",
            detection_methods=["依赖调用失败", "熔断器打开"]
        ),
        "dependency_timeout": Fault(
            name="依赖超时",
            category=FaultCategory.DEPENDENCY,
            severity=FaultSeverity.MEDIUM,
            description="使依赖调用超时",
            impact="线程阻塞，资源耗尽",
            rollback_procedure="调整超时配置",
            detection_methods=["超时错误", "线程池告警"]
        ),
    }
    
    @classmethod
    def get_fault(cls, name: str) -> Optional[Fault]:
        """获取故障定义"""
        return cls.FAULTS.get(name)
    
    @classmethod
    def get_by_category(cls, category: FaultCategory) -> List[Fault]:
        """按类别获取故障"""
        return [f for f in cls.FAULTS.values() if f.category == category]
    
    @classmethod
    def get_by_severity(cls, severity: FaultSeverity) -> List[Fault]:
        """按严重程度获取故障"""
        return [f for f in cls.FAULTS.values() if f.severity == severity]
    
    @classmethod
    def get_random_fault(cls, category: Optional[FaultCategory] = None) -> Fault:
        """随机获取故障"""
        if category:
            faults = cls.get_by_category(category)
        else:
            faults = list(cls.FAULTS.values())
        return random.choice(faults)


def print_fault_library():
    """打印故障库"""
    print("=" * 80)
    print("混沌工程故障库")
    print("=" * 80)
    
    library = ChaosFaultLibrary()
    
    for category in FaultCategory:
        faults = library.get_by_category(category)
        print(f"\n## {category.value} ({len(faults)} 种)")
        print("-" * 80)
        
        for fault in faults:
            print(f"\n{fault.name} [{fault.severity.value}]")
            print(f"  描述: {fault.description}")
            print(f"  影响: {fault.impact}")
            print(f"  回滚: {fault.rollback_procedure}")


if __name__ == '__main__':
    print_fault_library()
```

### 2.3 实验设计方法

```python
#!/usr/bin/env python3
"""
混沌实验设计与执行框架
"""

from dataclasses import dataclass, field
from typing import List, Dict, Callable, Any, Optional
from datetime import datetime, timedelta
from enum import Enum
import json
import time
import threading

class ExperimentStatus(Enum):
    """实验状态"""
    PENDING = "待执行"
    RUNNING = "执行中"
    PAUSED = "已暂停"
    COMPLETED = "已完成"
    FAILED = "失败"
    ROLLED_BACK = "已回滚"

@dataclass
class Hypothesis:
    """稳态假设"""
    metric: str  # 监控指标
    condition: str  # 条件 (>, <, ==, range)
    threshold: Any  # 阈值
    duration: int  # 持续时间(秒)
    
    def check(self, value: Any) -> bool:
        """检查假设是否成立"""
        if self.condition == ">":
            return value > self.threshold
        elif self.condition == "<":
            return value < self.threshold
        elif self.condition == "==":
            return value == self.threshold
        elif self.condition == "range":
            return self.threshold[0] <= value <= self.threshold[1]
        return False

@dataclass
class AbortCondition:
    """终止条件"""
    metric: str
    condition: str
    threshold: Any
    action: str = "rollback"  # rollback, pause, alert

@dataclass
class ChaosExperiment:
    """混沌实验定义"""
    id: str
    name: str
    description: str
    target: str  # 目标服务/实例
    fault_type: str  # 故障类型
    duration: int  # 实验持续时间
    
    # 实验配置
    steady_state_hypothesis: List[Hypothesis] = field(default_factory=list)
    abort_conditions: List[AbortCondition] = field(default_factory=list)
    rollback_procedures: List[str] = field(default_factory=list)
    
    # 运行时状态
    status: ExperimentStatus = ExperimentStatus.PENDING
    start_time: Optional[datetime] = None
    end_time: Optional[datetime] = None
    metrics_data: List[Dict] = field(default_factory=list)
    events: List[Dict] = field(default_factory=list)


class ChaosOrchestrator:
    """混沌实验编排器"""
    
    def __init__(self):
        self.experiments: Dict[str, ChaosExperiment] = {}
        self.running_threads: Dict[str, threading.Thread] = {}
        self.metric_collectors: Dict[str, Callable] = {}
    
    def register_metric_collector(self, name: str, collector: Callable):
        """注册指标收集器"""
        self.metric_collectors[name] = collector
    
    def create_experiment(self, experiment: ChaosExperiment) -> str:
        """创建实验"""
        self.experiments[experiment.id] = experiment
        print(f"[+] 创建实验: {experiment.name} (ID: {experiment.id})")
        return experiment.id
    
    def run_experiment(self, experiment_id: str) -> bool:
        """执行实验"""
        exp = self.experiments.get(experiment_id)
        if not exp:
            print(f"[!] 实验不存在: {experiment_id}")
            return False
        
        if exp.status == ExperimentStatus.RUNNING:
            print(f"[!] 实验已在运行: {experiment_id}")
            return False
        
        # 启动实验线程
        thread = threading.Thread(target=self._execute_experiment, args=(exp,))
        self.running_threads[experiment_id] = thread
        thread.start()
        
        return True
    
    def _execute_experiment(self, exp: ChaosExperiment):
        """实验执行逻辑"""
        print(f"\n{'='*60}")
        print(f"开始执行混沌实验: {exp.name}")
        print(f"{'='*60}")
        
        exp.status = ExperimentStatus.RUNNING
        exp.start_time = datetime.now()
        exp.events.append({
            'time': datetime.now().isoformat(),
            'event': 'EXPERIMENT_STARTED'
        })
        
        try:
            # 1. 稳态验证阶段
            print("\n[1/4] 稳态验证阶段...")
            if not self._verify_steady_state(exp):
                print("[!] 稳态验证失败，终止实验")
                exp.status = ExperimentStatus.FAILED
                return
            print("[+] 稳态验证通过")
            
            # 2. 注入故障阶段
            print(f"\n[2/4] 注入故障: {exp.fault_type}")
            self._inject_fault(exp)
            exp.events.append({
                'time': datetime.now().isoformat(),
                'event': 'FAULT_INJECTED',
                'fault': exp.fault_type
            })
            
            # 3. 观察和验证阶段
            print(f"\n[3/4] 观察阶段 (持续 {exp.duration} 秒)...")
            self._observe_and_validate(exp)
            
            # 4. 恢复阶段
            print("\n[4/4] 恢复阶段...")
            self._rollback(exp)
            exp.status = ExperimentStatus.COMPLETED
            
        except Exception as e:
            print(f"[!] 实验执行异常: {e}")
            exp.status = ExperimentStatus.FAILED
            self._rollback(exp)
        
        exp.end_time = datetime.now()
        self._generate_report(exp)
    
    def _verify_steady_state(self, exp: ChaosExperiment) -> bool:
        """验证稳态假设"""
        for hypothesis in exp.steady_state_hypothesis:
            # 收集指标
            collector = self.metric_collectors.get(hypothesis.metric)
            if collector:
                value = collector()
                if not hypothesis.check(value):
                    print(f"  [!] 稳态假设失败: {hypothesis.metric} = {value}")
                    return False
                print(f"  [+] {hypothesis.metric}: {value}")
        return True
    
    def _inject_fault(self, exp: ChaosExperiment):
        """注入故障 (模拟)"""
        print(f"  [+] 正在注入: {exp.fault_type}")
        # 实际实现中调用具体的故障注入工具
        time.sleep(1)  # 模拟注入时间
    
    def _observe_and_validate(self, exp: ChaosExperiment):
        """观察和验证"""
        start_time = time.time()
        
        while time.time() - start_time < exp.duration:
            # 收集指标
            for hypothesis in exp.steady_state_hypothesis:
                collector = self.metric_collectors.get(hypothesis.metric)
                if collector:
                    value = collector()
                    exp.metrics_data.append({
                        'timestamp': datetime.now().isoformat(),
                        'metric': hypothesis.metric,
                        'value': value
                    })
                    print(f"  [{int(time.time() - start_time)}s] {hypothesis.metric}: {value}")
            
            # 检查终止条件
            for abort in exp.abort_conditions:
                collector = self.metric_collectors.get(abort.metric)
                if collector:
                    value = collector()
                    if self._check_abort_condition(abort, value):
                        print(f"  [!] 触发终止条件: {abort.metric} {abort.condition} {abort.threshold}")
                        if abort.action == "rollback":
                            self._rollback(exp)
                            exp.status = ExperimentStatus.ROLLED_BACK
                            return
            
            time.sleep(5)
    
    def _check_abort_condition(self, abort: AbortCondition, value: Any) -> bool:
        """检查终止条件"""
        if abort.condition == ">":
            return value > abort.threshold
        elif abort.condition == "<":
            return value < abort.threshold
        return False
    
    def _rollback(self, exp: ChaosExperiment):
        """回滚故障"""
        print("  [+] 执行回滚...")
        for procedure in exp.rollback_procedures:
            print(f"    - {procedure}")
        time.sleep(1)
        exp.events.append({
            'time': datetime.now().isoformat(),
            'event': 'ROLLBACK_COMPLETED'
        })
    
    def _generate_report(self, exp: ChaosExperiment):
        """生成实验报告"""
        report = {
            'experiment_id': exp.id,
            'name': exp.name,
            'status': exp.status.value,
            'start_time': exp.start_time.isoformat() if exp.start_time else None,
            'end_time': exp.end_time.isoformat() if exp.end_time else None,
            'duration': (exp.end_time - exp.start_time).total_seconds() if exp.end_time else 0,
            'events': exp.events,
            'metrics': exp.metrics_data
        }
        
        filename = f"chaos_report_{exp.id}_{datetime.now().strftime('%Y%m%d_%H%M%S')}.json"
        with open(filename, 'w') as f:
            json.dump(report, f, indent=2)
        
        print(f"\n[+] 报告已保存: {filename}")


def demo_experiment():
    """演示实验创建和执行"""
    orchestrator = ChaosOrchestrator()
    
    # 注册模拟指标收集器
    import random
    orchestrator.register_metric_collector('response_time', 
        lambda: random.randint(50, 200))
    orchestrator.register_metric_collector('error_rate', 
        lambda: random.uniform(0, 0.05))
    orchestrator.register_metric_collector('throughput', 
        lambda: random.randint(800, 1200))
    
    # 创建实验
    experiment = ChaosExperiment(
        id="exp-001",
        name="API 延迟注入测试",
        description="测试 API 服务在延迟增加时的表现",
        target="api-service",
        fault_type="network_latency",
        duration=30,
        steady_state_hypothesis=[
            Hypothesis('response_time', '<', 100, 30),
            Hypothesis('error_rate', '<', 0.01, 30),
            Hypothesis('throughput', '>', 800, 30),
        ],
        abort_conditions=[
            AbortCondition('error_rate', '>', 0.1, 'rollback'),
            AbortCondition('response_time', '>', 5000, 'rollback'),
        ],
        rollback_procedures=[
            "停止网络延迟注入",
            "重启网络服务",
            "验证服务健康状态"
        ]
    )
    
    exp_id = orchestrator.create_experiment(experiment)
    orchestrator.run_experiment(exp_id)


if __name__ == '__main__':
    demo_experiment()
```

---

## 3. 技术实践

### 3.1 Chaos Monkey 风格实现

```python
#!/usr/bin/env python3
"""
Chaos Monkey 风格故障注入实现
Netflix 混沌工程工具的开源概念实现
"""

import random
import time
import threading
from datetime import datetime
from typing import List, Dict, Callable
from dataclasses import dataclass
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger('ChaosMonkey')


@dataclass
class Instance:
    """服务实例"""
    id: str
    name: str
    region: str
    status: str  # running, terminated, stopped
    tags: Dict[str, str]


class InstanceManager:
    """实例管理器 (模拟云服务)"""
    
    def __init__(self):
        self.instances: Dict[str, Instance] = {}
        self._lock = threading.Lock()
    
    def register_instance(self, instance: Instance):
        """注册实例"""
        with self._lock:
            self.instances[instance.id] = instance
        logger.info(f"注册实例: {instance.id} ({instance.name})")
    
    def get_running_instances(self, tag_filter: Dict[str, str] = None) -> List[Instance]:
        """获取运行中的实例"""
        with self._lock:
            instances = [
                inst for inst in self.instances.values() 
                if inst.status == 'running'
            ]
            if tag_filter:
                instances = [
                    inst for inst in instances
                    if all(inst.tags.get(k) == v for k, v in tag_filter.items())
                ]
            return instances
    
    def terminate_instance(self, instance_id: str) -> bool:
        """终止实例"""
        with self._lock:
            if instance_id in self.instances:
                self.instances[instance_id].status = 'terminated'
                logger.info(f"已终止实例: {instance_id}")
                return True
        return False
    
    def start_instance(self, instance_id: str) -> bool:
        """启动实例"""
        with self._lock:
            if instance_id in self.instances:
                self.instances[instance_id].status = 'running'
                logger.info(f"已启动实例: {instance_id}")
                return True
        return False


class ChaosMonkey:
    """
    Chaos Monkey - 随机关闭生产实例
    Netflix 开源概念实现
    """
    
    def __init__(self, instance_manager: InstanceManager):
        self.instance_manager = instance_manager
        self.running = False
        self.termination_probability = 0.2  # 20% 概率终止实例
        self.min_instances = 2  # 最小保留实例数
        self.excluded_tags = {'chaos': 'immune'}  # 免疫标签
        self.schedule_interval = 3600  # 每小时执行一次
        self._thread: threading.Thread = None
    
    def configure(self, 
                  probability: float = 0.2,
                  min_instances: int = 2,
                  excluded_tags: Dict[str, str] = None,
                  interval: int = 3600):
        """配置 Chaos Monkey"""
        self.termination_probability = probability
        self.min_instances = min_instances
        if excluded_tags:
            self.excluded_tags = excluded_tags
        self.schedule_interval = interval
        logger.info(f"Chaos Monkey 配置更新: probability={probability}")
    
    def start(self):
        """启动 Chaos Monkey"""
        if self.running:
            logger.warning("Chaos Monkey 已在运行")
            return
        
        self.running = True
        self._thread = threading.Thread(target=self._run_loop)
        self._thread.daemon = True
        self._thread.start()
        logger.info("Chaos Monkey 已启动")
    
    def stop(self):
        """停止 Chaos Monkey"""
        self.running = False
        if self._thread:
            self._thread.join(timeout=5)
        logger.info("Chaos Monkey 已停止")
    
    def _run_loop(self):
        """主运行循环"""
        while self.running:
            self._execute_chaos()
            time.sleep(self.schedule_interval)
    
    def _execute_chaos(self):
        """执行混沌操作"""
        logger.info("=" * 60)
        logger.info(f"执行混沌操作: {datetime.now()}")
        logger.info("=" * 60)
        
        # 获取符合条件的实例
        running = self.instance_manager.get_running_instances()
        
        # 排除免疫实例
        candidates = [
            inst for inst in running
            if not any(inst.tags.get(k) == v for k, v in self.excluded_tags.items())
        ]
        
        logger.info(f"运行中实例: {len(running)}, 候选实例: {len(candidates)}")
        
        if len(candidates) <= self.min_instances:
            logger.warning(f"实例数量 ({len(candidates)}) 低于最小阈值 ({self.min_instances})，跳过")
            return
        
        # 随机选择实例
        victim = random.choice(candidates)
        
        # 按概率决定是否终止
        if random.random() < self.termination_probability:
            logger.info(f"选中牺牲实例: {victim.id} ({victim.name})")
            logger.info(f"区域: {victim.region}, 标签: {victim.tags}")
            
            # 模拟终止前的通知
            self._send_notification(victim)
            
            # 执行终止
            self.instance_manager.terminate_instance(victim.id)
            
            logger.info(f"实例 {victim.id} 已终止")
        else:
            logger.info("本次未触发终止 (概率未命中)")
    
    def _send_notification(self, instance: Instance):
        """发送终止通知"""
        # 实际实现中可能发送到 Slack、PagerDuty 等
        logger.info(f"[通知] 即将终止实例: {instance.id}")


class LatencyMonkey:
    """
    Latency Monkey - 引入人工延迟
    模拟网络延迟和依赖服务变慢
    """
    
    def __init__(self):
        self.injected_delays: Dict[str, int] = {}  # service -> delay_ms
        self.active = False
    
    def inject_latency(self, service: str, delay_ms: int, probability: float = 1.0):
        """对指定服务注入延迟"""
        self.injected_delays[service] = {
            'delay': delay_ms,
            'probability': probability
        }
        logger.info(f"为 {service} 注入 {delay_ms}ms 延迟 (概率: {probability})")
    
    def remove_latency(self, service: str):
        """移除服务的延迟注入"""
        if service in self.injected_delays:
            del self.injected_delays[service]
            logger.info(f"已移除 {service} 的延迟注入")
    
    def wrap_call(self, service: str, func: Callable, *args, **kwargs):
        """包装函数调用，注入延迟"""
        if service in self.injected_delays:
            config = self.injected_delays[service]
            if random.random() < config['probability']:
                logger.debug(f"应用延迟: {config['delay']}ms")
                time.sleep(config['delay'] / 1000)
        
        return func(*args, **kwargs)


class ConformityMonkey:
    """
    Conformity Monkey - 查找并终止不符合最佳实践的实例
    """
    
    def __init__(self, instance_manager: InstanceManager):
        self.instance_manager = instance_manager
        self.conformity_rules: List[Callable[[Instance], bool]] = []
    
    def add_rule(self, rule: Callable[[Instance], bool]):
        """添加合规性规则"""
        self.conformity_rules.append(rule)
    
    def check_conformity(self, instance: Instance) -> List[str]:
        """检查实例合规性"""
        violations = []
        for i, rule in enumerate(self.conformity_rules):
            if not rule(instance):
                violations.append(f"Rule_{i}")
        return violations
    
    def audit_all(self):
        """审计所有实例"""
        logger.info("=" * 60)
        logger.info("Conformity Monkey 合规性审计")
        logger.info("=" * 60)
        
        for instance in self.instance_manager.instances.values():
            violations = self.check_conformity(instance)
            if violations:
                logger.warning(f"实例 {instance.id} 不合规: {violations}")
            else:
                logger.info(f"实例 {instance.id} 通过合规性检查")


def demo_chaos_monkey():
    """演示 Chaos Monkey"""
    print("\n" + "=" * 60)
    print("Chaos Monkey 演示")
    print("=" * 60)
    
    # 创建实例管理器
    manager = InstanceManager()
    
    # 注册一些实例
    for i in range(5):
        instance = Instance(
            id=f"instance-{i}",
            name=f"api-server-{i}",
            region=random.choice(["us-east-1", "us-west-2", "eu-west-1"]),
            status="running",
            tags={
                "env": "production",
                "service": "api",
                "chaos": "enabled" if i < 4 else "immune"
            }
        )
        manager.register_instance(instance)
    
    # 创建并配置 Chaos Monkey
    monkey = ChaosMonkey(manager)
    monkey.configure(
        probability=0.8,  # 80% 概率
        min_instances=2,
        interval=5  # 5秒间隔用于演示
    )
    
    # 启动
    monkey.start()
    
    # 运行一段时间
    time.sleep(15)
    
    # 停止
    monkey.stop()
    
    # 显示最终状态
    print("\n" + "=" * 60)
    print("最终实例状态")
    print("=" * 60)
    for inst in manager.instances.values():
        print(f"{inst.id}: {inst.status}")


if __name__ == '__main__':
    demo_chaos_monkey()
```

### 3.2 Litmus 风格混沌实验

```yaml
# chaos-experiment.yaml
# LitmusChaos 风格的混沌实验定义

apiVersion: litmuschaos.io/v1alpha1
kind: ChaosEngine
metadata:
  name: api-chaos
  namespace: default
spec:
  appinfo:
    appns: 'default'
    applabel: 'app=api-service'
    appkind: 'deployment'
  
  # 定义稳态假设
  steadyStateVerification:
    - name: "response-time-check"
      type: "probe"
      mode: "Continuous"
      runProperties:
        probeTimeout: "5s"
        retry: 2
        interval: "5s"
        probePollingInterval: "2s"
      httpProbe:
        inputs:
          url: "http://api-service/health"
          insecureSkipVerify: false
          method:
            get:
              criteria: ==
              responseCode: "200"
    
    - name: "error-rate-check"
      type: "promProbe"
      mode: "Edge"
      promProbe:
        inputs:
          endpoint: "http://prometheus:9090"
          query: "sum(rate(http_requests_total{status=~\"5..\"}[1m]))"
        comparator:
          criteria: "<"
          value: "0.01"
  
  # 定义实验
  experiments:
    - name: pod-cpu-hog
      spec:
        components:
          env:
            # CPU 核心数
            - name: CPU_CORES
              value: "1"
            # 压力测试持续时间
            - name: TOTAL_CHAOS_DURATION
              value: "60"
            # 目标容器
            - name: TARGET_CONTAINER
              value: "api-container"
          
          # 中止条件
          abortConditions:
            - probe: "error-rate-check"
              condition: ">"
              threshold: "0.1"
              action: "stop"
    
    - name: pod-network-latency
      spec:
        components:
          env:
            - name: TARGET_CONTAINER
              value: "api-container"
            - name: NETWORK_INTERFACE
              value: "eth0"
            - name: LIB_IMAGE
              value: "litmuschaos/go-runner:latest"
            - name: TC_IMAGE
              value: "gaiadocker/iproute2"
            - name: NETWORK_LATENCY
              value: "2000"
            - name: TOTAL_CHAOS_DURATION
              value: "60"
```

```python
#!/usr/bin/env python3
"""
Kubernetes 混沌实验控制器
模拟 LitmusChaos 的核心功能
"""

import yaml
import time
import subprocess
from typing import Dict, List
from dataclasses import dataclass
from datetime import datetime

@dataclass
class ChaosExperiment:
    """混沌实验定义"""
    name: str
    experiment_type: str
    target: Dict
    parameters: Dict
    abort_conditions: List[Dict]


class K8sChaosController:
    """K8s 混沌实验控制器"""
    
    def __init__(self, namespace: str = "default"):
        self.namespace = namespace
        self.active_experiments: Dict[str, ChaosExperiment] = {}
    
    def load_experiment(self, yaml_file: str) -> ChaosExperiment:
        """从 YAML 加载实验定义"""
        with open(yaml_file, 'r') as f:
            spec = yaml.safe_load(f)
        
        exp_data = spec['spec']['experiments'][0]
        env_vars = {e['name']: e['value'] for e in exp_data['spec']['components']['env']}
        
        return ChaosExperiment(
            name=exp_data['name'],
            experiment_type=exp_data['name'],
            target={
                'namespace': spec['spec']['appinfo']['appns'],
                'label': spec['spec']['appinfo']['applabel']
            },
            parameters=env_vars,
            abort_conditions=exp_data['spec']['components'].get('abortConditions', [])
        )
    
    def inject_cpu_stress(self, experiment: ChaosExperiment):
        """注入 CPU 压力"""
        print(f"[*] 注入 CPU 压力到 {experiment.target['label']}")
        
        cores = experiment.parameters.get('CPU_CORES', '1')
        duration = experiment.parameters.get('TOTAL_CHAOS_DURATION', '60')
        
        # 获取目标 Pod
        pods = self._get_target_pods(experiment.target['label'])
        
        for pod in pods:
            print(f"  [+] 对 Pod {pod} 注入 CPU 压力")
            # 在 Pod 中运行 stress-ng
            cmd = [
                'kubectl', 'exec', pod, '-n', self.namespace, '--',
                'stress-ng', '--cpu', cores, '--timeout', f"{duration}s"
            ]
            # subprocess.Popen(cmd)  # 后台运行
    
    def inject_network_latency(self, experiment: ChaosExperiment):
        """注入网络延迟"""
        print(f"[*] 注入网络延迟到 {experiment.target['label']}")
        
        latency = experiment.parameters.get('NETWORK_LATENCY', '100')
        duration = experiment.parameters.get('TOTAL_CHAOS_DURATION', '60')
        interface = experiment.parameters.get('NETWORK_INTERFACE', 'eth0')
        
        pods = self._get_target_pods(experiment.target['label'])
        
        for pod in pods:
            print(f"  [+] 对 Pod {pod} 注入 {latency}ms 延迟")
            # 使用 tc 命令注入延迟
            cmd = [
                'kubectl', 'exec', pod, '-n', self.namespace, '--',
                'tc', 'qdisc', 'add', 'dev', interface, 
                'root', 'netem', 'delay', f"{latency}ms"
            ]
            # subprocess.run(cmd)
            
            # 计划恢复
            time.sleep(int(duration))
            self._cleanup_network_chaos(pod, interface)
    
    def _cleanup_network_chaos(self, pod: str, interface: str):
        """清理网络混沌"""
        print(f"  [+] 清理 {pod} 的网络混沌")
        cmd = [
            'kubectl', 'exec', pod, '-n', self.namespace, '--',
            'tc', 'qdisc', 'del', 'dev', interface, 'root'
        ]
        # subprocess.run(cmd)
    
    def _get_target_pods(self, label_selector: str) -> List[str]:
        """获取目标 Pod 列表"""
        cmd = [
            'kubectl', 'get', 'pods', '-n', self.namespace,
            '-l', label_selector,
            '-o', 'jsonpath={.items[*].metadata.name}'
        ]
        # result = subprocess.run(cmd, capture_output=True, text=True)
        # return result.stdout.strip().split()
        return ["mock-pod-1", "mock-pod-2"]  # 模拟返回


def demo_k8s_chaos():
    """演示 K8s 混沌实验"""
    print("=" * 60)
    print("Kubernetes 混沌实验演示")
    print("=" * 60)
    
    controller = K8sChaosController(namespace="production")
    
    # 模拟加载实验
    experiment = ChaosExperiment(
        name="pod-cpu-hog",
        experiment_type="cpu-stress",
        target={
            'namespace': 'production',
            'label': 'app=api-service'
        },
        parameters={
            'CPU_CORES': '2',
            'TOTAL_CHAOS_DURATION': '60',
            'TARGET_CONTAINER': 'api-container'
        },
        abort_conditions=[]
    )
    
    print(f"\n加载实验: {experiment.name}")
    print(f"目标: {experiment.target}")
    print(f"参数: {experiment.parameters}")
    
    # 执行实验
    print("\n执行 CPU 压力实验...")
    controller.inject_cpu_stress(experiment)
    
    # 网络延迟实验
    network_exp = ChaosExperiment(
        name="pod-network-latency",
        experiment_type="network-latency",
        target={'namespace': 'production', 'label': 'app=api-service'},
        parameters={'NETWORK_LATENCY': '200', 'TOTAL_CHAOS_DURATION': '30'},
        abort_conditions=[]
    )
    
    print("\n执行网络延迟实验...")
    controller.inject_network_latency(network_exp)


if __name__ == '__main__':
    demo_k8s_chaos()
```

---

## 4. 资源索引

### 4.1 推荐书籍

| 书名 | 作者 | 难度 | 重点内容 |
|------|------|------|----------|
| 《混沌工程》 | Casey Rosenthal | ⭐⭐⭐ | Netflix 混沌工程实践 |
| 《生产环境混沌工程》 | Russ Miles | ⭐⭐⭐ | 实施指南和案例 |
| 《大规模分布式系统》 | Martin Kleppmann | ⭐⭐⭐⭐ | 分布式系统基础 |
| 《SRE：Google 运维解密》 | Google | ⭐⭐⭐ | 可靠性工程实践 |

### 4.2 开源工具

| 工具 | 用途 | 链接 |
|------|------|------|
| Chaos Monkey | 实例随机关闭 | https://github.com/Netflix/chaosmonkey |
| Chaos Mesh | K8s 混沌工程 | https://chaos-mesh.org/ |
| LitmusChaos | 云原生混沌工程 | https://litmuschaos.io/ |
| Gremlin | 企业级混沌平台 | https://www.gremlin.com/ |
| Toxiproxy | 网络故障代理 | https://github.com/Shopify/toxiproxy |
| Pumba | Docker 混沌测试 | https://github.com/alexei-led/pumba |

### 4.3 云厂商服务

| 厂商 | 服务 | 说明 |
|------|------|------|
| AWS | AWS Fault Injection Simulator | 托管混沌工程服务 |
| Azure | Azure Chaos Studio | 云混沌工程平台 |
| GCP | Chaos Engineering Framework | 基于 Spinnaker |
| 阿里云 | AHAS Chaos | 应用高可用服务 |
| 腾讯云 | TKE Chaos | 容器服务混沌工程 |

---

## 5. 关联知识

```
A04_Security_Quality/B02_Reliability_Evaluation/
├── C01_Chaos_Engineering (本模块)
│   ├── → C02_Failure_Modes_Analysis: 故障模式识别
│   └── → C03_Disaster_Recovery: 容灾演练
│
├── → B01_Information_Security: 安全混沌测试
├── → B03_Maintenance_Ops: 故障演练与响应
└── → B04_Quality_Assurance: 稳定性测试
```

---

## 6. 学习建议

### 6.1 学习路径

1. **基础阶段**: 理解分布式系统基础、CAP 理论
2. **工具学习**: 掌握 Chaos Mesh、LitmusChaos 的使用
3. **实验设计**: 学习设计有意义的混沌实验
4. **生产实践**: 在小范围生产环境逐步推广

### 6.2 实践项目

| 难度 | 项目 | 技能点 |
|------|------|--------|
| ⭐ | 搭建 Chaos Mesh 环境 | K8s 基础、工具部署 |
| ⭐⭐ | 设计微服务混沌实验 | 实验设计、指标监控 |
| ⭐⭐⭐ | 自动化混沌流水线 | CI/CD 集成、自动分析 |
| ⭐⭐⭐⭐ | 游戏日 (GameDay) | 组织演练、事后分析 |

---

*最后更新: 2024-01-30*
