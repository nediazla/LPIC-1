# 109.2. Configuración básica de red

**Puntuación:** 4

**Descripción:** Los candidatos deben ser capaces de ver, cambiar y verificar la configuración de los hosts cliente.

**Áreas clave de conocimiento:**

* Configurar interfaces de red de forma manual y automática
* Configuración básica de host TCP/IP
* Establecer una ruta predeterminada

**Términos y utilidades:**

* /etc/hostname
* /etc/hosts
* /etc/nsswitch.conf
* ifconfig
* ifup
* ifdown
* ip
* route
* ping

En este tutorial, aprenderemos cómo hacer que las configuraciones de red sean persistentes en Linux.

### ifconfig

La utilidad **ifconfig** (**configuración de interfaz)** se utiliza para configurar o ver la configuración de una interfaz de red mediante la interfaz de línea de comandos (CentOS6).

```
ifconfig [...OPTIONS] [INTERFACE]
```

El comando “**ifconfig**” sin opción muestra la información de configuración de red actual:

```
[root@centos6-1 ~]# 
[root@centos6-1 ~]# ifconfig
eth0      Link encap:Ethernet  HWaddr 00:0C:29:6D:D2:C5  
          inet addr:172.16.43.137  Bcast:172.16.43.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:fe6d:d2c5/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:235444 errors:0 dropped:0 overruns:0 frame:0
          TX packets:81562 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:309940884 (295.5 MiB)  TX bytes:5027974 (4.7 MiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:2964 errors:0 dropped:0 overruns:0 frame:0
          TX packets:2964 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:386271 (377.2 KiB)  TX bytes:386271 (377.2 KiB)
```

> el lo es el adaptador de bucle invertido, con dirección IP 127.0.0.1 que utiliza el sistema operativo para sus propias comunicaciones internas.

| ifconfig option | description                                                  |
| --------------- | ------------------------------------------------------------ |
| -a              | display all the interfaces available, even if they are down. |
| -s              | display a short list (like netstat -i)                       |
| -v              | be more verbose for some error conditions                    |

Podemos usar ifconfig para cambiar las configuraciones de red, pero esto requerirá acceso root:

| Comando ifconfig | Descripción |
| --------------------------------------------------- | -------------------- |
| ifconfig eth0 172.16.43.155 | Asignar una dirección IP |
| ifconfig eth0 netmask 255.255.255.224 | Asignar una máscara de red |
| ifconfig eth0 172.16.43.155 netmask 255.255.255.224 | Asignar una IP, máscara de red |

ifconfig puede usarse para activar y desactivar una interfaz. Para ello, se requiere acceso root:

* ifconfig _interface_ up: se usa para activar el controlador de la interfaz dada.
* ifconfig _interface _down: se usa para desactivar el controlador de la interfaz dada.

Si todavía usas ifconfig, ¡estás en el pasado! El comando ifconfig está obsoleto y se ha reemplazado por el comando ip.

### ifup, ifdown

Al igual que los comandos anteriores, ifup e ifdown se utilizan para habilitar y deshabilitar una interfaz.

```
[root@centos6-1 ~]# ifdown eth0
Device state: 3 (disconnected)
[root@centos6-1 ~]# ifup eth0
Active connection state: activating
Active connection path: /org/freedesktop/NetworkManager/ActiveConnection/10
state: activated
Connection activated
```

### Puerta de enlace

Una ventaja de dividir las direcciones IP en clases y subredes es controlar las transmisiones. Esto se realiza mediante la máscara de subred. Pero, ¿qué ocurre si dos computadoras de dos redes diferentes desean comunicarse entre sí?

Una **puerta de enlace** es un nodo o enrutador que actúa como punto de acceso para transferir datos de red desde redes locales a redes remotas.

La dirección que se utiliza como puerta de enlace para acceder a otras redes fuera de nuestra red local se denomina **puerta de enlace predeterminada**. Hay diferentes maneras de configurar una puerta de enlace.

### Archivos de configuración de red

