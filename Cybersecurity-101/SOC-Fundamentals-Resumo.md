# Write-up: TryHackMe - SOC Fundamentals - Resumo

**Plataforma:** TryHackMe

**Sala:** SOC Fundamentals (Parte da trilha Cybersecurity 101)

---

## 1. Introdução e Objetivos da Sala

A sala "SOC Fundamentals" do TryHackMe oferece uma visão geral essencial sobre o que é um Centro de Operações de Segurança (SOC), sua importância no cenário atual de ciberameaças e os componentes chave que o definem. O objetivo é construir uma base conceitual sobre as responsabilidades, a estrutura e o funcionamento de um SOC, preparando o aluno para entender o papel crucial que essa equipe desempenha na defesa de uma organização.

Os principais objetivos abordados incluíram:
* Entender o propósito de um SOC e seus principais componentes de Detecção e Resposta.
* Conhecer os três pilares de um SOC: Pessoas, Processos e Tecnologia.
* Identificar os diferentes papéis e responsabilidades dentro de uma equipe SOC.
* Compreender processos chave como a triagem de alertas (5 Ws).
* Ter uma visão geral das tecnologias comumente utilizadas em um SOC.
* Realizar um exercício prático simulando a análise de um alerta por um Analista Nível 1.

---

## 2. Principais Conceitos Abordados

Durante esta sala, os seguintes conceitos fundamentais foram apresentados:

* **Propósito do SOC:** Manter a Detecção e Resposta a incidentes de segurança, monitorando continuamente a rede e os sistemas de uma organização.
* **Capacidades de Detecção:**
  * Descoberta de vulnerabilidades.
  * Detecção de atividade não autorizada.
  * Detecção de violações de políticas de segurança.
  * Detecção de intrusões.
* **Capacidades de Resposta:** Suporte à equipe de resposta a incidentes.
* **Os Três Pilares do SOC:**
  * **Pessoas:** A equipe de profissionais de segurança.
  * **Processos:** Os procedimentos e fluxos de trabalho definidos.
  * **Tecnologia:** As ferramentas e soluções de segurança utilizadas.
* **Papéis na Equipe SOC:**
  * Analista SOC Nível 1 (Triage Specialist, primeira resposta).
  * Analista SOC Nível 2 (Investigação aprofundada).
  * Analista SOC Nível 3 (Caça proativa a ameaças, resposta a incidentes complexos).
  * Engenheiro de Segurança (Implantação e manutenção de ferramentas).
  * Engenheiro de Detecção (Criação de regras de alerta).
  * Gerente do SOC (Gestão da equipe e processos).
* **Processos Chave:**
  * **Triagem de Alertas (Alert Triage):** Análise inicial de um alerta para determinar severidade e prioridade, respondendo aos 5 Ws (What, When, Where, Who, Why).
  * **Relatórios (Reporting):** Escalonamento de alertas confirmados para níveis superiores.
  * **Resposta a Incidentes e Forense:** Processos mais aprofundados para incidentes críticos.
* **Tecnologias Comuns:**
  * SIEM (Security Information and Event Management).
  * EDR (Endpoint Detection and Response).
  * Firewall.
  * Outras (Antivírus, EPP, IDS/IPS, XDR, SOAR).

---

## 3. Explicação dos Conceitos com Minhas Palavras

**3.1. Os Três Pilares do SOC: Pessoas, Processos e Tecnologia**

Um SOC eficaz não se baseia apenas em ferramentas avançadas. Ele é sustentado por três pilares interdependentes:
* **Pessoas:** São os analistas e engenheiros com o conhecimento e a experiência para operar as ferramentas, interpretar os dados, tomar decisões e responder às ameaças. Sem pessoas qualificadas, as melhores tecnologias são inúteis, pois os alertas podem ser mal interpretados ou ignorados.
* **Processos:** São os procedimentos operacionais padrão (SOPs), fluxos de trabalho e playbooks que guiam as ações da equipe. Processos bem definidos garantem consistência, eficiência e que todos saibam o que fazer em diferentes cenários, desde a triagem de um alerta simples até a resposta a um incidente complexo.
* **Tecnologia:** Inclui as ferramentas de segurança como SIEM, EDR, firewalls, etc. Elas automatizam a coleta de dados, a detecção de anomalias e auxiliam na resposta. A tecnologia potencializa a capacidade das pessoas, mas precisa ser bem configurada e gerenciada (pelos engenheiros) e seus outputs interpretados (pelos analistas).

A maturidade de um SOC é alcançada quando esses três pilares estão em harmonia, com profissionais capacitados utilizando tecnologias adequadas através de processos eficientes.

**3.2. Triagem de Alertas (Alert Triage) e os 5 Ws**

