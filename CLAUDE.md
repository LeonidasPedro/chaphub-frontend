# ChapHub Frontend

Public site + `/painel` (user/admin) of ChapHub (Ecossistema de Inovação de Chapecó/SC), rebuilt
from scratch. Consumes the REST API in `~/Projects/chaphub-backend` (`/api/v1`, cookie session) on
the same domain behind nginx.

## Source of truth

- Business requirements: Obsidian `chaphub/Requisitos.md` (RF01–RF08, RNF01–RNF03, RN01–RN07).
  Cite RF/RN IDs in commits when relevant.
- Planning: `.planning/PROJECT.md`, `.planning/REQUIREMENTS.md`, `.planning/ROADMAP.md`,
  research in `.planning/research/`.
- Old prototype (`~/Projects/chaphub`): **never copy code or visuals**. Its description lives in
  `docs/ANALISE-PROTOTIPO.md`; only brand assets listed there are reused.
- Backend gaps the frontend depends on: `~/Projects/chaphub-backend/.planning/todos/pending/`.

## Rules (imported from the backend, agreed with the user)

- **English for all code** (identifiers, files, folders); **PT-BR for UI text**. Error `code`
  values from the API are stable English snake_case: branch on `code`, never on `message`.
- **JavaScript ESM, no TypeScript.** Node 24 via asdf (`.tool-versions`).
- **Test-first (TDD)**: red → green → refactor. Every feature starts with a failing test.
- **Regression tests (critical here)**: every bug fix ships with a test that reproduces the bug
  and fails before the fix.
- **New dependency → document it** in both `README.md` and this `CLAUDE.md` (what and why).
- **README keeps a "Se você é uma IA, comece aqui" section** up to date.
- **Continuous security**: every phase reviews XSS (Markdown without raw HTML, http/https-only
  links), open redirects (`?next=` only `/painel`), CSRF/Origin, cache leaks on logout, LGPD
  (never show e-mail/phone publicly). No final "hardening" phase.
- **Design**: follow the brand manual, not "AI slop" (see `docs/ANALISE-PROTOTIPO.md` §3–4 and
  `docs/DESIGN-RULES.md` once created). Headings in **Unbounded**, body in **Lato**; Organetto is
  forbidden (license). No glow, blur, decorative shadows, pill buttons, hover `translateY`,
  generic icon libraries, text over gradients. Allowed color pairs are enforced by tests.
- **Git**: agents may commit in this repo (unlike the backend). Never commit `.claude/`.
- Never run `sudo`; show the command and ask the user.

## Dependencies

None installed yet. Each one added must be listed here and in `README.md` with its purpose.

<!-- GSD:project-start source:PROJECT.md -->

## Project

**ChapHub Frontend**

Frontend novo (do zero) da plataforma ChapHub — Ecossistema de Inovação de Chapecó/SC. Substitui a landing estática (`~/Projects/chaphub`) por um site público (landing + notícias, eventos, GTs, documentos, indicadores) e um painel autenticado (`/painel`) para admins e usuários de instituição, consumindo a API REST de `~/Projects/chaphub-backend`. Público: visitantes do ecossistema; instituições parceiras que publicam; admins que aprovam.

**Core Value:** Visitante vê o conteúdo publicado do ecossistema numa identidade visual própria da marca (não "cara de IA"); instituições publicam e admins aprovam pelo mesmo app, sem fricção.

### Constraints

- **Stack**: React + Vite, **JavaScript ESM (sem TypeScript)**, Node 24 (asdf, `.tool-versions`) — decisão do usuário, alinhado ao backend
- **Testes (TDD obrigatório)**: Vitest + Testing Library + MSW (mock da API) + Playwright (E2E). Ambiente DOM: jsdom (padrão); `// @vitest-environment node` no cliente de API. happy-dom rejeitado na pesquisa (DOMPurify não sanitiza nele; ganho de velocidade irrelevante)
- **Regressão**: todo bug corrigido ganha teste que o reproduz — regra crítica neste projeto
- **Tipografia**: Organetto PROIBIDA (DEMO, protegida). Títulos em **Unbounded** (variável 200–900, PT-BR completo; Michroma/Krona One só têm peso 400); texto em Lato. Self-hosted
- **Deploy**: VPS própria, nginx serve o build estático e faz proxy de `/api` no mesmo domínio (sem CORS em produção); SPA fallback
- **Painel**: mesmo app, rotas `/painel/*`
- **Idioma**: código em inglês; textos de UI em PT-BR
- **Segurança contínua**: aplicada a cada fase, sem fase final de "endurecimento"
- **LGPD**: nunca exibir e-mail/telefone em telas públicas

<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->

## Technology Stack

## Executive decisions (one line each)

| # | Question | Decision | Confidence |
|---|----------|----------|------------|
| 1 | Router | **React Router 8.5 data mode** (`createBrowserRouter`, `route.middleware` guard, `lazy` for `/painel`) | HIGH |
| 2 | Server state | **TanStack Query 5** as the only server-state cache; RR loaders only *prefetch into it* (`ensureQueryData`) | HIGH |
| 3 | Forms | **React Hook Form 7.90 + Zod 4.6 + `@hookform/resolvers` pinned to 5.9.2** (6.0.0 fails `npm install`) | HIGH |
| 4 | News body | Backend stores **plain TEXT, Markdown by plan, no sanitization, no length cap**. Render with **react-markdown 10** (no `innerHTML`); edit with textarea + toolbar + preview. DOMPurify not needed. | HIGH |
| 5 | Styling | **CSS Modules + native CSS (custom properties, `@layer`, nesting, container queries)**. No Tailwind, no CSS-in-JS | HIGH (tooling) / MEDIUM (design judgement) |
| 6 | Head/SEO | **React 19 native `<title>/<meta>/<link>`**; crawlers do not run JS, so add an **nginx bot-UA shim** for `/noticias/:slug` and `/eventos/:slug` | HIGH (React) / MEDIUM (shim) |
| 7 | Testing | **Vitest 5 + RTL + user-event + MSW 3 + Playwright. Default env = jsdom 30** (explicit verdict below); `// @vitest-environment node` for the API layer; happy-dom **not** adopted | HIGH |
| 8 | Fonts | **Unbounded** (variable, `@fontsource-variable/unbounded`) for headings + **Lato** (static, incl. italic) for body | HIGH (facts) / MEDIUM (taste) |
| 9 | Lint/format | **ESLint 10 flat + Prettier 3.9** (parity with backend). Not Biome | HIGH |
| 10 | Images | `srcset` built from `/api/v1/files/:id/thumb` (400w) + `/medium` (1200w); fixed CSS `aspect-ratio` because cards get no width/height | HIGH |

