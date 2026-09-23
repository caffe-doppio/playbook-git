# Plusieurs chantiers en parallèle

Cette fiche répond à la question de la façon d'organiser un dépôt quand plusieurs fonctionnalités avancent en même temps, et de ce qui décide du nombre de chantiers qu'il est raisonnable d'ouvrir.

> [!TIP]
> **En une phrase.** Le nombre de branches ouvertes n'est pas une question de git, c'est une limite de travail en cours, et un dépôt qui accumule les branches est un tableau de tickets sans colonne « Terminé ».

## Le problème

Ouvrir une branche est instantané et gratuit. La refermer ne l'est pas. Cette asymétrie suffit à produire, en quelques semaines et sans qu'aucune décision explicite soit prise, un dépôt où plus rien ne rentre.

Le processus se déroule toujours de la même façon.

**Un travail commence, un autre devient urgent.** Le premier n'est pas fini, il n'est donc pas intégré. Le second démarre. Comme il a besoin de ce que le premier a produit, il part du premier plutôt que de la branche par défaut. La chaîne commence.

**La branche par défaut cesse d'avancer.** Elle reste à sa dernière position, pendant que tout le travail réel se trouve ailleurs. Elle devient l'état le plus ancien du projet au lieu d'en être l'état de référence.

**La relecture devient impossible à un coût acceptable.** Le dernier travail ouvert porte, à lui seul, tout ce qui a été écrit depuis le début. Personne ne relit trois cents fichiers. La relecture est donc reportée, ce qui rend l'écart encore plus grand la fois suivante.

**Des branches de sauvegarde apparaissent.** Nommées à la main, datées, portant parfois un mot comme `incomplet`. Elles attestent qu'une personne ou un agent a craint de perdre du travail et a créé une divergence de plus pour s'en prémunir. C'est un symptôme fiable : quand ce type de branche apparaît, le dépôt a déjà cessé d'être lisible pour celui qui y travaille.

> [!IMPORTANT]
> Aucune de ces quatre étapes n'est une erreur de manipulation de git. Chacune est une décision de conduite de projet, prise sans être formulée. C'est pourquoi une meilleure maîtrise des commandes ne corrige pas la situation.

## Les solutions

Quatre façons d'organiser le travail quand plusieurs fonctionnalités avancent. Elles ne s'opposent pas frontalement, elles supposent des contextes différents.

### Une branche par fonctionnalité, toutes parties de la branche par défaut

Chaque chantier ouvre sa branche depuis la branche par défaut, et y revient dès qu'il est fini. Les branches sont indépendantes : l'ordre d'intégration est libre, et l'abandon de l'une n'affecte pas les autres.

C'est le modèle décrit sous le nom de branches de sujet (*topic branches*). Source : Pro Git, chapitre 3.4 « Branching Workflows », <https://git-scm.com/book/en/v2>, consulté le 8 septembre 2026.

Condition d'emploi : chaque chantier doit pouvoir se terminer sans attendre un autre. Quand deux chantiers touchent les mêmes fichiers, des conflits apparaissent à l'intégration, et ils se résolvent d'autant plus facilement que les branches sont courtes.

### L'empilement volontaire

Une branche part d'une autre branche non encore intégrée (*stacked branches*, ou *stacked pull requests*), parce qu'elle a besoin du code que celle-ci contient.

C'est parfois la seule option honnête : une fonctionnalité qui s'appuie sur une couche technique qui n'existe pas encore ailleurs ne peut pas partir de la branche par défaut sans réécrire cette couche.

Condition d'emploi : la dépendance est réelle et technique, elle est déclarée, et la branche du dessous est intégrée avant qu'un troisième chantier vienne s'ajouter. L'empilement à deux étages est un compromis. À partir de quatre, il n'y a plus de compromis, il y a une file.

### La branche d'intégration de longue durée

Une branche intermédiaire (*long-running branch*) reçoit les fonctionnalités terminées, et la branche par défaut ne reçoit que ce qui est livré.

Le modèle est fait pour des équipes qui produisent des versions numérotées et doivent maintenir plusieurs versions en parallèle. Il apporte une séparation utile dans ce cas précis.

