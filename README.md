# Financial_Transaction_Application-Docker_project

J'ai decomposé ma méthodogie en 5 étapes. Quelques etapes sont accompagnés d'un ou plusieurs screenshots. Cela sera notifié par : Voir 'image n°_de_l'image.
Les screenshots sont disponibles dans le dossier "screenshot" du projet.

----------------------------------------------------------------------------------------------------------

Le projet repose sur l'architecture suivante :

- **Spring Boot App** (PayMyBuddy) : application Java backend.
- **MySQL** : base de données relationnelle.
- **Docker** : conteneurisation des services.
- **Docker Compose** : orchestration multi-conteneurs.
- **Docker Registry** : registre privé pour héberger les images Docker.
- **Docker Registry UI** : interface web pour consulter les images du registre.
- **Réseau Docker personnalisé** : permet la communication entre les conteneurs du projet.

Les services sont déployés ensemble grâce à Docker Compose pour permettre un environnement de développement ou de production isolé, reproductible et portable.

1- Lancer "git clone" du répository https://github.com/eazytraining/bootcamp/project-update.git et aller dans le mini-projet docker.

2- Créer un fichier .env contenant les variables d'environnement (comme les mots de passe) :
MYSQL_USER=paymybuddy_user
MYSQL_PASSWORD=supermotdepasse
MYSQL_DATABASE=paymybuddy
SPRING_DATASOURCE_URL=jdbc:mysql://paymybuddy-db:3306/paymybuddy
MYSQL_ROOT_PASSWORD=rootpassword

3- Créer le Dockerfile pour builder l’application:
FROM amazoncorretto:17-alpine
LABEL maintainer="Maxime Lanca"
VOLUME /data
COPY target/paymybuddy.jar /paymybuddy.jar
EXPOSE 8080
CMD ["java", "-jar", "/paymybuddy.jar"]

4- Créer le fichier docker-compose.yml:
version: '3.8'
services:
  paymybuddy-backend:
    build: .
    ports:
      - "8080:8080"
    environment:
      - SPRING_DATASOURCE_URL=${SPRING_DATASOURCE_URL}
      - SPRING_DATASOURCE_USERNAME=${MYSQL_USER}
      - SPRING_DATASOURCE_PASSWORD=${MYSQL_PASSWORD}
    depends_on:
      - paymybuddy-db

  paymybuddy-db:
    image: mysql:8.0
    environment:
	- MYSQL_ROOT_PASSWORD=${MYSQL_ROOR_PASSWORD}
	- MYSQL_DATABASE=${MYSQL_DATABASE}
    	- MYSQL_USER=${MYSQL_USER}
    	- MYSQL_PASSWORD=${MYSQL_PASSWORD}
    volumes:
      - db_data:/var/lib/mysql
      - ./initdb:/docker-entrypoint-initdb.d
    ports:
      - "3306:3306"

volumes:
  db_data:

5- Depuis le dossier contenant le docker-compose.yml, on execute : docker-compose up -d. Cela va construire les images et lancer les conteneurs.

6- Vérifier que tout fonctionne avec : docker ps et docker images

7- Créer un réseau Docker pour connecter les services: docker network create paymybuddy-network

8- Lancer un registre privé: docker run -d -p 5000:5000 --net paymybuddy-network --name paymybuddy-registry -e REGISTRY_STORAGE_DELETE_ENABLED=true registry:2

9- Lancer l'interface web du registre: docker run -d -p 8081:80 --net paymybuddy-network --name paymybuddy-frontend \
  -e REGISTRY_URL=http://paymybuddy-registry:5000 \
  -e REGISTRY_TITLE=PayMyBuddyRegistry \
  joxit/docker-registry-ui:1.5-static

10- En lancant la commande "docker push"

11- Vérifier que l'image à bien été poussé: curl -X GET http://localhost:5000/v2/_catalog


J'ai malheuresement un message d'erreur dans le registre privé qui m'empeche de montrer les images chargés. Message d'erreur :__{"errors":[{"code":"PAGINATION_NUMBER_INVALID","message":"invalid number of results requested","detail":{"n":100000}}]}__.