## Recommended Stack

### Core Technologies

| Technology | Version (npm 2026-10-22) | Purpose | Why Recommended |
|------------|--------------------------|---------|-----------------|
| Node.js | 24.x (>= 24.15; box runs 24.22.0) | Toolchain runtime | Backend already pins Node 24 via asdf. jsdom 30 needs `^24.15`, RR 8 needs `>=22.22`, Vitest 5 `^24.0.0`. Put `nodejs 24.22.0` (or newer 24) in `.tool-versions` |
| React / React DOM | 19.3.1 | UI | Native document metadata hoisting (`<title>`, `<meta>`, `<link>`), ref-as-prop, `<form>` actions. RR 8 requires `>=19.2.7` |
| Vite | 8.3.5 | Dev server + bundler | Rolldown-based (10-30x faster builds per Vite team), Node `^20.19 \|\| >=22.12`, default `build.target` = `baseline-widely-available` (chrome111/firefox114/safari16.4), lightningcss CSS minify by default. Vitest 5 peers on `vite ^6.4 \|\| ^7 \|\| ^8` |
| @vitejs/plugin-react | 6.1.2 | JSX + Fast Refresh | v6 drops Babel (Oxc-based refresh), **requires Vite 8**. React Compiler is opt-in/experimental, **do not enable** (extra moving part, no measured need) |
| react-router | 8.5.0 | Routing (data mode) | See Q1. v8 (2026-06-17) is ESM-only, removed `react-router-dom` (import `RouterProvider` from `react-router/dom`), middleware is always on in data mode |
| @tanstack/react-query | 5.104.5 | Server-state cache | See Q2. React `^18 \|\| ^19` |
| react-hook-form | 7.90.0 | Forms | See Q3 |
| zod | 4.6.7 | Validation schemas | Same major/minor as backend (`zod ^4.6.7`), same mental model and message style |
| react-markdown | 10.1.0 | Safe Markdown render | See Q4. Peers `react >= 18`; no raw HTML by default |

### Styling and fonts

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| CSS Modules (built into Vite) | n/a | Component-scoped CSS | Zero runtime, zero config, works in Vitest, JS-friendly |
| `@fontsource-variable/unbounded` | 5.3.0 (OFL-1.1) | Headings | See Q8 |
| `@fontsource/lato` | 5.3.0 (OFL-1.1) | Body + italic voice | Static 100/300/400/700/900, normal + italic, latin + latin-ext |

### Testing

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| vitest | 5.0.5 (5.1.0-beta exists; stay on `latest`) | Unit/component runner | v5 (2026-09-03). Node `^22.12 \|\| ^24 \|\| >=26`. `clearMocks` now defaults to `true`; `vi.mock/hoisted` must be top-level |
| @vitest/coverage-v8 | 5.0.5 | Coverage | Must match vitest version exactly |
| jsdom | 30.1.3 | Default DOM env | Q7 verdict. Node `^22.22.2 \|\| ^24.15 \|\| >=26` |
| @testing-library/react | 16.3.3 | Component tests | Peer `@testing-library/dom ^10` must be installed explicitly |
| @testing-library/dom | 10.4.2 | RTL peer | Install explicitly (npm will not auto-pull for you reliably) |
| @testing-library/user-event | 14.6.8 | Realistic interaction | Always `userEvent.setup()` per test |
| @testing-library/jest-dom | 7.0.1 | DOM matchers | Import `@testing-library/jest-dom/vitest`. Node `>=22` |
| msw | 3.0.3 | API mocking (Node) | v3 (2026-09-28) is ESM-only, Node `>=22`. **Breaking vs v2: see "MSW 3 traps"** |
| @playwright/test | 1.64.1 | E2E | Real browser, real backend with seeded DB |
| @axe-core/playwright | 4.14.0 | A11y smoke in E2E | Run axe on each page in E2E, not in jsdom |

### Development Tools

| Tool | Version | Purpose | Notes |
|------|---------|---------|-------|
| eslint | 10.13.0 | Lint | Backend is `^10.12.0`. Flat config only |
| @eslint/js | 10.0.1 | Base rules | |
| globals | 17.13.0 | Env globals | |
| eslint-plugin-react | 7.37.5 | JSX rules | **Peer range stops at ESLint 9.7** (npm `ERESOLVE`); works on 10.13 in practice (verified). Use `overrides`, see Installation |
| eslint-plugin-jsx-a11y | 6.10.2 | A11y lint | Same peer-range problem, same fix, verified working on 10.13 |
| eslint-plugin-react-hooks | 7.1.1 | Hooks + React Compiler rules | Declares ESLint 10 support. `flat.recommended` already flags `set-state-in-effect` |
| eslint-plugin-react-refresh | 0.5.7 | Fast-refresh safety | `configs.vite` |
| @tanstack/eslint-plugin-query | 5.104.5 | Query key/deps rules | Supports ESLint 10 |
| eslint-plugin-testing-library | 7.16.2 | RTL best practices | Supports ESLint 10 |
| @vitest/eslint-plugin | 1.6.28 | Vitest rules | Use this one. `eslint-plugin-vitest` 0.5.4 only declares ESLint 8/9 |
| eslint-plugin-playwright | 2.12.1 | E2E rules | Scope to `e2e/**` |
| eslint-config-prettier | 10.1.8 | Disable style rules | Put last |
| prettier | 3.9.10 | Formatting | Identical to backend |
| stylelint + stylelint-config-standard | 17.16.0 / 40.0.0 | CSS lint (optional, recommended) | Can enforce the anti-slop rules mechanically (see Q5) |

### Optional / defer

| Library | Version | When |
|---------|---------|------|
| dompurify | 3.4.17 | Only if some feature must inject an **HTML string** (none planned). Verified to sanitize correctly under jsdom |
| @tanstack/react-query-devtools | 5.104.5 | Dev-only, optional |
| @tiptap/react + @tiptap/starter-kit + @tiptap/markdown | 3.32.0 | Only if institution editors reject the Markdown textarea in UAT. Keeps Markdown as storage format |
| clsx | 2.1.1 | Only if conditional class joining gets noisy; template strings are fine otherwise |

