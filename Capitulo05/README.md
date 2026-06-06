# Proyecto demo funcional end-to-end

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

Esta práctica integradora consolida todos los aprendizajes del curso completando el **portal de noticias demo funcional end-to-end** iniciado en las prácticas anteriores. El estudiante validará el modelo canónico de tres Content Types interrelacionados, continuará el frontend React que consume la API, aplicará las medidas de seguridad aprendidas y ejecutará un checklist técnico formal de 20 ítems antes de considerar el proyecto terminado.

---

### Escenario de la práctica

El portal de noticias está listo para una revisión técnica final. Debes consolidar lo construido, comprobar su funcionamiento de extremo a extremo y documentar evidencias suficientes para entregar el proyecto.

### Objetivo de la práctica

Validar y completar el proyecto existente `portal-noticias` usando el modelo canónico, la estructura acordada y un checklist técnico que permita identificar y resolver pendientes antes de la entrega.

### Cómo trabajar esta práctica

1. Continúa desde el proyecto `portal-noticias` creado en la Práctica 4.
2. No rediseñes Content Types, API IDs ni rutas: valida el contrato técnico existente.
3. Registra cada comprobación como `PASS` o `FAIL` y resuelve los fallos antes de cerrar.
4. Conserva evidencias de la validación sin incluir credenciales reales.

> **Importante:** Esta práctica depende del estado final de las prácticas anteriores; no utiliza archivos de recuperación ni crea un proyecto frontend alternativo.

---

## Objetivos de Aprendizaje

Al completar esta práctica, el estudiante será capaz de:

- [ ] Validar el modelo canónico de 3 Content Types interrelacionados, comprobando campos, validaciones y referencias correctas
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
- Espacio de Contentful creado y conservado desde la Práctica 1
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

> **Compatibilidad de terminal:** Los comandos de Node.js, npm y Git son portables. En PowerShell utiliza `Test-Path`, `Get-Content` y `Select-String` en lugar de `ls`, `cat` y `grep` cuando aparezcan en ejemplos Bash.

```bash
# Verificar versiones de herramientas clave
node --version        # Debe mostrar v18.x.x o superior
npm --version         # Debe mostrar 9.x.x o superior
git --version         # Debe mostrar 2.40.x o superior
contentful --version  # Debe mostrar 3.x.x o superior

# Verificar que el proyecto base de la Práctica 4 existe
node -e "console.log('portal-noticias existe:',require('fs').existsSync('portal-noticias'))"
cd portal-noticias
npm list contentful
```

> **Prerrequisito obligatorio:** Completa las Prácticas 1 a 4 antes de continuar. Este proyecto final continúa sobre `portal-noticias` y sobre el modelo canónico existente. En esta fase no se proporciona un archivo de recuperación alternativo.

---

## Pasos del Laboratorio

> **Estructura de la práctica:**
> - **Fase 1** — Diseño del modelo de contenido (pasos 1–2, ~10 min)
> - **Fase 2** — Configuración en Contentful (pasos 3–4, ~5 min)
> - **Fase 3** — Implementación frontend (pasos 5–8, ~10 min)
> - **Fase 4** — Validación con checklist (paso 9, ~5 min)

---

### Paso 1: Revisar el Modelo de Contenido

**Objetivo:** Definir formalmente los tres Content Types del portal, sus campos, tipos de dato, validaciones y relaciones antes de tocar Contentful.

#### Instrucciones

1. **Usa el portal de noticias del curso** con los Content Types canónicos `article`, `author` y `category`. No cambies los API IDs ni los Field IDs, porque el proyecto continúa directamente desde las prácticas anteriores.

