# Introduction

VSCode is a popular open source text editor with IDE-like functionality.

This guide will help you setup a basic configuration for a good qBittorrent development experience, from editing to compiling and debugging.

Currently this guide is mostly targeted at Linux users, but Windows/macOS users should be able to follow along with possibly some tweaks to the suggested settings here and there, as most steps are the same or analogous.

# Prerequisites

## Tools

Install the following tools:

- `CMake` and `ninja` (also known as `ninja-build`), for compiling/building and project build integration in the editor via the `CMake Tools` extension.
- The `clangd` language server, to provide "language intelligence" features (code completion, compile errors, go to definition, ...). It is highly recommended to use a recent version, like 11 or later.
- `clang-tidy`, for static analysis and linting duties. It will can be executed as part of the build by `CMake` and automatically for each file as needed by `clangd`.

## VScode extensions

Install the following extensions in VSCode:

- `CMake Tools`, to provide good `CMake` project integration (but sadly, no syntax highlighting, yet).
  - ID: `ms-vscode.cmake-tools`
  - URL: https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools

- `clangd`, to provide C/C++ "language intelligence" features via the `clangd` program.
  - ID: `llvm-vs-code-extensions.vscode-clangd`
  - URL: https://marketplace.visualstudio.com/items?itemName=llvm-vs-code-extensions.vscode-clangd

- `C/C++` (from Microsoft), to provide debugging support. Later on, most of its features will be disabled to prevent conflicts with `clangd`.
  - ID: `ms-vscode.cpptools`
  - URL: https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools

# Setup

Clone the qBittorrent repository (to `/home/user/Documents/qBittorrent`, for example), and open that folder in VSCode. This will be referred to as "workspace"/"workspace root folder"/"project root" throughout the guide.

For this setup to work as intended, you will need to add some settings mentioned in the following sections to the project-level `.vscode/settings.json` file, in the workspace root.
If you find that you end up using many of these settings as-is for many projects, feel free to factor them out into the user-level settings file.

Note that this configuration is merely a suggestion with sane defaults, for the sake of this guide.
You may customize some them to your liking, after looking up what they do.
However, note that some of these settings are non-negotiable for this setup to work correctly.
For example, for `clangd` to work correctly in VSCode, the IntelliSense engine of the `C/C++` extension must not be enabled.

## Setup the `CMake Tools` extension

You can call `CMake` manually from the terminal for both configuring and/or building the project.
However, it is still worth installing and configuring this extension because it synergizes very well with `clangd`.
For example, it can be configured to automatically copy the generated `compile_commands.json` compilation database to the workspace root, where `clangd` needs it to be to generate its index in order to work correctly.
Otherwise, you would have to create a custom task to accomplish this.

In addition, the GUI can be a convenient way of running common tasks and customizing the options for the configure step command line.

In the status bar, the currently active build type and "kit" (the selected compiler) are displayed. You can change the currently active ones by clicking on them and selecting from the pop-up that appears.

These are the recommended settings:

```jsonc
{
    "cmake.configureOnOpen": false,
    "cmake.configureArgs": [
        "--graphviz=${workspaceFolder}/build/${buildType}/target_graph.dot",
    ],
    "cmake.buildDirectory": "${workspaceFolder}/build/${buildType}",
    // This is very useful to get clangd to "just work" after the first CMake build
    "cmake.copyCompileCommands": "${workspaceFolder}/compile_commands.json",
    // These are the options you would normally pass as -DVar=Value at configure-time
    "cmake.configureSettings": {
        "CMAKE_EXPORT_COMPILE_COMMANDS": true,
        // More options, including qBittorrent build configuration options go here...
    },
    "cmake.installPrefix": "/usr/local",
    "cmake.preferredGenerators": [
        "Ninja",
        "Unix Makefiles",
    ],
}
```

