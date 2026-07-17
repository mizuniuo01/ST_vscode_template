# STM32 Project Template for VS Code

> A transparent and customizable VSCode-based development workflow for STM32 microcontrollers.

A minimal and reproducible project template based on VSCode, STM32 development tools and an independent embedded toolchain.

It provides reference configurations and project templates rather than a single fixed application project.

This template includes:

- STM32CubeMX
- STM32 HAL / CMSIS
- ARM GNU Toolchain (`arm-none-eabi-gcc`)
- CMake + Ninja
- OpenOCD + GDB

It is designed to work without STM32CubeIDE, providing a lightweight, layered and fully controllable development workflow.

The template supports both Windows and macOS through profile-level VSCode configuration that can be reused across multiple STM32 projects.

Windows provides the complete build, flash and debug workflow.

macOS is currently supported for project development and build verification only. Flash and debug support on macOS is not included yet.

This template is intended for embedded developers who prefer a clean and IDE-independent workflow.

---

## Highlights

- **Independent development toolchain**  
  All development tools are installed and managed separately from STM32CubeIDE.

- **IDE-independent development workflow**  
  Build, flash and debug are implemented through standard tools such as CMake, Ninja, GDB and OpenOCD, providing a lightweight and fully controllable development environment.

- **CubeMX-based project generation**  
  STM32CubeMX is used for hardware configuration and code generation, while the build system and development workflow remain independent from vendor IDE projects.

- **CMake project configuration**  
  CMakeLists.txt serves as the project configuration entry point, managing source files, include directories and build options. Detailed project customization is described in GUIDE.md.

- **CMake-based build system**  
  The project uses CMake as the build configuration layer and Ninja as the build backend, providing a modern, portable and maintainable build workflow.

- **VSCode profile-based configuration**  
  Toolchain paths, tasks and keybindings are managed through VSCode profiles, allowing the same configuration model to be reused across multiple STM32 projects.

- **Cross-platform development support (Windows / macOS)**  
  Both platforms share the same project structure and source code workflow. Windows provides the complete build, flash and debug workflow, while macOS currently focuses on project development and build verification.

- **Cross-platform project structure**  
  Platform-specific differences are isolated into separate VSCode profiles and configuration files, while core project files remain consistent. This allows projects to be synchronized and developed across platforms through Git.

- **Standard Cortex-M debug architecture**  
  The workflow is based on the standard VS Code + GDB + OpenOCD debugging pipeline, enabling SWD debugging without dependency on STM32CubeIDE.

- **Customizable workflow**  
  The provided configuration serves as both a ready-to-use setup and a reference design. Users can customize build options, compiler settings, debug tools, keybindings and other VSCode configurations according to their own workflow.

---

## Project Structure

> The template separates VSCode profile configuration from STM32 project files and documentation, keeping platform-specific settings isolated while maintaining reusable project configurations.

- `win/`  
  VSCode profile configuration for Windows, including settings, tasks and keybindings.

- `mac/`  
  VSCode profile configuration for macOS, aligned with the Windows setup with platform-specific differences.

- `CHIP_SWITCH.md`  
  Quick reference for switching between different STM32 devices.

- `GUIDE.md`  
  English setup and usage guide, covering detailed configuration and project onboarding.

- `GUIDE(CHS).md`  
  Chinese version of the setup and usage guide.

> The `win/` and `mac/` Profile configurations provide pre-configured templates with placeholder paths. Users only need to replace the paths to get started. However, the configuration surface is large and the templates may not cover every setting — users should review and supplement the configuration as needed.

---

## Toolchain

This workflow separates STM32 development tools from the IDE environment, keeping the build and debug infrastructure independently managed.

The development workflow is based on the following components:

- **STM32CubeMX**  
  Provides graphical hardware configuration and code generation for STM32 devices.

- **STM32 HAL / CMSIS**  
  Provides CMSIS device support, startup files and STM32 HAL peripheral drivers required for firmware development.

- **ARM GNU Toolchain (`arm-none-eabi-gcc`)**  
  Used for firmware compilation, linking and debug symbol generation.

- **CMake + Ninja**  
  Provides a modern build system workflow. CMake manages project configuration through CMakeLists.txt, while Ninja performs the actual build process.

