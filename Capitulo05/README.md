---LAB_START---
LAB_ID: 05-00-01
---MARKDOWN---
# Proyecto Demo funcional end-to-end

## Metadatos

| Atributo        | Valor                                      |
|-----------------|--------------------------------------------|
| **Duración**    | 30 minutos                                 |
| **Complejidad** | Media                                      |
| **Nivel Bloom** | Crear (Create)                             |
| **Módulo**      | 5 — Proyecto Demo Funcional End-to-End     |
| **Versión**     | 1.0                                        |

---

## Descripción General

Esta práctica integradora consolida todos los aprendizajes del curso construyendo un **portal digital demo funcional end-to-end**. El estudiante diseñará un modelo de contenido completo con al menos tres Content Types interrelacionados, lo implementará en Contentful, conectará un frontend React que consuma la API, aplicará todas las medidas de seguridad aprendidas y ejecutará un checklist técnico formal de 20 ítems antes de considerar el proyecto terminado. La temática del portal es libre: blog tecnológico, portafolio, portal de eventos o catálogo de productos.

---

## Objetivos de Aprendizaje

Al completar esta práctica, el estudiante será capaz de:

- [ ] Diseñar un modelo de contenido coherente con al menos 3 Content Types interrelacionados, aplicando campos, validaciones y referencias correctas
- [ ] Implementar un frontend React funcional que muestre listado, detalle y filtrado de contenido consumido desde la Delivery API de Contentful
- [ ] Aplicar medidas de seguridad completas: variables de entorno, `.gitignore`, separación de API Keys y ausencia total de credenciales hardcodeadas
- [ ] Ejecutar y documentar un checklist técnico de 20 ítems (Modelado, API/Autenticación, Frontend, Seguridad) con resultado PASS/FAIL
- [ ] Producir un `README.md` técnico que describa la arquitectura, instrucciones de configuración y decisiones de diseño del proyecto

---

## Prerrequisitos

### Conocimiento Previo

- Haber completado las Prácticas 1, 2, 3 y 4 del curso
- Comprender la diferencia entre Content Delivery API Key, Content Preview API Key y Content Management API Key
- Familiaridad con React 18 + Vite, hooks (`useState`, `useEffect`) y React Router
- Conocimiento básico de consultas REST o GraphQL contra la API de Contentful
- Manejo de variables de entorno con `dotenv` / Vite (`import.meta.env`)

### Acceso y Recursos

- Cuenta de Contentful activa (Plan Community gratuito)
- Espacio de Contentful creado en la Práctica 1 (o importado con el script de recuperación)
- Content Delivery API Key disponible (obtenida en prácticas anteriores)
- Proyecto React de la Práctica 4 disponible localmente y funcional
- Git inicializado en el proyecto con `.gitignore` configurado

---

## Entorno de Laboratorio

### Hardware Recomendado

| Componente       | Mínimo                        | Recomendado                    |
|------------------|-------------------------------|--------------------------------|
| Procesador       | Intel Core i5 / AMD Ryzen 3   | Intel Core i7 / AMD Ryzen 5    |
| RAM              | 8 GB                          | 16 GB                          |
| Disco libre      | 20 GB                         | 30 GB                          |
| Pantalla         | 1280 × 720                    | 1920 × 1080                    |
| Conexión a red   | 10 Mbps                       | 25 Mbps                        |

### Software Requerido

| Herramienta              | Versión mínima | Verificación                        |
|--------------------------|----------------|-------------------------------------|
| Node.js                  | 18.x LTS       | `node --version`                    |
| npm                      | 9.x            | `npm --version`                     |
| Git                      | 2.40           | `git --version`                     |
| Visual Studio Code       | 1.85           | Verificar desde la UI               |
| Contentful CLI           | 3.x            | `contentful --version`              |
| Vite + React 18          | Vite 4.x       | `cat package.json`                  |
| Contentful SDK JS        | 10.x           | `npm list contentful`               |

### Verificación del Entorno

Ejecuta los siguientes comandos en tu terminal para confirmar que el entorno está listo antes de comenzar:

```bash
# Verificar versiones de herramientas clave
node --version        # Debe mostrar v18.x.x o superior
npm --version         # Debe mostrar 9.x.x o superior
git --version         # Debe mostrar 2.40.x o superior
contentful --version  # Debe mostrar 3.x.x o superior

# Verificar que el proyecto base de la Práctica 4 existe
ls ~/portal-demo      # Ajusta la ruta si tu proyecto está en otro directorio
cd ~/portal-demo && npm list contentful  # Debe mostrar contentful@10.x.x
```

> **⚠️ Si no completaste la Práctica 4:** Ejecuta el script de recuperación antes de continuar:
> ```bash
> contentful space import \
>   --space-id TU_SPACE_ID \
>   --content-file ./recovery-export.json \
>   --management-token TU_MANAGEMENT_TOKEN
> ```
> Solicita el archivo `recovery-export.json` a tu instructor.

---

## Pasos del Laboratorio

> **Estructura de la práctica:**
> - **Fase 1** — Diseño del modelo de contenido (pasos 1–2, ~10 min)
> - **Fase 2** — Configuración en Contentful (pasos 3–4, ~5 min)
> - **Fase 3** — Implementación frontend (pasos 5–8, ~10 min)
> - **Fase 4** — Validación con checklist (paso 9, ~5 min)

---

### Paso 1: Diseñar el Modelo de Contenido en Papel

**Objetivo:** Definir formalmente los tres Content Types del portal, sus campos, tipos de dato, validaciones y relaciones antes de tocar Contentful.

#### Instrucciones

1. **Elige la temática** de tu portal. Las opciones son:
   - Blog tecnológico (`article`, `author`, `category`)
   - Portafolio profesional (`project`, `skill`, `client`)
   - Portal de eventos (`event`, `speaker`, `venue`)
   - Catálogo de productos (`product`, `brand`, `productCategory`)

   > Para este lab usaremos **Blog tecnológico** como referencia. Si eliges otra temática, adapta los nombres de Content Types manteniendo la misma estructura de relaciones.

