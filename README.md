# SERVIÇO DE REDES / PROJETO E INFRAESTRUTURA DE REDES

##### ELAINE CLIS DE MENEZES SILVA - 914
##### MARIA THAYSE NASCIMENTO DA SILVA - 914
##### LUCAS EMANUEL LIMA DA SILVA - 914

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


#### VM configurada com (DNS) e o Bind9, utilizando-a como DNS Master. 

 
```	
$ sudo hostnamectl set-hostname ns1.grupo7.turma914.ifalara.local
```
	

	
##### Vamos realizar um update e instalar o bind9.

	
	
```	
$ sudo apt update
```
	
	

```
$ sudo apt-get install bind9 dnsutils bind9-doc
```
	
	
	
##### Verificação do Bind9 :
	
	
```
$ sudo systemctl status bind9
```
	
---

#####  Configuração de arquivos de zonas.
iremos precisar de um diretorio zones onde os nossos arquivos de zonas serão armazenados. Para realizar tal tarefa iremos utilizar o seguinte codigo: 
	
	
```
$ sudo mkdir /etc/bind/zones
```
	

	
--> Enviar arquivos para a pasta zones.

Zona Direta:


	
```
$ sudo cp /etc/bind/db.empty /etc/bind/zones/db.grupo7.turma914.ifalara.local
```
	
	
--> grupo7.turma914.ifalara.local nome de dominio

Zona Reversa:
	
	
	
	
```
$  sudo cp /etc/bind/db.127 /etc/bind/zones/db.10.9.14.rev
```

	
	

---> editar arquivo "db.grupo7.turma914.ifalara.local" ---> colocar ip´s e dns:
	
	
	
```
$ sudo nano /etc/bind/zones/db.grupo7.turma914.ifalara.local
```
	
	

#### Iremos obter a seguinte resposta:
	
	
	
