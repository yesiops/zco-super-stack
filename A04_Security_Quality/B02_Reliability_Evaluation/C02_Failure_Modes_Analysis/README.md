# C02 故障模式分析 (Failure Modes Analysis)

## 1. 主题定位

### 1.1 定义
故障模式分析是一种系统化的工程方法，用于识别系统、产品或过程中潜在的故障模式，评估其影响，并确定预防和纠正措施。主要方法包括 FMEA (Failure Mode and Effects Analysis) 和 FTA (Fault Tree Analysis)。

### 1.2 核心价值
- **预防为主**: 在设计阶段识别潜在故障
- **风险评估**: 量化故障的影响和发生概率
- **决策支持**: 为改进措施提供数据支撑
- **知识积累**: 建立组织级故障知识库

### 1.3 应用领域
- 硬件系统可靠性设计
- 软件架构风险评估
- 运维故障预防
- 安全威胁建模

---

## 2. 核心概念

### 2.1 FMEA 方法详解

```
FMEA 分析流程：
┌──────────────┐
│  定义范围    │
└──────┬───────┘
       │
┌──────▼───────┐
│ 识别潜在故障 │
└──────┬───────┘
       │
┌──────▼───────┐
│ 评估严重度   │ ──> S (Severity) 1-10
└──────┬───────┘
       │
┌──────▼───────┐
│ 评估发生率   │ ──> O (Occurrence) 1-10
└──────┬───────┘
       │
┌──────▼───────┐
│ 评估检测度   │ ──> D (Detection) 1-10
└──────┬───────┘
       │
┌──────▼───────┐
│ 计算 RPN     │ ──> RPN = S × O × D
└──────┬───────┘
       │
┌──────▼───────┐
│ 制定改进措施 │
└──────────────┘
```

