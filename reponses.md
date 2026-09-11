# TD DORA — réponses

- **Auteur** : Thomas Le Cam (`ThomasdevLC`)
- **Dépôt mesuré (amont, lecture seule)** : `excalidraw/excalidraw`
- **Fork instrumenté (phase 4)** : `ThomasdevLC/excalidraw`
- **Date du TD** : 11 septembre 2026
- **Support de référence** : `DORA-mesurer-la-performance/Readme.md`

---

## Phase 0 — Le contrat de définitions

**Contrat figé le 11 septembre 2026 à 11 h 01 CEST (09 h 01 UTC).**

### Réserve d'honnêteté sur l'ordre des phases

Le sujet prescrit de remplir ce contrat **avant de regarder les données** — pour
éviter le biais de justification rétrospective : qui connaît déjà les chiffres
choisit, sans s'en rendre compte, les définitions qui donnent le résultat le plus
flatteur.

Nous avons inversé cet ordre : le contrat a été rédigé **après l'étape 2 du
guide**. Trois décisions sont donc contaminées, cinq ne le sont pas :

| Décision | Prise à l'aveugle ? |
|---|---|
| `deploiement.compte_comme_deploiement` | non — les environnements avaient été relevés |
| `deploiement.exclut` | non — idem |
| `deploiement.horodatage` | non — les statuts avaient été consultés |
| `changement.point_de_depart` | oui |
| `incident.*` | oui |
| `rework.marqueur` | oui |
| `fenetre_de_reference` | oui |
| `agregation` | oui |

