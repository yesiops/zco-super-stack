# C03 技术债务管理 (Tech Debt Management)

## 1. 主题定位

### 1.1 定义
技术债务（Technical Debt）是 Ward Cunningham 提出的概念，指为了快速交付功能而采取的非最优技术方案所导致的额外工作成本。类似于金融债务，技术债务如果不及时偿还，会随着时间的推移产生"利息"，使得后续开发和维护成本越来越高。

### 1.2 技术债务类型

```
技术债务分类：
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐     │
│  │   故意债务   │    │   无意债务   │    │   环境债务   │     │
│  │  (Deliberate)│    │(Inadvertent) │    │(Environmental)│    │
│  └──────┬───────┘    └──────┬───────┘    └──────┬───────┘     │
│         │                    │                    │             │
│  ┌──────▼───────┐    ┌──────▼───────┐    ┌──────▼───────┐     │
│  │ 已知-做了权衡│    │ 已知-不知道  │    │ 平台演进     │     │
│  │ 明知有更好   │    │ 不知道自己   │    │ 框架版本     │     │
│  │ 方案但选择   │    │ 不知道       │    │ 技术栈       │     │
│  │ 了快速方案   │    │              │    │ 过时         │     │
│  └──────────────┘    └──────────────┘    └──────────────┘     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.3 核心价值
- **风险控制**: 防止技术债务积累导致系统崩溃
- **成本控制**: 量化和管理债务偿还成本
- **团队效率**: 减少因债务导致的开发阻碍
- **可持续性**: 保持系统长期健康发展

---

## 2. 核心概念

### 2.1 技术债务识别

```python
#!/usr/bin/env python3
"""
技术债务检测与分析工具
"""

from dataclasses import dataclass, field
from typing import List, Dict, Optional, Set
from enum import Enum
from datetime import datetime
import re
import ast
import os

class DebtCategory(Enum):
    """技术债务类别"""
    CODE_QUALITY = "代码质量"
    ARCHITECTURE = "架构"
    TESTING = "测试"
    DOCUMENTATION = "文档"
    SECURITY = "安全"
    PERFORMANCE = "性能"
    DEPENDENCY = "依赖"
    INFRASTRUCTURE = "基础设施"

class DebtSeverity(Enum):
    """债务严重程度"""
    CRITICAL = "严重"      # 必须立即处理
    HIGH = "高"            # 应该在下一个迭代处理
    MEDIUM = "中"          # 应该在一个季度内处理
    LOW = "低"             # 可以延后处理
    TRIVIAL = "轻微"       # 有时间再处理

@dataclass
class TechDebtItem:
    """技术债务项"""
    id: str
    title: str
    description: str
    category: DebtCategory
    severity: DebtSeverity
    file_path: str
    line_number: int
    created_at: datetime
    estimated_effort_hours: int  # 估计修复工时
    interest_rate: float  # 利息率 (0-1)
    
    # 跟踪信息
    tags: List[str] = field(default_factory=list)
    related_issues: List[str] = field(default_factory=list)
    owner: Optional[str] = None
    status: str = "open"  # open, in_progress, resolved


