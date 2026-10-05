# Briefing de implementação — LP Aula Individual de Jiu-Jitsu (Thierry Reis)

Página de vendas one-page, estilo LP com VSL, pra vender a aula individual do Thierry Reis (faixa preta, sócio e professor da SPARTA Escola de Artes Marciais, RJ). Objetivo único: clique no botão de WhatsApp pra agendar a aula de diagnóstico.

Este briefing documenta tudo que o dev precisa pra implementar a página no domínio próprio. A referência visual final está no ar em https://tiago1002bravy.github.io/lp-thierry-reis/ e o código-fonte é o `index.html` deste repositório (HTML estático, zero dependência externa, zero framework).

## Arquivos deste repositório

| Arquivo | O que é |
|---|---|
| `index.html` | Página completa (HTML + CSS + JS inline), pronta pra servir estática |
| `fonts.css` | Barlow Condensed 700 e Archivo (variável) embutidas em base64 woff2 |
| `img/` | Todas as imagens otimizadas pra web (JPG q78-82, máx 1400px) |
| `assets-src/` | PNGs originais em alta das imagens geradas por IA (faixas, medalhas, arena) |
| `lp-thierry-standalone.html` | Arquivo ÚNICO com tudo embutido (3,5MB). Abre em qualquer lugar, bom pra passar por WhatsApp/e-mail |
| `BRIEFING.md` | Este documento |

## Identidade visual

Segue a identidade da SPARTA/SPRT+: dark total, monocromático preto e branco, estética de streaming esportivo (referência: UFC Fight Pass, Netflix). Sem cor de acento, sem gradiente colorido, sem emoji, cantos retos (raio máximo 2px).

### Tokens de cor

```css
--ink0:  #0A0A0B;  /* fundo da página */
--ink1:  #131316;  /* superfície de card */
--ink2:  #1A1A1E;  /* superfície elevada / hover de botão */
--line:  #26262A;  /* hairlines e bordas */
--paper: #F4F4F5;  /* branco principal (texto, botões) */
--mut:   #B9B9BE;  /* texto secundário */
--dim:   #8E8E95;  /* legendas e eyebrows */
--ghost: #3A3A40;  /* números de card, aspas gigantes */
```

### Tipografia

- **Display:** Barlow Condensed 700 (Google Fonts), sempre CAIXA ALTA, leading .9-.95. Fallback: `'Arial Narrow', sans-serif`. Usada em todos os títulos, botões, números e chips.
- **Corpo:** Archivo 400/600 (Google Fonts). Fallback: `-apple-system, 'Helvetica Neue', sans-serif`.
- **Eyebrows/labels:** Archivo 600, 12px, `letter-spacing: .2em`, uppercase, cor `--dim`, com um traço branco de 26x2px antes (e depois, quando centralizado).
- Headline do hero: `clamp(50px, 9vw, 116px)`. H2 de seção: `clamp(36px, 5.4vw, 64px)`. Corpo: 16px/1.6.
- **Texto vazado (outline):** trechos de destaque nas headlines usam `color: transparent; -webkit-text-stroke: 2px #F4F4F5` (no hero: "O quase acaba aqui."; no CTA final: "Ou dá pra fazer isso em 1.").
- No site em produção, carregar as fontes do Google Fonts normalmente (o base64 do `fonts.css` só existe porque o preview rodava com CSP restrita).

### Layout

Container de 1200px centralizado, padding lateral `clamp(20px, 5vw, 48px)`. Seções separadas por hairline `1px --line`, padding vertical `clamp(64px, 10vw, 120px)`. Mobile-first, grids com `auto-fit/minmax` (nunca media query de coluna fixa).

## Estrutura da página (ordem das seções)

1. **Header sticky fino:** logo SPARTA branca à esquerda, botão branco "AGENDAR AULA" à direita. Fundo `rgba(10,10,11,.9)` com `backdrop-filter: blur(12px)`.
2. **Hero:** arena P&B de fundo, eyebrow, H1, sub, player da VSL, CTA, linha de confiança ("Faixa preta · Campeão Sul-Americano · SPARTA, Barra da Tijuca").
3. **Ticker (marquee):** faixa com os 7 blocos do método rolando infinito.
4. **A dor:** split 2 colunas, headline + texto com trecho marcado + punchline.
5. **O método:** 6 cards numerados (01-06).
6. **Pra quem é:** 4 cards com foto de faixa no topo (branca, azul, medalhas+troféu, roxa+marrom).
7. **Como funciona:** 4 passos com linha que se desenha + número vazado.
8. **O professor:** foto emoldurada + bio + 3 badges de título.
9. **Depoimentos:** carrossel com 3 cards (foto + citação), autoplay.
10. **FAQ:** 5 perguntas em accordion (`<details>/<summary>`).
11. **CTA final:** arena de fundo, headline gigante, botão.
12. **Footer:** logo + endereço completo.
13. **Barra fixa mobile:** CTA que sobe do rodapé após 600px de scroll (só < 900px).

