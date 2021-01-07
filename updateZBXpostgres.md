# Update Zabbix 4 para Zabbix 5

## PROCEDIMENTO DE ATUALIZAÇÃO

1. PARE OS PROCESSOS DO ZABBIX
Pare o servidor Zabbix para certificar-se de que nenhum novo dado seja inserido no banco de dados.

```sh
systemctl stop zabbix-server
Se estiver atualizando o proxy, pare o proxy também.
```

2. FAÇA BACKUP DO BANCO DE DADOS ZABBIX EXISTENTE E EXECUTE O DOUBLE.SQL
Este é um passo muito importante. Certifique-se de ter um backup do seu banco de dados. Isso ajudará se o procedimento de atualização falhar (falta de espaço em disco, desligamento, qualquer problema inesperado).

## Executar logado no postgres

```sh
set statement_timeout = 0;

*double.sql*
ALTER TABLE ONLY trends
	ALTER COLUMN value_min TYPE DOUBLE PRECISION,
	ALTER COLUMN value_min SET DEFAULT '0.0000',
	ALTER COLUMN value_avg TYPE DOUBLE PRECISION,
	ALTER COLUMN value_avg SET DEFAULT '0.0000',
	ALTER COLUMN value_max TYPE DOUBLE PRECISION,
	ALTER COLUMN value_max SET DEFAULT '0.0000';
ALTER TABLE ONLY history
	ALTER COLUMN value TYPE DOUBLE PRECISION,
	ALTER COLUMN value SET DEFAULT '0.0000';
```

3. FAÇA BACKUP DE ARQUIVOS DE CONFIGURAÇÃO, ARQUIVOS PHP E BINÁRIOS ZABBIX
Faça uma cópia de backup dos binários do Zabbix, arquivos de configuração e diretório de arquivos PHP.

## Arquivos de configuração:
```sh
mkdir /opt/zabbix-backup/
cp /etc/zabbix/zabbix_server.conf /opt/zabbix-backup/
cp /etc/httpd/conf.d/zabbix.conf /opt/zabbix-backup/
```

## Arquivos PHP e binários Zabbix:
```sh
cp -R /usr/share/zabbix/ /opt/zabbix-backup/
cp -R /usr/share/doc/zabbix-* /opt/zabbix-backup/
```

4. ATUALIZAR PACOTE DE CONFIGURAÇÃO DO REPOSITÓRIO
Para prosseguir com a atualização, seu pacote de repositório atual deve ser atualizado.

```sh
RHEL / CentOS 8
rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/8/x86_64/zabbix-release-5.0-1.el8.noarch.rpm

RHEL / CentOS 7
rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
```

5. ATUALIZAR COMPONENTES DO ZABBIX

```sh
yum upgrade zabbix-*
```

## Para atualizar o front-end da web com Apache no RHEL 8 corretamente, execute também:
```sh
yum install zabbix-apache-conf
```
e faça as alterações necessárias neste arquivo.

Para atualizar o front - end da web no RHEL 7, siga as instruções nesta página (etapas extras são necessárias para instalar o PHP 7.2 ou mais recente).
Em particular, certifique-se de instalar o zabbix-apache-conf-sclpacote se você usar o servidor web Apache.
```sh
yum install zabbix-apache-conf-scl
```
6. REVISE OS PARÂMETROS DE CONFIGURAÇÃO DO COMPONENTE
Consulte as notas de atualização para obter detalhes sobre as alterações obrigatórias .

7. INICIE OS PROCESSOS DO ZABBIX
Inicie os componentes atualizados do Zabbix.
```sh
systemctl start zabbix-server
systemctl start zabbix-proxy
```
8. LIMPE OS COOKIES E O CACHE DO NAVEGADOR DA WEB
Após a atualização, pode ser necessário limpar os cookies e o cache do navegador da web para que a interface da web do Zabbix funcione corretamente.

## Problemas Recorrentes:

Quando o Banco de dados é postgres e der o erro: 
ERROR:  canceling statement due to statement timeout

Executar logado no postgres
set statement_timeout = 0;
