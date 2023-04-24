#micromamba #conda

conda太慢了，还老崩溃，老烦人了。
所以我用micromamba来代替conda
默认情况下，肯定是没安装的，需要执行以下步骤来安装
```bash
#下载最新版的micromamba
curl -Ls https://micro.mamba.pm/api/micromamba/linux-64/latest | tar -xvj bin/micromamba

#设置micromamba的安装路径
miniconda_path=/data/Users/yourname/miniconda3/
mkdir -p $miniconda_path
export MAMBA_ROOT_PREFIX=$miniconda_path

#开始安装
./micromamba shell init

#设置清华源
echo 'channels:
  - defaults
show_channel_urls: true
default_channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  bioconda: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud' > ~/.condarc

#以下是建议步骤：给micromamba起个别名：conda
sed -i  '$ a alias conda=micromamba'  ~/.bashrc
```
