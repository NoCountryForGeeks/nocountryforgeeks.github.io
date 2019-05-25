---
layout: post
current: post
cover: assets/images/posts/2019-05-27-no-renderices-mas-de-la-cuenta/cover.jpg
navigation: True
title: "No renderices más de la cuenta"
date: 2019-05-27 12:00:00
tags: react javascript react-window virtualization
class: post-template
subclass: "post"
author: ivanrodri
---

Hoy os vengo a hablar un poco de la virtualización y como nos puede ayudar a salvar nuestra aplicación en cuanto a rendimiento cuando tenemos **listados** o **grids** con mucha información. Las aplicaciones cada vez manejan cantidades más grandes de datos, si a esto le sumamos un diseño de nuestro UI
donde tenemos que mostrar una gran cantidad de elementos páginados (100 items por página) o incluso un scroll infinito, tenemos una combinación perfecta para poder cargarnos el rendimiento y la experiencia de usuario de nuestra aplicación.
No vale echar la culpa a los diseñadores por su diseño o luchar para esconder la 💩 debajo de la alfombra, tenemos que conseguir una aplicación que no se degrade en el tiempo a mayor cantidad de información.

En este caso voy a tratar el tema con **React** ya que no existe un componente nativo del navegador que soporte virtualización. El objetivo es renderizar un listado de 20.000 usuarios de la forma más rápida y eficiente posible. Para ello también he utilizado **React.unstable_Profiler** para saber el tiempo que tarda **React** en renderizar los listados.

## Renderizando mi lista de usuarios

```javascript
import React from "react";

import { UserItem } from "../shared/UserItem";
import { Profiler } from "../shared/Profiler";

import { getUsers } from "../../services";

import { userList } from "./userList.module.css";

export const UserList = () => {
  const users = getUsers({ startIndex: 0, stopIndex: 20000 });

  return (
    <Profiler id="noVirtualized">
      <ul className={userList}>
        {users.map((user, index) => (
          <li key={index}>
            <UserItem user={user} />
          </li>
        ))}
      </ul>
    </Profiler>
  );
};
```

De esta forma tan sencilla podemos renderizar los usuarios en pantalla pero... ¿Que va a suceder exactamente?

![No Virtualized List Render Time](/assets/images/posts/2019-05-27-no-renderices-mas-de-la-cuenta/notVirtualized.gif)

