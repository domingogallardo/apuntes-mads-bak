
Vamos a ver cómo lanzar y desarrollar una aplicación Play Framework en
Java. Presentaremos las principales características de este framework
de desarrollo de aplicaciones web mediante ejemplos concretos de
código.

### Descarga de la aplicación `play-proyecto-inicial` ###

La aplicación `play-proyecto-inicial` se encuentra en GitHub:
<https://github.com/domingogallardo/play-proyecto-inicial>. Es un
esqueleto de aplicación Play con ejemplos básicos del funcionamiento
del framework.

Puedes descargarla usando el comando `git clone`:

```text
$  git clone https://github.com/domingogallardo/play-proyecto-inicial.git
```

Se creará el directorio `play-proyecto-inicial` en el que se habrá
descargado la aplicación Play.

### Uso de IntelliJ para trabajar con la aplicación Play ###

Vamos a desarrollar las aplicaciones Play usando el IDE IntelliJ
IDEA.

Las aplicaciones Play se pueden escribir en Java y en Scala. Nosotros
usaremos Java. Una parte importante de las librerías del framework
están escritas en Scala, por lo que debe estar descargado el plugin de
Scala de IntelliJ.

Se debe importar el proyecto usando la opción `sbt`, la herramienta de
build que usa Play. 

<img src="imagenes/import-intellij-1.png" width="400px"/>

Sbt es una herramienta similar a Maven para Java, es la herramienta que
se usa para construir proyectos Scala. El fichero de configuración de
un proyecto sbt es el fichero `build.sbt` situado en el directorio
raíz.

Para compilar los proyectos también es necesario tener instalado
JDK. Nos aseguramos de que aparece en el panel de importación. Si no,
seleccionamos el directorio donde se encuentra la versión del JDK que
vamos a utilizar.

<img src="imagenes/import-intellij-2.png" width="400px"/>

Es conveniente activar la auto-importación del proyecto sbt. De esta
forma, si IntelliJ detecta algún cambio en la configuración sbt
realiza la importación de forma automática.

<img src="imagenes/sbt-enable-auto-import.png" width="300px"/>

Si se pincha en el icono de la esquina inferior izquierda de la
ventana de IntelliJ podremos activar o desactivar la visualización de
los nombres de los paneles en los bordes de la ventana. 

<img src="imagenes/mostrar-nombre-paneles.png" width="80px"/>

Es recomendable dejarlos visibles.

<img src="imagenes/bordes-con-nombres-paneles.png" width="400px"/>

Utilizaremos el panel `Terminal` para trabajar con `sbt` y con
`git`. Es recomendable abrir dos tabs, uno para cada cosa. 

<img src="imagenes/terminal.png" width="500px"/>


### Lanzar la aplicación Play ###

Podemos ejecutar la aplicación Play de tres formas:

- Usando una máquina Docker.
- Usando el `sbt shell` que proporciona IntelliJ.
- Usando la configuración de ejecución de IntelliJ.

#### Máquina Docker ####

La máquina docker `domingogallardo/playframework` contiene todo el
software y librerías para ejecutar aplicaciones Play 2.5.18. Una vez
descargada la máquina ya no es necesario descargar librerías
adicionales.

Desde el terminal lanzamos el comando `docker run` para ejecutar la
máquina sobre el directorio actual conectando el puerto 9000 en el
mismo puerto de nuestra máquina:

```text
$ docker run --rm  -it -v "${PWD}:/code" -p 9000:9000 domingogallardo/playframework
```

Aparecerá el prompt de sbt con el nombre del proyecto. Podremos lanzar
comandos de sbt como `run` o `test`.

Haciendo `run` se ejecuta la aplicación web. Por defecto el puerto al
que hay que atacar es el 9000. Accediendo a la URL
<http://localhost:9000> pondremos en marcha la aplicación.

#### Shell sbt de IntelliJ ####

También es posible arrancar el shell de `sbt` en un panel propio que
proporciona IntelliJ.

<img src="imagenes/sbt-shell.png" width="400px"/>

Cuando lo ejecutamos por primera vez se descargan todas las librerías
necesarias. 

#### Configuración de ejecución de IntelliJ ####

También podemos crear una configuración de ejecución de IntelliJ con
la que podremos ejecutar y depurar los proyectos con la opción _Run >
Edit Configurations.._.

<img src="imagenes/run-debug-configuration.png" width="400px"/>

Una vez creada la configuración podremos seleccionarla y realizar una
ejecución o una depuración de la aplicación pulsando los botones
correspondientes en la parte superior de la ventana de IntelliJ.

<img src="imagenes/botones-run-debug.png" width="200px"/>


### Fichero routes ###

El `routes`, situado en el directorio `conf`, especifica el código a
ejecutar como respuesta a una petición HTTP.

**Fichero `conf/routes`**:

```text
# Routes
# This file defines all application routes (Higher priority routes first)
# ~~~~

# An example controller showing a sample home page
GET     /                           controllers.HomeController.index
# An example controller showing how to use dependency injection
GET     /count                      controllers.CountController.count
# An example controller showing how to write asynchronous code
GET     /message                    controllers.AsyncController.message

# Map static resources from the /public folder to the /assets URL path
GET     /assets/*file               controllers.Assets.versioned(path="/public", file: Asset)
```

