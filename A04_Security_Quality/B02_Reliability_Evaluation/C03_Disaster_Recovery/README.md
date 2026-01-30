# C03 灾难恢复 (Disaster Recovery)

## 1. 主题定位

### 1.1 定义
灾难恢复（Disaster Recovery, DR）是指在发生自然灾害、硬件故障、网络攻击或其他重大事件导致系统中断时，快速恢复 IT 基础设施和业务运营的过程。它是业务连续性计划（BCP）的核心组成部分。

### 1.2 核心价值
- **业务连续性**: 最小化停机时间和业务损失
- **数据保护**: 确保关键数据的安全和可恢复性
- **合规要求**: 满足行业和法规的连续性要求
- **风险降低**: 降低灾难对组织的影响

### 1.3 关键指标

| 指标 | 定义 | 目标值示例 |
|------|------|------------|
| **RTO** | 恢复时间目标 (Recovery Time Objective) | 4小时 |
| **RPO** | 恢复点目标 (Recovery Point Objective) | 1小时 |
| **MTD** | 最大允许中断时间 (Maximum Tolerable Downtime) | 8小时 |
| **WRT** | 工作时间恢复 (Work Recovery Time) | 2小时 |

---

## 2. 核心概念

### 2.1 灾难恢复策略

```
灾难恢复策略金字塔：

       ┌─────────────┐
       │  热备中心    │  (Active-Active, RTO≈0, RPO≈0)
       │ Hot Site    │  成本: $$$$$$
       ├─────────────┤
       │  温备中心    │  (Warm Standby, RTO=数小时)
       │ Warm Site   │  成本: $$$
       ├─────────────┤
       │  冷备中心    │  (Cold Site, RTO=数天)
       │ Cold Site   │  成本: $$
       ├─────────────┤
       │  备份恢复    │  (Backup & Restore, RTO=数小时-数天)
       │  云端灾备    │  成本: $
       ├─────────────┤
       │   无备份    │  (No Backup) - 不建议
       └─────────────┘
```

### 2.2 多活架构设计

