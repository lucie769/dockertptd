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

**2-1 What are testcontainers?**

Les Testcontainers sont des bibliothèques Java qui lancent de vrais conteneurs Docker pendant les tests, permettant de simuler des dépendances comme une base de données sans les installer localement.


**2-2 For what purpose do we need to use secured variables ?**

Les variables sécurisées protègent les informations sensibles (mots de passe, jetons) en les cachant dans le code, ce qui garantit qu’elles ne sont jamais publiées accidentellement tout en permettant l’automatisation avec GitHub Actions.


**2-3 Why did we put needs: build-and-test-backend on this job? Maybe try without this and you will see!**

On ajoute needs: build-and-test-backend pour que ce job ne démarre que si les tests du backend ont réussi, afin d’éviter de construire une image Docker avec un code cassé qui casserait le déploiement.



**2-4 For what purpose do we need to push docker images?**

Nous publions nos images Docker pour que d’autres machines puissent récupérer la même version de l’application et la déployer facilement et de façon fiable sans reconstruire le projet.




**TP part 03 - Ansible**

**3-1 Document your inventory and base commands**

- ansible_user : nom d’utilisateur utilisé pour se connecter au serveur
- ansible_ssh_private_key_file : chemin vers ta clé privée SSH pour l’authentification
- prod : groupe d’hôtes contenant ton serveur distant

- ansible all -i inventories/setup.yml -m ping :Vérifie la connectivité avec le serveur distant
- ansible all -i inventories/setup.yml -m setup :Récupère les faits système
- ansible all -i inventories/setup.yml -m apt -a "name=apache2 state=absent" --become :upprime Apache2 du serveur en forçant son état à "absent"

  
**3-2 Document your playbook**

Le playbook Ansible installe Docker sur un serveur distant, configure les dépendances nécessaires et prépare un environnement Python pour utiliser le SDK Docker, sur tous les hôtes de l’inventaire.
-Installer les paquets nécessaires :- name: Install required packages
      apt:
        name:
          - apt-transport-https
          - ca-certificates
          - curl
          - gnupg
          - lsb-release
          - python3-venv
        state: latest
        update_cache: yes
 - Ajouter la clé GPG officielle de Docker : name: Add Docker GPG key
      apt_key:
        url: https://download.docker.com/linux/debian/gpg
        state: present
- Ajouter le dépôt Docker :name: Add Docker APT repository
      apt_repository:
        repo: "deb [arch=amd64] https://download.docker.com/linux/debian {{ ansible_facts['distribution_release'] }} stable"
        state: present
        update_cache: yes
- Installer Docker :name: Install Docker
      apt:
        name: docker-ce
        state: present
- Installer Python3 et pip3 :name: Install Python3 and pip3
      apt:
        name:
          - python3
          - python3-pip
        state: present
- Créer un environnement virtuel pour le SDK Docker :name: Create a virtual environment for Docker SDK
      command: python3 -m venv /opt/docker_venv
      args:
        creates: /opt/docker_venv
- Installer le SDK Docker dans l’environnement virtuel :name: Install Docker SDK for Python in virtual environment
      command: /opt/docker_venv/bin/pip install docker
- ’assurer que Docker est en cours d’exécution :name: Make sure Docker is running
      service:
        name: docker
        state: started
      tags: docker
  

**3-3 Document your docker_container tasks configuration.**

Ansible permet de déployer des conteneurs Docker sur un serveur distant avec le module docker_container. Chaque tâche définit le nom du conteneur, l’image, les ports, les variables d’environnement, les réseaux Docker et les dépendances éventuelles.

- Création du réseau Docker : création du réseau app-network pour permettre aux conteneurs (base de données, backend, proxy) de communiquer de manière sécurisée et isolée. L’interpréteur Python est précisé pour assurer la compatibilité avec le module Docker d’Ansible.
- Installation de Docker avec Ansible : installation de Docker sur le serveur distant selon les recommandations officielles Debian, ajout de la clé GPG et du dépôt Docker, installation des paquets nécessaires et configuration d’un environnement Python pour le SDK Docker. Le service Docker est vérifié pour garantir une installation propre, reproductible et prête au déploiement.
- Installation simplifiée de Docker : installation rapide via le paquet docker.io des dépôts officiels Debian/Ubuntu, en mettant à jour le cache APT pour utiliser la dernière version. Moins personnalisable que l’installation via le dépôt officiel Docker.
- Déploiement du backend : lancement du conteneur luciemoreau/my-backend:latest connecté au réseau app-network pour communiquer avec la base de données et le proxy HTTP.
- Déploiement de la base de données : lancement du conteneur PostgreSQL luciemoreau/my-database:latest connecté à app-network, avec injection dynamique des variables d’environnement (nom de la base, utilisateur, mot de passe).
- Déploiement du proxy HTTP : lancement du conteneur luciemoreau/my-httpd:latest, connecté à app-network et exposant le port 80 comme point d’entrée vers les services internes.


**3-4 Is it really safe to deploy automatically every new image on the hub ? explain. What can I do to make it more secure?**

Le déploiement automatique des images sur le registre peut être risqué si elles n’ont pas été testées ou validées. Pour sécuriser ce processus, il faut ajouter des tests, limiter les déclenchements aux branches fiables et utiliser des images sûres et bien identifiées.
