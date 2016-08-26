# Práctica 1: Primera aplicación Play Framework (Java)

## Objetivos

El objetivo principal de esta primera práctica es implementar una aplicación web inicial con [_Play Framework_](https://playframework.com) en Java. Servirá para tomar un primer contacto con esta tecnología, para que podamos centrarnos en las próximas prácticas en aspectos más relacionados con las metodologías de desarrollo.

Comenzaremos también a trabajar con repositorios de código, usuando [Git](https://git-scm.com) como sistema de control de versiones y [Bitbucket](https://bitbucket.org) como repositorio remoto. Desarrollaremos la aplicación de forma iterativa, utilizando una versión simplificada del flujo de trabajo basado en Git denominado [_Gitflow_](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow).

## Características de la aplicación

### Problema a resolver

Veamos una descripción de muy alto nivel de la aplicación a desarrollar, describiendo un supuesto práctico que plantea el problema que queremos resolver. 

**Gestión de tareas en una _startup_**

>Una _startup_ recién creada ha crecido rápidamente y quiere organizar mejor el trabajo de sus empleados. La empresa tiene un sistema de organización ágil, con equipos multifuncionales constituidos alrededor de proyectos en desarrollo, en los que participan desarrolladores de software, diseñadores, _product managers_, responsables de marketing, etc.

>La empresa quiere ofrecer a sus trabajadores una aplicación web con la que puedan gestionar las tareas pendientes del trabajo, que permita realizar acciones como:

>- añadir tareas a realizar
>- marcar tareas como terminadas
>- ...

>La empresa pretende que con el uso de la aplicación se mejore la organización y la comunicación entre todos los miembros de los equipos.

Deberemos diseñar y desarrollar una aplicación web que mejore el problema anterior. Iremos desarrollando las características concretas de la aplicación a lo largo de las prácticas. La aplicación tiene como nombre clave **ToDoList**.

### Características a implementar en la práctica

En esta primera práctica debemos implementar un [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) de usuarios, que nos permita listar, añadir, modificar y eliminar usuarios del sistema.

- La página central de esta funcionalidad será un listado de todos los usuarios existentes.
- En alguna zona de la práctica habrá un enlace a una página para añadir un usuario.
- En la lista de usuarios, cada usuario tendrá un enlace a las posibles acciones a realizar sobre él: obtener detalles, editar y borrar

Se debe utilizar Bootstrap o algún otro framework CSS para que las páginas tengan un aspecto atractivo.

### Pantallas de la aplicación

Tienes libertad en el diseño de las pantallas de la aplicación. Las pantallas que mostramos a continuación son sólo un ejemplo.

<img src="imagenes/usuarios.png" width="500px">

<img src="imagenes/nuevo-usuario.png" width="500px">

<img src="imagenes/editar-usuario.png" width="500px">

<img src="imagenes/detalle-usuario.png" width="500px">


## Lenguajes, APIs y frameworks de programación

Para desarrollar la aplicación utilizaremos _Play Framework_ y un conjunto de tecnologías adicionales. Repasamos aquí las características principales e incluimos algunos enlaces para su estudio.

### Aplicación web basada en HTTP

Vamos a utilizar un enfoque basado en servicios REST para definir las rutas de las peticiones. Por ejemplo, para obtener los detalles del usuario con identificador `12` utilizaremos la petición:

```
GET /usuarios/12
```

Pero no implementaremos un servicio REST propiamente dicho, porque queremos desarrollar una aplicación que podamos demostrar y enseñar a los usuarios finales. Por ejemplo, las peticiones no van a devolver objetos JSON, sino páginas HTML que constituyen la interfaz de usuario. Tampoco vamos a ser estrictos en las peticiones. Por ejemplo, en lugar de hacer una petición PUT para modificar un recurso vamos a reutilizar la misma petición POST también van a utilizarse para modificar recursos (no sólo para añadir recursos nuevos).

Eso sí, el desarrollo se va a basar totalmente en las tecnologías HTTP y HTML. Es conveniente repasarlas para tener claros los conceptos más importantes, tanto de HTTP (tipo de petición, códigos de respuesta, elementos de una petición, etc.) como de HTML (formularios, CSS, algo de JavaScript, etc.).

Un libro imprescindible sobre HTTP que debe estar en la biblioteca de cualquier informático es el de O'Reilly: [HTTP - The Definitive Guide](http://shop.oreilly.com/product/9781565925090.do).

### Play Framework

[Play Framework](https://playframework.com) es un framework de desarrollo rápido de aplicaciones web disponible en los lenguajes Java y Scala. Vamos a utilizar la versión Java. El framework proporciona un soporte de ejecución que tiene como base el servidor [Netty](http://netty.io). Con este soporte es posible diseñar y poner en marcha distintos tipos de aplicaciones: servicios HTTP, servicios HTTP asíncronos basados en websockets, aplicaciones asíncronas basadas en eventos, etc. El modelo de ejecución es similar al modelo de programación reactiva asíncrona que ha popularizado JavaScript y el servidor **Node.js**. Nosotros vamos a implementar una aplicación tradicional que implementa un servicio HTTP.

Todas las prácticas de la asignatura las vamos a implementar en la última versión (2.4) [Play Framework en Java](https://www.playframework.com/documentation/2.4.x/JavaHome). Para entender el funcionamiento de esta primera práctica es necesario consultar la siguiente documentación del framework:

Sobre la instalación y el funcionamiento de Play:

- [Installing Play](https://playframework.com/documentation/2.4.x/Installing)
- [Creating a new application](https://playframework.com/documentation/2.4.x/NewApplication)
- [Using the Play console](https://playframework.com/documentation/2.4.x/PlayConsole)
- [Anatomy of a Play application](https://playframework.com/documentation/2.4.x/Anatomy)

Sobre peticiones y respuestas HTTP:

- [Actions, Controllers and Results](https://www.playframework.com/documentation/2.4.x/JavaActions)
- [HTTP routing](https://www.playframework.com/documentation/2.4.x/JavaRouting)
- [Flash scope](https://www.playframework.com/documentation/2.4.x/JavaSessionFlash#Flash-scope)

Sobre plantillas:

- [The template engine](https://www.playframework.com/documentation/2.4.x/JavaTemplates)
- [Common template use cases](https://www.playframework.com/documentation/2.4.x/JavaTemplateUseCases)

Sobre envío de datos de formularios:

- [Handling form submission](https://www.playframework.com/documentation/2.4.x/JavaForms)
- [Rendering an <input> element](https://www.playframework.com/documentation/2.4.x/JavaFormHelpers#Rendering-an-<input>-element)

Sobre el acceso a datos mediante JPA

- [Integrating with JPA](https://www.playframework.com/documentation/2.4.x/JavaJPA)

### Java Persistence API (JPA)

JPA es el API que utilizaremos para acceder a bases de datos y gestionar entidades persistentes usando un modelo ORM (_Object Relational Mapping_). Está integrado en Play, no es necesario instalar ninguna librería adicional.

JPA también es conocido por el nombre de una de sus implementaciones más populares, Hibernate. Es una tecnología muy usada y madura en el mundo Java. Permite gestionar la persistencia directamente con el modelo de objetos de la aplicación (se denominan _entidades_), independizándola del modelo relacional basado en tablas y registros.

La implementación de JPA ObjectDB tiene unos tutoriales muy completos y accesibles:

- [JPA Quick tour](http://www.objectdb.com/java/jpa/getting/started)
- [Entity classes](http://www.objectdb.com/java/jpa/entity)
- [Using JPA](http://www.objectdb.com/java/jpa/persistence)
- [JPA Queries](http://www.objectdb.com/java/jpa/query)

No es necesario estudiar todos los tutoriales. El objetivo de las prácticas no es aprender JPA, sino desarrollar de forma ágil una aplicación. Vamos a utilizar lo más básico de JPA y en la mayoría de las ocasiones se va a proporcionar el código necesario. Además, en caso de duda, siempre podrás realizar preguntas sobre cómo implementar una determinada funcionalidad en el foro de Moodle.

### Bootstrap

Para hacer más atractivo el diseño de las páginas HTML vamos a usuar el framework CSS http://getbootstrap.com/getting-started/[Bootstrap]. Es conveniente tener a mano su documentación, en concreto la lista de componentes:

- [Bootstrap components](http://getbootstrap.com/components/)

## Metodología de desarrollo

Es importante realizar la aplicación siguiendo las indicaciones de este apartado. Se deberá utilizar git como sistema de control de versiones, dejando en la rama principal (_master_) la última versión funcional de la aplicación y creando una rama para implementar cada nueva característica.

### Git

Git es el sistema de control de versiones más utilizado en la actualidad. Es muy flexible, distribuido, adaptable a múltiples flujos de trabajo e ideal para una metodología de desarrollo en equipo. Suponemos que ya tienes cierta experiencia con su uso. Puedes usar los siguientes enlaces para repasar su funcionamiento.

- [Atlassian Git Tutorials](https://www.atlassian.com/git/tutorials/): Tutoriales muy orientados al uso de Git con gran cantidad de ejemplos. Es recomendable repasar los tutoriales básicos (_Getting Started_) y los tutoriales _Syncing_ y _Using Branches_ en el apartado _Collaborating_.
- [Libro de Scott Chacon](https://git-scm.com/book/en/v2): Completo manual con todos los detalles de todos los comandos de Git.

Cuando utilicemos git es muy importante realizar unos mensajes de commit claros. Un mensaje de commit es la forma de comunicar a los compañeros del equipo (en este caso con _el profesor_) qué cambios se han introducido en la aplicación y ponerlos en contexto (explicar por qué se han hecho, dar algún detalle de implementación, etc.). El post [How to Write a Git Commit Message](http://chris.beams.io/posts/git-commit/) explica muy bien esto.

Los commits deben definir cambios coherentes de la aplicación que se desarrolla. No deben tener errores de compilación. El mensaje del commit deberá tener el siguiente formato:

- Descripción de menos de 50 caracteres
- Línea en blanco
- Cuerpo del commit con una descripción más detallada del mismo. Las líneas deben tener como máximo 72 caracteres.


Un ejemplo:

```
$ git commit -m "Bootstrap incluido en plantilla principal

Se ha añadido bootstrap en el directorio de assets públicos
y se ha incluido en la plantilla principal de la aplicación"
```

### Desarrollo incremental

Para realizar un desarrollo incremental de la aplicación, iremos definiendo claramente las características que debe cumplir y las desarrollaremos una a una de forma secuencial. No pasaremos a desarrollar la siguiente característica hasta haber comprobado que funciona la anterior. En el siguiente apartado se presenta una lista con las características que debe tener la práctica. El orden final que utilices en tu desarrollo puede ser distinto al planteado, pero la aplicación final debe contener todas las funcionalidades.

Utilizaremos git para remarcar el caráter incremental. Cuando empecemos a desarrollar una característica crearemos una rama con el nombre del identificador de la característica y desarrollaremos la característica en los commits de esa rama. Cuando hayamos terminado el desarrollo mezclaremos esa rama en la rama principal. De esta forma, en cualquier momento del desarrollo tendremos en la rama _master_ una versión operativa de la aplicación que podríamos enseñar en una demostración. Esta rama principal incluirá todas las características terminadas hasta el momento.

### Bitbucket

[Bitbucket](https://bitbucket.org) permite trabajar con repositorios git remotos y compartirlos con un máximo de 5 personas. Vamos a utilizar el servicio para crear allí un repositorio remoto copia de nuestro repositorio local, compartido en principio con el profesor (su login es `domingogallardo`). Más adelante crearemos equipos y se compartirá el repositorio del equipo entre todos sus miembros.

## Realización de la práctica

Para realizar la práctica necesitas lo siguiente:

- [Java SE 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
- [Play Framework 2.4.2](https://playframework.com/download)
- Editor o entorno de desarrollo. Recomendamos [Atom](https://atom.io) o [IntelliJ Community Edition](https://www.jetbrains.com/idea/download/).

### Máquina virtual

Hemos preparado una máquina virtual Virtual Box con LUbuntu 14.04 64 bits y el software necesario ya instado. El usuario de la MV es `mads` y su login también es `mads`.

Puedes descargar la imagen del disco duro de la MV (el fichero .vdi) de las siguientes URLs. Está comprimido en formato multi-part:

- [MADS_2015-16.vdi.z01](http://www.dccia.ua.es/dccia/inf/asignaturas/MADS/software/MADS_2015-16.vdi.z01) (1.024 MB)
- [MADS_2015-16.vdi.z02](http://www.dccia.ua.es/dccia/inf/asignaturas/MADS/software/MADS_2015-16.vdi.z02) (1.023 MB)
- [MADS_2015-16.vdi.z03](http://www.dccia.ua.es/dccia/inf/asignaturas/MADS/software/MADS_2015-16.vdi.z03) (1.024 MB)
- [MADS_2015-16.vdi.z04](http://www.dccia.ua.es/dccia/inf/asignaturas/MADS/software/MADS_2015-16.vdi.z04) (1.024 MB)
- [MADS_2015-16.vdi.zip](http://www.dccia.ua.es/dccia/inf/asignaturas/MADS/software/MADS_2015-16.vdi.zip) (123 MB)

Para descomprimir en **Ubuntu Linux** un fichero multi-part:

```
$ cat fich.z01 fich.z02 fich.zip > combinado.zip
$ unzip -FF combinado.zip
```

Para trabajar en el laboratorio de prácticas es recomendable que utilices un disco o lápiz USB externo 3.0 en el que guardes la imagen VDI y que utilices este fichero como imagen de una máquina virtual que deberás crear cada vez usando la aplicación VirtualBox instalada en Linux. Cuando termines la sesión deberás **apagar la MV** (¡¡no la dejes en suspensión!!) y desmontar el disco duro externo. Podrás utilizar el disco externo y la imagen para continuar trabajando en casa.

Para evitar que puedas perder el código desarrollado (y para poder hacer un seguimiento contínuo del trabajo realizado) deberás subir a bitbucket todos los cambios conformes los vayas programando.


### Guía con los pasos iniciales

Detallamos a continuación los primeros pasos de la práctica. Si los sigues con atención comprobarás cómo construir una primera aplicación web con Play Framework. Después deberás continuar realizando la práctica tu solo/a.

Cualquier duda que te surja sobre cualquiera de las tecnologías intenta resolverla primero de la forma habitual (consultando _google_ y _stack overflow_). Si la pregunta es sobre Play, comprueba bien que las respuestas se refieren a la versión que estamos utilizando (la 2.4), porque el framework ha cambiado con mucha frecuencia.

En el caso en que no encuentres la solución, no dudes en consultar en el foro de la asignatura en Moodle. Te podrá contestar cualquier compañero o el profesor de la asignatura, así podemos aprender estas tecnologías entre todos.

Veamos los pasos a seguir:

#### Creación de la aplicación inicial `play-todolist`

- En la máquina virtual ya está instalado Play Framework y el comando `activator` está en el PATH. Para comprobar dónde se encuentra ubicado, puedes lanzar la siguiente instrucción:

    ```
    $ which activator
    /home/mads/PlayFramework/activator-dist-1.3.5/activator
    ```

- Creamos la aplicación base en algún directorio, por ejemplo el escritorio:

    ```
    $ cd Escritorio
    $ activator new
    ```

    Nos mostrará una lista de plantillas, escribimos `play-java` y le damos al nuevo proyecto el nombre `mads-todolist`.

- Se habrá creado el directorio con ese nombre. Nos movemos a él y lanzamos play:

    ```
    $ cd mads-todolist
    $ activator run
    ```

    La primera vez que lanzamos el comando, tardará bastante en arrancar la aplicación porque play comprueba todas las dependencias y se descarga las librerías que no están disponibles en local. Después pondrá el servidor de play a escuchar en el puerto 9000

- Abre el navegador e abre la URL [http:localhost:9000](http:localhost:9000). Verás que play compila los programas fuentes y que responde con una página de saludo:

    <img src="imagenes/pantalla1-play.png" width="500px">

- Estudia cómo funciona la aplicación. Lanza `Atom` y carga el directorio `mads-todolist`. Estudia qué función cumple cada directorio y cada fichero. Comprueba cómo se muestra la frase `"Your new application is ready"` en la acción `index()` en el fichero `app/controllers/Application.java`. Prueba a introducir un error (por ejemplo, quitar las dobles comillas de alguna cadena) y vuelve a recargar la página. Verás que automáticamente Play recompila la aplicación y muestra el error en el propio navegador:


    <img src="imagenes/error-play.png" width="500px">

- Arregla el error y cambia el mensaje para comprobar que esta función es la define el mensaje que aparece en la plantilla.

Una vez que tenemos una primera versión en funcionamiento es un buen momento de crear el repositorio local con git:

- Cerramos la aplicación (`ctrl-d`)
- Estando dentro del directorio recién creado `Escritorio/mads-todolist/` ejecutamos el comando para inicializar git y llenar el repositorio local con la primera version:

    ```
    $ git init
    Initialized empty Git repository in /home/mads/Escritorio/mads-todolist/.git/
    $ git add .
    $ git commit -m "Versión inicial"

    *** Please tell me who you are.

    Run

    git config --global user.email "you@example.com"
    git config --global user.name "Your Name"

    to set your account's default identity.
    Omit --global to set the identity only in this repository.

    fatal: unable to auto-detect email address (got 'mads@mads.(none)')
    ```

    Introducimos nuestro correo electrónico (el que usaremos para autentificarnos en Bitbucket) y nuestro nombre verdadero:


    ```
    $ git config --global user.email "domingo.gallardo@ua.es"
    $ git config --global user.name "Domingo Gallardo"
    $ git commit -m "Versión inicial"
    ```

#### Bitbucket

- Crea una cuenta en Bitbucket
- Crea el repositorio `mads-todolist`
- Añade a `domingogallardo` como usuario con permiso de lectura (con la opción Settings del menú de la izquierda y Access Management)
- Conecta la rama principal del repositorio local con bibucket:

    ```
    $ git remote add origin https://<usuario>@bitbucket.org/<usuario>/mads-todolist.git  
    $ git push -u origin --all
    $ git push -u origin --tags
    ```

#### Primera funcionalidad

Vamos a crear una primera funcionalidad muy sencilla. Nos va a servir para comprobar el flujo de trabajo con git: crearemos una rama en la que desarrollaremos la característica y después, cuando hayamos comprobado que funciona correctamente, mezclaremos esa rama con la rama principal.

##### Página de saludo

- Creamos una rama con el nombre `feature1`:

    ```
    $ git checkout -b feature1
    Switched to a new branch 'feature1'
    mads@mads:~/Escritorio/mads-todolist$ git branch
    * feature1
    master
    ```

- Añadimos una nueva ruta en el fichero `conf/routes`:

    **conf/routes**

    ```
    ...
    GET     /saludo                     controllers.Application.saludo()
    ...
    ```

- Añadimos la acción `controllers.Application.saludo`:

    **app/controllers/Application.java**

    ```
    ...
    public class Application extends Controller {
        public Result saludo() {
        return ok(saludo.render());
    }
    ...
    ```

- Añadimos la vista `saludo.scala.html`:

    **app/views/saludo.scala.html**

    ```
    @main("Hola") {
        <h1>Hola mundo!</h1>
    }
    ```

- Comprobamos que se devuelve el saludo al acceder a la página _[http://localhost/saludo](http://localhost/saludo)

- Hacemos un commit con los cambios:


    ```
    $ git status
    $ git add app/views/saludo.scala.html
    $ git commit -m "Añadida página de saludo"
    ```

#### Página de saludo con nombre

Vamos ahora a añadir el parámetro `nombre` a la página de saludo.

- Modificamos la ruta:

    ```
    ...
    GET     /saludo                     controllers.Application.saludo(nombre: String)
    ...
    ```

- Modificamos la acción:

    ```
    public Result saludo(String nombre) {
        return ok(saludo.render(nombre));
    }
    ```

- Modificamos la plantilla:

    ```
    @(nombre: String)

    @main("Hola") {
        <h1>Hola <em>@nombre</em>!</h1>
    }
    ```

- Comprobamos que funciona correctamente realizando la petición [http://localhost:9000/saludo?nombre=Pepe](http://localhost:9000/saludo?nombre=Pepe)

- Por último, realizamos el commit y, como hemos terminado la característica, mezclamos la rama con la principal y borramos la rama

    ```
    $ git commit -a -m "Añadido parametro nombre a la pagia de saludo"
    $ git checkout master
    $ git merge --no-ff feature1 -m "Terminada la página de saludo"
    $ git branch -d feature1
    ```

- Subimos los cambios a Bitbucket:

    ```
    $ git push -all
    ```

- Si tuviéramos que subir los cambios sin haber mezclado la rama, deberíamos subir la rama al repositorio remoto:

    ```
    $ git push -u origin <rama>
    ```

    Y después, cuando borremos la rama en local después de hacer un merge, también hay que borrarla en el repositorio remoto:

    ```
    $ git push origin --delete <rama>
    ```

## _Backlog_ de la práctica

A continuación listamos el _backlog_ de características a implementar en esta práctica. Al igual que las pantallas, considera que es un ejemplo. Puedes modificarlo si lo consideras conveniente. El formato del _backlog_ está tomado del libro de Henrik Kniberg [Scrum and XP from the Trenches - 2nd Edition](http://www.infoq.com/minibooks/scrum-xp-from-the-trenches-2).

=== Características administración de usuarios

| ID  | Nombre | Demostración | Notas |
| --- | ------ | ------------ | ----- |
| feature1 | Página home con saludo | Abrir el navegador en la página raíz del servidor, realizar la petición [http://localhost:9000/saludo?nombre=Pepe](http://localhost:9000/saludo?nombre=Pepe) y comprobar que se devuelve un HTML con un saludo. | Sirve para comprobar que Play está funcionando. |
| feature2 | Crear usuario | Abrir la URL de creación de un usuario, introducir sus datos y comprobar que se han añadido a la base de datos. | En esta primera práctica vamos a trabajar con una base de datos en memoria. En lugar de comprobar que el usuario se ha añadido en la BD, sacaremos un mensaje de log si todo ha funcionado bien. |
| feature3 | Incluir Bootstrap | Abrir una página cualquiera de la aplicación y comprobar que el CSS incluye ese framework. | |
| feature4 | Listar usuarios | Abrir la URL de listado de usuarios y comprobar que aparecen todos los usuarios creados. | La página de listado de usuarios va a ser la página principal de la aplicación, desde la que se van a lanzar el resto de acciones. Incluir un enlace a la acción de crear usuario. |
| feature5 | Detalle de un usuario | Abrir la URL del usuario (o pinchar en la acción de _detalle_ asociada al usuario) y comprobar que aparecen correctamente todos sus datos. | El formato de la fecha de nacimiento debe ser mm-yy-aaa. |
| feature6 | Editar usuario | Pinchar en la acción de _editar_ asociada al usuario, realizar alguna modificación de alguno de sus datos y comprobar en el listado y en los detalles que la modificación se ha grabado. | El campo _login_ es obligatorio. Si no se introduce se debe volver al formulario y mostrar un error. |
| feature7 | Borrar usuario | Pinchar en la acción de borrado asociada al usuario y comprobar que el usuario desparece del listado. | Se debe enviar una petición DELETE a la URL del usuario. Utilizar un script JavaScript. |

### Características de logeo y registro de usuarios

Debes desarrollar una o dos características relacionadas con el logeo y el registro de usuarios. Puedes añadir los cambios que necesites en todos los componentes de la aplicación. Piensa cómo deberían ser las pantallas y qué se debería implementar.

## Detalles de diseño e implementación 

A continuación puedes encontrar ejemplos de código que te ayudarán a implementar la aplicación. 

**No copies y pegues todo el código de un fichero de golpe, sino sólo lo necesario para completar la característica que estés desarrollando.**


### Rutas de la aplicación

**conf/routes**

```
...

GET  /saludo              controllers.Application.saludo(nombre: String)
GET  /usuarios            controllers.Usuarios.listaUsuarios()
GET  /usuarios/nuevo      controllers.Usuarios.formularioNuevoUsuario()
POST /usuarios/nuevo      controllers.Usuarios.grabaNuevoUsuario()
POST /usuarios/modifica   controllers.Usuarios.grabaUsuarioModificado()
GET  /usuarios/:id        controllers.Usuarios.detalleUsuario(id: String)
GET  /usuarios/:id/editar controllers.Usuarios.editarUsuario(id: String)
DELETE /usuarios/:id      controllers.Usuarios.borraUsuario(id: String)

...
```

### Controller `Usuarios`

**app/controllers/Usuarios.java**

```
package controllers;

import java.util.List;

import play.*;
import play.mvc.*;
import views.html.*;
import static play.libs.Json.*;
import play.data.Form;
import play.db.jpa.*;

import models.*;

public class Usuarios extends Controller {

    ...
    
    @Transactional(readOnly = true)
    // Devuelve una página con la lista de usuarios
    public Result listaUsuarios() {
        // Obtenemos el mensaje flash guardado en la petición
        // por el controller grabaUsuario
        String mensaje = flash("grabaUsuario");
        List<Usuario> usuarios = UsuarioService.findAllUsuarios();
        return ok(listaUsuarios.render(usuarios, mensaje));
    }

    // Devuelve un formulario para crear un nuevo usuario
    public Result formularioNuevoUsuario() {
        return ok(formCreacionUsuario.render(Form.form(Usuario.class),""));
    }

    @Transactional
    // Añade un nuevo usuario en la BD y devuelve código HTTP
    // de redirección a la página de listado de usuarios
    public Result grabaNuevoUsuario() {
        Form<Usuario> usuarioForm = Form.form(Usuario.class).bindFromRequest();
        if (usuarioForm.hasErrors()) {

            return badRequest(formCreacionUsuario.render(usuarioForm, "Hay errores en el formulario"));
        }
        Usuario usuario = usuarioForm.get();
        usuario = UsuarioService.grabaUsuario(usuario);
        flash("grabaUsuario", "El usuario se ha grabado correctamente");
        return redirect(controllers.routes.Usuarios.listaUsuarios());
    }

    ...
}
```

### Modelos

**/app/models/UsuarioService.java**

```
package models;

import play.*;
import play.mvc.*;
import play.db.jpa.*;
import java.util.List;
import java.util.Date;

public class UsuarioService {
    public static Usuario grabaUsuario(Usuario usuario) {
        return UsuarioDAO.create(usuario);
    }

    public static Usuario modificaUsuario(Usuario usuario) {
        ...
    }

    public static Usuario findUsuario(String id) {
        ...
    }

    public static boolean deleteUsuario(String id) {
        ...
    }

    public static List<Usuario> findAllUsuarios() {
        ...
    }
}
```

**app/models/UsuarioDAO.java**

```
package models;

import play.*;
import play.mvc.*;
import play.db.jpa.*;
import java.util.List;
import java.util.Date;

public class UsuarioDAO {
    public static Usuario create (Usuario usuario) {
        usuario.nulificaAtributos();
        JPA.em().persist(usuario);
        // Hacemos un flush y un refresh para asegurarnos de que se realiza
        // la creación en la BD y se devuelve el id inicializado
        JPA.em().flush();
        JPA.em().refresh(usuario);
        Logger.debug(usuario.toString());
        return usuario;
    }

    public static Usuario find(String idUsuario) {
        return JPA.em().find(Usuario.class, idUsuario);
    }

    public static Usuario update(Usuario usuario) {
        return JPA.em().merge(usuario);
    }

    public static void delete(String idUsuario) {
        Usuario usuario = JPA.em().getReference(Usuario.class, idUsuario);
        JPA.em().remove(usuario);
    }

    public static List<Usuario> findAll() {
        return (List<Usuario>) JPA.em().createQuery("select u from Usuario u ORDER BY id").getResultList();
    }
}
```

**app/models/Usuario.java**

```
package models;

import java.util.Date;
import javax.persistence.*;
import play.data.validation.Constraints;
import play.data.format.*;

import java.text.DateFormat;
import java.text.SimpleDateFormat;

@Entity
public class Usuario {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    public String id;
    @Constraints.Required
    public String login;
    public String nombre;
    public String apellidos;
    public String eMail;
    @Formats.DateTime(pattern="dd-MM-yyyy")
    public Date fechaNacimiento;

    // Sustituye por null todas las cadenas vacías que pueda tener
    // un usuario en sus atributos
    public void nulificaAtributos() {
        if (nombre != null && nombre.isEmpty()) nombre = null;
        if (apellidos != null && apellidos.isEmpty()) apellidos = null;
        if (eMail != null && eMail.isEmpty()) eMail = null;
    }

    public String toString() {
        String fechaStr = null;
        if (fechaNacimiento != null) {
            SimpleDateFormat formateador = new SimpleDateFormat("dd-MM-yyyy");
            fechaStr = formateador.format(fechaNacimiento);
        }
        return String.format("Usuario id: %s login: %s nombre: %s " +
                      "apellidos: %s eMail: %s fechaNacimiento: %s",
                      id, login, nombre, apellidos, eMail, fechaStr);
    }
}
```

### Vistas

**app/views/formModificacionUsuario.scala.html**

```
@(usuarioForm: Form[Usuario], mensaje: String)
@main("Modificar usuario") {
    @if(mensaje != "") {
        <div class="alert alert-danger">
            @mensaje
        </div>
    }
    <h1>Modificar usuario</h1>
    @helper.form(action = routes.Usuarios.grabaUsuarioModificado()) {
        <fieldset>
            <legend>Usuario @usuarioForm("id").value</legend>
            <input type="hidden" name="id" value='@usuarioForm("id").value' >
            @helper.inputText(usuarioForm("login"), '_label -> "Login")
            @helper.inputText(usuarioForm("nombre"), '_label -> "Nombre")
            @helper.inputText(usuarioForm("apellidos"), '_label -> "Apellidos")
            @helper.inputText(usuarioForm("eMail"), '_label -> "Correo electrónico")
            @helper.inputText(usuarioForm("fechaNacimiento"), '_label -> "Fecha nacimiento (dd-mm-aaaa)")
        </fieldset>
        <p>
        <input type="submit" class="btn btn-primary" value="Guardar">
        <a class="btn btn-primary" href="@routes.Usuarios.listaUsuarios()">Cancelar</a></p>
    }
}
```

**app/views/listaUsuarios.scala.html**

```
@(usuarios: List[Usuario], mensaje: String)
@scripts = {
    <script type="text/javascript">
        function del(urlBorrar) {
            $.ajax({
                url: urlBorrar,
                type: 'DELETE',
                success: function(results) {
                    //refresh the page
                    location.reload();
                }
            });
        }
    </script>
}
@main("Listado de usuarios", scripts) {

    <h2> Listado de usuarios </h2>

    ...

    @for(usuario <- usuarios) {
        ...
        <a onmouseover="" style="cursor: pointer;" onclick="del('@routes.Usuarios.borraUsuario(usuario.id)')">
        <span class="glyphicon glyphicon-trash" aria-hidden="true"></span></a>
        ...
    }
    
    ...
    
    @if(mensaje != null) {
        <div class="alert alert-success">
            @mensaje
        </div>
    }
}
```

**app/views/main.scala.html**

```
@(title: String, scripts: Html = Html(""))(content: Html)

<!DOCTYPE html>

<html lang="en">
    <head>
        <title>@title</title>
        <link href="@routes.Assets.versioned("bootstrap/css/bootstrap.min.css")" rel="stylesheet" media="screen">
        <link rel="stylesheet" media="screen" href="@routes.Assets.versioned("stylesheets/main.css")">
        <link rel="shortcut icon" type="image/png" href="@routes.Assets.versioned("images/favicon.png")">
    </head>
    <body>
        <div class="container">
        @content
    </div>
    <script src="@routes.Assets.versioned("javascripts/jquery-1.11.3.min.js")" type="text/javascript"></script>
    <script src="@routes.Assets.versioned("bootstrap/js/bootstrap.min.js")" type="text/javascript"></script>
    @scripts
    </body>
</html>
```

### Configuración

Configuración de la aplicación Play:

**conf/application.conf**

```
# This is the main configuration file for the application.
# ~~~~~

# Secret key
# ~~~~~
# The secret key is used to secure cryptographics functions.
#
# This must be changed for production, but we recommend not changing it in this file.
#
# See http://www.playframework.com/documentation/latest/ApplicationSecret for more details.
play.crypto.secret = "changeme"

# The application languages
# ~~~~~
play.i18n.langs = [ "en" ]

# Router
# ~~~~~
# Define the Router object to use for this application.
# This router will be looked up first when the application is starting up,
# so make sure this is the entry point.
# Furthermore, it's assumed your route file is named properly.
# So for an application router like `my.application.Router`,
# you may need to define a router file `conf/my.application.routes`.
# Default to Routes in the root package (and conf/routes)
# play.http.router = my.application.Routes

# Database configuration
# ~~~~~
# You can declare as many datasources as you want.
# By convention, the default datasource is named `default`
#
db.default.driver=org.h2.Driver
db.default.url="jdbc:h2:mem:play"
# db.default.username=sa
# db.default.password=""

# You can expose this datasource via JNDI if needed (Useful for JPA)
db.default.jndiName=DefaultDS
jpa.default=defaultPersistenceUnit

# Evolutions
# ~~~~~
# You can disable evolutions if needed
# play.evolutions.enabled=false

# You can disable evolutions for a specific datasource if necessary
# play.evolutions.db.default.enabled=false
```

Configuración de dependencias:

**build.sbt**

```
name := """play-todolist"""

version := "1.0-SNAPSHOT"

lazy val root = (project in file(".")).enablePlugins(PlayJava)

scalaVersion := "2.11.6"

libraryDependencies ++= Seq(
  javaJpa,
  "org.hibernate" % "hibernate-entitymanager" % "4.3.7.Final",
  cache,
  javaWs
)

// Play provides two styles of routers, one expects its actions to be injected, the
// other, legacy style, accesses its actions statically.
routesGenerator := InjectedRoutesGenerator
```

Configuración de JPA:

**conf/META-INF/persistence.xml**

```
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_1.xsd"
             version="2.1">

    <persistence-unit name="defaultPersistenceUnit" transaction-type="RESOURCE_LOCAL">
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <non-jta-data-source>DefaultDS</non-jta-data-source>
        <properties>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
            <property name="hibernate.hbm2ddl.auto" value="update"/>
        </properties>
    </persistence-unit>

</persistence>
```

## Entrega y evaluación

- La práctica tiene una duración de 3 semanas y debe estar terminada el martes 6 de octubre.
- Durante el desarrollo se debe compartir con el profesor:
  - Repositorio Bitbucket (usuario de Bitbucket: `domingogallardo`)
  - Google spreadsheet con el backlog (usuario de Google: `domingo.gallardo@gcloud.ua.es`)
- En la fecha de la entrega de debe subir a Moodle un ZIP que contenga el proyecto Play terminado (incluyendo el directorio .git) y un PDF con el backlog.

Para la evaluación se tendrá en cuenta:

- Desarrollo contínuo (commits realizados a lo largo de las 3 semanas)
- Buen desarrollo y descripción de los cambios (commits bien documentados, ordenados, ramas de características visibles en la historia de commits)
- Correcto desarrollo de las funcionalidades de la práctica
- Cuidado en el aspecto, la terminación, control de errores
- Características adicionales desarrolladas

## Apéndice: repaso de comandos de Git

### Comandos básicos

- Inicializar git en un directorio:

    ```
    $ cd /ruta/a/mi/directorio  
    $ git config --global user.name <nombre-usuario>  
    $ git config --global user.email <email>  
    $ git init  
    $ git add .  
    $ git commit -m "Versión inicial"
    ```

- Publicar por primera vez el repositorio local en el remoto (en Bitbucket):

    ```
    $ git remote add origin https://<usuario>@bitbucket.org/<usuario>/mads-todolist.git  
    $ git push -u origin --all
    $ git push -u origin --tags
    ```

- Comprobar el estado del repositorio local:

    ```
    $ git status
    ```

- Comprobar las diferencias entre los ficheros modificados y el último commit:

    ```
    $ git diff
    ```

- Añadir un fichero al [_stage_](http://programmers.stackexchange.com/questions/119782/what-does-stage-mean-in-git) (dejarlo listo para el próximo commit):

    ```
    $ git add <fichero o directorio>
    ```

- Hacer un commit de los ficheros en el _stage_:

    ```
    $ git commit -m "Mensaje"
    ```

- Eliminar un fichero del _stage_ (si lo hemos añadido, pero al final decidimos no añadirlo en el siguiente commit):

    ```
    $ git reset HEAD <fichero>
    ```

- Añadir hacer un commit de todos los últimos cambios:

    ```
    $ git commit -a -m "Mensaje"
    ```

- Publicar los cambios en el repositorio remoto:

    ```
    $ git push
    ```

- Consultar los mensajes de los commits (toda la historia de la rama actual). La opción `--oneline` muestra sólo la primera línea del mensaje:

    ```
    $ git log [--oneline]
    ```

### Ramas

- Crear una rama nueva:

    ```
    $ git checkout -b nueva-rama  
    M   hola.txt (si hay cambios en el espacio de trabajo se llevan a la nueva rama)  
    Switched to a new branch 'nueva-rama'
    ```

- Listar las ramas de un repositorio:

    ```
    $ git branch  
    master  
    * nueva-rama  
    $ git commit -a -m "Confirmamos los cambios en la nueva rama"
    ```

- Moverse a otra rama:

    ```
    $ git checkout master  
    Switched to branch 'master'
    ```

- Mostrar un fichero de una rama (o commit) dado:

    ```
    $ git show <commit o rama>:<nombre-fichero>
    ```

- Comparar dos ramas:

    ```
    $ git diff rama1 rama2
    ```

    El comando git diff rama1 rama2 devuelve las diferencias entre las ramas rama1 y rama2: las modificaciones que resultarían de mezclar la rama rama2 en la rama rama1.

- Mezclar la rama nueva-rama en la rama actual (añade a la rama actual los commits adicionales de la rama nueva-rama):

    ```
    $ git merge --no-ff nueva-rama -m "Mensaje de commit"
    ```

    La opción `--no-ff` no hace un fast forward y mantiene separados los commits de la rama en el log de commits. Es útil para revisar la historia del repositorio.

- _Rebase_ de una rama. Si la rama master ha avanzado después de separar una rama alternativa y queremos incorporar esos cambios en la rama alternativa podemos hacer un `git rebase`:

    ```
    $ git checkout master  
    # hacemos cambios  
    $ git commit -a -m "Cambios en master"  
    $ git checkout rama-feature  
    $ git rebase master  
    First, rewinding head to replay your work on top of it...  
    Applying: Corregido bug1  
    Applying: Corregido bug2
    ```

    El comando cambia la historia de la rama: primero la mueve al final de la rama master (rewind head) y a partir de ahí aplica los cambios propios de la rama.

- Log en forma de grafo:

    ```
    $ git log --graph --oneline 
    ```

- Si en la rama que se mezcla y en la actual hay cambios que afectan a las mismas líneas de un fichero, git detecta un conflicto y combina esas líneas conservando las dos versiones y añadiendo la información de la procedencia. Debemos resolver el conflicto: editarlos a mano y volver a hacer add y commit.

    ```
    $ git merge  
    CONFLICT (content): Merge conflict in hola.txt  
    Automatic merge failed; fix conflicts and then commit the result.  
    $ editar a mano el fichero con conflictos  
    $ git commit -a -m "Arreglado el conflicto en el merge"
    ```

    El comando `git status` después de un merge nos indica qué ficheros no se han mezclado y hay que editar manualmente.

- Borrar una rama:

    ```
    $ git branch -d nueva-rama  
    Deleted branch nueva-rama (was c241d7b)
    ```
    Sólo podemos borrar de la forma anterior ramas en las que no estamos y que se han mezclado con alguna otra. El comando anterior no permite borrar ramas activas que tienen commits sin mezclar con otras.

- Borrar una rama descartando sus commits:

    ```
    $ git branch -D rama
    ```

- Subir una rama al repositorio remoto:

    ```
    $ git push -u origin <rama>
    ```

- Subir todas las ramas y etiquetas:

    ```
    $ git push -u -all origin
    ```

    Al poner la opción -u hacemos tracking del repositorio remoto y las referencias quedan almacenadas en el fichero de configuración .git/config. A partir de ahora sólo es necesario hacer `git push` para subir los cambios en cualquiera de las ramas presentes.

- Borrar una rama en repositorio remoto:

    ```
    $ git push origin --delete <branchName>
    ```

=== Modificar la historia

- Modificar el mensaje del último commit. Se abrirá un editor en el que modificar el mensaje. También se puede escribir el mensaje a mano:

    ```
    $ git commit --amend [--m "Nuevo mensaje"]
    ```

- Deshacer el último commit (sólo la acción del commit, dejando los ficheros sin modificar):

    ```
    $ git reset --soft HEAD^
    ```

- Descartar el último merge y volver a la situación anterior al hacer el merge:

    ```
    $ git reset --merge ORIG_HEAD
    ```

- Movernos atrás a un commit pasado, mirar los ficheros, crear una nueva rama allí (o no) y volver al commit actual:

    ```
    $ git checkout v0.0  
    # Ahora estás en un detached HEAD  
    $ git branch  
    * (no branch)  
    master  
    nueva-rama  
    $ git checkout -b v0.0.1  
    Switched to a new branch 'v0.0.1'  
    $ git branch  
    master  
    nueva-rama  
    * v0.0.1  
    $ git checkout master
    ```
