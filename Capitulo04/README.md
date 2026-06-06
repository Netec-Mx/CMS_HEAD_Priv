# Flujo completo publicación → despliegue

## Metadatos

| Campo | Valor |
|---|---|
| **Duración estimada** | 100 minutos |
| **Complejidad** | Alta |
| **Nivel Bloom** | Crear |
| **Módulo** | 4 — Integración con Frontend y Automatización |
| **Dependencias** | Labs 01, 02 y 03 completados |

---

## Descripción General

En esta práctica integrarás todos los conceptos estudiados hasta ahora en un flujo end-to-end completo: construirás un portal de noticias con React/Vite que consume contenido de Contentful, configurarás un Webhook que notifica a un servidor Express.js local cada vez que se publica o despublica una entrada, y explorarás el sistema de control de versiones de Contentful para realizar un rollback manual. Al finalizar tendrás un proyecto funcional que demuestra el ciclo completo desde la edición de contenido hasta el despliegue automatizado.

---

### Escenario de la práctica

El equipo frontend debe conectar el portal de noticias con Contentful y comprobar el flujo operativo que ocurre cuando una entrada se publica, genera un Webhook y requiere actualizar la experiencia entregada al usuario.

### Objetivo de la práctica

Construir el proyecto único `portal-noticias`, conectarlo al modelo canónico mediante `src/lib/contentfulClient.js` y validar el flujo de publicación, Webhook, actualización y rollback.

### Cómo trabajar esta práctica

1. Completa los bloques en orden; cada uno utiliza el resultado del anterior.
2. Mantén identificadas las terminales del frontend, servidor de Webhooks y ngrok.
3. Ejecuta la validación de cada bloque antes de modificar contenido o configuración adicional.
4. Conserva `portal-noticias` al finalizar; la Práctica 5 continúa sobre este mismo proyecto.

> **Importante:** El frontend solo debe usar el Delivery token. El secret del Webhook se configura en el servidor mediante `<WEBHOOK_SECRET>`.

---

## Objetivos de Aprendizaje

- [ ] Integrar la API de Contentful en una aplicación React (Vite) implementando los componentes `ArticleList` y `ArticleDetail` con renderizado de Rich Text.
- [ ] Comparar las estrategias CSR, SSR y SSG identificando cuándo aplicar cada una en proyectos con Contentful.
- [ ] Configurar un Webhook en Contentful y verificar la recepción del payload en un servidor Express.js local expuesto con ngrok.
- [ ] Implementar un endpoint Express.js que valide el secret del Webhook y simule la invalidación de caché.
- [ ] Explorar el historial de versiones de una entrada en Contentful y ejecutar un rollback a una versión anterior.

---

## Prerrequisitos

### Conocimientos previos
- Haber completado las Prácticas 1, 2 y 3 (espacio Contentful configurado, Content Types `article`, `category` y `author` creados, módulo de acceso a la API implementado).
- Componentes funcionales de React, hooks `useState` y `useEffect`.
- Conceptos básicos de REST y variables de entorno en Node.js.

