# ArchLinuxInstall

## Instructions for basic Arch installation

### Creating the install medium

#### Creating the USB image
In order to start installing Archlinux, we need a working ISO image. Similar to how it is done on Ubunutu, a bootable USB stick can be created in Windows using Rufus or something similar.

#### Certifying the ISO image (IMPORTANT)
Since the ISO is usually downloaded from mirrors, it is possible for malicious actors to alter the ISO, resulting in a faulty/non-operational installation at best, or a compromised installation with backdoors at worst. To prevent this, the ISO image needs to be certified. To do this, download the `.sig` file from the same source and keep it in the same folder as the ISO file.

Then, on a system with `GnuPG` installed, run

`gpg --keyserver-options auto-key-retrieve --verify archlinux-*version*-x86_64.iso.sig`

to check a specific Arch Linux *version*. To verify the authenticity of the signature also ensure that the public key fingerprint as printed above matches the key fingerprint of the [developer who signed the ISO-file as listed here](https://archlinux.org/people/developers/).

### Booting

1. Point boot device to the one that has the Arch Linux install medium.

2. You will be logged into the virtual console as root user. All install fun begins here :).

### Keymap

If you are used to the US keymap, that is the default and you don't need to do anything. Proceed to next step.

If you prefer a specific non-default keymap, available layouts are listed with

`localectl list-keymaps`

Specific keymaps can be loaded by

`loadkeys *keymap*`

### Verify Boot mode

`cat /sys/firmware/efi/fw_platform_size`

should return 64 if you are using UEFI mode on a 64-bit x64 UEFI.

### Connect to internet

The command

`ip link`

shows if a network interface is listed and enabled. Here we will focus on WiFi connections.

Type

`iwctl`

to open an interactive prompt prefixed by `[iwd]#`.

Now, list all available devices in this prompt by

`[iwd]# device list`

Turn a specific listed adapter whose name is `NAME` on (if its off) using

`[iwd]# device NAME set-property Powered on`

Next, initiate a scan of available networks using

`[iwd]# station NAME scan`

List all available networks using

`[iwd]# station NAME get-networks`

Finally connect to a network whose name is `SSID` using

`[iwd]# station NAME connect SSID`

You will be prompted for the Wifi password if its a private network. Once connected, exit the prompt by typing `exit` to return to the interactive console.

### Update System time

Type `timedatectl`

### Partition disks

Here is where the fun starts. To understand the type of disk-device available for booting, run

`fdisk -l`

This will list `/dev/sda, /dev/nvme0n1` etc, depending on the type of the hard disk.

Select the device on which you want to install the `root`, `efi_boot` and `swap` partitions of the Arch installation, using

`fdisk /dev/the_disk_to_be_partitioned`

We assume WLOG a `GPT` parition scheme (as opposed to `MBR`). For such a parition scheme, the following parititon structure is recommended.

| Mount point on installed system | Partition | Parition type | Recommended Size |
|---------------------------------|-----------|---------------|------------------|
| `/boot`                         | `/dev/efi_system_partition` | EFI system partition | 1GB |
| `[SWAP]`                        | `/dev/swap_partition` | Linux swap | >= 4GB |
| `/`                             | `/dev/root_partition` | Linux x86_64 root | Remainder of device |

Formatting of partitions is done by

`mkfs.ext4 /dev/root_partition`
`mkswap /dev/swap_partition`
`mkfs.fat -F 32 /dev/efi_system_partition`

Mount the partitions using

`mount /dev/root_partition /mnt`
`mount --mkdir /dev/efi_system_partition /mnt/boot`

Enable swap volume using

`swapon /dev/swap_partition`

### Install essential packages

`pacman -K /mnt base linux linux-firmware`

### Configure

`genfstab -U /mnt >> /mnt/etc/fstab`

### Chroot

`arch-chroot /mnt`

### Timezone

`ln -sf /usr/share/zoneinfo/*Region*/*City* /etc/localtime`

### Localization

`locale-gen`

`touch /etc/locale.conf`

`echo LANG=en_US.UTF-8 > /etc/locale.conf`

### Network configuration

`touch /etc/hostname`

`echo yourhostname > /etc/hostname`

### Root password

`passwd`

and then enter the password you want.

### Bootloader (Be careful else you wont be able to boot into your Arch Linux installation)

There are a variety of boot loaders available. We pick `grub` as it is the most familiar for users of Ubuntu.

We follow these steps to get the boot loader up and running.

#### Install pre-requisites

`pacman -S grub`
`pacman -S efibootmgr`

#### Mount the EFI system partition

`mount /dev/root_partition /mnt`
`mount /dev/efi_system_partition /mnt/boot`

You can choose specific mount points for the root and efi system partitions. We follow these mount points for illustrative purposes.

Next, we chroot

`arch-chroot /mnt`

Now run the grub install

`grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=grub`

Generate config files using

`grub mkconfig -o /boot/grub/grub.cfg`

Then, `exit` the chroot environment.

`unmount -R /mnt`

`reboot`

## Post installation customization and fine tuning

### Managing internet connections

Network Manager is a good way to orchestrate and connect with wired/wireless connections. However, a choice has to be made for the backend for this as multiple such services exist, and if all of them are active simultaneously the system won't be able to connect to the internet.

We follow here the process of setting `iwd` (which we used during installation) as the backend for Network Manager.

`touch /etc/NetworkManager/conf.d/wifi_backend.conf`

Edit this file to have the content

```
[device]
wifi.backend=iwd
```
Next, Network manager has a CLI version named `nmcli` that can be used to create and modify internet connections to the device.














