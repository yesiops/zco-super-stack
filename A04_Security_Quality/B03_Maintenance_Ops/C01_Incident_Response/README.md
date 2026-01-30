# C01 事件响应 (Incident Response)

## 1. 主题定位

### 1.1 定义
事件响应（Incident Response）是组织为处理安全事件（如网络攻击、数据泄露、系统入侵等）而建立的一套结构化流程。目标是快速检测、分析、遏制、根除和恢复，最小化安全事件的影响。

### 1.2 核心价值
- **快速响应**: 缩短从检测到响应的时间
- **影响控制**: 限制安全事件造成的损失
- **经验学习**: 从事故中吸取教训，改进防御
- **合规要求**: 满足法规和保险要求

### 1.3 事件严重程度分级

| 级别 | 名称 | 定义 | 响应时间 | 示例 |
|------|------|------|----------|------|
| P0 | 紧急 | 业务全面中断或数据大规模泄露 | 15分钟 | 勒索软件、核心数据库被删 |
| P1 | 严重 | 关键业务受影响或敏感数据泄露 | 1小时 | 生产环境入侵、客户数据泄露 |
| P2 | 高 | 非关键业务受影响 | 4小时 | 开发环境入侵、内部工具不可用 |
| P3 | 中 | 轻微影响，有规避方案 | 24小时 | 扫描告警、低危漏洞 |
| P4 | 低 | 信息性事件 | 48小时 | 配置建议、安全提示 |

---

## 2. 核心概念

### 2.1 NIST 事件响应生命周期

```
┌─────────────────────────────────────────────────────────────────┐
│                    事件响应生命周期                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│    ┌──────────┐    ┌──────────┐    ┌──────────┐                │
│    │  准备    │───>│  检测与  │───>│  遏制、  │                │
│    │          │    │  分析    │    │  根除    │                │
│    └──────────┘    └──────────┘    └────┬─────┘                │
│         ^                               │                       │
│         │                               v                       │
│    ┌────┴─────┐                   ┌──────────┐                 │
│    │  事后    │<──────────────────│  恢复    │                 │
│    │  活动    │                   │          │                 │
│    └──────────┘                   └──────────┘                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```python
#!/usr/bin/env python3
"""
事件响应生命周期实现
"""

from dataclasses import dataclass, field
from typing import List, Dict, Optional, Callable
from datetime import datetime
from enum import Enum
import json
import uuid

class IncidentPhase(Enum):
    """事件处理阶段"""
    PREPARATION = "准备"
    DETECTION = "检测"
    ANALYSIS = "分析"
    CONTAINMENT = "遏制"
    ERADICATION = "根除"
    RECOVERY = "恢复"
    POST_INCIDENT = "事后活动"

class IncidentStatus(Enum):
    """事件状态"""
    OPEN = "开放"
    IN_PROGRESS = "处理中"
    CONTAINED = "已遏制"
    RESOLVED = "已解决"
    CLOSED = "已关闭"

class IncidentSeverity(Enum):
    """事件严重程度"""
    CRITICAL = "P0-紧急"
    HIGH = "P1-严重"
    MEDIUM = "P2-高"
    LOW = "P3-中"
    INFORMATIONAL = "P4-低"

@dataclass
class TimelineEntry:
    """时间线条目"""
    timestamp: datetime
    phase: IncidentPhase
    action: str
    actor: str
    notes: str = ""

@dataclass
class Incident:
    """安全事件"""
    id: str
    title: str
    description: str
    severity: IncidentSeverity
    status: IncidentStatus
    reported_by: str
    assigned_to: Optional[str] = None
    
    created_at: datetime = field(default_factory=datetime.now)
    detected_at: Optional[datetime] = None
    contained_at: Optional[datetime] = None
    resolved_at: Optional[datetime] = None
    closed_at: Optional[datetime] = None
    
    timeline: List[TimelineEntry] = field(default_factory=list)
    artifacts: List[Dict] = field(default_factory=list)
    iocs: List[Dict] = field(default_factory=list)  # Indicators of Compromise
    affected_systems: List[str] = field(default_factory=list)
    
    root_cause: str = ""
    lessons_learned: str = ""
    preventive_measures: List[str] = field(default_factory=list)


