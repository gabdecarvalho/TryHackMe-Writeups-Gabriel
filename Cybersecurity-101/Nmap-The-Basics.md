# Write-up: TryHackMe - Nmap: The Basics

**Plataforma:** TryHackMe

**Sala:** Nmap: The Basics (da trilha Cybersecurity 101)

---

## 1. Introdução e Objetivos da Sala

Esta sala introdutória do TryHackMe, "Nmap: The Basics", tem como objetivo principal familiarizar o usuário com a ferramenta fundamental de escaneamento de rede Nmap (Network Mapper). A sala explora desde os conceitos mais básicos de varredura de portas até a utilização de opções para detecção de serviços e versões, habilidades essenciais para qualquer profissional de cibersegurança.

Os principais objetivos abordados incluíram:
* Descobrir hosts ativos em uma rede.
* Encontrar serviços rodando nos hosts ativos.
* Distinguir diferentes tipos de scans de portas.
* Detectar as versões dos serviços em execução.
* Controlar o timing dos scans.
* Formatar a saída dos resultados.

---

## 2. Ferramentas Utilizadas

* **Nmap (Network Mapper):** A principal ferramenta utilizada para todas as tarefas de escaneamento.
* **Terminal Linux (AttackBox do TryHackMe):** Para execução dos comandos Nmap.
* **Navegador Web:** Para acessar serviços web identificados.

---

## 3. Conceitos Chave e Tipos de Scan Abordados

Durante a sala, diversos conceitos e tipos de scan do Nmap foram introduzidos e praticados:

* **Descoberta de Hosts (`-sn`):** Identifica quais hosts estão online em uma rede sem realizar um scan de portas completo. Utiliza ARP para redes locais e uma combinação de ICMP echo, ICMP timestamp, TCP SYN para porta 443 e TCP ACK para porta 80 em redes remotas.
* **List Scan (`-sL`):** Lista os alvos que seriam escaneados sem realmente enviar pacotes a eles.
* **Escaneamento TCP Connect (`-sT`):** Realiza uma conexão TCP completa (three-way handshake) com cada porta alvo. É confiável, mas facilmente detectável.
* **Escaneamento SYN Stealth (`-sS`):** Envia um pacote SYN e espera por um SYN-ACK. Se recebido, a porta está aberta, e o Nmap envia um RST, não completando o handshake. Mais discreto e geralmente requer privilégios de root.
* **Escaneamento UDP (`-sU`):** Escaneia portas UDP. Mais lento e menos confiável que scans TCP.
* **Seleção de Portas:**
    * `-F` (Fast scan): Escaneia as 100 portas mais comuns.
    * `-p<range>`: Especifica um range de portas (ex: `-p1-1024`).
    * `-p-`: Escaneia todas as 65535 portas.
* **Detecção de OS e Versão:**
    * `-O`: Habilita a detecção do sistema operacional.
    * `-sV`: Habilita a detecção da versão dos serviços.
    * `-A`: Opção agressiva, habilita detecção de OS, versão, script scanning e traceroute.
* **Controle de Scan:**
    * `-Pn`: Trata todos os hosts como online, pulando a fase de descoberta de hosts (útil se os hosts bloqueiam pings).
* **Timing (`-T<0-5>`):** Controla a velocidade do scan (0-paranoid, 1-sneaky, 2-polite, 3-normal, 4-aggressive, 5-insane).
* **Saída de Resultados:**
    * `-v`: Aumenta a verbosidade da saída.
    * `-d`: Habilita a saída de debugging.
    * `-oN <filename>`: Salva a saída em formato normal.
    * `-oX <filename>`: Salva a saída em formato XML.
    * `-oG <filename>`: Salva a saída em formato "grepable".
    * `-oA <basename>`: Salva a saída nos três formatos principais.

---

## 4. Metodologia e Resolução dos Desafios (Walkthrough)

**Task 2: Host Discovery: Who Is Online**

