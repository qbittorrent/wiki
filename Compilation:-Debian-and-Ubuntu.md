This how-to will guide you through compiling both qBittorrent and optionally its backend, `libtorrent-rasterbar` (referred to only as `libtorrent` for the rest of the document) from source.

Only follow this guide if you know what you are doing and this is what you really want.
If you just want the latest version of qBittorrent, use our [stable](https://launchpad.net/~qbittorrent-team/+archive/ubuntu/) or [unstable](https://launchpad.net/~qbittorrent-team/+archive/ubuntu/qbittorrent-unstable) PPAs.

This guide is written for Debian/Ubuntu, but the process should be the same or similar for other Debian-derivative distributions.

If you run into trouble or errors at any step, check the Troubleshooting section at the bottom of the page first before posting an issue report.

# Dependencies

## Boost and other build dependencies

You need to install these packages in order to be able to compile qBittorrent from source.

```bash
sudo apt install build-essential pkg-config automake libtool git zlib1g-dev libssl-dev libgeoip-dev
sudo apt install libboost-dev libboost-system-dev libboost-chrono-dev libboost-random-dev
```

If you want, you can install the Boost libraries from source instead of installing them from the distro's repositories. This is quite easy to do, but generally does not make that much of a difference and is out of the scope of this guide.

## Runtime-only dependencies

qBittorrent has a runtime-only dependency on Python 3 for its search engine functionality. Old enough versions also supported Python 2, but this has long since been deprecated.

```bash
sudo apt install python3
```

## Qt libraries

qBittorrent uses the Qt framework as the basis for its GUI.

qBittorrent 4.0 - 4.1.x requires at least Qt 5.5.1, and qBittorrent 4.2 and later requires at least Qt 5.9.

For Debian 10, Ubuntu 18.04 LTS or later, just install Qt from the official repositories:

```bash
sudo apt install qtbase5-dev qttools5-dev libqt5svg5-dev
```

Note that Qt libraries in Debian 8/9 repository are too old for compiling newer qBittorrent versions, so you need to install newer Qt libraries some other way.

If you are using Ubuntu 16.04 LTS, you could add [this PPA](https://launchpad.net/~beineri/+archive/ubuntu/opt-qt597-xenial) for Qt 5.9 support:

```bash
sudo add-apt-repository ppa:beineri/opt-qt597-xenial
sudo apt update
sudo apt install qt59base qt59svg qt59tools
export PATH=/opt/qt59/bin:$PATH
export PKG_CONFIG_PATH=/opt/qt59/lib/pkgconfig:$PKG_CONFIG_PATH
```

Alternatively, you could of course compile Qt version of your choosing from source. However, only do this if you have a _very_ good reason to do so.

## libtorrent

qBittorrent uses the [libtorrent](https://libtorrent.org/) library by Arvid Norberg as the backend.

It is necessary to compile and install `libtorrent` before compiling qBittorrent.

It is possible to install the `libtorrent` development package directly from the distro's repositories:

```bash
sudo apt install libtorrent-rasterbar-dev
```

but the version may not be the most recent or exactly the one you want.

Alternatively, you can compile it yourself.

First, clone the `libtorrent` repository:

```bash
git clone https://github.com/arvidn/libtorrent.git
cd libtorrent
```

To compile, first choose the appropriate `git` and compile commands in the table below, according to the version of `libtorrent` you need, then run them:

<table>
<thead>
<tr>
<th><p>libtorrent version series</p></th>
<th><p>qBittorrent version support</p></th>
<th><p>git command (example with most recent tag in series at the time of writing)</p></th>
<th colspan="2"><p>Compile commands - after running the git command, choose between using CMake (recommended) <em>or</em> autotools</p></th>
</tr>
</thead>
<tbody>
<tr>
<td colspan="3"><hr /></td>
<td><p>CMake</p></td>
<td><p>autotools</p></td>
</tr>
<tr>
<td><p>1.2.x</p></td>
<td><p>&gt;= 4.2.0</p></td>
<td><pre><code>git checkout libtorrent-1_2_5</code></pre></td>
<td><pre><code>cmake -B cmake-build-dir/release -DCMAKE_BUILD_TYPE=Release -DCMAKE_CXX_STANDARD=14
cmake --build cmake-build-dir/release</code></pre></td>
<td><pre><code>./autotool.sh
./configure --disable-debug --enable-encryption CXXFLAGS=&quot;-std=c++14&quot;
make clean && make -j$(nproc)</code></pre></td>
</tr>
<tr>
<td><p>1.1.x</p></td>
<td><p>&gt;= 3.3.8 and &lt;4.2.0 (*)</p></td>
<td><pre><code>git checkout libtorrent-1_1_14</code></pre></td>
<td rowspan="2"><pre><code>cmake -B cmake-build-dir/release -DCMAKE_BUILD_TYPE=Release
cmake --build cmake-build-dir/release</code></pre></td>
<td rowspan="2"><pre><code>./autotool.sh
./configure --disable-debug --enable-encryption
make clean && make -j$(nproc)</code></pre></td>
</tr>
<tr>
<td><p>1.0.x</p></td>
<td><p>&lt;= 4.1.5</p></td>
<td><pre><code>git checkout libtorrent-1_0_11</code></pre></td>
</tr>
</tbody>
</table>

(*) Technically, the 4.2.x releases actually support the 1.1.x `libtorrent` series, but it is just life support and not properly tested/developed.

Finally, you can install `libtorrent`.

- If building with CMake:

    `sudo cmake --install cmake-build-dir/release`

    This generates an `install_manifest.txt` file in the build folder that can later be used to uninstall all installed files with `sudo xargs rm < install_manifest.txt`. The default installation prefix is `/usr/local`, as expected.

- If building with `autotools`:

    If you have `checkinstall`, the following command will generate and install a `.deb` package that can be tracked and managed by your package manager:

    ```bash
    sudo checkinstall --nodoc --backup=no --deldesc --pkgname libtorrent-rasterbar --pkgversion 1.x.x-source-compile # change the version to your liking
    ```

    Alternatively, the traditional way will do just fine (but there is no tracking of the installed files):

    ```bash
    sudo make install
    ```

For more information on building libtorrent, see [libtorrent downloading and building](https://www.libtorrent.org/building.html).

# Compiling qBittorrent (with the GUI)

First, obtain the qBittorrent source code.

Either download and extract a `.tar` archive from [the GitHub releases page](https://github.com/qbittorrent/qBittorrent/releases) or clone the `git` repository:

```bash
git clone https://github.com/qbittorrent/qBittorrent
```

If you clone the git repository, you can use the `master` branch, which contains the current development version, or checkout a specific release tag:

```bash
git checkout release-4.2.5
```

Open the folder in a terminal window and run this to configure the build:

```bash
./configure CXXFLAGS="-std=c++14"
```

Once that finishes successfully, run:

```bash
 make -j$(nproc)
```

Finally, you can install qBittorrent:

If you have `checkinstall`, the following command will generate and install a `.deb` package that can be tracked and managed by your package manager:

```bash
sudo checkinstall --nodoc --backup=no --deldesc --pkgname qbittorrent --pkgversion 4.x.x-source-compile # change the version to your liking
```

Alternatively, the traditional way will do just fine:

```bash
sudo make install
```

That's it! qBittorrent is now installed. You should now be able to run it from the terminal or the installed shortcuts.

# Compiling qBittorrent (without the GUI; aka qBittorrent-nox aka headless)

The steps are almost all the same as the GUI version except for the configure step, where you should add the `--disable-gui` flag to it, like so:

```bash
./configure CXXFLAGS="-std=c++14" --disable-gui
```

It is also recommended that you name the package `qbittorrent-nox` instead of `qbittorrent` if you use the checkinstall` method for installing.

That's it! qBittorrent headless is now installed. You should now be able to run it from the terminal with `qbittorrent-nox`.

Since you've disabled the graphical user interface, qBittorrent can only be controlled via its WebUI. By default, it can be accessed at `http://localhost:8080` with the following credentials, after accepting the legal disclaimer that's presented the first time you run the program:

```txt
Username: admin
Password: adminadmin
```

Documentation about running qBittorrent without GUI is available [here](https://github.com/qbittorrent/qBittorrent/wiki/Running-qBittorrent-without-X-server-(WebUI-only)).

# Troubleshooting

## Compiling (generic)

* In the `make` command, the `-j$(nproc)` flag makes the number of build jobs equal to the number of hardware threads available. To see the actual value your system is using, run `echo $(nproc)` in a terminal. You could also manually specify a value like so: `-j5`. Higher values may make the build faster, but an eye must be kept on the memory usage.

## Compiling `libtorrent`

* If you get a `configure: error: Boost.System library not found`, check if you installed all the above dependencies.

    If so, also pass the `--with-boost-libdir=/usr/lib/i386-linux-gnu` to the `./configure`

## Compiling qBittorrent

* If you're using `libtorrent-rasterbar` from the 0.16.x series, you also need to pass the `--with-libtorrent-rasterbar0.16` option to configure. qBittorrent v3.3.x has dropped the support of libtorrent 0.16.x.
* If you want to compile with Qt4 instead of qt5, you also need to pass the `--with-qt4` option to configure. qBittorrent v4.0.x has dropped support for Qt4

## Running qBittorrent

If you get an error similar to `qbittorrent: symbol lookup error: qbittorrent: undefined symbol:`
Simply run:
 ldconfig
The following method is too complex, but if you want to know what's going on, then you can read the following method (see https://github.com/qbittorrent/qBittorrent/issues/8047):

Check if your `LD_LIBRARY_PATH` environment variable is set and the path `/usr/local/lib` is included.

Simply run `env` in your terminal and look for `LD_LIBRARY_PATH`.
If so, you are good to go. If not, add the path to the variable:

```bash
export LD_LIBRARY_PATH=/usr/local/lib:${LD_LIBRARY_PATH}
```

## Notes

* If you experience any problem with this how to, do not hesitate to contact sledgehammer999(at)qbittorrent(dot)org.
