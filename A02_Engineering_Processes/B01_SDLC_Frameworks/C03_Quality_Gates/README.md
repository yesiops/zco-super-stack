# C03_Quality_Gates - 质量门禁

## 1. 主题定位

### 1.1 定义与范围

质量门禁（Quality Gates）是软件开发生命周期中用于确保产品质量的一系列自动化检查和人工评审机制。本知识单元涵盖从代码提交、构建、测试到部署各阶段的质量控制点，包括静态代码分析、自动化测试、安全扫描、性能基准测试等多维度质量验证方法。

### 1.2 业务价值

质量门禁通过自动化检测和标准化流程，为组织带来以下核心价值：

- **缺陷前移发现**：在开发阶段捕获80%以上的缺陷，降低修复成本
- **技术债务控制**：防止低质量代码进入主干，保持代码库健康度
- **合规性保障**：确保代码符合安全、法律、行业标准要求
- **发布信心提升**：基于客观质量指标做发布决策，降低生产故障
- **团队协作规范**：建立统一的质量标准和反馈机制

### 1.3 适用场景

| 场景类型 | 适用性 | 说明 |
|---------|-------|------|
| 持续交付流水线 | ⭐⭐⭐⭐⭐ | 质量门禁是CI/CD的核心组成部分 |
| 微服务架构 | ⭐⭐⭐⭐⭐ | 服务边界清晰，适合独立质量验证 |
| 金融/医疗系统 | ⭐⭐⭐⭐⭐ | 强监管场景，质量门禁是必需 |
| 开源项目 | ⭐⭐⭐⭐ | 保证贡献代码质量，维护项目声誉 |
| 快速原型开发 | ⭐⭐ | 初期可能简化，但核心门禁保留 |
| 遗留系统维护 | ⭐⭐⭐ | 需逐步建立，避免一次引入过多阻力 |

## 2. 核心概念

### 2.1 质量门禁架构

```
┌─────────────────────────────────────────────────────────────────┐
│                     质量门禁流水线架构                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  代码提交 ──► 预提交门禁 ──► 构建门禁 ──► 集成门禁 ──► 发布门禁   │
│    │            │            │            │            │         │
│    ▼            ▼            ▼            ▼            ▼         │
│ ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐          │
│ │Lint  │   │编译  │   │单元  │   │集成  │   │性能  │          │
│ │Format│   │静态  │   │测试  │   │测试  │   │安全  │          │
│ │      │   │分析  │   │覆盖率│   │契约  │   │合规  │          │
│ └──────┘   └──────┘   └──────┘   └──────┘   └──────┘          │
│    │            │            │            │            │         │
│  反馈          反馈          反馈          反馈          反馈      │
│  <5分钟      <10分钟       <30分钟       <2小时        <4小时      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 门禁类型与标准

| 门禁层级 | 触发时机 | 检查内容 | 耗时目标 | 失败策略 |
|---------|---------|---------|---------|---------|
| **预提交** | 本地提交前 | 代码格式、基础Lint、单元测试（变更相关） | <5分钟 | 阻止提交 |
| **构建门禁** | PR/MR创建 | 编译、静态分析、全量单元测试 | <10分钟 | 阻止合并 |
| **集成门禁** | 合并到主干 | 集成测试、组件测试、安全扫描 | <30分钟 | 阻止部署 |
| **验收门禁** | 部署到预发布 | E2E测试、性能基准、契约测试 | <2小时 | 阻止发布 |
| **发布门禁** | 生产部署前 | 合规检查、审计日志、回滚验证 | <4小时 | 阻止发布 |

### 2.3 代码质量度量维度

```
代码质量金字塔

                    ┌─────────┐
                    │ 可维护性 │  ← 圈复杂度、代码异味、技术债务比率
                   ┌┴─────────┤
                   │  可靠性  │  ← 缺陷密度、测试通过率、MTBF
                  ┌┴──────────┤
                  │   安全性   │  ← 漏洞数量、OWASP合规率
                 ┌┴───────────┤
                 │   可测试性   │  ← 测试覆盖率、测试覆盖率趋势
                ┌┴────────────┤
                │    可读性     │  ← 命名规范、注释覆盖率、文档完整性
                └─────────────┘
```

### 2.4 门禁策略模式

```python
# 质量门禁策略枚举
from enum import Enum

class GateStrategy(Enum):
    """质量门禁策略模式"""
    
    # 严格模式：任何质量问题都阻止流程
    STRICT = "strict"
    
    # 警告模式：记录问题但不阻止，超阈值才阻止
    WARNING = "warning"
    
    # 渐进模式：新代码严格，遗留代码宽松
    PROGRESSIVE = "progressive"
    
    # 基线模式：只允许改善，不允许恶化
    BASELINE = "baseline"
    
    # 豁免模式：特定情况可临时跳过
    EXEMPTION = "exemption"