### Acceso y cuentas
- Cuenta activa de Contentful (Plan Community) con el espacio de las prácticas anteriores.
- Cuenta de ngrok autenticada **o** acceso a [webhook.site](https://webhook.site) como alternativa.
- Acceso a terminal con permisos de instalación de paquetes npm.

### Dependencias a instalar antes de comenzar
```bash
# Verificar versiones mínimas
node --version   # >= 18.x
npm --version    # >= 9.x
ngrok --version  # >= 3.x (o usar webhook.site)
```

> **Compatibilidad de terminal:** Los comandos de Node.js, npm y Git funcionan en Bash y PowerShell. Para llamadas HTTP en PowerShell usa `Invoke-RestMethod` o escribe `curl.exe` explícitamente; `curl` puede ser un alias con comportamiento diferente.

---

## Entorno del Laboratorio

### Hardware recomendado

| Recurso | Mínimo | Recomendado |
|---|---|---|
| RAM | 8 GB | 16 GB |
| CPU | Intel Core i5 64-bit | i7 / Ryzen 5 |
| Disco libre | 20 GB | 30 GB |
| Conexión | 10 Mbps | 25 Mbps |

### Software requerido

| Herramienta | Versión | Notas |
|---|---|---|
| Node.js | 18.x LTS | Incluye npm 9.x |
| Visual Studio Code | 1.85+ | Editor principal |
| Vite (via npm create) | 5.x | Incluido en el paso de creación |
| React | 18.x | Template del proyecto |
| Express.js | 4.x | Servidor de Webhooks |
| @contentful/rich-text-react-renderer | 15.x | Renderizado de Rich Text |
| @contentful/rich-text-types | 16.x | Tipos para el árbol JSON |
| contentful SDK | 10.x | Acceso a la API |
| dotenv | 16.x | Variables de entorno |
| ngrok | 3.x | Túnel HTTP local |

### Configuración inicial del entorno

```bash
# 1. Crear el proyecto React con Vite
npm create vite@latest portal-noticias -- --template react
cd portal-noticias

# 2. Instalar dependencias del proyecto React
npm install contentful @contentful/rich-text-react-renderer @contentful/rich-text-types

# 3. Instalar dotenv (para el servidor Express en el Bloque 3)
npm install dotenv

# 4. Crear el proyecto del servidor de Webhooks (directorio separado)
mkdir -p ../webhook-server
cd ../webhook-server
npm init -y
npm install express dotenv

# 5. Volver al proyecto React
cd ../portal-noticias
```

---

## Instrucciones Paso a Paso

---

### Bloque 1 — Integración React con Contentful (40 minutos)

#### Paso 1.1 — Configurar variables de entorno de Vite

**Objetivo:** Proteger las credenciales de Contentful usando el sistema de variables de entorno de Vite (prefijo `VITE_`), nunca exponerlas en el código fuente.

> ⚠️ **CRÍTICO — Seguridad de credenciales:** Las variables `VITE_*` quedan embebidas en el bundle del cliente. Esto es aceptable **únicamente** con la `Content Delivery API Key` (solo lectura). La `Content Management API Key` jamás debe usarse en un proyecto frontend.

**Instrucciones:**

1. En la raíz de `portal-noticias`, crea el archivo `.env`:

```bash
# portal-noticias/.env
VITE_CONTENTFUL_SPACE_ID=<CONTENTFUL_SPACE_ID>
VITE_CONTENTFUL_DELIVERY_TOKEN=<CONTENTFUL_DELIVERY_TOKEN>
VITE_CONTENTFUL_ENVIRONMENT=<CONTENTFUL_ENVIRONMENT>
```

2. Crea el archivo `.env.example` (este SÍ se commitea):

```bash
# portal-noticias/.env.example
VITE_CONTENTFUL_SPACE_ID=<CONTENTFUL_SPACE_ID>
VITE_CONTENTFUL_DELIVERY_TOKEN=<CONTENTFUL_DELIVERY_TOKEN>
VITE_CONTENTFUL_ENVIRONMENT=<CONTENTFUL_ENVIRONMENT>
```

3. Verifica que `.gitignore` incluya `.env`:

```bash
# portal-noticias/.gitignore  (Vite lo genera automáticamente, verificar que contenga:)
.env
.env.local
.env.*.local
```

4. Si el proyecto usa Git, verifica que `.env` no está en staging:

```bash
git status   # .env NO debe aparecer como archivo a commitear
```

**Resultado esperado:** El archivo `.env` existe localmente pero no aparece en `git status` como archivo rastreado.

**Verificación:**
```bash
git check-ignore -v .env
# Criterio observable: Git muestra la regla de .gitignore que excluye .env
```

---

#### Paso 1.2 — Crear el cliente de Contentful centralizado

**Objetivo:** Centralizar la inicialización del SDK en un único módulo reutilizable, siguiendo la convención aprendida en la lección 4.1.

**Instrucciones:**

1. Crea la carpeta `src/lib/`:

```bash
node -e "require('fs').mkdirSync('src/lib',{recursive:true})"
```

2. Crea el archivo `src/lib/contentfulClient.js`:

```javascript
// src/lib/contentfulClient.js
import { createClient } from 'contentful';

// Nota: En Vite las variables de entorno usan el prefijo VITE_
// (diferente a Create React App que usa REACT_APP_)
const client = createClient({
  space: import.meta.env.VITE_CONTENTFUL_SPACE_ID,
  accessToken: import.meta.env.VITE_CONTENTFUL_DELIVERY_TOKEN,
  environment: import.meta.env.VITE_CONTENTFUL_ENVIRONMENT || 'master',
});

export default client;
```

> **Diferencia importante con CRA:** Vite usa `import.meta.env.VITE_*` en lugar de `process.env.REACT_APP_*`. Este es un error frecuente al migrar ejemplos de la documentación oficial.

**Resultado esperado:** El archivo `src/lib/contentfulClient.js` existe y exporta el cliente configurado.

**Verificación:**
```bash
node -e "const s=require('fs').readFileSync('src/lib/contentfulClient.js','utf8'); console.log('usa variables Vite:',s.includes('import.meta.env.VITE_CONTENTFUL_SPACE_ID')&&s.includes('import.meta.env.VITE_CONTENTFUL_DELIVERY_TOKEN'))"
```

---

#### Paso 1.3 — Implementar el componente `ArticleList`

**Objetivo:** Crear un componente que liste artículos de Contentful con capacidad de filtrar por categoría.

**Instrucciones:**

1. Crea la carpeta `src/components/`:

```bash
mkdir -p src/components
```

2. Crea el archivo `src/components/ArticleList.jsx`:

```jsx
// src/components/ArticleList.jsx
import { useEffect, useState } from 'react';
import client from '../lib/contentfulClient';

// Props:
//   onSelectArticle: función callback para navegar al detalle
//   selectedCategory: string con el ID de categoría para filtrar (opcional)
function ArticleList({ onSelectArticle, selectedCategory }) {
  const [articles, setArticles] = useState([]);
  const [categories, setCategories] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  // Cargar categorías disponibles al montar el componente
  useEffect(() => {
    client
      .getEntries({ content_type: 'category', order: 'fields.name' })
      .then((res) => setCategories(res.items))
      .catch((err) => console.error('Error al cargar categorías:', err));
  }, []);

  // Cargar artículos — se re-ejecuta cuando cambia el filtro de categoría
  useEffect(() => {
    setLoading(true);
    setError(null);

    // Construir query: si hay categoría seleccionada, filtrar por ella
    const query = {
      content_type: 'article',
      order: '-sys.createdAt', // Más recientes primero
    };

    // Filtro de referencia: busca artículos cuyo campo 'category'
    // apunte a la entrada con el ID indicado
    if (selectedCategory) {
      query['fields.category.sys.id'] = selectedCategory;
    }

    client
      .getEntries(query)
      .then((response) => {
        setArticles(response.items);
        setLoading(false);
      })
      .catch((err) => {
        setError('No se pudieron cargar los artículos. Verifica tu conexión.');
        setLoading(false);
        console.error('Error al obtener artículos:', err);
      });
  }, [selectedCategory]); // Dependencia: se re-ejecuta al cambiar la categoría

  if (loading) return <p className="loading">Cargando artículos...</p>;
  if (error) return <p className="error">{error}</p>;
  if (articles.length === 0) return <p>No hay artículos en esta categoría.</p>;

  return (
    <div className="article-list">
      <ul>
        {articles.map((article) => (
          <li
            key={article.sys.id}
            className="article-card"
            onClick={() => onSelectArticle(article.sys.id)}
            style={{ cursor: 'pointer' }}
          >
            {/* Imagen de portada (si existe) */}
            {article.fields.coverImage && (
              <img
                src={article.fields.coverImage.fields.file.url}
                alt={article.fields.coverImage.fields.title || article.fields.title}
                width={300}
              />
            )}
            <h2>{article.fields.title}</h2>
            {/* Categoría como badge */}
            {article.fields.category && (
              <span className="badge">
                {article.fields.category.fields?.name || 'Sin categoría'}
              </span>
            )}
            <small>
              Publicado: {new Date(article.fields.publishedAt).toLocaleDateString('es-MX')}
            </small>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default ArticleList;
```

**Resultado esperado:** El componente compila sin errores y muestra artículos cuando hay datos disponibles.

---

#### Paso 1.4 — Implementar el componente `ArticleDetail` con Rich Text

**Objetivo:** Renderizar el detalle de un artículo incluyendo el campo Rich Text usando `@contentful/rich-text-react-renderer`.

> ⚠️ **Advertencia sobre Rich Text:** El campo Rich Text de Contentful **NO retorna HTML**. Retorna un árbol de nodos JSON (objeto `Document`). Intentar renderizarlo directamente como `{article.fields.body}` mostrará `[object Object]`. Debes usar el renderer oficial.

**Instrucciones:**

1. Crea el archivo `src/components/ArticleDetail.jsx`:

```jsx
// src/components/ArticleDetail.jsx
import { useEffect, useState } from 'react';
import { documentToReactComponents } from '@contentful/rich-text-react-renderer';
import { BLOCKS, INLINES } from '@contentful/rich-text-types';
import client from '../lib/contentfulClient';

// Opciones de renderizado personalizadas para el Rich Text
// Permiten mapear cada tipo de nodo a un componente React específico
const richTextOptions = {
  renderNode: {
    // Párrafos normales
    [BLOCKS.PARAGRAPH]: (node, children) => (
      <p style={{ lineHeight: '1.7', marginBottom: '1rem' }}>{children}</p>
    ),
    // Encabezados
    [BLOCKS.HEADING_2]: (node, children) => (
      <h2 style={{ marginTop: '2rem', borderBottom: '1px solid #eee' }}>{children}</h2>
    ),
    [BLOCKS.HEADING_3]: (node, children) => (
      <h3 style={{ marginTop: '1.5rem' }}>{children}</h3>
    ),
    // Imágenes embebidas en el Rich Text
    [BLOCKS.EMBEDDED_ASSET]: (node) => {
      const { file, title } = node.data.target.fields;
      return (
        <figure style={{ margin: '2rem 0' }}>
          <img
            src={file.url}
            alt={title}
            style={{ maxWidth: '100%', borderRadius: '8px' }}
          />
          <figcaption style={{ fontSize: '0.85rem', color: '#666' }}>{title}</figcaption>
        </figure>
      );
    },
    // Hipervínculos en línea
    [INLINES.HYPERLINK]: (node, children) => (
      <a href={node.data.uri} target="_blank" rel="noopener noreferrer">
        {children}
      </a>
    ),
  },
};

// Props:
//   articleId: string con el ID de la entrada a mostrar
//   onBack: función callback para volver al listado
function ArticleDetail({ articleId, onBack }) {
  const [article, setArticle] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    if (!articleId) return;

    setLoading(true);
    // getEntry obtiene una sola entrada por ID
    // El SDK resuelve automáticamente las referencias (linked entries/assets)
    client
      .getEntry(articleId)
      .then((entry) => {
        setArticle(entry);
        setLoading(false);
      })
      .catch((err) => {
        setError('No se pudo cargar el artículo.');
        setLoading(false);
        console.error('Error al obtener entrada:', err);
      });
  }, [articleId]);

  if (loading) return <p className="loading">Cargando artículo...</p>;
  if (error) return <p className="error">{error}</p>;
  if (!article) return null;

  const { title, body, coverImage, category, author } = article.fields;

  return (
    <article className="article-detail">
      <button onClick={onBack} style={{ marginBottom: '1rem' }}>
        ← Volver al listado
      </button>

      {coverImage && (
        <img
          src={coverImage.fields.file.url}
          alt={coverImage.fields.title}
          style={{ width: '100%', maxHeight: '400px', objectFit: 'cover', borderRadius: '8px' }}
        />
      )}

      <h1>{title}</h1>

      {category && (
        <span className="badge">
          {category.fields?.name || 'Sin categoría'}
        </span>
      )}

      {author && <p><strong>Autor:</strong> {author.fields?.name || 'Autor desconocido'}</p>}

      {/* Renderizado del Rich Text — usa documentToReactComponents */}
      {body ? (
        <div className="rich-text-content">
          {documentToReactComponents(body, richTextOptions)}
        </div>
      ) : (
        <p><em>Este artículo no tiene contenido principal.</em></p>
      )}

      <hr />
      <small>
        ID de entrada: <code>{article.sys.id}</code> |{' '}
        Versión: <code>{article.sys.revision}</code> |{' '}
        Última actualización: {new Date(article.sys.updatedAt).toLocaleString('es-MX')}
      </small>
    </article>
  );
}

export default ArticleDetail;
```

**Resultado esperado:** El componente renderiza el contenido Rich Text como elementos HTML semánticos, no como `[object Object]`.

---

#### Paso 1.5 — Ensamblar la aplicación en `App.jsx`

**Objetivo:** Conectar `ArticleList` y `ArticleDetail` con navegación basada en estado (sin React Router para mantener el alcance del lab).

**Instrucciones:**

1. Reemplaza el contenido de `src/App.jsx`:

```jsx
// src/App.jsx
import { useState } from 'react';
import ArticleList from './components/ArticleList';
import ArticleDetail from './components/ArticleDetail';
import './App.css';

function App() {
  // null = vista de listado | string = ID del artículo seleccionado
  const [selectedArticleId, setSelectedArticleId] = useState(null);
  // null = sin filtro | string = ID de categoría seleccionada
  const [selectedCategory, setSelectedCategory] = useState(null);

  return (
    <div className="app-container" style={{ maxWidth: '900px', margin: '0 auto', padding: '2rem' }}>
      <header style={{ borderBottom: '2px solid #0070f3', marginBottom: '2rem' }}>
        <h1
          onClick={() => setSelectedArticleId(null)}
          style={{ cursor: 'pointer', color: '#0070f3' }}
        >
          📰 Portal de Noticias — Contentful + React
        </h1>
        <p style={{ color: '#666', fontSize: '0.9rem' }}>
          Integración headless CMS con React/Vite
        </p>
      </header>

      <main>
        {selectedArticleId ? (
          // Vista de detalle
          <ArticleDetail
            articleId={selectedArticleId}
            onBack={() => setSelectedArticleId(null)}
          />
        ) : (
          // Vista de listado con filtro
          <>
            <ArticleList
              onSelectArticle={(id) => setSelectedArticleId(id)}
              selectedCategory={selectedCategory}
            />
          </>
        )}
      </main>

      <footer style={{ marginTop: '3rem', borderTop: '1px solid #eee', paddingTop: '1rem', color: '#999', fontSize: '0.8rem' }}>
        Lab 04-00-01 | Contentful Headless CMS | Plan Community
      </footer>
    </div>
  );
}

export default App;
```

2. Inicia el servidor de desarrollo:

```bash
npm run dev
```

3. Abre `http://localhost:5173` en el navegador.

**Resultado esperado:** La aplicación muestra el listado de artículos. Al hacer clic en uno, navega al detalle con el Rich Text renderizado correctamente.

**Verificación:**
- [ ] El listado muestra al menos 2 artículos de Contentful.
- [ ] Al hacer clic en un artículo, se muestra la vista de detalle.
- [ ] El contenido Rich Text se renderiza como párrafos e imágenes, no como `[object Object]`.
- [ ] El botón "← Volver al listado" funciona correctamente.
- [ ] La consola del navegador no muestra errores de autenticación.

---

### Bloque 2 — Análisis Conceptual: CSR vs. SSR vs. SSG (15 minutos)

#### Paso 2.1 — Comparativa de estrategias de renderizado con Contentful

**Objetivo:** Comprender cuándo usar cada estrategia de renderizado en proyectos reales con Contentful, sin implementar Next.js pero documentando el patrón.

**Instrucciones:**

1. Crea el archivo `docs/rendering-strategies.md` en el proyecto:

```bash
mkdir -p docs
```

2. Crea `docs/rendering-strategies.md` con el siguiente análisis:

```markdown
# Estrategias de Renderizado con Contentful

## 1. Client-Side Rendering (CSR) — Lo que hicimos en este lab

**Cómo funciona:** El servidor envía un HTML vacío. El navegador descarga el JS,
lo ejecuta y luego hace la llamada a Contentful. El contenido aparece después.

**Cuándo usarlo con Contentful:**
- Dashboards internos o áreas autenticadas donde el SEO no importa.
- Aplicaciones con contenido personalizado por usuario.
- Prototipos rápidos y portales de administración.

**Desventaja:** El contenido no es indexable por buscadores en la primera carga.

## 2. Server-Side Rendering (SSR) — Next.js getServerSideProps

**Cómo funciona:** En cada request, el servidor llama a Contentful, obtiene los
datos y genera el HTML completo antes de enviarlo al navegador.

**Cuándo usarlo con Contentful:**
- Páginas con contenido que cambia frecuentemente (noticias en tiempo real).
- Contenido personalizado que requiere datos del usuario en el servidor.

**Snippet ilustrativo (Next.js — NO implementado en este lab):**
```

```javascript
// pages/articles/[id].js (Next.js — solo referencia conceptual)
import { createClient } from 'contentful';

export async function getServerSideProps({ params }) {
  const client = createClient({
    space: process.env.CONTENTFUL_SPACE_ID,
    accessToken: process.env.CONTENTFUL_DELIVERY_TOKEN,
  });

  const entry = await client.getEntry(params.id);

  return {
    props: {
      article: entry.fields,
    },
  };
}

export default function ArticlePage({ article }) {
  return <h1>{article.title}</h1>;
}
```

```markdown
## 3. Static Site Generation (SSG) — Next.js getStaticProps + getStaticPaths

**Cómo funciona:** En tiempo de BUILD, el servidor llama a Contentful una sola vez,
genera todos los HTML estáticos y los sirve como archivos. Velocidad máxima.

**Cuándo usarlo con Contentful:**
- Blogs, portales de noticias, documentación.
- Contenido que cambia pocas veces al día.
- Máximo rendimiento y SEO.

**Patrón con Webhooks:** Contentful dispara un Webhook al publicar → el servidor
de CI/CD recibe la señal → ejecuta el rebuild → despliega los nuevos estáticos.
Este es exactamente el patrón que implementamos en el Bloque 3.

**Snippet ilustrativo (Next.js — solo referencia):**
```

```javascript
// pages/articles/[id].js (Next.js — solo referencia conceptual)
export async function getStaticPaths() {
  const client = createClient({ /* config */ });
  const entries = await client.getEntries({ content_type: 'article' });

  return {
    paths: entries.items.map((item) => ({
      params: { id: item.sys.id },
    })),
    fallback: 'blocking', // ISR: regenera páginas nuevas bajo demanda
  };
}

export async function getStaticProps({ params }) {
  const client = createClient({ /* config */ });
  const entry = await client.getEntry(params.id);

  return {
    props: { article: entry.fields },
    revalidate: 60, // ISR: regenera cada 60 segundos
  };
}
```

```markdown
## Tabla de decisión rápida

| Criterio                    | CSR (React puro) | SSR (Next.js) | SSG (Next.js) |
|-----------------------------|:----------------:|:-------------:|:-------------:|
| SEO importante              | ❌               | ✅            | ✅            |
| Contenido cambia en tiempo real | ✅           | ✅            | ❌            |
| Máximo rendimiento          | ❌               | ⚠️            | ✅            |
| Webhooks para rebuild       | N/A              | N/A           | ✅            |
| Complejidad de implementación | Baja           | Media         | Media-Alta    |
```

3. Revisa el archivo creado y discute con tu instructor las implicaciones de cada estrategia para el proyecto de clase.

**Resultado esperado:** Archivo `docs/rendering-strategies.md` creado con el análisis comparativo. Comprensión clara de por qué el patrón Webhook → rebuild es esencial en SSG.

---

### Bloque 3 — Webhooks: Servidor Express.js + ngrok (30 minutos)

#### Paso 3.1 — Crear el servidor Express.js para recibir Webhooks

**Objetivo:** Implementar un servidor mínimo que reciba notificaciones HTTP de Contentful, valide el secret y simule la invalidación de caché.

**Instrucciones:**

1. Navega al directorio del servidor de Webhooks:

```bash
cd ../webhook-server
```

2. Crea el archivo `.env`:

```bash
# webhook-server/.env
WEBHOOK_SECRET=<WEBHOOK_SECRET>
PORT=3001
```

3. Crea el archivo `.env.example`:

```bash
# webhook-server/.env.example
WEBHOOK_SECRET=<WEBHOOK_SECRET>
PORT=3001
```

4. Crea el archivo `.gitignore`:

```bash
# Bash:
printf ".env\nnode_modules/\n" > .gitignore
```

```powershell
# PowerShell:
@('.env','node_modules/') | Set-Content .gitignore
```

5. Crea el archivo principal `server.js`:

```javascript
// webhook-server/server.js
require('dotenv').config();
const express = require('express');

const app = express();
const PORT = process.env.PORT || 3001;
const WEBHOOK_SECRET = process.env.WEBHOOK_SECRET;

// Middleware: parsear el body como JSON
// Contentful envía el payload como application/json
app.use(express.json());

// ============================================================
// Endpoint principal: POST /webhook
// Contentful enviará aquí las notificaciones de eventos
// ============================================================
app.post('/webhook', (req, res) => {
  // --- 1. Validar el secret de Contentful ---
  // Contentful puede enviar un header personalizado con el secret
  // que configuramos en el panel. Esto evita que cualquiera
  // pueda disparar nuestro endpoint.
  const receivedSecret = req.headers['x-contentful-webhook-secret'];

  if (receivedSecret !== WEBHOOK_SECRET) {
    console.warn(`[${new Date().toISOString()}] ⛔ Secret inválido recibido: ${receivedSecret}`);
    return res.status(401).json({ error: 'Secret inválido' });
  }

  // --- 2. Extraer información del evento ---
  const payload = req.body;
  const eventType = req.headers['x-contentful-topic'];    // e.g. "ContentManagement.Entry.publish"
  const spaceId = req.headers['x-contentful-space-id'];   // ID del espacio

  console.log('\n============================================================');
  console.log(`[${new Date().toISOString()}] 📨 Webhook recibido`);
  console.log(`  Evento:   ${eventType}`);
  console.log(`  Espacio:  ${spaceId}`);

  // --- 3. Extraer datos de la entrada afectada ---
  if (payload && payload.sys) {
    const entryId = payload.sys.id;
    const contentType = payload.sys.contentType?.sys?.id || 'desconocido';
    const environment = payload.sys.environment?.sys?.id || 'master';

    console.log(`  Entrada:  ${entryId}`);
    console.log(`  Tipo:     ${contentType}`);
    console.log(`  Entorno:  ${environment}`);
  }

  // --- 4. Simular invalidación de caché / trigger de rebuild ---
  simulateCacheInvalidation(eventType, payload);

  // --- 5. Responder 200 OK a Contentful ---
  // Es importante responder rápido (< 10 segundos) para que
  // Contentful no marque el Webhook como fallido.
  res.status(200).json({
    message: 'Webhook procesado correctamente',
    timestamp: new Date().toISOString(),
  });
});

// ============================================================
// Función: simular invalidación de caché
// En un proyecto real, aquí llamarías a:
//   - API de Vercel/Netlify para invalidar caché
//   - Script de rebuild del sitio estático
//   - Sistema de mensajería (Redis pub/sub, SQS, etc.)
// ============================================================
function simulateCacheInvalidation(eventType, payload) {
  const entryId = payload?.sys?.id;

  if (eventType?.includes('publish')) {
    console.log(`  ✅ ACCIÓN: Invalidando caché para entrada ${entryId}`);
    console.log('  ✅ ACCIÓN: Iniciando rebuild del sitio estático...');
    console.log('  ✅ ACCIÓN: [SIMULADO] Notificando a CDN...');
    // En producción: await triggerNetlifyBuild(process.env.NETLIFY_BUILD_HOOK);
  } else if (eventType?.includes('unpublish')) {
    console.log(`  🔴 ACCIÓN: Entrada ${entryId} despublicada`);
    console.log('  🔴 ACCIÓN: Eliminando página del sitio estático...');
    console.log('  🔴 ACCIÓN: [SIMULADO] Actualizando índice de búsqueda...');
  } else {
    console.log(`  ℹ️  ACCIÓN: Evento no manejado: ${eventType}`);
  }
  console.log('============================================================\n');
}

// ============================================================
// Endpoint de health check (útil para verificar que el servidor
// está corriendo antes de configurar el Webhook)
// ============================================================
app.get('/health', (req, res) => {
  res.json({
    status: 'ok',
    timestamp: new Date().toISOString(),
    message: 'Servidor de Webhooks activo',
  });
});

// Iniciar servidor
app.listen(PORT, () => {
  console.log(`\n🚀 Servidor de Webhooks escuchando en http://localhost:${PORT}`);
  console.log(`   Health check: http://localhost:${PORT}/health`);
  console.log(`   Endpoint Webhook: POST http://localhost:${PORT}/webhook\n`);
});
```

6. Inicia el servidor:

```bash
node server.js
```

**Resultado esperado:**
```
🚀 Servidor de Webhooks escuchando en http://localhost:3001
   Health check: http://localhost:3001/health
   Endpoint Webhook: POST http://localhost:3001/webhook
