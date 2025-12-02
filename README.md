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
HOOKS=(base udev autodetect modconf block keyboard keymap encrypt filesystems fsck)
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

Agregar parámetros de LUKS:
```
nano /etc/default/grub
```

Agregar dentro:
```
GRUB_CMDLINE_LINUX="cryptdevice=UUID=<UUID-de-sdX3>:cryptroot root=/dev/mapper/cryptroot"
```

Generar GRUB:
```
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

Obtener UUID:
```
blkid
```

---

## 10. Crear Usuario

```
passwd
useradd -m "<name>"
passwd "<name>"
```

Habilitar sudo:
```
nano /etc/sudoers
```
Copiar la línea de root y reemplazar usuario.

---

## 11. Swapfile (si NO usaste partición swap)

```
fallocate -l 4G /swapfile
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

## 13. Mapeo de Teclado Permanente

Crear archivo:
```
echo "KEYMAP=la-latin1" > /etc/vconsole.conf
```

---

## 14. Reiniciar

```
exit
umount -R /mnt
reboot
```

