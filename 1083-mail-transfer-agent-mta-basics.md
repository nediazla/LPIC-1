# 108.3. Fundamentos del Agente de Transferencia de Correo (MTA)

**Ponderación:** 3

**Descripción: **Los candidatos deben conocer los programas MTA más comunes y ser capaces de realizar la configuración básica de reenvío y alias en un host cliente. No se incluyen otros archivos de configuración.

**Áreas de conocimiento clave:**

* Crear alias de correo electrónico
* Configurar el reenvío de correo electrónico
* Conocimiento de los programas MTA más comunes (postfix, sendmail, qmail, exim) (sin configuración)

**Términos y utilidades:**

* \~/.forward
* sendmail emulation layer commands
* newaliases
* mail
* mailq
* postfix
* sendmail
* exim
* qmail

### MTAs

Los agentes de transferencia de correo entregan correo entre usuarios y sistemas. La mayoría del correo de Internet utiliza el Protocolo Simple de Transferencia de Correo (SMTP), pero el correo local puede transferirse mediante otras posibilidades. Existen diferentes tipos de MTAs disponibles y cada distribución puede tener uno predeterminado. Como administrador, podemos elegir el MTA adecuado según nuestras necesidades.

### sendmail

Sendmail es el MTA más antiguo de Linux. Originalmente se derivó del programa delivermail utilizado en ARPANET en 1979. Sendmail es grande y no es fácil de configurar. A lo largo de los años se han encontrado muchas vulnerabilidades en sendmail, aunque ahora está ligeramente mejorado.

#### Otros agentes de transferencia de correo

En respuesta a los problemas de seguridad de sendmail, se desarrollaron otros agentes de transferencia de correo durante la década de 1990. Postfix es quizás el más popular, pero qmail y exim también se utilizan ampliamente.

### qmail

Qmail es una de las soluciones de software de servidor de correo Linux más seguras del mercado actual. Aunque no cuenta con soporte técnico ni se encuentra actualmente en desarrollo (Qmail no se ha actualizado desde 1997), cuenta con una amplia base de seguidores.

Qmail también es más rápido y escala mejor con cargas de correo mayores que Sendmail. Sin embargo, Qmail no es fácil de configurar ni de ampliar.

> qmail no es un software GPL; es de dominio público.

### exim

Exim existe desde 1995 y su popularidad ha ido en aumento desde entonces. Su mayor ventaja es su nivel de personalización casi infinito. Exim permite que el administrador del servidor cree un conjunto de reglas personalizado que gestione los correos electrónicos entrantes y salientes de una forma específica (autenticación, lista de control de acceso).

Aunque Exim no fue diseñado para el alto rendimiento, puede configurarse para funcionar como un servidor de correo de alto rendimiento. Exim es un excelente MTA si necesita crear una configuración de correo compleja o personalizada.

### postfix

Postfix es posiblemente el MTA de más rápido crecimiento en el mercado actual. Postfix es extremadamente popular gracias a su rendimiento y a su historial de seguridad. Es mucho más difícil (o casi imposible) comprometer al usuario root en un servidor que ejecuta Postfix que, por ejemplo, Sendmail o Exim.

Postfix también funciona más rápido con menos recursos del sistema que la mayoría de los demás MTA (o al menos, con configuraciones estándar). Las configuraciones estándar son fáciles de crear, pero si necesitas una configuración única, puede ser un problema con Postfix.

### Capa de emulación de Sendmail

Dado que Sendmail lleva tanto tiempo disponible, independientemente del servidor de correo electrónico que usemos, incluye una capa de emulación de Sendmail. En otras palabras, todos los demás MTA incluyen herramientas compatibles con versiones anteriores. Por lo tanto, si escribimos los comandos `sendmail` o `mailq`, responden y actúan como si Sendmail estuviera instalado.

### Alias ​​de correo <a href="mail-aliases" id="mail-aliases"></a>

