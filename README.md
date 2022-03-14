# INSTITUTO FEDERAL DE ALAGOAS - CAMPUS ARAPIRACA
## Alunos:
- ELAINE CLIS DE MENEZES SILVA - 914
- RONALD EMANUEL DOS SANTOS SILVA - 914
- MARIA THAYSE NASCIMENTO DA SILVA - 914
- LUCAS EMANUEL LIMA DA SILVA - 914

### Prof. Alaelson Jatobá

# CONFIGURAÇÕES DAS INTERFACES DE REDE

| IP da Subrede:  |  10.9.14.0/24  | 192.168.14.48/29  | 
| ------------------- | ------------------- | ------------------- |
|IP de Broadcast: |  10.9.14.255/24 | 192.168.14.55 | 
|IP do GW:| ens160: 10.9.14.128 | ens192: 192.168.14.49| 
|IP do SAMBA:| ens160: 10.9.14.122 | ens192: 192.168.14.50| 
|IP do NS1: | ens160: 10.9.14.107 | ens192: 192.168.14.51| 
|IP do NS2:| ens160: 10.9.14.117| ens192: 192.168.14.52| 
|IP do WEB | ens160: 10.9.14.223 | ens192: 192.168.14.53|
|IP do BD| ens160: 10.9.14.224 | ens192: 192.168.14.54|


# Definição de Nomes e Domínio (<grupo>.<turma>.ifalara.local):
	
| VM  |  Domínio (zona): | grupo7.turma914.ifalara.local  | 
| ------------------- | ------------------- | ------------------- |
|Aluno28 |  FQDN do GW: | gw.grupo7.turma914.ifalara.local | 
|Aluno22| FQDN do SAMBA: | smb.grupo7.turma914.ifalara.local| 
|Aluno07| FQDN do NS1:| ns1.grupo7.turma914.ifalara.local| 
|Aluno17 | FQDN do NS2: | ns2.grupo7.turma914.ifalara.local| 
|Grupo7VM(www)| FQDN do WEB| www.grupo7.turma914.ifalara.local| 
|Grupo7VM(bd) | FQDN do BD | bd.grupo7.turma914.ifalara.local|	
	


### NS1


Nesta maquina ns1 vamos configurar o serviço de resolução de nome (DNS) com o Bind9, utilizando à como DNS Master. 

0- Primeiro vamos mudar o nome da maquina para o nome escolhhido que está na tabela:
> ns1.grupo7.turma914.ifalara.local
Com o comando:
$ sudo hostnamectl set-hostname ns1.grupo7.turma914.ifalara.local

1- Como primeiro devemos instalar o bind9.
Mas primeiro vamos fazer um update:
$ sudo apt update
Agora vamos instalar o bind:
$ sudo apt-get install bind9 dnsutils bind9-doc
Em seguida vamos verificar se o Bind9 está funcionando:
$ sudo systemctl status bind9
---

2- Vamos configurar os arquivos de zonas.
Mas primeiro vamos criar o diretorio zones para alocar os nossos arquivos de zonas, com: 
$ sudo mkdir /etc/bind/zones
Agora vamos copiar os arquivos para a pasta zones.

Zona Direta:
$ sudo cp /etc/bind/db.empty /etc/bind/zones/db.grupo7.turma914.ifalara.local
Obs.: grupo7.turma914.ifalara.local é o nome do dominio escolhido pelo grupo 7;

Zona Reversa:
$  sudo cp /etc/bind/db.127 /etc/bind/zones/db.10.9.14.rev
Obs.: 10.9.14 é o nosso espaço de rede;

Vamos editar o arquivo "db.grupo7.turma914.ifalara.local":
$ sudo nano /etc/bind/zones/db.grupo4.turma914.ifalara.local
Nele vamos colocar os DNS e os IPs:

RESULTADO
;
; BIND data file for internal network
;
$ORIGIN grupo7.turma914.ifalara.local.
$TTL	3h
@	IN	SOA	ns1.grupo7.turma914.ifalara.local. root.grupo7.turma914.ifalara.local. (
			      2022031400		; Serial
			      3h	; Refresh
			      1h	; Retry
			      1w	; Expire
			      1h )	; Negative Cache TTL
;nameservers
@	IN	NS	ns1.grupo7.turma914.ifalara.local.
@	IN	NS	ns2.grupo7.turma914.ifalara.local.

;hosts
ns1.grupo7.turma914.ifalara.local.	  IN	A	10.9.14.107
ns2.grupo7.turma914.ifalara.local.	  IN	A	10.9.14.117
gw.grupo7.turma914.ifalara.local.	  IN 	A	10.9.14.128
smb.grupo7.turma914.ifalara.local.	  IN	A	10.9.14.122
www.grupo7.turma914.ifalara.local.	  IN 	A	10.9.14.223
bd.grupo7.turma914.ifalara.local.	  IN 	A	10.9.14.224

