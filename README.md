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

### Opción A — App Engine (la más simple, con HTTPS y dominio *.appspot.com)

```bash
cd web
gcloud app deploy
```

Requiere un proyecto de Google Cloud con App Engine habilitado
(`gcloud app create --region=us-central1` la primera vez).

### Opción B — Cloud Storage (bucket como sitio estático)

```bash
gsutil mb -l us-central1 gs://TU-BUCKET
gsutil -m rsync -r -x "app.yaml|README.md" web gs://TU-BUCKET
gsutil web set -m index.html gs://TU-BUCKET
gsutil iam ch allUsers:objectViewer gs://TU-BUCKET
```

Para HTTPS con dominio propio sobre un bucket hace falta un balanceador de carga;
si quieres dominio propio sin complicarte, App Engine o Firebase Hosting es más fácil.

## Pendientes de contenido

- Precios y nombres del pan/postre de la casa (`[Pan / postre de la casa]`, `[$ PRECIO]` en index.html).
- Horario (`[HORARIO]` en la sección "Visítanos").
- URL de Instagram (el botón apunta a `#`).
