**TP part 01 - Docker**

**1-1 For which reason is it better to run the container with a flag -e to give the environment variables rather than put them directly in the Dockerfile?**

L’utilisation du -e permet de rendre le mot de passe invisible dans le Dockerfile, ce qui évite de l’enregistrer directement dans l’image Docker. C’est donc plus sécurisé, plus flexible, et cela protège les informations sensibles.


**1-2 Why do we need a volume to be attached to our postgres container?**

On attache un volume au conteneur Postgres pour garder les données même si le conteneur est supprimé. Sans volume, toutes les informations seraient perdues à chaque redémarrage.


**1-3 Document your database container essentials: commands and Dockerfile.**

- Construire ton image de base de données: docker build -t lucienorecaumy-database:latest .
- Créer un réseau Docker :docker network create app-network
- Exécuter PostgreSQL :docker run -d \
  --name mon-postgres \
  --network app-network \
  -e POSTGRES_DB=db \
  -e POSTGRES_USER=use \
  -e POSTGRES_PASSWORD=pwd \
  -v pgdata:/var/lib/postgresql/data \
  lucienorecaumy-database:latest
- Démarrer l’interface d’administration :docker run -d --name adminer \
  --network app-network \
  -p 8080:8080 \
  adminer


**1-4 Why do we need a multistage build? And explain each step of this dockerfile.**

Une construction en plusieurs étapes (multistage build) permet de séparer la phase de compilation de la phase d’exécution, afin de créer une image finale plus légère et plus rapide.
- Construction du projet :FROM eclipse-temurin:21-jdk-alpine AS myapp-build
- Utilise une image Java avec Maven pour compiler le projet :ENV MYAPP_HOME=/opt/myapp
WORKDIR $MYAPP_HOME
- Définit le dossier de travail :RUN apk add --no-cache maven
- Installe Maven pour pouvoir construire le projet Java :COPY pom.xml .
COPY src ./src
RUN mvn package -DskipTests
- Exécution de l’application :FROM eclipse-temurin:21-jre-alpine
- Utilise une image Java plus légère juste pour exécuter l’app :ENV MYAPP_HOME=/opt/myapp
WORKDIR $MYAPP_HOME
- Même dossier de travail que l’étape précédente :COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar
- Copie le fichier .jar compilé depuis l’étape 1 :ENTRYPOINT ["java", "-jar", "myapp.jar"]


**1-5 Why do we need a reverse proxy?**

Un proxy inverse sert de point d’entrée unique entre les clients et les serveurs internes. Il protège les serveurs backend, répartit le trafic, centralise l’accès, améliore les performances et assure la fiabilité du système.


**1-6 Why is docker-compose so important?**

Docker Compose permet de définir et lancer plusieurs conteneurs en une seule commande, ce qui simplifie la gestion et le déploiement d’applications complexes


**1-7 Document docker-compose most important commands.**

- docker-compose up :Démarre tous les conteneurs définis dans le fichier docker-compose.yml
- docker-compose up -d :Démarre les conteneurs en mode détaché
- docker-compose down :Arrête et supprime tous les conteneurs, le réseau et les volumes créés
- docker-compose logs :Affiche les messages de chaque service pour suivre ce qu’il se passe
- docker-compose up --build :Reconstruit les images avant de lancer les services


**1-8 Document your docker-compose file.**

- backend :Construit l’image à partir du dossier ./simpleapi, utilise les variables de .env, dépend de la base de données, et est connecté au réseau app-network
- database :Construit l’image depuis ./database, utilise aussi .env, et est connecté aux réseaux app-network et proxy-network pour communiquer avec les autres services
- httpd :Construit l’image depuis ./serveur, expose le port 80 pour l’accès web, dépend du backend, et est connecté au réseau proxy-network
- app-network :Sert à connecter le backend et la base de données ensemble
- proxy-network :Permet la communication entre la base de données et le serveur HTTP (httpd)
- db-data :Volume prévu pour stocker les données de la base de données de façon persistante, même si le conteneur est supprimé


**1-9 Document your publication commands and published images in dockerhub.**
- Se connecter à Docker Hub :docker login
- Prépare l’image pour qu’elle soit associée au bon compte Docker Hub :docker tag lucienorecaumy-backend lucienorecaumy/lucienorecaumy-backend
- Pousser l’image vers Docker Hub :docker push lucienorecaumy/lucienorecaumy-backend


**1-10 Why do we put our images into an online repo?**
On met nos images Docker dans un dépôt en ligne sur Docker Hub pour pouvoir les partager, réutiliser et déployer facilement depuis n’importe où, tout en centralisant et versionnant les images pour le projet.


**TP part 02 - Github Actions**
1-1 For which reason is it better to run the container with a flag -e to give the environment variables rather than put them directly in the Dockerfile?
1-2 Why do we need a volume to be attached to our postgres container?
1-3 Document your database container essentials: commands and Dockerfile.
1-4 Why do we need a multistage build? And explain each step of this dockerfile.
1-5 Why do we need a reverse proxy?
1-6 Why is docker-compose so important?
1-7 Document docker-compose most important commands.
1-8 Document your docker-compose file.
1-9 Document your publication commands and published images in dockerhub.
1-10 Why do we put our images into an online repo?

**TP part 03 - Ansible**
1-1 For which reason is it better to run the container with a flag -e to give the environment variables rather than put them directly in the Dockerfile?
1-2 Why do we need a volume to be attached to our postgres container?
1-3 Document your database container essentials: commands and Dockerfile.
1-4 Why do we need a multistage build? And explain each step of this dockerfile.
1-5 Why do we need a reverse proxy?
1-6 Why is docker-compose so important?
1-7 Document docker-compose most important commands.
1-8 Document your docker-compose file.
1-9 Document your publication commands and published images in dockerhub.
1-10 Why do we put our images into an online repo?
