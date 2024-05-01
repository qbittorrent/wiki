## Introduction

As of revision `63ff5e3` (this is after version 4.2.5), CMake should be considered the preferred way of building qBittorrent on all supported platforms.

Note that currently, the CMake build scripts may not handle installation to system directories both correctly and in a fully automated fashion for all platforms (this is noted in each platform-specific guide where relevant). Additionally, currently they don't handle packaging at all. Patches are welcome to improve this!

## Prerequisites

In order to build qBittorrent with CMake, first check that you have the following:

- Basic understanding of how to use CMake. Refer to [CMake Basics](#CMake-Basics) below if needed.
- A C++ compiler and toolchain (e.g. `build-essential` package on Ubuntu, Visual Studio/MSVC toolchain on Windows, Xcode toolchain on macOS, ...)
- CMake >= 3.16. If your distribution does not include a recent enough version in their repositories, install it from the official website or the official PPA.
- The Ninja build tool (optional, but recommended).
- Development files (headers and libraries, and potentially sources) of all dependencies (Qt, zlib, OpenSSL, Boost, libtorrent) for your platform.
More information about what is needed specifically is detailed in each [platform-specific guide](https://github.com/qbittorrent/qBittorrent/wiki#cmake).

## qBittorrent build configuration options

You may use the following qBittorrent-specific options to customize the build. Pass them in the **configure step** like so: `-D<option_name>=<value>`. This information applies to any of the [platform-specific guides](https://github.com/qbittorrent/qBittorrent/wiki#cmake).

| Option name | Type | Default value | Description | Conditions, if any |
|-|-|-|-|-|
| `DBUS` | Bool  | `ON` |  Enables support for notifications and power-management features on UNIX-like systems via D-Bus | Not Windows and not Apple |
| `GUI` | Bool | `ON` | Enable graphical user interface | Can't be disabled on Windows |
| `MSVC_RUNTIME_DYNAMIC` | Bool | `ON` | Use MSVC dynamic runtime library (-MD) instead of static (-MT) | Only relevant when building on Windows with MSVC |
| `QBT_VER_STATUS` | String | `alpha1` | Additional version string to append | Should be empty for release builds. Purposefully left undocumented for now, may change or be removed in the future without notice. |
| `STACKTRACE` | Bool | `ON` |Enable stacktraces | - |
| `SYSTEMD` | Bool | `OFF` | Install systemd service file to a directory manually overridable with `Systemd_SERVICES_INSTALL_DIR` | Not Windows and not Apple and not `GUI` |
| `Systemd_SERVICES_INSTALL_DIR` | Path | Default SystemD services directory | Where to install the SystemD service file to | Only relevant if `SYSTEMD` is `ON` |
| `VERBOSE_CONFIGURE` | Bool | `OFF` | Show more information in the configure output (only useful for debugging the CMake build scripts) | - |
| `WEBUI` | Bool | `ON` | Enables built-in HTTP server for headless use | - |

---
---
---

## CMake Basics

Building qBittorrent with CMake comes down to a 2-step process:

- the configuration and generation, also known as the **configure step/"configure-time"**, where 
- and the build itself, also known as the **build step/"build-time"**.

In the **configure step** where it is possible to set some options to customize the qBittorrent build, CMake checks that all prerequisites and dependencies are in order and generates input files for some underlying buildsystem.
These input files will be then used to actually build the project in the **build step** by the underlying buildsystem.

A typical CMake invocation looks like the following 2 commands:

```
cmake -B build -G "Ninja" -DCMAKE_BUILD_TYPE=Release -DWEBUI=OFF 
cmake --build build --parallel 4
```

Here is what is happening in each command:

- In the first one, which is the **configure step**:

    - `-B build` in the **configure step** tells CMake to generate build files for a native buildsystem in a build directory called `build`, separate from the main source file hierarchy.
        This is also known as an "out-of-source build"
    - `-G Ninja` tells CMake to use its Ninja generator to generate input files for the `ninja` buildsystem.
        This is the tool that will actually build the project
    - `-DCMAKE_BUILD_TYPE=Release` is a special CMake **"configure-time"** option to select the build type/configuration for single-config generators like the Ninja generator.
    - `-DWEBUI=OFF` is a **"configure-time"** option specific to the qBittorrent project that tells CMake to not build the WebUI components.

At the end of the **configure step** CMake will print some useful information to the terminal detailing the result of the configuration.

- In the second one, which is the **build step**:

    - `--build build` tells CMake to build the build tree generated under the `build/` directory (it is created if it doesn't already exist). All resulting build outputs will be under this directory when the build finishes.
    - Since in this example a single-config generator is being used, the `--config <BuildType>` argument is not passed (it would be ignored anyway), since the build configuration was already selected at **"configure-time"**.
    - `--parallel 4` tells CMake to instruct the underlying buildsytem to use 4 CPU cores for compilation. If not specified, the underlying buildsystem's default is used (for example, `Ninja` defaults to all available cores, `Unix Makefiles` defaults to 1).
    - It is possible to pass additional options directly to the underlying buildsystem after `--`. For example, `cmake --build build -- -j 4` would do the same thing as `cmake --build build --parallel 4` when using the `Unix Makefiles` generator.

The following subsections explain these points in further detail.

### Out-of-source builds

CMake encourages you to use "out-of-source builds" and has native support for such a workflow.
In an out-of-source build, all build outputs (object files, executables, ...) go into a build directory separate from the main source file hierarchy.

The build directory in CMake is specified by the `-B` flag at **configure time**. For example, `-B build`.

These are the main benefits of an out-of-source build vs. an in-source build:

- Your source tree is always clean no matter what
- As a consequence of the first point, your git working tree never gets polluted with build outputs
    - this is only possible because in the case of qBittorrent, the whole `build/` subtree is set to be ignored via `.gitignore`, so you are encouraged to create your build trees under `build/` or subdirectories inside it.
- You may have several different build directories at the same time, each containing build trees configured with different options.

To delete all the generated CMake files and build outputs in a build directory, you simply delete the whole build directory.
If you just want to clean the build outputs inside each build directory, you can of course execute the `clean` target or equivalent of the underlying build tool.

### CMake generators

The type of buildsystem generated in the **configure step** depends on the selected "CMake generator". If none is specified, CMake will choose a sensible default for the platform.

The CMake generator can be specified with the `-G` flag in the **configure step**, for example, `-G "Ninja"`

Typical choices would be:

- `Ninja`, generates files for the `ninja` build system, a cross-platform build system focused on speed. This is often the preferred choice whenever possible.
- `Ninja Multi-Config`, multi-config variant of the "Ninja" generator; see single-config vs. multi-config below.
- `Unix Makefiles`, traditional Makefiles for most Unix platforms, to be processed by `make` or equivalent.
- `Visual Studio 16 2019`, MSVC project files to be build using the MSVC toolchain on Windows.
- `Xcode`, Xcode project files to be built using the Xcode toolchain on Apple platforms such as macOS.

### Single-config vs. Multi-config generators, build types and configurations

CMake build types/configuration set certain defaults for compiler flags and other common options.
For example, on my Ubuntu machine with GCC, the `Release` build type ensures the `-O3 -DNDEBUG` flags are added to the C++ compiler command line.

The build type/configuration selection is a bit different depending on whether a single- or multi-config generator is used:

- Single-config generators generate build trees for only one configuration at a time.

    - The configuration is specified via `-DCMAKE_BUILD_TYPE=` in the **configure step**. For example, `-DCMAKE_BUILD_TYPE=RelWithDebInfo`.
    - If none is specified, the default is none.
    - To change configuration, one must generate a new build tree with a new configuration type each time.
    - The `--config` flag is ignored in the **build step**, if present.

- Multi-config generators generate several configurations at a time.

    - The build configuration is specified by the `--config` flag in the **build step**. For example `--config RelWithDebInfo`).
    - If none is specified, the default is `Debug`.
    - To change configuration, one just has to execute the build step on the same source tree with a different `--config` argument
    - The `-DCMAKE_BUILD_TYPE=` flag of the **configure step** is ignored, if present.

See the CMake documentation for more information about build types/configurations, including a list of built-in build types/configurations.


### Changing **configure-time** options

 After you have run at least one configure command you can see and edit the project settings using either the Qt-based CMake GUI, or `ccmake` for an interactive TUI experience:

```bash
ccmake .
```

To specify C++ flags that should be used for all build types, change the [`CMAKE_CXX_FLAGS`](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_FLAGS.html) variable.
To specify C++ flags that should only be used for a certain build type/configuration, change the [`CMAKE_CXX_FLAGS_<CONFIG>`](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_FLAGS_CONFIG.html) variables, such as `CMAKE_CXX_FLAGS_RELEASE`.
Note that this overwrites any previous value that these variables held.

For example, to always use the GCC flags `-w` (no warnings) and `-s` (strip unneeded symbols):

```bash
cmake -B build -DCMAKE_CXX_FLAGS="-w -s" # ...
```

### Exporting a [JSON Compilation Database](https://clang.llvm.org/docs/JSONCompilationDatabase.html)

Passing `-DCMAKE_EXPORT_COMPILE_COMMANDS=ON` at **configure-time** when using single-config generators will cause CMake to output a compilation database file (`compile_commands.json`) containing information about the effective command lines used to compile each file.

This can be useful for both manually inspecting the effective command lines that will be used to compile each file, as well as to feed into tools such as `clangd` to provide language intelligence features to smart IDEs/text editors that support the Language Server Protocol (LSP) and have a suitable C++ client implementation for it.

### Build tool invocation

The preferred way to actually invoke the underlying buildsystem to execute the build is with the `cmake --build ...` command.
This allows platform-agnostic invocation in a majority of scenarios.

The build is quiet by default with some generators such as Ninja. To enable verbose build, the recommended practice is to pass the `--verbose` flag at **build-time**:

```bash
cmake --build build --verbose
```

If more fine-grained control is required, CMake also allows passing options directly to the underlying build system after `--`.

### Dependencies

In the **configure step**, CMake checks your system for the required packages that qBittorrent depends on.

In some systems, CMake may find them automatically in your standard system installation directories without any other further intervention.
In others, you may need to supply hints in the form of options passed in the **configure step** to tell CMake where to find some packages.

For example, if CMake fails the **configure step** complaining that it can't find OpenSSL on Windows, you need to pass `-DOPENSSL_ROOT_DIR=C:\path\to\OpenSSL_root_directory` in the **configure step**.
CMake usually tells you what exactly it wants and the name of the variable to pass it as.

### Installation and stripping binaries

You can run the executable from the build directory, but you also may want to install the build artifacts into system directories.

This can be done with (you may need elevated privileges, depending on the install destination):

```
cmake --install build_dir
```

The install destination can be controlled via the `CMAKE_INSTALL_PREFIX` variable at **configure-time**.

CMake does not strip binaries by default. To install stripped binaries, pass the `--strip` flag to the install command:

```
make --install build_dir --strip
```

Note that stripped binaries will produce useless stacktraces.