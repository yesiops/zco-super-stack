# C03_Knowledge_Graphs - 知识图谱

> 基于图数据库的语义网络构建与知识推理技术

---

## 1. 主题定位

### 1.1 定义与演进

知识图谱（Knowledge Graph, KG）是一种用于描述现实世界实体及其关系的语义网络技术。它以图结构的形式组织知识，其中节点表示实体（Entities）或概念（Concepts），边表示实体间的关系（Relations）。知识图谱的核心目标是将分散的信息组织成机器可理解、可推理的结构化知识。

知识图谱的发展历程：

```
1960s: 语义网络(Semantic Networks) - Quillian
    ↓
1980s: 本体论(Ontology) - 哲学概念引入计算机科学
    ↓
2001: 语义网(Semantic Web) - Tim Berners-Lee提出
    ↓
2012: 知识图谱正式发布 - Google Knowledge Graph
    ↓
2014+: 大规模知识图谱构建 - Wikidata, DBpedia, CN-DBpedia
    ↓
2020+: 神经符号结合 - 知识图谱与深度学习融合
```

### 1.2 技术架构

```
┌─────────────────────────────────────────────────────────────┐
│                    知识图谱技术架构                           │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │  知识获取层   │  │  知识融合层   │  │  知识推理层   │       │
│  │              │  │              │  │              │       │
│  │ • 信息抽取   │  │ • 实体对齐   │  │ • 规则推理   │       │
│  │ • 关系抽取   │  │ • 知识合并   │  │ • 表示学习   │       │
│  │ • 事件抽取   │  │ • 质量评估   │  │ • 路径推理   │       │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘       │
│         │                 │                 │               │
│         └─────────────────┼─────────────────┘               │
│                           ▼                                 │
│                  ┌──────────────────┐                      │
│                  │    知识存储层     │                      │
│                  │                  │                      │
│                  │ • 图数据库(Neo4j)│                      │
│                  │ • RDF存储(Jena)  │                      │
│                  │ • 向量数据库     │                      │
│                  └────────┬─────────┘                      │
│                           ▼                                 │
│                  ┌──────────────────┐                      │
│                  │    知识应用层     │                      │
│                  │                  │                      │
│                  │ • 智能问答      │                      │
│                  │ • 推荐系统      │                      │
│                  │ • 语义搜索      │                      │
│                  └──────────────────┘                      │
└─────────────────────────────────────────────────────────────┘
```

### 1.3 应用场景

| 领域 | 应用 | 代表系统 |
|------|------|----------|
| **搜索引擎** | 语义理解、直接答案 | Google KG, Bing Satori |
| **智能问答** | 知识驱动的对话系统 | Watson, 小度 |
| **推荐系统** | 知识增强的推荐 | 淘宝商品推荐 |
| **金融风控** | 关系挖掘、反欺诈 | 企业关联图谱 |
| **医疗健康** | 药物发现、辅助诊断 | 医学知识图谱 |
| **智能制造** | 设备故障诊断 | 工业知识图谱 |

---

## 2. 核心概念

### 2.1 知识表示模型

#### 2.1.1 RDF数据模型

资源描述框架（Resource Description Framework）是W3C推荐的语义网基础标准：

```
RDF三元组：(Subject, Predicate, Object)
         （主语，    谓语，     宾语）
         （实体，    关系，     实体/值）

示例：
(人工智能, 是一种, 计算机科学分支)
(深度学习, 是, 人工智能的子领域)
(Geoffrey Hinton, 发明了, 反向传播算法)
(反向传播算法, 应用于, 神经网络训练)
```

RDF序列化格式：

```turtle
# Turtle格式 - 人类友好的RDF表示
@prefix ex: <http://example.org/> .
@prefix foaf: <http://xmlns.com/foaf/0.1/> .

ex:AI a ex:ComputerScienceBranch ;
    ex:hasSubfield ex:MachineLearning,
                   ex:ComputerVision,
                   ex:NLP .

ex:GeoffreyHinton a ex:Researcher ;
    foaf:name "Geoffrey Hinton" ;
    ex:invented ex:Backpropagation ;
    ex:receivedAward ex:TuringAward2018 .

ex:Backpropagation a ex:Algorithm ;
    ex:usedIn ex:NeuralNetworkTraining ;
    ex:yearInvented 1986 .
```

