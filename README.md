# i686交叉编译器制作指南
## 写在前面
交叉编译器的版本：
+ binutils 2.38
+ gcc 11.1.0
+ glibc 2.35
+ linux 5.15 

测试通过的机器列表：

+ Ubuntu 22.04 ARM64

## 源码编译交叉编译器
安装需要的环境
```shell
$ sudo apt update
$ sudo apt install -y git 
```

下载交叉编译器代码
```shell
$ cd ~/
$ git clone https://github.com/NelsonCheung-cn/i686-compiler.git
```
设置环境变量
```shell
$ export PATH=$PATH:/opt/i686/bin
$ alias sudo='sudo env PATH=$PATH:/opt/i686/bin'
```
编译binutils

```shell
$ cd ~/i686-compiler/binutils
$ mkdir build && cd build
$ ../configure --prefix=/opt/i686/ --target=i686-linux-gnu
$ make -j$(nproc)
$ sudo make -j$(nproc) install
```

编译Linux内核i386架构的头文件
```shell
$ cd ~/i686-compiler/linux
$ sudo make ARCH=i386 INSTALL_HDR_PATH=/opt/i686/i686-linux-gnu/ headers_install
```

编译gmp库
```shell
$ cd ~/i686-compiler/gcc/gmp
$ mkdir build && cd build
$ ../configure --prefix=/opt/i686/
$ make -j$(nproc)
$ sudo make install
```

编译mpfr库
```shell
$ cd ~/i686-compiler/gcc/mpfr
$ mkdir build && cd build
$ ../configure --prefix=/opt/i686/ --with-gmp=/opt/i686/ 
$ make -j$(nproc)
$ sudo make install
```

编译mpc库
```shell
$ cd ~/i686-compiler/gcc/mpc
$ mkdir build && cd build
$ ../configure --prefix=/opt/i686/ --with-gmp=/opt/i686/ --with-mpfr=/opt/i686/
$ make -j$(nproc)
$ sudo make install
```

编译isl库
```shell
$ cd ~/i686-compiler/gcc/isl
$ mkdir build && cd build
$ ../configure --prefix=/opt/i686/ --with-gmp-prefix=/opt/i686/
$ make -j$(nproc)
$ sudo make install
```

编译C/C++编译器
```shell
$ cd ~/i686-compiler/gcc
$ mkdir build && cd build
$ ../configure --prefix=/opt/i686/ --target=i686-linux-gnu --enable-languages=c,c++ --disable-multilib
$ make -j$(nproc) all-gcc
$ sudo make -j$(nproc) install-gcc
```

编译Glibc标准库文件和Gcc后续编译需要的文件
```shell
$ cd ~/i686-compiler/glibc
$ mkdir build && cd build
$ ../configure --prefix=/opt/i686/i686-linux-gnu/ --host=i686-linux-gnu --target=i686-linux-gnu --with-headers=/opt/i686/i686-linux-gnu/include/ --disable-werror --enable-add-ons --disable-multilib
$ sudo make install-bootstrap-headers=yes install-headers
$ sudo make -j$(nproc) csu/subdir_lib 
$ sudo install csu/crt1.o csu/crti.o csu/crtn.o /opt/i686/i686-linux-gnu/lib
$ sudo i686-linux-gnu-gcc -nostdlib -nostartfiles -shared -x c /dev/null -o /opt/i686/i686-linux-gnu/lib/libc.so
$ sudo touch /opt/i686/i686-linux-gnu/include/gnu/stubs.h
```

编译libgcc
```shell
$ cd ~/i686-compiler/gcc/build
$ make -j$(nproc) all-target-libgcc
$ sudo make install-target-libgcc
```

编译Glibc
```shell
$ cd ~/i686-compiler/glibc/build
$ make -j$(nproc)
$ sudo make install
```

编译标准C++库
```shell
$ cd ~/i686-compiler/gcc/build
$ make -j$(nproc)
$ sudo make install
```

最后，在`/opt/i686/bin`下可以看到交叉编译器
```shell
$ ls /opt/i686/bin
i686-linux-gnu-addr2line
i686-linux-gnu-ar
i686-linux-gnu-as
i686-linux-gnu-c++
i686-linux-gnu-c++filt
i686-linux-gnu-cpp
i686-linux-gnu-elfedit
i686-linux-gnu-g++
i686-linux-gnu-gcc
i686-linux-gnu-gcc-11.1.0
i686-linux-gnu-gcc-ar
i686-linux-gnu-gcc-nm
i686-linux-gnu-gcc-ranlib
i686-linux-gnu-gcov
i686-linux-gnu-gcov-dump
i686-linux-gnu-gcov-tool
i686-linux-gnu-gprof
i686-linux-gnu-ld
i686-linux-gnu-ld.bfd
i686-linux-gnu-lto-dump
i686-linux-gnu-nm
i686-linux-gnu-objcopy
i686-linux-gnu-objdump
i686-linux-gnu-ranlib
i686-linux-gnu-readelf
i686-linux-gnu-size
i686-linux-gnu-strings
i686-linux-gnu-strip
```