La mala noticia es que todas las configuraciones que hemos realizado con el comando `ifconfig` no son persistentes y desaparecen al reiniciar el sistema o al activarse y desactivarse la interfaz. Por lo tanto, debemos encontrar una manera de que nuestra configuración sea persistente, y la única manera de lograrlo es mediante archivos de configuración de red.

Desafortunadamente, los archivos de configuración de red en Linux se ubican en diferentes lugares y dependen de la distribución de la que estemos hablando.

Sistemas basados ​​en Red Hat

En Red Hat, CentOS y Fedora, los archivos se encuentran en `/etc/sysconfig/network-scripts/`.

```
[root@server1 ~]# ls /etc/sysconfig/network-scripts/
ifcfg-eth0   ifdown-ippp    ifdown-sit     ifup-ib     ifup-post      init.ipv6-global
ifcfg-lo     ifdown-ipv6    ifdown-tunnel  ifup-ippp   ifup-ppp       net.hotplug
ifdown       ifdown-isdn    ifup           ifup-ipv6   ifup-routes    network-functions
ifdown-bnep  ifdown-post    ifup-aliases   ifup-isdn   ifup-sit       network-functions-ipv6
ifdown-eth   ifdown-ppp     ifup-bnep      ifup-plip   ifup-tunnel
ifdown-ib    ifdown-routes  ifup-eth       ifup-plusb  ifup-wireless


```

y la puerta de enlace predeterminada se configura a través del archivo: `/etc/sysconfig/network`

```
[root@server1 Desktop]# cat /etc/sysconfig/network
NETWORKING=yes
HOSTNAME=server1
GATEWAY=172.16.43.2
```

Echemos un vistazo al archivo de configuración eth0:

```
[root@server1 ~]# cat  /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE="eth0"
BOOTPROTO=none
IPV6INIT="yes"
NM_CONTROLLED="yes"
ONBOOT="yes"
TYPE="Ethernet"
UUID="8f5774e8-607e-4629-8d54-a9cc4d12e851"
IPADDR=172.16.43.127
PREFIX=24
GATEWAY=172.16.43.2
DEFROUTE=yes
IPV4_FAILURE_FATAL=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
NAME="System eth0"
HWADDR=00:0C:29:6D:D2:C5
DNS1=8.8.8.8
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
LAST_CONNECT=1582381005
```

Sistemas basados ​​en Debian

En sistemas basados ​​en Debian, como Ubuntu, el archivo principal de configuración de red se encuentra en `/etc/network/interfaces` y no existe un archivo separado para la configuración de la puerta de enlace.

```
auto lo
iface lo inte loopback

auto eth0
#ifconfig eth0 inet dhcp
iface eth0 inet static 
address 172.16.43.135
netmask 255.255.255.0
gateway 172.16.43.2
dns-nameservers 8.8.8.8
```

Podría existir el directorio `/etc/network/interfaces.d` para los archivos de configuración.

> Los comandos ifdown e ifup usan este archivo de configuración.

#### Archivo de configuración DNS

### /etc/resolv.conf:

Como habrás notado, la configuración DNS se encuentra en el mismo archivo que la configuración de la interfaz, pero existe otro lugar en Linux que contiene información DNS, `/etc/resolv.conf`:

```
[root@server1 Desktop]# cat /etc/resolv.conf 
# Generated by NetworkManager
nameserver 8.8.8.8
```

Nuevamente, la configuración de este archivo no es permanente y no se recomienda modificarlo manualmente, excepto para pruebas temporales.

### nombre de host

**Nombre de host** es el programa que se utiliza para configurar o mostrar el nombre actual del host, dominio o nodo del sistema (usamos centOS).

```
[root@server1 Desktop]# hostname
server1
[root@server1 Desktop]# hostname centos6-1
[root@server1 Desktop]# hostname
centos6-1
```

El nuevo nombre de host aparecerá al abrir una nueva terminal, pero desaparecerá al reiniciar el sistema. Para configurar el nombre de host de forma permanente, hay un par de lugares que deben modificarse:

1. `/etc/hostname` (Ubuntu) O `/etc/sysconfig/network` (CentOS)
2. `/etc/hosts` (Ubuntu y CentOS)

### /etc/hostname

/etc/hostname contiene el nombre de la máquina y es uno de los archivos de configuración que deben modificarse para que el nuevo nombre de host sea persistente en sistemas basados ​​en Debian (ubuntu16 aquí).

```
root@ubuntu16-1:~# cat /etc/hostname 
ubuntu16-1
```

### /etc/sysconfig/network

Otro lugar que contiene el nombre de host y que debe cambiarse en los sistemas basados ​​en RedHat para tener un nombre de host persistente (CentOS6)

```
[root@server1 Desktop]# cat /etc/sysconfig/network
NETWORKING=yes
HOSTNAME=server1   ###<---- change it to centos6-1 in our ex
GATEWAY=172.16.43.2
```

### /etc/hosts

 **/etc/hosts** es un archivo del sistema operativo que traduce nombres de host o de dominio a direcciones IP. Su función es similar a la del DNS. Se puede usar para realizar pruebas o cuando el servidor DNS no está disponible. Recuerde que tiene mayor prioridad que el DNS (lo que significa que el sistema operativo primero consulta el archivo /etc/hosts para obtener la dirección IP de un host; si no lo consigue, consulta al servidor DNS).

```
[root@server1 Desktop]# cat /etc/hosts
127.0.0.1	centos6-1 localhost localhost.localdomain localhost4 localhost4.localdomain4
::1	server1	localhost localhost.localdomain localhost6 localhost6.localdomain6

```

Para que el nuevo nombre de host sea permanente, se debe modificar otro archivo.

### route

Todos los dispositivos de red, ya sean hosts, enrutadores u otros tipos de nodos de red, como impresoras conectadas a la red, deben decidir dónde enrutar los paquetes de datos TCP/IP. La tabla de enrutamiento proporciona la información de configuración necesaria para tomar estas decisiones. El comando **route** se utiliza para ver y modificar la tabla de enrutamiento del kernel.

El comando route crea una configuración temporal; en su lugar, utiliza los archivos de configuración.

Al ejecutar el comando **route** sin ninguna opción, se muestran las entradas de la tabla de enrutamiento:

```
[root@server1 Desktop]# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
172.16.43.0     *               255.255.255.0   U     1      0        0 eth0
default         172.16.43.2     0.0.0.0         UG    0      0        0 eth0
```

Esto nos muestra la configuración actual del sistema. Si un paquete llega al sistema y tiene un destino en el rango **172.16.43.0** a **172.16.43.255**, se reenvía a la puerta de enlace **\***, que es **0.0.0.0**, una dirección especial que representa un destino no válido o inexistente. Por lo tanto, en este caso, nuestro sistema no enrutará estos paquetes.

Si el destino no está en este rango de direcciones IP, se reenvía a la puerta de enlace predeterminada (en este caso, **172.16.43.2**), y ese sistema determinará cómo reenviar el tráfico al siguiente paso hacia su destino.

Por defecto, el comando `route` muestra el nombre del host en su salida. Podemos solicitarle que muestre la dirección IP numérica usando la opción `-n`:

```bash
[root@server1 Desktop]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
172.16.43.0     0.0.0.0         255.255.255.0   U     1      0        0 eth0
0.0.0.0         172.16.43.2     0.0.0.0         UG    0      0        0 eth0
```

 El siguiente comando de adición de ruta establecerá la puerta de enlace predeterminada como 172.16.43.2:

```
[root@server1 ~]# route add default gw 172.16.43.1
[root@server1 ~]# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
172.16.43.0     *               255.255.255.0   U     0      0        0 eth0
link-local      *               255.255.0.0     U     1002   0        0 eth0
default         172.16.43.1     0.0.0.0         UG    0      0        0 eth0
default         172.16.43.2     0.0.0.0         UG    0      0        0 eth0
```

Utilice la ruta del para eliminar:

```
[root@server1 ~]# route del  default gw 172.16.43.1
[root@server1 ~]# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
172.16.43.0     *               255.255.255.0   U     0      0        0 eth0
link-local      *               255.255.0.0     U     1002   0        0 eth0
default         172.16.43.2     0.0.0.0         UG    0      0        0 eth0
```

El kernel mantiene la información de la caché de enrutamiento para enrutar los paquetes más rápido. Podemos listar la información de la caché de enrutamiento del kernel usando la opción -C.

`netstat -rn` también muestra la tabla de enrutamiento.

### ip

Este comando reemplaza al antiguo y obsoleto comando ifconfig. Sin embargo, el comando **ifconfig** sigue funcionando y disponible para la mayoría de las distribuciones de Linux.

Se puede usar para asignar y eliminar direcciones, activar o desactivar interfaces, manipular el enrutamiento y mucho más.

```
ip [ OPTIONS ] OBJECT { COMMAND | help }
```

| Ejemplo de comando ip | Descripción |
| ------------------------------------------------------ | ------------------------------------------------------- |
| ip address show | Mostrar todas las direcciones IP asociadas a todos los dispositivos de red |
| ip address show eth0 | Ver la información de cualquier interfaz |
| ip addr add 192.168.50.5/24 dev eth0 | Asignar una dirección IP a una interfaz específica |
| ip addr del 192.168.50.5/24 dev eth0 | Eliminar una dirección IP |
| ip link show | Mostrar interfaces de red |
| ip link set eth0 up | Habilitar interfaz de red |
| ip link set eth0 down | Deshabilitar interfaz de red |
| ip route show | Mostrar información de la tabla de enrutamiento |
| ip route add 10.10.20.0/24 via 192.168.50.100 dev eth0 | Agregar ruta estática |
| ip route del 10.10.20.0/24 | Eliminar ruta estática |

> Toda la configuración anterior se perderá al reiniciar el sistema. En su lugar, utilice los archivos de configuración.

### ping

El comando `ping` es una de las utilidades más utilizadas para solucionar, probar y diagnosticar problemas de conectividad de red.

Ping funciona enviando uno o más paquetes de solicitud de eco ICMP (Protocolo de mensajes de control de Internet) a una dirección IP de destino específica en la red y espera una respuesta. Cuando el destino recibe el paquete, responde con una respuesta de eco ICMP.

```
[root@server1 ~]# ping 8.8.8.8 -c3
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=128 time=45.0 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=128 time=47.0 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=128 time=40.2 ms

--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2043ms
rtt min/avg/max/mdev = 40.262/44.107/47.040/2.851 ms
```

Con el comando `ping`, podemos determinar si una IP de destino remota está activa o inactiva. También puede calcular el retardo de ida y vuelta en la comunicación con el destino y comprobar si hay pérdida de paquetes.

| comando ping | descripción |
| ------------------- | -------------------------------------------------- |
| **-n** | Solo salida numérica. No intente resolver el nombre de host |
| **-i intervalo** | Segundos de intervalo de espera entre el envío de cada paquete |
| **-I interfaz** | Establecer la dirección de origen en la dirección de interfaz especificada |
| **-a** | Ping audible |

> Para entornos IPv6, utilice el comando ping6.

### /etc/nsswitch

Este archivo determina dónde encuentra el sistema información como nombres de host, contraseñas y números de protocolo:

A continuación, se muestra un fragmento de un archivo de ejemplo /etc/nsswitch.conf

```
passwd:   files nis
group:    files nis

hosts:    files dns myhostname
```

En este ejemplo, la información del usuario (los servicios de contraseña y grupo) proviene primero de "archivos" (como /etc/passwd o /etc/group). Si no se encuentran entradas allí, se utilizará una consulta a un servidor NIS (configurado en otro lugar).

La información del host proviene primero de /etc/hosts (archivos), luego de un servidor DNS (dns) y, si ninguno de estos funciona, se utiliza al menos un recurso alternativo de "myhostname" para que la máquina local tenga _algún_ nombre.

> La simplicidad reside en la regla "y si eso no funciona". Cuando se listan varios servicios, se prueban en orden, y un servicio puede tener éxito o fallar. Si falla, se prueba el siguiente, y así sucesivamente.