Atténuation, mais pas excuse : sur les trois décisions contaminées, le support
tranche de lui-même (2.3 et 7.1 pour le périmètre, 5.2 pour l'horodatage). Une
lecture du cours seul aurait conduit au même contrat. La conséquence réelle porte
sur la **valeur de contrôle de la question 6**, qui ne peut plus jouer son rôle —
voir cette réponse.

### `dora-definitions.yml` figé

```yaml
# dora-definitions.yml : contrat d'équipe, versionné avec le code
#
# TD phase 0 — rempli par Thomas Le Cam (ThomasdevLC)
# Figé le 11 septembre 2026 à 11 h 01 CEST (09 h 01 UTC).
#
# RÉSERVE D'HONNÊTETÉ : ce contrat a été rédigé APRÈS l'étape 2 du guide, et
# non avant comme le prescrit la phase 0. Les décisions `deploiement.*`
# (compte_comme_deploiement, exclut, horodatage) ont donc été prises en
# connaissance des données. Les décisions `changement`, `incident`, `rework`,
# `fenetre_de_reference` et `agregation` ont bien été prises à l'aveugle.
# Voir la réponse à la question 6 du rendu.

application: excalidraw   # l'application Excalidraw uniquement, ni sa doc, ni ses exemples

deploiement:
  # Support 2.3 (production seulement) + 7.1 (une vue par application).
  # La valeur d'environment est copiée-collée depuis l'API : son séparateur est
  # un TIRET DEMI-CADRATIN U+2013, pas un trait d'union U+002D. Retapée au
  # clavier, elle fait renvoyer zéro déploiement sans message d'erreur.
  compte_comme_deploiement: >-
    déploiement dont l'environment vaut exactement "Production – excalidraw"
    ET qui porte au moins un status 'success'

  exclut:
    - "les 4 environnements Preview – * : prévisualisations de PR, aucun utilisateur servi (support 2.3, 7.3)"
    - "Production – docs : production réelle, mais autre application (support 7.1)"
    - "Production – excalidraw-package-example : autre application (support 7.1)"
    - "Production – excalidraw-package-example-with-nextjs : autre application (support 7.1)"
    - "tout déploiement sans status 'success' : échoué, annulé ou encore en cours (support 2.5)"

  # Support 5.2. Un objet deployment est une INTENTION de déployer ; le status
  # est le RÉSULTAT. Retenir le created_at du deployment reviendrait à dater un
  # changement qui n'est peut-être jamais arrivé en production.
  # En cas de statuts 'success' multiples, le PREMIER fait foi.
  horodatage: "created_at du premier status 'success' du déploiement"

changement:
  # Support 2.2 : le point de départ est le commit, pas le ticket ni la PR.
  # Précisions que le support laisse ouvertes et que ce contrat ferme :
  #   - committer date (%cI de git log), conformément à l'exemple du support 5.2 ;
  #   - lot = commits présents dans le sha du déploiement N et absents de celui
  #     du déploiement N-1, sur la branche par défaut (master chez excalidraw) ;
  #   - un lead time par COMMIT, pas par déploiement : chaque commit du lot
  #     hérite de l'horodatage de déploiement du lot (support 2.2, coût des
  #     gros lots).
  point_de_depart: >-
    committer date (%cI) de chaque commit de la branche par défaut (master),
    le lot étant délimité par les sha de deux déploiements consécutifs

incident:
  # Nous n'exploitons pas la production d'excalidraw : ni alerting, ni astreinte,
  # ni post-mortem. Ce qui suit est donc un PROXY assumé, pas la métrique DORA.
  # Ce que le proxy ne mesure pas : les dégradations non signalées par une issue,
  # celles signalées ailleurs (Discord, Twitter), et celles jamais détectées.
  definition: >-
    PROXY — dégradation en production signalée par une issue publique du dépôt.
    La métrique DORA exige une dégradation "nécessitant une intervention
    immédiate" (support 2.4) ; une issue publique ne porte pas cette information.
  source: "issue GitHub portant le label 'incident' (label 'bug' utilisé comme proxy dégradé en phase 3)"

  # Support 2.4 : le chronomètre DORA démarre au début de la dégradation
  # DÉTECTÉE, pas à l'ouverture du ticket. Nous n'avons pas la date de
  # détection : nous retenons donc l'ouverture de l'issue, en sachant que
  # l'écart entre les deux est justement une mesure de l'observabilité — mesure
  # que nous renonçons ici à produire.
  debut: "created_at de l'issue (approximation par défaut ; la date de détection n'est pas disponible)"
  fin: "closed_at de l'issue, interprété comme 'service rétabli' (support 2.4 : pas 'post-mortem rédigé')"

  # LE MAILLON FAIBLE (support 5.1). Aucun outil ne peut déduire ce lien :
  # quelqu'un doit l'avoir écrit.
  rattachement_deploiement: >-
    ligne "caused_by: <id de déploiement>" dans le CORPS de l'issue (pas dans un
    commentaire), renseignée par la personne qui traite l'incident.
    Un incident sans cette ligne est compté dans aucune métrique.

rework:
  # Support 2.6 : la convention doit être décidée AVANT la collecte, sinon la
  # donnée n'existe pas. Excalidraw n'a aucune branche hotfix/* (311 branches
  # vérifiées) : la convention ne produira donc rien sur l'amont, et sera
  # appliquée sur le fork en phase 4.
  marqueur: "ref du déploiement commençant par 'hotfix/'"

# Support 7.3 : 7 jours produit du bruit ingérable, l'année masque les
# tendances. Entre 28 et 90, je retiens 90 : l'application excalidraw ne déploie
# pas en continu, et une fenêtre de 28 jours risquerait de ne contenir qu'un ou
# deux déploiements — trop peu pour qu'une médiane ait un sens.
fenetre_de_reference: "90 jours glissants"

# Support 2.2 et annexe B : la distribution des lead times est asymétrique à
# droite, la moyenne est tirée par la traîne et ne décrit personne.
agregation: "médiane (P50) pour toutes les durées, P90 publié systématiquement à côté ; taux en pourcentage"

# ---------------------------------------------------------------------------
# Phase 3 : à remplir après avoir vu les chiffres.
# Qu'auriez-vous écrit différemment ? Ne modifiez pas les lignes ci-dessus.
#
# revision_envisagee: |
#   ???
```

### Les décisions que le support ne tranchait pas

Trois points du contrat ne sont pas dictés par le cours. Ce sont ceux qui
expliqueront un écart avec un autre binôme.

1. **`point_de_depart` — committer date plutôt qu'author date.** Le support dit
   « le commit », sans préciser laquelle des deux dates git. Un commit porte
   une *author date* (quand le code a été écrit) et une *committer date* (quand
   l'objet commit a été créé). Un rebase ou un squash-merge réécrit la seconde,
   pas la première. Excalidraw fusionnant par pull request, retenir la committer
   date **exclut mécaniquement le temps passé en revue** du lead time mesuré.
   Nous suivons malgré tout le support (5.2) et l'outil (`git log --format=%cI`),
   par cohérence — mais c'est une convention, pas une vérité, et elle rend nos
   chiffres plus flatteurs que la réalité vécue.

2. **`incident.debut` — ouverture de l'issue, faute de mieux.** Le support (2.4)
   exige le début de la dégradation *détectée*. Nous n'avons aucune source de
   détection. Nous retenons donc le `created_at` de l'issue en le nommant pour
   ce qu'il est : une approximation qui **sous-estime** le temps de restauration,
   puisqu'elle ne compte pas le délai entre la dégradation et son signalement.
   Or ce délai est précisément la mesure de l'observabilité — nous renonçons donc
   à mesurer ce que la métrique était censée révéler.

3. **`fenetre_de_reference` — 90 jours plutôt que 28.** Le support (7.3) écarte
   7 jours (bruit ingérable) et l'année (tendances masquées), et laisse le choix
   entre 28 et 90. Nous retenons 90 : une application qui ne déploie pas en
   continu risque de ne présenter qu'un ou deux déploiements sur 28 jours, ce qui
   rend toute médiane dénuée de sens. Le prix de ce choix est une réactivité
   moindre : une amélioration de pratique mettra trois mois à devenir lisible
   dans les chiffres.

---

## Phase 1 — Reconnaissance

### Relevé des environnements

Source : `https://api.github.com/repos/excalidraw/excalidraw/deployments?per_page=100`,
pages 1 à 4, relevé le **11 septembre 2026**.

Les 100 déploiements de la page 1 ne couvrent que **3 jours** (8 → 11 septembre) :
c'est trop court pour voir tous les environnements. J'ai donc élargi à 400
déploiements (**16 août → 11 septembre 2026**).

| Occurrences (400 dépl.) | `environment` |
|---:|---|
| 360 | `Preview – excalidraw` |
| 10 | `Production – excalidraw-package-example-with-nextjs` |
| 10 | `Production – excalidraw-package-example` |
| 7 | `Preview – excalidraw-package-example-with-nextjs` |
| 7 | `Preview – excalidraw-package-example` |
| **3** | **`Production – excalidraw`** |
| 2 | `Production – docs` |
| 1 | `Preview – docs` |

Sur la seule page 1 (100 déploiements, 3 jours), on n'en voit que **5** :
`Preview – excalidraw` (91), les deux `Production – …package-example*` (3 chacun),
`Production – docs` (2) et `Production – excalidraw` (1).

**Note sur le tiret :** le séparateur de ces noms est un **tiret demi-cadratin
`U+2013`**, pas un trait d'union `U+002D`. Dans
`Production – excalidraw-package-example`, les deux caractères cohabitent : `U+2013`
après `Production`, puis de vrais `U+002D` dans `package-example`. Toute valeur
retapée au clavier fait renvoyer **zéro déploiement au collecteur, sans message
d'erreur**.

---

### 1. Combien de valeurs différentes d'`environment` trouvez-vous ? Listez-les.

**8 valeurs distinctes** sur la fenêtre relevée (400 déploiements, 16 août →
11 septembre 2026) : 4 commençant par `Production`, 4 par `Preview`, listées
dans le tableau ci-dessus.

Le périmètre du comptage fait partie de la réponse : sur les 100 déploiements de
la page 1, qui ne couvrent que 3 jours, on n'en trouve que **5**. Une liste
d'environnements sans sa fenêtre d'observation n'est pas reproductible.

C'est vérifiable sur le guide du TD lui-même, qui annonce **6** valeurs
(3 `Production`, 3 `Preview`) relevées le 6 septembre 2026 : la paire `docs` est
apparue depuis. Le guide n'est pas faux, il est **daté**. Une liste
d'environnements n'est pas un référentiel stable, c'est un état à re-constater à
chaque collecte.

---

### 2. Lesquelles correspondent à une mise en production au sens de DORA ? Lesquelles doivent être exclues, et pourquoi ?

**Un seul environnement est retenu : `Production – excalidraw`.**

Les 7 autres sont exclus, mais pour **deux motifs distincts** qu'il faut
distinguer, car ils ne relèvent pas du même chapitre du support.

**Exclusion 1 — les 4 `Preview` : ce n'est pas de la production.**

Excalidraw déploie sur Vercel, qui enregistre un déploiement par push de branche
afin de générer une URL de prévisualisation de la pull request. Aucun
utilisateur n'est servi par ces déploiements : ils n'existent que pour la revue.
Le support 2.3 est explicite — *« Comptez les déploiements en production, pas les
builds ni les livraisons en préproduction. »* Les compter reviendrait à mesurer
l'activité de revue, pas la livraison de valeur.

**Exclusion 2 — `Production – docs` et les deux `Production – …package-example*` : c'est de la production, mais d'une autre application.**

Ces trois environnements servent du contenu à de vrais visiteurs : le motif
précédent ne s'applique donc pas. Ce qui les exclut, c'est le chapitre 7.1, piège
*« Comparer l'incomparable »* — *« les métriques valent par application ou
service »*. Le site de documentation se déploie à chaque correction de faute de
frappe dans un `.md` ; l'application se déploie quand une release est prête. Les
deux populations n'ont ni le même rythme, ni le même risque, ni les mêmes
conséquences en cas d'échec.

L'effet est massif dans ce cas précis : sur les 25 déploiements `Production` de
la fenêtre, `Production – excalidraw` n'en pèse que **3**. Une métrique agrégeant
les 4 environnements serait pilotée à **88 %** par des artefacts qui ne sont pas
le produit — elle ne décrirait ni l'application, ni la doc, ni les exemples, mais
une application qui n'existe pas.

**Ce que l'exclusion n'est pas :** `docs` n'est pas écarté parce qu'il serait
« moins important ». Rien n'interdirait — DORA le recommande même — de calculer
une **seconde** série de métriques pour `Production – docs`. Ce qui est interdit,
c'est de les additionner.

Le collecteur fourni valide ce raisonnement par sa signature : `--environment`
attend **une chaîne unique**, pas une liste. L'outil a été écrit en supposant
« une application = une série de métriques ».

---

### 3. Si vous comptiez tous ces déploiements, de quel facteur votre deployment frequency serait-elle surestimée ?

**Facteur ≈ 133×.**

Le calcul retenu compare le total de tous les déploiements au seul périmètre
valide défini en question 2 :

```
400 déploiements toutes valeurs d'environment
  3 déploiements « Production – excalidraw »
--------------------------------------------
  facteur de surestimation ≈ 133
```

Sur la seule page 1, le rapport est de **100 / 1 = 100×**.

**Pourquoi ce rapport et pas un autre.** Une autre lecture était possible : ne
comparer que le total aux 25 déploiements `Production` (facteur ≈ 16×). Je la
rejette parce que le facteur doit mesurer l'erreur **effectivement commise** par
quelqu'un qui ne filtre rien. Or cette personne n'exclut ni les `Preview`, ni les
autres applications : elle cumule donc les deux erreurs de la question 2, et le
facteur doit les cumuler aussi.

L'ordre de grandeur est le vrai enseignement : la fréquence ne serait pas
« un peu » fausse, elle serait fausse de **plus de deux ordres de grandeur**.
Avec 0,033 déploiement/jour réel, un compteur naïf afficherait environ
4,4 déploiements/jour — soit le profil d'une équipe *elite* au sens de 4.2, pour
un projet qui déploie son application une fois par semaine et demie.

---

### 4. Retrouvez la ligne correspondante dans le tableau des erreurs d'implémentation (7.3 du support).

Tableau 7.3, deuxième ligne :

| Erreur | Symptôme | Correction |
|---|---|---|
| Compter les builds comme des déploiements | Fréquence 10× trop élevée | Filtrer sur `environment=production` |

**Le symptôme annoncé est sous-estimé dans ce cas.** Le support annonce « 10× trop
élevée » ; la mesure donne **≈ 133×**. L'écart s'explique : le support suppose une
CI qui produit quelques builds par changement, alors que Vercel crée une
prévisualisation **par push de branche** sur un projet à 311 branches et ~4 000
commits. Le mécanisme d'erreur est identique, son amplitude dépend entièrement de
l'outillage observé.

**Et la correction proposée est insuffisante ici.** Filtrer sur
`environment=production` ramènerait de 400 à 25 déploiements — soit encore **8×**
de surestimation, puisque le filtre laisse passer `docs` et les deux
`package-example`. La correction du support traite l'exclusion 1 de la question 2,
pas l'exclusion 2. Sur un dépôt réel, le filtre doit porter sur la valeur
**exacte** de l'environnement de l'application mesurée, pas sur un préfixe.

---

### 5. Que contient le tableau des statuts d'un déploiement ? Pourquoi l'existence d'un déploiement ne suffit-elle pas à établir qu'un changement est arrivé en production ?

**Relevé.** Dernier déploiement `Production – excalidraw` de la fenêtre,
id `6374481865` (10 septembre 2026) :

```
GET /repos/excalidraw/excalidraw/deployments/6374481865/statuses
```

```json
[
  {
    "state": "success",
    "created_at": "2026-09-10T14:53:28Z",
    "description": "Deployment has completed",
    "environment": "Production – excalidraw",
    "environment_url": "https://excalidraw-ceeaxmmfa-excalidraw.vercel.app"
  }
]
```

**Contenu.** Un tableau d'objets `deployment_status`, chacun portant un `state`,
son propre `created_at`, une `description`, l'`environment` et l'URL servie
(`environment_url`). Les états possibles côté GitHub sont `queued`, `pending`,
`in_progress`, puis l'un de `success`, `failure`, `error` ou `inactive`.

**Pourquoi l'existence d'un déploiement ne suffit pas.** Parce que dans l'API
GitHub, l'objet `deployment` n'est pas un fait mais une **intention** : il
déclare qu'on veut porter tel `sha` sur tel `environment`. Ce qui s'est
réellement produit ensuite n'est écrit nulle part dans cet objet — il est écrit
dans ses **statuts**, une sous-ressource distincte.

Un déploiement dont le seul statut est `failure` existe donc dans l'API avec son
`sha`, son `environment` et son `created_at` : tout ce qu'il faut pour qu'un
compteur naïf l'additionne aux autres. Il n'a pourtant jamais servi un seul
utilisateur. Compter des déploiements sans lire leurs statuts, c'est compter des
intentions et les présenter comme des livraisons.

Le support le dit du côté de l'interprétation, en 2.5 : *« Un échec du pipeline
avant la mise en production n'est pas un change failure : c'est le système qui
fonctionne. Ne comptez que ce qui a atteint la production. »*

Le collecteur fourni applique cette règle littéralement, dans
`fetch_deployments()` :

```python
ok = [s for s in statuses if s["state"] == "success"]
if not ok:
    continue          # le déploiement est purement et simplement écarté
```

**Coût de cette vérification.** C'est la raison pour laquelle le collecteur
consomme une quinzaine de requêtes par exécution : la liste des déploiements ne
porte pas les statuts, il faut une requête **par déploiement** pour les obtenir.
La correction de cette erreur a donc un prix en quota d'API — ce qui explique
pourquoi tant d'implémentations naïves s'en dispensent.

---

### 6. Quel horodatage faut-il retenir : le `created_at` du déploiement, ou celui du statut `success` ? Votre contrat de phase 0 avait-il tranché ?

**Il faut retenir le `created_at` du statut `success`.**

L'argument découle de la question 5 : le `created_at` du déploiement date une
*intention*. Le retenir revient à dater en production un changement qui n'y est
peut-être jamais arrivé — et à faire entrer les échecs dans une métrique de
débit. Le `created_at` du statut `success` date un *fait* : à cet instant, le
changement servait des utilisateurs. C'est bien le `deployed_at` de la formule
`lead_time = deployed_at − committed_at` du chapitre 2.2.

C'est aussi ce que retient l'exemple de contrat du support (5.2) —
`horodatage: "fin du déploiement (created_at du status success)"` — et ce
qu'implémente le collecteur, qui prend le **premier** statut `success` :

```python
first_success = min(ok, key=lambda s: parse_date(s["created_at"]))
```

**Observation qui rend la question sans effet sur ce dépôt.** Le guide annonce
que le statut `success` porte *« son propre `created_at`, différent de celui du
déploiement »*. Ce n'est pas ce qu'on observe. Vérification sur les trois
déploiements de production de la fenêtre, plus un `Preview` :

| Déploiement | nb de statuts | `created_at` du statut vs du déploiement |
|---|---|---|
| 6374481865 (prod) | 1 | identique à la seconde |
| 6312286174 (prod) | 1 | identique à la seconde |
| 6247042154 (prod) | 1 | identique à la seconde |
| 6387864695 (preview) | 1 | + 1 seconde |

C'est systématique, donc ce n'est pas un hasard : c'est la signature de Vercel.
Vercel ne crée pas l'évènement GitHub *avant* de déployer pour le mettre à jour
ensuite ; il déploie d'abord, puis enregistre l'évènement **et** son statut
`success` une fois le résultat connu. D'où un seul statut, et deux dates
confondues.

Conséquence : sur excalidraw, les deux choix produisent **exactement les mêmes
chiffres**. La décision reste néanmoins nécessaire — et elle produira un écart
réel sur le workflow GitHub Actions de la phase 4, où le déploiement est créé
*avant* son exécution et où les deux dates encadrent la durée du déploiement.

C'est un enseignement à part entière : **une décision de contrat peut être sans
effet sur un jeu de données et décisive sur un autre.** Ne pas la prendre parce
qu'elle ne change rien aujourd'hui, c'est la laisser à l'outil le jour où elle
comptera.

**Le contrat de phase 0 avait-il tranché ? Non — et il ne pouvait pas.**

Notre contrat n'existait pas encore au moment de cette observation : il a été
rédigé après l'étape 2 du guide (voir la réserve d'honnêteté en phase 0). La
ligne `horodatage: "created_at du premier status 'success' du déploiement"` y
figure bien, mais elle a été écrite **en connaissance** des statuts consultés
ici.

Cette question est donc la seule du TD à laquelle nous répondons en constatant un
manquement de méthode plutôt qu'un résultat. Ce qu'elle cherchait à vérifier —
un contrat écrit à l'aveugle résiste-t-il au contact des données réelles ? — nous
ne pouvons pas en témoigner. Nous pouvons seulement témoigner du contraire : que
l'ordre prescrit par le sujet n'est pas une formalité pédagogique, puisque son
non-respect rend une des vingt-quatre questions inexploitable.

---

## Phase 2 — Collecte outillée

### Commande exécutée

```bash
python3 outils/dora_metrics.py \
  --repo excalidraw/excalidraw \
  --git ../../excalidraw \
  --environment "Production – excalidraw" \
  --window 90
```

Deux écarts avec le guide, nécessaires pour que la commande fonctionne :
`--git ../../excalidraw` (le clone est frère du dossier de cours, pas du dossier
`TD`), et la valeur de `--environment` copiée-collée depuis l'API pour préserver
le tiret `U+2013`.

### Sortie brute

```
=== DORA : excalidraw/excalidraw ===
Fenêtre : 90 jours (depuis le 2026-06-13)
Environnement : "Production – excalidraw"

DÉBIT
  Deployment frequency                 0.167 /jour  (15 déploiements)
    délai médian entre deux            4.5 j
  Change lead time (P50)               43.8 h
  Change lead time (P90)               8.9 j
    base de calcul                     79 commits / 14 lots
  Failed deployment recovery time      n/a

INSTABILITÉ
  Change fail rate                     n/a
  Deployment rework rate               n/a
```

### Tableau de résultats

| Métrique | Valeur obtenue |
|---|---|
| Deployment frequency | **0,167 /jour** (15 déploiements sur 90 jours), soit ~1,2/semaine |
| Délai médian entre deux déploiements | **4,5 jours** |
| Change lead time P50 | **43,8 h** (1,8 jour) |
| Change lead time P90 | **8,9 jours** (213,6 h) |
| Commits analysés / nombre de lots | **79 commits / 14 lots** |

### Les deux rapports non affichés par l'outil

```
taille moyenne d'un lot  =  79 commits ÷ 14 lots      =  5,6 commits
rapport P90 / P50        =  213,6 h ÷ 43,8 h          =  4,9×
```

### Deux précisions de lecture

**15 déploiements mais 14 lots.** Ce n'est pas une perte de données : le
collecteur calcule les lead times entre **paires consécutives**
(`zip(deployments, deployments[1:])`). Quinze déploiements produisent quatorze
intervalles. Le premier déploiement de la fenêtre n'a pas de prédécesseur : on ne
sait pas depuis quand ses commits attendaient, son lot est donc indéterminé et
exclu du calcul.

**Aucun lot ignoré.** La ligne `lots ignorés (sha absent)` ne s'affiche que si le
compteur est non nul : tous les sha des déploiements étaient présents dans le
clone local. L'échec du `--filter=blob:none` (voir phase 1) a ici joué en notre
faveur.

---

### 7. Rapportez le nombre de commits au nombre de lots. Que vaut la taille moyenne d'un lot ? Que dit le chapitre 6.1 du support de ce chiffre ?

**79 ÷ 14 = 5,6 commits par déploiement.**

Le chapitre 6.1 ne donne pas de seuil — il n'existe pas de « bonne » taille de
lot en valeur absolue. Ce qu'il dit, c'est que « Travail par petits lots » est la
seule ligne de son tableau de capabilities dont l'effet principal est écrit en
gras : **« Le levier le plus rentable, sur les cinq métriques »**. Le
développement de l'annexe B en donne la mécanique : la loi de Little
(`lead time = travail en cours ÷ débit`) rend l'en-cours *la seule variable
directement actionnable* de l'équation, et les petits lots agissent sur les deux
termes à la fois — moins d'en-cours et moins de variabilité par changement.

**5,6 commits est un lot modeste en volume.** Mais le chiffre pris seul est
trompeur, et c'est le vrai enseignement de cette question : **le lead time ne
mesure pas le nombre de commits d'un lot, il mesure leur temps d'attente.**

Confrontons les deux chiffres du tableau : 5,6 commits par lot, mais un intervalle
médian de **4,5 jours** entre deux déploiements. Le lot n'est pas gros, il est
**lent**. Un commit fusionné juste après un déploiement attendra en moyenne
plusieurs jours le suivant, sans que rien dans sa taille ne l'explique.

Le P50 de 43,8 h le confirme : sur les 4,5 jours d'intervalle, près de deux jours
sont du pur délai de convoi. C'est exactement ce que décrit l'annexe B sur la
cartographie du flux de valeur — *« l'essentiel du lead time est de l'attente »*,
avec une efficacité de flux qui dépasse rarement 15 % dans une chaîne non
optimisée. Réduire encore la taille des lots n'y changerait pas grand-chose ;
augmenter la **cadence** de déploiement, si.

---

### 8. Comparez P50 et P90. Quel est le rapport entre les deux ? D'après la section « médiane et percentiles » de l'annexe B, que signale un tel écart, et que ne signale-t-il pas ?

**Rapport P90 / P50 = 213,6 h ÷ 43,8 h ≈ 4,9×.** Un changement sur dix met près
de neuf jours à atteindre la production, là où le cas courant en met moins de
deux.

**Ce que l'écart signale.** L'annexe B décrit ce cas de figure explicitement :

> Un P50 stable avec un P90 qui s'envole est un signal net : il existe une
> **catégorie** de changements qui coince — migrations de base de données,
> changements inter-équipes, livraisons nécessitant une validation externe.

Le mot qui porte tout est **catégorie**. Un facteur 4,9 ne décrit pas un bruit
statistique réparti au hasard : il décrit une population de changements qui suit
un chemin différent des autres. Sur un projet open source, les candidats sont
faciles à nommer : contributions externes attendant la revue d'un mainteneur,
traductions synchronisées par lots, PR de dépendances (Dependabot) fusionnées
tardivement, ou changements touchant le format de fichier `.excalidraw` qui
exigent une validation plus lourde.

**Ce que l'écart ne signale pas — et c'est la moitié de la réponse.** Il ne
signale **pas une dégradation générale** de la chaîne de livraison. Le P50 est
sain : la moitié des changements passent en moins de deux jours. L'annexe B en
tire la conséquence méthodologique : *« C'est une piste de cartographie du flux de
valeur, pas une dégradation générale. »*

L'erreur à ne pas commettre serait donc de lancer un chantier d'optimisation
globale du pipeline — automatiser davantage, accélérer la CI. Ça n'améliorerait
que les changements qui vont déjà vite. La bonne action est d'**identifier la
catégorie** qui peuple la traîne, puis de traiter cette catégorie-là. C'est le
sens de la phrase de l'annexe B : *« DORA vous dit que le lead time est de six
jours ; la cartographie vous dit où les six jours sont passés. »*

C'est aussi la justification de la règle de contrat « médiane **et** P90
systématiquement » : le P50 seul aurait laissé croire à une chaîne sans problème,
le P90 seul à une chaîne engorgée. Ni l'un ni l'autre n'est vrai.

---

### 9. La deployment frequency vous place dans quel ordre de grandeur au regard de la distribution 2024 (4.2) ? Tenez compte du chapitre 4.1 avant de conclure.

**Les repères de 4.2.** Le support ne donne des ordres de grandeur que pour le
cluster *elite* : déploiement **à la demande**, lead time **inférieur à la
journée**, change fail rate autour de 5 %, restauration en moins d'une heure.

**Confrontation.**

| | Repère *elite* (4.2) | Mesuré sur excalidraw |
|---|---|---|
| Deployment frequency | à la demande | 0,167/jour — ~1,2 par semaine |
| Change lead time (P50) | < 1 jour | 1,8 jour |

Le constat est donc : **en dessous du cluster *elite* sur les deux axes, mais du
même ordre de grandeur sur le lead time** (1,8 jour contre « moins d'un jour » —
un facteur 2, pas un facteur 100). La fréquence, elle, est loin du « à la
demande » : déployer une fois par semaine et demie n'est pas déployer quand on
veut.

**Et c'est tout ce qu'on peut dire.** Toute tentative d'aller plus loin — « donc
*high* », « donc *medium* » — serait une invention : le support ne publie pas les
frontières des trois autres clusters, et 4.1 explique pourquoi il ne le fait pas.

**Les trois raisons qui interdisent de conclure davantage :**

1. **Les frontières bougent chaque année** (4.1). Ce sont des clusters recalculés
   sur l'échantillon de l'enquête, pas des seuils. La preuve chiffrée donnée par
   le support : entre 2023 et 2024, le cluster *high* est passé de 31 % à 22 % des
   répondants et le cluster *low* de 17 % à 25 %, sans durcissement des critères.
   Un même chiffre peut donc changer de cluster d'une année sur l'autre sans que
   l'équipe ait changé quoi que ce soit. *« La performance de l'industrie n'est pas
   un objectif statique, c'est un mouvement perpétuel. »*

2. **Les données ne sont pas commensurables** (annexe B, « piège de
   comparaison »). L'enquête DORA collecte des réponses **par tranches** — « moins
   d'un jour », « entre un jour et une semaine » — déclarées par les répondants,
   pas des durées instrumentées. Notre 43,8 h sort d'un calcul sur 79 commits
   réels. Comparer les deux au chiffre près n'a pas de sens ; seuls les ordres de
   grandeur restent parlants.

3. **Une métrique ne se lit jamais seule** (4.3). Le support en donne la
   démonstration avec l'anomalie de 2024, où le cluster *medium* affichait un
   change fail rate **plus bas** que le cluster *high*. Or nous n'avons ni change
   fail rate, ni temps de restauration — trois métriques sur cinq sont à `n/a`.
   Nous ne disposons donc pas de la moitié du tableau nécessaire pour situer quoi
   que ce soit.

**Conclusion.** L'ordre de grandeur est celui d'un projet qui livre à un rythme
régulier et sans engorgement majeur, en dessous des repères *elite*. Le seul
usage légitime de ces chiffres n'est pas le classement mais la **série
temporelle** : les comparer aux mêmes chiffres d'excalidraw dans trois mois. Le
support est explicite en 4.1 — *« la seule comparaison solide est celle de votre
équipe avec elle-même dans le temps »*.

---

### 10. Le collecteur affiche aussi le délai médian entre deux déploiements. Pourquoi cette formulation est-elle préférable à la fréquence brute pour une équipe qui déploie peu (2.3) ?

Le support le dit en une puce de 2.3 :

> Pour des équipes qui déploient rarement, la fréquence brute est trompeuse.
> Préférez le **délai médian entre deux déploiements**, plus lisible et moins
> sensible aux fenêtres arbitraires.

Nos deux chiffres l'illustrent parfaitement :

| Formulation | Valeur |
|---|---|
| Fréquence brute | 0,167 déploiement/jour |
| Délai médian entre deux | 4,5 jours |

**Trois raisons, dont la troisième est la vraie.**

**1. Le premier chiffre ne décrit l'expérience de personne.** Personne ne déploie
0,167 fois par jour. C'est un artefact de division. « On met en production tous
les quatre jours et demi » est une phrase qu'un développeur reconnaît, qu'un
directeur comprend, et sur laquelle une rétrospective peut travailler. Or le
support rappelle en 2.1 que ces métriques servent à *« dire où regarder »* : un
chiffre que personne ne sait interpréter ne dit rien à personne.

**2. La fréquence brute dépend de la fenêtre, le délai médian beaucoup moins.**
0,167/jour est mécaniquement `15 ÷ 90`. Change la fenêtre pour 28 jours et le
numérateur change, mais pas proportionnellement : une semaine calme ou un jour
férié déplace le résultat. Le délai **médian**, lui, est une statistique d'ordre
sur les intervalles observés : il résiste aux valeurs extrêmes par construction.
C'est le sens de « moins sensible aux fenêtres arbitraires », et c'est la même
logique que « médiane, jamais la moyenne » de l'annexe B.

**3. Le délai médian porte l'information que la fréquence détruit : la
régularité.** Une fréquence est une moyenne, et une moyenne écrase la
distribution. 15 déploiements sur 90 jours donnent 0,167/jour **aussi bien** si
les déploiements sont espacés régulièrement de 6 jours que s'ils sont tous
concentrés sur une seule semaine suivie de deux mois de silence. Les deux
situations n'ont rien à voir : la première décrit une chaîne de livraison, la
seconde une campagne de release. Le délai médian de 4,5 jours, comparé à la
fréquence de 0,167/jour (soit un intervalle moyen de 6 jours), nous apprend
d'ailleurs quelque chose : **médiane < moyenne**, donc la distribution des
intervalles est tirée vers le haut par quelques longues pauses. La majorité des
déploiements sont plus rapprochés que 6 jours.

Cette dernière information est invisible dans la fréquence brute. C'est pour ça
que le collecteur affiche les deux lignes, l'une indentée sous l'autre.

---

### 11. Trois métriques sur cinq s'affichent `n/a`. Lesquelles ? Qu'ont-elles en commun ?

**Les trois métriques absentes :**

| Métrique | Facteur (support 1.4) |
|---|---|
| Failed deployment recovery time | Débit |
| Change fail rate | Instabilité |
| Deployment rework rate | Instabilité |

Autrement dit : **l'intégralité du facteur « instabilité de livraison », plus une
métrique de débit.** Ce n'est pas une coïncidence de périmètre — c'est la moitié
du modèle DORA qui manque.

**Ce qu'elles ont en commun.** La tentation est de répondre « il manque les
incidents ». C'est vrai mais superficiel. La bonne réponse porte sur la **nature**
de la donnée, et elle se voit en comparant avec les deux métriques qui ont
fonctionné.

| Métrique | Donnée nécessaire | Qui la produit |
|---|---|---|
| Deployment frequency | l'évènement de déploiement | Vercel, automatiquement |
| Change lead time | la date de commit | git, automatiquement |
| Change fail rate | le lien incident → déploiement causal | **un humain, délibérément** |
| Failed deployment recovery time | début et fin de la dégradation, + le même lien | **un humain, délibérément** |
| Deployment rework rate | le marquage « ce déploiement n'était pas prévu » | **un humain, délibérément** |

Les deux métriques qui marchent reposent sur des **traces** : des sous-produits
d'outils qui les émettent sans que personne y pense, pour des raisons qui n'ont
rien à voir avec DORA. Git horodate les commits parce que c'est son métier ;
Vercel enregistre un déploiement parce qu'il en a besoin pour router du trafic.
Ces données existent même dans un projet qui ignore l'existence de DORA — et
c'est pour cela qu'elles sont calculables **de l'extérieur**, sur un dépôt dont
nous ne sommes pas mainteneurs.

Les trois métriques absentes reposent sur des **déclarations** : des informations
qu'aucun outil n'émet comme effet de bord, parce qu'elles n'existent que dans la
tête de la personne qui a diagnostiqué la panne ou décidé du déploiement. « Ce
déploiement a cassé la prod » et « ce déploiement n'était pas planifié » ne sont
pas des faits techniques observables, ce sont des **jugements**.

**Le point qui rend la démonstration incontestable : ce n'est pas un problème
d'outil.** Le collecteur sait parfaitement calculer les trois. Le code est écrit,
testé, il attend :

```python
if args.incident_label and deployments:
    ...
    if linked:
        result["change_fail_rate"] = len(failing) / len(deployments)
```

Le `n/a` n'est pas un aveu d'incompétence du script, c'est un refus de calculer
sans donnée. Un outil moins scrupuleux aurait affiché `0 %` — ce qui aurait été
faux, et pire : crédible. Voir la question 17.

---

### 12. Le support désigne un maillon faible (5.1). Lequel, et pourquoi ne peut-il pas être reconstitué à partir des données publiques du dépôt ?

**Le maillon faible est le lien entre un incident et le déploiement qui l'a
causé.** Le support le formule dans l'encadré de 5.1 :

> le maillon faible n'est presque jamais le commit ni le déploiement : c'est **le
> lien entre un incident et le déploiement qui l'a causé**. Sans ce lien, deux
> métriques sur cinq sont impossibles à calculer correctement.

Nous en avons la confirmation expérimentale à la question 11, avec une aggravation :
sans ce lien, ce sont **trois** métriques sur cinq qui tombent dans notre cas, car
le `deployment_rework_rate` exige lui aussi une déclaration humaine — d'une autre
nature, mais tout aussi absente.

**Pourquoi aucun outil ne peut le déduire.**

**1. Parce que ce lien est une relation de causalité, et qu'un outil ne voit que
des dates.** C'est la raison de fond. Une API expose des horodatages ; la
causalité n'est pas un horodatage. Entre « l'issue #9842 a été ouverte trois
heures après le déploiement 6374481865 » et « le déploiement 6374481865 a causé
l'issue #9842 », il y a un **diagnostic** : quelqu'un a lu la stack trace, relu le
diff, reproduit le bug, et conclu. Cette conclusion est un travail intellectuel,
pas une donnée dérivable. Aucun volume de données publiques ne la contient.

**2. Parce que toute heuristique de substitution est fausse par construction.**
La seule qu'un outil puisse tenter est la proximité temporelle — et la question 13
montre qu'elle se trompe dans les deux sens. Le problème n'est pas qu'elle soit
imprécise, c'est qu'elle **inverse le sens de l'inférence** : elle déduit une
cause d'une coïncidence. Sur un projet à 100 déploiements par trois jours et des
centaines d'issues, les coïncidences sont garanties.

**3. Parce que la donnée manquante n'est pas « non collectée », elle est
« jamais produite ».** Vérifications sur excalidraw : aucun label `incident`
n'existe dans le dépôt, aucune convention `caused_by` n'apparaît dans les corps
d'issues, aucun template d'issue ne demande le déploiement en cause. Il ne s'agit
donc pas d'une donnée présente que nous ne saurions pas extraire — la
reconstitution n'est pas un problème d'accès ou de parsing. **L'information n'a
jamais existé sous forme écrite, nulle part.** Elle est restée dans la tête des
mainteneurs qui ont corrigé les bugs.

**4. Parce que le projet n'a aucune raison de la produire.** Et c'est le point le
plus intéressant pour un futur lead dev. Excalidraw n'est pas négligent : il n'a
simplement pas d'astreinte, pas de SLA, pas de client qui appelle à 3 h du matin.
Le lien incident → déploiement n'a de valeur que pour une organisation qui exploite
sa production. Un projet open source n'en tire aucun bénéfice opérationnel — il
n'y a donc personne pour payer le coût de la saisie.

**Conséquence pratique, et c'est le point d'arrivée du TD.** La seule façon
d'obtenir ces métriques est de **produire la donnée**, c'est-à-dire d'instaurer
une convention et de la faire respecter par des humains à chaque incident. C'est
l'objet de la phase 4 — et c'est pourquoi le support classe le
`deployment_rework_rate` comme *« la métrique DORA la plus sensible à la
discipline de saisie »* (2.6, 2.8). Pas à la qualité de l'outillage : à la
discipline.

---

### 13. Un collègue propose : un déploiement suivi d'un autre moins de 24 h après est un échec. Donnez deux situations où cette règle se trompe, une dans chaque sens.

La règle tente de remplacer le maillon faible de la question 12 par une
heuristique de proximité temporelle. Voici comment elle échoue dans chaque sens.

#### Sens 1 — Faux positif : deux déploiements rapprochés, aucun échec

**Situation : deux changements planifiés fusionnés le même jour.**

Une PR de fonctionnalité est fusionnée le matin et déployée ; une PR de traduction
ou de correction de documentation est fusionnée l'après-midi et déployée à son
tour. Les deux étaient prévus, aucun n'a rien cassé. La règle compte pourtant un
échec — et elle compte un échec **sur le premier déploiement**, qui est précisément
celui qui s'est bien passé.

Sur excalidraw, l'occasion est fréquente : 15 déploiements pour un intervalle
médian de 4,5 jours mais une moyenne de 6 jours, ce qui signale (voir question 10)
une distribution où des déploiements se regroupent puis laissent place à de
longues pauses. Chaque regroupement produirait un faux échec.

**Ce qui rend ce faux positif grave, et pas seulement gênant.** La règle pénalise
exactement la pratique que DORA recommande le plus fortement. Une équipe qui
applique le développement sur tronc commun et les petits lots (6.1) déploie
**plusieurs fois par jour** par conception. Sous cette règle, elle afficherait un
change fail rate proche de 100 %, tandis qu'une équipe déployant une fois par mois
afficherait 0 %. La règle mesure donc la **cadence** et la présente comme de
l'**instabilité** — soit exactement l'inverse du résultat de 1.1 (les équipes
performantes sont bonnes sur les deux axes simultanément).

**Et elle se retourne contre le produit si l'équipe la connaît.** C'est la loi de
Goodhart (7.1.1) en action : le moyen le plus simple d'améliorer ce taux est
d'**attendre 24 heures avant de déployer un correctif**. La métrique s'améliore,
les utilisateurs restent en panne un jour de plus. Le support le formule ainsi :
*« Dès qu'une mesure porte un enjeu, l'effort se déplace vers ce qui est mesuré. »*

#### Sens 2 — Faux négatif : un vrai échec que la règle ne voit pas

**Situation : un incident détecté tardivement.**

Un déploiement introduit une régression dans une fonctionnalité peu utilisée — par
exemple l'export d'un format de fichier particulier. Personne ne le remarque
immédiatement. Un utilisateur ouvre une issue trois jours plus tard, le correctif
est déployé le quatrième jour. L'intervalle entre le déploiement fautif et le
déploiement réparateur dépasse largement 24 h : la règle ne voit rien. C'est
pourtant un change failure au sens plein de 2.5, et l'incident a duré quatre jours.

Le paradoxe mérite d'être souligné : **plus l'observabilité d'une équipe est
mauvaise, plus son change fail rate paraît bon sous cette règle.** C'est
littéralement la ligne de mise en garde de 7.3 — *« Fail rate à 0 % affiché
fièrement → vous ne détectez peut-être pas vos incidents »* — transformée en
algorithme.

**Variante, encore plus invisible : la réparation sans déploiement.**

Toutes les réparations ne passent pas par un déploiement, et la règle ne voit que
des déploiements. Trois cas courants :

- **désactivation d'un feature flag** — le support signale en 2.3 que déployé ≠
  publié ; couper un flag rétablit le service sans produire le moindre évènement
  de déploiement ;
- **rollback par promotion d'un build antérieur** — chez Vercel, revenir à la
  version précédente republie un artefact déjà construit, ce qui ne ressemble pas
  à un nouveau déploiement de code ;
- **correctif hors du dépôt** — un paramètre d'infrastructure, une règle de
  sécurité Firebase, une purge de CDN.

Dans les trois cas, l'incident est réel, la panne a été subie par des
utilisateurs, et l'API Deployments ne contient **aucune** trace de la réparation.
La règle renvoie « pas d'échec » avec une parfaite assurance.

#### Ce que les deux sens ont en commun

La règle ne mesure pas ce qu'elle croit mesurer. Elle mesure **l'espacement des
déploiements**, et l'appelle taux d'échec. Elle fabriquerait donc un chiffre
faux, exploitable dans un tableau de bord, discutable en réunion, impossible à
invalider — ce qui est strictement pire qu'un `n/a`. C'est la différence entre ne
pas savoir et croire savoir.

Comme l'écrit le sujet à la fin de la phase 3 : *« Un outil rend toujours un
chiffre. Ce chiffre peut être le symptôme d'un marqueur qui a cessé d'être
appliqué, et non la mesure d'une réalité. »*

---

## Phase 3 — Le proxy et ses limites

### Commande exécutée

```bash
python3 outils/dora_metrics.py \
  --repo excalidraw/excalidraw \
  --git ../../excalidraw \
  --environment "Production – excalidraw" \
  --window 90 \
  --incident-label bug
```

Le bloc ajouté à la sortie :

```
INCIDENTS
  Issues "bug"                         23
    rattachées à un déploiement        0
```

Les métriques de débit sont inchangées, et `Change fail rate` reste à `n/a`.

---

### 14. Combien d'issues `bug` le collecteur trouve-t-il sur la fenêtre ? Combien sont rattachées à un déploiement ?

**23 issues trouvées, 0 rattachée à un déploiement.**

Mais ces deux nombres méritent d'être qualifiés, car le premier ne compte pas ce
que son intitulé annonce.

**Le « 23 » n'est pas un compte d'incidents de la fenêtre.** Le collecteur
interroge l'API avec :

```python
{"labels": label, "state": "all", "since": since.isoformat(), ...}
```

Or dans l'API Issues de GitHub, le paramètre `since` filtre sur **`updated_at`**,
pas sur `created_at`. Les 23 issues remontées ont donc été créées **entre 2020 et
2025** et seulement *touchées* pendant les 90 jours — un commentaire, un
changement de label, une mention croisée.

| Issue | Créée le | Mise à jour le |
|---|---|---|
| #1990 | 2020-07-30 | 2026-06-25 |
| #4863 | 2022-03-03 | 2026-08-19 |
| #7332 | 2023-11-23 | 2026-09-07 |
| #9757 | 2025-07-17 | 2026-07-01 |

**Aucune des 23 n'a été créée dans la fenêtre.** Ce que le collecteur appelle
« Issues "bug" » sur 90 jours est en réalité « vieux bugs encore actifs ». La
ligne du tableau est exacte au sens du code et fausse au sens de son libellé.

**Le « 0 », en revanche, est solide** : `caused_by` n'apparaît dans aucun des 23
corps d'issues. Ce n'est pas un défaut de parsing, la convention n'existe pas
dans ce projet.

C'est un cinquième écart guide/réalité à consigner, et le plus instructif : même
un outil écrit pour un TD sur les pièges de mesure en contient un. Un intitulé de
colonne n'est pas une définition.

---

### 15. Dans l'interface GitHub : combien d'issues toutes catégories ont été ouvertes sur ces 90 jours ? Combien portent le label `bug` depuis la création du dépôt ?

Recherches lancées sur le dépôt **amont** — le fork n'a pas d'onglet Issues, GitHub
désactivant le suivi d'issues sur les forks (conforme à l'étape 1.1 du guide :
un fork ne copie pas les issues).

