## 概述
当 Agent 需要执行代码、读写文件、安装依赖、运行分析脚本时，系统就不再只是“调用工具”，而是进入了代码执行环境管理问题。这一层通常被称为 Agent Sandbox 或 Code Interpreter。

它的目标不是单纯“能跑 Python”，而是让代码执行具备：
- 隔离性。
- 可审计性。
- 资源限制。
- 结果可回填性。

## 沙箱的核心目标

### 1. 隔离宿主环境
执行环境不能直接拥有宿主机的完整文件系统权限、网络权限和环境变量权限。

### 2. 限制资源消耗
需要控制 CPU、内存、磁盘、运行时长和输出体积，防止失控脚本占满资源。

### 3. 保留可观测性
执行的命令、标准输出、标准错误、退出码和生成文件都应可追踪。

### 4. 控制副作用
默认应使用临时工作目录和短生命周期环境，任务结束后可清理。

## 常见实现形式

### 1. 进程级隔离
在本机子进程中运行代码，并限制工作目录、超时、可见环境变量。
- 优点：实现简单。
- 缺点：安全边界较弱。

### 2. 容器级隔离
使用 Docker / Podman / Firecracker 等隔离代码执行环境。
- 优点：隔离性更好。
- 缺点：部署和运维成本更高。

### 3. 远程沙箱服务
把代码执行转移到独立服务或执行集群。
- 优点：安全边界最清晰，易统一治理。
- 缺点：工程复杂度最高。

## 对 Agent 的特殊要求
Code Interpreter 不只是跑代码，它还要支持如下闭环：
1. 模型生成代码。
2. Harness 在沙箱中执行代码。
3. 收集 stdout / stderr / 文件产物。
4. 把 observation 回填给模型。
5. 模型根据失败日志继续修复代码。

这使它天然与 [[工具调用(Action)#完整执行生命周期|工具调用]] 和 [[Harness Engineering/02_运行时循环与状态机#运行时循环的最小闭环|运行时循环]] 深度耦合。

## 高风险点
- 任意文件读取。
- 任意网络访问。
- 无限循环或超大输出。
- 安装危险依赖。
- 借助异常信息泄露宿主环境路径或秘密。

## 安全设计原则

### 1. 临时工作目录
每次执行都创建独立工作目录，任务结束可整体删除。

### 2. 最小环境变量
默认不透传宿主机环境变量，只注入必要变量。

### 3. 默认禁网
若任务不需要联网，就不应开放外网访问。

### 4. 明确可导出产物
只允许把明确声明过的文件带出沙箱。

## 规范实现示例
下面给出一个偏本地最小实现的沙箱执行器示例。它不是最强隔离方案，但具备独立工作目录、超时、环境变量控制和输出截断。

```python
from __future__ import annotations

import shutil
import subprocess
import tempfile
from dataclasses import dataclass
from pathlib import Path


@dataclass(slots=True)
class SandboxResult:
    ok: bool
    return_code: int
    stdout: str
    stderr: str
    work_dir: str


class PythonSandbox:
    def __init__(self, python_executable: str = "python", timeout_seconds: int = 10) -> None:
        self.python_executable = python_executable
        self.timeout_seconds = timeout_seconds

    def run_script(self, code: str) -> SandboxResult:
        work_dir = Path(tempfile.mkdtemp(prefix="agent_sandbox_"))
        script_path = work_dir / "main.py"
        script_path.write_text(code, encoding="utf-8")

        try:
            completed = subprocess.run(
                [self.python_executable, str(script_path)],
                cwd=work_dir,
                capture_output=True,
                text=True,
                timeout=self.timeout_seconds,
                env={},  # 不继承宿主机敏感环境变量
            )
            return SandboxResult(
                ok=completed.returncode == 0,
                return_code=completed.returncode,
                stdout=completed.stdout[:4000],
                stderr=completed.stderr[:4000],
                work_dir=str(work_dir),
            )
        except subprocess.TimeoutExpired as exc:
            return SandboxResult(
                ok=False,
                return_code=-1,
                stdout=(exc.stdout or "")[:4000] if exc.stdout else "",
                stderr="sandbox_timeout",
                work_dir=str(work_dir),
            )
        finally:
            shutil.rmtree(work_dir, ignore_errors=True)
```

## 这段代码的边界
这个实现适合笔记中的“最小可用理解”，但若进入真实生产环境，还存在明显不足：
- 仍然是宿主机进程级隔离，不够强。
- `env={}` 会让部分依赖脚本无法工作，需要按需注入白名单变量。
- 没有限制网络能力。
- 没有限制 CPU / memory / disk quota。

## 更接近生产的方案
- 使用容器运行每次代码执行任务。
- 对工作目录挂载只读输入和可写输出目录。
- 通过代理或策略层控制联网能力。
- 对包安装行为加 allowlist 或预构建镜像。
- 对生成文件做类型和大小检查后再回填给主流程。

## 与其他模块的关系
- 主流程如何消费代码执行结果，详见 [[Harness Engineering/07_端到端最小可用实现#完整示例代码|端到端最小可用实现]]。
- 预算和超时控制，详见 [[Harness Engineering/05_可靠性与安全#运行预算与熔断|可靠性与安全]]。
- 失败执行如何进入评测与回放，详见 [[Harness Engineering/06_评测与可观测性#回放与复现实验|评测与可观测性]]。
