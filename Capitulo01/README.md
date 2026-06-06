# Crear modelos y publicar primeras entradas

## 1. Metadatos

| Atributo        | Valor                                      |
|-----------------|--------------------------------------------|
| **Duración**    | 85 minutos                                 |
| **Complejidad** | Fácil                                      |
| **Nivel Bloom** | Crear (Create)                             |
| **Módulo**      | 1 — Fundamentos de Contentful              |
| **Versión**     | 1.0                                        |

---

## 2. Descripción General

En esta práctica crearás desde cero un espacio de trabajo en Contentful y diseñarás el modelo de contenido de un portal de noticias ficticio. Partirás del contraste arquitectónico entre un CMS tradicional y uno headless —visto en la lección 1.1— para justificar cada decisión de diseño que tomes. Al finalizar habrás publicado entradas reales y validarás que son accesibles mediante la Delivery API, cerrando el ciclo completo de modelado y publicación **sin escribir una sola línea de código JavaScript**.

> **Nota sobre Rich Text:** El campo Rich Text de Contentful **no** retorna HTML. Devuelve un árbol de nodos JSON llamado `Document`. En labs posteriores usarás librerías específicas para renderizarlo. Por ahora, simplemente observa su estructura en la respuesta de la API.

---

### Escenario de la práctica

Formas parte del equipo que prepara un portal de noticias. Antes de desarrollar el frontend, debes configurar en Contentful una base de contenido reutilizable, publicar ejemplos y comprobar que la Delivery API los expone correctamente.

### Objetivo de la práctica

Dejar listo el espacio de Contentful con el modelo canónico `article`, `category` y `author`, sus relaciones y un conjunto inicial de contenido publicado que se reutilizará durante el resto del curso.

### Cómo trabajar esta práctica

1. Completa los pasos en el orden indicado; los Content Types dependen unos de otros.
2. Conserva el environment `master` y usa `staging` únicamente cuando el paso lo solicite.
3. Compara cada configuración con su sección **Resultado esperado**.
4. No avances al siguiente laboratorio hasta completar la **Validación final**.

> **Importante:** Conserva el espacio, los Content Types y las entradas creadas. Son requisitos para las prácticas 2 a 5.

---

## 3. Objetivos de Aprendizaje

Al completar este lab serás capaz de:

- [ ] Distinguir las diferencias arquitectónicas entre un CMS tradicional acoplado y un CMS headless, justificando cuándo aplicar cada enfoque.
- [ ] Crear y configurar un espacio (Space) en Contentful con dos entornos: `master` y `staging`.
- [ ] Diseñar los tres Content Types canónicos (`article`, `category` y `author`) con sus campos y validaciones.
- [ ] Publicar entradas (entries) y assets (imágenes) siguiendo el ciclo de vida `draft → published`.
- [ ] Explorar la respuesta JSON de la Delivery API para verificar que el modelo y las entradas publicadas son accesibles correctamente.

---

## 4. Prerrequisitos

### Conocimientos previos

| Tema                                         | Nivel requerido |
|----------------------------------------------|-----------------|
| Conceptos de CMS tradicional vs headless     | Básico (lección 1.1) |
| Lectura e interpretación de JSON             | Básico          |
| Navegación en interfaces web                 | Básico          |
| Uso de un cliente HTTP (Postman o Bruno)     | Básico          |

### Acceso y cuentas

| Recurso                        | Detalle                                                                 |
|--------------------------------|-------------------------------------------------------------------------|
| Cuenta de Contentful           | Plan Community (gratuito). Registro en https://www.contentful.com/sign-up/ |
| Postman 10.x **o** Bruno 1.x   | Instalado y funcional en la máquina local                               |
| Navegador moderno              | Chrome 120+, Firefox 121+ o Edge 120+                                   |
| Conexión a Internet            | Mínimo 10 Mbps estable                                                  |

> **Importante — Límites del plan Community:** El plan gratuito permite **1 espacio**, **2 entornos**, **25 000 registros** y **48 tipos de contenido**. Reutilizarás este mismo espacio en todos los labs del curso. No crees espacios adicionales.

---

## 5. Entorno de Laboratorio

### Hardware recomendado

| Componente       | Mínimo                        | Recomendado                  |
|------------------|-------------------------------|------------------------------|
| Procesador       | Intel Core i5 64-bit / AMD equiv. | i7 / Ryzen 5 o superior  |
| RAM              | 8 GB                          | 16 GB                        |
| Almacenamiento   | 20 GB libres                  | SSD con 40 GB libres         |
| Pantalla         | 1280 × 720                    | 1920 × 1080                  |

### Software necesario para este lab

| Software              | Versión mínima | Propósito en este lab                     |
|-----------------------|----------------|-------------------------------------------|
| Navegador web         | Cualquier moderno | Acceso a Contentful Web App            |
| Postman **o** Bruno   | 10.x / 1.x     | Inspeccionar respuestas de la Delivery API |

> Este lab **no requiere** Node.js, VS Code ni ninguna herramienta de línea de comandos. Toda la interacción ocurre en la interfaz web de Contentful y en el cliente HTTP.

