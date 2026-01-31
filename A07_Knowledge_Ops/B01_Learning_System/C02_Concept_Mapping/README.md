# C02_Concept_Mapping - 概念图谱

> 构建可视化知识结构与概念间关系的认知工具

---

## 1. 主题定位

### 1.1 定义与背景

概念图谱（Concept Mapping）是一种以图形化方式组织和表示知识的认知工具，由康奈尔大学的Joseph D. Novak及其研究团队于1970年代在David Ausubel的意义学习理论基础上开发。概念图谱通过节点（概念）和连线（关系）的方式，将抽象的知识结构转化为可视化的网络形式，帮助学习者理解概念间的层次关系和语义连接。

与思维导图（Mind Map）不同，概念图谱强调概念间的**命题关系**（Propositional Relationships），即通过连接词（Linking Words）明确表达两个概念之间的语义关系。这种结构更符合人类认知系统中对知识的存储和检索方式。

### 1.2 核心特征

| 特征维度 | 描述 |
|----------|------|
| **层级结构** | 从最一般、最包容的概念到最具体、最特殊的概念 |
| **交叉连接** | 不同分支概念间的横向关联，体现知识的整合性 |
| **命题结构** | 概念-连接词-概念的三元组构成有意义的陈述 |
| **焦点问题** | 围绕一个核心问题或主题展开整个图谱 |

### 1.3 应用场景

- **教学设计**：帮助教师组织课程内容，识别学生的先验知识
- **知识评估**：通过分析学习者构建的概念图谱评估理解深度
- **知识整合**：将新知识与已有知识框架连接
- **协作学习**：团队成员共同构建共享知识模型
- **复杂系统分析**：理解软件架构、业务流程等复杂系统

---

## 2. 核心概念

### 2.1 理论基础：意义学习

David Ausubel的意义学习理论（Meaningful Learning Theory）是概念图谱的理论基石。该理论认为：

```
意义学习的条件：
1. 学习材料本身具有逻辑意义
2. 学习者具有相关的先验知识
3. 学习者具有意义学习的心向（非机械记忆）

关键机制：
- 附属学习（Subsumption）：新知识归属于已有认知结构中
- 总括学习（Superordinate Learning）：已有知识归属于更一般的概念
- 组合学习（Combinatorial Learning）：新知识与已有知识并列结合
```

### 2.2 概念图谱要素

#### 2.2.1 概念（Concepts）

概念是感知到的规律性或事件，以符号（通常是词汇）标记：

```
概念分类：
├─ 具体概念：可直接感知的对象（如：服务器、数据库）
├─ 抽象概念：需要抽象思维（如：并发、解耦）
├─ 过程概念：描述操作或流程（如：负载均衡、故障转移）
└─ 元概念：关于知识的概念（如：设计模式、架构风格）
```

#### 2.2.2 关系连接（Linking Words/Phrases）

连接词定义了概念间的关系类型：

| 关系类型 | 连接词示例 | 示例命题 |
|----------|------------|----------|
| **包含** | 包含、由...组成、有 | 微服务架构包含服务发现 |
| **导致** | 导致、产生、引起 | 高并发导致资源竞争 |
| **属性** | 具有、特征为 | 分布式系统具有CAP特性 |
| **示例** | 例如、如、实例 | 消息队列如Kafka |
| **依赖** | 依赖、需要、基于 | 缓存策略依赖于访问模式 |
| **对比** | 不同于、相比、而非 | REST不同于RPC |

#### 2.2.3 层级结构

概念图谱采用从上到下的层级组织：

```
                    [最一般概念]
                          │
            ┌─────────────┼─────────────┐
            ▼             ▼             ▼
       [子概念A]     [子概念B]     [子概念C]
            │             │             │
      ┌─────┴─────┐   ┌───┴───┐   ┌─────┴─────┐
      ▼           ▼   ▼       ▼   ▼           ▼
   [具体A1]   [具体A2] [B1]  [B2] [C1]     [具体C2]
   
层级原则：
- 上层概念包含下层概念
- 同一层级概念具有相似的一般性程度
- 避免跨层级直接连接
```

#### 2.2.4 交叉连接（Cross Links）

交叉连接是不同分支概念间的关系，是概念图谱的重要特征：

```
        [分布式系统]                          [数据库系统]
              │                                    │
      ┌───────┴───────┐                  ┌─────────┴─────────┐
      ▼               ▼                  ▼                   ▼
  [一致性模型]    [分区容错]          [ACID]             [BASE]
      │               │                  │                   │
      └───────────────┴──────────────────┘                   │
                      │                                      │
                      ▼                                      ▼
              [CAP定理] ──────────────────────────→ [最终一致性]
              
交叉连接揭示：
- 不同领域概念间的深层联系
- 知识整合和创新思维的基础
- 高级认知能力的标志
```

### 2.3 概念图谱类型

#### 2.3.1 蜘蛛型（Spoke/Radial）

