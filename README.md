# Portal de Solicitudes — Guía de despliegue

## Archivos del proyecto

```
formulario-dinamico/
├── index.html      ← formulario completo (frontend + lógica)
├── vercel.json     ← configuración de Vercel
└── README.md       ← esta guía
```

---

## Paso 1 — Configura n8n (antes de desplegar)

En `index.html`, edita el bloque `window.CONFIG` (~línea 320):

```js
window.CONFIG = {
  N8N_TOPICS_URL:    'https://TU-N8N.com/webhook/form-topics',
  N8N_QUESTIONS_URL: 'https://TU-N8N.com/webhook/form-questions',
  N8N_SUBMIT_URL:    'https://TU-N8N.com/webhook/form-submit',
  DEMO_MODE: false,   // ← cambiar a false en producción
};
```

### Webhooks de n8n que debes crear

| Webhook | Método | Devuelve |
|---|---|---|
| `/webhook/form-topics` | GET | `[{ id, name }]` |
| `/webhook/form-questions` | GET `?topicId=X` | `[{ id, label, tipo, obligatorio }]` |
| `/webhook/form-submit` | POST | `{ ok: true }` |

### Tipos de campo válidos en Monday

La columna `tipo_campo` de tus sub-ítems puede tener estos valores:

| Valor | Renderiza como |
|---|---|
| `texto` | Input de texto |
| `numero` | Input numérico |
| `fecha` | Date picker |
| `textarea` | Área de texto largo |
| `checkbox` | Toggle sí/no |
| `select` | Dropdown (incluir columna `opciones`) |

---

## Paso 2 — Sube a GitHub

```bash
# Crea un repositorio nuevo en github.com, luego:
git init
git add .
git commit -m "Portal de solicitudes v1"
git remote add origin https://github.com/TU_USUARIO/portal-solicitudes.git
git push -u origin main
```

---

## Paso 3 — Despliega en Vercel

### Opción A — Desde la web (recomendado para primera vez)

1. Ir a **vercel.com** → iniciar sesión con tu cuenta de GitHub
2. Clic en **"Add New Project"**
3. Seleccionar el repositorio `portal-solicitudes`
4. Dejar todo por defecto → clic en **"Deploy"**
5. En ~30 segundos tendrás una URL como `portal-solicitudes.vercel.app`

### Opción B — CLI de Vercel

```bash
npm install -g vercel
vercel login
vercel --prod
```

---

## Paso 4 — Variable de entorno (opcional, más seguro)

Si quieres ocultar las URLs de n8n del código fuente:

1. En Vercel → tu proyecto → **Settings → Environment Variables**
2. Agregar:
   - `N8N_TOPICS_URL` = tu URL
   - `N8N_QUESTIONS_URL` = tu URL
   - `N8N_SUBMIT_URL` = tu URL
3. Luego en `index.html` leer con `process.env` (requiere convertir el proyecto a Next.js)

Para esta versión estática, las URLs en `CONFIG` son suficientes.

---

## Paso 5 — Conecta con Slack

En n8n, el workflow del comando `/solicitud` debe responder con un mensaje que incluya el link a tu URL de Vercel:

```json
{
  "response_type": "ephemeral",
  "text": "👋 Aquí puedes crear tu solicitud:",
  "attachments": [{
    "fallback": "Abrir portal",
    "actions": [{
      "type": "button",
      "text": "Abrir formulario →",
      "url": "https://portal-solicitudes.vercel.app"
    }]
  }]
}
```

---

## Actualizaciones futuras

Cada `git push` al repositorio redespliega automáticamente en Vercel. No se necesita hacer nada más.
