# OUTDATED

We are building qBittorrent on Windows with MSVC compiler instead of MinGW. This choice was made because [http://www.mingw.org/wiki/wide_characters MinGW does not support wide characters]. MSVC compiler also generates smaller executables than MinGW.

* Download and install [http://www.microsoft.com/visualstudio/en-us/products/2010-editions/visual-cpp-express Visual C++ 2010 Express]
* Download and install [http://qt-windows-binaries.googlecode.com/svn/site/QtWindowsBinaries Qt binaries compiled with MSVC] (not compiled with MinGW)
* Add MSVC and Qt bin/ folders (''C:\Qt\4.8.2\bin'' and ''C:\Program Files\Microsoft Visual Studio 10.0\VC\bin'') to the ''PATH'' environment variable
* Download [http://downloads.sourceforge.net/project/boost/boost-jam/3.1.18/boost-jam-3.1.18-1-ntx86.zip?use_mirror=freefr Boost Jam] (bjam) and [http://sourceforge.net/projects/boost/files/boost/1.51.0/boost_1_51_0.zip/download Boost library] (v1.42-1.51 are known to work) and unzip them
* Compile boost using the following command in a terminal (bjam must be in the PATH)
 bjam -q --with-filesystem --with-thread --toolset=msvc variant=release link=static runtime-link=shared 
* Define ''BOOST_ROOT'' environment variable and point to the boost source folder (you will need to close and reopen your terminal window after that)
* Download [http://www.slproweb.com/download/Win32OpenSSL-0_9_8x.exe OpenSSL v0.9.x] and install it
* Copy OpenSSL include folder (''C:\OpenSSL\include\openssl'') to MSVC include/ folder (''C:\Program Files\Microsoft Visual Studio 10.0\VC\include'')
* Download [http://code.google.com/p/libtorrent/downloads/detail?name=libtorrent-rasterbar-0.16.5.tar.gz&can=2&q= libtorrent] and extract it (v0.14.10-v0.16.0svn are known to work)
* Compile libtorrent using the following command in a terminal:
 bjam -q --without-python --toolset=msvc variant=release link=static runtime-link=shared logging=none geoip=static dht-support=on boost=source define="BOOST_FILESYSTEM_VERSION=2" encryption=openssl
* Download a recent qBittorrent
** GIT repository is located at ''git://github.com/cdumez/qBittorrent.git''
** Tarballs are available from sourceforge [http://sourceforge.net/projects/qbittorrent/files/qbittorrent/ here]
* Download and install [http://qt.nokia.com/downloads/qt-creator-binary-for-windows Qt Creator IDE]
** Configure Qt Creator to use Qt-msvc
* Start QtCreator IDE and load qBittorrent project file (qbittorrent.pro)
* Edit ''winconf.pri'' to point to the correct include and lib paths on your file system (boost, libtorrent, OpenSSL)
** boost libs are generated in ''boost_1_51\stage\lib''
** libtorrent lib is generated in ''bin\msvc-10.0express\release\boost-source\geoip-static\link-static\threading-multi\''
** OpenSSL libs are located at ''C:\OpenSSL\lib\VC'' as a default
* Build qBittorrent using QtCreator, it should compile fine!