Condition d'emploi : une notion de version distincte de la notion de déploiement. Sans elle, la branche intermédiaire double la branche par défaut sans rien ajouter, et il faut désormais fusionner deux fois au lieu d'une.

### Le tronc unique avec drapeaux de fonctionnalité

Le travail va directement dans la branche par défaut, y compris inachevé, mais reste désactivé par un interrupteur jusqu'à ce qu'il soit prêt. L'approche est connue sous le nom de *trunk-based development*, et les interrupteurs sous celui de *feature flags*, ou *feature toggles*.

L'intégration cesse alors d'être un événement redouté, puisqu'elle a lieu tous les jours. En contrepartie, chaque interrupteur est du code conditionnel qu'il faut écrire, tester dans ses deux positions, et retirer une fois la fonctionnalité acquise. Source : trunkbaseddevelopment.com, page consacrée aux drapeaux de fonctionnalité, <https://trunkbaseddevelopment.com/feature-flags/>, consultée le 8 septembre 2026.

Condition d'emploi : des tests automatiques dignes de ce nom, et la discipline de retirer les interrupteurs. Sans les premiers, l'approche met en production du code non fini. Sans la seconde, le code se remplit de conditions dont plus personne ne connaît l'état attendu.

## La solution retenue dans ce contexte

Contexte visé : une ou deux personnes, un projet en cours d'apprentissage, des tests automatiques encore peu nombreux, pas de notion de version publiée.

**Une branche par fonctionnalité, partie de la branche par défaut, avec une limite explicite de deux chantiers ouverts simultanément.**

### Pourquoi une limite, et pourquoi deux

La limite est le cœur de la solution, pas le nom des branches.

Un chantier ouvert est un travail commencé et non livré. Il a un coût qui court : il faut se souvenir de son état, il vieillit par rapport à la branche par défaut, et il devra être relu un jour, d'autant plus difficilement qu'il aura grossi.

Deux est le nombre qui laisse la place à ce que l'interruption exige, sans laisser la place à l'accumulation : un chantier en cours, un chantier en attente de relecture. L'apparition d'un troisième besoin est le signal qu'il faut terminer avant d'ouvrir.

> [!NOTE]
> Cette limite est exactement une limite de travail en cours (*work in progress limit*, abrégée *WIP limit*) au sens d'un tableau de tickets. La différence est qu'un tableau l'affiche et la fait respecter, alors que git ne connaît pas cette notion et n'opposera jamais aucune résistance.

### Découper pour que le travail rentre

Une limite de deux ne tient que si un chantier se termine en quelques jours. C'est une contrainte sur le découpage, pas sur la vitesse de travail.

Le découpage qui fonctionne est vertical : un morceau traverse toutes les couches et produit un effet observable, aussi petit soit-il. Un morceau qui se limite à une couche, par exemple toute la base de données d'un module, ne peut pas être intégré seul, parce qu'il n'est vérifiable par rien.

Le critère d'arrêt : **un morceau est assez petit lorsqu'il peut être intégré sans rien casser et sans attendre autre chose.** Un morceau qui ne peut pas être intégré seul n'est pas un chantier, c'est une moitié de chantier.

### Quand l'empilement est malgré tout légitime

Trois conditions, cumulatives.

La dépendance est technique et réelle, pas seulement chronologique. Le second chantier a besoin du code du premier, il ne se trouve pas simplement démarré après lui.

L'empilement est déclaré, dans le ticket et dans le nom de la branche, de sorte que personne n'ait à le découvrir en calculant.

La branche du dessous est intégrée avant qu'un troisième étage s'ajoute. C'est la condition qui saute en premier, et sa disparition marque le passage de l'empilement à la file.

> [!WARNING]
> Une branche assise sur une autre contient tout ce que celle-ci contient. Intégrer l'étage du haut intègre donc l'étage du bas, sans relecture et sans avertissement. L'empilement n'est jamais neutre : il transforme une décision d'intégration en plusieurs, dont une seule est visible. Voir `branches-et-integration.md`.

### La branche par défaut doit avancer

Le meilleur indicateur de santé d'un dépôt n'est pas le nombre de branches, c'est la date du dernier commit de la branche par défaut.

