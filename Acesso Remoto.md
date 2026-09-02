(cuidar o caminho no Linux e Windows)



Acesso Remoto

Modo texto

-Telnet

-SSH





Modo Gráfico:

-RDP

-UNC



Instala as coisas



Configurar o IP, puxar da máquina que quer acessar(o cliente)



ssh -l <usuário> <ip>





arquivo de configuração do SSH



**vi/etc/ssh/sshd\_config**:

mudar o numero da porta padrão(por segurança)



quando altera arquivo de configuração de um serviço, dar restart naquele serviço



ssh -p <porta> (opcional) -l root <ip>1







apt-get install samba



mv /etc/samba/smb.conf /etc/samba/smb/smb.conf .BKP -> cria backup do arquivo original



vi /etc/samba/smb.conf



Confs divididas em seções divididas em \[]

\[global]

&nbsp;	#Grupo de trabalhos

&nbsp;	workgroup = RC2



&nbsp;	#Nome como as maquinas Windows vão detecar

&nbsp;	netbios name = Samba-Server RC2	



 	#Nome como as maquinas Linux vão detecar

&nbsp;	server string = %h Samba-Server RC2



&nbsp;	#Tipo do servidor compartilhamento (user); controlador de domínio(domain)

&nbsp;	security = user



&nbsp;	#Arquivo de log de conexões com o samba %m=IP, %u=usuario

&nbsp;	log file = /var/log/samba/%m.%u.log





&nbsp;	#Tamano arquivo de log

&nbsp;	max log size = 100000



&nbsp;	#secao de compartilhamento do diretório pessoal de cada usuário





\[homes]

&nbsp;	#Parametro para obter o nome do usuário (%S cria compartilhamento pra todos os usuários do sistema)

&nbsp;	valid users = %S



&nbsp;	# permissão de leitura

&nbsp;	read only = no



&nbsp;	#permissao de visibilidade

&nbsp;	browseable = yes



&nbsp;	#permissao de escrita

&nbsp;	create mask = 0700



&nbsp;	#permissao de acesso/leitura

&nbsp;	directory mask = 0700



\#compartilhamento de acesso publico

\[publico]

&nbsp;	comment = Diretorio



&nbsp;	#disponibilidade

&nbsp;	available = yes



&nbsp;	# Visibilidade

&nbsp;	browseable = yes



&nbsp;	#pasta que esta sendo compartilhada

&nbsp;	path = /mnt/dados 

&nbsp;	

&nbsp;	#permissão de visibilidade e leitura

&nbsp;	public = yes



&nbsp;	#permissao de escrita

&nbsp;	writable = yes









service smbd restart

service nmbd restart

&nbsp;	

&nbsp;	

&nbsp;	

smbpasswd -a <usuario>













chmod -R 777 /mnt/dados -> da permissão total a qual usuário do sistema pra leitura, escrita e execução pra esse diretório







&nbsp;	

&nbsp;	

&nbsp;	

