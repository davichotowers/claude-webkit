# Maderas del Cármen — Traspaso de Proyecto

> Documento de continuidad para retomar este proyecto en una sesión/Project nuevo de Claude Code, sin arrastrar el ruido de la sesión anterior (pruebas descartadas, la investigación de seguridad de MCP, etc).
>
> **Repo:** `davichotowers/claude-webkit` — rama `claude/landing-page-onboarding-d9c4ig`
> **Framework:** Claude Web Builder (Tododeia) — todo lo de `docs/`, `.claude/skills/`, `CLAUDE.md` en la raíz se mantiene tal cual, sin cambios.

---

## El proyecto

**Maderas del Cármen** — catálogo de video y fotografía aérea de la Sierra del Carmen, corredor de conservación entre Coahuila y Texas. Sin fines de lucro, sin licenciar el material — el objetivo es solo apreciar/explorar el área natural.

- **Audiencia:** profesionales de cine, naturalistas, ONGs de conservación, productoras, público en general.
- **Idioma:** español (default), inglés automático si el navegador del visitante no está en español.
- **Sin menú de navegación**, sin formulario de contacto, sin redes sociales por ahora.
- **Deploy:** privado hasta aprobación final. Dominio propio del cliente — se conecta hasta el final.
- **Fuente del video:** helicóptero con gimbal estabilizado de cine (Cinema Pro Plus) — nunca decir "dron".
- **Audio:** los clips van **sin audio**. Un ícono de sonido controla un ambiente separado (pendiente de definir esa pista).

---

## Sistema de diseño — APROBADO, no volver a proponer alternativas

| Elemento | Decisión final |
|---|---|
| Paleta | "Cañón a mediodía" — roca caliza dorada bajo sol duro, cielo en bruma azul-gris, sombra profunda. **No** es la paleta nocturna con glow dorado (esa fue la v1, descartada). |
| Tipografía título | **Big Shoulders** (peso 900 para el hero). En `next/font/google` el nombre de importación correcto es `Big_Shoulders` — **no** `Big_Shoulders_Display` (ese nombre no existe y rompe el build). |
| Tipografía cuerpo | Public Sans (400/500/600) |
| Tipografía datos/HUD | JetBrains Mono (400/500) |
| Ícono de marca | **Curvas de nivel** con un nodo que hace glow — versión final. Se descartaron: el ícono de "montaña + sol" (muy cliché) y la "retícula" tipo visor de cámara (opción B en su momento). |
| Efecto de título | Tratamiento con gradiente + `-webkit-text-stroke` + glow animado (`carmenGlow`), aportado por el cliente vía Claude Desktop — ya integrado en `site/src/app/globals.css` y aplicado en `page.tsx` con la clase `.carmen-glow-text`. |
| Favicon | Se genera automáticamente a partir del ícono de curvas de nivel (pendiente de implementar). |

**Descartado — no revivir:**
- Paleta nocturna / glow de atardecer (v1 del dossier)
- Ícono "montaña + sol"
- Ícono "retícula" (corner brackets tipo visor)
- Siluetas de animales integradas en las letras del título (se veía saturado)
- Grid de tarjetas parejas para el catálogo de escenas (v1 — "muy plano, sin diseño")

### Tokens de color (ya en `site/src/app/globals.css`)
```
--background: #15120d      --primary: #c9a876
--foreground: #f2ecdf      --primary-soft: #dec397
--surface: #1f1a13         --primary-foreground: #15120d
--surface-2: #28221a       --accent-deep: #7a5a34
--muted-foreground: #9c9280 --sky: #7c8fa0
--faint: #675e4e           --scrub: #767a55
--border: #332c22          --signal: #7c9082
--ring: #c9a876
```

### Coordenadas/datos de referencia (usar siempre los mismos)
- Lat/Lon base: `29.1043° N, 102.5391° W` (Sierra del Carmen, Coahuila)
- Framing: "Corredor de conservación · Coahuila–Texas"

---

## Arquitectura del sitio — definida

1. **Hero:** un solo video, full-bleed, en loop, silencioso, con ícono de sonido.
2. **Al bajar:** catálogo de otras escenas — **dirección visual aún en definición** (ver "En proceso" abajo).
3. Interacción de escena: al activarla, se despliega a pantalla completa con la misma marca/HUD/tratamiento de luz que el hero.

## Video — settings de exportación (DaVinci Resolve 18.1.4, ya acordados)

- Dos niveles de resolución: **1920×1080** (14,000 Kb/s, 2-pass, Level 4.2) y **3840×2160** (45,000 Kb/s, 2-pass, Level 5.1)
- H.264, High profile, 8-bit, MP4, modo "Individual clips"
- Frame rate: "Same as Project", igual en todos los clips (nunca mezclar fps entre clips)
- **Color: Rec.709 (Video levels), Gamma 2.4 — crítico.** Si el timeline trabaja en Log/Wide Gamut, aplicar el CST de salida antes de exportar o el video se ve lavado en el navegador.
- Sin audio en los clips.
- Nomenclatura: `01-nombre-escena-1080.mp4` / `01-nombre-escena-4k.mp4`
- El cliente **ya tiene clips exportados** con estos settings — falta recibirlos/ubicarlos.
- Los archivos de video **no van dentro del repo de git** — se decide su hosting (Vercel Blob u otro) cuando lleguemos a integrarlos.

