# TRABAJO FINAL DE MÁSTER

El objetivo es la detección de vulnerabilidades de una API que se usa como ejercicio para practicar y mejorar las habilidades a la hora de buscar y explotar vulnerabilidades. Como la idea es simular un caso de auditoría de la API, en la vida real es importante tener claro los requisitos mínimos para poder iniciarla y una estrategia, ya que el tiempo es algo que apremia y no puede desperdiciarse debido a los costos de operación que implica realizarla.

## 1. PLANIFICACIÓN

### 1.1. REQUISITOS MÍNIMOS

Para poder definirlos, se necesita saber qué es lo que se va a auditar y cuál es el alcance mínimo requerido como requisito. Para este caso, será una API llamada "Damn Vulnerable RESTaurant", una API hecha con vulnerabilidades a propósito para practicar hacking ético. Los datos más relevantes para poder establecer una estrategia son:

- La API está hecha en un framework llamado FastAPI, que usa Python como lenguaje de programación.
- La base de datos está en Postgres.
- Se puede desplegar usando docker compose.


Con estos datos, lo mínimo que se requiere es:

- Un computador que sea capaz de ejecutar Docker. Por practicidad, se utilizará Linux Ubuntu 24.04 como sistema operativo anfitrión.

- Un computador adicional que tenga OWASP ZAP instalado o una máquina virtual con Kali Linux, que ya lo trae por defecto.

- Documentación de las funcionalidades de la API (por ejemplo, Swagger OpenAPI).

