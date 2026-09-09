# Guia de Comandos e Serviços - Redes II (Acesso Remoto e Compartilhamento de Arquivos)

Este material organiza os conceitos, comandos e arquivos de configuração abordados nos módulos de **Acesso Remoto (SSH)** e **Compartilhamento de Arquivos em Rede (Samba / CIFS)**, indicando a função, em qual máquina executar (Cliente ou Servidor) e o nível de privilégio necessário (Usuário Comum ou Administrador/root).

---

## 1. Panorama Teórico: Protocolos de Acesso e Compartilhamento

### Acesso Remoto
- **Modo Texto (CLI):**
  - `Telnet`: Protocolo legado de emulação de terminal; trafega dados e credenciais em texto claro (inseguro).
  - `SSH (Secure Shell)`: Padrão seguro para acesso remoto criptografado via terminal (porta padrão 22 TCP).
- **Modo Gráfico (GUI):**
  - `RDP (Remote Desktop Protocol)`: Protocolo proprietário da Microsoft para área de trabalho remota.
  - `VNC (Virtual Network Computing)` *(na anotação original constava "UNC", que se refere a caminhos de rede como `\\servidor\pasta`)*: Protocolo gráfico aberto para controle remoto de desktops.

### Compartilhamento de Arquivos
- `NFS (Network File System)`: Nativo e otimizado para redes e ambientes Linux/Unix.
- `FTP / SFTP (File Transfer Protocol / SSH File Transfer Protocol)`: Transferência de arquivos cliente-servidor (SFTP encapsula o FTP com segurança sob SSH).
- `Samba (SMB/CIFS)`: Implementação livre do protocolo SMB da Microsoft no Linux, permitindo interoperabilidade de pastas e impressoras entre Linux e Windows.

---

## 2. Acesso Remoto via SSH

| Comando / Arquivo | Descrição / Pra que serve | Onde usar | Nível de Usuário |
| :--- | :--- | :--- | :--- |
| `vi /etc/ssh/sshd_config` | Arquivo de configuração do serviço servidor SSH (`sshd`). Permite alterar a porta padrão (`Port 22`), proibir login direto do root (`PermitRootLogin no`) e restringir usuários. | **Servidor** | **Admin (root)** |
| `service ssh restart` *(ou `systemctl restart ssh`)* | Reinicia o serviço SSH para validar e carregar alterações de porta e segurança feitas em `sshd_config`. | **Servidor** | **Admin (root)** |
| `ssh -l <usuario> <ip_servidor>` *(ou `ssh <usuario>@<ip_servidor>`)* | Inicia uma sessão remota criptografada no terminal do servidor utilizando a porta padrão (22). | **Cliente** | Comum ou Admin |
| `ssh -p <porta> -l <usuario> <ip_servidor>` | Conecta ao servidor SSH especificando uma porta personalizada definida no servidor. | **Cliente** | Comum ou Admin |

---

## 3. Servidor de Compartilhamento de Arquivos: Samba (Lado Servidor)

| Comando / Arquivo | Descrição / Pra que serve | Onde usar | Nível de Usuário |
| :--- | :--- | :--- | :--- |
| `apt-get install samba` | Instala os daemons e utilitários do Samba (`smbd` e `nmbd`). | **Servidor** | **Admin (root)** |
| `mv /etc/samba/smb.conf /etc/samba/smb.conf.BKP` | Renomeia o arquivo de configuração padrão original para mantê-lo como backup de segurança antes de criar um arquivo limpo. | **Servidor** | **Admin (root)** |
| `vi /etc/samba/smb.conf` | Cria/edita a parametrização do Samba (seções `[global]`, `[homes]` e compartilhamentos personalizados como `[publico]`). | **Servidor** | **Admin (root)** |
| `smbpasswd -a <usuario>` | Cria ou define a senha de acesso Samba para um usuário já existente no sistema Linux (`/etc/passwd`). | **Servidor** | **Admin (root)** |
| `chmod -R 777 /mnt/dados` | Atribui permissão total (leitura, escrita e execução para Dono, Grupo e Outros) na pasta compartilhada para evitar bloqueios de permissão do sistema de arquivos local. | **Servidor** | **Admin (root)** |
| `service smbd restart` | Reinicia o daemon responsável pelo compartilhamento de arquivos e autenticação SMB. | **Servidor** | **Admin (root)** |
| `service nmbd restart` | Reinicia o daemon responsável pela resolução de nomes NetBIOS na rede local. | **Servidor** | **Admin (root)** |

