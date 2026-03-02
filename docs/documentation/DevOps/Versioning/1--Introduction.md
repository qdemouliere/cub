# Introduction à la gestion des révisions

## 1. Qu'est-ce que le versioning ?

Le versioning est un concept essentiel dans le domaine du développement logiciel et de la gestion de projet. Il fait référence aux techniques et méthodes utilisées pour gérer les modifications d'un produit ou d'un fichier au fil du temps. En adoptant une stratégie de versioning, les équipes peuvent maintenir un historique des changements, faciliter la collaboration et garantir la qualité du produit final.

Bien que ce concept soit la plupart du temps mis en avant dans le cadre de la production collaborative de code sur une forge logicielle (GitHub, Gitlab, Codeberg). Il a également un réel intérêt pour les administrateurs systèmes et les DevOps lorsqu'ils produisent des scripts, gèrent de la documentation ou interviennent sur un système d'exploitation. 

Il existe depuis des années de nombreux outils permettant de répondre à ce besoin comme SVN, CVS, Bazaar, ou GIT. GIT est l'outil qui est le plus utilisé actuellement.

## 2. Retour sur l'histoire de GIT

Comme de nombreuses choses extraordinaires de la vie, Git est né avec une dose de destruction créative et de controverse houleuse.

Le noyau Linux est un projet libre de grande envergure. Pour la plus grande partie de sa vie (1991–2002), les modifications étaient transmises sous forme de patchs et d’archives de fichiers. En 2002, le projet du noyau Linux commença à utiliser un DVCS propriétaire appelé BitKeeper.

En 2005, les relations entre la communauté développant le noyau Linux et la société en charge du développement de BitKeeper furent rompues, et le statut de gratuité de l’outil fut révoqué. Cela poussa Linus Torvalds (le créateur du noyau Linux) à développer son propre outil en se basant sur les leçons apprises lors de l’utilisation de BitKeeper.

**Torvalds a écrit Git en seulement 10 jours !** La conception non conventionnelle et décentralisée de Git, aujourd'hui omniprésente et apparemment évidente, était révolutionnaire à l'époque et a modifié la manière dont les équipes logicielles collaborent et développent.

## 3. Le vocabulaire autour de GIT et du versioning

* **Commentaire**: Métadonnée qui permet à l'auteur d'un commit de documenter le contenu de la révision lors de son ajout à l'historique.

* **Commit**: Cette opération consiste à indiquer à Git que l'ensemble des changements effectués sur les sources du code doit être ajouté à l'historique.

* **Historique**: Ensemble de révisions de la première à la plus récente.

* **Révision**: série de modifications au sein d'un code source d'un projet (tout fichier associé au projet).

