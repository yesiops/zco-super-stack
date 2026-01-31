# C02_OSS_Licensing - 开源许可

## 1. 主题定位

### 1.1 定义与范围

开源许可（OSS Licensing）是管理软件项目中开源组件使用的合规实践，涵盖许可证识别、合规审查、许可证冲突检测、SBOM生成等内容。确保组织在使用开源软件时遵守相应的法律条款。

### 1.2 业务价值

- 规避法律风险
- 保护知识产权
- 确保供应链安全
- 满足合规审计
- 促进健康开源生态

### 1.3 适用场景

| 场景 | 适用性 |
|-----|-------|
| 商业软件开发 | ⭐⭐⭐⭐⭐ |
| 开源项目维护 | ⭐⭐⭐⭐⭐ |
| 供应商管理 | ⭐⭐⭐⭐⭐ |
| 企业采购 | ⭐⭐⭐⭐⭐ |

## 2. 核心概念

### 2.1 主要开源许可证

| 许可证 | 类型 | 商业使用 | 修改 | 专利授权 |
|-------|-----|---------|------|---------|
| MIT | 宽松 | ✅ | ✅ | ❌ |
| Apache 2.0 | 宽松 | ✅ | ✅ | ✅ |
| BSD | 宽松 | ✅ | ✅ | ❌ |
| GPL-2.0 | 强Copyleft | ✅ | 需开源 | ❌ |
| GPL-3.0 | 强Copyleft | ✅ | 需开源 | ✅ |
| LGPL | 弱Copyleft | ✅ | 有限制 | ❌ |
| AGPL | 超强Copyleft | ✅ | 需开源 | ✅ |
| MPL | 弱Copyleft | ✅ | 有限制 | ❌ |

### 2.2 许可证兼容性矩阵

```
          MIT  Apache  BSD  LGPL  GPL2  GPL3  AGPL
MIT        ✅    ✅    ✅    ✅    ✅    ✅    ❌
Apache     ✅    ✅    ✅    ✅    ✅    ✅    ❌
BSD        ✅    ✅    ✅    ✅    ✅    ✅    ❌
LGPL       ✅    ✅    ✅    ✅    ✅    ✅    ❌
GPL2       ❌    ❌    ❌    ✅    ✅    ❌    ❌
GPL3       ❌    ❌    ❌    ✅    ❌    ✅    ❌
AGPL       ❌    ❌    ❌    ❌    ❌    ❌    ✅
```

## 3. 技术实践

### 3.1 许可证扫描工具

```bash
#!/bin/bash
# license_scan.sh - 开源许可证扫描脚本

# 使用FOSSA扫描
echo "Running FOSSA analysis..."
fossa analyze

# 使用ScanCode扫描
echo "Running ScanCode Toolkit..."
scancode --license --copyright --json scan_results.json .

# 使用LicenseFinder
echo "Running LicenseFinder..."
license_finder
```

### 3.2 Python许可证检查

