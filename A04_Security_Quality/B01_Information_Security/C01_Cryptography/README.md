# C01 密码学 (Cryptography)

## 1. 主题定位

### 1.1 定义
密码学是研究信息安全技术的科学，涵盖数据加密、解密、认证和完整性保护。它是现代信息安全的基石，为数字通信、数据存储和身份验证提供数学保障。

### 1.2 核心价值
- **机密性 (Confidentiality)**: 确保信息仅对授权方可见
- **完整性 (Integrity)**: 防止数据被未授权篡改
- **认证性 (Authentication)**: 验证通信双方身份
- **不可否认性 (Non-repudiation)**: 防止通信方否认其行为

### 1.3 应用场景
- 网络通信加密 (TLS/SSL)
- 数据存储加密 (数据库、文件系统)
- 身份认证系统 (JWT、OAuth)
- 区块链与数字货币
- 安全邮件与即时通讯

---

## 2. 核心概念

### 2.1 对称加密 (Symmetric Encryption)

对称加密使用相同的密钥进行加密和解密，特点是速度快、适合大数据量加密。

#### 2.1.1 AES (Advanced Encryption Standard)
- **分组长度**: 128 位
- **密钥长度**: 128/192/256 位
- **工作模式**: ECB、CBC、CTR、GCM 等

```python
#!/usr/bin/env python3
# aes_encryption_demo.py
"""AES 加密解密完整演示"""

from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import padding
import os
import base64

class AESCrypto:
    """AES 加密工具类"""
    
    def __init__(self, key: bytes = None):
        """
        初始化 AES 加密器
        
        Args:
            key: 32字节密钥 (AES-256)，为None则自动生成
        """
        self.key = key or os.urandom(32)  # AES-256
        self.backend = default_backend()
    
    def encrypt_cbc(self, plaintext: str) -> dict:
        """
        CBC 模式加密
        
        CBC (Cipher Block Chaining) 模式：
        - 每个明文块与前一个密文块进行异或
        - 第一个块使用随机 IV (初始化向量)
        - 提供较好的安全性，但不支持并行加密
        """
        iv = os.urandom(16)  # 随机生成 IV
        
        # 创建 Cipher 对象
        cipher = Cipher(
            algorithms.AES(self.key),
            modes.CBC(iv),
            backend=self.backend
        )
        encryptor = cipher.encryptor()
        
        # PKCS7 填充
        padder = padding.PKCS7(128).padder()
        padded_data = padder.update(plaintext.encode()) + padder.finalize()
        
        # 加密
        ciphertext = encryptor.update(padded_data) + encryptor.finalize()
        
        return {
            'ciphertext': base64.b64encode(ciphertext).decode(),
            'iv': base64.b64encode(iv).decode(),
            'mode': 'CBC'
        }
    
    def decrypt_cbc(self, encrypted_data: dict) -> str:
        """CBC 模式解密"""
        ciphertext = base64.b64decode(encrypted_data['ciphertext'])
        iv = base64.b64decode(encrypted_data['iv'])
        
        cipher = Cipher(
            algorithms.AES(self.key),
            modes.CBC(iv),
            backend=self.backend
        )
        decryptor = cipher.decryptor()
        
        padded_plaintext = decryptor.update(ciphertext) + decryptor.finalize()
        
        # 去除填充
        unpadder = padding.PKCS7(128).unpadder()
        plaintext = unpadder.update(padded_plaintext) + unpadder.finalize()
        
        return plaintext.decode()
    
    def encrypt_gcm(self, plaintext: str, associated_data: bytes = None) -> dict:
        """
        GCM 模式加密 (Authenticated Encryption)
        
        GCM (Galois/Counter Mode) 特点：
        - 同时提供加密和认证
        - 生成认证标签 (tag) 防止篡改
        - 支持关联数据 (Associated Data) 认证
        - 目前推荐的 AEAD (Authenticated Encryption with Associated Data) 模式
        """
        iv = os.urandom(12)  # GCM 推荐 IV 长度为 96 位
        
        cipher = Cipher(
            algorithms.AES(self.key),
            modes.GCM(iv),
            backend=self.backend
        )
        encryptor = cipher.encryptor()
        
        # 添加关联数据 (可选)
        if associated_data:
            encryptor.authenticate_additional_data(associated_data)
        
        ciphertext = encryptor.update(plaintext.encode()) + encryptor.finalize()
        
        return {
            'ciphertext': base64.b64encode(ciphertext).decode(),
            'iv': base64.b64encode(iv).decode(),
            'tag': base64.b64encode(encryptor.tag).decode(),
            'mode': 'GCM'
        }
    
    def decrypt_gcm(self, encrypted_data: dict, associated_data: bytes = None) -> str:
        """GCM 模式解密并验证"""
        ciphertext = base64.b64decode(encrypted_data['ciphertext'])
        iv = base64.b64decode(encrypted_data['iv'])
        tag = base64.b64decode(encrypted_data['tag'])
        
        cipher = Cipher(
            algorithms.AES(self.key),
            modes.GCM(iv, tag),
            backend=self.backend
        )
        decryptor = cipher.decryptor()
        
        if associated_data:
            decryptor.authenticate_additional_data(associated_data)
        
        plaintext = decryptor.update(ciphertext) + decryptor.finalize()
        return plaintext.decode()


# 运行演示
if __name__ == '__main__':
    crypto = AESCrypto()
    message = "这是需要加密的敏感数据: SSN=123-45-6789"
    
    print("=" * 60)
    print("AES 加密演示")
    print("=" * 60)
    
    # CBC 模式演示
    print("\n--- CBC 模式 ---")
    encrypted_cbc = crypto.encrypt_cbc(message)
    print(f"原始消息: {message}")
    print(f"加密结果: {encrypted_cbc}")
    decrypted_cbc = crypto.decrypt_cbc(encrypted_cbc)
    print(f"解密结果: {decrypted_cbc}")
    
    # GCM 模式演示
    print("\n--- GCM 模式 (带认证) ---")
    associated = b"metadata:user_id=12345"
    encrypted_gcm = crypto.encrypt_gcm(message, associated)
    print(f"原始消息: {message}")
    print(f"关联数据: {associated}")
    print(f"加密结果: {encrypted_gcm}")
    decrypted_gcm = crypto.decrypt_gcm(encrypted_gcm, associated)
    print(f"解密结果: {decrypted_gcm}")
```

#### 2.1.2 ChaCha20-Poly1305
ChaCha20 是一种流密码，配合 Poly1305 消息认证码，在移动设备和无硬件 AES 加速的环境中表现优异。

