### Installation et configuration du serveur Minecraft

#### 1. Installation de Java
Le serveur Minecraft fonctionne avec Java, donc Java doit être installé. Installez Java d'abord.
```bash
sudo dnf install java-17-openjdk
```

Java 17 ou une version ultérieure est nécessaire, vérifiez et installez :
```bash
java -version
```

#### 2. Téléchargement du serveur Minecraft
Téléchargez le fichier JAR de la dernière version du serveur Minecraft.
```bash
wget https://launcher.mojang.com/v1/objects/1e563f68a3d88d602163ef342e5e9889fc2059bb/server.jar -O minecraft_server.jar
```

#### 3. Préparation du fichier d'exécution du serveur
Pour exécuter le serveur, créez un script start.sh pour faciliter le démarrage du serveur.
```bash
nano start.sh
```

Entrez le contenu suivant dans le fichier start.sh :
```bash
#!/bin/bash
java -Xmx2G -Xms1G -jar minecraft_server.jar nogui
```

#### 4. Attribution des droits d'exécution
Donnez les droits d'exécution au fichier start.sh.
```bash
chmod +x start.sh
```

#### 5. Démarrage du serveur
Pour démarrer le serveur, exécutez le fichier start.sh.
```bash
./start.sh
```

Lors du premier démarrage, un message demandant de créer le fichier eula.txt apparaîtra. Ouvrez le fichier eula.txt et changez l'option EULA en true.
```bash
nano eula.txt
```

Ouvrez le fichier et changez eula=false en eula=true, puis enregistrez.

#### 6. Démarrage du serveur
Vous pouvez maintenant redémarrer le script start.sh pour démarrer le serveur :
```bash
./start.sh
```

Une fois le serveur démarré correctement, vous pouvez vous connecter au serveur via le client Minecraft.

#### 7. Ouverture du port (configuration du pare-feu)
Pour permettre les connexions externes au serveur, ouvrez le port utilisé par Minecraft, par défaut 25565. Ouvrez ce port dans le pare-feu.
```bash
sudo firewall-cmd --zone=public --add-port=25565/tcp --permanent
sudo firewall-cmd --reload
```

#### 8. Configuration du serveur
Après le démarrage du serveur, vous pouvez ajuster plusieurs paramètres dans le fichier server.properties. Ouvrez ce fichier pour modifier l'IP, le port, le mode du serveur, etc.
```bash
nano server.properties
```

#### 9. Configuration du démarrage automatique (optionnel)
Pour démarrer automatiquement le serveur après un redémarrage, vous pouvez configurer un service systemd.

Création du fichier de service :
```bash
sudo nano /etc/systemd/system/minecraft.service
```

Contenu du fichier de service :
```ini
[Unit]
Description=Minecraft Server
After=network.target

[Service]
User=youruser
WorkingDirectory=/path/to/your/minecraft/server
ExecStart=/path/to/your/minecraft/server/start.sh
Restart=always
RestartSec=10s
TimeoutStopSec=20s

[Install]
WantedBy=multi-user.target
```

Remplacez youruser par l'utilisateur qui exécutera le serveur, et modifiez les chemins de WorkingDirectory et ExecStart avec le répertoire où se trouve le serveur Minecraft.

Application du fichier de service et démarrage du serveur :
```bash
sudo systemctl daemon-reload
sudo systemctl enable minecraft
sudo systemctl start minecraft
```

Vérification de l'état du serveur :
```bash
sudo systemctl status minecraft
```

Le serveur démarrera automatiquement après un redémarrage.

#### 10. Arrêt du serveur Minecraft
Pour arrêter le serveur, entrez la commande stop :
```bash
stop
```

### Installation et utilisation du client RCON

#### Installation du client RCON
Pour exécuter des commandes RCON, vous devez installer un client RCON. Par exemple : mcrcon (un client RCON simple et facile à utiliser).

Installation de mcrcon :
```bash
sudo yum install -y git gcc
git clone https://github.com/Tiiffi/mcrcon.git
cd mcrcon
make
sudo make install
```