Eso es todo.

### Nomenclatura consistente de dispositivos de red

Red Hat Enterprise Linux ofrece métodos para una nomenclatura consistente y predecible de dispositivos de red para interfaces de red. Estas funciones modifican el nombre de las interfaces de red en un sistema para facilitar su localización y diferenciación.

Tradicionalmente, las interfaces de red en Linux se enumeran como `eth[0123…]`, pero estos nombres no corresponden necesariamente a las etiquetas reales del chasis. Las plataformas de servidor modernas con múltiples adaptadores de red pueden encontrar una nomenclatura no determinista y contraria a la intuición para estas interfaces. Esto afecta tanto a los adaptadores de red integrados en la placa base (_Lan-on-Motherboard_ o _LOM_) como a los adaptadores adicionales (monopuerto y multipuerto).

En Red Hat Enterprise Linux, **udev** admite diversos esquemas de nomenclatura. El valor predeterminado es asignar nombres fijos basados ​​en el firmware, la topología y la información de ubicación. Esto tiene la ventaja de que los nombres son completamente automáticos y predecibles, se mantienen fijos incluso si se añade o se quita hardware (no se realiza ninguna reenumeración) y el hardware dañado se puede reemplazar sin problemas. La desventaja es que a veces son más difíciles de leer que los nombres eth0 o wlan0 que se usan tradicionalmente. Por ejemplo: enp5s0.

* Para deshabilitar esto (sin embargo, no se recomienda agregar `net.ifnames=0` y `biosdevname=0` como valores de parámetros del kernel a la variable `GRUB_CMDLINE_LINUX`).


- [https://developer.ibm.com/tutorials/l-lpic1-109-2/](https://developer.ibm.com/tutorials/l-lpic1-109-2/)
- [https://www.computerhope.com/unix/uifconfi.htm](https://www.computerhope.com/unix/uifconfi.htm)
- [https://www.tecmint.com/ifconfig-command-examples/](https://www.tecmint.com/ifconfig-command-examples/)
- [https://www.geeksforgeeks.org/ifconfig-command-in-linux-with-examples/](https://www.geeksforgeeks.org/ifconfig-command-in-linux-with-examples/)
- [https://www.unixmen.com/how-to-find-default-gateway-in-linux/](https://www.unixmen.com/how-to-find-default-gateway-in-linux/)
- [https://jadi.gitbooks.io/lpic1/content/1092\_basic_network_configuration.html](https://jadi.gitbooks.io/lpic1/content/1092\_basic_network_configuration.html)
- [https://www.tecmint.com/setup-local-dns-using-etc-hosts-file-in-linux/](https://www.tecmint.com/setup-local-dns-using-etc-hosts-file-in-linux/)
- [https://opensource.com/business/16/8/introduction-linux-network-routing](https://opensource.com/business/16/8/introduction-linux-network-routing)
- [https://www.computerhope.com/unix/route.htm](https://www.computerhope.com/unix/route.htm)
- [https://www.thegeekstuff.com/2012/04/route-examples/](https://www.thegeekstuff.com/2012/04/route-examples/)
- [https://linuxize.com/post/linux-ip-command/](https://linuxize.com/post/linux-ip-command/)
- [https://www.geeksforgeeks.org/ip-command-in-linux-with-examples/](https://www.geeksforgeeks.org/ip-command-in-linux-with-examples/)
- [https://www.tecmint.com/ip-command-examples/](https://www.tecmint.com/ip-command-examples/)
- [https://linuxize.com/post/linux-ping-command/](https://linuxize.com/post/linux-ping-command/
- [https://linux.die.net/man/8/ping](https://linux.die.net/man/8/ping)
- [https://developers.redhat.com/blog/2018/11/26/etc-nsswitch-conf-non-complexity/](https://developers.redhat.com/blog/2018/11/26/etc-nsswitch-conf-non-complexity/)
- [https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/networking_guide/ch-consistent_network_device_naming](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/networking_guide/ch-consistent_network_device_naming)