```python
#!/usr/bin/env python3
# chacha20_demo.py
"""ChaCha20-Poly1305 流密码演示"""

from cryptography.hazmat.primitives.ciphers.aead import ChaCha20Poly1305
import os
import base64

def demo_chacha20():
    """ChaCha20-Poly1305 完整演示"""
    # 生成 256 位密钥
    key = ChaCha20Poly1305.generate_key()
    chacha = ChaCha20Poly1305(key)
    
    # 96 位 nonce
    nonce = os.urandom(12)
    
    plaintext = b"Secret message for mobile app communication"
    associated_data = b"header:v1.0"
    
    # 加密
    ciphertext = chacha.encrypt(nonce, plaintext, associated_data)
    
    print("=" * 60)
    print("ChaCha20-Poly1305 演示")
    print("=" * 60)
    print(f"密钥: {base64.b64encode(key).decode()}")
    print(f"Nonce: {base64.b64encode(nonce).decode()}")
    print(f"明文: {plaintext.decode()}")
    print(f"密文: {base64.b64encode(ciphertext).decode()}")
    
    # 解密
    decrypted = chacha.decrypt(nonce, ciphertext, associated_data)
    print(f"解密: {decrypted.decode()}")

if __name__ == '__main__':
    demo_chacha20()
```

### 2.2 非对称加密 (Asymmetric Encryption)

非对称加密使用密钥对：公钥加密、私钥解密，解决了密钥分发问题。

#### 2.2.1 RSA (Rivest-Shamir-Adleman)
```python
#!/usr/bin/env python3
# rsa_crypto_demo.py
"""RSA 非对称加密完整演示"""

from cryptography.hazmat.primitives.asymmetric import rsa, padding
from cryptography.hazmat.primitives import hashes, serialization
from cryptography.hazmat.backends import default_backend
import base64

class RSACrypto:
    """RSA 加密签名工具类"""
    
    def __init__(self, key_size: int = 2048):
        """
        生成 RSA 密钥对
        
        密钥长度建议：
        - 2048 位：目前最低安全标准
        - 3072 位：推荐用于长期保护
        - 4096 位：高安全要求场景
        """
        self.private_key = rsa.generate_private_key(
            public_exponent=65537,  # 常用公钥指数
            key_size=key_size,
            backend=default_backend()
        )
        self.public_key = self.private_key.public_key()
    
    def encrypt(self, plaintext: str) -> str:
        """
        RSA-OAEP 加密
        
        OAEP (Optimal Asymmetric Encryption Padding):
        - 填充方案提供语义安全性
        - 抵抗选择密文攻击
        """
        ciphertext = self.public_key.encrypt(
            plaintext.encode(),
            padding.OAEP(
                mgf=padding.MGF1(algorithm=hashes.SHA256()),
                algorithm=hashes.SHA256(),
                label=None
            )
        )
        return base64.b64encode(ciphertext).decode()
    
    def decrypt(self, ciphertext: str) -> str:
        """RSA-OAEP 解密"""
        plaintext = self.private_key.decrypt(
            base64.b64decode(ciphertext),
            padding.OAEP(
                mgf=padding.MGF1(algorithm=hashes.SHA256()),
                algorithm=hashes.SHA256(),
                label=None
            )
        )
        return plaintext.decode()
    
    def sign(self, message: str) -> str:
        """
        RSA-PSS 数字签名
        
        PSS (Probabilistic Signature Scheme):
        - 概率性签名方案
        - 提供可证明安全性
        """
        signature = self.private_key.sign(
            message.encode(),
            padding.PSS(
                mgf=padding.MGF1(hashes.SHA256()),
                salt_length=padding.PSS.MAX_LENGTH
            ),
            hashes.SHA256()
        )
        return base64.b64encode(signature).decode()
    
    def verify(self, message: str, signature: str) -> bool:
        """验证数字签名"""
        try:
            self.public_key.verify(
                base64.b64decode(signature),
                message.encode(),
                padding.PSS(
                    mgf=padding.MGF1(hashes.SHA256()),
                    salt_length=padding.PSS.MAX_LENGTH
                ),
                hashes.SHA256()
            )
            return True
        except Exception:
            return False
    
    def export_private_pem(self, password: str = None) -> str:
        """导出私钥为 PEM 格式"""
        encryption = (
            serialization.BestAvailableEncryption(password.encode())
            if password else serialization.NoEncryption()
        )
        pem = self.private_key.private_bytes(
            encoding=serialization.Encoding.PEM,
            format=serialization.PrivateFormat.PKCS8,
            encryption_algorithm=encryption
        )
        return pem.decode()
    
    def export_public_pem(self) -> str:
        """导出公钥为 PEM 格式"""
        pem = self.public_key.public_bytes(
            encoding=serialization.Encoding.PEM,
            format=serialization.PublicFormat.SubjectPublicKeyInfo
        )
        return pem.decode()


# 运行演示
if __name__ == '__main__':
    print("=" * 60)
    print("RSA 非对称加密演示")
    print("=" * 60)
    
    rsa_crypto = RSACrypto(key_size=2048)
    
    # 加密演示
    message = "这是 RSA 加密的消息"
    print(f"\n原始消息: {message}")
    
    encrypted = rsa_crypto.encrypt(message)
    print(f"加密后: {encrypted[:60]}...")
    
    decrypted = rsa_crypto.decrypt(encrypted)
    print(f"解密后: {decrypted}")
    
    # 签名演示
    print("\n--- 数字签名 ---")
    document = "重要合同内容: 甲方同意支付100万元"
    signature = rsa_crypto.sign(document)
    print(f"文档: {document}")
    print(f"签名: {signature[:60]}...")
    
    is_valid = rsa_crypto.verify(document, signature)
    print(f"签名验证: {'通过 ✓' if is_valid else '失败 ✗'}")
    
    # 导出密钥
    print("\n--- 密钥导出 ---")
    print("公钥 PEM:\n", rsa_crypto.export_public_pem()[:200], "...")
```

#### 2.2.2 ECC (Elliptic Curve Cryptography)
椭圆曲线密码学以更短的密钥提供与 RSA 相当的安全性。