```
              [中心概念]
            /    │    \
          /      │      \
   [概念A]  [概念B]  [概念C]  [概念D]
        \        │        /
         \       │       /
          \      │      /
           \     │     /
            \    │    /
             \   │   /
              \  │  /
               \ │ /
            [整合概念]

适用场景：中心辐射式知识，如技术选型比较
```

#### 2.3.2 层级型（Hierarchical）

```
                    [根概念]
                   /   |   \
                  /    |    \
              [L1-A] [L1-B] [L1-C]
                / \      |      \
               /   \     |       \
           [L2-A1][L2-A2][L2-B1] [L2-C1]
            /         \    |        \
        [L3-A1a]  [L3-A2a] ...    ...

适用场景：分类体系、架构层次、继承关系
```

#### 2.3.3 流程型（Flow/Chain）

```
[输入] ──→ [处理1] ──→ [处理2] ──→ [处理3] ──→ [输出]
            │           │           │
            ▼           ▼           ▼
        [反馈1]     [监控]      [优化]
            │           │           │
            └───────────┴───────────┘
                        │
                        ▼
                   [回环到输入]

适用场景：算法流程、系统交互、状态转换
```

#### 2.3.4 系统型（System）

```
           ┌──────────────────────────────────────┐
           │           [系统边界]                  │
           │                                      │
    [外部实体A] ←───→ [核心组件] ←───→ [外部实体B]
           │              │                │
           │         ┌────┴────┐           │
           │         ▼         ▼           │
           │    [子系统X]  [子系统Y]        │
           │         │         │            │
           │         └────┬────┘            │
           │              │                 │
           │              ▼                 │
           │         [共享资源]              │
           │                              │
           └──────────────────────────────────────┘

适用场景：系统架构、生态分析、复杂交互
```

---

## 3. 技术实践

### 3.1 概念图谱构建流程

#### 3.1.1 系统化构建步骤

