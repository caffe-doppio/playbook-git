# Branches et intégration

Cette fiche répond à la question de la façon dont un travail mené à part revient dans la branche principale, et de ce qui décide de l'ordre quand plusieurs travaux attendent en même temps.

> [!TIP]
> **TL;DR** Ce qu'une fusion apporte ne se lit pas sur le titre de la demande ni sur la branche de destination qu'elle affiche, mais se calcule à partir du point de séparation réel entre les deux branches.

## Le problème

Faire revenir une branche dans la branche principale est trivial tant qu'il y en a une seule. La difficulté apparaît au moment où plusieurs branches attendent, et elle prend trois formes.

### La fusion livre plus que son titre n'annonce

Une demande de fusion (*pull request* sur GitHub, *merge request* sur GitLab) affiche deux choses : un titre, saisi par une personne, et une branche de destination (*base branch*, affichée simplement *base*), choisie par une personne. Ni l'un ni l'autre n'est un fait de l'historique.

Quand la branche a effectivement été créée à partir de la destination annoncée, les deux coïncident et la déclaration décrit la réalité. Quand la branche a été créée à partir d'une autre branche, ce qui arrive dès que le travail s'enchaîne, la destination annoncée reste affichable et devient fausse.

Le cas se présente ainsi : une demande intitulée « simplifier le formulaire de saisie », annonçant la branche par défaut comme destination, portant onze commits. La fusionner ne livre pas un formulaire simplifié, elle livre le formulaire simplifié plus les dix travaux sur lesquels il est assis, dont aucun n'a été relu à cette occasion.

> [!CAUTION]
> Ce cas ne produit aucune erreur, aucun conflit, aucun avertissement. La fusion réussit. C'est ce qui le rend coûteux : rien ne signale que dix travaux viennent d'entrer en production sans relecture.

### L'ordre décide de ce qui est relu

Quand des branches sont empilées, chacune contient les précédentes. Le sens dans lequel la pile est vidée décide donc de ce qui passe sous les yeux d'un relecteur, et de ce qui passe sans être vu.

Intégrer d'abord la branche du bas produit un premier apport complet, puis des apports de plus en plus petits, chacun étant la seule différence avec l'état précédent. Intégrer d'abord la branche du haut produit un unique apport contenant tout, après quoi les branches du dessous n'apportent plus rien et paraissent, à juste titre, déjà intégrées.

### La suppression d'une branche recible ce qui s'appuyait dessus

Quand une branche qui sert de destination à d'autres demandes est supprimée, les forges reciblent automatiquement ces demandes vers une autre destination. 💬 Comportement constaté, à confirmer sur la documentation de la forge employée.

Sur une pile de demandes qui se visent les unes les autres, ce reciblage se propage. L'ordre déclaré cesse alors de correspondre à l'ordre réel, sans notification et sans trace lisible.

## Les solutions

Trois opérations font revenir un travail dans une autre branche. Elles ne produisent pas le même historique et ne s'emploient pas dans les mêmes conditions.

### La fusion

Un nouvel état est créé (*merge*, option *Create a merge commit* sur GitHub), qui a deux prédécesseurs : l'état de la branche de destination et celui de la branche intégrée. Rien n'est réécrit, les deux histoires sont conservées et le point de jonction est visible.

C'est la seule des trois qui préserve l'information « ce travail a été mené à part, puis intégré à cette date ».

Contrepartie : sur un dépôt où beaucoup de branches courtes s'intègrent, le graphe devient dense et se lit mal sans outil.

### Le rebasage

Les commits de la branche sont rejoués un par un au-dessus de la destination (*rebase*, option *Rebase and merge*). L'historique obtenu est linéaire, et se lit comme si le travail avait été mené directement sur la destination.

Les états rejoués sont de nouveaux états. Les anciens existent toujours, mais plus rien ne les désigne. Toute personne ayant déjà récupéré la branche possède désormais une version qui ne correspond plus à rien.

Condition d'emploi : la branche n'a pas été publiée, ou elle l'a été et personne d'autre ne travaille dessus. Hors de cette condition, l'opération transfère le problème à quelqu'un d'autre.

