# Write-up: TryHackMe — Authentication Bypass

**Plataforma:** TryHackMe  
**Sala:** Authentication Bypass (da trilha Web Fundamentals / Introduction to Web Hacking)  

---

## 1. Introdução e objetivos da sala

A sala **Authentication Bypass** explora falhas e lógica incorreta em mecanismos de autenticação e sessão — áreas críticas onde um erro de design ou implementação pode permitir que um atacante acesse contas alheias, contorne logins ou ganhe privilégios indevidos. O objetivo é entender os vetores comuns de bypass (re-registro, tampering de cookies/JWT, lógica de reset de senha, controle fraco de sessão) e praticar técnicas de exploração em cenários guiados.

Objetivos específicos:
- Reconhecer falhas típicas em mecanismos de autenticação.
- Explorar falhas de lógica (ex.: re-registration, missing checks).
- Entender como manipular cookies / tokens quando apropriado.
- Aprender técnicas para testar robustez de processos como reset de senha.

---

## 2. Ferramentas utilizadas

* Navegador com DevTools (Network / Application) — inspecionar cookies, requests e respostas.  
* Burp Suite / proxy HTTP (opcional) — interceptar e modificar requisições.  
* Curl (terminal) — reproduzir requisições quando necessário.  
* AttackBox / máquina do TryHackMe fornecida pela sala — interagir com aplicações vulneráveis.

---

## 3. Conceitos chave

### Autenticação vs Autorização
- **Autenticação:** processo de verificar identidade (ex: login com usuário/senha).  
- **Autorização:** verificar se um usuário autenticado tem permissão para executar uma ação ou acessar um recurso.

### Vulnerabilidades comuns de Authentication Bypass
- **Re-registration / username normalization flaws:** permitir criar um usuário diferente que é tratado como o mesmo (ex.: `" darren"` vs `"darren"`), ganhando acesso indevido.  
- **Weak session handling:** cookies ou tokens previsíveis ou sem integridade permitem impersonação.  
- **JWT None / alg manipulation:** bibliotecas mal implementadas aceitam `alg: none` permitindo bypass de assinatura.  
- **Password reset logic flawed:** validação do reset baseada em dados inseguros (ex.: retornos previsíveis, falta de rate limiting).  
- **Default creds / debug endpoints expostos:** contas padrão ou consoles de debug deixam portas abertas.

### Boas práticas para proteção
- Normalizar e validar usernames (trim, case normalization) e evitar aceitar entradas com espaços/characters invisíveis.  
- Usar tokens com integridade e segredo (HMAC/RS256) e validar algoritmos corretamente.  
- Implementar rate-limiting e MFA para processos sensíveis (reset de senha).  
- Não confiar em dados do cliente para autorização; sempre checar ownership no servidor.

---

## 4. Metodologia e resolução (walkthrough)

> Abaixo descrevo como executei o laboratório e as respostas/flags obtidas. Adaptei o fluxo de resolução às tarefas típicas encontradas nessa sala.

### Cenário prático fornecido
A aplicação vulnerável tinha um bug de re-registro: tentar recriar um usuário existente com um espaço no início do nome (`" darren"`) permitia criar uma nova conta que o aplicativo tratava como se fosse o usuário original, concedendo acesso ao conteúdo restrito desse usuário. Esse comportamento exemplifica uma falha de validação/normalização do campo `username`.

---

### Passo 1 — Reconhecimento e teste básico
1. Acessei a página de login / register da VM (`http://MACHINE_IP:8088` — exemplo).  
2. Tentei registrar o usuário `darren` e notei que já existia.  
3. Tentei registrar `" darren"` (com um espaço inicial). O formulário permitiu o registro com sucesso.

**Observação técnica:** o servidor possivelmente **truncava** espaços ao comparar internamente, ou a verificação de existência fazia comparação sem normalização, permitindo a criação de uma entrada adicional com equivalência lógica.

---

### Passo 2 — Exploração: acessar a conta do usuário alvo
1. Após registrar `" darren"`, efetuei login com as credenciais da nova conta.  
2. A interface mostrou os dados que pertenciam ao usuário `darren` original, incluindo o flag da conta.

**Flag encontrado:** `fe86079416a21a3c99937fea8874b667`

3. Repeti o mesmo processo para o usuário `arthur`:
   - Registrei `" arthur"` e efetuei login.
   - Localizei o flag associado: `d9ac0f7db4fda460ac3edeb75d75e16e`

**Resumo do exploit:** criar uma conta com input mal-normalizado (espaço) permitiu o bypass da verificação de identidade e o acesso a recursos de outros usuários.  

---

### Task adicional — Verificação de implementação
- Inspecionei as requisições usando Developer Tools / Burp. Observei que o servidor aceitava usernames com espaços e não aplicava trimming antes da verificação de existência.  
- Tentei variações (tabs, múltiplos espaços, unicode invisível) para testar robustez — muitas vezes uma simples normalização resolveria a falha.

---

## 5. Comandos / técnicas úteis

### Reproduzir requisição com curl (exemplo)
```bash
# Exemplo hipotético de POST para registrar um usuário " darren"
curl -i -X POST http://MACHINE_IP:8088/register   -d 'username= darren&password=senha123'
```

### Usar Burp Suite
- Interceptar a requisição de registro do usuário.  
- Modificar o campo `username` para `" darren"` e reenviar.  
- Observar cookies de sessão e o conteúdo retornado.

---

## 6. Lições aprendidas e recomendações

### Por que isso é crítico?
Falhas de autenticação permitem que atacantes assumam contas de outros usuários, geralmente com impacto direto na confidencialidade e integridade dos dados. Mesmo problemas aparentemente “pequenos” como spaces invisíveis ou normalização incorreta podem resultar em compromissos completos de contas.

### Mitigações práticas (para desenvolvedores)
- **Normalize** entradas de username/email: `trim()` espaços, converta para case-normalization (lowercase) quando apropriado.  
- **Validate** e **sanitize** antes de gravar no banco e antes de comparar com registros existentes.  
- **Não confie** em dados do cliente para autorização: sempre verificar no servidor se o usuário autenticado é dono do recurso.  
- **Rate-limit** tentativas de registro e login para reduzir abuso.  
- **Logs e monitoramento** para detecção de comportamento anômalo (múltiplos registros com variações de username).

---

## 7. Conclusão

A sala **Authentication Bypass** demonstra como pequenos erros de lógica ou falta de normalização podem quebrar completamente a segurança de autenticação de uma aplicação. Para quem está migrando de desenvolvimento para segurança (ou vice-versa), essa sala mostra a importância de pensar em entrada e armazenamento de dados com segurança desde o início (secure by design).  

---