2. **Documenta y valida el modelo canónico** completando la siguiente tabla en papel, un documento o un comentario en tu código. No cambies los API IDs ni los Field IDs:

   **Content Type: `article`**

   | Campo         | ID del campo  | Tipo         | Requerido | Localizable | Validación especial                    |
   |---------------|---------------|--------------|-----------|-------------|----------------------------------------|
   | Título        | `title`       | Short Text   | Sí        | Sí          | Longitud máx. 120 caracteres           |
   | Slug          | `slug`        | Short Text   | Sí        | No          | Regex: `^[a-z0-9-]+$`; único          |
   | Cuerpo        | `body`        | Rich Text    | Sí        | Sí          | —                                      |
   | Imagen portada| `coverImage`  | Media        | No        | No          | Solo imágenes (JPEG, PNG, WebP)        |
   | Autor         | `author`      | Reference    | Sí        | No          | Solo tipo `author`                     |
   | Categoría     | `category`    | Reference    | Sí        | No          | Solo tipo `category`                   |
   | Fecha pub.    | `publishedAt` | Date         | Sí        | No          | —                                      |

   **Content Type: `author`**

   | Campo         | ID del campo | Tipo       | Requerido | Localizable |
   |---------------|--------------|------------|-----------|-------------|
   | Nombre        | `name`   | Short Text | Sí        | No          |
   | Slug          | `slug`       | Short Text | Sí        | No          |
   | Biografía     | `bio`        | Long Text  | No        | No          |
   | Avatar        | `avatar`     | Media      | No        | No          |

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
   cd ~/portal-noticias   # Ajusta la ruta a tu proyecto
   git status         # Debe mostrar un repositorio limpio o con cambios conocidos
   npm run dev        # Verifica que el servidor de desarrollo arranca sin errores
   # Presiona Ctrl+C para detener el servidor
   ```

2. **Verifica el archivo `.env`** en la raíz del proyecto. Debe contener las variables de Contentful:

   ```bash
   node -e "const fs=require('fs'); console.log('.env existe:',fs.existsSync('.env'),'tamaño:',fs.existsSync('.env')?fs.statSync('.env').size:0)"
   ```

   El archivo debe tener al menos estas variables (con tus valores reales):

   ```bash
   # .env — NUNCA commitear este archivo
   VITE_CONTENTFUL_SPACE_ID=<CONTENTFUL_SPACE_ID>
   VITE_CONTENTFUL_DELIVERY_TOKEN=<CONTENTFUL_DELIVERY_TOKEN>
   VITE_CONTENTFUL_ENVIRONMENT=<CONTENTFUL_ENVIRONMENT>
   ```

   > **🔒 SEGURIDAD CRÍTICA:** Si este archivo no existe, créalo ahora. Si ya existe, verifica que el `.gitignore` lo excluye.

3. **Verifica el `.gitignore`** para asegurarte de que `.env` está excluido:

   ```bash
   git check-ignore -v .env
   # Criterio observable: Git muestra la regla que excluye .env
   ```

   Si no aparece, agrégalo:

   ```bash
   # Bash:
   printf ".env\n.env.local\n.env.*.local\n" >> .gitignore
   ```

   ```powershell
   # PowerShell:
   @('.env','.env.local','.env.*.local') | Add-Content .gitignore
   ```

4. **Verifica que `.env.example` existe** con las variables sin valores reales:

   ```bash
   node -e "console.log(require('fs').readFileSync('.env.example','utf8'))"
   ```

   Si no existe, créalo:

   ```dotenv
   # Copia este archivo como .env y completa con tus valores reales
   # NUNCA commitees el archivo .env con valores reales
   VITE_CONTENTFUL_SPACE_ID=<CONTENTFUL_SPACE_ID>
   VITE_CONTENTFUL_DELIVERY_TOKEN=<CONTENTFUL_DELIVERY_TOKEN>
   VITE_CONTENTFUL_ENVIRONMENT=<CONTENTFUL_ENVIRONMENT>
   ```

   Crea `.env.example` con VS Code o tu editor usando el contenido anterior.

5. **Confirma la estructura del proyecto.** Debe existir al menos:

   ```
   portal-noticias/
   ├── .env                    ← con valores reales (NO en Git)
   ├── .env.example            ← sin valores reales (SÍ en Git)
   ├── .gitignore              ← incluye .env
   ├── src/
   │   ├── lib/
   │   │   └── contentfulClient.js ← cliente configurado con SDK v10
   │   ├── App.jsx
   │   └── components/
   └── package.json
   ```

6. **Verifica que el cliente de Contentful usa la Delivery API** (no la Management API):

   ```bash
   node -e "console.log(require('fs').readFileSync('src/lib/contentfulClient.js','utf8'))"
   ```

   Debe verse similar a:

   ```javascript
   // src/lib/contentfulClient.js
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

   - Campos: `name` (Short text, Required ✓), `slug` (Short text, Required ✓, Unique ✓), `bio` (Long text), `avatar` (Media)
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
   | `author`   | 1               | "Ana García", slug `ana-garcia`, bio breve            |
   | `article`  | 3               | Artículos con título, slug, body en Rich Text, autor y categoría asignados |

   > **⚠️ Rich Text:** El campo `body` retorna un árbol JSON, NO HTML. En el frontend deberás usar `@contentful/rich-text-react-renderer` para renderizarlo. Si lo instalaste en la Práctica 4, ya está disponible.

