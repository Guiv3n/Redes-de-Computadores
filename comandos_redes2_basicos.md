# Guia de Comandos Básicos - Redes II

Este material sintetiza e organiza os comandos básicos utilizados em ambiente Linux (máquinas virtuais, clientes e servidores) para administração, navegação e manipulação de arquivos.

---

## 1. Identificação do Prompt e Níveis de Acesso

- **Diferenciação no Prompt (`$` vs `#`):**
  - `$` indica uma sessão de **usuário comum**.
  - `#` indica uma sessão de **superusuário (root / administrador)**.

- **`su -`**
  - **O que é / Pra que serve:** Alterna o terminal para a conta do usuário administrador (`root`), carregando também as variáveis de ambiente completas do admin.
  - **Onde usar:** Cliente ou Servidor (em ambas).
  - **Usuário:** Executado inicialmente pelo **Comum** (requer senha de root) para se tornar **Admin**.

---

## 2. Navegação e Informações do Sistema

| Comando | Descrição / Pra que serve | Onde usar | Nível de Usuário |
| :--- | :--- | :--- | :--- |
| `whoami` | Retorna o nome exato do usuário autenticado no momento no terminal. | Cliente e Servidor | Comum ou Admin |
| `pwd` *(Print Working Directory)* | Exibe o caminho absoluto do diretório onde você se encontra atualmente. | Cliente e Servidor | Comum ou Admin |
| `ls` | Lista os arquivos e diretórios contidos no local atual (ou caminho indicado). | Cliente e Servidor | Comum ou Admin |
| `mkdir <nome>` | Cria uma nova pasta/diretório com o nome especificado. | Cliente e Servidor | Comum (na própria home) / Admin (em diretórios do sistema) |
| `cd <nome>` | Navega / entra no diretório indicado. | Cliente e Servidor | Comum ou Admin |
| `cd ..` | Volta exatamente um nível na árvore de diretórios (pasta pai). | Cliente e Servidor | Comum ou Admin |
| `hostname` | Exibe ou define o nome de identificação da máquina na rede. | Cliente e Servidor | Leitura: Comum / Alteração: Admin |
| `ip a` *(ou `ip addr`)* | Lista todas as interfaces de rede ativas/inativas, endereços IP (IPv4/IPv6), máscaras e status. | Cliente e Servidor | Comum ou Admin |

---

## 3. Gerenciamento de Pacotes (Debian / Ubuntu)

| Comando | Descrição / Pra que serve | Onde usar | Nível de Usuário |
| :--- | :--- | :--- | :--- |
| `apt-get update` | Atualiza a lista local de repositórios e versões de pacotes disponíveis para download (não instala pacotes, apenas sincroniza índices). | Cliente e Servidor | **Apenas Admin (`root` ou com `sudo`)** |
| `apt-get install <programa>` | Baixa e instala o serviço/programa solicitado (ex: servidores SSH, Apache, clientes de teste). | Cliente e Servidor | **Apenas Admin (`root` ou com `sudo`)** |

---

## 4. Manipulação de Arquivos e Editor de Texto (Vim / Shell)

| Comando / Tecla | Descrição / Pra que serve | Onde usar | Nível de Usuário |
| :--- | :--- | :--- | :--- |
| `vi <arquivo>` | Abre ou cria um arquivo no editor de texto em modo terminal. | Cliente e Servidor | Comum (arquivos próprios) / Admin (arquivos em `/etc/`, etc.) |
| **Tecla `i`** (no Vim) | Entra no **Modo de Inserção** (permite digitar e editar o texto). | Cliente e Servidor | Comum ou Admin |
| **`Esc` + `:wq`** (no Vim) | Salva as alterações (*write*) e fecha o editor (*quit*). | Cliente e Servidor | Comum ou Admin |
| **`Esc` + `:q!`** (no Vim) | Sai imediatamente do editor descartando qualquer alteração feita. | Cliente e Servidor | Comum ou Admin |
| **`dd`** (no Vim - modo comando) | Recorta/apaga a linha inteira onde o cursor está posicionado. | Cliente e Servidor | Comum ou Admin |
| **`u`** (no Vim - modo comando) | Desfaz a última ação realizada (*undo* / equivalente a Ctrl+Z). | Cliente e Servidor | Comum ou Admin |
| `cp <origem> <destino>` | Faz uma cópia física do arquivo ou pasta para um novo local ou com outro nome. | Cliente e Servidor | Comum / Admin (depende da pasta de destino) |
| `mv <origem> <destino>` | Move um arquivo de diretório ou o renomeia. | Cliente e Servidor | Comum / Admin (depende das pastas envolvidas) |
| `cat <arquivo>` | Imprime todo o conteúdo de um arquivo de texto diretamente no terminal. | Cliente e Servidor | Comum (se tiver permissão de leitura) ou Admin |
| `vi ~/.bashrc` | Abre o script de inicialização do terminal do usuário para configurar alias, variáveis ou prompt. | Cliente e Servidor | Comum (seu próprio perfil) / Admin |

---

## 5. Permissões de Arquivos no Linux

- **`ls -l`**
  - **O que é / Pra que serve:** Lista os itens em formato longo, detalhando permissões, dono, grupo, tamanho e data de modificação.
  - **Onde usar:** Cliente e Servidor.
  - **Usuário:** Comum ou Admin.

### Estrutura das Permissões (Exemplo: `-rw-r--r--`)

- **Tipos de Permissão:**
  - `r` (Read / Leitura): Permite ler o arquivo ou listar o conteúdo do diretório.
  - `w` (Write / Escrita): Permite modificar, salvar ou apagar o arquivo.
  - `x` (Execute / Execução): Permite executar como script/binário ou entrar no diretório (`cd`).

- **Divisão dos 10 Caracteres:**
  - **1º caractere:** Tipo do item (`-` para arquivo comum, `d` para diretório, `l` para link simbólico).
  - **2º ao 4º caracteres (`rw-`):** Permissões do **Dono / Proprietário** (*User*).
  - **5º ao 7º caracteres (`r--`):** Permissões do **Grupo** (*Group*).
  - **8º ao 10º caracteres (`r--`):** Permissões dos **Outros / Demais usuários** (*Others*).
