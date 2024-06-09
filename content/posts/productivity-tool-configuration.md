+++
title = 'Productivity Tool Configuration'
date = 2023-11-12T11:10:11+08:00
lastmod = 2024-06-10T02:17:00+08:00
draft = false
+++

## 安装WSL2

Windows Subsystem for Linux 2（简称WSL2），是Windows 10和Windows 11操作系统内置的Linux运行环境。WSL2提供了一个完整的Linux内核，使用户可以在Windows上运行Linux应用程序，同时可以通过使用Windows的本地文件系统来管理Linux文件。

1. **Windows版本要求**

必须运行 Windows 10 版本 2004 及更高版本(内部版本 19041 及更高版本)或 Windows 11

2. **开启硬件虚拟化**

WSL2要求开启硬件虚拟化功能。在任务管理器-性能中检查硬件虚拟化是否打开

3. **启用windows功能**

win+r 运行 OptionalFeatures

勾选以下三项功能，确定后等待安装完成，重启计算机

- 「Hyper-V」
- 「适用于Linux的Windows子系统」
- 「虚拟机平台」

注：适用于Linux的Windows子系统 是WSL1，安装好后需要手动升级到WSL2

4. **WSL1升级到WSL2**

以管理员身份打开powershell

```shell
# 执行升级命令
wsl --update

# 升级完成后，查看版本，如果显示「WSL 版本： 2.1.4.0」，以2开头的版本号，说明成功升级到WSL2
wsl --version

# 设置WSL默认版本为WSL2（设置1可以切回WSL1）
wsl --set-default-version 2
```

## 安装ubnutu

以管理员身份打开powershell

```shell
# 查看已安装的分发
wsl --list --verbose
wsl -l -v

# 显示所有可安装的分发
wsl --list --online

# 不指定则默认安装最新的Ubuntu
wsl --install
# 安装指定分发版本
wsl --install Ubuntu-24.04

# 将现有的WSL转换为WSL2
wsl --set-version Ubuntu-24.04 2
# 将现有的WSL2转换为WSL
wsl --set-version Ubuntu-24.04 1

# 强制停止运行
wsl --shutdown

# 卸载分发
wsl --unregister Ubuntu-24.04
```

安装完成后，会提示创建一个用户，默认会加入sudo组



## WSL出现Read-only file system报错

通常是磁盘有错误，可以用e2fsck修复

```shell
# 查看挂载的磁盘
mount | grep ext4
# 有一个属性errors=remount-ro，说明磁盘如果有错误，就会被挂载成只读状态

# 执行修复
sudo e2fsck /dev/sdc -y

# powershell中关闭wsl
wsl --shutdown

# 重新进入wsl
```



## 设置root密码

```shell
sudo passwd
```



## 新建用户

```shell
# 创建用户
sudo adduser howie
# 将用户添加到sudo组
sudo adduser howie sudo
# 打印用户和用户组信息
id howie

# 删除用户
sudo deluser howie
```



## WSL2支持镜像网络模式和硬盘空间自动回收

更新说明: https://devblogs.microsoft.com/commandline/windows-subsystem-for-linux-september-2023-update/

### 镜像网络模式

镜像网络模式可以让WSL2 和 Windows 主机的网络互通而且 IP 地址相同，还支持 IPv6，并且从外部（比如局域网）可以同时访问 WSL2 和 Windows 的网络

在 `%userprofile%\.wslconfig` 中写入以下内容然后保存

```
[experimental]
autoMemoryReclaim=gradual # 可以在 gradual 、dropcache 、disabled 之间选择
networkingMode=mirrored
dnsTunneling=true
firewall=true
autoProxy=true
sparseVhd=true
```

使用 `VSCode - WSL` 插件的，建议去 VSCode 设置里把自动端口转发关掉（`Remote: Auto Forward Ports`），避免冲突，因为 WSL2 更新之后新的网络已经是和你的 Windows 使用相同网络了，不再需要端口转发了

