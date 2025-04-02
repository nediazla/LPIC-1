# 109.3. Solución básica de problemas de red

**Posición:** 4

Descripción: Los candidatos deben ser capaces de solucionar problemas de red en hosts cliente.

**Áreas clave de conocimiento:**

* Configurar de forma manual y automática interfaces de red y tablas de enrutamiento, incluyendo agregar, iniciar, detener, reiniciar, eliminar o reconfigurar interfaces de red.
* Cambiar, ver o configurar la tabla de enrutamiento y corregir manualmente una ruta predeterminada incorrecta.
* Depurar problemas asociados con la configuración de red.

**Términos y utilidades:**

* ifconfig
* ip
* ifup
* ifdown
* route
* host
* hostname
* dig
* netstat
* ping
* ping6
* traceroute
* traceroute6
* tracepath
* tracepath6
* netcat

Hasta ahora, hemos aprendido los fundamentos de los protocolos de Internet y nos hemos familiarizado con algunos archivos y utilidades de configuración de red. Lo cierto es que a veces las cosas no funcionan como esperábamos y es necesario solucionar el problema. En esta sección, intentaremos mostrar algunos pasos para resolver el problema y, además, se presentarán nuevos comandos.

### ifconfig & ip (problemas de interfaz o dirección IP)

El primer comando que hemos aprendido es ifconfig. A veces, puede haber una interfaz inactiva que no aparezca en los resultados:

```
root@ubuntu16-1:~# ifconfig
lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:6 errors:0 dropped:0 overruns:0 frame:0
          TX packets:6 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:318 (318.0 B)  TX bytes:318 (318.0 B)
```

Utilice -a con el comando ifconfig o ip en su lugar:

```
root@ubuntu16-1:~# ifconfig -a
ens33     Link encap:Ethernet  HWaddr 00:0c:29:e2:1b:3e  
          BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:44 errors:0 dropped:0 overruns:0 frame:0
          TX packets:113 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:5169 (5.1 KB)  TX bytes:12521 (12.5 KB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:6 errors:0 dropped:0 overruns:0 frame:0
          TX packets:6 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:318 (318.0 B)  TX bytes:318 (318.0 B)

root@ubuntu16-1:~# ip a s
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast state DOWN group default qlen 1000
    link/ether 00:0c:29:e2:1b:3e brd ff:ff:ff:ff:ff:ff

```

Abra la interfaz con `ifup ens33` o `ifconfig ens33` up y, a continuación, verifique su dirección IP. 

```
root@ubuntu16-1:~# ifup ens33
```

Puedes comprobar tu dirección IP desde la interfaz gráfica o a través de los archivos de configuración. Si tienes activada la asignación automática de IP, usa `dhclient -r` y `dhclient` para liberar y renovar tu dirección IP.

### ping (detección del problema)

El ping es nuestro mejor aliado para solucionar problemas de red.

Comprueba si puedes hacer ping a otro ordenador en la misma red.

```
root@ubuntu16-1:~# ping 172.16.43.127 -c 2
PING 172.16.43.127 (172.16.43.127) 56(84) bytes of data.
64 bytes from 172.16.43.127: icmp_seq=1 ttl=64 time=0.748 ms
64 bytes from 172.16.43.127: icmp_seq=2 ttl=64 time=0.755 ms

--- 172.16.43.127 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.748/0.751/0.755/0.027 ms
```

Con un simple comando ping, podemos determinar si el objetivo está activo o no. Además, es posible que su red cuente con un firewall que filtre los paquetes ICMP. Compruebe primero el firewall del host y luego el firewall de hardware, si lo hay.

```
root@ubuntu16-1:~# ping 172.16.43.126 -c 2
PING 172.16.43.126 (172.16.43.126) 56(84) bytes of data.
From 172.16.43.135 icmp_seq=1 Destination Host Unreachable
From 172.16.43.135 icmp_seq=2 Destination Host Unreachable

--- 172.16.43.126 ping statistics ---
2 packets transmitted, 0 received, +2 errors, 100% packet loss, time 1007ms
pipe 2
```