- **GDB (ARM GNU Toolchain)**  
  Used as the debugging backend for breakpoint control, stepping and target inspection.

- **OpenOCD**  
  Provides flash programming and debug server support between GDB and STM32 debug probes.

This setup avoids dependency on STM32CubeIDE and enables a transparent, reproducible and fully controllable development workflow.

---

## Workflow Overview

The workflow separates the development environment into independent layers:

- VSCode provides the user interface and workflow orchestration layer.
- VSCode tasks and profiles manage project operations and platform-specific configurations.
- STM32CubeMX provides hardware configuration and generates STM32 project files.
- CMakeLists.txt defines project sources, include directories and build options.
- CMake manages project configuration and build generation.
- Ninja executes the build process.
- ARM GNU Toolchain provides compilation, linking and debug symbol generation.

The workflow keeps each component visible and independently managed, allowing users to customize or extend individual components.

For debugging, VSCode invokes Cortex-Debug, which connects GDB with OpenOCD and the STM32 target through SWD.

---

## VSCode Integration

This workflow uses VSCode as the primary development environment.

VSCode acts as the workflow orchestration and integration layer, while the underlying toolchain remains independently managed.

The workflow uses several VSCode extensions to integrate with external development tools:

- **C/C++ Extension**  
  Provides IntelliSense support and C/C++ language features.

- **CMake Tools**  
  Provides CMake project configuration and build integration.

- **Cortex-Debug**  
  Provides ARM Cortex-M debugging integration through GDB and OpenOCD.

- **clang-format**  
  Provides C/C++ code formatting support. The `.clang-format` rules file is not provided by this template — users should configure it according to their own coding style.

- **MCU Debug extensions (optional)**  
  Provide additional debugging features such as memory, peripheral and debug information visualization.

These extensions connect VSCode with the underlying development tools, while the toolchain itself remains independently installed, managed and replaceable.

### IntelliSense

The template uses Microsoft C/C++ IntelliSense instead of clangd.

The IntelliSense configuration is aligned with the ARM GNU Toolchain, allowing VSCode to correctly resolve embedded headers, compiler definitions and STM32 device-specific configurations.

This avoids inconsistencies caused by using a different compiler frontend, especially when working with STM32 HAL, CMSIS headers and compiler-specific definitions.

---

## Quick Start

For detailed installation and configuration instructions, please refer to:

- [GUIDE.md](GUIDE.md) (English)
- [GUIDE(CHS).md](GUIDE(CHS).md) (Chinese)

Basic workflow:

1. Install the required dependencies and configure the VSCode profile (see GUIDE.md).
2. Generate an STM32 project using STM32CubeMX (Toolchain / IDE → CMake).
3. Open the project in VS Code, select the Debug preset in the CMake sidebar, and run `CMake: Configure`.
4. Add application source files to `CMakeLists.txt` through `target_sources()`. Simply placing files in `User/Src/` is not enough — CMakeLists.txt is the project entry point.
5. Build with F7, flash and debug with F8.

### New Project Setup

1. Open STM32CubeMX, configure your target MCU, and generate the project with **Toolchain / IDE → CMake**.
2. Open the generated project folder in VS Code. In the CMake sidebar, select the **Debug** preset (required for debugging) and run `CMake: Configure`.
3. Add application code under `User/Src/` and `User/Inc/`. Register every new `.c` file in `CMakeLists.txt` through `target_sources()` — CMakeLists.txt is the project entry point; simply placing files in the directory is not enough.
4. Build with F7. On Windows, flash and debug with F8. Use `F1-download` / `F4-download` for flash only, `F1-debug` / `F4-debug` for debugging.

The provided template configuration can be reused across multiple STM32 projects while keeping platform-specific settings isolated.

---

## Usage

### Build

- **F7** — Incremental build  
- **F6** — Full rebuild (clean + build)  
- **F5** — Clean build artifacts  

### Flash & Debug

- **F8** — Flash or start debugging (Windows only)
- Select the `F1-download` / `F4-download` task for flash only (Windows only)
- Select the `F1-debug` / `F4-debug` task for debugging (Windows only)

- **Shift+F8** — Stop debugging session (Windows only)

### Workflow