Note that `CMake` can also output `clang-tidy` diagnostics as part of the build, at the cost of slowing it down.
This can be accomplished by specifying a valid `clang-tidy` command-line via the `CMAKE_CXX_CLANG_TIDY` configure-time option, for example: `-DCMAKE_CXX_CLANG_TIDY="clang-tidy --checks=boost-*,bugprone-*,clang-analyzer-*"`.

If you already have a generated `.clang-tidy` file at the root of the project, the checks will be automatically picked up from there, so if you need no additional options besides those checks, you just need `-DCMAKE_CXX_CLANG_TIDY="clang-tidy"`.

## Setup the `clangd` and `C/C++` extensions

The `clangd` extension will be configured to run in the background and enable `clang-tidy` diagnostics.
It  piggy-backs off of the `compile_commands.json` compilation database generated by the `CMake` build (and copied to the root of the workspace by the `CMake` Extension) to provide `clang-tidy` diagnostics and other "language intelligence" features.

`clangd` generates an index/cache of the diagnostics for every file inside a `.cache` folder at the root of the workspace.
You might want to add this to your global `.gitignore`.
Generating this index for the first time might take a while; the indexing progress and status is shown in VSCode's status bar.

These are the recommended settings:

```jsonc
{
    // Disable most of the C/C++ extension's features, except debugging support
    "C_Cpp.autocomplete": "Disabled",
    "C_Cpp.errorSquiggles": "Disabled",
    "C_Cpp.experimentalFeatures": "Disabled",
    "C_Cpp.formatting": "Disabled",
    "C_Cpp.intelliSenseEngine": "Disabled",
    // clangd configuration
    "clangd.semanticHighlighting": true,
    "clangd.arguments": [
        // maximum number of parallel jobs; increase or decrease to your liking
        "-j=4",
        "--all-scopes-completion",
        "--background-index",
        "--completion-style=detailed",
        "--header-insertion=iwyu",
        "--suggest-missing-includes",
        "--clang-tidy",
        // select which clang-tidy checks should be enabled
        "--clang-tidy-checks=boost-*,bugprone-*,cert-*,cppcoreguidelines-*,clang-analyzer-*,misc-*,modernize-*,performance-*,portability-*,readability-*,-readability-braces-around-statements",
    ],
}
```

`clangd` runs `clang-tidy` in the background as needed for each file as they are opened, and displays the detected problems in the "Problems" view, as well as squiggles in the text buffer.
More detailed descriptions and fix-it suggestions are displayed as mouse-over hovers in the text buffer for each problem.

If you already have a generated `.clang-tidy` file at the root of the project, the checks for which diagnostics should be emitted will be automatically picked up from there, so in this case you can remove the `--clang-tidy-checks` option.

- Note: to generate a `.clang-tidy` file: `clang-tidy -checks=<checks_string> --dump-config`. Refer to the `clang-tidy` documentation to learn more.

## Setup launch configurations for debugging

Integrated debugging support is provided mainly by the `C/C++` extension and also by the `CMake Tools` extension.
There is no additional extension configuration required for debugging, but the following general debugging settings are often desirable:

```jsonc
{
    "debug.allowBreakpointsEverywhere": true,
    "debug.inlineValues": true,
    "debug.showBreakpointsInOverviewRuler": true,
}
```

Thanks to the `CMake Tools` extension, it is possible to launch the debugger with default settings for executable `CMake` targets, even without any configured launch configuration.