## Q1. Router: React Router 8 data mode (HIGH)

- Node >= 22.22, React >= 19.2.7, Vite 7+ (framework mode only), ESM-only.
- `react-router-dom` is **gone**: `RouterProvider` from `react-router/dom`, everything else from `react-router`.
- `future.v8_middleware` removed: **client middleware is always on**. In data mode: `route.middleware = [fn]`, `fn({ request, context }, next)`, `createContext()`, `context.get/set`, `throw redirect(...)` stops navigation before loaders run.
- **Trap (verified):** an error thrown in a route's *own* middleware is caught by the **parent's** `errorElement`, not that route's. Put the 403/404 boundary on `/painel`, not on `/painel/admin`.
- `staleTime` on `meQuery` (e.g. 5 min) so the guard does not hit `/me` on every navigation. After login `setQueryData(['me'], user)`; after logout `queryClient.clear()` (drops private data from memory) then navigate to `/login`.
- The server is the real authorization boundary (backend enforces role + institution scope). The guard is UX only. Never hide security behind it.
- Code-split `/painel/*` with `lazy` so anonymous visitors do not download CMS code.
- `ScrollRestoration` + CSS `scroll-margin-top` replaces the prototype's `useSmoothScroll` hack.

## Q2. Server state: TanStack Query 5 as single cache; loaders prefetch only (HIGH)

| Option | Verdict | Reason |
|--------|---------|--------|
| TanStack Query + RR loaders that call `ensureQueryData` | **Chosen** | One cache. Navigation back/forward is instant (public read-heavy pages benefit from `staleTime`). `placeholderData: keepPreviousData` gives smooth pagination. Targeted invalidation after mutations. Loaders still give "data is ready before the page renders" without waterfalls. |
| RR loaders + actions only | Rejected | Every navigation refetches (no cache), and `actions` revalidate *all* active loaders after approve/reject. Also fights React Hook Form (`handleSubmit` vs `<Form>`), and server field errors via `useActionData` duplicate what RHF already does. |
| Plain `fetch` in `useEffect` | Rejected | Re-implements caching, races, abort, loading/error state badly. Lint rules (`react-hooks`) now flag it. |
- **Query-key factory** per resource: `newsKeys.list({page, perPage, workGroup, category})`, `newsKeys.detail(slug)`, `queueKeys.pending()`.
- **Thin `api()` wrapper** over `fetch`: `credentials: 'include'`, JSON in/out, throws `ApiError{status, code, message, fields}` from the backend envelope `{ error: { code, message, fields? } }`. No Axios (fetch is intercepted by MSW directly; Axios adds the XHR path for nothing).
- **Pagination** `{ data, meta: { page, per_page, total } }`: key includes `page`; `placeholderData: keepPreviousData`; page count = `Math.ceil(total / per_page)`. Keep `page` in the URL search params (shareable, back button works).
- **Mutations**: `useMutation` + `onSuccess: () => queryClient.invalidateQueries({ queryKey: queueKeys.all })`. For approve/reject specifically invalidate the approval queue, the affected `news`/`events`/`documents` admin lists, and the public lists (`newsKeys.lists()`), since publishing changes what visitors see. Optimistic update is *not* worth it for approve/reject (rare, must reflect server truth, rejection needs a reason).
- **401 handling**: in the `QueryCache`/`MutationCache` `onError`, on `status === 401` clear the cache and redirect to `/login?next=`.
- Query defaults: `retry: (n, e) => e.status >= 500 && n < 2` (never retry 4xx), `refetchOnWindowFocus: false` for the public site, `true` is fine in the panel queue.
- State management beyond this (Redux/Zustand/Context stores) is **not needed**: server state in Query, URL state in the router, form state in RHF, UI state local.

## Q3. Forms and validation: RHF 7.90 + Zod 4.6 (HIGH)

- `react-hook-form@7.90.0` + `zod@4.6.7` + **`@hookform/resolvers@5.9.2`**.
- **Pin resolvers to `5.9.2`, not `6.0.0`.** `6.0.0` (published 2026-10-19; its only declared breaking change is the Effect v4 resolver) **fails a fresh `npm i` with `ERESOLVE`** in a clean directory (reproduced). Revisit when 6.x fixes its optional-peer chain.
- Valibot 1.5.0 is smaller, but Zod gives parity with the backend (same library, same error idioms, and the team can copy field rules). Bundle cost is irrelevant for this app.
- Other codes: 401 -> redirect login; 403 -> form-level message; 409 -> `err.message`; 413 `payload_too_large` -> upload-field error; 429 (rate limit) -> form-level message.
- Pair every input with `<label htmlFor>` and `aria-invalid` + `aria-describedby` to the error node, so RTL `getByLabelText` and `getByRole('alert')` tests double as a11y checks (verified pattern, server error rendered after a client-valid submit).
- **File inputs rule (verified, both DOM envs):** do **not** build uploads with `new FormData(formElement)`. `user.upload()` sets `input.files` on the element, but `FormData(form)` reads jsdom/happy-dom internals and yields an **empty file (name `''`, size 0)**. Keep the `File` in component/RHF state (`onChange` -> `setFile(e.target.files[0])`) and build `const fd = new FormData(); fd.append('file', file)` manually. This is also the only version that is testable.
- Client-side upload guards (type, size) are UX only; backend does real MIME detection (`file-type`) and limits.

## Q4. News body: Markdown in, safe React elements out (HIGH)

