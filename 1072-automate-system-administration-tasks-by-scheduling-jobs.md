@**Peso:** 4

Descripción: Los candidatos deben poder usar cron o anacron para ejecutar trabajos a intervalos regulares y usar at para ejecutar trabajos en un momento específico.

**Áreas de conocimiento clave:**

* Administrar trabajos cron y at
* Configurar el acceso de los usuarios a cron y servicios at
* Configurar anacron

**Términos y utilidades:**

* /etc/cron.{d,daily,hourly,monthly,weekly}/
* /etc/at.deny
* /etc/at.allow
* /etc/crontab
* /etc/cron.allow
* /etc/cron.deny
* /var/spool/cron/
* crontab
* at
* atq
* atrm
* anacron
* /etc/anacrontab

Muchas tareas de administración del sistema deben realizarse con regularidad, como rotar archivos de registro, realizar copias de seguridad de archivos o bases de datos, preparar informes o instalar actualizaciones del sistema. En esta lección, aprenderemos a automatizar este tipo de trabajos configurando una programación para ellos.

Al programar:

* Podemos iniciar un trabajo en un momento en el que el uso del sistema sea bajo.
* Se reducirán los errores debido a que se necesita menos interacción manual.
* Estamos seguros de que los trabajos siempre se ejecutan de la misma manera.
* ¡Los administradores de sistemas pueden dormir más!

### Ejecutar trabajos a intervalos regulares <a href="run-jobs-at-regular-intervals" id="run-jobs-at-regular-intervals"></a>

Los sistemas Linux tienen dos funciones para programar trabajos para que se ejecuten a intervalos regulares:

* La función `cron` original es la más adecuada para servidores y sistemas que están encendidos continuamente.
* La función `anacron` (o el anacrónico `cron`) es adecuada para sistemas como computadoras de escritorio o portátiles que pueden estar en modo de suspensión o funcionando con energía de la batería.

| cron                                                 | anacron                                |
| ---------------------------------------------------- | -------------------------------------- |
| Good for servers                                     | Good for laptops and desktops          |
| Granularity from one minute to one year              | Daily, weekly, and monthly granularity |
| Job runs only if system is running at scheduled time | Job runs when system is next available |
| Can be used by normal users                          | Needs root authority                   |
### Programar trabajos periódicos con cron <a href="schedule-periodic-jobs-with-cron" id="schedule-periodic-jobs-with-cron"></a>

La función `cron` consta del demonio `cron` y un conjunto de tablas que describen qué trabajo se debe realizar y con qué frecuencia. El demonio `cron` generalmente lo inicia el proceso `init`, `upstart` o `systemd` al iniciar el sistema.

### Sintaxis y operadores del archivo crontab

Crontab (tabla cron) es un archivo de texto que especifica la programación de los trabajos cron. El demonio `cron` se activa cada minuto y verifica cada crontab en busca de trabajos que se deben ejecutar.

Hay dos tipos de archivos crontab. Los archivos crontab de usuario individual y los archivos crontab de todo el sistema.

#### Sintaxis y operadores de crontab

Cada línea del archivo crontab del usuario contiene seis campos separados por un espacio seguido del comando que se ejecutará.

```bash
* * * * * command(s)
- - - - -
| | | | |
| | | | ----- Day of week (0 - 7) (Sunday=0 or 7)
| | | ------- Month (1 - 12)
| | --------- Day of month (1 - 31)
| ----------- Hour (0 - 23)
------------- Minute (0 - 59)
```

* `*` - El operador asterisco significa cualquier valor o siempre.
* `,` - El operador coma le permite especificar una lista de valores para la repetición.
* `-` - El operador guion le permite especificar un rango de valores.
* `/` - El operador barra le permite especificar valores que se repetirán en un cierto intervalo entre ellos.

> Además, si tiene `@reboot` o `@daily` en lugar de campos de tiempo, el comando se ejecutará una vez después del reinicio o diariamente.

Veamos algunos ejemplos:

```
@reboot      ### on reboot

50 13 * * *  ### 1:50 PM daily
40 6 4 * *   ### 4th of every month at 6:40AM
05 * * 1 0   ### 5 mintues past everyhour,each Sunday in January
0 15 29 11 5 ### 3:00PM Every November 29th  that lands on a Friday
1,5,10 * * * * # 1,5,10 mintues past every hour
30 20 * * 1-5 ## weekdays at 8:20 PM
*/5 * * * *  ### ever 5 mintues
```

## Crons específicos del usuario

### /var/spool/cron/

Los archivos crontab de los usuarios se almacenan por nombre de usuario y su ubicación varía según el sistema operativo. En sistemas basados ​​en Red Hat como CentOS, los archivos crontab se almacenan en el directorio `/var/spool/cron` mientras que en Debian y Ubuntu los archivos se almacenan en el directorio `/var/spool/cron/crontabs`.

