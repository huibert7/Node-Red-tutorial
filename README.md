# Desarrollo de una aplicación web usando Node-RED
## Introducción
### ¿Qué es Node-RED?
Node-RED es un proyecto de código abierto y libre, desarrollado por el laboratorio de tecnologías emergentes de IBM y licenciado bajo Apache 2.0. Se trata de una herramienta de desarrollo diseñada para conectar sensores, APIs y servicios de forma sencilla, que permite crear nuevas soluciones innovadoras de forma rápida y con un mínimo de programación. Node- RED fue creado específicamente para simplificar el desarrollo de aplicaciones que tienen que gestionar altos volúmenes de eventos, una de las características que mejor define la era del Internet de las Cosas (Internet of Things o IoT), en la que además tendremos que lidiar con grandes volúmenes de información generados por todo tipo de sensores integrados en un sin número de dispositivos de todo tipo (medidores, dispositivos móviles y wearables como relojes y pulseras inteligentes).
### Node.js
Node.js es una plataforma de desarrollo multi-plataforma moderna construida sobre el motor de JavaScript que utiliza el navegador Chrome (conocido como V8). Lo que distingue a Node.js de otras soluciones de middleware usadas para crear aplicaciones web como los servidores de aplicaciones Java, es que el modelo usado para crear aplicaciones se basa en el manejo de eventos y que tiene unos APIs que intentan evitar a toda costa realizar operaciones que bloqueen el procesador, en particular las operaciones de I/O (Entrada/Salida). Esto significa que muchas de esas operaciones se realizan de forma asíncrona, lo que permite a la solución escalar de forma particularmente eficiente. Otro atractivo de Node.js es que se programa en JavaScript, lo que permite usar el mismo lenguaje de programación tanto para la capa de presentación (la página web) como para la lógica de negocios. Por eso, Node.js es una de las plataformas de creación de aplicaciones web y móviles que más ha crecido en los últimos años.

Node-RED corre sobre Node.js, por lo que es muy fácil integrar ambas tecnologías en una misma aplicación.

### IBM Cloud
IBM Cloud es la plataforma PaaS (Platform as a Service) de IBM. Con IBM Cloud es posible crear todo tipo de aplicaciones (web, móviles o de big data) sin necesidad de tener que invertir de antemano en infraestructura, ya sea de hardware o de software. Con los múltiples servicios que esta plataforma ofrece a los desarrolladores (bases de datos, colas de mensajes, notificaciones, etc.), se pueden crear todo tipo de aplicaciones, desde las más sencillas hasta las más complejas.

## Nuestro proyecto
El objetivo de este ejercicio es mostrar cómo crear una aplicación en IBM Cloud y cómo usar la plataforma. Para ello crearemos una aplicación web que comunique de forma síncrona con el servidor, intercambiando mensajes en tiempo real. Nuestro desarrollo incluirá una página web en la que tendremos un área de texto en la que podremos escribir un mensaje y un botón, que al pulsarlo invocará un servicio en el servidor, el cual traducirá el mensaje al idioma inglés y desplegará el texto en la página, sin necesidad de volver a cargar toda la página.

### Un poco de historia
En este ejercicio mostraremos cómo una aplicación web moderna puede interactuar con el servidor. Esto lo haremos utilizando una tecnología relativamente nueva, los WebSockets, que soportan todos los navegadores modernos. Conocer el funcionamiento de este protocolo de comunicaciones full-duplex es importante porque está cambiando la manera en la que se comunican las páginas web con el servidor. Esto permite crear páginas web mucho más dinámicas y cuyo funcionamiento se va a parecer mucho más al de las aplicaciones nativas.

