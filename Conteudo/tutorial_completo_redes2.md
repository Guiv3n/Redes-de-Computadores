# Guia Prático Laboratorial - Redes de Computadores II
## Configuração de Infraestrutura: Roteamento/NAT, DHCP, SSH e Compartilhamento de Arquivos (Samba/CIFS)

Este guia apresenta o roteiro completo de laboratório, organizado em blocos cronológicos sequenciais. Cada bloco especifica a **Máquina** (Servidor ou Cliente) e o **Usuário** (Admin/root ou Comum).

---

## SUMÁRIO DE FLUXO DE TRABALHO
1. **BLOCO 1 [Servidor | Usuário Comum]** - Reconhecimento inicial e identificação do prompt.
2. **BLOCO 2 [Servidor | Usuário Admin/root]** - Configuração estática de rede e teste de interfaces.
3. **BLOCO 3 [Servidor | Usuário Admin/root]** - Roteamento no Kernel, NAT (Masquerade) e Persistência do Firewall.
4. **BLOCO 4 [Servidor | Usuário Admin/root]** - Instalação e Configuração do Servidor DHCP.
5. **BLOCO 5 [Cliente | Usuário Admin/root]** - Configuração de rede via DHCP e validação de conectividade.
6. **BLOCO 6 [Servidor | Usuário Admin/root]** - Endurecimento do SSH e Usuários de Rede.
7. **BLOCO 7 [Cliente | Usuário Comum]** - Acesso Remoto Seguro via SSH.
8. **BLOCO 8 [Servidor | Usuário Admin/root]** - Instalação, Configuração do Samba e Criação dos Compartilhamentos.
9. **BLOCO 9 [Cliente | Usuário Admin/root]** - Instalação do CIFS, Credenciais Seguras e Ponto de Montagem Persistente (`/etc/fstab`).
10. **BLOCO 10 [Cliente | Usuário Comum]** - Testes Finais e Marcadores na Interface Gráfica.

---

## BLOCO 1: Reconhecimento Inicial do Sistema
- **Máquina:** Servidor
- **Usuário:** Comum (Prompt: `$`)

### Objetivo
Reconhecer as interfaces de rede e o usuário ativo antes de assumir privilégios administrativos.

1. Identificar o usuário atual:
   ```bash
   whoami
   ```
2. Identificar o nome da máquina:
   ```bash
   hostname
   ```
3. Listar as interfaces de rede e seus nomes atuais:
   ```bash
   ip a
   ```
   *Anote o nome das duas placas:*
   - Interface conectada à Internet (ex: `enp0s3` ou modo NAT/Bridge do hipervisor).
   - Interface voltada para a rede local/clientes (ex: `enp0s8` ou `enp0s3`).

4. Alternar para a conta administrativa (`root`):
   ```bash
   su -
   ```
   *Digite a senha de administrador. O prompt mudará de `$` para `#`.*

---

## BLOCO 2: Configuração de Interfaces no Servidor
- **Máquina:** Servidor
- **Usuário:** Admin / root (Prompt: `#`)

### Objetivo
Configurar a placa interna com IP estático e a placa externa para saída de rede.

1. Abrir o arquivo de interfaces de rede:
   ```bash
   vi /etc/network/interfaces
   ```
2. Configurar a interface interna (exemplo de IP estático `192.168.10.1` na rede interna):
   ```text
   # Interface interna (LAN)
   auto enp0s8
   iface enp0s8 inet static
       address 192.168.10.1
       netmask 255.255.255.0
   ```
   *(Salvar com `Esc` + `:wq`)*

3. Reiniciar/subir a interface para validar as alterações:
   ```bash
   ifdown enp0s8
   ifup enp0s8
   ```
4. Confirmar se o endereço IP foi aplicado corretamente:
   ```bash
   ip a show enp0s8
   ```

---

## BLOCO 3: Roteamento de Pacotes, NAT e Firewall
- **Máquina:** Servidor
- **Usuário:** Admin / root (Prompt: `#`)

### Objetivo
Tornar o servidor um Roteador/Gateway para que a rede local tenha acesso à Internet.

1. Habilitar o repasse de pacotes IPv4 no kernel:
   ```bash
   vi /etc/sysctl.conf
   ```
   *Descomente ou insira a linha:*
   ```text
   net.ipv4.ip_forward=1
   ```
   *(Salvar com `Esc` + `:wq`)*

