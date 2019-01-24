Instalar un proyecto de Django en un servidor CentOS
====================================================

Primero que nada, es un placer mencionar que recientemente he acabado el curso de Django de Platzi y he recordado muchas cosas, así como aprendido unas cuantas más que en proyectos anteriores yo desconocía.

Al momento de implementar mi proyecto de Django en producción, me dió curiosidad de como sería en caso de que el servidor fuera Redhat o CentOS, que son distribuciones de Linux con las que he trabajado en el pasado.

Les quiero compartir los pasos que hice para implementar mi proyecto de Platzigram en CentOS versión 7.6. Espero que les sea útil.

Como referencia, utilicé las siguientes dos URL como guía. Obvio tuve que adaptarlos para este proyecto en particular.

https://simpleisbetterthancomplex.com/tutorial/2017/05/23/how-to-deploy-a-django-application-on-rhel.html

https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-centos-7

Si ven que hay pasos que no funcionan correctamente, no duden de avisarme para hacer pruebas y ver en conjunto que sucedió.

##Modificaciones de proyecto Django##

En el archivo settings.py deben hacer lo siguiente.

Configurar que el modo debug sea False
Permitir el acceso a su página de hosts del exterior. En este ejemplo puse "ALLOWED_HOST = [*] pensando que cualquier URL puede entrar a nuestra red social, aunque dependiendo de sus necesidades, pueden limitarlo a ciertas IP’s o segmentos.
Configurar la base de datos Postgresql. En este caso usaremos como nombre de base de datos el de “platzi” y el usuario postgresql como “ingkstr”

```
DEBUG = False

ALLOWED_HOSTS = ['*']

DATABASES = {
	'default':{
		'ENGINE':''django.db.backends.postgresql_psycopg2',
		'NAME':'platzi',
		'USER':'ingkstr',
		'PASSWORD':'abcd1234',
		'HOST':'127.0.0.1',
		'PORT':'5432',
	}
}
```

## Creación de archivo requirements.txt ##

Al momento de instalar los componentes de Python a usar en nuestro proyecto, podemos hacerlo uno a uno manual, o cargar un archivo “requirements.txt” con los componentes para instalar de golpe.

```
Django==2.1.2
django-allauth==0.37.1
pillow==5.3.0
gunicorn==19.7.1
psycopg2==2.7.1
```

## Alta de archivo en Github ##

Aprovechando para practicar sus conocimientos de Github, pueden cargar su proyecto en dicho portal y con ello se ahorran tener que instalar un servidor SFTP o copiarlo manualmente con un dispositivo extraible.

## Actualizar CentOS ##
Como el caso de Ubuntu, realizaremos como buena práctica la actualización de nuestro servidor.

```
sudo yum update
sudo yum upgrade
sudo yum install centos-release-scl
shutdown -r now 
```

## Actualizar Python ##
CentOS al ser instalado viene con la versión 2.6. Recomiendo actualizarlo al 3.6, así como los componentes git, gcc y virtualenv

```
sudo yum install rh-python36
scl enable rh-python36 bash
sudo yum -y install git gcc python-virtualenv
```

El comando “scl enable rh-python36 bash” es para que la versión default de Python sea la 3.6. Lo puedes comprobar con el siguiente comando al final

```
python --version
```

## Creación de usuario para implementación ##
Crearemos el grupo “platzigram”, y dentro de este se crea el usuario “platzigram”, cuya carpeta raíz será /opt/platzigram

```
sudo groupadd --system platzigram
sudo useradd --system --gid platzigram --shell /bin/bash --home /opt/platzigram platzigram
```

Creamos la carpeta raíz del usuario platzigram y otorgarle privilegios

```
sudo mkdir /opt/platzigram
sudo chown platzigram:platzigram /opt/platzigram
```

## Instalación y configuración de Postgresql ##

Primero cargemos Postgresql a nuestro servidor

```
sudo yum -y install https://yum.postgresql.org/9.6/redhat/rhel-7-x86_64/pgdg-redhat96-9.6-3.noarch.rpm
sudo yum -y install postgresql96-server postgresql96-contrib postgresql96-devel
```

Luego inicializamos y habilitamos el servicio

```
sudo /usr/pgsql-9.6/bin/postgresql96-setup initdb
sudo systemctl start postgresql-9.6
sudo systemctl enable postgresql-9.6
```

