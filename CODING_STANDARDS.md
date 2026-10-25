# GrandRDK 工程代码规范

| 项目 | 内容 |
| --- | --- |
| 文档版本 | v1.0 |
| 适用范围 | GrandRDK 仓库全部源代码、配置、脚本与文档 |
| 生效日期 | 2026-10-18 |
| 维护责任 | 仓库 Code Owner / 技术负责人 |
| 配套配置 | `.editorconfig`、`.clang-format`、`.clang-tidy`、`.ruff.toml`、`.pre-commit-config.yaml`、`.gitignore` |

---

## 1. 总则

### 1.1 目的

本规范用于统一 GrandRDK 工程的代码风格、结构组织、质量要求与协作流程，使代码：

- **可读**：任何人可在不询问作者的情况下理解代码意图；
- **可维护**：局部修改不破坏整体结构，模块边界清晰；
- **可审计**：格式、命名、注释、提交均有统一且可机器校验的标准；
- **可复用**：目录组织与接口风格一致，便于模块抽取与跨项目迁移。

### 1.2 适用范围

本规范适用于本仓库内所有 **源代码（C/C++、Python、脚本）、配置文件、文档、构建脚本** 以及 **Git 提交行为**。第三方生成文件（如自动生成的头文件、文档构建产物）除外，但生成规则必须可复现。

### 1.3 规则分级

| 级别 | 关键字 | 含义 | 违反后果 |
| --- | --- | --- | --- |
| 强制 | MUST / MUST NOT | 无例外必须遵守 | Code Review 不通过；CI 阻断 |
| 强烈建议 | SHOULD / SHOULD NOT | 除非有充分理由并注明，否则应遵守 | Code Review 提出意见 |
| 建议 | MAY | 允许自行选择，鼓励符合规范的做法 | 不作强制要求 |

### 1.4 权威工具优先

**凡可由工具自动校验的规则，以工具配置为准**，本规范与工具配置冲突时，以工具输出为准。涉及的权威工具配置见第 10 章。

---

## 2. 目录与文件组织规范

### 2.1 顶层结构（MUST）

```
GrandRDK/
├── CMakeLists.txt            # 顶层构建入口（如使用 CMake）
├── README.md                 # 项目简介与快速上手
├── LICENSE                   # 开源许可（如适用）
├── CONTRIBUTING.md           # 贡献指南
├── CODING_STANDARDS.md       # 本规范
├── .editorconfig             # 跨编辑器编码与排版基线
├── .gitignore                # Git 忽略规则
├── .clang-format             # C/C++ 格式化配置
├── .clang-tidy               # C/C++ 静态检查配置
├── .ruff.toml                # Python 风格与检查配置
├── .pre-commit-config.yaml   # 提交前检查
├── include/                  # 对外公开头文件（C/C++）
│   └── grandrdk/             # 与库名一致的子目录，防止头文件冲突
├── src/                      # C/C++ 实现源码
│   ├── module_a/
│   └── module_b/
├── python/                   # Python 包源码
│   └── grandrdk/
├── tests/                    # 单元测试与集成测试
│   ├── cpp/
│   └── python/
├── scripts/                  # 构建、发布、工具脚本
├── docs/                     # 设计与用户文档
├── config/                   # 运行配置模板（不含密钥）
└── third_party/              # 第三方依赖（尽量使用包管理，避免源码入库）
```

### 2.2 目录职责（MUST）

- `include/` 只放对外公开头文件；内部实现用头文件放在 `src/` 对应模块下，禁止放入公开目录。
- 头文件必须按库名建子目录（如 `include/grandrdk/xxx.h`），引用时写 `#include "grandrdk/xxx.h"`。
- `tests/` 目录结构与被测代码保持一一对应。
- `scripts/` 下脚本必须可重复执行，禁止依赖本机特定绝对路径（详见 6.6）。
- 任何目录的创建必须服务于明确职责；空目录以 `.gitkeep` 占位并说明用途。

### 2.3 文件命名（MUST）

