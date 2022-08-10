# django-heroku

Vamos a ver como subir nuestro proyecto django a un servidor de heroku, tambien vamos a ver como migrar nuestra db a una base de datos postgresql (el tipo de db que usa heroku)

Vamos a configurar el proyecto local que trabajamos antes:

https://github.com/carabedo/django

Para tenerlo en produccion con todas sus funcionalidades asi:

https://djpg.herokuapp.com

## Indice:

- [creando repo](https://github.com/carabedo/django-heroku#creemos-un-repositorio-remoto-nuevo)
- [migrando a postgres](https://github.com/carabedo/django-heroku#migramos-nuestra-db-de-sqlite-a-postgres)
- [heroku](https://github.com/carabedo/django-heroku#subiendo-a-heroku-el-proyecto)


## Creamos un entorno virtual

Necesitamos crear un entorno virtual para encapsular todas las librerias que vamos a usar para nuestro sitio.

```bash
mkdir primer_proyecto
cd primer_proyecto
```

Con el parametro prompt le damos nombre al enviroment:

```
python3 -m venv ./venv --prompt primer_proyecto
```

Luego vemos lo siguiente:

```
primer_proyecto/
├─ venv
```

Activamos el enviromente:

```
source venv/bin/activate
```



### Clonamos el projecto final:

```bash
git clone https://github.com/carabedo/django.git
```


Luego vemos lo siguiente:

```
primer_proyecto/
├─ venv
├─ django
```

Abrimos el proyecto y levantamos el servidor `python3 manage.py runserver` instalamos todas las librerias que necesite el proyecto.

Adicionalmente instalamos gunicorn todas las librerias que vayamos a necesitar para subir nuestro proyecto a heroku :

```
pip3 install gunicorn
pip3 install whitenoise
pip3 install dj-database-url
pip3 install psycopg2
```

Luego de haber instalado todas las librerias vamos a editar nuestro proyecto `django` para ponerlo en produccion en heroku:

En `settings.py` tenemos que cambiar:

```python
##primer_proyecto\settings.py
ALLOWED_HOSTS = ['*']
DEBUG = False
```

## Debemos agregar al settings.py


```
## notar que se agrego un middleware luego del security

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'whitenoise.middleware.WhiteNoiseMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]




#esto es para los staticfiles
STATIC_ROOT = BASE_DIR / "staticfiles"

# Enable WhiteNoise's GZip compression of static assets.
STATICFILES_STORAGE = "whitenoise.storage.CompressedManifestStaticFilesStorage"
```

# Archivos necesarios para Heroku:

#### `requirements.txt`

Usamos el siguiente comando con el entorno del proyecto activado:

```
python3 -m pip freeze > requirements.txt
```

Nos deberia quedar algo similar:

```
asgiref==3.5.2
Django==4.1
gunicorn==20.1.0
Pillow==9.2.0
sqlparse==0.4.2
whitenoise==6.2.0
dj-database-url
psycopg2
``` 
Es por esta razon que creamos un entorno nuevo, por que si no lo haciamos este comando (freeze) listaria todas las librerias de nuestra computadora, librerias que seguramente nuestro proyecto no esta usando.


#### `Procfile`

Ingresamos la siguiente linea:

```
web: gunicorn primer_proyecto.wsgi --log-file -
```

#### `runtime.txt`

Debemos agregar la version de python de nuestro venv, podemos saber cual es usando `python3 --version`.

```
python-3.10.4
```

Finalmente deberiamos tener la siguiente estructura:

```
primer_proyecto/
├─ venv
├─ django
    ├─ manage.py
    ├─ app_prueba/
    ├─ primer_proyecto/
    ├─ contact/
    ├─ media/
    ├─ portfolio/
    ├─ templates/
    ├─ requirements.txt
    ├─ Procfile
    ├─ runtime.txt
    ├─ db.sqlite3
```


# Creemos un repositorio remoto nuevo:

Primero generamos un repositorio vacio sin readme.md en [GITHUB](https://github.com/new)

Removemos la url remota del repo original:

```
git remote rm origin
```

Ahora vamos a agregar la URL del repo remoto:

```
git remote add origin  <REMOTE_URL> 
```

Antes de hacer el `push` tenemos que generar un token para nuestra autentificacion lo usamo como password:

https://techglimpse.com/git-push-github-token-based-passwordless/

Y finalmente subimos todo al repo remoto:

```
git push --set-upstream origin main
```

Cuando nos pregunte por la contraseña copiamos y pegamos el token.

Tambien podemos setearlo usando:

```
git config --global user.name "username"
git config --global user.password "token"
```

# Migramos nuestra db de sqlite a postgres

## Instalamos postgresql

```
sudo apt install libpq-dev postgresql postgresql-contrib
```

Nos aseguramos que el servidor de postgresql este funcionando:

``` 
sudo service postgresql start
```

Hay que agregar nuestro usuario de linux (fernando) a postgresql, creamos el acceso a la db de nuestro usuario de linux, usando el '--createdb' le damos permiso de crear una db y con '--pwprompt' le vamos a poder asignar un password (1234) que va a ser **importantisimo** mas adelante.

```
sudo su - postgres -c "createuser fernando --createdb --pwprompt"
```

Creamos la db

```
sudo -u postgres createdb testdb
```

Le damos acceso a nuestro usuario de linux (fernando) a la db recien creada:

Con esta linea nos logeamos a la consola de postgresql
```
sudo -u postgres psql
```

Ejecutamos esto para darle privilegios a nuestro usuario:

```postgresql
grant all privileges on database testdb to fernando;
```
Salimos del psql con `\q`


## Migrando las db de sqlite3 a postgresql con `pgloader` 

Instalamos: https://pgloader.readthedocs.io/en/latest/

```
sudo apt-get install pgloader
```

Ahora si hacemos la migracion, vamos a la carpeta donde este la db de sqlite (db.sqlite3) que queremos migrar la base (testdb) de postgres y ejecutamos:

```
pgloader db.sqlite3 postgresql:///testdb
```
Este proyecto tiene 2 db, podemos importar varias con pgloader!

```
pgloader old.db postgresql:///testdb
```

No olvidar remover de las vistas las llamadas a la segunda db 'old' del proyecto!!! por que cuando migremos solo habra una db.

Deberiamos ver un monton de lineas donde se leen los nombres de las tablas de nuestra db de sqlite.

Probemos que haya funcionado conectandonos a la db de postgres:

Para eso, volvemo a la terminal psql con el usuario 'postgres':

```
sudo -u postgres psql
```

En la terminal de psql, listamos las dbs con `\l` y nos conectamos a la db con `\c testdb`. Por ultimo listamos las tabalas con `\dt`. 

Deberiamos ver todas las tablas del proyecto de django. 

Por ultimo, cambiamos el `settings.py` del proyecto cambiamos la db y agregamos una funcion que al subir el proyecto a heroku adjunte la db remota que vamos a crear despues.

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'testdb',
        'USER': 'fernando',
        'PASSWORD': '1234',
        'HOST': 'localhost',
        'PORT': '',
    }
}

import dj_database_url
db_from_env = dj_database_url.config(conn_max_age=600)
DATABASES['default'].update(db_from_env)
```
# Subiendo a Heroku el proyecto

## Instalamos heroku-cli 

Nos hacemos una cuenta en heroku y luego instalamos el cli:

https://devcenter.heroku.com/articles/heroku-cli

Una vez instalado nos logeamos por consola:

```
heroku login -i
```

Creamos nuestra nueva app:

```
heroku create django-pg
```

Ahora agregamos la opcion de usar una db remota de heroku para la app:

```
heroku addons:create heroku-postgresql:hobby-dev --app django-pg
```

Veremos algo parecido a:

```
Creating heroku-postgresql:hobby-dev on ⬢ djpg... free
Database has been created and is available
 ! This database is empty. If upgrading, you can transfer
 ! data from another database with pg:copy
Created postgresql-fluffy-3333 as DATABASE_URL
```

Guardemos la `DATABASE_URL`

Otra manera de ver el nombre de esta db remota es usando:

```
heroku pg:info --app django-pg
```

Deberias ver algo asi:

```
Plan:                  Hobby-dev
Status:                Available
Connections:           2/20
PG Version:            14.4
Created:               2022-08-09 21:56 UTC
Data Size:             10.0 MB/1.00 GB (In compliance)
Tables:                13
Rows:                  108/10000 (In compliance)
Fork/Follow:           Unsupported
Rollback:              Unsupported
Continuous Protection: Off
Add-on:                postgresql-fluffy-3333
```

Lo que tenes que guardar es el campo `Add-on` ese es el nombre de la db.

Finalmente migramos la db local a la db remota!

```
PGUSER=fernando PGPASSWORD=1234 heroku pg:push postgres://localhost/testdb postgresql-fluffy-3333 --app django-pg
```

Notar que ese pass es el que usamos al crear el usuario de postgress antes, `postgresql-fluffy-3333` esto es lo que tenes que cambiar al nombre que te aparece para tu db remota.


Ahora tenemos que publicar nuestra app:

```
heroku git:remote -a django-pg
```

```
git add.
git commit -m 'first'
git push heroku main
```

FIN~!
