# 编译Linux Kernel

## 下载源码
```bash
wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.10.tar.xz
tar xf linux-6.10.tar.xz
cd linux-6.10
```


## 配置并编译
安装编译工具和必要的库
```bash
sudo dnf install -y ncurses-devel bison flex elfutils-libelf-devel openssl-devel
```

使用默认配置，配置kernel
```bash
make defconfig
```
或者使用自定义配置
```bash
make menuconfig
```

编译
```bash
make -j$(nproc)
```

## 创建iso镜像
内核镜像文件（vmlinuz-* 或 bzImage）
编译完成后，内核镜像通常在源码目录的 arch/x86/boot/ 目录下（如果是 x86 平台）。

### 创建 initramfs 文件系统
下载 busybox
```bash
wget https://busybox.net/downloads/binaries/1.35.0-i686-linux-musl/busybox
```

```
mkdir temp
cd temp
```

```bash
mkdir -p rootfs/{bin,sbin,etc,proc,sys,usr/{bin,sbin}}
cp ../busybox rootfs/bin/
```

创建更多的busybox命令
```bash
cd rootfs/bin
ln -s busybox sh
ln -s busybox ls
ln -s busybox cp
ln -s busybox mv
ln -s busybox rm
ln -s busybox mkdir
ln -s busybox echo
ln -s busybox dmesg
ln -s busybox ps
ln -s busybox cat
ln -s busybox grep
ln -s busybox ping
ln -s busybox ifconfig
ln -s busybox mount
ln -s busybox umount
ln -s busybox vi
ln -s busybox more
ln -s busybox head
ln -s busybox tail
ln -s busybox free
ln -s busybox df
ln -s busybox du
```

创建 init 脚本
```bash
cat > rootfs/init << 'EOF'
#!/bin/sh
/bin/busybox mount -t proc none /proc
/bin/busybox mount -t sysfs none /sys
echo "Welcome to My Kernel"
exec /bin/sh
EOF
chmod +x rootfs/init
```

打包为 initramfs
```bash
cd rootfs
find . | cpio -o --format=newc | gzip > ../initramfs.cpio.gz
```

### 创建grub配置文件

拷贝内核镜像和 initramfs 到 iso 目录
```bash
mkdir -p iso/boot/grub
cp ../arch/x86/boot/bzImage iso/boot/vmlinuz
cp initramfs.cpio.gz iso/boot/
```

创建grub配置文件
```bash
cat > iso/boot/grub/grub.cfg << 'EOF'
set timeout=5
set default=0

menuentry "Custom Linux Kernel" {
    linux /boot/vmlinuz
    initrd /boot/initramfs.cpio.gz
}
EOF
```


### 创建iso镜像
```bash
sudo dnf install -y grub2 xorriso grub2-tools mtools
```
创建镜像文件
```bash
grub2-mkrescue -o custom-linux.iso iso
```