2. Criar a regra de NAT (Masquerade) no iptables:
   *(Ajuste `enp0s3` para o nome da interface externa conectada à Internet)*
   ```bash
   iptables -t nat -A POSTROUTING -o enp0s3 -s 192.168.10.0/24 -j MASQUERADE
   ```

3. Salvar a regra para persistência após reinicialização:
   ```bash
   iptables-save > /etc/iptables/rules.v4
   ```

4. Conferir se a regra foi salva corretamente no arquivo:
   ```bash
   cat /etc/iptables/rules.v4
   ```

---

## BLOCO 4: Instalação e Configuração do Servidor DHCP
- **Máquina:** Servidor
- **Usuário:** Admin / root (Prompt: `#`)

### Objetivo
Distribuir IPs dinamicamente para os clientes da rede local (`192.168.10.0/24`).

1. Atualizar o índice de pacotes e instalar o ISC-DHCP:
   ```bash
   apt-get update
   apt-get install isc-dhcp-server -y
   ```

2. Definir a interface de rede que responderá às requisições DHCP:
   ```bash
   vi /etc/default/isc-dhcp-server
   ```
   *Configure a diretiva apontando para a interface LAN:*
   ```text
   INTERFACESv4="enp0s8"
   ```
   *(Salvar com `Esc` + `:wq`)*

3. Configurar o escopo de endereços IP:
   ```bash
   vi /etc/dhcp/dhcpd.conf
   ```
   *Adicione as declarações da sub-rede:*
   ```text
   subnet 192.168.10.0 netmask 255.255.255.0 {
       range 192.168.10.100 192.168.10.200;
       option routers 192.168.10.1;
       option domain-name-servers 8.8.8.8, 8.8.4.4;
       default-lease-time 600;
       max-lease-time 7200;
   }
   ```
   *(Salvar com `Esc` + `:wq`)*

4. Reiniciar o serviço do DHCP para validar:
   ```bash
   service isc-dhcp-server restart
   ```

---

## BLOCO 5: Configuração e Teste de Conectividade no Cliente
- **Máquina:** Cliente
- **Usuário:** Admin / root (Prompt: `#`)

### Objetivo
Configurar o cliente para obter IP via DHCP a partir do servidor e validar conectividade.

1. Alternar para o usuário root no cliente:
   ```bash
   su -
   ```

2. Configurar a interface do cliente para usar DHCP:
   ```bash
   vi /etc/network/interfaces
   ```
   *Inserir:*
   ```text
   auto eth0
   iface eth0 inet dhcp
   ```
   *(Substitua `eth0` pelo nome da placa de rede local do cliente e salve com `Esc` + `:wq`)*

3. Renovar a interface de rede:
   ```bash
   ifdown eth0
   ifup eth0
   ```

4. Confirmar o IP recebido dentro da faixa (`192.168.10.X`):
   ```bash
   ip a
   ```

5. Testar conectividade (Gateway e Internet externa via NAT):
   ```bash
   ping -c 4 192.168.10.1
   ping -c 4 8.8.8.8
   ```

---

## BLOCO 6: Configuração de Segurança no SSH e Senha de Rede
- **Máquina:** Servidor
- **Usuário:** Admin / root (Prompt: `#`)

### Objetivo
Configurar o acesso remoto seguro e criar as credenciais do Samba.

1. Abrir o arquivo de configuração do daemon SSH:
   ```bash
   vi /etc/ssh/sshd_config
   ```
   *Ajustes recomendados:*
   - Modificar a porta (opcional/solicitado em aula, ex: `Port 2222` ou manter `Port 22`).
   - Bloquear acesso direto de root: `PermitRootLogin no`.
   *(Salvar com `Esc` + `:wq`)*

2. Reiniciar o serviço SSH:
   ```bash
   service ssh restart
   ```

3. Caso ainda não exista o usuário `aluno` no Linux do servidor, crie-o:
   ```bash
   adduser aluno
   ```

---

## BLOCO 7: Teste do Acesso Remoto via SSH
- **Máquina:** Cliente
- **Usuário:** Comum (Prompt: `$`)

### Objetivo
Conectar remotamente no servidor via terminal.

1. Se estiver como root no cliente, volte para o usuário comum:
   ```bash
   exit
   ```
   *Confirme que o prompt voltou a ser `$` com `whoami`.*

2. Executar a conexão SSH:
   - Se porta padrão (22):
     ```bash
     ssh -l aluno 192.168.10.1
     ```
   - Se foi alterada a porta (ex: 2222):
     ```bash
     ssh -p 2222 -l aluno 192.168.10.1
     ```