All operations are executed through VSCode tasks and keybindings defined in the project profile.

These tasks internally invoke the build system (CMake + Ninja) and debugging tools (OpenOCD + GDB).

The development workflow keeps build, flash and debug operations explicit and configurable, allowing users to inspect or modify each step according to their requirements.

---

## Debug Architecture

The debug pipeline is built on a standard Cortex-M toolchain, orchestrated through VSCode:

```text
VSCode + Cortex-Debug
        ↓
        GDB
        ↓
        OpenOCD
        ↓
        ST-Link
        ↓
        SWD
        ↓
        STM32
```

- **VSCode + Cortex-Debug**  
  Acts as the orchestration and integration layer, providing a frontend for launching and managing debug sessions.

- **GDB**  
  Handles breakpoints, stepping, variable inspection and target state control.

- **OpenOCD**  
  Serves as the debug server, connecting GDB with the target device through the debug probe.

- **ST-Link**  
  Provides the physical connection between the host computer and the STM32 device.

- **SWD (Serial Wire Debug)**  
  Provides the standard debug interface used to communicate with the Cortex-M target.

The VSCode debug extensions provide integration between the user workflow and the underlying tools, while keeping the complete development pipeline visible and configurable.

---

## FAQ

### Toolchain & Build

**Q: Compiler not found (`arm-none-eabi-gcc` not found)**  
Check that the ARM GNU Toolchain is correctly installed and that the compiler path is configured properly in the VSCode profile.

**Q: CMake configuration fails or the build system cannot be generated**  
Ensure that CMake and Ninja are installed correctly and that the selected toolchain file points to a valid ARM GNU Toolchain installation.

**Q: Newly added `.c` files are not compiled**  
Make sure the source files are added to `CMakeLists.txt` through the corresponding `target_sources()` configuration. Simply placing files in the project directory is not enough — CMakeLists.txt is the project source management entry point.

---

### STM32CubeMX

**Q: Should STM32CubeMX be installed separately from STM32CubeIDE?**  
Yes. This workflow uses the standalone STM32CubeMX application and does not depend on STM32CubeIDE.

**Q: How should CubeMX generated files be managed?**  
CubeMX is responsible for hardware configuration and code generation. The generated HAL, CMSIS and initialization files are managed as part of the STM32 project, while custom application code and build configuration should be maintained separately.

**Q: CubeMX regeneration overwrites project files. What should I do?**  
Avoid modifying CubeMX-generated files outside designated user sections whenever possible. Keep user application code separated from generated code and maintain custom build configuration outside CubeMX-managed files.

---

### Project Structure

**Q: Why are STM32 source files stored inside the project directory instead of external SDK paths?**  
STM32CubeMX generates project-level HAL/CMSIS dependencies, allowing each project to remain self-contained. Unlike SDK-based workflows where source files are referenced from external SDK paths, STM32 projects normally keep generated dependencies inside the project directory. This makes the project structure easier to synchronize and reproduce across different environments, without relying on an external SDK path during the build process.

---

### IntelliSense

**Q: Code has red underlines but the project builds successfully**  
Check that the C/C++ extension has received the correct include paths and compiler definitions from the CMake configuration.

**Q: Why does this template use Microsoft C/C++ IntelliSense instead of clangd?**  
The configuration is aligned with the ARM GNU Toolchain used for building the firmware. This provides better consistency when resolving STM32 HAL, CMSIS headers and compiler-specific definitions.

---

### Debug & Flash

**Q: OpenOCD cannot connect to the target device**  
Check the debug probe connection, target configuration and OpenOCD settings.

**Q: Debug session does not exit cleanly**  
Always stop debugging using the configured VSCode task or shortcut. Improper termination may leave OpenOCD processes running in the background.

**Q: Flash programming fails**  
Verify that OpenOCD is correctly configured for the selected STM32 target and that the debug probe is accessible.

---

### General

**Q: Why not use STM32CubeIDE?**  
This project does not aim to replace STM32CubeIDE for every use case.

Instead, it provides an independent and transparent development workflow based on standard tools such as CMake, ARM GNU Toolchain, OpenOCD and VSCode.

By separating the editor, build system and debug infrastructure, users can inspect, customize and extend each component according to their own requirements.