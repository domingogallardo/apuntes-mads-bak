# Práctica 4: Sprint de Scrum 

- [1. Objetivos y resumen de la práctica](#1-objetivos-y-resumen-de-la-práctica)
- [2. Planificación del sprint](#2-planificación-del-sprint)
- [3. Desarrollo del sprint](#3-desarrollo-del-sprint)
- [4. Entrega y evaluación](#4-entrega-y-evaluación)


## 1. Objetivos y resumen de la práctica

En esta práctica seguiremos trabajando con los mismos equipos y
proyecto que la práctica 3.

Durante las 4 semanas de la práctica el equipo realizará una iteración
para desarrollar un incremento de la aplicación
`todolist`. Usaremos el mismo flujo de trabajo de la práctica 3 para
desarrollar sobre la rama `master`:

- Una página en la Wiki de GitHub para cada historia de usuario, cuya
  implementación se divide en issues
- Rama por issue y pull request a master para integrarla 
- En el tablero en GitHub se representan el estado de los issues y
  pull requests
- El repositorio está conectado a Travis que comprueba de forma
  automática los tests en las integraciones de los pull requests en
  master y sube una imagen docker a Docker Hub.
- En Docker Hub se tienen numerados todos los builds exitosos de
  master, y está etiquetada como `latest` la última versión de la
  imagen.

Al final de la práctica se lanzará una nueva versión (`1.2`), usando
el mismo flujo de trabajo que la práctica anterior.

Además, aplicaremos algunos elementos de Scrum:

- Planificación del sprint
- Reuniones de _scrum diario_
- Revisión del sprint (se hará en la última clase de teoría de la asignatura)
- Retrospectiva del sprint

## 2. Planificación del sprint

Cada equipo creará un tablero en **Trello** al que se añadirán todos los
miembros y el profesor. El tablero constará de las siguientes
columnas, en las que se gestionarán las historias de usuario:

- **Backlog**: Posibles historias de usuario a seleccionar en el sprint.

- **Sprint**: Historias de usuario a terminar en el sprint.

- **En marcha**: Historias de usuario que se están desarrollando. 

  Para pasar la historia a esta columna, debe estar terminada su
  página en la Wiki en GitHub (y enumeradas todas las COS). En la
  tarjeta pondremos un enlace a esa wiki.

- **QA**: Historias de usuario a las que se les está realizando
  pruebas funcionales y de rendimiento. 
  
  Cuando todos los issues de una historia de usuario se han integrado,
  se anota el enlace al build en DockerHub y se prueba la imagen en un
  entorno de stage, realizándose pruebas funcionales y de rendimiento
  relacionadas con la historia de usuario. Si se detectan bugs, se
  abren nuevos issues en GitHub para resolverlos, y la historia queda
  detenida en QA.
  
- **Terminadas**: Historias de usuario que se han probado satisfactoriamente.

### 2.1. Selección del backlog ###

El equipo, junto con el profesor, seleccionará las posibles historias
de usuario a realizar en la iteración y estimará su tamaño. Podrán
escogerse cualquiera de las definidas en el _workshop_ de _mapping_ de
historias de usuario realizado en clase, o idear alguna nueva.

El resultado del _mapping_ de historias de usuario está recogido en
los siguientes tableros:

- [https://trello.com/b/UaatzbrN/mapa-1-todolist](https://trello.com/b/UaatzbrN/mapa-1-todolist)
- [https://trello.com/b/LGRlvvAU/mapa-2-todolist](https://trello.com/b/LGRlvvAU/mapa-2-todolist)
- [https://trello.com/b/L6gTNPaF/mapa-3-todolist](https://trello.com/b/L6gTNPaF/mapa-3-todolist)
- [https://trello.com/b/WbQxj8lW/mapa-4-todolist](https://trello.com/b/WbQxj8lW/mapa-4-todolist)

Todos los miembros deberán tomar el papel _product owner_ y aportar
ideas y sugerencias para definir las historias de usuario. 

Cada historia incluida en la columna de **Backlog** se analizará,
especificando claramente su descripción y su alcance. Para la
descripción se recomienda usar el formato que vimos en clase:

- Como _ROL_ quiero _ACCIÓN_ para _RESULTADO_ u _OBJETIVO_.

También se enumerarán una serie de condiciones de satisfacción (COS)
que deben cumplirse para dar la historia como terminada. Estas
condiciones de satisfacción son esenciales a la hora de determinar el
alcance de la historia.

Al incluir la historia en la columna de **Backlog** se realizará un
_planning pocker_ para volver a consensuar su tamaño (considerad el
tamaño previo de la historia como una indicación, pero podéis
modificarlo). El profesor no participará en el _planning pocker_, pero
podrá pedir aclaraciones sobre el tamaño de las historias.

En la tarjeta de Trello se escribirá el título, el tamaño (etiqueta) y
la descripción de la historia. 

### 2.2. Planificación del sprint ###

Se deben incluir en el sprint historias por un tamaño total 6 puntos x
número de personas del equipo:

- Equipo de 2 personas: 8 puntos
- Equipo de 3 personas: 12 puntos

Suponemos la siguiente asignación de puntos al tamaño de las tareas:

- S: 1 punto
- M: 2 puntos (el doble que una S)
- L: 4 puntos (el doble que una M)

Cuando se seleccione la historia para el sprint actual y se pase a la
columna de **Sprint** se añadirá un enlace a la página correspondiente
en la wiki de GitHub, donde se copiará el título, la descripción y se
detallarán las condiciones de satisfacción y los issues en los que se
prevé descomponer la historia.

### 2.3. Responsables de historia de usuario ###

Una vez seleccionadas todas las historias los miembros del equipo
elegirán responsables para cada historia y se asignarán en
Trello. 

Todos los miembros del equipo deberán realizar un trabajo equitativo,
y se repartirán las historias de forma que queden también equilibrados
los tamaños totales de las historias asignadas a cada uno.

El responsable de cada historia de usuario

En Trello:
- Se asigna como responsable de la tarea
- Mueve la tarea de un estado a otro

En GitHub:
- Escribe la página en la Wiki y rellena las COS y los issues
- Crea la etiqueta con la historia y los asigna a los issues


## 3. Desarrollo del sprint

Se deberán realizar los siguientes eventos definidos por Scrum,
tomando nota y redactando un informe con la fecha de la reunión, su
duración y su desarrollo:

1. Scrum diario (al menos simular 2 reuniones: la segunda y tercera
   semana). En nuestro "tiempo simulado" en las prácticas, una semana
   es como un día de trabajo completo en una empresa.
2. Retrospectiva del sprint

### 3.1. Resumen del desarrollo ###

- Cada miembro del equipo selecciona el issue a desarrollar. Lo normal
  es que cada miembro desarrolle un único issue de forma concurrente,
  aunque podría darse el caso de ser más (por ejemplo, si algún issue
  no puede terminarse por estar bloqueado por otro).
- Ramas para los issues y pull requests con revisión de código para
  integrar los pull requests en master.
- Cada issue debe contener **tests automáticos** que prueben los
  cambios. En el caso de no poder incluirse (por ejemplo en issues
  que desarrollan el controller y la vista) se debe especificar
  en la wiki de la historia de usuario del issue **los tests manuales que habría que hacer**.
- Modificamos las columnas del tablero de issues de GitHub para
  adaptarlo mejor a Kanban:
    - **Seleccionados** - Issue listo para el desarrollo. 
    - **En desarrollo** - Issue en desarrollo. Cada issue debe corresponderse con una rama de Git.
    - **En desarrollo - Done** - Se ha terminado el desarrollo y los tests unitarios.
    - **Integración** - Se ha creado el pull request pidiendo la
      integración del issue en la rama `master`. Se solicita la
      revisión del código por al menos 1 compañero.
    - **Integración - Done** - Se ha aprobado el pull request, se ha
      realizado la integración `master` y Travis da el OK. 
    - **Terminados**- La historia de usuario correspondiente al issue
      ha pasado las pruebas funcionales.
- Límite de _Work In Progress_ (WIP): Estimar un número de límite de
  WIP para las columnas `Seleccionados`, `En desarrollo` e
  `Integración`. No se podrá incluir en esas columnas más tarjetas que
  las definidas en el número límite de WIP. Las tarjetas en las
  columnas _done_ correspondientes también se suman a las que hay en
  las propias columnas. El límite WIP va a depender del número de
  integrantes del equipo. Si se define un límite WIP demasiado bajo
  habrá personas ociosas, mientras que si se define un límite WIP
  demasiado alto habrá acumulación de tareas sin terminar.
- Seguimos usando Travis para la integración continua.

### 3.2. Publicación de nueva versión  ###

Al final del desarrollo se deberá publicar la nueva versión (1.2) en la rama
`production`, y subir a DockerHub la imagen resultante con la etiqueta
`1.2` y `latest` (hacerlo manualmente).

### 3.3. Documentación del desarrollo ###

- Documentar los dailys, para incluir un informe en el documento.
- Documentar la evolución de los tableros (Trello y GitHub) y alguna
  métrica del desarrollo (pull requests por semana, velocidad de la
  semana, gráfica de burndown, etc.).

## 4. Entrega y evaluación

La práctica tiene una duración de 4 semanas:
- Se realizará una **revisión del sprint** de 10 minutos en **clase de
teoría del 20 de diciembre** (presentación de las funcionalidades
introducidas (PowerPoint) y demo).
- La fecha límite de entrega es el **viernes 22 de diciembre**.

Se deberá tener disponible en esa fecha:

- Versión 1.2 de la máquina Docker en DockerHub
- Tablero Trello
- Repositorio GitHub
  - Tablero de issues completados
  - Wiki con las historias de usuario
- Directorio `doc` del proyecto en el que se incluirá un documento
  markdown con el informe de la práctica y un PDF con el PowerPoint
  realizado en la demo. En el informe de la práctica se incluirá:
    - Historias de usuario escogidas para el sprint (copiar la
      descripción y las condiciones de satisfacción tal y como se
      definieron al principio).
    - Funcionalidades implementadas (breve descripción para el usuario
      y breve descripción técnica).
    - Informe sobre la metodología seguida (ejemplos de evolución del
      tablero, alguna métrica del desarrollo realizado en el sprint)
    - Informes sobre las reuniones de Scrum (planificación del sprint,
      scrum diario, revisión).
    - Resultado de la retrospectiva: qué ha ido bien y qué se podría
      mejorar.

Para la evaluación se tendrá en cuenta:

- Desarrollo continuo de los issues
- Corrección del código
- Correcto funcionamiento
- Informe de la práctica
- Si el trabajo de algún miembro del equipo es significativamente de
  menor calidad y/o cantidad que el del resto, se penalizará su
  calificación
