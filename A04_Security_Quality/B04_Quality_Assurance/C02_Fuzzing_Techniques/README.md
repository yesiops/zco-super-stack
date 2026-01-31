# C02 模糊测试 (Fuzzing Techniques)

## 1. 主题定位

### 1.1 定义
模糊测试（Fuzzing）是一种自动化软件测试技术，通过向程序输入大量随机、畸形或异常的数据，来发现程序中的崩溃、内存泄露、断言失败等安全漏洞和稳定性问题。

### 1.2 核心价值
- **漏洞发现**: 发现常规测试难以发现的边界问题
- **自动化**: 高度自动化的安全测试
- **深度覆盖**: 探索代码中的异常处理路径
- **零误报**: 发现的都是真实可触发的问题

### 1.3 模糊测试分类

```
模糊测试类型：

┌──────────────────────────────────────────────────────────────────┐
│                                                                  │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐       │
│  │   黑盒模糊   │    │   灰盒模糊   │    │   白盒模糊   │       │
│  │   测试       │    │   测试       │    │   测试       │       │
│  └──────┬───────┘    └──────┬───────┘    └──────┬───────┘       │
│         │                   │                   │                │
│  无程序知识             部分程序知识          完整程序知识        │
│  纯随机输入             代码覆盖反馈          符号执行            │
│                        (AFL, libFuzzer)    (KLEE, SAGE)         │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. 核心概念

### 2.1 模糊测试流程

```
模糊测试工作流程：

┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  种子输入   │────>│  变异引擎   │────>│  生成输入   │
│  (Corpus)   │     │ (Mutator)   │     │             │
└─────────────┘     └─────────────┘     └──────┬──────┘
                                               │
                                               v
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  保存 crash │<────│  监控反馈   │<────│  目标程序   │
│  & coverage │     │ (Coverage)  │     │  (Target)   │
└─────────────┘     └─────────────┘     └─────────────┘
```

### 2.2 变异策略

```python
#!/usr/bin/env python3
"""
模糊测试变异策略实现
"""

import random
import struct
from typing import List, Callable, Any
from dataclasses import dataclass


@dataclass
class Mutation:
    """变异操作"""
    name: str
    func: Callable[[bytes], bytes]


