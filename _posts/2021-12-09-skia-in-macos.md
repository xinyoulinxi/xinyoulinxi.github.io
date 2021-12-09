---
layout: post
title: "在macos上编译和运行skia"
date: 2021-12-09
tags:
    - skia
    - c++
    - macos
author: "ylvoid"
header-mask: 0.2
header-img: "imgs/page_bg.jpg"
---

# skia下载、编译
首先clone skia的工具仓库depot_tools，并将其加入到path中：
```
git clone 'https://chromium.googlesource.com/chromium/tools/depot_tools.git'
export PATH="${PWD}/depot_tools:${PATH}"
```

拉取skia仓库，运行tools/git-sync-deps自动下载skia的依赖：
```
git clone https://skia.googlesource.com/skia.git
cd skia
python2 tools/git-sync-deps
```

编译构建debug静态库：
```
## /skia/
bin/gn gen out/Static --args='is_official_build=false'
python2 tools/git-sync-deps
ninja -C out/Static
```
或者通过这个命令编译一个release的静态库：
```
## /skia/
bin/gn gen out/Release --args="is_official_build=true is_component_build=true skia_use_system_expat=false skia_use_system_icu=false skia_use_system_libjpeg_turbo=false skia_use_system_libpng=false skia_use_system_libwebp=false skia_use_system_zlib=false extra_cflags_cc=[\"-frtti\"]"

python2 tools/git-sync-deps

ninja -C out/Release skia
```
然后out/Static目录下生成了skia的静态库：libskia.a 和 libpathkit.a或者动态库so

```
## /skia/out/Release
ls -al
7.8M 12  2 21:49 libskia.so
```


# 编写cmake文件链接skia

将上面两个静态库和skia库中的include目录移动到我们自己工程目录下，然后编写我们的CMAKE文件（CMAKELISTS.txt）：

```
cmake_minimum_required(VERSION 3.12)
project(skia_tutorial)

set(CMAKE_CXX_STANDARD 17)

set(SKIA_SDK "${CMAKE_SOURCE_DIR}/skia")
set(SDL_INCLUDE "${CMAKE_SOURCE_DIR}/SDL2")

include_directories(
    ${SDL_INCLUDE}/
    ${SKIA_SDK}/
    ${SKIA_SDK}/include/
    ${SKIA_SDK}/include/android 
    ${SKIA_SDK}/include/atlastext
    ${SKIA_SDK}/include/c
    ${SKIA_SDK}/include/codec
    ${SKIA_SDK}/include/config
    ${SKIA_SDK}/include/core
    ${SKIA_SDK}/include/effects
    ${SKIA_SDK}/include/encode
    ${SKIA_SDK}/include/gpu
    ${SKIA_SDK}/include/gpu/gl
    ${SKIA_SDK}/include/gpu/mock
    ${SKIA_SDK}/include/gpu/mtl
    ${SKIA_SDK}/include/gpu/vk
    ${SKIA_SDK}/include/pathops
    ${SKIA_SDK}/include/ports
    ${SKIA_SDK}/include/private
    ${SKIA_SDK}/include/svg
    ${SKIA_SDK}/include/utils
    ${SKIA_SDK}/include/utils/mac
    ${SKIA_SDK}/include/views)


add_executable(skia_tutorial main.cc)

target_link_libraries(skia_tutorial ${SKIA_SDK}/libs/libskia.a)
target_link_libraries(skia_tutorial ${SKIA_SDK}/libs/libpathkit.a)
if (APPLE)
    target_link_libraries(skia_tutorial "-framework CoreServices")
    target_link_libraries(skia_tutorial "-framework CoreGraphics")
    target_link_libraries(skia_tutorial "-framework CoreText")
    target_link_libraries(skia_tutorial "-framework CoreFoundation")
endif()

```
# 测试demo
```
#include "SkSurface.h"
#include "SkPath.h"
#include "SkCanvas.h"
#include "SkData.h"
#include "SkImage.h"
#include "SkStream.h"

int main (int argc, char * const argv[]) {
  // hard coded example program parameters
  const char filePath[] = "${your_workspace}/skia_test/test_img.png";
  int width = 256;
  int height = 256;

  // create canvas to draw on
  sk_sp<SkSurface> rasterSurface = SkSurface::MakeRasterN32Premul(width, height);
  SkCanvas* canvas = rasterSurface->getCanvas();

  // creating a path to be drawn
  SkPath path;
  path.moveTo(0,0);
  SkRect rect;
  rect.setLTRB(20,20,200,200);
  path.addRect(rect);
  path.close();

  // creating a paint to draw with
  SkPaint p;
  p.setAntiAlias(true);

  // clear out which may be was drawn before and draw the path
  canvas->clear(SK_ColorWHITE);
  canvas->drawPath(path, p);

  // make a PNG encoded image using the canvas
  sk_sp<SkImage> img(rasterSurface->makeImageSnapshot());
  if (!img) { return 1; }
  sk_sp<SkData> png(img->encodeToData());
  if (!png) { return 1; }

  // write the data to the file specified by filePath
  SkFILEWStream out(filePath);
  (void)out.write(png->data(), png->size());

  return 0;
}
```
最后，运行cmake生成make文件并进行编译和运行：
```
## /my_project/
cmake .
make
./skia_tutorial
```
# 编译控制参数

可以通过如下命令查看编译和链接参数用于控制skia的编译产物：

```
gn args --list out/Release
```

```
ar
    Current value (from the default) = "ar"
      From //gn/BUILDCONFIG.gn:20

cc
    Current value (from the default) = "cc"
      From //gn/BUILDCONFIG.gn:21

cc_wrapper
    Current value (from the default) = ""
      From //gn/toolchain/BUILD.gn:35

clang_win
    Current value (from the default) = ""
      From //gn/BUILDCONFIG.gn:30

clang_win_version
    Current value (from the default) = ""
      From //gn/BUILDCONFIG.gn:31

current_cpu
    Current value (from the default) = ""
      (Internally set; try `gn help current_cpu`.)

current_os
    Current value (from the default) = ""
      (Internally set; try `gn help current_os`.)

cxx
    Current value (from the default) = "c++"
      From //gn/BUILDCONFIG.gn:22
xcode_sysroot
    Current value (from the default) = ""
      From //gn/skia/BUILD.gn:19

    ......
    ......
    ......
```