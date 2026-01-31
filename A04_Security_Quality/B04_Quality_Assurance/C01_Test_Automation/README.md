# C01 测试自动化 (Test Automation)

## 1. 主题定位

### 1.1 定义
测试自动化是利用软件工具自动执行测试、比较实际结果与预期结果、并生成测试报告的过程。它是现代软件工程的核心实践，能够显著提高测试效率、覆盖率和可靠性。

### 1.2 核心价值
- **效率提升**: 快速执行大量测试用例
- **覆盖增加**: 支持更全面的回归测试
- **质量保障**: 持续验证软件质量
- **快速反馈**: 及时发现问题
- **成本降低**: 长期减少人工测试成本

### 1.3 测试金字塔

```
        /\
       /  \     E2E 测试 (少量)
      /----\     端到端、UI 测试
     /      \    
    /--------\   集成测试 (中等)
   /          \  API、服务间集成
  /------------\
 /              \ 单元测试 (大量)
/----------------\ 函数、方法级别
```

---

## 2. 核心概念

### 2.1 测试类型

| 测试类型 | 范围 | 工具示例 | 执行频率 |
|----------|------|----------|----------|
| **单元测试** | 单个函数/方法 | pytest, Jest, JUnit | 每次提交 |
| **集成测试** | 模块间交互 | pytest, TestContainers | 每次构建 |
| **契约测试** | API 接口 | Pact, Spring Cloud Contract | 每日 |
| **E2E 测试** | 完整业务流程 | Cypress, Selenium, Playwright | 每日 |
| **性能测试** | 系统性能 | k6, JMeter, Locust | 每周 |
| **安全测试** | 安全漏洞 | OWASP ZAP, Burp Suite | 每周 |

### 2.2 单元测试最佳实践

