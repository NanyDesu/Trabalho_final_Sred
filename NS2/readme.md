### NS2


#### Configuração do Dns Slave na máquina ns2.


#### Mudaremos o nome da maquina. 

```
$ sudo hostnamectl set-hostname ns1.grupo7.turma914.ifalara.local
```


### Instale o Bind9



---> De um update na máquina:


```
$ sudo apt update
```

---> Instale:

```
$ sudo apt-get install bind9 dnsutils bind9-doc
```

---> verifique o status:

```
$ sudo systemctl status bind9
```


#### Configuração da interfaces de redes:

```
$ sudo nano /etc/netplan/00-installer-config.yaml
```

#### Iremos obter a seguinte resposta:

```
# This is the network config written by 'subiquity'
network:
  ethernets:
    ens160:
      addresses: [10.9.14.117/24]
      gateway4: 10.9.14.1
      dhcp4: false
      nameservers:
        addresses:
          - 10.9.14.107
          - 10.9.14.117
        search: [grupo7.turma914.ifalara.local]
    ens192:
      addresses: [192.168.14.52/29]
      nameservers:
        addresses:
        search: [grupo7.turma914.ifalara.local]
  version: 2
```

![installer_config_yaml](images/NS2/netplan_config.png)



#### Vamos aplicar as alterações.


```
$ sudo netplan apply
```

#### Verifique se está tudo certo:

```
$ ifconfig
```


![ifconfig](images/NS2/ifconfig.png)



#### Verifique se o Bind esta funcionando:


```
$ sudo systemctl status bind9
```

![status_bind9](images/NS2/ifconfig.png)


#### vamos editar o arquivo "named.conf.local" para informar o que esse é o DNS Slave e o ip do DNS do Master. 

```
$ sudo nano /etc/bind/named.conf.local
```

#### RESULTADO:
```
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "grupo7.turma914.ifalara.local" {
        type slave;
        file "/etc/bind/zones/db.ifalara.local";
        masters { 10.9.14.107; };
};

zone "14.9.10.in-addr.arpa" IN {
        type slave;
        file "/etc/bind/zones/db.10.9.14.rev";
        masters { 10.9.14.107; };
};
```

![named_conf_local](images/NS2/named_conf_local.png)



#### Verifique a sintax:

```
$ sudo named-checkconf
```

---> Se não aparecer erro então está tudo certo.



#### Teste se está funcionando:


```
$ systemd-resolve --status
```

![syatemd_resolve](images/NS2/systemd_resolve.png)



#### Teste se o DNS está retornando o do google.


```
$ ping google.com
```

![google_ping](images/NS2/ping_google.com.png)


---


## [Voltar para as tabelas/home](https://github.com/NanyDesu/Trabalho_final_Sred)