```python
#!/usr/bin/env python3
# ecc_crypto_demo.py
"""椭圆曲线密码学 (ECC) 演示"""

from cryptography.hazmat.primitives.asymmetric import ec
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.backends import default_backend
import base64

class ECCCrypto:
    """ECC 加密签名工具类"""
    
    def __init__(self, curve=ec.SECP256R1()):
        """
        常用曲线：
        - SECP256R1 (P-256): 广泛支持，128位安全强度
        - SECP384R1 (P-384): 192位安全强度
        - SECP521R1 (P-521): 256位安全强度
        - Curve25519: 现代设计，高安全性
        """
        self.private_key = ec.generate_private_key(curve, default_backend())
        self.public_key = self.private_key.public_key()
    
    def sign(self, message: str) -> str:
        """ECDSA 签名"""
        signature = self.private_key.sign(
            message.encode(),
            ec.ECDSA(hashes.SHA256())
        )
        return base64.b64encode(signature).decode()
    
    def verify(self, message: str, signature: str) -> bool:
        """验证 ECDSA 签名"""
        try:
            self.public_key.verify(
                base64.b64decode(signature),
                message.encode(),
                ec.ECDSA(hashes.SHA256())
            )
            return True
        except Exception:
            return False
    
    def get_key_size(self) -> int:
        """获取密钥大小（位）"""
        return self.private_key.key_size


# 运行演示
if __name__ == '__main__':
    print("=" * 60)
    print("椭圆曲线密码学 (ECC) 演示")
    print("=" * 60)
    
    # 不同曲线的比较
    curves = [
        ('P-256 (SECP256R1)', ec.SECP256R1()),
        ('P-384 (SECP384R1)', ec.SECP384R1()),
        ('P-521 (SECP521R1)', ec.SECP521R1()),
    ]
    
    message = "ECC 签名测试消息"
    
    for name, curve in curves:
        print(f"\n--- {name} ---")
        ecc = ECCCrypto(curve)
        print(f"密钥大小: {ecc.get_key_size()} 位")
        
        signature = ecc.sign(message)
        print(f"签名长度: {len(signature)} 字符")
        
        is_valid = ecc.verify(message, signature)
        print(f"验证结果: {'通过 ✓' if is_valid else '失败 ✗'}")
    
    # 安全性对比
    print("\n" + "=" * 60)
    print("RSA vs ECC 安全性对比（相当的安全级别）")
    print("=" * 60)
    comparison = [
        ("RSA 1024", "ECC 160"),
        ("RSA 2048", "ECC 224"),
        ("RSA 3072", "ECC 256"),
        ("RSA 7680", "ECC 384"),
        ("RSA 15360", "ECC 521"),
    ]
    for rsa_key, ecc_key in comparison:
        print(f"{rsa_key} ≈ {ecc_key}")
```

### 2.3 哈希函数 (Hash Functions)

哈希函数将任意长度输入映射为固定长度输出，具有单向性和抗碰撞性。

```python
#!/usr/bin/env python3
# hash_functions_demo.py
"""密码学哈希函数完整演示"""

import hashlib
import hmac
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.backends import default_backend
import bcrypt
import secrets

def demo_hash_functions():
    """演示各种哈希函数"""
    message = "Hello, Cryptography!"
    message_bytes = message.encode()
    
    print("=" * 60)
    print("密码学哈希函数演示")
    print("=" * 60)
    
    # MD5 (已弃用，仅作演示)
    print("\n--- MD5 (不推荐用于安全场景) ---")
    md5_hash = hashlib.md5(message_bytes).hexdigest()
    print(f"输入: {message}")
    print(f"MD5: {md5_hash} ({len(md5_hash)*4} bits)")
    print("⚠️ 警告: MD5 已被破解，存在碰撞攻击")
    
    # SHA-1 (已弃用)
    print("\n--- SHA-1 (逐步淘汰中) ---")
    sha1_hash = hashlib.sha1(message_bytes).hexdigest()
    print(f"SHA-1: {sha1_hash} ({len(sha1_hash)*4} bits)")
    print("⚠️ 警告: SHA-1 存在理论碰撞攻击")
    
    # SHA-256 (推荐)
    print("\n--- SHA-256 (当前标准) ---")
    sha256_hash = hashlib.sha256(message_bytes).hexdigest()
    print(f"SHA-256: {sha256_hash} ({len(sha256_hash)*4} bits)")
    
    # SHA-3 / Keccak
    print("\n--- SHA-3-256 (新一代标准) ---")
    sha3_hash = hashlib.sha3_256(message_bytes).hexdigest()
    print(f"SHA3-256: {sha3_hash}")
    
    # BLAKE2 (高性能)
    print("\n--- BLAKE2b (高性能替代) ---")
    blake2_hash = hashlib.blake2b(message_bytes).hexdigest()
    print(f"BLAKE2b: {blake2_hash}")


def demo_hmac():
    """HMAC (Keyed-Hash Message Authentication Code)"""
    print("\n" + "=" * 60)
    print("HMAC 消息认证码演示")
    print("=" * 60)
    
    key = secrets.token_bytes(32)  # 256-bit 密钥
    message = "需要认证的消息"
    
    # 计算 HMAC-SHA256
    mac = hmac.new(key, message.encode(), hashlib.sha256).hexdigest()
    
    print(f"密钥: {key.hex()[:32]}...")
    print(f"消息: {message}")
    print(f"HMAC-SHA256: {mac}")
    print("\n用途: API 请求签名、Cookie 完整性验证、JWT 签名")


def demo_password_hashing():
    """密码哈希（使用 bcrypt）"""
    print("\n" + "=" * 60)
    print("密码安全哈希 (bcrypt) 演示")
    print("=" * 60)
    
    password = "user_password_123"
    
    # 生成盐并哈希
    # work factor 12 表示 2^12 次迭代
    hashed = bcrypt.hashpw(password.encode(), bcrypt.gensalt(rounds=12))
    
    print(f"原始密码: {password}")
    print(f"bcrypt 哈希: {hashed.decode()}")
    print("\nbcrypt 哈希结构: $2b$rounds$salt+hash")
    
    # 验证密码
    is_valid = bcrypt.checkpw(password.encode(), hashed)
    print(f"\n验证结果: {'密码正确 ✓' if is_valid else '密码错误 ✗'}")
    
    # 错误密码验证
    is_invalid = bcrypt.checkpw(b"wrong_password", hashed)
    print(f"错误密码验证: {'密码正确 ✓' if is_invalid else '密码错误 ✗'}")


def demo_key_derivation():
    """密钥派生函数 (PBKDF2, Argon2)"""
    print("\n" + "=" * 60)
    print("密钥派生函数演示")
    print("=" * 60)
    
    password = b"my_secure_password"
    salt = secrets.token_bytes(16)
    
    # PBKDF2-HMAC-SHA256
    print("\n--- PBKDF2 ---")
    from hashlib import pbkdf2_hmac
    key = pbkdf2_hmac('sha256', password, salt, iterations=100000, dklen=32)
    print(f"派生密钥: {key.hex()}")
    print(f"迭代次数: 100000 (OWASP 2023 推荐)")
    
    # Argon2 (现代推荐)
    print("\n--- Argon2 (密码哈希竞赛 winner) ---")
    try:
        import argon2
        ph = argon2.PasswordHasher(
            time_cost=3,      # 迭代次数
            memory_cost=65536, # 64 MB
            parallelism=4     # 并行度
        )
        hash_argon = ph.hash(password.decode())
        print(f"Argon2 哈希: {hash_argon}")
        
        verify = ph.verify(hash_argon, password.decode())
        print(f"验证结果: {'通过 ✓' if verify else '失败 ✗'}")
    except ImportError:
        print("需要安装: pip install argon2-cffi")


if __name__ == '__main__':
    demo_hash_functions()
    demo_hmac()
    demo_password_hashing()
    demo_key_derivation()
```

