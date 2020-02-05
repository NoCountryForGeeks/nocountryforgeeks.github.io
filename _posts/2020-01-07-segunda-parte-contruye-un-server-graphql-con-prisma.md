---
layout: post
current: post
cover: assets/images/posts/2020-01-07-segunda-parte-contruye-un-server-graphql-con-prisma/cover.jpg
navigation: True
title: 'Prisma 2º parte: construye tu servidor GraphQL de una forma rápida y sencilla'
date: 2020-01-07 12:00:00
tags: mongodb prisma graphql-yoga
class: post-template
subclass: 'post'
author: ivanrodri
---

En el [primer post](/contruye-un-server-graphql-con-prisma) vimos como configurar nuestro **conector prisma** con nuestros modelos sobre una base de datos **MongoDB**, una vez configurado y desplegado, podíamos realizar todas las operaciones **CRUD** sobre nuestros modelos y podíamos lanzar todo tipo de queries.

Hoy vamos a ver como crear un servidor en **Node** y como conectarlo con nuestro **conector prisma** ya que de esta manera tendremos mas flexibilidad a la hora de generar lógica de negocio, autenticación, etc.

## Organizar proyecto

En el post anterior creamos todos los ficheros en la raíz del proyecto pero en mi caso prefiero tenerlos separados entre **server**, **database** y tener los modelos cada uno en un fichero:

![estructura proyecto](/assets/images/posts/2020-01-07-segunda-parte-contruye-un-server-graphql-con-prisma/fileStructure.png)

Para ello, primero he creado la carpeta **database** y he metido todos los ficheros relacionados con **prisma** como son _prisma.yml_, _datamodel.prisma_, seed.graphql y _docker-compose_.

Despues he creado la carpeta _models_ y he creado un fichero por cada uno de nuestros modelos que teníamos definidos en nuestro _datamodel.prisma_:

_customer.prisma_

```
type Customer implements Model {
  id: ID! @id
  createdAt: DateTime! @createdAt
  updatedAt: DateTime! @updatedAt
  name: String! @unique
  address1: String!
  order: [Order!]! @relation(link: INLINE)
}
```

_model.prisma_

```
interface Model {
  id: ID! @id
  createdAt: DateTime! @createdAt
  updatedAt: DateTime! @updatedAt
}
```

_order.prisma_

```
type Order implements Model {
  id: ID! @id
  createdAt: DateTime! @createdAt
  updatedAt: DateTime! @updatedAt
  orderStatus: OrderStatus!
  customer: Customer!
  orderLines: [OrderLine!]! @relation(link: INLINE)
}
```

_orderLine.prisma_

```
type OrderLine implements Model {
  id: ID! @id
  createdAt: DateTime! @createdAt
  updatedAt: DateTime! @updatedAt
  quantity: Int!
  order: Order!
  product: Product! @relation(link: INLINE)
}
```

_orderStatus.prisma_

```
enum OrderStatus {
    NotOrdererd,
    InProcess,
    Delivered
}
```

_product.prisma_

```
type Product implements Model {
  id: ID! @id
  createdAt: DateTime! @createdAt
  updatedAt: DateTime! @updatedAt
  name: String! @unique
  price: Int!
}
```

Las definiciones de los modelos son exactamente iguales a lo que había, lo único es que ahora cada uno tiene su propio fichero. Los ficheros pueden usar modelos de otros ficheros sin tener que referenciar nada. Para que nuestro modelo siga funcionando como antes, tenemos que modificar el fichero _prisma.yml_ para indicar donde están nuestros modelos

_prisma.yml_

```yaml
endpoint: http://localhost:4466
databaseType: document
datamodel:
  - ./models/customer.prisma
  - ./models/model.prisma
  - ./models/order.prisma
  - ./models/orderLine.prisma
  - ./models/orderStatus.prisma
  - ./models/product.prisma

seed:
  import: ./seed.graphql

secret: mySuperSecret

generate:
  - generator: javascript-client
    output: ./generated/prisma-client/
```

Si ahora ejecutamos el comando `prisma deploy` dentro de nuestra carpeta _database_, no deberíamos de tener nungún cambio respecto al último despliegue ya que realmente no hemos modificado nada de nuestro modelo.

![prisma deploy](/assets/images/posts/2020-01-07-segunda-parte-contruye-un-server-graphql-con-prisma/prismaDeploy.png)

Por último vamos a crear una carpeta _server_ al mismo nivel que la carpeta _database_ y vamos a cambiar la salida del cliente generado por **prisma**, también vamos a aprovechar a añadir otro **hook** para generar el **schema graphql** de nuestro **conector prisma** y así poder usar esas definiciones en nuestro servidor. Para ello voy a borrar el cliente que tenía generado previamente y voy a modificar la propiedad **output** del fichero _prisma.yml_ para cambiar el destino:

_prisma.yml_

