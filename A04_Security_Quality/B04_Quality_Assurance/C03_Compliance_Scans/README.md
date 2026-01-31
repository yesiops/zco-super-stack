# C03 合规扫描 (Compliance Scans)

## 1. 主题定位

### 1.1 定义
合规扫描是指通过自动化工具检查系统、代码、配置是否符合安全标准、法规要求和行业规范的过程。它是安全治理和风险管理的重要组成部分。

### 1.2 合规框架与标准

| 标准 | 适用范围 | 重点内容 |
|------|----------|----------|
| **PCI DSS** | 支付卡行业 | 持卡人数据保护 |
| **GDPR** | 欧盟 | 个人数据保护 |
| **HIPAA** | 医疗健康 | 健康信息隐私 |
| **ISO 27001** | 通用 | 信息安全管理体系 |
| **SOC 2** | 云服务 | 安全性、可用性、处理完整性 |
| **CIS Benchmarks** | 基础设施 | 安全配置基线 |
| **OWASP ASVS** | 应用安全 | 应用安全验证标准 |
| **NIST CSF** | 美国 | 网络安全框架 |

### 1.3 核心价值
- **法规遵循**: 满足法律和行业标准要求
- **风险识别**: 发现合规性缺口
- **证据留存**: 提供审计证据
- **持续监控**: 确保持续合规

---

## 2. 核心概念

### 2.1 合规检查类型

```
合规扫描层次：

┌─────────────────────────────────────────────────────────────────┐
│  Level 4: 应用层                                                 │
│  - 代码安全扫描 (SAST)                                           │
│  - 依赖漏洞扫描 (SCA)                                            │
│  - 密钥和凭证扫描                                                │
├─────────────────────────────────────────────────────────────────┤
│  Level 3: 容器层                                                 │
│  - 镜像漏洞扫描                                                  │
│  - 容器配置审计                                                  │
│  - 运行时安全扫描                                                │
├─────────────────────────────────────────────────────────────────┤
│  Level 2: 基础设施层                                             │
│  - 云配置审计 (CSPM)                                             │
│  - 网络策略检查                                                  │
│  - 访问控制审计                                                  │
├─────────────────────────────────────────────────────────────────┤
│  Level 1: 操作系统层                                             │
│  - CIS Benchmarks 扫描                                           │
│  - 补丁管理检查                                                  │
│  - 基线配置审计                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 CIS Benchmarks 自动化检查

```python
#!/usr/bin/env python3
"""
CIS Benchmarks 自动化合规检查
"""

import subprocess
import json
import re
from typing import Dict, List, Optional
from dataclasses import dataclass
from enum import Enum

class CheckSeverity(Enum):
    CRITICAL = "严重"
    HIGH = "高"
    MEDIUM = "中"
    LOW = "低"

class CheckStatus(Enum):
    PASS = "通过"
    FAIL = "失败"
    WARNING = "警告"
    SKIP = "跳过"

@dataclass
class ComplianceCheck:
    """合规检查项"""
    id: str
    title: str
    description: str
    severity: CheckSeverity
    remediation: str
    command: str
    expected_result: str
    actual_result: Optional[str] = None
    status: CheckStatus = CheckStatus.SKIP


