---
layout: post
current: post
cover: assets/images/posts/2017-08-29-micro-post-organizando-las-referencias-en-tus-clases/header.jpg
navigation: True
title: "[Micro-Post] Organizando las referencias en C#"
date: 2017-08-29 12:00:00
tags: csharp visualstudio micropost
class: post-template
subclass: 'post'
author: danimart1991
---

Hay muchas formas de mejorar nuestro código, algunas personas abogan por tener un código con una funcionalidad concreta y fiable, otros por un código limpio, y por qué no, otros simplemente por un código que funcione y punto.

> Available in English [here](https://www.danielmartingonzalez.com/micro-post-organizing-the-references-in-your-classes/).

En **No Country For Geeks** creemos que el código tiene que ser arte, y no nos referimos a un cuadro abstracto que cada persona interprete de una manera; hablamos de una sinfonía en el que cada variable, cada método, cada línea, sea un compuesto de funcionalidad, belleza, simpleza y elegancia que alcancen la excelencia en lo posible.

Entre estas definiciones, entra la organización. Y algo muy sencillo de mejorar es la organización de las referencias *using* en nuestro código.

![](/assets/images/posts/2017-08-29-micro-post-organizando-las-referencias-en-tus-clases/bad-organization.png)

Lo adecuado en estos casos es que las referencias se organicen siguiendo un **orden alfabético**, de este modo de un vistazo se puede buscar fácilmente la referencia en la que estamos interesados.
Por otro lado, algo opcional, pero que está considerado como una buena práctica, es que las referencias *System* aparezcan antes que cualquier otra referencia, y de nuevo, ordenadas alfabéticamente.

Con **Visual Studio**, toda esta organización se puede hacer de una manera muy sencilla.

1. Para que las referencias *System* aparezcan al inicio de la lista de referencias, basta con dirigirnos a *Tools -> Options* y en la nueva ventana que se nos despliega, buscar la opción *Text Editor -> C# -> Advanced* y activar el check: *"Place 'System' directives first when sorting usings"*.

    ![](/assets/images/posts/2017-08-29-micro-post-organizando-las-referencias-en-tus-clases/system-options.png)

2. Una vez estamos en una clase en la que queramos organizar las referencias, basta con utilizar un atajo de teclado poco conocido pero muy útil: *Ctrl+R, Ctrl+G*. Al hacerlo, veremos cómo se limpian las referencias no utilizadas en nuestra clase, las referencias a *System* se dispondrán en primera instancia en el listado, y todas las referencias se organizarán por orden alfabético.

    ![](/assets/images/posts/2017-08-29-micro-post-organizando-las-referencias-en-tus-clases/good-organization.png)

De este modo hemos organizado todas nuestras referencias por clase de una manera muy cómoda y rápida.

Algo que me gusta hacer cuando modifico una clase es presionar *Ctrl+R, Ctrl+G* (organizar referencias), *Ctrl+K, Ctrl+D* (dar formato al documento), y finalmente *Ctrl+S* para guardar el documento. Con esto quedaría todo nuestro código organizado y limpio de una manera muy rápida.

Nota: Dependiendo de la configuración de **Visual Studio**, es posible que el atajo de teclado para dar formato al documento sea *Ctrl+E, Ctrl+D*.