```

## 3. 技术实践

### 3.1 预提交门禁配置

#### 3.1.1 Husky + lint-staged 配置

```javascript
// package.json - Node.js项目预提交配置
{
  "name": "quality-gates-demo",
  "scripts": {
    "lint": "eslint . --ext .js,.ts,.tsx",
    "lint:fix": "eslint . --ext .js,.ts,.tsx --fix",
    "format": "prettier --write \"**/*.{js,ts,tsx,json,md}\"",
    "format:check": "prettier --check \"**/*.{js,ts,tsx,json,md}\"",
    "test:staged": "jest --bail --findRelatedTests --passWithNoTests",
    "type-check": "tsc --noEmit",
    "prepare": "husky install"
  },
  "lint-staged": {
    "*.{js,ts,tsx}": [
      "eslint --fix",
      "prettier --write",
      "jest --bail --findRelatedTests --passWithNoTests"
    ],
    "*.{json,md}": [
      "prettier --write"
    ]
  },
  "devDependencies": {
    "@typescript-eslint/eslint-plugin": "^6.0.0",
    "@typescript-eslint/parser": "^6.0.0",
    "eslint": "^8.45.0",
    "husky": "^8.0.3",
    "jest": "^29.6.0",
    "lint-staged": "^13.2.0",
    "prettier": "^3.0.0",
    "typescript": "^5.1.0"
  }
}
```

```yaml
# .husky/pre-commit
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

# 运行lint-staged
npx lint-staged

# 运行类型检查
npm run type-check

# 检查commit message格式
npx --no-install commitlint --edit ${1}
```

```javascript
// commitlint.config.js - 提交信息规范
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'feat',     // 新功能
        'fix',      // 修复
        'docs',     // 文档
        'style',    // 格式
        'refactor', // 重构
        'perf',     // 性能
        'test',     // 测试
        'chore',    // 构建
        'ci',       // CI
        'security', // 安全
      ],
    ],
    'scope-enum': [
      2,
      'always',
      ['api', 'ui', 'db', 'auth', 'config', 'deps'],
    ],
    'subject-min-length': [2, 'always', 10],
    'subject-max-length': [2, 'always', 72],
  },
};
```

#### 3.1.2 Python 预提交配置

```yaml
# .pre-commit-config.yaml
# Python项目预提交钩子配置
# 安装: pip install pre-commit
# 启用: pre-commit install

default_stages: [commit, push]
default_language_version:
  python: python3.11

repos:
  # 基础格式检查
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-json
      - id: check-toml
      - id: check-added-large-files
        args: ['--maxkb=500']
      - id: check-merge-conflict
      - id: check-case-conflict
      - id: detect-private-key
      - id: forbid-new-submodules

  # Python代码格式化 - Black
  - repo: https://github.com/psf/black
    rev: 23.12.1
    hooks:
      - id: black
        language_version: python3.11
        args: ['--line-length=100']

  # Python导入排序 - isort
  - repo: https://github.com/PyCQA/isort
    rev: 5.13.2
    hooks:
      - id: isort
        args: ['--profile=black', '--line-length=100']

  # Python代码检查 - Ruff
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.1.9
    hooks:
      - id: ruff
        args: ['--fix', '--exit-non-zero-on-fix']
      - id: ruff-format

  # Python类型检查 - mypy
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.8.0
    hooks:
      - id: mypy
        additional_dependencies: [types-all]
        args: ['--strict', '--ignore-missing-imports']

  # 安全检查 - Bandit
  - repo: https://github.com/PyCQA/bandit
    rev: 1.7.6
    hooks:
      - id: bandit
        args: ['-c', 'pyproject.toml', '-r', '.']
        additional_dependencies: ["bandit[toml]"]

  # 文档字符串检查 - pydocstyle
  - repo: https://github.com/PyCQA/pydocstyle
    rev: 6.3.0
    hooks:
      - id: pydocstyle
        args: ['--ignore=D100,D104']

  # Markdown格式化
  - repo: https://github.com/pre-commit/mirrors-prettier
    rev: v3.1.0
    hooks:
      - id: prettier
        types_or: [markdown, yaml, json]

  # 提交信息检查
  - repo: https://github.com/compilerla/conventional-pre-commit
    rev: v3.1.0
    hooks:
      - id: conventional-pre-commit
        stages: [commit-msg]
        args: []

  # Python单元测试（仅变更文件）
  - repo: local
    hooks:
      - id: pytest-check
        name: pytest-check
        entry: pytest
        language: system
        pass_filenames: false
        always_run: true
        args: [
          '-x',  # 遇到第一个失败就停止
          '--tb=short',
          '-q',
          '--cov=src',
          '--cov-report=term-missing',
          '--cov-fail-under=80'
        ]
