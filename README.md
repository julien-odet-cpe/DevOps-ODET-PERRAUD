# **TP DevOps - Compte rendu**

Filière IRC - 4ème année - Promotion 2024

## Auteur :
- Julien ODET
- Alex PERRAUD

---

## **Database**
**1-1 Document your database container essentials: commands and Dockerfile.**

Il s'agit de build l'image *postgres:14.1-alpine*.
Pour cela, on reprend le Dockerfile qui défini l'image source ainsi que les variables d'environment (nom de la base de données, le user, le mot de passe), puis on build à l'aide de la commande :
```
docker build -t pgdb .
```
> NB: le point à la fin de la commande signifie que l'on utilise le Dockerfile qui est là où la commande est exécutée.

Ensuite, on créé le network correspondant avec la commande :
```
docker network create app-network
```

On lance notre application en précisant bien le network avec la commande :
```
docker run --name pgdb -p 5432:5432 --net=app-network -d  pgdb
```

Enfin, on build l'adminer avec la commande :
```
docker build -t adminer .
```

Puis, on le lance avec la commande :
```
docker run -p "8090:8080" --net=app-network --name=adminer -d adminer
```

On accède alors à l'interface de la connexion à la base de données via localhost:8090.

On peut ajouter les scripts de création et d'alimentation des tables *CreateScheme.sql* et *InsertData.sql*.

On ajoute cette ligne dans le Dockerfile pour copier le dossier *data* dans le conteneur afin que les scripts soient exécutés au lancement :
```
COPY data /docker-entrypoint-initdb.d/
```

Puis, on relance :
```
docker run --name pgdb -p 5432:5432 -v volume:/var/lib/postgresql/data --net=app-network -d  pgdb
```
> NB: Le -v sert à obtenir une persistance des données.
---
## **Backend API**
On télécharge le JRE Java avec la commande :
```
docker pull openjdk:11-jre
```

On créé puis compile le fichier *Main.java* avec la commande :
```
javac Main.java
```

### Multistage build

**1-2 Why do we need a multistage build? And explain each step of this dockerfile.**

Nous devons utiliser le multistage build car on compile avec Maven pour obtenir le jar puis on lance le jar ensuite.

Voici le détail du Dockerfile
La première ligne défini qu'elle image nous allons utiliser pour compiler le Java :
```
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build
```

La variable d'environnement *MYAPP_HOME* est définie à */opt/myapp* :
```
ENV MYAPP_HOME /opt/myapp
```

WORKDIR défini l'endroit où les commandes seront exécutées :
```
WORKDIR $MYAPP_HOME
```

Copie des fichiers sources dans le conteneur Docker :
```
COPY pom.xml .
COPY src ./src
```

Création du fichier jar avec Maven :
```
RUN mvn package -DskipTests
```

Image utilisée :
```
FROM amazoncorretto:17
```

Copie du jar dans le conteneur Docker :
```
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar
```

Définission du jar en tant que point d'entrée :
```
ENTRYPOINT java -jar myapp.jar
```

---
## **Http server**
Après avoir construit notre Dockerfile, on établit la redirection des requêtes sur le conteneur Java. Pour cela, dans le fichier de configuration *http.conf*, du conteneur HTTPD, on met :
```
<VirtualHost *:80>
    ProxyPreserveHost On
    ProxyPass / http://javabackendapi:8080/
    ProxyPassReverse / http://javabackendapi:8080/
</VirtualHost>
```

Puis on build :
```
docker build . -t http
```

Et on run : 
```
docker run --name http -p 80:80 --net=app-network -d  http
```

---

## **Link application**
**1-3) Document docker-compose most important commands. 1-4 Document your docker-compose file.**

Commandes importantes du docker-compose :
```
docker-compose up -d
docker-compose done -v
```
> NB: -d pour background et -v pour supprimer les volumes

Voir le fichier *docker-compose.yaml* ainsi que ses commentaires

## **Publish**

**1-5) Document your publication commands and published images in dockerhub.**
On taggue l'image :
```
docker tag tp-docker-database  alexcpe/database:1.0
```

```
docker push alexcpe/database:1.0
```