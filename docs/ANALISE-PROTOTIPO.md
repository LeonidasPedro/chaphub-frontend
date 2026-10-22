# Análise do protótipo (`~/Projects/chaphub`)

> Levantado em 2026-10-22. Serve de **descrição** do que existe; o novo frontend é escrito do zero a partir deste documento, sem copiar código. Requisitos: Obsidian `chaphub/Requisitos.md`.

## 1. O que o protótipo é

Landing única (React 18 + Vite 6, deploy Cloudflare Workers), sem roteador e sem chamadas de rede. Conteúdo fixo em `src/data/*.js` e dentro dos componentes.

| # | Seção | Conteúdo | No novo front |
|---|---|---|---|
| 1 | Nav | Logo, âncoras Sobre/GTs/Localização/Contato, CTA "Faça Parte", hambúrguer | Nav com rotas reais (Notícias, Eventos, GTs, Documentos) + login discreto |
| 2 | Hero | "Incentivar Mentes Inovadoras para Desenvolver Chapecó do Futuro", texto digitado, partículas em canvas, triângulo flutuante, 2 CTAs | Manter a frase; trocar efeitos por composição de marca (bloco de cor + foto da cidade) |
| 3 | Sobre | Parágrafo + órbita SVG animada com 3 GTs fixos | Texto; diagrama de GTs gerado da API (n GTs) |
| 4 | Números | 5 cards com contador animado | Indicadores da API (`/indicadores`), apresentados com contexto (fonte, data) |
| 5 | GTs | 3 cards iguais: logo, texto, tags, logos das instituições | Lista/página por GT (`/work-groups/:slug`) |
| 6 | Chapecó | Texto + 3 dados + contorno de SC (IBGE) | Manter o mapa de SC (path SVG é dado geográfico, reaproveitável) |
| 7 | Governança | Grade de logos | `/governance` |
| 8 | Contato | LinkedIn, Instagram, endereço (formulário removido) | Igual, no rodapé |
| 9 | Footer | Logo, frase, logos dos GTs, "© 2024" | Ano dinâmico |

Conteúdo textual útil (copiar como **texto**, não como código): frase do hero, descrição "Primeiro ecossistema de inovação do Oeste Catarinense…", textos dos GTs, redes sociais (`linkedin.com/in/chaphub-ecossistema-de-inovação-de-chapecó-193978356`, `instagram.com/chaphub_`). GTs, instituições, governança e indicadores agora vêm da API.

## 2. Gambiarras e erros a não repetir

- `useSmoothScroll`: intercepta todo `a[href^="#"]` no documento e desconta "16px" mágicos do header. → Usar `scroll-margin-top` em CSS.
- `useFadeIn`: cada seção chama o hook e varre `document.querySelectorAll('.fade-in')` global. → Nada de animação de entrada em todo elemento.
- Partículas em canvas + overlay radial + triângulo flutuante + seta quicando: custo de CPU, zero informação.
- `OrbitaEcossistema` com 3 nós fixos (quebra com GTs dinâmicos, RF03) e `fontFamily="sans-serif"` hardcoded.
- PIB divergente: R$ 176,6 bi (Números) × R$ 17,6 bi (Chapecó). Fonte única agora é o backend.
- Delays de animação dentro dos dados (`delay: '0.08s'` em `gts.js`).
- Estilos inline em JSX (`style={{ transitionDelay }}`), CSS global de 22 KB num arquivo só.
- Paths com espaço e acento (`/chaphub/CLIENTE ECOSSISTEMA/Logos do Ecossistema de Inovação png/SAÚDE-branca.png`).
- `public/` com PDF de 9 MB e zips de fontes DEMO (Organetto, Acumin) — iriam para produção.
- `aria-label="Menu"` sem `aria-expanded`; links externos sem `noreferrer`; "© 2024".
- Título de Organetto substituído por Archivo/Saira sem decisão registrada (README e nota divergem).

## 3. Padrões "AI slop" presentes

