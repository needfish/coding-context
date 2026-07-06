# Web App Stack Choices

Web apps follow 5 levels. Default at Level 2.

| Level | Stack | When to use |
|---|---|---|
| 1: Static HTML | Astro, zero client JS | Content sites, docs, marketing pages |
| 2: Preact + HTMX | Vite + Preact + htmx + Tailwind + Basecoat, server-first state | Most web apps -- default |
| 3: Preact + HTMX + Vanilla JS | Vite + Preact + htmx + vanilla JS for client polish/validation | Forms with rich validation, optimistic UI |
| 4: Preact SPA | Preact with client-side routing | Isolated interactive widgets that coordinate via server (not client state) |
| 5: Full Framework | SvelteKit or SolidStart. Unified client-side app model. | Dashboards, PM tools, collaborative editors -- heavy client-state reactivity |
