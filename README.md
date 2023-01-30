# TP DevOps - Compte rendu

Filière IRC - 4ème année - Promotion 2024

## Auteur :
- Julien ODET
- Alex PERRAUD

---

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