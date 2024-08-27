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

# 生成编辑器索引文件
make LLVM=1 ARCH=riscv rust-analyzer
bear -- make ARCH=riscv LLVM=1 -j12
```



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



## 加载内核模块

在使用`make ARCH=riscv LLVM=1 menuconfig` 进行配置时，我们可以选择开启编译一些内核内置的rust模块

这些模块被编译后不会和内核镜像绑定在一起，因此我们需要手动将其拷贝到制作的文件系统中，在文件系统中，使用insmod相关的命令加载和卸载这些模块。

`make ARCH=riscv LLVM=1 modules_install` 这个命令会把编译的内核模块拷贝到本机目录下，因此通常不使用

`make ARCH=riscv LLVM=1 modules` 只会编译选择的内核模块



## 构建ubuntu镜像

为了后续方便在系统上进行测试，构建一个ubuntu/debian镜像是必须的。这样一来就可以使用常见的性能测试工具。

依赖安装：

```
sudo apt install debootstrap qemu qemu-user-static binfmt-support
```

生成最小 bootstrap rootfs 

```
sudo debootstrap --arch=riscv64 --foreign jammy ./temp-rootfs http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports
```

拷贝qemu-riscv64-static到rootfs中

```
cp /usr/bin/qemu-riscv64-static ./temp-rootfs/usr/bin/
```

 chroot 和 debootstrap

```
wget https://raw.githubusercontent.com/ywhs/linux-software/master/ch-mount.sh
chmod 777 ch-mount.sh
./ch-mount.sh -m temp-rootfs/
# 执行脚本后，没有报错会进入文件系统，显示 I have no name ，这是因为还没有初始化。
debootstrap/debootstrap --second-stage
exit
./ch-mount.sh -u temp-rootfs/
./ch-mount.sh -m temp-rootfs/
```

这里如果在WSL2进行实验，可能会遇到无法chroot的情况。issue https://github.com/microsoft/WSL/issues/2103#issuecomment-1829496706给出了解决办法。

修改软件源：

```
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-updates main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-backports main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-security main restricted universe multiverse
```

安装常见的工具

```
apt-get update
apt-get install --no-install-recommends -y util-linux haveged openssh-server systemd kmod initramfs-tools conntrack ebtables ethtool iproute2 iptables mount socat ifupdown iputils-ping vim dhcpcd5 neofetch sudo chrony
```



制作ext4的镜像

```
dd if=/dev/zero of=rootfs_ubuntu_riscv.ext4 bs=1M count=4096
mkfs.ext4 rootfs_ubuntu_riscv.ext4
mkdir -p tmpfs
sudo mount -t ext4 rootfs_ubuntu_riscv.ext4 tmpfs/ -o loop
sudo cp -af temp-rootfs/* tmpfs/
sudo umount tmpfs
chmod 777 rootfs_ubuntu_riscv.ext4
```

使用ssh连接启动的qemu

https://copyright1999.github.io/2023/10/03/wsl%E5%A6%82%E4%BD%95ssh%E5%88%B0qemu/

启动`qemu`之后，还要在我们`qemu`模拟的`debian`中做好`ssh`相关的配置，在`/etc/ssh/sshd_config`中加上一句

```
PermitRootLogin yes
```

然后重启`ssh`相关服务

```
service ssh restart
```

设置内核模块输出到控制台

https://blog.csdn.net/dp__mcu/article/details/119887176

http://www.only2fire.com/archives/27.html

将linux编译的内核模块安装到文件系统中：

```
sudo make ARCH=riscv LLVM=1 modules_install INSTALL_MOD_PATH=../mnt/kmod
```



https://blog.csdn.net/jingyu_1/article/details/135822574

https://cloud.tencent.com/developer/article/1914743

https://blog.csdn.net/xuesong10210/article/details/129167731

## 编写内核模块

c内核模块https://linux-kernel-labs-zh.xyz/labs/kernel_modules.html







## Reference

[编译 linux for rust 并制作 initramfs 最后编写 rust_helloworld 内核驱动 并在 qemu 环境加载](https://blog.csbxd.fun/archives/1699538198511)

[内核模块入门](https://github.com/ljrcore/linuxmooc/blob/master/%E7%B2%BE%E5%BD%A9%E6%96%87%E7%AB%A0/%E6%96%B0%E6%89%8B%E4%B8%8A%E8%B7%AF%EF%BC%9A%E5%86%85%E6%A0%B8%E6%A8%A1%E5%9D%97%E5%85%A5%E9%97%A8.md)

