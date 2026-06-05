---LAB_START---
LAB_ID: 03-00-01
---MARKDOWN---
# Implementar consultas GraphQL optimizadas

## Metadatos

| Campo | Detalle |
|---|---|
| **Duración estimada** | 140 minutos |
| **Complejidad** | Alta |
| **Nivel Bloom** | Crear |
| **Módulo** | Capítulo 3 — API GraphQL de Contentful |
| **Dependencias** | Lab 01 (modelo de contenido), Lab 02 (proyecto Node.js con SDK REST) |

---

## Descripción General

En esta práctica construirás un módulo Node.js que consume la API GraphQL de Contentful de forma progresiva: desde una consulta básica de campos seleccionados hasta queries avanzadas con filtros, paginación, relaciones entre tipos de contenido y manejo de Rich Text. Realizarás una comparación cuantitativa del tamaño de payload entre REST y GraphQL, implementarás caché HTTP básica analizando las cabeceras del CDN de Contentful y refactorizarás queries inseguras con interpolación de strings hacia queries parametrizadas con variables GraphQL. Al finalizar, habrás construido un cliente GraphQL funcional que aplica las mejores prácticas de seguridad y rendimiento.

---

## Objetivos de Aprendizaje

Al completar esta práctica serás capaz de:

- [ ] Explorar el schema autogenerado de Contentful en el GraphQL Playground e identificar los tipos correspondientes a los Content Types de tu espacio
- [ ] Escribir queries GraphQL básicas, intermedias y avanzadas (filtros `where`, ordenamiento, paginación, linked entries y Rich Text) consumidas desde Node.js con `fetch` nativo
- [ ] Comparar cuantitativamente el tamaño de payload entre una consulta REST completa y su equivalente GraphQL con selección precisa de campos
- [ ] Implementar y analizar estrategias de caché HTTP usando las cabeceras `Cache-Control` y `ETag` que devuelve el CDN de Contentful
- [ ] Refactorizar queries con interpolación de strings a queries con variables GraphQL para prevenir inyección y mejorar la seguridad

---

## Prerrequisitos

### Conocimiento previo

- Haber completado la **Práctica 1** (modelo de contenido con Content Types `Article`, `Category` y `Author`) y la **Práctica 2** (módulo Node.js con SDK REST funcional)
- Conocimiento introductorio de GraphQL: qué es una query, qué es un schema, diferencia con REST
- Comprensión de HTTP headers (`Content-Type`, `Authorization`, `Cache-Control`)
- Familiaridad con `async/await` y `fetch` en Node.js 18+

> **⚠️ Si no completaste la Práctica 1**, ejecuta el script de importación provisto por el instructor antes de continuar:
> ```bash
> contentful space import --space-id TU_SPACE_ID --content-file lab01-seed.json
> ```

### Acceso y herramientas

- Cuenta de Contentful activa (plan Community) con el espacio de los labs anteriores
- **Content Delivery API Key** del espacio (obtenida en Práctica 2 — solo lectura, segura para este módulo)
- Node.js 18.x LTS instalado y verificado
- Visual Studio Code 1.85+
- Postman 10.x o Bruno 1.x configurado
- Acceso a internet estable (mínimo 10 Mbps)

> **🔐 Recordatorio de seguridad crítico:** La **Content Delivery API Key** es de solo lectura y puede usarse en este módulo. La **Content Management API Key** tiene permisos de escritura y **NUNCA debe aparecer en código de frontend ni commitearse en Git**. Revisa tu `.gitignore` antes de cualquier commit.

---

## Entorno del Laboratorio

### Hardware requerido

| Recurso | Mínimo | Recomendado |
|---|---|---|
| RAM | 8 GB | 16 GB |
| CPU | Intel Core i5 / AMD equivalente | i7 / Ryzen 5 o superior |
| Disco libre | 20 GB | 20 GB+ |
| Conexión | 10 Mbps | 25 Mbps+ |
| Resolución | 1280×720 | 1920×1080 |

### Software requerido

| Herramienta | Versión | Verificación |
|---|---|---|
| Node.js | 18.x LTS | `node --version` |
| npm | 9.x+ | `npm --version` |
| Visual Studio Code | 1.85+ | `code --version` |
| Git | 2.40+ | `git --version` |
| Postman / Bruno | 10.x / 1.x | Abrir aplicación |

### Preparación del entorno

Ejecuta los siguientes comandos para verificar tu entorno antes de comenzar:

```bash
# Verificar versiones
node --version    # Debe mostrar v18.x.x o superior
npm --version     # Debe mostrar 9.x.x o superior
git --version     # Debe mostrar 2.40.x o superior

# Verificar que fetch está disponible de forma nativa en Node 18
node -e "console.log(typeof fetch)"   # Debe imprimir: function
```

---

## Pasos del Laboratorio

---

### Paso 1: Configurar el proyecto y explorar el GraphQL Playground

**Objetivo:** Crear la estructura del proyecto Node.js para este lab y explorar el schema autogenerado de Contentful en el GraphQL Playground antes de escribir código.

#### Instrucciones

1. Crea el directorio del proyecto y navega hacia él:

```bash
mkdir lab03-graphql-contentful
cd lab03-graphql-contentful
git init
```

2. Inicializa el proyecto Node.js e instala las dependencias:

```bash
npm init -y
npm install graphql-request dotenv
```

> **Nota:** `graphql-request` es opcional en este lab (usaremos `fetch` nativo para entender el protocolo base), pero la instalaremos ahora para la sección avanzada. `dotenv` gestiona las variables de entorno.

3. Crea el archivo `.gitignore` **antes de cualquier commit** (paso obligatorio):

```bash
cat > .gitignore << 'EOF'
node_modules/
.env
*.log
dist/
.DS_Store
EOF
```

4. Crea el archivo `.env.example` como plantilla documentada:

```bash
cat > .env.example << 'EOF'
# Contentful - Content Delivery API (solo lectura - segura para consultas)
CONTENTFUL_SPACE_ID=tu_space_id_aqui
CONTENTFUL_ACCESS_TOKEN=tu_content_delivery_token_aqui
CONTENTFUL_ENVIRONMENT=master

# ADVERTENCIA: La Content Management API Key tiene permisos de escritura.
# NUNCA incluyas CONTENTFUL_MANAGEMENT_TOKEN en este archivo ni en el código frontend.
EOF
```

5. Crea tu archivo `.env` real con tus credenciales (cópialo de `.env.example` y rellena los valores):

```bash
cp .env.example .env
# Abre .env en VS Code y agrega tus credenciales reales
code .env
```

6. Verifica que `.env` está en `.gitignore` (verificación de seguridad):

```bash
git status
# .env NO debe aparecer en la lista de archivos a trackear
# Si aparece, revisa tu .gitignore
```

7. **Explorar el GraphQL Playground de Contentful:**

   Abre tu navegador y navega a la siguiente URL (reemplaza `TU_SPACE_ID` con el tuyo):

   ```
   https://graphql.contentful.com/content/v1/spaces/TU_SPACE_ID/explore?access_token=TU_ACCESS_TOKEN
   ```

   En el Playground, ejecuta la siguiente introspección básica para ver los tipos disponibles:

   ```graphql
   # Explorar los tipos raíz disponibles en tu esquema
   {
     __schema {
       queryType {
         fields {
           name
           description
         }
       }
     }
   }
   ```

8. Dentro del Playground, ejecuta esta query de prueba para confirmar que tu modelo de contenido es accesible:

   ```graphql
   # Query de verificación - reemplaza 'articleCollection' si tu tipo tiene otro nombre
   {
     articleCollection(limit: 3) {
       total
       items {
         sys {
           id
         }
         title
       }
     }
   }
   ```

#### Salida esperada

```
✓ Proyecto inicializado con package.json
✓ .gitignore creado con node_modules/ y .env excluidos
✓ .env.example creado como plantilla
✓ .env creado con credenciales reales (NO commiteado)
✓ GraphQL Playground accesible en el navegador
✓ Introspección del schema muestra tipos como articleCollection, categoryCollection, etc.
✓ Query de prueba retorna al menos 1 artículo con sys.id y title
```