```

**Verificación:**
```bash
# En otra terminal, verificar el health check
curl http://localhost:3001/health
# Debe responder: {"status":"ok","timestamp":"...","message":"Servidor de Webhooks activo"}
```

---

#### Paso 3.2 — Exponer el servidor local con ngrok

**Objetivo:** Crear un túnel HTTPS público para que Contentful pueda alcanzar el servidor local.

**Instrucciones:**

**Opción A — ngrok (recomendado):**

1. En una **nueva terminal**, expón el puerto 3001:

```bash
ngrok http 3001
```

2. Copia la URL HTTPS generada (formato `https://xxxx-xx-xx-xxx-xx.ngrok-free.app`).

3. Verifica el túnel:

```bash
curl https://TU-URL-NGROK.ngrok-free.app/health
# Debe responder el mismo JSON que el health check local
```

**Opción B — Webhook.site (alternativa sin instalación):**

1. Ve a [https://webhook.site](https://webhook.site).
2. Copia la URL única generada (formato `https://webhook.site/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`).
3. Nota: Con esta opción solo podrás **ver** el payload, no ejecutar lógica de servidor. Es útil para la verificación inicial.

**Resultado esperado (Opción A):** ngrok muestra la URL pública y el estado `online`. Las peticiones al endpoint público se reenvían al servidor local.

**Verificación:**
```
Forwarding  https://xxxx.ngrok-free.app -> http://localhost:3001
```

---

#### Paso 3.3 — Configurar el Webhook en el panel de Contentful

**Objetivo:** Registrar el endpoint del servidor Express en Contentful para recibir notificaciones de publicación.

**Instrucciones:**

1. Ve a [app.contentful.com](https://app.contentful.com) → tu espacio → **Settings** → **Webhooks**.

2. Haz clic en **Add Webhook**.

3. Completa el formulario:

| Campo | Valor |
|---|---|
| **Name** | `Lab 04 - Rebuild Trigger` |
| **URL** | `https://TU-URL-NGROK.ngrok-free.app/webhook` |
| **Method** | POST |
| **Triggers** | Entry: Publish ✅, Entry: Unpublish ✅ |
| **Content type filter** | `article` (solo artículos) |

4. En la sección **Headers**, agrega un header personalizado:

| Header | Valor |
|---|---|
| `X-Contentful-Webhook-Secret` | `<WEBHOOK_SECRET>` |

> ⚠️ **Seguridad:** Este secret debe coincidir exactamente con `WEBHOOK_SECRET` en el `.env` del servidor. En producción, usa un valor aleatorio de al menos 32 caracteres.

5. Haz clic en **Save**.

6. Usa el botón **Send test notification** de Contentful para verificar la conectividad.

**Resultado esperado:** El panel de Contentful muestra el Webhook como activo (indicador verde). El servidor Express muestra en consola el evento de prueba.

**Verificación en la consola del servidor:**
```
============================================================
[2024-XX-XX] 📨 Webhook recibido
  Evento:   ContentManagement.Entry.publish
  Espacio:  <CONTENTFUL_SPACE_ID>
  Entrada:  xxxxxxxxxxxxxxxx
  Tipo:     article
  ✅ ACCIÓN: Invalidando caché para entrada xxxxxxxxxxxxxxxx
  ✅ ACCIÓN: Iniciando rebuild del sitio estático...
============================================================
```

---

#### Paso 3.4 — Verificar el flujo completo publicar → Webhook

**Objetivo:** Disparar el Webhook de forma real editando y publicando una entrada en Contentful.

**Instrucciones:**

1. En el panel de Contentful, ve a **Content** → selecciona un artículo existente.

2. Edita el campo `title` añadiendo " (v2)" al final.

3. Haz clic en **Publish** (o **Republish** si ya estaba publicado).

4. Observa la consola del servidor Express.

5. Ahora haz clic en **Unpublish** en el mismo artículo.

6. Observa nuevamente la consola del servidor.

**Resultado esperado:**

```
# Al publicar:
============================================================
[2024-XX-XX] 📨 Webhook recibido
  Evento:   ContentManagement.Entry.publish
  ...
  ✅ ACCIÓN: Invalidando caché para entrada abc123
  ✅ ACCIÓN: Iniciando rebuild del sitio estático...
============================================================

# Al despublicar:
============================================================
[2024-XX-XX] 📨 Webhook recibido
  Evento:   ContentManagement.Entry.unpublish
  ...
  🔴 ACCIÓN: Entrada abc123 despublicada
  🔴 ACCIÓN: Eliminando página del sitio estático...
============================================================
```

7. En el panel de Contentful → **Settings** → **Webhooks** → tu Webhook → pestaña **Activity log**, verifica que las llamadas muestran código `200 OK`.

**Verificación:**
- [ ] El servidor recibe el evento `publish` con código 200.
- [ ] El servidor recibe el evento `unpublish` con código 200.
- [ ] El Activity log de Contentful muestra `200 OK` para ambas llamadas.
- [ ] El secret inválido devuelve `401 Unauthorized` (prueba enviando un header incorrecto con Postman).

---

### Bloque 4 — Control de Versiones de Contenido (15 minutos)

#### Paso 4.1 — Explorar el historial de versiones en Contentful

**Objetivo:** Entender cómo Contentful gestiona las versiones de una entrada y cómo comparar cambios entre versiones.

**Instrucciones:**

1. En el panel de Contentful → **Content**, selecciona el artículo que editaste en el paso anterior.

2. En la barra lateral derecha, busca la sección **Version history** (o **Versiones**).

3. Haz clic para expandirla. Verás una lista de versiones numeradas con timestamp y autor.

4. Edita el artículo dos veces más (cambia el título y el cuerpo en cada edición) y publica cada vez:

   - Versión A: Título original
   - Versión B: Título + "(v2)"
- Versión C: Título + "(v3)" + cuerpo modificado

5. Regresa al historial de versiones. Deberías ver al menos 3 versiones.

6. Haz clic en una versión anterior para ver su contenido. Contentful mostrará una vista de comparación.

**Resultado esperado:** El panel muestra el historial de versiones con timestamps, autores y la opción de restaurar cada versión.

---

#### Paso 4.2 — Realizar un rollback a una versión anterior

**Objetivo:** Restaurar una entrada a una versión anterior y verificar el cambio a través de la API.

**Instrucciones:**

1. En el historial de versiones, localiza la **Versión A** (la original con el título sin modificar).

2. Haz clic en **Restore this version** (o el botón equivalente en el panel).

3. Contentful cargará el contenido de esa versión en el editor. **No se publica automáticamente** — debes revisar y publicar manualmente.

4. Verifica el contenido restaurado y haz clic en **Publish**.

5. Ahora verifica el cambio a través de la API desde la terminal:

```bash
# Bash; reemplaza ENTRY_ID y los placeholders
curl -H "Authorization: Bearer <CONTENTFUL_DELIVERY_TOKEN>" \
  "https://cdn.contentful.com/spaces/<CONTENTFUL_SPACE_ID>/environments/<CONTENTFUL_ENVIRONMENT>/entries/ENTRY_ID" \
  | node -e "
    const chunks = [];
    process.stdin.on('data', c => chunks.push(c));
    process.stdin.on('end', () => {
      const data = JSON.parse(chunks.join(''));
      console.log('Título actual:', data.fields.title);
      console.log('Versión sys:', data.sys.revision);
    });
  "
```

```powershell
# PowerShell
$headers = @{ Authorization = 'Bearer <CONTENTFUL_DELIVERY_TOKEN>' }
$data = Invoke-RestMethod -Headers $headers -Uri 'https://cdn.contentful.com/spaces/<CONTENTFUL_SPACE_ID>/environments/<CONTENTFUL_ENVIRONMENT>/entries/ENTRY_ID'
$data.fields.title
$data.sys.revision
```

6. Verifica que el título corresponde a la versión restaurada y que `sys.revision` ha incrementado (Contentful siempre incrementa el número de revisión, incluso en rollbacks).

**Resultado esperado:**
```
Título actual: Mi Artículo Original
Versión sys: 7
```

> **Nota conceptual:** Contentful no "retrocede" el número de versión al hacer un rollback. En cambio, crea una **nueva versión** con el contenido de la versión anterior. Esto garantiza un historial de auditoría completo e inmutable.

**Verificación:**
- [ ] El título del artículo en la API refleja el contenido de la versión restaurada.
- [ ] El número `sys.revision` es mayor que el de la última versión publicada (evidencia de que se creó una nueva versión).
- [ ] El historial de versiones en Contentful muestra la restauración como una nueva entrada en el log.

---

## Validación final

### Lista de verificación final

Ejecuta las siguientes verificaciones para confirmar que todos los bloques funcionan correctamente:

**Bloque 1 — Integración React:**
```bash
# Desde portal-noticias/
npm run dev
# Abrir http://localhost:5173 y verificar:
```
- [ ] La aplicación carga sin errores en consola del navegador.
- [ ] El listado muestra artículos reales de Contentful.
- [ ] La navegación al detalle funciona.
- [ ] El Rich Text se renderiza como HTML (párrafos, encabezados, imágenes).
- [ ] Las variables de entorno están en `.env` y NO en el código fuente.

**Bloque 2 — Análisis conceptual:**
- [ ] El archivo `docs/rendering-strategies.md` existe y contiene la comparativa.
- [ ] Puedes explicar verbalmente cuándo usar CSR vs. SSG con Contentful.

**Bloque 3 — Webhooks:**
```bash
# Bash: secret incorrecto debe devolver 401
curl -i -X POST http://localhost:3001/webhook \
  -H "Content-Type: application/json" \
  -H "X-Contentful-Webhook-Secret: secret_incorrecto" \
  -d '{"sys":{"id":"test123"}}'
# Criterio observable: status 401 y mensaje de secret inválido
```

```powershell
# PowerShell: un status 401 se mostrará como respuesta de error esperada
Invoke-RestMethod -Method Post -Uri 'http://localhost:3001/webhook' -Headers @{'X-Contentful-Webhook-Secret'='secret_incorrecto'} -ContentType 'application/json' -Body '{"sys":{"id":"test123"}}'
```
- [ ] El servidor responde `401` con secret incorrecto.
- [ ] El servidor responde `200` con secret correcto.
- [ ] El Activity log de Contentful muestra `200 OK` para eventos reales.

**Bloque 4 — Control de versiones:**
```bash
# Bash
curl -H "Authorization: Bearer <CONTENTFUL_DELIVERY_TOKEN>" "https://cdn.contentful.com/spaces/<CONTENTFUL_SPACE_ID>/environments/<CONTENTFUL_ENVIRONMENT>/entries/ENTRY_ID" | node -e "let s='';process.stdin.on('data',c=>s+=c);process.stdin.on('end',()=>{const d=JSON.parse(s);console.log(d.fields.title,d.sys.revision)})"
```

```powershell
# PowerShell
$data = Invoke-RestMethod -Headers @{Authorization='Bearer <CONTENTFUL_DELIVERY_TOKEN>'} -Uri 'https://cdn.contentful.com/spaces/<CONTENTFUL_SPACE_ID>/environments/<CONTENTFUL_ENVIRONMENT>/entries/ENTRY_ID'
$data.fields.title
$data.sys.revision
```
- [ ] El título refleja la versión restaurada.
- [ ] `sys.revision` es mayor al esperado (confirma que se creó nueva versión).

---

## Solución de Problemas

### Problema 1: El Rich Text se muestra como `[object Object]` en el navegador

**Síntomas:** El componente `ArticleDetail` muestra literalmente `[object Object]` donde debería estar el contenido del artículo, o la consola muestra un error de React indicando que un objeto no es un elemento React válido.

**Causa:** El campo `body` de tipo Rich Text en Contentful retorna un árbol de nodos JSON (objeto `Document`), no una cadena de texto HTML. Intentar renderizarlo directamente como `{article.fields.body}` dentro de JSX provoca este error porque React no sabe cómo convertir un objeto plano en elementos visuales.

**Solución:**
```bash
# Verificar que la dependencia está instalada
npm list @contentful/rich-text-react-renderer
# Si no aparece, instalarla:
npm install @contentful/rich-text-react-renderer @contentful/rich-text-types
```

```jsx
// ❌ INCORRECTO — causa [object Object]
<div>{article.fields.body}</div>

// ✅ CORRECTO — usar el renderer oficial
import { documentToReactComponents } from '@contentful/rich-text-react-renderer';
<div>{documentToReactComponents(article.fields.body)}</div>
```

Verifica también que el campo `body` en tu Content Type de Contentful sea de tipo **Rich Text** y no **Long Text**. Si es Long Text, retorna una cadena y no necesita el renderer.

---

### Problema 2: El Webhook recibe `401 Unauthorized` en el Activity log de Contentful

**Síntomas:** En el panel de Contentful → Settings → Webhooks → Activity log, todas las llamadas muestran código `401`. El servidor Express registra en consola `⛔ Secret inválido recibido: undefined` o el secret recibido no coincide.

**Causa:** El header `X-Contentful-Webhook-Secret` configurado en el panel de Contentful no coincide exactamente con el valor `WEBHOOK_SECRET` en el archivo `.env` del servidor. Las causas más comunes son: espacios en blanco al inicio/final del valor, comillas incluidas accidentalmente en el `.env`, o el servidor no recargó las variables de entorno después de modificar `.env`.

**Solución:**

1. Verifica que la variable existe sin imprimir el secret:
```bash
# En webhook-server/
node -e "require('dotenv').config(); console.log('WEBHOOK_SECRET definido:',Boolean(process.env.WEBHOOK_SECRET),'longitud:',process.env.WEBHOOK_SECRET?.length||0)"
# Criterio observable: definido=true y longitud mayor que 0
```

2. Reinicia el servidor Express para que recargue las variables:
```bash
# Detener con Ctrl+C y reiniciar:
node server.js
```

3. Agrega un log temporal para depurar el header recibido:
```javascript
// Añadir temporalmente en el endpoint /webhook:
console.log('Headers recibidos:', JSON.stringify(req.headers, null, 2));
```

4. Verifica que en el panel de Contentful el header está configurado exactamente como `X-Contentful-Webhook-Secret` (case-insensitive en HTTP, pero verificar que el nombre sea correcto).

5. Si usas ngrok, verifica que el túnel sigue activo (ngrok tiene timeout en el plan gratuito). Reinicia ngrok si es necesario y actualiza la URL en el panel de Contentful.

---

## Limpieza del Entorno

Al finalizar el laboratorio, realiza las siguientes acciones para mantener el entorno ordenado:

```bash
# 1. Detener el servidor de desarrollo de React
# (Ctrl+C en la terminal donde corre npm run dev)

# 2. Detener el servidor Express de Webhooks
# (Ctrl+C en la terminal donde corre node server.js)

# 3. Detener ngrok
# (Ctrl+C en la terminal donde corre ngrok)

# 4. En Contentful: deshabilitar el Webhook para evitar llamadas fallidas
# Settings → Webhooks → tu Webhook → toggle "Active" → OFF
# (No eliminarlo — lo reutilizarás en prácticas futuras)

# 5. Verificar que los artículos editados están en estado publicado
# para no afectar las prácticas siguientes

# 6. Opcional: hacer commits locales del trabajo realizado (SIN .env)
cd portal-noticias
git add .
git status  # Verificar que .env NO aparece en la lista
git commit -m "feat: Lab 04 - Portal noticias React + Webhook server"

cd ../webhook-server
git add .
git status  # Verificar que .env NO aparece
git commit -m "feat: Lab 04 - Express Webhook server"
```

> **Recordatorio:** El plan Community de Contentful permite 1 espacio y 2 entornos. No crees espacios adicionales. Reutiliza el mismo espacio en todos los labs restantes.

---

## Resumen

En esta práctica construiste un flujo end-to-end completo que integra Contentful con el ecosistema moderno de desarrollo frontend:

| Bloque | Logro |
|---|---|
| **Bloque 1** | Aplicación React/Vite con `ArticleList` y `ArticleDetail`, consumiendo Contentful con Rich Text renderizado correctamente |
| **Bloque 2** | Comprensión conceptual de CSR vs. SSR vs. SSG y su relación con el patrón Webhook → rebuild |
| **Bloque 3** | Servidor Express.js que recibe, valida y procesa Webhooks de Contentful con simulación de invalidación de caché |
| **Bloque 4** | Exploración del historial de versiones y rollback verificado a través de la API |

### Conceptos clave reforzados

- **Variables de entorno en Vite** usan el prefijo `VITE_*` y se acceden con `import.meta.env`, no con `process.env`.
- **Rich Text** en Contentful es un árbol JSON, no HTML. Siempre usar `documentToReactComponents()`.
- **Webhooks** permiten desacoplar el CMS del proceso de despliegue, habilitando el patrón de SSG con rebuild automático.
- **El rollback en Contentful crea una nueva versión** — el historial es inmutable y sirve como registro de auditoría.
- Las **API Keys de entrega** (solo lectura) son las únicas que pueden usarse en el frontend.

### Preguntas de reflexión

1. ¿Qué criterio usarías para elegir entre CSR, SSR y SSG en este portal?
2. ¿Qué garantiza un Webhook y qué responsabilidades siguen perteneciendo a la aplicación?
3. ¿Por qué un rollback crea una nueva versión en lugar de eliminar el historial?

### Recursos adicionales

- [Documentación oficial: @contentful/rich-text-react-renderer](https://github.com/contentful/rich-text/tree/master/packages/rich-text-react-renderer)
- [Contentful Webhooks — Referencia completa](https://www.contentful.com/developers/docs/concepts/webhooks/)
- [Guía de versiones de contenido en Contentful](https://www.contentful.com/help/version-control/)
- [Vite — Variables de entorno y modos](https://vitejs.dev/guide/env-and-mode.html)
- [Express.js — Guía oficial](https://expressjs.com/es/guide/routing.html)
- [ngrok — Documentación de inicio rápido](https://ngrok.com/docs/getting-started/)

---
