# C03_Version_Control - 版本控制

> 知识文档的版本管理、变更追踪与协作同步机制

---

## 1. 主题定位

### 1.1 定义与重要性

版本控制（Version Control）是管理文档、代码或其他数字资产变更历史的系统。在知识管理领域，版本控制不仅用于追踪内容的历史演变，还支持多人协作、冲突解决、内容回溯等关键功能。

对于知识库而言，版本控制的重要性体现在：

| 维度 | 价值 |
|------|------|
| **历史追溯** | 查看任何文档的任何历史版本 |
| **变更审计** | 了解谁在何时做了什么修改 |
| **错误恢复** | 轻松回滚到之前的稳定版本 |
| **协作支持** | 多人同时编辑同一文档 |
| **分支实验** | 安全地尝试不同内容方向 |
| **备份冗余** | 分布式存储防止数据丢失 |

### 1.2 版本控制发展

```
版本控制系统演进：

本地版本控制（1980s）
├─ SCCS (1972) - Source Code Control System
├─ RCS (1982) - Revision Control System
└─ 特点: 单机、本地管理

集中式版本控制（1990s）
├─ CVS (1990) - Concurrent Versions System
├─ SVN (2000) - Subversion
├─ Perforce (1995)
└─ 特点: 中央服务器、客户端-服务器架构

分布式版本控制（2000s+）
├─ BitKeeper (2000) - Linux内核曾使用
├─ Git (2005) - Linus Torvalds开发
├─ Mercurial (2005)
└─ 特点: 每个副本都是完整仓库、离线工作

内容寻址存储（2010s+）
├─ IPFS - 星际文件系统
├─ Dat - 分布式数据共享
└─ 特点: 内容寻址、去中心化

实时协作编辑（2010s+）
├─ Google Docs (2006)
├─ Notion (2016)
├─ Obsidian Sync (2020)
└─ 特点: OT/CRDT算法、实时同步
```

### 1.3 知识管理中的版本控制

```
知识库版本控制应用场景：

个人层面:
├─ 笔记历史回溯 - 查看想法的演变过程
├─ 写作版本迭代 - 比较不同草稿版本
├─ 知识归档 - 长期保存知识资产
└─ 多设备同步 - 跨平台访问知识库

团队协作:
├─ 文档协作编辑 - 多人同时编写
├─ 审核工作流 - 内容发布前的评审
├─ 知识审查 - 定期更新和验证知识
└─ 权限管理 - 不同角色的访问控制

组织层面:
├─ 知识基线 - 标记重要的知识里程碑
├─ 合规审计 - 满足法规要求
├─ 知识迁移 - 系统间的知识转移
└─ 灾难恢复 - 备份和恢复策略
```

---

## 2. 核心概念

### 2.1 Git基础原理

Git是最流行的分布式版本控制系统，理解其原理对知识库管理至关重要。

#### 2.1.1 Git对象模型

```
Git对象类型：

1. Blob (二进制大对象)
   - 存储文件内容
   - 内容寻址: SHA1(内容) = 对象ID
   - 不可变，内容相同则ID相同

2. Tree (树对象)
   - 表示目录结构
   - 包含: 文件名、模式、类型、对象ID
   - 递归引用其他tree或blob

3. Commit (提交对象)
   - 保存版本快照
   - 包含: 作者、时间、提交信息、父提交、tree ID
   - 形成历史链

4. Tag (标签对象)
   - 指向特定提交的可读引用
   - 轻量标签: 简单引用
   - 附注标签: 独立对象，可签名

对象关系示例：
                    ┌─────────────┐
                    │   commit    │
                    │   v1.0      │
                    │  message    │
                    └──────┬──────┘
                           │ tree
                           ▼
                    ┌─────────────┐
                    │    tree     │
                    │  (根目录)   │
                    └──────┬──────┘
                           │
           ┌───────────────┼───────────────┐
           │               │               │
           ▼               ▼               ▼
    ┌───────────┐   ┌───────────┐   ┌───────────┐
    │   blob    │   │   tree    │   │   blob    │
    │ README.md │   │  (src/)   │   │  LICENSE  │
    └───────────┘   └─────┬─────┘   └───────────┘
                          │
                ┌─────────┴─────────┐
                ▼                   ▼
         ┌───────────┐       ┌───────────┐
         │   blob    │       │   blob    │
         │ main.py   │       │ utils.py  │
         └───────────┘       └───────────┘
```