- `News.body` is `DataTypes.TEXT`, default `''`. Zod on write: `body: z.string().optional()` (**no max length, no sanitization**). Public `newsView.detail()` returns `body` verbatim. `summary` is also plain TEXT.
- `docs/PLANO.md` states the default "**corpo da notícia em Markdown**" (a planned default, **not enforced** by code). Gallery is separate: `images[]` with `caption`, `displayOrder`, `file{ id, width, height, url }`; `cover` is a URL string only.
- Consequence: an institution `user` can store arbitrary strings that the public site renders. Treat `body` as **untrusted** (stored-XSS vector).
- Subtitles = Markdown `##`/`###` -> `h2`/`h3` (page title is the `h1`; do not allow `#` or `h1`). Restyle through CSS Modules on the wrapper (`.prose h2 {...}`), not per-element inline styles.
- **No `img` in `allowedElements`**: external images in body are a tracking/LGPD vector; use the gallery (`images[]`) instead.
- **DOMPurify is unnecessary** with this approach. Keep `dompurify@3.4.17` as the escape hatch only if an HTML string must be injected; if so, sanitize at render, test under **jsdom** (verified: strips `onerror`, `javascript:`), never happy-dom (see Q7).
- **Editor (admin/user panel):** `<textarea>` + small toolbar (insert `## Subtítulo`, **bold**, *italic*, list, link) + live preview rendered by the **same** `<NewsBody>` component. Zero extra dependencies, matches the storage format, trivially testable. Defer Tiptap 3.32 (ProseMirror, ~0.5 MB unpacked for `@tiptap/markdown` alone) until UAT proves non-technical editors need WYSIWYG; it can round-trip Markdown so storage does not change.
- **Cross-project asks for the backend (record as roadmap dependencies):** (1) enforce a `max` length on `body`/`summary`; (2) document "body is Markdown" in the spec; (3) optionally a `og` JPEG variant (see Q6).

## Q5. Styling: CSS Modules + native CSS, no Tailwind (HIGH tooling / MEDIUM design)

- The brand manual's language is **flat colour blocks, split compositions (solid | photo), straight cuts, a long horizontal gradient, single-line illustration**. That is expressed with a handful of custom properties and a few layout primitives, not a utility vocabulary. Tailwind's defaults (rounded-xl, shadow-md, slate/indigo ramps, `gap-6 grid-cols-3` card grids) are exactly the "AI slop" catalogue in `docs/ANALISE-PROTOTIPO.md` section 3; fighting them costs more than not having them.
- Vite handles CSS Modules natively; Vitest 5 handles them with zero config (class names stable/hashed, assertions use roles not classes). Nothing to install.
- Native CSS in 2026 covers what preprocessors were for: nesting, `@layer`, `:has()`, container queries, `clamp()`, `color-mix()`, `@property`, `text-wrap: balance`, `scroll-margin-top`, `prefers-reduced-motion`. Vite 8's `build.target` is Baseline-widely-available (Safari 16.4+), and lightningcss (now a hard Vite dependency) minifies and lowers syntax at build.
- **vanilla-extract** (`@vanilla-extract/css` 1.21.3): typed-token ergonomics are the draw, and types do nothing in JS; it adds a Vite plugin, `.css.js` authoring files, and build-graph complexity inside Vitest. Rejected.
- **styled-components/emotion**: runtime CSS-in-JS, extra bundle, CSP `style-src` friction. Rejected.

## Q6. Head/SEO/OG in an SPA (HIGH React / MEDIUM shim)

- `<title>`, `<meta>`, `<link>` rendered anywhere are placed in `<head>`; on re-render they update; on unmount **they are removed** (title becomes `''`).
- `<title>` children must be a **single string**: write `<title>{`${news.title} | ChapHub`}</title>`, not `<title>{a} | ChapHub</title>`.
- **No dedupe (verified trap):** if `index.html` contains `<title>`/`<meta name="description">` and a route also renders them, **both end up in `<head>`** (React prepends its own; static copy stays). So **`index.html` must contain only charset, viewport, theme-color, icons, and a no-JS `<noscript>` note, never title/description/OG.** Every route renders its full set through one `<Seo title description image path type />` component; layouts do not render meta.
- Each page: `<title>`, `description`, `canonical` (`<link rel="canonical">`), `og:title|description|image|url|type|locale(pt_BR)|site_name`, `twitter:card=summary_large_image`. `og:image` and `og:url` must be **absolute** URLs.
- Google renders JS, so ranking-oriented tags work from the SPA. **Social crawlers do not** (facebookexternalhit, WhatsApp, LinkedInBot, Twitterbot, Slackbot, TelegramBot, Discordbot fetch raw HTML only). A bare SPA therefore shows the same blank preview for every share. Requirement in PROJECT.md ("imagem OG por notícia/evento") cannot be met by React alone.
- nginx maps bot User-Agents on `/noticias/:slug` and `/eventos/:slug` to a localhost-only Node process (`ops/share-shim.mjs`, no dependencies, systemd unit) that: reads the built `dist/index.html`, fetches `/api/v1/news/:slug` (or events), **HTML-escapes** every value, injects `<title>`/OG/Twitter/canonical tags before `</head>`, returns 200 (404 status for unknown slugs). Humans keep getting the static SPA. Logic is a pure function `renderShareHtml(template, entity, origin)` -> trivially unit-tested, including escaping (`"`, `<`, `&`).
- Why not alternatives: build-time prerender (`vite-plugin-prerender` 1.0.8 etc.) cannot know future slugs. Switching to SSR/framework mode contradicts the SPA decision. Changing the backend to emit share HTML is possible but couples a separate repo to frontend templates.
- nginx sketch:
- **OG image format risk (MEDIUM/LOW):** the backend's variants are **WebP only** (`/files/:id/medium`, 1200w). Twitter/X documents WebP support; Facebook/WhatsApp reportedly render it though their docs list JPEG/PNG/GIF; LinkedIn is reported to ignore WebP `og:image` (community sources, not official). Cheapest mitigation is a backend follow-up: an `og` variant (1200x630 JPEG). Until then use `/medium` and verify previews manually on WhatsApp and LinkedIn.
- Also set `<html lang="pt-BR">`, per-route `<title>`, and a `sitemap.xml` + `robots.txt` (generated by a cron/script hitting the API; out of scope for v1 if time-boxed).

## Q7. Testing (TDD critical)

### Versions and what changed (HIGH)

- **Vitest 5.0.5**: requires Node `^22.12 || ^24 || >=26`, Vite `>=6.4`. Changes that matter: `clearMocks` defaults to `true`; `vi.mock/unmock/hoisted` must be top-level (throws otherwise); `expect.poll` rejects on timeout and **unawaited async assertions fail the test** (good for TDD); coverage `include/exclude` are root-relative; config files are not searched in parent dirs; env `globalThis` assignments propagate to the DOM impl.
- **MSW 3.0.3** (v3 2026-09-28): ESM-only, Node >= 22, `http`/`HttpResponse` from **`msw/http`**, `setupServer` from `msw/node`, GraphQL moved to `msw/graphql`.

### MSW 3 traps (verified by execution)

### jsdom vs happy-dom: concrete comparison (all rows executed; see lab)