2. **Documenta el modelo** completando la siguiente tabla en papel, un documento o un comentario en tu código. Copia y adapta según tu temática:

   **Content Type: `article`**

   | Campo         | ID del campo  | Tipo         | Requerido | Localizable | Validación especial                    |
   |---------------|---------------|--------------|-----------|-------------|----------------------------------------|
   | Título        | `title`       | Short Text   | Sí        | Sí          | Longitud máx. 120 caracteres           |
   | Slug          | `slug`        | Short Text   | Sí        | No          | Regex: `^[a-z0-9-]+$`; único          |
   | Resumen       | `summary`     | Short Text   | Sí        | Sí          | Longitud máx. 300 caracteres           |
   | Cuerpo        | `body`        | Rich Text    | Sí        | Sí          | —                                      |
   | Imagen portada| `coverImage`  | Media        | No        | No          | Solo imágenes (JPEG, PNG, WebP)        |
   | Autor         | `author`      | Reference    | Sí        | No          | Solo tipo `author`                     |
   | Categoría     | `category`    | Reference    | Sí        | No          | Solo tipo `category`                   |
   | Fecha pub.    | `publishedAt` | Date         | Sí        | No          | —                                      |
   | Etiquetas     | `tags`        | Array (Text) | No        | No          | —                                      |

   **Content Type: `author`**

   | Campo         | ID del campo | Tipo       | Requerido | Localizable |
   |---------------|--------------|------------|-----------|-------------|
   | Nombre        | `fullName`   | Short Text | Sí        | No          |
   | Biografía     | `bio`        | Long Text  | No        | No          |
   | Avatar        | `avatar`     | Media      | No        | No          |
   | Email         | `email`      | Short Text | No        | No          |

   **Content Type: `category`**

   | Campo         | ID del campo  | Tipo       | Requerido | Localizable |
   |---------------|---------------|------------|-----------|-------------|
   | Nombre        | `name`        | Short Text | Sí        | Sí          |
   | Slug          | `slug`        | Short Text | Sí        | No          |
   | Descripción   | `description` | Long Text  | No        | Sí          |

3. **Dibuja el diagrama de relaciones** entre los tres tipos:

   ```
   ┌─────────────┐        ┌──────────────┐
   │   article   │───────▶│    author    │
   │             │  N:1   └──────────────┘
   │             │
   │             │        ┌──────────────┐
   │             │───────▶│   category   │
   └─────────────┘  N:1   └──────────────┘
   ```

4. **Verifica el modelo** contra este checklist de diseño previo:

   - [ ] Cada Content Type tiene al menos un campo de tipo slug con validación regex
   - [ ] Las relaciones usan campos `Reference` (no texto plano con IDs)
   - [ ] Los campos de texto libre tienen límites de longitud definidos
   - [ ] Ningún campo almacena datos que pertenecen a otro Content Type (sin redundancia)
   - [ ] Se han identificado qué campos necesitan localización

#### Resultado Esperado

Un documento (en papel, Markdown o comentario en código) con las tres tablas completas y el diagrama de relaciones. Este documento servirá como referencia durante toda la implementación.

#### Verificación

No hay verificación técnica en este paso. Confirma que puedes responder estas preguntas antes de continuar:
- ¿Cuántos campos tiene cada Content Type?
- ¿Cuál es la relación entre `article` y `author`? ¿Es 1:1 o N:1?
- ¿Qué validación tiene el campo `slug`?

---

### Paso 2: Preparar el Proyecto Base

**Objetivo:** Asegurar que el proyecto React de la Práctica 4 está en condiciones óptimas para ser extendido, con la estructura de archivos y variables de entorno correctas.

#### Instrucciones

1. **Navega al directorio del proyecto** y verifica su estado:

   ```bash
   cd ~/portal-demo   # Ajusta la ruta a tu proyecto
   git status         # Debe mostrar un repositorio limpio o con cambios conocidos
   npm run dev        # Verifica que el servidor de desarrollo arranca sin errores
   # Presiona Ctrl+C para detener el servidor
   ```

2. **Verifica el archivo `.env`** en la raíz del proyecto. Debe contener las variables de Contentful:

   ```bash
   cat .env
   ```

   El archivo debe tener al menos estas variables (con tus valores reales):

   ```bash
   # .env — NUNCA commitear este archivo
   VITE_CONTENTFUL_SPACE_ID=tu_space_id_aqui
   VITE_CONTENTFUL_DELIVERY_TOKEN=tu_delivery_token_aqui
   VITE_CONTENTFUL_ENVIRONMENT=master
   ```

   > **🔒 SEGURIDAD CRÍTICA:** Si este archivo no existe, créalo ahora. Si ya existe, verifica que el `.gitignore` lo excluye.

3. **Verifica el `.gitignore`** para asegurarte de que `.env` está excluido:

   ```bash
   cat .gitignore | grep ".env"
   # Debe mostrar: .env
   ```

   Si no aparece, agrégalo:

   ```bash
   echo ".env" >> .gitignore
   echo ".env.local" >> .gitignore
   echo ".env.*.local" >> .gitignore
   ```

4. **Verifica que `.env.example` existe** con las variables sin valores reales:

   ```bash
   cat .env.example
   ```

   Si no existe, créalo:

   ```bash
   cat > .env.example << 'EOF'
   # Copia este archivo como .env y completa con tus valores reales
   # NUNCA commitees el archivo .env con valores reales
   VITE_CONTENTFUL_SPACE_ID=
   VITE_CONTENTFUL_DELIVERY_TOKEN=
   VITE_CONTENTFUL_ENVIRONMENT=master
   EOF
   ```

5. **Confirma la estructura del proyecto.** Debe existir al menos:

   ```
   portal-demo/
   ├── .env                    ← con valores reales (NO en Git)
   ├── .env.example            ← sin valores reales (SÍ en Git)
   ├── .gitignore              ← incluye .env
   ├── src/
   │   ├── contentfulClient.js ← cliente configurado con SDK v10
   │   ├── App.jsx
   │   └── components/
   └── package.json
   ```