class CISLinuxBenchmark:
    """CIS Linux 基线检查"""
    
    CHECKS = [
        # 1. 初始设置
        ComplianceCheck(
            id="1.1.1.1",
            title="确保已挂载 cramfs 文件系统",
            description="cramfs 是一个压缩只读文件系统，不应在标准系统上使用",
            severity=CheckSeverity.LOW,
            remediation="创建 /etc/modprobe.d/cramfs.conf 添加 install cramfs /bin/true",
            command="modprobe -n -v cramfs",
            expected_result="install /bin/true"
        ),
        
        ComplianceCheck(
            id="1.3.1",
            title="确保已安装 AIDE",
            description="AIDE 是一个文件完整性检查工具",
            severity=CheckSeverity.MEDIUM,
            remediation="apt-get install aide aide-common",
            command="dpkg -s aide",
            expected_result="Status: install ok installed"
        ),
        
        # 2. 服务配置
        ComplianceCheck(
            id="2.1.1",
            title="确保时间同步服务已启用",
            description="确保系统时间准确",
            severity=CheckSeverity.MEDIUM,
            remediation="systemctl enable systemd-timesyncd",
            command="systemctl is-enabled systemd-timesyncd",
            expected_result="enabled"
        ),
        
        # 3. 网络配置
        ComplianceCheck(
            id="3.1.2",
            title="确保无线接口已禁用",
            description="如果不需要无线连接，应禁用无线接口",
            severity=CheckSeverity.MEDIUM,
            remediation="添加配置到 /etc/network/interfaces",
            command="iwconfig 2>/dev/null || echo 'No wireless extensions'",
            expected_result="No wireless extensions"
        ),
        
        # 4. 日志和审计
        ComplianceCheck(
            id="4.1.1.1",
            title="确保 auditd 已安装",
            description="auditd 提供系统审计功能",
            severity=CheckSeverity.MEDIUM,
            remediation="apt-get install auditd audispd-plugins",
            command="dpkg -s auditd",
            expected_result="Status: install ok installed"
        ),
        
        ComplianceCheck(
            id="4.1.1.2",
            title="确保 auditd 服务已启用",
            description="确保审计服务在启动时运行",
            severity=CheckSeverity.MEDIUM,
            remediation="systemctl enable auditd",
            command="systemctl is-enabled auditd",
            expected_result="enabled"
        ),
        
        # 5. 访问认证和授权
        ComplianceCheck(
            id="5.1.1",
            title="确保 cron 守护进程已启用",
            description="cron 用于定期执行命令",
            severity=CheckSeverity.MEDIUM,
            remediation="systemctl enable cron",
            command="systemctl is-enabled cron",
            expected_result="enabled"
        ),
        
        ComplianceCheck(
            id="5.2.1",
            title="确保 sudo 已安装",
            description="sudo 允许用户以其他用户身份执行命令",
            severity=CheckSeverity.MEDIUM,
            remediation="apt-get install sudo",
            command="dpkg -s sudo",
            expected_result="Status: install ok installed"
        ),
        
        ComplianceCheck(
            id="5.3.1",
            title="确保密码复杂度要求已配置",
            description="确保密码满足复杂度要求",
            severity=CheckSeverity.HIGH,
            remediation="配置 /etc/security/pwquality.conf",
            command="grep 'minlen' /etc/security/pwquality.conf",
            expected_result="minlen = 14"
        ),
        
        ComplianceCheck(
            id="5.4.1.1",
            title="确保密码过期策略已配置",
            description="设置密码最大使用期限",
            severity=CheckSeverity.MEDIUM,
            remediation="编辑 /etc/login.defs 设置 PASS_MAX_DAYS",
            command="grep PASS_MAX_DAYS /etc/login.defs | grep -v '#'",
            expected_result="PASS_MAX_DAYS 90"
        ),
        
        # 6. 系统维护
        ComplianceCheck(
            id="6.1.1",
            title="确保审计系统文件权限",
            description="确保 /etc/passwd 等文件权限正确",
            severity=CheckSeverity.HIGH,
            remediation="chmod 644 /etc/passwd",
            command="stat -c '%a' /etc/passwd",
            expected_result="644"
        ),
        
        ComplianceCheck(
            id="6.2.1",
            title="确保没有空密码账户",
            description="空密码账户是严重安全风险",
            severity=CheckSeverity.CRITICAL,
            remediation="为所有账户设置密码或删除账户",
            command="awk -F: '($2 == \"\") {print}' /etc/shadow",
            expected_result=""  # 应该为空
        ),
    ]
    
    def __init__(self):
        self.results: List[ComplianceCheck] = []
    
    def run_check(self, check: ComplianceCheck) -> ComplianceCheck:
        """执行单个检查"""
        print(f"  检查 {check.id}: {check.title[:40]}...", end=' ')
        
        try:
            result = subprocess.run(
                check.command,
                shell=True,
                capture_output=True,
                text=True,
                timeout=30
            )
            
            actual_output = result.stdout.strip() + result.stderr.strip()
            check.actual_result = actual_output[:200]  # 限制长度
            
            # 检查结果
            if check.expected_result == "":
                # 期望空结果
                check.status = CheckStatus.PASS if not actual_output else CheckStatus.FAIL
            elif check.expected_result.lower() in actual_output.lower():
                check.status = CheckStatus.PASS
            else:
                check.status = CheckStatus.FAIL
                
        except subprocess.TimeoutExpired:
            check.status = CheckStatus.WARNING
            check.actual_result = "命令超时"
        except Exception as e:
            check.status = CheckStatus.WARNING
            check.actual_result = str(e)
        
        status_icon = "✓" if check.status == CheckStatus.PASS else "✗" if check.status == CheckStatus.FAIL else "⚠"
        print(status_icon)
        
        return check
    
    def run_all_checks(self) -> Dict:
        """执行所有检查"""
        print("=" * 60)
        print("CIS Linux Benchmark 合规检查")
        print("=" * 60)
        
        for check in self.CHECKS:
            result = self.run_check(check)
            self.results.append(result)
        
        return self.generate_report()
    
    def generate_report(self) -> Dict:
        """生成检查报告"""
        total = len(self.results)
        passed = len([r for r in self.results if r.status == CheckStatus.PASS])
        failed = len([r for r in self.results if r.status == CheckStatus.FAIL])
        warnings = len([r for r in self.results if r.status == CheckStatus.WARNING])
        
        # 按严重度分组
        by_severity = {}
        for r in self.results:
            if r.status == CheckStatus.FAIL:
                sev = r.severity.value
                if sev not in by_severity:
                    by_severity[sev] = []
                by_severity[sev].append(r)
        
        report = {
            'summary': {
                'total': total,
                'passed': passed,
                'failed': failed,
                'warnings': warnings,
                'compliance_rate': f"{passed/total*100:.1f}%" if total > 0 else "N/A"
            },
            'by_severity': {k: len(v) for k, v in by_severity.items()},
            'failed_checks': [
                {
                    'id': r.id,
                    'title': r.title,
                    'severity': r.severity.value,
                    'remediation': r.remediation,
                    'expected': r.expected_result,
                    'actual': r.actual_result
                }
                for r in self.results if r.status == CheckStatus.FAIL
            ]
        }
        
        return report