[Ejemplo lista no virtualizada](https://vvgpb.codesandbox.io/no-virtualized)

Pues como se puede comprobar, el listado funciona pero tiene unos cuantos problemas:

- El renderizado inicial es muy lento debido a la gran cantidad de trabajo que tiene que hacer **React** **(7.5427s)**.
- La interfaz se queda completamente bloqueada hasta que los elementos son renderizados.
- El hover sobre los items va con mucho retraso.
- Cuando se hace scroll este va muy lageado.
- La prueba se ha hecho en **Chrome** si vamos a navegadores con menos capacidad de computación, estos problemas se agravarán.

Ahora podemos pensar que si esto lo hacemos con **lazy-load** de datos, no tendríamos este comportamiento **¿es eso cierto?**. Para explicarlo, primero tenemos que entender como funciona **React** y como sería la combinación con **lazy-load**. Si vamos cargando los elementos de 100 en 100, nuestra carga inicial será bastante más rápida pero a medida que vamos cargando más elementos, estamos modificando el estado, lo que significa que el **render** se va a volver a ejecutar. En el caso que usemos **memo** o **PureComponent** en los elementos de la lista, los elementos existentes no lanzaran un re-render **(será necesario hacer la comparación de props lo que puede hacer que el tiempo incremente)**, los nuevos elementos serán creados en el DOM, todo esto implica que nuestro performance inicial va a ser bueno pero a medida que se cargan más datos, nuestro performance va a ir siendo peor, por lo tanto tenemos un listado que se degrada en el tiempo.

## Virtualización al rescate

Cuando hablamos de virtualizar una lista o grid nos referimos a renderizar en pantalla solo los elementos que el usuario está viendo en ese momento. La virtualización en listas puede ser tanto horizontal como vertical (dependiendo de la dirección que le indiquemos a la lista) y para los grids la virtualización es tanto horizontal como vertical al mismo tiempo. Para conseguir la virtualización se usan técnicas de **windowing** para calcular los elementos que deben mostrarse y cuales no.

### ¿Cómo funciona?

#### Lista

![Lista virtualizada](/assets/images/posts/2019-05-27-no-renderices-mas-de-la-cuenta/listVirtualized.png)

![Lista virtualizada](/assets/images/posts/2019-05-27-no-renderices-mas-de-la-cuenta/listVirtualized.gif)

#### Grid

![Lista virtualizada](/assets/images/posts/2019-05-27-no-renderices-mas-de-la-cuenta/gridVirtualized.png)

Para **React** existen varias librerías que nos permiten crear listados virtualizados pero hay 2 que destacan del resto [react-virtualized](https://github.com/bvaughn/react-virtualized) y [react-window](https://github.com/bvaughn/react-virtualized). Ambas librerías son de [Brian Vaughn](https://twitter.com/brian_d_vaughn?lang=es) que es uno de los desarrolladores del equipo de **React**.

En este caso voy a utilizar **react-window** ya que es una librería muy sencilla de usar, tiene un rendimiento mayor al de **react-virtualized** y el peso de la librería es menor.

```javascript
import React from "react";
import { FixedSizeList } from "react-window";

import { UserItem } from "../shared/UserItem";

import { getUsers } from "../../services";
import { Profiler } from "../shared/Profiler";

export const UserList = () => {
  const users = getUsers({ startIndex: 0, stopIndex: 20000 });

  return (
    <Profiler id="virtualized">
      <FixedSizeList
        height={400}
        width={300}
        itemSize={50}
        itemData={users}
        itemCount={users.length}
        overscanCount={5}
      >
        {UserListItem}
      </FixedSizeList>
    </Profiler>
  );
};

const UserListItem = ({ style, index, data }) => (
  <div style={style}>
    <UserItem user={data[index]} />
  </div>
);
```

De esta manera tan sencilla podemos crear una lista vitualizada con **react-window**. En este caso he usado la lista con altura de items fijos (existe una que nos permite items con alturas variables). Las **props** que tenemos que pasarle al listado son bastante simples, podeis verlo en [la documentación](https://react-window.now.sh/#/api/FixedSizeList).

![No Virtualized List Render Time](/assets/images/posts/2019-05-27-no-renderices-mas-de-la-cuenta/virtualized.gif)

[Ejemplo lista virtualizada](https://vvgpb.codesandbox.io/virtualized)

En este caso el rendimiento de nuestra aplicación ha cambiado por completo, si comparamos con el caso anterior:

- **Tiempo de renderizado:** Ha pasado de **7.5427s** a **0.0093s**.
- **Bloqueo de interfaz:** Inexistente ya que la carga ha sido directa.
- **Hover:** El hover reacciona instantaneamente.
- **Scroll:** El scroll funciona sin ningún tipo de lag.
- **Browser:** El rendimiento es bueno en todos los navegadores (incluido IE 11).

### Peticiones de imágenes

En el ejemplo estamos cargando las imágenes del avatar de los usuarios, esto requiere de hacer peticiones al backend para poder cargarlas. **¿Como está funcionando eso en cada uno de los ejemplos?**

#### Carga de imágenes en listado sin virtualización

![No Virtualized List Render Time](/assets/images/posts/2019-05-27-no-renderices-mas-de-la-cuenta/imageRequests.gif)

Lanza todas las peticiones de las imágenes del avatar en este caso son **1257** peticiones (**Faker.js** repite imágenes sino, serían unas 20.000) esto puede generar sobrecargas en nuestro servidor y ralentizar otras llamadas.

#### Carga de imágenes en listado con virtualización

![No Virtualized List Render Time](/assets/images/posts/2019-05-27-no-renderices-mas-de-la-cuenta/imageVirtualizeRequests.gif)

Como vemos, de inicio carga 13 imágenes, 8 de ellas visibles por el usuario más las 5 de **overscan** que le hemos dicho a **react-window** que precargue para dar una mejor experiencia de usuario cuando hagamos scroll. A medida que se hace scroll se van pidiendo las demás.

En este caso hemos reducido la sobrecarga inicial de imágenes y solo vamos a pedir las que el usuario necesite. Pero esto incluso podemos mejorarlo un poquito más. **react-window** da la oportunidad de saber si se esta haciendo **scroll** sobre la lista. Si hacemos scroll rápidamente, vamos a cargar imágenes que el usuario no ha puesto atención en ellas. Con la **prop** que **react-window** nos ofrece, podemos mostrar un **placeholder** como imágen y cuando se pare de hacer scroll, poner las imágenes reales del avatar.

```javascript
import React from "react";
import { FixedSizeList } from "react-window";

import { UserItem } from "../shared/UserItem";

import { getUsers } from "../../services";

export const UserList = () => {
  const users = getUsers({ startIndex: 0, stopIndex: 20000 });

  return (
    <FixedSizeList
      height={400}
      width={300}
      itemSize={50}
      itemData={users}
      itemCount={users.length}
      overscanCount={5}
      useIsScrolling={true}
    >
      {UserListItem}
    </FixedSizeList>
  );
};

const UserListItem = ({ style, index, data, isScrolling }) => (
  <div style={style}>
    <UserItem user={data[index]} showImageAvatar={!isScrolling} />
  </div>
);
```

Indicandole a **react-window** la **prop** ```useIsScrolling={true}``` conseguimos que en el render de los items obtengamos la propr **isScrolling** con la que vamos a poder decidir que mostrar.

![No Virtualized List Render Time](/assets/images/posts/2019-05-27-no-renderices-mas-de-la-cuenta/isScrolling.gif)

[Ejemplo lista isScrolling](https://vvgpb.codesandbox.io/virtualized-scrolling)

De esta manera podemos concretar que las imágenes que pidamos van a ser las que el usuario necesita.

### virtualización + lazy-load

Antes hemos comentado que en una lista **no virtualizada** el **lazy-loading** iba a ayudarnos de inicio, pero a medida que tuviésemos más datos, ibamos a ir perdiendo rendimiento. Esto no significa que no haya que usar **lazy-loading**, debería de ser algo que hiciésemos por defecto con los listados de nuestras aplicaciones.

Implementar un **lazy-load** en **react-*window** es bastante sencillo. Existe una librería a parte llamada [**react-window-infinite-loader**](https://github.com/bvaughn/react-window-infinite-loader) que expone un componente para hacer de forma fácil y sencilla **lazy-load** con **react-window**

```javascript
import React, { useCallback, useState } from "react";
import { FixedSizeList } from "react-window";
import InfiniteLoader from "react-window-infinite-loader";

import { UserItem } from "../shared/UserItem";
import { UserItemSkeleton } from "./userList/UserItemSkeleton";

import { getUsers } from "../../services";

const TOTAL_USERS = 20000;

export const UserList = () => {
  const [users, setUsers] = useState([]);
  const loadMoreItems = useLoadMoreItems({ users, setUsers });
  const isItemLoaded = useCallback(index => !!users[index], [users]);
  return (
    <InfiniteLoader
      isItemLoaded={isItemLoaded}
      itemCount={20000}
      loadMoreItems={loadMoreItems}
      minimumBatchSize={100}
    >
      {({ onItemsRendered, ref }) => (
        <FixedSizeList
          ref={ref}
          onItemsRendered={onItemsRendered}
          height={400}
          width={300}
          itemSize={50}
          itemData={users}
          itemCount={TOTAL_USERS}
          overscanCount={5}
          useIsScrolling={true}
        >
          {UserListItem}
        </FixedSizeList>
      )}
    </InfiniteLoader>
  );
};

const useLoadMoreItems = ({ users, setUsers }) =>
  useCallback(
    (startIndex, stopIndex) =>
      new Promise(resolve => {
        setTimeout(() => {
          const loadedUsers = getUsers({ startIndex, stopIndex });
          setUsers([...users, ...loadedUsers]);
          resolve();
        }, 1000);
      }),
    [setUsers, users]
  );

const UserListItem = ({ style, index, data, isScrolling }) => (
  <div style={style}>
    {data[index] ? (
      <UserItem user={data[index]} showImageAvatar={!isScrolling} />
    ) : (
      <UserItemSkeleton />
    )}
  </div>
);
```

Con simplemente 63 lineas de código, tenemos una lista virtualizada con **lazy-load** de datos y en este caso para los items que aún no se han cargado, se muestra un skeleton. Para simular una request, he añadido 1 segundo de retardo a los nuevos elementos cargados.

![No Virtualized List Render Time](/assets/images/posts/2019-05-27-no-renderices-mas-de-la-cuenta/lazyLoad.gif)

[Ejemplo lista lazy-load](https://vvgpb.codesandbox.io/virtualized-lazy)

## react-window

Esta librería nos permite virtualizar listas y grids de una forma muy sencilla. Nos ofrece funcionalidades que si las tuviésemos que hacer nosotros nos llevarían un buen rato **(navegar a la posición de un item, la posición de scroll inicial en la que comenzar, la dirección de nuestras listas y grids para los idiomas que funcionan con rtl (right to left))**.

Una de las ventajas que tiene **react-window** sobre otras librerías es que simplemente se encarga de dar un esqueleto y unos cálculos, de este modo es muy sencillo estilarlo como necesitemos, añadir funcionalidades que no soporta por defecto **(stickyheader, drag & drop)**. Y para mí lo más importante es la comunidad que tiene al   rededor, donde podemos encontrar muchas librerías que ofrecen funcionalidades como las mencionadas funcionando directamente con **react-window**.

## Conclusión

Con la virtualización de listas y grids conseguimos tener vistas más eficientes ya que podemos poner el foco en **solo renderizar en pantalla lo que el usuario está viendo en ese momento.** De esta manera podemos asegurarnos que los listados van a funcionar de una manera muy similar en todos los navegadores y que su rendimiento va a ser el mismo independientemente de la cantidad de datos.

Cuando trabajamos con virtualización, tenemos que cambiar nuestra manera de pensar, ya que hay ciertos casos en que la manera de hacer algunas cosas no son igual a cuando todos los elementos existen, también hay que tenerlo en cuenta a la hora de dar estilos, ya que todos nuestros tamaños son fijos y las cajas no crecen según su contenido.

## Ejemplo codesandbox

<iframe src="https://codesandbox.io/embed/magical-bartik-vvgpb?fontsize=14" title="magical-bartik-vvgpb" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>