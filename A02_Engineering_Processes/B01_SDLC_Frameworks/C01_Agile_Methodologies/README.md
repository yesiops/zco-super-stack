# C01_Agile_Methodologies - 敏捷方法论

## 1. 主题定位

### 1.1 定义与范围

敏捷方法论是一套以人为核心、迭代增量式的软件开发哲学和实践框架。本知识单元聚焦于敏捷宣言（Agile Manifesto）的核心价值观和十二项原则，以及Scrum、Kanban、XP等主流敏捷框架的实施细节。涵盖从需求管理、迭代规划到持续改进的完整软件开发生命周期管理方法论。

### 1.2 业务价值

敏捷方法论通过短周期交付、快速反馈和持续适应变化，帮助组织实现：

- **业务响应速度提升**：缩短从需求到上线的周期，快速响应市场变化
- **交付价值最大化**：优先交付高价值功能，确保投资回报率
- **风险早期暴露**：通过频繁交付和反馈，尽早发现和解决问题
- **团队赋能与自组织**：激发团队创造力，提升成员参与度和满意度
- **客户满意度提升**：持续交付可用软件，建立信任关系

### 1.3 适用场景

| 场景类型 | 适用性 | 说明 |
|---------|-------|------|
| 互联网产品 | ⭐⭐⭐⭐⭐ | 需求变化快，需要快速迭代验证 |
| 企业数字化转型 | ⭐⭐⭐⭐ | 复杂业务场景，需要分阶段交付 |
| 创业团队 | ⭐⭐⭐⭐⭐ | 资源有限，需要最大化交付价值 |
| 大型遗留系统改造 | ⭐⭐⭐ | 需要渐进式迁移策略 |
| 强监管行业 | ⭐⭐ | 需要与合规要求平衡适配 |

## 2. 核心概念

### 2.1 敏捷宣言与价值观

```
我们正在通过亲身实践和帮助他人实践，揭示更好的软件开发方法。
通过这项工作，我们认为：

个体和互动 高于 流程和工具
工作的软件 高于 详尽的文档
客户合作 高于 合同谈判
响应变化 高于 遵循计划

也就是说，尽管右项有其价值，我们更重视左项的价值。
```

### 2.2 Scrum 框架

#### 2.2.1 核心角色

```
┌─────────────────────────────────────────────────────────────┐
│                      Scrum 团队结构                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌──────────────┐    ┌──────────────┐    ┌──────────────┐ │
│   │ Product      │    │ Scrum        │    │ Development  │ │
│   │ Owner        │◄──►│ Master       │◄──►│ Team         │ │
│   │              │    │              │    │              │ │
│   │ • 定义愿景    │    │ • 流程守护    │    │ • 跨职能      │ │
│   │ • 管理待办    │    │ • 障碍移除    │    │ • 自组织      │ │
│   │ • 优先级排序  │    │ • 引导会议    │    │ • 迭代交付    │ │
│   │ • 验收成果    │    │ • 教练团队    │    │ • 质量内建    │ │
│   └──────────────┘    └──────────────┘    └──────────────┘ │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### 2.2.2 工件与事件

```
Scrum 事件时间盒：

Sprint (1-4周)
├── Sprint Planning     (8小时/月)
├── Daily Scrum         (15分钟/天)
├── Sprint Review       (4小时/月)
├── Sprint Retrospective (3小时/月)
└── Backlog Refinement  (10% Sprint时间)
```

### 2.3 Kanban 方法

```
Kanban 核心原则：

1. 可视化工作流程
   ┌─────────────────────────────────────────┐
   │  待办  │  分析  │  开发  │  测试  │  完成 │
   │   5    │   2    │   3    │   2    │   ∞   │  ← WIP限制
   │        │   ▓▓   │  ▓▓▓   │   ▓▓   │  ★★★  │
   └─────────────────────────────────────────┘

2. 限制在制品 (WIP)
3. 管理流动
4. 明确流程策略
5. 建立反馈循环
6. 协作改进
```

### 2.4 极限编程 (XP)

```
XP 核心实践环：

              ┌─────────────────┐
              │   计划游戏      │
              └────────┬────────┘
                       │
    ┌──────────────────┼──────────────────┐
    │                  │                  │
    ▼                  ▼                  ▼
┌─────────┐     ┌─────────┐       ┌─────────┐
│简单设计 │◄───►│测试驱动 │◄─────►│持续集成 │
└────┬────┘     │开发(TDD)│       └────┬────┘
     │          └────┬────┘            │
     │               │                 │
     ▼               ▼                 ▼
┌─────────┐     ┌─────────┐       ┌─────────┐
│重构     │     │结对编程 │       │代码集体 │
│         │     │         │       │所有制   │
└─────────┘     └─────────┘       └─────────┘
```

### 2.5 规模化敏捷框架

| 框架 | 适用规模 | 核心特点 |
|-----|---------|---------|
| SAFe | 大型组织 | 精益-敏捷原则，价值流对齐 |
| LeSS | 中大型企业 | 简化Scrum扩展，减少复杂性 |
| Nexus | 多团队Scrum | 集成团队协调依赖 |
| Spotify模型 | 中型组织 | 部落-小队-分会-公会结构 |

## 3. 技术实践

### 3.1 用户故事实践

#### 3.1.1 用户故事卡片格式

```markdown
## 用户故事模板

### 故事标题
作为 [角色]，我想要 [功能]，以便于 [价值]

### 验收标准 (AC)
- [ ] 给定 [上下文]，当 [事件]，那么 [结果1]
- [ ] 给定 [上下文]，当 [事件]，那么 [结果2]
- [ ] 边界情况处理

### 技术任务拆分
- [ ] 数据库设计
- [ ] API接口开发
- [ ] 前端页面实现
- [ ] 单元测试编写
- [ ] 集成测试

