# TP11 -- Reemplazo completo de reservas desde la web

Aplicación web para administrar reservas de salas de estudio, desarrollada con **Node.js, Express, EJS y MongoDB** mediante Mongoose.

Este trabajo práctico incorpora la edición de una reserva desde la interfaz web mediante un **reemplazo completo**, conservando las funcionalidades web y la API heredadas del TP10.

## Tecnologías utilizadas

- **Node.js:** >= 22.13.0 (entorno de ejecución)
- **Express:** 5.2.1 (servidor y enrutamiento HTTP)
- **MongoDB Atlas:** y **Mongoose:** 9.11.2 (persistencia y validación de documentos) - base de datos `reservas_salas_u11`
- **EJS:** 6.0.1 y **express-ejs-layouts:** 2.5.1 (vistas HTML renderizadas en el servidor)
- **Morgan:** 1.12.1 (registro de solicitudes HTTP)
- **Prettier:** 3.9.9 y **ESLint:** 10.12.0 (formato y análisis estático del código)
- **Postman:** 12.35.1 (opcional para pruebas)

## Requisitos previos

- Node.js 22.13.0 o superior.
- npm.
- Una cuenta y un clúster de MongoDB Atlas.
- Acceso de red al clúster de MongoDB Atlas.

## Instalación

Clonar el repositorio y entrar en la carpeta del proyecto:

``` bash
git clone https://github.com/marcos-batallan/tp-11-salas-web-completa-mongodb.git
cd tp-11-salas-web-completa-mongodb
```

Instalar las dependencias declaradas en el lockfile:

``` bash
npm ci
```

## Configuración del entorno

Crear un archivo `.env` a partir de `.env.example`:

``` env
PORT=3000
NODE_ENV=development
MONGODB_URI=mongodb+srv://USUARIO:CLAVE@HOST/reservas_salas_u11?retryWrites=true&w=majority
```

Reemplazar `USUARIO`, `CLAVE` y `HOST` por los datos de conexión
correspondientes. La URI debe apuntar a la base `reservas_salas_u11`.

**Importante:** `.env` contiene credenciales y no debe subirse al repositorio. El archivo `.env.example` utiliza valores de ejemplo y no contiene secretos.

Comprobar la conexión:

``` bash
node --env-file=.env scripts/comprobar-conexion.js
```

Si la conexión es correcta, se informa:

``` text
Conexión con MongoDB establecida
```

## Ejecución

Iniciar la aplicación cargando las variables de `.env`:

``` bash
npm run start:local
```

También se puede ejecutar con:

``` bash
node --env-file=.env src/index.js
```

Por defecto, la aplicación queda disponible en:

``` text
http://localhost:3000
```

## Estructura del proyecto

```text
tp-11-salas-consultas-mongodb/
│
├── public/
│   ├── css/
│   │   └── estilos.css
│   └── img/
│       ├── favicon.svg
│       └── salas-estudio.svg
│
├── scripts/
│   ├── comprobar-conexion.js
│   ├── crear-reservas.js
│   ├── listar-reservas.js
│   └── validar-reserva.js
│
├── src/
│   ├── config/
│   │   ├── base-datos.js
│   │   └── entorno.js
│   │
│   ├── controladores/
│   │   └── reservas.js
│   │
│   ├── middleware/
│   │   ├── cambios-reservas.js
│   │   ├── errores.js
│   │   ├── reservas.js
│   │   └── solicitudes.js
│   │
│   ├── modelos/
│   │   └── reserva.js
│   │
│   ├── rutas/
│   │   ├── reservas-api.js
│   │   └── reservas.js
│   │
│   ├── servicios/
│   │   ├── reservas-mongo.js
│   │   └── reservas.js
│   │
│   ├── app.js
│   └── index.js
│
├── views/
│   ├── layouts/
│   │   └── main.ejs
│   ├── partials/
│   │   ├── encabezado.ejs
│   │   └── pie.ejs
│   ├── reservas/
│   │   ├── detalle.ejs
│   │   ├── editar.ejs
│   │   ├── lista.ejs
│   │   └── nueva.ejs
│   ├── inicio.ejs
│   └── no-encontrado.ejs
│
├── .env.example
├── .gitignore
├── eslint.config.js
├── package-lock.json
├── package.json
└── README.md
```