| Requête | Compteur |
|---|---|
| `is:issue state:all label:bug` (depuis la création du dépôt) | **765** |
| `is:issue state:all created:>=2026-06-13` (toutes catégories, 90 j) | **141** |
| `is:issue state:all label:bug created:>=2026-06-13` (90 j) | **0** |

La troisième requête n'était pas demandée explicitement mais elle est le
croisement qui rend la question 16 possible.

**Complément relevé pour dater le phénomène.** Les issues `bug` les plus
récentes, triées par date de création :

| Issue | Créée le |
|---|---|
| #10948 | 2026-03-13 |
| #10642 | 2026-01-12 |
| #10628 | 2026-01-08 |
| #10599 | 2026-01-04 |
| #9934 | 2025-09-02 |

**La dernière issue étiquetée `bug` date du 13 mars 2026**, soit trois mois avant
le début de notre fenêtre. Et le rythme s'effondrait déjà avant : quatre en
janvier 2026, rien en février, une en mars, plus rien pendant six mois. Ce n'est
pas un arrêt net mais un **abandon progressif**.

---

### 16. Confrontez les trois nombres. Que s'est-il passé dans ce projet ?

| Nombre | Ce qu'il établit |
|---|---|
| **765** `bug` depuis toujours | le label a été massivement utilisé pendant des années |
| **141** issues sur 90 jours | le projet est très vivant, le signalement fonctionne |
| **0** `bug` sur 90 jours | plus personne n'applique le label |

