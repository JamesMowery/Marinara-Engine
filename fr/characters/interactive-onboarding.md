# Accueil interactif

Ce guide présente l'**accueil interactif** (interactive onboarding), une fonction des fiches de personnage qui permet aux joueurs de construire leur persona en répondant aux questions de l'auteur de la fiche quand ils commencent un chat Roleplay. Il explique comment les auteurs le configurent dans le Character Editor (éditeur de personnage) et ce que les joueurs voient dans l'assistant de nouveau chat.

## À quoi sert l'accueil interactif

Un persona est le personnage qui te représente dans un chat. Voir [Personas utilisateur](personas.md). D'habitude, tu écris toi-même un persona avant de jouer. Avec l'accueil interactif, une fiche de personnage fournit à la place un modèle de persona. Ce modèle, ce sont les cinq champs du persona – **Description**, **Personality** (personnalité), **Backstory** (passé), **Appearance** (apparence) et **Scenario** (scénario) – avec des blancs que le joueur remplit, plus les questions qui servent à les remplir.

Quand un joueur commence un chat Roleplay avec la fiche, il peut choisir **Create your persona** (créer ton persona), répondre aux questions et obtenir un nouveau persona ordinaire. Les réponses sont inscrites une seule fois dans le texte. Le résultat est un persona normal, que le joueur peut modifier, garder ou supprimer comme n'importe quel autre. La fiche elle-même ne change pas.

L'accueil interactif utilise la syntaxe de macro existante. Il n'exécute aucun script.

## Le configurer en tant qu'auteur

1. Ouvre le personnage dans le Character Editor. Voir [Créer et modifier des personnages](creating-and-editing-characters.md).
2. Ouvre l'onglet **Onboarding** (accueil), à côté de **Card**.
3. Active **Interactive onboarding** (accueil interactif). Un court guide **How onboarding works** (fonctionnement de l'accueil) apparaît sous l'interrupteur.

Désactiver l'interrupteur masque les champs de l'accueil interactif, mais conserve ce que tu as écrit.

### Ajouter des questions

Sous **Onboarding Questions** (questions d'accueil), clique sur **Add Question** (ajouter une question). Chaque question fonctionne comme une variable de preset. Voir [Presets](../prompts/presets.md). Une fiche peut compter jusqu'à 100 questions.

- **Variable Name** (nom de variable) est le nom que tu utilises dans les champs, par exemple `faction`. Choisis n'importe quel nom fait de lettres, de chiffres et de tirets bas, qui commence par une lettre ou un tiret bas. Les noms sont sensibles à la casse : `Gender` et `gender` sont différents.
- **Question (shown to user)** est le texte que lit le joueur. Il peut reprendre des réponses précédentes, par exemple `What does {{player}} look like?`. Ici, `{{char}}` est le nom de la fiche et `{{user}}` celui du joueur.
- Avec **Options**, la question devient un choix. La première option est sélectionnée par défaut. Sans option, le joueur saisit une réponse libre.
- **Value** (valeur) est ce qui entre dans le persona. Les majuscules et les minuscules sont conservées telles que tu les tapes ; les macros de ce champ sont remplacées comme dans le reste des champs. Laisse ce champ vide pour utiliser le libellé de l'option. Saisis une valeur en minuscules, par exemple `ranger`, si la réponse s'insère au milieu d'une phrase.
- **Help text for players** (texte d'aide pour les joueurs) est un texte facultatif affiché sous l'option.
- **Allow own answer** (autoriser une réponse personnelle) ajoute un choix **Write your own** (écrire ta propre réponse) à côté de tes options.
- **Multi-Select** (sélection multiple) permet au joueur de choisir plusieurs options. Elles sont alors reliées par le séparateur.

Une question nommée `player` définit le nom du persona. Si tu n'en ajoutes pas, les joueurs doivent d'abord indiquer leur nom.

L'éditeur t'avertit quand un nom est vide, invalide, utilisé deux fois ou déjà pris par une macro intégrée, et quand une question n'est utilisée dans aucun champ.

### Rédiger les champs du persona

Écris les cinq champs comme pour un persona à toi, et place `{{name}}` là où une réponse doit apparaître :

```
Name: {{player}}
Class: {{class}}
Role: {{player}} is a {{class}} sworn to {{faction}}.
```

Quand tu cliques dans un champ, ses **Available variables** (variables disponibles) apparaissent dessous. Clique sur l'une d'elles pour l'insérer à l'emplacement de ton curseur.

Utilise `{{#if}}` pour n'afficher du texte que pour certaines réponses :

```
{{#if knowsHer == yes}}Grew up next door to {{char}}. {{relationToAna}}{{/if}}
```

Une question qui n'apparaît qu'à l'intérieur d'un `{{#if}}` n'est posée que lorsque cette condition est vraie. Dans l'exemple, `relationToAna` est une question de suivi qui apparaît dès que le joueur répond **yes**. Les comparaisons ignorent la différence entre majuscules et minuscules.

`{{user}}` et `{{char}}` restent des macros dans le persona créé : elles continuent donc de fonctionner comme dans n'importe quel persona. Les autres macros, comme `{{random}}`, `{{time}}` ou `{{getvar}}`, sont remplacées une seule fois à la création du persona, ou laissées vides si elles n'ont pas de valeur à ce moment-là.

La section **Rendered preview** (aperçu du rendu), en bas, montre le persona construit à partir de réponses d'exemple, et liste les questions posées, dans l'ordre.

## Créer un persona en tant que joueur

1. Commence un nouveau chat Roleplay. À l'étape **Persona & Characters** (persona et personnages), les fiches dotées de l'accueil interactif affichent une petite icône de personne avec un plus dans la liste des personnages.
2. Ajoute le personnage. Une ligne **Create your persona** apparaît en haut de **Your Persona** (ton persona), avec le nom de la fiche à laquelle elle appartient.
3. Sélectionne cette ligne et clique sur **Next** (suivant).
4. Une fenêtre **Create your persona** affiche les questions de la fiche. Les questions de suivi apparaissent dès que tes réponses les rendent pertinentes.
5. Clique sur **Create persona** (créer le persona). Le nouveau persona est enregistré et sélectionné pour le chat, puis l'assistant continue.

Si tu choisis un autre persona ou **None** (Stay anonymous, rester anonyme), aucune question n'est posée. **Back** (retour) dans la fenêtre des questions ramène au sélecteur de persona. Chaque fiche du chat dotée de l'accueil interactif reçoit sa propre ligne **Create your persona**.

L'accueil interactif est proposé dans l'assistant de nouveau chat Roleplay. Quick Start, la configuration Conversation et le lancement d'un chat directement depuis une fiche de personnage ne le proposent pas encore.

## Partager des fiches avec accueil interactif

L'accueil interactif est enregistré avec la fiche. Les exports **Marinara Native** et **Compatible PNG Card** le conservent : un joueur qui importe la fiche reçoit donc aussi les questions. L'export **Compatible JSON** le laisse de côté. Une fiche importée avec plus de 100 questions garde les 100 premières. Voir [Importer et exporter des fiches de personnage](import-export.md).

## Guides associés

- [Créer et modifier des personnages](creating-and-editing-characters.md)
- [Personas utilisateur : créer et modifier](personas.md)
- [Choisir ton persona dans un chat](choosing-your-persona.md)
- [Macros](../prompts/macros.md)
- [Importer et exporter des fiches de personnage](import-export.md)