```python
#!/usr/bin/env python3
"""
多活灾备架构设计与实现
"""

from dataclasses import dataclass
from typing import List, Dict, Optional
from enum import Enum
import hashlib

class DRStrategy(Enum):
    """灾备策略"""
    ACTIVE_ACTIVE = "双活/多活"
    ACTIVE_PASSIVE = "主备"
    ACTIVE_STANDBY = "主温备"
    PILOT_LIGHT = "指示灯"
    WARM_STANDBY = "温备"
    COLD_STANDBY = "冷备"

class ReplicationMode(Enum):
    """数据复制模式"""
    SYNCHRONOUS = "同步复制"
    ASYNCHRONOUS = "异步复制"
    SEMI_SYNC = "半同步复制"

@dataclass
class DataCenter:
    """数据中心定义"""
    id: str
    name: str
    region: str
    zone: str
    is_active: bool = True
    is_primary: bool = False
    latency_to_primary: float = 0.0  # ms

@dataclass
class ReplicationConfig:
    """复制配置"""
    mode: ReplicationMode
    lag_threshold_ms: int  # 复制延迟阈值
    conflict_resolution: str  # 冲突解决策略

class MultiActiveDRSystem:
    """多活灾备系统"""
    
    def __init__(self):
        self.data_centers: Dict[str, DataCenter] = {}
        self.primary_dc: Optional[str] = None
        self.replication_config: Optional[ReplicationConfig] = None
        self.health_status: Dict[str, bool] = {}
    
    def add_data_center(self, dc: DataCenter):
        """添加数据中心"""
        self.data_centers[dc.id] = dc
        self.health_status[dc.id] = True
        
        if dc.is_primary:
            self.primary_dc = dc.id
    
    def configure_replication(self, config: ReplicationConfig):
        """配置复制策略"""
        self.replication_config = config
    
    def route_request(self, user_id: str) -> str:
        """根据用户 ID 路由到最近的数据中心"""
        if not self.data_centers:
            raise ValueError("没有可用的数据中心")
        
        # 使用一致性哈希选择数据中心
        dc_list = list(self.data_centers.values())
        
        # 计算哈希
        hash_value = int(hashlib.md5(user_id.encode()).hexdigest(), 16)
        
        # 选择健康的数据中心
        healthy_dcs = [dc for dc in dc_list if self.health_status.get(dc.id, False)]
        
        if not healthy_dcs:
            raise RuntimeError("没有健康的数据中心")
        
        # 一致性哈希选择
        index = hash_value % len(healthy_dcs)
        selected_dc = healthy_dcs[index]
        
        return selected_dc.id
    
    def handle_failure(self, dc_id: str):
        """处理数据中心故障"""
        print(f"[!] 检测到数据中心故障: {dc_id}")
        self.health_status[dc_id] = False
        
        # 如果故障的是主数据中心，执行故障转移
        if dc_id == self.primary_dc:
            self._failover()
    
    def _failover(self):
        """执行故障转移"""
        healthy_dcs = [
            dc_id for dc_id, healthy in self.health_status.items() if healthy
        ]
        
        if not healthy_dcs:
            raise RuntimeError("没有可用的数据中心进行故障转移")
        
        # 选择新的主数据中心
        new_primary = healthy_dcs[0]
        self.data_centers[new_primary].is_primary = True
        self.primary_dc = new_primary
        
        print(f"[+] 故障转移完成，新的主数据中心: {new_primary}")
    
    def check_replication_lag(self) -> Dict[str, float]:
        """检查复制延迟"""
        lags = {}
        for dc_id, dc in self.data_centers.items():
            if not dc.is_primary:
                # 模拟复制延迟检测
                lags[dc_id] = dc.latency_to_primary
        return lags


class BackupStrategy:
    """备份策略管理"""
    
    def __init__(self):
        self.backup_schedules: List[Dict] = []
    
    def add_schedule(self, name: str, frequency: str, retention: int, 
                     target: str, encryption: bool = True):
        """添加备份计划
        
        Args:
            name: 备份名称
            frequency: 频率 (hourly, daily, weekly)
            retention: 保留份数
            target: 备份目标 (S3, NAS, Tape)
            encryption: 是否加密
        """
        schedule = {
            'name': name,
            'frequency': frequency,
            'retention': retention,
            'target': target,
            'encryption': encryption
        }
        self.backup_schedules.append(schedule)
    
    def get_3_2_1_strategy(self) -> List[Dict]:
        """获取 3-2-1 备份策略配置"""
        return [
            {
                'name': 'Primary Backup',
                'location': 'Local',
                'type': 'Full',
                'frequency': 'Daily'
            },
            {
                'name': 'Secondary Backup',
                'location': 'Local (Different Media)',
                'type': 'Incremental',
                'frequency': 'Hourly'
            },
            {
                'name': 'Offsite Backup',
                'location': 'Cloud/Remote',
                'type': 'Full',
                'frequency': 'Daily'
            }
        ]


def demo_dr_system():
    """演示灾备系统"""
    print("=" * 60)
    print("多活灾备系统演示")
    print("=" * 60)
    
    # 创建多活系统
    dr_system = MultiActiveDRSystem()
    
    # 添加数据中心
    dr_system.add_data_center(DataCenter(
        id="dc-beijing",
        name="北京数据中心",
        region="cn-north",
        zone="beijing",
        is_primary=True,
        is_active=True
    ))
    
    dr_system.add_data_center(DataCenter(
        id="dc-shanghai",
        name="上海数据中心",
        region="cn-east",
        zone="shanghai",
        is_primary=False,
        is_active=True,
        latency_to_primary=30.0
    ))
    
    dr_system.add_data_center(DataCenter(
        id="dc-shenzhen",
        name="深圳数据中心",
        region="cn-south",
        zone="shenzhen",
        is_primary=False,
        is_active=True,
        latency_to_primary=50.0
    ))
    
    # 配置复制
    dr_system.configure_replication(ReplicationConfig(
        mode=ReplicationMode.SYNCHRONOUS,
        lag_threshold_ms=100,
        conflict_resolution="timestamp_wins"
    ))
    
    # 模拟请求路由
    print("\n--- 请求路由测试 ---")
    test_users = ["user_001", "user_002", "user_003", "user_004"]
    for user in test_users:
        dc = dr_system.route_request(user)
        print(f"用户 {user} -> 数据中心: {dc}")
    
    # 模拟故障
    print("\n--- 故障转移测试 ---")
    print(f"当前主数据中心: {dr_system.primary_dc}")
    dr_system.handle_failure("dc-beijing")
    print(f"故障转移后主数据中心: {dr_system.primary_dc}")
    
    # 备份策略
    print("\n--- 备份策略 ---")
    backup = BackupStrategy()
    strategy = backup.get_3_2_1_strategy()
    for i, item in enumerate(strategy, 1):
        print(f"{i}. {item['name']}")
        print(f"   位置: {item['location']}")
        print(f"   类型: {item['type']}")
        print(f"   频率: {item['frequency']}")


if __name__ == '__main__':
    demo_dr_system()
```

