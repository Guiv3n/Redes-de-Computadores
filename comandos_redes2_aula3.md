# Guia de Comandos - Redes II (Aula 3: Configuração de Interfaces e Servidor DHCP)

Este material organiza os comandos da Aula 3, com foco em configuração de rede estática/dinâmica, gerenciamento de interfaces, instalação e parametrização do serviço ISC-DHCP-Server.

---

## 1. Configuração e Controle de Interfaces de Rede

| Comando | Descrição / Pra que serve | Onde usar | Nível de Usuário |
| :--- | :--- | :--- | :--- |
| `vi /etc/network/interfaces` | Abre o arquivo principal de configuração de rede no Debian/Ubuntu para definir IP estático, DHCP, gateway e máscara. | Cliente e Servidor | **Admin (root)** |
| `ifdown <interface>` | Desativa/desliga uma interface de rede específica (ex: `eth0`, `enp0s3`). *Nota: nas anotações constava `<nome_da_maquina>`, mas o parâmetro correto do comando é a interface de rede.* | Cliente e Servidor | **Admin (root)** |
| `ifup <interface>` | Ativa/sobe a interface de rede especificada aplicando as configurações definidas em `/etc/network/interfaces`. | Cliente e Servidor | **Admin (root)** |
| `ip a show <interface>` | Filtra e exibe o endereço IP, status e detalhes de uma interface de rede específica. | Cliente e Servidor | Comum ou Admin |
| `source .bashrc` | Recarrega as configurações do arquivo `.bashrc` na sessão atual sem precisar deslogar (útil após criar aliases ou caminhos). | Cliente e Servidor | Comum ou Admin |

---

## 2. Instalação e Gerenciamento do Serviço DHCP

| Comando | Descrição / Pra que serve | Onde usar | Nível de Usuário |
| :--- | :--- | :--- | :--- |
| `apt-get install isc-dhcp-server` | Instala o pacote do servidor DHCP (`isc-dhcp-server`) no sistema. | **Servidor** | **Admin (root)** |
| `service isc-dhcp-server restart` | Reinicia o serviço DHCP para aplicar alterações feitas nos arquivos de configuração ou verificar erros de sintaxe. | **Servidor** | **Admin (root)** |

---

## 3. Arquivos de Configuração do DHCP

| Arquivo / Comando | Descrição / Pra que serve | Onde usar | Nível de Usuário |
| :--- | :--- | :--- | :--- |
| `vi /etc/default/isc-dhcp-server` | Configura as opções de inicialização do DHCP, principalmente a diretiva `INTERFACESv4=""` (define em qual placa de rede o DHCP vai escutar). | **Servidor** | **Admin (root)** |
| `vi /etc/dhcp/dhcpd.conf` | Arquivo principal onde são declarados os escopos de rede, faixas de IP (`range`), máscara de sub-rede, gateway padrão, servidores DNS e leases estáticos. | **Servidor** | **Admin (root)** |
| `vi /etc/default/isc-dhcpd.conf` | *Correção técnica:* Variação de anotação; o caminho padrão correto e ativo no Debian para as sub-redes/IPs é o `/etc/dhcp/dhcpd.conf`. | **Servidor** | **Admin (root)** |
