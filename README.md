<!-- =======================================================
  * Template Name: Antares-OS - v2.26
  * Author: José Valdemir de Melo
  ======================================================== -->  

[![NPM](https://img.shields.io/npm/l/react)](https://github.com/valdemir26/valdemir26.github.io/blob/main/LICENSE)
<img src="https://komarev.com/ghpvc/?username=valdemir26&color=yellow" alt="Profile views" />

<p align="center">
   <img src="http://img.shields.io/static/v1?label=STATUS&message=EM%20DESENVOLVIMENTO&color=RED&style=for-the-badge" #vitrinedev/>
</p>

</p>   
    <p align="center">
      <img src="https://github-readme-streak-stats.herokuapp.com/?user=valdemir26&" alt="valdemir26"/>
   </p>  

[![Ashutosh's github activity graph](https://github-readme-activity-graph.vercel.app/graph?username=valdemir26&bg_color=0d1117&color=fff&line=0563bb&point=272829&area=true&hide_border=true)](https://github.com/ashutosh00710/github-readme-activity-graph)


# <p align="center">Crie seu linux do zero com debootstrap

## Como funciona
Optamos por utilizar um processo manual de customização do Linux, utilizando o squashfs, genisoimage e chroot, é simples, você usa a própria distribuição instalada como base, sem aplicativos extras, que as vezes é incompatível dependendo da distribuição.

## Preparando o ambiente
### Para criar o ambiente necessário para customização
Para criar o ambiente necessário para customização
Computador (Desktop ou Notebook) com Linux instalado e com suporte para o "squashfs" no kernel, desde 2006 o linux já possui suporte para o Squashfs no kernel, porém sugerimos utilizar a versão mais atual disponível.

_Como criar sua própria ISO com debootstrap, use o comando para certificar que os pacotes estão presentes em seu sistema_
```bash
sudo apt update && apt -y install \
    debootstrap \
    squashfs-tools \
    genisoimage \
```

## Criando a jaula do sistema
Antes de começarmos a criar a nossa *Distro*, devemos criar o diretório e os subdiretórios que serão nossa área de trabalho \
Para criar o diretório e os subdiretórios, podemos usar o navegador de aquivos ou simplesmente fazer isso no terminal como decrito logo abaixo \
Depois que o diretórios e os subdiretórios estiverem criados, é só seguir o passo a passo \
_Agora vamos criar o diretório que irão conter os arquivos nescessários para fazer o chroot_
```bash
mkdir -p $HOME/Distro/{chroot,antares/{EFI/boot,boot/grub/{theme,x86_64-efi},isolinux,live},files}
cd Distro
```

## Instala o sistema base com debootstrap
_A ferramenta debootstrap irá selecionar os pacotes nescessários da base so sistema escolhido para chroot, é nescessário instalar a chave gpg do sistema escolhido_
### Exemplo 
_debian-archive-keyring_ \
http://deb.debian.org/debian/pool/main/d/debian-archive-keyring/
```bash
sudo debootstrap \
    --arch=amd64 \
    --variant=minbase \
    trixie \
    $HOME/Distro/chroot \
    http://deb.debian.org/debian/
 ```    

 ## Iniciando o chroot
_Copia os arquivos /resolv.conf /hosts da maquina local e monta /dev /proc /sys para configuração de ambiente para uso do chroot_
```bash
sudo cp /etc/resolv.conf chroot/etc/
sudo cp /etc/hosts chroot/etc/
sudo mount --bind /dev chroot/dev
sudo mount --bind /proc chroot/proc
sudo mount --bind /sys chroot/sys
sudo chroot chroot
```

## Adicionando repositório Debian
_Source.list_
```bash
cat > /etc/apt/sources.list << 'EOF'
deb https://deb.debian.org/debian/ trixie contrib main non-free non-free-firmware
# deb-src https://deb.debian.org/debian/ trixie contrib main non-free non-free-firmware
deb https://deb.debian.org/debian/ trixie-updates contrib main non-free non-free-firmware
deb https://deb.debian.org/debian/ trixie-backports contrib main non-free non-free-firmware
# deb-src https://deb.debian.org/debian/ trixie-backports contrib main non-free non-free-firmware
# deb-src https://deb.debian.org/debian/ trixie-updates contrib main non-free non-free-firmware
deb https://deb.debian.org/debian/ trixie-proposed-updates contrib main non-free non-free-firmware
# deb-src https://deb.debian.org/debian/ trixie-proposed-updates contrib main non-free non-free-firmware
deb https://security.debian.org/debian-security/ trixie-security contrib main non-free non-free-firmware
# deb-src https://security.debian.org/debian-security/ trixie-security contrib main non-free non-free-firmware
EOF
```

## Atualizar a lista de pacotes
_Carrega a lista de pacotes para serem atualizados ou instalados_
```bash
apt update -y && apt full-upgrade -y
```

## Pacotes para instalação minima
_A lista de pacotes que eu escolhi, para uma interface limpa mas fica ao critério de cada escolher seus próprios pacotes_
```bash
apt install -y \
apt-transport-https build-essential btrfs-progs curl dbus-x11 dosfstools dkms rsync e2fsprogs exfatprogs \
efibootmgr linux-image-amd64 live-boot live-config squashfs-tools genisoimage isolinux lsb-base grub-common \
grub2-common grub-efi-amd64 grub-efi-amd64-bin wget os-prober gnome-accessibility-themes gnome-disk-utility \
gnome-shell gnome-shell-common gnome-shell-extension-prefs gnome-shell-extensions gnome-software gnome-session \
gnome-tweaks gnome-terminal nautilus mutter mtools gdm3 xinit gnome-control-center xdg-user-dirs-gtk gedit file-roller \
yad calamares calamares-settings-debian
```

## Firmwares
_Instalar os drivers firmware-linux-free e firmware-linux-nonfree, alguns firmware-nonfree é nescessário aceitar os termos para instalação do pacote_
```bash
apt install -y \
firmware-amd-graphics firmware-ast firmware-ath9k-htc firmware-atheros firmware-bnx2 firmware-bnx2x \
firmware-brcm80211 firmware-cavium firmware-intel-sound firmware-ipw2x00 firmware-ivtv firmware-iwlwifi \
firmware-libertas firmware-linux firmware-linux-free firmware-linux-nonfree firmware-misc-nonfree \
firmware-myricom firmware-netronome firmware-netxen firmware-qcom-soc firmware-qlogic firmware-realtek \
firmware-samsung firmware-siano firmware-sof-signed firmware-ti-connectivity
```


## Limpar cache do APT e finalizar o chroot
_Remove os arquivos de configuração usandos no chroot_
```bash    
rm -rf /tmp/* ~/.bash.history
rm /etc/resolv.conf
rm /etc/hosts
apt clean
exit
```

## Desmonta as partições do ambiente de customização
_Desmonta o /dev /proc /sys que também foram montados no chroot_
```bash   
sudo umount -lf chroot/dev
sudo umount -lf chroot/proc
sudo umount -lf chroot/sys
```
## Criando o usuário live do sistema
_Este é o usuário padrão do sistema live_
```bash 
cat > $HOME/Distro/files/antares.conf << 'EOF'
LIVE_USERNAME="antares"
LIVE_USER_FULLNAME="Antares Live User"
EOF
sudo cp $HOME/Distro/files/antares.conf $HOME/Distro/chroot/etc/live/config.conf.d/antares.conf
```

_Agora vamos criar o restante dos arquivos_
```bash
mkdir -p $HOME/Distro/{chroot,antares/{EFI/boot,boot/grub/{theme,x86_64-efi},isolinux,live},files}
```

## README.diskdefines
_Cria um rótulo para a imagem ISO_
```bash
cat > $HOME/Distro/antares/README.diskdefines << 'EOF'
#define DISKNAME   AntaresOS
#define TYPE       binary
#define TYPEbinary 1
#define ARCH       amd64
#define ARCHamd64  1
#define DISKNUM    1
#define DISKNUM1   1
#define TOTALNUM   0
#define TOTALNUM0  1
EOF
```

# <p align="center">Grub</p>
## Arquivos de boot do sistema
_Precisamos agora copiar os arquivos necessários de inicialização para BIOS Legacy para o diretório do LiveCD_
```bash
sudo cp -r $HOME/Distro/chroot/usr/lib/grub/x86_64-efi/* "$HOME/Distro/antares/boot/grub/x86_64-efi/"
```

_Agora, criaremos uma imagem de disco de inicialização FAT16 UEFI contendo o carregador de inicialização EFI_
```bash
cd $HOME/Distro/antares && \
    dd if=/dev/zero of=efi.img bs=1M count=20 && \
    mkfs.vfat efi.img && \
    mmd -i efi.img efi efi/boot && \
    mcopy -vi efi.img $HOME/Distro/files/bootx64.efi ::efi/boot/
cd
cd Distro
```

_Vamos criar uma imagem inicializável para o GRUB EFI_
```bash
grub-mkstandalone \
    --format=x86_64-efi \
    --output=$HOME/Distro/files/bootx64.efi \
    --locales="" \
    --themes="" \
    --fonts="" \
    "boot/grub/grub.cfg=$HOME/Distro/files/grub.cfg"
cp $HOME/Distro/files/bootx64.efi $HOME/Distro/antares/EFI/boot/
cp $HOME/Distro/files/bootx64.efi $HOME/Distro/antares/
cp $HOME/Distro/antares/boot/grub/x86_64-efi/monolithic/grubx64.efi $HOME/Distro/antares/EFI/boot/
cp $HOME/Distro/antares/boot/grub/x86_64-efi/monolithic/grubx64.efi $HOME/Distro/antares/
cp $HOME/Distro/antares/efi.img $HOME/Distro/antares/boot/grub
```

# <p align="center">Arquivos de arranque do sistema</p>
_Criar o config.cfg_
```bash
cat > $HOME/Distro/antares/boot/grub/config.cfg << 'EOF'
set default=0

if [ x$feature_default_font_path = xy ] ; then
    font=unicode
else
    font=$prefix/unicode.pf2
fi

if loadfont $font ; then
    set gfxmode=800x600
    set gfxpayload=keep
    insmod efi_gop
    insmod efi_uga
    insmod video_bochs
    insmod video_cirrus
else
    set gfxmode=auto
    insmod all_video
fi

insmod gfxterm
insmod png

source /boot/grub/theme.cfg

terminal_output gfxterm

insmod play
play 960 440 1 0 4 440 1
EOF
```

_Criar o grub.cfg_
```bash
cat > $HOME/Distro/antares/boot/grub/grub.cfg << 'EOF'
source /boot/grub/config.cfg

# Live boot
menuentry "Antares OS (amd64)" --hotkey=l {
	linux	/live/vmlinuz boot=live locales=pt_BR.UTF-8 keyboard-layouts=pt_BR username=antares hostname=antares autologin findiso=${iso_path}
	initrd	/live/initrd.lz
}
menuentry "Antares OS (amd64 fail-safe mode)" {
	linux	/live/vmlinuz boot=live components memtest noapic noapm nodma nomce nolapic nosmp nosplash vga=788
	initrd	/live/initrd.lz
}
EOF
```

_Criar o loopback.cfg_
```bash
cat > $HOME/Distro/antares/boot/grub/loopback.cfg << 'EOF'
source /boot/grub/grub.cfg
EOF
```

_Criar o theme.cfg_
```bash
cat > $HOME/Distro/antares/boot/grub/theme.cfg << 'EOF'
set color_normal=light-gray/black
set color_highlight=white/dark-gray

if [ -e /isolinux/splash.png ]; then
    set theme=/boot/grub/antares/theme.txt
elif [ -e /boot/grub/splash.png ]; then
    set theme=/boot/grub/antares/theme.txt
else
    set menu_color_normal=cyan/blue
    set menu_color_highlight=white/blue
fi
EOF
```

_Criar o theme.txt_
```bash
cat > $HOME/Distro/antares/boot/grub/theme/theme.txt << 'EOF'
desktop-image: "../splash.png"
title-color: "#ffffff"
title-font: "Unifont Regular 16"
title-text: "Live Boot Menu with GRUB"
message-font: "Unifont Regular 16"
terminal-font: "Unifont Regular 16"

#help bar at the bottom
+ label {
        top = 100%-50
        left = 0
        width = 100%
        height = 20
        text = "@KEYMAP_SHORT@"
        align = "center"
        color = "#ffffff"
	font = "Unifont Regular 16"
}

#boot menu
+ boot_menu {
        left = 10%
        width = 80%
        top = 52%
        height = 48%-80
        item_color = "#a8a8a8"
	item_font = "Unifont Regular 16"
        selected_item_color= "#ffffff"
	selected_item_font = "Unifont Regular 16"
        item_height = 16
        item_padding = 0
        item_spacing = 4
	icon_width = 0
	icon_heigh = 0
	item_icon_space = 0
}

#progress bar
+ progress_bar {
        id = "__timeout__"
        left = 15%
        top = 100%-80
        height = 16
        width = 70%
        font = "Unifont Regular 16"
        text_color = "#000000"
        fg_color = "#ffffff"
        bg_color = "#a8a8a8"
        border_color = "#ffffff"
        text = "@TIMEOUT_NOTIFICATION_LONG@"
}
EOF
```
# <p align="center">Isolinux</p>
_Copiar boot.cat e isolinux.bin_
```bash
cp $HOME/Distro/chroot/usr/share/desktop-base/debian-logos/logo-text-version-256.png $HOME/Distro/antares/isolinux/splash.png
```
_Copiar boot.cat e isolinux.bin_
```bash
cp $HOME/Distro/chroot/usr/share/desktop-base/debian-logos/logo-text-version-256.png $HOME/Distro/antares/isolinux/splash.png
```

_Copiar fonte unicode.pf2_
```bash
cp $HOME/Distro/chroot/boot/grub/unicode.pf2 $HOME/Distro/antares/boot/grub/
```

_Copiar vmlinuz e initrd.img_
```bash   
cp $HOME/Distro/chroot/boot/vmlinuz-* $HOME/Distro/antares/live/vmlinuz
cp $HOME/Distro/chroot/boot/initrd.img-* $HOME/Distro/antares/live/initrd.lz
```
# Apagar vmlinuz.old e initrd.img.old
```bash   
rm -r $HOME/Distro/chroot/vmlinuz && sudo rm -r $HOME/Distro/chroot/vmlinuz.old
rm -r $HOME/Distro/chroot/initrd.img && sudo rm -r $HOME/Distro/chroot/initrd.img.old
```

# <p align="center">Squashfs
### Regerando os arquivos
_Regerando o arquivo filesystem.manifest e filesystem.squashfs_
```bash
chmod +w antares/live/filesystem.manifest
sudo chroot chroot dpkg-query -f '${binary:Package}\n' -W > antares/live/filesystem.manifest
sudo cp antares/live/filesystem.manifest antares/live/filesystem.manifest
sudo rm antares/live/filesystem.squashfs
sudo mksquashfs chroot antares/live/filesystem.squashfs -comp xz
```
# MD5sum
_Criar o MD5sum_
```bash
cd antares
sudo rm md5sum.txt
find -type f -print0 | xargs -0 md5sum | grep -v isolinux/boot.cat | tee md5sum.txt
cd
cd Distro
```

# <p align="center">Gerando s imagem ISO
_Criando a imagem ISO com genisoimage_
```bash
genisoimage \
-D -r -V “Antares-OS” -cache-inodes -J -l -b isolinux/isolinux.bin -c isolinux/boot.cat \
-no-emul-boot -boot-load-size 4 -boot-info-table -o Antares-OS-13.6.0-amd64-gnome-$(date +%d-%m-%Y).iso antares/
```

# Excluir diretório
_Excluir diretório de customização_
```bash
sudo rm -r Distro
```
# <p align="center">Desenvolvedor
  <footer id="footer">
    <div class="container">      
      <h5 class="font-italic">
      Imagens ISOs customizada do  <strong><span>GNU/Linux</span></strong>
      </h5>
      <div class="social-links">
        <a href="https://github.com/valdemir26/Customize#readme" class="telegram"><i class="bx bxl-github"></i></a>
        <a href="https://t.me/jvmelo26linux" class="telegram"><i class="bx bxl-telegram"></i></a>
        <a href="https://twitter.com/jvmelo26?s=09" class="twitter"><i class="bx bxl-twitter"></i></a>
        <a href="https://www.facebook.com/josevaldemir.melo" class="facebook"><i class="bx bxl-facebook"></i></a>
        <a href="https://www.instagram.com/josevaldemir.melo/" class="instagram"><i class="bx bxl-instagram"></i></a>   
      </div>
      <div class="copyright">
        &copy; Copyright <strong><span>Antares OS</span></strong> 2020 2026 All Rights Reserved
      </div>
      <div class="credits">
      Para tirar dúvidas <a href="https://t.me/valdemir26antaresOS">José Valdemir de Melo</a>
      </div>
    </div>
  </footer>

