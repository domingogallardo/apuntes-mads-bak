## Resumen de comandos Git

### Comandos básicos

- Inicializar git en un directorio:

    ```
    $ cd /ruta/a/mi/directorio  
    $ git config --global user.name <nombre-usuario>  
    $ git config --global user.email <email>  
    $ git init  
    $ git add .  
    $ git commit -m "Versión inicial"
    ```

- Publicar por primera vez el repositorio local en el remoto (en Bitbucket):

    ```
    $ git remote add origin https://<usuario>@bitbucket.org/<usuario>/mads-todolist.git  
    $ git push -u origin --all
    $ git push -u origin --tags
    ```

- Comprobar el estado del repositorio local:

    ```
    $ git status
    ```

- Comprobar las diferencias entre los ficheros modificados y el último commit:

    ```
    $ git diff
    ```

- Añadir un fichero al [_stage_](http://programmers.stackexchange.com/questions/119782/what-does-stage-mean-in-git) (añadirlo para el próximo commit):

    ```
    $ git add <fichero o directorio>
    ```

<img src="imagenes/staging-area.png" width="400px">

- Hacer un commit de los ficheros en el _stage_:

    ```
    $ git commit -m "Mensaje"
    ```

- Eliminar un fichero del _stage_ (si lo hemos añadido, pero al final decidimos no añadirlo en el siguiente commit):

    ```
    $ git reset HEAD <fichero>
    ```

- Añadir hacer un commit de todos los últimos cambios:

    ```
    $ git commit -a -m "Mensaje"
    ```

- Publicar los cambios en el repositorio remoto:

    ```
    $ git push
    ```

- Consultar los mensajes de los commits (toda la historia de la rama actual). La opción `--oneline` muestra sólo la primera línea del mensaje:

    ```
    $ git log [--oneline]
    ```

### Ramas

- Crear una rama nueva:

    ```
    $ git checkout -b nueva-rama  
    M   hola.txt (si hay cambios en el espacio de trabajo se llevan a la nueva rama)  
    Switched to a new branch 'nueva-rama'
    ```

- Listar las ramas de un repositorio:

    ```
    $ git branch  
    master  
    * nueva-rama  
    $ git commit -a -m "Confirmamos los cambios en la nueva rama"
    ```

- Moverse a otra rama:

    ```
    $ git checkout master  
    Switched to branch 'master'
    ```

- Mostrar un fichero de una rama (o commit) dado:

    ```
    $ git show <commit o rama>:<nombre-fichero>
    ```

- Comparar dos ramas:

    ```
    $ git diff rama1 rama2
    ```

    El comando `git diff rama1 rama2` devuelve las diferencias entre las ramas `rama1` y `rama2`: las modificaciones que resultarían de mezclar la rama `rama2` en la rama `rama1`.

- Mezclar la rama `rama2` en la rama `rama1` (añade a la `rama1` los commits adicionales de la rama `rama2`):

    ```
    $ git checkout rama1
    $ git merge [--no-ff] nueva-rama -m "Mensaje de commit"
    ```

    La opción `--no-ff` no hace un fast forward y mantiene separados los commits de la rama en el log de commits. Es útil para revisar la historia del repositorio.

- Si en la rama que se mezcla y en la actual hay cambios que afectan a las mismas líneas de un fichero, git detecta un conflicto y combina esas líneas conservando las dos versiones y añadiendo la información de la procedencia. Debemos resolver el conflicto: editarlos a mano y volver a hacer add y commit.

    ```
    $ git merge  
    CONFLICT (content): Merge conflict in hola.txt  
    Automatic merge failed; fix conflicts and then commit the result.  
    # editar a mano el fichero con conflictos  
    $ git commit -a -m "Arreglado el conflicto en el merge"
    $ git merge
    ```

    El comando `git status` después de un merge nos indica qué ficheros no se han mezclado y hay que editar manualmente.

- _Rebase_ de una rama. Si la rama master ha avanzado después de separar una rama alternativa y queremos incorporar esos cambios en la rama alternativa podemos hacer un `git rebase`:

    ```
    $ git checkout master  
    # hacemos cambios  
    $ git commit -a -m "Cambios en master"  
    $ git checkout rama-feature  
    $ git rebase master  
    First, rewinding head to replay your work on top of it...  
    Applying: Corregido bug1  
    Applying: Corregido bug2
    ```

    El comando cambia la historia de la rama: primero la mueve al final de la rama master (rewind head) y a partir de ahí aplica los cambios propios de la rama.

- Si aparecen conflictos al hacer el _rebase_, basta con modificar los ficheros con conflictos, añadirlos y continuar el _rebase_:

    ```
    $ git rebase master
    CONFLICT (content): Merge conflict in <some-file>
    # hacemos git status para comprobar donde están los conflictos
    $ git status
    # Unmerged paths:
    # (use "git reset HEAD <some-file>..." to unstage)
    # (use "git add/rm <some-file>..." as appropriate to mark resolution)
    #
    # Editamos los ficheros para corregir los conflictos
    $ git add <some-file>
    $ git rebase --continue
    ```

- **IMPORTANTE**: Es posible integrar los cambios de una rama haciendo un _merge_ o haciendo un _rebase_. Ambas estrategias son correctas y cada una tiene sus pros y contras. Nosotros vamos a usar principalmente los _rebases_. Estudia las diferencias entre ambas estrategias en el tutorial de Atlassian [Merging vs. Rebasing](https://www.atlassian.com/git/tutorials/merging-vs-rebasing).

- Log en forma de grafo:

    ```
    $ git log --graph --oneline 
    ```

- Borrar una rama:

    ```
    $ git branch -d nueva-rama  
    Deleted branch nueva-rama (was c241d7b)
    ```
    Sólo podemos borrar de la forma anterior ramas en las que no estamos y que se han mezclado con alguna otra. El comando anterior no permite borrar ramas activas que tienen commits sin mezclar con otras.

- Borrar una rama descartando sus commits:

    ```
    $ git branch -D rama
    ```

- Subir una rama al repositorio remoto:

    ```
    $ git push -u origin <rama>
    ```

- Subir todas las ramas y etiquetas:

    ```
    $ git push -u -all origin
    ```

    Al poner la opción -u hacemos tracking del repositorio remoto y las referencias quedan almacenadas en el fichero de configuración .git/config. A partir de ahora sólo es necesario hacer `git push` para subir los cambios en cualquiera de las ramas presentes.

- Borrar una rama en repositorio remoto:

    ```
    $ git push origin --delete <branchName>
    ```

=== Modificar la historia

- Modificar el mensaje del último commit. Se abrirá un editor en el que modificar el mensaje. También se puede escribir el mensaje a mano:

    ```
    $ git commit --amend [--m "Nuevo mensaje"]
    ```

- Deshacer el último commit (sólo la acción del commit, dejando los ficheros sin modificar):

    ```
    $ git reset --soft HEAD^
    ```

- Descartar el último merge y volver a la situación anterior al hacer el merge:

    ```
    $ git reset --merge ORIG_HEAD
    ```

- Movernos atrás a un commit pasado, mirar los ficheros, crear una nueva rama allí (o no) y volver al commit actual:

    ```
    $ git checkout v0.0  
    # Ahora estás en un detached HEAD  
    $ git branch  
    * (no branch)  
    master  
    nueva-rama  
    $ git checkout -b v0.0.1  
    Switched to a new branch 'v0.0.1'  
    $ git branch  
    master  
    nueva-rama  
    * v0.0.1  
    $ git checkout master
    ```
