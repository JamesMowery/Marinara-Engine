# Écrire des ensembles de règles Game Mode

Un ensemble de règles explique à Game Mode le fonctionnement d'un système sur table : les dés d'un test, le contenu de la fiche, les ressources dépensées et ce qu'un repos rend. Ce guide s'adresse à ceux qui veulent en écrire un et le partager. Pour jouer avec celui d'une autre personne, commence par [Choisir les règles](../game/getting-started.md#choosing-rules).

Un ensemble est un fichier JSON. Ce sont des données, pas du code. Rien ne s'y exécute, donc l'importer ne peut rien faire à ton ordinateur. Avant d'importer un fichier d'autrui, lis attentivement le texte du Game Master : il est envoyé au modèle dans toutes les parties utilisant cet ensemble.

## À lire d'abord : ce qu'un ensemble peut et ne peut pas faire

Un ensemble ne peut que remplir les paramètres d'une mécanique déjà connue du moteur. Aujourd'hui, celui-ci connaît deux façons de résoudre un test ; ton fichier en choisit une avec `resolution.kind` :

- **`dice-sum`** : lancer des dés, ajouter des nombres de la fiche et atteindre ou dépasser une difficulté. Cela couvre les systèmes d20, ceux à 2d6 plus attribut et bien d'autres.
- **`dice-pool`** : lancer le nombre de dés du personnage et compter ceux qui atteignent une cible. Cela couvre les systèmes où une valeur représente une poignée de dés plutôt qu'un bonus.

