**Peso:** 3

**Descripción:** Los candidatos deben ser capaces de localizar un sistema en un idioma distinto del inglés. Además, deben comprender por qué LANG=C es útil al crear scripts.

**Áreas de conocimiento clave:**

* Configurar la configuración regional y las variables de entorno
* Configurar la configuración de la zona horaria y las variables de entorno

**Términos y utilidades:**

* /etc/timezone
* /etc/localtime
* /usr/share/zoneinfo/
* LC_\*
* LC_ALL
* LANG
* TZ
* /usr/bin/locale
* tzselect
* timedatectl
* date
* iconv
* UTF-8
* ISO-8859
* ASCII
* Unicode

Existen miles de idiomas diferentes en todo el mundo. Los números y las fechas se pueden formatear de forma diferente y existen más de 40 alfabetos. La gente utiliza un reloj de 12 horas o de 24 horas para indicar el tiempo. También existen diferentes sistemas de medición. En esta lección aprenderemos a configurar nuestro sistema Linux para adaptarlo a nuestra configuración regional.

## timezone

El término time zone se puede utilizar para describir varias cosas diferentes, pero principalmente se refiere a la hora local de una región o un país. La hora local dentro de una zona horaria se define por su diferencia con el Tiempo Universal Coordinado (UTC), el estándar horario mundial.

Hay varias utilidades de administración del tiempo disponibles en Linux, como los comandos **date** y **timedatectl** para obtener la zona horaria actual del sistema.

### date

El comando **date **se utiliza para mostrar la fecha y la hora del sistema. El comando date también se utiliza para establecer la fecha y la hora del sistema. De forma predeterminada, el comando de fecha muestra la fecha en la zona horaria en la que está configurado el sistema operativo Unix/Linux. Debe ser el superusuario (root) para cambiar la fecha y la hora. (ubuntu16)

```bash
user1@ubuntu16-1:~$ date
Sun Feb 16 03:27:44 PST 2020
```

`date -u` Muestra la hora en la zona horaria GMT (hora media de Greenwich)/UTC (hora universal coordinada).

```bash
user1@ubuntu16-1:~$ date -u
Sun Feb 16 11:30:09 UTC 2020
```

**Formato de fecha**: FORMAT es una secuencia de caracteres que especifica cómo aparecerá la salida. La sintaxis es `date +% <format-options>` :

| opción de formato de fecha | Propósito de la opción               |
| -------------------------- | ------------------------------------ |
| %Y                         | año                                  |
| %y                         | últimos dos dígitos del año (00..99) |
| %m                         | mes (01..12)                         |
| %d                         | día del mes (p. ej., 01)             |
| %D                         | %D fecha; igual que %m/%d/%y         |
| %H                         | hora (00..23)                        |
| %M                         | minuto (00..59)                      |

```bash
user1@ubuntu16-1:~$ date +"%Y%m%d-%H:%M"
20200216-03:49
```

### timedatectl

Para todas las distribuciones de Linux que utilizan systemd, debe existir un comando timedatectl.

El comando timedatectl nos permite consultar y cambiar la configuración del reloj del sistema y sus ajustes. Podemos utilizar este comando para configurar o cambiar la fecha, hora y zona horaria actuales o habilitar la sincronización automática del reloj del sistema con un servidor NTP remoto (próxima lección).

```bash
user1@ubuntu16-1:~$ timedatectl
      Local time: Sun 2020-02-16 03:57:22 PST
  Universal time: Sun 2020-02-16 11:57:22 UTC
        RTC time: Sun 2020-02-16 11:57:22
       Time zone: America/Los_Angeles (PST, -0800)
 Network time on: yes
NTP synchronized: yes
 RTC in local TZ: no
```

#### Configuración de la zona horaria del usuario

Podemos configurar nuestra zona horaria durante el proceso de instalación del SO, mediante la interfaz gráfica de usuario, o podemos utilizar la configuración de fecha y hora en el panel de la interfaz gráfica de usuario. Pero, como siempre, existen algunas herramientas de terminal que nos ayudan. En el pasado, se utilizaba el comando tzconfig, pero ya no se utiliza; en su lugar, utilice tzselect:

### tzselect

El programa **tzselect** solicita al usuario información sobre la ubicación actual y envía la descripción de la zona horaria resultante a la salida estándar. La salida es adecuada como valor para la variable de entorno **TZ**.

