
# Práctica 2: Pruebas y TDD con Play Framework

- [1. Objetivos](#1-objetivos)
- [2. Conexión a base de datos MySQL](#2-conexión-a-base-de-datos-mysql)
    - [2.1. Cambio de clave primaria de String a Integer](#21-cambio-de-clave-primaria-de-string-a-integer)
    - [2.2. Configuración de la conexión con MySQL](#22-configuración-de-la-conexión-con-mysql)
    - [2.3. Comprobación del funcionamiento](#23-comprobación-del-funcionamiento)
- [3. Tests en Play Framework](#3-tests-en-play-framework)
    - [3.1. Introducción](#31-introducción)
    - [3.2. Tests con bases de datos y JPA](#32-tests-con-bases-de-datos-y-jpa)
    - [3.3. Tests de la capa de servicios](#33-tests-de-la-capa-de-servicios)
- [4. Ampliación de la aplicación: CRUD de tareas usando TDD](#4-ampliación-de-la-aplicación-crud-de-tareas-usando-tdd)
    - [4.1. Primera funcionalidad: listado de tareas](#41-primera-funcionalidad-listado-de-tareas)
    - [4.2. Siguientes funcionalidades](#42-siguientes-funcionalidades)
- [5. Entrega y evaluación](#5-entrega-y-evaluación)

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
tests que sean útiles.

A pesar que Play permite una gran flexibilidad a la hora de testear
sus distintos elementos, sólo vamos a realizar tests de la **capa de
persistencia** (DAO) y de la **capa de servicios**.

- [Testing your application](https://playframework.com/documentation/2.5.x/JavaTest)
- [Testing with databases](https://playframework.com/documentation/2.5.x/JavaTestingWithDatabases)

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

**Fichero `test/resources/usuarios_dataset.xml`**:

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
        FileInputStream("test/resources/usuarios_dataset.xml"));
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

Los tests sobre la capa de servicios se hacen de forma similar a los
de la capa DAO. Podemos usar también DBUnit para inicializar la base
de datos de memoria con datos de prueba iniciales.

**Nuevo ticket a implementar**

> Debes implementar un nuevo ticket, en el que definas el fichero
> `services/UsuariosServiceTest.java` que ejecute un mínimo de 5 tests
> sobre la clase `UsuariosService` usando una base datos en memoria
> inicializada con DBUnit.

El primer test que debes implementar es uno en el que se compruebe que
**si intentamos modificar un usuario con un login ya existente se debe
lanzar una excepción**. Para que el test funcione correctamente tienes
que modificar el método `UsuariosService.modificaUsuario()` para que
lance una excepción de tipo `UsuariosException` si se ha modificado el
login que se pasa como parámetro a uno que ya existe en la base de
datos.

El test es interesante porque demuestra una importante característica
de JPA: si estamos dentro de una transacción las entidades devueltas por
los métodos del servicio están conectadas a la base de datos. Esta
conexión hace que si modificamos uno de sus atributos, automáticamente
se modifica la base de datos. 

Por ejemplo, el siguiente código no funcionaría bien, porque el objeto
`usuario` devuelto por `findUsuario()` **está conectado** a la base de
datos y JPA actualizará la base de datos cuando modifiquemos alguno de
sus atributos. En concreto, se modifica el login antes de invocar al
método `modificaUsuario()` y cuando `modificaUsuario()` hace la
comprobación de si el login está repetido, en la base de datos ya hay
dos usuarios con el login 'juan'.


```java
    // 
    // CÓDIGO ERRÓNEO, NO COPIAR
    //
    // 
    jpa.withTransaction(() -> {
        Usuario usuario = UsuariosService.findUsuario(2);

        // Modificamos el login por uno ya existente en
        // la base de datos.

        usuario.login = "juan";

        // Pero está mal: al estar el usuario conectado 
        // a la base de datos se actualiza el login en la base 
        // de datos justo en este momento, antes de llamar 
        // al método modificaUsuario.

        try {
            UsuariosService.modificaUsuario(usuario);
            fail("No se ha lanzado excepción login ya existe");
        } catch (UsuariosException ex) {
        }
    });
```

Una forma de corregir el código anterior es copiar los datos del
usuario conectado a otro dato nuevo, que no haya sido devuelto por JPA
y que esté desconectado de la base de datos. Para ello definimos un
nuevo método `copy()` en la clase `Usuario`, que únicamente crea un
usuario nuevo y copia sus datos.

```java
    @Test
    public void actualizaUsuarioLanzaExcepcionSiLoginYaExiste() {
        jpa.withTransaction(() -> {
            Usuario usuario = UsuariosService.findUsuario(2);

            // Copiamos el objeto usuario para crear un objeto igual
            // pero desconectado de la base de datos. De esta forma,
            // cuando modificamos sus atributos JPA no actualiza la
            // base de datos.

            Usuario desconectado = usuario.copy();

            // Cambiamos el login por uno ya existente
            desconectado.login = "juan";

            try {
                UsuariosService.modificaUsuario(desconectado);
                fail("No se ha lanzado excepción login ya existe");
            } catch (UsuariosException ex) {
            }
        });
    }
```

Escribe cuatro tests más que comprueben los métodos de la clase
`UsuariosService`.

## 4. Ampliación de la aplicación: CRUD de tareas usando TDD

La última parte de la práctica consiste en desarrollar, utilizando TDD
(_Test Driven Design_) las funcionalidades necesarias para realizar un
CRUD de tareas de usuarios.

Un resumen de lo que deberás hacer en esta parte de la práctica:

- Añadirás una a una nuevas tarjetas en Trello correspondientes a las
  nuevas funcionalidades que vayas desarrollando.
- Al igual que hicimos en la práctica 1, cada ticket debe
  desarrollarse en una rama independiente que después se mezcla con la
  rama master.
- Se definirá un fichero de test por cada funcionalidad, que contendrá
  todos los tests necesarios para implementar la funcionalidad y se
  denominará con un nombre similar al de la funcionalidad que queremos
  implementar.
- En el fichero de test se incluirán tests de la capa DAO y de capa de
  servicios. Deberás realizar un enfoque de dentro a fuera, realizando
  primero los tests de la capa DAO y después la de servicios.
- Cada test, junto con el código desarrollado para pasarlo, irá en un
  commit.
- Es posible hacer commits con refactorizaciones (recuerda el ciclo de
  TDD: Test, Codigo y Refactorización).
- Utilizarás DBUnit y una base de datos de memoria para poder hacer
  pruebas con datos iniciales. Utilizaremos un nuevo fichero
  `tareas_dataset.xml`.

Implementaremos cada característica utilizando el ciclo de desarrollo
de TDD:

1. Escribir un test que falla
2. Escribir el código que hace que el test deje de fallar
3. Refactorizar el código de los tests y el código escrito (sin
   añadir nuevos tests, ni nuevas funcionalidades).
4. Volver al paso 1

Veamos como ejemplo el desarrollo completo de la primera
funcionalidad: **listado de tareas de un usuario**.

### 4.1 Primera funcionalidad: listado de tareas

En esta característica queremos desarrollar la página con el listado
de tareas de un usuario, que se debe devolver cuando se haga una
petición GET a la URL correspondiente a las tareas del usuario. Por
ejemplo, si accedemos a la URL `/usuarios/1/tareas` se devolverá un
listado con todas las tareas del usuario `1`.

Para implementar esta característica usando TDD crearemos el fichero
de test `test/ListadoTareasTests.java` en el que iremos añadiendo los
distintos tests, desde los de la capa más interior que contiene las
entidades (los objetos de la base de datos), el DAO y el servicio.

Creamos una tarjeta en Trello con el nombre **Listado de tareas**,
le damos el número de _ticket_ correspondiente. Supongamos que es el
número 20.

Abrimos una rama en el repositorio en el que iremos añadiendo los
commits de la funcionalidad (uno por cada test). En todos los commits
usaremos el número de ticket anterior.

```
$ git checkout -b tic-20
```

Empezamos ahora a usar TDD para implementar la funcionalidad. El
primer test nos servirá para definir los elementos básicos de la
entidad `Tarea`.

Creamos un fichero `test/resources/tareas_dataset.xml`, en el que
definiremos los datos que cargaremos con DBUnit en la base de datos de prueba:

**Fichero `test/resources/tareas_dataset.xml`**:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<dataset>
    <Usuario id="1" login="juan" nombre="Juan" apellidos="Gutierrez"
        eMail="juan.gutierrez@gmail.com" fechaNacimiento="1993-12-10"/>
    <Usuario id="2" login="anabel" nombre="Anabel" eMail="anabel.perez@gmail.com"/>
    <Tarea id="1" descripcion="Preparar el trabajo del tema 1 de biología" />
    <Tarea id="2" descripcion="Estudiar el parcial de matemáticas" />
    <Tarea id="3" descripcion="Leer el libro de inglés" />
    <Tarea id="4" descripcion="Salir a correr" />
</dataset>
```

Vemos que cada tarea tiene, inicialmente, un identificador y una
descripción. Iremos ampliando estos datos en las distintas
iteraciones. 

Creamos el fichero `test/ListadoTareasTest.java` en el que iremos
escribiendo uno a uno los tests que implementarán la
funcionalidad. 

#### Primer test

Empezamos definiendo el código que carga las tareas en la base de
datos de memoria y el test más sencillo posible: buscar una tarea por
su identificador.

**Fichero `test/ListadoTareasTest.java`**:

```java
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

public class ListadoTareasTest {

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
        FileInputStream("test/resources/tareas_dataset.xml"));
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
    public void findTareaPorId() {
        jpa.withTransaction(() -> {
            Tarea tarea = TareaDAO.find(1);
            assertThat(tarea.descripcion, equalTo("Preparar el trabajo del tema 1 de biología"));
        });
    }
}
```

Si ahora lanzamos este test comprobaremos que falla:

```
$ activator "testOnly ListadoTareasTests"
```

Debemos ahora escribir el código que hace pasar el test y hacer un
commit cuando lo hayamos terminado. Si seguimos la metodología TDD al
pie de la letra, deberemos escribir **sólo el código estrictamente
necesario** que haga pasar el test. 

En este código se crearán la entidad `models.Tarea` y la clase
`models.TareaDAO` en la que se creará el método `find()` que busca
tareas por su identificador.

Al utilizar JPA, y haber definido en el fichero de configuración
`META-INF/persistence.xml` la propiedad `hibernate.hbm2ddl.auto` con
el valor `update`, se actualizará automáticamente el esquema de la
base de datos a partir de la clase `Tarea`. Esta característica no
debería utilizarse en producción, sólo en desarrollo.

El código es el siguiente:

**Fichero `models/Tarea.java`**:

```java
package models;

import java.util.Date;
import javax.persistence.*;
import play.data.validation.Constraints;
import play.data.format.*;

@Entity
public class Tarea {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    public Integer id;
    @Constraints.Required
    public String descripcion;

    // Un constructor vacío necesario para JPA
    public Tarea() {}

    // El constructor principal con los campos obligatorios
    public Tarea(String descripcion) {
        this.descripcion = descripcion;
    }

    public String toString() {
        return String.format("Tarea id: %s descripcion: %s", id, descripcion);
    }
}
```

**Fichero `models/TareaDAO.java`**:

```java
package models;

import play.*;
import play.mvc.*;
import play.db.jpa.*;
import javax.persistence.*;

public class TareaDAO {
    public static Tarea find(Integer idTarea) {
        return JPA.em().find(Tarea.class, idTarea);
    }
}
```

Probamos que pasa correctamente el test y hacemos un commit:

```
$ git add .
$ git commit -m "TIC-XX Entidad Tarea y método find en DAO"
```

#### Segundo test

Añadimos a ahora un nuevo test, en el que comparamos entidades con y
sin identificadores:

En el **fichero `test/ListadoTareasTest.java`**:

```java
    @Test
    public void compararTareas() {
        jpa.withTransaction(() -> {
            Tarea tarea1 = TareaDAO.find(1);

            // Creamos una copia de la tarea1
            // (otro objeto con los mismos atributos)
            Tarea tarea2 = tarea1.copy();
            assertEquals(tarea1, tarea2);

            // Creamos una tarea con la misma descripción,
            // pero sin id
            Tarea tarea3 = new Tarea(tarea1.descripcion);
            assertEquals(tarea1, tarea3);
        });
    }
```

Para conseguir que pase el test añadimos en la clase `Tarea` el código
necesario para comparar tareas correctamente:

En el **fichero `models/Tarea.java`**:

```java
    public Tarea copy() {
        Tarea nueva = new Tarea(this.descripcion);
        nueva.id = this.id;
        return nueva;
    }

    @Override
    public int hashCode() {
        final int prime = 31;
        int result = prime + ((descripcion == null) ? 0 : descripcion.hashCode());
        return result;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (getClass() != obj.getClass()) return false;
        Tarea other = (Tarea) obj;
        // Si tenemos los ID, comparamos por ID
        if (id != null && other.id != null)
            return (id == other.id);
        // sino comparamos por campos obligatorios
        else {
            if (descripcion == null) {
                if (other.descripcion != null) return false;
            } else if (!descripcion.equals(other.descripcion)) return false;
        }
        return true;
    }
```

#### Tercer test

Hasta ahora hemos definido tareas independientes. Pero esto no es lo
que queremos. Queremos que las tareas estén asociadas a usuarios, de
forma que podamos recuperar, por ejemplo, todas las tareas asociadas
al usuario 1. Por ahora una tarea sólo podrá estar asociada a un único
usuario. 

En JPA esto se consigue definiendo una **relación uno-a-muchos** entre
`Usuario` y `Tarea`. Un usuario está relacionado con muchas tareas y
cada tarea está relacionada con un único usuario. Para ello deberemos
usar la anotación
[OneToMany](http://www.objectdb.com/java/jpa/entity/fields) en las
entidades implicadas en la relación.

La anotación generará en la tabla de tareas una columna con una clave
ajena hacia el identificador de usuario asociada a la tarea.

Vamos entonces a modificar el dataset de prueba y a escribir el test
que debe pasarse.

**Fichero `test/resources/tareas_dataset.xml`**:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<dataset>
    <Usuario id="1" login="juan" nombre="Juan" apellidos="Gutierrez"
        eMail="juan.gutierrez@gmail.com" fechaNacimiento="1993-12-10"/>
    <Usuario id="2" login="anabel" nombre="Anabel" eMail="anabel.perez@gmail.com"/>
    <Tarea id="1" usuarioId="1" descripcion="Preparar el trabajo del tema 1 de biología" />
    <Tarea id="2" usuarioId="1" descripcion="Estudiar el parcial de matemáticas" />
    <Tarea id="3" usuarioId="1" descripcion="Leer el libro de inglés" />
    <Tarea id="4" usuarioId="2" descripcion="Salir a correr" />
</dataset>
```

En el test buscaremos una tarea y comprobaremos que el usuario de esa
tarea coincide con el que debe:

**Fichero `test/ListadoTareasTest.java`**:

```java
    @Test
    public void obtenerUsuarioDeTarea() {
        jpa.withTransaction(() -> {
            Tarea tarea = TareaDAO.find(1);
            Usuario usuario = UsuarioDAO.find(1);
            assertEquals(tarea.usuario, usuario);
        });
    }
```

Escribimos el código para que el test pase:

**Fichero `models/Usuario.java`**:

```java
import java.util.List;
import java.util.ArraySet;

@Entity
public class Usuario {
    ...
    @OneToMany(mappedBy="usuario")
    public List<Tarea> tareas = new ArrayList<Tarea>();
}
```

La anotación `mappedBy` indica el nombre del campo en el otro lado de
la relación que se mapea con la columna que contiene la clave ajena
(en este caso el campo `usuario` de la `Tarea`).

**Fichero `models/Tarea.java`**:

```java
@Entity
public class Tarea {
    ...
    @ManyToOne
    @JoinColumn(name="usuarioId")
    public Usuario usuario;
}
```

La anotación `@JoinColumn(name="usuarioId")` indica que el campo
`usuario` se mapea con la columna `usuarioId` de la tabla `Tarea` y
que es una clave ajena a la tabla `Usuario`.

#### Cuarto test

En el cuarto test queremos poder recuperar la lista de todas las
tareas de un usuario, accediendo al campo `tareas` del mismo:


**Fichero `test/ListadoTareasTest.java`**:

```java
    @Test
    public void obtenerTareasDeUsuario() {
        jpa.withTransaction(() -> {
            Usuario usuario = UsuarioDAO.find(1);
            assertEquals(usuario.tareas.size(), 3);
        });
    }
```

Esta vez no hace falta implementar nada, porque una de las
características de JPA es que podemos acceder a los elementos de la
colección en una relación uno-a-muchos, siempre que estemos dentro de
la misma transacción en la que se ha recuperado el objeto que contiene
la relación (el usuario en este caso). 

JPA recupera esos elementos de forma _perezosa_ y cuando se accede a
la colección se realiza la consulta a la base de datos y se recuperan
todos los elementos de la relación (todas las tareas, en este caso).

##### Quinto y último test

Definimos el último test, en el que especificamos el funcionamiento
del servicio `TareasService`:

```java

    @Test
    public void listadoTareasService() {
        jpa.withTransaction(() -> {
            List<Tarea> tareas = TareasService.listaTareasUsuario(1);
            assertEquals(tareas.size(), 3);
            
            // Comprobamos que las tareas se devuelven ordenadas por id

            Tarea anterior = null;
            for (Tarea t : tareas) {
                if (anterior != null) {
                    assertTrue(anterior.id < t.id);
                    anterior = t;
                }
            }
        });
    }
```

Y escribimos el código para pasar el test.

**Fichero `services/TareasService.java`**:

```java
package services;

import java.util.List;
import java.util.ArrayList;

import models.*;

public class TareasService {

    public static List<Tarea> listaTareasUsuario(Integer usuarioId) {
        Usuario usuario = UsuarioDAO.find(usuarioId);
        if (usuario != null) {
            return usuario.tareas;
        } else {
            throw new UsuariosException("Usuario no encontrado");
        }
    }
}
```

#### Controller y vista

Termina tú de implementar el controller y la vista con las tareas de
un usuario. Esta parte hazla sin tests, como hiciste el controller
`UsuariosController`.

Para comprobar que funciona correctamente debes ejecutar la
aplicación trabajando sobre la base de datos MySQL para que se
modifique el esquema de datos con la nueva tabla. 

Y añadir manualmente un par de tareas en la base de datos MySQL:

```
$ mysql -u root -p 
password: mads
mysql> use mads;
mysql> show tables;
+----------------+
| Tables_in_mads |
+----------------+
| Tarea          |
| Usuario        |
+----------------+
mysql> insert into Tarea (id, usuarioId, descripcion) 
values (1, 1, 'Preparar el trabajo del tema 1 de biología');
mysql> insert into Tarea (id, usuarioId, descripcion) 
values (2, 1, 'Estudiar el parcial de matemáticas');
```

Una vez implementado el controller y la vista debes hacer un último
commit y un merge de la rama en la rama master (con la opción
--no-ff).


### 4.2 Siguientes funcionalidades

Implementa usando TDD sobre las capas DAO y servicios las siguientes
funcionalidades CRUD:

- Añadir tarea a un usuario
- Editar tarea de un usuario
- Borrar tarea de un usuario

Hazlo igual que el ejemplo anterior: una tarjeta para cada
funcionalidad y un commit para test y el código para pasarlo.

El controller, las vistas, y los formularios pueden ser similares a
los que usaste en la práctica 1 con el CRUD de usuarios.


## 5. Entrega y evaluación

- La práctica tiene una duración de 3 semanas y debe estar terminada
  el **martes 25 de octubre**.
- Durante el desarrollo se debe añadir el código en el repositorio en
  GitHub `mads-todolist` compartido con el profesor, y los tickets en
  el tablero Trello compartido con el profesor.
- En la fecha de la entrega se debe subir a Moodle un ZIP que contenga todo el proyecto y dejar la URL del repositorio en GitHub

Para la evaluación se tendrá en cuenta:

- Desarrollo contínuo (commits realizados a lo largo de las 3 semanas)
- Buen desarrollo y descripción de los cambios (commits bien documentados, ordenados, ramas de características visibles en la historia de commits)
- Tablero Trello bien ordenado
- Uso correcto de la nomeclatura y de TDD 
- Correcto desarrollo de las funcionalidades de la práctica
- Cuidado en el aspecto de la aplicación, la terminación, control de errores
- Características adicionales desarrolladas