#### 2.1.2 Git工作流程

```
Git三棵树（Three Trees）：

┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Working    │     │    Index    │     │   HEAD      │
│  Directory  │ ←→  │   (Stage)   │  →  │  (Repository)│
│  (工作区)    │     │  (暂存区)    │     │  (版本库)    │
└─────────────┘     └─────────────┘     └─────────────┘
     实际文件              下次提交             最新提交
     的修改              的内容快照           的版本快照

状态流转：
                         git add
Untracked/Unmodified ─────────────→ Staged
                                             │
                                             │ git commit
                                             ▼
Modified ←───────────────────────────── Committed
   │
   │ edit
   ▼
Modified → git add → Staged → git commit → Committed
```

### 2.2 分支策略

#### 2.2.1 Git Flow

```
Git Flow分支模型：

main (master) ─────────────────────────────────────────────
    │                                                    │
    │  hotfix-1.1.1                                       │
    ▼────────┐                                           ▼
┌────────┐   │    merge                          tag v1.1.1
│ v1.0.0 │   ▼                                           │
└────────┘←───────────────────────────────────────────────┘
    │                                                    │
    │ develop ────────────────────────────────────────────┤
    │    │    \                                          │
    │    │     \  feature/auth                           │
    │    │      \───────┐                                │
    │    │               │                               │
    │    │ feature/api   │                               │
    │    │────────┐      │                               │
    │    │        │      │                               │
    │    ▼        ▼      ▼                               ▼
    │   merge    merge  merge                      release/v1.1
    │    │        │      │                               │
    │    └────────┴──────┘                               │
    │                                                    ▼
    └──────────────────────────────────────────────── merge
                                                       │
                                                 tag v1.1.0

分支类型：
- main: 生产代码，永远可部署
- develop: 开发主线，集成所有功能
- feature/*: 功能开发分支，从develop创建
- release/*: 发布准备分支，从develop创建
- hotfix/*: 紧急修复分支，从main创建
```

#### 2.2.2 GitHub Flow（简化版）

```
GitHub Flow工作流：

main ────────────────────────────────────────────────────
  │                                                      │
  │  feature/user-auth                                   │
  │───────┐                                              │
  │       │ 1. 从main创建分支                             │
  │       ▼                                              │
  │   ┌─────────┐                                        │
  │   │ 开发功能 │ 2. 提交更改                            │
  │   └────┬────┘                                        │
  │        │ 3. 创建Pull Request                          │
  │        │    - 代码审查                                │
  │        │    - 自动化测试                              │
  │        │    - 讨论反馈                                │
  │        ▼                                              │
  │   ┌─────────┐                                        │
  │   │ 合并到main│ 4. 审查通过后合并                       │
  │   └────┬────┘    (使用squash或rebase)                  │
  │        │                                              │
  └────────┴──────────────────────────────────────────────┘
           5. 部署到生产环境
```

### 2.3 冲突解决

```
合并冲突场景：

场景1: 同一文件不同行修改
main:    A ──── B ──── C
          \            /
feature    └──── X ───┘
结果: 可以自动合并 → A-B-C-X

场景2: 同一文件同一行修改（冲突）
main:    A ──── B(text: "hello") ──── C
          \                          /
feature    └──── X(text: "world") ──┘
结果: 冲突！需要手动解决

冲突标记：
<<<<<<< HEAD
内容来自当前分支
=======
内容来自合并分支
>>>>>>> feature-branch

解决步骤：
1. 打开冲突文件
2. 编辑文件，保留需要的修改
3. 删除冲突标记
4. git add <file>
5. git commit
```

---

## 3. 技术实践

### 3.1 Git知识库管理

