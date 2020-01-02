# Buildout base para Odoo 12

## Instrucciones
1. Instalar PostgreSQL 10. No probado con Postgre 11. 
Como se han eliminado en el buildout las referencias a PostgreSQL, se debe instalar primero, de forma independiente a buildout. 
**No es necesario crear la Base de Datos de Odoo**: ya se encargará Odoo de crearla la primera vez que lo iniciemos (con el nombre que indiquemos en devel-odoo.cfg)  

2. Instalar las dependencias de Anybox
* Añadir el repositorio a `/etc/apt/sources.list`:
```
$ sudo nano /etc/apt/sources.list
deb http://apt.anybox.fr/openerp common main
$ sudo apt update
```
* Añadir la firma:
```
$ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com --recv-keys 0xE38CEB07
```
* Si no funciona lo anterior: clicar en el enlace pub y copiar a un archivo de texto.
```
$ sudo apt-key add key.txt
```

3. Instalar dependencias python3
```
$ sudo apt-get install python3-dev
```
4. Actualizar e instalar
```
$ sudo apt-get update
$ sudo apt-get install openerp-server-system-build-deps
```

- Para poder compilar e instalar postgres

```
$ sudo apt-get install libreadline-dev
```

5. Crear un **virtualenv** dentro de la carpeta del respositorio.
Si no está instalado, instalar el paquete de virtualenv. Es necesario tener la versión que se instala con easy_install o con pip, desinstalar el paquete python-virtualenv si fuera necesario e instalarlo con easy_install
```
$ sudo easy_install virtualenv
$ virtualenv -p python3.5 sandbox
```

6. Ahora procedemos a ejecutar el buildout en nuestro entorno virtual

```
$ sandbox/bin/python3.5 bootstrap.py -c [archivo_buildout]
$ sandbox/bin/python3 bootstrap.py --setuptools-version=40.8.0 -c devel-buildout.cfg
```

7. Lanzar buildout (el -c [archivo_buildout] se usa cuando no tiene el nombre por defecto buildout.cfg)

```
$ bin/buildout -c [archivo_buildout]
```

8. Si falla al instalar xmlsec ....
```
$ apt-get install libxml2-dev libxmlsec1-dev libxmlsec1-openssl
```

- Actualmente supervisor no funciona en python3, por lo que si no se instala manualmente es necesario lanzar postgres y odoo con los comandos
```
$ parts/postgres/bin/postmaster --config-file=etc/postgresql.conf
$ bin/start_odoo
```

- Puede que de error, hay que lanzar el supervisor y volver a hacer bin/buildout:
```
$ bin/supervisord
$ bin/buildout -c [archivo_buildout]
```

- Conectarse al supervisor con localhost:9002
- Si fuera necesario hacer update all, se puede parar desde el supervisor y en la consola hacer:
```
$ cd bin
$ ./upgrade_odoo
```
- odoo se lanza en el puerto 9069 (se pude configurar en otro)

## Securizar el acceso al supervisor
```
$ sudo apt-get install iptables
$ sudo iptables -A INPUT -i lo -p tcp --dport 9002 -j ACCEPT
$ sudo iptables -A INPUT -p tcp --dport 9002 -j DROP
$ sudo apt-get install iptables-persistent (marcamos "yes" en las preguntas que nos hace al instalarse)
```

## Configurar Odoo
Archivo de configuración: etc/odoo.cfg, si se quieren cambiar opciones en  odoo.cfg, no se debe editar el fichero,
si no añadirlas a la sección [odoo] del buildout.cfg
y establecer esas opciones .'add_option' = value, donde 'add_option'  y ejecutar buildout otra vez.

Por ejemplo: cambiar el nivel de logging de odoo
```
'buildout.cfg'
...
[odoo]
options.log_handler = [':ERROR']
...
```

Si se quiere ejecutar más de una instancia de odoo, se deben cambiar los puertos,
please change ports:
```
odoo_xmlrpc_port = 9069  (8069 default odoo)
odoo_xmlrpcs_port = 9071 (8071 default odoo)
supervisor_port = 9002      (9001 default supervisord)
postgres_port = 5434        (5432 default postgres)
```

## Creators

Rastislav Kober, http://www.kybi.sk
