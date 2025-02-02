# Opciones para server

## server.host

- **Tipo:** `string | boolean`
- **Por defecto:** `'localhost'`

  Especifica en qué direcciones IP debe escuchar el servidor.
  Configuralo en `0.0.0.0` o `true` para escuchar en todas las direcciones, incluidas las LAN y las direcciones públicas.

  Esto se puede configurar a través de la CLI usando `--host 0.0.0.0` o `--host`.

  :::tip NOTA

  Hay casos en los que otros servidores pueden responder en lugar de Vite.

  El primer caso es cuando se usa `localhost`. Node.js por debajo de v17 reordena el resultado de la dirección resuelta por DNS de forma predeterminada. Al acceder a `localhost`, los navegadores usan DNS para resolver la dirección y esa dirección puede diferir de la dirección que está escuchando Vite. Vite imprime en consola la dirección resuelta cuando difiere.

  Puedes configurar [`dns.setDefaultResultOrder('verbatim')`](https://nodejs.org/api/dns.html#dns_dns_setdefaultresultorder_order) para deshabilitar el comportamiento de reordenación. O podrías configurar `server.host` a `127.0.0.1` explícitamente.

  ```js
  // vite.config.js
  import { defineConfig } from 'vite'
  import dns from 'dns'
  dns.setDefaultResultOrder('verbatim')
  export default defineConfig({
    // omitido
  })
  ```
  El segundo caso es cuando se utilizan hosts comodín (por ejemplo, `0.0.0.0`). Esto se debe a que los servidores que escuchan en hosts que no son comodín tienen prioridad sobre los que escuchan en hosts comodín.

  :::

## server.port

- **Tipo:** `number`
- **Por defecto:** `5173`

  Especifica el puerto del servidor. Ten en cuenta que si el puerto ya se está utilizando, Vite probará automáticamente el siguiente puerto disponible, por lo que es posible que este no sea el puerto real en el que el servidor terminará escuchando.

## server.strictPort

- **Tipo:** `boolean`

  Colocalo en `true` para finalizar si el puerto ya está en uso, en lugar de probar automáticamente el siguiente puerto disponible.

## server.https

- **Tipo:** `boolean | https.ServerOptions`

  Habilita TLS + HTTP/2. Ten en cuenta que esto cambia a TLS solo cuando también se usa la opción [`server.proxy`](#server-proxy).

  El valor también puede ser un [objeto de opciones](https://nodejs.org/api/https.html#https_https_createserver_options_requestlistener) pasado a `https.createServer()`.

## server.open

- **Tipo:** `boolean | string`

  Abre automáticamente la aplicación en el navegador al iniciar el servidor. Cuando el valor es una cadena, se utilizará como nombre de ruta de la URL. Si deseas abrir el servidor en un navegador específico, puedes configurar `process.env.BROWSER` (por ejemplo, `firefox`). Consulta [el paquete `open`](https://github.com/sindresorhus/open#app) para obtener más detalles.

  **Ejemplo:**

  ```js
  export default defineConfig({
    server: {
      open: '/docs/index.html'
    }
  })
  ```

## server.proxy

- **Tipo:** `Record<string, string | ProxyOptions>`

  Configura reglas de proxy personalizadas para el servidor de desarrollo. Espera un objeto de `{ key: options }` pares. Si la clave comienza con `^`, se interpretará como `RegExp`. La opción `configure` se puede utilizar para acceder a la instancia del proxy.

  Usa [`http-proxy`](https://github.com/http-party/node-http-proxy). Todas las opciones [aquí](https://github.com/http-party/node-http-proxy#options).

  En algunos casos, es posible que también desees configurar el servidor de desarrollo relacionado (por ejemplo, para agregar middlewares personalizados a la aplicación interna [connect](https://github.com/senchalabs/connect)). Para hacerlo, debes escribir tu propio [complemento](/guide/using-plugins.html) y usar la función [configureServer](/guide/api-plugin.html#configureserver).

  **Ejemplo:**

  ```js
  export default defineConfig({
    server: {
      proxy: {
        // string shorthand
        '/foo': 'http://localhost:4567',
        // con opciones
        '/api': {
          target: 'http://jsonplaceholder.typicode.com',
          changeOrigin: true,
          rewrite: (path) => path.replace(/^\/api/, '')
        },
        // con RegEx
        '^/fallback/.*': {
          target: 'http://jsonplaceholder.typicode.com',
          changeOrigin: true,
          rewrite: (path) => path.replace(/^\/fallback/, '')
        },
        // Usando la instancia de proxy
        '/api': {
          target: 'http://jsonplaceholder.typicode.com',
          changeOrigin: true,
          configure: (proxy, options) => {
            // proxy será una instancia de 'http-proxy'
          }
        },
        // Haciendo proxy de websockets o socket.io
        '/socket.io': {
          target: 'ws://localhost:5173',
          ws: true
        }
      }
    }
  })
  ```

## server.cors

- **Tipo:** `boolean | CorsOptions`

  Configura las CORS para el servidor de desarrollo. Esto está habilitado por defecto y permite cualquier origen. Pase un [objeto de opciones](https://github.com/expressjs/cors) para ajustar el comportamiento o `false` para deshabilitarlo.

## server.headers

- **Tipo:** `OutgoingHttpHeaders`

  Especifica los encabezados de respuesta del servidor.

## server.hmr

- **Tipo:** `boolean | { protocol?: string, host?: string, port?: number, path?: string, timeout?: number, overlay?: boolean, clientPort?: number, server?: Server }`

  Deshabilita o configura la conexión HMR (en los casos en que el websocket HMR deba usar una dirección diferente del servidor http).

  Coloca `server.hmr.overlay` en `false` para deshabilitar la superposición de errores del servidor.

  `clientPort` es una opción avanzada que sobreescribe el puerto solo en el lado del cliente, lo que le permite servir el websocket en un puerto diferente al que busca el código del cliente.

  Cuando se define `server.hmr.server`, Vite procesará las solicitudes de conexión HMR a través del servidor provisto. Si no está en modo middleware, Vite intentará procesar las solicitudes de conexión HMR a través del servidor existente. Esto puede ser útil cuando se usan certificados autofirmados o cuando desea exponer a Vite a través de una red en un solo puerto.

  :::tip NOTA

  Con la configuración predeterminada, se espera que los proxies inversos frente a Vite admitan WebSocket de proxy. Si el cliente de Vite HMR no logra conectar WebSocket, el cliente recurrirá a conectar WebSocket directamente al servidor de Vite HMR sin pasar por los proxies inversos:

  ```
  Direct websocket connection fallback. Check out https://vitejs.dev/config/server-options.html#server-hmr to remove the previous connection error.
  ```
  Se puede ignorar el error que aparece en el navegador cuando ocurre el fallback. Para evitar el error al omitir directamente los proxies inversos, podrías:

  - configurar el proxy inverso para el proxy de WebSocket también
  - configurar [`server.strictPort = true`](#server-strictport) y configurar `server.hmr.clientPort` con el mismo valor que `server.port`
  - configurar `server.hmr.port` en un valor diferente de [`server.port`](#server-port)
  :::

## server.watch

- **Tipo:** `object`

  Opciones para el observador del sistema de archivos que serán pasados a [chokidar](https://github.com/paulmillr/chokidar#api).

  Al ejecutar Vite en el subsistema de Windows para Linux (WSL) 2, si la carpeta del proyecto reside en un sistema de archivos de Windows, deberá configurar esta opción en `{ usePolling: true }`. Esto se debe a [una limitación de WSL2](https://github.com/microsoft/WSL/issues/4739) con el sistema de archivos de Windows.

  El observador del servidor Vite omite los directorios `.git/` y `node_modules/` de forma predeterminada. Si deseas ver un paquete dentro de `node_modules/`, puede pasar un patrón global negado a `server.watch.ignored`. Es decir:

  ```js
  export default defineConfig({
    server: {
      watch: {
        ignored: ['!**/node_modules/your-package-name/**']
      }
    },
    // El paquete observado debe excluirse de la optimización,
    // para que pueda aparecer en el gráfico de dependencia y activar hot reload.
    optimizeDeps: {
      exclude: ['your-package-name']
    }
  })
  ```

## server.middlewareMode

- **Tipo:** `boolean`
- **Por defecto:** `false`

  Crea un servidor Vite en modo middleware.

- **Relacionado:** [appType](./shared-options#apptype), [SSR - Configuración del servidor de desarrollo](/guide/ssr#configuracion-del-servidor-de-desarrollo)

- **Ejemplo:**

```js
const express = require('express')
const { createServer: createViteServer } = require('vite')

async function createServer() {
  const app = express()

  // Crea servidor Vite en modo middleware
  const vite = await createViteServer({
    server: { middlewareMode: true },
    appType: 'custom' // no incluir middlewares de manejo de HTML predeterminado de Vite
  // Usa la instancia de conexión de vite como middleware
  app.use(vite.middlewares)

  app.use('*', async (req, res) => {
    // Dado que `appType` es `'custom'`, debería servir la respuesta aquí.
    // Nota: si `appType` es `'spa'` o `'mpa'`, Vite incluye middlewares para manejar
    // Solicitudes HTML y 404, por lo que se deben agregar middlewares de usuario
    // antes de que los middlewares de Vite surtan efecto en su lugar
  })
}

createServer()
```

## server.base

- **Tipo:** `string | undefined`

  Antepone esta carpeta a las solicitudes http, para usar cuando se haga proxy de vite como una subcarpeta. Debe comenzar y terminar con el carácter `/`.

## server.fs.strict

- **Tipo:** `boolean`
- **Por defecto:** `true` (habilitado por defecto desde Vite 2.7)

  Restringe el servicio de archivos fuera de la raíz del espacio de trabajo.

## server.fs.allow

- **Tipo:** `string[]`

  Restringe los archivos que podrían servirse a través de `/@fs/`. Cuando `server.fs.strict` se coloca en `true`, el acceso a archivos fuera de esta lista de directorios que no se importaron de un archivo permitido resultará en un 403.

  Vite buscará la raíz del potencial espacio de trabajo y la usará por defecto. Un espacio de trabajo válido cumple con las siguientes condiciones; de lo contrario, se recurrirá a la [raíz del proyecto](/guide/#index-html-y-raiz-del-proyecto).

  - contiene el campo `workspaces` en `package.json`
  - contiene uno de los siguientes archivos
    - `pnpm-workspace.yaml`

  Acepta una ruta para especificar la raíz del espacio de trabajo personalizado. Podría ser una ruta absoluta o una ruta relativa a la [raíz del proyecto](/guide/#index-html-y-raiz-del-proyecto). Por ejemplo:

  ```js
  export default defineConfig({
    server: {
      fs: {
        // Permitir servir archivos desde un nivel hasta la raíz del proyecto
        allow: ['..']
      }
    }
  })
  ```

  Cuando se especifica `server.fs.allow`, la detección automática de la raíz del espacio de trabajo se desactivará. Para ampliar el comportamiento original, se expone una utilidad `searchForWorkspaceRoot`:

  ```js
  import { defineConfig, searchForWorkspaceRoot } from 'vite'

  export default defineConfig({
    server: {
      fs: {
        allow: [
          // busca la raíz del espacio de trabajo
          searchForWorkspaceRoot(process.cwd()),
          // tus reglas personalizadas
          '/path/to/custom/allow'
        ]
      }
    }
  })
  ```

## server.fs.deny

- **Tipo:** `string[]`

  Lista de bloqueo para archivos sensibles que están restringidos para ser servidos por el servidor de desarrollo de Vite.

  Por defecto a `['.env', '.env.*', '*.{pem,crt}']`.

## server.origin

- **Tipo:** `string`

Define el origen de las URL de recursos generadas durante el desarrollo.

```js
export default defineConfig({
  server: {
    origin: 'http://127.0.0.1:8080'
  }
})
```