---
layout: post
current: post
cover: assets/images/posts/2020-03-18-react-data-fetching-con-suspense/cover.png
navigation: True
title: "React data-fetching con Suspense"
date: 2020-03-18 12:00:00
tags: react suspense react-cache
class: post-template
subclass: "post"
author: ivanrodri
---

Hoy quiero hablaros sobre como **React** nos permite hacer una carga de datos asíncrona de una forma totalmente declarativa y casi convirtiéndolo en una operación síncrona.

En la actualidad si usais alguna librería de renderizado de **UI** (**React**, **Vue**, **Angular**, etc) casi seguro que habéis tenido que lidiar con carga de datos mediante llamadas asíncronas al servidor. Cuando realizamos una llamada asíncrona al servidor desde el **UI** tenemos que controlar al menos 3 estados posibles de nuestra **UI** estos son:

**Loading**: Tenemos que controlar un estado de espera de nuestra petición ya que es una llamada asíncrona y dependemos de servidor, network, latencia, etc. En este punto podemos mostrar una pantalla en blanco hasta que nuestros datos carguen, un loading para dar feedback al usuario o un skeleton dando una sensación mas real de los datos al usuario.

**Error**: En este punto tenemos que controlar como debe reaccionar nuestra app cuando algo falla en el proceso de petición de datos. Puede ser que nos encontremos un error devuelto por el servidor o un problema de conexión, en este caso debemos dar feedback al usuario de que algo ha pasado y mostrar una **UI** consistente a lo sucedido.

**Datos**: Esto es lo que nosotros veníamos buscando datos para mostrar al usuario una vez obtenidos tenemos que renderizar nuestra **UI** con los datos obtenidos del servidor.

## Patrón isLoading, error, data