### 硬盘空间自动回收

运行 `wsl --manage 发行版名字 --set-sparse true` 启用稀疏 VHD 允许 WSL2 的硬盘空间自动回收

```powershell
# 配置硬盘空间自动回收需要先停掉wsl
wsl --shutdown

# 启用稀疏VHD
wsl --manage Ubuntu --set-sparse true
```

如果你在 WSL 里使用 docker，那需要将 `autoMemoryReclaim` 配置为 `dropcache` 或者 `disabled`，然后在 `/etc/docker/daemon.json` 里添加一句 `"iptables": false` ，否则你可能无法正常使用 docker



## host和wsl通信

如果是桥接网卡或者端口转发等，需要host关闭防火墙，才能进行通信

如果是wsl2镜像网络模式，可以不用关防火墙就能通信



## MobaXterm登录wsl配置

### 安装nerd字体

nerd字体可以很好的显示出neovim的图标效果，个人常用 `HackNerdFontMono-Regular.ttf` 字体，下载地址: https://www.nerdfonts.com/font-downloads

### 会话 WSL-Ubuntu 右键 Edit session - Terminal settings

- 取消「Backspace sends ^H」勾选，勾选后会和vim操作存在冲突
- 「Terminal type」改为 xterm-256color 支持256色
- 「Font settings」- Font 改为「Hack Nerd Font Mono」



## 配置dns

```shell
# 关闭自动生成dns配置
sudo bash -c 'echo "[network]
generateResolvConf = false" >> /etc/wsl.conf'

# 配置dns服务器
sudo rm /etc/resolv.conf; sudo bash -c 'echo "nameserver 114.114.114.114 
nameserver 180.76.76.76" > /etc/resolv.conf'

# 检查是否能正常访问外网
ping www.baidu.com
```



## 配置软件源

### Ubuntu 22.04 jammy

```shell
# 清华大学开源软件镜像站使用帮助 https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/

# 替换为清华源

sudo mv /etc/apt/sources.list{,.bak}

sudo bash -c 'echo "# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse

deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
# # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
" > /etc/apt/sources.list'

# 更新软件源
sudo apt update

# 更新可升级的软件
sudo apt upgrade
```

### Ubuntu 24.04 noble DEB822 格式

```shell
# 清华大学开源软件镜像站使用帮助 https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/

# 替换为清华源
# 在 Ubuntu 24.04 之前，Ubuntu 的软件源配置文件使用传统的 One-Line-Style，路径为 /etc/apt/sources.list；从 Ubuntu 24.04 开始，Ubuntu 的软件源配置文件变更为 DEB822 格式，路径为 /etc/apt/sources.list.d/ubuntu.sources

sudo mv /etc/apt/sources.list.d/ubuntu.sources{,.bak}

sudo bash -c 'echo "Types: deb
URIs: https://mirrors.tuna.tsinghua.edu.cn/ubuntu
Suites: noble noble-updates noble-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
# Types: deb-src
# URIs: https://mirrors.tuna.tsinghua.edu.cn/ubuntu
# Suites: noble noble-updates noble-backports
# Components: main restricted universe multiverse
# Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

# 以下安全更新软件源包含了官方源与镜像站配置，如有需要可自行修改注释切换
Types: deb
URIs: http://security.ubuntu.com/ubuntu/
Suites: noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

# Types: deb-src
# URIs: http://security.ubuntu.com/ubuntu/
# Suites: noble-security
# Components: main restricted universe multiverse
# Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

# 预发布软件源，不建议启用

# Types: deb
# URIs: https://mirrors.tuna.tsinghua.edu.cn/ubuntu
# Suites: noble-proposed
# Components: main restricted universe multiverse
# Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

# # Types: deb-src
# # URIs: https://mirrors.tuna.tsinghua.edu.cn/ubuntu
# # Suites: noble-proposed
# # Components: main restricted universe multiverse
# # Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
" > /etc/apt/sources.list.d/ubuntu.sources'

# 更新软件源
sudo apt update

# 更新可升级的软件
sudo apt upgrade
```



