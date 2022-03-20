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