```python
#!/usr/bin/env python3
"""
FMEA (故障模式与影响分析) 实现
"""

from dataclasses import dataclass, field
from typing import List, Optional, Dict
from enum import Enum
from datetime import datetime
import json

class SeverityLevel(Enum):
    """严重度等级"""
    NONE = (1, "无影响", "用户无感知")
    VERY_LOW = (2, "很低", "轻微影响，用户不在意")
    LOW = (3, "低", "轻微烦恼，有替代方案")
    MINOR = (4, "轻微", "轻微功能损失")
    MODERATE = (5, "中等", "功能降级但可接受")
    SIGNIFICANT = (6, "显著", "功能显著降级")
    HIGH = (7, "高", "主要功能丧失")
    VERY_HIGH = (8, "很高", "功能完全丧失，无替代")
    HAZARDOUS = (9, "危险", "安全风险，法规问题")
    VERY_HAZARDOUS = (10, "很危险", "安全风险，无预警")
    
    def __init__(self, score: int, name: str, description: str):
        self.score = score
        self.name = name
        self.description = description


class OccurrenceLevel(Enum):
    """发生度等级"""
    REMOTE = (1, "极不可能", "发生率 <= 0.01%")
    VERY_LOW = (2, "很低", "发生率 0.1%")
    LOW = (3, "低", "发生率 0.5%")
    MODERATE_LOW = (4, "中等偏低", "发生率 1%")
    MODERATE = (5, "中等", "发生率 2%")
    MODERATE_HIGH = (6, "中等偏高", "发生率 5%")
    HIGH = (7, "高", "发生率 10%")
    VERY_HIGH = (8, "很高", "发生率 20%")
    EXTREMELY_HIGH = (9, "极高", "发生率 30%")
    ALMOST_CERTAIN = (10, "几乎必然", "发生率 >= 50%")
    
    def __init__(self, score: int, name: str, description: str):
        self.score = score
        self.name = name
        self.description = description


class DetectionLevel(Enum):
    """检测度等级"""
    ALMOST_CERTAIN = (1, "几乎肯定", "自动检测，100%")
    VERY_HIGH = (2, "很高", "高概率检测")
    HIGH = (3, "高", "中等概率检测")
    MODERATE_HIGH = (4, "中等偏高", "中等偏低概率检测")
    MODERATE = (5, "中等", "中等概率检测")
    LOW = (6, "低", "低概率检测")
    VERY_LOW = (7, "很低", "很低概率检测")
    REMOTE = (8, "很偏远", "很偏远概率检测")
    VERY_REMOTE = (9, "很偏远", "几乎无法检测")
    UNCERTAIN = (10, "不确定", "无法检测或无检测")
    
    def __init__(self, score: int, name: str, description: str):
        self.score = score
        self.name = name
        self.description = description


@dataclass
class FailureMode:
    """故障模式定义"""
    id: str
    component: str  # 组件/功能
    failure_mode: str  # 故障模式
    failure_cause: str  # 故障原因
    failure_effect_local: str  # 局部影响
    failure_effect_system: str  # 系统影响
    failure_effect_user: str  # 用户影响
    
    severity: int = 1
    occurrence: int = 1
    detection: int = 1
    
    current_controls: List[str] = field(default_factory=list)
    recommended_actions: List[str] = field(default_factory=list)
    action_owner: str = ""
    target_date: Optional[str] = None
    status: str = "Open"  # Open, In Progress, Closed
    
    @property
    def rpn(self) -> int:
        """风险优先数 (Risk Priority Number)"""
        return self.severity * self.occurrence * self.detection
    
    @property
    def risk_level(self) -> str:
        """风险等级"""
        if self.rpn >= 200:
            return "Critical"
        elif self.rpn >= 100:
            return "High"
        elif self.rpn >= 50:
            return "Medium"
        else:
            return "Low"
    
    def to_dict(self) -> dict:
        return {
            'id': self.id,
            'component': self.component,
            'failure_mode': self.failure_mode,
            'rpn': self.rpn,
            'risk_level': self.risk_level,
            'severity': self.severity,
            'occurrence': self.occurrence,
            'detection': self.detection,
        }


class FMEAAnalyzer:
    """FMEA 分析器"""
    
    def __init__(self, system_name: str):
        self.system_name = system_name
        self.failure_modes: List[FailureMode] = []
        self.created_at = datetime.now()
    
    def add_failure_mode(self, fm: FailureMode):
        """添加故障模式"""
        self.failure_modes.append(fm)
    
    def get_critical_items(self, threshold: int = 100) -> List[FailureMode]:
        """获取高风险项"""
        return [fm for fm in self.failure_modes if fm.rpn >= threshold]
    
    def get_top_risks(self, n: int = 10) -> List[FailureMode]:
        """获取 Top N 风险"""
        return sorted(self.failure_modes, key=lambda x: x.rpn, reverse=True)[:n]
    
    def generate_report(self) -> str:
        """生成 FMEA 报告"""
        lines = []
        lines.append("=" * 100)
        lines.append(f"FMEA 分析报告 - {self.system_name}")
        lines.append(f"生成时间: {self.created_at.strftime('%Y-%m-%d %H:%M:%S')}")
        lines.append("=" * 100)
        
        lines.append(f"\n故障模式总数: {len(self.failure_modes)}")
        
        # 风险分布
        risk_dist = {"Critical": 0, "High": 0, "Medium": 0, "Low": 0}
        for fm in self.failure_modes:
            risk_dist[fm.risk_level] += 1
        
        lines.append("\n风险分布:")
        for level, count in risk_dist.items():
            lines.append(f"  {level}: {count}")
        
        # Top 10 风险
        lines.append("\n" + "=" * 100)
        lines.append("Top 10 高风险故障模式")
        lines.append("=" * 100)
        
        for i, fm in enumerate(self.get_top_risks(10), 1):
            lines.append(f"\n{i}. [{fm.risk_level}] {fm.component} - {fm.failure_mode}")
            lines.append(f"   RPN: {fm.rpn} (S:{fm.severity} × O:{fm.occurrence} × D:{fm.detection})")
            lines.append(f"   原因: {fm.failure_cause}")
            lines.append(f"   系统影响: {fm.failure_effect_system}")
            lines.append(f"   建议措施: {', '.join(fm.recommended_actions)}")
        
        return "\n".join(lines)
    
    def export_to_excel(self, filename: str):
        """导出到 Excel 格式"""
        import csv
        
        with open(filename, 'w', newline='', encoding='utf-8') as f:
            writer = csv.writer(f)
            writer.writerow([
                'ID', '组件', '故障模式', '故障原因',
                '严重度(S)', '发生度(O)', '检测度(D)', 'RPN', '风险等级',
                '当前控制', '建议措施', '负责人', '状态'
            ])
            
            for fm in self.failure_modes:
                writer.writerow([
                    fm.id, fm.component, fm.failure_mode, fm.failure_cause,
                    fm.severity, fm.occurrence, fm.detection, fm.rpn, fm.risk_level,
                    ';'.join(fm.current_controls), ';'.join(fm.recommended_actions),
                    fm.action_owner, fm.status
                ])


# 软件系统 FMEA 示例
def create_software_fmea() -> FMEAAnalyzer:
    """创建软件系统的 FMEA 分析"""
    fmea = FMEAAnalyzer("电商订单系统")
    
    # 故障模式 1: 数据库连接池耗尽
    fmea.add_failure_mode(FailureMode(
        id="F-001",
        component="订单服务",
        failure_mode="数据库连接池耗尽",
        failure_cause="并发请求过高，连接未及时释放",
        failure_effect_local="新请求无法获取连接",
        failure_effect_system="订单创建失败，交易中断",
        failure_effect_user="无法下单，提示系统错误",
        severity=8,
        occurrence=5,
        detection=4,
        current_controls=["连接池监控", "连接超时设置"],
        recommended_actions=[
            "增加连接池大小",
            "实现连接池自动扩容",
            "添加熔断机制"
        ],
        action_owner="后端团队",
        target_date="2024-03-15"
    ))
    
    # 故障模式 2: 缓存雪崩
    fmea.add_failure_mode(FailureMode(
        id="F-002",
        component="缓存层",
        failure_mode="缓存雪崩",
        failure_cause="大量缓存同时过期，请求直达数据库",
        failure_effect_local="数据库压力激增",
        failure_effect_system="数据库过载，服务响应缓慢",
        failure_effect_user="页面加载慢，部分功能不可用",
        severity=7,
        occurrence=4,
        detection=5,
        current_controls=["缓存过期时间随机化"],
        recommended_actions=[
            "实现缓存预热",
            "添加互斥锁防止击穿",
            "熔断降级策略"
        ],
        action_owner="架构团队",
        target_date="2024-02-28"
    ))
    
    # 故障模式 3: 消息队列堆积
    fmea.add_failure_mode(FailureMode(
        id="F-003",
        component="消息队列",
        failure_mode="消息队列堆积",
        failure_cause="消费者处理速度慢于生产者",
        failure_effect_local="消息延迟增加",
        failure_effect_system="订单状态同步延迟，数据不一致",
        failure_effect_user="订单状态显示不正确",
        severity=6,
        occurrence=6,
        detection=3,
        current_controls=["队列深度监控", "消费者告警"],
        recommended_actions=[
            "增加消费者实例",
            "实现消息分区并行处理",
            "添加死信队列"
        ],
        action_owner="消息队列团队",
        target_date="2024-03-01"
    ))
    
    # 故障模式 4: 第三方支付超时
    fmea.add_failure_mode(FailureMode(
        id="F-004",
        component="支付网关",
        failure_mode="第三方支付超时",
        failure_cause="支付服务商网络延迟或故障",
        failure_effect_local="支付请求挂起",
        failure_effect_system="订单支付状态不确定",
        failure_effect_user="支付结果不明确，可能重复支付",
        severity=9,
        occurrence=4,
        detection=2,
        current_controls=["支付超时设置", "支付结果查询"],
        recommended_actions=[
            "实现支付幂等性",
            "添加支付结果异步通知",
            "人工对账机制"
        ],
        action_owner="支付团队",
        target_date="2024-02-15"
    ))
    
    # 故障模式 5: 库存超卖
    fmea.add_failure_mode(FailureMode(
        id="F-005",
        component="库存服务",
        failure_mode="库存超卖",
        failure_cause="并发扣减库存导致数据不一致",
        failure_effect_local="库存数量变为负数",
        failure_effect_system="超卖商品无法发货",
        failure_effect_user="下单成功但无法收到商品",
        severity=9,
        occurrence=3,
        detection=6,
        current_controls=["数据库乐观锁"],
        recommended_actions=[
            "实现分布式锁",
            "使用 Redis 原子操作",
            "库存预扣机制"
        ],
        action_owner="库存团队",
        target_date="2024-02-20"
    ))
    
    return fmea


def main():
    """主函数"""
    fmea = create_software_fmea()
    print(fmea.generate_report())


if __name__ == '__main__':
    main()
```