A copy é a do `index.html`, palavra por palavra. Não reescrever.

## Efeitos visuais (a alma da página)

Todos os efeitos respeitam `prefers-reduced-motion: reduce` (viram estado estático) e a página funciona 100% sem JavaScript: a classe `anim` só entra no `<html>` via JS, e todo estado "escondido" de animação vive atrás de `.anim`, então sem JS nada fica invisível.

### 1. Grão de filme global
`body::after` fixo cobrindo a viewport, `opacity: .05`, `pointer-events: none`, com SVG de `feTurbulence` (fractalNoise, baseFrequency .85) como data URI. Dá textura de filme sem pesar nada.

### 2. Sequência de entrada do hero
Os 6 filhos diretos do hero (eyebrow, H1, sub, VSL, CTA, trust) sobem em cascata: `translateY(26px) + opacity 0 → 0`, animação `rise` de .85s com easing `cubic-bezier(.2,.7,.2,1)`, delays escalonados de .05s a .8s.

### 3. Zoom lento da arena
A imagem de fundo do hero faz `scale(1) → scale(1.1)` em 26s, `ease-out forwards`. Overlay duplo por cima: radial do topo + linear pra preto no pé, pra fundir com a página.

### 4. Ticker (marquee)
Lista dos 7 blocos do método duplicada 2x dentro de um track `width: max-content`, animação `translateX(0 → -50%)` em 30s linear infinito. Separador: quadradinho 8x8 `--ghost` entre os itens. Barlow Condensed 19px, letter-spacing .18em, cor `--dim`.

### 5. Reveal on-scroll com stagger
Elementos `.rv` nascem `opacity 0 / translateY(26px)` e ganham `.in` via IntersectionObserver (`rootMargin: 0 0 -8%`), transição .75s. Delay por elemento via atributo `data-d` (ex. `.08`, `.16`, `.24`) lido pelo JS e aplicado como `transition-delay` em custom property `--d`.

### 6. Linhas que se desenham (Como funciona)
Cada passo tem um `::before` de 2px branco que faz `scaleX(0) → scaleX(1)` (transform-origin left, 1s) quando entra na viewport, com delay progressivo (0/.15/.3/.45s). Números dos passos em texto vazado (stroke 1.5px).

### 7. Moldura editorial das fotos (`.frame`)
- Cantoneiras: dois pseudo-elementos de 26x26px com borda branca 2px em L, no canto superior esquerdo e inferior direito, 10px pra dentro.
- Degradê escuro na base (`linear-gradient 185deg, transparent 55% → rgba(0,0,0,.72)`).
- Chip de contexto no canto inferior esquerdo: fundo `rgba(10,10,11,.82)` + borda `rgba(255,255,255,.22)` + `backdrop-filter: blur(4px)`, Barlow 13px uppercase com quadradinho branco 7x7 antes. Textos: "1º LUGAR · COPA NOVO LEBLON", "MEDALHISTA · BRASILEIRO CBJJ", "DO ZERO À FAIXA AZUL", "SUL-AMERICANO IBJJF · OURO".
- Foto em meio P&B: `filter: grayscale(55%) contrast(1.06)`, indo pra `grayscale(0)` no hover com transição .5s.

### 8. Parallax sutil
Imagens com `data-plx` (foto do professor, fotos dos depoimentos, arena do CTA) transladam no eixo Y proporcional à posição na viewport: `translateY(distância_do_centro * -26px) scale(1.12)`, atualizado em `requestAnimationFrame` no scroll. O `scale(1.12)` evita aparecer borda.

### 9. Botões CTA
Brancos com texto preto, Barlow 700 uppercase. Efeito shine: pseudo-elemento em degradê escuro translúcido inclinado (skewX -18deg) varrendo da esquerda pra direita a cada 3.6s. Hover: sobe 1px e a seta `→` desliza 5px.