class CodeDebtDetector:
    """代码债务检测器"""
    
    # 代码异味模式
    CODE_SMELLS = {
        'todo_fixme': re.compile(r'(TODO|FIXME|XXX|HACK):\s*(.+)', re.IGNORECASE),
        'long_method': 50,  # 行数阈值
        'large_class': 300,  # 行数阈值
        'many_parameters': 5,  # 参数数量阈值
        'nested_depth': 4,  # 嵌套深度阈值
        'duplicate_code': None,  # 由专门工具检测
    }
    
    def __init__(self, project_path: str):
        self.project_path = project_path
        self.debts: List[TechDebtItem] = []
    
    def scan_file(self, file_path: str) -> List[TechDebtItem]:
        """扫描单个文件"""
        debts = []
        
        try:
            with open(file_path, 'r', encoding='utf-8') as f:
                content = f.read()
                lines = content.split('\n')
        except:
            return debts
        
        # 检测 TODO/FIXME
        for i, line in enumerate(lines, 1):
            match = self.CODE_SMELLS['todo_fixme'].search(line)
            if match:
                debt_type = match.group(1).upper()
                description = match.group(2).strip()
                
                severity = {
                    'FIXME': DebtSeverity.HIGH,
                    'XXX': DebtSeverity.CRITICAL,
                    'HACK': DebtSeverity.HIGH,
                    'TODO': DebtSeverity.MEDIUM
                }.get(debt_type, DebtSeverity.LOW)
                
                debts.append(TechDebtItem(
                    id=f"TD-{len(self.debts)+1:04d}",
                    title=f"{debt_type}: {description[:50]}",
                    description=description,
                    category=DebtCategory.CODE_QUALITY,
                    severity=severity,
                    file_path=file_path,
                    line_number=i,
                    created_at=datetime.now(),
                    estimated_effort_hours=2,
                    interest_rate=0.1
                ))
        
        # 检测长方法
        self._detect_long_methods(file_path, content, debts)
        
        # 检测大文件
        if len(lines) > self.CODE_SMELLS['large_class']:
            debts.append(TechDebtItem(
                id=f"TD-{len(self.debts)+1:04d}",
                title="文件过大",
                description=f"文件 {len(lines)} 行，建议拆分成更小的模块",
                category=DebtCategory.ARCHITECTURE,
                severity=DebtSeverity.MEDIUM,
                file_path=file_path,
                line_number=1,
                created_at=datetime.now(),
                estimated_effort_hours=8,
                interest_rate=0.05
            ))
        
        return debts
    
    def _detect_long_methods(self, file_path: str, content: str, debts: List):
        """检测长方法"""
        try:
            tree = ast.parse(content)
            for node in ast.walk(tree):
                if isinstance(node, (ast.FunctionDef, ast.AsyncFunctionDef)):
                    method_lines = node.end_lineno - node.lineno
                    if method_lines > self.CODE_SMELLS['long_method']:
                        debts.append(TechDebtItem(
                            id=f"TD-{len(self.debts)+1:04d}",
                            title=f"方法过长: {node.name}",
                            description=f"方法 {method_lines} 行，建议拆分成更小的方法",
                            category=DebtCategory.CODE_QUALITY,
                            severity=DebtSeverity.MEDIUM,
                            file_path=file_path,
                            line_number=node.lineno,
                            created_at=datetime.now(),
                            estimated_effort_hours=4,
                            interest_rate=0.05
                        ))
                    
                    # 检测过多参数
                    if len(node.args.args) > self.CODE_SMELLS['many_parameters']:
                        debts.append(TechDebtItem(
                            id=f"TD-{len(self.debts)+1:04d}",
                            title=f"参数过多: {node.name}",
                            description=f"方法有 {len(node.args.args)} 个参数，建议使用对象封装",
                            category=DebtCategory.CODE_QUALITY,
                            severity=DebtSeverity.LOW,
                            file_path=file_path,
                            line_number=node.lineno,
                            created_at=datetime.now(),
                            estimated_effort_hours=2,
                            interest_rate=0.03
                        ))
        except SyntaxError:
            pass
    
    def scan_project(self) -> List[TechDebtItem]:
        """扫描整个项目"""
        for root, dirs, files in os.walk(self.project_path):
            # 跳过常见目录
            dirs[:] = [d for d in dirs if d not in ['node_modules', '.git', '__pycache__', 'venv']]
            
            for file in files:
                if file.endswith('.py'):
                    file_path = os.path.join(root, file)
                    debts = self.scan_file(file_path)
                    self.debts.extend(debts)
        
        return self.debts