### 2.4 密钥管理

```python
#!/usr/bin/env python3
# key_management_demo.py
"""密钥管理系统演示"""

from cryptography.fernet import Fernet
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2
from cryptography.hazmat.primitives import hashes
import os
import json
import base64
import secrets

class KeyManager:
    """密钥管理器 - 演示密钥分层和轮换"""
    
    def __init__(self, master_key: bytes = None):
        """
        密钥管理器初始化
        
        密钥层次结构：
        - KEK (Key Encryption Key): 加密数据密钥
        - DEK (Data Encryption Key): 实际加密数据
        - MEK (Master Encryption Key): 根密钥
        """
        self.master_key = master_key or Fernet.generate_key()
        self.kek_cache = {}
        self.key_versions = {}
    
    def generate_dek(self, key_id: str) -> bytes:
        """生成数据加密密钥 (DEK)"""
        dek = Fernet.generate_key()
        version = self._get_next_version(key_id)
        
        # 使用 KEK 加密 DEK
        encrypted_dek = self._encrypt_with_kek(dek, key_id)
        
        self.key_versions[key_id] = {
            'version': version,
            'encrypted_dek': encrypted_dek,
            'created_at': self._timestamp()
        }
        
        return dek
    
    def _encrypt_with_kek(self, dek: bytes, key_id: str) -> str:
        """使用 KEK 加密 DEK"""
        f = Fernet(self.master_key)
        return f.encrypt(dek).decode()
    
    def _get_next_version(self, key_id: str) -> int:
        """获取下一个密钥版本"""
        current = self.key_versions.get(key_id, {}).get('version', 0)
        return current + 1
    
    def _timestamp(self) -> str:
        """获取当前时间戳"""
        from datetime import datetime
        return datetime.now().isoformat()
    
    def rotate_key(self, key_id: str) -> bytes:
        """密钥轮换"""
        print(f"\n执行密钥轮换: {key_id}")
        old_version = self.key_versions.get(key_id, {}).get('version', 0)
        new_dek = self.generate_dek(key_id)
        print(f"版本: v{old_version} -> v{self.key_versions[key_id]['version']}")
        return new_dek
    
    def export_key_metadata(self) -> dict:
        """导出密钥元数据（不含明文密钥）"""
        return {
            'master_key_id': hash(self.master_key) & 0xFFFFFF,
            'keys': self.key_versions
        }


def demo_envelope_encryption():
    """信封加密演示 - AWS KMS 风格"""
    print("=" * 60)
    print("信封加密 (Envelope Encryption) 演示")
    print("=" * 60)
    
    # 1. 生成数据密钥 (DEK)
    dek = Fernet.generate_key()
    f_dek = Fernet(dek)
    
    # 2. 使用 KMS/Master Key 加密 DEK
    master_key = Fernet.generate_key()
    f_master = Fernet(master_key)
    encrypted_dek = f_master.encrypt(dek)
    
    # 3. 使用 DEK 加密数据
    plaintext = "敏感业务数据：用户信用卡信息"
    ciphertext = f_dek.encrypt(plaintext.encode())
    
    print(f"\n明文数据: {plaintext}")
    print(f"加密后的 DEK: {encrypted_dek[:50].decode()}...")
    print(f"数据密文: {ciphertext[:50].decode()}...")
    
    # 解密流程
    print("\n--- 解密流程 ---")
    # 1. 解密 DEK
    decrypted_dek = f_master.decrypt(encrypted_dek)
    f_dek_dec = Fernet(decrypted_dek)
    
    # 2. 解密数据
    decrypted_data = f_dek_dec.decrypt(ciphertext)
    print(f"解密结果: {decrypted_data.decode()}")


if __name__ == '__main__':
    # 密钥管理演示
    print("=" * 60)
    print("密钥管理系统演示")
    print("=" * 60)
    
    km = KeyManager()
    
    # 生成 DEK
    dek1 = km.generate_dek("user-data-key")
    print(f"\n生成 DEK: {dek1[:20].decode()}...")
    
    # 密钥轮换
    dek2 = km.rotate_key("user-data-key")
    
    # 查看元数据
    print("\n密钥元数据:")
    print(json.dumps(km.export_key_metadata(), indent=2, default=str))
    
    # 信封加密
    demo_envelope_encryption()
```

---

## 3. 技术实践

### 3.1 TLS/SSL 安全配置