class Mutator:
    """数据变异器"""
    
    def __init__(self):
        self.mutations: List[Mutation] = [
            Mutation("bit_flip", self.bit_flip),
            Mutation("byte_flip", self.byte_flip),
            Mutation("arithmetic", self.arithmetic_mutate),
            Mutation("interesting_values", self.interesting_values),
            Mutation("delete_bytes", self.delete_bytes),
            Mutation("insert_bytes", self.insert_bytes),
            Mutation("shuffle_bytes", self.shuffle_bytes),
        ]
    
    def mutate(self, data: bytes, count: int = 1) -> bytes:
        """对数据进行变异"""
        result = bytearray(data)
        
        for _ in range(count):
            mutation = random.choice(self.mutations)
            result = bytearray(mutation.func(bytes(result)))
        
        return bytes(result)
    
    def bit_flip(self, data: bytes) -> bytes:
        """位翻转"""
        if not data:
            return data
        
        result = bytearray(data)
        # 随机翻转一个位
        byte_idx = random.randint(0, len(result) - 1)
        bit_idx = random.randint(0, 7)
        result[byte_idx] ^= (1 << bit_idx)
        
        return bytes(result)
    
    def byte_flip(self, data: bytes) -> bytes:
        """字节翻转"""
        if not data:
            return data
        
        result = bytearray(data)
        byte_idx = random.randint(0, len(result) - 1)
        result[byte_idx] ^= 0xFF
        
        return bytes(result)
    
    def arithmetic_mutate(self, data: bytes) -> bytes:
        """算术变异"""
        if len(data) < 2:
            return data
        
        result = bytearray(data)
        byte_idx = random.randint(0, len(result) - 2)
        
        # 读取16位整数
        val = struct.unpack('<H', result[byte_idx:byte_idx+2])[0]
        
        # 随机加减
        delta = random.choice([-128, -64, -32, -16, -8, -1, 1, 8, 16, 32, 64, 128])
        val = (val + delta) & 0xFFFF
        
        # 写回
        result[byte_idx:byte_idx+2] = struct.pack('<H', val)
        
        return bytes(result)
    
    def interesting_values(self, data: bytes) -> bytes:
        """替换为有趣值"""
        if not data:
            return data
        
        interesting_8 = [0, 1, 16, 32, 64, 100, 127, 128, 255, 255]
        interesting_16 = [0, 1, 256, 1024, 4096, 32767, 32768, 65535]
        interesting_32 = [0, 1, 65536, 1000000, 2147483647, 4294967295]
        
        result = bytearray(data)
        byte_idx = random.randint(0, len(result) - 1)
        
        # 随机选择变异宽度
        choice = random.random()
        if choice < 0.5 and byte_idx < len(result):
            # 8-bit
            result[byte_idx] = random.choice(interesting_8) & 0xFF
        elif choice < 0.8 and byte_idx + 1 < len(result):
            # 16-bit
            val = random.choice(interesting_16)
            result[byte_idx:byte_idx+2] = struct.pack('<H', val)
        elif byte_idx + 3 < len(result):
            # 32-bit
            val = random.choice(interesting_32)
            result[byte_idx:byte_idx+4] = struct.pack('<I', val)
        
        return bytes(result)
    
    def delete_bytes(self, data: bytes) -> bytes:
        """删除随机字节"""
        if len(data) <= 1:
            return data
        
        result = bytearray(data)
        start = random.randint(0, len(result) - 1)
        length = random.randint(1, min(16, len(result) - start))
        del result[start:start+length]
        
        return bytes(result)
    
    def insert_bytes(self, data: bytes) -> bytes:
        """插入随机字节"""
        result = bytearray(data)
        pos = random.randint(0, len(result))
        length = random.randint(1, 16)
        new_bytes = bytes(random.randint(0, 255) for _ in range(length))
        result[pos:pos] = new_bytes
        
        return bytes(result)
    
    def shuffle_bytes(self, data: bytes) -> bytes:
        """打乱字节顺序"""
        if len(data) < 4:
            return data
        
        result = bytearray(data)
        start = random.randint(0, len(result) - 4)
        chunk = result[start:start+4]
        random.shuffle(chunk)
        result[start:start+4] = chunk
        
        return bytes(result)


class StructureAwareMutator(Mutator):
    """结构感知变异器 - 理解数据格式的变异"""
    
    def __init__(self, data_format: str = "json"):
        super().__init__()
        self.data_format = data_format
    
    def mutate_json(self, json_data: str) -> str:
        """针对 JSON 的变异"""
        import json
        
        try:
            data = json.loads(json_data)
        except:
            return json_data
        
        # 随机变异
        if isinstance(data, dict):
            # 变异键或值
            if data and random.random() < 0.5:
                key = random.choice(list(data.keys()))
                if isinstance(data[key], (int, float)):
                    data[key] = random.choice([0, -1, 999999999, float('inf')])
                elif isinstance(data[key], str):
                    data[key] = random.choice(["", "A" * 10000, "null", "undefined"])
                elif isinstance(data[key], list):
                    data[key] = data[key] * 100  # 大量重复
            else:
                # 添加嵌套
                data['__fuzz__'] = {"nested": [1, 2, 3] * 100}
        
        return json.dumps(data)
    
    def mutate(self, data: bytes, count: int = 1) -> bytes:
        """根据格式选择变异策略"""
        if self.data_format == "json":
            try:
                result = data.decode('utf-8')
                for _ in range(count):
                    result = self.mutate_json(result)
                return result.encode('utf-8')
            except:
                pass
        
        # 回退到通用变异
        return super().mutate(data, count)


