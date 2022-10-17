---
title: Conceptos sobre WebAssembly
slug: WebAssembly/Concepts
translation_of: WebAssembly/Concepts
---
{{WebAssemblySidebar}}

En este artículo se explican conceptos de cómo funciona WebAssembly, sus objetivos, los problemas que resuelve, y como se ejecuta dentro del motor de renderizado de un navegador.

## ¿Qué es WebAssembly?

WebAssembly es un nuevo tipo de código que puede ser ejecutado en navegadores modernos, y provee nuevas funcionalidades y mejoras en rendimiento. No está pensado para ser ser escrito a mano, si no que está diseñado par ser un objeto final de compilación para lenguajes de bajo nivel como C, C++, Rust, etc.

Esto tiene enormes implicaciones para la plataforma web: proporciona una forma de ejecutar en la web código escrito en múltiples lenguages a una velocidad casi nativa, con aplicaciones cliente ejecutándose en la web como antes no podrían haberlo hecho.

Además, ni siquiera se tiene que saber cómo crear código WebAssembly para aprovecharlo. Los módulos de WebAssembly se pueden importar a una aplicación web (o Node.js), exponiendo las funciones de WebAssembly para su uso a través de JavaScript. Los frameworks de JavaScript podrían hacer uso de WebAssembly para conferir ventajas de rendimiento masivas y nuevas funciones, al mismo tiempo que hacen que la funcionalidad esté fácilmente disponible para los desarrolladores web.

## Objetivos de WebAssembly

