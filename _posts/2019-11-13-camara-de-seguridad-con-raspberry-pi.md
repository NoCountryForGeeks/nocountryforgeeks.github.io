---
layout: post
current: post
cover: assets/images/posts/2019-11-13-camara-de-seguridad-con-raspberry-pi/header.jpg
navigation: True
title: 'Cámara de seguridad con Raspberry Pi'
tags: homeassistant hassio domotica raspberrypi
class: post-template
subclass: 'post'
author: danimart1991
---

[***Raspberry Pi***](https://www.raspberrypi.org/) (en todas sus versiones) es uno de los micro-ordenadores más potentes y versátiles del momento. Gracias a la comunidad se puede conseguir casi cualquier cosa, y hoy, vamos a ver como convertirlo en una potente cámara de seguridad que nada tiene que envidiar a las disponibles en el mercado a precios mucho más altos.

> Available in English [here](https://www.danielmartingonzalez.com/security-camera-with-raspberry-pi/).

## Materiales

![RPi Zero W + Cámara](/assets/images/posts/2019-11-13-camara-de-seguridad-con-raspberry-pi/image01.jpg)

Yo tengo un *Kit* de *Raspberry Pi Zero W* y [una cámara clónica](https://amzn.to/2WLRpEi), pero como es difícil de encontrar hoy en día, podemos buscar otra alternativa. Es recomendable comprar un pack con disipador y/o ventilador, ya que la carga gráfica y de procesador que supone una cámara de vigilancia hace que se caliente. Es recomendable poner la cámara fuera de la caja por el mismo problema de temperatura. Un ejemplo de lo que podemos comprar:

- *Kit Raspberry Pi 3 B+* con tarjeta, disipador, cargador, caja...: [https://amzn.to/2qn3wLA](https://amzn.to/2qn3wLA)
- Cámara *RPi Noir V2* con visión nocturna (oficial): [https://amzn.to/2rePix1](https://amzn.to/2rePix1)
- Caja para cámara: [https://amzn.to/2NEoM7R](https://amzn.to/2NEoM7R)

El montaje es bastante sencillo ya que la propia *Raspberry Pi* cuenta con un conector específico para cámaras.

![Montaje RPi + Cámara](/assets/images/posts/2019-11-13-camara-de-seguridad-con-raspberry-pi/image02.jpg)

> Algunas *Webcam USB* son compatibles con el sistema operativo que vamos a instalar. Si tienes alguna disponible, como la instalación son 5 minutos, puedes probar.

## *motionEyeOS*

Aunque hay muchos sistemas operativos y software para la tarea que nos concierne. Tras probar varios, en mi opinión, el que tiene una comunidad más activa, más sencillo y a la vez potente es sin duda [***motionEyeOS***](https://github.com/ccrisan/motioneyeos).

![motionEyeOS](/assets/images/posts/2019-11-13-camara-de-seguridad-con-raspberry-pi/image03.jpg)

***motionEyeOS*** es una distribución de *Linux*, que está basada en *BuildRoot*, usa *motion* como *backend* y *motionEye* como *frontend*. Una ventaja pues nos ahorramos configuraciones innecesarias, y tenemos todo lo necesario empaquetado para nuestro proyecto.

## Preparación

1. Descargamos la imagen acorde a nuestro dispositivo de [https://github.com/ccrisan/motioneyeos/releases](https://github.com/ccrisan/motioneyeos/releases)

2. Usando [***balenaEtcher***](https://www.balena.io/etcher/), grabamos la imagen descargada en una tarjeta *microSD*.

    ![Pasos balenaEtcher](/assets/images/posts/2019-11-13-camara-de-seguridad-con-raspberry-pi/image04.gif)

3. En el caso de querer usar una conexión *WiFi*, es necesario configurar la red previamente. Una vez grabada la imagen, abrimos el dispositivo con el explorador de archivos (es posible que tengamos que expulsar y volver a introducir la tarjeta en el ordenador) y creamos el archivo `wpa_supplicant.conf`. Dentro incluimos el siguiente código (sustituyendo las variables necesarias):

    ```conf
    update_config=1
    ctrl_interface=/var/run/wpa_supplicant

    network={
            scan_ssid=1
            ssid="NOMBRE_RED_WIFI"
            psk="PASSWORD_WIFI"
    }
    ```

4. Introducimos la tarjeta en la *Raspberry Pi*, encendemos y esperamos... Una vez pasados unos minutos, buscamos nuestro dispositivo en la lista de dispositivos conectados del *Router* para conocer su *IP*.

    ![Listado dispositivos Router](/assets/images/posts/2019-11-13-camara-de-seguridad-con-raspberry-pi/image05.jpg)

## Primer inicio

Accedemos a la *IP* que acabamos de encontrar con un navegador *Web* desde nuestro ordenador y nos mostrará una imagen parecida a esta:

![Ventana motionEyeOS](/assets/images/posts/2019-11-13-camara-de-seguridad-con-raspberry-pi/image06.jpg)

> En el caso de que no se muestre la imagen de la cámara, es posible que el dispositivo esté alejado del punto de acceso *WiFi*, o que hayamos conectado mal la cámara a la *Raspberry Pi*.

Para empezar a configurar el sistema, haz clic en el icono de persona que se encuentra arriba a la izquierda e inicia sesión con el usuario ***admin***, dejando el campo ***password*** en blanco.

![Inicio de sesión](/assets/images/posts/2019-11-13-camara-de-seguridad-con-raspberry-pi/image07.jpg)

Si hacemos clic en el menú "hamburguesa" de arriba a la izquierda se nos mostrarán varias opciones.

## Configuración básica

*motionEyeOS* necesita pequeñas configuraciones básicas para facilitar accesos posteriores, asegurar el sistema y dejar todo preparado para usos más avanzados.

1. Cambiar el campo **TimeZone** a la correspondiente a nuestro país.

2. Añadir una contraseña a los usuarios `admin` y `user`. El primero nos dará acceso a todas las configuraciones, el segundo, es el usuario que debemos usar cuando solo queremos ver la cámara, pero no configurarla.

    ![Configuración usuarios y zona horaria](/assets/images/posts/2019-11-13-camara-de-seguridad-con-raspberry-pi/image08.jpg)

3. Cambiamos la configuración de red para asignar una *IP* manual al sistema. Opcionalmente también puede reservarse una *IP* desde el *Router*.

    ![Configuración de red](/assets/images/posts/2019-11-13-camara-de-seguridad-con-raspberry-pi/image09.jpg)

4. Reiniciamos.

5. Una vez reiniciado, entramos de nuevo con la *IP* y usuario `admin` que acabamos de configurar y hacemos clic en *Check* por si existen actualizaciones del sistema.

    ![Actualización motionEyeOS](/assets/images/posts/2019-11-13-camara-de-seguridad-con-raspberry-pi/image10.jpg)

## Configuración avanzada

Ahora vamos a realizar las configuraciones necesarias para optimizar el sistema y el ***Streaming* de vídeo**.

1. Primero vamos a activar o desactivar solo los servicios que queramos utilizar. Por el momento, a mí solo me interesa conservar los servicios de ***SSH*** (por si falla algo poder acceder por *terminal*), ***Video Device***, ***Text Overlay*** y **Video *Streaming***.

    ![motionEyeOS configuración servicios](/assets/images/posts/2019-11-13-camara-de-seguridad-con-raspberry-pi/image11.jpg)

2. Dado que solo tenemos una cámara en el sistema, vamos a poner una sola fila y columna en la interfaz.

    ![motionEyeOS configuración interfaz](/assets/images/posts/2019-11-13-camara-de-seguridad-con-raspberry-pi/image12.jpg)

3. Vamos a configurar la imagen de entrada acorde al *hardware* que estamos usando. Dado que depende del dispositivo, así como de la cámara conectada, lo ideal es ir jugando con los valores. Para el caso de mi *Raspberry Pi Zero W* y la cámara clónica que uso he puesto una resolución de *720p* (1280x720 pixeles) y una tasa de cuadros por segundo de 5.

    ![motionEyeOS configuración Video Device](/assets/images/posts/2019-11-13-camara-de-seguridad-con-raspberry-pi/image13.jpg)

4. Por último, dentro de *Streaming*, configuramos la misma resolución y tasa de cuadros por segundo que hemos indicado antes (o la que más nos interese), una seguridad básica (*basic*) y un puerto.

    ![motionEyeOS configuración Video Streaming](/assets/images/posts/2019-11-13-camara-de-seguridad-con-raspberry-pi/image14.jpg)

Con este último punto podremos acceder al *Streaming* usando la *Url* y puerto desde cualquier navegador usando las credenciales del usuario `user`.

![motionEyeOS Video Streaming](/assets/images/posts/2019-11-13-camara-de-seguridad-con-raspberry-pi/image15.jpg)

## Integración en Home Assistant

Una vez configurado el *Streaming*, la integración con [***Home Assistant***](https://www.home-assistant.io/) es muy sencilla. Basta con modificar el archivo `configuration.yaml` y añadir las siguientes líneas (sustituyendo según convenga):

```yaml
camera:
  - platform: mjpeg
    name: Camara Ejemplo
    mjpeg_url: http://XXX.XXX.XXX.XXX:XXXX # Url Streaming
    username: user
    password: PASSWORD_USER
    authentication: basic
```

Esto creará la entidad `camera.camara_ejemplo` en nuestro servidor *Home Assistant*. Para mostrarlo en la interfaz, si tenemos [*Lovelace*](https://www.home-assistant.io/lovelace/) en modo edición podemos usar la tarjeta [*Picture Entity*](https://www.home-assistant.io/lovelace/picture-entity/):

```yaml
type: picture-entity
entity: camera.camara_ejemplo
```

![Home Assistant Camera Picture Entity](/assets/images/posts/2019-11-13-camara-de-seguridad-con-raspberry-pi/image16.jpg)

## Conclusión

Hemos visto como en unos sencillos pasos hemos configurado de manera básica y barata una cámara de seguridad y su integración con *Home Assistant*. Aún nos queda por ver todo el potencial disponible por *motionEyeOS*, pero eso lo dejamos para más adelante.