| Capability | jsdom 30.1.3 | happy-dom 21.0.4 |
|---|---|---|
| Relative `fetch('/api/v1/...')` + relative MSW handler | OK (resolves against `location`) | OK |
| JSON, error envelope 400, streaming `ReadableStream` response, `blob()` | OK | OK |
| `AbortSignal` into `fetch`/RR loaders, `createBrowserRouter` + loaders + `<Link>` navigation | OK | OK |
| RHF + Zod + server field errors, constraint validation (`required` blocks submit) | OK | OK |
| React 19 `<title>/<meta>` hoist, update, remove | OK | OK |
| TanStack Query + MSW | OK | OK |
| **Multipart upload: `FormData` + `File` -> `fetch` -> MSW `request.formData()`** | **FAILS**: `RequestContentLengthMismatchError`/"Premature close" (jsdom's `FormData`/`File` are not undici's; Node fetch serializes it as `[object FormData]`). Vitest 5.0.5's built-in jsdom shim still delivers an **empty, renamed (`blob`) file** with jsdom 30.1.3 | **OK** (name, size, bytes intact) |
| `new FormData(form)` after `user.upload()` | Empty file (user-event overrides `files` on the element) | Same: empty file |
| **DOMPurify 3.4.17 sanitization** | **Correct** (`<img src="x">`, `<a>x</a>`) | **BROKEN: `onerror` and `javascript:` href survive** (`isSupported` is `true`, so no warning). Do not trust any sanitizer test here |
| `HTMLDialogElement.showModal` | Missing (stub 2 lines) | Present |
| `matchMedia`, `IntersectionObserver`, `ResizeObserver`, `scrollIntoView` | Missing (stub) | Present |
| `getComputedStyle` resolves `var(--x)` | No (returns `var(--x)`) | Yes |
| `popover` API, `inert`, `navigator.share`, `DataTransfer` | popover/inert/share/DataTransfer missing | popover/share missing, `inert` present |
| `Intl` pt-BR + `America/Sao_Paulo` | OK (Node ICU) | OK |
| Speed, small suite (41 tests) | 2.5 s | 2.0 s |
| Speed, stress (300 tests, 200-link DOM, `getAllByRole`) | **33 s** | **13 s** (~2.5x faster) |
| Install size | 7 MB | 18 MB |
| Maturity / risk | Reference DOM implementation; used by RTL and MSW docs as default | 21.0.0 shipped 2026-10-11 (breaking: CSS/iframe loading disabled by default to prevent SSRF). Earlier majors had a sandbox-escape class of CVEs. Younger, less spec-strict |
- `// @vitest-environment node` for: the `api()` client, error mapper, query-key factories, pagination maths, the **share-shim** (`renderShareHtml`), and any test that sends multipart (`uploadImage(file)` against MSW with Node-native `FormData`/`File` passes). In node env there is no `location`: make the API base URL injectable (`import.meta.env.VITE_API_URL ?? ''`, `vi.stubEnv('VITE_API_URL', 'http://localhost:5173')`).
- Component tests for upload forms: either `vi.mock('../api/uploads.js')` and assert the component hands the `File` to `uploadImage`, or (only if you truly need the whole path in one test) opt that single file into `// @vitest-environment happy-dom`. Never switch the project default.
- Never use `happy-dom` for anything involving a sanitizer.

### Recommended config and setup (verified in lab)

- With `globals: false`, RTL's auto-cleanup does not register: the explicit `cleanup()` above is mandatory.
- New `QueryClient({ defaultOptions: { queries: { retry: false, gcTime: 0 } } })` **per test** via a `renderWithProviders()` helper; routes tested via `createMemoryRouter(routes, { initialEntries })` + `RouterProvider` from `react-router/dom`.
- Query by role/label/text only; never by class (CSS Modules hash them).
- Do **not** assert computed colours or `var()` values in jsdom; that belongs to Playwright.
- jsdom prints "not implemented" noise for navigation/scroll; the stubs above remove it.
- Regression rule from PROJECT.md: every bug fix commits a test that fails first. Name them `regression: <symptom>`.
- Playwright (`@playwright/test` 1.64.1): `webServer` runs `vite build && vite preview` with `preview.proxy['/api'] -> http://localhost:3333`, against the **real backend on a seeded test DB** (login, create draft -> submit -> admin approve -> visible on public page, upload, 401 redirect, 403 admin guard). Use `@axe-core/playwright` per page. Do not mock the API in E2E; MSW is for unit/component tests only.
- Vitest Browser Mode (`@vitest/browser`, real browser component tests) is a legitimate later alternative that would remove the jsdom question entirely; not now (needs browsers in CI, slower start, adds a third test layer). Revisit if jsdom gaps (focus/dialog/layout) keep biting.

## Q8. Fonts: choose **Unbounded** (variable) + Lato (HIGH facts / MEDIUM taste)