3. Digitar `yes` para aceitar a chave do host e inserir a senha do usuário `aluno`.
4. Digite `exit` para encerrar a sessão SSH remota e voltar ao terminal local do cliente.

---

## BLOCO 8: Instalação e Configuração do Servidor Samba
- **Máquina:** Servidor
- **Usuário:** Admin / root (Prompt: `#`)

### Objetivo
Criar as pastas físicas, permissões e diretivas de compartilhamento SMB.

1. Instalar o pacote Samba:
   ```bash
   apt-get install samba -y
   ```

2. Fazer backup do arquivo de configuração original:
   ```bash
   mv /etc/samba/smb.conf /etc/samba/smb.conf.BKP
   ```

3. Criar a pasta pública e ajustar as permissões no sistema de arquivos:
   ```bash
   mkdir -p /mnt/dados
   chmod -R 777 /mnt/dados
   ```

4. Cadastrar a senha do usuário `aluno` no banco de dados do Samba:
   ```bash
   smbpasswd -a aluno
   ```

5. Criar o novo arquivo de configuração do Samba:
   ```bash
   vi /etc/samba/smb.conf
   ```
   *Inserir a parametrização estruturada:*
   ```ini
   [global]
       workgroup = RC2
       netbios name = Samba-Server RC2
       server string = %h Samba-Server RC2
       security = user
       log file = /var/log/samba/%m.%u.log
       max log size = 100000

   [homes]
       comment = Diretorio Pessoal
       valid users = %S
       read only = no
       browseable = yes
       create mask = 0700
       directory mask = 0700

   [publico]
       comment = Diretorio Publico
       path = /mnt/dados
       available = yes
       browseable = yes
       public = yes
       writable = yes
   ```
   *(Salvar com `Esc` + `:wq`)*

6. Reiniciar os serviços do Samba:
   ```bash
   service smbd restart
   service nmbd restart
   ```

---

## BLOCO 9: Instalação do CIFS e Montagem Persistente
- **Máquina:** Cliente
- **Usuário:** Admin / root (Prompt: `#`)

### Objetivo
Configurar o cliente para montar o compartilhamento automaticamente na inicialização via `/etc/fstab`.

1. Acessar como root no cliente:
   ```bash
   su -
   ```

2. Instalar as ferramentas de montagem CIFS e o Vim:
   ```bash
   apt-get update
   apt-get install cifs-utils vim -y
   ```

3. Criar a pasta local que servirá como ponto de montagem:
   ```bash
   mkdir -p /mnt/home
   chmod -R 777 /mnt/home
   ```

4. Criar o arquivo de credenciais seguras:
   ```bash
   vi /root/.samba
   ```
   *Adicionar:*
   ```text
   user=aluno
   password=aluno
   workgroup=RC2
   ```
   *(Salvar com `Esc` + `:wq`)*

5. Proteger o arquivo para leitura exclusiva do root:
   ```bash
   chmod 600 /root/.samba
   ```

6. Configurar a montagem permanente no arquivo de inicialização:
   ```bash
   vi /etc/fstab
   ```
   *Adicionar a linha ao final do arquivo:*
   ```text
   //192.168.10.1/publico  /mnt/home  cifs  credentials=/root/.samba,iocharset=utf8,_netdev  0  0
   ```
   *(Salvar com `Esc` + `:wq`)*

7. Testar a montagem imediata sem precisar reiniciar:
   ```bash
   mount -a
   ```

8. Conferir se a pasta remota está montada:
   ```bash
   ls -l /mnt/home
   ```

---

## BLOCO 10: Validação Gráfica e Marcador de Acesso Rápido
- **Máquina:** Cliente
- **Usuário:** Comum (Interface Gráfica / Prompt: `$`)

### Objetivo
Testar a experiência final do usuário e criar atalhos.

1. Voltar ao usuário comum:
   ```bash
   exit
   ```

2. **Acesso manual rápido via Interface Gráfica:**
   - Abra o Gerenciador de Arquivos (Nautilus).
   - Na barra de endereços (Ctrl + L), digite:
     ```text
     smb://192.168.10.1/
     ```
   - Pressione Enter e informe o usuário `aluno` e a senha `aluno`.

3. **Criar Marcador do Ponto de Montagem Permanente:**
   - No gerenciador de arquivos, navegue até `/mnt/home`.
   - Clique no menu superior ou clique com o botão direito e selecione **Adicionar Marcador** (ou arraste a pasta para a barra lateral).
   - A pasta compartilhada ficará disponível com um clique a qualquer momento.