A veces, podría hacer ping a una dirección IP incorrecta o el servidor podría tener dos interfaces o dos direcciones IP.

### ping6

El comando ping normal solo funciona con direcciones IPv4. Use el comando ping6 para enviar paquetes ICMPv6 ECHO_REQUEST a los hosts de la red desde un host o una puerta de enlace.

### ruta (problemas de puerta de enlace y enrutamiento)

Si no puede acceder a ninguna red, excepto a los equipos que están en la misma subred, debería dudar de su puerta de enlace.

```
root@ubuntu16-1:~# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         172.16.43.2     0.0.0.0         UG    0      0        0 ens33
link-local      *               255.255.0.0     U     1000   0        0 ens33
172.16.43.0     *               255.255.255.0   U     0      0        0 ens33
```

También podemos usar el comando netstat -rn para ver la puerta de enlace actual:

```
root@ubuntu16-1:~# netstat -rn
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         172.16.43.2     0.0.0.0         UG        0 0          0 ens33
169.254.0.0     0.0.0.0         255.255.0.0     U         0 0          0 ens33
172.16.43.0     0.0.0.0         255.255.255.0   U         0 0          0 ens33
```

Si no hubiera una puerta de enlace predeterminada, debería usar `route add default gw x.x.x.x` para agregar una. A continuación, verifique la puerta de enlace y asegúrese de que los paquetes salgan de la interfaz correcta:

```
root@ubuntu16-1:~# ping -c 3 172.16.43.2
PING 172.16.43.2 (172.16.43.2) 56(84) bytes of data.
64 bytes from 172.16.43.2: icmp_seq=1 ttl=128 time=0.434 ms
64 bytes from 172.16.43.2: icmp_seq=2 ttl=128 time=0.379 ms
64 bytes from 172.16.43.2: icmp_seq=3 ttl=128 time=0.166 ms

--- 172.16.43.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2025ms
rtt min/avg/max/mdev = 0.166/0.326/0.434/0.116 ms
```

Todo parece estar bien, pero no se puede acceder a una dirección IP específica en otro edificio. ¡Podría haber problemas de enrutamiento en los routers físicos! ¿Cómo comprobarlo?

### traceroute

El comando traceroute mapea el recorrido que realiza un paquete de información desde su origen hasta su destino. Esta herramienta utiliza mensajes **ICMP**, pero a diferencia de _ping_, identifica todos los routers en la ruta. **traceroute** es útil para solucionar problemas de red, ya que puede ayudar a localizar problemas de conectividad (quizás necesite instalarlo `apt install traceroute`).

```
root@ubuntu16-1:~# traceroute google.com
traceroute to google.com (172.217.18.142), 30 hops max, 60 byte packets
 1  172.16.43.2 (172.16.43.2)  0.272 ms  0.329 ms  0.172 ms
 2  172.16.130.1 (172.16.130.1)  0.969 ms  1.183 ms  1.128 ms
 3  192.168.1.66 (192.168.1.66)  0.448 ms  0.574 ms  0.655 ms
 4  192.168.66.41 (192.168.66.41)  5.123 ms  4.924 ms  7.027 ms
 5  * * *
 6  * * *
 7  10.10.53.93 (10.10.53.93)  14.334 ms  14.304 ms  14.256 ms
 8  10.201.147.214 (10.201.147.214)  19.629 ms  19.580 ms  21.650 ms
 9  10.21.21.22 (10.21.21.22)  19.445 ms  16.203 ms  16.131 ms
10  10.21.21.22 (10.21.21.22)  13.911 ms  13.793 ms  13.723 ms
11  213.202.4.172 (213.202.4.172)  48.527 ms  48.476 ms  48.416 ms
12  134.0.220.62 (134.0.220.62)  48.361 ms  48.285 ms  47.198 ms
13  108.170.246.113 (108.170.246.113)  49.444 ms  49.409 ms 108.170.240.49 (108.170.240.49)  49.677 ms
14  172.253.51.137 (172.253.51.137)  52.925 ms  46.305 ms  45.917 ms
15  arn02s05-in-f142.1e100.net (172.217.18.142)  45.783 ms  45.708 ms  4
```

