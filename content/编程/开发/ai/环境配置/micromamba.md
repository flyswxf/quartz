相当于conda, 而且兼容conda, 可以使用conda注册的env

一键安装(下载并配置)
```ps
Invoke-Expression ((Invoke-WebRequest -Uri https://micro.mamba.pm/install.ps1 -UseBasicParsing).Content)
```
会有一个默认的环境于C盘, 但这个环境是空的, 所以不会占用空间

## 下载源配置
下载源配置在`~\.condarc`, 即用户目录下的condarc文件

通过此命令查看**config文件位置**和**具体config内容**
```
micromamba config list --sources   
```

```
channels:
  - defaults
show_channel_urls: true
default_channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  msys2: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
```

- `pkgs/main/`: Anaconda 主源
- `pkgs/free/`: 已废弃的源（Anaconda 多年前已将 pkgs/free 合并到 pkgs/main），虽目录存在但包极少且过时
- `conda-forge`: 社区最大的第三方源，默认会走官方源
- `defaults`: Anaconda 官方默认源（包含 pkgs/main、pkgs/r 等），但未配置镜像时在国内访问较慢
- `msys2`: Windows 编译工具链源，仅在需要编译 C/C++ 包时使用

当install package时, micromamba会从最上方的源开始寻找, 如果都没有找到, 则会报错

注意: 
并不需要设置`micromamba config set channel_priority strict`如果设置了, 可能还会出现错误. 通过`micromamba config set channel_priority flexible`改回来

对应设置命令, **也可以直接修改condarc文件**
```ps
micromamba config append channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
micromamba config append channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge
micromamba config append channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/
micromamba config append channels defaults
micromamba config append channels conda-forge
```
