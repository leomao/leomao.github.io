+++
title = "Arch Linux 安裝筆記"
date = 2017-09-17T18:19:52+08:00
slug = "archlinux-install-note"
categories = []
tags = ["linux"]
+++

# 前言

感覺近年來(?)裝了無數次的 Arch Linux，最近安裝都已經幾乎不需要看 [Installation Guide][install-guide] 了。於是決定來把現在我最常用的安裝流程記錄一下。

整體來說跟 [Installation Guide][install-guide] 結構一樣，只是稍微限縮在一些我最常碰到的狀況下：

- [前置作業]({{< ref "archlinux-install-note.md#前置作業" >}})
- [安裝套件]({{< ref "archlinux-install-note.md#安裝套件" >}})
- [設定系統]({{< ref "archlinux-install-note.md#設定系統" >}})
- [完成之後]({{< ref "archlinux-install-note.md#完成之後" >}})

<!--more-->

# 前置作業

> 下面的安裝流程都是假設要安裝一台個人用的電腦，所以都會裝 GUI 介面之類的東西。  
> 另外，也都假設電腦足夠新，是用 UEFI 的方式開機的。並且是 Intel 的 CPU。

## 開機碟
首先當然是要弄一個 Arch Linux 的開機碟，製作方法就參照 [USB flash installation media](https://wiki.archlinux.org/index.php/USB_flash_installation_media)。但簡單來說，如果已經有一個 GNU/Linux 的環境，那就用 `dd`，如果是 Windows 的話，可以考慮用 [Rufus](https://rufus.akeo.ie/) 或是 [USBwriter](https://sourceforge.net/p/usbwriter/wiki/Documentation/)。
弄好之後就用開機碟開機 (記得要用 UEFI mode)。

不想把整個隨身碟洗掉的話，其實也可以自己切好足夠大的分割區 (大概 600 MB) 並格式化成 FAT32，然後把 ISO (解開或掛載起來) 裡面的東西丟進去即可。

## 網路
這邊還滿多 cases 的，無線網路的話可以用 `wifi-menu`，有線的話分成兩個 cases：

- 有 dhcp：那應該什麼都不用做，`curl google.com` 確認一下有沒有連到網路就是。
- 手動 IP：那就需要寫寫設定然後用 `netctl` 來連網路。步驟如下：
  1. 用 `ip link` 看看網卡的 interface name，有線網路卡的名稱應該都是 "e" 開頭的，下面假設成 `enp3s0` 好了。
  1. `cd /etc/netctl` 之後 `cp examples/ethernet-static ./`。
  1. 用 `vim ethernet-static` 打開設定檔，修改下面的東西然後存檔
    - `Interface=enp3s0`
    - `Address=('xxx.xxx.xxx.xxx/24')`：子網路遮罩 `255.255.255.0` 即是 `/24`。
    - `Gateway='xxx.xxx.xxx.xxx'` 這就不用解釋了吧
    - `DNS=('8.8.8.8')` 我通常都直接用 google DNS
  1. `ip link set enp3s0 down` 把 interface 關掉。
  1. `netctl start ethernet-static`，最後當然還是檢查一下是否有連上網路。

## 分割區

這邊也是有各種狀況，先用 `lsblk` 看一下硬碟的分割情況。每個分割區的代號都是 `/dev/sda1`、`/dev/sdb2` 這種樣子。最簡單的情況就是已經是 GPT 的分割表，那就用 `cgdisk` 切成自己想要的樣子。然後再用 `mkfs` 把分割區格式化。幾點備忘：

- EFI partition 要用 FAT32 格式化 `mkfs.fat -F32 /dev/sdxY`
- Swap 要用 `mkswap /dev/sdxY`
- 其他都可以用 `mkfs.ext4 /dev/sdxY` 來格式化
- **如果要 dual boot 且已經裝了 windows**，那 EFI parition 應該已經存在，就不用動它也不用多切一塊了。
- 如果沒有 GPT 分割表，可以用 `gdisk` 建一個新的。註：似乎也可以轉換 MBR 之類的，但我好像沒試過。

格式化完就可以把分割區一個個掛起來 (`mount /dev/sdxY /path/to/mountpoint`)：

1. 先把 root 分割區掛到 `/mnt` 上
2. `mkdir /mnt/boot` 之後把 EFI parition 掛上去
3. `swapon /dev/sdxY` 把 swap 啟用
4. 其他 `/var`、`/home` 之類的就看需要掛載吧

# 安裝套件

首先要選個 mirror 才夠快，在台灣的話我覺得用海大或是淡大好像都還算是好選擇。  
`vim /etc/pacman.d/mirrorlist` 進去後把有 `ntou` 的那個網址移到檔案最上方存檔就行了。
(註：其他台灣的 mirror 也可以試試，交大的 mirror 很快但有時候會好幾天不更新...OTL)

`pacstrap` 簡單來說就是把某個位置當成根目錄然後把後面給的套件全部裝進去，官方的安裝流程是直接 `pacstrap /mnt base`，不過我通常都習慣在這個時候就把我會用到的套件一次性裝好。主要是常用的 tools 跟 DE 還有字體之類的。

```bash
pacstrap /mnt base base-devel \
intel-ucode \ # intel CPU 的話
zsh tmux git openssh rsync sshfs neovim gvim \
python python-pip python-neovim xclip \
htop exa ripgrep the_silver_searcher \ # 上面幾行都是一些常用的 tools 相關的東西
nodejs npm yarn rust go go-tools \ # 程式語言相關
ipython python-numpy python-scipy python-matplotlib python-scikit-learn \ # python 常用
gnome gnome-tweak-tool \ # DE 我現在主要都用 gnome shell
cups system-config-printer hplip \ # 列印相關 (hplip for HP 的印表機)
noto-fonts noto-fonts-cjk adobe-source-code-pro-fonts \ # 字體
nvidia cuda cudnn cudnn6 \ # 如果有 GPU 要用 CUDA 的話
firefox # 雖然我平常主要是用 google chrome ...
```

# 設定系統

在 chroot 之前，`genfstab -U /mnt | sed -e 's/relatime/noatime/g' >> /mnt/etc/fstab`。
然後就可以 `arch-chroot /mnt` 進去裡面做設定了。Chroot 進去後要做的事情可以說是非常 routine，於是我就不解釋直接寫成 script 的樣子了...

```bash
# 語系
sed -i -e 's/^#\(en_US\|zh_TW\)\(\.UTF-8\)/\1\2/g' /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
# 時間
export TIMEZONE=Asia/Taipei # 台北時區
ln -sf /usr/share/zoneinfo/${TIMEZONE} /etc/localtime
hwclock --systohc
systemctl enable systemd-timesyncd
# hostname 相關
export HOSTNAME=<hostname> # 給電腦取個名字
echo $HOSTNAME > /etc/hostname
sed -ie "8i 127.0.1.1\t$HOSTNAME.localdomain\t$HOSTNAME" /etc/hosts
# 一些需要開機做的事
systemctl enable fstrim.timer # 好像是有 SSD 才需要
systemctl enable NetworkManager
systemctl enable gdm
sed -ie 's/#\(WaylandEnable\)/\1/' /etc/gdm/custom.conf # Wayland 還不太穩...
systemctl enable cups-browsed # 如果要用 printer
# boot loader
bootctl install
cat > /boot/loader/loader.conf << EOF
default	arch
timeout	3
editor	0
EOF
cat > /boot/loader/entries/arch.conf << EOF
title	Arch Linux
linux	/vmlinuz-linux
initrd	/intel-ucode.img
initrd	/initramfs-linux.img
options root=PARTUUID=<PARTUUID> rw
EOF
# assume root partition is /dev/sdxY
export PARTUUID=$(blkid -s PARTUUID -o value /dev/sdxY)
sed -ie "s/<PARTUUID>/${PARTUUID}/" /boot/loader/entries/arch.conf
# sudo
sed -ie 's/# \(%wheel ALL=(ALL) ALL\)/\1/' /etc/sudoers
```

最後就是弄一個 user 出來：

```
export USERNAME=<username> # 你的 username
useradd -mG wheel,storage,power,video,audio $USERNAME
passwd $USERNAME # 修改密碼
```

搞定之後就可以 `exit` 離開 chroot 環境，然後 `umount -R /mnt` 並重開機了 =D

## 完成之後

全都搞定之後，我通常還會裝個 `pacaur` 方便裝一些 AUR 裡面的東西，記錄一下流程：

```bash
gpg --recv-keys --keyserver hkp://pgp.mit.edu 1EB2638FF56C0C53
bash <(curl aur.sh) -si cower pacaur
rm -r cower pacaur # cleanup
```

其他雜項我就隨便列在下面了：

- 佈景主題和桌面環境 (用 `gnome-tweak-tool` 改設定)：
  - [Materia theme](https://github.com/nana-4/materia-theme) 還不錯 (記得要啟用 Extensions 裡面的 User Themes)
  - [paper icon theme](https://github.com/snwh/paper-icon-theme) 的圖示也滿好看的
  - 推薦 [Topicons Plus](https://extensions.gnome.org/extension/1031/topicons/) 這個 gnome shell extention 不錯用
  - 其他 Dash to X 系列的 extensions 可以自己選自己喜歡的XD
- 輸入法我是用 ibus-rime，不過因為我是用嘸蝦米，沒太大的參考價值
- Tilix 是一個滿好看的 terminal，目前在 AUR 裡面，可以裝 `tilix-bin`
- 要用 Docker 的話就 `pacaur -S docker` 並 `systemctl enable docker`
- 如果有在用 numpy 之類的話，裝個 AUR 裡的 `openblas-lapack` 效能會好很多
- 有時候會用到一些其他 filesystem，可以裝個 `dosfstools`, `ntfs-3g`, `exfat-utils`

# 後記

重新把之前沒完成的安裝 scripts 寫了一下，現在放在 [arch-bootstrap](https://github.com/leomao/arch-bootstrap)，不過我還沒測試過就是了。之後測試一下或是想到什麼再來補完好了。

[install-guide]: https://wiki.archlinux.org/index.php/installation_guide
[modeline]: # ( vim: set cc=0 tw=0: )