### Verificación del entorno

Antes de comenzar, confirma que puedes acceder a los siguientes dominios desde tu red:

```
https://app.contentful.com
https://cdn.contentful.com
https://api.contentful.com
```

Abre cada URL en el navegador y verifica que no hay bloqueos de firewall o proxy corporativo.

---

## 6. Pasos del Laboratorio

---

### Paso 1 — Reflexión arquitectónica: CMS tradicional vs headless (10 min)

**Objetivo:** Interiorizar la diferencia entre ambas arquitecturas antes de comenzar a modelar, para que cada decisión de diseño esté fundamentada.

#### Instrucciones

1. Observa el siguiente diagrama mental que resume lo visto en la lección 1.1:

   ```
   CMS TRADICIONAL (monolítico)
   ┌─────────────────────────────────────────┐
   │  Base de datos → Backend → Plantillas   │
   │              └──────────────────────→  │
   │                    HTML al navegador    │
   └─────────────────────────────────────────┘

   CMS HEADLESS (desacoplado)
   ┌──────────────────────┐      API (JSON)     ┌─────────────────────────┐
   │  Base de datos       │ ──────────────────→ │  React / Vue / Mobile   │
   │  + Backend Contentful│                     │  Quiosco / Voz / etc.   │
   └──────────────────────┘                     └─────────────────────────┘
   ```

2. Responde mentalmente (o en papel) las siguientes preguntas antes de continuar:
   - ¿Por qué un portal de noticias que quiere publicar en web **y** en una app móvil se beneficia del modelo headless?
   - ¿Qué parte del sistema de WordPress equivale al "head" que Contentful no tiene?
   - ¿Qué formato de datos usa Contentful para exponer el contenido? (Respuesta esperada: JSON vía REST o GraphQL)

3. Identifica los dos roles que ejercerás durante este lab:
   - **Editor de contenido:** creas y publicas entradas en la interfaz web.
   - **Desarrollador consumidor:** consultas la API para verificar que el contenido está disponible.

#### Resultado esperado

Comprensión clara de que en este lab modelarás el **backend de contenido** (Contentful) y al final actuarás como un frontend que consume su API, sin que ambas capas estén acopladas.

#### Verificación

Responde en voz alta o por escrito: *"En Contentful, cuando publico un artículo, ¿el sistema genera una página HTML automáticamente?"* — La respuesta correcta es **No**. Contentful solo expone el contenido como JSON a través de su API; la presentación es responsabilidad del frontend.

---

### Paso 2 — Crear el espacio (Space) en Contentful (8 min)

**Objetivo:** Configurar el espacio de trabajo que se usará en todos los labs del curso.

#### Instrucciones

