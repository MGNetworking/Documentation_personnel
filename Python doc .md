# Python 

## Shortcut python 

- Ouvrir le shell python
```ps
python
```

- Liste des packages python, répertorie tous les paquets que vous avez installés (et leurs dépendances) dans un format intelligible
```ps
pip list
```

- répertorie les paquets que vous avez installés (mais pas leurs dépendances) dans un format adapté au stockage dans un fichier (souvent appelé fichier requirements.txt)
```ps
pip freeze
```

- Obtenir des informations sur plusieurs paquets, montre des informations utiles sur un ou plusieurs paquets que vous avez installés
```ps
pip show < package >
```

- Installer un version de package ciblé
```ps
pip install < package >==2.1.0
```
Plus d'info :<https://openclassrooms.com/fr/courses/6951236-mettez-en-place-votre-environnement-python/7013778-utilisez-le-systeme-de-versioning-de-paquets#/id/r-7013760>

- Désintallation d'un package
```ps
pip uninstall < package >
```

## Gestion des environements python
- Création d'un environement python 

Lorsque vous démarrez un projet, vous créez un environnement virtuel. Chaque environnement virtuel comprend **sa propre version de Python** et tous les paquets Python que vous décidez d'y installer.
  
```ps
python -m venv env
```

- Activer de l'environnement sous windows avec powershell => ps1 

Dans cette exemple env_1 est le nom du repertoire contenant l'environement virtuel.
Aussi en fonction du terminal avec le quel vous travailler, vous devez soit un '.bat' ou '.ps1' etc
```ps
env/Scripts/Activate.ps1
```
- Activation sous linux
```ps
source env/bin/activate
```
Selon le terminal dans le quel vous travaillé (apres activation de votre environement), le nom de votre environement virtuel sera inscript devant votre terminal, pour testé l'environement virtuel excuter la list des packages. Normalment la list doit étre vide
```ps
pip freeze
```
Désactivation de l'environement 
```ps
deactivate
```

NB: Il est intéressant de noter que vous n'avez pas besoin de désactiver un environnement virtuel avant d'en activer un autre. Le fait d'activer un environnement prévaudra si vous vous trouvez déjà dans un autre.

Dans le but d'avoir une bonne pratique de gestion des ces environement virtuel, ont les créer dans repertoir, avec pour convention de nommage d'environement env. Si le projet versionner il possedera son '**.gitignore**' pour y ajoute l'environemnt qui ne doit pas être present dan le dépot.  

Créatrion d'un fichier sous Powershell

```ps
New-Item -ItemType file .gitignore
```

- Le fichier requirements.txt
Pour garantir que tous les développeurs travaillant sur un projet utilisent le même environnement virtuel, nous utilisons un fichier ***requirements.txt***

Creation automatique du fichier requirements.txt
```ps
pip freeze > requirements.txt
```

- Installer des package à l'aide du fichier requirements
```ps
 pip install -r requirements.txt
```

 Cette manipulation de manière récursif va installer tou les packages listé dans le fichier
 
 ***requirements.txt***

 - Supprimer le dossier et les sous-dossier : linux commande 
```ps
 rm -r env
```

  - Supprimer le dossier et les sous-dossier : powershell
```ps
Remove-Item repertoir -Recurse -Force
```
