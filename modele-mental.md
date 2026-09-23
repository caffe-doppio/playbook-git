# Modèle mental

Cette fiche répond à la question de ce que git manipule réellement, et de ce à quoi ces objets correspondent pour une personne habituée à conduire des projets sur un tableau de tickets.

> [!TIP]
> **TL;DR** Git ne conserve pas des modifications, il conserve des états successifs du projet, et une branche n'est rien d'autre qu'une étiquette posée sur l'un de ces états.

## Le problème

Git s'apprend le plus souvent par ses commandes : `add`, `commit`, `push`, `merge`. La séquence fonctionne tant que rien ne dévie. Le jour où elle dévie, les commandes ne suffisent plus, parce qu'elles décrivent des gestes et non ce sur quoi ces gestes agissent.

Trois symptômes signalent qu'un modèle mental manque.

**Le vocabulaire ne veut plus rien dire.** « Cette branche est en avance de onze commits », « la base de la demande de fusion n'est pas son parent réel », « ce commit est déjà contenu dans la branche principale ». Chacune de ces phrases est exacte et vérifiable. Aucune n'est interprétable sans savoir ce qu'est un commit et ce qu'est une branche.

**Les gestes deviennent superstitieux.** La séquence qui a fonctionné la fois précédente est rejouée à l'identique, y compris quand la situation diffère. Quand elle échoue, la réaction courante consiste à créer une copie de sauvegarde du dossier, ou une branche nommée `backup`, pour ne rien perdre. Le travail est préservé, mais le dépôt gagne une divergence de plus.

**Les décisions se prennent sur la mauvaise information.** Une demande de fusion annonce un titre et une branche de destination. Ces deux éléments sont saisis par une personne. Ce que la fusion apporte réellement est un fait de l'historique, qui peut différer. Sans modèle mental, rien ne signale que l'écart est possible, donc rien ne conduit à le vérifier.

Ces trois symptômes ont la même cause : les commandes ont été apprises, la structure sur laquelle elles opèrent ne l'a pas été.

## Les solutions

Trois représentations de git circulent. Elles ne se valent pas.

### La représentation par le dossier de sauvegardes

Git est vu comme un système qui range des copies datées du projet, et une branche comme un dossier contenant une variante.

Cette représentation a le mérite d'être immédiate. Elle se casse sur trois points, et vite : elle n'explique pas pourquoi changer de branche ne change pas de répertoire, ni pourquoi une branche peut « contenir » une autre, ni pourquoi la suppression d'une branche ne supprime pas le travail.

### La représentation par la liste de modifications

Git est vu comme un journal de différences successives, chaque commit stockant ce qui a changé.

Elle est plus proche de ce que l'utilisateur observe, puisque les outils affichent effectivement des différences. Elle reste fausse sur le fond, et surtout elle rend inexplicable le comportement qui compte le plus : la façon dont un travail se retrouve présent dans une branche sans que personne ait décidé de l'y mettre.

### La représentation par la chaîne d'états

Git est vu comme une suite d'états complets du projet, chacun désignant celui qui le précède.

C'est la représentation exacte. Un commit enregistre un instantané de l'ensemble des fichiers, et contient un pointeur vers le ou les commits qui le précèdent immédiatement : aucun pour le premier commit, un pour un commit ordinaire, plusieurs pour un commit issu d'une fusion. Source : Pro Git, chapitre 3.1, <https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell>, consulté le 8 septembre 2026.

Elle demande un effort de départ un peu plus grand que les deux autres, et elle est la seule dont tout le reste découle sans exception à retenir.

## La solution retenue dans ce contexte

La troisième. Elle tient en quatre objets et une conséquence.

### Le commit est un état, pas une modification

Un commit est une photographie de l'ensemble des fichiers à un instant, accompagnée d'un auteur, d'une date, d'un message, et d'un pointeur vers l'état précédent.

