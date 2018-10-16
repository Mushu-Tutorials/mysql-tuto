# Tutoriel MySQL

## Commandes générales pour MySQL

- Pour se connecter à une base de données : `mysql -h ip_address -P 3306 -u login_bdd -p  database_name`
- Créer une base de données : `CREATE DATABASE database_name;`
- Voir les bases de données : `SHOW databases;`
- Se connecter à la base de donnée souhaitée : `USE database_name`
- Voir l'ensemble des tables de la base de données : `SHOW tables;`

### Gestion des utilisateurs

- Vérifier l'ensemble des utilisateurs créés (droits et portée) : `SELECT host, user FROM mysql.user;`
- Ajouter un utilisateur avec comme accès :
  - `%` accès depuis toutes les adresses
  - `ip_address` accès depuis l'adresse souhaitée UNIQUEMENT
  - `localhost` accès seulement en local (127.0.0.1)

```sql
CREATE USER 'login_bdd'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'Nom-Administrateur'@'%' WITH GRANT OPTION;
```

### Exporter / Importer une base de données

#### Export

- Exporter une base entière                         (1)
- Exporter toutes les bases de données              (2)
- Exporter seulement des tables spécifiques         (3)
- Compresser la base de données à exporter          (4)
- Exporter une base distante (adresse IP + port)    (5)

```shell
mysqldump -u login_bdd -p database_name > database_backup.sql                       # (1)
mysqldump -u login_bdd -p --all-databases > all_databases_backup.sql                # (2)
mysqldump -u login_bdd -p database_name table1 table2 > table_backup.sql            # (3)
mysqldump -u login_bdd -p database_name | gzip > database_backup.sql.gz             # (4)
mysqldump -P 3306 -h ip_address -u login_bdd -p database_name > database_backup.sql # (5)
```

#### Import

- Importer une base de données                                                                  (1)
- Si la base de données est sur un serveur dédié, on peut utiliser l'adresse IP                 (2)
- Ou le nom d'hôte                                                                              (3)
- Si on ne connait pas le nom de la base de donnée importée, on peut essayer sans le préciser   (4)
 
```shell
mysql -u login_bdd -p -h localhost database_name < database_backup.sql              # (1)
mysql -u login_bdd -p -h 202.54.1.10 database_name < database_backup.sql            # (2)
mysql -u login_bdd -p -h mysql.mabase.org database_name < database_backup.sql       # (3)
mysql -u login_bdd -p -h 202.54.1.10 < database_backup.sql                          # (4)
```

## Docker MySQL + PHPMyAdmin

Tutoriel pour Docker [ici](https://github.com/MushuLeDragon/docker-tuto "Tuto Docker")

### Docker MySQL

- [DockerHub MySQL](https://hub.docker.com/_/mysql/ "DockerHub MySQL")

Télécharger l'image du container MySQL :
```shell
docker pull mysql
```

Lancer l'image du container avec les paramètres souhaités :
- `--name some-mysql` : nom du container MySQL souhaité (test-mysql)
- `-e MYSQL_ROOT_PASSWORD=my-secret-pw` : mot de passe souhaité (Admin1234)
- `-d mysql:tag` : version de MySQL utilisé
```shell
docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
```

Récupérer l'adresse IP du container :
```shell
docker inspect some-mysql
```

Se connecter à la base MySQL :
```shell
docker exec -it some-mysql mysql -u root -p
```

### Docker phpMyAdmin

- [DockerHub phpMyAdmin](https://hub.docker.com/r/phpmyadmin/phpmyadmin/ "DockerHub phpMyAdmin")

Télécharger l'image du container phpMyAdmin :
```shell
docker pull phpmyadmin/phpmyadmin
```
