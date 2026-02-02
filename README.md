# Cotizador SSO (Cloudflare Worker + D1 + GitHub Pages)

Este paquete trae **todo listo** para publicar:

- **Backend**: Cloudflare Worker (API) + **D1** (base de datos)
- **Frontend**: HTML estático (SPA) para **GitHub Pages** (carpeta `/docs`)
- **SSO**: Microsoft Entra ID (MSAL en el front + verificación de JWT en el worker)
- **Módulos**:
  - Cotización (manual y por carga masiva)
  - Matriz de trabajo + validación de colaboradores (edad, sexo, tipo)
  - Admin: carga de tarifas + gestión de descuentos + matriz
  - Convenios: registro + firma simple (hash/auditoría)

---

## 1) Publicar el Worker (con Wrangler)

### Requisitos
- Tener cuenta de Cloudflare
- Tener **Node.js** instalado

### Paso A — Entrar a la carpeta del worker
En tu computador, descomprime el ZIP y abre una terminal en:

```bash
cd worker
```

### Paso B — Instalar Wrangler (la herramienta de Cloudflare)
Opción recomendada (sin instalar global):

```bash
npm install
```

### Paso C — Iniciar sesión en Cloudflare
```bash
npx wrangler login
```

### Paso D — Crear base de datos D1
```bash
npx wrangler d1 create cotizador_db
```

Te devolverá un **database_id**. Ábrelo y **cópialo**.

### Paso E — Pegar el database_id en `wrangler.toml`
En `worker/wrangler.toml`, cambia:

```toml
database_id = "REEMPLAZA_ESTE_ID"
```

por el id real.

### Paso F — Crear tablas (schema)
Ejecuta el esquema **en la base remota**:

```bash
npx wrangler d1 execute cotizador_db --remote --file=./schema.sql
```

### Paso G — Publicar el Worker
```bash
npx wrangler deploy
```

Al finalizar, Wrangler te mostrará una URL tipo:

- `https://cotizador-sso-worker.<tu-subdominio>.workers.dev`

**Guárdala**: la necesitarás en el front.

---

## 2) Publicar el HTML con GitHub Pages

En GitHub, crea un repositorio (ej: `cotizador-sso-ui`) y sube el contenido de la carpeta:

- `/docs`  ✅ (esta carpeta ya incluye `index.html` y `.nojekyll`)

Luego:

1. Ve a tu repositorio en GitHub
2. `Settings` → `Pages`
3. En **Build and deployment**:
   - Source: **Deploy from a branch**
   - Branch: `main`
   - Folder: `/docs`
4. `Save`

Tu sitio quedará en una URL tipo:

- `https://TU-USUARIO.github.io/cotizador-sso-ui/`

---

## 3) Conectar el Front con el Worker + Entra

1. Abre el sitio en GitHub Pages
2. Clic en **Configurar**
3. Completa:
   - **Worker URL**: la de `wrangler deploy`
   - **Tenant ID / Client ID / Scope**: tus valores de Entra
4. Clic **Probar conexión**
5. Clic **Guardar**

Después, inicia sesión.

---

## 4) Notas de seguridad (muy importante)

- En el Worker, configura `ALLOWED_ORIGINS` para que **solo acepte** tu dominio de GitHub Pages.
- Usa un **scope** dedicado para este sistema (recomendado) y un **App Role** para admin.
- No subas datos sensibles al repositorio (GitHub Pages es público por defecto).

---

## 5) Qué hacer primero (orden recomendado)

1. Publicar Worker
2. Publicar GitHub Pages
3. Configurar `ALLOWED_ORIGINS` (en `wrangler.toml`)
4. Cargar tarifas (Admin → Tarifas)
5. Crear matriz (Admin → Matriz)
6. Crear descuentos (Admin → Descuentos)
7. Probar cotización manual y por carga masiva
