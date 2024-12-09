
# Day Of Chaos - Activités accélératrices de résilience
Les activités suivantes vous sont proposées en parallèle du Day Of Chaos. Elles vous permettront d'augmenter vos chances de succès au sein du Day Of Chaos, mais aussi de découvrir les actions qui peuvent être réalisées par des administrateurs système afin de rendre un système plus résilient et facile à maintenir. Nous vous conseillons de les réaliser lorsque vous avez fini de réparer une panne en attendant les autres équipes.

## A. Ajouter des paires de clés ssh 


### A1. Pourquoi le faire ?
- Faciliter un accès sécurisé sans mot de passe entre différentes machines. 
- Automatiser des tâches administratives nécessitant des connexions SSH. 
- Réduire les risques liés au partage ou à l'utilisation de mots de passe. 

Intérêt pour le jeu : 🟢🟢🟢🟢⚪ - Facilité de mise en oeuvre : 🟢🟢🟢🟢🟢

### A2. Principe de fonctionnement
L'authentification avec clé SSH repose sur la cryptographie asymétrique pour sécuriser l'accès sans utiliser de mot de passe. Voici le fonctionnement détaillé :

1. **Génération de la paire de clés :**

-   Vous générez une paire de clés (généralement stockée dans `~/.ssh` :
    -   **Clé privée** : Conservée sur votre machine, secrète. Nommée par défaut `id_rsa.pub`
    -   **Clé publique** : Transmise au serveur. Nommée par défaut `id_rsa`

2. **Ajout de la clé publique sur le serveur :**

-   La clé publique est ajoutée au fichier `~/.ssh/authorized_keys` du serveur.
-   Ce fichier contient les clés autorisées à se connecter.

3. **Processus d'authentification :**

-   Vous initiez une connexion SSH au serveur.
-   Le serveur envoie un **challenge chiffré** (une sorte de message crypté) à votre machine.
-   Votre client SSH utilise la **clé privée** pour déchiffrer ce message.
-   Si le déchiffrement est correct, le serveur accorde l'accès.

4. **Sécurisation :**

-   La clé privée ne quitte jamais votre machine.
-   Le serveur ne stocke que la clé publique, qui ne permet pas de reconstruire la clé privée.

Cette méthode élimine le besoin d'un mot de passe et renforce la sécurité en protégeant contre les attaques par interception.

### A3. Comment le faire ?
1. Générer une paire de clés SSH :

` ssh-keygen -t rsa -b 4096 -C "votre_email@example.com" `

2. Copier la clé publique vers une autre machine: 

` ssh-copy-id utilisateur@adresse_ip`



## B. Mettre en œuvre des services Linux

### B1. Pourquoi le faire ?
Mettre en œuvre un service Linux offre plusieurs avantages par rapport à un simple script shell de démarrage :

1. **Gestion avancée du cycle de vie**

-   Les services Linux, gérés par des outils comme `systemd`, permettent de **démarrer**, **arrêter**, **redémarrer** ou **recharger** les processus de manière standardisée.
-   Ils offrent une meilleure gestion des dépendances entre services (ex. : démarrer un service uniquement après un autre).

2. **Supervision et résilience**

-   Les services peuvent être configurés pour redémarrer automatiquement en cas de plantage.
-   Les outils comme `systemctl` fournissent un suivi en temps réel de l’état des services.

3. **Exécution au démarrage**

-   Les services peuvent être configurés pour s'exécuter automatiquement au démarrage du système, avec des priorités définies.

4. **Uniformité et maintenabilité**

-   Les services suivent une convention commune (`/etc/systemd/system/`), rendant leur gestion plus simple pour les administrateurs.
-   Les scripts de démarrage manuels, en revanche, peuvent être dispersés et difficiles à maintenir.

5. **Sécurité**

-   Les services Linux permettent de définir des utilisateurs spécifiques, des limites de ressources et des environnements isolés pour réduire les risques liés à des processus mal configurés.

**En résumé** :

Un service Linux est plus robuste, maintenable et sécurisé, surtout pour des processus critiques ou de longue durée, tandis qu’un script shell peut suffire pour des tâches simples et ponctuelles.

Intérêt pour le jeu : 🟢🟢🟢🟢⚪ - Facilité de mise en œuvre : 🟢🟢🟢⚪⚪


### B2. Comment le faire ?
Voici les étapes pour convertir un script shell en service géré par `systemd` :

1. **Créer le script shell**

Assurez-vous que votre script shell fonctionne correctement et est exécutable. Par exemple :

    #!/bin/bash
    echo "Mon service est démarré !" >> /var/log/mon_service.log
    # Commandes à exécuter

Enregistrez-le, par exemple, sous `/usr/local/bin/mon_script.sh` et rendez-le exécutable :

    chmod +x /usr/local/bin/mon_script.sh

2. **Créer un fichier de service systemd**

Créez un fichier de configuration dans `/etc/systemd/system/mon_service.service` :

    [Unit]
    Description=Mon Service Personnalisé
    After=network.target  # Démarre après le réseau
    
    [Service]
    ExecStart=/usr/local/bin/mon_script.sh  # Commande pour lancer le script
    Restart=always                          # Redémarre en cas d'échec
    User=mon_utilisateur                    # (Optionnel) Définit l'utilisateur qui exécute le service
    WorkingDirectory=/usr/local/bin         # (Optionnel) Répertoire de travail
    Environment="VAR1=valeur1" "VAR2=valeur2"  # (Optionnel) Variables d'environnement
    
    [Install]
    WantedBy=multi-user.target

3. **Activer et démarrer le service**

Rechargez les configurations de `systemd` :

    sudo systemctl daemon-reload

Activez le service pour qu’il démarre au démarrage du système :

    sudo systemctl enable mon_service
Démarrez le service immédiatement :

    sudo systemctl start mon_service

4. **Vérifier l’état du service**

Vérifiez si le service fonctionne correctement :

    sudo systemctl status mon_service
Consultez les logs du service :

    journalctl -u mon_service

5. **Tester le redémarrage automatique (optionnel)**

Simulez une panne en arrêtant manuellement le processus ou en utilisant `kill`. Le service doit redémarrer automatiquement si `Restart=always` est configuré.

Avec ces étapes, votre script shell est désormais géré comme un service système robuste et maintenable.


## C. Centraliser les logs

### C1. Pourquoi le faire ?
-   Simplifier l'analyse des événements en rassemblant les journaux de plusieurs machines ou applications.
-   Détecter et résoudre rapidement des anomalies grâce à une vision consolidée.
-   Respecter les bonnes pratiques de gestion et de conformité en matière de logs.

Intérêt pour le jeu : 🟢🟢🟢🟢🟢 - Facilité de mise en oeuvre : 🟢⚪⚪⚪⚪

### C2. Comment le faire ?
Une solution simple et moderne pour centraliser les logs d'un système consiste à utiliser un système d'agrégation et d'interrogation de logs tel que **Grafana Loki** combiné à un agent de collecte et d'envoi de logs tel que **Promtail**. 

#### C2.1. Installation de Grafana Loki
Voici un fichier `docker-compose.yml` pour installer **Grafana** et **Loki** avec **Docker Compose** :

```yaml
version: "3.8"

services:
  loki:
    image: grafana/loki:latest
    container_name: loki
    ports:
      - "3100:3100"
    volumes:
      - ./loki-config.yaml:/etc/loki/local-config.yaml
    command: -config.file=/etc/loki/local-config.yaml

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
    depends_on:
      - loki

volumes:
  loki_data:

```


1. **Créer la configuration pour Loki**

Créez un fichier nommé `loki-config.yaml` dans le même dossier que le fichier `docker-compose.yml` :

```yaml
auth_enabled: false

server:
  http_listen_port: 3100
  grpc_listen_port: 9096

common:
  instance_addr: 127.0.0.1
  path_prefix: /tmp/loki
  storage:
    filesystem:
      chunks_directory: /tmp/loki/chunks
      rules_directory: /tmp/loki/rules
  replication_factor: 1
  ring:
    kvstore:
      store: inmemory

query_range:
  results_cache:
    cache:
      embedded_cache:
        enabled: true
        max_size_mb: 100

schema_config:
  configs:
    - from: 2020-10-24
      store: tsdb
      object_store: filesystem
      schema: v13
      index:
        prefix: index_
        period: 24h

ruler:
  alertmanager_url: http://localhost:9093
  reporting_enabled: false
```
Plus d'information sur la configuration de promtail: [https://grafana.com/docs/loki/latest/send-data/promtail/configuration/](https://grafana.com/docs/loki/latest/send-data/promtail/configuration/)

2. **Démarrer les conteneurs**

Lancez Docker Compose :

```bash
docker-compose up -d
```


3. **Accéder à Grafana**

-   Accédez à Grafana à l’adresse : `http://localhost:3000`.
-   Connectez-vous avec les identifiants par défaut (`admin` / `admin`) ou ceux définis dans le fichier Compose.

4. **Configurer Loki comme source de données dans Grafana**

	1.  Dans Grafana, allez dans **Configuration > Data Sources**.
	2.  Ajoutez une nouvelle source de données avec le type **Loki**.
	3.  Configurez l'URL comme `http://loki:3100`.
	4.  Enregistrez et testez la connexion.

5. **Visualiser les logs**

-   Dans Grafana, utilisez **Explore** pour rechercher les logs.
-   Configurez vos applications ou services pour envoyer des logs à Loki via des agents comme  **Promtail**.

#### C2.2. Installation de Promtail
Promtail est un agent léger et simple qui lit les fichiers journaux (même en texte brut) et les envoie directement à Loki. Il est facile à configurer et ne nécessite aucune modification des logs existants.

Voici comment installer et configurer **Promtail** sur un serveur en utilisant **Docker** pour envoyer des logs à Loki.

1. **Créer un fichier de configuration Promtail**

Créez un fichier de configuration `promtail-config.yaml` sur votre machine, par exemple dans `/etc/promtail/config.yml`. Ce fichier définit comment Promtail collectera et enverra les logs à Loki.

Exemple de configuration :

```yaml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

clients:
  - url: http://<adresse_du_serveur_loki>:3100/loki/api/v1/push

positions:
  filename: /tmp/positions.yaml

scrape_configs:
  - job_name: "system-logs"
    static_configs:
      - targets:
          - localhost
        labels:
          job: varlogs
          host: my-machine
          __path__: /var/log/*.log

```

-   **`<adresse_du_serveur_loki>`** : Remplacez par l’adresse IP ou le domaine de votre serveur Loki.
-   **`__path__`** : Spécifie les fichiers logs à lire (ici tous les logs dans `/var/log/`).
-   **Labels** : Vous pouvez ajouter des labels comme `host` pour identifier la machine.



2. **Lancer Promtail avec Docker**

Dans le même répertoire que votre fichier de configuration `promtail-config.yaml`, exécutez la commande suivante pour démarrer Promtail dans un conteneur Docker.

```bash
docker run -d \
  --name promtail \
  --restart unless-stopped \
  -v /etc/promtail/config.yml:/etc/promtail/config.yml \
  -v /var/log:/var/log \
  -v /tmp:/tmp \
  grafana/promtail:latest \
  -config.file=/etc/promtail/config.yml

```

-   **`-v /etc/promtail/config.yml:/etc/promtail/config.yml`** : Montez le fichier de configuration Promtail dans le conteneur.
-   **`-v /var/log:/var/log`** : Montez les logs du système pour que Promtail puisse les lire.
-   **`-v /tmp:/tmp`** : Montez `/tmp` pour stocker les fichiers de positions.



3. **Vérification et suivi**

Pour vérifier que Promtail fonctionne correctement, vous pouvez consulter les logs du conteneur Docker :

```bash
docker logs -f promtail
```

Assurez-vous que Promtail envoie les logs au serveur Loki sans erreur.



4. **Accéder à Grafana**

Accédez à Grafana (exemple : `http://<adresse_de_votre_serveur>:3000`), et configurez **Loki** comme source de données :

 -  Allez dans **Configuration > Data Sources**.
 -  Sélectionnez **Loki** comme type de source de données.
 -  Mettez l'URL de votre serveur Loki (exemple : `http://<adresse_du_serveur_loki>:3100`).


5.  **Visualisation des logs dans Grafana**

-   Dans Grafana, allez dans **Explore**.
-   Sélectionnez votre source de données Loki et explorez les logs envoyés depuis Promtail.