### Detalhamento das Diretivas do `smb.conf`:
- **Seção `[global]`:**
  - `workgroup = RC2`: Define o grupo de trabalho Windows/Samba.
  - `netbios name = Samba-Server RC2`: Nome com o qual a máquina se identifica para clientes Windows na rede.
  - `server string = %h Samba-Server RC2`: Descrição/comentário exibido para clientes Linux/Windows (`%h` expande o hostname).
  - `security = user`: Exige autenticação por usuário e senha cadastrados no Samba.
  - `log file = /var/log/samba/%m.%u.log`: Gera logs individualizados por máquina (`%m`) e usuário (`%u`).
  - `max log size = 100000`: Tamanho máximo do arquivo de log em KB antes da rotação.
- **Seção `[homes]`:**
  - Compartilha dinamicamente o diretório pessoal (`/home/<usuario>`) de quem se autentica (`valid users = %S`), com máscara `0700`.
- **Seção `[publico]`:**
  - `path = /mnt/dados`: Caminho físico da pasta no disco do servidor.
  - `public = yes` e `writable = yes`: Permite visibilidade pública e gravação de arquivos por usuários autorizados.

---

## 4. Cliente de Compartilhamento: Acesso e Montagem de Pastas (Lado Cliente)

| Comando / Ação | Descrição / Pra que serve | Onde usar | Nível de Usuário |
| :--- | :--- | :--- | :--- |
| `smb://<ip_do_servidor>/` | Endereço digitado na barra de navegação do gerenciador de arquivos gráfico (Nautilus, Dolphin, etc.) para acessar os compartilhamentos via rede. | **Cliente** | Comum ou Admin |
| `apt-get install cifs-utils vim` | Instala os utilitários de montagem de sistemas de arquivos SMB/CIFS (`mount.cifs`) e o editor de texto. | **Cliente** | **Admin (root)** |
| `vi /root/.samba` *(ou `~/.samba`)* | Cria o arquivo com as credenciais do usuário (`username=...`, `password=...`, `workgroup=...`) para que a montagem ocorra de forma automática e sem expor a senha no terminal. | **Cliente** | Comum ou Admin |
| `chmod 600 .samba` | Restringe as permissões do arquivo de credenciais exclusivamente para leitura/escrita do proprietário (`rw-------`), impedindo outros usuários de lerem a senha. | **Cliente** | Comum ou Admin (dono do arquivo) |
| `chmod -R 777 /mnt/home` | Garante permissões irrestritas no diretório local do cliente que servirá como ponto de montagem. | **Cliente** | **Admin (root)** |
| `vi /etc/fstab` | Tabela de sistemas de arquivos (*File Systems Table*). Usado para configurar a montagem automática do compartilhamento Samba na inicialização da máquina cliente. | **Cliente** | **Admin (root)** |
| Adicionar Marcador no gerenciador gráfico | Cria um atalho no painel lateral do gerenciador de arquivos apontando para `/mnt/...`, facilitando o acesso rápido do usuário. | **Cliente** | Usuário Comum |

### Como funciona o `/etc/fstab` com Samba:
Uma linha de montagem CIFS no `/etc/fstab` segue a sintaxe:
`//<ip_servidor>/<compartilhamento>  /ponto/de/montagem  cifs  credentials=/caminho/.samba,iocharset=utf8,_netdev  0  0`
- `cifs`: Sistema de arquivos do protocolo Samba.
- `credentials=...`: Indica o arquivo com usuário e senha protegidos com `chmod 600`.
- `_netdev`: Avisa o Linux para esperar a rede subir antes de tentar montar a pasta no boot.