class IncidentResponseLifecycle:
    """事件响应生命周期管理器"""
    
    def __init__(self):
        self.incidents: Dict[str, Incident] = {}
        self.playbooks: Dict[str, Callable] = {}
    
    def register_playbook(self, incident_type: str, playbook: Callable):
        """注册响应手册"""
        self.playbooks[incident_type] = playbook
    
    def create_incident(self, title: str, description: str, 
                       severity: IncidentSeverity,
                       reported_by: str) -> Incident:
        """创建新事件"""
        incident_id = f"INC-{uuid.uuid4().hex[:8].upper()}"
        
        incident = Incident(
            id=incident_id,
            title=title,
            description=description,
            severity=severity,
            status=IncidentStatus.OPEN,
            reported_by=reported_by
        )
        
        # 添加创建时间线
        incident.timeline.append(TimelineEntry(
            timestamp=datetime.now(),
            phase=IncidentPhase.DETECTION,
            action="事件创建",
            actor=reported_by,
            notes=f"严重程度: {severity.value}"
        ))
        
        self.incidents[incident_id] = incident
        
        # 触发通知
        self._notify_creation(incident)
        
        return incident
    
    def _notify_creation(self, incident: Incident):
        """通知事件创建"""
        print(f"\n[!] 新安全事件创建")
        print(f"    ID: {incident.id}")
        print(f"    标题: {incident.title}")
        print(f"    严重程度: {incident.severity.value}")
        print(f"    报告人: {incident.reported_by}")
    
    def assign_incident(self, incident_id: str, assignee: str):
        """分配事件"""
        incident = self.incidents.get(incident_id)
        if not incident:
            raise ValueError(f"事件不存在: {incident_id}")
        
        incident.assigned_to = assignee
        incident.status = IncidentStatus.IN_PROGRESS
        
        incident.timeline.append(TimelineEntry(
            timestamp=datetime.now(),
            phase=IncidentPhase.ANALYSIS,
            action="事件分配",
            actor="系统",
            notes=f"分配给: {assignee}"
        ))
        
        print(f"[+] 事件 {incident_id} 已分配给 {assignee}")
    
    def add_ioc(self, incident_id: str, ioc_type: str, value: str, 
                source: str, confidence: str = "medium"):
        """添加 IOC"""
        incident = self.incidents.get(incident_id)
        if not incident:
            raise ValueError(f"事件不存在: {incident_id}")
        
        ioc = {
            "type": ioc_type,  # ip, domain, hash, url, email
            "value": value,
            "source": source,
            "confidence": confidence,
            "added_at": datetime.now().isoformat()
        }
        
        incident.iocs.append(ioc)
        
        incident.timeline.append(TimelineEntry(
            timestamp=datetime.now(),
            phase=IncidentPhase.ANALYSIS,
            action="添加 IOC",
            actor="分析师",
            notes=f"{ioc_type}: {value}"
        ))
    
    def contain_incident(self, incident_id: str, actions: List[str]):
        """遏制事件"""
        incident = self.incidents.get(incident_id)
        if not incident:
            raise ValueError(f"事件不存在: {incident_id}")
        
        incident.status = IncidentStatus.CONTAINED
        incident.contained_at = datetime.now()
        
        for action in actions:
            incident.timeline.append(TimelineEntry(
                timestamp=datetime.now(),
                phase=IncidentPhase.CONTAINMENT,
                action="遏制措施",
                actor=incident.assigned_to or "系统",
                notes=action
            ))
        
        print(f"[+] 事件 {incident_id} 已遏制")
    
    def resolve_incident(self, incident_id: str, root_cause: str,
                        preventive_measures: List[str]):
        """解决事件"""
        incident = self.incidents.get(incident_id)
        if not incident:
            raise ValueError(f"事件不存在: {incident_id}")
        
        incident.status = IncidentStatus.RESOLVED
        incident.resolved_at = datetime.now()
        incident.root_cause = root_cause
        incident.preventive_measures = preventive_measures
        
        incident.timeline.append(TimelineEntry(
            timestamp=datetime.now(),
            phase=IncidentPhase.RECOVERY,
            action="事件解决",
            actor=incident.assigned_to or "系统",
            notes=f"根本原因: {root_cause}"
        ))
        
        print(f"[+] 事件 {incident_id} 已解决")
    
    def close_incident(self, incident_id: str, lessons_learned: str):
        """关闭事件"""
        incident = self.incidents.get(incident_id)
        if not incident:
            raise ValueError(f"事件不存在: {incident_id}")
        
        incident.status = IncidentStatus.CLOSED
        incident.closed_at = datetime.now()
        incident.lessons_learned = lessons_learned
        
        incident.timeline.append(TimelineEntry(
            timestamp=datetime.now(),
            phase=IncidentPhase.POST_INCIDENT,
            action="事件关闭",
            actor="系统",
            notes="完成事后总结"
        ))
        
        print(f"[+] 事件 {incident_id} 已关闭")
    
    def generate_report(self, incident_id: str) -> Dict:
        """生成事件报告"""
        incident = self.incidents.get(incident_id)
        if not incident:
            raise ValueError(f"事件不存在: {incident_id}")
        
        # 计算 MTTR
        if incident.resolved_at and incident.created_at:
            mttr = (incident.resolved_at - incident.created_at).total_seconds() / 3600
        else:
            mttr = None
        
        report = {
            "incident_id": incident.id,
            "title": incident.title,
            "severity": incident.severity.value,
            "status": incident.status.value,
            "timeline": {
                "created": incident.created_at.isoformat(),
                "detected": incident.detected_at.isoformat() if incident.detected_at else None,
                "contained": incident.contained_at.isoformat() if incident.contained_at else None,
                "resolved": incident.resolved_at.isoformat() if incident.resolved_at else None,
                "closed": incident.closed_at.isoformat() if incident.closed_at else None,
                "mttr_hours": round(mttr, 2) if mttr else None
            },
            "affected_systems": incident.affected_systems,
            "iocs": incident.iocs,
            "root_cause": incident.root_cause,
            "lessons_learned": incident.lessons_learned,
            "preventive_measures": incident.preventive_measures,
            "detailed_timeline": [
                {
                    "time": entry.timestamp.isoformat(),
                    "phase": entry.phase.value,
                    "action": entry.action,
                    "actor": entry.actor,
                    "notes": entry.notes
                }
                for entry in incident.timeline
            ]
        }
        
        return report


