# Introduction

VSCode is a popular open source text editor with IDE-like functionality.

This is guide will help you setup a basic configuration for a good qBittorrent development experience, from editing to compiling and debugging.

Currently this guide is mostly targeted at Linux users, but Windows/macOS users should be able to follow along, as most steps are the same or analogous.

# Extensions

The main extension required is MS's VSCode C/C++ tools:

- ID: `ms-vscode.cpptools`
- URL: https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools

Optionally, you may also install the CMake Tools extension, for integration with qBittorrents (still experimental) CMake build support.

- ID: `ms-vscode.cmake-tools`
- URL: https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools

# Workspace settings

After cloning the qBittorrent repository (to `/home/user/Documents/qBittorrent`, for example), open the folder in VSCode. This is now your workspace.

## Editing
**NOTE** If you're using CMake Tools, it should be able to create `c_cpp_properties.json` itself and you wouldn't need to do following steps,


For all editor features such as code completion, navigation, hovers and error reporting to work properly you need to configure the C/C++ settings.

1. Create a folder called `.vscode` in the root of the workspace, if it does not exist already.

2. Create a file called `c_cpp_properties.json` inside the `.vscode` folder if it doesn't already exist with the following contents:
    ```
    {
        "configurations": [
            {
                "name": "Linux",
                "includePath": [
                    "${workspaceFolder}/**",
                    "/usr/include/libtorrent/**",
                    "/usr/include/x86_64-linux-gnu/qt5/**"
                ],
                "defines": ["QBT_VERSION_MAJOR=4", "QBT_VERSION_MINOR=3", "QBT_VERSION_BUGFIX=0", "QBT_VERSION_BUILD=0", "QBT_VERSION=\"v4.3.0alpha1\"", "QBT_VERSION_2=\"4.3.0alpha1\""],
                "cppStandard": "c++17"
            }
        ],
        "version": 4
    }
    ```
    - You may change the paths to the `Qt` and `libtorrent` headers in the `includePath`. The paths in this example are the defaults for Ubuntu.
    - If using an alternative Boost library version (if you compiled from source, for example), make sure to specify the proper include path for it as well (e.g. `"/opt/boost/**"`).
    - You may change the `QBT_VERSION*` `defines` to your preference. This is only needed because VSCode does not read the version defines from the `version.pri` file, so it throws errors whenever it sees them otherwise.
    - Feel free to browse the documentation of this file and customize more of the available options to your liking.

## Compiling

### Manually

You can use VSCode's integrated terminal to run qBittorrent's traditional compilation commands.

For example, on Ubuntu:

```bash
./configure --enable-debug
make -j2
# optional: creates a .deb package and installs to /usr/local/*
sudo checkinstall --nodoc --strip=no --stripso=no --pkgname qbittorrent --pkgversion 4.2.x-test
```

For more information see https://github.com/qbittorrent/qBittorrent/wiki#compilation

### Using tasks

You can use VSCode tasks to automate the above process and run it from within VSCode. See https://code.visualstudio.com/docs/editor/tasks for more information.

### Using CMake + CMake Tools extension

CMake Tools support both building and running for different targets from CMakeLists.txt, so you should be able to compile, run, and debug without needing to create `launch.json` and `tasks.json`. 

## Running and debugging

Create a `launch.json` inside the `.vscode` folder.