```python
from enum import Enum, auto
from dataclasses import dataclass, field
from typing import List, Dict, Set, Optional, Tuple
import json
import uuid

class RelationType(Enum):
    """关系类型枚举"""
    IS_A = auto()              # 是一种
    PART_OF = auto()           # 是...的一部分
    HAS = auto()               # 具有
    LEADS_TO = auto()          # 导致
    DEPENDS_ON = auto()        # 依赖于
    EXAMPLE_OF = auto()        # 是...的例子
    CONTRASTS_WITH = auto()    # 与...对比
    INFLUENCES = auto()        # 影响
    ENABLES = auto()           # 使能
    PREVENTS = auto()          # 阻止

RELATION_PHRASES = {
    RelationType.IS_A: ["是一种", "是", "属于"],
    RelationType.PART_OF: ["是...的一部分", "组成", "包含于"],
    RelationType.HAS: ["具有", "拥有", "特征为"],
    RelationType.LEADS_TO: ["导致", "产生", "引起", "造成"],
    RelationType.DEPENDS_ON: ["依赖于", "需要", "基于"],
    RelationType.EXAMPLE_OF: ["是...的例子", "例如", "如"],
    RelationType.CONTRASTS_WITH: ["与...对比", "不同于", "而非"],
    RelationType.INFLUENCES: ["影响", "作用于", "改变"],
    RelationType.ENABLES: ["使能", "允许", "支持"],
    RelationType.PREVENTS: ["阻止", "防止", "避免"]
}

@dataclass
class Concept:
    """概念节点"""
    id: str
    label: str
    description: str = ""
    category: str = "general"  # 概念类别
    level: int = 0  # 层级（0为最高层）
    examples: List[str] = field(default_factory=list)
    tags: List[str] = field(default_factory=list)
    
    def __post_init__(self):
        if not self.id:
            self.id = str(uuid.uuid4())[:8]

@dataclass
class Proposition:
    """命题：概念之间的关系"""
    id: str
    source_id: str
    target_id: str
    relation_type: RelationType
    linking_phrase: str
    bidirectional: bool = False
    
    def __post_init__(self):
        if not self.id:
            self.id = str(uuid.uuid4())[:8]
    
    def to_statement(self, concepts: Dict[str, Concept]) -> str:
        """转换为自然语言陈述"""
        source = concepts.get(self.source_id, Concept("", "?")).label
        target = concepts.get(self.target_id, Concept("", "?")).label
        return f"{source} {self.linking_phrase} {target}"

class ConceptMap:
    """概念图谱核心类"""
    
    def __init__(self, title: str, focus_question: str = ""):
        self.id = str(uuid.uuid4())[:8]
        self.title = title
        self.focus_question = focus_question
        self.concepts: Dict[str, Concept] = {}
        self.propositions: Dict[str, Proposition] = {}
        self.root_concept_id: Optional[str] = None
        
    def add_concept(self, label: str, description: str = "", 
                    category: str = "general", level: int = None,
                    parent_id: str = None) -> Concept:
        """添加概念节点"""
        # 自动计算层级
        if level is None and parent_id:
            parent = self.concepts.get(parent_id)
            if parent:
                level = parent.level + 1
            else:
                level = 0
        elif level is None:
            level = 0
            
        concept = Concept(
            id=str(uuid.uuid4())[:8],
            label=label,
            description=description,
            category=category,
            level=level
        )
        self.concepts[concept.id] = concept
        
        # 设置根概念
        if level == 0 and not self.root_concept_id:
            self.root_concept_id = concept.id
            
        return concept
    
    def add_proposition(self, source_id: str, target_id: str,
                       relation_type: RelationType = RelationType.IS_A,
                       linking_phrase: str = None,
                       bidirectional: bool = False) -> Proposition:
        """添加命题（关系）"""
        if linking_phrase is None:
            linking_phrase = RELATION_PHRASES[relation_type][0]
            
        prop = Proposition(
            id=str(uuid.uuid4())[:8],
            source_id=source_id,
            target_id=target_id,
            relation_type=relation_type,
            linking_phrase=linking_phrase,
            bidirectional=bidirectional
        )
        self.propositions[prop.id] = prop
        return prop
    
    def get_concept_by_label(self, label: str) -> Optional[Concept]:
        """通过标签查找概念"""
        for concept in self.concepts.values():
            if concept.label == label:
                return concept
        return None
    
    def get_children(self, concept_id: str) -> List[Concept]:
        """获取子概念（通过关系连接）"""
        children = []
        for prop in self.propositions.values():
            if prop.source_id == concept_id:
                child = self.concepts.get(prop.target_id)
                if child:
                    children.append(child)
        return children
    
    def get_parents(self, concept_id: str) -> List[Concept]:
        """获取父概念"""
        parents = []
        for prop in self.propositions.values():
            if prop.target_id == concept_id:
                parent = self.concepts.get(prop.source_id)
                if parent:
                    parents.append(parent)
        return parents
    
    def find_cross_links(self) -> List[Proposition]:
        """查找交叉连接（不同分支间的连接）"""
        cross_links = []
        
        # 获取每个概念的层级路径
        def get_path(concept_id: str, visited: Set[str] = None) -> List[str]:
            if visited is None:
                visited = set()
            if concept_id in visited:
                return []
            visited.add(concept_id)
            
            concept = self.concepts.get(concept_id)
            if not concept or concept.level == 0:
                return [concept_id]
            
            parents = self.get_parents(concept_id)
            if not parents:
                return [concept_id]
            
            # 返回层级最高的父节点
            top_parent = min(parents, key=lambda x: x.level)
            return get_path(top_parent.id, visited) + [concept_id]
        
        for prop in self.propositions.values():
            source_path = get_path(prop.source_id)
            target_path = get_path(prop.target_id)
            
            # 如果顶级祖先不同，则是交叉连接
            if source_path and target_path and source_path[0] != target_path[0]:
                cross_links.append(prop)
        
        return cross_links
    
    def calculate_complexity(self) -> Dict[str, any]:
        """计算概念图谱复杂度指标"""
        n_concepts = len(self.concepts)
        n_propositions = len(self.propositions)
        
        # 网络密度
        max_possible_links = n_concepts * (n_concepts - 1) / 2 if n_concepts > 1 else 1
        density = n_propositions / max_possible_links if max_possible_links > 0 else 0
        
        # 平均度数
        degrees = {}
        for concept_id in self.concepts:
            degree = sum(1 for p in self.propositions.values() 
                        if p.source_id == concept_id or p.target_id == concept_id)
            degrees[concept_id] = degree
        avg_degree = sum(degrees.values()) / n_concepts if n_concepts > 0 else 0
        
        # 层级深度
        max_depth = max(c.level for c in self.concepts.values()) if self.concepts else 0
        
        # 交叉连接数
        cross_links = len(self.find_cross_links())
        
        return {
            "concepts": n_concepts,
            "propositions": n_propositions,
            "density": round(density, 4),
            "avg_degree": round(avg_degree, 2),
            "max_depth": max_depth,
            "cross_links": cross_links,
            "hierarchy_ratio": 1 - (cross_links / n_propositions) if n_propositions > 0 else 0
        }
    
    def export_to_cmap(self, filepath: str):
        """导出为CmapTools格式"""
        # CmapTools使用XML格式
        import xml.etree.ElementTree as ET
        
        root = ET.Element("cmap")
        root.set("xmlns", "http://cmap.ihmc.us/xml/cmap/")
        
        # 添加概念
        concepts_elem = ET.SubElement(root, "concepts")
        for concept in self.concepts.values():
            c_elem = ET.SubElement(concepts_elem, "concept")
            c_elem.set("id", concept.id)
            c_elem.set("label", concept.label)
        
        # 添加连接
        connections = ET.SubElement(root, "connections")
        for prop in self.propositions.values():
            conn = ET.SubElement(connections, "connection")
            conn.set("id", prop.id)
            conn.set("from", prop.source_id)
            conn.set("to", prop.target_id)
            conn.set("label", prop.linking_phrase)
        
        tree = ET.ElementTree(root)
        tree.write(filepath, encoding='utf-8', xml_declaration=True)
    
    def export_to_json(self, filepath: str):
        """导出为JSON格式"""
        data = {
            "id": self.id,
            "title": self.title,
            "focus_question": self.focus_question,
            "concepts": [
                {
                    "id": c.id,
                    "label": c.label,
                    "description": c.description,
                    "category": c.category,
                    "level": c.level,
                    "examples": c.examples,
                    "tags": c.tags
                }
                for c in self.concepts.values()
            ],
            "propositions": [
                {
                    "id": p.id,
                    "source_id": p.source_id,
                    "target_id": p.target_id,
                    "relation_type": p.relation_type.name,
                    "linking_phrase": p.linking_phrase,
                    "bidirectional": p.bidirectional
                }
                for p in self.propositions.values()
            ],
            "root_concept_id": self.root_concept_id,
            "complexity": self.calculate_complexity()
        }
        
        with open(filepath, 'w', encoding='utf-8') as f:
            json.dump(data, f, ensure_ascii=False, indent=2)
    
    @classmethod
    def from_json(cls, filepath: str) -> 'ConceptMap':
        """从JSON加载概念图谱"""
        with open(filepath, 'r', encoding='utf-8') as f:
            data = json.load(f)
        
        cmap = cls(data['title'], data.get('focus_question', ''))
        cmap.id = data['id']
        cmap.root_concept_id = data.get('root_concept_id')
        
        # 加载概念
        for c_data in data['concepts']:
            concept = Concept(
                id=c_data['id'],
                label=c_data['label'],
                description=c_data.get('description', ''),
                category=c_data.get('category', 'general'),
                level=c_data.get('level', 0),
                examples=c_data.get('examples', []),
                tags=c_data.get('tags', [])
            )
            cmap.concepts[concept.id] = concept
        
        # 加载命题
        for p_data in data['propositions']:
            prop = Proposition(
                id=p_data['id'],
                source_id=p_data['source_id'],
                target_id=p_data['target_id'],
                relation_type=RelationType[p_data['relation_type']],
                linking_phrase=p_data['linking_phrase'],
                bidirectional=p_data.get('bidirectional', False)
            )
            cmap.propositions[prop.id] = prop
        
        return cmap


# 示例：构建"微服务架构"概念图谱
def create_microservices_concept_map() -> ConceptMap:
    """创建微服务架构概念图谱示例"""
    
    cmap = ConceptMap(
        title="微服务架构",
        focus_question="微服务架构的核心概念和组件是什么？"
    )
    
    # 添加核心概念
    root = cmap.add_concept("微服务架构", "一种将应用构建为小型服务集合的架构风格", "architecture", 0)
    cmap.root_concept_id = root.id
    
    # 第一层概念
    service = cmap.add_concept("服务", "独立部署的业务功能单元", "component", 1, root.id)
    comm = cmap.add_concept("服务通信", "服务间交互机制", "mechanism", 1, root.id)
    data = cmap.add_concept("数据管理", "分布式数据策略", "mechanism", 1, root.id)
    governance = cmap.add_concept("服务治理", "服务运行时的管理", "management", 1, root.id)
    
    # 添加关系
    cmap.add_proposition(root.id, service.id, RelationType.HAS, "包含")
    cmap.add_proposition(root.id, comm.id, RelationType.REQUIRES, "需要")
    cmap.add_proposition(root.id, data.id, RelationType.INVOLVES, "涉及")
    cmap.add_proposition(root.id, governance.id, RelationType.NEEDS, "依赖")
    
    # 第二层：服务特性
    independence = cmap.add_concept("独立性", "服务可独立开发部署", "characteristic", 2)
    bounded_ctx = cmap.add_concept("限界上下文", "领域边界划分", "pattern", 2)
    cmap.add_proposition(service.id, independence.id, RelationType.HAS, "具有")
    cmap.add_proposition(service.id, bounded_ctx.id, RelationType.IMPLEMENTS, "实现")
    
    # 第二层：通信方式
    sync = cmap.add_concept("同步通信", "请求-响应模式", "pattern", 2)
    async_comm = cmap.add_concept("异步通信", "消息驱动模式", "pattern", 2)
    cmap.add_proposition(comm.id, sync.id, RelationType.INCLUDES, "包括")
    cmap.add_proposition(comm.id, async_comm.id, RelationType.INCLUDES, "包括")
    
    # 具体技术示例
    http = cmap.add_concept("HTTP/REST", "同步通信协议", "technology", 3)
    grpc = cmap.add_concept("gRPC", "高性能RPC框架", "technology", 3)
    cmap.add_proposition(sync.id, http.id, RelationType.EXAMPLE_OF, "例如")
    cmap.add_proposition(sync.id, grpc.id, RelationType.EXAMPLE_OF, "例如")
    
    kafka = cmap.add_concept("Kafka", "分布式消息队列", "technology", 3)
    rabbitmq = cmap.add_concept("RabbitMQ", "消息中间件", "technology", 3)
    cmap.add_proposition(async_comm.id, kafka.id, RelationType.EXAMPLE_OF, "例如")
    cmap.add_proposition(async_comm.id, rabbitmq.id, RelationType.EXAMPLE_OF, "例如")
    
    # 数据管理
    db_per_service = cmap.add_concept("每服务数据库", "数据隔离原则", "pattern", 2)
    sagas = cmap.add_concept("Saga模式", "分布式事务处理", "pattern", 2)
    cmap.add_proposition(data.id, db_per_service.id, RelationType.USES, "使用")
    cmap.add_proposition(data.id, sagas.id, RelationType.REQUIRES, "需要")
    
    # 服务治理
    discovery = cmap.add_concept("服务发现", "动态服务定位", "mechanism", 2)
    config = cmap.add_concept("配置中心", "集中配置管理", "mechanism", 2)
    gateway = cmap.add_concept("API网关", "统一入口", "component", 2)
    cmap.add_proposition(governance.id, discovery.id, RelationType.INCLUDES, "包括")
    cmap.add_proposition(governance.id, config.id, RelationType.INCLUDES, "包括")
    cmap.add_proposition(governance.id, gateway.id, RelationType.INCLUDES, "包括")
    
    # 交叉连接：发现Saga与异步通信的关系
    cmap.add_proposition(sagas.id, async_comm.id, RelationType.DEPENDS_ON, "依赖于", bidirectional=True)
    
    # 交叉连接：API网关与同步通信
    cmap.add_proposition(gateway.id, sync.id, RelationType.USES, "使用")
    
    return cmap


# 运行示例
if __name__ == "__main__":
    cmap = create_microservices_concept_map()
    
    print("=" * 60)
    print(f"概念图谱: {cmap.title}")
    print(f"焦点问题: {cmap.focus_question}")
    print("=" * 60)
    
    print("\n📊 复杂度分析:")
    complexity = cmap.calculate_complexity()
    for key, value in complexity.items():
        print(f"  {key}: {value}")
    
    print("\n🔗 交叉连接:")
    for prop in cmap.find_cross_links():
        statement = prop.to_statement(cmap.concepts)
        print(f"  • {statement}")
    
    print("\n📁 导出到JSON...")
    cmap.export_to_json("microservices_concept_map.json")
    print("完成!")
```