A veces, es posible que desee que todo el correo de un usuario se envíe a otra ubicación. Por ejemplo, puede tener una granja de servidores y desear que todo el correo raíz se envíe a un administrador central del sistema. O puede crear una lista de correo donde el correo se envía a varias personas. Para ello, utilizamos alias que nos permiten definir uno o más destinos para un nombre de usuario determinado.

Los alias de correo se encuentran en `/etc/aliases`. `Primero debe convertirse en root antes de modificar este archivo: (CentOS7, la salida ha sido truncada)

```
[root@centos7-1 ~]# cat /etc/aliases
#
#  Aliases in this file will NOT be expanded in the header from
#  Mail, but WILL be visible over networks or from /bin/mail.
#
#	>>>>>>>>>>	The program "newaliases" must be run after
#	>> NOTE >>	this file is updated for any changes to
#	>>>>>>>>>>	show through to sendmail.
#

# Basic system aliases -- these MUST be present.
mailer-daemon:	postmaster
postmaster:	root

# General redirections for pseudo accounts.
bin:		root
daemon:		root
adm:		root
lp:		root
sync:		root
shutdown:	root
halt:		root
mail:		root
ftp:		root
nobody:		root
amandabackup:		root
xfs:		root
gdm:		root


lpic1:  root     ###<---------


# trap decode to catch security attacks
decode:		root

# Person who should get root's mail
#root:		marc
root:    payam   ###<-----------
```

Por lo tanto, si hay un mensaje para `"lpic1"`, se enviará al usuario `root`.

La última línea indica que `payam` está leyendo los correos `root`. Esto permite que `payam` lea los correos `root` sin iniciar sesión como root.

### newaliases

Las modificaciones al archivo `/etc/aliases` no se completan hasta que se ejecuta el comando `newaliases` para compilar `/etc/aliases.db`.

```
[root@centos7-1 ~]# newaliases
[root@centos7-1 ~]# 
```

### Comando mail

Existen muchas maneras de enviar correos electrónicos usando la interfaz gráfica de usuario, el navegador o un cliente de correo electrónico. Sin embargo, las opciones son limitadas en lo que respecta a la interfaz de línea de comandos.

El comando `mail` es un recurso clásico que permite programar el envío de correo, así como recibir y administrar el correo entrante. (CentOS: instalar mailx)

| mail command example                                | usage                     |
| --------------------------------------------------- | ------------------------- |
| mail -s “subject” user1@domain.com                  | sending an email          |
| mail -s “subject” user1@domain.com < /root/test.txt | send an email from a file |
| mail -s “subject” user1@domain.com -a /path/to/file | add an attachment         |

También se puede enviar correos electrónicos desde scripts como 'echo -e "contenido del correo" | mail -s "asunto del correo" "ejemplo@ejemplo.com"'.

Podemos usar `mail` de forma interactiva para enviar mensajes pasando una lista de destinatarios, o bien, sin argumentos, para revisar el correo entrante.

```
[user1@centos7-1 ~]$ mail lpic1@localhost
Subject: test lpic1
Hi this is my first email.
I am sending it to lpic1 email address.
EOT
```

Debido a los alias que hemos modificado, cualquier correo electrónico que se envíe a la dirección de correo electrónico lpic1 se r&eenviará a root, ya que hemos definido a payam como una persona que debe recibir los correos electrónicos de root:

```
[payam@centos7-1 ~]$ mail
Heirloom Mail version 12.5 7/5/10.  Type ? for help.
"/var/spool/mail/payam": 1 message 1 new
>N  1 user1@centos7-1.loca  Wed Feb 19 05:10  19/683   "test lpic1"
& 1
Message  1:
From user1@centos7-1.localdomain  Wed Feb 19 05:10:35 2020
Return-Path: <user1@centos7-1.localdomain>
X-Original-To: lpic1@localhost
Delivered-To: lpic1@localhost.localdomain
Date: Wed, 19 Feb 2020 05:10:35 -0500
To: lpic1@localhost.localdomain
Subject: test lpic1
User-Agent: Heirloom mailx 12.5 7/5/10
Content-Type: text/plain; charset=us-ascii
From: user1@centos7-1.localdomain
Status: R

Hi this is my first email.
I am sending it to lpic1 email address.

