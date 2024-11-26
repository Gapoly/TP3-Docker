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

L'inconvénient de ce logiciel est que le paramétrage ce fait sur l'interface web. Pour palier à ce problème, j'ai prévu une config déjà prête à utiliser. La documentation pour l'installation se trouver dans **config.md**.

<br>

- **Réseau :** frontend
- **Volumes :** nginx-data:/data - nginx-tls:/etc/letsencrypt
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

Pourquoi j'ai décidé de mettre Wordpress dans les trois réseaux? Docker marche un peu différement comparé à un réseau normal. Admettons, je suis dans un réseau avec plusieurs VLAN, je fairais une règle pour que les VLAN puissent parler entre eux, or en Docker ce n'est pas possible. Les réseaux ne peuvent pas communiquer entre eux, du coup je suis obliger de mettre Wordpress dans plusieurs réseaux pour qu'ils puissent parler aux autres.

Voici comment je l'ai appliquer :

```yaml
services:

    wordpress:
        networks:
            - backnet
            - bdd
            - frontnet

    mysql:
        networks:
            - bdd

    nginx:
        networks:
            - frontnet

networks:
  bdd:
  frontnet:
  backnet:
```

# 🧳 2. Les volumes

Les volumes sont extremement important dans Docker pour assurer la continuité des services. Elles permettent de garder nos données si un de nos conteneurs venaient à tomber.

J'ai fait en sorte que chaque conteneurs garde les données importantes pour mettre en place la continuité des services :

```yaml
services:

    wordpress:
        volumes:
            - wordpress:/var/www/html

    mysql:
        volumes:
            - mysql:/var/lib/mysql
    nginx:
        volumes:
            - nginx-data:/data
            - nginx-tls:/etc/letsencrypt

volumes:
  wordpress:
  mysql:
  nginx-data:
  nginx-tls:
```

# 🛥️ 3. Les ports

Par défaut dans un containeur Docker, les ports sont fermés. Il faut ajouter un paramétre dans notre fichier *.yml pour qu'il ouvre les ports dont on a besoin pour que les containeurs puissent bien communiquer entre eux.

Chaque containeur ont besoin de ports spécifiques, précisez dans la doc officiel. Certains protocole dont a besoin utilisent les mêmes ports entre eux. On va faire en sorte qu'il n'y ai pas de conflit de ports :

```yaml
services:

    wordpress:
        ports:
            - 8800:80 # HTTP
            - 4433:443 # HTTPS

    mysql:
        ports:
            - 3306:336 # MySQL - Base de données
    nginx:
        ports:
            - 8008:80 # Redirection HTTP
            - 4443:443 # Redirection HTTPS
            - 8181:81 # HTTP - Interface Admin web        
```

# 🛤️ 4. Les variables d'environnement

Les variables d'environnement permettent d'automatiser le plus possible de déploiement des containeurs. Comme dans le cas des volumes, si on a un conteneur qui tombe, le Docker Compose ou Swarm à juste à relire le fichier *.yml pour le relancer avec exactement les mêmes paramètres.

Je suis rester sur les paramètres essentiels pour le bon fonctionnement du TP :

```yaml
services:

    wordpress:
        environment:
            WORDPRESS_DB_HOST: mysql
            WORDPRESS_DB_USER: wp-user
            WORDPRESS_DB_PASSWORD: wordpress
            WORDPRESS_DB_NAME: wordpress_esgi

    mysql:
        environment:
            MYSQL_DATABASE: wordpress_esgi
            MYSQL_USER: wp-user
            MYSQL_PASSWORD: wordpress
            MYSQL_RANDOM_ROOT_PASSWORD: '1'     
```

Nginx Proxy Manager vient sans variable d'environnement car son paramétrage ce fait sur l'interface web.

# ❤️ 5. Le healthcheck

Cette étape, n'était pas obligatoire mais j'ai quand même voulu la faire. C'est quoi le healthcheck et à quoi ça sert?

Le healthcheck est un module qu'on va ajouter à un ficher *.yml ou un dockerfile. On va ajouter une condition qui va nous permettre de dire si notre containeur marche de la manière qu'il est sensé marché. Par exemple, pour un wordpress, on va faire une commande curl pour être sûr que le site foncionne bien. Si le curl ressort une réponse négatif, alors notre containeur est "unhealthy".

Voici les healthcheck que j'ai mis dans mon fichier *.yml :

```yaml
services:

  wordpress:
    healthcheck:
      test: ["CMD", "curl", "--fail", "http://localhost:80"]
      interval: 60s
      retries: 5
      start_period: 20s
      timeout: 10s
    depends_on:
      mysql:
        condition: service_healthy
# J'ai fais en sorte que le wordpress ne lance pas, si il MySQL à un status unhealthy
  mysql:
    healthcheck:
      test: ["CMD", "mysqladmin" ,"ping", "-h", "localhost"]
      timeout: 20s
      retries: 10

  nginx:
    healthcheck:
      test: ["CMD", "curl", "--fail", "http://localhost:81"]
      interval: 60s
      retries: 5
      start_period: 20s
      timeout: 10s
```

# 6. Conclusion

En conclusion, je fais vais faire un récapitulatif rapide des technos utilisés et comment elles ont pu répondre à la problématique imposé par l'exercice.

Tout d'abord, je ne suis pas resté sur les technos de base étant donné que j'ai aucune compétence en dev. J'ai décidé de partir sur des technos en rapport avec mon spécialité, systèmes & réseaux qui sont :

- **Wordpress** *( J'aurais pu remplacer par un GLPI ou un Centreon )*
- **MySQL** *( La seule base de données que je maitrise )*
- **Nginx Proxy Manager** *( Logiciel permettant la redirection vers le site en le gardant sécurisé )*

En suite pour les problématiques :

- **Les réseau** *(Créer 3 réseaux pour chaque containeur pour que le client ai seulement accès au frontend et backend)*
- **Les volumes** *(Mettre en place des volumes pour assurer la continuité des services en cas de panne de containeur)*
- **Les ports** *(Ouvrir les ports nécessaire au bon fonctionnement et la communication entre les containeurs et le client)*
- **Les variables d'environnement** *(Assurer la continuité des services en automatisant le redéploiement des containeurs automatiquement)*
- **Le healthcheck** *(Non obligatoire. Permet de vérifier le bon fonctionnement d'un containeur sans avoir besoin de se connecter directement dessus)*