```python
#!/usr/bin/env python3
"""
单元测试最佳实践示例
使用 pytest 框架
"""

import pytest
from typing import List, Optional
from dataclasses import dataclass
from unittest.mock import Mock, patch, MagicMock


@dataclass
class User:
    """用户领域模型"""
    id: int
    username: str
    email: str
    is_active: bool = True
    
    def validate_email(self) -> bool:
        """验证邮箱格式"""
        import re
        pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        return bool(re.match(pattern, self.email))
    
    def deactivate(self):
        """停用用户"""
        self.is_active = False


class UserRepository:
    """用户仓储"""
    
    def __init__(self, db_session):
        self.db = db_session
    
    def find_by_id(self, user_id: int) -> Optional[User]:
        """根据 ID 查找用户"""
        # 实际实现会查询数据库
        pass
    
    def save(self, user: User) -> User:
        """保存用户"""
        # 实际实现会写入数据库
        pass
    
    def find_active_users(self) -> List[User]:
        """查找所有活跃用户"""
        pass


class UserService:
    """用户服务"""
    
    def __init__(self, repository: UserRepository):
        self.repo = repository
    
    def get_user(self, user_id: int) -> User:
        """获取用户"""
        user = self.repo.find_by_id(user_id)
        if not user:
            raise ValueError(f"User not found: {user_id}")
        return user
    
    def create_user(self, username: str, email: str) -> User:
        """创建用户"""
        if not username or not email:
            raise ValueError("Username and email are required")
        
        user = User(id=0, username=username, email=email)
        if not user.validate_email():
            raise ValueError("Invalid email format")
        
        return self.repo.save(user)
    
    def deactivate_user(self, user_id: int) -> User:
        """停用用户"""
        user = self.get_user(user_id)
        user.deactivate()
        return self.repo.save(user)


# ==================== 测试代码 ====================

class TestUser:
    """User 模型测试"""
    
    def test_validate_email_valid(self):
        """测试有效的邮箱格式"""
        user = User(id=1, username="test", email="test@example.com")
        assert user.validate_email() is True
    
    def test_validate_email_invalid(self):
        """测试无效的邮箱格式"""
        user = User(id=1, username="test", email="invalid-email")
        assert user.validate_email() is False
    
    def test_validate_email_empty(self):
        """测试空邮箱"""
        user = User(id=1, username="test", email="")
        assert user.validate_email() is False
    
    def test_deactivate(self):
        """测试停用用户"""
        user = User(id=1, username="test", email="test@example.com", is_active=True)
        user.deactivate()
        assert user.is_active is False


class TestUserService:
    """UserService 测试 - 使用 Mock"""
    
    @pytest.fixture
    def mock_repo(self):
        """创建 Mock Repository"""
        return Mock(spec=UserRepository)
    
    @pytest.fixture
    def user_service(self, mock_repo):
        """创建 UserService 实例"""
        return UserService(mock_repo)
    
    def test_get_user_success(self, user_service, mock_repo):
        """测试成功获取用户"""
        # Arrange
        expected_user = User(id=1, username="test", email="test@example.com")
        mock_repo.find_by_id.return_value = expected_user
        
        # Act
        result = user_service.get_user(1)
        
        # Assert
        assert result == expected_user
        mock_repo.find_by_id.assert_called_once_with(1)
    
    def test_get_user_not_found(self, user_service, mock_repo):
        """测试用户不存在"""
        # Arrange
        mock_repo.find_by_id.return_value = None
        
        # Act & Assert
        with pytest.raises(ValueError, match="User not found: 999"):
            user_service.get_user(999)
    
    def test_create_user_success(self, user_service, mock_repo):
        """测试成功创建用户"""
        # Arrange
        saved_user = User(id=1, username="newuser", email="new@example.com")
        mock_repo.save.return_value = saved_user
        
        # Act
        result = user_service.create_user("newuser", "new@example.com")
        
        # Assert
        assert result == saved_user
        mock_repo.save.assert_called_once()
    
    def test_create_user_invalid_email(self, user_service, mock_repo):
        """测试创建用户时邮箱无效"""
        with pytest.raises(ValueError, match="Invalid email format"):
            user_service.create_user("newuser", "invalid-email")
        
        mock_repo.save.assert_not_called()
    
    def test_create_user_empty_username(self, user_service, mock_repo):
        """测试创建用户时用户名为空"""
        with pytest.raises(ValueError, match="Username and email are required"):
            user_service.create_user("", "test@example.com")


# 参数化测试
@pytest.mark.parametrize("email,expected", [
    ("test@example.com", True),
    ("user.name@domain.co.uk", True),
    ("invalid-email", False),
    ("@example.com", False),
    ("test@", False),
    ("", False),
])
def test_email_validation_parametrized(email, expected):
    """参数化测试邮箱验证"""
    user = User(id=1, username="test", email=email)
    assert user.validate_email() == expected


# Fixture 示例
@pytest.fixture(scope="function")
def sample_user():
    """提供测试用的用户 fixture"""
    return User(id=1, username="testuser", email="test@example.com")


@pytest.fixture(scope="module")
def db_connection():
    """模块级别的数据库连接 fixture"""
    # Setup
    connection = "database_connection"
    yield connection
    # Teardown
    connection = None


# 使用 skip 和 xfail
@pytest.mark.skip(reason="功能尚未实现")
def test_feature_not_implemented():
    """跳过未实现的功能测试"""
    pass


@pytest.mark.xfail(reason="已知缺陷 #123")
def test_known_bug():
    """标记预期失败的测试"""
    assert 1 == 2  # 这个会失败，但测试套件不会报错


# 集成测试示例
@pytest.mark.integration
class TestUserRepositoryIntegration:
    """集成测试 - 需要真实数据库"""
    
    def test_save_and_find_user(self, db_connection):
        """测试保存和查找用户"""
        # 这需要真实的数据库连接
        pass
```

### 2.2 API 测试自动化

