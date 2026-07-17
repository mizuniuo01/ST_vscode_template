# STM32 VS Code 开发环境配置指南

本文档用于指导 Windows / macOS 平台搭建基于 **VS Code + STM32CubeMX + CMake + Ninja + arm-none-eabi-gcc** 的 STM32 独立开发环境。

本配置流程不依赖 STM32CubeIDE，所有核心工具均独立安装和管理。

本指南中引用的模板文件（`settings.json`、`tasks.json`、`keybindings.json` 等）位于本仓库的 `win/` 或 `mac/` 目录下。

---

# 1. 环境组成

开发环境由以下组件组成：

| 组件                    | 作用             |
| --------------------- | -------------- |
| VS Code               | 开发环境与工作流入口     |
| VS Code Profile       | 管理平台相关配置       |
| STM32CubeMX           | 图形化硬件配置与工程代码生成 |
| STM32 HAL / CMSIS     | 设备头文件、启动文件和驱动库  |
| ARM GNU Toolchain     | 固件编译、链接与调试符号生成 |
| CMake                 | 工程配置与构建生成      |
| Ninja                 | 构建后端           |
| OpenOCD               | 烧录与调试服务器       |
| Cortex-Debug          | VS Code 调试集成   |

---

# 2. 安装前准备

> 注意：如果下列工具已经安装且可用，跳过对应章节，继续下一项即可。

## 2.1 安装 VS Code

安装最新版 Visual Studio Code：

