# Cuentas Nacionales 2026 — Handoff de diseño

Documento de traspaso para continuar el trabajo de diseño en otra cuenta/sesión de Claude.
**`editorial.html` es la versión de referencia** — es el diseño sobre el que hay que seguir
trabajando e iterando. `index.html` es el diseño anterior, ya publicado; no se toca salvo
pedido explícito (ver sección 2).

Última actualización: sesión que terminó en el commit `16c33a6` + los cambios descritos en la
sección 3, que al momento de escribir esto **siguen sin commitear** (ver sección de git más abajo).

## 1. Qué es este proyecto

Landing page del informe "La economía ecuatoriana creció 2,1% en el primer trimestre de 2026"
(cifras preliminares del Banco Central del Ecuador, BCE). Es un sitio estático (HTML/CSS/JS
puro, sin framework ni build step), con marca Keyword / KeyEconomics.

- **Repo GitHub:** https://github.com/disenokiwi/Cuentas-Nacionales-2026 (rama `main`)
- **Deploy Vercel:** https://cuentas-nacionales-2026.vercel.app (auto-deploy al hacer push a `main`)
- **Carpeta local:** `/Users/sofiaandrade/Documents/Claude/Repositorio 2/`
- **Fuente del texto trimestral (1T 2026):** un documento de Notion del que se extrajo el texto
  original (BCE, Cuentas Nacionales Trimestrales).
- **Fuente del texto anual 2025** (agregado como contexto adicional en las secciones 02 y 03,
  ver sección 3.5): otro documento de Notion — "En 2025, el PIB del Ecuador registró un
  crecimiento de 3,7%" (BCE, Cuentas Nacionales Anuales).

### ⚠️ Regla de contenido (la más importante)

**No se puede cambiar ninguna cifra ni palabra del texto del informe.** Sí se puede:
- resaltar cifras dentro del texto (negrita, color),
- reordenar/reestructurar visualmente el mismo contenido,
- cambiar el diseño, colores, tipografía, animaciones, íconos, fotos.

Si en algún momento hace falta un dato que no está en el texto original, no inventarlo —
preguntar al usuario o volver a la fuente de Notion.