```python
import subprocess
import os
from pathlib import Path
from datetime import datetime
from typing import List, Dict, Optional, Tuple
import json

class KnowledgeBaseGitManager:
    """知识库的Git版本管理"""
    
    def __init__(self, repo_path: str):
        self.repo_path = Path(repo_path)
        self._ensure_repo()
    
    def _ensure_repo(self):
        """确保是Git仓库"""
        git_dir = self.repo_path / '.git'
        if not git_dir.exists():
            self._run_git_command(['init'])
            # 初始化Git LFS（如果需要管理大文件）
            self._setup_lfs()
    
    def _setup_lfs(self):
        """设置Git LFS"""
        # 追踪大文件类型
        lfs_patterns = [
            '*.pdf',
            '*.zip',
            '*.png',
            '*.jpg',
            '*.mp4',
            '*.psd'
        ]
        for pattern in lfs_patterns:
            self._run_git_command(['lfs', 'track', pattern])
    
    def _run_git_command(self, args: List[str]) -> Tuple[str, str]:
        """运行Git命令"""
        cmd = ['git'] + args
        result = subprocess.run(
            cmd,
            cwd=self.repo_path,
            capture_output=True,
            text=True
        )
        return result.stdout, result.stderr
    
    def add_file(self, filepath: str):
        """添加文件到暂存区"""
        rel_path = Path(filepath).relative_to(self.repo_path)
        self._run_git_command(['add', str(rel_path)])
    
    def commit(self, message: str, author: str = None, 
               email: str = None) -> str:
        """
        提交更改
        
        参数:
            message: 提交信息
            author: 作者名称（可选）
            email: 作者邮箱（可选）
        """
        env_args = []
        if author:
            env_args.extend(['-c', f'user.name={author}'])
        if email:
            env_args.extend(['-c', f'user.email={email}'])
        
        stdout, stderr = self._run_git_command(
            env_args + ['commit', '-m', message]
        )
        
        # 提取commit hash
        if stdout:
            lines = stdout.strip().split('\n')
            for line in lines:
                if line.startswith('[main'):
                    # [main abc1234] message
                    parts = line.split()
                    if len(parts) >= 2:
                        return parts[1].rstrip(']')
        return ""
    
    def get_history(self, filepath: str = None, 
                   max_entries: int = 50) -> List[Dict]:
        """
        获取提交历史
        
        参数:
            filepath: 特定文件的历史（可选）
            max_entries: 最大条目数
        """
        format_str = '%H|%an|%ae|%ad|%s'
        cmd = [
            'log',
            f'--pretty=format:{format_str}',
            f'-n{max_entries}',
            '--date=iso'
        ]
        
        if filepath:
            cmd.extend(['--follow', '--', filepath])
        
        stdout, _ = self._run_git_command(cmd)
        
        history = []
        for line in stdout.strip().split('\n'):
            if '|' in line:
                parts = line.split('|', 4)
                if len(parts) == 5:
                    history.append({
                        'hash': parts[0],
                        'author': parts[1],
                        'email': parts[2],
                        'date': parts[3],
                        'message': parts[4]
                    })
        
        return history
    
    def get_diff(self, commit_hash: str = None, 
                 filepath: str = None) -> str:
        """
        获取差异
        
        参数:
            commit_hash: 与指定提交比较（默认与工作区比较）
            filepath: 特定文件的差异
        """
        if commit_hash:
            cmd = ['diff', f'{commit_hash}^..{commit_hash}']
        else:
            cmd = ['diff', 'HEAD']
        
        if filepath:
            cmd.extend(['--', filepath])
        
        stdout, _ = self._run_git_command(cmd)
        return stdout
    
    def checkout_version(self, commit_hash: str, 
                        filepath: str = None):
        """
        检出特定版本
        
        参数:
            commit_hash: 提交哈希
            filepath: 特定文件（可选，默认检出整个提交）
        """
        if filepath:
            self._run_git_command(['checkout', commit_hash, '--', filepath])
        else:
            # 创建临时分支
            self._run_git_command(['checkout', '-b', f'temp-{commit_hash[:7]}', commit_hash])
    
    def create_branch(self, branch_name: str, 
                     base: str = 'main'):
        """创建分支"""
        self._run_git_command(['checkout', '-b', branch_name, base])
    
    def merge_branch(self, branch_name: str, 
                    strategy: str = 'merge'):
        """
        合并分支
        
        参数:
            branch_name: 要合并的分支
            strategy: 合并策略 ('merge', 'squash', 'rebase')
        """
        if strategy == 'squash':
            self._run_git_command(['merge', '--squash', branch_name])
        elif strategy == 'rebase':
            self._run_git_command(['rebase', branch_name])
        else:
            self._run_git_command(['merge', branch_name])
    
    def get_stats(self) -> Dict:
        """获取仓库统计信息"""
        stats = {}
        
        # 总提交数
        stdout, _ = self._run_git_command(['rev-list', '--count', 'HEAD'])
        stats['total_commits'] = int(stdout.strip() or 0)
        
        # 贡献者统计
        stdout, _ = self._run_git_command([
            'shortlog', '-sn', 'HEAD'
        ])
        contributors = []
        for line in stdout.strip().split('\n'):
            parts = line.strip().split('\t')
            if len(parts) == 2:
                contributors.append({
                    'commits': int(parts[0]),
                    'name': parts[1]
                })
        stats['contributors'] = contributors
        
        # 文件统计
        stdout, _ = self._run_git_command(['ls-files'])
        files = stdout.strip().split('\n')
        stats['total_files'] = len([f for f in files if f])
        
        # 按扩展名统计
        extensions = {}
        for f in files:
            if '.' in f:
                ext = f.split('.')[-1]
                extensions[ext] = extensions.get(ext, 0) + 1
        stats['file_types'] = extensions
        
        return stats
    
    def search_content(self, query: str, 
                      path: str = None) -> List[Dict]:
        """
        搜索内容历史
        
        使用git grep搜索所有版本中的内容
        """
        cmd = ['grep', '-n', '-i', query]
        if path:
            cmd.extend(['--', path])
        
        stdout, _ = self._run_git_command(cmd)
        
        results = []
        for line in stdout.strip().split('\n'):
            if ':' in line:
                parts = line.split(':', 2)
                if len(parts) >= 2:
                    results.append({
                        'file': parts[0],
                        'line': int(parts[1]),
                        'content': parts[2] if len(parts) > 2 else ''
                    })
        
        return results


# 自动化提交示例
class AutoCommitHandler:
    """自动提交处理器"""
    
    def __init__(self, git_manager: KnowledgeBaseGitManager):
        self.git = git_manager
    
    def auto_commit_changes(self, author: str = "Knowledge Bot"):
        """自动提交所有更改"""
        # 检查是否有更改
        stdout, _ = self.git._run_git_command(['status', '--porcelain'])
        
        if not stdout.strip():
            print("没有待提交的更改")
            return None
        
        # 添加所有更改
        self.git._run_git_command(['add', '-A'])
        
        # 生成提交信息
        changed_files = stdout.strip().split('\n')
        file_count = len(changed_files)
        
        if file_count == 1:
            # 单个文件的详细提交信息
            status_line = changed_files[0]
            status = status_line[:2]
            filename = status_line[3:]
            
            if 'A' in status:
                message = f"docs: 添加文档 {filename}"
            elif 'D' in status:
                message = f"docs: 删除文档 {filename}"
            else:
                message = f"docs: 更新文档 {filename}"
        else:
            message = f"docs: 更新 {file_count} 个文件"
        
        # 提交
        commit_hash = self.git.commit(message, author=author)
        print(f"已提交: {commit_hash}")
        return commit_hash
    
    def scheduled_backup(self):
        """定时备份"""
        from datetime import datetime
        
        timestamp = datetime.now().strftime('%Y-%m-%d %H:%M')
        message = f"backup: 自动备份 {timestamp}"
        
        self.git._run_git_command(['add', '-A'])
        stdout, stderr = self.git._run_git_command(['commit', '-m', message])
        
        if 'nothing to commit' in stderr:
            print("没有需要备份的更改")
        else:
            print(f"备份完成: {timestamp}")
```

