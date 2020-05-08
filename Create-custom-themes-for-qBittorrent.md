# Introduction

This page contains the necessary information to get started developing custom themes for qBittorrent. They are supported since version 4.2.2.

# What are .qbttheme files?

These are theme bundles for qBittorrent.
They should contain all files required to support theming in qBittorrent and are packed using [Qt's Resource Compiler](https://doc.qt.io/qt-5/rcc.html).
qBittorrent accesses files inside `.qbttheme` using [Qt's Resource System](https://doc.qt.io/qt-5/resources.html).
Currently, qBittorrent only searches for a `stylesheet.qss` file inside a `.qbttheme` file.

# How to create your own theme bundles?

You can check out this Python script([here](https://github.com/jagannatharjun/qbt-theme/blob/master/Builds/make-resource.py)) for the easy creation of `.qbttheme` files.

At runtime, qBittorrent loads the `stylesheet.qss` file to support theming. `stylesheet.qss` is a [Qt stylesheet](https://doc.qt.io/qt-5/stylesheet.html), which is basically custom CSS for Qt.
You can read more about Qt stylesheet syntax [here](https://doc.qt.io/Qt-5/stylesheet-syntax.html). A reference is also available [here](https://doc.qt.io/qt-5/stylesheet.html).
qBittorrent uses [QResources::registerResource](https://doc.qt.io/qt-5/qresource.html#registerResource) to use the bundle file, you can imagine it like QBittorrent extracts bundle file in a special path which is `:/uitheme`, so every file should be referenced accordingly f.e let's see we have to change the image QRadiaButtons indicator, so to reference a `light/radio_button.svg` that we had it inside our rcc, we should reference it something like this
```
QRadioButton::indicator:unchecked,
QRadioButton::indicator:unchecked:focus
{
    image: url(:/uitheme/dark/radio_unchecked_disabled.svg);
}
```

# Examples

- To change Transfer List row colors based on the torrent state, one can add the following to their `stylesheet.qss`.

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
