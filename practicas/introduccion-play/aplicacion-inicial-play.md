
## Aplicación inicial Play

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

