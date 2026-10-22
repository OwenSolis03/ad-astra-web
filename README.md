# Ad Astra — Sitio web

Landing page estática de Ad Astra (Coffee & More), basada en el diseño del canvas
"Ad Astra — Sitio web". No tiene build ni dependencias: es HTML + CSS + SVG.

```
web/
├── index.html
├── privacidad.html aviso de privacidad
├── terminos.html   términos de los pedidos para recoger
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

## Menú en vivo (desde el Moka POS)

`index.html` lee una vez al cargar el documento público `publico/menu` de Firestore,
que publica el POS tras cada venta o movimiento de inventario:

- `productos`: estatus de cada bebida activa. Una bebida que el POS ya no publica
  (desactivada o borrada del catálogo) desaparece del menú de la página.
- `carta` (cuando el POS la publica): la lista completa de bebidas, con nombre, precio,
  categoría, temperaturas y descripción. Con ella la página arma el menú entero, en
  computadora y en celular: una bebida nueva aparece sola y una categoría nueva es una
  tarjeta más. Sin ella se usa la lista escrita en `index.html`.
- `descripciones`: textos que se capturan en el POS para los sabores (por nombre del
  insumo). Reemplazan a los de `DESCRIPCIONES_SABOR` en `index.html`.
- `sabores` y `leches`: arman la sección "Sabores y leches". Salen de las categorías
  **Jarabes y polvos** y **Leches** del inventario del POS; para que un insumo aparezca
  (o deje de aparecer) basta con cambiarle la categoría en Inventario → Editar. Los
  insumos que nunca se han capturado no se publican.
- `pedidosEnLinea`: `{ activo, ocupados }`. `ocupados` son las bebidas en línea ya apartadas
  por turno de 15 min, con la hora de Hermosillo, por ejemplo `{ "2026-10-22 10:15": 2 }`.
  Cada turno admite 2 bebidas en línea; las horas sin lugar no se ofrecen. Con `activo: false`
  (el POS pausó los pedidos en línea) no se puede pagar. Sin el campo, todos los turnos se ven libres.

Reglas de los pedidos para recoger (`TOPE`, `CUPO_TURNO`, `TURNO` y `GUARDADO` al inicio del
script del pedido en `index.html`): máximo 4 bebidas por pedido, 2 bebidas en línea por turno
de 15 min, y el pedido se guarda 15 min después de su hora. Un pedido de 3 o 4 bebidas ocupa su
turno y el anterior. La página solo las muestra: el POS tiene que volver a revisarlas al crear el
pedido, porque cualquiera puede saltarse lo que hace la página.

## Pedidos para recoger (con el Moka POS)

«Pagar» manda el pedido a la función `crearPedido` del Moka POS (`functions/` en ese repositorio):

```json
{ "items": [{ "nombre": "Latte", "detalle": "Frío · Leche de avena · Jarabe de Vainilla", "cantidad": 2 }],
  "nombre": "Owen", "recoger": "2026-10-22 10:45", "turnos": [{ "turno": "2026-10-22 10:45", "bebidas": 2 }] }
```

La función recalcula los precios con el catálogo del POS (los de la página no se mandan), revisa tope,
cupo, horario y lo agotado, aparta el turno y devuelve `{ id, init_point }`: la página redirige a Checkout Pro.
Si responde un error (`{ error, mensaje }`), el mensaje se muestra junto al botón; con 409 o 503 se vuelve a leer
`pedidosEnLinea` para quitar las horas sin lugar. Contrato completo: `docs/pedidos-en-linea.md` del Moka POS.

Mercado Pago regresa a `?pedido=<número>&status=…`. El pago no lo confirma la URL sino el webhook: el seguimiento
lee cada 15 s (cada 4 s mientras está «pendiente») el documento público `pedidos/<número>` con `estado`
(`pendiente`, `pagado`, `preparando`, `listo`, `entregado`, `no_recogido`, `vencido`, `cancelado` o `error`),
`codigo`, `recoger` e `inicial`. Solo se puede leer sabiendo el número; listar no. El enlace del pedido sirve
en cualquier navegador: sin el pedido guardado, se arma con lo público (inicial en lugar del nombre). Al pasar a
«Listo», la página vibra, suena, cambia el título de la pestaña y, con permiso, manda una notificación.

- **App Check**: si la página define `window.AdAstraAppCheck` (una función que devuelve una promesa con el token,
  por ejemplo con el SDK de Firebase y reCAPTCHA Enterprise), se manda en `X-Firebase-AppCheck`. Mientras no
  esté, la función debe correr con `EXIGIR_APP_CHECK=false`.
- **Encabezados de seguridad** (`firebase.json`): `Content-Security-Policy` (scripts y estilos solo de este sitio,
  fuentes de Google Fonts, conexiones solo a Firestore y a la función de pedidos, sin iframes), `nosniff`,
  `X-Frame-Options`, `Referrer-Policy` y `Permissions-Policy`. Si se agrega otro servicio (App Check con reCAPTCHA,
  analítica, otro dominio de la función), hay que permitirlo en la CSP o el navegador lo bloquea. En App Engine
  van los mismos básicos en `app.yaml` y `.gcloudignore` deja fuera el README y la configuración.
- La página solo redirige a `init_point` si es de `mercadopago.com(.mx)` (o del emulador, en local).
- **Probar en local**: con este repositorio junto al del Moka POS (`…/Moka_POS` y `…/ad-astra-web`), correr
  `pedidos-local.bat` en el Moka POS y abrir `http://127.0.0.1:5000/#pedir`. Servida en el puerto 5000, la
  página usa los emuladores (`window.AdAstraNube` al inicio del primer script) y Mercado Pago simulado; el POS
  queda en `http://127.0.0.1:5050/pos/`. Con `python3 -m http.server 8000` sigue leyendo producción.
- El aviso de privacidad (`privacidad.html`) y los términos (`terminos.html`) describen lo que hace el sistema;
  si cambia qué datos se piden o la política de cancelación, hay que actualizarlos (y su fecha).

Sin datos (sin conexión o sin permiso), el menú se ve sin etiquetas y la sección de
sabores y leches muestra solo sus precios, con un "pregunta en el puesto" en lugar de la lista.

Precios: cada uno se escribe una sola vez en `index.html`. El del Latte saborizado va en su
fila del menú (la tarjeta de sabores lo copia) y el de la leche vegetal en el encabezado de la
tarjeta de Leches (las filas de cada leche lo copian). Con `carta`, los precios de las bebidas
son los del POS. El sabor Moka solo se oculta mientras el Mocha esté en el menú.