```
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
```
	
	
	
	
---	
![sudo nano /etc/netplan/00-installer-config.yaml](https://github.com/NanyDesu/Trabalho_final_Sred/blob/main/images/NS1/zonaDireta.PNG)
---

	
	
---> editar o arquivo "db.10.9.14.rev" onde mais uma vez iremos adicionar os ip´s e os dns:
	

```
$ sudo nano /etc/bind/zones/db.10.9.14.rev
```
	
	
#### Iremos obter a seguinte resposta:
	
	
	
```
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
```

	
---	
![sudo nano /etc/netplan/00-installer-config.yaml](https://github.com/NanyDesu/Trabalho_final_Sred/blob/main/images/NS1/zonaReversa.PNG)
---
	
	

##### Para tornar as zonas ativas , precisamos configurar o arquivo "named.conf.local":
	
	
```	
$  sudo nano /etc/bind/named.conf.local
```
	
---	
#### Iremos obter a seguinte resposta:
---	
	
	
	
	
```	
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

Nele vamos adicionas as seguintes linhas de codigos logo a baixo, pulando uma linha:

zone "grupo7.turma914.ifalara.local" {
	type master;
	file "/etc/bind/zones/db.grupo7.turma914.ifalara.local";
	allow-transfer{ 10.9.14.117; };  
	allow-query{any;};
};

zone "14.9.10.in-addr.arpa" IN {
	type master;
	file "/etc/bind/zones/db.10.9.14.rev";
	allow-transfer{ 10.9.14.117; };
};
```
	
	
	
Nas zones devemos escolher um nome de dominio, Em "allow-transfer" o IP é o do ns2, ou seja Slave.
	
	
---
![sudo nano /etc/netplan/00-installer-config.yaml](https://github.com/NanyDesu/Trabalho_final_Sred/blob/main/images/NS1/namedConfLocal.PNG)
---


##### Iremos verificar a sintax do arquivo named.conf.local usando:
	
	
	
```
$ sudo named-checkconf
```
	
---	
---> para entrar no diretório zones utilize o comando abaixo:
---	

	
```
$ cd /etc/bind/zones
```
	
	
	
	
e para garantir que "db.grupo7.turma914.ifalara.local" está funcionando:
	
	
	
	
```
$ sudo named-checkzone grupo7.turma914.ifalara.local db.grupo7.turma914.ifalara.local
```
	
	
	
#### Iremos obter a seguinte resposta:

	
	
	
	
	
```
zone grupo7.turma914.ifalara.local/IN: loaded serial 1
OK
```
	
---	
Iremos Verificar se o arquivo "db.10.9.14.rev" está certo a partir de.
---	

	
```
$ sudo named-checkzone 14.9.10.in-addr.arpa db.10.9.14.rev
```	
	
#### Iremos obter a seguinte resposta:
	
```
zone 14.9.10.in-addr.arpa/IN: loaded serial 1
OK
```
	
	
	
---

	
	
##### Devemos então realizar uma configuração para apenas IPv4.
No arquivo name vamos adicionar "-4" na linha "OPTINS=-u bind"
	
	
	
```
$ sudo nano /etc/default/named
```
	
	
#### Iremos obter a seguinte resposta:
	
	
	
```
# run resolvconf?
RESOLVCONF=no
# startup options for the server
OPTIONS="-4 -u bind"
```
---

#### Reiniciando o bind:
```
$ sudo systemctl restart bind9
```
##### Configuração de interface:
```
$ sudo nano /etc/netplan/00-installer-config.yaml
	
---> Para a ens160 vamos manter apenas os endereçoes do Master e Sleve.
```	
#### Iremos obter a seguinte resposta:
```
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
```
#### Teste do DNS Master.
```
$ dig @10.9.14.128 gw.grupo7.turma914.ifalara.local
```
---> Na linha "ANSWER SECTION:" precisa aparecr o dominio e o IP juntamente com a informação se foi resolvido ou não.

---> Para verificar a funcionalidade da 160:
```
$ systemd-resolve --status ens160
```
---> Teste: resolvendo o DNS do google:
```
$ ping google.com
```
	
![sudo nano /etc/netplan/00-installer-config.yaml](https://github.com/NanyDesu/Trabalho_final_Sred/blob/main/images/NS1/dig-system_resolve-ping.PNG)



### Página Web

#### Configuração do servidor web da maquina "www" utilizando a ferramenta Apache2 e fazer a configuração da maquina como cliente do serviço DNS. 

-- Primeiro iremos mudar o nome da máquina:

```
$ sudo hostnamectl set-hostname ns2.grupo7.turma914.ifalara.local
```
---

---> Depois iremos configurar essa maquina como cliente do serviço DNS. 

```
$ sudo nano /etc/netplan/00-installer-config.yaml
```
#### Resultado
```
#This is the network config written by 'subiquity'
network:
    ethernets:
        ens160:
          dhcp4: false
          addresses: [10.9.14.217/24]
          gateway4: 10.9.14.1
          nameservers:
            addresses:
               - 10.9.14.126
               - 10.9.14.109
            search: [grupo4.turma914.ifalara.local]
        ens192:
          dhcp4: false
          addresses: [192.168.0.29/29]
    version: 2
```
![](/www/installer_confing.PNG)

Em seguida vamos aplicar as alterações:
```
$ sudo netplan apply
```
---

---> Na linha "ANSWER SECTION:" tem que aparecer que foi resolvido, com o dominio e o IP. Então iremos verificar se está funcionando:
```
$ systemd-resolve --status ens160
```
---

---> Para finalizar o processo iremos ver se o nosso serviço DNS revolve o DNS do google:
$ ping google.com
![](/www/dig-system_resolve-ping.PNG)
---

-- Iremos instalar o apache e configura-lo para subir nosso servidor apacha, fazendo um update na nossa máquina, via apt:
```
$ sudo apt update
```
---

---> Vamos instalar o apache2 na versão que utilizaremos:
```
$ sudo apt install apache2
```
---

---> Iremos verificar se o servidor web está funcionando:

```
$ sudo systemctl status apache2
```
#### Resultado:
```
● apache2.service - The Apache HTTP Server
     Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2022-03-08 12:30:50 UTC; 4 days ago
       Docs: https://httpd.apache.org/docs/2.4/
   Main PID: 3630 (apache2)
      Tasks: 55 (limit: 1066)
     Memory: 7.0M
     CGroup: /system.slice/apache2.service
             ├─  3630 /usr/sbin/apache2 -k start
             ├─129424 /usr/sbin/apache2 -k start
             └─129425 /usr/sbin/apache2 -k start

Mar 08 12:30:50 www systemd[1]: Started The Apache HTTP Server.
Mar 10 00:00:29 www systemd[1]: Reloading The Apache HTTP Server.
Mar 10 00:00:29 www apachectl[51821]: AH00558: apache2: Could not reliably determine the serv>
Mar 10 00:00:29 www systemd[1]: Reloaded The Apache HTTP Server.
Mar 11 00:00:01 www systemd[1]: Reloading The Apache HTTP Server.
Mar 11 00:00:01 www apachectl[95559]: AH00558: apache2: Could not reliably determine the serv>
Mar 11 00:00:01 www systemd[1]: Reloaded The Apache HTTP Server.
Mar 12 00:00:37 www systemd[1]: Reloading The Apache HTTP Server.
Mar 12 00:00:37 www apachectl[129422]: AH00558: apache2: Could not reliably determine the ser>
Mar 12 00:00:37 www systemd[1]: Reloaded The Apache HTTP Server.
```
![](/www/status_apache2.PNG)

-- Então para acessar o servidor web basta ir ao navegador e colocar o IP da maquina "www", onde foi feita a instalação do Aopache2.
---

---> Iremos criar o nosso site. Primeiro vamos criar um diretorio com o nome do nosso domínio, utilisando:
```
$ sudo mkdir /var/www/grupo7.turma914.ifalara.local
```
-- Vamos atribuir a propriedade do diretório com a variável de ambiente $USER:
```
$ sudo chown -R $USER:$USER /var/www/grupo7.turma914.ifalara.local
```
-- Agora vamos fazer as configurações das permições dos webs hosts, com:
```
$ sudo chmod -R 755 /var/www/grupo7.turma914.ifalara.local
```
-- Depois vamos criar e editar a página index.html:
```
$ sudo nano /var/www/gupo7.turma914.ifalara.local/index.html
```
-- Pra apresentar a página criamos um arquivo de configuração:

```
$ sudo nano /etc/apache2/sites-available/grupo7.turma914.ifalara.local.conf
```
--E neste arquivo iremos colocar o seguinte bloco de código:
```
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName grupo7.turma914.ifalara.local
    ServerAlias www.grupo7.turma914.ifalara.local
    DocumentRoot /var/www/grupo7.turma914.ifalara.local
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
![](/www/domain_conf.PNG)

-- Em segida iremos habilitar o arquivo com a ferramenta a2ensite:
```
$ sudo a2ensite grupo7.turma914.ifalara.local.conf
```
-- E vamos desabilitar o site padrão:
```
$ sudo a2dissite 000-default.conf
```
-- Teste para erros de configuração:
```
$ sudo apache2ctl configtest
```
-- Então iremos reiniciar o Apache:
```
$ sudo systemctl restart apache2
```
#### Pronto agora é só acessar novamente no navegador com o IP da máquina. 
![](/www/a2ensite-configtest-restart.PNG)
![](/www/site.PNG)


### BD



#### Configurando o MySql: 


#### Mudaremos o nome da maquina. 

```
$ sudo hostnamectl set-hostname bd.grupo4.turma914.ifalara.local
```

![hostname](https://github.com/NanyDesu/Trabalho_final_Sred/blob/main/images/BD/hostname.PNG)


#### Vamos configurar a maquina para se tornar cliente do serviço DNS (precisaremos também configurar as interfaces de rede.


```
$ sudo nano /etc/netplan/00-installer-config.yaml
```

#### Iremos obter a seguinte resposta:

```
#This is the network config written by 'subiquity'
network:
    ethernets:
        ens160:
          dhcp4: false
          addresses: [10.9.14.217/24]
          gateway4: 10.9.14.1
          nameservers:
            addresses:
               - 10.9.14.126
               - 10.9.14.109
            search: [grupo4.turma914.ifalara.local]
        ens192:
          dhcp4: false
          addresses: [192.168.0.29/29]
    version: 2
```

![installer_config](https://github.com/NanyDesu/Trabalho_final_Sred/blob/main/images/BD/installer_confing.PNG)


#### Vamos aplicar as alterações.


```
$ sudo netplan apply
```

#### Teste se está funcionando.

```
$ dig @10.9.14.126 gw.grupo4.turma914.ifalara.local
```

#### Verifique se na linha "ANSWER SECTION:" apareceu que foi resolvido, com o dominio e o IP.


#### Vamos verificar se a interface 160 está funcionando:

```
$ systemd-resolve --status ens160
```

#### Veja se o serviço DNS está resolvendo o DNS do google:


```
$ ping google.com
```

![resolve-ping](https://github.com/NanyDesu/Trabalho_final_Sred/blob/main/images/BD/dig-system_resolve-ping.PNG)
