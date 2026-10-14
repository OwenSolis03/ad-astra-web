# Ad Astra — Sitio web

Landing page estática de Ad Astra (Coffee & More), basada en el diseño del canvas
"Ad Astra — Sitio web". No tiene build ni dependencias: es HTML + CSS + SVG.

```
web/
├── index.html
├── css/styles.css
├── img/            logo, gato astronauta, isotipo y favicon (SVG)
└── app.yaml        configuración para Google App Engine
```

## Probar en local

```bash
cd web
python3 -m http.server 8000
# abrir http://localhost:8000
```

## Desplegar en Google Cloud

### Opción A — Firebase Hosting (recomendada: mismo proyecto que la app)

El POS ya sincroniza con Firebase bajo el proyecto "Ad Astra"; el sitio puede
vivir en ese mismo proyecto. Ya hay un `firebase.json` listo en esta carpeta.

```bash
npm install -g firebase-tools   # primera vez
firebase login                  # primera vez
cd web
firebase use --add              # elegir el proyecto Ad Astra, alias p.ej. "default"
firebase deploy --only hosting
```

Queda publicado en `https://TU-PROYECTO.web.app` con HTTPS y CDN. Para dominio
propio: consola de Firebase → Hosting → Agregar dominio personalizado.

### Opción B — App Engine (con HTTPS y dominio *.appspot.com)

```bash
cd web
gcloud app deploy
```

Requiere un proyecto de Google Cloud con App Engine habilitado
(`gcloud app create --region=us-central1` la primera vez).

### Opción C — Cloud Storage (bucket como sitio estático)

```bash
gsutil mb -l us-central1 gs://TU-BUCKET
gsutil -m rsync -r -x "app.yaml|README.md" web gs://TU-BUCKET
gsutil web set -m index.html gs://TU-BUCKET
gsutil iam ch allUsers:objectViewer gs://TU-BUCKET
```

Para HTTPS con dominio propio sobre un bucket hace falta un balanceador de carga;
si quieres dominio propio sin complicarte, App Engine o Firebase Hosting es más fácil.

## Caché

El HTML se sirve con `no-cache` (siempre fresco) y los archivos CSS/JS/SVG con
caché larga de 7 días. Por eso, al cambiar `css/styles.css`, `img/logo.svg`,
etc., hay que subir la versión del parámetro `?v=N` con el que `index.html`
los referencia — así todos los visitantes ven el cambio de inmediato.

## Pendientes de contenido

- Pan de la casa: retirado del sitio por ahora (no está disponible); reintegrar menú y
  tarjeta cuando vuelva. Copy aprobado por Edgar para ese momento: "Repostería hecha
  con calma, directo del horno a tu mano. No hay todos los días: te avisamos por
  Instagram un día antes y cuando se acaba, se acaba."
- Voz de la marca (cuestionario del 12/oct): sin "barista certificado", sin "recetas
  propias/de familia", sin "rico/delicioso", sin "estudiantes"; preferir "al momento",
  "del día", "calma", "local", "honesto", "real", "mezclas", "horneamos".
- Horario de jueves y viernes (por confirmar en la sección "Visítanos").

## Menú en vivo (desde el Moka POS)

`index.html` lee una vez al cargar el documento público `publico/menu` de Firestore,
que publica el POS tras cada venta o movimiento de inventario:

- `productos`: estatus de cada bebida; se pinta junto a las filas del menú cuyo nombre
  coincide con el del catálogo del POS.
- `sabores` y `leches`: arman la sección "Sabores y leches". Salen de las categorías
  **Jarabes y polvos** y **Leches** del inventario del POS; para que un insumo aparezca
  (o deje de aparecer) basta con cambiarle la categoría en Inventario → Editar. Los
  insumos que nunca se han capturado no se publican.

Sin datos (sin conexión o sin permiso), el menú se ve sin etiquetas y la sección de
sabores y leches muestra solo sus precios, con un "pregunta en el puesto" en lugar de la lista.

Precios: cada uno se escribe una sola vez en `index.html`. El del Latte saborizado va en su
fila del menú (la tarjeta de sabores lo copia) y el de la leche vegetal en el encabezado de la
tarjeta de Leches (las filas de cada leche lo copian). El sabor Moka no se anuncia: es el Mocha.