```python
#!/usr/bin/env python3
"""
API 测试自动化示例
使用 requests 和 pytest
"""

import pytest
import requests
from typing import Dict, Any


class APITestClient:
    """API 测试客户端"""
    
    def __init__(self, base_url: str, api_key: str = None):
        self.base_url = base_url.rstrip('/')
        self.session = requests.Session()
        if api_key:
            self.session.headers['Authorization'] = f'Bearer {api_key}'
        self.session.headers['Content-Type'] = 'application/json'
    
    def get(self, endpoint: str, params: Dict = None) -> requests.Response:
        """GET 请求"""
        url = f"{self.base_url}{endpoint}"
        return self.session.get(url, params=params)
    
    def post(self, endpoint: str, data: Dict = None) -> requests.Response:
        """POST 请求"""
        url = f"{self.base_url}{endpoint}"
        return self.session.post(url, json=data)
    
    def put(self, endpoint: str, data: Dict = None) -> requests.Response:
        """PUT 请求"""
        url = f"{self.base_url}{endpoint}"
        return self.session.put(url, json=data)
    
    def delete(self, endpoint: str) -> requests.Response:
        """DELETE 请求"""
        url = f"{self.base_url}{endpoint}"
        return self.session.delete(url)


# 测试基类
class BaseAPITest:
    """API 测试基类"""
    
    @pytest.fixture(scope="class")
    def api_client(self):
        """提供 API 客户端"""
        return APITestClient(
            base_url="https://api.example.com/v1",
            api_key="test-api-key"
        )


class TestUserAPI(BaseAPITest):
    """用户 API 测试"""
    
    def test_get_users_list(self, api_client):
        """测试获取用户列表"""
        response = api_client.get("/users")
        
        assert response.status_code == 200
        data = response.json()
        assert isinstance(data, list)
        assert len(data) >= 0
    
    def test_get_user_by_id(self, api_client):
        """测试获取单个用户"""
        response = api_client.get("/users/1")
        
        assert response.status_code == 200
        data = response.json()
        assert data['id'] == 1
        assert 'username' in data
        assert 'email' in data
    
    def test_get_user_not_found(self, api_client):
        """测试获取不存在的用户"""
        response = api_client.get("/users/99999")
        
        assert response.status_code == 404
        data = response.json()
        assert 'error' in data or 'message' in data
    
    def test_create_user(self, api_client):
        """测试创建用户"""
        new_user = {
            "username": "testuser",
            "email": "test@example.com",
            "password": "securepassword123"
        }
        
        response = api_client.post("/users", data=new_user)
        
        assert response.status_code == 201
        data = response.json()
        assert data['username'] == new_user['username']
        assert data['email'] == new_user['email']
        assert 'id' in data
        assert 'password' not in data  # 密码不应返回
    
    def test_create_user_validation_error(self, api_client):
        """测试创建用户时的验证错误"""
        invalid_user = {
            "username": "",  # 空用户名
            "email": "invalid-email"
        }
        
        response = api_client.post("/users", data=invalid_user)
        
        assert response.status_code == 400
        data = response.json()
        assert 'errors' in data or 'message' in data
    
    def test_update_user(self, api_client):
        """测试更新用户"""
        update_data = {
            "email": "updated@example.com"
        }
        
        response = api_client.put("/users/1", data=update_data)
        
        assert response.status_code == 200
        data = response.json()
        assert data['email'] == update_data['email']
    
    def test_delete_user(self, api_client):
        """测试删除用户"""
        response = api_client.delete("/users/1")
        
        assert response.status_code == 204
    
    def test_api_response_time(self, api_client):
        """测试 API 响应时间"""
        import time
        
        start = time.time()
        response = api_client.get("/users")
        elapsed = time.time() - start
        
        assert response.status_code == 200
        assert elapsed < 1.0, f"API 响应过慢: {elapsed:.2f}s"


# 契约测试示例 (Pact 风格)
class ContractTest:
    """契约测试基类"""
    
    def test_user_service_contract(self):
        """用户服务契约测试"""
        expected_contract = {
            "consumer": "frontend-app",
            "provider": "user-service",
            "interactions": [
                {
                    "description": "获取用户信息",
                    "request": {
                        "method": "GET",
                        "path": "/users/1"
                    },
                    "response": {
                        "status": 200,
                        "headers": {
                            "Content-Type": "application/json"
                        },
                        "body": {
                            "id": 1,
                            "username": "string",
                            "email": "string",
                            "created_at": "string"
                        }
                    }
                }
            ]
        }
        
        # 验证实际响应符合契约
        pass
```