L'affichage sous forme de différence est un calcul fait à la demande, entre deux états. C'est une commodité de lecture, pas ce qui est stocké.

Conséquence immédiate : l'historique est une chaîne. Chaque maillon désigne le précédent. Un maillon ne se retire pas du milieu sans reconstruire tous ceux qui se trouvent au-dessus, ce qui explique pourquoi les manipulations d'historique sont coûteuses et pourquoi un secret commité ne s'efface pas par une suppression au commit suivant.

### La branche est une étiquette

Une branche (*branch*) est un nom qui désigne un commit, et qui se déplace vers le nouveau commit à chaque fois qu'un commit est créé.

Ce n'est pas un dossier. Ce n'est pas une copie du projet. C'est une ligne de texte contenant un identifiant.

Trois comportements, opaques dans les autres représentations, en découlent directement :

- créer une branche est instantané, quelle que soit la taille du projet, puisque l'opération écrit un identifiant ;
- supprimer une branche ne supprime pas les commits, elle retire l'étiquette, et les commits deviennent simplement plus difficiles à retrouver ;
- deux branches peuvent désigner le même état, auquel cas elles ne se distinguent en rien.

### `HEAD` désigne la position courante

`HEAD` est l'étiquette qui indique sur quelle branche le travail se fait, donc quelle étiquette avancera au prochain commit.

Changer de branche revient à déplacer `HEAD`, puis à remettre le répertoire de travail dans l'état que la branche désigne. Le répertoire reste le même, son contenu change.

### Trois zones, pas une

Un fichier modifié traverse trois zones avant d'entrer dans l'historique. Source : Pro Git, chapitre 1.3, <https://git-scm.com/book/en/v2>, consulté le 8 septembre 2026.

| Zone | Ce qu'elle contient | Commande qui fait entrer |
| --- | --- | --- |
| copie de travail (*working tree*) | les fichiers tels qu'ils sont sur le disque | l'éditeur de texte |
| zone de préparation (*staging area*, ou *index*) | ce qui entrera dans le prochain commit | `git add` |
| dépôt (*repository*) | les états déjà enregistrés, définitifs | `git commit` |

L'index est la zone dont l'existence surprend, et c'est celle qui explique le malentendu le plus fréquent : un commit ne contient pas ce qui a été modifié, il contient ce qui a été ajouté à l'index. Un fichier modifié mais non ajouté n'entre pas dans le commit, sans qu'aucune erreur ne soit signalée.

### La conséquence qui compte

Puisque chaque état désigne son prédécesseur, et puisqu'une branche désigne un état, **une branche contient tout ce qui se trouve en dessous d'elle dans la chaîne.**

C'est de là que vient le comportement le plus contre-intuitif de git, et le seul qui produise des surprises coûteuses : intégrer une branche située haut dans une chaîne intègre du même coup tout le travail situé en dessous, y compris ce que personne n'a relu, et sans qu'aucun message ne le signale.

> [!WARNING]
> Le titre d'une demande de fusion et la branche de destination qu'elle affiche sont des déclarations saisies par une personne. Ce que la fusion apporte réellement est un fait de l'historique. Les deux coïncident quand la branche est effectivement partie de la destination annoncée, et pas autrement. Seule la commande `git merge-base` établit le fait.

## Correspondance avec un tableau de tickets

La conduite de projet sur un tableau de tickets fournit un modèle déjà maîtrisé, et la correspondance est bonne sur presque toute la ligne.

| Sur un tableau de tickets | En git |
| --- | --- |
| un ticket en cours | une branche (*branch*) |
| une mise à jour du travail sur le ticket | un commit (*commit*) |
| le commentaire qui explique la mise à jour | le message de commit |
| un ticket soumis à relecture | une demande de fusion (*pull request*, ou *merge request*) |
| la colonne « Terminé » | la branche par défaut (*default branch*), `main` |
| faire passer un ticket en « Terminé » | fusionner (*merge*) la branche dans `main` |
| un ticket bloqué par un autre ticket | une branche assise sur une autre branche |
| une chaîne de tickets bloqués les uns par les autres | un empilement de branches (*stacked branches*) |
| le tableau lui-même, vu par tous | le dépôt distant (*remote*), sur la forge |
| l'exemplaire de travail sur le poste | le clone local |