La aplicación mantiene una separación por responsabilidades:

-   **Configuración:** lectura del entorno y conexión/desconexión de
    MongoDB.
-   **Modelo:** esquema Mongoose y reglas de validación de la reserva.
-   **Servicio:** operaciones de persistencia y representación de los
    documentos.
-   **Controlador:** coordinación entre solicitudes, servicio y
    respuestas.
-   **Rutas:** definición de endpoints web y API.
-   **Middleware:** validación de entradas, registro de solicitudes y
    manejo centralizado de errores.
-   **Vistas:** formularios, listado, detalle y mensajes HTML.

## Modelo de reserva

Cada documento de la colección `reservas` contiene los siguientes
campos:

  -----------------------------------------------------------------------
  Campo                   Tipo                    Regla
  ----------------------- ----------------------- -----------------------
  `id`                    ObjectId representado   Identificador generado
                          como texto              por MongoDB; se obtiene
                                                  de la ruta al editar

  `estudiante`            String                  Obligatorio

  `email`                 String                  Obligatorio; debe
                                                  contener `@`

  `sala`                  String                  `Sala Norte`,
                                                  `Sala Sur` o
                                                  `Sala Multimedia`

  `fecha`                 String                  Obligatoria

  `turno`                 String                  `Mañana`, `Tarde` o
                                                  `Noche`

  `personas`              Number                  Entero entre 1 y 6

  `estado`                String                  `pendiente` o
                                                  `confirmada`; al crear
                                                  se asigna `pendiente`
                                                  por defecto

  -----------------------------------------------------------------------

## Funcionalidades web

La interfaz permite consultar reservas, crear nuevas, ver su detalle y
editar una reserva existente.

  -----------------------------------------------------------------------
  Método                 Ruta                     Descripción
  ---------------------- ------------------------ -----------------------
  GET                    `/`                      Página de inicio

  GET                    `/estado`                Estado del servicio y
                                                  cantidad de reservas

  GET                    `/reservas`              Listado de reservas

  GET                    `/reservas/nueva`        Formulario de alta

  POST                   `/reservas`              Crear una reserva

  GET                    `/reservas/:id`          Ver el detalle de una
                                                  reserva

  GET                    `/reservas/:id/editar`   Mostrar el formulario
                                                  de edición con los
                                                  siete campos
                                                  precargados

  POST                   `/reservas/:id/editar`   Enviar el reemplazo
                                                  completo de una reserva

  -----------------------------------------------------------------------

### Edición por reemplazo completo

El formulario de edición precarga los siete campos editables:
`estudiante`, `email`, `sala`, `fecha`, `turno`, `personas` y `estado`.
El identificador se toma exclusivamente del parámetro `:id` de la URL y
no se envía como campo editable.

Al guardar, el servidor valida los datos y construye un objeto con
los siete campos admitidos. Luego reutiliza `servicio.reemplazar`,
 la misma operación de servicio utilizada por el endpoint PUT de la API.

El formulario debe enviar los siete campos aunque solo se haya
modificado uno. No se trata de una actualización parcial.

Cuando el reemplazo se completa correctamente, el servidor responde con
**HTTP 303 See Other** y redirige al detalle de la reserva:

``` text
/reservas/:id
```

Si los datos no son válidos, se responde con HTML 400 y se vuelve a
mostrar el formulario de edición con los valores recibidos y los
mensajes correspondientes. La reserva almacenada no se modifica.

Para identificadores inválidos se devuelve HTML 400; para
identificadores válidos que no corresponden a una reserva existente,
HTML 404. La acción Cancelar vuelve al detalle sin enviar el formulario
ni guardar cambios.

