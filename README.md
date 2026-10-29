# Franken Panel

**Panel de administración autohospedado para Ubuntu Server: sitios, dominios, Nginx, SSL y despliegues desde el navegador.**

> 🔒 Código privado (proyecto en desarrollo activo desde marzo 2026).
> Demo disponible bajo solicitud · Revisión de código en vivo disponible para procesos de entrevista.
> Contacto: javierkarimandradetalamante@gmail.com

*EN: Self-hosted control panel for Ubuntu Server (Node.js/Fastify) that manages sites, domains, Nginx vhosts, Let's Encrypt SSL, GitHub-webhook deploys and a sandboxed web terminal, with RBAC auth, audit logs and real-time events over WebSockets.*

<!-- TODO: captura del dashboard / GIF de crear sitio → SSL → deploy -->

---

## ¿Qué problema resuelve?

Montar y mantener sitios en un VPS implica tocar Nginx, Certbot, DNS, permisos y servicios a mano. Franken Panel lo convierte en un flujo guiado desde el navegador, instalado con un solo script, y pensado para que una persona sin experiencia en Linux pueda operar el servidor sin romperlo.

## Funcionalidades

- **Sitios y dominios**: alta de sitios, verificación de dominio, registro de sitios desplegados.
- **Nginx**: crear, habilitar/deshabilitar, validar (`nginx -t`) y recargar vhosts sin downtime.
- **SSL**: emisión de certificados Let's Encrypt (Certbot) por sitio.
- **Despliegue continuo**: webhook de GitHub con verificación de firma HMAC-SHA256 y reconstrucción automática.
- **Gestor de archivos**: explorar, editar, buscar y subir medios dentro de rutas permitidas.
- **Terminal web**: ejecución de comandos con allowlist, directorio de trabajo persistente por sesión y autocompletado.
- **Exposición pública**: integración con Cloudflare Tunnel (`cloudflared`) + Nginx como capa de borde.
- **Eventos en tiempo real**: notificaciones de operaciones vía WebSockets.
- **Instalador y CLI**: instalador TUI con *preflight* del sistema, manifest de instalación y CLI global (`franken-panel status|start|stop|logs|update|uninstall`).

## Arquitectura

```text
                 Internet
                    │
         Cloudflare Tunnel (cloudflared)
                    │
                  Nginx  ──── vhosts de sitios + SSL (Certbot)
                    │
        ┌───────────┴────────────┐
        │  Franken Panel API     │  Node.js · Fastify 4 · 127.0.0.1:3001
        │  ├─ auth (RBAC, sesiones, auditoría)
        │  ├─ nginx / hosting (policy + intent router)
        │  ├─ deploy / auto-rebuild (webhooks GitHub)
        │  ├─ files / terminal (sandbox por allowlist)
        │  ├─ network / hardware-profile
        │  └─ events (WebSocket)
        └───────────┬────────────┘
                    │
          systemd · Ubuntu Server
```

- **~50 endpoints REST** organizados en módulos de dominio (`auth`, `nginx`, `hosting`, `deploy`, `files`, `terminal`, `network`, `events`…).
- El API escucha solo en loopback; toda exposición pasa por Nginx/túnel.
- Servicio gestionado por **systemd**; estado persistido en archivos JSON autogenerados en el primer arranque.

## Seguridad (decisiones de diseño)

| Área | Implementación |
|---|---|
| Contraseñas | PBKDF2-SHA512 (120k iteraciones) con salt por usuario y comparación en tiempo constante |
| Sesiones | Tokens Bearer aleatorios con expiración (12 h) y logout explícito |
| Autorización | RBAC por roles, *bootstrap* seguro del primer administrador |
| Auditoría | Registro de acciones administrativas consultable desde el panel |
| HTTP | `@fastify/helmet`, `@fastify/rate-limit`, CORS restringido |
| Webhooks | Verificación de firma `X-Hub-Signature-256` |
| Terminal | Allowlist de comandos que rechaza metacaracteres de shell; `execFile` sin shell intermedio |

## Calidad y operación

- Tests unitarios y *smoke tests* con el runner nativo de Node (`node --test`).
- Pruebas E2E de autenticación y QA extendido por script.
- Scripts de operación: *preflight* de Ubuntu, *go/no-go* de despliegue, autofix, diagnóstico del sistema, chequeo de seguridad y desinstalación limpia.

## Stack

`Node.js` · `Fastify 4` · `WebSockets` · `Bash` · `Nginx` · `Certbot` · `Cloudflare Tunnel` · `systemd` · `Ubuntu Server` · `JavaScript (vanilla) + CodeMirror` en el frontend

## Lo que aprendí

- Diseñar un API que ejecuta operaciones privilegiadas sin abrir la puerta a inyección de comandos.
- Modularizar un backend que nació monolítico, migrándolo por fases sin romper producción.
- Automatizar la instalación y el ciclo de vida de un servicio Linux de punta a punta.

---

**Autor:** Javier Karim Andrade Talamante — Backend Developer · Hermosillo, México
