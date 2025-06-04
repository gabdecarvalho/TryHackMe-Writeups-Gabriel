# Write-up: TryHackMe - Linux Fundamentals

**Plataforma:** TryHackMe

**Salas:** Linux Fundamentals (da trilha Cybersecurity 101)

---

## 1. Introdução e Objetivos das Salas

As salas "Linux Fundamentals" do TryHackMe são projetadas para construir uma base sólida no uso do sistema operacional Linux a partir da linha de comando. O objetivo é capacitar o usuário com as habilidades essenciais para navegar no sistema de arquivos, manipular arquivos e diretórios, gerenciar processos, entender permissões e utilizar comandos comuns que são pré-requisitos para praticamente todas as áreas da tecnologia e, de forma crítica, para a cibersegurança.

Os principais objetivos abordados incluíram:
* Familiarização com o terminal Linux e a execução de comandos.
* Navegação eficiente no sistema de arquivos.
* Criação, visualização, edição, cópia, movimentação e remoção de arquivos e diretórios.
* Compreensão e modificação de permissões de arquivos.
* Introdução aos editores de texto de terminal (como Nano).
* Noções básicas sobre transferência de arquivos (wget, scp) e como servir arquivos localmente.
* Gerenciamento básico de processos e como eles iniciam.
* Introdução à automação de tarefas com crontabs.
* Conceitos de gerenciamento de pacotes e logs do sistema.

---

## 2. Comandos e Conceitos Chave Abordados/Revisados

Durante esta(s) sala(s), os seguintes comandos e conceitos fundamentais do Linux foram o foco principal:

* **Navegação no Sistema de Arquivos:**
    * `pwd`, `cd`, `ls` (com opções como `-l`, `-a`, `-lh`)
* **Manipulação de Arquivos e Diretórios:**
    * `touch`, `mkdir`, `rm`, `rmdir`, `cp`, `mv`
* **Visualização de Conteúdo de Arquivos:**
    * `cat`, `less`, `more`, `head`, `tail`
* **Edição de Arquivos (Task 3):**
    * `nano` (e menção ao `vim`)
* **Transferência e Compartilhamento de Arquivos (Task 4):**
    * `wget` (para baixar arquivos da web)
    * `scp` (Secure Copy - para transferir arquivos entre máquinas via SSH)
    * `python3 -m http.server` (para criar um servidor web simples para servir arquivos)
* **Gerenciamento de Processos (Task 5):**
    * `ps` (com opções como `aux`)
    * `top`
    * `kill` (com sinais como `SIGTERM`, `SIGKILL`, `SIGSTOP`)
    * `systemctl` (para gerenciar serviços - `start`, `stop`, `enable`, `disable`)
    * Operadores de background (`&`) e `Ctrl + Z`
    * `fg` (para trazer processos para o foreground)
* **Automação (Task 6):**
    * `crontab` (para agendar tarefas - `crontab -e`)
    * Sintaxe de arquivos crontab (minuto, hora, dia do mês, mês, dia da semana, comando)
* **Gerenciamento de Pacotes (Task 7 - Conceitual):**
    * `apt` (conceitos de `update`, `install`, `remove`)
    * `add-apt-repository` (conceito)
    * Importância de GPG keys e `/etc/apt/sources.list.d/`
* **Logs do Sistema (Task 8):**
    * Localização (`/var/log`)
    * Tipos de logs (ex: `apache2/access.log`, `apache2/error.log`)
* **Permissões:**
    * `chmod`, `chown`
    * `sudo`
* **Busca e Filtragem:**
    * `find`, `grep`
* **Outros Comandos Úteis:**
    * `man`, `clear`, `history`

---

## 3. Metodologia e Resolução dos Desafios (Walkthrough)

Nesta seção, detalharei os passos e comandos utilizados para completar as tarefas e responder às perguntas da(s) sala(s) "Linux Fundamentals", com foco nas Tasks 3, 4, 5, 6 e 8, conforme o conteúdo fornecido.

**Task 3: Editores de Texto de Terminal**

Esta tarefa introduziu o editor de texto `nano` e mencionou o `vim` como uma alternativa mais avançada. O desafio prático envolvia editar um arquivo específico.

1.  **Criando/Editando um arquivo com Nano:**
    Para editar o arquivo "task3" no diretório home do usuário "tryhackme", o comando utilizado foi:
    ```bash
    nano task3 
    ```
    (Assumindo que já estava no diretório `/home/tryhackme` ou usei `nano /home/tryhackme/task3`)

2.  **Inserindo/Modificando Texto:**
    Dentro do editor `nano`, adicionei/modifiquei o texto conforme necessário para revelar a flag. A navegação foi feita com as teclas de seta e a inserção de texto é direta.

3.  **Salvando e Saindo:**
    Para salvar as alterações, utilizei `Ctrl + O` (Write Out), confirmei o nome do arquivo e pressionei Enter. Para sair, utilizei `Ctrl + X`.

4.  **Flag Encontrada:**
    * A flag presente no arquivo `task3` era: `THM{TEXT_EDITORS}`

**Task 4: Utilitários Gerais/Úteis (Transferência de Arquivos)**

Esta tarefa focou em como baixar e servir arquivos no Linux.