**Le diagnostic s'obtient par élimination**, et c'est pour cela qu'il faut les
trois nombres — aucun ne suffit seul.

- Si le label n'avait jamais servi, le premier nombre serait faible. Il est de
  765 : l'hypothèse « convention jamais adoptée » est écartée.
- Si le projet s'était arrêté, le deuxième serait faible. Il est de 141 sur
  90 jours, soit environ 1,6 issue ouverte par jour : l'hypothèse « dépôt mort »
  est écartée.
- Il ne reste qu'une explication compatible avec les trois : **la convention de
  marquage a été abandonnée en cours de route, alors que l'activité du projet, et
  le flux de signalements, se poursuivaient sans faiblir.**

La date de bascule est le 13 mars 2026, avec une décroissance amorcée dès
l'automne 2025. On ignore pourquoi — changement de processus de triage, migration
vers un autre système de labels, ou simple lassitude. Ce n'est pas observable de
l'extérieur, et ça n'a pas d'importance pour la conclusion.

**Ce que cela change pour la mesure.** Le taux d'échec calculé sur ce proxy ne
serait pas « imprécis » ou « approximatif » : il serait **un artefact sans
rapport avec la réalité mesurée**. Un `0 %` affiché ici ne dirait rien sur la
qualité des déploiements d'excalidraw. Il dirait uniquement qu'un label a cessé
d'être coché.