def demo_mutator():
    """演示变异器"""
    print("=" * 60)
    print("模糊测试变异策略演示")
    print("=" * 60)
    
    mutator = Mutator()
    
    # 原始数据
    original = b"Hello, World! 12345"
    print(f"\n原始数据: {original}")
    print(f"十六进制: {original.hex()}")
    
    # 各种变异
    mutations = [
        ("位翻转", mutator.bit_flip),
        ("字节翻转", mutator.byte_flip),
        ("算术变异", mutator.arithmetic_mutate),
        ("有趣值", mutator.interesting_values),
        ("删除字节", mutator.delete_bytes),
        ("插入字节", mutator.insert_bytes),
    ]
    
    print("\n--- 变异结果 ---")
    for name, func in mutations:
        result = func(original)
        print(f"\n{name}:")
        print(f"  结果: {result}")
        print(f"  十六进制: {result.hex()}")
    
    # JSON 结构感知变异
    print("\n--- JSON 结构感知变异 ---")
    json_mutator = StructureAwareMutator("json")
    json_data = b'{"name": "test", "age": 25, "items": [1, 2, 3]}'
    print(f"原始 JSON: {json_data.decode()}")
    
    for i in range(3):
        mutated = json_mutator.mutate(json_data)
        print(f"变异 {i+1}: {mutated.decode()}")


if __name__ == '__main__':
    demo_mutator()
```

### 2.3 覆盖率引导模糊测试

```python
#!/usr/bin/env python3
"""
覆盖率引导的模糊测试 (Coverage-Guided Fuzzing)
简化实现，展示核心概念
"""

import os
import sys
import time
import signal
import hashlib
from pathlib import Path
from typing import Set, List, Dict
from dataclasses import dataclass
from collections import defaultdict


@dataclass
class FuzzingStats:
    """模糊测试统计"""
    execs: int = 0
    crashes: int = 0
    unique_crashes: int = 0
    corpus_size: int = 0
    coverage: int = 0
    start_time: float = 0
    
    @property
    def execs_per_sec(self) -> float:
        elapsed = time.time() - self.start_time
        return self.execs / elapsed if elapsed > 0 else 0


class SimpleCoverageTracker:
    """简化版覆盖率追踪器"""
    
    def __init__(self):
        self.covered_edges: Set[int] = set()
        self.corpus_coverage: Dict[str, Set[int]] = {}
    
    def trace_edge(self, edge_id: int):
        """记录执行的边"""
        self.covered_edges.add(edge_id)
    
    def get_coverage(self) -> int:
        """获取覆盖率计数"""
        return len(self.covered_edges)
    
    def has_new_coverage(self) -> bool:
        """检查是否有新的覆盖"""
        # 简化实现
        return True