Para entender el beneficio que aportan los WebSockets es necesario recordar que hasta hace poco, crear aplicaciones web que se pudieran considerar realmente dinámicas era virtualmente imposible. La World Wide Web había sido diseñada originalmente para consultar información estática almacenada en archivos HTML. Navegar de una página a otra solo se podía hacer siguiendo ligas. Posteriormente, los desarrolladores empezaron a crear sitios web un poco más interactivos, mediante el uso de formas. La información proporcionada por el usuario se mandaba al servidor (a un CGI o a un servlet) y el servidor contestaba generando una nueva página dinámica. Eso sirvió para crear todo tipo de aplicaciones básicas, pero los desarrolladores no estaban satisfechos, porque el sistema seguía siendo muy ineficiente. El hecho de que cada vez que se tuviera que interactuar con el servidor hubiera que cargar de nuevo toda la página hacía que los usuarios percibieran que las aplicaciones web eran mucho más lentas que sus equivalentes nativas y bastante menos atractivas. Para resolver el problema, Microsoft ideó una solución. La empresa creó una nueva función JavaScript, XMLHttpRequest, la cual añadió a su navegador Internet Explorer. Esta función, aún muy utilizada actualmente, permite invocar un URL (pasando parámetros de forma opcional) y obtener un resultado del servidor, el cual se puede utilizar para actualizar la página (usando el DOM o Document Object-Model), sin necesidad de volver a cargarla. Mediante el uso de XMLHttpRequest, Microsoft pudo crear un cliente web (Outlook Web Access) para su sistema de correo que funcionaba de forma muy similar a Outlook sobre Windows. El éxito fue inmediato y los demás browsers no tardaron en adoptar esa función, que se volvió rápidamente estándar.

Esta innovación creó toda una nueva generación de aplicaciones web que se conoció como AJAX (Asynchronous JavaScript and XML, un nombre pegadizo pero engañoso, porque en realidad no se necesita usar XML al invocar la función XMLHttpRequest). Las aplicaciones AJAX permitieron que las aplicaciones web se parecieran cada vez más a las aplicaciones nativas y lograron que cada vez pasemos más tiempo dentro del browser y que usemos menos aplicaciones nativas. De hecho, gracias a este fenómeno es cada vez más viable usar dispositivos que solo pueden ejecutar aplicaciones web, como lo demuestra el éxito de las Chromebooks de Google y la existencia de los teléfonos celulares basados en Firefox OS que solo pueden ejecutar aplicaciones basadas en tecnologías web.

Sin embargo, a pesar de todas sus virtudes, la función XMLHttpRequest tiene una serie de limitantes importantes. La principal es que la interacción con el servidor debe iniciarse desde el cliente, ya sea por una interacción desde el interfaz gráfico (por ejemplo apretando un botón) o a través de un temporizador. Esto significa que si queremos actualizar las páginas conectadas a nuestro servidor como respuesta a un evento enviado por el servidor, simplemente no es posible hacerlo usando XMLHttpRequest. Por eso, cuando seguimos un evento deportivo por Internet, tenemos que estar recargando la página una y otra vez, para ver si ha cambiado el marcador. Ese sistema de estar poleando el servidor una y otra vez es sumamente ineficiente y la carga de peticiones que genera obliga a invertir mucho en infraestructura, un gasto que se podría evitar. Por eso era necesario buscar otra solución mucho más elegante.

## WebSockets
Se trata de un concepto muy similar al de los sockets TCP/IP y que permite que una aplicación web establezca un canal de comunicación bi-direccional persistente entre la capa de presentación HTML en el browser y el servidor. Los WebSockets utilizan los puertos estándar de http y https (80 y 443 respectivamente) y la especificación fue diseñada para evitar problemas causados por el uso de proxies y de firewalls. Es importante recalcar que aunque los WebSockets usan el puerto 80 al igual que HTTP, usan un protocolo de comunicación distinto, razón por la cual se usa ws:// en la URL en lugar de http://. También es posible encriptar la comunicación usando el protocolo wss://, el cual usará el puerto 443 (el mismo que HTTPS).
En el cliente usaremos JavaScript para interactuar con los WebSockets en la página web y del lado del servidor se puede usar una variedad de lenguajes de programación, entre los cuales están JavaScript (usado tras bambalinas por Node-Red, la tecnología que usaremos en este ejercicio) o Java (a partir de JEE 7).

### Posibles problemas
Sin embargo, a pesar de las precauciones que tomaron los desarrolladores del estándar, es posible encontrarnos con problemas porque los administradores de los firewalls y/o de los sistemas de detección o prevención de intrusos (IDS/IPS) pueden haber decidido bloquear el uso de websockets. ¿Porqué iban a querer hacer eso? Es un tema de seguridad. Si bien los websockets no representan una amenaza especial, son una forma adicional de poder comunicarse con el exterior. En una época en la que la mayoría de las amenazas provienen de lo que se conoce como Advanced Persistance Threats (APT), programas que se ejecutan dentro de la red de las víctimas y que intentan recabar el máximo de información para enviarla de forma discreta al exterior donde los hackers la están esperando, un canal más de comunicaciones que se debe monitorear siempre es visto con recelo por parte de los responsables de seguridad.

