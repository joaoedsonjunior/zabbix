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

### *3* - Instalar postgres
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
IMG

Verifique o status do postgres
```sh
systemctl status postgresql.service
systemctl status postgresql@12-main.service
```
Ativar o serviço na inicialização
```sh
systemctl is-enabled postgresql
```



### *4* - Criar o banco de dados inicial
```sh
sudo -u postgres createuser --pwprompt zabbix
sudo -u postgres createdb -O zabbix zabbix
```

### *5* - Importar o schema do banco de dados
```sh
zcat /usr/share/doc/zabbix-sql-scripts/postgresql/create.sql.gz | sudo -u zabbix psql zabbix
```