#### 2.1.2 本体论（Ontology）

本体论是对特定领域知识的形式化、结构化描述：

```
本体论组成要素：
├─ 类（Classes）
│   └─ 概念的集合，如：Person, Organization, Algorithm
├─ 属性（Properties）
│   ├─ 对象属性（Object Properties）：连接两个实体
│   │   └─ 如：worksFor, invented, locatedIn
│   └─ 数据属性（Data Properties）：连接实体与字面量
│       └─ 如：name, age, foundedYear
├─ 个体（Individuals）
│   └─ 类的实例，如：GeoffreyHinton, OpenAI
├─ 公理（Axioms）
│   ├─ 子类关系：Researcher ⊑ Person
│   ├─ 互斥关系：Person ⊓ Organization ≡ ⊥
│   └─ 属性约束：∀invented.Algorithm
└─ 规则（Rules）
    └─ 推理规则：if (x, worksFor, y) and (y, locatedIn, z) 
                then (x, worksInCountry, z)
```

OWL（Web Ontology Language）示例：

```xml
<!-- OWL本体定义示例 -->
<owl:Class rdf:about="#Researcher">
    <rdfs:subClassOf rdf:resource="#Person"/>
    <rdfs:comment>A person who conducts research</rdfs:comment>
</owl:Class>

<owl:ObjectProperty rdf:about="#invented">
    <rdfs:domain rdf:resource="#Researcher"/>
    <rdfs:range rdf:resource="#Algorithm"/>
    <owl:inverseOf rdf:resource="#inventedBy"/>
</owl:ObjectProperty>

<owl:DatatypeProperty rdf:about="#foundedYear">
    <rdfs:domain rdf:resource="#Organization"/>
    <rdfs:range rdf:resource="&xsd;integer"/>
</owl:DatatypeProperty>

<!-- 实例定义 -->
<Researcher rdf:about="#GeoffreyHinton">
    <name>Geoffrey Everest Hinton</name>
    <invented rdf:resource="#Backpropagation"/>
    <worksFor rdf:resource="#UniversityOfToronto"/>
</Researcher>
```

#### 2.1.3 属性图模型（Property Graph）

属性图是图数据库（如Neo4j）采用的数据模型：

```
属性图要素：
├─ 节点（Node）
│   ├─ 标签（Label）：实体的类型
│   ├─ 属性（Properties）：键值对形式的属性
│   └─ ID：唯一标识符
│
└─ 关系（Relationship）
    ├─ 类型（Type）：关系类型
    ├─ 方向（Direction）：有向边
    ├─ 属性（Properties）：关系也可有属性
    └─ 起止节点：关系的两端

示例：
(节点)                    (关系)
(:Person {               [:WORKS_FOR {
  name: "Geoffrey Hinton",  since: 2013,
  nationality: "UK",        role: "University Professor"
  born: 1947            }]→
})-[:WORKS_FOR {since: 2013}]→(:Organization {
                                   name: "University of Toronto",
                                   type: "University",
                                   founded: 1827
                                 })
```

### 2.2 知识图谱生命周期

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│ 知识获取  │───→│ 知识表示  │───→│ 知识融合  │───→│ 知识推理  │───→│ 知识应用  │
└──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘
     │               │               │               │               │
     ▼               ▼               ▼               ▼               ▼
 实体识别         RDF/图模型       实体对齐        规则推理        智能问答
 关系抽取         本体构建         冲突消解        表示学习        推荐系统
 事件抽取         知识存储         质量评估        路径推理        语义搜索
```

### 2.3 知识推理

#### 2.3.1 基于规则的推理

```
推理规则示例：

规则1：传递性
如果 (A, 位于, B) 且 (B, 位于, C)
那么 (A, 位于, C)

示例：(北京, 位于, 中国), (中国, 位于, 亚洲)
     → (北京, 位于, 亚洲)

规则2：对称性
如果 (A, 合作, B)
那么 (B, 合作, A)

规则3：逆关系
如果 (A, 是...的上级, B)
那么 (B, 是...的下级, A)

