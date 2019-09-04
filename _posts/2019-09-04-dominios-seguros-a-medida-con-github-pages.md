---
layout: post
current: post
cover: assets/images/posts/2019-09-04-dominios-seguros-a-medida-con-github-pages/header.jpg
navigation: True
title: 'Dominios seguros a medida con GitHub Pages'
date: 2019-09-04 12:00:00
tags: devops github
class: post-template
subclass: 'post'
author: danimart1991
---

[*GitHub Pages*](https://pages.github.com/) es la posibilidad de crear **un sitio web estático desde un repositorio** que ofrece [*GitHub*](https://github.com/). Esto es útil cuando tienes una página personal, un proyecto con su documentación, o un blog (como este).

A su vez, gracias a una asociación con [*Let's Encrypt*](https://letsencrypt.org/) de [*Internet Security Research Group (ISRG)*](https://www.abetterinternet.org/), se ofrece una encriptación *HTTPS* totalmente gratuita que incluye además, una red de distribución de contenido (*CDN*) respaldados por [*Cloudfare*](https://www.cloudflare.com/). Podrás conseguir tener un sitio web estático escalable, redundante, rápido y con una encriptación segura.

> Available in English [here](https://www.danielmartingonzalez.com/custom-secure-domains-with-github-pages/).

Desde la propia configuración del repositorio de *GitHub* se ofrecen las opciones para configurar el sitio de manera sencilla. Basta con indicar la rama desde la que se va a desplegar el sitio web, activar la opción "*Enforce HTTPS*", y si así lo deseamos un dominio personalizado (que previamente deberemos comprar en nuestro proveedor de dominios y configurar para que redireccione a nuestro sitio web de *GitHub Pages*).

![Configuración GitHub Pages](/assets/images/posts/2019-09-04-dominios-seguros-a-medida-con-github-pages/image01.jpg)

> En este artículo no vamos a ver cómo funciona la compilación de sitios web estáticos en *GitHub Pages*, así como de *Jekyll* o sus *themes*. Por el momento con una simple página `index.html` con cualquier texto es suficiente para probar.

## *CNAME* y sus restricciones

Si quieres tener más de un repositorio con su sitio web estático y dominios para cada uno, que sepas que no es tarea fácil, de ahí este artículo; puesto que la manera de trabajar de *GitHub Pages* supone que la página principal de tu usuario u organización (por defecto un repositorio con nombre `usuario.github.io`) será de tipo: *https://usuario.github.io/* y el resto de repositorios (considerados proyectos) seguirán un esquema de subdominios de tipo: *https://usuario.github.io/nombrerepositorio/*. Se sigue el siguiente esquema:

| Repositorio | URL |
|---|---|
| https://github.com/nocountryforgeeks/nocountryforgeeks.github.io | https://www.nocountryforgeeks.github.io/ |
| https://github.com/nocountryforgeeks/proyecto1 | https://www.nocountryforgeeks.github.io/proyecto1/ |
| https://github.com/nocountryforgeeks/proyecto2 | https://www.nocountryforgeeks.github.io/proyecto2/ |
| ... | ... |

A priori no supone un problema. Si compramos dos dominios (uno para cada repositorio) y según la teoría, bastaría con poner el dominio personalizado en las opciones de configuración del repositorio para *GitHub Pages*.

![Configuración dominio GitHub Pages](/assets/images/posts/2019-09-04-dominios-seguros-a-medida-con-github-pages/image02.jpg)

Sin embargo, este cambio lo que crea es un archivo *CNAME* en la raíz de nuestro repositorio, que básicamente lo que le dice a nuestro sitio web es el dominio al que apunta. En la configuración *DNS* de nuestro proveedor de dominios, deberemos hacer lo propio indicando la web a que queremos redireccionar, y aquí es cuando entran en juego los subdominios, ya que una redirección *CNAME* no acepta subdominios (*https://dominio.com/subdominio*), y por tanto, no podremos crear la redirección con el repositorio secundario. Dando errores tanto en la propia redirección como en la activación de la encriptación *HTTPS*.

![CNAME Github Pages](/assets/images/posts/2019-09-04-dominios-seguros-a-medida-con-github-pages/image03.jpg)

## Usando registros *DNS* de tipo *A*

La solución pasa por utilizar otro tipo de registros *DNS* a parte de *CNAME*. Con ello, indicamos que se debe crear una jerarquía que quedará oculta tras los dominios personalizados y ambos repositorios dispondrán de sitios web y dominios propios.

Esta es la configuración de registros *DNS* para ambos dominios. Observa como en ambos casos se usan registros de tipo *A* apuntando a las *IPs* de *GitHub* y crea un registro *CNAME* apuntando siempre al dominio de repositorio principal (sin *www*).

![Registros *DNS*](/assets/images/posts/2019-09-04-dominios-seguros-a-medida-con-github-pages/image04.jpg)

A continuación, configura ambos dominios en la configuración de repositorio de *GitHub* para cada uno. Por último, al crear la redirección de este modo, te permitirá además usar la opción de crear un dominio seguro *HTTPS*.

| Repositorio | URL |
|---|---|
| https://github.com/nocountryforgeeks/nocountryforgeeks.github.io | https://www.nocountryforgeeks.com/ |
| https://github.com/nocountryforgeeks/proyecto1 | https://www.proyecto1.com/ |
| https://github.com/nocountryforgeeks/proyecto2 | https://www.proyecto2.com/ |
| ... | ... |

## Conclusión

Llegados a este punto tendremos tantos dominios personalizados como queramos apuntando a nuestros repositorios con sitios web de *GitHub Pages*. El único problema de este método es que al crear una asociación por jerarquía de subdominio, si accedemos a *https://www.nocountryforgeeks.com/proyecto1*, nos redireccionará al dominio *https://www.proyecto1.com/*, a mi parecer algo asumible.

Espero que con este artículo os ahorréis los quebraderos de cabeza que he tenido con los dominios personalizados y *GitHub Pages*.
