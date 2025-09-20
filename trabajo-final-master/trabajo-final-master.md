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

Como es un ejercicio de una API pequeña (ver su documentación en el README.md en su repo: https://github.com/theowni/Damn-Vulnerable-RESTaurant-API-Game.git), con un computador con la siguiente configuración es suficiente:

- Hardware: procesador Intel Core i7 10750H, 32 GB de RAM y 240 GB de almacenamiento.
- SO anfitrión: Ubuntu 24.04, con docker y docker compose.
- Máquina virtual con Kali Linux, que ya trae OWASP ZAP, que corre en VirtualBox v7.1.12
- El computador estará conectado a una red LAN gobernada por un router, que le asignará las direcciones IP dinámicamente tanto al anfitrión como a la MV.

### 1.2. ESTRATEGIA DE LA PRUEBA

1. Preparación de la prueba. Se despliega la API con docker compose en el anfitrión. En el mismo computador, se ejecuta una MV con Kali Linux, que ya trae OWASP ZAP.
2. Se hace una exploración manual usando los navegadores Google Chrome y Mozilla Firefox apuntando a unas direcciones definidas.
3. Para el caso de encontrar documentación (Swagger por ejemplo), se intentará descargar la especificación OpenAPI. Si no la tuviere o no se pudiere encontrar, entonces lo que se haría en la vida real sería preguntar sobre el descriptor de la API en un formato estandarizado (OpenAPI).
4. Escaneos automatizados y manuales con OWASP ZAP.
5. Explotación de vulnerabilidades encontradas en el punto 4 y si la explotación lo permite, iterar desde el punto 4 para escalar privilegios en la misma API (tipo de usuario).


## 2. EJECUCIÓN DE LA ESTRATEGIA DE LA PRUEBA

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

### 2.2.1. EXPLORACIÓN MANUAL CON MOZILLA FIREFOX

"/":

!["/"](./recursos/1-1.png)

"/admin":

!["/admin"](./recursos/1-6.png)

"/docs":

!["/docs"](./recursos/1-7.png)

"/redoc":

!["/redoc"](./recursos/1-8.png)

### 2.2.2. EXPLORACIÓN MANUAL CON GOOGLE CHROME

"/":

!["/"](./recursos/1-2.png)

"/admin":

!["/admin"](./recursos/1-3.png)

"/docs":

!["/docs"](./recursos/1-4.png)

"/redoc":

!["/redoc"](./recursos/1-5.png)


### 2.2.3. CONCLUSIÓN DE EXPLORACIÓN MANUAL

1. Se pudo obtener la documentación del uso de la API a través de los enlaces /docs y /redoc. Además se puede obtener la especificación OpenAPI, que será muy útil para la sección 2. EXPLORACIÓN CON HERRAMIENTA DE ESCANEO AUTOMATIZADO de este trabajo.

2. Llama la atención aquí es que Firefox incluye dos barras de opciones adicionales para poder interactuar mejor con la API (ver "/" y "/admin").


## 2.3. EXPLORACIÓN CON HERRAMIENTA DE ESCANEO AUTOMATIZADO

Para este trabajo se empleará OWASP ZAP v2.16.0.

### 2.1. CONFIGURACIÓN DE OWASP ZAP

Se dejará activado el escáner pasivo, ya que permite realizar una auditoría de forma más rápida y controlada, esto es que en la medida que se avanza manualmente, el escáner pasivo hará tareas en segundo plano para detectar posibles vulnerabilidades. La configuración de este quedará así para este trabajo:

### 2.1.1. PASSIVE SCANNER

#### Passive Scan Rules:

Se dejarán todos los valores por defecto, y esto es:

- Threshold = Medium para todos los tests, no importando el status (beta, alpha o release).

!["passive scan rules"](./recursos/2/1.png)

#### Passive scan tags:

Se dejarán todos los valores por defecto, y esto es:

- Todas habilitadas (enabled).

!["passive scan tags"](./recursos/2/2.png)

#### Passive scanner:

Se dejarán todos los valores por defecto, y esto es:

- Se ecanearán todos los mensajes (incluso fuera de la cobertura (scope)).
- No incluirá tráfico del fuzzer durante escaneo pasivo.
- 3 hilos para escaneo pasivo.
- Alertas y tamaño del cuerpo en bytes para escanear sin límites.

!["passive scanner"](./recursos/2/3.png)

### 2.1.2. MODO DE ZAP

Se trabajará en modo estándar (Standard Mode).

### 2.1.3. VALUE GENERATOR

Esta configuración es importante también, porque a pesar de que se puede dejar por defecto, añadirle valores puede ayudar a mejorar la auditoría, haciéndola personalizada en las búsquedas y ataques. El add-on OpenAPI, que se usará en el punto 2.2, la usa durante la importación de una definición, otra razón más para configurarla correctamente. Algunos campos que no estén definidos serán rellenados por valores que tiene este generador por defecto en ZAP, por ejemplo "John Due".

Además de los que están por defecto, se pondrá uno adicional llamado "id" con valor "1".

!["value generator id"](./recursos/2/4.png)


### 2.2 IMPORTACIÓN DE OPENAPI.JSON A ZAP

Como la documentación permite obtener este documento y ZAP permite su importación, se realizará para agilizar los ataques. Durante la auditoría de una API, esto es un requisito mínimo para poder realizarla.

![](./recursos/2-1.png)

![](./recursos/2-2.png)

![](./recursos/2-3.png)

![](./recursos/2-4.png)

![](./recursos/2-5.png)


### 2.3. ANÁLISIS DE 







### 2.4. CONCLUSIÓN DE ESCANEO AUTOMATIZADO

- Se encontró que se puede conocer las tecnologías que usa el servicio web, en "/healthchecker".
- En 4 rutas se encontró que la cabecera de X-Content-Type-Options está ausente.
- En la ruta /menu, es posible acceder al menú sin estar autenticado. Puede que sea a propósito para que cualquiera pueda visualizar la página.

## 3. EXPLORACIÓN MANUAL

Se explorarán las ubicaciones dadas en la documentación, realizando consultas y solicitudes HTTP (GET, POST, PUT, DELETE, etc), desde las credenciales con privilegios más bajos (invitado), verificando si hay forma de hacer acciones que no deberían poder hacerse, e intentando hacer ataques de instrusión para buscar acceder al sistema y escalar los privilegios actuales. En caso de que no se haga, se irán explorando usuarios poco a poco hacia niveles más altos para verificar coherencia con sus permisos y buscando lo mismo para el caso del plan ya descrito para el usuario de privilegio de menor importancia.

Para esta exploración de emplea OWASP ZAP v2.16.0.

### 3.1. EXPLORACIÓN DE ENLACES DE UN USUARIO SIN PRIVILEGIOS

Esto es como si fuera un invitado el que estuviera explorando la API.

Se encontró que:


~~~
POST http://192.168.1.47:8091/register
~~~

Es capaz de crear un usuario desde invitado (ningún usuario accedido).




## 4. SSRF

1. Ir a docs. Para mi caso, el servidor está lanzado en la siguiente dirección:

~~~
http://192.168.2.92:8091/docs
~~~

y buscar en la sección "menu" el método "put" para actualizarlo.

2. Intentar ejecutar el método, así se obtiene el comando para curl:

~~~
curl -X 'PUT' \
  'http://localhost:8091/menu' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "name": "string",
  "price": 0,
  "category": "string",
  "image_url": "string",
  "description": "string"
}'
~~~

3. Preparar el comando curl de manera que apunte al servidor y además tenga un token válido como empleado (employee).

El token se obtiene así, para el usuario y contraseña creados en el 

~~~
curl -X 'POST' \
  'http://192.168.2.92:8091/token' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=password&username=customer&password=password&scope=&client_id=string&client_secret=string'
~~~

~~~
curl -X 'PUT' \
  'http://192.168.2.92:8091/menu' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "name": "string",
  "price": 0,
  "category": "string",
  "image_url": "string",
  "description": "string"
}'
~~~


