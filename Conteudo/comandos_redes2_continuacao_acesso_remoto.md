# Guia de Comandos - Redes II (Continuação Acesso Remoto: Montagem de Compartilhamento CIFS/Samba)

Este material organiza os comandos e passos do lado **Cliente** para montagem manual e automática de compartilhamentos Samba/CIFS.

---

## Montagem de Compartilhamento de Rede no Cliente

| Comando / Ação | Descrição / Pra que serve | Onde usar | Nível de Usuário |
| :--- | :--- | :--- | :--- |
| `smb://<ip_do_servidor>/` | Endereço digitado no navegador de arquivos gráfico (Nautilus, Dolphin, etc.) para acessar pastas compartilhadas na rede sem precisar montá-las via terminal. | **Cliente** | Comum ou Admin |
| `apt-get install cifs-utils vim` | Instala os utilitários de montagem do protocolo CIFS/SMB (`mount.cifs`) e o editor de texto Vim. É pré-requisito para montar compartilhamentos remotos no sistema de arquivos local. | **Cliente** | **Admin (root)** |
| `vi .samba` *(geralmente em `/root/.samba` ou `~/.samba`)* | Cria o arquivo com credenciais de rede (`user=aluno`, `password=aluno`, `workgroup=RC2`) para permitir montagem automática sem expor senhas em scripts ou no comando de montagem. | **Cliente** | Comum ou Admin |
| `chmod 600 .samba` | Altera a permissão do arquivo de credenciais para leitura e escrita exclusivas do dono (`rw-------`). Impede que outros usuários do sistema leiam a senha. | **Cliente** | Comum ou Admin (dono do arquivo) |
| `vi /etc/fstab` | Abre a tabela de sistemas de arquivos do sistema. Serve para declarar a partição de rede para que ela seja montada automaticamente a cada boot da máquina cliente. | **Cliente** | **Admin (root)** |
| `chmod -R 777 /mnt/home` | Concede permissão total na pasta local do cliente que servirá como ponto de montagem, garantindo que usuários locais possam ler e gravar após a montagem. | **Cliente** | **Admin (root)** |
| Adicionar Marcador no gerenciador gráfico | Cria um atalho no painel lateral do explorador de arquivos gráfico apontando para a pasta montada em `/mnt/home`, facilitando o acesso rápido do usuário. | **Cliente** | Usuário Comum |

---

### Como funciona o arquivo `/etc/fstab` com CIFS

O arquivo `/etc/fstab` contém as instruções de montagem de discos e redes na inicialização do sistema operacional. Para montar o compartilhamento Samba automaticamente com o arquivo `.samba`, adiciona-se uma linha com a seguinte estrutura:

```text
//<IP_DO_SERVIDOR>/<COMPARTILHAMENTO>  /mnt/home  cifs  credentials=/root/.samba,iocharset=utf8,_netdev  0  0
```

- **`//<IP_DO_SERVIDOR>/<COMPARTILHAMENTO>`**: O caminho de rede da pasta exportada pelo Samba.
- **`/mnt/home`**: O diretório local do cliente onde os arquivos remotos aparecerão.
- **`cifs`**: O tipo de sistema de arquivos do protocolo de rede.
- **`credentials=/root/.samba`**: Aponta para o arquivo que contém usuário e senha com permissão restrita (`600`).
- **`_netdev`**: Impede que o sistema tente montar essa pasta antes que a placa de rede e a conexão de rede estejam ativas durante a inicialização.
- **`0 0`**: Desativa dump de backup e checagem de integridade (fsck) para esse compartilhamento de rede.