Et c'est là que la mesure devient dangereuse plutôt qu'inutile : **rien dans la
donnée ne signale cette rupture.** Le collecteur a rendu « 23 » sans broncher.
L'anomalie n'apparaît qu'en comparant la fenêtre à l'historique — c'est-à-dire en
faisant une chose qu'aucun tableau de bord ne fait automatiquement.

---

### 17. Le chapitre 7.3 énumère trois explications à un taux d'échec de 0 %. Aucune ne décrit ce cas : formulez la quatrième.

**Les trois explications du support** (7.3, développement de la dernière ligne du
tableau) :

> Soit vous **déployez trop rarement** pour que la métrique ait un sens, soit vous
> **ne détectez pas vos dégradations**, soit vos **incidents ne sont pas rattachés
> aux déploiements**.

**Vérification, une par une :**

1. *Déploiements trop rares.* Écarté. 15 déploiements de production sur la
   fenêtre : le dénominateur existe et il est suffisant pour qu'un taux ait un
   sens. La fréquence est modeste (0,167/jour) mais ce n'est pas le cas limite
   que le support décrit.

2. *Dégradations non détectées.* Écarté, et même à l'opposé. 141 issues ouvertes
   en 90 jours : la détection est non seulement fonctionnelle mais intense. Les
   utilisateurs signalent, les mainteneurs reçoivent.

3. *Incidents non rattachés aux déploiements.* C'est le plus proche, mais il ne
   décrit pas notre cas. Cette explication suppose qu'il **existe** des incidents
   identifiés comme tels, et qu'il manque seulement le lien vers leur cause. Ici,
   il n'y a **aucun incident identifié** : on ne peut pas manquer un
   rattachement pour une population vide. Le défaut est un cran plus en amont.

**La quatrième explication :**