| 文件类型 | 命名规则 | 示例 |
| --- | --- | --- |
| C/C++ 头文件 | 小写 snake_case，`.h` | `imu_driver.h` |
| C/C++ 源文件 | 小写 snake_case，`.cpp` | `imu_driver.cpp` |
| Python 模块 | 小写 snake_case | `sensor_fusion.py` |
| 配置文件 | 小写 snake_case | `build_options.cmake` |
| Markdown 文档 | 大写加下划线或 PascalCase | `CODING_STANDARDS.md`、`GettingStarted.md` |
| 测试文件 | 被测名 + `_test` / `test_` | `imu_driver_test.cpp`、`test_sensor.py` |

### 2.4 目录与文件大小（SHOULD）

- 单个源文件建议不超过 800 行；超过应评估拆分模块。
- 单层目录下文件数超过 30 个时，应按功能拆分子目录。

---

## 3. 通用编码规范

### 3.1 字符编码（MUST）

- 全部文本文件统一 **UTF-8（无 BOM）** 编码，由 `.editorconfig` 强制。
- 源代码内禁止出现非 ASCII 字符，除非位于字符串字面量或注释中；注释推荐使用中文，但禁止使用 emoji 等非常规符号替代文字。

### 3.2 换行（MUST）

- 统一 **LF** 换行。Windows 开发环境下由 Git `core.autocrlf` 转换，入库必须为 LF。
- 文件末尾必须有且仅有一个换行符（`insert_final_newline`）。
- 禁止行尾空白（`trim_trailing_whitespace`）。

### 3.3 缩进与行长（MUST）

| 语言 | 缩进 | 行长上限 |
| --- | --- | --- |
| C/C++ | 4 空格 | 100 列 |
| Python | 4 空格 | 120 列 |
| YAML / JSON / TOML / CMake | 2 空格 | 120 列 |
| Makefile | 制表符 | 120 列 |

- 禁止使用 Tab 缩进（Makefile 除外）；同一文件内不得混用 Tab 与空格。
- 超长行必须换行，换行处对齐规则由各语言格式化工具统一处理。

### 3.4 引号与括号（SHOULD）

- 字符串优先使用双引号（C/C++ 按项目现有惯例，Python 一律双引号）。
- 括号、方括号、花括号必须成对闭合；IDE 未自动补全时人工核对。

---

## 4. 命名规范

### 4.1 命名总表（MUST）

| 对象 | 风格 | 示例 | 备注 |
| --- | --- | --- | --- |
| 目录 | snake_case | `sensor_fusion` | |
| 文件 | snake_case | `imu_driver.cpp` | |
| 类 / 结构体 / 枚举类型 | CamelCase | `ImuDriver`, `SensorConfig` | |
| 函数 / 方法 | snake_case | `read_sample()`, `get_odom()` | |
| 变量 / 参数 / 成员 | snake_case | `sample_rate`, `is_ready` | |
| 常量（constexpr/const） | k + CamelCase | `kMaxBufferSize` | C++ |
| 宏 | UPPER_SNAKE_CASE | `GRAND_RDK_MAX_RETRY` | 少用宏 |
| 枚举值 | k + CamelCase | `kStateRunning` | C++ |
| 命名空间 | 小写，通常为库名 | `namespace grandrdk` | C++ |
| Python 类 | PascalCase | `SensorFusion` | |
| Python 函数 / 变量 | snake_case | `read_sample()` | |
| Python 常量 | UPPER_SNAKE_CASE | `MAX_BUFFER_SIZE` | |
| Python 私有成员 | 单下划线前缀 | `_internal_state` | |

### 4.2 命名通用要求（MUST）

- 名称必须 **表意明确**，禁止无意义缩写（`a`、`tmp`、`x1` 仅限极短作用域内使用）。
- 禁止使用拼音缩写命名（允许拼音仅当其为既定术语，如 `pid_controller`）。
- 布尔变量 / 函数使用 `is_`、`has_`、`can_`、`should_` 前缀。
- 禁止用 `l`（小写 L）与 `O`（大写 O）作为单字符变量名。
- 缩写词在 CamelCase 中仅首字母大写（如 `PidController` 而非 `PIDController`）。
- 全局变量禁止出现（C/C++），必要全局状态必须封装为单例或显式注入。

### 4.3 命名禁用清单（MUST NOT）

- 不使用匈牙利命名法（`m_strName`、`iCount` 等）。
- 不使用与系统/库函数混淆的名称（`printf`、`open`、`read`）。
- 不以 `_` 开头定义普通标识符（保留给标准库实现）；Python 中单下划线前缀表示私有除外。
- 禁止使用双下划线开头（`__xxx`），其为编译器/解释器保留。