规则4：继承
如果 (A, 是, 程序员) 且 (程序员, 使用, Python)
那么 (A, 使用, Python)
```

#### 2.3.2 基于嵌入的推理

知识图谱嵌入将实体和关系映射到低维向量空间：

```python
# 知识图谱嵌入原理
"""
TransE模型（Bordes et al., 2013）:

基本假设: h + r ≈ t
- h: 头实体嵌入向量
- r: 关系嵌入向量  
- t: 尾实体嵌入向量

得分函数: f(h, r, t) = ||h + r - t||
分数越低表示三元组越可能成立

其他模型：
- TransH: 关系超平面投影
- TransR: 关系特定空间
- ComplEx: 复数向量空间
- RotatE: 旋转模型
"""

import torch
import torch.nn as nn

class TransE(nn.Module):
    """TransE知识图谱嵌入模型"""
    
    def __init__(self, num_entities, num_relations, embedding_dim):
        super().__init__()
        self.entity_embeddings = nn.Embedding(num_entities, embedding_dim)
        self.relation_embeddings = nn.Embedding(num_relations, embedding_dim)
        
        # 初始化
        nn.init.xavier_uniform_(self.entity_embeddings.weight)
        nn.init.xavier_uniform_(self.relation_embeddings.weight)
        
        # 归一化实体向量
        self.entity_embeddings.weight.data = \
            torch.nn.functional.normalize(self.entity_embeddings.weight.data, p=2, dim=1)
    
    def forward(self, heads, relations, tails):
        """
        参数:
            heads: 头实体索引 [batch_size]
            relations: 关系索引 [batch_size]
            tails: 尾实体索引 [batch_size]
        """
        h = self.entity_embeddings(heads)
        r = self.relation_embeddings(relations)
        t = self.entity_embeddings(tails)
        
        # 计算得分（距离）
        score = torch.norm(h + r - t, p=1, dim=1)
        return score
    
    def predict_tail(self, head, relation, k=10):
        """
        预测给定头实体和关系的最可能尾实体
        
        参数:
            head: 头实体索引
            relation: 关系索引
            k: 返回前k个最可能的实体
        """
        h = self.entity_embeddings(torch.tensor([head]))
        r = self.relation_embeddings(torch.tensor([relation]))
        
        # 计算所有实体作为尾实体的得分
        all_entities = self.entity_embeddings.weight
        scores = torch.norm(h + r - all_entities, p=1, dim=1)
        
        # 返回得分最低的k个实体索引
        top_k = torch.topk(scores, k, largest=False)
        return top_k.indices.tolist()
```

---

## 3. 技术实践

### 3.1 Neo4j图数据库实战

```python
from neo4j import GraphDatabase, basic_auth
from typing import List, Dict, Optional, Tuple
import json

