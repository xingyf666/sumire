# OpenImage

## 安装流程

需要先获得依赖库，配置库路径。



### Boost

首先是 BOOST 库。创建 BOOST_ROOT 路径

```shell
wget https://boostorg.jfrog.io/artifactory/main/release/1.82.0/source/boost_1_82_0.tar.gz
cd {BOOST_ROOT}
./bootstrap.sh
./b2
```

如果出现找不到 pyconfig.h 的错误，需要安装 python2.7 版本

```shell
sudo apt-get install python-dev
```

如果需要安装到系统目录下，执行

```shell
sudo ./b2 install
```

默认的头文件在 /usr/local/include/boost 目录下，库文件在 /usr/local/lib/ 目录下。当其它库链接 boost 生成动态库时，需要添加 -fPIC 标记，否则无法定位到函数

```shell
sudo ./b2 cxxflags=-fPIC cflags=-fPIC install
```



### zlib

去掉 CMAKE_INSTALL_PREFIX 来安装到系统中会更好

```shell
git clone https://github.com/madler/zlib.git
cd {ZLIB_ROOT}
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=.
cmake --build build --config Release --target install
```



### libTIFF

去掉 CMAKE_INSTALL_PREFIX 来安装到系统中会更好

```shell
git clone https://gitlab.com/libtiff/libtiff.git
cd {TIFF_ROOT}
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF -DCMAKE_INSTALL_PREFIX=.
cmake --build build --target install
```



### libjpeg-turbo

去掉 `CMAKE_INSTALL_PREFIX` 来安装到系统中会更好

```shell
git clone https://github.com/libjpeg-turbo/libjpeg-turbo.git
cd {JPEG_ROOT}
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release -DENABLE_SHARED=OFF -DCMAKE_INSTALL_PREFIX=.
cmake --build build --config Release --target install
```

链接生成动态库时，需要 -fPIC 标记，因此找到源码根目录下的 CMakelists 文件，添加

```cmake
add_compile_options(-fPIC)
```

由于之后 OpenImage 的编译使用的是相对路径，因此最好安装两份，一份不添加 `CMAKE_INSTALL_PREFIX` 选项，一份添加选项。

> > 编译 libpng 时也需要添加 `-fPIC` 标记。



### OpenEXR

去掉 CMAKE_INSTALL_PREFIX 来安装到系统中会更好

```shell
git clone https://github.com/AcademySoftwareFoundation/openexr.git
cd {EXR_ROOT}
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=dist -DBUILD_TESTING=OFF -DBUILD_SHARED_LIBS=OFF -DOPENEXR_BUILD_TOOLS=OFF -DOPENEXR_INSTALL_TOOLS=OFF -DOPENEXR_INSTALL_EXAMPLES=OFF -DZLIB_ROOT={ZLIB_ROOT}
sudo cmake --build build --target install --config Release
```



### pybind11

通过 `PYBIND11_PYTHON_VERSION` 变量指定匹配的 python 版本

```shell
git clone https://github.com/pybind/pybind11.git
cd {PYBIND_ROOT}
sudo cmake -S . -B build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=. -DBoost_INCLUDE_DIR=/usr/local/include/boost -DPYBIND11_PYTHON_VERSION=3.10
sudo cmake --build build --config Release --target install
```

安装过程中可能会提示需要 pip 安装 pytest，此时创建并进入虚拟环境，然后安装

```shell
pip install pytest
```

然后可以正常编译安装。



### lzma

安装 OpenImageIO 时可能会报错 undefined reference to `lzma_end`，这就需要安装 lzma 库。

```shell
git clone https://github.com/tukaani-project/xz
cd xz
cmake -B build
cmake --build build --config Release --target install
sudo cp /usr/local/lib/liblzma.a /usr/lib/
```



### OpenImageIO

前面的库安装在系统目录下后可以避免一些编译问题，例如 lzma 出现未定义的引用。

```shell
git clone https://github.com/AcademySoftwareFoundation/OpenImageIO.git
cd {OIIO_ROOT}
cmake -S . -B build -DVERBOSE=ON -DCMAKE_BUILD_TYPE=Release -DBoost_USE_STATIC_LIBS=ON -DBoost_NO_WARN_NEW_VERSIONS=ON -DBoost_ROOT=../boost_1_82_0/ -DZLIB_ROOT=../zlib/ -DTIFF_ROOT=../libtiff/ -DOpenEXR_ROOT=../openexr/dist -DImath_DIR=../openexr/dist/lib/cmake/Imath/ -DJPEG_ROOT=../libjpeg-turbo/ -Dpybind11_ROOT=../pybind11/ -DUSE_PYTHON=1 -DUSE_QT=0 -DBUILD_SHARED_LIBS=0 -DLINKSTATIC=1
cmake --build build --config Release --target install
```

> > 要确保 python 版本正确，最好在指定版本的虚拟环境下进行编译。