def demo_cis_scan():
    """演示 CIS 扫描"""
    scanner = CISLinuxBenchmark()
    report = scanner.run_all_checks()
    
    print("\n" + "=" * 60)
    print("检查结果摘要")
    print("=" * 60)
    print(f"总检查项: {report['summary']['total']}")
    print(f"通过: {report['summary']['passed']}")
    print(f"失败: {report['summary']['failed']}")
    print(f"警告: {report['summary']['warnings']}")
    print(f"合规率: {report['summary']['compliance_rate']}")
    
    if report['failed_checks']:
        print("\n失败项详情:")
        for check in report['failed_checks'][:5]:  # 只显示前5个
            print(f"\n[{check['id']}] {check['title']}")
            print(f"  严重度: {check['severity']}")
            print(f"  修复建议: {check['remediation']}")


if __name__ == '__main__':
    demo_cis_scan()
```

### 2.3 云安全态势管理 (CSPM)

```python
#!/usr/bin/env python3
"""
云安全配置审计 (CSPM)
支持 AWS、Azure、阿里云等
"""

from typing import Dict, List, Optional
from dataclasses import dataclass
from enum import Enum

class CloudProvider(Enum):
    AWS = "aws"
    AZURE = "azure"
    GCP = "gcp"
    ALIBABA = "alibaba"

class RiskLevel(Enum):
    CRITICAL = "严重"
    HIGH = "高"
    MEDIUM = "中"
    LOW = "低"

@dataclass
class CloudFinding:
    """云安全发现"""
    provider: CloudProvider
    service: str
    resource_id: str
    finding_type: str
    risk_level: RiskLevel
    title: str
    description: str
    remediation: str
    compliance_frameworks: List[str]


