# Tutoriel MySQL

Login/Mdp (MySQL) : root/azerty

## Commandes générales pour MySQL

- Pour se connecter à une base de données : `mysql -h ip_address -P 3306 -u login_bdd -p  database_name`
- Créer une base de données : `CREATE DATABASE database_name;`
- Voir les bases de données : `SHOW databases;`
- Se connecter à la base de donnée souhaitée : `USE database_name`
- Voir l'ensemble des tables de la base de données : `SHOW tables;`
- Commande __BASH__ MySQL : `mysql --execute"CREATE USER 'test'@'%' IDENTIFIED BY 'pwd'; GRANT ALL PRIVILEGES ON *.* TO 'test'@'%' WITH GRANT OPTION;"`
- Commande __BASH__ MySQL : `mysql --execute"SELECT host, user FROM mysql.user;"`

### Gestion des utilisateurs

- Vérifier l'ensemble des utilisateurs créés (droits et portée) : `SELECT host, user FROM mysql.user;`
- Ajouter un utilisateur avec comme accès :
  - `%` accès depuis toutes les adresses
  - `ip_address` accès depuis l'adresse souhaitée UNIQUEMENT
  - `localhost` accès seulement en local (127.0.0.1)

```sql
CREATE USER 'login_bdd'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'login_bdd'@'%' WITH GRANT OPTION;
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

Télécharger l'image du container MySQL (`mysql/mysql-server:tag` pour spécifier une version) :
```shell
docker pull mysql/mysql-server
```

Lancer l'image du container avec les paramètres souhaités :
```shell
docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
```
- `--name some-mysql` : nom du container MySQL souhaité (test-mysql)
- `-e MYSQL_ROOT_PASSWORD=my-secret-pw` : mot de passe souhaité (Admin1234)
- `-d` : permet d'exécuter le container en arrière plan
- `mysql:tag` : version de MySQL utilisé, `:latest` ou vide pour la dernière version

Lancer l'image étape par étape :
- `docker pull mysql/mysql-server`
- `docker run -it --name some-mysql -d -p 3000:3306 mysql/mysql-server`
  - L'attribut `-p 3000:3306` permet de définir les ports. Le __port 3000__ est le port d'entrée qui redirige vers le __port 3306__.
- Pour se connecter en BASH/SHELL :
  - `docker logs mysql1 2>&1 | grep GENERATED` génèrera le mot de passe ROOT de MySQL (exemple : `GENERATED ROOT PASSWORD: Axegh3kAJyDLaRuBemecis&EShOs`)
  - `docker exec -it some-mysql bash` pour se connecter en bash
    - `bash-4.2# mysql -u root -p`
    - Taper le mot de passe, dans notre cas : `Axegh3kAJyDLaRuBemecis&EShOs`
    - `mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'password';` Changer le mot de passe
- Pour se connecter à MySQL :
  - `docker exec -it some-mysql mysql -u root -p` pour se connecter à MySQL
  - `mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'password';` Changer le mot de passe

Récupérer l'adresse IP du container :
```shell
docker inspect some-mysql
```

Se connecter à la base MySQL :
```shell
docker exec -it some-mysql mysql -u root -p
```

#### Utilisation d'un fichier .env

L'utilisation d'un fichier .env permet de passer les paramètres en __variables d'environnement__ et ainsi cacher les paramètres sensibles.
Créer le fichier `touch .env` et l'éditer avec __vim__ ou __nano__ `vim .env`

```shell
MYSQL_ROOT_PASSWORD=my_root_password
MYSQL_DATABASE=database
MYSQL_USER=dbuser
MYSQL_PASSWORD=dbpassword
```

Commande pour lancer le container avec un fichier .env :
```shell
docker run \
    --env-file .env \
    --mount type=volume,src=crv_mysql,dst=/var/lib/mysql \
    -p 3306:3306 \
    -d \
    mysql:latest
```

#### DOCKERFILE MySQL

Le DOCKERFILE permet de mettre en place  le container. Il prend en compte les paramètres à installer.  
Ne pas oublier d'ouvrir et configurer le serveur pour qu'on puisse se connecter à la base depuis l'extérieur. Le fichier `50-server.cnf` et l'ajout de la ligne `bind-address            = 0.0.0.0` permet cette configuration

```dockerfile
FROM debian
WORKDIR /mysql
COPY . /mysql

RUN apt-get update
RUN apt-get -y install \
    mysql-server

# Définition des variables d'environnent Root et User
ENV MYSQL_ROOT_PASSWORD=azerty
ENV MYSQL_USER=bob
ENV MYSQL_PASSWORD=pwd

# Copie le ficher dans le container. Ce fichier permet une connexion externet
# Modification de la ligne 'bind-address = 0.0.0.0' > 'bind-address = 127.0.0.1'
COPY ./50-server.cnf /etc/mysql/mariadb.conf.d/50-server.cnf

EXPOSE 3306
CMD ["/usr/bin/mysqld_safe"]
```

### Docker phpMyAdmin

Après avoir mis en place le container __MySQL__ (ici on l'appelera mon_container_mysql), on peut maintenant mettre en place celui de __phpMyAdmin__.
- [DockerHub phpMyAdmin](https://hub.docker.com/r/phpmyadmin/phpmyadmin/ "DockerHub phpMyAdmin")

Télécharger l'image du container phpMyAdmin puis le lancer :
```shell
docker pull phpmyadmin/phpmyadmin
# Le port 8080 est le port local auquel on accède, qui renvoi vers celui du container le port 80
docker run --name myadmin -d --link mon_container_mysql:db -p 8080:80 phpmyadmin/phpmyadmin
```

Un fois lancé on peut maintenant accéder à la page de phpMyAdmin via l'URL `http://localhost:8080`

# Commandes pour créer et lancer le container

```shell
docker build -t mysql-test .
docker run -it -d --rm -p 3000:3306 --name azerty mysql-test
docker exec -it azerty bash

CREATE USER 'test'@'%' IDENTIFIED BY 'pwd'; GRANT ALL PRIVILEGES ON *.* TO 'test'@'%' WITH GRANT OPTION;

docker exec -it azerty mysql -u root -p
```