## 生成ssh密钥

```
ssh-keygen
```



## 安装neovim

https://github.com/neovim/neovim/tags

下载nvim-linux64.tar.gz，不建议用appimage版本，有些运行文件如会话等会保存在临时目录中，恢复程序和会话的插件无法正常工作

```shell
wget https://github.com/neovim/neovim/releases/download/stable/nvim-linux64.tar.gz
tar -zxvf nvim-linux64.tar.gz

# 只给当前用户安装
mv nvim-linux64 ~/.local/share/nvim-linux64
mkdir -p ~/.local/bin
ln -s ~/.local/share/nvim-linux64/bin/nvim ~/.local/bin/nvim

# 给所有用户安装
sudo mv nvim-linux64 /usr/local/share/nvim-linux64
sudo chown -R root:root /usr/local/share/nvim-linux64
sudo ln -s /usr/local/share/nvim-linux64/bin/nvim /usr/local/bin/nvim

# WSL运行appimage二进制依赖libfuse2
sudo apt install libfuse2
```



## 安装pip3

ubuntu22.04自带Python 3.10.12，但没有pip，需要手动安装

ubuntu24.04自带Python 3.12.3，也没有pip，需要手动安装

```shell
sudo apt install python3-pip
```



## 配置pip源

```shell
# 设置pip使用清华源
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
# 设置信任host，当pip源为http网站时使用，https可以不设置
pip config set install.trusted-host pypi.tuna.tsinghua.edu.cn
# 超时时间，单位s，默认15s，当下载较大的包时，需要将超时设置大一些
pip config set global.timeout 600

# root用户配置
sudo pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
sudo pip config set install.trusted-host pypi.tuna.tsinghua.edu.cn
sudo pip config set global.timeout 600

# 普通用户配置
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
pip config set install.trusted-host pypi.tuna.tsinghua.edu.cn
pip config set global.timeout 600
```



## 配置wget代理

`vim ~/.wgetrc` 添加

```
use_proxy = on
https_proxy = http://127.0.0.1:7890
http_proxy = http://127.0.0.1:7890
ftp_proxy = http://127.0.0.1:7890
no_proxy = "localhost,127.0.0.1,localaddress,.localdomain.com"
check-certificate = off
```



## 配置curl代理

`vim ~/.curlrc` 添加

```
proxy="http://127.0.0.1:7890"
insecure
```



## 升级git

```shell
sudo add-apt-repository ppa:git-core/ppa
sudo apt-get update
sudo apt-get install git
git --version
```



## 配置git

```shell
git config --global user.name "howie"
git config --global user.email "howie.wei@outlook.com"

git config --global core.editor nvim

# 提交检出时均不转换，保持原样
git config --global core.autocrlf false
# 拒绝提交包含混合换行符的文件
git config --global core.safecrlf true

# 关闭ssl验证
git config --global http.sslVerify false

# 设置代理走clash
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890

# 取消git代理
git config --global --unset http.proxy
git config --global --unset https.proxy

# 查看所有全局设置
git config --global --list
```



## 安装git-delta

https://github.com/dandavison/delta/releases

```shell
GIT_DELTA_VERSION=$(curl -s "https://api.github.com/repos/dandavison/delta/releases/latest" | grep -Po '"tag_name": "\K[^"]*')
wget "https://github.com/dandavison/delta/releases/download/${GIT_DELTA_VERSION}/git-delta_${GIT_DELTA_VERSION}_amd64.deb"
sudo dpkg -i git-delta_${GIT_DELTA_VERSION}_amd64.deb
```



## 安装lazygit

https://github.com/jesseduffield/lazygit

