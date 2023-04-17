# Jenkins

### Introduction

Nous allons voir avec Jenkins parti CI/CD (relation avec github et les projet sur le serveur) la mise en place de pipeline.

#### Parti 1 le Conteneur jenkins

1. Création Dockerfile jenkins
2. Création d'un docker compose Jenkins
   1. La création du docker-compose
   2. L'accès au socket Docker Host depuis le conteneur jenkins

#### Parti 2 Github et Jenkins

4. Configuration de la clef d'host GIT
5. Générationn d'une clef SSH et la lier avec Github
6. Création d'un pipeline dans Jenlins et d'un WebHook vers Github

## 1. Création Dockerfile jenkins

Voici un exemple de création de dockerfile. Ce dockerfile utilise l'images `jenkins/jenkins:lts-jdk11` voir [docker hub](https://hub.docker.com/r/jenkins/jenkins/tags).

Ce dockerfile ajoute les dependances docker et docker compose a l'image **Jenkins** puis fait le mise à jour et ajout les droits jenkins au groupe docker.

```dockerfile
FROM jenkins/jenkins:lts-jdk11
LABEL authors="Maxime Ghalem"

USER root

# Configurer le référentiel
# Mettez à jour
RUN apt-get update && apt-get full-upgrade -y

# Puis l’installation des dépendances
RUN apt-get install \
    ca-certificates \
    curl \
    gnupg

# Ajouter la clé GPG officielle de Docker
RUN install -m 0755 -d /etc/apt/keyrings \
    && curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg \
    && chmod a+r /etc/apt/keyrings/docker.gpg

# Utilisez la commande suivante pour configurer le dépôt
RUN echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  tee /etc/apt/sources.list.d/docker.list > /dev/null

# Mise à jour de la sources de paquets
RUN apt-get update && apt-get full-upgrade -y

# Installer la dernière version Docker
RUN apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# Ajoute l'utilisateur jenkins au groupe docker
RUN usermod -aG docker jenkins

# retour au user jenkins
USER jenkins
```

## 2. Création d'un docker compose Jenkins

### La création du docker-compose

Voici une implementation de docker-compose. Elle utilise le dockerfile précédent dans la parti `context: .` qui localise le dockerfile. Ce docker compose excutera le build du dockerfile pour créer son image.

```docker
version: '3.8'

services:

    jenkins:
        image: jenkins/docker
        container_name: jenkins
        build:
            context: .
        user: jenkins
        restart: always
        ports:
            -   ${JENKINS_HTTP}           # http_port  le client jenkins
            -   ${JENKINS_AGENT}          # agent_port  master / slave communication

        volumes:
            -  $PWD/jenkins_home:/var/jenkins_home          # le dossier de jenkins
            - /var/run/docker.sock:/var/run/docker.sock     # accès au démon Docker

        environment:
            - JENKINS_OPTS="--prefix=/jenkins"

```

Commade pour lancer le docker compose dans l'Host

```PS
docker compose up -d
```

### L'accès au socket Docker Host depuis le conteneur jenkins

Les droits en lecture/écriture du socket Docker doivent correspondre à l'utilisateur ou au groupe qui exécute le conteneur Docker, afin que le conteneur puisse communiquer avec le daemon Docker sur l'hôte. Par exemple, si vous exécutez le docker-compose en tant qu'utilisateur "user1", vous devez vous assurer que l'utilisateur "user1" a les droits en lecture/écriture sur le socket Docker. Vous pouvez également ajouter l'utilisateur "user1" au groupe Docker et donner au groupe Docker les droits en lecture/écriture sur le socket Docker.

![resultat commande](./pictures/jenkins_11.png)

Pour vérifier les droites :

```ps
ls -al /var/run/docker.sock
```

Dans cette exemple, on peut voir que le groupe max a les droits de lecture écriture sur le socket docker.Ce qui veux dire que l'utilisateur qui appartient a ce groupe doit exécuter le docker-compose.

La commande pour la modification de groupe vers max :

```ps
sudo chgrp max /var/run/docker.sock
```

La commande pour la modification le user vers max et le groupe vers root:

```ps
sudo chown max:root /var/run/docker.sock
```

La commande pour modifier les accès

```ps
sudo chmod 660 /var/run/docker.sock
```

Liste du processus d'exécution:

- Le docker compose est excuter avec les droits `max`
- Le dockerfile ajoute a l'images Jenkins les couches docker
- Manuellement on vérifie que le conteneur jenkins puis accèder au socket docker
- Si il ne l'ai pas, on lui données accès via la machine Host en ajout au groupe l'utiisateur courant ou l'utilisateur avec l'id `1000` si besoin.

## 4. Configuration de la clef d'host GIT

Avant de créer une clef SSH pour la communicationn entre **_jenkins_** et **_Github_** , il faut impérativement procédes a la configuaration de la clef d'host.

1. Aller dans configurtion globale de sécuriter

![image info](./pictures/jenkins_1.png)

2. En bas de cette page, dans la parti **_Git Host Key Verification configuration_**. Selectionner Accept first connection et enregistrer. Sans cette configuation, vous ne pourrez pas créer de connection entre Github et jenkins avec vos clef de cryptage.

![image info](./pictures/jenkins_2.png)

## 5. Générationn d'une clef SSH et la lier avec Github

1. Dans votre Host, vérifier les clef existant

```ps
ls -al ~/.ssh
```

2. Vérifier que l'agent ssh est en cours d'exécution d'execution avec la commande :

```ps
eval "$(ssh-agent -s)"
```

Exemple de résultat:

```ps
Agent pid 4100
```

3. Génération d'une clef SSH

La commande suivant créera une nouvelle clé SSH, en utilisant l'e-mail fourni comme étiquette avec l'algo de cryptage selectionné.

Pour plus d'info => [Comment utiliser ssh-keygen pour générer une nouvelle clé SSH ?](https://www.ssh.com/academy/ssh/keygen)

Ici l'algo de cyptage : **ed25519**

```ps
ssh-keygen -t ed25519 -C "email@gmail.com"
```

Ici l'algo de cyptage : **RSA**

```ps
ssh-keygen -t rsa -b 4096
```

4. Ajoutez votre clé privée SSH au ssh-agent

```ps
ssh-add ~/.ssh/id_ed25519
```

```ps
ssh-add ~/.ssh/id_rsa
```

copier la clef public "**.pub**" pour l'ajoute à GitHub

```ps
cat id_ed25519.pub
```

Puis copier là et aller sur Github dans la section **Setting**

![image info](./pictures/jenkins_3.png)

Aller dans SSH and GPG keys

![image info](./pictures/jenkins_4.png)

Cliqué sur New SSH Key

![image info](./pictures/jenkins_5.png)

Puis coller votre clef SSH et attribuer un nom a cette clef de connection.

![image info](./pictures/jenkins_6.png)

Et voilà , vous avez plus cas tester votre clef.

5. Importation de la clef dans Jenkins

## 6. Création d'un pipeline Jenlins et d'un WebHook vers Github

Aller dans la section Nouvelle item, puis entre le nom de pipeline selectionné Pipeline puis OK , comme sur l'images.

![image info](./pictures/jenkins_7.png)

Pour le **WebHook** Aller dans Build triggers et fait la selection **GitHub hook trigger for GISTcm polling**

![image info](./pictures/jenkins_8.png)

Puis aller dans la section Pipeline et entre les donnée correspondant a votre projet. La parti credentials et obligatoire pour la connection de projet privet.
Dans cette exmple j'ai créer une clef privet entre Github et jenkins nommé preprod-key. Pour créer une cette clef dans jenkins et Githut ce reporter a la section : **Générationn d'une clef SSH et la lier avec Github**

1. Vous pouvez voir l'url et en SSH
2. La clef de connection
3. La branch de travail sur laquel on excutera le build

![image info](./pictures/jenkins_9.png)

Puis la gestion du script qui sera utiliser. Dans notre cas ce sera un **jenkinfile**

![image info](./pictures/jenkins_10.png)

Puis cliquer sur sauver pour quiter. Vous n'avez plus qu'a lancer le build pour test la communication est l'excution.
