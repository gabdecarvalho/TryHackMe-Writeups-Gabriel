# Write-up: TryHackMe - Introduction to SIEM

**Plataforma:** TryHackMe

**Sala:** Introduction to SIEM (Parte da trilha Cybersecurity 101)

---

## 1. Introdução e Objetivos da Sala

A sala "Introduction to SIEM" do TryHackMe fornece uma visão fundamental sobre os sistemas de Gerenciamento de Informações e Eventos de Segurança (SIEM). O objetivo é explicar o que é um SIEM, por que ele é uma ferramenta crucial para a cibersegurança moderna, como ele funciona coletando e correlacionando dados de diversas fontes, e quais são suas principais capacidades e benefícios para um Centro de Operações de Segurança (SOC).

Os principais objetivos de aprendizado abordados incluíram:
* Definir SIEM e seu funcionamento básico.
* Entender a necessidade de visibilidade da rede e como o SIEM contribui para isso.
* Identificar diferentes fontes de log (host-centric e network-centric) e métodos de ingestão de logs.
* Reconhecer as capacidades chave que um SIEM oferece para detecção e resposta a ameaças.

---

## 2. Principais Conceitos Abordados

Durante esta sala, os seguintes conceitos cruciais sobre SIEM foram apresentados:

* **Definição de SIEM:** Security Information and Event Management system – uma solução que coleta, armazena, analisa e correlaciona dados de log de múltiplas fontes para identificar atividades suspeitas e incidentes de segurança.
* **Visibilidade da Rede:**
  * **Host-Centric Log Sources:** Eventos ocorridos dentro de um host (ex: logs de eventos do Windows, Sysmon, acesso a arquivos, execução de processos).
  * **Network-Centric Log Sources:** Eventos relacionados à comunicação entre hosts ou com a internet (ex: logs de firewall, VPN, tráfego HTTP, conexões SSH).
* **Importância do SIEM:** Centralização de logs, correlação de eventos, alertas em tempo real, monitoramento 24/7, detecção precoce de ameaças, visualização de dados e capacidade de investigar incidentes passados.
* **Fontes de Log Comuns:** Máquinas Windows (Event Viewer), Estações Linux (`/var/log/`), Servidores Web (Apache, Nginx).
* **Métodos de Ingestão de Logs:**
  * Agente/Forwarder (ex: Splunk Universal Forwarder).
  * Syslog.
  * Upload Manual.
  * Port-Forwarding.
* **Capacidades do SIEM:** Coleta de logs, correlação de eventos, alertas, visibilidade, investigação, caça a ameaças (threat hunting).
* **Responsabilidades do Analista SOC com SIEM:** Monitoramento, investigação de alertas, identificação de falsos positivos, ajuste de regras, relatórios, identificação de "pontos cegos".
* **Dashboards SIEM:** Apresentação visual de dados, alertas, saúde do sistema, principais eventos.
* **Regras de Correlação:** Expressões lógicas para identificar padrões suspeitos e disparar alertas (ex: múltiplas falhas de login, logs sendo apagados, execução de comandos suspeitos).
* **Investigação de Alertas:** Determinar se um alerta é um falso positivo ou um verdadeiro positivo e tomar as ações apropriadas.

---

## 3. Explicação dos Conceitos com Minhas Palavras

**3.1. O que é um SIEM e Por Que é Essencial?**

Um SIEM, ou Sistema de Gerenciamento de Informações e Eventos de Segurança, é como um detetive central altamente tecnológico para a rede de uma organização. Imagine todos os dispositivos e aplicações (computadores, servidores, firewalls) gerando constantemente "diários" (logs) sobre o que está acontecendo. Seria humanamente impossível ler todos esses diários individualmente e em tempo real para encontrar algo suspeito.

O SIEM resolve isso: ele coleta todos esses logs de forma automática, armazena-os em um local centralizado e, o mais importante, usa "inteligência" (regras de correlação, análise de comportamento, machine learning) para cruzar informações de diferentes fontes. Se ele detecta um padrão que parece uma atividade maliciosa (como alguém tentando adivinhar senhas em vários sistemas ao mesmo tempo, ou um computador começando a se comunicar com um endereço IP conhecido por distribuir malware), ele gera um alerta para os analistas de segurança investigarem. Essencialmente, o SIEM transforma um mar de dados brutos em insights acionáveis para proteger a organização.

**3.2. Fontes de Log Host-Centric vs. Network-Centric**

Para que o SIEM funcione, ele precisa de dados. Esses dados vêm de "fontes de log", que podemos dividir em duas categorias principais:

* **Host-Centric (Focadas no Host/Dispositivo):** São logs que registram atividades que acontecem *dentro* de um computador ou servidor específico. Exemplos incluem:
  * **Windows Event Logs:** Registram logins, falhas de login, criação de processos, alterações no sistema, etc.
  * **Sysmon (System Monitor):** Uma ferramenta mais avançada para Windows que loga atividades detalhadas de processos, conexões de rede por processo, e outras ações no nível do sistema.
  * **Logs do Linux (`/var/log/auth.log`, `/var/log/syslog`):** Registram autenticações, atividades do sistema, erros, etc.
  * **Logs de Antivírus/EDR:** Registram detecções de malware ou atividades suspeitas no endpoint.
    Esses logs nos dizem o que está acontecendo *naquela máquina específica*.