**Importante:** este sitio mezcla dos períodos de datos distintos a propósito (1T 2026
trimestral = el informe principal; 2025 anual = contexto adicional en secciones 02 y 03,
claramente separado por un divisor `.annual-divider` con la etiqueta "Contexto anual ·
Cuentas Nacionales 2025"). **Nunca fusionar o confundir ambas cifras** — son fuentes y
períodos distintos del BCE.

### ⚠️ Estado de git — hay cambios sin commitear

```bash
cd "/Users/sofiaandrade/Documents/Claude/Repositorio 2"
git status --short   # ver qué falta
```

Al momento de escribir esto:
- El repo local tiene **19 commits por delante de `origin/main`** (nada de esto se subió a
  GitHub todavía — el usuario no lo ha pedido en ninguna sesión).
- Además hay trabajo de **esta sesión sin commitear**: `editorial.html` modificado, más 7 fotos
  nuevas en `assets/` (`fondo-ejecutivo.jpg` y las `sec-*.jpg` de industrias) y una carpeta
  `assets/originals/` con los respaldos sin comprimir (ver sección 3.4).

No hacer `git push` sin que el usuario lo pida explícitamente — ha sido su patrón constante.
Si se retoma el trabajo, probablemente haya que hacer un `git add` + `git commit` de estos
cambios primero (preguntar al usuario el mensaje/alcance del commit).

## 2. Dos diseños en el mismo repo

| Archivo | Qué es | Estado |
|---|---|---|
| `index.html` | Diseño original, ya publicado y aprobado. Secciones apiladas con menú superior. | Live en Vercel. **No tocar sin que el usuario lo pida.** |
| `editorial.html` | Diseño "editorial": barra lateral fija oscura + columna de historia con scroll. **Es el diseño activo, el que hay que seguir usando como base.** | En progreso, no está enlazado desde ningún lado todavía. Al hacer push, queda accesible en `https://cuentas-nacionales-2026.vercel.app/editorial.html`. |

Ambos comparten la carpeta `assets/` (fotos, gráficos, logos, fuente).

## 3. `editorial.html` — arquitectura y decisiones

### 3.1 Layout
- **Desktop (≥1000px):** grid de 2 columnas — `.side` (barra lateral oscura, `position: sticky`,
  `height: 100vh`) + `.story` (columna de contenido con scroll normal).
- **Móvil/tablet (<1000px):** `.side` se convierte en una barra superior compacta con un
  "stepper" de 4 círculos numerados (01–04) conectados por una línea. El nombre del capítulo
  activo aparece como texto debajo de los círculos y se actualiza al hacer scroll
  (`#sideCurrent`, vía el mismo `IntersectionObserver` que controla todo lo demás).

**Cuidado:** `.side-nav` tiene una regla base (móvil) y otra dentro de
`@media (min-width: 1000px)` (desktop). Como ambas apuntan al mismo selector, cualquier
propiedad no re-declarada en la media query **se hereda** de la base. Ya causó un bug
(`align-items: center` del móvil "se colaba" al desktop) — ver commit `d8be596`.

**Logos de apertura (`.opening-logos`):** se posicionan con `position:absolute` respecto a
`.story` (todo el ancho de la columna), **no** respecto a `.opening` (que tiene
`max-width:920px`). Si se mueve este posicionamiento de vuelta a `.opening`, los logos quedan
"atrapados" muy a la izquierda en pantallas anchas en vez de pegar al borde derecho real de
la página.

### 3.2 Colores por capítulo
Cada `<section class="chapter">` trae su propio color vía la variable inline `--stat-c`:

```html
<section class="chapter" id="componentes" style="--stat-c:color-mix(in srgb, #ffbf5e 68%, #000000 32%)">
```

- El título del capítulo (`.kick`), el numeral gigante de fondo (`.num-bg`) y las cifras
  resaltadas en el texto (`.stat`) usan `var(--stat-c)`.
- La barra lateral (círculo activo del nav, "cifra del capítulo") usa un color **separado**,
  definido en el objeto `chapters` dentro del `<script>` (campo `c`). Es a propósito: los
  colores de marca más claros (naranja `#ffbf5e`, olivo `#cde368`) se ven bien sobre el fondo
  oscuro de la barra lateral, pero son casi ilegibles como texto sobre el fondo claro de la
  página — por eso el texto usa una versión oscurecida con `color-mix()` y la barra lateral usa
  el color de marca puro. Ver commit `c45cb78` para la explicación completa con contraste.
- El mismo patrón de `color-mix()` se reutilizó dentro de las tarjetas nuevas (galería de
  industrias, conclusiones) para asignar un acento de color por ítem sin perder legibilidad.

Colores usados hasta ahora (paleta de marca):
| Sección / uso | Color de marca | Texto en página (si aplica) |
|---|---|---|
| 01 Evolución del PIB | `#1375f6` (azul) | mismo, ya tiene buen contraste |
| 02 Componentes del PIB | `#ffbf5e` (naranja) | `color-mix(in srgb, #ffbf5e 68%, #000 32%)` |
| 03 Análisis por industria | `#cde368` (olivo) | `color-mix(in srgb, #cde368 60%, #000 40%)` |
| 04 Conclusiones del BCE | `#7a26bc` (violeta) | mismo |

Otros colores de marca disponibles y ya usados en tarjetas/gráficos/fotos: `#890abf` (magenta),
`#cdb0ff` (lavanda), `#ddf086` (olivo claro), `#201061` (azul oscuro), `#55575f` / `#4A5876`
(grises neutros para industrias sin un color de marca obvio), `#18191f` (night), `#050505`
(dark knight).

### 3.3 Tipografía
Figtree, autoalojada en `assets/fonts/` (no depende de Google Fonts). Se carga con
`<link rel="stylesheet" href="assets/fonts/figtree.css">`. Todos los pesos (300–900)
disponibles vía variable font.

### 3.4 Fotos — convención de nombres y compresión

**Todas las fotos pesadas se comprimen antes de dejarlas en `assets/`.** Las cámaras/exports
modernos entregan JPGs de 6000–9000px de ancho (8–22 MB cada uno); en el sitio nunca se
muestran a más de ~900px, así que ese peso es puro desperdicio. Proceso usado (con `sips`,
viene instalado en macOS, no requiere instalar nada):

```bash
# 1. Respaldar el original sin comprimir (nunca perder el original)
mkdir -p assets/originals
cp "assets/foto.jpg" assets/originals/

# 2. Redimensionar (máx. 1800px en el lado más largo) y comprimir (calidad 78)
sips -Z 1800 -s formatOptions 78 "assets/foto.jpg" --out "assets/foto.jpg"
```

Resultado real de esta sesión: 7 fotos de industrias, de **95 MB → 2,6 MB en total**, sin
pérdida de calidad perceptible. `assets/originals/` está en `.gitignore` — los originales
quedan solo en el filesystem local, no se suben al repo.

**Nombres de archivo reales usados en `editorial.html` (sección de industrias)** — el usuario
sube las fotos con nombres propios, no un slug estandarizado; hay que respetar exactamente lo
que sube (mayúsculas, tildes, espacios incluidos) en el `url('assets/...')`:

| Industria | Archivo real en `assets/` |
|---|---|
| Comercio | `sec-Comercio.jpg` |
| Actividades financieras y de seguros | `sec-Actividades financieras.jpg` |
| Manufactura de productos alimenticios | `sec-manufactura alimentos.jpg` |
| Agricultura, ganadería y silvicultura | `sec-agricultura.jpg` |
| Manufactura de productos no alimenticios | `sec-manufactura productos no alimenticios.jpg` |
| Construcción | `sec-construcción.jpg` |
| Transporte y almacenamiento | `sec-transporte.jpg` |

Si se agrega una industria nueva o se reemplaza una foto, **preguntar el nombre exacto del
archivo que subió el usuario** en vez de asumir un slug — ya pasó dos veces en esta sesión que
el nombre propuesto no coincidía con el que realmente se subió.

`assets/fondo-ejecutivo.jpg` es la foto de fondo del panel lateral (ver 3.6). Las fotos
`assets/photos/conclusion-1.jpg` … `conclusion-4.jpg` ya **no se usan** (se quitaron las fotos
de la sección de Conclusiones, ver 3.7) — se dejaron en el repo sin borrar por si se reusan.

### 3.5 Contexto anual 2025 (secciones 02 y 03)

Además del dato trimestral (1T 2026, el hilo principal del informe), se agregó un bloque de
contexto con datos **anuales 2025**, sacados de un segundo documento de Notino del BCE. Patrón
usado en ambas secciones:

```html
<div class="annual-divider"><span class="ad-line"></span><span class="ad-label">Contexto anual · Cuentas Nacionales 2025</span><span class="ad-line"></span></div>
<p class="annual-intro">...</p>
```

- **Sección 02 (Componentes del PIB):** debajo del divisor, 5 tarjetas `<details>` (clase
  `.e-terms`/`.e-term`, la misma que ya usaban las definiciones de PIB Real/Nominal en la
  sección 01) con el detalle de cada componente del gasto en 2025.
- **Sección 03 (Industrias):** debajo del divisor, la galería de fotos por industria (ver 3.6),
  en vez de tarjetas — el usuario pidió explícitamente que esta info fuera "parte de la
  diagramación", no tarjetas.

**Bug de `.e-terms` y cómo se arregló:** el grid `.e-terms` no tenía `align-items: start`, así
que por default CSS Grid usa `stretch` — al abrir una tarjeta `<details>`, la de al lado en la
misma fila se estiraba para igualar la altura, mostrando una caja vacía (parecía que se había
abierto sola). Se arregló agregando `align-items: start` a `.e-terms`. Si se crea un nuevo grid
de `<details>`, replicar ese `align-items: start` desde el inicio.

### 3.6 Galería de industrias (`.ind-gallery` / `.ind-row`)

Filas alternadas foto/texto (izquierda-derecha, derecha-izquierda con `.rev`), una por
industria, dentro de la sección 03. Cada fila:

```html
<div class="ind-row [rev]" style="--ind-c:#1375f6">
  <div class="ind-photo" style="background-image:url('assets/sec-Comercio.jpg')"><span class="ind-photo-num">01 / 07</span></div>
  <div class="ind-text">
    <span class="ind-eyebrow">Comercio</span>
    <div class="ind-stat">+5,2<small>%</small></div>
    <p class="ind-body">...</p>
    <ul class="ind-facts"><li><strong>+10,8%</strong> ventas del sector (SRI)</li>...</ul>
  </div>
</div>
```

- `.ind-photo` usa `background-image` (no `<img>`) con un color de fondo de respaldo
  (`color-mix` con `--ind-c`) — si la foto todavía no existe, se ve como un bloque de color
  liso, no como una imagen rota.
- `--ind-c` es el acento de color de esa fila (ver tabla de colores en 3.2); se hereda a
  `.ind-eyebrow`, `.ind-stat` y los bordes de `.ind-facts` vía `var(--ind-c, var(--blue))`.
- Este patrón **reemplazó** un diseño anterior de tarjetas `<details>` (como las de 3.5) porque
  el usuario pidió explícitamente que la info de industrias fuera parte del diseño editorial,
  no tarjetas desplegables.

### 3.7 Conclusiones (`.concl-stack` / `.concl-item`) — iteración de diseño

La sección 04 pasó por **tres diseños distintos** en esta sesión antes de llegar al actual.
Si se propone rediseñarla de nuevo, repasar esto primero para no repetir intentos descartados:

1. **Filas alternadas foto/texto** (`.gallery`/`.g-row`, el diseño original) — descartado
   porque quedó visualmente idéntico al nuevo patrón de galería de industrias (3.6).
2. **Mosaico de tarjetas foto con texto superpuesto** (una destacada + 3 en grid) — el usuario
   lo rechazó sin dar razones específicas ("no me gusta :(").
3. **Grid 2×2 de tarjetas sin foto, con bordes** — mejor recibido pero el usuario pidió que las
   4 fueran visualmente idénticas (ya lo eran) y sobre todo que **no se recortara texto**
   (la primera iteración de este diseño sí abrevió algunas frases — nunca hacer eso, ver la
   regla de contenido en la sección 1).
4. **Diseño final, el que queda ahora:** lista editorial apilada a ancho completo. Cada
   conclusión es una fila con un número gigante a color (01–04) a la izquierda y el texto
   completo a la derecha, separadas por líneas finas. Sin fotos. Ajustes finos pedidos después:
   - Sin línea (`border-top`) justo debajo del título "Conclusiones del BCE".
   - El asterisco de marca (`.ci-mark`) va **en línea, al inicio del párrafo** (dentro de
     `.ci-text`, junto al `<p>`) — no como marca de fondo grande y semitransparente superpuesta
     al número, que fue el problema de una iteración intermedia.
   - Sin línea duplicada entre el último ítem (04) y la nota de fuente — se logró quitando el
     `border-bottom` del último `.concl-item` con `:last-child` (la nota de fuente ya trae su
     propio `border-top`, así que solo queda una línea).
   - El asterisco de cada conclusión gira lentamente (`animation: markSpin 14s linear infinite`,
     reutilizando el keyframe que ya existía para la marca del sidebar), respetando
     `prefers-reduced-motion`.

### 3.8 Fondo del panel lateral (`.side::before`)

El panel lateral (`.side`) tiene una foto de fondo muy sutil, a modo de textura:

```css
.side { position: relative; background: var(--night); ... }
.side::before {
  content: ""; position: absolute; inset: 0; z-index: -1;
  background: url('assets/fondo-ejecutivo.jpg') center / cover no-repeat;
  opacity: 0.2;
}
```

`z-index: -1` es necesario para que quede detrás del contenido del sidebar (nav, cifra del
capítulo, etc.) sin tener que tocar el `z-index` de cada hijo individualmente. Si el 20% de
opacidad se ve muy tenue o muy fuerte al probarlo, es el primer valor a ajustar — el usuario
lo pidió explícitamente como "prueba" (valor no cerrado del todo).

### 3.9 Gráficos incrustados

Los 5 gráficos (`assets/01_crecimiento_interanual.html` … `assets/05_componentes_pib.html`) son
páginas HTML independientes, cargadas en `<iframe>`, y **siguen mostrando solo datos
trimestrales (1T 2026)** — no se tocaron para el contexto anual 2025 (ver 3.5), que se agregó
como texto narrativo en vez de rehacer los SVG. Cada iframe hace `postMessage` con su altura
real al cargar; el documento padre necesita un listener que la reciba y ajuste
`iframe.style.height` — si falta, el iframe queda con la altura mínima del navegador (~150px).
Ya pasó una vez (commit `7197d1a`). El listener vive en el `<script>` de `editorial.html`,
buscar `chart-height`.

### 3.10 Animaciones
- **Conteo de cifras** (`.ticker .count`, hero): cuentan desde 0 al cargar (`animateCount`).
- **Aparición al hacer scroll** (`.reveal`/`.reveal.in`): fade + translateY vía
  `IntersectionObserver`. Aplica a títulos, párrafos, gráficos, `.stat-row`, `.ind-row` y
  `.concl-item` — **si se agrega una sección/patrón nuevo con este mismo efecto, sumarlo al
  selector `querySelectorAll` del bloque "reveal on scroll"**, si no, el elemento no tiene la
  clase `.reveal` y no anima.
- **Íconos de sección 02** (`v2comp1–4.svg`): se dibujan trazo por trazo al aparecer
  (`stroke-dasharray`/`pathLength`).
- **Marca-asterisco girando** (`markSpin`, 14s lineal infinito): usado en `.side-mark` (barra
  lateral) y en `.ci-mark` (conclusiones, ver 3.7).
- Todo lo animado respeta `prefers-reduced-motion: reduce`.

### 3.11 Cosas que se probaron y se descartaron
Por si se repite la idea — ya se intentó y no gustó:
- **Mini-gráfico de líneas en la barra lateral:** quitado, muy angosto ahí.
- **Semicírculos/domos 3D flotando en la barra lateral:** implementado, corregido dos veces,
  finalmente quitado — "está horrible, regresa a negro".
- **Silueta de cordillera (Andes) con los datos del PIB en la barra lateral:** implementado,
  también quitado sin razón específica más allá de "regresa a negro". La barra lateral vacía
  (negra, sin decoración salvo el fondo sutil de 3.8) se deja así hasta nueva indicación.
- **Logo de Keyword en la barra lateral oscura:** quitado (duplicaba el lockup de la apertura).
- **Video en loop sobre la barra lateral:** agregado y luego quitado en la misma sesión.
- Ver 3.7 para los 3 diseños descartados de la sección de Conclusiones.

## 4. Cómo previsualizar en local

```bash
cd "/Users/sofiaandrade/Documents/Claude/Repositorio 2"
python3 -m http.server 8755 --bind 127.0.0.1
```

Abrir `http://127.0.0.1:8755/editorial.html`. Usar un query string distinto cada vez que se
recarga (`?t=1`, `?t=2`…) para evitar caché del navegador durante pruebas.

## 5. Pendiente / sin resolver

- **Sección 02, las 4 filas de componentes (ícono + cifra + párrafo, datos trimestrales):** en
  algún momento se propuso hacerlas "más infográficas" (barra horizontal proporcional, o
  escalar el tamaño según la magnitud del dato). Nunca se decidió y nunca se tocó — sigue como
  estaba originalmente. No asumir una preferencia; preguntar antes de tocarlo.
- **Compresión de fotos:** solo se comprimieron las 7 `sec-*.jpg` de industrias y ya venían
  livianas las `conclusion-*.jpg`. Si se suben fotos nuevas y pesadas (ej. para `index.html`,
  u otra sección), repetir el proceso de 3.4 — no asumir que ya están optimizadas.

## 6. Buenas prácticas de esta sesión (seguir igual)

- Revisar cada cambio (balance de etiquetas HTML al menos con un chequeo rápido tipo
  `grep`/Python, idealmente también en el navegador) antes de darlo por terminado.
- **No hacer `git commit` ni `git push` sin que el usuario lo pida explícitamente** — en esta
  sesión se acumularon varios cambios sin commitear a propósito, ver sección de git arriba.
- No crear archivos `.md` ni documentación salvo que el usuario lo pida (como este mismo
  archivo, y el `.gitignore` que se tocó junto con él).
- Cuando el usuario pide algo ambiguo o señala un error, verificar con medidas reales
  (`getBoundingClientRect`, `getComputedStyle`, o leer el CSS con cuidado) antes de asumir la
  causa — varios bugs de esta sesión (grid `stretch`, líneas duplicadas, logos mal
  posicionados) tenían una causa CSS concreta y no eran "gusto" del usuario.
- Si un rediseño no convence al usuario, no reintentar variaciones del mismo concepto — cambiar
  de enfoque (ver 3.7, el caso de Conclusiones, como ejemplo de esto funcionando bien al tercer
  intento distinto).
- Al recibir fotos del usuario, no asumir un nombre de archivo "limpio" — confirmar el nombre
  real con `ls assets/` antes de referenciarlo en el HTML.