```

```toml
# pyproject.toml - 工具配置集中管理
[tool.black]
line-length = 100
target-version = ['py311']
include = '\.pyi?$'
extend-exclude = '''
/(
  # directories
  \.eggs
  | \.git
  | \.hg
  | \.mypy_cache
  | \.tox
  | \.venv
  | build
  | dist
)/
'''

[tool.isort]
profile = "black"
line_length = 100
multi_line_output = 3
include_trailing_comma = true
force_grid_wrap = 0
use_parentheses = true
ensure_newline_before_comments = true

[tool.ruff]
target-version = "py311"
line-length = 100
select = [
    "E",   # pycodestyle errors
    "W",   # pycodestyle warnings
    "F",   # Pyflakes
    "I",   # isort
    "N",   # pep8-naming
    "B",   # flake8-bugbear
    "C4",  # flake8-comprehensions
    "UP",  # pyupgrade
    "S",   # flake8-bandit
]
ignore = ["E501"]

[tool.mypy]
python_version = "3.11"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
disallow_incomplete_defs = true
check_untyped_defs = true
disallow_untyped_decorators = false
no_implicit_optional = true
warn_redundant_casts = true
warn_unused_ignores = true
warn_no_return = true
warn_unreachable = true
strict_equality = true

[tool.bandit]
exclude_dirs = ["tests"]
skips = ["B101", "B601"]

[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py", "*_test.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]
addopts = [
    "--strict-markers",
    "--strict-config",
    "--verbose",
]
markers = [
    "slow: marks tests as slow (deselect with '-m \"not slow\"')",
    "integration: marks tests as integration tests",
    "unit: marks tests as unit tests",
]

[tool.coverage.run]
source = ["src"]
omit = ["*/tests/*", "*/test_*.py"]

[tool.coverage.report]
exclude_lines = [
    "pragma: no cover",
    "def __repr__",
    "if self.debug:",
    "if settings.DEBUG",
    "raise AssertionError",
    "raise NotImplementedError",
    "if 0:",
    "if __name__ == .__main__.:",
    "class .*\\bProtocol\\):",
    "@(abc\\.)?abstractmethod",
]
show_missing = true
skip_covered = false
fail_under = 80
```

### 3.2 CI/CD 质量门禁

#### 3.2.1 GitHub Actions 完整配置

```yaml
# .github/workflows/quality-gates.yml
name: Quality Gates

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

env:
  PYTHON_VERSION: '3.11'
  NODE_VERSION: '20'

