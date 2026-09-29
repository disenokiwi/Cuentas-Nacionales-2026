# Cuentas Nacionales 2026 — Handoff de diseño

Documento de traspaso para continuar el trabajo de diseño en otra cuenta/sesión de Claude.
Última actualización: commit `a9814e1` (ver `git log` para el historial completo).

## 1. Qué es este proyecto

Landing page del informe "La economía ecuatoriana creció 2,1% en el primer trimestre de 2026"
(cifras preliminares del Banco Central del Ecuador). Es un sitio estático (HTML/CSS/JS puro,
sin framework ni build step), con marca Keyword / KeyEconomics.

- **Repo GitHub:** https://github.com/disenokiwi/Cuentas-Nacionales-2026 (rama `main`)
- **Deploy Vercel:** https://cuentas-nacionales-2026.vercel.app (auto-deploy al hacer push a `main`)
- **Carpeta local:** `/Users/sofiaandrade/Documents/Claude/Repositorio 2/`
- **Fuente de la información:** un documento de Notion del que se extrajo el texto original
  (BCE, Cuentas Nacionales Trimestrales, 1T 2026).

### ⚠️ Regla de contenido (la más importante)

**No se puede cambiar ninguna cifra ni palabra del texto del informe.** Sí se puede:
- resaltar cifras dentro del texto (negrita, color),
- reordenar/reestructurar visualmente el mismo contenido,
- cambiar el diseño, colores, tipografía, animaciones, íconos, fotos.

Si en algún momento hace falta un dato que no está en el texto original, no inventarlo —
preguntar al usuario o volver a la fuente de Notion.

### ⚠️ Estado de git — hay commits sin subir

Al momento de escribir esto, el repo local tiene **16 commits por delante de `origin/main`**
(todo el trabajo de `editorial.html`, ver sección 3). No se subieron a GitHub porque el usuario
no lo pidió explícitamente en esta sesión. Antes de seguir trabajando desde otra cuenta, correr:

```bash
cd "/Users/sofiaandrade/Documents/Claude/Repositorio 2"
git log origin/main..HEAD --oneline   # ver qué falta subir
git push origin main                   # subir (pedir confirmación al usuario primero)
```

## 2. Dos diseños en el mismo repo

| Archivo | Qué es | Estado |
|---|---|---|
| `index.html` | Diseño original, ya publicado y aprobado. Secciones apiladas con menú superior. | Live en Vercel. **No tocar sin que el usuario lo pida.** |
| `editorial.html` | Diseño alternativo "editorial": barra lateral fija oscura + columna de historia con scroll. Es el que se estuvo iterando en esta sesión. | En progreso, no está enlazado desde ningún lado todavía. Al hacer push, queda accesible en `https://cuentas-nacionales-2026.vercel.app/editorial.html`. |

Ambos comparten la carpeta `assets/` (fotos, gráficos, logos, fuente, video).

## 3. `editorial.html` — arquitectura y decisiones

### Layout
- **Desktop (≥1000px):** grid de 2 columnas — `.side` (barra lateral oscura, `position: sticky`,
  `height: 100vh`) + `.story` (columna de contenido con scroll normal).
- **Móvil/tablet (<1000px):** `.side` se convierte en una barra superior compacta con un
  "stepper" de 4 círculos numerados (01–04) conectados por una línea, en vez de la lista vertical.
  El nombre del capítulo activo aparece como texto debajo de los círculos y se actualiza al hacer
  scroll (`#sideCurrent`, actualizado por el mismo `IntersectionObserver` que controla todo lo demás).

**Cuidado:** `.side-nav` tiene una regla base (para el modo móvil) y otra dentro de
`@media (min-width: 1000px)` (para desktop). Como ambas apuntan al mismo selector, cualquier
propiedad que no se re-declare explícitamente en la media query **se hereda** de la regla base.
Esto ya causó un bug (`align-items: center` del modo móvil "se colaba" al desktop y descentraba
el menú) — ver commit `d8be596`. Si se toca `.side-nav`, revisar ambos bloques.

### Colores por capítulo
Cada `<section class="chapter">` trae su propio color vía la variable inline `--stat-c`:

```html
<section class="chapter" id="componentes" style="--stat-c:color-mix(in srgb, #ffbf5e 68%, #000000 32%)">
```

- El título del capítulo (`.kick`), el numeral gigante de fondo (`.num-bg`) y las cifras resaltadas
  en el texto (`.stat`) usan `var(--stat-c)`.
- La barra lateral (círculo activo del nav, "cifra del capítulo") usa un color **separado**,
  definido en el objeto `chapters` dentro del `<script>` (campo `c`). Es a propósito: los colores
  de marca más claros (naranja `#ffbf5e`, olivo `#cde368`) se ven bien sobre el fondo oscuro de la
  barra lateral, pero son casi ilegibles como texto sobre el fondo claro de la página — por eso el
  texto usa una versión oscurecida con `color-mix()` y la barra lateral usa el color de marca puro.
  Ver commit `c45cb78` para la explicación completa con números de contraste.

