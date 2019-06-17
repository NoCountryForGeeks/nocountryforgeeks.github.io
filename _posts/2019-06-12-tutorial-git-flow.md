---
layout: post
current: post
cover: assets/images/posts/2019-06-12-tutorial-git-flow/header.jpg
navigation: True
title: "Tutorial - Trabajando con ramas"
date: 2019-06-10 12:00:00
tags: git gitflow
class: post-template
subclass: "post"
author: maktub82
---

¡Hola a todos! Es importante cuando estamos trabajando con Git **tener una política de ramas con la que el equipo se sienta cómodo**. En otro post profundizaremos en [Gitflow](https://es.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow), pero hoy os voy a dar unas nociones básicas de cómo empezar a trabajar con ramas y os dejaré un tutorial práctico para que podáis probarlo vosotros mismos.

## Política de ramas

Cuando trabajamos con **Gitflow tenemos varias ramas que nos sirven para diferentes propósitos**: implementar una funcionalidad, corregir un bug en producción, preparar una release...

![Gitflow](/assets/images/posts/2019-06-12-tutorial-git-flow/gitflow.png)

Nosotros para este tutorial, vamos a simplificarlo bastante. Vamos a trabajar con una rama principal (master) y vamos a ir implementando funcionalidades mediante *feature branches*.

![Feature branch](/assets/images/posts/2019-06-12-tutorial-git-flow/branch.png)

## Introducción

Planteamos un ejemplo con unos tests unitarios. Vamos a ir implementando la funcionalidad poco a poco, trabajando con ramas y colaborando con nuestros compañeros. El código para empezar el tutorial lo tenéis disponible en [Github](https://github.com/ikeinyyo/Talks.RecursiveGit).

Para probar el ejemplo, basta con ejecutar el script de python:

```bash
python main.py
```

A medida que vayamos avanzando en el tutorial, iremos completando los métodos, haciendo que pasen todos los tests.

### Squash

El Squash es una técnica que nos permite agrupar los cambios de varios `commit` en uno.

El Squash se hace a través de un `rebase` interactivo, que nos da mayor control de cómo se ejecuta la reorganización del trabajo.

El comando básico es:

```bash
git rebase -i HEAD~{number_of_commits}
```

Lo que indicamos es el número de `commit` que queremos agrupar. Para que sea más sencillo y no tener que contar cuántos `commit` quiero agrupar, podemos indicar contra qué rama queremos que se haga la resta. De tal forma que nos agrupa aquellos `commit` que están por delante en la rama actual.

```bash
git rebase -i origin/master
```

### Nueva rama de feature

Estando en `master`, creamos una nueva rama para desarrollar la funcionalidad que afecta a las operaciones de las listas.

```bash
(master)$ git checkout -b feature/list
```

### Count

Lo primero que vamos a hacer es implementar la función de `count`:

```python
def count(input):
    try:
        input[0]
        return 1 + count(input[1:])
    except:
        return 0
```

Una vez terminada, hacemos un `commit`:

```bash
(feature/list)$ git commit -a -m "Add count"
```

### Sum

A continuación, implementamos la función `sum` y hacemos `commit`:

```python
def sum(input):
    try:
        input[0]
        return input[0] + sum(input[1:])
    except:
        return 0
```

```bash
(feature/list)$ git commit -a -m "Add sum"
```

### Max/Min

Implementamos las funciones `max` y `min`. A continuación, hacemos `commit`:

```python
def max(input):
    items_count = count(input)
    if items_count == 0:
        return 0 # Only for 0 items list
    elif items_count == 1:
        return input[0]
    else:
        max_other = max(input[1:])
        return max_other if max_other > input[0] else input[0]

def min(input):
    items_count = count(input)
    if items_count == 0:
        return 0 # Only for 0 items list
    elif items_count == 1:
        return input[0]
    else:
        min_other = min(input[1:])
        return min_other if min_other < input[0] else input[0]
```

```bash
(feature/list)$ git commit -a -m "Add max and mix"
```

### Sort

A continuación, implementamos la función `sort` y hacemos `commit`:

```python
def sort(input):
    items_count = count(input)
    if items_count <= 1:
        return input
    return sort([e for e in input[1:] if e <= input[0]]) + [input[0]] + sort([e for e in input[1:] if e > input[0]])
```

```bash
(feature/list)$ git commit -a -m "Add sort"
```

### Squash

Una vez hemos terminado la funcionalidad, podemos hacer el `rebase -i` para agrupar los commits.

**Nota:** Como lo queremos hacer contra `origin/master`, hacemos un `fetch` para asegurarnos que la rama está actualizada.

```bash
(feature/list)$ git fetch
(feature/list)$ git rebase -i origin/master
```

A continuación, nos aparecerá un listado con los `commit` que hay para que podamos elegir cómo queremos que se haga el `rebase`. Debemos dejar el primer marcado como pick y el resto los tenemos que marcar como squash.

```bash
pick 43fa23 Add count
s 13fa1e Add sum
s 2aff1e Add max and min
s 245eef Add sort
```

Simplemente añadimos el mensaje del `commit`:

```bash
Add list functions
```

### Push y Pull Request

Una vez tenemos nuestra rama preparada para integrarse con la rama principal, en este caso `master`, tenemos dos opciones: o hacer un `merge` en `master` o subir la rama y hacer una Pull Request.

#### Merge

Lo que tenemos que hacer es ir a la rama `master`, asegurarnos de que está actualizada, hacer un `merge` y subir `master` al remoto.

```bash
(feature/list)$ git checkout master
(master)$ git fetch
(master)$ git pull
(master)$ git merge feature/lists
(master)$ git push origin master
```

#### Pull Request

En este caso, únicamente subimos la rama `feature/list` y hacemos la gestión de la Pull Request desde el portal de Azure DevOps o Github.

```bash
(feature/list)$ git push origin feature/list
```

### Rebase

Otro punto importante cuando trabajamos con Git es poder adaptarnos a que haya código que necesitemos y que se encuentren en otras ramas.

Para poder traernos ese código utilizaremos el comando `rebase`.

Al igual que hemos hecho antes, vamos a implementar la nueva funcionalidad en diferentes `commit` y luego haremos un Squash.

Primero creamos una nueva rama desde `master`.

```bash
(master)$ git checkout -b feature/calc
```

### Mult

Impementamos el método `mult` y hacemos un `commit`.

```python
def mult(n1, n2):
    if n1 == 0 or n2 == 0:
        return 0
    if n1 < 0 and n2 < 0:
        return abs(n1) + mult(abs(n1), abs(n2)-1)
    elif n2 < 0:
        return n2 + mult(n1-1, n2)
    else:
        return n1 + mult(n1, n2-1)
```

```bash
(feature/calc)$ git commit -a -m "Add mult"
```

### Div

A continuación, implementamos la función `div` y hacemos un `commit`.

```python
def div(n1,n2):
    if n2 == 0:
        raise ZeroDivisionError
    elif n1 == 0:
        return 0
    elif abs(n1) < abs(n2):
        return 0
    elif n1 < 0 and n2 < 0:
        return 1 + div(abs(n1)-abs(n2), abs(n2))
    elif n1 < 0 or n2 < 0:
        return -1 + div(n1+n2, n2)
    else:
        return 1 + div(n1-n2, n2)
```

```bash
(feature/calc)$ git commit -a -m "Add div"
```

### Squash y crear una Pull Request

Lo siguiente que hacemos es hacer un Squash de nuestros dos `commit` y subir la rama `feature/calc` para hacer una Pull Request.

```bash
(feature/calc)$ git fetch
(feature/calc)$ git rebase -i origin/master
(feature/calc)$ git push origin feature/calc
```

Hacemos una Pull Request, pero nosotros tenemos que seguir trabajando.

### Nueva rama y Rebase

Una vez que hemos hecho la Pull Request, tenemos que dejar tiempo para que nuestros compañeros puedan revisarla y validarla. Pero nosotros tenemos que seguir trabajando.

Para ello vamos a crear una nueva rama desde `master` y hacernos un `rebase` desde `origin/feature/calc` para poder seguir con el código que en que ya estábamos trabajando.

```bash
(feature/calc)$ git checkout master
(master)$ git fetch
(master)$ git pull
(master)$ git checkout -b feature/calc-1
(feature/calc-1)$ git rebase origin/feature/calc
```

### Pot

Implementamos el comando `pot` y hacemos un `commit`.

```python
def pot(b,e):
    if e == 0:
        return 1
    elif b == 0:
        return 0
    elif e < 0:
        return 1 / pot(b, -e)
    else:
        return mult(b, pot(b, e-1))
```

```bash
(feature/calc-1)$ git commit -a -m "Add pot"
```

### Cambios en la Pull Request

En ese momento, nos sugieren cambios en la Pull Requst. Nos piden implementar nuestro propio método `abs`. Lo primero que tenemos que hacer es volver a la rama `feature/calc` para implementar ahí la nueva funcionalidad.

```bash
(feature/calc-1)$ git checkout feature/calc
```

Añadimos los test unitarios para el método `abs`:

```python
{'test': lambda: abs(0), 'expected': 0, 'method': "abs of zero"},
{'test': lambda: abs(5), 'expected': 5, 'method': "abs of positive number"},
{'test': lambda: abs(-5), 'expected': 5, 'method': "abs of negative"},
```

E implementamos la función `abs`:

```python
def abs(n):
    if n >= 0:
        return n
    else:
        return -n
```

Por último, hacemos un `commit`.

```bash
(feature/calc)$ git commit -a -m "Add abs"
```

De nuevo, hacemos un Squash y subimos el código. Esta vez, como hemos cambiado la historia de Git, tendremos que añadir el argumento `-f` al `push`.

```bash
(feature/calc)$ git fetch
(feature/calc)$ git rebase -i origin/master
(feature/calc)$ git push origin feature/calc -f
```

Por fin nuestros compañeros nos aprueban la Pull Request, así que ya podemos completarla.

### Actualizar mi rama con el trabajo integrado en master

Lo siguiente que hacemos, es volver a la rama `feature/calc-1` y hacer un `rebase` de `origin/master`, después de hacer un `fetch` para tener la rama actualizada.

```bash
(feature/calc)$ git checkout feature/calc-1
(feature/calc-1)$ git fetch
(feature/calc-1)$ git rebase origin/master
```

### Fac

Continuamos con nuestro trabajo de forma normal en la rama `feature/calc-1`. Añadimos ahora, la función `fac` y hacemos `commit`.

```python
def fac(n):
    if n == 0:
        return 1
    elif n < 0:
        return mult(n, fac(n+1))
    else:
        return mult(n, fac(n-1))
```

```bash
(feature/calc-1)$ git commit -a -m "Add fac"
```

### Finalizamos nuestro trabajo: Squash, push y Pull Request

Hacemos el Squash y subimos la rama.

```bash
(feature/calc-1)$ git fetch
(feature/calc-1)$ git rebase -i origin/master
(feature/calc.1)$ git push origin feature/calc-1
```

Creamos la Pull Request y, tras que nos aprueben nuestros compañeros, la completamos para finalizar nuestro trabajo.

## Siguientes pasos

El objetivo de este tutorial es ver el uso de las *feature branches* y poder mostrar cómo se trabaja con ellas y cómo podemos integrar código en las ramas principales.

Cualquier duda sobre el tutorial, por favor, ponedla en los comentarios. En siguientes post podemos ver más sobre Gitflow y cómo trabajar con otro tipo de ramas.

¡Nos vemos en el futuro!