A triagem de alertas é a primeira linha de análise em um SOC, geralmente realizada por Analistas Nível 1. Quando uma ferramenta de segurança gera um alerta (ex: "Malware detectado no Host X"), o analista precisa rapidamente determinar se é um falso positivo, qual a sua gravidade e se precisa de escalonamento. Para isso, busca-se responder aos "5 Ws":
* **What (O quê?):** Qual atividade ou evento disparou o alerta? (Ex: Detecção de um arquivo malicioso, múltiplas tentativas de login falhas).
* **When (Quando?):** Em que data e hora o evento ocorreu? (Timestamp é crucial para correlação).
* **Where (Onde?):** Em qual sistema, rede, diretório ou conta o evento ocorreu? (Ex: IP do host, nome do arquivo, conta de usuário).
* **Who (Quem?):** Qual usuário, processo ou sistema de origem está associado ao evento? (Ex: Conta de usuário que baixou o arquivo, IP de origem de um ataque).
* **Why (Por quê?):** Qual a possível causa ou intenção por trás do evento? (Ex: Usuário clicou em link de phishing, sistema desatualizado foi explorado, atividade de pentest autorizada).

Responder a essas perguntas ajuda a contextualizar o alerta, priorizar a resposta e decidir os próximos passos.

---

## 4. Resolução dos Desafios (Walkthrough das Tasks)

**Task 2: Purpose and Components**

* **Pergunta:** The SOC team discovers an unauthorized user is trying to log in to an account. Which capability of SOC is this?
  * **Resposta:** `Detection`
* **Pergunta:** What are the three pillars of a SOC?
  * **Resposta:** `People, Process, Technology`

**Task 3: People**

* **Pergunta:** Alert triage and reporting is the responsibility of?
  * **Resposta:** `SOC Analyst (Level 1)`
* **Pergunta:** Which role in the SOC team allows you to work dedicatedly on establishing rules for alerting security solutions?
  * **Resposta:** `Detection Engineer`

**Task 4: Process**

* **Pergunta:** At the end of the investigation, the SOC team found that John had attempted to steal the system's data. Which 'W' from the 5 Ws does this answer?
  * **Resposta:** `who`
* **Pergunta:** The SOC team detected a large amount of data exfiltration. Which 'W' from the 5 Ws does this answer?
  * **Resposta:** `what`

**Task 5: Technology**

* **Pergunta:** Which security solution monitors the incoming and outgoing traffic of the network?
  * **Resposta:** `firewall`
* **Pergunta:** Do SIEM solutions primarily focus on detecting and alerting about security incidents? (yea/nay)
  * **Resposta:** `yea`

---

## 5. Exercício Prático da Sala (Task 6)

A Task 6 apresentou um cenário prático onde, como Analista Nível 1, recebi um alerta de atividade de varredura de portas. Utilizando uma interface simulada de SIEM para visualizar os logs associados, respondi aos 5 Ws:

* **What:** Activity that triggered the alert?
  * **Resposta:** `Port Scan`
* **When:** Time of the activity?
  * **Resposta:** `June 12, 2024 17:24`
* **Where:** Destination host IP?
  * **Resposta:** `10.0.0.3`
* **Who:** Source host name?
  * **Resposta:** `Nessus` 
* **Why:** Reason for the activity? Intended/Malicious
  * **Resposta:** `Intended`
  * **Observação:** A nota da sala informava que a equipe de avaliação de vulnerabilidades estava realizando um scan a partir do IP `10.0.0.8`. O nome do host de origem "Nessus" possui esse IP.
* **What is the flag found after closing the alert?**
  * **Resposta:** `THM{000_INTRO_TO_SOC}`

---

## 6. Lições Aprendidas e Relevância para SOC

A sala "SOC Fundamentals" foi essencial para estabelecer uma compreensão clara do que é um Centro de Operações de Segurança e como ele funciona. As principais lições foram:
* A importância vital de um SOC na estratégia de defesa de uma organização, focando na detecção e resposta a ameaças.
* A interdependência crucial entre Pessoas, Processos e Tecnologia para a eficácia de um SOC. Um não funciona bem sem os outros.
* A clareza sobre os diferentes papéis dentro de uma equipe SOC e como eles colaboram.
* A aplicação prática da metodologia dos 5 Ws na triagem de alertas, que ajuda a contextualizar e priorizar eventos de segurança.
* Uma introdução às principais categorias de tecnologias utilizadas, com destaque para o papel central do SIEM.

Este conhecimento teórico é a base para qualquer profissional que aspira a trabalhar em um SOC. Entender a estrutura, os papéis, os processos e as ferramentas é o primeiro passo para se tornar um analista capaz de contribuir efetivamente para a segurança de uma organização. Esta sala forneceu uma excelente fundação para os estudos mais práticos da trilha SOC Level 1.

---