Aunque puede editar los archivos crontab del usuario manualmente, se recomienda utilizar el comando `crontab`.

### Comando crontab

El comando crontab le permite instalar o abrir un archivo crontab para editarlo.

Puede utilizar el comando crontab para ver, agregar, eliminar o modificar trabajos cron utilizando las siguientes opciones:

* `crontab -e` : Editar archivo crontab o crear uno si aún no existe.
* `crontab -l` : Mostrar el contenido del archivo crontab.
* `crontab -r` : elimina el archivo crontab actual.
* `crontab -i` : elimina el archivo crontab actual y muestra un mensaje antes de eliminarlo.
* `crontab -u <nombre de usuario>` : edita el archivo crontab de otro usuario. Requiere privilegios de administrador del sistema.

```bash
user1@ubuntu16-1:~$ crontab -e
no crontab for user1 - using an empty one

Select an editor.  To change later, run 'select-editor'.
  1. /bin/ed
  2. /bin/nano        <---- easiest
  3. /usr/bin/vim.basic
  4. /usr/bin/vim.tiny

Choose 1-4 [2]: 3
```

El comando crontab -e abre el archivo crontab usando el editor especificado por las variables de entorno `VISUAL` o `EDITOR`.

```
# Edit this file to introduce tasks to be run by cron.
# 
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
# 
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').# 
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
# 
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
# 
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
# 
# For more information see the manual pages of crontab(5) and cron(8)
# 
# m h  dom mon dow   command
~                                                                               
"/tmp/crontab.f5VKYq/crontab" 22L, 888C                       1,1           All
```

Como ejemplo, agregue la siguiente línea a la anterior y se enviará un correo electrónico cada 5 minutos:

```
5 * * * * echo "Hello" | mail -s "Cron Test" user1@localhost.com
```

crontab -e también verifica la sintaxis antes de salir del archivo, lo cual es realmente útil.

crontab -l mostraría el contenido anterior. Verifiquemos si se creó el archivo crontab del usuario:

```bash
root@ubuntu16-1:/var/spool/cron/crontabs# ls -l
total 4
-rw------- 1 user1 crontab 1154 Feb 15 05:07 user1
```

## cron de todo el sistema

Además de los archivos crontab del usuario en /var/spool/cron, el demonio `cron` también revisa /etc/crontab y cualquier crontab en el directorio /etc/cron.d.

### /etc/crontab , /etc/cron.d

`/etc/crontab` y los archivos dentro del directorio `/etc/cron.d` son archivos **crontab** de todo el sistema que solo pueden ser editados por los administradores del sistema.

> /etc/crontab se actualiza mediante edición directa. No puede usar el comando `crontab` para actualizar archivos o archivos en el directorio /etc/cron.d.

#### Archivos crontab de todo el sistema <a href="system-wide-crontab-files" id="system-wide-crontab-files"></a>

La sintaxis de los archivos crontab de todo el sistema es ligeramente diferente a la de los crontabs de usuario. Contiene un campo de usuario obligatorio adicional que especifica qué usuario ejecutará el trabajo cron.

```
* * * * * <username> command(s)
```

Este archivo debe editarse directamente con un editor y podemos mencionar qué usuario ejecuta el/los comando(s).

```bash
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
~                                                                               
~                                                                               
~                                                                                                                                                                                                                                         
"/etc/crontab" 15L, 722C                                      14,27-35      All
```

### /etc/cron.{daily,hourly,monthly,weekly}/

En la mayoría de las distribuciones de Linux también puedes poner **scripts** dentro de los directorios `/etc/cron.{hourly,daily,weekly,monthly}` y los scripts se ejecutarán cada `hora/día/semana/mes`.

```bash
root@ubuntu16-1:~# tree /etc/cron*
/etc/cron.d
├── anacron
├── php
└── popularity-contest
/etc/cron.daily
├── 0anacron
├── apache2
├── apport
├── apt-compat
├── aptitude
├── bsdmainutils
├── cracklib-runtime
├── dpkg
├── logrotate
├── man-db
├── mlocate
├── passwd
├── popularity-contest
├── quota
├── samba
├── update-notifier-common
└── upstart
/etc/cron.hourly
/etc/cron.monthly
└── 0anacron
/etc/crontab [error opening dir]
/etc/cron.weekly
├── 0anacron
├── fstrim
├── man-db
└── update-notifier-common

0 directories, 25 files
```

A modo de ejemplo, veamos uno de ellos:

```bash
root@ubuntu16-1:~# cat /etc/cron.daily/passwd 
#!/bin/sh

cd /var/backups || exit 0

for FILE in passwd group shadow gshadow; do
        test -f /etc/$FILE              || continue
        cmp -s $FILE.bak /etc/$FILE     && continue
        cp -p /etc/$FILE $FILE.bak && chmod 600 $FILE.bak
done
```

