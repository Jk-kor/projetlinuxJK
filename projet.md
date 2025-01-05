# 1. Objectifs du projet :
## 1.Créer une plateforme interactive : Un site web qui permet aux utilisateurs de se connecter à un serveur Minecraft via une interface intuitive.
## 2.Gérer les données du serveur Minecraft : Utiliser une base de données MySQL pour stocker et gérer des informations liées au serveur Minecraft, comme les utilisateurs, les scores, les statistiques de jeu, etc.
## 3.Assurer la sécurité : Implémenter des mécanismes robustes pour protéger les données des utilisateurs et sécuriser les connexions au serveur Minecraft.



j'utilise systemd pour que le serveur commentce automatiquement des que je lance rocky 

```
sudo nano /etc/systemd/system/minecraft.service
                                                            [Unit]
Description=Minecraft Server
After=network.target

[Service]
User=root
WorkingDirectory=/minecraft
ExecStart=/minecraft/start.sh
Restart=always
RestartSec=10s
TimeoutStopSec=20s

[Install]
WantedBy=multi-user.target

sudo systemctl daemon-reload
sudo systemctl enable minecraft
sudo systemctl start minecraft


```