| Criterion | **Unbounded** | Michroma | Krona One |
|---|---|---|---|
| License | OFL-1.1 | OFL-1.1 | OFL-1.1 |
| Weights | 200-900 (8) | 400 only | 400 only |
| Variable font | **Yes** (`wght` 200-900), `@fontsource-variable/unbounded` | No | No |
| PT-BR coverage (`ã õ ç é ê á í ó ú â`, caps, `« » – — …`, `€`) | Full, in the `latin` subset | Full | Full |
| Subsets shipped | latin, latin-ext, vietnamese, cyrillic(+ext) | latin, latin-ext | latin, latin-ext |
| `latin` woff2 size | 51 KB (whole weight range) | 18 KB | 10 KB |
| Kerning (GPOS) | Yes | Yes | **No** |
| Lowercase | Yes, fully usable in mixed case | Yes | Yes (quirky letterforms: unusual `N`, `t`, `j`) |
| Character at 14-18 px | Readable at 500-600; 700+ for UI | Hairline strokes, spaced out, weak on dark backgrounds at small size | Readable but wide and idiosyncratic |
| Closeness to Organetto (all-caps, extra-wide geometric) | Wide rounded geometric, closest in *attitude* when set in caps at 700-800; less extreme width (M ~1.18 em) | Widest (M ~1.37 em) but thin, sci-fi/HUD feel | Wide and heavy-ish, but quirky |
| Hierarchy (h1 vs h3 vs button) | Achievable with one family via weight | Impossible without faux bold | Impossible without faux bold |
- The variable package ships only `index.css` and `wght.css`, and **both declare every subset** (cyrillic, cyrillic-ext, latin, latin-ext, vietnamese). Importing them makes Vite copy all five woff2 files into `dist/` even though browsers only download the subsets they need. Instead write your own `@font-face` in `src/styles/fonts.css` pointing at the package files for **`latin`** (`files/unbounded-latin-wght-normal.woff2`, 51 KB) and **`latin-ext`** (118 KB, fetched only when a glyph needs it), with `font-weight: 200 900`, `format('woff2-variations')`, and the `unicode-range` values copied from `wght.css`. Keep the family name `'Unbounded Variable'` as Fontsource emits it.
- Lato: import `@fontsource/lato/latin-400.css`, `latin-400-italic.css`, `latin-700.css` (and `latin-700-italic.css` only if used). Fontsource's Lato CSS also references legacy `.woff`; to ship woff2 only, write your own `@font-face` with `src: url('@fontsource/lato/files/lato-latin-400-normal.woff2') format('woff2')` (Vite resolves package paths).
- `font-display: swap`; `<link rel="preload" as="font" type="font/woff2" crossorigin>` for **Unbounded latin** and **Lato 400** only (declare in `index.html`; preload is not metadata so the "no meta in index.html" rule does not apply).
- Fallback stacks: `--font-display: 'Unbounded Variable', 'Arial Black', system-ui, sans-serif;` `--font-body: 'Lato', system-ui, sans-serif;`. Use `size-adjust` on a local fallback `@font-face` if CLS shows up in Lighthouse.
- Self-hosting means no third-party font requests, which keeps a strict CSP possible (`font-src 'self'`) and is LGPD-friendly (no Google Fonts IP leak).

## Q9. Lint/format: ESLint 10 flat + Prettier 3.9 (HIGH)

- Same toolchain as the backend (`eslint ^10.12`, `prettier ^3.9`), so one mental model, one set of editor settings. Add `.editorconfig` and reuse the backend's `.prettierrc`.
- Biome 2.6.2 is fast and tempting, but lacks `eslint-plugin-react-hooks` (incl. the React Compiler rules like `set-state-in-effect`, which fire in our lab), `eslint-plugin-testing-library`, `@tanstack/eslint-plugin-query`, and `eslint-plugin-playwright`. Those are exactly the guardrails TDD in JS needs without a type checker.
- **`react/prop-types` must be turned off** (verified: `recommended` fires `react/prop-types` on every component in a TS-less project) *or* PropTypes adopted everywhere. Decision: turn it off; correctness comes from tests and Zod at boundaries.
- Working flat config skeleton (executed under ESLint 10.13):

### What NOT to use

| Avoid | Why | Use instead |
|-------|-----|-------------|
| TypeScript, `@types/*`, PropTypes | Project decision (JS ESM); PropTypes add noise without a build-time benefit | Tests + Zod at API/form boundaries |
| `happy-dom` as default env | Verified: DOMPurify silently fails to sanitize; younger, less spec-strict; new SSRF-hardening major 11 days old | jsdom; per-file override only for isolated upload tests |
| `onUnhandledRequest` in MSW 3 | Silently ignored | `onUnhandledFrame: 'error'` |
| `@hookform/resolvers@6.0.0` | Fresh `npm i` fails `ERESOLVE` (verified) | `@5.9.2` |
| `eslint-plugin-vitest` 0.5.4 | Peer range ESLint 8/9 only | `@vitest/eslint-plugin` |
| `react-router-dom` | Removed in v8 | `react-router` + `react-router/dom` |
| React Router **framework mode** / Next / Remix | SSR/server runtime contradicts "nginx serves static build" | Data mode SPA + share shim |
| TanStack Router | Types are its selling point and are irrelevant in JS; codegen friction | RR 8 data mode |
| `react-helmet-async` | React 19 does it natively | `<title>/<meta>` in `<Seo>` |
| Static `<title>`/description/OG in `index.html` | Duplicates with React's hoisted tags (verified) | Only icons/viewport/charset in `index.html` |
| Tailwind, MUI, Chakra, shadcn defaults, Bootstrap | Produce the catalogued "AI slop" look; the brand manual defines a different language | CSS Modules + tokens |
| styled-components, emotion, vanilla-extract | Runtime or build complexity with no JS benefit; CSP friction | CSS Modules |
| Axios | No benefit over `fetch`; extra interceptor path in tests | `fetch` wrapper |
| Redux, Zustand, MobX | No client-state problem to solve | Query (server), router (URL), RHF (forms) |
| Formik, Enzyme, `jest`/`@jest/*`, `moment`, `date-fns` | Superseded / unnecessary | RHF, RTL, Vitest, `Intl.DateTimeFormat('pt-BR', { timeZone: 'America/Sao_Paulo' })` |
| `dangerouslySetInnerHTML` on `news.body` | Stored XSS (backend does not sanitize) | react-markdown with `allowedElements` |
| `new FormData(form)` for file inputs | Empty files under `user.upload()` in both DOM envs | File in state + manual `append` |
| `vitest-axe` 0.1.0 | Stale, jsdom has no layout/contrast | `@axe-core/playwright` |
| React Compiler (`compiler: true`, `oxc-transform-react`) | Experimental in plugin-react 6.1 | Plain React; revisit after launch |
| `localStorage` for session/user | Session is an HttpOnly cookie | `/me` through Query |
| Google Fonts `<link>` | Third-party request, LGPD, CSP | Fontsource self-hosting |

## Q10. Images: backend webp variants -> `srcset` (HIGH)