### anacron

La función cron funciona bien para sistemas que se ejecutan continuamente. Si el sistema está inactivo cuando el cron debería ejecutar una tarea, ese trabajo cron no se ejecutará hasta la próxima ocurrencia. Pero anacron crea la marca de tiempo cada vez que se ejecuta un trabajo diario, semanal o mensual.

> Nota: anacron verifica las marcas de tiempo en el MOMENTO DEL ARRANQUE y no maneja trabajos que deben ejecutarse cada hora o cada minuto.

> ### /etc/anacron

La tabla de trabajos para `anacron` se almacena en /etc/anacrontab, que tiene un formato ligeramente diferente de /etc/crontab.

```bash
root@ubuntu16-1:/# cat /etc/anacrontab 
# /etc/anacrontab: configuration file for anacron

# See anacron(8) and anacrontab(5) for details.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
HOME=/root
LOGNAME=root

# These replace cron's entries
1	5	cron.daily	run-parts --report /etc/cron.daily
7	10	cron.weekly	run-parts --report /etc/cron.weekly
@monthly	15	cron.monthly	run-parts --report /etc/cron.monthly
```

> al igual que /etc/crontab, /etc/anacrontab se actualiza mediante edición directa.
#### Formato de Anacrontab

```bash
period   delay   job-identifier   command
```

* ** período en días**: especifica la frecuencia de ejecución de un trabajo en _N_ días.
* ** retraso en minutos**: cantidad de minutos que anacron debe esperar antes de ejecutar el trabajo después del reinicio.
* ** identificador de trabajo: ** es el nombre del archivo de marca de tiempo del trabajo. Debe ser único para cada trabajo. Este estará disponible como un archivo en el directorio /var/spool/anacron.
* ** comando**: especifica el comando a ejecutar.
### /var/spool/anacron

`anacron` mantiene un archivo de marca de tiempo en /var/spool/anacron para cada trabajo para registrar cuándo se ejecuta el trabajo. Cuando `anacron` se ejecuta, verifica si ha transcurrido la cantidad de días requerida desde la última ejecución de un trabajo y ejecuta el trabajo si es necesario.

```bash
root@ubuntu16-1:/# ls -l /var/spool/anacron/
total 12
-rw------- 1 root root 9 Feb 15 07:35 cron.daily
-rw------- 1 root root 9 Feb  1 07:45 cron.monthly
-rw------- 1 root root 9 Feb 10 00:46 cron.weekly
```

Este archivo contendrá una sola línea que indica la última vez que se ejecutó este trabajo.

```bash
root@ubuntu16-1:/# cat /var/spool/anacron/cron.daily 
20200215
```

### at

A veces, es necesario ejecutar un trabajo en un momento futuro solo una vez, en lugar de hacerlo con regularidad. Para ello, se utiliza el comando `at`. (ubuntu: `apt install at`)

Una secuencia de comandos **at** típica se ve así

```bash
at 5:45
```

Al ejecutar el comando at, se lo coloca en un mensaje especial, donde puede escribir el comando (o la serie de comandos) que se ejecutará en el momento programado. Cuando haya terminado, presione **Control-D** en una nueva línea y su comando se colocará en la cola.

```bash
user1@ubuntu16-1:~$ at 5:45
warning: commands will be executed using /bin/sh
at> touch BlaHBlaH 
at> <EOT>
job 3 at Sat Feb 15 05:45:00 2020

user1@ubuntu16-1:~$ ls -ltrh | grep -i blah
-rw-rw-r-- 1 user1 user1    0 Feb 15 05:45 BlaHBlaH
```

> Advertencia: los comandos se ejecutarán mediante /bin/sh

Otros ejemplos del comando at:

```bash
  Example                  Schedule Task at
-----------                -------------------
at 10:00 AM                at coming 10:00 AM
at 10:00 AM Sun            at 10:00 AM on coming Sunday
at 10:00 AM July 25        at 10:00 AM on coming 25’th July
at 10:00 AM 6/22/2019      at 10:00 AM on coming 22’nd June 2019
at 10:00 AM next month     at 10:00 AM on the same date at next month
at 10:00 AM tomorrow       at 10:00 AM tomorrow
at teatime                 to execute on next 4:00 pM
at midnight                to execute on next 12:00 AM
at now + 1 hour            to execute just after 1 hour
at now + 1 week            to execute just after 1 week
at now + 1 month           to execute just after 1 month
at now + 1 year            to execute just after 1 year
```

El comando at tiene otros miembros en su familia:

### atq

enumera los trabajos pendientes de los usuarios