Les deux sont décrits intégralement dans [Types de résolution](#resolution-kinds).

Une mécanique qui n'entre dans aucune de ces formes ne peut pas s'écrire dans le fichier. Par exemple : garder le plus grand dé d'une réserve, réussir sous un pourcentage, utiliser des dés à symboles ou des réserves opposées. Chacune exige un nouveau type de résolution dans le moteur : une contribution de code avec des tests, pas un JSON. Si ton système en a besoin, ouvre une demande de fonctionnalité sur le dépôt du moteur et décris la mécanique avec quelques jets détaillés. Ces exemples deviennent les tests.

Game Mode peut résoudre un combat avec les règles de Marinara ou celles de ton ensemble. Le bloc facultatif `battle` prête au combat Marinara les nombres des fiches : consulte [Batailles](#battles-lending-the-sheet-to-marinaras-combat). Le bloc facultatif `combat` définit comment ton ensemble résout le combat : consulte [Combat](#combat-a-fight-your-own-rules-resolve). Le moteur applique déjà ces règles. **Combat Preference** (Préférence de combat) choisit la présentation Classic ou, si l'ensemble définit une distance, un champ Tactical.

## Démarrage rapide

1. Copie l'exemple correspondant aux jets de ton système. [`ember-roads.json`](https://github.com/Pasta-Devs/Marinara-Engine/blob/staging/docs/examples/rulesets/ember-roads.json) est un petit système à 2d6 et trois attributs montrant que le format ne suppose ni d20 ni six caractéristiques. [`gravewatch.json`](https://github.com/Pasta-Devs/Marinara-Engine/blob/staging/docs/examples/rulesets/gravewatch.json) est une petite réserve de dés à dix faces avec trois valeurs et six métiers. Pour un exemple complet, consulte le fichier 5e (SRD 5.1), [`ruleset-5e-2014.example.json`](https://github.com/Pasta-Devs/Marinara-Engine/blob/staging/docs/development/ruleset-5e-2014.example.json).
2. Remplace `id` par le tien. Il utilise minuscules, chiffres et tirets simples, comme `ember-roads`.
3. Modifie la fiche, les repos et le texte du Game Master.
4. Importe-le ; consulte [Essayer ton ensemble](#trying-your-ruleset). L'importation vérifie tout le fichier et indique les erreurs ligne par ligne avant de sauvegarder quoi que ce soit.
5. Crée une partie, choisis ton ensemble sous **Rules** (Règles) et joue quelques tests.

Pour une aide pendant la saisie, indique le JSON Schema à ton éditeur en ajoutant cette première ligne à l'intérieur des accolades extérieures du fichier :

```json
"$schema": "https://raw.githubusercontent.com/Pasta-Devs/Marinara-Engine/staging/docs/extending/ruleset.schema.json",
```

Le schéma repère les clés mal écrites et les mauvais types pendant la saisie. Il ne vérifie pas que les noms désignent des éléments existants, par exemple une compétence nommant un attribut. L'importation s'en charge.

Tu peux ajouter une ligne `"$comment": "..."` à n'importe quel objet pour te laisser une note. Le moteur l'ignore.

## Les parties du fichier

| Clé | Contenu |
| --- | --- |
| `schemaVersion` | Toujours `1`. |
| `id`, `version` | Le nom de l'ensemble pour le moteur et un entier que tu augmentes à chaque changement publié. |
| `name` | Ce que voient les joueurs dans l'assistant. |
| `edition` | Facultatif. Une ligne sur l'édition ou le brouillon. |
| `license` | Facultatif. Un identifiant SPDX et l'attribution exigée par ta source. |
| `coverage` | Ce que couvre l'ensemble et le résumé d'une ligne affiché dans l'assistant. |
| `resolution` | Comment lancer un test ou une sauvegarde. |
| `sheet` | Tout le contenu de la fiche. |
| `rests` | Ce que chaque repos restaure et efface. |
| `gm` | Le texte donné au modèle du Game Master et les valeurs de fiche qu'il voit par personnage. |
| `catalogs` | Facultatif. Les entrées préparées que propose l'éditeur pour éviter de saisir de longues listes. |
| `items` | Facultatif. Les mots dans lesquels tes objets sont écrits : catégories, raretés, statistiques, emplacements, monnaie. |
| `battle` | Facultatif. Ce qu'une bataille lit dans la fiche et y réécrit ensuite. |
| `combat` | Facultatif. Comment tes règles résolvent un combat et ce que joue l'écran de bataille. |
| `layers` | Facultatif. Les variantes activées par le joueur à la création d'une partie. |

Le fichier peut atteindre 256 KB. Le texte destiné à un prompt (noms, libellés, texte du Game Master) ne peut pas contenir de sauts de ligne, de crochets ni de doubles accolades.

Les identifiants de la fiche (attributs, compétences, champs, réserves, etc.) utilisent minuscules, chiffres et traits de soulignement, en commençant par une lettre, comme `grit_max`.

<a id="resolution-kinds"></a>

### Types de résolution

`resolution.kind` choisit le jet. Les deux types lisent la même fiche et partagent trois clés ; les parties du fichier après `resolution` ne changent donc pas quand tu passes de l'un à l'autre :

- `abilityModifier` : comment une valeur de fiche devient un nombre. `identity` utilise la valeur elle-même ; `floorHalfMinusTen` est la règle 5e ; `stepTable` permet de définir tes seuils sous la forme `[[score, number], ...]`.
- `proficiencyTiers` : les niveaux d'entraînement d'une compétence ou sauvegarde. Le premier s'applique à une compétence non renseignée. Un niveau ajoute `flat`, ou `multiplier` fois un bonus de maîtrise, ou les deux. Si ton système possède ce bonus, indique sa source avec `"proficiency": { "bonus": { "derived": "proficiency_bonus" } }`.
- `proficiency` : facultatif, nécessaire seulement pour un niveau qui multiplie.

Le sens du nombre dépend du type : `dice-sum` l'ajoute au jet ; `dice-pool` lance ce nombre de dés.

#### `dice-sum` : additionner les dés

```json
"resolution": {
  "kind": "dice-sum",
  "dice": { "count": 2, "sides": 6 },
  "abilityModifier": { "op": "identity" },
  "proficiencyTiers": [
    { "id": "untrained", "label": "Untrained" },
    { "id": "trained", "label": "Trained", "flat": 1 }
  ],
  "advantage": false,
  "difficultyLadder": [
    { "label": "Easy", "dc": 6 },
    { "label": "Hard", "dc": 10 }
  ]
}
```

- `dice` : nombre de dés et de faces. Le total est comparé à la difficulté.
- `advantage` : si le Game Master peut demander deux jets et en garder un.
- `naturals` : effet des faces minimale et maximale d'un seul dé sur les tests et sauvegardes : `none`, `both`, `max-only` ou `min-only`. Omets-le pour une simple addition. Il exige un seul dé : un système 2d6 doit utiliser `none`.
- `difficultyLadder` : difficultés parmi lesquelles le GM doit choisir. `dc` est le nombre que le total doit atteindre. Le Game Master peut nommer un palier au lieu d'écrire son nombre, comme `difficulty="Hard"`, et le moteur lit le nombre sur ce palier.

#### `dice-pool` : lancer les dés et les compter

Le nombre de la fiche est la **taille de la réserve**, pas un bonus ajouté. Une valeur de 3 et un métier valant 2 lancent cinq dés. C'est tout le principe : aucun nouveau vocabulaire de fiche ni nouvel éditeur ; un système aux valeurs exprimées en poignées de dés utilise les mêmes `abilities`, `skills` et `proficiencyTiers` que les autres.

```json
"resolution": {
  "kind": "dice-pool",
  "die": { "sides": 10 },
  "abilityModifier": { "op": "identity" },
  "proficiencyTiers": [
    { "id": "rating_0", "label": "Untried" },
    { "id": "rating_1", "label": "Shown once", "flat": 1 }
  ],
  "pool": { "min": 1, "max": 15, "abilityPlusAbility": true },
  "target": { "default": 7, "min": 5, "max": 9 },
  "explode": { "from": 10, "min": 8 },
  "cancel": { "upTo": 1 },
  "botch": { "upTo": 1 },
  "exceptional": { "successes": 5 },
  "situationalDice": { "min": -3, "max": 3 },
  "reroll": [{ "id": "careful", "upTo": 6, "mode": "once" }],
  "difficultyLadder": [
    { "label": "Plain work", "successes": 1, "target": 6 },
    { "label": "Grim", "successes": 3, "target": 8 }
  ]
}
```

- `die` : nombre de faces d'un dé de la réserve, de 2 à 100.
- `pool` : limites appliquées au nombre de la fiche avant toute explosion. Un `min` de 0 laisse une réserve vide échouer sans jet ; `max` ne dépasse pas 100. `abilityPlusAbility: true` permet à un test d'attribut de nommer un second attribut avec `with=` et de lancer les deux, pour les systèmes qui lancent deux attributs ensemble ; sans lui, `with=` ne signifie rien sur un test d'attribut.
- `target` : face à atteindre pour compter. Mets `min` sous `max` pour permettre au GM de la déplacer par test avec `threshold=` ; rends les trois valeurs identiques pour la fixer.
- `double` : facultatif. Une face égale ou supérieure à `from` compte deux fois.
- `explode` : facultatif. Une face égale ou supérieure à `from` lance un dé supplémentaire, qui peut lui-même exploser. Les dés supplémentaires sont plafonnés à `pool.max` en plus de la réserve initiale : un test lance au maximum deux fois `pool.max` dés et un `from` bas ne peut pas provoquer des jets infinis.
- Les deux peuvent porter un `min` : la face la plus basse jusqu'à laquelle un test peut abaisser la règle, avec `explode="N"` ou `double="N"` sur la balise ou une entrée utilisée par le personnage. Avec un `min`, tu peux omettre `from` : la règle ne se déclenche alors que sur un test qui la demande. Une spécialité qui fait relancer les dix sur une seule compétence s'écrit `"explode": { "min": 10 }`. `from` ne peut pas être inférieur à `min`.
- `cancel` : facultatif. Une face égale ou inférieure à `upTo` retire une réussite. Le compte ne descend jamais sous zéro.
- `botch` : facultatif. Si **aucun** dé n'a réussi et qu'une face égale ou inférieure à `upTo` est sortie, le test est un échec critique. Une réserve dont l'unique réussite a été annulée a échoué, mais n'a pas fait de maladresse critique. `"rule": "halfOrMore"` le lit autrement : des faces égales ou inférieures à `upTo` sur au moins la moitié des dés lancés au départ (explosions non comptées) donnent un échec critique si aucun dé n'a réussi ; sinon, le résultat est maintenu et marqué `complication="true"`, quelque chose tourne mal en même temps. `"noSuccesses"` est la valeur par défaut.
- `exceptional` : facultatif. Ce nombre de réussites nettes ou plus, sur un test réussi, donne une réussite critique.
- `situationalDice` : facultatif. Limites des dés ajoutés ou retirés par le GM avec `bonus=` pour une prouesse, des blessures ou une mauvaise lumière.
- `reroll` : facultatif. Des relances que le système accorde sans que personne ne les paie, jusqu'à six, chacune avec un `id` que le Game Master nomme sur un test avec `reroll=`. Les faces égales ou inférieures à `upTo` sont relancées : `"once"`, et la nouvelle face reste quelle qu'elle soit, ou `"until"`, jusqu'à ce qu'elles montrent plus. `upTo` va de 1 au nombre de faces du dé moins un, puisqu'une relance de toutes les faces ne s'arrêterait jamais, et un `until` s'arrête de toute façon après cent relances sur un même test. Aucune n'est lancée si le test ne la nomme pas. Un test lance une seule relance : si une dépense ou une entrée utilisée par le personnage en a aussi acheté une, celle qui atteint le plus de faces est lancée, et `until` l'emporte sur `once` à faces égales.
- `difficultyLadder` : `successes` indique le nombre de réussites nécessaire. Un palier peut nommer un `target`, seulement si la cible est réglable et dans ses limites. C'est avec cette cible que compte un test à ce palier : quand le Game Master nomme le palier avec `difficulty=`, ou écrit un `dc=` qu'exige un seul palier. Si plusieurs paliers exigent le même nombre de réussites, `dc=` seul ne peut pas dire lequel était visé : la cible reste celle par défaut ; nomme le palier.

Les faces de `cancel` et `botch` doivent être inférieures à la cible minimale, et toutes les faces nommées par ces règles doivent exister sur le dé. Une règle impossible à déclencher est refusée à l'importation plutôt que découverte en jeu.

Un ensemble en package utilisant des réserves relève de Capability API 1.24 ; un `min` sur `explode` ou `double`, `pool.abilityPlusAbility` et `botch.rule` relèvent de 1.37 ; `reroll`, de 1.38. Un ensemble communautaire importé est validé par le moteur qui le lit et n'a donc besoin de rien d'autre.

#### Ce que le Game Master peut écrire sur un test de réserve

```
[skill_check: skill="Ward" dc="2" who="Bram the Quiet" threshold="8" bonus="-2" with="Sinew"]
[skill_check: skill="Ward" difficulty="Grim" explode="9"]
[skill_check: skill="Nerve" dc="1" with="Warmth"]
[skill_check: skill="Ward" dc="2" reroll="careful"]
```

- `dc` est le nombre de **réussites** demandé, pas une cible par dé. Il va de 1 au maximum que pourrait compter un jet : maximum de la réserve, doublé si les dés explosent, puis doublé encore si les faces comptent deux fois.
- `threshold=` déplace la cible par dé et n'est proposé que si `target.min` est inférieur à `target.max`.
- `bonus=` ajoute ou retire des dés et n'est proposé que si `situationalDice` est déclaré.
- `with=` lance une compétence ou sauvegarde avec un autre attribut. Il fonctionne pour les deux types ; 5e obtient ainsi "Strength (Intimidation)" par le même attribut. Sur un test d'attribut, il ajoute plutôt un second attribut, si `pool.abilityPlusAbility` est activé.
- `difficulty=` nomme un palier de l'échelle à la place de `dc=`, pour les deux types : ses réussites (ou sa difficulté pour une somme) et, pour une réserve, sa cible. Un `dc=` écrit à côté l'emporte toujours pour le nombre, et `threshold=` pour la cible. Un nom auquel aucun palier ne répond, ou que deux paliers partagent, est ignoré.
- `explode=` et `double=` déplacent les faces de ces règles pour ce test et ne sont proposés que si la règle a un `min`.
- `reroll=` nomme l'une de tes entrées `reroll` pour ce test et n'est proposé que si tu en as déclaré. Un nom auquel aucune ne répond est ignoré.

Tout respecte ton fichier : une valeur hors limites est ramenée à l'extrémité la plus proche et un attribut absent est ignoré sans refuser le test. Le registre montre ce qui a réellement servi : la difficulté et la cible sous forme de nombres, les dés supplémentaires après limitation, `with=` seulement si l'attribut a été remplacé ou ajouté, une face seulement si le test l'a déplacée, `reroll=` seulement si la relance nommée est celle qui a été lancée et a relancé quelque chose, et `complication="true"` quand une maladresse `halfOrMore` a mal tourné à côté d'un résultat. Le moteur lance toujours les dés. Il remplace tout résultat de réserve écrit par le modèle, ignore `mode="advantage"` puisque le type n'a pas d'avantage et n'utilise pas un dé lancé par le joueur avant le tour.

#### Ce qui reste hors périmètre, et pourquoi

Chacun de ces cas nécessite son propre type de résolution : aucun ne s'exprime en comptant des dés qui atteignent une cible.

- **Garder le plus grand dé** (comme dans Blades in the Dark) nécessite un niveau de réussite partielle absent du résultat d'un test.
- **Réserves de posture comparées à un attribut** (comme dans Lasers and Feelings) choisissent le sens de comparaison à chaque test.
- **Dés à symboles** (comme dans Genesys) ne produisent aucun nombre.
- **Réserves opposées** résolvent deux personnages à la fois ; un test n'a qu'un lanceur.
- **Jets sous un pourcentage et pourcentages ouverts** comparent dans l'autre direction.
- **Réserves additionnées avec un dé spécial** (comme dans OpenD6) additionnent les dés en traitant l'un d'eux à part.

Les deux éléments autrefois exclus sont maintenant modélisés ; la section sur les dépenses pour modifier un jet, ci-dessous, explique comment. Une règle du système lui-même, "spend a point for a success" ou "spend a point to throw the failures again", est `resolution.spend`. Une relance que le système accorde gratuitement quand la situation l'exige est `resolution.reroll`, ci-dessus. Celle qui appartient à un élément choisi par le personnage est `mechanics.check` sur une entrée de catalogue, payée avec le coût de cette entrée.

### Dépenser pour modifier un jet

Certains systèmes permettent de payer un jet à venir : un point de volonté pour une réussite automatique. `resolution.spend` l'exprime comme règle permanente du système, plutôt que comme achat propre à un personnage :

```json
"spend": [{ "pool": "resolve", "amount": 1, "successes": 1, "perCheck": 2 }]
```

- `pool` désigne l'une de tes `live.pools`. Elle ne peut pas commencer vide : il n'y aurait rien à dépenser au début de la partie.
- `amount` est le coût d'UN achat. `successes`, `dice` et `reroll` indiquent ce qu'il achète ; il faut au moins l'un d'eux. Les réussites sont ajoutées après le comptage et les annulations, puisque personne ne les a lancées. Les dés sont lancés avec la réserve, dans ses limites. Un `reroll` s'écrit `{ "upTo": 6, "mode": "once" }`, se lit comme `resolution.reroll` et n'est acheté qu'une fois, quel que soit le nombre d'achats du test : un dé relancé deux fois reste une seule relance.
- `perCheck` limite le nombre d'achats d'un test : il peut acheter au maximum pour `amount * perCheck` points. Cela empêche une réserve pleine d'acheter un jet impossible à perdre. C'est un nombre de 1 à 10, une valeur de la fiche sous la même forme que celle lue par une valeur dérivée (`{ "abilityScore": "nerve" }`, `{ "derived": "focus" }`), ou `"pool"` : autant que le test a de dés, c'est-à-dire le nombre de la fiche pour ce test avant blessures, dés bonus ou modificateurs. Une valeur de la fiche est lue pour la personne qui lance, arrondie vers le bas et maintenue entre 0 et 100 ; un personnage dont la limite vaut 0 n'achète rien et ne paie rien.
- Seul `dice-pool` le permet : un jet additionné n'a ni réussites ni réserve auxquelles ajouter quelque chose. Un ensemble `dice-sum` déclarant `spend` est refusé à l'importation.
- Jusqu'à quatre entrées, et deux ne peuvent pas nommer la même réserve, sinon le test ne pourrait pas les distinguer.
- Une dépense qui achète un `reroll`, un `perCheck` qui n'est pas un nombre et plus de deux entrées relèvent de Capability API 1.38 pour un ensemble en package.

**Cela se place sur le test lui-même.** Le GM écrit `[skill_check: skill="Nerve" dc="2" spend="resolve:1"]`, pas une commande `[sheet:]` séparée : les dés sont lancés avant d'appliquer les commandes de fiche et il ne resterait rien à modifier. Une seule résolution lance les dés et paie ce qui les a modifiés.

### Un charme qui modifie un jet

`resolution.spend` est une règle du système. Une entrée réellement CHOISIE par le personnage peut aussi modifier un test, avec `mechanics.check` sur son entrée de catalogue :

```json
"mechanics": {
  "kind": "utility",
  "cost": [{ "pool": "blood", "amount": 1 }],
  "perCostStep": { "flat": 1 },
  "check": { "reroll": { "upTo": 1, "mode": "once" }, "successes": 1 }
}
```

- `reroll` relance les dés égaux ou inférieurs à `upTo`. `once` remplace chacun une fois et garde la nouvelle face ; `until` continue. `upTo` doit être inférieur à la face maximale, sinon toute la réserve serait relancée indéfiniment. Le moteur plafonne le nombre de dés relançables par test, quoi que dise le fichier.
- `dice` ajoute des dés avant le jet ; `successes` ajoute des réussites après comptage ; `threshold` fixe la cible par dé de ce jet dans la plage autorisée par `target`.
- `explode` et `double` déplacent les faces de ces règles pour ce seul jet, uniquement si ta règle a un `min` et sans descendre en dessous : `"check": { "explode": 8 }` est une entrée qui fait relancer les huit. Ils l'emportent sur ce que le Game Master a écrit sur la balise, comme `threshold`. Capability API 1.37.
- Il faut au moins l'un d'eux, sinon l'entrée ne dit rien et est refusée.
- Seul `dice-pool` peut les appliquer ; un ensemble `dice-sum` avec `mechanics.check` est refusé à l'importation.

**Le coût est le `cost` de l'entrée**, payé par le même mécanisme que tout autre usage : la réserve et une utilisation de chaque compteur écrit par cette entrée. `perCostStep` indique qu'elle CHANGE D'ÉCHELLE : si elle le déclare, elle est achetée autant de fois que son prix a été payé ; sinon, elle est achetée une seule fois, quelle que soit la somme offerte.

**Le GM la nomme sur le test :** `[skill_check: skill="Brawl" dc="3" use="Potence" spend="blood:3"]`. Pas dans une commande de fiche séparée, pour la même raison : les dés sont lancés avant la mise à jour des comptes.

**Tout ou rien.** Si la réserve ne suffit pas, aucun achat ni prélèvement n'a lieu ; le jet reste celui qu'il aurait été. Des points ne correspondant pas à un nombre entier d'achats n'achètent rien non plus. Une demande au-delà de `perCheck` est plafonnée plutôt que refusée, et seul ce plafond est payé. Le moteur calcule tout cela ; le GM nomme la dépense annoncée par le joueur sans toucher aux dés. Le registre indique ce qui a vraiment été payé, l'entrée appliquée, les réussites non lancées et le nombre de dés relancés. Un charme non choisi, ou dont le moteur ne peut pas lire le catalogue, ne fait rien au lieu d'être appliqué sur confiance.

### La fiche

- `sections` regroupe les éléments dans l'éditeur. Les champs, valeurs dérivées et listes nomment avec `section` celle où ils se trouvent, et les attributs, compétences et sauvegardes le peuvent aussi : l'éditeur, la fiche de la partie et le bloc de fiche du Game Master les montrent alors sous ce titre (une fiche qui ne leur nomme aucune section les montre exactement comme avant). Une section peut aussi porter une règle `untrained` (ci-dessous).
- `abilities` contient les valeurs principales. `skills` et `saves` peuvent nommer l'attribut de leur jet, un `cap` et une règle `untrained` (ci-dessous).
- `fields` contient des valeurs simples. Types : `number`, `text`, `longtext`, `boolean`, `enum` (liste fixe de choix) et `dice` (texte comme `1d8`).
- Les valeurs `derived` sont calculées depuis d'autres valeurs et ne peuvent pas être écrasées à la saisie. Opérations : `sum`, `min`, `max`, `scale` (multiplier et arrondir), `stepTable` (chercher dans des seuils, comme un niveau donnant un bonus de maîtrise) et `enumTable` (un nombre pour chaque valeur d'un champ enum ou d'un état vivant ; voir États vivants ci-dessous).
- `lists` contient des tableaux à colonnes personnalisées, comme équipement, sorts ou capacités. Une liste avec `pools` transforme chaque ligne en ressource ayant son propre maximum, pour les capacités de classe à usages limités.
- `live` contient ce qui change en jeu : `pools` (points de vie, emplacements de sorts, Grit), `tracks` (nombre sur une échelle, comme l'épuisement, ou piste de blessures à cases), `text` (notes courtes, comme la concentration), `conditions` et `states` (une valeur parmi un ensemble fermé, comme une forme ou une posture).

Tout ce qui lit un nombre le nomme par une référence de valeur : un objet avec exactement une clé parmi `const`, `field`, `derived`, `abilityScore`, `abilityMod`, `abilityModFromField`, `skillMod`, `saveMod`, `listSum`, `livePool`, `liveTrack` et `itemStat`. Par exemple, une réserve au maximum dérivé : `"max": { "derived": "grit_max" }`.

- `listSum` additionne une colonne numérique d'une liste : `{ "listSum": { "list": "gear", "column": "bulk", "onlyWhen": "packed" } }`. `onlyWhen` est facultatif et nomme une colonne booléenne ; seules les lignes où elle est activée comptent. Une cellule vide compte comme la valeur par défaut de sa colonne, et une liste cachée par `hideWhen` n'ajoute rien. Ember Roads calcule ainsi Burden à partir de l'équipement emporté par un personnage : il ne peut jamais s'écarter de la liste comme le ferait un nombre saisi. La colonne à échelle d'un catalogue ne peut pas lire une somme de liste, même par une valeur dérivée : une liste peut contenir des cellules à échelle, les siennes ou d'autres qui la relisent, et le recalcul ne se stabiliserait jamais.
- `livePool` lit ce qui reste dans l'une de tes `live.pools` (la réserve d'une ligne de liste n'en fait pas partie). `liveTrack` lit l'une de tes `live.tracks`, avec un `read` facultatif : `"value"`, sa position (par défaut) ; `"filled"`, l'écart au-dessus de son `min` ; `"remaining"`, l'écart sous son `max` ; ou, seulement sur une piste de blessures, `"penalty"`, la pénalité active. Une réserve ou une piste cachée vaut 0. Le "Harm still to take" de Gravewatch s'écrit `{ "liveTrack": "harm", "read": "remaining" }`.
- Une lecture de l'état courant prend celui-ci tel qu'il est quand un test est lancé, qu'un combat commence ou que le bloc de fiche du Game Master est écrit. S'il n'existe pas encore (l'éditeur de fiche, l'examen d'une importation), elle lit l'état de départ du jeu : une réserve pleine ou vide selon son `start`, une piste à son `default`, une piste de blessures sans marque.
- Rien de ce qui est calculé avant qu'un état courant existe ne peut le lire : le `max` d'une réserve ou d'une piste, le bonus de maîtrise, ou la colonne à échelle ou la mise à l'échelle d'un catalogue. Cela vaut aussi à travers une valeur dérivée qui le lit et à travers une compétence plafonnée par une valeur vivante, et l'importation nomme la valeur fautive.
- Ces trois références relèvent de Capability API 1.39 pour un ensemble en package.
- `itemStat` lit les objets que tient le personnage, comme la protection de l'armure qu'il porte, et suit la même règle qu'une lecture de l'état courant. Consulte [Les objets sur la fiche](#items-on-the-sheet). Capability API 1.52.

**Ce que fait un test sans entraînement.** Une compétence ou sauvegarde dans laquelle le personnage n'a aucun entraînement (son niveau est le premier) se lance normalement, sauf si l'ensemble en décide autrement avec `untrained` sur la compétence, sur la sauvegarde ou sur la section où elle se trouve (la règle propre à la compétence ou à la sauvegarde l'emporte) :

```json
"sections": [{ "id": "labour", "label": "Labour", "untrained": { "by": -1 } }],
"skills": [
  { "id": "dig", "label": "Dig", "ability": "sinew", "section": "labour", "untrained": "refuse" },
  { "id": "listen", "label": "Listen", "ability": "nerve", "untrained": "harder" }
]
```

- `"normal"` : comme d'habitude (par défaut).
- `{ "by": -3 }` : ajouté au nombre du test, de -20 à 20 : des dés pour une réserve, un montant fixe pour une somme. Il fait partie du nombre que montre la fiche et que lance chaque test, avant tout `cap`.
- `"harder"` : un degré plus difficile. Pour une réserve, la cible par dé monte d'un cran (en restant dans la plage de la cible) ; il faut donc un ensemble à réserve dont la cible peut bouger. Partout ailleurs, il est refusé à l'importation, car il ne pourrait jamais changer un jet.
- `"refuse"` : le test n'est pas lancé du tout. Le moteur réécrit la demande du Game Master avec `reason="untrained"`, ce qui la règle : rien ne la lance plus tard, le Game Master apprend que le personnage ne pouvait pas tenter l'action et la narration le dit. Le point de terminaison des tests de compétence répond à un tel test par une erreur 400 (`skill_check_untrained`), et un embranchement qui en dépend ne garde aucune de ses deux issues. L'éditeur de fiche n'affiche aucun nombre pour lui.
- Un personnage sans fiche n'a aucun entraînement à lire ; aucune règle sans entraînement ne s'applique donc à lui.
- Le rappel du Game Master liste les règles sans entraînement, par section ou par compétence, pour qu'il demande des tests que le personnage peut faire.

Les sections sur les attributs, compétences et sauvegardes, et `untrained` où qu'il soit, relèvent de Capability API 1.41 pour un ensemble en package.

Une compétence ou sauvegarde peut porter un `cap`, une référence de valeur qui fixe le maximum que son test puisse jamais atteindre : une valeur qu'elle ne peut dépasser, ou une piste qui la retient. Gravewatch plafonne Soothe à la Resolve qui reste à un gardien, `"cap": { "livePool": "resolve" }`. Le nombre plafonné est celui que montre la fiche et que lance chaque test, et un échange `with=` est calculé sur le nombre avant le plafond, puis plafonné à nouveau. Un plafond ne peut pas lire un modificateur de compétence ou de sauvegarde, pas plus que la valeur dérivée qu'il lit ni aucune valeur dérivée déclarée au-dessus de celle-ci : la fiche est toujours calculée une seule fois, de haut en bas. Capability API 1.39.

`hideWhen` cache un champ, une valeur dérivée, une liste, une réserve ou une piste ordinaire selon la valeur d'un autre champ, d'exactement une façon parmi trois : `equals` une valeur, `notEquals` une valeur (caché dès que le champ contient autre chose : c'est ainsi qu'on dit "seulement pour ce type de personnage"), ou `in` une liste de valeurs. Chaque valeur nommée doit pouvoir figurer dans le champ, sinon la règle ne correspondrait jamais, et elle est vérifiée valeur par valeur ; pour `notEquals` et `in`, cela inclut la plage d'un champ numérique et la longueur d'un champ texte. La règle lit la valeur que montre l'éditeur de fiche : un champ non défini a la valeur de départ d'une fiche vierge, et une valeur enum que l'ensemble ne propose plus vaut la valeur par défaut du champ. Le fichier 5e utilise `equals` pour cacher les emplacements de sorts aux personnages qui n'en lancent pas ; Gravewatch retire l'huile de lanterne de la fiche de chaque gardien qui n'est pas de la garde de nuit avec `"hideWhen": { "field": "watch", "notEquals": "night" }`. Une couche ne peut pas retirer une valeur avec laquelle l'une de ces règles compare. Une piste de blessures ne peut pas être cachée, car les jets et les combats la lisent quoi que montre la fiche. `notEquals` et `in` relèvent de Capability API 1.39.

Le `max` d'une piste ordinaire peut être une référence de valeur plutôt qu'un nombre, pour qu'une valeur du personnage fixe sa longueur : `"max": { "field": "willpower_rating" }`. Son `min` et son `default` restent des nombres. Une piste de blessures à niveaux nommés a un `max` égal à son nombre de niveaux ; une piste à cases numérotées est aussi longue que son propre `max` (voir Cases numérotées ci-dessous). `alwaysShow: true` affiche une piste dans le bloc de fiche du Game Master même à sa valeur par défaut, pour une valeur qui compte à chaque tour ; sans lui, une piste à sa valeur par défaut est omise, car trois sauvegardes contre la mort à zéro ne disent rien. Ces trois éléments relèvent de Capability API 1.37 pour un ensemble en package.

### États vivants : une forme, une posture, une lumière

Un champ enum est choisi à la création du personnage et le reste, et un état (condition) est seulement actif ou inactif. Un **état vivant** (live state) est une valeur parmi un ensemble fermé qui change en jeu : la forme d'un métamorphe, une posture de combat, la clarté d'une lanterne.

```json
"live": {
  "states": [
    {
      "id": "stance",
      "label": "Stance",
      "values": ["guarded", "steady", "reckless"],
      "valueLabels": { "guarded": "Guarded", "steady": "Steady", "reckless": "Reckless" },
      "default": "steady"
    }
  ]
}
```

- `values` en contient de deux à quarante, chacune affichée par son entrée `valueLabels` ou telle quelle. Elles suivent la règle de tout libellé (une ligne, sans crochets ni accolades de macro) et ne contiennent jamais de guillemet double, car la commande de fiche les met entre guillemets. Jusqu'à douze états.
- `default` est le point de départ de chaque fiche, et la première valeur si tu l'omets. Seul un état éloigné de sa valeur par défaut est stocké, et une valeur stockée que l'état ne propose plus se lit comme la valeur par défaut.
- `hideWhen` retire un état d'une fiche selon la valeur d'un champ, comme pour une réserve : il n'est pas affiché, aucune commande ne le fixe, et une table (ci-dessous) le lit comme une absence de valeur.
- Le Game Master en fixe un avec `[sheet: who="Name" op="state" state="Stance" value="Reckless"]`, en nommant l'état par son identifiant ou son libellé, et la valeur telle quelle ou par son libellé. Un état ou une valeur que la fiche n'a pas est refusé (`unknown-state`, `unknown-value`). Le moteur n'enseigne la commande et ne liste toutes les valeurs que si ton ensemble a un état, et chaque bloc de fiche indique la valeur de chaque état, valeur par défaut comprise, car "sous forme humaine" est un fait dont la narration a besoin à chaque tour.
- L'écran de fiche de la partie affiche un sélecteur pour chaque état.
- Un repos en rétablit un avec une étape de restauration qui le nomme : `{ "state": "stance", "to": "default" }`, ou `"to"` l'une de ses valeurs.

**Des nombres qui suivent.** Une valeur dérivée avec `"op": "enumTable"` donne un nombre pour chaque valeur d'un champ enum ou d'un état vivant :

```json
{
  "id": "stance_brawn",
  "label": "Stance on Brawn",
  "op": "enumTable",
  "from": { "liveState": "stance" },
  "table": { "guarded": -1, "reckless": 2 },
  "default": 0
}
```

- `from` nomme exactement l'un de `field` (un champ enum) ou `liveState`. Chaque ligne de `table` est une valeur que celui-ci peut prendre, de une à quarante. Une valeur absente de la table, et un état que la fiche cache, lisent `default` (0 si tu l'omets).
- Un champ est fixé à la création du personnage : une table indexée sur un champ peut donc alimenter un maximum. Gravewatch donne une Resolve de plus à un gardien de la garde de l'aube. Un état vivant change en jeu : une table indexée sur lui est donc une lecture de l'état courant, comme `liveTrack`, et rien de ce qui est calculé avant qu'un état courant existe ne peut la lire, directement ou par une autre valeur dérivée. C'est pourquoi la clé est `liveState` plutôt que `state` : toute lecture de l'état courant est nommée `live`, ce qui permet à toi comme à l'importation de savoir ce qu'un maximum ne peut pas utiliser.
- Pour appliquer le nombre aux dés, fais pointer `resolution.adjust` dessus (voir Modificateurs de la fiche). Ember Roads ajoute sa Stance à chaque jet de Brawn ; Gravewatch retire des dés à chaque jet de Nerve quand la lanterne est voilée ou éteinte.
- Une couche peut retirer d'un champ une valeur pour laquelle une table a une ligne. Cette ligne n'est alors simplement jamais lue tant que la couche est active.

Les états vivants, `enumTable` et une étape de repos qui nomme un `state` relèvent de Capability API 1.42 pour un ensemble en package.

### Pistes de blessures : une santé marquée sur une piste

Beaucoup de systèmes ne comptent pas de points de vie. Ils utilisent une colonne de cases, chacune pire que la précédente, dont on coche une lorsqu'on est blessé. Donne à une entrée `live.tracks` des `kinds` et soit des `levels` nommés, soit des `boxes` numérotées (ci-dessous), pour remplacer un nombre sur une échelle par une telle piste :

**Quelle forme convient à ton système ?** Une réserve enregistre COMBIEN de dégâts sont passés ; une piste enregistre combien ET le type de chaque part. Si une blessure est contondante, létale ou aggravée et que ce type importe après le coup (parce que l'aggravé guérit moins vite, ne peut pas être absorbé ou finit par tuer), il doit être conservé après le jet. Seule une marque le porte. Une réserve ne le peut pas : après soustraction des dégâts, il reste un plus petit nombre, sans souvenir de la nature des points. Voilà pourquoi `combat.damageKinds` est refusé lorsque la santé est une réserve, plutôt qu'ignoré. Une réserve peut toujours avoir `damageTypes` et un adversaire peut y résister ou en être immunisé : cela concerne la quantité qui passe, pas la nature de la blessure après le coup.

```json
{
  "id": "harm",
  "label": "Harm",
  "min": 0,
  "max": 4,
  "levels": [
    { "label": "Scuffed", "penalty": 0 },
    { "label": "Winded", "penalty": -1 },
    { "label": "Bleeding", "penalty": -3 },
    { "label": "Down", "penalty": -99 }
  ],
  "kinds": [
    { "id": "knock", "label": "K", "severity": 0 },
    { "id": "tear", "label": "T", "severity": 1 }
  ]
}
```

- `levels` compte 1 à 16 degrés, du meilleur au pire. Chacun a `label` et un entier `penalty` inférieur ou égal à 0. Un grand nombre négatif exprime la mise hors d'état, donc `-99` convient.
- `kinds` compte 1 à 6 types de dégâts acceptés, chacun avec `id`, un court `label` pour la case et `severity`. Les gravités doivent être distinctes ; seul leur ordre compte, pas les nombres eux-mêmes, donc espace-les comme tu veux.
- Les deux vont ensemble. `kinds` sans `levels` ni `boxes` est refusé faute de cases à marquer, et `levels` ou `boxes` sans `kinds` est refusé car une marque doit avoir un type. Une piste a `levels` ou `boxes`, jamais les deux.
- **Distingue les deux mots.** `kinds` définit ce qu'une marque PEUT ÊTRE. Une MARQUE est l'un de ces types placé sur la piste pendant le jeu. La définition contient des types ; la fiche contient des marques.
- Une piste de blessures commence sans marque ; son `min` vaut donc 0. Une piste à niveaux nommés est aussi longue que ses niveaux : son `max` vaut `levels.length`, et un fichier qui indique autre chose est refusé plutôt que corrigé, pour qu'il n'exprime jamais deux longueurs contradictoires. Une piste à cases est aussi longue que son propre `max`, qui peut être une valeur calculée par la fiche (voir Cases numérotées ci-dessous).

**Les règles exactes**, car une lecture vague produit la mauvaise piste :

- Les marques sont triées, **les plus graves d'abord**. Sept niveaux contiennent au maximum sept marques.
- Une marque est **insérée dans l'ordre de gravité**, jamais ajoutée simplement au bout. Elle prend le plus haut niveau qui lui revient et pousse les plus légères vers le bas.
- La pénalité active est celle du **niveau marqué le plus bas**, jamais la somme. Trois marques sur l'exemple donnent `-3`, pas `0 + -1 + -3`.
- `amount` est un nombre de marques d'un type, **appliquées une par une**. Une piste qui se remplit en cours de route suit donc la même règle qu'une piste déjà pleine.
- Marquer une piste **pleine** **augmente d'un degré sa marque la moins grave**, au lieu d'en ajouter une. Un degré sur ton échelle de types, quel que soit le type de la nouvelle marque.
- Une marque qui dépasserait la gravité maximale y reste ; celle qui n'a pas pu entrer compte comme **débordement**. Celui-ci est sauvegardé, afin qu'un rechargement n'oublie pas les dégâts déjà subis.
- **Guérir utilise la même commande avec une quantité négative.** Elle efface le débordement avant toute marque. Un soin qui nomme l'un des types de la piste n'efface que les marques de ce type : "le contondant guérit, le létal reste" s'écrit donc `kind="bashing"` avec une quantité négative ; un soin qui ne nomme aucun type de la piste efface d'abord les marques les plus légères.

**Marquer en jeu.** Le GM écrit `[sheet: op="damage" track="harm" kind="knock" amount="1"]` et soigne avec un `amount` négatif. Sur une piste qui se remplit par case, il ajoute `box="3"` pour la case que le coup atteint. La forme de `damage` pour réserves, qui nomme `pool=`, ne change pas. La commande ordinaire `track` est refusée sur une piste de blessures : un nombre ne précise pas le type des nouvelles marques. Le joueur peut aussi marquer et effacer les cases à la main, comme ces systèmes le prévoient.

**Un combat peut aussi la marquer.** Fais pointer `combat.health` vers la piste plutôt qu'une réserve : un coup réussi marque les cases indiquées par `combat.damageKinds.marks`, du type auquel ce bloc associe ses dégâts. Un personnage à la piste pleine est à terre, ce que lit la règle de mort. Sur une piste qui se remplit par case, des dégâts `per-point` nomment la case qu'ils atteignent et `per-blow` vise la première ; un coup qu'aucune case ne peut recevoir met le personnage à terre, ce que signifie être mis hors de combat dans les systèmes qui tiennent une telle piste. Guérir efface une marque, la plus légère, quel que soit son type. Les points temporaires sont refusés faute de tampon où les placer. Le moteur lit la piste comme les niveaux RESTANTS ; tout le reste du combat, tomber, être ranimé, le journal et le récapitulatif, reste inchangé.

**Un repos peut guérir une piste de blessures.** Une étape de restauration qui la nomme avec `"to"` la ramène à ce nombre de marques, débordement compris ; avec un `"by"` NÉGATIF, elle en efface ce nombre, débordement d'abord : `"by": { "const": -2 }` en efface donc deux. Une étape qui nomme aussi un `kind` n'efface que les marques de ce type : le "Catch your breath" de Gravewatch s'écrit `{ "track": "harm", "kind": "knock", "to": "min" }`, qui soigne les coups et laisse les déchirures. Une étape qui AJOUTERAIT des marques ne fait rien, car un repos ne marque aucune blessure, et cela inclut un `"by"` positif : il est ignoré sans un mot, alors écris le signe moins.

#### Cases numérotées au lieu de niveaux nommés

Quand une piste est aussi longue que le personnage est robuste, donne-lui `boxes` au lieu de `levels`. Elle a autant de cases que son propre `max`, qui peut être une valeur calculée par la fiche (jamais une valeur qui lit l'état courant), jusqu'à 64 :

```json
{
  "id": "strain",
  "label": "Strain",
  "min": 0,
  "max": { "derived": "strain_boxes" },
  "boxes": {
    "penalty": {
      "by": "remaining",
      "table": [
        [0, -1],
        [1, 0]
      ]
    }
  },
  "kinds": [{ "id": "strain", "label": "S", "severity": 0 }],
  "fill": "indexed",
  "onFull": "refuse"
}
```

- La pénalité active est lue dans `table` (une table à seuils, comme celle d'une valeur dérivée, avec des pénalités inférieures ou égales à 0) au nombre de cases `filled` (remplies) ou `remaining` (restantes), quel que soit ce nombre : une table peut donc coûter quelque chose sans aucune marque. Elle ne dépend pas des cases marquées.
- Les cases sont numérotées : la fiche affiche "Box 3", et le bloc de fiche du Game Master indique combien sont marquées sur combien au lieu de nommer un niveau.

#### Remplir par case, et pistes qui refusent

- `"fill": "indexed"` place une marque sur la case que la commande nomme avec `box=` (la première si elle n'en nomme aucune), ou sur la prochaine case libre au-dessus, et ne déplace jamais une marque une fois posée. Un soin efface la plus haute des marques les plus légères. L'écran de fiche permet au joueur de cliquer sur n'importe quelle case vide.
- `"onFull": "refuse"` refuse une marque qui n'a aucune case libre, au lieu d'aggraver la marque la plus légère. La commande entière est refusée : trois marques avec deux cases libres n'en posent aucune. Une piste indexée refuse toujours ; indique-le avec `"onFull": "refuse"` à côté.

#### Niveaux ajoutés par une liste

`"extra": { "list": "scars", "countColumn": "levels", "penaltyColumn": "penalty" }` sur une piste à niveaux nommés permet à une liste de la fiche de l'allonger pour chaque personnage. Chaque ligne, dans l'ordre, ajoute son nombre de niveaux (16 au plus) avec sa pénalité, après le dernier niveau dont la pénalité est au moins aussi bonne ; ces niveaux prennent le nom de ce niveau quand les pénalités correspondent, et celui de la pénalité sinon. Les cicatrices de Gravewatch fonctionnent ainsi. Une liste cachée par la fiche n'ajoute rien, et si une ligne est supprimée alors que ses niveaux sont marqués, les marques qui ne tiennent plus sont conservées comme débordement : modifier la fiche n'annule pas une blessure.

`boxes`, `fill`, `onFull`, `extra` et le `kind` d'une étape de repos relèvent de Capability API 1.40 pour un ensemble en package.

### La pénalité sur tes jets

`resolution.penaltyFrom` nomme la piste de blessures dont la pénalité touche chaque test de l'ensemble. Elle est déclarée, pas supposée : un ensemble qui l'omet lance exactement comme avant l'existence des pistes.

L'EFFET de la pénalité dépend du type de résolution, comme celui du nombre de la fiche :

- Sous `dice-pool`, elle **retire des dés de la réserve**, sans descendre sous `pool.min`. Un `pool.min` de 1 permet encore de lancer un dé au dernier degré ; un `pool.min` de 0 permet de n'en lancer aucun et d'échouer sans jet.
- Sous `dice-sum`, c'est un **modificateur fixe du jet**, intégré au même nombre auquel contribuent l'attribut et l'entraînement.

Elle doit nommer une piste de blessures. Une piste ordinaire n'a pas de pénalité et est refusée à l'importation. Le résultat indique celle qui s'applique pour expliquer au joueur ses dés en moins ; le bloc de fiche du GM montre aussi le degré et son coût.

### Modificateurs de la fiche

Certains nombres d'une fiche accompagnent chaque jet qu'ils concernent : un sac lourd à chaque escalade, une pénalité d'armure à chaque déplacement discret, une bénédiction sur tout. `resolution.adjust` l'exprime, pour les deux types :

```json
"adjust": [{ "value": { "derived": "burdened" }, "abilities": ["brawn"] }]
```

- `value` est une valeur de la fiche, sous la même forme que celle lue par une valeur dérivée, état courant compris. Un nombre négatif retire. Ember Roads additionne l'encombrement de l'équipement emporté par un personnage en un Burden, et une table à seuils transforme chaque tranche de trois en un point retiré à chaque jet de Brawn. Une table indexée sur un état vivant permet à une forme ou à une posture d'atteindre les dés (voir États vivants).
- `abilities` est facultatif. Avec lui, le modificateur ne s'applique qu'à un test lancé avec l'un de ces attributs : l'attribut lui-même, une compétence ou une sauvegarde qui l'utilise, une compétence lancée avec lui par `with=`, ou une paire d'attributs qui l'inclut. Sans lui, il s'applique à tous les tests.
- Il s'applique au même endroit que la pénalité de blessure : des dés pour une réserve, avec le même plancher `pool.min`, et un nombre fixe pour une somme, dans le modificateur que le registre additionne. Jusqu'à huit entrées, additionnées pour chaque test.
- Un combat régi par l'ensemble l'ajoute à chaque jet qu'il construit à partir d'une fiche, au début du combat : une attaque lancée avec un attribut ou une compétence, une sauvegarde, un test de confrontation et une initiative lus sur un attribut, une compétence ou une sauvegarde prennent les entrées de cet attribut, et tout autre jet ne prend que les entrées valables pour tous les jets.
- Un personnage sans fiche n'en reçoit rien, de même qu'il ne reçoit aucun modificateur.

Le résultat indique ce que la fiche a ajouté (`adjust="-2"` dans le registre, et une ligne sur la carte des dés), pour que le joueur voie pourquoi un jet a donné ce résultat. Cela relève de Capability API 1.38 pour un ensemble en package.

### Repos

Un repos liste des restaurations et des éléments à effacer. Chaque étape nomme une cible (`pool`, `poolGroup`, `listPools`, `track` ou `state`) et la fixe (`"to": "max"`, `"to": "min"` ou un nombre) ou la modifie (`"by": { "const": 1 }` ou `"by": { "fractionOfMax": 0.5 }`). Une étape visant une piste de blessures peut seulement la soigner : son `by` est donc négatif ; voir plus haut. Une étape visant un état vivant le fixe seulement, à `"default"` ou à l'une de ses valeurs.

### Texte du Game Master

- `checkGuidance` remplace le paragraphe intégré qui explique comment demander un test. Nomme le système et les occasions de lancer. Le GM nomme seulement la compétence et la difficulté. Le moteur lance et calcule depuis la fiche : ne demande pas de calculs au modèle.
- `sheetGuidance` introduit les fiches dans le prompt. Explique quelles ressources comptent et quand les dépenser.
- `worldGuidance` est facultatif et lu une fois à la création du monde pour adapter le cadre aux règles : aucune poudre à canon, magie rare, morts qui marchent. Il n'atteint jamais un tour.
- `sheetSummary` choisit les champs, valeurs dérivées et lignes de listes que voit le GM par personnage. Le moteur montre toujours les modificateurs d'attribut, les compétences et sauvegardes entraînées et les valeurs vivantes. Garde le reste court, car il part à chaque tour. La `nameColumn` d'une liste du résumé peut être une colonne texte ou enum (un enum affiche le libellé de sa valeur), et `columns` nomme jusqu'à trois autres colonnes de la ligne à afficher après le nom ; le bloc se lit alors `Gear: Crowbar 1d6` plutôt que des noms seuls. Une colonne booléenne affiche son libellé quand elle est activée. Les deux relèvent de Capability API 1.37 pour un ensemble en package.
- Tu n'as pas besoin d'enseigner la commande de fiche dans `sheetGuidance`. Le moteur enseigne lui-même chaque commande avec les noms que déclare ton fichier ; quand ton ensemble a une piste de blessures, il ajoute la commande `op="damage"` pour la marquer et liste ses niveaux et ses types de blessures. Quand il a des états vivants, il ajoute la commande `op="state"` et liste les valeurs que chacun peut prendre.

## Catalogues : entrées préparées pour les listes de la fiche

Saisir une liste de sorts, un tableau d'équipement ou une page de capacités ligne par ligne est pénible. Un catalogue est une collection nommée d'entrées préparées que tu distribues avec l'ensemble. L'éditeur les propose dans un sélecteur sur chaque liste alimentée par le catalogue ; un choix remplit la ligne.

Les catalogues sont facultatifs. Un ensemble peut en avoir douze ; le moteur ne connaît pas leur sujet : chaque identifiant, colonne, filtre et mot vient de ton fichier.

### L'en-tête

L'en-tête va dans `catalogs`, au premier niveau du fichier, à côté de `gm`.

```json
"catalogs": [
  {
    "id": "knacks",
    "label": "Knacks",
    "feeds": ["knacks", "tricks"],
    "filters": [
      { "id": "grit", "label": "Grit cost", "type": "number" },
      { "id": "road", "label": "Road", "type": "text" },
      { "id": "callings", "label": "Calling", "type": "tags", "startFrom": { "field": "calling" } }
    ],
    "units": { "distance": { "label": "paces", "perCell": 2 } },
    "entries": []
  }
]
```

- `id` et `label` : l'identifiant suit les règles des identifiants de fiche ; le libellé nomme le sélecteur.
- `holds` : `"rows"` (valeur par défaut, et celle de tous les catalogues antérieurs à cette version), `"creatures"` ou `"items"`. Un catalogue de créatures est un bestiaire lu par le combat, et un catalogue d'objets liste ce que transporte une équipe. Aucun des deux n'écrit dans une fiche, ne déclare `feeds` ni n'apparaît dans le sélecteur. Consulte [Créatures](#creatures-a-bestiary-a-fight-reads) et [Objets](#items-what-a-party-carries) ci-dessous.
- `feeds` : les une à huit listes de fiche où ses entrées peuvent écrire. Obligatoire pour les lignes, refusé pour les créatures et les objets. Une entrée ne peut écrire ni dans une autre liste, ni une valeur que ses colonnes ne peuvent contenir.
- `filters` : facultatif, jusqu'à huit. Définit comment réduire le sélecteur : `number`, une valeur `text` ou `tags` (plusieurs mots). `startFrom` nomme le champ utilisé à l'ouverture : un personnage dont Calling vaut Tinker voit d'abord les entrées Tinker.
- `units` : facultatif. Définit le sens d'une portée ou d'une taille de zone dans le bloc `mechanics` d'une entrée.

### Une entrée

```json
{
  "id": "road-sense",
  "label": "Road Sense",
  "summary": "You read a road the way other people read a face.",
  "filters": { "grit": 0, "road": "Ash Flats", "callings": ["Scout", "Courier"] },
  "rows": [
    {
      "list": "knacks",
      "values": { "name": "Road Sense", "notes": "Sneak to notice where a road turns bad." }
    }
  ]
}
```

- `id` : minuscules, chiffres et tirets simples, unique dans le catalogue.
- `label` et `summary` : ce qu'affiche le sélecteur. Le résumé est facultatif, sur une ligne et limité à 300 caractères.
- `filters` : valeurs des filtres déclarés par l'en-tête. `number` prend un nombre ; `text`, une chaîne ; `tags`, une liste de chaînes.
- `rows` : ce qu'écrit le choix, entre une et six lignes. `list` appartient à `feeds` ; les clés de `values` sont les identifiants de colonnes de cette liste.
- `creature` : un adversaire plutôt que des lignes, dans un catalogue dont `holds` indique des créatures. Une entrée a exactement l'un de `rows` ou `creature`. Une créature ne porte pas `mechanics` : elle exprime ses effets dans ses actions.

Chaque valeur est vérifiée contre les colonnes cibles. Un nom mal écrit ou un nombre hors limites est signalé avec son entrée d'origine. Les entrées dans le fichier de règles sont vérifiées à son chargement, donc à l'importation pour un fichier importé. Un catalogue séparé de package est vérifié lorsque le sélecteur le demande pour la première fois ; s'il contient une erreur, il montre les raisons plutôt que des entrées.

### Une entrée, plusieurs listes

Une capacité à usages limités occupe deux lignes : la capacité et son compteur. Cela reste un seul choix.

```json
{
  "id": "last-ember",
  "label": "Last Ember",
  "rows": [
    {
      "list": "knacks",
      "values": { "name": "Last Ember", "notes": "Spend 1 Grit to give a downed friend 3 Grit back." }
    },
    { "list": "tricks", "values": { "name": "Last Ember", "uses": 1, "recharge": "camp" } }
  ]
}
```

### Valeurs maintenues à jour par l'ensemble

Les nombres d'une ligne appartiennent au joueur dès qu'il la choisit. Une exception est utile : un maximum dépendant du personnage, comme des usages égaux à un attribut ou une ressource de classe qui augmente avec le niveau. Une ligne peut nommer jusqu'à quatre de ses colonnes numériques dans `scaled` ; l'éditeur maintient ces cellules à jour.

```json
{
  "list": "tricks",
  "values": { "name": "Last Ember", "uses": 1, "recharge": "camp" },
  "scaled": { "uses": { "from": { "abilityScore": "heart" } } }
}
```

- La clé est l'une des colonnes `number` de la liste.
- `from` est une référence de valeur ordinaire, utilisant le même vocabulaire fermé qu'ailleurs. Pour plus complexe, déclare une valeur `derived` et pointe `from` dessus (`"from": { "derived": "lay_on_hands_max" }`). Aucune nouvelle arithmétique n'est ajoutée ici.
- `table` est facultatif. Il cherche la valeur de référence dans une table à seuils, comme un niveau donnant un nombre : `"scaled": { "max": { "from": { "field": "level" }, "table": [[1, 2], [3, 3], [6, 4]] } }`.
- `values` doit toujours contenir un nombre simple pour la colonne ; son absence est refusée. C'est la valeur avant de connaître une fiche et celle que garde une fiche sans cette référence.
- Une ligne avec `scaled` doit être l'unique ligne de l'entrée pour cette liste ; une ligne marquée correspond ainsi toujours à une seule spécification.

La valeur est calculée à l'édition, jamais à la lecture : une ligne sauvegardée contient toujours le nombre qu'elle annonce. Elle est adaptée à la colonne : bornée par `min` et `max`, puis arrondie vers le bas si la colonne prend des entiers. Dans l'exemple, Heart 3 donne trois usages et Heart 0 ou moins n'en donne aucun. La ligne reste avec 0 usage ; un compteur au maximum de 0 n'étant pas une réserve, rien n'est dépensable en jeu.

Les colonnes à échelle nécessitent Capability API 1.23 pour les ensembles en package. Un ensemble communautaire importé est validé par le moteur qui le lit et n'a besoin de rien d'autre.

### Les lignes choisies sont des copies

Chaque ligne choisie est copiée sur la fiche avec une clé supplémentaire, `_catalog`, contenant `<catalog id>/<entry id>`. Les identifiants de colonnes commencent par une lettre ; cette clé ne peut donc pas être l'une des tiennes.

La copie appartient au personnage. Le joueur peut tout modifier ensuite ; la fiche fonctionne sans l'ensemble installé et publier une version n'écrase jamais les personnages. La marque permet au sélecteur d'afficher ce que possède déjà la fiche et à l'actualisation décrite ci-dessous de le reconnaître.

### Actualiser depuis l'ensemble de règles

Grâce à la marque, l'éditeur peut avertir le joueur si ton nouveau texte diffère de sa ligne. Une courte ligne sous la liste indique combien de lignes ont un texte nouveau ; **Review** (Examiner) montre le contenu de la fiche à côté de celui de l'ensemble, avec une case par ligne. Rien n'est écrit avant **Update selected** (Mettre à jour la sélection) ; seules les colonnes différentes des lignes cochées changent. Tout le reste est conservé, marque comprise.

La comparaison est volontairement étroite :

- Les valeurs existantes ne sont comparées que dans `text`, `longtext`, `dice` et `enum`. Les valeurs `number` et `boolean` appartiennent au joueur et sont conservées, y compris 0 et false. Une colonne encore absente peut être proposée avec sa valeur typée, nombres et interrupteurs compris. Une colonne à échelle est exclue car elle suit déjà la fiche.
- Seules les colonnes définies par ton entrée sont comparées. Une colonne omise n'est jamais touchée, quel que soit son contenu.
- Une valeur que refuserait la colonne, comme un `enum` retiré ou un texte dépassant `maxLength`, est ignorée plutôt qu'écrite.
- La ligne est reliée à celle d'origine par sa position parmi les lignes de même marque dans cette liste, tant que la fiche en garde autant que l'entrée en écrit. Sinon, cela fonctionne seulement si l'entrée écrit une ligne unique pour cette liste. Si le joueur supprime l'une de deux lignes, l'entrée reste intacte plutôt que devinée.
- Une ligne dont l'entrée n'existe plus dans le catalogue est laissée intacte, sans bruit.

Reformuler ou renommer une entrée peut donc atteindre les personnages qui l'ont déjà choisie, s'ils l'acceptent. Changer le sens d'un nombre ne le peut pas et ne le fera pas : cette colonne appartient au joueur depuis qu'il possède la ligne.

### `mechanics` : ce qu'une entrée fait en nombres

Une entrée peut porter un bloc `mechanics` facultatif exprimant ses effets numériques : `kind` (`attack`, `heal`, `buff`, `debuff`, `utility`, `rider`), `range`, `area`, `targets`, `targetCount`, `friendlyFire`, `amount` (dés comme `2d6` ou nombre fixe), `damageType`, `attackRoll`, `autoHit`, `save` (une sauvegarde de la fiche et l'effet d'une réussite), `applies` (états appliqués), `temporary` (points temporaires de santé), `scales` (quantité qui grandit avec la fiche), `cost` (réserve dépensée), `perCostStep`, `budget` (part de l'économie d'actions dépensée), `concentration`, `reaction`, `plus`, `free`, `gives`, `standard`, `rider` et `check`.

Le sélecteur affiche ce bloc sur une ligne. Qui lit le reste dépend du bloc choisi par l'ensemble :

- Avec [`combat`](#combat-a-fight-your-own-rules-resolve), le combat lit ses effets. `range`, `area` et `friendlyFire` s'appliquent sur un champ avec positions ; `reaction` marque une entrée qui répond à quelque chose plutôt qu'une entrée jouée pendant un tour, et nomme le moment qu'elle attend (voir [Fenêtres](#windows-holding-the-fight-open)) ; une entrée marquée simplement `true` n'en nomme aucun et ne figure sur aucun menu. `check` concerne les tests de compétence décrits plus haut.
- Avec seulement [`battle`](#battles-lending-the-sheet-to-marinaras-combat), la bataille lit `kind`, `range`, `area`, `friendlyFire`, `amount`, `damageType` et `cost`, car le combat propre à Marinara peut les utiliser.

Le vocabulaire est fermé : une clé ou valeur absente de cette liste est refusée au lieu d'être ignorée.

`cost` est aussi le prix payé hors bataille par la commande `use` du GM, décrite ci-dessous.

### La commande `use` : laisser le Game Master payer un prix que tu as écrit

Pendant la narration, le GM actualise les fiches avec `[sheet: ...]` : `spend`, `restore` (`heal` a le même sens), `damage`, `temp`, `track`, `condition`, `note` et `rest`. Un ensemble avec catalogues en reçoit une autre :

```
[sheet: who="Mira" op="use" name="Fireball"]
[sheet: who="Mira" op="use" name="Fireball" pool="3rd-level slots"]
```

`op="cast"` équivaut à `op="use"` et `spell=` à `name=` : l'expression choisie par le GM fonctionne sans que ton format connaisse le mot "spell".

Le nom est comparé sans tenir compte de la casse aux lignes du personnage issues de tes catalogues. Une ligne répond au nom montré au GM (colonne de nom `sheetSummary`, puis `pools.nameColumn`, puis première colonne de texte) et au `label` de l'entrée d'origine. Renommer une ligne ne fait donc pas perdre son accès. Un nom sans correspondance et un nom partagé par deux entrées distinctes sont tous deux refusés.

Ce qui est dépensé :

- Chaque terme de `mechanics.cost`. Un terme nommant une réserve vivante paie depuis celle-ci ; un terme nommant un GROUPE paie depuis la première réserve du groupe capable de payer, dans l'ordre de déclaration. Aucune montée automatique vers une réserve supérieure : un groupe n'est pas toujours une échelle.
- Plus un de chaque réserve de ligne écrite par la même entrée, comme un compteur d'usages : la seconde ligne de `Last Ember`. Un compteur au maximum de 0 n'a aucun usage ; la commande est refusée plutôt qu'exécutée gratuitement.

`pool=` représente l'incantation à niveau supérieur : le même prix unique, payé depuis une autre réserve du groupe. Il est accepté seulement si le coût a un terme unique et si la réserve nommée appartient au même groupe. Tout autre cas est refusé plutôt que réinterprété.

C'est tout ou rien. Si une partie du prix ne peut pas être payée, toute la commande est refusée, rien ne change et le joueur est averti. Une entrée gratuite, comme un tour de magie ou une capacité passive, est acceptée sans rien changer.

### Dans le fichier, ou dans son propre fichier

Un petit catalogue est intégré à `ruleset.json`, dans les `entries` de son en-tête. Un long catalogue a son fichier, nommé par `asset` dans l'en-tête. Il possède exactement l'une de ces deux formes.

```json
{ "id": "knacks", "label": "Knacks", "feeds": ["knacks"], "asset": "catalogs/knacks.json" }
```

Le chemin est toujours `catalogs/<the catalog's id>.json`. Le fichier ressemble à ceci :

```json
{ "schemaVersion": 1, "catalog": "knacks", "entries": [] }
```

Les fichiers séparés concernent les packages publiés par le catalogue officiel : le package les liste dans `contributions.assets.paths` avec `ruleset.json` et nécessite Capability API 1.21. Un catalogue de créatures, intégré ou séparé, nécessite Capability API 1.27, et un catalogue d'objets, 1.49. **Un ensemble importé comme fichier unique ou partagé par GitHub contient ses catalogues dans le fichier**, donc ils doivent tenir dans sa limite totale de 256 KB. Cela permet quelques centaines d'entrées courtes.

Les limites sont douze catalogues par ensemble, 2000 entrées par catalogue dans les deux cas et 1 MB par fichier de catalogue.

<a id="items-what-a-party-carries"></a>

## Objets : ce que transporte une équipe

Les armures, les armes, les potions, l'équipement, les munitions et l'argent sont des objets. Un bloc facultatif `items` déclare les mots dans lesquels chaque objet de ton ensemble est écrit, et un catalogue avec `holds: "items"` liste les objets eux-mêmes. Les deux nécessitent Capability API 1.49 ; les `rarityCaps` et `propose` du bloc, qui encadrent les objets inventés par le Game Master, nécessitent 1.51 ; une valeur qui lit les objets que tient un personnage (`itemStat`), 1.52 ; l'effet d'un objet sur les tests quand il est porté ou transporté, avec les plafonds `bonus` du bloc, 1.53 ; ce qu'un objet exige de son porteur et les attributs qu'il modifie, 1.54 ; l'`attack` d'une arme, 1.55 ; ce que fait un objet en combat, 1.56 ; ce qu'une arme tire et garde chargé, 1.57 ; les modes d'une arme, l'attaque de la main secondaire, le plancher et les états infligés sur un coup, 1.58 ; ce que fait l'utilisation d'un objet en combat, avec les charges qu'il contient, 1.59 ; une utilisation qui restaure une réserve, 1.60 ; des charges regagnées au repos ou un objet qui se brise une fois vide, 1.61 ; une utilisation qui demande d'abord un test (`gate`), 1.62 ; les tables de butin et le butin d'une créature, 1.63 ; une ligne de butin qui lâche des pièces ou une couche qui retire des pièces, 1.64 ; un marché, le lieu `sold` d'un objet et un `service`, 1.65 ; un jour de marché (une bourse, `buyBack`, le `stock` d'un vendeur, une quantité `limited` ou un `sold` qui ne nomme aucun lieu), 1.69 ; et un effet supplémentaire qui se déclenche sur les armes tenues (`weapons`), 1.70.

### Le bloc des objets

Le bloc va dans `items`, au premier niveau du fichier. Voici celui d'Ember Roads, un peu raccourci :

```json
"items": {
  "categories": [
    { "id": "weapon", "label": "Weapon" },
    { "id": "armor", "label": "Armor" },
    { "id": "ammunition", "label": "Ammunition" }
  ],
  "rarities": [
    { "id": "common", "label": "Common" },
    { "id": "storied", "label": "Storied" }
  ],
  "tags": [
    { "id": "thrown", "label": "Thrown" },
    { "id": "ranged", "label": "Ranged" },
    { "id": "two_handed", "label": "Two-handed" },
    { "id": "arrow", "label": "Arrow" }
  ],
  "stats": [
    { "id": "bulk", "label": "Bulk", "type": "number", "min": 0, "max": 10, "default": 0 },
    { "id": "guard", "label": "Guard", "type": "number", "min": 0, "max": 4, "default": 0 },
    { "id": "damage", "label": "Damage", "type": "dice", "example": "1d6" },
    { "id": "swing", "label": "Rolls with", "type": "enum", "values": ["brawn", "wits", "heart"], "default": "brawn" },
    { "id": "reach", "label": "Reach", "type": "enum", "values": ["close", "near", "far"], "default": "close" }
  ],
  "slots": [
    { "id": "body", "label": "Body", "count": 1 },
    { "id": "hands", "label": "Hands", "count": 2 }
  ],
  "carry": { "stat": "bulk", "encumberedAbove": { "derived": "load" }, "limit": { "const": 12 } },
  "rarityCaps": [
    { "rarity": "common", "stats": { "guard": 1 }, "bonus": 1 },
    { "rarity": "storied", "stats": { "guard": 3 }, "bonus": 2 }
  ],
  "currencies": [
    {
      "id": "coin",
      "label": "Coin",
      "perWeight": 100,
      "units": [
        { "id": "bit", "label": "bits", "value": 1 },
        { "id": "mark", "label": "marks", "value": 10 },
        { "id": "sovereign", "label": "sovereigns", "value": 100 }
      ]
    },
    {
      "id": "salt",
      "label": "Salt",
      "perWeight": 10,
      "units": [
        { "id": "pinch", "label": "pinches", "value": 1 },
        { "id": "cake", "label": "cakes", "value": 20 }
      ]
    }
  ]
}
```

- `categories` : de une à 24. Chaque objet en a exactement une, comme armure, arme ou potion.
- `rarities` : facultatif, jusqu'à 12. Liste-les de la plus courante à la plus rare.
- `tags` : facultatif, jusqu'à 48. Les propriétés qu'un objet peut avoir, comme être une arme de lancer ou être argenté.
- `stats` : facultatif, jusqu'à 24. Les nombres et les mots que porte un objet, déclarés exactement comme une colonne de liste : `number`, `text`, `boolean`, `enum` ou `dice`, avec une plage, des valeurs et une valeur par défaut quand le type en a. `promptVisible` (par défaut `true`) indique si le Game Master voit la statistique à côté de l'objet. Mets-le à `false` pour ce que seul le joueur doit voir, comme l'endroit où un objet est caché.
- `slots` : facultatif, jusqu'à 12. Où un objet est porté ou tenu, et combien d'emplacements de ce type a un personnage, de 1 à 20.
- `binding` : facultatif. L'harmonisation, l'investiture ou tout autre mécanisme qui limite le nombre d'objets qu'un personnage peut avoir liés à la fois. `label` est son nom, et `max` une valeur lue sur la fiche du personnage, comme `{ "abilityScore": "nerve" }` ou une valeur dérivée. Il ne peut pas lire l'état courant, car le nombre d'objets qu'un personnage peut lier ne change pas à chaque coup.
- `carry` : facultatif. `stat` nomme la statistique numérique qui donne le poids d'un objet, dont le `min` vaut 0 ou plus. `encumberedAbove` est ce qu'un personnage peut porter avant d'être encombré, et `limit` (facultatif) le maximum qu'il peut porter. Les deux sont lus sur la fiche comme `binding.max`. Sans `carry`, le poids ne signifie rien et personne n'est jamais encombré.
- `currencies` : facultatif, jusqu'à six familles de une à dix pièces chacune.
  - Les pièces d'une même famille s'échangent entre elles selon `value`, compté en plus petite pièce de la famille. La plus petite pièce vaut donc 1, et deux pièces d'une famille ne valent jamais la même chose.
  - Deux familles ne s'échangent jamais entre elles. La monnaie d'une autre nation, ou les faveurs d'un univers, forment une famille à part.
  - L'identifiant d'une pièce est unique parmi toutes les familles, car le coût d'un objet ne nomme que la pièce.
  - `perWeight` (facultatif, et seulement à côté de `carry`) indique combien de pièces de la famille pèsent une unité de la statistique de poids.
  - Les pièces sont l'argent de l'équipe, rangées dans les sacs comme n'importe quel objet (voir Argent, ci-dessous).
- `rarityCaps` : facultatif, au plus un par rareté. Le maximum qu'un objet inventé par le Game Master peut donner à cette rareté : la plus grande valeur de chaque statistique numérique nommée dans `stats`, dans la plage de cette statistique et en nombre entier pour une statistique qui prend des entiers, et `bonus` (Capability API 1.53), le plus grand bonus fixe que l'un de ses modificateurs portés ou transportés peut ajouter. À une rareté dotée d'un `bonus`, un bonus en dés est omis, car des dés ne peuvent pas être limités à un nombre ; une pénalité n'est jamais plafonnée. Un objet inventé y est soumis ; les objets listés par tes catalogues t'appartiennent et ne sont jamais plafonnés.
- `propose` : `true` par défaut. `false` interdit au Game Master d'inventer des objets de ton ensemble.
- `native` : `true` par défaut. `false` désactive les objets non typés propres à Game Mode dans les parties de ton ensemble (voir **Ce qui lit les objets**). Le Game Master peut toujours inventer des objets, écrits dans les mots de ton ensemble.
- `freeform` : ce que devient un objet saisi par le joueur. `"plain"` (par défaut) le garde comme objet sans règles, comme aujourd'hui. `"refuse"` n'autorise que les objets de ton ensemble.
- `lootTables` (Capability API 1.63) : facultatif, jusqu'à 24. Ce que lâchent les créatures d'un combat gagné et la commande `[loot:]` du Game Master (voir Butin, ci-dessous).

### Un objet

Un catalogue d'objets ne déclare aucun `feeds` :

```json
{ "id": "outfitter", "label": "Outfitter", "holds": "items", "entries": [] }
```

Chaque entrée porte un `item` au lieu de `rows` ou d'une `creature` :

```json
{
  "id": "hunting-bow",
  "label": "Hunting bow",
  "summary": "Yew, waxed string, and a grip worn smooth by someone else's hand.",
  "item": {
    "category": "weapon",
    "rarity": "common",
    "tags": ["ranged", "two_handed"],
    "stats": { "bulk": 2, "damage": "1d8", "swing": "wits", "reach": "far" },
    "slots": { "hands": 2 },
    "cost": { "amount": 3, "unit": "sovereign" }
  }
}
```

- `id`, `label`, `summary` et `filters` fonctionnent comme pour toute entrée. Le libellé est le nom de l'objet.
- `category` est obligatoire. `rarity` et `tags` sont facultatifs, et tous trois nomment les identifiants propres au bloc. Un objet ne peut avoir de rareté que si le bloc en déclare.
- `stats` : les valeurs des statistiques du bloc. Chacune est une valeur que sa statistique peut contenir.
- `slots` : combien d'emplacements de chaque type l'objet occupe, jamais plus qu'un personnage n'en a.
- `stack` : le maximum d'une pile, de 1 à 999 999. Sans lui, une pile contient autant que n'importe quelle pile de Game Mode. Un objet qui contient des `charges` est seul dans sa pile.
- `cost` : un `amount` entier d'une seule pièce, nommée par son identifiant `unit`. Le sélecteur l'affiche, et le Game Master le voit à côté de l'objet (voir Argent, ci-dessous).
- `sold` (Capability API 1.65) : `{ "place": "city" }`, le plus petit des lieux de ton marché qui vend l'objet, prioritaire sur toute règle contraire ; avec 1.69, il peut à la place, ou en plus, indiquer combien d'exemplaires un lieu garde en stock pour une journée (`"limited": 2` ou `"limited": "1d3"`) (voir Marchés, ci-dessous).
- `service` (Capability API 1.65) : `true` pour un logement, une traversée, une bénédiction : acheté comme un objet et jamais transporté. L'acheter ne fait que payer ; rien d'autre n'en met un dans un sac (un ajout est refusé comme `service`), et le sélecteur omet les services. Un service a un `cost` et rien de ce qui concerne le transport ou l'utilisation (`slots`, `stack`, `binds`, `worn`, `carried`, `requires`, `attack`, `use`, `charges`).
- `binds` : l'objet doit être lié avant de faire quoi que ce soit quand il est porté. `restriction` (facultatif) indique en mots qui peut le lier, et `cursed: true` marque un objet qui ne lâche plus prise. Seul un ensemble avec `binding` peut avoir des objets à lier.
- `worn` et `carried` (facultatifs, Capability API 1.53) : l'effet de l'objet sur les tests et sauvegardes de son porteur quand il est porté, et quand il est seulement transporté, (1.54) les attributs qu'il fixe ou augmente, et (1.56) ce qu'il fait en combat. Consulte [Les tests hors combat](#checks-outside-a-fight) et [Armures et effets portés en combat](#armor-and-worn-effects-in-a-fight).
- `requires` (facultatif, Capability API 1.54) : ce que l'objet exige de son porteur, et ce qui s'applique tant qu'il n'y satisfait pas. Consulte [Les tests hors combat](#checks-outside-a-fight).
- `attack` (facultatif, Capability API 1.55) : ce que fait l'objet comme arme en combat quand il est porté, (1.57) ce qu'il tire et garde chargé, et (1.58) ses autres modes, une attaque de la main secondaire, un plancher pour ses dégâts et les états qu'il inflige sur un coup. Consulte [Les armes en combat](#weapons-in-a-fight), [Munitions et rechargement](#ammunition-and-reloading) et [Modes, seconde arme, plancher et états sur un coup](#modes-a-second-weapon-a-floor-and-conditions-on-a-hit).
- `use` et `charges` (facultatifs, Capability API 1.59) : ce que fait l'utilisation de l'objet, en combat et hors combat, les charges qu'il contient pour cette utilisation, et (1.60) une réserve qu'il restaure. Consulte [Utiliser des objets en combat](#using-items-in-a-fight) et [Utiliser des objets hors combat](#using-items-outside-a-fight).

Un objet ne porte pas de `mechanics` : ce qu'il fait est écrit dans son bloc `item`.

### Ce qui lit les objets

Tout ce qui précède est vérifié à l'importation de l'ensemble, et tes catalogues d'objets sont distribués avec lui. En partie, l'inventaire de Game Mode les lit :

- **Tes objets sont dans le sac.** Une pile peut être l'un de tes objets, et reste cet objet quel que soit le nom que lui donne le joueur. Le bouton **From the ruleset** (depuis l'ensemble de règles) de l'inventaire ouvre un sélecteur de tous les objets listés par tes catalogues, avec la recherche et les filtres du catalogue ; une couche qui masque une entrée la masque aussi là.
- **Les noms trouvent tes objets.** Un nom que saisit le joueur, ou que le Game Master écrit dans `[inventory: action="add"]`, et qui est le `label` de l'un de tes objets, quelle que soit la casse, ajoute cet objet. Si deux objets partagent un libellé, un nom trouve celui que tes catalogues listent en premier.
- **`stack`** est respecté : ajouter, fixer, fusionner ou donner au-delà remplit la pile et en commence une nouvelle.
- **`freeform: "refuse"`** ne laisse au joueur que tes objets : le sélecteur, et les noms qui sont tes objets.
- **`native: false`** ne laisse au Game Master que tes objets et ceux qu'il invente : un nom qui n'est ni l'un ni l'autre est refusé (`not-ruleset-item`), alors qu'on peut toujours ajouter davantage d'un objet déjà tenu, et ses instructions le disent. Un combat ne demande plus à un modèle ce que font les objets de l'inventaire ; seuls tes objets combattent donc : dans un combat régi par l'ensemble, tes armes tenues et les utilisations de tes objets (voir Les armes en combat, ci-dessous), et dans une bataille Classic ou Tactical, tes objets dotés d'un `use` (voir Les objets dans les batailles Classic et Tactical, ci-dessous). Ce que saisit le joueur suit toujours `freeform`, et ce que l'équipe transportait revient dans une nouvelle session dans les deux cas.
- **Ce qu'est un objet** s'affiche sur la pile sélectionnée : sa catégorie, sa rareté et ses tags par leurs libellés, les statistiques qu'il donne, son résumé et le contenu maximal d'une pile. Le sélecteur affiche aussi son `cost`. Le Game Master voit chacun de tes objets tenus avec sa catégorie, sa rareté, ses tags et les statistiques laissées `promptVisible`, comme `Hand axe [Weapon, Common, Thrown; Damage 1d6, Reach close]`.

- **`slots`** : un objet qui occupe des emplacements peut être équipé par celui qui le transporte, tant qu'il a ces emplacements libres, et un exemplaire d'une pile plus grande est placé dans sa propre pile pour être porté. L'inventaire affiche les emplacements utilisés par chaque personnage.
- **`binding`** : un objet qui `binds` peut être lié, jusqu'à `binding.max` lu sur la propre fiche de son porteur (un personnage sans fiche lit une fiche vierge). Un objet `cursed`, une fois lié, le reste : le joueur ne peut ni le délier, ni l'enlever, ni le donner, ni le retirer, et seul le Game Master peut mettre fin à la malédiction.
- **`carry`** : un objet pèse sa valeur de `carry.stat` (un objet qui n'en a pas ne pèse rien), et la charge d'un personnage est le poids de son sac, comparé à `encumberedAbove` et `limit` lus sur sa propre fiche. Un objet ajouté dans la vue partagée de l'inventaire, par le joueur ou par le Game Master sans `who=`, va à celui qui peut le porter sans devenir encombré (le joueur d'abord, puis l'équipe dans l'ordre), réparti selon la place qui reste à chacun quand personne ne peut tout prendre. Rien ne dépasse la `limit` de quiconque : ce que personne ne peut porter est laissé sur place et le Game Master en est informé. Donne à une statistique de poids `"integer": false` pour des poids comme un quart de livre.
- **Le Game Master invente des objets dans tes mots.** Sauf si tu fixes `propose: false`, sa commande `[inventory: action="add"]` peut décrire un nouvel objet : `like=` l'un de tes objets comme point de départ, puis n'importe lesquels de `category=`, `rarity=`, `tags=`, `stats=` (paires `id=value`), `slots=` (`id=count`), `binds=` (`yes`, `cursed` ou `no`), `worn=` et `carried=` (des changements séparés par `;`, chacun `+N`, `-N`, `advantage`, `disadvantage` ou `fails` pour les sauvegardes, sur des compétences ou des sauvegardes par leur nom, ou sur `checks` ou `saves` pour toutes, `+N` ou `-N` sur le nom d'un attribut pour l'augmenter ou le baisser, et, si tu as des combats, `+N`, `-N`, l'avantage ou le désavantage sur les attaques et `+N` ou `-N` sur la défense ou ton mot pour elle : `worn="+1 Sneak; disadvantage on Sway checks"`, `carried="+1 Brawn"`, `worn="+1 Guard"`, ou `none`) et `summary=`, chaque partie par son identifiant ou son libellé. Une statistique d'objet que ta défense compte déjà (la Guard d'Ember Roads additionne la `guard` de ce qui est porté) est la manière dont une armure l'augmente ; un changement porté de la défense à côté d'une telle statistique est donc omis, et la réponse le dit : un petit modèle écrit `guard=1` et `+1 Guard` pour un seul +1. Le moteur lit aussi ce qu'un petit modèle a tendance à écrire à la place : un nombre après le nom (`Brawn +1`), "bonus" après les attaques ou la défense (`+1 attack roll bonus`), `none` pour une liste vide de tags, de statistiques ou d'emplacements, et `worn=`, `carried=` ou `summary=` à l'intérieur de `stats=` quand tu n'as pas de statistique de ce nom. Le moteur ne garde que ce que contient ton bloc : une catégorie, un tag, une statistique, un emplacement, une compétence ou une sauvegarde inconnus sont omis, une rareté que tu n'as pas devient ta plus basse, un nombre est limité à la plage de sa statistique puis aux `rarityCaps` de sa rareté, un bonus porté ou transporté au `bonus` de sa rareté (y compris la partie héritée de `like`), et un nom qui est l'un de tes objets est simplement cet objet. La réponse indique au Game Master ce qui a été changé (jamais à propos d'une statistique que tu ne lui montres pas), et les détails de l'objet montrent chaque changement au joueur. La partie garde l'objet : le même nom désigne cet objet jusqu'à la fin de la partie, et une nouvelle session le conserve tant que quelqu'un le tient encore.
- Le Game Master peut utiliser `equip` et `unequip` sur tes objets avec sa commande d'inventaire quand tu as des `slots`, et `bind` et `unbind` quand tu as `binding` : on ne lui présente que les actions dont ton ensemble dispose. Il voit la charge, les objets liés et les emplacements de chaque personnage, et ce qui est porté ou lié.

- **Ce que fait un objet porté ou transporté** s'affiche sur la pile sélectionnée et dans le sélecteur ("While worn: -1 on checks (Sneak)"), et le Game Master le voit à côté de l'objet (`worn: -1 on checks (Sneak)`). Les tests hors combat l'appliquent (voir Les tests hors combat, ci-dessous).
- **Ce que fait une arme** s'affiche de la même façon ("Attack (Action): Brawn to hit, 1d6 + Brawn cut damage"), et un combat la propose tant qu'elle est tenue (voir Les armes en combat, ci-dessous).
- **Ce que fait l'utilisation d'un objet** s'affiche de la même façon ("Use (Action): heals 1d4 + 1, range 0 paces, used up"), avec les charges qui lui restent. Un combat la propose, tout comme le bouton **Use** (utiliser) de l'inventaire et la commande `[inventory: action="use"]` du Game Master (voir Utiliser des objets en combat et Utiliser des objets hors combat, ci-dessous).

Dans une bataille Classic ou Tactical, chacun de tes objets doté d'un `use` y fait ce qu'il dit, et un objet qui n'en a pas n'est pas proposé (voir Les objets dans les batailles Classic et Tactical, ci-dessous). Quoi que dise `native`, la fiche peut lire tes objets (ci-dessous), une arme tenue est une attaque dans un combat régi par l'ensemble et tire ce qu'elle prend dans le sac, ce que fait un objet porté ou transporté y compte (voir Armures et effets portés en combat, ci-dessous), un objet doté d'un `use` est utilisé selon ses propres règles, en combat et hors combat (voir Utiliser des objets en combat et Utiliser des objets hors combat, ci-dessous), et tes `currencies` sont l'argent de l'équipe (voir Argent, ci-dessous).

<a id="items-on-the-sheet"></a>

### Les objets sur la fiche

Une référence de valeur peut lire les objets que tient un personnage, avec `itemStat` (Capability API 1.52). Ember Roads ajoute la protection de l'armure que porte un voyageur à sa Guard, le nombre qu'un coup doit battre :

```json
{
  "id": "guard",
  "label": "Guard",
  "op": "sum",
  "of": [{ "const": 6 }, { "abilityMod": "wits" }, { "itemStat": { "stat": "guard", "from": "worn", "pick": "sum" } }]
}
```

- `from` : `"worn"` désigne les objets sur le personnage : un objet qui occupe des emplacements tant qu'il est équipé, un objet qui `binds` tant qu'il est lié, et un objet qui fait les deux tant que les deux conditions sont remplies. Un objet qui ne fait ni l'un ni l'autre n'est jamais porté. `"carried"` désigne le reste de ce qu'il tient, et `"all"` les deux.
- `pick` : `"sum"` additionne la valeur de chaque objet multipliée par sa quantité, `"max"` et `"min"` lisent la plus haute ou la plus basse valeur individuelle, et `"count"` compte les objets, selon leur quantité.
- `stat` nomme l'une de tes statistiques d'objet. `sum`, `max` et `min` exigent une statistique `number`. `count` peut l'omettre pour compter tous les objets, ou nommer n'importe quelle statistique pour ne compter que les objets qui la donnent.
- `slot`, `category` et `tag` (chacun facultatif) ne gardent que les objets qui occupent cet emplacement, sont de cette catégorie ou portent ce tag.
- `default` (facultatif, 0 si omis) est ce qui est lu quand aucun objet n'est retenu, ou qu'aucun ne donne la statistique.

Les objets changent en jeu ; `itemStat` suit donc la même règle qu'une lecture de l'état courant : le `max` d'une réserve ou d'une piste, le bonus de maîtrise, `binding.max`, les nombres de `carry` et la colonne à échelle ou la mise à l'échelle d'un catalogue ne peuvent pas le lire, même par une valeur dérivée. Partout ailleurs où un nombre est lu, il le peut : une valeur dérivée, l'`adjust` d'un test, le `cap` d'une compétence, la défense, l'absorption ou l'initiative d'un combat.

Les objets d'un personnage sont ceux de son propre sac. Le personnage du joueur (celui qui porte le nom de la personne qu'incarne le chat, sinon le premier) lit le sac du joueur, et chaque autre personnage, le sien. Seuls les objets de ton ensemble comptent, car un objet ordinaire n'a aucune statistique à lire. La fiche à l'écran, un test, le bloc de fiche du Game Master et un combat à son début lisent tous ce que chaque personnage tient à ce moment-là. Hors partie, dans l'éditeur de fiche ou dans l'examen d'une importation, personne ne tient rien et `itemStat` lit son `default`.

<a id="checks-outside-a-fight"></a>

### Les tests hors combat

Un test ou une sauvegarde que le Game Master demande hors combat lit plus que la fiche : les états actifs du personnage, les niveaux qu'il a atteints, ce que font ses objets portés ou transportés, et ce que coûte l'exigence d'un objet porté tant qu'il n'y satisfait pas. C'est le même vocabulaire que celui des états d'un combat (Capability API 1.53 pour les clés d'objet et la restriction à certaines compétences ou sauvegardes).

Un objet indique ce qu'il fait avec `worn`, qui s'applique tant qu'il est porté (et lié, s'il doit l'être), et `carried`, qui s'applique tant qu'il est seulement transporté. Le manteau de cuir d'Ember Roads grince quand tu te faufiles, et une pierre de route pousse les caravaniers à te faire confiance :

```json
"worn": { "modifiers": [{ "to": "checks", "skills": ["sneak"], "flat": -1 }] }
```

```json
"carried": { "modifiers": [{ "to": "checks", "skills": ["sway"], "flat": 1 }] }
```

Chacun prend les parties d'un état que lit un test :

- `effects` : `own-checks-advantage`, `own-checks-disadvantage`, `own-saves-advantage` et `own-saves-disadvantage`.
- `modifiers` : des changements `to` des tests ou des sauvegardes, avec `flat`, `dice` (et `minus`), `mode`, et leurs propres `skills` ou `saves`, exactement comme ceux d'un état.
- `failsSaves` : les sauvegardes que son porteur rate sans jet.
- `skills` et `saves` : les compétences ou sauvegardes que concernent les effets, et les modificateurs qui n'en nomment aucune.
- `abilities` (Capability API 1.54) : les attributs qu'il modifie, par identifiant, chacun `{ "set": N }` (au moins N : une valeur plus élevée reste) ou `{ "add": N }` (un nombre négatif le baisse). Ils s'appliquent avant le calcul de la fiche ; tout ce qui lit l'attribut lit donc la valeur modifiée : la fiche à l'écran, les valeurs dérivées, les tests, le bloc de fiche du Game Master et un combat à son début. Les additions s'appliquent d'abord, puis le `set` le plus élevé, et les `min` et `max` propres à l'attribut tiennent. Le maximum d'une réserve ou d'une piste et le bonus de maîtrise sont calculés à partir de la fiche seule ; un objet ne les déplace donc pas.

Les gantelets en peau de bœuf d'Ember Roads donnent à une main faible la poigne d'un bouvier :

```json
"worn": { "abilities": { "brawn": { "set": 2 } } }
```

Ce que fait un objet en combat (défense, attaques, vitesse, les autres effets du combat, et les dégâts et états qu'il écarte) se trouve dans [Armures et effets portés en combat](#armor-and-worn-effects-in-a-fight) ; un test hors combat ne lit que les parties ci-dessus. Un objet s'applique une seule fois, quel que soit le nombre de piles que quelqu'un en tient.

Un objet peut aussi exiger quelque chose de son porteur, avec `requires` : une valeur de sa fiche (`value`, toute référence de valeur, lue comme tout ce qui est en jeu), le minimum requis (`atLeast`), et ce qui s'applique tant qu'il n'y satisfait pas (`otherwise`, dans le même vocabulaire, sauf qu'il ne peut pas modifier un attribut, puisque l'exigence peut en lire un). Jusqu'à quatre. La bêche de fossoyeur de Gravewatch est un dur labeur pour un gardien faible :

```json
"requires": [
  {
    "value": { "abilityScore": "sinew" },
    "atLeast": 3,
    "otherwise": { "modifiers": [{ "to": "checks", "skills": ["dig"], "flat": -1 }] }
  }
]
```

Une exigence lit l'attribut tel que les objets du porteur le laissent : des gantelets qui fixent Sinew à 3 satisfont donc celle de la bêche.

Sur un test :

- Chaque modificateur qui le concerne ajoute son nombre, et ses dés sont lancés : un nombre fixe sur un test additionné, des dés sur une réserve. Un test d'attribut, et un test que l'ensemble ne sait pas nommer, ne lisent que ce qui n'est restreint à rien, et une sauvegarde ne lit que ce qui concerne les sauvegardes.
- L'avantage et le désavantage de toutes les sources, y compris le `mode=` du Game Master, s'annulent : dès qu'il y a au moins un de chaque, le test est lancé une seule fois. Seul un ensemble qui lance deux fois (`resolution.advantage`) penche d'un côté.
- Une sauvegarde qu'un état ou un objet fait rater est ratée sans jet, et rien n'est dépensé pour elle.
- Le registre indique ce qui l'a modifié : `effects="-1"` pour le nombre, `from="Leather coat"` pour ce qui l'a modifié, et `automatic="true"` pour une sauvegarde ratée sans jet. La carte des dés montre la même chose, et le Game Master le lit au tour suivant. Il sait que le moteur applique ces modifications, donc il ne les ajoute pas une seconde fois.

Gravewatch montre la même chose sur une réserve : la Dawn bell liée ajoute un dé à Ward, et Rattled en retire un à Soothe et à Barter.

<a id="weapons-in-a-fight"></a>

### Les armes en combat

Un objet peut être une arme (Capability API 1.55). Son `attack` a la forme des [lignes d'attaque](#combat-a-fight-your-own-rules-resolve) d'un bloc de combat, avec des valeurs à la place des colonnes, et un [combat régi par l'ensemble](#combat-a-fight-your-own-rules-resolve) la propose comme attaque tant que l'objet est porté : tenu dans les mains qu'il occupe, et lié s'il doit l'être. Un objet rangé ou seulement transporté ne propose rien, et les lignes d'attaque et les actions propres d'une créature fonctionnent exactement comme avant. La hachette d'Ember Roads :

```json
"attack": {
  "budget": "act",
  "toHit": { "abilities": { "stat": "swing" } },
  "damage": { "dice": { "stat": "damage" }, "abilities": { "stat": "swing" }, "type": "cut" },
  "reach": 2,
  "range": { "normal": 10, "long": 20 }
}
```

- `budget` : le budget qu'elle dépense, l'un de `combat.economy.budgets`.
- `toHit` : ce qu'elle ajoute pour toucher. `abilities` contient un ou plusieurs identifiants d'attributs, et le meilleur compte (une arme de finesse en liste deux). `skill` ajoute ce qu'ajoute un test de cette compétence, avec l'attribut de l'attaque à la place de celui de la compétence, comme pour une ligne d'attaque. `proficiency` est une valeur de la fiche du porteur : si elle vaut plus de 0, le bonus de maîtrise de l'ensemble est ajouté (`{ "const": 1 }` pour toujours). `bonus` est un nombre. `target` est la cible par dé propre à l'attaque de cette arme dans un combat à réserve (la bêche de Gravewatch compte chaque dé à partir de 6 au lieu du 7 de la réserve), et seul un ensemble `dice-pool` dont la `target` peut bouger peut en donner une ; dans un combat par somme, son `bonus` dit la même chose.
- `damage` : les `dice` qu'elle inflige, le meilleur de ses `abilities` de dégâts ajouté, un `bonus` et un `type`. Un combat par somme exige `dice` ; dans un combat à réserve, un coup inflige ses réussites et `dice` ajoute autant de dés supplémentaires du dé de la réserve.
- `reach` et `range` (`normal`, et `long` pour ce qu'elle porte encore au-delà) sont exprimés dans ton unité `combat.distance`, qui doit exister. Une arme dotée des deux est une arme de lancer, comme une ligne d'attaque : une frappe de près, un lancer au-delà. Sans aucun des deux, elle atteint une case.
- `versatile` : les `dice` qu'elle inflige à la place tant que chaque emplacement qu'elle occupe a encore autant de place libre parmi ce que porte son porteur : une lance tenue d'une main, l'autre libre.
- `strikes` : combien d'attaques achète une dépense, lu sur la fiche du porteur, comme le `strikes` d'une ligne d'attaque. Une arme qui ne le déclare pas donne une attaque par dépense, quel que soit le nombre d'attaques de son porteur.

N'importe lequel de ces nombres ou mots peut lire à la place une statistique de l'objet : `{ "stat": "damage" }` est le `damage` de l'objet, et une statistique pour laquelle l'objet ne donne rien vaut comme si la valeur n'était pas écrite. `abilities` et `skill` lisent une statistique enum dont les mots sont des identifiants d'attributs ou de compétences, et `type` une statistique texte ou enum. Ainsi, un objet que le Game Master invente `like=` une arme est aussi une arme, qui combat avec ses propres statistiques, limitées par `rarityCaps` comme toute statistique (une valeur que tu écris en dur est copiée telle quelle). Un objet inventé sans emplacement ni lien n'est jamais porté ; il est donc créé sans l'attaque. Un petit modèle a tendance à décrire une arme en omettant `like=` ; une arme inventée dans une catégorie de tes armes sans point de départ combat donc comme celle de tes armes de cette catégorie dont le nom ressemble le plus au sien (un mot partagé dans un sens ou dans l'autre : un crossbow ressemble donc à un bow), ou comme la première d'entre elles, en prenant toute statistique lue par cette attaque que la proposition a omise, et la réponse le dit ("It fights as Hand axe does."). Le formulaire de proposition du Game Master lui indique aussi qu'une arme faite comme une autre combat comme elle.

Les tags d'une arme sont ce que portent ses coups. L'entrée `resist` ou `immune` d'une créature peut dire ce qui la traverse : `{ "type": "tearing", "except": ["silver"] }` est subi en entier d'une arme avec le tag `silver`, et résisté pour tout le reste. La grave wight de Gravewatch est écrite ainsi, et le clou de cercueil en argent la traverse.

Les détails de l'arme et la ligne du Game Master disent ce qu'elle fait : `attack (Action): Brawn to hit, 1d6 + Brawn cut, reach 2 paces, range 10 to 20 paces`, avec `1d8 with a hand free` pour l'épieu à sanglier d'Ember Roads et `at 6` à côté de la bêche de Gravewatch.

<a id="ammunition-and-reloading"></a>

#### Munitions et rechargement

Une arme peut tirer quelque chose, et garder son propre compte de munitions chargées (Capability API 1.57). L'arc de chasse d'Ember Roads tire des flèches, et le pistolet de guet de Gravewatch contient une balle, chargée à partir du plomb et de la poudre du gardien :

```json
"ammo": { "tag": "arrow", "recover": 0.5 }
```

```json
"ammo": { "tag": "shot" },
"clip": { "max": 1, "reload": "act" }
```

- `ammo` : ce que tire l'arme, par l'un de tes `tags` d'objet. Chaque attaque prend `perAttack` (1 s'il n'est pas indiqué) des objets portant ce tag que transporte son porteur, portés ou non, dans la première pile de ce type de son sac, puis dans la suivante. L'arme n'est proposée que tant qu'il en transporte assez, et le menu de combat indique combien il en reste. Après un combat gagné par l'équipe, `recover` (de 0 à 1) de ce que chacun a tiré revient à la pile d'origine : la part est additionnée sur tout le combat et arrondie vers le bas une fois par pile ; la moitié de cinq flèches tirées d'un même carquois donne donc deux. Un combat perdu ou fui ne rend rien.
- `clip` : un compte chargé que l'arme garde sur elle. `max` est sa capacité, écrite en dur ou lue sur une statistique numérique de l'objet, et `reload` est le budget que dépense un rechargement. Une attaque dépense ce qui est chargé au lieu de prendre dans le sac, et l'arme n'est proposée que tant qu'elle contient assez pour une attaque. Une option **Reload** (recharger) du menu de combat dépense `reload` et la remplit : à partir du sac si elle a `ammo` (autant que le sac contient encore si c'est moins), et entièrement si elle n'en a pas. Le compte est conservé sur la pile d'inventaire de l'arme ; un pistolet vidé lors d'un combat est donc toujours vide au suivant. Une arme dont personne ne s'est encore servi est chargée, tout comme celle qu'un joueur verse dans une pile de plusieurs exemplaires, car un compte chargé appartient à une seule arme. Les munitions d'un chargeur ne sont pas ramassées après un combat ; `recover` concerne donc une arme sans chargeur.

Ce qu'un combat tire, charge et ramasse est écrit dans l'inventaire à chaque étape, comme la santé de l'équipe, et le journal de la partie indique ce qui a été utilisé. Une étape dont la pile a disparu, ou contient moins que ce que le combat comptait, est refusée, comme un objet dépensé que le combat ne trouve pas. Le journal de combat indique chaque tir, rechargement et ramassage ("Hunting bow: 2 left to shoot.", "Ada loads 1 into Watch pistol: 1 of 1 loaded."), et les détails de l'objet et la ligne du Game Master indiquent ce que l'arme tire et contient : `ammunition Arrow (1 an attack, 50% picked up after a won fight)` et `holds 1, reload (Act)`.

<a id="modes-a-second-weapon-a-floor-and-conditions-on-a-hit"></a>

#### Modes, seconde arme, plancher et états sur un coup

Une arme peut avoir d'autres façons de porter son attaque, une partenaire dans la main secondaire, un minimum de dégâts et des états qu'elle inflige à ce qu'elle touche (Capability API 1.58). L'arc de chasse d'Ember Roads peut tirer une volée, et le clou de cercueil en argent de Gravewatch peut être tenu dans chaque main et marque ce qu'il mord profondément :

```json
"modes": [{ "id": "volley", "label": "Volley", "ammo": 2, "toHit": -2, "targets": 2 }]
```

```json
"offHand": true,
"onHit": [{ "condition": "marked", "atLeast": 2, "rounds": 2 }]
```

- `modes` : jusqu'à six autres façons de porter l'attaque, chacune avec un `id`, un `label` et ce qu'elle change. `ammo` est le nombre de tirs d'une attaque dans ce mode ; l'arme a donc un `ammo` ou un `clip` où puiser. `toHit` est ce qu'elle ajoute pour toucher, des dés dans un combat à réserve et un nombre dans un combat par somme. `target` déplace de cette valeur la cible par dé d'un combat à réserve (à partir de celle de l'arme, sinon de celle de la réserve ; `+1` est plus difficile), si la cible de la réserve peut bouger. `targets` indique combien de cibles elle peut viser, chacune avec son propre jet d'attaque. Le menu de combat demande le mode une fois l'attaque choisie (après son style d'initiative, s'il y a des styles), chacun avec son effet attendu, et ne propose que les modes pour lesquels le porteur a les tirs nécessaires. Un membre de l'équipe joué par le moteur évalue chaque mode visant une seule cible, et laisse à un joueur les modes à plusieurs cibles. Dans une fenêtre, une attaque est portée telle quelle.
- `offHand` : une arme pour la main secondaire. Le bloc de combat nomme le budget que dépense une seconde attaque, et indique si ses dégâts gardent un attribut positif :

  ```json
  "offHand": { "budget": "quick", "ability": "full" }
  ```

  Une fois que son porteur a attaqué pendant ce tour avec une arme marquée `offHand`, chaque autre arme de ce type qu'il porte propose une seconde attaque sur ce budget, nommée "Silver coffin nail, off hand" dans le menu. `ability` vaut `full` (par défaut) ou `penalty-only`, qui n'ajoute l'attribut aux dégâts que quand il retire quelque chose, comme le combat à deux armes de 5e. Une attaque de la main secondaire est un seul coup, quel que soit le nombre d'attaques qu'achète l'attaque principale, et jamais une frappe sur quelqu'un qui s'éloigne. Une arme marquée `offHand` exige l'`offHand` du bloc de combat.
- `floor` : le minimum qu'inflige un coup, écrit en dur ou lu sur une statistique numérique de l'objet. Les dégâts d'un combat à réserve après absorption, ou ceux d'un combat par somme, y sont relevés avant qu'une résistance ne les réduise de moitié, et le journal le dit. Un coup qui dépense lance le nombre de son auteur et n'est jamais relevé.
- `onHit` : jusqu'à quatre de tes états que subit la cible quand les dégâts du coup ont atteint `atLeast`, pendant `rounds` de ses propres tours ou, sans lui, jusqu'à ce que quelque chose les retire. Les dégâts sont ce qui l'a atteinte après absorption et résistances ; un coup qui prend retire de l'initiative plutôt que de la santé, donc il n'inflige aucun état.

Les détails de l'arme et la ligne du Game Master disent tout : `modes Volley (2 shots, -2 to hit, up to 2 targets)`, `off hand (Quick)`, `at least 1 on a hit before resistance` et `Marked for 2 rounds when a hit deals 2 or more`.

<a id="armor-and-worn-effects-in-a-fight"></a>

### Armures et effets portés en combat

Ce que fait un objet porté ou transporté peut aussi changer un combat (Capability API 1.56), pour son porteur, à côté de ses états et de ses niveaux. Chacun prend les parties d'un état que lit un combat :

- `effects` : n'importe lequel de ceux d'un état, sauf les quatre qu'un niveau ne peut pas avoir (`half-move-to-stand`, `ends-on-damage`, `cannot-target-source`, `cannot-approach-source`), puisque personne n'a imposé l'objet à son porteur et qu'il ne prend jamais fin tout seul : `own-attacks-advantage`, `attacks-against-disadvantage`, `resist-all`, `speed-zero` et les autres.
- `modifiers` : des changements `to` la défense, les attaques et la vitesse ainsi que les tests et sauvegardes, exactement comme ceux d'un état ; dans un combat `dice-pool`, un changement aux attaques est un nombre fixe de dés, comme pour un état.
- `resist`, `vulnerable` et `immune` : les types de dégâts dont son porteur subit la moitié, le double ou rien, comme pour une créature (et vérifiés contre `combat.damageTypes` si tu en déclares). `conditionImmunities` : tes propres états qui ne sont jamais infligés à son porteur.

L'anneau de veuve maudit de Gravewatch coûte un dé à chaque attaque, sa Dawn bell liée protège son porteur de Rattled, et une pierre de route d'Ember Roads transportée tient la chaleur à distance :

```json
"worn": { "modifiers": [{ "to": "attacks", "flat": -1 }] }
```

```json
"worn": { "modifiers": [{ "to": "checks", "skills": ["ward"], "flat": 1 }], "conditionImmunities": ["rattled"] }
```

```json
"carried": { "modifiers": [{ "to": "checks", "skills": ["sway"], "flat": 1 }], "resist": ["burn"] }
```

Un combat lit chaque objet une fois, nommé d'après la pile ("Widow's ring" à côté d'un jet dans le journal, et sur une défense), comme le fait un test ; l'`otherwise` d'une exigence non satisfaite compte aussi en combat, si bien qu'une armure trop lourde pour son porteur peut coûter de la vitesse. Les détails d'un objet et la ligne du Game Master disent tout ("While worn: -1 on attacks", "worn: +1 on checks (Ward), immune to Rattled"). Une armure qui ajoute simplement à une défense ou à une absorption est une statistique d'objet lue avec `itemStat` (voir Les objets sur la fiche), comme la Guard d'Ember Roads lit le manteau porté.

**Dureté.** Quand l'initiative est un nombre que les attaques déplacent, `combat.pool.hardness` est la dureté d'un combattant, lue sur sa fiche au début du combat (celle de son armure, par `itemStat`), et une créature donne sa propre `hardness`. Un coup qui dépense et dont les dés sont inférieurs à elle touche sans rien faire : le nombre de son auteur revient quand même à la base, le journal le dit ("Ada's Grave spade lands on the warden with 5 dice, below a hardness of 6, and does nothing."), et le menu ne lui prévoit aucun dégât. Seul un ensemble doté d'un style qui dépense peut l'avoir, et une créature écrite sous forme de fiche la prend sur sa fiche.

<a id="using-items-in-a-fight"></a>

### Utiliser des objets en combat

Un objet peut être utilisé en combat (Capability API 1.59) : un cataplasme pressé sur une coupure, un tonique avalé, une cloche qu'on fait sonner. Son `use` indique ce que fait son utilisation dans les mots du `mechanics` d'une entrée de catalogue (voir [Ce qu'un combat lit dans `mechanics`](#what-a-fight-reads-from-mechanics)) : `kind` (`heal`, `attack`, `buff` ou `debuff`), `amount`, `damageType`, `plus`, `attackRoll`, `save`, `applies`, `temporary`, `range`, `area`, `targets`, `targetCount` et `friendlyFire`, avec le `budget` qu'il dépense, ou `free`. Le cataplasme d'Ember Roads, ainsi que le tonique réchauffant et la cloche de l'aube de Gravewatch :

```json
"use": { "kind": "heal", "budget": "act", "range": 0, "targets": "ally", "amount": { "dice": "1d4", "flat": 1 }, "consumes": true }
```

```json
"use": { "kind": "heal", "budget": "quick", "targets": "self", "amount": { "flat": 1 }, "consumes": true }
```

```json
"stack": 1,
"use": {
  "kind": "debuff",
  "budget": "act",
  "targets": "enemy",
  "save": { "save": "steel", "onSuccess": "negates" },
  "saveDifficulty": 7,
  "applies": [{ "condition": "rattled", "duration": { "rounds": 2 } }],
  "charges": 1
},
"charges": { "max": 3 }
```

- Une utilisation figure dans le menu de combat sous **Items** (objets), nommée d'après l'objet ("Poultice") quel que soit le nom de la pile. Un objet qui occupe des emplacements ou qui se lie est utilisé tant qu'il est porté (et lié, s'il doit l'être), comme la cloche ; tout autre objet, tant qu'il est transporté.
- Elle dépense un budget, ou est `free`, et se paie elle-même : un `cost`, `perCostStep`, `check`, `concentration`, `reaction`, `scales`, `gives`, `standard` ou `rider` concerne une entrée de catalogue que contient une fiche, et est refusé sur une utilisation.
- `toHit`, sur une utilisation avec `attackRoll`, est ce qu'elle ajoute pour toucher, comme le dit le `toHit` d'une arme (voir Les armes en combat) : des attributs, une compétence, un bonus et, dans un combat à réserve, une cible par dé. Sans lui, le jet n'ajoute rien.
- `saveDifficulty` : le nombre contre lequel est lancée une sauvegarde qu'elle demande (la sienne, celle qui met fin à un état qu'elle applique, ou celle d'une clause sans `difficulty`), écrit en dur ou lu sur une statistique numérique de l'objet. Une entrée d'une fiche lit ce nombre sur la source de son catalogue ; un objet n'en a pas, il donne donc le sien.
- `consumes: true` retire un exemplaire de la pile de l'objet à chaque utilisation, et le menu indique combien il en reste. Le dernier utilisé emporte la pile avec lui.
- `restore` (Capability API 1.60) rend une partie d'une réserve à chaque fiche qu'il atteint, comme une poche de sang rend du sang : `{ "pool": "resolve", "amount": { "flat": 1 } }`, avec `dice` et `flat` comme dans un `amount`. Il nomme l'une de tes réserves vivantes, jamais la réserve de santé (la santé revient par un soin), et se place sur une utilisation qui aide, un `heal` ou un `buff`. Une créature n'a pas de réserves, elle ne reçoit donc rien. Le journal le dit ("Ada gets back 1 Resolve, and is on 3 of 4."), et un membre de l'équipe joué par le moteur laisse tranquille une réserve pleine. Le tonique réchauffant de Gravewatch restaure un point de Resolve en plus de son soin.
- `charges` dépense ce nombre de `charges` propres à l'objet, dont le `max` est la capacité, de 1 à 100, écrit en dur ou lu sur une statistique numérique de l'objet (un objet qui donne cette statistique à moins de 1 est refusé, et un objet au-delà de 100 en contient 100). Le compte est conservé sur la pile d'inventaire de l'objet : une cloche sonnée deux fois lors d'un combat n'a plus qu'une charge au suivant, et une cloche que personne n'a sonnée est pleine. Les charges appartiennent à un seul objet ; un objet qui en contient a donc un `stack` de 1. Une utilisation consomme l'objet ou dépense des charges, jamais les deux, et les charges d'un objet sont toujours dépensées par son utilisation. Deux clés s'ajoutent à `charges` (Capability API 1.61), comme sur la cloche de l'aube de Gravewatch :

  ```json
  "charges": { "max": 3, "recharge": { "rests": ["vigil"], "amount": "max" }, "breaksOn": { "die": 20, "atMost": 1 } }
  ```

  - `recharge` : lesquels de tes `rests` rendent les charges, et combien : `"max"` remplit l'objet, et une quantité (`{ "dice": "1d6", "flat": 1 }`) en rend autant, sans jamais dépasser `max`. Un repos rend les charges de chaque objet que transporte son personnage et qui le nomme, que ce soit par le bouton Rest de la fiche ou par la commande `[sheet: op="rest"]` du Game Master ; la fiche et le sac sont écrits ensemble, et une réponse régénérée ne recharge qu'une fois. La fiche indique ce qui est revenu après le repos ("Dawn bell 3/3 charges"), et le Game Master voit à côté de chaque objet de l'inventaire les charges qui lui restent ("2 of 3 charges left").
  - `breaksOn` : quand une utilisation dépense la dernière charge, le moteur lance un d`die`, et à `atMost` ou moins l'objet se brise et disparaît du sac, en combat comme hors combat. Le journal de combat le dit ("Dawn bell breaks (a 1 on its die)."), le rapport du bouton Use aussi ("Its last charge spent, it breaks."), et le journal de la partie le compte comme perdu.
- `gate` (Capability API 1.62) est un test que l'utilisateur doit réussir avant que l'objet fonctionne, comme un parchemin d'un sort au-dessus du niveau du lecteur en exige un. `check` nomme ce qui est lancé : `{ "skill": "ward" }`, `{ "ability": "wits" }`, ou une valeur de la fiche (`{ "value": { "derived": "spell_mod" } }`, pour un nombre qui diffère d'un personnage à l'autre, comme le modificateur d'incantation d'un lanceur de sorts 5e). `difficulty` va de 1 à 100, écrit en dur ou lu sur une statistique numérique de l'objet (un objet qui ne la donne pas est retiré du menu, et son bouton Use le dit clairement). `unless` saute le test quand une valeur de la fiche de l'utilisateur vaut au moins `atLeast`. Un test raté consomme l'objet (ou dépense ses charges) pour rien. En combat, le test est lancé au moment où l'objet est dépensé, avec les dés du combat (une réserve dans un combat à réserve, où `difficulty` est le nombre de réussites nécessaires) et le nombre de la fiche lu au début du combat, et ce qu'ajoutent à ce test les états et les objets portés de l'utilisateur compte, y compris ce qui vise la compétence elle-même. Le journal le dit ("Ada rolls Ward to use Page of the vigil litany: 2 successes from 2 dice at 7 or more (8, 9), needing 2 successes, a success."). La page de la litanie de veille de Gravewatch en a un :

  ```json
  "gate": { "check": { "skill": "ward" }, "difficulty": 2, "unless": { "value": { "abilityScore": "nerve" }, "atLeast": 3 } }
  ```

Ce qu'un combat consomme et les charges qu'il laisse sont écrits dans l'inventaire à chaque étape, comme les tirs d'une arme, et le journal de la partie indique ce qui a été utilisé. Le journal de combat indique chaque utilisation ("Poultice: 1 of 2 left.", "Dawn bell: 2 of 3 left."). Un membre de l'équipe joué par le moteur n'utilise un soin que sur quelqu'un de blessé, et personne n'utilise un objet pour frapper quelqu'un qui s'éloigne. Les détails de l'objet et la ligne du Game Master indiquent ce que fait son utilisation : "Use (Action): heals 1d4 + 1, range 0 paces, used up" et `use (Act): Steel 7 save negates it, Rattled, 1 of 3 charges`, et les détails montrent les charges restantes. Un objet que le Game Master invente `like=` l'un des tiens s'utilise comme lui, charges comprises.

### Les objets dans les batailles Classic et Tactical

Un ensemble qui ne résout pas ses propres combats les confie aux batailles Classic et Tactical propres à Game Mode, où l'on demande à un modèle ce que font les objets de l'inventaire au début d'un combat. Tes objets ne sont pas devinés : chacun de ceux dotés d'un `use` fait ce que celui-ci dit, avec les nombres propres au moteur, et l'estimation du Game Master n'est demandée que pour les objets qui ne sont pas les tiens (et pour aucun avec `native: false`).

- Un `heal` soigne et une `attack` blesse d'une part de la santé maximale de la cible, calculée à partir de la moyenne de son `amount` : une moyenne de 7, le `1d8+3` d'une arme de base, représente un peu plus d'un cinquième de cette santé, jamais moins d'un vingtième et au plus la totalité. C'est l'échelle à laquelle le pont de combat lit les nombres d'une entrée de catalogue. Le `damageType` est l'élément.
- Le premier état de `applies` s'applique comme un statut portant son propre nom, pendant ses `rounds` (2 sans eux), et un `buff` ou un `debuff` se réduit à ce statut, qui augmente ou baisse la défense comme les statuts propres au moteur.
- `targets` indique sur qui il est utilisé. Sans lui, un soin ou une amélioration va à un allié, et une attaque ou un affaiblissement à un ennemi.
- `consumes: true` retire un exemplaire de la pile, et une utilisation sans lui laisse l'objet là où il est.
- Un jet pour toucher, une sauvegarde, une zone, des points temporaires et une réserve restaurée n'ont pas de place dans ces batailles, comme pour le `mechanics` d'une entrée de catalogue : la description de l'objet dans le menu les mentionne toujours.
- Un objet qui occupe des emplacements n'est utilisé que tant qu'il est porté, et un objet qui se lie que tant qu'il est lié, comme dans un combat régi par l'ensemble. Un objet consommé est pris parmi ce qui est porté, jamais sur un exemplaire de rechange dans le sac.
- Un objet dont l'utilisation dépense des `charges` est proposé tant qu'il reste une utilisation, et le menu **Items** compte ses utilisations. Chaque utilisation les dépense sur la pile de l'objet, celle du joueur en premier, et la dernière dépensée lance son `breaksOn` : brisé, l'objet disparaît du sac et le journal de la partie le compte comme perdu.
- Un objet qui demande d'abord un test (`gate`) le fait lancer sur le serveur quand un membre de l'équipe l'utilise, avec la fiche de ce membre et ce qu'il porte (un compagnon qui n'a pas sa propre fiche de personnage lance sur une fiche vierge, et seule l'unité du joueur se rabat sur la fiche de personnage du joueur), comme le lance le bouton Use, et le test est sauté quand `unless` est satisfait. En cas d'échec, l'objet est dépensé (ou ses charges le sont) sans effet, et le journal le dit : "Ada rolls Ward to use Page of the vigil litany: 0 against 2, failed, and it is used up for nothing."
- Un objet sans `use` n'est pas proposé dans ces batailles, pas plus qu'un soin sans `amount`.

Le moteur calcule lui-même l'effet de chaque objet ; ce qu'envoie l'écran ne décide donc jamais de ce que fait l'un de tes objets.

### Butin

Un combat gagné lâche du butin dans chaque partie Game Mode, une seule fois, dans les sacs de l'équipe. Sans ensemble de règles, il vient des tables propres à Game Mode (plus abondant et plus rare dans une partie plus difficile), et il en va de même dans un ensemble qui ne déclare aucune table de butin et laisse `native` activé. Un ensemble qui déclare des tables de butin lâche plutôt ses propres objets, et un ensemble avec `native: false` et sans tables ne lâche rien.

`lootTables` dans le bloc `items` (Capability API 1.63) les liste, comme le font les objets funéraires de Gravewatch :

```json
"lootTables": [
  {
    "id": "grave_goods",
    "label": "Grave goods",
    "rolls": "1d2",
    "entries": [
      { "item": "kit/shot-and-powder", "weight": 4, "count": "1d4" },
      { "item": "kit/warming-tonic", "weight": 3 },
      { "item": "kit/litany-page", "weight": 2 },
      { "filter": { "category": "arm" }, "weight": 1 },
      { "coins": "shilling", "weight": 2, "count": "1d6" }
    ]
  }
]
```

- `rolls` indique combien de tirages fait la table : un nombre de 0 à 20, ou des dés (`"1d2"`), 1 par défaut.
- Chaque tirage choisit une ligne selon son `weight` (1 par défaut) par rapport aux autres. Une ligne nomme l'un de tes objets sous la forme `<catalog>/<entry>`, un `filter` par `rarity`, `category` et `tag`, qui choisit uniformément parmi tous les objets correspondants, ou (Capability API 1.64) `coins`, l'une de tes pièces par son identifiant. `count` indique combien en tombent, un nombre ou des dés, 1 par défaut. Un objet ou une pièce qu'une couche retire ne lâche rien, pas plus qu'un filtre qui ne trouve rien.
- Une créature de bestiaire nomme la table qu'elle porte avec `loot` (voir Créatures, ci-dessous). Un combat régi par l'ensemble et gagné lance la table de chaque créature vaincue, et un combat que le moteur ne résout pas selon tes règles n'a pas de créatures de bestiaire, donc il ne lâche rien.
- Ce qui tombe va dans la vue partagée comme un ajout, le sac du joueur étant sollicité en premier, et ce que personne ne peut porter est laissé sur place. Le récapitulatif que reçoit le Game Master indique ce qui est tombé et que c'est déjà dans les sacs, le journal de la partie le note, et une notification l'affiche.
- Le Game Master lance une table dans l'histoire avec `[loot: table="grave_goods" who="Ada"]` (`who=` pour le sac d'un seul personnage), mesuré depuis le début du tour comme ses balises d'inventaire ; une réponse régénérée ne lâche donc le butin qu'une fois. Chaque objet tombé reçoit la même réponse qu'un ajout, et ses instructions listent tes tables.

### Argent

Tes `currencies` sont l'argent de l'équipe. Une pièce est un objet dans les sacs, une pile par pièce comme n'importe quelle autre, nommée par son `label` :

- Elle pèse une unité de ta statistique de poids pour chaque `perWeight` pièces de sa famille, compte dans la charge de son porteur et est placée selon la règle de transport à son arrivée. Une famille sans `perWeight` ne pèse rien.
- Le joueur ajoute des pièces depuis la liste **Coins** (pièces) du sélecteur, et les divise, les donne et les fusionne comme n'importe quelle pile.
- Une ligne au-dessus des piles de l'inventaire affiche les pièces visibles de chaque famille et leur valeur en plus petite pièce de la famille. Le Game Master voit cette valeur à côté de chaque sac (`Coin worth 432 pennies`, ou `no coins` pour une bourse vide, le sac du joueur étant listé même s'il ne contient rien), et le `cost` d'un objet à côté de l'objet (`costs 3 shillings`).
- Le Game Master paie avec `[inventory: action="pay" amount="3 shillings" who="Ada"]` et est payé avec `action="earn"`. `amount` est un nombre et une pièce, par son identifiant ou son libellé, au singulier ou au pluriel, quelle que soit la casse (`1 penny`, `12 pennies`). Un paiement sort d'un seul sac (celui du joueur sans `who=`), uniquement dans la famille de la pièce : les plus grosses pièces qui conviennent partent d'abord, puis la plus petite pièce restante qui couvre ce qui reste dû est rompue, et la monnaie revient en pièces plus petites de la famille, les plus grosses d'abord. La réponse indique ce qui a été payé et ce qui est revenu : 3 shillings tirés d'une bourse contenant une couronne donnent `Paid with crowns ×1; shillings ×2 back.` Un paiement que le sac ne peut pas couvrir est refusé (`cannot-afford`), tout comme une pièce que tu n'as pas (`unknown-coin`) et tout paiement dans un ensemble sans monnaies (`no-currencies`). Un gain est un ajout de cette pièce, dans le sac de `who` ou réparti selon la règle de transport. Le journal de la partie liste ce qui a été dépensé et gagné, et une notification l'affiche. Ses instructions listent tes pièces, et seul un ensemble qui en a les reçoit.
- Acheter, c'est payer puis ajouter, ou, dans un ensemble doté d'un marché, utiliser `buy` (ci-dessous). Il n'y a pas d'écran de boutique.

### Marchés

Un `market` dans le bloc `items` (Capability API 1.65) indique ce que vend un lieu et à quel prix, au lieu que chaque boutique vende tout à son prix catalogue. Celui de Gravewatch, un peu raccourci :

```json
"market": {
  "prices": [
    { "id": "cheap", "label": "cheap", "times": 0.75 },
    { "id": "fair", "label": "fair", "times": 1, "default": true },
    { "id": "dear", "label": "dear", "times": 1.5 }
  ],
  "places": [
    { "id": "hamlet", "label": "hamlet" },
    { "id": "village", "label": "village" },
    { "id": "town", "label": "market town" },
    { "id": "city", "label": "city" }
  ],
  "sold": [
    { "filter": { "rarity": "rare" }, "place": "town" },
    { "filter": { "category": "arm" }, "place": "village" }
  ],
  "sellers": [
    { "id": "chandler", "label": "chandler", "sells": [{ "category": "coat" }, { "category": "tonic" }] },
    { "id": "smith", "label": "smith", "sells": [{ "category": "arm" }], "place": "village" },
    {
      "id": "chapel",
      "label": "chapel of the Vigil",
      "sells": [{ "category": "page" }],
      "place": "town",
      "only": { "value": { "abilityScore": "nerve" }, "atLeast": 3, "label": "wardens of Nerve 3 or more" }
    }
  ]
}
```

- `prices` : de un à huit niveaux, chacun un multiplicateur (`times`, supérieur à 0) du `cost` d'un objet, dont l'un est le `default`. Un prix est le coût en plus petite pièce de la famille multiplié par le niveau, arrondi à cette pièce (jamais moins d'une pièce pour ce qui coûte quelque chose), et exprimé dans la pièce où son coût est écrit si cette pièce le paie exactement, sinon dans la plus grosse pièce, parmi celles que laissent tes couches, qui le paie exactement : à `dear`, un manteau qui coûte 8 shillings en coûte 12, et une potion 5e qui coûte 50 gp coûte 50 pièces d'or, jamais 5 pièces de platine. Le marchandage ou l'humeur d'un vendeur déplace le niveau, jamais le nombre.
- `places` : de une à douze tailles de lieux, de la plus petite à la plus grande, dans tes propres mots.
- `sold` : des règles donnant le plus petit lieu qui vend ce que choisit un filtre. Un filtre nomme une `rarity`, une `category` et un `tag`, n'importe lesquels, et correspond à un objet qui a tous ceux qu'il nomme ; la première règle qui correspond à un objet décide. Le `sold` propre à un objet l'emporte sur les règles, et un objet que rien ne nomme est vendu partout.
- `sellers` : des types de vendeurs, chacun avec ce qu'il vend (`sells`, des filtres dont n'importe lequel choisit un objet), le plus petit lieu (`place`) qui en compte un (partout, sans lui), et facultativement à qui il vend exclusivement (`only`) : une valeur de la fiche de l'acheteur, avec son état courant et ce qu'il transporte, d'au moins `atLeast`, présentée au Game Master sous la forme de `label`. Sans `sellers`, n'importe qui dans un lieu vend tout ce que ce lieu vend.
- Un objet sans `cost` n'est pas à vendre, pas plus qu'un objet qu'une couche masque ou dont une couche a retiré les pièces.

**Où se trouve l'équipe.** Le Game Master indique dans quel lieu se situe une scène et sa taille avec `[place: name="Millbrook" size="market town"]`, la taille par son identifiant ou son libellé, et omet la taille pour un endroit sans marché (une route, la nature sauvage). Le moteur répond à chaque balise sur place, en refusant une taille que tu n'as pas, et le dernier lieu auquel il a répondu dans les réponses du Game Master que voit le joueur, depuis le dernier début de conversation, est le lieu en vigueur jusqu'à ce qu'un autre soit indiqué. Une balise de lieu saisie par un joueur ne compte jamais. Une réponse régénérée ne lit que ce qui précède la réponse qu'elle remplace.

**Acheter.** Le Game Master achète avec `[inventory: action="buy" item="Hand axe" count="1" level="dear" seller="smith" who="Ada"]`, `level` (le niveau par défaut sans lui) et `seller` (n'importe quel vendeur présent qui vend l'objet, sans lui) étant facultatifs. Le moteur fixe le prix selon le lieu en vigueur à la fin de la réponse, retire ce prix de la bourse de l'acheteur (celle du joueur sans `who=`) comme pour un paiement, avec la monnaie, et place l'objet dans son sac, ou se contente de payer pour un service. Soit les deux se produisent, soit aucun. La réponse indique le coût (`price="12 shillings"`) ainsi que ce qui a été payé et rendu, et une notification l'affiche. Un achat est refusé, sans rien changer, si aucun lieu n'est indiqué (`no-place`), si le niveau, l'objet ou le vendeur n'existe pas chez toi (`unknown-level`, `unknown-item`), s'il n'y a pas de prix (`not-for-sale`), si le lieu est trop petit pour l'objet (`not-here`), si aucun vendeur présent ne le vend (`no-seller`), si l'acheteur ne remplit pas l'`only` du vendeur (`not-to-you`), si la bourse ne peut pas le couvrir (`cannot-afford`), ou si le sac ne peut pas le porter.

**Ce que voit le Game Master.** Tant qu'une scène se déroule dans un lieu doté d'un marché, un bloc MARKET nomme le lieu et sa taille, les niveaux de prix, et chaque vendeur présent avec une douzaine de ses articles au niveau par défaut, les moins chers d'abord ; tout ce qu'il vend peut être acheté par son nom. Ses instructions enseignent `[place:]` et `buy`, et seul un ensemble doté d'un marché les reçoit.

#### Un jour de marché

Un jour de marché (Capability API 1.69) fait changer et s'épuiser les étals d'un lieu, et permet à l'équipe de revendre. Celui de Gravewatch, un peu raccourci :

```json
"places": [
  { "id": "hamlet", "label": "hamlet", "purse": { "amount": 10, "unit": "shilling" } },
  { "id": "town", "label": "market town", "purse": { "amount": 100, "unit": "shilling" } }
],
"buyBack": 0.5,
"sold": [{ "filter": { "rarity": "rare" }, "place": "town", "limited": "1d2" }],
"sellers": [
  { "id": "smith", "label": "smith", "sells": [{ "category": "arm" }], "stock": { "table": "smiths_odd_lot" } }
]
```

- **Le jour.** Un jour dans un lieu dure jusqu'à ce que le Game Master nomme un autre lieu ; il commence un nouveau jour au même lieu en le nommant à nouveau avec `day="new"` (`[place: name="Millbrook" size="market town" day="new"]`). Tout ce qui suit est calculé par jour à partir des réponses du Game Master que voit le joueur ; un swipe, une réponse régénérée ou une branche n'ont donc besoin de rien stocker, et un même jour donne toujours le même étal.
- **`limited`** sur une règle `sold` ou sur le `sold` propre à un objet : combien d'exemplaires un lieu garde en stock pour une journée, un nombre ou des dés lancés pour la journée. Une règle peut nommer une limite sans lieu (vendu partout, quelques exemplaires à la fois) ; le lieu d'un objet vient de la première règle qui nomme un lieu, et sa limite de la première qui nomme une limite. Une fois ces exemplaires achetés, un achat est refusé (`sold-out`) jusqu'au jour suivant.
- **`stock`** sur un vendeur : l'une de tes tables de butin, lancée une fois par jour pour ce que ce vendeur a en stock en plus de ce qu'il vend. Ce qu'elle donne (des objets seulement, jamais de pièces) n'est vendu que par ce vendeur, en autant d'exemplaires qu'obtenus, quoi que disent `sells` et `sold` ; l'`only` d'un vendeur s'applique toujours.
- **`buyBack`** : la part (supérieure à 0, au plus 1) du prix par défaut d'un objet qu'un vendeur paie pour le racheter. Un vendeur présent qui vend ce genre d'article le rachète (n'importe qui, sans vendeurs). Sans `buyBack`, personne ne rachète rien.
- **`purse`** sur une taille de lieu : le maximum que ses vendeurs déboursent en une journée, tous ensemble, dans l'une de tes pièces. Les ventes payées dans la famille de cette pièce sont décomptées de cette bourse ; sans bourse, ils achètent sans limite.
- **Vendre.** Le Game Master vend avec `[inventory: action="sell" item="Silver coffin nail" count="1" seller="smith" who="Ada"]`. Le moteur fixe le prix, retire l'objet du sac de celui qui vend (celui du joueur sans `who=` ; jamais un objet maudit lié) et y verse les pièces, les plus grosses d'abord, ou refuse sans rien changer : aucun lieu indiqué, personne ici ne l'achète (`not-bought`), pas de prix (`not-for-sale`), bourse du jour épuisée (`purse-spent`), aucun exemplaire détenu, ou des pièces que le sac ne peut pas porter. La réponse indique ce qu'il a rapporté (`price="4 shillings"`), le journal de la partie le liste comme vendu, et une notification l'affiche.
- Le bloc MARKET ajoute ce que chaque vendeur a en stock aujourd'hui, combien il reste d'un objet limité, ce que paient les vendeurs présents et ce qu'il reste de la bourse du jour ; ses instructions enseignent `sell` et `day="new"`.

<a id="using-items-outside-a-fight"></a>

### Utiliser des objets hors combat

Le même `use` fonctionne hors combat (Capability API 1.60 pour `restore` ; le reste ne nécessite rien de nouveau). Le bouton **Use** de l'inventaire, sur l'un de tes objets doté d'un `use`, et la commande `[inventory: action="use" item="Poultice" who="Juno"]` du Game Master font tous deux ce que fait l'objet à celui qui le transporte, avec les dés du moteur :

- Un soin rend de la santé : la quantité sur une réserve de santé, ou une marque effacée sur une piste de blessures, comme un soin en combat. Les points `temporary` vont sur une réserve de santé, `restore` rend sa réserve, et chaque état qu'il `applies` est infligé, jusqu'à ce que quelque chose le retire.
- Une utilisation qui blesse, ou qui vise le camp adverse (`kind` `attack` ou `debuff`, ou `targets` `enemy`), n'atteint personne : hors combat, il n'y a personne sur le plateau à frapper. L'objet est tout de même utilisé, et le Game Master apprend ce qu'il fait, pour le raconter.
- Elle retire un exemplaire de la pile ou dépense ses charges exactement comme en combat, et un objet qui n'en a plus, ou qui occupe un emplacement ou se lie et n'est pas porté, est refusé sans rien changer. La fiche et le sac sont écrits ensemble.
- Un `gate` est lancé d'abord, comme un test : les dés propres à l'ensemble avec le nombre de la fiche, la pénalité de blessure de l'utilisateur et ce que ses états et ses objets portés ou transportés font à ce test. Un échec consomme l'objet pour rien, et la ligne le dit ("Ada uses Page of the vigil litany: Ward check 1 against 2, failed; it is used up for nothing. 1 left.").

Ce qui s'est passé est dit en une ligne : "Juno uses Poultice: heals 4 (Grit 6/11). 1 left." Le bouton Use l'envoie au Game Master dans un bloc `[item_used]` après "I use my Poultice.", que le chat affiche sous forme de badge ; la balise du Game Master lui-même reçoit la même réponse, mesurée depuis le début du tour comme ses autres balises, si bien qu'une réponse régénérée n'utilise jamais un objet deux fois. Un objet sans `use`, ou dans une partie sans ensemble de règles, est toujours simplement mentionné : "I use my rope."

<a id="battles-lending-the-sheet-to-marinaras-combat"></a>

## Batailles : prêter la fiche au combat de Marinara

Par défaut, une bataille ignore la fiche. Elle construit les combattants comme toujours ; un personnage peut sortir du combat sans changement de ses points de vie sur la fiche.

Le bloc facultatif `battle` change cela dans un seul sens : il prête les nombres au combat et réécrit son résultat. **Il ne fait pas suivre tes règles au combat.** Les calculs, les coups qui touchent et leurs dégâts restent ceux de Marinara. La santé est donc transférée comme proportion du maximum, pas comme ton nombre : une fiche à moitié de sa santé commence à mi-barre de celle construite par Marinara. Ta réserve de 9 points n'est jamais placée directement dans une bataille où un coup fait 12.

```json
"battle": {
  "health": { "pool": "grit" },
  "energy": { "pool": "luck" },
  "skills": [{ "list": "knacks" }]
}
```

- `health` : obligatoire. Réserve vivante représentant les points de vie en combat. Elle doit appartenir à `sheet.live.pools`, pas être une liste dont les lignes sont des réserves.
- `energy` : facultatif. Réserve dépensable par la bataille, qui devient la barre MP. Elle doit différer de `health` : le combat ne peut pas consommer la santé comme énergie.
- `slots` : facultatif. Réserves dépensées une unité à la fois, chacune avec `level` de 1 à 9 : `[{ "pool": "slots_1", "level": 1 }]`. Chaque niveau et chaque réserve ne sont utilisés qu'une fois.
- `skills` : facultatif, jusqu'à huit. Listes dont les lignes deviennent des compétences de combat. Seules comptent les lignes issues de tes catalogues et dont l'entrée porte `mechanics` : une ligne saisie à la main n'exprime rien numériquement. `onlyWhen` nomme une colonne booléenne qui doit être active, comme un sort préparé. `alwaysWhen` nomme une colonne et une valeur qui laissent passer malgré tout, comme un sort sans préparation. C'est l'exception à `onlyWhen`, donc elle est refusée sans lui.

### Ce qui entre et ce qui revient

**À l'entrée**, pour chaque membre dont la partie possède la fiche : la proportion de santé fixe le début sur la barre Marinara ; l'énergie devient MP ; chaque réserve d'emplacements devient ceux de ce niveau ; les lignes marquées deviennent des compétences. Santé maximale, attaque, défense, vitesse et niveau restent les nombres de Marinara. Une réserve de santé à zéro fait commencer à terre ; au-dessus de zéro, le personnage commence toujours avec au moins un point, pour qu'une petite proportion ne le fasse pas tomber par arrondi.

**À la sortie**, quand le combat est terminé : la proportion finale de la barre est convertie à l'échelle de la réserve, et la différence avec le début est appliquée en dégâts ou soins. Énergie et emplacements sont des comptes, pas des proportions ; ils reviennent tels quels. Tout suit les règles des boutons de la fiche ; un changement refusé est ignoré et signalé, pas forcé. Un combat sans changement de santé n'en écrit aucun, pour que les deux conversions ne puissent pas déplacer la fiche à elles seules.

**Dans aucun sens** : jets d'attaque, sauvegardes, concentration et effets d'un coût supérieur. Ils figurent dans `mechanics` pour être lus un jour par un véritable système de combat ; ce pont ne les applique pas et l'ensemble ne doit pas prétendre le contraire.

Une bataille abandonnée ne réécrit rien. Si tu supprimes le message de départ ou si elle ne finit jamais, la fiche reste identique : le combat n'a pas eu lieu.

### Comment une entrée devient une compétence

Le bloc `mechanics` est lu ainsi :

- `kind` devient le type de compétence. Les entrées `utility` et celles marquées `reaction` sont omises, car le combat Marinara n'a pas où les placer.
- `amount` fixe la puissance comme multiplicateur de l'attaque du combattant, pas comme nombre de dégâts. Des dés plus grands ne frappent jamais moins fort ; le multiplicateur reste dans la plage des compétences générées.
- `range` et `area.size` sont divisés par `units.distance.perCell` du catalogue pour obtenir des cases, sans jamais arrondir à zéro. Une explosion prend son rayon, un cône la moitié et une ligne une case. Une zone cible tous les ennemis couverts et respecte `friendlyFire`.
- `damageType` devient l'élément. `targets` n'est pas transféré : Marinara décide des cibles des soins, améliorations et attaques selon le type de compétence.
- Un `cost` d'énergie devient un coût MP ; plusieurs sont additionnés. Un `cost` d'exactement un emplacement en consomme un de ce niveau. Marinara facture une quantité d'énergie ou un emplacement, jamais les deux : il omet deux emplacements, des emplacements de deux niveaux ou un emplacement plus énergie. Les coûts d'autres réserves, comme la santé ou une ressource de classe, sont aussi omis pour ne pas les offrir gratuitement.
- `buff` et `debuff` deviennent les effets propres à Marinara. Les autres promesses du texte, comme retirer un état de la fiche, ne sont pas appliquées. Omets `mechanics` si l'effet n'a de sens qu'en dehors d'une bataille.

`coverage.combat` reste distinct et garde son sens : active-le seulement si les batailles suivent réellement les règles de ton système.

<a id="combat-a-fight-your-own-rules-resolve"></a>

## Combat : une lutte résolue par tes propres règles

Le bloc `battle` prête les nombres de la fiche au combat en gardant l'arithmétique Marinara. Le bloc facultatif `combat` définit comment tes règles RÉSOLVENT le combat. Il paramètre un type de combat détenu par le moteur, comme `resolution` paramètre les tests ; tous ses noms sont les tiens. Il en existe deux : `attack-vs-defense`, où les dés sont additionnés et comparés à un nombre, et `dice-pool`, où un combat lance les réserves propres à ton ensemble `dice-pool` et compte les réussites (voir Un combat lancé en réserves, ci-dessous).

**Une partie dont l'ensemble déclare `combat` combat selon ton bloc.** Les nombres du groupe viennent de leurs fiches, les adversaires de ton bestiaire ou échelle de menace, et chaque tour est résolu par tes dés. Toute dépense ou perte est écrite dans la fiche immédiatement : fermer l'onglet au milieu ne perd rien. L'écran utilise tes mots : attaques et capacités au menu, budgets, états et journal des vrais calculs. Ce qui manque est listé dans la section sur les limites actuelles.

```json
"combat": {
  "kind": "attack-vs-defense",
  "health": { "pool": "grit" },
  "defense": { "derived": "guard" },
  "initiative": { "dice": { "count": 2, "sides": 6 }, "modifier": { "abilityMod": "wits" } },
  "attackRoll": { "dice": { "count": 2, "sides": 6 } },
  "economy": { "budgets": [{ "id": "act", "label": "Action", "per": "turn", "count": 1 }] },
  "attacks": [
    {
      "list": "gear",
      "budget": "act",
      "name": "name",
      "toHit": { "ability": { "column": "swing" } },
      "damage": { "dice": { "column": "damage" }, "ability": { "column": "swing" }, "type": { "column": "harm" } }
    }
  ],
  "abilities": [{ "list": "knacks", "budget": "act" }],
  "standard": ["dodge", "help"],
  "conditions": [
    { "condition": "shaken", "effects": ["own-attacks-disadvantage", "ends-on-damage"] },
    { "condition": "pinned", "effects": ["cannot-act", "speed-zero"] }
  ]
}
```

C'est tout le bloc Ember Roads, et ses parties l'utilisent pour combattre. Le brouillon 5e utilise les mêmes clés pour d20 :

```json
"combat": {
  "kind": "attack-vs-defense",
  "health": { "pool": "hp" },
  "defense": { "field": "ac" },
  "initiative": { "dice": { "count": 1, "sides": 20 }, "modifier": { "derived": "initiative" } },
  "attackRoll": {
    "dice": { "count": 1, "sides": 20 },
    "advantage": true,
    "naturals": { "max": "critical", "min": "miss" },
    "critical": "double-dice"
  },
  "economy": {
    "budgets": [
      { "id": "action", "label": "Action", "per": "turn", "count": 1 },
      { "id": "bonus", "label": "Bonus action", "per": "turn", "count": 1 },
      { "id": "reaction", "label": "Reaction", "per": "turn", "count": 1 }
    ],
    "movement": { "field": "speed" }
  },
  "abilities": [
    {
      "list": "spells",
      "onlyWhen": "prepared",
      "alwaysWhen": { "column": "level", "equals": 0 },
      "budget": "action",
      "toHit": { "derived": "spell_attack" },
      "saveDifficulty": { "derived": "spell_save_dc" }
    }
  ],
  "concentration": { "text": "concentration", "save": "con_save", "floor": 10, "fromDamage": 0.5 }
}
```

### Toutes les clés

- `kind` : `"attack-vs-defense"` (un camp lance les dés contre la défense de l'autre ; toucher inflige des dégâts) ou `"dice-pool"` (voir Un combat lancé en réserves). Chaque clé ci-dessous a le même sens dans les deux cas, lue à la manière dont ce type lit les nombres.
- `health` : obligatoire. Ce que retire le combat : `{ "pool": "grit" }`, une réserve qui diminue et dont le tampon temporaire absorbe les dégâts d'abord s'il existe ; ou `{ "track": "harm" }`, une piste de blessures MARQUÉE. La piste exige `damageKinds` et n'accorde aucun point temporaire.
- `defense` : obligatoire, référence de valeur. Un champ saisi par le joueur ou une valeur dérivée calculée.
- `initiative` : obligatoire. Dés lancés une fois au début et référence facultative de modificateur. Une égalité favorise le plus grand modificateur, puis l'ordre de création du combat. `"each": "round"` relance l'initiative de tout le monde au début de chaque nouvelle ronde, avec le modificateur tel qu'il est alors (un modificateur qui lit une piste de blessures ralentit donc une fois blessé), et la ronde commence par celui qui est premier dans le nouvel ordre. Capability API 1.47. Un combat `dice-pool` peut plutôt la lancer comme une réserve : `pool` (une référence de valeur) indique combien de dés, et leurs réussites plus `plus` donnent le nombre, sans `dice` ni `modifier` à côté. `resource`, qui exige `pool`, conserve ce nombre et laisse les attaques le déplacer (voir "Une initiative que les attaques déplacent", ci-dessous). Capability API 1.48.
- `attackRoll` : obligatoire sous `attack-vs-defense`, et refusé sous `dice-pool`, qui lance les réserves propres à ta résolution. Définit les dés, si l'on lance deux fois pour en garder un (`advantage`), les faces extrêmes d'un seul dé (`naturals.max` : `critical`, `hit` ou `none` ; `naturals.min` : `miss` ou `none`) et l'effet d'un critique (`critical` : `double-dice` relance les dés de dégâts ; `max-dice` ajoute une fois leurs faces maximales ; `none` donne un coup ordinaire). Les faces spéciales exigent un seul dé, comme les tests. Les sauvegardes du combat utilisent les mêmes dés.
- `economy` : obligatoire. `budgets` définit le contenu d'un tour : identifiant, libellé, `per` (`turn` remplit au début du tour du porteur ; `round`, au début d'une ronde) et `count`. Le PREMIER budget est le principal, dépensé par les actions standard. `movement` est une référence facultative de distance parcourue par tour, dans ton unité. Les combats sur plateau la lisent ; voir Positions.
- `attacks` : facultatif. Listes de fiche dont les lignes sont des armes. `name` est la colonne de nom ; `damage.dice`, celle des dés ; `toHit.ability`, `toHit.proficiency`, `toHit.bonus`, `damage.ability`, `damage.bonus` et `damage.type` nomment des colonnes de la même liste. Une colonne `ability` est un `enum` contenant un identifiant d'attribut ; un autre contenu n'ajoute rien. `proficiency` est `boolean` : l'activer ajoute la maîtrise. Sans dés lisibles, la ligne n'est pas une attaque ; une corde reste une corde. `toHit.skill` nomme une colonne `enum` contenant l'un de tes identifiants de compétence : la ligne ajoute ce qu'ajoute un test de cette compétence, avec l'`ability` propre à la ligne à la place de celui de la compétence si elle en nomme un, exactement comme le `with=` d'un test l'échange (Capability API 1.47). `strikes` référence le nombre d'attaques acheté par UNE dépense du budget. Choisir une ligne sans attaques disponibles dépense le budget et met les restantes en réserve. Tant qu'il en reste, toutes les lignes déclarant `strikes` sont gratuites : changer d'arme, de cible ou marcher entre les attaques découle du menu. `strikesCappedBy` nomme une colonne booléenne limitant SA ligne à une seule attaque, quel que soit le nombre acheté : cela sert aux armes tirant une fois par tour, comme la propriété Loading du SRD 5.1. Cela n'a aucun sens, et est refusé, pour une liste qui achète déjà une attaque par dépense. Les attaques disponibles appartiennent au COMBATTANT, pas à une liste : deux listes déclarant `strikes` partagent le même nombre, quelle que soit la ligne utilisée. Elles sont effacées à la fin du tour qui les a achetées. Sans déclaration, chaque dépense achète une attaque, comme auparavant.

  ```json
  {
    "list": "attacks",
    "budget": "action",
    "name": "name",
    "strikes": { "field": "attacks_per_action" },
    "damage": { "dice": { "column": "damage" } }
  }
  ```

- `abilities` : facultatif. Listes dont les lignes marquées par catalogue sont des capacités, filtrées comme `battle.skills` avec `onlyWhen` et `alwaysWhen`. Leurs effets sont le `mechanics` de l'entrée ; le bloc fixe le `budget` par défaut, le `toHit` ajouté si elle lance pour toucher et la `saveDifficulty` des sauvegardes. Une entrée demandant une sauvegarde, propre ou terminant un état appliqué, est refusée si sa liste n'a pas `saveDifficulty` : une sauvegarde contre rien réussirait toujours.
- `standard` : facultatif, liste fermée `dash`, `disengage`, `dodge`, `help`, `hide`, `ready`. `dodge` (attaques reçues lancées deux fois en gardant la pire) et `help` (prochaine attaque de l'allié lancée deux fois en gardant la meilleure) sont toujours résolus. `dash` (un autre déplacement complet) et `disengage` (personne ne frappe pour ton éloignement ce tour) sont résolus sur plateau et seulement enregistrés sans lui. `hide` et `ready` sont acceptés sans effet pour l'instant.
- `standardEffects` : facultatif, effets d'une action standard absents de son indicateur. Seul `dodge` en a aujourd'hui : `{ "dodge": { "saves": ["dex_save"] } }` indique les sauvegardes lancées deux fois en gardant la meilleure tant qu'il dure. Nomme seulement des sauvegardes déclarées, et seulement si `standard` comprend `dodge`. Sans cela, esquiver rend plus difficile à toucher, sans autre effet.
- `conditions` : facultatif. Associe TES identifiants d'états à leurs effets pour que fiche et combat partagent un registre : un personnage empoisonné le reste ensuite. Liste fermée : `own-attacks-advantage`, `own-attacks-disadvantage`, `attacks-against-advantage`, `attacks-against-disadvantage`, `attacks-against-adjacent-advantage`, `attacks-against-far-disadvantage`, `attacks-from-adjacent-critical`, `cannot-act`, `cannot-react`, `speed-zero`, `half-move-to-stand`, `ends-on-damage`, `own-saves-advantage`, `own-saves-disadvantage`, `own-checks-advantage`, `own-checks-disadvantage`, `resist-all`, `cannot-target-source` et `cannot-approach-source`. `failsSaves` nomme les sauvegardes échouées sans jet. Les six effets nécessitant distance ou déplacement (`attacks-against-adjacent-advantage`, `attacks-against-far-disadvantage`, `attacks-from-adjacent-critical`, `speed-zero`, `half-move-to-stand`, `cannot-approach-source`) n'agissent que sur plateau ; voir Positions. `cannot-react` exclut le porteur des fenêtres ouvertes par un déplacement : il n'est jamais interrogé. Quatre clés s'ajoutent :
  - `modifiers` : les nombres qu'il modifie tant qu'il dure. Voir Les nombres qu'un état modifie, ci-dessous.
  - `saves` : sauvegardes visées par les deux effets de sauvegarde et par tout modificateur de sauvegarde. Sans cette clé, toutes ; la déclarer sans l'un ni l'autre à côté est refusé.
  - `skills` : compétences visées par les deux effets de test et par tout modificateur de test qui ne nomme pas ses propres compétences. Sans cette clé, toutes ; la déclarer sans l'un ni l'autre à côté est refusé. Les confrontations d'un combat lancent les tests propres au combat, pas des compétences : ce qui est restreint à des compétences n'en atteint donc jamais aucune, mais un test hors combat sur l'une de ces compétences le lit (voir Les tests hors combat, ci-dessous). Capability API 1.53.
  - `whileSourceInSight` : ce qui compte seulement quand l'auteur de l'état est visible du porteur. `true` conditionne tout l'état ; une liste de ses effets conditionne seulement ceux-ci et laisse les autres, comme une peur empêchant de s'approcher même sans voir la source. Un effet absent de l'état est refusé. Sans plateau, aucune ligne de vue ne peut être rompue et tout compte pareillement.
  - `endsWhenSourceDown` : l'état disparaît dès que son auteur tombe.

  `own-saves-advantage` et son contraire lancent deux fois et gardent un jet, comme les attaques ; ils s'annulent. `own-checks-advantage` et son contraire font de même pour le camp du porteur dans une confrontation, là où un combat fait ses tests d'attribut, et pour ses tests hors combat. `resist-all` réduit de moitié tous les dégâts en plus des défenses propres à la cible et s'annule avec une vulnérabilité de la même manière. `cannot-target-source` interdit de viser l'auteur de l'état avec quoi que ce soit. `cannot-approach-source` interdit de s'en rapprocher davantage que la case actuelle, chemin compris : contourner jusqu'à une case aussi éloignée reste permis ; passer près de lui puis ressortir de l'autre côté ne l'est pas.

  ```json
  { "condition": "restrained", "effects": ["own-saves-disadvantage"], "saves": ["dex_save"] }
  ```

  **Les nombres qu'un état modifie.** Chaque entrée de `modifiers` nomme ce qu'elle modifie, `to`, et de combien :
  - `defense` : le nombre qu'une attaque contre le porteur doit atteindre. Un nombre fixe uniquement.
  - `attacks`, `saves`, `checks` : les jets d'attaque, sauvegardes et tests de confrontation du porteur. Un nombre fixe, ou des `dice` lancés à chaque jet (`"1d4"`) ; `minus: true` retire les dés au lieu de les ajouter.
  - `speed` : la distance que parcourt le porteur, dans ton unité de distance (`flat`), ou `times` 0.5 ou 2 pour la moitié ou le double, appliqué après tous les changements fixes. Lu seulement sur un plateau, comme `speed-zero`.

  Un modificateur de `checks` peut nommer ses propres `skills`, et un modificateur de `saves` ses propres `saves`, qui le restreignent à la place de ceux de l'état. L'un comme l'autre peut faire lancer deux fois avec `mode: "advantage"` ou `"disadvantage"`, compté avec les effets du même nom et s'annulant comme eux, avec ou sans nombre à côté ; un même état peut ainsi rendre Sneak plus difficile et Climb plus facile à la fois. Ces trois clés nécessitent Capability API 1.53.

  ```json
  { "condition": "muddy", "modifiers": [{ "to": "checks", "skills": ["sneak"], "mode": "disadvantage" }] }
  ```

  Un nombre fixe porte son propre signe, et 0 est refusé. Chaque modificateur apparaît dans le journal à côté du jet qu'il a modifié, avec le nom de l'état ("12 + 5 + 3 (Blessed) = 20"), et les chances de toucher et de gagner affichées dans le menu en tiennent compte. Les états se cumulent entre eux ; le même état deux fois reste un seul état.

  ```json
  "conditions": [
    { "condition": "blessed", "modifiers": [{ "to": "attacks", "dice": "1d4" }, { "to": "saves", "dice": "1d4" }] },
    { "condition": "shielded", "modifiers": [{ "to": "defense", "flat": 5 }] },
    { "condition": "slowed", "modifiers": [{ "to": "speed", "times": 0.5 }, { "to": "defense", "flat": -2 }] }
  ]
  ```

- `levels` : facultatif. Des niveaux d'une piste vivante qui comptent comme des états tant que la piste est assez haute : c'est ainsi que s'exprime un état qui empire par degrés, comme l'épuisement. Chaque entrée nomme une `track` ordinaire (pas une piste de blessures), le niveau `at` auquel il commence et ce qu'il fait, avec les mêmes `effects`, `modifiers`, `failsSaves`, `saves` et `skills` qu'un état. Chaque niveau atteint par la piste compte : ils s'additionnent donc à mesure qu'elle monte. Un niveau n'a pas d'auteur et ne prend fin que lorsque ce qu'il lit redescend ; `half-move-to-stand`, `ends-on-damage`, `cannot-target-source` et `cannot-approach-source` y sont donc refusés. Seul un combattant doté d'une fiche a des pistes. Le journal nomme un niveau par sa piste et son nombre ("- 1 (Heat 3)").

  À la place de `track`, un niveau peut lire une valeur `derived` (Capability API 1.54), calculée avec l'état courant et les objets du personnage, pour que personne n'ait rien à cocher : Ember Roads ralentit quiconque transporte 10 d'encombrement ou plus.

  ```json
  { "derived": "bulk_carried", "at": 10, "modifiers": [{ "to": "speed", "flat": -2 }, { "to": "checks", "skills": ["sneak"], "flat": -1 }] }
  ```

  Un seuil qui diffère d'un personnage à l'autre ("plus de cinq fois la Force") est une valeur dérivée qui le soustrait : `scale` par -1 donne la part à retirer, et une `sum` des deux est ce que lit le niveau.

  ```json
  "levels": [
    { "track": "exhaustion", "at": 1, "effects": ["own-checks-disadvantage"] },
    { "track": "exhaustion", "at": 2, "modifiers": [{ "to": "speed", "times": 0.5 }] },
    { "track": "exhaustion", "at": 3, "effects": ["own-attacks-disadvantage", "own-saves-disadvantage"] }
  ]
  ```

  Les modificateurs, les niveaux et les effets de test nécessitent Capability API 1.45.

- `checks` et `contests` : facultatifs. Ce que lit une confrontation, et les confrontations que n'importe qui peut engager en combat : agripper, pousser, se libérer. Voir Confrontations, ci-dessous. Capability API 1.43.
- `concentration` : facultatif. Champ vivant `text` enregistrant l'effet maintenu, `save` imposée par les dégâts, `floor` minimal de difficulté et `fromDamage`, part des dégâts qui la fixe si elle est supérieure. Commencer une autre capacité concentrée termine la première ; rater la sauvegarde la termine et retire les états qu'elle maintenait.
- `dying` : facultatif, `kind: "saves"`. Deux pistes comptant les jets (le nombre nécessaire est leur maximum), `dice`, `succeedAt`, effets des faces extrêmes (`naturals.max` : `revive-1` ou `success` ; `naturals.min` : `one-failure` ou `two-failures`), coût des dégâts à terre (`damageWhileDown`, `criticalWhileDown`) et `condition` du personnage à terre. Sans bloc, zéro santé met simplement à terre et les soins le ramènent.
- `damageTypes` : facultatif. Types du système, comparés sans tenir compte de la casse.
- `damageKinds` : obligatoire si `health` nomme une piste de blessures, refusé pour une réserve : seule une marque conserve un type. Définit les `kinds` marqués par un coup et le nombre de cases. `default` reçoit tout ce qui n'est pas associé, y compris les dégâts sans type ; `byType` associe `damageTypes` aux types de marques sans distinguer la casse : `"Fire"` et `"fire"` sont la même clé et les déclarer ensemble est refusé. `marks` n'a pas de valeur par défaut car ses deux réponses s'opposent : `"per-point"` compte les niveaux de santé selon les dégâts, donc trois dégâts marquent trois cases et les réduire est utile ; `"per-blow"` marque une case si le coup touche, quelle que soit sa force. Un coup à plusieurs clauses marque encore une seule case en prenant le type le plus grave passé. Indique le choix de ton système : `{ "default": "bashing", "byType": { "fire": "aggravated" }, "marks": "per-point" }`.
- `pool` : obligatoire sous `dice-pool`, refusé sous `attack-vs-defense`. Voir Un combat lancé en réserves.
- `spendLimits` : facultatif. Combien d'une réserve vivante un combattant peut dépenser par `turn` ou par `round` en combat, quel que soit son type : `[{ "pool": "blood", "max": { "derived": "blood_per_turn" }, "per": "turn" }]`. `max` est lu une fois au début du combat. Un coût qui dépasse ce qui reste de la limite n'est pas abordable et sort donc du menu, quel que soit le contenu de la réserve. Une limite par tour repart au début du propre tour de son porteur, une limite par ronde au début d'une ronde. Un adversaire écrit en nombres simples ne paie rien sur une fiche, une limite ne le contraint donc jamais ; un adversaire écrit sous forme de fiche paie avec ses propres réserves et est soumis à la limite comme n'importe qui. Capability API 1.47.
- `threat` : facultatif, nécessaire à un bestiaire. `tiers` est l'échelle d'adversaires : identifiant, libellé, plage `health`, `defense`, `toHit`, plage `damagePerRound` et `saveDifficulty`. Chaque créature nomme un palier ; un adversaire non écrit est ramené à celui demandé par le GM, sans sortir de l'échelle. `damagePerRound` décrit ce qu'il inflige à UNE cible par ronde, séquence complète comprise.

### Un combat lancé en réserves : `dice-pool`

Un ensemble `dice-pool` compte des réussites au lieu d'additionner les dés, et ses combats le peuvent aussi. `"kind": "dice-pool"` exige que `resolution.kind` vaille `"dice-pool"`, et il lit la fiche comme le font déjà tes tests de réserve : **chaque nombre qu'un jet ajoute est un nombre de dés, et chaque nombre qu'il doit atteindre est un nombre de réussites.** Rien n'est donc renommé :

- Un nombre pour toucher (celui d'une ligne d'attaque, le `toHit` d'une liste de capacités, celui de l'action d'une créature) est la réserve que lance une attaque. Le dé, la cible qu'un dé doit atteindre, et les faces qui doublent, explosent, annulent ou provoquent une maladresse sont ceux de ta `resolution`. La piste de blessures nommée par ton `resolution.penaltyFrom` retire sa pénalité de la réserve, comme pour un test.
- `defense` est le nombre de réussites dont une attaque a besoin, jamais moins d'une. Le modificateur d'un état à `defense`, et `cover.bonus`, s'y ajoutent.
- Chaque réussite au-delà de celles nécessaires ajoute un dé de dégâts. Une maladresse rate, quoi qu'elle ait compté. Il n'y a pas de critiques : un coup puissant vaut ses dés supplémentaires.
- Le nombre d'une sauvegarde est sa réserve, et sa difficulté le nombre de réussites nécessaires. Une confrontation lance le test de chaque camp comme une réserve, et le camp qui obtient le plus de réussites gagne.
- Le modificateur d'un état aux attaques, sauvegardes ou tests est un nombre de dés ajoutés ou retirés ; c'est donc un nombre `flat` : un modificateur lancé (`dice`) est refusé.
- Les dégâts sont des dés de ton dé : les `dice` d'une quantité (celle d'une créature, d'une entrée) indiquent combien, et doivent être de ton dé, et sa partie `flat` correspond à des réussites automatiques, jamais lancées. La colonne de dés de dégâts d'une ligne d'attaque est lue pour son nombre de dés, `damage.ability` ajoute la valeur de cet attribut en dés, et `damage.bonus` ajoute des réussites automatiques. `perCostStep` et `scales` ajoutent des dés, comme toujours. Les soins et les points temporaires sont des quantités, additionnées comme avant.
- L'initiative est une somme (`initiative.dice` et `modifier`), ou une réserve si tu en demandes une (`initiative.pool`, ci-dessous), et une règle d'agonie garde ses propres dés.

Ce qu'un combat à réserve lance en plus se trouve dans son bloc `pool` :

```json
"pool": {
  "advantage": true,
  "damageTarget": 6,
  "soak": { "roll": true, "byKind": { "knock": { "abilityMod": "sinew" } } }
}
```

- `advantage` : si un jet qui penche est lancé deux fois en gardant celui qui a le plus de réussites (une maladresse compte comme moins que tout jet sans maladresse). Sans lui, un combat lance une seule fois, quoi que dise un état.
- `damageTarget` : la cible par dé contre laquelle sont lancés les dés de dégâts et d'absorption. La cible par défaut de ta résolution si tu l'omets. Ces dés comptent une fois chaque face égale ou supérieure, et rien d'autre : aucune face n'y double, n'explose, n'annule ni ne provoque de maladresse.
- `soak` : ce qu'une cible retire des dégâts d'un coup, par type de blessure. `all` est une référence de valeur pour tous les types, et `byKind` en donne une par type de ta piste de santé, qui l'emporte sur `all` pour son type ; un type que ni l'un ni l'autre ne nomme n'est pas absorbé. Avec `"roll": true`, la cible lance ce nombre de dés contre la cible de dégâts et chaque réussite en retire un. Avec `"roll": false`, le nombre est retiré des dés de dégâts avant leur lancer : il ne touche donc jamais les réussites automatiques, que l'absorption lancée retire comme les autres. Rien ne descend sous zéro, et rien n'est lancé pour absorber un coup qui n'a rien compté. Une créature donne ses propres nombres sous la forme `soak: { "all": 1, "byKind": { "knock": 3 } }`, ce qui exige que le `soak` du bloc indique comment l'absorption se fait. La résistance, la vulnérabilité, l'immunité et `resist-all` s'appliquent après l'absorption, à ce qui reste.
- `hardness` : une référence de valeur pour la dureté d'un combattant, quand l'initiative est un nombre que les attaques déplacent. Voir [Armures et effets portés en combat](#armor-and-worn-effects-in-a-fight).

Les dégâts de chaque cible sont lancés pour cette cible, car ce qu'un coup a gagné au-delà des réussites nécessaires, et ce qu'elle absorbe, lui appartiennent. La seconde clause d'un coup, et un effet supplémentaire, forment chacun leur propre réserve de dégâts de leur propre type, sans dés supplémentaires du coup. Le menu prévoit une réserve exactement : la chance d'atteindre les réussites nécessaires, et ce que valent les dés de dégâts après ce qu'absorbe la première cible.

Gravewatch est écrit ainsi : ses gardiens lancent une valeur et un métier, absorbent les coups avec Sinew et pas du tout les déchirures, peuvent dépenser un point de Resolve par tour, et relancent l'initiative à chaque ronde.

**Déclarer dans l'ordre inverse n'est pas implémenté.** Certains systèmes font déclarer son action à chacun avant que quiconque n'agisse, du plus lent au plus rapide. Dans un combat où chaque combattant choisit une action quand vient son tour, déclarer en premier ne change rien de ce que lit une règle ; une clé n'aurait donc rien à dire.

### Une initiative que les attaques déplacent

Certains systèmes à réserve gardent l'initiative comme un nombre pour tout le combat et laissent les attaques le déplacer : une façon d'attaquer la prend à la cible, une autre dépense celle de l'attaquant comme dégâts, et quiconque tombe jusqu'à une ligne fixée s'effondre. Un combat `dice-pool` peut l'exprimer (Capability API 1.48) :

```json
"initiative": {
  "pool": { "abilityMod": "nerve" },
  "plus": 3,
  "resource": {
    "base": 3,
    "styles": [
      { "id": "press", "label": "Press", "takes": { "gain": 1 } },
      { "id": "telling", "label": "Telling blow", "spends": { "onMiss": [[0, 1], [6, 2], [11, 3]] } }
    ],
    "crash": { "at": 0, "condition": "reeling", "bonus": 5, "recoverAfter": 3 }
  }
}
```

- **L'ouverture.** Chacun lance `pool` dés au début du combat, et les réussites plus `plus` forment son nombre ; `resource` exige donc `pool` : des dés additionnés donnent un ordre, pas un nombre de dés. L'`initiativeModifier` d'une créature est sa réserve, puisque tout nombre qu'ajoute un combat à réserve est en dés. Le nombre est conservé : il n'est jamais relancé (`each` est refusé à côté de `resource`), et au début de chaque ronde l'ordre est trié selon les nombres du moment.
- **Styles.** Chaque attaque (une ligne d'attaque, une capacité ou une entrée de catalogue qui lance pour toucher et inflige des dégâts, l'action d'une créature qui en inflige, et une action faite d'autres actions) est proposée une fois par style, et un joueur choisit le style avant la cible. Jusqu'à quatre ; chacun prend ou dépense, et au moins un prend. Un choix qui ne nomme aucun style se fait dans le premier ; un choix dans un style où l'attaque n'est pas proposée est refusé. Une attaque garde le style dans lequel elle a été faite jusqu'à sa fin, même quand une réponse change le nombre de son auteur avant qu'elle ne touche.
- **Un style qui prend.** Sur un coup au but, les dégâts sont lancés comme d'habitude (les dés supplémentaires du coup, les dés de l'arme, l'absorption par type, les réussites automatiques), et ce qu'ils comptent est retiré du nombre de la cible plutôt que de sa santé. L'attaquant gagne le tout plus `gain`.
- **Un style qui dépense.** Proposé seulement tant que le nombre de l'attaquant est au-dessus de la ligne d'effondrement (0 si tu ne donnes pas de `crash`), et jamais pour une action faite d'autres actions, puisqu'un nombre se dépense sur un seul coup. Sur un coup au but, les dégâts sont le nombre de l'attaquant au moment de l'attaque, en dés contre la cible de dégâts, et rien d'autre : ni dés d'arme, ni dés supplémentaires du coup, ni réussites automatiques, ni absorption. Il marque la santé selon le type de blessure propre à l'attaque. Une fois l'attaque terminée, un nombre qui a touché quelque part revient à `base`, et un nombre qui n'a rien touché perd ce qu'indique `onMiss` pour le nombre avec lequel l'attaque a été faite (une table à seuils : paires `[at least, lose]`, croissantes ; rien sous le premier palier).
- **L'effondrement.** Un nombre qui tombe à `crash.at` ou en dessous s'est effondré : la `condition` (l'un de tes états, facultatif) est infligée, et celui qui l'a fait tomber là gagne `bonus`. Un échec qui fait s'effondrer son propre auteur ne donne de bonus à personne. Un combattant effondré ne peut pas dépenser. L'effondrement prend fin, état compris, quand son nombre repasse au-dessus de la ligne, après `recoverAfter` de ses propres tours (son nombre revient à `base` au début de ce tour), et quand le combat se termine, quelle qu'en soit l'issue. Quelqu'un dont le jet d'ouverture est sur la ligne ou en dessous commence le combat effondré. `base` doit être au-dessus de `crash.at`, sinon y revenir le ferait s'effondrer à nouveau.

Le menu montre la prévision propre à chaque style : la chance de toucher, et ce que prendrait un style qui prend ou les dégâts que ferait un style qui dépense. Le panneau d'état montre le nombre de chacun, et le journal indique chaque changement et sa raison : "Ada gains 5 initiative for crashing Rats, and is on 14." Le sélecteur du moteur évalue un coup qui prend un tour à l'avance : prendre puis dépenser, comparé à dépenser maintenant puis de nouveau depuis la base.

<a id="what-a-fight-reads-from-mechanics"></a>

### Ce qu'un combat lit dans `mechanics`

`kind` détermine si `amount` représente des dégâts ou des soins ; toute entrée marquée `reaction` reste hors du menu, tout comme une entrée `utility`, sauf si elle modifie ce que le tour lui-même peut contenir (voir ci-dessous). `attackRoll` effectue un jet contre la défense de la cible avec le `toHit` de la liste ; `autoHit` saute entièrement cette étape. `save` lance la sauvegarde de la cible contre la `saveDifficulty` de la liste, et `onSuccess` détermine si une réussite subit la moitié ou rien. `targetCount` indique combien de cibles elle peut viser. Une capacité sans jet d'attaque (une zone contre laquelle chacun effectue une sauvegarde, quelque chose qui touche directement) lance ses dés UNE FOIS pour toutes les cibles ; une capacité qui lance pour toucher chaque cible relance ses dés à chaque coup au but. `applies` impose des états à ce qu'elle affecte, chacun avec une `duration` de `instant` (sans horloge propre : il reste jusqu'à ce que quelque chose le retire), `until-save` (qui exige `saveEnds` à côté) ou `{ "rounds": n }`, et éventuellement un `saveEnds` qui nomme la sauvegarde et indique si elle se répète à `turn-end` ou `turn-start`. Les rondes se décomptent sur les propres tours du porteur, à la fin de chacun ; `{ "rounds": 1, "at": "turn-start" }` les décompte au début de chacun, ce qui permet de dire "jusqu'au début de ton prochain tour". `endsAfter` retire l'état après le premier `own-attack` (le prochain jet d'attaque du porteur), `attacked` (le prochain jet d'attaque fait contre le porteur) ou `own-save` (la prochaine sauvegarde du porteur), quoi que dise son horloge ; `{ "duration": { "rounds": 1 }, "endsAfter": "own-attack" }` signifie donc "sur sa prochaine attaque avant la fin de son prochain tour". Un état qui dure le temps d'une attaque dure toute cette attaque, dégâts compris, et un nouvel état infligé par le même coup reste. Les deux nécessitent Capability API 1.45. `temporary` accorde des points temporaires à la réserve de santé, qui ne se cumulent jamais : la plus grande protection reste. `scales` augmente la quantité du nombre de DÉS supplémentaires que sa table donne pour la valeur lue. `cost` se paie par la commande `use` de la fiche elle-même, et `budget` remplace la partie de l'économie dépensée.

`plus` contient jusqu'à trois quantités SUPPLÉMENTAIRES sur le même coup, à côté de `amount`, chacune avec son propre jet et son propre type ("and 2d6 fire"). Une clause est `{ "dice": "2d6", "flat": 1, "type": "fire" }` et peut porter sa propre `save`, `{ "save": "con_save", "difficulty": 13, "onSuccess": "none" | "half" }`, que lance la CIBLE indépendamment de ce que l'action lui demandait déjà : `none` supprime toute cette clause en cas de réussite, `half` en laisse la moitié, et le reste du coup demeure intact dans les deux cas. Sans `difficulty`, elle reprend le nombre utilisé par la sauvegarde de l'action, puis la `saveDifficulty` de la liste. Un critique double les dés de chaque clause selon la même règle que ceux de la première quantité ; une clause sans `type` reprend le type de dégâts du coup, et le coup entier reste UN test de concentration, avec les dégâts additionnés, et un test de chute. Une clause nécessite un `amount` à accompagner, et un `heal` n'en porte aucune.

```json
{
  "kind": "attack",
  "attackRoll": true,
  "amount": { "dice": "1d8" },
  "damageType": "piercing",
  "plus": [{ "dice": "2d6", "type": "fire" }]
}
```

Trois clés indiquent ce qu'une entrée fait à l'économie du tour lui-même, et une entrée `utility` qui en déclare une est proposée au lieu d'être écartée :

- `free` : elle ne coûte aucun budget. Elle paie toujours le `cost` indiqué et ne peut pas aussi nommer un `budget`.
- `gives` : `[{ "budget": "action", "count": 1 }]`, jusqu'à quatre. L'utiliser ajoute immédiatement à ces budgets, avec un plafond égal à ce qu'un tour contient plus le don, pour que rien ne puisse être mis de côté pour un tour ultérieur.
- `standard` : `{ "actions": ["dash", "disengage", "hide"], "budget": "bonus" }`. Son détenteur peut effectuer ces actions standard avec CE budget. Elles sont proposées à côté des actions ordinaires sous la forme `standard:<id>@<budget>`, et l'entrée elle-même reste hors du menu lorsqu'elle n'est que cette permission, car une permission n'est pas quelque chose que l'on exécute.

Une entrée du nouveau `kind: "rider"` est PASSIVE : personne ne l'exécute, elle n'apparaît jamais au menu et ajoute automatiquement une clause de dégâts au premier coup au but admissible d'une période. Elle porte `rider` et rien d'autre qui puisse être exécuté :

```json
{
  "kind": "rider",
  "rider": {
    "on": "hit",
    "sources": ["attacks"],
    "requires": { "column": "finesse" },
    "when": ["advantage", "ally-adjacent"],
    "oncePer": "turn",
    "amount": { "dice": "1d6" }
  },
  "scales": {
    "from": { "field": "level" },
    "table": [
      [1, 0],
      [3, 1]
    ]
  }
}
```

`sources` nomme les listes d'attaques dont elle provient et `requires` une colonne vraie de leurs lignes ; ainsi, un effet supplémentaire qui ne se déclenche qu'avec certaines armes les précise sans qu'Engine sache ce qu'est une arme. N'indiquer ni l'un ni l'autre signifie n'importe quel coup au but de son détenteur. Une arme tenue depuis le sac n'est pas une ligne ; `weapons` (Capability API 1.70) nomme donc aussi les tags d'objet sur lesquels il se déclenche : `"weapons": { "tags": ["finesse",
"ranged"] }` se déclenche sur un coup au but avec une arme tenue portant l'un ou l'autre tag, frappe de la main secondaire comprise. Ne nommer que `weapons` ne se déclenche sur aucune ligne, et ne nommer que des lignes ne se déclenche sur aucune arme tenue. `when` accepte N'IMPORTE LAQUELLE des options : `advantage` est l'orientation finale du jet d'attaque, et `ally-adjacent` désigne un allié de l'attaquant debout et capable d'agir, à une case de la cible sur un plateau et n'importe où sans plateau. `oncePer` vaut `turn` (renouvelé au début de chaque tour, de sorte qu'un coup porté pendant qu'un autre agit peut encore en bénéficier) ou `round`. `amount` augmente avec le `scales` de l'entrée, et `type` est le type de dégâts, celui du coup par défaut.

<a id="creatures-a-bestiary-a-fight-reads"></a>

### Créatures : un bestiaire lu par le combat

Un catalogue qui déclare `"holds": "creatures"` contient des adversaires au lieu de lignes de fiche. Il n'alimente aucune liste, le sélecteur de l'éditeur de fiches ne le propose jamais, et tous ses nombres utilisent les clés déjà déclarées par ton bloc `combat`. Il nécessite un bloc `combat` et une échelle `threat`, car une créature est classée sous l'un de tes propres paliers.

```json
{
  "id": "road_trouble",
  "label": "Road trouble",
  "holds": "creatures",
  "filters": [{ "id": "tier", "label": "How bad", "type": "text" }],
  "entries": [
    {
      "id": "rust-jackal",
      "label": "Rust Jackal",
      "summary": "A lean thing that lives on the metal roads.",
      "filters": { "tier": "Pack trouble" },
      "creature": {
        "health": { "dice": "3d6" },
        "defense": 6,
        "initiativeModifier": 1,
        "speed": 16,
        "abilities": { "brawn": 1, "wits": 0, "heart": -1 },
        "tier": "pack",
        "actions": [
          {
            "id": "bite",
            "name": "Bite",
            "budget": "act",
            "toHit": 2,
            "damage": { "dice": "1d6", "flat": 1, "type": "cut" },
            "reach": 2
          },
          {
            "id": "worry",
            "name": "Worry",
            "budget": "act",
            "toHit": 2,
            "damage": { "dice": "1d4", "type": "cut" },
            "applies": [{ "condition": "shaken", "duration": { "rounds": 2 } }]
          },
          {
            "id": "snap_and_worry",
            "name": "Snap and worry",
            "budget": "act",
            "sequence": [
              { "action": "bite", "times": 1 },
              { "action": "worry", "times": 1 }
            ]
          }
        ]
      }
    }
  ]
}
```

Les nombres ci-dessous sont la manière directe d'écrire une créature. Une créature écrite dans les termes de ton ensemble de règles, sous forme de `sheet`, tire plutôt `health`, `defense`, `initiativeModifier`, `speed`, `abilities`, `saves`, `checks`, `soak` et `hardness` de cette fiche (voir "Une créature écrite dans les termes de ton ensemble de règles", ci-dessous).

- `health` : un nombre, ou `{ "dice": "3d6", "flat": 2 }`, lancé une fois à la création du combat. Une prévision lit la moyenne, donc un menu ne promet jamais un dé que personne n'a lancé.
- `defense`, `initiativeModifier`, `speed` : ce contre quoi l'attaque est lancée, ce qui est ajouté à l'initiative (ou, quand l'initiative est lancée comme une réserve, combien de dés elle lance) et la distance parcourue en un tour, dans ton unité de distance.
- `abilities` et `saves` : indexés par les identifiants de caractéristiques et de sauvegardes déclarés par ta fiche. Une sauvegarde absente vaut zéro.
- `checks` : ce qu'elle ajoute dans une confrontation, indexé par les identifiants de `combat.checks`. Un test qu'elle ne nomme pas vaut zéro. Capability API 1.43.
- `soak` : dans un combat `dice-pool` uniquement, ce qu'elle absorbe : `all` pour toute blessure, `byKind` pour un type de ta piste de santé. Capability API 1.47.
- `hardness` : quand l'initiative est un nombre que les attaques déplacent, un coup qui dépense et dont les dés sont inférieurs à cette valeur ne fait rien. Capability API 1.56.
- `resist`, `vulnerable`, `immune` : types de dégâts comparés sans tenir compte de la casse et vérifiés contre `combat.damageTypes` si tu en déclares. Une entrée `resist` ou `immune` peut aussi dire ce qui la traverse, `{ "type": "tearing", "except": ["silver"] }` : un coup porté par une arme (un objet) qui porte l'un de ces tags d'objet est subi tel quel (Capability API 1.55, voir
[Les armes en combat](#weapons-in-a-fight)). `conditionImmunities` nomme tes propres états.
- `tier` : le barreau de `combat.threat` auquel elle appartient.
- `loot` (Capability API 1.63) : l'identifiant de la table de butin qu'un combat gagné lance pour elle (voir Butin, ci-dessus).
- `traits` : de courtes paires nom/texte montrées au Game Master. Elles ne sont jamais résolues, donc tout ce qui contient des nombres appartient à une action.
- `signaturePoints` : points rendus au début de son propre tour et dépensés pour les actions `signature`.
- `riders` : jusqu'à quatre, identiques au `rider` d'une entrée de catalogue, écrits sur le bloc. Chacun vaut `{ "id": "pack", "name": "Pack", "on": "hit", "oncePer": "turn" | "round", "amount": { "dice": "1d6" } }`, avec un `type` facultatif et un `actions` facultatif qui nomme les actions de ce bloc sur lesquelles il se déclenche. L'effet supplémentaire d'un bloc ne lit aucune liste de fiche : `sources` et `requires` sont donc les deux clés qu'il ne possède pas. Une créature écrite sous forme de fiche reçoit les effets supplémentaires de ses listes, exactement comme un personnage.
- `actions` : jusqu'à douze, chacune avec son propre `id`. Une action porte ce qu'un bloc de statistiques écrit à la main porte (`toHit`, `autoHit`, `damage`, `save`, `applies`, `targetCount`, `reach`, `range`, `area`), plus quatre éléments propres aux créatures. `reach` est sa portée de frappe, `range` sa portée de lancer ou de tir et `area` sa forme d'impact, le tout dans ton unité de distance ; `range` peut être un simple nombre ou `{ "normal": 30, "long": 120 }` si elle porte plus loin avec une pénalité, et `area` vaut `{ "shape": "burst" | "cone" | "line", "size": n, "friendlyFire": false }` (voir Positions) :
  - `uses` : `{ "per": "encounter" | "day", "count": n }`. Lorsqu'ils sont épuisés, l'action quitte le menu.
  - `recharge` : `{ "dice": { "count": 1, "sides": 6 }, "from": 5 }`. Elle commence le combat disponible, est dépensée à l'utilisation, puis lance les dés au début du propre tour de la créature : `from` ou plus la rétablit. Le journal contient les dés dans les deux cas.
  - `sequence` : d'autres actions du même bloc, dans l'ordre, chacune avec sa propre cible. **C'est ainsi que s'écrit une créature qui frappe deux fois en une action.** Un budget paie toute la séquence. Une séquence ne porte rien de propre et ne peut jamais nommer une autre séquence.
  - `signature` : `{ "cost": n }`, payée avec les points de la créature plutôt qu'un budget et uniquement pendant qu'un autre agit : le combat la propose dans la fenêtre entre deux tours (voir Fenêtres).
  - `reaction` : le même objet que le `mechanics.reaction` d'une entrée de catalogue (voir Fenêtres) : l'action ne figure sur le menu d'aucun tour et est proposée à la place au moment qu'elle nomme. Une réaction n'est pas aussi une action distinctive, et aucune séquence ne peut en faire une.
  - `self: true` : elle s'applique à la créature elle-même plutôt qu'à quelqu'un d'autre, et ne prend donc ni `targetCount` ni `area`. Les deux nécessitent Capability API 1.46. Une parade (Parry) est les deux, avec un état `parrying` qui ajoute 2 à la défense :

    ```json
    {
      "id": "parry",
      "name": "Parry",
      "budget": "reaction",
      "self": true,
      "reaction": { "on": "hit" },
      "applies": [{ "condition": "parrying", "duration": { "rounds": 1 }, "endsAfter": "attacked" }]
    }
    ```
- Une sauvegarde nécessite une difficulté sur l'action elle-même : `save.difficulty` pour une sauvegarde imposée par l'action, ou `saveDifficulty` pour un état qui prend fin sur sauvegarde lorsque l'action n'a pas sa propre sauvegarde. Une action de bloc s'écrit avec des nombres simples même sur une créature dotée d'une fiche : ce nombre réside donc sur l'action. La sauvegarde d'une clause peut omettre sa `difficulty` et reprendre ce même nombre.
- `damage.plus` est la même liste de clauses que le `plus` d'une entrée de catalogue, et se lit exactement de la même manière : `"damage": { "dice": "1d6", "flat": 2, "type": "piercing", "plus": [{ "dice": "1d4", "type": "fire" }] }` est une morsure qui porte la chaleur comme quantité distincte, avec sa propre résistance et son propre doublement.

Le bestiaire du brouillon 5e contient cinq créatures écrites à la main dans `docs/development/ruleset-5e-2014.example.json`, couvrant une séquence, une recharge, une sauvegarde avec un état, des résistances et immunités, des utilisations limitées, des points d'actions distinctives et une créature écrite sous forme de fiche.

#### Une créature écrite dans les termes de ton ensemble de règles

Une créature ne doit pas forcément s'écrire avec des nombres simples. Donne-lui plutôt une `sheet`, avec exactement la structure d'une fiche de personnage, et le combat la construit comme un membre du groupe : sa santé, sa défense, ses sauvegardes, son initiative, sa vitesse et chaque attaque et capacité de ses listes résultent de tes propres formules de fiche. C'est ainsi qu'un ensemble de règles indique que ses adversaires possèdent les mêmes caractéristiques, compétences et listes que ses personnages, quelles qu'elles soient. Le Toll Warden d'Ember Roads :

```json
{
  "id": "toll-warden",
  "label": "Toll Warden",
  "creature": {
    "tier": "pack",
    "traits": [{ "name": "Knows the road", "text": "It will not follow anyone past the last milestone." }],
    "sheet": {
      "abilities": { "brawn": 2, "wits": 1, "heart": 1 },
      "skills": { "sway": "trained" },
      "fields": { "calling": "Hauler", "toughness": 3 },
      "lists": {
        "gear": [{ "name": "Toll hook", "swing": "brawn", "damage": "1d6", "harm": "cut" }],
        "knacks": [{ "name": "Hold the Line", "_catalog": "knacks/hold-the-line" }]
      }
    }
  }
}
```

- **Chaque partie est facultative** : `abilities`, `skills`, `saves`, `bonuses`, `fields` et `lists`, indexés par les identifiants déclarés dans ta fiche. Tout élément omis prend la valeur par défaut de ta fiche, exactement comme sur un personnage vierge. Le Grit du gardien vaut 9 parce que ton `grit_max` additionne 4, sa Toughness et son Brawn, et sa Guard vaut 7 pour une raison similaire.
- **Chaque nombre n'a qu'une source.** Une créature avec une fiche ne fournit pas aussi `health`, `defense`, `initiativeModifier`, `speed`, `abilities`, `saves` ou `checks`, et Engine refuse le fichier si elle le fait. Elle peut n'avoir aucune `actions` propre, puisque ses listes définissent ce qu'elle fait. Une créature sans fiche fournit toujours les trois premiers et au moins une action.
- **Elle est vérifiée comme les données d'auteur qu'elle est.** Chaque identifiant doit être déclaré par ta fiche ; une compétence ou sauvegarde prend l'un des paliers de maîtrise que tu lui proposes ; un champ, score, bonus ou colonne contient ce qu'il est déclaré contenir (un entier dans son intervalle, une de ses valeurs, etc.), et une liste ne contient pas plus de lignes qu'elle n'en autorise. Il n'y a pas de partie `live`, car le combat conserve les dépenses de la créature.
- **Une ligne peut provenir d'un catalogue.** `_catalog: "<catalog>/<entry>"` nomme l'entrée dont une ligne a été sélectionnée, comme sur une fiche de personnage, et le combat lit dans cette entrée le coût et l'effet de la ligne. Le catalogue doit alimenter cette liste. Quand le catalogue est intégré au fichier, l'entrée doit s'y trouver ; quand il possède son propre fichier, une ligne qui nomme une entrée absente de ce fichier ne donne simplement rien à la créature. Les catalogues nommés par les fiches d'un bestiaire sont chargés pour le combat avec le bestiaire.
- **Ce que l'entrée indique à côté de la fiche compte toujours** : `tier`, `traits`, `actions`, `signaturePoints`, `riders`, `resist`, `vulnerable`, `immune` et `conditionImmunities`.
- **Elle paie dans ses propres réserves.** Elles commencent pleines ; elle les dépense pour ce que lui donnent ses listes, et les paiements supérieurs (un sort avec un emplacement supérieur) lui sont proposés exactement comme à un membre du groupe, qu'Engine ou le Game Master décide pour elle. Hold the Line coûte de la Luck au gardien.
- **Avec une piste de blessures, sa santé est la piste.** Un coup marque la propre piste de la créature selon tes `damageKinds`, après application de ses `resist`, `vulnerable` et `immune` ; une créature immunisée à un type de dégâts n'en reçoit donc aucune marque.
- **Elle reste un adversaire.** À zéro, elle est hors combat plutôt que mourante ; elle ne lance jamais de sauvegarde contre la mort ; l'écran montre ce qu'il montrait toujours d'un adversaire et rien de sa fiche, et aucune de ses dépenses n'est réécrite ailleurs, même si un personnage porte le même nom.
- **Une fiche dont la santé totale est nulle** est exclue du combat, et le journal d'ouverture explique pourquoi, plutôt que de faire entrer quelque chose que personne ne peut blesser.
- Une couche qui retire une valeur d'un de tes champs énumérés ne supprime jamais une créature qui l'utilise : tant que la couche est active, ce champ prend sa valeur par défaut pour la créature, exactement comme pour un personnage, et la créature n'est pas refusée pour cette raison.
- Un Game Master peut aussi en inventer une, qui reste limitée à son palier (voir Adversaires que personne n'a écrits).
- Un package qui en fournit une déclare Capability API 1.34.

Le Toll Sergeant du brouillon 5e est la même chose sur une fiche d20 : son Armor Class, ses points de vie, ses sauvegardes et ses deux frappes par action proviennent de ses propres champs et de sa liste d'attaques.

#### Adversaires que personne n'a écrits

Lorsqu'un Game Master invente un adversaire, Engine ramène la proposition sur ton échelle `threat` avant tout jet : la santé dans la plage du palier ; la défense, le bonus d'attaque et les difficultés de sauvegarde au maximum à deux au-dessus du palier ; et les dégâts diminués jusqu'à ce que le meilleur round de la créature (sa séquence la plus forte ou son action individuelle la plus forte, mesurée contre une cible) entre dans le `damagePerRound` du palier. Il réduit d'abord le nombre de dés, puis la partie fixe, puis une frappe d'une séquence, et seulement ensuite la taille du dé, sans jamais réduire quoi que ce soit à néant. Les noms absents de ton ensemble de règles sont retirés : types de dégâts, états et sauvegardes inconnus, ainsi que tout ce qui dépasse les six premières actions. Un palier jamais déclaré se rabat sur le bas de ton échelle. Chaque changement est renvoyé sous forme d'une phrase simple pour que le journal puisse expliquer son intervention.

Une invention peut aussi s'écrire sous forme de `sheet`, comme une créature de bestiaire ; c'est ainsi qu'un mage inventé obtient emplacements et sorts. Le Game Master voit les identifiants de ta fiche et ce que chacun peut contenir, les listes lues par le combat et les noms que tes catalogues leur proposent ; ainsi, un sort est nommé plutôt que décrit : une ligne qui nomme une entrée de catalogue, quelle que soit sa casse, devient cette entrée, et les propres valeurs du Game Master (comme la préparation d'un sort) s'appliquent par-dessus. La fiche est lue avec tolérance, car un modèle l'a écrite : un nom absent de ton ensemble de règles est retiré, une valeur est ajustée à son champ ou sa colonne, et les nombres écrits à côté de la fiche ne sont pas utilisés.

Une créature inventée qui n'est pas un boss est limitée à ce que ton ensemble de règles lui ouvre. Un filtre de catalogue avec `startFrom` nomme le champ de fiche qui organise ses entrées (la liste de sorts du package 5e par `class`), et une créature inventée ne conserve que les entrées dont le filtre correspond à sa propre valeur de ce champ, selon la comparaison utilisée à l'ouverture du sélecteur. Ainsi, un Sorcerer n'a jamais toute la liste de sorts, et une créature sans classe indiquée ne reçoit rien d'un catalogue organisé par classe. Les choix laissés ouverts sont ensuite complétés sans interroger à nouveau le Game Master : pour chaque liste dont les lignes ne comptent qu'une fois choisies (`onlyWhen` sur une source de capacités de combat), parmi les entrées accessibles qu'elle peut payer avec ses propres réserves, chaque réserve qu'elle possède reçoit un petit nombre d'entrées (davantage pour une créature plus compétente), tout comme ce qu'elle peut utiliser à volonté. Ce qu'elle reçoit dépend de son tempérament et de sa compétence, les mêmes qui gouvernent son combat : une créature protectrice ou de soutien cherche ce qui soutient son camp ; une téméraire, les dégâts ; une méthodique ou patiente, ce qui entrave un ennemi ; et plus elle est compétente, plus elle risque de porter une réaction, un contre ou tout autre élément qui modifie le tour. Le tirage vient de la graine du combat, donc un même combat complète toujours de la même façon. Une ligne que le Game Master a nommée dans une telle liste compte comme choisie.

Un boss appartient entièrement au Game Master, en tant qu'exception possible : rien ne lui est retiré et rien n'est complété.

Puis l'un comme l'autre est limité à son palier :

- La santé entre dans la plage du palier par l'unique champ dont elle est lue : le maximum de la réserve est ce champ, ou une `sum` contenant exactement un champ (le maximum de points de vie de 5e, la Toughness d'Ember Roads). Une formule de santé sans champ unique reste telle quelle, et le journal le précise. La longueur d'une piste de blessures t'appartient et ne change jamais.
- Une fois la créature construite, la défense, le bonus d'attaque et les difficultés de sauvegarde sont limités à deux au-dessus du palier, et les dégâts diminuent jusqu'à ce que son meilleur round entre dans le `damagePerRound` du palier, en comptant le plus gros paiement qu'elle peut se permettre. Ce qu'achète un paiement supérieur diminue d'abord, puis les dés, la partie fixe, une frappe, et seulement ensuite la taille du dé.

Ton propre bestiaire n'est jamais plafonné. Ce sont des données que tu as écrites, donc Engine les accepte telles quelles.

### Positions : un combat sur un plateau

Un combat se déroule dans l'imagination jusqu'à ce que ton bloc indique la valeur d'une case de plateau. Déclare `distance` et il peut se jouer sur une grille ; mouvement, allonge, portées, zones, ligne de vue, couverture et frappes sur quelqu'un qui s'éloigne prennent alors un sens. Chacun est un nombre que tu as écrit ; Engine fournit le plateau et rien d'autre.

```json
"distance": { "label": "ft", "perCell": 5 },
"ranged": { "long": "disadvantage", "adjacentFoe": "disadvantage" },
"cover": { "bonus": 2 },
"opportunity": { "budget": "reaction" }
```

Ember Roads n'en déclare qu'une ligne, et rien d'autre ; c'est tout l'intérêt : le reste n'est pas obligatoire.

```json
"distance": { "label": "paces", "perCell": 2 }
```

**La case.** `distance.perCell` indique combien de TON unité vaut une case, et `label` nomme cette unité. Toutes les distances du monde du bloc l'utilisent : `economy.movement`, la `speed` d'une créature, le `reach` et le `range` d'une arme, et le `reach` et le `range` d'une action de créature. Un catalogue qui déclare son propre `units.distance` convertit ses `mechanics.range` et `area.size` avec son propre `perCell` ; sinon, il utilise celui-ci. Une distance supérieure à zéro est arrondie à la case la plus proche, jamais à aucune : tout ce à quoi tu as donné un nombre atteint donc au moins une case. Zéro n'est pas une courte distance et conserve son sens propre : un `mechanics.range` de 0 signifie soi-même ou contact (toucher quelqu'un d'autre atteint la case voisine), et une colonne d'arme `reach` ou `range` valant 0 sur une ligne signifie que cette ligne n'a pas cette distance.

**Le combat utilise-t-il un plateau ?** Deux éléments doivent concorder : ton bloc déclare `distance`, et la partie du joueur utilise le style de combat Tactical. Avec le style Classic, ou un ensemble de règles sans `distance`, le combat reste dans l'imagination : n'importe qui peut viser n'importe qui, et rien de ce qui suit n'est lu.

**Ce que voit le joueur.** Le plateau est dessiné avec le terrain du style tactique. Chaque case est un bouton, accessible au pointeur ou aux touches fléchées, et indique sa nature, son occupant et ce qu'en fait le choix en cours. Marcher éclaire les cases proposées par le menu, chacune avec son coût DANS TON UNITÉ, trace le chemin et marque en ambre toute case dont le chemin provoquerait la frappe de quelqu'un, nommé sous le plateau. Une option qui prend une cible éclaire les personnes admissibles, à la fois sur le plateau et dans la liste. Une option avec une `area` vise une case, et celle sous le pointeur indique qui serait touché, alliés compris. Le mouvement restant apparaît à côté de tes budgets, toujours dans ton unité. L'écran ne mesure rien : chaque case, coût, chemin, cible et point de visée est envoyé par le serveur.

**Mouvement.** Le déplacement autorisé d'un tour vaut `economy.movement` pour un membre du groupe, ou la `speed` de la créature, divisé par `perCell` et arrondi VERS LE BAS, sans jamais descendre sous une case tant qu'elle peut bouger. Il se recharge au début du propre tour de son détenteur et peut se dépenser avant, entre et après les actions : marcher, frapper, marcher encore. Entrer dans une case coûte un, ou davantage en terrain difficile. Huit directions, toutes au même coût, car les grilles de jeu de rôle sur table visées fonctionnent ainsi. On peut traverser un allié mais s'arrêter sur personne ; un adversaire est un mur ; on ne peut entrer dans rien de solide ni couper un coin entre deux cases solides.

**Allonge et portée.** Une ligne d'arme les tire de `combat.attacks[].reach` et `.range`, chacun étant une colonne de la même liste ou le même nombre sur toutes les lignes :

```json
"attacks": [
  {
    "list": "attacks",
    "budget": "action",
    "name": "name",
    "toHit": { "ability": { "column": "ability" } },
    "damage": { "dice": { "column": "damage" } },
    "reach": { "column": "reach" },
    "range": { "normal": { "column": "range" }, "long": { "column": "long_range" } }
  }
]
```

Une colonne qui vaut 0 sur une ligne indique que cette ligne n'a pas cette distance ; c'est ainsi qu'une épée ordinaire figure dans la même liste qu'une hache de lancer. Une ligne sans allonge atteint une case. Une action de créature utilise son propre `reach` ou `range`, et une capacité de catalogue utilise `mechanics.range` (0 signifie soi-même ou contact, soit une case lorsqu'elle vise quelqu'un d'autre).

Une ligne avec LES DEUX est une arme de lancer : dans son allonge, c'est une frappe ; au-delà, un tir. Les règles de tir ci-dessous ne l'affectent donc pas en main, et elle permet de frapper quelqu'un qui passe, contrairement à un arc.

Une action de créature peut aussi porter l'`area` dans laquelle elle touche, dans ton unité : `{ "shape": "cone",
"size": 15 }`, avec `"friendlyFire": false` pour épargner son camp. C'est ainsi qu'un souffle devient un vrai cône sur le plateau plutôt qu'un nombre de cibles. Une séquence n'a pas de forme propre ; les actions qu'elle nomme ont les leurs. Un combat sans plateau ignore la forme et utilise `targetCount` ; une entrée de créature peut donc porter les deux et rester fidèle dans les deux cas.

**Jusqu'où envoyer une forme.** `range` le dit : une boule lancée à cent pieds en possède un. Sans portée, une explosion se produit là où elle est posée, sur la case de l'acteur, et un cône ou une ligne peut viser n'importe où dans la longueur qu'il dessine, car la case n'y indique que la direction. Cela vaut autant pour le `mechanics.area` d'une entrée de catalogue que pour l'aire d'une créature.

`ranged` indique le coût d'un tir au-delà de sa distance ordinaire `normal`, ou avec quelqu'un du camp adverse dans la case voisine. Chacun vaut `"disadvantage"` ou `"normal"` ; omets le bloc et aucun ne coûte quoi que ce soit. Une frappe n'est jamais un tir, donc aucune règle ne l'affecte, ni une arme de lancer utilisée dans sa propre allonge.

**Zones.** Le `mechanics.area` d'une entrée devient une vraie forme sur le plateau, visant une case plutôt qu'une personne, et `targetCount` n'y décide rien : la forme détermine combien elle atteint. Tous ceux qui occupent ces cases sont touchés, amis comme ennemis, sauf si l'entrée indique `"friendlyFire": false`.

```
burst, size 2, aimed at X        cone, size 3, aimed right      line, size 3, aimed right
. . . . .                        . . . .                        . . . .
. # # # .                        . . # .                        A # # #
. # X # .                        A # # #                        . . . .
. # # # .                        . . # .
. . . . .                        . . . .
```

Une explosion englobe toutes les cases à une distance inférieure ou égale à sa taille de la case visée. Un cône part de l'acteur vers cette case, aussi large à chaque pas qu'il est éloigné. Une ligne suit la même direction, sur une case de large. Les trois s'arrêtent devant tout élément solide.

**Ligne de vue et couverture.** Une ligne droite de cases entre les deux : tout élément solide bloque un tir et empêche une zone de s'étendre au-delà, et la cible n'apparaît simplement pas au menu. Le terrain qui offre une couverture ajoute `cover.bonus` à la défense contre laquelle l'attaque est lancée, et le journal le précise. Il n'y a ni couverture aux trois quarts, ni couverture totale, ni altitude.

**Frappes sur quelqu'un qui s'éloigne.** Déclare `opportunity.budget` : lorsqu'un combattant quitte en marchant l'allonge d'un ennemi debout, capable d'agir, disposant de ce budget et d'une arme de mêlée, le déplacement S'ARRÊTE là où il est et cet ennemi est invité à frapper ou non. Accepter dépense le budget et se résout exactement comme la même attaque pendant son propre tour ; laisser passer ne coûte rien. Dans les deux cas, la marche reprend où elle a été suspendue, en payant chaque case réellement traversée ; une frappe qui fait tomber le personnage en mouvement termine sa marche là où il tombe. Une occasion chacun pour toute la marche, quel que soit le nombre de fois où le chemin quitte la même allonge. `disengage` l'empêche pour le reste du tour, et un ensemble de règles sans `opportunity` n'a rien de tout cela.

La demande est une FENÊTRE, et elle suspend tout le combat : rien d'autre ne bouge avant la réponse de toutes les personnes interrogées. Le joueur répond à la fenêtre d'un membre du groupe, avec la frappe ou **Pass** (passer) à côté ; les autres sont gérées par celui qui les joue, un boss du Game Master par la décision du Game Master lui-même. Voir Fenêtres ci-dessous.

**Ce qu'un adversaire fait du plateau.** Un adversaire que personne ne joue évalue chaque case accessible face à chaque option qu'il pourrait y prendre, retire des points pour chaque frappe provoquée par la marche et préfère rester immobile s'il peut déjà faire de son mieux depuis sa position. Sans rien à portée, il se rapproche et sprinte d'abord si ta liste `standard` contient `dash`.

**Refus possibles.** `out-of-reach` (au-delà de la portée), `no-line-of-sight` (un obstacle solide), `unreachable` (une case que la marche ne peut payer ou sur laquelle elle ne peut finir) et `bad-cell` (une forme visant un endroit interdit).

### Ce que le combat fait de ton bloc sur le serveur

Une partie dont l'ensemble de règles déclare `combat` obtient un combat résolu par celui-ci, dans la même bataille sauvegardée qu'Engine a toujours utilisée :

- **Qui participe.** Le Game Master indique qui combat ; Engine lit les nombres de chaque membre du groupe dans sa propre fiche. Un membre sans fiche pour ton ensemble de règles est refusé par son nom plutôt que de recevoir des nombres que tu n'as pas écrits.
- **D'où viennent les nombres d'un adversaire**, dans cet ordre : la créature nommée par le Game Master dans ton bestiaire ; puis celle dont le libellé correspond au nom de l'adversaire ; puis un bloc de statistiques proposé pour ce combat par le Game Master, ajusté à ton échelle de menace ; enfin, une créature simple construite avec les nombres du palier. Chaque solution de repli et chaque ajustement est consigné en mots simples pour que le combat puisse expliquer son intervention. Un ensemble de règles sans entrée de bestiaire, proposition ni échelle de menace refuse le combat plutôt que d'en inventer un.
- **Tes fiches font foi.** Santé, réserves, états, concentration et compteurs de ta règle d'agonie sont écrits selon les propres règles de la fiche après chaque action acceptée ; un rechargement en plein combat montre donc exactement ce qu'il a laissé, sans bilan final qui pourrait le contredire.
- **Ton menu est la seule source de légalité.** Tout acteur, joueur ou adversaire, choisit un identifiant dans le même menu produit par ton bloc. Un adversaire joué par Engine y choisit avec les tactiques d'Engine ; un adversaire joué par le Game Master est invité à choisir un identifiant dans ce même menu, voit tes nombres et n'apprend jamais ce que feront les dés.
- **Tes dés.** Un combat possède sa propre graine et un curseur ; relu depuis le disque, il reprend donc avec les dés qu'il aurait lancés.

### À l'écran

Le combat se joue sur l'écran de bataille dans tes mots. Le menu contient tes attaques, tes capacités, tes confrontations et les actions standard que tu as listées, chacune indiquant ce qu'elle dépense dans tes budgets et réserves. L'ordre des tours, le round, chaque état nommé avec ses rounds restants, les points temporaires, la concentration et les deux compteurs de ta règle d'agonie sont affichés. Le journal imprime le calcul réel dans tes termes : "Juno attacks Rust jackal with Road axe: 8 (5 + 3) + 3 = 11 against Guard 6, a hit." Chaque action acceptée est écrite sur la fiche au moment où elle se produit ; un rechargement en plein combat est donc exact, et le Game Master reçoit ensuite l'instruction de ne pas changer ces nombres à nouveau.

Un combat avec positions est dessiné sur le plateau plutôt que sur la scène des portraits ; voir Positions pour les interactions du joueur. Chaque distance, sur le plateau, dans le menu et dans le journal, est exprimée dans TON unité : "Juno moves to 4, 6 for 6 paces and has 2 paces left."

### Confrontations : agripper, pousser, se libérer

Certaines actions ne sont pas une attaque contre une défense mais une confrontation : les deux camps lancent les dés, et celui qui fait mieux obtient gain de cause. Agripper quelqu'un, le renverser ou le repousser, et se libérer ont tous la même forme ; un ensemble exprime donc chacune de ces actions sous forme de données :

```json
"checks": [
  { "id": "brawn", "label": "Brawn", "value": { "abilityMod": "brawn" } },
  { "id": "wits", "label": "Wits", "value": { "abilityMod": "wits" } }
],
"contests": [
  {
    "id": "grab",
    "label": "Grab",
    "budget": "act",
    "attacker": { "checks": ["brawn"] },
    "defender": { "checks": ["brawn", "wits"] },
    "onWin": { "applies": [{ "condition": "held" }] }
  },
  {
    "id": "break_free",
    "label": "Break free",
    "budget": "act",
    "attacker": { "checks": ["brawn", "wits"] },
    "defender": { "checks": ["brawn"] },
    "from": { "holding": "held" },
    "onWin": { "ends": [{ "condition": "held", "on": "actor" }] }
  },
  {
    "id": "shove",
    "label": "Shove back",
    "budget": "act",
    "attacker": { "checks": ["brawn"] },
    "defender": { "checks": ["brawn"] },
    "ties": "attacker",
    "onWin": { "push": 4 }
  }
]
```

- **`checks`** sont les nombres que lit une confrontation, chacun une valeur de la fiche, lue une fois au début du combat comme une défense ou une sauvegarde. Une créature écrite en nombres simples donne les siens (`"checks": { "brawn": 2 }`) ; une créature dotée d'une fiche les lit sur celle-ci. Jusqu'à douze.
- **Le jet.** Les deux camps lancent tes `attackRoll.dice` et ajoutent le meilleur des tests qu'ils peuvent utiliser ici (`attacker.checks`, `defender.checks`). Le total le plus élevé gagne ; une égalité profite au défenseur, sauf si `ties` indique `"attacker"`. Le journal indique les deux camps : "Juno tries Grab on Ash-hound: 12 (6 + 6) + 3 = 15 with Brawn against 2 (1 + 1) + 2 = 4 with Brawn, and wins."
- **Gagner** fait ce que dit `onWin`, au moins une chose :
  - `applies` inflige des états au perdant avec le gagnant comme auteur ; un état que tu marques `endsWhenSourceDown` prend donc fin quand celui qui maintient la prise tombe. `rounds` lui donne une horloge ; sans lui, il dure jusqu'à ce que quelque chose y mette fin.
  - `ends` retire des états à l'acteur ou à la cible.
  - `push` éloigne le perdant en ligne droite du gagnant, de cette distance dans ton unité, en s'arrêtant avant tout obstacle solide, toute personne debout, le bord du plateau et un coin trop étroit pour s'y faufiler. C'est un déplacement forcé : il ne dépense rien de son mouvement et personne ne le frappe pour autant. Un combat sans plateau ne déplace personne.
    Perdre ne fait rien : c'est ce qu'est une prise ratée.
- **`from: { "holding": "held" }`** fait qu'une confrontation ne vise que celui qui a infligé cet état à l'acteur, et ne la place au menu que tant que l'état dure : c'est ainsi que s'exprime se libérer.
- **`reach`**, dans ton unité de distance, est sa portée ; la case voisine si tu l'omets. `reach` et `push` exigent `combat.distance`.
- **`strike: true`** lui permet de remplacer une attaque quand une action en achète plusieurs, comme le fait une attaque : prise en premier, elle dépense le budget et laisse les autres attaques disponibles ; prise avec des attaques disponibles, elle en coûte une. La référence 5e gère ainsi la lutte et la bousculade.
- **Dans le menu**, une confrontation a son propre groupe et indique sa chance de gagner. Un adversaire joué par le moteur l'évalue comme le reste, mais avec mesure : une prise ou une poussée prépare quelque chose, et se libérer vaut le plus, seulement quand on est tenu. Un Game Master qui joue un adversaire en choisit une dans le même menu.
- Un adversaire inventé par un Game Master voit ses tests limités à son palier, comme sa chance de toucher.

Les confrontations relèvent de Capability API 1.43 pour un ensemble en package.

<a id="windows-holding-the-fight-open"></a>

### Fenêtres : maintenir le combat ouvert

Certains moments appartiennent à quelqu'un d'autre que l'acteur en cours. Engine maintient le combat ouvert pour cette personne plutôt que de décider pour elle ; cette pause est une fenêtre.

Six événements en ouvrent une, dont deux proviennent de ce que tu as déjà déclaré :

- **Quelqu'un s'éloigne.** Une marche qui quitte l'allonge d'un ennemi capable de frapper s'arrête à cette étape et l'interroge. Voir Frappes sur quelqu'un qui s'éloigne, ci-dessus.
- **Entre deux tours.** À la fin d'un tour, chaque adversaire possédant des `signaturePoints` et pouvant payer l'une de ses propres actions `signature` est invité à en acheter une, avant le début du tour suivant. C'est le seul moment où elles s'achètent : une action distinctive ne figure sur aucun menu de tour, pas même le sien.
- **Quelqu'un utilise quelque chose.** Avant sa résolution, toutes les personnes de l'AUTRE camp qui possèdent une entrée attendant ce moment sont interrogées, quelle que soit la cible, tant que cette entrée atteint celui qui l'utilise. Une action standard et une confrontation n'ouvrent pas ce moment.
- **Quelque chose vise quelqu'un.** Avant sa résolution, toutes les cibles de l'AUTRE camp possédant une entrée qui attend ce moment sont interrogées. Un ami qui te soigne n'est pas une menace à laquelle répondre ; son action n'ouvre donc aucune fenêtre.
- **Un jet d'attaque touche quelqu'un.** Avant ses dégâts, la personne touchée est interrogée si elle possède une entrée qui attend ce moment. Ce qu'elle choisit compte pour cette attaque : voir Le moment après un coup au but, ci-dessous.
- **Quelque chose a blessé quelqu'un.** Après sa résolution, toutes les personnes blessées possédant une entrée qui attend CE moment sont interrogées, quel qu'en soit l'auteur. Être blessé est un fait qui te concerne ; une entrée renvoyée vers le responsable ne peut toujours pas viser un ami.

Les quatre dernières sont celles qu'une entrée de catalogue, ou l'action propre d'une créature, demande en nommant le moment qu'elle attend. Quand une même action ouvre à la fois `used` et `aimed`, l'utilisation est interrogée d'abord ; si personne ne l'annule, ceux qu'elle vise sont interrogés ensuite, et elle se résout une fois que les deux ont répondu. Être touché est interrogé à chaque jet qui touche, et être blessé une fois l'action entière terminée.

Ce que fait une fenêtre, quelle que soit son origine :

- **Rien d'autre ne bouge tant qu'elle est ouverte.** Ni l'acteur dont c'est le tour, ni la fin de ce tour, ni une autre fenêtre. Le combat attend.
- **Elle interroge une personne à la fois**, dans l'ordre des tours, une seule fois chacune. Passer est toujours une réponse et ne coûte rien. Une personne interrogée qui ne peut rien exécuter est ignorée plutôt qu'interrogée.
- **Elle reprend exactement où elle s'était arrêtée.** Une marche termine les cases restantes, en payant chacune de celles réellement traversées.
- **Celui qui joue répond.** La fenêtre de ton membre du groupe t'appartient, avec l'option et **Pass** (passer) à côté dans le menu ; celle d'un adversaire est gérée par celui qui le joue, et un boss du Game Master est interrogé par l'intermédiaire du Game Master, avec laisser passer ce moment parmi ses réponses.
- **Elle est sauvegardée avec le combat.** Une partie fermée au milieu d'une marche revient avec les mêmes personnes à interroger et les mêmes cases à parcourir.

Tu ne déclares rien pour les deux premières : un ensemble de règles avec `opportunity.budget` obtient l'une, un bestiaire avec `signaturePoints` obtient l'autre, et un ensemble de règles sans aucun des deux ne les voit jamais.

**Indiquer le moment qu'une entrée attend.** Écris `mechanics.reaction` comme un objet plutôt que `true` :

```json
"reaction": { "on": "aimed", "at": "source", "cancels": true }
```

- `on` vaut `used`, `aimed`, `hit` ou `harmed` et place l'entrée dans le menu de cette fenêtre. Ce sont les quatre seuls moments qu'Engine surveille. Une entrée qui indique encore `"reaction": true` dit seulement qu'elle ne s'exécute pas pendant un tour ; cela ne suffit pas à la proposer quelque part, elle ne figure donc sur aucun menu.
- `at` vaut `source` (par défaut) ou `chosen`. `source` dirige l'action vers le responsable du moment et remplit la cible, donc personne n'est invité à choisir ; `chosen` conserve les propres cibles de l'entrée et demande un choix.
- `cancels` empêche entièrement ce que la fenêtre retenait de se produire. Seule une entrée `used` ou `aimed` peut le déclarer : un moment déjà survenu ne peut être annulé, et un coup au but a déjà été lancé.
- `against` restreint ce à quoi l'entrée répond aux éléments utilisés depuis certains catalogues : `"against": { "catalogs": ["spells"] }` répond à un sort et à rien d'autre. Une action sans entrée derrière elle (une arme d'une liste, l'action propre d'une créature) ne vient d'aucun catalogue ; elle n'ouvre donc jamais le moment pour une entrée qui nomme des catalogues. Omets `against` et l'entrée répond à tout.

Donne-lui aussi un `budget`, sinon elle dépense celui par défaut de la liste. Une réaction dépense presque toujours un budget propre, ce qui empêche un tour d'en contenir plusieurs.

**Son coût est dépensé avant d'interroger qui que ce soit.** Une action annulée est empêchée de se produire, pas d'avoir été achetée : budget et réserves sont déjà dépensés. Si ton système les rembourse, il ne peut pas encore le dire.

Un contre s'écrit ainsi, et sur un plateau il ne répond qu'à quelqu'un situé dans sa propre `range` :

```json
"mechanics": {
  "kind": "utility",
  "range": 8,
  "free": true,
  "cost": [{ "pool": "luck", "amount": 1 }],
  "reaction": { "on": "used", "cancels": true, "against": { "catalogs": ["knacks"] } }
}
```

**Le moment après un coup au but.** Une entrée sur `hit` est la manière d'exprimer Shield, une parade (Parry) et Uncanny Dodge. L'attaque est retenue après son jet, et la personne touchée est interrogée avant que des dégâts soient infligés. Ce qu'elle choisit compte pour cette attaque : son jet n'est pas refait mais vérifié à nouveau, contre sa défense telle qu'elle est maintenant ; un état qui ajoute à la défense peut donc transformer le coup au but en échec, et une face naturelle qui touche toujours touche encore. Un état qui dure une attaque (`endsAfter: "attacked"`) infligé en réponse couvre cette attaque, dégâts compris, et s'épuise avec elle. C'est ainsi que s'exprime "réduire de moitié les dégâts de cette attaque" : `resist-all` pour une seule attaque.

```json
"mechanics": {
  "kind": "buff",
  "targets": "self",
  "budget": "reaction",
  "reaction": { "on": "hit" },
  "applies": [{ "condition": "shielded", "duration": { "rounds": 1, "at": "turn-start" } }]
}
```

L'attaque reprend exactement où elle a été retenue : le reste de ses cibles et, pour une action de créature faite d'autres actions, le reste de ses parties. Chaque jet qui touche quelqu'un possédant une telle réponse est retenu à son tour, et un combat sauvegardé pendant que la fenêtre est ouverte revient de la même manière. Une attaque faite à l'intérieur d'une fenêtre (une frappe sur quelqu'un qui s'éloigne, une action distinctive, une réponse) n'est jamais retenue, car rien de ce qui s'ouvre à l'intérieur d'une fenêtre n'en ouvre une autre. Le journal montre le jet et ce contre quoi il a été fait à l'ouverture de la fenêtre et, quand la réponse a changé la défense, s'il rate désormais. Les adversaires joués par le moteur ne prennent une garde que lorsqu'elle transforme le coup au but en échec.

Un package qui nomme un moment nécessite Capability API 1.33, et un package qui utilise `used` ou `against` nécessite 1.44. Un package qui attend `hit` nécessite 1.46.

### Pas encore

Dit clairement, car un ensemble de règles ne doit pas promettre ce qu'Engine ne fait pas :

- **Au-delà du plateau modeste** : ni couverture aux trois quarts ou totale, ni altitude, ni vol au-dessus des obstacles, ni passage serré, ni montures, ni dissimulation ou surprise, et rien ne déplace personne sinon sa propre marche et la poussée d'une confrontation. Personne ne traîne une créature qu'il tient.
- **Une confrontation est simple.** Elle n'a pas de limites de taille et n'ouvre aucune fenêtre (personne ne peut y répondre).
- **Une entrée ne peut attendre que quatre moments**, `used`, `aimed`, `hit` et `harmed` (voir Fenêtres, ci-dessus). Ce sont les moments qu'Engine remarque pour une entrée ; les deux autres fenêtres, quelqu'un qui s'éloigne et la pause entre deux tours, sont ouvertes par le combat lui-même et ne sont pas des moments qu'une entrée peut demander. Il n'existe pas de moment pour un jet de sauvegarde, une mort, un début de tour ou une chute.
- **Un contre arrête purement et simplement ce à quoi il répond.** Il ne peut pas distinguer une entrée d'un catalogue d'une autre, et il n'existe aucun test pour arrêter quelque chose de plus grand que lui.
- **Pas d'enchaînement.** Le combat conserve une fenêtre plutôt qu'une pile, donc rien ouvert à l'intérieur d'une fenêtre n'en ouvre une autre : un contre ne peut pas être contré à son tour, ce qu'inflige une réaction n'ouvre aucun autre moment, et une attaque faite à l'intérieur d'une fenêtre n'est pas retenue quand elle touche.
- **Seul un jet d'attaque est un coup au but.** Ce qui atteint sa cible sans jet (des fléchettes qui touchent d'office, une zone contre laquelle chacun fait une sauvegarde) n'ouvre aucun moment `hit`, et une réponse ne peut pas distinguer un coup porté d'un tir ; une parade que les règles réservent aux épées pare donc aussi les flèches.
- **Les dégâts se modifient en les réduisant de moitié, pas d'un montant.** Une réponse peut réduire de moitié les dégâts d'une attaque (`resist-all` pour cette attaque) ; elle ne peut pas en retirer un nombre lancé.
- **Rien n'est remboursé.** Le coût d'une action annulée reste dépensé.
- **Ce qu'un état modifie est une liste fermée.** La défense, les jets d'attaque, les sauvegardes, les tests de confrontation et la vitesse (et il en va de même pour un niveau et pour ce que fait un objet en combat), et rien d'autre : aucun bonus aux dégâts qu'inflige son porteur, et aucun niveau qui abaisse le maximum qu'un personnage peut avoir ou le met hors de combat.
- **Une créature écrite en nombres simples n'a pas de piste de blessures.** Dans un ensemble de règles dont la santé est une piste, cette créature perd toujours des points ; donne-lui une `sheet` et les coups qu'elle reçoit marqueront des cases, d'abord atténués par ses propres `resist`, `vulnerable` et `immune`.
- **Un effet supplémentaire se déclenche seul.** `on` n'a qu'une valeur, `hit` : le premier coup au but admissible de la période l'applique, sans moment où l'on te demande si tu souhaites en dépenser un.
- **Un nombre que les attaques déplacent est simple.** Un coup qui dépense lance le nombre et rien d'autre : aucune arme ne le modifie, son `floor` compris (une cible robuste absorbe un coup qui prend, et sa dureté arrête un coup qui dépense, mais rien d'autre ne réduit l'un ou l'autre) ; un effondrement prend fin après un nombre fixe de tours, quelle que soit sa profondeur ; et une attaque faite dans une fenêtre (une frappe sur quelqu'un qui s'éloigne, une réaction, une action distinctive) se fait dans le premier style.
- **Un adversaire inventé n'absorbe rien et n'a pas de dureté.** Une créature que le Game Master invente pour un combat est limitée à ton échelle de menace, qui ne dit rien de l'absorption ni de la dureté ; elle n'a donc ni l'une ni l'autre.

## Couches : variantes de ton propre ensemble de règles

Une couche est une variante nommée de ton ensemble de règles, activée par le joueur à la création d'une partie : magie rare, hiver rude, difficulté plus âpre. Les couches résident dans le fichier de l'ensemble de règles, dans un tableau facultatif `layers` ; elles voyagent donc avec lui et ne peuvent jamais manquer à une partie qui les a utilisées. L'assistant les affiche comme des interrupteurs sous ton ensemble de règles, et le choix reste fixé pour toute la durée de vie de cette partie, exactement comme l'ensemble lui-même.

```json
"layers": [
  {
    "id": "hard_winter",
    "label": "Hard winter",
    "summary": "Cold, hunger and short days. Everything is harder.",
    "conflicts": ["mud_season"],
    "gm": {
      "guidance": "Hard winter is on. Let a failed check cost warmth, food or daylight as well as progress.",
      "worldGuidance": "Hard winter is on. Build a world of closed roads, thin stores and rationed settlements."
    },
    "fields": [{ "id": "calling", "removeValues": ["Sailor"], "default": "Hauler" }],
    "difficultyLadder": [{ "label": "Easy", "dc": 7 }],
    "catalogs": [{ "id": "knacks", "hide": { "filter": "grit", "above": 0 } }]
  },
  {
    "id": "mud_season",
    "label": "Mud season",
    "summary": "Thaw, flooded roads and slow going."
  }
]
```

**Ce qu'une couche peut faire.** La liste est fermée, et chaque effet restreint quelque chose ou ajoute du texte :

- `gm.guidance` est ajouté à la fin de ton `gm.checkGuidance`, après ton propre texte et celui des couches précédentes. `gm.worldGuidance` est ajouté à `gm.worldGuidance` de la même manière.
- `fields` retire des valeurs d'un champ **énuméré**. `removeValues` nomme des valeurs déjà présentes ; au moins une doit subsister, et si le `default` du champ est retiré, la couche nomme à la place un `default` qui subsiste.
- `difficultyLadder` remplace ton échelle par une autre, dans la structure de ton propre type de résolution : `{label, dc}` pour `dice-sum` et `{label, successes, target?}` pour `dice-pool`. Elle subit exactement les mêmes vérifications que ton échelle. Lorsque plusieurs couches actives en déclarent une, la dernière l'emporte.
- `catalogs` masque des entrées du sélecteur de l'éditeur de fiches. Chaque règle nomme l'un des `filters` déclarés par ce catalogue et exactement une comparaison : `above` ou `below` pour un filtre `number`, `equals` ou `notIn` pour un filtre `text` ou `tags`. Une entrée qui ne définit pas ce filtre n'est jamais masquée.
- `currencies` retire des pièces : `removeUnits` nomme des pièces isolées et `removeFamilies` des familles entières. La plus petite pièce d'une famille ne part qu'avec sa famille ; chaque famille restante peut donc toujours payer et rendre la monnaie. Tant que la couche est active, personne ne gagne, ne paie avec, ne choisit ni ne lâche une pièce retirée, et la ligne de la bourse l'omet. Un prix exprimé dans cette pièce est donné dans la plus grosse pièce restante qui le paie exactement, à valeur égale : la longue nuit de Gravewatch retire la couronne ; un pistolet de guet à 3 couronnes coûte donc 15 shillings. Un objet dont toute la famille a disparu n'a pas de prix.

**Ce qu'une couche ne peut pas faire.** Elle ne peut ajouter une valeur énumérée, un champ, une compétence, une réserve ou un repos, changer le type de résolution, toucher l'état en cours ou les nombres de combat, ni ajouter un appel de modèle. Une valeur qu'une couche _ajouterait_ serait inconnue de tous les autres lecteurs de la fiche ; les valeurs ne peuvent donc que disparaître. Tout ce qui dépasse cette liste constitue une modification de l'ensemble de règles lui-même ou un second ensemble.

**Conflits.** `conflicts` nomme les couches qui ne peuvent être actives ensemble. Nommer un seul côté de la paire suffit. L'assistant désactive l'autre interrupteur, et si un choix sauvegardé contient malgré tout les deux, la couche déclarée **plus tard** est retirée ; les deux mêmes choix donnent donc toujours les mêmes règles.

**Une fiche contenant déjà une valeur retirée la conserve.** Rien ne réécrit un personnage. L'éditeur cesse simplement de proposer la valeur, et un personnage qui la possédait déjà l'affiche telle quelle. Désactive la couche dans une nouvelle partie et la valeur sera proposée à nouveau. Il en va de même pour une entrée de catalogue masquée : elle disparaît du sélecteur, et une ligne déjà choisie par le joueur reste sur la fiche. Les pièces qu'une couche retire et que quelqu'un transporte déjà restent dans son sac.

**Limites.** 12 couches par ensemble de règles et 4000 caractères de consignes par couche, les deux chaînes comptées ensemble. Un ensemble de règles en package qui déclare `layers`, ou un `gm.worldGuidance` de base, nécessite Capability API 1.25, et un ensemble dont les couches retirent des pièces nécessite 1.64. Un ensemble que tu importes est validé par Engine à la lecture et n'a donc besoin de rien.

**Les couches écrites par quelqu'un d'autre** (une couche de magie rare pour un ensemble de règles que tu n'as pas écrit, livrée dans son propre fichier) viendront plus tard. Aujourd'hui, une couche est livrée à l'intérieur de l'ensemble de règles auquel elle appartient.

<a id="trying-your-ruleset"></a>

## Essayer ton ensemble de règles

Les ensembles de règles communautaires utilisent le même interrupteur que les agents importés. Ouvre **Settings** (réglages) > **Advanced** (avancé) > **Danger Zone** (zone de danger) et vérifie que **Allow custom Agent imports** (autoriser l'importation d'agents personnalisés) est activé. L'importation nécessite aussi un accès localhost ou **Admin Access** (accès administrateur) configuré.

1. Ouvre le panneau **Agents** (agents) et choisis le bouton **Import agents** (importer des agents), l'icône de téléchargement dans la rangée de boutons en haut du panneau.
2. Choisis **Game Mode ruleset** (ensemble de règles Game Mode) et sélectionne ton fichier JSON.
3. Lis l'examen. Il montre le nom, la version, la licence, ce que couvre l'ensemble de règles et le texte du Game Master. Choisis **Import** (importer).

Ton ensemble apparaît dans la section **Rules** (règles) du panneau et dans le choix **Rules** de l'assistant de configuration des nouvelles parties. Un ensemble importé depuis un fichier est classé sous `local/<your id>`, pour ne jamais être confondu avec un ensemble officiel ou celui de quelqu'un d'autre.

### Modifier un ensemble de règles déjà importé

Une version importée n'est jamais réécrite. Si tu modifies le fichier et le réimportes avec la même `version`, l'importation est refusée et te demande d'augmenter le numéro. C'est volontaire : une partie est liée à la version exacte utilisée à sa création, pour qu'une campagne en cours ne se réveille jamais avec des calculs différents.

Pendant la rédaction, le cycle est donc : modifier, augmenter `version`, importer, commencer une nouvelle partie. Les anciennes versions restent installées à côté de la nouvelle jusqu'à ce que tu retires l'ensemble de règles de la section **Rules**. Retirer un ensemble encore utilisé par une partie fait signaler à celle-ci que son ensemble de règles manque, jusqu'à sa réimportation.

Si tu changes la structure de la fiche (ajout, suppression ou renommage d'éléments), augmente aussi `sheet.version`. Les fiches existantes sont lues avec tolérance : les valeurs inconnues de la nouvelle fiche sont conservées, et les valeurs manquantes prennent leur valeur par défaut.

## Partager ton ensemble de règles

**Sous forme de fichier.** Envoie le fichier JSON à un ami. Il l'importe comme toi.

**Depuis un dépôt GitHub.** Si tu conserves ton travail dans un dépôt GitHub public, place chaque ensemble de règles dans un dossier `rulesets` à la racine, un fichier par ensemble :

```text
your-repository/
  agents.json        (optional, only if you also share agents)
  rulesets/
    ember-roads.json
    another-system.json
```

Un utilisateur ajoute ton dépôt une seule fois via la liste des dépôts d'agents personnalisés, examine son contenu et peut le synchroniser plus tard pour recevoir les nouvelles versions. Cette liste est une fonction avancée que la personne qui gère le serveur doit activer avec `ENABLE_CUSTOM_AGENT_REPOS=true`. Les ensembles de règles d'un dépôt sont classés sous le nom de son propriétaire, comme `alice/ember-roads`, pour que deux auteurs puissent chacun publier un ensemble nommé `v20` sans conflit.

Deux limites s'appliquent. Un dépôt peut contenir au maximum 32 fichiers JSON directement dans `rulesets` ; au-delà, il est refusé. Un compte nommé `local` ne peut publier d'ensembles de règles, car `local/` est réservé à ceux importés depuis un fichier.

**Dans le catalogue officiel.** Un système largement joué, dont les licences sont claires, peut être proposé à tous via **Download Agents** (télécharger des agents). Cela passe par une pull request au dépôt [Marinara-Agents](https://github.com/Pasta-Devs/Marinara-Agents). Consulte le package `ruleset-5e-2014` pour sa structure.

## Licences

Ne publie que du texte de règles que tu as le droit de partager. Beaucoup de systèmes publient un document de référence sous licence ouverte ; c'est ce document que tu peux copier. Place l'identifiant de licence et le texte d'attribution exigé sous `license`. Ne copie pas le texte de livres de règles sans licence ouverte. Un ensemble de règles a surtout besoin de noms et de nombres, et le texte du Game Master doit être rédigé dans tes propres mots.

## Dépannage

- **L'importation dit qu'un nom n'existe pas.** Quelque chose dans le fichier pointe vers un identifiant non déclaré, comme une compétence nommant une caractéristique supprimée. Le message donne le chemin vers la ligne.
- **L'importation dit qu'une version est déjà installée avec un contenu différent.** Augmente `version` et réimporte.
- **Mon ensemble de règles n'apparaît pas dans l'assistant de configuration.** Vérifie que **Allow custom Agent imports** est activé. Lorsqu'il est désactivé, les ensembles importés sont exclus des nouvelles parties. Les parties qui en utilisent déjà un continuent de fonctionner.
- **Une partie dit que son ensemble de règles manque.** La version exacte utilisée à sa création n'est pas installée. Réimporte cette version du fichier.