class Neo4jKnowledgeGraph:
    """基于Neo4j的知识图谱管理类"""
    
    def __init__(self, uri: str, user: str, password: str):
        self.driver = GraphDatabase.driver(uri, auth=basic_auth(user, password))
    
    def close(self):
        """关闭数据库连接"""
        self.driver.close()
    
    def create_entity(self, label: str, properties: Dict) -> str:
        """
        创建实体节点
        
        参数:
            label: 实体类型标签
            properties: 实体属性字典
            
        返回:
            创建的节点ID
        """
        with self.driver.session() as session:
            # 生成唯一ID
            entity_id = properties.get('id', self._generate_id())
            properties['entity_id'] = entity_id
            
            query = f"""
            CREATE (n:{label} $properties)
            RETURN n.entity_id as id
            """
            result = session.run(query, properties=properties)
            return result.single()["id"]
    
    def create_relationship(self, from_id: str, to_id: str, 
                           rel_type: str, properties: Dict = None) -> bool:
        """
        创建实体间关系
        
        参数:
            from_id: 起始实体ID
            to_id: 目标实体ID
            rel_type: 关系类型
            properties: 关系属性
        """
        with self.driver.session() as session:
            query = f"""
            MATCH (from), (to)
            WHERE from.entity_id = $from_id AND to.entity_id = $to_id
            CREATE (from)-[r:{rel_type} $properties]->(to)
            RETURN r
            """
            result = session.run(query, 
                               from_id=from_id, 
                               to_id=to_id,
                               properties=properties or {})
            return result.single() is not None
    
    def find_entity(self, entity_id: str = None, label: str = None, 
                   **properties) -> List[Dict]:
        """
        查找实体
        
        参数:
            entity_id: 实体ID
            label: 类型标签
            **properties: 其他属性筛选
        """
        with self.driver.session() as session:
            where_clauses = []
            params = {}
            
            if entity_id:
                where_clauses.append("n.entity_id = $entity_id")
                params['entity_id'] = entity_id
            
            for key, value in properties.items():
                where_clauses.append(f"n.{key} = ${key}")
                params[key] = value
            
            where_str = " AND ".join(where_clauses) if where_clauses else "1=1"
            
            label_str = f":{label}" if label else ""
            
            query = f"""
            MATCH (n{label_str})
            WHERE {where_str}
            RETURN n
            """
            
            result = session.run(query, **params)
            return [dict(record["n"].items()) for record in result]
    
    def query_knowledge(self, cypher_query: str, **parameters) -> List[Dict]:
        """
        执行Cypher查询
        
        参数:
            cypher_query: Cypher查询语句
            **parameters: 查询参数
        """
        with self.driver.session() as session:
            result = session.run(cypher_query, **parameters)
            return [dict(record) for record in result]
    
    def find_path(self, start_id: str, end_id: str, 
                 max_depth: int = 5) -> List[Dict]:
        """
        查找两个实体间的路径
        
        参数:
            start_id: 起始实体ID
            end_id: 目标实体ID
            max_depth: 最大搜索深度
        """
        query = """
        MATCH path = shortestPath(
            (start)-[*1..$max_depth]->(end)
        )
        WHERE start.entity_id = $start_id AND end.entity_id = $end_id
        RETURN path
        """
        
        with self.driver.session() as session:
            result = session.run(query, 
                               start_id=start_id, 
                               end_id=end_id,
                               max_depth=max_depth)
            
            paths = []
            for record in result:
                path = record["path"]
                path_data = {
                    'length': len(path.relationships),
                    'nodes': [dict(node.items()) for node in path.nodes],
                    'relationships': [{
                        'type': rel.type,
                        'properties': dict(rel.items())
                    } for rel in path.relationships]
                }
                paths.append(path_data)
            return paths
    
    def get_neighbors(self, entity_id: str, 
                     relationship_type: str = None,
                     direction: str = 'both') -> List[Dict]:
        """
        获取实体的邻居节点
        
        参数:
            entity_id: 实体ID
            relationship_type: 关系类型筛选
            direction: 方向 ('out', 'in', 'both')
        """
        rel_filter = f":{relationship_type}" if relationship_type else ""
        
        if direction == 'out':
            pattern = f"(n)-[{rel_filter}]->(neighbor)"
        elif direction == 'in':
            pattern = f"(n)<-[{rel_filter}]-(neighbor)"
        else:
            pattern = f"(n)-[{rel_filter}]-(neighbor)"
        
        query = f"""
        MATCH {pattern}
        WHERE n.entity_id = $entity_id
        RETURN neighbor, type(rel) as relationship_type, 
               startNode(rel).entity_id = $entity_id as is_outgoing
        """
        
        with self.driver.session() as session:
            result = session.run(query, entity_id=entity_id)
            return [{
                'entity': dict(record["neighbor"].items()),
                'relationship': record["relationship_type"],
                'is_outgoing': record["is_outgoing"]
            } for record in result]
    
    def infer_by_transitivity(self, entity_id: str, 
                             relationship_type: str) -> List[Dict]:
        """
        基于传递性进行推理
        
        例如：如果 A-位于->B 且 B-位于->C，则 A-位于->C
        """
        query = f"""
        MATCH (start)-[:{relationship_type}*2..]->(end)
        WHERE start.entity_id = $entity_id
        AND NOT (start)-[:{relationship_type}]->(end)
        RETURN DISTINCT end, length(shortestPath(
            (start)-[:{relationship_type}*]->(end)
        )) as distance
        """
        
        with self.driver.session() as session:
            result = session.run(query, 
                               entity_id=entity_id,
                               relationship_type=relationship_type)
            return [{
                'entity': dict(record["end"].items()),
                'inferred_distance': record["distance"]
            } for record in result]
    
    def get_statistics(self) -> Dict:
        """获取知识图谱统计信息"""
        queries = {
            'entity_count': 'MATCH (n) RETURN count(n) as count',
            'relationship_count': 'MATCH ()-[r]->() RETURN count(r) as count',
            'entity_types': 'MATCH (n) RETURN labels(n)[0] as label, count(*) as count',
            'relationship_types': 'MATCH ()-[r]->() RETURN type(r) as type, count(*) as count'
        }
        
        stats = {}
        with self.driver.session() as session:
            for key, query in queries.items():
                result = session.run(query)
                if key in ['entity_types', 'relationship_types']:
                    stats[key] = {record[result.keys()[0]]: record["count"] 
                                 for record in result}
                else:
                    stats[key] = result.single()["count"]
        
        return stats
    
    def _generate_id(self) -> str:
        """生成唯一ID"""
        import uuid
        return str(uuid.uuid4())[:8]


