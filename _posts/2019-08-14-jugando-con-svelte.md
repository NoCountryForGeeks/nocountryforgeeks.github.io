---
layout: post
current: post
cover: assets/images/posts/2019-08-14-jugando-con-svelte/header.png
navigation: True
title: "Jugando con svelte"
date: 2019-08-14 12:00:00
tags: svelte javascript frontend
class: post-template
subclass: "post"
author: franmolmedo
---

Sí, otra _[librería / framework / ponga aquí el nombre que quiera]_ para ayudarnos en la tarea de realizar nuestras interfaces de usuario web utilizando javascript. ¿Por qué? ¿Para qué?. Parafraseando a San Agustín:

> Por muy lejos que hayamos llegado, el ideal siempre estará más allá.

Tenemos herramientas fiables que han demostrado su capacidad para ayudarnos a realizar esta tarea. Librerías como _React_ o frameworks como _Vue_ o _Angular_ deberían ser suficientes para satisfacer las necesidades de cualquier proyecto. Sin embargo, siempre existen otros enfoques que hay que analizar y que nos pueden aportar otra perspectiva a la hora de encarar nuestros proyectos. En este punto es donde entra **Svelte**.

## ¿Qué es Svelte?

Según su [propia página web](https://svelte.dev) _Svelte_ es un framework orientado a componentes que nos ayuda a implementar nuestros interfaces de usuario. ¿Entonces, qué nos aporta con respecto a los que hemos nombrado anteriormente? En este caso el enfoque a la hora de realizar esta tarea es lo que va a marcar la diferencia. En los casos conocidos (_React_, _Vue_ o _Angular_) se nos anima a desarrollar código declarativo que, posteriormente, exige un trabajo extra al navegador para interpretarlo. Igualmente, exigen introducir en los archivos que servimos al compilador la librería correspondiente que estemos utilizando. _Svelte_, por el contrario, introduce una etapa de compilación que va a transformar nuestro código declarativo en un código imperativo con diversas optimizaciones. Además, evita introducir mecanismos como **_Virtua DOM_** para realizar las modificaciones en el DOM de la página; sino que, utilizando este proceso de compilación con sus optimizaciones, puede determinar en tiempo de compilación qué va a cambiar en un determinado componente y generar el código más adecuado que permita realizar estos cambios. ¿Entonces es más rápido o eficiente que _React_, _Vue_ y compañía? ¿Las aplicaciones ocupan menos peso?. Trataremos de verlo en los siguientes apartados.

## Iniciando un proyecto

Iniciar un proyecto utilizando _Svelte_ no puede ser más sencillo. Descargamos la plantilla por defecto que nos proporcionan los autores y ya podemos empezar a trabajar. Para ello, ejecutamos en nuestro terminal lo siguiente:

```bash
npx degit sveltejs/template [nombre del proyecto]
```

![Clonando el repositorio](/assets/images/posts/2019-08-14-jugando-con-svelte/cloning-repository.png)

Se nos habrá clonado el repositorio _sveltejs/template_ en una carpeta con el mismo nombre que el nombre del proyecto que hayamos señalado. Con ello, sólo nos quedaría por irnos a dicha carpeta, instalar las dependencias y ejecutar el proyecto:

![Ejecutando el proyecto](/assets/images/posts/2019-08-14-jugando-con-svelte/ejecutando-proyecto.gif)

Ya tendríamos el proyecto funcionando y esperando nuestro cambios para volver a recompilar y mostrar el resultado por pantalla. Si abrimos nuestro navegador favorito en la dirección que se nos indica obtenemos lo siguiente:

![Hello world](/assets/images/posts/2019-08-14-jugando-con-svelte/hello-world.png)

### Analizamos la plantilla

El proyecto que se genera a partir de la plantilla presenta la siguiente estructura:

![Structure](/assets/images/posts/2019-08-14-jugando-con-svelte/structure.png)

Si hemos utilizado alguna de las librerías / frameworks que hemos mencionado anteriormente esta estructura de ficheros nos debe de resultar familiar. Destacamos los siguientes puntos:

- Se utiliza [rollup](https://rollupjs.org/guide/en/) en lugar de [webpack](https://webpack.js.org/) como herramienta para construir el bundle de salida. En esta entrada no profundizaremos en _rollup_, pero no estaría de más adentrarse un poco en cómo podemos ajustarlo para satisfacer nuestras necesidades más allá de la configuración básica que nos ofrece la plantilla de _svelte_.
- La aplicación tiene un punto de entrada en el fichero **_main.js_**. En él, cargaremos el componente de entrada de la aplicación y lo situaremos en el DOM:

```javascript
import App from "./App.svelte";

const app = new App({
  target: document.body,
  props: {
    name: "world"
  }
});

export default app;
```

La composición de este fichero tampoco debe de extrañar. Se importa el componente de entrada desde el archivo _App.svelte_ que veremos posteriormente y lo instanciamos. Esta instancia la creamos utilizando un objeto de configuración donde le indicamos el elemento del DOM donde queremos que se inserte nuestro componente (utilizando la propiedad _target_), y le pasamos al componente unas _props_ (igual que hacemos en _React_). Estas props consisten en un objeto con una propiedad llamada _name_.

- Por último, analizamos el fichero _App.svelte_:

```javascript
<script>
  export let name;
</script>

<style>
  h1 {
    color: purple;
  }
</style>

<h1>Hello {name}!</h1>
```

Esto ya es algo más inusual, aunque los que hayan utilizado _Vue_ sí estarán más familirizados con esta sintaxis. Los componentes _svelte_ se caracterizan por estar estructurados en tres secciones:

- Una sección donde situamos el código javascript del componente (enmarcada dentro de las etiquetas _script_).
- Una sección donde describeremos el estilo que se utilizará en el componente (dentro de las etiquetas _style_).
- Por último, tenemos el markup HTML que renderizará la estructura de nuestro componente, o sea, como se pintará en el DOM.

Pero, los más observadores, habrán notado alguna peculiaridad más:

- **¡¡Se hace un export de una variable _name_ en la parte de los scripts!!**. Sí, y no es casualidad que tenga el mismo nombre que la prop que le pasamos desde el archivo _main.js_. Esta es la notación que utilizamos en _Svelte_ para recuperar la prop que nos hace llegar el componente padre y poder utlizarla luego en nuestro markup.
- **¡¡Qué pasará cuando varios componentes usen el mismo h1!!**. Los estilos son locales a cada componente donde se declaran por lo que evitaremos colisiones. Eso sí, disponemos de un fichero de estilos globales para definir aquellos estilos que vayan a utilizarse a nivel más general dentro de la aplicación.
- **¡¡Esas llaves no son propias del lenguage HTML!!**. Correcto. Esa sintaxis es procesada por _Svelte_ en tiempo de compilación y permite introducir un valor que estemos declarando en la parte de scripts.

Hay algunas peculiaridades más en la sintaxis que iremos comentando conforme vayamos explicando las bondades de este framework. Sin embargo, en este momento, me gustaría hacer hincapié en el tamaño del bundle generado. Para ello, generamos un bundle de producción con el comando _yarn build_ y observamos que el fichero _bundle.js_ ocupa **3KB**:

![Bundle](/assets/images/posts/2019-08-14-jugando-con-svelte/bundle.png)

Si ahora creamos una aplicación _React_ utilizando _create react app_ y generamos el bunlde de salida obtemos lo siguiente:

![Bundle React](/assets/images/posts/2019-08-14-jugando-con-svelte/bundle-react.png)

**Ou mama!** La diferencia (abismal) no nos debería de preocupar en estos momentos, pero nadie puede negar que es un buen comienzo.

## Construyamos una aplicación

Entramos en faena. Con el fin de ilustrar la sintaxis a usar y ciertos detalles interesantes del framework, vamos a crear una pequeña aplicación. En esta aplicación vamos a mostrar una lista de razas de perros y al pulsar sobre uno de ellos se nos abrirá una ventana modal mostrándonos más información sobre dicha raza. Parece sencillo, pero creo que es una buena manera de mostrar algunas peculiaridades de _Svelte_.

### Creando la lista

El primer paso es almacenar en algún lugar nuestra lista de razas. Para ello, en el fichero _App.svelte_ vamos a eliminar la variable _name_ y vamos a generar una constante que contega una lista de 3 razas de perros con los siguientes datos: nombre de la raza, un thumbnail, breve descripción, una foto a mayor tamaño y un identificador. Crearemos la lista dentro de la sección _scripts_ y luego la utilizaremos en nuestro markup utilizando un bloque _each_ que nos provee _Svelte_:

```javascript
<script>
  import DogItem from "./DogItem.svelte";

  const dogs = [
    {
      id: 1,
      name: "German Shepherd",
      thumbnail:
        "https://cdn1-www.dogtime.com/assets/uploads/2011/01/file_23188_german-shepherd-dog-300x189.jpg",
      details:
        "The German Shepherd Dog is one of America’s most popular dog breeds—for good reason. They’re intelligent and capable working dogs. Their devotion and courage are unmatched. And they’re amazingly versatile, excelling at most anything they’re trained to do: guide and assistance work for the handicapped, police and military service, herding, search and rescue, drug detection, competitive obedience, and–last but not least–faithful companion",
      bigPicture:
        "https://upload.wikimedia.org/wikipedia/commons/thumb/a/a8/02.Owczarek_niemiecki_u%C5%BCytkowy_kr%C3%B3tkow%C5%82osy_suka.jpg/1920px-02.Owczarek_niemiecki_u%C5%BCytkowy_kr%C3%B3tkow%C5%82osy_suka.jpg"
    },
    {
      id: 2,
      name: "Siberian husky",
      thumbnail:
        "https://cdn3-www.dogtime.com/assets/uploads/2011/01/file_22948_siberian-husky-300x189.jpg",
      details:
        "The Siberian Husky is a beautiful dog breed with a thick coat that comes in a multitude of colors and markings. Their blue or multi-colored eyes and striking facial masks only add to the appeal of this breed, which originated in Siberia. It is easy to see why many are drawn to the Siberian’s wolf-like looks, but be aware that this athletic, intelligent dog can be independent and challenging for first-time dog owners. Huskies also put the “H” in Houdini and need a fenced yard that is sunk in the ground to prevent escapes",
      bigPicture:
        "https://upload.wikimedia.org/wikipedia/commons/a/a3/Black-Magic-Big-Boy.jpg"
    },
    {
      id: 3,
      name: "Doberman pinsher",
      thumbnail:
        "https://cdn1-www.dogtime.com/assets/uploads/2011/01/file_22920_doberman-pinscher-300x189.jpg",
      details:
        "The Doberman Pinscher was developed in Germany during the late 19th century, primarily as a guard dog. His exact ancestry is unknown, but he’s believed to be a mixture of many dog breeds, including the Rottweiler, Black and Tan Terrier, and German Pinscher. With his sleek coat, athletic build, and characteristic cropped ears and docked tail, the Doberman Pinscher looks like an aristocrat. He is a highly energetic and intelligent dog, suited for police and military work, canine sports, and as a family guardian and companion",
      bigPicture:
        "https://upload.wikimedia.org/wikipedia/commons/c/c0/0Doberman-40172501920.jpg"
    }
  ];
</script>

<section id="dogs">
  {#each dogs as dog}
    <DogItem name={dog.name} thumbnail={dog.thumbnail} id={dog.id} />
  {/each}
</section>
```

- El primer aspecto a tener en cuenta es que estamos importando un componente llamado _DogItem_ que utilizaremos para mostrar cada una de las razas que queramos pintar en nuestra lista. El código asociado a este componente, que crearemos en un archivo llamado _DogItem.svelte_, es el siguiente:

```javascript
// DogItem.svelte
<script>
  export let id;
  export let name;
  export let thumbnail;
</script>

<article>
  <h1>{name}</h1>
  <img alt={name} src={thumbnail} />
</article>
```

Básicamente recogemos las props que nos llegan desde el componente padre (_App.svelte_) y utilizamos esos valores para mostrarlos en nuestro HTML.

- Un segundo aspecto importante a comentar es la forma en la que hemos recorrido los elementos de la lista. _Svelte_ nos proporciona bloques que nos permiten implementar funcionalidad diferente: en este caso recorrer la lista con el bloque _each_, también podemos tener condiciones utilizando el bloque _if_ en conjunción con el bloque _else_ o con los bloques _else if_ o incluso tener la capacidad de trabajar con promesas con el bloque _await_. Sinceramente, este tipo de sintaxis es uno de los puntos que menos me gustan de _Svelte_ ya que preferiría un enfoque más directo utilizando solamente javascript.

  En este punto me gustaría trabajar un poco más en nuestro bloque _each_ para ver las distintas variaciones de la sintaxis que podemos usar:

  - Podemos aplicar destructuring en los valores de los elementos de la lista:

    ```javascript
    {#each dogs as {name, thumbnail, id}}
        <DogItem name={name} thumbnail={thumbnail} id={id} />
    {/each}
    ```

    - Igualmente, al coincidir el nombre de la propiedad con la variable, podemos utilizar una sintaxis más reducida:

    ```javascript
    {#each dogs as { name, thumbnail, id }}
        <DogItem {name} {thumbnail} {id} />
    {/each}
    ```

    - También podemos obtener el índice de cada elemento de la iteración:

    ```javascript
    {#each dogs as { name, thumbnail, id }, index}
        <DogItem {name} {thumbnail} {id} />
    {/each}
    ```

El resultado de este código es el siguiente:

![Dog list](/assets/images/posts/2019-08-14-jugando-con-svelte/dog-list.png)

### Añadiendo algo de funcionalidad

Con el fin de ilustrar algunas características más que nos ofrece _Svelte_ vamos a implementar el detalle de los elmentos de nuestra lista. La idea es que vamos a generar un componente modal que, al pulsar sobre uno de los elementos de nuestra lista, se va a pintar dentro de él el detalle de la raza que hayamos escogido. Implemnetando el modal vamos a poder introducir conceptos como los _eventos definidos por el usuario_ y los _slots_.

El primer paso de todos es crear un nuevo fichero (llamado _Modal.svelte_) donde vamos a implementar nuestro componente modal genérico (surpimimos la parte de estilos para evitar que el código se alargue demasiado):

```javascript
<script>
  import { createEventDispatcher } from "svelte";

  const dispatch = createEventDispatcher();

  const closeModal = () => dispatch("closemodal");
</script>

<div class="back" on:click={closeModal} />
<div class="modal">
  <header>
    <slot name="header">TEST</slot>
  </header>
  <div class="content">
    <slot />
  </div>
  <footer>
    <slot name="footer">
      <button on:click={closeModal}>Close</button>
    </slot>
  </footer>
</div>
```

En primer lugar vamos a centrarnos en los **slots**. Estos slots forman parte del standard _HTML_ (dentro de la parte destinada a la creación de componentes web) y cuya función es servir de placeholder para poder insertar contenido desde otro lugar, en nuestro caso desde otro componente. Estos _slots_ pueden ser nombrados (como el caso de _header_ o _footer_) utilizando el atributo _name_ o por defecto, como es el que se utiliza dentro del div con la clase _content_. Un compomente sólo puede contener un slot por defecto, pero sí puede tener un número indeterminado de slots nombrados. Otro aspecto a destacar es que podemos proporcionar una implementación por defecto del slot. Por ejmplo, en el caso del slot _footer_ estamos diciendo que si no se provee contenido para el slot actual, queremos que se muestre un botón.

**¿Cómo pasamos el contenido a los slots?** Para ello, lo mejor es mostrar el código final de nuestro archivo _App.svelte_ (igualmente omitimos los estilos y la implementación del array _dogs_):

```javascript
<script>
  import DogItem from "./DogItem.svelte";
  import Modal from "./Modal.svelte";

  const dogs = [
...
  ];

  let selectedDog = null;

  const selectItemHandler = event => {
    const { id } = event.detail;

    selectedDog = dogs.find(dog => dog.id === id);
  };

  const closeModalHandler = () => (selectedDog = null);
</script>

<section id="dogs">
  <header>Our dogs</header>
  {#each dogs as { name, thumbnail, id }, index}
    <DogItem {name} {thumbnail} {id} on:selectitem={selectItemHandler} />
  {/each}
  {#if selectedDog !== null}
    <Modal on:closemodal={closeModalHandler}>
      <span slot="header">{selectedDog.name}</span>
      <div class="detail-content">
        <div>{selectedDog.details}</div>
        <img alt={selectedDog.name} src={selectedDog.bigPicture} />
      </div>
    </Modal>
  {/if}
</section>
```

Como vemos, para pasar el contenido a un slot, podemos utilizar otro elemento _HTML_ (en nuestro caso _span_) indicándole que ese _span_ se ha de colocar en el _slot_ cuyo nombre le estamos pasando dentro del atributo _slot_. De esta forma, pasamos los slots nombrados. En el caso del resto del contenido situado entre las etiquetas _<Modal></Modal>_ se pasará al slot por defecto.

Por último, nos quedaría hablar de la creación de eventos. Como vemos en el código del _modal_ usamos la utilidad _createEventDispatcher_ que importamos directamente desde _svelte_. Esta utilidad nos permite construir una función _dispatch_, a partir de la cual podemos generar este tipo de eventos personalizados. La función _dispatch_ puede recibir un par de parámetros:

- El primero sería el nombre del evento. Este nombre es muy importante porque en el componente padre podremos asociar manejadores del evento utilizando dicho nombre. Por ejemplo, si creamos un evento personalizado llamado _closemodal_, posteriormente podemos definirle un manejador de la siguiente forma: _on:closemodal={manejador}_, tal y como hemos hecho en el ejemplo.

Además, estos eventos pueden contener información. Esta información se le indica en el segundo parámetro de la función _dispatch_. Lo vemos en el código definitivo del componente _DotItem.svelte_ (también suprimimos los estilos por concretitud):

```javascript
<script>
  import { createEventDispatcher } from "svelte";

  const dispatch = createEventDispatcher();

  export let id;
  export let name;
  export let thumbnail;
</script>

<article on:click={() => dispatch('selectitem', { id })}>
  <h1>{name}</h1>
  <img alt={name} src={thumbnail} />
</article>
```

En la función _dispatch_ del evento _selectitem_ hemos añadido como segundo parámetro un objeto que contiene la propiedad _id_ refiriéndose al identficador del elemento seleccionado. Posteriormente, este valor podemos recuperarlo desde el propio evento, en la propiedad _details_, como vemos en el código del manejador siguiente:

```javascript
const selectItemHandler = event => {
  const { id } = event.detail;

  selectedDog = dogs.find(dog => dog.id === id);
};
```

De esta forma hemos implementado la apertura o no del modal para mostrar los detalles del perro seleccionado. Cuando se selecciona un elmeento de la lista, se manda el evento _selectitem_ que tiene asociado el manejador anterior. Así actualizamos la variable _selectedDog_ con los detalles del perro elegido. Posteriormente en el markup tenemos un senticia _if_ que nos permite pintar el modal si tenemos un perro seleccionado o no pintarlo si _selectedDog_ es _null_. Para volver a establecer _selectedDog_ a _null_, tenemos un evento llamado _closemodal_ cuyo manejador realiza esa acción:

```javascript
const closeModalHandler = () => (selectedDog = null);
```

Nuestra aplicación terminada luce así:

![Aplicación finalizada](/assets/images/posts/2019-08-14-jugando-con-svelte/app-finished.gif)

## Routing y Server Side Rendering con Sapper

Los autores de _Svelte_ están trabajando igualmente en un framework que nos proporciona routing y renderizado de servidor utilizando _Svelte_. Constituye lo que representa [Next](nextjs.org) a _React_. Por lo tanto no es una alternativa a todo lo que hemos contando, sino que nos va ayudar a llevar a cabo estas dos tareas concretas.

- Sapper implementa el enrutado asignando componentes de _Svelte_ a direcciones URL.
- con el renderizado de servidor, el primer renderizado de la página se realiza en servidor (obviedad!) con lo que ganaremos velocidad en este primer render y también facilitaremos los procesos de SEO. Con ello, al navegador le enviaremos una página HTML con contenido y algo de javascript, en lugar de un HTML prácticamente vacío y gran carga de javascript (modelo de SPA).
  **¿Cómo se configura todo esto?** Al igual que ocurre en _Next.js_ es necesario comprender el sistema de carpeta ya que mucha de la configuración se lleva a cabo dependiendo de la localización de los ficheros.

### Empezando nuestro proyecto con Sapper

Al igual que con _Svelte_ tenemos varias plantillas con la que podemos empezar. Utilizaremos, igual que antes, la plantilla basada en _Rollup_, pero podemos elegir la basada en _Webpack_ si así lo preferimos.

```bash
npx degit "sveltejs/sapper-template#rollup" [nombre del proyecto]
```

En el siguiente gif vemos cómo llevamos la inicialización del proyecto:

![Init con sapper](/assets/images/posts/2019-08-14-jugando-con-svelte/sapper-init.gif)

Y listo, no se tarda prácticamente nada en tener montado el scaffolding básico y listo para trabajar. Lo que obtenemos por pantalla es lo siguiente:

![Sapper pantalla inicial](/assets/images/posts/2019-08-14-jugando-con-svelte/sapper-initial-screen.png)

### Hablemos de la estructura del proyecto

La estructura del proyecto que se nos ha generado es la siguiente (hemos eliminado el fichero _.gitignore_ y el _README.md_ ya que en nuestro repositorio hemos recogido tanto el proyecto básico de svelte como éste y esos dos ficheros los tendremos en la raíz):

![Sapper estructura](/assets/images/posts/2019-08-14-jugando-con-svelte/sapper-structure.png)

Destacamos los aspectos más importantes:

- En la carpeta _src_ tenemos, a su vez, dos carpetas: una carpeta _components_ donde, en principio, se colocarán los componentes que utilizaremos en nuestras páginas y una carpeta _routes_ que contendrá las rutas o páginas de la aplicación. Por lo tanto, cada componente que situemos en esta carpeta se corresponderá, automáticamente, con una ruta. En nuestro ejemplo actual tenemos en esta carpeta cuatro ficheros: _index.svelte_ que se corresponderá a la dirección raíz de nuestra página y _about.svelte_ que se corresponderá a la página about. Tal es el caso, que si navegamos a la página _localhost:3000/about_ se renderizará el componente del fichero _about.svelte_ situado en la carpeta _routes_. El nombre del directorio _components_ podemos modifcarlo, el de _routes_ no.

![Sapper about](/assets/images/posts/2019-08-14-jugando-con-svelte/sapper-about.png)

- Dentro de la carpeta _routes_ encontramos también una carpeta _blog_ que contiene, a su vez, una serie de ficheros que pasamos a comentar:

  - _index.svelte_: Componente que se renderizará cuando se navegue a la página _host/blog_. En él se renderizarán los posts que se encuentren disponibles en el blog.
  - _index.json.js_: Este es un fichero de servidor. En él establecemos la respuesta a una llamada http preguntando por los posts disponibles. De ellos devolveremos el título y el _slug_
  - _[slug].svelte_: Componente que se renderizará cuando se navegue a la dirección _host/blog/[slug]_ Es importante hacer notar que _slug_ es un placeholder, que puede ser sustituído por cualquier cadena de caracteres (que en este caso serían los slugs de cada post). Así podemos navegar a url personalizadas, como, por ejemplo:

![Sapper slugs](/assets/images/posts/2019-08-14-jugando-con-svelte/sapper-slugs.png)

    - _[slug].json.js_: Es el equivalente al _index.json.js_ pero para la página de slugs.
    - *_posts.js*: Fichero que contiene la información de los posts. Actúa de respositorio y será utilizado por los ficheros de servidor para recuperar la información de los posts.
    - Quedarían dos archivos más por comenentar y ambos comienzan por guión bajo: *_error.svelte* es el componente que se mostará cuando haya algún error, sería algo así como la página de error general por defecto y *_layout.svelte* que constituye un esquema general que van a seguir todas las páginas. Es decir, el resto de páginas tomarán lo descrito en este fichero como plantilla para rederizarse de la manera descrita en él.

- Los ficheros _client.js_, _server.js_ y _service-worker.js_ son ficheros de utilidad que se encargan de preparar al cliente que se utilizará en el navegador, el servidor (que por defecto utiliza _polka_, pero que podemos cambiar a _express_ si se desea) y el último nos proporciona una configuración básica de service worker que podemos modificar para construir una aplicación _PWA_.
- Por último, tenemos el fichero _template.html_ que constituye la estructura básica del _HTML_ que se generará.

Es importante hacer notar que en la configuración básica tenemos preparado [cypress](www.cypress.io) y que el _live reloading_ está implementado por defecto. Por otro lado, y aunque en esta introducción no ahondaremos en ello, tenemos funcionalidad como el _prefetching_, uso de _stores_ generales y demás que se encuentran igualmente disponibles.

### Implementando nuestro ejemplo en Sapper

Una vez conocida la estructura y la disposición de los ficheros que tenemos que usar, para que nuestro pequeño ejemplo funcione con _sapper_ tenemos que adaptarlo a dicha estructura. Así, eliminamos todos los componentes y las rutas que nos interesan y copiamos nuestros componentes _DogItem_ y _Modal_ en la carpeta _components_. Mientras, copiamos el contenido de _App.svelte_ en el fichero _index.svelte_ de la carpeta routes. Con ello, y cambiando las direcciones de los _imports_ debería de bastar para que funcione este ejemplo. Si lanzamos ambas aplicaciones podemos observar la diferencia entre una y otra:

- Aplicación con _Svelte_:

![Svelte network](/assets/images/posts/2019-08-14-jugando-con-svelte/svelte-network.png)

- Aplicación con _Sapper_:

![Sapper network](/assets/images/posts/2019-08-14-jugando-con-svelte/sapper-network.png)

Como vemos, mientras en el primer caso se descarga la plantilla del _HTML_ básica para después crear la página a partir del _bundle.js_; en el segundo, la página _HTML_ ya se ha renderizado en servidor y la bajamos ya construida. Eso no quita que se descargue también contenido javascript para poder hacer las interacciones en el cliente.

## Conclusiones

Sólo hemos mostrado una pequeña parte de lo que nos ofrece _Svelte_. Si os resulta interesante en siguientes entradas podemos hablar de cómo implementar _stores_ o _contextos_ para almacenar los datos de nuestra aplicación, profundizar en el uso de _sapper_ y comentar funcionalidad como el _prefetch_ de datos o adentrarnos en interacciones más complejas como los distintos tipos de bindings exitentes, transiciones, formas de organizar el código o cómo fomentar el reuso de componentes.

En definitiva, me _Svelte_ me parece una gran alternativa a las opciones más populares existentes hoy en día, tanto por su versatilidad, por la generación de bundles de tamaño reducido y por la concepción y la idea en general. Además, tenemos ya componentes interesantes elaborados por la comunidad como:

- Listas virtualizadas utilizando [Svelte virtual list](https://github.com/sveltejs/svelte-virtual-list)
- Podemos utilizar librerías de CSS-in-JS como [emotion](https://svelte.dev/blog/svelte-css-in-js)
- Librerías de testing como [Svelte testing library](https://github.com/testing-library/svelte-testing-library)
- Componentes de material design: [Smelte](https://github.com/matyunya/smelte)

Así mismo, os dejo el repositorio con el código de los dos ejemplos en: [svelte101](https://github.com/franmolmedo/svelte101)

¡Mantente curioso!