- `GET /api/v1/files/:id` serves the original; `/files/:id/thumb` (**400w**) and `/files/:id/medium` (**1200w**) are **generated on every request with sharp (`withoutEnlargement`, webp)** and sent with `Cache-Control: public, max-age=31536000, immutable`. Variants are not stored on disk. SVG is served as-is (admin-only uploads, CSP sandbox).
- `news.cover` / list cards give only a **URL string** (`/api/v1/files/12`), **no width/height**. Detail `images[].file` and `fileView` include `width`, `height`, `url`.
- Because originals are whatever the user uploaded (possibly multi-MB JPEG/PNG), **never render the bare original in a card or the article body.**
- Article cover/hero (above the fold, LCP): `loading="eager"`, `fetchPriority="high"`, `sizes="(min-width: 64rem) 960px, 100vw"`. Gallery thumbnails use `sizes` for their grid cell and link to `/medium`.
- **Layout shift:** since list/cover URLs carry no dimensions, give each image slot a fixed CSS `aspect-ratio` (e.g. `16 / 9`) with `object-fit: cover`. Where `width`/`height` are known (gallery images), also set the attributes.
- `withoutEnlargement` means a 300px original yields a 300px "thumb" while the descriptor says `400w`: harmless (the browser just uses a smaller image), but do not compute layout from the descriptor.
- A `medium` variant at 1200w is enough for 2x on a 600 CSS-px column; there is no larger variant, so full-bleed 1920px hero art needs the original (use only for admin-uploaded brand imagery, and have the client supply a pre-optimized file).
- **Operational dependency (infra phase):** since variants are computed per request, nginx should `proxy_cache` `/api/v1/files/` (they are immutable per id) or the first visits will burn CPU on sharp. Flag to the backend repo: persist variants on upload, or accept nginx cache.
- Brand assets (logos, GT illustrations) stay as bundled static assets in `src/assets/brand/` (prefer SVG; ask the client for vectors), imported through Vite so they get hashed names. Vite inlines assets < 4 KiB by default.

## Installation

# Runtime

# Build + test

# Lint / format

## Infrastructure notes that constrain the stack (MEDIUM)

| Concern | Decision |
|---------|----------|
| Static serving | nginx `root dist`, `try_files $uri /index.html` (SPA fallback); `/assets/*` (hashed) `Cache-Control: public, max-age=31536000, immutable`; `index.html` `no-cache` |
| API | `location /api/ { proxy_pass http://127.0.0.1:3333; }` + `proxy_set_header Host/X-Forwarded-Proto/X-Forwarded-For`; `client_max_body_size` aligned with backend upload limit |
| File variants | `proxy_cache` for `/api/v1/files/` (immutable, sharp is per-request) |
| CSP (security is continuous, not a final phase) | `default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data:; font-src 'self'; connect-src 'self'; frame-ancestors 'none'; base-uri 'self'; form-action 'self'`. Self-hosted fonts + bundled CSS make this feasible. React applies `style={}` through CSSOM, which `style-src` does not block, but **confirm in Playwright** with the CSP enabled in `vite preview` before relying on it |
| Share shim | Local Node process behind nginx bot map (Q6) |

## Alternatives Considered

| Category | Recommended | Alternative | Why Not / When |
|----------|-------------|-------------|----------------|
| Router | RR 8 data mode | TanStack Router 1.170.46 | Pick it only if the project moves to TypeScript and wants type-safe search params |
| Server state | TanStack Query | RR loaders+actions only | Fine for a tiny CRUD app; here caching/pagination/invalidation matter and RHF owns forms |
| Validation | Zod 4 | Valibot 1.5.0 | Smaller bundle; choose only if bundle size becomes a measured issue |
| Test DOM | jsdom 30 | happy-dom 21 | ~2.5x faster on role-heavy suites; adopt per-file only for upload form tests; revisit if DOMPurify-class bugs are fixed and speed becomes the bottleneck |
| Test DOM | jsdom 30 | Vitest Browser Mode | Real browser fidelity; heavier CI, defer |
| Body editor | Markdown textarea + preview | Tiptap 3.32 / MDXEditor 4.3.3 | WYSIWYG for non-technical editors if UAT demands; keep Markdown storage |
| Markdown render | react-markdown 10.1 | `marked` 18.1 + DOMPurify | Needs `innerHTML` + sanitizer (and DOMPurify is only reliable on jsdom/real browsers) |
| Styling | CSS Modules | vanilla-extract, Tailwind, CSS-in-JS | See Q5 |
| Lint | ESLint + Prettier | Biome 2.6.2 | Faster, one tool; lacks React-hooks-compiler/testing-library/query rules |
| Heading font | Unbounded | Michroma, Krona One | Michroma: single thin weight; Krona One: single weight, no kerning, quirky letterforms |
| Head management | React 19 native | react-helmet-async 3.0.0 | Only needed on React < 19 |
| SSR/OG | nginx + share shim | Prerender service (prerender.io), SSR framework | Paid/hosted dependency; or contradicts SPA decision |

## Version Compatibility

| Package A | Compatible With | Notes |
|-----------|-----------------|-------|
| vitest@5.0.5 | vite@8.3.5 (peer `^6.4 \|\| ^7 \|\| ^8`), Node 24 | `@vitest/coverage-v8` must equal 5.0.5 |
| @vitejs/plugin-react@6.1.2 | **vite ^8 only** | Using Vite 7 requires plugin-react 5 |
| react-router@8.5.0 | react >= 19.2.7, Node >= 22.22 | ESM-only; `RouterProvider` from `react-router/dom` |
| msw@3.0.3 | Node >= 22.12 | ESM-only; `msw/http`, `msw/node`; `onUnhandledFrame` |
| jsdom@30.1.3 | Node `^22.22.2 \|\| ^24.15 \|\| >=26` | Pin `.tool-versions` >= 24.15 |
| @testing-library/react@16.3.3 | react 18/19, `@testing-library/dom ^10` (install explicitly) | |
| @testing-library/jest-dom@7.0.1 | vitest >= 0.32, `@testing-library/dom >=10 <11`, Node >= 22 | |
| @hookform/resolvers@5.9.2 | react-hook-form ^7.55, zod ^3.25 \|\| ^4 | **Avoid 6.0.0** (ERESOLVE) |
| eslint@10.13.0 | react-hooks 7.1.1, react-refresh 0.5.7, query 5.104.5, testing-library 7.16.2 declare support | react 7.37.5 / jsx-a11y 6.10.2 need the `overrides` shown above |
| zod@4.6.7 | backend zod ^4.6.7 | Keep minor versions aligned |

## Empirical lab results (reproducible facts behind the verdicts)