class DependencyDebtAnalyzer:
    """依赖债务分析器"""
    
    def __init__(self):
        self.debts: List[TechDebtItem] = []
    
    def analyze_python_dependencies(self, requirements_file: str) -> List[TechDebtItem]:
        """分析 Python 依赖"""
        debts = []
        
        try:
            with open(requirements_file, 'r') as f:
                requirements = f.readlines()
        except FileNotFoundError:
            return debts
        
        for line in requirements:
            line = line.strip()
            if not line or line.startswith('#'):
                continue
            
            # 检查无版本限制的依赖
            if '==' not in line and '>=' not in line and '<=' not in line:
                package_name = line.split('[')[0].strip()
                debts.append(TechDebtItem(
                    id=f"TD-DEP-{len(debts)+1:04d}",
                    title=f"依赖无版本限制: {package_name}",
                    description=f"依赖 {package_name} 没有指定版本，可能导致构建不稳定",
                    category=DebtCategory.DEPENDENCY,
                    severity=DebtSeverity.MEDIUM,
                    file_path=requirements_file,
                    line_number=0,
                    created_at=datetime.now(),
                    estimated_effort_hours=1,
                    interest_rate=0.05
                ))
        
        return debts


class ArchitectureDebtDetector:
    """架构债务检测器"""
    
    COMMON_ARCHITECTURE_SMELLS = [
        {
            'name': '循环依赖',
            'pattern': r'from\s+(\w+)\s+import.*\n.*from\s+(\w+)\s+import',
            'severity': DebtSeverity.HIGH
        },
        {
            'name': '上帝类',
            'indicators': ['很多方法', '很多属性', '做很多事情'],
            'severity': DebtSeverity.HIGH
        },
        {
            'name': '贫血模型',
            'indicators': ['只有 getter/setter', '业务逻辑在服务层'],
            'severity': DebtSeverity.MEDIUM
        }
    ]


def demo_debt_detection():
    """演示债务检测"""
    print("=" * 60)
    print("技术债务检测演示")
    print("=" * 60)
    
    # 创建一个示例 Python 文件
    sample_code = '''
# FIXME: 这个函数需要重构，性能太差
def process_data(data):
    # TODO: 添加异常处理
    result = []
    for item in data:
        if item.active:
            if item.value > 10:
                if item.category == 'A':
                    if item.status == 'pending':
                        result.append(item)
    return result

# XXX: 临时解决方案，需要重新设计
def calculate(a, b, c, d, e, f, g, h):
    return a + b + c + d + e + f + g + h

class GodClass:
    def method1(self): pass
    def method2(self): pass
    def method3(self): pass
    # ... 50 more methods
'''
    
    import tempfile
    with tempfile.NamedTemporaryFile(mode='w', suffix='.py', delete=False) as f:
        f.write(sample_code)
        temp_file = f.name
    
    try:
        # 检测债务
        detector = CodeDebtDetector(temp_file)
        debts = detector.scan_file(temp_file)
        
        print(f"\n发现 {len(debts)} 项技术债务:\n")
        
        for debt in debts:
            print(f"[{debt.severity.value}] {debt.title}")
            print(f"  文件: {debt.file_path}:{debt.line_number}")
            print(f"  描述: {debt.description}")
            print(f"  预计修复: {debt.estimated_effort_hours} 小时")
            print()
    finally:
        os.unlink(temp_file)


if __name__ == '__main__':
    demo_debt_detection()
```

### 2.2 技术债务量化

```python
#!/usr/bin/env python3
"""
技术债务量化模型
"""

from dataclasses import dataclass
from typing import List, Dict
from datetime import datetime, timedelta
import math

@dataclass
class DebtMetrics:
    """债务指标"""
    total_debt_items: int
    total_estimated_hours: int
    principal_cost: float  # 本金成本
    interest_cost_per_month: float  # 月利息成本
    technical_debt_ratio: float  # 技术债务比率
    debt_density: float  # 债务密度 (每千行代码)


