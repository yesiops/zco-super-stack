# C01_Code_Craftsmanship - 代码匠艺

## 1. 主题定位

### 1.1 定义与范围

代码匠艺（Code Craftsmanship）是将软件开发视为一门手艺，追求代码质量、可维护性和优雅性的工程实践哲学。本知识单元涵盖软件设计原则、代码整洁实践、重构技术、测试驱动开发等核心内容。

### 1.2 业务价值

- 降低维护成本
- 减少缺陷率
- 提升开发效率
- 促进团队协作
- 技术债务控制

### 1.3 适用场景

| 场景 | 适用性 |
|-----|-------|
| 长期维护项目 | ⭐⭐⭐⭐⭐ |
| 团队协作开发 | ⭐⭐⭐⭐⭐ |
| 复杂业务系统 | ⭐⭐⭐⭐⭐ |
| 快速原型 | ⭐⭐⭐ |

## 2. 核心概念

### 2.1 SOLID原则

```python
# S: 单一职责原则
class UserRepository:
    def save(self, user): pass

class UserService:
    def register(self, user_data): pass

# O: 开闭原则
class PaymentProcessor:
    def process(self, payment_method, amount):
        return payment_method.pay(amount)

# L: 里氏替换原则
class Bird:
    def move(self): pass

class FlyingBird(Bird):
    def fly(self): pass

# I: 接口隔离原则
class Workable:
    def work(self): pass

class Eatable:
    def eat(self): pass

# D: 依赖倒置原则
class Database:
    def query(self, sql): pass

class UserService:
    def __init__(self, db: Database):
        self.db = db
```

### 2.2 代码整洁实践

- 有意义的命名
- 小函数
- 单一职责
- 避免重复
- 清晰注释

## 3. 技术实践

### 3.1 代码重构示例

```python
# 重构前：坏代码
class OrderBad:
    def process(self, order):
        total = 0
        for item in order.items:
            total += item.price * item.qty
        if order.customer.vip:
            total *= 0.9
        return total

# 重构后：好代码
class Order:
    def __init__(self, items, customer):
        self.items = items
        self.customer = customer
    
    def calculate_total(self):
        subtotal = sum(item.total for item in self.items)
        return self.customer.apply_discount(subtotal)

class OrderItem:
    def __init__(self, price, qty):
        self.price = price
        self.qty = qty
    
    @property
    def total(self):
        return self.price * self.qty

class Customer:
    def __init__(self, vip=False):
        self.vip = vip
    
    def apply_discount(self, amount):
        return amount * 0.9 if self.vip else amount
```

### 3.2 测试驱动开发

```python
import unittest

class TestCalculator(unittest.TestCase):
    def test_add(self):
        calc = Calculator()
        self.assertEqual(calc.add(2, 3), 5)
    
    def test_divide_by_zero(self):
        calc = Calculator()
        with self.assertRaises(ValueError):
            calc.divide(1, 0)

class Calculator:
    def add(self, a, b):
        return a + b
    
    def divide(self, a, b):
        if b == 0:
            raise ValueError("Cannot divide by zero")
        return a / b
```

## 4. 资源索引

| 资源 | 链接 |
|-----|------|
| Clean Code | https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882 |
| Refactoring | https://refactoring.com |
| SOLID原则 | https://www.digitalocean.com/community/conceptual_articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design |

## 5. 关联知识

- B02_Automation_Strategy
- B03_Documentation_Ops
- A03_Core_Technologies

## 6. 学习建议

1. 阅读《代码整洁之道》
2. 练习重构 kata
3. 参与代码审查
4. 使用静态分析工具

---
*最后更新：2024年*