```bash
user1@ubuntu16-1:~$ tzselect 
Please identify a location so that time zone rules can be set correctly.
Please select a continent, ocean, "coord", or "TZ".
 1) Africa
 2) Americas
 3) Antarctica
 4) Asia
 5) Atlantic Ocean
 6) Australia
 7) Europe
 8) Indian Ocean
 9) Pacific Ocean
10) coord - I want to use geographical coordinates.
11) TZ - I want to specify the time zone using the Posix TZ format.
```

Al final el proceso nos sugiere configurar la variable de entorno TZ (Zona Horaria):

```
You can make this change permanent for yourself by appending the line
	TZ='Asia/Tehran'; export TZ
to the file '.profile' in your home directory; then log out and log in again.
```

Como se indica en el resultado, puede configurarlo y exportarlo en su archivo .profile si desea utilizar una zona horaria diferente a la zona horaria de su sistema.

#### configuración de la zona horaria del sistema

### /usr/share/zoneinfo

El directorio /usr/share/zoneinfo es el que guarda toda la información de la zona horaria.

```bash
root@ubuntu16-1:~# ls -l /usr/share/zoneinfo/ | head
total 328
drwxr-xr-x  2 root root  4096 Nov  4  2018 Africa
drwxr-xr-x  6 root root 20480 Nov  4  2018 America
drwxr-xr-x  2 root root  4096 Nov  4  2018 Antarctica
drwxr-xr-x  2 root root  4096 Nov  4  2018 Arctic
drwxr-xr-x  2 root root 12288 Nov  4  2018 Asia
drwxr-xr-x  2 root root  4096 Nov  4  2018 Atlantic
drwxr-xr-x  2 root root  4096 Nov  4  2018 Australia
drwxr-xr-x  2 root root  4096 Nov  4  2018 Brazil
drwxr-xr-x  2 root root  4096 Nov  4  2018 Canada
...
```

y contiene los archivos binarios de zona horaria necesarios:

```bash
root@ubuntu16-1:~# cat /usr/share/zoneinfo/Asia/Tehran 
TZif2e��l}������Ht-@�@0�:@Ug�EJ�7��-�( v�(۝�)˜
�*�"�+��H,�V8-��.���/o7H0a�81Pj�2B��32��4%u�5#H
6�86�V�7�ܸ8֊H9�8 . . .
```

### /etc/localtime

Linux consulta /etc/localtime para determinar la hora actual de su máquina. Puede ser un enlace simbólico a la zona horaria correcta o una copia directa del archivo de zona horaria.

```
root@ubuntu16-1:~# ls -l /etc/localtime 
lrwxrwxrwx 1 root root 39 Nov  4  2018 /etc/localtime -> /usr/share/zoneinfo/America/Los_Angeles
```

Podemos usar uno de los siguientes comandos para cambiar la zona horaria del sistema (Teherán):

* `ln -s /usr/share/zoneinfo/Asia/Tehran /etc/localtime`
* `cp /usr/share/zoneinfo/Asia/Tehran /etc/localtime`

> Si recibió un error al intentar crear un enlace simbólico, elimínelo primero: `sudo unlink /etc/localtime` o ` sudo rm -rf /etc/localtime`

### /etc/timezone

Este archivo contiene el nombre de la zona horaria en los sistemas basados ​​en Debian. `/etc/sysconfig/clock ` contiene el nombre de la zona horaria en los sistemas basados ​​en RHEL.

Hay otras formas de configurar la zona horaria en las distribuciones de Linux.

* uso de **timedatectl** en distribuciones con systemd:`timedatectl set-timezone Europe/Amsterdam`
* uso de dpkg-reconfigure en distribuciones (Debian/Ubuntu): `dpkg-reconfigure tzdata`
## Configuración de idiomas

Podemos configurar los idiomas del sistema desde las opciones (Regional\&Languages) pero siempre hay herramientas de terminal

### configuración regional

Una **configuración regional** es un conjunto de variables ambientales que definen la configuración de idioma, país y codificación de caracteres (o cualquier otra preferencia de variante especial) para sus aplicaciones y sesión de shell en un sistema Linux. Estas variables ambientales son utilizadas por las bibliotecas del sistema y las aplicaciones que reconocen la configuración regional en el sistema.

> Para ver información sobre la configuración regional instalada actualmente, utilice la opción **configuración regional**:

```bash
root@ubuntu16-1:~# locale
LANG=en_US.UTF-8
LANGUAGE=
LC_CTYPE="en_US.UTF-8"
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_PAPER="en_US.UTF-8"
LC_NAME="en_US.UTF-8"
LC_ADDRESS="en_US.UTF-8"
LC_TELEPHONE="en_US.UTF-8"
LC_MEASUREMENT="en_US.UTF-8"
LC_IDENTIFICATION="en_US.UTF-8"
LC_ALL=
```