jobs:
  # ========== 预检阶段 ==========
  preflight:
    name: 预检检查
    runs-on: ubuntu-latest
    outputs:
      should_skip: ${{ steps.skip_check.outputs.should_skip }}
      changed_files: ${{ steps.changes.outputs.files }}
    steps:
      - uses: actions/checkout@v4
      
      - name: 检查是否可跳过
        id: skip_check
        uses: fkirc/skip-duplicate-actions@v5
        with:
          skip_after_successful_duplicate: 'true'
          concurrent_skipping: 'same_content_newer'
          paths_ignore: '["**.md", "**.txt", "docs/**"]'
      
      - name: 检测变更文件
        id: changes
        uses: dorny/paths-filter@v2
        with:
          filters: |
            python:
              - 'src/**/*.py'
              - 'tests/**/*.py'
            javascript:
              - 'src/**/*.{js,ts,tsx}'
              - 'tests/**/*.{js,ts}'
            infrastructure:
              - 'terraform/**'
              - 'kubernetes/**'

  # ========== Python代码质量 ==========
  python-quality:
    name: Python代码质量
    needs: preflight
    if: ${{ needs.preflight.outputs.should_skip != 'true' }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: 设置Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}
          cache: 'pip'
      
      - name: 安装依赖
        run: |
          pip install -r requirements-dev.txt
          pip install -e .
      
      - name: 代码格式化检查 (Black)
        run: black --check --diff src tests
      
      - name: 导入排序检查 (isort)
        run: isort --check-only --diff src tests
      
      - name: 代码规范检查 (Ruff)
        run: ruff check src tests
      
      - name: 类型检查 (mypy)
        run: mypy src
      
      - name: 安全扫描 (Bandit)
        run: bandit -r src -f json -o bandit-report.json || true
      
      - name: 上传安全报告
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: bandit-report
          path: bandit-report.json

  # ========== Python单元测试 ==========
  python-tests:
    name: Python单元测试
    needs: python-quality
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ['3.10', '3.11', '3.12']
    steps:
      - uses: actions/checkout@v4
      
      - name: 设置Python ${{ matrix.python-version }}
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
          cache: 'pip'
      
      - name: 安装依赖
        run: |
          pip install -r requirements-dev.txt
          pip install -e .
      
      - name: 运行单元测试
        run: |
          pytest tests/unit \
            --cov=src \
            --cov-report=xml \
            --cov-report=term-missing \
            --cov-fail-under=80 \
            -v
      
      - name: 上传覆盖率报告
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage.xml
          flags: unittests
          name: codecov-umbrella

  # ========== 集成测试 ==========
  integration-tests:
    name: 集成测试
    needs: python-tests
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
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
      - uses: actions/checkout@v4
      
      - name: 设置Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}
          cache: 'pip'
      
      - name: 安装依赖
        run: |
          pip install -r requirements-dev.txt
          pip install -e .
      
      - name: 运行集成测试
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db
          REDIS_URL: redis://localhost:6379/0
        run: |
          pytest tests/integration -v --tb=short

  # ========== 静态应用安全测试 (SAST) ==========
  sast:
    name: SAST安全扫描
    needs: preflight
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - name: 运行Semgrep扫描
        uses: returntocorp/semgrep-action@v1
        with:
          config: >-
            p/security-audit
            p/owasp-top-ten
            p/cwe-top-25
            p/python
      
      - name: 运行CodeQL分析
        uses: github/codeql-action/init@v3
        with:
          languages: python, javascript
          queries: security-extended,security-and-quality
      
      - name: 执行CodeQL分析
        uses: github/codeql-action/analyze@v3

  # ========== 依赖安全检查 ==========
  dependency-check:
    name: 依赖安全扫描
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: 设置Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}
      
      - name: Python依赖扫描 (Safety)
        run: |
          pip install safety
          safety check -r requirements.txt --full-report
      
      - name: Python依赖扫描 (pip-audit)
        run: |
          pip install pip-audit
          pip-audit --desc --format=json -o pip-audit-report.json || true
      
      - name: 许可证合规检查 (FOSSA)
        uses: fossas/fossa-action@main
        with:
          api-key: ${{ secrets.FOSSA_API_KEY }}

  # ========== 代码覆盖率门禁 ==========
  coverage-gate:
    name: 覆盖率门禁
    needs: python-tests
    runs-on: ubuntu-latest
    steps:
      - name: 覆盖率阈值检查
        uses: 5monkeys/cobertura-action@master
        with:
          path: coverage.xml
          repo_token: ${{ secrets.GITHUB_TOKEN }}
          minimum_coverage: 80
          show_line: true
          show_branch: true
          fail_below_threshold: true

  # ========== 性能基准测试 ==========
  performance-benchmark:
    name: 性能基准测试
    needs: integration-tests
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'
    steps:
      - uses: actions/checkout@v4
      
      - name: 设置Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}
      
      - name: 安装依赖
        run: |
          pip install -r requirements-dev.txt
          pip install pytest-benchmark
      
      - name: 运行基准测试
        run: |
          pytest tests/benchmark --benchmark-only --benchmark-json=output.json
      
      - name: 与基准比较
        uses: benchmark-action/github-action-benchmark@v1
        with:
          tool: 'pytest'
          output-file-path: output.json
          github-token: ${{ secrets.GITHUB_TOKEN }}
          auto-push: false
          comment-on-alert: true
          fail-on-alert: true
          alert-threshold: '150%'

  # ========== 容器镜像扫描 ==========
  container-scan:
    name: 容器镜像扫描
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'
    steps:
      - uses: actions/checkout@v4
      
      - name: 构建测试镜像
        run: docker build -t test-image:latest .
      
      - name: Trivy镜像扫描
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'test-image:latest'
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'
          exit-code: '1'
          ignore-unfixed: true
      
      - name: 上传扫描结果
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: 'trivy-results.sarif'

  # ========== 质量门禁汇总 ==========
  quality-summary:
    name: 质量门禁汇总
    needs:
      - python-quality
      - python-tests
      - integration-tests
      - sast
      - dependency-check
      - coverage-gate
      - performance-benchmark
      - container-scan
    runs-on: ubuntu-latest
    if: always()
    steps:
      - name: 检查所有任务状态
        run: |
          echo "Python代码质量: ${{ needs.python-quality.result }}"
          echo "Python单元测试: ${{ needs.python-tests.result }}"
          echo "集成测试: ${{ needs.integration-tests.result }}"
          echo "SAST扫描: ${{ needs.sast.result }}"
          echo "依赖检查: ${{ needs.dependency-check.result }}"
          echo "覆盖率门禁: ${{ needs.coverage-gate.result }}"
          echo "性能测试: ${{ needs.performance-benchmark.result }}"
          echo "容器扫描: ${{ needs.container-scan.result }}"
      
      - name: 质量门禁决策
        if: |
          needs.python-quality.result != 'success' ||
          needs.python-tests.result != 'success' ||
          needs.integration-tests.result != 'success' ||
          needs.sast.result != 'success' ||
          needs.dependency-check.result != 'success' ||
          needs.coverage-gate.result != 'success'
        run: |
          echo "❌ 质量门禁未通过"
          exit 1
      
      - name: 门禁通过
        run: echo "✅ 所有质量门禁已通过"
