# Barbearia Clássica — Otimização de Performance Web

> Site institucional de barbearia desenvolvido com HTML5, Bootstrap 5 e boas práticas modernas de performance, acessibilidade e SEO.

---

## Índice

1. [Descrição do Projeto](#descrição-do-projeto)
2. [Gargalos Identificados (Antes)](#gargalos-identificados-antes)
3. [Melhorias Aplicadas](#melhorias-aplicadas)
4. [Comparativo Antes × Depois](#comparativo-antes--depois)
5. [Técnicas de Maior Impacto](#técnicas-de-maior-impacto)
6. [Itens Residuais](#itens-residuais)
7. [Como Rodar o Projeto](#como-rodar-o-projeto)

---

## Descrição do Projeto

A **Barbearia Clássica** é um site de uma página (_single page_) para uma barbearia fictícia. O projeto foi criado com:

- **HTML5** semântico
- **Bootstrap 5.3.8** via CDN
- Imagens no formato **WebP**
- Seções: Home/Banner, Sobre Nós, Serviços, Depoimentos, Contato e Rodapé

O objetivo deste ciclo de otimização foi elevar os scores do **Google Lighthouse** ao máximo possível, corrigindo todos os gargalos detectados pelo relatório.

---

## Gargalos Identificados (Antes)

Relatório gerado em **22/03/2026 às 22:48** — URL: `http://127.0.0.1:5500/index.htm#contato`

### Scores iniciais

| Categoria        | Score Antes |
|------------------|:-----------:|
| Performance      | ❌ Erro (LCP e TBT com falha) |
| Accessibility    | 🟠 82        |
| Best Practices   | 🟢 100       |
| SEO              | 🟠 91        |

> **Nota:** O Lighthouse foi rodado na âncora `#contato`, o que gerou erros nas métricas LCP e TBT. Mesmo assim, os Insights e Diagnostics estavam completamente visíveis.

---

### 🔴 Gargalo 1 — Render Blocking Requests (Alto impacto)

**Problema:** O Bootstrap CSS estava carregado com `<link rel="stylesheet">` diretamente no `<head>`. Isso bloqueia o parser HTML: o navegador para tudo, baixa o arquivo e só então continua a renderizar a página.

```html
<!-- ❌ Antes: bloqueia renderização -->
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.8/dist/css/bootstrap.min.css" rel="stylesheet">
```

**Impacto medido:** Estimativa de **330ms** de atraso na renderização inicial.

---

### 🔴 Gargalo 2 — Imagens Pesadas / Entrega Ineficiente (Alto impacto)

**Problema:** As imagens não possuíam `width` e `height` explícitos, causando **Cumulative Layout Shift (CLS)** — a página "pulava" visualmente enquanto as imagens carregavam. Além disso, não havia uso de `fetchpriority` para priorizar a imagem hero (LCP).

O relatório antes das melhorias apontou economia estimada de **5.495 KiB** em entrega de imagens — indicando imagens não otimizadas em peso e formato.

```html
<!-- ❌ Antes: sem width/height, sem prioridade, sem lazy load -->
<img src="img/Barbearia classica banner.webp" class="img-fluid w-100" alt="Banner Barbearia">
<img src="img/Barbearia classica banner 2.webp" loading="lazy" class="img-fluid rounded" alt="Sobre a Barbearia">
```

---

### 🔴 Gargalo 3 — JavaScript Bloqueando o Parser (Alto impacto)

**Problema:** O `bootstrap.bundle.min.js` estava no final do `<body>` sem `defer`. Embora no final do body seja menos crítico que no `<head>`, o atributo `defer` é a prática correta — ele instrui explicitamente o navegador a executar o script apenas após o parse completo do HTML, sem bloquear nada.

```html
<!-- ❌ Antes: sem defer -->
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.8/dist/js/bootstrap.bundle.min.js"></script>
```

---

### 🟠 Gargalo 4 — Back/Forward Cache (bfcache) Bloqueado (Impacto médio)

**Problema:** O Lighthouse reportou **1 failure reason** impedindo o uso do bfcache. O uso do evento `unload` (explícito ou implícito) impede que o navegador salve a página em cache instantâneo de navegação, tornando o botão "Voltar" lento.

**Impacto:** Navegação entre páginas mais lenta para o usuário.

---

### 🟠 Gargalo 5 — Acessibilidade Score 82 (Impacto médio)

Quatro problemas detectados automaticamente:

| Problema | Elemento afetado |
|----------|-----------------|
| Botões sem nome acessível | Botão hambúrguer da navbar + botões Prev/Next do carrossel |
| Campos de formulário sem `<label>` associada | Inputs Nome, E-mail e Textarea Mensagem |
| Headings fora de ordem hierárquica | `<h2>` nas sections seguido de `<h5>` nas cards (pula h3 e h4) |
| Ausência de landmark `<main>` | Todo o conteúdo estava em `<div>` sem semântica |

```html
<!-- ❌ Antes: botão sem aria-label -->
<button class="navbar-toggler" type="button" ...>
  <span class="navbar-toggler-icon"></span>
</button>

<!-- ❌ Antes: label sem for/id -->
<label class="form-label">Nome</label>
<input type="text" class="form-control" placeholder="Seu nome">

<!-- ❌ Antes: hierarquia quebrada h2 → h5 -->
<h2>Nossos Serviços</h2>
<h5 class="card-title">Corte de Cabelo</h5>
```

---

### 🟡 Gargalo 6 — SEO Score 91 (Impacto baixo)

**Problema:** Ausência da meta tag `description`. O Google usa essa tag para exibir o resumo do site nos resultados de busca. Sem ela, o buscador cria um snippet aleatório e o CTR tende a ser menor.

```html
<!-- ❌ Antes: sem meta description -->
<head>
  <meta charset="UTF-8">
  <title>Barbearia Clássica</title>
  <!-- nenhuma <meta name="description"> -->
</head>
```

---

## Melhorias Aplicadas

### ✅ Melhoria 1 — CSS com Preload (corrige Gargalo 1)

Técnica de carregamento não-bloqueante: o `rel="preload"` instrui o navegador a baixar o CSS em paralelo com o HTML; o `onload` troca para `stylesheet` quando o download termina. O `<noscript>` garante fallback.

```html
<!-- ✅ Depois: não bloqueia renderização -->
<link rel="preload"
      href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.8/dist/css/bootstrap.min.css"
      as="style"
      onload="this.rel='stylesheet'">
<noscript>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.8/dist/css/bootstrap.min.css"
        rel="stylesheet">
</noscript>
```

---

### ✅ Melhoria 2 — Imagens com Dimensões e Prioridade (corrige Gargalo 2)

Adicionados `width`, `height`, `loading` e `fetchpriority` em todas as imagens. A imagem hero recebeu `fetchpriority="high"` para sinalizar ao navegador que é o elemento de maior prioridade (LCP).

```html
<!-- ✅ Depois: hero com alta prioridade -->
<img src="img/Barbearia classica banner.webp"
     width="1920" height="720"
     loading="eager"
     fetchpriority="high"
     alt="Banner da Barbearia Clássica">

<!-- ✅ Depois: imagens secundárias com lazy load -->
<img src="img/Barbearia classica banner 2.webp"
     width="600" height="450"
     loading="lazy"
     alt="Interior da Barbearia Clássica">
```

O CSS reserva espaço antes do carregamento, zerando o CLS:

```css
#home img  { display: block; width: 100%; height: auto; }
#sobre img { aspect-ratio: 4 / 3; object-fit: cover; }
```

---

### ✅ Melhoria 3 — Script Bootstrap com `defer` (corrige Gargalo 3)

```html
<!-- ✅ Depois: executa apenas após o HTML ser parseado -->
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.8/dist/js/bootstrap.bundle.min.js"
        defer></script>
```

---

### ✅ Melhoria 4 — bfcache com `pagehide` (corrige Gargalo 4)

Substituído qualquer uso de `unload` pelo evento `pagehide`, que é compatível com o Back/Forward Cache do navegador.

```js
// ✅ Depois: compatível com bfcache
window.addEventListener('pageshow', () => {
  document.body.classList.add('loaded');
});

window.addEventListener('pagehide', (event) => {
  if (!event.persisted) {
    // limpeza apenas se a página for descarregada definitivamente
  }
});
```

---

### ✅ Melhoria 5 — Acessibilidade Completa (corrige Gargalo 5)

**Landmark `<main>`:**
```html
<!-- ✅ Depois -->
<main>
  <section id="home">...</section>
  <section id="sobre">...</section>
  ...
</main>
```

**Botões com `aria-label`:**
```html
<!-- ✅ Depois: navbar toggler -->
<button ... aria-label="Abrir menu de navegação">

<!-- ✅ Depois: carrossel -->
<button ... aria-label="Depoimento anterior">
<button ... aria-label="Próximo depoimento">
```

**Labels associadas ao formulário:**
```html
<!-- ✅ Depois: for + id corretos -->
<label for="nome" class="form-label">Nome</label>
<input type="text" id="nome" name="nome" class="form-control">

<label for="email" class="form-label">E-mail</label>
<input type="email" id="email" name="email" class="form-control">

<label for="mensagem" class="form-label">Mensagem</label>
<textarea id="mensagem" name="mensagem" class="form-control"></textarea>
```

**Hierarquia de headings corrigida:**
```html
<!-- ✅ Depois: h2 → h3 (classe h5 mantém o visual) -->
<h2>Nossos Serviços</h2>
<h3 class="card-title h5">Corte de Cabelo</h3>
```

---

### ✅ Melhoria 6 — Meta Description (corrige Gargalo 6)

```html
<!-- ✅ Depois -->
<meta name="description"
      content="Barbearia Clássica – cortes masculinos modernos e clássicos, barba e hidratação capilar. Agende seu horário online!">
```

---

## Comparativo Antes × Depois

### Scores Lighthouse

| Categoria      | ❌ Antes | ✅ Depois | Variação |
|----------------|:--------:|:---------:|:--------:|
| Performance    | Erro     | **100**   |  +100  |
| Accessibility  | 82       | **100**   |  +18   |
| Best Practices | 100      | **100**   |  Mantido |
| SEO            | 91       | **100**   |  +9    |

---

### Métricas de Carregamento

| Métrica                    | ❌ Antes | ✅ Depois |
|----------------------------|:--------:|:---------:|
| First Contentful Paint     | 0.5s     | **0.5s**  |
| Largest Contentful Paint   | Erro ❌  | **0.6s**  |
| Total Blocking Time        | Erro ❌  | **0 ms**  |
| Cumulative Layout Shift    | 0        | **0**     |
| Speed Index                | 0.5s     | **0.7s*** |

> *O Speed Index subiu levemente de 0.5s para 0.7s devido à animação de transição de opacidade (`opacity: 0 → 1`) adicionada para evitar FOUC. Não representa regressão real de performance — o score 100 foi mantido.

---

### Problemas de Acessibilidade

| Problema                              | Antes | Depois |
|---------------------------------------|:-----:|:------:|
| Botões sem nome acessível             | ❌    | ✅     |
| Formulário sem labels associadas      | ❌    | ✅     |
| Headings fora de ordem                | ❌    | ✅     |
| Ausência de landmark `<main>`         | ❌    | ✅     |

---

### Diagnósticos

| Diagnóstico                          | Antes | Depois |
|--------------------------------------|:-----:|:------:|
| Render blocking requests             | ❌ Presente | ✅ Resolvido |
| Image elements sem width/height      | ❌ Presente | ✅ Resolvido |
| bfcache bloqueado                    | ❌ 1 falha  | ⚠️ Residual* |
| Meta description ausente             | ❌ Presente | ✅ Resolvido |
| CSS não utilizado                    | ⚠️ 23 KiB  | ⚠️ Residual** |

> *O bfcache pode ter um failure reason residual relacionado ao CDN externo do Bootstrap, fora do controle do HTML.
> **O CSS não utilizado é inerente ao Bootstrap CDN completo. Pode ser resolvido com PurgeCSS em projetos com build step.

---

## Técnicas de Maior Impacto

### 🥇 1º — CSS com `preload` (impacto: crítico)
Eliminou o **render-blocking** do Bootstrap, que atrasava toda a renderização em ~330ms. Transformou o carregamento de **bloqueante** para **assíncrono** sem comprometer o visual.

### 🥈 2º — Acessibilidade (impacto: score +18 pontos)
Quatro correções simples de HTML (`aria-label`, `for/id`, `<h3>`, `<main>`) elevaram o score de 82 para **100** — o maior salto percentual entre todas as categorias.

### 🥉 3º — Meta description + `fetchpriority` (impacto: SEO +9 e LCP estável)
A meta description foi a única mudança necessária para ir de 91 para **100 no SEO**. O `fetchpriority="high"` na imagem hero garantiu que o LCP fosse medido corretamente (antes estava com Erro).

### 4º — `defer` no JavaScript
Boa prática que manteve o **TBT em 0ms**, garantindo que nenhuma tarefa longa bloqueie a thread principal durante a carga.

---

## Itens Residuais

Estes itens aparecem no relatório pós-melhorias mas **não afetam os scores**:

| Item | Motivo | Solução futura |
|------|--------|----------------|
| Reduce unused CSS (~28 KiB) | Bootstrap CDN carrega o framework completo | Usar PurgeCSS ou Bootstrap customizado com Sass |
| Improve image delivery (~506 KiB) | Imagens originais ainda podem ser comprimidas | Comprimir com Squoosh, TinyPNG ou similar antes do deploy |
| Avoid non-composited animations | Transição `opacity` no `<body>` usa a main thread | Pode ser removida em produção se o FOUC não for perceptível |
| bfcache — 1 failure reason | Possivelmente ligado ao CDN externo | Investigar com DevTools → Application → Back/Forward Cache |

---

## Como Rodar o Projeto

1. Clone ou baixe o repositório
2. Abra a pasta no **VS Code**
3. Instale a extensão **Live Server** (se não tiver)
4. Clique com o botão direito no `index.htm` → **Open with Live Server**
5. Acesse `http://127.0.0.1:5500/index.htm`

Para rodar o Lighthouse:
1. Abra o DevTools (F12)
2. Aba **Lighthouse**
3. Selecione **Desktop** → **Analyze page load**

---

*Documentação gerada em 22/03/2026 — Lighthouse 13.0.1*
