# Consumo de la API REST de Contentful con el SDK de JavaScript

## Metadatos

| Campo            | Detalle                                      |
|------------------|----------------------------------------------|
| **Duración**     | 85 minutos                                   |
| **Complejidad**  | Media                                        |
| **Nivel Bloom**  | Crear (*Create*)                             |
| **Capítulo**     | 2 — Autenticación, REST, Paginación y Errores |
| **Lab ID**       | 02-00-01                                     |

---

## Descripción General

En esta práctica construirás desde cero una **librería de utilidades de acceso a contenido** en Node.js que encapsula el consumo de la Content Delivery API de Contentful. Configurarás credenciales de forma segura mediante variables de entorno, inicializarás el cliente oficial del SDK de JavaScript y crearás funciones reutilizables para consultar, filtrar y paginar entradas. Finalizarás implementando un manejo de errores robusto que distinga entre errores de autenticación, recursos no encontrados y rate limiting. El proyecto resultante servirá como base para las prácticas 3, 4 y 5 del curso.

---

## Objetivos de Aprendizaje

Al completar este laboratorio serás capaz de:

- [ ] Configurar la autenticación en Contentful usando API Keys de Delivery y Preview, almacenándolas de forma segura con `dotenv` y sin exponerlas en el repositorio Git.
- [ ] Consumir la API REST de Contentful mediante el SDK oficial de JavaScript (`contentful@10.x`) para obtener colecciones de entradas filtrando por `content_type`, campo y valor.
- [ ] Implementar paginación con los parámetros `skip` y `limit` para recuperar grandes volúmenes de contenido en lotes controlados.
- [ ] Gestionar errores de API de forma diferenciada para códigos HTTP 401, 404 y 429, aplicando retry con backoff exponencial básico para el caso de rate limiting.
- [ ] Aplicar buenas prácticas de seguridad y estructura de proyecto: separación de configuración, `.gitignore` correcto y logging estructurado.

---

## Prerrequisitos

### Conocimientos

| Área                                  | Nivel requerido                                          |
|---------------------------------------|----------------------------------------------------------|
| JavaScript asíncrono (async/await)    | Básico — entender Promises y manejo de errores           |
| Node.js y npm                         | Básico — crear proyectos, instalar dependencias          |
| Variables de entorno                  | Básico — saber qué son y para qué sirven                 |
| Contentful (espacio y content types)  | Completado en Práctica 1 del curso                       |
| HTTP (verbos, códigos de estado)      | Básico — GET, 200, 401, 404, 429                         |

### Acceso y Recursos

| Recurso                                   | Estado requerido                                                      |
|-------------------------------------------|-----------------------------------------------------------------------|
| Cuenta Contentful (Plan Community)        | Activa con espacio creado en Práctica 1                              |
| Space ID de tu espacio Contentful         | Disponible en *Settings → General Settings*                          |
| Content Delivery API Key                  | Generada en *Settings → API Keys*                                    |
| Content Preview API Key                   | Generada junto con la Delivery Key (mismo objeto API Key)            |
| Node.js 18.x LTS                          | Instalado y verificado (`node --version`)                            |
| Visual Studio Code 1.85+                  | Instalado con extensión **ESLint** activa                            |
| Git 2.40+                                 | Instalado y configurado con nombre/email de usuario                  |

> **⚠️ Si no completaste la Práctica 1:** Ejecuta el script de importación provisto por el instructor antes de continuar:
> ```bash
> contentful space import --space-id <TU_SPACE_ID> --content-file starter-data.json
> ```

---

## Entorno del Laboratorio

### Hardware Mínimo

| Componente        | Mínimo                        | Recomendado                  |
|-------------------|-------------------------------|------------------------------|
| Procesador        | Intel Core i5 64-bit / AMD equiv. | i7 / Ryzen 5 o superior  |
| RAM               | 8 GB                          | 16 GB                        |
| Espacio en disco  | 20 GB libres                  | 30 GB libres                 |
| Conexión          | 10 Mbps estables              | 25 Mbps o superior           |
| Resolución        | 1280 × 720                    | 1920 × 1080                  |

### Software Requerido

| Herramienta              | Versión mínima | Verificación                        |
|--------------------------|----------------|-------------------------------------|
| Node.js                  | 18.x LTS       | `node --version`                    |
| npm                      | 9.x            | `npm --version`                     |
| Visual Studio Code       | 1.85           | Menú *Help → About*                 |
| Git                      | 2.40           | `git --version`                     |
| Contentful CLI           | 3.x            | `contentful --version`              |

### Comandos de Preparación del Entorno

Ejecuta los siguientes comandos en tu terminal para verificar que todo está listo antes de comenzar:

```bash
# Verificar versiones instaladas
node --version      # Debe mostrar v18.x.x o superior
npm --version       # Debe mostrar 9.x.x o superior
git --version       # Debe mostrar 2.40.x o superior

# Verificar conectividad con Contentful (sin autenticación)
curl -s -o /dev/null -w "%{http_code}" \
  "https://cdn.contentful.com/spaces/cfexampleapi/entries?access_token=b4c0n73n7fu1"
# Debe devolver: 200
```

---

## Pasos del Laboratorio

---

### Paso 1 — Crear la Estructura del Proyecto y Configurar Git

**Objetivo:** Inicializar un proyecto Node.js limpio con la estructura de carpetas correcta y configurar Git con las salvaguardas de seguridad necesarias para nunca exponer credenciales.

**Instrucciones:**

1. Abre una terminal y navega al directorio donde guardas tus proyectos del curso:

```bash
cd ~/proyectos-contentful   # Ajusta esta ruta según tu sistema
```

2. Crea el directorio del proyecto e inicializa npm:

```bash
mkdir lab-02-contentful-sdk
cd lab-02-contentful-sdk
npm init -y
```

3. Inicializa el repositorio Git:

```bash
git init
git branch -M main
```

4. Crea el archivo `.gitignore` **antes de cualquier otro archivo**. Este paso es obligatorio:

```bash
# En macOS/Linux:
cat > .gitignore << 'EOF'
# Variables de entorno - NUNCA commitear
.env
.env.local
.env.*.local

# Dependencias
node_modules/

# Logs
*.log
npm-debug.log*

# Sistema operativo
.DS_Store
Thumbs.db

# Editor
.vscode/settings.json
EOF
```