* **Network-Centric (Focadas na Rede):** São logs que registram a comunicação *entre* dispositivos na rede ou entre a rede interna e a internet. Exemplos incluem:
  * **Logs de Firewall:** Mostram quais conexões foram permitidas ou bloqueadas na borda da rede.
  * **Logs de Servidores Web (Apache, Nginx):** Registram todas as requisições HTTP feitas ao servidor, incluindo IPs de origem, páginas acessadas, e erros.
  * **Logs de VPN:** Registram quem se conectou à rede remotamente, de onde e quando.
  * **Logs de IDS/IPS:** Registram tentativas de intrusão ou tráfego de rede malicioso detectado.
    Esses logs nos dão uma visão do tráfego e das interações na rede.

Um SIEM eficaz precisa de ambos os tipos de log para ter uma visão completa e poder correlacionar eventos que ocorrem em um host específico com atividades na rede.

**3.3. Regras de Correlação e Alertas**

A "mágica" do SIEM acontece em grande parte através das regras de correlação. Uma regra de correlação é basicamente uma instrução lógica que diz ao SIEM: "Se você vir o evento A acontecer, seguido pelo evento B em um certo período de tempo, e talvez o evento C também estiver presente, então isso é suspeito e você deve gerar um alerta".

Por exemplo, uma regra simples poderia ser:
* **Regra:** "Se um usuário tiver 5 tentativas de login falhas (Evento X) em menos de 1 minuto, seguidas por 1 tentativa de login bem-sucedida (Evento Y) do mesmo IP, gere um alerta de 'Possível Ataque de Força Bruta Bem-Sucedido'."

Outro exemplo dado na sala:
* **Regra:** "Se o Log de Eventos do Windows (Fonte) registrar o Event ID 104 (logs foram apagados), gere um alerta de 'Logs de Eventos Apagados'." Isso é altamente suspeito, pois atacantes frequentemente apagam logs para cobrir seus rastros.

Os analistas SOC (especialmente Engenheiros de Detecção ou Analistas Nível 2/3) criam e ajustam essas regras para que o SIEM possa detectar automaticamente uma vasta gama de atividades maliciosas ou anormais, permitindo uma resposta mais rápida.

---

## 4. Resolução dos Desafios (Walkthrough das Tasks)

**Task 1: Introduction**

* **Pergunta:** What does SIEM stand for?
  * **Resposta:** `Security Information and Event Management system`

**Task 2: Network Visibility through SIEM**

* **Pergunta:** Is Registry-related activity host-centric or network-centric?
  * **Resposta:** `host-centric`
* **Pergunta:** Is VPN related activity host-centric or network-centric?
  * **Resposta:** `network-centric`

**Task 3: Log Sources and Log Ingestion**

* **Pergunta:** In which location within a Linux environment are HTTP logs stored?
  * **Resposta:** `/var/log/httpd` (ou `/var/log/apache2`, `/var/log/nginx` dependendo da distribuição e servidor web)

**Task 4: Why SIEM**
* (Esta tarefa era de leitura para entender as capacidades do SIEM)

**Task 5: Analysing Logs and Alerts**

* **Pergunta:** Which Event ID is generated when event logs are removed?
  * **Resposta:** `104`
* **Pergunta:** What type of alert may require tuning?
  * **Resposta:** `False Alarm` (ou Falso Positivo)

**Task 6: Lab Work**

No laboratório prático, um alerta foi acionado e precisei investigar os eventos associados no dashboard simulado do SIEM.

1.  **Ativação da Atividade Suspeita:** Cliquei em "Start Suspicious Activity".
2.  **Identificação do Processo:**
    * **Pergunta:** Click on Start Suspicious Activity, which process caused the alert?
      * **Resposta:** `cudominer.exe`
3.  **Identificação do Usuário Responsável:**
    * **Pergunta:** Find the event that caused the alert, which user was responsible for the process execution?
      * **Resposta:** `chris.fort`
4.  **Identificação do Hostname:**
    * **Pergunta:** What is the hostname of the suspect user?
      * **Resposta:** `HR_02`
5.  **Termo que Correspondeu à Regra:**
    * **Pergunta:** Examine the rule and the suspicious process; which term matched the rule that caused the alert?
    * ![image](https://github.com/user-attachments/assets/fad7ebf2-dbf4-4517-8347-56033c84b3ad)
      * **Resposta:** `miner`
6.  **Classificação do Evento:**
    * **Pergunta:** What is the best option that represents the event? Choose from the following: False-Positive / True-Positive
      * **Resposta:** `True-Positive` (Dado que era um processo de mineração não esperado).
7.  **Flag:**
    * **Pergunta:** Selecting the right ACTION will display the FLAG. What is the FLAG?
      * **Resposta:** `THM{000_SIEM_INTRO}`

---

## 5. Lições Aprendidas e Relevância para SOC

A sala "Introduction to SIEM" foi fundamental para entender o papel central que essa tecnologia desempenha nas operações de segurança modernas. As principais lições foram:
* A capacidade do SIEM de agregar e correlacionar logs de diversas fontes é o que permite uma visibilidade abrangente da rede e a detecção eficaz de ameaças.
* A distinção entre logs host-centric e network-centric e como ambos são vitais para uma análise completa.
* O poder das regras de correlação para automatizar a detecção de padrões suspeitos que seriam difíceis de identificar manualmente.
* A importância dos dashboards para fornecer aos analistas uma visão consolidada e acionável do estado da segurança.
* O processo básico de investigação de um alerta, desde a sua geração até a determinação se é um verdadeiro ou falso positivo.

Para um aspirante a Analista SOC, compreender os fundamentos do SIEM é o primeiro passo para ser capaz de utilizar essa ferramenta no dia a dia para monitorar, detectar e responder a incidentes. Esta sala forneceu uma excelente base teórica e um gostinho prático de como um SIEM opera.

---