# 知识图谱构建示例
def build_tech_knowledge_graph(kg: Neo4jKnowledgeGraph):
    """构建技术领域知识图谱示例"""
    
    # 创建技术领域实体
    entities = {
        'ai': kg.create_entity('Field', {
            'name': '人工智能',
            'english_name': 'Artificial Intelligence',
            'description': '研究、开发用于模拟、延伸和扩展人的智能的理论、方法、技术及应用系统',
            'established_year': 1956
        }),
        'ml': kg.create_entity('Field', {
            'name': '机器学习',
            'english_name': 'Machine Learning',
            'description': '人工智能的一个子领域，使计算机能够从数据中学习',
            'established_year': 1959
        }),
        'dl': kg.create_entity('Field', {
            'name': '深度学习',
            'english_name': 'Deep Learning',
            'description': '基于多层神经网络的机器学习方法',
            'established_year': 2006
        }),
        'nlp': kg.create_entity('Field', {
            'name': '自然语言处理',
            'english_name': 'Natural Language Processing',
            'description': '计算机科学、人工智能和语言学的交叉领域',
            'established_year': 1950
        }),
        'hinton': kg.create_entity('Researcher', {
            'name': 'Geoffrey Hinton',
            'nationality': 'British-Canadian',
            'birth_year': 1947,
            'institution': 'University of Toronto'
        }),
        'bengio': kg.create_entity('Researcher', {
            'name': 'Yoshua Bengio',
            'nationality': 'Canadian',
            'birth_year': 1964,
            'institution': 'University of Montreal'
        }),
        'lecun': kg.create_entity('Researcher', {
            'name': 'Yann LeCun',
            'nationality': 'French-American',
            'birth_year': 1960,
            'institution': 'New York University'
        }),
        'transformer': kg.create_entity('Model', {
            'name': 'Transformer',
            'type': 'Neural Network Architecture',
            'paper': 'Attention Is All You Need',
            'year': 2017,
            'institution': 'Google Brain'
        }),
        'gpt': kg.create_entity('Model', {
            'name': 'GPT',
            'full_name': 'Generative Pre-trained Transformer',
            'type': 'Language Model',
            'organization': 'OpenAI'
        }),
        'bert': kg.create_entity('Model', {
            'name': 'BERT',
            'full_name': 'Bidirectional Encoder Representations from Transformers',
            'type': 'Language Model',
            'organization': 'Google'
        })
    }
    
    # 创建关系
    relationships = [
        ('ml', 'ai', 'SUBFIELD_OF', {'relevance': 'core'}),
        ('dl', 'ml', 'SUBFIELD_OF', {'relevance': 'key'}),
        ('nlp', 'ai', 'SUBFIELD_OF', {'relevance': 'major'}),
        ('dl', 'nlp', 'APPLIED_TO', {'impact': 'revolutionary'}),
        ('hinton', 'dl', 'PIONEER_OF', {'contribution': 'foundational'}),
        ('bengio', 'dl', 'PIONEER_OF', {'contribution': 'foundational'}),
        ('lecun', 'dl', 'PIONEER_OF', {'contribution': 'foundational'}),
        ('hinton', 'bengio', 'COLLABORATES_WITH', {'duration_years': 20}),
        ('hinton', 'lecun', 'COLLABORATES_WITH', {'duration_years': 25}),
        ('bengio', 'lecun', 'COLLABORATES_WITH', {'duration_years': 20}),
        ('transformer', 'nlp', 'USED_IN', {'significance': 'breakthrough'}),
        ('transformer', 'dl', 'BASED_ON', {'component': 'attention'}),
        ('gpt', 'transformer', 'DERIVED_FROM', {'modification': 'decoder_only'}),
        ('bert', 'transformer', 'DERIVED_FROM', {'modification': 'bidirectional'}),
        ('gpt', 'nlp', 'APPLIED_TO', {'task': 'generation'}),
        ('bert', 'nlp', 'APPLIED_TO', {'task': 'understanding'})
    ]
    
    for from_key, to_key, rel_type, props in relationships:
        kg.create_relationship(entities[from_key], entities[to_key], rel_type, props)
    
    print("✅ 知识图谱构建完成！")
    return entities


