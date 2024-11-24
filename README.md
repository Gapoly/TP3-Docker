# TP3-Docker

Dans ce TP, je vais expliquer les technos que j'ai mis en place pour répondre à la problématique et comment je les ai appliqués.

Avant de commencer, je vais remettre en évidence la problématique imposé par l'exercice :

## Problématique

> Partant d'une application 3-tiers existante : un frontend en VueJs, un backend en Python-Flask/FastAPI, une base de données Postgres, il va falloir mettre en place trois conteneurs interagissant entre eux d'une part et manipulant des données de manière pérenne : le code source des applications et les données de la base de données. Cette solution sera déployée dans un réseau dédié, afin de ne pas lui permettre d'interagir avec d'autres conteneurs (et par réciproque, de ne pas se faire attaquer, ou le moins possible, depuis l'extérieur).

L'exercice à était séparé en 4 étapes, qui sont :

- **Le réseau**
- **Les volumes**
- **Les ports**
- **Les variables d'environnement**

### 📶 Le réseau

> Nous devons déployer 3 conteneurs chacun dans son réseau. Un frontend, un backend et une base de données. Le frontend communique avec le backend et le backend communique avec la base de données. Le client se connecte juste au frontend.



### 🧳 Les volumes

> Pour un gérer la disponibilité des données au cas où que les conteneurs ne marche plus. Il faut savoir gérer les volumes pour mettre en place un système de continuité. Pour cela, on va mettre en place des volumes sur l'hôte Docker. Si le conteneur tombe, Docker pourra relancer le fichier *.yml sans perte données, car elles sont sur l'hôte.

### 🛥️ Les ports

> Bien évidemment, il faudra ouvrir les ports sur les conteneurs pour qu'elles puissent communiqués entre elles :

- **MySQL : 3306**
- **HTTP : 80**

> J'ai mis en valeur les 2 ports principaux qui seront utilisés mais bien évidemment il en faudra plus.

### 🛤️ Les variables d'environnement

> Les variables d'environnement permette de pouvoir automatiser le plus possibble le déploiement des conteneurs. Chaque conteneurs propose des variables propre à lui même. Il faudra lire la doc pour les connaitre et savoir lesquels doivent être appliqués.


# 0. Mes Technos

De base, il étai demandé d'utiliser VueJs en frontend, Python-Flask/FastAPI en backend et Postgres en base de données. Etant donné que je suis spécialisé systèmes & réseaux, je ne maitrise ni le Python, ni le VueJs. Je ne pouvais pas partir sur ces technos. De même pour le postgres, je ne connais pas.

Pour palier à ce problème, tout en respectant l'exercice j'ai choisie d'autres technos.

## 0.1 Wordpress

<img hspace="20" align="left" src="https://pngimg.com/uploads/wordpress/wordpress_PNG47.png" alt="Logo Wordpress" width="100"/>

Wordpress est logiciel qui permet de créer des sites web. Dans notre cas, il répond parfaitement au besoin du backend. Un logiciel qui a besoin d'un accès à une base de données, mais on ne veut pas que le client se connecte directement dessus. Evidemment, on ne mettra pas en place de site web, mais il faudra faire en sorte que le Wordpress se connecte à la BDD et que le frontend le redirige bien sur le Wordpress.

- **Réseau :** backend
- **Volumes :** wordpress:/var/www/html
- **Ports :** HTTP 8800:80
 
## 0.2 MySQL

<img hspace="20" align="left" src="https://pngimg.com/uploads/mysql/mysql_PNG23.png" alt="Logo Wordpress" height="100"/>

Pour la base de données, je suis resté sur technologie classique et connue qui est MySQL. Pas besoin de présentation dessus. J'ai choisie cette techno car sur le peux de base de données que j'ai pu touché, cela fut toujours MySQL ou MariaDB. Elle sera suffisante pour la partie base de données.

- **Réseau :** bdd
- **Volumes :** mysql:/var/lib/mysql
- **Ports :** MySQL 3306:3306


## 0.3 Nginx Proxy Manager

<img hspace="20" align="left" src="https://nginxproxymanager.com/icon.png" alt="Logo Wordpress" height="100"/>

Nginx Proxy Manager (NPM) est un proxy inverse doté d'une interface web pour gérer facilement les paramétrage du serveur. Il propose de configurer et d'administrer des proxys inverses, des certificats TLS, et des redirections de sites web. Cette techno rentrera parfaitement dans le rôle de frontend.

<br>

- **Réseau :** frontend
- **Volumes :** /docker/nginx/data:/data - /docker/nginx/letsencrypt:/etc/letsencrypt
- **Ports :** Interface HTTP 8181:81 - Redirection HTTP 8008:80 - Redirection HTTPS 4443:443

## 0.4 Finalité

Pour reprendre vite fait les technos utilisés. Nous aurons un NPM qui fera office de frontend pour rediriger le traffic vers le Wordpress, qui à son tour se connectera à MySQL qui hébergera la base de données.

L'arborescence final ressemblera à cela :

```
└── Hôte Docker
    ├── Nginx Proxy Manager
    ├── Wordpress
    └── MySQL
```

# 📶 1. Le réseau

Comme dit précedemment, le réseau sera coupé en trois pour chaque service. Le frontnet qui fera office de frontend, le backnet qui fera office de backend le bdd pour la base de donées.

- **Wordpress :** frontnet, backnet, bdd
- **NPM :** frontnet
- **MySQL :** bdd