Catálogos consultados: [pixelslop](https://cdn.jsdelivr.net/npm/pixelslop@0.3.8/dist/skill/resources/ai-slop-patterns.md), [925studios](https://www.925studios.co/blog/ai-slop-design-tells), [prg.sh](https://prg.sh/ramblings/Why-Your-AI-Keeps-Building-the-Same-Purple-Gradient-Website), [noqta](https://noqta.tn/en/blog/ai-design-slop-overused-ui-patterns-fix-2026), [superdesign](https://www.superdesign.dev/blog/why-ai-design-looks-generic).

| Padrão | Onde no protótipo |
|---|---|
| Fundo quase preto com brilhos/halos | `hero__overlay`, `::before` radiais em Sobre/GTs, `gt-card__glow`, filtros `feGaussianBlur` |
| Ciano neon em fundo escuro | Toda a paleta de destaque |
| Hero genérico: título + subtítulo + 2 CTAs + ilustração flutuante | Hero |
| Grade de cards idênticos (3 ou 5) | GTs, Números |
| Métrica gigante com rótulo pequeno + ícone acima | `NumeroCard` |
| Tudo centralizado | `.section-header { text-align: center }` |
| Botões pílula `border-radius: 50px`, todos em caixa alta | `.btn` |
| Raio uniforme 12/20px + sombra suave em tudo | cards |
| Hover `translateY` + sombra em todo card | `.gt-card:hover`, `.numero-card:hover`, `.btn:hover` |
| `backdrop-filter: blur` na nav | `.nav.scrolled` |
| Fade-in ao rolar em todo bloco | `.fade-in` |
| Texto digitado com cursor piscando | `useTyped` |
| Barra de destaque que cresce no hover | `gt-card__bar`, `numero-card::before` |
| Rótulo pequeno espaçado acima de título com palavra colorida | `.section-label` + `.text-cyan` em toda seção |
| Texto cinza translúcido em fundo escuro | `rgba(255,255,255,0.55)` em vários |

## 4. O manual da marca (PDF do cliente, 38 págs) — a referência real

- **Palavras-chave:** Inovação, Desenvolvimento, Compartilhamento, Networking, Criatividade, Experiência.
- **Símbolos:** casa enxaimel/triângulo (compartilhar, reunir), engrenagens (processo, agir), triângulo sólido (unidade, força, estabilidade). Marca = triângulo em **3 segmentos chanfrados** separados.
- **Cores:** preto, ciano, roxo, branco. Usadas como **blocos chapados de página inteira** (capa ciano, capa roxa, metade preto/metade foto). Degradê oficial é longo e horizontal: preto → ciano → roxo → branco desbotado. Não é "ciano→roxo" em botão.
- **Tipografia:** Organetto (caixa alta, extra larga), Lato (itálico como voz da marca, ex.: "Ecossistema de Inovação de Chapecó"), Acumin. Organetto e Acumin **não podem ser usadas** (licença).
- **Ilustração:** ícones dos GTs em **traço único contínuo de contorno duplo** (porco + planta = Agro, cruz + estetoscópio = Saúde, livro + capelo = Educação/Câmara, circuito = TIC, engrenagem + lâminas = Máquinas). Linguagem própria, não ícone de biblioteca.
- **Composição:** split vertical (cor sólida | foto), logo + divisória vertical fina + sub-marca, padrão geométrico de semicírculos (preto/ciano/roxo/branco) em peças gráficas.
- **Fotografia:** Chapecó real (estátua do Desbravador), campo, saúde — com o triângulo em contorno branco sobreposto.

### Direção para o novo design (proposta)

- Blocos de cor chapados e cortes retos; zero glow, zero blur, zero sombra decorativa.
- Alinhamento à esquerda e grade assimétrica; cards só onde há item clicável (notícia, evento).
- Ilustrações de traço único dos GTs como elemento gráfico grande (não ícone pequeno).
- Contorno do triângulo sobre fotos reais, como no manual.
- Lato itálico para frases de voz; título em fonte livre larga (Unbounded, Michroma ou Krona One).
- Botões retangulares com hierarquia (1 primário por tela).
- Movimento só para mudança de estado; respeitar `prefers-reduced-motion`.
- Fundo: off-black `#06070C` do manual, não `#000`.

## 5. Assets a importar (só estes)

| Origem (protótipo) | Destino sugerido | Motivo |
|---|---|---|
| `CLIENTE ECOSSISTEMA/Logos…/CHAPHUB-{horizontal,vertical}-{branca,preta}.png` | `src/assets/brand/` (renomear sem espaço/acento) | Logo |
| `…/{AGRO,SAÚDE,EDUCAÇÃO}-*.png` | `src/assets/brand/work-groups/` | Logos de GT (EDUCAÇÃO-branca existe, ao contrário do que diz a spec) |
| `CLIENTE ECOSSISTEMA/chaphub-{1..4,branco*,preto*}.png` | avaliar | Variações de marca |
| `public/favicon.ico`, `favicon-256.png` | `public/` | Favicon |
| Path SVG de SC (`SCOutlineMap.jsx`, dado IBGE) | `src/assets/sc-outline.svg` | Dado geográfico |
| `public/instituicoes/*`, `public/governanca/*` | **não** — vêm de upload via API | Logos de terceiros |
| PDF, zips de fontes | **não** | Peso e licença |

Ideal: pedir ao cliente os logos/ilustrações em **SVG** (traço único dos GTs, triângulo) — PNG limita uso em grande escala.