# Cypher查询示例
SAMPLE_QUERIES = {
    'find_all_researchers': """
        MATCH (r:Researcher)
        RETURN r.name, r.institution, r.nationality
        ORDER BY r.birth_year
    """,
    
    'find_turing_award_winners': """
        MATCH (r:Researcher)-[:RECEIVED]->(a:Award {name: 'Turing Award'})
        RETURN r.name, a.year
        ORDER BY a.year
    """,
    
    'find_model_lineage': """
        MATCH path = (m:Model)-[:DERIVED_FROM*]->(ancestor)
        WHERE m.name = 'GPT-4'
        RETURN path
    """,
    
    'find_collaboration_network': """
        MATCH (r1:Researcher)-[:COLLABORATES_WITH]-(r2:Researcher)
        RETURN r1.name, r2.name
    """,
    
    'recommend_related_papers': """
        MATCH (p1:Paper)<-[:AUTHORED]-(a:Researcher)-[:AUTHORED]->(p2:Paper)
        WHERE p1.title = $paper_title AND p1 <> p2
        RETURN p2.title, count(*) as shared_authors
        ORDER BY shared_authors DESC
        LIMIT 5
    """
}
```

### 3.2 RDF与SPARQL

```python
from rdflib import Graph, Namespace, Literal, URIRef
from rdflib.namespace import RDF, RDFS, OWL

class RDFKnowledgeGraph:
    """基于RDF/SPARQL的知识图谱管理"""
    
    def __init__(self):
        self.g = Graph()
        
        # 定义命名空间
        self.EX = Namespace("http://example.org/kg/")
        self.SCHEMA = Namespace("http://schema.org/")
        self.WD = Namespace("http://www.wikidata.org/entity/")
        
        self.g.bind('ex', self.EX)
        self.g.bind('schema', self.SCHEMA)
        self.g.bind('wd', self.WD)
    
    def add_triple(self, subject, predicate, object_val):
        """添加三元组"""
        self.g.add((subject, predicate, object_val))
    
    def add_entity(self, entity_uri, entity_type, properties):
        """
        添加实体及其属性
        
        参数:
            entity_uri: 实体URI
            entity_type: 实体类型
            properties: 属性字典
        """
        entity = URIRef(entity_uri)
        
        # 添加类型
        self.add_triple(entity, RDF.type, entity_type)
        
        # 添加属性
        for prop, value in properties.items():
            prop_uri = self.EX[prop]
            if isinstance(value, (str, int, float, bool)):
                self.add_triple(entity, prop_uri, Literal(value))
            else:
                self.add_triple(entity, prop_uri, URIRef(value))
        
        return entity
    
    def add_relationship(self, subject_uri, predicate_name, object_uri):
        """添加实体间关系"""
        sub = URIRef(subject_uri)
        pred = self.EX[predicate_name]
        obj = URIRef(object_uri)
        self.add_triple(sub, pred, obj)
    
    def query(self, sparql_query):
        """
        执行SPARQL查询
        
        参数:
            sparql_query: SPARQL查询字符串
        """
        return self.g.query(sparql_query)
    
    def serialize(self, format='turtle'):
        """序列化为指定格式"""
        return self.g.serialize(format=format)
    
    def save(self, filepath, format='turtle'):
        """保存到文件"""
        self.g.serialize(destination=filepath, format=format)
    
    @classmethod
    def load(cls, filepath, format='turtle'):
        """从文件加载"""
        kg = cls()
        kg.g.parse(filepath, format=format)
        return kg