### L'écrasement en un seul état

L'ensemble du travail de la branche devient un état unique posé sur la destination (*squash*, option *Squash and merge*). L'historique de la destination reste très lisible, une ligne par fonctionnalité.

Contrepartie : le détail disparaît. La possibilité de retrouver le commit précis qui a introduit un défaut disparaît avec lui, ce qui est exactement l'information dont on a besoin le jour où l'on cherche un défaut.

Condition d'emploi raisonnable : une branche dont l'historique interne n'a pas de valeur, typiquement une suite de commits d'essai.

## La solution retenue dans ce contexte

Contexte visé : un dépôt tenu par une ou deux personnes, avec des branches publiées, éventuellement empilées, et un besoin de traçabilité sur ce qui est entré et quand.

**Fusion sans réécriture, une branche à la fois, du bas de la pile vers le haut, avec vérification du point de séparation avant chaque intégration.**

Trois motifs.

Le rebasage et l'écrasement sont écartés parce que les branches sont publiées et que l'historique détaillé a de la valeur sur un projet dont la personne qui l'écrit apprend en même temps qu'elle l'écrit.

Le sens bas vers haut est retenu parce qu'il est le seul où chaque apport reste à taille relisible. Il n'a pas d'autre avantage : il est plus lent, et c'est le prix de la relecture.

La vérification préalable est retenue parce qu'elle est le seul moyen de savoir ce qu'une fusion apporte. Elle coûte deux commandes.

### Vérifier avant d'intégrer

La question est toujours la même : qu'est-ce que cette fusion apporte réellement.

```bash
# 1. Se donner un état à jour du dépôt distant.
#    Sans cette étape, les commandes suivantes répondent sur une photographie périmée.
git fetch origin --prune

BRANCHE=<nom-de-la-branche>

# 2. Où les deux branches se sont-elles séparées ?
#    C'est le point de séparation, le merge base. C'est un fait, par opposition
#    à la destination affichée par la demande.
git merge-base origin/main "origin/$BRANCHE"

# 3. Combien de commits la fusion apporterait-elle ?
git log --oneline "origin/main..origin/$BRANCHE" | wc -l

# 4. Quels fichiers, et quel volume ?
git diff --stat --merge-base origin/main "origin/$BRANCHE"
```

Le résultat de l'étape 3 se compare au titre de la demande. Un titre qui annonce une modification de formulaire et un décompte à onze commits ne sont pas conciliables : la demande porte autre chose que ce qu'elle dit.

> [!IMPORTANT]
> **Les deux points et les trois points ne veulent pas dire la même chose, et pas la même chose selon la commande.**
>
> Pour `git log`, `A..B` désigne les commits accessibles depuis `B` et pas depuis `A`, c'est-à-dire ce que `B` apporte. Source : Pro Git, chapitre 7.1, section sur les intervalles de commits, <https://git-scm.com/book/en/v2/Git-Tools-Revision-Selection>, consulté le 8 septembre 2026.
>
> Pour `git diff`, la notation à trois points ne désigne pas un intervalle : `git diff A...B` est une abréviation de `git diff --merge-base A B`, soit la comparaison entre le point de séparation et `B`. Source : documentation git, `gitrevisions`, <https://git-scm.com/docs/gitrevisions>, consultée le 8 septembre 2026.
>
> La forme explicite `--merge-base` est préférée dans cette fiche : elle dit ce qu'elle fait, et elle ne repose pas sur une notation dont la dépréciation est discutée en amont. 💭 Dépréciation évoquée dans les discussions du projet git, sans version d'effet arrêtée à ce jour.

### Savoir ce qui est déjà intégré

Deux questions reviennent en fin de chantier, et deux commandes y répondent.

```bash
# Quelles branches sont entièrement contenues dans main, donc fermables sans perte ?
git branch -r --merged origin/main

# Quelles branches contiennent tel commit précis ?
#    Sert à établir qu'un travail présent en double a bien été intégré par ailleurs.
git branch -r --contains <identifiant-du-commit>
```

