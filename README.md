# QttpServer

<b>QttpServer</b> is a fork from [node.native](https://github.com/d5/node.native) with some additional contributions from [tojocky](https://github.com/tojocky/node.native).  Intended as an alternative to [QHttpServer](https://github.com/nikhilm/qhttpserver), this is ideal for those who want the benefits of libuv with basic Qt libraries.

## Prerequisites

1. [git](http://git-scm.com/)
2. [python 2.x](https://www.python.org/)
3. [qt installer](http://www.qt.io/download/) or [build](http://doc.qt.io/qt-5/linux-building.html) from [source](http://download.qt.io/official_releases/qt/5.5/5.5.1/single/)

   ```bash
   # Building from source is a breeze - works well for Ubuntu 12 & 14 LTS
   sudo apt-get install libssl-dev
   wget http://download.qt.io/official_releases/qt/5.5/5.5.1/single/qt-everywhere-opensource-src-5.5.1.tar.gz
   tar -xvf qt-everywhere-opensource-src-5.5.1.tar.tz
   cd qt-everywhere-opensource-src-5.5.1
   ./configure -nomake examples -opensource -skip qtwebkit -skip qtwebchannel -skip qtquick1 -skip qtdeclarative -openssl-linked
   make
   sudo make install
   ```
4. perl

## Build (*nix only)

```bash
git clone https://github.com/supamii/QttpServer.git
cd QttpServer
```

Git submodules/dependencies automatically pulls in mongodb-drivers, boost, libuv, http-parser
```bash
git submodule update --init
```

Generate build files and compile to ./out/ folder
```bash
./build.py
make -C out
```

Generate makefile and compile to ./out/qtdebug/ or ./out/qtrelease/ or launch `qttp.pro` with Qt Creator
```bash
qmake CONFIG+=debug qttp.pro
make
```

Run the executable!
```bash
ln -s out/qtdebug/qttp startQttp
./startQttp &
```

## Examples

Example 1: Using a raw std::function based callback
```c++
#include <native.h>
#include <QCoreApplication>
#include <QtCore>
#include <thread>
#include "httpserver.h"

using namespace std;
using namespace qttp;
using namespace native::http;

int main(int argc, char** argv)
{
  QCoreApplication app(argc, argv);

  // Always initialize in the main thread.
  HttpServer* httpSvr = HttpServer::getInstance();

  httpSvr->addAction("test", [](HttpData& data) {
    QJsonObject& json = data.getJson();
    json["response"] = "Test C++ FTW";
  });

  // Bind routes and actions together.
  httpSvr->registerRoute("test", "/test");
  httpSvr->registerRoute("test", "/test2");

  thread webSvr(HttpServer::start);
  webSvr.detach();

  auto result = app.exec();
  return result;
}
```

Example 2: Using the action interface
```c++
// Adds the action interface via template method.
httpSvr->addAction<Sample>();

httpSvr->registerRoute("sample", "/sample");
httpSvr->registerRoute("sample", "/sampleAgain");

class Sample : public Action {
  void onAction(HttpData& data) {
    QJsonObject& json = data.getJson();
    json["response"] = "Sample C++ FTW";
  }
  std::string getActionName() { return "sample"; }
};
```

## Optional components
##### Build Redis client

1. `cd lib/qredisclient` and execute `git submodule update --init`
2. `qmake CONFIG+=debug DESTDIR=. qredisclient.pro`
3. `make`


##### Build MongoDb driver

1. Install [scons](http://www.scons.org/) - e.g. `brew install scons`
2. Install [boost](https://github.com/mongodb/mongo-cxx-driver/wiki/Download-and-Compile-the-Legacy-Driver)
   e.g. `brew search boost` and `brew install homebrew/versions/boost155` 
   Generally recommend using brew, apt-get, or the pre-built [binary installer for windows](http://sourceforge.net/projects/boost/files/boost-binaries/)

    ```bash
    # Or skip the whole build, just install what you can with ubuntu
    sudo apt-get install mongodb-dev
    sudo apt-get install libboost1.54-all-dev
    ```
4. Build the driver!

   Ubuntu

    ```bash
    cd QttpServer/lib/mongo-cxx-driver
    scons --prefix="/usr/local/opt/mongo-client" --libpath=/usr/local/opt/boost155/lib --cpppath=/usr/local/opt/boost155/include
    ```

   Windows

    ```batch
    set PATH=C:\Python27;C:Pyathon27\Scripts;%PATH%
    scons --32 --dbg=on --opt=off --sharedclient --dynamic-windows --prefix="C:\local\mongo-client" --cpppath="C:\local\boost_1_59_0" --libpath="C:\local\boost_1_59_0\lib32-msvc-14.0" install
    ```

For more information visit [mongodb.org](https://docs.mongodb.org/getting-started/cpp/client/)


## Windows 8+ Build - MSVC 2015 only

The MSVC 2012 and 2013 compilers don't support C++1y well enough so QttpServer is limited to  Windows 8+ with Visual Studio 2015.

Compounded by the fact that Qt has yet to explicity support MSVC 2015, there are a considerable amount of steps to complete.  We'll first need to install Qt with MSVC 2013 for QtCreator, download sources, and finally build Qt5 against MSVC 2015.  The guidwas been developed and tested using Windows 10.

#### Prerequisites
1. Visual Studio 2013 express (MSVC 2013 tool-chain) - C++ tools must be activated
2. Visual Studio 2015 community (MSVC 2015 tool-chain
3. Qt 5.x (Initially configure to match with MSVC 2013)
4. [git](http://git-scm.com/)
5. [python 2.x](https://www.python.org/)
6. [strawberry perl](http://strawberryperl.com/) - As a precaution

#### Building Qt from source
1. [Building](http://doc.qt.io/qt-5/windows-building.html) from [zip file](http://download.qt.io/official_releases/qt/5.5/5.5.0/single/) - I didn't bother with Qt's Git components - [Stackoverflow](http://stackoverflow.com/questions/32894097/how-do-i-use-qt-in-my-visual-studio-2015-projects) always helps
2. Extract to `C:\qt-5.5.0` - beware, zip is massive so don't use default windows extractor.  I used cygwin's unzip or jar to extract contents
3. Create file `c:\qt-5.5.0\qt5vars.cmd` with the contents below:

    ```batch
    REM Set up \Microsoft Visual Studio 2013, where <arch> is \c amd64, \c x86, etc.
    CALL "C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\vcvarsall.bat"
    SET _ROOT=C:\qt-5.5.0
    SET PATH=%_ROOT%\qtbase\bin;%_ROOT%\gnuwin32\bin;C:\Python27;C:\Strawberry\perl\bin;%PATH%
    REM Uncomment the below line when using a git checkout of the source repository
    REM SET PATH=%_ROOT%\qtrepotools\bin;%PATH%
    SET QMAKESPEC=win32-msvc2015
    SET _ROOT=
    ```
   I didn't specify an arch value in qt5vars.cmd since MSVC2015 is x86 anyway (Strange, I know).  Also make sure to add python and perl to the PATH.  Lastly and most importantly - be sure to update QMAKESPEC to `win32-msvc2015`
4. Launch MSVC 2015 developer console, `cd c:\qt-5.5.0\` and execute `qt5vars.cmd` to load environment variables
5. In the same MSVC developer console, execute the configuration script.  Below are configuration options that worked well on my machine against MSVC 2015:

   To include [OpenSSL](https://code.google.com/p/openssl-for-windows/downloads/list), download and extract to `C:\openssl-0.9.8k_WIN32`.  Also add the lib and bin paths to the global %PATH% environment variable or equivalent.  This is **recommended** in general and also helps support the qredis library.  Tip: If you get stuck on qtwebsockets and don't intend on using it, you may also pass in "-skip qtwebsockets"
    ```batch 
    configure.bat -platform win32-msvc2015 -debug -nomake examples -opensource -skip qtwebkit -skip qtwebchannel -skip qtquick1 -skip qtdeclarative -openssl-linked OPENSSL_LIBS="-lssleay32 -llibeay32" -I C:\openssl-0.9.8k_WIN32\include -L C:\openssl-0.9.8k_WIN32\lib -l Gdi32 -l User32
    ```

    If you really don't want OpenSSL support just run this:
    ```batch
    configure.bat -platform win32-msvc2015 -debug -nomake examples -opensource -skip qtwebkit -skip qtwebchannel -skip qtquick1 -skip qtdeclarative
    ```
6. Build with `nmake` or `jom`
7. Add the MSVC 2015 tool-chain to QtCreator:

   * Create an empty project and load it in QtCreator
   * Go to the `Projects > Manage Kits.. > Build & Run > Qt Versions > Add...` and select C:\qt-5.5.0\qtbase\bin\qmake and include somewhere in the label MSVC2015.
   * Then within `Projects > Manage Kits.. > Build & Run > Kits > Add` a new kit named "Qt 5.5.0 MSVC2015 32bit", choose the compiler that matches MSVC 2015 (Compiler v14), and lastly choose the Qt version that we recently created.

#### Finally building QttpServer

```bash
git clone https://github.com/supamii/QttpServer
cd QttpServer
```

Git submodules/dependencies automatically pulls in mongodb-drivers, boost, libuv, http-parser
```bash
git submodule update --init
```

Generate build files and compile to ./out/ folder
```bash
# Double-click on build.py
./build.py
```

Open solution build/all.sln with VS2015

Individually configure each project to use MSVC 2015 tool-chain and **BUILD individually**

1. `http_parser`
2. `libuv`
3. `node_native`
4. **SKIP all other projects not listed**

Copy lib and pdb files to /out/ folder
```bash
# Under the build directory, double-click
./postbuild.sh
```

Launch `qttp.pro`, build, run with Qt Creator and...

**ENJOY a beer - you've earned it!**

# TODOs

1. Address subtle techdebt surrounding references with native::http components
2. Create default preprocessors for meta data for each incomming request guid generation
3. Config parsing is still incomplete - action-routes should be configurable instead of being set in code
4. Determine versioning support in the path e.g. /v1/ /v2/
5. Clean up configuration deployment (make install files to the correct folder)
6. Setup utilities for MongoDB and Redis access
7. ~~Add pre and post processor callbacks as an alternative to the interface class~~
8. Make available a metrics pre/post processor
9. Design an error response mechanism