> **En Windows (PowerShell):** Crea el archivo `.gitignore` manualmente en VS Code con el contenido anterior.

5. Crea el archivo `.env.example` como plantilla documentada (este archivo **sí** se commitea):

```bash
cat > .env.example << 'EOF'
# ============================================================
# PLANTILLA DE VARIABLES DE ENTORNO - lab-02-contentful-sdk
# ============================================================
# Copia este archivo como .env y rellena tus valores reales.
# NUNCA commitees el archivo .env al repositorio.
# ============================================================

# Contentful - Space ID
# Encuéntralo en: Settings → General Settings
CONTENTFUL_SPACE_ID=your_space_id_here

# Contentful - Content Delivery API Key (solo lectura, contenido publicado)
# Encuéntrala en: Settings → API Keys → [Tu API Key] → Content Delivery API
CONTENTFUL_DELIVERY_TOKEN=your_delivery_token_here

# Contentful - Content Preview API Key (solo lectura, borradores)
# Encuéntrala en: Settings → API Keys → [Tu API Key] → Content Preview API
CONTENTFUL_PREVIEW_TOKEN=your_preview_token_here

# Contentful - Entorno (normalmente 'master' en plan gratuito)
CONTENTFUL_ENVIRONMENT=master
EOF
```

6. Crea el archivo `.env` real con tus credenciales. Abre VS Code:

```bash
code .
```

Crea un nuevo archivo llamado `.env` (en la raíz del proyecto) y rellena tus valores reales:

```
CONTENTFUL_SPACE_ID=xxxxxxxxxxxxxxxxx
CONTENTFUL_DELIVERY_TOKEN=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
CONTENTFUL_PREVIEW_TOKEN=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
CONTENTFUL_ENVIRONMENT=master
```