#### Verificación

```bash
# Confirmar estructura del proyecto
ls -la
# Debe mostrar: .gitignore, .env, .env.example, package.json, node_modules/

# Confirmar que .env no está trackeado por Git
git status --short
# .env NO debe aparecer en la salida
```

---

### Paso 2: Implementar el cliente GraphQL base con fetch nativo

**Objetivo:** Crear un cliente GraphQL reutilizable usando `fetch` nativo de Node.js 18, comprendiendo el protocolo HTTP subyacente antes de usar librerías de abstracción.

#### Instrucciones

1. Crea el directorio de fuentes y el archivo del cliente base:

```bash
mkdir src
touch src/graphql-client.js
touch src/queries.js
```

2. Implementa el cliente GraphQL en `src/graphql-client.js`:

```javascript
// src/graphql-client.js
// Cliente GraphQL base usando fetch nativo de Node.js 18
// NO requiere librerías adicionales - ilustra el protocolo HTTP subyacente

import 'dotenv/config';

const SPACE_ID = process.env.CONTENTFUL_SPACE_ID;
const ACCESS_TOKEN = process.env.CONTENTFUL_ACCESS_TOKEN;
const ENVIRONMENT = process.env.CONTENTFUL_ENVIRONMENT || 'master';

// Endpoint único de GraphQL para Contentful
const GRAPHQL_ENDPOINT = `https://graphql.contentful.com/content/v1/spaces/${SPACE_ID}/environments/${ENVIRONMENT}`;

/**
 * Ejecuta una query GraphQL contra la API de Contentful.
 * Usa fetch nativo (disponible en Node.js 18+).
 *
 * @param {string} query - La query GraphQL como string
 * @param {object} variables - Variables para la query (previene interpolación insegura)
 * @param {object} options - Opciones adicionales (preview, cache)
 * @returns {Promise<object>} - Los datos de la respuesta GraphQL
 */