> **Le marqueur qui servait à identifier les incidents a cessé d'être appliqué,
> alors que l'outil continue de l'interroger.** Le taux n'est alors ni une mesure
> de la production, ni un défaut d'attribution : c'est le symptôme d'une
> **instrumentation morte**. Le label existe toujours, la requête réussit
> toujours, le chiffre se calcule toujours — seule la convention humaine derrière
> a disparu, et rien dans la donnée ne l'annonce.

**Pourquoi cette quatrième est plus insidieuse que les trois autres.** Les trois
causes du support laissent une trace détectable dans la donnée elle-même : un
dénominateur minuscule se voit, une absence totale d'issues se voit, des
incidents orphelins se voient. Ici, tous les indicateurs de surface sont sains —
le dépôt est actif, l'API répond, l'outil ne signale aucune erreur. Il n'existe
qu'un seul moyen de découvrir le problème : **comparer la fenêtre de mesure à
l'historique long du même marqueur.** Aucun tableau de bord ne fait ça
spontanément, puisque son rôle est précisément de ne montrer que la fenêtre
courante.

**Corollaire opérationnel** — et c'est ce que je retiens du TD pour ma pratique :
une instrumentation DORA a besoin d'une **surveillance de sa propre
instrumentation**. Pas seulement « quel est mon change fail rate », mais aussi
« combien d'incidents ai-je déclarés ce mois-ci par rapport aux précédents ? ».
Une chute à zéro de ce second indicateur est une alerte sur le processus, pas une
bonne nouvelle sur le produit.

Cela rejoint la maxime du support, à laquelle notre cas ajoute une nuance :

> **Mentra :** Si c'est parfait, c'est suspect → vérifier, c'est la clé.

Ici, « vérifier » ne veut pas dire vérifier la production. Cela veut dire
**vérifier que le capteur est encore branché**.

---

### 18. Quelle métrique le support décrit-il comme la plus sensible à la discipline de saisie (2.8) ? Ce rapprochement vous paraît-il fortuit ?

**Le `deployment rework rate`.** Le chapitre 2.8 le formule ainsi :

> le deployment rework rate est la métrique DORA la plus sensible à la
> **discipline de saisie** et à la qualité de vos dépendances.

Et 2.6 en donne la raison : *« il faut un moyen fiable de marquer un déploiement
comme "non planifié" (label `hotfix` sur la PR, branche `hotfix/*`, champ dédié
dans le pipeline). Décidez de la convention **avant** de collecter. »*

**Le rapprochement n'est pas fortuit. Il est structurel.**

Nous venons d'assister exactement à ce que cette phrase décrit : une donnée qui
disparaît non par défaut d'outillage, mais par **abandon d'une convention
humaine**. Le collecteur fonctionne, l'API répond, le code du calcul est écrit —
et le résultat est `n/a` parce que personne ne coche plus rien.

**Ce qui rend la fragilité structurelle et pas accidentelle** apparaît en
comparant, pour chaque métrique, *pourquoi la donnée qu'elle consomme existe* :

| Métrique | La donnée existe parce que… | Survit à l'inattention ? |
|---|---|---|
| Deployment frequency | Vercel doit router du trafic | ✅ oui |
| Change lead time | git horodate les commits par conception | ✅ oui |
| Change fail rate | quelqu'un a diagnostiqué et écrit la cause | ❌ non |
| Failed deployment recovery time | idem, plus les bornes de la dégradation | ❌ non |
| Deployment rework rate | **quelqu'un a nommé sa branche `hotfix/`** | ❌ non |

Les deux premières lignes décrivent des **sous-produits** : la donnée existe pour
des raisons qui n'ont rien à voir avec DORA, et elle continuerait d'exister même
si toute l'équipe oubliait le mot « DORA » demain. Les trois dernières décrivent
des **déclarations** : elles n'existent que par un acte volontaire et répété.

Et le `rework rate` est le cas extrême de la série. Un préfixe de branche
`hotfix/` ne sert **rien d'autre** que la mesure. Il n'apporte aucun bénéfice
technique immédiat à celui qui le tape : le code se comporte identiquement avec
`fix/`, `urgent/` ou `patch-3`. C'est un pur coût de discipline, payé par une
personne, au profit d'une statistique qu'elle ne lira peut-être jamais. C'est le
marqueur le plus facile du monde à laisser tomber un jour de crise — précisément
le jour où il compterait le plus.

**Une observation qui aggrave le constat.** Le label `bug` d'excalidraw, lui,
avait *deux* raisons d'exister : le triage quotidien des mainteneurs, **et** une
mesure potentielle. Il était donc mieux protégé qu'un préfixe `hotfix/` — il
rendait un service au-delà de la métrique. Il est mort quand même, après 765
usages.

Si un marqueur à double usage ne survit pas, un marqueur à usage unique n'a
aucune chance sans un dispositif qui le soutient : un template d'issue qui exige
le champ, une vérification en CI, une revue qui le réclame. **La discipline ne se
demande pas, elle s'outille.** C'est aussi ce que la phase 4 va coûter en temps :
le prix n'est pas dans le calcul, il est dans la production de la donnée.

---

## Phase 4 — Produire la donnée manquante

### Ce qui a été mis en place sur le fork

**1. Le workflow `.github/workflows/deploy.yml`** (adapté du chapitre 5.3), avec
trois écarts assumés par rapport à l'exemple du support :

| Écart | Raison |
|---|---|
| `branches: [master, 'hotfix/**']` au lieu de `[main]` | `master` est la branche par défaut d'excalidraw ; `hotfix/**` est indispensable pour que le rework soit mesurable |
| `ref: context.ref.replace('refs/heads/','')` au lieu de `context.sha` | le collecteur lit `d["ref"].startswith("hotfix/")` : avec un sha, le rework rate resterait à `n/a` |
| pas de `environment: production` au niveau du job | GitHub crée automatiquement un déploiement pour tout job déclarant un `environment` : combiné à notre `createDeployment` explicite, chaque push produirait **deux** déploiements |

**2. Cinq déploiements**, dont un issu d'une branche `hotfix/` :

| id | ref | horodatage (UTC) |
|---|---|---|
| 6390814188 | `master` | 10:01:52 |
| 6390943877 | `master` | 10:10:27 |
| 6390985390 | `master` | 10:13:10 |
| 6391006770 | `master` | 10:14:36 |
| 6391028640 | `hotfix/correctif-urgent` | 10:16:04 |

**3. Un incident rattaché.** Issue #1, label `incident`, corps contenant
`caused_by: 6391006770` — le déploiement 4 est désigné comme fautif, le
déploiement `hotfix/` étant le correctif. Ouverte à 10:19:07, fermée à 10:22:08,
soit **3 minutes**.

### Note sur la stabilité de ce relevé

Les chiffres ci-dessous ont été relevés **avant** le commit de ce rendu. Le
workflow se déclenchant sur tout push vers `master`, le commit qui publie ce
fichier produit un **sixième** déploiement : le change fail rate passe à 1/6
≈ 16,7 % et le rework rate à 1/6 ≈ 16,7 %.

Ce n'est pas un défaut du relevé, c'est une propriété de la mesure : sur une
fenêtre de 15 minutes contenant cinq déploiements, **l'acte de documenter la
mesure modifie la mesure**. C'est une illustration supplémentaire de la question
21 — un dénominateur de cette taille n'a aucune stabilité.

### Résultat du collecteur sur le fork

```bash
python3 outils/dora_metrics.py \
  --repo ThomasdevLC/excalidraw \
  --git ../../excalidraw \
  --environment production \
  --window 90 \
  --incident-label incident \
  --rework-prefix hotfix/
```

```
DÉBIT
  Deployment frequency                 0.056 /jour  (5 déploiements)
    délai médian entre deux            0.0 h
  Change lead time (P50)               0.0 h
  Change lead time (P90)               0.0 h
    base de calcul                     4 commits / 4 lots
  Failed deployment recovery time      0.1 h

INSTABILITÉ
  Change fail rate                     20.0 %
  Deployment rework rate               20.0 %

INCIDENTS
  Issues "incident"                    1
    rattachées à un déploiement        1
```

---

### 19. Que pouvez-vous calculer maintenant que vous ne pouviez pas calculer avant ?

**Les trois métriques qui étaient à `n/a` :**

| Métrique | Amont | Fork | Ce qui l'a rendue calculable |
|---|---|---|---|
| Failed deployment recovery time | `n/a` | **0,1 h** (3 min) | l'issue porte une date d'ouverture, une date de clôture **et** un `caused_by` |
| Change fail rate | `n/a` | **20,0 %** (1/5) | le `caused_by` désigne un déploiement de la fenêtre |
| Deployment rework rate | `n/a` | **20,0 %** (1/5) | le déploiement porte `ref = "hotfix/correctif-urgent"` |

Le bloc incidents bascule lui aussi : **1 trouvée / 1 rattachée**, contre 23 / 0
sur l'amont.

**Ce qui a changé n'est ni l'outil, ni le dépôt, ni la méthode de calcul.** C'est
le même script, la même commande, le même algorithme. Trois différences
seulement, et toutes les trois sont des **écritures humaines délibérées** :

1. une convention de nommage de branche appliquée (`hotfix/correctif-urgent`) ;
2. un label posé sur une issue (`incident`) ;
3. **une ligne de texte saisie à la main : `caused_by: 6391006770`.**

La troisième est le maillon faible du chapitre 5.1, matérialisé. Elle fait
basculer deux métriques sur cinq à elle seule. Sans elle, le collecteur trouve
l'issue (`incidents_found = 1`) mais ne peut rien en faire (`incidents_linked = 0`),
et retombe exactement sur le comportement observé en phase 3.

**La démonstration du TD tient dans cet écart.** Le passage de deux métriques à
cinq n'a rien coûté en technologie et tout coûté en **convention**. Aucun outil,
aussi sophistiqué soit-il, ne pouvait deviner que le déploiement 6391006770 avait
causé l'issue #1 : les deux évènements sont séparés de 4 minutes, mais quatre
autres déploiements le sont autant, et la proximité temporelle n'établit pas la
causalité (voir question 13). Quelqu'un devait l'écrire. Je l'ai écrit.

---

### 20. Combien de temps a demandé la production de cette donnée, comparé au temps passé à tenter de la déduire en phase 3 ?

**Le décompte honnête.**

| Phase | Activité | Temps | Résultat |
|---|---|---|---|
| Phase 3 | lancer le collecteur avec `--incident-label bug`, puis 3 recherches GitHub, puis interpréter | ~15 min | **aucune métrique** — trois `n/a`, et un « 23 » trompeur |
| Phase 4 | workflow, 5 déploiements, label, issue, mesure | ~45 min | **les cinq métriques** |

Soit un rapport d'environ **1 à 3** en temps. Mais ce rapport brut est le chiffre
le moins intéressant de la réponse, pour trois raisons.

