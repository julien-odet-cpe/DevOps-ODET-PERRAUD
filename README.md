# TP DevOps - Compte rendu

Filière IRC - 4ème année - Promotion 2024

## Auteur :
- Julien ODET
- Alex PERRAUD

---

## Database
Il s'agit de build l'image *postgres:14.1-alpine*.
Pour cela, on reprend le Dockerfile qui défini l'image source ainsi que les variables d'environment (nom de la base de données, le user, le mot de passe), puis on build à l'aide de la commande :
```
docker build -t fcheval/database-tp1 .
```
> NB: le point à la fin de la commande signifie que l'on utilise le Dockerfile qui est là où la commande est exécutée.

Ensuite, on créé le network correspondant avec la commande :
```
docker network create app-network
```

On lance notre application en précisant bien le network :
```
docker run -d --net=app-network --name database-tp1 fcheval/database-tp1
```
> NB: Le -d permet de le run en background.

Enfin, on lance le container adminer 
```
docker run -p "8090:8080" --net=app-network --name=adminer -d adminer
```

## Backend API


## Http server


## Questions
### 1-1) Document your database container essentials: commands and Dockerfile.
> 

### 1-2) Why do we need a multistage build? And explain each step of this dockerfile.
> 

### 1-3) Document docker-compose most important commands. 1-4 Document your docker-compose file.
>

### 1-5) Document your publication commands and published images in dockerhub.
>


alexdev@MC-G7V9THG2MC java % docker run --name pgdb -p 5432:5432 -v volume:/var/lib/postgresql/data --net=app-network -d  pgdb


docker run --net=app-network -p 8090:8080 --name pgadmin -d adminer

FROM openjdk:17-jre
COPY Main.class /
RUN java Main