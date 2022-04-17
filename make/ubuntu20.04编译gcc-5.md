# `ubuntu20.04`编译`gcc-5.5.0`

> 由于需要在`qemu`中运行`kernel-3.18.6`，需要`gcc-5`来编译内核。
> 
> 但`gcc-5`在`ubuntu20.04`上不受支持，需要手动移植。

### 1、源码下载

```sh
git clone git://gcc.gnu.org/git/gcc.git
cd gcc
git checkout releases/gcc-5.5.0
# git submodule update --init
```

### 2、依赖安装

```sh
sudo apt install build-essential libgmp-dev libmpc-dev texinfo
```

### 3、源码修改

**./libsanitizer/sanitizer_common/sanitizer_internal_defs.h**

```cpp
/* 第270行 */
#define IMPL_PASTE(a, b) a##b
#define IMPL_COMPILER_ASSERT(pred, line) \
    typedef char IMPL_PASTE(assertion_failed_##_, line)[2*(int)(pred)-1 > 0 ? 2*(int)(pred)-1 : 1] //修改
```

**./libsanitizer/sanitizer_common/sanitizer_platform_limits_posix.cc**

```cpp
/* 141行 */
#include <sys/ustat.h> //删除
/* 234行 */
  unsigned struct_ustat_sz = sizeof(struct ustat); //删除
```

### 4、`make`编译

```sh
mkdir build
cd build
../configure --prefix=<自定安装位置>
make -j$nproc
make install
```

### 5、内核编译使用方式

```sh
make -j$nproc CC=<gcc-5安装目录>/bin/gcc CXX=<gcc-5安装目录>/bin/g++
```
