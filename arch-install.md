# Чистая установка Arch Linux в qemu или на реальном железе

## Сеть и Загрузка (Реальное железо, wifi, для кабеля не нужно)
```bash
iwctl
station wlan0 scan
station wlan0 get-networks
station wlan0 connect <Имя_вашей_сети>
exit
```
## Разметка диска

Пишите команду:
```bash
lsblk
```
Найдите ваш диск, например:
```bash
/dev/sda
или
/dev/nvme0n1
или
/dev/vda
```
После — запустите:
```bash
cfdisk /dev/что_нашли
```
### Если на QEMU (20G)

Создайте 2 раздела:

- /dev/sda1 — 1G, BIOS boot
- /dev/sda2 — 19G, Linux filesystem

### Если на реальном железе (500G)

Создайте 2 раздела:

- /dev/nvme0n1p1 — 1G, EFI system partition
- /dev/nvme0n1p2 — 499G, Linux filesystem

---

## Форматирование диска

### Если на QEMU
```bash
mkfs.ext4 /dev/sda2
```
### Если на реальном железе
```bash
mkfs.fat -F32 /dev/nvme0n1p1
mkfs.ext4 /dev/nvme0n1p2
```
---

## Монтирование

### QEMU
```bash
mount /dev/sda2 /mnt
```
### Реальное железо
```bash
mount /dev/nvme0n1p2 /mnt
mkdir /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot
```
---

## Установка базовых пакетов
```bash
pacstrap /mnt base linux linux-firmware vim grub efibootmgr networkmanager
```
---

## Генерация fstab
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```
---

## Настройка системы (chroot)
```bash
arch-chroot /mnt
```
---

## Внутри chroot:

### Настройка временной зоны
```bash
ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime
hwclock --systohc
```
### Локализация
```bash
echo "ru_RU.UTF-8 UTF-8" >> /etc/locale.gen
locale-gen
echo "LANG=ru_RU.UTF-8" > /etc/locale.conf
```
### Имя хоста
```bash
echo "archlinux" > /etc/hostname
```
### Установка пароля root
```bash
passwd
```
---

## Установка и настройка загрузчика

### Для QEMU (BIOS)
```bash
grub-install --target=i386-pc /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
```
### Для реального железа (UEFI)
```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```
---

## Включение NetworkManager
```bash
systemctl enable NetworkManager
```
---

## Выход из chroot, размонтирование и перезагрузка
```bash
exit
umount -R /mnt
reboot
```
---

*Система установлена и готова к загрузке!*
# Скачивания окружения + драйверов
## Проверка интернета
```bash
ping -c 3 archlinux.org
```
# Еcли нету интернета
```bash
systemctl enable systemd-networkd
systemctl start systemd-networkd

systemctl enable systemd-resolved
systemctl start systemd-resolved

networkctl status

ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
```
```bash
ip a
# Тут надо найти что то кроме lo
vim /etc/systemd/network/20-wired.network
```
```ini
[Match]
Name=имя_интерфейса_тут

[Network]
DHCP=ipv4
```


## Звук
```bash
pacman -S alsa-utils pulseaudio pavucontrol
```
## Драйвера на видюху
```bash
pacman -S nvidia
```
## KDE Plasma
```bash
pacman -S plasma kde-applications sddm

systemctl enable sddm.service
systemctl start sddm.service
```
## GNOME
```bash
pacman -S gnome gnome-extra gdm

systemctl enable gdm.service
systemctl start gdm.service

```
### Создание юзера с doas
```bash
pacman -S doas
```
Дальше редактируете файл `/etc/doas.conf`
```bash
permit :wheel
```
И пишете в терминал
```bash
useradd -m -G wheel -s /bin/bash ваш_юзернейм
passwd ваш_юзернейм
```