def demo_incident_response():
    """演示事件响应流程"""
    print("=" * 60)
    print("事件响应生命周期演示")
    print("=" * 60)
    
    ir = IncidentResponseLifecycle()
    
    # 1. 创建事件
    print("\n--- 阶段 1: 检测 ---")
    incident = ir.create_incident(
        title="可疑登录活动",
        description="检测到多个来自异常地理位置的管理员登录尝试",
        severity=IncidentSeverity.HIGH,
        reported_by="SIEM 系统"
    )
    
    # 2. 分配和分析
    print("\n--- 阶段 2: 分析 ---")
    ir.assign_incident(incident.id, "安全分析师-张三")
    
    # 添加 IOC
    ir.add_ioc(incident.id, "ip", "192.168.100.50", "防火墙日志", "high")
    ir.add_ioc(incident.id, "domain", "evil-site.com", "威胁情报", "medium")
    
    incident.affected_systems = ["admin-server-01", "vpn-gateway"]
    
    # 3. 遏制
    print("\n--- 阶段 3: 遏制 ---")
    ir.contain_incident(incident.id, [
        "阻断可疑 IP 192.168.100.50",
        "重置受影响管理员密码",
        "启用 MFA 强制策略"
    ])
    
    # 4. 解决
    print("\n--- 阶段 4: 恢复 ---")
    ir.resolve_incident(incident.id, 
        root_cause="使用泄露凭证的暴力破解攻击",
        preventive_measures=[
            "实施登录失败锁定策略",
            "部署地理位置访问控制",
            "加强密码复杂度要求",
            "定期安全培训"
        ]
    )
    
    # 5. 关闭
    print("\n--- 阶段 5: 事后活动 ---")
    ir.close_incident(incident.id, 
        lessons_learned="需要加强对异常登录模式的实时监控，缩短检测时间"
    )
    
    # 生成报告
    print("\n--- 事件报告 ---")
    report = ir.generate_report(incident.id)
    print(json.dumps(report, indent=2, ensure_ascii=False))


if __name__ == '__main__':
    demo_incident_response()
```

### 2.2 数字取证基础

```python
#!/usr/bin/env python3
"""
数字取证基础工具
"""

import hashlib
import os
import json
from datetime import datetime
from typing import Dict, List, Optional
from dataclasses import dataclass, asdict

