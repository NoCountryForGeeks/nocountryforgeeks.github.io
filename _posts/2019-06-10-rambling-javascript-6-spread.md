---
layout: post
current: post
cover: assets/images/posts/2019-06-10-operador-spread-js-6/header.png
navigation: True
title: "Operador Spread - Javascript #6"
date: 2019-06-10 12:00:00
tags: javascript rambling-javascript
class: post-template
subclass: "post"
author: aclopez
---

"La simplicidad es la gloria de la expresión" Walt Whitman

# Operador Spread (...js)

Un operador que ayuda mucho a tener una programación inmutable y que aporta mucha legibilidad a nuestro código es el operador _spread_ o coloquialmente también conocido como "punto punto punto".

La traducción de _spread_ es propagador, y es que el operador `...` nos permite "propagar" las propiedades de los elementos a los que se les aplica. Propagar es una manera un poco abstracta de verlo. El objetivo es que las propiedades del elemento sobre el que se aplica el operador se expandan donde son esperadas, por ejemplo en los argumentos de una función.

```js
const array = [ 1, 2, 3 ];
const result = [ ...array, 4, 5, 6 ];

// result: [ 1, 2, 3, 4, 5, 6 ]
```

En el ejemplo, el array ```[ 1, 2, 3 ]``` se propaga en un nuevo array al ejecutar la segunda línea y dar valor a la variable _result_ con sus valores y los que se definen a continuación.

## Operador _spread_ en objetos

Los objetos también pueden ser expandidos con ```...```. Veamos el ejemplo:

```js
const obj = {
    name: 'pepe',
    age: 27
};

const result = {
    ...obj,
    country: 'spain'
}

// result: {
//      name: pepe,
//      age: 27,
//      country: 'spain'   
// }
```

El nuevo objeto ```result``` tiene las dos propiedades que tiene el objeto ```obj```. El objeto ```obj``` se ha expandido dentro de ```result```. Esto también se puede hacer sobreescribiendo alguna propiedad de esta manera:

```js
const obj = {
    name: 'pepe',
    age: 27,
    country: 'spain'
};

const result = {
    ...obj,
    country: 'france'
}

// result: {
//      name: pepe,
//      age: 27,
//      country: 'france' 
// }
```

Al sobreescribir la propiedad ```country``` el uso del operador _spread_ queda reducido a la expansión de todas las propiedades que no están sobreescritas. Así, podemos hacer que un objeto mute solo cambiando una propiedad de un modo sencillo.

## Operador _spread_ en _arrays_

En la definición del operador ```...``` hemos visto una manera de poder concatenar _arrays_. Ahora vamos a ver en profundidad las operaciones que podemos hacer con los mismos.

### Push en un _array_

La función push se hace más sencilla con este operador:

```js
const array = [ 1, 2, 3 ];
const arrayPush = [ 4, 5 ];

const result1 = Array.prototype.push.apply(array, arrayPush);
const result2 = array.push(...arrayPush);
// result1 & result2 = [ 1, 2, 3, 4, 5 ]
```

Al expandir los elementos de ```arrayPush``` en la variable ```array``` podemos usar la función _push_ directamente sin tener que recurrir al _push_ de la clase ```Array``` y todo queda mucho más legible. 

Este es un ejemplo para que veamos la equivalencia del _push_ de la clase _array_ con el uso de ```...```; pero lo que se suele emplear para el _push_ es directamente la concatenación de _arrays_ que veremos a continuación, ya que es equivalente a realizar cualquier añadido al _array_, aunque sea solo un elemento.

### Concatenación de _arrays_

Para concatenar dos _arrays_ se usa la función definida en el prototype de ```Array``` _concat_. Podemos obtener el mismo resultado con _concat_ que usando _spread_:

```js
const array = [ 4, 5, 6 ];
const result = [ 1, 2, 3 ].concat(array);

const resultWithSpread = [ 1, 2, 3, ...array]

// result & resultWithSpread = [ 1, 2, 3, 4, 5, 6 ]
```

En este caso, el resultado de concatenar los _arrays_ es el mismo; pero en mi opinión el operador ```...``` le da mucha más expresividad a nuestro código ya que lo hace mucho más fácil de leer.

### Operación _splice_ en un _array_

La función [_splice_](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Array/splice) combina dos _arrays_. Es una función algo compleja que recibe como parámetros el inicio de donde se inserta el nuevo _array_, el número de elementos que sustituye y el nuevo _array_:

```js
const numbers = [ 4, 5 ];
numbers.splice(0, 0, [1, 2, 3]);
// numbers: [ 1, 2, 3, 4, 5]

const numbers = [ 1, 2, 2, 4, 5 ];
numbers.splice(1, 2, [3, 3]);
// numbers: [ 1, 3, 3, 4, 5]

const numbers = [ 1, 2, 3, 5 ];
numbers.splice(3, 0, [4]);
// numbers: [ 1, 2, 3, 4, 5]
```

Estas operaciones usando el operador _spread_ pueden quedar mucho más legibles. Además otra ventaja de usar _spread_ es que se crea un nuevo _array_ y no se modifica el que existe como hace la función _splice_, lo que viene mejor para una programación inmutable. Para ver donde colocar el nuevo _array_ nos ayudaremos de la función [_slice_](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Array/slice), que nos da el subconjunto del _array_ que necesitamos:

```js
const array = [ 1, 2, 3 ];
const numbers = [ 4, 5 ];
const result = [
    ...array,
    ...numbers
];
// result: [ 1, 2, 3, 4, 5]
numbers.splice(0, 0, array);
// numbers: [ 1, 2, 3, 4, 5]

const array = [ 3, 3 ];
const numbers = [ 1, 2, 2, 4, 5 ];
const result = [
    ...numbers.slice(0, 1),
    ...array,
    ...numbers.slice(3)
];
// result: [ 1, 3, 3, 4, 5]
numbers.splice(1, 2, array);
// numbers: [ 1, 3, 3, 4, 5]

const array = [ 4 ];
const numbers = [ 1, 2, 3, 5 ];
const result = [
    ...numbers.slice(0, 3),
    ...array,
    ...numbers.slice(3)
];
// result: [ 1, 2, 3, 4, 5]
numbers.splice(3, 0, array);
// numbers: [ 1, 2, 3, 4, 5]

```

Si lo pensamos con detenimiento una vez vista la función _slice_ esto es equivalente:

```js
const array = [ 1, 2, 3 ];

const result1 = [ array.slice() ];
const result2 = [ ...array ];

// result1 & result2 = [ 1, 2, 3 ]

```

## Operador _spread_ y los parámetros Rest

El resultado del operador _spread_ también puede se puede poner al pasarle los parámetros a una función:

```js
const numbers = [ 1, 2, 3 ];

function sum(a, b, c) {
    return a + b + c;
}

const result = sum(...numbers);
// result: 6
```

Esto ya lo habíamos hecho en la función _push_ del _array_. Podemos combinarlo como queramos, usarlo es muy cómodo:

```js
const dateData = [1989, 1, 7]; 
const d = new Date(...dateData);
```

En cambio, si queremos poner la expresión ```...``` en los propios parámetros de una función, el comportamiento es el contrario. En lugar de convertir una _array_ en una lista de elementos, convierte una lista de elementos en un _array_. Podemos ver este comportamiento usando la función _suma_ del revés:

```js
// En su expresión con arrays
function sum(...numbers) {
    let total = 0;
    for (var i=0; i < numbers.length; i++) {
        total += numbers[i];
    }
    return total;
}

// Aunque mejor podemos poner un reduce
const sum = (...numbers) =>
    numbers.reduce(
        (total, nextNum) => total + nextNum, 
        0
    );

const result = sum(1, 2, 3);
// result: 6
```

