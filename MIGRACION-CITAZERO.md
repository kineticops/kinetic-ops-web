# Plan de migración a citazero.com

Este documento contiene el procedimiento completo para migrar de `kineticopss.com` a `citazero.com` SIN perder SEO y SIN downtime. **No ejecutar hasta tener el dominio comprado y DNS apuntando.**

---

## 🎯 Objetivo

- Tener citazero.com como dominio principal, sirviendo el mismo contenido.
- Mantener kineticopss.com operativo con **redirect 301 permanente** hacia citazero.com durante al menos 180 días.
- Transferir el 100% del SEO ganado en kineticopss.com al nuevo dominio.
- 0 segundos de downtime.

---

## 📋 FASE 1 — Antes de disparar el script (lo hace el usuario)

### 1.1. Comprar dominio
- En Hostinger, Namecheap o Cloudflare Registrar: comprar `citazero.com` (~10–15 €/año).
- Recomendado **Cloudflare Registrar** por precio a coste + DNS gratis + protección WHOIS incluida.

### 1.2. Configurar DNS
Añadir estos registros al dominio `citazero.com`:

| Tipo  | Nombre | Valor              | TTL   |
|-------|--------|--------------------|-------|
| A     | @      | `187.77.175.27`    | Auto  |
| A     | www    | `187.77.175.27`    | Auto  |

Esperar propagación (10 min – 2 h). Verificar con:
```bash
dig citazero.com +short
dig www.citazero.com +short
```
Ambos deben devolver `187.77.175.27`.

### 1.3. Avisar a Claude con: "dominio citazero.com comprado y DNS apuntando, lanza migración"

---

## ⚙️ FASE 2 — Configuración en EasyPanel (lo hace Claude)