Une branche entièrement contenue dans la branche par défaut n'a rien à lui apporter. La forge l'affiche avec une avance nulle, *0 ahead*, ou l'étiquette *merged* dans la liste des branches. Elle se ferme, elle ne se fusionne pas.

> [!WARNING]
> Avant de fermer sur ce critère, vérifier que l'absence d'apport signifie « déjà intégré » et non « travail perdu par une réécriture d'historique ». La première commande ci-dessus établit la première hypothèse ; elle n'écarte pas la seconde à elle seule.

### L'ordre d'intégration se déduit, il ne se choisit pas

Sur une pile, l'ordre n'est pas une préférence de méthode. Il est donné par la structure : chaque branche ne peut être intégrée qu'après celle sur laquelle elle est assise, sauf à intégrer les deux d'un coup.

La commande qui donne la forme de la pile :

```bash
git log --graph --oneline --decorate --simplify-by-decoration --all
```

Une seule colonne signifie une pile. Plusieurs colonnes signifient des travaux réellement parallèles, dont l'ordre, lui, est libre.

## Correspondance avec un tableau de tickets

Un travail assis sur un autre travail correspond exactement à un ticket bloqué par un autre ticket. La différence tient à ce qui est visible.

Sur un tableau, le blocage est déclaré dans un champ, il s'affiche, et le tableau refuse en général de fermer un ticket bloquant sans le signaler.

En git, le blocage n'est pas déclaré, il existe. Il ne s'affiche sur aucune interface de demande de fusion. Il se calcule, avec la commande de l'étape 2 ci-dessus.

> [!NOTE]
> C'est la seule chose à retenir de cette comparaison : **sur un tableau de tickets, une dépendance est une déclaration ; en git, c'est un fait, et un fait qui ne s'affiche pas.**

## Pièges courants

**Prendre la branche de destination affichée pour la branche de départ réelle.** La première est saisie, la seconde se calcule. Elles coïncident souvent, et l'écart n'est jamais signalé.

**Intégrer par le haut pour aller plus vite.** Cela fonctionne, au sens où rien ne casse. Tout ce qui se trouvait en dessous entre sans relecture, et devient ensuite indistinguable de ce qui a été relu.

**Supprimer une branche pendant que le chantier est en cours.** Les demandes qui la visaient sont reciblées, et l'ordre déclaré se désaligne de l'ordre réel. La suppression est une opération de fin de chantier.

**Regrouper deux branches dans une seule intégration pour gagner du temps.** L'apport cesse d'être attribuable à un chantier, et le retour arrière cesse d'être possible chantier par chantier.

**Rebaser une branche que quelqu'un d'autre a récupérée.** L'opération réussit chez celui qui la lance et casse chez tous les autres.

**Décider sans avoir récupéré l'état distant.** Sans `git fetch` préalable, toutes les commandes ci-dessus répondent sur l'état constaté à la dernière récupération.

**Traiter une chaîne verte comme une preuve.** Une chaîne d'intégration continue vérifie ce qu'elle a été écrite pour vérifier. Elle établit qu'un apport n'a pas cassé ce périmètre, pas qu'il est bon. Ce que le périmètre couvre relève de `../50-integration-continue/`.

## Pour aller plus loin

- Pro Git, chapitre 3.2 « Basic Branching and Merging », pour le déroulement d'une fusion : <https://git-scm.com/book/en/v2>. Consulté le 8 septembre 2026.
- Pro Git, chapitre 3.6 « Rebasing », dont la section sur les dangers du rebasage d'une branche publiée : <https://git-scm.com/book/en/v2>. Consulté le 8 septembre 2026.
- Pro Git, chapitre 7.1 « Revision Selection », pour les intervalles de commits : <https://git-scm.com/book/en/v2/Git-Tools-Revision-Selection>. Consulté le 8 septembre 2026.
- Documentation git, `gitrevisions`, pour les notations à deux et trois points : <https://git-scm.com/docs/gitrevisions>. Consultée le 8 septembre 2026.
- Les termes anglais correspondants sont réunis dans `glossaire.md`, dans le même répertoire.
- La façon d'éviter que les piles se forment fait l'objet de la fiche `plusieurs-chantiers-en-parallele.md`, dans le même répertoire.

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
