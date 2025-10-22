# Write-up: TryHackMe — IDOR (Insecure Direct Object References)

**Plataforma:** TryHackMe  
**Sala:** IDOR / Broken Access Control (parte do OWASP Top 10 practicals)  

---

## 1. Introdução e objetivos da sala

A sala **IDOR (Insecure Direct Object Reference)** demonstra um tipo de *Broken Access Control* onde a aplicação expõe identificadores diretos de objetos (IDs) sem verificar corretamente se o usuário autenticado tem autorização para acessá‑los. O objetivo é entender como identificar referências diretas inseguras, explorá‑las para acessar recursos de outros usuários e aprender as mitigations apropriadas.

Objetivos específicos:
- Entender o que é um IDOR e por que ocorre.
- Aprender a manipular parâmetros que referenciam objetos (IDs, filenames, paths).
- Praticar exploração em uma aplicação vulnerável para extrair dados sensíveis.
- Conhecer contramedidas para evitar esse tipo de falha.

---

## 2. Ferramentas utilizadas

* Navegador (DevTools — Network) — inspecionar URLs e parâmetros.  
* Burp Suite / proxy HTTP (opcional) — interceptar e modificar requisições com facilidade.  
* Curl (terminal) — fazer requisições manipuladas.  
* AttackBox / máquina do TryHackMe — ambiente da VM para testar a exploração.

---

## 3. Conceito: o que é IDOR?

**IDOR (Insecure Direct Object Reference)** acontece quando a aplicação usa um identificador direto para referenciar um objeto (por exemplo `?id=12345`, `user/42`, `file=backup.db`) e não valida se o usuário logado tem permissão para acessar aquele objeto. Em vez de usar valores indiretos (tokens de acesso, checkserver-side de ownership), ela confia apenas no identificador enviado pelo cliente.

Exemplo simples:
```
GET /account?id=111111
```
Se o servidor retorna os dados da conta sem checar se o `id` pertence ao usuário autenticado, um atacante pode mudar `111111` para `222222` e ver outra conta.

---

## 4. Metodologia e resolução (walkthrough)

### Cenário da sala
A VM disponibilizada tinha uma aplicação com autenticação. Ao logar como o usuário `noot` (credenciais fornecidas na task), era possível ver notas/objetos atrelados à conta. A aplicação expunha um parâmetro `id` na URL para acessar notas de usuários.

**Passos que segui:**

1. **Login**
   - Acessei `http://MACHINE_IP` (endereço da VM).
   - Efetuei login com o usuário `noot` / senha `test1234` conforme instruído.

2. **Enumerar recursos**
   - Naveguei até a página que lista notas/recursos — observei URLs como:
     ```
     http://MACHINE_IP/note?id=1
     http://MACHINE_IP/note?id=2
     ```
   - Notei que `id` é um identificador direto (Direct Object Reference).

3. **Testar manipulação de ID**
   - Modifiquei manualmente o parâmetro `id` na barra de endereço (ou via Burp) para `?id=3`, `?id=4` etc.
   - Ao aumentar o `id`, consegui visualizar notas que claramente pertenciam a outros usuários (conteúdo sensível).

4. **Encontrar a flag**
   - Entre as notas acessadas por manipulação do `id`, encontrei a flag:
     ```
     flag{fivefourthree}
     ```

### Observações técnicas
- A aplicação não verificou no servidor se o `id` solicitado pertencia ao usuário logado.
- Provavelmente a verificação de ownership foi omitida na rota que serve as notas, permitindo leitura arbitrária por qualquer ID válido.

---

## 5. Comandos / técnicas úteis

### Reproduzir com curl (exemplo)
```bash
# visualizar nota com id 1 (após login/session cookie)
curl -i 'http://MACHINE_IP/note?id=1' --cookie "session=SESSVALUE"
```

### Usar Burp para testar rapidamente
1. Interceptar requisição ao acessar `/note?id=1`.  
2. Modificar `id` para vários valores e reenviar.  
3. Observar respostas e identificar conteúdo sensível.

---

## 6. Mitigações e boas práticas

Para prevenir IDORs, adote as seguintes práticas:

1. **Authorization Server-side**: nunca confiar que um identificador válido implica autorização. Sempre verificar no servidor se o usuário logado tem permissão/ownership do recurso.
2. **Use referências indiretas**: em vez de IDs sequenciais, usar tokens aleatórios/unguessable (por ex. UUIDs ou mapeamento interno) quando apropriado.
3. **Least Privilege**: devolver apenas campos necessários ao usuário, e sempre ocultar dados sensíveis quando não autorizado.
4. **Logging e monitoring**: registrar acessos anômalos (ex.: muitos acessos a diferentes IDs em curtos intervalos).
5. **Input validation e rate limiting**: dificultar a enumeração automática de IDs.
6. **Testes automatizados**: incluir verificações de autorização em testes de integração.

---

## 7. Lições aprendidas e conclusão

O IDOR é uma falha que resulta frequentemente de um pequeno descuido (esquecer de checar ownership) mas que tem impacto alto: exposição de dados de outros usuários. A exploração é frequentemente trivial (alterar um parâmetro) e, por isso, é um risco crítico em APIs e aplicações web.

Ao terminar esta sala você deve:
- Ser capaz de identificar padrões de URL/param que indicam risco (IDs expostos);
- Testar rapidamente alterando parâmetros via navegador ou proxy;
- Saber as mitigations essenciais para eliminar o risco.

---