> **¿Dónde encontrar estos valores?**
> - Accede a [app.contentful.com](https://app.contentful.com)
> - Ve a **Settings → API Keys**
> - Selecciona tu API Key existente (o crea una nueva)
> - Copia el **Space ID**, el **Content Delivery API - access token** y el **Content Preview API - access token**

7. Crea la estructura de carpetas del proyecto:

```bash
mkdir src
mkdir src/config
mkdir src/services
mkdir src/utils
touch src/config/contentful.js
touch src/services/contentService.js
touch src/utils/errorHandler.js
touch src/utils/logger.js
touch index.js
```

8. Realiza el primer commit (solo archivos seguros):

```bash
git add .gitignore .env.example
git status   # Verifica que .env NO aparece en la lista
git commit -m "chore: inicializar proyecto con configuración de seguridad"
```

**Salida Esperada del `git status`:**

```
On branch main
Changes to be committed:
  new file:   .gitignore
  new file:   .env.example

Untracked files:
  (use "git add <file>..." to include in what you want to commit)
        src/
        index.js
```

> El archivo `.env` **no debe aparecer** en `Changes to be committed`. Si aparece, detente y verifica tu `.gitignore`.

**Verificación:**

```bash
# Confirmar que .env está siendo ignorado por Git
git check-ignore -v .env
# Salida esperada: .gitignore:3:.env    .env
```

---

### Paso 2 — Instalar Dependencias e Inicializar el Cliente del SDK

**Objetivo:** Instalar el SDK oficial de Contentful y `dotenv`, luego crear el módulo de configuración que inicializa el cliente una sola vez para ser reutilizado en todo el proyecto.

**Instrucciones:**

1. Instala las dependencias necesarias:

```bash
npm install contentful dotenv
```

2. Verifica las versiones instaladas:

```bash
npm list contentful dotenv
```

Debes ver `contentful@10.x.x` y `dotenv@16.x.x` (o superior).

3. Abre `src/config/contentful.js` en VS Code y escribe el siguiente código:

```javascript
// src/config/contentful.js
// ============================================================
// Módulo de configuración del cliente Contentful SDK v10.x
// Este módulo inicializa el cliente UNA SOLA VEZ y lo exporta
// para ser reutilizado en todos los servicios del proyecto.
// ============================================================

import * as contentful from 'contentful';
import 'dotenv/config';

// Validación de variables de entorno obligatorias al arrancar
const requiredEnvVars = [
  'CONTENTFUL_SPACE_ID',
  'CONTENTFUL_DELIVERY_TOKEN',
  'CONTENTFUL_ENVIRONMENT',
];

for (const varName of requiredEnvVars) {
  if (!process.env[varName]) {
    throw new Error(
      `[Config] Variable de entorno requerida no encontrada: ${varName}\n` +
      `Asegúrate de que el archivo .env existe y contiene esta variable.`
    );
  }
}

// ============================================================
// Cliente de Content Delivery API (contenido publicado)
// Usar en: producción, frontend público
// ============================================================
export const deliveryClient = contentful.createClient({
  space: process.env.CONTENTFUL_SPACE_ID,
  accessToken: process.env.CONTENTFUL_DELIVERY_TOKEN,
  environment: process.env.CONTENTFUL_ENVIRONMENT || 'master',
});

// ============================================================
// Cliente de Content Preview API (borradores)
// Usar en: entornos de previsualización, staging
// NOTA: Este cliente usa el host de preview, NO el de delivery
// ============================================================
export const previewClient = contentful.createClient({
  space: process.env.CONTENTFUL_SPACE_ID,
  accessToken: process.env.CONTENTFUL_PREVIEW_TOKEN || process.env.CONTENTFUL_DELIVERY_TOKEN,
  environment: process.env.CONTENTFUL_ENVIRONMENT || 'master',
  host: 'preview.contentful.com',   // Host diferente para Preview API
});

// Exportar configuración para logging (sin exponer tokens)
export const clientConfig = {
  spaceId: process.env.CONTENTFUL_SPACE_ID,
  environment: process.env.CONTENTFUL_ENVIRONMENT || 'master',
  // NUNCA incluir tokens en logs ni en objetos de configuración exportados
};
```

4. Actualiza el `package.json` para usar ES Modules (necesario para la sintaxis `import`):

```bash
# Abre package.json y agrega "type": "module"
```

Edita `package.json` para que luzca así (los valores de `name`, `version`, etc. pueden variar):

```json
{
  "name": "lab-02-contentful-sdk",
  "version": "1.0.0",
  "description": "Librería de utilidades para consumo de la API de Contentful",
  "main": "index.js",
  "type": "module",
  "scripts": {
    "start": "node index.js",
    "dev": "node --watch index.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "contentful": "^10.0.0",
    "dotenv": "^16.0.0"
  }
}
```

5. Crea el módulo de logger en `src/utils/logger.js`:

```javascript
// src/utils/logger.js
// Logger estructurado simple para el proyecto.
// En producción se reemplazaría por Winston o Pino.

const LOG_LEVELS = { ERROR: 'ERROR', WARN: 'WARN', INFO: 'INFO', DEBUG: 'DEBUG' };

function formatMessage(level, message, meta = {}) {
  const timestamp = new Date().toISOString();
  const metaStr = Object.keys(meta).length > 0 ? ` | ${JSON.stringify(meta)}` : '';
  return `[${timestamp}] [${level}] ${message}${metaStr}`;
}

export const logger = {
  error: (message, meta) => console.error(formatMessage(LOG_LEVELS.ERROR, message, meta)),
  warn:  (message, meta) => console.warn(formatMessage(LOG_LEVELS.WARN,  message, meta)),
  info:  (message, meta) => console.log(formatMessage(LOG_LEVELS.INFO,   message, meta)),
  debug: (message, meta) => console.log(formatMessage(LOG_LEVELS.DEBUG,  message, meta)),
};
```

6. Haz un commit del progreso:

```bash
git add src/ package.json
git commit -m "feat: inicializar cliente SDK de Contentful con configuración segura"
```

**Salida Esperada al ejecutar `npm list contentful`:**

```
lab-02-contentful-sdk@1.0.0
└── contentful@10.x.x
```

**Verificación:**

```bash
# Prueba rápida de que el cliente se inicializa sin errores
node -e "import('./src/config/contentful.js').then(m => console.log('✅ Cliente inicializado. Space:', m.clientConfig.spaceId))"
# Salida esperada: ✅ Cliente inicializado. Space: xxxxxxxxxxxxxxxxx
```

---

### Paso 3 — Implementar el Manejador de Errores Diferenciado

**Objetivo:** Crear un módulo de manejo de errores que identifique los códigos HTTP relevantes de la API de Contentful (401, 404, 429) y aplique retry con backoff exponencial para el caso de rate limiting.

**Instrucciones:**

1. Abre `src/utils/errorHandler.js` y escribe el siguiente código:

```javascript
// src/utils/errorHandler.js
// ============================================================
// Manejador de errores diferenciado para la API de Contentful.
// Identifica códigos HTTP relevantes y aplica estrategias
// de recuperación según el tipo de error.
// ============================================================

import { logger } from './logger.js';

// Códigos de error relevantes de la API de Contentful
export const HTTP_CODES = {
  UNAUTHORIZED:    401,  // Token inválido o ausente
  NOT_FOUND:       404,  // Recurso (entry, space, content type) no existe
  RATE_LIMITED:    429,  // Se superó el límite de 7 req/seg en plan gratuito
  SERVER_ERROR:    500,  // Error interno del servidor de Contentful
};

// ============================================================
// Función principal de manejo de errores
// Recibe el error del SDK y lanza un error descriptivo
// ============================================================
export function handleContentfulError(error, context = '') {
  const prefix = context ? `[${context}]` : '[Contentful]';

  // El SDK de Contentful v10.x expone el status HTTP en error.status
  const status = error?.status || error?.response?.status;

  switch (status) {
    case HTTP_CODES.UNAUTHORIZED:
      logger.error(`${prefix} Error de autenticación (401). Verifica tu API Key.`, {
        spaceId: error?.sys?.space?.sys?.id,
        message: error?.message,
      });
      throw new ContentfulAuthError(
        `Autenticación fallida: token inválido o sin permisos para este espacio. ` +
        `Verifica CONTENTFUL_DELIVERY_TOKEN en tu archivo .env`
      );

    case HTTP_CODES.NOT_FOUND:
      logger.warn(`${prefix} Recurso no encontrado (404).`, {
        message: error?.message,
      });
      throw new ContentfulNotFoundError(
        `El recurso solicitado no existe en Contentful. ` +
        `Verifica el ID de la entrada o el nombre del content type.`
      );

    case HTTP_CODES.RATE_LIMITED:
      logger.warn(`${prefix} Rate limit alcanzado (429). Aplicando backoff...`, {
        retryAfter: error?.response?.headers?.get?.('x-contentful-ratelimit-reset'),
      });
      // No lanzamos error aquí; el retry se maneja en withRetry()
      throw new ContentfulRateLimitError(
        `Se superó el límite de peticiones (7 req/seg en plan Community). ` +
        `La función withRetry() reintentará automáticamente.`
      );

    default:
      logger.error(`${prefix} Error inesperado de la API.`, {
        status,
        message: error?.message,
      });
      throw new Error(`Error de Contentful (${status || 'desconocido'}): ${error?.message}`);
  }
}

// ============================================================
// Clases de error personalizadas para manejo granular
// ============================================================
export class ContentfulAuthError extends Error {
  constructor(message) {
    super(message);
    this.name = 'ContentfulAuthError';
    this.status = 401;
  }
}

export class ContentfulNotFoundError extends Error {
  constructor(message) {
    super(message);
    this.name = 'ContentfulNotFoundError';
    this.status = 404;
  }
}

export class ContentfulRateLimitError extends Error {
  constructor(message) {
    super(message);
    this.name = 'ContentfulRateLimitError';
    this.status = 429;
  }
}

// ============================================================
// Función de retry con backoff exponencial
// Reintenta automáticamente en caso de rate limiting (429)
//
// Parámetros:
//   fn         - Función async que puede lanzar ContentfulRateLimitError
//   maxRetries - Número máximo de reintentos (default: 3)
//   baseDelay  - Delay base en ms (default: 1000ms = 1 segundo)
// ============================================================
export async function withRetry(fn, maxRetries = 3, baseDelay = 1000) {
  let lastError;

  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error;

      // Solo reintentamos en caso de rate limiting
      if (error instanceof ContentfulRateLimitError && attempt < maxRetries) {
        // Backoff exponencial: 1s, 2s, 4s, 8s...
        const delay = baseDelay * Math.pow(2, attempt - 1);
        logger.warn(`[withRetry] Intento ${attempt}/${maxRetries} fallido. Reintentando en ${delay}ms...`);
        await sleep(delay);
        continue;
      }

      // Para otros errores o si agotamos los reintentos, relanzamos
      throw error;
    }
  }

  throw lastError;
}

// Utilidad interna: esperar N milisegundos
function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

**Salida Esperada (sin errores de sintaxis):** El archivo se guarda sin errores en VS Code. ESLint no debe mostrar advertencias críticas.

**Verificación:**

```bash
# Verificar que el módulo se importa correctamente
node -e "
import('./src/utils/errorHandler.js').then(m => {
  console.log('✅ Clases exportadas:', Object.keys(m).join(', '));
}).catch(e => console.error('❌ Error:', e.message));
"
# Salida esperada: ✅ Clases exportadas: HTTP_CODES, handleContentfulError, ContentfulAuthError, ContentfulNotFoundError, ContentfulRateLimitError, withRetry
```

---

### Paso 4 — Implementar el Servicio de Contenido con Filtros y Búsqueda

**Objetivo:** Crear las funciones principales del servicio de contenido: obtener todas las entradas de un content type, buscar por campo específico y obtener una entrada por ID, usando el SDK de Contentful con manejo de errores integrado.

**Instrucciones:**

1. Abre `src/services/contentService.js` y escribe el siguiente código:

```javascript
// src/services/contentService.js
// ============================================================
// Servicio de acceso a contenido de Contentful.
// Encapsula todas las operaciones de lectura usando el SDK v10.x.
// Este módulo es reutilizable en las prácticas 3, 4 y 5.
// ============================================================

import { deliveryClient, clientConfig } from '../config/contentful.js';
import { handleContentfulError, withRetry } from '../utils/errorHandler.js';
import { logger } from '../utils/logger.js';

// ============================================================
// FUNCIÓN 1: Obtener todas las entradas de un Content Type
//
// Parámetros:
//   contentType - El API ID del content type (ej: 'article', 'blogPost')
//   options     - Opciones adicionales: { select, order, include }
//
// Retorna: Array de entradas con sus campos
// ============================================================
export async function getEntriesByType(contentType, options = {}) {
  logger.info('[contentService] Obteniendo entradas por content type', {
    contentType,
    space: clientConfig.spaceId,
    environment: clientConfig.environment,
  });

  return withRetry(async () => {
    try {
      const response = await deliveryClient.getEntries({
        content_type: contentType,     // Filtro obligatorio por content type
        include: options.include ?? 2, // Nivel de resolución de referencias (0-10)
        order: options.order ?? '-sys.createdAt', // Más recientes primero
        ...options.extraParams,        // Parámetros adicionales opcionales
      });

      logger.info('[contentService] Entradas obtenidas exitosamente', {
        contentType,
        total: response.total,
        returned: response.items.length,
        skip: response.skip,
        limit: response.limit,
      });

      return {
        items: response.items,
        total: response.total,
        skip: response.skip,
        limit: response.limit,
      };
    } catch (error) {
      handleContentfulError(error, `getEntriesByType(${contentType})`);
    }
  });
}

// ============================================================
// FUNCIÓN 2: Buscar entradas por valor de un campo específico
//
// Parámetros:
//   contentType - El API ID del content type
//   fieldName   - Nombre del campo a filtrar (ej: 'category', 'author')
//   fieldValue  - Valor a buscar en ese campo
//
// Retorna: Array de entradas que coinciden con el filtro
//
// NOTA: Para campos de tipo referencia (Link), usar
//       fields.fieldName.sys.id en lugar de fields.fieldName
// ============================================================
export async function getEntriesByField(contentType, fieldName, fieldValue) {
  logger.info('[contentService] Buscando entradas por campo', {
    contentType,
    fieldName,
    fieldValue,
  });

  return withRetry(async () => {
    try {
      // Construcción dinámica del filtro de campo
      // El SDK v10.x usa la notación: fields.NOMBRE_CAMPO
      const queryParams = {
        content_type: contentType,
        [`fields.${fieldName}`]: fieldValue,
        include: 1,
      };

      const response = await deliveryClient.getEntries(queryParams);

      logger.info('[contentService] Búsqueda por campo completada', {
        fieldName,
        fieldValue,
        resultCount: response.items.length,
      });

      return response.items;
    } catch (error) {
      handleContentfulError(error, `getEntriesByField(${contentType}, ${fieldName})`);
    }
  });
}

// ============================================================
// FUNCIÓN 3: Obtener una entrada específica por su Entry ID
//
// Parámetros:
//   entryId - El ID único de la entrada en Contentful
//   include - Nivel de resolución de referencias (default: 2)
//
// Retorna: El objeto de entrada completo con sus campos
// ============================================================
export async function getEntryById(entryId, include = 2) {
  logger.info('[contentService] Obteniendo entrada por ID', { entryId });

  return withRetry(async () => {
    try {
      // getEntry() obtiene una entrada específica por su sys.id
      const entry = await deliveryClient.getEntry(entryId, { include });

      logger.info('[contentService] Entrada obtenida exitosamente', {
        entryId,
        contentType: entry.sys.contentType.sys.id,
        updatedAt: entry.sys.updatedAt,
      });

      return entry;
    } catch (error) {
      handleContentfulError(error, `getEntryById(${entryId})`);
    }
  });
}
```

2. Guarda el archivo. Verifica en VS Code que no hay errores de sintaxis (ESLint no debe mostrar subrayados rojos).

**Verificación:**

```bash
# Prueba de importación del servicio
node -e "
import('./src/services/contentService.js').then(m => {
  const fns = Object.keys(m).filter(k => typeof m[k] === 'function');
  console.log('✅ Funciones exportadas:', fns.join(', '));
});
"
# Salida esperada: ✅ Funciones exportadas: getEntriesByType, getEntriesByField, getEntryById
```

---

### Paso 5 — Implementar Paginación Completa con `skip` y `limit`

**Objetivo:** Crear una función de paginación que recupere **todas** las entradas de un content type en lotes de tamaño configurable, usando los parámetros `skip` y `limit` de la API REST de Contentful.

**Instrucciones:**

1. Agrega las siguientes funciones al final de `src/services/contentService.js`:

```javascript
// ============================================================
// FUNCIÓN 4: Obtener una página específica de entradas
//
// Parámetros:
//   contentType - El API ID del content type
//   page        - Número de página (base 1, ej: 1, 2, 3...)
//   pageSize    - Entradas por página (default: 5, máx: 1000)
//
// Retorna: { items, total, page, pageSize, totalPages, hasNextPage }
// ============================================================
export async function getEntriesPage(contentType, page = 1, pageSize = 5) {
  // Validación de parámetros
  if (page < 1) throw new Error('El número de página debe ser mayor o igual a 1');
  if (pageSize < 1 || pageSize > 1000) throw new Error('pageSize debe estar entre 1 y 1000');

  // Cálculo de skip: para página 1 skip=0, página 2 skip=5, página 3 skip=10...
  const skip = (page - 1) * pageSize;

  logger.info('[contentService] Obteniendo página de entradas', {
    contentType, page, pageSize, skip,
  });

  return withRetry(async () => {
    try {
      const response = await deliveryClient.getEntries({
        content_type: contentType,
        skip,           // Número de entradas a saltar desde el inicio
        limit: pageSize, // Número máximo de entradas a retornar
        order: 'sys.createdAt',
      });

      const totalPages = Math.ceil(response.total / pageSize);

      logger.info('[contentService] Página obtenida', {
        page, totalPages, returned: response.items.length, total: response.total,
      });

      return {
        items: response.items,
        total: response.total,
        page,
        pageSize,
        totalPages,
        hasNextPage: page < totalPages,
        hasPrevPage: page > 1,
      };
    } catch (error) {
      handleContentfulError(error, `getEntriesPage(${contentType}, page=${page})`);
    }
  });
}

// ============================================================
// FUNCIÓN 5: Recuperar TODAS las entradas usando paginación
// (útil cuando el total supera el límite máximo de 1000)
//
// Parámetros:
//   contentType - El API ID del content type
//   batchSize   - Tamaño de cada lote (default: 5 para esta práctica)
//
// Retorna: Array con TODAS las entradas del content type
//
// ADVERTENCIA: En espacios con miles de entradas, esta función
// puede hacer muchas llamadas a la API. Úsala con precaución
// y respeta el rate limit de 7 req/seg del plan Community.
// ============================================================
export async function getAllEntriesPaginated(contentType, batchSize = 5) {
  logger.info('[contentService] Iniciando recuperación paginada completa', {
    contentType, batchSize,
  });

  const allItems = [];
  let currentPage = 1;
  let hasMore = true;

  while (hasMore) {
    // Pausa entre peticiones para respetar el rate limit (7 req/seg)
    // En plan Community: esperar al menos 150ms entre llamadas
    if (currentPage > 1) {
      await sleep(200); // 200ms de pausa = máximo ~5 req/seg (seguro)
    }

    const pageResult = await getEntriesPage(contentType, currentPage, batchSize);

    allItems.push(...pageResult.items);
    hasMore = pageResult.hasNextPage;
    currentPage++;

    logger.debug('[contentService] Progreso de paginación', {
      paginaActual: currentPage - 1,
      totalPaginas: pageResult.totalPages,
      entradasRecuperadas: allItems.length,
      totalEsperado: pageResult.total,
    });
  }

  logger.info('[contentService] Paginación completa', {
    contentType,
    totalRecuperado: allItems.length,
    lotesTotales: currentPage - 1,
  });

  return allItems;
}

// Utilidad interna para pausas entre peticiones
function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

**Salida Esperada:** El archivo se guarda sin errores. La función `getAllEntriesPaginated` implementa un bucle `while` que itera hasta que `hasNextPage` sea `false`.

**Verificación:**

```bash
# Verificar que todas las funciones están exportadas
node -e "
import('./src/services/contentService.js').then(m => {
  const fns = Object.keys(m).filter(k => typeof m[k] === 'function');
  console.log('✅ Total de funciones exportadas:', fns.length);
  fns.forEach(fn => console.log('  -', fn));
});
"
# Salida esperada:
# ✅ Total de funciones exportadas: 5
#   - getEntriesByType
#   - getEntriesByField
#   - getEntryById
#   - getEntriesPage
#   - getAllEntriesPaginated
```

---

### Paso 6 — Crear el Script Principal de Demostración

**Objetivo:** Integrar todas las funciones creadas en un script `index.js` que demuestre el funcionamiento completo del proyecto: autenticación, consultas con filtros, paginación y manejo de errores.

**Instrucciones:**

1. Abre `index.js` en la raíz del proyecto y escribe el siguiente código:

```javascript
// index.js
// ============================================================
// Script principal de demostración del Lab 02-00-01
// Ejecuta: node index.js
// ============================================================

import 'dotenv/config';
import { logger } from './src/utils/logger.js';
import { clientConfig } from './src/config/contentful.js';
import {
  getEntriesByType,
  getEntriesByField,
  getEntryById,
  getEntriesPage,
  getAllEntriesPaginated,
} from './src/services/contentService.js';
import {
  ContentfulAuthError,
  ContentfulNotFoundError,
  ContentfulRateLimitError,
} from './src/utils/errorHandler.js';

// ============================================================
// IMPORTANTE: Reemplaza estos valores con los de tu espacio.
// El CONTENT_TYPE debe coincidir con el API ID del content type
// que creaste en la Práctica 1 (ej: 'article', 'blogPost', etc.)
// ============================================================
const CONTENT_TYPE = 'article';       // ← Cambia al API ID de tu content type
const FIELD_NAME   = 'category';      // ← Cambia a un campo existente en tu CT
const FIELD_VALUE  = 'tecnología';    // ← Cambia a un valor real de ese campo
// Para obtener un Entry ID real: ejecuta primero la DEMO 1 y copia un sys.id

async function main() {
  console.log('\n' + '═'.repeat(60));
  console.log('  Lab 02-00-01 — Consumo de API REST de Contentful');
  console.log('═'.repeat(60));
  logger.info('Iniciando demostración', {
    space: clientConfig.spaceId,
    environment: clientConfig.environment,
  });

  // ----------------------------------------------------------
  // DEMO 1: Obtener todas las entradas de un content type
  // ----------------------------------------------------------
  console.log('\n📋 DEMO 1: getEntriesByType()');
  console.log('─'.repeat(40));
  try {
    const result = await getEntriesByType(CONTENT_TYPE);
    console.log(`✅ Total de entradas en Contentful: ${result.total}`);
    console.log(`   Entradas retornadas en esta llamada: ${result.items.length}`);

    if (result.items.length > 0) {
      const primera = result.items[0];
      console.log(`   Primera entrada:`);
      console.log(`     ID:    ${primera.sys.id}`);
      console.log(`     Título: ${primera.fields.title || primera.fields.titulo || '(campo no encontrado)'}`);
      console.log(`     Creada: ${primera.sys.createdAt}`);

      // Guardamos el ID para la DEMO 3
      global._firstEntryId = primera.sys.id;
    }
  } catch (error) {
    handleDemoError('DEMO 1', error);
  }

  // ----------------------------------------------------------
  // DEMO 2: Buscar entradas por campo y valor
  // ----------------------------------------------------------
  console.log('\n🔍 DEMO 2: getEntriesByField()');
  console.log('─'.repeat(40));
  try {
    const items = await getEntriesByField(CONTENT_TYPE, FIELD_NAME, FIELD_VALUE);
    console.log(`✅ Entradas con ${FIELD_NAME}="${FIELD_VALUE}": ${items.length}`);
    items.forEach((item, idx) => {
      console.log(`   [${idx + 1}] ${item.fields.title || item.fields.titulo || item.sys.id}`);
    });
  } catch (error) {
    handleDemoError('DEMO 2', error);
  }

  // ----------------------------------------------------------
  // DEMO 3: Obtener una entrada específica por ID
  // ----------------------------------------------------------
  console.log('\n🎯 DEMO 3: getEntryById()');
  console.log('─'.repeat(40));
  try {
    const entryId = global._firstEntryId;
    if (!entryId) {
      console.log('⚠️  Saltando DEMO 3: no se obtuvo ningún ID en la DEMO 1');
    } else {
      const entry = await getEntryById(entryId);
      console.log(`✅ Entrada obtenida:`);
      console.log(`   ID:           ${entry.sys.id}`);
      console.log(`   Content Type: ${entry.sys.contentType.sys.id}`);
      console.log(`   Actualizada:  ${entry.sys.updatedAt}`);
      console.log(`   Campos:       ${Object.keys(entry.fields).join(', ')}`);
    }
  } catch (error) {
    handleDemoError('DEMO 3', error);
  }

  // ----------------------------------------------------------
  // DEMO 4: Paginación — obtener página específica
  // ----------------------------------------------------------
  console.log('\n📄 DEMO 4: getEntriesPage() — Página 1 de 5 entradas');
  console.log('─'.repeat(40));
  try {
    const pagina1 = await getEntriesPage(CONTENT_TYPE, 1, 5);
    console.log(`✅ Página 1/${pagina1.totalPages}`);
    console.log(`   Entradas en esta página: ${pagina1.items.length}`);
    console.log(`   Total en el espacio:     ${pagina1.total}`);
    console.log(`   ¿Hay página siguiente?   ${pagina1.hasNextPage ? 'Sí' : 'No'}`);

    if (pagina1.hasNextPage) {
      // Pausa breve para respetar rate limit
      await new Promise(r => setTimeout(r, 200));
      const pagina2 = await getEntriesPage(CONTENT_TYPE, 2, 5);
      console.log(`\n   Página 2/${pagina2.totalPages}:`);
      console.log(`   Entradas en esta página: ${pagina2.items.length}`);
    }
  } catch (error) {
    handleDemoError('DEMO 4', error);
  }

  // ----------------------------------------------------------
  // DEMO 5: Paginación completa — recuperar TODAS las entradas
  // ----------------------------------------------------------
  console.log('\n🔄 DEMO 5: getAllEntriesPaginated() — Recuperar todo');
  console.log('─'.repeat(40));
  try {
    const todas = await getAllEntriesPaginated(CONTENT_TYPE, 5);
    console.log(`✅ Total de entradas recuperadas: ${todas.length}`);
    console.log(`   IDs recuperados:`);
    todas.forEach((entry, idx) => {
      const titulo = entry.fields.title || entry.fields.titulo || '(sin título)';
      console.log(`   [${idx + 1}] ${entry.sys.id} — ${titulo}`);
    });
  } catch (error) {
    handleDemoError('DEMO 5', error);
  }

  // ----------------------------------------------------------
  // DEMO 6: Simulación de manejo de errores
  // ----------------------------------------------------------
  console.log('\n⚠️  DEMO 6: Manejo de errores — ID inexistente');
  console.log('─'.repeat(40));
  try {
    await getEntryById('este-id-no-existe-12345');
  } catch (error) {
    if (error instanceof ContentfulNotFoundError) {
      console.log(`✅ Error 404 capturado correctamente: ${error.message}`);
    } else {
      console.log(`❌ Error inesperado: ${error.message}`);
    }
  }

  console.log('\n' + '═'.repeat(60));
  console.log('  ✅ Demostración completada exitosamente');
  console.log('═'.repeat(60) + '\n');
}

// Función auxiliar para mostrar errores en las demos sin detener el script
function handleDemoError(demoName, error) {
  if (error instanceof ContentfulAuthError) {
    console.log(`❌ [${demoName}] Error de autenticación: ${error.message}`);
    console.log('   → Verifica tu CONTENTFUL_DELIVERY_TOKEN en el archivo .env');
  } else if (error instanceof ContentfulNotFoundError) {
    console.log(`⚠️  [${demoName}] Recurso no encontrado: ${error.message}`);
    console.log(`   → Verifica que el content type '${CONTENT_TYPE}' existe en tu espacio`);
  } else if (error instanceof ContentfulRateLimitError) {
    console.log(`⏳ [${demoName}] Rate limit alcanzado: ${error.message}`);
  } else {
    console.log(`❌ [${demoName}] Error: ${error.message}`);
  }
}

// Punto de entrada
main().catch(error => {
  logger.error('Error fatal en main()', { message: error.message, stack: error.stack });
  process.exit(1);
});
```

2. **Ajusta las constantes** al inicio del archivo `index.js` según tu espacio:
   - `CONTENT_TYPE`: el **API ID** de tu content type (lo encuentras en Contentful → Content Model → [tu tipo] → clic en el nombre → API Identifier)
   - `FIELD_NAME` y `FIELD_VALUE`: un campo y valor reales de tus entradas

3. Ejecuta el script:

```bash
node index.js
```

**Salida Esperada:**

```
════════════════════════════════════════════════════════════
  Lab 02-00-01 — Consumo de API REST de Contentful
════════════════════════════════════════════════════════════
[2024-01-15T10:30:00.000Z] [INFO] Iniciando demostración | {"space":"xxxxxxxxx","environment":"master"}

📋 DEMO 1: getEntriesByType()
────────────────────────────────────────
[2024-01-15T10:30:00.123Z] [INFO] [contentService] Obteniendo entradas por content type | {"contentType":"article",...}
[2024-01-15T10:30:00.456Z] [INFO] [contentService] Entradas obtenidas exitosamente | {"total":12,"returned":12,...}
✅ Total de entradas en Contentful: 12
   Entradas retornadas en esta llamada: 12
   Primera entrada:
     ID:    abc123def456
     Título: Mi Primer Artículo
     Creada: 2024-01-10T09:00:00.000Z

📋 DEMO 2: getEntriesByField()
...

✅ Demostración completada exitosamente
════════════════════════════════════════════════════════════
```

**Verificación:**

```bash
# El script debe terminar con código de salida 0
echo "Código de salida: $?"
# Salida esperada: Código de salida: 0
```

---

### Paso 7 — Commit Final y Revisión de Seguridad

**Objetivo:** Asegurar que el proyecto completo está correctamente versionado y que ninguna credencial ha sido expuesta en el repositorio Git.

**Instrucciones:**

1. Verifica el estado de Git antes del commit final:

```bash
git status
```

2. Confirma que `.env` **no aparece** en la lista de archivos a commitear.

3. Agrega todos los archivos del proyecto (excepto los ignorados):

```bash
git add .
git status   # Revisión final antes del commit
```

4. Realiza el commit final:

```bash
git commit -m "feat: implementar librería de utilidades de acceso a Contentful

- Configuración segura con dotenv y validación de variables de entorno
- Cliente SDK inicializado una sola vez (patrón singleton)
- getEntriesByType: consulta con filtro por content type
- getEntriesByField: búsqueda por campo y valor
- getEntryById: obtención de entrada específica
- getEntriesPage: paginación con skip/limit
- getAllEntriesPaginated: recuperación completa con pausa entre lotes
- Manejo diferenciado de errores 401, 404 y 429
- Retry con backoff exponencial para rate limiting
- Logger estructurado con timestamp e información de contexto"
```

5. Verifica el historial de commits:

```bash
git log --oneline
```

**Salida Esperada de `git log --oneline`:**

```
a1b2c3d (HEAD -> main) feat: implementar librería de utilidades de acceso a Contentful
e4f5g6h feat: inicializar cliente SDK de Contentful con configuración segura
i7j8k9l chore: inicializar proyecto con configuración de seguridad
```

**Verificación de Seguridad Final:**

```bash
# Buscar si algún token fue accidentalmente incluido en el historial de Git
git log --all --full-history -- .env
# Salida esperada: (vacío — .env nunca fue commiteado)

# Verificar que .env está en .gitignore
grep "^\.env$" .gitignore
# Salida esperada: .env
```

---

## Validación y Pruebas

Ejecuta las siguientes verificaciones para confirmar que el laboratorio está completo y funcional:

### Prueba 1 — Estructura del Proyecto

```bash
# Verificar que todos los archivos existen
ls -la src/config/contentful.js \
       src/services/contentService.js \
       src/utils/errorHandler.js \
       src/utils/logger.js \
       index.js \
       .env.example \
       .gitignore
# Todos los archivos deben existir (sin errores "No such file")
```

### Prueba 2 — Autenticación Correcta

```bash
# Prueba directa de autenticación con curl
source .env 2>/dev/null || export $(cat .env | xargs)
curl -s -o /dev/null -w "HTTP Status: %{http_code}\n" \
  -H "Authorization: Bearer $CONTENTFUL_DELIVERY_TOKEN" \
  "https://cdn.contentful.com/spaces/$CONTENTFUL_SPACE_ID/environments/$CONTENTFUL_ENVIRONMENT/entries?limit=1"
# Salida esperada: HTTP Status: 200
```

### Prueba 3 — Paginación Correcta

```bash
# Ejecutar solo la función de paginación
node -e "
import('./src/services/contentService.js').then(async ({ getEntriesPage }) => {
  const p1 = await getEntriesPage('article', 1, 3);
  console.log('Página 1:', p1.items.length, 'entradas, total:', p1.total);
  console.log('Páginas totales:', p1.totalPages);
  console.log('hasNextPage:', p1.hasNextPage);
  console.log('✅ Paginación funciona correctamente');
}).catch(e => console.error('❌', e.message));
"
```

### Prueba 4 — Manejo de Error 404

```bash
# Verificar que el error 404 se captura correctamente
node -e "
import('./src/services/contentService.js').then(async ({ getEntryById }) => {
  try {
    await getEntryById('id-que-no-existe-99999');
    console.log('❌ Debería haber lanzado un error');
  } catch (e) {
    if (e.name === 'ContentfulNotFoundError') {
      console.log('✅ Error 404 manejado correctamente:', e.name);
    } else {
      console.log('⚠️ Error capturado pero tipo incorrecto:', e.name, e.message);
    }
  }
});
"
```

### Prueba 5 — Seguridad de Credenciales

```bash
# Verificar que el token no está en ningún archivo commiteado
git show HEAD --stat | grep -v "\.env"
git grep -l "CONTENTFUL_DELIVERY_TOKEN" -- ':!.env.example' ':!.gitignore'
# Salida esperada: (vacío) — ningún archivo commiteado contiene el token real
```

### Checklist de Validación Final

Marca cada ítem como completado antes de entregar el laboratorio:

- [ ] `node index.js` ejecuta sin errores fatales y muestra resultados de las 6 demos
- [ ] El archivo `.env` **no** aparece en `git log` ni en `git status`
- [ ] La función `getEntriesByType()` retorna entradas con `total` y `items`
- [ ] La función `getEntriesByField()` filtra correctamente por campo y valor
- [ ] La función `getEntryById()` retorna la entrada con `sys.id` correcto
- [ ] La función `getEntriesPage()` retorna `{ items, total, page, totalPages, hasNextPage }`
- [ ] La función `getAllEntriesPaginated()` recupera todas las entradas en lotes de 5
- [ ] El error con ID inexistente lanza `ContentfulNotFoundError` (no un error genérico)
- [ ] El proyecto tiene al menos 3 commits con mensajes descriptivos
- [ ] El archivo `.env.example` está commiteado y documenta todas las variables necesarias

---

## Resolución de Problemas

### Problema 1 — Error `401 Unauthorized` al ejecutar el script

**Síntomas:**
```
[ERROR] [Contentful] Error de autenticación (401). Verifica tu API Key.
ContentfulAuthError: Autenticación fallida: token inválido o sin permisos para este espacio.
```

**Causa:**
El token en el archivo `.env` es incorrecto, está incompleto (tiene espacios o saltos de línea), o el archivo `.env` no existe en la ruta correcta. También puede ocurrir si se está usando el **Preview token** donde se requiere el **Delivery token**, o viceversa.

**Solución:**

1. Verifica que el archivo `.env` existe en la raíz del proyecto (donde está `package.json`):
   ```bash
   ls -la .env
   # Debe mostrar el archivo con tamaño > 0
   ```
2. Verifica el contenido del token (sin espacios ni comillas):
   ```bash
   # Ver el valor actual (solo para diagnóstico, nunca commitear)
   node -e "import('dotenv/config').then(() => console.log('Token length:', process.env.CONTENTFUL_DELIVERY_TOKEN?.length))"
   # El token de Contentful tiene entre 43 y 64 caracteres
   ```
3. Regresa a Contentful → **Settings → API Keys** → selecciona tu API Key → copia el token completo nuevamente y pégalo en `.env` sin espacios adicionales.
4. Confirma que estás usando el **Content Delivery API - access token** (no el Preview token) para el cliente de entrega.

---

### Problema 2 — Error `Cannot find module` o `SyntaxError: Cannot use import statement`

**Síntomas:**
```
Error [ERR_MODULE_NOT_FOUND]: Cannot find module '/ruta/src/config/contentful.js'
```
o
```
SyntaxError: Cannot use import statement in a module
```

**Causa:**
El proyecto no está configurado para usar ES Modules. El campo `"type": "module"` falta en `package.json`, o se está ejecutando el script con una versión de Node.js anterior a la 18.x que no soporta ES Modules de la misma forma.

**Solución:**

1. Verifica que `package.json` contiene `"type": "module"`:
   ```bash
   node -e "const p = JSON.parse(require('fs').readFileSync('package.json','utf8')); console.log('type:', p.type)"
   # Debe mostrar: type: module
   ```
   Si no está, agrégalo manualmente en `package.json`.

2. Verifica la versión de Node.js:
   ```bash
   node --version
   # Debe mostrar v18.x.x o superior
   ```
   Si tienes una versión anterior, actualiza Node.js usando `nvm`:
   ```bash
   nvm install 18
   nvm use 18
   ```

3. Si el error es `Cannot find module`, verifica que las rutas de importación en los archivos `.js` incluyen la extensión `.js` al final (requerido en ES Modules):
   ```javascript
   // ✅ Correcto en ES Modules
   import { logger } from './logger.js';
   
   // ❌ Incorrecto — falta la extensión
   import { logger } from './logger';
   ```

---

## Limpieza

Al finalizar el laboratorio, ejecuta los siguientes pasos para dejar el entorno ordenado:

```bash
# 1. Verificar que el proyecto está completamente commiteado
git status
# Salida esperada: "nothing to commit, working tree clean"

# 2. Verificar el historial final de commits
git log --oneline

# 3. (Opcional) Comprimir el proyecto para entrega
cd ..
zip -r lab-02-contentful-sdk.zip lab-02-contentful-sdk/ \
  --exclude "*/node_modules/*" \
  --exclude "*/.env"
echo "✅ Archivo de entrega creado: lab-02-contentful-sdk.zip"
```

> **⚠️ No elimines el proyecto.** Las prácticas 3, 4 y 5 reutilizan el espacio de Contentful y el código de `src/services/contentService.js` como base. Guarda el proyecto en un lugar accesible.

> **Nota sobre el Plan Community:** Has utilizado llamadas a la Delivery API durante este lab. Recuerda que el límite es de 7 peticiones/segundo. El parámetro `sleep(200ms)` en `getAllEntriesPaginated()` garantiza que no superes este límite (~5 req/seg).

---

## Resumen

En este laboratorio construiste una **librería de utilidades de acceso a contenido** completamente funcional que encapsula el consumo de la Content Delivery API de Contentful. Los logros principales fueron:

| Componente                    | Lo que construiste                                                            |
|-------------------------------|-------------------------------------------------------------------------------|
| **Seguridad de credenciales** | `.env` + `.gitignore` + `.env.example` — tokens nunca expuestos en Git        |
| **Configuración del cliente** | `src/config/contentful.js` — cliente SDK inicializado una sola vez            |
| **Consultas con filtros**     | `getEntriesByType()` y `getEntriesByField()` — filtrado por tipo y campo      |
| **Búsqueda por ID**           | `getEntryById()` — acceso directo a una entrada específica                    |
| **Paginación**                | `getEntriesPage()` y `getAllEntriesPaginated()` — manejo de grandes volúmenes |
| **Manejo de errores**         | Clases personalizadas para 401, 404, 429 con retry y backoff exponencial      |
| **Logging estructurado**      | `src/utils/logger.js` — trazabilidad con timestamp y contexto                 |

### Conceptos Clave Aplicados

- **Mínimo privilegio:** El cliente de Delivery API solo tiene acceso de lectura a contenido publicado. El Preview token se inicializa por separado para no mezclar contextos.
- **Patrón Singleton para el cliente SDK:** El cliente se crea una vez en `config/contentful.js` y se reutiliza en todos los servicios, evitando conexiones redundantes.
- **Paginación con `skip`/`limit`:** La fórmula `skip = (page - 1) * pageSize` es el patrón estándar para paginación basada en offset en APIs REST.
- **Backoff exponencial:** Los delays de 1s, 2s, 4s entre reintentos evitan saturar la API durante picos de tráfico.

### Recursos Adicionales

- [Documentación oficial del SDK de JavaScript de Contentful (v10.x)](https://contentful.github.io/contentful.js/)
- [Referencia completa de la Content Delivery API](https://www.contentful.com/developers/docs/references/content-delivery-api/)
- [Guía de autenticación en Contentful](https://www.contentful.com/developers/docs/references/authentication/)
- [Parámetros de búsqueda y filtros en la CDA](https://www.contentful.com/developers/docs/references/content-delivery-api/#/reference/search-parameters)
- [Límites y rate limiting del plan Community](https://www.contentful.com/developers/docs/technical-limits/)
- [Guía OWASP para manejo seguro de secretos en Node.js](https://cheatsheetseries.owasp.org/cheatsheets/Nodejs_Security_Cheat_Sheet.html)

---
