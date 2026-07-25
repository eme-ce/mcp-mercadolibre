<<<<<<< HEAD
# mcp-mercadolibre
=======
# Servidor MCP de MercadoLibre

Un servidor [Model Context Protocol (MCP)](https://modelcontextprotocol.io) que conecta a Claude directamente con tu cuenta de vendedor de MercadoLibre. Gestioná publicaciones, pedidos y envíos — todo mediante lenguaje natural.

Apunta a **MLA (Argentina)** por defecto. Cambiá `ML_SITE` para otros mercados.

---

## Qué podés hacer con esto

- *"Listame mis publicaciones activas"*
- *"Creá una nueva publicación para un iPhone 14 usado, ARS 500000, categoría MLA1055"*
- *"Mostrame mis últimos 10 pedidos pagados"*
- *"Conseguime la etiqueta de envío del despacho 12345678"*
- *"¿Qué categoría debería usar para auriculares con cancelación de ruido?"*

---

## Requisitos

- Python 3.11 o superior
- Una cuenta de vendedor de MercadoLibre
- Una aplicación de desarrollador de MercadoLibre (`client_id` + `client_secret`)
- Una cuenta de Claude.ai Pro, Team o Enterprise (para conexiones MCP remotas)

---

## Paso 1 — Crear una aplicación de MercadoLibre

1. Andá a [developers.mercadolibre.com.ar](https://developers.mercadolibre.com.ar) e iniciá sesión
2. Hacé clic en **Crear aplicación**
3. Completá los datos de la app. En **URI de redirección** ingresá tu URL de callback:
   - Local: `http://localhost:8000/auth/callback`
   - Railway: `https://your-app.up.railway.app/auth/callback`
4. Guardá la app y copiá tu **App ID** (esto es `ML_CLIENT_ID`) y **Secret key** (`ML_CLIENT_SECRET`)

---

## Paso 2 — Configurar el servidor localmente

### Cloná este repo

```bash
git clone https://github.com/your-username/mercadolibre-mcp.git
cd mercadolibre-mcp
```

### Instalá las dependencias

```bash
pip install -r requirements.txt
```

### Configurá tus variables de entorno

```bash
cp env.example .env
```

Abrí `.env` y completá tus valores:

```env
ML_CLIENT_ID=your-app-id
ML_CLIENT_SECRET=your-secret-key
ML_REDIRECT_URI=http://localhost:8000/auth/callback
BEARER_TOKEN=pick-a-long-random-string-here
```

### Iniciá el servidor

```bash
python server.py
```

### Completá el flujo de OAuth

ML requiere autorización del usuario antes de que el servidor pueda acceder a los datos de tu cuenta.

1. Abrí `http://localhost:8000/auth/url` — devuelve un JSON con `auth_url`
2. Abrí esa URL en tu navegador y autorizá la app
3. ML te redirige a `/auth/callback` — el servidor intercambia el código por tokens
4. El servidor registra `ML_ACCESS_TOKEN` y `ML_REFRESH_TOKEN` en la consola

**Pegá esos dos valores en tu archivo `.env`** para que sobrevivan a los reinicios del servidor:

```env
ML_ACCESS_TOKEN=APP_USR-...
ML_REFRESH_TOKEN=TG-...
```

> Los tokens de acceso de ML expiran cada 6 horas. El servidor los renueva automáticamente usando el refresh token, pero el refresh token se rota en cada renovación — revisá tus logs y actualizá `ML_REFRESH_TOKEN` después de cada reinicio hasta que tengas una configuración más duradera.

---

## Paso 3 — Desplegar en la nube

### Desplegar en Railway

1. Forkeá este repo de GitHub a tu propia cuenta
2. Andá a [railway.app](https://railway.app) e iniciá sesión con GitHub
3. Hacé clic en **New Project** → **Deploy from GitHub repo**
4. Seleccioná tu repo forkeado
5. Cuando termine el build, andá a tu servicio → **Settings** → **Networking** → **Generate Domain**
6. Copiá tu URL pública — se ve algo así como `https://ml-mcp-production.up.railway.app`

### Agregá tus variables de entorno en Railway

| Variable | Requerida | Valor |
|---|---|---|
| `ML_CLIENT_ID` | ✅ | Tu App ID de ML |
| `ML_CLIENT_SECRET` | ✅ | Tu Secret key de ML |
| `ML_REDIRECT_URI` | ✅ | `https://your-app.up.railway.app/auth/callback` |
| `ML_ACCESS_TOKEN` | ✅* | Obtenido después del flujo de OAuth |
| `ML_REFRESH_TOKEN` | ✅* | Obtenido después del flujo de OAuth |
| `BEARER_TOKEN` | ✅ | Cadena aleatoria larga — el mismo valor que vas a ingresar en Claude |
| `ML_SITE` | No | `MLA` (por defecto) |
| `PORT` | No | `8000` |
| `ALLOW_TOKEN_QUERY_PARAM` | No | `1` — solo si Claude.ai no puede enviar headers de Authorization |

*Requerido para que el servidor pueda hacer llamadas a la API. Completá el flujo de OAuth en tu instancia local primero, y después pegá los tokens acá.

---

## Paso 4 — Conectar con Claude

1. Andá a [claude.ai](https://claude.ai) → hacé clic en tu ícono de perfil → **Settings**
2. Navegá a **Integrations**
3. Hacé clic en **Add integration**
4. Completá:
   - **Name:** `MercadoLibre`
   - **URL:** `https://your-app.up.railway.app/mcp`
5. En el campo de **authentication token**: pegá el valor de tu `BEARER_TOKEN`

---

## Herramientas disponibles

| Herramienta | Descripción |
|---|---|
| `ml_get_my_user` | Obtiene el perfil del vendedor autenticado |
| `ml_list_items` | Lista tus publicaciones (con filtro de estado y paginación) |
| `ml_get_item` | Obtiene una publicación por su ID |
| `ml_create_item` | Crea una nueva publicación de producto |
| `ml_update_item` | Actualiza una publicación existente |
| `ml_change_item_status` | Pausa, reactiva o cierra una publicación |
| `ml_list_orders` | Lista tus pedidos (con filtro de estado) |
| `ml_get_order` | Obtiene un pedido por su ID |
| `ml_get_shipment` | Obtiene los detalles de envío de un despacho |
| `ml_get_shipment_label` | Obtiene una etiqueta de envío imprimible (ZPL2 o PDF) |
| `ml_predict_category` | Predice la mejor categoría para una descripción de producto |
| `ml_get_category_attributes` | Obtiene los atributos requeridos/opcionales de una categoría |

---

## Referencia de variables de entorno

| Variable | Requerida | Valor por defecto | Descripción |
|---|---|---|---|
| `ML_CLIENT_ID` | ✅ | — | App ID de la aplicación de ML |
| `ML_CLIENT_SECRET` | ✅ | — | Secret key de la aplicación de ML |
| `ML_REDIRECT_URI` | ✅ | — | URL de callback de OAuth registrada en tu app de ML |
| `ML_ACCESS_TOKEN` | ✅* | — | Token de acceso de ML (del flujo de OAuth) |
| `ML_REFRESH_TOKEN` | ✅* | — | Refresh token de ML (del flujo de OAuth) |
| `BEARER_TOKEN` | ✅ | — | Protege tu endpoint MCP — configurá el mismo valor en Claude |
| `ML_SITE` | No | `MLA` | ID del sitio: MLA=Argentina, MLB=Brasil, MLM=México, MLC=Chile |
| `TOKEN_REFRESH_BUFFER` | No | `1800` | Segundos antes del vencimiento del token para disparar una renovación |
| `PORT` | No | `8000` | Puerto en el que escucha el servidor |
| `MCP_TRANSPORT` | No | `streamable-http` | Protocolo de transporte |
| `ALLOW_TOKEN_QUERY_PARAM` | No | — | Poner en `1` solo si tu cliente MCP no puede enviar headers de Authorization |
| `MAX_REQUEST_BODY` | No | `1048576` | Tamaño máximo del cuerpo de la request entrante, en bytes (1 MB) |
| `RATE_LIMIT_RPM` | No | `60` | Máximo de requests por minuto por IP |
| `RATE_LIMIT_MAX_IPS` | No | `10000` | Máximo de IPs rastreadas por el limitador de tasa |
| `TRUSTED_PROXY_COUNT` | No | `1` | Cantidad de proxies inversos delante del servidor |

*Requerido para el acceso a la API. Se obtiene completando el flujo de OAuth.

---

## Resolución de problemas

**Error "No ML_REFRESH_TOKEN available"**
Todavía no se completó el flujo de OAuth. Visitá `/auth/url`, autorizá la app y pegá los tokens de los logs del servidor en tus variables de entorno.

**401 Unauthorized de MercadoLibre**
Tu token de acceso es inválido o expiró, y el refresh token falta o es inválido. Volvé a ejecutar el flujo de OAuth para obtener tokens nuevos.

**"Token exchange failed" en /auth/callback**
Asegurate de que `ML_REDIRECT_URI` coincida exactamente con lo registrado en tu app de ML (incluyendo el esquema y el path). Incluso una diferencia de barra final va a causar un desajuste.

**Claude no puede conectarse al servidor**
Asegurate de que tu despliegue en Railway esté activo y de que se haya generado un dominio. Visitá `https://your-app.up.railway.app/health` — debería devolver `{"status": "ok"}`.

**El refresh token deja de funcionar después de un reinicio**
ML rota el refresh token en cada llamada de renovación. El servidor registra el nuevo `ML_REFRESH_TOKEN` cada vez que renueva. Actualizá ese valor en tus variables de entorno de Railway para mantenerlo vigente.

---

## Licencia

MIT
>>>>>>> 61ebe8b (primer commit. forkeado desde shopify-mcp)
