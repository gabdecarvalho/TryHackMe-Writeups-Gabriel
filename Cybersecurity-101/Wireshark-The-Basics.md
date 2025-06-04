# Write-up: TryHackMe - Wireshark: The Basics

**Plataforma:** TryHackMe

**Sala:** Wireshark: The Basics (da trilha Cybersecurity 101)

---

## 1. Introdução e Objetivos da Sala

A sala "Wireshark: The Basics" do TryHackMe é uma introdução essencial à poderosa ferramenta de análise de tráfego de rede Wireshark. O objetivo principal é capacitar o usuário a capturar, visualizar e interpretar pacotes de dados, uma habilidade fundamental para Analistas SOC na investigação de atividades suspeitas, troubleshooting de problemas de rede e compreensão de como os protocolos de comunicação funcionam.

Os principais objetivos abordados incluíram:
* Compreender a interface do Wireshark e seus principais painéis.
* Aprender a aplicar filtros de captura e filtros de visualização (display filters).
* Identificar e analisar protocolos comuns como TCP, UDP, HTTP, DNS, etc.
* Seguir conversas de rede (TCP streams).
* Extrair informações relevantes de pacotes capturados e arquivos PCAP.

---

## 2. Ferramentas Utilizadas

* **Wireshark:** A principal ferramenta para captura e análise de pacotes.
* **Arquivos PCAP Fornecidos:** `http1.pcapng` (para simulação de screenshots) e `Exercise.pcapng` (para responder às perguntas).

---

## 3. Conceitos Chave e Funcionalidades Aprendidas

Durante esta sala, os seguintes conceitos e funcionalidades do Wireshark foram explorados:

* **Interface do Wireshark:**
    * Toolbar, Display Filter Bar, Recent Files, Capture Filter and Interfaces, Status Bar.
    * Painel de Lista de Pacotes (Packet List Pane)
    * Painel de Detalhes do Pacote (Packet Details Pane)
    * Painel de Bytes do Pacote (Packet Bytes Pane)
* **Carregamento e Mesclagem de Arquivos PCAP.**
* **Visualização de Detalhes do Arquivo de Captura** (Propriedades do Arquivo).
* **Dissecação de Pacotes (Packet Dissection):** Análise das camadas OSI dentro de um pacote.
* **Navegação de Pacotes:** Ir para pacotes específicos, encontrar pacotes por conteúdo, marcar pacotes e adicionar comentários.
* **Exportação de Pacotes e Objetos (Arquivos).**
* **Formatos de Exibição de Tempo.**
* **Informações de Especialista (Expert Info):** Para identificar anomalias e problemas.
* **Filtragem de Pacotes:**
    * Aplicar como Filtro (Apply as Filter).
    * Filtro de Conversa (Conversation Filter).
    * Colorir Conversa (Colourise Conversation).
    * Preparar como Filtro (Prepare as Filter).
    * Aplicar como Coluna (Apply as Column).
    * Seguir Streams (Follow TCP/UDP/HTTP Stream).

---

## 4. Metodologia e Resolução dos Desafios (Walkthrough)

Utilizei o arquivo `Exercise.pcapng` para responder às perguntas, conforme instruído.

**Task 1: Introduction**

* **Pergunta:** Which file is used to simulate the screenshots?
    * **Resposta:** `http1.pcapng`
* **Pergunta:** Which file is used to answer the questions?
    * **Resposta:** `Exercise.pcapng`

**Task 2: Tool Overview**

1.  **Lendo os Comentários do Arquivo de Captura:**
    Para encontrar a flag nos comentários, abri o arquivo `Exercise.pcapng` e naveguei até "Statistics" -> "Capture File Properties".
    * **Pergunta:** Read the "capture file comments". What is the flag?
        * **Resposta:** `TryHackMe_Wireshark_Demo`

2.  **Número Total de Pacotes:**
    Ainda na janela "Capture File Properties" (ou na barra de status principal do Wireshark após carregar o arquivo).
    * **Pergunta:** What is the total number of packets?
        * **Resposta:** `58620`

3.  **Hash SHA256 do Arquivo:**
    Encontrado também na janela "Capture File Properties".
    * **Pergunta:** What is the SHA256 hash value of the capture file?
        * **Resposta:** `f446de335565fb0b0ee5e5a3266703c778b2f3dfad7efeaeccb2da5641a6d6eb`

**Task 3: Packet Dissection**

1.  **Analisando Pacote Número 38:**
    Naveguei até o pacote 38 na lista de pacotes.
    * **Pergunta:** View packet number 38. Which markup language is used under the HTTP protocol?
        * **Resposta:** `eXtensible Markup Language`
        * **Observação:** Isso foi identificado no Painel de Detalhes do Pacote, expandindo a camada de "Hypertext Transfer Protocol" e procurando por tipos de conteúdo ou dados que indicassem XML.

    * **Pergunta:** What is the arrival date of the packet? (Answer format: Month/Day/Year)
        * **Resposta:** `05/13/2004`
        * **Observação:** Esta informação é visível na camada "Frame" no Painel de Detalhes do Pacote.

    * **Pergunta:** What is the TTL value?
        * **Resposta:** `47`
        * **Observação:** Encontrado na camada "Internet Protocol Version 4" (IPv4), no campo "Time to live".

    * **Pergunta:** What is the TCP payload size?
        * **Resposta:** `424`
        * **Observação:** Geralmente, o Wireshark mostra o tamanho dos dados da camada de transporte (TCP Segment Length) ou o tamanho dos dados da aplicação. Se a pergunta se refere ao payload da camada TCP que contém os dados da aplicação HTTP, este valor seria encontrado inspecionando os detalhes da camada TCP ou HTTP.

    * **Pergunta:** What is the e-tag value? (For example: 82ecb-6321-9e904585)
        * **Resposta:** `9a01a-4696-7e354b00`
        * **Observação:** Encontrado nos cabeçalhos HTTP da resposta, no Painel de Detalhes do Pacote, sob a camada "Hypertext Transfer Protocol".

