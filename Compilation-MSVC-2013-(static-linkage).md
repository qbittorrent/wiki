This page describes how to compile qBittorrent for both x86 and x64 platforms using Microsoft Visual C++ 2013 under Windows.

# Table of Contents #

* [Requirements](#requirements)
* [Preparation](#preparation)
* [Compiling zlib](#compiling-zlib)
* [Compiling OpenSSL](#compiling-openssl)
* [Compiling Boost](#compiling-boost)
* [Compiling libtorrent](#compiling-libtorrent)
* [Compiling Qt4](#compiling-qt4)
* [Configuring Qt Creator](#configuring-qt-creator)
* [Compiling qBittorrent](#compiling-qbittorrent)

***

# Requirements #
* The Microsoft Visual C++ 2013 compiler.
* A [Perl language](http://www.perl.org/) interpreter. [Strawberry Perl](http://strawberryperl.com/) or [ActivePerl](http://www.activestate.com/activeperl/downloads) can be used. Download Strawberry Perl 5.20.1.1 [here](http://strawberryperl.com/download/5.20.1.1/strawberry-perl-5.20.1.1-32bit.msi).
* Latest release of the [NASM assembler](http://www.nasm.us/). Download version 2.11.05 [here](http://www.nasm.us/pub/nasm/releasebuilds/2.11.05/win32/nasm-2.11.05-win32.zip).
* Latest release of [zlib](http://www.zlib.net/). Download version 1.2.8 [here](http://zlib.net/zlib128.zip).
* Latest release of [OpenSSL](http://www.openssl.org/). Download version 1.0.1i [here](https://www.openssl.org/source/openssl-1.0.1i.tar.gz).
* Latest release of the [Boost libraries](http://www.boost.org/). Download version 1.56.0 [here](http://sourceforge.net/projects/boost/files/boost/1.56.0/boost_1_56_0.7z/download).
* Latest release of [libtorrent-rasterbar](http://www.libtorrent.org/). Download version 0.16.17 [here](http://sourceforge.net/projects/libtorrent/files/libtorrent/libtorrent-rasterbar-0.16.17.tar.gz/download).
* Latest release of the [Qt4 source](http://qt-project.org/). Download version 4.8.6 [here](http://download.qt-project.org/archive/qt/4.8/4.8.6/qt-everywhere-opensource-src-4.8.6.zip).
* Latest release of [Qt Creator](http://qt-project.org/wiki/category:Tools::QtCreator). Download version 3.2.1 [here](http://download.qt-project.org/official_releases/qtcreator/3.2/3.2.1/qt-creator-opensource-windows-x86-3.2.1.exe).
* Latest release of [qBittorrent](http://www.qbittorrent.org). Download version 3.1.10 [here](http://sourceforge.net/projects/qbittorrent/qbittorrent/qbittorrent-3.1.10/qbittorrent-3.1.10.tar.gz/download).

Let's assume that the working directory(ies) for our compilation will be:

* For **x86**: ```C:\qBittorrent```
* For **x64**: ```C:\qBittorrent64```

# Preparation #
* Install **Perl**.
* Install **Qt Creator**.
* Extract **NASM** inside our working dir, eg.:
	* For **x86**: ```C:\qBittorrent\nasm```

# Compiling zlib #
* Extract the **zlib** source inside our working dir(s).
* Rename the resulting dir(s) from ```zlib-x.x.x``` to just ```zlib```.
* Open the **MSVC2013 command shell**:
	* For **x86**: ```Start->Programs->Visual Studio 2013->Visual Studio Tools->VS2013 x86 Native Tools Command Prompt```
	* For **x64**: ```Start->Programs->Visual Studio 2013->Visual Studio Tools->VS2013 x64 Native Tools Command Prompt```
* Navigate to the **zlib** source folder:
	* For **x86**: ```pushd C:\qBittorrent\zlib```
	* For **x64**: ```pushd C:\qBittorrent64\zlib```
* Edit the ```win32\Makefile.msc``` file.
* Find the line starting with ```CFLAGS```
	* Replace ```-MD``` with ```-GL -MT -Zc:wchar_t-```
* Find the line starting with ```LDFLAGS```
	* Replace ```-debug``` with ```-opt:icf -dynamicbase -nxcompat -ltcg /nodefaultlib:msvcrt```
* Issue the following commands:

	* For **x86**:
        ```
        nmake -f win32/Makefile.msc LOC="-DASMV -DASMINF -DNDEBUG -I." OBJA="inffas32.obj match686.obj"
        ```

	* For **x64**:
        ```
        nmake -f win32/Makefile.msc AS=ml64 LOC="-DASMV -DASMINF -DNDEBUG -I." OBJA="inffasx64.obj gvmat64.obj inffas8664.obj"
        ```


* Copy **zlib.h**, **zconf.h**, **zlib.lib**, **zlib.pdb** to our install dir.

	* For **x86**:
        ```
        xcopy zlib.h C:\qBittorrent\install\include\
        xcopy zconf.h C:\qBittorrent\install\include\
        xcopy zlib.lib C:\qBittorrent\install\lib\
        xcopy zlib.pdb C:\qBittorrent\install\lib\
        ```

	* For **x64**:
        ```
        xcopy zlib.h C:\qBittorrent64\install\include\
        xcopy zconf.h C:\qBittorrent64\install\include\
        xcopy zlib.lib C:\qBittorrent64\install\lib\
        xcopy zlib.pdb C:\qBittorrent64\install\lib\
        ```

# Compiling OpenSSL #
* Make sure you have installed **Perl** and **NASM** (for **x86**).
* Extract the **OpenSSL** source inside our working dir(s).
* Rename the resulting dir(s) from ```openssl-x.x.x``` to just ```openssl```.
* Open the **MSVC2013 command shell**:
	* For **x86**: ```Start->Programs->Visual Studio 2013->Visual Studio Tools->VS2013 x86 Native Tools Command Prompt```
	* For **x64**: ```Start->Programs->Visual Studio 2013->Visual Studio Tools->VS2013 x64 Native Tools Command Prompt```
* Navigate to the **OpenSSL** source folder:
	* For **x86**: ```pushd C:\qBittorrent\openssl```
	* For **x64**: ```pushd C:\qBittorrent64\openssl```
* Now we will build a static version of **OpenSSL**. Issue the following commands:

	* For **x86**:
        ```
        set "PATH=%PATH%;C:\qBittorrent\nasm"
        perl Configure VC-WIN32 no-shared zlib no-zlib-dynamic threads --prefix=C:\qBittorrent\install -IC:\qBittorrent\install\include -LC:\qBittorrent\install\lib
        ms\do_nasm
        ```

	* For **x64**:
        ```
        perl Configure VC-WIN64A no-shared zlib no-zlib-dynamic threads --prefix=C:\qBittorrent64\install -IC:\qBittorrent64\install\include -LC:\qBittorrent64\install\lib
        ms\do_win64a.bat
        ```

* Edit the ```ms\nt.mak``` file:
	* Find the line that starts with: ```EX_LIBS```
    	* Replace ```zlib1.lib``` with ```zlib.lib```
	* Find the line that starts with: ```CFLAG```
    	* Append: ```/Zc:wchar_t- /GL /Zi```
	* Find the line that starts with: ```LFLAGS```
		* Replace ```/debug``` with ```/incremental:no /opt:icf /dynamicbase /nxcompat /ltcg /nodefaultlib:msvcrt```
	* Then issue the following:

        ```
        nmake -f ms\nt.mak
        nmake -f ms\nt.mak install
        ```

# Compiling Boost #
* Extract the **Boost** sources in the working dir(s).
* Rename the resulting dir(s) from ```boost_x_x_x``` to just ```boost```.
* Open the **MSVC2013 command shell**:
	* For **x86**: ```Start->Programs->Visual Studio 2013->Visual Studio Tools->VS2013 x86 Native Tools Command Prompt```
	* For **x64**: ```Start->Programs->Visual Studio 2013->Visual Studio Tools->VS2013 x64 Native Tools Command Prompt```
* Navigate to the **Boost** source folder:
	* For **x86**: ```pushd C:\qBittorrent\boost```
	* For **x64**: ```pushd C:\qBittorrent64\boost```

* Now you will need to bootstrap **Boost** so it will build ```b2.exe```. Issue the following command: ```bootstrap.bat vc12```
* Compile a static version of **Boost**. Issue the following command (if you want to set how many threads b2 uses when compiling, add ```-j N``` to the end of the command, where ```N``` is the number of threads; the ```toolset``` parameter is needed for the correct choice of ```msvc-12.0``` if other versions are installed):

	* For **x86**:
        ```
        b2 -q --with-system --toolset=msvc-12.0 variant=release link=static runtime-link=static debug-symbols=off warnings=off warnings-as-errors=off
        ```

	* For **x64**:
        ```
        b2 -q --with-system --toolset=msvc-12.0 variant=release link=static runtime-link=static debug-symbols=off warnings=off warnings-as-errors=off architecture=x86 address-model=64
        ```

# Compiling libtorrent #
* Extract the **libtorrent** sources in the working dir(s).
* Rename the resulting dir(s) from ```libtorrent-rasterbar-x.x.x``` to just ```libtorrent```.
* Open the **MSVC2013 command shell**:
	* For **x86**: ```Start->Programs->Visual Studio 2013->Visual Studio Tools->VS2013 x86 Native Tools Command Prompt```
	* For **x64**: ```Start->Programs->Visual Studio 2013->Visual Studio Tools->VS2013 x64 Native Tools Command Prompt```
* Navigate to the **libtorrent** source folder:
	* For **x86**: ```pushd C:\qBittorrent\libtorrent```
	* For **x64**: ```pushd C:\qBittorrent64\libtorrent```
* Compile a static version of **libtorrent**. Issue the following command (if you want to set how many threads b2 uses when compiling, add ```-j N``` to the end of the command, where ```N``` is the number of threads):
	* For **x86**:
        ```
        ..\boost\b2 -q --prefix="stage" --without-python --toolset=msvc-12.0 variant=release link=static runtime-link=static encryption=openssl logging=none geoip=static dht-support=on boost=source character-set=unicode boost-link=static -sBOOST_ROOT="C:\qBittorrent\boost" include="C:\qBittorrent\install\include" library-path="C:\qBittorrent\install\lib" install
        ```

	* For **x64**:
        ```
        ..\boost\b2 -q --prefix="stage" --without-python --toolset=msvc-12.0 variant=release link=static runtime-link=static encryption=openssl logging=none geoip=static dht-support=on boost=source character-set=unicode boost-link=static -sBOOST_ROOT="C:\qBittorrent64\boost" include="C:\qBittorrent64\install\include" library-path="C:\qBittorrent64\install\lib" architecture=x86 address-model=64 install
        ```

	* Note: When you use **libtorrent-rasterbar-1.0.4**: 
		change ```dht-support=on``` to ```dht=on```
		see [libtorrent manual](http://www.libtorrent.org/building.html)



# Compiling Qt4 #
* Extract the **Qt4** sources in the working dir(s).
* Rename the resulting dir(s) from ```qt-everywhere-opensource-src-x.x.x``` to just ```qt4```.
* Open the **MSVC2013 command shell**:
	* For **x86**: ```Start->Programs->Visual Studio 2013->Visual Studio Tools->VS2013 x86 Native Tools Command Prompt```
	* For **x64**: ```Start->Programs->Visual Studio 2013->Visual Studio Tools->VS2013 x64 Native Tools Command Prompt```
* Navigate to the Qt4 source folder:
	* For **x86**: ```pushd C:\qBittorrent\qt4```
	* For **x64**: ```pushd C:\qBittorrent64\qt4```
* Now we will build a static version of **Qt4** with making as small as possible.
* Open ```mkspecs\win32-msvc2013\qmake.conf``` and replace the
	* Find the line that starts with: ```QMAKE_CFLAGS_RELEASE```
		* Replace ```-MD``` with ```-GL -MT```
	* Find the line that starts with: ```QMAKE_LFLAGS```
		* Replace ```/NXCOMPAT``` with ```/NXCOMPAT /LTCG```
	* Find the line that starts with: ```QMAKE_LFLAGS_RELEASE```
		* Replace ```/INCREMENTAL:NO``` with ```/INCREMENTAL:NO /NODEFAULTLIB:MSVCRT```
* Open ```src\3rdparty\zlib_dependency.pri``` **and** ```src\tools\bootstrap\bootstrap.pri```
	* Replace ```zdll.lib``` with ```zlib.lib```
* Issue the following commands:
	* For **x86**:
        ```
        configure.exe -release -opensource -confirm-license -static -ltcg -fast -system-zlib -no-qt3support -no-opengl -no-openvg -no-dsp -no-vcproj -no-dbus -no-phonon -no-phonon-backend -no-multimedia -no-audio-backend -no-webkit -no-script -no-scripttools -no-declarative -no-declarative-debug -mp -arch windows -qt-style-windowsxp -nomake examples -nomake demos -platform win32-msvc2013 -openssl-linked -largefile -I "C:\qBittorrent\install\include" -L "C:\qBittorrent\install\lib"
        bin\qmake.exe projects.pro QT_BUILD_PARTS="libs translations"
        nmake
        ```

	* For **x64**:
        ```
        configure.exe -release -opensource  -confirm-license -static -ltcg -fast -system-zlib -no-qt3support -no-opengl -no-openvg -no-dsp -no-vcproj -no-dbus -no-phonon -no-phonon-backend -no-multimedia -no-audio-backend -no-webkit -no-script -no-scripttools -no-declarative -no-declarative-debug -mp -arch windows -qt-style-windowsxp -nomake examples -nomake demos -platform win32-msvc2013 -openssl-linked -largefile -I "C:\qBittorrent64\install\include" -L "C:\qBittorrent64\install\lib"
        bin\qmake.exe projects.pro QT_BUILD_PARTS="libs translations"
        nmake
        ```

* You can close the command prompt now.

# Configuring Qt Creator #
* Launch **Qt Creator** and select ```Tools->Options...```
* Select the ```Build & Run``` item from the left and select the ```Qt Versions``` tab.
	* For **x86**:
		* Click the ```Add...``` button and select ```qmake.exe```. It should be in: ```C:\qBittorrent\qt4\bin\qmake.exe```
		* Name it something meaningful like ```Qt4 - MSVC2013 (x86)```
	* For **x64**:
		* Click the ```Add...``` button and select ```qmake.exe```. It should be in: ```C:\qBittorrent64\qt4\bin\qmake.exe```
		* Name it something meaningful like ```Qt4 - MSVC2013 (x64)```
* Still in the ```Build & Run``` item, select the ```Kits``` tab.
	* For **x86**:
		* Click the ```Add...``` button and name it ```x86```
		* In ```Compiler``` select ```Microsoft Visual C++ Compiler 12.0 (x86)```
		* In ```Qt version``` select ```Qt4 - MSVC2013 (x86)```
	* For **x64**:
		* Click the ```Add...``` button and name it ```x64```
		* In ```Compiler``` select ```Microsoft Visual C++ Compiler 12.0 (amd64)```
		* In ```Qt version``` select ```Qt4 - MSVC2013 (x64)```

# Compiling qBittorrent #
* Extract the **qBittorrent** sources in the working dir(s).
* Go to [Maxmind](http://www.maxmind.com) and download the [GeoLite Country database](http://dev.maxmind.com/geoip/legacy/geolite/) in binary format. Here is the [link](http://geolite.maxmind.com/download/geoip/database/GeoLiteCountry/GeoIP.dat.gz). Extract the ```GeoIP.dat``` file to: ```src\gui\geoip```
* Open the ```winconf.pri``` file, edit and adjust the paths:
	* For **both**:
    	* Replace ```_WIN32_WINNT=0x0500``` with ```_WIN32_WINNT=0x0501```
	* For **x86**:
    	* Replace ```C:/qBittorrent/boost_1_51_0``` with ```C:/qBittorrent/boost```
    	* Replace ```C:/qBittorrent/RC_0_16/include``` with ```C:/qBittorrent/libtorrent/include```
    	* Replace ```C:/qBittorrent/Zlib/include``` with ```C:/qBittorrent/install/include```
    	* Replace ```C:/qBittorrent/boost_1_51_0/stage/lib``` with ```C:/qBittorrent/boost/stage/lib```
    	* Replace ```C:/qBittorrent/RC_0_16/bin/<path-according-to-the-build-options-chosen>``` with ```C:/qBittorrent/libtorrent/stage/lib```
    	* Replace ```C:/qBittorrent/Zlib/lib``` with ```C:/qBittorrent/install/lib```
	* For **x64**:
    	* Replace ```C:/qBittorrent/boost_1_51_0``` with ```C:/qBittorrent64/boost```
    	* Replace ```C:/qBittorrent/RC_0_16/include``` with ```C:/qBittorrent64/libtorrent/include```
    	* Replace ```C:/qBittorrent/Zlib/include``` with ```C:/qBittorrent64/install/include```
    	* Replace ```C:/qBittorrent/boost_1_51_0/stage/lib``` with ```C:/qBittorrent64/boost/stage/lib```
    	* Replace ```C:/qBittorrent/RC_0_16/bin/<path-according-to-the-build-options-chosen>``` with ```C:/qBittorrent64/libtorrent/stage/lib```
    	* Replace ```C:/qBittorrent/Zlib/lib``` with ```C:/qBittorrent64/install/lib```

* Open the ```winconf-msvc.pri``` file and adjust the filename of the lib of Boost:
	* Check ```C:\qBittorrent\boost\stage\lib``` or ```C:\qBittorrent64\boost\stage\lib``` for the ```.lib``` name, it should be named in this format: ```libboost_system-vcx-mt-s-x_x.lib```, eg: ```libboost_system-vc120-mt-s-1_56.lib```
	* Replace ```libboost_system-vc90-mt-s-1_51.lib``` with the ```.lib``` name.

* Launch **Qt Creator** and open the ```qbittorrent.pro``` file.
	* For **x86**: check the ```x86``` checkbox and click ```Configure Project```
	* For **x64**: check the ```x64``` checkbox and click ```Configure Project```

* Change the configuration from ```Debug``` to ```Release``` in the left side of the window (where the computer icon is).

* Select ```Build->Build All```

* After the compilation ends you should have ```qbittorrent.exe``` in:
	* For **x86**: ```C:\qBittorrent\build-qbittorrent-x86-Release\src\release```
	* For **x64**: ```C:\qBittorrent64\build-qbittorrent-x64-Release\src\release```