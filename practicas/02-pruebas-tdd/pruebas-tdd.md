<!--

Dejamos las pruebas con Git para la práctica 2.

5.2 Repositorio prueba Git

Vamos a empezar practicando Git. Debes crear un repositorio llamado
`mads-prueba-git` en el que pruebes los comandos básicos de Git. En
[estos apuntes de la asignatura](comandos-git.md) puedes encontrar una
explicación del funcionamiento de estos comandos.

Debes crear en el repositorio un mínimo de tres ficheros de texto
sobre algún tema que elijas (películas, series de televisión, libros,
chistes, etc.) y probar todos los siguientes comandos:

- git add
- git commit
- git status
- git diff
- git log --oneline
- creación de ramas
- merge de ramas (usando --no-ff)
- rebase de ramas (avanzamos el master y la rama, y después hacemos un rebase de la rama y un merge --no-ff desde la master)
- git log --oneline --graph (para comprobar el grafo de commits)
- comandos para modificar la historia: cambiar el último mensaje de commit, deshacer el último commit y crear una rama en un punto pasado de la historia

Sube el repositorio a GitHub y compártelo con el profesor
(`domingogallardo`).


-->

<!--

Indice:

1 - Ejecución de Play con distintas configuraciones
2 - Definición de las distintas configuraciones
3 - Realización de tests de integración y tests funcionales en los 
    entornos definidos
4 - Realización de la nueva funcionalidad con TDD

--->

# Práctica 2: Gestión de configuraciones y TDD con Play Framework

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

(Cambiar)

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


## Tests en Play Framework

