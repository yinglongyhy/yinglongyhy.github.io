# manjaro xfce 重装要点

> 使用pacaur时，建议先用C-c撤销，使用pacman下载软件库列表的软件，在重新用pacaur下载

- ## 更换源

  `sudo pacman-mirrors -i -c China -m rank`，选择2-3个快速的源。

  在`/etc/pacman.conf`最后添加一下几行

  ```shell
  ## 浙江大学 (浙江杭州) (ipv4, ipv6, http, https)
  ## Added: 2017-06-05
  ## [archlinuxcn]
  ## Server = https://mirrors.zju.edu.cn/archlinuxcn/$arch
  ## 中国科学技术大学 (ipv4, ipv6, http, https)
  [archlinuxcn]
  Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
  ## 清华大学 (ipv4, ipv6, http, https)
  ## [archlinuxcn]
  ## Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
  ```

  然后`sudo pacman -Syyu`更新系统

- ## 安装必备软件

  - #### 坚果云

    `sudo pacman -S nutstore`

  - #### vim 和 youcompleteme

    `sudo pacman -S gvim`

    把`.vimrc`、`.ideavimrc`和`.vim`放在`home`目录

    追加`.vimrc`到`/etc/vimrc`

    先备份`sudo cp /etc/vimrc /etc/vimrc.bak`

    切换到管理员`sudo su`

    `cat .vimrc >> /etc/vimrc`

    `sudo pacman -S vim-youcompleteme-git`

    安装molokai主题

    `sudo pacman -S vim-molokai`

    设置vim透明

    ```shell
    " ~/.vimrc
    hi Normal       ctermfg=252     ctermbg=none " 不空白地方透明
    hi NonText      ctermfg=250 	ctermbg=none " 空白地方透明
    ```

  - #### 搜狗输入法（需重启）

    安装fcitx、fcitx-configtool、fcitx-im

    `sudo pacman -S fcitx fcitx-configtool fcitx-im`

    其中`fcitx`默认全选

    安装搜狗拼音

    `sudo pacman -S fcitx-sogoupinyin`

    进入fcitx配置输入法顺序

    配置

    ```shell
    # vim ~/.xprofile
    export GTK_IM_MODULE=fcitx
    export QT_IM_MODULE=fcitx
    export XMODIFIERS="@im=fcitx"
    ```

  - #### shadowsocks 和 proxychains-ng

    安装shadowsocks 和 proxychanis-ng

    `sudo pacman -S shadowsocks`

    复制`yhy.json`到`/etc/shadowsocks/`目录

    后台自启

    `systemctl enable shadowsocks@yhy`

    把`/etc/proxychains.conf`最后面改为`socks5	127.0.0.1 1080`

  - #### aria2-fast（命令行下载工具    aria2的改版）

    安装aria2-fast

    `sudo pacman -S aria2-fast`

    把`.aria2`目录放在`home`目录

  - #### ssh密钥

    1. 原来没有ssh密钥的：

       > 执行 `ssh-keygen -t rsa -C "your_email.com"

    2. 原来有ssh密钥的：

       > 把公钥(id_rsa.pub)和私钥(id_rsa)拷贝到`~/.ssh`目录里
       >
       > 调整权限：公钥644，私钥600， 如下：
       >
       > `chmod 644 ~/.ssh/id_rsa.pub`
       >
       > `chmod 600 ~/.ssh/id_rsa`

  - #### tmux

    把`.tmux`目录放在`home`目录，并把里面的`.tmux.conf`和`.tmux.conf.local`复制到`home`目录

    安装字体

    `sudo pacman -S powerline-fonts`

    `pacaur -S ttf-symbola`

  - #### zsh 和 oh-my-zsh（需重启）

    安装zsh

    `sudo pacman -S zsh`

    设置**zsh**为默认shell

    `chsh -s $(which zsh)`

    安装oh-my-zsh

    `sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"`

    修改zsh主题

    `vim .zshrc`

    把`ZSH_THEME`的值改为`ys`

    在末尾添加

    ```shell
    alias pc='proxychains4'
    alias rs='rsync -avzP --rsh=ssh'
    ```

  - #### chromium

    `sudo pacman -S chromium`

    把插件导入chromium，并从备份文件恢复

  - #### pacaur

    `sudo pacman -S pacaur`

  - #### 网易云音乐

    `sudo pacman -S netease-cloud-music`

    在`qt5ct`把风格改为`Fusion`

  - #### docky

    `sudo pacman -S docky`

  - #### 微软雅黑-consolas字体

    `pacaur -S ttf-consolas-with-yahei`

    设为终端字体

  - #### 微信和Tim

    `sudo pacman -S deepin.com.qq.office`

    `pacaur -S deepin-wechat`

  - #### thunderbird中文包

    `sudo pacman -S thunderbird-i18n-zh-cn`

  - #### timeshift

    `sudo pacman -S timeshift`

  - #### virtualbox

    `sudo pacman -S virtualbox`

    `sudo modprobe vboxdrv`

  - #### libinput-gestures(需重启)

    `sudo pacman -S libinput-gestures`

    把`libinput-gestures.conf`放在`/etc/`目录

    加入用户组

    `sudo gpasswd -a $USER input`

    启动

    `libinput-gestures-setup start`

    开机启动

    把启动命令添加到会话与启动中

  - #### typora

    `sudo pacman -S typora`

  - #### intellij-idea-ultimate

    `sudo pacman -S intellij-idea-ultimate-edition`

  - #### vscode

    `sudo pacman -S visual-studio-code-bin`

  - #### wps

    `sudo pacman -S wps-office`

    字体

    `pacaur -S ttf-wps-fonts`

  - #### docker

    `sudo pacman -S docker`

    把用户加入组

    `sudo gpasswd -a $USER docker`

    换源

    `/etc/docker/daemon.json`

    ```json
    {
      "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
    }
    ```




- ## 设置git

  `git config --global user.name "name"`

  `git config --global user.email "email"`

  `git config --global core.editor vim`



- ## 主题美化

  - #### 安装主题包

  `sudo pacman -S gtk-theme-arc-git`

  `sudo pacman -S numix-circle-icon-theme`

  在**设置-外观**选择主题

  - #### 设置时间

    打开时钟属性-自定义

    `<span font='14' weight='bold'>%Y-%m-%d    <span font='13' weight="bold">%A</span>   %R</span>`

  - #### 设置快捷键

    [这个网址](https://blog.csdn.net/cFarmerReally/article/details/53375956)

  - #### 高分屏字体和图标

    **字体**

    设置编辑器——xsettings——DPI，更改数值

    **图标**

    设置——桌面——图标