Por ese motivo, considerando que es probable que algunos usuarios no podrán usar websockets para comunicarse con el servidor, es recomendable detectar una posible falla de conexión para al menos avisar del problema o mejor aún, dar a los usuarios de la aplicación web una alternativa, usando otra tecnología como XMLHttpRequest que no tiene el riesgo de estar bloqueada.
Pueden probar fácilmente si este ejercicio va a funcionar en su máquina, conectándose a la página https://www.websocket.org/echo.html

[["https://github.com/huibert7/Node-Red-tutorial/blob/master/Documentation/Images/1 Echo Test.png"|alt="Echo Test"]]

## ¿Cómo funcionan los WebsSockets?
### Modelo de programación
Los WebSockets son muy fáciles de utilizar, tal y como veremos a continuación. En realidad solo debemos conocer dos grupos de funciones. Por un lado están aquellas con las que vamos a administrar el ciclo de vida del objeto y por el otro, los callbacks o sea las funciones que el browser va a mandar invocar cuando detecte un evento relacionado con el websocket.

### El ciclo de vida de los websockets
El primer paso consiste en crear el WebSocket, normalmente al terminar de cargarse la página, dentro de la función onload() del objeto window.
```
      var socket = new WebSocket(‘ws://www.example.com/socketserver');
```
Podrá observar que lo único que necesitamos pasar como argumento es el url al que nos vamos a conectar. Cuando trabajemos en nuestro ejemplo sobre Bluemix, el url se dividirá en dos partes, el url de nuestro servidor (por ejemplo “Node-RED-prueba1.mybluemix.net”) y la ruta que definiremos para el websocket (por ejemplo “/ws/misocket”).

Por motivos de seguridad, la especificación obliga que los navegadores solo puedan abrir un websocket. La idea es evitar posibles ataques de tipo DoS (Negación de Servicio) desde un browser. A pesar de ello, no todos los navegadores han implementado esta limitación. Sin embargo, es importante conocerla, porque de otra manera podría resultar incomprensible porqué un determinado programa funciona en un browser y en otro no. Otra consideración a tomar en cuenta es que si una página web se accesa usando el protocolo HTTPS, deberá usar forzosamente el protocolo wss para conectarse con el servidor. Esto es lógico porque no tiene sentido tener páginas seguras, con elementos inseguros, pero vale la pena recalcarlo.
A menos de que nuestra página solo esté buscando recibir mensajes del servidor, en algún momento tendremos que mandar información al servidor usando la función send, tal y como se muestra a continuación:
```
      socket.send(”Esta es una prueba”);
```
En nuestro ejemplo, mandaremos una cadena de caracteres al servidor. Sin embargo, utilizando tan solo un poco de JavaScript podemos intercambiar datos en formato JSON con el servidor, de forma muy natural. Veamos un ejemplo:
```
      var mensaje = {
            nombre: “Juan”,
            apellido: “Arbeloa”, edad: 27 };
      socket.send(JSON.stringify(mensaje));
```
A efectos prácticos, este ejemplo es similar al anterior, porque en realidad también estamos mandando una cadena de caracteres, solo que en este caso, esa cadena representa un objeto JSON que puede ser convertido de nuevo a un objeto JavaScript muy fácilmente en su punto de destino tal y como veremos más adelante.

Finalmente, es posible que en algún momento desee cerrar la conexión hacia el servidor, lo cual se hace de forma lógica, tal y como se muestra a continuación:
```
      socket.close();
```
Si la página web no cierra el websocket de forma explícita, no pasa nada, porque cuando el usuario se mueva a otra página, el servidor detectará automáticamente que el cliente ya no está conectado y cerrará la conexión. Esto es algo que es responsabilidad exclusiva del servidor, el cual debe estar comprobando a intervalos constantes que los clientes siguen conectados. Afortunadamente, eso es algo que hace por nosotros NODE-Red de forma automática, lo que simplifica el problema de forma significativa.

### Callbacks para manejar los eventos
Los WebSockets funcionan de forma asíncrona. Esto significa que una aplicación nunca se quedará bloqueada en una línea de código esperando a que le llegue un mensaje del servidor. En lugar de eso, será el navegador el que nos avise cuando llegue un mensaje, invocando una función callback en la cual nosotros podremos procesarlo. Esto es importante porque permite evitar que la página web deje de responder a las interacciones con el usuario mientras se interactúa con el servidor.

