# Práctica 2: Pruebas y TDD con Play Framework

## 1. Objetivos

Esta práctica tiene dos objetivos principales: aprender a utilizar las
características de Play Framework relacionadas con las pruebas y
utilizar la metodología TDD para añadir funcionalidades a nuestra
aplicación.

Al igual que la primera práctica, el desarrollo de esta práctica será
individual. Tendrá una duración de 3 semanas, siendo la fecha límite
de entrega el **25 de octubre**.

Seguiremos trabajando en la aplicación `mads-todolist` que has estado
desarrollando en la primera práctica.

Continuaremos también trabajando con Git como sistema de control de
versiones y GitHub como repositorio remoto. Intregraremos Git y TDD,
haciendo que cada commit represente un incremento funcional en el
desarrollo de la aplicación y contenga y pase sus propios tests.

## 2. Conexión a base de datos MySQL

### 2.1. Cambio de clave primaria de `String` a `Integer`

En la práctica 1 el tipo de dato de la clave primaria de `Usuario` lo
especificamos como `String`. Sin embargo, en esta práctica vamos a
trabajar con una base de datos MySQL que no puede gestionar claves
autogeneradas de este tipo. Debes cambiar por ello la clave primaria
de `Usuario` a `Integer`.

**Nuevo ticket a implementar**

> Igual que en la práctica anterior, abre un ticket nuevo en Trello
> para cambiar la clave primaria de `String` a `Integer`. Realiza
> todas las modificaciones en una rama que después debes mezclar con
> _master_ usando `--no-ff`.


### 2.2. Configuración de la conexión con MySQL

Hasta ahora hemos desarrollado la aplicación usando la base de datos
en memoria `H2`. Sin embargo, para poder hacer comprobaciones y
demostraciones de la aplicación que estamos desarrollando, es
interesante poder trabajar con una base de datos que haga persistente
los datos entre distintas ejecuciones de la aplicación. También es
recomendable trabajar con bases de datos idénticas a las que
utilizaremos en producción.

**Nuevo ticket a implementar**

> Debes implementar un nuevo ticket, en el que modifiques la base de
> datos de la aplicación para usar MySQL.

Para modificar la configuración de la base de datos debemos cambiar
las preferencias definidas en el fichero de configuración
`application.conf`. Y también debemos modificar la configuración de la
unidad de persistencia de JPA. 

Para poder trabajar con MySQL debemos incluir el driver
`mysql-connector-java` en las dependencias de la aplicación, en el
fichero `build.sbt`.

**Fichero `build.sbt`**:

```
libraryDependencies ++= Seq(
   javaJpa,
   "org.hibernate" % "hibernate-entitymanager" % "4.3.7.Final",
   "mysql" % "mysql-connector-java" % "5.1.18", 
   cache,
   javaWs
```


La configuración de MySQL que vamos a definir va a acceder a una base
de datos `mads` gestionada por un servidor mysql con el usuario `root`
y la contraseña `mads`. Esta base de datos y este servidor debe
existir en la máquina en la que ejecutamos la aplicación Play. Para
instalar MySQL en Linux y crear esta base de datos debemos activar en
la máquina virtual el acceso al repositorio de software `Canonical`
(se encuentra deshabilitado):

_Configuración del sistema > Software y actualizaciones > Canonical_

<img src="imagenes/canonical.png" width="500px">

```
$ sudo apt-get install mysql-server
```

Durante la instalación nos pedirá la contraseña del usuario
root. Introducimos `mads`. Una vez instalado MySQL debemos crear la
base de datos `mads`:

```
$ mysql -u root -p
Enter password: mads
mysql> create database mads;
```

Definimos después dos unidades de persistencia en el fichero
`conf/META-INF/persistence.xml`, para poder trabajar desde JPA con
cualquiera de las dos configuraciones:

**Fichero `conf/META-INF/persistence.xml`**:

```xml
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence 
http://xmlns.jcp.org/xml/ns/persistence/persistence_2_1.xsd"
             version="2.1">

    <!-- Memory persistence Unit -->

    <persistence-unit name="memoryPersistenceUnit" transaction-type="RESOURCE_LOCAL"> 
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <non-jta-data-source>DefaultDS</non-jta-data-source>
        <properties>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
            <property name="hibernate.hbm2ddl.auto" value="update"/>
        </properties>
    </persistence-unit>

    <!-- MySQL Persistence Unit -->

    <persistence-unit name="mySqlPersistenceUnit" transaction-type="RESOURCE_LOCAL">
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <non-jta-data-source>DefaultDS</non-jta-data-source>
        <properties>
            <property name="hibernate.dialect" value="org.hibernate.dialect.MySQL5Dialect"/>
            <property name="hibernate.hbm2ddl.auto" value="update"/>
        </properties>
    </persistence-unit>
</persistence>
```

La unidad de persistencia `memoryPersistenceUnit` define las
características de la conexión con la base de datos en memoria H2 y la
unidad `mySqlPersistenceUnit` define la conexión con una base de datos
MySQL.

Y cambiamos el fichero de configuración `application.conf` para
trabajar con la base de datos MySQL. Cambia la configuración
comentando las propiedades de la base de datos en memoria y
actualizando las de MySQL (puedes hacerlo al revés, si necesitas
volver a trabajar con `H2`).

**Fichero `application.conf`**:

```
# Memory H2 database
# jpa.default = memoryPersistenceUnit

# MySQL database
jpa.default = mySqlPersistenceUnit

db {
  # Memory H2 database
  # https://www.playframework.com/documentation/latest/Developing-with-the-H2-Database
  # default.driver=org.h2.Driver
  # default.url="jdbc:h2:mem:play"

  # MySQL database
  default.driver=com.mysql.jdbc.Driver
  default.url="jdbc:mysql://localhost:3306/mads"
  default.username=root
  default.password="mads"

  # You can expose this datasource via JNDI if needed (Useful for JPA)
  default.jndiName=DefaultDS

  # You can turn on SQL logging for any datasource
  # https://www.playframework.com/documentation/latest/Highlights25#Logging-SQL-statements
  #default.logSql=true
}
```

### 2.3. Comprobación del funcionamiento

Podemos ejecutar la aplicación y comprobar que los datos introducidos
en ella aparecen en la base de datos:

```
$ mysql -u root -p 
Enter password: mads
mysql> use mads;
mysql> show tables;
mysql> select * from Usuario;
```

También puedes modificar algún dato y comprobar que en la aplicación
también aparece modificado:

```
mysql> update Usuario set apellidos = 'Pepito Pérez' where id = 1;
mysql> insert into Usuario (id, apellidos, login, nombre) 
values (3, 'Cantó', 'anabel', 'Anabel');
```


## 3. Tests en Play Framework