class TechnicalDebtQuantification:
    """技术债务量化模型"""
    
    # 平均开发人员成本 (元/小时)
    DEVELOPER_COST_PER_HOUR = 200
    
    # 利息系数 (债务随时间增加的成本)
    INTEREST_MULTIPLIERS = {
        'Code': 1.2,      # 代码质量债务每年增长 20%
        'Architecture': 1.5,  # 架构债务每年增长 50%
        'Testing': 1.1,   # 测试债务每年增长 10%
        'Security': 2.0,  # 安全债务每年增长 100%
    }
    
    def __init__(self, debts: List):
        self.debts = debts
    
    def calculate_principal(self) -> float:
        """计算本金 (立即偿还所有债务的成本)"""
        total_hours = sum(d.estimated_effort_hours for d in self.debts)
        return total_hours * self.DEVELOPER_COST_PER_HOUR
    
    def calculate_interest(self, months: int = 12) -> float:
        """计算利息 (延迟偿还的额外成本)"""
        total_interest = 0
        
        for debt in self.debts:
            # 根据债务类别和年龄计算利息
            category = debt.category.value if hasattr(debt, 'category') else 'Code'
            multiplier = self.INTEREST_MULTIPLIERS.get(category, 1.2)
            
            # 计算债务年龄
            if hasattr(debt, 'created_at'):
                age_years = (datetime.now() - debt.created_at).days / 365
            else:
                age_years = 0
            
            # 复利计算
            debt_cost = debt.estimated_effort_hours * self.DEVELOPER_COST_PER_HOUR
            future_cost = debt_cost * (multiplier ** age_years)
            interest = future_cost - debt_cost
            
            total_interest += interest
        
        return total_interest
    
    def calculate_td_ratio(self, total_code_lines: int) -> float:
        """计算技术债务比率 (TDR)
        
        TDR = (修复债务成本 / 重写整个系统成本) × 100%
        
        一般认为:
        - TDR < 5%: 健康
        - 5% <= TDR < 10%: 需要关注
        - 10% <= TDR < 20%: 需要改进
        - TDR >= 20%: 危险
        """
        if total_code_lines == 0:
            return 0
        
        principal = self.calculate_principal()
        
        # 假设重写成本 = 代码行数 × 0.5 小时 × 时薪
        rewrite_cost = total_code_lines * 0.5 * self.DEVELOPER_COST_PER_HOUR
        
        return (principal / rewrite_cost) * 100 if rewrite_cost > 0 else 0
    
    def calculate_sqale_index(self) -> int:
        """计算 SQALE 指数 (Software Quality Assessment based on Lifecycle Expectations)
        
        将债务映射到不同质量特性，计算总体修复成本
        """
        # 质量特性权重
        weights = {
            'Code': 1.0,
            'Architecture': 2.0,
            'Testing': 1.5,
            'Security': 3.0,
            'Documentation': 0.5
        }
        
        sqale_index = 0
        for debt in self.debts:
            category = debt.category.value if hasattr(debt, 'category') else 'Code'
            weight = weights.get(category, 1.0)
            sqale_index += debt.estimated_effort_hours * weight
        
        return int(sqale_index)
    
    def get_debt_rating(self, total_code_lines: int) -> str:
        """获取债务评级"""
        tdr = self.calculate_td_ratio(total_code_lines)
        
        if tdr < 5:
            return 'A'  # 优秀
        elif tdr < 10:
            return 'B'  # 良好
        elif tdr < 15:
            return 'C'  # 一般
        elif tdr < 20:
            return 'D'  # 较差
        else:
            return 'E'  # 危险
    
    def generate_report(self, total_code_lines: int) -> Dict:
        """生成债务报告"""
        principal = self.calculate_principal()
        annual_interest = self.calculate_interest(12)
        tdr = self.calculate_td_ratio(total_code_lines)
        sqale = self.calculate_sqale_index()
        rating = self.get_debt_rating(total_code_lines)
        
        # 按类别分组
        by_category = {}
        for debt in self.debts:
            cat = debt.category.value if hasattr(debt, 'category') else 'Unknown'
            if cat not in by_category:
                by_category[cat] = {'count': 0, 'hours': 0}
            by_category[cat]['count'] += 1
            by_category[cat]['hours'] += debt.estimated_effort_hours
        
        return {
            'summary': {
                'total_debt_items': len(self.debts),
                'total_estimated_hours': sum(d.estimated_effort_hours for d in self.debts),
                'principal_cost_cny': principal,
                'annual_interest_cny': annual_interest,
                'tech_debt_ratio': round(tdr, 2),
                'sqale_index': sqale,
                'debt_rating': rating,
                'total_code_lines': total_code_lines
            },
            'by_category': by_category,
            'recommendations': self._generate_recommendations(tdr)
        }
    
    def _generate_recommendations(self, tdr: float) -> List[str]:
        """生成改进建议"""
        recommendations = []
        
        if tdr >= 20:
            recommendations.append("技术债务比率极高，建议暂停新功能开发，全力偿还债务")
            recommendations.append("考虑系统重构或重写")
        elif tdr >= 10:
            recommendations.append("技术债务较高，建议每个迭代分配 30% 时间处理债务")
        elif tdr >= 5:
            recommendations.append("技术债务可控，建议每个迭代分配 20% 时间处理债务")
        else:
            recommendations.append("技术债务状况良好，继续保持")
        
        return recommendations


