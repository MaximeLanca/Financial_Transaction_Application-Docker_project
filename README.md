# Financial_Transaction_Application-Docker_project

J'ai decomposé ma méthodogie en 5 étapes. Quelques etapes sont accompagnés d'un ou plusieurs screenshots. Cela sera notifié par : Voir 'image n°_de_l'image.
Les screenshots sont disponibles dans le dossier "screenshot" du projet.

----------------------------------------------------------------------------------------------------------


Etape-1 : Création des fichier Dockerfile et docker-compose


Aprés d'avoir lancer "__git clone__" du répository https://github.com/eazytraining/bootcamp-project-update.git, il faut completer les fichers Dockerfile et docker-compose du projet. Voir image 1, 2, 3 et 4

Dockerfile: 
- Nous utlisons une image basée sur Alpine Linux. 
- Le label: moi même. 
- je déclare un volume Docker /data. Cela signifie que les fichiers écrits dans /data seront persistants.
- La commande "COPY target/paymybuddy.jar /paymybuddy.jar" copie le fichier paymybuddy.jar depuis le dossier target de la machine vers l’image Docker.
- la commande "EXPOSE 8080" indique que le conteneur écoute sur le port 8080.

Docker-compose.yml:
- la ligne " build: ." signifie que l’image sera construite à partir du Dockerfile dans le répertoire courant.
- la ligne " ports: - "8080:8080" signifie que le conteneur écoute sur le port 8080 de la machine hôte et le port 8080 du conteneur.
- Pour les variables d'environnement, SPRING_DATASOURCE_URL=jdbc:mysql://paymybuddy-db:3306/paymybuddy : paymybuddy-db est le nom du service MySQL, utilisé comme nom d'hôte (Docker gère la résolution DNS interne), 3306 est le port MySQL,SPRING_DATASOURCE_USERNAME=root et SPRING_DATASOURCE_PASSWORD=rootpassword permettent d’accéder à MySQL en tant que root.
- La ligne "depends_on: paymybuddy-db" indique que le backend ne doit démarrer qu’après paymybuddy-db.
- La ligne "MYSQL_ROOT_PASSWORD=rootpassword" définit le mot de passe pour l’utilisateur root de MySQL.
- La ligne "MYSQL_DATABASE=paymybuddy" permet de créer automatiquement une base de données nommée paymybuddy.
- La ligne "db_data:/var/lib/mysql" permet de monter un volume nommé db_data pour stocker les données MySQL.
- La ligne "./initdb:/docker-entrypoint-initdb.d"permet d’exécuter automatiquement des scripts SQL lors du premier démarrage.


Etape-2 : Création des images et des conteneurs


Après d'avoir creer les fichiers de l'etape 1, nous pouvons creer les images et les conteneurs de nos applications en lancant la commande "docker-compose up -d". Voir image 5 et 6.
Des vérifications peuvent etre faite en lancant "__docker ps__" et "__docker images__". Voir image 7.


Etape-3 : Création d'un réseau pour le repository.


On lance "__Docker network create paymebuddy-network__" pour creer un réseau qui servira à stocker nos images plus tard. Voir image 8.


Etape-4 : Création du registre privé et de son backend


- On lance "__docker run -d -p 5000:5000 -e REGISTRY_STORAGE_DELETE_ENABLED=true —net paymybuddy-network —name paymybuddy-registry registry:2__" pour creer notrs registre privé "paymybuddy-registry". la variable REGISTRY_STORAGE_DELETE_ENABLED égale à "true" nous permettra de supprimer les images. Voir image 9.
- On lance ensuite "__docker run -d -p 8080:80 —net paymybuddy-network  -e REGISTRY_URL=http://paymybuddy-registry:5000 -e REGISTRY_TITLE=registre_projet —name paymybuddy-frontend joxit/docker-registry-ui:1.5-stati__c". Cette commande permet de rajouter le backend du régistre privé sur le port 8080. Voir image 10, 11 et 12.


Etape-5 : Tag et envoi des images

  
Taguer une image Docker permet de l'identifier de manière unique et d'indiquer où et comment elle sera stockée. Nous mettons donc de nouveau tag pour bien les identifier. 
En lancant la commande "docker push", nous envoyons donc les images dnas le registre privée.Voir image 13 et 14
J'ai malheuresement un message d'erreur dans le registre privé qui m'empeche de montrer les images chargés. Message d'erreur :__{"errors":[{"code":"PAGINATION_NUMBER_INVALID","message":"invalid number of results requested","detail":{"n":100000}}]}__
Mais avec la commande "curl -X GET http://localhost:5000/v2/_catalog" on peut voir que les images ont bien été poussées. Voir image 14.