# SPARQL查询示例
SPARQL_EXAMPLES = {
    'select_all_entities': """
        SELECT ?entity ?type ?label
        WHERE {
            ?entity rdf:type ?type .
            OPTIONAL { ?entity rdfs:label ?label }
        }
        LIMIT 100
    """,
    
    'find_related_entities': """
        SELECT ?related ?relation
        WHERE {
            <http://example.org/kg/entity1> ?relation ?related .
            FILTER (?relation != rdf:type)
        }
    """,
    
    'find_by_property': """
        SELECT ?entity ?value
        WHERE {
            ?entity ex:propertyName ?value .
            FILTER (?value > 100)
        }
        ORDER BY DESC(?value)
    """,
    
    'path_query': """
        SELECT ?intermediate ?target
        WHERE {
            <http://example.org/kg/start> ex:rel1 ?intermediate .
            ?intermediate ex:rel2 ?target .
        }
    """,
    
    'construct_new_triples': """
        CONSTRUCT {
            ?person ex:collaboratesWith ?colleague .
        }
        WHERE {
            ?person ex:worksAt ?org .
            ?colleague ex:worksAt ?org .
            FILTER (?person != ?colleague)
        }
    """
}
```

### 3.3 知识抽取（NER与RE）

```python
import spacy
from transformers import AutoTokenizer, AutoModelForTokenClassification, pipeline
from typing import List, Dict, Tuple

class KnowledgeExtractor:
    """知识抽取工具：命名实体识别 + 关系抽取"""
    
    def __init__(self):
        # 加载spaCy模型
        self.nlp = spacy.load('en_core_web_sm')
        
        # 加载Hugging Face NER pipeline
        self.ner_pipeline = pipeline(
            "ner",
            model="dslim/bert-base-NER",
            tokenizer="dslim/bert-base-NER",
            aggregation_strategy="simple"
        )
    
    def extract_entities_spacy(self, text: str) -> List[Dict]:
        """使用spaCy抽取实体"""
        doc = self.nlp(text)
        entities = []
        
        for ent in doc.ents:
            entities.append({
                'text': ent.text,
                'label': ent.label_,
                'start': ent.start_char,
                'end': ent.end_char,
                'source': 'spacy'
            })
        
        return entities
    
    def extract_entities_bert(self, text: str) -> List[Dict]:
        """使用BERT抽取实体"""
        results = self.ner_pipeline(text)
        
        entities = []
        for result in results:
            entities.append({
                'text': result['word'],
                'label': result['entity_group'],
                'score': result['score'],
                'start': result['start'],
                'end': result['end'],
                'source': 'bert'
            })
        
        return entities
    
    def extract_relations(self, text: str, entities: List[Dict]) -> List[Dict]:
        """
        基于依存句法分析抽取关系
        
        这是一个简化实现，实际应用中可以使用训练好的关系抽取模型
        """
        doc = self.nlp(text)
        relations = []
        
        # 构建实体位置映射
        entity_spans = {}
        for i, ent in enumerate(entities):
            entity_spans[(ent['start'], ent['end'])] = i
        
        # 分析依存关系
        for token in doc:
            # 寻找连接两个实体的依存路径
            if token.dep_ in ['nsubj', 'dobj', 'pobj']:
                head = token.head
                
                # 查找token和head所属的实体
                subj_ent = None
                obj_ent = None
                
                for (start, end), idx in entity_spans.items():
                    if start <= token.idx < end:
                        subj_ent = entities[idx]
                    if start <= head.idx < end:
                        obj_ent = entities[idx]
                
                if subj_ent and obj_ent and subj_ent != obj_ent:
                    relations.append({
                        'subject': subj_ent['text'],
                        'predicate': head.text if token.dep_ == 'nsubj' else f"{head.text}_{token.text}",
                        'object': obj_ent['text'],
                        'type': token.dep_,
                        'confidence': 0.7  # 简化置信度
                    })
        
        return relations
    
    def extract_from_text(self, text: str) -> Dict:
        """
        从文本中抽取完整知识
        
        返回包含实体和关系的字典
        """
        # 抽取实体
        spacy_ents = self.extract_entities_spacy(text)
        bert_ents = self.extract_entities_bert(text)
        
        # 合并实体（简单去重）
        all_entities = spacy_ents + bert_ents
        
        # 基于位置去重
        unique_entities = {}
        for ent in all_entities:
            key = (ent['start'], ent['end'])
            if key not in unique_entities:
                unique_entities[key] = ent
        
        entities = list(unique_entities.values())
        
        # 抽取关系
        relations = self.extract_relations(text, entities)
        
        return {
            'text': text,
            'entities': entities,
            'relations': relations,
            'triples': [
                (r['subject'], r['predicate'], r['object'])
                for r in relations
            ]
        }