### 2.3 容灾架构模式

```python
#!/usr/bin/env python3
"""
容灾架构模式实现
"""

from abc import ABC, abstractmethod
from typing import Dict, List, Optional
from dataclasses import dataclass
import time

@dataclass
class RecoveryResult:
    """恢复结果"""
    success: bool
    rto_achieved: bool
    rpo_achieved: bool
    recovery_time_minutes: float
    data_loss_minutes: float
    message: str


class DRArchitecture(ABC):
    """容灾架构抽象基类"""
    
    def __init__(self, name: str):
        self.name = name
        self.rto_target: int = 0  # 分钟
        self.rpo_target: int = 0  # 分钟
    
    @abstractmethod
    def recover(self) -> RecoveryResult:
        """执行恢复"""
        pass
    
    @abstractmethod
    def calculate_cost(self) -> float:
        """计算成本"""
        pass
    
    def validate_rto(self, actual_time: int) -> bool:
        """验证是否满足 RTO"""
        return actual_time <= self.rto_target
    
    def validate_rpo(self, data_loss: int) -> bool:
        """验证是否满足 RPO"""
        return data_loss <= self.rpo_target


class BackupRestoreArchitecture(DRArchitecture):
    """备份恢复架构"""
    
    def __init__(self):
        super().__init__("Backup & Restore")
        self.rto_target = 480  # 8小时
        self.rpo_target = 1440  # 24小时
    
    def recover(self) -> RecoveryResult:
        """执行备份恢复"""
        print("[*] 执行备份恢复...")
        
        steps = [
            "从备份存储下载备份文件",
            "验证备份完整性",
            "恢复数据库",
            "恢复应用程序",
            "验证系统功能"
        ]
        
        total_time = 0
        for step in steps:
            print(f"  - {step}")
            time.sleep(0.1)  # 模拟时间
            total_time += 60  # 每步60分钟
        
        return RecoveryResult(
            success=True,
            rto_achieved=total_time <= self.rto_target,
            rpo_achieved=True,
            recovery_time_minutes=total_time,
            data_loss_minutes=self.rpo_target,
            message="备份恢复完成"
        )
    
    def calculate_cost(self) -> float:
        """成本估算 (月度)"""
        # 存储成本 + 传输成本
        storage_cost = 500  # 备份存储
        transfer_cost = 100  # 恢复时的传输
        return storage_cost + transfer_cost


class PilotLightArchitecture(DRArchitecture):
    """指示灯架构"""
    
    def __init__(self):
        super().__init__("Pilot Light")
        self.rto_target = 60  # 1小时
        self.rpo_target = 60  # 1小时
    
    def recover(self) -> RecoveryResult:
        """执行指示灯恢复"""
        print("[*] 执行指示灯恢复...")
        
        steps = [
            "启动核心数据库实例 (已配置，未运行)",
            "恢复最新数据",
            "扩展应用服务器实例",
            "更新 DNS 指向",
            "验证服务"
        ]
        
        total_time = 0
        for step in steps:
            print(f"  - {step}")
            time.sleep(0.05)
            total_time += 15
        
        return RecoveryResult(
            success=True,
            rto_achieved=total_time <= self.rto_target,
            rpo_achieved=True,
            recovery_time_minutes=total_time,
            data_loss_minutes=5,
            message="指示灯恢复完成"
        )
    
    def calculate_cost(self) -> float:
        """成本估算"""
        # 数据库运行成本 + 最小应用配置
        db_cost = 800
        app_cost = 200
        return db_cost + app_cost


class WarmStandbyArchitecture(DRArchitecture):
    """温备架构"""
    
    def __init__(self):
        super().__init__("Warm Standby")
        self.rto_target = 15  # 15分钟
        self.rpo_target = 5   # 5分钟
    
    def recover(self) -> RecoveryResult:
        """执行温备恢复"""
        print("[*] 执行温备恢复...")
        
        steps = [
            "检测主站点故障",
            "提升备站点为主",
            "扩展备站点容量",
            "切换流量到备站点",
            "验证服务"
        ]
        
        total_time = 0
        for step in steps:
            print(f"  - {step}")
            time.sleep(0.02)
            total_time += 3
        
        return RecoveryResult(
            success=True,
            rto_achieved=total_time <= self.rto_target,
            rpo_achieved=True,
            recovery_time_minutes=total_time,
            data_loss_minutes=1,
            message="温备切换完成"
        )
    
    def calculate_cost(self) -> float:
        """成本估算"""
        # 50% 主站点配置运行
        return 2500


class HotStandbyArchitecture(DRArchitecture):
    """热备/双活架构"""
    
    def __init__(self):
        super().__init__("Hot Standby / Active-Active")
        self.rto_target = 0   # 接近0
        self.rpo_target = 0   # 接近0
    
    def recover(self) -> RecoveryResult:
        """执行热备恢复（自动故障转移）"""
        print("[*] 执行热备自动故障转移...")
        
        steps = [
            "健康检查失败检测",
            "自动移除故障节点",
            "流量自动路由到健康节点",
            "启动新实例补充容量"
        ]
        
        total_time = 0
        for step in steps:
            print(f"  - {step}")
            time.sleep(0.01)
            total_time += 0.5
        
        return RecoveryResult(
            success=True,
            rto_achieved=total_time <= self.rto_target,
            rpo_achieved=True,
            recovery_time_minutes=total_time,
            data_loss_minutes=0,
            message="自动故障转移完成"
        )
    
    def calculate_cost(self) -> float:
        """成本估算"""
        # 100% 主站点配置 × 2
        return 5000


def compare_architectures():
    """比较不同架构"""
    architectures = [
        BackupRestoreArchitecture(),
        PilotLightArchitecture(),
        WarmStandbyArchitecture(),
        HotStandbyArchitecture()
    ]
    
    print("=" * 80)
    print("容灾架构对比")
    print("=" * 80)
    print(f"{'架构':<25} {'RTO':<10} {'RPO':<10} {'月度成本':<12} {'复杂度'}")
    print("-" * 80)
    
    for arch in architectures:
        cost = arch.calculate_cost()
        complexity = {
            "Backup & Restore": "低",
            "Pilot Light": "中低",
            "Warm Standby": "中",
            "Hot Standby / Active-Active": "高"
        }.get(arch.name, "未知")
        
        rto_str = f"{arch.rto_target}分钟" if arch.rto_target > 0 else "<1分钟"
        rpo_str = f"{arch.rpo_target}分钟" if arch.rpo_target > 0 else "<1分钟"
        
        print(f"{arch.name:<25} {rto_str:<10} {rpo_str:<10} ${cost:<11} {complexity}")
    
    print("\n" + "=" * 80)
    print("恢复测试")
    print("=" * 80)
    
    for arch in architectures:
        print(f"\n>>> {arch.name}")
        result = arch.recover()
        print(f"  恢复时间: {result.recovery_time_minutes}分钟")
        print(f"  数据丢失: {result.data_loss_minutes}分钟")
        print(f"  满足 RTO: {'是' if result.rto_achieved else '否'}")
        print(f"  满足 RPO: {'是' if result.rpo_achieved else '否'}")


if __name__ == '__main__':
    compare_architectures()
```