### 2.2 故障树分析 (FTA)

```python
#!/usr/bin/env python3
"""
故障树分析 (Fault Tree Analysis) 实现
"""

from dataclasses import dataclass, field
from typing import List, Optional, Set
from enum import Enum

class GateType(Enum):
    """逻辑门类型"""
    AND = "与门"
    OR = "或门"
    NOT = "非门"
    XOR = "异或门"
    VOTING = "表决门"  # k-of-n
    INHIBIT = "禁门"


@dataclass
class BasicEvent:
    """基本事件（叶节点）"""
    id: str
    name: str
    description: str
    probability: float  # 发生概率
    
    def __hash__(self):
        return hash(self.id)


@dataclass
class IntermediateEvent:
    """中间事件"""
    id: str
    name: str
    description: str
    gate_type: GateType
    inputs: List  # BasicEvent 或 IntermediateEvent
    voting_threshold: int = 0  # 表决门阈值


class FaultTree:
    """故障树"""
    
    def __init__(self, top_event: IntermediateEvent):
        self.top_event = top_event
        self.basic_events: Set[BasicEvent] = set()
        self._collect_basic_events(top_event)
    
    def _collect_basic_events(self, event):
        """收集所有基本事件"""
        if isinstance(event, BasicEvent):
            self.basic_events.add(event)
        elif isinstance(event, IntermediateEvent):
            for inp in event.inputs:
                self._collect_basic_events(inp)
    
    def calculate_probability(self, event=None) -> float:
        """计算事件的发生概率"""
        if event is None:
            event = self.top_event
        
        if isinstance(event, BasicEvent):
            return event.probability
        
        if isinstance(event, IntermediateEvent):
            probabilities = [self.calculate_probability(inp) for inp in event.inputs]
            
            if event.gate_type == GateType.AND:
                # P(A AND B) = P(A) * P(B) (假设独立)
                result = 1.0
                for p in probabilities:
                    result *= p
                return result
            
            elif event.gate_type == GateType.OR:
                # P(A OR B) = 1 - (1-P(A)) * (1-P(B))
                result = 1.0
                for p in probabilities:
                    result *= (1 - p)
                return 1 - result
            
            elif event.gate_type == GateType.VOTING:
                # k-of-n 表决门 (简化计算)
                import math
                n = len(probabilities)
                k = event.voting_threshold
                # 假设所有概率相同
                p = probabilities[0] if probabilities else 0
                result = 0
                for i in range(k, n + 1):
                    result += math.comb(n, i) * (p ** i) * ((1 - p) ** (n - i))
                return result
        
        return 0.0
    
    def find_minimal_cut_sets(self) -> List[Set[BasicEvent]]:
        """找出最小割集"""
        # 使用递推法找割集
        cut_sets = self._find_cut_sets_recursive(self.top_event)
        
        # 移除超集（保留最小割集）
        minimal = []
        for cs in sorted(cut_sets, key=len):
            is_minimal = True
            for m in minimal:
                if m.issubset(cs):
                    is_minimal = False
                    break
            if is_minimal:
                minimal.append(cs)
        
        return minimal
    
    def _find_cut_sets_recursive(self, event) -> List[Set[BasicEvent]]:
        """递归找割集"""
        if isinstance(event, BasicEvent):
            return [{event}]
        
        if isinstance(event, IntermediateEvent):
            if event.gate_type == GateType.AND:
                # AND 门的割集是输入割集的笛卡尔积
                result = [{event.inputs[0]}]
                for inp in event.inputs[1:]:
                    new_result = []
                    for cs1 in result:
                        for cs2 in self._find_cut_sets_recursive(inp):
                            new_result.append(cs1 | cs2)
                    result = new_result
                return result
            
            elif event.gate_type == GateType.OR:
                # OR 门的割集是输入割集的并集
                result = []
                for inp in event.inputs:
                    result.extend(self._find_cut_sets_recursive(inp))
                return result
        
        return []
    
    def print_tree(self, event=None, indent=0):
        """打印故障树"""
        if event is None:
            event = self.top_event
        
        prefix = "  " * indent
        
        if isinstance(event, BasicEvent):
            print(f"{prefix}[{event.id}] {event.name} (P={event.probability:.4f})")
        elif isinstance(event, IntermediateEvent):
            print(f"{prefix}[{event.id}] {event.name} [{event.gate_type.value}]")
            for inp in event.inputs:
                self.print_tree(inp, indent + 1)


def demo_fault_tree():
    """演示故障树分析"""
    print("=" * 60)
    print("故障树分析 (FTA) 演示")
    print("=" * 60)
    
    # 定义基本事件：数据库系统故障
    power_failure = BasicEvent("BE-001", "电源故障", "主电源供应中断", 0.001)
    hardware_failure = BasicEvent("BE-002", "硬件故障", "服务器硬件损坏", 0.01)
    network_failure = BasicEvent("BE-003", "网络故障", "网络连接中断", 0.005)
    software_bug = BasicEvent("BE-004", "软件缺陷", "数据库软件bug", 0.02)
    config_error = BasicEvent("BE-005", "配置错误", "错误的配置参数", 0.03)
    backup_failure = BasicEvent("BE-006", "备份系统故障", "备份机制失效", 0.05)
    
    # 中间事件
    infrastructure_failure = IntermediateEvent(
        "IE-001", "基础设施故障", "硬件或网络问题",
        GateType.OR,
        [power_failure, hardware_failure, network_failure]
    )
    
    software_failure = IntermediateEvent(
        "IE-002", "软件层故障", "软件或配置问题",
        GateType.OR,
        [software_bug, config_error]
    )
    
    # 顶事件
    database_outage = IntermediateEvent(
        "TE-001", "数据库系统宕机", "数据库服务不可用",
        GateType.OR,
        [infrastructure_failure, software_failure, backup_failure]
    )
    
    # 创建故障树
    ft = FaultTree(database_outage)
    
    # 打印树结构
    print("\n故障树结构:")
    ft.print_tree()
    
    # 计算顶事件概率
    top_prob = ft.calculate_probability()
    print(f"\n顶事件发生概率: {top_prob:.4f}")
    
    # 找出最小割集
    cut_sets = ft.find_minimal_cut_sets()
    print(f"\n最小割集 ({len(cut_sets)} 个):")
    for i, cs in enumerate(cut_sets, 1):
        events = ", ".join([e.name for e in cs])
        # 计算割集概率
        prob = 1.0
        for e in cs:
            prob *= e.probability
        print(f"  {i}. {{{events}}} (P={prob:.6f})")


if __name__ == '__main__':
    demo_fault_tree()
```