Entrar al panel EasyPanel (http://187.77.175.27:3000) y en el servicio **kinetic_ops → nginx**:

1. **Domains → Add domain** → escribir `citazero.com` y `www.citazero.com`.
2. Activar **HTTPS (Let's Encrypt)** — EasyPanel gestiona la emisión automática del certificado SSL.
3. Mantener `kineticopss.com` y `www.kineticopss.com` también en la lista — siguen sirviendo el mismo contenido temporalmente.
4. Guardar y esperar 2 min a que Traefik emita certificados.

**Verificación:**
```bash
curl -sI https://citazero.com/ | head -5
# Debe devolver HTTP 200
```

---

## ⚙️ FASE 3 — Cambios en el código HTML (lo hace Claude)

### 3.1. Buscar y reemplazar en los archivos

Archivos a modificar:
- `index.html`
- `aviso-legal.html`
- `privacidad.html`
- `cookies.html`
- `sitemap.xml`
- `robots.txt`

Reemplazar en TODOS: `kineticopss.com` → `citazero.com`

Incluye: `canonical`, `og:url`, `twitter:image`, `og:image`, JSON-LD `@id`/`url`/`logo`, enlaces internos del aviso legal.

### 3.2. Añadir redirect 301 de kineticopss.com → citazero.com

El contenedor nginx sirve desde `/app/`. Para que kineticopss.com redirija al nuevo dominio SIN romper el SEO, configurar en nginx.conf del contenedor (dentro de `server { server_name kineticopss.com www.kineticopss.com; ... }`):

```nginx
server {
    listen 80;
    listen 443 ssl;
    server_name kineticopss.com www.kineticopss.com;
    return 301 https://citazero.com$request_uri;
}

server {
    listen 80;
    listen 443 ssl;
    server_name citazero.com www.citazero.com;
    root /app;
    index index.html;
    ...
}
```

**En EasyPanel esto se hace marcando `kineticopss.com` como "redirect" hacia `citazero.com`** en la configuración del dominio (más fácil que tocar nginx.conf).

Alternativa: añadir al principio de index.html (si no se puede tocar nginx):
```html
<script>
if (location.hostname === 'kineticopss.com' || location.hostname === 'www.kineticopss.com') {
  location.replace('https://citazero.com' + location.pathname + location.search);
}
</script>
```
⚠️ Esta alternativa NO transfiere SEO (Google no procesa JS redirects como 301). Es solo para el user. **Usar SIEMPRE la opción nginx/EasyPanel.**

### 3.3. Commit y deploy
```bash
cd "/Users/elduque/Desktop/proyecto vs code/kinetic-ops-web"
git add -A && git commit -m "migrate: switch canonical domain from kineticopss.com to citazero.com"
git push origin main

# Deploy a VPS (el mismo proceso de siempre)
expect -c "..."
```

---

## ⚙️ FASE 4 — Google Search Console (lo hace Claude + usuario)

### 4.1. Añadir propiedad `citazero.com`
- En GSC → Añadir propiedad → Prefijo de URL → `https://citazero.com/`
- Verificación automática: la etiqueta `<meta name="google-site-verification" content="6V_E5LZD3twezXOzuOiqXXXwp5sb3TBXWwcfauKsmtM">` ya está en el HTML (funcionará también en citazero.com).

### 4.2. Herramienta "Cambio de dirección"
Este es el paso CRÍTICO para que Google transfiera la autoridad:
1. En GSC → seleccionar propiedad **kineticopss.com** (la vieja)
2. Ajustes (engranaje abajo izquierda) → **Cambio de dirección**
3. Seleccionar dominio destino: `citazero.com`
4. Google valida que el redirect 301 funciona y que ambas propiedades están verificadas.
5. Confirmar → Google transfiere señales de ranking durante 180 días.

### 4.3. Enviar sitemap nuevo
En la propiedad `citazero.com`:
- Sitemaps → añadir `sitemap.xml` → enviar.
- Inspección de URL → `https://citazero.com/` → Solicitar indexación.

### 4.4. Mantener GSC de kineticopss.com activa 180+ días
No eliminar la propiedad vieja. Google la necesita para procesar el cambio.

---

## ⚙️ FASE 5 — Otros sitios donde cambiar el dominio

Checklist después de la migración:

- [ ] Bing Webmaster Tools → añadir citazero.com + herramienta "Site Move"
- [ ] Perfil de Google Business (si aplica) → actualizar URL
- [ ] Redes sociales (LinkedIn, Instagram, X) → actualizar bio
- [ ] Firma de email → cambiar a citazero.com
- [ ] Tarjetas de visita / material impreso
- [ ] Perfil WhatsApp Business → actualizar URL
- [ ] Menciones en otros sitios donde aparezca kineticopss.com (backlinks)

---

## ⏱️ Downtime esperado

**0 segundos.**

Durante las primeras 24-48h ambos dominios responderán a la vez. Después, kineticopss.com solo devolverá 301 → citazero.com. Los usuarios que escriban kineticopss.com en el navegador serán redirigidos automáticamente.

---

## 📊 Qué esperar en Google

- **Semana 1:** citazero.com empieza a aparecer en búsquedas de marca ("citazero").
- **Semana 2-4:** Google transfiere ranking progresivamente. Pérdida temporal de 10-15% de visibilidad (normal, se recupera).
- **Mes 2-3:** posiciones consolidadas en citazero.com. kineticopss.com prácticamente desaparecido de resultados.
- **Mes 6:** se puede empezar a considerar liberar kineticopss.com (aunque conviene renovarlo 1-2 años más como defensa).

---

## 🚨 Qué NO hacer

- ❌ No dejar caer kineticopss.com antes de 6 meses — rompes el flujo SEO.
- ❌ No usar redirect 302 (temporal) — tiene que ser 301 (permanente).
- ❌ No cambiar el contenido a la vez que el dominio — haz migración "limpia" primero, rediseño después si quieres.
- ❌ No mantener contenido duplicado (misma web en dos dominios sin canonical) — penaliza SEO.
- ❌ No olvidar la herramienta "Cambio de dirección" de GSC — es el 80% del SEO preservado.

---

## ✅ Checklist pre-lanzamiento

- [ ] citazero.com comprado
- [ ] DNS apunta a 187.77.175.27
- [ ] `dig citazero.com +short` devuelve la IP correcta
- [ ] EasyPanel tiene el dominio configurado con SSL
- [ ] HTML actualizado con nuevo dominio (Claude lo hace)
- [ ] Redirect 301 activo en kineticopss.com (Claude lo hace)
- [ ] GSC propiedad citazero.com verificada (Claude lo hace)
- [ ] GSC herramienta "Cambio de dirección" ejecutada (usuario lo confirma)
- [ ] Sitemap enviado en nueva propiedad (Claude lo hace)