class CloudSecurityScanner:
    """云安全配置扫描器"""
    
    # AWS 安全规则
    AWS_RULES = [
        {
            'id': 'AWS-001',
            'title': 'S3 Bucket 公开访问',
            'service': 'S3',
            'risk': RiskLevel.CRITICAL,
            'check': 'bucket.public_access == True',
            'remediation': '禁用 S3 Bucket 的公开访问',
            'frameworks': ['CIS-AWS', 'PCI-DSS']
        },
        {
            'id': 'AWS-002',
            'title': '未加密的 RDS 实例',
            'service': 'RDS',
            'risk': RiskLevel.HIGH,
            'check': 'instance.encrypted == False',
            'remediation': '启用 RDS 加密',
            'frameworks': ['CIS-AWS', 'HIPAA']
        },
        {
            'id': 'AWS-003',
            'title': '过度宽松的 Security Group',
            'service': 'EC2',
            'risk': RiskLevel.HIGH,
            'check': 'sg.allows_port(22) and sg.source == 0.0.0.0/0',
            'remediation': '限制 SSH 访问来源',
            'frameworks': ['CIS-AWS']
        },
        {
            'id': 'AWS-004',
            'title': '未启用 MFA 的 Root 账户',
            'service': 'IAM',
            'risk': RiskLevel.CRITICAL,
            'check': 'root_account.mfa_enabled == False',
            'remediation': '为 Root 账户启用 MFA',
            'frameworks': ['CIS-AWS', 'PCI-DSS', 'SOC2']
        },
        {
            'id': 'AWS-005',
            'title': '过期的 IAM Access Key',
            'service': 'IAM',
            'risk': RiskLevel.MEDIUM,
            'check': 'key.age_days > 90',
            'remediation': '轮换 IAM Access Key',
            'frameworks': ['CIS-AWS']
        },
    ]
    
    # 阿里云安全规则
    ALIBABA_RULES = [
        {
            'id': 'ALI-001',
            'title': 'OSS Bucket 公开访问',
            'service': 'OSS',
            'risk': RiskLevel.CRITICAL,
            'check': 'bucket.acl == public-read',
            'remediation': '设置 OSS Bucket 为私有',
            'frameworks': ['等保2.0']
        },
        {
            'id': 'ALI-002',
            'title': '安全组开放高危端口',
            'service': 'ECS',
            'risk': RiskLevel.HIGH,
            'check': 'sg.allows_port([22, 3389]) and sg.source == 0.0.0.0/0',
            'remediation': '限制安全组规则',
            'frameworks': ['等保2.0']
        },
    ]
    
    def __init__(self, provider: CloudProvider):
        self.provider = provider
        self.findings: List[CloudFinding] = []
    
    def scan(self, resources: List[Dict]) -> List[CloudFinding]:
        """执行扫描"""
        rules = self._get_rules()
        
        for resource in resources:
            for rule in rules:
                if self._evaluate_rule(rule, resource):
                    finding = CloudFinding(
                        provider=self.provider,
                        service=rule['service'],
                        resource_id=resource.get('id', 'unknown'),
                        finding_type=rule['id'],
                        risk_level=rule['risk'],
                        title=rule['title'],
                        description=f"资源 {resource.get('id')} 违反规则 {rule['id']}",
                        remediation=rule['remediation'],
                        compliance_frameworks=rule['frameworks']
                    )
                    self.findings.append(finding)
        
        return self.findings
    
    def _get_rules(self) -> List[Dict]:
        """获取规则列表"""
        if self.provider == CloudProvider.AWS:
            return self.AWS_RULES
        elif self.provider == CloudProvider.ALIBABA:
            return self.ALIBABA_RULES
        return []
    
    def _evaluate_rule(self, rule: Dict, resource: Dict) -> bool:
        """评估规则 (简化实现)"""
        # 实际实现中需要解析和执行规则表达式
        return False  # 占位
    
    def generate_report(self) -> Dict:
        """生成报告"""
        by_level = {}
        for f in self.findings:
            level = f.risk_level.value
            if level not in by_level:
                by_level[level] = []
            by_level[level].append(f)
        
        return {
            'provider': self.provider.value,
            'total_findings': len(self.findings),
            'by_severity': {k: len(v) for k, v in by_level.items()},
            'findings': [
                {
                    'id': f.finding_type,
                    'title': f.title,
                    'service': f.service,
                    'resource': f.resource_id,
                    'severity': f.risk_level.value,
                    'remediation': f.remediation
                }
                for f in self.findings
            ]
        }


