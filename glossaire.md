# Glossaire du chapitre git

Les termes employés dans les fiches de `10-git/`, avec le terme anglais correspondant et, quand il en existe un, l'équivalent sur un tableau de tickets.

Le terme anglais n'est pas décoratif : les interfaces de forge et les messages de git sont en anglais, sans traduction disponible. C'est le mot affiché à l'écran qui sert à retrouver une notion, pas sa traduction.

Ce glossaire ne se lit pas d'un bout à l'autre. Il se consulte quand un mot d'une fiche résiste.

## Les objets

**Commit** (*commit*). État complet du projet à un instant, accompagné d'un auteur, d'une date, d'un message, et d'un pointeur vers l'état qui le précède. Ce n'est pas une liste de modifications : la différence affichée par les outils est calculée à la lecture, entre deux états.
*Équivalent : une mise à jour de ticket, avec son commentaire.*

**Branche** (*branch*). Nom qui désigne un commit, et qui se déplace vers le nouveau commit à chaque fois qu'un commit est créé. Ce n'est ni un dossier, ni une copie du projet, mais une ligne de texte contenant un identifiant. D'où le fait que sa création soit instantanée et sa suppression sans perte.
*Équivalent : un ticket en cours.*

**Branche par défaut** (*default branch*). Branche de référence du dépôt, `main` par convention. Elle porte l'état considéré comme acquis.
*Équivalent : la colonne « Terminé ».*

**`HEAD`**. Étiquette indiquant la position courante, donc quelle branche avancera au prochain commit. Pas de traduction, le mot est le nom de l'objet.

**Dépôt** (*repository*, abrégé *repo*). L'ensemble des états enregistrés, plus les étiquettes qui les désignent.

**Clone** (*clone*). Exemplaire local d'un dépôt. Un clone contient tout l'historique, pas seulement l'état courant.

**Forge** (pas de terme anglais unique, on nomme la plateforme). Service hébergeant le dépôt partagé et les demandes de fusion. GitHub, GitLab, Framagit.
*Équivalent : l'outil qui héberge le tableau.*

**`origin`**. Nom conventionnel du dépôt distant de référence (*remote*). `origin/main` désigne l'état de `main` sur la forge **tel que le clone local le connaissait à la dernière récupération**, pas son état courant.

**Demande de fusion** (*pull request* sur GitHub, abrégé *PR* ; *merge request* sur GitLab, abrégé *MR*). Proposition d'intégrer une branche dans une autre, ouverte à la relecture. Les deux noms désignent la même chose.
*Équivalent : un ticket soumis à relecture.*

**Brouillon** (*draft*). État d'une demande de fusion signalant qu'elle n'est pas prête à être intégrée. Une demande en brouillon reste invisible dans les listes filtrées par défaut, ce qui explique qu'un décompte de demandes ouvertes puisse varier selon le filtre employé.

## Les zones

**Copie de travail** (*working tree*, ou *working directory*). Les fichiers tels qu'ils sont sur le disque.

**Zone de préparation** (*staging area*, ou *index*). Ce qui entrera dans le prochain commit. Les interfaces affichent *staged* et *unstaged*. Un fichier modifié mais non ajouté n'entre pas dans le commit, et rien ne le signale.

**Suivi, non suivi** (*tracked*, *untracked*). Un fichier suivi est connu du dépôt. Une règle d'exclusion ne s'applique qu'aux fichiers non suivis, ce qui explique qu'ajouter une règle après coup ne retire rien.

## Les opérations

**Récupérer** (*fetch*). Met à jour la connaissance qu'a le clone local de l'état de la forge. N'affecte pas le travail en cours. Sans cette opération, toute décision se prend sur une photographie périmée.

**Fusionner** (*merge*). Crée un état ayant deux prédécesseurs. Rien n'est réécrit, les deux histoires restent lisibles. Sur GitHub, l'option correspondante est *Create a merge commit*.
*Équivalent : faire passer un ticket en « Terminé ».*

