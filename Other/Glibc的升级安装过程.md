# Glibc的升级安装过程

## Glibc版本低导致的问题`/lib/libc.so.6: version `GLIBC_2.14' not found`

下载最新的Glibc源码安装升级[网址](https://www.gnu.org/software/libc/index.html)

最好下载`tar.xz`格式的源码，因为在下载的过程中发现`tar.gz`格式总是有文件损坏的问题。

`tar.xz`格式的源码先需要使用`xz -d xxx.tar.xz`命令将其转换为`xxx.tar`格式的文件，然后利用`tar -xvf xxx.tar`即可以解压源码

进入源码目录，开始编译安装

```shell
mkdir build
cd build
../configure --prefix=/opt/glibc-2.14 --disable-profile
make -j8
make install
```
## 错误`inlining failed in call to always_inline ‘syslog’: function not inlinable`

需要配置编译命令参数：

```shell
export CFLAGS="-O2 -U_FORTIFY_SOURCE -mtune=native -fno-stack-protector"
```

## 错误`can't open configuration file /opt/glibc-2.14/etc/ld.so.conf no such file or directory`

创建此文件

```shell
touch /etc/ld.so.conf
```

## 编译成功

编译成功后，需要将新的glibc加入到环境变量中：

```shell
LD_LIBRARY_PATH=/opt/glibc-2.14/lib:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH
```
