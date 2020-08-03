# IMX6ULL-QEMU

## Gcc version 7.5.0 (Linaro GCC 7.5-2019.12)

[gcc下载地址http://releases.linaro.org/components/toolchain/binaries/7.5-2019.12/arm-linux-gnueabi/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabi.tar.xz](http://releases.linaro.org/components/toolchain/binaries/7.5-2019.12/arm-linux-gnueabi/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabi.tar.xz)

## BusyBox(1.31.1)

使用[配置文件imx6ull_defconfig](imx6ull_defconfig)编译busybox,默认安装在当前目录ramdiskfs下

	make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- imx6ull_defconfig
	make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- -j4
	make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- install

制作根文件系统镜像文件

    mkdir -p ramdiskfs/{var,tmp,sys,root,proc,opt,mnt,lib,home,etc,dev}
    sudo mknod ramdiskfs/dev/console c 5 1
    sudo mknod ramdiskfs/dev/null c 1 3
    dd if=/dev/zero of=ramdisk.raw bs=1M count=16
    mkfs.ext4 -F ramdisk.raw
	mv ramdisk.raw ramdisk.ext4
    mkdir -p tempfs
    sudo mount -t ext4 -o loop ramdisk.ext4 tempfs
    sudo cp -raf ramdiskfs/* tempfs/
    sudo umount tempfs/
    gzip --best -c ramdisk.ext4 > ramdisk.gz
    mkimage -n "ramdisk" -A arm -O linux -T ramdisk -C gzip -d ramdisk.gz ramdisk.img

## Kernel(4.9.88)

代码下载

	git clone https://e.coding.net/weidongshan/qemu_imx6ull_kernel

编译内核

	make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- 100ask_imx6ull_qemu_defconfig
	make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- zImage -j4
	make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- dtbs -j4

## Qemu(v4.1.50)

下载v4.1.50版本的代码打上相应补丁

	git checkout f8f2eac4e5 -b 100ask
	git am < 0001-100ask-imx6ull-board-patch.patch

配置编译

	./configure --prefix=$PWD/ --target-list="arm-softmmu arm-linux-user" --enable-debug --enable-sdl --enable-kvm --enable-tools --disable-curl
	make -j4 && make install

## 启动虚拟机

使用下面命令启动虚拟机

	qemu-system-arm -M mcimx6ul-evk -m 512M \
		-kernel zImage \
		-dtb 100ask_imx6ull_qemu.dtb  \
		-nographic -serial mon:stdio \
		-drive file=ramdisk.ext4,format=raw,id=mysdcard -device sd-card,drive=mysdcard \
		-append "console=ttymxc0,115200 rootfstype=ext4 root=/dev/mmcblk1 rw rootwait init=/sbin/init loglevel=8" \
		-nic user

带图形显示启动虚拟机

	qemu-system-arm -M mcimx6ul-evk --show-cursor -m 512M \
		-kernel zImage \
		-dtb 100ask_imx6ull_qemu.dtb  \
		-display sdl -serial mon:stdio \
		-drive file=ramdisk.ext4,format=raw,id=mysdcard -device sd-card,drive=mysdcard \
		-append "console=ttymxc0,115200 rootfstype=ext4 root=/dev/mmcblk1 rw rootwait init=/sbin/init loglevel=8" \
		-nic user -com 100ask