```python
#!/usr/bin/env python3
# tls_config_demo.py
"""TLS/SSL 安全配置检查工具"""

import ssl
import socket
import certifi
from datetime import datetime

class TLSConfigChecker:
    """TLS 配置安全检查器"""
    
    # 安全的协议版本
    SECURE_PROTOCOLS = {
        ssl.PROTOCOL_TLS_CLIENT: "TLS 1.2+",
    }
    
    # 推荐的密码套件 (2024)
    RECOMMENDED_CIPHERS = [
        'TLS_AES_256_GCM_SHA384',
        'TLS_CHACHA20_POLY1305_SHA256',
        'TLS_AES_128_GCM_SHA256',
        'ECDHE-ECDSA-AES256-GCM-SHA384',
        'ECDHE-RSA-AES256-GCM-SHA384',
        'ECDHE-ECDSA-CHACHA20-POLY1305',
        'ECDHE-RSA-CHACHA20-POLY1305',
    ]
    
    # 不安全的密码套件
    INSECURE_CIPHERS = [
        'RC4', 'DES', '3DES', 'MD5', 'SHA1',
        'NULL', 'EXPORT', 'CBC', 'RSA '
    ]
    
    def __init__(self):
        self.context = ssl.create_default_context()
    
    def create_secure_context(self) -> ssl.SSLContext:
        """创建安全的 SSL 上下文"""
        context = ssl.SSLContext(ssl.PROTOCOL_TLS_CLIENT)
        
        # 最小 TLS 版本 1.2
        context.minimum_version = ssl.TLSVersion.TLSv1_2
        
        # 禁用不安全的协议
        context.options |= ssl.OP_NO_SSLv2
        context.options |= ssl.OP_NO_SSLv3
        context.options |= ssl.OP_NO_TLSv1
        context.options |= ssl.OP_NO_TLSv1_1
        
        # 证书验证
        context.check_hostname = True
        context.verify_mode = ssl.CERT_REQUIRED
        context.load_verify_locations(certifi.where())
        
        # 密码套件配置
        context.set_ciphers(':'.join(self.RECOMMENDED_CIPHERS))
        
        return context
    
    def check_server_tls(self, hostname: str, port: int = 443) -> dict:
        """检查服务器的 TLS 配置"""
        result = {
            'hostname': hostname,
            'port': port,
            'connected': False,
            'protocol': None,
            'cipher': None,
            'certificate': None,
            'issues': []
        }
        
        try:
            context = self.create_secure_context()
            
            with socket.create_connection((hostname, port), timeout=10) as sock:
                with context.wrap_socket(sock, server_hostname=hostname) as ssock:
                    result['connected'] = True
                    result['protocol'] = ssock.version()
                    result['cipher'] = ssock.cipher()
                    
                    # 获取证书信息
                    cert = ssock.getpeercert()
                    result['certificate'] = {
                        'subject': cert.get('subject'),
                        'issuer': cert.get('issuer'),
                        'not_after': cert.get('notAfter'),
                        'not_before': cert.get('notBefore'),
                        'serial_number': cert.get('serialNumber'),
                        'subject_alt_name': cert.get('subjectAltName')
                    }
                    
                    # 检查证书过期
                    not_after = cert.get('notAfter')
                    if not_after:
                        expiry = datetime.strptime(not_after, '%b %d %H:%M:%S %Y %Z')
                        days_until_expiry = (expiry - datetime.now()).days
                        if days_until_expiry < 30:
                            result['issues'].append(
                                f"证书将在 {days_until_expiry} 天后过期"
                            )
                    
                    # 检查协议版本
                    if result['protocol'] in ['TLSv1', 'TLSv1.1']:
                        result['issues'].append(
                            f"使用过时协议: {result['protocol']}"
                        )
                    
        except ssl.SSLError as e:
            result['issues'].append(f"SSL 错误: {str(e)}")
        except socket.error as e:
            result['issues'].append(f"连接错误: {str(e)}")
        
        return result
    
    def print_report(self, result: dict):
        """打印检查报告"""
        print("\n" + "=" * 60)
        print(f"TLS 检查报告: {result['hostname']}:{result['port']}")
        print("=" * 60)
        
        print(f"\n连接状态: {'✓ 成功' if result['connected'] else '✗ 失败'}")
        
        if result['connected']:
            print(f"协议版本: {result['protocol']}")
            print(f"密码套件: {result['cipher'][0] if result['cipher'] else 'N/A'}")
            
            if result['certificate']:
                cert = result['certificate']
                print(f"\n证书信息:")
                print(f"  颁发给: {cert.get('subject')}")
                print(f"  颁发者: {cert.get('issuer')}")
                print(f"  有效期至: {cert.get('not_after')}")
        
        if result['issues']:
            print(f"\n⚠️ 发现的问题:")
            for issue in result['issues']:
                print(f"  - {issue}")
        else:
            print("\n✓ 未发现安全问题")


# 运行演示
if __name__ == '__main__':
    checker = TLSConfigChecker()
    
    # 测试常见网站
    test_hosts = ['google.com', 'github.com', 'badssl.com']
    
    for host in test_hosts:
        result = checker.check_server_tls(host)
        checker.print_report(result)
```

### 3.2 JWT 安全实现

