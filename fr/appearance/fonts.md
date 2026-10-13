# Polices personnalisées et Google Fonts

Ce guide explique comment changer la police que Marinara Engine utilise dans toute l'application. Tu peux garder la police intégrée, ajouter tes propres fichiers de police, ou télécharger une police depuis Google Fonts en indiquant son nom.

## Choisir la police de l'application

Le réglage de la police se trouve dans **Settings** (Paramètres), sous l'onglet **Appearance** (Apparence), dans la section **Text & Scale** (texte et échelle).

1. Ouvre **Settings** et clique sur l'onglet **Appearance**.
2. Repère la section **Text & Scale**.
3. Ouvre le menu déroulant **Font**.
4. Choisis une police dans la liste.

La valeur par défaut est **Default (Inter)**. Inter est une police sobre, retenue pour le confort de lecture à l'écran. Les polices que tu ajoutes apparaissent dans ce même menu déroulant **Font**, sous l'option par défaut.

Les champs de prompt de l'éditeur de presets et le champ Example Dialogue d'un personnage utilisent une police à chasse fixe tant que la police par défaut est sélectionnée. Choisis n'importe quelle autre police et ils l'utilisent aussi. Les autres champs de prompt, comme le modèle de prompt d'un agent, restent en chasse fixe.

Les widgets du chat ont leur propre réglage **Font**, dans **Settings > Appearance > App > Chat widget style** (Paramètres > Apparence > Application > Style des widgets du chat). Active **Apply preset font** (appliquer la police du preset) au même endroit pour utiliser aussi cette police dans les messages, les champs de saisie et les contrôles du chat. Laisse cet interrupteur désactivé pour qu'ils gardent leur police habituelle. Voir [Styles de fenêtres de chat prêts à utiliser](custom-css-themes.md#ready-made-chat-window-styles).

Le choix de police se synchronise entre les appareils. Dès que tu sélectionnes une police, tous les navigateurs et appareils connectés au même serveur Marinara l'adoptent. Pour comprendre le fonctionnement de cette synchronisation, consulte le guide [Vue d'ensemble des paramètres](../settings/settings-overview.md).

## Ajouter tes propres polices

Pour ajouter une police personnalisée, il suffit de déposer un fichier de police dans un dossier sur le serveur, c'est-à-dire la machine qui fait tourner Marinara.

1. Repère le dossier `data/fonts/` à l'intérieur du dossier de données de Marinara, sur la machine serveur.
2. Copie le fichier de police dans ce dossier.
3. Retourne dans **Settings**, puis **Appearance**, puis **Text & Scale**.
4. Ouvre le menu déroulant **Font**. La police figure désormais dans la liste.
5. Sélectionne-la.

Marinara lit ces types de fichiers de police : `.ttf`, `.otf`, `.woff` et `.woff2`. Les fichiers portant une autre extension sont ignorés.

Marinara compose un nom d'affichage à partir du nom de fichier. Par exemple, un fichier nommé `OpenSans-Bold.ttf` apparaît sous la forme "Open Sans". Nomme donc tes fichiers clairement si tu veux une liste bien rangée.

Les fichiers de police du dossier `data/fonts/` résident sur le serveur. Tous les appareils connectés au même serveur Marinara peuvent les utiliser. Le choix de police se synchronise lui aussi entre ces appareils, qui affichent donc tous la même police.

## Télécharger depuis Google Fonts

Marinara peut récupérer une police directement sur Google Fonts. Le serveur a besoin d'un accès à Internet pour cela.

1. Ouvre **Settings**, puis **Appearance**, puis **Text & Scale**.
2. Repère le champ **Google Fonts**.
3. Saisis le nom exact de la police, par exemple `Fira Code` ou `Lora`.
4. Clique sur **Add**.
5. Attends la fin du téléchargement. La nouvelle police apparaît alors dans le menu déroulant **Font**.

Saisis le nom exactement tel que Google Fonts l'orthographie. Le lien **Browse fonts at fonts.google.com** se trouve juste à côté du champ. Il ouvre le site Google Fonts dans un nouvel onglet pour que tu puisses y vérifier les noms.

Le nom accepte uniquement des lettres, des chiffres et des espaces. Si tu télécharges plus tard la même police, Marinara remplace l'ancienne copie au lieu de créer un doublon.

Si le téléchargement échoue, lis le message d'erreur. Quand Marinara n'arrive pas à joindre Google Fonts, il te demande de vérifier ta connexion Internet. Quand il indique que la police est introuvable, deux causes sont possibles. Le nom ne correspond peut-être à aucune police sur Google Fonts. Ou alors la police n'a pas de graisse regular (400), c'est-à-dire le style normal, non gras. Vérifie l'orthographe, et vérifie sur le site Google Fonts que la police propose bien un style Regular.

## Open Fonts Folder ne fonctionne qu'en local

À côté du menu déroulant **Font** se trouve un bouton **Open Fonts Folder** (ouvrir le dossier des polices). Il ouvre le dossier `data/fonts/` dans l'explorateur de fichiers de la machine serveur.

Ce bouton agit sur le serveur, pas sur l'appareil depuis lequel tu consultes Marinara. Si tu fais tourner Marinara sur ton propre ordinateur, il t'ouvre bien le dossier. Si tu te connectes depuis un téléphone ou un second ordinateur, le bouton ne te sert à rien. Dans ce cas, copie toi-même les fichiers de police dans le dossier `data/fonts/` du serveur.

## Guides associés

- [Paramètres d'apparence](appearance-settings.md)
