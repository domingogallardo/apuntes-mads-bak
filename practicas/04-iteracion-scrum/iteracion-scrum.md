# Práctica 4: Sprint de Scrum 

## Objetivos y resumen de la práctica

En esta práctica seguiremos trabajando con los mismos equipos y
proyecto que la práctica 3.

Durante las 4 semanas de la práctica el equipo realizará una iteración
para desarrollar un incremento de la aplicación
`todolist`. Usaremos el mismo flujo de trabajo de la práctica 3 para
desarrollar sobre la rama `master`:

- Una página en la Wiki de GitHub y un issue y pull request para
  cada historia de usuario. 
- En el tablero en GitHub se representan el estado de los issues y
  pull requests
- El repositorio está conectado a Travis que comprueba de forma
  automática los tests en las integraciones de los pull requests en
  master y sube una imagen docker a Docker Hub.
- En Docker Hub se tienen numerados todos los builds exitosos de
  master, y está etiquetada como `latest` la última versión de la
  imagen.

Al final de la práctica se lanzará una nueva versión (`1.4.0`), usando
el mismo flujo de trabajo que la práctica anterior.

Además, aplicaremos algunos elementos de Scrum y prácticas de XP:

- Planificación del sprint
- Reuniones de _scrum diario_
- Revisión del sprint (se hará el último día de clases de la asignatura)
- Retrospectiva del sprint
- Al menos dos sesiones de _pair programming_ de 1,5 h. cada una.


## Planificación del sprint

En la reunión de planificación del sprint se seleccionarán las
historias de usuario a realizar en el sprint y se completarán sus
condiciones de satisfacción.

### Artefactos del sprint ###

Cada equipo creará un tablero Trello compartido en el que se anotarán
en forma de tarjeta las historias de usuario a realizar en el
proyecto. Utilizaremos la wiki de GitHub para anotar más detalles de
las historias de usuario escogidas para el sprint.

Por último, utilizaremos el tablero del proyecto para realizar el
backlog del sprint.


#### Tablero Trello ####

El tablero Trello servirá para trabajar con las historias de usuario
en formato de tarjeta, escribirlas rápidamente, estimarlas y ordenarlas.

#### Wiki ####

La wiki del proyecto deberá contener las historias de usuario
escogidas para realizar en el sprint.

Como hasta ahora, se definirá una página de la wiki para cada historia
de usuario. En la página se escribirá el título de la historia, su
descripción y sus condiciones de satisfacción.

