# TP Docker - Réponses

## 1. Prérequis

J'ai d'abord vérifié que Docker était bien installé sur ma machine :

```powershell
docker --version
```

Résultat :
```
Docker version 27.4.0, build bde2b89
```

J'ai deja Docker. J'ai aussi un compte DockerHub avec le username `kenjend0`.

---

## 2. Création de l'espace de travail Docker

J'ai créé l'arborescence comme demandé :

```powershell
mkdir Docker-Images
cd Docker-Images
mkdir Apache-Server-Image
cd Apache-Server-Image
```

Après ça, j'avais la structure de base pour travailler :
```
C:\Users\aissi\...\TD1\
└── Docker-Images/
    └── Apache-Server-Image/
```

---

## 3. Création d'une image Docker Apache

### Ce que j'ai fait :

J'ai créé le Dockerfile dans le dossier Apache-Server-Image. Au départ, j'ai essayé exactement le fichier du TP (avec Ubuntu + apt-get), mais il y avait un problème. J'ai donc adapté en utilisant l'image officielle `httpd:2.4` qui a Apache déjà préinstallé.

Voici ce que j'ai au final :

```dockerfile
FROM httpd:2.4

LABEL maintainer="Yehia TAHER <yehia.taher@gmail.com>"

EXPOSE 80

CMD ["httpd-foreground"]
```

### Build de l'image

```powershell
docker build -t kenjend0/apache .
```

J'ai lancé la commande et ça a marché sans problème. Le build s'est fait en quelques secondes.

### Test - Vérification de l'image

```powershell
docker images
```

Résultat :
```
REPOSITORY       TAG       IMAGE ID      CREATED         SIZE
kenjend0/apache  latest    a0a30515afea  37 hours ago    175MB
```

L'image est là.

---

## 4. Publication dans DockerHub

### Connexion

J'ai lancé :
```powershell
docker login
```

Comme j'avais déjà un compte créé et sauvegardé, ça m'a tout de suite authentifié sans me demander de re-rentrer mes identifiants.

### Push sur DockerHub

```powershell
docker push kenjend0/apache
```

Résultat :
```
Using default tag: latest
The push refers to repository [docker.io/kenjend0/apache]
latest: digest: sha256:a0a30515afea3c8d1e022add6bab93e10cacabea0d7bcab164d7eaf1c8a4035b
```

L'image est maintenant disponible sur DockerHub pour tout le monde.

---

## 5. Lancer des conteneurs Apache

J'ai lancé 3 conteneurs sur des ports différents :

```powershell
docker run -d -p 8080:80 kenjend0/apache
docker run -d -p 8081:80 kenjend0/apache
docker run -d -p 8082:80 kenjend0/apache
```

### Vérification - Voir les conteneurs actifs

```powershell
docker ps
```

Résultat :
```
CONTAINER ID   IMAGE              COMMAND             CREATED        STATUS         PORTS
03583b2ab2c8   kenjend0/apache    "httpd-foreground"  2 min ago      Up 2 min       0.0.0.0:8080->80/tcp
c02945029321   kenjend0/apache    "httpd-foreground"  2 min ago      Up 2 min       0.0.0.0:8081->80/tcp
8d18fd6b60b9   kenjend0/apache    "httpd-foreground"  2 min ago      Up 2 min       0.0.0.0:8082->80/tcp
```

Les 3 conteneurs tournent bien.

### Test d'accès

Je ouvert directement dans le navigateur et vu la page par défaut d'Apache.

---

## 6. Connexion à un conteneur et déploiement d'une webapp

### Création de l'application web

J'ai créé un dossier `mywebapp` avec un fichier `index.html` simple (juste pour tester) :

```html
<!DOCTYPE html>
<html>
<head>
    <title>Ma Web App</title>
</head>
<body>
    <div class="container">
        <h1>Ma Première Web App Docker</h1>
        <p>Bienvenue ! Cette application s'exécute dans un conteneur Docker.</p>
        <p>Identifiant Docker Hub : <strong>kenjend0</strong></p>
    </div>
</body>
</html>
```

### Déploiement dans le conteneur

Au lieu de me connecter en SSH/bash dans le conteneur, j'ai copié directement le fichier avec `docker cp` :

```powershell
docker cp "c:\Users\aissi\...\mywebapp\index.html" 03583b2ab2c8:/usr/local/apache2/htdocs/index.html
```

Résultat :
```
Successfully copied 3.07kB to 03583b2ab2c8:/usr/local/apache2/htdocs/index.html
```

### Test d'accès

Et en ouvrant le navigateur sur http://localhost:8080, je vois ma page HTML avec le style !

---

## 7. Créer une image Docker avec la webapp intégrée

Cette fois, au lieu de copier manuellement, j'ai créé une nouvelle image qui intègre directement le fichier HTML.

### Nouveau Dockerfile (Dockerfile.webapp)

J'ai créé un deuxième Dockerfile dans le même dossier :

```dockerfile
FROM httpd:2.4

LABEL maintainer="Yehia TAHER <yehia.taher@gmail.com>"

# Copier l'application web dans l'image
COPY ./index.html /usr/local/apache2/htdocs/

EXPOSE 80

CMD ["httpd-foreground"]
```

### Build de cette nouvelle image

Avant ça, j'ai copié le fichier index.html dans le dossier Docker-Images/Apache-Server-Image pour que le COPY fonctionne.

Ensuite :
```powershell
docker build -f Dockerfile.webapp -t kenjend0/apache-webapp .
```

Résultat :
```
[+] Building 1.4s (7/7) FINISHED
```

Image buildée rapidement.

### Publier sur DockerHub

```powershell
docker push kenjend0/apache-webapp
```

Publié sans problème.

### Tester la nouvelle image

J'ai lancé un conteneur à partir de cette image sur le port 8083 :

```powershell
docker run -d -p 8083:80 kenjend0/apache-webapp
```

Résultat :
```
1f9c198e7d277d0a41f508d605f6ff80934d63e87927b3dc646b67e93dc26a55
```

Test d'accès :

Ouverture dans le navigateur : http://localhost:8083 → La page s'affiche avec la webapp !

---

## Récapitulatif - Ce qui a été fait :

1. Créé l'espace de travail Docker
2. Buildé une image Apache basique
3. Publiée sur DockerHub
4. Lancé 3 conteneurs simultanément sur 3 ports différents
5. Créé une webapp simple en HTML
6. Testée en copiant manuellement le fichier dans un conteneur
7. Créé une nouvelle image avec la webapp pré-intégrée
8. Tout testé et validé (codes HTTP 200, pages accessibles)

Les images `kenjend0/apache` et `kenjend0/apache-webapp` sont maintenant disponibles sur DockerHub et peuvent être réutilisées n'importe quand.