---

## 3. 技术实践

### 3.1 自动化灾备演练

```python
#!/usr/bin/env python3
"""
自动化灾难恢复演练系统
"""

import json
import time
import smtplib
from datetime import datetime
from typing import Dict, List, Callable
from dataclasses import dataclass, asdict
from enum import Enum

class DrillType(Enum):
    """演练类型"""
    TABLETOP = "桌面推演"  # 讨论式
    FUNCTIONAL = "功能演练"  # 部分系统
    FULL_SCALE = "全面演练"  # 完整切换

class DrillStatus(Enum):
    """演练状态"""
    PLANNED = "已计划"
    IN_PROGRESS = "进行中"
    COMPLETED = "已完成"
    FAILED = "失败"

@dataclass
class DrillStep:
    """演练步骤"""
    id: int
    name: str
    description: str
    expected_duration: int  # 分钟
    validation_checks: List[str]
    rollback_procedure: str

@dataclass
class DrillResult:
    """演练结果"""
    step_id: int
    start_time: datetime
    end_time: datetime
    success: bool
    actual_duration: int
    notes: str
    logs: List[str]

class DisasterRecoveryDrill:
    """灾备演练管理器"""
    
    def __init__(self, name: str, drill_type: DrillType):
        self.name = name
        self.drill_type = drill_type
        self.status = DrillStatus.PLANNED
        self.steps: List[DrillStep] = []
        self.results: List[DrillResult] = []
        self.start_time: datetime = None
        self.end_time: datetime = None
        self.notifications: List[str] = []
    
    def add_step(self, step: DrillStep):
        """添加演练步骤"""
        self.steps.append(step)
    
    def run_drill(self, skip_actual_changes: bool = True):
        """执行演练"""
        print(f"\n{'='*60}")
        print(f"开始灾备演练: {self.name}")
        print(f"类型: {self.drill_type.value}")
        print(f"{'='*60}")
        
        self.status = DrillStatus.IN_PROGRESS
        self.start_time = datetime.now()
        
        # 发送通知
        self._send_notification(f"灾备演练开始: {self.name}")
        
        try:
            for step in self.steps:
                print(f"\n[步骤 {step.id}] {step.name}")
                print(f"  描述: {step.description}")
                print(f"  预期耗时: {step.expected_duration}分钟")
                
                step_start = datetime.now()
                
                # 执行步骤
                if skip_actual_changes:
                    print("  [模拟模式] 跳过实际变更")
                    time.sleep(0.1)
                    success = True
                    logs = ["模拟执行成功"]
                else:
                    success, logs = self._execute_step(step)
                
                step_end = datetime.now()
                duration = int((step_end - step_start).total_seconds() / 60)
                
                result = DrillResult(
                    step_id=step.id,
                    start_time=step_start,
                    end_time=step_end,
                    success=success,
                    actual_duration=duration,
                    notes="" if success else "需要关注",
                    logs=logs
                )
                self.results.append(result)
                
                status_icon = "✓" if success else "✗"
                print(f"  结果: {status_icon} (实际耗时: {duration}分钟)")
                
                if not success:
                    print(f"  [!] 步骤失败，执行回滚...")
                    self._rollback_step(step)
            
            self.status = DrillStatus.COMPLETED
            
        except Exception as e:
            self.status = DrillStatus.FAILED
            print(f"[!] 演练失败: {e}")
        
        self.end_time = datetime.now()
        self._send_notification(f"灾备演练完成: {self.name} - {self.status.value}")
        
        return self.generate_report()
    
    def _execute_step(self, step: DrillStep) -> (bool, List[str]):
        """执行单个步骤（实际实现中）"""
        # 这里实现具体的执行逻辑
        logs = []
        
        for check in step.validation_checks:
            # 执行检查
            logs.append(f"执行检查: {check}")
            # ...
        
        return True, logs
    
    def _rollback_step(self, step: DrillStep):
        """回滚步骤"""
        print(f"  回滚: {step.rollback_procedure}")
    
    def _send_notification(self, message: str):
        """发送通知"""
        self.notifications.append(f"{datetime.now()}: {message}")
        print(f"[通知] {message}")
    
    def generate_report(self) -> Dict:
        """生成演练报告"""
        total_steps = len(self.steps)
        successful_steps = len([r for r in self.results if r.success])
        
        report = {
            "drill_name": self.name,
            "drill_type": self.drill_type.value,
            "status": self.status.value,
            "start_time": self.start_time.isoformat() if self.start_time else None,
            "end_time": self.end_time.isoformat() if self.end_time else None,
            "total_duration_minutes": int((self.end_time - self.start_time).total_seconds() / 60) if self.end_time else 0,
            "summary": {
                "total_steps": total_steps,
                "successful_steps": successful_steps,
                "failed_steps": total_steps - successful_steps,
                "success_rate": f"{successful_steps/total_steps*100:.1f}%" if total_steps > 0 else "N/A"
            },
            "steps_detail": [
                {
                    "step_id": r.step_id,
                    "success": r.success,
                    "expected_duration": next(s.expected_duration for s in self.steps if s.id == r.step_id),
                    "actual_duration": r.actual_duration,
                    "notes": r.notes,
                    "logs": r.logs
                }
                for r in self.results
            ],
            "improvements": self._identify_improvements()
        }
        
        return report
    
    def _identify_improvements(self) -> List[str]:
        """识别改进点"""
        improvements = []
        
        for result in self.results:
            step = next(s for s in self.steps if s.id == result.step_id)
            
            # 检查是否超时
            if result.actual_duration > step.expected_duration:
                improvements.append(
                    f"步骤 {step.id} ({step.name}) 超时: "
                    f"预期 {step.expected_duration}分钟, 实际 {result.actual_duration}分钟"
                )
            
            # 检查是否失败
            if not result.success:
                improvements.append(
                    f"步骤 {step.id} ({step.name}) 执行失败，需要改进流程"
                )
        
        return improvements
    
    def export_report(self, filename: str):
        """导出报告到文件"""
        report = self.generate_report()
        with open(filename, 'w', encoding='utf-8') as f:
            json.dump(report, f, indent=2, ensure_ascii=False)
        print(f"\n[+] 报告已导出: {filename}")


def create_sample_drill() -> DisasterRecoveryDrill:
    """创建示例演练"""
    drill = DisasterRecoveryDrill(
        name="2024 Q1 数据中心故障切换演练",
        drill_type=DrillType.FUNCTIONAL
    )
    
    # 添加演练步骤
    drill.add_step(DrillStep(
        id=1,
        name="故障检测",
        description="模拟主数据中心故障检测",
        expected_duration=2,
        validation_checks=[
            "监控系统告警触发",
            "健康检查失败确认"
        ],
        rollback_procedure="恢复主数据中心连接"
    ))
    
    drill.add_step(DrillStep(
        id=2,
        name="决策与通知",
        description="评估情况并启动灾备流程",
        expected_duration=5,
        validation_checks=[
            "灾备团队通知完成",
            "切换决策确认"
        ],
        rollback_procedure="取消切换，恢复主站点"
    ))
    
    drill.add_step(DrillStep(
        id=3,
        name="数据库切换",
        description="将数据库主节点切换到备站点",
        expected_duration=10,
        validation_checks=[
            "备库提升为主库",
            "数据同步状态确认"
        ],
        rollback_procedure="回滚数据库角色"
    ))
    
    drill.add_step(DrillStep(
        id=4,
        name="应用切换",
        description="切换应用到备数据中心",
        expected_duration=5,
        validation_checks=[
            "应用实例启动完成",
            "配置更新完成"
        ],
        rollback_procedure="停止备站点应用"
    ))
    
    drill.add_step(DrillStep(
        id=5,
        name="流量切换",
        description="将用户流量切换到备数据中心",
        expected_duration=3,
        validation_checks=[
            "DNS 切换完成",
            "负载均衡器更新"
        ],
        rollback_procedure="切回主站点流量"
    ))
    
    drill.add_step(DrillStep(
        id=6,
        name="验证与监控",
        description="验证服务可用性和功能正常",
        expected_duration=10,
        validation_checks=[
            "核心业务功能测试",
            "监控指标正常"
        ],
        rollback_procedure="根据具体情况执行回滚"
    ))
    
    return drill


def main():
    """主函数"""
    drill = create_sample_drill()
    report = drill.run_drill(skip_actual_changes=True)
    
    # 打印报告摘要
    print("\n" + "=" * 60)
    print("演练报告摘要")
    print("=" * 60)
    print(f"演练名称: {report['drill_name']}")
    print(f"演练类型: {report['drill_type']}")
    print(f"演练状态: {report['status']}")
    print(f"总耗时: {report['total_duration_minutes']} 分钟")
    print(f"\n步骤统计:")
    print(f"  总计: {report['summary']['total_steps']}")
    print(f"  成功: {report['summary']['successful_steps']}")
    print(f"  失败: {report['summary']['failed_steps']}")
    print(f"  成功率: {report['summary']['success_rate']}")
    
    if report['improvements']:
        print(f"\n改进建议:")
        for imp in report['improvements']:
            print(f"  - {imp}")
    
    # 导出报告
    drill.export_report("drill_report.json")


if __name__ == '__main__':
    main()
```