### 3.2 概念图谱可视化

```python
import networkx as nx
import matplotlib.pyplot as plt
from matplotlib.patches import FancyBboxPatch, FancyArrowPatch
import numpy as np

class ConceptMapVisualizer:
    """概念图谱可视化工具"""
    
    def __init__(self, cmap: ConceptMap):
        self.cmap = cmap
        self.G = nx.DiGraph()
        self._build_graph()
    
    def _build_graph(self):
        """构建NetworkX图"""
        # 添加节点
        for concept in self.cmap.concepts.values():
            self.G.add_node(
                concept.id,
                label=concept.label,
                level=concept.level,
                category=concept.category
            )
        
        # 添加边
        for prop in self.cmap.propositions.values():
            self.G.add_edge(
                prop.source_id,
                prop.target_id,
                label=prop.linking_phrase,
                relation=prop.relation_type.name
            )
    
    def visualize(self, figsize=(16, 12), layout='hierarchical'):
        """
        可视化概念图谱
        
        参数:
            figsize: 图像大小
            layout: 布局算法 ('hierarchical', 'spring', 'kamada_kawai')
        """
        fig, ax = plt.subplots(figsize=figsize)
        
        # 计算布局
        if layout == 'hierarchical':
            pos = self._hierarchical_layout()
        elif layout == 'spring':
            pos = nx.spring_layout(self.G, k=2, iterations=50)
        else:
            pos = nx.kamada_kawai_layout(self.G)
        
        # 按层级着色
        level_colors = {
            0: '#FF6B6B',  # 红色 - 根概念
            1: '#4ECDC4',  # 青色 - 一级概念
            2: '#45B7D1',  # 蓝色 - 二级概念
            3: '#96CEB4',  # 绿色 - 三级概念
            4: '#FFEAA7',  # 黄色 - 四级概念
        }
        
        node_colors = []
        for node in self.G.nodes():
            level = self.G.nodes[node].get('level', 0)
            color = level_colors.get(level, '#DDA0DD')
            node_colors.append(color)
        
        # 绘制节点
        nx.draw_networkx_nodes(
            self.G, pos,
            node_color=node_colors,
            node_size=3000,
            alpha=0.9,
            ax=ax
        )
        
        # 绘制边
        # 区分层级边和交叉连接
        hierarchical_edges = []
        cross_edges = []
        
        for edge in self.G.edges():
            source_level = self.G.nodes[edge[0]].get('level', 0)
            target_level = self.G.nodes[edge[1]].get('level', 0)
            
            # 检查是否是向下连接（层级）或跨分支
            if target_level == source_level + 1:
                hierarchical_edges.append(edge)
            else:
                cross_edges.append(edge)
        
        # 绘制层级边（实线）
        nx.draw_networkx_edges(
            self.G, pos,
            edgelist=hierarchical_edges,
            edge_color='#666666',
            width=2,
            arrows=True,
            arrowsize=20,
            arrowstyle='->',
            connectionstyle='arc3,rad=0.1',
            ax=ax
        )
        
        # 绘制交叉边（虚线，不同颜色）
        if cross_edges:
            nx.draw_networkx_edges(
                self.G, pos,
                edgelist=cross_edges,
                edge_color='#E74C3C',
                width=1.5,
                style='dashed',
                arrows=True,
                arrowsize=15,
                arrowstyle='->',
                connectionstyle='arc3,rad=0.2',
                ax=ax
            )
        
        # 绘制节点标签
        labels = {node: self.G.nodes[node]['label'] for node in self.G.nodes()}
        nx.draw_networkx_labels(
            self.G, pos,
            labels=labels,
            font_size=9,
            font_weight='bold',
            ax=ax
        )
        
        # 绘制边标签（关系）
        edge_labels = {(u, v): d['label'] for u, v, d in self.G.edges(data=True)}
        nx.draw_networkx_edge_labels(
            self.G, pos,
            edge_labels=edge_labels,
            font_size=7,
            label_pos=0.5,
            ax=ax
        )
        
        ax.set_title(f"概念图谱: {self.cmap.title}", fontsize=16, fontweight='bold', pad=20)
        ax.axis('off')
        
        # 添加图例
        legend_elements = [
            plt.Line2D([0], [0], marker='o', color='w', markerfacecolor=color, 
                      markersize=12, label=f'层级 {level}')
            for level, color in level_colors.items() if level <= max(c.level for c in self.cmap.concepts.values())
        ]
        legend_elements.append(plt.Line2D([0], [0], color='#666666', linewidth=2, label='层级关系'))
        legend_elements.append(plt.Line2D([0], [0], color='#E74C3C', linewidth=1.5, linestyle='--', label='交叉连接'))
        
        ax.legend(handles=legend_elements, loc='upper left', bbox_to_anchor=(0, 1))
        
        plt.tight_layout()
        return fig, ax
    
    def _hierarchical_layout(self) -> dict:
        """计算层级布局"""
        pos = {}
        
        # 按层级分组节点
        level_nodes = {}
        for node in self.G.nodes():
            level = self.G.nodes[node].get('level', 0)
            if level not in level_nodes:
                level_nodes[level] = []
            level_nodes[level].append(node)
        
        # 垂直分层
        max_level = max(level_nodes.keys())
        for level, nodes in level_nodes.items():
            y = 1 - (level / max(max_level, 1))
            n = len(nodes)
            for i, node in enumerate(nodes):
                # 水平分布
                if n > 1:
                    x = (i / (n - 1)) if n > 1 else 0.5
                    # 居中调整
                    x = x * 0.8 + 0.1
                else:
                    x = 0.5
                pos[node] = (x, y)
        
        return pos
    
    def export_to_dot(self, filepath: str):
        """导出为Graphviz DOT格式"""
        lines = [f'digraph "{self.cmap.title}" {{']
        lines.append('  rankdir=TB;')
        lines.append('  node [shape=box, style="rounded,filled", fillcolor=lightblue];')
        lines.append('  edge [color="#666666"];')
        lines.append('')
        
        # 添加节点
        for concept in self.cmap.concepts.values():
            level_colors = {0: '#FF6B6B', 1: '#4ECDC4', 2: '#45B7D1', 3: '#96CEB4', 4: '#FFEAA7'}
            color = level_colors.get(concept.level, '#DDA0DD')
            lines.append(f'  "{concept.id}" [label="{concept.label}", fillcolor="{color}"];')
        
        lines.append('')
        
        # 添加边
        for prop in self.cmap.propositions.values():
            source = self.cmap.concepts.get(prop.source_id)
            target = self.cmap.concepts.get(prop.target_id)
            if source and target:
                lines.append(f'  "{prop.source_id}" -> "{prop.target_id}" '
                           f'[label="{prop.linking_phrase}"];')
        
        lines.append('}')
        
        with open(filepath, 'w', encoding='utf-8') as f:
            f.write('\n'.join(lines))
        
        print(f"已导出到: {filepath}")


# 使用示例
def demo_visualization():
    """演示概念图谱可视化"""
    cmap = create_microservices_concept_map()
    
    visualizer = ConceptMapVisualizer(cmap)
    fig, ax = visualizer.visualize(figsize=(18, 14), layout='hierarchical')
    
    plt.savefig('concept_map_visualization.png', dpi=150, bbox_inches='tight')
    plt.show()
    
    # 导出DOT格式
    visualizer.export_to_dot('microservices_concept_map.dot')


if __name__ == "__main__":
    demo_visualization()
```