def demo_quantification():
    """演示债务量化"""
    from tech_debt_management import TechDebtItem, DebtCategory, DebtSeverity
    
    print("=" * 60)
    print("技术债务量化分析")
    print("=" * 60)
    
    # 创建示例债务
    debts = [
        TechDebtItem("TD-001", "代码重复", "描述", DebtCategory.CODE_QUALITY, 
                    DebtSeverity.MEDIUM, "file.py", 10, datetime.now(), 8, 0.1),
        TechDebtItem("TD-002", "缺少测试", "描述", DebtCategory.TESTING, 
                    DebtSeverity.HIGH, "file.py", 20, datetime.now(), 16, 0.15),
        TechDebtItem("TD-003", "架构耦合", "描述", DebtCategory.ARCHITECTURE, 
                    DebtSeverity.CRITICAL, "file.py", 50, datetime.now(), 40, 0.2),
    ]
    
    # 量化分析
    quant = TechnicalDebtQuantification(debts)
    total_lines = 10000
    
    report = quant.generate_report(total_lines)
    
    print(f"\n债务概况:")
    print(f"  债务项数: {report['summary']['total_debt_items']}")
    print(f"  预计修复工时: {report['summary']['total_estimated_hours']} 小时")
    print(f"  本金成本: ¥{report['summary']['principal_cost_cny']:,.2f}")
    print(f"  年利息成本: ¥{report['summary']['annual_interest_cny']:,.2f}")
    print(f"  技术债务比率: {report['summary']['tech_debt_ratio']}%")
    print(f"  SQALE 指数: {report['summary']['sqale_index']}")
    print(f"  债务评级: {report['summary']['debt_rating']}")
    
    print(f"\n按类别分布:")
    for cat, data in report['by_category'].items():
        print(f"  {cat}: {data['count']} 项, {data['hours']} 小时")
    
    print(f"\n改进建议:")
    for rec in report['recommendations']:
        print(f"  - {rec}")


if __name__ == '__main__':
    demo_quantification()
```

---

## 3. 技术实践

### 3.1 债务管理流程

```python
#!/usr/bin/env python3
"""
技术债务管理工作流
"""

from typing import List, Dict, Optional
from datetime import datetime, timedelta
from dataclasses import dataclass
import json

@dataclass
class DebtSprint:
    """债务冲刺"""
    id: str
    name: str
    start_date: datetime
    end_date: datetime
    debt_items: List
    allocated_hours: int
    completed_hours: int = 0


