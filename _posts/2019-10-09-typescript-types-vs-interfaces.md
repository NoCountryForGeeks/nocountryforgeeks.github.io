---
layout: post
current: post
cover: assets/images/posts/2019-10-09-typescript-types-vs-interfaces/header.jpg
navigation: True
title: 'Typescript: types vs interfaces'
date: 2019-10-09 12:00:00
tags: react typescript
class: post-template
subclass: 'post'
author: anuez
---

[Typescript](http://www.typescriptlang.org/) , es un "superset" del lenguaje Javascript cuyo objetivo es esencialmente añadir un tipado estático. Es un lenguaje en actual evolución, actualmente en la versión 3.6,  amado y odiado en partes iguales por los programadores, pero de esta guerra no vamos a hablar por el momento.

Debido a las requisitos del proyecto en el que se encuentra un servidor, ⚛️ [React](https://es.reactjs.org/) con Typescript para la SPA, me he visto obligado a programar en este lenguaje y me han surgido una serie de preguntas respecto a la utilización de **alias de tipos e interfaces** que me gustaría compartir.

Los tipos y las interfaces han evolucionado a lo largo de las versiones y actualmente son muy similares, aunque todavía mantienen unas pequeñas diferencias: las interfaces son más ***"extensibles"*** debido a la posibilidad de unir sus declaraciones, y los tipos son más ***"componibles"*** debido a la posibilidad de unir los tipos. 💥

Veamos algunas diferencias con código:

1. Los tipos pueden declararse como tipos primitivos, uniones o tuplas mientras que las interfaces no.

![diferencia 1](/assets/images/posts/2019-10-09-typescript-types-vs-interfaces/diff-1.png)


2.  Se tarda menos en escribir un tipo que una interfaz.

![diferencia 2](/assets/images/posts/2019-10-09-typescript-types-vs-interfaces/diff-2.png)


3. Los tipos pueden crear interseccion con otros tipos, mientras que las interfaces no.

![diferencia 3](/assets/images/posts/2019-10-09-typescript-types-vs-interfaces/diff-3.png)

4. La unión en la declaración de una interfaz no funciona con los tipos.

![diferencia 4](/assets/images/posts/2019-10-09-typescript-types-vs-interfaces/diff-4.gif)

5. No puedes extender una interfaz con un tipo si se usa el operador unión en su definición de tipo.

![diferencia 5](/assets/images/posts/2019-10-09-typescript-types-vs-interfaces/diff-5.gif)

6. La palabra clave **in** puede usarse para iterar sobre todos los elementos en una unión de claves. Podemos utilizar esta función para generar mapped types. Con interfaces no se puede.

![diferencia 6](/assets/images/posts/2019-10-09-typescript-types-vs-interfaces/diff-6.gif)

7. La sintaxis para **implementar** una clase con un tipo o con una interfaz es la misma. Pero la sintaxis para **extender** una clase no.

![diferencia 7](/assets/images/posts/2019-10-09-typescript-types-vs-interfaces/diff-7.png)

## Conclusión

Debido a estas pequeñas diferencias entre tipos e interfaces, la decisión de usar una sobre otra generalmente depende de su estilo de programación. Si escribes código orientado a objetos, usa interfaces, si escribes código funcional, usa alias de tipo.

¿Y que pasa con React? React es un lenguaje funcional por naturaleza. Los componentes funcionales generalmente son preferibles a los componentes basados en clases. Los Hooks en React son funciones que sólamente se utilizan sobre componentes funcionales, los HOC, Redux, funciones puras, etc, salen de una programación funcional.

📢 **Por todas estas razones, se deben usar los alias de tipos en vez de las interfaces en todas tus aplicaciones de React.** 📢