Play Framework en Java utiliza [JUnit](http://junit.org) como
framework de testing. Los siguientes enlaces proporcionan información
inicial sobre testing en Play Framework. No vamos a entrar en
profundidad en todas las posibilidades (_mocking_, inyección de
dependencias, etc.), sólo aprender lo necesario para definir algunos
tests que sean útiles:

- Documentación Play -
  [Testing your application](https://playframework.com/documentation/2.5.x/JavaTest)
- Documentación Play -
  [Writing functional tests](https://playframework.com/documentation/2.5.x/JavaFunctionalTest)
- Documentación Play -
  [Testing with databases](https://playframework.com/documentation/2.5.x/JavaTestingWithDatabases)
- Documentación Play -
  [The Play WS API](https://www.playframework.com/documentation/2.5.x/JavaWS)
- Play Java API -
  [play.test.Helpers](https://www.playframework.com/documentation/2.5.x/api/java/play/test/Helpers.html)
- Play Java API -
  [play.mvc.Http.Status](https://www.playframework.com/documentation/2.5.x/api/java/play/mvc/Http.Status.html)

Todos los tests deben incluirse en el directorio `tests`. Podemos
lanzar todos los tests desde la consola Activator

```
[mads-todolist] $ test
```

Y también podemos lanzar sólo los tests definidos en una clase:

```
[mads-todolist] $ testOnly my.namespace.MyTest
```

Y para lanzar sólo los tests que han fallado:

```
[mads-todoslist] $ testQuick
```

Para hacerlo desde el terminal, llamando a `activator`:

```
$ activator test
$ activator "testOnly my.namespace.MyTest" (cuidado: hay que escribir las comillas)
$ activator testQuick
```

En Play es posible lanzar tests sobre ejecuciones de la aplicación. De
esta forma no es necesario crear _mocks_ ni _stubs_ (aunque es posible
hacerlo, si queremos mejorar el rendimiento de la ejecución de los
tests o aislar el código a probar del resto de componentes y
recursos). 

Podemos también configurar en los tests la conexión a la base de datos
con la que se lanza el test, haciendo independiente la ejecución del
test de la configuración de desarrollo. Por ejemplo, podemos usar una
configuración de desarrollo en la que trabajamos con una base de datos
MySQL, y hacer que los tests se ejecuten con una configuración de base
de datos en memoria. También podemos utilizar una base de datos de
tests en la que hemos añadido casos de prueba que se utilizarán en los
tests. Veremos distintos ejemplos en el siguiente apartado.

### 3.1. Introducción

#### 3.1.1. JUnit

Los tests se construyen usando JUnit:

```java
import static org.junit.Assert.*;

import org.junit.Test;

public class SimpleTest {

  @Test
  public void testSum() {
    int a = 1 + 1;
    assertEquals(2, a);
  }
    
  @Test
  public void testString() {
    String str = "Hello world";
    assertFalse(str.isEmpty());
  }

}
```

#### 3.1.2. Aserciones y matchers

PlayFramework también incluye la librería
[Hamcrest](http://hamcrest.org/JavaHamcrest/) que permite escribir 
aserciones de más alto nivel:

```java
import static org.hamcrest.CoreMatchers.*;
import static org.junit.Assert.assertThat;

import org.junit.Test;

public class HamcrestTest {
  
  @Test
  public void testString() {
    String str = "good";
    assertThat(str, allOf(equalTo("good"), startsWith("goo")));
  }

}
```


#### 3.1.3. Mocks

Los _mocks_ se usan para aislar los tests unitarios de dependencias
externas. Por ejemplo, si la clase a probar depende de una clase
externa de acceso a datos, es posible _mockear_ esta clase para
proporcional datos controlados y eliminar la necesidad del recurso
externo. 

Para ello se usa la librería [Mockito](https://github.com/mockito/mockito)

Usando Mockito, es posible _mockear_ clases o interfaces. Por ejemplo:

```java
import static org.mockito.Mockito.*;

// Crea e inicializa el mock
List<String> mockedList = mock(List.class);
when(mockedList.get(0)).thenReturn("first");

// comprueba el valor
assertEquals("first", mockedList.get(0));

// verifica la interacción
verify(mockedList).get(0);
```

### 3.2. Tests con bases de datos y JPA

#### 3.2.1. Conexión de JPA con una base de datos en memoria

Para verificar código que accede a la base de datos es posible
inicializar en el propio test una base de datos con el mismo nombre
JNDI que el que utiliza JPA. De esta forma, el código a probar usará
la base de datos configurada en el test, en lugar de la definida en la
propia aplicación.

Lo habitual es utilizar una base de datos en memoria (como H2) sobre
la que ejecutar los tests. Así el test se ejecutará mucho más rápido y
podrá realizarse sin necesidad de una base de datos real que trabaje
sobre el disco duro. Podremos, por ejemplo, ejecutar los tests en
entornos _cloud_ sin necesidad de configurar una base de datos como
MySQL. 

**Nuevo ticket a implementar**

> Debes implementar un nuevo ticket, en el que definas el fichero
> `dao/UsuarioDaoTest.java` que ejecute un mínimo de 5 tests sobre la
> clase `UsuarioDao` usando una base de datos vacía en memoria.

El siguiente fichero `UsuarioDaoTest.java` muestra cómo hacerlo.

**Fichero `test/dao/UsuarioDaoTest.java`**:

```java
package dao;

import play.db.Database;
import play.db.Databases;
import play.db.jpa.*;
import org.junit.*;
import static org.hamcrest.CoreMatchers.*;
import static org.junit.Assert.*;

import models.*;

public class UsuarioDaoTest {

    Database db;
    JPAApi jpa;

    @Before
    public void initDatabase() {
        db = Databases.inMemoryWith("jndiName", "DefaultDS");
        // Necesario para inicializar el nombre JNDI de la BD
        db.getConnection();
        // Se activa la compatibilidad MySQL en la BD H2
        db.withConnection(connection -> {
            connection.createStatement().execute("SET MODE MySQL;");
        });
        jpa = JPA.createFor("memoryPersistenceUnit");
    }

    @After
    public void shutdownDatabase() {
        db.withConnection(connection -> {
            connection.createStatement().execute("DROP TABLE Usuario;");
        });
        jpa.shutdown();
        db.shutdown();
    }

    @Test
    public void creaBuscaUsuario() {
        Integer id = jpa.withTransaction(() -> {
            Usuario nuevo = new Usuario("pepe", "pepe");
            nuevo = UsuarioDAO.create(nuevo);
            return nuevo.id;
        });

        jpa.withTransaction(() -> {
            Usuario usuario = UsuarioDAO.find(id);
            assertThat(usuario.login, equalTo("pepe"));
        });
    }

    @Test
    public void buscaUsuarioLogin() {
        jpa.withTransaction(() -> {
            Usuario usuario = UsuarioDAO.findUsuarioPorLogin("pepe");
            assertNull(usuario);
        });
    }
}
```

La inicialización de la base de datos en memoria se hace con los
métodos anotados con `@Before` y `@After`:

- Las anotaciones **`@Before`** y **`@After`** definen código que se ejecuta
  antes y después de cada test. 
- El método **`initDatabase()`** (que se ejecuta antes de cada test)
  crea una base de datos en memoria y le asigna el nombre JNDI
  `DefaultDS`. Después la instrucción
  `JPA.createFor("memoryPersistenceUnit")` inicializa JPA usando la
  configuración definida en la unidad de persistencia
  `memoryPersistenciUnit` (en el fichero `META-INF/persistence.xml`)
  y, por último, guarda un `JPAApi` en la variable `jpa` para poder
  lanzar transacciones en los tests.
- El método **`shutdownDatabase()`** limpia la base de datos al final
  de cada test y cierra las conexiones.

En cada test:

- La base de datos está vacía al comenzar el test.
- Se actualiza la base de datos insertando los datos que nos interesa,
  usando el método `create()` del DAO. Una vez insertados los datos,
  se llaman a los método a probar. Las acciones se realizan en
  transacciones independientes usando el método `withTransaction()`
  del `JPAApi`.


Para lanzar los tests:

```
$ activator "testOnly dao.UsuarioDaoTest"
```

En el fichero mostrado anteriormente hay dos tests. Debes definir tres
tests adicionales que comprueben otros métodos del DAO.


#### 3.2.2. Inicialización de la base de datos con DBUnit

Para poder inicializar la base de datos con un conjunto de datos
previos es mucho más aconsejable utilizar la librería DBUnit.

**Nuevo ticket a implementar**

> Debes implementar un nuevo ticket, en el que definas el fichero
> `dao/UsuarioDaoDbUnitTest.java` que ejecute un mínimo de 5 tests
> sobre la clase `UsuarioDao` usando una base datos en memoria
> inicializada con DBUnit.

Para ello hay que hacer los siguiente cambios:

**Añadir las librerías de DBUnit en `build.sbt`**:

```
...
libraryDependencies ++= Seq(
  javaJpa,
  "org.hibernate" % "hibernate-entitymanager" % "4.3.7.Final",
  "mysql" % "mysql-connector-java" % "5.1.18",
  "org.dbunit" % "dbunit" % "2.4.9",
  cache,
  javaWs
n)
```

Modificamos la clase `Usuario` para indicar que el campo
`fechaNacimiento` se mapea con una columna de la base de datos de tipo
`DATE` (si lo dejamos como antes, en la base de datos se crea una
columna del tipo `TIMESTAMP`, que hay que inicializar también con
horas, minutos y segundos).

**Fichero `app/models/Usuario.java`**:

```java
Entity
public class Usuario {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    public Integer id;
    ...
    @Formats.DateTime(pattern="dd-MM-yyyy")
    @Temporal(TemporalType.DATE)
    public Date fechaNacimiento;
    ...
}
```

Definimos en los datos iniciales de la base de datos en un fichero de
configuración XML.

**Fichero `test/resources/usuario_dataset.xml`**:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<dataset>
    <Usuario id="1" login="juan" nombre="Juan" apellidos="Gutierrez"
        eMail="juan.gutierrez@gmail.com" fechaNacimiento="1993-12-10"/>
    <Usuario id="2" login="anabel" nombre="Anabel" eMail="anabel.perez@gmail.com"/>
</dataset>
```

Hay que hacer notar que en DBUnit hay que definir las fechas con el formato
`AAAA-MM-DD`.

Una vez hecho esto, ya podemos definir una nueva clase de test en la
que se utiliza DBUnit para cargar los datos y limpiar la base de datos
en memoria:

**Fichero `test/dao/UsuarioDaoDbUnitTest.java`**:

```java
package dao;

import play.db.Database;
import play.db.Databases;
import play.db.jpa.*;
import org.junit.*;
import org.dbunit.*;
import org.dbunit.dataset.*;
import org.dbunit.dataset.xml.*;
import java.io.FileInputStream;
import static org.hamcrest.CoreMatchers.*;
import static org.junit.Assert.*;
import java.text.SimpleDateFormat;
import java.util.Date;

import models.*;

public class UsuarioDaoDbUnitTest {

    static Database db;
    static JPAApi jpa;
    JndiDatabaseTester databaseTester;

    @BeforeClass
    static public void initDatabase() {
        db = Databases.inMemoryWith("jndiName", "DefaultDS");
        // Necesario para inicializar el nombre JNDI de la BD
        db.getConnection();
        // Se activa la compatibilidad MySQL en la BD H2
        db.withConnection(connection -> {
            connection.createStatement().execute("SET MODE MySQL;");
        });
        jpa = JPA.createFor("memoryPersistenceUnit");
    }

    @Before
    public void initData() throws Exception {
        databaseTester = new JndiDatabaseTester("DefaultDS");
        IDataSet initialDataSet = new FlatXmlDataSetBuilder().build(new
        FileInputStream("test/resources/usuario_dataset.xml"));
        databaseTester.setDataSet(initialDataSet);
        databaseTester.onSetup();
    }

    @After
    public void clearData() throws Exception {
        databaseTester.onTearDown();
    }

    @AfterClass
    static public void shutdownDatabase() {
        jpa.shutdown();
        db.shutdown();
    }

    @Test
    public void findUsuarioPorLogin() {
        jpa.withTransaction(() -> {
            Usuario usuario = UsuarioDAO.findUsuarioPorLogin("juan");
            SimpleDateFormat sdf = new SimpleDateFormat("dd-MM-yyy");
            try {
                Date diezDiciembre93 = sdf.parse("10-12-1993");
                assertTrue(usuario.login.equals("juan") &&
                           usuario.fechaNacimiento.compareTo(diezDiciembre93) == 0);
            } catch (java.text.ParseException ex) {
            }
        });
    }

    @Test
    public void actualizaUsuario() {
        jpa.withTransaction(() -> {
            Usuario usuario = UsuarioDAO.find(2);
            usuario.apellidos = "Anabel Pérez";
            UsuarioDAO.update(usuario);
        });

        jpa.withTransaction(() -> {
            Usuario usuario = UsuarioDAO.find(2);
            assertThat(usuario.apellidos, equalTo("Anabel Pérez"));
        });
    }

}
```

Esta vez la inicialización de la base de datos es algo distinta:

- Las anotaciones **`@BeforeClass`** y **`@AfterClass`** definen
  métodos que se ejecutan sólo una vez, al principio y al final de
  todos los tests.
- El método **`initDatabase()`** crea una base de datos en memoria y
  le asigna el nombre JNDI `DefaultDS`. Después la instrucción
  `JPA.createFor("memoryPersistenceUnit")` inicializa JPA usando la
  configuración definida en la unidad de persistencia
  `memoryPersistenciUnit` y, por último, guarda un `JPAApi` en la
  variable `jpa` para poder lanzar transacciones en los tests. La base
  de datos y la JPAApi son estáticas, y tienen un ámbito de vida de
  todos los tests.
- El método **`shutdownDatabase()`** cierra las conexiones con la base
  de datos y con JPA al final de ejecutar todos los tests.
- Los métodos **`initData()`** y **`clearData()`** son los métodos que
  usan el API de DBUnit para inicializar la base de datos y borrarla
  al principio y al final de ejecutar cada test.

En cada test se comprueba dentro de una transacción que el resultado
de ejecutar una llamada a un método del DAO, partiendo del estado de
la base de datos definido por el fichero XML, es el esperado.

En el fichero mostrado anteriormente hay dos tests. Debes definir tres
tests adicionales que comprueben otros métodos del DAO.

### 3.3. Tests de la capa de servicios

### 3.4. Tests de la capa de controladores


## 4. Ampliación de la aplicación: tareas usando TDD

