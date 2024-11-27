## Pré-paramétrage Wordpress

> Avant de configurer Nginx Proxy Manager, il faudra d'abord créer le site Wordpress. Choisir la langue, le nom du site et un login.

## Configuration rapide de NPM :

1. Télécharger le dossier nginx-data

2. Mettez son contenu dans votre volume nginx-data créer par Docker

3. Se connecter à NPM sur le port 8181.

4. Login avec les identifiants par défaut qui sont :

```
Login : 1@1.com
Password : azerty123
```
5. Cliquer sur le Proxy Host existant

6. Changer l'IP par celle de votre hôte Docker

7. Rajouter une entrée DNS à votre machine pour associer l'IP au domaine :

### Linux :

```bash
sudo nano -l /etc/hosts
```

Ajouter :

```bash
# IP de la machine - TAB - nom de domaine
# exemple
192.168.1.205   www.tp3.lan
```

### Windows

1. Ouvrir un éditeur de texte en admininistrateur

2. Modifier le fichier "C:\Windows\System32\drivers\etc\hosts"

3. Ajouter la ligne suivante en bas :

```powershell
# IP de la machine - espace - nom de domaine
# exemple
192.168.1.205 www.tp3.lan
```

4. Enregistrer

5. Vous pouvez maintenant vous connecter au Wordpress en entrant l'adresse suivante :

> http://www.tp3.lan:8008

<br>

## Configuration manuelle de NPM :

1. Se connecter à NPM sur le port 8181.

2. Login avec les identifiants par défaut qui sont :

```
Login : admin@example.com
Password : changeme
```

3. Changer le login par défaut

4. Dans Dashboard, cliquer sur Proxy Hosts puis Add Proxy Host.

5. Paramétrer le Proxy Host

- Domain Names : Entrez un domaine au choix (finissant par *.lan ou *.local)
- Scheme : HTTP
- Forward Hostname / IP : IP de la machine Wordpress
- Forward Port : 8800
- Save

Voici un exemple :

![config npm](image.png)

6. Rajouter une entrée DNS à votre machine pour associer l'IP au domaine :

### Linux :

```bash
sudo nano -l /etc/hosts
```

Ajouter :

```bash
# IP de la machine - TAB - nom de domaine
# exemple
192.168.1.205   www.tp3.lan
```

### Windows

1. Ouvrir un éditeur de texte en admininistrateur

2. Modifier le fichier "C:\Windows\System32\drivers\etc\hosts"

3. Ajouter la ligne suivante en bas :

```powershell
# IP de la machine - espace - nom de domaine
# exemple
192.168.1.205 www.tp3.lan
```

4. Enregistrer

5. Vous pouvez maintenant vous connecter au Wordpress en entrant l'adresse suivante :

> http://www.tp3.lan:8008