6. **Publica todas las entradas** haciendo clic en **Publish** en cada una. Las entradas en estado "Draft" no son visibles a través de la Delivery API.

#### Resultado Esperado

En la sección **Content** del espacio de Contentful deben aparecer al menos 6 entradas publicadas (2 categorías + 1 autor + 3 artículos), todas con estado verde "Published".

#### Verificación

Verifica la Delivery API sin colocar el token en la URL:

```bash
node -e "fetch('https://cdn.contentful.com/spaces/<CONTENTFUL_SPACE_ID>/environments/<CONTENTFUL_ENVIRONMENT>/entries?content_type=article',{headers:{Authorization:'Bearer <CONTENTFUL_DELIVERY_TOKEN>'}}).then(r=>r.json()).then(d=>console.log('Artículos publicados:',d.total))"
```

Reemplaza los placeholders. El criterio observable es recibir una colección de `article` publicados; el total depende del contenido creado.

---

### Paso 4: Instalar Dependencias Adicionales si es Necesario

**Objetivo:** Asegurar que todas las dependencias del frontend están instaladas, incluyendo el renderer de Rich Text si no se instaló en prácticas anteriores.

#### Instrucciones

1. **Verifica las dependencias actuales** del proyecto:

   ```bash
   cd portal-noticias
   npm list --depth=0
   ```

2. **Instala el renderer de Rich Text** si no está presente:

   ```bash
   # Verificar si ya está instalado
   npm list @contentful/rich-text-react-renderer

   # Si no está instalado, instalarlo ahora
   npm install @contentful/rich-text-react-renderer @contentful/rich-text-types
   ```

3. **Instala React Router** si no está presente (necesario para navegación entre listado y detalle):

   ```bash
   npm list react-router-dom
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
   node -e "const fs=require('fs');fs.mkdirSync('src/services',{recursive:true});fs.closeSync(fs.openSync('src/services/contentfulService.js','a'))"
   ```

2. **Implementa el servicio** con las funciones necesarias para el portal:

   ```javascript
   // src/services/contentfulService.js
   // Servicio centralizado para todas las consultas a Contentful
   // Usa EXCLUSIVAMENTE la Delivery API (solo lectura)

   import client from '../lib/contentfulClient.js';

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
# Verificar existencia y sintaxis sin asumir una cantidad exacta de líneas
node --check src/services/contentfulService.js

# Confirmar que el servicio reutiliza el cliente centralizado
node -e "const s=require('fs').readFileSync('src/services/contentfulService.js','utf8');console.log('usa cliente centralizado:',s.includes(\"../lib/contentfulClient.js\"))"
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
                 <small>
                   Por: {article.fields.author?.fields?.name || 'Autor desconocido'} |{' '}
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

     const { title, body, author, category, publishedAt, coverImage } = article.fields;

     return (
       <article>
         <Link to="/">← Volver al listado</Link>

         <h1>{title}</h1>
         {/* Metadatos */}
         <div>
           <span>Autor: {author?.fields?.name}</span> |{' '}
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

1. **Prueba el manejo de errores** desconectando temporalmente la conexión a la API. Deja momentáneamente vacío el Delivery token en `.env`:

   ```bash
   # TEMPORAL — Solo para probar, revertir inmediatamente
   # En .env, deja temporalmente vacío el token:
   # VITE_CONTENTFUL_DELIVERY_TOKEN=
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

   Crea o actualiza el archivo con VS Code o tu editor usando esta estructura:

   ````markdown
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
   | `article`    | Artículo del portal       | title, slug, body, publishedAt, coverImage, category, author |
   | `author`     | Autor de artículos        | name, slug, bio, avatar           |
   | `category`   | Categoría de artículos    | name, slug, description               |

   ## Configuración

   1. Clona el repositorio
   2. Instala dependencias: `npm install`
   3. Copia `.env.example` a `.env`: Bash `cp .env.example .env`; PowerShell `Copy-Item .env.example .env`
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

   ````