![sudo nano /etc/netplan/00-installer-config.yaml](https://github.com/NanyDesu/Trabalho_final_Sred/blob/main/images/NS1/zonaDireta.PNG)
---

Vamos editar o arquivo "db.10.9.14.rev":
$ sudo nano /etc/bind/zones/db.10.9.14.rev
Nele vamos colocar os DNS e os IPs:

RESULTADO
;
; BIND reverse data file of reverse zone for local area network 10.9.14.0/24
;
$TTL    604800
@       IN      SOA     grupo7.turma914.ifalara.local. root.grupo7.turma914.ifalara.local. (
                              2022031400         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL

; name servers
@      IN      NS      ns1.grup7.turma914.ifalara.local.
@      IN      NS      ns2.grupo7.turma914.ifalara.local.

; PTR Records
107   IN      PTR     ns1.grupo7.turma914.ifalara.local.
117   IN      PTR     ns2.grupo7.turma914.ifalara.local.
128   IN      PTR     gw.grupo7.turma914.ifalara.local.
122   IN      PTR     smb.grupo7.turma914.ifalara.local.
223   IN      PTR     www.grupo7.turma914.ifalara.local.
224   IN      PTR     bd.grupo7.turma914.ifalara.local.

![sudo nano /etc/netplan/00-installer-config.yaml](https://github.com/NanyDesu/Trabalho_final_Sred/blob/main/images/NS1/zonaReversa.PNG)
---

3- Vamos ativar as zonas, então devesse configurar o arquivo "named.conf.local":
$  sudo nano /etc/bind/named.conf.local

RESULTADO
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

Nele vamos adicionas as seguintes linhas de codigos logo a baixo, pulando uma linha:

zone "grupo7.turma914.ifalara.local" {
	type master;
	file "/etc/bind/zones/db.grupo4.turma914.ifalara.local";
	allow-transfer{ 10.9.14.117; };  
	allow-query{any;};
};

zone "14.9.10.in-addr.arpa" IN {
	type master;
	file "/etc/bind/zones/db.10.9.14.rev";
	allow-transfer{ 10.9.14.117; };
};

Em zones tem que ter o nome de dominio escolhido. Em "allow-transfer" o IP é o do ns2, ou seja Slave.

![sudo nano /etc/netplan/00-installer-config.yaml](https://github.com/NanyDesu/Trabalho_final_Sred/blob/main/images/NS1/namedConfLocal.PNG)
---

4- Agora vamos verificar a sintax dos arquivos que editamos.
Para checar a sintax do arquivo named.conf.local usamos:
$ sudo named-checkconf
Vamos entrar no diretório zones:
$ cd /etc/bind/zones
Vamos Verificar se o arquivo "db.grupo4.turma914.ifalara.local" está ok.
$ sudo named-checkzone grupo4.turma914.ifalara.local db.grupo7.turma914.ifalara.local
RESULTADO
...
zone grupo7.turma914.ifalara.local/IN: loaded serial 1
OK

Vamos Verificar se o arquivo "db.10.9.14.rev" está ok.
$ sudo named-checkzone 14.9.10.in-addr.arpa db.10.9.14.rev
RESULTADO
...	
zone 14.9.10.in-addr.arpa/IN: loaded serial 1
OK

---

5- Vamos configurar para resolver apenas IPv4.
Entraremos no arquivo name e adicionaremos "-4" na linha "OPTINS=-u bind":
$ sudo nano /etc/default/named
RESULTADO
# run resolvconf?
RESOLVCONF=no

# startup options for the server
OPTIONS="-4 -u bind"
---

6- Vamos reiniciar o bind, com:
$ sudo systemctl restart bind9

7- Agora vamos configurar as nossas interfaces, ens160 e ens192:
$ sudo nano /etc/netplan/00-installer-config.yaml
Na interface ens160 vamos retirar os endereços de IPs e adiconaremos os indereçoes do Master e Sleve.
RESULTADO
network:
  ethernets:
    ens160:
      addresses: [10.9.14.107/24]
      gateway4: 10.9.14.1
      dhcp4: false
      nameservers:
        addresses:
          - 10.9.14.107
          - 10.9.14.117
        search: [grupo7.turma914.ifalara.local]
    ens192:
      addresses: [192.168.14.51/29]
  version: 2

8- Agora vamos testar o DNS Marter.
$ dig @10.9.14.128 gw.grupo7.turma914.ifalara.local
Na linha "ANSWER SECTION:" tem que aparecer que foi resolvido, com o dominio e o IP.

Vamos usar o seguinte comando para verificar se a interface 160 está funcionando:
$ systemd-resolve --status ens160
Para finalizar vamos ver se o nosso serviço DNS revolve o DNS do google:
$ ping google.com
(IMG: dig-system_resolve-ping.PNG)
---
