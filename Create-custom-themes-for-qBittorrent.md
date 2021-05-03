## Introduction

This page contains the necessary information to get started developing custom themes for qBittorrent. They are supported since version 4.2.2.

## What are .qbttheme files?

These are theme bundles for qBittorrent.
They should contain all files required to support theming in qBittorrent and are packed using [Qt's Resource Compiler](https://doc.qt.io/qt-5/rcc.html).
qBittorrent accesses files inside `.qbttheme` using [Qt's Resource System](https://doc.qt.io/qt-5/resources.html).
Currently, qBittorrent only searches for a `stylesheet.qss` and `config.json` inside a `.qbttheme` file but you can also add your own custom resources (read more [here](https://github.com/qbittorrent/qBittorrent/wiki/Create-custom-themes-for-qBittorrent#using-custom-resources-with-bundles)).

## How to create your own theme bundles?

You can check out this Python script([here](https://github.com/jagannatharjun/qbt-theme/blob/master/Builds/make-resource.py)) for the easy creation of `.qbttheme` files.

At runtime, qBittorrent loads only `stylesheet.qss` to support theming and `config.json` which currently only manages GUI colors. You can include custom resources with your bundle files to use it with `stylesheet.qss`

## Managing overall feel of GUI
`stylesheet.qss` is a [Qt stylesheet](https://doc.qt.io/qt-5/stylesheet.html), which is basically custom CSS for Qt.
You can read more about Qt stylesheet syntax [here](https://doc.qt.io/Qt-5/stylesheet-syntax.html). A reference is also available [here](https://doc.qt.io/qt-5/stylesheet.html).

## Changing QBittorrent specific colors
for changing qBittorrent specific GUI colors, you have to use config.json, NOTE: A large part of colors is already changeable from stylesheet but the following are QBitTorrent's specific context-sensitive colors,  

```json
{
    "colors": {
        "Palette.Window": "<color>",
        "Palette.WindowText": "<color>",
        "Palette.Base": "<color>",
        "Palette.AlternateBase": "<color>",
        "Palette.Text": "<color>",
        "Palette.ToolTipBase": "<color>",
        "Palette.ToolTipText": "<color>",
        "Palette.BrightText": "<color>",
        "Palette.Highlight": "<color>",
        "Palette.HighlightedText": "<color>",
        "Palette.Button": "<color>",
        "Palette.ButtonText": "<color>",
        "Palette.Link": "<color>",
        "Palette.LinkVisited": "<color>",
        "Palette.Light": "<color>",
        "Palette.Midlight": "<color>",
        "Palette.Mid": "<color>",
        "Palette.Dark": "<color>",
        "Palette.Shadow": "<color>",
        "Palette.WindowTextDisabled": "<color>",
        "Palette.TextDisabled": "<color>",
        "Palette.ToolTipTextDisabled": "<color>",
        "Palette.BrightTextDisabled": "<color>",
        "Palette.HighlightedTextDisabled": "<color>",
        "Palette.ButtonTextDisabled": "<color>",
        
        "Log.Time": "<color>",
        "Log.Normal": "<color>",
        "Log.Info": "<color>",
        "Log.Warning": "<color>",
        "Log.Critical": "<color>",
        "Log.BannedPeer": "<color>",
        
        "TransferList.Downloading": "<color>",
        "TransferList.StalledDownloading": "<color>",
        "TransferList.DownloadingMetadata": "<color>",
        "TransferList.ForcedDownloading": "<color>",
        "TransferList.Allocating": "<color>",
        "TransferList.Uploading": "<color>",
        "TransferList.StalledUploading": "<color>",
        "TransferList.ForcedUploading": "<color>",
        "TransferList.QueuedDownloading": "<color>",
        "TransferList.QueuedUploading": "<color>",
        "TransferList.CheckingDownloading": "<color>",
        "TransferList.CheckingUploading": "<color>",
        "TransferList.CheckingResumeData": "<color>",
        "TransferList.PausedDownloading": "<color>",
        "TransferList.PausedUploading": "<color>",
        "TransferList.Moving": "<color>",
        "TransferList.MissingFiles": "<color>",
        "TransferList.Error": "<color>"     
    }
}
```
Here 
1. Palette referes to [QPalette](https://doc.qt.io/qt-5/qpalette.html), and following string(after .) denotes [Color roles](https://doc.qt.io/qt-5/qpalette.html#ColorRole-enum)
2. Log refers to log messages and the following string (after .) denotes the type of messages
3. Transfer List refers to central view containing all of your torrents entries and the following string(after .) denotes torrent state on which based on which row colors would be decided. 

`<color>` value supports normal rgb values(#rrggbb) or svg color names. It follows [Qt's named color convention](https://doc.qt.io/qt-5/qcolor.html#setNamedColor) convention

This is introduced in qBittorrent v4.3.0

## Using custom resources with bundles
qBittorrent uses [QResources::registerResource](https://doc.qt.io/qt-5/qresource.html#registerResource) to use the bundle file, you can imagine it like QBittorrent extracts bundle file in a special path which is `:/uitheme`, so every file should be referenced accordingly f.e let's see you have to change the image QRadiaButtons indicator, so to reference a `light/radio_button.svg` inside your bundle file, you should do something like this
```
QRadioButton::indicator:unchecked,
QRadioButton::indicator:unchecked:focus
{
    image: url(:/uitheme/light/radio_button.svg);
}
```

reserved files and their structure in bundle files are - `stylesheet.qss` and `config.json` both should occur at the root of your bundle file.

## Changing icons of GUI

Starting with v4.3.0 you can change the icons of GUIs too, you just have to include your icons in the theme bundle under the icons prefix. The icon name should be the same as the originals icons. See all the icons here (https://github.com/qbittorrent/qBittorrent/tree/master/src/icons)
an example can be read here https://github.com/jagannatharjun/qbt-theme/issues/29#issuecomment-713305420

## Notes

Following way of changing TransferList row's color is removed after version 4.2.5, please change your bundle files accordingly and instead use `config.json`

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
