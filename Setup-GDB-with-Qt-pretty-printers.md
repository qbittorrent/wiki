# Introduction

When debugging qBittorrent with GDB, values of variables with Qt types such as `QByteArray` and  `QString` are not easily printable.

Fortunately, GDB has a pretty printer functionality that enables it to easily print such values in human readable form.

This guide assumes you are relatively inexperienced with GDB, and either don't know what a `.gdbinit` file is or don't use one. If you are an experienced user with an existing `.gdbinit` setup, you should be able follow along and customize your existing setup as needed.

# Setup

1. Clone the qBittorrent repository if you haven't done so already. In this example, it is assumed it was cloned to `/home/user/Documents/qBittorrent`

2. Create a folder called `.gdb` inside the cloned repository folder (`/home/user/Documents/qBittorrent`). Inside `.gdb` create another folder called `qt5prettyprinters`.

3. Download the `qt.py` and `helper.py` files from [here](https://invent.kde.org/kdevelop/kdevelop/-/tree/master/plugins/gdb/printers)  ([backup link](https://github.com/KDE/kdevelop/tree/master/plugins/gdb/printers)) and place them inside the `.gdb/qt5prettyprinters` folder.

4. Create a file called `.gdbinit` inside the cloned repository folder with the following contents:
    ```python
    python

    import sys, os

    print(f".gdbinit Python: current working directory is {os.getcwd()}")
    print(f".gdbinit Python: adding custom pretty-printers directory to the GDB path: {os.getcwd() + '/.gdb/qt5prettyprinters'}")

    sys.path.insert(0, "./.gdb/qt5prettyprinters")

    from qt import register_qt_printers
    register_qt_printers (None)

    end
    ```

# Usage

After starting a debugging session with qBittorrent (`gdb qbittorrent`, for example), execute `source .gdbinit` in  the GDB console. You should see some output informing you that the pretty printers have been loaded.

```gdb
(gdb) source .gdbinit
.gdbinit Python: current working directory is /home/user/Documents/qBittorrent
.gdbinit Python: adding custom pretty-printers directory to the GDB path: /home/user/Documents/qBittorrent/.gdb/qt5prettyprinters
```

Now, you can use GDB as you usually would. Whenever you print a variable with a Qt type, the output is immediately human-readable without additional effort.

For example, suppose you are at a breakpoint and print the value of a variable named `data`, which is a `QByteArray`, containing an HTTP request.

- output without pretty printers:
    ```
    (gdb) print data
    $1 = (const QByteArray &) @0x555555e2ecd0: {d = 0x555555e46610}
    ```

- output with pretty printers:

    ```
    (gdb) print data
    $1 = "POST /api/v2/auth/login HTTP/1.1\r\nHost: localhost:8080\r\nUser-Agent: python-requests/2.22.0\r\nAccept-Encoding: gzip, deflate\r\nAccept: */*\r\nConnection: keep-alive\r\nContent-Length: 34\r\nContent-Type: appli"... = {[0] = 80 'P', [1] = 79 'O', 
  [2] = 83 'S', [3] = 84 'T', [4] = 32 ' ', [5] = 47 '/', [6] = 97 'a', [7] = 112 'p', [8] = 105 'i', [9] = 47 '/', [10] = 118 'v', [11] = 50 '2', [12] = 47 '/', [13] = 97 'a', [14] = 117 'u', [15] = 116 't', [16] = 104 'h', [17] = 47 '/', [18] = 108 'l', 
  [19] = 111 'o', [20] = 103 'g', [21] = 105 'i', [22] = 110 'n', [23] = 32 ' ', [24] = 72 'H', [25] = 84 'T', [26] = 84 'T', [27] = 80 'P', [28] = 47 '/', [29] = 49 '1', [30] = 46 '.', [31] = 49 '1', [32] = 13 '\r', [33] = 10 '\n', [34] = 72 'H', 
  [35] = 111 'o', [36] = 115 's', [37] = 116 't', [38] = 58 ':', [39] = 32 ' ', [40] = 108 'l', [41] = 111 'o', [42] = 99 'c', [43] = 97 'a', [44] = 108 'l', [45] = 104 'h', [46] = 111 'o', [47] = 115 's', [48] = 116 't', [49] = 58 ':', [50] = 56 '8', 
  [51] = 48 '0', [52] = 56 '8', [53] = 48 '0', [54] = 13 '\r', [55] = 10 '\n', [56] = 85 'U', [57] = 115 's', [58] = 101 'e', [59] = 114 'r', [60] = 45 '-', [61] = 65 'A', [62] = 103 'g', [63] = 101 'e', [64] = 110 'n', [65] = 116 't', [66] = 58 ':', 
  [67] = 32 ' ', [68] = 112 'p', [69] = 121 'y', [70] = 116 't', [71] = 104 'h', [72] = 111 'o', [73] = 110 'n', [74] = 45 '-', [75] = 114 'r', [76] = 101 'e', [77] = 113 'q', [78] = 117 'u', [79] = 101 'e', [80] = 115 's', [81] = 116 't', [82] = 115 's', 
  [83] = 47 '/', [84] = 50 '2', [85] = 46 '.', [86] = 50 '2', [87] = 50 '2', [88] = 46 '.', [89] = 48 '0', [90] = 13 '\r', [91] = 10 '\n', [92] = 65 'A', [93] = 99 'c', [94] = 99 'c', [95] = 101 'e', [96] = 112 'p', [97] = 116 't', [98] = 45 '-', [99] = 69 'E', 
  [100] = 110 'n', [101] = 99 'c', [102] = 111 'o', [103] = 100 'd', [104] = 105 'i', [105] = 110 'n', [106] = 103 'g', [107] = 58 ':', [108] = 32 ' ', [109] = 103 'g', [110] = 122 'z', [111] = 105 'i', [112] = 112 'p', [113] = 44 ',', [114] = 32 ' ', 
  [115] = 100 'd', [116] = 101 'e', [117] = 102 'f', [118] = 108 'l', [119] = 97 'a', [120] = 116 't', [121] = 101 'e', [122] = 13 '\r', [123] = 10 '\n', [124] = 65 'A', [125] = 99 'c', [126] = 99 'c', [127] = 101 'e', [128] = 112 'p', [129] = 116 't', 
  [130] = 58 ':', [131] = 32 ' ', [132] = 42 '*', [133] = 47 '/', [134] = 42 '*', [135] = 13 '\r', [136] = 10 '\n', [137] = 67 'C', [138] = 111 'o', [139] = 110 'n', [140] = 110 'n', [141] = 101 'e', [142] = 99 'c', [143] = 116 't', [144] = 105 'i', 
  [145] = 111 'o', [146] = 110 'n', [147] = 58 ':', [148] = 32 ' ', [149] = 107 'k', [150] = 101 'e', [151] = 101 'e', [152] = 112 'p', [153] = 45 '-', [154] = 97 'a', [155] = 108 'l', [156] = 105 'i', [157] = 118 'v', [158] = 101 'e', [159] = 13 '\r', 
  [160] = 10 '\n', [161] = 67 'C', [162] = 111 'o', [163] = 110 'n', [164] = 116 't', [165] = 101 'e', [166] = 110 'n', [167] = 116 't', [168] = 45 '-', [169] = 76 'L', [170] = 101 'e', [171] = 110 'n', [172] = 103 'g', [173] = 116 't', [174] = 104 'h', 
  [175] = 58 ':', [176] = 32 ' ', [177] = 51 '3', [178] = 52 '4', [179] = 13 '\r', [180] = 10 '\n', [181] = 67 'C', [182] = 111 'o', [183] = 110 'n', [184] = 116 't', [185] = 101 'e', [186] = 110 'n', [187] = 116 't', [188] = 45 '-', [189] = 84 'T', 
  [190] = 121 'y', [191] = 112 'p', [192] = 101 'e', [193] = 58 ':', [194] = 32 ' ', [195] = 97 'a', [196] = 112 'p', [197] = 112 'p', [198] = 108 'l', [199] = 105 'i'...}
    ```

The pretty printer output is further customizable via the usual GDB options for output customization. For example, you can suppress output of individual array elements, or reduce the number of array elements printed.