---

## 3. 技术实践

### 3.1 系统可靠性计算

```python
#!/usr/bin/env python3
"""
系统可靠性计算工具
"""

import math
from typing import List, Tuple
from dataclasses import dataclass

@dataclass
class Component:
    """系统组件"""
    name: str
    reliability: float  # 可靠度 (0-1)
    failure_rate: float  # 故障率 λ (每小时)
    repair_rate: float = 0.0  # 修复率 μ (每小时)


class ReliabilityCalculator:
    """可靠性计算器"""
    
    @staticmethod
    def series_system(components: List[Component]) -> float:
        """串联系统可靠度
        
        R_system = R1 × R2 × ... × Rn
        """
        reliability = 1.0
        for comp in components:
            reliability *= comp.reliability
        return reliability
    
    @staticmethod
    def parallel_system(components: List[Component]) -> float:
        """并联系统可靠度
        
        R_system = 1 - (1-R1) × (1-R2) × ... × (1-Rn)
        """
        unreliability = 1.0
        for comp in components:
            unreliability *= (1 - comp.reliability)
        return 1 - unreliability
    
    @staticmethod
    def k_of_n_system(components: List[Component], k: int) -> float:
        """k/n 表决系统可靠度
        
        至少需要 k 个组件正常工作
        """
        from itertools import combinations
        
        n = len(components)
        reliability = 0.0
        
        # 计算所有 k/n 组合
        for working_indices in combinations(range(n), k):
            # 这些组件工作，其余故障
            prob = 1.0
            for i in range(n):
                if i in working_indices:
                    prob *= components[i].reliability
                else:
                    prob *= (1 - components[i].reliability)
            reliability += prob
        
        return reliability
    
    @staticmethod
    def mttf(failure_rate: float) -> float:
        """计算平均故障间隔时间 (MTTF)
        
        MTTF = 1 / λ
        """
        return 1 / failure_rate if failure_rate > 0 else float('inf')
    
    @staticmethod
    def mttr(repair_rate: float) -> float:
        """计算平均修复时间 (MTTR)
        
        MTTR = 1 / μ
        """
        return 1 / repair_rate if repair_rate > 0 else float('inf')
    
    @staticmethod
    def mtbf(failure_rate: float, repair_rate: float) -> float:
        """计算平均故障间隔时间 (MTBF)
        
        MTBF = MTTF + MTTR = 1/λ + 1/μ
        """
        return ReliabilityCalculator.mttf(failure_rate) + \
               ReliabilityCalculator.mttr(repair_rate)
    
    @staticmethod
    def availability(failure_rate: float, repair_rate: float) -> float:
        """计算系统可用性
        
        A = MTTF / (MTTF + MTTR) = μ / (λ + μ)
        """
        return repair_rate / (failure_rate + repair_rate)
    
    @staticmethod
    def reliability_exponential(failure_rate: float, time: float) -> float:
        """指数分布可靠度函数
        
        R(t) = e^(-λt)
        """
        return math.exp(-failure_rate * time)
    
    @staticmethod
    def reliability_weibull(lambda_param: float, k: float, time: float) -> float:
        """Weibull 分布可靠度函数
        
        R(t) = e^(-(t/λ)^k)
        """
        return math.exp(-((time / lambda_param) ** k))


def demo_reliability_calc():
    """演示可靠性计算"""
    print("=" * 60)
    print("系统可靠性计算演示")
    print("=" * 60)
    
    # 定义组件
    components = [
        Component("Web服务器1", 0.99, 0.001),
        Component("Web服务器2", 0.99, 0.001),
        Component("数据库主", 0.995, 0.0005, 0.1),
        Component("数据库备", 0.99, 0.001, 0.1),
        Component("负载均衡器", 0.999, 0.0001),
    ]
    
    calc = ReliabilityCalculator()
    
    # 串联系统
    print("\n--- 串联系统 (所有组件必须工作) ---")
    series_rel = calc.series_system(components)
    print(f"系统可靠度: {series_rel:.6f}")
    
    # 并联系统
    print("\n--- 并联系统 (至少一个组件工作) ---")
    web_servers = [components[0], components[1]]
    parallel_rel = calc.parallel_system(web_servers)
    print(f"Web 服务器并联可靠度: {parallel_rel:.6f}")
    
    # 2/2 表决系统
    print("\n--- 2/2 表决系统 (两个 Web 服务器都需要工作) ---")
    voting_rel = calc.k_of_n_system(web_servers, 2)
    print(f"2/2 表决可靠度: {voting_rel:.6f}")
    
    # 1/2 表决系统 (热备)
    print("\n--- 1/2 表决系统 (至少一个 Web 服务器工作) ---")
    voting_rel = calc.k_of_n_system(web_servers, 1)
    print(f"1/2 表决可靠度: {voting_rel:.6f}")
    
    # MTTF/MTTR 计算
    print("\n--- 可用性计算 ---")
    db_master = components[2]
    mttf = calc.mttf(db_master.failure_rate)
    mttr = calc.mttr(db_master.repair_rate)
    availability = calc.availability(db_master.failure_rate, db_master.repair_rate)
    
    print(f"数据库 MTTF: {mttf:.2f} 小时 ({mttf/24:.2f} 天)")
    print(f"数据库 MTTR: {mttr:.2f} 小时")
    print(f"数据库可用性: {availability:.6f} ({availability*100:.4f}%)")
    print(f"年度停机时间: {(1-availability)*365*24:.2f} 小时")


if __name__ == '__main__':
    demo_reliability_calc()
```

