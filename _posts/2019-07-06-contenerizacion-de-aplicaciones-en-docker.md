---
layout: post
current: post
cover: assets/images/posts/2019-07-06-contenerizacion-de-aplicaciones-en-docker/header.png
navigation: True
title: "Contenerización de aplicaciones en Docker"
date: 2019-07-06 12:00:00
tags: Docker Containers
class: post-template
subclass: 'post'
author: jgarcia
---

# Contenerización con Docker

Antes de empezar con el tema en cuestión me gustaría agradecer a los amigos de **"No Country For Geeks"** el haberme invitado a realizar una publicación en su blog. Como yo también soy un seguidor de este sitio sé de buena tinta la calidad de los contenidos que se publican, así que espero que no me puedan los nervios y estar a la altura. Para los que se pregunten quién soy, pues decir que no soy más que un Murcianico con la inquietud de aprender continuamente nuevas tecnologías que pueda aplicar a lo que es mi hobby y al mismo tiempo mi profesión, el desarrollo de software. Y sin enrollarme más en contar mi vida comencemos con la contenerización de aplicaciones en Docker, que es a lo que hemos venido. 

La vida del desarrollador es demasiado corta como para perder el tiempo entre compilación y despliegue, en comprobar y mantener estable la perfecta armonía de librerías de código en la que un simple cambio de versión puede destruirte el edificio por los cimientos. Pues aquí vengo a enseñaros la contenerización de aplicaciones usando [Docker](https://www.docker.com/), una herramienta que alargará la esperanza de vida del programador medio al menos un poco.

Antes de entrar en materia vamos a asentar algunos conceptos base para que no sólo imitemos los comandos que se enseñarán en éste post, si no que además entendamos qué es lo que esta sucediendo por debajo.

## ¿Qué es contenerizar?

Simple y llanamente se trata de un método de virtualización que abarca a nivel de Sistema Operativo para implementar y ejecutar aplicaciones distribuidas sin lanzar una [Máquina Virtual](https://www.xataka.com/especiales/maquinas-virtuales-que-son-como-funcionan-y-como-utilizarlas) completa para cada aplicación. Para mantener todo el ecosistema de contenedores que pueden llegar a existir, se habilita un punto central de control o host con el que todos los contenedores se comunican y desde donde son gestionados.

Dentro de cada contenedor se incluye todo lo necesario para arrancar nuestras aplicaciones, es decir, archivos necesarios como **librerías de código compiladas, variables de entorno e incluso archivos necesarios para la depuración** (Si si, se puede depurar a través de un contenedor).

Como los recursos están compartidos, ya que los contenedores comparten Sistema Operativo (y pueden compartir también librerías de código), además se pueden crear contenedores de aplicaciones que no ejercerán tanta carga de trabajo en nuestro sistema puesto que no están utilizando los recursos globales, o dicho de otra manera, al no tener ejecutadas las aplicaciones directamente en nuestra máquina si no que se encuentran en una máquina virtualizada a la cual ya se le han asignado unos recursos, las aplicaciones cogerán los recursos ya asignados a dicha máquina.

![](http://mindbodyncode.com/wp-content/uploads/2019/06/docker-container-architecture-1024x575.jpg)

Al ejecutarse sobre una máquina virtual, la portabilidad entre distintos tipos de máquina es posible. Por lo que un contenedor de aplicación puede ejecutarse en cualquier sistema sin requerir cambios en el código, tan solo tendríamos inconvenientes al intentar ejecutar un contenedor Windows sobre un sistema operativo Linux. Más o menos con estos detalles ya podríamos entender como funciona la contenerización, a partir de aquí hablaremos específicamente de Docker, por ser el más popular y el que más empresas están adaptando en su sistema.

## ¿Qué es Docker?

Se trata de una plataforma libre para desarrollar y ejecutar aplicaciones en contenedores de una forma simple y sencilla. Te permite manejar tu infraestructura como manejarías tu aplicación, y esto es así porque podrías, por ejemplo, crear una instancia de Docker en la que tan solo ejecutarías una imagen de base de datos SQL, con lo que podrías arrancar dicha imagen desde un contenedor y gestionarlo desde éste sin necesidad de tener que tenerlo instalado en local o en algún servidor.

![](http://mindbodyncode.com/wp-content/uploads/2019/06/descarga.png)

Esto nos ofrece una versatilidad a la hora de desarrollar increíble, se pueden lanzar tanto nuestras aplicaciones propias en cualquier tecnología y comunicarlas con otros contenedores con diversos servicios de infraestructura sin necesidad de instalarlos en nuestra máquina, y si además te digo que ni siquiera tendrías que decirle o configurar en profundidad (Siempre que no quieras o no lo necesites) la instancia de SQL server que pudieras tener, ya que gracias al repositorio  [DockerHub](https://hub.docker.com/), en el cual se encuentran de manera libre miles de  **imágenes**  **Docker** con diversos servicios, bastaría con indicarle qué imagen queremos usar y el solito se la descargaría, instalaría, y configuraría la base para poder funcionar, y tu solo tendrías que decirle el puerto en el que quieres dicho servicio a través del contenedor.

Hablamos de una reducción de los tiempos entre los que un desarrollador implementa una nueva funcionalidad y la llegada de dicha _feature_ a producción se reducirían notoriamente. Siempre y cuando nuestro destino final en producción esté contenerizado claro, que si éste es nuestro objetivo y tenemos un sistema grande, distribuido y complejo, no es tarea sencilla seamos sinceros. Esta herramienta es la que antes comentaba que ampliaría la vida media del desarrollador, o por lo menos la hace más llevadera.

Como bien comentaba Docker nos permite ejecutar  **N**  aplicaciones simultáneamente en un host en base a contenedores, y además podríamos hacer una jerarquía de dependencias entre aplicaciones, es decir, si nuestra aplicación  **A** necesita de otra aplicación  **B**  para funcionar, podemos configurar mediante Docker y establecer que antes de ejecutar  **A**  se espere y lance previamente el contenedor  **B**. Con ésto se acabo la rutina diaria de alguien está tocando el servidor compartido y mientras realiza su tarea esta dejando sin entorno de trabajo a los demás desarrolladores (Donde seguro que aprovecharán todos para tomarse un café), cada desarrollador podría lanzar su aplicación con una dependencia a una imagen configurada del servidor y cada uno desde su máquina local.

Para concluir esta sección recapitulemos que características nos aporta Docker:

1.  **Desarrollar** aplicaciones usando contenedores, con todos sus beneficios.
2.  **Testear** usando contenedores, mediante imágenes previamente configuradas.
3.  **Llegar a producción**  con contenedores, donde es recomendable el uso de [orquestadores](https://docs.microsoft.com/es-es/dotnet/standard/microservices-architecture/architect-microservice-container-applications/scalable-available-multi-container-microservice-applications) para aprovechar todo el potencial de los contenedores.

## Primeros pasos

Pues para empezar, como es obvio, será descargarnos e instalarnos  [Docker Desktop](https://docs.docker.com/docker-for-windows/install/), la aplicación de escritorio de Docker en nuestro ordenador para empezar a trastear con contenedores. En el enlace que he añadido para Docker Desktop se muestran ambas versiones tanto para Windows como para Mac. Todos los ejemplos que mostraré serán bajo un sistema Windows, sobre el que pueden ser ejectuados contenedores tanto Windows como Linux, sin embargo en un Sistema Operativo Linux solo pueden ser lanzados contenedores Linux. Además la instalación base del cliente de Docker puede ser diferente según qué plataforma. Aun así el mismo enlace que he puesto (De la página oficial de Docker) explica detalladamente como instalarnos Docker en nuestra máquina paso por paso y de forma muy sencilla, por lo que no me detendré en todos los detalles de la instalación.

_**Nota**_:  _Existe una versión anterior del cliente de Docker para Windows llamada  **[Docker ToolBox](https://docs.docker.com/toolbox/toolbox_install_windows/)**_.  _Es posible que tengáis que utilizarla si vuestro sistema operativo Windows no es la versión Pro, Enterprise o Education. ¿Por qué? Pues sencillamente porque la versión de Docker Desktop utiliza como hipervirtualizador  [Hyper-V](https://es.wikipedia.org/wiki/Hyper-V)_,  _y éste tan solo se encuentra en estas versiones de Windows._

**_¡Nota_** **aún más importante!**:  _Si por lo que sea os instaláis Docker ToolBox, ya sea por error, por probar o porque no te queda más remedio. Y después os pasáis a Docker Desktop, la instalación previa de Docker ToolBox os creará unas variables de entorno que necesitará para poder funcionar, sin embargo, la versión nueva de Docker Desktop no necesita de dichas variables de entorno, y si por lo que sea pasáis de uno a otro esas variables de entorno convertirán vuestra vida en un infierno. Tenedlo en cuenta y_ **_¡Borradlas!_**

### ¿Que tenemos instalado?

Terminado el paso anterior ahora mismo tenemos instalado en nuestra máquina la aplicación cliente-servidor Docker Engine, todo el sistema necesario para desarrollar, contenerizar y ejecutar aplicaciones. El cual se compone de:

-   Servidor arrancado como un demonio ([Daemon](https://es.wikipedia.org/wiki/Daemon_(inform%C3%A1tica))) en nuestro sistema, que sería el encargado de ejecutar los comandos propios de Docker (docker command).
-   Un API REST actuando como interfaz para comunicarse con el demonio.
-   Entrada de línea de comandos (CLI).

![](http://mindbodyncode.com/wp-content/uploads/2019/06/docker-CLI.png)

El CLI interactua con el API REST para ejecutar las acciones deseadas por el usuario, y a su vez el API REST ejecuta las acciones en el demonio (docker daemon). Otras aplicaciones en Docker pueden comunicarse directamente con el API REST. Con ésto tenemos todo lo necesario para manejar elementos de Docker como  **imágenes**,  **contenedores**,  **networks** y  **volumenes**.

#### Imágenes en Docker

Antes ya había comentado por encima el concepto de imagen en Docker, pues bien, una imagen de Docker representa una instantánea de una máquina virtual, pero siendo esta mucho más ligera. Por ejemplo una imagen podría tener la configuración de un  **Sistema Operativo como Windows**, con  **nuestra aplicación**  instalada y una  **BD**  como puede ser  [MongoDB](https://www.mongodb.com/) de la que se nutrirá nuestra aplicación.

Estas imágenes las usará Docker para crear los contenedores, además como ya indiqué existe un repositorio público (**DockerHub**) donde habitan decenas de imágenes con aplicaciones pre-configuradas como pueden ser  **Redis, Apache, MySQL**… y un largo etc.

#### Network de Docker

Los requisitos de red de las aplicaciones y el propio entorno de la red puede resultar algo complejo de gestionar cuando hablamos de contenedores, en Docker se encuentra para facilitarnos la vida la red Docker o también denominado el modelo de red de contenedor ([Container Network Model, CNM](https://docs.docker.com/v17.09/engine/userguide/networking/#default-networks)). El CNM es el encargado de administrar la conectividad entre los contenedores Docker, y todo ello de forma abstracta para nosotros, por lo que nos podemos ir olvidando de la tarea que el mantenimiento de la red conlleva.

![](http://mindbodyncode.com/wp-content/uploads/2019/06/cables-close-up-connection-159304-300x200.jpg)

Aun así esto no significa que no podamos configurar  **nuestras direcciones de red o aplicar un cifrado en la capa de red o incluso usar descubrimiento de servicios**. Más adelante en otro articulo, es posible que os enseñe algunas de las cosas que se pueden configurar con las interfaces de red, de momento continuemos con lo básico de Docker.

#### Volúmenes

Un volumen es un mecanismo que permite persistir datos en contenedores, lo ideal es que los contenedores no contengan estado alguno de nuestras aplicaciones, ya que un contenedor es un ente que está destinado a morir, los contenedores mueren y vuelven a instanciarse con la ayuda de  **orquestadores**. Y no solo eso, si no que además es posible arrancar varias instancias de un mismo contenedor y balancear la carga de trabajo entre ellos. Es por esto por lo que no es recomendable que los contenedores contengan estado o configuración de direcciones IP estáticas.

Así el acceso a los contenedores a través de la red se realiza mediante técnicas de descubrimiento de servicios, con lo que cada vez que arranquemos un contenedor es muy probable que su dirección de red haya cambiado. Aun así en ciertas situaciones es posible que necesitemos almacenar información en los contenedores, por lo que no está de mas que conozcamos esta herramienta.

### Primera aplicación a contenerizar

Antes de nada vamos a comprobar que nuestra instalación de Docker funciona correctamente, y para ello vamos a ejecutar el siguiente comando sobre el cmd de nuestra máquina:
```bash
    docker run hello-world
```
  
  Y si todo ha salido como debería de ser, aparecerán los siguientes mensajes:

![](http://mindbodyncode.com/wp-content/uploads/2019/06/image.png)

Siguiendo un poco la trazabilidad de los mensajes lo que ha ocurrido es lo siguiente:

1.  Ha buscado si la imagen ‘_hello-world_‘ ya la tenía creada localmente la maquina host de Docker.
2.  Al no ser así, ha buscado una imagen con el mismo nombre en el repositorio público  **DockerHub**, en este caso si que la ha encontrado y se dispone a descargársela.
3.  Una vez finalizada la descarga de la imagen, Docker creará una instancia en un contenedor de esta misma y la ejecutará. Donde los últimos mensajes mostrados provienen de la misma aplicación descargada, en los que se detalla todo el proceso de comunicación en el cliente de Docker entre el CLI y la interfaz que ya os comenté.

Es importante que se entienda que la imagen ‘_hello-world_‘ es una imagen que yo no he creado y que no está en mi dispositivo local, es una imagen ya creada en el repositorio de  **DockerHub**  y la cual yo puedo usar a base de un simple comando. De la misma forma que me he descargado esta imagen que tan solo es un ejemplo, podría descargar y utilizar de forma similar una imagen de una caché  **Redis**  usando el mismo comando pero con el nombre de imagen ‘_redis_‘.

Continuamos creando una aplicación  **.Net Core** muy sencilla en la que solo estableceremos un controlador que devolverá una cadena de texto con un mensaje, esta será la aplicación que se contenerizara a continuación.

**Program.cs**
```csharp
    public class Program
    {
        public static void Main(string[] args)
        {
            CreateWebHostBuilder(args).Build().Run();
        }

        public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
            WebHost.CreateDefaultBuilder(args)
                .UseStartup<Startup>();
    }
```
**Startup.cs**
```csharp
    public class Startup
    {
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }
            else
            {
                // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
                app.UseHsts();
            }

            app.UseHttpsRedirection();
            app.UseMvc();
        }
    }
```
**GreetingsController.cs**
```csharp
    [Route("api/[controller]")]
    [ApiController]
    public class GreetingsController : ControllerBase
    {

        [HttpGet]
        public string Greetings()
        {
            return "Greetings developer! I´m a contanerized API :)";
        }
    }
```
Como se puede apreciar es un proyecto  **.Net Core**  sencillo con un simple controlador  _**GreetingsController**_, con una llamada GET en la que devolverá un mensaje de texto saludando al desarrollador que realice tal petición. A continuación lo que vamos a hacer es publicar nuestra aplicación en una carpeta, dejándola lista para ser desplegada en servicios de hosting. Para ello ejecutamos el siguiente comando:
```bash
    dotnet publish "ContanerizedVisits.csproj" -c Release -o ./publish
```
Ejecutando el comando anterior en la ruta en la que se encuentre el proyecto, lo que estamos haciendo es compilando la aplicación con la configuración establecida en el perfil de  _**Release**_  y dejando el conjunto de librerías compiladas en la carpeta ubicada en la misma ruta denominada  _**publish**_. Al ejecutar el comando deberíamos de ver por consola la siguiente salida (Si todo ha ido correctamente):

![](http://mindbodyncode.com/wp-content/uploads/2019/06/publishContanerizedApp-1024x121.png)

Y si queremos observar el resultado, podemos ver lo que hay en la carpeta  _**publish**_:

![](http://mindbodyncode.com/wp-content/uploads/2019/06/ls-publish-folder-1024x54.png)

Como vemos genera los ficheros de configuración junto con la librería compilada y el  _**web.config**_  necesario para la configuración del sitio donde se hospede. Con todos esos ficheros podemos publicar nuestra aplicación en servidores web como puede ser  **IIS ([Internet Information Service](https://es.wikipedia.org/wiki/Internet_Information_Services))**, en nuestro caso procederemos a publicarla en un contenedor Docker.

### Dockerfile

En seguida tendremos contenerizada nuestra aplicación y en funcionamiento, pero para ello vamos a necesitar el uso de un fichero de configuración el cual necesita Docker para saber cómo tiene que contenerizar nuestra aplicación. Este fichero se denomina  **Dockerfile**, y debe de ser ubicado en la ruta en la que se encuentre nuestro proyecto de arranque.

Básicamente lo que el Dockerfile contiene es un conjunto de comandos que  **podríamos ejecutar directamente desde el CLI de Docker,**  pero que como siempre suelen ser los mismos para una aplicación se genera este fichero con toda la lista de comandos a ejecutar en orden, y con esto Docker es capaz de generar automáticamente una imagen de nuestra aplicación. Nosotros utilizaremos un Dockerfile como el siguiente:

    FROM mcr.microsoft.com/dotnet/core/aspnet:2.2-nanoserver-1803 AS base
    
    WORKDIR /app
    
	EXPOSE 80
	
	COPY ./publish .
	
	ENTRYPOINT ["dotnet", "ContanerizedVisits.dll"]

Existe una cantidad variada de comandos de configuración que se pueden establecer en un Dockerfile, pero no los voy a explicar todos porque para eso se encuentran las referencias de documentación oficial como puede ser  [Docker Docs](https://docs.docker.com/engine/reference/builder/)  en este caso. Si que iré explicando los que se usarán para crear la imagen de la aplicación de ejemplo mostrados.

**FROM**. Todos los Dockerfile deben de comenzar con éste tipo de instrucción. Debido a que en éste punto estamos especificando qué imagen tomaremos de base para construir la aplicación. ¿Que quiere decir todo esto? Básicamente lo que estamos haciendo es indicarle a Docker que  **necesitará el compilador de .Net Core 2.2**, en nuestro caso, para poder crear la aplicación. ¿Recordáis lo que os comenté cuando usamos la imagen de  _hello-world_? Pues aquí esta haciendo algo parecido, buscará si tenemos localmente el compilador de .Net Core y si no, se lo descargará del repositorio de  **DockerHub**.

**_Nota_**_: Nosotros ya hemos compilado la aplicación previamente y puesto en una carpeta_ **_publish_** _todos los ficheros resultantes, por lo que realmente lo que haremos será utilizar el_ **[_CLI de dotnet_](https://docs.microsoft.com/es-es/dotnet/core/tools/?tabs=netcore2x)** _para ejecutar la aplicación en el contenedor._  _Pero podríamos compilar nuestra aplicación desde dentro del contenedor para después ejecutarla._

**WORKDIR**. Como su propio nombre indica, lo que realiza este comando es la creación del directorio de trabajo dentro del contenedor, en este caso estamos creando el directorio  _/app_. Se pueden crear en realidad todos los directorios y subdirectorios que queramos, pero para nuestro objetivo con un solo directorio de trabajo nos basta.

**EXPOSE**. Con ésta instrucción lo que estamos indicando es que la máquina expondrá en tiempo de ejecución el puerto especificado, es decir, nuestro contenedor podrá ver y comunicarse con lo que le entre por el puerto 80. Si no realizamos esta acción nuestros contenedores estarían ciegos cuando intenten comunicarse entre ellos, así como si nuestra aplicación se encuentra programada para escuchar por el puerto 5000, por ejemplo, pero no lo indicamos desde el contenedor, sería imposible acceder a él.

**COPY**. Realiza la acción de copiar desde nuestro sistema de ficheros local, partiendo desde la ruta en la que se encuentre el Dockerfile, hacia el sistema de ficheros del contenedor. En este caso estamos copiando todos los ficheros generados en la publicación ubicados en la carpeta  _**./publish**_, y transferirlos a la base del directorio de trabajo del contenedor  _**/app**_.

**ENTRYPOINT**. Con ésta instrucción lo que estamos haciendo es establecer que nuestra aplicación pueda ser ejecutada desde el cliente de Docker, y para ello le indicamos el punto de entrada a la imagen generada desde donde se puede lanzar la aplicación como un ejecutable. Por esto se le especifica en nuestra situación que el punto de entrada será  **«dotnet»**  junto con la librería base de nuestra aplicación  **«ContanerizedVisits.dll»**, para que cuando le digamos a Docker que queremos una instancia de ésta aplicación el internamente sepa como arrancar la imagen ejecutando:
```bash
    dotnet ContanerizedVisits.dll
```
Con todas estas configuraciones establecidas  **dentro del Dockerfile**  ya estaríamos listos para lanzar nuestra aplicación contenerizada.

### Creando la imagen Docker

Recapitulemos un poco antes de continuar, en este punto tendremos una estructura de carpetas como la siguiente:

![](http://mindbodyncode.com/wp-content/uploads/2019/06/folder-strucure.png)

Tenemos la base del proyecto con los clases de  **Program.cs y Startup.cs**  para arranque y configuración de la aplicación, el controlador  **GreetingsController.cs**  con la llamada GET que devuelve una cadena de texto, la carpeta  **publish**  con la compilación del proyecto lista para ser contenerizada y el fichero  **Dockerfile** con todas las instrucciones necesarias para que Docker sepa como crear la imagen y ejecutarla. Con todos estos ingredientes listos pasamos a encender los fogones y poner la olla a hervir.

Para crear la imagen Docker bastaría con ejecutar el siguiente comando:
```bash
    docker build -t contanerizedvisitsimage .
```
Con la opción  **«-t»**  le estamos indicando que el nombre de la imagen sera el de  **«contanerizedvisitsimage»**  y el siguiente parámetro será la ruta donde se encuentre el proyecto junto con el Dockerfile, que en mi caso ya estaba ubicado en la misma ruta y por eso se nombra con  **«.»**.

Si todo ha funcionado a la perfección deberíamos de obtener un resultado como el siguiente:

![](http://mindbodyncode.com/wp-content/uploads/2019/06/dockerBuildContanerizedApp.png)

Como vemos se puede ver paso por paso como va ejecutando todas las instrucciones puestas en el Dockerfile, y el resultado será una imagen Docker con nuestra aplicación a la que hemos denominado «**contanerizedvisitsimage**«. Comprobemos el resultado con el comando:
```bash
    docker image ls
```
El cual nos muestra todas las imágenes que hemos creado con Docker.

![](http://mindbodyncode.com/wp-content/uploads/2019/06/dockerImageLs.png)

Tanto las imágenes de Docker como luego los contenedores serán identificados siempre mediante un ID aleatorio por parte de Docker, por lo que si no damos nombre a nuestras imágenes siempre podremos trabajar con ellas en base al ID.

### Ejecutando el contenedor

Una vez creada nuestra imagen procedemos al último paso, ejecutar la aplicación en un contenedor, y para ello tan solo nos bastaremos de lanzar el siguiente comando:
```bash
    docker run -p 5500:80 contanerizedvisitsimage
```
Es tan simple como indicarle el puerto al que queremos publicar en el host local el que está utilizando internamente nuestra aplicación para que sea visible desde nuestra máquina, es decir, con  **«-p 5500:80»** estamos indicando que todo lo que vaya hacía el contenedor por el puerto 5500 sea dirigido dentro del contenedor al puerto 80.

Recordemos que nosotros ya habíamos expuesto el puerto 80 en el Dockerfile, lo que sucede es que exponer el puerto significa que  **tu contenedor expone ese puerto pero no que desde tu máquina local (localhost o 127.0.0.1) puedas acceder a esa dirección**. Los contenedores cuando se crean se establecen por defecto con una dirección IP aleatoria gestionada por el  **CNM de Docker**, así que lo que estamos haciendo es básicamente que desde una llamada en el navegador hacía mi máquina local _**localhost:5500**_ me redirija hacía el contenedor en el puerto 80.

El resultado por la salida de la línea de comandos que estemos usando será la siguiente:

![](http://mindbodyncode.com/wp-content/uploads/2019/06/dockerRunContanerizedApp.png)

Indicando que nuestra aplicación se está ejecutando y la salida mostrada es la del ejecutable dentro del contenedor. Si ahora hacemos una llamada desde nuestro navegador hacía la ruta  _http://localhost:5500/api/greetings_  obtendremos la respuesta desde el controlador de ejemplo dentro del contenedor:

![](http://mindbodyncode.com/wp-content/uploads/2019/06/exampleContanerizedAPI.png)

Por si todavía no te lo crees que esta aplicación esta contenerizada vamos a ejecutar el siguiente comando:
```bash
    docker ps
```
El cual te muestra todos los contenedores que existen en ejecución actualmente:

![](http://mindbodyncode.com/wp-content/uploads/2019/06/dockerPS-1024x53.png)

Como vemos muestra cierta información de configuración que hemos ido añadiendo durante todo el proceso. Si ahora queremos parar el contenedor en ejecución bastará con ejecutar el comando:
```bash
    docker stop recursing_gauss
```
Se puede apreciar como he utilizado el nombre aleatorio que te ofrece Docker para que una persona la cual está más acostumbrada a memorizar conceptos con significado en el lenguaje antes que Identificadores aleatorios pueda gestionar más fácilmente el uso de contenedores. De todas formas si eres más propenso a parecerte a una máquina también puedes utilizar el identificador para su gestión.

## Conclusión

Aquí terminamos este primer ejemplo de contenerización de aplicaciones con Docker, la verdad que ha sido mucho más teórico que práctico, pero personalmente me gusta más aprender los conceptos base de una tecnología en cuestión antes de lanzarme de cabeza a la piscina e ir probando a modo de ensayo y error hasta conseguir el objetivo.

Tan solo hemos vislumbrado un poco la punta del iceberg en éste post, la contenerización de aplicaciones abarca varios aspectos a parte de tan solo contenerizar una aplicación ya compilada. Y además existen herramientas hoy en día que están muy bien integradas con contenedores Docker, como puede ser el propio  **[Visual Studio 2019](https://visualstudio.microsoft.com/es/vs/)**, con el que es posible contenerizar aplicaciones pulsando un simple ‘_click_‘, generando automáticamente el Dockerfile, la imagen y ejecutarla en un contenedor de una forma muy sencilla. Pero por supuesto antes de montar en bici tenemos que aprender a caminar, por eso es por lo que recomiendo que una vez que estés acostumbrado a contenerizar aplicaciones con comandos nos pasemos a la «magia» de los IDE.

En otros post que seguiré publicando sobre Docker, enseñare esto último que os he comentado con Visual Studio, y el uso de orquestadores. Con esto me despido, ¡Un saludo a todos los aventureros del aprendizaje!