También se deberá incluir en cada historia de usuario un borrador del
aspecto de la interfaz de usuario resultante ([cómo añadir imágenes a
la wiki en
GitHub](https://help.github.com/articles/adding-images-to-wikis/)). No
es necesario que se haga con ninguna herramienta de mockup, puede
ser un dibujo sencillo hecho a mano.

#### Tablero ####

La primera columna del tablero será el backlog del sprint en el que se
añadirán los issues correspondientes a historias de usuario y bugs.

Podría darse el caso en el que una historia de usuario se divida en
más de un issue. En ese caso indicaremos en el título del issue y
en su descripción a qué historia de usuario se corresponde.

El resto de columnas las utilizaremos como un tablero Kanban. Como
hasta ahora, los issues/_pull requests_ se irán moviendo por ellas
según se vayan desarrollando.

Tendremos las siguientes columnas:

- **Sprint backlog**: Issues con las historias de usuario (o subtareas
  de las mismas) y los bugs a terminar en el sprint.
  
- **In progress**: Historias de usuario y bugs que se están
  desarrollando.

    Para pasar un issue que corresponde a una historia de
    usuario a esta columna, debe estar terminada su página en la wiki
    y debe tener un responsable.

- **In pull request**: issues que tienen un pull request abierto (se
  archivará la tarjeta del issue y se dejará sólo la tarjeta del
  pull request.

- **QA**: Historias de usuario a las que se les está realizando
  pruebas funcionales y de rendimiento. 
  
    Cuando todos los pull requests (correspondientes a issues y
    bugs) de una historia de usuario se han integrado, se anota en la
    página de la wiki el enlace al build en DockerHub y se prueba la
    imagen en un entorno de stage, realizándose pruebas funcionales y
    de rendimiento relacionadas con esa historia de usuario. Si se
    detectan nuevos bugs, se abren nuevos issues en GitHub para
    resolverlos, y la historia queda detenida en QA.
  
- **Done**: pull requests que se han probado satisfactoriamente.

### Selección del backlog ###

El equipo seleccionará las posibles historias de usuario a realizar en
la iteración y estimará su tamaño. Las anotarán en un tablero Trello
compartido con el equipo y con el profesor. El profesor validará la
selección y la estimación de tamaños.

Podrán escogerse cualquiera de las definidas en el _workshop_ de
_mapping_ de historias de usuario realizado en clase, o idear alguna
nueva.

El resultado del _mapping_ de historias de usuario está recogido en
los siguientes tableros:

- [https://trello.com/b/KbUefDdN/mapa-1-todolist](https://trello.com/b/KbUefDdN/mapa-1-todolist)
- [https://trello.com/b/OyMQTu3W/mapa-2-todolist](https://trello.com/b/OyMQTu3W/mapa-2-todolist)
- [https://trello.com/b/MdxWoXPZ/mapa-3-todolist](https://trello.com/b/MdxWoXPZ/mapa-3-todolist)
- [https://trello.com/b/ZnK1Kq5R/mapa-4-todolist](https://trello.com/b/ZnK1Kq5R/mapa-4-todolist)
- [https://trello.com/b/kHml8CjF/mapa-5-todolist](https://trello.com/b/kHml8CjF/mapa-5-todolist)

Todos los miembros deberán tomar el papel _product owner_ y aportar
ideas y sugerencias para definir las historias de usuario. 

En el tablero Trello se definirá una única columna **Backlog** en la
que se incluirán las historias. Cada historia se analizará,
especificando claramente su descripción y su alcance. Para la
descripción se recomienda usar el formato que vimos en clase:

- Como _ROL_ quiero _ACCIÓN_ para _RESULTADO_ u _OBJETIVO_.

También se enumerarán brevemente una serie de condiciones de
satisfacción (COS) que deben cumplirse para dar la historia como
terminada. Estas condiciones de satisfacción son esenciales a la hora
de determinar el alcance de la historia.

Al incluir la historia se realizará un _planning pocker_ para volver a
consensuar su tamaño (considerad el tamaño previo de la historia como
una indicación, pero podéis modificarlo). El profesor no participará
en el _planning pocker_, pero podrá pedir aclaraciones sobre el tamaño
de las historias.

En la tarjeta de Trello se escribirá el título, el tamaño (etiqueta) y
la descripción de la historia. 

### Planificación del sprint ###

Se deben incluir en el sprint historias por un tamaño total 4 puntos x
número de personas del equipo:

- Equipo de 3 personas: 12 puntos
- Equipo de 4 personas: 16 puntos

Suponemos la siguiente asignación de puntos al tamaño de las tareas:

- S: 1 punto
- M: 2 puntos (el doble que una S)
- L: 4 puntos (el doble que una M)

### Responsables de historia de usuario ###

Una vez seleccionadas todas las historias los miembros del equipo
elegirán responsables para cada historia y se creará el issue (o
issues) correspondiente a la historia de usuario y se añadirá el
responsable al issue.

El responsable de la historia creará su página en la wiki de GitHub,
detallará allí las condiciones de satisfacción y añadirá el borrador
de la interfaz de usuario.

Todos los miembros del equipo deberán realizar un trabajo equitativo,
y se repartirán las historias de forma que queden también equilibrados
los tamaños totales de las historias asignadas a cada uno.

## Desarrollo del sprint

Se deberán realizar los siguientes eventos definidos por Scrum y XP,
tomando nota y redactando un informe con la fecha de la reunión, su
duración y su desarrollo:

1. Scrum diario (al menos simular 2 reuniones: la segunda y tercera
   semana). En nuestro "tiempo simulado" en las prácticas, una semana
   es como un día de trabajo completo en una empresa.
2. Retrospectiva del sprint
3. 2 sesiones de pair programming con turnos de 20 minutos (en cada
   sesión se deben hacer 4 turnos). Se podrán hacer estas sesiones en
   clase de prácticas.

### Resumen del desarrollo ###

- Cada miembro del equipo selecciona el issue a desarrollar. Lo normal
  es que cada miembro desarrolle un único issue de forma concurrente,
  aunque podría darse el caso de ser más (por ejemplo, si algún issue
  no puede terminarse por estar bloqueado por otro).

- Ramas para los issues y pull requests con revisión de código para
  integrar los pull requests en master. Es suficiente con que haya una
  única aprobación para integrar el pull request. 

- Cada issue debe contener **tests automáticos** que prueben los
  cambios. En el caso de no poder incluirse (por ejemplo en issues
  que desarrollan el controller y la vista) se debe especificar
  en la wiki de la historia de usuario del issue **los tests manuales que habría que hacer**.

- Límite de _Work In Progress_ (WIP): Estimar un número de límite de
  WIP para las columnas `In progress` e `In pull request`. No se podrá
  incluir en esas columnas más tarjetas que las definidas en el número
  límite de WIP. 
  
    Las tarjetas en las columna `En desarrollo - Done` se suman a las
    que hay en la columna `En desarrollo`.
  
    El límite WIP va a depender del número de integrantes del
    equipo. Si se define un límite WIP demasiado bajo habrá personas
    ociosas, mientras que si se define un límite WIP demasiado alto
    habrá acumulación de tareas sin terminar.

- Añadimos las columnas de _buffer_ en el tablero de issues GitHub
  para adaptarlo mejor a Kanban:
    - **In progress - Done** - Se ha terminado el desarrollo y los
      tests unitarios y no se puede crear el PR porque la columna de `In pull
      request` ha alcanzado su límite WIP.
    - **In pull request - Done** - Se ha aprobado el pull request, se ha
      realizado la integración `master` y Travis da el OK. No se puede
      pasar a `QA` porque la columna ha alcanzado su límite WIP.

- Seguimos usando Travis para la integración continua.

### 3.2. Publicación de nueva versión  ###

Al final del desarrollo se deberá lanzar una nueva release (1.4.0) en
la rama `master`, y subir a DockerHub la imagen resultante con la
etiqueta `1.4.0`.

### 3.3. Documentación del desarrollo ###

- Documentar los dailys, para incluir un informe en el documento.
- Documentar las sesiones de pair programming.
- Documentar la evolución del tablero GitHub y alguna métrica del
  desarrollo (pull requests por semana, velocidad de la semana,
  gráfica de burndown, etc.).

## 4. Entrega y evaluación

La práctica tiene una duración de 4 semanas:

- Se realizará una **revisión del sprint** de 20 minutos en las **clases de
teoría y práctica del 19 de diciembre** (presentación de las funcionalidades
introducidas (diapositivas) y demo).
- La fecha límite de entrega es el **viernes 21 de diciembre**.

Se deberá tener disponible en esa fecha:

- Versión 1.4.0 de la máquina Docker en DockerHub
- Tablero Trello
- Repositorio GitHub
  - Tablero de issues completados
  - Wiki con las historias de usuario
- Directorio `doc` en el repositorio del proyecto en el que se incluirá un documento
  markdown con el informe de la práctica y un PDF con las diapositivas
  presentadas en la demo. En el informe de la práctica se incluirá:
    - Historias de usuario escogidas para el sprint (copiar la
      descripción, las condiciones de satisfacción y el borrador de
      interfaz de usuario tal y como aparecen en la wiki).
    - Funcionalidades implementadas (breve descripción para el usuario
      y breve descripción técnica).
    - Informe sobre la metodología seguida (ejemplos de evolución del
      tablero, alguna métrica del desarrollo realizado en el sprint)
    - Informes sobre las reuniones de Scrum (planificación del sprint,
      scrum diario, revisión) y sobre las sesiones de pair programming.
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
