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
	path('admin/', admin.site.urls),
```
> cd ..

1.4) Correr proyecto
> python manage.py runserver
 

*************************************************
2) APLICACIONES Y MODELOS
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
from APLICACION.models import MODELO

class MODELOADMIN(MODELO):
	#Busca "The Django admin site" en la documentación de Django
```
> cd ..

2.4) Migra tus modelos



******************
PASOS FINALES ANTES DE LIBERAR A PRODUCCIÓN