### 估算
- 故事点：5
- 预估工时：16小时

### 优先级
- MoSCoW：Must Have
- WSJF分数：23.5
```

#### 3.1.2 Python 用户故事管理工具

```python
#!/usr/bin/env python3
"""
用户故事管理系统
支持故事点估算、优先级排序和Sprint规划
"""

from dataclasses import dataclass, field
from enum import Enum
from typing import List, Optional
from datetime import datetime, timedelta
import json


class Priority(Enum):
    MUST_HAVE = 1
    SHOULD_HAVE = 2
    COULD_HAVE = 3
    WONT_HAVE = 4


class Status(Enum):
    BACKLOG = "待办"
    SELECTED = "已选"
    IN_PROGRESS = "进行中"
    DONE = "已完成"


@dataclass
class UserStory:
    """用户故事数据模型"""
    id: str
    title: str
    role: str
    feature: str
    value: str
    story_points: int
    priority: Priority
    status: Status = Status.BACKLOG
    acceptance_criteria: List[str] = field(default_factory=list)
    tasks: List[dict] = field(default_factory=list)
    created_at: datetime = field(default_factory=datetime.now)
    completed_at: Optional[datetime] = None
    
    def to_dict(self) -> dict:
        """转换为字典格式"""
        return {
            "id": self.id,
            "title": self.title,
            "role": self.role,
            "feature": self.feature,
            "value": self.value,
            "story_points": self.story_points,
            "priority": self.priority.name,
            "status": self.status.value,
            "acceptance_criteria": self.acceptance_criteria,
            "tasks": self.tasks,
            "created_at": self.created_at.isoformat(),
            "completed_at": self.completed_at.isoformat() if self.completed_at else None
        }
    
    def calculate_wsjf(self, user_business_value: float, 
                       time_criticality: float,
                       risk_reduction: float,
                       job_size: float) -> float:
        """
        计算WSJF (Weighted Shortest Job First) 分数
        用于优先级排序
        """
        if job_size == 0:
            return 0
        cost_of_delay = user_business_value + time_criticality + risk_reduction
        return round(cost_of_delay / job_size, 2)
    
    def __str__(self) -> str:
        return f"[{self.id}] {self.title} ({self.story_points}pts)"


class Sprint:
    """Sprint管理类"""
    
    def __init__(self, name: str, start_date: datetime, duration_weeks: int = 2):
        self.name = name
        self.start_date = start_date
        self.end_date = start_date + timedelta(weeks=duration_weeks)
        self.stories: List[UserStory] = []
        self.capacity = 0  # 团队容量（故事点）
        
    def add_story(self, story: UserStory) -> bool:
        """添加故事到Sprint，检查容量"""
        current_load = sum(s.story_points for s in self.stories)
        if current_load + story.story_points <= self.capacity:
            self.stories.append(story)
            story.status = Status.SELECTED
            return True
        return False
    
    def get_burndown_data(self) -> dict:
        """生成燃尽图数据"""
        total_points = sum(s.story_points for s in self.stories)
        done_points = sum(s.story_points for s in self.stories 
                         if s.status == Status.DONE)
        remaining = total_points - done_points
        
        return {
            "sprint_name": self.name,
            "total_points": total_points,
            "completed_points": done_points,
            "remaining_points": remaining,
            "completion_percentage": round(done_points/total_points*100, 2) if total_points else 0
        }
    
    def export_to_json(self, filepath: str):
        """导出Sprint数据到JSON"""
        data = {
            "sprint": self.name,
            "start_date": self.start_date.isoformat(),
            "end_date": self.end_date.isoformat(),
            "capacity": self.capacity,
            "stories": [s.to_dict() for s in self.stories],
            "burndown": self.get_burndown_data()
        }
        with open(filepath, 'w', encoding='utf-8') as f:
            json.dump(data, f, ensure_ascii=False, indent=2)


class ProductBacklog:
    """产品待办列表管理"""
    
    def __init__(self):
        self.stories: List[UserStory] = []
        
    def add_story(self, story: UserStory):
        self.stories.append(story)
        
    def prioritize_by_wsjf(self) -> List[UserStory]:
        """按WSJF分数排序"""
        return sorted(self.stories, 
                     key=lambda s: s.priority.value,
                     reverse=True)
    
    def filter_by_status(self, status: Status) -> List[UserStory]:
        """按状态过滤"""
        return [s for s in self.stories if s.status == status]
    
    def generate_velocity_report(self, sprints: List[Sprint]) -> dict:
        """生成速度报告"""
        velocities = []
        for sprint in sprints:
            completed = sum(s.story_points for s in sprint.stories 
                          if s.status == Status.DONE)
            velocities.append(completed)
        
        avg_velocity = sum(velocities) / len(velocities) if velocities else 0
        
        return {
            "sprint_velocities": velocities,
            "average_velocity": round(avg_velocity, 2),
            "predictability": self._calculate_predictability(velocities)
        }
    
    def _calculate_predictability(self, velocities: List[int]) -> float:
        """计算可预测性（变异系数倒数）"""
        if len(velocities) < 2:
            return 0
        mean = sum(velocities) / len(velocities)
        variance = sum((v - mean) ** 2 for v in velocities) / len(velocities)
        std_dev = variance ** 0.5
        cv = std_dev / mean if mean else 0
        return round((1 - min(cv, 1)) * 100, 2)