Por ejemplo, cuando se recibe la petición `GET /` (cuando en
el navegador escribimos la URL `http::/localhost/`) se ejecutará el
método `index` de la clase `controllers.HomeController`. O cuando
desde el navegador accedamos a la URL `http://localhost/count` se
generará la petición `GET /count` y se ejecutará el método `count` de
la clase `controllers.CountController`.

### Controllers ###

El código a ejecutar cuando se realiza una petición HTTP se define en
métodos de clases que heredan de `Controller`. Se suelen colocar en el paquete
`controllers`. 

Por ejemplo, en la clase `controllers.HomeController` se define el
método `index`.

**Fichero `app/controllers/HomeController.java`**:

```java
public class HomeController extends Controller {

    /**
     * An action that renders an HTML page with a welcome message.
     * The configuration in the <code>routes</code> file means that
     * this method will be called when the application receives a
     * <code>GET</code> request with a path of <code>/</code>.
     */
    public Result index() {
        return ok(index.render("Your new application is ready."));
    }

}
```

Los métodos de las clases `Controller` deben devolver un objeto
`Result` que representa la respuesta HTTP a la petición. En el caso
anterior se devuelve un OK (código HTTP 200) junto con el código HTML
resultante de renderizar la vista `index`.

### Vistas ###

Las vistas se definen mediante plantillas Scala definidas en el
directorio `views`. En la llamada a renderizar la vista se pueden
pasar parámetros cuyos valores se utilizan en la propia vista.

Por ejemplo, la vista anterior `index` se define en el fichero
`index.scala.html` situado en el directorio `views`.

**Fichero `views/index.scala.html`**:

```text
@*
 * This template takes a single argument, a String containing a
 * message to display.
 *@
@(message: String)

@*
 * Call the `main` template with two arguments. The first
 * argument is a `String` with the title of the page, the second
 * argument is an `Html` object containing the body of the page.
 *@
@main("Welcome to Play") {

    @*
     * Get an `Html` object by calling the built-in Play welcome
     * template and passing a `String` message.
     *@
    @welcome(message, style = "java")

}
```

La vista recibe un parámetro `String`. En el cuerpo de la vista se
puede escribir código HTML, código Scala, y también llamar a otras
plantillas. Por ejemplo, en la vista anterior se llama a la plantilla
`main` que se define en el fichero `main.scala.html`, pasándole como
parámetro el código HTML generado por la plantilla `welcome`, definida
en el fichero `welcome.scala.html`.

**Fichero `views/main.scala.html`**:

```html
@*
 * This template is called from the `index` template. This template
 * handles the rendering of the page header and body tags. It takes
 * two arguments, a `String` for the title of the page and an `Html`
 * object to insert into the body of the page.
 *@
@(title: String)(content: Html)

<!DOCTYPE html>
<html lang="en">
    <head>
        @* Here's where we render the page title `String`. *@
        <title>@title</title>
        <link rel="stylesheet" media="screen" href="@routes.Assets.versioned("stylesheets/main.css")">
        <link rel="shortcut icon" type="image/png" href="@routes.Assets.versioned("images/favicon.png")">
        <script src="@routes.Assets.versioned("javascripts/hello.js")" type="text/javascript"></script>
    </head>
    <body>
        @* And here's where we render the `Html` object containing
         * the page content. *@
        @content
    </body>
</html>

```

**Fichero `views/welcome.scala.html`**:

```html
@(message: String, style: String = "java")

@defining(play.core.PlayVersion.current) { version =>

    <link rel="stylesheet" media="screen" href="/@@documentation/resources/style/main.css">

    <section id="top">
        <div class="wrapper">
            <h1><a href="https://playframework.com/documentation/@version/Home">@message</a></h1>
        </div>
    </section>

    <div id="content" class="wrapper doc">
        <article>

            <h1>Welcome to Play</h1>

            <p>
                Congratulations, you’ve just created a new Play application. This page will help you with the next few steps.
            </p>

            <blockquote>
                <p>
                    You’re using Play @version
                </p>
            </blockquote>

    ...
}
```

Para incorporar el valor del parámetro en la plantilla hay que
preceder el parámetro con `@`. En el ejemplo anterior se obtiene así
el mensaje, que se pinta en la parte superior de la página.

La directiva `@defining` permite obtener un valor y
asignárselo a una variable que se utiliza en un bloque de código. En
el caso anterior se utiliza para obtener la versión de Play. Otro
ejemplo de su utilización es el que aparece en la [documentación de
Play](https://www.playframework.com/documentation/2.5.x/JavaTemplates)
sobre plantillas:

```html
@defining(user.getFirstName() + " " + user.getLastName()) { fullName =>
  <div>Hello @fullName</div>
}
```

La página HTML resultante mostrada en el navegador es la siguiente:

<img src="imagenes/vista-main.png" width="500px"/>

