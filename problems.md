# 问题记录

## 1. PyTorch 加载 c10.dll 失败

### 问题场景

在 Windows 的 Jupyter Notebook 中，使用 Conda 环境 `d2l-zh`。
先执行：

```python
import pandas as pd
```
再执行：
```python
import torch
```
PyTorch 会在加载 `c10.dll` 时出现以下错误：

```text
OSError: [WinError 1114] 动态链接库（DLL）初始化例程失败。
Error loading "...\torch\lib\c10.dll" or one of its dependencies.
```
### 问题原因

当前环境中的版本为：

- Pandas：2.0.3
- PyTorch：2.11.0+cu128
- Python：3.10
- 操作系统：Windows

Pandas 2.0.3 自带旧版 MSVC 运行库。当 Pandas 被优先导入时，Windows 会先加载这些旧版 DLL。

随后导入新版 PyTorch 时，PyTorch 的 `c10.dll` 与已经加载的旧版 MSVC 运行库发生冲突，因此初始化失败。
### 简单解决方法

在 Jupyter Notebook 的第一个代码单元中，先导入 PyTorch，再导入 Pandas：

```python
import torch
import pandas as pd
PyTorch 会优先加载所需的新版 MSVC 运行库，从而避免 Pandas 自带的旧版运行库与 c10.dll 发生冲突。
```
### 解决环境问题

如果希望不再依赖固定的导入顺序，可以让 Python 启动时自动加载 Windows 系统中的新版 MSVC 运行库。

### 1. 创建配置文件

在下面的目录中创建 `sitecustomize.py`：

```text
C:\Users\Administrator\.conda\envs\d2l-zh\Lib\site-packages\sitecustomize.py
```

### 2. 写入预加载代码

```python
import ctypes
import os
from pathlib import Path

_msvc_handles = []

system32 = (
    Path(os.environ.get("SystemRoot", r"C:\Windows"))
    / "System32"
)

for dll_name in (
    "vcruntime140.dll",
    "vcruntime140_1.dll",
    "msvcp140.dll",
):
    dll_path = system32 / dll_name

    if dll_path.is_file():
        handle = ctypes.WinDLL(str(dll_path))
        _msvc_handles.append(handle)
```

Python 启动时会自动执行 `sitecustomize.py`。以上代码会优先加载 Windows 系统目录中的新版 MSVC 运行库，避免 Pandas 加载旧版 DLL 后与 PyTorch 发生冲突。

### 3. 重启 Jupyter 内核

保存文件后，在 Jupyter Notebook 中选择：

```text
Kernel → Restart Kernel
```
