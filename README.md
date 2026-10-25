# GrandRDK

GrandRDK 工程仓库。本项目遵循统一的工程代码规范，任何贡献（代码、配置、文档、脚本）都必须符合仓库内规范要求。

## 快速开始

1. 阅读 **[CODING_STANDARDS.md](./CODING_STANDARDS.md)** —— 全部代码格式与工程规范的要求和标准。
2. 本地初始化（新成员必做）：

   ```bash
   pip install pre-commit
   pre-commit install
   ```

3. 提交代码前运行全量检查：

   ```bash
   pre-commit run --all-files
   ```

## 规范文件一览

| 文件 | 作用 |
| --- | --- |
| `CODING_STANDARDS.md` | 工程代码规范总文档（权威说明） |
| `.editorconfig` | 跨编辑器编码、缩进、换行基线 |
| `.clang-format` | C/C++ 格式化规则（Google 风格定制） |
| `.clang-tidy` | C/C++ 静态检查规则 |
| `.ruff.toml` | Python 风格与检查规则（PEP 8 定制） |
| `.pre-commit-config.yaml` | 提交前自动检查钩子 |
| `.gitignore` | Git 忽略规则（构建产物、密钥、IDE 配置） |

## 文档目录

- `docs/` —— 设计与用户文档（按需补充）
- `CONTRIBUTING.md` —— 贡献指南（按需补充）