### 3.2 故障模式影响自动化分析

```python
#!/usr/bin/env python3
"""
自动化故障模式影响分析 (FMECA)
"""

import json
from typing import Dict, List
from dataclasses import dataclass
from collections import defaultdict

@dataclass
class FMECAItem:
    """FMECA 项目"""
    function: str
    failure_mode: str
    failure_cause: str
    failure_effect: str
    severity: int
    probability: str  # 高/中/低
    detectability: str  # 高/中/低
    risk_matrix: str  # 高/中/低
    mitigation: str


class AutomatedFMECA:
    """自动化 FMECA 分析器"""
    
    def __init__(self, system_name: str):
        self.system_name = system_name
        self.items: List[FMECAItem] = []
        self.function_dependencies: Dict[str, List[str]] = defaultdict(list)
    
    def add_dependency(self, function: str, depends_on: List[str]):
        """添加功能依赖关系"""
        self.function_dependencies[function] = depends_on
    
    def propagate_effects(self, item: FMECAItem) -> List[str]:
        """传播故障影响"""
        effects = [item.failure_effect]
        
        # 查找依赖此功能的上游功能
        for func, deps in self.function_dependencies.items():
            if item.function in deps:
                effects.append(f"影响 {func} 功能")
                # 递归传播
                # 简化处理，实际应用中可能需要更复杂的图遍历
        
        return effects
    
    def calculate_risk_matrix(self, severity: int, probability: str, detectability: str) -> str:
        """计算风险矩阵等级"""
        # 概率映射
        prob_map = {"高": 3, "中": 2, "低": 1}
        det_map = {"高": 1, "中": 2, "低": 3}  # 检测度越高风险越低
        
        p = prob_map.get(probability, 2)
        d = det_map.get(detectability, 2)
        
        # 风险分数
        risk_score = severity * p * d
        
        if risk_score >= 72:
            return "高"
        elif risk_score >= 24:
            return "中"
        else:
            return "低"
    
    def analyze(self) -> Dict:
        """执行分析"""
        results = {
            "system": self.system_name,
            "total_items": len(self.items),
            "risk_distribution": defaultdict(int),
            "critical_items": [],
            "recommendations": []
        }
        
        for item in self.items:
            # 传播影响
            propagated = self.propagate_effects(item)
            
            # 风险矩阵
            risk = self.calculate_risk_matrix(
                item.severity, item.probability, item.detectability
            )
            results["risk_distribution"][risk] += 1
            
            if risk == "高":
                results["critical_items"].append({
                    "function": item.function,
                    "failure_mode": item.failure_mode,
                    "severity": item.severity,
                    "effects": propagated
                })
        
        # 生成建议
        for critical in results["critical_items"]:
            results["recommendations"].append(
                f"优先处理: {critical['function']} 的 {critical['failure_mode']}"
            )
        
        return results
    
    def generate_report(self) -> str:
        """生成分析报告"""
        results = self.analyze()
        
        lines = []
        lines.append("=" * 60)
        lines.append(f"FMECA 分析报告 - {self.system_name}")
        lines.append("=" * 60)
        lines.append(f"\n分析项目数: {results['total_items']}")
        
        lines.append("\n风险分布:")
        for risk, count in results['risk_distribution'].items():
            lines.append(f"  {risk}风险: {count} 项")
        
        lines.append(f"\n关键项目 ({len(results['critical_items'])} 项):")
        for item in results['critical_items']:
            lines.append(f"  - {item['function']}: {item['failure_mode']}")
            lines.append(f"    严重度: {item['severity']}, 影响: {', '.join(item['effects'])}")
        
        lines.append("\n改进建议:")
        for rec in results['recommendations']:
            lines.append(f"  * {rec}")
        
        return "\n".join(lines)


def demo_fmeca():
    """演示 FMECA 分析"""
    fmeca = AutomatedFMECA("云存储系统")
    
    # 添加依赖关系
    fmeca.add_dependency("文件上传", ["认证服务", "存储服务"])
    fmeca.add_dependency("文件下载", ["认证服务", "存储服务", "CDN服务"])
    fmeca.add_dependency("权限管理", ["认证服务", "数据库"])
    
    # 添加分析项目
    fmeca.items.extend([
        FMECAItem("认证服务", "认证失败", "数据库连接超时", "用户无法登录", 9, "中", "高", "", "连接池优化"),
        FMECAItem("存储服务", "写入失败", "磁盘空间不足", "文件上传失败", 8, "低", "高", "", "存储扩容"),
        FMECAItem("CDN服务", "缓存未命中", "缓存过期", "下载速度慢", 5, "高", "低", "", "缓存策略优化"),
    ])
    
    print(fmeca.generate_report())


if __name__ == '__main__':
    demo_fmeca()
```

