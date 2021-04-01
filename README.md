# INSTALAÇÃO BASE

## 1º Verificar se instalação está no modo UEFI ou BIOS
```
efivar -l
```
obs: Se retornar alguma coisa, quer dizer que a instalação é em modo **UEFI**, se não retornar nada ou retornar algum erro a instalação é em modo **BIOS**.

## 2º Mudar layout do teclado para brasileiro
```
loadkeys br-abnt2
```

## 3º Ativar conexão com internet e verificar a conexão
```
systemctl start dhcpcd
systemctl status dhcpcd
ping -c3 google.com
```

## 4º Listar HD/Partições, formatar HD e criar as partições necessárias

> Listar HD/Partições
```
fdisk -l

ou

lsblk
```
> Formatar HD
```
sgdisk --zap-all /dev/sda
```
> Criar partições
```
cfdisk /dev/sda
```
obs: Ao entrar selecione a opção **dos**, apague todas as partições caso existir, crie somente 1 com todo o restante do HD para isso basta clicar em **New** para criar uma nova partição e dar **Enter** para colocar todo o HD, não se esqueça de ir em **Write** que é para escrever as alterações no disco e digitar **yes** para confirmar.

## 5º Formatar e montar as partições
```
mkfs.ext4 /dev/sda1
mount -t ext4 /dev/sda1 /mnt
```

## 6º Instalar sistema base
```
pacstrap /mnt base base-devel linux linux-firmware
```

## 7º Gerar FSTAB
```
genfstab -U -p /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
```

## 8º Entrar no novo sistema
```
arch-chroot /mnt
```

## 9º Mudar a linguagem do sistema
obs: O editor de texto **nano** não estará mais disponível nesse novo ambiente, então você deve instalar primeiro:

```
pacman -S nano
nano /etc/locale.gen
```
obs: Pressionar **Control + W** para pesquisar, e buscar por **pt_BR.UTF-8**, agora descomente essa linha que irá ser retornado pela pesquisa, removendo o **#** da frente dela, depois basta pressionar **Control + X** apertar **Y** e dar **Enter** para salvar as configurações.

> Atualizar/Gerar essa linguagem definida acima:
```
locale-gen
echo LANG=pt_BR.UTF-8 > /etc/locale.conf
export LANG=pt_BR.UTF-8
```
Após isso o sistema estará em português.

## 10º Realizando persistência do layout de teclado
```
nano /etc/vconsole.conf
```
Coloque dentro desse arquivo esse conteúdo:
```
KEYMAP=br-abnt2
```
obs: Pressionar **Control + X** apertar **S** e dar **Enter** para salvar as configurações.

## 11º Criar link simbólico para configurar TIMEZONE
```
ln -sf /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime
```

## 12º Habilitar MULTILIB para realizar downloads de pacotes 32bits
```
nano /etc/pacman.conf
```
obs: As linhas referentes ao multilib estará assim:
```
#[multilib]
#Include = /etc/pacman.d/mirrorlist
```
> Ela terá de ficar assim:
```
[multilib]
Include = /etc/pacman.d/mirrorlist
```
obs-2: Basta remover os **#** (remova todos os **#** que tiver abaixo da linha referente ao **multilib**) mas não mexa no **multilib-testing**, basta salvar com **Control + X** apertar **S** e **Enter**.


> Depois basta executar o comando abaixo para sincronizar o novo repositório habilitado.

```
pacman -Syy
```

## 13º Definir senha do usuário ROOT
```
passwd
```

## 14º Criar usuário e definir senha para ele
```
useradd -m -g users -G wheel,storage,power -s /bin/bash SEU_USUARIO
passwd SEU_USUARIO
```

## 15º Instalar GRUB
```
pacman -S grub
grub-install /dev/sda
```

## 16º Rodar MKINITCPIO
```
mkinitcpio -p linux
```

## 17º Configurar GRUB
```
grub-mkconfig -o /boot/grub/grub.cfg
```

## 18º Instalar pacotes necessários para inicialização correta
```
pacman -S netctl dialog dhcpcd
```

## 19º Desmontar partições e desligar
```
exit
umount -a
poweroff
```
Após isso a base do sistema estará pronta, remova o pendrive, abra a **BIOS** e mude a ordem de boot para o **HD** mesmo.


# PÓS INSTALAÇÃO

## 1º Logue como ROOT
> Entre no sistema com o usuário **root** e a **senha** que você definiu para ele

## 2º Ativar rede para iniciar automaticamente
```
systemctl enable dhcpcd
systemctl start dhcpcd
systemctl status dhcpcd
ping -c3 google.com
```

## 3º Definir um hostname para o sistema
obs: Não pode conter espaços, letras maiúsculas e caracteres especiais
```
hostnamectl set-hostname NOME
```
> Onde **NOME** será o hostname que você definirá, por exemplo: arch

## 4º Instalando drivers de áudio
```
pacman -S alsa-utils pulseaudio
```

## 5º Instalar e testar servidor gráfico XORG
```
pacman -S xorg-server xorg-xinit xorg-xprop xorg-xrandr xorg-twm xorg-xclock xterm
```
Para testar se está funcionando:
```
startx
```
Se abrir uma tela feia com 3 terminais e um relógio, seu servidor gráfico está funcionando, bastante executar **exit** no terminal maior que tiver na vertical a esquerda

> Remova os pacotes instalados para testes
> `pacman -Rnsc xorg-twm xorg-xclock xterm`

## 6º Instalar driver gráfico e de aceleração 3D
```
pacman -S xf86-video-vesa mesa
```

## 7º Instalar interface gráfica e display manager
```
pacman -S deepin deepin-extra gdm
systemctl enable gdm
```

## 8º Últimas configurações para desfrutar essa delícia
```
reboot
```
obs: Logue com seu usuário normal, quando estiver dentro da interface gráfica, abra o terminal:

> Digite o comando abaixo e informe sua senha do usuário
```
sudo su
```
> Abra o arquivo de configuração **sudoers**
```
nano /etc/sudoers
```
obs-2: Localize a linha 79 onde terá o seguinte conteúdo
```
##
## User privilege specification
##
root ALL=(ALL) ALL
```
E abaixo de root você colocará o nome do seu usuário seguido de ALL=(ALL) ALL
Ficando assim:
```
##
## User privilege specification
##
root ALL=(ALL) ALL
SEU_USUARIO ALL=(ALL) ALL
```
# SISTEMA INSTALADO <3
### COM GRANDES PODERES VEM GRANDES RESPONSABILIDADES!


.

> Para usar o dual boot com Windowns vai ser necessário instalar os pacotes: os-prober e ntfs-3g
```
# pacman -S os-prober ntfs-3g

# grub-mkconfig -o /boot/grub/grub.cfg
```