However, it is still recommended to create launch configurations in order to have a better overall experience.
For example, with an actual launch configuration, it is possible to use custom pretty-printing functionality, which enables better display of values Qt types in the variables view (see https://github.com/qbittorrent/qBittorrent/wiki/Setup-GDB-with-Qt-pretty-printers).

Here is an example `.vscode/launch.json` file for launching/attaching to the qBitorrent executable from either the build folder or the system installation directories with GDB and Qt pretty printer functionality:

```jsonc
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) Launch from CMake build directory",
            "type": "cppdbg",
            "request": "launch",
            "MIMode": "gdb",
            "program": "${command:cmake.launchTargetPath}",
            "cwd": "${workspaceFolder}",
            "args": [],
            "stopAtEntry": false,
            "externalConsole": false,
            // the setup commands are the same for the remaining configurations
            "setupCommands": [
                {
                    "description": "Change gdb's working directory to the root workspace folder",
                    "text": "-environment-cd ${workspaceFolder}",
                },
                {
                    "description": "Execute the commands from the .gdbinit file in the root workspace folder",
                    "text": "-interpreter-exec console \"source .gdbinit\"",
                },
                {
                    "description": "Enable custom pretty printers to work on the variables view",
                    "text": "-enable-pretty-printing",
                },
                {
                    "description": "Enable standard pretty-printing in the gdb console",
                    "text": "-gdb-set print pretty on",
                },
                {
                    "description": "GDB: no limit on print output for arrays",
                    "text": "-gdb-set print elements unlimited",
                },
                {
                    "description": "GDB: pretty print arrays",
                    "text": "-gdb-set print array on",
                },
                {
                    "description": "GDB: show array indexes",
                    "text": "-gdb-set print array-indexes on",
                }
            ]
        },
        {
            "name": "(gdb) Attach from CMake build directory",
            "type": "cppdbg",
            "request": "attach",
            "MIMode": "gdb",
            "program": "${command:cmake.launchTargetPath}",
            "processId": "${command:pickProcess}",
            "setupCommands": [
                {
                    "text": "-environment-cd ${workspaceFolder}",
                },
                {
                    "text": "-interpreter-exec console \"source .gdbinit\"",
                },
                {
                    "text": "-enable-pretty-printing",
                },
                {
                    "text": "-gdb-set print pretty on",
                },
                {
                    "text": "-gdb-set print elements unlimited",
                },
                {
                    "text": "-gdb-set print array on",
                },
                {
                    "text": "-gdb-set print array-indexes on",
                }
            ]
        },
        {
            "name": "(gdb) Launch from system install location",
            "type": "cppdbg",
            "request": "launch",
            "MIMode": "gdb",
            "program": "/usr/local/bin/qbittorrent", // adjust path as needed
            "cwd": "${workspaceFolder}",
            "args": [],
            "stopAtEntry": false,
            "externalConsole": false,
            "setupCommands": [
                {
                    "text": "-environment-cd ${workspaceFolder}",
                },
                {
                    "text": "-interpreter-exec console \"source .gdbinit\"",
                },
                {
                    "text": "-enable-pretty-printing",
                },
                {
                    "text": "-gdb-set print pretty on",
                },
                {
                    "text": "-gdb-set print elements unlimited",
                },
                {
                    "text": "-gdb-set print array on",
                },
                {
                    "text": "-gdb-set print array-indexes on",
                }
            ]
        },
        {
            "name": "(gdb) Attach from system install location",
            "type": "cppdbg",
            "request": "attach",
            "MIMode": "gdb",
            "program": "/usr/local/bin/qbittorrent", // adjust path as needed
            "processId": "${command:pickProcess}",
            "setupCommands": [
                {
                    "text": "-environment-cd ${workspaceFolder}",
                },
                {
                    "text": "-interpreter-exec console \"source .gdbinit\"",
                },
                {
                    "text": "-enable-pretty-printing",
                },
                {
                    "text": "-gdb-set print pretty on",
                },
                {
                    "text": "-gdb-set print elements unlimited",
                },
                {
                    "text": "-gdb-set print array on",
                },
                {
                    "text": "-gdb-set print array-indexes on",
                }
            ]
        },
    ]
}
```

## Setup additional tools

VScode allows defining arbitrary tasks in `.vscode/tasks.json`.

You can make use of this to drive additional tooling straight from the editor, such as [Clazy](https://github.com/KDE/clazy), for example.