Colores usados hasta ahora (paleta de marca):
| Sección | Color de marca (barra lateral) | Texto en página |
|---|---|---|
| 01 Evolución del PIB | `#1375f6` (azul) | mismo, ya tiene buen contraste |
| 02 Componentes del PIB | `#ffbf5e` (naranja) | `color-mix(in srgb, #ffbf5e 68%, #000 32%)` |
| 03 Análisis por industria | `#cde368` (olivo) | `color-mix(in srgb, #cde368 60%, #000 40%)` |
| 04 Conclusiones del BCE | `#7a26bc` (violeta) | mismo |

Otros colores de marca disponibles y ya usados en tarjetas/gráficos: `#890abf` (magenta),
`#cdb0ff` (lavanda), `#ddf086` (olivo claro), `#201061` (azul oscuro), `#18191f` (night),
`#050505` (dark knight).

### Tipografía
Figtree, autoalojada en `assets/fonts/` (no depende de Google Fonts). Se carga con
`<link rel="stylesheet" href="assets/fonts/figtree.css">`. Todos los pesos (300–900) están
disponibles vía variable font.

### Gráficos incrustados
Los 5 gráficos (`assets/01_crecimiento_interanual.html` … `assets/05_componentes_pib.html`) son
páginas HTML independientes, cargadas en `<iframe>`. **Cada una hace `postMessage` con su altura
real al cargar.** El documento padre necesita un listener que la reciba y ajuste `iframe.style.height`
— si falta, el iframe queda con la altura mínima del navegador (~150px) y el gráfico "no se ve".
Ya pasó una vez (commit `7197d1a`). El listener vive en el `<script>` de `editorial.html`,
buscar `chart-height`.

Los gráficos ya usan la paleta de marca (azul/magenta en vez de sus colores originales) y están
animados (barras que crecen, línea que se dibuja) — ver commit `93273eb` y siguientes.

### Animaciones
- **Conteo de cifras** (`.ticker .count`, hero): cuentan desde 0 al cargar. Función `animateCount`
  en el `<script>`. Respeta `prefers-reduced-motion`.
- **Aparición al hacer scroll** (`.reveal` / `.reveal.in`): fade + translateY, vía
  `IntersectionObserver`. Aplica a títulos, párrafos, gráficos, tarjetas.
- **Íconos de las tarjetas de sección 02** (`v2comp1–4.svg`, inlineados en el HTML): se dibujan
  trazo por trazo cuando la tarjeta aparece (técnica `stroke-dasharray`/`pathLength`).
- Todo lo animado respeta `prefers-reduced-motion: reduce`.

### Cosas que se probaron y se descartaron
Por si se repite la idea — ya se intentó y no gustó:
- **Mini-gráfico de líneas en la barra lateral:** quitado, muy angosto ahí (ver "no me gusta" del usuario).
- **Semicírculos/domos 3D flotando en la barra lateral:** implementado, corregido dos veces
  (geometría + recorte), pero finalmente el usuario pidió quitarlo — "está horrible, regresa a negro".
- **Silueta de cordillera (Andes) trazada con los datos reales del PIB, en la barra lateral:**
  idea propia sugerida como alternativa a lo anterior, implementada — el usuario también la quitó
  sin dar razón específica más allá de "regresa a negro". Conclusión: la barra lateral vacía se
  deja tal cual (negra, sin decoración) hasta nueva indicación explícita del usuario. No proponer
  llenarla de nuevo sin que lo pidan.
- **Logo de Keyword en la barra lateral oscura:** quitado por completo (duplicaba el lockup de
  logos que ya está en la sección de apertura, arriba a la derecha).

## 4. Cómo previsualizar en local

```bash
cd "/Users/sofiaandrade/Documents/Claude/Repositorio 2"
python3 -m http.server 8755 --bind 127.0.0.1
```

Abrir `http://127.0.0.1:8755/editorial.html` (o `index.html`). Usar un query string distinto
cada vez que se recarga (`?t=1`, `?t=2`…) para evitar caché del navegador durante pruebas.

## 5. Pendiente ahora mismo (donde se quedó la conversación)

El usuario pidió una versión "más infográfica" de los 4 ítems de la sección 02 (Componentes del
PIB): actualmente son filas de texto (ícono + cifra grande + párrafo). Se le propusieron dos
opciones y **todavía no eligió ninguna**:

1. Convertir las 4 filas en un solo gráfico de barras horizontales proporcionales (el largo de la
   barra = el porcentaje real), con ícono y cifra apoyados sobre la barra. Más "infográfico", y
   permitiría fusionarlo con el gráfico de barras que ya existe debajo (`05_componentes_pib.html`),
   evitando redundancia.
2. Mantener las 4 filas como están, pero escalar el tamaño del ícono y la cifra según la magnitud
   del dato (12,8% más grande que 1,7%). Más simple de construir, menos "infográfico".

**Próximo paso:** preguntarle al usuario cuál prefiere antes de tocar esa sección.

## 6. Buenas prácticas de esta sesión (seguir igual)

- Revisar cada cambio en el navegador (desktop y celular) antes de darlo por terminado.
- Hacer `git commit` local después de cada cambio con mensaje descriptivo. **No hacer `git push`
  sin que el usuario lo pida explícitamente** (ha sido su patrón constante en esta sesión).
- No crear archivos `.md` ni documentación salvo que el usuario lo pida (como este mismo archivo).
- Cuando el usuario pide algo ambiguo o parece un error de mi parte, revisar con medidas reales
  en el navegador (`getBoundingClientRect`, `getComputedStyle`) antes de asumir la causa.