```

### 3.3 SonarQube 质量门禁配置

```properties
# sonar-project.properties - SonarQube项目配置
# 必须配置
sonar.projectKey=my-project
sonar.projectName=My Project
sonar.projectVersion=1.0.0
sonar.sourceEncoding=UTF-8

# 源码路径
sonar.sources=src
sonar.tests=tests

# Python特定配置
sonar.python.version=3.11
sonar.python.coverage.reportPaths=coverage.xml
sonar.python.xunit.reportPath=test-results.xml
sonar.python.bandit.reportPaths=bandit-report.json
sonar.python.pylint.reportPaths=pylint-report.txt

# 排除路径
sonar.exclusions=**/migrations/**,**/tests/**,**/test_*.py
sonar.coverage.exclusions=**/tests/**,**/test_*.py,**/migrations/**

# 文件类型关联
sonar.inclusions=**/*.py
```

```yaml
# sonar-quality-gate.yml - 质量门禁定义
# SonarQube 10.x 质量门禁配置

name: "严格质量门禁"
conditions:
  # 代码覆盖率
  - metric: coverage
    operator: LT
    threshold: "80"
    
  # 重复代码
  - metric: duplicated_lines_density
    operator: GT
    threshold: "3"
    
  # 代码异味
  - metric: code_smells
    operator: GT
    threshold: "50"
    
  # 漏洞
  - metric: vulnerabilities
    operator: GT
    threshold: "0"
    
  # 安全热点
  - metric: security_hotspots_reviewed
    operator: LT
    threshold: "100"
    
  # 技术债务比率
  - metric: sqale_debt_ratio
    operator: GT
    threshold: "5"
    
  # 可靠性评级
  - metric: reliability_rating
    operator: GT
    threshold: "1"
    
  # 安全评级
  - metric: security_rating
    operator: GT
    threshold: "1"
    
  # 可维护性评级
  - metric: sqale_rating
    operator: GT
    threshold: "1"
```

```python
#!/usr/bin/env python3
"""
SonarQube质量门禁检查脚本
用于CI/CD流水线中程序化检查质量门禁状态
"""

import requests
import sys
import time
import argparse
from typing import Dict, Optional


class SonarQubeQualityGate:
    """SonarQube质量门禁客户端"""
    
    def __init__(self, base_url: str, token: str):
        self.base_url = base_url.rstrip('/')
        self.auth = (token, '')  # SonarQube使用token作为用户名，密码为空
        self.headers = {"Accept": "application/json"}
    
    def get_quality_gate_status(self, project_key: str, 
                                branch: str = None) -> Dict:
        """获取质量门禁状态"""
        url = f"{self.base_url}/api/qualitygates/project_status"
        params = {"projectKey": project_key}
        if branch:
            params["branch"] = branch
        
        response = requests.get(url, auth=self.auth, 
                              headers=self.headers, params=params)
        response.raise_for_status()
        return response.json()
    
    def wait_for_analysis(self, project_key: str, 
                         timeout: int = 300,
                         interval: int = 10) -> bool:
        """等待分析完成"""
        url = f"{self.base_url}/api/ce/component"
        params = {"component": project_key}
        
        start_time = time.time()
        while time.time() - start_time < timeout:
            response = requests.get(url, auth=self.auth, 
                                  headers=self.headers, params=params)
            if response.status_code == 200:
                data = response.json()
                current = data.get("current", {})
                if current.get("status") == "SUCCESS":
                    return True
                if current.get("status") in ["FAILED", "CANCELED"]:
                    return False
            time.sleep(interval)
        
        return False
    
    def check_quality_gate(self, project_key: str, 
                          branch: str = None,
                          wait: bool = True,
                          timeout: int = 300) -> bool:
        """
        检查质量门禁
        返回True表示通过，False表示失败
        """
        if wait:
            print(f"等待SonarQube分析完成...")
            if not self.wait_for_analysis(project_key, timeout):
                print("❌ SonarQube分析失败或超时")
                return False
        
        result = self.get_quality_gate_status(project_key, branch)
        status = result.get("projectStatus", {}).get("status", "ERROR")
        conditions = result.get("projectStatus", {}).get("conditions", [])
        
        print(f"\n质量门禁状态: {status}")
        print("-" * 60)
        
        for condition in conditions:
            metric = condition.get("metricKey", "unknown")
            status_icon = "✅" if condition.get("status") == "OK" else "❌"
            actual = condition.get("actualValue", "N/A")
            threshold = condition.get("errorThreshold", "N/A")
            print(f"{status_icon} {metric}: 实际值={actual}, 阈值={threshold}")
        
        return status == "OK"


