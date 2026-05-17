# VS Code + STM32CubeMX 开发模板

基于 **VS Code + STM32CubeMX + CMake + Ninja + arm-none-eabi-gcc** 的轻量级开发流，彻底抛弃 STM32CubeIDE。

本模板以 STM32F407VET6 为例，但适用于 CubeMX 支持的大部分 STM32 芯片（F0/F1/F2/F3/F4/G0/G4/H5/H7/L0/L1/L4/L5/U5/WB/WL 等），只要生成时选择 CMake 工具链即可。

支持 Windows 和 macOS 双平台。

---

## 目录结构

```
ST_vscode_template/
├── mac/                    # macOS 平台 VS Code Profile 配置模板
│   ├── settings.json       ← 粘贴到 VS Code ST profile 全局设置
│   └── tasks.json          ← 粘贴到 VS Code ST profile 全局任务
├── win/
└── readme.md
```

> 根目录 `.gitignore` 中写的是 `local`，表示整个 `local/` 目录不会被 git 追踪。`local/` 是我放在本地的模版工程。clone 本仓库的人无需关心它，按下方步骤从 CubeMX 生成自己的工程即可。
>
> 你自己的工程根目录需要另外放一份 `.gitignore`，内容见第四步。

`settings.json` 和 `tasks.json` 需粘贴到 VS Code 的 **ST profile** 全局配置中，不是工程内的 `.vscode/`。通过此举可以不用每次新建工程都重新配置 json。

---

## 前置依赖

| 工具 | 说明 |
|------|------|
| STM32CubeMX | ST 官网下载，用于生成 CMake 工程和 HAL 驱动代码 |
| arm-none-eabi-gcc | ARM 嵌入式交叉编译器，**必须包含 newlib** |
| CMake (≥3.22) | 构建系统 |
| Ninja | 构建后端 |
| VS Code 扩展 | `llvm-vs-code-extensions.vscode-clangd`、`ms-vscode.cmake-tools`、`ms-vscode.cpptools` |

> **编译器注意：** 在 macOS 上 Homebrew 的 `arm-none-eabi-gcc` 是 `--without-headers` 编译的，缺少 newlib，**不可用**。推荐使用 xpack 分发版（见下方安装步骤）。
>
> C_Cpp 扩展仅用于调试，其 intellisense 会在 `settings.json` 中禁用（与 clangd 冲突）。

---

## macOS 配置步骤

### 第一步：安装工具链

```bash
# 安装 xpm（xpack 包管理器）
npm install -g xpm

# 安装 arm-none-eabi-gcc（自带 newlib）
xpm install @xpack-dev-tools/arm-none-eabi-gcc@latest --global
```

记下安装路径，例如 `~/Library/xPacks/@xpack-dev-tools/arm-none-eabi-gcc/15.2.1-1.1.1/.content/bin`。

### 第二步：安装 VS Code 扩展

必装：**clangd**、**CMake Tools**、**C/C++**（仅调试）。

其余按需（Cortex-Debug 烧录等，macOS 暂不涉及）。

### 第三步：创建 VS Code Profile

在 VS Code 中为 ST 开发单独建一个 **Profile**，`settings.json`、`tasks.json` 配置在 Profile 级别，所有 STM32 工程共享。

1. VS Code 左下角齿轮 → **Profiles** → **Create Profile**
2. 输入名称（如 `ST`），选择从空白创建
3. 在该 Profile 下打开命令面板 (`Cmd+Shift+P`)
4. 搜索 **Open User Settings (JSON)**，将 `mac/settings.json` 的内容粘贴进去
5. 将 `/path/to/arm-none-eabi-gcc/bin` 替换为第一步的实际路径
6. 搜索 **Open User Tasks**，将 `mac/tasks.json` 的内容粘贴进去

### 第四步：配置 clangd（一次性）

创建 `~/.config/clangd/config.yaml`：

```yaml
Diagnostics:
  Suppress:
    - unused-includes
    - unknown_typename
    - unknown_typename_suggest
    - typename_requires_specqual
```

这几条仅抑制 clangd 对嵌入式 CMSIS/HAL 代码的误报波浪线，不影响编译。

---

## 新建工程流程

### 1. CubeMX 生成代码

- 打开 **STM32CubeMX**（不是 CubeIDE），选择芯片（以 **STM32F407VET6** 为例）
- 配好引脚、时钟、外设
- **Project Manager → Project → Toolchain / IDE → CMake**
- 点击 **GENERATE CODE**

CubeMX 会自动生成 `CMakeLists.txt`、`CMakePresets.json`、`cmake/gcc-arm-none-eabi.cmake`、`cmake/stm32cubemx/CMakeLists.txt` 及所有 HAL 驱动文件。

