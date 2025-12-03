# Guía de Instalación de Arch Linux con Encriptación LUKS + EFI + Swapfile

## 1. Boot desde USB

1. Iniciar la PC con el pendrive.
2. Configurar teclado (opcional):
   ```
   loadkeys la-latin1
   ```

3. Conectarse a Wi‑Fi:
   ```
   iwctl
   device list
   station <device> scan
   station <device> get-networks
   station <device> connect "<SSID>"
   exit
   ```
4. Probar conexión:
   ```
   ping archlinux.org
   ```

---

## 2. Particionado del disco

Ver discos:
```
fdisk -l
```

Entrar al disco:
```
cfdisk /dev/sdX
```

Crear:
- 1 GB → `EFI System`
- (opcional) swap partition
- resto → para root (será encriptado)

---

## 3. Encriptar Partición Root (LUKS)

Inicializar LUKS:
```
cryptsetup luksFormat /dev/sdX3
```

Abrirla:
```
cryptsetup open /dev/sdX3 cryptroot
```

---

## 4. Formateo

Formatear root encriptado:
```
mkfs.ext4 /dev/mapper/cryptroot
```

Formatear EFI:
```
mkfs.fat -F 32 /dev/sdX1
```

Si usás partición swap:
```
mkswap /dev/sdX2
swapon /dev/sdX2
```

---

## 5. Montaje

Montar root:
```
mount /dev/mapper/cryptroot /mnt
```

Montar EFI:
```
mount --mkdir /dev/sdX1 /mnt/boot
```

---

## 6. Instalación del Sistema Base

```
pacstrap -K /mnt base linux linux-firmware
```

Generar fstab:
```
genfstab -U /mnt >> /mnt/etc/fstab
```

Entrar al sistema:
```
arch-chroot /mnt
```

---

## 7. Configuraciones Básicas

Instalar editor de texto:
```
pacman -S nano
```

Keymap:
```
echo "KEYMAP=la-latin1" > /etc/vconsole.conf
```

Zona horaria:
```
ln -sf /usr/share/zoneinfo/America/Argentina/Buenos_Aires /etc/localtime
hwclock --systohc
```

Locales:
```
nano /etc/locale.gen
locale-gen
```

Crear archivo:
```
echo "LANG=es_AR.UTF-8" > /etc/locale.conf
```

Hostname:
```
echo "mipc" > /etc/hostname
```

---

## 8. mkinitcpio con Soporte LUKS

Editar HOOKS:
```
nano /etc/mkinitcpio.conf
```

Usar:
```
HOOKS=(base systemd autodetect microcode modconf kms keyboard keymap sd-vconsole sd-encrypt block filesystems fsck)
```

Regenerar:
```
mkinitcpio -P
```

---

## 9. Configurar GRUB con Encriptación

CPU microcode:
```
pacman -S intel-ucode
```
o:
```
pacman -S amd-ucode
```

Instalar bootloader:
```
pacman -S grub efibootmgr
```

Generar GRUB:
```
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```

Obtener UUID:
```
blkid
```

Agregar parámetros de LUKS:
```
nano /etc/default/grub
```

Agregar dentro:
```
GRUB_CMDLINE_LINUX="rd.luks.name=<UUID>=cryptroot root=/dev/mapper/cryptroot"

```

Generar GRUB:
```
grub-mkconfig -o /boot/grub/grub.cfg
```

---

## 10. Crear Usuario

```
passwd
useradd -m "<name>"
passwd "<name>"
```

Instalar sudo:
```
pacman -S sudo
```

Habilitar sudo:
```
nano /etc/sudoers
```
Copiar la línea de root y reemplazar usuario.

---

## 11. Swapfile (si NO usaste partición swap)

Saber capacidad de memoria RAM:
```
free -h
```

En vez de 4G se recomienda el doble de tu memoria ram
```
fallocate -l 4G /swapfile
````

Mount swap
```
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```

Agregar a fstab:
```
echo "/swapfile none swap defaults 0 0" >> /etc/fstab
```

---

## 12. Activar NetworkManager

```
pacman -S networkmanager
systemctl enable NetworkManager
```

---

## 13. Reiniciar

```
exit
swapoff /mnt/swapfile
umount -R /mnt
reboot
```

---

login en tu usuario
ping archlinux.org
umcli
sudo pacman -S hyprland uwsm xdg-desktop-portal-hyprland hyprpolkitagent pipewire-pulse pipewire-alsa pipewire-jack wofi waybar kitty mako git base-devel

git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
cd ..
rm -rf ./yay

ls -a
nano .bash_profile:

if uwsm check may-start; then
   exec uwsm start hyprland.desktop
fi

reboot

login en tu usuario
Win + Q -> abrir la terminal
Win + C -> cerrar ventana

[Copiar configuración del tutorial](https://github.com/LukeElrod/dotfiles.git)