1.  **Iniciando o Servidor Web Python:**
    No diretório home do usuário `tryhackme` na máquina da sala, iniciei o servidor HTTP Python para servir os arquivos daquele diretório:
    ```bash
    cd /home/tryhackme 
    python3 -m http.server 8000
    ```
    Isso iniciou um servidor na porta 8000, servindo os arquivos do diretório `/home/tryhackme`.

2.  **Baixando o Arquivo na AttackBox:**
    Em um **novo terminal** na AttackBox (ou na minha máquina, se configurado para acesso), utilizei o `wget` para baixar o arquivo `.flag.txt`:
    ```bash
    wget http://[MACHINE_IP]:8000/.flag.txt
    ```
    (Sendo `[MACHINE_IP]` o IP da máquina da sala).

3.  **Conteúdo do Arquivo .flag.txt:**
    Após o download, visualizei o conteúdo do arquivo:
    ```bash
    cat .flag.txt
    ```
    * O conteúdo do arquivo (a flag) era: `THM{WGET_WEBSERVER}`

4.  **Parando o Servidor Python:**
    No terminal onde o servidor Python estava rodando, usei `Ctrl + C` para pará-lo.

**Task 5: Processos 101**

Esta tarefa abordou a visualização e gerenciamento de processos.

1.  **ID do Novo Processo:**
    * Se o ID do processo anterior fosse "300", o ID do novo processo seria **301**. (O PID geralmente incrementa em 1).

2.  **Sinal para Encerrar Processo de Forma Limpa:**
    * Para encerrar um processo de forma limpa, permitindo que ele realize tarefas de limpeza, enviaríamos o sinal **SIGTERM** (sinal 15).

3.  **Localizando o Processo e a Flag:**
    Para localizar o processo específico rodando na instância, utilizei o comando `ps aux` e procurei por um processo que parecesse incomum ou relacionado ao desafio.
    ```bash
    ps aux | grep "THM{"
    ```
    * A flag fornecida pelo processo era: `THM{PROCESSES}`

4.  **Comando para Parar um Serviço:**
    * Para parar um serviço chamado "myservice" usando `systemctl`, o comando seria:
      ```bash
      sudo systemctl stop myservice
      ```

5.  **Comando para Iniciar Serviço no Boot:**
    * Para configurar o serviço "myservice" para iniciar no boot do sistema, o comando seria:
      ```bash
      sudo systemctl enable myservice
      ```

6.  **Comando para Trazer Processo para Foreground:**
    * Para trazer um processo previamente colocado em background (com `Ctrl + Z` ou `&`) de volta para o foreground, o comando é:
      ```bash
      fg
      ```

**Task 6: Mantendo seu Sistema: Automação (Crontab)**

Esta tarefa focou no agendamento de tarefas com `cron`.

1.  **Analisando o Crontab:**
    Para visualizar o crontab do usuário atual na máquina da sala, usei:
    ```bash
    crontab -l
    ```
    * A resposta para "When will the crontab on the deployed instance (MACHINE_IP) run?" era: `@reboot`

**Task 8: Mantendo seu Sistema: Logs**

Esta tarefa focou na análise de logs do Apache.

1.  **Localizando e Analisando Logs do Apache2:**
    Naveguei até o diretório de logs do Apache2, que geralmente é `/var/log/apache2/`.
    ```bash
    cd /var/log/apache2
    ls -l
    ```
    Os arquivos de interesse são `access.log` (para requisições) e `error.log` (para erros).

2.  **Identificando o IP do Usuário que Visitou o Site:**
    Para encontrar o IP, analisei o `access.log`. Poderia usar `cat`, `less`, ou `grep`.
    ```bash
    less access.log 
    ```
    (Procurei por entradas que parecessem ser de um usuário externo ou específico do desafio).
    * O endereço IP do usuário que visitou o site era: `10.9.232.111`

3.  **Identificando o Arquivo Acessado:**
    Na mesma linha do log de acesso correspondente ao IP encontrado, verifiquei o recurso que foi solicitado.
    * O arquivo que o usuário acessou foi: `catsanddogs.jpg`

---

## 4. Lições Aprendidas e Conclusão

As salas "Linux Fundamentals" do TryHackMe, incluindo as tarefas sobre editores de texto, transferência de arquivos, gerenciamento de processos, automação com cron e análise de logs, foram cruciais para desenvolver uma proficiência prática com a linha de comando Linux.

As principais lições foram:
* A importância de editores de texto de terminal como `nano` para modificações rápidas de arquivos de configuração ou scripts.
* A versatilidade de ferramentas como `wget`, `scp` e o servidor HTTP do Python para transferir arquivos em diferentes cenários, uma habilidade útil tanto para administradores de sistema quanto para pentesters (para exfiltrar dados ou baixar ferramentas).
* O entendimento de como os processos são gerenciados (`ps`, `kill`, `systemctl`), como eles iniciam e como controlá-los no foreground/background, essencial para troubleshooting e para entender o comportamento de malwares ou ferramentas de segurança.
* A capacidade de agendar tarefas com `cron` para automação de rotinas, como backups ou scripts de monitoramento.
* A habilidade de localizar e interpretar logs do sistema (como os do Apache) para identificar atividades, erros ou potenciais incidentes de segurança.

Este conjunto de habilidades é fundamental e forma a espinha dorsal para qualquer profissional que trabalhe com sistemas Linux, sendo diretamente aplicável em investigações de SOC, administração de sistemas seguros, e na execução de testes de intrusão.

---