### 3.2 备份验证与恢复测试

```python
#!/usr/bin/env python3
"""
备份验证与恢复测试自动化
"""

import hashlib
import os
from datetime import datetime, timedelta
from typing import Dict, List, Optional
from dataclasses import dataclass
from enum import Enum

class BackupStatus(Enum):
    VALID = "有效"
    INVALID = "无效"
    CORRUPTED = "损坏"
    EXPIRED = "过期"
    UNKNOWN = "未知"

@dataclass
class Backup:
    """备份记录"""
    id: str
    name: str
    source: str
    destination: str
    created_at: datetime
    size_bytes: int
    checksum: str
    retention_days: int
    backup_type: str  # full, incremental, differential
    status: BackupStatus = BackupStatus.UNKNOWN
    last_verified: Optional[datetime] = None


class BackupValidator:
    """备份验证器"""
    
    def __init__(self):
        self.validation_results: Dict[str, Dict] = {}
    
    def calculate_checksum(self, file_path: str) -> str:
        """计算文件校验和"""
        sha256_hash = hashlib.sha256()
        with open(file_path, "rb") as f:
            for byte_block in iter(lambda: f.read(4096), b""):
                sha256_hash.update(byte_block)
        return sha256_hash.hexdigest()
    
    def verify_integrity(self, backup: Backup, file_path: str) -> bool:
        """验证备份完整性"""
        if not os.path.exists(file_path):
            return False
        
        current_checksum = self.calculate_checksum(file_path)
        return current_checksum == backup.checksum
    
    def verify_restoration(self, backup: Backup, 
                          restore_path: str,
                          test_queries: List[str] = None) -> Dict:
        """验证恢复过程"""
        result = {
            'backup_id': backup.id,
            'restore_tested': False,
            'restore_time_seconds': 0,
            'integrity_check': False,
            'functionality_check': False,
            'errors': []
        }
        
        # 1. 执行恢复
        print(f"[*] 测试恢复备份: {backup.name}")
        start_time = datetime.now()
        
        try:
            # 模拟恢复过程
            # 实际实现中调用具体的恢复命令
            # 例如：tar -xzf backup.tar.gz -C restore_path
            # 或：pg_restore -d test_db backup.dump
            
            result['restore_tested'] = True
            result['restore_time_seconds'] = (datetime.now() - start_time).total_seconds()
            
        except Exception as e:
            result['errors'].append(f"恢复失败: {str(e)}")
            return result
        
        # 2. 验证完整性
        print(f"[*] 验证恢复数据完整性")
        # 验证关键文件存在、数据库表结构正确等
        result['integrity_check'] = True
        
        # 3. 功能测试
        print(f"[*] 执行功能测试")
        if test_queries:
            for query in test_queries:
                print(f"  执行: {query}")
                # 执行测试查询
        result['functionality_check'] = True
        
        return result
    
    def run_backup_audit(self, backups: List[Backup]) -> Dict:
        """执行备份审计"""
        audit_results = {
            'audit_time': datetime.now().isoformat(),
            'total_backups': len(backups),
            'valid_backups': 0,
            'invalid_backups': 0,
            'expired_backups': 0,
            'size_analysis': {
                'total_size_gb': 0,
                'average_size_gb': 0
            },
            'recommendations': []
        }
        
        total_size = 0
        
        for backup in backups:
            # 检查过期
            expiry_date = backup.created_at + timedelta(days=backup.retention_days)
            if datetime.now() > expiry_date:
                backup.status = BackupStatus.EXPIRED
                audit_results['expired_backups'] += 1
                continue
            
            # 验证状态
            if backup.status == BackupStatus.VALID:
                audit_results['valid_backups'] += 1
            else:
                audit_results['invalid_backups'] += 1
            
            total_size += backup.size_bytes
        
        # 统计分析
        total_gb = total_size / (1024**3)
        audit_results['size_analysis']['total_size_gb'] = round(total_gb, 2)
        audit_results['size_analysis']['average_size_gb'] = round(total_gb / len(backups), 2) if backups else 0
        
        # 生成建议
        if audit_results['invalid_backups'] > 0:
            audit_results['recommendations'].append(
                f"有 {audit_results['invalid_backups']} 个备份需要重新验证"
            )
        
        if audit_results['expired_backups'] > 0:
            audit_results['recommendations'].append(
                f"有 {audit_results['expired_backups']} 个过期备份需要清理"
            )
        
        # 3-2-1 规则检查
        locations = set(b.destination for b in backups)
        if len(locations) < 2:
            audit_results['recommendations'].append(
                "备份不符合 3-2-1 规则：需要至少 2 种不同存储介质"
            )
        
        return audit_results


def demo_backup_validation():
    """演示备份验证"""
    print("=" * 60)
    print("备份验证与恢复测试演示")
    print("=" * 60)
    
    validator = BackupValidator()
    
    # 创建示例备份记录
    backups = [
        Backup(
            id="BKP-001",
            name="生产数据库全量备份",
            source="prod-db-01",
            destination="s3://backup-bucket/db/",
            created_at=datetime.now() - timedelta(days=1),
            size_bytes=10737418240,  # 10GB
            checksum="a1b2c3d4e5f6...",
            retention_days=30,
            backup_type="full",
            status=BackupStatus.VALID,
            last_verified=datetime.now() - timedelta(days=1)
        ),
        Backup(
            id="BKP-002",
            name="应用代码仓库备份",
            source="gitlab-server",
            destination="nas://backup-nas/git/",
            created_at=datetime.now() - timedelta(days=2),
            size_bytes=536870912,  # 512MB
            checksum="f6e5d4c3b2a1...",
            retention_days=90,
            backup_type="full",
            status=BackupStatus.VALID,
            last_verified=datetime.now() - timedelta(days=2)
        ),
        Backup(
            id="BKP-003",
            name="过期备份示例",
            source="old-server",
            destination="tape://vault/",
            created_at=datetime.now() - timedelta(days=100),
            size_bytes=2147483648,  # 2GB
            checksum="...",
            retention_days=30,
            backup_type="full",
            status=BackupStatus.EXPIRED
        ),
    ]
    
    # 执行审计
    print("\n执行备份审计...")
    audit = validator.run_backup_audit(backups)
    
    print(f"\n审计结果:")
    print(f"  备份总数: {audit['total_backups']}")
    print(f"  有效备份: {audit['valid_backups']}")
    print(f"  无效备份: {audit['invalid_backups']}")
    print(f"  过期备份: {audit['expired_backups']}")
    print(f"  总大小: {audit['size_analysis']['total_size_gb']} GB")
    
    if audit['recommendations']:
        print(f"\n改进建议:")
        for rec in audit['recommendations']:
            print(f"  - {rec}")
    
    # 测试恢复
    print("\n--- 恢复测试 ---")
    test_result = validator.verify_restoration(
        backups[0],
        "/tmp/restore_test",
        test_queries=["SELECT COUNT(*) FROM users", "SELECT MAX(created_at) FROM orders"]
    )
    
    print(f"恢复测试完成:")
    print(f"  恢复耗时: {test_result['restore_time_seconds']:.2f} 秒")
    print(f"  完整性检查: {'通过' if test_result['integrity_check'] else '失败'}")
    print(f"  功能检查: {'通过' if test_result['functionality_check'] else '失败'}")


if __name__ == '__main__':
    demo_backup_validation()
```

