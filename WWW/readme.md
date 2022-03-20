### Página Web

#### Configuração do servidor web da maquina "www" utilizando a ferramenta Apache2 e fazer a configuração da maquina como cliente do serviço DNS. 


---> Primeiro iremos mudar o nome da máquina:

```
$ sudo hostnamectl set-hostname www.grupo7.turma914.ifalara.local
```

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
          addresses: [10.9.14.117/24]
          gateway4: 10.9.14.1
          nameservers:
            addresses:
               - 10.9.14.107
               - 10.9.14.117
            search: [grupo7.turma914.ifalara.local]
        ens192:
          dhcp4: false
          addresses: [192.168.0.52/29]
    version: 2
```

![](/www/installer_confing.PNG)

---> Em seguida vamos aplicar as alterações:

```
$ sudo netplan apply
```


---> Na linha "ANSWER SECTION:" tem que aparecer que foi resolvido, com o dominio e o IP. Então iremos verificar se está funcionando:

```
$ systemd-resolve --status ens160
```


---> Para finalizar o processo iremos ver se o nosso serviço DNS revolve o DNS do google:

```
$ ping google.com
```

![sudo nano /etc/netplan/00-installer-config.yaml](https://github.com/NanyDesu/Trabalho_final_Sred/blob/main/images/WWW/dig-system_resolve-ping.PNG)



---> Iremos instalar o apache e configura-lo para subir nosso servidor apacha, fazendo um update na nossa máquina, via apt:

```
$ sudo apt update
```


---> Vamos instalar o apache2 na versão que utilizaremos:

```
$ sudo apt install apache2
```


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

![sudo nano /etc/netplan/00-installer-config.yaml](https://github.com/NanyDesu/Trabalho_final_Sred/blob/main/images/WWW/status_apache2.PNG)

---> Então para acessar o servidor web basta ir ao navegador e colocar o IP da maquina "www", onde foi feita a instalação do Aopache2.


---> Iremos criar o nosso site. Primeiro vamos criar um diretorio com o nome do nosso domínio, utilisando:

```
$ sudo mkdir /var/www/grupo7.turma914.ifalara.local
```

---> Vamos atribuir a propriedade do diretório com a variável de ambiente $USER:

```
$ sudo chown -R $USER:$USER /var/www/grupo7.turma914.ifalara.local
```

---> Agora vamos fazer as configurações das permições dos webs hosts, com:

```
$ sudo chmod -R 755 /var/www/grupo7.turma914.ifalara.local
```

---> Depois vamos criar e editar a página index.html:

```
$ sudo nano /var/www/gupo7.turma914.ifalara.local/index.html
```

---> Pra apresentar a página criamos um arquivo de configuração:

```
$ sudo nano /etc/apache2/sites-available/grupo7.turma914.ifalara.local.conf
```

---> E neste arquivo iremos colocar o seguinte bloco de código:

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

![sudo nano /etc/netplan/00-installer-config.yaml](https://github.com/NanyDesu/Trabalho_final_Sred/blob/main/images/WWW/domain_conf.PNG)


---> Em segida iremos habilitar o arquivo com a ferramenta a2ensite:

```
$ sudo a2ensite grupo7.turma914.ifalara.local.conf
```

---> E vamos desabilitar o site padrão:

```
$ sudo a2dissite 000-default.conf
```

---> Teste para erros de configuração:

```
$ sudo apache2ctl configtest
```

---> Então iremos reiniciar o Apache:

```
$ sudo systemctl restart apache2
```

	
---> Pronto agora é só acessar novamente no navegador com o IP da máquina. 
![sudo nano /etc/netplan/00-installer-config.yaml](https://github.com/NanyDesu/Trabalho_final_Sred/blob/main/images/WWW/a2ensite-configtest-restart.PNG)
![](/www/site.PNG)