Muchos de vosotros por no decir todos habréis usado esta manera de controlar una petición al servidor ya sea implementando vuestra propia lógica para controlar los 3 estados o ya se a porque habéis usado una librería de terceros que lo implementa de esta manera ([react-query](https://github.com/tannerlinsley/react-query), [useSWR](https://swr.now.sh/)).

```javascript
const FetchData = () => {
  const [{ data, error, isLoading }, dispatch] = useState({
    data: null,
    error: null,
    isLoading: false
  });

  useEffect(() => {
    dispatch(state => ({ ...state, isLoading: true }));
    fetchData()
      .then(data => dispatch({ data, error: null, isLoading: false }))
      .catch(error => dispatch({ data: null, error, isLoading: false }));
  }, []);

  if (isLoading) return <Loading />;
  if (error) return <Error error={error} />;
  return <Data data={data} />;
};
```

Este sería un ejemplo de lo que solemos hacer normalmente, tener un estado en el componente con los 3 posibles estados de la petición, lanzar la petición en un **useEffect** y dependiendo del resultado de la promesa, actualizar de una u otra manera nuestro estado para renderizar la **UI** que dependiendo de los valores va a renderizar uno u otro estado.

## Data fetching con Suspense

Desde el equipo de **React** han estado introduciendo nuevos cambios para poder hacer **data-fetching** de una manera mas declarativa y potente que en el caso anterior para ello vamos a manejar los distintos estados de una petición con **componentes** en vez de variables con estado.

```javascript
export const FetchData = () => {
  return (
    <ErrorBoundary>
      <Suspense fallback={<Loading />}>
        <Data />
      </Suspense>
    </ErrorBoundary>
  );
};

const dataResource = createResource(fetchData);

const Data = () => {
  const data = dataResource.read();
  return <RenderData data={data} />;
};
```

En este ejemplo podemos ver que no tenemos ningún estado pero somos capaces de controlar los mismos casos anteriores, esto es debido a 2 componentes que ofrece **React** que nos permite controlar ese estado.

- **isLoading** -> **Suspense**: Si comparamos los 2 ejemplos anteriores la equivalencia de **isLoading** en este caso sería el componente **Suspense** (un componente importado de **React**). Este componente nos mostrará el **Loading** mientras nuestra petición espera una respuesta.

- **error** -> **ErrorBoundary**: En caso de error en la petición nuestro componente **ErrorBoundary** mostrará una **UI** acorde al error capturado.

- **data** -> **Data**: El componente **Data** es el encargado de renderizar los datos en el **UI** cuando nuestra petición nos ha retornado datos, **los obtenemos de una manera sincrona** aun que realmente todo el proceso haya sido **asíncrono**.

## ¿Que es createResource?

Este es un código de ejemplo pero lo que realmente estamos haciendo en este caso es crear un recurso, ese recurso recibe la función que va a hacer el fetch a nuestro servidor.

Si nos fijamos hacemos **dataResource.read()** dentro del render del componente, este **read** va a buscar el recurso en la caché, si existe nos lo retorna (**sin promesa**) y sino, lanza el fetch (que es una promesa) para que **Suspense lo capture**

## ¿Cómo funciona **Suspense**?

**Suspense** es un componente que importamos de **React**, este componente captura **cualquier promesa que sea lanzada en tiempo de render**. Es muy importante remarcar el **tiempo de render**, esto no funcionará si nuestra promesa es lanzada en un **callback** o un **useEffect**. Cuando me refiero a lanzar una promesa en tiempo de render me refiero a algo como esto:

```javascript
export const FetchData = () => {
  return (
    <Suspense fallback={<Loading />}>
      <Data />
    </Suspense>
  );
};

const Data = () => {
  const data = throw fetch("/data");
  return <RenderData data={data} />;
};
```

La función **fetch** es la propia del navegador que nos permite hacer peticiones al servidor, esta función retorna una promesa (a la cual podemos atachar la función then y catch). En javascript podemos hacer un **throw** de una promesa, no tiene porque ser un error. De esta manera, cuando lanzamos una promesa, **Suspense** captura ese **throw** y empieza a mostrar nuestro **Loading** hasta que nuestra promesa se resuelve y se lanza el render de nuevo, en ese caso ya tendríamos que mostrar nuestros datos.

Es muy importante que tengamos en cuenta que **Suspense** tiene que estar por encima de aquellos componentes que lanzan la promesa, por eso en el ejemplo está separado en 2 componentes distintos, si estuviese todo al mismo nivel, no sería capaz de capturarlo.

## ¿Cómo funciona **ErrorBoundary**?

**ErrorBoundary** es un componente custom que nos podemos crear usando el método de ciclo de vida que nos ofrece **React** **componentDidCatch**:

```javascript
class ErrorBoundary extends Component {
  state = {
    error: null
  };

  componentDidCatch = (error, errorInfo) => {
    this.setState({ error });
  };

  render = () => {
    if (this.state.error) return <CatsError error={this.state.error} />;

    return this.props.children;
  };
}
```

**componentDidCatch** solo captura aquellos errores lanzados en tiempo de render. Funciona completamente igual que **Suspense** pero en vez de capturar promesas, captura errores.

**NOTA: Para los ejemplos que he creado yo he usado react-cache que es un paquete experimental (que he tenido que empaquetar yo mismo) en el que está trabajando el equipo de react. Pero para trabajar con suspense podéis usar react-query o useSWR que implementan el flujo con Suspense e implementan su propia cache**

Para mostrar las diferencias de cargas de datos entre la carga normal y **Suspense** he creado una app de ejemplo llamada [**InstaCat**](https://67gy6.csb.app). El código lo puedes encontrar en [Github](https://github.com/IvanRodriCalleja/instacat) y puedes jugar con ello en [CodeSandbox](https://codesandbox.io/s/festive-fermat-67gy6).

_app_

![instacat](/assets/images/posts/2020-03-18-react-data-fetching-con-suspense/app.gif)

Como podeis observar en el repositorio, los ejemplos de **NormalFetch** y **SuspenseFetch** usan los mismo componentes y servicios **(estos son fake)**, lo único que cambia es la implementación de ellos.

## Simple Fetch

_NormalFetch.js_

```javascript
import React, { useState, useEffect } from "react";

import { CatsSkeleton } from "./shared/CatsSkeleton";
import { CatsError } from "./shared/CatsError";
import { CatsList } from "./shared/CatsList";

import { fetchCats } from "../service/catsApi";

export const NormalFetchData = () => {
  const [{ data, error, isLoading }, dispatch] = useState({
    data: null,
    error: null,
    isLoading: true
  });

  useEffect(() => {
    fetchCats()
      .then(data => dispatch({ data, error: null, isLoading: false }))
      .catch(error => dispatch({ data: null, error, isLoading: false }));
  }, []);

  if (isLoading) return <CatsSkeleton />;
  if (error) return <CatsError error={error} />;
  return <CatsList cats={data} />;
};
```

_SuspenseFetch.js_

```javascript
import React, { Suspense } from "react";
import { unstable_createResource as createResource } from "../packages/react-cache";

import { CatsSkeleton } from "./shared/CatsSkeleton";
import { CatsList } from "./shared/CatsList";
import { ErrorBoundary } from "./shared/ErrorBoundary";

import { fetchCats } from "../service/catsApi";

export const SuspenseFetchData = () => {
  return (
    <ErrorBoundary>
      <Suspense fallback={<CatsSkeleton />}>
        <Pets />
      </Suspense>
    </ErrorBoundary>
  );
};

const petsResource = createResource(fetchCats);

const Pets = () => {
  const pets = petsResource.read();
  return <CatsList cats={pets} />;
};
```

En ambos casos la **UI** se comporta de la misma manera:

![fetch-data](/assets/images/posts/2020-03-18-react-data-fetching-con-suspense/data.gif)

Es casi el mismo ejemplo que hemos visto al inicio del blog para comparar ambas, los cambios más visibles son la manera en la que se hace el **fetch** de los datos, uno lo hace dentro de un **useEffect** y otro lo hace en tiempo de render con la ayuda de **react-cache**, esta ayuda nos permite usar **Suspense** y **ErrorBoundary** para controlar los distintos estados de nuestra llamada.

[Ejemplo simple fetch](https://67gy6.csb.app/normal-fetch-data)

[Ejemplo suspense fetch](https://67gy6.csb.app/suspense-fetch-data)

## Simple Fetch Error

_NormalFetchError.js_

```javascript
import React, { useState, useEffect } from "react";

import { CatsSkeleton } from "./shared/CatsSkeleton";
import { CatsError } from "./shared/CatsError";
import { CatsList } from "./shared/CatsList";

import { fetchCatsError } from "../service/catsApi";

export const NormalFetchError = () => {
  const [{ data, error, isLoading }, dispatch] = useState({
    data: null,
    error: null,
    isLoading: true
  });

  useEffect(() => {
    fetchCatsError()
      .then(data => dispatch({ data, error: null, isLoading: false }))
      .catch(error => dispatch({ data: null, error, isLoading: false }));
  }, []);

  if (isLoading) return <CatsSkeleton />;
  if (error) return <CatsError error={error} />;
  return <CatsList cats={data} />;
};
```

_SuspenseFetchError.js_

```javascript
import React, { Suspense } from "react";
import { unstable_createResource as createResource } from "../packages/react-cache";

import { CatsSkeleton } from "./shared/CatsSkeleton";
import { CatsList } from "./shared/CatsList";
import { ErrorBoundary } from "./shared/ErrorBoundary";

import { fetchCatsError } from "../service/catsApi";

export const SuspenseFetchError = () => {
  return (
    <ErrorBoundary>
      <Suspense fallback={<CatsSkeleton />}>
        <Pets />
      </Suspense>
    </ErrorBoundary>
  );
};

const petsResource = createResource(fetchCatsError);

const Pets = () => {
  const pets = petsResource.read();
  return <CatsList cats={pets} />;
};
```

En este caso el comportamiento del **UI** también es completamente igual:

![fetch-data-error](/assets/images/posts/2020-03-18-react-data-fetching-con-suspense/error.gif)

Es exactamente el mismo caso que el anterior lo único que cambia en este caso es la función que hace el fetch de los datos (**fetchCatsError**). En este caso siempre retorna un error para que podamos probar el caso de error tanto con el fetch normal o con **ErrorBoundary**.

## Fetch race condition

Para aquellos que no sepais que es un **race condition** son aquellos casos en los que lanzamos el mismo **fetch** multiples veces y el orden en que se resuelven las promesas no es el deseado y nos quedamos con una **UI** inconsistente. Para este caso he puesto unos botones de búsqueda y las peticiones tienen un delay random de máximo **3000ms**. Si hacemos 4 peticiones rápidamente y se resuelven en un orden distinto al enviado (esto puede pasar ya que dependemos del servidor) vamos a tener unos datos que no queremos tener.

_NormalFetchRaceCondition.js_

```javascript
import React, { useState, useEffect } from "react";

import { CatsSkeleton } from "./shared/CatsSkeleton";
import { CatsError } from "./shared/CatsError";
import { CatsList } from "./shared/CatsList";
import { RadioGroup } from "./shared/RadioGroup";

import { searchCats } from "../service/catsApi";

const options = [
  "sukiicat",
  "albertbabycat",
  "smoothiethecat",
  "realgrumpycat"
];

export const NormalFetchRaceCondition = () => {
  const [search, setSearch] = useState("");

  const [{ data, error, isLoading }, dispatch] = useState({
    data: null,
    error: null,
    isLoading: true
  });

  useEffect(() => {
    dispatch(state => ({ ...state, isLoading: true }));
    searchCats(search)
      .then(data => dispatch({ data, error: null, isLoading: false }))
      .catch(error => dispatch({ data: null, error, isLoading: false }));
  }, [search]);
  console.log({ data, error, isLoading });

  return (
    <>
      <RadioGroup
        selectedValue={search}
        onChange={setSearch}
        options={options}
        name="search"
      />
      {isLoading && <CatsSkeleton />}
      {error && <CatsError error={error} />}
      {!isLoading && !error && <CatsList cats={data} />}
    </>
  );
};
```

_Normal fetch race condition_
![race-condition](/assets/images/posts/2020-03-18-react-data-fetching-con-suspense/race-condition.gif)

Como podemos ver, en este case nuestra **UI** es inconsistente, acabamos mostrando una **card** distinta al filtro aplicado, esto es debido a que no hay control de las peticiones y la última que se resuelva, va a ser la que mostraremos (no tiene por que ser la última petición que hemos hecho). Para solventar este problema tendríamos que cancelar la petición anterior.

_SuspenseFetchRaceCondition.js_

```javascript
import React, { Suspense, useState } from "react";
import { unstable_createResource as createResource } from "../packages/react-cache";

import { CatsSkeleton } from "./shared/CatsSkeleton";
import { CatsList } from "./shared/CatsList";
import { ErrorBoundary } from "./shared/ErrorBoundary";
import { RadioGroup } from "./shared/RadioGroup";

import { searchCats } from "../service/catsApi";

const options = [
  "sukiicat",
  "albertbabycat",
  "smoothiethecat",
  "realgrumpycat"
];

export const SuspenseFetchRaceCondition = () => {
  const [search, setSearch] = useState("");

  return (
    <>
      <RadioGroup
        selectedValue={search}
        onChange={setSearch}
        options={options}
        name="search"
      />
      <ErrorBoundary>
        <Suspense fallback={<CatsSkeleton />}>
          <Pets search={search} />
        </Suspense>
      </ErrorBoundary>
    </>
  );
};

const petsResource = createResource(searchCats);

const Pets = ({ search }) => {
  const pets = petsResource.read(search);
  return <CatsList cats={pets} />;
};
```

_Suspense fetch race condition_
![race-condition](/assets/images/posts/2020-03-18-react-data-fetching-con-suspense/suspense-race-condition.gif)

En este caso si usamos **Suspense** con **react-cache** no tenemos que preocuparnos de que esto nos pase ya que en este caso dependemos de la **key** con la que hemos pedido nuestro rescurso, como nuestra **key** es la propia búsqueda, siempre vamos a tener los datos de la **key** actual.

## Listado de peticiones

Hay ocasiones en las que tenemos que realizar varias peticiones al servidor para cargar distintas partes de una misma vist. En estos casos tenemos que manejar más de una llamada con cada uno de sus estados. Para ello vamos a imaginar que cada una de las cards del listado se carga independientemente.

_NormalFetchList.js_

```javascript
import React, { useEffect, useState } from "react";
import styled from "styled-components";

import { SkeletonCard } from "./shared/catsSkeleton/SkeletonCard";
import { CatsError } from "./shared/CatsError";
import { CatCard } from "./shared/catsList/CatCard";

import { fetchCat } from "../service/catsApi";

const CatsListContainer = styled.section`
  margin-top: 56px;
`;

export const NormalFetchList = () => (
  <CatsListContainer>
    <NormalFetchListCat id={0} />
    <NormalFetchListCat id={1} />
    <NormalFetchListCat id={2} />
    <NormalFetchListCat id={3} />
  </CatsListContainer>
);

const NormalFetchListCat = ({ id }) => {
  const [{ data, error, isLoading }, dispatch] = useState({
    data: null,
    error: null,
    isLoading: true
  });

  useEffect(() => {
    fetchCat({ id })
      .then(data => dispatch({ data, error: null, isLoading: false }))
      .catch(error => dispatch({ data: null, error, isLoading: false }));
  }, []);

  if (isLoading) return <SkeletonCard />;
  if (error) return <CatsError error={error} />;
  return <CatCard cat={data} />;
};
```

_Ejemplo múltiples fetch_
![race-condition](/assets/images/posts/2020-03-18-react-data-fetching-con-suspense/list-fetch.gif)

Este sería el comportamiento que tendríamos en el caso normal, no podríamos controlar la resolución de la llamada y las cards aparecerían cada una en un momento distinto, esto puede hacer que nos de una sensación mala de carga. Si quisiéramos mostrar todos a la vez, tendríamos que hacer la carga de todos en el componente padre y resolverlo con un `Promise.all` pero en ese caso si una de las llamadas fallase, nuestra promesa fallaría y no podríamos ver ninguno de los casos que no han fallado.

Para el caso de **Suspense** también han pensado en ello, en este caso existe un componente **SuspenseList**, este componente coordina todos los componentes **Suspense** que han saltado al primer nivel y nos permite configurar como queremos mostrar el loading y como resolverlo cuando las llamadas se van resolviendo. Este componente admite 2 props:

- **revealOrder**: Define el order en el que los hijos son mostrados (**forwards**, **backwards**, **together"**)
- **tail**: Define como los Loadings son mostrados (**collapsed**, **hidden**)

_SuspenseFetchList.js_

```javascript
import React, { Suspense, SuspenseList, useState } from "react";
import styled from "styled-components";
import { unstable_createResource as createResource } from "../packages/react-cache";

import { SkeletonCard } from "./shared/catsSkeleton/SkeletonCard";
import { CatCard } from "./shared/catsList/CatCard";
import { RadioGroup } from "./shared/RadioGroup";
import {
  SuspenseListConfigProvider,
  useSuspenseListConfig
} from "./suspenseFetchList/SuspenseListConfigContext";

import { fetchCat } from "../service/catsApi";

const SuspenseFetchListContainer = styled.div`
  margin-top: 56px;
`;

const revealOrderOptions = ["forwards", "backwards", "together"];
const tailOptions = ["collapsed", "hidden"];

export const SuspenseFetchList = () => {
  const [revealOrder, setRevealOrder] = useState("forwards");
  const [tail, setTail] = useState("collapsed");

  return (
    <>
      <RadioGroup
        selectedValue={revealOrder}
        onChange={setRevealOrder}
        options={revealOrderOptions}
        name="revealOrder"
      />
      <RadioGroup
        selectedValue={tail}
        onChange={setTail}
        options={tailOptions}
        name="tail"
      />
      <SuspenseListConfigProvider revealOrder={revealOrder} tail={tail}>
        <SuspenseFetchListContainer>
          <SuspenseList revealOrder={revealOrder} tail={tail}>
            <Suspense fallback={<SkeletonCard />}>
              <Cat id={0} />
            </Suspense>
            <Suspense fallback={<SkeletonCard />}>
              <Cat id={1} />
            </Suspense>
            <Suspense fallback={<SkeletonCard />}>
              <Cat id={2} />
            </Suspense>
            <Suspense fallback={<SkeletonCard />}>
              <Cat id={3} />
            </Suspense>
          </SuspenseList>
        </SuspenseFetchListContainer>
      </SuspenseListConfigProvider>
    </>
  );
};

const catResource = createResource(
  fetchCat,
  ({ id, revealOrder, tail }) => `${id}${revealOrder}${tail}`
);

const Cat = ({ id }) => {
  const { revealOrder, tail } = useSuspenseListConfig();
  const cat = catResource.read({ id, revealOrder, tail });
  return <CatCard cat={cat} />;
};
```

_Ejemplo múltiples fetch suspense_
![race-condition](/assets/images/posts/2020-03-18-react-data-fetching-con-suspense/suspense-list.gif)

De esta manera tan sencilla podemos coordinar distintas peticiones y mostrarlas de una manera que nos convenga. En este caso podemos controlar los errores de las peticiones por separado (habría que añadir un ErrorBoundary por cada Suspense)

## Resumen

Usar **Suspense** para **data-fetching** nor permite hacerlo de una forma declarativa con una gran potencia, hay que tener en cuenta que todavía es experimental aunque hay librerías que ya se integran con **Suspense** para hacer la carga de datos (tanto react-query como useSWR soportan ambas maneras). En este caso yo he hecho el bundle de **react-cache** ya que es un paquete experimental. La ventaja de trabajar con una cache es que en ciertas ocasiones no será neceasio salir al servidor ya que los datos los habremos cargado previamente y serán renderizados de inmediato (si habéis jugado con los ejemplos anteriores habréis visto que si vuelves a una página con **Suspense** en la que ya habíais estado previamente , los datos se muestran instantáneamente).

Aún que podemos usar **Suspense** para data fetching todavía esta en versión **experimental** y puede ser que nos encontremos con algunos casos en los que no funciona. Respecto al paquete **react-cache** está en un estado **muy experimental**. Actualmente **react-cache** permite hacer **preload** de datos para así poder ir haciendo el fetch de los datos mientras hacemos el fetch de los **chunks**, de esta manera paralelizamos la carga de datos con la carga de código en el browser en vez de hacerlo secuencial (tendríamos que esperar a la carga de código para luego hacer la caraga de datos lo que lleva mas tiempo para dar feedback al usuario).

## CodeSandbox

<iframe
     src="https://codesandbox.io/embed/festive-fermat-67gy6?fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="festive-fermat-67gy6"
     allow="geolocation; microphone; camera; midi; vr; accelerometer; gyroscope; payment; ambient-light-sensor; encrypted-media; usb"
     sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"
   ></iframe>