En JavaScript, las funciones son objetos. Esto significa que son del tipo Object y que es posible asignar a una variable una función, como si de cualquier otro tipo de objeto se tratara. De hecho, las funciones no solo pueden ser almacenadas en variables, sino que además, pueden pasarse como argumento a una función o ser devueltas como resultado de la invocación de una función. Esta característica interesantísima de JavaScript, es la que se explota para implementar las funciones de callback que invocará el navegador cuando un WebSocket reciba un evento.

El objeto WebSocket tiene las siguientes propiedades:

* onmessage
* onopen
* onclose
* onerror

La idea es asignar a esos atributos funciones, las cuales serán invocadas cuando se produzcan los eventos que queremos detectar. En el siguiente ejemplo, asignamos al atributo onmessage una función anónima (se le llama de esta manera porque, como pueden observar, no tiene nombre).
```
 // Gestión de la recepción de mensajes mandados por el servidor.
  socket.onmessage = function(event) {
    var message = event.data;
    ...
  };
```
Cuando se crea un WebSocket, ninguno de los atributos del objeto tiene valor alguno, por lo que es necesario crearlas si queremos recibir los eventos. Tengan en cuenta que como JavaScript, a diferencia de otros lenguajes como C o Java, no es strongly typed (es decir que no se tiene que definir de antemano el tipo de las variables y que el compilador verifica que no haya errores de confusión de tipos), nada impide que escribamos el siguiente código:
```
   socket.onmessage = “Hola”;
```
Esa línea de código está claramente mal porque el browser espera que el atributo onmessage contenga una función, no una cadena de caracteres. Para evitar problemas, internamente, antes de invocar la función onmessage, el navegador ejecutará un control para verificar que los atributos que deban contener funciones callback realmente contengan funciones. El código ejecutado por el browser internamente probablemente se parezca mucho al código que se muestra a continuación:
```
   if (typeof socket.onmessage === "function") {
      // Podemos invocar la función porque hemos comprobado que el atributo
      // onmessage realmente contiene una función
      socket.onmessage(evento);
   }
```
## Trabajo preliminar
Si lo desea, puede omitir esta sección, ya que todo el trabajo preliminar ya ha sido realizado para usted por IBM. Sin embargo, esta información puede ser interesante si dese ejecutar posteriormente el ejercicio con su propia cuenta de IBM Cloud.
### Creación de una cuenta en IBM Cloud
Para empezar a desarrollar su primer proyecto sobre IBM Cloud conéctese a http:// cloud.ibm.com/login