> use -i para especificar la interfaz a través de la cual traceroute debe enviar paquetes.
### traceroute6

Traceroute con la opción `-6` es compatible con IPv6; en su lugar, podemos usar el comando traceroute6.

¿Qué es la MTU? La unidad máxima de transmisión (MTU) es el tamaño de la unidad de datos de protocolo más grande que se puede comunicar en una sola transacción de capa de red.
### tracepath

Tracepath rastrea una ruta a una dirección de red designada, informando sobre el tiempo de vida (TTL) y las unidades máximas de transmisión (MTU) a lo largo del camino. Este comando puede ser ejecutado por cualquier usuario que no tenga acceso a la línea de comandos.

```
root@ubuntu16-1:~# tracepath google.com
 1?: [LOCALHOST]                                         pmtu 1500
 1:  172.16.43.2                                           0.148ms 
 1:  172.16.43.2                                           0.139ms 
 2:  172.16.130.1                                          4.944ms asymm  1 
 3:  192.168.1.66                                          1.224ms asymm  1 
 4:  192.168.66.41                                         8.198ms asymm  1 
 5:  192.168.198.169                                       5.567ms asymm  1 
 6:  192.168.0.254                                         5.923ms asymm  1 
 7:  10.10.53.93                                          10.018ms asymm  1 
 8:  10.201.147.214                                       11.534ms asymm  1 
 9:  10.21.21.22                                          15.286ms asymm  1 
10:  10.21.21.22                                          11.588ms asymm  1 
11:  213.202.4.172                                        43.501ms asymm  1 
12:  no reply
...
30:  no reply
     Too many hops: pmtu 1500
     Resume: pmtu 1500 

```

Traceroute es básicamente igual que Tracepath, excepto que, por defecto, solo proporciona el valor TTL.

### tracepath6

**tracepath6** es un buen sustituto de **traceroute6**.

### dig

**Dig** (**Domain Information Groper**) es una herramienta de línea de comandos de administración de red para consultar servidores de nombres del **Sistema de Nombres de Dominio** (**DNS**). Es útil para verificar y solucionar problemas de **DNS**, así como para realizar búsquedas de **DNS** y mostrar las respuestas del servidor de nombres consultado.


Por defecto, dig envía la consulta DNS a los servidores de nombres listados en el resolver (/etc/resolv.conf), a menos que se le solicite consultar un servidor de nombres específico.

```
root@ubuntu16-1:~# dig aol.com

; <<>> DiG 9.10.3-P4-Ubuntu <<>> aol.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 51870
;; flags: qr rd ra; QUERY: 1, ANSWER: 5, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;aol.com.			IN	A

;; ANSWER SECTION:
aol.com.		587	IN	A	188.125.72.165
aol.com.		587	IN	A	66.218.87.12
aol.com.		587	IN	A	67.195.231.10
aol.com.		587	IN	A	124.108.115.87
aol.com.		587	IN	A	106.10.218.150

;; Query time: 46 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Mon Feb 24 16:12:46 +0330 2020
;; MSG SIZE  rcvd: 116
```

La salida del comando dig tiene varias secciones, para tener solo la sección de Respuesta use el interruptor corto +:

```
root@ubuntu16-1:~# dig aol.com +short
124.108.115.87
66.218.87.12
188.125.72.165
106.10.218.150
67.195.231.10
```

Para consultar un servidor de nombres específico, utilice `@NameServerIP`:

```
root@ubuntu16-1:~# dig aol.com  @64.6.65.6 +short
106.10.218.150
188.125.72.165
66.218.87.12
124.108.115.87
67.195.231.10
```

### netstat

**netstat** (estadísticas de red) es una herramienta de línea de comandos que muestra las conexiones de red (entrantes y salientes), las tablas de enrutamiento, el número de interfaces de red e incluso las estadísticas de protocolos de red.