* **Pergunta:** What is the last IP address that will be scanned when your scan target is 192.168.0.1/27?
    * **Resposta:** `192.168.0.31`
    * **Explicação:** Uma máscara /27 significa que os primeiros 27 bits são para a rede e os 5 bits restantes são para hosts. 2^5 = 32 endereços. Como o primeiro endereço (192.168.0.0) é o endereço de rede e o último (192.168.0.31) é o endereço de broadcast, os hosts válidos iriam de .1 a .30. No entanto, o Nmap ao escanear um range /27 a partir de .1, consideraria até o último IP do bloco, que é .31. Se a pergunta se referisse ao último *host endereçável*, seria .30. Dado o contexto de "último IP que será escaneado", .31 é a resposta que esperada para esse tipo de cálculo de range.

**Task 3: Port Scanning: Who Is Listening**

Para esta tarefa, foi necessário escanear a máquina alvo `[MACHINE_IP]`.

1.  **Contando Portas TCP Abertas:**
    Realizei um scan SYN para identificar as portas TCP abertas:
    ```bash
    sudo nmap -sS [MACHINE_IP]
    ```
    * **Resultado Observado:** [Descreva a saída do Nmap, listando as portas abertas encontradas].
    * **Pergunta:** How many TCP ports are open on the target system at MACHINE_IP?
        * **Resposta:** `6`

2.  **Encontrando o Servidor Web e a Flag:**
    Identifiquei a porta do servidor web através do scan anterior (geralmente porta 80 ou outra porta HTTP). Acessei o endereço `http://[MACHINE_IP]:[PORTA_DO_WEBSERVER]` no navegador.
    * **Pergunta:** Find the listening web server on MACHINE_IP and access it with your browser. What is the flag that appears on its main page?
        * **Resposta:** `THM{SECRET_PAGE_38B9P6}`

**Task 4: Version Detection: Extract More Information**

1.  **Identificando Nome e Versão do Servidor Web:**
    Para detectar a versão do serviço web, utilizei o Nmap com a opção `-sV` na porta correspondente.
    ```bash
    sudo nmap -sS -sV -p[PORTA_DO_WEBSERVER] [MACHINE_IP]
    ```
    * **Resultado Observado:** [Descreva a saída do Nmap que mostra a versão do servidor web].
    * **Pergunta:** What is the name and detected version of the web server running on MACHINE_IP?
        * **Resposta:** `lighttpd 1.4.74`

**Task 5: Timing: How Fast is Fast**

* **Pergunta:** What is the non-numeric equivalent of -T4?
    * **Resposta:** `-T aggressive`

**Task 6: Output: Controlling What You See**

* **Pergunta:** What option must you add to your nmap command to enable debugging?
    * **Resposta:** `-d`

**Task 7: Conclusion and Summary**

* **Pergunta:** What kind of scan will Nmap use if you run nmap MACHINE_IP with local user privileges?
    * **Resposta:** `Connect Scan` (ou `-sT`)
    * **Explicação Adicional:** Sem privilégios de root, o Nmap não pode criar pacotes raw necessários para scans como o SYN Stealth (`-sS`), então ele recorre ao TCP Connect Scan (`-sT`), que utiliza chamadas de sistema de rede de nível superior e não requer privilégios especiais.

---

## 5. Desafios Enfrentados

* Lembrei-me da importância de usar `sudo` para scans `-sS` após o primeiro scan não funcionar como esperado e recorrer ao `-sT`.

---

## 6. Lições Aprendidas e Conclusão

A sala "Nmap: The Basics" foi uma excelente introdução prática ao Nmap, demonstrando sua versatilidade e poder para reconhecimento de rede. As principais lições foram:

* A capacidade de descobrir hosts ativos em uma rede local e remota usando `-sn`.
* A diferença fundamental e os casos de uso para scans TCP Connect (`-sT`) e SYN Stealth (`-sS`).
* A importância crucial da detecção de versão de serviço (`-sV`) para identificar softwares específicos e suas potenciais vulnerabilidades.
* Como controlar a velocidade e a verbosidade dos scans para diferentes cenários.
* A necessidade de privilégios elevados para utilizar todo o potencial do Nmap.

Este conhecimento é fundamental para qualquer profissional de cibersegurança. Para um Analista SOC, entender como o Nmap funciona ajuda a interpretar logs de scans na rede, identificar atividades de reconhecimento e validar a postura de segurança dos ativos monitorados. Para quem tem interesse em segurança ofensiva, o Nmap é a ferramenta de partida para mapear alvos. Esta sala solidificou a importância do Nmap como uma ferramenta essencial no meu aprendizado.

---