# 使用示例
if __name__ == "__main__":
    # 创建产品待办列表
    backlog = ProductBacklog()
    
    # 添加用户故事
    story1 = UserStory(
        id="US-001",
        title="用户登录功能",
        role="注册用户",
        feature="通过邮箱和密码登录系统",
        value="安全访问个人账户",
        story_points=5,
        priority=Priority.MUST_HAVE,
        acceptance_criteria=[
            "输入正确凭据时成功登录",
            "密码错误时显示友好提示",
            "支持"记住我"功能"
        ]
    )
    
    story2 = UserStory(
        id="US-002",
        title="密码重置功能",
        role="忘记密码的用户",
        feature="通过邮箱重置密码",
        value="恢复账户访问权限",
        story_points=3,
        priority=Priority.SHOULD_HAVE
    )
    
    backlog.add_story(story1)
    backlog.add_story(story2)
    
    # 创建Sprint
    sprint = Sprint(
        name="Sprint 1",
        start_date=datetime.now(),
        duration_weeks=2
    )
    sprint.capacity = 20  # 团队容量20故事点
    
    # 规划Sprint
    for story in backlog.prioritize_by_wsjf():
        if sprint.add_story(story):
            print(f"✓ 已添加 {story} 到 {sprint.name}")
        else:
            print(f"✗ {story} 超出容量，移至下一Sprint")
    
    # 导出报告
    sprint.export_to_json("sprint_plan.json")
    print("\nSprint规划已导出到 sprint_plan.json")
```

### 3.2 迭代规划与追踪

#### 3.2.1 Sprint规划脚本

```python
#!/usr/bin/env python3
"""
Sprint规划自动化工具
集成JIRA API进行迭代管理
"""

import requests
from datetime import datetime, timedelta
from typing import Dict, List, Optional
import os


class JiraSprintManager:
    """JIRA Sprint管理器"""
    
    def __init__(self, base_url: str, username: str, api_token: str):
        self.base_url = base_url.rstrip('/')
        self.auth = (username, api_token)
        self.headers = {
            "Accept": "application/json",
            "Content-Type": "application/json"
        }
    
    def get_active_sprint(self, board_id: int) -> Optional[Dict]:
        """获取活跃Sprint"""
        url = f"{self.base_url}/rest/agile/1.0/board/{board_id}/sprint"
        params = {"state": "active"}
        
        response = requests.get(url, auth=self.auth, 
                              headers=self.headers, params=params)
        
        if response.status_code == 200:
            sprints = response.json().get("values", [])
            return sprints[0] if sprints else None
        return None
    
    def create_sprint(self, board_id: int, name: str, 
                     start_date: datetime, end_date: datetime) -> Dict:
        """创建新Sprint"""
        url = f"{self.base_url}/rest/agile/1.0/sprint"
        
        payload = {
            "name": name,
            "originBoardId": board_id,
            "startDate": start_date.strftime("%Y-%m-%dT%H:%M:%S.000+0800"),
            "endDate": end_date.strftime("%Y-%m-%dT%H:%M:%S.000+0800")
        }
        
        response = requests.post(url, auth=self.auth,
                               headers=self.headers, json=payload)
        response.raise_for_status()
        return response.json()
    
    def get_sprint_issues(self, sprint_id: int) -> List[Dict]:
        """获取Sprint中的所有问题"""
        url = f"{self.base_url}/rest/agile/1.0/sprint/{sprint_id}/issue"
        
        issues = []
        start_at = 0
        max_results = 50
        
        while True:
            params = {"startAt": start_at, "maxResults": max_results}
            response = requests.get(url, auth=self.auth,
                                  headers=self.headers, params=params)
            
            if response.status_code != 200:
                break
                
            data = response.json()
            issues.extend(data.get("issues", []))
            
            if len(data.get("issues", [])) < max_results:
                break
            start_at += max_results
        
        return issues
    
    def generate_burndown_chart(self, sprint_id: int) -> Dict:
        """生成燃尽图数据"""
        issues = self.get_sprint_issues(sprint_id)
        
        total_story_points = 0
        completed_points = 0
        
        for issue in issues:
            fields = issue.get("fields", {})
            points = fields.get("customfield_10016", 0) or 0  # 故事点字段
            status = fields.get("status", {}).get("name", "")
            
            total_story_points += points
            if status in ["Done", "Closed", "Resolved"]:
                completed_points += points
        
        return {
            "sprint_id": sprint_id,
            "total_issues": len(issues),
            "total_story_points": total_story_points,
            "completed_points": completed_points,
            "remaining_points": total_story_points - completed_points,
            "completion_rate": round(completed_points / total_story_points * 100, 2) 
                              if total_story_points else 0
        }


class SprintCapacityPlanner:
    """Sprint容量规划器"""
    
    def __init__(self, team_members: int, hours_per_day: float = 8):
        self.team_members = team_members
        self.hours_per_day = hours_per_day
        self.holidays = []
        
    def set_holidays(self, holidays: List[str]):
        """设置节假日列表"""
        self.holidays = [datetime.strptime(h, "%Y-%m-%d").date() for h in holidays]
    
    def calculate_capacity(self, sprint_weeks: int = 2,
                          focus_factor: float = 0.7,
                          pto_days: Dict[str, int] = None) -> Dict:
        """
        计算Sprint容量
        
        Args:
            sprint_weeks: Sprint周数
            focus_factor: 专注系数（考虑会议等中断）
            pto_days: 成员休假天数 {成员名: 天数}
        """
        # 计算工作日
        today = datetime.now().date()
        total_days = sprint_weeks * 7
        end_date = today + timedelta(days=total_days)
        
        workdays = 0
        current = today
        while current < end_date:
            # 跳过周末和节假日
            if current.weekday() < 5 and current not in self.holidays:
                workdays += 1
            current += timedelta(days=1)
        
        # 计算总可用工时
        total_available_hours = workdays * self.hours_per_day * self.team_members
        
        # 减去休假时间
        pto_hours = sum(pto_days.values()) * self.hours_per_day if pto_days else 0
        
        # 应用专注系数
        effective_hours = (total_available_hours - pto_hours) * focus_factor
        
        # 估算故事点（假设1故事点=4小时）
        story_point_capacity = int(effective_hours / 4)
        
        return {
            "sprint_duration_weeks": sprint_weeks,
            "workdays": workdays,
            "total_available_hours": total_available_hours,
            "pto_hours": pto_hours,
            "focus_factor": focus_factor,
            "effective_hours": round(effective_hours, 2),
            "story_point_capacity": story_point_capacity,
            "team_members": self.team_members,
            "holidays_in_sprint": [str(h) for h in self.holidays 
                                   if today <= h < end_date]
        }