class SimpleFuzzer:
    """简化版覆盖率引导模糊器"""
    
    def __init__(self, target_func, seed_corpus: List[bytes], 
                 output_dir: str = "./fuzzing_output"):
        self.target = target_func
        self.corpus: List[bytes] = seed_corpus.copy()
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(exist_ok=True)
        
        self.crashes_dir = self.output_dir / "crashes"
        self.crashes_dir.mkdir(exist_ok=True)
        
        self.corpus_dir = self.output_dir / "corpus"
        self.corpus_dir.mkdir(exist_ok=True)
        
        self.stats = FuzzingStats(start_time=time.time())
        self.unique_crashes: Set[str] = set()
        
        self.mutator = None  # 将导入 Mutator
        
        self.running = True
        signal.signal(signal.SIGINT, self._signal_handler)
    
    def _signal_handler(self, signum, frame):
        """处理中断信号"""
        print("\n[!] 收到中断信号，正在停止...")
        self.running = False
    
    def run(self, max_iterations: int = None):
        """运行模糊测试"""
        from fuzzing_techniques import Mutator
        self.mutator = Mutator()
        
        print("=" * 60)
        print("覆盖率引导模糊测试")
        print("=" * 60)
        print(f"初始语料库: {len(self.corpus)} 个种子")
        print(f"输出目录: {self.output_dir}")
        print(f"按 Ctrl+C 停止\n")
        
        iteration = 0
        
        while self.running:
            if max_iterations and iteration >= max_iterations:
                break
            
            # 从语料库选择种子
            seed = random.choice(self.corpus)
            
            # 变异
            mutated = self.mutator.mutate(seed, count=random.randint(1, 8))
            
            # 执行测试
            self._execute(mutated)
            
            iteration += 1
            self.stats.execs += 1
            
            # 定期报告
            if iteration % 1000 == 0:
                self._report()
        
        self._report()
        print(f"\n模糊测试完成")
        print(f"发现崩溃: {self.stats.unique_crashes} 个")
        print(f"崩溃保存位置: {self.crashes_dir}")
    
    def _execute(self, data: bytes):
        """执行目标函数"""
        try:
            # 设置超时
            signal.alarm(1)
            
            # 执行目标
            self.target(data)
            
            signal.alarm(0)
            
        except Exception as e:
            signal.alarm(0)
            self._handle_crash(data, e)
    
    def _handle_crash(self, data: bytes, exception: Exception):
        """处理崩溃"""
        self.stats.crashes += 1
        
        # 计算崩溃哈希
        crash_hash = hashlib.md5(data).hexdigest()[:16]
        
        if crash_hash not in self.unique_crashes:
            self.unique_crashes.add(crash_hash)
            self.stats.unique_crashes += 1
            
            # 保存崩溃输入
            crash_file = self.crashes_dir / f"crash-{crash_hash}"
            crash_file.write_bytes(data)
            
            print(f"\n[!] 发现新崩溃: {crash_hash}")
            print(f"    异常: {type(exception).__name__}: {exception}")
            print(f"    数据长度: {len(data)}")
    
    def _report(self):
        """报告状态"""
        elapsed = time.time() - self.stats.start_time
        print(f"\r执行: {self.stats.execs} | "
              f"速度: {self.stats.execs_per_sec:.0f}/s | "
              f"语料库: {len(self.corpus)} | "
              f"崩溃: {self.stats.unique_crashes} | "
              f"时间: {elapsed:.0f}s", end='', flush=True)


# 示例目标函数
def vulnerable_parser(data: bytes):
    """有漏洞的解析器 - 用于测试"""
    # 模拟解析逻辑
    if len(data) < 4:
        return
    
    # 模拟整数溢出
    length = int.from_bytes(data[:4], 'big')
    
    # 有问题的内存分配
    if length > 0x7FFFFFFF:
        raise MemoryError("Integer overflow")
    
    # 模拟缓冲区溢出检查
    if b"AAAA" in data and len(data) > 100:
        buffer = bytearray(50)
        for i, byte in enumerate(data):
            if i >= length:  # 边界检查被绕过
                break
            buffer[i] = byte  # 可能溢出
    
    # 模拟除零错误
    if b"DIV0" in data:
        divisor = data[data.find(b"DIV0") + 4]
        if divisor == 0:
            result = 100 / divisor
    
    # 模拟递归深度问题
    if data.startswith(b"REC"):
        def recursive(n):
            if n <= 0:
                return
            recursive(n - 1)
        recursive(10000)


import random


def demo_fuzzer():
    """演示模糊测试"""
    # 种子语料库
    seeds = [
        b"hello",
        b"test data",
        b"\x00\x01\x02\x03",
        b"AAAA" + b"B" * 200,
        b"RECURSION",
        b"DIVIDE BY ZERO",
    ]
    
    fuzzer = SimpleFuzzer(vulnerable_parser, seeds, max_iterations=5000)
    fuzzer.run()


if __name__ == '__main__':
    demo_fuzzer()
```

---

## 3. 技术实践

### 3.1 使用 AFL++

```bash
#!/bin/bash
# AFL++ 模糊测试脚本示例