## API REST

La API conserva las operaciones disponibles del proyecto anterior:

  ------------------------------------------------------------------------
  Método           Ruta                  Descripción      Respuesta
                                                          esperada
  ---------------- --------------------- ---------------- ----------------
  GET              `/api/reservas`       Consultar        200
                                         reservas         

  GET              `/api/reservas/:id`   Obtener una      200
                                         reserva por ID   

  POST             `/api/reservas`       Crear una        201
                                         reserva          

  PUT              `/api/reservas/:id`   Reemplazar todos 200
                                         los campos       
                                         editables        

  PATCH            `/api/reservas/:id`   Actualizar       200
                                         parcialmente los 
                                         campos admitidos 
                                         por la API       

  DELETE           `/api/reservas/:id`   Eliminar una     204
                                         reserva


Los errores de la API se responden en formato JSON. Los identificadores
inválidos y las reservas inexistentes se informan con los códigos HTTP
correspondientes.

### Consulta de reservas

El endpoint `GET /api/reservas` admite estos parámetros:

  -----------------------------------------------------------------------
  Parámetro               Descripción             Valores / límites
  ----------------------- ----------------------- -----------------------
  `sala`                  Filtrar por sala        Texto de sala

  `orden`                 Ordenar resultados      `estudiante`,
                                                  `personas-asc`,
                                                  `personas-desc`

  `pagina`                Número de página        Entero entre 1 y 1000;
                                                  por defecto 1

  `limite`                Cantidad de resultados  Entero entre 1 y 20;
                                                  por defecto 2

  -----------------------------------------------------------------------

Ejemplos:

``` text
GET /api/reservas
GET /api/reservas?sala=Sala%20Norte
GET /api/reservas?orden=personas-desc
GET /api/reservas?pagina=1&limite=2
GET /api/reservas?sala=Sala%20Sur&orden=estudiante&pagina=1&limite=5
```

La respuesta de listado incluye `datos`, `pagina`, `limite`, `total` y
`paginas`.

### Ejemplo de reemplazo completo con PUT

En Postman seleccionar **PUT**, ingresar la URL con un ID real y
configurar **Body → raw → JSON**. Enviar los siete campos:

``` http
PUT http://localhost:3000/api/reservas/ID_REAL
Content-Type: application/json
```

``` json
{
  "estudiante": "Ana Ejemplo",
  "email": "ana@example.com",
  "sala": "Sala Norte",
  "fecha": "2026-11-04",
  "turno": "Mañana",
  "personas": 3,
  "estado": "pendiente"
}
```

El reemplazo completo responde HTTP 200 y devuelve la reserva con el
mismo ID y los valores actualizados. Si se omite un campo obligatorio
del reemplazo ---por ejemplo, `estado`--- la API responde HTTP 400 y no
modifica el documento.

**PATCH y DELETE se conservan únicamente como operaciones de la API
heredada. No se implementan acciones web de modificación parcial ni
eliminación en este TP.**

## Validaciones y manejo de errores

-   Los textos obligatorios deben contener información.
-   El email debe incluir `@`.
-   La sala y el turno deben pertenecer a las opciones permitidas.
-   `personas` debe recibirse con contenido y convertirse en un número
    entero entre 1 y 6.
-   `estado` debe ser `pendiente` o `confirmada`.
-   En la edición web se rechazan campos no admitidos, identificadores
    enviados en el cuerpo y propiedades adicionales.
-   El reemplazo se ejecuta con validadores de Mongoose habilitados y
    sin `upsert`.
-   Los errores de validación de formularios web se muestran en HTML;
    los errores de API se responden en JSON.
-   La vista compartida de error utiliza un mensaje neutral para no
    presentar como 404 los errores 400 o 500.

## Persistencia y datos de prueba

La aplicación utiliza la base `reservas_salas_u11` y la colección
`reservas`. Los documentos permanecen almacenados en MongoDB al detener
y volver a iniciar el servidor.

