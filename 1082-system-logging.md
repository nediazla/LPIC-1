# 108.2. Registro del sistema

**Puntuación: **3

**Descripción:** Los candidatos deben ser capaces de configurar el demonio de syslog. Este objetivo también incluye la configuración del demonio de registro para enviar la salida del registro a un servidor de registro central o aceptarla como tal. Se aborda el uso del subsistema de registro systemd. Además, se incluye el conocimiento de rsyslog y syslog-ng como sistemas de registro alternativos.

**Áreas de conocimiento clave:**

* Configuración del demonio syslog
* Conocimiento de las funciones, prioridades y acciones estándar
* Configuración de logrotate
* Conocimiento de rsyslog y syslog-ng

**Términos y utilidades:**

* syslog.conf
* syslogd
* klogd
* /var/log/
* logger
* logrotate
* /etc/logrotate.conf
* /etc/logrotate.d/
* journalctl
* /etc/systemd/journald.conf
* /var/log/journal/

#### ¿Por qué registrar?

Un sistema Linux tiene muchos subsistemas y aplicaciones en ejecución. Usamos el registro del sistema para recopilar datos sobre nuestro sistema en ejecución desde el momento en que arranca. A veces, simplemente necesitamos saber que todo está bien. Otras veces, usamos estos datos para auditorías, depuración, para saber cuándo un disco u otro recurso se está quedando sin capacidad y para muchos otros fines.

### syslog

Syslog es un estándar para enviar y recibir mensajes de notificación (en un formato específico) desde varios dispositivos de red. Los mensajes incluyen marcas de tiempo, mensajes de eventos, gravedad, direcciones IP del host, diagnósticos y más.

### syslogd

El demonio syslog es un proceso de servidor que proporciona una función de registro de mensajes para aplicaciones y procesos del sistema. Desafortunadamente, el registro de Linux es uno de los aspectos de Linux que se encuentra en fase de transición. La función tradicional de syslog y su demonio syslogd se han complementado con otras funciones de registro como rsyslog, syslog-ng y el subsistema de registro systemd.

### syslog.conf

El archivo syslog.conf es el archivo de configuración principal de **syslogd**. Cada vez que syslogd recibe un mensaje de registro, actúa según el tipo (o función) del mensaje y su prioridad (denominados campos selectores).

```
type (facility).priority (severity)  destination(where to send the log)
```

**Las instalaciones** son simplemente categorías. Algunas instalaciones en Linux son: **`auth, user, kern, cron, daemon, mail, local1, local2, ...`**

* **`auth`**` `: Mensajes de seguridad/autenticación
* **`user `**: Mensajes a nivel de usuario
* **`kern `**: Mensajes del kernel
* **`corn `**: Daemon de reloj
* **`daemon `**: Daemons del sistema
* **`mail `**: Sistema de correo
* **`local0 – local7`**: Instalaciones de uso local

**prioridades **A diferencia de las instalaciones, que no guardan relación entre sí, las prioridades son jerárquicas. Las posibles prioridades en Linux son: **` emerg/panic, alert, crit, err/error, warn/warning, notice, info, debug`**

1. **`emerg: **El sistema no se puede usar
2. **`alert: **Se debe tomar una acción inmediata
3. **`critical: **Condiciones críticas
4. **`err: **Condiciones de error
5. **`warning: **Condiciones de advertencia
6. **`notice: **Condiciones normales pero significativas
7. **`info: **Mensajes informativos
8. **`debug: **Mensajes de depuración

> Si registramos una prioridad específica, también se registrarán los datos más importantes.

**Acción:** Cada línea de este archivo especifica uno o más selectores de facilidad/prioridad seguidos de una acción. En el campo de acción podemos incluir elementos como:

| action   | example           | notes                                                                                                  |
| -------- | ----------------- | ------------------------------------------------------------------------------------------------------ |
| filename | /var/log/messages | Writes logs to specified file                                                                          |
| username | user2             | Will notify that person on the screen                                                                  |
| @ip      | @192.168.10.42    | Will send logs to specified log server and that server decides how to treat logs based on its configs. |

En la siguiente línea syslog.conf, mail.notice es el selector y /var/log/mail es la acción (es decir, “escribir mensajes en /var/log/mail”):

```
mail.notice     /var/log/mail
```

Dentro del selector, «mail» representa la función (categoría del mensaje) y «notice» el nivel de prioridad. Puede ver parte de syslog.conf (CentOS6):

