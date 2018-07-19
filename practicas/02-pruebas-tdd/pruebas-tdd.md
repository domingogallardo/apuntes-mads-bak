# Práctica 2: Gestión de configuraciones y TDD con Play Framework

- [1. Objetivos](#1-objetivos)
  - [1.2. Refactorización de la obtención de las tareas de usuario](#12-refactorización-de-la-obtención-de-las-tareas-de-usuario)
- [2. Tests en Play Framework](#2-tests-en-play-framework)
  - [2.1. JUnit](#21-junit)
  - [2.2. Tests con bases de datos y JPA](#22-tests-con-bases-de-datos-y-jpa)
  - [2.3. Nuevo _issue_ con tests](#23-nuevo-issue-con-tests)
- [3. Definición de configuraciones](#3-definición-de-configuraciones)
  - [3.1. Configuración de pruebas de integración](#31-configuración-de-pruebas-de-integración)
  - [3.2. Configuración de _stage_](#32-configuración-de-stage)
- [4. Nueva historia de usuario: tableros (usando TDD)](#4-nueva-historia-de-usuario-tableros-usando-tdd)
  - [4.1. Primer _issue_: Modelo de tablero y repositorio básico](#41-primer-issue-modelo-de-tablero-y-repositorio-básico)
  - [4.2. Tests de integración antes de cerrar el _issue_ y confirmar el pull request](#42-tests-de-integración-antes-de-cerrar-el-issue-y-confirmar-el-pull-request)
  - [4.3. Resto de _issues_ (parte
    obligatoria)](#43-resto-de-issues-parte-obligatoria)
  - [4.4 Resto de _issues_ (parte
    opcional)](#44-resto-de-issues-parte-opcional)
  - [4.5. Finalización de la versión 0.2](#45-finalización-de-la-versión-02)
- [5. Entrega y evaluación](#5-entrega-y-evaluación)
    

## 1. Objetivos

Esta práctica tiene dos objetivos principales: profundizar en las
características de Play Framework relacionadas con las pruebas y
utilizar la metodología TDD para añadir funcionalidades a nuestra
aplicación.

Al igual que la primera práctica, el desarrollo de esta práctica será
individual. Tendrá una duración de 3 semanas, siendo la fecha límite
de entrega el **24 de octubre**.

Seguiremos trabajando en la aplicación `mads-todolist` que has estado
desarrollando en la primera práctica.

Continuaremos también trabajando con Git como sistema de control de
versiones y GitHub como repositorio remoto. Intregraremos Git y TDD,
haciendo que cada commit represente un incremento funcional en el
desarrollo de la aplicación y contenga y pase sus propios tests.

### 1.2. Refactorización de la obtención de las tareas de usuario ###

Antes de empezar la práctica, vamos a hacer una refactorización de un
código mejorable introducido en la práctica 1. Se trata del código que
recupera la lista de tareas de un usuario. 

La podéis encontrar en el [PR
#34](https://github.com/domingogallardo/mads-todolist-guia/pull/34),
echadle un vistazo a los cambios y hacedlo también en vuestro
proyecto.

Un usuario tiene una relación a-muchos con tareas. La refactorización
consiste en hacer que al recuperar un usuario, JPA se traiga a memoria
todas sus tareas y se queden guardadas en su atributo `tareas`. Para
ello basta con poner en la entidad el atributo de JPA `fetch=FetchType.EAGER`.

De esta forma evitamos el método `findAllTareas(usuarioId)` en el
`TareaRepository`.

En términos de [Domain-Driven
Design](https://stackoverflow.com/questions/1222392/can-someone-explain-domain-driven-design-ddd-in-plain-english-please/1222488#1222488)
(un método de diseño muy interesante, que os recomiendo que aprendáis)
estamos hablando de que `Usuario` y `Tareas` son **entidades
agregadas**. Siempre tendremos en memoria en los objetos usuario su
lista completa de tareas. 

Un libro muy recomendable para aprender sobre DDD es [Patterns,
Principles, and Practices of Domain-Driven
Design](https://www.amazon.com/Patterns-Principles-Practices-Domain-Driven-Design/dp/1118714709).


## 2. Tests en Play Framework

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

### 2.1. JUnit

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


### 2.2. Tests con bases de datos y JPA

#### Conexión de JPA con una base de datos en memoria

Lo habitual para realizar tests unitarios que necesiten trabajar con
una base de datos es, o bien _mockear_ la base de datos, o bien
utilizar una base de datos en memoria (como H2) para que los tests
puedan ejecutarse mucho más rápido sin necesidad de una base de datos
real que trabaje sobre el disco duro.

En la primera práctica hemos utilizado el segundo enfoque.


El problema de los tests tal y como están escritos en la primera
práctica es que la base de datos de prueba sobre la que trabajan no se
puede definir en el fichero de configuración, sino que está definida
en su código fuente.

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

### 2.3. Nuevo _issue_ con tests ###

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


## 3. Definición de configuraciones 

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


### 3.1. Configuración de pruebas de integración ###

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
obteniendo la JPAApi mediante la inyección de dependencias. 

El problema es que las anotaciones `@inject` no funcionan en los
tests. Los tests son programas Java independientes que se ejecutan al
margen de la aplicación principal, que es donde se ejecuta el código
que inicializa los objetos inyectados. 

Tenemos nosotros que obtener a mano los objetos llamando
explícitamente a la librería `Guice` que es la que gestiona la
inyección de dependencias. A continuación vemos cómo hacerlo en un
test concreto, por ejemplo en el fichero `UsuarioServiceTest.java`:


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
        <non-jta-data-source>DBTest</non-jta-data-source>
        <properties>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
            <property name="hibernate.hbm2ddl.auto" value="update"/>
        </properties>
    </persistence-unit>

    <!-- MySQL Persistence Unit -->

    <persistence-unit name="mySqlPersistenceUnit" transaction-type="RESOURCE_LOCAL">
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <non-jta-data-source>DBTest</non-jta-data-source>
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
DB_USER_PASSWD="mads" -v "${PWD}:/code" domingogallardo/playframework /bin/bash
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

Una vez que hayas lanzado los tests JPA habrá creado el esquema de
datos en la base de datos MySQL que está corriendo en el contenedor `play-mysql`.

Podemos entonces ejecutar el comando `mysqldump` dentro del contenedor:

```
$ docker exec play-mysql sh -c 'exec mysqldump --no-data mads -uroot -pmads' > schema.sql
```

El comando anterior ejecuta el comando `mysqldump` en el contenedor
docker que exporta el esquema de datos (sin los datos añadidos) y lo
graba el fichero `schema.sql` en el directorio actual. Edítalo para
eliminar el día y la hora, ya que queremos que sólo aparezca
información estrictamente del esquema de la base de datos. 

Copia el fichero `schema.sql` en un nuevo directorio `sql` en la raíz
del proyecto Play. De esta forma podremos comprobar más adelante las
diferencias cuando modifiquemos la base de datos al evolucionar las
entidades de la aplicación.

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
-- Table structure for table `hibernate_sequence`
--

DROP TABLE IF EXISTS `hibernate_sequence`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `hibernate_sequence` (
  `next_val` bigint(20) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
/*!40101 SET character_set_client = @saved_cs_client */;

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;
```

Añade al final del fichero la inicialización de la tabla de secuencias de
Hibernate, necesaria para que Hibernate genere las claves primarias de
las entidades:

```sql
--
-- Dumping data for table `hibernate_sequence`
--

LOCK TABLES `hibernate_sequence` WRITE;
/*!40000 ALTER TABLE `hibernate_sequence` DISABLE KEYS */;
INSERT INTO `hibernate_sequence` VALUES (1),(1);
/*!40000 ALTER TABLE `hibernate_sequence` ENABLE KEYS */;
UNLOCK TABLES;
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;
```

Realiza un último commit en el que se incluya el fichero `schema.sql`
y **cierra el _issue_ con un pull request**.

### 3.2. Configuración de _stage_ ###

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

Nos movemos a ese directorio y lanzamos el comando `docker run` 
montando el directorio actual en el directorio `/docker-entrypoint-initdb.d`:

```
$ docker run -d --rm --name play-mysql -v ${PWD}:/docker-entrypoint-initdb.d -e MYSQL_ROOT_PASSWORD=mads -e MYSQL_DATABASE=mads mysql
```

De esta forma se lanza MySQL cargando el esquema de datos inicial
(vacío).

##### Parada de MySQL #####

Cuando hayamos terminado de ejecutar las pruebas funcionales volcamos
la base de datos a un fichero y paramos MySQL. Para simplificar
podemos volcarlo para que sobreescriba el anterior fichero `schema.sql`:

```
$ docker exec play-mysql sh -c 'exec mysqldump mads -uroot -pmads' > schema.sql
$ docker container stop play-mysql
```

Dejamos este fichero `schema.sql` en el directorio actual (`/stage`)
para que la próxima vez que lancemos MySQL desde él cargue los datos
anteriormente salvados.

De esta forma podemos simular que tenemos un servidor de stage
en el que se mantienen los datos que se introducen en las pruebas.

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

Arregla los errores, realiza un último commit y **cierra el _issue_ con un pull request**.

## 4. Nueva historia de usuario: tableros (usando TDD) ##

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
  master (consultar en el apartado 3.1 cómo realizar los tests de
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


### 4.1. Primer _issue_: Modelo de tablero y repositorio básico

En este _issue_ completaremos una clase básica de entidad `Tablero`
con los atributos:

- Nombre
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

#### Commit 1: Primer test ####

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
$ git add .
$ git status
$ git commit -m "Creada clase Tablero"
```

#### Commit 2: Segundo test ####

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
...
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
$ git add .
$ git commit -m "Creado TableroRepository"
```

#### Commit 3: Tercer test ####

En el tercer test vamos a comprobar si se crea la tabla `TABLERO` en
la base de datos. Fallará y escribiremos el código para que pase.

El test consistirá en buscar en los metadatos de la base de datos si
existe esa tabla.

**Cambios en el fichero `test/sg3-creacion-asociacion-tableros/ModeloRepositorioTableroTest.java`**:

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
...
+
+   @Test
+   public void testCrearTablaTableroEnBD() throws Exception {
+      Database db = injector.instanceOf(Database.class);
+      Connection connection = db.getConnection();
+      DatabaseMetaData meta = connection.getMetaData();
+      ResultSet res = meta.getTables(null, null, "TABLERO", new String[] {"TABLE"});
+      assertTrue(res.next());
+   }
 }
```


El test fallará porque la tabla `TABLERO` no se encuentra. Modificamos
entonces el código para solucionarlo, añadiendo las anotaciones JPA
a la clase `Tablero`:

**Cambios en el fichero `models/Tablero.java`**:

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

Y en el fichero `persistence.xml` debemos añadir la nueva clase entidad:

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
$ git add .
$ git commit -m "Añadida entidad Tablero"
```


####  Commit 4: Cuarto test ####

Este cuarto test va a servir para crear la función `add()` en el
`TableroRepository`.

Añadimos el siguiente test:

```diff
 import models.TableroRepository;
+import models.UsuarioRepository;
 
 public class ModeloRepositorioTableroTest {
    static private Injector injector;
 
...
 
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
+      Database db = injector.instanceOf(Database.class);
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


Comprueba que el test pasa.

Si lanzas todos los tests, verás que algunos fallan. Esto es debido a
que el test anterior introduce en la base de datos un usuario nuevo y
un tablero con una clave ajena que lo referencia. Cuando DbUnit
intenta limpiar la tabla de usuarios en otros tests se provoca un
error _Referential integrity constraint violation_ porque existe un
tablero que se quedaría sin usuario administrador.

La forma más fácil de solucionarlo es añadir al data set de DbUnit un
elemento `<Tablero/>` para provocar que se borren todos los datos
también de la tabla `Tablero`:

**Fichero `test/resources/usuarios_dataset.xml`**:

```diff
 <dataset>
     <Usuario id="1000" login="juangutierrez" nombre="Juan" apellidos="Gutierrez"
          password="123456789" eMail="juan.gutierrez@gmail.com" fechaNacimiento="1993-12-10"/>
     <Tarea id="1000" titulo="Renovar DNI" usuarioId="1000"/>
     <Tarea id="1001" titulo="Práctica 1 MADS" usuarioId="1000"/>
+    <Tablero/>
  </dataset>
```

Por último, refactoriza el código para eliminar duplicidad de la
inicialización de la variable `db` en dos tests. Mueve esa
inicialización al método `@BeforeClass`.

Realiza un nuevo commit:

```
$ git add .
$ git commit -m "Añadido método add() en TableroRepository"
```

#### Commit 5: Refactorización ####

Vamos ahora a hacer un commit en el que vamos a refactorizar el código
para arreglar un par de problemas. Los tests nos servirán para
comprobar que no se introduce ningún error.

##### Transformación `List` en `Set` #####

En primer lugar vamos a arreglar un error que hemos cometido al
definir las entidades. Realiza una refactorización en la que deberás
convertir el tipo del atributo `tareas` de `Usuario` de tipo `List` a
tipo `Set`.

Deberá quedar como sigue:

**Fichero `models/Usuario.java`**:

```diff
- import java.util.List;
- import java.util.ArrayList;
+ import java.util.Set;
+ import java.util.HashSet;

@Entity
public class Usuario {
   ...
   private Date fechaNacimiento;
   // Relación uno-a-muchos entre usuario y tarea
   @OneToMany(mappedBy="usuario", fetch=FetchType.EAGER)
-  public List<Tarea> tareas = new <Tarea>();
+  public Set<Tarea> tareas = new HashSet<Tarea>();
   ...
```

La razón de la refactorización es que es más correcto definir las
colecciones de los atributos de las entidades del tipo `Set` porque
eso garantiza que no habrá elementos repetidos en las
colecciones. 

Realiza todos los cambios necesarios y asegúrate de que se siguen
pasando todos los tests.

##### Conversión de `Long` a `long` #####

Para corregir otro error que no han detectado los tests debemos
realizar la siguiente refactorización: en los métodos `equals` de
`Tarea` y `Usuario` hacer una conversión de `Long` a `long`, para que
realmente detecte como iguales dos identificadores iguales (el
método `==` compara referencias). 

Para comprobar que el código tenía el error modificamos el test
`testEqualsTareasConId` del fichero `TareaTest.java`:

**Fichero `test/models/TareaTest.java`**:

```diff
   // Test #14: testEqualsTareasConId
   @Test
   public void testEqualsTareasConId() {
      Usuario usuario = new Usuario("juangutierrez", "juangutierrez@gmail.com");
      Tarea tarea1 = new Tarea(usuario, "Práctica 1 de MADS");
      Tarea tarea2 = new Tarea(usuario, "Renovar DNI");
      Tarea tarea3 = new Tarea(usuario, "Pagar el alquiler");
-     tarea1.setId(1L);
-     tarea2.setId(1L);
+     tarea1.setId(1000L);
+     tarea2.setId(1000L);
      tarea3.setId(2L);
      assertEquals(tarea1, tarea2);
      assertNotEquals(tarea1, tarea3);
   }
```

Si lanzamos el test comprobaremos que dos tareas que deberían ser
iguales porque tienen el mismo identificador, son detectadas como
distintas.

Para conseguir hacer pasar el test debemos cambiar en el modelo
`Tarea.java` el siguiente código del método equals:

```diff
      // Si tenemos los ID, comparamos por ID
      if (id != null && other.id != null)
-     return (id == other.id);
+     return ((long) id == (long) other.id);
      // sino comparamos por campos obligatorios      
```

Comprobamos que el test ahora sí que pasa correctamente.

Hacemos el mismo cambio en la clase `Usuario.java`, que contiene el
mismo error (aunque no lo hay ningún test que lo compruebe) y añadimos
el commit:

```
$ git add .
$ git commit -m "Refactorizados List y equals"
```


#### Commit 6: Quinto test ####

Vamos a por el siguiente test, en el que haremos posible que un
usuario pueda administrar varios tableros.


```diff
    }
       });
       return nombre;
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
+      // Recuperamos el administrador del repository
+      administrador = usuarioRepository.findById(administrador.getId());
+      // Y comprobamos si tiene los tableros
+      assertEquals(2, administrador.getAdministrados().size());
+   }
 }
```


Y añadimos el código para conseguir que pase:

**Fichero `models/Usuario.java`**:

```diff
public class Usuario {
    // Relación uno-a-muchos entre usuario y tarea
    @OneToMany(mappedBy="usuario", fetch=FetchType.EAGER)
    private Set<Tarea> tareas = new HashSet<Tarea>();
+   @OneToMany(mappedBy="administrador", fetch=FetchType.EAGER)
+   private Set<Tablero> administrados = new HashSet<Tablero>();
 
    // Un constructor vacío necesario para JPA
    public Usuario() {}
...
    public class Usuario {
       this.tareas = tareas;
    }
 
+   public Set<Tablero> getAdministrados() {
+      return administrados;
+   }
+
+   public void setAdministrados(Set<Tablero> administrados) {
+      this.administrados = administrados;
+   }
+
```

Una vez que compruebes que el test funciona correctamente, debes
confirmar los cambios:

```
$ git add .
$ git commit -m "Un usuario puede administrar varios tableros"
```


#### Commit 7: Sexto y último test ####

Vamos por último a añadir la funcionalidad de que los usuarios puedan
participar en tableros y los tableros tienen varios participantes
(además del administrador).

Empezamos con el siguiente test, en el que usamos DbUnit, para probar
la posibilidad de que un usuario pueda tener varios tableros
asociados:

```diff

import java.sql.*;
  		  
+import java.util.Set;
+
+import play.db.jpa.*;
+
+import org.dbunit.*;
+import org.dbunit.dataset.*;
+import org.dbunit.dataset.xml.*;
+import org.dbunit.operation.*;
+import java.io.FileInputStream;

...
   
+
+   private void initDataSet() throws Exception {
+      JndiDatabaseTester databaseTester = new JndiDatabaseTester("DBTest");
+      IDataSet initialDataSet = new FlatXmlDataSetBuilder().build(new FileInputStream("test/resources/usuarios_dataset.xml"));
+      databaseTester.setDataSet(initialDataSet);
+      databaseTester.setSetUpOperation(DatabaseOperation.CLEAN_INSERT);
+      databaseTester.onSetup();
+   }
+
+   @Test
+   public void testUsuarioParticipaEnVariosTableros() throws Exception {
+      initDataSet();
+      UsuarioRepository usuarioRepository = injector.instanceOf(UsuarioRepository.class);
+      TableroRepository tableroRepository = injector.instanceOf(TableroRepository.class);
+      Usuario admin = usuarioRepository.findById(1000L);
+      Usuario usuario = usuarioRepository.findById(1001L);
+      Set<Tablero> tableros = admin.getAdministrados();
+      // Tras cargar los datos del dataset el usuario2 no tiene ningún
+      // tablero asociado y el usuario 1 tiene 2 tableros administrados
+      assertEquals(0, usuario.getTableros().size());
+      assertEquals(2, tableros.size());
+      for (Tablero tablero : tableros) {
+         // Actualizamos la relación en memoria, añadiendo el usuario
+         // al tablero
+         tablero.getParticipantes().add(usuario);
+         // Actualizamos la base de datos llamando al repository
+         tableroRepository.update(tablero);
+      }
+      // Comprobamos que se ha actualizado la relación en la BD y
+      // el usuario pertenece a los dos tableros a los que le hemos añadido
+      usuario = usuarioRepository.findById(1001L);
+      Set<Tablero> tablerosUsuario = usuario.getTableros();
+      assertEquals(2, tablerosUsuario.size());
+      for (Tablero tablero: tableros) {
+         assertTrue(tablerosUsuario.contains(tablero));
+      }
+   }
```

Añadimos en el data set de DbUnit lo siguiente:

```diff
 <dataset>
     <Usuario id="1000" login="juangutierrez" nombre="Juan" apellidos="Gutierrez"
          password="123456789" eMail="juan.gutierrez@gmail.com" fechaNacimiento="1993-12-10"/>
+    <Usuario id="1001" login="juangutierrez2" nombre="Juan" apellidos="Gutierrez Dos"
+         password="123456789" eMail="juan.gutierrez@gmail.com" fechaNacimiento="1993-12-10"/>
+    <Usuario id="1002" login="juangutierrez3" nombre="Juan" apellidos="Gutierrez Tres"
+         password="123456789" eMail="juan.gutierrez@gmail.com" fechaNacimiento="1993-12-10"/>
     <Tarea id="1000" titulo="Renovar DNI" usuarioId="1000"/>
     <Tarea id="1001" titulo="Práctica 1 MADS" usuarioId="1000"/>
-    <Tablero/>
+    <Tablero id="1000" nombre="Tablero test 1" administradorId="1000"/>
+    <Tablero id="1001" nombre="Tablero test 2" administradorId="1000"/>
  </dataset>
```

Estamos añadiendo dos usuarios nuevos y dos tableros, estos últimos
administrados por el usuario 1000 (`juangutierrez`).

Vamos a codificar lo necesario para que el test pase. El test
anterior comprueba que un usuario pueda tener varios tableros y,
aplicando TDD de forma estricta, sólo deberíamos codificar la relación
`ONE_TO_MANY`. Sin embargo, como es muy sencillo introducir
directamente la relación `MANY_TO_MANY` que necesitamos nos saltamos
la regla de TDD de introducir un único test y añadimos el siguiente test:

```diff
+
+   @Test
+   public void testTableroTieneVariosUsuarios() throws Exception {
+      initDataSet();
+      UsuarioRepository usuarioRepository = injector.instanceOf(UsuarioRepository.class);
+      TableroRepository tableroRepository = injector.instanceOf(TableroRepository.class);
+      // Obtenemos datos del dataset
+      Tablero tablero = tableroRepository.findById(1000L);
+      Usuario usuario1 = usuarioRepository.findById(1000L);
+      Usuario usuario2 = usuarioRepository.findById(1001L);
+      Usuario usuario3 = usuarioRepository.findById(1002L);
+      assertEquals(0, tablero.getParticipantes().size());
+      assertEquals(0, usuario1.getTableros().size());
+      // Añadimos los 3 usuarios al tablero
+      tablero.getParticipantes().add(usuario1);
+      tablero.getParticipantes().add(usuario2);
+      tablero.getParticipantes().add(usuario3);
+      tableroRepository.update(tablero);
+      // Comprobamos que los datos se han actualizado
+      tablero = tableroRepository.findById(1000L);
+      usuario1 = usuarioRepository.findById(1000L);
+      assertEquals(3, tablero.getParticipantes().size());
+      assertEquals(1, usuario1.getTableros().size());
+      assertTrue(tablero.getParticipantes().contains(usuario1));
+      assertTrue(usuario1.getTableros().contains(tablero));
+   }
 }
```

En este test añadimos tres usuarios a un tablero y después comprobamos
que los datos se han actualizado correctamente al recuperar el tablero
del `tableroRepository`.

Vamos entonces a escribir el código que haga pasar estos dos tests.

Empezamos por el modelo, añadiendo la relación `MANY_TO_MANY` en
`Usuario` y `Tablero`:

**Fichero `models/Usuario.java`**:

```diff
    private Date fechaNacimiento;
    // Relación uno-a-muchos entre usuario y tarea
    @OneToMany(mappedBy="usuario", fetch=FetchType.EAGER)
    private Set<Tarea> tareas = new HashSet<Tarea>();
    @OneToMany(mappedBy="administrador", fetch=FetchType.EAGER)
    private Set<Tablero> administrados = new HashSet<Tablero>();
+   @ManyToMany(mappedBy="participantes", fetch=FetchType.EAGER)
+   private Set<Tablero> tableros = new HashSet<Tablero>();
  
    // Un constructor vacío necesario para JPA
    public Usuario() {}

...

        this.administrados = administrados;
     }
  
+   public Set<Tablero> getTableros() {
+      return tableros;
+   }
+
+   public void setTableros(Set<Tablero> tableros) {
+      this.tableros = tableros;
+   }
+
    public String toString() {
       String fechaStr = null;
       if (fechaNacimiento != null) {
```

En el modelo `Tablero` añadimos la relación y los métodos `hashCode` y
`equals` necesarios para comparar tableros y buscar tableros en
colecciones (sin ese método no funciona correctamente la llamada a
`contains` del primer test):

**Fichero `models/Tablero.java`**:

```java

 import javax.persistence.*;
  
+import java.util.Set;
+import java.util.HashSet;
+
 @Entity
 public class Tablero {
    @Id
...
    @ManyToOne
    @JoinColumn(name="administradorId")
    private Usuario administrador;
+   @ManyToMany(fetch=FetchType.EAGER)
+   @JoinTable(name="Persona_Tablero")
+   private Set<Usuario> participantes = new HashSet<Usuario>();
  
    public Tablero() {}
  
...

    public void setAdministrador(Usuario usuario) {
       this.administrador = administrador;
    }
+
+   public Set<Usuario> getParticipantes() {
+      return participantes;
+   }
+
+   public void setParticipantes(Set<Usuario> participantes) {
+      this.participantes = participantes;
+   }
+
+   @Override
+   public int hashCode() {
+      final int prime = 31;
+      int result = prime + ((nombre == null) ? 0 : nombre.hashCode());
+      result = result + ((administrador == null) ? 0 : administrador.hashCode());
+      return result;
+   }
+
+   @Override
+   public boolean equals(Object obj) {
+      if (this == obj) return true;
+      if (getClass() != obj.getClass()) return false;
+      Tablero other = (Tablero) obj;
+      // Si tenemos los ID, comparamos por ID
+      if (id != null && other.id != null)
+         return ((long) id == (long) other.id);
+      // sino comparamos por campos obligatorios
+      else {
+         if (nombre == null) {
+            if (other.nombre != null) return false;
+         } else if (!nombre.equals(other.nombre)) return false;
+         if (administrador == null) {
+            if (other.administrador != null) return false;
+            else if (!administrador.equals(other.administrador)) return false;
+         }
+      }
+      return true;
+   }
 }
```

Necesitamos también añadir los métodos `update` y `findById` en la
interfaz `TableroRepository`:

**Fichero `models/TableroRepository.java`**:

```diff
@ImplementedBy(JPATableroRepository.class)
 public interface TableroRepository {
    public Tablero add(Tablero tablero);
+   public Tablero update(Tablero tablero);
+   public Tablero findById(Long idTablero);
 }
```

Y su implementación en `JPATableroRepository.java`**:

```diff
+
+   public Tablero update(Tablero tablero) {
+      return jpaApi.withTransaction(entityManager -> {
+         Tablero actualizado = entityManager.merge(tablero);
+         return actualizado;
+      });
+   }
+
+   public Tablero findById(Long idTablero) {
+      return jpaApi.withTransaction(entityManager -> {
+         return entityManager.find(Tablero.class, idTablero);
+      });
+   }
+
 }
```

Lanzamos los tests y comprobamos que todo funciona correctamente.

Antes de hacer el commit es necesario añadir en el fichero
`usuarios_dataset.xml` la línea para limpiar la nueva tabla que genera
la relación `MANY_TO_MANY`. Si no se añade, fallarán las siguientes
ejecuciones de los tests.

**Fichero `test/resources/usuarios_dataset.xml`**:

```diff
     <Tablero id="1001" nombre="Tablero test 2" administradorId="1000"/>
+    <Persona_Tablero/>
  </dataset>
```

Hacemos el commit y subimos la rama para crear el pull request:

```
$ git add .
$ git commit -m "Relación muchos a muchos entre usuarios y tableros"
$ git push -u origin modelo-tablero
```

En GitHub creamos el pull request con la rama.

### 4.2. Tests de integración antes de cerrar el _issue_ y confirmar el pull request ###

Con los commits anteriores hemos terminado de codificar el
_issue_. Una vez que hemos creado el pull request en GitHub, antes de
aceptarlo, debemos **comprobar que funciona correctamente la
integración con `master`**. 

GitHub nos informa si el PR tiene algún conflicto con `master`.
En ese caso deberíamos también resolverlos.

Para ello debemos hacer lo siguiente:

1. Hacer un merge de `master` en la rama del _issue_. En este merge
   pueden producirse conflictos que deberemos arreglar.
2. Una vez realizado el merge comprobar que pasan los tests de
   integración y de _stage_. Si el esquema de datos ha cambiado,
   obtener el nuevo esquema y aplicar los cambios en la base de datos
   de _stage_.
3. Aceptar el pull request: subir los nuevos commits al pull request,
   confirmarlo para realizar la integración con `master` en remoto y
   actualizar la rama `master` local.


#### Merge con la rama master ####

Mezclamos la rama `master` con la rama actual del _issue_. Pero antes
de ello debemos actualizar `master` para descargar los cambios que se
hayan podido subir a remoto. Aunque en este caso no habrá ningún
cambio, este paso es muy importante cuando estemos trabajando en
equipo y haya múltiples _issues_ integrándose simultáneamente. Una vez
actualizado `master` volvemos a la rama `modelo-tablero` y realizamos
el merge de `master`:

```
$ git checkout master
$ git pull
$ git checkout modelo-tablero
$ git merge master
```

Puede que no sea posible realizar el _merge_ porque git detecta algún
conflicto. Aparecería un mensaje como el siguiente:

```
$ git merge master
Auto-merging app/services/TareaService.java
CONFLICT (content): Merge conflict in app/services/TareaService.java
Auto-merging app/models/Usuario.java
CONFLICT (content): Merge conflict in app/models/Usuario.java
```

Debes solucionarlo siguiendo las instrucciones que aparecen en el `git
status`:

```
$ git status
On branch modelo-tablero
Your branch is up-to-date with 'solucion/modelo-tablero'.
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Changes to be committed:

	modified:   test/models/TareaTest.java

Unmerged paths:
  (use "git add <file>..." to mark resolution)

	both modified:   app/models/Usuario.java
	both modified:   app/services/TareaService.java
```

Debes editar los ficheros en los que haya conflicto. Verás que git ha
modificado los ficheros incluyendo marcas que indican las líneas que
están en conflicto. Edita las líneas para dejar el fichero como te
interesa y sálvalo.

Una vez arreglados los conflictos hay que informar a git de esta
resolución. Como indica el `git status` debemos hacer un
`git add` de los ficheros en los que hemos resuelto los conflictos:

```
$ git add app/models/Usuario.java
$ git add app/services/TareaService.java
```

Si hacemos ahora un `git status` git nos explica cómo continuar:

```
$ git status
On branch modelo-tablero
Your branch is up-to-date with 'solucion/modelo-tablero2'.
All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)

Changes to be committed:

	modified:   app/services/TareaService.java
	modified:   test/models/TareaTest.java

```

Nos dice que debemos hacer un commit para concluir el merge. Hacemos
después un `push` para subirlo a GitHub:

```
$ git commit -m "Solucionados conflictos"
$ git push
```

Veremos en GitHub que el pull request se actualiza con el nuevo commit
y que ya no aparecen conflictos con master.

En el caso en que no hayan aparecido conflictos no es necesario hacer
un `push` a GitHub (no hay cambios añadidos en la rama).


#### Tests de integración ####

Una vez realizada la integración de master en la rama del _issue_
debes lanzar los tests de integración tal y como se explica en el
apartado 3.1.

Si aparece algún error debes solucionarlo, crear un nuevo commit y
subirlo a GitHub.

En nuestro caso uno de los errores que aparecerán está relacionado con
cómo se almacenan los nombres de las tablas en la base de datos H2
(memoria) y MySQL. Una de las bases de datos las guarda en mayúsculas
y la otra no.

Debes modificar el test `testCrearTableTableroEnBD` para contemplar
las dos posibilidades:

```diff
   @Test
   public void testCrearTablaTableroEnBD() throws Exception {
      Database db = injector.instanceOf(Database.class);
      Connection connection = db.getConnection();
      DatabaseMetaData meta = connection.getMetaData();
-     ResultSet resH2 = meta.getTables(null, null, "TABLERO", new String[] {"TABLE"});
+     // En la BD H2 el nombre de las tablas se define con mayúscula y en
+     // MySQL con minúscula
+     ResultSet resH2 = meta.getTables(null, null, "TABLERO", new String[] {"TABLE"});
+     ResultSet resMySQL = meta.getTables(null, null, "Tablero", new String[] {"TABLE"});
+     boolean existeTabla = resH2.next() || resMySQL.next();
      assertTrue(existeTabla);
   }
```

Una vez que los tests de integración funcionan, generamos la tabla con
el esquema de datos (`schema.sql`) tal y como se explica en el
apartado 3.1 y la copiamos en el directorio correspondiente del
proyecto (`sql/schema.sql`).

Si hay algún cambio en el esquema git los detectará. Podemos comprobar
los cambios haciendo un `git diff`:

```diff
--
+-- Table structure for table `Persona_Tablero`
+--
+
+DROP TABLE IF EXISTS `Persona_Tablero`;
+/*!40101 SET @saved_cs_client     = @@character_set_client */;
+/*!40101 SET character_set_client = utf8 */;
+CREATE TABLE `Persona_Tablero` (
+  `tableros_id` bigint(20) NOT NULL,
+  `participantes_id` bigint(20) NOT NULL,
+  KEY `FKnghbrhyh7eal30o78h3293n72` (`participantes_id`),
+  KEY `FKbpw5yq3ofgud0ra8a916kddjm` (`tableros_id`),
+  CONSTRAINT `FKbpw5yq3ofgud0ra8a916kddjm` FOREIGN KEY (`tableros_id`) REFERENCES `Tablero` (`id`),
+  CONSTRAINT `FKnghbrhyh7eal30o78h3293n72` FOREIGN KEY (`participantes_id`) REFERENCES `Usuario` (`id`)
+) ENGINE=InnoDB DEFAULT CHARSET=latin1;
+/*!40101 SET character_set_client = @saved_cs_client */;
+
+--
+-- Table structure for table `Tablero`
+--
+
+DROP TABLE IF EXISTS `Tablero`;
+/*!40101 SET @saved_cs_client     = @@character_set_client */;
+/*!40101 SET character_set_client = utf8 */;
+CREATE TABLE `Tablero` (
+  `id` bigint(20) NOT NULL,
+  `nombre` varchar(255) DEFAULT NULL,
+  `administradorId` bigint(20) DEFAULT NULL,
+  PRIMARY KEY (`id`),
+  KEY `FKq82919iay2b8h77msdj8289p0` (`administradorId`),
+  CONSTRAINT `FKq82919iay2b8h77msdj8289p0` FOREIGN KEY (`administradorId`) REFERENCES `Usuario` (`id`)
+) ENGINE=InnoDB DEFAULT CHARSET=latin1;
+/*!40101 SET character_set_client = @saved_cs_client */;
+
+--
```

En este caso los cambios consisten en las dos tablas nuevas
añadidas. Una para la entidad `Tablero` y otra para mantener la
relación `MANY_TO_MANY` con `Usuario`.

Añadimos los cambios del esquema con un commit y los subimos a GitHub:

```
$ git add sql/schema.sql
$ git commit -m "Cambios en el esquema SQL"
$ git push
```

##### Tests de stage #####

Si lanzamos ahora el entorno _stage_ tal y como explicamos en el
apartado 3.2 comprobaremos que la aplicación no funciona, porque el
esquema guardado no se corresponde con el de la aplicación.

Debemos crear un **script de actualización** de la base de datos a partir
de los cambios observados en el apartado anterior.

En nuestro caso el script consistirá en las mismas sentencias SQL para
crear las nuevas tablas. En otros casos tendrás que hacer un `ALTER
TABLE` para actualizar tablas ya existentes con nuevas columnas.

Llamamos al fichero `Upgrade.sql`:

```sql
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

CREATE TABLE `Persona_Tablero` (
  `tableros_id` bigint(20) NOT NULL,
  `participantes_id` bigint(20) NOT NULL,
  KEY `FKnghbrhyh7eal30o78h3293n72` (`participantes_id`),
  KEY `FKbpw5yq3ofgud0ra8a916kddjm` (`tableros_id`),
  CONSTRAINT `FKbpw5yq3ofgud0ra8a916kddjm` FOREIGN KEY (`tableros_id`) REFERENCES `Tablero` (`id`),
  CONSTRAINT `FKnghbrhyh7eal30o78h3293n72` FOREIGN KEY (`participantes_id`) REFERENCES `Usuario` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

CREATE TABLE `Tablero` (
  `id` bigint(20) NOT NULL,
  `nombre` varchar(255) DEFAULT NULL,
  `administradorId` bigint(20) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `FKq82919iay2b8h77msdj8289p0` (`administradorId`),
  CONSTRAINT `FKq82919iay2b8h77msdj8289p0` FOREIGN KEY (`administradorId`) REFERENCES `Usuario` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```

La línea 

```sql
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
```

es importante porque le indica a MySQL que no chequee la existencia de
claves ajenas al crear las tablas. De esta forma podemos crearlas en
el orden en el que aparecen.

Guardamos el fichero `upgrade.sql` en el mismo directorio `stage` en
el que está el fichero `schema.sql` con el backup de los datos
de anteriores ejecuciones de pruebas funcionales en el directorio.

Al arrancar el contenedor docker MySQL de _stage_ (ver apartado 3.2) se
cargarán los ficheros SQL en orden alfabético, de forma que primero se
cargarán los datos anteriores (fichero `schema.sql`) y después se
realizará la actualización (fichero `upgrade.sql`).

Arrancamos la aplicación Play en modo _stage_ (ver apartado 3.2) y
comprobamos que todo funciona correctamente.

Por último añadimos el fichero `upgrade.sql` al proyecto en el
directorio SQL, llamándolo `upgrade1.sql`. De esta forma guardamos
también los scripts de actualización en el control de versiones.


```
$ cp upgrade.sql DIR_PROYECTO/sql/upgrade1.sql
$ git commit -m "Añadido fichero actualización BD"
$ git push
```

#### Confirmación del pull request ####

Por último confirmamos el pull request en GitHub y se hace la
integración en `master` remoto. Borramos la rama remota
`modelo-tablero`. 

Actualizamos la mezcla en local y borramos también la rama local:

```
$ git checkout master
$ git pull
$ git branch -d modelo-tablero
$ git remote prune origin
```

### 4.3. Resto de _issues_ (parte obligatoria)

Termina los issues 2 y 3 de la historia de usuario, realizando el
primero también con TDD:

2. Métodos de servicio para crear un tablero y para obtener el listado
   de tableros administrados por un usuario.
3. Controlador, acción y vista para un listado de tableros
   administrados y posibilidad de añadir nuevos tableros
   administrados.

### 4.4 Resto de _issues_ (parte opcional) ###

Termina los issues 4, 5, 6 y 7, haciendo con TDD todos los
correspondientes a métodos de servicio:

4. Métodos de servicio para apuntarse a un tablero (como participante)
   y para obtener los siguientes listados:
   - Listado de tableros en los que participa el usuario
   - Listado de resto de tableros (en los que el usuario ni participa ni es administrador).
5. Controlador, acción y modificación de la vista con el listado de
   tableros para mostrar los listados anteriores y permitir apuntarse
   como participante a un tablero.
6. Métodos de modelo y servicio para obtener descripción de un tablero
   (nombre, administrador y lista de participantes).
7. Controlador y vista con descripción de un tablero y añadir enlaces en el
   listado de tableros para que al pinchar se vaya a su descripción.

### 4.5. Finalización de la versión 0.2 ###

Una vez terminada la práctica, creamos un nuevo _release_.

- Cuando hayas integrado el último PR, haz un commit en master en el
  que modifiques la versión del proyecto en el fichero build.sbt:

    ```
    version := "0.2"
    ```

  Publica directamente el commit en master (sin hacer PR).

- Añade en GitHub el tag con el número de versión:

    - Pincha enlace `releases` en la página principal
    - Añade una nueva versión: `v0.2` y pulsa el botón para publicar el
      release. Esto creará la etiqueta y la versión en GitHub.

- Por último, cambia la versión actual (en `build.sbt` en `master`) a
  `0.3-SNAPSHOT` haciendo y publicando un nuevo commit. De esta forma,
  indicamos que ahora en master se está desarrollando la versión 0.3.


## 5. Entrega y evaluación

- La práctica tiene una duración de 3 semanas y debe estar terminada
  el martes 24 de octubre.
- La parte obligatoria puntúa sobre 7 y la opcional sobre 3 puntos.
- La calificación de la práctica tiene un peso de un 8% en la nota
  final de la asignatura. 
- Para realizar la entrega se debe subir a Moodle un ZIP que contenga
  todo el proyecto, incluyendo la historia Git. Para ello comprime tu
  directorio local del proyecto **después de haber hecho un
  `clean`**. Debes dejar también en Moodle la URL del repositorio en
  GitHub.

Para la evaluación se tendrá en cuenta:

- Desarrollo continuo (los _commits_ deben realizarse a lo largo de
  las 3 semanas y no dejar todo para la última semana).
- Correcto desarrollo de la metodología.
- Corrección del código.