# 1. 编译目标程序（使用 afl-clang-fast）
afl-clang-fast -o target_fuzz harness.c target.c

# 2. 创建语料库目录
mkdir -p corpus findings

# 3. 添加种子文件
echo "seed1" > corpus/seed1
echo "seed2" > corpus/seed2

# 4. 启动 AFL++ 模糊测试
afl-fuzz -i corpus -o findings ./target_fuzz @@

# 5. 查看结果
# - findings/crashes: 崩溃输入
# - findings/hangs: 挂起输入
# - findings/queue: 语料库
```

### 3.2 使用 libFuzzer

```cpp
// fuzz_target.cpp
// libFuzzer 目标函数示例

#include <cstdint>
#include <cstddef>

// 要测试的函数
extern "C" int LLVMFuzzerTestOneInput(const uint8_t *data, size_t size) {
    // 调用被测代码
    if (size < 4) return 0;
    
    // 示例：解析器测试
    if (data[0] == 'F' && data[1] == 'U' && data[2] == 'Z' && data[3] == 'Z') {
        // 可能触发 bug 的代码路径
        if (size > 100) {
            char buffer[50];
            memcpy(buffer, data, size);  // 缓冲区溢出
        }
    }
    
    return 0;
}
```

```bash
# 编译
c++ -g -O1 -fsanitize=fuzzer,address fuzz_target.cpp -o fuzz_target

# 运行
./fuzz_target -max_total_time=300 corpus/
```

### 3.3 Web API 模糊测试

```python
#!/usr/bin/env python3
"""
Web API 模糊测试
"""

import requests
import random
import string
import json
from typing import Dict, Any


class APIFuzzer:
    """API 模糊测试器"""
    
    def __init__(self, base_url: str):
        self.base_url = base_url
        self.session = requests.Session()
        self.findings = []
    
    def random_string(self, min_len: int = 1, max_len: int = 1000) -> str:
        """生成随机字符串"""
        length = random.randint(min_len, max_len)
        return ''.join(random.choices(string.printable, k=length))
    
    def random_number(self) -> Any:
        """生成随机数字"""
        choices = [
            random.randint(-1000000, 1000000),
            random.random() * 1000000,
            0, -1, 1,
            2**31 - 1, 2**31, 2**32,
            float('inf'), float('-inf'),
        ]
        return random.choice(choices)
    
    def fuzz_value(self, original: Any) -> Any:
        """模糊单个值"""
        fuzzers = [
            lambda: None,
            lambda: "",
            lambda: self.random_string(),
            lambda: self.random_number(),
            lambda: [],
            lambda: {},
            lambda: True,
            lambda: False,
            lambda: "null",
            lambda: "undefined",
        ]
        return random.choice(fuzzers)()
    
    def fuzz_json(self, template: Dict) -> Dict:
        """模糊 JSON 数据"""
        result = {}
        for key, value in template.items():
            if random.random() < 0.3:
                # 30% 概率变异
                result[key] = self.fuzz_value(value)
            elif isinstance(value, dict):
                result[key] = self.fuzz_json(value)
            elif isinstance(value, list):
                result[key] = [self.fuzz_value(item) for item in value]
            else:
                result[key] = value
        return result
    
    def test_endpoint(self, method: str, endpoint: str, 
                     template: Dict = None,
                     iterations: int = 100):
        """测试单个端点"""
        url = f"{self.base_url}{endpoint}"
        
        for i in range(iterations):
            try:
                if template:
                    data = self.fuzz_json(template)
                else:
                    data = {"fuzz": self.random_string()}
                
                if method == "GET":
                    response = self.session.get(url, params=data, timeout=5)
                elif method == "POST":
                    response = self.session.post(url, json=data, timeout=5)
                elif method == "PUT":
                    response = self.session.put(url, json=data, timeout=5)
                elif method == "DELETE":
                    response = self.session.delete(url, timeout=5)
                
                # 检查异常响应
                if response.status_code >= 500:
                    self._record_finding("Server Error", url, method, data, response)
                
                # 检查响应时间
                if response.elapsed.total_seconds() > 5:
                    self._record_finding("Slow Response", url, method, data, response)
                
            except requests.exceptions.Timeout:
                self._record_finding("Timeout", url, method, data, None)
            except requests.exceptions.RequestException as e:
                self._record_finding("Request Error", url, method, data, None, str(e))
            
            if (i + 1) % 10 == 0:
                print(f"  进度: {i+1}/{iterations}")
    
    def _record_finding(self, finding_type: str, url: str, method: str,
                       data: Dict, response, error: str = None):
        """记录发现"""
        finding = {
            'type': finding_type,
            'url': url,
            'method': method,
            'data': data,
            'timestamp': time.time()
        }
        
        if response:
            finding['status_code'] = response.status_code
            finding['response_time'] = response.elapsed.total_seconds()
        
        if error:
            finding['error'] = error
        
        self.findings.append(finding)
        print(f"[!] 发现: {finding_type} - {method} {url}")
    
    def generate_report(self) -> str:
        """生成测试报告"""
        lines = []
        lines.append("# API 模糊测试报告")
        lines.append(f"\n目标: {self.base_url}")
        lines.append(f"发现数: {len(self.findings)}")
        
        # 按类型分组
        by_type = {}
        for f in self.findings:
            t = f['type']
            if t not in by_type:
                by_type[t] = []
            by_type[t].append(f)
        
        lines.append(f"\n## 发现摘要")
        for t, findings in by_type.items():
            lines.append(f"- {t}: {len(findings)}")
        
        return '\n'.join(lines)


