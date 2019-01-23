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
## 2) APLICACIONES Y MODELOS ##
2.1) Crea apps
> python manage.py startapp APLICACION

2.2) En cada proyecto, crea tu modelo
> cd APLICACION

> touch MODELO.py

> vi MODELO.py
```
from django.db import models

class MODELO(models.Model):
	#Busca "Models" y "Model field reference" en la documentación de Django
```
> cd ..

2.3) De ser requerido, otorga mecanismos al modelo para ser accesible con el admin

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

2.4) Añade tu aplicación al proyecto
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


2.5) Migra tus modelos
> python manage.py makemigrations

> python manage.py migrate



******************
## PASOS FINALES ANTES DE LIBERAR A PRODUCCIÓN ##