Si aún no tiene un ID de IBM, como parte del proceso de registro deberá obtener uno, el cual le servirá para poder acceder a su cuenta de IBM Cloud. Además de ser un requisito para poder usar la plataforma PaaS, tener un ID de IBM le permitirá realizar otras operaciones en el sitio web de IBM como por ejemplo obtener white papers, descargar software de evaluación de IBM o participar en los foros de discusión de IBM developerWorks (http://www.ibm.com/ developerworks), un sitio web que es una mina de oro para desarrolladores.

Si ya tiene un ID de IBM, puede usarlo para solicitar su cuenta de evaluación y una vez completado el proceso podrá usar el servicio con el ID de IBM que ya tenía.

Para obtener su ID de IBM, pulse el botón
   
A continuación le aparecerá la siguiente forma, la cual deberá rellenar para poder obtener su ID de IBM y poder empezar a trabajar.

Primero rellene los datos de su correo electrónico y valídelo. Posteriormente rellene los datos de la forma y pulse el botón

A continuación, IBM le mandará un correo electrónico a la dirección que proporcionó. Lea el correo y siga las instrucciones para terminar el proceso de registro. Ya está listo para empezar a usar la plataforma PaaS de IBM.

## Activación de los servicios que usará nuestra aplicación
Nuestra aplicación será muy sencilla y solo usará dos servicios. Por un lado necesitaremos el servicio de traducción y por otro lado usaremos Node-RED para crear el web-socket que invocará el servicio de traducción cuando lo solicite el usuario.
### Activación del servicio de traducción de Watson

El servicio de traducción de Watson, al igual que todos los demás, está disponible en el catálogo de servicios de IBM Cloud, el cual se muestra a continuación.
 
Seleccione la sección de AI (Inteligencia artificial para encontrar el servicio de traducción de Watson.
Ahora pulse el botón de Language Translator.

En la siguiente pantalla se muestra información adicional del servicio que usaremos.
Simplemente pulse el botón
Una vez creado el servicio, la página le muestra información acerca de cómo usar el servicio.
 
Si hace click sobre la pestaña “Credencials” podrá obtener información acerca de los parámetros que necesitará para pode enfocar el servicio de forma segura sin que ningún otro cliente de IBM Cloud pueda usarlo.

Anote el API Key porque lo necesitará más adelante en el ejercicio.

### Activación del servicio de Node-RED
Tal y como ya mencionamos anteriormente, usaremos Node-RED para crear el flujo aflictivo en el servidor. Para poder crear el servicio de Node-RED, debemos regresar al catálogo de servicios de IBM Cloud y seleccionar la pestaña Starter Kit, tal y como se muestra a continuación.
  
 Pulse el botón Node-RED Starter para iniciar la activación del servicio.
 Introduzca un nombre cualquiera (por ejemplo MiTraductor) para su aplicación.
  Pulse el botón
IBM Cloud creará entonces un nuevo entorno para ejecutar sus aplicaciones. Este es un proceso que puede durar unos minutos. Espere a que en la pantalla se indique que su aplicación está despierta (“Awake” en inglés), tal y como se muestra a continuación.
 
 Para iniciar la configuración de su entorno de desarrollo Node-RED pulse el siguiente botón:
Pulse el botón “Next” para continuar.
Como se trata de un ejercicio que pretendemos ejecutar en menos de 45 minutos, vamos a realizar la configuración más básica posible del entorno Node-RED, sin ponerle seguridad alguna. Es evidente, que en condiciones normales, esto no es algo recomendable.
   
 Haga click en la casilla para confirmar que no desea seguridad y luego pulse el botón “Next” para continuar.
Ignore la siguiente pantalla y simplemente pulse el botón “Next” para continuar.
Ya casi terminamos. En la siguiente pantalla simplemente pulse el botón “Finish” para terminar de configurar el ambiente y arrancar la instancia. Cuando aparezca la siguiente pantalla, habremos completado el proceso de configuración y estaremos listos para empezar.
  
## Desarrollo de la aplicación con Node-RED

### Conectarse a IBM Cloud
Para empezar a desarrollar su primer proyecto sobre IBM Cloud conéctese a http://
cloud.ibm.com/login e introduzca el IBM ID que creó o que se le proporcionó. Pulse el botón “Continue” y luego proporcione la clave de acceso.

Si sus datos son correctos, el sistema le dará al acceso a su tablero de control (“Dashboard” en inglés), en el cual se muestra de forma resumida la información de su ambiente PaaS.
    
Como podrá observar, ya tiene dos servicios activados, los cuales fueron creados previamente (un ambiente de desarrollo con Node-RED y el servicio de traducción de Watson que usaremos más adelante).

Haga click sobre “View Resources”.

Ahora haga click sobre la liga correspondiente a su aplicación dentro del apartado “Cloud Foundry Apps”.

Esto le lleva a una página en la que puede ver el estatus de su aplicación. En este momento debe tener una sola instancia funcionando, la cual tiene asignada 256MB de memoria.

Al tratarse de una cuenta de evaluación, sin costo, la cantidad de memoria que puede tener la instancia está limitada. Sin embargo, para ambientes productivos es posible contratar entornos mucho más grandes.

Para acceder al entorno de desarrollo Node-RED pulse el triángulo situado dentro del botón “Routes” para que aparezca el siguiente menú:

Haga click sobre el URL (en este caso MiTraductor.mybluemix.net, pero en su caso depende
del nombre que le haya puesto a la aplicación). Esto le llevará a la página de Node-RED. Pulse el siguiente botón para empezar a desarrollar:
   
### Desarrollo del flujo en Node-RED
Node-RED es un entorno de programación visual que corre sobre Node.js. Con Node-RED vamos a construir flujos que reciben una entrada y depositan el resultado en una salida. En nuestro caso, lo que vamos a hacer es un flujo que reciba el texto en un idioma cualquiera , que reconozca en qué idioma fue escrito y si Watson sabe traducirlo al inglés, devuelva el texto en ese idioma como salida.

La comunicación entre el navegador y el servidor se hará mediante el uso de web-sockets y usaremos JSON para codificar los datos que intercambiemos.

### Construcción del flujo
Si siguió las instrucciones previas, en este momento su pantalla debe mostrar algo muy similar a lo que se aprecia en la siguiente imagen:

El flujo se construye deslizando nodos de la paleta situada del lado izquierdo de la pantalla al espacio vacío en el centro cuyo tab tiene el nombre “Flow 1”.

Sabemos que nuestro flujo va a recibir el texto a traducir a través de un websocket. Por lo tanto vamos a empezar a diseñar nuestro flujo arrastrando un websocket de la sección “input”.
  
Tengan cuidado porque hay otro websocket en la sección “output” que es diferente y que usaremos más adelante.
Es necesario configurar el websocket, por lo que simplemente hacemos doble-click sobre el ícono para editar la configuración.

El tipo de websocket definido por defecto (“Listen on”) es correcto, por lo que no es necesario cambiarlo.
Lo que sí debemos definir es la ruta (path en inglés) en la que estará escuchando nuestro websocket. Para ello es necesario pulsar el botón con el ícono en forma de lápiz:

Esto nos abre una nueva ventana en la que podemos definir la ruta en la que estará escuchando nuestro WebSocket. Por defecto vemos que la ruta usada es /ws/example. Sin embargo es muy fácil cambiarlo por la ruta que más le guste.

Para este ejercicio decidí usar la ruta /ws/misocket, por lo que le sugiero que haga lo mismo para poder seguir las instrucciones. Esto significa que, en mi caso, para conectarme a este websocket deberé usar el siguiente URL, ws://mitraductor.mybluemix.net/ws/misocket.
 
Para continuar, pulse el botón “Add”.
Ahora ya solo falta ponerle un nombre a este nodo para facilitar la lectura del flujo. En mi caso decidí llamarlo “inputsocket”.

Pulse el botón “Done” para continuar”.
Como nuestro flujo tan solo tiene un nodo, aún no hace, así que continuemos agregando más nodos. Sabemos que la respuesta de nuestro flujo se entregará a través de un websocket. Por lo tanto, podemos arrastrar un websocket de tipo “output” a nuestro espacio de trabajo.
   
Este websocket también se necesita configurar. Utilizando el mismo procedimiento que el que usamos para el websocket de entrada, modifique los campos tal y como se indica en la siguiente pantalla.

En este caso la configuración será más sencilla, porque vamos a reutilizar la misma ruta que la que usamos para el websocket de entrada. Esto es normal, dado que en realidad estamos hablando de un solo WebSocket, el cual se va a usar tanto para recibir mensajes como para mandar la contestación al browser. Elija un nombre para este nodo (yo decidí llamarlo “outputsocket”). Pulse el botón “Done” para continuar”.

En este momento, en su espacio de trabajo deben aparecer dos nodos, tal y como se muestra a continuación.
   
En nuestro flujo, lo primero que debemos hacer tras recibir el mensaje es identificar el idioma en el que fue escrito. Para ello vamos a usar el nodo “language identifify" que se encuentra dentro del grupo “IBM Watson” en la paleta de nodos. Arrastre el nodo desde la paleta y colóquelo entre los dos websockets que ya ha configurado.
Conecte el nodo “inputsocket” con el nodo “language identity”. Haga doble click sobre el segundo nodo para configurarlo. Vamos a hacer dos cosas. Vamos a cambiar el nombre del nodo a “idioma” y vamos a proporcionar las credenciales que Node-RED necesita para poder invocar el servicio.
Los campos Username y Password se deben rellenar con el usuario y password que usan para conectarse a IBM Cloud. El API Key es un poco más difícil de obtener. En su navegador regrese a la página que muestra su tablero de control.
   
Haga click sobre “Services” dentro de la tarjeta “Resource summary”.En la siguiente página se muestra una lista de recursos.

Haga click sobre el servicio cuyo nombre empieza por “Language Translator-“. Eso lo lleva a una página en la que se muestra el 
API Key y el URL del servicio de traducción. Como la clave está oculta por cuestiones de seguridad, debe pulsar “show credentials” para poder verla y copiarla.

Con esta información ya puede completar la información requerida. Pulse el botón “Done” para terminar de configurar el nodo “idioma” y poder continuar.
   
Ahora ya tenemos los dos primeros nodos de nuestro flujo configurados y conectados.
Cuando un websocket recibe un mensaje, automáticamente crea una variable JavaScript llamada msg y guarda el cuerpo del mensaje (en nuestro caso una cadena de caracteres) en el atributo payload de esa misma variable.
El nodo de detección de idioma funciona de forma muy sencilla, analiza el texto que le llega en msg.payload y regresa el resultado (bajo la forma de un código de idioma, como por ejemplo “en-US”) en msg.lang.language. Como la detección de idiomas no es una ciencia exacta, especialmente con mensajes cortos, también nos provee el porcentaje de probabilidad de que el resultado sea correcto en msg.lang.confidence.
El nodo de detección de idiomas no altera de forma alguna el contenido de msg.payload, por lo que este valor puede ser utilizado por el siguiente nodo. Sin embargo, no podemos confiar en que todos los nodos siempre preservarán los atributos de la variable msg. Por ejemplo, el nodo de traducción automática recibe el texto a traducir en msg.payload y lo reemplaza por la versión traducida. Eso significa que si quisiéramos conservar el texto original, deberíamos conservarlo en otro atributo de la variable msg que sepamos que no será alterado posteriormente por el flujo. Como en este caso no tenemos interés en preservar el mensaje original, el problema es más sencillo y no tenemos de qué preocuparnos.
El nodo de traducción, que usaremos más adelante, necesita saber de qué idioma a qué otro idioma debe traducir el texto. El nodo de traducción espera que msg.srclang contenga el identificador del idioma original (por ejemplo es-MX para un texto escrito en español mexicano) y que msg.destlang. contenga el identificador del idioma destino (en nuestro caso siempre será en, o sea inglés).
Ya tenemos el identificador del idioma original del texto, el cual está almacenado en msg.lang.language. El único problema es que el nodo de traducción espera que ese valor esté en msg.srclang por lo que es necesario copiar el valor de un atributo a otro. Eso es algo que vamos a hacer usando una función JavaScript.
Agregue un nodo de tipo “function” a su flujo y conéctelo al nodo de idioma. Haga doble click sobre el mismo para editarlo.
A este nodo le vamos a poner el nombre de “Prepara” porque solo se va a encargar de preparar los valores que espera recibir el siguiente nodo, que será el de traducción de texto. También vamos a completar la función JavaScript para que haga lo que necesitamos. En este caso solo necesitamos agregar la siguiente línea:
msg.srclang = msg.lang.language
Cuando se ejecute la función JavaScript, msg.srclang tomará como valor el identificador del idioma que detectó el nodo “idioma”.
 
  Pulse el botón “Done” para terminar de configurar el nodo “Prepara” y poder continuar.
Su flujo debería ahora estar tal y como se muestra a continuación.
Ahora ya estamos en condiciones de utilizar el nodo de traducción, conocido en inglés como “language translator” y que se encuentra bajo el grupo de “IBM Watson”. Arrastre el nodo sobre su espacio de trabajo y colóquelo junto al nodo de “Prepara”. Ahora conecte ambos nodos.
  
 Haga doble click sobre el nodo de traducción para configurarlo. Llame “Traducción” a este nodo. Use las mismas credenciales que usó anteriormente para invocar el servicio de detección de idioma para configurar este nodo.
Como este servicio va a traducir siempre el texto al inglés es importante que el idioma de destino (“Target”) esté marcado como “English” ya que nuestra función no fija ese valor.
Finalmente, asegúrese de que está marcada la opción “Not using translation utility” tal y como se muestra a continuación.
Pulse el botón “Done” cuando haya terminado de configurar el nodo.
  
 Ya nos falta muy poco para terminar nuestro flujo. Solo falta conectar el nodo de “Traducción” con el de “outputsocket” que creamos al inicio. Una vez que lo haya hecho, su flujo debe quedar de la siguiente manera:
Nuestro flujo está completo. Para poder probarlo, primero es necesario ponerlo en producción en su servidor. Para ello simplemente pulse el botón “Deploy”.
En principio, tras unos pocos segundos, en su browser debería aparecer el mensaje “Successfully Deployed” (Publicado exitosamente).
Ya podemos conectarnos al WebSocket desde cualquier página web. Sin embargo, para ello necesitamos crear primero una página HTML, lo cual es exactamente lo que haremos en la siguiente sección. 
  
## Desarrollo de la aplicación web
La aplicación web se dividirá en tres archivos:
* WebSockets.html
* WebSockets.js
* style.css

El instructor debe haberle entregado un archivo comprimido que contiene los tres archivos.
Descomprima el archivo .zip.

El archivo WebSockets.html
Este es el contenido del archivo HTML. Se trata de un documento muy sencillo, dado que no contiene nada de JavaScript (el código está en el archivo WebSockets.js) y toda la información referente al diseño está en el archivo style.css.
```
<!DOCTYPE html>
<html lang="es">
<head>
      <meta charset="utf-8">
      <title>Traducci&oacute;n de texto con IBM Cloud</title>
      <link rel="stylesheet" href="style.css">
</head>
<body>
      <div id="page-wrapper">
            <h1>Traducci&oacute;n de texto</h1>
            <div id="status">Conectando...</div>
            <ul id="messages"></ul>
            <form id="message-form" action="#" method="post">
                  <textarea id="message" placeholder="Escribe tu texto
aqu&iacute;..." required></textarea>
                  <button type=“submit">Traducir texto</button>
            </form>
</div>
      <script src="appTest.js"></script>
</body>
</html>

Como podrá observar, esta página web realmente solo contiene una lista en la que mostraremos el mensaje enviado al servidor así como la respuesta que recibamos y una forma que usaremos para capturar el mensaje junto con el botón que presionará el usuario para mandarlo.
El archivo WebSockets.js
Este archivo contiene el código JavaScript que le da la inteligencia a nuestra página HTML. Si observa el código, verá que solo definimos una función, la cual se ejecutará cuando se termine de cargar la página HTML. En ella se crea el WebSocket y se definen todas las funciones callback que invocará el navegador cuando se produzcan eventos relacionados con ese objeto, como el establecimiento de la conexión, la recepción de un mensaje, etc.
window.onload = function() {

   // El DOM permite obtener referencias hacia los elementos de la página HTML
con los que vamos a interactuar.
  var form = document.getElementById('message-form');
  var messageField = document.getElementById('message');
  var messagesList = document.getElementById('messages');
  var socketStatus = document.getElementById('status');
  // Crea una conexión a nuestro WebSocket en BlueMix.
  var socket = new WebSocket('ws://mitraductor.mybluemix.net/ws/misocket');
  // Manejo básico de cualquier error que pueda ocurrir con respecto al
WebSocket.
  socket.onerror = function() {
    console.log('Error de WebSocket');
};
  // Muestra un mensaje de estatus de conectado cuando el WebSocket es
abierto de forma exitosa.
  socket.onopen = function(event) {
    socketStatus.innerHTML = 'Conectado a: ' + event.currentTarget.URL;
};
  // Gestión de la recepción de mensajes mandados por el servidor.
  socket.onmessage = function(event) {
    var message = event.data;
    messagesList.innerHTML += '<li class="received"><span>Recibido:</span>' +
message + '</li>';
};
  // Muestra un mensaje de estatus de desconectado si por algún motivo se
cerrara el WebSocket.
  socket.onclose = function(event) {
    socketStatus.innerHTML = 'Desconectado del WebSocket.';
};
  // Manda el mensaje cuando se pulsa el botón de submit (Enviar mensaje).
  form.onsubmit = function(e) {
    // Lee el mensaje escrito por el usuario en la textarea.
    var message = messageField.value;
    // Manda el mensaje al servidor usando el WebSocket.
    socket.send(message);
    // Agrega el mensaje a la lista de mensajes.
    messagesList.innerHTML = '<li class="sent"><span>Env&iacute;ado:</span>'
+ message + '</li>';
    // Borra el contenido de la caja de texto en el que el usuario escribe el
mensaje.
    messageField.value = '';
    return false;
  };

 };
El archivo style.css
Este archivo realmente no es muy importante. Solo contiene la hoja de estilos que le da forma y color a la página HTML. Si no le gusta la apariencia de la página, puede usar un editor de CSS3 para modificar este archivo y así cambiar el diseño de la misma.
Probemos la aplicación
Normalmente este es el punto en el que subiríamos la aplicación web a la nube para probarla. Sin embargo, en este caso, para fines puramente pedagógicos, primero vamos a hacer una prueba local. La idea es tener la aplicación web en nuestra PC y dejar que se conecte al WebSocket que estará corriendo sobre BlueMix. Eso demostrará que nuestro flujo es realmente un componente que puede ser utilizado de forma remota desde cualquier página web, aunque no esté en Bluemix.
Abra cualquier navegador reciente y abra el archivo WebSockets.html, ya sea usando la opción “Open File...” o equivalente o arrastrando el archivo sobre la ventana de su browser. Esto es lo que debería ver:
 
 Aquí les dejo unas frases en distinto idiomas para que puedan probar la aplicación.
Holandés
Ik ben in Utrecht geboren.
Op een dag zal Nederland het WK winnen.
Francés
J’ai fait mes études en France, à Bordeaux.
Je regrette énormément l’incendie qui a brulé la cathédrale de Nôtre Dame de Paris.
Alemán
Ich habe Deutsch in Genf gelernt.
Fussball ist ein spiel und am ende gewinnt Deutschland