# CLI工具
if __name__ == "__main__":
    import argparse
    
    parser = argparse.ArgumentParser(description="Sprint规划工具")
    parser.add_argument("--capacity", action="store_true", help="计算团队容量")
    parser.add_argument("--members", type=int, default=5, help="团队成员数")
    parser.add_argument("--weeks", type=int, default=2, help="Sprint周数")
    parser.add_argument("--focus", type=float, default=0.7, help="专注系数")
    
    args = parser.parse_args()
    
    if args.capacity:
        planner = SprintCapacityPlanner(team_members=args.members)
        capacity = planner.calculate_capacity(
            sprint_weeks=args.weeks,
            focus_factor=args.focus
        )
        print("\n=== Sprint容量规划 ===")
        for key, value in capacity.items():
            print(f"{key}: {value}")
```

### 3.3 看板可视化工具

```python
#!/usr/bin/env python3
"""
看板板可视化工具
生成HTML格式的看板视图
"""

from dataclasses import dataclass
from typing import List, Dict
from datetime import datetime


@dataclass
class KanbanCard:
    """看板卡片"""
    id: str
    title: str
    assignee: str
    priority: str
    tags: List[str]
    blocked: bool = False
    blocked_reason: str = ""
    created_at: datetime = None
    
    def __post_init__(self):
        if self.created_at is None:
            self.created_at = datetime.now()


class KanbanColumn:
    """看板列"""
    
    def __init__(self, name: str, wip_limit: int = None):
        self.name = name
        self.wip_limit = wip_limit
        self.cards: List[KanbanCard] = []
    
    def add_card(self, card: KanbanCard) -> bool:
        """添加卡片，检查WIP限制"""
        if self.wip_limit and len(self.cards) >= self.wip_limit:
            return False
        self.cards.append(card)
        return True
    
    def remove_card(self, card_id: str) -> KanbanCard:
        """移除卡片"""
        for i, card in enumerate(self.cards):
            if card.id == card_id:
                return self.cards.pop(i)
        return None
    
    def is_over_limit(self) -> bool:
        """检查是否超过WIP限制"""
        return self.wip_limit is not None and len(self.cards) > self.wip_limit
    
    def get_cycle_time_stats(self) -> Dict:
        """获取周期时间统计"""
        if not self.cards:
            return {}
        
        cycle_times = [
            (datetime.now() - card.created_at).days 
            for card in self.cards
        ]
        
        return {
            "count": len(self.cards),
            "avg_days": sum(cycle_times) / len(cycle_times),
            "max_days": max(cycle_times),
            "min_days": min(cycle_times)
        }


class KanbanBoard:
    """看板板"""
    
    def __init__(self, name: str):
        self.name = name
        self.columns: List[KanbanColumn] = []
        self.flow_metrics = []
    
    def add_column(self, column: KanbanColumn):
        self.columns.append(column)
    
    def move_card(self, card_id: str, from_column: str, to_column: str) -> bool:
        """移动卡片"""
        source = next((c for c in self.columns if c.name == from_column), None)
        target = next((c for c in self.columns if c.name == to_column), None)
        
        if not source or not target:
            return False
        
        card = source.remove_card(card_id)
        if not card:
            return False
        
        if not target.add_card(card):
            source.add_card(card)  # 回滚
            return False
        
        # 记录流指标
        self.flow_metrics.append({
            "card_id": card_id,
            "from": from_column,
            "to": to_column,
            "timestamp": datetime.now()
        })
        
        return True
    
    def generate_html(self) -> str:
        """生成HTML看板"""
        html = f"""<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>{self.name} - Kanban Board</title>
    <style>
        body {{
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            background: #f5f5f5;
            margin: 0;
            padding: 20px;
        }}
        .board {{
            display: flex;
            gap: 20px;
            overflow-x: auto;
        }}
        .column {{
            min-width: 280px;
            background: #ebecf0;
            border-radius: 8px;
            padding: 12px;
        }}
        .column-header {{
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 12px;
            font-weight: 600;
            color: #172b4d;
        }}
        .wip-limit {{
            background: {'#ff5630' if self.columns[0].is_over_limit() else '#0052cc'};
            color: white;
            padding: 2px 8px;
            border-radius: 12px;
            font-size: 12px;
        }}
        .card {{
            background: white;
            border-radius: 6px;
            padding: 12px;
            margin-bottom: 8px;
            box-shadow: 0 1px 3px rgba(0,0,0,0.1);
            cursor: pointer;
            transition: transform 0.2s;
        }}
        .card:hover {{
            transform: translateY(-2px);
            box-shadow: 0 4px 8px rgba(0,0,0,0.15);
        }}
        .card-blocked {{
            border-left: 4px solid #ff5630;
        }}
        .card-priority-high {{
            border-top: 3px solid #ff5630;
        }}
        .card-priority-medium {{
            border-top: 3px solid #ffab00;
        }}
        .card-priority-low {{
            border-top: 3px solid #36b37e;
        }}
        .card-id {{
            font-size: 11px;
            color: #5e6c84;
            margin-bottom: 4px;
        }}
        .card-title {{
            font-size: 14px;
            color: #172b4d;
            margin-bottom: 8px;
            line-height: 1.4;
        }}
        .card-meta {{
            display: flex;
            justify-content: space-between;
            align-items: center;
            font-size: 12px;
        }}
        .card-assignee {{
            background: #dfe1e6;
            padding: 2px 8px;
            border-radius: 12px;
        }}
        .card-tags {{
            display: flex;
            gap: 4px;
            flex-wrap: wrap;
            margin-top: 8px;
        }}
        .tag {{
            font-size: 10px;
            padding: 2px 6px;
            border-radius: 4px;
            background: #e3fcef;
            color: #006644;
        }}
        .blocked-badge {{
            background: #ff5630;
            color: white;
            font-size: 10px;
            padding: 2px 6px;
            border-radius: 4px;
            margin-top: 8px;
        }}
    </style>
</head>
<body>
    <h1>{self.name}</h1>
    <div class="board">
"""
        
        for column in self.columns:
            wip_class = "wip-exceeded" if column.is_over_limit() else ""
            html += f"""
        <div class="column {wip_class}">
            <div class="column-header">
                <span>{column.name}</span>
                <span class="wip-limit">{len(column.cards)}/{column.wip_limit if column.wip_limit else '∞'}</span>
            </div>
"""
            for card in column.cards:
                priority_class = f"card-priority-{card.priority.lower()}"
                blocked_class = "card-blocked" if card.blocked else ""
                
                html += f"""
            <div class="card {priority_class} {blocked_class}">
                <div class="card-id">{card.id}</div>
                <div class="card-title">{card.title}</div>
                <div class="card-meta">
                    <span class="card-assignee">{card.assignee}</span>
                </div>
                <div class="card-tags">
                    {''.join(f'<span class="tag">{tag}</span>' for tag in card.tags)}
                </div>
                {f'<div class="blocked-badge">阻塞: {card.blocked_reason}</div>' if card.blocked else ''}
            </div>
"""
            html += "        </div>\n"
        
        html += """
    </div>
</body>
</html>
"""
        return html
    
    def export_html(self, filepath: str):
        """导出为HTML文件"""
        with open(filepath, 'w', encoding='utf-8') as f:
            f.write(self.generate_html())