---

## 4. 资源索引

### 4.1 标准与框架

| 标准 | 名称 | 说明 |
|------|------|------|
| ISO 22301 | 业务连续性管理体系 | 国际标准 |
| ISO 27031 | ICT 业务连续性指南 | 信息安全相关 |
| NIST SP 800-34 | IT 应急计划指南 | 美国标准 |
| BCI GPG | 业务连续性良好实践指南 | BCI 发布 |

### 4.2 云服务

| 厂商 | 灾备服务 | 特点 |
|------|----------|------|
| AWS | AWS Disaster Recovery | CloudEndure 集成 |
| Azure | Azure Site Recovery | 异构环境支持 |
| GCP | Cloud Disaster Recovery | 自动化编排 |
| 阿里云 | 灾备解决方案 | 混合云支持 |
| 腾讯云 | 容灾解决方案 | 一键切换 |

---

## 5. 关联知识

```
A04_Security_Quality/B02_Reliability_Evaluation/
├── C03_Disaster_Recovery (本模块)
│   ├── → C01_Chaos_Engineering: 灾备能力验证
│   └── → C02_Failure_Modes_Analysis: 故障场景分析
│
├── → B01_Information_Security: 备份数据安全
├── → B03_Maintenance_Ops: 灾备运维
└── → B04_Quality_Assurance: 恢复测试
```

---

## 6. 学习建议

### 6.1 学习路径

1. **基础概念**: 理解 RTO、RPO、MTD 等关键指标
2. **架构学习**: 掌握各种灾备架构的优缺点
3. **工具实践**: 使用云平台搭建多活环境
4. **演练组织**: 参与或组织灾备演练

### 6.2 认证推荐

- **CBCP**: 认证业务连续性专家
- **MBCP**: 主业务连续性专家
- **AWS/Azure/GCP 灾备专项认证**

---

*最后更新: 2024-01-30*