### 2.3 E2E 测试

```python
#!/usr/bin/env python3
"""
端到端测试示例
使用 Playwright
"""

import pytest
from playwright.sync_api import Page, expect


class TestUserWorkflow:
    """用户工作流 E2E 测试"""
    
    def test_user_registration_flow(self, page: Page):
        """测试用户注册流程"""
        # 访问注册页面
        page.goto("https://example.com/register")
        
        # 填写注册表单
        page.fill("[data-testid='username-input']", "testuser123")
        page.fill("[data-testid='email-input']", "test@example.com")
        page.fill("[data-testid='password-input']", "SecurePass123!")
        page.fill("[data-testid='confirm-password-input']", "SecurePass123!")
        
        # 提交表单
        page.click("[data-testid='register-button']")
        
        # 验证成功跳转
        expect(page).to_have_url("https://example.com/dashboard")
        expect(page.locator("[data-testid='welcome-message']")).to_contain_text("欢迎")
    
    def test_login_flow(self, page: Page):
        """测试登录流程"""
        page.goto("https://example.com/login")
        
        page.fill("[data-testid='username-input']", "testuser")
        page.fill("[data-testid='password-input']", "password")
        page.click("[data-testid='login-button']")
        
        # 验证登录成功
        expect(page.locator("[data-testid='user-menu']")).to_be_visible()
    
    def test_product_purchase_flow(self, page: Page):
        """测试商品购买流程"""
        # 登录
        page.goto("https://example.com/login")
        page.fill("[data-testid='username-input']", "testuser")
        page.fill("[data-testid='password-input']", "password")
        page.click("[data-testid='login-button']")
        
        # 浏览商品
        page.goto("https://example.com/products")
        page.click("[data-testid='product-1']")
        
        # 添加到购物车
        page.click("[data-testid='add-to-cart-button']")
        expect(page.locator("[data-testid='cart-count']")).to_have_text("1")
        
        # 结算
        page.click("[data-testid='cart-icon']")
        page.click("[data-testid='checkout-button']")
        
        # 填写地址
        page.fill("[data-testid='address-input']", "测试地址")
        page.click("[data-testid='continue-button']")
        
        # 选择支付方式
        page.click("[data-testid='payment-method-card']")
        page.fill("[data-testid='card-number']", "4111111111111111")
        page.fill("[data-testid='expiry-date']", "12/25")
        page.fill("[data-testid='cvv']", "123")
        
        # 确认支付
        page.click("[data-testid='place-order-button']")
        
        # 验证订单成功
        expect(page.locator("[data-testid='order-success']")).to_be_visible()
        expect(page.locator("[data-testid='order-number']")).to_be_visible()


# 视觉回归测试
class TestVisualRegression:
    """视觉回归测试"""
    
    def test_homepage_visual(self, page: Page):
        """测试首页视觉"""
        page.goto("https://example.com")
        page.wait_for_load_state("networkidle")
        
        # 截图并与基线对比
        expect(page).to_have_screenshot("homepage.png")
    
    def test_responsive_design(self, page: Page):
        """测试响应式设计"""
        # 桌面端
        page.set_viewport_size({"width": 1920, "height": 1080})
        page.goto("https://example.com")
        expect(page.locator("[data-testid='desktop-nav']")).to_be_visible()
        
        # 平板端
        page.set_viewport_size({"width": 768, "height": 1024})
        page.reload()
        expect(page.locator("[data-testid='tablet-nav']")).to_be_visible()
        
        # 手机端
        page.set_viewport_size({"width": 375, "height": 667})
        page.reload()
        expect(page.locator("[data-testid='mobile-nav']")).to_be_visible()
```