2. **Confirma que el README se creó correctamente**:

   ```bash
   node -e "const s=require('fs').readFileSync('README.md','utf8');console.log('Líneas:',s.split(/\r?\n/).length)"
   # Criterio observable: el README contiene las secciones solicitadas y puede revisarse en el editor
   ```

#### Resultado Esperado

`README.md` en la raíz del proyecto con secciones de arquitectura, Content Types, configuración, variables de entorno, seguridad y decisiones de diseño.

---

### Paso 9: Ejecutar el Checklist Técnico de Validación

**Objetivo:** Evaluar formalmente el proyecto completo contra 20 criterios técnicos agrupados en cuatro categorías, documentando cada resultado como PASS o FAIL.

#### Instrucciones

1. **Crea el archivo de resultados del checklist**:

   ```bash
   node -e "require('fs').closeSync(require('fs').openSync('CHECKLIST_RESULTS.md','a'))"
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
| B2 | La Delivery API responde con los artículos publicados | Llamada directa con `<CONTENTFUL_SPACE_ID>`, `<CONTENTFUL_ENVIRONMENT>` y `<CONTENTFUL_DELIVERY_TOKEN>` | PASS/FAIL | |
   | B3 | Las consultas incluyen `include: 2` para resolver referencias en una sola llamada | Revisar contentfulService.js | PASS/FAIL | |
   | B4 | El filtrado por categoría se realiza como query parameter a la API (no en cliente) | Revisar la función `getArticles` con parámetro `categorySlug` | PASS/FAIL | |
   | B5 | No hay llamadas a la Management API desde el frontend | Buscar `contentful-management` en package.json y en el código fuente | PASS/FAIL | |
   
   ## Sección C: Frontend (5 ítems)
   
   | # | Criterio | Verificación | Resultado | Notas |
   |---|----------|--------------|-----------|-------|
   | C1 | La página de listado muestra al menos 3 artículos con título, fecha y autor | Verificar en http://localhost:5173 | PASS/FAIL | |
   | C2 | El filtro por categoría actualiza la lista correctamente | Hacer clic en cada botón de categoría y verificar que la lista cambia | PASS/FAIL | |
   | C3 | La página de detalle muestra el cuerpo del artículo renderizado (no [object Object]) | Navegar a /article/[slug] y verificar que el body se muestra como texto | PASS/FAIL | |
   | C4 | Los estados de carga y error se muestran correctamente en la UI | Probar con token inválido y slug inexistente | PASS/FAIL | |
   | C5 | La navegación listado ↔ detalle funciona correctamente con React Router | Verificar que el botón "Volver" y los enlaces de artículo funcionan | PASS/FAIL | |
   
   ## Sección D: Seguridad (5 ítems)
   
   | # | Criterio | Verificación | Resultado | Notas |
   |---|----------|--------------|-----------|-------|
   | D1 | El archivo `.env` NO está trackeado por Git | Ejecutar: `git ls-files .env` → no debe mostrar output | PASS/FAIL | |
   | D2 | El archivo `.env.example` SÍ está trackeado y no contiene valores reales | `git ls-files .env.example` debe mostrar el archivo; revisar que solo use placeholders | PASS/FAIL | |
   | D3 | No hay credenciales hardcodeadas en ningún archivo de código fuente | Buscar el valor real del Delivery token entre los archivos rastreados | PASS/FAIL | |
   | D4 | El `.gitignore` incluye `.env`, `.env.local` y variantes | Ejecutar `git check-ignore -v .env .env.local` | PASS/FAIL | |
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
   # Comandos portables para Bash y PowerShell
   git ls-files .env
   # D1: no debe mostrar salida

   git ls-files .env.example
   # D2: debe mostrar .env.example

   git check-ignore -v .env .env.local
   # D4: debe mostrar las reglas que ignoran ambos archivos

   node -e "const fs=require('fs');const cp=require('child_process');const env=fs.readFileSync('.env','utf8');const token=env.match(/^VITE_CONTENTFUL_DELIVERY_TOKEN=(.+)$/m)?.[1]?.trim();const files=cp.execFileSync('git',['ls-files'],{encoding:'utf8'}).split(/\r?\n/).filter(Boolean);const hits=token?files.filter(f=>{try{return fs.readFileSync(f,'utf8').includes(token)}catch{return false}}):[];console.log(hits.length?'FAIL: revisar '+hits.join(', '):'PASS: valor del token no detectado en archivos rastreados')"

   node -e "const fs=require('fs');const p=require('path');function walk(d){return fs.readdirSync(d,{withFileTypes:true}).flatMap(e=>{const f=p.join(d,e.name);return e.isDirectory()?walk(f):[f]})}const hits=walk('src').filter(f=>fs.readFileSync(f,'utf8').includes('contentful-management'));console.log(hits.length?'FAIL: revisar '+hits.join(', '):'PASS: sin contentful-management en frontend')"
   ```

