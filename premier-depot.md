# Premier dépôt

Cette fiche répond à la question de ce qu'il faut poser dans un dépôt avant le tout premier commit, et de ce qu'il faut vérifier avant le tout premier envoi vers une forge.

## Le problème

Un dépôt git conserve tout ce qui y entre. Un fichier commité puis supprimé au commit suivant reste dans l'historique, accessible à quiconque obtient le dépôt.

Trois conséquences se rencontrent régulièrement sur un premier dépôt.

**Un secret entre dans l'historique.** Un fichier de configuration contenant un mot de passe de base de données ou une clé d'interface de programmation est ajouté au premier commit, souvent par un `git add .` lancé avant qu'un `.gitignore` existe. Le retirer ensuite ne le retire que de l'état courant. La seule réponse correcte est la révocation du secret, la réécriture de l'historique n'étant qu'un complément.

**L'historique devient illisible.** Répertoires de dépendances, artefacts de compilation, fichiers de l'environnement de bureau. Le dépôt gonfle, les différentiels deviennent inexploitables, et la revue devient impossible parce que le bruit noie le signal.

**Les fins de ligne divergent.** Un système d'exploitation termine ses lignes par un retour chariot suivi d'un saut de ligne, les autres par un saut de ligne seul. Sans consigne, deux personnes travaillant sur des systèmes différents produisent des différentiels où chaque ligne du fichier apparaît modifiée alors qu'aucune ne l'est. Le problème se rencontre aussi avec une seule personne travaillant depuis deux machines.

Ces trois problèmes ont la même propriété : ils coûtent presque rien à prévenir, et cher à corriger. La fenêtre où la prévention est gratuite se referme au premier commit.

## Les solutions

### Initialiser, puis corriger ensuite

Le dépôt est créé, le contenu est commité, et les fichiers indésirables sont retirés au fur et à mesure qu'ils sont remarqués.

Cette approche est celle qui se pratique par défaut, faute de décision. Elle fonctionne tant qu'aucun secret n'est concerné. Elle échoue exactement dans le cas où l'échec coûte le plus cher.

### Initialiser, poser les trois fichiers, puis commiter

Le dépôt est créé vide, trois fichiers y sont posés avant tout ajout de contenu, et le premier commit ne contient qu'eux.

Les trois fichiers :

- `.gitignore`, qui énumère ce qui n'entre jamais dans le dépôt ;
- `.gitattributes`, qui fixe le traitement des fins de ligne ;
- `README.md`, qui indique ce que le dépôt contient.

Le premier commit devient ainsi une base connue, dont le contenu est vérifiable d'un coup d'œil.

### Créer le dépôt sur la forge, puis le cloner

La forge propose des modèles de `.gitignore` par technologie et crée le dépôt avec un premier commit. Le clone récupère un dépôt déjà configuré.

Cette approche convient quand rien n'existe encore localement. Elle convient moins quand un répertoire de travail existe déjà, cas où le clone et le répertoire doivent être réconciliés.

Les modèles de la forge sont un point de départ, pas un résultat. Ils couvrent les artefacts d'une technologie, jamais les conventions propres à un projet.

## La solution retenue dans ce contexte

La deuxième, pour un répertoire de travail préexistant.

L'ordre exact :

```bash
# 1. Créer le dépôt, sans rien ajouter
git init

# 2. Poser les trois fichiers, puis vérifier ce qui serait ajouté
git status

# 3. Lire cette liste en entier, ligne par ligne
#    Tout ce qui ne devrait pas y figurer signale un .gitignore incomplet

# 4. Ajouter explicitement, jamais avec un point
git add .gitignore .gitattributes README.md
git commit -m "chore: initialise le dépôt"
```

L'étape 3 est celle qui se saute, et c'est celle qui a de la valeur. Un `git status` avant le premier `git add` liste exactement ce qui entrerait dans l'historique. C'est le dernier moment où cette liste tient sur un écran.

### Contenu minimal du `.gitattributes`

```gitattributes
# Normalisation des fins de ligne, saut de ligne seul partout
* text=auto eol=lf

# Fichiers binaires, aucune conversion
*.png binary
*.jpg binary
*.pdf binary
```

Avec `text=auto`, git détermine lui même si un fichier est textuel ou binaire, et normalise ses fins de ligne à l'entrée dans l'index. L'attribut `eol` fixe le style employé dans la copie de travail au moment de l'extraction, et sa valeur `lf` impose le saut de ligne seul quel que soit le système. Source : documentation git, `gitattributes`, consultée le 8 septembre 2026.

