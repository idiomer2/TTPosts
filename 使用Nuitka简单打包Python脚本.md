
## 安装
```bash
conda create -n nuitka python=3.12 -y
conda activate nuitka

pip install nuitka
```


## 编译打包
```bash
nuitka --standalone --onefile --output-filename=douyin_mp3_download.exe douyin_mp3_download.py
```

如果报错没有gcc类似编译器，则需要安装编译器（因为nuitka是将py脚本编译为C来实现的）

1. 访问 [mingw的github release](https://github.com/niXman/mingw-builds-binaries/releases)
2. 下载 14.2.0：[x86_64-14.2.0-release-posix-seh-msvcrt-rt_v12-rev2.7z](https://github.com/niXman/mingw-builds-binaries/releases/download/14.2.0-rt_v12-rev2/x86_64-14.2.0-release-posix-seh-msvcrt-rt_v12-rev2.7z)
3. 解压后，将解压后的bin目录添加到环境变量即可
4. 重新运行上面的nuitka打包命令，中间提示需要下载mingw的一些libs，同意即可


## 参考资料
- [Nuitka User Manual — Nuitka the Python Compiler](https://nuitka.net/user-documentation/user-manual.html)


