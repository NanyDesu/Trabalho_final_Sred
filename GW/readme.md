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



