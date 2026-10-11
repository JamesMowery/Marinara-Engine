# Interaktives Onboarding

In dieser Anleitung erfährst du, wie **interaktives Onboarding** funktioniert. Mit dieser Funktion einer Charakterkarte bauen Spieler ihre Persona auf, indem sie zu Beginn eines Roleplay-Chats die Fragen des Kartenautors beantworten. Die Anleitung zeigt, wie Autoren es im Charakter-Editor einrichten und was Spieler im Einrichtungsassistenten für neue Chats sehen.

## Was es bewirkt

Eine Persona ist der Charakter, der dich in einem Chat vertritt. Siehe [Nutzer-Personas](personas.md). Normalerweise schreibst du eine Persona selbst, bevor du spielst. Mit interaktivem Onboarding bringt eine Charakterkarte stattdessen eine Persona-Vorlage mit. Die Vorlage besteht aus den fünf Persona-Feldern (**Description**, **Personality**, **Backstory**, **Appearance** und **Scenario**) mit Lücken, die der Spieler ausfüllt, und aus den Fragen, die diese Lücken füllen.

Startet ein Spieler mit der Karte einen Roleplay-Chat, kann er **Create your persona** (deine Persona erstellen) wählen, die Fragen beantworten und so eine neue, gewöhnliche Persona erhalten. Die Antworten werden einmalig in den Text geschrieben. Das Ergebnis ist eine normale Persona, die der Spieler wie jede andere bearbeiten, behalten oder löschen kann. Die Karte selbst ändert sich nicht.

Onboarding nutzt die vorhandene Makro-Syntax. Es führt keine Skripte aus.

## Als Autor einrichten

1. Öffne den Charakter im Charakter-Editor. Siehe [Charaktere anlegen und bearbeiten](creating-and-editing-characters.md).
2. Öffne den Tab **Onboarding** neben **Card**.
3. Schalte **Interactive onboarding** (interaktives Onboarding) ein. Unter dem Schalter erscheint eine kurze Anleitung **How onboarding works** (so funktioniert Onboarding).

Schaltest du den Schalter aus, werden die Onboarding-Felder ausgeblendet, dein Text bleibt aber erhalten.

### Fragen hinzufügen

Klick unter **Onboarding Questions** (Onboarding-Fragen) auf **Add Question** (Frage hinzufügen). Jede Frage funktioniert wie eine Preset-Variable. Siehe [Presets](../prompts/presets.md). Eine Karte kann bis zu 100 Fragen haben.

- **Variable Name** (Variablenname) ist der Name, den du in den Feldern verwendest, etwa `faction`. Wähle einen beliebigen Namen aus Buchstaben, Ziffern und Unterstrichen, der mit einem Buchstaben oder einem Unterstrich beginnt. Groß- und Kleinschreibung zählt: `Gender` und `gender` sind verschiedene Namen.
- **Question (shown to user)** ist das, was der Spieler liest. Die Frage kann frühere Antworten verwenden, etwa `What does {{player}} look like?`. Dabei steht `{{char}}` für den Namen der Karte und `{{user}}` für den Namen des Spielers.
- **Options** (Optionen) machen aus der Frage eine Auswahl. Die erste Option ist vorausgewählt. Ohne Optionen tippt der Spieler eine freie Antwort.
- **Value** (Wert) ist das, was in die Persona eingesetzt wird. Groß- und Kleinschreibung bleiben so, wie du sie tippst; Makros darin werden wie im Rest der Felder ausgefüllt. Lässt du das Feld leer, wird die Beschriftung der Option verwendet. Tipp einen kleingeschriebenen Wert wie `ranger`, wenn die Antwort mitten im Satz steht.
- **Help text for players** (Hilfetext für Spieler) ist optionaler Text, der unter der Option erscheint.
- **Allow own answer** (eigene Antwort erlauben) ergänzt neben deinen Optionen die Auswahl **Write your own** (eigene Antwort schreiben).
- **Multi-Select** (Mehrfachauswahl) lässt den Spieler mehrere Optionen wählen. Sie werden mit dem Trennzeichen verbunden.

