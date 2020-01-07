---
layout: post
current: post
cover: assets/images/posts/2019-08-22-contruye-un-server-graphql-con-prisma/coverPage.jpg
navigation: True
title: 'Prisma: construye tu servidor GraphQL de una forma rápida y sencilla'
date: 2019-08-21 12:00:00
tags: mongodb prisma graphql-yoga
class: post-template
subclass: 'post'
author: ivanrodri
---

Este es el primer post de una serie donde vamos a ver como podemos crear un servidor **GraphQL** para un carrito de la compra con [**Prisma**](https://www.prisma.io/) y [**graphql-yoga**](https://github.com/prisma/graphql-yoga) de una forma rápida y sencilla

# Requisitos

- [yarn](https://yarnpkg.com/lang/en/)
- [docker](https://www.docker.com/)

# Que es Prisma?

Antes de empezar a trabajar con **Prisma** vamos a comentar **¿Qué es?**.

**Prisma** es un conector de base de datos **GraphQL** en tiempo real que convierte la base de datos en un API **GraphQL**, podemos verlo como una especie de **ORM**, pero es mucho más poderoso que los **ORM** tradicionales.

Con **Prisma**, obtenemos un servidor (servidor **Prisma**) que actúa como un proxy para nuestra base de datos y un motor de consultas de alto rendimiento que se ejecuta en el servidor, lo que genera consultas de bases de datos reales para nosotros.

El API **GraphQL** de **prisma** proporciona abstracciones para desarrollar backends **GraphQL** flexibles y escalables.

**Prisma** nos genera un cliente (cliente **Prisma**), que podemos usar para interactuar con el servidor. **Prisma** también agrega un sistema de eventos en tiempo real a nuestra base de datos, para que podamos suscribirnos a los eventos de la base de datos en tiempo real.

La arquitectura de trabajo con **Prisma** sería la siguiente:

![Create database](/assets/images/posts/2019-08-22-contruye-un-server-graphql-con-prisma/prisma.png)

En este caso estamos hablando únicamente de generar una API **GraphQL** con prisma pero también podemos generar un API **REST** y **GRPC**.

## Características

- **Prisma** nos proporciona un API con un **tipado-seguro** desde el back-end hasta el front-end.

- **SDL** (Schema Definition Language) nos permite escribir nuestros **esquemas GraphQL** que define el API de operaciones **GraphQL** para nuestra base de datos. Este tipo de definición puede jugar el mismo papel que los **protos** en **GRPC**, nos sirve de interfaz para la generación de tipos entre front-end y back-end.

- **Prisma** maneja las migraciones de base de datos automáticamente.

- **Prisma** puede trabajar con distintos tipos de base de datos (tanto relacional como documental) y actualmente soporta **MySQL**, **MongoDB** y **PostgresSQL**.

- **Prisma** puede generar el API para **GraphQL**, **REST** y **GRPC**.

- **Prisma** funciona con bases de datos existente o **schema first**

- **Prisma** resuelve el problema N + 1 queries gracias al uso de dataloaders y su motor de queries.

- **Prisma** permite lanzar queries a través de distintas bases de datos usando **schema stitching** o en el futuro con **Apollo federation**

- **Prisma** nos permite **real-time** API con **Subscriptions**

- **Prisma** nos proporciona un sistema de eventos con la base de datos a los que podemos suscribirnos (**Prisma** nos asegura que cualquier base de datos que soporte, nos permitirá suscribirnos a cualquier tipo de evento aún que la base de datos no lo soporte)

Ahora que tenemos un poquito mas de contexto sobre que es **prisma**, vamos a empezar a construir nuestro servidor **GraphQL**.

# Instalar prisma

Para comenzar, necesitamos instalar [**Prisma CLI**](https://www.prisma.io/docs/prisma-cli-and-configuration/using-the-prisma-cli-alx4/) el cual nos va a permitir arrancar nuestro proyecto.

```
yarn global add prisma
```

# Crear proyecto

```
cd ruta_carpeta
yarn init
```

En este punto nos preguntará si queremos partir de una base de datos existente o crear un proyecto desde cero, también nos preguntará si queremos alojar nuestro proyecto en desarrollo, en sus servidores o en local con **docker**. Sus servidores son gratuitos para desarrollo y genera un namespace único para cada desarrollador, lo que no genera conflictos entre otros desarrolladores.

En mi caso voy a elegir la opción **"Create new database"** para tener todo en local.

![Create database](/assets/images/posts/2019-08-22-contruye-un-server-graphql-con-prisma/createDatabase.png)

Después nos va a preguntar la base de datos que queremos usar, en mi caso voy a elegir **MongoDB**.

![Create database](/assets/images/posts/2019-08-22-contruye-un-server-graphql-con-prisma/mongoDatabase.png)

Y por último nos va a preguntar por el tipo de lenguaje en el que vamos a crear el **cliente prisma**, en este caso yo voy a elegir **JavaScript**.

![Create database](/assets/images/posts/2019-08-22-contruye-un-server-graphql-con-prisma/prismaClient.png)

# Echando un ojo al proyecto generado

Actualmente en el proyecto tenemos 3 ficheros y una carpeta **generated**.

_docker-compose.yml_

```yaml
version: '3'
services:
  prisma:
    image: prismagraphql/prisma:1.34
    restart: always
    ports:
      - '4466:4466'
    environment:
      PRISMA_CONFIG: |
        port: 4466
        # uncomment the next line and provide the env var PRISMA_MANAGEMENT_API_SECRET=my-secret to activate cluster security
        # managementApiSecret: my-secret
        databases:
          default:
            connector: mongo
            uri: 'mongodb://prisma:prisma@mongo'
  mongo:
    image: mongo:3.6
    restart: always
    # Uncomment the next two lines to connect to your your database from outside the Docker environment, e.g. using a database GUI like Compass
    # ports:
    # - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: prisma
      MONGO_INITDB_ROOT_PASSWORD: prisma
    ports:
      - '27017:27017'
    volumes:
      - mongo:/var/lib/mongo
volumes: mongo:
```

Este fichero contiene la configuración de los contenedores que vamos a necesitar para ejecutar **Prisma** en local con **docker**, este fichero contiene la imagen para arrancar el **servidor prisma** y la base de datos **MongoDb**

_prisma.yml_

```yaml
endpoint: http://localhost:4466
datamodel: datamodel.prisma
databaseType: document

generate:
  - generator: javascript-client
    output: ./generated/prisma-client/
```

En este fichero tenemos la configuración para el nuestro **servidor prisma** cuando lo despleguemos.

La propiedad **"endpoint"** es la url donde se va a alojar nuestro servidor, de usar el los servidored de **prisma** para desarrollo, la url sería mas o menos algo así: `https://eu1.prisma.sh/prisma_user/server/dev`.

La propiedad **"datamodel"** es la localización de fichero _datamodel.prisma_ donde se encuentra nuestro modelo que vamos a generar en la base de datos (mas a delante lo comentaré).

La propiedad **"databaseType"** es en la cual indicamos que tipo de base de datos que vamos a usar, como en nuestro caso habíamos elegido **MongoDB**, nos pone **documental**.

Por último, tenemos la propiedad **"generate"** donde le indicamos el tipo de cliente que vamos a auto generar para nuestro servidor, como habíamos elegido **javascript** tenemos **javascript-client**, y el output es donde nos va a generar el cliente.

_datamodel.prisma_

```
type User {
  id: ID! @id
  name: String!
}
```

En este fichero encontramos la definición de nuestro **esquema GraphQL** que va a manejar **prisma** con la base de datos. Este fichero, usa sintaxis **SDL** y a través de él podemos configurar las referencias/relaciones entre entidades. El modelo no es necesario que esté en un único fichero, se puede separar en múltiples ficheros, en este caso, tendremos que indicar en la propiedad **datamodel** del fichero _prisma.yml_ donde se encuentran nuestros ficheros.

# Lanzar nuestro servidor prisma

Ahora tenemos que ejecutar los 2 contenedores **docker** necesarios para correr nuestro servidor prisma en local, uno es para la base de datos **MongoDB** y otro es para el **servidor prisma**

```
docker-compose up -d
```

Una vez las imágenes de los container se hayan descargado y estén corriendo, deberíais tener algo parecido a esto:

![Docker images](/assets/images/posts/2019-08-22-contruye-un-server-graphql-con-prisma/dockerImages.png)

Por último, tenemos que desplegar nuestro código dentro de los contenedores para poder consumirlo.

```
prisma deploy
```

Nos indicará que ha generado cambios en el esquema con una entidad **User** con los campos **id** de tipo **ID!** y **name** de tipo **string**. Estos cambios son la creación del modelo que tenemos en el fichero _datamodel.prisma_. También nos indicará la url donde podemos empezar a consumir nuestro **server prisma** que se encuentra por defecto en **http://localhost:4466**. Si entramos en la url, tenemos accesible la interfaz con el **Playground** para poder empezar a lanzar queries directamente contra nuestra base de datos.

![Grapiql](/assets/images/posts/2019-08-22-contruye-un-server-graphql-con-prisma/grapiql.png)

# Repaso

Hasta ahora no he explicado nada del otro mundo, simplemente hemos instalado el **prisma cli** el cual nos permite arrancar un nuevo proyecto. Despues hemos seguido los pasos eligiendo ejecutar **prisma** en local con **docker**, hemos elegido **MongoDB** como base de datos y que nuestro cliente prisma se genere con **javascript**. Los siguientes pasos nos los ha dado directamente **prisma** tras la generación del proyecto. Primero ejecutamos los contenedores **docker** con un **docker-compose** y despues hemos desplegado nuestro proyecto en los contenedore.

Con estos simples pasos, tenemos nuestro **servidor prisma** y la base de datos corriendo. Tenemos el **Playground** corriendo en la url `http://localhost:4466` donde podemos empezar a a lanzar **queries** y **mutations** que se han generado para nuestro esquema.

# API GraphQL

He comentado que **Prisma** nos genera un API **GraphQL** y también nos genera un cliente **javascript** pero... **¿Cómo?**.

En nuestro proyecto, tenemos definido un esquema en _datamodel.prisma_ que es el esquema que nos genera por defecto **Prisma**. A partir del esquema, **Prisma** auto genera un cliente **javascript** y el API **GraphQL** (lo que consumimos a través del **Playground**). **Prisma** nos genera todas las operaciones posibles para cada una de las entidades del esquema (tanto en el cliente como en el API).

## Queries

- **user**: retorna un usuario filtrado por id
- **users**: retorna un listado de usuarios, se puede incluir clausula **where** filtrando por cada uno de los campos con todo tipo de opciones y también puede paginarse.

## Mutations

- **createUser**: permite crear un nuevo usuario
- **updateUser**: permite actualizar un usuario a través de su id
- **updateManyUsers**: permite actualizar varios usuarios que cumplan la condición **where**
- **upsertUser**: permite actualizar un usuario si cumple la condición **where** de lo contrario lo creará.
- **deleteUser**: permite eliminar un usuario a través del id.
- **deleteManyUsers**: permite eliminar múltiples usuarios que cumplan la condición **where**

Para las clausulas **where**, **prisma** nos genera una inmensa variedad de opciones de filtrado para cada uno de los campos de nuestro modelo, por ejemplo `name_starts_with`, `name_not_starts_with`, `name_in`, etc`

# Creando nuestro modelo

La idea es construir el modelo de una tienda donde existen usuarios, que pueden realizar un pedido con productos. Para eso voy a definir el siguiente modelo.

![database model](/assets/images/posts/2019-08-22-contruye-un-server-graphql-con-prisma/databaseModel.png)

Ahora nos toca representarlo en _datamodel.prisma_ mediante lenguaje SDL.

_datamodel.prisma_

```javascript
interface Model {
  id: ID! @id
  createdAt: DateTime! @createdAt
  updatedAt: DateTime! @updatedAt
}

type Customer implements Model {
  id: ID! @id
  createdAt: DateTime! @createdAt
  updatedAt: DateTime! @updatedAt
  name: String! @unique
  address1: String!
  order: [Order!]! @relation(link: INLINE)
}

enum OrderStatus {
    NotOrdererd,
    InProcess,
    Delivered
}

type Order implements Model {
  id: ID! @id
  createdAt: DateTime! @createdAt
  updatedAt: DateTime! @updatedAt
  orderStatus: OrderStatus!
  customer: Customer!
  orderLines: [OrderLine!]! @relation(link: INLINE)
}

type Product implements Model {
  id: ID! @id
  createdAt: DateTime! @createdAt
  updatedAt: DateTime! @updatedAt
  name: String! @unique
  price: Int!
}

type OrderLine implements Model {
  id: ID! @id
  createdAt: DateTime! @createdAt
  updatedAt: DateTime! @updatedAt
  quantity: Int!
  order: Order!
  product: Product! @relation(link: INLINE)
}
```

Comentemos un poco lo que hemos hecho con nuestro modelo, hemos definido una interfaz **Model** que van a implementar todos nuestros modelo. En ella estamos obligando a implementar la propiedad `id: ID! @id` esto nos indica que vamos a tener un campo **id** de tipo **ID** que es obligatorio (por el **!**) y que va a ser manejado directamente por **prisma** debido al uso de la directiva **@id**.

En la interfaz también hemos declarado las propiedades `createdAt: DateTime! @createdAt` y `updatedAt: DateTime! @updatedAt`, estos campos los manejará automáticamente **prisma** cuando la entidad se cree y se actualice gracias a las directivas `@createdAt`y `@updatedAt`.

Con la directiva `@unique` podemos indicar a **Prisma** que nuestro campo es único, el se encargara de generar el error correspondiente si intentamos insertar un dato duplicado.

Para las referencias/relaciones, podemos usar directamente un tipo o un array de tipos para declarar 1 o N referencias.

Nos encontramos una sintaxis `order: [Order!]!` de esta manera indicamos que nuestro campo order, no puede ser nulo y además que la referencia/relación que generemos no puede ser nula.

Para las referencias/relaciones, usamos una directiva para indicar que tipo de referencia/relación queremos, ya que en mongo, podemos tener documentos embebidos o no, en mi caso he decidido que se genere un documento a parte y que la relación se representa con el id al documento.

También podemos configurar el tipo de borrado que queremos para nuestras referencias/relaciones (yo no lo he configurado). Todas las opciones de configuración del **datamodel** las puedes encontrar [aquí](https://www.prisma.io/docs/datamodel-and-migrations/datamodel-MYSQL-knul/).

# Actualizando nuestro modelo

Ahora que hemos definido un nuevo modelo, tenemos que desplegarlo en nuestro servidor **prisma** ya que ahora mismo tenemos el modelo antiguo, para eso, tenemos que volver a ejecutar el comando:

`prisma deploy`

Una vez ejecutado, veremos un log del estilo:

```
λ prisma deploy
Deploying service `default` to stage `default` to server `local` 336ms

Changes:

  User (Type)
  - Deleted type `User`

  Customer (Type)
  + Created type `Customer`
  + Created field `id` of type `ID!`
  + Created field `createdAt` of type `DateTime!`
  + Created field `updatedAt` of type `DateTime!`
  + Created field `name` of type `String!`
  + Created field `address1` of type `String!`
  + Created field `order` of type `[Order!]!`

  Order (Type)
  + Created type `Order`
  + Created field `id` of type `ID!`
  + Created field `createdAt` of type `DateTime!`
  + Created field `updatedAt` of type `DateTime!`
  + Created field `orderStatus` of type `Enum!`
  + Created field `customer` of type `Customer!`
  + Created field `orderLines` of type `[OrderLine!]!`

  OrderLine (Type)
  + Created type `OrderLine`
  + Created field `id` of type `ID!`
  + Created field `createdAt` of type `DateTime!`
  + Created field `updatedAt` of type `DateTime!`
  + Created field `quantity` of type `Int!`
  + Created field `order` of type `Order!`
  + Created field `product` of type `Product!`

  Product (Type)
  + Created type `Product`
  + Created field `id` of type `ID!`
  + Created field `createdAt` of type `DateTime!`
  + Created field `updatedAt` of type `DateTime!`
  + Created field `name` of type `String!`
  + Created field `price` of type `Int!`

  OrderStatus (Enum)
  + Created enum OrderStatus with values `NotOrdererd`, `InProcess`, `Delivered`

  OrderLineToProduct (Relation)
  + Created an inline relation between `OrderLine` and `Product` in the column `product` of table `OrderLine`

  OrderToOrderLine (Relation)
  + Created an inline relation between `Order` and `OrderLine` in the column `orderLines` of table `Order`

  CustomerToOrder (Relation)
  + Created an inline relation between `Customer` and `Order` in the column `order` of table `Customer`

Applying changes 541ms
Generating schema 37ms
```

Este es el log de cambios que ha generado en la migración respecto a nuestro modelo anterior, como vemos, ha eliminado la entidad **User** existente y ha creado todas las nuevas entidades con sus campos y relaciones.

Si ejecutamos el **Playground** o inspeccionamos el cliente **javascript** autogenerado, podemos observar que han cambiado, nuestro API **GraphQL** ahora es distinta, ahora tenemos todas las opciones que teníamos para nuestra entidad usuario (user, users, createUsers, updateUser...) las tenemos para cada una de nuestras entidades (updateProduct, createOrder, deleteCustomer...).

# Semilla de datos

Si ahora mismo ejecutamos el **Playground** y hacemos una query de productos, no vamos a obtener ningún resultado.

![empty data](/assets/images/posts/2019-08-22-contruye-un-server-graphql-con-prisma/emptyData.png)

**Prisma** nos permite configurar un fichero como semilla para insertar datos en la base de datos, la semilla la creamos lanzando mutations generadas por **prisma**

_seed.graphql_

```javascript
mutation {
  product1: createProduct(data: {name: "Product 1", price: 10}){ id }
  product2: createProduct(data: {name: "Product 2", price: 20}){ id }
  product3: createProduct(data: {name: "Product 3", price: 30}){ id }
  product4: createProduct(data: {name: "Product 4", price: 40}){ id }
  product5: createProduct(data: {name: "Product 5", price: 50}){ id }
}
```

Usando la mutation **createProduct** hemos definido lanzar un **batch** de **mutations** para crear productos de ejemplo.

En nuestro _prisma.yml_ tenemos que añadir la referencia de nuestro **seed**

_prisma.yml_

```yaml
endpoint: http://localhost:4466
datamodel: datamodel.prisma
databaseType: document

generate:
  - generator: javascript-client
    output: ./generated/prisma-client/

seed:
  import: ./seed.graphql
```

Ahora tenemos que lanzar la semilla a la base de datos, para ello, **prisma** nos ofrece un comando:

`prisma seed`

Una vez ejecutado, si nos vamos a al **Playground** y ejecutamos la query de productos, veremos que ahora si tenemos datos.

![products data](/assets/images/posts/2019-08-22-contruye-un-server-graphql-con-prisma/productsData.png)

# Proteger nuestro servidor prisma

Ahora mismo nuestro **Servidor prisma**, está abierto a cualquier petición sin ningún tipo de autenticación y eso no es bueno si queremos que nuestros datos estén seguros y protegidos, para eso podemos securizarlo de una forma muy sencilla.

**Prisma** nos permite añadir la propiedad **secret** en nuestro _prisma.yaml_, esta propiedad es un string random con el cual se va a verificar el token en la llamada.

_prisma.yaml_

```yaml
endpoint: http://localhost:4466
datamodel: datamodel.prisma
databaseType: document

generate:
  - generator: javascript-client
    output: ./generated/prisma-client/

seed:
  import: ./seed.graphql

secret: mySuperSecret
```

De nuevo tenemos que desplegar nuestro servicio para que los cambios se hagan efectivos.

`prisma deploy`

Una vez desplegado nuestro cambio en el servidor, si intentamos ejecutar la query anterior de productos, vamos a ver que no estamos autorizados para acceder al **servidor prisma**.

![no autorizado](/assets/images/posts/2019-08-22-contruye-un-server-graphql-con-prisma/noAuthorized.png)

Para poder acceder a nuestro conector, tenemos que generar un token firmado con la misma **secret** que hemos añadido a nuestra configuración. En **prisma** también han pensado en ello, el **CLI** nos ofrece el comando.

`prisma token`

Cuando lo ejecutemos, en la consola nos mostrará un **token** totalmente válido para poder usar nuestro conector, ahora tendremos que ir al **Playground** y meterlo en la **header** de la request y con esto ya podremos usar nuestro conector teniendo nuestros datos autenticados.

![no autorizado](/assets/images/posts/2019-08-22-contruye-un-server-graphql-con-prisma/authorization.png)

# Recapitulando

Con esta serie de pasos tenemos un **servidor prisma** que nos expone un API **GraphQL** con el esquema definido en el **datamodel** y consumiendo una base de datos **MongoDB**. También tenemos un fichero con una semilla para añadir datos de prueba a nuestra base de datos y lo hemos securizado con un token.

Hay que tener en cuenta que es un servidor con una entrada **GraphQL** securizado y que puede ser totalmente consumido pero eso no significa que tengamos que tirar directamente de el con nuestro cliente (IOs, Android, react...). En este post solo hemos construido una parte de nuestro servidor, la parte de conexión de datos, pero nos falta crear nuestro servidor entre la parte **cliente** y el **servidor prisma**, ahora mismo solo hemos construido lo siguiente:

![arquitectura prisma](/assets/images/posts/2019-08-22-contruye-un-server-graphql-con-prisma/prismaArquitecture.png)

El servidor es necesario para meter sistema de autenticación mas flexible usando roles, proveedores de autenticación, etc. También necesitamos el server para meter toda la lógica de negocio que necesitemos ya que eso no lo podemos hacer en el **servidor prisma**.

Con esto termino por hoy, en el próximo post veremos como crear un servidor en **node** con **graphql-yoga** y como consumir el cliente generado por **Prisma** para poder consumirlo desde nuestros clientes.

El repositorio **Github** lo puedes encontrar [Aquí](https://github.com/NoCountryForGeeks/prisma-with-graphql-yoga-server/tree/post-1-setup-prisma)

[Aquí](/segunda-parte-contruye-un-server-graphql-con-prisma) puedes continuar con la segunda parte del post.
