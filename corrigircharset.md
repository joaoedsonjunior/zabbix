# Reparando o charset do banco de dados Zabbix


## MYSQL/MARIADB

### 1 - Verificar o charset.
```sh
mysql> SELECT @@character_set_database, @@collation_database;
+--------------------------+----------------------+
| @@character_set_database | @@collation_database |
+--------------------------+----------------------+
| utf8mb4                  | utf8mb4_general_ci   |
+--------------------------+----------------------+
```

Caso seja diferente de 'utf8' e collation diferente de'utf8_bin', fazer o seguinte procedimento:

### 2 - Parar o Zabbix Server
```sh
systemctl stop zabbix-server (Caso de distro baseada em RedHat)
/etc/init.d/zabbix-server stop (Caso de distro baseada em debian)
```

### 3 - Conecte no mysql e execute o comando abaixo
```sh
SELECT CONCAT('ALTER TABLE ',TABLE_SCHEMA,'.',TABLE_NAME,' CONVERT TO CHARACTER SET utf8 COLLATE utf8_bin; ')
AS alter_sql
FROM information_schema.TABLES
WHERE TABLE_SCHEMA = 'zabbixdb';
```

### 4 - Copiar o resultado do select e executar.

