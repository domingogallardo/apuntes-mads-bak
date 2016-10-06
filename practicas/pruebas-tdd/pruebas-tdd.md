# Práctica 2: Pruebas y TDD con Play Framework

## Objetivos

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

## Conexión a base de datos MySQL

### Cambio de clave primaria de `String` a `Integer`

En la práctica 1 el tipo de dato de la clave primaria de `Usuario` lo
especificamos como `String`. Sin embargo, en la práctica 3 vamos a
trabajar con una base de datos MySQL que no puede gestionar claves
autogeneradas de este tipo. Debes cambiar por ello la clave primaria
de `Usuario` a `Integer`.

> Igual que en la práctica anterior, abre un tícket nuevo en Trello y
> realiza todas las modificaciones en una rama que después debes mezclar
> con _master_ usando `--no-ff`.


### Configuración de la conexión

Hasta ahora hemos desarrollado la aplicación usando la base de datos
en memoria `H2`. Sin embargo, para poder hacer comprobaciones y
demostraciones de la aplicación que estamos desarrollando, es
interesante poder trabajar con una base de datos que haga persistente
los datos entre distintas ejecuciones de la aplicación. También es
recomendable trabajar con bases de datos idénticas a las que
utilizaremos en producción.

> Debes implementar un nuevo tícket, en el que modifiques la base de
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

### Comprobación del funcionamiento

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


## Tests en Play Framework

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

### Introducción

#### JUnit

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

#### Aserciones y matchers

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


#### Mocks

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

### Tests con bases de datos

### Tests de la capa `services`

### Tests de la capa `controllers`


## Tests a realizar en tu aplicación


## Ampliación de la aplicación: tareas usando TDD