### 10. Play da VSL
Círculo branco com triângulo preto, anel pulsante via box-shadow (`0 → 26px` de spread transparente, 2.4s infinito). O player é placeholder: thumb + barra fake de progresso; clique mostra overlay "VÍDEO EM BREVE". **Na implementação final, trocar por embed real do vídeo** (YouTube unlisted, Vimeo ou player próprio), mantendo moldura com cantoneiras e proporção 16:9.

### 11. Palavras vazadas de fundo (ghostwords)
Palavras gigantes (`clamp(96px, 17vw, 230px)`) em Barlow vazado com stroke `rgba(255,255,255,.06)`, absolutas no topo direito de 3 seções: MÉTODO, FAIXA PRETA, PÓDIO. `pointer-events: none`, seção com `overflow: hidden`.

### 12. Cards com hover
Cards do método: barra branca de 3px à esquerda cresce de cima pra baixo no hover, número 01-06 acende de `--ghost` pra branco, card sobe 3px. Cards de faixa: foto dá zoom 1.06 em .6s e o card sobe 4px.

### 13. Carrossel de depoimentos
- Scroll-snap horizontal nativo (`scroll-snap-type: x mandatory`, cada slide `flex: 0 0 100%`), swipe natural no mobile, scrollbar escondida.
- Controles no cabeçalho da seção: setas ← → em botões quadrados 52px e contador "1 / 3" (número atual em branco, resto em `--dim`, `tabular-nums`).
- **Autoplay:** avança a cada 6s em loop infinito (módulo), pausa com mouse em cima, pausa 12s após toque no mobile, pausa com aba em segundo plano (`document.visibilityState`), desligado em reduced-motion. Clicar numa seta reinicia o timer.
- Ordem dos slides (por faixa, da maior pra menor): 1º faixa marrom (medalhista Brasileiro CBJJ no-gi), 2º faixa azul (depoimento do empresário), 3º Copa Novo Leblon (campeão faixa branca).
- Divisor de citação: "belt-rule", barra de 88x6px branca com ponteira cinza de 26px (imita a ponteira da faixa preta).

### 14. Interface de leitura
- Barra de progresso de scroll: 2px branca fixa no topo, largura = % da página rolada.
- Header some ao rolar pra baixo (translateY -100%, .35s) e volta ao rolar pra cima.
- FAQ: `<details>` nativo, símbolo `+` gira 45° virando `✕` quando aberto.
- Barra CTA fixa no mobile aparece após 600px de scroll.

## Acessibilidade e qualidade

- `prefers-reduced-motion: reduce` desliga TODAS as animações (reveals ficam visíveis, autoplay do carrossel não roda).
- Sem JS a página renderiza completa (nenhum conteúdo nasce invisível).
- `:focus-visible` com outline branco 2px em todo elemento interativo.
- `alt` descritivo em todas as fotos; botões do carrossel com `aria-label`.
- Fotos de iPhone (HEIC→JPG): **remover a tag EXIF de orientação antes de publicar** (sips/Preview mantém a tag e o navegador rotaciona de novo; limpar metadata com PIL/exiftool).
- `lang="pt-BR"`, `<meta charset="utf-8">`, viewport mobile.

## Pendências pro go-live no domínio próprio

1. **WhatsApp:** trocar `https://wa.me/5500000000000` pelo número real do Thierry (aparece em 4 lugares: header, CTA do hero, CTA final e barra mobile). Sugestão: incluir `?text=` com mensagem pré-preenchida tipo "Oi Thierry! Quero agendar uma aula individual de jiu-jitsu."
2. **VSL:** gravar o vídeo e trocar o placeholder por embed real.
3. **Depoimentos:** os 2 longos são falas reais de alunos enviadas pelo Tiago (o da faixa azul foi resumido, vale o aluno bater o olho); a frase curta do card do Brasileiro é descritiva, não é citação de aluno.
4. **SEO/compartilhamento:** adicionar `<meta name="description">`, Open Graph (og:title, og:description, og:image com a foto do Thierry campeão) e favicon próprio.
5. **Deploy:** é estático puro, serve em qualquer lugar (Vercel, Netlify, Cloudflare Pages, nginx). Basta subir `index.html` + `fonts.css` + `img/`.