> 其他 STM32 芯片同理，只要生成时选了 CMake，后续流程完全一致。

### 2. VS Code 打开工程

在 **ST Profile** 下打开工程文件夹。

### 3. 添加 `.gitignore`

在工程根目录新建 `.gitignore`，写入：

```
build
mx.scratch
compile_commands.json
.cache
.vscode
.settings
```

### 4. Configure

运行 **CMake Configure (Debug)** 任务（`Cmd+Shift+P` → `Tasks: Run Task`），或在 CMake Tools 侧边栏点击 Configure。

Configure 完成后会自动在工程根目录创建 `compile_commands.json` 软链接（→ `build/Debug/compile_commands.json`），clangd 能立即开始工作。

### 5. 开始编码

clangd 通过 `compile_commands.json` 拿到所有编译参数，代码补全、跳转、错误提示全部就绪。

构建方式二选一：运行默认 **CMake Build (Debug)** 任务，或使用 CMake Tools 的 Build 命令。快捷键按个人习惯自行绑定。

---

## 工程目录结构约定

```
MyProject/
├── CMakeLists.txt            # 顶层构建脚本，用户源文件在此添加
├── CMakePresets.json         # Debug / Release preset
├── cmake/
│   ├── gcc-arm-none-eabi.cmake   # 跨平台工具链文件
│   └── stm32cubemx/CMakeLists.txt  # CubeMX 生成，不手动改
├── Core/
│   ├── Inc/                  # HAL 配置头文件
│   └── Src/                  # main.c、HAL 初始化
├── Drivers/                  # CMSIS + STM32 HAL 库
├── User/                     # 用户代码（自行创建）
│   ├── Inc/
│   └── Src/
├── startup_*.s               # 启动汇编（芯片相关，CubeMX 生成）
├── *_FLASH.ld                # 链接脚本（芯片相关，CubeMX 生成）
└── compile_commands.json     # 软链接 → build/Debug/compile_commands.json
```

添加用户源文件时，在 `CMakeLists.txt` 的 `target_sources` 和 `target_include_directories` 中添加对应路径即可。

CubeMX 重新生成代码时只会修改它管理的文件（`Core/`、`Drivers/`、`cmake/stm32cubemx/`），你在 `CMakeLists.txt` 和 `User/` 中的改动不受影响。

---

## 常用操作

| 操作 | 方式 |
|------|------|
| 编译 | 默认 build task 或 CMake Tools Build（快捷键自行绑定） |
| 重新 Configure | `Cmd+Shift+P` → `Tasks: Run Task` → `CMake Configure (Debug)` |
| 清理 | `Cmd+Shift+P` → `Tasks: Run Task` → `CMake Clean` |
| 切换 Debug/Release | CMake Tools 侧边栏选择 preset |

---

## 跨平台说明

| 组件 | Win | Mac | 冲突？ |
|------|-----|-----|--------|
| `cmake/gcc-arm-none-eabi.cmake` | 完全一样（`arm-none-eabi-` 靠 PATH 找到） | 同 | 无 |
| 所有 CubeMX 生成文件 | 一样 | 一样 | 无 |
| 工具链 PATH | 系统环境变量 / Profile `cmake.environment` | Profile `cmake.environment` | 各自本地配 |
| VS Code Profile 配置 | `win/`（待补充） | `mac/` | 各自本地配 |
| clangd 全局配置 | 可选 | `~/.config/clangd/config.yaml` | 各自本地配 |

所有 CubeMX 生成文件 + 用户代码均可正常提交 git，Win/Mac 完全共用。

---

## 常见问题

**Q: 编译报 `stdint.h: No such file or directory`**

编译器缺少 newlib（C 标准库头文件）。检查是否用的是 Homebrew 的 `--without-headers` 版本，换用 xpack 或 ARM 官方工具链。

**Q: 代码里一堆红色波浪线但编译没问题**

clangd 的误报，创建 `~/.config/clangd/config.yaml`（见配置步骤第四步）可消除大部分。

**Q: clangd 没有代码补全/跳转**

确认工程根目录有 `compile_commands.json` 软链接。如果没有，手动运行一次 Configure task 或执行 `ln -sf build/Debug/compile_commands.json compile_commands.json`。

**Q: CubeMX 重新生成后要不要重新 Configure**

如果只改了引脚/外设配置（`Core/` 内变化），编译一次即可，CMake 会自动检测变化。如果改了 CMake 结构（增删源文件），需要重新 Configure。

**Q: 新建 `.c` 文件后编译找不到**

确认文件已加到 `CMakeLists.txt` 的 `target_sources` 中，且头文件路径在 `target_include_directories` 中。
