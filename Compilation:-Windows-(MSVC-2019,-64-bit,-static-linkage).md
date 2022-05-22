# Introduction

**NOTE:** for qBittorrent revisions older than `63ff5e3` (2020-09-19), use the [legacy guide](https://github.com/qbittorrent/qBittorrent/wiki/Compiling-with-MSVC-2019-(static-linkage)) instead.

This page describes how to compile qBittorrent 64-bit/x64/x86-64/amd64(*) (static linkage) using MSVC 2019 under Windows 10(**).

(*): as of writing, compiling 32-bit/x86 versions is also possible but not recommended.

(**): as of writing, it is still possible to compile on/for older versions of Windows, but support for that should be dropped soon.

# Install the toolchain and dependencies

## Toolchain

NOTE: For those who don't have a Windows system or would like a clean isolated environment for compiling qBittorrent, Microsoft provides [free VMs for developers compatible with many mainstream virtualization platforms](https://developer.microsoft.com/en-us/windows/downloads/virtual-machines/). These VMs have all or most of these tools already installed and set up for you.

You will need:

- The MSVC 2019 compiler. You can get it by installing [Visual Studio 2019 Community Edition](https://docs.microsoft.com/en-us/visualstudio/releases/2019/release-notes).
- The following MSVC 2019 individual components (open Visual Studio and go to `Tools` -> `Get Tools and Features`). Some or all of these may already be installed (where applicable, feel free to choose between the Spectre-mitigated alternatives or the normal versions):
    - `MSBuild`
    - `C++ core features`
    - `Text template transformation`
    - `C++ 2019 Redistributable Update`
    - `C++ ATL for latest v142 build tools with Spectre Mitigations (x86 & x64)`
    - `C++ MFC for latest v142 build tools with Spectre Mitigations (x86 & x64)`
    - `MSVC v142 - VS 2019 C++ x64/x86 build tools` (whatever the latest version is)
    - `MSVC v142 - VS 2019 C++ x64/x86 build Spectre-mitigated libs` (whatever the latest version is)
    - `Windows 10 SDK` (whatever the latest version is)
- [CMake](https://cmake.org/download/) (the portable zip version is recommended) and [Ninja](https://github.com/ninja-build/ninja/releases) (download them and [add the directories where the executables are located to the system's `PATH`](https://www.architectryan.com/2018/03/17/add-to-the-path-on-windows-10/))
- [git for Windows](https://git-scm.com/download/win) or `git` via WSL/WSL 2, if you have WSL/WSL 2 enabled.

You will need to use either `cmd` or `pwsh` (Powershell Core) with the appropriate MSVC environment variables set up to run most commands in this guide successfully:

- For `cmd`, always run it from the `x64 Native Tools Command Prompt for VS 2019` shortcut that Visual Studio installs.
- For `pwsh`, run the following command in `cmd`:
    ```
    pwsh -NoExit -Command "&{Import-Module ""C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\Common7\Tools\Microsoft.VisualStudio.DevShell.dll""; Enter-VsDevShell -VsInstallPath ""C:\Program Files (x86)\Microsoft Visual Studio\2019\Community"" -DevCmdArguments ""-host_arch=amd64 -arch=amd64 -vcvars_spectre_libs=spectre""}"
    ```
    (to run the same command from an already-running `pwsh` instance, turn every dual double-quote into a quadruple double-quote).

Using the `Developer Powershell for VS 2019` shortcut is not recommended, since it uses the legacy `powershell` and defaults to using the 32-bit/x86 tools.

The new [Windows Terminal](https://github.com/microsoft/terminal) is recommended for comfortably using and managing multiple shells on Windows.

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

```
.\vcpkg install boost-circular-buffer:x64-windows-static boost-stacktrace:x64-windows-static openssl:x64-windows-static qt5-base:x64-windows-static qt5-svg:x64-windows-static qt5-tools:x64-windows-static qt5-winextras:x64-windows-static
```

Note that by default, `vcpkg` keeps all the buildtrees after each package installation. This is useful for patching and rebuilding, but they can take up a lot of space, so if you don't need them, you can additionally pass the `--clean-after-build` flag to the above command so that the buildtrees are deleted automatically after the installation. Or just delete them manually after the build is done.

### Install libtorrent, option 1 - just install it via `vcpkg`

```
.\vcpkg install libtorrent:x64-windows-static
```

NOTE: Currently, `vcpkg` doesn't have versioning support, so this command will install whatever `libtorrent` version was present in the commit that `vcpkg` was at when you cloned the repository. At the time of writing, you could use `--head` to get the latest `RC_1_2` commit instead, but if you want a specific commit/branch, you should go for **option 2** instead.

### Install libtorrent, option 2 - manually from source, using `vcpkg` dependencies

The easiest way to install `libtorrent`'s dependencies via `vcpkg` is to actually install it and remove it right after; `vcpkg` will install all dependencies automatically in the process. To save time, interrupt the installation process once it reaches the `libtorrent` package itself:

```
.\vcpkg install libtorrent:x64-windows-static
.\vcpkg remove libtorrent:x64-windows-static
```

Then, clone the repository, checkout your preferred commit/tag, apply any patches you want to apply, and build and install it yourself (change the toolchain file path as needed):

```pwsh
git clone https://github.com/arvidn/libtorrent.git
git checkout RC_1_2
cmake -G "Ninja" -B cmake-build-dir -DCMAKE_BUILD_TYPE=Release -DCMAKE_TOOLCHAIN_FILE="C:\path\to\vcpkg\scripts\buildsystems\vcpkg.cmake" -DVCPKG_TARGET_TRIPLET="x64-windows-static" -DBUILD_SHARED_LIBS=OFF -Dstatic_runtime=ON -Ddeprecated-functions=ON
cmake --build cmake-build-dir
cmake --install cmake-build-dir --prefix C:\some\folder\not\requiring\admin\privileges\libtorrent-install-dir
```

More information about building `libtorrent` can be found at https://libtorrent.org/building.html#building-with-cmake. Passing `-Ddeveloper-options=ON` to the configure command line will enable advanced build customization options.

Note: if you are developing/testing qBittorrent, you are encouraged to use `-Ddeprecated-functions=OFF` instead, to catch any potential use of deprecated libtorrent functionality (it will result in a compile error). Then, if you do, submit a bug report, or better yet, a PR fixing the problem :)

# Build qBittorrent

Download and extract a `.tar` archive from [the GitHub releases page](https://github.com/qbittorrent/qBittorrent/releases) or clone the repository and checkout the branch/tag of your choice.

Then, configure and build with CMake, using the `vcpkg` toolchain file (change the path to the file as needed):

- Assuming `libtorrent` was installed via `vcpkg` (**option 1**):

    ```pwsh
    cmake -G "Ninja" -B build -DCMAKE_BUILD_TYPE=Release -DCMAKE_TOOLCHAIN_FILE="C:\path\to\vcpkg\scripts\buildsystems\vcpkg.cmake" -DVCPKG_TARGET_TRIPLET="x64-windows-static" -DMSVC_RUNTIME_DYNAMIC=OFF
    cmake --build build
    ```

- Assuming `libtorrent` was built manually, but still with `vcpkg`-installed dependencies (**option 2**):

    Same as above, but also pass `-DLibtorrentRasterbar_DIR=C:\path\to\libtorrent-install-dir\lib\cmake\LibtorrentRasterbar` to the configure command line.

Check out the [common information](https://github.com/qbittorrent/qBittorrent/wiki/Compilation-with-CMake:-common-information) page to learn more about the available build configuration options (for compiling without WebUI functionality, for instance).

Once qBittorrent is built, you can run it straight from the build directory.