### 3.3 概念图谱分析工具

```python
from collections import defaultdict, Counter
import pandas as pd

class ConceptMapAnalyzer:
    """概念图谱分析工具"""
    
    def __init__(self, cmap: ConceptMap):
        self.cmap = cmap
        self.G = nx.DiGraph()
        self._build_graph()
    
    def _build_graph(self):
        """构建图"""
        for concept in self.cmap.concepts.values():
            self.G.add_node(concept.id, **concept.__dict__)
        for prop in self.cmap.propositions.values():
            self.G.add_edge(prop.source_id, prop.target_id, **prop.__dict__)
    
    def analyze_centrality(self) -> pd.DataFrame:
        """分析概念中心性"""
        # 度中心性
        in_degree = dict(self.G.in_degree())
        out_degree = dict(self.G.out_degree())
        
        # 接近中心性（对于有向图）
        try:
            closeness = nx.closeness_centrality(self.G)
        except:
            closeness = {n: 0 for n in self.G.nodes()}
        
        # 介数中心性
        try:
            betweenness = nx.betweenness_centrality(self.G)
        except:
            betweenness = {n: 0 for n in self.G.nodes()}
        
        # 整合结果
        data = []
        for concept_id in self.G.nodes():
            concept = self.cmap.concepts.get(concept_id)
            data.append({
                'concept': concept.label if concept else concept_id,
                'in_degree': in_degree.get(concept_id, 0),
                'out_degree': out_degree.get(concept_id, 0),
                'closeness': round(closeness.get(concept_id, 0), 4),
                'betweenness': round(betweenness.get(concept_id, 0), 4),
                'total_degree': in_degree.get(concept_id, 0) + out_degree.get(concept_id, 0)
            })
        
        df = pd.DataFrame(data)
        return df.sort_values('betweenness', ascending=False)
    
    def detect_clusters(self) -> List[Set[str]]:
        """检测概念聚类（将无向图用于社区检测）"""
        undirected = self.G.to_undirected()
        
        try:
            from networkx.algorithms import community
            clusters = community.greedy_modularity_communities(undirected)
            return [set(c) for c in clusters]
        except:
            # 简单连通分量作为备选
            return list(nx.connected_components(undirected))
    
    def find_knowledge_gaps(self) -> List[Dict]:
        """识别潜在的知识缺口"""
        gaps = []
        
        # 1. 孤立节点（无连接的概念）
        for node in self.G.nodes():
            if self.G.degree(node) == 0:
                concept = self.cmap.concepts.get(node)
                gaps.append({
                    'type': '孤立概念',
                    'concept': concept.label if concept else node,
                    'suggestion': '考虑删除或建立与其他概念的关系'
                })
        
        # 2. 层级跳跃（跳过中间层级的连接）
        for edge in self.G.edges():
            source_level = self.G.nodes[edge[0]].get('level', 0)
            target_level = self.G.nodes[edge[1]].get('level', 0)
            
            if abs(target_level - source_level) > 2:
                source_concept = self.cmap.concepts.get(edge[0])
                target_concept = self.cmap.concepts.get(edge[1])
                gaps.append({
                    'type': '层级跳跃',
                    'concept': f"{source_concept.label} -> {target_concept.label}",
                    'suggestion': f'考虑添加层级{source_level+1}的中介概念'
                })
        
        # 3. 缺少交叉连接的不同分支
        # 简化：检查高度独立的概念组
        clusters = self.detect_clusters()
        if len(clusters) > 1:
            gaps.append({
                'type': '知识孤岛',
                'concept': f'检测到{len(clusters)}个独立的知识簇',
                'suggestion': '考虑添加不同簇之间的交叉连接'
            })
        
        return gaps
    
    def generate_study_guide(self) -> str:
        """基于概念图谱生成学习指南"""
        lines = [f"# {self.cmap.title} 学习指南", ""]
        lines.append(f"**焦点问题**: {self.cmap.focus_question}")
        lines.append("")
        
        # 按层级组织
        level_concepts = defaultdict(list)
        for concept in self.cmap.concepts.values():
            level_concepts[concept.level].append(concept)
        
        # 生成学习路径
        lines.append("## 学习路径")
        lines.append("")
        
        for level in sorted(level_concepts.keys()):
            lines.append(f"### 层级 {level}")
            lines.append("")
            
            for concept in level_concepts[level]:
                lines.append(f"#### {concept.label}")
                if concept.description:
                    lines.append(f"- **定义**: {concept.description}")
                
                # 相关关系
                parents = self.cmap.get_parents(concept.id)
                children = self.cmap.get_children(concept.id)
                
                if parents:
                    parent_names = [p.label for p in parents]
                    lines.append(f"- **上位概念**: {', '.join(parent_names)}")
                
                if children:
                    children_names = [c.label for c in children[:5]]  # 最多显示5个
                    lines.append(f"- **下位概念**: {', '.join(children_names)}")
                
                if concept.examples:
                    lines.append(f"- **示例**: {', '.join(concept.examples)}")
                
                lines.append("")
        
        # 交叉连接
        cross_links = self.cmap.find_cross_links()
        if cross_links:
            lines.append("## 重要关联")
            lines.append("")
            for prop in cross_links[:10]:  # 最多显示10个
                statement = prop.to_statement(self.cmap.concepts)
                lines.append(f"- {statement}")
            lines.append("")
        
        return '\n'.join(lines)


def analyze_concept_map(cmap: ConceptMap):
    """完整分析概念图谱"""
    analyzer = ConceptMapAnalyzer(cmap)
    
    print("=" * 60)
    print(f"概念图谱分析报告: {cmap.title}")
    print("=" * 60)
    
    # 中心性分析
    print("\n📊 概念中心性分析 (Top 10):")
    centrality = analyzer.analyze_centrality().head(10)
    print(centrality.to_string(index=False))
    
    # 知识缺口
    print("\n⚠️ 潜在知识缺口:")
    gaps = analyzer.find_knowledge_gaps()
    for gap in gaps:
        print(f"  [{gap['type']}] {gap['concept']}")
        print(f"    建议: {gap['suggestion']}")
    
    # 聚类分析
    print("\n🔍 知识聚类:")
    clusters = analyzer.detect_clusters()
    for i, cluster in enumerate(clusters, 1):
        concept_names = [cmap.concepts.get(n).label for n in cluster if n in cmap.concepts]
        print(f"  聚类 {i} ({len(cluster)} 个概念): {', '.join(concept_names[:5])}")
    
    # 生成学习指南
    guide = analyzer.generate_study_guide()
    with open(f"{cmap.title}_study_guide.md", 'w', encoding='utf-8') as f:
        f.write(guide)
    print(f"\n📄 学习指南已保存: {cmap.title}_study_guide.md")


if __name__ == "__main__":
    cmap = create_microservices_concept_map()
    analyze_concept_map(cmap)
```