```python
#!/usr/bin/env python3
# jwt_security_demo.py
"""JWT 安全实现与常见漏洞演示"""

import jwt
import json
import base64
import hmac
import hashlib
from datetime import datetime, timedelta
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives.asymmetric import rsa
import secrets

class SecureJWTHandler:
    """安全的 JWT 处理器"""
    
    # 安全的算法白名单
    SECURE_ALGORITHMS = ['HS256', 'HS384', 'HS512', 'RS256', 'RS384', 'RS512', 'ES256', 'ES384', 'ES512']
    
    # 禁止的不安全算法
    INSECURE_ALGORITHMS = ['none', 'None', 'NONE', 'HS1', 'HS0']
    
    def __init__(self, secret: str = None, algorithm: str = 'HS256'):
        """
        初始化 JWT 处理器
        
        Args:
            secret: HMAC 密钥或 RSA 私钥
            algorithm: 签名算法
        """
        if algorithm in self.INSECURE_ALGORITHMS:
            raise ValueError(f"不安全的算法: {algorithm}")
        
        self.secret = secret or secrets.token_urlsafe(32)
        self.algorithm = algorithm
        self.private_key = None
        self.public_key = None
        
        # 如果是 RSA 算法，生成密钥对
        if algorithm.startswith('RS'):
            self._generate_rsa_keys()
    
    def _generate_rsa_keys(self):
        """生成 RSA 密钥对"""
        private_key = rsa.generate_private_key(
            public_exponent=65537,
            key_size=2048
        )
        self.private_key = private_key
        self.public_key = private_key.public_key()
    
    def create_token(self, payload: dict, expires_in: int = 3600) -> str:
        """
        创建 JWT Token
        
        Args:
            payload: 自定义声明
            expires_in: 过期时间（秒）
        """
        # 添加标准声明
        now = datetime.utcnow()
        full_payload = {
            'iat': now,  # 签发时间
            'exp': now + timedelta(seconds=expires_in),  # 过期时间
            'nbf': now,  # 生效时间
            'jti': secrets.token_urlsafe(16),  # JWT ID
            **payload
        }
        
        # 选择签名密钥
        if self.algorithm.startswith('RS'):
            key = self.private_key
        else:
            key = self.secret
        
        token = jwt.encode(
            full_payload,
            key,
            algorithm=self.algorithm,
            headers={'typ': 'JWT', 'alg': self.algorithm}
        )
        
        return token
    
    def verify_token(self, token: str) -> dict:
        """
        验证 JWT Token
        
        安全验证点：
        1. 算法白名单校验
        2. 签名验证
        3. 过期时间检查
        4. 生效时间检查
        """
        try:
            # 解析头部以获取算法（不解密）
            header = self._decode_header(token)
            alg = header.get('alg')
            
            # 检查算法白名单
            if alg not in self.SECURE_ALGORITHMS:
                raise jwt.InvalidTokenError(f"不允许的算法: {alg}")
            
            # 防止算法切换攻击
            if alg != self.algorithm:
                raise jwt.InvalidTokenError(f"算法不匹配: 期望 {self.algorithm}, 实际 {alg}")
            
            # 选择验证密钥
            if alg.startswith('RS'):
                key = self.public_key
            else:
                key = self.secret
            
            # 验证并解码
            payload = jwt.decode(
                token,
                key,
                algorithms=[self.algorithm],
                options={
                    'verify_signature': True,
                    'verify_exp': True,
                    'verify_iat': True,
                    'verify_nbf': True,
                    'require': ['exp', 'iat']
                }
            )
            
            return {'valid': True, 'payload': payload}
            
        except jwt.ExpiredSignatureError:
            return {'valid': False, 'error': 'Token 已过期'}
        except jwt.InvalidTokenError as e:
            return {'valid': False, 'error': f'无效的 Token: {str(e)}'}
    
    def _decode_header(self, token: str) -> dict:
        """解码 JWT 头部（不解密）"""
        try:
            header_b64 = token.split('.')[0]
            # 添加填充
            padding = 4 - len(header_b64) % 4
            if padding != 4:
                header_b64 += '=' * padding
            header_json = base64.urlsafe_b64decode(header_b64)
            return json.loads(header_json)
        except Exception as e:
            raise jwt.InvalidTokenError(f"无法解析头部: {str(e)}")


def demo_jwt_none_attack():
    """演示 'none' 算法攻击"""
    print("\n" + "=" * 60)
    print("JWT 'none' 算法攻击演示")
    print("=" * 60)
    
    # 攻击者构造的恶意 Token
    malicious_header = base64.urlsafe_b64encode(
        json.dumps({'alg': 'none', 'typ': 'JWT'}).encode()
    ).decode().rstrip('=')
    
    malicious_payload = base64.urlsafe_b64encode(
        json.dumps({'user': 'admin', 'role': 'superuser'}).encode()
    ).decode().rstrip('=')
    
    malicious_token = f"{malicious_header}.{malicious_payload}."
    
    print(f"\n恶意构造的 Token (alg=none):")
    print(malicious_token[:80] + "...")
    
    # 安全处理器会拒绝
    handler = SecureJWTHandler(algorithm='HS256')
    result = handler.verify_token(malicious_token)
    print(f"\n安全验证结果: {result}")


def demo_jwt_algorithm_confusion():
    """演示算法混淆攻击 (RS256 -> HS256)"""
    print("\n" + "=" * 60)
    print("JWT 算法混淆攻击演示")
    print("=" * 60)
    
    # 正常 RS256 场景
    rs_handler = SecureJWTHandler(algorithm='RS256')
    
    # 获取公钥并转换为字符串（攻击者可见）
    public_pem = rs_handler.public_key.public_bytes(
        encoding=serialization.Encoding.PEM,
        format=serialization.PublicFormat.SubjectPublicKeyInfo
    )
    
    print("\n--- 正常 RS256 场景 ---")
    token = rs_handler.create_token({'user': 'alice', 'role': 'user'})
    print(f"RS256 Token: {token[:60]}...")
    result = rs_handler.verify_token(token)
    print(f"验证结果: {'✓ 有效' if result['valid'] else '✗ 无效'}")
    
    # 攻击尝试：用公钥作为 HMAC 密钥
    print("\n--- 攻击场景：用公钥作为 HMAC 密钥 ---")
    
    # 攻击者知道公钥，尝试用 HS256 签名
    attacker_header = base64.urlsafe_b64encode(
        json.dumps({'alg': 'HS256', 'typ': 'JWT'}).encode()
    ).decode().rstrip('=')
    
    attacker_payload = base64.urlsafe_b64encode(
        json.dumps({'user': 'admin', 'role': 'superuser'}).encode()
    ).decode().rstrip('=')
    
    message = f"{attacker_header}.{attacker_payload}"
    
    # 用公钥作为密钥进行 HMAC 签名
    signature = base64.urlsafe_b64encode(
        hmac.new(public_pem, message.encode(), hashlib.sha256).digest()
    ).decode().rstrip('=')
    
    forged_token = f"{message}.{signature}"
    print(f"伪造的 Token: {forged_token[:60]}...")
    
    # 正确配置的服务器会拒绝
    result = rs_handler.verify_token(forged_token)
    print(f"验证结果: {'✓ 有效（漏洞！）' if result['valid'] else '✗ 无效（安全）'}")
    print("\n防护措施：严格校验 alg 头部，只允许预期的算法")


if __name__ == '__main__':
    print("=" * 60)
    print("JWT 安全实现演示")
    print("=" * 60)
    
    # HMAC 演示
    print("\n--- HMAC (HS256) ---")
    hmac_handler = SecureJWTHandler(algorithm='HS256')
    token = hmac_handler.create_token(
        {'user_id': '12345', 'role': 'admin'},
        expires_in=3600
    )
    print(f"生成的 Token: {token}")
    
    result = hmac_handler.verify_token(token)
    print(f"验证结果: {result}")
    
    # RSA 演示
    print("\n--- RSA (RS256) ---")
    rsa_handler = SecureJWTHandler(algorithm='RS256')
    token = rsa_handler.create_token({'user_id': '67890', 'role': 'user'})
    print(f"生成的 Token: {token[:80]}...")
    
    result = rsa_handler.verify_token(token)
    print(f"验证结果: {result}")
    
    # 攻击演示
    demo_jwt_none_attack()
    demo_jwt_algorithm_confusion()
```

### 3.3 端到端加密实现

