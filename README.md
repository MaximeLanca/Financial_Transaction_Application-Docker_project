# Financial_Transaction_Application-Docker_project

Les étapes sont accompagnées d'un ou plusieurs screenshots. Cela sera notifié par : Voir 'image n°_de_l'image.
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

Copier le lien HTTPS du répository https://github.com/MaximeLanca/Financial_Transaction_Application-Docker_project.git  aller dans le mini-projet docker.
 ![01](./screenshot/01.png)

Aller sur le repertoire suivant: /root/Financial_Transaction_Application-Docker_project.
 ![02](./screenshot/02.png)

Ce répertoire comporte trois fichiers importants pour la containerisation:
-le fichier .env
-le fichier Dockerfile
-le fichier docker-compose.yml
 ![03](./screenshot/03.png)
 ![04](./screenshot/04.png)

J'ai crée Le fichier .env pour contenir les variables d'environnement suivant: 
MYSQL_USER=paymybuddy_user \
MYSQL_PASSWORD=supermotdepasse \
MYSQL_DATABASE=paymybuddy \
SPRING_DATASOURCE_URL=jdbc:mysql://paymybuddy-db:3306/paymybuddy \
MYSQL_ROOT_PASSWORD=rootpassword
 ![05](./screenshot/05.png)
 ![06](./screenshot/06.png)

J'ai completé le Dockerfile pour builder l’application de cette façon:
FROM amazoncorretto:17-alpine \
LABEL maintainer="Maxime Lanca" \
VOLUME /data \
COPY target/paymybuddy.jar /paymybuddy.jar \
EXPOSE 8080 \
CMD ["java", "-jar", "/paymybuddy.jar"] \
 ![07](./screenshot/07.png)
 ![08](./screenshot/08.png)

J'ai completé le fichier docker-compose.yml avec les élements suivant::
version: '3.8' \
services: \
  paymybuddy-backend: \
    build: . \
    ports: \
      - "8080:8080" \
    environment: \
      - SPRING_DATASOURCE_URL=${SPRING_DATASOURCE_URL} \
      - SPRING_DATASOURCE_USERNAME=${MYSQL_USER} \
      - SPRING_DATASOURCE_PASSWORD=${MYSQL_PASSWORD} \
    depends_on: \
      - paymybuddy-db 

  paymybuddy-db: \
    image: mysql:8.0 \
    environment: \
	- MYSQL_ROOT_PASSWORD=${MYSQL_ROOR_PASSWORD} \
	- MYSQL_DATABASE=${MYSQL_DATABASE} \
    	- MYSQL_USER=${MYSQL_USER} \
    	- MYSQL_PASSWORD=${MYSQL_PASSWORD} \
    volumes: \
      - db_data:/var/lib/mysql \
      - ./initdb:/docker-entrypoint-initdb.d \
    ports: \
      - "3306:3306" 

volumes: \
  db_data: 
 ![09](./screenshot/09.png)
 ![10](./screenshot/10.png)
 ![11](./screenshot/11.png)

Depuis le dossier contenant le docker-compose.yml, on execute : docker-compose up -d. Cela va construire les images et lancer les conteneurs.
 ![12](./screenshot/12.png)

Vérifier que tout fonctionne avec : docker images.
 ![13](./screenshot/13.png)

Créer un réseau Docker pour connecter les services: docker network create paymybuddy-network et faire lancer docker network ls pour vérifier la création du réseau.
 ![14](./screenshot/14.png)

Lancer un registre privé: docker run -d -p 5000:5000 --net paymybuddy-network --name paymybuddy-registry -e REGISTRY_STORAGE_DELETE_ENABLED=true registry:2.
 ![15](./screenshot/15.png)

Lancer l'interface web du registre: docker run -d -p 8081:80 --net paymybuddy-network --name paymybuddy-frontend \
  -e REGISTRY_URL=http://paymybuddy-registry:5000 \
  -e REGISTRY_TITLE=PayMyBuddyRegistry \
  joxit/docker-registry-ui:1.5-static
  ![16](./screenshot/16.png)

Vérification des conteneurs et des images avec docker images et docker ps.
 ![17](./screenshot/17.png)

Taguer les images "mini-projet-docker-paymybuddy-backend" et "mysql" par "localhost:5000/paymybuddy-backend:local" et "localhost:5000/paymybuddy-db:local" avec docker tag.
 ![18](./screenshot/18.png)

On envoie les images dans le repository en lancant docker push sur les deux images taguées.
 ![19](./screenshot/19.png)

J'ai toujours un problème avec le registre privé qui m'empeche de montrer les images chargés en allant dans le port dédié.
 ![20](./screenshot/20.png)
 ![21](./screenshot/21.png)

14- On peut quand meme vérifier la validation des envoies dans le repository avec la commande "curl -X GET http://localhost:5000/v2/_catalog".
 ![22](./screenshot/22.png)