Une branche par défaut immobile depuis plusieurs semaines pendant que le travail continue signifie que rien ne rentre. Le nombre de branches ouvertes n'est que la conséquence.

```bash
# Depuis quand la branche par défaut n'a-t-elle pas bougé ?
git log -1 --format=%cs origin/main

# Combien de branches sont ouvertes, et laquelle a cessé de bouger ?
git for-each-ref refs/remotes/origin --sort=-committerdate \
  --format='%(refname:lstrip=3) %(committerdate:short)'
```

### Nommer les branches

Une convention simple suffit : un préfixe de type, puis la portée, en minuscules et tirets.

```text
feat/saisie-formulaire-contact
fix/erreur-connexion-preproduction
chore/mise-a-jour-dependances
```

La convention n'a aucune valeur en soi. Elle en a une seule : permettre de lire ce qu'une branche fait sans ouvrir son historique.

> [!CAUTION]
> **Changer de convention en cours de route coûte plus cher qu'en choisir une mauvaise.** Un dépôt qui porte à la fois `feature/` et `feat/` conserve la trace d'un changement de pratique, et cette trace se lit ensuite comme si elle signifiait quelque chose alors qu'elle ne date que le moment où la branche a été créée.
>
> Si une convention doit changer, elle change pour les branches nouvelles, et le fait est écrit quelque part avec sa date.

## Correspondance avec un tableau de tickets

| Sur un tableau de tickets | Dans un dépôt |
| --- | --- |
| une limite de travail en cours (*WIP limit*) | aucun équivalent, git ne connaît pas la notion |
| un ticket bloqué par un autre | une branche assise sur une autre |
| un ticket qui traîne en colonne « En cours » | une branche qui ne bouge plus depuis des semaines |
| la colonne « Terminé » qui ne se remplit plus | la branche par défaut qui n'avance plus |

La colonne de droite se lit mal, et c'est tout le problème : sur un tableau, l'accumulation se voit d'un coup d'œil parce que la colonne s'allonge. Dans un dépôt, elle ne se voit qu'en listant les branches, ce que personne ne fait spontanément.

Un relevé mensuel des branches ouvertes et de la date du dernier commit de la branche par défaut rétablit la visibilité que le tableau donnait gratuitement.

## Pièges courants

**Partir de la branche courante par défaut, sans y penser.** Créer une branche depuis l'endroit où l'on se trouve empile sans décider d'empiler. Le point de départ se choisit explicitement.

**Ouvrir une branche parce qu'une idée arrive.** Une idée se note dans un ticket. Une branche s'ouvre quand le travail commence.

**Créer une branche de sauvegarde plutôt que de terminer.** Le travail est préservé et le dépôt gagne une divergence que plus personne n'ose fermer, faute de savoir ce qu'elle contient de propre.

**Laisser une branche vieillir en pensant l'intégrer plus tard.** Le coût d'intégration croît avec l'écart. Une branche de trois jours s'intègre, une branche de trois mois se réécrit.

**Confondre « la branche est à jour » et « le travail est livré ».** Une branche qui a reçu les dernières nouveautés de la branche par défaut n'a rien livré. Le flux qui compte va dans l'autre sens.

**Ajouter des branches pour paralléliser le travail d'une seule personne.** Une personne travaille sur un chantier à la fois. Les branches supplémentaires ne créent pas de parallélisme, elles créent de l'en-cours.

## Pour aller plus loin

- Pro Git, chapitre 3.4 « Branching Workflows », pour les branches de longue durée et les branches de sujet : <https://git-scm.com/book/en/v2>. Consulté le 8 septembre 2026.
- Trunk Based Development, page consacrée aux drapeaux de fonctionnalité, et ses références vers l'article fondateur de Martin Fowler : <https://trunkbaseddevelopment.com/feature-flags/>. Consultée le 8 septembre 2026.
- Les termes anglais correspondants sont réunis dans `glossaire.md`, dans le même répertoire.
- La façon d'intégrer une pile déjà constituée fait l'objet de la fiche `branches-et-integration.md`, dans le même répertoire.
- La nature d'une branche, et la raison pour laquelle une branche contient ce qui se trouve en dessous d'elle, sont établies dans `modele-mental.md`.

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