#### Exécution des commandes RCON
Pour envoyer des commandes au serveur Minecraft, exécutez :
```bash
mcrcon -H 127.0.0.1 -P 25575 -p your_password reload
```

1. Vérification de la configuration RCON
Assurez-vous que RCON est activé et correctement configuré sur le serveur Minecraft.

Vérifiez les paramètres RCON dans le fichier server.properties :
```bash
sudo nano /path/to/minecraft/server.properties
```
Assurez-vous que les paramètres suivants sont corrects :
```ini
enable-rcon=true
rcon.password=1234
rcon.port=25575
```

Création du fichier de service systemd
Créez un fichier de service systemd nommé minecraft.service pour configurer le démarrage automatique du serveur Minecraft.

Chemin du fichier de service : /etc/systemd/system/minecraft.service

Contenu du fichier de service :
```ini
[Unit]
Description=Minecraft Server
After=network.target

[Service]
User=mcuser
WorkingDirectory=/home/mcuser/minecraft
ExecStart=/usr/bin/java -Xmx1024M -Xms1024M -jar minecraft_server.jar nogui
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Avec cette configuration, le serveur sera exécuté par l'utilisateur mcuser et redémarrera automatiquement en cas d'échec.

Démarrage et activation du service systemd :
```bash
sudo systemctl daemon-reload
sudo systemctl start minecraft.service
sudo systemctl enable minecraft.service
```

Utilisation d'iptables pour la configuration
iptables est un outil de pare-feu pour contrôler le trafic réseau sur Linux.

1. Vérification des règles de pare-feu actuelles
Vérifiez d'abord les règles de pare-feu actuelles :
```bash
sudo iptables -L -n -v
```

2. Autorisation uniquement du port Minecraft
Le serveur Minecraft utilise par défaut le port 25565. Autorisez la plage d'IP 192.0.0.0/8 (IP commençant par 192) et bloquez le trafic de toutes les autres IP.
```bash
# 1. Autoriser les IP commençant par 192
sudo iptables -A INPUT -p tcp --dport 25565 -s 192.0.0.0/8 -j ACCEPT

# 2. Bloquer les autres IP
sudo iptables -A INPUT -p tcp --dport 25565 -j DROP
```

3. Vérification des règles de pare-feu
Vérifiez que les règles ont été correctement ajoutées :
```bash
sudo iptables -L -n -v
```

Exemple de sortie :
```sql
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 ACCEPT     tcp  --  *      *       192.0.0.0/8          0.0.0.0/0            tcp dpt:25565
    0     0 DROP       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:25565
```

4. Sauvegarde des règles
Les règles iptables sont réinitialisées au redémarrage, donc sauvegardez la configuration.

Debian/Ubuntu :
```bash
sudo apt install iptables-persistent
sudo netfilter-persistent save
sudo netfilter-persistent reload
```

RHEL/CentOS :
```bash
sudo service iptables save
sudo systemctl restart iptables
```

Remarques :
Gestion des plages d'IP : Pour autoriser des IP spécifiques en dehors du réseau interne, ajoutez-les à la plage autorisée.
```bash
sudo iptables -A INPUT -p tcp --dport 25565 -s 203.0.113.0/24 -j ACCEPT
```

IPv6 : Si vous devez prendre en charge IPv6, ajoutez des règles IPv6 séparées.
```bash
sudo ip6tables -A INPUT -p tcp --dport 25565 -s 1920::/16 -j ACCEPT
```

Ajout de logs : Pour enregistrer les IP bloquées dans les logs, ajoutez la règle suivante.
```bash
sudo iptables -A INPUT -p tcp --dport 25565 -j LOG --log-prefix "MC_BLOCKED: "
```

Avec cette configuration, vous pouvez contrôler l'accès au serveur Minecraft en fonction des IP.

Vérification du script
Si le script ne fonctionne pas correctement, vérifiez et résolvez les problèmes potentiels.

Points à vérifier :
Contenu du script : Assurez-vous que le fichier script.sh contient le bon contenu. Vérifiez que le code traite correctement les logs.

Exemple de contenu du script.sh :
```bash
#!/bin/bash

