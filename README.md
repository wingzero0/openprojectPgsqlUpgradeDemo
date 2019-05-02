# Steps to dump PostgresSQL 9.4 and restore to PostgresSQL 9.6


## dump 9.4 to plain text sql

login openproject7.3 container bash shell and dump db to sql
```bash
# in host os
cd openproject7.3/
docker-compose up -d
docker exec -it openproject73_app_1 bash

# in docker container root bash shell
su postgres

# in postgres bash shell
cd /tmp/pgsqldump/
pg_dump openproject > pg_dump.sql
exit
# exit postgres bash shell
exit

# in host os
cp ./pgdata/pg_dump.sql ../openproject7.4/pgdata/
```

## restore plain text sql to 9.6

login openproject7.4 container bash shell and re-create db with sql
```bash
cd ../openproject7.4/
docker-compose up -d
docker exec -it openproject74_app_1 bash

# in docker container root bash shell
supervisorctl -i
# in supervisor cmd, stop service that may lock db
stop memcached
stop web
stop worker
exit
# exit supervisor cmd

su postgres
# in postgres bash shell
psql
# in psql cmd, re-create db with name "openproject"
DROP DATABASE openproject;
CREATE DATABASE openproject WITH OWNER = openproject;
\q
# exit psql cmd


# in postgres bash shell, restore sql
cd /tmp/pgsqldump/
psql -d openproject -f pg_dump.sql
exit
# exit postgres bash shell

# in root bash shell
supervisorctl -i
# in supervisor cmd
start memcached
start web
start worker
exit
# exit supervisor cmd

exit
# exit container
```