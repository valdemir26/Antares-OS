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

## _Como funciona_
Optamos por utilizar um processo manual de customização do Linux, utilizando o squashfs, genisoimage e chroot, é simples, você usa a própria distribuição instalada como base, sem aplicativos extras, que as vezes é incompatível dependendo da distribuição.

## _Preparando o ambiente_
### Para criar o ambiente necessário para customização
Para criar o ambiente necessário para customização
Computador (Desktop ou Notebook) com Linux instalado e com suporte para o "squashfs" no kernel, desde 2006 o linux já possui suporte para o Squashfs no kernel, porém sugerimos utilizar a versão mais atual disponível.

Como criar sua própria ISO com debootstrap, use o comando para certificar que os pacotes estão presentes em seu sistema
```bash
sudo apt update && apt -y install \
    debootstrap \
    squashfs-tools \
    genisoimage \
```

## _Criando a jaula do sistema_
Antes de começarmos a criar a nossa *Distro*, devemos criar o diretório e os subdiretórios que serão nossa área de trabalho
Para criar o diretório e os subdiretórios, podemos usar o navegador de aquivos ou simplesmente fazer isso no terminal como decrito logo abaixo \
Depois que o diretórios e os subdiretórios estiverem criados, é só seguir o passo a passo \
Agora vamos criar o diretório que irão conter os arquivos nescessários para fazer o chroot
```bash
mkdir -p $HOME/Distro/{chroot,antares/{EFI/boot,boot/grub/{theme,x86_64-efi},isolinux,live},files}
cd Distro
```

## _Instala o sistema base com debootstrap_
A ferramenta debootstrap irá selecionar os pacotes nescessários da base so sistema escolhido para chroot, é nescessário instalar a chave gpg do sistema escolhido
### Exemplo 
debian-archive-keyring \
http://deb.debian.org/debian/pool/main/d/debian-archive-keyring/
```bash
sudo debootstrap \
    --arch=amd64 \
    --variant=minbase \
    trixie \
    $HOME/Distro/chroot \
    http://deb.debian.org/debian/
 ```    

 ## _Iniciando o chroot_
Copia os arquivos /resolv.conf /hosts da maquina local e monta /dev /proc /sys para configuração de ambiente para uso do chroot
```bash
sudo cp /etc/resolv.conf chroot/etc/
sudo cp /etc/hosts chroot/etc/
sudo mount --bind /dev chroot/dev
sudo mount --bind /proc chroot/proc
sudo mount --bind /sys chroot/sys
sudo chroot chroot
```

## _Adicionando repositório Debian_
Source.list
```bash
cat > /etc/apt/sources.list << 'EOF'
deb http://deb.debian.org/debian trixie main non-free-firmware contrib non-free
deb http://deb.debian.org/debian trixie-updates main non-free-firmware contrib non-free
deb http://deb.debian.org/debian trixie-proposed-updates main non-free-firmware contrib non-free
deb http://security.debian.org/debian-security/ trixie-security main non-free-firmware contrib non-free
EOF
```

## _Atualizar a lista de pacotes_
Carrega a lista de pacotes para serem atualizados ou instalados
```bash
apt update && apt dist-upgrade -y
```

## _Pacotes para instalação minima_
A lista de pacotes que eu escolhi, para uma interface limpa mas fica ao critério de cada escolher seus próprios pacotes
```bash
apt install -y \
apt-transport-https build-essential btrfs-progs curl dbus-x11 dosfstools dkms rsync e2fsprogs exfatprogs \
efibootmgr linux-image-amd64 live-boot live-config squashfs-tools genisoimage isolinux lsb-base grub-common \
grub2-common grub-efi-amd64 grub-efi-amd64-bin wget os-prober gnome-accessibility-themes gnome-disk-utility \
gnome-shell gnome-shell-common gnome-shell-extension-prefs gnome-shell-extensions gnome-software gnome-session \
gnome-tweaks gnome-terminal nautilus mutter mtools gdm3 xinit gnome-control-center xdg-user-dirs-gtk gedit file-roller \
yad calamares calamares-settings-debian
```

## _Firmwares_
Instalar os drivers firmware-linux-free e firmware-linux-nonfree, alguns firmware-nonfree é nescessário aceitar os termos para instalação do pacote
```bash
apt install -y \
firmware-amd-graphics firmware-ast firmware-ath9k-htc firmware-atheros firmware-bnx2 firmware-bnx2x \
firmware-brcm80211 firmware-cavium firmware-intel-sound firmware-ipw2x00 firmware-ivtv firmware-iwlwifi \
firmware-libertas firmware-linux firmware-linux-free firmware-linux-nonfree firmware-misc-nonfree \
firmware-myricom firmware-netronome firmware-netxen firmware-qcom-soc firmware-qlogic firmware-realtek \
firmware-samsung firmware-siano firmware-sof-signed firmware-ti-connectivity
```


