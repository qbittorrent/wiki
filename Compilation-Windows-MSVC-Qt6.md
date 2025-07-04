# Introduction

This is a short guide for compiling qBittorrent 64-bit/x64/x86-64/amd64 (static linkage) with Qt6 using MSVC under Windows 10.

# Install the toolchain and dependencies

## Toolchain

NOTE: For those who don't have a Windows system or would like a clean isolated environment for compiling qBittorrent, Microsoft provides [free VMs for developers compatible with many mainstream virtualization platforms](https://developer.microsoft.com/en-us/windows/downloads/virtual-machines/). These VMs have all or most of these tools already installed and set up for you.

You will need:

- The MSVC compiler. As of writing, latest version of Visual Studio is Visual Studio 2022, You can get it by installing [Visual Studio 2022 Community Edition](https://learn.microsoft.com/en-us/visualstudio/releases/2022/release-notes).
- The following individual components (open Visual Studio and go to `Tools` -> `Get Tools and Features`). Some or all of these may already be installed (where applicable, feel free to choose between the Spectre-mitigated alternatives or the normal versions):
    - `MSBuild`
    - `C++ core features`
    - `Text template transformation`
    - `C++ 2019 Redistributable Update`
    - `C++ ATL for latest v143 build tools with Spectre Mitigations (x86 & x64)`
    - `C++ MFC for latest v143 build tools with Spectre Mitigations (x86 & x64)`
    - `MSVC v143 - VS 2022 C++ x64/x86 build tools` (whatever the latest version is)
    - `MSVC v143 - VS 2022 C++ x64/x86 build Spectre-mitigated libs` (whatever the latest version is)
    - `Windows 10 SDK` (or any version SDK if you want to run qBittorrent on specific Windows)
- [CMake](https://cmake.org/download/) (the portable zip version is recommended) and [Ninja](https://github.com/ninja-build/ninja/releases) (download them and [add the directories where the executables are located to the system's `PATH`](https://www.architectryan.com/2018/03/17/add-to-the-path-on-windows-10/))
- [git for Windows](https://git-scm.com/download/win) or `git` via WSL/WSL 2, if you have WSL/WSL 2 enabled.

You will need to use either `cmd` or `pwsh` (Powershell Core) with the appropriate MSVC environment variables set up to run most commands in this guide successfully:

- For `cmd`, always run it from the `x64 Native Tools Command Prompt for VS 2022` shortcut that Visual Studio installs.
- For `pwsh`, run the following command in `cmd`:
    ```
    pwsh.exe -NoExit -Command "&{Import-Module """C:\Program Files\Microsoft Visual Studio\2022\Community\Common7\Tools\Microsoft.VisualStudio.DevShell.dll"""; Enter-VsDevShell 9f1439d7 -SkipAutomaticLocation -DevCmdArguments """-arch=x64 -host_arch=x64"""}"
    ```
    (to run the same command from an already-running `pwsh` instance, turn every dual double-quote into a quadruple double-quote).

Using the `Developer Powershell for VS 2022` shortcut is not recommended, since it uses the legacy `powershell` and defaults to using the 32-bit/x86 tools.

The new [Windows Terminal](https://learn.microsoft.com/en-us/windows/terminal/) is recommended for comfortably using and managing multiple shells on Windows. You can also add a pwsh profile for VS 2022 in Windows Terminal.

## Dependencies (static linkage)

None of qBittorrent's dependencies have officially released statically linked builds. Thus, the best way to get good general-purpose builds of recent versions of the dependencies easily is to install them via the `vcpkg` package manager. `vcpkg` can also be used to install `libtorrent`.

The nice thing about `vcpkg` is that it also allows you to easily patch any of the packages that it installs, should you need/want to do so. Furthermore, you can even compile some dependencies with `vcpkg`, others manually, and mix-and-match when building qBittorrent.

### Install and build `vcpkg`

```
git clone https://github.com/microsoft/vcpkg
cd .\vcpkg
.\bootstrap-vcpkg.bat -disableMetrics
.\vcpkg integrate install
```

### Install the base dependencies

Install the base dependencies except Qt6.

```
.\vcpkg install boost-circular-buffer:x64-windows-static boost-stacktrace:x64-windows-static openssl:x64-windows-static
```

#### Install Qt6

C++20 support is not enabled by default on `vcpkg`. You need to patch portfile in vcpkg manually.

```
list(APPEND _qarg_OPTIONS "-DQT_FEATURE_cxx20:BOOL=ON")
list(APPEND _qarg_OPTIONS "-DQT_DISABLE_DEPRECATED_UP_TO=0x060500")
```
Add those two line to `root\of\vcpkg\ports\qtbase\cmake\qt_install_submodule.cmake`, before CMake function `vcpkg_cmake_configure`. As of writing, current Qt6 on `vcpkg` is Qt 6.7.3. This patch may not be need or not applicable in later version of Qt. It is not a beautiful way to build Qt6 with C++20.

The first line enables C++20 support. The second line removes deprecated before Qt 6.5.0. Other version numbers may also work, but 6.5.0 just works fine.

Install Qt6

```
./vcpkg install qtbase:x64-windows-static qtsvg:x64-windows-static qttools:x64-windows-static
```

Note that by default, `vcpkg` keeps all the buildtrees after each package installation. This is useful for patching and rebuilding, but they can take up a lot of space, so if you don't need them, you can additionally pass the `--clean-after-build` flag to the above command so that the buildtrees are deleted automatically after the installation. Or just delete them manually after the build is done.

### Install libtorrent

```
.\vcpkg install libtorrent:x64-windows-static
```

NOTE: Currently, `vcpkg` doesn't have versioning support, so this command will install whatever `libtorrent` version was present in the commit that `vcpkg` was at when you cloned the repository. At the time of writing, you could use `--head` to get the latest `RC_1_2` commit instead, but if you want a specific commit/branch, you should build from source instead.

# Build qBittorrent

Download and extract a `.tar` archive from [the GitHub releases page](https://github.com/qbittorrent/qBittorrent/releases) or clone the repository and checkout the branch/tag of your choice.

Then, configure and build with CMake, using the `vcpkg` toolchain file (change the path to the file as needed):

- Assuming `libtorrent` was installed via `vcpkg`:

    ```pwsh
    cmake -G "Ninja" -B build -DCMAKE_BUILD_TYPE=Release -DCMAKE_TOOLCHAIN_FILE="C:\path\to\vcpkg\scripts\buildsystems\vcpkg.cmake" -DVCPKG_TARGET_TRIPLET="x64-windows-static" -DMSVC_RUNTIME_DYNAMIC=OFF
    cmake --build build
    ```

- Assuming `libtorrent` was built manually, but still with `vcpkg`-installed dependencies:

    Same as above, but also pass `-DLibtorrentRasterbar_DIR=C:\path\to\libtorrent-install-dir\lib\cmake\LibtorrentRasterbar` to the configure command line.

Check out the [common information](https://github.com/qbittorrent/qBittorrent/wiki/Compilation-with-CMake-common-information) page to learn more about the available build configuration options (for compiling without WebUI functionality, for instance).

Once qBittorrent is built, you can run it straight from the build directory.