def demo_cspm():
    """演示 CSPM"""
    print("=" * 60)
    print("云安全配置审计 (CSPM)")
    print("=" * 60)
    
    # AWS 扫描
    scanner_aws = CloudSecurityScanner(CloudProvider.AWS)
    
    # 模拟 AWS 资源
    aws_resources = [
        {'id': 's3-bucket-1', 'type': 'S3', 'public_access': True},
        {'id': 'sg-12345', 'type': 'SecurityGroup', 'allows_port': [22], 'source': '0.0.0.0/0'},
    ]
    
    findings = scanner_aws.scan(aws_resources)
    report = scanner_aws.generate_report()
    
    print(f"\nAWS 扫描结果:")
    print(f"  发现问题: {report['total_findings']}")
    
    # 阿里云扫描
    scanner_ali = CloudSecurityScanner(CloudProvider.ALIBABA)
    print(f"\n阿里云扫描就绪，规则数: {len(scanner_ali.ALI BABA_RULES)}")


if __name__ == '__main__':
    demo_cspm()
```

---

## 3. 技术实践

### 3.1 代码安全扫描

```python
#!/usr/bin/env python3
"""
代码安全扫描工具
检测硬编码凭证、安全漏洞等
"""

import re
import ast
import os
from typing import List, Dict, Optional
from dataclasses import dataclass
from pathlib import Path

@dataclass
class SecurityFinding:
    """安全发现"""
    file_path: str
    line_number: int
    rule_id: str
    severity: str
    title: str
    description: str
    match_text: str
    remediation: str


class SecretScanner:
    """凭证扫描器"""
    
    PATTERNS = [
        {
            'id': 'SECRET-001',
            'name': 'AWS Access Key ID',
            'pattern': r'AKIA[0-9A-Z]{16}',
            'severity': 'CRITICAL',
            'description': 'AWS Access Key ID 泄露'
        },
        {
            'id': 'SECRET-002',
            'name': 'AWS Secret Access Key',
            'pattern': r'["\']?[a-zA-Z0-9/+=]{40}["\']?',
            'severity': 'CRITICAL',
            'description': '可能的 AWS Secret Key'
        },
        {
            'id': 'SECRET-003',
            'name': 'Private Key',
            'pattern': r'-----BEGIN (RSA |DSA |EC |OPENSSH )?PRIVATE KEY-----',
            'severity': 'CRITICAL',
            'description': '私钥文件泄露'
        },
        {
            'id': 'SECRET-004',
            'name': 'Password in Code',
            'pattern': r'(?i)(password|passwd|pwd)\s*=\s*["\'][^"\']+["\']',
            'severity': 'HIGH',
            'description': '代码中硬编码密码'
        },
        {
            'id': 'SECRET-005',
            'name': 'API Key',
            'pattern': r'(?i)(api[_-]?key|apikey)\s*[:=]\s*["\'][^"\']{16,}["\']',
            'severity': 'HIGH',
            'description': 'API Key 泄露'
        },
        {
            'id': 'SECRET-006',
            'name': 'Database Connection String',
            'pattern': r'(?i)(mongodb|mysql|postgresql|redis)://[^\s"\']+',
            'severity': 'HIGH',
            'description': '数据库连接字符串泄露'
        },
    ]
    
    def scan_file(self, file_path: str) -> List[SecurityFinding]:
        """扫描单个文件"""
        findings = []
        
        try:
            with open(file_path, 'r', encoding='utf-8', errors='ignore') as f:
                content = f.read()
                lines = content.split('\n')
        except:
            return findings
        
        for pattern_def in self.PATTERNS:
            regex = re.compile(pattern_def['pattern'])
            
            for line_num, line in enumerate(lines, 1):
                for match in regex.finditer(line):
                    # 排除注释行
                    if line.strip().startswith('#'):
                        continue
                    
                    finding = SecurityFinding(
                        file_path=file_path,
                        line_number=line_num,
                        rule_id=pattern_def['id'],
                        severity=pattern_def['severity'],
                        title=pattern_def['name'],
                        description=pattern_def['description'],
                        match_text=match.group()[:50],
                        remediation='将凭证移至环境变量或密钥管理服务'
                    )
                    findings.append(finding)
        
        return findings
    
    def scan_directory(self, directory: str, 
                      extensions: List[str] = None) -> List[SecurityFinding]:
        """扫描目录"""
        if extensions is None:
            extensions = ['.py', '.js', '.java', '.go', '.yml', '.yaml', 
                         '.json', '.xml', '.properties', '.env']
        
        all_findings = []
        
        for root, dirs, files in os.walk(directory):
            # 跳过特定目录
            dirs[:] = [d for d in dirs if d not in ['.git', 'node_modules', 
                                                     'venv', '__pycache__']]
            
            for file in files:
                if any(file.endswith(ext) for ext in extensions):
                    file_path = os.path.join(root, file)
                    findings = self.scan_file(file_path)
                    all_findings.extend(findings)
        
        return all_findings


