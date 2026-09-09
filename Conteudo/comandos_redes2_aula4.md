# Guia de Comandos - Redes II (Aula 4: Roteamento, NAT e Firewall com Iptables)

Este material organiza os comandos da Aula 4, com foco no encaminhamento de pacotes (IP forwarding), tradução de endereços de rede (NAT / Masquerade) e persistência de regras de firewall.

---

## 1. Roteamento, NAT e Firewall

| Comando / Arquivo | Descrição / Pra que serve | Onde usar | Nível de Usuário |
| :--- | :--- | :--- | :--- |
| `vi /etc/sysctl.conf` *(ou `/etc/sysctl`)* | Arquivo onde se ativa o roteamento/encaminhamento de pacotes no kernel do Linux, descomentando ou adicionando `net.ipv4.ip_forward=1`. Sem isso, o Linux não repassa pacotes entre redes. | **Servidor (Roteador/Gateway)** | **Admin (root)** |
| `iptables -t nat -A POSTROUTING -o enpc0s3 -s 192.168.10.0/24 -j MASQUERADE` | Cria uma regra de NAT (Masquerade) na tabela `nat` da cadeia `POSTROUTING`. Permite que a rede interna (`192.168.10.0/24`) acesse a rede externa/Internet através da interface de saída (`enpc0s3` ou `enp0s3`), mascarando os IPs locais com o IP da interface externa. | **Servidor (Roteador/Gateway)** | **Admin (root)** |
| `iptables-save > /etc/iptables/rules.v4` | Salva o estado atual das regras do iptables em disco. Como as regras do iptables se perdem ao reiniciar o sistema, esse comando grava as regras no arquivo lido pelo pacote `iptables-persistent` durante o boot. | **Servidor (Roteador/Gateway)** | **Admin (root)** |
| `cat /etc/iptables/rules.v4` | Lê e exibe no terminal o conteúdo do arquivo onde as regras persistentes do iptables foram salvas, permitindo conferir se o NAT foi gravado com sucesso. | **Servidor (Roteador/Gateway)** | Comum ou Admin |