> El formato de las variables es como: "Language_COUNTRY._ENCODING"_

### LANG

El valor de la variable de entorno **LANG** se establece en la instalación. (Esto proporciona el valor predeterminado para las variables LC_\* a menos que se configure dicha variable).

* LANGUAGE
* LC_CTYPE Cómo se clasifican los caracteres como letras, números, etc. Esto determina aspectos como la conversión de caracteres entre mayúsculas y minúsculas.
* LC_NUMERIC Cómo formatea los números. Por ejemplo, en muchos países se utiliza un punto (.) como separador decimal, mientras que en otros se utiliza una coma (,).
* LC_TIME Cómo se formatea la hora y la fecha. Utilice, por ejemplo, "en_DK.UTF-8" para obtener un reloj de 24 horas en algunos programas.
* LC_COLLATE Cómo se ordenan alfabéticamente las cadenas (nombres de archivos...). El uso de la configuración regional "C" o "POSIX" aquí da como resultado un orden de clasificación similar a strcmp(), que puede ser preferible a las configuraciones regionales específicas del idioma.
* LC_MONETARY Qué moneda utiliza, su nombre y su símbolo.
* LC_MESSAGES Qué idioma se debe utilizar para los mensajes del sistema.
* LC_PAPER Tamaños de papel: 11 x 17 pulgadas, A4, etc.
* LC_NAME Cómo se representan los nombres (apellido primero o último, etc.).
* LC_ADDRESS Cómo se formatean las direcciones (país primero o último, dónde va el código postal, etc.).
* LC_TELEPHONE Cómo se ven sus números de teléfono.
* LC_MEASUREMENT Qué unidades de medida se utilizan (pies, metros, libras, kilos, etc.).
* LC_IDENTIFICATION Metadatos sobre la información de la configuración regional.

como ejemplo, cambiemos LC_TIME por otra cosa:

```
root@ubuntu16-1:~# LC_TIME=en_GB.UTF-8 date
Sun 16 Feb 19:04:37 +0330 2020
root@ubuntu16-1:~# LC_TIME=en_US.UTF-8 date
Sun Feb 16 19:04:44 +0330 2020
```

### LC_ALL

Anula el valor de la variable de entorno **LANG** y los valores de cualquier otra variable de entorno **LC_\***.

```
root@ubuntu16-1:~# export LC_ALL="en_GB.UTF-8"
root@ubuntu16-1:~# locale
LANG=en_US.UTF-8
LANGUAGE=
LC_CTYPE="en_GB.UTF-8"
LC_NUMERIC="en_GB.UTF-8"
LC_TIME="en_GB.UTF-8"
LC_COLLATE="en_GB.UTF-8"
LC_MONETARY="en_GB.UTF-8"
LC_MESSAGES="en_GB.UTF-8"
LC_PAPER="en_GB.UTF-8"
LC_NAME="en_GB.UTF-8"
LC_ADDRESS="en_GB.UTF-8"
LC_TELEPHONE="en_GB.UTF-8"
LC_MEASUREMENT="en_GB.UTF-8"
LC_IDENTIFICATION="en_GB.UTF-8"
LC_ALL=en_GB.UTF-8
root@ubuntu16-1:~# unset LC_ALL
```

> También podemos usar el siguiente comando para trabajar con la configuración regional y configurar o instalar idiomas:
>
> * `dpkg-reconfigure locales` (Debian)
> * `system-config-language` (Redhat)
>
> También existe _localectl _en los sistemas systemd, que muestra y controla la configuración regional del sistema.

### LANG=C

LANG=C hace dos cosas:

* Obliga a las aplicaciones a usar el idioma predeterminado para la salida:

```
$ LC_ALL=es_ES man
¿Qué página de manual desea?

$ LC_ALL=C man
What manual page do you want?
```

* fuerza que la clasificación se realice byte a byte

```
$ LC_ALL=en_US sort <<< $'a\nb\nA\nB'
a
A
b
B

$ LC_ALL=C sort <<< $'a\nb\nA\nB'
A
B
a
b
```

También es posible hacer un LC_ALL=C.

### Codificación de caracteres

Una computadora representa información en números y, cuando es necesario comunicarlos a humanos (y viceversa), es necesario codificarlos.