& d
& q
```

Al enviar un correo, el comando "mail" llama inicialmente al binario de correo, que a su vez se conecta al MTA local para enviar el correo a su destino. El MTA local es un servidor SMTP que se ejecuta localmente y acepta correos en el puerto 25.

### \~/.forward

El archivo de alias debe ser administrado por un administrador del sistema. Los usuarios pueden habilitar el reenvío de su propio correo mediante un archivo .forward en su directorio personal. Puede incluir cualquier elemento permitido en su archivo .forward, a la derecha del archivo de alias. El archivo contiene texto sin formato y no necesita compilarse. Cuando el correo se dirige a usted, sendmail busca un archivo .forward en su directorio personal y procesa las entradas de la misma manera que procesa los alias.

```
[user1@centos7-1 ~]$ pwd
/home/user1
[user1@centos7-1 ~]$ ls -la | grep -i forward
[user1@centos7-1 ~]$ vi .forward
[user1@centos7-1 ~]$ cat .forward 
user2
```

Ahora, enviemos un correo electrónico al usuario1 y verifiquemos su casilla de correo:

```
[payam@centos7-1 ~]$ mail user1@localhost
Subject: mail to user1
Hello!!!
EOT
```

```
[user1@centos7-1 ~]$ mail
No mail for user1
```

```
[user2@centos7-1 ~]$ mail
Heirloom Mail version 12.5 7/5/10.  Type ? for help.
"/var/spool/mail/user2": 1 message 1 new
>N  1 payam@centos7-1.loca  Wed Feb 19 06:33  21/771   "mail to user1"
& 1
Message  1:
From payam@centos7-1.localdomain  Wed Feb 19 06:33:28 2020
Return-Path: <payam@centos7-1.localdomain>
X-Original-To: user1@localhost
Delivered-To: user2@centos7-1.localdomain
Delivered-To: user1@localhost.localdomain
Date: Wed, 19 Feb 2020 06:33:27 -0500
To: user1@localhost.localdomain
Subject: mail to user1
User-Agent: Heirloom mailx 12.5 7/5/10
Content-Type: text/plain; charset=us-ascii
From: payam@centos7-1.localdomain
Status: R

Hello!!!

& d
& q
[user2@centos7-1 ~]$ 
```

### mailq

La gestión de correo en Linux utiliza un modelo de almacenamiento y reenvío. Ya has visto que el correo entrante se almacena en un archivo en /var/mail hasta que lo lees. El correo saliente también se almacena hasta que se establece una conexión con el servidor receptor. Usa el comando `mailq` para ver qué correo está en cola.

```
[root@centos7-1 ~]# mailq
Mail queue is empty
[root@centos7-1 ~]# mailq
-Queue ID- --Size-- ----Arrival Time---- -Sender/Recipient-------
E085461E470C      473 Wed Feb 19 06:39:07  payam@centos7-1.localdomain
(Host or domain name not found. Name service error for name=gmail.com type=MX: Host not found, try again)
                                         test@gmail.com

-- 0 Kbytes in 1 Request.
```

Obviamente en un sistema de salud mailq debe estar siempre vacío.


[https://en.wikipedia.org/wiki/Message_transfer_agent](https://en.wikipedia.org/wiki/Message_transfer_agent)

[https://developer.ibm.com/tutorials/l-lpic1-108-3/](https://developer.ibm.com/tutorials/l-lpic1-108-3/)

[https://jadi.gitbooks.io/lpic1/content/1083\_mail_transfer_agent_mta_basics.html](https://jadi.gitbooks.io/lpic1/content/1083\_mail_transfer_agent_mta_basics.html)

[http://linuxconsultant.info/tutorials/linux-mail-server-software.html](http://linuxconsultant.info/tutorials/linux-mail-server-software.html)

[https://hoststud.com/resources/comparison-between-mtas-postfix-vs-exim-vs-sendmail.158/](https://hoststud.com/resources/comparison-between-mtas-postfix-vs-exim-vs-sendmail.158/)

with the special thanks of shawn powers.