Para preparar los datos indicados en la consigna, se pueden crear desde
el formulario web cuando la base está vacía:

  -----------------------------------------------------------------------------------------
  Estudiante          Email          Sala      Fecha         Turno      Personas   Estado
                                             ilustrativa                           inicial
  ------------- ------------------- ------- -------------- ----------  ---------- ---------
  Ana Ejemplo     ana@example.com    Sala    2026-11-04      Mañana           2   pendiente
                                     Norte                                     
  Bruno Ejemplo  bruno@example.com   Sala    2026-11-04      Tarde            3   pendiente
                                     Sur               
                                 
  -----------------------------------------------------------------------------------------

Si los registros ya existen, no deben crearse nuevamente. La aplicación
no realiza una carga automática de estas reservas al iniciar.

## Pruebas realizadas

Las verificaciones funcionales se realizaron con la aplicación conectada
a MongoDB, utilizando el navegador y Postman.

  ---------------------------------------------------------------------
  Caso                               Verificación
  ---------------------------------- ----------------------------------
  Precarga de edición                GET de edición muestra los siete
                                     campos actuales

  Reemplazo desde la web             POST exitoso responde 303 y
                                     redirige al detalle

  Persistencia web/API               GET de API devuelve el mismo ID y
                                     los valores guardados desde la web

  PUT completo                       Responde 200; el formulario web
                                     refleja los siete valores
                                     recibidos

  PUT incompleto                     Omitir `estado` responde 400 y
                                     conserva el documento anterior

  Formulario inválido                Enviar `personas=0` devuelve HTML
                                     400 con el mensaje de validación

  Reintento válido                   Corregir el dato y reenviar
                                     produce la redirección 303

  Cancelación                        Volver al detalle sin enviar el
                                     formulario no modifica la reserva

  ID inválido                        La edición web responde HTML 400

  ID válido inexistente              La edición web responde HTML 404

  POST con ID inválido o inexistente Se comprueban las respuestas HTML
                                     400 y 404

  Reinicio del servidor              Los documentos y sus IDs
                                     persisten; el total permanece en
                                     dos, sin altas duplicadas

  Funcionalidades heredadas          Se comprueban creación y lectura
                                     de reservas
  ---------------------------------------------------------------------

### Ejecución de controles de calidad

Formatear los archivos:

``` bash
npm run format
```

Comprobar formato y lint:

``` bash
npm run check
```

El script `check` ejecuta `prettier --check src scripts` y ESLint sobre
`src scripts`.

## Scripts disponibles

  ---------------------------------------------------------------------
  Comando                            Función
  ---------------------------------- ----------------------------------
  `npm start`                        Inicia `src/index.js` usando las
                                     variables ya disponibles en el
                                     entorno

  `npm run start:local`              Inicia la aplicación cargando
                                     `.env`

  `npm run format`                   Formatea `src` y `scripts`

  `npm run format:check`             Comprueba el formato de `src` y
                                     `scripts`

  `npm run lint`                     Ejecuta ESLint sobre `src` y
                                     `scripts`

  `npm run check`                    Ejecuta comprobación de formato y
                                     lint
  ---------------------------------------------------------------------

Scripts auxiliares:

``` bash
node --env-file=.env scripts/comprobar-conexion.js
node --env-file=.env scripts/crear-reservas.js
node --env-file=.env scripts/listar-reservas.js
node --env-file=.env scripts/validar-reserva.js
```

`crear-reservas.js` inserta un registro de prueba cada vez que se
ejecuta; utilizarlo conscientemente para evitar altas repetidas.

## Alcance del TP11

El trabajo práctico incorpora el reemplazo completo de reservas desde la
interfaz web y mantiene las funcionalidades anteriores del proyecto. No
agrega PATCH ni DELETE web, filtros, ordenamiento ni paginación al
listado HTML. Las operaciones PATCH, DELETE y las consultas con
parámetros continúan disponibles en la API heredada.