def main():
    parser = argparse.ArgumentParser(description="SonarQube质量门禁检查")
    parser.add_argument("--url", required=True, help="SonarQube URL")
    parser.add_argument("--token", required=True, help="SonarQube Token")
    parser.add_argument("--project", required=True, help="项目Key")
    parser.add_argument("--branch", help="分支名")
    parser.add_argument("--wait", action="store_true", help="等待分析完成")
    parser.add_argument("--timeout", type=int, default=300, help="超时时间")
    
    args = parser.parse_args()
    
    gate = SonarQubeQualityGate(args.url, args.token)
    passed = gate.check_quality_gate(
        args.project, 
        args.branch, 
        args.wait, 
        args.timeout
    )
    
    sys.exit(0 if passed else 1)


if __name__ == "__main__":
    main()
```

### 3.4 自定义质量指标收集

```python
#!/usr/bin/env python3
"""
质量指标收集与分析系统
收集代码质量指标，生成趋势报告
"""

import json
import subprocess
from dataclasses import dataclass, asdict
from typing import Dict, List, Optional
from datetime import datetime
from pathlib import Path
import sqlite3


@dataclass
class CodeMetrics:
    """代码质量指标"""
    timestamp: str
    commit_hash: str
    branch: str
    
    # 代码规模
    total_lines: int
    code_lines: int
    comment_lines: int
    blank_lines: int
    
    # 复杂度
    avg_complexity: float
    max_complexity: int
    functions_over_10: int
    
    # 覆盖率
    line_coverage: float
    branch_coverage: float
    
    # 代码异味
    code_smells: int
    code_smell_density: float  # 每千行代码异味数
    
    # 安全
    vulnerabilities: int
    security_hotspots: int
    
    # 重复
    duplicated_lines: int
    duplication_percent: float
    
    # 债务
    technical_debt_minutes: int
    debt_ratio: float


class MetricsCollector:
    """指标收集器"""
    
    def __init__(self, db_path: str = "quality_metrics.db"):
        self.db_path = db_path
        self._init_db()
    
    def _init_db(self):
        """初始化数据库"""
        conn = sqlite3.connect(self.db_path)
        conn.execute("""
            CREATE TABLE IF NOT EXISTS metrics (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                timestamp TEXT,
                commit_hash TEXT,
                branch TEXT,
                total_lines INTEGER,
                code_lines INTEGER,
                comment_lines INTEGER,
                blank_lines INTEGER,
                avg_complexity REAL,
                max_complexity INTEGER,
                functions_over_10 INTEGER,
                line_coverage REAL,
                branch_coverage REAL,
                code_smells INTEGER,
                code_smell_density REAL,
                vulnerabilities INTEGER,
                security_hotspots INTEGER,
                duplicated_lines INTEGER,
                duplication_percent REAL,
                technical_debt_minutes INTEGER,
                debt_ratio REAL
            )
        """)
        conn.commit()
        conn.close()
    
    def collect_git_info(self) -> Dict:
        """收集Git信息"""
        def run_git(cmd: List[str]) -> str:
            result = subprocess.run(
                ["git"] + cmd,
                capture_output=True,
                text=True
            )
            return result.stdout.strip()
        
        return {
            "commit": run_git(["rev-parse", "HEAD"]),
            "branch": run_git(["rev-parse", "--abbrev-ref", "HEAD"]),
            "timestamp": datetime.now().isoformat()
        }
    
    def collect_code_stats(self, source_dir: str = "src") -> Dict:
        """收集代码统计信息"""
        total_lines = 0
        code_lines = 0
        comment_lines = 0
        blank_lines = 0
        
        source_path = Path(source_dir)
        for file_path in source_path.rglob("*.py"):
            with open(file_path, 'r', encoding='utf-8') as f:
                for line in f:
                    total_lines += 1
                    stripped = line.strip()
                    if not stripped:
                        blank_lines += 1
                    elif stripped.startswith('#'):
                        comment_lines += 1
                    else:
                        code_lines += 1
        
        return {
            "total_lines": total_lines,
            "code_lines": code_lines,
            "comment_lines": comment_lines,
            "blank_lines": blank_lines
        }
    
    def collect_coverage(self, coverage_file: str = "coverage.xml") -> Dict:
        """从coverage.xml收集覆盖率数据"""
        try:
            import xml.etree.ElementTree as ET
            tree = ET.parse(coverage_file)
            root = tree.getroot()
            
            line_rate = float(root.get("line-rate", 0))
            branch_rate = float(root.get("branch-rate", 0))
            
            return {
                "line_coverage": line_rate * 100,
                "branch_coverage": branch_rate * 100
            }
        except Exception as e:
            print(f"警告：无法读取覆盖率文件: {e}")
            return {"line_coverage": 0, "branch_coverage": 0}
    
    def collect_complexity(self, source_dir: str = "src") -> Dict:
        """收集圈复杂度信息"""
        try:
            import radon
            from radon.complexity import cc_visit
            from radon.raw import analyze
            
            complexities = []
            functions_over_10 = 0
            
            source_path = Path(source_dir)
            for file_path in source_path.rglob("*.py"):
                with open(file_path, 'r', encoding='utf-8') as f:
                    blocks = cc_visit(f.read())
                    for block in blocks:
                        complexities.append(block.complexity)
                        if block.complexity > 10:
                            functions_over_10 += 1
            
            if complexities:
                return {
                    "avg_complexity": sum(complexities) / len(complexities),
                    "max_complexity": max(complexities),
                    "functions_over_10": functions_over_10
                }
        except ImportError:
            print("警告：radon未安装，跳过复杂度分析")
        
        return {"avg_complexity": 0, "max_complexity": 0, "functions_over_10": 0}
    
    def collect_all(self, source_dir: str = "src") -> CodeMetrics:
        """收集所有指标"""
        git_info = self.collect_git_info()
        code_stats = self.collect_code_stats(source_dir)
        coverage = self.collect_coverage()
        complexity = self.collect_complexity(source_dir)
        
        # 计算代码异味密度
        smell_density = 0
        if code_stats["code_lines"] > 0:
            smell_density = 0  # 需要从SonarQube或其他工具获取
        
        return CodeMetrics(
            timestamp=git_info["timestamp"],
            commit_hash=git_info["commit"],
            branch=git_info["branch"],
            total_lines=code_stats["total_lines"],
            code_lines=code_stats["code_lines"],
            comment_lines=code_stats["comment_lines"],
            blank_lines=code_stats["blank_lines"],
            avg_complexity=complexity["avg_complexity"],
            max_complexity=complexity["max_complexity"],
            functions_over_10=complexity["functions_over_10"],
            line_coverage=coverage["line_coverage"],
            branch_coverage=coverage["branch_coverage"],
            code_smells=0,  # 需要从外部工具获取
            code_smell_density=smell_density,
            vulnerabilities=0,
            security_hotspots=0,
            duplicated_lines=0,
            duplication_percent=0,
            technical_debt_minutes=0,
            debt_ratio=0
        )
    
    def save_metrics(self, metrics: CodeMetrics):
        """保存指标到数据库"""
        conn = sqlite3.connect(self.db_path)
        conn.execute("""
            INSERT INTO metrics VALUES (
                NULL, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?
            )
        """, tuple(asdict(metrics).values()))
        conn.commit()
        conn.close()
    
    def get_trends(self, branch: str = "main", limit: int = 30) -> List[Dict]:
        """获取指标趋势"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.execute("""
            SELECT * FROM metrics 
            WHERE branch = ? 
            ORDER BY timestamp DESC 
            LIMIT ?
        """, (branch, limit))
        
        columns = [description[0] for description in cursor.description]
        results = []
        for row in cursor.fetchall():
            results.append(dict(zip(columns, row)))
        
        conn.close()
        return results
    
    def generate_report(self, branch: str = "main") -> str:
        """生成质量报告"""
        trends = self.get_trends(branch)
        
        if len(trends) < 2:
            return "数据不足，无法生成趋势报告"
        
        latest = trends[0]
        previous = trends[1]
        
        report = f"""
