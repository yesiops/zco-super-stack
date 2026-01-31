# C03_Security_Baselines - 安全基线

## 1. 主题定位

### 1.1 定义与范围

安全基线（Security Baselines）是定义系统、应用、网络最低安全配置要求的标准集合。本知识单元涵盖安全开发实践、基础设施加固、访问控制、安全监控等内容，为组织提供可执行的安全标准。

### 1.2 业务价值

- 降低安全风险
- 满足合规要求
- 防止数据泄露
- 保障业务连续性
- 建立安全文化

### 1.3 适用场景

| 场景 | 适用性 |
|-----|-------|
| 云基础设施 | ⭐⭐⭐⭐⭐ |
| 应用开发 | ⭐⭐⭐⭐⭐ |
| 数据保护 | ⭐⭐⭐⭐⭐ |
| 网络边界 | ⭐⭐⭐⭐⭐ |

## 2. 核心概念

### 2.1 安全基线框架

```
安全基线体系：

组织级基线
├── 安全策略
├── 风险评估
└── 合规要求
    │
    ▼
技术级基线
├── 基础设施基线 (CIS Benchmarks)
├── 应用安全基线 (OWASP ASVS)
├── 数据安全基线
└── 网络安全基线
    │
    ▼
运维级基线
├── 变更管理
├── 漏洞管理
└── 应急响应
```

### 2.2 关键安全控制

| 控制域 | 控制项 | 优先级 |
|-------|-------|-------|
| 身份管理 | 多因素认证 | 关键 |
| 访问控制 | 最小权限原则 | 关键 |
| 数据保护 | 传输和存储加密 | 关键 |
| 日志审计 | 集中日志收集 | 高 |
| 漏洞管理 | 定期扫描修复 | 高 |
| 备份恢复 | 定期备份演练 | 高 |

## 3. 技术实践

### 3.1 基础设施安全基线

```hcl
# Terraform安全基线 - AWS示例

# 启用CloudTrail审计
resource "aws_cloudtrail" "main" {
  name           = "security-trail"
  s3_bucket_name = aws_s3_bucket.logs.bucket
  
  is_multi_region_trail = true
  enable_logging        = true
  
  event_selector {
    read_write_type           = "All"
    include_management_events = true
  }
}

# S3 bucket安全基线
resource "aws_s3_bucket" "secure" {
  bucket = "my-secure-bucket"
}

resource "aws_s3_bucket_versioning" "secure" {
  bucket = aws_s3_bucket.secure.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "secure" {
  bucket = aws_s3_bucket.secure.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "aws_s3_bucket_public_access_block" "secure" {
  bucket = aws_s3_bucket.secure.id
  
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# VPC安全基线
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name = "secure-vpc"
  }
}

resource "aws_flow_log" "main" {
  vpc_id                   = aws_vpc.main.id
  traffic_type             = "ALL"
  log_destination_type     = "cloud-watch-logs"
  log_destination          = aws_cloudwatch_log_group.flow_logs.arn
  max_aggregation_interval = 60
}
```

### 3.2 应用安全基线检查