> [!NOTE]
> **Deux endroits où la correspondance ne tient pas.** Ce sont exactement les deux endroits où un tableau de tickets ne prépare pas à ce qui va se passer.
>
> **Un ticket est indépendant, une branche ne l'est pas.** Deux tickets peuvent être clos dans n'importe quel ordre. Deux branches empilées ne le peuvent pas : intégrer celle du dessus intègre celle du dessous, qu'on le veuille ou non.
>
> **Fermer un ticket ne ferme rien d'autre, fusionner une branche peut en fermer plusieurs.** Sur le tableau, un blocage se lit dans un champ dédié et se constate. En git, l'empilement ne se lit nulle part sur l'interface de la demande de fusion, il se calcule.

## Pièges courants

**Croire qu'une branche est une copie du projet.** Elle est une étiquette. Le répertoire de travail est unique et son contenu est remplacé au changement de branche.

**Croire qu'un commit enregistre une différence.** Il enregistre un état. La différence est calculée à l'affichage.

**Croire que supprimer une branche supprime le travail.** L'étiquette part, les commits restent. La difficulté est inverse de ce qui est redouté : ce n'est pas la perte, c'est le fait de retrouver un état devenu anonyme.

**Confondre « la branche est à jour » et « le travail est dans `main` ».** Une branche à jour a reçu ce que `main` contient. L'inverse n'est pas vrai, et c'est l'inverse qui compte pour livrer.

**Oublier que `origin/main` est une copie datée.** La branche distante telle que le clone local la connaît reflète l'état constaté au dernier `git fetch`, pas l'état courant de la forge. Une décision prise sans `fetch` préalable est prise sur une photographie périmée.

**Créer une branche de sauvegarde pour ne rien perdre.** Le réflexe est compréhensible et le résultat est durablement encombrant : une branche nommée à la main, datée, que plus personne n'ose fermer parce que plus personne ne sait ce qu'elle contient de propre.

**Confondre l'index avec la copie de travail.** Un fichier modifié et non ajouté n'entre pas dans le commit. Rien ne le signale au moment du commit.

## Pour aller plus loin

- Pro Git, chapitre 1.3 « What is Git? », pour les trois zones et le principe d'instantané : <https://git-scm.com/book/en/v2>. Consulté le 8 septembre 2026.
- Pro Git, chapitre 3.1 « Branches in a Nutshell », pour la nature du commit et de la branche : <https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell>. Consulté le 8 septembre 2026.
- Les termes anglais correspondants sont réunis dans `glossaire.md`, dans le même répertoire. Ce sont eux qui s'affichent sur les interfaces de forge.
- Ce qu'il faut poser dans un dépôt avant le premier commit fait l'objet de la fiche `premier-depot.md`, dans le même répertoire.
- La conduite à tenir quand plusieurs branches s'empilent fait l'objet de la fiche `branches-et-integration.md`, dans le même répertoire.

<!-- epistem:begin
convention: epistem/1.2.0
legend:
  nature : 🔗 inférence · 💭 hypothèse · 💬 non vérifié · ➖ non vérifiable
  cycle  : 🔜 à vérifier · 🔢 à chiffrer · ✔️ confirmé · ✖️ infirmé
ai:
  assisted: true
  model: claude-opus-5
  scope: rédaction
  framework: AI Fluency 4D (Delegation, Description, Discernment, Diligence)
  accountability: caffe-doppio
doc:
  version: 0.01.000
  versioning: pridever/0.2.0 (https://pridever.org/)
  demarque: false
epistem:end -->