Cette configuration est gratuite tant que le dépôt n'a pas d'historique. Ensuite, la normalisation produit un commit qui touche tous les fichiers, ce qui rend inexploitable la commande attribuant chaque ligne à son auteur.

### Contenu minimal du `.gitignore`

Quatre catégories, dans cet ordre de priorité :

1. **Les secrets.** Fichiers d'environnement, clés privées, certificats. Un modèle d'exemple sans valeur réelle est versionné à leur place.
2. **Les dépendances installées.** Elles se réinstallent depuis un fichier de déclaration, qui est versionné.
3. **Les artefacts produits.** Répertoires de compilation, sorties d'outils.
4. **Les fichiers locaux.** Configuration d'éditeur, fichiers du système d'exploitation, brouillons personnels.

Chaque catégorie porte un commentaire indiquant son motif. Un `.gitignore` sans commentaires devient, en quelques mois, une liste que personne n'ose modifier faute de savoir pourquoi chaque ligne s'y trouve.

### Nom de la branche initiale

`git init` crée une branche nommée `master`. Depuis la version 2.28, ce nom se configure. Source : Pro Git, chapitre sur la configuration initiale, consulté le 8 septembre 2026.

Les forges, elles, créent leurs dépôts avec une branche `main`. Sans configuration, le dépôt local et le dépôt distant portent donc deux noms différents pour la même chose, ce qui se manifeste au premier envoi par un message que rien ne laissait prévoir.

```bash
# Une fois pour toutes, avant le prochain dépôt
git config --global init.defaultBranch main

# Sur un dépôt déjà initialisé, renommer la branche courante
git branch -m main
```

💭 Le changement de ce nom par défaut est annoncé pour une version majeure à venir de git. Configurer explicitement met à l'abri de la transition, dans un sens comme dans l'autre.

### Créer le dépôt distant, et le créer vide

Sur la forge, le dépôt se crée sans aucun fichier initial : ni `README`, ni `.gitignore`, ni licence.

La raison est mécanique. Un dépôt créé avec un fichier initial contient déjà un commit, le dépôt local en contient un autre, et les deux historiques n'ont aucun ancêtre commun. Git refuse alors de les rapprocher. Les deux issues qui se présentent sont l'une et l'autre mauvaises : forcer la publication écrase le contenu distant, autoriser la fusion d'historiques sans lien produit un arbre incohérent dès le premier jour.

Un dépôt distant vide n'a pas ce problème : il n'a rien à écraser.

### Vérifier la visibilité avant de publier

La visibilité se vérifie à la création du dépôt, pas après le premier envoi.

Un dépôt basculé de public à privé ne redevient pas confidentiel. Entre la publication et le basculement, le contenu a pu être copié, indexé par des moteurs, ou repris dans une bifurcation qui, elle, reste accessible. La correction ne rattrape que l'avenir.

Cette vérification est le pendant distant du `git status` de l'étape 3 : dans les deux cas, il s'agit de regarder ce qui va sortir pendant que c'est encore réversible.

### Rattacher le dépôt local et publier

```bash
# 1. Déclarer le dépôt distant sous le nom conventionnel origin
git remote add origin <adresse fournie par la forge>

# 2. Vérifier vers quoi il pointe, en lecture et en écriture
git remote -v

# 3. Publier, en associant la branche locale à la branche distante
git push -u origin main
```

L'option `-u` de l'étape 3 enregistre l'association une fois pour toutes : les envois suivants se font par un `git push` sans argument.

Deux formes d'adresse coexistent.

| Forme | Authentification | Convient quand |
| --- | --- | --- |
| HTTPS | jeton d'accès personnel, à périmètre et durée limités | machine occasionnelle, réseau filtrant les autres protocoles |
| SSH | paire de clés, la clé privée protégée par une phrase secrète | machine de travail habituelle |

Dans les deux cas, la forge n'accepte pas le mot de passe du compte. Un jeton se crée avec le périmètre minimal et une date d'expiration. Une clé SSH sans phrase secrète équivaut à un accès permanent pour quiconque obtient le fichier.

### Vérifier ce qui est effectivement en ligne

Après le premier envoi, l'état réel se lit sur la forge, pas dans la copie locale.

Trois vérifications, dans cet ordre :

1. **La liste des fichiers publiés.** Elle doit correspondre à ce que le `git status` de l'étape 3 annonçait, et à rien de plus.
2. **La visibilité effective du dépôt**, telle que la forge l'affiche.
3. **Les alertes de la forge**, qui signale souvent d'elle même un secret repéré dans un commit.