### 3.2 内容差异可视化

```python
import difflib
from typing import List, Tuple
import html

class DiffVisualizer:
    """差异可视化工具"""
    
    @staticmethod
    def unified_diff(old_text: str, new_text: str, 
                    old_name: str = 'old', 
                    new_name: str = 'new') -> str:
        """
        生成统一差异格式
        
        输出格式：
        --- old
        +++ new
        @@ -1,3 +1,4 @@
         line 1
        -line 2 removed
        +line 2 added
        +line 3 added
         line 4
        """
        old_lines = old_text.splitlines(keepends=True)
        new_lines = new_text.splitlines(keepends=True)
        
        # 确保每行以换行符结尾
        if old_lines and not old_lines[-1].endswith('\n'):
            old_lines[-1] += '\n'
        if new_lines and not new_lines[-1].endswith('\n'):
            new_lines[-1] += '\n'
        
        diff = difflib.unified_diff(
            old_lines, new_lines,
            fromfile=old_name,
            tofile=new_name,
            lineterm=''
        )
        
        return ''.join(diff)
    
    @staticmethod
    def html_diff(old_text: str, new_text: str) -> str:
        """生成HTML格式的差异"""
        differ = difflib.HtmlDiff(wrapcolumn=80)
        
        old_lines = old_text.splitlines()
        new_lines = new_text.splitlines()
        
        html_output = differ.make_table(
            old_lines, new_lines,
            fromdesc='旧版本',
            todesc='新版本'
        )
        
        return f"""
        <!DOCTYPE html>
        <html>
        <head>
            <style>
                table.diff {{
                    font-family: monospace;
                    border-collapse: collapse;
                    width: 100%;
                }}
                table.diff td {{
                    padding: 4px 8px;
                    white-space: pre-wrap;
                    word-wrap: break-word;
                }}
                .diff_header {{
                    background-color: #f0f0f0;
                    font-weight: bold;
                }}
                .diff_add {{
                    background-color: #d4edda;
                    color: #155724;
                }}
                .diff_chg {{
                    background-color: #fff3cd;
                    color: #856404;
                }}
                .diff_sub {{
                    background-color: #f8d7da;
                    color: #721c24;
                }}
            </style>
        </head>
        <body>
            {html_output}
        </body>
        </html>
        """
    
    @staticmethod
    def word_diff(old_text: str, new_text: str) -> List[Dict]:
        """
        逐词差异对比
        
        返回格式:
        [
            {'type': 'equal', 'text': '相同文本'},
            {'type': 'delete', 'text': '删除的文本'},
            {'type': 'insert', 'text': '新增的文本'}
        ]
        """
        old_words = old_text.split()
        new_words = new_text.split()
        
        sm = difflib.SequenceMatcher(None, old_words, new_words)
        
        result = []
        for op, i1, i2, j1, j2 in sm.get_opcodes():
            if op == 'equal':
                result.append({
                    'type': 'equal',
                    'text': ' '.join(old_words[i1:i2])
                })
            elif op == 'delete':
                result.append({
                    'type': 'delete',
                    'text': ' '.join(old_words[i1:i2])
                })
            elif op == 'insert':
                result.append({
                    'type': 'insert',
                    'text': ' '.join(new_words[j1:j2])
                })
            elif op == 'replace':
                result.append({
                    'type': 'delete',
                    'text': ' '.join(old_words[i1:i2])
                })
                result.append({
                    'type': 'insert',
                    'text': ' '.join(new_words[j1:j2])
                })
        
        return result
```

