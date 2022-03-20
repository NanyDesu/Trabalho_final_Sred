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
	
	
---
	
	
# [Acesse aqui os testes das máquinas](https://github.com/NanyDesu/Trabalho_final_Sred/tree/main/images/teste)

	
---

# [NS1](https://github.com/NanyDesu/Trabalho_final_Sred/tree/main/NS1)	
	
---


# [NS2](https://github.com/NanyDesu/Trabalho_final_Sred/tree/main/NS2)	


---

	
# [www](https://github.com/NanyDesu/Trabalho_final_Sred/tree/main/WWW)	
	
	
---

### BD



#### Configurando o MySql: 


#### Mudaremos o nome da maquina. 

```
$ sudo hostnamectl set-hostname bd.grupo7.turma914.ifalara.local
```

![hostname](https://github.com/NanyDesu/Trabalho_final_Sred/blob/main/images/BD/hostname.PNG)


#### Vamos configurar a maquina para se tornar cliente do serviço DNS.


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
          addresses: [10.9.14.224/24]
          gateway4: 10.9.14.1
          nameservers:
            addresses:
               - 10.9.14.107
               - 10.9.14.117
            search: [grupo7.turma914.ifalara.local]
        ens192:
          dhcp4: false
          addresses: [192.168.0.54/29]
    version: 2
```

![installer_config](https://github.com/NanyDesu/Trabalho_final_Sred/blob/main/images/BD/installer_confing.PNG)


#### Vamos aplicar as alterações.


```
$ sudo netplan apply
```

#### Teste se está funcionando.

```
$ dig @10.9.14.107 gw.grupo7.turma914.ifalara.local
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



# [SAMBA](https://github.com/NanyDesu/Trabalho_final_Sred/tree/main/SAMBA)	

	
	
### Gateway


- Iremos configurar o servidor gateway como NAT
- Habilitar e configurar

#### Iniciando:



#### Mudaremos o nome da maquina. 

```
$ sudo hostnamectl set-hostname bd.grupo7.turma914.ifalara.local
```

---> Habilitar o firewall.

```
 sudo ufw enable
```

---> Permitir acesso ssh.


```
 sudo ufw allow ssh
```



![eneble_allow_ssh](images/GW/habilitar_firewall.png)



---> Encaminhamento de pacotes de wan para lan.

```
 sudo nano /etc/ufw/sysctl.conf
``` 

---> Remova a marca "#" do comentário.


```
...
net/ipv4/ip_forwarding=1
...
```


[/etc/ufw/sysctl.conf](images/GW/sysctl.conf.png)


---> Confira os nomes das interface interna e externa.

```
 ifconfig -a
```

```
WAN interface: ens160
LAN interface: ens192
```



![ifconfig -a](images/GW/nome_das_interfaces.png)


---> Recriando o arquivo /etc/rc.local.

```
 sudo nano /etc/rc.local
```

```
#!/bin/bash

# /etc/rc.local

# Default policy to drop all incoming packets.

iptables -P INPUT DROP
iptables -P FORWARD DROP

# Accept incoming packets from localhost and the LAN interface.

iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -i enp192 -j ACCEPT

# Accept incoming packets from the WAN if the router initiated the connection.
#
iptables -A INPUT -i enp160 -m conntrack \

# rc.local needs to exit with 0
# rc.local precisa sair com 0
exit 0
```




![rclocal](images/GW/rc.local.png)



---> Torne o arquivo executável.


```
 sudo chmod 755 /etc/rc.local
```


---> Verifique a atividade do firewall.


```
 sudo ufw status
```

ou 

```
 systemctl status ufw.service
```

![status_firewall](images/GW/status_firewall.png)



---> Reinie a máquina.

```
 sudo reboot
```


---> Recebe de pacotes na porta 445 e 139 da interface externa do gw e encaminha para o servidor interno na porta 445 e 139.


        
```
#Recebe pacotes na porta 445 da interface externa do gw e encaminha para o servidor interno na porta 445
iptables -A PREROUTING -t nat -i enp160 -p tcp –-dport 445 -j DNAT –-to 10.9.14.122:445
iptables -A FORWARD -p tcp -d 10.9.14.122 –-dport 445 -j ACCEPT

#Recebe pacotes na porta 139 da interface externa do gw e encaminha para o servidor interno na porta 139
iptables -A PREROUTING -t nat -i enp160 -p tcp –-dport 139 -j DNAT –-to 10.9.14.122:139
iptables -A FORWARD -p tcp -d 10.9.14.122 –-dport 445 -j ACCEPT
```




---> Recebe pacotes na porta 53 da interface externa do gw e encaminha para o servidor DNS Master interno na porta 53.


```
#Recebe pacotes na porta 53 da interface externa do gw e encaminha para o servidor DNS Mas interno na porta 53
iptables -A PREROUTING -t nat -i enp160 -p tcp –-dport 53 -j DNAT –-to 10.9.14.107:53
iptables -A FORWARD -p udp -d 10.9.14.107 –-dport 53 -j ACCEPT
```



---> Recebe pacotes na porta 80 da interface externa do gw e encaminha para o servidor Web na porta 80.

```
#Recebe pacotes na porta 80 da interface externa do gw e encaminha para o servidor Web na porta 80
iptables -A PREROUTING -t nat -i enp160 -p tcp –-dport 80 -j DNAT –-to 10.9.14.223:53
iptables -A FORWARD -p udp -d 10.9.14.223 –-dport 53 -j ACCEPT
```


![rc.local](images/GW/rc.local_encaminhar_dados.png)





#### Em todas as máquinas: ``ns1``, ``ns2``, ``smb``, ``www`` e ``bd`` , indique que a máquina ``gw`` que é o nosso gateway:



```
$ sudo nano /etc/netplan/00-installer-config.yaml
```

---> Adicione na interface ens160:

```
#gateway4: 10.9.14.1
```


---> Adicione na interface ens192:

```
gateway4: 192.168.14.49
```



