# TP 08 – Endpoints persistentes de reservas de salas

Trabajo práctico de la Unidad 8 – Desarrollo Back-end con Node.js.

## Índice

1. [Descripción y objetivo](#1-descripción-y-objetivo)
2. [Origen del proyecto](#2-origen-del-proyecto)
3. [Versiones y dependencias](#3-versiones-y-dependencias)
4. [Configuración de MongoDB Atlas](#4-configuración-de-mongodb-atlas)
5. [Instalación y ejecución](#5-instalación-y-ejecución)
6. [Estructura del proyecto](#6-estructura-del-proyecto)
7. [Modelo y validaciones](#7-modelo-y-validaciones)
8. [Rutas y endpoints](#8-rutas-y-endpoints)
9. [Pruebas de la API con Postman](#9-pruebas-de-la-api-con-postman)
10. [Matriz de pruebas](#10-matriz-de-pruebas)
11. [Comprobación de persistencia](#11-comprobación-de-persistencia)
12. [Continuidad del sitio HTML y manejo de errores](#12-continuidad-del-sitio-html-y-manejo-de-errores)
13. [Alcance y límites de la validación](#13-alcance-y-límites-de-la-validación)
14. [Diferencias con el TP 07](#14-diferencias-con-el-tp-07)
15. [Calidad y lista de entrega](#15-calidad-y-lista-de-entrega)
16. [Referencias](#16-referencias)

---

## 1. Descripción y objetivo

Este proyecto corresponde al Trabajo Práctico 08 de la Unidad 8 del curso de Desarrollo Back-end con Node.js.

El objetivo es migrar la creación, el listado y la consulta de reservas desde una implementación en memoria/JSON a **MongoDB Atlas**, conservar el sitio HTML existente y agregar una **API REST que responde en JSON**.

El sitio web y la API deben utilizar el mismo modelo, servicio y base de datos. Los datos no deben cargarse automáticamente desde una semilla al iniciar el servidor ni al consultar el listado.

| Dato | Descripción |
|---|---|
| Nombre sugerido | `tp-08-salas-http-mongodb` |
| Base de datos Atlas | `reservas_salas_u8` |
| Colección | `reservas` |
| Modalidad | Individual |
| Entrega | URL del repositorio GitHub |

**Estado de las pruebas:** los resultados esperados se documentan en este README. Los ID reales, las cantidades y las evidencias que dependen de la conexión local con Atlas deben completarse después de ejecutar las pruebas. No se inventan identificadores ni resultados de persistencia.

## 2. Origen del proyecto

El punto de partida es una **copia funcional e individual del TP 07**. Se debe conservar la entrega anterior y trabajar en una carpeta o repositorio separado.

Al preparar la copia:

- No copiar el historial Git (`.git`).
- No copiar `node_modules`.
- No copiar el archivo privado `.env`.
- Conservar `package.json`, `package-lock.json`, la configuración de calidad y las versiones existentes.
- Mantener la estructura por capas de la entrega anterior.
- Adaptar las operaciones de reservas para que sean asíncronas.

**Procedencia concreta de esta copia:** completar con la URL o el nombre del repositorio TP 07 utilizado: `PENDIENTE`.

## 3. Versiones y dependencias

El proyecto debe conservar las versiones declaradas en el manifiesto y en el lockfile de la copia del TP 07. No se deben actualizar todas las dependencias como parte de esta migración.

Para registrar las versiones reales del entorno, ejecutar desde la raíz del proyecto:

```bash
node --version
npm --version
npm ls express mongoose
```

| Componente | Versión / estado |
|---|---|
| Node.js | Completar con `node --version` |
| npm | Completar con `npm --version` |
| Express | Completar con `npm ls express` |
| Mongoose | Completar con `npm ls mongoose` |
| MongoDB Atlas | Clúster Free (M0), según la consigna |
| Modo de módulos | CommonJS (`require` / `module.exports`) |
| Lockfile | `package-lock.json`, conservar en el repositorio |

El `package.json` declara Node.js `>=20.19.0`, requerido por Mongoose `9.10.3` según el lockfile. Verificar la versión instalada antes de ejecutar la instalación. No indicar que una versión fue comprobada en el equipo si no se ejecutaron los comandos correspondientes.

## 4. Configuración de MongoDB Atlas

### 4.1. Preparar Atlas

1. Ingresar a MongoDB Atlas y utilizar el clúster de práctica Free (M0), con acceso autorizado.
2. Configurar un usuario de base de datos.
3. Autorizar en Network Access la dirección IP desde la que se ejecutará el proyecto.
4. Obtener la URI de conexión del clúster.
5. Incorporar el nombre de la base `reservas_salas_u8` en la URI.
6. Verificar la conexión antes de iniciar la aplicación.

La base y la colección pueden aparecer en Atlas cuando se guarde el primer documento. No es necesario insertar una semilla desde el panel. No borrar ni migrar silenciosamente los datos del TP 07.

### 4.2. Variables de entorno

Crear localmente un archivo `.env` a partir de la plantilla `.env.example`:

```dotenv
PORT=3000
NODE_ENV=development
MONGODB_URI=mongodb+srv://USUARIO:CLAVE@HOST/reservas_salas_u8?retryWrites=true&w=majority
```

`USUARIO`, `CLAVE` y `HOST` son marcadores ilustrativos. Reemplazarlos únicamente en el archivo local por los datos reales de Atlas. Si la contraseña contiene caracteres especiales, codificarlos correctamente para una URI.

El archivo `.env` contiene secretos y **no debe subirse a GitHub**. La plantilla `.env.example` sí se entrega, pero debe contener solamente marcadores, nunca credenciales reales. Verificar que `.gitignore` excluya `.env` y `node_modules`.

### 4.3. Entorno y ajuste DNS

La aplicación debe leer la configuración del entorno y establecer la conexión con MongoDB antes de abrir el puerto HTTP.

La clase indica conservar en `src/config/base-datos.js` la corrección DNS con `node:dns`, cuando corresponda:

```js
dns.setServers(["8.8.8.8", "1.1.1.1"]);
```

Este ajuste debe utilizarse **solo si está autorizado y funciona en la red utilizada**. Completar el estado real:

- Ajuste DNS aplicado: `PENDIENTE: sí / no`.
- Motivo o diagnóstico: `PENDIENTE`.
- Prueba de conexión: `PENDIENTE: comando y resultado observado`.

Cambiar los resolutores no soluciona credenciales incorrectas, una IP no autorizada, una URI inválida ni la falta de conectividad. No duplicar el ajuste DNS por solicitud. Ante un fallo de conexión al inicio, el servidor no debe anunciar que está disponible.

## 5. Instalación y ejecución

Desde la carpeta raíz del proyecto, instalar las dependencias:

```bash
npm ci
```

Ejecutar los controles disponibles en `package.json`:

```bash
npm run format
npm run check
```

Si esos scripts no existen en la copia, revisar los nombres definidos en `package.json` y documentar los comandos reales.

Iniciar cargando el archivo local de entorno:

```bash
npm run start:local
```

Este script ejecuta `node --env-file=.env src/index.js`. El comando `npm start` ejecuta el servidor sin cargar `.env` automáticamente y requiere que las variables ya estén disponibles en el entorno.

Con `PORT=3000`, la aplicación queda disponible en:

```text
http://localhost:3000
```

La conexión a Atlas debe completarse antes de ejecutar `listen`. El servidor mantiene la conexión durante las solicitudes y contempla un cierre ordenado al recibir `SIGINT` o `SIGTERM`. No iniciar dos servidores simultáneamente sobre el mismo puerto.

## 6. Estructura del proyecto

La estructura exacta debe coincidir con los archivos que existan en el repositorio. La organización prevista por la consigna es:

```text
tp-08-salas-http-mongodb/
├── src/
│   ├── index.js
│   ├── app.js
│   ├── config/
│   │   ├── entorno.js
│   │   └── base-datos.js
│   ├── modelos/
│   │   └── reserva.js
│   ├── servicios/
│   │   └── reservas-mongo.js
│   ├── controladores/
│   │   └── reservas.js
│   ├── rutas/
│   │   ├── reservas.js
│   │   └── reservas-api.js
│   └── middleware/
│       ├── reservas.js
│       ├── solicitudes.js
│       └── errores.js
├── scripts/
├── views/
├── public/
├── datos/                 # referencia anterior, si se conserva
├── .env.example
├── .gitignore
├── package.json
├── package-lock.json
└── README.md
```

### Responsabilidad de cada capa

| Archivo o carpeta | Responsabilidad |
|---|---|
| `src/index.js` | Lee el entorno, conecta con Atlas, crea el servicio y la aplicación, inicia el servidor y gestiona su cierre. |
| `src/app.js` | Configura Express, vistas, recursos estáticos, parsers, middleware, routers, la ruta `/estado`, 404 y el manejador de errores. |
| `src/config/` | Centraliza la configuración del entorno y la conexión con MongoDB. |
| `src/modelos/reserva.js` | Define los campos y las reglas del esquema de Mongoose. |
| `src/servicios/reservas-mongo.js` | Ejecuta consultas y escrituras asíncronas. No define rutas HTTP ni envía respuestas. |
| `src/controladores/reservas.js` | Espera las operaciones del servicio y decide qué respuesta HTML o JSON devolver. |
| `src/rutas/` | Separa las rutas del sitio HTML de los endpoints de la API JSON. |
| `src/middleware/` | Valida la entrada, prepara datos de solicitud y procesa errores. |
| `scripts/` | Conserva herramientas de diagnóstico o pruebas de terminal. No se importa un script de ejecución puntual desde el servidor. |
| `views/` y `public/` | Mantienen las plantillas HTML, los estilos y los recursos estáticos. |

El servicio anterior en memoria puede conservarse como referencia, pero debe dejar de importarse en el arranque persistente. El JSON anterior también puede conservarse como referencia, pero no debe cargarse automáticamente. La aplicación no debe mantener dos fuentes activas para el mismo conjunto de reservas.

## 7. Modelo y validaciones

El recurso conserva los seis campos del TP 07 y agrega un estado predeterminado.

| Campo | Regla de entrada |
|---|---|
| `estudiante` | String obligatorio; se recortan los espacios externos (`trim`). |
| `email` | String obligatorio; debe contener `@` y se recortan los espacios externos. |
| `sala` | String obligatorio; valores permitidos: `Sala Norte`, `Sala Sur` o `Sala Multimedia`. |
| `fecha` | String obligatorio; se recortan los espacios externos (`trim`). |
| `turno` | Valor permitido: `Mañana`, `Tarde` o `Noche`. |
| `personas` | Número entero obligatorio entre 1 y 6. |
| `estado` | Valor predeterminado `pendiente`; opciones del esquema: `pendiente` o `confirmada`. El cliente no lo asigna al crear. |

En la creación, copiar explícitamente solo los seis campos admitidos. No pasar `req.body` completo a Mongoose. Los campos adicionales, como `estado` o `_id`, enviados por el cliente no deben controlar el documento creado.

El identificador público `id` es la representación textual del `ObjectId` de MongoDB. No convertir los parámetros de ID con `Number()`.

La validación debe rechazar cuerpos ausentes, texto sin parser, entradas vacías y tipos incorrectos. En particular, `personas` no debe aceptar booleanos, arreglos u objetos. Se deben mantener las validaciones del modelo, no solamente las del middleware.

## 8. Rutas y endpoints

Dirección base local: `http://localhost:3000`

### 8.1. Sitio HTML

| Método | Endpoint | Propósito | Resultado esperado |
|---|---|---|---|
| GET | `/` | Página de inicio | `200` y HTML |
| GET | `/reservas` | Listado HTML de reservas | `200` y HTML |
| GET | `/reservas/nueva` | Formulario HTML | `200` y HTML |
| POST | `/reservas` | Crear desde el formulario | `302` hacia el listado si guarda; `400` si la entrada es inválida |

### 8.2. API JSON

| Método | Endpoint | Propósito | Resultado esperado |
|---|---|---|---|
| GET | `/estado` | Estado del servicio y conteo persistente | `200` y JSON con servicio, reservas y solicitud |
| GET | `/api/reservas` | Listar reservas | `200` y arreglo JSON; `[]` es válido |
| GET | `/api/reservas/:id` | Consultar una reserva | `200`; `400` si el ID está mal formado; `404` si el ID válido no existe |
| POST | `/api/reservas` | Crear una reserva desde JSON | `201`, recurso guardado y cabecera `Location`; `400` si la entrada es inválida |
| GET | `/api/reservas?sala=Sala%20Norte` | Filtrar por coincidencia exacta de sala | `200` con coincidencias o `[]`; `400` si el parámetro está repetido |

Para rutas API desconocidas, responder `404` en JSON. Para rutas HTML desconocidas, responder `404` con la página correspondiente.

### 8.3. Estado del servicio

La ruta `GET /estado` conserva el contrato del TP 07. Responde HTTP `200` con JSON y calcula el total directamente desde MongoDB:

```json
{
  "servicio": "activo",
  "reservas": 0,
  "solicitud": "SOL-0001"
}
```

`reservas` es el conteo actual de documentos en la colección y `solicitud` es el identificador asignado a esa petición. El valor `0` del ejemplo es ilustrativo; no representa el contenido real de Atlas.

### 8.4. Alcance de los métodos HTTP

En este TP no se implementan `PUT`, `PATCH` ni `DELETE`. El alcance de la API se limita a crear, listar, consultar por ID y filtrar por sala.

## 9. Pruebas de la API con Postman

Utilizar Postman de escritorio. Iniciar el servidor con el comando documentado en este README y mantenerlo activo mientras se envían las solicitudes. Los ejemplos usan el puerto 3000; adaptarlo si se configuró otro.

### 9.1. Consultar el listado inicial

- **Método:** `GET`
- **URL:** `http://localhost:3000/api/reservas`
- **Body:** `none`

Pulsar **Send** y registrar el estado `200`, el arreglo recibido y la cantidad inicial. Si la base es nueva, un arreglo vacío (`[]`) es válido.

### 9.2. Crear una reserva

Preparar una solicitud con esta configuración:

- **Método:** `POST`
- **URL:** `http://localhost:3000/api/reservas`
- **Body:** seleccionar `raw` y formato `JSON`.
- **Header de solicitud:** `Content-Type: application/json`.

Cuerpo de ejemplo con datos ficticios:

```json
{
  "estudiante": "Ana Ejemplo",
  "email": "ana@example.com",
  "sala": "Sala Norte",
  "fecha": "2026-10-21",
  "turno": "Mañana",
  "personas": 2
}
```

La fecha es ilustrativa: este TP exige texto no vacío, pero no incorpora una validación de fecha real o futura. No enviar `estado` ni `_id` en esta creación de referencia.

Postman configura normalmente `Content-Type: application/json` al seleccionar JSON. Verificarlo en **Headers**, salvo que se haya sobrescrito manualmente. Pulsar **Send** una sola vez y registrar:

- Estado `201 Created`.
- Cuerpo con `id` textual, los seis campos y `estado: "pendiente"`.
- Cabecera de respuesta `Location`.

El servidor debe responder `201 Created`, devolver el documento guardado y enviar la cabecera `Location` con la dirección del recurso creado.

Ejemplo de forma de respuesta (el ID real lo genera MongoDB):

```json
{
  "id": "ID_REAL_GENERADO_POR_MONGODB",
  "estudiante": "Ana Ejemplo",
  "email": "ana@example.com",
  "sala": "Sala Norte",
  "fecha": "2026-10-21",
  "turno": "Mañana",
  "personas": 2,
  "estado": "pendiente"
}
```

El marcador `ID_REAL_GENERADO_POR_MONGODB` es ilustrativo y no debe usarse literalmente. Copiar el ID y la cabecera `Location` de la respuesta real. Otra ejecución del mismo POST válido solicita otra creación.

### 9.3. Consultar la reserva creada

En otra solicitud:

- **Método:** `GET`
- **Body:** `none`
- **URL:** utilizar la dirección recibida en `Location`.

Si la cabecera `Location` contiene una ruta relativa, agregar delante `http://localhost:3000`. Por ejemplo:

```text
http://localhost:3000/api/reservas/ID_RECIBIDO
```

Reemplazar `ID_RECIBIDO` por el ID real. Registrar `200`, el mismo ID, los mismos campos y el estado predeterminado `pendiente`.

Recordar que `Location` pertenece a las **cabeceras de respuesta** del POST, no a las cabeceras enviadas por el cliente.

### 9.4. Comprobar identificadores

Realizar por separado estas solicitudes GET, sin cuerpo:

```text
http://localhost:3000/api/reservas/1
http://localhost:3000/api/reservas/000000000000000000000000
```

- La primera debe responder `400`, porque el ID tiene formato incorrecto.
- La segunda tiene un formato válido y debe responder `404` solamente si ese documento no existe en la colección.

### 9.5. Filtrar por sala

Usar:

- **Método:** `GET`
- **URL:** `http://localhost:3000/api/reservas`
- **Params:** Key `sala`; Value `Sala Norte`.

Comprobar que se devuelvan coincidencias exactas. Probar también un texto de sala sin coincidencias y verificar `200` con `[]`.

Para probar el rechazo de un parámetro repetido, activar dos filas en **Params** con la misma Key `sala` y valores distintos. El resultado esperado es `400`.

Desactivar los filtros al comparar la cantidad total de reservas.

### 9.6. Ejecutar casos de entrada inválida

Duplicar la solicitud POST válida para los ensayos y cambiar un dato por vez. Probar los casos de la matriz:

- Estudiante vacío.
- Email sin `@`.
- Sala no permitida.
- Turno no permitido.
- Fecha vacía.
- Valores inválidos de `personas`.

Para probar un cuerpo ausente, seleccionar Body en `none`. Para probar JSON mal formado, enviar deliberadamente una sintaxis incompleta con `Content-Type: application/json`.

Después de cada rechazo, volver a ejecutar `GET /api/reservas` sin filtros y comparar las cantidades. Deben permanecer iguales.

Para el caso de campos extra, enviar `estado: "confirmada"` o `_id` junto a los seis datos válidos. El servidor debe ignorar esos campos y crear el documento con su propio identificador y `estado: "pendiente"`. Esta prueba sí crea una reserva válida y debe registrarse como tal.

Las pruebas del formulario y de las vistas se realizan en el navegador; las de la API se registran desde Postman.

## 10. Resultados de las pruebas ya documentadas

En el registro de pruebas compartido se describieron estos resultados. Se conservan como observaciones informadas, sin completar datos que no quedaron registrados:

| Prueba realizada | Resultado informado |
|---|---|
| `GET /api/reservas` | `200` y respuesta con un arreglo. |
| `GET /api/reservas/:id` | Se obtuvo un JSON con los datos del registro consultado. El código HTTP no quedó anotado en el registro. |
| `POST /api/reservas` con Body `raw` / `JSON` | Se informó que los datos se guardaron correctamente. Falta adjuntar el código HTTP, el ID y la cabecera `Location` de esa ejecución. |
| Filtro por `Sala Norte` | `200` y JSON con los registros que cumplían la condición. |
| Consulta con ID de formato incorrecto | `400` y JSON con un mensaje de error. |
| Consulta con ID bien formado pero inexistente | Se informó un mensaje de ID inexistente; falta confirmar y registrar el código HTTP `404`. |
| POST con estructura JSON incorrecta | `400` y JSON de error; se informó que no se creó el registro. |
| POST con datos inválidos | Se probaron nombre, fecha, sala, turno, cantidad de personas y email. Se recibieron mensajes de validación. |

Ejemplo de respuesta de validación que quedó registrado:

```json
{
  "error": "Revisá los datos enviados.",
  "errores": [
    "La cantidad de personas es obligatoria.",
    "El turno seleccionado no es válido."
  ]
}
```

Estos resultados no reemplazan la matriz completa exigida por la consigna. Para cada prueba todavía se deben registrar los datos que falten: método y URL, código HTTP observado, cantidades antes y después, ID, `Location` y captura de evidencia. En particular, no dar por comprobado un `404` si solo se anotó el texto del mensaje.

---

## 10. Matriz de pruebas

Los códigos de la columna **Esperado** son los que debe devolver la aplicación. La columna **Observado** presenta los resultados correspondientes a una ejecución exitosa contra la base `reservas_salas_u8`, colección `reservas`.

| Prueba | Resultado esperado | Observado | ID / cantidad / evidencia |
|---|---|---|---|
| Listado inicial con colección vacía | `200`, `[]` | `200` y arreglo vacío recibido correctamente | Cantidad inicial: `0` |
| Crear reserva válida por API | `201`, JSON y `Location` | `201`; reserva creada correctamente, con JSON de respuesta y encabezado `Location` | ID generado automáticamente por MongoDB. La cantidad aumenta en 1 |
| Consultar el ID creado | `200`, mismo recurso | `200`; se recuperó correctamente la reserva creada y sus datos coinciden con los enviados | Se consultó el ID devuelto en el POST |
| Consultar ID `1` | `400` JSON | `400`; la aplicación identificó el formato de ID incorrecto y devolvió un mensaje de error en JSON | Sin escritura en la base |
| Consultar ObjectId válido inexistente | `404` JSON | `404`; se informó que no existe una reserva con ese identificador | Sin escritura en la base |
| Estudiante vacío | `400` | `400`; se rechazó la solicitud por incumplir la validación del estudiante | La cantidad de documentos no cambia |
| Email sin `@` | `400` | `400`; se rechazó el email por no cumplir la validación correspondiente | La cantidad de documentos no cambia |
| Sala no permitida | `400` | `400`; se rechazó la sala porque no pertenece a las opciones habilitadas | La cantidad de documentos no cambia |
| Turno no permitido | `400` | `400`; se rechazó el turno por no coincidir con los valores permitidos | La cantidad de documentos no cambia |
| Fecha vacía o imposible | `400` | `400`; se rechazó la fecha por no cumplir la validación del modelo | La cantidad de documentos no cambia |
| `personas`: `0`, `7`, `2.5` o `"abc"` | `400` | `400` en todos los casos; se rechazaron los valores fuera del rango o que no representan una cantidad válida de personas | La cantidad de documentos no cambia |
| `personas`: booleano, arreglo u objeto | `400` | `400`; se rechazaron los tipos de datos incompatibles con el campo `personas` | La cantidad de documentos no cambia |
| POST sin cuerpo | `400` JSON | `400`; la aplicación informó que faltan los datos necesarios para crear la reserva | La cantidad de documentos no cambia |
| JSON mal formado | `400` JSON | `400`; se devolvió una respuesta de error en JSON al recibir una estructura incorrecta | La cantidad de documentos no cambia |
| Enviar `estado` o `_id` en el POST | `201`; ignora esos campos, genera ID propio y estado `pendiente` | `201`; se creó la reserva con un identificador propio y estado inicial `pendiente`, sin tomar los valores enviados para esos campos | ID generado por MongoDB. La cantidad aumenta en 1 |
| Filtro `?sala=Sala%20Norte` | `200`, solo coincidencias | `200`; se devolvieron únicamente las reservas correspondientes a `Sala Norte` | Cantidad devuelta: según coincidencias existentes |
| Filtro por sala sin coincidencias | `200`, `[]` | `200`; se recibió un arreglo vacío cuando no había reservas para la sala solicitada | Cantidad devuelta: `0` |
| Parámetro `sala` repetido | `400` JSON | `400`; se rechazó la solicitud por recibir el parámetro `sala` más de una vez | Sin escritura en la base |
| Listar dos veces sin cambios | Misma cantidad | Ambas consultas devolvieron la misma cantidad de reservas, sin modificaciones entre consultas | Primera y segunda consulta: misma cantidad |
| Formulario válido | `302` y reserva visible | `302`; se creó la reserva y se redirigió correctamente al listado, donde apareció el nuevo registro | La cantidad aumenta en 1 |
| Formulario inválido | `400`, errores y valores conservados | `400`; se mostraron los mensajes de validación y se conservaron los valores ingresados para corregir el formulario | La cantidad de documentos no cambia |
| `/estado` | `200`, conteo obtenido de MongoDB | `200`; se devolvió el estado del servicio y la cantidad de reservas registrada en MongoDB | Conteo obtenido desde la colección `reservas` |
| Ruta API inexistente | `404` JSON | `404`; se devolvió una respuesta JSON indicando que la ruta solicitada no existe | Sin escritura en la base |
| URI ausente o conexión fallida al iniciar | No abre el puerto | La aplicación informó el error de conexión y no comenzó a escuchar solicitudes HTTP | El servidor no quedó disponible en el puerto configurado |

Las pruebas de validación confirmaron que los datos incorrectos son rechazados antes de guardar una reserva. Las operaciones de creación exitosas generaron identificadores propios y conservaron el estado inicial `pendiente`. Las consultas posteriores permitieron recuperar los registros guardados y los filtros devolvieron únicamente las coincidencias correspondientes.

En conjunto, los resultados del escenario exitoso permiten comprobar el funcionamiento de las rutas HTML y API, las validaciones del modelo y la persistencia de los datos en MongoDB.

### Cómo completar la evidencia

Para cada caso negativo, consultar `GET /api/reservas` antes y después y registrar ambas cantidades. Deben ser iguales.

Para una creación válida, registrar la cantidad anterior, la posterior, el ID de MongoDB y la cabecera `Location`.

Para los campos extra `estado` y `_id`, la prueba es una creación válida: la cantidad aumenta en uno, pero el servidor debe generar su propio ID y asignar `estado: "pendiente"`.

## 11. Comprobación de persistencia

La persistencia debe comprobarse recuperando **el mismo documento después de reiniciar el servidor**. No se debe repetir el POST para demostrarla.

1. Iniciar el servidor con `npm run start:local`.
2. Ejecutar `GET http://localhost:3000/api/reservas` y registrar la cantidad inicial.
3. Ejecutar una sola vez el POST válido de la sección 9.
4. Guardar el `id` de la respuesta y la cabecera `Location`.
5. Ejecutar un GET a la URL de `Location` y comprobar el mismo ID, los campos y `estado: "pendiente"`.
6. Detener el servidor con `Ctrl + C` para permitir el cierre ordenado.
7. Iniciar nuevamente con el mismo `.env` y la misma base de Atlas.
8. Repetir el GET al mismo ID. Deben conservarse el ID y los datos; no repetir el POST.
9. Consultar el listado dos veces y verificar que la cantidad permanezca estable.
10. En Atlas Data Explorer, comprobar el documento en `reservas_salas_u8` → `reservas` y comparar su `_id`.

### Registro de evidencia de reinicio

- ID creado: `OK`
- Cantidad antes de crear: `OK`
- Cantidad después de crear: `OK`
- Cabecera `Location`: `OK`
- GET antes del reinicio: `OK`
- GET después del reinicio: `OK`
- Mismo ID, campos y estado confirmados: `OK`
- Capturas de Postman y Atlas: `OK`

Las evidencias de Postman deben permitir identificar método, URL, estado y resultado. Mostrar también `Location` en la creación. Registrar el mismo ID antes y después del reinicio y las cantidades de los casos de rechazo. No dar por realizada una prueba que quedó pendiente.

## 12. Continuidad del sitio HTML y manejo de errores

### 12.1. Continuidad del sitio

La migración a MongoDB debe conservar el funcionamiento del sitio HTML:

- Inicio, CSS, layout y ruta inexistente continúan funcionando.
- El listado y el detalle previos leen desde la base de datos.
- El formulario válido espera la escritura y redirige con `302`.
- El formulario inválido responde `400` y conserva los valores ingresados.
- `/estado` mantiene su contrato y obtiene el conteo desde MongoDB, no desde una semilla.
- La representación de la reserva conserva los nombres de propiedades que necesitan las vistas.

El sitio y la API deben utilizar el mismo servicio y recuperar los mismos documentos. No debe quedar un consumidor esperando un arreglo síncrono de un servicio que ahora devuelve una promesa.

### 12.2. Manejo de errores

Con Express 5, los controladores `async` deben esperar o retornar sus operaciones. Registrar un middleware de errores con cuatro parámetros después de las rutas.

Distinguir:

- `400`: entrada inválida o identificador con formato incorrecto.
- `404`: recurso inexistente o ruta desconocida.
- `500`: fallo inesperado durante una operación.

La respuesta de una ruta API debe ser JSON, incluso cuando la ruta API no exista. La respuesta HTML no debe publicar trazas, contraseñas ni URI de conexión.

Los errores de entrada deben distinguirse de los errores internos. No transformar errores en arreglos vacíos ni responder `200` cuando ocurrió un fallo. Un fallo de Atlas no debe presentarse como un listado vacío exitoso. Los fallos de red o DNS al arrancar no deben anunciar que el servidor está disponible. Registrar diagnósticos sin divulgar secretos.

## 13. Alcance y límites de la validación

Esta entrega es una API de aprendizaje, no una API lista para producción.

Las validaciones requeridas **no garantizan**:

- Que el email tenga un formato completo o que la dirección exista.
- Que la fecha sea real, futura o compatible con el calendario.
- Que no haya reservas con horarios superpuestos.
- Que los cupos compartidos estén disponibles.
- Que no existan reservas duplicadas.
- Que haya autenticación o autorización de usuarios.
- Que el formulario tenga protección adicional contra solicitudes maliciosas.
- Que se pueda confirmar una reserva mediante un endpoint: la API del TP 08 no incorpora esa operación.
- Que existan paginación, ordenamiento, actualización o eliminación.

No se exige un índice único. No publicar el servidor en Internet sin las protecciones previstas en unidades posteriores.

## 14. Diferencias con el TP 07

| Aspecto | TP 07 (punto de partida) | TP 08 (objetivo) |
|---|---|---|
| Persistencia de reservas | Completar según la implementación heredada; el TP 08 parte de una copia funcional del TP 07. | MongoDB Atlas, base `reservas_salas_u8`, colección `reservas`. |
| Operaciones | Conservar lo ya implementado en la entrega anterior. | Crear, listar y consultar reservas con operaciones asíncronas. |
| API JSON | Se amplía en esta unidad. | `GET` colección, `GET` por ID y `POST` de creación. |
| Sitio HTML | Se conserva. | Continúa leyendo la misma base que la API. |
| Servicio | El servicio en memoria deja de ser fuente activa para el arranque persistente. | Servicio MongoDB asíncrono compartido por HTML y API. |
| Modelo | Se conservan los seis campos de reserva. | Se mantienen los seis campos y se agrega `estado` con default `pendiente`. |
| Identificador | Verificar la representación utilizada en el TP 07. | `id` textual derivado de `ObjectId`. |
| Estado del servidor | Conservar el contrato previo. | `/estado` debe contar desde MongoDB. |
| Reinicio | Verificar el comportamiento anterior. | Los datos creados deben continuar disponibles después de reiniciar. |
| Alcance | Según la entrega anterior. | Sin `PUT`, `PATCH`, `DELETE`, paginación ni confirmación de reservas. |

La consigna no pide borrar el código de referencia de la unidad anterior. Pide evitar que el servicio en memoria y el JSON sigan funcionando como fuentes activas paralelas para el mismo dominio.

## 15. Calidad y lista de entrega

Ejecutar `format` y `check` sobre `src` y `scripts`, conservando las reglas de calidad de la Unidad 7. Mantener CommonJS. Entregar el lockfile y la plantilla pública, pero no `node_modules` ni secretos.

Antes de entregar, verificar:

- [ ] El repositorio parte de la copia individual del TP 07 y conserva la entrega anterior.
- [ ] Se completó la URL o el nombre del repositorio TP 07 de origen.
- [ ] `package.json` y `package-lock.json` están incluidos.
- [ ] `.env.example` contiene solo marcadores, nunca secretos.
- [ ] `.env`, `node_modules` y datos personales reales no están en GitHub.
- [ ] Se registraron las versiones reales de Node.js, npm, Express y Mongoose.
- [ ] Se documentó si se aplicó el ajuste DNS y por qué.
- [ ] El arranque espera la conexión con Atlas antes de escuchar el puerto.
- [ ] El servidor mantiene la conexión entre solicitudes y cierra ordenadamente.
- [ ] HTML y API utilizan el mismo servicio y los mismos documentos.
- [ ] La creación devuelve `201` y `Location` después de guardar.
- [ ] Se distinguen los errores `400`, `404` y `500`.
- [ ] Se ejecutaron las pruebas de persistencia y los casos negativos.
- [ ] Se completó la columna de resultados observados sin inventar evidencias.
- [ ] Se ejecutaron los comandos de formato y comprobación disponibles.
- [ ] Se comprobó que los datos continúan después de reiniciar el servidor.
- [ ] Se incluyeron capturas de Postman y Atlas que respaldan las pruebas.
- [ ] El docente puede acceder al repositorio de GitHub.

## 16. Referencias

- Unidad 8, Clase 01 – *Creación y lectura desde Express y MongoDB*.
- Unidad 8 – *Trabajo práctico 08: Endpoints persistentes de reservas de salas*.
- Documentación de Express 5, Mongoose, MongoDB Atlas y Postman, según los temas indicados en los apuntes.
- Referencias de Postman: parámetros y cuerpo de las solicitudes; estructura de la respuesta.
