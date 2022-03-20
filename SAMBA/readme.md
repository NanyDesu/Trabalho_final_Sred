### SAMBA



#### Configurando o serviço de compartilhamento de arquivos Samba:



#### Mudaremos o nome da maquina. 

```
$ sudo hostnamectl set-hostname smb.grupo7.turma914.ifalara.local
```



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
          addresses: [10.9.14.122/24]
          gateway4: 10.9.14.1
          nameservers:
            addresses:
               - 10.9.14.107
               - 10.9.14.117
            search: [grupo7.turma914.ifalara.local]
        ens192:
          dhcp4: false
          addresses: [192.168.0.50/29]
    version: 2
```

![installer_config_smb](images/SMB/installer_cofig.png)


#### Vamos aplicar as alterações.


```
$ sudo netplan apply
```

#### Teste se está funcionando.

```
$ dig @10.9.14.107 gw.grupo7.turma914.ifalara.local
```

---> Verifique se na linha "ANSWER SECTION:" apareceu que foi resolvido, com o dominio e o IP.


---> Vamos verificar se a interface 160 está funcionando:

```
$ systemd-resolve --status ens160
```

#### Veja se o serviço DNS está retornando o DNS do google:


```
$ ping google.com
```

![resolve_ping_smb](images/SMB/ping_google.png)



### Instale o Samba



---> De um update na máquina:


```
$ sudo apt update
```

---> Instale:

```
$ sudo apt install samba 
```

---> verifique o status:

```
$ sudo systemctl status smbd
```

---> Iremos obter a seguinte resposta:


![status_smb](images/SMB/status_samba.png)



---> Verifique se na maquina as portas TCP 445 e 136 estão rodando. 

```
$ netstat -an | grep LISTEN
```

![grep_listen](images/SMB/listen1.png)



#### Backup do arquivo de configuração do samba e criando um novo:


```
$ sudo cp /etc/samba/smb.conf{,.backup}
```


---> Verifique se foi criado um novo arquivo.

```
$ ls -la /etc/samba
```

---> Iremos obter a seguinte resposta:

![etc/samba](images/SMB/etc_samba.png)


---> Usando o comando grep remova os comentarios dos arquivos.


```
$ sudo bash -c 'grep -v -E "^#|^;" /etc/samba/smb.conf.backup | grep . > /etc/samba/smb.conf'
```

---> Vamos editar o arquivo de configurações do samba.

```
$ sudo nano /etc/samba/smb.conf
```

---> Deixando ele dessa maneira:


```
[global]
   workgroup = WORKGROUP
   netbios name = samba-srv
   security = user
   server string = %h server (Samba, Ubuntu)
   interfaces = 127.0.0.1/8 ens160 ens192
   bind interfaces only = yes
   log file = /var/log/samba/log.%m
   max log size = 1000
   logging = file
   panic action = /usr/share/samba/panic-action %d
   server role = standalone server
   obey pam restrictions = yes
   unix password sync = yes
   passwd program = /usr/bin/passwd %u
   passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .
   pam password change = yes
   map to guest = bad user
   usershare allow guests = yes
[printers]
   comment = All Printers
   browseable = no
   path = /var/spool/samba
   printable = yes
   guest ok = no
   read only = yes
   create mask = 0700
[print$]
   comment = Printer Drivers
   path = /var/lib/samba/printers
   browseable = yes
   read only = yes
   guest ok = no
[homes]
   comment = Home Directories
   browseable = yes
   read only = no
   create mask = 0700
   directory mask = 0700
   valid users = %S
[public]
   comment = public anonymous access
   path = /samba/public
   browsable =yes
   create mask = 0660
   directory mask = 0771
   writable = yes
   guest ok = yes
   guest only = yes
   force user = nobody
   force create mode = 0777
   force directory mode = 0777
```
![smb_conf](images/SMB/smb.conf.png)

---> Modifique a pasta public para que apenas os usuarios do grupo sambashare possam acessar:

```
[public]
   comment = public anonymous access
   path = /samba/public
   browsable =yes
   create mask = 0660
   directory mask = 0771
   writable = yes
   guest ok = no
   valid users = @sambashare
   #guest only = yes
   #force user = nobody
   #force create mode = 0777
   #force directory mode = 0777
```

![smb_conf_no](images/SMB/no_guest_smb.png)



---> Reinicie o servidor

```
$ sudo systemctl restart smbd
```



#### Verificação do funcionamento das portas 445 e 139 nas interfaces ens160 e ens192

```
$ netstat -an | grep LISTEN
```

---> Iremos obter a seguinte resposta:


![ve_grep_listen](images/SMB/listen2.png)



#### Crie um usuario: ``aluno``, com a senha: ``alunoifal``:

```
$ sudo adduser aluno
```

---> Iremos obter a seguinte resposta:


```
Adding user `aluno' ...
Adding new group `aluno' (1001) ...
Adding new user `aluno' (1001) with group `aluno' ...
Creating home directory `/home/aluno' ...
Copying files from `/etc/skel' ...
New password: 
Retype new password: 
passwd: password updated successfully
Changing the user information for aluno
Enter the new value, or press ENTER for the default
    Full Name []: Aluno de SRED no IFAL Arapiraca
    Room Number []: 
    Work Phone []: 
    Home Phone []: 
    Other []: 
Is the information correct? [Y/n] y
```

---> Vincule esse usuário ao serviço samba usamos:


```
$ sudo usermod -aG sambashare aluno
```

---> Crie um diretorio com compartilhamento em rede.

```
$ sudo mkdir /home/aluno/sambashare
```

```
$ sudo mkdir -p /samba/public
```



---> Altere a permição da pasta public

```
$ sudo chown -R nobody:nogroup /samba/public
```

```
$ sudo chmod -R 0775 /samba/public
```

```
$ sudo chgrp sambashare /samba/public
```



####  Teste no explorador de arquivos:



---> Em rede digite na barra de rotas o IP:

```
\\10.9.14.122
```


![teste_smb_ip]()



---> Como resultado mostrará as nossas pastas.



---> Com o DNS já implementado acesse usando o nome da maquina ``smb`` de IP: ``10.9.14.122``


```
\\smb.grupo7.turma914.ifalara.local
```


![teste_smb_nome]()



--> Para acessar a pasta public use o usuario: ``aluno`` e a senha: ``alunoifal``



---> Agora teste criando um arquivo no explorador de arquivos.



![teste_smb_texto]()


![teste_smb_ls]()


