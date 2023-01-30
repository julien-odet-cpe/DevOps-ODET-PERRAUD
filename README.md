# On se donne les permissions requises
newgrp docker

# On teste la bonne installation de Docker
docker run hello-world

# On télécharge la dernière image Alpine depuis le repository Docker
docker pull alpine

# Docker images

alexdev@MC-G7V9THG2MC java % docker run --name pgdb -p 5432:5432 -v volume:/var/lib/postgresql/data --net=app-network -d  pgdb


docker run --net=app-network -p 8090:8080 --name pgadmin -d adminer

FROM openjdk:17-jre
COPY Main.class /
RUN java Main