**Rebaser** (*rebase*). Rejoue les commits d'une branche au-dessus d'une autre. Produit un historique linéaire, au prix de nouveaux états : toute personne ayant déjà récupéré la branche possède désormais une version qui ne correspond plus à rien. Option *Rebase and merge*.

**Écraser en un seul état** (*squash*). Transforme tout le travail d'une branche en un commit unique. L'historique de destination reste lisible, le détail interne disparaît. Option *Squash and merge*.

**Publier** (*push*). Envoie les états locaux vers la forge.

**Publier en forçant** (*force push*). Réécrit l'historique distant et supprime ce que d'autres avaient récupéré. Sur la branche par défaut, ne se justifie pratiquement jamais.

**Fermer** (*close*). Clôt une demande de fusion sans l'intégrer. À distinguer de *merged*, qui signale qu'elle l'a été.

## Les mesures

**Avance** (*ahead*). Nombre de commits qu'une branche possède et que la branche de référence n'a pas. C'est ce qu'elle apporterait.

**Retard** (*behind*). Nombre de commits que la branche de référence possède et que la branche n'a pas. Ce n'est pas un défaut en soi : une branche en retard peut très bien apporter quelque chose.

**Avance nulle** (*0 ahead*, ou *merged* dans la liste des branches). La branche n'a rien à apporter. Elle se ferme, elle ne se fusionne pas.

**Point de séparation** (*merge base*, ou *common ancestor*). Le dernier état commun à deux branches. C'est le fait auquel se compare la destination affichée par une demande de fusion.

**Destination affichée** (*base branch*, affichée simplement *base* dans le sélecteur d'une demande de fusion). Branche vers laquelle une demande annonce qu'elle sera intégrée. Saisie par une personne, donc déclarative. Elle coïncide avec le point de séparation quand la branche est effectivement partie de là, et pas autrement.

**Empilement** (*stacked branches*, ou *stacked pull requests*). Situation où une branche est assise sur une autre branche non encore intégrée. La branche du haut contient tout ce que contient celle du bas.
*Équivalent : une chaîne de tickets bloqués les uns par les autres, à ceci près que le blocage n'est déclaré nulle part et se calcule.*

**Conflit** (*merge conflict*). Deux branches ont modifié les mêmes lignes d'un même fichier, et git ne choisit pas. L'absence de conflit ne signifie pas que la fusion est bonne, seulement qu'aucun arbitrage textuel n'est requis.

**Chaîne de contrôles automatiques** (*checks*, ou *CI*). Suite de vérifications exécutées à chaque envoi. Une chaîne verte établit qu'un apport n'a pas cassé ce qui est vérifié, pas qu'il est bon.

## Les notations

**`A..B`, dans `git log`.** Les commits accessibles depuis `B` et pas depuis `A`, c'est-à-dire ce que `B` apporte.

**`A...B`, dans `git diff`.** Une abréviation de `git diff --merge-base A B`, soit la comparaison entre le point de séparation et `B`. La notation ne veut donc pas dire la même chose selon la commande, ce qui justifie de préférer la forme explicite.

## Ce qui n'a pas d'équivalent sur un tableau de tickets

Trois notions n'ont aucune contrepartie, et ce sont exactement celles qui produisent des surprises.

**La limite de travail en cours** (*WIP limit*). Un tableau l'affiche et la fait respecter. Git ne connaît pas la notion et n'opposera jamais aucune résistance à l'ouverture d'une trentième branche.

**La dépendance non déclarée.** Sur un tableau, un blocage est saisi dans un champ et s'affiche. En git, il existe sans être déclaré, et ne se lit sur aucune interface.

**L'effet de bord d'une clôture.** Fermer un ticket ne ferme rien d'autre. Fusionner une branche peut en intégrer onze, sans avertissement.

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
  version: 0.02.000
  versioning: pridever/0.2.0 (https://pridever.org/)
  demarque: false
epistem:end -->
