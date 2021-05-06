## Manual de instalação do Zabbix 5.4 no Ubuntu 18.04 com Postgres12 e timescaledb


### *1* - Faça download e instale o repositório
```sh
wget https://repo.zabbix.com/zabbix/5.3/ubuntu/pool/main/z/zabbix-release/zabbix-release_5.3-1+ubuntu18.04_all.deb
dpkg -i zabbix-release_5.3-1+ubuntu18.04_all.deb
apt update
```
### *2* - Instalar Zabbix Server, Frontend e Agent
```sh
apt install zabbix-server-pgsql zabbix-frontend-php php7.2-pgsql zabbix-nginx-conf zabbix-sql-scripts zabbix-agent
```

### *3* - Instalar postgres12
Adicione o Repositório do postgres
```sh
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
```
Importe a chave GPG
```sh
echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
```
Agora instale o postgres12
```sh
sudo apt update
sudo apt -y install postgresql-12 postgresql-client-12
```
Vai aparecer uma imagem conforme abaixo
![alt text](/img/postgres.png)

Verifique o status do postgres
```sh
systemctl status postgresql.service
systemctl status postgresql@12-main.service
```
Ativar o serviço na inicialização
```sh
systemctl is-enabled postgresql
```

### *4* - Criar o banco de dados inicial e usuário
```sh
sudo -u postgres createuser --pwprompt zabbix
sudo -u postgres createdb -O zabbix zabbix
```
### *5* - Alterar a senha
```sh
sudo su - postgres
psql -c "alter user postgres with password '1234@Mudar'"
```

### *6* - Importar o schema do banco de dados
```sh
zcat /usr/share/doc/zabbix-sql-scripts/postgresql/create.sql.gz | sudo -u zabbix psql zabbix
```

### *7* - Alterar locale
```sh
locale-gen pt_BR.UTF-8
dpkg-reconfigure locales
update-locale LANG=pt_BR.UTF-8
```

### *8* - Configurar o ngnix
```sh
vi /etc/nginx/sites-available/default
```
Faça uma cópia do arquivo original e altere conforme abaixo:
```sh
server {
listen 80 default_server;
listen [::]:80 default_server;
root /var/www/html;
index index.php index.html index.htm;
server_name _;
location / {
try_files $uri $uri/ =404;
}
location ~ .php$ {
include snippets/fastcgi-php.conf;
fastcgi_pass unix:/var/run/php/php7.3-fpm.sock;
}
}
```
Verifique se o arquivo de configuração Nginx não tem erro.
```sh
nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
Edite o arquivo /etc/zabbix/nginx.conf, e descomente 'listen'.
```sh
# listen 80;
```

### *9* - Reinicie e ative todos os serviços
```sh
systemctl restart zabbix-server zabbix-agent nginx php7.2-fpm
systemctl enable zabbix-server zabbix-agent nginx php7.2-fpm
```
### *10* - Instalar timescaledb
Adicionar o repositório
```sh
sudo add-apt-repository ppa:timescale/timescaledb-ppa
```
Instalar timescaledb
```sh
sudo apt update
sudo apt install timescaledb-postgresql-12
```
Configure o timescaledb
```sh
sudo timescaledb-tune
```
Criar a extensão do timescaledb
```sh
echo "CREATE EXTENSION IF NOT EXISTS timescaledb CASCADE;" | sudo -u postgres psql zabbix
```
Criar as hypertable
```sh
cat /usr/share/doc/zabbix-sql-scripts/postgresql/timescaledb.sql | sudo -u zabbix psql zabbix
```