---

## Estado técnico del código (`site/`)

- Next.js 16 + Tailwind v4 + TypeScript, `npm run build` pasa sin errores.
- `ui.shadcn.com` está **bloqueado por la política de red de este entorno** — el `shadcn init`/`add` no funciona. Los componentes (`button`, `badge`, `separator`) están **escritos a mano** en `site/src/components/ui/`, siguiendo las convenciones de shadcn (cva, Radix, `cn()` en `site/src/lib/utils.ts`). No perder tiempo reintentando el CLI de shadcn — replicar componentes a mano cuando se necesiten más.
- `site/src/app/layout.tsx`: las tres fuentes cargadas vía `next/font/google`, metadata básica en español.
- `site/src/app/page.tsx`: **solo un placeholder** con el título en glow — el hero real con video, el HUD, el catálogo de escenas, etc. **todavía no están construidos en el proyecto real**, solo existen como bocetos/artifacts fuera del código (ver abajo).
- GitHub: la app "Claude" ya está instalada con permiso de escritura sobre este repo — `git push` funciona normal. No hay pendientes de acceso.
- Vercel: no se ha desplegado nada todavía (Fase 6 del flujo no alcanzada).

### Explícitamente fuera de este traspaso
- Todo el tema de **Playwright MCP / `.mcp.json`** — se intentó para poder navegar sitios de referencia bloqueados, no se logró estabilizar en este entorno (problemas de navegador, proxy, y un caché de configuración que no se refrescaba ni reiniciando sesión). Se descarta la vía MCP por ahora. El archivo `.mcp.json` fue eliminado del repo.
- Sitios de referencia externos (`ui.shadcn.com`, `polarpro.com`, `depoluxe.xyz`) están bloqueados por la política de red de este entorno — la única vía que ha funcionado es que el cliente mande **capturas de pantalla** directo en el chat.

---

## Bocetos ya aprobados/entregados (fuera del código, como artifacts)

- Dossier de dirección de diseño v1→v4 (paleta, tipografía, ícono, HUD) — v4 es la versión final aprobada.
- PDF de settings de exportación de video (DaVinci Resolve) — ya entregado al cliente.
- Boceto de tarjetas v2 (destacada + bitácora) — mejor que v1, pero el cliente pidió explorar otra dirección antes de decidir.
- Boceto de scroll full-frame con parallax — construido, pendiente de aprobación y de sumarle el detalle de la referencia `depoluxe.xyz`.

---

## ✅ Checklist — dónde estamos

### Hecho
- [x] Brief completo (negocio, audiencia, contenido, tono)
- [x] Sistema de diseño aprobado (paleta, tipografía, ícono, tratamiento de luz)
- [x] Efecto de glow del título integrado en el código real
- [x] Proyecto Next.js armado y corriendo (`site/`)
- [x] Componentes base (Button, Badge, Separator) construidos a mano
- [x] Settings de exportación de video acordados y documentados (PDF entregado)
- [x] Acceso a GitHub resuelto — push funcionando
- [x] Arquitectura del sitio decidida: 1 hero + catálogo de escenas expandibles

### En proceso — decisión pendiente del cliente
- [ ] Dirección visual final del catálogo de escenas (tarjetas descartadas; scroll parallax en revisión; falta ver referencia de `depoluxe.xyz` — gráfico lateral de números romanos animado + controles de video: fullscreen, play, sonido on/off, stop)
- [ ] Eslogan final (propuesta: "Tal como siempre ha estado" — el cliente dijo que también pensaría el suyo)
- [ ] Definir la pista de audio ambiente para el hero

### Pendiente — no iniciado
- [ ] Construir el hero real con video (hoy solo hay placeholder)
- [ ] Construir el catálogo de escenas en el código real (una vez se apruebe la dirección visual)
- [ ] Recibir/ubicar los clips ya exportados por el cliente e integrarlos
- [ ] Decidir hosting de los archivos de video (no van en git)
- [ ] Favicon generado desde el ícono de curvas de nivel
- [ ] QA responsivo (375 / 768 / 1024 / 1440px) y accesibilidad
- [ ] Deploy privado de prueba (Vercel) — dominio propio se conecta al final

---

## Siguiente paso inmediato

**Pídele al cliente:** capturas de pantalla de `depoluxe.xyz` — una del gráfico lateral de números romanos en reposo, y una o dos con un video reproduciéndose mostrando los controles (fullscreen, play, sonido on/off, stop). Con eso se cierra la dirección del catálogo de escenas y se puede avanzar a construirlo en el código real.