## _Limpar cache do APT e finalizar o chroot_
Remove os arquivos de configuração usandos no chroot
```bash    
rm -rf /tmp/* ~/.bash.history
rm /etc/resolv.conf
rm /etc/hosts
exit
```

## _Desmontar o ambiente de customização_
Desmonta o /dev /proc /sys que também foram montados no chroot
```bash   
sudo umount -lf chroot/dev
sudo umount -lf chroot/proc
sudo umount -lf chroot/sys
```
## Criando o usuário live do sistema
Este é o usuário padrão do sistema live
```bash 
cat > $HOME/Distro/files/antares.conf << 'EOF'
LIVE_USERNAME="antares"
LIVE_USER_FULLNAME="Antares Live User"
EOF
sudo cp $HOME/Distro/files/antares.conf $HOME/Distro/chroot/etc/live/config.conf.d/antares.conf
```

## _README.diskdefines_
Cria um rótulo para a imagem ISO
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

## Arquivos de boot do sistema
# <p align="center">Grub</p>
Precisamos agora copiar os arquivos necessários de inicialização para BIOS Legacy para o diretório do LiveCD
```bash
sudo cp -r $HOME/Distro/chroot/usr/lib/grub/x86_64-efi/* "$HOME/Distro/antares/boot/grub/x86_64-efi/"
```

Vamos criar uma imagem inicializável para o GRUB EFI
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
```

Agora, criaremos uma imagem de disco de inicialização FAT16 UEFI contendo o carregador de inicialização EFI
```bash
cd $HOME/Distro/antares && \
    dd if=/dev/zero of=efi.img bs=1M count=20 && \
    mkfs.vfat efi.img && \
    mmd -i efi.img efi efi/boot && \
    mcopy -vi efi.img $HOME/Distro/files/bootx64.efi ::efi/boot/
cd
cd Distro
```

# <p align="center">Isolinux</p>
Criar o config.cfg
```bash
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
```

Criar o config.cfg
```bash
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
```

Criar o loopback.cfg
```bash
source /boot/grub/grub.cfg
```

Criar o theme.txt
```bash
source /boot/grub/grub.cfg
```

Criar o config.cfg
```bash

```

## _Copiar vmlinuz e initrd.img_
```bash   
mkdir -p $HOME/Distro/antares/live
sudo cp $HOME/Distro/chroot/boot/vmlinuz-* $HOME/Distro/antares/live/vmlinuz
sudo cp $HOME/Distro/chroot/boot/initrd.img-* $HOME/Distro/antares/live/initrd.lz
```
# _Apagar vmlinuz.old e initrd.img.old_
```bash   
sudo rm -r $HOME/Distro/chroot/vmlinuz && sudo rm -r $HOME/Distro/chroot/vmlinuz.old
sudo rm -r $HOME/Distro/chroot/initrd.img && sudo rm -r $HOME/Distro/chroot/initrd.img.old
```

# <p align="center">Squashfs
### Regerando os arquivos
Regerando o arquivo filesystem.manifest e filesystem.squashfs
```bash
chmod +w antares/live/filesystem.manifest
sudo chroot chroot dpkg-query -f '${binary:Package}\n' -W > antares/live/filesystem.manifest
sudo cp antares/live/filesystem.manifest antares/live/filesystem.manifest
sudo rm antares/live/filesystem.squashfs
sudo mksquashfs chroot antares/live/filesystem.squashfs -comp xz
```
# MD5sum
Criar o MD5sum
```bash
cd antares
sudo rm md5sum.txt
find -type f -print0 | xargs -0 md5sum | grep -v isolinux/boot.cat | tee md5sum.txt
cd
cd Distro
```

# <p align="center">Gerando s imagem ISO
Criando a imagem ISO com genisoimage
```bash
genisoimage \
-D -r -V “Antares-OS” -cache-inodes -J -l -b isolinux/isolinux.bin -c isolinux/boot.cat \
-no-emul-boot -boot-load-size 4 -boot-info-table -o Antares-OS-amd64-$(date +%d-%m-%Y).iso antares/
```

# Excluir diretório
Excluir diretório de customização
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

