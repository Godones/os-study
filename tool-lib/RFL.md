# Rust for Linux

记录rust for linux 的使用过程, 这里尝试在`riscv`平台启动，因为后期的代码与`riscv`相关。

## 编译

```
git clone --depth=1 git@github.com:Godones/linux.git -b rust-dev
```

`rust-dev` 分支是开发最快的分支，因此我们选择这个分支进行实验。

```
cd linux
# 从https://www.rust-lang.org安装rustup
rustup override set $(scripts/min-tool-version.sh rustc)	# rustc
rustup component add rust-src	# rust-src
sudo apt install clang llvm		# libclang
cargo install --locked --version $(scripts/min-tool-version.sh bindgen) bindgen	# bindgen
rustup component add rustfmt	# rustfmt
rustup component add clippy		# clippy
make LLVM=1 rustavailable		# 验证如上依赖安装无误，输出“Rust is available!”
```

```
# for amd64
make LLVM=1 menuconfig
	# Kernel hacking -> Sample kernel code -> Rust samples 选择一些模块
make LLVM=1 -j

# for aarch64
make ARCH=arm64 CLANG_TRIPLE=aarch64_linux_gnu LLVM=1 menuconfig
make ARCH=arm64 CLANG_TRIPLE=aarch64_linux_gnu LLVM=1 -j

# for riscv64
# 注： meuconfig时关闭kvm模块，否则内核有bug不能成功编译
make LLVM=1  ARCH=riscv defconfig
make ARCH=riscv LLVM=1 menuconfig
make ARCH=riscv LLVM=1 -j32
make LLVM=1 ARCH=riscv rust-analyzer
bear -- make ARCH=riscv LLVM=1 -j12
```



https://blog.csbxd.fun/archives/1699538198511

## Linux on riscv

https://blog.csdn.net/youzhangjing_/article/details/129556309

1. 安装riscv的工具链:  riscv64-linux-gnu-
2. 安装qemu
3. 编译kernel

```
export ARCH=riscv
export CROSS_COMPILE=riscv64-linux-gnu-
 
make defconfig
make -j8
```

编译完成后，在arch/riscv/boot下生成Image

4. 制作rootfs，使用buildroot工具直接一步到位生成。编译完后，生成文件在output/images目录下
5. 将Image、rootfs.ext2拷贝到同一目录下，运行qemu命令

```
#!/bin/sh
 
qemu-system-riscv64 -M virt \
-kernel Image \
-append "rootwait root=/dev/vda ro" \
-drive file=rootfs.ext2,format=raw,id=hd0 \
-device virtio-blk-device,drive=hd0 \
-netdev user,id=net0 -device virtio-net-device,netdev=net0 -nographic
```



在wsl2中，编译buildroot可能会报错，原因是其会检查环境变量，但是PATH环境变量中包含windows的路径，里面包含空格。

解决办法是直接指定PATH变量进行编译

```
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin make
```

https://blog.csdn.net/weixin_40837318/article/details/134328622



另一种途径：

1. 编译linux

```
make ARCH=riscv CROSS_COMPILE=riscv64-linux-gnu- defconfig
make ARCH=riscv CROSS_COMPILE=riscv64-linux-gnu- -j $(nproc)
```

2. 使用busybox构建根文件系统

```
git clone https://gitee.com/mirrors/busyboxsource
cd busyboxsource
export CROSS_COMPILE=riscv64-linux-gnu-
make defconfig
make menuconfig
这里启用了 Settings-->Build Options 里的 Build static binary (no shared libs) 选项
make -j $(nproc)
make install
```

3. 制作文件系统并新建一个启动脚本

```
$ cd ~
$ qemu-img create rootfs.img 1g
$ mkfs.ext4 rootfs.img
$ mkdir rootfs
$ sudo mount -o loop rootfs.img rootfs
$ cd rootfs
$ sudo cp -r ../busyboxsource/_install/* .
$ sudo mkdir proc sys dev etc etc/init.d
$ cd etc/init.d/
$ sudo touch rcS
$ sudo vi rcS
```

https://gitee.com/YJMSTR/riscv-linux/blob/master/articles/20220816-introduction-to-qemu-and-riscv-upstream-boot-flow.md