class VulnerabilityScanner:
    """漏洞扫描器 - 检测不安全的代码模式"""
    
    VULNERABILITIES = [
        {
            'id': 'VULN-001',
            'name': 'SQL Injection',
            'pattern': r'execute\s*\(\s*["\'].*%s',
            'severity': 'CRITICAL',
            'description': '可能存在 SQL 注入漏洞'
        },
        {
            'id': 'VULN-002',
            'name': 'Eval Usage',
            'pattern': r'\beval\s*\(',
            'severity': 'HIGH',
            'description': '使用 eval 可能存在代码注入风险'
        },
        {
            'id': 'VULN-003',
            'name': 'Pickle Usage',
            'pattern': r'pickle\.(loads|load)\s*\(',
            'severity': 'HIGH',
            'description': 'pickle 反序列化可能存在安全风险'
        },
        {
            'id': 'VULN-004',
            'name': 'Debug Mode Enabled',
            'pattern': r'DEBUG\s*=\s*True',
            'severity': 'MEDIUM',
            'description': '调试模式可能在生产环境启用'
        },
    ]


def demo_secret_scan():
    """演示凭证扫描"""
    print("=" * 60)
    print("代码安全扫描")
    print("=" * 60)
    
    # 创建测试文件
    import tempfile
    test_code = '''
# 这是一个测试文件
DATABASE_URL = "postgresql://user:password123@localhost/db"

# AWS 凭证 (假的)
AWS_ACCESS_KEY_ID = "{AWS_ACCESS_KEY_ID}"
AWS_SECRET_ACCESS_KEY = "{AWS_SECRET_ACCESS_KEY}"

# API Key
api_key = "{API_KEY}"

def connect():
    password = "hardcoded_password"
    return password
'''
    
    with tempfile.NamedTemporaryFile(mode='w', suffix='.py', delete=False) as f:
        f.write(test_code)
        temp_file = f.name
    
    try:
        scanner = SecretScanner()
        findings = scanner.scan_file(temp_file)
        
        print(f"\n发现 {len(findings)} 个安全问题:\n")
        
        for f in findings:
            print(f"[{f.severity}] {f.title}")
            print(f"  文件: {f.file_path}:{f.line_number}")
            print(f"  描述: {f.description}")
            print(f"  匹配: {f.match_text}")
            print(f"  修复: {f.remediation}\n")
    finally:
        os.unlink(temp_file)


if __name__ == '__main__':
    demo_secret_scan()
```

---

## 4. 资源索引

### 4.1 工具推荐

| 工具 | 用途 | 链接 |
|------|------|------|
| OpenSCAP | 合规扫描 | https://www.open-scap.org/ |
| ScoutSuite | 多云安全审计 | https://github.com/nccgroup/ScoutSuite |
| Prowler | AWS 安全工具 | https://github.com/prowler-cloud/prowler |
| CloudSploit | 云安全扫描 | https://cloudsploit.com/ |
| Terrascan | IaC 安全扫描 | https://runterrascan.io/ |
| Trivy | 容器漏洞扫描 | https://trivy.dev/ |
| Checkov | IaC 合规 | https://www.checkov.io/ |

### 4.2 合规框架

| 框架 | 描述 | 链接 |
|------|------|------|
| CIS Benchmarks | 安全配置基线 | https://www.cisecurity.org/cis-benchmarks |
| NIST 800-53 | 安全和隐私控制 | https://csrc.nist.gov/publications/detail/sp/800-53/final |
| CSA CCM | 云控制矩阵 | https://cloudsecurityalliance.org/research/cloud-controls-matrix/ |

---

## 5. 关联知识

```
A04_Security_Quality/B04_Quality_Assurance/
├── C03_Compliance_Scans (本模块)
│   ├── → C01_Test_Automation: 扫描自动化
│   └── → C02_Fuzzing_Techniques: 安全测试
│
├── → B01_Information_Security: 安全合规
└── → B03_Maintenance_Ops: 合规运维
```

---

*最后更新: 2024-01-30*
