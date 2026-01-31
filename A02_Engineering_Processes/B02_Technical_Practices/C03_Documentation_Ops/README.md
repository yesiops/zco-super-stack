# C03_Documentation_Ops - 文档运维

## 1. 主题定位

### 1.1 定义与范围

文档运维（Documentation Ops，简称DocsOps）是将文档视为软件产品的一部分，通过版本控制、自动化构建、持续发布等技术手段管理技术文档的生命周期。本知识单元涵盖文档即代码、API文档自动化、知识库管理、文档质量保障等内容。

### 1.2 业务价值

- 降低知识传递成本
- 提升团队协作效率
- 加速新人上手
- 保障业务连续性
- 提升产品质量

### 1.3 适用场景

| 场景 | 适用性 |
|-----|-------|
| API产品 | ⭐⭐⭐⭐⭐ |
| 开源项目 | ⭐⭐⭐⭐⭐ |
| 复杂系统 | ⭐⭐⭐⭐⭐ |
| 合规要求 | ⭐⭐⭐⭐⭐ |

## 2. 核心概念

### 2.1 文档即代码

```
文档即代码工作流：

Markdown/Markup ──► Git仓库 ──► CI构建 ──► 自动发布
     │                │           │           │
     ▼                ▼           ▼           ▼
  本地编写        版本控制      格式检查    在线站点
  实时预览        分支管理      链接校验    搜索索引
```

### 2.2 文档类型矩阵

| 类型 | 受众 | 工具 | 更新频率 |
|-----|-----|-----|---------|
| API文档 | 开发者 | OpenAPI/Swagger | 每次发布 |
| 架构文档 | 架构师 | Markdown/PlantUML | 架构变更 |
| 运维手册 | SRE | Markdown/Runbooks | 按需更新 |
| 用户指南 | 最终用户 | Docusaurus | 版本发布 |

## 3. 技术实践

### 3.1 API文档自动化

```python
# FastAPI自动API文档
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI(
    title="用户管理API",
    description="用户注册、认证、管理接口",
    version="1.0.0"
)

class UserCreate(BaseModel):
    """用户创建模型"""
    username: str
    email: str
    password: str

class UserResponse(BaseModel):
    """用户响应模型"""
    id: int
    username: str
    email: str

@app.post("/users", response_model=UserResponse, tags=["用户"])
def create_user(user: UserCreate):
    """
    创建新用户
    
    - **username**: 用户名，3-20字符
    - **email**: 有效邮箱地址
    - **password**: 密码，至少8位
    """
    return {"id": 1, "username": user.username, "email": user.email}

@app.get("/users/{user_id}", response_model=UserResponse, tags=["用户"])
def get_user(user_id: int):
    """根据ID获取用户信息"""
    return {"id": user_id, "username": "test", "email": "test@example.com"}
```

### 3.2 Docusaurus文档站点

```javascript
// docusaurus.config.js
module.exports = {
  title: '技术文档中心',
  tagline: '产品技术文档',
  url: 'https://docs.example.com',
  baseUrl: '/',
  
  themeConfig: {
    navbar: {
      title: '技术文档',
      items: [
        {to: 'docs/intro', label: '文档', position: 'left'},
        {to: 'api', label: 'API', position: 'left'},
        {
          type: 'docsVersionDropdown',
          position: 'right',
        },
      ],
    },
    footer: {
      links: [
        {
          title: '文档',
          items: [
            {label: '快速开始', to: 'docs/quickstart'},
            {label: 'API参考', to: 'docs/api'},
          ],
        },
      ],
    },
  },
  
  presets: [
    [
      '@docusaurus/preset-classic',
      {
        docs: {
          sidebarPath: require.resolve('./sidebars.js'),
          editUrl: 'https://github.com/org/docs/edit/main/',
        },
        theme: {
          customCss: require.resolve('./src/css/custom.css'),
        },
      },
    ],
  ],
}
```

### 3.3 文档CI/CD

```yaml
# .github/workflows/docs.yml
name: Documentation

on:
  push:
    paths:
      - 'docs/**'
      - 'src/**'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Check links
        run: npm run check-links
      
      - name: Lint Markdown
        run: npx markdownlint-cli docs/
      
      - name: Build
        run: npm run build
      
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./build
```

## 4. 资源索引

| 工具 | 用途 | 官网 |
|-----|-----|------|
| Docusaurus | 文档站点 | https://docusaurus.io |
| MkDocs | 静态文档 | https://mkdocs.org |
| Swagger/OpenAPI | API文档 | https://swagger.io |
| PlantUML | 图表 | https://plantuml.com |
| Mermaid | 图表 | https://mermaid.js.org |

## 5. 关联知识

- C01_Code_Craftsmanship
- C02_Automation_Strategy
- A03_Core_Technologies

## 6. 学习建议

1. 选择适合的文档框架
2. 建立文档编写规范
3. 集成到CI/CD流程
4. 定期审查更新

---
*最后更新：2024年*
