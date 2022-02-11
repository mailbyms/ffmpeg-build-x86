# ffmpeg-build-x86
自行编译的 x86 32位版本 FFMPEG SDK，使用 MSYS2 MinGW x86 环境编译

## Windows 下编译 ffmpeg 的说明

 使用 msys2 编译（成功），参考：https://www.freesion.com/article/675088944/ 

> msys 相当于操作系统，提供 bash, make, pacman（类似 yum）mingw 相当于 MSVC，提供 gcc 等。【引用官网】
>
> mingw 会继承 msys 的环境变量。（分别打开 msys2.exe 和 mingw64, 运行 echo $PATH 可知）

msys 可以直接用 pacman 安装 gcc ，但生成的程序会是依赖 msys-2.0.dll，所以要安装 mingw 的 gcc 是否使用 MSVC 的 link.exe，没有影响



## 编译步骤

安装 msys2：https://www.msys2.org/ （msys2 自带 mingw），例如 https://github.com/msys2/msys2-installer/releases/download/2022-01-18/msys2-x86_64-20220118.exe （安装时断开网络，否则卡在 更新信任库步骤直到超时）

以下在 msys 的环境下运行：

```shell
#安装公有的库
pacman -S make diffutils pkg-config yasm
#分别安装 32 位和 64 位的 gcc
pacman -Sl | grep gcc
pacman -S mingw-w64-i686-gcc
pacman -S mingw-w64-x86_64-gcc
```



注意

- 在运行 ffmpeg 的 configure 命令前，删除 C:\msys64\mingw32 和 C:\msys64\mingw64 下的所有 *.dll.a 文件，即 在 mingw 下，mingw 相关的库使用静态方式链接

- 32位的 mingw gcc 编译出来的库，会依赖 libgcc_s_dw2-1.dll。需要在 ./configure 时增加参数： `--extra-ldflags='-static-libgcc -static-libstdc++' `；64位的不需要；

- 目前官网下载的 FFMPEG 预编译包是 64位。见官方 wiki 说明【ffmpeg.org 左边 WIKI -> "FFmpeg Compilation Guides " -> "CompilationGuide/MinGW"】：https://trac.ffmpeg.org/wiki/CompilationGuide/MinGW



开始菜单打开 MingW x86 【 如果编译 64 位的库，则打开 MingW x64】的图标

  ```shell
  # 复制自官方 ffmpeg 的配置
  ./configure --enable-gpl --enable-version3 --enable-shared --disable-w32threads --disable-autodetect --extra-ldflags='-static-libgcc -static-libstdc++'
  ```

  说明： 运行 ./configure 后 shell 窗口停顿，可以 `tail -f ffbuild/config.log` 查看进展
  ```
  # configure 命令完成后，运行以下命令，编译后的文件会安装在 /usr/local/bin 下
  make -j 4
  make install
  ```

  

## 添加 h264 编码库

默认带了 h264 解码库，如果要加入 h264 编码库(libx264)，使用 msys 的 pacman 安装 x264 库，再重新编译 ffmpeg 即可，在 msys 窗口下：

```shell
# 查看仓库
pacman -Sl | grep x264
# 安装 32 位和 64 位的
pacman -S mingw-w64-i686-x264
pacman -S mingw-w64-x86_64-x264
```


同样，删除 C:\msys64\mingw32 和 C:\msys64\mingw64 下的所有 *.dll.a 文件，然后在 ./configure 时加入 --enable-libx264



## 题外

如果要用 MSVC 的 link 工具，则需要修改 msys2_shell.cmd ，把 L17 的 注释去掉，即 `rem set MSYS2_PATH_TYPE=inherit` 改为 `set MSYS2_PATH_TYPE=inherit`

然后找到 Windows 开始菜单 -> "适用于 VS 2017 的 x64 本机工具命令提示 (2)"，初始化 VS 的环境变量，再使用 `msys2_shell.cmd -mingw64` 或者 `msys2_shell.cmd -mingw32` 弹出 mingw 窗口。这样 mingw 会继承 VS 的环境变量，运行 `link` 命令可证实

以下仅参考：

>https://www.cnblogs.com/schips/p/12579218.html
>
>https://blog.csdn.net/listener51/article/details/81605472
>
>用 wsl 编译 FFMPEG：https://www.bilibili.com/read/cv7058269/
>
>官方说明：https://trac.ffmpeg.org/wiki/CompilationGuide/MSVC



Q: 如果判断 exe 是32 位还是 64 位？

A: 文件右键 -> 属性 -> 兼容性，

- 32位的兼容模式可以回退到 xp/win98/win95，64位的只能选到 vista
- 32位的可以设置简化的颜色模式，64位的不可勾选