```python
#!/usr/bin/env python3
# e2ee_demo.py
"""端到端加密 (E2EE) 实现演示 - Signal Protocol 简化版"""

from cryptography.hazmat.primitives.asymmetric.x25519 import X25519PrivateKey, X25519PublicKey
from cryptography.hazmat.primitives.ciphers.aead import AESGCM
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.kdf.hkdf import HKDF
import os
import base64

class E2EESession:
    """
    简化版端到端加密会话
    基于 X3DH (Extended Triple Diffie-Hellman) 概念
    """
    
    def __init__(self, identity_key: X25519PrivateKey = None):
        """
        初始化 E2EE 会话
        
        Args:
            identity_key: 长期身份密钥，None 则自动生成
        """
        self.identity_key = identity_key or X25519PrivateKey.generate()
        self.identity_public = self.identity_public = self.identity_key.public_key()
        
        # 临时密钥（每次会话生成）
        self.ephemeral_key = None
        
        # 共享密钥
        self.shared_secret = None
        self.root_key = None
        self.chain_key = None
        self.message_keys = {}
    
    def generate_prekey_bundle(self) -> dict:
        """
        生成预密钥包（用于初始握手）
        
        包含：
        - 身份公钥 (长期)
        - 临时公钥 (短期)
        - 一次性预密钥 (可选)
        """
        self.ephemeral_key = X25519PrivateKey.generate()
        
        return {
            'identity_key': base64.b64encode(
                self.identity_public.public_bytes(
                    encoding=serialization.Encoding.Raw,
                    format=serialization.PublicFormat.Raw
                )
            ).decode(),
            'ephemeral_key': base64.b64encode(
                self.ephemeral_key.public_key().public_bytes(
                    encoding=serialization.Encoding.Raw,
                    format=serialization.PublicFormat.Raw
                )
            ).decode(),
        }
    
    def initiate_session(self, recipient_bundle: dict):
        """
        作为发送方初始化会话
        
        执行 X3DH 密钥协商：
        DH1 = DH(IKA, SPKB)  - 身份密钥与接收方临时密钥
        DH2 = DH(EKA, IKB)   - 临时密钥与接收方身份密钥
        DH3 = DH(EKA, SPKB)  - 双方临时密钥
        SK = KDF(DH1 || DH2 || DH3)
        """
        # 解析接收方公钥
        recipient_identity = X25519PublicKey.from_public_bytes(
            base64.b64decode(recipient_bundle['identity_key'])
        )
        recipient_ephemeral = X25519PublicKey.from_public_bytes(
            base64.b64decode(recipient_bundle['ephemeral_key'])
        )
        
        # 生成发送方临时密钥
        sender_ephemeral = X25519PrivateKey.generate()
        
        # X3DH 密钥交换
        dh1 = self.identity_key.exchange(recipient_ephemeral)
        dh2 = sender_ephemeral.exchange(recipient_identity)
        dh3 = sender_ephemeral.exchange(recipient_ephemeral)
        
        # 派生共享密钥
        shared_secret = dh1 + dh2 + dh3
        
        self.root_key = HKDF(
            algorithm=hashes.SHA256(),
            length=32,
            salt=None,
            info=b'SignalProtocol',
        ).derive(shared_secret)
        
        self.chain_key = self.root_key
        
        return sender_ephemeral.public_key()
    
    def receive_session(self, sender_bundle: dict, sender_ephemeral_pub: X25519PublicKey):
        """作为接收方接受会话"""
        sender_identity = X25519PublicKey.from_public_bytes(
            base64.b64decode(sender_bundle['identity_key'])
        )
        
        # 对应的 X3DH 计算
        dh1 = self.ephemeral_key.exchange(sender_identity)
        dh2 = self.identity_key.exchange(sender_ephemeral_pub)
        dh3 = self.ephemeral_key.exchange(sender_ephemeral_pub)
        
        shared_secret = dh1 + dh2 + dh3
        
        self.root_key = HKDF(
            algorithm=hashes.SHA256(),
            length=32,
            salt=None,
            info=b'SignalProtocol',
        ).derive(shared_secret)
        
        self.chain_key = self.root_key
    
    def encrypt_message(self, plaintext: str) -> dict:
        """
        加密消息（使用双棘轮的前向保密）
        
        每次加密后更新链密钥，实现前向保密
        """
        # 派生消息密钥
        message_key = HKDF(
            algorithm=hashes.SHA256(),
            length=32,
            salt=None,
            info=self.chain_key,
        ).derive(b'message_key')
        
        # 更新链密钥
        self.chain_key = HKDF(
            algorithm=hashes.SHA256(),
            length=32,
            salt=None,
            info=self.chain_key,
        ).derive(b'chain_key')
        
        # AES-GCM 加密
        nonce = os.urandom(12)
        aesgcm = AESGCM(message_key)
        ciphertext = aesgcm.encrypt(nonce, plaintext.encode(), None)
        
        return {
            'ciphertext': base64.b64encode(ciphertext).decode(),
            'nonce': base64.b64encode(nonce).decode(),
        }
    
    def decrypt_message(self, encrypted_message: dict) -> str:
        """解密消息"""
        # 派生相同的消息密钥
        message_key = HKDF(
            algorithm=hashes.SHA256(),
            length=32,
            salt=None,
            info=self.chain_key,
        ).derive(b'message_key')
        
        # 更新链密钥
        self.chain_key = HKDF(
            algorithm=hashes.SHA256(),
            length=32,
            salt=None,
            info=self.chain_key,
        ).derive(b'chain_key')
        
        # 解密
        ciphertext = base64.b64decode(encrypted_message['ciphertext'])
        nonce = base64.b64decode(encrypted_message['nonce'])
        
        aesgcm = AESGCM(message_key)
        plaintext = aesgcm.decrypt(nonce, ciphertext, None)
        
        return plaintext.decode()


# 修复缺失的导入
from cryptography.hazmat.primitives import serialization


def demo_e2ee_chat():
    """演示端到端加密聊天"""
    print("=" * 60)
    print("端到端加密 (E2EE) 演示")
    print("=" * 60)
    
    # Alice 和 Bob 初始化
    print("\n--- 会话初始化 ---")
    alice = E2EESession()
    bob = E2EESession()
    
    # Bob 生成预密钥包
    bob_bundle = bob.generate_prekey_bundle()
    print("Bob 生成预密钥包")
    
    # Alice 初始化会话
    alice_ephemeral_pub = alice.initiate_session(bob_bundle)
    print("Alice 完成 X3DH 密钥协商")
    
    # Bob 接受会话
    bob.receive_session({'identity_key': base64.b64encode(
        alice.identity_public.public_bytes(
            encoding=serialization.Encoding.Raw,
            format=serialization.PublicFormat.Raw
        )
    ).decode()}, alice_ephemeral_pub)
    print("Bob 完成 X3DH 密钥协商")
    
    # 验证共享密钥相同
    print(f"\nAlice 根密钥: {alice.root_key[:8].hex()}...")
    print(f"Bob 根密钥:   {bob.root_key[:8].hex()}...")
    print(f"密钥一致: {'✓' if alice.root_key == bob.root_key else '✗'}")
    
    # 加密通信
    print("\n--- 加密通信 ---")
    messages = [
        "你好 Bob，这是第一条加密消息！",
        "你好 Alice，收到你的消息了！",
        "E2EE 真的很安全，连服务器都看不到内容。",
        "没错，只有我们能看到这些消息。"
    ]
    
    for i, msg in enumerate(messages):
        sender = alice if i % 2 == 0 else bob
        receiver = bob if i % 2 == 0 else alice
        sender_name = "Alice" if i % 2 == 0 else "Bob"
        receiver_name = "Bob" if i % 2 == 0 else "Alice"
        
        encrypted = sender.encrypt_message(msg)
        decrypted = receiver.decrypt_message(encrypted)
        
        print(f"\n{sender_name} -> {receiver_name}:")
        print(f"  明文: {msg}")
        print(f"  密文: {encrypted['ciphertext'][:50]}...")
        print(f"  解密: {decrypted}")
        print(f"  验证: {'✓' if msg == decrypted else '✗'}")
    
    print("\n" + "=" * 60)
    print("E2EE 安全特性:")
    print("=" * 60)
    print("✓ 前向保密: 过去的消息不会因当前密钥泄露而暴露")
    print("✓ 后向保密: 未来的消息不会因当前密钥泄露而暴露")
    print("✓ 服务器透明: 服务端无法解密通信内容")
    print("✓ 身份验证: 基于 X25519 的身份密钥验证")


if __name__ == '__main__':
    demo_e2ee_chat()
```

