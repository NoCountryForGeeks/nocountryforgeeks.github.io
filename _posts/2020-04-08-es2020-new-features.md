---
layout: post
current: post
cover: assets/images/posts/2020-04-08-es2020-new-features/cover.png
navigation: True
title: 'ES2020: new features'
date: 2020-04-08 12:00:00
tags: javascript es2020
class: post-template
subclass: 'post'
author: anuez
---

Hoy vamos a hablar de las nuevas características que nos ha traido la reciente actualización de ECMAScript para este año 2020 (ES2020). Aunque ya sabemos que estas "mejoras" que introduce ECMA no estan 100% soportadas por los navegadores, podemos empezar a utilizarlas con Babel siempre que utilicemos la versión 7.8 o superior, sino es necesario añadir los plugins correspondientes para cada una de las características.

Estas nuevas caracteristicas las decide un comité de Ecma International (TC39) que sigue un proceso de varias [iteraciones](https://tc39.es/process-document/) hasta que finalmente sacan una nueva versión cada año. Si quieres saber más sobre esta organización, y ver en lo que están trabajando, podeis hacerlo a través de su repositorio en Github [TC39](https://github.com/tc39).

Las propuestas que han alcanzado el stage 4 este año son incluidas en [ES2020](https://github.com/tc39/proposals/blob/master/finished-proposals.md) y son las siguientes:

| Propuesta                                     | Autor                                                  | Año              |
| --------------------------------------------- | ------------------------------------------------------ | ---------------- |
| [`String.prototype.matchAll`]                 | Jordan Harband                                         | 2020             |
| [`import()`]                                  | Domenic Denicola                                       | 2020             |
| [`BigInt`]                                    | Daniel Ehrenberg                                       | 2020             |
| [`Promise.allSettled`]                        | Jason Williams<br />Robert Pamely<br />Mathias Bynens  | 2020             |
| [`globalThis`]                                | Jordan Harband                                         | 2020             |
| [`for-in mechanics`]                          | Kevin Gibbons                                          | 2020             |
| [`Optional Chaining`]                         | Gabriel Isenberg<br />Claude Pache<br />Dustin Savery  | 2020             |
| [`Nullish coalescing Operator`]               | Gabriel Isenberg                                       | 2020             |
| [`import.meta`]                               | Domenic Denicola                                       | 2020             |                   


Dejemonos de literatura y vamos a ver un poquito de código que es lo que nos gusta a todos. Algunas de estas características son maravillosas y no vais a poder creer como habeis podido vivir sin ultilizarlas hasta ahora.

## String.prototype.matchAll

**matchAll**  es un método nuevo que se añade a String que esta relacionado con una expresión regular. El método devuelve un iterador con todos los grupos a los que pertenece uno tras otro. Veamos un ejemplo:

![matchAll](/assets/images/posts/2020-04-08-es2020-new-features/matchAll.png)

## Imports dinámicos

Los **imports dinámicos** nos dan la opción en Javascript de importar modulos dinámicamente en la aplicación de forma nativa, tal como hace Webpack o Babel. Nos permiten cargar el código bajo demanda, incluso utilizarlo dentro de operadores if-else.


Esta característica permite importar un módulo y no ensuciar el espacio de nombres global.

![dynamic-import](/assets/images/posts/2020-04-08-es2020-new-features/dynamicImport.png)

## BigInt

El número entero más grande que podíamos alcanzar en Javascript era 9007199254740991 (2 elevado a 53) - 1. Ahora, podemos crear números enteros más grandes añadiendo una "n" al final del entero gracias a esta nueva característica.


![bigint](/assets/images/posts/2020-04-08-es2020-new-features/bigInt.png)

## Promise.allSettled

El funcionamiento de esta utilidad es bastante parecido a Promise.all que nos devuelve una promesa cuando todas las promesas pasadas han sido ejecutadas.
El problema que tiene Promise.all es que en el momento que haya una sola promesa rejected el resto de promesas no se ejecutan, pero gracias a Promise.allSettled solucionamos el problema, ya que todas las promesas se ejecutaran independientemente de si su estado es `resolved` o `rejected`.

![allsettled](/assets/images/posts/2020-04-08-es2020-new-features/promise.allSettled.png)

## globalThis

El acceso al objeto **this** en JavaScript depende del entorno, usando `window` en el navegador, `global` en Nodejs o `self` en el web worker, y no había una forma estandarizada para acceder a este objeto global hasta ahora.

![globalThis](/assets/images/posts/2020-04-08-es2020-new-features/globalThis.png)

## Orden definido para For-in 

La especificación ECMA no decía en que orden podía ejecutarse la sentencia for-in. Aunque los navegadores habían estandarizado un orden previamente, ahora es oficialmente standard con ES2020.

##Operador Nullish Coalescing

Este operador **??** es un operador lógico muy parecido al operador OR **||** con la salvedad de que **??** solo comprueba verdaderos valores nulos como son null y undefined, mientras que los valores falsos como son 0, false o NaN ejecutarían la segunda parte de la sentencia.  

![nullish-coalescing](/assets/images/posts/2020-04-08-es2020-new-features/nullishCoalesce.png)


## Optional Chaining

La sintaxis con optional chaining permite acceder a cualquier propiedad de cualquier nivel del objeto sin tener que preocuparse si existe o no. En caso de no existir simplemente devuelve `undefined`. *¡¡Ouh Mama, menuda obra de orfebrería!!*
El código ahora nos queda mucho más limpio y evitamos utilizar y/o encadenar el operador AND para acceder a propiedades más profundas de un objeto. **Esta sintaxis no es exclusiva para acceder a propiedades de objetos, tambien se puede utilizar en llamadas a funciones o arrays.**

 
![chaining](/assets/images/posts/2020-04-08-es2020-new-features/optionalChaining.png)


## Import.meta

Ahora puedes acceder a la meta-información del módulo usando el objeto **import.meta.**

![import.meta](/assets/images/posts/2020-04-08-es2020-new-features/import.meta.png)


Pues estas son, todas las funcionalidades que han alcanzado el stage 4 y finalmente salen en la versión ECMAScript 2020. Si os habeis quedado con ganas de más, podeis echar un vistazo a las propuestas que hay en el resto de stages y que en un futuro puede que salgan.