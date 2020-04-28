# Introduction

As of version 4.2.2, custom unofficial Qt UI themes can be used on all platforms. This is mostly relevant for Windows users, because on Windows there is currently no native themeing integration.

# Procedure

You can find some unofficial themes here: [List of unofficial themes](https://github.com/qbittorrent/qBittorrent/wiki/List-of-known-qBittorrent-themes)

After you have downloaded your chosen `.qbtheme` theme file, using it is quite easy:

1. Enable `Menu->Tools->Options->Behavior->Interface->Use custom UI Theme` and select the `.qbtheme` file
2. Restart QBittorrent for the `.qbtheme` to take effect.

![example theme setup](https://user-images.githubusercontent.com/36061843/77887653-cef5f680-721f-11ea-88ab-d7b33c190f9b.png)

# What are .qbttheme files?
These are theme bundles for qBittorrent. This should contain all files required to support theming in qBittorrent and are packed using [Qt's Resource Compiler](https://doc.qt.io/qt-5/rcc.html). QBittorrent accesses files inside `.qbttheme` using [Qt's Resource System](https://doc.qt.io/qt-5/resources.html), currently, QBittorrent only searches `stylesheet.qss` inside `.qbttheme` file.

# How to create your own theme bundles?
You can check out this Python script([here](https://github.com/jagannatharjun/qbt-theme/blob/master/Builds/make-resource.py)) for the easy creation of .qbttheme file. QBitTorrent mounts `.qbttheme` file in `:uitheme` prefix, so every file should be referenced accordingly inside `stylesheet.qss`. QBittorrent loads `stylesheet.qss` to support theming. `stylesheet.qss` is basically a [Qt's style sheet](https://doc.qt.io/qt-5/stylesheet.html). You can read more about stylesheet's syntax [here](https://doc.qt.io/Qt-5/stylesheet-syntax.html) and a reference can be found [here](https://doc.qt.io/qt-5/stylesheet.html) 

# Special Cases
For changing Transfer List row colors which are based on concerned torrent state, one can add the following to their `stylesheet.qss`.

```css
TransferListWidget 
{
  qproperty-downloadingStateForeground: limegreen;
  qproperty-forcedDownloadingStateForeground: limegreen;
  qproperty-downloadingMetadataStateForeground: limegreen;
  qproperty-allocatingStateForeground: #cccccc;
  qproperty-stalledDownloadingStateForeground: #cccccc;
  qproperty-stalledUploadingStateForeground: #cccccc;
  qproperty-uploadingStateForeground: #63b8ff;
  qproperty-forcedUploadingStateForeground: #63b8ff;
  qproperty-pausedDownloadingStateForeground: #fa8090;
  qproperty-pausedUploadingStateForeground: #4f94cd;
  qproperty-errorStateForeground: red;
  qproperty-missingFilesStateForeground: red;
  qproperty-queuedDownloadingStateForeground: #00cdcd;
  qproperty-queuedUploadingStateForeground: #00cdcd;
  qproperty-checkingDownloadingStateForeground: #00cdcd;
  qproperty-checkingUploadingStateForeground: #00cdcd;
  qproperty-checkingResumeDataStateForeground: #00cdcd;
  qproperty-movingStateForeground: #00cdcd;
  qproperty-unknownStateForeground: red; 
}
```



 