# Tema 101: Arquitectura del Sistema

- [Tema 101: Arquitectura del Sistema](#tema-101-arquitectura-del-sistema)
  - [101.1 Determinar y configurar los ajustes de hardware](#1011-determinar-y-configurar-los-ajustes-de-hardware)
    - [BIOS y UEFI](#bios-y-uefi)
    - [Sys y proc](#sys-y-proc)
    - [/dev y el comando **lsdev**](#dev-y-el-comando-lsdev)
    - [Módulos del Kernel](#m%c3%b3dulos-del-kernel)
    - [modprobe](#modprobe)
    - [lsmod](#lsmod)
    - [lspci y lsusb](#lspci-y-lsusb)
  - [101.2 Arranque del sistema](#1012-arranque-del-sistema)
    - [Secuencia de arranque](#secuencia-de-arranque)
      - [Cargadores de arranque (boot loader)](#cargadores-de-arranque-boot-loader)
    - [Tipos de arranque](#tipos-de-arranque)
  - [101.3 Cambiar los niveles de ejecución / objetivos de arranque y apagar o reiniciar el sistema](#1013-cambiar-los-niveles-de-ejecuci%c3%b3n--objetivos-de-arranque-y-apagar-o-reiniciar-el-sistema)

## 101.1 Determinar y configurar los ajustes de hardware

Importancia: 2
Descripción: Los candidatos deberán ser capaces de determinar y configurar el hardware fundamental del sistema.

**Áreas de conocimiento clave**:

- Activar y desactivar los periféricos integrados.
- Diferenciar entre los distintos tipos de dispositivos de almacenamiento masivo.
- Determinar los recursos de hardware para los dispositivos.
- Herramientas y utilidades para listar información de hardware (por ejemplo, lsusb, lspci, etc.).
- Herramientas y utilidades para manipular dispositivos USB.
- Conocimientos conceptuales de sysfs, udev y dbus.

Lista parcial de los archivos, términos y utilidades utilizados:

```bash
/sys/
/proc/
/dev/
modprobe
lsmod
lspci
lsusb
```

------

Página de guia -> [https://developer.ibm.com/tutorials/l-lpic1-101-1/]

### BIOS y UEFI

(BIOS): basic input/output system.
(UEFI): Unified Extensible Firmware Interface.

El **Firmware** de un dispositivo es el *software* de solo lectura que sirve para comunicar los diferentes componentes de hardware que componen los equipos informáticos entre si.

### Sys y proc

**sysfs** y **procfs** son sistemas de ficheros virtuales que se montan sobre `/sys` y `/proc` y que son usados por el kernel del sistema Linux. `Procfs` es el antiguo y `sysfs` es el moderno, aunque se siguen estando presente ambos en el sistema. `Sysfs` fue añadido en el *Kernel 2.6*.

**Sysfs** o `/sys` es un sistema de ficheros simple con ficheros que representan atributos de objetos, estos los objetos también son representados como directorios(carpetas) y son utilizados para obtener información sobre el sistema.

<https://github.com/torvalds/linux/blob/master/Documentation/filesystems>

Las carpetas que podemos encontrar dentro de `/sys` son:

```bash
xnoxos@ubuntu:~$ ls /sys
block  bus  class  dev  devices  firmware  fs  hypervisor  kernel  module  power
```

- **block**: encontramos enlaces simbólicos a dispositivos bloqueados
- **bus**: Contiene un diseño de directorio plano de los diferentes tipos de
kernel(núcleo).
- **dev**: contiene enlaces simbólicos para cada dispositivo descubierto en el sistema que apuntan al directorio del dispositivo bajo root /.
- **devices**: contiene un directorio por cada driver.

```bash
xnoxos@ubuntu:~$ ls -l /sys/block/
total 0
lrwxrwxrwx 1 root root 0 abr  3 14:19 loop0 -> ../devices/virtual/block/loop0
lrwxrwxrwx 1 root root 0 abr  3 14:19 loop1 -> ../devices/virtual/block/loop1
lrwxrwxrwx 1 root root 0 abr  3 14:19 loop2 -> ../devices/virtual/block/loop2
lrwxrwxrwx 1 root root 0 abr  3 14:19 loop3 -> ../devices/virtual/block/loop3
lrwxrwxrwx 1 root root 0 abr  3 14:19 loop4 -> ../devices/virtual/block/loop4
lrwxrwxrwx 1 root root 0 abr  3 14:19 loop5 -> ../devices/virtual/block/loop5
lrwxrwxrwx 1 root root 0 abr  3 14:19 loop6 -> ../devices/virtual/block/loop6
lrwxrwxrwx 1 root root 0 abr  3 14:19 loop7 -> ../devices/virtual/block/loop7
lrwxrwxrwx 1 root root 0 abr  3 14:19 sr0 -> ../devices/pci0000:00/0000:00:01.1/ata2/host1/target1:0:0/1:0:0:0/block/sr0
lrwxrwxrwx 1 root root 0 abr  3 13:59 vda -> ../devices/pci0000:00/0000:00:06.0/virtio2/block/vda
```

**procfs** y `/proc`
Contienen información sobre el sistema, como información de la memoria, cpu etc. Algunas configuracionenes es posible cambiarlas 'on the fly' de forma que cuenta con el cambio despues de un reinicio.

También contiene información sobre las peticiones de interrupción los Accesos Directos a Memoria (DMA) y los puertos de entradas y salidas los interfaces usados por el sistema (I/O port interface)

Podemos ver la información del sistema operativo con:

```bash
cat /proc/version
Linux version 4.4.0-143-generic (buildd@lgw01-amd64-037) (gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.10) ) #169-Ubuntu SMP Thu Feb 7 07:56:51 UTC 2019
```

También se puede mostrar la información de las variables del con el comando `sysctl -a`.

Por ejemplo podemos mostrar las entradas del proceso activo. `$$` es el PID del la consola que esta activa.

```bash
xnoxos@ubuntu:~$ ls -l /proc/$$/ | head -n 15
total 0
dr-xr-xr-x 2 xnoxos xnoxos 0 abr  3 15:13 attr
-rw-r--r-- 1 xnoxos xnoxos 0 abr  3 15:13 autogroup
-r-------- 1 xnoxos xnoxos 0 abr  3 15:13 auxv
-r--r--r-- 1 xnoxos xnoxos 0 abr  3 15:13 cgroup
--w------- 1 xnoxos xnoxos 0 abr  3 15:13 clear_refs
-r--r--r-- 1 xnoxos xnoxos 0 abr  3 15:13 cmdline
-rw-r--r-- 1 xnoxos xnoxos 0 abr  3 15:13 comm
-rw-r--r-- 1 xnoxos xnoxos 0 abr  3 15:13 coredump_filter
-r--r--r-- 1 xnoxos xnoxos 0 abr  3 15:13 cpuset
lrwxrwxrwx 1 xnoxos xnoxos 0 abr  3 15:13 cwd -> /home/xnoxos
-r-------- 1 xnoxos xnoxos 0 abr  3 15:13 environ
lrwxrwxrwx 1 xnoxos xnoxos 0 abr  3 15:13 exe -> /bin/bash
dr-x------ 2 xnoxos xnoxos 0 abr  3 12:29 fd
dr-x------ 2 xnoxos xnoxos 0 abr  3 15:13 fdinfo
```

### `/dev` y el comando **`lsdev`**

El directorio [**`/dev`**][4] contiene los archivos especiales de dispositivos.

`lsdev` nos muestra la información de `/proc` presentándolo de una forma ordenada. Las columnas que podemos ver son:

- Device
- DMA
- IRQ
- I/O Port

>Es necesario tener instalado el paquete [procinfo][1]

### Módulos del Kernel

En GNU/Linux, el hardware lo administran los drivers del kernel, algunos de los cuales se encuentran integrados (compilados) en el kernel, y otros, en su mayor parte, son módulos independientes. Estos módulos son ficheros que suelen almacenarse en el árbol de directorios `/lib/modules`, y se pueden cargar o descargar para proporcionar acceso al hardware. Normalmente, GNU/Linux carga los módulos que necesita cuando se inicia, pero puede que en ocasiones se necesite cargar módulos manualmente.

### modprobe
El comando **modprobe** sirve para añadir o eliminar modulos del kernel de linux.

Para eliminar manualmente un driver podemos utilizar `modprobe -r` y para mostrar la información podemos utilizar `modprobe -c`. Con la opcion -v podemos ver lo que se ejecutado y con -n las opciones completadas con exito.

### Añadir un modulo:
Vamos a añadir el módulo de bluetooth de este equipo, lo primero es comprobar que tenemos los módulos de los Drivers compilados:

```
xnoxos@ubuntu:~$ ls /lib/modules/6.8.0-40-generic/kernel/drivers/bluetooth/
ath3k.ko        bt3c_cs.ko      btmtk.ko      btrsi.ko    hci_bcm4377.ko
bcm203x.ko      btbcm.ko        btmtksdio.ko  btrtl.ko    hci_nokia.ko
bfusb.ko        btintel.ko      btmtkuart.ko  btsdio.ko   hci_uart.ko
bluecard_cs.ko  btmrvl.ko       btnxpuart.ko  btusb.ko    hci_vhci.ko
bpa10x.ko       btmrvl_sdio.ko  btqca.ko      dtl1_cs.ko  virtio_bt.ko

```

En esta caso vemos que disponemos de varios modules pertenecientes a varios fabricantes, para nuestra prueba cargaremos el módulo del Bluetooth de Atheos _ath3k.ko_ 

Lo ideal es probar que el módulo se puede cargar correctamente con la opción de `–dry-run` y `–verbose` para ver cual serian las acciones a tomar pero sin efectuarlas en realidad 

```
xnoxos@ubuntu:~$ modprobe --dry-run --verbose ath3k
insmod /lib/modules/6.8.0-40-generic/kernel/drivers/bluetooth/ath3k.ko 
```

Una vez comprobado que si añadimos el módulo encontrará todas sus dependencias y no generará ningún error procedemos a cargarlo

`#modprobe ath3k`

Y comprobamos que el módulo esta cargado correctamente listando los módulos con el comando **lsmod**:

```
xnoxos@ubuntu:~ lsmod | grep ath3k
```

También podremos ver la información el módulo con el comando **modinfo**:
```
xnoxos@ubuntu:~$ modinfo ath3k
filename:       /lib/modules/6.8.0-40-generic/kernel/drivers/bluetooth/ath3k.ko
firmware:       ath3k-1.fw
license:        GPL
version:        1.0
description:    Atheros AR30xx firmware driver
author:         Atheros Communications
srcversion:     A6C59810234C24ABA01660F
alias:          usb:v0489pE03Cd*dc*dsc*dp*ic*isc*ip*in*
alias:          usb:v0489pE036d*dc*dsc*dp*ic*isc*ip*in*
alias:          usb:v0489pE02Cd*dc*dsc*dp*ic*isc*ip*in*
alias:          usb:v13D3p3490d*dc*dsc*dp*ic*isc*ip*in*
alias:          usb:v13D3p3487d*dc*dsc*dp*ic*isc*ip*in*
```

Al reiniciar el módulo no se vuelve a cargar, si queremos que los cambios sean permanentes tendremos que añadir el nombre del módulo en el fichero **_`/etc/modules`_** 

**_Quitar módulos_**

Lo primero es ver bajo que nombre está el módulo cargado, para ello disponemos del comando _lsmod_ o podemos ver el contenido del archivo _/proc/modules_. Como ejemplo vamos a quitar el modulo de cdrom de nuestro equipo.

Buscamos el nombre el módulo
```
lsmod |grep cdrom
```

Ahora que sabemos que el nombre del modulo es _cdrom_ procederemos a descargarlo con el siguiente comando:
```
modprobe -r cdrom
```

Al reiniciar el sistema se volverá a cargar el módulo, si queremos evitar que un módulo en concreto se vuelva a cargar deberemos agregar la siguiente línea _blacklist [nombre_modulo]_ en el fichero _`/etc/modprobe.d/blacklist.conf`_  
```
xnoxos@ubuntu:~$ cat /etc/modprobe.d/blacklist.conf 
# This file lists those modules which we don't want to be loaded by
# alias expansion, usually so some other driver will be loaded for the
# device instead.

# evbug is a debug tool that should be loaded explicitly
blacklist evbug

# these drivers are very simple, the HID drivers are usually preferred
blacklist usbmouse
blacklist usbkbd

# replaced by e100
blacklist eepro100

# replaced by tulip
blacklist de4x5

# causes no end of confusion by creating unexpected network interfaces
blacklist eth1394

# snd_intel8x0m can interfere with snd_intel8x0, doesn't seem to support much
# hardware on its own (Ubuntu bug #2011, #6810)
blacklist snd_intel8x0m

# Conflicts with dvb driver (which is better for handling this device)
blacklist snd_aw2

# replaced by p54pci
blacklist prism54
```

Para los módulos que se «autocargan» en el Kernel no les afecta la configuración del fichero blacklist.conf, para evitar su carga se deberá realizar los siguientes pasos:

- Crear el fichero _/etc/modprobe.d/[nombre_modulo].conf_ y escribir la línea _blacklist [nombre_modulo]_
- Ejecutar _depmod -ae_
- Recrear el initrd con el comando _update-initramfs –_u

### lsmod

Con el comando **lsmod** podemos ver los módulos que hay actualmente cargados en el Kernel.

>Nos muestra la información formateada del archivo `/proc/modules`.

Con el comando `modinfo` podemos ver la información de un modulo. Podemos localizar los ficheros de modulos en la ruta `cd /lib/modules/$(uname -r)`

>`$(uname)- r` nos devuelve la version del kernel de nuestro sistema

```bash
xnoxos@ubuntu:~$ modinfo xor
filename:       /lib/modules/4.4.0-143-generic/kernel/crypto/xor.ko
license:        GPL
srcversion:     C02DF7938B1596D55158340
depends:
retpoline:      Y
intree:         Y
vermagic:       4.4.0-143-generic SMP mod_unload modversions 686 retpoline
...
```

### lspci y lsusb

Son herramientas para obtener información sobre los dispositivos pci y usb.

El comando lspci busca información en el fichero `/usr/share/misc/pci.ids`, que contiene la lista de los IDs conocidos (vendors, devices, classes, and subclasses) [info][3]

```bash
xnoxos@ubuntu:~$ lspci
00:00.0 Host bridge: Intel Corporation 440FX - 82441FX PMC [Natoma] (rev 02)
00:01.0 ISA bridge: Intel Corporation 82371SB PIIX3 ISA [Natoma/Triton II]
00:01.1 IDE interface: Intel Corporation 82371SB PIIX3 IDE [Natoma/Triton II]
00:01.3 Bridge: Intel Corporation 82371AB/EB/MB PIIX4 ACPI (rev 03)
00:02.0 VGA compatible controller: Device 1234:1111
00:03.0 Ethernet controller: Red Hat, Inc. Virtio network device
00:04.0 USB controller: NEC Corporation uPD720200 USB 3.0 Host Controller (rev 03)
00:05.0 Unclassified device [00ff]: Red Hat, Inc. Virtio memory balloon
00:06.0 SCSI storage controller: Red Hat, Inc. Virtio block device
```

```bash
xnoxos@ubuntu:~$ lsusb
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 002: ID 0627:0001 Adomax Technology Co., Ltd
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

## 101.2 Arranque del sistema

Importancia: 3
Descripción: Los candidatos deberán ser capaces de guiar el sistema a lo largo del proceso de arranque.

**Áreas de conocimiento clave:**

- Proporcionar comandos comunes al gestor de arranque y opciones al kernel en el momento del arranque.
- Demostrar conocimiento de la secuencia de arranque desde BIOS/UEFI hasta la finalización del arranque.
- Comprensión de SysVinit y systemd.
- Conocimiento de Upstart.
- Comprobar los eventos de arranque en los archivos de registro.
- Lista parcial de los archivos, términos y utilidades utilizados:

```bash
dmesg
journalctl
BIOS
UEFI
bootloader
kernel
initramfs
init
SysVinit
systemd
```

-----

### Secuencia de arranque

La secuencia de arranque (Boot loader) es la que se encarga de realizar los procesos iniciales cuando el sistema es arrancado o reiniciado.

El programa de arranque reside en la partición del sistema y es invocado antes de iniciar el SO.

#### Cargadores de arranque (boot loader)

[GRUB2][6] (GRand Unified Bootloader, Cargador de arranque unificado

Son particiones que contienen ficheros de configuración para indicar donde se arranca el sistema.

Cuando GRUB2 lee el archivo de configuración, muestra un menu para seleccionar el sistema que se quiere arrancar.

`DMESG`: Se utiliza para mostrar o controlar el buffer del anillo del kernel.(es el antiguo comando, ahora se utiliza journalctl)

`journalctl`: utilidad para mostrar el buffer del anillo del kernel, podemos ver todos los elementos que han sido cargados en la memoria. Con `journalctl -k` mostramos toda la información


### Tipos de arranque

`init`: es el sistema de inicialización.

```
kernel -> /sbin/init -> /etc/inittab
```

```
/etc/inittab
<identificador>:<runlevel>:<accion>:<proceso>
```

`sysVinit`:
El fichero  `/etc/inittab`  era el el antiguo fichero utilizado por el demonio System V init(8).
El demonio Upstart init(8) no usa este fichero,  sin embargo lee la configuracion de los ficheros alojados el el directorio /etc/init. 

`śystemd`: es el gestor de systema y de servicios, es el primer proceso en arrancar y tiene el PID (1)

Ejemplo del proceso del sistema de arranque:

1. La partición de arranque es encontrada.
2. Kernel y RAM incial son cargadas.
3. El kernel carga los drivers iniciales y configura las herramientas desde la RAM.
4. El kernel toma el control del sistema a través de `/sbin/init`.
5. init realiza alguna tareas de mantenimiento desde `/etc/rc.d/rc.sysinit`
6. initi lee la linea por defecto de inidefault en /etc/inittab y entra en el runlevel 3

Estos ficheros se encuentran en diferentes localizaciones dependiendo de la distribución

- Distribuciones basdas en Red Hat: /etc/rc.d/
- Distribuciones basadas en debian: /etc/init.d/

**rc** = run commands, cada una de las carpetas equivale a los diferentes niveles de arranque.

Mostramos los elmentos de arranque del nivel de arranque 3(todos son enlaces simbolicos a los ficheros) Se cargaran en el orden que aparecen.

```console
xnoxos@Lenovo-ideapad-710S-Plus-13IKB  ~/test_command  ls /etc/rc3.d
S01acpid             S01cups-browsed  S01rsyslog
S01anacron           S01dbus          S01saned
S01apport            S01docker        S01speech-dispatcher
S01avahi-daemon      S01gdm3          S01spice-vdagent
S01binfmt-support    S01grub-common   S01unattended-upgrades
S01bluetooth         S01irqbalance    S01uuidd
S01console-setup.sh  S01kerneloops    S01whoopsie
S01cron              S01plymouth
S01cups              S01rsync
```

`upstart`: es el demononio de arranque de ubunto desarrollado en 2006 y mas adelante utilizado en distribuciones Red Hat, debian y fedora. 

Ha diferencia de init, **upstart** ofrece arranques de servicios asincronos, reduciendo el tiempo de arranque.
### init vs upstart
Hemos sacado mucho partido a sysvinit en Debian, pero sus límites se han hecho notar desde hace tiempo; de hecho, fueron estos límites los que llevaron a escribir upstart en primer lugar.

Sysvinit carece de supervisión de servicios. Si bien /etc/inittab proporciona esta capacidad, la gestión de /etc/inittab es bastante restrictiva. Upstart elimina la necesidad de un manejo complicado de archivos PID para todos los servicios. Hay complementos que permiten realizar la supervisión de servicios sobre sysvinit, como runit, pero está claro que son complementos.

Sysvinit no realiza un seguimiento de las dependencias entre servicios. Insserv/startpar lo proporciona sobre sysvinit, pero nuevamente se trata de un complemento, y solo maneja dependencias en el momento del arranque/apagado (es decir, durante los cambios de nivel de ejecución) y no puede manejar ninguna interdependencia de servicio complicada en tiempo de ejecución. Upstart expresa esta información en el formato de configuración de servicio nativo, de forma clara y concisa.

Sysvinit requiere scripts de shell complejos para cada servicio. Si bien parte de la complejidad se ha abstraído en ayudantes comunes (funciones lsb; start-stop-daemon), tener que representar el manejo de inicio/parada de cada servicio como un programa es una desventaja grave. Los trabajos de Upstart son simples, declarativos, fáciles de mantener y fáciles de modificar localmente según sea necesario. También eliminan la molestia de larga data de update-rc.d reactivando servicios a espaldas del administrador en la actualización.

Sysvinit es lineal. Dejó de ser una buena opción para la gestión de arranque en Debian en el momento en que Debian adoptó udev. Hay muchas condiciones de carrera que persisten en Debian hoy en día cuando se arranca con sysvinit, y aunque pueden ser reparables, la complejidad para arreglarlas con sysvinit es muy alta. Es mejor cambiar a un sistema de inicio que esté diseñado para trabajar junto con el núcleo basado en eventos y udev.


## 101.3 Cambiar los niveles de ejecución / objetivos de arranque y apagar o reiniciar el sistema

La inicialización del sistema es el proceso de ejecutar scripts que preparan el sistema operativo para su funcionamiento. Existen varios estilos diferentes de inicialización del sistema, utilizados en diferentes familias e incluso en diferentes versiones del sistema operativo.

El método clásico de inicialización del sistema operativo es la inicialización estilo SysV (prácticamente no se utiliza en OCLinux moderno). El demonio clave es init `sbin/init`, que es el proceso principal que inicia todos los demás. Puede ver el árbol de procesos y ver el padre usando el comando `pstree` (para centos necesita instalar el paquete `psmisc`)

La inicialización estilo `SysV` opera con el concepto de nivel de ejecución, que representa los siguientes modos de inicio del sistema operativo:

0 – apagado;
1 – modo de usuario único;
2 – Valor predeterminado de Debian/Ubuntu (GUI o CUI);
3 – RedHat/Suse por defecto (modo CUI_);_
4 – WildCard (modo programado);
5 – RedHat/Suse por defecto (modo GUI_);_
6 – reiniciar.

La configuración de arranque predeterminada se especifica en el archivo `/etc/inittab` (archivo de configuración de inicialización del sistema), por ejemplo:

id** :3: ** `initdefault` :  (el nivel de arranque predeterminado es el tercero);

Todos los scripts utilizados para iniciar servicios se encuentran en el directorio `/etc/init`, por ejemplo:

`/etc/init. d/network restart`  _(reiniciar el servicio de red);_

El directorio `/etc` contiene los directorios `rc0.d`, `rc1.d` (etc.), que contienen conjuntos de scripts (más precisamente, enlaces a scripts) que se utilizan al cambiar a diferentes modos de funcionamiento, por ejemplo en `rc.3.d` hay scripts ejecutándose en el nivel de ejecución **3**.

Algunos scripts (nombre que comienza con "S") inician demonios y otros (nombre que comienza con "K") los detienen.

Para trabajar con niveles de ejecución, utilice los siguientes comandos:

`init` o `telinit`: cambia al modo de inicio;
nivel de ejecución: descubre el modo de funcionamiento actual;
detener: apagar el sistema operativo;
reiniciar: reinicie la PC;
Apagar: apaga la PC.

Para controlar demonios, use el comando `service daemon_name` con claves (no todos los demonios pueden tener todos los comandos enumerados en la configuración; a menudo puede ver un script diferido con solo los comandos de inicio y detención):

empezar - empezar;
**estado** - mostrar estado;
detener - detener;
reiniciar - reiniciar;

recargar: recarga el archivo de configuración del servicio.

Un estilo de inicialización más moderno es `systemd`. Ahora se utiliza en la mayoría de las distribuciones de Linux modernas (Centos 7.0 y superiores, Ubuntu 15.10 y superiores), debido a la velocidad de arranque (paralelización del lanzamiento de demonios) y la tolerancia automática a fallas (monitoreo del estado de los demonios). Utiliza el concepto de unidades (`units`), que pueden ser servicios (`.service`), puntos de montaje (`.mount`), dispositivos (`.device`) o sockets (`.socket`).

Los módulos (unidades) creados automáticamente después de instalar paquetes de software se encuentran en el directorio `/usr/lib/systemd/`. También puede colocar unidades en el directorio `/etc/systemd/system/` (para el sistema operativo en su conjunto) o `/etc/systemd/user/` (para los usuarios).

Para administrar unidades, use la utilidad systemctl, por ejemplo:

- **`systemctl list-units`** (mostrar unidades en ejecución);
- **`systemctl start network.service`** (iniciar demonio de red);
- **`systemctl status crond`** (muestra el estado del demonio del programador).

En lugar de nivel de ejecución, `systemd` utiliza el concepto de objetivo, pero a diferencia de los niveles de ejecución, no están numerados y algunos de ellos pueden ejecutarse simultáneamente; Los destinos son compatibles con la inicialización de sysV, por lo que puede usar el comando `telinit` para cambiar a un modo de ejecución diferente.

La utilidad systemctl también se utiliza para controlar los modos de funcionamiento, por ejemplo:

`systemctl isolate reboot.target` (ejecutar reinicio objetivo);
`systemctl set-default -f multi-user.target` (establece el destino multiusuario como modo de inicio predeterminado);
También puedes usar systemctl para controlar la energía, por ejemplo:

**`systemctl restart`** _ (reiniciar PC);_
**`systemctl poweroff`**_ (apaga la PC)._

Una característica importante de systemd es el sistema de registro flexible, que recopila información de varias fuentes y la asocia con varias unidades. Ejemplos de su uso:

`journalctl –f` _ (ver mensajes en tiempo real);_
`journalctl -n10` _ (ver los 10 últimos mensajes);_
`journalctl _UID=70` _ (muestra todos los mensajes, incluido el usuario con ID=70);_

___

Desde una perspectiva histórica, observamos que el sistema de inicialización del sistema es un advenedizo, que depende de su trabajo de los eventos que ocurren en el sistema operativo. Se usó en ubuntu desde la versión 6.10 a la 15.04, y en muchas otras distribuciones que ahora usan systemd.

`Upstart` opera con los conceptos de un servicio, que se mantiene en un modo de operación constante, y una tarea, que se realiza una sola vez. Durante el proceso de inicialización, `upstart` lee la configuración de los archivos de configuración (trabajos) en el directorio `/etc/init/`.

Cada tarea representa scripts para iniciar demonios con diferentes criterios y condiciones de ejecución.

Los niveles de inicialización o modos de funcionamiento utilizados son los mismos que en el `sysV` clásico, por lo que los comandos `runlevel` y `telinit` siguen funcionando. La sintaxis para gestionar energía y servicios también es similar a la clásica.

El nivel de inicialización predeterminado se especifica en el archivo `/etc/init/rc-sysinit.conf`

Para controlar la inicialización de estilo advenedizo, utilice la utilidad `initctl`, por ejemplo:

`initctl start networking` _(iniciar el servicio de red);_
`initctl list` _(mostrar una lista de servicios);_