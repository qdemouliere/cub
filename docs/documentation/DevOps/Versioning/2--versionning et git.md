# Gestion des versions et Git

La **gestion des versions** couvre la planification, la conception, le développement, les tests, le déploiement et le contrôle des logiciels. Son objectif est de garantir que les équipes livrent les applications et mises à jour nécessaires tout en préservant l’intégrité de l’environnement de production. Dans un contexte d’entreprise dynamique, elle permet de gérer efficacement les changements grâce au contrôle des versions et à l’automatisation des déploiements, favorisant la livraison continue et la transformation numérique.

## I. Gestion des versions dans ITIL

Dans le cadre d’ITIL, la gestion des versions et des déploiements fait partie de la phase de **Transition des services**. ITIL est une référence en gouvernance des services informatiques, visant à fournir des produits de qualité tout en maîtrisant les coûts.

## II. Processus de gestion des versions

1. **Demande** : identification des nouvelles fonctions ou modifications, évaluation de leur pertinence et faisabilité.  
2. **Planification** : définition de la structure de la version, des jalons et responsabilités via un workflow ou une checklist.  
3. **Conception et réalisation** : développement et intégration des fonctionnalités dans le logiciel.  
4. **Tests** : validation fonctionnelle et non fonctionnelle, avec corrections itératives jusqu’à la préparation au déploiement.  
5. **Déploiement** : mise en production et formation des utilisateurs.  
6. **Après-déploiement** : support et enregistrement des bugs pour les futures versions.

## III. Systèmes de gestion de versions

- **Locaux** : simples mais peu fiables, risques d’écrasement ou de perte de fichiers.  
- **Centralisés (CVCS)** : dépôt central partagé (ex. Subversion, CVS), facilite la collaboration mais présente un point de défaillance unique.  
- **Distribués (DVCS)** : chaque client possède une copie complète du dépôt (ex. Git, Mercurial), résilient et adapté à des développements parallèles multiples.

## IV. Git

Git a été créé en 2005 pour remplacer BitKeeper dans le projet Linux. Il se distingue par sa rapidité, sa nature distribuée, son efficacité pour les projets complexes et sa capacité à gérer des développements non linéaires avec de nombreuses branches parallèles.

Git fonctionne en enregistrant l’état du projet sous forme d’**instantanés** ou snapshots. Cela permet de ne stocker à nouveau que les fichiers modifiés, économisant ainsi de l’espace. La plupart des opérations dans Git s’effectuent **localement**, sans nécessiter de connexion réseau, ce qui rend le travail rapide et autonome.

L’**intégrité** des fichiers est assurée par des empreintes SHA-1. Chaque fichier et répertoire est vérifié grâce à cette signature unique, garantissant que toute modification ou corruption est détectée.

### États des fichiers dans Git

- **Modifié** : fichier changé mais non préparé pour l’enregistrement.  
- **Indexé** : fichier prêt à être inclus dans le prochain snapshot.  
- **Validé** : fichier sauvegardé dans la base locale.

### Structure d’un projet Git

- **Répertoire Git** : contient les objets et métadonnées du projet.  
- **Répertoire de travail** : fichiers extraits pour modification.  
- **Zone d’index** : fichiers préparés pour le prochain snapshot.

### Cycle standard

Le cycle standard dans Git suit trois étapes :  
1. Modification des fichiers.  
2. Indexation pour préparer le snapshot.  
3. Validation pour sauvegarder les changements dans la base locale.

## V. Tutoriel des commandes Git essentielles

Voici les commandes de base pour démarrer avec Git :  

### Initialisation et configuration

```bash
git init           # Crée un nouveau dépôt Git
git config --global user.name "Votre Nom"
git config --global user.email "votre.email@example.com"
```
### Gestion des fichiers

```bash
git status         # Vérifie l'état des fichiers
git add <fichier>  # Ajoute un fichier à la zone d'index
git add .          # Ajoute tous les fichiers modifiés
git commit -m "Message"  # Valide les changements dans le dépôt
```

### Gestion des branches

```bash
git branch         # Liste les branches existantes
git branch <nom>   # Crée une nouvelle branche
git checkout <nom> # Bascule vers une autre branche
git merge <branche> # Fusionne une branche dans la branche courante
```

### Historique et exploration

```bash
git log            # Affiche l'historique des commits
git diff           # Montre les différences non indexées
git show <commit>  # Affiche les détails d'un commit
```

### Travail avec un dépôt distant

```bash
git remote add origin <url>  # Lie un dépôt distant
git push -u origin main       # Envoie les commits locaux vers le dépôt distant
git pull origin main          # Récupère et fusionne les changements du dépôt distant
```