6. **Verifica que el cliente de Contentful usa la Delivery API** (no la Management API):

   ```bash
   cat src/contentfulClient.js
   ```

   Debe verse similar a:

   ```javascript
   // src/contentfulClient.js
   import { createClient } from 'contentful';

   const client = createClient({
     space: import.meta.env.VITE_CONTENTFUL_SPACE_ID,
     accessToken: import.meta.env.VITE_CONTENTFUL_DELIVERY_TOKEN,
     environment: import.meta.env.VITE_CONTENTFUL_ENVIRONMENT || 'master',
   });

   export default client;
   ```

   > **⚠️ IMPORTANTE:** `accessToken` aquí es la **Delivery API Key** (solo lectura). La Management API Key NUNCA debe aparecer en el frontend.

#### Resultado Esperado

```
✓ Servidor de desarrollo arranca en http://localhost:5173 sin errores
✓ .env existe con las tres variables de Contentful
✓ .gitignore excluye .env
✓ .env.example existe sin valores reales
✓ contentfulClient.js usa createClient con Delivery Token
```

#### Verificación

```bash
# Confirmar que .env NO está trackeado por Git
git ls-files .env
# No debe mostrar ninguna salida (archivo no trackeado)

# Confirmar que .env.example SÍ está trackeado
git ls-files .env.example
# Debe mostrar: .env.example
```

---

### Paso 3: Configurar o Actualizar los Content Types en Contentful

**Objetivo:** Crear o ajustar los tres Content Types en el espacio de Contentful según el modelo diseñado en el Paso 1.

#### Instrucciones

