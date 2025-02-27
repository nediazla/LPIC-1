# 105.2. Personalizar o escribir scripts sencillos

**Peso: **4

**Descripción: **Los candidatos deben poder personalizar scripts existentes o escribir nuevos scripts Bash simples.

**Áreas de conocimiento clave:**

* Utilizar la sintaxis sh estándar (bucles, pruebas)
* Utilizar la sustitución de comandos
* Probar los valores de retorno para comprobar si se han realizado correctamente o no, u otra información proporcionada por un comando
* Realizar envíos condicionales al superusuario
* Seleccionar correctamente el intérprete de scripts mediante la línea shebang (#!)
* Gestionar la ubicación, la propiedad, la ejecución y los derechos suid de los scripts

**Términos y utilidades:**

* for
* while
* test
* if
* read
* seq
* exec

Esta lección trata sobre scripts de shell en Linux y comenzaremos a escribir algunos scripts simples. Podemos poner casi todos los comandos que usamos dentro de un script de shell. Pero antes de comenzar, hay algunas notas que debemos tener en cuenta.

### shebang (#!)

En Linux y en el mundo del código abierto, los archivos de script son muy importantes. Existen diferentes tipos de lenguajes de script que se utilizan para escribir archivos de script. Como la extensión de archivo es solo una etiqueta para un script, **shebang** (file script file interpreter line) se utiliza para especificar el lenguaje de script.

```
#!/bin/bash

echo "Hello World!
```

Además de eso, hay diferentes tipos de shell y no puedes garantizar qué shell preferirán tus potenciales usuarios. shebang hace que los scripts sean portables a todos los entornos de shell al indicarle al shell que ejecute tu script en un shell en particular.

> `#` y `!` hacen que esta línea sea especial porque `#` se usa como línea de comentario en bash.

La línea shebang debe ser la primera línea de tu script y el resto de la línea contiene la ruta al shell en el que debe ejecutarse tu programa.

### variables

En secciones anteriores de esta serie, aprendimos cómo definir una variable `VARIABLE="value" `. Podemos hacer lo mismo en un script, como ejemplo:

```
#!/bin/bash

MYNAME="payam"
echo "Hello $MYNAME!"
```

Hemos asignado valores de cadena a las variables. Bash admite la aritmética de shell utilizando números enteros.

**declare**

**declare** es un comando incorporado del shell **bash**. Se utiliza para declarar variables y funciones del shell, establecer sus atributos y mostrar sus valores.` -i `haz que una VARIABLE tenga el atributo 'entero'.

```
user1@ubuntu16-1:~/sandbox$ declare VAR1="hello"
user1@ubuntu16-1:~/sandbox$ declare -i VAR2="42"
```

**`-p` **muestra las opciones y atributos de cada nombre de variable si se utiliza con argumentos de nombre:

```
user1@ubuntu16-1:~/sandbox$ declare -p VAR1 VAR2
declare -- VAR1="hello"
declare -i VAR2="42"
```

### sustitución de comandos

A veces, es posible que necesitemos utilizar el resultado de un comando para otro comando o una variable.

Si rodeamos un comando con `$(` y `)`, o con un par de comillas invertidas, podemos sustituir la salida del comando como entrada de otro comando. Esta técnica se denomina _sustitución de comandos_.

```
$(command)
### OR
`command`
```

example:

```
user1@ubuntu16-1:~/sandbox$ MYLIST=$(ls -l)
user1@ubuntu16-1:~/sandbox$ echo $MYLIST
total 4 -rw-rw-r-- 1 user1 user1 58 فوریه 4 12:24 script1

user1@ubuntu16-1:~/sandbox$ MYSPACE=`du -sh`
user1@ubuntu16-1:~/sandbox$ echo $MYSPACE
8.0K .
```

### Ejecución de scripts
Para ejecutar un script de shell, el script debe ser ejecutable.

```
user1@ubuntu16-1:~/sandbox$ ls -l
total 4
-rw-rw-r-- 1 user1 user1 58 فوریه  4 12:24 script1
user1@ubuntu16-1:~/sandbox$ chmod +x script1 
user1@ubuntu16-1:~/sandbox$ ls -l
total 4
-rwxrwxr-x 1 user1 user1 32 فوریه  4 14:14 script1
```

> Otra forma de ejecutar un script es dar nuestro nombre de script como parámetro a los comandos `sh` o `bash`. `sh ./script1` o `bash ./script1`

### localización de scripts

Si nuestro script es ejecutable podemos localizarlo de 3 formas:

* Podemos usar `./scriptname` si estamos en el mismo directorio
* Podemos usar la ruta absoluta `/home/user1/sandbox/scriptname`
* poniendo nuestro script en uno de los directorios `$PATH` o editando la variable `PATH`.

```
user1@ubuntu16-1:~/sandbox$ ./script1 
Hello World

user1@ubuntu16-1:~/sandbox$ /home/user1/sandbox/script1
Hello World

user1@ubuntu16-1:~$ echo $PATH
/home/user1/bin:/home/user1/.local/bin:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games:/snap/bin

user1@ubuntu16-1:~$ mkdir bin
user1@ubuntu16-1:~$ cp sandbox/script1 bin/
user1@ubuntu16-1:~$ scr
screendump    script        script1       scriptreplay  
user1@ubuntu16-1:~$ script1
Hello World
```

### condicionamiento

Para escribir programas útiles, casi siempre necesitamos la capacidad de verificar condiciones y cambiar el comportamiento del programa en consecuencia. Las declaraciones condicionales nos brindan esta capacidad.

La forma más simple es la declaración **if**, que tiene la forma general:

### if

```
if [expression]
then
   command1 
   command2
else
   command3
   command4
fi
```

> la parte else es opcional y puede omitirse.

```
#!/bin/bash

MYOS="linux"
echo "Lets see What is your favirite OS"

if [ $MYOS = "linux" ] 
then
 echo "Haha happy to hear that"
else
 echo "I wish you liked linux"
fi
```

> La comprobación real de la condición se realiza mediante el comando `test`, que se escribe como `test EXPRESIÓN`, es lo mismo que `[EXPRESSION].`
Las expresiones adoptan las siguientes formas:

| Expression            | Meaning                                       |
| --------------------- | --------------------------------------------- |
| STRING1 **=** STRING2 | the strings are equal                         |
| STRING1 != STRING2    | the strings are not equal                     |
| INTEGER1 -eq INTEGER2 | INTEGER1 is equal to INTEGER2                 |
| INTEGER1 -ge INTEGER2 | INTEGER1 is greater than or equal to INTEGER2 |
| INTEGER1 -gt INTEGER2 | INTEGER1 is greater than INTEGER2             |
| INTEGER1 -le INTEGER2 | INTEGER1 is less than or equal to INTEGER2    |
| INTEGER1 -lt INTEGER2 | INTEGER1 is less than INTEGER2                |
| INTEGER1 -ne INTEGER2 | INTEGER1 is not equal to INTEGER2             |
| -n STRING             | the length of STRING is nonzero               |
| -z STRING             | the length of STRING is zero                  |
| -f FILE               | FILE exists and is a regular file             |
| -s FILE               | FILE exists and has a size greater than zero  |
| -X FILE               | FILE exists and execute (or s                 |
| -d FILE               | FILE exists and is a directory                |

### read

El comando `read` lee una línea de la entrada estándar.

```
#!/bin/bash

MYOS="linux"
echo "Lets see What is your favirite OS"

if [ $MYOS = "linux" ] 
then
 echo "Haha happy to hear that"
else
 echo "I wish you liked linux"
fi
```

### bucles

El propósito de los bucles es repetir el mismo código o uno similar varias veces. Esta cantidad de veces se puede especificar en un número determinado, o la cantidad de veces se puede determinar en función de que se cumpla una determinada condición. Por lo general, existen distintos tipos de bucles, incluidos los bucles for, while y... .

### for

El bucle `for` procesa cada elemento de una secuencia

```
for VAR in LIST;
do
  command1
  command2
done
```

> Preste atención a la sintaxis `;` , `do` , `done` .
> LIST puede ser una lista de cadenas, una matriz o la salida de comandos, etc.

```
#!/bin/bash

for i in 1 2 3 ;
do
 echo $i
done
```

### **seq**

**seq es un comando muy útil. **Genera una secuencia de números para un bucle del PRIMERO al ÚLTIMO en pasos de INCREMENTO.

```
seq [OPTION]... LAST
 ### or
seq [OPTION]... FIRST LAST
 ### or
seq [OPTION]... FIRST INCREMENT LAST
```

```
user1@ubuntu16-1:~/sandbox$ seq 3
1
2
3
user1@ubuntu16-1:~/sandbox$ seq 1 3
1
2
3
user1@ubuntu16-1:~/sandbox$ seq 1 10 2
1
user1@ubuntu16-1:~/sandbox$ seq 1 3 10
1
4
7
10
```

secuencia en bucle for:

```
user1@ubuntu16-1:~/sandbox$ for i in $(seq 1 3); do echo $i; done
1
2
3
```

> También podemos definir un rango con la sintaxis {START..END..INCREMENT} y haría lo mismo.

El bucle for también funciona con cadenas:

```
#!/bin/bash

for MYFILE in $(ls);
do
    echo $MYFILE
done
```

### while

Los bucles while evalúan una condición cada vez que se inicia el bucle y ejecutan la lista de comandos si la condición es verdadera. Si la condición no es verdadera inicialmente, los comandos nunca se ejecutan.

```
while [condition]
do
    command1
    command2
done
```

example:

```
#!/bin/bash

MYVAR=3

while [ $MYVAR -gt 0 ]
do
  echo $MYVAR
  let MYVAR=MYVAR-1
done
```

> El **let** es un comando incorporado y se utiliza para evaluar expresiones aritméticas en variables de shell. ¿Qué significa? Sin let MYVAR=MYVAR-1 tendría el valor "3-1" como cadena.

> Si desea utilizar un valor de variable en una expresión aritmética, no necesita utilizar `$` antes del nombre de la variable, aunque puede hacerlo si lo desea.

**bucles infinitos**

Si la condición del bucle while nunca se cumple, continúa ejecutándose para siempre, eso se llama bucle infinito y tenemos que usar Ctrl+C para romperlo.

¡Los bucles infinitos no son tan malos! El SO es un bucle infinito que verifica constantemente la entrada y la respuesta correspondiente.

**until**\
Los bucles till ejecutan la lista de comandos y evalúan una condición cada vez que finaliza el bucle. Si la condición es verdadera, el bucle se ejecuta nuevamente. Incluso si la condición no es verdadera inicialmente, los comandos se ejecutan al menos una vez.

### exec <a href="mailing-notifications-to-root" id="mailing-notifications-to-root"></a>

El comando **exec** en Linux se utiliza para ejecutar un comando desde el propio bash. Este comando no crea un nuevo proceso, solo reemplaza el bash con el comando que se va a ejecutar. Si el comando exec es exitoso, no regresa al proceso que lo llamó. Pruebe con `exec BLAH` y `exec ls` y compare los resultados.

### Notificaciones por correo al usuario root <a href="mailing-notifications-to-root" id="mailing-notifications-to-root"></a>

Para enviar correo primero necesitamos tener instalado `mailutils`. Luego podemos enviar correo:

```
root@ubuntu16-1:~# mail root@localhost
Cc: 
Subject: test mail!
this is my first email!
```

Supongamos que su script está ejecutando una tarea administrativa en su sistema en medio de la noche mientras usted está profundamente dormido. ¿Qué sucede cuando algo sale mal? Afortunadamente, es fácil enviar por correo electrónico información de error o archivos de registro a usted mismo, a otro administrador o al usuario root. Simplemente envíe el mensaje al comando `mail` y use la opción `-s` para agregar una línea de asunto.

```
user1@ubuntu16-1:~$ echo "Body!" | mail -s "Subject" root@localhost
```

> Si necesita enviar un archivo de registro por correo, use la función de redirección `<` para redirigirlo como entrada al comando `mail`. Si necesita enviar varios archivos, puede usar `cat` para combinarlos y canalizar la salida a `mail`.

**Hoja de trucos de scripts de bash**

También le recomiendo que visite mi hoja de trucos de scripts de bash en: [https://borosan.gitbook.io/bash-scripting/](https://borosan.gitbook.io/bash-scripting/)


[https://developer.ibm.com/tutorials/l-lpic1-105-2/](https://developer.ibm.com/tutorials/l-lpic1-105-2/)

[https://www.poftut.com/linux-bin-bash-shell-and-script-tutorial/](https://www.poftut.com/linux-bin-bash-shell-and-script-tutorial/)

[https://www.geeksforgeeks.org/declare-command-in-linux-with-examples/](https://www.geeksforgeeks.org/declare-command-in-linux-with-examples/)

[https://www.gnu.org/software/bash/manual/html_node/Command-Substitution.html](https://www.gnu.org/software/bash/manual/html_node/Command-Substitution.html)

[https://jadi.gitbooks.io/lpic1/content/1052\_customize_or_write_simple_scripts.html](https://jadi.gitbooks.io/lpic1/content/1052\_customize_or_write_simple_scripts.html)

[https://www.computerhope.com/unix/test.htm](https://www.computerhope.com/unix/test.htm)

[https://www.dpscomputing.com/blog/2012/09/13/programming-the-purpose-of-loops/](https://www.dpscomputing.com/blog/2012/09/13/programming-the-purpose-of-loops/)

[https://www.geeksforgeeks.org/seq-command-in-linux-with-examples/](https://www.geeksforgeeks.org/seq-command-in-linux-with-examples/)

[https://linux4one.com/bash-for-loop-with-examples/](https://linux4one.com/bash-for-loop-with-examples/)

[https://www.geeksforgeeks.org/let-command-in-linux-with-examples/](https://www.geeksforgeeks.org/let-command-in-linux-with-examples/)

[https://www.quora.com/What-are-some-practical-uses-of-infinite-loops](https://www.quora.com/What-are-some-practical-uses-of-infinite-loops)

[https://www.geeksforgeeks.org/exec-command-in-linux-with-examples/](https://www.geeksforgeeks.org/exec-command-in-linux-with-examples/)

.