# 使用示例
if __name__ == "__main__":
    # 创建看板
    board = KanbanBoard("产品迭代看板")
    
    # 添加列
    board.add_column(KanbanColumn("待办", wip_limit=10))
    board.add_column(KanbanColumn("分析中", wip_limit=3))
    board.add_column(KanbanColumn("开发中", wip_limit=5))
    board.add_column(KanbanColumn("测试中", wip_limit=3))
    board.add_column(KanbanColumn("已完成"))
    
    # 添加卡片
    todo_col = board.columns[0]
    todo_col.add_card(KanbanCard(
        id="KAN-001",
        title="实现用户登录功能",
        assignee="张三",
        priority="High",
        tags=["前端", "认证"]
    ))
    todo_col.add_card(KanbanCard(
        id="KAN-002",
        title="优化数据库查询性能",
        assignee="李四",
        priority="Medium",
        tags=["后端", "性能"],
        blocked=True,
        blocked_reason="等待DBA资源"
    ))
    
    # 导出
    board.export_html("kanban_board.html")
    print("看板已导出到 kanban_board.html")
```

### 3.4 持续改进 - 回顾会议工具

```python
#!/usr/bin/env python3
"""
Sprint回顾会议工具
自动化收集反馈、生成行动项
"""

from dataclasses import dataclass, field
from typing import List, Dict
from datetime import datetime
from enum import Enum
import json
import random


class FeedbackType(Enum):
    WENT_WELL = "做得好的"
    NEEDS_IMPROVEMENT = "需要改进的"
    ACTION_ITEM = "行动项"


@dataclass
class RetrospectiveFeedback:
    """回顾反馈项"""
    id: str
    content: str
    feedback_type: FeedbackType
    author: str
    votes: int = 0
    voters: List[str] = field(default_factory=list)
    category: str = "general"  # process, communication, tools, etc.
    created_at: datetime = field(default_factory=datetime.now)
    
    def vote(self, voter: str) -> bool:
        """投票"""
        if voter not in self.voters:
            self.voters.append(voter)
            self.votes += 1
            return True
        return False


@dataclass
class ActionItem:
    """行动项"""
    id: str
    description: str
    owner: str
    due_date: datetime
    status: str = "open"  # open, in_progress, done
    related_feedback: List[str] = field(default_factory=list)
    
    def to_dict(self) -> Dict:
        return {
            "id": self.id,
            "description": self.description,
            "owner": self.owner,
            "due_date": self.due_date.isoformat(),
            "status": self.status,
            "related_feedback": self.related_feedback
        }


