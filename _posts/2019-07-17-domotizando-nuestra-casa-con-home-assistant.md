---
layout: post
current: post
cover: assets/images/posts/2019-07-17-domotizando-nuestra-casa-con-home-assistant/header.jpg
navigation: True
title: "Domotizando nuestra casa con Home Assistant"
date: 2019-07-17 12:00:00
tags: homeassistant hassio domotica
class: post-template
subclass: 'post'
author: danimart1991
---

Admitamos algo, el ser humano es vago por excelencia, pero los informáticos lo somos aún más si cabe. Es por este motivo por lo que intentamos automatizar todo lo posible, a veces demasiado. Una frase muy común es:

> Si haces algo más de dos veces, automatiza para no tener que hacerlo una tercera.

Podemos extrapolar esta frase a nuestro hogar. ¿Cuántas veces se nos ha olvidado una luz encendida? ¿cuántas hemos perdido el mando a distancia? ¿y si nos vamos unos días de casa y nos entran a robar?

> Available in English [here](https://www.danielmartingonzalez.com/domotizing-our-house-with-home-assistant/).

Solucionar todos estos "problemas" es más accesible hoy en día que nunca, pero si empezamos a comprar "cacharros", nos encontraríamos con un móvil repleto de aplicaciones de terceros, y muchos servicios con peligro de [ser espiados](https://www.xataka.com/privacidad/amazon-admite-que-conserva-algunos-algunos-datos-alexa-forma-indefinida-incluso-usuario-elimina-grabacion-voz) o que en algún momento [dejen de dar ese servicio](https://www.xataka.com/seguridad/quedarse-encerrado-calefaccion-imprevisibles-consecuencias-apagon-google-casa-conectada), y con ninguna interconexión entre todos los sistemas.

Con el concepto de crear un sistema de control [*Open Source*](https://es.wikipedia.org/wiki/Open_Source_Definition) que lleva por bandera la privacidad y el servicio local nace [**Home Assistant**](https://www.home-assistant.io/).

## Home Assistant

![Home Assistant](/assets/images/posts/2019-07-17-domotizando-nuestra-casa-con-home-assistant/image01.jpg)

[Home Assistant](https://www.home-assistant.io/) es un software de gestión de domótica para nuestro hogar capaz de integrar gran cantidad de dispositivos y servicios, tanto de terceros como propios. Lo mejor es, que aunque de evolución lenta, es [un software de código abierto muy vivo](https://github.com/home-assistant), con una comunidad enorme, potente y con una curva de aprendizaje muy buena.

Podéis probar una *DEMO* [**aquí**](https://demo.home-assistant.io/#/lovelace/0)

Al ser un software desarrollado en [*python*](https://www.python.org/) es compatible con multitud de sistemas operativos y dispositivos. Es distribuido de diferentes maneras para hacer las delicias de los usuarios principiantes y más avanzados.

## *Hass.io* vs. *Home Assistant* vs. *Docker*

Antes de entrar en materia y dado que esto suele ser bastante complejo. Vamos a distinguir las maneras más comunes de hacer funcionar **Home Assistant**.

- [**Hass.io**](https://www.home-assistant.io/hassio/installation/) es una distribución de *Linux* optimizada para ejecutar **Home Assistant** sobre [*Docker*](https://www.nocountryforgeeks.com/contenerizacion-de-aplicaciones-en-docker/). Es decir, se ha creado una distribución a medida para que todo esté listo para instalar y funcionar. Como ventaja, es sencillo y tiene un sistema de [**Add-ons**](https://www.home-assistant.io/addons/) muy potente. Como desventaja, no tenemos acceso real al sistema operativo que ejecuta el contenedor de *Hass.io*. Es la opción de instalación que veremos en este post.
- **Home Assistant** como hemos mencionado es un software y como tal, puede instalarse en cualquier distribución *Linux* (incluso si te atreves en *Windows*), lo normal es instalar una distribución *Linux* sencilla, como [**Hassbian**](https://www.home-assistant.io/docs/hassbian/installation/) para [*Raspberry Pi*](https://www.raspberrypi.org/) por ejemplo, y luego *Home Assistant* como "aplicación", teniendo toda la potencia y control.
- [**Docker**](https://www.home-assistant.io/docs/installation/docker/) es la opción más ventajosa para algunos, pero también la más avanzada. Consiste en crear un [contenedor de *Docker*](https://www.nocountryforgeeks.com/contenerizacion-de-aplicaciones-en-docker/) con *Home Assistant*, de tal manera que tengamos un control total, y al mismo tiempo aislado de todo *Home Assistant* con las ventajas que ello conlleva.

Pese a todas estas diferencias, luego se pueden combinar, por ejemplo, instalar *Home Assistant* con *Docker* y meter *Hass.io* por encima.

## Instalando *Hass.io*

Tal y como hemos mencionado, vamos a ver como instalar **Hass.io**, la razón, tiene la instalación más sencilla y rápida. Se puede probar *Home Assistant* tras unos minutos y según aprendamos, pasar a los otros sistemas.

Lo recomendable para empezar, dada la calidad-precio, es utilizar una [**Raspberry Pi 3 B+**](https://amzn.to/2lBMWFL) con [una **MicroSD** de al menos *32Gb*](https://amzn.to/2lBMWFL). Por supuesto, se puede utilizar [*hardware* de lo más variado](https://www.home-assistant.io/hassio/installation/), pero creo que esta es una de las mejores opciones.

1. Descargamos la imagen más actual de *Hass.io* para nuestro dispositivo. En el momento de escribir este artículo, [**Hass.io 2.12** para *Raspberry Pi 3 B / B+ 32bit*](https://github.com/home-assistant/hassos/releases/download/2.12/hassos_rpi3-2.12.img.gz).

2. Descargamos e instalamos en nuestro ordenador un quemador de imágenes en tarjetas *SD*. A mí me gusta [**balenaEtcher**](https://www.balena.io/etcher).

3. Quemamos la imagen en la tarjeta *SD*. Para ello, basta con abrir el programa previamente instalado, insertar la tarjeta *SD* en el lector, seleccionar la imagen descargada en el **paso 1** y hacer clic en *Flash*. No extraigáis la tarjeta todavía.

    ![balenaEtcher](/assets/images/posts/2019-07-17-domotizando-nuestra-casa-con-home-assistant/image01.gif)

4. **Opcional**, pero que os recomiendo encarecidamente es crear un archivo para configurar la conexión de red del dispositivo, es un paso algo complicado, vamos por partes.

    Mi recomendación personal pasa por asignar una ***IP* estática** a nuestra *Raspberry Pi*. Aunque se pueden reservar *IPs* en el *Router*, creo que una mejor opción es crear un rango de *IPs* dinámicas (*DCHP*) a partir de un número alto de dispositivos, por ejemplo, ``192.168.1.128-255``, de tal manera que todas las *IPs* inferiores a *128* las podremos usar para dispositivos con *IP* estática. Todo ello conectando un cable ***Ethernet*** entre el dispositivo y el *Router*.

    1. Abrimos el directorio de la tarjeta *SD* recién creada y creamos los directorios *`CONFIG`* y dentro *`network`*. Por último, el fichero *`my-network`* dentro de este último, sin extensión. Quedará **`X:\CONFIG\network\my-network`**.
    2. Editamos el fichero **`my-network`** e incluimos el siguiente código:

        ```
        [connection]
        id=my-network
        uuid=d55162b4-6152-4310-9312-8f4c54d86afa
        type=802-3-ethernet

        [ipv4]
        method=manual
        address=192.1.1.4/24,192.168.1.1
        dns=8.8.8.8;8.8.4.4;

        [ipv6]
        addr-gen-mode=stable-privacy
        method=auto
        ```

        Cambiamos el campo `address` por la *IP* y *puerta de enlace* deseada, y el campo `dns`, por aquellas *DNS* que quedamos usar. Apuntad la *IP* que vais a asignar al dispositivo porque es bastante importante.

5. Expulsamos la tarjeta *SD* del ordenador y la introducimos dentro de la *Raspberry Pi*. La enchufamos a la corriente y esperamos, depende del dispositivo la descarga de última versión e instalación puede llevar **hasta 20 minutos**.

    ![Instalando Hass.io](/assets/images/posts/2019-07-17-domotizando-nuestra-casa-con-home-assistant/image02.jpg)

6. Mientras tanto, podemos buscar el manual de nuestro *Router* y ver si tiene disponible **mDNS**, en cuyo caso, podremos acceder a la interfaz gráfica de *Home Assistant* desde cualquier otro dispositivo accediendo con el navegador a la siguiente dirección: **http://hassio.local:8123**, en caso contrario, y dado que hemos configurado una *IP* estática en nuestro servidor, podremos acceder a la interfaz con la dirección: **http://192.168.1.4:8123** (siendo la *IP* la puesta anteriormente en el fichero de configuración de red).

## Configuración básica

Una vez instalado, se nos mostrará un par de pantallas para realizar la configuración básica de *Home Assistant*, como el usuario y la ubicación del dispositivo. Por último, iniciamos sesión y aparecerá la pantalla principal de nuestro *Home Assistant*.

![Inicio Sesión en Home Assistant](/assets/images/posts/2019-07-17-domotizando-nuestra-casa-con-home-assistant/image03.jpg)

> Cabe destacar que, durante la instalación, *Home Assistant* buscará dispositivos inteligentes en la red con los que tenga auto-integración para incluirlos directamente en su configuración. No os preocupéis, podremos añadirlos más tarde. De hecho, los usuarios más avanzados en *Home Assistant* suelen evitar estas integraciones automáticas.

## Conclusión

![Página inicial Home Assistant](/assets/images/posts/2019-07-17-domotizando-nuestra-casa-con-home-assistant/image04.jpg)

En este momento, ya tendremos disponible nuestro propio **Home Assistant** ejecutándose en un servidor local. Y con ello, todo un abanico de posibilidades que descubriremos en próximos artículos. Hemos tardado apenas unos minutos en empezar, pero pronto descubriremos todo el potencial que **Home Assistant** tiene reservado para nosotros.

Este y otros artículos complementan la explicación del [**repositorio de *GitHub***](https://github.com/danimart1991/home-assistant-config) donde se encuentra disponible la configuración de la domotización de mi casa.
