# git, kézako ?

Ce chapitre répond à une seule question, déclinée en six fiches : comment tenir l'historique d'un projet de façon qu'il reste lisible et réversible, y compris quand plusieurs chantiers avancent en même temps.

## À qui il s'adresse

À une personne qui conduit des projets, connaît le numérique, sait ce qu'est un ticket, une revue et une colonne « Terminé », et découvre le développement logiciel.

Aucune connaissance préalable de git n'est supposée. La première fiche part du modèle mental et s'appuie sur la correspondance avec un tableau de tickets, qui est bonne partout sauf à deux endroits, lesquels sont précisément ceux qui produisent des surprises.

## Table des matières

| Fiche | Question à laquelle elle répond | État |
| --- | --- | --- |
| [`modele-mental.md`](modele-mental.md) | Ce que git manipule réellement, et l'équivalent côté tableau de tickets | rédigée |
| [`premier-depot.md`](premier-depot.md) | Ce qu'il faut poser dans un dépôt avant le premier commit, et vérifier avant le premier envoi | rédigée |
| [`messages-de-commit.md`](messages-de-commit.md) | Ce qu'un message de commit doit contenir, et pour quel lecteur | à rédiger |
| [`branches-et-integration.md`](branches-et-integration.md) | Comment un travail revient dans la branche principale, et pourquoi l'ordre n'est pas libre | rédigée |
| [`plusieurs-chantiers-en-parallele.md`](plusieurs-chantiers-en-parallele.md) | Comment organiser un dépôt quand plusieurs fonctionnalités avancent en même temps | rédigée |
| [`incident-secret-commite.md`](incident-secret-commite.md) | La conduite à tenir quand un secret est entré dans l'historique | à rédiger |

[`glossaire.md`](glossaire.md) hors ordre de lecture : il se consulte quand un mot d'une fiche résiste, il ne se lit pas d'un bout à l'autre.

## Les trois idées qui portent le chapitre

Elles sont établies dans `modele-mental.md` et reprises partout ensuite sans être redémontrées.

**Un commit est un état, pas une modification.** Git enregistre une photographie complète du projet, pas la liste de ce qui a changé. La différence affichée par les outils est un calcul fait à la lecture.

**Une branche est une étiquette, pas un dossier.** Un nom qui désigne un état, et qui se déplace à chaque nouveau commit. D'où le fait que créer une branche soit instantané, et que la supprimer ne supprime aucun travail.

**Une branche contient tout ce qui se trouve en dessous d'elle.** C'est la conséquence des deux premières, et la seule qui produise des surprises coûteuses : intégrer une branche haute intègre du même coup tout le travail situé en dessous, y compris ce que personne n'a relu, sans qu'aucun message ne le signale.

## Ce que le chapitre ne contient pas

La configuration de l'hébergeur de dépôt, qui relève de `../80-deploiement/`.

La chaîne d'intégration continue et ce qu'elle vérifie, qui relèvent de `../50-integration-continue/`. Le chapitre 10 s'arrête à ce qui entre dans l'historique et à l'ordre dans lequel il y entre.

La gestion des secrets elle-même, qui relève de `../20-environnement/`. La fiche `incident-secret-commite.md` traite du rattrapage, pas de la prévention.

Aucune situation réelle, aucun nom de branche, de personne ou de dépôt du projet en cours. Les exemples sont fictifs, conformément à [`../CONVENTIONS.md`](../CONVENTIONS.md).

## Convention de vocabulaire

Le français est employé dans le texte, avec le terme anglais entre parenthèses à la première occurrence de chaque fiche : demande de fusion (*pull request*), point de séparation (*merge base*), avance et retard (*ahead*, *behind*).

Le motif est pratique et non stylistique. Les interfaces de forge et les messages de git sont en anglais, sans traduction disponible. Un lecteur qui ne connaît que « avance » ne retrouve pas la colonne « Ahead » de son écran, et une notion qu'on ne sait pas nommer dans la langue de l'outil ne se cherche pas.

Le terme anglais est conservé seul quand il désigne une commande ou un objet sans nom français, et il apparaît alors en `police à chasse fixe` : `merge-base`, `HEAD`, `origin`.

Les commandes ne sont jamais données seules. Chacune est précédée de la question à laquelle elle répond, parce qu'une commande retenue sans sa question se rejoue ensuite dans des situations où elle ne s'applique pas.

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