**1. La phase 3 n'a pas produit une donnée imparfaite : elle n'a rien produit.**
Comparer 15 minutes et 45 minutes suppose que les deux démarches livrent des
résultats comparables en nature. Ce n'est pas le cas. Le temps de la phase 3 n'est
pas un investissement moins cher, c'est une perte sèche — sauf comme
apprentissage. Une heure de plus n'y aurait rien changé : aucune quantité
d'analyse ne fait apparaître une donnée qui n'a jamais été écrite.

**2. L'essentiel des 45 minutes n'est pas le coût de la donnée, c'est le coût des
frictions.** Trois incidents de parcours, dont aucun ne concerne DORA :

| Friction | Nature | Coût |
|---|---|---|
| jeton d'accès sans la portée `workflow` | GitHub refuse qu'un jeton crée un fichier dans `.github/workflows/` sans portée dédiée | ~10 min, contournée en passant par l'éditeur web |
| `context.ref_name` inexistant dans `github-script` | l'objet `context` expose `ref` (`refs/heads/master`), pas `ref_name` — `ref_name` est une expression de workflow, pas une propriété JS | ~5 min, erreur `"ref" wasn't supplied` |
| `caused_by:` saisi dans l'éditeur web, enregistré en `**caused_by** 6391006770` | l'éditeur a ajouté des astérisques et supprimé les deux-points | ~5 min |

Le cœur du travail — écrire le workflow, produire les déploiements, ouvrir
l'issue — représente moins de 20 minutes. Le reste est du coût
d'apprentissage d'outillage, qui ne se repaie pas à chaque incident.

**3. Le vrai coût n'est pas le coût d'installation, c'est le coût récurrent.** Et
c'est là que la comparaison devient éclairante pour une équipe réelle :

| | Coût unique | Coût par incident |
|---|---|---|
| Le workflow d'émission | ~15 min, une fois | 0 — il tourne tout seul |
| La convention `hotfix/` | 0 | ~0 — il suffit de nommer sa branche |
| **La ligne `caused_by:`** | 0 | **~30 secondes, à chaque incident, par un humain** |

Trente secondes par incident. C'est tout ce que le change fail rate et le failed
deployment recovery time coûtent réellement. Et c'est exactement ce
qu'excalidraw n'a jamais payé — non par négligence, mais parce que le projet n'en
retire aucun bénéfice opérationnel (voir question 12).

**Ce que je retiens pour ma pratique.** L'arbitrage n'est pas « déduire ou
produire », parce que déduire est impossible. Il est « instrumenter maintenant, ou
ne jamais disposer de l'historique ». Une instrumentation posée aujourd'hui
produit une série exploitable dans trois mois ; une instrumentation posée dans
trois mois ne dit rien sur les trois mois écoulés. **Les données de livraison ne
sont pas rétroactives.** C'est un argument d'antériorité, pas un argument de coût
— et c'est celui qu'il faut porter devant une direction qui veut « d'abord voir
si ça sert ».

L'incident des astérisques mérite d'être relevé à part, parce qu'il illustre le
chapitre 2.6 mieux que n'importe quel exemple théorique : le contrat était écrit,
l'outil était prêt, le déploiement existait, l'issue était correctement
étiquetée — et la métrique restait à `n/a` **à cause de deux caractères de
ponctuation**. Une convention de saisie qui repose sur la mémoire d'un humain
face à un éditeur qui reformate son texte n'est pas une convention, c'est un
espoir. Il lui faut un support technique : un template d'issue avec un champ
dédié, un formulaire GitHub (`issue forms`), ou une vérification en CI qui refuse
une issue `incident` sans `caused_by` valide.

---

### 21. Votre change fail rate est-il représentatif ? Que faudrait-il pour qu'il le devienne ?

**Non, il ne l'est pas. Il n'a aucune valeur descriptive.**

Le chiffre est **20 %**, et il est arithmétiquement exact : 1 déploiement fautif
sur 5. Mais il ne décrit rien du tout, pour six raisons cumulatives.

**1. L'échantillon est trop petit pour qu'un pourcentage ait un sens.** Avec 5
déploiements, la granularité de la métrique est de 20 points : les seules valeurs
possibles sont 0, 20, 40, 60, 80 ou 100 %. Un incident de plus ou de moins fait
bouger le taux de 20 points. Le support (7.3) écarte les fenêtres de 7 jours pour
« bruit ingérable » ; ici le problème n'est pas la fenêtre, c'est le dénominateur.

**2. L'incident est inventé.** Aucune dégradation ne s'est produite : le
déploiement exécute `echo "déploiement"`. J'ai décidé du numérateur en écrivant
l'issue. Un taux d'échec dont l'opérateur choisit le numérateur mesure les
intentions de l'opérateur.

**3. Les déploiements sont vides.** Cinq `echo` dans un runner, sur des commits
qui ajoutent une ligne à `NOTES.md`. Aucune surface de changement, donc aucun
risque réel — le dénominateur est aussi fictif que le numérateur. C'est
littéralement le premier piège de 7.1 : *« déploiements vides pour gonfler le
compteur »*, ici pour peupler un denominateur.

**4. La fenêtre est de 15 minutes, pas de 90 jours.** Le collecteur annonce
`--window 90`, mais les cinq déploiements se sont produits entre 10:01 et 10:16
le même jour. Les deux chiffres qui l'exposent sont le `Change lead time P50` et
le `délai médian entre deux` à **`0.0 h`** : les commits ont été créés quelques
secondes avant leur déploiement, et les déploiements sont espacés de 1 à 8
minutes. Une métrique de débit à zéro n'est pas un excellent score, c'est le
signe que l'échelle de temps mesurée n'existe pas.

**5. Le temps de restauration de 3 minutes ne mesure pas une restauration.** Il
mesure l'intervalle entre deux de mes clics. Le support (2.4) exige que le
chronomètre démarre au début de la dégradation *détectée* ; ici il n'y a ni
dégradation, ni détection.

**6. Le rework rate de 20 % ne décrit pas du retravail.** La branche
`hotfix/correctif-urgent` ne corrige rien : elle ajoute la ligne
`correctif urgent` à `NOTES.md`. Le support (2.6) distingue les déploiements
« qui cassent » et ceux « qui réparent » ; ici aucun des deux ne se produit.

**Ce que le chiffre mesure réellement : la chaîne d'instrumentation, pas la
production.** Et c'est son seul usage légitime. Il établit que le workflow émet,
que le label est lu, que la regex `caused_by` matche, que les dénominateurs sont
correctement peuplés. C'est un **test de bout en bout de la mesure**, et à ce
titre il est réussi.

**Ce qu'il faudrait pour qu'il devienne représentatif.**

*Conditions sur les données :*

1. **Une production réelle avec de vrais utilisateurs.** Sans quoi il n'y a pas
   de dégradation possible, donc rien à mesurer.
2. **Des déploiements qui portent un changement réel** — le dénominateur doit
   compter des mises en production, pas des `echo`.
3. **Un volume suffisant sur la fenêtre.** À 5 % de taux d'échec (repère *elite*
   de 4.2), il faut plusieurs dizaines de déploiements pour que le taux soit
   autre chose qu'un bruit binaire. Le support recommande 28 ou 90 jours
   glissants (7.3) : encore faut-il que la fenêtre contienne des déploiements
   étalés sur toute sa durée.

*Conditions sur le processus — les plus difficiles :*

4. **Une détection indépendante de la bonne volonté.** Alerting, supervision,
   remontées utilisateurs tracées. Sans elle, le taux mesure la vigilance de
   l'équipe, et un `0 %` signifie « nous n'avons rien vu » (7.3).
5. **Une déclaration systématique du `caused_by`,** y compris les jours de crise
   — donc outillée, pas confiée à la mémoire (voir question 20). L'incident des
   astérisques montre que même avec la convention écrite et l'intention présente,
   la saisie échoue.
6. **Aucun enjeu d'évaluation attaché au chiffre.** C'est l'interdit de 7.2, et
   c'est une condition de *validité*, pas de politesse : dès que le taux porte un
   enjeu de carrière, les incidents sont requalifiés en « maintenance planifiée »
   et le chiffre s'améliore sans que rien ne change. La condition 5 et la
   condition 6 sont d'ailleurs en tension : plus on outille la saisie, plus le
   chiffre est fiable, et plus il devient tentant de s'en servir pour évaluer.

*Et une condition d'interprétation :*

7. **Ne jamais le lire seul** (4.3). Le support en donne la démonstration avec
   l'anomalie de 2024, où le cluster *medium* affichait un change fail rate plus
   bas que le cluster *high*. Un taux de 20 % n'est ni bon ni mauvais : il dépend
   entièrement de la fréquence de déploiement et du temps de restauration qui
   l'accompagnent.

**Conclusion.** Il est plus simple de rendre un change fail rate *calculable* que
de le rendre *représentatif*. Le calculable a demandé 45 minutes et une ligne de
texte. Le représentatif demande une production, du volume, de l'observabilité, une
discipline outillée et une garantie politique — c'est-à-dire une organisation, pas
un script. C'est exactement ce que dit le chapitre 5.5 en plaçant l'outillage en
dernier.

---

## Phase 5 — Lecture critique des outils

### Relevé au 11 septembre 2026

| Outil | Dernière version publiée | Dernier commit | État |
|---|---|---|---|
| **Apache DevLake** | `v1.0.3-beta16` — 27/08/2026 | 07/09/2026 | actif |
| **Middleware** | `0.3.1` — 30/05/2025 | 03/08/2026 | dépôt actif, versions figées |
| **Four Keys** | `v1.0.2` — 04/05/2023 | 23/01/2024 | **archivé, lecture seule** |

**Trois observations au-delà de ce que demandait le relevé :**

1. **`apache/incubator-devlake` n'existe plus sous ce nom.** L'API répond
   `Moved Permanently` : le dépôt est devenu `apache/devlake` après sortie
   d'incubation. Le support (5.5) et le guide pointent encore l'ancienne URL.
   Elle redirige, donc rien ne casse — mais c'est un indice supplémentaire sur la
   fraîcheur des références écrites.

2. **DevLake n'a publié aucune version stable depuis 18 mois.** Ses trois
   dernières publications sont `v1.0.3-beta16`, `beta15` et `beta14`. Le projet
   est très actif (dernier commit il y a quatre jours) mais publie en continu
   sans stabiliser. « Actif » et « stable » sont deux propriétés distinctes, et
   l'étiquette du sujet mérite cette nuance.

3. **L'écart de Middleware est spectaculaire** : dernier commit il y a cinq
   semaines, dernière version il y a quinze mois. Qui installe par release prend
   un outil vieux d'un an et demi ; qui installe depuis la branche principale
   prend du code jamais publié. Aucune des deux options n'est confortable.

---

### 22. Ces outils calculeraient-ils le change fail rate d'excalidraw ? Sur quelle donnée s'appuieraient-ils ? Votre conclusion de la phase 2 change-t-elle parce que l'outil est professionnel plutôt qu'un script de 200 lignes ?

**Non, aucun des trois ne le calculerait. Et non, ma conclusion ne change pas.**

**Sur quelle donnée s'appuieraient-ils ?** Sur les mêmes trois flux que le
chapitre 5.1 : commits, déploiements, incidents. DevLake, Middleware et Four Keys
sont des collecteurs et des agrégateurs : ils se branchent sur GitHub, GitLab,
Jira, Jenkins, PagerDuty, normalisent les évènements et calculent. Aucun des
trois n'invente de source. DevLake se distingue par le **nombre** de connecteurs,
Middleware par sa simplicité de démarrage sur GitHub/GitLab — mais leur
dépendance est identique à celle du script du TD.

**Donc, face à excalidraw, ils rencontreraient exactement le même mur :** le
dépôt ne contient aucun incident déclaré, aucun label `incident`, et aucune
convention de rattachement à un déploiement. Le numérateur du change fail rate
n'existe pas dans la donnée. Un outil ne fabrique pas un numérateur absent.

**La question de fond est la troisième**, et elle vaut plus que les deux autres.
Ma conclusion de la phase 2 ne change pas, et c'est important de dire pourquoi
elle ne *peut pas* changer : la limite rencontrée n'est pas une limite
d'implémentation, c'est une limite **d'existence de la donnée**. Aucune
sophistication n'y répond, parce qu'il n'y a rien à sophistiquer. Le chapitre 5.1
ne dit pas « le maillon faible est un problème technique difficile », il dit que
c'est un problème de **production d'information** — et on l'a vérifié en phase 4 :
ce qui a débloqué les trois métriques n'était pas un meilleur outil, c'était une
ligne de texte écrite à la main.

**Le risque réel, avec un outil professionnel, est inverse de celui du script.**
Notre collecteur affiche `n/a` — un refus honnête de calculer sans donnée. Un
outil doté d'un tableau de bord Grafana affiche des tuiles, des jauges et des
courbes. Face à une source vide, la question devient : cette tuile affiche-t-elle
« pas de donnée », ou « 0 % » ? Le support prévient en 7.3 qu'un taux de 0 %
affiché fièrement est une alerte, pas un résultat — et un affichage soigné rend
cette alerte considérablement plus difficile à voir. Plus l'outil est crédible,
plus son silence est audible comme une affirmation.

**Ce que les outils professionnels apportent réellement**, et qui est hors du
périmètre de la question : l'historisation (notre script recalcule tout à chaque
exécution, il ne conserve rien), le multi-sources, la gestion du quota d'API, et
le partage des mêmes chiffres entre équipes. Ce sont des gains d'exploitation
réels. Ce ne sont pas des gains de **mesurabilité**.

**Formulation courte pour la restitution :** un meilleur thermomètre ne crée pas
de fièvre. Si personne n'a pris la température, aucun instrument ne la retrouvera.

---

### 23. Le chapitre 5.5 propose l'ordre suivant : Quick Check, puis conversation d'équipe, puis instrumentation. Au vu de votre demi-journée, pourquoi l'outillage arrive-t-il en dernier ?

Le support formule l'avertissement ainsi :

> DORA met en garde contre le fait de « se concentrer sur la mesure au détriment
> de l'amélioration ». Construire des intégrations vers cinq systèmes pour obtenir
> des chiffres au dixième près n'est pas un bon premier investissement.

Et le tableau 7.1 en fait un piège à part entière : *« Mesurer au lieu
d'améliorer → six mois de projet dashboard, zéro changement de pratique »*.

**Cette demi-journée en donne quatre démonstrations concrètes.**

**1. L'outillage ne répond pas aux questions qui décident des chiffres.** Le
`dora-definitions.yml` de la phase 0 contient huit décisions. Aucune n'est
technique : quel environnement compte, quel horodatage fait foi, ce qu'est un
incident, quelle fenêtre. Un outil installé avant ces décisions les prend à notre
place, silencieusement — c'est littéralement la phrase en tête du fichier :
*« Une décision non prise ici sera prise par l'outil, sans que vous le
sachiez. »* Et nous en avons mesuré l'enjeu : un facteur **133** entre un
comptage naïf et un comptage conforme au contrat (question 3).

**2. L'outillage ne peut pas créer la donnée dont il a besoin.** Trois métriques
sur cinq exigent une convention humaine appliquée dans la durée (questions 11,
12, 19). Installer DevLake sur excalidraw en janvier n'aurait rien produit de
plus que notre script de 200 lignes. La conversation d'équipe, elle, est
exactement le moment où l'on décide qui écrit `caused_by` et quand — c'est-à-dire
le moment qui rend l'outillage utile plus tard.

**3. Cette demi-journée a passé l'essentiel de son temps sur des questions non
techniques.** Le décompte est éloquent : la manipulation outillée
(lancer le collecteur, quatre fois) représente moins de dix minutes. Tout le
reste — le contrat, l'identification des environnements, la confrontation des
trois compteurs d'issues, le diagnostic du label mort — est de la lecture, de
l'interprétation et de la décision. Le chiffre est arrivé en trois secondes ; sa
signification a demandé quatre heures.

**4. Le Quick Check ne coûte rien et oriente tout.** Quelques minutes
d'auto-évaluation déclarative disent déjà où se situe l'équipe. Si le ressenti
déclaré dit « nos déploiements font peur », l'instrumentation n'est pas la
priorité : l'automatisation du déploiement l'est. Le support (6.3) enchaîne
d'ailleurs par « s'engager collectivement sur **une seule** contrainte » — et il
n'y a pas besoin de métriques instrumentées pour identifier la contrainte la plus
criante.

**Et une raison qui n'est pas dans le support**, mais que la phase 4 a rendue
évidente : l'outillage installé trop tôt mesure une chaîne que personne n'a
encore accepté de changer. Ses chiffres n'ont alors aucun destinataire. Or une
métrique sans destinataire ne survit pas : c'est exactement ce qui est arrivé au
label `bug` d'excalidraw, appliqué 765 fois puis abandonné (question 16). L'ordre
du chapitre 5.5 n'est pas une précaution pédagogique, c'est une condition de
**survie de l'instrumentation**.

**Nuance à ne pas perdre.** « L'outillage en dernier » ne veut pas dire « le plus
tard possible ». Les données de livraison ne sont pas rétroactives (question 20) :
une instrumentation posée dans six mois ne dira rien des six mois écoulés.
L'ordre est une question de **séquence**, pas de délai — Quick Check et
conversation d'équipe tiennent en une réunion, pas en un trimestre.

---

### 24. Four Keys était la référence citée dans la plupart des tutoriels jusqu'en 2024. Quelle habitude de travail cela suggère-t-il avant d'adopter un outil trouvé en ligne ?

**L'habitude : vérifier l'état vivant du dépôt avant de lire le tutoriel qui le
recommande — et le vérifier soi-même, à la date du jour.**

Concrètement, cinq contrôles qui prennent deux minutes :

| Contrôle | Où | Ce qu'il révèle |
|---|---|---|
| Bandeau d'archivage | page d'accueil du dépôt | projet en lecture seule |
| Date du dernier commit | onglet Code | activité réelle |
| Date de la **dernière version publiée** | onglet Releases | maturité de ce qui est installable |
| Écart entre les deux | — | projet qui bouge sans stabiliser |
| Issues ouvertes sans réponse | onglet Issues | maintenance effective |

Le cas de Four Keys montre pourquoi les deux premières ne suffisent pas : le
projet portait la caution de Google, était cité partout, et son dernier commit de
janvier 2024 pouvait passer pour récent en 2024. C'est le **bandeau d'archivage**
qui tranche — une information binaire, gratuite, et que le tutoriel ne contient
jamais, puisqu'il a été écrit avant.

**Le mécanisme à comprendre, et il est plus général que Four Keys.** La
documentation en ligne est datée à l'instant de sa rédaction et ne se met jamais à
jour ; les dépôts, eux, changent. L'écart se creuse mécaniquement avec le temps.
Plus un tutoriel est populaire — donc ancien, bien référencé, souvent recopié —
plus il a de chances de recommander un outil dont l'état a changé depuis. **Le
référencement favorise l'obsolescence** : les premiers résultats de recherche sont
les plus anciens.

**Ce TD en fournit quatre illustrations, ce qui est beaucoup pour une
demi-journée :**

| Écart constaté | Le document dit | La réalité du jour |
|---|---|---|
| Environnements d'excalidraw | 6 valeurs, 3 `Production` | 8 valeurs, 4 `Production` |
| Statuts d'un déploiement | `created_at` du statut différent de celui du déploiement | identiques à la seconde |
| Symptôme du tableau 7.3 | « fréquence 10× trop élevée » | ≈ 133× |
| URL de DevLake | `apache/incubator-devlake` | redirige vers `apache/devlake` |

Aucun de ces documents n'est mal écrit. Le guide du TD a été relevé le
6 septembre 2026, cinq jours avant nous. **Cinq jours ont suffi à en périmer une
partie.** C'est l'argument décisif : le problème n'est pas la négligence des
auteurs, c'est la nature même d'un document écrit sur un système vivant.

**Ce que j'en retiens pour ma pratique de lead dev**, au-delà du choix d'outil :

1. **Traiter un tutoriel comme un point de départ daté, pas comme une
   référence.** La date de publication est la première chose à chercher, avant le
   contenu.
2. **Aller à la source primaire.** Pour DORA, c'est dora.dev et le rapport
   annuel ; pour un outil, c'est son dépôt. Le tutoriel est un intermédiaire, et
   tout intermédiaire vieillit.
3. **Consigner ce qu'on a vérifié, et quand.** C'est ce que fait ce rendu en
   datant chaque relevé — et c'est ce qui permettra de dire, dans six mois, si un
   écart est nouveau ou s'il existait déjà.
4. **Se souvenir que le support de cours est soumis à la même règle.** Le
   chapitre 7.3 annonce « 10× », nous avons mesuré « 133× ». Le mécanisme décrit
   est juste, son amplitude est datée. Vérifier sur ses propres données n'est pas
   de la défiance envers l'enseignement, c'en est l'application.

Et l'ironie vaut d'être relevée : Four Keys était l'outil de référence pour
mesurer une chaîne de livraison, publié par l'organisation qui porte DORA. Rien
n'immunise contre l'obsolescence, pas même l'expertise sur le sujet.

---

## Restitution collective

> _À compléter._
