# Manejos de archivos y asincronismo en NodeJS

## Asincronismo

### Programación asincronica en Node.js

Uno de los primeros conceptos que debemos incorporar cuando empezamos a programar en Node.js es la idea de asincronismo. En Node.js todas las operaciones de entrada/salida (IO) se ejecutan de manera asincrónica y no bloquente. Esto tiene muchas ventajas en cuanto a la optimización de los recursos (permite majenar muchas operaciones IO en un mismo thread de forma eficiente), pero al mismo tiempo nos obliga a adoptar algunos patrones de programación diferentes a otras plataformas mas tradicionales que usan IO bloqueante (como Java o PHP).

[Event Loop](https://en.wikipedia.org/wiki/Event_loop)
[Asynchronous I/O](https://en.wikipedia.org/wiki/Asynchronous_I/O)

#### [Continuation Passing Style](https://en.wikipedia.org/wiki/Continuation-passing_style)

En el estilo de programacion tradicional, el flujo de datos de un programa es directo. Cada funcion al ser invocada recibe ciertos parametros y realiza cierto procesamiento. El resultado del mismo se comunica al flow principal mediante el valor de retorno. Por ejemplo:

```
// Declaracion de la función
function add(a, b) {
  return a + b;
}

// Invocación
const result = add(1, 2);

// Uso del resultado
console.log(result);
```

Si se utiliza este estilo de programación para operaciones IO, la función deberia "esperar" a que la operacion IO termine antes de retornar, bloqueando la ejecucion del programa hasta este punto. Como vimos, node no trabaja de este modo, por lo que el estilo directo no sirve para expresar operaciones IO y necesitamos de otro tipo de semanticas. Continuation Passing Style (CPS) es el patron de ejecucion que se ultiliza en todo el ecosistema de Node.js para manejar el flujo de datos en operaciones asincronicas.

En CPS, cada función recibe un parámetro adicional, que representa la Continuación de la función. En lugar de retornar, la función invocará la continuación recibida pasando el valor de retorno.

```
// Declaracion de la función
function add(a, b, callback) {
  return a + b;
}

// Invocación
add(1, 2, function (result) {
    // Uso del resultado
    console.log(result);
});
```

El CPS puede ser asincroónico o sincrónico. En el ejemplo anterior, al no realizarse nunguna opercion de IO, el callback es llamado inmediatamente, por lo tanto su ejecución es secuencial al resto del código. Sin embargo lo mas comun es usar este estilo solo para la API de una funcion que realiza algun tipo de operacion IO. Podemos simpular una operacion IO, usando la funcion setTimeout y retrasando la ejecucion del callback por 3000 milisegundos.

```
function addCPSAsync(x, y, callback) {
  setTimeout(
    function() {
      callback(x + y);
    },
    3000
  );
}
```

## Promises

Utilizar callbacks nos puede llevar a una situación donde la anidación de los mismos puede empezar a ser dificil de entender. Este antipatrón es conocido como [callback hell](https://en.wiktionary.org/wiki/callback_hell) y una de las formas de resolverlo es a través del uso de promises.

[docs](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise)

### Definición

```
The Promise object is used for asynchronous computations. A Promise represents a value which may be available now, or in the future, or never.
```

Como podemos ver en la definición, una promesa es utilizada para realizar operaciones asincrónicas. La misma representa un valor que podrá estar disponible ahora, en el futuro o nunca. Es por este motivo que una promesa nos debe ofrecer una forma de iniciar una operación asincrónica la cual una vez finalizada pueda devolver el valor de la operación realizada.

La forma ofrecida por una Promise se denomina __executor__

### Sintaxis

```javascript
new Promise( /* executor */ function(resolve, reject) { ... } );
```

### Executor

Un executor es una función que recibe dos parámetros resolve y reject. La función executor es ejecutada inmediatamente por la implementación de una Promesa (el *executor* hasta es ejecutado antes que el cosntructor de la promesa retorne el objeto creado).

Las funciones __resolve__ y __reject__, resuelven y rechazan la promesa respectivamente. Y si se lanza un error dentro del executor la promesa también es rechazada.

### Estados de una promesa

1. Pendiente: es el estado inicial, no finalizó ni se rechazó.
2. Finalizada: se completó la operación sin errores.
3. Rechazada: la operación falló.

### Promise.prototype.then()

Anteriormente mencionamos que un *executor* devolverá el valor de su ejecución una vez finalizado o devolverá el error en caso de haber fallado. La forma de obtener el valor o el error de una promesa es a través de .then()

El método __then( )__ devuelve una promesa y recibe dos funciones, la primera recibirá el valor esperado y la segunda nos devolverá el error en caso de que haya fallado.

```javascript
p.then( value => {
  // La promesa devolvió un valor
}, error => {
  // Hubo un error
});
```

### Promise.prototype.catch()

El método __catch()__ al igual que __then()__ retorna una promesa pero a diferencia del anterior sólo maneja los casos de error.

Su funcionamiento es el mismo que llamar a __.then(undefined, onRejected)__

```javascript
p.catch( error => {
  // Hubo un error
});
```

#### Ejemplo .then() y .catch()
```javascript

// funcion que simula una obtención asincrónica de usuarios 
const getUsers = () => {
  return new Promise( (resolve, reject) => {
    asyncmethod( (err, result) => {
      if(err) return reject(err);
      return resolve(result);
    });
  });
};

// Ejemplo
getUsers()
 .then( users => {
    console.log(users);
 })
 .catch( error => {
    console.error(error);
 });
```

### Promise.prototype.all()

El método __all()__ retorna una única promesa que finalizará una vez que finalicen todas las promesas del argumento recibido, or fallará cuando una de las promesas recibidas fallé.

```javascript
const p1 = Promise.resolve(3);
const p2 = 1337;
const p3 = new Promise((resolve, reject) => {
  resolve(10);
}); 

Promise.all([p1, p2, p3]).then(values => { 
  console.log(values); // [3, 1337, 10] 
});
```

### Encadenamiento de promises

Como se mencionó anteriormente __.then()__ y __.catch()__ devuelven por default promesas, esto hace posible que las mismas sean encadenadas.

```javascript
p
  .then( value => {
    // La promesa devolvió un valor
    // quiero seguir pasando el valor a las siguientes promesas
    return value;
  })
  .then( value => {
    // Valor devuelto por 1° then()
    // then() por default devuelve una promesa
  })
  .then( value => {
    // value es undefined
  })
  .catch( error => {
    // Hubo un error en cualquier parte de la cadena
  });
```

## Manejos de archivos en NodeJS

La API de NodeJS nos ofrece métodos para acceso a archivos. 

[Documentación](https://nodejs.org/api/fs.html)

## Métodos que utilizaremos en clase

### fs.writeFile

```
fs.writeFile(file, data[, options], callback)
	file <string> | <Buffer> | <integer> filename or file descriptor
	data <string> | <Buffer> | <Uint8Array>
	options <Object> | <string>
		encoding <string> | <null> default = 'utf8'
		mode <integer> default = 0o666
		flag <string> default = 'w'
	callback <Function>
```

Como vemos en la documentación, el método recibe el path del archivo, el string a esscribir, un objeto de opciones y un callback.

Como el objecto de opciones es opcional sólo le pasamos file, data, y callback;

#### Ejemplo

```javascript
// Escribir en el archivo 'hello.txt'
fs.readFile('hello.txt', 'Hello World!!!', err => {
  if (err) {
    return console.log('No se pudo abrir el archivo hello.txt');
  }
	console.log('Se guardo el archivo Hello World!!!');
});
```

### fs.readFile

```
fs.readFile(file[, options], callback)
	file <string> | <Buffer> | <integer> filename or file descriptor
	options <Object> | <string>
		encoding <string> | <null> default = null
		flag <string> default = 'r'
	callback <Function>
```
Como vemos en la documentación, el método recibe el path del archivo, un objeto de opciones y un callback.

Como el objecto de opciones es opcional sólo le pasamos file, data, y callback;

#### Ejemplo

```javascript
// Leer el archivo de people.json
fs.readFile('../data/people.json', 'utf-8', (err, data) => {
  if (err) {
    return console.log('No se pudo leer el archivo ../data/people.json');
  }
});
```

# [Ejemplos y ejercicios](https://github.com/utn-fullstack/clase1)

```bash
git clone https://github.com/utn-fullstack/clase1.git
```