---

## 4. 资源索引

### 4.1 软件工具

| 工具名称 | 类型 | 特点 | 适用平台 |
|----------|------|------|----------|
| **CmapTools** | 专业工具 | 学术研究标准，支持协作 | Windows/Mac/Linux |
| **XMind** | 思维导图 | 功能丰富，模板多样 | 跨平台 |
| **MindMeister** | 在线工具 | 实时协作，云同步 | Web |
| **Lucidchart** | 在线工具 | 专业图表，团队协作 | Web |
| **draw.io** | 开源免费 | 完全免费，功能强大 | Web/Desktop |
| **Obsidian** | 知识管理 | 双向链接+图谱视图 | 跨平台 |
| **yFiles** | 开发库 | 可编程，商业级 | Java/JS/.NET |

### 4.2 学习资源

**入门书籍**：
- 《Learning How to Learn》 - Barbara Oakley
- 《The Art of Changing the Brain》 - James E. Zull

**学术论文**：
- Novak, J. D. & Cañas, A. J. - *The Theory Underlying Concept Maps* (2006)
- Ausubel, D. P. - *Educational Psychology: A Cognitive View* (1968)

**在线资源**：
- Cmap官网 (cmap.ihmc.us) - 理论、工具和案例
- Florida Institute for Human & Machine Cognition - 研究论文

