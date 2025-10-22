# Write-up: TryHackMe — Vulnerable and Outdated Components

**Plataforma:** TryHackMe  
**Sala:** Vulnerable and Outdated Components — Cybersecurity 101  

---

## 1. Introdução e objetivos da sala

A sala **Vulnerable & Outdated Components** mostra o risco de usar software, bibliotecas ou serviços com vulnerabilidades conhecidas (CVEs). Aprendemos a identificar versões, pesquisar CVEs/exploits e, quando possível, aplicar um exploit disponível em Exploit-DB ou adaptar scripts públicos.

Objetivos:
- Identificar versões de software vulnerável (ex.: Nostromo, WordPress antigo).  
- Pesquisar CVE e exploits (Exploit-DB).  
- Entender remediações (atualização, mitigação de risco, compensating controls).

---

## 2. Ferramentas utilizadas

* Nmap / browser — identificar versão do serviço.  
* Search em Exploit-DB, CVE Details e GitHub.  
* Python/Perl scripts de exploit (ajustar conforme necessário).  
* AttackBox / terminal para rodar scripts e capturar output.

---

## 3. Conceitos chave

- **CVEs** (Common Vulnerabilities and Exposures) indexam vulnerabilidades públicas com identificadores padrão.  
- **Exploit-DB** contém scripts e POCs públicos que podem ser adaptados.  
- **Versão disclosure**: identificar software e versão a partir de banners, headers HTTP, respostas específicas ou fingerprints.  
- **Risco real**: mesmo que exista POC, validar impacto e exigir autorização para qualquer execução (somente lab).

---

## 4. Walkthrough / Metodologia (exemplo prático)

> Exemplo: detectar `nostromo 1.9.6` e usar exploit da Exploit-DB para RCE.

### 1) Identificar versão
- Usar Nmap com `-sV` ou olhar headers `Server:` em respostas HTTP. Ex.: `Server: nostromo 1.9.6`.

### 2) Pesquisar vulnerabilidades
- Pesquisar por `nostromo 1.9.6 exploit` em Exploit-DB ou Google. Encontrar CVE (ex.: CVE-2019-16278) e scripts associados.

### 3) Ajustar e executar exploit POC
- Baixar script da Exploit-DB, revisar o código e corrigir pequenas inconsistências (comentários, dependências, caminhos).  
- Executar com os argumentos corretos, por exemplo:
```bash
python2 47837.py 127.0.0.1 80 id
```
- Interpretar o output (se o exploit prover `uid=1001(_nostromo)` etc.).

### 4) Pós-exploração e limpeza
- Documentar o exploit usado (link/ID do Exploit-DB), o comando exato e o output.  
- Discutir riscos e como remediar (patch/upgrade).

---

## 5. Remediações e recomendações

1. **Mantenha dependências atualizadas** (npm audit / Maven versions / `pip` updates).  
2. **Use scanning automatizado**: Dependabot, Snyk, OSS Index, Retire.js.  
3. **Monitore CVEs das bibliotecas críticas** e crie processo de resposta (triagem, testes, deploy de patch).  
4. **Mitigação temporária**: restringir acesso via WAF/Firewall, aplicar regras de IDS; compensating controls até patch.  
5. **Hardening**: reduzir a superfície (desativar módulos não necessários, limitar privilégios dos serviços).

---

## 6. Lições aprendidas e conclusão

Esta sala reforça que vulnerabilidades públicas são um grande vetor de comprometimento — muitas vezes existe POC pronto. Para desenvolvedores, integrar verificação de dependências e atualizar regularmente reduz grandemente o risco. Documente CVEs relevantes e mantenha processo de aprovação e deploy de correções.

---