Luego entramos al usuario “postgres” (no pide password) y creamos nuestro usuario y la base de datos (en mi caso fue el usuario “ingkstr” y la base de datos “platzi”

```
sudo su - postgres
createuser ingkstr
psql -c "ALTER USER ingkstr WITH PASSWORD 'abcd1234';"
createdb --owner ingkstr platzi
exit
```

Al final, haremos unos cambios en el archivo “/var/lib/pgsql/9.6/data/pg_hba.conf”

Al entrar a este archivo, buscamos las siguientes líneas

```
# IPv4 local connections:
host    all             all             127.0.0.1/32            ident
# IPv6 local connections:
host    all             all             ::1/128                 ident
```

El cambo a realizar es cambiar las leyendas “ident” a “md5”.

```
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5  
# IPv6 local connections:
host    all             all             ::1/128                 md5
```

Al final, reiniciamos nuestr servicio

```
sudo systemctl restart postgresql-9.6
```

## Creación de entorno virtual ##

Entramos a nuestro usuario “platzigram”. Verán que nos va a salir el prompt de “-bash-4.2$”, lo cual es normal. Si le dan un “pwd”, verán que estarán posicionados en la carpeta “/opt/platzigram”

```
sudo su - platzigram
```

Es necesario poner de nuevo que la versión default de Python a usar para este usuario sea la 3.6

```
scl enable rh-python36 bash
```

Creamos un entorno “venv” y lo activamos.

```
virtualenv venv
source venv/bin/activate
```

Creamos una carpeta de logs y run para nuestra implementación.

```
mkdir logs
mkdir run
```

Si cargaron su proyecto en git, es el momento para clonarlo

```
git clone [URL HTTPS O SSH GIT]
```

Se da de alta el siguiente path de Postgresql

```
export PATH=$PATH:/usr/pgsql-9.6/bin/
```

Actualizamos nuestra paquetería de instalación de pip

```
pip install pip --upgrade
```

Entramos en la carpeta de nuestro proyecto descargado (en mi caso, se llama también “platzigram”) e instalamos los componentes de nuestro archivo “requirements”.

Después, hacemos la migración de nuestra base de datos a Postgresql, así como la carga de archivos estáticos de nuestro portal admin.

```
cd platzigram
pip install -r requirements.txt 
python manage.py makemigrations
python manage.py migrate
python manage.py collectstatic --noinput
```

## Instalación y configuración de Gunicorn ##
Nos salimos de nuestro usuario “platzigram”

```
su -
```

Creamos el archivo /etc/systemd/system/gunicorn.service con el siguiente contenido

```
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=platzigram
Group=platzigram
WorkingDirectory=/opt/platzigram/platzigram
ExecStart=/opt/platzigram/venv/bin/gunicorn  --access-logfile - --workers 3 --bind unix:/opt/platzigram/run/gunicorn.sock platzigram.wsgi:application

[Install]
WantedBy=multi-user.target
```

Al final, iniciamos y activamos el servicio de gunicorn.

```
sudo systemctl start gunicorn
sudo systemctl enable gunicorn
```

## Instalación y configuración de NGINX ##

Creamos el archivo “/etc/yum.repos.d/nginx.repo” con el siguiente contenido

```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/rhel/7/$basearch/
gpgcheck=0
enabled=1
```

Instalamos nginx y habilitamos la salida de protocolo http

```
sudo yum -y install nginx policycoreutils-python
sudo semanage permissive -a httpd_t 
```

Nos dirigimos a la carpeta de configuración de nginx, y eliminamos el archivo “default.conf”

```
cd /etc/nginx/conf.d/
sudo rm -r default.conf
```

Creamos el archivo “platzigram.conf” con el siguiente contenido

```
upstream app_server {
    server unix:/opt/platzigram/run/gunicorn.sock fail_timeout=0;
}

server {
    listen 80;
    server_name IP_ADDRESS_OR_DOMAIN_NAME;  # <- insert here the ip address/domain name

    keepalive_timeout 5;
    client_max_body_size 4G;

    access_log /opt/platzigram/logs/nginx-access.log;
    error_log /opt/platzigram/logs/nginx-error.log;

    location /static/ {
        alias /opt/platzigram/platzigram/static/;
    }

    location /media/ {
        alias /opt/platzigram/platzigram/media/;
    }

    location / {
        try_files $uri @proxy_to_app;
    }

    location @proxy_to_app {
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header Host $http_host;
      proxy_redirect off;
      proxy_pass http://app_server;
    }
}
```

Testeamos que nuestro archivo este creado correctamente.

```
sudo nginx -t
```

Iniciamos el servicio

```
sudo systemctl start nginx
sudo systemctl enable nginx
```

## Pasos finales ##

Reiniciamos nuestro servidor Linux

```
sudo reboot
```

CentOS tiene una serie de tablas de seguridad. Para probar nuestro servicio, quitaremos sin guardar todas las reglas. Al reiniciar nuestro servidor, estas volverán a la normalidad (esto para que se configuren después a consciencia, ya que quitar todas las reglas es una fuerte vulnerabilidad)

```
iptables -F
```