class TechnicalDebtBacklog:
    """技术债务待办列表"""
    
    def __init__(self):
        self.debts: List = []
        self.sprints: List[DebtSprint] = []
        self.capacity_per_sprint: int = 40  # 每个迭代可用于债务的时间
    
    def add_debt(self, debt):
        """添加债务"""
        self.debts.append(debt)
    
    def prioritize(self) -> List:
        """债务优先级排序"""
        # 排序算法：严重程度 × 利息率 / 修复成本
        def priority_score(debt):
            severity_weight = {
                'Critical': 10, 'High': 7, 'Medium': 5, 'Low': 2, 'Trivial': 1
            }
            sev = severity_weight.get(debt.severity.value if hasattr(debt, 'severity') else 'Medium', 5)
            interest = debt.interest_rate if hasattr(debt, 'interest_rate') else 0.1
            effort = debt.estimated_effort_hours if hasattr(debt, 'estimated_effort_hours') else 1
            
            return (sev * interest) / max(effort, 1)
        
        return sorted(self.debts, key=priority_score, reverse=True)
    
    def plan_sprint(self, sprint_id: str, sprint_name: str, 
                   start_date: datetime, duration_weeks: int = 2) -> DebtSprint:
        """计划债务冲刺"""
        end_date = start_date + timedelta(weeks=duration_weeks)
        
        # 计算可用容量 (通常 20% 的迭代时间用于债务)
        allocated_hours = self.capacity_per_sprint
        
        # 选择债务
        prioritized = self.prioritize()
        selected = []
        total_hours = 0
        
        for debt in prioritized:
            if debt.status != 'open':
                continue
            
            effort = debt.estimated_effort_hours if hasattr(debt, 'estimated_effort_hours') else 0
            if total_hours + effort <= allocated_hours:
                selected.append(debt)
                total_hours += effort
        
        sprint = DebtSprint(
            id=sprint_id,
            name=sprint_name,
            start_date=start_date,
            end_date=end_date,
            debt_items=selected,
            allocated_hours=allocated_hours
        )
        
        self.sprints.append(sprint)
        return sprint
    
    def generate_payoff_plan(self, months: int = 12) -> Dict:
        """生成还债计划"""
        plan = {
            'target_months': months,
            'sprints': [],
            'projected_debt_reduction': []
        }
        
        open_debts = [d for d in self.debts if d.status == 'open']
        total_hours = sum(d.estimated_effort_hours for d in open_debts)
        
        # 每月可用工时
        monthly_hours = self.capacity_per_sprint * 2  # 假设每月2个迭代
        
        remaining_hours = total_hours
        for month in range(1, months + 1):
            remaining_hours = max(0, remaining_hours - monthly_hours)
            plan['projected_debt_reduction'].append({
                'month': month,
                'remaining_hours': remaining_hours,
                'reduction_percentage': (1 - remaining_hours / total_hours) * 100 if total_hours > 0 else 100
            })
            
            if remaining_hours == 0:
                break
        
        return plan


# 与项目管理工具集成
class JiraIntegration:
    """Jira 集成示例"""
    
    @staticmethod
    def create_debt_ticket(debt, project_key: str = "TECH") -> str:
        """创建 Jira 工单"""
        # 实际实现需要调用 Jira API
        ticket = {
            'project': {'key': project_key},
            'summary': f"[Tech Debt] {debt.title}",
            'description': debt.description,
            'issuetype': {'name': 'Technical Debt'},
            'customfield_debt_category': debt.category.value if hasattr(debt, 'category') else 'Other',
            'customfield_estimated_hours': debt.estimated_effort_hours if hasattr(debt, 'estimated_effort_hours') else 0,
        }
        
        # 模拟返回 ticket key
        return f"{project_key}-{hash(debt.title) % 10000}"


# 报告生成
class DebtReportGenerator:
    """债务报告生成器"""
    
    @staticmethod
    def generate_markdown_report(backlog: TechnicalDebtBacklog) -> str:
        """生成 Markdown 格式报告"""
        lines = []
        lines.append("# 技术债务报告")
        lines.append(f"\n生成时间: {datetime.now().strftime('%Y-%m-%d %H:%M')}")
        lines.append(f"\n## 概况")
        lines.append(f"\n- 总债务项: {len(backlog.debts)}")
        lines.append(f"- 开放债务: {len([d for d in backlog.debts if d.status == 'open'])}")
        lines.append(f"- 已完成: {len([d for d in backlog.debts if d.status == 'resolved'])}")
        
        lines.append(f"\n## 按严重程度分布")
        # ... 分组统计
        
        lines.append(f"\n## 近期冲刺")
        for sprint in backlog.sprints[-3:]:  # 最近3个
            lines.append(f"\n### {sprint.name}")
            lines.append(f"- 时间: {sprint.start_date.date()} - {sprint.end_date.date()}")
            lines.append(f"- 计划工时: {sprint.allocated_hours}")
            lines.append(f"- 债务项数: {len(sprint.debt_items)}")
        
        return '\n'.join(lines)