---

## 5. C/C++ 编码规范

本仓库 C/C++ 以 **Google C++ Style（定制版）** 为基线，以下条目为强制要求，细节以 `.clang-format` 与 `.clang-tidy` 配置为准。

### 5.1 头文件（MUST）

- 所有头文件使用 `#pragma once` 防止重复包含。
- 头文件必须自带完整依赖（能独立编译），禁止依赖包含顺序。
- 包含顺序（由 `.clang-format` 自动排序）：
  1. 本模块对应头文件（如 `imu_driver.cpp` 先包含 `imu_driver.h`）；
  2. 本项目其他头文件；
  3. 系统与第三方头文件（`<...>`）。
- 头文件中禁止 `using namespace`；禁止在头文件定义非内联的全局对象。

### 5.2 格式（MUST）

- 缩进 4 空格、行宽 100 列（见 `.clang-format`）。
- 花括号风格：**函数、类、结构体定义使用 Allman（花括号另起一行）；控制语句使用 1TBS（花括号同行）**。
- 指针与引用：星号紧贴类型（`int* p`）。
- 运算符两侧、逗号后、关键字后保留规范空白；由工具统一执行。

### 5.3 类型与对象（MUST）

- 优先 `const` 修饰：能 const 则 const，包括成员函数（`const` 成员函数）与参数。
- 优先使用 `std::string_view`（只读字符串参数）、`std::unique_ptr` / `std::shared_ptr`，禁止裸 `new`/`delete`（`std::make_unique` / `std::make_shared`）。
- 禁止 C 风格强制转换，使用 `static_cast` / `const_cast` / `reinterpret_cast` 并在必要时注释。
- 使用 `nullptr` 而非 `NULL` / `0`。
- 使用 `using` 而非 `typedef`。

### 5.4 函数（MUST）

- 函数应短小单一：一个函数只做一件事，超过 40 行应拆分。
- 输出参数使用指针并在注释中标注；优先通过返回值（结构体、`std::optional`）传递结果。
- 禁止可变参数函数（`printf` 族封装除外，须加 `__attribute__((format))` 注解）。
- 需要返回错误时优先 `std::expected` / `std::optional` 或错误码枚举，避免异常跨模块边界（嵌入式/实时场景）。

### 5.5 类设计（SHOULD）

- 类必须职责单一（SRP）；优先组合而非继承。
- 公开接口暴露在头文件，实现细节进入 `.cpp` 匿名命名空间或 `detail` 命名空间。
- 构造函数避免重活，资源管理遵循 RAII。
- 多态基类必须声明虚析构函数。

### 5.6 错误处理与日志（SHOULD）

- 所有可能失败的资源操作必须检查返回值并处理。
- 日志统一使用仓库日志组件，禁止 `printf`/`cout` 混入库代码；日志级别分级（DEBUG/INFO/WARN/ERROR）。

### 5.7 并发与实时性（SHOULD）

- 共享状态必须加锁或使用原子操作；优先使用 `std::mutex` + `std::lock_guard`。
- 实时任务路径禁止动态内存分配、禁止阻塞调用、禁止不确定循环。

### 5.8 静态检查（MUST）

- 代码必须通过 `.clang-tidy` 定义的检查（诊断、静态分析器、bugprone、performance、readability、modernize），提交前由 pre-commit / CI 执行。

---

## 6. Python 编码规范

本仓库 Python 以 **PEP 8（定制版）** 为基线，由 `ruff` 统一执行。

### 6.1 格式（MUST）

- 缩进 4 空格、行宽 120 列。
- 字符串一律双引号；导入按标准库 → 第三方 → 第一方分组排序（isort 规则）。
- 禁止 `import *`；禁止在函数内导入（顶层导入，循环依赖除外并注释）。

### 6.2 类型与注解（SHOULD）

- 新代码必须书写类型注解（函数参数、返回值、公开属性）。
- 使用 `dataclass` 表达数据容器；优先类型联合 `X | None` 表达可空。

### 6.3 命名（MUST）

- 类 PascalCase；函数/变量 snake_case；常量 UPPER_SNAKE_CASE；私有成员单下划线前缀。
- 模块名短小全小写；包名与仓库一致（`grandrdk`）。