1. Inicia sesión en [https://app.contentful.com](https://app.contentful.com).

2. Si es tu primera vez, Contentful te guiará por un asistente de onboarding. **Omite el asistente** haciendo clic en "Explore on your own" o el enlace equivalente para ir directamente al dashboard.

3. En el dashboard principal, haz clic en **"Add a space"** (esquina superior izquierda, junto al nombre de tu organización).

4. Selecciona **"Create an empty space"** (no uses las plantillas predefinidas; construirás el modelo desde cero).

5. Completa el formulario:

   | Campo           | Valor                          |
   |-----------------|--------------------------------|
   | Space name      | `portal-noticias-curso`        |
   | Plan            | Community (Free) — ya seleccionado |

6. Haz clic en **"Proceed to confirmation"** y luego en **"Confirm and create space"**.

7. Espera a que el espacio se cree (5–10 segundos). Serás redirigido al dashboard del nuevo espacio.

8. **Anota el Space ID:** ve a **Settings → General settings** y copia el valor del campo **Space ID**. Guárdalo en un bloc de notas; lo necesitarás en el Paso 6.

   ```
   Space ID: xxxxxxxxxxxx   ← cópialo aquí en tu bloc de notas
   ```

#### Resultado esperado

Un espacio vacío llamado `portal-noticias-curso` visible en el dashboard de Contentful, con el entorno `master` creado automáticamente.

#### Verificación

Navega a **Settings → General settings** y confirma que:
- El nombre del espacio es `portal-noticias-curso`.
- El Space ID está visible y anotado.
- El entorno activo es `master`.

---

### Paso 3 — Crear el entorno de staging (7 min)

**Objetivo:** Configurar un segundo entorno (`staging`) que simule un flujo de trabajo real con separación entre producción y pre-producción.

#### Instrucciones

1. Dentro del espacio `portal-noticias-curso`, ve a **Settings → Environments**.

2. Haz clic en **"Add environment"**.

3. Completa el formulario:

   | Campo              | Valor                                      |
   |--------------------|--------------------------------------------|
   | Environment ID     | `staging`                                  |
   | Clone from         | `master` (clona la estructura base)        |

4. Haz clic en **"Create environment"**. El proceso puede tardar 10–20 segundos.

5. Una vez creado, verás dos entornos en la lista:

   | Entorno   | Propósito                                                      |
   |-----------|----------------------------------------------------------------|
   | `master`  | Producción. Contenido publicado y accesible por el frontend.   |
   | `staging` | Pre-producción. Pruebas de modelo y contenido sin afectar prod.|

6. **Vuelve al entorno `master`** para los pasos siguientes: haz clic en el selector de entornos (parte superior de la barra lateral) y selecciona `master`.

> **Nota:** El contrato técnico del curso usa exclusivamente el environment `master`. Conserva `staging` como referencia conceptual, pero no lo selecciones en los laboratorios posteriores.

#### Resultado esperado

Dos entornos visibles en **Settings → Environments**: `master` y `staging`.

#### Verificación

Confirma que el selector de entornos en la barra lateral muestra `master` como entorno activo antes de continuar.

---

### Paso 4 — Crear los Content Types `category` y `author` (12 min)

**Objetivo:** Diseñar primero los Content Types referenciados por `article`.

#### Instrucciones

1. En la barra lateral izquierda, haz clic en **"Content model"**.

2. Haz clic en **"Add content type"**.

3. Crea el Content Type de categoría:

   | Campo           | Valor                                        |
   |-----------------|----------------------------------------------|
   | Name            | `Category`                                   |
   | API Identifier  | `category`                                   |
   | Description     | `Categorías temáticas del portal de noticias` |

4. Haz clic en **"Create"**.

5. Añade los siguientes campos:

   **Campo 1 — Nombre:**

   | Propiedad        | Valor                  |
   |------------------|------------------------|
   | Field type       | Short text             |
   | Name             | `Nombre`               |
   | Field ID         | `name`               |

   - Después de crearlo, haz clic en el campo y activa **"Required field"**.
   - En la pestaña **"Validations"**, activa **"Unique field"** para evitar categorías duplicadas.
   - Haz clic en **"Confirm"**.

   **Campo 2 — Descripción:**

   | Propiedad        | Valor                  |
   |------------------|------------------------|
   | Field type       | Long text              |
   | Name             | `Descripción`          |
   | Field ID         | `description`          |

   - No es obligatorio. Haz clic en **"Confirm"**.

   **Campo 3 — Slug:**

   | Propiedad        | Valor                  |
   |------------------|------------------------|
   | Field type       | Short text             |
   | Name             | `Slug`                 |
   | Field ID         | `slug`                 |

   - Activa **"Required field"** y **"Unique field"**.
   - En **Validations → Match a specific pattern**, usa `^[a-z0-9]+(?:-[a-z0-9]+)*$`.
   - Haz clic en **"Confirm"**.

6. Guarda y activa el Content Type `category`.

7. Crea el Content Type `Author` con API Identifier `author` y estos campos:

   | Name | Field ID | Field type | Configuración |
   |---|---|---|---|
   | Nombre | `name` | Short text | Required |
   | Slug | `slug` | Short text | Required, Unique, patrón `^[a-z0-9]+(?:-[a-z0-9]+)*$` |
   | Biografía | `bio` | Long text | Opcional |
   | Avatar | `avatar` | Media | Opcional, solo imágenes |

8. Guarda y activa el Content Type `author`.

#### Resultado esperado

Los Content Types `category` y `author` aparecen activos con los campos definidos en el contrato técnico del curso.

#### Verificación

- Verifica que `category` contiene exactamente `name`, `slug` y `description`.
- Verifica que `author` contiene exactamente `name`, `slug`, `bio` y `avatar`.
- Confirma que los API Identifiers son `category` y `author`.

---

### Paso 5 — Crear el Content Type `article` (15 min)

**Objetivo:** Diseñar el Content Type principal del portal con referencias a `category` y `author`.

#### Instrucciones

1. En **Content model**, haz clic en **"Add content type"**.

2. Completa el formulario inicial:

   | Campo           | Valor                                           |
   |-----------------|-------------------------------------------------|
   | Name            | `Article`                                       |
   | API Identifier  | `article`                                      |
   | Description     | `Artículos de noticias del portal`              |

3. Haz clic en **"Create"** y añade los siguientes campos:

   **Campo 1 — Título:**

   | Propiedad        | Valor                  |
   |------------------|------------------------|
   | Field type       | Short text             |
   | Name             | `Título`               |
   | Field ID         | `title`               |

   - Activa **"Required field"**.
   - En **Validations**, establece **"Limit character count"**: mínimo `5`, máximo `120`.
   - Haz clic en **"Confirm"**.

   **Campo 2 — Slug:**

   | Propiedad        | Valor                  |
   |------------------|------------------------|
   | Field type       | Short text             |
   | Name             | `Slug`                 |
   | Field ID         | `slug`                 |

   - Activa **"Required field"**.
   - Activa **"Unique field"**.
   - En **Validations → Match a specific pattern**, introduce:
     ```
     ^[a-z0-9]+(?:-[a-z0-9]+)*$
     ```
   - Custom error message: `Solo letras minúsculas, números y guiones (ej: mi-article-2024)`
   - En **Validations → Limit character count**: máximo `80`.
   - Haz clic en **"Confirm"**.

   **Campo 3 — Cuerpo (Rich Text):**

   | Propiedad        | Valor                  |
   |------------------|------------------------|
   | Field type       | Rich text              |
   | Name             | `Cuerpo`               |
   | Field ID         | `body`               |

   - Activa **"Required field"**.
   - Haz clic en **"Confirm"**.

   > ⚠️ **Advertencia importante:** Este campo **NO** retornará HTML en la API. Retornará un árbol JSON con nodos de tipo `Document`. En los Labs 3 y 4 aprenderás a renderizarlo correctamente con `@contentful/rich-text-react-renderer`.

   **Campo 4 — Fecha de publicación:**

   | Propiedad        | Valor                  |
   |------------------|------------------------|
   | Field type       | Date and time          |
   | Name             | `Fecha de publicación` |
   | Field ID         | `publishedAt`     |

   - Activa **"Required field"**.
   - Haz clic en **"Confirm"**.

   **Campo 5 — Imagen destacada:**

   | Propiedad        | Valor                  |
   |------------------|------------------------|
   | Field type       | Media                  |
   | Name             | `Imagen destacada`     |
   | Field ID         | `coverImage`      |

   - En **Validations**, activa **"Accept only specified file types"** y selecciona únicamente **"Images"**.
   - Activa **"Required field"**.
   - Haz clic en **"Confirm"**.

   **Campo 6 — Categoría (referencia):**

   | Propiedad        | Valor                  |
   |------------------|------------------------|
   | Field type       | Reference              |
   | Name             | `Categoría`            |
   | Field ID         | `category`            |

   - Activa **"Required field"**.
   - En **Validations → Accept only specified entry type**, selecciona `category`.
   - Haz clic en **"Confirm"**.

   **Campo 7 — Autor (referencia):**

   | Propiedad        | Valor                  |
   |------------------|------------------------|
   | Field type       | Reference              |
   | Name             | `Autor`                |
   | Field ID         | `author`               |

   - Activa **"Required field"**.
   - En **Validations → Accept only specified entry type**, selecciona `author`.
   - Haz clic en **"Confirm"**.

4. Haz clic en **"Save"** y luego en **"Activate"**.

#### Resultado esperado

El Content Type `article` aparece en el listado con 7 campos. Los campos `category` y `author` muestran referencias a sus Content Types correspondientes.

#### Verificación

Revisa el diagrama de relaciones en Content model: debes ver `article` → `category` y `article` → `author`. Confirma que los siete campos canónicos existen.

---

### Paso 6 — Publicar entradas de `category` y `author` (8 min)

**Objetivo:** Crear y publicar las categorías y el autor antes de los artículos, ya que los artículos los referencian.

#### Instrucciones

1. En la barra lateral, haz clic en **"Content"**.

2. Haz clic en **"Add entry"** → selecciona **"Category"**.

3. Crea la primera categoría con los siguientes datos:

   | Campo        | Valor                                                   |
   |--------------|---------------------------------------------------------|
   | Nombre       | `Tecnología`                                            |
   | Slug         | `tecnologia`                                            |
   | Descripción  | `Noticias sobre innovación, software y hardware`        |

4. Haz clic en **"Publish"** (botón verde en la esquina superior derecha). El estado cambiará de `Draft` a `Published`.

5. Repite el proceso para la segunda categoría:

   | Campo        | Valor                                                   |
   |--------------|---------------------------------------------------------|
   | Nombre       | `Ciencia`                                               |
   | Slug         | `ciencia`                                               |
   | Descripción  | `Descubrimientos científicos y avances en investigación`|

6. Publica también esta segunda categoría.

7. Crea y publica una entrada de `Author`:

   | Campo | Valor |
   |---|---|
   | Nombre | `Ana García` |
   | Slug | `ana-garcia` |
   | Biografía | `Autora principal del portal de noticias` |
   | Avatar | Opcional |

#### Resultado esperado

En la vista **Content** aparecen 2 entradas de `category` y 1 entrada de `author` con estado `Published`.

#### Verificación

- Confirma que ambas entradas muestran el estado **Published** en la columna de estado.
- Confirma que la entrada `Ana García` está publicada y tiene slug `ana-garcia`.

---

### Paso 7 — Subir assets (imágenes) (7 min)

**Objetivo:** Cargar las imágenes que usarán los artículos como imagen destacada.

#### Instrucciones

1. En la barra lateral, haz clic en **"Media"**.

2. Haz clic en **"Add asset"** → **"Single asset"**.

3. Para el primer asset:
   - Haz clic en **"Upload"** y selecciona una imagen de tu computadora (JPG o PNG, mínimo 800×400 px). Puedes descargar imágenes de libre uso de [https://unsplash.com](https://unsplash.com).
   - En el campo **Title**, escribe: `Imagen artículo tecnología 1`
   - En el campo **Description**, escribe: `Foto representativa de tecnología`
   - Haz clic en **"Publish"** una vez que la imagen haya terminado de procesarse.

4. Repite el proceso para dos assets adicionales:
   - `Imagen artículo tecnología 2` — otra imagen de tecnología
   - `Imagen artículo ciencia 1` — imagen relacionada con ciencia

5. Confirma que los 3 assets tienen estado **Published** en la vista de Media.

> **Consejo:** Contentful procesa las imágenes y genera múltiples resoluciones automáticamente. El procesamiento puede tardar 5–15 segundos por imagen.

#### Resultado esperado

3 assets publicados visibles en la vista **Media**, cada uno con una URL de imagen funcional.

#### Verificación

Haz clic en cualquier asset publicado y copia la URL del campo **File URL**. Pégala en el navegador y confirma que la imagen se visualiza correctamente. La URL tendrá el formato:
```
https://images.ctfassets.net/{spaceId}/{assetId}/{token}/nombre-imagen.jpg
```

---

### Paso 8 — Publicar entradas de `article` (12 min)

**Objetivo:** Crear y publicar 3 artículos que referencien las categorías, el autor y los assets creados previamente.

#### Instrucciones

1. En **Content**, haz clic en **"Add entry"** → **"Article"**.

2. Crea el **primer artículo**:

   | Campo                | Valor                                                                 |
   |----------------------|-----------------------------------------------------------------------|
   | Título               | `La inteligencia artificial transforma el desarrollo de software`    |
   | Slug                 | `ia-transforma-desarrollo-software`                                  |
   | Cuerpo (Rich Text)   | Escribe al menos 3 párrafos usando el editor. Usa **negrita**, *itálica* y al menos un encabezado H2 para explorar las opciones del editor. |
   | Fecha de publicación | Selecciona la fecha de hoy                                           |
   | Imagen destacada     | Selecciona `Imagen artículo tecnología 1`                            |
   | Categoría            | Selecciona `Tecnología`                                              |
   | Autor                | Selecciona `Ana García`                                              |

3. Haz clic en **"Publish"**.

4. Crea el **segundo artículo**:

   | Campo                | Valor                                                                 |
   |----------------------|-----------------------------------------------------------------------|
   | Título               | `Nuevas fronteras en computación cuántica`                           |
   | Slug                 | `computacion-cuantica-nuevas-fronteras`                              |
   | Cuerpo (Rich Text)   | Al menos 2 párrafos sobre computación cuántica (puede ser texto ficticio). |
   | Fecha de publicación | Fecha de ayer                                                        |
   | Imagen destacada     | Selecciona `Imagen artículo tecnología 2`                            |
   | Categoría            | Selecciona `Tecnología`                                              |
   | Autor                | Selecciona `Ana García`                                              |

5. Haz clic en **"Publish"**.

6. Crea el **tercer artículo**:

   | Campo                | Valor                                                                 |
   |----------------------|-----------------------------------------------------------------------|
   | Título               | `Descubren nueva especie de bacteria resistente al calor extremo`    |
   | Slug                 | `nueva-especie-bacteria-calor-extremo`                               |
   | Cuerpo (Rich Text)   | Al menos 2 párrafos sobre el descubrimiento ficticio.                |
   | Fecha de publicación | Fecha de hace 3 días                                                 |
   | Imagen destacada     | Selecciona `Imagen artículo ciencia 1`                               |
   | Categoría            | Selecciona `Ciencia`                                                 |
   | Autor                | Selecciona `Ana García`                                              |

7. Haz clic en **"Publish"**.

#### Resultado esperado

En la vista **Content** filtrada por tipo **Article**, aparecen 3 entradas con estado `Published`.

#### Verificación

- Verifica que los 3 artículos muestran estado `Published`.
- Abre el primer artículo y confirma que el campo `Categoría` muestra el enlace a la entrada `Tecnología` (no un ID vacío).
- Confirma que el campo `Autor` muestra el enlace a la entrada `Ana García`.
- Confirma que el campo `Cuerpo` muestra el contenido Rich Text con formato visual en el editor.

---

### Paso 9 — Obtener la API Key de entrega (5 min)

**Objetivo:** Generar las credenciales necesarias para consultar la Delivery API.

#### Instrucciones

1. Ve a **Settings → API keys**.

2. Haz clic en **"Add API key"**.

3. Completa el formulario:

   | Campo       | Valor                              |
   |-------------|------------------------------------|
   | Name        | `Lab01 - Delivery Key`             |
   | Description | `Clave de entrega para el Lab 01`  |

4. Haz clic en **"Save"**.

5. En la pantalla de detalle de la API Key, anota los siguientes valores en tu bloc de notas:

   ```
   Space ID:              <CONTENTFUL_SPACE_ID>
   Content Delivery API - access token:  <CONTENTFUL_DELIVERY_TOKEN>
   Content Preview API - access token:   <CONTENTFUL_PREVIEW_TOKEN>
   Environment:           <CONTENTFUL_ENVIRONMENT> (usar master)
   ```

> ⚠️ **Seguridad crítica:** Aunque la Content Delivery API Key es de solo lectura y puede usarse en frontends públicos, **nunca la expongas en repositorios Git**. En los labs siguientes usarás un archivo `.env` para manejarla. Por ahora, guárdala únicamente en tu bloc de notas local.
>
> **Diferencia importante entre tipos de keys:**
> - **Content Delivery API Key** — Solo lectura. Para contenido publicado. Segura en frontends.
> - **Content Preview API Key** — Solo lectura. Para borradores (draft). Solo en entornos de preview.
> - **Content Management API Key** — Lectura y escritura. **NUNCA en frontends ni en repositorios públicos.**

#### Resultado esperado

Una API Key creada con su `access token` visible y anotado.

#### Verificación

Confirma que en la pantalla de la API Key puedes ver dos tokens: el de **Content Delivery API** y el de **Content Preview API**. El token de Management API se gestiona desde una sección diferente (**Settings → API keys → Content management tokens**).

---

### Paso 10 — Consultar la Delivery API con Postman (12 min)

**Objetivo:** Actuar como consumidor de la API para verificar que el modelo y las entradas son accesibles, y explorar la estructura JSON resultante.

#### Instrucciones

1. Abre **Postman** (o Bruno).

2. Crea una nueva **colección** llamada `Contentful - Portal Noticias`.

3. **Consulta 1: Listar todos los artículos**

   Crea una nueva request con los siguientes datos:

   | Propiedad    | Valor                                                                                      |
   |--------------|--------------------------------------------------------------------------------------------|
   | Method       | `GET`                                                                                      |
   | URL          | `https://cdn.contentful.com/spaces/<CONTENTFUL_SPACE_ID>/environments/<CONTENTFUL_ENVIRONMENT>/entries`                |
   | Query Params | `content_type` = `article` |
   | Query Params | `access_token` = `<CONTENTFUL_DELIVERY_TOKEN>` |

   Reemplaza `<CONTENTFUL_SPACE_ID>` y `<CONTENTFUL_DELIVERY_TOKEN>` con los valores anotados en el Paso 9.

   Haz clic en **"Send"** y observa la respuesta.

4. Verifica que la respuesta tiene status **`200 OK`** y una estructura similar a:

   ```json
   {
     "sys": { "type": "Array" },
     "total": 3,
     "skip": 0,
     "limit": 100,
     "items": [
       {
         "sys": {
           "id": "xxxxxxxx",
           "type": "Entry",
           "contentType": {
            "sys": { "id": "article" }
           }
         },
         "fields": {
          "title": "La inteligencia artificial transforma el desarrollo de software",
           "slug": "ia-transforma-desarrollo-software",
          "body": {
             "nodeType": "document",
             "content": [ ... ]
           },
          "publishedAt": "2024-06-01T00:00:00",
          "category": {
            "sys": { "type": "Link", "linkType": "Entry", "id": "yyyyyyyy" }
          },
          "author": {
            "sys": { "type": "Link", "linkType": "Entry", "id": "yyyyyyyy" }
          }
         }
       }
     ],
     "includes": { ... }
   }
   ```

   > **Observa:** El campo `body` (Rich Text) **no es HTML**. Es un objeto JSON con `nodeType: "document"` y un array `content` con nodos anidados. Este es el comportamiento no intuitivo advertido en la descripción del lab.

   > **Observa también:** El campo `category` no contiene los datos de la categoría directamente; contiene un objeto `Link` con el `id` de la entrada referenciada. Los datos completos de la categoría aparecen en el objeto `includes.Entry` al final de la respuesta.

5. **Consulta 2: Obtener un artículo específico por slug**

   Crea una segunda request:

   | Propiedad    | Valor                                                                                      |
   |--------------|--------------------------------------------------------------------------------------------|
   | Method       | `GET`                                                                                      |
   | URL          | `https://cdn.contentful.com/spaces/<CONTENTFUL_SPACE_ID>/environments/<CONTENTFUL_ENVIRONMENT>/entries`                |
   | Query Params | `content_type` = `article` |
   | Query Params | `fields.slug` = `ia-transforma-desarrollo-software` |
   | Query Params | `access_token` = `<CONTENTFUL_DELIVERY_TOKEN>` |

   Verifica que la respuesta retorna exactamente **1 item** correspondiente al artículo con ese slug.

6. **Consulta 3: Listar todas las categorías**

   Crea una tercera request:

   | Propiedad    | Valor                                                                                      |
   |--------------|--------------------------------------------------------------------------------------------|
   | Method       | `GET`                                                                                      |
   | URL          | `https://cdn.contentful.com/spaces/<CONTENTFUL_SPACE_ID>/environments/<CONTENTFUL_ENVIRONMENT>/entries`                |
   | Query Params | `content_type` = `category` |
   | Query Params | `access_token` = `<CONTENTFUL_DELIVERY_TOKEN>` |

   Verifica que retorna **2 items** (Tecnología y Ciencia).

7. Guarda las 3 requests en la colección `Contentful - Portal Noticias`.

#### Resultado esperado

Tres requests exitosas con status `200 OK`:
- La primera retorna `"total": 3` artículos.
- La segunda retorna `"total": 1` artículo con el slug especificado.
- La tercera retorna `"total": 2` categorías.

#### Verificación

En la respuesta de la Consulta 1, expande el objeto `includes.Entry` y confirma que contiene las entradas de las categorías referenciadas. Esto demuestra que Contentful resuelve automáticamente las referencias en la respuesta cuando se usa el parámetro `include` (por defecto `include=1`).

---

## 7. Validación final

Ejecuta la siguiente lista de verificación completa antes de dar el lab por terminado:

### Lista de verificación final

| # | Verificación                                                                 | Resultado esperado |
|---|------------------------------------------------------------------------------|--------------------|
| 1 | El espacio `portal-noticias-curso` existe en Contentful                     | ✅ Visible en dashboard |
| 2 | Existen dos entornos: `master` y `staging`                                  | ✅ En Settings → Environments |
| 3 | Los Content Types `category` y `author` tienen los campos canónicos         | ✅ API IDs y Field IDs exactos |
| 4 | El Content Type `article` tiene 7 campos canónicos                          | ✅ Referencias a `category` y `author` |
| 5 | Existen 2 categorías y 1 autor publicados                                   | ✅ Estado `Published` |
| 6 | Existen 3 assets de imagen publicados                                       | ✅ Estado `Published` en Media |
| 7 | Existen 3 entradas de `article` publicadas con todos los campos completos   | ✅ Estado `Published` |
| 8 | La Delivery API retorna los 3 artículos con status 200                      | ✅ `"total": 3` en Postman |
| 9 | El campo `body` en la respuesta JSON es un objeto `Document`, no HTML     | ✅ `"nodeType": "document"` |
| 10 | El campo `category` en la respuesta es un `Link`, no los datos directos   | ✅ `"type": "Link"` en `sys` |

### Prueba de validación de campos (opcional, +5 min)

Para confirmar que las validaciones funcionan:

1. Intenta crear una nueva entrada de `Article` con un slug que contenga mayúsculas, por ejemplo: `Mi-Article-Invalido`.
2. Intenta publicarla. Contentful debe mostrar un error de validación indicando que el slug no cumple el patrón.
3. Intenta crear una entrada de `Author` con un slug repetido. Contentful debe impedir publicarla por la validación de unicidad.
4. Elimina estas entradas de prueba sin publicarlas.

---

## 8. Resolución de Problemas

### Problema 1: La Delivery API retorna `401 Unauthorized`

**Síntoma:**
```json
{
  "sys": { "type": "Error", "id": "AccessTokenInvalid" },
  "message": "The access token you sent could not be found or is invalid.",
  "status": 401
}
```

**Causa:** El `access_token` en la URL de Postman es incorrecto, está mal copiado (con espacios extra o caracteres faltantes), o se está usando el token de **Content Management API** en lugar del de **Content Delivery API**.

**Solución:**
1. Ve a **Settings → API keys** en Contentful.
2. Haz clic en la key `Lab01 - Delivery Key`.
3. Copia el token del campo **"Content Delivery API - access token"** (no el de Preview ni el de Management).
4. En Postman, actualiza el valor del query param `access_token` con el token recién copiado.
5. Asegúrate de que no hay espacios antes o después del token.
6. Verifica también que el `<CONTENTFUL_SPACE_ID>` en la URL corresponde al Space ID de tu espacio (visible en **Settings → General settings**).

---

### Problema 2: La consulta retorna `"total": 0` aunque las entradas existen

**Síntoma:** La request a la Delivery API retorna status `200 OK` pero el cuerpo de la respuesta muestra `"total": 0` y el array `items` está vacío, a pesar de que las entradas son visibles en la interfaz web de Contentful.

**Causa:** Las entradas están en estado `Draft` (borrador) y no han sido publicadas. La **Content Delivery API** solo expone contenido en estado **Published**. Alternativamente, el parámetro `content_type` puede estar mal escrito (diferencia entre `article` y `Artículo`).

**Solución:**
1. Ve a **Content** en Contentful y verifica que las entradas muestran el estado `Published` (ícono verde). Si muestran `Draft` (ícono gris), abre cada entrada y haz clic en **"Publish"**.
2. Verifica el valor del query param `content_type` en Postman: debe ser exactamente `article` (el API Identifier del Content Type, en minúsculas, sin acento), no `Artículo`.
3. Si acabas de publicar las entradas, espera 10–15 segundos y reintenta la consulta (la Delivery API tiene un caché con TTL corto).
4. Si el problema persiste, verifica que el entorno en la URL de la API es `master` y no `staging`.

---

## 9. Limpieza

> **Importante:** A diferencia de otros labs de infraestructura, **NO elimines** el espacio ni el modelo de contenido creado en este lab. Los Labs 02, 03, 04 y 05 dependen directamente de este espacio y modelo.

### Acciones de limpieza recomendadas

1. **Elimina las entradas de prueba de validación** (si las creaste en la sección 7): ve a **Content**, filtra por `Draft` y elimina cualquier entrada que hayas creado para probar validaciones y que no publicaste.

2. **Revoca acceso a la API Key si compartes la computadora:** Si trabajas en una máquina compartida, considera revocar la API Key al finalizar la sesión (**Settings → API keys** → selecciona la key → **"Delete"**) y crearla nuevamente en la próxima sesión.

3. **Cierra la sesión de Contentful** si trabajas en un equipo compartido: haz clic en tu avatar → **"Log out"**.

4. **Guarda tus notas:** Asegúrate de tener anotados en un lugar seguro (no en un repositorio Git):
   - Space ID
   - Content Delivery API access token
   - Los slugs de los 3 artículos creados

### Estado esperado al finalizar

| Recurso                    | Estado           |
|----------------------------|------------------|
| Espacio `portal-noticias-curso` | ✅ Activo, conservar |
| Entorno `master`           | ✅ Activo, conservar |
| Entorno `staging`          | ✅ Activo, conservar |
| Content Type `category`    | ✅ Activo, conservar |
| Content Type `author`      | ✅ Activo, conservar |
| Content Type `article`     | ✅ Activo, conservar |
| 2 entradas de Categoría    | ✅ Published, conservar |
| 1 entrada de Autor         | ✅ Published, conservar |
| 3 assets de imagen         | ✅ Published, conservar |
| 3 entradas de Artículo     | ✅ Published, conservar |
| API Key `Lab01 - Delivery Key` | ✅ Activa, conservar |

---

## 10. Resumen

### Lo que construiste

En este lab realizaste el ciclo completo de modelado y publicación de contenido en un CMS headless:

1. **Reflexionaste** sobre la diferencia arquitectónica entre CMS tradicional y headless, comprendiendo que Contentful nunca genera HTML sino que expone JSON a través de su API.
2. **Creaste un espacio** de trabajo con dos entornos (`master` y `staging`), simulando un flujo de trabajo profesional con separación entre producción y pre-producción.
3. **Diseñaste tres Content Types** (`category`, `author` y `article`) con campos de distintos tipos y validaciones personalizadas.
4. **Publicaste contenido real**: 2 categorías, 1 autor, 3 assets de imagen y 3 artículos, siguiendo el ciclo de vida `draft → published`.
5. **Validaste la API** consultando la Delivery API desde Postman y observando la estructura JSON resultante, incluyendo el comportamiento no intuitivo del campo Rich Text (árbol `Document`) y las referencias (objeto `Link`).

### Conceptos clave reforzados

| Concepto                        | Aplicación en este lab                                               |
|---------------------------------|----------------------------------------------------------------------|
| CMS headless vs tradicional     | Contentful no genera HTML; el frontend consume JSON de su API        |
| Content Type                    | Plantilla de estructura para un tipo de contenido (`article`, `category`, `author`) |
| Entry                           | Instancia concreta de un Content Type con datos reales              |
| Asset                           | Archivo multimedia (imagen) gestionado por Contentful               |
| Draft → Published               | Ciclo de vida del contenido; solo `Published` es visible en la Delivery API |
| Content Delivery API            | API REST de solo lectura para contenido publicado                   |
| Rich Text como JSON             | El campo Rich Text retorna un árbol `Document`, no HTML             |
| Referencias como Links          | Los campos de referencia retornan `{ "sys": { "type": "Link" } }` en la API |

### Preparación para el siguiente lab

En el **Lab 02** comenzarás a consumir este mismo modelo de contenido usando el **SDK de JavaScript de Contentful (v10.x)** desde Node.js. Aprenderás a:
- Instalar y configurar el SDK.
- Realizar consultas con filtros y paginación.
- Manejar errores y respetar los límites vigentes de la API usando respuestas `429` y headers de reintento.
- Gestionar las API Keys de forma segura mediante variables de entorno (`.env`).

### Preguntas de reflexión

1. ¿Qué ventajas aporta modelar `author` y `category` como Content Types separados de `article`?
2. ¿Qué diferencia práctica observaste entre guardar una entrada como borrador y publicarla?
3. ¿Por qué el campo `body` se entrega como un documento JSON en lugar de HTML?

### Recursos adicionales

| Recurso                                                                                      | Propósito                                         |
|----------------------------------------------------------------------------------------------|---------------------------------------------------|
| [Documentación oficial: Content model](https://www.contentful.com/developers/docs/concepts/data-model/) | Referencia completa del modelo de datos de Contentful |
| [Content Delivery API Reference](https://www.contentful.com/developers/docs/references/content-delivery-api/) | Todos los endpoints, parámetros y filtros disponibles |
| [Contentful: Rich Text](https://www.contentful.com/developers/docs/concepts/rich-text/)      | Estructura del árbol JSON de Rich Text            |
| [Validations Reference](https://www.contentful.com/developers/docs/references/content-management-api/#/reference/content-types/content-type/create-a-content-type/console) | Tipos de validaciones disponibles en Content Types |
| [Smashing Magazine: Headless CMS Explained](https://www.smashingmagazine.com/2021/08/history-future-jamstack/) | Contexto arquitectónico del modelo headless       |

---

> **Recuerda para los próximos labs:** El espacio, el modelo de contenido y las entradas que creaste hoy son la base de todos los laboratorios del curso. Conserva este estado y completa la práctica antes de continuar.