def demo_workflow():
    """演示债务管理工作流"""
    print("=" * 60)
    print("技术债务管理工作流演示")
    print("=" * 60)
    
    from tech_debt_management import TechDebtItem, DebtCategory, DebtSeverity
    
    backlog = TechnicalDebtBacklog()
    
    # 添加债务
    debts = [
        TechDebtItem("TD-001", "重构用户模块", "代码耦合严重", 
                    DebtCategory.ARCHITECTURE, DebtSeverity.HIGH, 
                    "users.py", 10, datetime.now(), 40, 0.2),
        TechDebtItem("TD-002", "补充单元测试", "覆盖率低于 60%",
                    DebtCategory.TESTING, DebtSeverity.MEDIUM,
                    "tests/", 1, datetime.now(), 20, 0.1),
        TechDebtItem("TD-003", "升级依赖版本", "存在安全漏洞",
                    DebtCategory.DEPENDENCY, DebtSeverity.CRITICAL,
                    "requirements.txt", 1, datetime.now(), 8, 0.3),
    ]
    
    for debt in debts:
        backlog.add_debt(debt)
    
    # 优先级排序
    print("\n--- 债务优先级排序 ---")
    prioritized = backlog.prioritize()
    for i, debt in enumerate(prioritized, 1):
        print(f"{i}. [{debt.severity.value}] {debt.title} ({debt.estimated_effort_hours}h)")
    
    # 计划冲刺
    print("\n--- 计划债务冲刺 ---")
    sprint = backlog.plan_sprint(
        sprint_id="Sprint-25",
        sprint_name="2024-Q1-Debt-1",
        start_date=datetime.now()
    )
    
    print(f"冲刺: {sprint.name}")
    print(f"分配工时: {sprint.allocated_hours} 小时")
    print(f"选中债务项:")
    for debt in sprint.debt_items:
        print(f"  - {debt.title} ({debt.estimated_effort_hours}h)")
    
    # 还债计划
    print("\n--- 还债计划 ---")
    plan = backlog.generate_payoff_plan(months=6)
    for month_data in plan['projected_debt_reduction']:
        print(f"第 {month_data['month']} 个月: 剩余 {month_data['remaining_hours']} 小时 "
              f"({month_data['reduction_percentage']:.1f}% 已偿还)")


if __name__ == '__main__':
    demo_workflow()
```

---

## 4. 资源索引

### 4.1 工具推荐

| 工具 | 用途 | 链接 |
|------|------|------|
| SonarQube | 代码质量与债务分析 | https://www.sonarqube.org/ |
| CodeClimate | 技术债务追踪 | https://codeclimate.com/ |
| Snyk | 依赖债务与安全 | https://snyk.io/ |
| TechDebt.org | 技术债务资源 | https://www.techdebt.org/ |

### 4.2 学习资源

- **《重构》**: Martin Fowler 著，改善既有代码的设计
- **《技术债务管理》**: 技术债务系统性管理方法

---

## 5. 关联知识

```
A04_Security_Quality/B03_Maintenance_Ops/
├── C03_Tech_Debt_Management (本模块)
│   ├── → C01_Incident_Response: 债务导致的事件
│   └── → C02_Performance_Tuning: 性能债务
│
├── → B04_Quality_Assurance: 代码质量
└── → A05_Architecture_Patterns: 架构演进
```

---

*最后更新: 2024-01-30*