```
#### RULES ####

# Log all kernel messages to the console.
# Logging much else clutters up the screen.
#kern.*                                                 /dev/console

# Log anything (except mail) of level info or higher.
# Don't log private authentication messages!
*.info;mail.none;authpriv.none;cron.none                /var/log/messages

# The authpriv file has restricted access.
authpriv.*                                              /var/log/secure

# Log all the mail messages in one place.
mail.*                                                  -/var/log/maillog


# Log cron stuff
cron.*                                                  /var/log/cron

# Everybody gets emergency messages
*.emerg                                                 *

# Save news errors of level crit and higher in a special file.
uucp,news.crit                                          /var/log/spooler

# Save boot messages also to boot.log
local7.*                                                /var/log/boot.log

```

> `*`: wildcard. que significa "cualquier instalación" o "cualquier prioridad".
>
> Guión -: significa que puede usar la caché de memoria (para no perder tiempo escribiendo constantemente en el disco).
>
> Signo igual =: para registrar SOLO un nivel específico de prioridad. `instalación.=acción de prioridad`

También existe el directorio /etc/rsyslog.d/, y es mejor que los diferentes programas y administradores agreguen sus configuraciones específicas allí, en lugar de editar el archivo de configuración principal (véase Ubuntu 16).

### klogd

¿Cómo se registran los mensajes del kernel durante el arranque, incluso antes de que se monte un sistema de archivos? El kernel almacena los mensajes en un búfer circular en memoria. El demonio `klogd` procesa estos mensajes directamente en una consola, en un archivo como /var/log/dmesg o a través de la función syslog.

### /var/log

Casi todos los archivos de registro se encuentran en el directorio /var/log y sus subdirectorios en Linux (CentOS6).

```
[root@centos6-1 ~]# ls /var/log
anaconda.ifcfg.log    cron              messages-20200217  tallylog
anaconda.log          cron-20200217     ntpstats           vmware-caf
anaconda.program.log  cups              pm-powersave.log   vmware-install.log
anaconda.storage.log  dmesg             ppp                vmware-tools-upgrader.log
anaconda.syslog       dmesg.old         prelink            vmware-vgauthsvc.log.0
anaconda.xlog         dracut.log        sa                 vmware-vmsvc.log
anaconda.yum.log      gdm               samba              wpa_supplicant.log
audit                 httpd             secure             wtmp
boot.log              lastlog           secure-20200217    Xorg.0.log
btmp                  maillog           spice-vdagent.log  Xorg.0.log.old
btmp-20200217         maillog-20200217  spooler            yum.log
ConsoleKit            messages          spooler-20200217   yum.log-20200217
```

Puedes usar tu editor de texto favorito o los comandos less o tail junto con grep para leer estos archivos de registro.

### Creando un receptor de rsyslog

Podemos crear un receptor de rsyslog y capturar los mensajes de registro de otros sistemas. Es bastante sencillo.

```
###CentOS 6
vi /etc/rsyslog.conf

###Change from 
#$UDPServerRun 514
#$ModLoad imtcp
#$InputTCPServerRun 514

### to
$UDPServerRun 514
$ModLoad imtcp
$InputTCPServerRun 514
```

```
###ubuntu 16
vim /etc/default/rsyslog

### Change From
RSYSLOGD_OPTIONS=""

### To
RSYSLOGD_OPTIONS="-r"
```

Y por último, no olvides reiniciar el servicio `systemctl restart rsyslog`.

### journalctl

Systemd también tiene su propio programa de registro llamado **journald**, que almacena información en archivos binarios. No podemos acceder a los archivos de texto (como hicimos con syslog/rsyslog), por lo que debemos usar la herramienta especial journalctl para acceder a ellos (CentOS7).