1. **Accede a tu espacio** en [app.contentful.com](https://app.contentful.com) y navega a **Content model**.

2. **Crea o verifica el Content Type `category`** (debe crearse primero porque `article` lo referencia):

   - Si ya existe de prácticas anteriores, verifica que tiene los campos `name`, `slug` y `description`.
   - Si no existe, haz clic en **Add content type**, nombra `Category` con ID `category` y agrega los campos:
     - `name`: Short text, Required ✓, Localizable ✓
     - `slug`: Short text, Required ✓. En **Validation** → agrega regex `^[a-z0-9-]+$` con mensaje "Solo letras minúsculas, números y guiones"
     - `description`: Long text, Required ✗, Localizable ✓
   - Haz clic en **Save** y luego **Publish**.

3. **Crea o verifica el Content Type `author`**:

   - Campos: `fullName` (Short text, Required ✓), `bio` (Long text), `avatar` (Media), `email` (Short text)
   - **Save** → **Publish**.

4. **Crea o verifica el Content Type `article`**:

   - Campos según la tabla del Paso 1. Para los campos de referencia:
     - `author`: Link → Entry → en **Validation** selecciona "Accept only specified entry types" → `author`
     - `category`: Link → Entry → en **Validation** selecciona "Accept only specified entry types" → `category`
     - `coverImage`: Link → Asset → en **Validation** selecciona "Accept only specified MIME type groups" → Images
   - Para el campo `slug`: agrega regex `^[a-z0-9-]+$`
   - **Save** → **Publish**.

5. **Crea las entradas de demostración** (mínimo 5 entradas distribuidas):

   Navega a **Content** → **Add entry**:

   | Tipo       | Cantidad mínima | Ejemplo de datos                                      |
   |------------|-----------------|-------------------------------------------------------|
   | `category` | 2               | "Tecnología" (slug: `tecnologia`), "Tutoriales" (slug: `tutoriales`) |
   | `author`   | 1               | "Ana García", bio breve, email opcional               |
   | `article`  | 3               | Artículos con título, slug, resumen, body en Rich Text, autor y categoría asignados |

   > **⚠️ Rich Text:** El campo `body` retorna un árbol JSON, NO HTML. En el frontend deberás usar `@contentful/rich-text-react-renderer` para renderizarlo. Si lo instalaste en la Práctica 4, ya está disponible.

6. **Publica todas las entradas** haciendo clic en **Publish** en cada una. Las entradas en estado "Draft" no son visibles a través de la Delivery API.

#### Resultado Esperado

En la sección **Content** del espacio de Contentful deben aparecer al menos 6 entradas publicadas (2 categorías + 1 autor + 3 artículos), todas con estado verde "Published".

#### Verificación

Verifica vía la Delivery API directamente en el navegador:

```
https://cdn.contentful.com/spaces/TU_SPACE_ID/entries?content_type=article&access_token=TU_DELIVERY_TOKEN
```

Reemplaza `TU_SPACE_ID` y `TU_DELIVERY_TOKEN` con tus valores. Debes ver un JSON con `total: 3` (o el número de artículos que hayas creado) y los campos de cada artículo.

---

### Paso 4: Instalar Dependencias Adicionales si es Necesario

**Objetivo:** Asegurar que todas las dependencias del frontend están instaladas, incluyendo el renderer de Rich Text si no se instaló en prácticas anteriores.

#### Instrucciones

1. **Verifica las dependencias actuales** del proyecto:

   ```bash
   cd ~/portal-demo
   cat package.json | grep -A 20 '"dependencies"'
   ```

2. **Instala el renderer de Rich Text** si no está presente:

   ```bash
   # Verificar si ya está instalado
   npm list @contentful/rich-text-react-renderer 2>/dev/null

   # Si no está instalado, instalarlo ahora
   npm install @contentful/rich-text-react-renderer @contentful/rich-text-types
   ```

3. **Instala React Router** si no está presente (necesario para navegación entre listado y detalle):

   ```bash
   npm list react-router-dom 2>/dev/null
   # Si no está instalado:
   npm install react-router-dom@6
   ```

4. **Confirma que todas las dependencias están instaladas** sin errores:

   ```bash
   npm install   # Instala cualquier dependencia faltante del package.json
   npm run dev   # Verifica que el proyecto sigue arrancando correctamente
   # Presiona Ctrl+C
   ```

#### Resultado Esperado

```bash
# package.json debe incluir al menos:
"dependencies": {
  "contentful": "^10.x.x",
  "@contentful/rich-text-react-renderer": "^15.x.x",
  "@contentful/rich-text-types": "^16.x.x",
  "react": "^18.x.x",
  "react-dom": "^18.x.x",
  "react-router-dom": "^6.x.x"
}
```

---

### Paso 5: Implementar el Servicio de Datos de Contentful

**Objetivo:** Crear o actualizar un módulo de servicio centralizado que encapsule todas las llamadas a la API de Contentful.

#### Instrucciones

1. **Crea el archivo `src/services/contentfulService.js`**:

   ```bash
   mkdir -p src/services
   touch src/services/contentfulService.js
   ```

2. **Implementa el servicio** con las funciones necesarias para el portal:

   ```javascript
   // src/services/contentfulService.js
   // Servicio centralizado para todas las consultas a Contentful
   // Usa EXCLUSIVAMENTE la Delivery API (solo lectura)

   import client from '../contentfulClient.js';

   /**
    * Obtiene todos los artículos publicados, ordenados por fecha descendente.
    * Incluye referencias de autor y categoría resueltas (include: 2).
    */
   export async function getArticles({ categorySlug = null, limit = 10, skip = 0 } = {}) {
     const query = {
       content_type: 'article',
       order: '-fields.publishedAt',
       include: 2,
       limit,
       skip,
     };

     // Filtro opcional por categoría
     if (categorySlug) {
       query['fields.category.fields.slug'] = categorySlug;
       query['fields.category.sys.contentType.sys.id'] = 'category';
     }

     const response = await client.getEntries(query);
     return {
       articles: response.items,
       total: response.total,
       limit: response.limit,
       skip: response.skip,
     };
   }

   /**
    * Obtiene un artículo específico por su slug.
    * Retorna null si no se encuentra.
    */
   export async function getArticleBySlug(slug) {
     const response = await client.getEntries({
       content_type: 'article',
       'fields.slug': slug,
       include: 2,
       limit: 1,
     });

     return response.items.length > 0 ? response.items[0] : null;
   }

   /**
    * Obtiene todas las categorías disponibles.
    */
   export async function getCategories() {
     const response = await client.getEntries({
       content_type: 'category',
       order: 'fields.name',
     });
     return response.items;
   }
   ```

   > **Nota sobre `include: 2`:** Este parámetro le indica a la API que resuelva las referencias anidadas hasta 2 niveles de profundidad. Así, al obtener un artículo, los campos `author` y `category` ya vendrán con todos sus datos, sin necesidad de hacer llamadas adicionales.

#### Resultado Esperado

El archivo `src/services/contentfulService.js` existe con las tres funciones exportadas. No hay errores de sintaxis.

#### Verificación

```bash
# Verificar que el archivo existe y tiene contenido
wc -l src/services/contentfulService.js
# Debe mostrar aproximadamente 50 líneas

# Verificar que no hay credenciales hardcodeadas en el servicio
grep -n "CFPAT\|spaceId\|accessToken" src/services/contentfulService.js
# No debe mostrar ningún resultado (las credenciales están en .env)
```

---

### Paso 6: Implementar las Páginas del Portal

**Objetivo:** Crear o actualizar los componentes React para mostrar el listado de artículos, el filtrado por categoría y el detalle de un artículo.

#### Instrucciones

1. **Actualiza `src/App.jsx`** para configurar el enrutamiento:

   ```jsx
   // src/App.jsx
   import { BrowserRouter, Routes, Route } from 'react-router-dom';
   import ArticleList from './pages/ArticleList';
   import ArticleDetail from './pages/ArticleDetail';
   import './App.css';

   function App() {
     return (
       <BrowserRouter>
         <div className="app">
           <header>
             <h1>Portal Demo — Contentful</h1>
           </header>
           <main>
             <Routes>
               <Route path="/" element={<ArticleList />} />
               <Route path="/article/:slug" element={<ArticleDetail />} />
             </Routes>
           </main>
         </div>
       </BrowserRouter>
     );
   }

   export default App;
   ```

2. **Crea el directorio de páginas** y el componente de listado:

   ```bash
   mkdir -p src/pages
   ```

   ```jsx
   // src/pages/ArticleList.jsx
   import { useState, useEffect } from 'react';
   import { Link } from 'react-router-dom';
   import { getArticles, getCategories } from '../services/contentfulService';

   function ArticleList() {
     const [articles, setArticles] = useState([]);
     const [categories, setCategories] = useState([]);
     const [selectedCategory, setSelectedCategory] = useState(null);
     const [loading, setLoading] = useState(true);
     const [error, setError] = useState(null);

     // Cargar categorías al montar el componente
     useEffect(() => {
       getCategories()
         .then(setCategories)
         .catch((err) => console.error('Error cargando categorías:', err));
     }, []);

     // Cargar artículos cuando cambia la categoría seleccionada
     useEffect(() => {
       setLoading(true);
       setError(null);
       getArticles({ categorySlug: selectedCategory })
         .then(({ articles }) => {
           setArticles(articles);
           setLoading(false);
         })
         .catch((err) => {
           setError('Error al cargar los artículos. Intenta de nuevo.');
           setLoading(false);
           console.error(err);
         });
     }, [selectedCategory]);

     if (loading) return <p>Cargando artículos...</p>;
     if (error) return <p style={{ color: 'red' }}>{error}</p>;

     return (
       <div>
         {/* Filtro por categoría */}
         <nav aria-label="Filtro por categoría">
           <button
             onClick={() => setSelectedCategory(null)}
             style={{ fontWeight: !selectedCategory ? 'bold' : 'normal' }}
           >
             Todas
           </button>
           {categories.map((cat) => (
             <button
               key={cat.sys.id}
               onClick={() => setSelectedCategory(cat.fields.slug)}
               style={{
                 fontWeight: selectedCategory === cat.fields.slug ? 'bold' : 'normal',
               }}
             >
               {cat.fields.name}
             </button>
           ))}
         </nav>

         {/* Listado de artículos */}
         {articles.length === 0 ? (
           <p>No hay artículos en esta categoría.</p>
         ) : (
           <ul>
             {articles.map((article) => (
               <li key={article.sys.id}>
                 <Link to={`/article/${article.fields.slug}`}>
                   <h2>{article.fields.title}</h2>
                 </Link>
                 <p>{article.fields.summary}</p>
                 <small>
                   Por: {article.fields.author?.fields?.fullName || 'Autor desconocido'} |{' '}
                   {new Date(article.fields.publishedAt).toLocaleDateString('es-ES')}
                 </small>
               </li>
             ))}
           </ul>
         )}
       </div>
     );
   }

   export default ArticleList;
   ```

3. **Crea el componente de detalle de artículo**:

   ```jsx
   // src/pages/ArticleDetail.jsx
   import { useState, useEffect } from 'react';
   import { useParams, Link } from 'react-router-dom';
   import { documentToReactComponents } from '@contentful/rich-text-react-renderer';
   import { getArticleBySlug } from '../services/contentfulService';

   function ArticleDetail() {
     const { slug } = useParams();
     const [article, setArticle] = useState(null);
     const [loading, setLoading] = useState(true);
     const [error, setError] = useState(null);

     useEffect(() => {
       setLoading(true);
       getArticleBySlug(slug)
         .then((data) => {
           if (!data) {
             setError('Artículo no encontrado.');
           } else {
             setArticle(data);
           }
           setLoading(false);
         })
         .catch((err) => {
           setError('Error al cargar el artículo.');
           setLoading(false);
           console.error(err);
         });
     }, [slug]);

     if (loading) return <p>Cargando artículo...</p>;
     if (error) return <p style={{ color: 'red' }}>{error}</p>;

     const { title, summary, body, author, category, publishedAt, coverImage } = article.fields;

     return (
       <article>
         <Link to="/">← Volver al listado</Link>

         <h1>{title}</h1>
         <p><em>{summary}</em></p>

         {/* Metadatos */}
         <div>
           <span>Autor: {author?.fields?.fullName}</span> |{' '}
           <span>Categoría: {category?.fields?.name}</span> |{' '}
           <span>Publicado: {new Date(publishedAt).toLocaleDateString('es-ES')}</span>
         </div>

         {/* Imagen de portada */}
         {coverImage?.fields?.file?.url && (
           <img
             src={`https:${coverImage.fields.file.url}`}
             alt={coverImage.fields.title || title}
             style={{ maxWidth: '100%' }}
           />
         )}

         {/* Cuerpo en Rich Text — USA documentToReactComponents, NO innerHTML */}
         <div className="article-body">
           {documentToReactComponents(body)}
         </div>
       </article>
     );
   }

   export default ArticleDetail;
   ```

   > **⚠️ Rich Text:** El campo `body` es un árbol JSON (Document), NO HTML. `documentToReactComponents(body)` lo convierte a componentes React correctamente. Usar `dangerouslySetInnerHTML` con este valor causaría que se renderice `[object Object]`.

4. **Inicia el servidor de desarrollo** y verifica que las páginas cargan:

   ```bash
   npm run dev
   # Abre http://localhost:5173 en el navegador
   ```

#### Resultado Esperado

- La página principal (`/`) muestra los artículos con botones de filtro por categoría
- Al hacer clic en una categoría, la lista se actualiza mostrando solo artículos de esa categoría
- Al hacer clic en un artículo, navega a `/article/[slug]` y muestra el detalle completo
- El cuerpo del artículo se renderiza correctamente (párrafos, negritas, listas si las hay)
- El enlace "← Volver al listado" funciona correctamente

---

### Paso 7: Manejo de Errores y Estados de Carga

**Objetivo:** Verificar que el frontend maneja correctamente los escenarios de error y estados de carga, evitando pantallas en blanco o errores no controlados.

#### Instrucciones

1. **Prueba el manejo de errores** desconectando temporalmente la conexión a la API. Modifica momentáneamente el token en `.env` a un valor inválido:

   ```bash
   # TEMPORAL — Solo para probar, revertir inmediatamente
   # En .env, cambia el token por un valor inválido:
   # VITE_CONTENTFUL_DELIVERY_TOKEN=token_invalido_para_prueba
   ```

2. **Recarga el servidor** (`npm run dev`) y verifica en el navegador que aparece el mensaje de error definido en los componentes ("Error al cargar los artículos. Intenta de nuevo."), NO una pantalla en blanco o un error de JavaScript sin capturar.

3. **Revierte el token** a su valor correcto en `.env` y verifica que la aplicación vuelve a funcionar:

   ```bash
   # Restaurar el token correcto en .env
   # Reiniciar el servidor de desarrollo si es necesario
   npm run dev
   ```

4. **Prueba la ruta de artículo no encontrado** navegando manualmente a una URL con slug inexistente:

   ```
   http://localhost:5173/article/slug-que-no-existe
   ```

   Debe mostrar el mensaje "Artículo no encontrado." sin errores en consola.

#### Resultado Esperado

- Con token inválido: mensaje de error visible en la UI, sin excepciones no capturadas en consola
- Con slug inexistente: mensaje "Artículo no encontrado." visible
- Con token correcto: aplicación funciona normalmente

---

### Paso 8: Crear el README Técnico del Proyecto

**Objetivo:** Documentar el proyecto con un README técnico que cualquier desarrollador pueda seguir para configurar y ejecutar el portal.

#### Instrucciones

1. **Crea o actualiza el archivo `README.md`** en la raíz del proyecto:

   ```bash
   cat > README.md << 'READMEEOF'
   # Portal Demo — Contentful End-to-End

   Portal digital demo construido con React 18 + Vite y Contentful como CMS headless.

   ## Arquitectura

   ```
   Contentful (CMS Headless)
        │
        │ Delivery API (REST, solo lectura)
        ▼
   contentfulClient.js (SDK v10)
        │
        ▼
   contentfulService.js (capa de servicio)
        │
        ▼
   Componentes React (ArticleList, ArticleDetail)
   ```

   ## Content Types

   | Content Type | Descripción               | Campos clave                          |
   |--------------|---------------------------|---------------------------------------|
   | `article`    | Artículo del portal       | title, slug, body (Rich Text), author, category |
   | `author`     | Autor de artículos        | fullName, bio, avatar                 |
   | `category`   | Categoría de artículos    | name, slug, description               |

   ## Configuración

   1. Clona el repositorio
   2. Instala dependencias: `npm install`
   3. Copia `.env.example` a `.env`: `cp .env.example .env`
   4. Completa las variables en `.env` con tus credenciales de Contentful
   5. Inicia el servidor: `npm run dev`

   ## Variables de Entorno

   | Variable                          | Descripción                        |
   |-----------------------------------|------------------------------------|
   | `VITE_CONTENTFUL_SPACE_ID`        | ID del espacio de Contentful       |
   | `VITE_CONTENTFUL_DELIVERY_TOKEN`  | Delivery API Key (solo lectura)    |
   | `VITE_CONTENTFUL_ENVIRONMENT`     | Entorno (default: `master`)        |

   ## Seguridad

   - Las API Keys NUNCA están hardcodeadas en el código fuente
   - Se usa exclusivamente la **Delivery API Key** en el frontend (solo lectura)
   - La **Management API Key** no se usa en este proyecto
   - `.env` está excluido del repositorio vía `.gitignore`

   ## Decisiones de Diseño

   - **Rich Text**: Se usa `@contentful/rich-text-react-renderer` para convertir el árbol JSON del campo `body` a componentes React
   - **Referencias con `include: 2`**: Las consultas incluyen datos de autor y categoría en una sola llamada API, evitando N+1 requests
   - **Filtrado en cliente vs. servidor**: El filtrado por categoría se realiza en el servidor (parámetro de query a la Delivery API), no en el cliente, para evitar cargar datos innecesarios

   READMEEOF
   ```

2. **Confirma que el README se creó correctamente**:

   ```bash
   wc -l README.md   # Debe mostrar al menos 50 líneas
   cat README.md     # Revisa el contenido visualmente
   ```

#### Resultado Esperado

`README.md` en la raíz del proyecto con secciones de arquitectura, Content Types, configuración, variables de entorno, seguridad y decisiones de diseño.

---

### Paso 9: Ejecutar el Checklist Técnico de Validación

**Objetivo:** Evaluar formalmente el proyecto completo contra 20 criterios técnicos agrupados en cuatro categorías, documentando cada resultado como PASS o FAIL.

#### Instrucciones

1. **Crea el archivo de resultados del checklist**:

   ```bash
   touch CHECKLIST_RESULTS.md
   ```

2. **Evalúa cada ítem** y completa la tabla. A continuación se presenta el checklist completo con instrucciones de verificación para cada ítem:

   ```markdown
   # Checklist Técnico de Validación — Portal Demo Contentful
   
   Fecha: [COMPLETAR]
   Estudiante: [COMPLETAR]
   
   ## Sección A: Modelado de Contenido (5 ítems)
   
   | # | Criterio | Verificación | Resultado | Notas |
   |---|----------|--------------|-----------|-------|
   | A1 | El espacio tiene al menos 3 Content Types creados y publicados | Verificar en Contentful → Content model | PASS/FAIL | |
   | A2 | Cada Content Type tiene al menos un campo marcado como Required | Verificar en la definición de cada Content Type | PASS/FAIL | |
   | A3 | Los campos `slug` tienen validación regex `^[a-z0-9-]+$` | Verificar en Settings del campo slug | PASS/FAIL | |
   | A4 | Las relaciones entre Content Types usan campos Reference (no texto plano) | Verificar que `author` y `category` en `article` son tipo Link | PASS/FAIL | |
   | A5 | Hay mínimo 5 entradas publicadas distribuidas entre los Content Types | Content → verificar estado Published en todas | PASS/FAIL | |
   
   ## Sección B: API y Autenticación (5 ítems)
   
   | # | Criterio | Verificación | Resultado | Notas |
   |---|----------|--------------|-----------|-------|
   | B1 | El frontend usa la Delivery API Key (no la Management Key) | Revisar contentfulClient.js y comparar token con el de la UI de Contentful | PASS/FAIL | |
   | B2 | La Delivery API responde con los artículos publicados | Llamada directa: `https://cdn.contentful.com/spaces/{id}/entries?content_type=article&access_token={token}` | PASS/FAIL | |
   | B3 | Las consultas incluyen `include: 2` para resolver referencias en una sola llamada | Revisar contentfulService.js | PASS/FAIL | |
   | B4 | El filtrado por categoría se realiza como query parameter a la API (no en cliente) | Revisar la función `getArticles` con parámetro `categorySlug` | PASS/FAIL | |
   | B5 | No hay llamadas a la Management API desde el frontend | Buscar `contentful-management` en package.json y en el código fuente | PASS/FAIL | |
   
   ## Sección C: Frontend (5 ítems)
   
   | # | Criterio | Verificación | Resultado | Notas |
   |---|----------|--------------|-----------|-------|
   | C1 | La página de listado muestra al menos 3 artículos con título, resumen y autor | Verificar en http://localhost:5173 | PASS/FAIL | |
   | C2 | El filtro por categoría actualiza la lista correctamente | Hacer clic en cada botón de categoría y verificar que la lista cambia | PASS/FAIL | |
   | C3 | La página de detalle muestra el cuerpo del artículo renderizado (no [object Object]) | Navegar a /article/[slug] y verificar que el body se muestra como texto | PASS/FAIL | |
   | C4 | Los estados de carga y error se muestran correctamente en la UI | Probar con token inválido y slug inexistente | PASS/FAIL | |
   | C5 | La navegación listado ↔ detalle funciona correctamente con React Router | Verificar que el botón "Volver" y los enlaces de artículo funcionan | PASS/FAIL | |
   
   ## Sección D: Seguridad (5 ítems)
   
   | # | Criterio | Verificación | Resultado | Notas |
   |---|----------|--------------|-----------|-------|
   | D1 | El archivo `.env` NO está trackeado por Git | Ejecutar: `git ls-files .env` → no debe mostrar output | PASS/FAIL | |
   | D2 | El archivo `.env.example` SÍ está trackeado y no contiene valores reales | `git ls-files .env.example` → debe aparecer; `cat .env.example` → sin tokens reales | PASS/FAIL | |
   | D3 | No hay credenciales hardcodeadas en ningún archivo de código fuente | `grep -r "CFPAT\|spaceId.*=.*[a-z0-9]" src/` → sin resultados con tokens reales | PASS/FAIL | |
   | D4 | El `.gitignore` incluye `.env`, `.env.local` y variantes | `cat .gitignore | grep env` → debe mostrar las entradas | PASS/FAIL | |
   | D5 | El README documenta la separación de API Keys y el procedimiento de configuración segura | Revisar README.md sección Seguridad | PASS/FAIL | |
   
   ## Resumen de Resultados
   
   | Sección | PASS | FAIL | % Completado |
   |---------|------|------|--------------|
   | A: Modelado | /5 | /5 | % |
   | B: API y Auth | /5 | /5 | % |
   | C: Frontend | /5 | /5 | % |
   | D: Seguridad | /5 | /5 | % |
   | **Total** | **/20** | **/20** | **%** |
   
   ## Plan de Acción para ítems FAIL
   
   [Documentar los pasos para resolver cada ítem que no pasó]
   ```

3. **Ejecuta las verificaciones técnicas automatizables**:

   ```bash
   echo "=== Verificación D1: .env no trackeado ==="
   git ls-files .env && echo "FAIL: .env está trackeado" || echo "PASS: .env no está trackeado"

   echo "=== Verificación D2: .env.example trackeado ==="
   git ls-files .env.example | grep -q ".env.example" && echo "PASS" || echo "FAIL: .env.example no está trackeado"

   echo "=== Verificación D3: Sin credenciales hardcodeadas ==="
   # Busca patrones de tokens de Contentful (empiezan con letras y tienen guiones)
   grep -rn "accessToken\s*=\s*['\"][a-zA-Z0-9_-]\{10,\}" src/ && echo "FAIL: Posibles credenciales hardcodeadas" || echo "PASS: Sin credenciales hardcodeadas detectadas"

   echo "=== Verificación D4: .gitignore completo ==="
   grep -c "\.env" .gitignore | xargs -I{} test {} -ge 1 && echo "PASS" || echo "FAIL: .gitignore incompleto"

   echo "=== Verificación B5: Sin contentful-management en frontend ==="
   grep -r "contentful-management" src/ && echo "FAIL: Management API en frontend" || echo "PASS: Sin Management API en frontend"
   ```

4. **Completa el archivo `CHECKLIST_RESULTS.md`** con los resultados de cada ítem, incluyendo notas para los ítems FAIL.

5. **Objetivo de aprobación:** El proyecto se considera completo con un mínimo de **18/20 ítems en PASS**. Los ítems de la Sección D (Seguridad) son **obligatorios todos en PASS** sin excepción.

#### Resultado Esperado

`CHECKLIST_RESULTS.md` completado con los 20 ítems evaluados. Mínimo 18 PASS, con todos los ítems de seguridad (D1–D5) en PASS.

---

## Validación y Pruebas

### Prueba Integral del Portal

Ejecuta esta secuencia de pruebas funcionales completas antes de considerar la práctica terminada:

```bash
# 1. Iniciar el servidor de desarrollo
cd ~/portal-demo
npm run dev

# 2. Abrir http://localhost:5173 y verificar:
#    ✓ La página carga sin errores en la consola del navegador
#    ✓ Se muestran los artículos con título, resumen y datos de autor
#    ✓ Los botones de categoría están visibles

# 3. Probar el filtrado:
#    ✓ Hacer clic en una categoría → la lista se actualiza
#    ✓ Hacer clic en "Todas" → vuelven todos los artículos

# 4. Probar la navegación al detalle:
#    ✓ Hacer clic en un artículo → navega a /article/[slug]
#    ✓ El cuerpo del artículo se renderiza como texto (no JSON)
#    ✓ El botón "← Volver" regresa al listado

# 5. Verificar en la consola del navegador (F12):
#    ✓ No hay errores en rojo
#    ✓ No hay warnings de React sobre keys faltantes
#    ✓ Las llamadas a la API en la pestaña Network muestran status 200
```

### Verificación de Seguridad Final

```bash
# Verificación completa de seguridad antes del commit final
echo "--- AUDITORÍA DE SEGURIDAD ---"

echo "[1] Archivos trackeados por Git:"
git ls-files | grep -E "\.env$|\.env\.local$"
# No debe mostrar nada

echo "[2] Variables de entorno en código fuente:"
grep -rn "import\.meta\.env\." src/ | grep -v "VITE_"
# No debe mostrar nada (todas las variables deben usar el prefijo VITE_)

echo "[3] Tokens en código fuente:"
grep -rn "[a-zA-Z0-9_-]\{40,\}" src/
# No debe mostrar tokens reales (40+ caracteres alfanuméricos)

echo "[4] Uso correcto del SDK:"
grep -n "createClient" src/contentfulClient.js
# Debe mostrar una sola instancia de createClient

echo "--- FIN DE AUDITORÍA ---"
```

---

## Solución de Problemas

### Problema 1: Los artículos se muestran pero el campo `author` o `category` aparece como `undefined`

**Síntomas:** En la página de listado o detalle, el nombre del autor aparece como "Autor desconocido" o la categoría no se muestra, a pesar de que las entradas en Contentful tienen autor y categoría asignados.

**Causa:** La consulta a la API no incluye el parámetro `include` con valor suficiente para resolver las referencias. Sin `include: 2`, la API retorna solo el `sys.id` de las referencias, no los campos del entry referenciado. Esto es un comportamiento por defecto de la Delivery API: las referencias no se resuelven automáticamente.

**Solución:**

```javascript
// ❌ INCORRECTO — No resuelve referencias
const response = await client.getEntries({
  content_type: 'article',
});
// article.fields.author será { sys: { id: '...', type: 'Link' } }

// ✅ CORRECTO — Resuelve referencias hasta 2 niveles
const response = await client.getEntries({
  content_type: 'article',
  include: 2,  // ← Este parámetro es clave
});
// article.fields.author será { sys: {...}, fields: { fullName: 'Ana García', ... } }
```

Verifica que `contentfulService.js` incluye `include: 2` en todas las consultas que necesitan datos de referencias. Reinicia el servidor de desarrollo después de hacer el cambio.

---

### Problema 2: El cuerpo del artículo se muestra como `[object Object]` o como texto JSON en bruto

**Síntomas:** En la página de detalle del artículo, el campo `body` se muestra como `[object Object]` o como una cadena JSON con la estructura del árbol de nodos, en lugar del texto formateado del artículo.

**Causa:** El campo `body` de tipo Rich Text en Contentful NO retorna HTML. Retorna un árbol de nodos JSON con estructura `{ nodeType: 'document', content: [...] }`. Si se intenta renderizar este objeto directamente como `{article.fields.body}` en JSX, React lo convierte a `[object Object]`. Si se usa `JSON.stringify`, se muestra el JSON en bruto.

**Solución:**

```jsx
// ❌ INCORRECTO — Renderiza [object Object]
<div>{article.fields.body}</div>

// ❌ INCORRECTO — Renderiza JSON en bruto
<div>{JSON.stringify(article.fields.body)}</div>

// ✅ CORRECTO — Usa el renderer oficial
import { documentToReactComponents } from '@contentful/rich-text-react-renderer';

<div className="article-body">
  {documentToReactComponents(article.fields.body)}
</div>
```

Si el paquete no está instalado:

```bash
npm install @contentful/rich-text-react-renderer @contentful/rich-text-types
```

Verifica que el import está en la parte superior del componente `ArticleDetail.jsx` y que se llama la función `documentToReactComponents` pasando `article.fields.body` como argumento.

---

## Limpieza

Al finalizar la práctica, ejecuta los siguientes pasos para dejar el entorno en orden:

```bash
# 1. Detener el servidor de desarrollo (si está corriendo)
# Presiona Ctrl+C en la terminal donde corre npm run dev

# 2. Hacer commit del proyecto completado
cd ~/portal-demo
git add .
git status
# Verifica que .env NO aparece en los archivos a commitear

# Si .env no aparece (correcto), procede con el commit:
git commit -m "feat: portal demo end-to-end completado - Práctica 5

- Modelo de contenido: article, author, category
- Frontend: listado, filtrado por categoría, detalle de artículo
- Seguridad: variables de entorno, sin credenciales hardcodeadas
- Checklist técnico: 20 ítems validados"

# 3. Verificar el historial de commits
git log --oneline -5

# 4. Verificar que .env no está en ningún commit
git log --all --full-history -- .env
# No debe mostrar ningún resultado
```

> **⚠️ IMPORTANTE:** Si accidentalmente commiteaste el archivo `.env` con credenciales reales en algún momento del curso, debes **rotar (regenerar) las API Keys** en Contentful inmediatamente. Ve a Settings → API Keys → selecciona la key → regenera el token. Las credenciales expuestas en Git deben considerarse comprometidas aunque el repositorio sea privado.

---

## Resumen

### Logros de la Práctica

En esta práctica integradora construiste un **portal digital demo funcional end-to-end** que demuestra el dominio completo del ciclo de desarrollo con Contentful como CMS headless:

| Fase | Actividad | Resultado |
|------|-----------|-----------|
| **Diseño** | Modelo de 3 Content Types con campos, validaciones y relaciones | Documentado en tabla y diagrama |
| **Configuración** | Content Types y entradas publicadas en Contentful | Mínimo 5 entradas publicadas |
| **Servicio** | Capa de servicio con funciones para artículos, detalle y categorías | `contentfulService.js` con `include: 2` |
| **Frontend** | Listado, filtrado por categoría y detalle con Rich Text | 3 rutas funcionales en React Router |
| **Seguridad** | `.env`, `.gitignore`, sin credenciales hardcodeadas | Todos los ítems D en PASS |
| **Documentación** | README técnico y checklist de 20 ítems | `README.md` + `CHECKLIST_RESULTS.md` |

### Conceptos Clave Consolidados

- **Modelo de contenido como plano arquitectónico:** Las decisiones de modelado (tipos de campo, relaciones, validaciones) impactan directamente en la complejidad de las consultas y en la calidad del frontend.
- **`include: 2` es fundamental:** Sin este parámetro, las referencias entre Content Types no se resuelven y el frontend recibe solo IDs en lugar de datos completos.
- **Rich Text requiere un renderer específico:** `documentToReactComponents` es la forma correcta de convertir el árbol JSON de Rich Text a componentes React. Este comportamiento es contraintuitivo pero está documentado.
- **Separación de API Keys:** La Delivery API Key (solo lectura) es la única que debe usarse en el frontend. La Management API Key nunca debe exponerse en el cliente.
- **El checklist técnico como práctica profesional:** Validar formalmente el proyecto contra criterios de modelado, API, frontend y seguridad antes de considerar el trabajo terminado es una práctica de calidad que debe aplicarse en proyectos reales.

### Recursos Adicionales

- [Documentación oficial del modelo de datos de Contentful](https://www.contentful.com/developers/docs/concepts/data-model/)
- [Guía de la Delivery API REST de Contentful](https://www.contentful.com/developers/docs/references/content-delivery-api/)
- [Documentación de `@contentful/rich-text-react-renderer`](https://github.com/contentful/rich-text/tree/master/packages/rich-text-react-renderer)
- [Mejores prácticas de seguridad con API Keys en frontend](https://www.contentful.com/developers/docs/references/authentication/)
- [Guía de React Router v6](https://reactrouter.com/en/main/start/tutorial)
- [Parámetro `include` en la Delivery API](https://www.contentful.com/developers/docs/references/content-delivery-api/#/reference/links)

---
LAB_END---