# Chemin du fichier de log (modifiez en fonction de l'emplacement de start.sh)
LOG_FILE="/var/log/start.sh"
# Fichier pour enregistrer les connexions
PLAYER_LOG_FILE="perstente.txt"

# Lire le fichier de log en temps réel et extraire les noms et IP des joueurs
tail -f $LOG_FILE | grep --line-buffered "logged in with entity id" | while read -r line
do
    # Extraire le nom et l'IP des joueurs
    PLAYER_NAME=$(echo $line | sed -n 's/.*\[\(.*\):.*\] logged in.*/\1/p')
    PLAYER_IP=$(echo $line | sed -n 's/.*\[\([^:]*\):.*\] logged in.*/\1/p')

    # Enregistrer le nom et l'IP si présents
    if [ -n "$PLAYER_NAME" ] && [ -n "$PLAYER_IP" ]; then
        echo "$PLAYER_NAME [$PLAYER_IP] logged in at $(date)" >> $PLAYER_LOG_FILE
        echo "Logged: $PLAYER_NAME [$PLAYER_IP]"
    fi
done
```

Emplacement du fichier de log start.sh : Vérifiez que le chemin du fichier de log est correct. Assurez-vous que le chemin LOG_FILE (/var/log/start.sh) correspond bien au fichier où les logs du serveur Minecraft sont enregistrés.

Problème de permissions : Assurez-vous que le fichier perstente.txt a les permissions nécessaires pour être écrit. Si ce n'est pas le cas, utilisez la commande suivante pour changer les permissions :
```bash
sudo chmod 666 perstente.txt
```

Vérification de la commande tail -f : Assurez-vous que la commande tail -f $LOG_FILE fonctionne correctement. Cette commande affiche les nouvelles lignes ajoutées au fichier de log. Si le fichier de log ne contient pas de nouvelles lignes, rien ne sera affiché. Vérifiez que les logs sont correctement générés.

Vérification de l'exécution du script : Vous pouvez exécuter le script.sh en arrière-plan en utilisant nohup pour qu'il continue de fonctionner même après la fermeture du terminal :
```bash
sudo nohup ./script.sh &
```

Emplacement du fichier perstente.txt : Vérifiez que le fichier perstente.txt est bien créé dans le répertoire de travail actuel. Le script.sh étant exécuté dans le répertoire quiatente, le fichier perstente.txt sera également créé à cet emplacement. Utilisez la commande ls pour vérifier la création du fichier.

Vérification finale :
```bash
sudo iptables -A INPUT -p tcp --dport 25565 -j LOG --log-prefix "Minecraft connection attempt: " --log-level 4
```

1. Modifier le fichier de configuration SSH
Ouvrez le fichier de configuration SSH :
```bash
sudo nano /etc/ssh/sshd_config
```
Trouvez ou ajoutez la ligne suivante :
```plaintext
PermitRootLogin no
```
Explication : PermitRootLogin no empêche les connexions SSH avec le compte root.
Enregistrez les modifications et fermez le fichier (Ctrl + O → Entrée → Ctrl + X).

2. Redémarrer le service SSH
Redémarrez le service SSH pour appliquer les modifications :
```bash
sudo systemctl restart sshd
```

3. Désactiver complètement le compte root (optionnel)
Pour verrouiller complètement le compte root, exécutez cette commande :
```bash
sudo passwd -l root
```
Explication : Cette commande verrouille le mot de passe du compte root, empêchant ainsi toute connexion avec ce compte.

4. Tester la configuration
Essayez de vous connecter avec le compte root via SSH :
```bash
ssh root@IP_du_serveur
```
Résultat attendu : Un message d'interdiction de connexion doit apparaître.
Connectez-vous à la place avec le compte jin :
```bash
ssh jin@IP_du_serveur
```
Entrez le mot de passe du compte jin pour accéder au serveur.

5. Ajouter des privilèges sudo au compte jin (si nécessaire)
Pour permettre à l’utilisateur jin d’exécuter des commandes avec des droits d’administration, ajoutez-le au groupe sudo :
```bash
sudo usermod -aG sudo jin
```
Vérifiez que cela fonctionne :
```bash
su - jin
sudo ls /root
```