# 代码质量趋势报告

## 当前状态 ({latest['timestamp'][:10]})
| 指标 | 当前值 | 变化 | 趋势 |
|-----|-------|-----|-----|
| 代码行数 | {latest['code_lines']} | {latest['code_lines'] - previous['code_lines']:+d} | {'📈' if latest['code_lines'] > previous['code_lines'] else '📉'} |
| 行覆盖率 | {latest['line_coverage']:.1f}% | {latest['line_coverage'] - previous['line_coverage']:+.1f}% | {'📈' if latest['line_coverage'] > previous['line_coverage'] else '📉'} |
| 平均复杂度 | {latest['avg_complexity']:.1f} | {latest['avg_complexity'] - previous['avg_complexity']:+.1f} | {'📉' if latest['avg_complexity'] < previous['avg_complexity'] else '📈'} |
| 高复杂度函数 | {latest['functions_over_10']} | {latest['functions_over_10'] - previous['functions_over_10']:+d} | {'📉' if latest['functions_over_10'] < previous['functions_over_10'] else '📈'} |

## 健康度评分
"""
        # 计算综合评分
        score = 100
        if latest['line_coverage'] < 80:
            score -= (80 - latest['line_coverage'])
        if latest['avg_complexity'] > 5:
            score -= (latest['avg_complexity'] - 5) * 2
        if latest['functions_over_10'] > 0:
            score -= latest['functions_over_10'] * 2
        
        score = max(0, min(100, score))
        
        emoji = "🟢" if score >= 80 else "🟡" if score >= 60 else "🔴"
        report += f"\n综合评分: {emoji} {score:.1f}/100\n"
        
        return report


if __name__ == "__main__":
    collector = MetricsCollector()
    metrics = collector.collect_all()
    collector.save_metrics(metrics)
    print(collector.generate_report())
```

## 4. 资源索引

### 4.1 官方文档

| 资源名称 | 链接 | 说明 |
|---------|------|------|
| SonarQube文档 | https://docs.sonarqube.org | 代码质量管理平台 |
| SonarCloud | https://sonarcloud.io | 云端代码质量服务 |
| Codecov | https://codecov.io | 覆盖率分析服务 |
| Pre-commit | https://pre-commit.com | Git钩子框架 |
| GitHub Actions | https://docs.github.com/actions | CI/CD自动化 |
| GitLab CI | https://docs.gitlab.com/ee/ci | GitLab持续集成 |
| OWASP | https://owasp.org | 安全最佳实践 |

### 4.2 推荐工具

| 类别 | 工具 | 用途 |
|-----|-----|-----|
| **Python代码质量** | Black | 代码格式化 |
| | isort | 导入排序 |
| | Ruff | 快速代码检查 |
| | mypy | 类型检查 |
| | Bandit | 安全扫描 |
| | Pylint | 代码规范检查 |
| | Radon | 复杂度分析 |
| **JavaScript/TypeScript** | ESLint | 代码检查 |
| | Prettier | 代码格式化 |
| | TypeScript | 类型检查 |
| **SAST/DAST** | Semgrep | 静态分析 |
| | Trivy | 漏洞扫描 |
| | Snyk | 依赖安全 |
| | CodeQL | GitHub安全分析 |
| **测试** | pytest | Python测试框架 |
| | Jest | JavaScript测试 |
| | Playwright | E2E测试 |
| **覆盖率** | coverage.py | Python覆盖率 |
| | Istanbul | JavaScript覆盖率 |

### 4.3 参考标准

1. **ISO/IEC 25010** - 系统和软件质量模型
2. **CISQ标准** - 软件质量国际标准
3. **OWASP ASVS** - 应用安全验证标准
4. **NIST SSDF** - 安全软件开发框架

### 4.4 书籍推荐

1. **《重构》** - Martin Fowler
2. **《代码整洁之道》** - Robert C. Martin
3. **《软件质量保障》** - 朱少民
4. **《Continuous Delivery》** - Jez Humble, David Farley
5. **《Accelerate》** - Nicole Forsgren et al.

## 5. 关联知识

### 5.1 上游关联

```
A02_Engineering_Processes/B01_SDLC_Frameworks
├── C01_Agile_Methodologies ────► 质量门禁是敏捷技术实践的一部分
├── C02_DevOps_Integration ──────► CI/CD流水线承载质量门禁
└── C03_Quality_Gates (本单元)
```

### 5.2 下游关联

```
C03_Quality_Gates
├── A03_Core_Technologies
│   ├── B01_Programming_Languages ► 各语言的代码质量工具
│   └── B02_Software_Architecture ► 架构质量门禁
├── A05_Operations_Excellence
│   └── B01_Monitoring_Observability ► 运行时质量监控
└── A06_Leadership_Management
    └── B02_Project_Management ────► 质量指标与项目管理
```

### 5.3 与其他流程的关系

| 流程 | 关系 | 说明 |
|-----|-----|-----|
| 持续集成 | 承载者 | 质量门禁在CI流水线中执行 |
| 代码审查 | 互补者 | 自动化检查+人工审查双保险 |
| 测试策略 | 组成部分 | 测试覆盖率是质量门禁核心指标 |
| 安全开发 | 组成部分 | 安全扫描是质量门禁必要环节 |
| 发布管理 | 决策者 | 质量门禁结果决定能否发布 |

## 6. 学习建议

### 6.1 新手入门路径

```
第1周：理解概念
├── 了解质量门禁的定义和价值
├── 学习常见的质量指标含义
└── 熟悉团队现有的质量门禁配置

第2周：工具上手
├── 安装并配置pre-commit
├── 配置本地代码格式化工具
└── 运行单元测试并查看覆盖率

第3周：CI/CD集成
├── 了解团队CI/CD流水线结构
├── 学习GitHub Actions/GitLab CI基础
└── 在本地模拟CI运行

第4周：实践优化
├── 识别当前门禁的改进点
├── 提交质量改进PR
└── 总结个人质量门禁使用心得
```

### 6.2 进阶学习路径

1. **度量体系设计**：学习如何定义适合团队的质量指标
2. **工具链定制**：根据团队技术栈定制质量门禁
3. **质量数据分析**：使用数据驱动质量改进
4. **团队赋能**：推广质量文化，培训团队成员

### 6.3 实践检查清单

- [ ] 本地开发环境配置了pre-commit
- [ ] 提交前运行所有本地质量检查
- [ ] 理解CI/CD各阶段的门禁检查
- [ ] 能够解读覆盖率报告
- [ ] 能够修复常见的代码规范问题
- [ ] 了解安全扫描发现的问题类型
- [ ] 参与团队质量门禁规则的讨论
- [ ] 能够为新项目配置质量门禁

---

*最后更新：2024年 | 维护者：工程流程委员会*