---

## 4. 资源索引

### 4.1 标准与指南

| 标准 | 名称 | 说明 |
|------|------|------|
| AIAG FMEA | 潜在失效模式与影响分析 | 汽车行业标准 |
| IEC 60812 | 系统可靠性分析技术 | 国际标准 |
| MIL-STD-1629 | 故障模式影响及危害性分析 | 美国军用标准 |
| ISO 31000 | 风险管理指南 | 通用风险管理 |

### 4.2 工具推荐

| 工具 | 用途 | 链接 |
|------|------|------|
| APIS IQ-RM | FMEA 管理 | https://www.apis.de/ |
| PLATO e1ns | 可靠性工程 | https://www.plato.de/ |
| ReliaSoft | 可靠性分析 | https://www.reliasoft.com/ |
| XFMEA | FMEA 软件 | https://www.weibull.com/ |

---

## 5. 关联知识

```
A04_Security_Quality/B02_Reliability_Evaluation/
├── C02_Failure_Modes_Analysis (本模块)
│   ├── → C01_Chaos_Engineering: 验证故障假设
│   └── → C03_Disaster_Recovery: 制定恢复策略
│
├── → B01_Information_Security: 安全故障分析
└── → B04_Quality_Assurance: 质量风险评估
```

---

## 6. 学习建议

### 6.1 学习路径

1. **基础理论**: 理解 FMEA 的基本概念和计算方法
2. **标准学习**: 学习 AIAG & VDA FMEA 手册
3. **工具实践**: 使用专业 FMEA 软件
4. **实际项目**: 参与产品的 FMEA 分析

### 6.2 认证推荐

- **CRE**: ASQ 认证可靠性工程师
- **CRSS**: 认证风险与安全专家

---

*最后更新: 2024-01-30*
