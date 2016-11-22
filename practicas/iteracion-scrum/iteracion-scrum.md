# Práctica 4: Sprint de Scrum 

- [1. Objetivos y resumen de la práctica](#1-objetivos-y-resumen-de-la-práctica)
- [2. Planificación del sprint](#2-planificación-del-sprint)
- [3. Desarrollo del sprint usando Scrum](#3-desarrollo-del-sprint-usando-scrum)
- [4. Entrega y evaluación](#4-entrega-y-evaluación)

## 1. Objetivos y resumen de la práctica

En esta práctica seguiremos trabajando con los mismos equipos y
proyecto que la práctica 3.

Durante las 4 semanas de la práctica el equipo realizará una iteración
para desarrollar un incremento de la aplicación
`play-todolist`. Usaremos el mismo flujo de trabajo que hemos
utilizado en la práctica 3:

- Rama por ticket y pull request a master para integrarla 
- Tickets y features (conjunto de tickets) en Trello
- Repositorio conectado a Travis para comprobar la ejecución de los tests

Además, aplicaremos algunos elementos de Scrum:

- Planificación del sprint
- Reuniones de _scrum diario_
- Revisión del sprint (se hará en la última clase de teoría de la asignatura)
- Retrospectiva del sprint

## 2. Planificación del sprint

El equipo, junto con el profesor, seleccionará las historias de
usuario a realizar en la iteración y estimará su tamaño. Podrán
escogerse cualquiera de las definidas en el _workshop_ de _mapping_ de
historias de usuario realizado en clase, o idear alguna nueva.

Todos los miembros deberán tomar el papel _product owner_ y aportar
ideas y sugerencias para definir las historias de usuario.

Se deben incluir en el sprint historias por un tamaño total 6 puntos x
número de personas del equipo:

- Equipo de 2 personas: 12 puntos
- Equipo de 3 personas: 18 puntos
- Equipo de 4 personas: 24 puntos

Suponemos la siguiente asignación de puntos al tamaño de las tareas:

- S: 1 punto
- M: 2 puntos
- L: 4 puntos

Se analizarán las historias, especificando claramente su descripción y
su alcance. Para la descripción se recomienda usar el formato que
vimos en clase:

- Como _ROL_ quiero _ACCIÓN_ para _CONSECUENCIA_.

Hay que intentar que la consecuencia sea **bastante concreta**, para tener
más claro el alcance de la historia. 

También se enumerarán una serie de condiciones de satisfacción (COS)
que deben cumplirse para dar la historia como terminada.

> **Historias de usuario en Trello**:
> Crear las tarjetas con las historias de usuario elegidas para el
> sprint en la columna `En espera` de Trello. Numerar consecutivamente
> las historias y escribir el nombre de la historia precedido por
> `FT-` (feature). Incluir en la descripción de la tarjeta la
> descripción de la historia y una lista de las condiciones de
> satisfacción que se utilizarán para validarla. Si es necesario
> añadir un enlace a un documento GoogleDocs en el que se añada más
> información sobre la historia de usuario.

Si es posible, realizar en esta primera reunión una descomposición de
alguna historia en tareas más pequeñas (_tickets_, en la terminología
que hemos estado comentando) para empezar ya a trabajar.

Tomar nota del desarrollo de la reunión, para documentarla en el
informe final de la práctica.

## 3. Desarrollo del sprint usando Scrum

Se deberán realizar los siguientes eventos definidos por Scrum,
tomando nota y redactando un informe con la fecha de la reunión, su
duración y su desarrollo:

1. planificación del sprint
2. scrum diario (al menos simular 2 reuniones)
3. retrospectiva del sprint

> **Desarrollo de las historias de usuario**: desarrollaremos todas
> las historias seleccionadas para el sprint usando la versión de
> GitFlow y Trello que hemos utilizado en la práctica 3. Añadiremos
> alguna columna adicional al tablero, para adaptarlo mejor a un
> tablero Kanban.

Resumen del desarrollo:

- Ramas para los tickets y _pull requests_ con revisión de código para
  integrar los tickets en master.
- Cada ticket debe contener **tests automáticos** que prueben los
  cambios. En el caso de no poder incluirse (por ejemplo en tickets
  que añaden elementos a la interfaz de usuario) se debe especificar
  en la tarjeta de Trello (en el documento Google Docs que se enlace)
  **los tests manuales que habría que hacer**.
- En Trello, numeramos los tickets con el número de _feature_ y usamos
  las etiquetas como lo hemos hecho en la práctica 2.
- Columnas en Trello:
    - **Seleccionados** - _Ticket_ listo para el desarrollo
    - **En desarrollo** - Ticket en desarrollo. Cada ticket debe corresponderse con una rama de Git.
    - **En desarrollo - Done** - Se ha terminado el desarrollo y los tests unitarios.
    - **Integración** - Se ha lanzado el _pull request_ pidiendo la integración del _ticket_ en la rama `master`.
    - **Integración - Done** - Se ha aprobado el _pull request_, se ha realizado la integración `master` y Travis da el OK.
    - **Pruebas funcionales** - Se realizan las pruebas funcionales del _ticket_.
    - **Terminados**- El ticket ha pasado las pruebas funcionales.
- Límite de _Work In Progress_ (WIP): Estimar un número de límite de
  WIP para las columnas `Seleccionados`, `En desarrollo`,
  `Integración` y `Pruebas funcionales`. No se podrá incluir en esas
  columnas más tarjetas que las definidas en el número límite de
  WIP. Las tarjetas en las columnas _done_ correspondientes también se
  suman a las que hay en las propias columnas.
- Seguimos usando Travis para la integración continua.

> Al final del desarrollo se deberá publicar la nueva versión en la
>rama `production`.

### Documentación del desarrollo

- Documentar las reuniones anteriores, para incluir un informe en el documento.
- Documentar la evolución del tablero Trello y alguna métrica del desarrollo.

## 4. Entrega y evaluación

La práctica tiene una duración de 4 semanas:
- Se realizará una **revisión del sprint** de 10 minutos en **clase de
teoría del 21 de diciembre** (presentación de las funcionalidades
introducidas y demo).  
- La fecha límite de entrega es el **viernes 23 de diciembre**.

Se deberá entregar:

- Repositorio GitHub
- Informe de la práctica deberá (fichero markdown en carpeta `doc` del
  proyecto) con al menos:
    - Historias de usuario escogidas para el sprint (copiar la
      descripción y las condiciones de satisfacción tal y como se
      definieron al principio).
    - Funcionalidades implementadas (breve descripción para el usuario
      y breve descripción técnica).
    - Informe sobre la metodología seguida (ejemplos de evolución del
      tablero, alguna métrica del desarrollo realizado en el sprint)
    - Informes sobre las reuniones de Scrum (planificación del sprint,
      3 scrum diario, revisión).
    - Resultado de la retrospectiva: qué ha ido bien y qué se podría
      mejorar.