- Same results in both: relative/absolute fetch via MSW, JSON/400/stream/blob, AbortSignal, `createMemoryRouter` and `createBrowserRouter` loaders with `<Link>` navigation and redirect guard, RR 8 middleware guard (anonymous -> `/login?next=`, user/admin role split), RHF+Zod with server field errors, constraint validation, React 19 head hoisting, Query + MSW, react-markdown safety.
- Only jsdom fails: multipart upload through `fetch` -> MSW (content-length mismatch; Vitest 5.0.5's compat shim yields an empty `blob` file with jsdom 30.1.3).
- Only happy-dom fails: DOMPurify (survivors: `onerror`, `javascript:`).
- Missing in jsdom, present in happy-dom: `showModal`, `matchMedia`, `IntersectionObserver`, `ResizeObserver`, `scrollIntoView`, `var()` in `getComputedStyle`, `inert`.
- Both: `new FormData(form)` loses the file set by `user.upload()`; `popover` missing; `DataTransfer` missing in jsdom.
- MSW 3: `onUnhandledRequest: 'error'` ignored (warn + passthrough); `onUnhandledFrame: 'error'` honoured.
- `@vitest-environment node` docblock works under Vitest 5; native `FormData`/`File` multipart through MSW works there.
- ESLint 10.13 with react 7.37.5 + jsx-a11y 6.10.2 + react-hooks 7.1.1 + react-refresh 0.5.7: runs; `npm i` needs the `overrides` shown.
- `@hookform/resolvers@6.0.0` in an empty dir with RHF 7.90 + Zod 4.6.7: `ERESOLVE`.
- Perf stress (300 tests x 200 links, `getAllByRole`): jsdom 33.2 s / 32.8 s vs happy-dom 13.2 s / 14.5 s.

## Gaps / items to validate in the phase that uses them

- **Share shim + OG image format** (LOW/MEDIUM): confirm WhatsApp/LinkedIn rendering of WebP `og:image` against the real domain; request a JPEG `og` variant from the backend if LinkedIn drops the image.
- **CSP vs React inline styles** (MEDIUM): verify under `vite preview` with the production CSP in Playwright before enabling `style-src 'self'` without `'unsafe-inline'`.
- **Vitest 5.0.5 jsdom multipart shim** (HIGH that it is broken with jsdom 30.1.3): worth a minimal repro upstream; re-test when upgrading Vitest/jsdom (a fix would let upload tests live in jsdom).
- **Lato versions:** Fontsource ships Google Fonts' Lato (v25 metadata, latin/latin-ext). If the brand wants the Lato 2.0 character set (extra glyph coverage), that is a different distribution; PT-BR is fully covered either way.
- **Backend follow-ups** (separate repo): `body`/`summary` max length, "Markdown" documented in spec, persisted or nginx-cached image variants, optional JPEG OG variant, `cover` carrying `width`/`height`.
- Query/RR **Suspense** usage (`useSuspenseQuery`) not evaluated; default `useQuery` + route-level pending UI is the baseline.

## Sources

- npm registry (`npm view`, 2026-10-22): versions, `engines`, `peerDependencies` for react, react-dom, vite, @vitejs/plugin-react, react-router, @tanstack/react-query, @tanstack/react-router, react-hook-form, zod, valibot, @hookform/resolvers, vitest, jsdom, happy-dom, msw, @testing-library/*, @playwright/test, eslint and plugins, @fontsource packages, react-markdown, dompurify, tiptap. HIGH.
- React Router CHANGELOG `v8.0.0` (https://github.com/remix-run/react-router/blob/main/CHANGELOG.md): baseline, removed flags, `react-router-dom` removal. HIGH.
- React Router middleware docs (https://reactrouter.com/how-to/middleware): data-mode `middleware`, `createContext`, redirect-from-middleware. HIGH.
- Vitest migration guide (https://vitest.dev/guide/migration): v5 breaking changes. HIGH.
- MSW 3.0 release notes (https://github.com/mswjs/msw/releases/tag/v3.0.0) and migration guide (https://mswjs.io/docs/migrations/2.x-to-3.x); quick start (https://mswjs.io/docs/quick-start). HIGH (and `onUnhandledFrame` reproduced).
- Vite 8 announcement (https://vite.dev/blog/announcing-vite8), build options (https://vite.dev/config/build-options), shared options (https://vite.dev/config/shared-options). HIGH.
- plugin-react CHANGELOG (https://github.com/vitejs/vite-plugin-react/blob/main/packages/plugin-react/CHANGELOG.md). HIGH.
- happy-dom v21.0.0 release (https://github.com/capricorn86/happy-dom/releases/tag/v21.0.0): CSS/iframe loading off by default (SSRF). MEDIUM.
- Vitest issue about jsdom FormData vs Node fetch (https://github.com/vitest-dev/vitest/issues/5672): root cause of multipart mismatch. MEDIUM; reproduced locally.
- React docs: `<title>` (https://react.dev/reference/react-dom/components/title), `<meta>` (https://react.dev/reference/react-dom/components/meta). HIGH; duplicate behaviour reproduced.
- WebP in `og:image`: https://www.ctrl.blog/entry/webp-ogp/ and HubSpot community thread on LinkedIn skipping WebP. LOW/MEDIUM, community sources.
- Fontsource API (https://api.fontsource.org/v1/fonts/{unbounded,michroma,krona-one,lato}) plus fontTools inspection of the shipped `latin` woff2 files (cmap, GPOS, advance widths) and a rendered specimen. HIGH.
- Backend source read directly: `~/Projects/chaphub-backend/src/{models/News.js, views/newsView.js, views/fileView.js, controllers/user/newsController.js, controllers/files/fileController.js, services/imageService.js, middlewares/validate.js, middlewares/errors.js, utils/HttpError.js, utils/pagination.js, app.js, config/env.js}`, `docs/PLANO.md`, `package.json`, `CLAUDE.md`. HIGH.
- Project docs: `.planning/PROJECT.md`, `docs/ANALISE-PROTOTIPO.md`. HIGH.

<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->

## Conventions

Conventions not yet established. Will populate as patterns emerge during development.
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->

## Architecture

Architecture not yet mapped. Follow existing patterns found in the codebase.
<!-- GSD:architecture-end -->

<!-- GSD:skills-start source:skills/ -->

## Project Skills

No project skills found. Add skills to any of: `.claude/skills/`, `.agents/skills/`, `.cursor/skills/`, `.github/skills/`, or `.codex/skills/` with a `SKILL.md` index file.
<!-- GSD:skills-end -->

<!-- GSD:workflow-start source:GSD defaults -->

## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd-fast` for a trivial task inline, with no subagents and no PLAN.md
- `/gsd-quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd-debug` for investigation and bug fixing
- `/gsd-execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->

<!-- GSD:profile-start -->

## Developer Profile

> Profile not yet configured. Run `/gsd-profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->
