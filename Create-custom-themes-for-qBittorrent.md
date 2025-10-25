## Introduction
This page contains the necessary information to get started developing custom themes for qBittorrent. They are supported since version 4.2.2.

As of qBittorrent 4.6.0 (Oct 2023), qBittorrent migrated to Qt6 and dropped Qt5. The theming format (.qbtheme with stylesheet.qss + config.json) remains compatible; use the matching Qt Resource Compiler (rcc) for your target version.

## What are .qbtheme files?
These are theme bundles for qBittorrent.
They should contain all files required to support theming in qBittorrent and are packed using [Qt's Resource Compiler](https://doc.qt.io/qt-6/rcc.html).
qBittorrent accesses files inside `.qbtheme` using [Qt's Resource System](https://doc.qt.io/qt-6/resources.html).
Currently, qBittorrent only searches for a `stylesheet.qss` and `config.json` inside a `.qbtheme` file but you can also add your own custom resources (read more below in “Using custom resources with bundles”).

## How to create your own theme bundles?
You can check out [this Python script](https://github.com/jagannatharjun/qbt-theme/blob/master/Builds/make-resource.py) for the easy creation of `.qbtheme` files.

Alternatively, use Qt's `rcc` directly with a simple `resources.qrc` listing your files. For Qt6-based qBittorrent (4.6.0+), build with the Qt6 `rcc`; for older Qt5-based versions (4.2.2–4.5.x), build with the Qt5 `rcc`.

At runtime, qBittorrent loads only `stylesheet.qss` to support theming and `config.json` which currently only manages GUI colors. You can include custom resources with your bundle files to use in `stylesheet.qss`.

## Managing overall feel of GUI
`stylesheet.qss` is a [Qt stylesheet](https://doc.qt.io/qt-6/stylesheet.html), which is basically custom CSS for Qt.
You can read more about Qt stylesheet syntax here: https://doc.qt.io/qt-6/stylesheet-syntax.html. A reference is also available here: https://doc.qt.io/qt-6/stylesheet-reference.html.

Notes for Qt client theming:
- Use selectors like `QWidget`, `QPushButton:hover`, `QToolBar QLineEdit`, etc.
- QSS generally takes precedence over palette colors; palettes still drive defaults like text/selection.
- Some widgets expose extra paintable properties via `qproperty-<Name>` that can be set from QSS.
- Keep resource URLs absolute inside the bundle, e.g. `url(:/uitheme/icons/name.svg)`.

## Changing qBittorrent specific colors
For changing qBittorrent-specific GUI colors, use `config.json`. NOTE: A large part of colors is already changeable from the stylesheet but the following are qBittorrent's specific context-sensitive colors.

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
1. Palette refers to [QPalette](https://doc.qt.io/qt-6/qpalette.html), and the following string (after `.`) denotes [Color roles](https://doc.qt.io/qt-6/qpalette.html#ColorRole-enum)
2. Log refers to log messages and the following string (after `.`) denotes the type of messages
3. Transfer List refers to the central view containing all of your torrents entries and the following string (after `.`) denotes the torrent state, based on which row colors are decided.

`<color>` supports normal RGB values (`#rrggbb`) or SVG color names. It follows [Qt's named color convention](https://doc.qt.io/qt-6/qcolor.html#setNamedColor).

This was introduced in qBittorrent v4.3.0.

## Using custom resources with bundles
qBittorrent uses [QResource::registerResource](https://doc.qt.io/qt-6/qresource.html#registerResource) to use the bundle file; you can imagine qBittorrent extracts the bundle file in a special path which is `:/uitheme`, so every file should be referenced accordingly. For example, to change the image for a `QRadioButton` indicator and to reference a `light/radio_button.svg` inside your bundle file, do something like this:

```css
QRadioButton::indicator:unchecked,
QRadioButton::indicator:unchecked:focus
{
    image: url(:/uitheme/light/radio_button.svg);
}
```

Reserved files and their structure in bundle files are: `stylesheet.qss` and `config.json`; both should occur at the root of your bundle file.

## Changing icons of GUI
Starting with v4.3.0 you can change the icons of the GUI too; include your icons in the theme bundle under the icons prefix. The icon name should be the same as the original icons. See the icons list here: https://github.com/qbittorrent/qBittorrent/tree/master/src/icons
An example can be read here: https://github.com/jagannatharjun/qbt-theme/issues/29#issuecomment-713305420

## Notes
The following way of changing TransferList row colors was removed after version 4.2.5. Please change your bundle files accordingly and instead use `config.json`.

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