# 使用示例
EXAMPLE_TEXTS = [
    "Geoffrey Hinton, Yann LeCun and Yoshua Bengio won the Turing Award in 2019 for their work on deep learning.",
    "OpenAI was founded in 2015 by Sam Altman, Elon Musk, and others in San Francisco.",
    "Transformer architecture, introduced by Google researchers in 2017, revolutionized natural language processing."
]
```

---

## 4. 资源索引

### 4.1 开源知识图谱

| 名称 | 规模 | 领域 | 访问方式 |
|------|------|------|----------|
| **Wikidata** | 1亿+实体 | 通用 | SPARQL Endpoint |
| **DBpedia** | 460万+实体 | 通用 | SPARQL/Dump |
| **YAGO** | 1000万+实体 | 通用 | 下载/Dump |
| **Freebase** | 2400万+实体 | 通用 | 已归档 |
| **CN-DBpedia** | 千万级 | 中文通用 | API |
| **OpenKG** | 百万级 | 中文 | 门户/API |
| **Bio2RDF** | 数百万 | 生物医学 | SPARQL |
| **DrugBank** | 数万 | 药物 | API/下载 |

### 4.2 图数据库

| 数据库 | 模型 | 特点 | 适用场景 |
|--------|------|------|----------|
| **Neo4j** | 属性图 | 成熟、生态好、Cypher查询 | 企业级应用 |
| **JanusGraph** | 属性图 | 分布式、开源 | 大规模图 |
| **ArangoDB** | 多模型 | 文档+图+KV | 灵活场景 |
| **Dgraph** | 原生分布式 | 水平扩展、GraphQL+- | 海量数据 |
| **Amazon Neptune** | 属性图+RDF | 托管服务、AWS生态 | 云原生 |
| **TigerGraph** | 原生并行 | 高性能分析 | 实时分析 |
| **Apache Jena** | RDF | 语义网标准 | 学术研究 |
| **Virtuoso** | 多模型 | SPARQL端点 | 知识库托管 |

### 4.3 工具与库

**Python生态**：
- `rdflib`: RDF处理
- `py2neo`: Neo4j Python驱动
- `SPARQLWrapper`: SPARQL查询
- `networkx`: 图分析
- `graphene`: GraphQL for Python

**知识抽取**：
- `spaCy`: 工业级NLP
- `Hugging Face Transformers`: 预训练模型
- `OpenNRE`: 开源关系抽取
- `DeepKE`: 深度知识抽取

---

## 5. 关联知识

```
C03_Knowledge_Graphs
├── C02_Concept_Mapping
│   └── 概念图谱是知识图谱的认知基础
├── A04_Data_AI/B01_Machine_Learning/C03_Deep_Learning
│   └── 图神经网络(GNN)用于知识图谱推理
├── A02_Programming_Languages/B02_Java/C03_Spring_Data
│   └── Spring Data Neo4j用于图数据库访问
└── A03_Design_Architecture/B03_Data_Storage/C03_Vector_Databases
    └── 向量数据库存储知识图谱嵌入
```

---

## 6. 学习建议

### 6.1 学习路径

**第1-2周**：掌握RDF/SPARQL基础，理解三元组模型
**第3-4周**：学习Neo4j图数据库，掌握Cypher查询
**第5-6周**：研究本体工程，学习OWL和推理规则
**第7-8周**：实践知识抽取，使用NLP工具从文本构建KG
**第9周+**：研究知识图谱嵌入和神经网络方法

### 6.2 实践项目

1. **个人技术知识库**：构建自己学习领域的小型知识图谱
2. **问答系统**：基于知识图谱实现简单问答
3. **推荐引擎**：使用知识图谱增强的推荐系统
4. **可视化工具**：开发知识图谱可视化界面

---

*最后更新：2026-01-30*
*维护者：ZCO Knowledge Ops Team*