```
[root@centos7-1 ~]# journalctl 
-- Logs begin at Mon 2020-02-10 02:51:48 EST, end at Tue 2020-02-18 09:19:13 EST. --
Feb 10 02:51:48 localhost.localdomain systemd-journal[106]: Runtime journal is using 8.
Feb 10 02:51:48 localhost.localdomain kernel: Initializing cgroup subsys cpuset
Feb 10 02:51:48 localhost.localdomain kernel: Initializing cgroup subsys cpu
Feb 10 02:51:48 localhost.localdomain kernel: Initializing cgroup subsys cpuacct
Feb 10 02:51:48 localhost.localdomain kernel: Linux version 3.10.0-693.el7.x86_64 (buil
Feb 10 02:51:48 localhost.localdomain kernel: Command line: BOOT_IMAGE=/vmlinuz-3.10.0-
Feb 10 02:51:48 localhost.localdomain kernel: Disabled fast string operations
Feb 10 02:51:48 localhost.localdomain kernel: e820: BIOS-provided physical RAM map:
Feb 10 02:51:48 localhost.localdomain kernel: BIOS-e820: [mem 0x0000000000000000-0x0000
Feb 10 02:51:48 localhost.localdomain kernel: BIOS-e820: [mem 0x000000000009ec00-0x0000
Feb 10 02:51:48 localhost.localdomain kernel: BIOS-e820: [mem 0x00000000000dc000-0x0000
```

Como mencionamos anteriormente, el registro de Linux es uno de los aspectos que está cambiando. Las distribuciones con systemd incluyen journald, aunque algunas aún conservan rsyslog y otras no. Intenta averiguar cuál es el sistema de registro de tu Linux.

### /etc/systemd/journald.conf

El archivo de configuración de journalctl se encuentra en /etc/systemd/journald.conf (CentOS7).

```
[root@centos7-1 ~]# cat  /etc/systemd/journald.conf 
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.
#
# Entries in this file show the compile time defaults.
# You can change settings by editing this file.
# Defaults can be restored by simply deleting this file.
#
# See journald.conf(5) for details.

[Journal]
#Storage=auto
#Compress=yes
#Seal=yes
#SplitMode=uid
#SyncIntervalSec=5m
#RateLimitInterval=30s
#RateLimitBurst=1000
#SystemMaxUse=
#SystemKeepFree=
#SystemMaxFileSize=
#RuntimeMaxUse=
#RuntimeKeepFree=
#RuntimeMaxFileSize=
#MaxRetentionSec=
#MaxFileSec=1month
#ForwardToSyslog=yes
#ForwardToKMsg=no
#ForwardToConsole=no
#ForwardToWall=yes
#TTYPath=/dev/console
#MaxLevelStore=debug
#MaxLevelSyslog=debug
#MaxLevelKMsg=notice
#MaxLevelConsole=info
#MaxLevelWall=emerg
```

### logger

El comando logger de Linux proporciona una manera sencilla de generar algunos registros (centOS6)

```
[root@centos6-1 ~]# logger local1.emerg Hello! This is my log!
```

 y aparecerá en /var/log/syslog (o /var/log/messages):

```
[root@centos6-1 ~]# tail -5 /var/log/messages
Feb 18 06:54:42 server1 NetworkManager[2183]: <info>   nameserver '172.16.43.2'
Feb 18 06:54:42 server1 NetworkManager[2183]: <info>   domain name 'localdomain'
Feb 18 06:54:42 server1 dhclient[29269]: bound to 172.16.43.137 -- renewal in 693 seconds.
Feb 18 06:54:42 server1 nm-dispatcher.action: Script '/etc/NetworkManager/dispatcher.d/13-named' exited with error status 1.
Feb 18 06:59:34 server1 payam: Hello! This is my log!

```

### logrotate

Con la cantidad de registros posibles, necesitamos controlar el tamaño de los archivos de registro. Esto se realiza mediante la utilidad `logrotate`, que suele ejecutarse como una tarea cron.

Los archivos importantes a tener en cuenta son:

* /usr/sbin/logrotate: el comando logrotate (el ejecutable)
* /etc/cron.daily/logrotate: el script de shell que ejecuta logrotate a diario (tenga en cuenta que podría ser /etc/cron.daily/logrotate.cron en algunos sistemas)
* /etc/logrotate.conf: el archivo de configuración de rotación de registros

Otro archivo importante es /etc/logrotate.d, incluido en el proceso mediante esta línea en el archivo /etc/logrotate.conf:

### /etc/logrotate.conf

Utilice el archivo de configuración /etc/logrotate.conf para especificar cómo debe realizarse la rotación y el archivado de registros.

```
[root@centos6-1 ~]# cat /etc/logrotate.conf 
# see "man logrotate" for details
# rotate log files weekly
weekly

# keep 4 weeks worth of backlogs
rotate 4

# create new (empty) log files after rotating old ones
create

# use date as a suffix of the rotated file
dateext

# uncomment this if you want your log files compressed
#compress

# RPM packages drop log rotation information into this directory
include /etc/logrotate.d

# no packages own wtmp and btmp -- we'll rotate them here
/var/log/wtmp {
    monthly
    create 0664 root utmp
	minsize 1M
    rotate 1
}

/var/log/btmp {
    missingok
    monthly
    create 0600 root utmp
    rotate 1
}

# system-specific logs may be also be configured here.
```

