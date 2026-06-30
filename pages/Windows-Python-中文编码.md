---
type: concept
aliases: [PYTHONUTF8, PYTHONIOENCODING, Windows终端中文乱码, Windows编码坑]
tags: [windows, python, encoding, 踩坑, cli]
status: stable
created: 2026-06-29
updated: 2026-06-29
source: "[[boss-求职工作区]]"
related: ["[[机器可读的-Agent-工具约定]]"]
---

# Windows · Python 中文编码坑

## 摘要 / Summary

在 Windows 终端跑会输出中文(或任意非 ASCII)的 Python CLI 时,默认编码常导致**乱码或 `UnicodeEncodeError`**。根因:Windows 控制台默认代码页不是 UTF-8,Python 的 `stdout` 沿用了它。

## 解法 / Fix

调用前设两个环境变量,强制 Python 全程走 UTF-8:

```bash
PYTHONIOENCODING=utf-8 PYTHONUTF8=1 <python程序或CLI>
```

- `PYTHONUTF8=1` —— 开启 Python **UTF-8 模式**(Py3.7+),让文件系统/标准流默认 UTF-8,不再看系统区域设置。
- `PYTHONIOENCODING=utf-8` —— 显式钉死 `stdin/stdout/stderr` 编码,双保险。

## 要点 / Key points

- 这是**调用方**的修复,不需要改被调程序的源码——适合操作别人发布的 CLI。
- 退路:把输出**重定向到文件再读**,绕开控制台代码页(`cmd ... > out.json`,然后读文件)。
- 用 Bash 工具(Git Bash)调用比 PowerShell 更省心,但仍建议带上这两个变量。
- 同类现象也出现在:中文路径、`print()` 中文、JSON 里的中文字段——UTF-8 模式一并解决。
- 不只 Python:任何在 Windows 上输出非 ASCII 的 CLI 都要先想到「控制台代码页」这一层。

## 关联 / Connections

- 实战来源:[[boss-cli]] 在 Windows 上解析中文 JSON 信封时依赖此约定;见 [[boss-求职工作区]]。
- 解析机器可读输出的更通用约定见 [[机器可读的-Agent-工具约定]]。