@dataclass
class Evidence:
    """证据项"""
    id: str
    type: str  # file, memory, network, log
    source: str
    collected_at: datetime
    collector: str
    hash_md5: str
    hash_sha256: str
    size: int
    path: str
    metadata: Dict
    chain_of_custody: List[Dict]

class DigitalForensics:
    """数字取证工具"""
    
    @staticmethod
    def calculate_hashes(file_path: str) -> Dict[str, str]:
        """计算文件哈希值"""
        md5_hash = hashlib.md5()
        sha256_hash = hashlib.sha256()
        
        with open(file_path, 'rb') as f:
            while chunk := f.read(8192):
                md5_hash.update(chunk)
                sha256_hash.update(chunk)
        
        return {
            'md5': md5_hash.hexdigest(),
            'sha256': sha256_hash.hexdigest()
        }
    
    @staticmethod
    def collect_file_evidence(file_path: str, collector: str) -> Evidence:
        """收集文件证据"""
        if not os.path.exists(file_path):
            raise FileNotFoundError(f"文件不存在: {file_path}")
        
        # 计算哈希
        hashes = DigitalForensics.calculate_hashes(file_path)
        
        # 获取文件元数据
        stat = os.stat(file_path)
        
        evidence = Evidence(
            id=f"EVD-{hashlib.sha256(file_path.encode()).hexdigest()[:12].upper()}",
            type="file",
            source=file_path,
            collected_at=datetime.now(),
            collector=collector,
            hash_md5=hashes['md5'],
            hash_sha256=hashes['sha256'],
            size=stat.st_size,
            path=file_path,
            metadata={
                'created': datetime.fromtimestamp(stat.st_ctime).isoformat(),
                'modified': datetime.fromtimestamp(stat.st_mtime).isoformat(),
                'accessed': datetime.fromtimestamp(stat.st_atime).isoformat(),
                'permissions': oct(stat.st_mode)[-3:]
            },
            chain_of_custody=[{
                'timestamp': datetime.now().isoformat(),
                'action': 'COLLECTED',
                'actor': collector,
                'location': file_path
            }]
        )
        
        return evidence
    
    @staticmethod
    def verify_integrity(evidence: Evidence, file_path: str) -> bool:
        """验证证据完整性"""
        current_hashes = DigitalForensics.calculate_hashes(file_path)
        return (
            current_hashes['md5'] == evidence.hash_md5 and
            current_hashes['sha256'] == evidence.hash_sha256
        )
    
    @staticmethod
    def analyze_logs(log_file: str, patterns: List[str]) -> List[Dict]:
        """分析日志文件"""
        findings = []
        
        with open(log_file, 'r', encoding='utf-8', errors='ignore') as f:
            for line_num, line in enumerate(f, 1):
                for pattern in patterns:
                    if pattern.lower() in line.lower():
                        findings.append({
                            'line_number': line_num,
                            'content': line.strip(),
                            'pattern_matched': pattern,
                            'timestamp': DigitalForensics._extract_timestamp(line)
                        })
        
        return findings
    
    @staticmethod
    def _extract_timestamp(line: str) -> Optional[str]:
        """从日志行提取时间戳（简化版）"""
        import re
        # 匹配常见的日期时间格式
        patterns = [
            r'\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}',
            r'\d{2}/\w{3}/\d{4}:\d{2}:\d{2}:\d{2}',
            r'\w{3} \d{2} \d{2}:\d{2}:\d{2}'
        ]
        
        for pattern in patterns:
            match = re.search(pattern, line)
            if match:
                return match.group()
        return None


# 内存分析模拟
class MemoryForensics:
    """内存取证"""
    
    @staticmethod
    def extract_processes(memory_dump: str) -> List[Dict]:
        """提取进程列表"""
        # 实际实现需要使用 Volatility 等工具
        print(f"[*] 分析内存镜像: {memory_dump}")
        
        # 模拟返回进程列表
        return [
            {'pid': 1234, 'name': 'svchost.exe', 'ppid': 500},
            {'pid': 5678, 'name': 'malware.exe', 'ppid': 1234},  # 可疑进程
            {'pid': 9999, 'name': 'notepad.exe', 'ppid': 1},
        ]
    
    @staticmethod
    def find_malicious_patterns(memory_dump: str) -> List[Dict]:
        """查找恶意特征"""
        # 实际实现中搜索已知的恶意软件特征码
        patterns = [
            {'name': 'MZ Header', 'description': 'Windows 可执行文件'},
            {'name': 'Suspicious API', 'description': '可疑 API 调用序列'},
        ]
        return patterns