4. **Completa el archivo `CHECKLIST_RESULTS.md`** con los resultados de cada ítem, incluyendo notas para los ítems FAIL.

5. **Objetivo de aprobación:** El proyecto se considera completo con un mínimo de **18/20 ítems en PASS**. Los ítems de la Sección D (Seguridad) son **obligatorios todos en PASS** sin excepción.

#### Resultado Esperado

`CHECKLIST_RESULTS.md` completado con los 20 ítems evaluados. Mínimo 18 PASS, con todos los ítems de seguridad (D1–D5) en PASS.

---

## Resultado esperado

Al finalizar, `portal-noticias` utiliza el modelo canónico y muestra el listado, filtrado y detalle de artículos. El checklist técnico registra los 20 criterios evaluados, no existen credenciales reales en el repositorio y la documentación permite ejecutar y revisar el proyecto.

## Validación final

### Prueba Integral del Portal

Ejecuta esta secuencia de pruebas funcionales completas antes de considerar la práctica terminada:

```bash
# 1. Iniciar el servidor de desarrollo
cd ~/portal-noticias
npm run dev

# 2. Abrir http://localhost:5173 y verificar:
#    ✓ La página carga sin errores en la consola del navegador
#    ✓ Se muestran los artículos con título, fecha y datos de autor
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
# Verificación portable de seguridad
git ls-files .env .env.local
# No debe mostrar salida

git check-ignore -v .env .env.local
# Debe mostrar las reglas aplicables

node -e "const fs=require('fs');const s=fs.readFileSync('src/lib/contentfulClient.js','utf8');console.log('usa cliente único:',(s.match(/createClient/g)||[]).length===1);console.log('usa variables VITE:',s.includes('import.meta.env.VITE_CONTENTFUL_DELIVERY_TOKEN'))"
```

> **Importante:** Las búsquedas automáticas son una ayuda, no una garantía. Revisa también `git status`, el staging y el historial; si un secreto real fue expuesto, rótalo.

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
// article.fields.author será { sys: {...}, fields: { name: 'Ana García', ... } }
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

# 2. Opcional: hacer un commit local del proyecto completado
cd portal-noticias
git add .
git status
# Verifica que .env NO aparece en los archivos a commitear

# Si .env no aparece, puedes crear el commit local opcional:
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

### Preguntas de reflexión

1. ¿Qué decisión del modelo de contenido tuvo mayor impacto en la implementación del frontend?
2. ¿Qué comprobación del checklist reduce el riesgo técnico más importante del proyecto?
3. ¿Qué aspectos deberían fortalecerse antes de llevar este portal a producción?

### Recursos Adicionales

- [Documentación oficial del modelo de datos de Contentful](https://www.contentful.com/developers/docs/concepts/data-model/)
- [Guía de la Delivery API REST de Contentful](https://www.contentful.com/developers/docs/references/content-delivery-api/)
- [Documentación de `@contentful/rich-text-react-renderer`](https://github.com/contentful/rich-text/tree/master/packages/rich-text-react-renderer)
- [Mejores prácticas de seguridad con API Keys en frontend](https://www.contentful.com/developers/docs/references/authentication/)
- [Guía de React Router v6](https://reactrouter.com/en/main/start/tutorial)
- [Parámetro `include` en la Delivery API](https://www.contentful.com/developers/docs/references/content-delivery-api/#/reference/links)

---