* **ASCII** es una técnica de codificación de siete bits que asigna un número a cada uno de los 128 caracteres que se usan con más frecuencia en inglés americano. Esto permite que la mayoría de las computadoras registren y muestren texto básico. ASCII no incluye símbolos que se usan con frecuencia en otros países, como el símbolo de la libra esterlina o la diéresis alemana. ASCII es entendido por casi todo el software de correo electrónico y comunicaciones.
* **ISO 8859** es una extensión de ocho bits de ASCII desarrollada por ISO (la Organización Internacional de Normalización). ISO 8859 incluye los 128 caracteres ASCII junto con 128 caracteres adicionales, como el símbolo de la libra esterlina y el símbolo del centavo americano. Existen varias variaciones de la norma ISO 8859 para diferentes familias de idiomas
* **Unicode** es un intento de la ISO y el Consorcio Unicode de desarrollar un sistema de codificación para texto electrónico que incluya todos los alfabetos escritos existentes. Unicode utiliza caracteres de 8, 16 o 32 bits según la representación específica, por lo que los documentos Unicode suelen requerir hasta el doble de espacio en disco que ASCII

* ASCII: 7 bits. 128 code points.
* ISO-8859-1: 8 bits. 256 code points.
* UTF-8: 8-32 bits (1-4 bytes). 1,112,064 code points.

> Verifique la codificación disponible en su sistema con el comando `locale -m`.

> Use el comando `file` para ver la codificación de caracteres de un archivo.

### iconv

Podemos usar el programa `iconv` para convertir entre codificaciones de caracteres. Obviamente, si pasa de un conjunto de caracteres grande a uno más pequeño, la conversión no se realiza correctamente.

```bash
iconv [options] [-f from-encoding] [-t to-encoding] [inputfile]...
```

ejemplo:

```bash
iconv -f ASCII -t UTF-8 /path/to/MyOldFile > MyNewFile
```

Si no se proporciona ningún archivo de entrada, se lee desde la entrada estándar. De manera similar, si no se proporciona ningún archivo de salida, se escribe en la salida estándar. Ejemplo:

> `iconv -l` enumera todas las codificaciones de conjuntos de caracteres conocidos.

- [https://developer.ibm.com/technologies/linux/tutorials/l-lpic1-map/](https://developer.ibm.com/technologies/linux/tutorials/l-lpic1-map/)
- [https://www.tecmint.com/check-linux-timezone/](https://www.tecmint.com/check-linux-timezone/)
- [https://www.geeksforgeeks.org/date-command-linux-examples/](https://www.geeksforgeeks.org/date-command-linux-examples/)
- [https://www.computerhope.com/unix/udate.htm](https://www.computerhope.com/unix/udate.htm)
- [https://www.tecmint.com/set-time-timezone-and-synchronize-time-using-timedatectl-command/](https://www.tecmint.com/set-time-timezone-and-synchronize-time-using-timedatectl-command/)
- [https://www.timeanddate.com/time/time-zones.html](https://www.timeanddate.com/time/time-zones.html)
- [https://jadi.gitbooks.io/lpic1/content/1073\_localisation_and_internationalisation.html](https://jadi.gitbooks.io/lpic1/content/1073\_localisation_and_internationalisation.html)
- [https://linux.die.net/man/8/tzselect](https://linux.die.net/man/8/tzselect)
- [https://linuxacademy.com/blog/linux/changing-the-time-zone-in-linux-command-line/](https://linuxacademy.com/blog/linux/changing-the-time-zone-in-linux-command-line/)
- [https://www.2daygeek.com/linux-change-check-timezone/](https://www.2daygeek.com/linux-change-check-timezone/)
- [https://linux-audit.com/configure-the-time-zone-tz-on-linux-systems/](https://linux-audit.com/configure-the-time-zone-tz-on-linux-systems/)
- [https://help.ubuntu.com/community/Locale](https://help.ubuntu.com/community/Locale)
- [https://docs.oracle.com/cd/E23824\_01/html/E26033/glmbx.html](https://docs.oracle.com/cd/E23824\_01/html/E26033/glmbx.html)
- [https://unix.stackexchange.com/questions/87745/what-does-lc-all-c-do](https://unix.stackexchange.com/questions/87745/what-does-lc-all-c-do)
- [https://www.praim.com/en/news/character-encodings-in-linux-ascii-utf-8-and-iso-8859/](https://www.praim.com/en/news/character-encodings-in-linux-ascii-utf-8-and-iso-8859/)
- [https://kb.iu.edu/d/ahfr](https://kb.iu.edu/d/ahfr)
- [https://www.geeksforgeeks.org/iconv-command-in-linux-with-examples/](https://www.geeksforgeeks.org/iconv-command-in-linux-with-examples/)