```
user1@ubuntu16-1:~$ at 06:20
warning: commands will be executed using /bin/sh
at> touch file1
at> <EOT>
job 4 at Sat Feb 15 06:20:00 2020
user1@ubuntu16-1:~$ at 06:30
warning: commands will be executed using /bin/sh
at> touch file2
at> <EOT>
job 5 at Sat Feb 15 06:30:00 2020

user1@ubuntu16-1:~$ atq
4	Sat Feb 15 06:20:00 2020 a user1
5	Sat Feb 15 06:30:00 2020 a user1
```

### atrm 

Eliminar trabajos por su número de trabajo

```
user1@ubuntu16-1:~$ atrm 5
user1@ubuntu16-1:~$ atq
4	Sat Feb 15 06:20:00 2020 a user1
```

El comando atq solo muestra la lista de trabajos, pero si desea verificar qué script/comandos están programados con esa tarea, use el comando `at -c JobNum` y vea la última línea.

Tanto cron como son servicios del sistema.
## Configurar el acceso de los usuarios a la programación de trabajos

Podemos controlar el acceso al comando crontab usando dos archivos en el directorio /etc/cron.d: cron.deny y cron.allow. Estos archivos permiten que solo usuarios específicos realicen tareas del comando crontab, como crear, editar, mostrar o eliminar sus propios archivos crontab.

> Los archivos cron.deny y cron.allow consisten en una lista de nombres de usuario, un nombre de usuario por línea.

### /etc/cron.allow , /etc/cron.deny

Estos archivos de control de acceso funcionan juntos de la siguiente manera:

* Si existe cron.allow, solo los usuarios que se enumeran en este archivo pueden crear, editar, mostrar o eliminar archivos crontab.
* Si no existe cron.allow, todos los usuarios pueden enviar archivos crontab, excepto los usuarios que se enumeran en cron.deny.
* Si no existe cron.allow ni cron.deny, se requieren privilegios de superusuario para ejecutar el comando crontab.

Se requieren privilegios de superusuario para editar o crear los archivos cron.deny y cron.allow.

```
root@ubuntu16-1:~# cat /etc/cron.deny
user2
```

###  /etc/at.allow , /etc/at.deny

Los archivos /etc/at.allow y /etc/at.deny correspondientes tienen efectos similares para la función `at`
### Variables de crontab <a href="crontab-variables" id="crontab-variables"></a>

El demonio cron establece automáticamente varias variables de entorno.

* La ruta predeterminada está establecida en `PATH=/usr/bin:/bin`. Si el comando que está llamando está presente en la ruta especificada de cron, puede utilizar la ruta absoluta al comando o cambiar la variable `$PATH` de cron. No puede agregar implícitamente `:$PATH` como lo haría con un script normal.
* El shell predeterminado está establecido en `/bin/sh`. Puede establecer un shell diferente cambiando la variable `SHELL`.
* Cron invoca el comando desde el directorio de inicio del usuario. La variable `HOME` puede ser anulada por las configuraciones en el crontab.
* La notificación por correo electrónico se envía al propietario del crontab. Para sobrescribir el comportamiento predeterminado, puede utilizar la variable de entorno `MAILTO` con una lista (separada por comas) de todas las direcciones de correo electrónico que desea que reciban las notificaciones por correo electrónico. Si `MAILTO` está definido pero está vacío (`MAILTO=""`), no se envía ningún correo.

- [https://developer.ibm.com/tutorials/l-lpic1-107-2/](https://developer.ibm.com/tutorials/l-lpic1-107-2/)
- [https://linuxize.com/post/scheduling-cron-jobs-with-crontab/](https://linuxize.com/post/scheduling-cron-jobs-with-crontab/)
- [https://jadi.gitbooks.io/lpic1/content/1072\_automate_system_administration_tasks_by_scheduling_jobs.html](https://jadi.gitbooks.io/lpic1/content/1072\_automate_system_administration_tasks_by_scheduling_jobs.html)
- [https://www.thegeekdiary.com/centos-rhel-anacron-basics-what-is-anacron-and-how-to-configure-it/](https://www.thegeekdiary.com/centos-rhel-anacron-basics-what-is-anacron-and-how-to-configure-it/)
- [https://www.thegeekstuff.com/2011/05/anacron-examples/](https://www.thegeekstuff.com/2011/05/anacron-examples/)
- [https://www.computerhope.com/unix/uat.htm](https://www.computerhope.com/unix/uat.htm)
- [https://tecadmin.net/one-time-task-scheduling-using-at-commad-in-linux/](https://tecadmin.net/one-time-task-scheduling-using-at-commad-in-linux/)
- [https://docs.oracle.com/cd/E19253-01/817-0403/sysrescron-23/index.html](https://docs.oracle.com/cd/E19253-01/817-0403/sysrescron-23/index.html)
