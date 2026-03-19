# Contributing Guide
1. [Software Contributing Guide](#software-contributions)
2. [Hardware Contributing Guide](#hardware-contributions)


## Software Contributions
### Example Used for this guide is project (EcoDrive)
Embedded motor-control firmware and simulation stack for PMSM/inverter development, with host simulation and STM32F4 hardware targets.


### Quick Start (Host Simulation)

1. Install dependencies: `cmake`, a C/C++ toolchain, `glfw3`, and OpenGL dev headers.
2. Configure: `cmake -S . -B build -DPLATFORM_NAME=host -DCMAKE_BUILD_TYPE=Release`
3. Build: `cmake --build build -j`
4. Run: `./build/bin/EcoDrive` or a test binary like `./build/bin/test_pmsm`



### Quick Start (STM32F4 Hardware)

1. Install `arm-none-eabi-gcc` and `cmake`.
2. Configure: `cmake -S . -B build -DPLATFORM_NAME=stm32f4 -DCMAKE_BUILD_TYPE=Release`
3. Build: `cmake --build build -j`
4. Artifacts are emitted to `build/bin/` and `.hex/.bin` are generated post-build.
OR easier Just run setup.sh file if provided

### Project Layout

- `application/`: Product-level application logic and system orchestration.
- `core/`: Core math, control, and shared utilities.
- `middleware/`: Shared middleware (e.g., streaming/serialization).
- `platform/host/`: Host simulation platform, FreeRTOS POSIX, ImGui UI, and drivers.
- `platform/stm32f4/`: STM32F4 HAL/LL platform, USB, startup, and FreeRTOS.
- `test/`: Host-only test and model executables.
- `tools/`: Python utilities, comm tooling, and scripts.
- `build/`: Out-of-tree build output (generated).

### Coding Standard (C/C++)

This repository follows a pragmatic embedded C/C++ standard intended to be portable across similar projects.

Language and compiler settings:

- C is `C11`, C++ is `C++20` (see `CMakeLists.txt`).
- Warnings should be treated as errors for new code when feasible.
- Prefer `-O2` or `-Og` for debug builds on embedded targets.

Style rules:

- Naming: `snake_case` for C functions/variables, `PascalCase` for types, `UPPER_SNAKE_CASE` for macros.
- Module prefixes: use a consistent, short prefix per subsystem for identifiers and macros.
- Prefix examples:
- `ELDRIVER_` for driver-level macros/constants and compile-time flags.
- `eldriver_` for driver-level C functions.
- `ELCORE_` for core macros/constants and compile-time flags.
- `elcore_` for core C functions , reusable datastructures.
- `ELMATH_` for math macros/constants and compile-time flags.
- `elmath_` for math C functions.
- Files: `module_name.c/.h` for C, `ModuleName.cpp/.hpp` for C++ where practical.
- Headers: include guards or `#pragma once` consistently within a file.
- Includes: order as local header, project headers, then system headers.
- Globals: avoid unless they are `static` or clearly documented hardware singletons.
- Interfaces: keep C headers `extern "C"` friendly for mixed C/C++ builds.

Embedded safety rules:

- No dynamic allocation on embedded targets unless explicitly justified.
- Keep ISR work minimal; move heavy work to tasks or deferred handlers.
- Validate bounds on buffers and peripheral data paths.
- Prefer fixed-point where timing determinism is required.

Formatting:

- Use 4 spaces, no tabs.
- Max line length 100 where reasonable.
- If you introduce a formatter, document it in `tools/` and add a `format` target.

### Porting Guide (New Hardware Target)

To add a new hardware platform, mirror the `platform/stm32f4/` approach:

1. Create `platform/<new_platform>/` with a `CMakeLists.txt` that exports `PLATFORM_LIB`, `PLATFORM_INTERFACE`, `PLATFORM_INCLUDES`, `PLATFORM_DEFINITIONS`, and `PLATFORM_SOURCE`.
2. Add a toolchain file at `platform/<new_platform>/cmake/` and reference it in the root `CMakeLists.txt`.
3. Provide startup, system, and IRQ files as needed for your MCU. ( you can use vendor tools like CubeMX for that)
4. Implement platform glue in `platform/<new_platform>/platform.c` and any board drivers.
5. Add any RTOS or USB stacks under the new platform directory.
6. Validate by building and running the host tests for shared core logic.

Minimal platform contract:

- `platform.c` should implement board init, clock init, and any HAL hooks.
- IRQ handlers must be strong symbols when overriding weak startup defaults.
- `PLATFORM_SOURCE` should include driver core sources so overrides link correctly.

### Documentation Guide

Create or update documentation under a `docs/` folder for reusable patterns. Suggested structure:

- `docs/overview.md`: system overview and architecture.
- `docs/build.md`: build instructions for host and hardware.
- `docs/porting.md`: detailed porting checklist and hardware bring-up.
- `docs/testing.md`: test strategy and how to run tests.
- `docs/hardware.md`: board notes, pin maps, power, and safety constraints.

Doxygen guidance:

- Keep public headers documented with `@brief` and parameter descriptions.
- Document control loops, scaling, and unit conventions.
- Add a short module comment block at the top of each subsystem header.

### Onboarding Checklist

1. Build the host target and run `test_pmsm`.
2. Read `application/` entry points and the main control loop.
3. Review `core/` math utilities and motor models.
4. Review `platform/host/` and `platform/stm32f4/` for platform boundaries.
5. Read `docs/overview.md` (or create it for a new project).

### File/Folder Heirarchy Map
- Core Code , Custom Data structures , Core
- Control algorithms: `application/`.
- Hardware drivers: `platform/<target>/` and `platform/<target>/eldriver/`.
- Simulation/GUI: `platform/host/` and `tools/`.
- Middleware/serialization: `middleware/`.
- Test and model validation: `test/` and datasets in the repository root.

### Build Targets

- `EcoDrive`: main application binary.
- `test_scopeStream`: host-only scope stream test.
- `test_inverter`: host-only inverter test.
- `test_pmsm`: host-only PMSM test.

### Contributions, Issues, and GitHub Projects

Contributions:

- Fork the repo and create a branch: `feature/<short-name>` or `fix/<short-name>`.
- Keep changes focused and aligned with the coding standard in this README.
- Add or update tests where behavior changes.
- Open a Pull Request with a concise summary and a checklist of what changed.

Issues:

- File issues for bugs, regressions, unclear behavior, and feature additions.
- Include target platform (`host` or `stm32f4`), steps to reproduce, expected vs actual behavior, and logs if available.
- Use labels to clarify type and priority (bug, enhancement, documentation, hardware).

GitHub Projects:

- Use a Project board to track Epics, Features, and Tasks.
- Each task should link to an Issue and a PR.
- Assign an owner and a due date when the work starts.
- Close Issues via PR descriptions using `Closes #<issue-number>`.


---

## 🧩 Hardware Contributions
(KiCad + GitHub)
- Keep all KiCad sources in a single top-level folder per board or subsystem.
- Commit schematic and PCB files together for any change.
- Prefer project-local libraries to avoid missing symbols/footprints.
- Do not commit KiCad autosave, backup, or lock files.
- Export manufacturing outputs (Gerbers, drill, BOM, pick-and-place) into a dedicated folder and keep those outputs versioned only for releases.
- Include a short `README` in each hardware folder with board purpose, revision, and KiCad version.
- For PRs, include screenshots of schematic and PCB diffs when possible.


 GitHub Projects:

- Use a Project board to track Epics, Features, and Tasks.
- Each task should link to an Issue and a PR.
- Assign an owner and a due date when the work starts.
- Close Issues via PR descriptions using `Closes #<issue-number>`.

## License

MIT. See license headers in platform dependencies where applicable.