def demo_forensics():
    """演示数字取证"""
    print("=" * 60)
    print("数字取证演示")
    print("=" * 60)
    
    # 创建一个临时测试文件
    test_file = "/tmp/test_evidence.txt"
    with open(test_file, 'w') as f:
        f.write("这是一个测试文件，用于演示数字取证功能。\n")
        f.write(f"创建时间: {datetime.now()}\n")
    
    # 收集证据
    print("\n--- 证据收集 ---")
    evidence = DigitalForensics.collect_file_evidence(test_file, "调查员-李四")
    
    print(f"证据 ID: {evidence.id}")
    print(f"证据类型: {evidence.type}")
    print(f"MD5: {evidence.hash_md5}")
    print(f"SHA256: {evidence.hash_sha256}")
    print(f"大小: {evidence.size} bytes")
    print(f"元数据: {json.dumps(evidence.metadata, indent=2)}")
    
    # 验证完整性
    print("\n--- 完整性验证 ---")
    is_valid = DigitalForensics.verify_integrity(evidence, test_file)
    print(f"完整性验证: {'通过' if is_valid else '失败'}")
    
    # 清理
    os.remove(test_file)


if __name__ == '__main__':
    demo_forensics()
```

---

## 3. 技术实践

### 3.1 自动化事件处理

```python
#!/usr/bin/env python3
"""
安全事件自动化处理系统
"""

import json
import re
from typing import Dict, List, Callable
from dataclasses import dataclass
from datetime import datetime

@dataclass
class Alert:
    """安全告警"""
    id: str
    source: str  # SIEM, IDS, Firewall, etc.
    alert_type: str
    severity: str
    description: str
    timestamp: datetime
    source_ip: str
    destination_ip: str
    raw_data: Dict


class AutomatedIncidentHandler:
    """自动化事件处理器"""
    
    def __init__(self):
        self.rules: List[Dict] = []
        self.actions: Dict[str, Callable] = {}
        self.blocked_ips: set = set()
    
    def register_action(self, name: str, action: Callable):
        """注册响应动作"""
        self.actions[name] = action
    
    def add_rule(self, name: str, conditions: Dict, actions: List[str]):
        """添加自动化规则"""
        rule = {
            'name': name,
            'conditions': conditions,
            'actions': actions,
            'enabled': True
        }
        self.rules.append(rule)
    
    def process_alert(self, alert: Alert) -> Dict:
        """处理单个告警"""
        results = {
            'alert_id': alert.id,
            'processed_at': datetime.now().isoformat(),
            'matched_rules': [],
            'executed_actions': [],
            'blocked': False
        }
        
        for rule in self.rules:
            if not rule['enabled']:
                continue
            
            if self._match_conditions(alert, rule['conditions']):
                results['matched_rules'].append(rule['name'])
                
                # 执行动作
                for action_name in rule['actions']:
                    if action_name in self.actions:
                        try:
                            self.actions[action_name](alert)
                            results['executed_actions'].append(action_name)
                        except Exception as e:
                            results['executed_actions'].append(f"{action_name}(失败: {e})")
                
                if 'block_ip' in rule['actions']:
                    results['blocked'] = True
        
        return results
    
    def _match_conditions(self, alert: Alert, conditions: Dict) -> bool:
        """检查告警是否匹配规则条件"""
        for key, value in conditions.items():
            if key == 'severity':
                if alert.severity != value:
                    return False
            elif key == 'alert_type':
                if alert.alert_type != value:
                    return False
            elif key == 'source_ip_in':
                if alert.source_ip not in value:
                    return False
            elif key == 'description_contains':
                if value.lower() not in alert.description.lower():
                    return False
        return True


# 预定义响应动作
def block_ip(alert: Alert):
    """阻断 IP"""
    print(f"  [动作] 阻断 IP: {alert.source_ip}")
    # 实际实现中调用防火墙 API

def send_email_notification(alert: Alert):
    """发送邮件通知"""
    print(f"  [动作] 发送邮件通知: {alert.id}")

def isolate_endpoint(alert: Alert):
    """隔离终端"""
    print(f"  [动作] 隔离终端: {alert.source_ip}")

def create_ticket(alert: Alert):
    """创建工单"""
    print(f"  [动作] 创建工单: {alert.id}")