class RetrospectiveMeeting:
    """回顾会议管理"""
    
    # 预设的回顾格式
    FORMATS = {
        "standard": ["做得好的", "需要改进的", "行动项"],
        "4ls": ["喜欢的", "学到的", "缺乏的", "渴望的"],
        "sailboat": ["风（助力）", "锚（阻碍）", "礁石（风险）", "目标（愿景）"],
        "mad_sad_glad": ["愤怒的", "悲伤的", "高兴的"],
        "starfish": ["继续做的", "少做些的", "多做的", "开始做的", "停止做的"]
    }
    
    def __init__(self, sprint_name: str, format_type: str = "standard"):
        self.sprint_name = sprint_name
        self.format_type = format_type
        self.feedback_items: List[RetrospectiveFeedback] = []
        self.action_items: List[ActionItem] = []
        self.participants: List[str] = []
        self.meeting_date = datetime.now()
        
    def add_feedback(self, content: str, feedback_type: FeedbackType,
                    author: str, category: str = "general") -> RetrospectiveFeedback:
        """添加反馈"""
        feedback = RetrospectiveFeedback(
            id=f"FDB-{len(self.feedback_items)+1:03d}",
            content=content,
            feedback_type=feedback_type,
            author=author,
            category=category
        )
        self.feedback_items.append(feedback)
        if author not in self.participants:
            self.participants.append(author)
        return feedback
    
    def generate_action_items(self, min_votes: int = 2) -> List[ActionItem]:
        """基于高票反馈生成行动项"""
        high_vote_items = [
            f for f in self.feedback_items 
            if f.feedback_type == FeedbackType.NEEDS_IMPROVEMENT 
            and f.votes >= min_votes
        ]
        
        # 按投票数排序，取前3个
        top_items = sorted(high_vote_items, key=lambda x: x.votes, reverse=True)[:3]
        
        for item in top_items:
            action = ActionItem(
                id=f"ACT-{len(self.action_items)+1:03d}",
                description=f"改进: {item.content}",
                owner="待分配",
                due_date=self.meeting_date + timedelta(weeks=2),
                related_feedback=[item.id]
            )
            self.action_items.append(action)
        
        return self.action_items
    
    def get_sentiment_analysis(self) -> Dict:
        """简单的情感分析"""
        total = len(self.feedback_items)
        if total == 0:
            return {}
        
        positive = len([f for f in self.feedback_items 
                       if f.feedback_type == FeedbackType.WENT_WELL])
        negative = len([f for f in self.feedback_items 
                       if f.feedback_type == FeedbackType.NEEDS_IMPROVEMENT])
        
        return {
            "total_feedback": total,
            "positive_ratio": round(positive / total * 100, 2),
            "negative_ratio": round(negative / total * 100, 2),
            "sentiment_score": round((positive - negative) / total, 2),
            "trend": "positive" if positive > negative else "negative" if negative > positive else "neutral"
        }
    
    def generate_report(self) -> str:
        """生成回顾会议报告"""
        report = f"""
# Sprint回顾报告

## 基本信息
- Sprint: {self.sprint_name}
- 日期: {self.meeting_date.strftime("%Y-%m-%d")}
- 参与人数: {len(self.participants)}
- 回顾格式: {self.format_type}

## 情感分析
{json.dumps(self.get_sentiment_analysis(), indent=2, ensure_ascii=False)}

## 反馈汇总

### 做得好的 ({len([f for f in self.feedback_items if f.feedback_type == FeedbackType.WENT_WELL])})
"""
        for item in self.feedback_items:
            if item.feedback_type == FeedbackType.WENT_WELL:
                report += f"- {item.content} (👍 {item.votes})\n"
        
        report += "\n### 需要改进的\n"
        for item in self.feedback_items:
            if item.feedback_type == FeedbackType.NEEDS_IMPROVEMENT:
                report += f"- {item.content} (👍 {item.votes})\n"
        
        report += "\n## 行动项\n"
        for item in self.action_items:
            report += f"- [ ] {item.description}\n"
            report += f"  - 负责人: {item.owner}\n"
            report += f"  - 截止日期: {item.due_date.strftime('%Y-%m-%d')}\n"
        
        return report
    
    def export_report(self, filepath: str):
        """导出报告"""
        with open(filepath, 'w', encoding='utf-8') as f:
            f.write(self.generate_report())


# 使用示例
if __name__ == "__main__":
    # 创建回顾会议
    retro = RetrospectiveMeeting("Sprint 42", format_type="standard")
    
    # 收集反馈
    retro.add_feedback(
        "每日站会效率提高，大家都准时参加",
        FeedbackType.WENT_WELL,
        "张三",
        "communication"
    )
    retro.add_feedback(
        "代码审查时间过长，影响开发效率",
        FeedbackType.NEEDS_IMPROVEMENT,
        "李四",
        "process"
    )
    retro.add_feedback(
        "CI/CD流水线稳定性提升",
        FeedbackType.WENT_WELL,
        "王五",
        "tools"
    )
    
    # 模拟投票
    for item in retro.feedback_items:
        item.vote("成员A")
        item.vote("成员B")
        if item.content.startswith("代码审查"):
            item.vote("成员C")
    
    # 生成行动项
    retro.generate_action_items(min_votes=2)
    
    # 导出报告
    retro.export_report("sprint_retrospective.md")
    print(retro.generate_report())
```

### 3.5 敏捷指标仪表板

```python
#!/usr/bin/env python3
"""
敏捷指标仪表板生成器
生成包含关键敏捷指标的可视化报告
"""

import matplotlib.pyplot as plt
import matplotlib.dates as mdates
from datetime import datetime, timedelta
from typing import List, Dict
import json