```shell
LAZYGIT_VERSION=$(curl -s "https://api.github.com/repos/jesseduffield/lazygit/releases/latest" | grep -Po '"tag_name": "v\K[^"]*')
curl -Lo lazygit.tar.gz "https://github.com/jesseduffield/lazygit/releases/latest/download/lazygit_${LAZYGIT_VERSION}_Linux_x86_64.tar.gz"
tar xf lazygit.tar.gz lazygit
sudo install lazygit /usr/local/bin
sudo rm -rf lazygit lazygit.tar.gz
```



### 自定义pagers

https://github.com/jesseduffield/lazygit/blob/master/docs/Custom_Pagers.md

打开lazygit，按h移动到Status窗口，按e进入配置文件编辑界面，复制以下配置

```yaml
gui:
  sidePanelWidth: 0.2 # gives you more space to show things side-by-side

git:
  paging:
    colorArg: always
    pager: delta --dark --paging=never --line-numbers --side-by-side
```



## 安装基础软件

```shell
sudo apt install zsh tmux ripgrep fd-find xsel python3-venv unzip
```



## 安装oh-my-zsh

```shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
# 或
sh -c "$(wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
```



## 安装oh-my-zsh主题powerlevel10k

```shell
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k

# 修改ZSH_THEME
sed -i 's/^ZSH_THEME=.*/ZSH_THEME="powerlevel10k\/powerlevel10k"/g' ~/.zshrc

# 加载配置文件，使主题生效，根据向导提示，选择主题的各种图标显示效果
source ~/.zshrc
```



## 配置tmux

`vim .tmux.conf`

```
# 修改前缀键为 C-a
set -g prefix C-a
unbind C-b
bind C-a send-prefix

# 重新绑定分屏快捷键
# | 向右分屏
unbind %
bind | split-window -h -c "#{pane_current_path}"

# - 向下分屏
unbind '"'
bind - split-window -v -c "#{pane_current_path}"

# <prefix>r 加载配置文件
unbind r
bind r source-file ~/.tmux.conf

# <prefix>h/j/k/l 调整窗格大小
bind -r j resize-pane -D 2
bind -r k resize-pane -U 2
bind -r l resize-pane -R 2
bind -r h resize-pane -L 2

# 进入zoom同时开启copy-mode，退出zoom同时关闭copy-mode
bind a if-shell -F "#{window_zoomed_flag}"    'resize-pane -Z; copy-mode -q'    'resize-pane -Z; copy-mode'

# 开启鼠标支持
set -g mouse on
# 设置前缀键等待时间
set -sg escape-time 10 # make delay shorter

# 设置默认shell
set -g default-shell /usr/bin/zsh
# 设置终端
set -g default-terminal "screen-256color"
# 设置终端颜色属性
set-option -sa terminal-features ',xterm-256color:RGB'
# 低版本没有terminal-features选项，可以用terminal-overrides
# set-option -ga terminal-overrides ',xterm-256color:Tc'


# 插件管理器tpm
set -g @plugin 'tmux-plugins/tpm'

# 自动调用外部工具如xsel，xclip等，共享tmux与系统剪切板
set -g @plugin 'tmux-plugins/tmux-yank'
# Copy Mode 使用vi操作
set -g mode-keys vi
# Copy Mode 按v进入选择模式，按y配置选择内容
bind-key -T copy-mode-vi 'v' send -X begin-selection # start selecting text with "v"
bind-key -T copy-mode-vi 'y' send -X copy-selection # copy text with "y"

# 导航插件，实现ctrl + h/j/k/l 在窗格之间移动，并提供接口，在vim中配置好以后，可以实现vim，tmux窗口无缝跳转
set -g @plugin 'christoomey/vim-tmux-navigator'

# 主题包插件
set -g @plugin 'jimeh/tmux-themepack'
# powerline主题
set -g @themepack 'powerline/block/cyan'

# 保存和恢复tmux会话
set -g @plugin 'tmux-plugins/tmux-resurrect'
# 保存窗格布局
set -g @resurrect-capture-pane-contents 'on'
# 快捷键
# prefix + Ctrl-s - save
# prefix + Ctrl-r - restore

# 定时保存，自动加载会话，依赖tmux-resurrect插件
set -g @plugin 'tmux-plugins/tmux-continuum'
# 默认15分钟保存一次会话
set -g @continuum-save-interval '15'
# 开启tmux启动时自动恢复会话
set -g @continuum-restore 'on'
# 自动恢复nvim会话，只能支持原生的Session.vim会话文件
# set -g @resurrect-strategy-nvim 'session'

# lazyvim建议开启focus-events
set -g focus-events 'on'

run '~/.tmux/plugins/tpm/tpm' # Initialize TPM
```