> Por defecto, netstat muestra una lista de sockets abiertos. (Un socket es un extremo de un enlace de comunicación bidireccional entre dos programas que se ejecutan en la red). Por ejemplo, X11:

```
root@ubuntu16-1:~# netstat  | grep X11 | head -n3
unix  3      [ ]         STREAM     CONNECTED     34942    @/tmp/.X11-unix/X0
unix  3      [ ]         STREAM     CONNECTED     33525    @/tmp/.X11-unix/X0
unix  3      [ ]         STREAM     CONNECTED     32753    @/tmp/.X11-unix/X0

```

Por lo general, utilizamos una combinación de conmutadores con netstat:

| Ejemplo del comando netstat | Uso |
| ----------------------- | ---------------------------------------------------------- |
| netstat -a | Listado de todos los puertos de escucha de las conexiones TCP y UDP |
| netstat -na | Todos los puertos de escucha, pero con direcciones numéricas |
| netstat -at | Listado de conexiones de puertos TCP |
| netstat -au | Listado de conexiones de puertos UDP |
| netstat -l | Listado de todas las conexiones de escucha (TCP\&UDP) |
| netstat -s | Visualización de estadísticas por protocolo (TCP\&UDP&...) |
| netstat -tp | Visualización del nombre del servicio con PID |
| netstat -rn | Visualización del enrutamiento IP del kernel |
> Use netstat junto con grep para obtener mejores resultados.

### netcat

La utilidad `nc` (o netcat) se utiliza para prácticamente cualquier aplicación que involucre TCP o UDP. Puede abrir conexiones TCP, enviar paquetes UDP, escuchar en puertos TCP y UDP arbitrarios, escanear puertos y gestionar tanto IPv4 como IPv6. A diferencia de Telnet, `nc` se ejecuta correctamente en scripts y separa los mensajes de error en la salida estándar en lugar de enviarlos a la salida estándar, como hace Telnet con algunos.

```
root@ubuntu16-1:~# netcat -l 8888
```



```
root@ubuntu16-1:~# netstat -na | grep 8888
tcp        0      0 0.0.0.0:8888            0.0.0.0:*               LISTEN 
```

El parámetro -l significa que netcat está en modo de escucha (servidor) y el puerto al que escucha es 8888. netcat creará un servidor de sockets y esperará conexiones en el puerto 8888. La terminal permanecerá en espera a que un cliente se conecte al servidor abierto con netcat. Podemos verificar que un servicio host escucha en el puerto 8888. Necesitamos abrir una nueva terminal en la estación host y ejecutar el comando:


- [https://www.cyberciti.biz/faq/howto-test-ipv6-network-with-ping6-command/](https://www.cyberciti.biz/faq/howto-test-ipv6-network-with-ping6-command/)
- [https://www.lifewire.com/traceroute-linux-command-4092586](https://www.lifewire.com/traceroute-linux-command-4092586)
- [https://geek-university.com/linux/traceroute-command/](https://geek-university.com/linux/traceroute-command/)
- [https://www.techwalla.com/articles/differences-between-traceroute-tracepath](https://www.techwalla.com/articles/differences-between-traceroute-tracepath)
- [http://netstat.net/](http://netstat.net)
- [http://journals.ecs.soton.ac.uk/java/tutorial/networking/sockets/definition.html](http://journals.ecs.soton.ac.uk/java/tutorial/networking/sockets/definition.html)
- [https://www.geeksforgeeks.org/netstat-command-linux/](https://www.geeksforgeeks.org/netstat-command-linux/)
- [https://www.tecmint.com/20-netstat-commands-for-linux-network-management/](https://www.tecmint.com/20-netstat-commands-for-linux-network-management/)
- [https://linux.die.net/man/1/nc](https://linux.die.net/man/1/nc)
- [https://www.mvps.net/docs/what-is-netcat-and-how-to-use-it/](https://www.mvps.net/docs/what-is-netcat-and-how-to-use-it/)

.