class AgileMetricsDashboard:
    """敏捷指标仪表板"""
    
    def __init__(self, team_name: str):
        self.team_name = team_name
        self.sprint_data = []
        self.velocity_history = []
        self.cycle_times = []
        
    def add_sprint_data(self, sprint_data: Dict):
        """添加Sprint数据"""
        self.sprint_data.append(sprint_data)
    
    def calculate_lead_time(self, created_date: datetime, 
                          completed_date: datetime) -> int:
        """计算前置时间（天）"""
        return (completed_date - created_date).days
    
    def calculate_cycle_time(self, started_date: datetime,
                           completed_date: datetime) -> int:
        """计算周期时间（天）"""
        return (completed_date - started_date).days
    
    def calculate_velocity(self, completed_story_points: List[int]) -> Dict:
        """计算速度指标"""
        if not completed_story_points:
            return {}
        
        n = len(completed_story_points)
        avg = sum(completed_story_points) / n
        
        # 计算标准差
        variance = sum((x - avg) ** 2 for x in completed_story_points) / n
        std_dev = variance ** 0.5
        
        return {
            "average_velocity": round(avg, 2),
            "min_velocity": min(completed_story_points),
            "max_velocity": max(completed_story_points),
            "std_deviation": round(std_dev, 2),
            "predictability": round((1 - std_dev/avg) * 100, 2) if avg else 0
        }
    
    def calculate_flow_efficiency(self, active_time: float, 
                                 wait_time: float) -> float:
        """
        计算流动效率
        流动效率 = 活跃时间 / (活跃时间 + 等待时间)
        """
        total_time = active_time + wait_time
        return round(active_time / total_time * 100, 2) if total_time else 0
    
    def generate_velocity_chart(self, output_path: str = "velocity_chart.png"):
        """生成速度图"""
        sprints = [f"Sprint {i+1}" for i in range(len(self.velocity_history))]
        velocities = self.velocity_history
        
        if len(velocities) < 2:
            print("需要至少2个Sprint的速度数据")
            return
        
        fig, ax = plt.subplots(figsize=(12, 6))
        
        # 柱状图
        bars = ax.bar(sprints, velocities, color='#0052cc', alpha=0.7)
        
        # 平均线
        avg_velocity = sum(velocities) / len(velocities)
        ax.axhline(y=avg_velocity, color='#ff5630', linestyle='--', 
                  label=f'平均速度: {avg_velocity:.1f}')
        
        # 预测区间（±1标准差）
        variance = sum((v - avg_velocity) ** 2 for v in velocities) / len(velocities)
        std_dev = variance ** 0.5
        ax.fill_between(range(len(sprints)), 
                       [avg_velocity - std_dev] * len(sprints),
                       [avg_velocity + std_dev] * len(sprints),
                       alpha=0.2, color='#ff5630', label=f'预测区间 (±{std_dev:.1f})')
        
        ax.set_xlabel('Sprint')
        ax.set_ylabel('故事点')
        ax.set_title(f'{self.team_name} - 团队速度趋势')
        ax.legend()
        ax.grid(True, alpha=0.3)
        
        plt.xticks(rotation=45)
        plt.tight_layout()
        plt.savefig(output_path, dpi=150)
        print(f"速度图已保存到: {output_path}")
    
    def generate_cfd(self, flow_data: Dict, output_path: str = "cfd.png"):
        """
        生成累积流图 (Cumulative Flow Diagram)
        
        flow_data格式: {
            "dates": [date1, date2, ...],
            "columns": {
                "待办": [count1, count2, ...],
                "进行中": [count1, count2, ...],
                ...
            }
        }
        """
        fig, ax = plt.subplots(figsize=(14, 7))
        
        dates = flow_data["dates"]
        colors = ['#0052cc', '#00b8d9', '#36b37e', '#ffab00', '#ff5630']
        
        # 计算累积值
        bottom = [0] * len(dates)
        for i, (column, values) in enumerate(flow_data["columns"].items()):
            ax.fill_between(dates, bottom, 
                          [b + v for b, v in zip(bottom, values)],
                          label=column, color=colors[i % len(colors)], alpha=0.8)
            bottom = [b + v for b, v in zip(bottom, values)]
        
        ax.set_xlabel('日期')
        ax.set_ylabel('工作项数量')
        ax.set_title(f'{self.team_name} - 累积流图')
        ax.legend(loc='upper left')
        ax.grid(True, alpha=0.3)
        
        # 日期格式化
        ax.xaxis.set_major_formatter(mdates.DateFormatter('%m-%d'))
        plt.xticks(rotation=45)
        plt.tight_layout()
        plt.savefig(output_path, dpi=150)
        print(f"累积流图已保存到: {output_path}")
    
    def generate_metrics_summary(self) -> str:
        """生成指标摘要报告"""
        velocity_metrics = self.calculate_velocity(self.velocity_history)
        
        summary = f"""
# {self.team_name} - 敏捷指标摘要

## 速度指标
- 平均速度: {velocity_metrics.get('average_velocity', 'N/A')} 故事点/Sprint
- 速度范围: {velocity_metrics.get('min_velocity', 'N/A')} - {velocity_metrics.get('max_velocity', 'N/A')}
- 可预测性: {velocity_metrics.get('predictability', 'N/A')}%

## 流动效率
健康的流动效率应在15-40%之间。
低于15%表示等待时间过长，需要优化流程。

## 建议
1. 保持Sprint长度一致，提高可预测性
2. 限制在制品数量，减少上下文切换
3. 定期进行回顾，持续改进流程
"""
        return summary


# 使用示例
if __name__ == "__main__":
    dashboard = AgileMetricsDashboard("前端开发团队")
    
    # 添加速度历史
    dashboard.velocity_history = [23, 28, 25, 30, 27, 29, 31, 26]
    
    # 生成图表
    dashboard.generate_velocity_chart()
    
    # 生成累积流图数据
    from datetime import datetime, timedelta
    dates = [datetime.now() - timedelta(days=i) for i in range(14, 0, -1)]
    cfd_data = {
        "dates": dates,
        "columns": {
            "待办": [15, 14, 16, 15, 13, 12, 14, 13, 15, 14, 16, 15, 14, 13],
            "分析中": [3, 4, 3, 4, 5, 4, 3, 4, 3, 4, 3, 3, 4, 3],
            "开发中": [5, 5, 6, 5, 5, 6, 5, 5, 6, 5, 5, 6, 5, 5],
            "测试中": [2, 3, 2, 3, 2, 3, 2, 3, 2, 3, 2, 2, 3, 2],
            "已完成": [8, 10, 12, 15, 18, 21, 24, 27, 30, 33, 36, 39, 42, 45]
        }
    }
    dashboard.generate_cfd(cfd_data)
    
    # 打印报告
    print(dashboard.generate_metrics_summary())