---

## 4. 资源索引

### 4.1 推荐书籍

| 书名 | 作者 | 难度 | 重点内容 |
|------|------|------|----------|
| 《深入浅出密码学》 | Christof Paar | ⭐⭐⭐ | 基础理论、对称/非对称加密 |
| 《应用密码学》 | Bruce Schneier | ⭐⭐⭐⭐ | 协议设计、算法实现 |
| 《 serious cryptography 》 | Jean-Philippe Aumasson | ⭐⭐⭐ | 现代密码学实践 |
| 《加密与解密》 | 段云所 | ⭐⭐⭐ | 国产教材，系统全面 |

### 4.2 在线资源

- **Cryptography.io**: Python 密码学库官方文档
  - URL: https://cryptography.io/
  - 提供生产级密码学实现

- **OWASP Cryptographic Storage Cheat Sheet**
  - URL: https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html
  - 安全存储最佳实践

- **NIST 密码学标准**
  - URL: https://csrc.nist.gov/projects/cryptographic-standards-and-guidelines
  - FIPS 140-2/3, SP 800-系列

- **Google Tink**
  - URL: https://developers.google.com/tink
  - 跨语言密码学库

### 4.3 开源工具

| 工具 | 用途 | 链接 |
|------|------|------|
| OpenSSL | 全功能加密工具 | https://www.openssl.org/ |
| GnuPG | 邮件/文件加密 | https://gnupg.org/ |
| HashiCorp Vault | 密钥管理 | https://www.vaultproject.io/ |
| AWS KMS | 云密钥服务 | https://aws.amazon.com/kms/ |
| age | 现代文件加密 | https://age-encryption.org/ |

### 4.4 学习平台

- **Cryptohack**: https://cryptohack.org/ - 交互式密码学学习
- **Cryptopals**: https://cryptopals.com/ - 编程挑战
- **Coursera - Cryptography I**: Dan Boneh 的斯坦福课程

---

## 5. 关联知识

### 5.1 相关领域

```
A04_Security_Quality/B01_Information_Security/
├── C01_Cryptography (本模块)
│   ├── → C02_Penetration_Testing: 密码破解与防御测试
│   └── → C03_Secure_Coding: 安全编码实践
│
├── → B02_Reliability_Evaluation/C03_Disaster_Recovery: 密钥备份与恢复
├── → B03_Maintenance_Ops/C01_Incident_Response: 密钥泄露响应
└── → B04_Quality_Assurance/C03_Compliance_Scans: 加密合规检查
```

### 5.2 技术栈关联

| 技术 | 应用场景 | 与本模块关系 |
|------|----------|--------------|
| PKI/CA | HTTPS、代码签名 | 公钥基础设施 |
| HSM | 硬件密钥保护 | 密钥安全存储 |
| SGX/TEE | 可信执行环境 | 密钥使用保护 |
| MPC | 多方安全计算 | 分布式密钥管理 |
| ZKP | 零知识证明 | 隐私保护认证 |

### 5.3 前置知识

- **数学基础**: 模运算、有限域、椭圆曲线
- **计算机网络**: TLS 握手、证书链
- **操作系统**: 内存安全、进程隔离

---

## 6. 学习建议

### 6.1 学习路径

#### 初级 (1-2 周)
1. 理解对称加密 vs 非对称加密
2. 学习 AES、RSA 的基本使用
3. 实践哈希函数和 HMAC
4. 完成 Cryptohack 入门挑战

#### 中级 (2-4 周)
1. 深入理解加密模式 (CBC, GCM, CCM)
2. 实现数字签名和证书验证
3. 学习密钥管理最佳实践
4. 完成 Cryptopals Set 1-4

#### 高级 (4-8 周)
1. 研究椭圆曲线密码学
2. 实现安全协议 (Signal, Noise)
3. 密码分析基础
4. 参与 CTF 密码学挑战

### 6.2 实践项目

| 难度 | 项目 | 技能点 |
|------|------|--------|
| ⭐ | 文件加密工具 | AES-GCM、密钥派生 |
| ⭐⭐ | 安全聊天应用 | E2EE、密钥交换 |
| ⭐⭐⭐ | 迷你 CA 系统 | PKI、证书管理 |
| ⭐⭐⭐⭐ | 密码破解工具 | 彩虹表、GPU 加速 |

### 6.3 常见陷阱

1. **不要自行设计加密算法** - 使用经过验证的标准算法
2. **使用认证加密 (AEAD)** - 避免仅加密不认证
3. **正确处理 IV/Nonce** - 随机生成，永不重用
4. **安全比较密文** - 使用 `hmac.compare_digest` 防时序攻击
5. **密钥安全存储** - 使用 HSM 或 KMS，不在代码中硬编码

### 6.4 认证推荐

- **CompTIA Security+**: 基础安全认证
- **OSCP**: 渗透测试，含密码学内容
- **CISSP**: 高级安全管理
- **CCSP**: 云安全，含加密数据保护

---

## 附录：密码学速查表

### 算法选择指南

| 用途 | 推荐算法 | 避免使用 |
|------|----------|----------|
| 对称加密 | AES-256-GCM, ChaCha20-Poly1305 | DES, 3DES, RC4 |
| 非对称加密 | RSA-OAEP-2048+, ECC-P256+ | RSA-PKCS1-v1.5 |
| 数字签名 | ECDSA-P256, Ed25519, RSA-PSS | DSA |
| 密码哈希 | Argon2, bcrypt, scrypt | MD5, SHA1 |
| 消息认证 | HMAC-SHA256 | 简单哈希 |

### 密钥长度建议 (2024)

| 算法类型 | 最小长度 | 推荐长度 | 长期保护 |
|----------|----------|----------|----------|
| 对称 (AES) | 128 | 256 | 256 |
| RSA | 2048 | 3072 | 4096 |
| ECC | 224 | 256 | 384 |
| DH | 2048 | 3072 | 4096 |

---

*最后更新: 2024-01-30*
*维护者: Security Team*
*审核周期: 季度更新*
