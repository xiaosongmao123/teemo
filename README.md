[![Build Status](https://travis-ci.com/winsoft666/teemo.svg?branch=master)](https://travis-ci.com/winsoft666/teemo) 
[![Vcpkg package](https://img.shields.io/badge/Vcpkg-package-blueviolet)](https://github.com/microsoft/vcpkg/tree/master/ports/teemo)
[![badge](https://img.shields.io/badge/license-GUN-blue)](https://github.com/winsoft666/teemo/blob/master/LICENSE)

[ >>> 中文版](README_ch.md)

# Introduction
Although there are many mature and powerful download tools at present, such as `Free Download Manager`, `Aria2`, etc. However when I want to find a library that support multiple protocols (such as http, ftp), multi-threaded download, breakpoint resume download, cross-platform, I realize that this is difficult to find a satisfactory library, especially developed by C++. 

So I developed this download library named `"teemo"` based on libcurl, which can support the following features:

1. Support Multi-protocol. Since it is based on libcurl, so it supports all protocols that supported by libcurl, such as http, https, ftp, etc.

2. Support multi-threaded download.

3. Support breakpoint resume.

4. Support for obtaining real-time download rate.

5. Support download speed limit.

---

# Dependencies
I prefer to use `vcpkg` to install dependencies, of course, this is not the only way, you can install dependencies through any ways.

- libcurl

    ```bash
    # if you want support non-http protocol, such as ftp, the [non-http] option must be specified.
    vcpkg install curl[non-http]:x86-windows
    ```
- cpprestsdk

`Teemo` depend on pplx that give you access to the Concurrency Runtime, a concurrent programming framework for C++, pplx is a part of `cpprestsdk` library.

    ```bash
    vcpkg install cpprestsdk:x86-windows
    ```

- gtest

unit test project depend on gtest.

    ```bash
    vcpkg install gtest:x86-windows
    ```

# Build
Firstly using CMake to generate project or makefile, then comiple.

```bash
# Windows Sample
cmake.exe -G "Visual Studio 15 2017" -DBUILD_SHARED_LIBS=ON -DBUILD_TESTS=ON -S %~dp0 -B %~dp0build

# Linux Sample
cmake -DBUILD_SHARED_LIBS=ON -DBUILD_TESTS=ON
make
make install
```

---

# Getting Started
```c++
#include <iostream>
#include "teemo.h"

using namespace teemo;

int main(int argc, char **argv) {
    Teemo::GlobalInit();

    Teemo efd;
    efd.Start(u8"http://xxx.xxx.com/test.exe",
              u8"D:\\test.exe",
    [](long total, long downloaded) {
        // progress callback
    }, 
    [](long byte_per_secs) {
        // realtime speed callback
    })
    .then([=](pplx::task<Result> result) {
        std::cout << std::endl << GetResultString(result.get()) << std::endl;
        if (result.get() == Result::Successed) {
			// Successed
        }
    }).wait();
	
    Teemo::GlobalUnInit();
	
	return 0;
}
```

---

# Command line tool
`teemo` is command line download tool based on `teemo` library. Usage:

```bash
teemo_tool URL TargetFilePath [ThreadNum] [MD5] [EnableSaveSliceToTmp] [SliceCacheExpiredSeconds] [MaxSpeed]
```

- URL: Download URL.
- TargetFilePath: target file saved path.
- ThreadNum: thread number, optional, default is `1`.
- MD5: target file md5, optional, if this value isn't empty, tools will check file md5 after download finished.
- EnableSaveSliceToTmp: 0 or 1, optional, whether save slice file to system temp directory or not, Windows system is the path returned by `GetTempPath` API, Linux is `/var/tmp/`.
- SliceCacheExpiredSeconds: seconds, optional, slice cache file expired after these senconds.
- MaxSpeed: max download speed(byte/s).