Como es un ejercicio de una API pequeña (ver su documentación en el README.md en su repo: https://github.com/theowni/Damn-Vulnerable-RESTaurant-API-Game.git), con un computador con la siguiente configuración es suficiente:

- Hardware: procesador Intel Core i7 10750H, 32 GB de RAM y 240 GB de almacenamiento.
- SO anfitrión: Ubuntu 24.04, con docker y docker compose.
- Máquina virtual con Kali Linux, que ya trae OWASP ZAP, que corre en VirtualBox v7.1.12. Para este trabajo se empleará OWASP ZAP v2.16.0.
- El computador estará conectado a una red LAN gobernada por un router, que le asignará las direcciones IP dinámicamente tanto al anfitrión como a la MV.

### 1.2. ESTRATEGIA DE LA PRUEBA

1. Preparación de la prueba. Se despliega la API con docker compose en el anfitrión. En el mismo computador, se ejecuta una MV con Kali Linux, que ya trae OWASP ZAP.
2. Se hace una exploración manual usando los navegadores Google Chrome y Mozilla Firefox apuntando a unas direcciones definidas.
3. Para el caso de encontrar documentación (Swagger por ejemplo), se intentará descargar la especificación OpenAPI. Si no la tuviere o no se pudiere encontrar, entonces lo que se haría en la vida real sería preguntar sobre el descriptor de la API en un formato estandarizado (OpenAPI).
4. Escaneos automatizados y manuales con OWASP ZAP.
5. Explotación de vulnerabilidades encontradas en el punto 4 y si la explotación lo permite, iterar desde el punto 4 para escalar privilegios en la misma API (tipo de usuario). Se le dará prioridad a aquellas vulnerabilidades que permitan la escalada de privilegios, por ejemplo, que un usuario de cierto rol pueda realizar cosas que no debería.


## 2. EJECUCIÓN DE LA ESTRATEGIA PARA LA PRUEBA

### 2.1. PREPARACIÓN DE LA PRUEBA

#### DESPLIEGUE DE LA API

El repo de la API a probar es el siguiente:

~~~
https://github.com/theowni/Damn-Vulnerable-RESTaurant-API-Game.git
~~~

Se realiza la descarga en una carpeta del usuario usando git clone:

~~~
$ git clone https://github.com/theowni/Damn-Vulnerable-RESTaurant-API-Game.git
~~~

Y en la documentación (README.md) aparecen las instrucciones para desplegar la API, pero se realizó una pequeña modificación al docker-compose.yml para que la base de datos quede en la misma carpeta del repo. De esta manera es posible borrarla usando:

~~~
$ sudo rm -r ./postgres_data
~~~

Cada vez que se requiera. Esto se puede hacer cuando la API no está desplegada. PAra el caso en que lo esté, toca desinstalar el despliegue usando:

Usando docker compose v2:
~~~
$ docker compose down
~~~
O usando docker compose v1:
~~~
$ docker-compose down
~~~

#### MÁQUINA VIRTUAL (MV) DE KALI LINUX

Usando VirtualBox v7 se despliega una máquina virtual cuya imagen se puede conseguir en el sitio oficial de Kali Linux. Con 3 núcleos de procesador y 8 GB de RAM se trabaja bien.


### 2.2. EXPLORACIÓN MANUAL MEDIANTE NAVEGADORES WEB

Para esto se emplearán:

- Mozilla Firefox v115.11.0esr (64-bit)
- Google Chrome v120.0.6099.216 (Build oficial) (64 bits)

Las direcciones que se van a explorar son las siguientes:

- "/" (raíz), normalmente se espera la página de inicio o la redirección a una página de autenticación.
- "/admin": normalmente las apis web tienen una entrada "admin".
- "/docs": el manual de Damn Vulnerable RESTaurant sugiere que aquí hay documentación de la API en Swagger.
- "/redoc": el manual de Damn Vulnerable RESTaurant sugiere que aquí hay documentación de la API en ReDoc.

#### 2.2.1. EXPLORACIÓN MANUAL CON MOZILLA FIREFOX

"/":

!["/"](./recursos/1-1.png)

"/admin":

!["/admin"](./recursos/1-6.png)

"/docs":

!["/docs"](./recursos/1-7.png)

"/redoc":

!["/redoc"](./recursos/1-8.png)

#### 2.2.2. EXPLORACIÓN MANUAL CON GOOGLE CHROME

"/":

!["/"](./recursos/1-2.png)

"/admin":

!["/admin"](./recursos/1-3.png)

"/docs":

!["/docs"](./recursos/1-4.png)

"/redoc":

!["/redoc"](./recursos/1-5.png)


#### 2.2.3. CONCLUSIÓN DE EXPLORACIÓN MANUAL

1. Se pudo obtener la documentación del uso de la API a través de los enlaces /docs y /redoc. Además se puede obtener la especificación OpenAPI, que será muy útil para la sección 2.3. EXPLORACIÓN CON HERRAMIENTA DE ESCANEO AUTOMATIZADO INICIAL: ESPECIFICACIÓN DE LA API de este trabajo.

2. Llama la atención aquí es que Firefox incluye dos barras de opciones adicionales para poder interactuar mejor con la API (ver "/" y "/admin").


### 2.3. EXPLORACIÓN CON HERRAMIENTA DE ESCANEO AUTOMATIZADO INICIAL: ESPECIFICACIÓN DE LA API

Para un caso de la vida real la idea es que el equipo de trabajo que desarrolla la API entregue una documentación de esta especificación, ya sea como un archivo, o si es una empresa que sigue los estándares de una API RESTful se pueda conseguir mediante un enlace, por ejemplo "/docs" o "/redoc". Para esta caso en particular, los desarrolladores sí han suministrado este recurso y se puede descargar efectivamente mediante los enlaces "/docs" o "/redoc" (ver README.md de la API), aunque también fueron descubiertos en el punto 2.2 EXPLORACIÓN MANUAL.

### 2.4. ESCANEOS AUTOMATIZADOS Y MANUALES CON OWASP ZAP

OWASP ZAP es el programa principal que se utilizará para realizar la prueba de penetración en esta API, que implica escaneos automatizados y manuales, así como técnicas de escaneo activo (peticiones con con cargas pagas envenenadas), pasivo (solo revisión de vulnerabilidades en cuyas transacciones peticiones-respuesta pueda haber revelación de información delicada, sea técnica por ejemplo Cookie sin HttpOnly o la permitividd de inclusiones de scripts en dominios cruzados, así como secretos).

#### 2.4.1. CONFIGURACIÓN DE OWASP ZAP

Lo primero que hay que hacer es configurar el programa para prepararlo para las pruebas. Con esto habrá la seguridad de sobre qué se está trabajando y qué resultados esperarse durante las pruebas, ya que esto define el comportamiento del programa para la realización de estas.

Se dejará activado el escáner pasivo, ya que permite realizar una auditoría de forma más rápida y controlada, esto es que en la medida que se avanza manualmente, el escáner pasivo hará tareas en segundo plano para detectar posibles vulnerabilidades. La configuración de este quedará así para este trabajo:

##### 2.4.1.1. PASSIVE SCANNER

###### Passive Scan Rules:

Se dejarán todos los valores por defecto, y esto es:

- Threshold = Medium para todos los tests, no importando el status (beta, alpha o release).

!["passive scan rules"](./recursos/2/1.png)

###### Passive scan tags:

Se dejarán todos los valores por defecto, y esto es:

- Todas habilitadas (enabled).

!["passive scan tags"](./recursos/2/2.png)

###### Passive scanner:

Se dejarán todos los valores por defecto, y esto es:

- Se ecanearán todos los mensajes (incluso fuera de la cobertura (scope)).
- No incluirá tráfico del fuzzer durante escaneo pasivo.
- 3 hilos para escaneo pasivo.
- Alertas y tamaño del cuerpo en bytes para escanear sin límites.

!["passive scanner"](./recursos/2/3.png)

##### 2.4.1.2. MODO DE ZAP

Se trabajará en modo estándar (Standard Mode). Se habría podido utilizar el modo de ataque (ATTACK mode), pero habría que estar pendiente de las configuraciones del escáner activo, ya que podría en ciertos casos consumir más recursos de los necesarios, además de que hay que reconocer que se necesita experticia para esto.

##### 2.4.1.3. VALUE GENERATOR

Esta configuración es importante también, porque a pesar de que se puede dejar por defecto, añadirle valores puede ayudar a mejorar la auditoría, haciéndola personalizada en las búsquedas y ataques. El add-on OpenAPI, que se usará en el punto 2.3, la usa durante la importación de una definición, otra razón más para configurarla correctamente. Algunos campos que no estén definidos serán rellenados por valores que tiene este generador por defecto en ZAP, por ejemplo "John Due".

##### 2.4.1.4. SCRIPTS

Cualquier script generado desde "HTTP Sender" debe estar deshabilitado:


!["Script HTTP Sender deshabilitado"](./recursos/2/3_1.png)

Si no se deshabilita, la API podría interpretarlo como peticiones de un usuario registrado y autorizado. Aquí la idea es hacer las peticiones como si fuera un usuario invitado o no registrado (público).

#### 2.4.2 IMPORTACIÓN DE OPENAPI.JSON A ZAP

Como la documentación permite obtener este documento y ZAP permite su importación, se realizará para agilizar los ataques. Durante la auditoría de una API, esto es un requisito mínimo para poder realizarla.

![](./recursos/2-1.png)

![](./recursos/2-2.png)

![](./recursos/2-3.png)

![](./recursos/2-4.png)

![](./recursos/2-5.png)


#### 2.4.3. ANÁLISIS DEL RESULTADO LA IMPORTACIÓN DE LA ESPECIFICACIÓN DE LA API

Como se había dicho previamente durante la configuración del OWASP ZAP en el punto 2.4.1.3. VALUE GENERATOR, el addon que importa el archivo OpenAPI.json realiza un escaneo activo breve a cada URL descrita en dicho descriptor, lo cual implica que le haya pasado una carga paga envenenada con información, así que es posible que haya podido incluso modificado la información que está almacenada en la base de datos (sea añadido, editado o borrado). Se revisará cada una de las URL, y se revisará qué tipo de respuesta se ha obtenido para cada una, así como las alertas obtenidas predeterminadas.

Lo primero que hay que revisar es ver si en el historial ha habido peticiones que se han respondido satisfactoriamente, es decir, aquellas cuyo código de estado HTTP es 2xx:


![](./recursos/2/4.png)

Se pueden ver 4 respuestas con este código. Llama mucho la atención una que tiene código 201 que es cuando se crea una entidad y se almacena en la base de datos. Algunas deberían ser posibles de realizar desde un usuario invitado, pero otras no. Hay que revisar si este es el caso:

![](./recursos/2/5.png)

Se puede apreciar que es el registro de un cliente, lo cual es una actividad común y prácticamente fundamental en un programa en el que se gestiona un restaurante, pues cualquiera que esté en Internet podría acceder y tener la capacidad de poder registrarse. Si se tratara de un usuario cuyos privilegios son más altos (por ejemplo un empleado o alguien de cargo más alto, o que tenga más responsabilidad sobre el sistema), sería algo para marcar como una "vulnerabilidad al escalamiento de privilegios". Pero, por defecto, el usuario ha sido designado como cliente.

Analizando las alertas, hay unas que llaman la atención, por ejemplo la tecnología utilizada para el despliegue expuesta en las respuestas. Esto añade información para poder realizar ataques más precisos y menos demorados en alcanzar:


![](./recursos/2/6.png)

Se tiene entonces que el framework es FastAPI v0.103.0, que trabaja sobre Python v3.10. A continuación, se mostrará la información que hay en https://www.cvedetails.com/ al día de hoy 23 de septiembre de 2025 con respecto a estas tecnologías:

Respecto a FastAPI, no hay vulnerabilidades encontradas para la v0.103.0:

![](./recursos/2/7.png)

Respecto a Python, no hay resgistro de vulnerabilidades para la v3.10.18:

![](./recursos/2/8.png)

#### 2.4.4. EXPLORACIÓN MANUAL DE LAS DIRECCIONES QUE CONTIENEN PARÁMETROS EN LA RUTA DE LA URL EN BÚSQUEDA DE IDOR

Es importante saber si las rutas de URL que apuntan a recursos están seguras. Las rutas de URL que apuntan a recursos tienen el siguiente formato:

~~~
GET dir_web/api/{id_de_recurso}
~~~

Por ejemplo, la siguiente petición debería devolver como resultado una orden cuya id = order_id:

~~~
GET http://192.168.2.92:8091/orders/{order_id}
~~~


Hay casos en los que se puede suceder (y no debería) en los que se puede hacer una solicitud de tipo POST, PUT, DELETE o cualquiera que implique la modficación en la base de datos sin estar autorizados, y a veces sin siquiera preguntar si el usuario está seguro de desear hacerlo. A esto se le conoce como **referencia directa a objeto insegura** o en inglés *Insecure Direct Object Reference* o simplemente el acrónimo **IDOR**. Si fuera una API más grande, valdría la pena automatizar el proceso para que el escáner activo las busque por nosotros, pero como son pocas las direcciones que permiten realizar esta operación, esto se hará manualmente.


![](./recursos/2/8-1.png)

De la imagen anterior se deducen las siguientes funciones:

~~~
DELETE /menu/{item_id}
PUT /menu/{item_id}
GET /orders/{order_id}
~~~

Aunque son más de interés aquellas que modifiquen datos en la BB.DD., se harán también las que se hagan con GET. Esto es porque para cuando se haga el escaneo activo, el programa sepa de antemano que hay direcciones que se pueden atacar con solo ponerle el id del recurso.

Se aprovechará la misma carga paga en el cuerpo que se le ha enviado en el momento de hacer la importación del descriptor de la API.

Resultado para DELETE /menu/{item_id}:

![](./recursos/2/8-2.png)

Resultado para PUT /menu/{item_id}

![](./recursos/2/8-3.png)

Resultado para GET /orders/{order_id}

![](./recursos/2/8-4.png)

Como se ha podido ver, ninguna petición ha sido aprobada por falta de autenticación y, por ende, de autorización.

#### 2.4.5. ESCANEO ACTIVO DE LA API

Este es un buen momento para hacer un escaneo activo, ya que hay suficientes recursos a probar, y que el programa ya los conoce de antemano. Pero antes, hay que hacer algunas configuraciones al programa para hacer que el escaneo sea breve, pero efectivo.

##### 2.4.5.1. CONFIGURACIÓN DE OWASP ZAP ANTES DEL ESCANEO ACTIVO

Las configuraciones son las siguientes:

1. Todas las rutas por debajo de http://192.168.2.92:8091 (donde se desplegó la API) estarán en el contexto por defecto (default context):

![](./recursos/2/8-5-1.png)
![](./recursos/2/8-5-2.png)

2. El contexto por defecto tendrá configurada la sección de tecnología (technology) así:

- Para DB, PostgreSQL.
- Para el lenguaje, Python.
- Para OS, Linux.
- La gestión de código fuente (SCM) y el servidor web (WS) se dejaran por defecto activos todos.

![](./recursos/2/8-6.png)

Con esto se reduce la cantidad de peticiones y el tiempo de la prueba, ya que se reduce a una cobertura más precisa con relación a la API.

3. El escáner activo usará la política de escaneo "API". Con esto se reduce aún más el tiempo y la cobertura se vuelve más precisa de cara a la prueba de la API, ya que no será necesario realizar pruebas relacionadas con un navegador web (nodo de cliente).

![](./recursos/2/8-7-1.png)
![](./recursos/2/8-7-2.png)

La cobertura (scope) del escaneo quedará entonces en el contexto por defecto, ya configurado previamente, y el escaneo será recursivo, para que explore todos los enlaces que tiene por debajo (recurse):


![](./recursos/2/8-8.png)

##### 2.4.5.2. PRIMER ESCANEO ACTIVO

Una vez hecho esto, ya el programa estará preparado para el escaneo como se desea. 

**NOTA: Antes de darle "Start Scan" hay que tener una consideración y es el monitor del progreso. Apenas se haga clic, inmediatamente deberá ir al monitor (al parar el cursor encima del siguiente ícono, paraece "Show sacn progress details") y a la pestaña "Response Chart". Si no hace esto, no podrá ver la gráfica completa de lo monitorizado. En breve se explicará esto. Esto parece ser un error del programa, y se podría esperar que la tengan en cuenta para una actualización pronta.**

![](./recursos/2/8-9-0-1.png)

![](./recursos/2/8-9-0-2.png)



A continuación, se muestran dos resúmenes gráficos del escaneo activo:

![](./recursos/2/8-9-1.png)

![](./recursos/2/8-9-2.png)

De la imagen donde está la gráfica de r/s contra tiempo, están graficadas los 5 tipos de respuestas HTTP, que van de 1xx a 5xx. Se pondrá más atención a aquellas que son 2xx (verde), ya que significa que hubo peticiones que se respondieron correctamente y 5xx (rojo), que son aquellas que el servidor no pudo manejar al punto de ocasionarle un error interno.

Para el caso de las 2xx, es para ver si dichas peticiones tienen correspondencia con lo que está autorizado para el usuario invitado:

![](./recursos/2/8-10.png)

De las peticiones que implican modificación (POST), aquellas que respondieron con 2xx simplemente ha sido una notificación para validar el cambio de la contraseña.


Para el caso de las 5xx, es para ver si existe la posibilidad de que haya una vulnerabilidad que podría ser grave, al punto de poder efectuar incluso cambios en el sistema (ejecuciones con privilegios demasiado altos).

![](./recursos/2/8-11.png)

Solo hubo una petición, y no se le ha inyectado ninguna carga paga que implique una inyección de comandos, ni de SQL. Solo parece que no pudo manejar el valor "\u0000" en username, que sería una nota a tener en cuenta para que se mejore la presentación de la API. Se emitirá una alerta de información para que cuando se genere el reporte, aparezca.

##### 2.4.5.3. GUARDADO DEL RESULTADO DEL ESCANEO ACTIVO

Es importante guardar los resultados del escaneo activo, ya que, incluso con persistencia de sesión, no quedarían guardados. Esto podría servir para el análisis futuro de datos que no se alcanzaron a analizar, o por si se pueden analizar con otro programa especializado o que el auditor tenga desarrollado para dicho propósito.

Para guardar el registro de la gráfica de respuestas "Response Chart":

![](./recursos/2/8-11-1.png)

![](./recursos/2/8-11-2.png)

Para guardar el registro del progreso (Scan Progress). Esto se guardará en un archivo separado por tabulaciones:

![](./recursos/2/8-11-3.png)

##### 2.4.5.4. CREACIÓN DE ALERTA MANUAL


En el punto anterior se evidenció un error interno en el servidor porque no fue capaz de manejar correctamente un campo en la api que permite reiniciar la contraseña. Para la generación de una alerta, se siguen los siguientes pasos:

Se crea la nueva alerta:
![](./recursos/2/8-12-1.png)

Se especifica de qué se trata, parámetros y la evidencia.:
![](./recursos/2/8-12-2.png)

En la lista de alertas (Alerts) debe aparecer como una con bandera azul, ya que es a modo de información:
![](./recursos/2/8-12-3.png)

#### 2.4.6. PERSISTENCIA DE LA SESIÓN

Cuando se inició este proyecto, no se había considerado el hecho de persistir la sesión, pero dado a que ya hay mucho trabajo realizado, vale la pena guardar los cambios hechos en una sesión, que seguirá creciendo en la medida que se sigan creando y ejecutando más pruebas, así como alertas para el reporte final.

Paso 1:
![](./recursos/2/8-13-1.png)
Paso 2:
![](./recursos/2/8-13-2.png)
Paso 3:
![](./recursos/2/8-13-3.png)


#### 2.4.7. EXPLORACIÓN DE LA API DESDE UN USUARIO CON ROL "CUSTOMER"

A partir de este momento, se revisará cuál es el alcance de un usuario cliente o "Customer", accediendo como este y explorando todos los enlaces de la API, a ver si se encuentra una abertura hacia una escalada de privilegios.

En el punto 2.4.3. ANÁLISIS DEL RESULTADO LA IMPORTACIÓN DE LA ESPECIFICACIÓN DE LA API se ha creado un usuario de rol "customer", cuando se realizó la importación del descriptor de la API en OpenAPI, cuyos campos y valores respectivos son:

~~~
{"username":"John Doe","phone_number":"John Doe","first_name":"John Doe","last_name":"John Doe","role":"Customer"}
~~~


Para poder realizar las peticiones desde el usuario "John Doe" que es de rol "Customer" usando las herramientas como el escaneo activo o el fuzzer, hay que configurar el programa primero, y se hace siguiendo el siguiente procedimiento:

1. Identificar método de autenticación: esto es crucial, ya que de este depende la estrategia que se diseñará para el punto 2.
2. Crear 2 scripts: uno de autenticación (basado en Authentication) y otro de emisor de mensajes (basado en HTTP Sender).
3. Configurar la sesión actual: la sesión actual tiene un contexto por defecto, que lo único que se le ha hecho es añadirle las direcciones provistas por el descriptor de la API previamente importado en el punto 2.4.2. Se creará un contexto llamado "Customer", que permitirá hacer peticiones desde un usuario "Customer" en este caso llamado "John Doe". Entonces se le añadirán las direcciones de la API, y se configurará la autenticación (Authentication), los usuarios (Users), el usuario forzado (Forced User) y la gestión de la sesión (Session Management).

##### 2.4.7.1. MÉTODO DE AUTENTICACIÓN

Para conocer cómo funciona este método, se revisará en la documentación (/docs o /redoc) cuál es la funcionalidad que permite la autenticación. Si se exploran las API, hay una que se llama "auth" y seguramente debe tener esta funcionalidad, es decir, que pida un usuario y contraseña como argumentos. Para este caso, la que coincide con estas características es la función "token", de tipo "POST":

![](./recursos/2/9.png)

Esta operación se hace manual, utilizando el solicitador de peticiones de OWASP ZAP (Requester). Lo que se hará es enviar una solicitud, utilizando de plantilla la que ya envió por primera vez cuando se importó el descriptor de la API. Ya había lanzado un error, pues las credenciales no coincidían:

![](./recursos/2/10.png)

![](./recursos/2/11.png)

Se aprecia que el cuerpo está codificado como application/x-www-form-urlencoded. Se modificará el cuerpo para poner las credenciales de "John Doe" y para corregir el "grant_type", que puede ser con una cadena de caracteres con valor "password". Se empleará el cuerpo (Body) en forma de tabla (Table) para que sea más cómoda la edición. Una vez editado, al hacer clic en "Send" se tiene la siguiente respuesta:

![](./recursos/2/12.png)

Es como dice en la documentación. Se recibieron 2 cadenas de caracteres (strings), una llamada "access_token" y otra llamada "token_type". Entonces se tiene que el método de autenticación se hace solicitando un token de tipo **"bearer"**, lo cual indica que lo más probable es que se esté utilizando el framework de autenticación **OAuth 2.0**. Sabiendo esto, el plan para el punto 2 es utilizar un script que sea capaz de manejar tokens tipo bearer.



##### 2.4.7.2. CREACIÓN DE SCRIPTS PARA AUTOMATIZAR LA AUTENTICACIÓN Y ENVÍO DE MENSAJES AUTENTICADOS CON BEARER

Esto se hace a través de dos scripts:

1. Está basado en Authentication, y se va a llamar "REST-API-Bearer_auth.js". Este se va a ejecutar cada vez que se haga una solicitud a la API.
2. Está basado en HTTP Sender que se va a llamar "Http-sender".


###### CREACIÓN DE "REST-API-Bearer_auth.js"

Para este, se va a usar de referencia el script "OfflineTokenRefresh.js", que está en el repo "https://github.com/zaproxy/community-scripts", exactamente en la siguiente URL: https://github.com/zaproxy/community-scripts/blob/main/authentication/OfflineTokenRefresh.js

En este repositorio se encuentran scripts muy útiles para este tipo de situaciones, cuyas plantillas ya están predefinidas en el programa.


Se realizaron algunas modificaciones, como en los logs en consola y la edición de los campos, ya que no son los mismos para esta API. Parte del encabezado, así como las funciones "getLoggedInIndicator" y "getLoggedOutIndicator" fueron realizadas por la IA DeepSeek. Con estas dos funciones se le hace saber al programa cuándo el usuario está o no autorizado por acceso (logged in). El script quedó así:

~~~
// The authenticate function will be called for authentications made via ZAP.

// The authenticate function is called whenever ZAP requires to authenticate, for a Context for which this script
// was selected as the Authentication Method. The function should send any messages that are required to do the authentication
// and should return a message with an authenticated response so the calling method.
//
// NOTE: Any message sent in the function should be obtained using the 'helper.prepareMessage()' method.
//
// Parameters:
//		helper - a helper class providing useful methods: prepareMessage(), sendAndReceive(msg), getHttpSender()
//		paramsValues - the values of the parameters configured in the Session Properties -> Authentication panel.
//					The paramsValues is a map, having as keys the parameters names (as returned by the getRequiredParamsNames()
//					and getOptionalParamsNames() functions below)
//		credentials - an object containing the credentials values, as configured in the Session Properties -> Users panel.
//					The credential values can be obtained via calls to the getParam(paramName) method. The param names are the ones
//					returned by the getCredentialsParamsNames() below

/*
 * This script is intended to be used along with  httpsender/AddBearerTokenHeader.js  to
 * handle an OAUTH2 offline token refresh workflow.
 *
 * authentication/OfflineTokenRefresher.js will automatically fetch the new access token for every unauthorized
 * request determined by the "Logged Out" or "Logged In" indicator previously set in Context -> Authentication.
 *
 *  httpsender/AddBearerTokenHeader.js  will add the new access token to all requests in scope
 * made by ZAP (except the authentication ones) as an "Authorization: Bearer [access_token]" HTTP Header.
 *
 * @author Laura Pardo <lpardo at redhat.com>
 */

/*
 * Modificado por Andrés E. Torres H., para probar la API Damn Vulnerable RESTaurant,
 * para el trabajo de grado de la maestría en ciberseguridad con IMF SE, Deloitte y la U. Católica de Ávila.*/

// Acceso directo a funciones importadas:
var HttpRequestHeader = Java.type(
  "org.parosproxy.paros.network.HttpRequestHeader"
);
var HttpHeader = Java.type("org.parosproxy.paros.network.HttpHeader");
var URI = Java.type("org.apache.commons.httpclient.URI");
var ScriptVars = Java.type("org.zaproxy.zap.extension.script.ScriptVars");


function authenticate(helper, paramsValues, credentials) {
	print("[API Auth] Starting authentication...");

  // 1. BUILD THE LOGIN REQUEST
  // Obtener URL desde la interfaz gráfica de las propiedades de la sesión en el programa:
  var loginUrl = paramsValues.get('TargetURL');
  // Construir el cuerpo de la solicitud:
  var requestBody = "grant_type=password";
  requestBody += "&username=" + credentials.getParam('username').replaceAll(' ', '+');
  requestBody += "&password=" + credentials.getParam('password');
  requestBody += "&scope=" + "%22John+Doe%22";
  requestBody += "&client_id=" + "%22John+Doe%22";
  requestBody += "&client_secret=" + "%22John+Doe%22";

  // Mostrar en la consola la URL de acceso del usuario y el contenido a enviar:
  print('Login URL: ' + loginUrl);
  print('Credentials:' + requestBody);

  // Construir mensaje HTTP:
  var msg = helper.prepareMessage();
  msg.setRequestBody(requestBody);
  var RequestHeader = new HttpRequestHeader(
    'POST',
    new URI(loginUrl),
    HttpHeader.HTTP11
  );
  // Configurar encabezado para la solicitud de acceso:
  msg.setRequestHeader(RequestHeader);
  msg.getRequestHeader().setHeader('content-type', 'application/x-www-form-urlencoded');
  msg.getRequestHeader().setHeader('accept', 'application/json');
  msg.getRequestHeader().setContentLength(msg.getRequestBody().length());
  // Enviar mensaje y recibir su respuesta:
  print('Enviado: ' + msg.getRequestHeader() + msg.getRequestBody());
  helper.sendAndReceive(msg);
  print('\n\nRecibido: ' + msg.getResponseHeader());
  print("Auth Response Status: " + msg.getResponseHeader().getStatusCode());

  // 2. EXTRACT THE TOKEN FROM THE JSON RESPONSE
  var responseBody = msg.getResponseBody().toString();
  // Mostrar contenido de la respuesta:
  print("Auth Response Body: " + responseBody);

  try {
    var json = JSON.parse(responseBody);
    // Validar recepción del token:
    var authToken = json.access_token || json.token || json.jwt;
    if (!authToken) {
      throw new Error("Token not found in the expected JSON field.");
    }
    print("Successfully extracted Bearer Token.");
    // Guardar token en una variable global. Esto es para que el script "Http-sender" lo use en sus peticiones de acceso:
    ScriptVars.setGlobalVar("access_token", authToken);
    // 3. RETURN THE VALUE FOR THE AUTHORIZATION HEADER
    return msg;
  } catch (err) {
    print("ERROR: Failed to parse JSON or find token: " + err.message);
    return null;
  }
}

// This function is called during the script loading to obtain a list of the names of the required configuration parameters,
// that will be shown in the Session Properties -> Authentication panel for configuration. They can be used
// to input dynamic data into the script, from the user interface (e.g. a login URL, name of POST parameters etc.)
function getRequiredParamsNames(){
	return ["TargetURL"];
}

// This function is called during the script loading to obtain a list of the names of the optional configuration parameters,
// that will be shown in the Session Properties -> Authentication panel for configuration. They can be used
// to input dynamic data into the script, from the user interface (e.g. a login URL, name of POST parameters etc.)
function getOptionalParamsNames(){
	return [];
}

// This function is called during the script loading to obtain a list of the names of the parameters that are required,
// as credentials, for each User configured corresponding to an Authentication using this script 
function getCredentialsParamsNames(){
  // Solo se requiere el usuario y la constraseña para el acceso:
	return ["username", "password"];
}

// This optional function is called during the script loading to obtain the logged in indicator.
// NOTE: although optional this function must be implemented along with the function getLoggedOutIndicator().
function getLoggedInIndicator() {
  // A successful API call often returns 200/201/204.
  // This regex is a broad indicator of "not an auth error".
  return "\"status\":? ?200|HTTP/1.1 20[0-4]";
}

// Tell ZAP how to know if a request was made without being authenticated.
function getLoggedOutIndicator() {
  // This is critical. Use the exact error your API returns on auth failure.
  return "HTTP/1.1 401|HTTP/1.1 403|\"error\":? ?\"Unauthorized\"";
}
~~~


###### CREACIÓN DE "Http-sender.js"

Para este se usó de referencia AddBearerTokenHeader.js, del repo "https://github.com/zaproxy/community-scripts", ya que como lo sugiere el script original "authentication/OfflineTokenRefresher.js" ahora llamado "REST-API-Bearer_auth.js", se usa en conjunto con este. Algunas cosas se modificaron para acomodarlas a este trabajo, y quedó finalmente así:

~~~
/*
 * This script is intended to be used along with authentication/OfflineTokenRefresher.js to
 * handle an OAUTH2 offline token refresh workflow.
 *
 * authentication/OfflineTokenRefresher.js will automatically fetch the new access token for every unauthorized
 * request determined by the "Logged Out" or "Logged In" indicator previously set in Context -> Authentication.
 *
 *  httpsender/AddBearerTokenHeader.js will add the new access token to all requests in scope
 * made by ZAP (except the authentication ones) as an "Authorization: Bearer [access_token]" HTTP Header.
 *
 * @author Laura Pardo <lpardo at redhat.com>
 */

 /*
 * Modificado por Andrés E. Torres H., para probar la API Damn Vulnerable RESTaurant,
 * para el trabajo de grado de la maestría en ciberseguridad con IMF SE, Deloitte y la U. Católica de Ávila.*/

var HttpSender = Java.type("org.parosproxy.paros.network.HttpSender");
var ScriptVars = Java.type("org.zaproxy.zap.extension.script.ScriptVars");

function sendingRequest(msg, initiator, helper) {
  // add Authorization header to all request in scope except the authorization request itself
  print('Insertando token en solicitud actual...');
  if (initiator !== HttpSender.AUTHENTICATION_INITIATOR && msg.isInScope()) {
    msg
      .getRequestHeader()
      .setHeader(
        "Authorization",
        "Bearer " + ScriptVars.getGlobalVar("access_token")
      );
  }
}

function responseReceived(msg, initiator, helper) {}
~~~


##### 2.4.7.3. CONFIGURAR LA SESIÓN ACTUAL


Finalmente se hará un nuevo contexto llamado "Customer", desde el que se harán peticiones con el usuario "John Doe", que actualmente es "Customer". Para esto, primero se creará el nuevo contexto:

![](./recursos/2/13-1.png)

![](./recursos/2/13-2.png)

![](./recursos/2/13-3.png)

Como buena práctica se va a quitar "Default Context" de la cobertura (scope), y se activará que muestre solo las URL que estén en ella:

![](./recursos/2/14-1.png)

![](./recursos/2/14-2.png)

![](./recursos/2/14-3.png)

Ahora se exportarán las direcciones que están en el "Default Context" y se importarán en el "Customer":

![](./recursos/2/15-1.png)

![](./recursos/2/15-2.png)

![](./recursos/2/15-3.png)

![](./recursos/2/15-4.png)

![](./recursos/2/15-5.png)

Ahora se configurará la autenticación. Deberá ser vía scripts, así:

![](./recursos/2/16-1.png)

Dentro de los scripts escogibles aparecerá el script que se ha creado para solicitar la autorización a través de un token bearer:

![](./recursos/2/16-2.png)

Al momento de cargarlo (clic en load), aparecerá un campo nuevo solicitando la URL objetivo (TargetURL) como obligatoria, ya que en el script se ha definido previamente como campo obligatorio desde esta interfaz gráfica:

![](./recursos/2/16-3.png)

También han aparecido (en verde) los campos de los indicadores de acceso y no acceso.

Ahora queda añadir el usuario con el rol de "Customer", que en este caso será "John Doe". Se forzará a usar este usario para todas las peticiones:

![](./recursos/2/17-1.png)

![](./recursos/2/17-2.png)

![](./recursos/2/17-3.png)

La gestión de la sesión se hará con HTTP:

![](./recursos/2/18.png)

Se vuelve a reconfigurar la tecnología para reducir la prueba a una que vaya al grano respecto a las tecnologías que usa esta API:

![](./recursos/2/19.png)

Y por último, hay que asegurarse de que nada esté por ahora excluido del contexto:

![](./recursos/2/20.png)

A partir de ahora el programa está listo para realizar pruebas automatizadas desde este usuario como si estuviera autorizado (desde su sesión).


##### 2.4.7.4. ESCANEO AUTOMATIZADO

Antes de hacer el escaneo automatizado, hay que habilitar el script que permite enviar las peticiones HTTP envenenadas con el token bearer generado por el script de autenticación:

![](./recursos/2/21-1.png)

Ahora se inicia el escaneo automático, pero antes hay que configurar algunos parámetros:

![](./recursos/2/21-2.png)

En la cobertura (Scope) se hará en el contexto "Customer", con el usuario "John Doe", que sea recursivo (Recurse) y que permita visualizar los ajustes avanzados:

![](./recursos/2/21-3.png)

En los vectores de entrada, se añadirán los de URL Path y HTTP Headers, a todas las peticiones. Hará un poco más lenta la prueba, pero ya se ha cerrado bastante desde la configuración "Technology" de este contexto, en las propiedades de la sesión:

![](./recursos/2/21-4.png)

En las políticas, se usará "API". Enseguida se inicia el escaneo.

**NOTA: no olvidarse abrir el monitor del progreso.**

![](./recursos/2/21-5.png)

###### RESULTADOS

Esta vez fue un escaneo de un poco más de 20 minutos:

![](./recursos/2/22-1.png)

Antes de continuar con el análisis, se va a guardar el progreso (ver 2.4.5.3 para ver el procedimiento), y adicionalmente el historial del escaneo. Se pudo haber hecho en el escaneo anterior también, pero dado los resultados, no fue relevante hacerlo, pero esta vez es recomendable, pues esta información se puede usar para realizar análisis más especializados, por ejemplo correlación de datos:

![](./recursos/2/22-1-1.png)

![](./recursos/2/22-1-2.png)

Esta vez se ven más peticiones respondidas con 2xx, así como con 5xx:

![](./recursos/2/22-2.png)

Además, el programa automáticamente ha detectado vulnerabilidades más serias (banderas rojas):

![](./recursos/2/22-3.png)

###### ANÁLISIS DE RESULTADOS Y ESTRATEGIAS PARA NUEVAS PRUEBAS

###### ALERTAS ROJAS AUTOMÁTICAS
El objetivo por lo pronto es tratar de buscar una escalada en los privilegios, ahora del usuario actual. Por lo pronto se ignorarán las alertas de redirección temporal, pero serán expuestas en el informe final.

- ALERTA ROJA EN ROL ACTUAL DEL USUARIO

En las alertas rojas, hay una asociada a la inyección SQL (SQL injection), y está nada más y nada menos que en la actualización del rol del usuario actual:

![](./recursos/2/23.png)

Se puede ver que la petición no la supo manejar la API, y esto pudo haber pasado por varias razones, pero claramente muestra una posible abertura a algo que se intentó pero que no se sabe si es o no permitido. Se podría imaginar, por ejemplo, que puede resultar ser que el rol "John Doe" no es válido, sabemos que "Customer" sí. Para el caso de un atacante, se intentaría hacer un fuzzing, para revisar si hay algún argumento en cadena de caracteres que sea válido, y tenga un mayor privilegio, por ejemplo que se llame "Administrator" o algo similar. ¿Y si se le pregunta a los desarrolladores qué posibles roles son admisibles? En este caso, se hará así para poder realizar la prueba más rápido, ya que se está simulando un escenario seguro en el que se pueden hacer este tipo de preguntas a ellos.

Para este caso, se revisará a través del código (es como si se le preguntara directamente a ellos), y ver qué roles son posibles.

###### ALERTAS ROJAS MANUALES

Para estas se va a realizar el procedimiento realizado para el usuario invitado, de organizar el historial por el código de respuesta, empezando por analizar los 2xx.

Estas son de especial cuidado, ya que podrían ser falsos positivos (parece que fueron, pero no son) o falsos negativos (parecen mostrar que todo va bien, pero el daño sí se hizo y está oculto todavía). La ventaja que tiene OWASP ZAP es que permite emitir una alerta y etiquetarle el grado de seguridad con la que se juzga un presunto evento. Esto ayudará además a diseñar una prueba manual, que permitiría elevar dicho grado de seguridad en el juicio de la alerta, en caso de que se compruebe.

- DETECCIÓN DE ACEPTACIÓN DE CAMBIOS EN CAMPOS CON CARACTERES QUE NO DEBERÍAN SER VÁLIDOS

Por ejemplo, campos como el nombre de una persona no debería aceptar caracteres extraños como "@", "\", etc:

![](./recursos/2/24.png)

También se descubrió en el número de teléfono (phone number).

- DETECCIÓN DE CAMBIO EN EL CONTENIDO DEL MENÚ

Algo llamó la atención y fue que se borró el id # 1 del contenido del menú. Probablemente pudo haber sido un DELETE del menú. Si el usuario de rol "Customer" no debería estar autorizado para hacer esto, deberá lanzarse una alerta roja. Por lo pronto se hará con seguridad de juicio media (confidence):


![](./recursos/2/25.png)

Para asegurarse de esto, se hará una prueba manual intentando quitar el id # 2 en el punto 2.4.7.5.  ESCANEO MANUAL DE RECURSOS SOSPECHOSOS DE SER VULNERABLES.

También valdría la pena realizar las pruebas manuales que se hicieron desde un usuario invitado, estos son para buscar IDOR.

##### 2.4.7.5. ESCANEO MANUAL DE RECURSOS SOSPECHOSOS DE SER VULNERABLES

Del escaneo automatizado del punto 2.4.7.4, se realizarán a cabo las siguientes pruebas:

- Pruebas de IDOR que se hicieron desde el invitado, pero ahora desde el usuario John Doe (como se hicieron en el punto 2.4.4 EXPLORACIÓN MANUAL DE LAS DIRECCIONES QUE CONTIENEN PARÁMETROS EN LA RUTA DE LA URL EN BÚSQUEDA DE IDOR).
- Intentar cambiar el role con un rol válido al usuario John Doe.

###### PRUEBAS DE IDOR

Se harán para los siguientes valores:
~~~
DELETE /menu/2
PUT /menu/1
GET /orders/1
~~~

Han pasado un poco más de 3 horas desde que se hizo el escaneo activo, y cuando se intentó realizar el escaneo manual apareció la petición para "DELETE /menu/2" como no autorizada. Esto significa que el token bearer tiene un vencimiento menor a este tiempo.

![](./recursos/2/26.png)

Dado que manualmente solo envía el último token realizado automáticamente, se desactivará el script del Http-sender, se solicitará el token, y manualmente se introducirá en las siguientes peticiones.

Probando con "DELETE /menu/1", se tiene como elemento no encontrado:

![](./recursos/2/27.png)


Probando con "DELETE /menu/2" se tiene este resultado:

![](./recursos/2/28.png)

Volviendo a hacer la misma petición, se tiene este resultado:

![](./recursos/2/29.png)

Se marcará como una alerta de alto riesgo como confirmada. Esto es un caso de IDOR, además de que no debería estar autorizado.

Probando con "PUT /menu/1":

![](./recursos/2/30.png)

¿Que pasaría si de trata de modificar el elemento 3 del menú?:

![](./recursos/2/30-1.png)

La acción está prohibida para el usuario actual.

Probando con "GET /orders/1":

![](./recursos/2/31.png)

Esto parece ser otro caso de IDOR. Se emitirá la alerta como de medio riesgo por fuga de información, y por lo pronto como seguridad de juicio media. Para esto se hará una prueba haciendo un POST a la creación de una orden. Para ser un usuario "Customer" debería poder realizarse (en la vida real a través de la validación de un pago, de lo contrario, debería ser la responsabilidad de alguien interno).

Se encontró que no hay suficiente información en /docs ni en /redoc de cómo debería llenarse el campo "items":

![](./recursos/2/32.png)

Se marcará como una alerta informacional para que la documentación quede clara para este campo.

![](./recursos/2/33.png)

Para esto, se revisó el código (el equivalente a preguntarle a los desarrolladores), y se hizo la petición siguiente, con el siguiente resultado:

![](./recursos/2/34.png)

Respecto de la última vez que se solicitó el último token, ha pasado media hora, y probablemente ese es el tiempo que dura cada uno. Al solicitar uno nuevo y de nuevo hacer el POST en orders, se tiene lo siguiente:

![](./recursos/2/35.png)

Ahora se probarán los GET de orders/ y /orders/1.


![](./recursos/2/36.png)

![](./recursos/2/37.png)


El usuario "Customer" puede tener acceso a todas las órdenes del restaurante, así como una por una. Para ambos casos se les generó una alerta independiente y esto es porque:

GET /orders permite visualizar todas las órdenes del restaurante, algo que no debería permitírsele al usuario "Customer". Esto es información interna de la operación de este.

GET /orders/{order_id} es un caso de IDOR. Lo que se debería hacer es poderse acceder a este, siempre y cuando tenga alguna relación con el usuario actual, además de que el recurso debería estar enmascarado.

###### PRUEBAS PARA EL CAMBIO DE ROL DEL USUARIO

Estas podrían explotar la vulnerabilidad de una escalada de privilegios. Lo que se intentará es tratar de cambiar el rol a uno con privilegios más altos. Como se desconocen sus posibles valores (no están documentados), se hará como si se le preguntara a los desarrolladores.

En la API, en db/models.py se encuentran los posibles valores:

![](./recursos/2/38.png)

Un atacante habría podido usar fuzzing con valores predeterminados, y sin con suerte tiene algunos que tengan mayores privilegios, seguramente podría hacer algo. Pero para acelerar el proceso, se evaluará la posibilidad de cambiar el rol siendo "Customer" a uno más elevado. Pero esta sería una buena oportunidad de probar el fuzzer de OWASP ZAP. Se pondrán como valores predeterminados a "Chef" y "Employee" a ver que sucederá:

![](./recursos/2/39.png)

Hay que añadir un nuevo token, porque ya seguro debe estar vencido:

![](./recursos/2/40.png)

![](./recursos/2/41.png)

Aquí se va a configurar el veneno de la carga paga, en la parte que interesa que es la del rol (role):

![](./recursos/2/42.png)

Se añaden los valores venenosos:

![](./recursos/2/43.png)

![](./recursos/2/44.png)

![](./recursos/2/45.png)

Una vez configurado el veneno, se ejecuta el fuzzer:
![](./recursos/2/46.png)

Y como se puede observar, se ha podido realizar una escalada en los privilegios, desde "Customer" a "Employee", desde el usuario "Customer". Se notificará la alerta.

![](./recursos/2/47.png)

Cualquier cosa que se haga automatizada ahora, o que se haga desde este usuario tendrá la autorización de un empleado (Employee). Por eso, en la próxima sección 2.4.8. EXPLORACIÓN DE LA API DESDE UN USUARIO CON ROL "EMPLOYEE" se ejecutarán las mismas pruebas realizadas en esta sección de rol "Customer".

#### 2.4.8. EXPLORACIÓN DE LA API DESDE UN USUARIO CON ROL "EMPLOYEE"

En la última parte de la sección anterior se pudo realizar una escalada en los privilegios y ahora se está en un usuario con rol de empleado (Employee), que por intuición, debería tener privilegios mucho más altos, que permita realizar ciertas operaciones adicionales que no se podían hacer en los roles anteriores.

Las pruebas que se harán tendrán el mismo orden de las de "Customer":

- 1. Escaneo automatizado. Se hará sobre el mismo contexto "Customer".
- 2. Escaneo manual de recursos sospechosos de ser vulnerables.

##### 2.4.8.1. ESCANEO AUTOMATIZADO

Este escaneo tardó casi media hora:

![](./recursos/2/48.png)

Se puede visualizar que hubo muchas respuestas 2xx así como 5xx:

![](./recursos/2/49.png)

###### RESULTADOS

No se encontraron resultados significativos, pero hay algunos para considerar, como el hecho de que los empleados no puedan cambiar sus contraseñas. Ya se ha generado la alerta, como riesgo medio y confirmada.


##### 2.4.8.2. ESCANEO MANUAL DE RECURSOS SOSPECHOSOS DE SER VULNERABLES


Se intentará cambiar el rol del usuario a "Chef":

![](./recursos/2/50.png)

Aunque manualmente se hizo el cambio del token y se intentó pasar a "Chef", este rol de usuario no es capaz de permitir ese cambio, lo cual está bien.

A partir de este punto se van a hacer unas pruebas en las que hay vulnerabilidades en la API que requieren técnicas más sofisticadas de explotación, y que están explicadas en la sección de tests/vulns del juego (API):

![](./recursos/2/51.png)

Con las técnicas utilizadas hasta ahora se han podido descubrir 4 de ellas (level_0 hasta level_3). Para el caso de level_4, es una *falsificación de petición del lado del servidor* (Server Side Request Forgery o SSRF). Se espera que con este ataque se pueda escalar al rol de "Chef". Pero antes se va a revisar en la documentación de la API el porqué podría presentarse esta vulnerabilidad. Y para el nivel 5, se intentará ejecutar algún comando en el SO en el que está ejecutándose la API.

###### PRUEBA DE FALSIFICACIÓN DE PETICIÓN DEL LADO DEL SERVIDOR (SSRF)

Para que pueda haber una SSRF, la API debe cumplir con un requisito de sospecha y este es que haya al menos una función de la API que el usuario puede realizar que implique que el servidor deba hacer una petición a otra dirección, de manera que la respuesta de esa petición quede incrustada en la respuesta original realizada por el usuario. Gráficamente, se vería así:

![](./recursos/2/52.png)

Aunque el mismo POST habría podido responder también directamente con la respuesta incrustada si la API realizara esta tarea internamente.

Analizando la documentación, hay una función que hace sospechar la posibilidad de que esta sea vulnerable a SSRF:

~~~
PUT menu/
request body: {
  "name": "string",
  "price": 0,
  "category": "string",
  "image_url": "string",
  "description": "string"
}
response body cuando es 201:
{
  "id": int,
  "name": "string",
  "price": float,
  "category": "string",
  "description": "string",
  "image_base64": "string"
}
~~~

Si se hace una hipotética prueba de escritorio de cómo funciona esta petición, la respuesta sería así:

- id: la API o la BB.DD. la asignaría automáticamente.
- name: la sacaría de "name" de la petición directamente.
- price: la sacaría de "price" de la petición directamente.
- category: la sacaría de "category" de la petición directamente.
- description: la sacaría de "description" de la petición directamente.
- image_base64: o convierte la "url" de la petición en base64, o **hace una solicitud http a la URL, coge la respuesta y la transforma en base64**.

La última parte está resaltada en negrilla porque es la que la haría vulnerable a una SSRF.

Las pruebas se harán manualmente desde el usuario con rol "Employee" a ver qué respuestas se tienen. Por último se sospecha de esta función de la API:

~~~
GET /admin/reset-chef-password"
~~~

Aunque está oculta en la documentación, y se intentó hacer un arañazo (spidering), no pudo conseguirse por ninguno de estos medios.

![](./recursos/2/53.png)


Pero esto es algo que se habría podido conseguir en el momento de una auditoría, en el caso hipotético de que por políticas de ciberseguridad se tengan que cambiar las contraseñas cada cierto tiempo (incluida la del chef), preguntar si la API es capaz de permitir eso, y cómo se haría.

Con las indagaciones hechas hasta ahora, se puede plantear con seguridad un vector de ataque para realizar una SSRF.

El plan será así:

1. Manualmente enviar la siguiente solicitud y esperar a ver que responde:

PUT menu/
request body: {
  "name": "string",
  "price": 0,
  "category": "string",
  "image_url": "http://localhost:8091/admin/reset-chef-password",
  "description": "string"
}

Con la esperanza de que la API haga una solicitud GET a esa URL (http://localhost:8091/admin/reset-chef-password), ver qué responde. Lo más probable es que devuelva una nueva contraseña aleatoria codificada en base64.

**NOTA: no olvidar desactivar el script de Http-sender.**

2. Coger lo que vino en el campo image_base64 y decodificarlo. Esta debería ser la nueva contraseña del chef.

3. Intentar solicitar un token del usuario Chef.

4. En caso de que el paso 3 responda con un token, verificar que el usuario sí es el chef usando:

~~~
GET /profile
~~~

###### RESULTADO


1. Se obtuvo la siguiente respuesta:

![](./recursos/2/54.png)


~~~
{
  "id":15,
  "name":"string",
  "price":0.0,
  "category":"string",
  "description":"Zaproxy alias impedit expedita quisquam pariatur exercitationem. Nemo rerum eveniet dolores rem quia dignissimos.",
  "image_base64":"eyJwYXNzd29yZCI6IltWPUpQQll5SiFSdzQ6NTFMSk0kaigxOjdVdl5RPUJzIn0="}
~~~

2. Al decodificar "image_base64" se tiene este resultado:

![](./recursos/2/55.png)

~~~
{"password":"[V=JPBYyJ!Rw4:51LJM$j(1:7Uv^Q=Bs"}
~~~

Se generará una alerta de alto riesgo y alta seguridad en el juicio de que si haya sido probable una SSRF.

3. AL intentar solicitar un token con las credenciales:

~~~
{
  "username": "chef",
  "password": [V=JPBYyJ!Rw4:51LJM$j(1:7Uv^Q=Bs
}
~~~

Se obtuvo el siguiente resultado:

![](./recursos/2/56.png)


4. Hasta ahora parece que sí se ha realizado la SSRF con éxito. Para comprobar que el usuario actual que usa este token es el chef, se usará la función GET /profile:

![](./recursos/2/57.png)

El ataque ha sido un éxito, permitiendo una nueva escalada de privilegios al usuario "Chef". Se generará otra alerta con riesgo alto y seguridad en el juicio confirmada (es 100 % seguro de que sí sucedió) relacionada con la recuperación de contraseñas, para este caso, la del chef.

#### 2.4.9. EXPLORACIÓN DE LA API DESDE UN USUARIO CON ROL "CHEF"

En la última parte de la sección anterior se pudo realizar una escalada en los privilegios hasta llegar a un usuario con rol "Chef" que por intuición, debería tener privilegios ahora mucho más altos, que permita realizar ciertas operaciones adicionales que no se podían hacer en los roles anteriores.

Las pruebas que se harán tendrán el mismo orden de las de "Customer" y "Employee":

- 1. Escaneo automatizado. Se hará sobre el mismo contexto "Customer".
- 2. Escaneo manual de recursos sospechosos de ser vulnerables.

##### 2.4.9.1. ESCANEO AUTOMATIZADO

Antes de hacer el escaneo automatizado, hay que habilitar el script que permite enviar las peticiones HTTP envenenadas con el token bearer generado por el script de autenticación.

Al contexto actual hay que agregar un nuevo usuario, este será el chef:

![](./recursos/2/58.png)

Y se fuerza el usuario "chef":

![](./recursos/2/59.png)

Ahora se inicia el escaneo automático, pero antes hay que configurar algunos parámetros:

En la cobertura (Scope) se hará en el contexto "Customer", con el usuario "Chef", que sea recursivo (Recurse) y que permita visualizar los ajustes avanzados:

![](./recursos/2/60.png)


En los vectores de entrada, se añadirán los de URL Path y HTTP Headers, a todas las peticiones. Hará un poco más lenta la prueba, pero ya se ha cerrado bastante desde la configuración "Technology" de este contexto, en las propiedades de la sesión:

![](./recursos/2/61.png)


En las políticas, se usará "API". Enseguida se inicia el escaneo.

Durante la prueba aparecía un error de autenticación en el script "REST-API-Bearer_auth.js", esto pudo ser porque durane la prueba se pudo haber restablecido la contraseña. Sin embargo, conserva el mismo último token. Lo que se hará es detener la prueba pasado los 30 minutos, que es lo que dura el token válido. Luego se hará otra con los parámetros faltantes hasta completar todas las pruebas asociadas a la política "API".

![](./recursos/2/62.png)

Pasado los 30 minutos, se vuelve a hacer la SSRF que permite restablecer la contraseña. Para decodificar base64, también se puede usar Python así, en caso de no tener o no querer depender de Internet:

![](./recursos/2/63.png)

Como se había planificado, pasada la media hora debe detenerse la prueba manualmente:

![](./recursos/2/64.png)

Para la próxima prueba, se hará una nueva política basada en "API" que se va a llamar "API2", en la que se excluirán las pruebas a excepción de la última (Remote OS Command Injection), porque se estaba haciendo en el momento que se agotó el tiempo:

![](./recursos/2/65.png)

![](./recursos/2/66.png)

![](./recursos/2/67.png)

![](./recursos/2/68.png)

![](./recursos/2/69.png)

![](./recursos/2/70.png)

Hubo 13 alertas hasta aquí, pero estaban asociadas a redicreción externa y a inyección SQL.

A continuación, los resultados de la 2ª iteración:

![](./recursos/2/71.png)

Aquí llama la atención que hubo 2 alertas asociadas a ejecución remota de comandos en el sistema operativo:

![](./recursos/2/72.png)

![](./recursos/2/73.png)

Al revisar el ataque automatizado que hizo ZAP usando la inyección "cat /etc/passwd", un comando en Linux para almacenar informción esencial acerca de las cuentas de usuario. Al compararlo contra el del contenedor que ejecuta el servicio de la API, se puede apreciar que tienen el mismo contenido, lo cual es un caso certero de *ejecución remota en el sistema operativo*.


##### 2.4.9.2. ESCANEO MANUAL DE RECURSOS SOSPECHOSOS DE SER VULNERABLES

Como se han alcanzado todas las banderas, no hay más procedimientos para hacer manualmente.



# 3. CONCLUSIONES

Las alertas se emitirán en un reporte que OWASP ZAP es capaz de generar así:

![](./recursos/2/74.png)

![](./recursos/2/75.png)

![](./recursos/2/76.png)


Como es un reporte relativamente extenso, se adjuntará como anexo a este trabajo con el nombre "2025-10-01-ZAP-Report-.html", dentro de la carpeta comprimida "RESTaurant.zip".