```yaml
endpoint: http://localhost:4466
databaseType: document
datamodel:
  - ./models/customer.prisma
  - ./models/model.prisma
  - ./models/order.prisma
  - ./models/orderLine.prisma
  - ./models/orderStatus.prisma
  - ./models/product.prisma

seed:
  import: ./seed.graphql

secret: mySuperSecret

generate:
  - generator: javascript-client
    output: ../server/generated/javascript-client/
  - generator: graphql-schema
    output: ../server/generated/prisma-schema.graphql
```

Si ahora ejecutamos `prisma generate` nos va a generar el cliente **prisma** en la carpeta _/server/generated/javascript-client/_ y el fichero _prisma-schema.graphql_ que tiene definido todo el _schema_ que soporta el **conector prisma**.

Después de estos pasos ya tenemos el proyecto estructurado y listo para continuar.

## Crear server node

Lo primero que tenemos que hacer es crear un _package.json_ para poder añadir todas nuestras dependencias **node** que vamos a necesitar, para ello ejecutamos en el raíz del proyecto el comando `yarn init`y rellenamos las preguntas:

![yarn init](/assets/images/posts/2020-01-07-segunda-parte-contruye-un-server-graphql-con-prisma/yarnInit.png)

Ahora podemos añadir las dependencias necesarias a nuestro proyecto, de momento solo necesitamos instalar **graphql-yoga** y **prisma-client-lib**, para ello ejecutamos `yarn add graphql-yoga prisma-client-lib`.

Con las dependencias instaladas ya podemos configurar nuestro servidor con **graphql-yoga** para ello vamos a hacerlo en el fichero _/server/index.js_.

_index.js_

```javascript
const {GraphQLServer} = require('graphql-yoga');
const {prisma} = require('./generated/javascript-client');

const server = new GraphQLServer({
  context: req => ({
    ...req,
    prisma,
  }),
});

const options = {
  port: 8000,
  cors: {
    origin: '*',
  },
};

server.start(options, args =>
  console.log(`Server is running on http://localhost:${args.port}`)
);
```

Tenemos que importar el servidor de **graphql-yoga** y el cliente autogenerado de **prisma**, dentro del contexto del servidor, pasamos nuestro cliente de **prisma** para poder consumirlo más adelante. Configuramos las opciones del puerto y **CORS** y ejecutamos el server, para probarlo, tenemos que ejecutar desde nuestro raíz el comando `node ./server/index.js`

![server no schema error](/assets/images/posts/2020-01-07-segunda-parte-contruye-un-server-graphql-con-prisma/serverNoSchema.png)

Cuando corremos el comando, nos da un error indicando que no hemos definido un _schema_, esto es debido a que tenemos que definir el **schema graphql** para nuestro servidor y crear sus correspondientes _resolvers_. Por los tanto voy a crear un fichero _schema.graphql_ dentro de la carpeta _server_, en el solo voy a definir una query para obtener todos los productos.

_schema.graphql_

```graphql
# import Product from "./generated/prisma-schema.graphql"

type Query {
  products: [Product!]!
}
```

Nuestro **schema graphql** define una query llamada **products** que nos retorna todo el listado de productos, en este caso no admite ningun tipo de filtro. El tipo de objeto que retorna lo importamos del **schema graphql** del **conector prisma**, este lo hemos autogenerado metiendo el hook correspondiente a nuestro \_prisma.yml** y en el tenemos la definición completa del **schema** del **conector prisma\*\*.

Tenemos que diferenciar entre el **modelo** que tenemos definido para nuestra base de datos (todos los ficheros dentro de la carpeta models), el **schema graphql** autogenerado con `prisma generate`que nos da toda la interfaz que define nuestro **conector** y el **schema graphql** que definimos para nuestro servidor.

Lo más probable es que no queramos exponer todas las operaciones que nos define el conector o quizá queramos no exponer todos los campos de los modelos o exponer campos computados o incluso meter autorización, por eso definimos nuestro **schema graphql** específico para nuestro servidor.

Ahora tenemos que indicar al servidor donde está nuestra definición del _schema_, para ello añadiremos la propiedad **typeDefs** con este objetivo.

_index.js_

```javascript
const path = require('path');
const {GraphQLServer} = require('graphql-yoga');
const {prisma} = require('./generated/javascript-client');

const server = new GraphQLServer({
  typeDefs: path.resolve(__dirname, 'schema.graphql'),
  context: req => ({
    ...req,
    prisma,
  }),
});

const options = {
  port: 8000,
  cors: {
    origin: '*',
  },
};

server.start(options, args =>
  console.log(`Server is running on http://localhost:${args.port}`)
);
```

Antes de poder ejecutar nuestro servidor, tenemos que crear los **resolvers** correspondientes a nuestro schema graphql de nuestro servidor.

_index.js_

```javascript
const path = require('path');
const {GraphQLServer} = require('graphql-yoga');
const {prisma} = require('./generated/javascript-client');

const resolvers = {
  Query: {
    products: (parent, args, ctx, info) => ctx.prisma.products({}, info),
  },
};

