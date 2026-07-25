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
Como criar sua própria ISO com debootstrap, use o comando para certificar que os pacotes estão presentes em seu sistema
```bash
apt update && apt -y install \
    debootstrap \
    squashfs-tools \
    genisoimage \
```

## _Criando a jaula do sistema_
Agora vamos criar o diretório que irão conter os arquivos nescessários para fazer o chroot
```bash
mkdir -p $HOME/Distro/{chroot,antares/{EFI/boot,boot/grub/live,isolinux,live},files}
cd Distro
```

## _Instala o sistema base com debootstrap_
A ferramenta debootstrap irá selecionar os pacotes nescessários da base so sistema escolhido para chroot, é nescessário instalar a chave gpg do sistema escolhido
### Exemplo 
debian-archive-keyring
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
apt update && apt dist-upgrade
```

## _Pacotes para instalação minima_
A lista de pacotes que eu escolhi, mas remove a opção de instalção dos aquivos recomendados no CONTROL do pacote .deb, para uma construção de ISOs sem nenhuma dependência futura remova o --no-install-recommends
```bash
apt install --no-install-recommends \
apt-transport-https build-essential btrfs-progs curl dbus-x11 dosfstools dkms rsync e2fsprogs exfatprogs \
efibootmgr linux-image-amd64 live-boot live-config squashfs-tools genisoimage isolinux lsb-base grub-common \
grub2-common grub-efi-amd64 grub-efi-amd64-bin wget os-prober gnome-accessibility-themes gnome-disk-utility \
gnome-terminal gnome-shell gnome-shell-common gnome-shell-extension-prefs gnome-shell-extensions \
gnome-software gnome-session gnome-tweaks nautilus mutter gdm3 xinit gnome-control-center xdg-user-dirs-gtk \
gedit file-roller yad calamares calamares-settings-debian
```

## _Firmwares_
Instalar os drivers firmware-linux-free e firmware-linux-nonfree, alguns firmware-nonfree é nescessário aceitar os termos para instalação do pacote
```bash
apt install \
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

## _Squashfs_
Regerando o arquivo filesystem.manifest e filesystem.squashfs
```bash
chmod +w antares/live/filesystem.manifest
sudo chroot chroot dpkg-query -f '${binary:Package}\n' -W > antares/live/filesystem.manifest
sudo cp antares/live/filesystem.manifest antares/live/filesystem.manifest
sudo rm antares/live/filesystem.squashfs
sudo mksquashfs chroot antares/live/filesystem.squashfs -comp xz
```

## _README.diskdefines_
Cria um rótulo para a imagem ISO
```bash
cd antares
sudo rm md5sum.txt
find -type f -print0 | xargs -0 md5sum | grep -v isolinux/boot.cat | tee md5sum.txt
cd
cd Distro
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

# Squashfs
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

Se você conseguir chegar até aqui então falta muito pouco, este final de semana eu vou concluir o passo a passo, faltam alguns arquivos de configuração a serem disponibilizados aqui, então nao remova as pastas.



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

