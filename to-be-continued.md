# feat: playbook git de init à push, et plus encore

## TODO v2

### Préparer son environnement de travail

    Configurations globale et locale
    Aliases et raccourcis incontournables
    Informations d’état automatiques dans le prompt/terminal

### Concepts de la gestion de dépôt Git

    Les zones et états, mention spéciale à la zone “stage”
    Aperçu du dossier .git
    La notion critique de HEAD

### Premiers pas

    Initialiser un dépôt : init ou clone
    Exclure des fichiers de la gestion de version : .gitignore
    Ajout des premiers fichiers
    Savoir analyser l’état courant ou des versions passées : status, diff, log, show

### Gérer les commits

    Ajouts partiels (seulement certains fichiers ou certaines parties de fichiers)
    Annuler les ajouts “unstaging”
    Annuler le dernier commit
    Modifier le dernier commit
    Modifier un commit plus ancien
    Raccourcis fréquents et pratiques

### Gérer l’urgence avec le stash

    Mettre son travail de côté le temps d’une tâche urgente
    Récupérer le stash (multiples façons)
    Aspect pratique face au conservatisme de Git lors d’une demande de fusion
    Stash vs worktrees

### Gestion de branches

    Les branches : de simples étiquettes
    Fusion classique
    Fusion “à plat” : le fast forward
    Résoudre les conflits
    Complémentarité de la fusion et du Rebasing

### Git reset, savoir défaire et refaire

    Un reset, qu’est-ce que c’est ?
    Les 5 modes de git reset
    Scenarii classiques où reset nous sauve la vie
    Resets irréparables ou non
    Récupérer un commit « perdu » avec le reflog

### Prendre soin de son historique de révisions : le rebasing

    Un historique au cordeau : quels intérêts ?
    Réordonner les commits
    Supprimer des commits
    Découper des commits
    Fusionner des commits
    Annuler un commit ancien

### Dépôts distants

    Définir, lister les remotes avec la commande remote
    Intéractions classiques : fetch/pull, push
    Bien configurer le pull pour un historique “propre”
    Corriger un historique partagé : forcer le push, mais correctement !

### Récupérer des portions choisies de l’historique

    D’une branche à une autre avec le cherry-picking
    D’un projet à un autre avec les patches

### Automatiser des traitements sur événements

    Les hooks locaux et coté serveur
    Scenarii classiques d’utilisation

### Bien penser son workflow

    Paralléliser les tâches
    Gérer les corrections
    Réflexions sur l’automatisation
    Connaître les conventions : semantic versioning, conventional commit.