const server = new GraphQLServer({
  typeDefs: path.resolve(__dirname, 'schema.graphql'),
  resolvers,
  context: req => ({
    ...req,
    prisma,
  }),
});

const options = {
  port: 8000,
  cors: {
    origin: '*',
  },
};

server.start(options, args =>
  console.log(`Server is running on http://localhost:${args.port}`)
);
```

A la configuración de nuestro servidor de **graphql-yoga** tenemos que pasarle la configuración con los resolvers que hemos definido en nuestro schema (tienen que existir todos los resolvers que hemos definido, sino, el servidor nos dará un error al arrancar). En este caso, hemos creado un objeto que contiene la propiedad **Query** como la definida en el schema y esta propiedad a su vez es otro objeto que contiene los resolvers, en este caso **products**.

El resolver es una función que admite 4 parámetros, entre ellos el contexto, en el que previamente habíamos inyectado el cliente autogenerado de **prisma**, en este caso a través de el cliente, podemos lanzar una query al servidor de **prisma** `ctx.prisma.products({}, info)` el primer parámetro que recibe, son los filtros que vamos a aplicar a esta query (en nuestro caso nada) y el segundo parámetro son los campos que se han pedido en la query (id, nombre, etc).

Dentro de **ctx.prisma** están todas las operaciones disponibles sobre todos los modelos que tiene definido el **conector prisma** como puede ser **orders**. **orderLine**, **createOrderLine**, **deleteOrderLine**, etc. Son todas aquellas que puedes encontrar en la documentación del playground del **conector** `http://localhost:4466/`.

Si ahora ejecutamos nuestro servidor `node ./server/index.js`, arrancará en `http://localhost:8000/` donde tendremos disponible el **playgound** de graphql, en este caso, solo tenemos disponible la query de **products** que es lo que hemos definido en el schema del servidor.

![server no schema error](/assets/images/posts/2020-01-07-segunda-parte-contruye-un-server-graphql-con-prisma/serverRunning.png)

![server no schema error](/assets/images/posts/2020-01-07-segunda-parte-contruye-un-server-graphql-con-prisma/serverProducts.png)

El resultado debería de ser el mismo que si lanzamos esta misma query en el **playground** del conector de prisma `http://localhost:4466/` (hay que recordar que ahora tenemos 2 playgrounds, uno de nuestro conector y otro de nuestro server).

![server no schema error](/assets/images/posts/2020-01-07-segunda-parte-contruye-un-server-graphql-con-prisma/prismaProducts.png)

**> NOTA: No compareis los ids del primer post con los del segundo post porque he tenido que regenerar todas las imagenes docker y lanzar la semilla de nuevo por lo que no son iguales, pero en las 2 imagenes anteriores si que han de ser iguales**.

Por si no lo recordabais, a nuestro conector **prisma** lo configuramos con un **secret** en nuestro _prisma.yml_ para que no fuese accesible desde fuera por cualquiera pero nosotros estamos accediendo desde nuestro **servidor node** a nuestro **conector prisma** sin configurar este **secret** y nos está funcionando, esto es debido a que cuando ejecutamos `prisma generate` nos genera el cliente prisma ya configurado con el **secret** para que podamos usarlo directamente, si cambiasemos este **secret** deberíamos de regenerar el cliente (lo hace por defecto) y re-desplegar nuestro **servidor node** para que llame con la nueva **secret**.

Con la parte del servidor añadida, la foto de la arquitectura de nuestro servicio sería la siguiente:

![service arquitecture](/assets/images/posts/2020-01-07-segunda-parte-contruye-un-server-graphql-con-prisma/prismaArquitecture.PNG)

## Resumen

En este post hemos visto como crear nuestro **servidor node** y como conectarlo a nuestro **conector prisma** con **graphql-yoga**, para ello hemos:

- Reestructurado el proyecto para separar entre **database** y **server**
- Separar los modelos cada uno en su fichero
- Añadir el hook en el fichero _prisma.yml_ para generar el **schema graphql** del **conector prisma**
- Crear nuestro servidor con **graphql-yoga**
- Crear el **schema graphql** para nuestro **servidor node**
- Crear los resolvers necesarios para nuestro **servidor node** para cumplir con el **schema graphql**

Ahora que ya sabemos como consumir nuestro conector desde el servidor, podemos generar todas las **queries**, **mutations** y **subscriptions** que necesitemos para crear un flujo completo para realizar un pedido, pero eso lo dejamos ya para el siguiente post 😝

Por si te has perdido el post anterior, lo puedes encontrar [aquí](/contruye-un-server-graphql-con-prisma)

El [código completo](https://github.com/NoCountryForGeeks/prisma-with-graphql-yoga-server) de los posts los puedes encontrar en el repo de **GitHub** y [aquí](https://github.com/NoCountryForGeeks/prisma-with-graphql-yoga-server/tree/post-2-setup-server) puedes encontrar el código de este post.