### 6.4 文档字符串（MUST）

- 模块、公开类、公开函数必须编写 docstring（Google 风格，见第 7 章）。

### 6.5 异常与错误（SHOULD）

- 捕获异常必须指定具体异常类型，禁止裸 `except:`。
- 资源使用 `with` 语句管理（文件、锁、连接）。

### 6.6 脚本规范（MUST）

- 脚本（`scripts/` 及 CI 脚本）必须：
  - 使用 `#!/usr/bin/env python3`（或对应 shebang）；
  - 全部路径基于脚本位置或环境变量解析，禁止硬编码本机绝对路径；
  - 失败时以非零退出码结束；
  - 输出到 stdout/stderr 有清晰区分，支持 `--help`。

---

## 7. 注释与文档规范

### 7.1 注释原则（MUST）

- **注释解释"为什么"而非"是什么"**；代码本身应能说明"是什么"。
- 禁止注释掉大段代码后保留在仓库中（历史请用 Git）。
- 修改代码必须同步修改相关注释；注释与代码矛盾视为代码缺陷。
- 非 ASCII 中文注释允许，但须规范用词、句末使用中文标点。

### 7.2 C/C++ 文档注释（SHOULD）

- 公开头文件中的类、公开函数使用 Doxygen 风格注释（`///` 或 `/** */`），说明用途、参数、返回值、异常/错误。
- 复杂算法、状态机、时序逻辑必须附设计注释或指向 `docs/` 设计文档。

### 7.3 Python 文档字符串（MUST）

- 采用 Google 风格 docstring：

```python
def read_sample(sensor_id: int) -> float:
    """读取指定传感器的最新采样值。

    Args:
        sensor_id: 传感器通道编号，从 0 开始。

    Returns:
        最新采样值，单位由调用方约定。

    Raises:
        ValueError: 传感器编号超出范围。
    """
```

### 7.4 文档与代码一致性（MUST）

- `README.md` 中引用的命令、路径、参数必须与真实仓库一致。
- 接口变更（头文件、公开 API）必须同步更新相应文档与示例。

---

## 8. 版本控制与提交规范（Git）

### 8.1 分支模型（SHOULD）

- 采用主干开发 + 特性分支：`main`（受保护）为唯一发布基线；新功能从 `main` 拉取 `feature/<简述>` 分支；修复用 `fix/<简述>`。
- 禁止直接向 `main` 提交；合并必须经过 Pull Request 与 Code Review。

### 8.2 提交信息（MUST）

- 提交信息使用 **Conventional Commits** 规范：

```
<type>(<scope>): <subject>

<body>（可选，说明动机与影响）
```

- `<type>` 取值：`feat`（新功能）、`fix`（缺陷修复）、`docs`（文档）、`style`（格式）、`refactor`（重构）、`perf`（性能）、`test`（测试）、`build`（构建）、`ci`（CI）、`chore`（杂项）。
- `<subject>` 使用祈使语气、小写开头（中文可豁免）、不超过 50 字符、句末不加句号。
- 一次提交只做一件事；禁止把无关改动混入同一提交。

示例：

```
feat(sensor): 增加 IMU 零漂校准接口

- 新增 Kalman 滤波初始化流程
- 补充单元测试与文档
```

### 8.3 提交内容（MUST NOT）

- 禁止提交构建产物、IDE 配置、日志、密钥（见 `.gitignore`）。
- 禁止提交包含作者个人信息之外的大文件（>500KB，由 pre-commit 拦截）。

### 8.4 忽略规则（MUST）

- 新增忽略规则统一写入 `.gitignore`，禁止使用 `git rm --cached` 之外的方式绕过已忽略文件。

---

## 9. 质量门禁与代码审查

### 9.1 提交前（MUST，pre-commit 自动执行）

1. 行尾空白、文件末尾换行检查；
2. YAML / JSON 语法校验；
3. 大文件与合并冲突标记检查；
4. C/C++：`clang-format` 格式校验；
5. Python：`ruff check`（含自动修复）与 `ruff format`。

### 9.2 CI 门禁（MUST，合并前必须通过）