---

## 5. 关联知识

### 5.1 上游关联

```
C02_Concept_Mapping
├── C01_Spaced_Repetition
│   └── 概念图谱可作为间隔重复卡片的知识框架
├── A03_Design_Architecture/B05_System_Modeling
│   └── 系统建模方法与概念图谱的技术实现
└── A06_Technical_Intuition/B01_CS_Theories
    └── 认知科学理论支撑
```

### 5.2 下游应用

```
C02_Concept_Mapping
├── B02_Content_Strategy/C01_Atomic_Notes
│   └── 概念分解为原子笔记
├── C03_Knowledge_Graphs
│   └── 概念图谱的数据结构扩展
└── A99_Sandbox/B01_Sandbox_Drafts/C02_Architecture_Sketches
    └── 架构草图的概念组织
```

---

## 6. 学习建议

### 6.1 构建技巧

1. **从焦点问题开始**：每张概念图谱都应该围绕一个明确的问题构建
2. **遵循"由大到小"**：先确定最一般的概念，再逐步细化
3. **重视交叉连接**：不同分支间的连接往往体现深层理解
4. **迭代完善**：概念图谱是动态演化的，随学习深入持续更新

### 6.2 常见错误

| 错误类型 | 表现 | 解决方案 |
|----------|------|----------|
| 概念过于宽泛 | 一个概念包含太多内容 | 拆分为多个子概念 |
| 关系不明确 | 使用模糊连接词 | 使用精确的动词或短语 |
| 缺少层级 | 所有概念在同一层级 | 明确概念的抽象程度 |
| 过度连接 | 每个概念都连接到其他所有概念 | 只保留有意义的关系 |

### 6.3 实践路径

**第1周**：学习基础理论，用纸质工具练习绘制简单概念图谱
**第2-3周**：使用CmapTools或draw.io绘制技术主题的概念图谱
**第4周+**：将概念图谱融入日常工作流，与其他知识工具结合

---

*最后更新：2026-01-30*
*维护者：ZCO Knowledge Ops Team*
