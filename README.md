
Iniciar un proyecto desde cero con NodeJs
Crea un directorio nuevo en tu ordenador con el nombre de tu proyecto.

Navega con tu terminal hasta él y escribe el comando NPM para inicializar un nuevo «package». 

npm init
Puedes aceptar todas las opciones predefinidas. Al final del proceso, deberás tener un archivo llamado package.json parecido a este.

package.json
{
  "name": "nodejsproject",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
Habilitar el uso de módulos ES
El sistema de importación y exportación de módulos que viene por defecto en NodeJs es “CommonJs”.  Así que ese será uno de los primeros puntos que vamos a modificar.

Abre «package.json» con tu editor de código, e incluye la propiedad «type» con el valor «module», tal y como aparece en este ejemplo.

package.json
{
  "name": "nodejsproject",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "type": "module",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
Incluyendo ésta propiedad, podremos hacer uso del sistema de manejo de dependencias con ESM.

De este modo podremos utilizar la sintaxis basada en import / export en vez de require.

Variables de entorno
Tarde o temprano, nuestro proyecto va a hacer uso de datos sensibles como credenciales de acceso, o parámetros de configuración.

Es muy mala idea almacenar esa información como parte del código fuente, de modo que a continuación, vamos a añadir un par de archivos con variables de entorno.

Genera dos nuevos documentos en la raíz del directorio, con los nombres «.env.local» y “.env”.

A modo de prueba incluiremos las siguientes variables en cada uno:

.env.local
FOO = BAR
.env
FOO = END
Si has iniciado un repositorio local, comprueba que el archivo «.gitignore» excluya estos ficheros, así como el directorio «node_modules». 

.gitignore
build
.env.local
.env
node_modules/
La mejor forma de que nuestro script reconozca e importe las variables de entorno de archivos «.env», es mediante la librería «dotenv». 

Así que vamos a proceder a instalarla con NPM. 

npm i dotenv
De momento no te preocupes por su uso, llegaremos a eso en el momento de definir los scripts de package.json y el proceso a ejecutar con Nodemon.

Habilitar TypeScript en NodeJs
Programar con TypeScript puede ayudarte a evitar «bugs» y errores potenciales en tu código, por eso te enseñaré a habilitar su uso en el entorno que estamos preparando. 

En el caso de que no seas partidario de usar TypeScript, te recomiendo NO saltarte este punto. Podrás seguir utilizando Vanilla JavaScript si lo deseas, pero tendrás tu entorno igualmente preparado, por si en un futuro quieres migrar tu código a TypeScript.

Instala las siguientes librerías con el comando que sigue:

npm i typescript ts-node @types/node
typescript: La librería principal que se encargará de transpilar el código de TS a JS.
ts-node: Ésta herramienta se encargará de interpretar y ejecutar el código TS directamente en NodeJs durante la fase de desarrollo.
@types/node: Aprovechamos para instalar definiciones y tipos propios de NodeJs para que TypeScript los reconozca.
Acto seguido añadimos el archivo de configuración en la raíz del proyecto.

tsconfig.json
{
  "exclude": ["./build", "jest.config.ts", "./test"],
  "ts-node": { "files": true },
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["es6"],
    "module": "NodeNext",
    "rootDir": "src",
    "resolveJsonModule": true,
    "allowJs": true,
    "outDir": "build",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "noImplicitAny": true,
    "skipLibCheck": true
  }
}
Algunas de las propiedades del archivo que merecen una mención son las siguientes:

«ts-node»: { «files»: true }, Permite que “ts-node” reconozca tus archivos .d.ts
«target»: «ES2020», Definimos “ES2020” como versión del lenguaje de salida.
«module»: «NodeNext», Especificamos con qué versión de módulos vamos a trabajar nuestro código. La propiedad más reciente es “NodeNext”, aunque como veremos más adelante genera alguna incongruencia en el código que deberemos tener en cuenta.
Te animo a que dediques unos minutos a leer el resto de configuraciones.

Reinicio automático tras realizar cambios con Nodemon.
Nodemon es una herramienta imprescindible durante la fase de desarrollo de cualquier proyecto backend.

Gracias a esta librería, podremos configurar un sistema de reinicio automático del proceso, cada vez que editemos un archivo del proyecto.

Esto nos ahorrará muchas horas de parar e iniciar el proceso manualmente.

Estoy seguro de que esto te resultará familiar, si has trabajado en proyectos frontend con sistemas de “livereload” que refresca el navegador en cuanto detecta un cambio.

Instala Nodemon con el siguiente comando:

npm i nodemon
Seguidamente preparamos un archivo de configuración llamado “nodemon.json”, en él detallaremos qué comportamiento deseamos que se aplique:

nodemon.json
{
  "watch": ["src"],
  "ext": ".ts,.js",
  "ignore": ["node_modules/*"],
  "exec": "node --trace-warnings --inspect --loader ts-node/esm -r dotenv/config ./src/index.ts dotenv_config_path=.env.local dotenv_config_debug=true"
}
Las cuatro propiedades que hemos definido en este archivo representan las siguientes opciones:

“watch”: Indicamos qué directorios desencadenarán el reinicio automático.
“ext”: Dentro de dicho directorio, solo atenderemos a cambios aplicados sobre archivos con las extensiones definidas en “ext”.
“ignore”: En la array de “ignore” especificamos estrictamente qué directorios deben ser ignorados por Nodemon.
“exec”: Este parámetro acepta una instrucción como si de un script se tratase. Se ejecutará al inicio del proceso, cada vez que Nodemon detecte un cambio.
Es importante analizar en detalle la instrucción definida en “exec”, ya que es allí donde se conecta Nodemon con el resto de dependencias que hemos estado preparando.

En el listado que sigue, te voy a indicar qué es cada parte del comando “exec”

node –trace-warnings –inspect –loader ts-node/esm -r dotenv/config ./src/index.ts dotenv_config_path=.env.local dotenv_config_debug=true

node: Nodemon lanza el comando node tras detectar un reinicio, ya que a fin de cuentas, el entorno de desarrollo está basado en NodeJs.
–trace-warnings: Con este argumento, indicamos que queremos listar en la consola todas las capas de software implicadas cuando se da un error. En mi opinión, más información siempre es mejor.
–inspect: Instrucción para habilitar la depuración de código en el navegador. Más adelante veremos cómo hacer uso de esta característica.
–loader ts-node/esm: Para establecer una compatibilidad entre “ts-node” y el sistema de módulos definido en “tsconfig.json” es necesario cargar un “loader” específico. Cabe destacar que éste se encuentra en fase experimental, de modo que puede mostrar alguna advertencia en la terminal al iniciar el proceso.
-r dotenv/config ./src/index.ts dotenv_config_path=.env.local dotenv_config_debug=true: Con esta porción, pedimos a node que cargue el programa que se encuentra en “./src/index.ts”, incluyendo en él, las variables de entorno del archivo “.env.local”.
Añadir scripts en package.json
Ahora que tenemos gran parte de los elementos conectados entre sí, ha llegado el momento de ponerlos en marcha a través de scripts personalizados.

Las tres instrucciones que vamos a introducir servirán para: iniciar el programa en modo desarrollo, compilar el programa y ejecutar el programa en el entorno de producción.

El resultado del package.json será el siguiente:

package.json
{
  "name": "nodejsproject",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "type": "module",
  "scripts": {
    "start:dev": "npx nodemon",
    "build": "rimraf ./build && tsc",
    "start": "npm run build && node -r dotenv/config ./build/index.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
Crearemos primero “start:dev” y asignaremos el comando “npx nodemon”. 

Todos los parámetros asociados a esta instrucción, se encuentran definidos en el archivo “nodemon.json” tal y como ya hemos visto en el paso anterior.

La segunda instrucción la llamaremos “build”. Para que funcione es necesario instalar la dependencia “rimraf”. Así que deberías instalarla con NPM:

npm i rimraf
Con el comando “npm run build” nos aseguraremos de borrar el directorio “build” y crear uno nuevo con el programa totalmente compilado.

Por último, el comando “npm run start” está pensado para ser llamado desde el servidor de producción. 

Realizará una compilación y seguidamente ejecutará el programa del directorio “build” pasándole las variables de entorno del archivo “.env”.

Llegados a este punto, ya podemos probar el entorno, pero para ello hace falta un programa.

Por supuesto para el caso que nos ocupa, no hace falta nada muy complejo, un par de archivos con unas funciones y un “console.log” serán suficientes. Ya tendremos tiempo de crear programas más interesantes más adelante.

Crea un archivo dentro del directorio “src” y llámalo “helloFoo.ts”, dentro declara y exporta una función con el mismo nombre, que construya un string a partir de la variable de entorno.

src/helloFoo.ts
export const helloFoo: () => string = () => {
  return `Hello ${process.env.FOO}`;
};
En “src/index.ts” importa la función, ejecutala y muestra el resultado en la consola. Tal como aparece en el ejemplo que sigue.

src/index.ts
import { helloFoo } from "./helloFoo.js";
const fooSalute = helloFoo();
debugger;
console.log(fooSalute);
Fíjate que he añadido una instrucción “debugger”, en el siguiente punto trataremos de detener el programa en ese punto, para inspeccionar el código en tiempo de ejecución.

Con este pequeño programa listo, ejecuta el comando de compilación del programa:

npm run build
Deberías ver un directorio “build” con el código totalmente convertido a JavaScript.

Por otra parte, si ejecutas el comando “start”, como si del entorno de producción se tratara, además de la compilación, deberías ver en la terminal el mensaje “Hello END”.

npm run start
> nodeproject@1.0.0 start
> npm run build && node -r dotenv/config ./build/index.js

> nodeproject@1.0.0 build
> rimraf ./build && tsc
Hello END
Eso es porque esa instrucción lee el archivo “.env” en vez de “.env.local”.

Inspeccionar y depurar el código, en las herramientas de desarrollador del navegador.
Genial, lo tenemos todo listo para enviar el código a un servidor de producción. Pero donde realmente brilla esta base que hemos construido es durante la fase de programación.

Vamos a probarla.

Antes de ejecutar el comando “npm run start:dev” abre una consola de herramientas de desarrollador, en cualquier pestaña de tu navegador de Chrome.

Después lanza el comando:

npm run start:dev
En la consola de la terminal deberías ver el mensaje “Hello BAR”, ya que lee la variable FOO del archivo “.env.local”

> nodeproject@1.0.0 start:dev
> npx nodemon
[nodemon] 2.0.20
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): src/**/*
[nodemon] watching extensions: ts,js
[nodemon] starting `node --trace-warnings --inspect --loader ts-node/esm -r dotenv/config ./src/index.ts dotenv_config_path=.env.local dotenv_config_debug=true`
Debugger listening on ws://127.0.0.1:9229/dffd205d-8d5f-4007-971e-99849b1dfebb
Hello BAR
[nodemon] clean exit - waiting for changes before restart
Puede que te aparezca el error “ExperimentalWarning: Custom ESM Loaders is an experimental feature. This feature could change at any time”. Es normal, ya que como dije de momento ts-node usa una función experimental para el tipo de módulos configurados.

Por otro lado, en las herramientas de desarrollador de Chrome, deberías ver un nuevo icono en la parte superior izquierda.

Icono para depurar código NodeJs en el navegador Chrome
Icono para depurar código NodeJs en el navegador Chrome
Si haces clic en él, se te abrirá una ventana nueva que te va a permitir depurar el código Nodejs.

Solo hace falta que lances de nuevo el comando “npm run start:dev”, deberías ver como se detiene el programa justo en el punto donde pusiste el “debugger”.

Depurar el código en la consola con "debugger"
Depurar el código en la consola con «debugger»
Allí podrás inspeccionar las variables en tiempo de ejecución como si de una aplicación frontend se tratara.

También puedes editar el código y verás como automáticamente Nodemon reinicia el proceso por tí. (Asegúrate de haber “descongelado” el programa en el punto de debugger)

Incluir tests de unidad con Jest
Concluimos el tutorial añadiendo la funcionalidad de tests de unidades con Jest. No voy a detenerme a explicar cómo usar ésta librería ya que eso daría para un artículo dedicado.

Pero sí que vamos a ver cómo se encaja en todo este puzzle.

En primer lugar, asegúrate de excluir el directorio “./test” y el archivo “.jest.config.ts” en el archivo “tsconfig.json”.

A continuación, instala las siguientes dependencias:

npm i jest ts-jest @types/jest
jest es la librería para realizar tests propiamente.
ts-jest permite el uso de TypeScript para dichos tests.
@types/jest importa los tipos de la librería Jest para TypeScript.
Actualiza el comando “test” en el archivo “package.json”

package.json
{
  "name": "nodejsproject",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "type": "module",
  "scripts": {
    "start:dev": "npx nodemon",
    "build": "rimraf ./build && tsc",
    "start": "npm run build && node -r dotenv/config ./build/index.js",
    "test": "jest"
  },
  "author": "",
  "license": "ISC"
}
Crea un archivo de configuración “jest.config.ts” en la raíz. En él definiremos el preset con el que cargar los tests, el tipo de entorno y el archivo de “setup”.

jest.config.ts
/** @type {import('ts-jest').JestConfigWithTsJest} */
module.exports = {
  preset: "ts-jest",
  testEnvironment: "node",
  setupFiles: ["./test/setup-tests.ts"],
};
Para Jest que pueda leer las variables de entorno de “.env.local” en los tests, creamos un archivo llamado “setup-tests.ts” dentro del directorio “test”.

En su interior añadimos las siguientes líneas:

test/setup-tests.ts
import dotenv from "dotenv";
dotenv.config({ path: "./.env.local" });
Finalmente podemos probar el sistema de testing incluyendo un archivo comprobará la función “helloFoo”.

Describimos el test generando un documento nuevo dentro de “tests” llamado “helloFoo.test.ts” e incluímos la siguiente definición.

test/helloFoo.test.ts
import { helloFoo } from "../src/helloFoo";
test("should return Hello BAR", () => {
  expect(helloFoo()).toBe("Hello BAR");
});
Tras guardar, puedes ejecutar el comando “npm run test”, si todo ha ido bien, debería pasar el test correctamente.

npm run test
Test pasados con Jest
Test pasados con Jest