def demo_api_fuzzing():
    """演示 API 模糊测试"""
    print("=" * 60)
    print("API 模糊测试演示")
    print("=" * 60)
    
    fuzzer = APIFuzzer("http://localhost:8080")
    
    # 测试登录接口
    print("\n--- 测试登录接口 ---")
    login_template = {
        "username": "test",
        "password": "password"
    }
    fuzzer.test_endpoint("POST", "/api/login", login_template, iterations=50)
    
    # 测试用户注册
    print("\n--- 测试用户注册接口 ---")
    register_template = {
        "username": "testuser",
        "email": "test@example.com",
        "password": "password123"
    }
    fuzzer.test_endpoint("POST", "/api/register", register_template, iterations=50)
    
    print("\n--- 测试报告 ---")
    print(fuzzer.generate_report())


import time


if __name__ == '__main__':
    demo_api_fuzzing()
```

---

## 4. 资源索引

### 4.1 工具推荐

| 工具 | 类型 | 特点 | 链接 |
|------|------|------|------|
| AFL++ | 覆盖率引导 | 最强大的模糊测试器 | https://aflplus.plus/ |
| libFuzzer | 覆盖率引导 | LLVM 内置，易集成 | 随 LLVM 分发 |
| Honggfuzz | 覆盖率引导 | 支持硬件反馈 | https://honggfuzz.dev/ |
| Peach | 结构感知 | 协议模糊测试 | https://peach.software/ |
| Boofuzz | 网络协议 | 网络协议模糊测试 | https://github.com/jtpereyda/boofuzz |
| Jazzer | JVM | Java/Kotlin 模糊测试 | https://github.com/CodeIntelligenceTesting/jazzer |

### 4.2 学习资源

- **Fuzzing Book**: https://www.fuzzingbook.org/ - 交互式模糊测试教程
- **Google Fuzzing**: https://github.com/google/fuzzing - Google 模糊测试资源

---

## 5. 关联知识

```
A04_Security_Quality/B04_Quality_Assurance/
├── C02_Fuzzing_Techniques (本模块)
│   ├── → C01_Test_Automation: 自动化集成
│   └── → C03_Compliance_Scans: 安全合规
│
├── → B01_Information_Security: 安全漏洞挖掘
└── → B04_Quality_Assurance: 质量保证
```

---

*最后更新: 2024-01-30*