WebAssembly ha sido creado por como un estándar abierto dentro de [W3C WebAssembly Community Group](https://www.w3.org/community/webassembly/) con los siguientes objetivos:

- Ser rápido, eficiente y portable — el código WebAssembly se puede ejecutar a una velocidad casi nativa en diferentes plataformas aprovechando las [capacidades comunes del hardware](http://webassembly.org/docs/portability/#assumptions-for-efficient-execution).
- Ser legible y depurable — WebAssembly es un lenguaje ensamblador de bajo nivel, pero tiene un formato de texto legible por humanos (la especificación aún se está terminando) lo cual permite al código ser escrito, visualizado y depurado a mano.
- Mantenerse seguro — WebAssembly se especifica para ser ejecutado de manera segura en un entorno de ejecución de espacio aislado (sandbox).Como otros códigos web, reforzará el propio origen del navegador así como sus políticas de seguridad.
- No romper la red — WebAssembly está diseñado de tal forma que se lleve bien con otras tecnologías web y mantenga compatibilidad con versiones anteriores.

> **Nota:** WebAssembly tendrá también usos fuera de la red y de los ambientes JavaScript (vea [Incrustaciones no-web](http://webassembly.org/docs/non-web/)).

## ¿Cómo se inserta WebAssembly dentro de la plataforma web?

La plataforma web puede pensarse como constituida de dos partes:

- Una máquina virtual (VM por sus siglas en inglés) que ejecuta el código de la aplicación Web, p.e. el código JavaScript, que potencia sus aplicaciones.
- Un conjunto de ([Web APIs](/es/docs/Web/API)) que la aplicación Web puede llamar para controlar la funcionalidad del navegador/dispositivo web y hace que las cosas sucedan ([DOM](/es/docs/Web/API/Document_Object_Model), [CSSOM](/es/docs/Web/API/CSS_Object_Model), [WebGL](/es/docs/Web/API/WebGL_API), [IndexedDB](/es/docs/Web/API/IndexedDB_API), [Web Audio API](/es/docs/Web/API/Web_Audio_API), etc.).

Históricamente, la máquina virtual ha sido capaz de cargar solamente JavaScript. Esto nos ha funcionado bien debido a que JavaScript es suficientemente capaz de resolver la mayor parte de los problemas que las personas tienen en la web hoy en día. Sin embargo hemos llegado a tener problemas de rendimiento cuando se trata de usar JavaScript para casos de uso más intensos como juegos 3D, Realidad Virtual y Aumentada, visión por computadora, edición de vídeo/imágenes y algunos otros dominios de cosas que demandan rendimiento similar al de código nativo (vea [Casos de Uso WebAssembly](http://webassembly.org/docs/use-cases/) para más ideas).

Adicionalmente, el costo de descargar, analizar sintácticamente (parsing) y compilar aplicaciones JavaScript muy grandes resulta prohibitivo. Plataformas en móviles (celulares y otros) y otras de recursos limitados (tabletas, etc.) pueden amplificar más estos cuellos de botella del desempeño.

WebAssembly es un lenguaje distinto a JavaScript, pero no pretende reepmlazarlo. En lugar de ello, se diseña para complementar y trabajar en conjunto con JavaScript permitiendo a los desarrolladores web obtener ventaja de las fortalezas de ambos lenguajes:

- JavaScript es un lenguaje de alto nivel, flexible y suficientemente expresivo para desarrollar aplicaciones web. Tiene muchas ventajas - es tipado dinámicamente, no necesita el paso de compilado, y tiene un gran ecosistema que lo provee de frameworks, librerías y otras herramientas.
- WebAssembly es un lenguaje de bajo nivel similar a ensamblador, con un binario de un tamaño compacto que se ejecuta con una rendimiento casi nativo, y provee a lenguajes con esquemas de memoria de bajo nivel, como C++ y Rust, con un objeto de compilación que también pueden ejecutar en la web. (Notar que WebAssembly también tiene el objetivo de soportar a lenguajes de alto nivel con recolector de basura (garbage-collector) en el futuro).

Con la llegada de WebAssembly en los navegadores, la máquina virtual que se mencionó anteriormente, cargará y ejecutará dos tipos de código - JavaScript y WebAssembly.

Los distintos tipos de código pueden llamarse uno al otro según necesiten. [WebAssembly JavaScript API](/es/docs/Web/JavaScript/Reference/Global_Objects/WebAssembly) envuelve el código WebAssembly exportado con funciones JavaScript que pueden ser llamadas normalmente, y WebAssembly puede importar y llamar síncronamente funciones JavaScript. De hecho, la unidad básica de código en WebAssembly se denomina módulo, y los módulos en WebAssembly son simétricos en muchos aspectos a los módulos de ES2015.

### Conceptos Clave en WebAssembly

Hay varios conceptos claves que son necesarios para entender cómo se ejecuta WebAssembly en un navegador. Todos estos conceptos están reflejados uno a uno en [WebAssembly JavaScript API](/es/docs/Web/JavaScript/Reference/Global_Objects/WebAssembly).

- **Módulo**: Representa un binario de WebAssembly que ha sido compilado por el navegador en un código de máquina ejecutable. Un módulo no tiene estado, y por lo tanto, como un [Blob](/es/docs/Web/API/Blob), puede ser explícitamente compartido entre ventanas y workers (por medio de [`postMessage()`](/en-US/docs/Web/API/MessagePort/postMessage)). Un módulo declara 'imports' y 'exports' igual que un módulo ES2015.
- **Memoria**: Es un ArrayBuffer redimensionable que contiene un array lineal de bytes leídos y escritos por las instrucciones de acceso a la memoria de bajo nivel de WebAssembly.
- **Tabla**: array tipado de tamaño variable de referencias (p.e. a funciones) que de otro modo no podrían ser guardadas como bytes crudos en memoria (por razones de seguridad o portabilidad).
- **Instancia**: Un módulo emparejado con el estado total en tiempo de ejecución, incluida una memoria, una tabla y un conjunto de valores importados. Una instancia es como un módulo ES que se ha cargado dentro de un "global particular" con un conjunto particular de importaciones. 

La API de JavaScript brinda a los desarrolladores la habilidad de crear módulos, memoria, tablas e instancias. Dada una instancia de WebAssembly, el código JavaScript puede llamar sincrónicamente a sus exportaciones, que se exponen como funciones JavaScript normales. El código de WebAssembly también puede llamar sincrónicamente a funciones de JavaScript arbitrarias al pasar esas funciones de JavaScript como importaciones a una instancia de WebAssembly.

Dado que JavaScript tiene un control completo sobre cómo el código de WebAssembly es descargado, compilado y ejecutado, los desarrolladores de JavaScript pueden pensar en WebAssembly como simplemente una funcionalidad de JavaScript para generar funciones de alto rendimiento.

En el futuro, los módulos de WebAssembly se podrán cargar igual que los módulos de [ES2015](https://github.com/WebAssembly/proposals/issues/12) (usando `<script type='module'>`), implicando que JavaScript será capaz de ir a buscar, compilar e importar un módulo de WebAssembly tan fácilmente como un módulo de ES2015.

## ¿Cómo usar WebAssembly en mi aplicación?

Previamente se describieron las primitivas que WebAssembly añade a la plataforma Web: un formato binario para el código, y APIs para cargar y ejecutar este código binario. Ahora se describirá cómo usar estas primitivas en la práctica.

El ecosistema de WebAssembly está en sus comienzos; sin duda más herramientas aparecerán en un futuro. Ahora mismo hay cuatro puntos principales donde comenzar:

- Migrar una aplicación C/C++ con [Emscripten](/es/docs/Mozilla/Projects/Emscripten).
- Escribir o generar WebAssembly directamente a nivel de ensamblador.
- Escribir una aplicación en Rust y generar su salida como WebAssembly.
- Usar [AssemblyScript](https://assemblyscript.org/) que se parece a TypeScript y se compila a un binario de WebAssembly.

Detallemos más cada una de estas opciones:

### Migrando desde C/C++

Dos de las muchas opciones para crear código WASM son un ensamblador WASM en línea o [Emscripten](/es/docs/Mozilla/Projects/Emscripten). Hay varias opciones para un ensamblador WASM en línea, como pueden ser:

- [WasmFiddle](https://wasdk.github.io/WasmFiddle/)
- [WasmFiddle++](https://anonyco.github.io/WasmFiddlePlusPlus/)
- [WasmExplorer](https://mbebenita.github.io/WasmExplorer/)

Estos son excelentes recursos para las personas que están tratando de averiguar por dónde empezar, pero carecen de algunas de las herramientas y optimizaciones de Emscripten.

La herramienta Emscripten puede tomar prácticamente cualquier código fuente de C/C++ y compilarlo en un módulo .wasm, además del código "pegamento" de JavaScript necesario para cargar y ejecutar el módulo, y un documento HTML para mostrar los resultados del código.

![](emscripten-diagram.png)

Resumiendo, el proceso es el que sigue:

1. Emscripten primero introduce C/C++ en clang+LLVM, una madura cadena de herramientas de compilación de C/C++ de código abierto, incluida, por ejemplo, como parte de XCode en OSX.
2. Emscripten transforma el resultado de la compilación de clang+LLVM en un binario .wasm.
3. Por sí mismo, WebAssembly actualmente no puede acceder directamente al DOM; solo puede llamar a JavaScript, pasando tipos de datos primitivos enteros y de punto flotante. Por lo tanto, para acceder a cualquier API web, WebAssembly necesita llamar a JavaScript, que luego realiza la llamada a la API web. Entonces, Emscripten crea el código de enlace HTML y JavaScript necesario para lograrlo.
> 
> **Nota:** Hay planes para que en el futuro [WebAssembly llame a las APIs Web directamente](https://github.com/WebAssembly/gc/blob/master/README.md).

El código de "pegamento" (o de enlace) de JavaScript no es tan simple como podría imaginar. Para empezar, Emscripten implementa bibliotecas C/C++ populares como [SDL](https://en.wikipedia.org/wiki/Simple_DirectMedia_Layer), [OpenGL](https://en.wikipedia.org/wiki/OpenGL), [OpenAL](https://en.wikipedia.org/wiki/OpenAL) y partes de [POSIX](https://en.wikipedia.org/wiki/POSIX). Estas bibliotecas se implementan en términos de APIs Web y, por lo tanto, cada una requiere un código de enlace de JavaScript para conectar WebAssembly a la API web subyacente.

Entonces, parte del código de enlace implementa la funcionalidad de cada biblioteca respectiva utilizada por el código C/C++. El código de enlace también contiene la lógica para llamar a las APIs de JavaScript de WebAssembly mencionadas anteriormente para llamar, cargar y ejecutar el archivo .wasm.

El documento HTML generado carga el archivo de enlace de JavaScript y escribe la salida estándar en un {{htmlelement("textarea")}}. Si la aplicación usa OpenGL, el HTML también contiene un elemento {{htmlelement("canvas")}} que se usa como destino de renderizado. Es muy fácil modificar la salida de Emscripten y convertirla en cualquier aplicación web que necesite.

Puede encontrar documentación completa sobre Emscripten en [emscripten.org](https://emscripten.org), y una guía para implementar la cadena de herramientas y compilar su propia aplicación C/C++ a través de wasm en [Compilación de C/C++ a WebAssembly ](/es/docs/WebAssembly/C_to_wasm).

### Excribiendo directamente en WebAssembly

¿Quiere crear su propio compilador, sus propias herramientas o crear una biblioteca de JavaScript que genere WebAssembly en tiempo de ejecución?

De la misma manera que los lenguajes ensambladores físicos, el formato binario WebAssembly tiene una representación de texto (ensamblador): los dos tienen una correspondencia 1:1. Puede escribir o generar este formato a mano y luego convertirlo al formato binario con cualquiera de las varias [herramientas de conversión de texto a binario de WebAssemby] (https://webassembly.org/getting-started/advanced-tools/).

Para obtener una guía sencilla sobre cómo hacer esto, consulte nuestro artículo [Conversión de formato de texto WebAssembly a wasm](/es/docs/WebAssembly/Text_format_to_wasm).

### Escribiendo en Rust - Compilando a WebAssembly

También es posible escribir código en Rust y compilarlo a WebAssembly, gracias al trabajo incansable del grupo de trabajo de Rust WebAssembly. Puede comenzar instalando la cadena de herramientas necesaria, compilando un programa Rust de muestra en un paquete npm de WebAssembly y usándolo en una aplicación web de ejemplo, como muestra nuestro artículo [Compilación de Rust a WebAssembly] (/es/docs/WebAssembly/Rust_to_wasm).

### Usando AssemblyScript

Para los desarrolladores web que quieran probar WebAssembly sin necesidad de conocer los detalles de C o Rust, AssemblyScript será la mejor opción. Genera un paquete pequeño y su rendimiento es ligeramente más lento en comparación con C o Rust. Puede consultar su documentación en <https://assemblyscript.org/>.

## Resumen

Este artículo le ha brindado una explicación de qué es WebAssembly, por qué es tan útil, cómo se adapta a la web y cómo puede utilizarlo.

## Vea también

- [Artículos de WebAssembly en el blog Trucos Mozilla](https://hacks.mozilla.org/category/webassembly/)
- [WebAssembly en Mozilla Research](https://research.mozilla.org/webassembly/)
- [Cargando y ejecutando código WebAssembly](/es/docs/WebAssembly/Loading_and_running) — descubra cómo cargar su propio módulo WebAssembly en una página web.
- [Uso de la API JavaScript de WebAssembly](/es/docs/WebAssembly/Using_the_JavaScript_API) — descubra cómo usar otras características principales de la API JavaScript de WebAssembly.