C'est aussi le moment de protéger la branche par défaut, avant qu'elle ne reçoive du travail : interdire l'envoi direct, interdire l'envoi forcé, exiger une demande de fusion. Une protection posée le premier jour ne coûte rien, posée plus tard elle contrarie des habitudes déjà prises.

🔗 Une protection de branche ne protège pas seulement d'un tiers. Elle protège surtout la personne qui détient les droits contre sa propre commande lancée trop vite, ce qui est de loin le cas le plus fréquent sur un dépôt à un seul auteur.

## Pièges courants

**`git add .` au premier commit.** La commande ajoute tout ce que le `.gitignore` n'exclut pas, y compris ce qui a été oublié. Un ajout explicite oblige à nommer ce qui entre.

**Poser le `.gitignore` après le premier commit.** Un fichier déjà suivi par git continue de l'être même s'il correspond ensuite à une règle d'exclusion. La règle ne s'applique qu'aux fichiers non suivis.

**Croire qu'une suppression efface.** Un secret commité est un secret publié. La première mesure est sa révocation, la réécriture de l'historique vient après et ne la remplace pas.

**Poser le `.gitattributes` plus tard.** Il fonctionnera, au prix d'un commit de normalisation qui touche tout le dépôt.

**Créer le dépôt distant avec un `README` automatique.** Deux historiques sans ancêtre commun, un envoi refusé, et deux mauvaises façons de s'en sortir. Le dépôt distant se crée vide.

**Publier d'abord, régler la visibilité ensuite.** Le basculement en privé ne dépublie pas ce qui a déjà été vu, copié ou bifurqué.

**Un `origin` qui pointe vers le mauvais dépôt.** Le cas se présente dès qu'une machine héberge plusieurs clones du même projet à des fins différentes. Un `git remote -v` avant le premier envoi coûte une seconde.

**Recourir à l'envoi forcé pour débloquer une situation.** La commande réécrit l'historique distant et supprime ce que d'autres avaient déjà récupéré. Sur la branche par défaut, elle ne se justifie pratiquement jamais, et la protection de branche est là pour la rendre impossible.

**Laisser une session assistée commiter sans relecture.** Développé ci dessous.

### Quand le commit est produit par une session assistée

Un assistant de codage disposant d'un accès au terminal peut initialiser un dépôt, écrire un `.gitignore` et commiter. Le résultat est souvent correct, et la commodité est réelle.

Trois précautions s'imposent néanmoins.

**Le `.gitignore` est relu par une personne avant le premier commit.** Un assistant produit un fichier plausible pour la technologie qu'il détecte. Il ne connaît ni les fichiers propres au projet, ni les conventions de rangement de la personne qui travaille dessus.

**Le périmètre accessible depuis le terminal est délimité.** Un assistant qui peut lire l'ensemble d'un disque peut, en principe, ajouter au dépôt des fichiers situés hors du répertoire de travail.

**L'instruction peut venir d'ailleurs que de la personne.** C'est le point le moins intuitif. Un assistant qui lit un fichier, une page ou un ticket traite ce contenu comme du texte, et ce texte peut contenir des instructions rédigées à son intention. Un fichier de dépendances, une documentation appelée en ligne ou un rapport d'anomalie suffisent.

Quand ce même assistant dispose d'un accès au terminal et d'un droit de publication vers une forge, la conséquence n'est plus une réponse erronée mais une action exécutée : un fichier lu, une commande lancée, un commit publié.

La mesure qui tient est la séparation des capacités. Un assistant qui lit du contenu extérieur ne publie pas. Un assistant qui publie ne lit que ce que la personne lui fournit. Réunies, les deux capacités forment un chemin allant d'un contenu non maîtrisé jusqu'à un dépôt distant, sans qu'aucune décision humaine s'interpose.

À défaut de séparation, la publication reste manuelle : l'assistant prépare, une personne relit le différentiel et lance elle même la commande.

## Pour aller plus loin

- Documentation git, `gitattributes`, section sur la conversion des fins de ligne : <https://git-scm.com/docs/gitattributes>. Consultée le 8 septembre 2026.
- Documentation git, `gitignore`, pour la syntaxe des motifs : <https://git-scm.com/docs/gitignore>. Consultée le 8 septembre 2026.
- Pro Git, configuration initiale, pour le nom de la branche par défaut : <https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup>. Consulté le 8 septembre 2026.
- La conduite à tenir après un secret commité fait l'objet de la fiche `incident-secret-commite.md`, dans le même répertoire.

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