Play Framework en Java utiliza [JUnit](http://junit.org) como
framework de testing. Los siguientes enlaces proporcionan información
inicial sobre testing en Play Framework. No vamos a entrar en
profundidad en todas las posibilidades (_mocking_, inyección de
dependencias, etc.), sólo aprender lo necesario para definir algunos
tests que sean útiles.

A pesar que Play permite una gran flexibilidad a la hora de testear
sus distintos elementos, sólo vamos a realizar tests de la **capa de
persistencia** (clases Repository) y de la **capa de servicios**.

- [Testing your application](https://playframework.com/documentation/2.5.x/JavaTest)
- [Testing with databases](https://playframework.com/documentation/2.5.x/JavaTestingWithDatabases)

Todos los tests se incluyen en el directorio `tests`. Podemos
lanzar todos los tests desde la consola sbt

```
[mads-todolist-2017] $ test
```

Y también podemos lanzar sólo los tests definidos en una clase:

```
[mads-todolist-2017] $ testOnly TareaServiceTest
```

O utilizar un comodín `*` para ejecutar sólo un conjunto de tests
cuyas clases comiencen por una cadena:


```
[mads-todolist-2017] $ testOnly Tarea*
```

Y para lanzar sólo los tests que han fallado:

```
[mads-todoslist] $ testQuick
```

En Play es posible lanzar tests sobre ejecuciones de la aplicación con
la sentencia `withApplication` (aunque no lo vamos a hacer). De esta
forma no es necesario crear _mocks_ ni _stubs_ (aunque es posible
hacerlo, si queremos mejorar el rendimiento de la ejecución de los
tests o aislar el código a probar del resto de componentes y
recursos).

### JUnit

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


### Tests con bases de datos y JPA

#### Conexión de JPA con una base de datos en memoria

Lo habitual para realizar tests unitarios que necesiten trabajar con
una base de datos es, o bien _mockear_ la base de datos, o bien
utilizar una base de datos en memoria (como H2) para que los tests
puedan ejecutarse mucho más rápido sin necesidad de una bases de datos
real que trabaje sobre el disco duro.

En la primera práctica hemos utilizado el segundo enfoque.


El problema de los tests tal y como están escritos en la actualidad es
que la base de datos de prueba sobre la que trabajan no se puede
definir en el fichero de configuración, sino que está definida en su
código fuente.

En concreto, la base de datos de memoria la creamos manualmente con la
instrucción `Databases.inMemoryWith()` pasando un nombre JNDI con el
que inicializarla (`DBTest`). Y después obtenemos el `JPAApi`
obteniendo la configuración de JPA `memoryPersistenceUnit`.

```java
public class UsuarioServiceTest {

   static Database db;
   static JPAApi jpaApi;
   
   @BeforeClass
   static public void initDatabase() {
      // Inicializamos la BD en memoria y su nombre JNDI
      db = Databases.inMemoryWith("jndiName", "DBTest");
      ...
      // Activamos en JPA la unidad de persistencia "memoryPersistenceUnit"
      // declarada en META-INF/persistence.xml y obtenemos el objeto
      // JPAApi
      jpaApi = JPA.createFor("memoryPersistenceUnit");
   }
   
   ...
   
   @Test
   public void crearNuevoUsuarioCorrectoTest(){
      UsuarioRepository repository = new JPAUsuarioRepository(jpaApi);
      UsuarioService usuarioService = new UsuarioService(repository);
      Usuario usuario = usuarioService.creaUsuario("luciaruiz", "lucia.ruiz@gmail.com", "123456");
      assertNotNull(usuario.getId());
      assertEquals("luciaruiz", usuario.getLogin());
      assertEquals("lucia.ruiz@gmail.com", usuario.getEmail());
      assertEquals("123456", usuario.getPassword());
   }

```


Recordemos que el perfil `memoryPersistenceUnit` está definido en el
fichero `conf/META-INF/persistence.xml`:

```xml
   <persistence-unit name="memoryPersistenceUnit" transaction-type="RESOURCE_LOCAL">
      <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
      <non-jta-data-source>DBTest</non-jta-data-source>
      <properties>
         <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
         <property name="hibernate.hbm2ddl.auto" value="update"/>
      </properties>
   </persistence-unit>
```

La anotación `@BeforeClass` hace que el método estático `initDatabase()` se
ejecute una única vez antes de todos los tests y guarde en las
variables de clase la base de datos y el JPAApi obtenido. Estas dos
variables se utilizarán más adelante.

#### Inicialización de la base de datos con DBUnit

Para poder inicializar la base de datos con un conjunto de datos
previos utilizamos la librería DBUnit.

La dependencia de la librería se declara en el fichero `build.sbt`

```
...

libraryDependencies += javaJpa
libraryDependencies += "org.hibernate" % "hibernate-core" % "5.2.5.Final"
libraryDependencies +=  "mysql" % "mysql-connector-java" % "5.1.18"
libraryDependencies += "org.dbunit" % "dbunit" % "2.4.9"

...

```

Los datos iniciales de la base de datos se definen en un fichero de
configuración XML. 

**Fichero `test/resources/usuarios_dataset.xml`**:

```xml
<?xml version="1.0" encoding="UTF-8"?>
 <dataset>
    <Usuario id="1000" login="juangutierrez" nombre="Juan" apellidos="Gutierrez"
         password="123456789" eMail="juan.gutierrez@gmail.com" fechaNacimiento="1993-12-10"/>
    <Tarea id="1000" titulo="Renovar DNI" usuarioId="1000"/>
    <Tarea id="1001" titulo="Práctica 1 MADS" usuarioId="1000"/>
 </dataset>
```

Cada línea XML define un dato en la tabla con el nombre indicado. Por
ejemplo, en el fichero anterior se define un usuario y dos
tareas. DbUnit se encarga de convertir los atributos en los tipos
correspondientes a los definidos en las tablas cuando se realiza la
inserción. Hay que hacer notar que en DBUnit hay que definir las
fechas con el formato `AAAA-MM-DD`.

Para realizar la inserción en la base de datos con la que van a
trabajar los tests se ejecuta la siguiente función:

```java

   @Before
   public void initData() throws Exception {
      JndiDatabaseTester databaseTester = new JndiDatabaseTester("DBTest");
      IDataSet initialDataSet = new FlatXmlDataSetBuilder().build(new FileInputStream("test/resources/usuarios_dataset.xml"));
      databaseTester.setDataSet(initialDataSet);
      databaseTester.setSetUpOperation(DatabaseOperation.CLEAN_INSERT);
      databaseTester.onSetup();
   }

```

Se obtiene la base de datos a través de su nombre JNDI. Y después se
cargan los datos realizando una operación `CLEAN_INSERT`. En este tipo
de operación DbUnit borra todos los datos de todas las tablas que
aparecen en el fichero XML y después realiza la inserción.

La etiqueta `@Before` hace que la función se ejecute antes de cada
test.

### Nuevo _issue_ con tests ###

Crea un nuevo _issue_ (y una rama, y después un pull request para
integrarlo con master, igual que en la práctica 1) con la descripción
`Nuevos tests práctica 2`. Puedes ponerle la etiqueta `tech`. 

Para completar el _issue_ debes hacer lo siguiente:

- Añade un usuario y una tarea en el fichero XML.
- Crea una nueva clase de tests el directorio `test/practica2` con el
  nombre `Practica2Test`. Añade en él 4 nuevos tests que comprueben
  condiciones que no hemos comprobado en la práctica 1. 
- Ejemplos: 
   - Test que comprueba que el método `findUsuarioPorId` de
  `UsuarioService` devuelve `null` si se le pasa un identificador no
  existente.
  - Test que comprueba que el método `borraTarea` de `TareaService`
    lanza una excepción si se le pasa un identificador de tarea no
    existente. 
 
## Definición de configuraciones 

Hemos comprobado en la práctica 1 que el fichero `application.conf` sirve
para definir la configuración con la que se va ejecutar la aplicación
Play. Hemos definido en él la base de datos en memoria `H2` con la que
se ejecuta la aplicación. Esta configuración es la que denominamos
**configuración de desarrollo**, que es la configuración por defecto
utilizada durante el desarrollo de la aplicación.

Vamos a ver en este apartado cómo definir configuraciones alternativas
en las que ejecutar los tests:

- **Configuración de integración**: configuración que se utilizaremos
  en las pruebas de integración. Estará constituida por una base de
  datos MySQL contendrá los datos introducidos por los tests. El hecho
  de trabajar con MySQL en lugar de la base de datos de memoria hará
  que esta configuración sea más parecida a la que se utiliza en el
  despliegue real en producción de la aplicación.
- **Configuración de stage**: configuración muy similar a la de
  producción en la que se realizarán pruebas funcionales. Estará
  formada por una base de datos MySQL con datos estables que se irán
  añadiendo en los tests funcionales, creando un entorno muy similar
  al de producción.

Lanzaremos los tests en la configuración de integración. Y
ejecutaremos la aplicación para lanzar pruebas funcionales en la
configuración de _stage_.


### Configuración de pruebas de integración ###

Empezaremos solucionando un problema importante de los tests:
conseguir que carguen el fichero de configuración, en lugar de definir
en el código las conexiones a la base de datos. Después añadiremos un
fichero de configuración de conexión con una base de datos MySQL y
veremos cómo lanzar los tests para que usen esa configuración.

#### Nuevo issue: Configuración entorno de integración ####

Debes crear un nuevo _issue_ con la descripción `Configuración entorno
de integración` en el que deberás incluir los
cambios que se indiquen los siguientes apartados. Puedes ponerle la
etiqueta `tech`.

Crea una rama en la que desarrolles este nuevo _issue_.

#### Refactorización de los tests ####

Vamos a ver cómo refactorizar los tests para que se lancen sobre una
base de datos definida en el fichero de configuración.

##### Problema: los tests no usan el fichero de configuración #####

El problema de los tests tal y como están escritos en la actualidad es
que la base de datos de prueba sobre la que trabajan no se puede
definir en el fichero de configuración, sino que está definida en su
código fuente.

Como hemos visto anteriormente, la base de datos se inicializa a mano, obteniéndose
el `JPAApi` usando una configuración concreta:

```java
   @BeforeClass
   static public void initDatabase() {
      // Inicializamos la BD en memoria y su nombre JNDI
      db = Databases.inMemoryWith("jndiName", "DBTest");
      ...
      // Activamos en JPA la unidad de persistencia "memoryPersistenceUnit"
      // declarada en META-INF/persistence.xml y obtenemos el objeto
      // JPAApi
      jpaApi = JPA.createFor("memoryPersistenceUnit");
```

Después, en cada test, usamos el `jpaApi` obtenido para construir un
_service_ o un _repostory_:

```java
   @Test
   public void crearNuevoUsuarioCorrectoTest(){
      UsuarioRepository repository = new JPAUsuarioRepository(jpaApi);
      UsuarioService usuarioService = new UsuarioService(repository);
      Usuario usuario = usuarioService.creaUsuario("luciaruiz", "lucia.ruiz@gmail.com", "123456");
      assertNotNull(usuario.getId());
      assertEquals("luciaruiz", usuario.getLogin());
      assertEquals("lucia.ruiz@gmail.com", usuario.getEmail());
      assertEquals("123456", usuario.getPassword());
   }
```

Este código hace que los tests sean poco configurables ya que no
estamos obteniendo la base de datos definida en el fichero de
configuración. Si queremos cambiar la base de datos sobre la que
trabajan los tests hay que modificar el propio código fuente.

##### Refactorización de los tests #####

Vamos a modificar el código para que la base de datos que se utilice
no esté escrita _a fuego_ en el código, sino que sea la definida en el
fichero de configuración. De esta forma, modificando el fichero
modificaremos también la configuración de los tests.

Para conseguirlo haremos en los tests igual que en la aplicación:
obteniendo la JPAApi mediante la inyección de dependencias y 

El problema es que las anotaciones `@inject` no funcionan en los
tests. Los tests son programas Java independientes que se ejecutan al
margen de la aplicación principal, que es donde se ejecuta el código
que inicializa los objetos inyectados. Tenemos nosotros que obtener a
mano los objetos llamando explícitamente a la librería `Guice` que es
la que gestiona la inyección de dependencias.

A continuación vemos cómo hacerlo en un test concreto, por ejemplo en
el fichero `UsuarioServiceTest.java`:

```java
public class UsuarioServiceTest {
   static private Injector injector;

   @BeforeClass
   static public void initApplication() {
      GuiceApplicationBuilder guiceApplicationBuilder =
          new GuiceApplicationBuilder().in(Environment.simple());
      injector = guiceApplicationBuilder.injector();
      // Instanciamos un JPAApi para que inicializar JPA
      injector.instanceOf(JPAApi.class);
   }

   private UsuarioService newUsuarioService() {
      return injector.instanceOf(UsuarioService.class);
   }
   
   @Test
   public void crearNuevoUsuarioCorrectoTest(){
      UsuarioService usuarioService = newUsuarioService();
      Usuario usuario = usuarioService.creaUsuario("luciaruiz", "lucia.ruiz@gmail.com", "123456");
      assertNotNull(usuario.getId());
      assertEquals("luciaruiz", usuario.getLogin());
      assertEquals("lucia.ruiz@gmail.com", usuario.getEmail());
      assertEquals("123456", usuario.getPassword());
   }

   ...
}
```

En el código anterior la creación de un `GuiceApplicationBuilder` con
un entorno simple realiza una inicialización de la aplicación Play con
el fichero de configuración por defecto. Después se obtiene un objeto
`Injector` que usamos para obtener instancias de clases mediante la
inyección de dependencias. En concreto, el método `newUsuarioService()`
obtiene del inyector una instancia de un `UsuarioService`. A este
método se llama en cada test para obtener un `UsuarioService` del que
probar sus métodos.

##### Commit: Refactorización de los tests #####

Debes crear un nuevo commit en la rama recién creada con la descripción
`Refactorizados tests` en el que deberás refactorizar todos los tests
para que se obtengan los objetos necesarios usando inyección de
dependencias.

Para ello realiza todos los cambios en los ficheros de tests que
aparecen en el [PR
#27](https://github.com/domingogallardo/mads-todolist-guia/pull/27/files)
de la práctica guía. No modifiques por ahora el fichero de
configuración, ni crees nuevos tests (aunque son modificaciones que
aparecen en el PR, no las debes hacer).

Una vez hecho esto deberás comprobar que los tests funcionan
correctamente los tests con la configuración original (base de datos
en memoria `H2`) y añadir en otro commit cambios para que los tests carguen una
configuración de base de datos alternativa, como una base de datos
MySQL. En el siguiente apartado vemos cómo hacerlo.

#### Ejecución de tests con base de datos MySQL ####

Vamos a ver cómo lanzar una base de datos MySQL y cómo definir el
fichero de configuración para que la aplicación Play se conecte a
ella.

##### Ejecución de una base de datos MySQL con un contenedor Docker #####

En primer lugar vamos a ver cómo lanzar la base de datos
MySQL. Podríamos realizar una instalación de MySQL en nuestro
ordenador, pero vamos a usar también Docker en lugar de
ello. Utilizaremos la imagen
[MySQL](https://store.docker.com/images/mysql).

Para lanzar un contenedor con la imagen anterior ejecutamos el
siguiente comando:

```
$ docker run -d --rm -p 3306:3306 --name play-mysql -e MYSQL_ROOT_PASSWORD=mads -e MYSQL_DATABASE=mads mysql
```

Este comando lanza un contenedor que crea una base de datos llamada
`mads` y con la contraseña de `root` `mads`. Le da el nombre
`play-mysql` que después usaremos para enlazar con el contenedor de
Play. La opción `-p 3306:3306` permite acceder a la base de datos
desde el host (lo veremos más adelante).

Podemos comprobar que el contenedor está funcionando con el comando
`docker container ls`:

```
$ docker container ls
CONTAINER ID  IMAGE  COMMAND                  CREATED         STATUS         PORTS                  NAMES
7c1bed0b5b7e  mysql  "docker-entrypoint..."   6 seconds ago   Up 4 seconds   0.0.0.0:3306->3306/tcp play-mysql
```

El comando `docker container ls` lista los contenedores activos. Podemos usar el
identificador o el nombre del contenedor para pararlo:

```
$ docker container stop play-mysql
```

Una vez parado se borrará automáticamente por haber usado la opción
`--rm` en su lanzamiento. (en el caso de no usar esta opción tendremos
que borrarlo nosotros manualmente con el comando `docker container rm
play-mysql`).


##### Ejecución de los tests con la base de datos MySQL #####

Para ejecutar los tests con la base de datos MySQL vamos a definir un
nuevo fichero de configuración que llamaremos `integration.conf`.

Utilizaremos una característica del sistema de configuración de
Play en la que podemos dar a las variables los valores definidos por
variables de entorno usando la sintaxis:

```
variable = ${?VARIABLE_ENTORNO}
```

El nuevo fichero de configuración es el siguiente:

**Fichero conf/integration.conf**  

```
include "application.conf"

jpa.default = mySqlPersistenceUnit

db.default.driver=com.mysql.jdbc.Driver
db.default.url=${?DB_URL}
db.default.username=${?DB_USER_NAME}
db.default.password=${?DB_USER_PASSWD}
```

En primer lugar el fichero incluye la configuración por defecto
`application.conf` y después modifica el valor de las variables que
nos interesan. 

En concreto inicializamos la conexión JPA por defecto al perfil
`mySqlPersistenceUnit`. Veremos más adelante la definición de este
perfil en el fichero `persistence.xml`.

Y después definimos las variables necesarias para la conexión a una
base de datos MySQL. Para que la configuración sea flexible, obtenemos
los valores de variables de entorno que estarán definidas en el
sistema en el que ejecutemos los tests de integración.

El fichero de configuración de JPA debe quedar como sigue, añadiendo el nuevo perfil
de conexión con una base de datos MySQL (el perfil `mySqlPersistenceUnit`):

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

Cuando la aplicación Play se conecte a la base de datos el esquema de
datos de actualiza automáticamente debido al valor `update` en la
propiedad `hibernate.hbm2ddl.auto`. Este valor indica que el esquema
de datos de la base de datos se actualiza para ajustarse a las
entidades definidas en la aplicación. Por ejemplo, cuando se añade
algún atributo a alguna entidad, se añade automáticamente una columna
más a la tabla correspondiente.

Una vez realizados los cambios anteriores, y estando el contenedor
docker MySQL en funcionamiento, podemos lanzar la aplicación Play 
con el siguiente comando:

```
$ docker run --link play-mysql:mysql --rm -it -p 80:9000 -e \
DB_URL="jdbc:mysql://play-mysql:3306/mads" -e DB_USER_NAME="root" -e \
DB_USER_PASSWD="mads" -v${PWD}:/code domingogallardo/playframework /bin/bash
```

La opción `--link` define el nombre del contenedor con el que enlazar
en el que se está ejecutando MySQL y las opciones `-e` definen los
valores de las variables de entorno que usará el fichero de
configuración. 

Añadimos `/bin/bash` al final para que sea este el
comando que se ejecute en el contenedor, en lugar de `sbt`. Por ello
el comando anterior lanzará un shell:

```
bash-4.3# 
```

Por último, sólo nos falta llamar a `sbt` desde el shell para que
cargue el nuevo fichero de configuración y lance los tests:

```
bash-4.3# sbt '; set javaOptions += "-Dconfig.file=conf/integration.conf"; test'
```

También podríamos hacer un `run` o un `testOnly`:

```
$ sbt '; set javaOptions += "-Dconfig.file=conf/integration.conf"; run'
$ sbt '; set javaOptions += "-Dconfig.file=conf/integration.conf"; testOnly Integration*'
```

Una vez que compruebes que los tests funcionan correctamente con la
base de datos MySQL puedes confirmar los cambios en un nuevo commit
(puedes poner el mensaje `Añadido integration.conf para conexión con
BD Mysql`), sube el nuevo commit.


##### Conexión a la base de datos desde el host #####

Es posible conectarse al servicio en el `hostname` `127.0.0.1` y en el
puerto 3306.  Por ejemplo con el cliente MySQL desde el terminal:

```
$ mysql -h 127.0.0.1 -u root -p
...
mysql> use mads;
Database changed
mysql> show tables;
+--------------------+
| Tables_in_mads     |
+--------------------+
| Tarea              |
| Usuario            |
| hibernate_sequence |
+--------------------+
3 rows in set (0,00 sec)

mysql> quit;
```

O desde alguna herramienta como el _MySQL Workbench_, creando una
conexión como muestra la siguiente imagen:

<img src="imagenes/mysql-workbench.png" width="600px"/>


#### Exportación del esquema de datos ####

Por último, vamos a terminar el _issue_ añadiendo al repositorio un fichero SQL
con el esquema de datos.

Comienza por reiniciar MySQL para que la base de datos quede limpia,
sin los datos de los tests:

```
$ docker container stop play-mysql
$ docker run -d --rm --name play-mysql -e MYSQL_ROOT_PASSWORD=mads -e MYSQL_DATABASE=mads mysql
```

Ejecuta la aplicación usando la configuración `integration.conf` (para
que se inicialicen las tablas de la base de datos):

```
$ sbt '; set javaOptions += "-Dconfig.file=conf/integration.conf"; run'
```

Conéctate a la aplicación desde el host, para que se active JPA y se
inicialicen las tablas con el esquema de datos de JPA. No debes añadir
ningún dato, sólo conectarte a la página de login.

Para volcar el esquema de datos podemos ejecutar el comando
`mysqldump` dentro del contenedor docker `play-mysql`:

```
$ docker exec play-mysql sh -c 'exec mysqldump mads -uroot -pmads' > schema.sql
```

El comando anterior ejecuta el comando `mysqldump` en el contenedor
docker y lo graba el fichero `schema.sql` en el directorio
actual. Edítalo para eliminar el día y la hora, ya que queremos que
sólo aparezca información estrictamente del esquema de la base de
datos.  Copia el fichero `schema.sql` en un nuevo directorio
`sql` en la raíz del proyecto Play.

De esta forma podremos comprobar más adelante las diferencias cuando
modifiquemos la base de datos al evolucionar las entidades de la
aplicación.

El fichero debería ser parecido a este:

**Fichero `sql/schema.sql`**:  

```sql
-- MySQL dump 10.13  Distrib 5.7.19, for Linux (x86_64)
--
-- Host: localhost    Database: mads
-- ------------------------------------------------------
-- Server version	5.7.19

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;

--
-- Table structure for table `Tarea`
--

DROP TABLE IF EXISTS `Tarea`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `Tarea` (
  `id` bigint(20) NOT NULL,
  `titulo` varchar(255) DEFAULT NULL,
  `usuarioId` bigint(20) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `FKepne2t52y8dmn8l9da0dd7l51` (`usuarioId`),
  CONSTRAINT `FKepne2t52y8dmn8l9da0dd7l51` FOREIGN KEY (`usuarioId`) REFERENCES `Usuario` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `Tarea`
--

LOCK TABLES `Tarea` WRITE;
/*!40000 ALTER TABLE `Tarea` DISABLE KEYS */;
/*!40000 ALTER TABLE `Tarea` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `Usuario`
--

DROP TABLE IF EXISTS `Usuario`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `Usuario` (
  `id` bigint(20) NOT NULL,
  `apellidos` varchar(255) DEFAULT NULL,
  `email` varchar(255) DEFAULT NULL,
  `fechaNacimiento` date DEFAULT NULL,
  `login` varchar(255) DEFAULT NULL,
  `nombre` varchar(255) DEFAULT NULL,
  `password` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `Usuario`
--

LOCK TABLES `Usuario` WRITE;
/*!40000 ALTER TABLE `Usuario` DISABLE KEYS */;
/*!40000 ALTER TABLE `Usuario` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `hibernate_sequence`
--

DROP TABLE IF EXISTS `hibernate_sequence`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `hibernate_sequence` (
  `next_val` bigint(20) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `hibernate_sequence`
--

LOCK TABLES `hibernate_sequence` WRITE;
/*!40000 ALTER TABLE `hibernate_sequence` DISABLE KEYS */;
INSERT INTO `hibernate_sequence` VALUES (1),(1);
/*!40000 ALTER TABLE `hibernate_sequence` ENABLE KEYS */;
UNLOCK TABLES;
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;
```

Realiza un último commit en el que se incluya el fichero `schema.sql`
y **cierra el _issue_ con un pull request**.

### Configuración de _stage_ ###

Una configuración de _stage_ es una configuración de despliegue de la
aplicación preparada para que sea lo más similar posible al despliegue
de producción. Se utiliza para realizar las pruebas funcionales y de
rendimiento de la aplicación.

Una de las características fundamentales de la configuración de
_stage_ es que su gestor base de datos debe ser idéntica al gestor de
base de datos de producción, debe tener el mismo esquema de datos y
contener datos similares (en cantidad y complejidad) a los que tiene
la aplicación en producción.

Vamos a definir una configuración de _stage_ para nuestra aplicación
Play. Vamos a usar la misma imagen docker MySQL que en la
configuración de pruebas de integración y una ejecución de nuestra
aplicación Play en modo _stage_.

Lo normal es definir la configuración en un ordenador dedicado, y
tenerla siempre en funcionamiento (de forma similar a como funciona la
aplicación en producción). En lugar de esto, para simplificar el
proceso, vamos a hacer que la imagen MySQL cargue los datos que
guardamos en cada nueva realización de pruebas funcionales. En cada
ejecución de pruebas funcionales se probarán con los datos ya
existentes y se añadirán nuevos datos para realizar nuevas
pruebas. Al final de la sesión de pruebas funcionales grabaremos todos
los datos en un fichero que volveremos a usar como datos iniciales la
próxima vez que arranquemos la configuración de _stage_.

También veremos cómo usar sbt para desplegar y lanzar la aplicación
Play en modo _stage_.

**Nuevo issue: Configuración entorno de stage**

Creamos un nuevo _issue_ llamado `Configuración entorno stage`. Puedes
ponerle la etiqueta `tech`. Crea una rama en la que desarrolles este
nuevo _issue_.

Crea el fichero de `conf/stage.conf`:

```
include "application.conf"

play.crypto.secret="abcdefghijkl"

jpa.default = mySqlPersistenceUnitProduction

db.default.driver=com.mysql.jdbc.Driver
db.default.url=${?DB_URL}
db.default.username=${?DB_USER_NAME}
db.default.password=${?DB_USER_PASSWD}
```

Añade en el fichero `conf/META-INF/persistence.xml` el nuevo perfil
`mySqlPersistenceUnitProduction`. La única diferencia (importante) con
el perfil de integración es que el `hbm2ddl.auto` está definido con el
valor `validate`. Debido a ello, cuando arranquemos la aplicación Play
y se conecte a la base de datos lo primero que va a hacer JPA es
comprobar que el esquema de datos (las tablas definidas en la base de
datos) se corresponde con lo codificado en las entidades. En el caso
en que no exista el esquema de datos o no se corresponda con el
esquema definido por las entidades JPA lanzará un error y no
funcionará el acceso a los datos.


```diff
    </properties>
 </persistence-unit>
 
+<!-- MySQL Persistence Unit - Production -->
+
+<persistence-unit name="mySqlPersistenceUnitProduction" transaction-type="RESOURCE_LOCAL">
+   <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
+   <non-jta-data-source>DBTest</non-jta-data-source>
+   <class>models.Usuario</class>
+   <class>models.Tarea</class>
+   <properties>
+      <property name="hibernate.dialect" value="org.hibernate.dialect.MySQL5Dialect"/>
+      <property name="hibernate.hbm2ddl.auto" value="validate"/>
+   </properties>
+</persistence-unit>
+
 </persistence>
 ```


#### Ejecución de pruebas funcionales en el entorno _stage_ ####

La diferencia de esta configuración con la anterior es que en lugar de
dejar que JPA cree las tablas de la base de datos, vamos nosotros a
inicializar la base de datos MySQL con los esquemas y datos
predefinidos, tal y como sucedería en producción.

##### Lanzamiento de MySQL #####

Para inicializar los datos de la imagen docker MySQL podemos utilizar
su directorio `/docker-entrypoint-initdb.d`. Lo primero que hace la
imagen es consultar ese directorio y ejecutar todos los ficheros con
la extensión `.sql` que encuentre ahí. Los ejecuta en el orden
alfabético del nombre de fichero. 

Vamos entonces a lanzar la configuración de _stage_ por primera
vez. Creamos un directorio `stage` fuera del repositorio git y
copiamos ahí el fichero `schema.sql` del repositorio (el que contiene
el esquema de base de datos actual):

```
$ cd <ruta-directorio-trabajo>
$ mkdir stage
$ cd stage
$ cp <ruta-proyecto-play>/sql/schema.sql .
```

Nos movemos a ese directorio y lanzamos el comando `docker
run` montando el directorio actual en el directorio `/docker-entrypoint-initdb.d`:

```
$ docker run -d --rm --name play-mysql -v ${PWD}:/docker-entrypoint-initdb.d -e MYSQL_ROOT_PASSWORD=mads -e MYSQL_DATABASE=mads mysql
```

De esta forma se lanza MySQL cargando el esquema de datos inicial
(vacío).

##### Parada de MySQL #####

Cuando hayamos terminado de ejecutar las pruebas funcionales volcamos
la base de datos a un fichero y paramos MySQL

```
$ docker exec play-mysql sh -c 'exec mysqldump mads -uroot -pmads' > stage-data.sql
$ docker container stop play-mysql
```

Deberemos guardar este fichero `stage-data.sql` en algún sitio (**no
en el repositorio Git del proyecto**, porque no representan cambios en
el desarrollo del mismo), por ejemplo en algún directorio compartido
de Dropbox o en un USB. De forma que lo podamos recuperar y utilizar
la próxima vez que pongamos en marcha el entorno de stage.


##### Ejecución de la aplicación #####

Para ejecutar la aplicación Play en modo producción lanzamos el
contenedor play de la misma forma que hacíamos para ejecutar los tests
de integración (definiendo las variables de entorno para que la
configuración se conecte con la BD y lanzando el shell bash). Y
después usamos el comando `stage` de sbt que crea un fichero
ejecutable en el directorio `target/universal/bin/<proyecto>`. 

Por último lanzamos el ejecutable generado cargando la configuración
`stage.conf`. 

```
$ cd <ruta-proyecto-play>
$ docker run --link play-mysql:mysql --rm -it -p 80:9000 -e \
DB_URL="jdbc:mysql://play-mysql:3306/mads" -e DB_USER_NAME="root" -e \
DB_USER_PASSWD="mads" -v${PWD}:/code domingogallardo/playframework /bin/bash
bash-4.3# sbt clean stage
bash-4.3# target/universal/stage/bin/mads-todolist-2017 -Dconfig.file=${PWD}/conf/stage.conf
```

El ejecutable que se construye es similar al de producción. Sbt tiene
un comando `dist` que permite generar un generar un binario listo para
su distribución. Puedes consultar cómo hacerlo en [Deploying your
application](https://www.playframework.com/documentation/2.5.x/Deploying).

La aplicación se lanza en modo producción y podemos abrir el navegador
y realizar los tests funcionales.

Verás que aparecen unos errores debido que el funcionamiento de la aplicación en producción
es ligeramente distinto al funcionamiento en desarrollo. En el
repositorio con la guía de la práctica puedes encontrar los cambios a
realizar, en el [PR
#26](https://github.com/domingogallardo/mads-todolist-guia/pull/26):

- Añadir getters y setters.
- Añadir la declaración de las clases de entidad todos los perfiles en el fichero `persistence.xml`.


## Nueva historia de usuario: tableros (usando TDD) ##

La última parte de la práctica consiste en desarrollar, utilizando TDD
(_Test Driven Design_) una nueva historia de usuario:
**Creación y asociación a tableros**.

<kbd><img src="imagenes/historias-usuario.png" width="600px"/></kbd>

Definimos la descripción de la historia que se muestra en la siguiente
imagen:

<kbd><img src="imagenes/historia-tableros.png" width="600px"/></kbd>

Implementaremos cada _issue_ de la capa de modelo o la capa de
servicios utilizando el ciclo de desarrollo de TDD (el código de la
capa de vista y controlador también se debería probar de esta forma,
pero no lo vamos a hacer por falta de tiempo):

1. Escribir un test que falla.
2. Escribir el código que hace que el test deje de fallar.
3. Refactorizar el código de los tests y el código escrito (sin
   añadir nuevos tests, ni nuevas funcionalidades).
4. Volver al paso 1

Un resumen de lo que deberás hacer en esta última parte de la práctica:

- Desarrollarás uno a uno los nuevos _issues_ de la historia de
  usuario.
- Al igual que hicimos en la práctica 1, cada _issue_ debe
  desarrollarse en una rama independiente. Después crearemos un pull
  request y, tras pasar los tests de integración, lo integraremos con
  master (consultar en el apartado XX cómo realizar los tests de
  integración antes de aprobar el pull request).
- Todos los ficheros de tests deberán crearse en un directorio con el
  nombre de la historia de usuario. Por ejemplo
  `sg3-creacion-asociacion-tableros`.
- Definirás un fichero de test por cada _issue_, que contendrá
  todos los tests necesarios para implementar el _issue_ y se
  denominará con un nombre similar.
- Cada test, junto con el código desarrollado para pasarlo, irá en un
  commit.
- Es posible hacer commits con refactorizaciones (recuerda el ciclo de
  TDD: Test, Codigo y Refactorización).
- Utilizarás DBUnit para poder hacer pruebas con datos
  iniciales, añadiendo nuevos datos en el fichero `usuarios_dataset.xml`.

Veamos a continuación como ejemplo el desarrollo completo del primer
_issue_.


### 4.1 Primer _issue_: Modelo de tablero y repositorio básico

En este _issue_ completaremos una clase básica de entidad `Tablero`
con los atributos:

- Nombre
- Descripción
- Fecha de creación
- Administrador (relación a-uno con la entidad `Usuario`)

También crearemos la clase `TableroRepository` con un primer método
para crear tableros. 

Para implementar esta característica usando TDD crearemos el fichero
de test
`test/sg3-creacion-asociacion-tableros/ModeloRepositorioTableroTest.java`
en el que iremos añadiendo los
distintos tests que irán construyendo el código.

Abrimos una rama en la que iremos añadiendo los commits del _issue_
(uno por cada test). 

```
$ git checkout -b modelo-tablero
```

Empezamos ahora a usar TDD para implementar el _issue_. El
primer test nos servirá para definir los elementos básicos de la
entidad `Tablero`.

#### Primer test ####

Empezamos con el test más sencillo: crear un tablero. Para crear un
tablero necesitaremos pasar un usuario y un nombre de tablero.

**Fichero `test/sg3-creacion-asociacion-tableros/ModeloRepositorioTableroTest.java`**:

```java
import org.junit.*;
import static org.junit.Assert.*;

import models.Usuario;
import models.Tablero;

public class ModeloRepositorioTableroTest {

   @Test
   public void testCrearTablero() {
      Usuario usuario = new Usuario("juangutierrez", "juangutierrez@gmail.com");
      Tablero tablero = new Tablero(usuario, "Tablero 1");

      assertEquals("juangutierrez", tablero.getAdministrador().getLogin());
      assertEquals("juangutierrez@gmail.com", tablero.getAdministrador().getEmail());
      assertEquals("Tablero 1", tablero.getNombre());
   }
}
```

Lanzamos los tests y, evidentemente, obtendremos errores de
compilación porque no existen las clases:

```
[mads-todolist-2017] $ testOnly
[info] Compiling 6 Java sources to /code/target/scala-2.11/test-classes...
[error] /code/test/models/TableroTest.java:5: cannot find symbol
[error]   symbol:   class Tablero
[error]   location: package models
...
[error] Total time: 15 s, completed Sep 30, 2017 3:30:33 PM
```

Creamos las clases con **solo el código necesario para que el test pase**:

```java
import org.junit.*;
import static org.junit.Assert.*;

import models.Usuario;
import models.Tablero;

public class ModeloRepositorioTableroTest {

   @Test
   public void testCrearTablero() {
      Usuario usuario = new Usuario("juangutierrez", "juangutierrez@gmail.com");
      Tablero tablero = new Tablero(usuario, "Tablero 1");

      assertEquals("juangutierrez", tablero.getAdministrador().getLogin());
      assertEquals("juangutierrez@gmail.com", tablero.getAdministrador().getEmail());
      assertEquals("Tablero 1", tablero.getNombre());
   }
}
```

Hacemos un commit con el test y el código que hemos creado

```
$ git add *
$ git status
$ git commit -m "Creada clase Tablero"
```

#### Segundo test ####

Vamos con un segundo test en el que añadimos otro pequeño incremento:
creación del `JPATableroRepository` y comprobación de que se obtiene
correctamente un objeto `TableroRepository`.

Escribimos el test, añadiendo el código al fichero existente:

```diff
 import org.junit.*;
 import static org.junit.Assert.*;
 
+import play.inject.guice.GuiceApplicationBuilder;
+import play.inject.Injector;
+import play.inject.guice.GuiceInjectorBuilder;
+import play.Environment;
+
+import play.db.jpa.*;
+
 import models.Usuario;
 import models.Tablero;
+import models.TableroRepository;
 
 public class ModeloRepositorioTableroTest {
+   static private Injector injector;
+
+   @BeforeClass
+   static public void initApplication() {
+      GuiceApplicationBuilder guiceApplicationBuilder =
+          new GuiceApplicationBuilder().in(Environment.simple());
+      injector = guiceApplicationBuilder.injector();
+      // Necesario para inicializar JPA
+      injector.instanceOf(JPAApi.class);
+   }
 
    @Test
    public void testCrearTablero() {
@@ -15,4 +33,10 @@ public class ModeloRepositorioTableroTest {
       assertEquals("juangutierrez@gmail.com", tablero.getAdministrador().getEmail());
       assertEquals("Tablero 1", tablero.getNombre());
    }
+
+   @Test
+   public void testObtenerTableroRepository() {
+      TableroRepository tableroRepository = injector.instanceOf(TableroRepository.class);
+      assertNotNull(tableroRepository);
+   }
 }
```


Lanzamos el test y comprobamos que no funciona. Escribimos el código
mínimo para hacer que funcione:

**Fichero `models/TableroRepository.java`**:

```java
package models;

import javax.inject.Inject;
import play.db.jpa.JPAApi;

import java.util.List;

import javax.persistence.EntityManager;

public class TableroRepository {
   JPAApi jpaApi;

   @Inject
   public TableroRepository(JPAApi api) {
      this.jpaApi = api;
   }
}
```

Comprobamos que el test funciona:

```
[mads-todolist-2017] $ testOnly ModeloRepositorio*
[info] Test run started
...
[info] Test ModeloRepositorioTableroTest.testObtenerTableroRepository started
[info] Test run finished: 0 failed, 0 ignored, 2 total, 12.975s
[info] Passed: Total 2, Failed 0, Errors 0, Passed 2
```

Una vez que el test pasa correctamente **hacemos una refactorización**,
sin tocar el test, para convertir `TableroRepository` en una interfaz
y definir una clase específica de la interfaz: `JPATableroRepository`:


**Fichero `models/TableroRepository.java`**:

```java
package models;

import com.google.inject.ImplementedBy;

import java.util.List;

@ImplementedBy(JPATableroRepository.class)
public interface TableroRepository {
}
```


**Fichero `models/JPATableroRepository.java`**:

```java
package models;

import javax.inject.Inject;
import play.db.jpa.JPAApi;

import java.util.List;

import javax.persistence.EntityManager;

public class JPATableroRepository implements TableroRepository {
   JPAApi jpaApi;

   @Inject
   public JPATableroRepository(JPAApi api) {
      this.jpaApi = api;
   }
}
```

Terminamos haciendo el commit del segundo test:


```
$ git status
$ git add *
$ git commit -m "Creado TableroRepository"
```

#### Tercer test ####

En el tercer test vamos a comprobar si se crea la tabla `Tablero` en
la base de datos. Fallará y escribiremos el código para que pase.

El test consistirá en usar DbUnit para intentar cargar en la base de
datos una nueva fila añadida en el fichero `usuarios_dataset.xml`

**Cambios en el fichero `test/resources/usuarios_dataset.xml`**:

```diff
          password="123456789" eMail="juan.gutierrez@gmail.com" fechaNacimiento="1993-12-10"/>
     <Tarea id="1000" titulo="Renovar DNI" usuarioId="1000"/>
     <Tarea id="1001" titulo="Práctica 1 MADS" usuarioId="1000"/>
+    <Tablero id="1000" nombre="Tablero 1" administradorId="1000"/>
  </dataset>

```


**Cambios en el fichero `test/sg3-creacion-asociacion-tableros/ModeloRepositorioTableroTest.java`**:

```diff
import play.Environment;
 
import play.db.jpa.*;
 
+import org.dbunit.*;
+import org.dbunit.dataset.*;
+import org.dbunit.dataset.xml.*;
+import org.dbunit.operation.*;
+import java.io.FileInputStream;
+
 import models.Usuario;
 import models.Tablero;
 import models.TableroRepository;

...

 public class ModeloRepositorioTableroTest {
       TableroRepository tableroRepository = injector.instanceOf(TableroRepository.class);
       assertNotNull(tableroRepository);
    }
+
+   @Test
+   public void testCrearTablaTableroEnBD() throws Exception {
+      JndiDatabaseTester databaseTester = new JndiDatabaseTester("DBTest");
+      IDataSet initialDataSet = new FlatXmlDataSetBuilder().build(new FileInputStream("test/resources/usuarios_dataset.xml"));
+      databaseTester.setDataSet(initialDataSet);
+      databaseTester.setSetUpOperation(DatabaseOperation.CLEAN_INSERT);
+      databaseTester.onSetup();
+      assertTrue(true);
+   }
 }
```


El test fallará porque la tabla `Tablero` no se encuentra. Debemos
escribir el código que corrige esto:

**Cambios en el fichero `models/Tablero.java`:

```diff

+import javax.persistence.*;
+
+@Entity
 public class Tablero {
+   @Id
+   @GeneratedValue(strategy=GenerationType.AUTO)
+   Long id;
    private String nombre;
+   @ManyToOne
+   @JoinColumn(name="administradorId")
    private Usuario administrador;
 
+   public Tablero() {}
+
    public Tablero(Usuario administrador, String nombre) {
       this.nombre = nombre;
       this.administrador = administrador;
    }
 
+   public Long getId() {
+      return id;
+   }
+
+   public void setId(Long id) {
+      this.id = id;
+   }
+
    public String getNombre() {
       return nombre;
    }
 
+   public void setNombre(String nombre) {
+      this.nombre = nombre;
+   }
+
    public Usuario getAdministrador() {
       return administrador;
    }
+
+   public void setAdministrador(Usuario usuario) {
+      this.administrador = administrador;
+   }
 }
 ```


**Cambios en el fichero `conf/META-INF/persistence.xml`**:

```diff
       <non-jta-data-source>DBTest</non-jta-data-source>
       <class>models.Usuario</class>
       <class>models.Tarea</class>
+      <class>models.Tablero</class>
       <properties>
          <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
          <property name="hibernate.hbm2ddl.auto" value="update"/>
...
    <non-jta-data-source>DBTest</non-jta-data-source>
    <class>models.Usuario</class>
    <class>models.Tarea</class>
+   <class>models.Tablero</class>
    <properties>
         <property name="hibernate.dialect" value="org.hibernate.dialect.MySQL5Dialect"/>
         <property name="hibernate.hbm2ddl.auto" value="update"/>
...
    <non-jta-data-source>DBTest</non-jta-data-source>
    <class>models.Usuario</class>
    <class>models.Tarea</class>
+   <class>models.Tablero</class>
    <properties>
       <property name="hibernate.dialect" value="org.hibernate.dialect.MySQL5Dialect"/>
       <property name="hibernate.hbm2ddl.auto" value="validate"/>
```

Realiza los cambios, reflexionando sobre lo que hacen, y comprueba que
el test pasa.

Ya puedes cerrar el commit, llamándolo por ejemplo `Añadida entidad
Tablero`:

```
$ git add *
$ git commit -m "Añadida entidad Tablero"
```


####  Cuarto test ####

Este cuarto test va a servir para crear la función `add()` en el
`TableroRepository`.

Añadimos el siguiente test:

```diff
 import org.dbunit.dataset.xml.*;
 import org.dbunit.operation.*;
 import java.io.FileInputStream;
 
+import play.db.Database;
+import play.db.Databases;
+
+import java.sql.*;
+
 import models.Usuario;
 import models.Tablero;
 import models.TableroRepository;
+import models.UsuarioRepository;
 
 public class ModeloRepositorioTableroTest {
    static private Injector injector;
+   static Database db;
 
    @BeforeClass
    static public void initApplication() {
...
   public class ModeloRepositorioTableroTest {
       injector = guiceApplicationBuilder.injector();
       // Necesario para inicializar JPA
       injector.instanceOf(JPAApi.class);
+      db = injector.instanceOf(Database.class);
    }
 
    @Test
...
    public class ModeloRepositorioTableroTest {
       assertTrue(true);
    }
 
+   @Test
+   public void testAddTableroInsertsDatabase() {
+      UsuarioRepository usuarioRepository = injector.instanceOf(UsuarioRepository.class);
+      TableroRepository tableroRepository = injector.instanceOf(TableroRepository.class);
+      Usuario administrador = new Usuario("juangutierrez", "juangutierrez@gmail.com");
+      administrador = usuarioRepository.add(administrador);
+      Tablero tablero = new Tablero(administrador, "Tablero 1");
+      tablero = tableroRepository.add(tablero);
+      assertNotNull(tablero.getId());
+      assertEquals("Tablero 1", getNombreFromTableroDB(tablero.getId()));
+   }
 
+   private String getNombreFromTableroDB(Long tableroId) {
+      String nombre = db.withConnection(connection -> {
+         String selectStatement = "SELECT Nombre FROM Tablero WHERE ID = ? ";
+         PreparedStatement prepStmt = connection.prepareStatement(selectStatement);
+         prepStmt.setLong(1, tableroId);
+         ResultSet rs = prepStmt.executeQuery();
+         rs.next();
+         return rs.getString("Nombre");
+      });
+      return nombre;
+   }
 }
```

El test fallará. Y escribimos el código para que pase:

**Fichero `models/JPATableroRepository.java`**:

```diff
public class JPATableroRepository implements TableroRepository {
    public JPATableroRepository(JPAApi api) {
       this.jpaApi = api;
    }
+
+   public Tablero add(Tablero tablero) {
+      return jpaApi.withTransaction(entityManager -> {
+         entityManager.persist(tablero);
+         entityManager.flush();
+         entityManager.refresh(tablero);
+         return tablero;
+      });
+   }
 }
```


**Fichero `models/TableroRepository.java`**:

```diff
 @ImplementedBy(JPATableroRepository.class)
 public interface TableroRepository {
+   public Tablero add(Tablero tablero);
 }
```


Comprueba que el test pasa y realiza un nuevo commit:

```
$ git add *
$ git commit -m "Añadido método add() en TableroRepository"
```

#### Quinto y último test ####

Vamos a por el siguiente test, en el que haremos posible que un
usuario pueda administrar varios tableros.


```diff

import java.sql.*;

+ import java.util.List;

import models.Usuario;
import models.Tablero;

...

    }
+
+   @Test
+   public void testUsuarioAdministraVariosTableros() {
+      UsuarioRepository usuarioRepository = injector.instanceOf(UsuarioRepository.class);
+      TableroRepository tableroRepository = injector.instanceOf(TableroRepository.class);
+      Usuario administrador = new Usuario("juangutierrez", "juangutierrez@gmail.com");
+      administrador = usuarioRepository.add(administrador);
+      Tablero tablero1 = new Tablero(administrador, "Tablero 1");
+      tableroRepository.add(tablero1);
+      Tablero tablero2 = new Tablero(administrador, "Tablero 2");
+      tableroRepository.add(tablero2);
+      List<Tablero> tableros = tableroRepository.getAdministrados(administrador.getId());
+      assertEquals(2, tableros.size());
+   }
 }
```


Y añadimos el código para conseguir que pase:

**Fichero `models/Usuario.java`**:

```diff
public class Usuario {
    // Relación uno-a-muchos entre usuario y tarea
    @OneToMany(mappedBy="usuario")
    public List<Tarea> tareas = new ArrayList<Tarea>();
+   @OneToMany(mappedBy="administrador")
+   public List<Tablero> administrados = new ArrayList<Tablero>();
 
    // Un constructor vacío necesario para JPA
    public Usuario() {}
...
    public class Usuario {
       this.tareas = tareas;
    }
 
+   public List<Tablero> getAdministrados() {
+      return administrados;
+   }
+
+   public void setAdministrados(List<Tablero> administrados) {
+      this.administrados = administrados;
+   }
+
```

**Fichero `models/TableroRepository.java`**:

```diff
import java.util.List;
 @ImplementedBy(JPATableroRepository.class)
 public interface TableroRepository {
    public Tablero add(Tablero tablero);
+   public List<Tablero> getAdministrados(Long idUsuario);
 }
```

**Fichero `models/JPATableroRepository.java`**:

```diff
    public class JPATableroRepository implements TableroRepository {
          return tablero;
       });
    }
+
+   public List<Tablero> getAdministrados(Long idUsuario) {
+      return jpaApi.withTransaction(entityManager -> {
+         Usuario usuario = entityManager.find(Usuario.class, idUsuario);
+         // Cargamos todas las tareas del usuario en memoria
+         usuario.getAdministrados().size();
+         return usuario.getAdministrados();
+      });
+   }
+
 }
```

Una vez que compruebes que el test funciona correctamente, debes
confirmar los cambios:

```
$ git add *
$ git commit -m "Un usuario puede administrar varios tableros"
```


### Tests de integración antes de cerrar el _issue_ y confirmar el pull request ###













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
import org.dbunit.operation.*;
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
        FileInputStream("test/resources/usuarios_dataset.xml"));
        databaseTester.setDataSet(initialDataSet);

        // Definimos como operación SetUp CLEAN_INSERT, que hace un
        // DELETE_ALL de todas las tablase del dataset, seguido por un
        // INSERT. (http://dbunit.sourceforge.net/components.html)
        // Es lo que hace DbUnit por defecto, pero así queda más claro.
        databaseTester.setSetUpOperation(DatabaseOperation.CLEAN_INSERT);

        // Definimos como operación TearDown DELETE_ALL para que se
        // borren todos los datos de las tablas del dataset
        // (el valor por defecto DbUnit es DatabaseOperation.NONE)
        databaseTester.setTearDownOperation(DatabaseOperation.DELETE_ALL);

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
- La calificación de la práctica tiene un peso de un 5% en la nota
  final de la asignatura.
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