Here is an example for launching/attaching either `qbittorrent` or `qbittorrent-nox` with GDB and Qt pretty printer functionality (see https://github.com/qbittorrent/qBittorrent/wiki/Setup-GDB-with-Qt-pretty-printers). Change the executable locations if needed.

You will be able to set breakpoints/watchpoints, step through the code, view variable values, call stack, etc from within VSCode.

```
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) Launch qbittorrent",
            "request": "launch",
            "type": "cppdbg",
            "program": "/usr/local/bin/qbittorrent", // change if needed
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "externalConsole": false,
            "MIMode": "gdb",
            "setupCommands": [
                {
                    // make sure we are in the root of the workspace
                    "description": "Change gdb's working directory to the root workspace folder",
                    "text": "-environment-cd ${workspaceFolder}",
                    "ignoreFailures": false
                },
                {
                    "description": "Execute the commands from the .gdbinit file in the root workspace folder",
                    "text": "-interpreter-exec console \"source .gdbinit\"",
                    "ignoreFailures": false
                },
                {
                    // this one is needed for pretty printers to work in the variables view
                    "description": "Enable custom pretty printers to work on the variables view",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": false
                },
                {
                    // this one enables standard pretty printing in the debug console
                    "description": "Enable standard pretty-printing in the gdb console",
                    "text": "-gdb-set print pretty on",
                    "ignoreFailures": false
                },
                {
                    "description": "GDB: no limit on print output for arrays",
                    "text": "-gdb-set print elements unlimited",
                    "ignoreFailures": false
                },
                {
                    "description": "GDB: pretty print arrays",
                    "text": "-gdb-set print array on",
                    "ignoreFailures": false
                },
                {
                    "description": "GDB: show array indexes",
                    "text": "-gdb-set print array-indexes on",
                    "ignoreFailures": false
                }
            ]
        },
        {   
            "name": "(gdb) Launch qbittorrent-nox",
            "type": "cppdbg",
            "request": "launch",
            "program": "/usr/local/bin/qbittorrent-nox", // change if needed
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "externalConsole": false,
            "MIMode": "gdb",
            "setupCommands": [
                {
                    // make sure we are in the root of the workspace
                    "description": "Change gdb's working directory to the root workspace folder",
                    "text": "-environment-cd ${workspaceFolder}",
                    "ignoreFailures": false
                },
                {
                    "description": "Execute the commands from the .gdbinit file in the root workspace folder",
                    "text": "-interpreter-exec console \"source .gdbinit\"",
                    "ignoreFailures": false
                },
                {
                    // this one is needed for pretty printers to work in the variables view
                    "description": "Enable custom pretty printers to work on the variables view",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": false
                },
                {
                    // this one enables standard pretty printing in the debug console
                    "description": "Enable standard pretty-printing in the gdb console",
                    "text": "-gdb-set print pretty on",
                    "ignoreFailures": false
                },
                {
                    "description": "GDB: no limit on print output for arrays",
                    "text": "-gdb-set print elements unlimited",
                    "ignoreFailures": false
                },
                {
                    "description": "GDB: pretty print arrays",
                    "text": "-gdb-set print array on",
                    "ignoreFailures": false
                },
                {
                    "description": "GDB: show array indexes",
                    "text": "-gdb-set print array-indexes on",
                    "ignoreFailures": false
                }
            ]
        },
        {
            "name": "(gdb) Attach qbittorrent",
            "type": "cppdbg",
            "request": "attach",
            "program": "/usr/local/bin/qbittorrent", // change if needed
            "processId": "${command:pickProcess}",
            "MIMode": "gdb",
            "setupCommands": [
                {
                    // make sure we are in the root of the workspace
                    "description": "Change gdb's working directory to the root workspace folder",
                    "text": "-environment-cd ${workspaceFolder}",
                    "ignoreFailures": false
                },
                {
                    "description": "Execute the commands from the .gdbinit file in the root workspace folder",
                    "text": "-interpreter-exec console \"source .gdbinit\"",
                    "ignoreFailures": false
                },
                {
                    // this one is needed for pretty printers to work in the variables view
                    "description": "Enable custom pretty printers to work on the variables view",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": false
                },
                {
                    // this one enables standard pretty printing in the debug console
                    "description": "Enable standard pretty-printing in the gdb console",
                    "text": "-gdb-set print pretty on",
                    "ignoreFailures": false
                },
                {
                    "description": "GDB: no limit on print output for arrays",
                    "text": "-gdb-set print elements unlimited",
                    "ignoreFailures": false
                },
                {
                    "description": "GDB: pretty print arrays",
                    "text": "-gdb-set print array on",
                    "ignoreFailures": false
                },
                {
                    "description": "GDB: show array indexes",
                    "text": "-gdb-set print array-indexes on",
                    "ignoreFailures": false
                }
            ]
        },
        {
            "name": "(gdb) Attach qbittorrent-nox",
            "type": "cppdbg",
            "request": "attach",
            "program": "/usr/local/bin/qbittorrent-nox", // change if needed
            "processId": "${command:pickProcess}",
            "MIMode": "gdb",
            "setupCommands": [
                {
                    // make sure we are in the root of the workspace
                    "description": "Change gdb's working directory to the root workspace folder",
                    "text": "-environment-cd ${workspaceFolder}",
                    "ignoreFailures": false
                },
                {
                    "description": "Execute the commands from the .gdbinit file in the root workspace folder",
                    "text": "-interpreter-exec console \"source .gdbinit\"",
                    "ignoreFailures": false
                },
                {
                    // this one is needed for pretty printers to work in the variables view
                    "description": "Enable custom pretty printers to work on the variables view",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": false
                },
                {
                    // this one enables standard pretty printing in the debug console
                    "description": "Enable standard pretty-printing in the gdb console",
                    "text": "-gdb-set print pretty on",
                    "ignoreFailures": false
                },
                {
                    "description": "GDB: no limit on print output for arrays",
                    "text": "-gdb-set print elements unlimited",
                    "ignoreFailures": false
                },
                {
                    "description": "GDB: pretty print arrays",
                    "text": "-gdb-set print array on",
                    "ignoreFailures": false
                },
                {
                    "description": "GDB: show array indexes",
                    "text": "-gdb-set print array-indexes on",
                    "ignoreFailures": false
                }
            ]
        }
    ]
}
```