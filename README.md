# Instalación del sistema Arch Linux con UEFI
Guia en castellano para instalar Arch Linux en un etorno Virtual "Virtualbox" con UEFI-boot.

Se recomienda leer el archivo Arch-linux-instalacion-UEFI.pdf para la configuración de Virtualbox.

Aquí encuentras todos los comandos de shell para la instalación despues de haber arrancado el sistema en un etorno virtual para tener un Arch Linux funcionando en un etorno virtual.

## 1. Primeros comandos
Empezamos con los siguientes ordenes:

Cambiamos al teclado español con nuestra querrida 'ñ'.

    loadkeys es
    
Si el archivo existe, entonces está el EFI-boot activado.

    ls /sys/firmware/efi/efivars
    
Controlamos la conexión internet. ping, etc,...

    ip link
    
Hora del sistema
    
    timedatectl set-ntp true
   
Comprobar el nombre del disco duro, normalmente es /dev/sda  
   
    fdisk -l 
    
Arrancamos el Manager de particiones gdisk
        
    gdisk /dev/sda → arranca Manager de particiones



## 2. GDISK - particionado del disco duro.
Crear partición con gdisk
n → nueva partición
```diff
+ Command (? for help): n
+ Partition number (1-128, default 1):
+ First sector (34-5860533134, default = 2048) or {+-}size{KMGTP}:
+ Last sector (2048-5860533134, default = 5860533134) or {+-}size{KMGTP}:
+ Current type is 'Linux filesystem'
+ Hex code or GUID (L to show codes, Enter = 8300):

```

Tabla, ejemplo con un disco duro mas de 20GB. 
1. partición: +1024M (para EFI, 512 es suficiente, dicen) HEX-code: ef00
2. partición: +2048M (SWAP, tamaño de memoria RAM, con 2G va bien) HEX-code: 8200
3. partición: +10G   (root partición, minimo 4G), HEX-code: 8303
4. partición: +4G    (var partición, no es necesario), HEX-code: 8310
5. partición: Resto del espacio libre (home partición) , HEX-code: 8302
  
**Info:** Instalación mínima con EFI son 3 partitiones. EFI, SWAP, ROOT - ‘/’.

Finalmente terminamos el particionado – comando ‘**w**’.


## 3. Formatesar particiones
BOOT paritición. (‘n’ es Label en caso de fat, normalmente es ‘L’)

    mkfs.fat -F 32 -n BOOT /dev/sda1
    
SWAP
    
    mkswap -L SWAP /dev/sda2
    
ROOT

    mkfs.ext4 -L ROOT /dev/sda3

VAR

    mkfs.ext4 -L VAR /dev/sda4

HOME
    
    mkfs.ext4 -L HOME /dev/sda5



## 4. Crear puntos de montajes y montar particiones
Trabajamos con Lables ‘L’:

Root partition directamente montado en /mnt.

    mount -L ROOT /mnt
    
Crea carpeta boot

    mkdir /mnt/boot        
   
Montar partición con efi - “BOOT” en boot.
    
    mount -L BOOT /mnt/boot
    
Crea carpeta “var”
    
    mkdir /mnt/var
    
Monta la partition VAR en /mnt/var.    

    mount -L VAR /mnt/var
    
Crea carpeta "home"

    mkdir /mnt/home     
    
Monta la partición HOME en /mnt/home.

    mount -L HOME /mnt/home 


## 5. Activar SWAP
Activamos el espacio swap. El ejemplo con Label, opcional "swapon /dev/sda".

    swapon -L SWAP
  
  
## 6 Instalación del sistema básico - pacstrap
  Comando pacstrap instala paquetes. Empezamos con la instalación de algunos paquetes básicos.
  
    pacstrap /mnt base base-devel linux linux-firmware gptfdisk efibootmgr bash-completion vim lvm2 networkmanager
  
  Recomiendo instalar ya el networkmanager o dhcpcd. Nano se encuentra en el paquete “base”.
  No es necesario para tener un sistema Arch funcionando en Virtualbox pero se puede instalar
  los paquetes de virtualbox que pronto hacen falta para p.E. soporte USB. Para nucleos linux
  se llama “virtualbox-host-modules-arch” y para otros nucleos incluido linux-lts es 
  “virtualbox-host-dkms”.
  
  https://wiki.archlinux.org/title/VirtualBox_(Español)
  

## 7 Primer configuración del sistema

### 7.1 Crear archivo fstab
    genfstab -Lp /mnt >> /mnt/etc/fstab
 
### 7.2 Crear archivo hostname
Cambia “myhost” con tu nombre preferido.
    echo myhost > /mnt/etc/hostname 
    
### 7.3 Cambiar a nuestro sistema nuevo con arch-chroot
Cambiamos a nuestra consola del nuevo sistema:
    arch-chroot /mnt     

### 7.4 Crear arranque efi
No hace falta instalar un bootloader. Tenemos que decir al systemd que arranca en modo efi.
    bootctl install

### 7.5 Modificar archivo de configuración ‘loader.conf’
    vim /boot/loader/loader.conf
Borrar el contenido y reemplazarlo con:
  
    # para copiar y pegar en el archivo /boot/loader/loader.conf
    default arch
    timeout 3
    editor 0
    
   editor 0 → Es importante para no pasar la contraseña de root al arrancar el sistema.
   
### 7.6 Crear archivo de configuración ‘arch.conf’
    vim /boot/loader/entries/arch.conf
 Texto para copiar: 
 
    # para copiar y pegar al archivo /boot/loader/entries/arch.conf
    title
    Arch Linux
    linux
    /mvlinuz-linux
    initrd
    /initramfs.linux.imgç
    options root=LABEL=ROOT rw
    
### 7.7 Crear contraseña root
Poner root password: 

    passwd
    
No necesario pero mejor haberlo hecho ya.
Si no se pone, se queda vacio y el usuario puede conectarse sin contraseña.
**Ojo:** no eliges nada raro de contraseña al principio.
Tenemos que configurar el teclado defenitivamente mas adelante.
  
  
### 7.8 Terminamos por fin
Salir del sistema:
    
    exit
    
Arrancar el sistema de nuevo para comprobar la instalación:
    
    reboot
 
Ahora nos encontramos en nuestra nueva instancia de Arch Linux.

No olvides de nuevo el teclado: loadkeys es y empezamos a configurar. Teclado, red, desktop, server, .....

Mas aquí:

https://wiki.archlinux.org/title/Arch_Linux_(Español)
  
  
 
