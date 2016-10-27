
# Práctica 3: Integración continua y trabajo en equipo

- [1. Objetivos](#1-objetivos-y-resumen-de-la-práctica)
- [2. Conectar el proyecto `mads-todolist` con Travis](#2-conectar-el-proyecto-mads-todolist-con-travis)
    - [2.1. Cómo darse de alta en Travis](#21-cómo-darse-de-alta-en-travis-y-conectar-el-repositorio)
    - [2.2. Cómo configurar el build en Travis](#22-cómo-configurar-el-build-en-travis)
- [3. Formación de equipos](#3-formación-de-equipos)
- [4. Nuevo flujo de trabajo en Trello y GitHub](#4-nuevo-flujo-de-trabajo-en-trello-y-github)
    - [4.1. Rama remota en la que trabajan más de un compañero](#41-rama-remota-en-la-que-trabajan-más-de-un-compañero)
    - [4.2. Pull request en GitHub](#42-pull-request-en-github)
- [5. Representación de _features_ en Trello](#5-representación-de-features-en-trello)
- [6. Desarrollo de 3 features en equipo](#6-desarrollo-de-3-features-en-equipo)
    - [6.1. Feature 1: Login, registro y tareas](#61-feature-1-login-registro-y-tareas)
    - [6.2. Feature 2: Proyectos](#62-feature-2-proyectos)
- [7. Entrega y evaluación](#7-entrega-y-evaluación)

## 1. Objetivos y resumen de la práctica

En esta práctica se pretende conseguir:

- Publicación del proyecto en Travis, un servicio de integración
  continua.
- Formar equipos de trabajo y configurar las herramientas (Trello,
  GitHub) para el trabajo conjunto.
- Adaptar el flujo de trabajo en Git y en Trello al trabajo en
  equipo y al trabajo con _features_ de mayor tamaño que los tickets.

## 2. Conectar el proyecto `mads-todolist` con Travis


> **Tarea a realizar**: Hay que integrar el repositorio con Travis
> para que realice el build del proyecto y pase los tests en todas las
> ramas del mismo.  Deberás modificar el proyecto incluyendo la
> configuración para Travis y modificando el README.md para que
> incluya el icono del estado del último build de la rama master.
> Deberás realizar todo ello en un **nuevo ticket** abierto en Trello
> llamado "Integración con Travis".

Travis es un servicio de integración continua que se integra
fácilmente con GitHub. Tiene una versión gratuita para proyectos
abiertos ([travis-ci.org](https://travis-ci.org)) y una versión de
pago para proyectos privados
([travis-ci.com](https://travis-ci.org)). La cuenta educativa de
GitHub nos da permisos para trabajar en la versión de pago de forma
gratuita.

Travis se conecta con GitHub y lanza un proceso de _build_ que
descarga, construye y prueba todas las ramas del repositorio cada vez
que se realiza algún nuevo commit en alguna de ellas.


Una vez hecha la integración con Travis se podrá comprobar en cada
commit de GitHub si han pasado los tests correctamente:

<img src="imagenes/build-commits.png" width="600px">

A su vez, en Travis, podrás acceder y consultar los detalles de cada
build:

<img src="imagenes/build-travis.png" width="600px">


### 2.1. Cómo darse de alta en Travis y conectar el repositorio

Para darte de alta en Travis debes acceder a
[travis-ci.com](https://travis-ci.com) autentificándote desde GitHub.

Debes seguir las instrucciones que aparecerán para conectar el
repositorio `mads-todolist` con Travis. Comprueba en los ajustes que
el repositorio se ha conectado correctamente con Travis:
 
<img src="imagenes/repository-settings.png" width="700px">


### 2.2. Cómo configurar el build en Travis

El build en Travis se configura con el fichero `.travis.yml` que debe
estar en la raíz del repositorio.

**Crea una rama nueva** en el repositorio local, añade el fichero de
configuración `.travis.yml` y súbela al repo remoto en GitHub:


```
$ git checkout -b tic-20-integracion-travis
# subimos la rama al repositorio remoto
$ git push -u origin tic-20-integracion-travis
# añadimos .travis.yml, hacemos commit y subimos los cambios
$ git add .
$ git commit -m "Añadido .travis.yml"
$ git push 
```

El fichero de configuración es el siguiente:

**Fichero `.travis.yml`**: 

```
language: scala
sudo: false
addons:
  apt:
    packages:
      - oracle-java8-installer
scala:
  - 2.11.7
jdk:
  - oraclejdk8
cache:
  directories:
    - $HOME/.ivy2/cache
before_cache:
  # nos aseguramos de que los cambios en la cache no persisten
  - rm -rf $HOME/.ivy2/cache/com.typesafe.play/*
  - rm -rf $HOME/.ivy2/cache/scala_*/sbt_*/com.typesafe.play/*
  # borramos todos los archivos de ivydata ya que se tocan en cada build
  - find $HOME/.ivy2/cache -name "ivydata-*.properties" -print0 | xargs -n10 -0 rm
```

Una vez hecho el push, Travis detectará automáticamente el cambio, la
nueva rama y realizará el build, pasando todos los tests. Podremos ver
en tiempo real la ejecución de los tests, en el frontal de Travis y en
`https://travis-ci.com/<usuario>/mads-todolist`.

Cuando pasen correctamente los tests (tarda unos 6 minutos la primera
vez y menos de 3 las siguientes) podrás ver el tick en el commit de GitHub.

Ahora ya puedes hacer el **merge con master**, subir master y borrar la
rama local:

```
$ git checkout master
$ git merge tic-20-integracion-travis
$ git push 
$ git branch -d tic-20-integracion-travis
$ 
```

Puedes borrar la rama remota desde línea de comando:

```
$ git push origin --delete tic-20-integracion-travis
```

También se puede borrar desde la web de GitHub en la página de _branches_.


### 2.3. Modificación del README del proyecto

Por último, una vez que la rama master esté pasando los tests
correctamente en Travis, renombra el fichero `README` a `README.md`
(Markdown) y modifica su contenido para que muestre una presentación
del repositorio en la que aparezca la imagen del estado del último
build realizado en Travis.

La imagen final del `README.md` deberá ser similar a la siguiente:

<img src="imagenes/readme.png" width="500px">

Puedes consultar cómo embeber esta imagen en la página de
documentación de Travis
[Embedding Status Images](https://docs.travis-ci.com/user/status-images/).

## 3. Formación de equipos

> **Tarea a realizar**: Formar un equipo de 3 o 4 personas, subir la
> composición al foro de Moodle. Seleccionar el repositorio que va a
> usar el equipo y añadir los miembros tanto en GitHub como en el
> tablero Trello del repositorio. Por último cambiar el nombre del
> proyecto Trello y llamarlo **TodoList (Equipo &lt;número&gt;)**


## 4. Nuevo flujo de trabajo en Trello y GitHub

> **Tareas a realizar**:
>
> (1) Probar en el repositorio prueba-git el nuevo flujo de trabajo en
> el que todos los miembros del equipo trabajen sobre una rama
> compartida en el repositorio remoto y terminen integrando los
> cambios usando un pull request.
>
> (2) Probar en el repositorio mads-todolist el nuevo flujo de trabajo
> creando un nuevo ticket en el que se deberá añadir al proyecto una
> página "Acerca de" con la lista de miembros del equipo y la fecha y
> el número de versión (1.0). Todos los miembros del equipo deberán
> participar en el ticket y añadir cada uno su nombre a la lista.

Como ya tenemos equipos de trabajo, debemos adaptar el flujo de
trabajo tanto en Trello como en GitHub a más de una persona.

Cambiaremos lo siguiente:

- **Selección en Trello**: Al pasar un ticket de "Seleccionado" a "En
  marcha" se debe asignar un responsable.
- **Nueva rama con el ticket**: El responsable será el que abra una
  rama nueva para el desarrollo del ticket y la subirá a
  GitHub. También añadirá la URL de la rama a la descripción de la
  tarjeta de Trello.
- **Desarrollo**: Se trabaja en la rama. Cualquier compañero puede
  unirse al ticket y trabajar junto con el responsable.
- **Pull request**: Cuando el ticket se ha terminado, el responsable
  abre un pull request en GitHub, pone el enlace al pull request en la
  tarjeta de Trello y la pasa a una nueva columna llamada **En pull
  request**.
- **Revisión de código**: Los miembros del equipo revisan el código en
  el pull request. Al final, todos los miembros del equipo deben dar
  el OK.
- **Integración del pull request**: Cuando todos dan el OK, el
  responsable de la tarea integra el pull request en master haciendo
  previamente un rebase para actualizar los cambios que otros hayan
  podido subir a master.
- **Actualización de los repositorios locales**: Todos hacen un `pull`
  en `master` para actualizar los cambios del pull request. Y se borra
  la rama local ya integrada.

A continuación explicamos con más detalle algunos aspectos del flujo
de trabajo.

### 4.1. Rama remota en la que trabajan más de un compañero

Veamos algunos comandos de Git relacionados con el trabajo compartido
con repositorios remotos.

- Subir una rama al repositorio remoto:

    ```
    $ git checkout -b nueva-rama
    $ git push -u origin nueva-rama
    ```

- Descargar una rama del repositorio remoto:

    ```
    $ git fetch 
    $ git checkout -b nueva-rama origin/nueva-rama
    ```

    El comando `git fetch` se descarga todos los cambios pero no los
    mezcla con las ramas locales. Los deja en ramas cacheadas a las
    que les da el nombre del servidor y la rama
    (`origin/nueva-rama`). 

    En el caso del comando anterior, una vez cacheada la rama
    `origin/nueva-rama` se crea la `nueva-rama` local con todos sus
    commits.

- Actualizar una rama con cambios que otros compañeros han subido al
  repositorio remoto:

    ```
    $ git checkout nueva-rama
    $ git pull
    ```

    El comando `git pull` es equivalente a un `git fetch` seguido de
    un `git merge`. Algunos recomiendan no usar `git pull`, sino hacer
    siempre el merge manual. Por ejemplo:

    ```
    $ git checkout nueva-rama
    $ git fetch
    $ git merge origin/nueva-rama
    ```

- Subir cambios de la rama actual:

    ```
    (estando en la rama que queremos subir)
    $ git push
    ```

    El comando `git push` funcionará correctamente sin más parámetros
    si previamente hemos subido la rama con un `git push -u`.

- Comprobar el estado de las ramas locales y remotas:

    ```
    $ git branch -vv
    ```

    Este comando no accede directamente al servidor, sino que muestra
    la información de la última vez que se accedió a él. Si queremos
    la información actualizada podemos hacer un `git fetch --all`
    antes:

    ```
    $ git fetch --all
    $ git branch -vv
    ```

    Es importante recordar que `git fetch` (a diferencia de `git
    pull`) no modifica los repositorios locales, sino que baja las
    ramas remotas cachés locales.

- Información de los repositorios remotos:

    ```
    $ git remote show origin
    ```

    Proporciona información del repositorio remoto, todas sus ramas,
    del local y de la conexión entre ambos.

    ```
    git remote -v update
    ```

    Proporciona información del estado de las ramas remotas y locales
    (si están actualizadas o hay cambios en algunas no bajadas o
    subidas).

- Borrado de ramas remotas:

    ```
    $ git push origin --delete nueva-rama
    ```

### 4.2. Pull request en GitHub

Para crear un pull request se debe pulsar el botón `New pull request`
en la pantalla de ramas de GitHub:

<img src="imagenes/pull-request1.png" width="800px">

Se crea un nuevo pull request:

<img src="imagenes/pull-request2.png" width="800px">

Hay que hacer notar que una vez creado el pull request en GitHub se
puede seguir subiendo cambios a la rama que se quiere mezclar. El pull
request se actualizará con los cambios que se suban a la rama. Pasa
igual con los cambios subidos a master.

Los miembros del equipo revisan el código en el pull request (consultar
documentación en GitHub:
[Reviewing proposed changes in a pull request](https://help.github.com/articles/reviewing-proposed-changes-in-a-pull-request/)). Al
final, todos los miembros del equipo deben dar el OK, añadiendo una reacción.

El responsable del ticket mezclará el pull request con master desde
GitHub. Justo antes de mezclar el pull request, cuando ya se ha tomado
la decisión de hacerlo y nadie tiene que subir más cambios, **hará un
_rebase_ con master** para asegurarse que el pull request se
introduce en cabeza de master:


```
$ git checkout master
$ git pull
$ git checkout nueva-rama
$ git rebase master
# lanzamos los tests para comprobar que todo funciona OK
# y subimos la rama a GitHub (tenemos que usar --force por el rebase)
$ git push --force
```

## 5. Representación de _features_ en Trello

Una _feature_ (funcionalidad) va a agrupar varios tickets. Vamos a
guardar las _features_ en el mismo tablero, pero usando **etiquetas**
para poder visualizarlas y filtrarlas fácilmente. También utilizaremos
una nomenclatura especial para numerarlas.

Cada funcionalidad en desarrollo tendrá una etiqueta específica, que
pondremos en todos los tickets de esa funcionalidad. De esta forma
podemos filtrar por esa etiqueta y observar rápidamente todos los
tickets asociados.

Todas las funcionalidades (en cualquier estado: en espera, desarrollo
o terminadas) tendrán una etiqueta especial denominada `Feature`
(color amarillo). Sólo se debe asignar esa etiqueta a las
funcionalidades, no a los tickets asociadas. De esta forma podremos
filtrar por esa etiqueta y observar sólo las funcionalidades que hay
en el tablero.

En cuanto a la **nomenclatura**, precederemos el nombre de la
funcionalidad con el prefijo "FT-1", "FT-2", ... (de _FeaTure_). Y los
tickets asociados a cada funcionalidad los numeraremos con dos
números: primero el correspondiente a la funcionalidad y después un
número correlativo correspondiente al número de ticket. Por ejemplo:
"TIC-1.1", "TIC-1.2".

Por ejemplo, podríamos definir la funcionalidad "FT-1 Gestión
básica de tareas" con la descripción:

```
FT-1 Gestión básica de tareas:
Un usuario deberá poder listar, añadir, borrar y modificar tareas,
para poder visualizar las tareas que está realizando
```

Y esta funcionalidad podría tener los siguientes tickets: "TIC-1.1
Entidad tarea", "TIC-1.2 Añadir tarea por usuario", "TIC-1.3 Listar
tareas de un usuario", ...

Las siguientes imágenes muestran un ejemplo.

Suponemos que se ha definido una funcionalidad llamada "FT-1 CRUD
Tareas" y que hemos creado 4 tickets: 

- "TIC-1.1 Listar tareas de usuario" (terminado)
- "TIC-1.2 Añadir tarea a usuario" (en pull request)
- "TIC-1.3 Borrar tarea de un usuario" (en pull request)
- "TIC-1.4 Editar tarea de usuario" (en marcha). 

El tablero se vería de la siguiente forma:

<img src="imagenes/tablero-features.png" width="800px">

Las etiquetas creadas son las siguientes:

<img src="imagenes/etiquetas.png" width="250px">

Las etiquetas son muy útiles para filtrar las tarjetas. Por ejemplo,
podemos definir un filtro para mostrar únicamente la etiqueta "CRUD
Tareas" (y sus tickets):

<img src="imagenes/filtro-feature-1.png" width="800px">

El filtro es el siguiente:

<img src="imagenes/filtro-feature.png" width="250px">

También podríamos ver sólo las tarjetas de tipo _Feature_:

<img src="imagenes/filtro-features.png" width="700px">

## 6. Desarrollo de 2 features en equipo

Utilizando el flujo de trabajo visto, tenéis que implementar y
documentar dos funcionalidades nuevas.

### 6.1. Feature 1: Login, registro y tamaño estimado de tareas

Tamaño: pequeño.

**Funcionalidad**: Un usuario podrá utilizar la aplicación para
logearse, gestionar sus tareas y sus datos y salir. Se puede asignar
un tamaño estimado a cada tarea. El tamaño puede tener uno de los
siguientes valores: pequeño, mediano o grande.

Las pantallas deben estar organizadas de forma que la navegación por
las distintas opciones sea sencilla y lógica. No gestionaremos la
seguridad (proteger las URLs para que sólo un usuario puede acceder a
sus datos), eso lo dejamos para el futuro.

### 6.2. Feature 2: Proyectos 

Tamaño: mediano.

**Funcionalidad**: Un usuario podrá agrupar las tareas por proyectos
para organizarlas mejor. En principio, un proyecto tendrá únicamente
un nombre.

El usuario podrá crear, cambiar el nombre y borrar proyectos. Deberá
poder asignar tareas a proyectos y presentar de forma organizada las
tareas, ordenadas por proyectos.


## 7. Entrega y evaluación

- La práctica tiene una duración de 3 semanas y debe estar terminada
  el **martes 15 de noviembre**.
- La calificación de la práctica tiene un peso de un 5% en la nota
  final de la asignatura.
- Durante el desarrollo se debe añadir el código en el repositorio en
  GitHub `mads-todolist` compartido con el profesor, y los tickets en
  el tablero Trello compartido con el profesor.
- En la fecha de la entrega se debe subir a Moodle un ZIP que contenga
  todo el proyecto y dejar la URL del repositorio en GitHub

Para la evaluación se tendrá en cuenta:

- Desarrollo contínuo (commits realizados a lo largo de las 3 semanas)
- Buen desarrollo y descripción de los cambios (commits bien
  documentados, ordenados, ramas de características visibles en la
  historia de commits)
- Tablero Trello bien ordenado
- Correcto desarrollo de las funcionalidades de la práctica
- Cuidado en el aspecto de la aplicación, la terminación, control de
  errores
- Características adicionales desarrolladas
- Para cada funcionalidad se debe realizar un documento técnico y un
  pequeño manual de usuario. Escribir una página `.md` para cada
  funcionalidad nueva.