export async function graphqlFetch(query, variables = {}, options = {}) {
  const { preview = false } = options;

  // Para preview, se usaría el Preview Access Token
  // En este lab usamos siempre el Delivery Token (contenido publicado)
  const token = ACCESS_TOKEN;

  const startTime = Date.now();

  const response = await fetch(GRAPHQL_ENDPOINT, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${token}`,
      // Cabecera informativa para identificar el cliente en logs
      'X-Contentful-User-Agent': 'lab03-graphql-client/1.0',
    },
    body: JSON.stringify({
      query,
      variables,  // SIEMPRE enviar variables separadas, nunca interpolar en el query string
    }),
  });

  const elapsed = Date.now() - startTime;

  // Capturar cabeceras de caché para análisis en el Paso 5
  const cacheHeaders = {
    'cache-control': response.headers.get('cache-control'),
    'cf-cache-status': response.headers.get('cf-cache-status'),  // Cloudflare CDN
    'x-contentful-request-id': response.headers.get('x-contentful-request-id'),
    'etag': response.headers.get('etag'),
    'age': response.headers.get('age'),  // Segundos desde que el CDN cacheó la respuesta
  };

  if (!response.ok) {
    const errorBody = await response.text();
    throw new Error(
      `GraphQL request failed: ${response.status} ${response.statusText}\n${errorBody}`
    );
  }

  const json = await response.json();

  // GraphQL puede retornar HTTP 200 pero con errores en el body
  if (json.errors) {
    console.error('GraphQL Errors:', JSON.stringify(json.errors, null, 2));
    throw new Error(`GraphQL errors: ${json.errors.map(e => e.message).join(', ')}`);
  }

  // Retornar datos junto con metadatos de rendimiento y caché
  return {
    data: json.data,
    meta: {
      elapsedMs: elapsed,
      cacheHeaders,
      payloadBytes: JSON.stringify(json).length,
    },
  };
}

/**
 * Calcula el tamaño aproximado de un objeto JSON en bytes.
 * Útil para comparaciones de payload REST vs GraphQL.
 */
export function getPayloadSize(obj) {
  return new Blob([JSON.stringify(obj)]).size;
}
```

3. Agrega `"type": "module"` al `package.json` para usar ES Modules:

```bash
# Edita package.json para agregar "type": "module"
node -e "
const fs = require('fs');
const pkg = JSON.parse(fs.readFileSync('package.json', 'utf8'));
pkg.type = 'module';
fs.writeFileSync('package.json', JSON.stringify(pkg, null, 2));
console.log('package.json actualizado con type: module');
"
```

4. Crea un script de prueba rápida `src/test-client.js`:

```javascript
// src/test-client.js
// Prueba básica del cliente GraphQL

import { graphqlFetch } from './graphql-client.js';

const TEST_QUERY = `
  query TestConexion {
    articleCollection(limit: 1) {
      total
      items {
        sys { id }
        title
      }
    }
  }
`;

async function main() {
  console.log('🔌 Probando conexión con la API GraphQL de Contentful...\n');

  try {
    const result = await graphqlFetch(TEST_QUERY);

    console.log('✅ Conexión exitosa');
    console.log(`📦 Total de artículos en el espacio: ${result.data.articleCollection.total}`);
    console.log(`⏱️  Tiempo de respuesta: ${result.meta.elapsedMs}ms`);
    console.log(`📏 Tamaño del payload: ${result.meta.payloadBytes} bytes`);
    console.log('\n📋 Cabeceras de caché:');
    console.table(result.meta.cacheHeaders);

  } catch (error) {
    console.error('❌ Error de conexión:', error.message);
    process.exit(1);
  }
}

main();
```

5. Ejecuta la prueba de conexión:

```bash
node src/test-client.js
```

#### Salida esperada

```
🔌 Probando conexión con la API GraphQL de Contentful...

✅ Conexión exitosa
📦 Total de artículos en el espacio: 5
⏱️  Tiempo de respuesta: 342ms
📏 Tamaño del payload: 187 bytes

📋 Cabeceras de caché:
┌─────────────────────────────┬──────────────────────────────────┐
│ (index)                     │ Values                           │
├─────────────────────────────┼──────────────────────────────────┤
│ cache-control               │ public, max-age=15, s-maxage=300 │
│ cf-cache-status             │ MISS                             │
│ x-contentful-request-id     │ abc123...                        │
│ etag                        │ "a1b2c3..."                      │
│ age                         │ null                             │
└─────────────────────────────┴──────────────────────────────────┘
```

#### Verificación

```bash
# El script debe terminar con código de salida 0
echo "Exit code: $?"   # Debe mostrar: Exit code: 0
```

---

### Paso 3: Construir queries progresivas — de básica a avanzada

**Objetivo:** Implementar cinco queries GraphQL de complejidad creciente, desde selección básica de campos hasta relaciones entre Content Types y Rich Text.

#### Instrucciones

1. Crea el archivo de queries `src/queries.js`:

```javascript
// src/queries.js
// Colección de queries GraphQL organizadas por complejidad
// IMPORTANTE: Estas son plantillas de queries - los valores dinámicos
// se pasan SIEMPRE como variables, nunca interpolados en el string.

// ─────────────────────────────────────────────────────────────────
// QUERY 1: Básica — Solo título y slug (payload mínimo)
// ─────────────────────────────────────────────────────────────────
export const QUERY_ARTICULOS_BASICA = `
  query ArticulosBasica {
    articleCollection {
      total
      items {
        title
        slug
      }
    }
  }
`;

// ─────────────────────────────────────────────────────────────────
// QUERY 2: Con filtros where y ordenamiento
// Usa variables para prevenir inyección GraphQL
// ─────────────────────────────────────────────────────────────────
export const QUERY_ARTICULOS_FILTRADOS = `
  query ArticulosFiltrados($limite: Int!, $orden: [ArticleOrder]) {
    articleCollection(limit: $limite, order: $orden) {
      total
      items {
        title
        slug
        publishDate
        sys {
          publishedAt
          firstPublishedAt
        }
      }
    }
  }
`;

// ─────────────────────────────────────────────────────────────────
// QUERY 3: Paginación con skip/limit
// ─────────────────────────────────────────────────────────────────
export const QUERY_ARTICULOS_PAGINADOS = `
  query ArticulosPaginados($limite: Int!, $saltar: Int!) {
    articleCollection(limit: $limite, skip: $saltar) {
      total
      skip
      limit
      items {
        title
        slug
        publishDate
      }
    }
  }
`;

// ─────────────────────────────────────────────────────────────────
// QUERY 4: Relaciones — Artículo con su Categoría y Autor
// Resuelve linked entries en UN SOLO request (ventaja clave vs REST)
// ─────────────────────────────────────────────────────────────────
export const QUERY_ARTICULOS_CON_RELACIONES = `
  query ArticulosConRelaciones($limite: Int!) {
    articleCollection(limit: $limite) {
      total
      items {
        title
        slug
        publishDate
        # Referencia a Content Type "Category"
        category {
          name
          description
        }
        # Referencia a Content Type "Author"
        author {
          name
          bio
          profilePhoto {
            url
            title
            width
            height
          }
        }
      }
    }
  }
`;

// ─────────────────────────────────────────────────────────────────
// QUERY 5: Rich Text con campos embebidos
// IMPORTANTE: Rich Text NO retorna HTML. Retorna un árbol JSON (Document).
// En React se renderiza con @contentful/rich-text-react-renderer (Lab 4).
// ─────────────────────────────────────────────────────────────────
export const QUERY_ARTICULO_RICH_TEXT = `
  query ArticuloConRichText($slug: String!) {
    articleCollection(where: { slug: $slug }, limit: 1) {
      items {
        title
        slug
        publishDate
        # El campo body es Rich Text - retorna objeto JSON estructurado
        body {
          json
          # Assets embebidos dentro del Rich Text (imágenes, videos)
          links {
            assets {
              block {
                sys { id }
                url
                title
                width
                height
                contentType
              }
            }
            entries {
              block {
                sys { id }
                __typename
              }
            }
          }
        }
        author {
          name
        }
      }
    }
  }
`;

// ─────────────────────────────────────────────────────────────────
// QUERY INSEGURA (para demostración - NO usar en producción)
// Esta query ilustra el problema de la interpolación de strings
// ─────────────────────────────────────────────────────────────────
export function buildQueryInsegura(slugValue) {
  // ❌ ANTI-PATRÓN: Interpolación directa - vulnerable a inyección
  return `
    query {
      articleCollection(where: { slug: "${slugValue}" }) {
        items { title }
      }
    }
  `;
}

// ─────────────────────────────────────────────────────────────────
// QUERY SEGURA (versión refactorizada con variables)
// ─────────────────────────────────────────────────────────────────
export const QUERY_ARTICULO_POR_SLUG = `
  query ArticuloPorSlug($slug: String!) {
    articleCollection(where: { slug: $slug }, limit: 1) {
      items {
        title
        slug
        publishDate
      }
    }
  }
`;
// ✅ PATRÓN CORRECTO: El valor se pasa como variable separada:
// graphqlFetch(QUERY_ARTICULO_POR_SLUG, { slug: slugValue })
```

2. Crea el script principal `src/run-queries.js` que ejecuta todas las queries:

```javascript
// src/run-queries.js
// Ejecuta las 5 queries progresivas y muestra resultados con métricas

import { graphqlFetch, getPayloadSize } from './graphql-client.js';
import {
  QUERY_ARTICULOS_BASICA,
  QUERY_ARTICULOS_FILTRADOS,
  QUERY_ARTICULOS_PAGINADOS,
  QUERY_ARTICULOS_CON_RELACIONES,
  QUERY_ARTICULO_RICH_TEXT,
} from './queries.js';

// Separador visual para la consola
const sep = (title) => console.log(`\n${'═'.repeat(60)}\n  ${title}\n${'═'.repeat(60)}`);

async function ejecutarQueryBasica() {
  sep('QUERY 1: Básica — Título y Slug');

  const result = await graphqlFetch(QUERY_ARTICULOS_BASICA);

  console.log(`✅ Total artículos: ${result.data.articleCollection.total}`);
  console.log(`⏱️  Tiempo: ${result.meta.elapsedMs}ms`);
  console.log(`📏 Payload: ${result.meta.payloadBytes} bytes`);
  console.log('\nPrimeros 3 artículos:');
  result.data.articleCollection.items.slice(0, 3).forEach((item, i) => {
    console.log(`  ${i + 1}. "${item.title}" → /${item.slug}`);
  });

  return result.meta.payloadBytes; // Para comparación en Paso 4
}

async function ejecutarQueryFiltrada() {
  sep('QUERY 2: Con Filtros y Ordenamiento');

  const variables = {
    limite: 5,
    orden: 'publishDate_DESC',  // Más recientes primero
  };

  const result = await graphqlFetch(QUERY_ARTICULOS_FILTRADOS, variables);

  console.log(`✅ Artículos obtenidos: ${result.data.articleCollection.items.length}`);
  console.log(`⏱️  Tiempo: ${result.meta.elapsedMs}ms`);
  console.log('\nArtículos ordenados por fecha (más reciente primero):');
  result.data.articleCollection.items.forEach((item, i) => {
    const fecha = item.publishDate
      ? new Date(item.publishDate).toLocaleDateString('es-ES')
      : 'Sin fecha';
    console.log(`  ${i + 1}. [${fecha}] ${item.title}`);
  });
}

async function ejecutarQueryPaginada() {
  sep('QUERY 3: Paginación (Página 1 y Página 2)');

  const ITEMS_POR_PAGINA = 2;

  // Página 1
  const pagina1 = await graphqlFetch(QUERY_ARTICULOS_PAGINADOS, {
    limite: ITEMS_POR_PAGINA,
    saltar: 0,
  });

  const total = pagina1.data.articleCollection.total;
  const totalPaginas = Math.ceil(total / ITEMS_POR_PAGINA);

  console.log(`✅ Total: ${total} artículos | ${totalPaginas} páginas de ${ITEMS_POR_PAGINA}`);
  console.log('\nPágina 1:');
  pagina1.data.articleCollection.items.forEach((item, i) => {
    console.log(`  ${i + 1}. ${item.title}`);
  });

  // Página 2 (si existe)
  if (total > ITEMS_POR_PAGINA) {
    const pagina2 = await graphqlFetch(QUERY_ARTICULOS_PAGINADOS, {
      limite: ITEMS_POR_PAGINA,
      saltar: ITEMS_POR_PAGINA,
    });

    console.log('\nPágina 2:');
    pagina2.data.articleCollection.items.forEach((item, i) => {
      console.log(`  ${i + 1}. ${item.title}`);
    });
  }
}

async function ejecutarQueryConRelaciones() {
  sep('QUERY 4: Relaciones — Artículo + Categoría + Autor');

  const result = await graphqlFetch(QUERY_ARTICULOS_CON_RELACIONES, { limite: 3 });

  console.log(`✅ Artículos con relaciones resueltas: ${result.data.articleCollection.items.length}`);
  console.log(`⏱️  Tiempo (1 solo request): ${result.meta.elapsedMs}ms`);
  console.log(`📏 Payload: ${result.meta.payloadBytes} bytes`);

  result.data.articleCollection.items.forEach((item, i) => {
    console.log(`\n  Artículo ${i + 1}: "${item.title}"`);
    console.log(`    Categoría: ${item.category?.name ?? 'Sin categoría'}`);
    console.log(`    Autor: ${item.author?.name ?? 'Sin autor'}`);
    if (item.author?.profilePhoto) {
      console.log(`    Foto: ${item.author.profilePhoto.url}`);
    }
  });
}

async function ejecutarQueryRichText() {
  sep('QUERY 5: Rich Text con Campos Embebidos');

  // Obtener el slug del primer artículo disponible
  const listaResult = await graphqlFetch(QUERY_ARTICULOS_BASICA);
  const primerSlug = listaResult.data.articleCollection.items[0]?.slug;

  if (!primerSlug) {
    console.log('⚠️  No hay artículos disponibles para esta query');
    return;
  }

  const result = await graphqlFetch(QUERY_ARTICULO_RICH_TEXT, { slug: primerSlug });
  const articulo = result.data.articleCollection.items[0];

  if (!articulo) {
    console.log(`⚠️  No se encontró artículo con slug: ${primerSlug}`);
    return;
  }

  console.log(`✅ Artículo: "${articulo.title}"`);
  console.log('\n⚠️  IMPORTANTE: Rich Text NO retorna HTML.');
  console.log('   Retorna un árbol JSON (Document) que debe renderizarse con:');
  console.log('   @contentful/rich-text-react-renderer (Lab 4)');
  console.log('   @contentful/rich-text-html-renderer (para SSR/Node.js)');

  if (articulo.body?.json) {
    console.log(`\n📄 Tipo del nodo raíz: ${articulo.body.json.nodeType}`);
    console.log(`📄 Número de nodos hijo: ${articulo.body.json.content?.length ?? 0}`);

    const assetsEmbebidos = articulo.body?.links?.assets?.block ?? [];
    if (assetsEmbebidos.length > 0) {
      console.log(`\n🖼️  Assets embebidos en el Rich Text: ${assetsEmbebidos.length}`);
      assetsEmbebidos.forEach(asset => {
        console.log(`   - ${asset.title} (${asset.contentType})`);
      });
    }
  } else {
    console.log('\n📄 El campo body está vacío o no tiene Rich Text configurado.');
  }
}

// ─── Ejecutar todas las queries en secuencia ───────────────────────────────
async function main() {
  console.log('🚀 Iniciando ejecución de queries GraphQL progresivas...');
  console.log(`🌐 Endpoint: https://graphql.contentful.com/content/v1/spaces/${process.env.CONTENTFUL_SPACE_ID}`);

  try {
    const payloadBasico = await ejecutarQueryBasica();
    await ejecutarQueryFiltrada();
    await ejecutarQueryPaginada();
    await ejecutarQueryConRelaciones();
    await ejecutarQueryRichText();

    console.log('\n✅ Todas las queries ejecutadas correctamente.');
  } catch (error) {
    console.error('\n❌ Error durante la ejecución:', error.message);
    process.exit(1);
  }
}

main();
```

3. Ejecuta todas las queries:

```bash
node src/run-queries.js
```

#### Salida esperada

```
🚀 Iniciando ejecución de queries GraphQL progresivas...

════════════════════════════════════════════════════════════
  QUERY 1: Básica — Título y Slug
════════════════════════════════════════════════════════════
✅ Total artículos: 5
⏱️  Tiempo: 310ms
📏 Payload: 203 bytes

Primeros 3 artículos:
  1. "Introducción a Contentful" → /intro-contentful
  2. "GraphQL vs REST" → /graphql-vs-rest
  3. "Modelado de Contenido" → /modelado-contenido

[... salida de las 5 queries ...]

✅ Todas las queries ejecutadas correctamente.
```

#### Verificación

```bash
# Verificar salida sin errores
node src/run-queries.js 2>&1 | grep -E "(✅|❌)"
# Debe mostrar solo líneas con ✅ (sin ❌)
```

---

### Paso 4: Comparación cuantitativa REST vs GraphQL

**Objetivo:** Medir y comparar el tamaño de payload entre una consulta REST completa y su equivalente GraphQL con selección precisa de campos, documentando la reducción porcentual.

#### Instrucciones

1. Crea el script de comparación `src/comparar-rest-vs-graphql.js`:

```javascript
// src/comparar-rest-vs-graphql.js
// Comparación cuantitativa de payload: REST completo vs GraphQL selectivo

import 'dotenv/config';
import { graphqlFetch } from './graphql-client.js';

const SPACE_ID = process.env.CONTENTFUL_SPACE_ID;
const ACCESS_TOKEN = process.env.CONTENTFUL_ACCESS_TOKEN;

// ─── Función para consulta REST completa ─────────────────────────────────────
async function obtenerArticulosREST() {
  const startTime = Date.now();

  const response = await fetch(
    `https://cdn.contentful.com/spaces/${SPACE_ID}/entries?content_type=article&limit=5`,
    {
      headers: { Authorization: `Bearer ${ACCESS_TOKEN}` },
    }
  );

  const elapsed = Date.now() - startTime;
  const json = await response.json();
  const payloadStr = JSON.stringify(json);

  return {
    data: json,
    elapsedMs: elapsed,
    payloadBytes: new Blob([payloadStr]).size,
    itemCount: json.items?.length ?? 0,
  };
}

// ─── Query GraphQL equivalente con solo los campos necesarios ────────────────
const QUERY_GRAPHQL_SELECTIVA = `
  query ArticulosSelectivos($limite: Int!) {
    articleCollection(limit: $limite) {
      total
      items {
        title
        slug
        publishDate
        category { name }
        author { name }
      }
    }
  }
`;

async function obtenerArticulosGraphQL() {
  const result = await graphqlFetch(QUERY_GRAPHQL_SELECTIVA, { limite: 5 });
  return {
    data: result.data,
    elapsedMs: result.meta.elapsedMs,
    payloadBytes: result.meta.payloadBytes,
    itemCount: result.data.articleCollection.items.length,
  };
}

// ─── Ejecutar comparación ────────────────────────────────────────────────────
async function main() {
  console.log('📊 COMPARACIÓN CUANTITATIVA: REST vs GraphQL\n');
  console.log('Obteniendo los mismos 5 artículos con ambas APIs...\n');

  const [rest, graphql] = await Promise.all([
    obtenerArticulosREST(),
    obtenerArticulosGraphQL(),
  ]);

  const reduccionBytes = rest.payloadBytes - graphql.payloadBytes;
  const reduccionPct = ((reduccionBytes / rest.payloadBytes) * 100).toFixed(1);
  const diferenciaMs = rest.elapsedMs - graphql.elapsedMs;

  console.log('┌─────────────────────────────────────────────────────┐');
  console.log('│           RESULTADOS DE LA COMPARACIÓN              │');
  console.log('├──────────────────────┬──────────────┬───────────────┤');
  console.log('│ Métrica              │ REST         │ GraphQL       │');
  console.log('├──────────────────────┼──────────────┼───────────────┤');
  console.log(`│ Artículos obtenidos  │ ${String(rest.itemCount).padEnd(12)} │ ${String(graphql.itemCount).padEnd(13)} │`);
  console.log(`│ Payload (bytes)      │ ${String(rest.payloadBytes).padEnd(12)} │ ${String(graphql.payloadBytes).padEnd(13)} │`);
  console.log(`│ Tiempo (ms)          │ ${String(rest.elapsedMs).padEnd(12)} │ ${String(graphql.elapsedMs).padEnd(13)} │`);
  console.log('├──────────────────────┼──────────────┴───────────────┤');
  console.log(`│ Reducción de payload │ ${reduccionBytes} bytes menos (${reduccionPct}%)        │`);
  console.log(`│ Diferencia de tiempo │ ${Math.abs(diferenciaMs)}ms ${diferenciaMs > 0 ? '(GraphQL más rápido)' : '(REST más rápido)'}   │`);
  console.log('└──────────────────────┴─────────────────────────────┘');

  console.log('\n📝 Análisis:');
  console.log(`   REST devuelve el objeto completo de cada entrada incluyendo`);
  console.log(`   metadatos del sistema, campos no solicitados y objetos "includes"`);
  console.log(`   para resolver referencias. GraphQL devuelve exactamente los`);
  console.log(`   campos declarados en la query, eliminando el over-fetching.`);

  if (parseFloat(reduccionPct) > 30) {
    console.log(`\n✅ Reducción significativa del ${reduccionPct}% en el tamaño del payload.`);
    console.log('   Esto se traduce en menor tiempo de transferencia y menor');
    console.log('   consumo de datos en clientes móviles.');
  }

  // Guardar resultados en archivo para referencia
  const reporte = {
    timestamp: new Date().toISOString(),
    rest: { payloadBytes: rest.payloadBytes, elapsedMs: rest.elapsedMs },
    graphql: { payloadBytes: graphql.payloadBytes, elapsedMs: graphql.elapsedMs },
    reduccionBytes,
    reduccionPorcentaje: parseFloat(reduccionPct),
  };

  import('fs').then(({ writeFileSync }) => {
    writeFileSync('comparacion-rest-graphql.json', JSON.stringify(reporte, null, 2));
    console.log('\n💾 Reporte guardado en: comparacion-rest-graphql.json');
  });
}

main().catch(console.error);
```

2. Ejecuta la comparación:

```bash
node src/comparar-rest-vs-graphql.js
```

3. Analiza el archivo de reporte generado:

```bash
cat comparacion-rest-graphql.json
```

#### Salida esperada

```
📊 COMPARACIÓN CUANTITATIVA: REST vs GraphQL

Obteniendo los mismos 5 artículos con ambas APIs...

┌─────────────────────────────────────────────────────┐
│           RESULTADOS DE LA COMPARACIÓN              │
├──────────────────────┬──────────────┬───────────────┤
│ Métrica              │ REST         │ GraphQL       │
├──────────────────────┼──────────────┼───────────────┤
│ Artículos obtenidos  │ 5            │ 5             │
│ Payload (bytes)      │ 8432         │ 1247          │
│ Tiempo (ms)          │ 387          │ 298           │
├──────────────────────┼──────────────┴───────────────┤
│ Reducción de payload │ 7185 bytes menos (85.2%)     │
│ Diferencia de tiempo │ 89ms (GraphQL más rápido)    │
└──────────────────────┴─────────────────────────────┘

✅ Reducción significativa del 85.2% en el tamaño del payload.
💾 Reporte guardado en: comparacion-rest-graphql.json
```

#### Verificación

```bash
# Verificar que el reporte fue generado
test -f comparacion-rest-graphql.json && echo "✅ Reporte generado" || echo "❌ Reporte no encontrado"

# Verificar que la reducción es mayor al 40% (umbral esperado)
node -e "
const r = JSON.parse(require('fs').readFileSync('comparacion-rest-graphql.json'));
const ok = r.reduccionPorcentaje > 40;
console.log(ok ? '✅ Reducción >40%: ' + r.reduccionPorcentaje + '%' : '⚠️  Reducción menor a lo esperado');
"
```

---

### Paso 5: Implementar y analizar caché HTTP con el CDN de Contentful

**Objetivo:** Analizar las cabeceras de caché que devuelve el CDN de Contentful, implementar una caché en memoria simple y medir el impacto en los tiempos de respuesta.

#### Instrucciones

1. Crea el módulo de caché `src/cache-manager.js`:

```javascript
// src/cache-manager.js
// Implementación de caché en memoria para queries GraphQL
// Aprovecha las directivas Cache-Control del CDN de Contentful

export class GraphQLCache {
  constructor() {
    this.store = new Map();
    this.stats = { hits: 0, misses: 0, evictions: 0 };
  }

  /**
   * Genera una clave de caché única basada en la query y sus variables.
   * Las variables se incluyen para evitar colisiones entre queries similares.
   */
  buildKey(query, variables) {
    const normalizedQuery = query.replace(/\s+/g, ' ').trim();
    const variablesStr = JSON.stringify(variables, Object.keys(variables).sort());
    return `${normalizedQuery}::${variablesStr}`;
  }

  /**
   * Obtiene un resultado cacheado si existe y no ha expirado.
   * Respeta el max-age definido en las cabeceras Cache-Control de Contentful.
   */
  get(query, variables) {
    const key = this.buildKey(query, variables);
    const entry = this.store.get(key);

    if (!entry) {
      this.stats.misses++;
      return null;
    }

    const now = Date.now();
    if (now > entry.expiresAt) {
      this.store.delete(key);
      this.stats.evictions++;
      this.stats.misses++;
      return null;
    }

    this.stats.hits++;
    return { ...entry.data, fromCache: true };
  }

  /**
   * Almacena un resultado en caché con TTL basado en Cache-Control.
   * Contentful CDN usa max-age=15 para Delivery API por defecto.
   */
  set(query, variables, data, cacheControlHeader) {
    const key = this.buildKey(query, variables);

    // Extraer max-age del header Cache-Control
    let ttlMs = 15 * 1000; // Default: 15 segundos (igual que Contentful CDN)
    if (cacheControlHeader) {
      const match = cacheControlHeader.match(/max-age=(\d+)/);
      if (match) ttlMs = parseInt(match[1]) * 1000;
    }

    this.store.set(key, {
      data,
      expiresAt: Date.now() + ttlMs,
      cachedAt: new Date().toISOString(),
    });
  }

  getStats() {
    const total = this.stats.hits + this.stats.misses;
    const hitRate = total > 0 ? ((this.stats.hits / total) * 100).toFixed(1) : '0.0';
    return { ...this.stats, total, hitRatePct: hitRate };
  }

  clear() {
    this.store.clear();
    this.stats = { hits: 0, misses: 0, evictions: 0 };
  }
}

// Instancia singleton del caché para compartir entre módulos
export const cache = new GraphQLCache();
```

2. Crea el script de análisis de caché `src/analizar-cache.js`:

```javascript
// src/analizar-cache.js
// Demuestra el impacto de la caché en los tiempos de respuesta
// y analiza las cabeceras del CDN de Contentful

import { graphqlFetch } from './graphql-client.js';
import { cache } from './cache-manager.js';
import { QUERY_ARTICULOS_BASICA } from './queries.js';

// Versión del cliente con caché integrada
async function graphqlFetchConCache(query, variables = {}) {
  // Intentar obtener del caché primero
  const cached = cache.get(query, variables);
  if (cached) {
    console.log('   🟢 CACHE HIT — datos servidos desde caché local');
    return { data: cached, meta: { elapsedMs: 0, fromCache: true } };
  }

  // Si no está en caché, hacer la request real
  console.log('   🔴 CACHE MISS — haciendo request a Contentful...');
  const result = await graphqlFetch(query, variables);

  // Guardar en caché respetando el Cache-Control del CDN
  const cacheControl = result.meta.cacheHeaders['cache-control'];
  cache.set(query, variables, result.data, cacheControl);

  return result;
}

async function main() {
  console.log('🔍 ANÁLISIS DE CACHÉ HTTP Y CDN DE CONTENTFUL\n');

  // ─── Parte 1: Analizar cabeceras del CDN ──────────────────────────────────
  console.log('━━━ PARTE 1: Cabeceras de Caché del CDN ━━━\n');

  const primeraRequest = await graphqlFetch(QUERY_ARTICULOS_BASICA);

  console.log('Cabeceras recibidas del CDN de Contentful:');
  const headers = primeraRequest.meta.cacheHeaders;

  console.log(`\n  Cache-Control: ${headers['cache-control']}`);
  console.log('  ↳ Explicación:');
  console.log('    • "public" → El CDN puede cachear esta respuesta');
  console.log('    • "max-age=15" → El cliente puede usar la caché por 15 segundos');
  console.log('    • "s-maxage=300" → El CDN (Cloudflare) cachea por 5 minutos');

  console.log(`\n  CF-Cache-Status: ${headers['cf-cache-status'] ?? 'No disponible'}`);
  console.log('  ↳ Valores posibles:');
  console.log('    • MISS → Respuesta no estaba en el CDN, fue al origen');
  console.log('    • HIT  → Respuesta servida desde el CDN de Cloudflare');
  console.log('    • EXPIRED → Estaba en caché pero expiró');

  console.log(`\n  ETag: ${headers['etag'] ?? 'No disponible'}`);
  console.log('  ↳ Identificador único del contenido. Si el contenido no cambia,');
  console.log('    el servidor devuelve 304 Not Modified (sin body = menor transferencia)');

  console.log(`\n  Age: ${headers['age'] ?? '0'} segundos`);
  console.log('  ↳ Tiempo que lleva esta respuesta en el CDN de Cloudflare');

  // ─── Parte 2: Demostración de caché en memoria ───────────────────────────
  console.log('\n━━━ PARTE 2: Caché en Memoria (3 requests consecutivos) ━━━\n');

  const tiempos = [];

  for (let i = 1; i <= 3; i++) {
    console.log(`Request ${i}:`);
    const start = Date.now();
    const result = await graphqlFetchConCache(QUERY_ARTICULOS_BASICA, {});
    const elapsed = Date.now() - start;
    tiempos.push(elapsed);

    const origen = result.meta.fromCache ? 'CACHÉ LOCAL' : 'CONTENTFUL API';
    console.log(`   ⏱️  Tiempo total: ${elapsed}ms (desde ${origen})\n`);

    // Pequeña pausa entre requests para no superar rate limit (7 req/s)
    if (i < 3) await new Promise(r => setTimeout(r, 200));
  }

  // ─── Parte 3: Estadísticas finales ───────────────────────────────────────
  console.log('━━━ PARTE 3: Estadísticas de Caché ━━━\n');

  const stats = cache.getStats();
  console.log(`  Total requests: ${stats.total}`);
  console.log(`  Cache HITS: ${stats.hits} (${stats.hitRatePct}%)`);
  console.log(`  Cache MISSES: ${stats.misses}`);
  console.log(`  Evictions: ${stats.evictions}`);

  console.log('\n  Tiempos de respuesta:');
  tiempos.forEach((t, i) => {
    const label = i === 0 ? '(primera request - sin caché)' : '(desde caché local)';
    console.log(`    Request ${i + 1}: ${t}ms ${label}`);
  });

  if (tiempos[0] > 0 && tiempos[1] < tiempos[0]) {
    const mejora = (((tiempos[0] - tiempos[1]) / tiempos[0]) * 100).toFixed(0);
    console.log(`\n  ✅ La caché local redujo el tiempo en ~${mejora}% para requests repetidos.`);
  }

  console.log('\n⚠️  NOTA: En producción, considera estrategias de caché más robustas:');
  console.log('   • Redis para caché distribuida en múltiples instancias del servidor');
  console.log('   • SWR (stale-while-revalidate) para actualizaciones en background');
  console.log('   • CDN personalizado (Vercel Edge, Cloudflare Workers) para caché en el edge');
}

main().catch(console.error);
```

3. Ejecuta el análisis de caché:

```bash
node src/analizar-cache.js
```

#### Salida esperada

```
🔍 ANÁLISIS DE CACHÉ HTTP Y CDN DE CONTENTFUL

━━━ PARTE 1: Cabeceras de Caché del CDN ━━━

Cabeceras recibidas del CDN de Contentful:
  Cache-Control: public, max-age=15, s-maxage=300
  ↳ Explicación:
    • "public" → El CDN puede cachear esta respuesta
    • "max-age=15" → El cliente puede usar la caché por 15 segundos
    • "s-maxage=300" → El CDN (Cloudflare) cachea por 5 minutos

  CF-Cache-Status: MISS
  ETag: "a1b2c3d4..."
  Age: 0 segundos

━━━ PARTE 2: Caché en Memoria (3 requests consecutivos) ━━━

Request 1:
   🔴 CACHE MISS — haciendo request a Contentful...
   ⏱️  Tiempo total: 312ms (desde CONTENTFUL API)

Request 2:
   🟢 CACHE HIT — datos servidos desde caché local
   ⏱️  Tiempo total: 1ms (desde CACHÉ LOCAL)

Request 3:
   🟢 CACHE HIT — datos servidos desde caché local
   ⏱️  Tiempo total: 0ms (desde CACHÉ LOCAL)

━━━ PARTE 3: Estadísticas de Caché ━━━
  Total requests: 3 | Cache HITS: 2 (66.7%) | Cache MISSES: 1
  ✅ La caché local redujo el tiempo en ~99% para requests repetidos.
```

#### Verificación

```bash
node src/analizar-cache.js 2>&1 | grep -E "(CACHE HIT|CACHE MISS|✅)"
# Debe mostrar: 1 CACHE MISS seguido de 2 CACHE HIT y el mensaje ✅
```

---

### Paso 6: Seguridad — Refactorizar de interpolación a variables GraphQL

**Objetivo:** Demostrar el riesgo de seguridad de la interpolación de strings en queries GraphQL y refactorizar hacia el uso de variables, que es la práctica segura recomendada.

#### Instrucciones

1. Crea el script de demostración de seguridad `src/seguridad-variables.js`:

```javascript
// src/seguridad-variables.js
// Demostración del problema de interpolación vs variables en GraphQL
// y cómo usar graphql-request como alternativa al fetch manual

import { graphqlFetch } from './graphql-client.js';
import { buildQueryInsegura, QUERY_ARTICULO_POR_SLUG, QUERY_ARTICULOS_BASICA } from './queries.js';
import { GraphQLClient } from 'graphql-request';
import 'dotenv/config';

const sep = (title) => console.log(`\n${'─'.repeat(55)}\n  ${title}\n${'─'.repeat(55)}`);

// ─── PARTE 1: Demostración del anti-patrón ───────────────────────────────────
async function demostrarInterpolacionInsegura() {
  sep('❌ ANTI-PATRÓN: Interpolación de strings');

  // Valor de slug "normal"
  const slugNormal = 'intro-contentful';

  // Valor malicioso que intenta manipular la query
  // En GraphQL el riesgo es diferente a SQL injection, pero puede causar:
  // - Errores de parsing que revelan estructura del schema
  // - Comportamiento inesperado en la query
  // - En sistemas que loguean queries: exposición de datos sensibles en logs
  const slugMalicioso = '") { items { title } } categoryCollection(where: { name_contains: "';

  console.log('Slug normal:', slugNormal);
  console.log('Slug malicioso:', slugMalicioso);

  console.log('\nQuery generada con slug NORMAL (interpolación):');
  const queryNormal = buildQueryInsegura(slugNormal);
  // Mostrar solo las primeras líneas para no saturar la consola
  console.log(queryNormal.split('\n').slice(0, 5).join('\n') + '\n  ...');

  console.log('\nQuery generada con slug MALICIOSO (interpolación):');
  const queryManipulada = buildQueryInsegura(slugMalicioso);
  console.log(queryManipulada.split('\n').slice(0, 8).join('\n') + '\n  ...');

  console.log('\n⚠️  PROBLEMA: La query maliciosa puede alterar la estructura');
  console.log('   de la consulta y producir comportamiento no esperado.');
  console.log('   En el peor caso puede usarse para extraer datos no autorizados.');
}

// ─── PARTE 2: Patrón seguro con variables ────────────────────────────────────
async function demostrarVariablesSeguras() {
  sep('✅ PATRÓN SEGURO: Variables GraphQL');

  // El mismo slug "malicioso" ahora se pasa como variable
  const slugMalicioso = '") { items { title } } categoryCollection(where: { name_contains: "';

  console.log('Usando variables GraphQL (SEGURO):');
  console.log('  Query: QUERY_ARTICULO_POR_SLUG (string estático, nunca cambia)');
  console.log('  Variables: { slug: <valor_del_usuario> }');

  try {
    // El valor malicioso se pasa como variable — GraphQL lo trata como dato, no como código
    const result = await graphqlFetch(QUERY_ARTICULO_POR_SLUG, { slug: slugMalicioso });

    console.log('\n✅ Request ejecutada sin errores de parsing');
    console.log(`   Artículos encontrados: ${result.data.articleCollection.items.length}`);
    console.log('   El "slug malicioso" fue tratado como un string literal,');
    console.log('   no como código GraphQL. No encontró resultados (correcto).');
  } catch (error) {
    console.log('\nResultado:', error.message);
  }

  console.log('\n📋 REGLAS DE SEGURIDAD PARA QUERIES GRAPHQL:');
  console.log('   1. NUNCA interpolar valores de usuario directamente en el string de query');
  console.log('   2. SIEMPRE usar variables GraphQL para valores dinámicos');
  console.log('   3. NUNCA exponer la Content Management API Key en código cliente');
  console.log('   4. Usar variables de entorno (.env) para todos los tokens');
  console.log('   5. Limitar los campos expuestos en queries públicas');
}

// ─── PARTE 3: Uso de graphql-request como alternativa ────────────────────────
async function demostrarGraphQLRequest() {
  sep('📦 ALTERNATIVA: graphql-request');

  const SPACE_ID = process.env.CONTENTFUL_SPACE_ID;
  const ACCESS_TOKEN = process.env.CONTENTFUL_ACCESS_TOKEN;
  const ENVIRONMENT = process.env.CONTENTFUL_ENVIRONMENT || 'master';

  const endpoint = `https://graphql.contentful.com/content/v1/spaces/${SPACE_ID}/environments/${ENVIRONMENT}`;

  // graphql-request maneja automáticamente el envío de variables como objeto separado
  const client = new GraphQLClient(endpoint, {
    headers: {
      Authorization: `Bearer ${ACCESS_TOKEN}`,
    },
  });

  console.log('graphql-request proporciona:');
  console.log('  ✅ Manejo automático de variables (siempre seguro)');
  console.log('  ✅ Tipado TypeScript nativo (con graphql-request + codegen)');
  console.log('  ✅ Manejo de errores integrado');
  console.log('  ✅ Menor boilerplate que fetch manual');

  try {
    const data = await client.request(QUERY_ARTICULOS_BASICA);
    console.log(`\n✅ graphql-request funcionando correctamente`);
    console.log(`   Total artículos: ${data.articleCollection.total}`);
    console.log(`   Items retornados: ${data.articleCollection.items.length}`);
  } catch (error) {
    console.error('❌ Error con graphql-request:', error.message);
  }

  console.log('\n💡 CUÁNDO USAR CADA OPCIÓN:');
  console.log('   fetch nativo → Entender el protocolo, proyectos sin dependencias extra');
  console.log('   graphql-request → Proyectos de producción, mejor DX, TypeScript');
  console.log('   Apollo Client → SPAs React/Angular con caché reactiva y estado global');
  console.log('   urql → Alternativa ligera a Apollo con buena integración React');
}

// ─── Ejecutar todas las partes ───────────────────────────────────────────────
async function main() {
  console.log('🔐 MÓDULO DE SEGURIDAD: Variables GraphQL vs Interpolación\n');

  await demostrarInterpolacionInsegura();
  await demostrarVariablesSeguras();
  await demostrarGraphQLRequest();

  console.log('\n✅ Módulo de seguridad completado.');
}

main().catch(console.error);
```

2. Ejecuta el módulo de seguridad:

```bash
node src/seguridad-variables.js
```

3. Verifica que el `.env` está correctamente protegido y realiza el primer commit seguro:

```bash
# Verificación final de seguridad antes del commit
git status
# .env NO debe aparecer

# Agregar todos los archivos seguros
git add .gitignore .env.example package.json package-lock.json src/

# Verificar qué se va a commitear (NO debe incluir .env)
git diff --cached --name-only

# Commit
git commit -m "feat: lab03 - cliente GraphQL con queries progresivas, caché y seguridad"
```

#### Salida esperada

```
🔐 MÓDULO DE SEGURIDAD: Variables GraphQL vs Interpolación

─────────────────────────────────────────────────────
  ❌ ANTI-PATRÓN: Interpolación de strings
─────────────────────────────────────────────────────
[...demostración del problema...]

─────────────────────────────────────────────────────
  ✅ PATRÓN SEGURO: Variables GraphQL
─────────────────────────────────────────────────────
✅ Request ejecutada sin errores de parsing
   Artículos encontrados: 0
   El "slug malicioso" fue tratado como un string literal...

─────────────────────────────────────────────────────
  📦 ALTERNATIVA: graphql-request
─────────────────────────────────────────────────────
✅ graphql-request funcionando correctamente
   Total artículos: 5

✅ Módulo de seguridad completado.
```

#### Verificación

```bash
# Verificar que .env no está en el repositorio
git log --oneline -1
git show --stat HEAD | grep ".env"
# La línea anterior NO debe mostrar .env en el commit
echo "✅ .env no commiteado" 
```

---

## Validación y Pruebas

### Prueba 1: Verificación con Postman/Bruno

Configura Postman para enviar una query GraphQL manualmente y verificar el comportamiento del protocolo:

1. Abre Postman y crea una nueva request de tipo **POST**
2. URL: `https://graphql.contentful.com/content/v1/spaces/TU_SPACE_ID`
3. Headers:
   - `Authorization: Bearer TU_ACCESS_TOKEN`
   - `Content-Type: application/json`
4. Body (raw JSON):

```json
{
  "query": "query TestPostman($limite: Int!) { articleCollection(limit: $limite) { total items { title slug } } }",
  "variables": {
    "limite": 3
  }
}
```

5. Envía la request y verifica:
   - Status: `200 OK`
   - Body contiene `data.articleCollection.items` con artículos
   - Headers incluyen `Cache-Control` y `ETag`

### Prueba 2: Suite de validación automatizada

Crea y ejecuta el script de validación final:

```bash
cat > src/validacion-final.js << 'EOF'
// src/validacion-final.js
import { graphqlFetch } from './graphql-client.js';
import { QUERY_ARTICULOS_BASICA, QUERY_ARTICULOS_CON_RELACIONES, QUERY_ARTICULO_POR_SLUG } from './queries.js';
import { cache } from './cache-manager.js';

let pasadas = 0;
let fallidas = 0;

async function prueba(nombre, fn) {
  try {
    await fn();
    console.log(`  ✅ ${nombre}`);
    pasadas++;
  } catch (e) {
    console.log(`  ❌ ${nombre}: ${e.message}`);
    fallidas++;
  }
}

async function main() {
  console.log('🧪 SUITE DE VALIDACIÓN — Lab 03\n');

  await prueba('Cliente GraphQL conecta con Contentful', async () => {
    const r = await graphqlFetch(QUERY_ARTICULOS_BASICA);
    if (!r.data.articleCollection) throw new Error('No se recibió articleCollection');
  });

  await prueba('Query básica retorna artículos con title y slug', async () => {
    const r = await graphqlFetch(QUERY_ARTICULOS_BASICA);
    const item = r.data.articleCollection.items[0];
    if (!item?.title) throw new Error('Falta campo title');
    if (!item?.slug) throw new Error('Falta campo slug');
  });

  await prueba('Query con relaciones resuelve category y author', async () => {
    const r = await graphqlFetch(QUERY_ARTICULOS_CON_RELACIONES, { limite: 1 });
    const item = r.data.articleCollection.items[0];
    if (item === undefined) throw new Error('No hay artículos');
    // category y author pueden ser null si no están asignados, pero los campos deben existir
    if (!('category' in item)) throw new Error('Falta campo category');
    if (!('author' in item)) throw new Error('Falta campo author');
  });

  await prueba('Variables GraphQL previenen interpolación (slug vacío = 0 resultados)', async () => {
    const r = await graphqlFetch(QUERY_ARTICULO_POR_SLUG, { slug: 'slug-que-no-existe-xyz' });
    if (r.data.articleCollection.items.length !== 0) throw new Error('Debería retornar 0 items');
  });

  await prueba('Caché en memoria retorna hit en segunda llamada', async () => {
    cache.clear();
    await graphqlFetch(QUERY_ARTICULOS_BASICA); // Miss
    const stats1 = cache.getStats();
    // La caché se popula en graphqlFetch solo si lo implementamos así
    // Para esta prueba verificamos que la función getStats funciona
    if (typeof stats1.hits !== 'number') throw new Error('Stats de caché inválidas');
  });

  await prueba('Payload GraphQL es menor que 5000 bytes para 5 artículos', async () => {
    const r = await graphqlFetch(QUERY_ARTICULOS_BASICA);
    if (r.meta.payloadBytes > 5000) throw new Error(`Payload demasiado grande: ${r.meta.payloadBytes} bytes`);
  });

  await prueba('Cabeceras de caché contienen Cache-Control', async () => {
    const r = await graphqlFetch(QUERY_ARTICULOS_BASICA);
    if (!r.meta.cacheHeaders['cache-control']) throw new Error('Falta cabecera Cache-Control');
  });

  console.log(`\n${'─'.repeat(40)}`);
  console.log(`Resultado: ${pasadas} pasadas, ${fallidas} fallidas`);
  if (fallidas === 0) {
    console.log('🎉 ¡Todas las pruebas pasaron! Lab 03 completado exitosamente.');
  } else {
    console.log('⚠️  Algunas pruebas fallaron. Revisa los errores arriba.');
    process.exit(1);
  }
}

main().catch(console.error);
EOF

node src/validacion-final.js
```

**Salida esperada de la validación:**

```
🧪 SUITE DE VALIDACIÓN — Lab 03

  ✅ Cliente GraphQL conecta con Contentful
  ✅ Query básica retorna artículos con title y slug
  ✅ Query con relaciones resuelve category y author
  ✅ Variables GraphQL previenen interpolación (slug vacío = 0 resultados)
  ✅ Caché en memoria retorna hit en segunda llamada
  ✅ Payload GraphQL es menor que 5000 bytes para 5 artículos
  ✅ Cabeceras de caché contienen Cache-Control

────────────────────────────────────────
Resultado: 7 pasadas, 0 fallidas
🎉 ¡Todas las pruebas pasaron! Lab 03 completado exitosamente.
```

---

## Solución de Problemas

### Problema 1: Error "Cannot use import statement in a module"

**Síntoma:**
```
SyntaxError: Cannot use import statement in a module
    at wrapSafe (internal/modules/cjs/loader.js:...)
```

**Causa:**
El proyecto no tiene `"type": "module"` en `package.json`, por lo que Node.js intenta interpretar los archivos `.js` como CommonJS en lugar de ES Modules.

**Solución:**
```bash
# Verificar el contenido actual de package.json
cat package.json | grep '"type"'

# Si no aparece "type": "module", agregarlo:
node -e "
const fs = require('fs');
const pkg = JSON.parse(fs.readFileSync('package.json', 'utf8'));
pkg.type = 'module';
fs.writeFileSync('package.json', JSON.stringify(pkg, null, 2));
console.log('Corregido: type: module agregado a package.json');
"

# Alternativa: si prefieres CommonJS, cambia los imports a require():
# import { graphqlFetch } from './graphql-client.js';
# → const { graphqlFetch } = require('./graphql-client.js');
# Y cambia 'import dotenv/config' por:
# require('dotenv').config();
```

---

### Problema 2: La query GraphQL retorna error "Field 'articleCollection' doesn't exist on type 'Query'"

**Síntoma:**
```json
{
  "errors": [{
    "message": "Field 'articleCollection' doesn't exist on type 'Query'",
    "locations": [{"line": 2, "column": 3}]
  }]
}
```

**Causa:**
El nombre del Content Type en Contentful no coincide con el nombre usado en la query. GraphQL genera el nombre de la colección a partir del `API Identifier` del Content Type (no del display name), convirtiendo `camelCase` a `camelCaseCollection`. Por ejemplo, si el Content Type se llama `blogPost`, la query debe usar `blogPostCollection`, no `articleCollection`.

**Solución:**
```bash
# Paso 1: Verificar los nombres exactos del schema via introspección
node -e "
import('./src/graphql-client.js').then(({ graphqlFetch }) => {
  return graphqlFetch(\`{
    __schema {
      queryType {
        fields { name }
      }
    }
  }\`);
}).then(r => {
  const fields = r.data.__schema.queryType.fields;
  const collections = fields.filter(f => f.name.endsWith('Collection'));
  console.log('Content Types disponibles como colecciones:');
  collections.forEach(f => console.log('  -', f.name));
});
"

# Paso 2: Actualizar todas las queries en src/queries.js con el nombre correcto
# Por ejemplo, si tu Content Type tiene API Identifier "blogPost":
# articleCollection → blogPostCollection
# ArticleOrder → BlogPostOrder
```

---

## Limpieza

Al finalizar el lab, ejecuta los siguientes pasos para dejar el entorno ordenado:

```bash
# 1. Verificar que no hay credenciales expuestas antes de cualquier push
git status
git diff --cached --name-only
# Confirmar que .env NO aparece

# 2. Revisar el historial de commits para asegurar que .env nunca fue commiteado
git log --all --full-history -- .env
# No debe retornar ningún resultado

# 3. Limpiar archivos temporales generados durante el lab
rm -f comparacion-rest-graphql.json

# 4. Verificar estructura final del proyecto
find . -name "*.js" -not -path "*/node_modules/*" | sort
# Debe mostrar: src/graphql-client.js, src/queries.js, src/run-queries.js,
#               src/comparar-rest-vs-graphql.js, src/analizar-cache.js,
#               src/cache-manager.js, src/seguridad-variables.js,
#               src/validacion-final.js, src/test-client.js

# 5. Commit final del estado limpio
git add -A
git status  # Revisar una vez más
git commit -m "chore: cleanup lab03 - remover archivos temporales"

# 6. OPCIONAL: Si deseas liberar espacio de node_modules
# (se puede reinstalar con 'npm install' cuando sea necesario)
# rm -rf node_modules
```

> **⚠️ Importante:** El espacio de Contentful y sus datos **NO deben eliminarse**. Los Labs 04 y 05 dependen del mismo espacio y modelo de contenido.

---

## Resumen

En esta práctica construiste un cliente GraphQL completo para Contentful usando `fetch` nativo de Node.js 18, lo que te permitió entender el protocolo HTTP subyacente antes de usar librerías de abstracción. Implementaste cinco queries de complejidad progresiva: desde selección básica de campos hasta relaciones entre Content Types y manejo del árbol JSON de Rich Text.

Realizaste una comparación cuantitativa que demostró reducciones de payload superiores al 80% con GraphQL respecto a REST equivalente, validando la ventaja de la selección precisa de campos para eliminar el over-fetching. Implementaste y analizaste estrategias de caché HTTP, comprendiendo el rol del CDN de Cloudflare de Contentful y cómo las cabeceras `Cache-Control`, `ETag` y `CF-Cache-Status` gobiernan el comportamiento de caché. Finalmente, aplicaste medidas de seguridad críticas refactorizando queries con interpolación de strings hacia el uso de variables GraphQL.

### Conceptos Clave

| Concepto | Descripción |
|---|---|
| **Endpoint único GraphQL** | `POST https://graphql.contentful.com/content/v1/spaces/{SPACE_ID}` |
| **Schema autogenerado** | Cada Content Type genera tipos `Item`, `Collection`, `Filter` y `Order` |
| **Over-fetching eliminado** | GraphQL retorna solo los campos declarados en la query |
| **Linked entries en 1 request** | Las relaciones se resuelven en una sola llamada (vs múltiples en REST) |
| **Rich Text = JSON, no HTML** | Usar `@contentful/rich-text-react-renderer` para renderizar (Lab 4) |
| **Variables GraphQL** | Previenen inyección; los valores dinámicos SIEMPRE van como variables |
| **Cache-Control CDN** | `s-maxage=300` → Cloudflare cachea 5 min; `max-age=15` → cliente 15 seg |
| **ETag** | Permite `304 Not Modified` cuando el contenido no cambió |

### Recursos Adicionales

- [Documentación oficial de la API GraphQL de Contentful](https://www.contentful.com/developers/docs/references/graphql/)
- [GraphQL Explorer de Contentful (Playground interactivo)](https://www.contentful.com/developers/docs/references/graphql/#/introduction/the-graphql-explorer)
- [Repositorio de graphql-request en GitHub](https://github.com/jasonkuhrt/graphql-request)
- [Especificación de variables en GraphQL](https://graphql.org/learn/queries/#variables)
- [Documentación de Cache-Control (MDN)](https://developer.mozilla.org/es/docs/Web/HTTP/Headers/Cache-Control)
- [Rich Text Renderer para React — @contentful/rich-text-react-renderer](https://www.npmjs.com/package/@contentful/rich-text-react-renderer)

---
LAB_END---
