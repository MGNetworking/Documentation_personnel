# Sphinx création d'une documentation étape part étape

## I Composition de base d'un projet python
  - son environement virtuel 
  - son fichier git et '.gitignore' pour le versionning
  - son fichier requirements.txt pour la gestion des packages du projet
  
## II Liste des étapes de création d'une documentation avec sphinx
### 1. Création du repertoir projet et déplacement dans celui-ci
```ps1
mkdir projet

cd projet
``` 
### 2. Création d'un environement virtuel python et son activation (avec Powershell)

```ps1
python -m venv env

env\Scripts\Activate.ps1
``` 

### 3. Installé sphinx dans l'environement virtuel du projet 

```ps1
pip install sphinx
```

ou 

```ps1
pip install sphinx sphinx-autobuild
```

Dans le questionnaire, pour la séparation entre y pour oui cela permettra d'utilisé l'autobuild avec le repertoir source.

### 4. Création du référenciel git   
Initialisation du git local
```ps1
git init
```
- Avec PowerShell

Création du fichier '**.gitignore**'
```ps1
New-Item -ItemType file .gitignore
```

- Avec Linux

Création du fichier '**.gitignore**'
```ps1
touch .gitignore
```

### 5. Création du fichier requirements.txt

- Avec PowerShell
```ps1
New-Item -ItemType file requirements.txt
```
- Avec PowerShell
```ps1
touch requirements.txt
```
L'ajout des packages de l'environement au fichier requirements.txt permet la gestion de son environement local.  
La commande suivant permet, de créer le fichier 'requirements.txt' (si il n'est pas déjà créer) et dèy ajout tout les packages de l'environement 
```ps1
pip freeze > requirements.txt
```

Vérification de l'ajout 
```ps1
cat requirements.txt
```

### 6. Puis lancement de la creation avec la commande rapide    
```ps1
sphinx-quickstart
``` 
Apres avoir créer ce repertoire avec tout les fichiers sphinx, vous pouvez l'exploité et ecrire votre documention dans le fichier **index.rst**

### 7. Fair la compliation de la documentation 
- powershell 
```ps1
./make html
```
- linux 
```ps1
make html
```
Apres compilation vous trouverez, dans le repertoire '**build/html**', le fichier index.html générer  

```ps1
.\build\html\index.html
```
ou 
```ps1
 .\_build\html\index.html
```
NB: Si vous déplacez le répertoire contenant le projet vers un autre en droit sur le PC, vous devrez supprimer puis recréer un environnement python.

L'environnement python garde en mémoire le ``path`` de sa création. Est donc, dès qu'il sera utilisé cela créera des erreurs. 

Le fichier ``requirements.txt`` permet de recréer celui-ci et d'évite d'avoir dans les dépots une somme de fichier inutile.
 
#### 7.1 Autre façon de créer un build 

Installation de sphinx-autobuild     
voir ici :<https://pypi.org/project/sphinx-autobuild/>   

```ps1
pip install sphinx-autobuild
```

Faire un build classique 
```ps1
sphinx-autobuild docs docs/_build/html
```
Ou , si vous avez séparer le repertoire build 
```ps1
sphinx-autobuild source build/html
```

### 8. Installation le thème 
```ps1
pip install sphinx-rtd-theme
```
Et dans le fichier 'conf.py', modifier la variable 'html_theme'
```ps1
html_theme = 'sphinx_rtd_theme'
```
