# STM32 VS Code Development Environment Setup Guide

This document guides users through setting up an independent STM32 development environment on Windows / macOS based on **VS Code + STM32CubeMX + CMake + Ninja + arm-none-eabi-gcc**.

This workflow does not depend on STM32CubeIDE. All core tools are installed and managed independently.

The template files referenced in this guide (`settings.json`, `tasks.json`, `keybindings.json`, etc.) are located in the `win/` or `mac/` directory of this repository.

---

# 1. Development Environment Components

The development environment consists of the following components:

| Component             | Purpose                                                |
| --------------------- | ------------------------------------------------------ |
| VS Code               | Development environment and workflow entry point       |
| VS Code Profile       | Manages platform-specific configuration                |
| STM32CubeMX           | Graphical hardware configuration and project code gen  |
| STM32 HAL / CMSIS     | Device headers, startup files and peripheral drivers   |
| ARM GNU Toolchain     | Firmware compilation, linking and debug symbol gen     |
| CMake                 | Project configuration and build generation             |
| Ninja                 | Build backend                                          |
| OpenOCD               | Flash programming and debug server                     |
| Cortex-Debug          | VS Code debug integration                              |

---

# 2. Preparation

> Before installing: if a tool listed below is already installed and working on your system, skip its section and move to the next one.

## 2.1 Install VS Code

Install the latest version of Visual Studio Code:

[https://code.visualstudio.com/](https://code.visualstudio.com/)

After installation, make sure VS Code starts correctly.

---

# 3. Install ST Tools

## 3.1 Install STM32CubeMX (Standalone)

Go to the ST website (st.com) → find "STM32CubeMX" → download the standalone installer.

**Do not use the version bundled with STM32CubeIDE.**

CubeMX is used for pin assignment, clock tree configuration, peripheral initialization, and generating HAL driver code and CMake project files.

---

# 4. Install Platform-Specific Tools

### Windows

#### 4.1 Install ARM GNU Toolchain

Install ARM GNU Toolchain, which provides `arm-none-eabi-gcc` (compiler) and `arm-none-eabi-gdb` (debugger).

```powershell
winget install Arm.GnuArmEmbeddedToolchain
```

Verify:

```bash
arm-none-eabi-gcc --version
arm-none-eabi-gdb --version
```

---

#### 4.2 Install CMake

CMake is used for project configuration and build generation. Ninja is bundled with CMake, no separate installation required.

```powershell
winget install Kitware.CMake
```

Verify:

```bash
cmake --version
```

---

#### 4.3 Install OpenOCD

Go to the OpenOCD website (openocd.org) → download the latest Windows release → extract to:

```
D:/OpenOCD/
```

Verify:

```bash
openocd --version
```

---

### macOS

#### 4.4 Install ARM GNU Toolchain

The Homebrew `arm-none-eabi-gcc` lacks newlib and is not suitable. Use the xpack distribution instead:

```bash
npm install -g xpm
xpm install @xpack-dev-tools/arm-none-eabi-gcc@latest --global
```

Note the xpack installation path.

Verify:

```bash
arm-none-eabi-gcc --version
```

---

#### 4.5 Install CMake + Ninja

```bash
brew install cmake ninja
```

---

#### 4.6 GDB and OpenOCD

Current template status:

* macOS supports development and build verification
* Flash programming and debugging are not supported yet

Therefore, OpenOCD is not required on macOS.

---

# 5. Create VS Code Profile

Open VS Code:

```
Gear icon (bottom-left)
    ↓
Profiles
    ↓
Create Profile
```

Create a new Profile.

Recommended name:

```
ST
```

Select:

```
Create Empty Profile
```

All following configurations will be stored in this Profile.

---

# 6. Install VS Code Extensions

Open the ST Profile and install the following extensions:

| Extension                                      | Purpose                                     |
| ---------------------------------------------- | ------------------------------------------- |
| `ms-vscode.cpptools`                           | C/C++ IntelliSense + cppdbg debug backend   |
| `ms-vscode.cmake-tools`                        | CMake integration: Configure / Build        |
| `marus25.cortex-debug`                         | ARM Cortex-M debugging                      |
| `xaver.clang-format`                           | C/C++ code formatting                       |
| `mcu-debug.memory-view`                        | Memory viewer                               |
| `mcu-debug.peripheral-viewer`                  | Peripheral register viewer                  |
| `mcu-debug.debug-tracker-vscode`               | Debug trace tracking                        |
| `mcu-debug.rtos-views`                         | RTOS task/queue view                        |

> Do not install extensions starting with `stmicroelectronics.*` (Cube IDE suite) or `llvm-vs-code-extensions.vscode-clangd` (deprecated; cpptools is used for IntelliSense instead).

---

# 7. Configure VS Code Profile

Copy the following files from the template:

```
settings.json
tasks.json
keybindings.json
```

`Ctrl + Shift + P` → **Open User Settings (JSON)**, paste `settings.json` and replace the placeholders.

Use the same method for **Open User Tasks** and **Open Keyboard Shortcuts (JSON)** to configure `tasks.json` and `keybindings.json`.

> The templates provide a pre-configured setup — just replace the placeholder paths to get started. However, the configuration surface is large and the templates may not cover every scenario. Users should review and supplement the configuration as needed.

---

## 7.1 Replace Paths

Replace the following variables:

```
<path-to-arm-none-eabi-gcc>
```

ARM GNU Toolchain directory (without `/bin`).

---

Windows:

```
<path-to-openocd>
```

OpenOCD directory (without `/bin`).

---

Windows:

```
<path-to-home>/.clang-format
```

Path to the `.clang-format` file in the user directory. The formatting rules file must be configured by the user; it is not provided by this template.

macOS uses a fixed `clang-format.executable` path (`/opt/homebrew/bin/clang-format`) and does not require this placeholder.

---

# 8. Create a Project

### 1. Generate Code with CubeMX

- Open **STM32CubeMX** (standalone)
- Select the target MCU, configure pins, clocks and peripherals
- **Project Manager → Project → Toolchain / IDE → CMake**
- Click **GENERATE CODE**

CubeMX overwrites the `cmake/stm32cubemx/` directory. The root `CMakeLists.txt` is generated once and maintained manually afterwards.

### 2. Open Project and Select Preset

Open the project folder in VS Code.

In the CMake sidebar, select the build preset:

- **Debug** — for debugging, includes debug symbols, slower compilation
- **Release** — for release, enables optimizations, **cannot debug**

> Debugging requires the Debug preset. Release builds cannot set breakpoints, and pausing will not navigate to the correct source line.

On first open, run `Ctrl+Shift+P` → `CMake: Configure` for the initial configuration.

### 3. Project Structure

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

# 9. Configure CMakeLists.txt

The root `CMakeLists.txt` is the project configuration entry point, managing source files, include directories and build options.

When adding new `.c` files, register them in `CMakeLists.txt`:

```cmake
target_sources(${CMAKE_PROJECT_NAME} PRIVATE
    User/Src/your_module.c
)
```

When adding new include directories:

```cmake
target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE
    User/Inc
)
```

---

# 10. CMake Kit

cmake-tools usually auto-detects the ARM GCC Kit. If the status bar already shows the correct Kit (`GCC 14.2.1 arm-none-eabi`), no manual action is needed.

If the Kit is incorrect or Configure fails with a compiler-not-found error:

1. `Ctrl+Shift+P` → `CMake: Scan for Kits`
2. In the status bar, select Kit → `GCC 14.2.1 arm-none-eabi`
3. `Ctrl+Shift+P` → `CMake: Delete Cache and Reconfigure`

---

# 11. Cross-Platform Git Workflow

This template enables cross-platform development through Profile isolation and Git sharing:

- **Project source files** (`Core/`, `Drivers/`, `User/`, `CMakeLists.txt`, `CMakePresets.json`, `cmake/`, `*.ioc`) are shared via Git. These files contain no local paths and are platform-independent.
- **Profile configuration** (`settings.json`, `tasks.json`, `keybindings.json`) contains absolute toolchain paths and is not committed. Windows and macOS each maintain their own Profile setup.
- **build directory** is added to `.gitignore`.

A complete `.gitignore` reference:

```
build/
mx.scratch
compile_commands.json
.cache
.vscode
.settings
.DS_Store
```

After cloning a project on a new machine, copy the appropriate platform's Profile configuration, select the Kit (Win) or verify toolchain paths (Mac), and build immediately.

---

# 12. Verify the Environment

Before building, verify that all required tools are available:

```bash
arm-none-eabi-gcc --version     # Win / Mac
cmake --version                 # Win / Mac
ninja --version                 # Mac
openocd --version               # Win
```

If all commands output version information, the environment is ready.

---

# 13. First Build

Make sure the CMake sidebar shows the Debug preset is selected (required for debugging) and that Configure has been run.

Incremental build:

```
F7
```

Full rebuild:

```
F6
```

Clean:

```
F5
```

---

# 14. Flash and Debug (Windows)

Select the VS Code debug configuration:

Flash only:

```
F1-download / F4-download
```

Debug:

```
F1-debug / F4-debug
```

Start:

```
F8
```

Stop debugging:

```
Shift + F8
```

Do not use the red stop button in the VS Code debug toolbar.

This may leave OpenOCD processes running in the background.

---

### 14.1 Adding a New Chip Series

The template provides launch configurations for F1 and F4 as examples. To add another series (e.g., H7, G0, L4), create a new pair of configs and a corresponding Flash task.

**Launch configuration** — in `settings.json`, copy an existing debug config (e.g., `F1-debug`) and its matching download config (`F1-download`), then update:

| Field | How to change |
|-------|---------------|
| `name` | Debug: `{series}-debug`, Download: `{series}-download` |
| `device` | Target MCU identifier, e.g., `STM32H743ZI` |
| `configFiles` target | `stm32h7x.cfg` — browse `D:/OpenOCD/scripts/target/` for the correct file |
| `svdFile` | `${workspaceFolder}/STM32H7xx.svd` |

`interface/stlink.cfg` in `configFiles` stays the same as long as you use ST-Link.

**Flash task** — in `tasks.json`, copy `Flash (F1)`, update the target config file and the label. Make sure the download config's `preLaunchTask` matches the new task label.

---

# 15. Switching Chips

When switching to a different MCU series:

- **Mac**: manually update `includePath` and `defines` in Profile `settings.json`.
- **Win**: IntelliSense auto-syncs from CMake via `configurationProvider`, but the launch config and Flash task (target cfg, device, svdFile) still need manual changes.

See `CHIP_SWITCH.md` for a field-by-field reference covering both platforms.

---

# 16. IntelliSense Configuration

This template uses:

```
Microsoft C/C++ IntelliSense
```

instead of clangd.

Reason:

clangd is based on the Clang frontend and is not the same compiler frontend as arm-none-eabi-gcc. Parsing GCC-specific features across different compiler environments (such as `__attribute__`, compiler built-in macros, etc.) may generate many false-positive diagnostics. Microsoft C/C++ IntelliSense queries the actual compiler through `compilerPath`, automatically obtaining the real built-in macros and system include paths for better GCC compatibility.

**Win**: uses `configurationProvider: ms-vscode.cmake-tools` — IntelliSense auto-syncs from CMake after Configure, no manual includePath/defines needed.

**Mac**: uses explicit `C_Cpp.default.includePath` + `defines`. When switching target MCU series, update these manually. See CHIP_SWITCH.md for details.

---

# 17. Hardware Notes

## 17.1 SWD Debug Pins

STM32 uses the following default SWD pins:

| Pin  | Function |
| ---- | -------- |
| PA13 | SWDIO    |
| PA14 | SWCLK    |

Do not assign these pins to other functions in CubeMX.

Otherwise, the debugger may no longer be able to connect to the device.

---

# 18. Common Issues

## Q: Command-line tools not found (`arm-none-eabi-gcc`, `cmake`, `openocd` not recognized)

After installation, the tool's `bin` directory must be added to the system PATH. Verify with the corresponding command in a terminal. If not recognized, reinstall or manually add the path to system environment variables.

## Q: Win CMake cannot find arm-none-eabi-gcc

`Ctrl+Shift+P` → `CMake: Scan for Kits` → select `GCC 14.2.1 arm-none-eabi` in the status bar. If the kit is not listed, check that ARM GCC is installed and that `cmake.environment` PATH points to the correct directory.

## Q: Code has red underlines but builds successfully

cpptools has not received the correct includePath/defines. Win: verify `C_Cpp.default.configurationProvider` is set to `ms-vscode.cmake-tools`, then run `CMake: Delete Cache and Reconfigure`. Mac: check that `C_Cpp.default.includePath` and `defines` match the target MCU.

## Q: CMake errors / cannot build after opening the project

Make sure a build preset (Debug or Release) is selected in the CMake sidebar. On first open, run `Ctrl+Shift+P` → `CMake: Configure`. If the preset list is empty, check that `CMakePresets.json` exists.

## Q: Newly added `.c` files cause linker errors

Make sure the source files are added to `CMakeLists.txt` through `target_sources()`. Simply placing files in the `User/Src/` directory is not enough — CMakeLists.txt is the project source management entry point.

## Q: OpenOCD remains after debugging exits

Always exit debugging with `Shift + F8`. Do not click the red stop button in the VS Code debug toolbar. If OpenOCD remains running on Windows, open Task Manager and terminate `openocd.exe`.

## Q: Breakpoints have no effect / unable to navigate after pause (Release preset cannot debug)

Make sure you are using the Debug build configuration (Debug preset), not Release. Release builds enable optimizations that break the mapping between debug symbols and source lines.

## Q: Flash fails with target connection error

Check the ST-Link USB connection and OpenOCD configuration. Verify that `cortex-debug.openocdPath` points to `D:/OpenOCD/bin/openocd.exe`.

## Q: Build fails with `stdint.h: No such file or directory`

The compiler is missing newlib. On Mac, do not use Homebrew's `arm-none-eabi-gcc` — use the xpack distribution instead.

## Q: How do I add support for a chip series not covered by the template (H7, G0, L4, etc.)?

See §14.1 for detailed steps. In short: copy an existing launch config pair in `settings.json` and a Flash task in `tasks.json`, then update `device`, `configFiles` target, `svdFile` and task labels to match the new chip series. `interface/stlink.cfg` stays the same if you use ST-Link.

---

# 19. Completion

After completing the above steps, the STM32 independent development workflow is ready:

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

This environment provides a fully transparent workflow for:

* Project configuration
* Build process
* Flash programming
* Debugging

while keeping the entire workflow independent from STM32CubeIDE.