```

## 4. 资源索引

### 4.1 官方文档

| 资源名称 | 链接 | 说明 |
|---------|------|------|
| Scrum指南 | https://scrumguides.org | Scrum官方指南，Ken Schwaber & Jeff Sutherland |
| Kanban指南 | https://kanbanguides.org | Kanban官方指南 |
| 敏捷宣言 | https://agilemanifesto.org | 敏捷软件开发宣言原文 |
| LeSS框架 | https://less.works | 大规模Scrum官方站点 |
| SAFe框架 | https://scaledagileframework.com | 规模化敏捷框架 |

### 4.2 推荐工具

| 类别 | 工具 | 官网 | 特点 |
|-----|-----|------|-----|
| 项目管理 | Jira | https://www.atlassian.com/software/jira | 企业级敏捷管理 |
| 项目管理 | Azure DevOps | https://azure.microsoft.com/devops | 微软全链路方案 |
| 看板工具 | Trello | https://trello.com | 简洁易用的看板 |
| 看板工具 | Kanbanize | https://kanbanize.com | 高级看板分析功能 |
| 协作白板 | Miro | https://miro.com | 远程回顾会议首选 |
| 协作白板 | Mural | https://mural.co | 可视化协作平台 |
| 敏捷估算 | PlanningPoker | https://www.planningpoker.com | 在线规划扑克 |
| 指标分析 | ActionableAgile | https://actionableagile.com | 流数据分析专业工具 |

### 4.3 书籍推荐

1. **《Scrum指南》** - Ken Schwaber, Jeff Sutherland
2. **《看板方法》** - David J. Anderson
3. **《用户故事与敏捷方法》** - Mike Cohn
4. **《敏捷估计与规划》** - Mike Cohn
5. **《Succeeding with Agile》** - Mike Cohn
6. **《Essential Kanban Condensed》** - David J. Anderson
7. **《Team Topologies》** - Matthew Skelton, Manuel Pais

### 4.4 认证路径

| 认证 | 颁发机构 | 级别 |
|-----|---------|-----|
| PSM (Professional Scrum Master) | Scrum.org | 基础/中级/高级 |
| CSM (Certified ScrumMaster) | Scrum Alliance | 基础 |
| CSPO (Certified Scrum Product Owner) | Scrum Alliance | 基础 |
| ICP (ICAgile Certified Professional) | ICAgile | 多个专业方向 |
| SAFe Agilist (SA) | Scaled Agile | 规模化敏捷 |
| KMP (Kanban Management Professional) | Kanban University | Kanban专业 |

## 5. 关联知识

### 5.1 上游关联

```
A02_Engineering_Processes
├── B01_SDLC_Frameworks
│   ├── C01_Agile_Methodologies (本单元)
│   ├── C02_DevOps_Integration ──► 需要敏捷的迭代节奏支撑
│   └── C03_Quality_Gates ───────► 敏捷中的质量内建实践
```

### 5.2 下游关联

```
C01_Agile_Methodologies
├── A03_Core_Technologies
│   ├── B01_Programming_Languages ──► 技术债务管理
│   ├── B02_Software_Architecture ──► 演进式架构
│   └── B03_Infrastructure ────────► 持续交付支撑
├── A05_Operations_Excellence
│   ├── B01_Monitoring_Observability ► 敏捷反馈循环
│   └── B02_Incident_Management ────► 快速响应机制
└── A06_Leadership_Management
    ├── B01_Technical_Leadership ───► 服务型领导
    └── B03_Team_Dynamics ──────────► 高绩效团队
```

### 5.3 横向关联

- **与DevOps的融合**：敏捷提供迭代节奏，DevOps提供技术实践，共同构成现代软件交付基础
- **与精益思想的结合**：消除浪费、持续改进是两者的共同目标
- **与系统思维的整合**：从局部优化转向价值流优化

## 6. 学习建议

### 6.1 新手入门路径（1-3个月）

```
第1周：理论基础
├── 阅读《Scrum指南》（中文版约16页）
├── 观看Scrum.org官方视频
└── 完成Scrum Open评估

第2-3周：实践观摩
├── 参加真实Sprint会议（作为观察者）
├── 记录会议中的问题和亮点
└── 与Scrum Master交流

第4周：工具上手
├── 创建Jira/Confluence账户
├── 完成一个简单项目的设置
└── 练习看板配置

第2-3月：深度参与
├── 担任Scrum团队一员
├── 参与用户故事编写
└── 主持至少一次回顾会议
```

### 6.2 进阶提升路径（3-12个月）

1. **获得专业认证**：PSM I、CSM 或 ICP
2. **扩展框架知识**：学习Kanban、XP、SAFe
3. **掌握度量技能**：学习流数据分析、预测技术
4. **教练技能培养**：学习引导技术、冲突解决
5. **规模化经验**：参与多团队协作的大型敏捷项目

### 6.3 专家发展路径（1年+）

1. **获得高级认证**：PSM III、SPC、KMP
2. **成为变革推动者**：主导组织级敏捷转型
3. **贡献社区**：撰写博客、演讲分享、开源工具
4. **跨领域融合**：将敏捷思想应用到非IT领域
5. **研究与创新**：探索敏捷的新边界（AI辅助、远程协作等）

### 6.4 学习检查清单

- [ ] 能够清晰解释敏捷宣言的4个价值观
- [ ] 能够主持完整的Sprint周期所有仪式
- [ ] 能够使用至少两种用户故事估算方法
- [ ] 能够分析和改进团队的流动效率
- [ ] 能够设计和主持有效的回顾会议
- [ ] 能够指导团队实施持续改进
- [ ] 能够处理敏捷实施中的常见阻力
- [ ] 能够将敏捷原则应用到新领域

---

*最后更新：2024年 | 维护者：工程流程委员会*