[https://code.visualstudio.com/](https://code.visualstudio.com/)

安装完成后确认 VS Code 可以正常启动。

---

# 3. 安装 ST 工具

## 3.1 安装 STM32CubeMX（独立版）

前往 ST 官网（st.com）→ 找到 "STM32CubeMX" → 下载独立版安装程序。

**不要使用 STM32CubeIDE 内置版本。**

CubeMX 用于引脚分配、时钟树配置、外设初始化，并生成 HAL 驱动代码和 CMake 工程文件。

---

# 4. 安装平台相关工具

### Windows

#### 4.1 安装 ARM GNU Toolchain

安装 ARM GNU Toolchain，提供 `arm-none-eabi-gcc`（编译器）和 `arm-none-eabi-gdb`（调试器）。

```powershell
winget install Arm.GnuArmEmbeddedToolchain
```

验证：

```bash
arm-none-eabi-gcc --version
arm-none-eabi-gdb --version
```

---

#### 4.2 安装 CMake + Ninja

CMake 负责工程配置，Ninja 是构建后端。Windows 下应显式安装两者，不要假定 CMake 安装包一定附带可用的 `ninja.exe`。

```powershell
winget install --id Kitware.CMake -e
winget install --id Ninja-build.Ninja -e
```

验证：

```bash
cmake --version
ninja --version
```

---

#### 4.3 安装 OpenOCD

前往 OpenOCD 官网（openocd.org）→ 下载最新 Windows 版本 → 解压到：

```
D:/OpenOCD/
```

Windows 模板按以下解压目录结构配置：

```text
D:/OpenOCD/bin/openocd.exe
D:/OpenOCD/openocd/scripts/interface/stlink.cfg
D:/OpenOCD/openocd/scripts/target/
```

验证：

```bash
openocd --version
```

---

### macOS

#### 4.4 安装 ARM GNU Toolchain

macOS Homebrew 的 `arm-none-eabi-gcc` 缺少 newlib，不可用。使用 xpack 分发版：

```bash
npm install -g xpm
xpm install @xpack-dev-tools/arm-none-eabi-gcc@latest --global
```

记下 xpack 安装路径。

验证：

```bash
arm-none-eabi-gcc --version
```

---

#### 4.5 安装 CMake + Ninja

```bash
brew install cmake ninja
```

---

#### 4.6 GDB 与 OpenOCD

当前模板：

* macOS 支持开发和编译验证
* 暂不支持烧录和调试

因此无需安装 OpenOCD。

---

# 5. 创建 VS Code Profile

打开 VS Code：

```
左下角齿轮
    ↓
Profiles
    ↓
Create Profile
```

创建新的 Profile。

建议名称：

```
ST
```

选择：

```
Create Empty Profile
```

后续所有配置均放入该 Profile。

---

# 6. 安装 VS Code 扩展

进入 ST Profile，安装以下扩展：

| 扩展 | 用途 |
|------|------|
| `ms-vscode.cpptools` | C/C++ IntelliSense + Windows Flash launch 包装器 |
| `ms-vscode.cmake-tools` | CMake 集成：Configure / Build |
| `marus25.cortex-debug` | ARM Cortex-M 调试 |
| `xaver.clang-format` | C/C++ 代码格式化 |
| `mcu-debug.memory-view` | 内存查看 |
| `mcu-debug.peripheral-viewer` | 外设寄存器查看 |
| `mcu-debug.debug-tracker-vscode` | 调试轨迹跟踪 |
| `mcu-debug.rtos-views` | RTOS 任务/队列视图 |

> 不要安装 `stmicroelectronics.*` 开头的扩展（Cube IDE 全家桶）和 `llvm-vs-code-extensions.vscode-clangd`（已弃用，统一用 cpptools IntelliSense）。

---

# 7. 配置 VS Code Profile

将模板中的配置文件复制到 Profile：

```
settings.json
tasks.json
keybindings.json
```

`Ctrl + Shift + P` → **Open User Settings (JSON)**，粘贴 `settings.json`，替换占位符。

`Ctrl + Shift + P` → **Open User Tasks**，粘贴 `tasks.json`。

`Ctrl + Shift + P` → **Open Keyboard Shortcuts (JSON)**，粘贴 `keybindings.json`。

> 模板提供了预配置的开箱即用方案，只需替换占位符路径即可使用。但配置项较多，模板可能未覆盖所有场景——使用者应自行检查并根据需要补充配置。

---

## 7.1 修改路径

替换以下变量：

```
<path-to-arm-none-eabi-gcc>
```

ARM GNU Toolchain 目录（不含 `/bin`）。

---

Windows：

```
<path-to-openocd>
```

OpenOCD 安装根目录（不含 `/bin`）。按上述结构填写 `D:/OpenOCD`。如果使用的发行包把 `scripts/` 放在其他位置，需要同时修改 `settings.json` 中 cortex-debug 的 `configFiles` 和 `tasks.json` 中的 `-s` 参数。

---

Windows：

```
<path-to-home>/.clang-format
```

用户目录下的 `.clang-format` 文件路径。格式化规则文件需用户自行配置，本模板不提供。

macOS 使用固定路径 `clang-format.executable`（`/opt/homebrew/bin/clang-format`），无需此占位符。

---

# 8. 创建工程

### 1. CubeMX 生成代码

- 打开 **STM32CubeMX**（独立版）
- 选择芯片，配好引脚、时钟、外设
- **Project Manager → Project → Toolchain / IDE → CMake**
- 点击 **GENERATE CODE**

CubeMX 会覆盖 `cmake/stm32cubemx/` 目录，根 `CMakeLists.txt` 只生成一次，之后手动维护。

### 2. 打开工程并选择预设

在 VS Code 中打开工程文件夹。

在 CMake 侧边栏中选择构建预设：

- **Debug** — 调试用，含调试符号，编译较慢
- **Release** — 发布用，开启优化，**不能调试**

> 调试必须使用 Debug 预设。Release 编译的固件无法打断点，暂停后无法跳转到目标行。

确保 CMake 侧边栏已选择 Debug 预设。Profile 有意关闭打开工程时自动 Configure；`Build` 和 `Rebuild` task 会在编译前执行 Configure，因此首次配置也可直接使用 F7/F6 完成。

### 3. 工程结构

```
MyProject/
├── CMakeLists.txt
├── CMakePresets.json
├── cmake/
│   ├── gcc-arm-none-eabi.cmake
│   └── stm32cubemx/CMakeLists.txt
├── Core/
│   ├── Inc/
│   └── Src/
├── Drivers/
├── User/
│   ├── Inc/
│   └── Src/
├── startup_*.s
├── *_FLASH.ld
└── build/
```

---

# 9. 配置 CMakeLists.txt

工程根目录下的 `CMakeLists.txt` 是工程配置入口，管理源文件、头文件路径和构建选项。

新增 `.c` 文件时，需在 `CMakeLists.txt` 中添加：

```cmake
target_sources(${CMAKE_PROJECT_NAME} PRIVATE
    User/Src/your_module.c
)
```

新增头文件路径时：

```cmake
target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE
    User/Inc
)
```

---

# 10. CMake Kit

cmake-tools 通常会自动检测到 ARM GCC Kit。如果状态栏显示的 Kit 正确（`GCC 14.2.1 arm-none-eabi`），无需手动操作。

若 Kit 不正确或 Configure 报找不到编译器：

1. `Ctrl+Shift+P` → `CMake: Scan for Kits`
2. 状态栏 Kit 选择器 → 选择 `GCC 14.2.1 arm-none-eabi`
3. `Ctrl+Shift+P` → `CMake: Delete Cache and Reconfigure`

---

# 11. 跨平台 Git 工作流

本模板通过 Profile 隔离 + Git 共享实现 Windows / macOS 跨平台开发：

- **工程源码**（`Core/`、`Drivers/`、`User/`、`CMakeLists.txt`、`CMakePresets.json`、`cmake/`、`*.ioc`）通过 Git 共享。这些文件不含本地路径，跨平台通用。
- **Profile 配置**（`settings.json`、`tasks.json`、`keybindings.json`）包含绝对路径，不提交 Git。Windows 和 macOS 分别在对应 Profile 下维护。
- **build 目录** 加入 `.gitignore`。

完整的 `.gitignore` 参考：

```
build/
mx.scratch
compile_commands.json
.cache
.vscode
.settings
.DS_Store
```

克隆工程到新机器后，拷贝对应平台的 Profile 配置，选好 Kit（Win）或确认工具链路径（Mac），即可直接编译。

---

# 12. 验证环境

编译前先确认工具链就绪：

```bash
arm-none-eabi-gcc --version     # Win / Mac
cmake --version                 # Win / Mac
ninja --version                 # Win / Mac
openocd --version               # Win
```

全部正常输出版本号即可继续。

---

# 13. 第一次编译

确认 CMake 侧边栏已选择 Debug 预设（调试需要）。F7/F6 会先 Configure 再编译，正常 task 工作流不需要单独执行 `CMake: Configure`。

编译（配置 + 编译）：

```
F7
```

重新编译（配置 + 清理 + 完整编译）：

```
F6
```

清理：

```
F5
```

---

# 14. 烧录与调试（Windows）

选择 VS Code 调试配置：

仅烧录：

```
F1-flash / F4-flash
```

调试：

```
F1-debug / F4-debug
```

启动：

```
F8
```

Flash launch 配置通过 `preLaunchTask` 调用 `Flash (F1)` / `Flash (F4)`。每个 task 会先构建，再用对应的 ST-Link interface 和 MCU target cfg 启动 OpenOCD，关闭不使用的 GDB 端口，最后完成烧录、校验、复位并退出。task 成功后，`cppvsdbg` 会用 `exit /b 0` 短暂启动并退出 `${env:ComSpec}`，使 Flash 项可以保留在“运行和调试”选择器中，同时避免把 ARM ELF 当作 Windows 程序启动。真正用于烧录的 ELF 路径仍由 Flash task 的 OpenOCD 命令指定。

debug 配置通过 `preLaunchTask` 调用 `Build`。随后 Cortex-Debug 启动 OpenOCD 和 GDB、下载当前 ELF，并进入调试会话。

停止调试：

```
Shift + F8
```

按当前实测工作流，不要使用 VS Code 调试工具栏中的红色停止按钮，该操作可能残留 OpenOCD 进程；应使用已配置的 `Shift + F8`，它会分别处理 launch 和 attach 会话。

---

### 14.1 添加新芯片系列

模板以 F1 和 F4 为例提供了 Flash + debug launch 配置对。要支持其他系列（如 H7、G0、L4），需创建新的配置对和对应的 Flash 任务。

**Launch 配置** — 在 `settings.json` 中复制现有调试配置（如 `F1-debug`）和对应的 Flash 包装配置（`F1-flash`），修改以下字段：

| 字段 | 修改方式 |
|------|---------|
| `name` | 调试：`{系列}-debug`，烧录：`{系列}-flash` |
| `device` | 仅 debug 配置：目标芯片型号，如 `STM32H743ZI` |
| `configFiles` target | 仅 debug 配置：`stm32h7x.cfg` — 在 `<path-to-openocd>/openocd/scripts/target/` 下查看正确文件名 |
| `svdFile` | 仅 debug 配置：`${workspaceFolder}/STM32H7xx.svd` |
| `preLaunchTask` | Flash 包装配置：对应任务标签，如 `Flash (H7)`；debug 配置：`Build` |

只要使用 ST-Link，`configFiles` 中的 `interface/stlink.cfg` 无需修改。

**Flash 任务** — 在 `tasks.json` 中复制 `Flash (F1)`，修改 target 配置文件名和任务标签。确保 Flash 包装配置的 `preLaunchTask` 指向新的任务标签。

---

# 15. 换芯片

更换目标芯片系列时：

- **Mac**：需手动修改 Profile `settings.json` 中的 `includePath` 和 `defines`。
- **Win**：IntelliSense 通过 `configurationProvider` 从 CMake 自动同步，但 launch 配置和 Flash 任务（target cfg、device、svdFile）仍需手动修改。

详见 `CHIP_SWITCH.md` 中双平台逐字段对照表。

---

# 16. IntelliSense 配置

本模板使用：

```
Microsoft C/C++ IntelliSense
```

不是 clangd。

原因：

clangd 是 clang 前端，不是 arm-none-eabi-gcc。跨编译器解析 GCC 特有语法（`__attribute__`、编译器内置宏等）会产生大量假阳性。Microsoft C/C++ IntelliSense 通过 `compilerPath` 直接 query 编译器，自动获取真实的内置宏和系统头文件路径，对 GCC 兼容性更好。

**Win**：将 `compilerPath` 指向实际的 `arm-none-eabi-gcc.exe`，并使用 `configurationProvider: ms-vscode.cmake-tools`。Configure 后，cpptools 从 ARM GCC 获取编译器内置配置，从 CMake 获取工程 includePath/defines，无需手动填写这些路径和宏。

**Mac**：使用显式 `C_Cpp.default.includePath` + `defines`。换芯片时需手动修改，详见 CHIP_SWITCH.md。

---

# 17. 硬件注意事项

## 17.1 SWD 调试引脚

STM32 默认使用：

| 引脚   | 功能    |
| ---- | ----- |
| PA13 | SWDIO |
| PA14 | SWCLK |

不要在 CubeMX 中将这两个引脚复用为其他功能。

否则可能导致无法再次连接调试器。

---

# 18. 常见问题

## Q: 命令行工具未找到（`arm-none-eabi-gcc`、`cmake`、`openocd` 等命令无法识别）

安装工具后需将其 `bin` 目录加入系统 PATH。在终端中运行对应命令验证。如果无法识别，重新安装或手动添加路径到系统环境变量。

## Q: Win CMake 报找不到 arm-none-eabi-gcc

`Ctrl+Shift+P` → `CMake: Scan for Kits` → 底部状态栏选 `GCC 14.2.1 arm-none-eabi`。若 Kit 列表中无此项，检查 ARM GCC 是否已安装且 `cmake.environment` 中 PATH 指向正确。

## Q: 代码红色波浪线但编译通过

cpptools 未正确获取 includePath/defines。Win：确认 `C_Cpp.default.configurationProvider` 为 `ms-vscode.cmake-tools`，然后 `CMake: Delete Cache and Reconfigure`。Mac：检查 `C_Cpp.default.includePath` 和 `defines` 是否与芯片匹配。

## Q: 打开工程后 CMake 报错 / 无法编译

确认已在 CMake 侧边栏选择构建预设（Debug 或 Release），再使用 F7/F6，让 task 在编译前执行 Configure。如果预设列表为空，检查 `CMakePresets.json` 是否存在。

## Q: 新增 `.c` 文件后编译报错 / 链接不到

确认源文件已添加到 `CMakeLists.txt` 的 `target_sources()` 中。仅将文件放入 `User/Src/` 目录是不够的——CMakeLists.txt 是工程源文件管理入口。

## Q: 调试退出后 OpenOCD 残留

退出调试时使用 `Shift + F8`。按当前实测工作流，红色停止按钮可能残留 OpenOCD，而快捷键会根据会话类型执行已配置的 stop/disconnect 动作。如果已经残留，Windows 下打开任务管理器，结束 `openocd.exe`。

## Q: 断点无反应 / 暂停后无法跳转到目标行（Release 预设无法调试）

确保使用的是 Debug 构建配置（Debug preset），而非 Release。Release 编译开启了优化，调试符号与源码行对应关系会错乱。

## Q: Flash 失败，提示无法连接目标

检查 ST-Link USB 连接和 OpenOCD 配置。确认 `cortex-debug.openocdPath` 指向 `D:/OpenOCD/bin/openocd.exe`。

## Q: 编译报 `stdint.h: No such file or directory`

编译器缺少 newlib。Mac 不要用 Homebrew 的 `arm-none-eabi-gcc`，使用 xpack 分发版。

## Q: 模板没有覆盖的芯片系列（H7、G0、L4 等）如何添加支持？

详见 §14.1。简言之：在 `settings.json` 中复制现有 launch 配置对，在 `tasks.json` 中复制 Flash 任务，将 `device`、`configFiles` target、`svdFile` 和任务标签改为新芯片系列对应的值。如果使用 ST-Link，`interface/stlink.cfg` 保持不变。

---

# 19. 完成

完成以上步骤后，即可使用：

```
VS Code
+
STM32CubeMX
+
ARM GNU Toolchain
+
CMake + Ninja
+
OpenOCD
+
GDB
```

完成 STM32 的独立开发流程。

该环境提供完整透明的：

* 工程配置
* 编译流程
* 烧录流程
* 调试流程

并保持与 STM32CubeIDE 工作流解耦。