```python
#!/usr/bin/env python3
"""
应用安全基线检查工具
"""

import json
import re
from pathlib import Path
from typing import List, Dict
from dataclasses import dataclass


@dataclass
class SecurityFinding:
    rule_id: str
    severity: str  # critical, high, medium, low
    title: str
    description: str
    file_path: str
    line_number: int
    remediation: str


class SecurityBaselineChecker:
    """安全基线检查器"""
    
    RULES = {
        "SECRETS_001": {
            "title": "硬编码密钥",
            "pattern": r'(password|secret|key|token)\s*=\s*[\'"][^\'"]{8,}[\'"]',
            "severity": "critical",
            "description": "代码中检测到可能的硬编码凭证",
            "remediation": "使用密钥管理服务或环境变量"
        },
        "SQL_001": {
            "title": "SQL注入风险",
            "pattern": r'execute\s*\(.*%s.*\)',
            "severity": "critical",
            "description": "字符串拼接SQL查询",
            "remediation": "使用参数化查询"
        },
        "CRYPTO_001": {
            "title": "弱加密算法",
            "pattern": r'(md5|sha1)\s*\(',
            "severity": "high",
            "description": "使用已弃用的哈希算法",
            "remediation": "使用SHA-256或更强算法"
        },
        "HTTPS_001": {
            "title": "不安全的HTTP",
            "pattern": r'http://[^\s\'"]+',
            "severity": "medium",
            "description": "使用明文HTTP通信",
            "remediation": "使用HTTPS协议"
        },
        "DEBUG_001": {
            "title": "调试模式启用",
            "pattern": r'debug\s*=\s*True',
            "severity": "high",
            "description": "生产环境不应启用调试模式",
            "remediation": "设置debug = False"
        }
    }
    
    def __init__(self, source_path: str):
        self.source_path = Path(source_path)
        self.findings: List[SecurityFinding] = []
    
    def scan(self) -> List[SecurityFinding]:
        """扫描源代码"""
        for file_path in self.source_path.rglob("*.py"):
            self._scan_file(file_path)
        return self.findings
    
    def _scan_file(self, file_path: Path):
        """扫描单个文件"""
        try:
            content = file_path.read_text(encoding='utf-8')
            lines = content.split('\n')
            
            for rule_id, rule in self.RULES.items():
                pattern = re.compile(rule["pattern"], re.IGNORECASE)
                
                for line_num, line in enumerate(lines, 1):
                    if pattern.search(line):
                        finding = SecurityFinding(
                            rule_id=rule_id,
                            severity=rule["severity"],
                            title=rule["title"],
                            description=rule["description"],
                            file_path=str(file_path),
                            line_number=line_num,
                            remediation=rule["remediation"]
                        )
                        self.findings.append(finding)
        except Exception as e:
            print(f"Error scanning {file_path}: {e}")
    
    def generate_report(self) -> Dict:
        """生成检查报告"""
        severity_count = {"critical": 0, "high": 0, "medium": 0, "low": 0}
        for finding in self.findings:
            severity_count[finding.severity] += 1
        
        return {
            "summary": {
                "total_findings": len(self.findings),
                "severity_distribution": severity_count,
                "compliant": len(self.findings) == 0
            },
            "findings": [
                {
                    "rule_id": f.rule_id,
                    "severity": f.severity,
                    "title": f.title,
                    "file": f.file_path,
                    "line": f.line_number,
                    "description": f.description,
                    "remediation": f.remediation
                }
                for f in self.findings
            ]
        }


if __name__ == "__main__":
    checker = SecurityBaselineChecker(".")
    checker.scan()
    report = checker.generate_report()
    print(json.dumps(report, indent=2))
```

### 3.3 Docker安全基线

```dockerfile
# Dockerfile安全基线示例

# 使用最小基础镜像
FROM python:3.11-slim-bookworm

# 创建非root用户
RUN groupadd -r appgroup && useradd -r -g appgroup appuser

# 设置工作目录
WORKDIR /app

# 先复制依赖文件，利用缓存层
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 复制应用代码
COPY --chown=appuser:appgroup . .

# 移除setuid/setgid权限
RUN find / -perm /6000 -type f -exec chmod a-s {} \; 2>/dev/null || true

# 切换到非root用户
USER appuser

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD python -c "import requests; requests.get('http://localhost:8000/health')" || exit 1

# 暴露端口
EXPOSE 8000

# 使用exec形式运行
CMD ["python", "-m", "uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## 4. 资源索引

| 资源 | 链接 |
|-----|------|
| CIS Benchmarks | https://www.cisecurity.org/cis-benchmarks |
| OWASP ASVS | https://owasp.org/www-project-application-security-verification-standard |
| NIST CSF | https://www.nist.gov/cyberframework |
| Cloud Security Alliance | https://cloudsecurityalliance.org |
| AWS Security | https://aws.amazon.com/security |

## 5. 关联知识

- C01_Compliance_Framework
- C02_OSS_Licensing
- B02_Technical_Practices

## 6. 学习建议

1. 学习CIS Benchmarks
2. 掌握OWASP Top 10
3. 实践安全代码审查
4. 建立安全测试流程

---
*最后更新：2024年*
