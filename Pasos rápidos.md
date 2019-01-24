PASOS GENERICOS PARA REALIZAR PROYECTOS EN DJANGO DE MANERA FLUIDA
==================================================================

## 1) PASOS INICIALES ##
1.1) Crea proyecto
> django-admin startproyect PROYECTO

1.2) Ingresa a tu proyecto
> cd PROYECTO

1.3) En url.py del proyecto, agregar la línea para las vistas de admin (si se requieren)
> cd PROYECTO

> vi urls.py
```
#Añadir en el list de urlpatterns
urlpatterns = [
	path('admin/', admin.site.urls),
]
```
> cd ..

1.4) Crear un usuario superadministrador
> python manage.py createsuperuser

1.5) Correr proyecto
> python manage.py runserver


*************************************************

## 2) PREPARATIVOS PREVIOS ##

2.1) Uso de templates.
Se crea la carpeta "templates" y se declara en el archivo setting.py.

> mkdir templates

> cd PROYECTO

> vi settings.py

```
TEMPLATES = [
    {
	# Se le carga el siguiente valor al indice 'DIRS'
	'DIRS': [os.path.join(BASE_DIR, 'templates')],
        # Se añade en el diccionario dentro del list TEMPLATES la siguiente línea
        'APP_DIRS': True,
    },
]
```

> cd ..

2.2) Manejo de imagenes (media)
En el archivo original de urls.py se añade la línea siguiente al final del list de la variable urlpatterns

> cd PROYECTO

> vi urls.py

```
urlpatterns = [
	...
	]  + static(settings.MEDIA_URL,  document_root=settings.MEDIA_ROOT)
```

> cd ..

En settings.py se añaden las siguientes dos líneas

> cd PROYECTO

> vi settings.py

```
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
MEDIA_URL = '/media/'
```

> cd ..

2.3) Manejo de archivos estáticos (css, ims, js)
Se crea la carpeta "static" y se declara en el archivo setting.py.

> mkdir static

> cd PROYECTO

> vi settings.py

```
# Añadir las siguientes líneas

STATIC_URL = '/static/'

STATICFILES_DIRS = (
	os.path.join(BASE_DIR, 'static'),
	)

STATICFILES_FINDERS = [
    'django.contrib.staticfiles.finders.FileSystemFinder',
    'django.contrib.staticfiles.finders.AppDirectoriesFinder',
]
```

> cd ..


*************************************************
## 3) APLICACIONES Y MODELOS ##
3.1) Crea apps
> python manage.py startapp APLICACION

3.2) En cada proyecto, crea tu modelo
> cd APLICACION

> touch MODELO.py

> vi MODELO.py
```
from django.db import models

class MODELO(models.Model):
	#Busca "Models" y "Model field reference" en la documentación de Django
```
> cd ..

3.3) De ser requerido, otorga mecanismos al modelo para ser accesible con el admin

> cd APLICACION
> touch admin.py

> vi admin.py

```
from django.contrib import admin
from APLICACION.models import MODELO

@admin.register(MODELO)
class MODELOADMIN(admin.ModelAdmin):
	#Busca "The Django admin site" en la documentación de Django
```
> cd ..

3.4) Añade tu aplicación al proyecto
> cd PROYECTO

> vi settings.py
```
#En el list de INSTALLED_APPS agregar la siguiente línea
INSTALLED_APPS = [
	...
	'APLICACION',
	...
]
```


3.5) Migra tus modelos
> python manage.py makemigrations

> python manage.py migrate

*************************************************
## 4) VISTAS ##

4.1) En el archivo de urls principal, se agrega la siguiente línea para apuntar a las urls específicas de la aplicación.

> cd PROYECTO

> vi urls.py
```
<<<<<<< HEAD
#Procurar importar la clase include
from django.urls import path, include
=======
>>>>>>> f65ba1ab784032b58a0b49f0854ec07461ab9c18
#Añadir en el list de urlpatterns
urlpatterns = [
	path('', include (('APLICACION.urls','APLICACION'),namespace='APLICACION')),
]
```
> cd ..

4.2) En la aplicación, valida que exista un archivo urls.py y dentro de esta agrega la URL de tu vista.

> cd APLICACION

> touch urls.py

> vi urls.py

```
from django.urls import path
from APLICACION import views

urlpatterns = [
	#Añade tu url. Busca "URL dispatcher" en la documentación de Django
	]
```

> cd ..

4.3) En los views crear la lógica de las vistas

> cd APLICACION

> touch views.py

> vi views.py 

```
def VISTA(request):
	#Si se usan funciones de vista, se puede consultar "Writing views" en la documentación de Django


class CLASSVIEW(VISTAS_REQUERIDA):
	#Al usar Class View, se puede consultar "Class-based views" en la documentación de Django
```

> cd ..

******************
## PASOS FINALES ANTES DE LIBERAR A PRODUCCIÓN ##



