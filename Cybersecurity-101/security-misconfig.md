# Write-up: TryHackMe — Security Misconfiguration

**Plataforma:** TryHackMe  
**Sala:** Security Misconfiguration — Cybersecurity 101  

---

## 1. Introdução e objetivos da sala

A sala **Security Misconfiguration** foca em erros de configuração que deixam aplicações e servidores expostos: debug consoles em produção, permissões laxas, endpoints de administração publicados, headers inseguros, diretórios públicos com arquivos sensíveis, entre outros.

Objetivos:
- Identificar e explorar configurações perigosas (ex.: console Werkzeug, diretórios públicos).
- Demonstrar como um simples ajuste/remover debug pode prevenir exploração.
- Aprender práticas de configuração seguras para ambientes de produção.

---

## 2. Ferramentas utilizadas

* Navegador + DevTools — acessar `/console`, ver stack traces.  
* Burp Suite — automatizar requisições e modificar parâmetros.  
* Terminal para comandos HTTP (`curl`) e para examinar arquivos caso consiga RCE/local file read.  

---

## 3. Exemplos comuns de misconfigurações

- **Debug consoles habilitados** (ex.: Werkzeug debug console) — permitem executar código arbitrário via interface web.  
- **Diretórios públicos com arquivos sensíveis** (`/backup`, `/.git`, `/config`) — permitem download de credenciais.  
- **Servidores com versões padrões / contas default** — credenciais padrão não trocadas.  
- **CORS permissivo / headers inseguros** — expõe recursos a terceiros.  
- **Exposição de painéis administrativos** sem autenticação adequada.

---

## 4. Walkthrough / Metodologia (exemplo prático)

> Exemplo típico: aplicação com Werkzeug console acessível em `/console`.

### 1) Encontrar consoles / endpoints
- Acesse possíveis paths comuns: `/console`, `/admin`, `/debug`, `/.git/`, `/phpinfo.php`.  
- Revisar erros/stack traces que expõem caminhos do filesystem ou variáveis de ambiente.

### 2) Usar console para executar comandos
- Na interface Werkzeug, executar `import os; print(os.listdir('.'))` para listar arquivos.  
- Ler `app.py` com `print(open('app.py').read())` para descobrir `secret_key`/`secret_flag`.

### 3) Exploração prática comum
- Ler arquivos de configuração (`config.py`, `.env`, `todo.db`) e extrair flags/credenciais.  
- Se o console permitir, executar `os.popen('ls -l').read()` e outras operações I/O.  
- Baixar o arquivo de banco de dados e examiná-lo localmente (`sqlite3 todo.db`) para encontrar senhas/flags.

### 4) Documentar achados
- Registrar caminho do endpoint explorado, comandos executados e conteúdo sensível extraído (nome de arquivo e flag).

---

## 5. Mitigações recomendadas

1. **Desativar debug em produção**: variáveis de ambiente e configs devem garantir debug=False.  
2. **Remover endpoints de administração não necessários** e proteger com autenticação forte.  
3. **Restringir acesso por IP** para painéis administrativos (VPN / firewall).  
4. **Não versionar arquivos sensíveis** em repositórios públicos (`.env`, `credentials`).  
5. **Headers de segurança HTTP**: `X-Frame-Options`, `X-Content-Type-Options`, `Content-Security-Policy`.  
6. **Gerenciar permissões de arquivos** e separar ambientes (dev/staging/prod).

---

## 6. Lições aprendidas e conclusão

Security misconfiguration mostra que vulnerabilidades frequentemente surgem por descuido operacional, não por bugs no código. Para times de desenvolvimento, práticas de deploy, revisão de configurações e automação (checks em CI/CD) reduzem muito esse risco. Este write-up documenta como descobrir, explorar e corrigir esse tipo de falha.
