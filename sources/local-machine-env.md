# 本机开发环境快照

> 收集时间: 2026-06-30
> 收集方式: PowerShell / nvidia-smi 命令行查询

## 硬件

### CPU
- 型号: AMD Ryzen 7 9700X 8-Core Processor
- 核心/线程: 8核 / 16线程
- 基频: 3800 MHz
- L2 Cache: 8192 KB (8 MB)
- L3 Cache: 32768 KB (32 MB)

### 内存 (RAM)
- 总量: 64 GB
- 规格: 2× 32GB Micron DDR5
- Part Number: CP32G64C40U5B.M8B3
- JEDEC 额定: 5600 MT/s
- XMP 运行频率: **6400 MT/s**（双通道 DIMM1 A + DIMM1 B）
- SMBIOSMemoryType: 34（DDR5）

### GPU
- 主显卡: NVIDIA GeForce RTX 5090 D v2
  - 显存: 24455 MiB ≈ **24 GB GDDR7**（"D"版中国定制，标准 5090 为 32GB）
  - CUDA Compute Capability: 12.0
  - 驱动版本: 596.49（WDDM: 32.0.15.9649）
- 辅助: AMD Radeon(TM) Graphics（核显，2 GB，随 CPU 集成）
- 虚拟: OrayIddDriver Device（虚拟显示驱动）

### 磁盘
- C盘: 已用 ~198 GB / 总计 ~1906 GB（SSD）
- D盘: 已用 ~634 GB / 总计 ~1908 GB

### 操作系统
- Windows 11 专业版 64-bit
- 版本: 10.0.26200

---

## 运行时环境

### Python
- 版本: **3.14.5**
- 可执行路径: `C:\Users\admin\AppData\Local\Programs\Python\Python314\python.exe`
- pip: 26.1.1
- uv: 0.11.24 (2026-06-23, x86_64-pc-windows-msvc)
- conda: 未安装
- PATH 条目:
  - `C:\Users\admin\AppData\Local\Programs\Python\Python314\`
  - `C:\Users\admin\AppData\Local\Programs\Python\Python314\Scripts\`

### Java
- JDK: **24.0.1** (Oracle, build 24.0.1+9-30)
- 路径: `C:\Program Files\Java\jdk-24`
- javac: 24.0.1
- Maven: **3.9.16** (2bdd9fd...)
  - 路径: `D:\ProgramFiles\apache-maven-3.9.16`
  - 默认语言区域: zh_CN, UTF-8
- Gradle: 未安装
- PATH 条目: `C:\Program Files\Common Files\Oracle\Java\javapath`

### Node.js
- Node: **v24.15.0**
- npm: **11.14.1**
- pnpm: 未安装
- yarn: 未安装
- nvm: 未安装
- 路径: `C:\Program Files\nodejs\`
- npm 全局 scripts: `C:\Users\admin\AppData\Roaming\npm`