| 阶段 | 工具/命令 | 要求 |
| --- | --- | --- |
| 格式化校验 | `clang-format --dry-run --Werror` / `ruff format --check` | 零差异 |
| 静态检查 | `clang-tidy` / `ruff check` | 零 error；warning 需说明 |
| 编译 | 构建脚本（CMake 等） | 无错误、无新增 warning |
| 单元测试 | `ctest` / `pytest` | 全部通过 |
| 覆盖率 | 覆盖率工具 | 新增代码行覆盖率 ≥ 80% |

### 9.3 Code Review 检查单（SHOULD）

审查人至少核对：

- [ ] 命名、格式、注释符合本规范；
- [ ] 单函数职责单一，无复制粘贴代码；
- [ ] 错误路径全部处理，资源正确释放；
- [ ] 新增公开接口有文档注释与测试；
- [ ] 提交信息与改动内容一致；
- [ ] 无遗留调试代码、TODO 已登记（`TODO(用户名): 说明` 格式）。

### 9.4 测试要求（MUST）

- 新增功能必须配套单元测试；缺陷修复必须附带复现该缺陷的回归测试。
- 测试命名与被测对象一一对应，测试内禁止依赖执行顺序与真实时钟（需要时注入）。

---

## 10. 规范落地与工具链

### 10.1 工具链清单

| 工具 | 用途 | 配置来源 |
| --- | --- | --- |
| EditorConfig 插件（各 IDE） | 编码/缩进/换行基线 | `.editorconfig` |
| clang-format（≥ 16） | C/C++ 格式化 | `.clang-format` |
| clang-tidy（≥ 16） | C/C++ 静态检查 | `.clang-tidy` |
| ruff | Python 检查与格式化 | `.ruff.toml` |
| pre-commit | 提交前钩子 | `.pre-commit-config.yaml` |
| CMake + CTest（C/C++）、pytest（Python） | 构建与测试 | `CMakeLists.txt`、`pyproject.toml` |

### 10.2 本地初始化（MUST，新成员入职必做）

```bash
# 1. 安装 pre-commit 并激活钩子
pip install pre-commit
pre-commit install

# 2. 验证格式工具可用
clang-format --version
ruff --version
```

### 10.3 常用命令速查

```bash
# C/C++ 单文件格式化
clang-format -i src/imu_driver.cpp

# C/C++ 全量校验（不写回）
find src include -name '*.cpp' -o -name '*.h' | xargs clang-format --dry-run --Werror

# C/C++ 静态检查
clang-tidy -p build/compile_commands.json src/imu_driver.cpp

# Python 检查与格式化
ruff check . && ruff format .

# 提交前全量检查
pre-commit run --all-files
```

### 10.4 违规处理

- CI 拦截项（9.2）：由提交者修复后重新提交，禁止跳过门禁合并。
- 规范文档与工具配置冲突：以工具配置为准，并提交 Issue 修正文档。
- 反复违反规范：由 Code Owner 在 Review 中明确指出，必要时拆分提交回退重做。

### 10.5 规范更新流程

1. 任何成员可提出修改建议（Issue 或 PR）；
2. 变更必须说明动机与对存量代码的影响；
3. 经 Code Owner 评审并同步更新全部配套配置后生效；
4. 生效后文档版本号递增，并在变更日志中记录。

---

## 附录 A：命名速查表（速览版）

| 场景 | 写法 | 对/错示例 |
| --- | --- | --- |
| 类/结构体 | CamelCase | `SensorFusion` / ~~`sensor_fusion`~~ |
| 函数 | snake_case | `read_sample` / ~~`ReadSample`~~ |
| 常量 | k + CamelCase（C++）/ UPPER（Python） | `kMaxSize` / `MAX_SIZE` |
| 宏 | UPPER_SNAKE_CASE | `GRAND_RDK_RETRY_MAX` |
| 布尔 | is/has/can 前缀 | `is_ready` / ~~`ready`~~ |

## 附录 B：格式化一页纸

- 缩进：C/C++ 4 空格；Python 4 空格；YAML/JSON 2 空格；Makefile Tab。
- 行长：C/C++ 100 列；Python 120 列。
- 换行：LF；文件末尾一个换行；无行尾空白。
- 花括号：函数/类定义另起一行；控制语句同行。
- 指针：星号靠类型（`int* p`）。
- 编码：UTF-8 无 BOM。