Eine Frage namens `player` legt den Namen der Persona fest. Fügst du keine solche Frage hinzu, werden Spieler zuerst nach ihrem Namen gefragt.

Der Editor warnt dich, wenn ein Name leer, ungültig, doppelt vergeben oder von einem eingebauten Makro belegt ist und wenn kein Feld eine Frage verwendet.

### Die Persona-Felder schreiben

Schreib die fünf Felder wie bei einer eigenen Persona und setz `{{name}}` dort ein, wo eine Antwort hingehört:

```
Name: {{player}}
Class: {{class}}
Role: {{player}} is a {{class}} sworn to {{faction}}.
```

Klickst du in ein Feld, erscheinen darunter seine **Available variables** (verfügbare Variablen). Klick auf eine davon, um sie an der Cursorposition einzufügen.

Mit `{{#if}}` zeigst du Text nur bei bestimmten Antworten an:

```
{{#if knowsHer == yes}}Grew up next door to {{char}}. {{relationToAna}}{{/if}}
```

Eine Frage, die nur innerhalb eines `{{#if}}` vorkommt, wird nur gestellt, wenn diese Bedingung erfüllt ist. Im Beispiel ist `relationToAna` eine Anschlussfrage, die erscheint, sobald der Spieler **yes** antwortet. Vergleiche ignorieren Groß- und Kleinschreibung.

`{{user}}` und `{{char}}` bleiben in der erstellten Persona als Makros stehen und funktionieren deshalb wie in jeder anderen Persona. Andere Makros wie `{{random}}`, `{{time}}` oder `{{getvar}}` werden beim Erstellen der Persona einmalig ausgefüllt oder bleiben leer, wenn sie zu diesem Zeitpunkt keinen Wert haben.

Die **Rendered preview** (gerenderte Vorschau) ganz unten zeigt die Persona, die aus Beispielantworten entsteht, und listet die gestellten Fragen der Reihe nach auf.

## Als Spieler eine Persona erstellen

1. Starte einen neuen Roleplay-Chat. Im Schritt **Persona & Characters** (Persona und Charaktere) zeigen Karten mit Onboarding in der Charakterliste ein kleines Symbol mit Person und Pluszeichen.
2. Füge den Charakter hinzu. Oben in **Your Persona** (deine Persona) erscheint eine Zeile **Create your persona**, die die zugehörige Karte nennt.
3. Wähl diese Zeile aus und klick auf **Next** (Weiter).
4. Ein Fenster **Create your persona** zeigt die Fragen der Karte. Anschlussfragen erscheinen, sobald deine Antworten sie relevant machen.
5. Klick auf **Create persona** (Persona erstellen). Die neue Persona wird gespeichert und für den Chat ausgewählt, und der Einrichtungsassistent macht weiter.

Wählst du eine andere Persona oder bleibst du mit **None** („Stay anonymous“) anonym, werden keine Fragen gestellt. **Back** (Zurück) im Fragenfenster führt zurück zur Persona-Auswahl. Jede Karte mit Onboarding im Chat bekommt ihre eigene Zeile **Create your persona**.

Onboarding wird im Einrichtungsassistenten für neue Roleplay-Chats angeboten. Quick Start, die Einrichtung eines Conversation-Chats und der Start eines Chats direkt von einer Charakterkarte aus bieten es noch nicht an.

## Karten mit Onboarding teilen

Das Onboarding wird mit der Karte gespeichert. Exporte als **Marinara Native** und **Compatible PNG Card** behalten es, sodass ein Spieler, der die Karte importiert, die Fragen ebenfalls bekommt. Der Export als **Compatible JSON** lässt es weg. Eine importierte Karte mit mehr als 100 Fragen behält die ersten 100. Siehe [Charakterkarten importieren und exportieren](import-export.md).

## Verwandte Anleitungen

- [Charaktere anlegen und bearbeiten](creating-and-editing-characters.md)
- [Nutzer-Personas: anlegen und bearbeiten](personas.md)
- [Die Persona für einen Chat auswählen](choosing-your-persona.md)
- [Makros](../prompts/macros.md)
- [Charakterkarten importieren und exportieren](import-export.md)