### 3.3 知识库同步策略

```python
import hashlib
import json
from datetime import datetime
from typing import Dict, List, Optional
from dataclasses import dataclass

@dataclass
class SyncRecord:
    """同步记录"""
    file_path: str
    local_hash: str
    remote_hash: str
    local_mtime: datetime
    remote_mtime: datetime
    sync_status: str  # 'synced', 'local_ahead', 'remote_ahead', 'conflict'


class KnowledgeBaseSync:
    """知识库同步管理"""
    
    def __init__(self, local_path: str, remote_url: str = None):
        self.local_path = local_path
        self.remote_url = remote_url
        self.sync_state_file = f"{local_path}/.sync_state.json"
        self.sync_state = self._load_sync_state()
    
    def _load_sync_state(self) -> Dict:
        """加载同步状态"""
        try:
            with open(self.sync_state_file, 'r') as f:
                return json.load(f)
        except FileNotFoundError:
            return {}
    
    def _save_sync_state(self):
        """保存同步状态"""
        with open(self.sync_state_file, 'w') as f:
            json.dump(self.sync_state, f, indent=2)
    
    def _compute_file_hash(self, filepath: str) -> str:
        """计算文件哈希"""
        hasher = hashlib.md5()
        with open(filepath, 'rb') as f:
            for chunk in iter(lambda: f.read(4096), b""):
                hasher.update(chunk)
        return hasher.hexdigest()
    
    def scan_local_changes(self) -> List[SyncRecord]:
        """扫描本地更改"""
        records = []
        
        import os
        for root, dirs, files in os.walk(self.local_path):
            # 跳过隐藏目录
            dirs[:] = [d for d in dirs if not d.startswith('.')]
            
            for filename in files:
                if filename.startswith('.'):
                    continue
                
                filepath = os.path.join(root, filename)
                rel_path = os.path.relpath(filepath, self.local_path)
                
                current_hash = self._compute_file_hash(filepath)
                current_mtime = datetime.fromtimestamp(os.path.getmtime(filepath))
                
                # 与上次同步状态比较
                last_state = self.sync_state.get(rel_path, {})
                last_hash = last_state.get('hash', '')
                
                if current_hash != last_hash:
                    records.append(SyncRecord(
                        file_path=rel_path,
                        local_hash=current_hash,
                        remote_hash=last_state.get('remote_hash', ''),
                        local_mtime=current_mtime,
                        remote_mtime=datetime.fromisoformat(last_state.get('remote_mtime', '1970-01-01')),
                        sync_status='local_ahead'
                    ))
        
        return records
    
    def resolve_conflict(self, filepath: str, 
                        strategy: str = 'manual') -> str:
        """
        解决冲突
        
        参数:
            filepath: 冲突文件路径
            strategy: 解决策略 ('local', 'remote', 'merge', 'manual')
        
        返回:
            冲突解决后的文件路径
        """
        if strategy == 'local':
            # 保留本地版本
            return filepath
        elif strategy == 'remote':
            # 使用远程版本
            return filepath  # 实际应下载远程版本
        elif strategy == 'merge':
            # 尝试自动合并
            # 这里简化处理，实际应实现三方合并
            return filepath
        else:
            # 手动解决 - 创建冲突标记文件
            conflict_path = f"{filepath}.conflict"
            return conflict_path
    
    def sync(self) -> Dict:
        """
        执行同步
        
        返回同步报告
        """
        report = {
            'synced': [],
            'uploaded': [],
            'downloaded': [],
            'conflicts': [],
            'errors': []
        }
        
        # 1. 扫描本地更改
        local_changes = self.scan_local_changes()
        
        # 2. 获取远程更改（模拟）
        # 实际实现应与远程服务器交互
        remote_changes = []  # self.fetch_remote_changes()
        
        # 3. 分析同步状态
        for record in local_changes:
            if record.sync_status == 'local_ahead':
                # 本地领先 - 上传
                report['uploaded'].append(record.file_path)
                # 更新同步状态
                self.sync_state[record.file_path] = {
                    'hash': record.local_hash,
                    'remote_hash': record.local_hash,
                    'mtime': record.local_mtime.isoformat(),
                    'remote_mtime': record.local_mtime.isoformat()
                }
        
        self._save_sync_state()
        
        return report
```