```python
#!/usr/bin/env python3
"""
开源许可证合规检查工具
"""

import subprocess
import json
from dataclasses import dataclass
from typing import List, Dict, Set


@dataclass
class LicenseInfo:
    package: str
    version: str
    license: str
    license_type: str  # permissive, weak_copyleft, strong_copyleft
    is_compliant: bool


class LicenseChecker:
    """许可证检查器"""
    
    # 许可证分类
    PERMISSIVE = {"MIT", "Apache-2.0", "BSD-2-Clause", "BSD-3-Clause", "ISC"}
    WEAK_COPYLEFT = {"LGPL-2.1", "LGPL-3.0", "MPL-2.0"}
    STRONG_COPYLEFT = {"GPL-2.0", "GPL-3.0", "AGPL-3.0"}
    
    # 组织允许列表
    ALLOWED_LICENSES = PERMISSIVE | WEAK_COPYLEFT
    FORBIDDEN_LICENSES = STRONG_COPYLEFT
    
    def __init__(self, project_path: str):
        self.project_path = project_path
    
    def scan_python_deps(self) -> List[LicenseInfo]:
        """扫描Python依赖"""
        # 使用pip-licenses获取许可证信息
        result = subprocess.run(
            ["pip-licenses", "--format=json"],
            capture_output=True,
            text=True,
            cwd=self.project_path
        )
        
        packages = json.loads(result.stdout)
        license_infos = []
        
        for pkg in packages:
            license_name = pkg.get("License", "Unknown")
            license_type = self._classify_license(license_name)
            
            info = LicenseInfo(
                package=pkg["Name"],
                version=pkg["Version"],
                license=license_name,
                license_type=license_type,
                is_compliant=license_name in self.ALLOWED_LICENSES
            )
            license_infos.append(info)
        
        return license_infos
    
    def _classify_license(self, license_name: str) -> str:
        """分类许可证"""
        if license_name in self.PERMISSIVE:
            return "permissive"
        elif license_name in self.WEAK_COPYLEFT:
            return "weak_copyleft"
        elif license_name in self.STRONG_COPYLEFT:
            return "strong_copyleft"
        return "unknown"
    
    def generate_sbom(self) -> Dict:
        """生成SBOM (Software Bill of Materials)"""
        licenses = self.scan_python_deps()
        
        sbom = {
            "specVersion": "1.4",
            "bomFormat": "CycloneDX",
            "components": []
        }
        
        for lic in licenses:
            sbom["components"].append({
                "name": lic.package,
                "version": lic.version,
                "licenses": [{"license": {"id": lic.license}}],
                "purl": f"pkg:pypi/{lic.package}@{lic.version}"
            })
        
        return sbom
    
    def check_compliance(self) -> Dict:
        """检查合规性"""
        licenses = self.scan_python_deps()
        
        violations = [l for l in licenses if not l.is_compliant]
        unknown = [l for l in licenses if l.license_type == "unknown"]
        
        return {
            "total_packages": len(licenses),
            "compliant": len(licenses) - len(violations) - len(unknown),
            "violations": len(violations),
            "unknown": len(unknown),
            "violation_details": [
                {
                    "package": v.package,
                    "license": v.license,
                    "reason": f"{v.license} is not in allowed list"
                }
                for v in violations
            ]
        }


if __name__ == "__main__":
    checker = LicenseChecker(".")
    
    # 生成合规报告
    report = checker.check_compliance()
    print(json.dumps(report, indent=2))
    
    # 生成SBOM
    sbom = checker.generate_sbom()
    with open("sbom.json", "w") as f:
        json.dump(sbom, f, indent=2)
```

### 3.3 CI/CD集成

```yaml
# .github/workflows/license-check.yml
name: License Compliance

on: [push, pull_request]

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: |
          pip install pip-licenses
          pip install -r requirements.txt
      
      - name: Check licenses
        run: python scripts/license_check.py
      
      - name: Run FOSSA scan
        uses: fossas/fossa-action@main
        with:
          api-key: ${{ secrets.FOSSA_API_KEY }}
      
      - name: Generate SBOM
        run: |
          pip install cyclonedx-bom
          cyclonedx-py -o sbom.json
      
      - name: Upload SBOM
        uses: actions/upload-artifact@v4
        with:
          name: sbom
          path: sbom.json
```

## 4. 资源索引

| 资源 | 链接 |
|-----|------|
| OSI | https://opensource.org/licenses |
| SPDX | https://spdx.org/licenses |
| FOSSA | https://fossa.com |
| Snyk | https://snyk.io |
| ChooseALicense | https://choosealicense.com |

## 5. 关联知识

- C01_Compliance_Framework
- C03_Security_Baselines
- B02_Technical_Practices

## 6. 学习建议

1. 学习常见许可证条款
2. 建立许可证审查流程
3. 使用自动化扫描工具
4. 定期审计依赖

---
*最后更新：2024年*