Para acordarse de como poder usar _reduce_ podéis visitar [trabajando con arrays](https://www.nocountryforgeeks.com/rambling-javascript-3-arrays/).

La expresión ```...``` utilizada como parámetros de una función se denomina __parámetros Rest__ y, como hemos dicho anteriormente es justo la expresión contraria al operador _spread_.

### Destructuring y los parámetros rest

Se puede combinar los parámetros rest junto con el [destructuring](https://www.nocountryforgeeks.com/destructuring/).

```js
const numbers = [ 1, 2, 3 ];

const { firstNumber, ...otherNumbers } = numbers;

// firstNumber: 1
// otherNumbers: [ 2, 3 ]
```

Podemos combinarlo de la manera que queramos, usando ```...``` al principio, al final o en medio.

```js
const numbers = [ 1, 2, 3, 4, 5 ];

const { firstNumber, ...otherNumbers, lastNumber } = numbers;

// firstNumber:  1
// otherNumbers: [ 2, 3, 4 ]
// lastNumber:   5
```

Y podríamos usar todo esto en los parámetros de una función.

```js

const greeting = ({ country, name, ...person }) => country === 'spain' ? `Hola ${name}!` : `Hello ${name}!`;

```

## Objetos inmutables

El operador _spread_ es muy útil a la hora de trabajar con objetos inmutables, ya que si queremos crear un nuevo _array_ o un nuevo objeto pero cambiando solo una de sus propiedades podemos usarlo con lo visto anteriormente.

Si por ejemplo tenemos una lista de objetos de tipo persona y queremos cambiar el nombre de una de ellas y hacer un nuevo objetos para pasarlo al estado de nuestro componente en React por poner un caso de uso, podríamos aplicar algo parecido a:

```js

const people = [
    {
        id: 1,
        name: 'pepe',
        age: 45,
        country: 'spain',
    },
    {
        id: 2,
        name: 'jaime',
        age: 34,
        country: 'france',
    },
    {
        id: 3,
        name: 'laura',
        age: 37,
        country: 'spain',
    },
    {
        id: 4,
        name: 'gema',
        age: 20,
        country: 'usa',
    },
    {
        id: 5,
        name: 'donnald',
        age: 18,
        country: 'usa',
    },
];

const changePersonName(personid, newname) {
    const newArray = people.map(person => 
        person.id === personid 
        ? ({
            {
                ...person,
                name: newname,
            }
        })
        : person);
    return newArray;
};

```

### Deep copy

El ejemplo anterior es sencillo y tiene un pequeño efecto demo. Hay que tener cuidado cuando lo que queremos hacer es un _deep copy_, es decir, el copiado no solo de propiedades primitivas sino de propiedades que sean objetos. En el caso de que la propiedad que copiemos sea un objeto se estará copiando solamente la referencia:

```js
let person =  {
    id: 5,
    name: 'donnald',
    age: 18,
    country: {
        name: 'france',
        capital: 'paris',
    },
};
const newPerson = {
    ...person,
    age: 28,
}
person.country.name = 'spain';

console.log(newPerson.country);
// country: {
//    name: 'spain',
//    capital: 'paris',
// }
```

Por ello si queremos hacer un copiado en profundidad (_deep copy_) debemos seguir alguna otra estrategia, por ejemplo serializar un nuevo objeto:

```js
let person =  {
    id: 5,
    name: 'donnald',
    age: 18,
    country: {
        name: 'france',
        capital: 'paris',
    },
};
const newPerson = JSON.parse(JSON.stringify(person));
person.country.name = 'spain';

console.log(newPerson.country);
// country: {
//    name: 'france',
//    capital: 'paris',
// }
```

## Object assign

El operador _spread_ y la función ```Object.assign({}, person)``` hacen exactamente lo mismo siempre que el primer argumento de _Object.assign_ sea un objeto vacío. De hecho, _spread_ usa por debajo _Object.assign_ si el navegador lo soporta.

```js
const person =  {
    id: 5,
    name: 'donnald',
    age: 18,
    country: 'france',
};
const newPerson = {
    ...person,
}

const newPersonAssign = Object.assign({}, person);

// newPerson y newPersonAssign serian una copia exacta de person
```

El objetivo de _Object.assign_ es copiar las propiedades de un objeto origen a un objeto destino. El primer parámetro de la función es el objeto destino. Por ello, para usar inmutabilidad y ser idéntico al comportamiento de _spread_, el primer parámetro debería ser ese objeto vacío. Veamos un ejemplo sin este objeto vacío:

```js
let newPerson = {
    id: 6,
};

const person =  {
    id: 5,
    name: 'donnald',
};

const newPersonAssign = Object.assign(newPerson, person);

// newPersonAssign: {
//  id: 6,
//  name: 'donnald',
// }

// newPerson: {
//  id: 6,
//  name: 'donnald',
// }

```

_Object.assign_ tiene los mismos problemas que _spread_ con el copiado en profundidad.

Por tanto, si lo que buscamos es una solución para la inmutabilidad, lo mejor por legibilidad en el código es usar el operador _spread_ en lugar de _Object.assign_. Es verdad que en _perfonmance_ _Object.assign_ tiene un pequeño mejor resultado, pero es casi despreciable.

Para ver más detalles de _Object.assign_ podéis mirar [aquí](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Object/assign).

# Conclusiones

Debemos tener siempre presente un operador como _spread_ para poder conjugarlo en cualquier ocasión y dejar nuestro código mucho más legible. 

Usar ```...``` da [energía](https://geeks.ms/windowsplatform/2017/05/10/la-energia-del-codigo/) a nuestras líneas de Javascript.

Happy coding!