---

## 3. 技术实践

### 3.1 CI/CD 集成

```yaml
# .github/workflows/test.yml
name: Test Automation

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: test_db
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
      
      redis:
        image: redis:7
        ports:
          - 6379:6379
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.11'
    
    - name: Install dependencies
      run: |
        pip install -r requirements.txt
        pip install pytest pytest-cov pytest-xdist
    
    - name: Run unit tests
      run: |
        pytest tests/unit -v --cov=src --cov-report=xml -n auto
    
    - name: Run integration tests
      run: |
        pytest tests/integration -v
      env:
        DATABASE_URL: postgresql://test:test@localhost:5432/test_db
        REDIS_URL: redis://localhost:6379
    
    - name: Upload coverage
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage.xml
    
    - name: Run E2E tests
      run: |
        pytest tests/e2e -v
```

### 3.2 测试数据管理

```python
#!/usr/bin/env python3
"""
测试数据管理
使用 Factory Boy 模式
"""

import factory
from factory import Faker
from datetime import datetime


class UserFactory(factory.Factory):
    """用户工厂"""
    
    class Meta:
        model = dict  # 或使用实际模型
    
    id = factory.Sequence(lambda n: n)
    username = factory.LazyAttribute(lambda obj: f"user_{obj.id}")
    email = factory.LazyAttribute(lambda obj: f"{obj.username}@example.com")
    is_active = True
    created_at = factory.LazyFunction(datetime.now)


class OrderFactory(factory.Factory):
    """订单工厂"""
    
    class Meta:
        model = dict
    
    id = factory.Sequence(lambda n: n)
    user_id = factory.SubFactory(UserFactory)
    total_amount = factory.Faker('pydecimal', left_digits=5, right_digits=2, positive=True)
    status = factory.Iterator(['pending', 'paid', 'shipped', 'delivered'])
    created_at = factory.LazyFunction(datetime.now)


# 使用示例
def test_with_factories():
    """使用工厂创建测试数据"""
    # 创建单个用户
    user = UserFactory()
    print(f"Created user: {user}")
    
    # 创建批量用户
    users = UserFactory.build_batch(10)
    print(f"Created {len(users)} users")
    
    # 创建订单
    order = OrderFactory(user_id=user['id'])
    print(f"Created order: {order}")
```

---

## 4. 资源索引

### 4.1 测试框架

| 语言 | 单元测试 | E2E 测试 | 性能测试 |
|------|----------|----------|----------|
| Python | pytest | Playwright | Locust |
| JavaScript | Jest/Vitest | Cypress/Playwright | k6 |
| Java | JUnit/TestNG | Selenium | JMeter |
| Go | testing + testify | Playwright | k6 |

### 4.2 工具推荐

| 工具 | 用途 | 链接 |
|------|------|------|
| pytest | Python 测试框架 | https://docs.pytest.org/ |
| Playwright | 现代 E2E 测试 | https://playwright.dev/ |
| TestContainers | 集成测试容器 | https://www.testcontainers.org/ |
| Pact | 契约测试 | https://pact.io/ |
| Allure | 测试报告 | https://qameta.io/allure-report/ |

---

## 5. 关联知识

```
A04_Security_Quality/B04_Quality_Assurance/
├── C01_Test_Automation (本模块)
│   ├── → C02_Fuzzing_Techniques: 模糊测试
│   └── → C03_Compliance_Scans: 合规扫描
│
├── → B03_Maintenance_Ops: 测试维护
└── → A03_Software_Engineering: 测试驱动开发
```

---

*最后更新: 2024-01-30*