Cada archivo de registro se puede manejar diariamente, semanalmente, mensualmente o cuando crezca demasiado.

| parámetro | significado |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| missingok | no escribir un mensaje de error si falta el archivo de registro |
| diario, semanal, mensual | rotar registros diario, semanal, mensual |
| rotar _N_ | conservar los _N_ registros más recientes y eliminar los más antiguos |
| comprimir | comprimir el registro (crea archivos gz) |
| **crear** modo grupo propietario | Inmediatamente después de la rotación (antes de ejecutar el script **postrotate**) se crea el archivo de registro con este acceso y propietario |
| minsize N | Los archivos de registro se rotan cuando superan el tamaño de bytes, pero no antes del intervalo de tiempo especificado adicionalmente (diario,...) |
Este archivo contiene algunas configuraciones predeterminadas y configura la rotación de algunos registros que no pertenecen a ningún paquete del sistema. También utiliza una instrucción `include` para extraer la configuración de cualquier archivo en el directorio `/etc/logrotate.d` (CentOS6).


### /etc/logrotate.d


Cualquier paquete que instalemos que necesite ayuda con la rotación de registros guardará aquí su configuración de Logrotate.

```
[root@centos6-1 ~]# ls /etc/logrotate.d/
ConsoleKit  cups  dracut  httpd  named  ppp  psacct  syslog  wpa_supplicant  yum

[root@centos6-1 ~]# cat /etc/logrotate.d/httpd 
/var/log/httpd/*log {
    missingok
    notifempty
    sharedscripts
    delaycompress
    postrotate
        /sbin/service httpd reload > /dev/null 2>/dev/null || true
    endscript
}
```

Estos son el significado de algunos de estos parámetros:

| parámetro | significado |
| -------------- | --------------------------------------------------------------------------------- |
| missingok | no escribir un mensaje de error si falta el archivo de registro |
| notifempty | no rotar el archivo de registro si está vacío. |
| scripts compartidos | ejecutar scripts **prerotate** y **postrotate** para cada archivo de registro rotado |
| delaycompress | posponer la compresión del archivo de registro anterior hasta el siguiente ciclo de rotación |

¡Eso es todo!

[https://developer.ibm.com/tutorials/l-lpic1-108-2/](https://developer.ibm.com/tutorials/l-lpic1-108-2/)

[https://stackify.com/syslog-101/](https://stackify.com/syslog-101/)

[https://www.ibm.com/support/knowledgecenter/en/SSB23S\_1.1.0.15/gtpc1/hsyslog.html](https://www.ibm.com/support/knowledgecenter/en/SSB23S\_1.1.0.15/gtpc1/hsyslog.html)

[https://linux.die.net/man/5/syslog.conf](https://linux.die.net/man/5/syslog.conf)

[https://www.linuxjournal.com/article/5476](https://www.linuxjournal.com/article/5476)

[https://jadi.gitbooks.io/lpic1/content/1082\_system_logging.html](https://jadi.gitbooks.io/lpic1/content/1082\_system_logging.html)

[https://en.wikipedia.org/wiki/Syslog](https://en.wikipedia.org/wiki/Syslog)

[https://renenyffenegger.ch/notes/Linux/logging/klogd/index](https://renenyffenegger.ch/notes/Linux/logging/klogd/index)

[https://www.tecmint.com/create-centralized-log-server-with-rsyslog-in-centos-7/](https://www.tecmint.com/create-centralized-log-server-with-rsyslog-in-centos-7/)

[https://www.digitalocean.com/community/tutorials/how-to-manage-logfiles-with-logrotate-on-ubuntu-16-04](https://www.digitalocean.com/community/tutorials/how-to-manage-logfiles-with-logrotate-on-ubuntu-16-04)

[https://linux.die.net/man/8/logrotate](https://linux.die.net/man/8/logrotate)

[https://www.digitalocean.com/community/tutorials/how-to-manage-logfiles-with-logrotate-on-ubuntu-16-04](https://www.digitalocean.com/community/tutorials/how-to-manage-logfiles-with-logrotate-on-ubuntu-16-04)

.