---

## 4. 资源索引

### 4.1 Git工具与资源

| 资源 | 类型 | 说明 |
|------|------|------|
| **Git官方文档** | 文档 | git-scm.com/doc |
| **Pro Git** | 书籍 | 免费在线Git教程 |
| **GitHub Learning Lab** | 教程 | 交互式Git学习 |
| **Oh My Zsh Git插件** | 工具 | 增强命令行体验 |
| **GitKraken** | GUI工具 | 跨平台Git客户端 |
| **SourceTree** | GUI工具 | 免费Git客户端 |

### 4.2 知识管理同步方案

| 方案 | 特点 | 适用场景 |
|------|------|----------|
| **Git + GitHub** | 免费、功能强大 | 技术用户 |
| **Obsidian Sync** | 端到端加密 | Obsidian用户 |
| **Syncthing** | P2P同步、去中心化 | 隐私优先 |
| **Dropbox** | 简单易用 | 普通用户 |
| **iCloud Drive** | 苹果生态 | macOS/iOS用户 |
| **Self-hosted Git** | 完全控制 | 企业/高级用户 |

---

## 5. 关联知识

```
C03_Version_Control
├── A02_Programming_Languages/B03_Git/C01_Core_Commands
│   └── Git核心命令与技术
├── A07_Knowledge_Ops/B02_Content_Strategy/C01_Atomic_Notes
│   └── 原子笔记的版本管理
└── A07_Knowledge_Ops/B02_Content_Strategy/C02_Cross-Linking
    └── 链接变更的版本追踪
```

---

## 6. 学习建议

### 6.1 Git学习路径

**第1周**：掌握基础命令（init, add, commit, status, log）
**第2周**：学习分支和合并（branch, checkout, merge, rebase）
**第3周**：远程协作（remote, push, pull, clone）
**第4周**：高级功能（stash, cherry-pick, reflog）

### 6.2 知识库版本控制最佳实践

1. **小步提交**：经常提交，每次提交只做一件事
2. **清晰的提交信息**：描述"为什么"而不是"做了什么"
3. **使用分支**：实验性内容使用分支隔离
4. **定期推送**：防止数据丢失
5. **标签重要版本**：为里程碑版本打标签

---

*最后更新：2026-01-30*
*维护者：ZCO Knowledge Ops Team*