**Task 4: Packet Navigation**

1.  **Buscando String "r4w" e Lendo Comentários:**
    * **Busca pela string:** Usei "Edit" -> "Find Packet...". Selecionei "String" como tipo de pesquisa, "Packet details" como campo de busca, e procurei por "r4w".
    * **Pergunta:** Search the "r4w" string in packet details. What is the name of artist 1?
        * **Resposta:** `r4w8173`
        * **Observação:** Após encontrar o pacote com a string, a informação do artista estaria nos dados da aplicação (provavelmente HTTP).

    * **Comentários do Pacote 12:** Naveguei até o pacote 12. Cliquei com o botão direito -> "Packet Comment..." (ou verifiquei se havia um ícone de comentário).
    * **Pergunta:** Go to packet 12 and read the packet comments. What is the answer? (Note: use md5sum <filename> terminal command to get MD5 hash)
        * **Resposta:** `911cd574a42865a956ccde2d04495ebf`
        * **Observação:** O comentário provavelmente continha um nome de arquivo. Para obter o hash MD5, seria necessário exportar esse arquivo (se possível) e usar o comando `md5sum` no terminal da AttackBox.

2.  **Encontrando Arquivo .txt e Nome do Alien:**
    Para encontrar um arquivo ".txt" transferido, utilizei a funcionalidade "File" -> "Export Objects" -> "HTTP...". Procurei por arquivos com a extensão .txt na lista.
    * **Pergunta:** There is a ".txt" file inside the capture file. Find the file and read it; what is the alien's name?
        * **Resposta:** `PACKETMASTER`
        * **Observação:** Após exportar o arquivo .txt, abri-o para ler seu conteúdo.

3.  **Verificando Informações de Especialista (Expert Info):**
    Acessei "Analyse" -> "Expert Information".
    * **Pergunta:** Look at the expert info section. What is the number of warnings?
        * **Resposta:** `1636`

**Task 5: Packet Filtering**

1.  **Aplicando Filtro HTTP:**
    Fui ao pacote número 4, cliquei com o botão direito em "Hypertext Transfer Protocol" no Painel de Detalhes do Pacote e selecionei "Apply as Filter" -> "Selected".
    * **Pergunta:** Go to packet number 4. Right-click on the "Hypertext Transfer Protocol" and apply it as a filter. Now, look at the filter pane. What is the filter query?
        * **Resposta:** `http`

    * **Pergunta:** What is the number of displayed packets?
        * **Resposta:** `1089`

2.  **Seguindo Stream HTTP e Analisando Respostas:**
    Naveguei até o pacote número 33790. Cliquei com o botão direito e selecionei "Follow" -> "HTTP Stream".
    * **Pergunta:** Go to packet number 33790, follow the HTTP stream, and look carefully at the responses. Looking at the web server's
response, what is the total number of artists?
        * **Resposta:** `3`
        * **Observação:** Analisei o conteúdo da resposta do servidor na janela do stream HTTP para contar os artistas.

    * **Pergunta:** What is the name of the second artist?
        * **Resposta:** `Blad3`
        * **Observação:** Identificado na mesma análise do stream HTTP.

---

## 5. Desafios Enfrentados (Opcional)

* Inicialmente, tive dificuldade em encontrar a opção de exportar objetos HTTP, mas explorar os menus 'File' me levou à funcionalidade correta.
* Diferenciar entre filtros de captura e filtros de visualização exigiu um pouco de prática para internalizar.

---

## 6. Lições Aprendidas e Conclusão

A sala "Wireshark: The Basics" foi extremamente valiosa para desmistificar a análise de tráfego de rede. As principais lições aprendidas foram:

* A importância dos filtros de visualização para isolar rapidamente o tráfego de interesse em capturas grandes.
* Como navegar pela interface do Wireshark para inspecionar detalhes de pacotes em diferentes camadas do modelo OSI.
* A capacidade de reconstruir e entender conversas de rede, especialmente streams HTTP e TCP, para análise de dados de aplicação.
* A identificação de protocolos comuns e seus campos chave, que podem revelar informações cruciais durante uma investigação.
* A utilidade de funcionalidades como "Find Packet", "Packet Comments", "Export Objects" e "Expert Info" para uma análise mais eficiente.

Este conhecimento é essencial para um Analista SOC, permitindo a análise de alertas de IDS/IPS, a investigação de atividades de rede suspeitas, a identificação de malware se comunicando e o entendimento geral do que está acontecendo na rede. O Wireshark é uma ferramenta poderosa, e esta sala forneceu os fundamentos necessários para começar a utilizá-lo de forma eficaz.

---