def capture_traffic(alert: Alert):
    """捕获流量"""
    print(f"  [动作] 开始流量捕获: {alert.source_ip}")


def demo_automation():
    """演示自动化处理"""
    print("=" * 60)
    print("安全事件自动化处理演示")
    print("=" * 60)
    
    handler = AutomatedIncidentHandler()
    
    # 注册动作
    handler.register_action('block_ip', block_ip)
    handler.register_action('send_email', send_email_notification)
    handler.register_action('isolate_endpoint', isolate_endpoint)
    handler.register_action('create_ticket', create_ticket)
    handler.register_action('capture_traffic', capture_traffic)
    
    # 添加规则
    handler.add_rule(
        name="暴力破解自动阻断",
        conditions={
            'alert_type': 'brute_force',
            'severity': 'high'
        },
        actions=['block_ip', 'send_email', 'create_ticket']
    )
    
    handler.add_rule(
        name="恶意软件检测响应",
        conditions={
            'alert_type': 'malware_detected',
            'severity': 'critical'
        },
        actions=['isolate_endpoint', 'capture_traffic', 'send_email', 'create_ticket']
    )
    
    # 模拟告警
    alerts = [
        Alert(
            id="ALT-001",
            source="WAF",
            alert_type="brute_force",
            severity="high",
            description="检测到大量登录失败尝试",
            timestamp=datetime.now(),
            source_ip="192.168.1.100",
            destination_ip="10.0.0.10",
            raw_data={}
        ),
        Alert(
            id="ALT-002",
            source="EDR",
            alert_type="malware_detected",
            severity="critical",
            description="检测到勒索软件活动",
            timestamp=datetime.now(),
            source_ip="192.168.1.50",
            destination_ip="0.0.0.0",
            raw_data={}
        ),
        Alert(
            id="ALT-003",
            source="IDS",
            alert_type="suspicious_traffic",
            severity="medium",
            description="异常出站流量",
            timestamp=datetime.now(),
            source_ip="192.168.1.25",
            destination_ip="8.8.8.8",
            raw_data={}
        ),
    ]
    
    # 处理告警
    for alert in alerts:
        print(f"\n处理告警: {alert.id} ({alert.alert_type})")
        result = handler.process_alert(alert)
        print(f"  匹配规则: {result['matched_rules']}")
        print(f"  执行动作: {result['executed_actions']}")


if __name__ == '__main__':
    demo_automation()
```

---

## 4. 资源索引

### 4.1 标准与框架

| 标准 | 名称 | 说明 |
|------|------|------|
| NIST SP 800-61 | 计算机安全事件处理指南 | 美国标准 |
| ISO/IEC 27035 | 信息安全事件管理 | 国际标准 |
| SANS IR Framework | SANS 事件响应框架 | 行业框架 |
| MITRE ATT&CK | 攻击技术知识库 | 威胁分析 |

### 4.2 工具推荐

| 工具 | 用途 | 链接 |
|------|------|------|
| TheHive | 安全事件响应平台 | https://thehive-project.org/ |
| Cortex | 威胁情报分析 | https://github.com/TheHive-Project/Cortex |
| MISP | 威胁情报共享 | https://www.misp-project.org/ |
| Velociraptor | 端点监控取证 | https://docs.velociraptor.app/ |
| GRR | 远程实时取证 | https://grr.dev/ |

---

## 5. 关联知识

```
A04_Security_Quality/B03_Maintenance_Ops/
├── C01_Incident_Response (本模块)
│   ├── → C02_Performance_Tuning: 事件影响分析
│   └── → C03_Tech_Debt_Management: 安全债务处理
│
├── → B01_Information_Security: 安全事件分析
├── → B02_Reliability_Evaluation: 故障响应
└── → B04_Quality_Assurance: 安全测试
```

---

## 6. 学习建议

### 6.1 学习路径

1. **基础理论**: 学习 NIST 事件响应框架
2. **工具实践**: 掌握 SIEM、SOAR 平台使用
3. **取证技术**: 学习数字取证和内存分析
4. **实战演练**: 参与蓝队演练和 CTF

### 6.2 认证推荐

- **GCIH**: GIAC 认证事件处理师
- **GCFA**: GIAC 认证取证分析师
- **CISSP**: 覆盖事件响应领域
- **ECIH**: EC-Council 认证事件处理师

---

*最后更新: 2024-01-30*
