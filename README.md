# DEVOPS TP3

**Questions**

> Documentez votre inventaire et vos commandes de base

**Commandes de base d'Ansible**

* `ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"`: Exécute une tâche Ansible pour collecter des informations sur les hôtes définis dans le fichier de configuration inventories/setup.yml
* `ansible all` signifie qu'Ansible devra exécuter la tâche sur tous les hôtes définis dans le fichier de configuration.
* `-i inventories/setup.yml` indique à Ansible le chemin vers le fichier de configuration qui définit les hôtes cibles.
* `-m` setup indique à Ansible d'utiliser le module setup pour collecter les informations système.
* `-a "filter=ansible_distribution*"` est un argument supplémentaire pour le module setup qui permet de filtrer les informations collectées pour n'afficher que les informations relatives à la distribution d'OS (ansible_distribution).
* `-a "name=httpd state=absent"` est un argument supplémentaire pour le module yum qui définit le nom du paquet à désinstaller (httpd) et son état final désiré (absent).
* `--become` indique à Ansible de s'exécuter en tant qu'utilisateur root ou avec les privilèges nécessaires pour effectuer des modifications système.
* `ansible-inventory` : Affiche l'inventaire d'Ansible, qui décrit les hôtes et les groupes d'hôtes cibles pour les tâches d'Ansible.
* `ansible all -m ping` : Vérifie la connectivité avec tous les hôtes définis dans l'inventaire d'Ansible.
* `ansible-playbook` : Exécute un playbook Ansible, qui est un ensemble de tâches à exécuter sur les hôtes cibles.
* `ansible-galaxy` : Gère les dépendances d'Ansible sous forme de rôles.
* `ansible-vault` : Chiffre et déchiffre des fichiers sensibles, tels que les mots de passe, pour les utiliser dans des playbooks Ansible.
* `ansible-config` : Gère les paramètres de configuration d'Ansible.

**Documentation de l'inventaire**

```yaml
# Fichier de configuration Ansible

# Définition des variables pour la connexion SSH
vars:
  ansible_user: centos
  ansible_ssh_private_key_file: ../../id_rsa

# Définition d'un enfant de type "prod"
children:
  prod:
    # Définition de l'hôte cible pour les tâches d'Ansible
    hosts: morgane.quelennec.takima.cloud
```

> Documentez votre playbook et la configuration de vos tâches docker_container

```yaml
# Ce code est un playbook Ansible qui configure les hôtes pour exécuter une application web.

- hosts: all
  # Cette ligne définit la cible des tâches à venir. "all" signifie que tous les hôtes du inventaire seront utilisés.
  
  gather_facts: false
  # Désactive la collecte de données à partir des hôtes. Cela peut accélérer le processus d'exécution du playbook.
  
  become: yes
  # Exécuter les tâches suivantes avec les privilèges nécessaires pour effectuer des modifications sur les systèmes.

# Install Docker
  tasks:
  # Les tâches à exécuter sur les hôtes.
  
  - name: Clean packages
    # Nom de la tâche pour identifier facilement ce qu'elle fait.
    
    command:
      cmd: dnf clean -y packages
    # Exécute la commande `dnf clean -y packages` sur les hôtes cibles.
    
  - name: Install device-mapper-persistent-data
    # Nom de la tâche pour identifier facilement ce qu'elle fait.
    
    dnf:
      name: device-mapper-persistent-data
      state: latest
    # Installe le paquet "device-mapper-persistent-data" sur les hôtes cibles. Le paramètre "state" définit que la dernière version disponible sera installée.
    
  - name: Install lvm2
    # Nom de la tâche pour identifier facilement ce qu'elle fait.
    
    dnf:
      name: lvm2
      state: latest
    # Installe le paquet "lvm2" sur les hôtes cibles. Le paramètre "state" définit que la dernière version disponible sera installée.
    
  - name: add repo docker
    # Nom de la tâche pour identifier facilement ce qu'elle fait.
    
    command:
      cmd: sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
    # Ajout de le dépôt Docker à la liste des sources disponibles pour les hôtes cibles.
    
  - name: Install Docker
    # Nom de la tâche pour identifier facilement ce qu'elle fait.
    
    dnf:
      name: docker-ce
      state: present
    # Installe le paquet Docker sur les hôtes cibles. Le paramètre "state" définit que le paquet doit être installé.
    
  - name: install python3
    # Nom de la tâche pour identifier facilement ce qu'elle fait.
    
    dnf:
      name: python3
    # Installe Python 3 sur les hôtes cibles.
    
  - name: Pip install
    # Nom de la tâche pour identifier facilement ce qu'elle fait.
      name: docker

# Ensure Docker service is running
  - name: Make sure Docker is running
    service: name=docker state=started
    tags: docker

# Create Docker network for the containers
  - name: Create Docker network
    docker_network:
        name: app-network
        driver: bridge
        state: present

# Start the BDD container using the specified Docker image
  - name: Start BDD
    docker_container:
        name: bdd_pg
        image: mquelennec/web_db:1.0
        env:
          POSTGRES_DB: db
          POSTGRES_USER: usr
          POSTGRES_PASSWORD: pwd
        ports:
        - 5432:5432
        networks:
        - name: app-network
        state: started
        volumes:
        - /my/own/datadir:/var/lib/postgresql/data

# Start the backend container using the specified Docker image
  - name: Start backend
    docker_container:
        name: my-java-api
        image: mquelennec/web_backend:1.0
        ports:
        - 8080:8080
        networks:
        - name: app-network
        state: started

# Start the http_front container using the specified Docker image
  - name: http_front run
    docker_container:
        name: http_front
        image: mquelennec/docker-https
        ports:
        - 80:80
        networks:
        - name: app-network
        state: started
        links:
        - my-java-api:my-java-api

# Start the devops-front container using the specified Docker image
  - name: devops-front run
    docker_container:
        name: devops-front
        image: mquelennec/docker-https
        ports:
        - 82:82
        networks:
        - name: app-network
        state: started
        links:
        - http_front:http_front
```