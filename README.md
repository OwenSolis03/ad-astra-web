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

## Pendientes de contenido

- Pan de la casa: retirado del sitio por ahora (no está disponible); reintegrar menú y
  tarjeta cuando vuelva.
- Horario de jueves y viernes (por confirmar en la sección "Visítanos").