安装插件管理器tpm

https://github.com/tmux-plugins/tpm

```shell
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

tmux补全

https://github.com/imomaliev/tmux-bash-completion

下载completions/tmux，复制到`/usr/share/bash-completion/completions/`目录中

```shell
wget https://raw.githubusercontent.com/imomaliev/tmux-bash-completion/master/completions/tmux
sudo mv tmux /usr/share/bash-completion/completions/
sudo chown root:root /usr/share/bash-completion/completions/tmux
sudo chmod 644 /usr/share/bash-completion/completions/tmux
```

运行 `tmux`

```shell
# 安装插件
<prefix>I
```

常用操作

```shell
# 创建session
tmux new-session -s startup

# detach session
<prefix>d

# 关闭window，只剩一个window时关闭session
<c-d>

# attach session
tmux attach-session -t startup

# 下方创建窗格
<prefix>-

# 右方创建窗格
<prefix>|

# 调整窗格高度
<prefix>j
<prefix>k

# 最大化当前窗格
<prefix>z

# 在窗口之间跳转
<c-j>
<c-k>
<c-h>
<c-l>

# 进入copy mode
<prefix>[
# 退出copy mode
q 或者 shift-a 或者 ctrl-c

# 重命名会话session
<prefix>$

# 重命名窗口window
<prefix>,

# 创建窗口
<prefix>c

# 选择窗口
<prefix>w

# 加载配置，该快捷键在.tmux.conf中配置
<prefix>r
```

完整操作

https://tmuxcheatsheet.com/



## 安装virtualenvwrapper

https://virtualenvwrapper.readthedocs.io/en/latest/install.html

```shell
sudo pip3 install virtualenvwrapper

# Ubuntu24.04版本会有告警提示，但是virtualenvwrapper本来就应该在全局环境安装的，所以指定--break-system-packages进行安装
sudo pip3 install virtualenvwrapper --break-system-packages

which virtualenvwrapper.sh
# /usr/local/bin/virtualenvwrapper.sh

which python3
# /usr/bin/python3

which virtualenv
# /usr/local/bin/virtualenv

vim ~/.zshrc
# 末尾添加
if [ -f /usr/local/bin/virtualenvwrapper.sh ]; then
       export WORKON_HOME=$HOME/.virtualenvs
       export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
       export VIRTUALENVWRAPPER_VIRTUALENV=/usr/local/bin/virtualenv
       source /usr/local/bin/virtualenvwrapper.sh
fi

source ~/.zshrc

# 创建虚拟环境
mkvirtualenv -p python3 nvim_venv

# 查看虚拟环境
workon

# 进入虚拟环境
workon nvim_venv

# 安装pynvim
pip install pynvim

# 退出虚拟环境
deactivate
```



## 安装nodejs

https://nodejs.org/en/download

下载**Linux Binaries** **(x64)**

```shell
sudo rm -rf /opt/node* /usr/local/bin/node /usr/local/bin/npm

NODDJS_VERSION=$(curl -s "https://api.github.com/repos/nodejs/node/releases/latest" | grep -Po '"tag_name": "v\K[^"]*')
wget "https://nodejs.org/dist/v${NODDJS_VERSION}/node-v${NODDJS_VERSION}-linux-x64.tar.xz"
tar -xvf node-v${NODDJS_VERSION}-linux-x64.tar.xz
sudo mv node-v${NODDJS_VERSION}-linux-x64 /opt/
sudo chown -R root:root /opt/node-v${NODDJS_VERSION}-linux-x64
sudo ln -s /opt/node-v${NODDJS_VERSION}-linux-x64/bin/node /usr/local/bin/
sudo ln -s /opt/node-v${NODDJS_VERSION}-linux-x64/bin/npm /usr/local/bin/
node -v
npm -v
```



### 配置npm/yarn源

```shell
# 查看npm源
sudo npm config get registry
# npm更换淘宝源
sudo npm config set registry https://registry.npmmirror.com
# 恢复官方源
sudo npm config set registry https://registry.npmjs.org

# 安装yarn
sudo npm install -g yarn
sudo rm /usr/local/bin/yarn
sudo ln -s `whereis yarn | cut -d' ' -f2` /usr/local/bin/

# 查看yarn源
sudo yarn config get registry
# yarn更换淘宝源
sudo yarn config set registry https://registry.npmmirror.com
# yarn恢复官方源
sudo yarn config set registry https://registry.yarnpkg.com


# 普通用户更换淘宝源
npm config set registry https://registry.npmmirror.com
yarn config set registry https://registry.npmmirror.com

# 强制清除缓存
npm cache clean --force
# 验证缓存文件夹的内容
npm cache verify
```



### npm安装neovim包

```shell
sudo npm install -g neovim
```



## 安装lazyvim

https://www.lazyvim.org/installation

```shell
# 备份
mv ~/.config/nvim{,.bak}
mv ~/.local/share/nvim{,.bak}
mv ~/.local/state/nvim{,.bak}
mv ~/.cache/nvim{,.bak}

# 清理
yes | rm -rf ~/.config/nvim{,.bak}
yes | rm -rf ~/.local/share/nvim{,.bak}
yes | rm -rf ~/.local/state/nvim{,.bak}
yes | rm -rf ~/.cache/nvim{,.bak}

# 下载官方提供的配置
git clone https://github.com/LazyVim/starter ~/.config/nvim
rm -rf ~/.config/nvim/.git

# 下载自己的配置
git clone git@github.com:gemhowie/nvim.git ~/.config/nvim

# 恢复配置文件后启动
nvim
```



### 配置keymaps

`vim ~/.config/nvim/lua/config/keymaps.lua`

```lua
local keymap = vim.keymap

keymap.set("i", "jk", "<Esc>")
keymap.set("i", "kj", "<Esc>")
keymap.set("n", "<leader>tt", "<cmd>ToggleTerm<cr>")

-- vim-tmux-navigator
if os.getenv("TMUX") then
  keymap.set("n", "<C-h>", "<cmd>TmuxNavigateLeft<cr>")
  keymap.set("n", "<C-j>", "<cmd>TmuxNavigateDown<cr>")
  keymap.set("n", "<C-k>", "<cmd>TmuxNavigateUp<cr>")
  keymap.set("n", "<C-l>", "<cmd>TmuxNavigateRight<cr>")
end

vim.api.nvim_set_keymap("n", "j", "<Plug>(accelerated_jk_gj)", {})
vim.api.nvim_set_keymap("n", "k", "<Plug>(accelerated_jk_gk)", {})
vim.api.nvim_set_keymap("n", "w", "<CMD>lua require'accelerated-jk'.move_to('w')<CR>", {})
vim.api.nvim_set_keymap("n", "e", "<CMD>lua require'accelerated-jk'.move_to('e')<CR>", {})
vim.api.nvim_set_keymap("n", "b", "<CMD>lua require'accelerated-jk'.move_to('b')<CR>", {})
```



### 配置options

`vim ~/.config/nvim/lua/config/options.lua`

```lua
local g = vim.g
local opt = vim.opt
g.tmux_navigator_no_mappings = 1
opt.relativenumber = false
opt.undofile = false
opt.undolevels = 1000
g.python3_host_prog = "~/.virtualenvs/nvim_venv/bin/python3"
opt.clipboard = "unnamedplus"
```



### 配置colorscheme

`vim ~/.config/nvim/lua/plugins/colorscheme.lua`

```lua
return {
  {
    "folke/tokyonight.nvim",
    lazy = true,
    opts = { style = "storm" },
  },
}
```



### 配置treesitter

`vim ~/.config/nvim/lua/plugins/treesitter.lua`

```lua
return {
  {
    "nvim-treesitter/nvim-treesitter",
    -- opts = function()
    --   require("nvim-treesitter.install").prefer_git = true
    -- end,
    opts = {
      install = { prefer_git = true },
      incremental_selection = {
        enable = true,
        keymaps = {
          init_selection = "vv",
          node_incremental = "v",
          scope_incremental = false,
          node_decremental = "V",
        },
      },
    },
  },
}
```



### 配置venv-selector

`vim ~/.config/nvim/lua/plugins/venv-selector.lua`

```lua
return {
  {
    "linux-cultist/venv-selector.nvim",
    dependencies = {
      "neovim/nvim-lspconfig",
      "mfussenegger/nvim-dap",
      "mfussenegger/nvim-dap-python", --optional
      { "nvim-telescope/telescope.nvim", branch = "0.1.x", dependencies = { "nvim-lua/plenary.nvim" } },
    },
    lazy = false,
    branch = "regexp", -- This is the regexp branch, use this for the new version
    config = function()
      require("venv-selector").setup()
    end,
    keys = {
      { "<leader>cv", "<cmd>VenvSelect<cr>", desc = "Select VirtualEnv" },
    },
  },
}
```



### 配置nvim-lint

`vim ~/.config/nvim/lua/plugins/nvim-lint.lua`

```lua
return {
  {
    "mfussenegger/nvim-lint",
    opts = {
      -- Event to trigger linters
      events = { "BufWritePost", "BufReadPost", "InsertLeave" },
      linters_by_ft = {
        python = { "mypy" },
        -- fish = { "fish" },
        -- Use the "*" filetype to run linters on all filetypes.
        -- ['*'] = { 'global linter' },
        -- Use the "_" filetype to run linters on filetypes that don't have other linters configured.
        -- ['_'] = { 'fallback linter' },
      },
      -- LazyVim extension to easily override linter options
      -- or add custom linters.
      ---@type table<string,table>
      linters = {
        -- -- Example of using selene only when a selene.toml file is present
        -- selene = {
        --   -- `condition` is another LazyVim extension that allows you to
        --   -- dynamically enable/disable linters based on the context.
        --   condition = function(ctx)
        --     return vim.fs.find({ "selene.toml" }, { path = ctx.filename, upward = true })[1]
        --   end,
        -- },
      },
    },
  },
}
```



### 配置Supertab

`vim ~/.config/nvim/lua/plugins/nvim-cmp.lua`

```lua
return {
  {
    "hrsh7th/nvim-cmp",
    dependencies = { "hrsh7th/cmp-emoji" },
    ---@param opts cmp.ConfigSchema
    opts = function(_, opts)
      local has_words_before = function()
        unpack = unpack or table.unpack
        local line, col = unpack(vim.api.nvim_win_get_cursor(0))
        return col ~= 0 and vim.api.nvim_buf_get_lines(0, line - 1, line, true)[1]:sub(col, col):match("%s") == nil
      end
      table.insert(opts.sources, { name = "emoji" })

      local cmp = require("cmp")

      opts.mapping = vim.tbl_extend("force", opts.mapping, {
        ["<Tab>"] = cmp.mapping(function(fallback)
          if cmp.visible() then
            -- You could replace select_next_item() with confirm({ select = true }) to get VS Code autocompletion behavior
            cmp.select_next_item()
          elseif vim.snippet.active({ direction = 1 }) then
            vim.schedule(function()
              vim.snippet.jump(1)
            end)
          elseif has_words_before() then
            cmp.complete()
          else
            fallback()
          end
        end, { "i", "s" }),
        ["<S-Tab>"] = cmp.mapping(function(fallback)
          if cmp.visible() then
            cmp.select_prev_item()
          elseif vim.snippet.active({ direction = -1 }) then
            vim.schedule(function()
              vim.snippet.jump(-1)
            end)
          else
            fallback()
          end
        end, { "i", "s" }),
      })
    end,
  },
}
```



### 配置vim-tmux-navigation

`vim ~/.config/nvim/lua/plugins/vim-tmux-navigation.lua`

```lua
return {
  {
    "christoomey/vim-tmux-navigator",
  },
}
```

`vim ~/.config/nvim/lua/config/keymaps.lua` 添加

```lua
-- vim-tmux-navigator
if os.getenv("TMUX") then
  keymap.set("n", "<C-h>", "<cmd>TmuxNavigateLeft<cr>")
  keymap.set("n", "<C-j>", "<cmd>TmuxNavigateDown<cr>")
  keymap.set("n", "<C-k>", "<cmd>TmuxNavigateUp<cr>")
  keymap.set("n", "<C-l>", "<cmd>TmuxNavigateRight<cr>")
end
```



### 配置accelerated-jk

https://github.com/rainbowhxch/accelerated-jk.nvim

`vim ~/.config/nvim/lua/plugins/accelerated-jk.lua`

```lua
return {
  {
    "rainbowhxch/accelerated-jk.nvim",
  },
}
```

`vim ~/.config/nvim/lua/config/keymaps.lua` 添加

```lua
vim.api.nvim_set_keymap("n", "j", "<Plug>(accelerated_jk_gj)", {})
vim.api.nvim_set_keymap("n", "k", "<Plug>(accelerated_jk_gk)", {})
vim.api.nvim_set_keymap("n", "w", "<CMD>lua require'accelerated-jk'.move_to('w')<CR>", {})
vim.api.nvim_set_keymap("n", "e", "<CMD>lua require'accelerated-jk'.move_to('e')<CR>", {})
vim.api.nvim_set_keymap("n", "b", "<CMD>lua require'accelerated-jk'.move_to('b')<CR>", {})
```



### 配置conform

`vim ~/.config/nvim/lua/plugins/conform.lua` 添加

```lua
return {
  {
    "stevearc/conform.nvim",
    opts = {
      format = {
        timeout_ms = 10000,
      },
    },
  },
}
```



### 配置mason安装列表

`vim ~/.config/nvim/lua/plugins/mason.lua`

```lua
return {
  {
    "williamboman/mason.nvim",
    opts = {
      ensure_installed = {
        "stylua",
        "shfmt",
        "json-lsp",
        "lua-language-server",
        "pyright",
        "ruff-lsp",
        "black",
        "mypy",
        "clangd",
        "clang-format",
      },
    },
  },
}
```



### 配置LazyExtras安装列表

`vim ~/.config/nvim/lazyvim.json`

```json
{
  "extras": [
    "lazyvim.plugins.extras.dap.core",
    "lazyvim.plugins.extras.editor.aerial",
    "lazyvim.plugins.extras.formatting.black",
    "lazyvim.plugins.extras.lang.python"
  ],
  "news": {
    "NEWS.md": "5204"
  },
  "version": 6
}
```



## lazyvim使用技巧

### 搜索文件telescope-fzf-native

| Token     | Match type                 | Description                          |
| --------- | -------------------------- | ------------------------------------ |
| `sbtrkt`  | fuzzy-match                | Items that match `sbtrkt`            |
| `'wild`   | exact-match (quoted)       | Items that include `wild`            |
| `^music`  | prefix-exact-match         | Items that start with `music`        |
| `.mp3$`   | suffix-exact-match         | Items that end with `.mp3`           |
| `!fire`   | inverse-exact-match        | Items that do not include `fire`     |
| `!^music` | inverse-prefix-exact-match | Items that do not start with `music` |
| `!.mp3$`  | inverse-suffix-exact-match | Items that do not end with `.mp3`    |

