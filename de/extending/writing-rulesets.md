# Regelsätze für Game Mode schreiben

Ein Regelsatz erklärt Game Mode, wie ein Tischrollenspiel funktioniert: welche Würfel eine Probe wirft, was auf dem Charakterbogen steht, welche Ressourcen verbraucht werden und was eine Rast wiederherstellt. Dieser Leitfaden richtet sich an alle, die eigene Regelsätze schreiben und teilen möchten. Wenn du mit einem fremden Regelsatz spielen möchtest, beginne bei [Regeln auswählen](../game/getting-started.md#choosing-rules).

Ein Regelsatz besteht aus einer JSON-Datei. Sie enthält Daten, keinen Code. Nichts darin wird ausgeführt; der Import kann daher keine Aktionen auf deinem Computer ausführen. Lies vor dem Import einer fremden Datei vor allem den Text für den Game Master sorgfältig: Er wird in jedem Spiel mit diesem Regelsatz an das Modell gesendet.

## Zuerst lesen: Was ein Regelsatz kann und was nicht

Ein Regelsatz kann nur eine Mechanik ausgestalten, die die Engine bereits kennt. Derzeit kennt die Engine zwei Arten, eine Probe aufzulösen. Deine Datei wählt eine davon mit `resolution.kind`:

- **`dice-sum`**: Würfel werfen, Zahlen vom Bogen addieren und einen Schwierigkeitswert erreichen oder übertreffen. Das deckt d20-Systeme, Systeme mit 2d6 plus Attribut und viele weitere ab.
- **`dice-pool`**: Die eigene Würfelanzahl des Charakters werfen und die Würfel zählen, die einen Zielwert erreichen. Das deckt Systeme ab, in denen ein Wert für eine Handvoll Würfel statt für einen Bonus steht.

Beide werden unter [Auflösungsarten](#resolution-kinds) vollständig beschrieben.

Eine Mechanik, die in keine dieser Formen passt, lässt sich nicht in einer Regelsatzdatei ausdrücken. Beispiele sind der höchste Würfel eines Pools, Prozentproben mit Unterwürfeln, Symbolwürfel und vergleichende Pools. Jede dieser Mechaniken braucht eine neue Auflösungsart in der Engine, also einen Codebeitrag mit Tests statt einer JSON-Datei. Wenn dein System so etwas benötigt, öffne einen Funktionswunsch im Engine-Repository und beschreibe die Mechanik anhand einiger durchgerechneter Würfe. Diese Beispiele werden zu den Tests.

Game Mode kann einen Kampf nach den eigenen Kampfregeln von Marinara oder nach deinem Regelsatz auflösen. Ein optionaler `battle`-Block stellt Marinaras Kampfsystem die Zahlen deiner Charakterbögen bereit; siehe [Kämpfe](#battles-lending-the-sheet-to-marinaras-combat). Ein optionaler `combat`-Block legt dagegen fest, wie der Regelsatz den Kampf auflöst; siehe [Kampfregeln](#combat-a-fight-your-own-rules-resolve). Die Engine spielt diese Regeln bereits aus. **Combat Preference** (Kampfpräferenz) wählt die Darstellung **Classic** oder, wenn der Regelsatz Entfernungen definiert, ein Schlachtfeld in **Tactical**.

## Schnellstart

1. Kopiere die Beispieldatei, deren Würfelmechanik zu deinem System passt. [`ember-roads.json`](https://github.com/Pasta-Devs/Marinara-Engine/blob/staging/docs/examples/rulesets/ember-roads.json) ist ein kleines 2d6-System mit drei Werten. Es zeigt, dass das Format weder einen d20 noch sechs Attribute voraussetzt. [`gravewatch.json`](https://github.com/Pasta-Devs/Marinara-Engine/blob/staging/docs/examples/rulesets/gravewatch.json) ist ein kleiner Pool mit zehnseitigen Würfeln, drei Grundwerten und sechs Tätigkeiten. Ein vollständiges Beispiel ist die Datei für 5e (SRD 5.1) in [`ruleset-5e-2014.example.json`](https://github.com/Pasta-Devs/Marinara-Engine/blob/staging/docs/development/ruleset-5e-2014.example.json).
2. Ersetze `id` durch deine eigene ID. Sie besteht aus Kleinbuchstaben, Ziffern und einzelnen Bindestrichen, etwa `ember-roads`.
3. Bearbeite den Bogen, die Rasten und den Text für den Game Master.
4. Importiere die Datei; siehe [Deinen Regelsatz ausprobieren](#trying-your-ruleset). Der Import prüft die gesamte Datei und nennt dir Zeile für Zeile die Fehler, bevor etwas gespeichert wird.
5. Erstelle ein neues Spiel, wähle deinen Regelsatz unter **Rules** (Regeln) und spiele einige Proben.

Damit dein Editor dir beim Schreiben hilft, verweise auf das JSON-Schema. Füge dazu folgende Zeile als erste Zeile innerhalb der äußeren geschweiften Klammern der Datei ein:

```json
"$schema": "https://raw.githubusercontent.com/Pasta-Devs/Marinara-Engine/staging/docs/extending/ruleset.schema.json",
```

Das Schema erkennt beim Schreiben falsch geschriebene Schlüssel und falsche Typen. Es kann nicht prüfen, ob die Namen in deiner Datei auf vorhandene Dinge verweisen, etwa ob eine Fertigkeit ein existierendes Attribut nennt. Das prüft der Import.

Du kannst in jedem Objekt der Datei eine Zeile `"$comment": "..."` als Notiz ergänzen. Die Engine ignoriert sie.

## Die Bestandteile der Datei

| Schlüssel | Inhalt |
| --- | --- |
| `schemaVersion` | Immer `1`. |
| `id`, `version` | Der Name deines Regelsatzes für die Engine und eine ganze Zahl, die du bei jeder veröffentlichten Änderung erhöhst. |
| `name` | Der im Einrichtungsassistenten sichtbare Name. |
| `edition` | Optional. Eine Zeile zur Ausgabe oder zum Entwurf. |
| `license` | Optional. Eine SPDX-ID und der von deiner Quelle verlangte Urheberhinweis. |
| `coverage` | Was der Regelsatz abdeckt, mit einer einzeiligen Zusammenfassung für den Einrichtungsassistenten. |
| `resolution` | Wie eine Probe oder ein Rettungswurf gewürfelt wird. |
| `sheet` | Alles auf dem Charakterbogen. |
| `rests` | Was jede Art von Rast wiederherstellt und entfernt. |
| `gm` | Der Text für das Game-Master-Modell und die Bogenwerte, die es für jeden Charakter sieht. |
| `catalogs` | Optional. Fertige Einträge im Bogeneditor, damit Spieler lange Listen nicht von Hand eingeben müssen. |
| `items` | Optional. Die Begriffe, in denen deine Gegenstände geschrieben sind: Kategorien, Seltenheiten, Werte, Trageplätze, Geld. |
| `battle` | Optional. Was ein Kampf vom Bogen lesen und danach zurückschreiben darf. |
| `combat` | Optional. Wie deine eigenen Regeln einen Kampf auflösen und was der Kampfbildschirm anschließend ausspielt. |
| `layers` | Optional. Varianten deines Regelsatzes, die ein Spieler bei der Spielerstellung einschaltet. |

Die Datei darf bis zu 256 KB groß sein. Text, der in einem Prompt landet – Namen, Beschriftungen und Game-Master-Text –, darf keine Zeilenumbrüche, eckigen Klammern oder doppelten geschweiften Klammern enthalten.

IDs innerhalb des Bogens, etwa für Attribute, Fertigkeiten, Felder und Pools, bestehen aus Kleinbuchstaben, Ziffern und Unterstrichen und beginnen mit einem Buchstaben, zum Beispiel `grit_max`.

<a id="resolution-kinds"></a>

### Auflösungsarten

`resolution.kind` wählt, wie eine Probe gewürfelt wird. Beide Arten lesen denselben Charakterbogen und teilen drei Schlüssel. Die Bestandteile unterhalb von `resolution` ändern sich beim Wechsel deshalb nicht:

- `abilityModifier`: Wie ein Bogenwert zu einer Zahl wird. Bei `identity` ist der Wert selbst die Zahl. `floorHalfMinusTen` ist die 5e-Regel. Mit `stepTable` kannst du eigene Schwellen als `[[score, number], ...]` auflisten.
- `proficiencyTiers`: Die Ausbildungsstufen einer Fertigkeit oder eines Rettungswurfs. Eine nicht eingetragene Fertigkeit bekommt die erste Stufe. Eine Stufe addiert `flat`, das `multiplier`-Fache eines Übungsbonus oder beides. Wenn dein System einen Übungsbonus hat, gib seine Herkunft mit `"proficiency": { "bonus": { "derived": "proficiency_bonus" } }` an.
- `proficiency`: Optional und nur für eine multiplizierende Stufe nötig.

Was die berechnete Zahl bedeutet, bestimmt die Auflösungsart: `dice-sum` addiert sie zum Wurf, `dice-pool` wirft entsprechend viele Würfel.

#### `dice-sum`: Würfel addieren

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

- `dice`: Die Anzahl und Seitenzahl der Würfel. Ihre Summe wird mit der Schwierigkeit verglichen.
- `advantage`: Ob der Game Master verlangen darf, zweimal zu würfeln und einen Wurf zu behalten.
- `naturals`: Was die höchste und niedrigste Seite eines einzelnen Würfels bei Proben und Rettungswürfen bewirken: `none`, `both`, `max-only` oder `min-only`. Lass den Eintrag weg, wenn nur gerechnet werden soll. Er setzt einen einzelnen Würfel voraus; ein 2d6-System muss daher `none` verwenden.
- `difficultyLadder`: Die Schwierigkeitswerte, aus denen der Game Master wählen soll. `dc` ist die Zahl, die die Summe erreichen muss. Statt die Zahl zu schreiben, darf der Game Master auch eine Stufe nennen, etwa `difficulty="Hard"`; die Engine liest die Zahl dann an der Stufe ab.

#### `dice-pool`: Würfel werfen und zählen

Die Zahl auf dem Bogen ist die **Größe des Pools**, kein zusätzlicher Bonus. Ein Grundwert von 3 und eine Tätigkeit mit Wert 2 ergeben fünf Würfel. Mehr steckt nicht dahinter: Es braucht weder neue Bogenbegriffe noch einen neuen Editor. Ein System, dessen Werte eine Handvoll Würfel bedeuten, wird mit denselben `abilities`, `skills` und `proficiencyTiers` geschrieben wie jedes andere.

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

- `die`: Wie viele Seiten ein Würfel des Pools hat, von 2 bis 100.
- `pool`: Der Bereich, auf den die Zahl vom Bogen vor allen Explosionen begrenzt wird. Ein `min` von 0 lässt einen leeren Pool ohne Wurf scheitern. `max` darf höchstens 100 sein. Mit `abilityPlusAbility: true` darf eine Attributsprobe über `with=` ein zweites Attribut nennen und beide werfen – für Systeme, die zwei Attribute zusammen würfeln. Ohne diese Angabe bedeutet `with=` bei einer Attributsprobe nichts.
- `target`: Die Augenzahl, die ein Würfel erreichen muss, um zu zählen. Setze `min` unter `max`, damit der Game Master sie mit `threshold=` je Probe verändern darf. Setze alle drei Werte gleich, um sie festzulegen.
- `double`: Optional. Eine Augenzahl ab `from` zählt doppelt.
- `explode`: Optional. Eine Augenzahl ab `from` wirft einen weiteren Würfel, der seinerseits explodieren kann. Zusätzlich zum ursprünglichen Pool sind höchstens `pool.max` weitere Würfel erlaubt. Eine Probe wirft daher höchstens zweimal `pool.max` Würfel, und ein niedriger `from`-Wert kann nicht endlos weiterwürfeln.
- Beide dürfen ein `min` tragen: die niedrigste Augenzahl, auf die eine Probe die Regel mit `explode="N"` oder `double="N"` am Tag oder über einen vom Charakter genutzten Eintrag senken darf. Mit einem `min` darfst du `from` weglassen; die Regel greift dann nur bei einer Probe, die sie verlangt. Eine Spezialisierung, durch die Zehnen bei einer einzigen Fertigkeit erneut würfeln, ist `"explode": { "min": 10 }`. `from` darf nicht unter `min` liegen.
- `cancel`: Optional. Eine Augenzahl bis einschließlich `upTo` zieht einen Erfolg ab. Die Anzahl sinkt nie unter null.
- `botch`: Optional. Wenn **kein** Würfel erfolgreich war und eine Augenzahl bis einschließlich `upTo` vorkam, ist die Probe ein Patzer. Ein Pool, dessen einziger Erfolg nachträglich gestrichen wurde, ist gescheitert, aber kein Patzer. `"rule": "halfOrMore"` liest den Patzer anders: Zeigt mindestens die Hälfte der zuerst geworfenen Würfel (Explosionen nicht mitgezählt) eine Augenzahl bis einschließlich `upTo`, ist das ein Patzer, wenn kein Würfel erfolgreich war. Andernfalls bleibt das Ergebnis bestehen und wird mit `complication="true"` markiert: Neben dem Ergebnis geht etwas schief. `"noSuccesses"` ist der Standard.
- `exceptional`: Optional. Eine erfolgreiche Probe mit mindestens so vielen verbleibenden Erfolgen ist ein kritischer Erfolg.
- `situationalDice`: Optional. Wie viele Würfel der Game Master bei einer Probe mit `bonus=` hinzufügen oder abziehen darf, etwa für Kunststücke, Wunden oder schlechtes Licht.
- `reroll`: Optional. Wiederholungswürfe, die das System gewährt, ohne dass jemand dafür bezahlt – bis zu sechs, jeweils mit einer `id`, die der Game Master bei einer Probe mit `reroll=` nennt. Augenzahlen bis einschließlich `upTo` werden erneut geworfen: mit `"once"` einmal, und die neue Augenzahl bleibt, wie sie fällt, mit `"until"` so lange, bis sie höher ausfällt. `upTo` reicht von 1 bis eins unter der Seitenzahl des Würfels, weil ein erneuter Wurf jeder Augenzahl nie enden würde; ein `until` endet ohnehin nach hundert erneuten Würfen bei einer Probe. Geworfen wird nur ein Wiederholungswurf, den die Probe nennt. Eine Probe wirft genau einen Wiederholungswurf: Hat auch ein Einsatz oder ein vom Charakter genutzter Eintrag einen gekauft, wird derjenige geworfen, der mehr Augenzahlen erfasst; bei gleicher Reichweite geht `until` vor `once`.
- `difficultyLadder`: `successes` gibt die nötigen Erfolge an. Eine Stufe darf auch ein `target` nennen, aber nur, wenn der Zielwert veränderlich ist, und nur innerhalb seines Bereichs. Mit diesem Zielwert zählt eine Probe auf dieser Stufe: wenn der Game Master die Stufe mit `difficulty=` nennt oder ein `dc=` schreibt, das genau eine Stufe verlangt. Verlangen mehrere Stufen dieselbe Zahl an Erfolgen, kann `dc=` allein nicht sagen, welche gemeint war, und der Zielwert bleibt beim Standard. Nenne dann die Stufe.

Die Augenzahlen für `cancel` und `botch` müssen unter dem niedrigsten Zielwert liegen. Jede von diesen Regeln genannte Augenzahl muss auf dem Würfel vorkommen können. Eine Regel, die nie greifen könnte, wird beim Import abgelehnt, statt erst im Spiel aufzufallen.

Ein als Paket ausgelieferter Pool-Regelsatz braucht Capability API 1.24; ein `min` bei `explode` oder `double`, `pool.abilityPlusAbility` und `botch.rule` brauchen 1.37, `reroll` braucht 1.38. Ein importierter Community-Regelsatz wird von der Engine geprüft, die ihn liest, und braucht deshalb keine solche Angabe.

#### Was der Game Master bei einer Pool-Probe schreiben darf

```
[skill_check: skill="Ward" dc="2" who="Bram the Quiet" threshold="8" bonus="-2" with="Sinew"]
[skill_check: skill="Ward" difficulty="Grim" explode="9"]
[skill_check: skill="Nerve" dc="1" with="Warmth"]
[skill_check: skill="Ward" dc="2" reroll="careful"]
```

- `dc` ist die Zahl der nötigen **Erfolge**, kein Zielwert. Sie darf von 1 bis zum größtmöglichen Ergebnis eines Wurfs reichen: das Maximum des Pools, verdoppelt bei explodierenden Würfeln und erneut verdoppelt bei doppelt zählenden Augenzahlen.
- `threshold=` verändert den Zielwert je Würfel und wird nur angeboten, solange `target.min` unter `target.max` liegt.
- `bonus=` fügt Würfel hinzu oder zieht sie ab und wird nur angeboten, wenn `situationalDice` deklariert ist.
- `with=` würfelt eine Fertigkeit oder einen Rettungswurf mit einem anderen als dem zugeordneten Attribut. Das funktioniert bei beiden Arten; ein 5e-Regelsatz erhält über dasselbe Tag-Attribut also „Strength (Intimidation)“. Bei einer Attributsprobe fügt es stattdessen ein zweites Attribut hinzu, sofern `pool.abilityPlusAbility` eingeschaltet ist.
- `difficulty=` nennt bei beiden Arten anstelle von `dc=` eine Stufe der Skala: ihre Erfolge (bei einer Summe ihre Schwierigkeit) und bei einem Pool ihren Zielwert. Ein daneben geschriebenes `dc=` hat bei der Zahl weiterhin Vorrang, `threshold=` beim Zielwert. Ein Name, auf den keine Stufe passt oder den sich zwei Stufen teilen, wird ignoriert.
- `explode=` und `double=` verschieben die Augenzahlen dieser Regeln für diese Probe und werden nur angeboten, wenn die Regel ein `min` hat.
- `reroll=` nennt einen deiner `reroll`-Einträge für diese Probe und wird nur angeboten, wenn du welche deklariert hast. Ein Name, auf den keiner passt, wird ignoriert.

Alle Angaben bleiben innerhalb deiner Dateivorgaben: Ein Wert außerhalb des Bereichs wird auf die nächste Grenze gesetzt. Ein vom Regelsatz nicht angebotenes Tag-Attribut wird ignoriert, statt die Probe abzulehnen. Der gespeicherte Datensatz zeigt anschließend, was der Wurf tatsächlich verwendet hat: Schwierigkeit und Schwelle als Zahlen, die zusätzlichen Würfel nach Anwendung deiner Grenzen, `with=` nur dann, wenn das Attribut wirklich ausgetauscht oder hinzugefügt wurde, eine Augenzahl nur, wenn die Probe sie verschoben hat, `reroll=` nur, wenn der genannte Wiederholungswurf der geworfene war und tatsächlich etwas geworfen hat, und `complication="true"`, wenn bei einem `halfOrMore`-Patzer neben einem Ergebnis etwas schiefging. Die Engine wirft die Würfel immer selbst. Ein vom Modell geschriebenes Pool-Ergebnis wird ersetzt. `mode="advantage"` wird ignoriert, weil diese Auflösungsart keinen Vorteil kennt. Ein vor dem Zug vom Spieler gewürfelter Würfel wird nicht verwendet.

#### Was nicht abgedeckt ist und warum

Jede der folgenden Mechaniken braucht eine eigene Auflösungsart, weil sie sich nicht durch das Zählen von Würfeln gegen einen Zielwert ausdrücken lässt:

- **Den höchsten Würfel nehmen**, wie in Blades in the Dark, braucht eine Stufe für Teilerfolge, die ein Probenergebnis nicht hat.
- **Haltungspools mit einem Wert vergleichen**, wie in Lasers and Feelings, entscheidet je Probe über "darüber oder darunter" und ist damit ein anderer Vergleich.
- **Symbolwürfel**, wie in Genesys, liefern überhaupt keine Zahlen.
- **Vergleichende Pools** lösen zwei Charaktere gleichzeitig auf; eine Probe hat nur einen würfelnden Charakter.
- **Unterwürfeln und offene Prozentwürfe** vergleichen in die andere Richtung.
- **Summenpools mit einem besonderen Würfel**, wie in OpenD6, addieren die Würfel und behandeln einen davon gesondert.

Die beiden früher fehlenden Möglichkeiten dieser Auflösungsart sind inzwischen modelliert. Der folgende Abschnitt zum Ausgeben von Ressourcen erklärt sie. Eine allgemeine Systemregel wie „einen Punkt für einen Erfolg ausgeben“ oder „einen Punkt ausgeben, um die Fehlschläge erneut zu werfen“ wird mit `resolution.spend` beschrieben. Einen Wiederholungswurf, den das System kostenlos gewährt, wenn die Situation es verlangt, beschreibt `resolution.reroll` (siehe oben). Ein Wiederholungswurf, der zu etwas gehört, das ein Charakter ausgewählt hat, steht als `mechanics.check` an einem Katalogeintrag und wird mit dessen eigenen Kosten bezahlt.

### Ressourcen ausgeben, um einen Wurf zu verändern

Manche Systeme lassen Spieler vor einem Wurf dafür bezahlen, etwa einen Willenspunkt für einen automatischen Erfolg. `resolution.spend` beschreibt dies als allgemeine Systemregel, nicht als etwas, das ein Charakter erst erwerben musste:

```json
"spend": [{ "pool": "resolve", "amount": 1, "successes": 1, "perCheck": 2 }]
```

- `pool` ist einer deiner `live.pools`. Er darf nicht leer starten, denn dann wäre zu Spielbeginn nichts zum Ausgeben darin.
- `amount` ist der Preis für EINEN Kauf. `successes`, `dice` und `reroll` geben an, was er kauft; mindestens eines davon muss er liefern. Erfolge werden erst nach dem Zählen der Würfel und nach dem Streichen von Erfolgen hinzugefügt, weil niemand sie gewürfelt hat. Zusätzliche Würfel werden innerhalb der Pool-Grenzen mitgeworfen. Ein `reroll` lautet `{ "upTo": 6, "mode": "once" }`, wird wie `resolution.reroll` gelesen und nur einmal gekauft, egal wie viele Käufe die Probe macht: Ein zweimal erneut geworfener Würfel ist immer noch ein Wiederholungswurf.
- `perCheck` bestimmt, wie viele Käufe eine Probe machen darf. Sie kann also höchstens für `amount * perCheck` Punkte einkaufen. Diese Grenze verhindert, dass ein voller Ressourcenpool einen unfehlbaren Wurf kauft. Erlaubt sind eine Zahl von 1 bis 10, ein Wert vom Bogen in derselben Form, die ein abgeleiteter Wert liest (`{ "abilityScore": "nerve" }`, `{ "derived": "focus" }`), oder `"pool"`: so viele, wie die Probe Würfel hat, also die Zahl vom Bogen dafür vor Wunden, Bonuswürfeln oder Modifikatoren. Ein Wert vom Bogen wird für den jeweils würfelnden Charakter gelesen, abgerundet und zwischen 0 und 100 gehalten. Ein Charakter, dessen Grenze 0 ergibt, kauft nichts und bezahlt nichts.
- Das ist nur für einen `dice-pool`-Regelsatz möglich. Ein Summenwurf hat weder Erfolge zum Hinzufügen noch einen Würfelpool zum Vergrößern. Ein `dice-sum`-Regelsatz mit `spend` wird deshalb beim Import abgelehnt.
- Bis zu vier Einträge sind erlaubt, und zwei davon dürfen nicht denselben Pool nennen, sonst könnte eine Probe nicht sagen, welchen sie meint.
- Ein Einsatz, der einen `reroll` kauft, ein `perCheck`, das keine Zahl ist, und mehr als zwei Einträge brauchen bei einem als Paket ausgelieferten Regelsatz Capability API 1.38.

**Die Angabe steht direkt an der Probe.** Der Game Master schreibt `[skill_check: skill="Nerve" dc="2" spend="resolve:1"]`, keinen getrennten `[sheet:]`-Befehl. Die Würfel werden vor den Bogenbefehlen geworfen; danach gäbe es nichts mehr zu verändern. Dieselbe Auflösung würfelt und bezahlt die Änderung.

### Ein Talisman, der einen Wurf verändert

`resolution.spend` ist eine Systemregel. Auch ein Eintrag, den ein Charakter tatsächlich AUSGEWÄHLT hat, kann eine Probe verändern: mit `mechanics.check` am Katalogeintrag.

```json
"mechanics": {
  "kind": "utility",
  "cost": [{ "pool": "blood", "amount": 1 }],
  "perCostStep": { "flat": 1 },
  "check": { "reroll": { "upTo": 1, "mode": "once" }, "successes": 1 }
}
```

- `reroll` wirft Würfel mit Augenzahlen bis einschließlich `upTo` erneut. `once` ersetzt jeden einmal und behält die neue Augenzahl; `until` wiederholt weiter. `upTo` muss unter der höchsten Würfelseite liegen, sonst würde der gesamte Pool endlos neu geworfen. Unabhängig von der Datei begrenzt die Engine, wie viele Würfel eine Probe erneut werfen darf.
- `dice` fügt vor dem Wurf Würfel hinzu, `successes` nach dem Zählen Erfolge. `threshold` setzt den Zielwert je Würfel für diesen einen Wurf innerhalb des von `target` erlaubten Bereichs.
- `explode` und `double` verschieben die Augenzahlen dieser Regeln für diesen einen Wurf, und zwar nur, wenn deine Regel ein `min` hat, und nur innerhalb davon: `"check": { "explode": 8 }` ist ein Eintrag, durch den Achten erneut würfeln. Wie `threshold` haben sie Vorrang vor dem, was der Game Master ins Tag geschrieben hat. Capability API 1.37.
- Mindestens eine dieser Angaben ist nötig. Sonst beschreibt der Eintrag nichts und wird abgelehnt.
- Nur ein `dice-pool`-Regelsatz kann diese Angaben umsetzen. Ein `dice-sum`-Regelsatz mit `mechanics.check` wird deshalb beim Import abgelehnt.

**Bezahlt werden die eigenen `cost` des Eintrags**, über denselben Ablauf wie bei jeder anderen Nutzung: der Pool sowie je eine Nutzung jedes Zählers, den derselbe Eintrag angelegt hat. `perCostStep` sagt aus, dass der Eintrag SKALIERT. Ein Eintrag mit dieser Angabe wird so oft gekauft, wie sein Preis bezahlt wurde. Ohne diese Angabe wird er einmal gekauft, unabhängig vom angebotenen Betrag.

**Der Game Master nennt ihn direkt an der Probe:** `[skill_check: skill="Brawl" dc="3" use="Potence" spend="blood:3"]`. Auch hier gibt es keinen getrennten Bogenbefehl, denn die Würfel werden vor der Buchführung geworfen.

**Alles oder nichts.** Reicht der Pool nicht aus, findet kein Kauf statt und nichts wird abgezogen. Der Wurf bleibt genau derselbe wie ohne Kauf. Auch Punkte, die keiner ganzen Zahl von Käufen entsprechen, kaufen nichts. Eine Anforderung über `perCheck` wird begrenzt statt abgelehnt; bezahlt wird nur das erlaubte Maximum. Die Engine berechnet all das. Der Game Master nennt den vom Spieler angekündigten Einsatz und fasst die Würfel nie an. Der Datensatz nennt den tatsächlich bezahlten Betrag, den angewendeten Eintrag, die ungewürfelten Erfolge und die Anzahl erneut geworfener Würfel. Ein nicht ausgewählter Talisman oder einer, dessen Katalog die Engine nicht lesen kann, bewirkt gar nichts; seine Angaben werden nicht ungeprüft übernommen.

### Der Bogen

- `sections` gruppiert die Einträge im Editor. Felder, abgeleitete Werte und Listen nennen mit `section` den Abschnitt, in dem sie stehen; Attribute, Fertigkeiten und Rettungswürfe dürfen das ebenfalls. Der Editor, der Bogen im Spiel und der Bogenblock des Game Master zeigen sie dann unter dieser Überschrift. Ein Bogen, der bei ihnen keinen Abschnitt nennt, zeigt sie genau wie bisher. Ein Abschnitt kann außerdem eine `untrained`-Regel tragen (siehe unten).
- `abilities` enthält die Grundwerte. `skills` und `saves` können jeweils das Attribut nennen, mit dem sie würfeln, sowie ein `cap` und eine `untrained`-Regel (siehe unten).
- `fields` enthält einzelne Werte. Typen: `number`, `text`, `longtext`, `boolean`, `enum` (eine feste Auswahlliste) und `dice` (Text wie `1d8`).
- `derived`-Werte werden aus anderen Werten berechnet und können nicht überschrieben werden. Die Operationen sind `sum`, `min`, `max`, `scale` (multiplizieren und runden), `stepTable` (einen Wert in Schwellen nachschlagen, etwa den Übungsbonus anhand der Stufe) und `enumTable` (eine Zahl für jeden Wert eines Enum-Felds oder eines Live-Zustands; siehe unten „Live-Zustände“).
- `lists` enthält Tabellen mit eigenen Spalten, etwa für Ausrüstung, Zauber oder Merkmale. Eine Liste mit `pools` macht jede Zeile zu einer Ressource mit eigenem Maximum, etwa für Klassenmerkmale mit begrenzten Nutzungen.
- `live` enthält, was sich im Spiel verändert: `pools` (Trefferpunkte, Zauberplätze, Grit), `tracks` (eine Zahl auf einer Skala, etwa Erschöpfung, oder eine Wundleiste mit ankreuzbaren Kästchen), `text` (kurze Notizen, etwa zur Konzentration eines Charakters), `conditions` und `states` (ein Wert aus einer geschlossenen Menge, etwa eine Gestalt oder eine Haltung).

Alles, was eine Zahl liest, benennt sie über einen Wertverweis: ein Objekt mit genau einem Schlüssel aus `const`, `field`, `derived`, `abilityScore`, `abilityMod`, `abilityModFromField`, `skillMod`, `saveMod`, `listSum`, `livePool`, `liveTrack` oder `itemStat`. Beispiel für einen Pool mit abgeleitetem Maximum: `"max": { "derived": "grit_max" }`.

- `listSum` addiert eine Zahlenspalte einer Liste: `{ "listSum": { "list": "gear", "column": "bulk", "onlyWhen": "packed" } }`. `onlyWhen` ist optional und nennt eine boolesche Spalte; nur die Zeilen, in denen sie gesetzt ist, zählen. Eine leere Zelle zählt mit dem Standardwert ihrer Spalte, und eine von `hideWhen` ausgeblendete Liste trägt nichts bei. Ember Roads berechnet so Burden aus der Ausrüstung, die ein Charakter eingepackt hat. Anders als eine eingetippte Zahl kann der Wert dadurch nie von der Liste abweichen. Die skalierte Spalte eines Katalogs kann keine Listensumme lesen, auch nicht über einen abgeleiteten Wert: Eine Liste kann skalierte Zellen enthalten, eigene oder solche, die die Summe zurücklesen, und die Neuberechnung käme nie zur Ruhe.
- `livePool` liest, was in einem deiner `live.pools` übrig ist (der Pool einer Listenzeile gehört nicht dazu). `liveTrack` liest eine deiner `live.tracks`, mit einem optionalen `read`: `"value"`, ihr aktueller Stand (Standard); `"filled"`, wie weit sie über ihrem `min` steht; `"remaining"`, wie weit sie unter ihrem `max` steht; oder, nur bei einer Wundleiste, `"penalty"`, der geltende Abzug. Ein ausgeblendeter Pool und eine ausgeblendete Leiste lesen 0. „Harm still to take“ in Gravewatch ist `{ "liveTrack": "harm", "read": "remaining" }`.
- Ein Live-Lesezugriff liest den Live-Zustand so, wie er steht, wenn eine Probe gewürfelt wird, ein Kampf beginnt oder der Bogenblock des Game Master geschrieben wird. Gibt es noch keinen (im Bogeneditor, in einer Importprüfung), liest er den Zustand, mit dem das Spiel beginnt: einen Pool voll oder leer, wie es sein `start` vorgibt, eine Leiste auf ihrem `default`, eine Wundleiste ohne Markierung.
- Nichts, was berechnet wird, bevor es einen Live-Zustand gibt, darf einen lesen: das `max` eines Pools oder einer Leiste, der Übungsbonus sowie die skalierte Spalte oder Skalierung eines Katalogs. Das gilt auch über einen abgeleiteten Wert, der einen liest, und über eine Fertigkeit, die ein Live-Wert begrenzt. Der Import nennt den Wert, der das tut.
- Bei einem als Paket ausgelieferten Regelsatz brauchen diese drei Capability API 1.39.
- `itemStat` liest die Gegenstände, die der Charakter besitzt, etwa den Guard-Wert der getragenen Rüstung, und unterliegt derselben Regel wie ein Live-Lesezugriff. Siehe [Gegenstände auf dem Bogen](#items-on-the-sheet). Capability API 1.52.

**Was eine Probe ohne Ausbildung tut.** Eine Fertigkeit oder ein Rettungswurf, in dem der Charakter keine Ausbildung hat (seine Stufe ist die erste), wird wie gewohnt gewürfelt, sofern der Regelsatz nichts anderes sagt. Das geschieht mit `untrained` an der Fertigkeit, am Rettungswurf oder an dem Abschnitt, in dem sie stehen; die eigene Regel der Fertigkeit oder des Rettungswurfs hat Vorrang:

```json
"sections": [{ "id": "labour", "label": "Labour", "untrained": { "by": -1 } }],
"skills": [
  { "id": "dig", "label": "Dig", "ability": "sinew", "section": "labour", "untrained": "refuse" },
  { "id": "listen", "label": "Listen", "ability": "nerve", "untrained": "harder" }
]
```

- `"normal"`: wie gewohnt (Standard).
- `{ "by": -3 }`: wird zur Zahl der Probe addiert, von -20 bis 20: Würfel bei einem Pool, ein fester Betrag bei einer Summe. Der Wert gehört zu der Zahl, die der Bogen zeigt und mit der jede Probe würfelt, und gilt vor einem etwaigen `cap`.
- `"harder"`: eine Stufe schwerer. Bei einem Pool steigt der Zielwert je Würfel um eins (und bleibt innerhalb seines Bereichs). Das braucht also einen Pool-Regelsatz mit veränderlichem Zielwert; überall sonst wird es beim Import abgelehnt, weil es nie einen Wurf verändern könnte.
- `"refuse"`: Die Probe wird gar nicht gewürfelt. Die Engine schreibt die Anforderung des Game Master mit `reason="untrained"` zurück, und damit ist sie erledigt: Nichts würfelt sie später, der Game Master erfährt, dass der Charakter sie nicht versuchen konnte, und die Erzählung sagt das. Der Endpunkt für Fertigkeitsproben beantwortet eine solche Probe mit 400 (`skill_check_untrained`), und eine Verzweigung daraus übernimmt keine der beiden Hälften. Der Bogeneditor zeigt dafür keine Zahl.
- Für einen Charakter ohne Bogen gibt es keine Ausbildung zu lesen; für ihn gilt daher keine Regel für fehlende Ausbildung.
- Die Erinnerung für den Game Master listet die Regeln für fehlende Ausbildung nach Abschnitt oder Fertigkeit auf, damit er nur Proben verlangt, die ein Charakter ablegen kann.

Abschnitte an Attributen, Fertigkeiten und Rettungswürfen sowie `untrained` an beliebiger Stelle brauchen bei einem als Paket ausgelieferten Regelsatz Capability API 1.41.

Eine Fertigkeit oder ein Rettungswurf kann ein `cap` tragen: einen Wertverweis auf das Höchste, was ihre Probe je ergeben darf – einen Wert, über den sie nicht hinauswachsen kann, oder eine Leiste, die sie niederhält. Gravewatch begrenzt Soothe auf das verbleibende Resolve eines Wächters: `"cap": { "livePool": "resolve" }`. Die begrenzte Zahl zeigt der Bogen, und mit ihr würfelt jede Probe. Ein Tausch mit `with=` wird mit der Zahl vor der Begrenzung berechnet und danach erneut begrenzt. Eine solche Obergrenze kann keinen Fertigkeits- oder Rettungswurfmodifikator lesen, ebenso wenig der abgeleitete Wert, den sie liest, oder ein davor deklarierter abgeleiteter Wert. So wird der Bogen weiterhin einmal von oben nach unten berechnet. Capability API 1.39.

`hideWhen` blendet ein Feld, einen abgeleiteten Wert, eine Liste, einen Pool oder eine einfache Leiste anhand des Werts eines anderen Feldes aus, und zwar auf genau eine von drei Arten: `equals` einem Wert, `notEquals` einem Wert (ausgeblendet, sobald das Feld etwas anderes enthält; so drückt man „nur für diese Art Charakter“ aus) oder `in` einer Liste von Werten. Jeder genannte Wert muss einer sein, den das Feld annehmen kann, sonst könnte die Regel nie zutreffen. Das wird Wert für Wert geprüft; bei `notEquals` und `in` gehören dazu auch der Bereich eines Zahlenfelds und die Länge eines Textfelds. Die Regel liest den Wert, den der Bogeneditor zeigt: Ein nicht gesetztes Feld hat den Wert, mit dem ein leerer Bogen beginnt, und ein Enum-Wert, den der Regelsatz nicht mehr anbietet, gilt als Standardwert des Feldes. Die 5e-Datei blendet mit `equals` Zauberplätze bei Charakteren ohne Zauber aus. Gravewatch hält Lampenöl mit `"hideWhen": { "field": "watch", "notEquals": "night" }` vom Bogen jedes Wächters fern, der nicht zur Nachtwache gehört. Eine Ebene kann keinen Wert entfernen, mit dem eine dieser Regeln vergleicht. Eine Wundleiste kann nicht ausgeblendet werden, weil Würfe und Kämpfe sie unabhängig von der Anzeige des Bogens lesen. `notEquals` und `in` brauchen Capability API 1.39.

Das `max` einer einfachen Leiste darf statt einer Zahl ein Wertverweis sein; ein Wert des Charakters legt dann fest, wie weit sie reicht: `"max": { "field": "willpower_rating" }`. `min` und `default` bleiben Zahlen. Eine Wundleiste mit benannten Stufen hat ein `max` gleich der Anzahl ihrer Stufen; eine Leiste mit nummerierten Kästchen ist so lang wie ihr eigenes `max` (siehe unten „Nummerierte Kästchen“). `alwaysShow: true` gibt eine Leiste auch auf ihrem Standardwert im Bogenblock des Game Master aus, für einen Wert, der in jedem Zug zählt. Ohne diese Angabe wird eine Leiste auf ihrem Standardwert weggelassen, weil drei Todesrettungswürfe auf null nichts aussagen. Bei einem als Paket ausgelieferten Regelsatz brauchen diese drei Capability API 1.37.

### Live-Zustände: Eine Gestalt, eine Haltung, ein Licht

Ein Enum-Feld wird beim Erstellen eines Charakters gewählt und bleibt dann so, und ein Zustand ist nur an oder aus. Ein **Live-Zustand** ist dagegen ein Wert aus einer geschlossenen Menge, der sich im Spiel ändert: die Gestalt eines Gestaltwandlers, eine Kampfhaltung, wie hell eine Laterne brennt.

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

- `values` sind zwei bis vierzig Werte, jeweils angezeigt als ihr Eintrag in `valueLabels` oder als der Wert selbst. Sie folgen der Regel für jede Beschriftung (eine Zeile, keine eckigen Klammern, keine Makro-Klammern) und enthalten nie ein doppeltes Anführungszeichen, weil der Bogenbefehl sie in Anführungszeichen setzt. Bis zu zwölf Live-Zustände.
- `default` ist der Startwert jedes Bogens; ohne Angabe ist es der erste Wert. Gespeichert wird nur ein Live-Zustand, der von seinem Standardwert abweicht, und ein gespeicherter Wert, den der Live-Zustand nicht mehr anbietet, gilt als Standardwert.
- `hideWhen` nimmt einen Live-Zustand anhand eines Feldwerts vom Bogen, wie bei einem Pool: Er wird nicht angezeigt, durch keinen Befehl gesetzt und von einer Tabelle (siehe unten) als gar kein Wert gelesen.
- Der Game Master setzt einen mit `[sheet: who="Name" op="state" state="Stance" value="Reckless"]`. Den Live-Zustand nennt er per ID oder Beschriftung, den Wert direkt oder per Beschriftung. Ein Live-Zustand oder ein Wert, den der Bogen nicht hat, wird abgelehnt (`unknown-state`, `unknown-value`). Die Engine bringt dem Game Master den Befehl nur bei und listet jeden Wert nur auf, wenn dein Regelsatz einen Live-Zustand hat. Jeder Bogenblock nennt den Wert jedes Live-Zustands, auch den Standardwert, denn „in menschlicher Gestalt“ ist eine Tatsache, die die Erzählung in jedem Zug braucht.
- Der Bogenbildschirm des Spiels zeigt für jeden Live-Zustand eine Auswahl.
- Eine Rast setzt einen mit einem Wiederherstellungsschritt zurück, der ihn nennt: `{ "state": "stance", "to": "default" }` oder `"to"` mit einem seiner Werte.

**Zahlen, die folgen.** Ein abgeleiteter Wert mit `"op": "enumTable"` liefert eine Zahl für jeden Wert eines Enum-Felds oder eines Live-Zustands:

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

- `from` nennt genau eines von `field` (ein Enum-Feld) oder `liveState`. Jede Zeile von `table` ist ein Wert, den dieses annehmen kann, eine bis vierzig davon. Ein von der Tabelle ausgelassener Wert und ein vom Bogen ausgeblendeter Live-Zustand lesen `default` (0, wenn du ihn weglässt).
- Ein Feld steht fest, sobald der Charakter erstellt ist. Eine darauf beruhende Tabelle darf daher ein Maximum speisen: Gravewatch gibt einem Wächter der Morgenwache ein Resolve mehr. Ein Live-Zustand ändert sich im Spiel. Eine darauf beruhende Tabelle ist daher wie `liveTrack` ein Live-Lesezugriff: Nichts, was berechnet wird, bevor es einen Live-Zustand gibt, darf sie lesen, weder direkt noch über einen anderen abgeleiteten Wert. Deshalb heißt der Schlüssel `liveState` und nicht `state`: Jeder Lesezugriff auf den Live-Zustand trägt `live` im Namen. Daran erkennen du und der Import, was ein Maximum nicht verwenden darf.
- Um die Zahl auf die Würfel zu bringen, verweise mit `resolution.adjust` darauf (siehe „Modifikatoren vom Bogen“). Ember Roads addiert seine Stance zu jedem Brawn-Wurf; Gravewatch zieht eine abgeblendete oder dunkle Laterne von jedem Nerve-Wurf ab.
- Eine Ebene darf aus einem Feld einen Wert entfernen, für den eine Tabelle eine Zeile hat. Solange die Ebene aktiv ist, wird diese Zeile einfach nie gelesen.

Live-Zustände, `enumTable` und ein Rast-Schritt, der einen `state` nennt, brauchen bei einem als Paket ausgelieferten Regelsatz Capability API 1.42.

### Wundleisten: Gesundheit als Leiste statt als Zahl

Viele Systeme zählen überhaupt keine Trefferpunkte. Sie haben eine Spalte aus Kästchen, jedes schlimmer als das vorige, und bei einer Verletzung wird eines markiert. Gib einem Eintrag in `live.tracks` die Angabe `kinds` und entweder benannte `levels` oder nummerierte `boxes` (siehe unten); damit wird aus einer Zahl auf einer Skala eine solche Leiste:

**Welche Form braucht dein System?** Ein Pool erfasst, WIE VIEL Schaden ankam. Eine Leiste erfasst die Menge UND die Art jeder einzelnen Verletzung. Wenn dein System zwischen Schlagschaden, tödlichem und schwer heilbarem Schaden unterscheidet und die Art nach dem Treffer weiter wichtig ist – weil schwer heilbarer Schaden langsamer heilt, nicht absorbiert werden kann oder schließlich tötet –, muss diese Art nach dem Wurf irgendwo gespeichert bleiben. Das kann nur eine Markierung. Ein Punktepool kann es nicht: Nach dem Abzug bleibt nur eine kleinere Zahl, und der Bogen weiß nicht mehr, welche Punkte welche Art hatten. Deshalb wird `combat.damageKinds` bei Gesundheit als Pool abgelehnt, statt still ignoriert. Ein Pool kann weiterhin `damageTypes` haben, gegen die ein Gegner resistent oder immun ist. Dabei geht es um die Menge des ankommenden Schadens, nicht um die Art der danach verbleibenden Wunde.

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

- `levels` enthält 1 bis 16 Stufen, die beste zuerst und die schlechteste zuletzt. Jede hat ein `label` und einen ganzzahligen `penalty` von höchstens 0. Eine große negative Zahl bedeutet in solchen Systemen "du bist außer Gefecht"; `-99` ist also erlaubt.
- `kinds` enthält 1 bis 6 Schadensarten, die die Leiste aufnehmen kann, jeweils mit `id`, kurzem `label` für das Kästchen und `severity`. Die Schweregrade müssen unterschiedlich sein. Ihre Zahlen legen nur die Reihenfolge fest; die Abstände kannst du frei wählen.
- Beides gehört zusammen. `kinds` ohne `levels` oder `boxes` wird abgelehnt, weil es nichts zu markieren gäbe. `levels` oder `boxes` ohne `kinds` wird abgelehnt, weil jede Markierung eine Art braucht. Eine Leiste hat `levels` oder `boxes`, nie beides.
- **Trenne die beiden Begriffe.** `kinds` definiert, was eine Markierung laut Regelsatz SEIN darf. Eine MARKIERUNG ist eine dieser Arten auf der Leiste während des Spiels. Die Definition enthält Arten; der Charakterbogen enthält Markierungen.
- Eine Wundleiste beginnt ohne Markierung; ihr `min` ist daher 0. Eine Leiste mit benannten Stufen ist so lang wie ihre Stufen, ihr `max` ist also `levels.length`. Eine anderslautende Datei wird abgelehnt statt still korrigiert, damit sie niemals zwei widersprüchliche Längen enthält. Eine Leiste mit Kästchen ist so lang wie ihr eigenes `max`, das ein vom Bogen berechneter Wert sein darf (siehe unten „Nummerierte Kästchen“).

**Die genauen Regeln**, denn eine ungenaue Auslegung führt zur falschen Leiste:

- Markierungen bleiben sortiert, **die schwerste zuerst**. Eine Leiste mit sieben Stufen hält höchstens sieben Markierungen.
- Eine Markierung wird **nach Schweregrad einsortiert**, niemals einfach hinten angehängt. Sie nimmt die höchste ihrem Schweregrad zustehende Position ein und schiebt leichtere Markierungen nach unten.
- Es gilt der Abzug der **untersten markierten Stufe**, niemals die Summe der markierten Stufen. Drei Markierungen auf der obigen Leiste ergeben `-3`, nicht `0 + -1 + -3`.
- `amount` nennt eine Anzahl Markierungen einer Art, die **einzeln nacheinander angewendet** werden. Eine währenddessen voll werdende Leiste folgt damit derselben Regel wie eine schon vorher volle.
- Eine Markierung auf einer **vollen** Leiste **erhöht deren leichteste Markierung um eine Stufe**, statt eine weitere hinzuzufügen: genau eine Stufe auf deiner eigenen Rangfolge der Schadensarten, unabhängig von der neu hinzukommenden Art.
- Eine Markierung, die über den höchsten Schweregrad hinaus erhöht würde, bleibt beim höchsten. Die nicht mehr untergebrachte Markierung zählt als **Überlauf**. Dieser wird gespeichert, damit ein Neuladen bereits erlittenen Schaden nicht vergisst.
- **Heilung verwendet denselben Befehl mit negativer Menge.** Sie entfernt den Überlauf vor jeder Markierung. Eine Heilung, die eine der Arten der Leiste nennt, entfernt nur Markierungen dieser Art. „Der Schlagschaden heilt, der tödliche bleibt“ ist also `kind="bashing"` mit negativer Menge. Eine Heilung, die keine Art der Leiste nennt, entfernt zuerst die leichtesten Markierungen.

**Im Spiel markieren.** Der Game Master schreibt `[sheet: op="damage" track="harm" kind="knock" amount="1"]` und heilt mit negativem `amount`. Bei einer Leiste, die nach Kästchen gefüllt wird, ergänzt er `box="3"` für das Kästchen, auf das der Treffer fällt. Die Pool-Form von `damage`, die stattdessen `pool=` nennt, bleibt unverändert. Der einfache `track`-Befehl wird für Wundleisten abgelehnt: Eine bloße Zahl kann die Art der neuen Markierungen nicht nennen. Spieler können Kästchen auch von Hand auf dem Bogen markieren und leeren, wie es solche Systeme vorsehen.

**Auch ein Kampf kann eine Wundleiste markieren.** Verweise mit `combat.health` auf die Leiste statt auf einen Pool. Ein ankommender Treffer markiert dann so viele Kästchen, wie `combat.damageKinds.marks` vorgibt, und verwendet die Art, auf die dieser Block den Schadenstyp abbildet. Ein Charakter mit voller Leiste geht zu Boden; das liest deine Sterberegel. Bei einer Leiste, die nach Kästchen gefüllt wird, nennt `per-point`-Schaden das Kästchen, auf das er fällt, und `per-blow` zielt auf das erste. Ein Treffer, den kein Kästchen mehr aufnehmen kann, schickt den Charakter zu Boden – genau das bedeutet Ausgeschaltetsein in den Systemen mit einer solchen Leiste. Heilung entfernt eine Markierung, die leichteste, gleich welcher Art. Temporäre Punkte werden abgelehnt, weil eine Leiste keinen Puffer dafür hat. Die Engine liest eine Leiste als ihre VERBLEIBENDEN Stufen. Alles andere am Kampf – zu Boden gehen, wiederbelebt werden, Protokoll und Rückblick – bleibt dadurch unverändert.

**Eine Rast kann eine Wundleiste heilen.** Ein Wiederherstellungsschritt mit `"to"` reduziert sie einschließlich Überlauf auf so viele Markierungen. Ein Schritt mit NEGATIVEM `"by"` entfernt die genannte Anzahl, zuerst den Überlauf; `"by": { "const": -2 }` entfernt also zwei. Ein Schritt, der zusätzlich eine `kind` nennt, entfernt nur Markierungen dieser Art: „Catch your breath“ in Gravewatch ist `{ "track": "harm", "kind": "knock", "to": "min" }`; das heilt die Knock-Markierungen und lässt die Tear-Markierungen stehen. Ein Schritt, der Markierungen HINZUFÜGEN würde, tut nichts, weil eine Rast keinen Schaden markiert. Das gilt auch für ein positives `"by"`: Es wird kommentarlos übersprungen. Schreib also das Minuszeichen.

#### Nummerierte Kästchen statt benannter Stufen

Ist eine Leiste so lang, wie ein Charakter zäh ist, gib ihr `boxes` statt `levels`. Sie hat so viele Kästchen wie ihr eigenes `max`, höchstens 64. Dieser Wert darf vom Bogen berechnet werden, aber nie den Live-Zustand lesen:

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

- Der geltende Abzug wird in `table` nachgeschlagen (eine Stufentabelle wie bei einem abgeleiteten Wert, mit Abzügen von höchstens 0), und zwar bei der Zahl der markierten (`filled`) oder freien (`remaining`) Kästchen, wie hoch sie auch ist. Eine Tabelle kann also auch ohne jede Markierung etwas kosten. Welche Kästchen markiert sind, spielt für den Abzug keine Rolle.
- Die Kästchen sind nummeriert: Der Bogen zeigt „Box 3“, und der Bogenblock des Game Master nennt, wie viele von wie vielen markiert sind, statt eine Stufe zu nennen.

#### Füllen nach Kästchen und Leisten, die ablehnen

- `"fill": "indexed"` setzt eine Markierung auf das Kästchen, das der Befehl mit `box=` nennt (ohne Angabe auf das erste), oder auf das nächste freie darüber, und verschiebt eine gesetzte Markierung nie. Eine Heilung entfernt die höchste der leichtesten Markierungen. Auf dem Bogenbildschirm kann ein Spieler jedes freie Kästchen anklicken.
- `"onFull": "refuse"` lehnt eine Markierung ab, für die kein Kästchen frei ist, statt die leichteste Markierung zu erhöhen. Abgelehnt wird der gesamte Befehl: Drei Markierungen bei zwei freien Kästchen setzen gar keine. Eine Leiste, die nach Kästchen gefüllt wird, lehnt immer ab; gib das mit `"onFull": "refuse"` daneben an.

#### Stufen, die eine Liste hinzufügt

`"extra": { "list": "scars", "countColumn": "levels", "penaltyColumn": "penalty" }` an einer Leiste mit benannten Stufen lässt eine Liste auf dem Bogen sie je Charakter verlängern. Jede Zeile fügt der Reihe nach ihre Anzahl an Stufen (höchstens 16) mit ihrem Abzug hinzu, und zwar hinter der letzten Stufe, deren Abzug mindestens genauso gut ist. Stimmen die Abzüge überein, werden die neuen Stufen nach dieser Stufe benannt, sonst nach ihrem Abzug. Die Narben in Gravewatch funktionieren so. Eine vom Bogen ausgeblendete Liste fügt nichts hinzu. Wird eine Zeile gelöscht, während ihre Stufen markiert sind, bleiben die nicht mehr passenden Markierungen als Überlauf erhalten: Schaden wird nicht durch Bearbeiten des Bogens rückgängig gemacht.

`boxes`, `fill`, `onFull`, `extra` und die `kind` eines Rast-Schritts brauchen bei einem als Paket ausgelieferten Regelsatz Capability API 1.40.

### Der Abzug auf deine Würfe

`resolution.penaltyFrom` nennt die Wundleiste, deren Abzug für jede Probe dieses Regelsatzes gilt. Die Angabe wird deklariert, nicht vorausgesetzt. Ohne sie würfelt ein Regelsatz daher genauso wie vor der Einführung von Wundleisten.

Was der Abzug BEWIRKT, bestimmt wie beim eigenen Bogenwert die Auflösungsart:

- Bei `dice-pool` werden **Würfel aus dem Pool entfernt**, höchstens bis zu deinem `pool.min`. Bei `pool.min` gleich 1 wirft selbst ein Charakter auf der untersten Stufe einen Würfel. Bei `pool.min` gleich 0 wirft er keinen und scheitert ohne Wurf.
- Bei `dice-sum` gilt ein **fester Modifikator auf den Wurf**, verrechnet mit derselben Zahl, die bereits Attribut und Ausbildung addieren.

Die genannte Leiste muss eine Wundleiste sein. Eine einfache Zahlenleiste hat keinen anzuwendenden Abzug; ein Verweis darauf wird beim Import abgelehnt. Das Ergebnis nennt den angewendeten Abzug, damit Spieler die kleinere Würfelanzahl nachvollziehen können. Auch der Bogenblock des Game Master zeigt die Stufe und ihren Abzug.

### Modifikatoren vom Bogen

Manche Zahlen auf einem Bogen fahren bei jedem Wurf mit, den sie betreffen: ein schwerer Rucksack bei jedem Klettern, ein Rüstungsabzug bei jedem Schleichen, ein Segen bei allem. `resolution.adjust` drückt das bei beiden Auflösungsarten aus:

```json
"adjust": [{ "value": { "derived": "burdened" }, "abilities": ["brawn"] }]
```

- `value` ist ein Wert vom Bogen in derselben Form, die ein abgeleiteter Wert liest, einschließlich des Live-Zustands. Eine negative Zahl zieht ab. Ember Roads addiert den Bulk-Wert der eingepackten Ausrüstung eines Charakters zu Burden, und eine Stufentabelle macht aus je drei Punkten davon einen Abzug von eins auf jeden Brawn-Wurf. Über eine Tabelle, die auf einem Live-Zustand beruht, wirkt sich eine Gestalt oder Haltung auf die Würfel aus (siehe „Live-Zustände“).
- `abilities` ist optional. Mit dieser Angabe gilt der Modifikator nur für eine Probe, die mit einem dieser Attribute würfelt: das Attribut selbst, eine Fertigkeit oder ein Rettungswurf mit diesem Attribut, eine per `with=` damit gewürfelte Fertigkeit oder ein Attributspaar, das es enthält. Ohne die Angabe gilt er für jede Probe.
- Er wird dort angewendet, wo auch der Wundabzug gilt: als Würfel bei einem Pool, mit derselben Untergrenze `pool.min`, und als feste Zahl bei einer Summe, innerhalb des Modifikators, den der Datensatz zusammenzählt. Bis zu acht Einträge, die für jede Probe addiert werden.
- Ein Regelsatzkampf addiert ihn zu Kampfbeginn zu jedem Wurf, den er aus einem Bogen bildet: Ein mit einem Attribut oder einer Fertigkeit gewürfelter Angriff, ein Rettungswurf sowie eine Probe in einer vergleichenden Probe und eine Initiative, die aus einem Attribut, einer Fertigkeit oder einem Rettungswurf gelesen werden, erhalten die Einträge für dieses Attribut. Alles andere, was gewürfelt wird, erhält nur die Einträge für jeden Wurf.
- Ein Charakter ohne Bogen erhält nichts davon, genauso wie er keinen Modifikator erhält.

Das Ergebnis nennt, was der Bogen hinzugefügt hat (`adjust="-2"` im Datensatz und eine Zeile auf der Würfelkarte), damit Spieler nachvollziehen können, warum ein Wurf so ausgefallen ist. Bei einem als Paket ausgelieferten Regelsatz braucht das Capability API 1.38.

### Rasten

Eine Rast ist eine Liste von Wiederherstellungsschritten und zu entfernenden Dingen. Jeder Schritt nennt genau ein Ziel (`pool`, `poolGroup`, `listPools`, `track` oder `state`) und setzt es entweder auf einen Wert (`"to": "max"`, `"to": "min"` oder eine Zahl) oder verändert es (`"by": { "const": 1 }` oder `"by": { "fractionOfMax": 0.5 }`). Ein Schritt für eine Wundleiste kann sie nur heilen, sein `by` ist daher negativ; siehe oben. Ein Schritt für einen Live-Zustand setzt ihn nur, und zwar auf `"default"` oder einen seiner Werte.

### Text für den Game Master

- `checkGuidance` ersetzt den eingebauten Absatz, der dem Game Master erklärt, wie er eine Probe anfordert. Nenne das System und beschreibe, wann gewürfelt werden soll. Der Game Master nennt nur Fertigkeit und Schwierigkeit. Die Engine würfelt und rechnet anhand des Bogens; fordere das Modell deshalb nicht zum Rechnen auf.
- `sheetGuidance` stellt die Charakterbögen im Prompt vor. Erkläre darin, welche Ressourcen wichtig sind und wann sie verbraucht werden sollen.
- `worldGuidance` ist optional und wird einmal bei der Welterzeugung gelesen. So passt die vom Game Master erfundene Welt zu deinen Regeln: kein Schießpulver, seltene Magie, wandelnde Tote. Der Text gelangt nie in einen Zug.
- `sheetSummary` wählt die Felder, abgeleiteten Werte und Listenzeilen, die der Game Master pro Charakter sieht. Die Engine zeigt immer Attributsmodifikatoren, ausgebildete Fertigkeiten und Rettungswürfe sowie laufende Werte. Halte den Rest kurz, denn er wird in jedem Zug gesendet. Die `nameColumn` einer Zusammenfassungsliste darf eine Text- oder Enum-Spalte sein (ein Enum zeigt die Beschriftung seines Werts), und `columns` nennt bis zu drei weitere Spalten der Zeile, die nach dem Namen ausgegeben werden. Der Block lautet dann `Gear: Crowbar 1d6` statt nur Namen. Eine boolesche Spalte gibt ihre Beschriftung aus, wenn sie gesetzt ist. Beides braucht bei einem als Paket ausgelieferten Regelsatz Capability API 1.37.
- Den Bogenbefehl musst du in `sheetGuidance` nicht erklären. Die Engine bringt dem Game Master jeden Befehl selbst bei, mit den Namen, die deine Datei deklariert. Hat dein Regelsatz eine Wundleiste, ergänzt sie den Befehl `op="damage"` zum Markieren und listet die Stufen und Schadensarten der Leiste auf. Hat er Live-Zustände, ergänzt sie den Befehl `op="state"` und listet die möglichen Werte jedes Live-Zustands auf.

## Kataloge: Fertige Einträge für die Listen des Bogens

Zauberlisten, Ausrüstungstabellen oder seitenlange Klassenmerkmale Zeile für Zeile einzugeben, ist mühsam. Ein Katalog ist eine benannte Sammlung fertiger Einträge, die du mit dem Regelsatz auslieferst. Der Bogeneditor bietet sie in einer Auswahl an jeder vom Katalog befüllten Liste an. Die Auswahl eines Eintrags füllt die Zeile aus.

Kataloge sind optional. Ein Regelsatz darf bis zu zwölf enthalten. Die Engine kennt den Inhalt keines davon: Jede ID, Spalte, jeder Filter und jedes Wort stammt aus deiner Datei.

### Der Katalogkopf

Der Kopf steht in `catalogs` auf der obersten Ebene der Datei neben `gm`.

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

- `id` und `label`: Die ID folgt den Regeln für Bogen-IDs; die Beschriftung benennt die Auswahl.
- `holds`: `"rows"` (Standard und der Wert aller vor dieser Veröffentlichung geschriebenen Kataloge), `"creatures"` oder `"items"`. Ein Kreaturenkatalog ist ein Bestiarium für Kämpfe, und ein Gegenstandskatalog listet Dinge auf, die eine Gruppe mit sich führt. Keiner von beiden schreibt etwas auf einen Bogen, deklariert `feeds` oder wird je in der Auswahl angeboten. Siehe unten [Kreaturen](#creatures-a-bestiary-a-fight-reads) und [Gegenstände](#items-what-a-party-carries).
- `feeds`: Die Listen deines Bogens, in die Einträge dieses Katalogs schreiben dürfen, mindestens eine und höchstens acht. Für Zeilenkataloge erforderlich, bei Kreaturen- und Gegenstandskatalogen nicht erlaubt. Ein Eintrag kann niemals in eine hier nicht genannte Liste schreiben oder einen Wert ablegen, den die Spalten der Liste nicht aufnehmen können.
- `filters`: Optional, höchstens acht. Die Filter, mit denen die Auswahl eingegrenzt werden kann. Ein Filter ist `number`, ein `text`-Wert oder `tags` (mehrere Wörter). `startFrom` nennt ein Bogenfeld, mit dessen Wert die Auswahl startet. Hat ein Charakter die Berufung Tinker, sieht er dadurch zuerst Tinker-Einträge.
- `units`: Optional. Was eine Reichweite oder Flächengröße im `mechanics`-Block eines Eintrags in deinem System bedeutet.

### Ein Eintrag

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

- `id`: Kleinbuchstaben, Ziffern und einzelne Bindestriche; innerhalb des Katalogs eindeutig.
- `label` und `summary`: Was die Auswahl zeigt. Die Zusammenfassung ist optional, einzeilig und höchstens 300 Zeichen lang.
- `filters`: Die Werte für die im Kopf deklarierten Filter. Ein `number`-Filter nimmt eine Zahl, ein `text`-Filter eine Zeichenfolge und ein `tags`-Filter eine Liste von Zeichenfolgen.
- `rows`: Was die Auswahl des Eintrags schreibt, eine bis sechs Zeilen. `list` ist eine der `feeds`-Listen des Katalogs. Die Schlüssel von `values` sind die Spalten-IDs dieser Liste.
- `creature`: Ein Gegner statt Zeilen in einem Katalog, dessen `holds` Kreaturen angibt. Ein Eintrag enthält genau eines von `rows` oder `creature`. Eine Kreatur trägt kein `mechanics`; sie beschreibt ihr Verhalten in ihren eigenen Aktionen.

Jeder Wert wird gegen die Spalten der Zielliste geprüft. Ein falsch geschriebener Spaltenname oder eine Zahl außerhalb des Spaltenbereichs wird daher zusammen mit dem zugehörigen Eintrag gemeldet. Einträge innerhalb der Regelsatzdatei werden beim Laden des Regelsatzes geprüft, bei einer importierten Datei also beim Import. Eine separate Katalogdatei eines Pakets wird geprüft, sobald die Auswahl sie erstmals anfordert. Hat die Datei einen Fehler, zeigt die Auswahl dessen Gründe statt Einträgen.

### Ein Eintrag, mehrere Listen

Ein Merkmal mit begrenzten Nutzungen besteht auf dem Bogen aus zwei Zeilen: dem Merkmal selbst und seinem Nutzungszähler. Ausgewählt wird es trotzdem nur einmal.

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

### Werte, die der Regelsatz aktuell hält

Nach der Auswahl gehören die Zahlen einer Zeile dem Spieler. Eine sinnvolle Ausnahme ist ein Maximum, das dem Charakter folgt, etwa Nutzungen in Höhe eines Attributswerts oder eine Klassenressource, die mit der Stufe wächst. Eine Zeile darf in einer `scaled`-Zuordnung bis zu vier ihrer eigenen Zahlenspalten nennen. Der Bogeneditor hält diese Zellen dann aktuell.

```json
{
  "list": "tricks",
  "values": { "name": "Last Ember", "uses": 1, "recharge": "camp" },
  "scaled": { "uses": { "from": { "abilityScore": "heart" } } }
}
```

- Der Schlüssel ist eine der `number`-Spalten der Liste.
- `from` ist ein gewöhnlicher Wertverweis mit demselben geschlossenen Vokabular wie überall sonst. Komplexere Berechnungen gehören in einen vom Bogen deklarierten `derived`-Wert, auf den `from` verweist (`"from": { "derived": "lay_on_hands_max" }`). Hier kommt keine neue Rechenart hinzu.
- `table` ist optional. Mit dieser Angabe wird der referenzierte Wert in einer Stufentabelle nachgeschlagen. So wird aus einer Stufe eine Zahl: `"scaled": { "max": { "from": { "field": "level" }, "table": [[1, 2], [3, 3], [6, 4]] } }`.
- `values` muss weiterhin eine einfache Zahl für die Spalte enthalten. Fehlt sie, wird die Zeile abgelehnt. Sie gilt, bevor ein Bogen bekannt ist, und bleibt auf einem Bogen ohne den entsprechenden Verweis erhalten.
- Eine Zeile mit `scaled` muss die einzige Zeile dieses Eintrags für die betreffende Liste sein. Dadurch passt eine markierte Bogenzeile immer zu genau einer Vorgabe.

Der Wert wird beim Bearbeiten des Bogens berechnet, niemals beim Lesen. Eine gespeicherte Zeile enthält daher immer tatsächlich die angegebene Zahl. Sie wird an die Zielspalte angepasst: auf deren `min` und `max` begrenzt und bei Ganzzahlspalten abgerundet. Im obigen Beispiel hat ein Charakter mit Heart 3 drei Nutzungen, einer mit Heart 0 oder weniger keine. Die Zeile bleibt mit 0 Nutzungen auf seinem Bogen. Da ein Zähler mit Maximum 0 kein Pool ist, kann im Spiel nichts daraus verbraucht werden.

Skalierte Spalten brauchen bei einem als Paket ausgelieferten Regelsatz Capability API 1.23. Ein importierter Community-Regelsatz wird von der lesenden Engine geprüft und braucht keine solche Angabe.

### Ausgewählte Zeilen sind Kopien

Jede ausgewählte Zeile wird mit einem zusätzlichen Schlüssel `_catalog` auf den Bogen kopiert. Er enthält `<catalog id>/<entry id>`. Spalten-IDs beginnen immer mit einem Buchstaben; dieser Schlüssel kann deshalb nie mit einem deiner Schlüssel zusammenfallen.

Die Kopie gehört dem Charakter. Spieler können anschließend alles daran bearbeiten. Der Bogen funktioniert weiter, wenn dein Regelsatz nicht installiert ist, und eine neue veröffentlichte Regelsatzversion schreibt niemals einen Charakter um. Anhand der Markierung zeigt die Auswahl, was der Bogen bereits enthält. Auch die nachfolgend beschriebene Aktualisierung liest sie.

### Aus dem Regelsatz aktualisieren

Weil eine ausgewählte Zeile ihre Markierung behält, kann der Bogeneditor auf abweichende neuere Texte hinweisen. Eine kurze Zeile unter der Liste nennt die Zahl der Zeilen mit neuerem Text. **Review** (prüfen) zeigt für jede davon den Bogeninhalt neben der Regelsatzfassung sowie ein Auswahlkästchen je Zeile. Erst mit **Update selected** (Auswahl aktualisieren) wird etwas geschrieben, und nur die abweichenden Spalten der ausgewählten Zeilen. Alles andere in der Zeile bleibt erhalten, einschließlich der Markierung.

Der Vergleich ist absichtlich eng begrenzt:

- Vorhandene Werte werden nur in Spalten der Typen `text`, `longtext`, `dice` und `enum` verglichen. Vorhandene `number`- und `boolean`-Werte gehören dem Spieler und bleiben erhalten, auch 0 und false. Eine noch nicht in der Zeile enthaltene Spalte kann mit ihrem typisierten Wert angeboten werden, einschließlich Zahlen und Schaltern. Skalierte Spalten sind ausgenommen, weil sie bereits dem Bogen folgen.
- Nur Spalten, die dein Eintrag setzt, werden betrachtet. Eine ausgelassene Spalte wird nie verändert, unabhängig von ihrem Bogeninhalt.
- Ein von der Spalte selbst abgelehnter Wert wird übersprungen statt geschrieben, etwa ein nicht mehr angebotenes `enum`-Element oder Text über `maxLength`.
- Eine Zeile wird ihrer ursprünglichen Eintragszeile anhand ihrer Position unter den gleich markierten Zeilen dieser Liste zugeordnet. Das funktioniert, solange der Bogen noch genau so viele davon enthält, wie dein Eintrag schreibt. Andernfalls ist die Zuordnung nur möglich, wenn dein Eintrag genau eine Zeile für diese Liste schreibt. Hat ein Spieler eine Zeile eines zweizeiligen Eintrags gelöscht, bleibt der Eintrag unverändert, statt die Zuordnung zu erraten.
- Eine Zeile, deren Eintrag dein Katalog nicht mehr enthält, bleibt still unverändert.

Umformulierungen und Umbenennungen können also nach Zustimmung des Spielers auch Charaktere erreichen, die den Eintrag schon ausgewählt haben. Eine Änderung der Bedeutung einer Zahl kann und wird das nicht: Sobald die Zeile dem Spieler gehört, gehört ihm auch diese Spalte.

### `mechanics`: Was ein Eintrag in Zahlen bewirkt

Ein Eintrag kann einen optionalen `mechanics`-Block mit seinen Wirkungen in Zahlen tragen: `kind` (`attack`, `heal`, `buff`, `debuff`, `utility`, `rider`), `range`, `area`, `targets`, `targetCount`, `friendlyFire`, `amount` (Würfel wie `2d6` oder eine feste Zahl), `damageType`, `attackRoll`, `autoHit`, `save` (einer der Rettungswürfe des Bogens samt Erfolgswirkung), `applies` (Zustände für betroffene Ziele), `temporary` (temporäre Punkte auf dem Gesundheitspool), `scales` (eine mit dem Bogen wachsende Menge), `cost` (der bei Nutzung belastete Pool), `perCostStep`, `budget` (der verbrauchte Teil des Aktionsbudgets), `concentration`, `reaction`, `plus`, `free`, `gives`, `standard`, `rider` und `check`.

Die Auswahl zeigt diesen Block als einzelne Zeile. Wer den Rest liest, hängt vom gewählten Regelsatzblock ab:

- Mit einem [`combat`-Block](#combat-a-fight-your-own-rules-resolve) liest der Kampf dessen Kampfwirkungen. `range`, `area` und `friendlyFire` gelten auf einem Schlachtfeld mit Positionen. `reaction` markiert einen Eintrag als Antwort auf etwas statt als Aktion in einem Zug und nennt den Moment, auf den er wartet (siehe [Fenster](#windows-holding-the-fight-open)). Ein nur mit `true` markierter Eintrag nennt keinen Moment und erscheint in keinem Menü. `check` gilt wie oben beschrieben für Fertigkeitsproben.
- Mit ausschließlich einem [`battle`-Block](#battles-lending-the-sheet-to-marinaras-combat) liest der Kampf `kind`, `range`, `area`, `friendlyFire`, `amount`, `damageType` und `cost`, weil Marinaras eigenes Kampfsystem genau diese Angaben aufnehmen kann.

Das Vokabular ist geschlossen. Ein nicht in der obigen Liste enthaltener Schlüssel oder Wert wird deshalb abgelehnt statt still ignoriert.

`cost` bezahlt auch der `use`-Befehl des Game Master außerhalb eines Kampfes. Darum geht es im nächsten Abschnitt.

### Der `use`-Befehl: Den Game Master einen von dir festgelegten Preis bezahlen lassen

Während der Erzählung hält der Game Master jeden Bogen mit `[sheet: ...]`-Befehlen aktuell: `spend`, `restore` (`heal` bedeutet dasselbe), `damage`, `temp`, `track`, `condition`, `note` und `rest`. Ein Regelsatz mit Katalogen erhält einen weiteren:

```
[sheet: who="Mira" op="use" name="Fireball"]
[sheet: who="Mira" op="use" name="Fireball" pool="3rd-level slots"]
```

`op="cast"` bedeutet dasselbe wie `op="use"`, und `spell=` dasselbe wie `name=`. So funktioniert die vom Game Master gewählte Formulierung, ohne dass dein Format das Wort "Zauber" kennen muss.

Der Name wird ohne Beachtung der Groß- und Kleinschreibung mit den Zeilen dieses Charakterbogens abgeglichen, die aus einem deiner Kataloge stammen. Eine Zeile reagiert auf den dem Game Master gezeigten Namen – zuerst die `sheetSummary`-Namensspalte der Liste, dann deren `pools.nameColumn`, dann ihre erste Textspalte – und auf das `label` des ursprünglichen Eintrags. So bleibt sie auch nach einer Umbenennung durch den Spieler erreichbar. Ein Name ohne Treffer und ein Name, der auf zwei verschiedene Einträge passt, werden beide abgelehnt.

Was verbraucht wird:

- Jeder Bestandteil von `mechanics.cost` des Eintrags. Nennt ein Bestandteil einen laufenden Pool, wird daraus bezahlt. Nennt er eine Pool-GRUPPE, wird in Deklarationsreihenfolge der erste Pool der Gruppe gewählt, der den Preis bezahlen kann. Es gibt keinen automatischen Aufstieg zu einem höheren Pool, denn eine Gruppe ist nicht immer eine Stufenfolge.
- Zusätzlich eine Nutzung aus jedem Listenzeilen-Pool, den derselbe Eintrag angelegt hat, etwa dem Nutzungszähler eines Merkmals. Das ist die zweite Zeile des obigen Eintrags `Last Ember`. Ein Zähler mit Maximum 0 kann keine Nutzung bereitstellen. Der Befehl wird deshalb abgelehnt, statt kostenlos ausgeführt zu werden.

`pool=` ermöglicht das Verstärken durch einen anderen Pool: derselbe einzelne Preis, bezahlt aus einem anderen Pool derselben Gruppe. Das wird nur akzeptiert, wenn die Kosten genau einen Bestandteil haben und der genannte Pool zu dessen Gruppe gehört. Alles andere wird abgelehnt statt umgedeutet.

Es gilt alles oder nichts. Kann ein Teil nicht bezahlt werden, wird der gesamte Befehl abgelehnt, nichts verändert und der Spieler informiert. Ein völlig kostenloser Eintrag, etwa ein Zaubertrick oder ein passives Merkmal, wird angenommen und verändert nichts.

### Direkt in der Regelsatzdatei oder als eigene Datei

Ein kleiner Katalog steht direkt in `ruleset.json`, im Feld `entries` des Katalogkopfs. Ein langer Katalog liegt in einer eigenen Datei, die der Kopf stattdessen mit `asset` nennt. Ein Katalog enthält genau eines von beiden.

```json
{ "id": "knacks", "label": "Knacks", "feeds": ["knacks"], "asset": "catalogs/knacks.json" }
```

Der Pfad ist immer `catalogs/<the catalog's id>.json`. Die Datei selbst sieht so aus:

```json
{ "schemaVersion": 1, "catalog": "knacks", "entries": [] }
```

Separate Katalogdateien gibt es für Pakete, die über den offiziellen Katalog veröffentlicht werden. Das Paket führt die Datei in `contributions.assets.paths` neben `ruleset.json` auf und braucht Capability API 1.21. Ein Kreaturenkatalog braucht Capability API 1.27, unabhängig davon, ob er direkt eingebettet ist oder eine eigene Datei hat, und ein Gegenstandskatalog braucht 1.49. **Ein als Einzeldatei importierter oder über ein GitHub-Repository geteilter Regelsatz enthält seine Kataloge direkt in der Datei.** Sie müssen daher in die 256-KB-Grenze der gesamten Regelsatzdatei passen. Das reicht für einige Hundert kurze Einträge.

Die Grenzen sind 12 Kataloge pro Regelsatz, in beiden Formen 2000 Einträge je Katalog und 1 MB für eine Katalogdatei.

<a id="items-what-a-party-carries"></a>

## Gegenstände: Was eine Gruppe mit sich führt

Rüstungen, Waffen, Tränke, Ausrüstung, Munition und Geld sind Gegenstände. Ein optionaler `items`-Block deklariert die Begriffe, in denen jeder Gegenstand deines Regelsatzes geschrieben ist, und ein Katalog mit `holds: "items"` listet die Gegenstände selbst auf. Beides braucht Capability API 1.49. Darüber hinaus brauchen: `rarityCaps` und `propose` des Blocks, die die vom Game Master erfundenen Gegenstände regeln, 1.51; ein Wert, der die Gegenstände eines Charakters liest (`itemStat`), 1.52; was ein Gegenstand beim Tragen oder Mitführen an Proben bewirkt, samt den `bonus`-Grenzen des Blocks, 1.53; was ein Gegenstand von seinem Träger verlangt und die Attribute, die er verändert, 1.54; der `attack` einer Waffe 1.55; was ein Gegenstand im Kampf bewirkt, 1.56; was eine Waffe verschießt und geladen hält, 1.57; die Modi einer Waffe, ihr Angriff mit der Nebenhand, ihr Mindestschaden und ihre Zustände bei einem Treffer 1.58; was die Nutzung eines Gegenstands im Kampf bewirkt, samt der Ladungen, die er hält, 1.59; eine Nutzung, die einen Pool wiederherstellt, 1.60; bei einer Rast zurückgewonnene Ladungen oder ein Gegenstand, der zerbricht, wenn er leer ist, 1.61; eine Nutzung, die zuerst eine Probe verlangt (`gate`), 1.62; Beutetabellen und die Beute einer Kreatur 1.63; eine Beutezeile, die Münzen fallen lässt, oder eine Ebene, die Münzen herausnimmt, 1.64; ein Markt, der `sold`-Ort eines Gegenstands und ein `service` 1.65; ein Markttag (eine Kasse, `buyBack`, der `stock` eines Händlers, eine `limited`-Anzahl oder ein `sold` ohne Ort) 1.69; und ein Zusatzeffekt, der bei gehaltenen Waffen auslöst (`weapons`), 1.70.

### Der Gegenstandsblock

Der Block steht in `items` auf der obersten Ebene der Datei. Hier der Block aus Ember Roads, leicht gekürzt:

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

- `categories`: Eine bis 24. Jeder Gegenstand hat genau eine, etwa Rüstung, Waffe oder Trank.
- `rarities`: Optional, höchstens 12. Liste sie von der häufigsten bis zur seltensten auf.
- `tags`: Optional, höchstens 48. Eigenschaften, die ein Gegenstand haben kann, etwa „geworfen“ oder „versilbert“.
- `stats`: Optional, höchstens 24. Die Zahlen und Wörter, die ein Gegenstand trägt, genau wie eine Listenspalte deklariert: `number`, `text`, `boolean`, `enum` oder `dice`, mit Bereich, Werten und Standardwert, sofern der Typ sie hat. `promptVisible` (Standard `true`) legt fest, ob der Game Master den Gegenstandswert neben dem Gegenstand sieht. Setze es auf `false` für etwas, das nur der Spieler sehen soll, etwa wo ein Gegenstand versteckt ist.
- `slots`: Optional, höchstens 12. Wo ein Gegenstand getragen oder gehalten wird und wie viele dieser Trageplätze ein Charakter hat, von 1 bis 20.
- `binding`: Optional. Einstimmung, Investitur oder alles andere, was begrenzt, wie viele Gegenstände ein Charakter gleichzeitig gebunden haben darf. `label` ist die Bezeichnung, und `max` ist ein vom Bogen des Charakters gelesener Wert, etwa `{ "abilityScore": "nerve" }` oder ein abgeleiteter Wert. Er kann den Live-Zustand nicht lesen, denn wie viele Gegenstände ein Charakter binden darf, ändert sich nicht mit jedem Treffer.
- `carry`: Optional. `stat` nennt den Zahlenwert, der das Gewicht eines Gegenstands ist; sein `min` ist 0 oder mehr. `encumberedAbove` gibt an, wie viel ein Charakter tragen kann, bevor er belastet ist, und `limit` (optional), wie viel er überhaupt tragen kann. Beides wird wie `binding.max` vom Bogen gelesen. Ohne `carry` bedeutet Gewicht nichts, und niemand ist je belastet.
- `currencies`: Optional, höchstens sechs Familien mit je einer bis zehn Münzen.
  - Münzen einer Familie werden nach `value` ineinander umgerechnet, gezählt in der kleinsten Münze der Familie. Die kleinste Münze ist also 1 wert, und keine zwei Münzen einer Familie sind gleich viel wert.
  - Zwei Familien werden nie ineinander umgerechnet. Die Münze einer zweiten Nation oder die Gefälligkeiten eines Settings bilden eine eigene Familie.
  - Die ID einer Münze ist über alle Familien hinweg eindeutig, weil die Kosten eines Gegenstands nur die Münze nennen.
  - `perWeight` (optional und nur neben `carry`) gibt an, wie viele Münzen der Familie eine Einheit des Traglastwerts wiegen.
  - Münzen sind das Geld der Gruppe und werden wie jeder Gegenstand in den Taschen getragen (siehe unten „Geld“).
- `rarityCaps`: Optional, höchstens einer je Seltenheit. Das Höchste, was ein vom Game Master erfundener Gegenstand bei dieser Seltenheit geben darf: den größten Wert jedes in `stats` genannten Zahlenwerts, innerhalb des eigenen Bereichs dieses Werts und als ganze Zahl bei einem Wert, der ganze Zahlen nimmt, sowie `bonus` (Capability API 1.53), den größten festen Bonus, den einer seiner Modifikatoren beim Tragen oder Mitführen addieren darf. Bei einer Seltenheit mit `bonus` wird ein Bonus in Würfeln weggelassen, weil sich Würfel nicht auf eine Zahl begrenzen lassen; ein Abzug wird nie begrenzt. Ein erfundener Gegenstand wird darauf begrenzt; die Gegenstände in deinen Katalogen gehören dir und werden nie begrenzt.
- `propose`: Standardmäßig `true`. `false` verbietet dem Game Master, Gegenstände deines Regelsatzes zu erfinden.
- `native`: Standardmäßig `true`. `false` schaltet in den Spielen deines Regelsatzes die eigenen untypisierten Gegenstände von Game Mode ab (siehe **Was Gegenstände liest**). Der Game Master kann weiterhin Gegenstände erfinden, geschrieben in den Begriffen deines Regelsatzes.
- `freeform`: Was aus einem Gegenstand wird, den der Spieler eintippt. `"plain"` (Standard) behält ihn wie bisher als Gegenstand ohne Regeln. `"refuse"` erlaubt nur Gegenstände deines Regelsatzes.
- `lootTables` (Capability API 1.63): Optional, höchstens 24. Was die Kreaturen eines gewonnenen Kampfes und das `[loot:]` des Game Master fallen lassen (siehe unten „Beute“).

### Ein Gegenstand

Ein Gegenstandskatalog deklariert kein `feeds`:

```json
{ "id": "outfitter", "label": "Outfitter", "holds": "items", "entries": [] }
```

Jeder Eintrag trägt ein `item` statt `rows` oder einer `creature`:

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

- `id`, `label`, `summary` und `filters` funktionieren wie bei jedem Eintrag. Die Beschriftung ist der Name des Gegenstands.
- `category` ist erforderlich. `rarity` und `tags` sind optional, und alle drei nennen die eigenen IDs des Blocks. Eine Seltenheit kann ein Gegenstand nur haben, wenn der Block welche deklariert.
- `stats`: Werte für die Gegenstandswerte des Blocks. Jeder muss ein Wert sein, den der jeweilige Gegenstandswert annehmen kann.
- `slots`: wie viele Trageplätze jeder Art der Gegenstand belegt, nie mehr, als ein Charakter hat.
- `stack`: wie viele ein Stapel höchstens fasst, von 1 bis 999.999. Ohne Angabe fasst ein Stapel so viele wie jeder Stapel in Game Mode. Ein Gegenstand mit `charges` bildet immer einen Stapel für sich allein.
- `cost`: ein ganzzahliger `amount` einer Münze, benannt über ihre `unit`-ID. Die Auswahl zeigt ihn, und der Game Master sieht ihn neben dem Gegenstand (siehe unten „Geld“).
- `sold` (Capability API 1.65): `{ "place": "city" }`, der kleinste Ort deines Markts, der den Gegenstand verkauft, mit Vorrang vor jeder Regel, die etwas anderes sagen würde. Mit 1.69 darf es stattdessen oder zusätzlich angeben, wie viele ein Ort an einem Tag vorrätig hat (`"limited": 2` oder `"limited": "1d3"`) (siehe unten „Märkte“).
- `service` (Capability API 1.65): `true` für Unterkunft, Überfahrt, einen Segen: wie ein Gegenstand gekauft, aber nie mitgeführt. Der Kauf bezahlt nur; nichts anderes legt eine Dienstleistung in eine Tasche (ein Hinzufügen wird als `service` abgelehnt), und die Auswahl lässt Dienstleistungen weg. Eine Dienstleistung hat eine `cost` und nichts, was das Tragen oder Nutzen betrifft (`slots`, `stack`, `binds`, `worn`, `carried`, `requires`, `attack`, `use`, `charges`).
- `binds`: Der Gegenstand muss gebunden sein, bevor er beim Tragen etwas bewirkt. `restriction` (optional) beschreibt in Worten, wer ihn binden darf, und `cursed: true` markiert einen, der nicht mehr loslässt. Nur ein Regelsatz mit `binding` kann Gegenstände haben, die gebunden werden.
- `worn` und `carried` (optional, Capability API 1.53): was der Gegenstand an den Proben und Rettungswürfen seines Besitzers bewirkt, solange er getragen wird und solange er nur mitgeführt wird, (1.54) die Attribute, die er setzt oder erhöht, und (1.56) was er im Kampf bewirkt. Siehe [Proben außerhalb eines Kampfes](#checks-outside-a-fight) und [Rüstung und getragene Wirkungen im Kampf](#armor-and-worn-effects-in-a-fight).
- `requires` (optional, Capability API 1.54): was der Gegenstand von seinem Träger verlangt und was gilt, solange dieser die Anforderung nicht erfüllt. Siehe [Proben außerhalb eines Kampfes](#checks-outside-a-fight).
- `attack` (optional, Capability API 1.55): was der Gegenstand als Waffe im Kampf bewirkt, solange er getragen wird, (1.57) was er verschießt und geladen hält und (1.58) seine anderen Modi, ein Angriff mit der Nebenhand, eine Untergrenze für seinen Schaden und die Zustände, die er bei einem Treffer verursacht. Siehe [Waffen im Kampf](#weapons-in-a-fight), [Munition und Nachladen](#ammunition-and-reloading) und [Modi, eine zweite Waffe, eine Untergrenze und Zustände bei einem Treffer](#modes-a-second-weapon-a-floor-and-conditions-on-a-hit).
- `use` und `charges` (optional, Capability API 1.59): was die Nutzung des Gegenstands bewirkt, im Kampf und außerhalb, und die Ladungen, die er für diese Nutzung bereithält, sowie (1.60) ein Pool, den er wiederherstellt. Siehe [Gegenstände im Kampf nutzen](#using-items-in-a-fight) und [Gegenstände außerhalb eines Kampfes nutzen](#using-items-outside-a-fight).

Ein Gegenstand trägt kein `mechanics`: Was er bewirkt, steht in seinem `item`-Block.

### Was Gegenstände liest

Alles oben Genannte wird beim Import des Regelsatzes geprüft, und deine Gegenstandskataloge werden mit ihm ausgeliefert. Im Spiel liest das Inventar von Game Mode sie:

- **Deine Gegenstände liegen in der Tasche.** Ein Stapel kann einer deiner Gegenstände sein und bleibt dieser Gegenstand, wie auch immer der Spieler ihn nennt. Die Schaltfläche **From the ruleset** (aus dem Regelsatz) im Inventar öffnet eine Auswahl aller Gegenstände, die deine Kataloge auflisten, mit Suche und Filtern des Katalogs. Ein Eintrag, den eine Ebene ausblendet, ist auch dort ausgeblendet.
- **Namen finden deine Gegenstände.** Ein Name, den der Spieler eintippt oder den der Game Master in `[inventory: action="add"]` schreibt und der ohne Beachtung der Groß- und Kleinschreibung dem `label` eines deiner Gegenstände entspricht, fügt diesen Gegenstand hinzu. Teilen sich zwei Gegenstände eine Beschriftung, findet ein Name den, den deine Kataloge zuerst auflisten.
- **`stack`** wird eingehalten: Hinzufügen, Setzen, Zusammenführen oder Weitergeben über die Grenze hinaus füllt den Stapel und beginnt einen neuen.
- **`freeform: "refuse"`** lässt dem Spieler nur deine Gegenstände: die Auswahl und Namen, die deine Gegenstände sind.
- **`native: false`** lässt dem Game Master nur deine Gegenstände und die, die er erfindet: Ein Name, der keines von beiden ist, wird abgelehnt (`not-ruleset-item`). Mehr von etwas bereits Vorhandenem kann weiterhin hinzugefügt werden, und seine Anweisungen sagen ihm das. Ein Kampf fragt kein Modell mehr, was die Gegenstände im Inventar bewirken; es kämpfen also nur deine Gegenstände: in einem Regelsatzkampf deine gehaltenen Waffen und die Nutzungen deiner Gegenstände (siehe unten „Waffen im Kampf“), in einem Classic- oder Tactical-Kampf deine Gegenstände mit `use` (siehe unten „Gegenstände in Classic- und Tactical-Kämpfen“). Was der Spieler eintippt, folgt weiterhin `freeform`, und was die Gruppe mit sich führte, kehrt in jedem Fall in einer neuen Sitzung zurück.
- **Was ein Gegenstand ist**, zeigt der ausgewählte Stapel: Kategorie, Seltenheit und Tags mit ihren Beschriftungen, die Werte, die er gibt, seine Zusammenfassung und wie viele ein Stapel fasst. Die Auswahl zeigt außerdem seine `cost`. Der Game Master sieht jeden deiner Gegenstände im Inventar mit Kategorie, Seltenheit, Tags und den Werten, die du `promptVisible` gelassen hast, etwa `Hand axe [Weapon, Common, Thrown; Damage 1d6, Reach close]`.

- **`slots`**: Ein Gegenstand, der Trageplätze belegt, kann von seinem Träger angelegt werden, solange dieser die Plätze frei hat. Aus einem größeren Stapel wird dafür ein Stück in einen eigenen Stapel genommen. Das Inventar zeigt für jeden Charakter die belegten Trageplätze.
- **`binding`**: Ein Gegenstand mit `binds` kann gebunden werden, bis zu `binding.max`, gelesen vom eigenen Bogen seines Trägers (ein Charakter ohne Bogen liest einen leeren). Ein `cursed`-Gegenstand bleibt nach dem Binden gebunden: Der Spieler kann ihn nicht lösen, ablegen, weggeben oder entfernen, und nur der Game Master kann den Fluch beenden.
- **`carry`**: Ein Gegenstand wiegt seinen Wert von `carry.stat` (ein Gegenstand ohne diesen Wert wiegt nichts), und die Last eines Charakters ist das Gewicht seiner Tasche, gemessen an `encumberedAbove` und `limit` von seinem eigenen Bogen. Ein Gegenstand, der in die gemeinsame Ansicht des Inventars kommt – durch den Spieler oder durch den Game Master ohne `who=` –, geht an den, der ihn tragen kann, ohne belastet zu werden (zuerst der Spieler, dann die Gruppe der Reihe nach). Kann niemand alles nehmen, wird nach dem Platz verteilt, der jedem noch bleibt. Nichts geht über das `limit` von irgendjemandem hinaus: Was niemand tragen kann, bleibt zurück, und der Game Master erfährt das. Gib einem Gewichtswert `"integer": false` für Gewichte wie ein Viertelpfund.
- **Der Game Master erfindet Gegenstände in deinen Begriffen.** Sofern du nicht `propose: false` setzt, kann sein `[inventory: action="add"]` einen neuen Gegenstand beschreiben: `like=` einen deiner Gegenstände als Ausgangspunkt, dann beliebige von `category=`, `rarity=`, `tags=`, `stats=` (`id=value`-Paare), `slots=` (`id=count`), `binds=` (`yes`, `cursed` oder `no`), `worn=` und `carried=` (Änderungen, getrennt durch `;`, jeweils `+N`, `-N`, `advantage`, `disadvantage` oder bei Rettungswürfen `fails`, auf Fertigkeiten oder Rettungswürfe nach Namen oder auf `checks` oder `saves` für alle; `+N` oder `-N` auf den Namen eines Attributs, um es zu erhöhen oder zu senken; und, wo du Kämpfe hast, `+N`, `-N`, Vorteil oder Nachteil auf Angriffe und `+N` oder `-N` auf die Verteidigung oder dein Wort dafür: `worn="+1 Sneak; disadvantage on Sway checks"`, `carried="+1 Brawn"`, `worn="+1 Guard"` oder `none`) und `summary=`, jeder Teil per ID oder Beschriftung. Ein Gegenstandswert, den deine Verteidigung bereits zählt (Guard in Ember Roads addiert den `guard` des Getragenen), ist der Weg, auf dem Rüstung sie erhöht. Eine getragene Änderung der Verteidigung neben einem solchen Wert wird deshalb weggelassen, und die Antwort sagt das: Ein kleines Modell schreibt `guard=1` und `+1 Guard` für ein einziges +1. Gelesen wird auch, was ein kleines Modell oft stattdessen schreibt: eine Zahl nach dem Namen (`Brawn +1`), „bonus“ nach Angriffen oder Verteidigung (`+1 attack roll bonus`), `none` für eine leere Liste von Tags, Werten oder Trageplätzen sowie `worn=`, `carried=` oder `summary=` innerhalb von `stats=`, wenn du keinen Wert dieses Namens hast. Die Engine behält nur, was dein Block hat: Eine unbekannte Kategorie, ein unbekanntes Tag, ein unbekannter Wert, Trageplatz, eine unbekannte Fertigkeit oder ein unbekannter Rettungswurf wird weggelassen, eine Seltenheit, die du nicht hast, wird zu deiner niedrigsten, eine Zahl wird auf den Bereich ihres Werts und dann auf `rarityCaps` für ihre Seltenheit begrenzt, ein Bonus beim Tragen oder Mitführen auf den `bonus` ihrer Seltenheit (auch der Teil, den `like` mitgebracht hat), und ein Name, der einer deiner Gegenstände ist, ist einfach dieser Gegenstand. Die Antwort teilt dem Game Master mit, was geändert wurde (nie bei einem Wert, den du ihm nicht zeigst), und die Details des Gegenstands zeigen dem Spieler jede Änderung. Das Spiel behält den Gegenstand, sodass derselbe Name für den Rest des Spiels dieser Gegenstand ist, und eine neue Sitzung behält ihn, solange noch jemand ihn besitzt.
- Der Game Master kann deine Gegenstände mit seinem Inventarbefehl per `equip` und `unequip` anlegen und ablegen, wenn du `slots` hast, und per `bind` und `unbind` binden und lösen, wenn du `binding` hast; er erfährt nur von den Befehlen, die dein Regelsatz hat. Er sieht die Last, die gebundenen Gegenstände und die Trageplätze jedes Charakters sowie, was getragen oder gebunden ist.

- **Was ein Gegenstand beim Tragen oder Mitführen bewirkt**, zeigen der ausgewählte Stapel und die Auswahl („While worn: -1 on checks (Sneak)“), und der Game Master sieht es neben dem Gegenstand (`worn: -1 on checks (Sneak)`). Proben außerhalb eines Kampfes wenden es an (siehe unten „Proben außerhalb eines Kampfes“).
- **Was eine Waffe bewirkt**, wird auf dieselbe Weise angezeigt („Attack (Action): Brawn to hit, 1d6 + Brawn cut damage“), und ein Kampf bietet sie an, solange sie gehalten wird (siehe unten „Waffen im Kampf“).
- **Was die Nutzung eines Gegenstands bewirkt**, wird ebenso angezeigt („Use (Action): heals 1d4 + 1, range 0 paces, used up“), zusammen mit den verbleibenden Ladungen. Ein Kampf bietet sie an, ebenso die Schaltfläche **Use** (nutzen) im Inventar und das `[inventory: action="use"]` des Game Master (siehe unten „Gegenstände im Kampf nutzen“ und „Gegenstände außerhalb eines Kampfes nutzen“).

In einem Classic- oder Tactical-Kampf tut jeder deiner Gegenstände mit `use` dort, was dieses angibt, und einer ohne wird nicht angeboten (siehe unten „Gegenstände in Classic- und Tactical-Kämpfen“). Unabhängig von `native` kann der Bogen deine Gegenstände lesen (siehe unten), eine gehaltene Waffe ist in einem Regelsatzkampf ein Angriff und verschießt, was sie aus der Tasche nimmt, was ein Gegenstand beim Tragen oder Mitführen bewirkt, zählt dort (siehe unten „Rüstung und getragene Wirkungen im Kampf“), ein Gegenstand mit `use` wird nach seinen eigenen Regeln genutzt, im Kampf und außerhalb (siehe unten „Gegenstände im Kampf nutzen“ und „Gegenstände außerhalb eines Kampfes nutzen“), und deine `currencies` sind das Geld der Gruppe (siehe unten „Geld“).

<a id="items-on-the-sheet"></a>

### Gegenstände auf dem Bogen

Ein Wertverweis kann mit `itemStat` (Capability API 1.52) die Gegenstände lesen, die ein Charakter besitzt. Ember Roads addiert den Guard-Wert der Rüstung, die ein Reisender trägt, zu seinem Guard, also zu dem Wert, den ein Treffer übertreffen muss:

```json
{
  "id": "guard",
  "label": "Guard",
  "op": "sum",
  "of": [{ "const": 6 }, { "abilityMod": "wits" }, { "itemStat": { "stat": "guard", "from": "worn", "pick": "sum" } }]
}
```

- `from`: `"worn"` sind die Gegenstände am Charakter: ein Gegenstand, der Trageplätze belegt, solange er angelegt ist, einer mit `binds`, solange er gebunden ist, und einer mit beidem, solange beides gilt. Ein Gegenstand, der keines von beiden tut, wird nie getragen. `"carried"` ist der Rest dessen, was der Charakter besitzt, und `"all"` ist beides.
- `pick`: `"sum"` addiert den Wert jedes Gegenstands mal seiner Anzahl, `"max"` und `"min"` lesen den höchsten oder niedrigsten Einzelwert, und `"count"` zählt die Gegenstände nach ihrer Anzahl.
- `stat` nennt einen deiner Gegenstandswerte. `sum`, `max` und `min` brauchen einen `number`-Wert. `count` darf ihn weglassen, um jeden Gegenstand zu zählen, oder einen beliebigen Wert nennen, um nur die Gegenstände zu zählen, die ihn geben.
- `slot`, `category` und `tag` (jeweils optional) behalten nur die Gegenstände, die diesen Trageplatz belegen, zu dieser Kategorie gehören oder dieses Tag tragen.
- `default` (optional, ohne Angabe 0) ist der gelesene Wert, wenn kein Gegenstand ausgewählt ist oder keiner davon den Wert gibt.

Gegenstände ändern sich im Spiel. `itemStat` unterliegt daher derselben Regel wie ein Live-Lesezugriff: Das `max` eines Pools oder einer Leiste, der Übungsbonus, `binding.max`, die `carry`-Zahlen und die skalierte Spalte oder Skalierung eines Katalogs können es nicht lesen, auch nicht über einen abgeleiteten Wert. Überall sonst, wo eine Zahl gelesen wird, geht es: in einem abgeleiteten Wert, im `adjust` einer Probe, im `cap` einer Fertigkeit sowie bei Verteidigung, Schadensminderung oder Initiative eines Kampfes.

Die Gegenstände eines Charakters sind die in seiner eigenen Tasche. Die Karte des Spielers (die Karte mit dem Namen, unter dem im Chat gespielt wird, sonst die erste) liest die Tasche des Spielers, jede andere Karte ihre eigene. Nur die Gegenstände deines Regelsatzes zählen, weil ein einfacher Gegenstand keine Werte zum Lesen hat. Der Bogen auf dem Bildschirm, eine Probe, der Bogenblock des Game Master und ein Kampf bei seinem Beginn lesen alle, was jeder Charakter genau in diesem Moment besitzt. Außerhalb eines Spiels, im Bogeneditor oder in einer Importprüfung, besitzt niemand etwas, und `itemStat` liest seinen `default`.

<a id="checks-outside-a-fight"></a>

### Proben außerhalb eines Kampfes

Eine Probe oder ein Rettungswurf, den der Game Master außerhalb eines Kampfes verlangt, liest mehr als den Bogen: die eigenen aktiven Zustände des Charakters, die Stufen, die er erreicht hat, was seine Gegenstände beim Tragen oder Mitführen bewirken und was die Anforderung eines getragenen Gegenstands kostet, solange er sie nicht erfüllt. Es ist dasselbe Vokabular, das die Zustände eines Kampfes verwenden (Capability API 1.53 für die Gegenstandsschlüssel und die Eingrenzung).

Ein Gegenstand gibt mit `worn` an, was er bewirkt, solange er angelegt (und, wo er gebunden wird, gebunden) ist, und mit `carried`, was er bewirkt, solange er nur mitgeführt wird. Der Ledermantel aus Ember Roads knarrt beim Schleichen, und ein Wegstein bringt Karawanenleute dazu, dir zu vertrauen:

```json
"worn": { "modifiers": [{ "to": "checks", "skills": ["sneak"], "flat": -1 }] }
```

```json
"carried": { "modifiers": [{ "to": "checks", "skills": ["sway"], "flat": 1 }] }
```

Beide nehmen die Teile eines Zustands auf, die eine Probe liest:

- `effects`: `own-checks-advantage`, `own-checks-disadvantage`, `own-saves-advantage` und `own-saves-disadvantage`.
- `modifiers`: Änderungen mit `to` an Proben oder Rettungswürfen, mit `flat`, `dice` (und `minus`), `mode` und eigenen `skills` oder `saves`, genau wie bei einem Zustand.
- `failsSaves`: Rettungswürfe, die der Besitzer ohne Wurf nicht schafft.
- `skills` und `saves`: welche Fertigkeiten oder Rettungswürfe die Wirkungen und die Modifikatoren betreffen, die selbst keine nennen.
- `abilities` (Capability API 1.54): Attribute, die er verändert, per ID, jeweils `{ "set": N }` (mindestens N: ein höherer Wert bleibt) oder `{ "add": N }` (eine negative Zahl senkt es). Sie gelten, bevor der Bogen berechnet wird. Alles, was das Attribut liest, liest also das veränderte: der Bogen auf dem Bildschirm, abgeleitete Werte, Proben, der Bogenblock des Game Master und ein Kampf bei seinem Beginn. Zuerst werden die Additionen angewendet, dann das höchste `set`, und `min` und `max` des Attributs gelten weiter. Das Maximum eines Pools oder einer Leiste und der Übungsbonus werden allein aus dem Bogen berechnet; ein Gegenstand verschiebt sie also nicht.

Die Ochsenleder-Handschuhe aus Ember Roads verleihen einem schwachen Griff den eines Viehtreibers:

```json
"worn": { "abilities": { "brawn": { "set": 2 } } }
```

Was ein Gegenstand im Kampf bewirkt (Verteidigung, Angriffe, Geschwindigkeit, die übrigen Kampfwirkungen sowie der Schaden und die Zustände, die er abhält), steht unter [Rüstung und getragene Wirkungen im Kampf](#armor-and-worn-effects-in-a-fight); eine Probe außerhalb eines Kampfes liest nur die oben genannten Teile. Ein Gegenstand gilt einmal, egal wie viele Stapel davon jemand besitzt.

Ein Gegenstand kann von seinem Träger mit `requires` auch etwas verlangen: einen Wert von dessen Bogen (`value`, ein beliebiger Wertverweis, gelesen wie alles im Spiel), das Mindestmaß (`atLeast`) und was gilt, solange der Träger es nicht erreicht (`otherwise`, im selben Vokabular, nur dass es kein Attribut verändern kann, weil die Anforderung eines lesen kann). Bis zu vier. Der Grabspaten aus Gravewatch ist schwere Arbeit für einen schwachen Wächter:

```json
"requires": [
  {
    "value": { "abilityScore": "sinew" },
    "atLeast": 3,
    "otherwise": { "modifiers": [{ "to": "checks", "skills": ["dig"], "flat": -1 }] }
  }
]
```

Eine Anforderung liest das Attribut so, wie die Gegenstände des Trägers es hinterlassen. Handschuhe, die Sinew auf 3 setzen, erfüllen also die des Spatens.

Bei einer Probe gilt:

- Jeder Modifikator, der sie betrifft, addiert seine Zahl, und seine Würfel werden geworfen: eine feste Zahl bei einer Summenprobe, Würfel bei einem Pool. Eine Attributsprobe und eine Probe, die der Regelsatz nicht benennen kann, lesen nur, was nicht eingegrenzt ist, und ein Rettungswurf liest nur, was Rettungswürfe betrifft.
- Vorteil und Nachteil aus allen Quellen, darunter das `mode=` des Game Master, heben sich auf: Ist von beidem etwas da, wird die Probe einmal gewürfelt. Nur ein Regelsatz, der überhaupt zweimal würfelt (`resolution.advantage`), neigt sich in eine Richtung.
- Ein Rettungswurf, den ein Zustand oder Gegenstand scheitern lässt, scheitert ohne Wurf, und dafür wird nichts ausgegeben.
- Der Datensatz nennt, was die Probe verändert hat: `effects="-1"` für die Zahl, `from="Leather coat"` für das, was sie verändert hat, und `automatic="true"` für einen ohne Wurf gescheiterten Rettungswurf. Die Würfelkarte zeigt dasselbe, und der Game Master liest es im nächsten Zug. Ihm wird gesagt, dass die Engine diese Änderungen anwendet, damit er sie nicht noch einmal addiert.

Gravewatch zeigt dasselbe bei einem Pool: Die gebundene Dawn bell fügt Ward einen Würfel hinzu, und Rattled nimmt Soothe und Barter je einen weg.

<a id="weapons-in-a-fight"></a>

### Waffen im Kampf

Ein Gegenstand kann eine Waffe sein (Capability API 1.55). Sein `attack` hat die Form der [Angriffszeilen](#combat-a-fight-your-own-rules-resolve) eines Kampfblocks, mit Werten statt Spalten, und ein [Regelsatzkampf](#combat-a-fight-your-own-rules-resolve) bietet ihn als Angriff an, solange der Gegenstand getragen wird: in den Händen gehalten, die er belegt, und gebunden, wo er gebunden wird. Ein weggesteckter oder nur mitgeführter Gegenstand bietet nichts an, und die Angriffszeilen und die eigenen Aktionen einer Kreatur funktionieren genau wie bisher. Die Handaxt aus Ember Roads:

```json
"attack": {
  "budget": "act",
  "toHit": { "abilities": { "stat": "swing" } },
  "damage": { "dice": { "stat": "damage" }, "abilities": { "stat": "swing" }, "type": "cut" },
  "reach": 2,
  "range": { "normal": 10, "long": 20 }
}
```

- `budget`: das Budget, das er verbraucht, eines aus `combat.economy.budgets`.
- `toHit`: was er zum Treffen addiert. `abilities` sind eine oder mehrere Attributs-IDs, und die beste davon zählt (eine Finesse-Waffe listet zwei). `skill` addiert, was eine Probe dieser Fertigkeit addiert, mit dem Attribut des Angriffs statt dem der Fertigkeit, wie bei einer Angriffszeile. `proficiency` ist ein Wert vom Bogen des Trägers: Liegt er über 0, wird der Übungsbonus des Regelsatzes addiert (`{ "const": 1 }` für immer). `bonus` ist eine Zahl. `target` ist der eigene Zielwert je Würfel eines Pool-Kampfes für den Angriff dieser Waffe (der Spaten aus Gravewatch zählt jeden Würfel ab 6 statt ab der 7 des Pools), und nur ein `dice-pool`-Regelsatz mit veränderlichem `target` darf einen angeben; in einem Summenkampf drückt sein `bonus` dasselbe aus.
- `damage`: die `dice`, die er verursacht, das beste seiner Schadens-`abilities` addiert, ein `bonus` und ein `type`. Ein Summenkampf braucht `dice`; in einem Pool-Kampf verursacht ein Treffer seine Erfolge, und `dice` fügt entsprechend viele weitere Würfel vom Würfeltyp des Pools hinzu.
- `reach` und `range` (`normal` und `long` für das, was darüber hinaus noch reicht) sind in deiner `combat.distance`-Einheit angegeben und brauchen eine solche. Eine Waffe mit beidem wird geworfen, wie eine Angriffszeile: ein Schlag aus der Nähe, ein Wurf darüber hinaus. Ohne beides reicht sie eine Zelle weit.
- `versatile`: die `dice`, die sie stattdessen verursacht, solange in jedem Trageplatz, den sie belegt, unter dem, was ihr Träger trägt, noch einmal so viel Platz frei ist: ein Speer in einer Hand, während die andere frei ist.
- `strikes`: wie viele Angriffe ein Verbrauch kauft, gelesen vom Bogen des Trägers, wie das `strikes` einer Angriffszeile. Eine Waffe ohne diese Angabe ist ein Angriff pro Verbrauch, egal wie viele Angriffe ihr Träger hat.

Jede dieser Zahlen und Wörter kann stattdessen einen eigenen Wert des Gegenstands lesen: `{ "stat": "damage" }` ist der `damage`-Wert des Gegenstands, und ein Wert, für den der Gegenstand nichts angibt, gilt, als wäre die Angabe nicht geschrieben. `abilities` und `skill` lesen einen Enum-Wert, dessen Wörter Attributs- oder Fertigkeits-IDs sind, und `type` einen Text- oder Enum-Wert. Ein Gegenstand, den der Game Master `like=` einer Waffe erfindet, ist also ebenfalls eine Waffe, kämpft mit seinen eigenen Werten und wird wie jeder Wert auf `rarityCaps` begrenzt (ein ausdrücklich geschriebener Wert wird so übernommen, wie er ist). Einer, der ohne Trageplatz und ohne Bindung erfunden wird, wird nie getragen und entsteht deshalb ohne Angriff. Ein kleines Modell beschreibt oft eine Waffe und lässt `like=` weg. Ein Gegenstand, der in einer Kategorie deiner Waffen ohne Ausgangspunkt erfunden wird, kämpft deshalb wie deine Waffe dieser Kategorie, der er dem Namen nach am ähnlichsten ist (ein in beide Richtungen gemeinsames Wort, ein „crossbow“ ist also wie ein „bow“), oder wie die erste davon. Er übernimmt jeden Wert, den dieser Angriff liest und den der Vorschlag ausgelassen hat, und die Antwort sagt das („It fights as Hand axe does.“). Das Vorschlagsformular des Game Master sagt ihm außerdem, dass eine nach dem Vorbild einer Waffe erfundene Waffe auch wie diese kämpft.

Die Tags einer Waffe gehen mit jedem ihrer Treffer mit. Der `resist`- oder `immune`-Eintrag einer Kreatur kann angeben, was hindurchkommt: Mit `{ "type": "tearing", "except": ["silver"] }` erleidet die Kreatur den Schaden einer Waffe mit dem Tag `silver` voll und widersteht allem anderen. Der Grabwiedergänger aus Gravewatch ist so geschrieben, und der silberne Sargnagel des Regelsatzes kommt hindurch.

Die Details der Waffe und die Zeile für den Game Master nennen, was sie tut: `attack (Action): Brawn to hit, 1d6 + Brawn cut, reach 2 paces, range 10 to 20 paces`, mit `1d8 with a hand free` für den Eberspieß aus Ember Roads und `at 6` neben dem Spaten aus Gravewatch.

<a id="ammunition-and-reloading"></a>

#### Munition und Nachladen

Eine Waffe kann etwas verschießen und eine eigene geladene Anzahl führen (Capability API 1.57). Der Jagdbogen aus Ember Roads verschießt Pfeile, und die Wachpistole aus Gravewatch hält eine Kugel, geladen aus dem Kugel- und Pulvervorrat des Wächters:

```json
"ammo": { "tag": "arrow", "recover": 0.5 }
```

```json
"ammo": { "tag": "shot" },
"clip": { "max": 1, "reload": "act" }
```

- `ammo`: was die Waffe verschießt, benannt über eines deiner Gegenstands-`tags`. Jeder Angriff nimmt `perAttack` (1 ohne Angabe) der Gegenstände mit diesem Tag, die ihr Träger mitführt, getragen oder nicht, aus dem ersten solchen Stapel in seiner Tasche, dann aus dem nächsten. Die Waffe wird nur angeboten, solange er genug davon mitführt, und das Kampfmenü nennt, wie viele übrig sind. Nach einem Kampf, den die Gruppe gewinnt, kehrt der Anteil `recover` (von 0 bis 1) dessen, was jeder verschossen hat, zu dem Stapel zurück, aus dem es kam. Der Anteil wird über den ganzen Kampf zusammengezählt und für jeden Stapel einmal abgerundet; die Hälfte von fünf aus einem Köcher verschossenen Pfeilen ist also zwei. Ein verlorener Kampf oder einer, aus dem die Gruppe geflohen ist, gibt nichts zurück.
- `clip`: eine geladene Anzahl, die die Waffe selbst führt. `max` gibt an, wie viel sie fasst, ausdrücklich angegeben oder aus einem Zahlenwert des Gegenstands gelesen, und `reload` ist das Budget, das ein Nachladen verbraucht. Ein Angriff verbraucht dann das Geladene, statt aus der Tasche zu nehmen, und die Waffe wird nur angeboten, solange sie genug für einen Angriff enthält. Eine Option **Reload** (nachladen) im Kampfmenü verbraucht `reload` und füllt sie: aus der Tasche, wenn sie `ammo` hat (so viel, wie die Tasche noch enthält, falls das weniger ist), und vollständig, wenn sie keine hat. Die Anzahl wird am Inventarstapel der Waffe gespeichert; eine in einem Kampf leer geschossene Pistole ist also auch im nächsten leer. Eine Waffe, die noch niemand abgefeuert hat, ist geladen, ebenso eine, die ein Spieler in einen Stapel mit mehreren davon legt, denn eine geladene Anzahl gehört zu einer einzelnen Waffe. Die Munition eines Magazins wird nach einem Kampf nicht aufgesammelt; `recover` ist also für eine Waffe ohne Magazin gedacht.

Was ein Kampf verschießt, lädt und aufsammelt, wird bei jedem Schritt ins Inventar geschrieben, so wie die Gesundheit der Gruppe, und das Journal nennt, was verbraucht wurde. Ein Schritt, dessen Stapel verschwunden ist oder weniger enthält, als der Kampf angenommen hat, wird abgelehnt, genau wie ein verbrauchter Gegenstand, den der Kampf nicht findet. Das Kampfprotokoll nennt jeden Schuss, jedes Nachladen und jedes Aufsammeln („Hunting bow: noch 2 Schuss.“, „Ada lädt 1 in Watch pistol: 1 von 1 geladen.“), und die Details des Gegenstands sowie die Zeile für den Game Master nennen, was die Waffe verschießt und fasst: `ammunition Arrow (1 an attack, 50% picked up after a won fight)` und `holds 1, reload (Act)`.

<a id="modes-a-second-weapon-a-floor-and-conditions-on-a-hit"></a>

#### Modi, eine zweite Waffe, eine Untergrenze und Zustände bei einem Treffer

Eine Waffe kann weitere Arten haben, ihren Angriff auszuführen, einen Partner in der Nebenhand, einen Mindestschaden und Zustände, die sie dem Getroffenen zufügt (Capability API 1.58). Der Jagdbogen aus Ember Roads kann eine Salve abgeben, und der silberne Sargnagel aus Gravewatch kann in jeder Hand gehalten werden und markiert, was er tief trifft:

```json
"modes": [{ "id": "volley", "label": "Volley", "ammo": 2, "toHit": -2, "targets": 2 }]
```

```json
"offHand": true,
"onHit": [{ "condition": "marked", "atLeast": 2, "rounds": 2 }]
```

- `modes`: bis zu sechs weitere Arten, den Angriff auszuführen, jede mit `id`, `label` und dem, was sie ändert. `ammo` gibt an, wie viel ein Angriff in diesem Modus verschießt; die Waffe braucht also `ammo` oder einen `clip`, aus dem sie schießt. `toHit` ist, was er zum Treffen addiert: Würfel in einem Pool-Kampf, eine Zahl in einem Summenkampf. `target` verschiebt den Zielwert je Würfel eines Pool-Kampfes um diesen Betrag (ausgehend vom eigenen der Waffe, sonst von dem des Pools; `+1` ist schwerer), sofern der Zielwert des Pools veränderlich ist. `targets` gibt an, auf wie viele Ziele er gerichtet werden darf, jedes mit eigenem Angriffswurf. Sobald der Angriff ausgewählt ist (nach seinem Initiativestil, wo es Stile gibt), fragt das Kampfmenü nach der Art, jeweils mit dem erwarteten Ergebnis, und bietet nur die Modi an, für die der Träger die Schüsse hat. Ein von der Engine gespieltes Gruppenmitglied wägt jeden auf ein Ziel gerichteten Modus ab und überlässt einen Modus für mehrere Ziele dem Spieler. In einem Fenster wird ein Angriff unverändert ausgeführt.
- `offHand`: eine Waffe für die Nebenhand. Der Kampfblock nennt das Budget, das ein zweiter Angriff verbraucht, und ob dessen Schaden ein positives Attribut behält:

  ```json
  "offHand": { "budget": "quick", "ability": "full" }
  ```

  Hat ihr Träger in diesem Zug mit einer als `offHand` markierten Waffe angegriffen, bietet jede andere solche Waffe, die er trägt, einen zweiten Angriff mit diesem Budget an, im Menü „Silver coffin nail, off hand“ genannt. `ability` ist `full` (Standard) oder `penalty-only`; bei Letzterem wird das Schadensattribut nur addiert, wenn es etwas abzieht, wie beim Kampf mit zwei Waffen in 5e. Ein Angriff mit der Nebenhand ist ein einzelner Schlag, egal wie viele Angriffe der Hauptangriff kauft, und nie ein Angriff auf jemanden, der sich entfernt. Eine als `offHand` markierte Waffe braucht das `offHand` des Kampfblocks.
- `floor`: das Mindeste, was ein Treffer verursacht, ausdrücklich angegeben oder aus einem Zahlenwert des Gegenstands gelesen. Der Schaden eines Pool-Kampfes nach der Schadensminderung oder der Schaden eines Summenkampfes wird darauf angehoben, bevor eine Resistenz ihn halbiert, und das Protokoll sagt das. Ein Schlag im ausgebenden Stil wirft die Zahl seines Urhebers und wird nie angehoben.
- `onHit`: bis zu vier deiner Zustände, die das Ziel erhält, wenn der Schaden des Schlags `atLeast` erreicht hat, für `rounds` seiner eigenen Züge oder, ohne diese Angabe, bis etwas sie entfernt. Gemeint ist der Schaden, der nach Schadensminderung und Resistenzen ankam; ein Schlag im nehmenden Stil nimmt Initiative statt Schaden und verursacht daher keinen.

Die Details der Waffe und die Zeile für den Game Master nennen alles davon: `modes Volley (2 shots, -2 to hit, up to 2 targets)`, `off hand (Quick)`, `at least 1 on a hit before resistance` und `Marked for 2 rounds when a hit deals 2 or more`.

<a id="armor-and-worn-effects-in-a-fight"></a>

### Rüstung und getragene Wirkungen im Kampf

Was ein Gegenstand beim Tragen oder Mitführen bewirkt, kann auch einen Kampf verändern (Capability API 1.56), und zwar für seinen Besitzer, neben dessen Zuständen und Stufen. Beide nehmen die Teile eines Zustands auf, die ein Kampf liest:

- `effects`: alle Wirkungen eines Zustands außer den vier, die eine Stufe nicht haben kann (`half-move-to-stand`, `ends-on-damage`, `cannot-target-source`, `cannot-approach-source`), weil niemand dem Besitzer den Gegenstand auferlegt hat und er nie von selbst endet: `own-attacks-advantage`, `attacks-against-disadvantage`, `resist-all`, `speed-zero` und die übrigen.
- `modifiers`: Änderungen mit `to` an Verteidigung, Angriffen und Geschwindigkeit sowie an Proben und Rettungswürfen, genau wie bei einem Zustand; in einem `dice-pool`-Kampf ist eine Änderung an Angriffen eine feste Zahl von Würfeln, wie bei einem Zustand.
- `resist`, `vulnerable` und `immune`: Schadensarten, von denen der Besitzer die Hälfte, das Doppelte oder nichts erleidet, wie bei einer Kreatur (und gegen `combat.damageTypes` geprüft, wenn du welche deklarierst). `conditionImmunities`: deine eigenen Zustände, die dem Besitzer nie auferlegt werden.

Der verfluchte Witwenring aus Gravewatch kostet bei jedem Angriff einen Würfel, die gebundene Dawn bell schützt ihren Träger davor, Rattled zu werden, und ein mitgeführter Wegstein aus Ember Roads hält die Hitze ab:

```json
"worn": { "modifiers": [{ "to": "attacks", "flat": -1 }] }
```

```json
"worn": { "modifiers": [{ "to": "checks", "skills": ["ward"], "flat": 1 }], "conditionImmunities": ["rattled"] }
```

```json
"carried": { "modifiers": [{ "to": "checks", "skills": ["sway"], "flat": 1 }], "resist": ["burn"] }
```

Ein Kampf liest jeden Gegenstand einmal, benannt nach dem Stapel („Widow's ring“ neben einem Wurf im Protokoll und bei einer Verteidigung), so wie eine Probe. Das `otherwise` einer unerfüllten Anforderung zählt auch im Kampf; eine für ihren Träger zu schwere Rüstung kann also Geschwindigkeit kosten. Die Details eines Gegenstands und die Zeile für den Game Master nennen alles davon („While worn: -1 on attacks“, „worn: +1 on checks (Ward), immune to Rattled“). Rüstung, die einfach zu einer Verteidigung oder einer Schadensminderung addiert, ist ein Gegenstandswert, der mit `itemStat` gelesen wird (siehe „Gegenstände auf dem Bogen“), so wie Guard in Ember Roads den getragenen Mantel liest.

**Härte.** Wo Initiative eine Zahl ist, die Angriffe verschieben, ist `combat.pool.hardness` die Härte eines Kämpfers, zu Kampfbeginn von seinem Bogen gelesen (die seiner Rüstung, über `itemStat`), und eine Kreatur gibt ihre eigene `hardness` an. Ein Schlag im ausgebenden Stil, dessen Würfel darunter liegen, trifft und bewirkt nichts: Die Zahl des Angreifers fällt trotzdem auf die Basis zurück, das Protokoll sagt das („Adas Grave spade trifft den Wächter mit 5 Würfeln, unter einer Härte von 6, und bewirkt nichts.“), und das Menü sagt dafür keinen Schaden voraus. Nur ein Regelsatz mit einem ausgebenden Stil darf sie haben, und eine als Bogen geschriebene Kreatur nimmt sie aus dem Bogen.

<a id="using-items-in-a-fight"></a>

### Gegenstände im Kampf nutzen

Ein Gegenstand kann im Kampf genutzt werden (Capability API 1.59): ein auf eine Schnittwunde gedrückter Umschlag, ein geschluckter Trank, eine geläutete Glocke. Sein `use` beschreibt, was die Nutzung bewirkt, in den Begriffen, die der `mechanics`-Block eines Katalogeintrags verwendet (siehe [Was ein Kampf aus `mechanics` liest](#what-a-fight-reads-from-mechanics)): `kind` (`heal`, `attack`, `buff` oder `debuff`), `amount`, `damageType`, `plus`, `attackRoll`, `save`, `applies`, `temporary`, `range`, `area`, `targets`, `targetCount` und `friendlyFire`, mit dem `budget`, das sie verbraucht, oder `free`. Der Umschlag aus Ember Roads sowie der wärmende Trank und die Morgenglocke aus Gravewatch:

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

- Eine Nutzung steht im Kampfmenü unter **Items** (Gegenstände) und ist nach dem Gegenstand benannt („Poultice“), egal wie der Stapel heißt. Ein Gegenstand, der Trageplätze belegt oder gebunden wird, wird genutzt, solange er getragen (und, wo er gebunden wird, gebunden) ist, wie die Glocke; jeder andere, solange er mitgeführt wird.
- Sie verbraucht ein Budget oder ist `free` und bezahlt mit sich selbst: `cost`, `perCostStep`, `check`, `concentration`, `reaction`, `scales`, `gives`, `standard` und `rider` sind für einen Katalogeintrag auf einem Bogen gedacht und werden bei einer Nutzung abgelehnt.
- `toHit` bei einer Nutzung mit `attackRoll` ist, was sie zum Treffen addiert, so wie das `toHit` einer Waffe es angibt (siehe „Waffen im Kampf“): Attribute, eine Fertigkeit, ein Bonus und in einem Pool-Kampf ein Zielwert je Würfel. Ohne diese Angabe addiert der Wurf nichts.
- `saveDifficulty`: die Zahl, gegen die ein verlangter Rettungswurf gewürfelt wird (der eigene der Nutzung, einer, der einen auferlegten Zustand beendet, oder der eines Schadensbestandteils ohne `difficulty`), ausdrücklich angegeben oder aus einem Zahlenwert des Gegenstands gelesen. Ein Eintrag auf einem Bogen liest diese Zahl aus der Quelle seines Katalogs; ein Gegenstand hat keine und gibt sie deshalb selbst an.
- `consumes: true` nimmt bei jeder Nutzung ein Stück vom Stapel des Gegenstands, und das Menü nennt, wie viele übrig sind. Das zuletzt genutzte Stück nimmt den Stapel mit.
- `restore` (Capability API 1.60) gibt jedem Bogen, auf den die Nutzung wirkt, einen Teil eines Pools zurück, so wie ein Blutbeutel Blut zurückgibt: `{ "pool": "resolve", "amount": { "flat": 1 } }`, mit `dice` und `flat` wie bei einem `amount`. Es nennt einen deiner laufenden Pools, nie den Gesundheitspool (Gesundheit kehrt mit einer Heilung zurück), und steht an einer helfenden Nutzung, einem `heal` oder einem `buff`. Eine Kreatur hat keine Pools und erhält daher nichts. Das Protokoll sagt es („Ada erhält 1 Resolve zurück und steht bei 3 von 4.“), und ein von der Engine gespieltes Gruppenmitglied lässt einen vollen Pool in Ruhe. Der wärmende Trank aus Gravewatch stellt neben seiner Heilung einen Punkt Resolve wieder her.
- `charges` verbraucht so viele der eigenen `charges` des Gegenstands, dessen `max` angibt, wie viele er fasst, von 1 bis 100, ausdrücklich angegeben oder aus einem Zahlenwert des Gegenstands gelesen (ein Gegenstand, der diesen Wert unter 1 angibt, wird abgelehnt, und einer über 100 fasst 100). Die Anzahl wird am Inventarstapel des Gegenstands gespeichert; eine Glocke, die in einem Kampf zweimal geläutet wurde, hat im nächsten also noch eine Ladung, und eine, die noch niemand geläutet hat, ist voll. Ladungen gehören zu einem einzelnen Gegenstand; ein Gegenstand mit Ladungen hat daher einen `stack` von 1. Eine Nutzung verbraucht den Gegenstand oder Ladungen, nie beides, und die Ladungen eines Gegenstands werden immer von seiner Nutzung verbraucht. Zwei weitere Schlüssel gehören zu `charges` (Capability API 1.61), wie bei der Morgenglocke aus Gravewatch:

  ```json
  "charges": { "max": 3, "recharge": { "rests": ["vigil"], "amount": "max" }, "breaksOn": { "die": 20, "atMost": 1 } }
  ```

  - `recharge`: welche deiner `rests` die Ladungen zurückbringen und wie viele: `"max"` füllt den Gegenstand, und eine Menge (`{ "dice": "1d6", "flat": 1 }`) gibt so viele zurück, nie über `max` hinaus. Eine Rast bringt die Ladungen jedes Gegenstands zurück, den ihr Charakter mitführt und der sie nennt, egal ob die Rast-Schaltfläche des Bogens sie auslöst oder das `[sheet: op="rest"]` des Game Master. Bogen und Tasche werden gemeinsam geschrieben, und eine neu generierte Antwort lädt nur einmal auf. Der Bogen nennt nach der Rast, was zurückkam („Dawn bell 3/3 charges“), und der Game Master sieht neben jedem Gegenstand im Inventar die verbleibenden Ladungen („2 of 3 charges left“).
  - `breaksOn`: Verbraucht eine Nutzung die letzte Ladung, würfelt die Engine einen d`die`, und bei höchstens `atMost` zerbricht der Gegenstand und verschwindet aus der Tasche, im Kampf und außerhalb. Das Kampfprotokoll sagt das („Dawn bell zerbricht (eine 1 auf ihrem Würfel).“), der Bericht der Schaltfläche **Use** ebenfalls („Mit der letzten Ladung zerbricht sie.“), und das Journal zählt den Gegenstand als verloren.
- `gate` (Capability API 1.62) ist eine Probe, die der Nutzer bestehen muss, bevor der Gegenstand wirkt, so wie eine Schriftrolle mit einem Zauber über dem eigenen Niveau des Lesers eine verlangt. `check` nennt, was gewürfelt wird: `{ "skill": "ward" }`, `{ "ability": "wits" }` oder einen Wert vom Bogen (`{ "value": { "derived": "spell_mod" } }`, für eine Zahl, die sich von Charakter zu Charakter unterscheidet, wie der Zauberwirkungsmodifikator eines 5e-Zauberwirkers). `difficulty` reicht von 1 bis 100, ausdrücklich angegeben oder aus einem Zahlenwert des Gegenstands gelesen (ein Gegenstand, der ihn nicht angibt, fehlt im Menü, und seine Schaltfläche **Use** sagt das deutlich). `unless` überspringt die Probe, wenn ein Wert auf dem Bogen des Nutzers mindestens `atLeast` beträgt. Eine misslungene Probe verbraucht den Gegenstand (oder seine Ladungen) für nichts. Im Kampf wird die Probe gewürfelt, sobald der Gegenstand verbraucht wird, mit den eigenen Würfeln des Kampfes (einem Pool in einem Pool-Kampf, wo `difficulty` die nötigen Erfolge angibt) und der zu Kampfbeginn gelesenen Zahl vom Bogen. Was die Zustände und getragenen Gegenstände des Nutzers zu dieser Probe beitragen, zählt mit, einschließlich dessen, was nur für die Fertigkeit gilt. Das Protokoll sagt es („Ada würfelt Ward, um Page of the vigil litany zu nutzen: 2 Erfolge aus 2 Würfeln bei 7 oder mehr (8, 9), 2 Erfolge nötig, ein Erfolg.“). Die Seite der Vigil-Litanei aus Gravewatch hat eine solche Probe:

  ```json
  "gate": { "check": { "skill": "ward" }, "difficulty": 2, "unless": { "value": { "abilityScore": "nerve" }, "atLeast": 3 } }
  ```

Was ein Kampf verbraucht und welche Ladungen er übrig lässt, wird bei jedem Schritt ins Inventar geschrieben, wie die Schüsse einer Waffe, und das Journal nennt, was verbraucht wurde. Das Kampfprotokoll nennt jede Nutzung („Poultice: 1 von 2 übrig.“, „Dawn bell: 2 von 3 übrig.“). Ein von der Engine gespieltes Gruppenmitglied setzt eine Heilung nur bei Verletzten ein, und niemand nutzt einen Gegenstand für einen Angriff auf jemanden, der sich entfernt. Die Details des Gegenstands und die Zeile für den Game Master nennen, was seine Nutzung bewirkt: „Use (Action): heals 1d4 + 1, range 0 paces, used up“ und `use (Act): Steel 7 save negates it, Rattled, 1 of 3 charges`, und die Details zeigen die verbleibenden Ladungen. Ein Gegenstand, den der Game Master `like=` einem deiner Gegenstände erfindet, wird wie dieser genutzt, samt Ladungen.

### Gegenstände in Classic- und Tactical-Kämpfen

Ein Regelsatz, der seine Kämpfe nicht selbst auflöst, überlässt sie den eigenen Classic- und Tactical-Kämpfen von Game Mode, in denen zu Kampfbeginn ein Modell gefragt wird, was die Gegenstände im Inventar bewirken. Deine Gegenstände werden nicht geraten: Jeder mit `use` tut, was dort steht, mit den eigenen Zahlen der Engine, und die Einschätzung des Game Master wird nur für Gegenstände erfragt, die nicht deine sind (und bei `native: false` für keine).

- Ein `heal` heilt und ein `attack` verletzt um einen Anteil der maximalen Gesundheit des Ziels, ausgehend vom Durchschnitt seines `amount`: Ein Durchschnitt von 7, die `1d8+3` einer einfachen Waffe, ist etwas mehr als ein Fünftel davon, nie weniger als ein Zwanzigstel und höchstens alles. Auf dieser Skala liest die Kampfanbindung die Zahlen eines Katalogeintrags. Der `damageType` ist das Element.
- Der erste Zustand in `applies` wird als Status unter seinem eigenen Namen auferlegt, für seine `rounds` (ohne Angabe 2), und ein `buff` oder `debuff` ist allein dieser Status, der die Verteidigung erhöht oder senkt, wie die eigenen Status der Engine.
- `targets` gibt an, auf wen die Nutzung zielt. Ohne Angabe geht eine Heilung oder Verstärkung an einen Freund und ein Angriff oder eine Schwächung an einen Gegner.
- `consumes: true` nimmt ein Stück vom Stapel, und eine Nutzung ohne diese Angabe lässt den Gegenstand, wo er ist.
- Ein Angriffswurf, ein Rettungswurf, eine Fläche, temporäre Punkte und ein wiederhergestellter Pool haben in diesen Kämpfen keinen Platz, wie die `mechanics` eines Katalogeintrags dort: Die Beschreibung des Gegenstands im Menü nennt sie trotzdem.
- Ein Gegenstand, der Trageplätze belegt, wird nur genutzt, solange er getragen wird, und einer, der gebunden wird, nur, solange er gebunden ist, wie in einem Regelsatzkampf. Ein verbrauchter Gegenstand wird aus dem Getragenen genommen, nie aus einem Ersatzstück in der Tasche.
- Ein Gegenstand, dessen Nutzung `charges` verbraucht, wird angeboten, solange eine Nutzung übrig ist, und das Menü **Items** zählt seine Nutzungen. Jede Nutzung verbraucht sie aus dem Stapel des Gegenstands, zuerst aus dem eigenen des Spielers, und die zuletzt verbrauchte würfelt ihr `breaksOn`: Zerbrochen verschwindet der Gegenstand aus der Tasche, und das Journal zählt ihn als verloren.
- Ein Gegenstand, der zuerst eine Probe verlangt (`gate`), lässt sie auf dem Server würfeln, wenn ein Gruppenmitglied ihn nutzt, mit dem Bogen dieses Mitglieds und dem, was es trägt (ein Begleiter ohne eigene Karte würfelt auf einem leeren Bogen, und nur die eigene Einheit des Spielers greift auf die Karte des Spielers zurück), so wie die Schaltfläche **Use** würfelt; trifft `unless` zu, wird sie übersprungen. Misslingt sie, ist der Gegenstand verbraucht (oder seine Ladungen sind es) und bewirkt nichts, und das Protokoll sagt das: „Ada würfelt Ward, um Page of the vigil litany zu nutzen: 0 gegen 2, misslungen, und der Gegenstand ist umsonst verbraucht.“
- Ein Gegenstand ohne `use` wird in diesen Kämpfen nicht angeboten, ebenso wenig eine Heilung ohne `amount`.

Die Engine berechnet die Wirkung jedes Gegenstands selbst; was der Bildschirm sendet, entscheidet also nie, was einer deiner Gegenstände bewirkt.

### Beute

Ein gewonnener Kampf lässt in jedem Game-Mode-Spiel einmal Beute fallen, direkt in die Taschen der Gruppe. Ohne Regelsatz stammt sie aus den eigenen Tabellen von Game Mode (mehr und seltener bei höherer Schwierigkeit), ebenso bei einem Regelsatz, der keine Beutetabellen deklariert und `native` eingeschaltet lässt. Ein Regelsatz, der Beutetabellen deklariert, lässt stattdessen seine eigenen Gegenstände fallen, und einer mit `native: false` und ohne Tabellen lässt nichts fallen.

`lootTables` im `items`-Block (Capability API 1.63) listet sie auf, wie die Grabbeigaben aus Gravewatch:

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

- `rolls` gibt an, wie viele Ziehungen die Tabelle macht: eine Zahl von 0 bis 20 oder Würfel (`"1d2"`), standardmäßig 1.
- Jede Ziehung wählt eine Zeile nach ihrem `weight` (standardmäßig 1) im Verhältnis zu den anderen. Eine Zeile nennt einen deiner Gegenstände als `<catalog>/<entry>`, einen `filter` nach `rarity`, `category` und `tag`, der gleichmäßig unter allen passenden Gegenständen wählt, oder (Capability API 1.64) `coins`, eine deiner Münzen per ID. `count` gibt an, wie viele fallen, eine Zahl oder Würfel, standardmäßig 1. Ein Gegenstand oder eine Münze, die eine Ebene herausnimmt, fällt nicht, ebenso wenig ein Filter, der nichts findet.
- Eine Bestiariumkreatur nennt mit `loot` die Tabelle, die sie trägt (siehe unten „Kreaturen“). Ein gewonnener Regelsatzkampf würfelt die Tabelle jeder besiegten Kreatur. Ein Kampf, den die Engine nicht nach deinen Regeln auflöst, hat keine Bestiariumkreaturen und lässt daher nichts fallen.
- Was fällt, kommt wie ein Hinzufügen in die gemeinsame Ansicht, wobei zuerst die Tasche des Spielers gefragt wird, und was niemand tragen kann, bleibt zurück. Die Zusammenfassung für den Game Master nennt, was gefallen ist und dass es schon in den Taschen liegt, das Journal sagt das, und eine Benachrichtigung zeigt es an.
- Der Game Master würfelt eine Tabelle in der Geschichte mit `[loot: table="grave_goods" who="Ada"]` (`who=` für die Tasche eines einzelnen Charakters), gemessen ab dem Beginn des Zuges wie seine Inventar-Tags, sodass eine neu generierte Antwort nur einmal Beute fallen lässt. Jeder fallende Gegenstand wird wie ein Hinzufügen beantwortet, und seine Anweisungen listen deine Tabellen auf.

### Geld

Deine `currencies` sind das Geld der Gruppe. Eine Münze ist ein Gegenstand in den Taschen, ein Stapel jeder Münze wie jeder andere, benannt nach ihrem `label`:

- Sie wiegt je `perWeight` Münzen ihrer Familie eine Einheit deines Traglastwerts, zählt zur Last ihres Trägers und wird beim Eingang nach der Tragregel verteilt. Eine Familie ohne `perWeight` wiegt nichts.
- Der Spieler fügt Münzen über die Liste **Coins** (Münzen) der Auswahl hinzu und teilt, gibt und führt sie zusammen wie jeden Stapel.
- Eine Zeile über den Stapeln des Inventars zeigt die sichtbaren Münzen jeder Familie und ihren Wert in der kleinsten Münze der Familie. Der Game Master sieht diesen Wert neben jeder Tasche (`Coin worth 432 pennies` oder `no coins` für einen leeren Geldbeutel, wobei die Tasche des Spielers auch dann aufgeführt wird, wenn sie nichts enthält) und die `cost` eines Gegenstands neben dem Gegenstand (`costs 3 shillings`).
- Der Game Master bezahlt mit `[inventory: action="pay" amount="3 shillings" who="Ada"]` und wird mit `action="earn"` bezahlt. `amount` ist eine Anzahl und eine Münze, per ID oder Beschriftung, Einzahl oder Mehrzahl, ohne Beachtung der Groß- und Kleinschreibung (`1 penny`, `12 pennies`). Eine Zahlung kommt aus einer einzigen Tasche (ohne `who=` aus der des Spielers) und nur in der eigenen Familie der Münze: Zuerst gehen die größten passenden Münzen, dann wird die kleinste verbleibende Münze gewechselt, die den Restbetrag deckt, und das Wechselgeld kommt in den kleineren Münzen der Familie zurück, die größten zuerst. Die Antwort nennt, was bezahlt wurde und was zurückkam: 3 shillings aus einem Geldbeutel mit einer einzigen crown ergeben `Paid with crowns ×1; shillings ×2 back.` Eine Zahlung, die die Tasche nicht leisten kann, wird abgelehnt (`cannot-afford`), ebenso eine Münze, die du nicht hast (`unknown-coin`), und jede Zahlung in einem Regelsatz ohne Währungen (`no-currencies`). Ein Verdienst ist ein Hinzufügen dieser Münze, in die Tasche von `who` oder nach der Tragregel verteilt. Das Journal listet Ausgaben und Einnahmen auf, und eine Benachrichtigung zeigt sie an. Die Anweisungen des Game Master listen deine Münzen auf; diese Befehle erhält nur ein Regelsatz mit Münzen.
- Kaufen ist eine Zahlung und anschließend ein Hinzufügen oder, in einem Regelsatz mit Markt, ein `buy` (siehe unten). Einen Ladenbildschirm gibt es nicht.

### Märkte

Ein `market` im `items`-Block (Capability API 1.65) legt fest, was ein Ort zu welchem Preis verkauft, statt dass jeder Laden alles zu seinem Listenpreis verkauft. Der Markt aus Gravewatch, leicht gekürzt:

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

- `prices`: eine bis acht Preisstufen, jede ein Multiplikator (`times`, über 0) auf die `cost` eines Gegenstands, und eine davon ist der `default`. Ein Preis ist die Kosten in der kleinsten Münze der Familie mal der Stufe, auf diese Münze gerundet (nie unter eine Münze für etwas, das überhaupt etwas kostet). Angegeben wird er in der Münze, in der die Kosten geschrieben sind, sofern diese ihn genau bezahlt, sonst in der größten Münze, die deine Ebenen übrig lassen und die ihn genau bezahlt: Bei `dear` kostet ein Mantel für 8 shillings 12 shillings, und ein 5e-Trank für 50 gp kostet 50 gold pieces, nie 5 platinum. Feilschen oder die Laune eines Händlers verschieben die Stufe, nie die Zahl.
- `places`: eine bis zwölf Ortsgrößen, die kleinste zuerst, in deinen eigenen Worten.
- `sold`: Regeln für den kleinsten Ort, der verkauft, was ein Filter auswählt. Ein Filter nennt `rarity`, `category` und `tag`, beliebig viele davon, und passt auf einen Gegenstand, der alles Genannte hat; die erste Regel, die auf einen Gegenstand passt, entscheidet. Das eigene `sold` eines Gegenstands hat Vorrang vor den Regeln, und ein Gegenstand, den nichts nennt, wird überall verkauft.
- `sellers`: Arten von Händlern, jede mit dem, was sie verkauft (`sells`: Filter, von denen jeder einen Gegenstand auswählen kann), dem kleinsten Ort (`place`), an dem es sie gibt (ohne Angabe überall), und optional mit `only`, an wen allein sie verkauft: ein Wert vom Bogen des Käufers, einschließlich seines Live-Zustands und seiner mitgeführten Gegenstände, mindestens `atLeast`, dem Game Master als `label` genannt. Ohne `sellers` verkauft jeder an einem Ort, was dieser Ort verkauft.
- Ein Gegenstand ohne `cost` ist unverkäuflich, ebenso einer, den eine Ebene ausblendet, oder einer, dessen Münzen eine Ebene herausgenommen hat.

**Wo die Gruppe ist.** Der Game Master nennt mit `[place: name="Millbrook" size="market town"]` den Ort einer Szene und seine Größe, die Größe per ID oder Beschriftung, und lässt die Größe für einen Ort ohne Markt weg (eine Straße, die Wildnis). Die Engine beantwortet jedes Tag direkt und lehnt eine Größe ab, die du nicht hast. Der zuletzt in den für den Spieler sichtbaren Antworten des Game Master beantwortete Ort, gezählt ab dem jüngsten Gesprächsbeginn, gilt, bis ein anderer genannt wird. Ein Orts-Tag, das ein Spieler eintippt, zählt nie. Eine neu generierte Antwort liest nur, was vor der Angabe kam, die sie ersetzt.

**Kaufen.** Der Game Master kauft mit `[inventory: action="buy" item="Hand axe" count="1" level="dear" seller="smith" who="Ada"]`; `level` (ohne Angabe die Standardstufe) und `seller` (ohne Angabe jeder Händler hier, der den Gegenstand verkauft) sind optional. Die Engine berechnet den Preis an dem Ort, der beim Ende der Antwort gilt, nimmt ihn wie bei einer Zahlung samt Wechselgeld aus dem Geldbeutel des Käufers (ohne `who=` aus dem des Spielers) und legt den Gegenstand in seine Tasche, oder sie bezahlt bei einer Dienstleistung nur. Entweder geschieht beides oder nichts. Die Antwort nennt, was es gekostet hat (`price="12 shillings"`) und was bezahlt und zurückgegeben wurde, und eine Benachrichtigung zeigt es an. Ein Kauf wird abgelehnt und ändert nichts, wenn kein Ort genannt ist (`no-place`), bei einer Stufe, einem Gegenstand oder Händler, den du nicht hast (`unknown-level`, `unknown-item`), ohne Preis (`not-for-sale`), bei einem für den Gegenstand zu kleinen Ort (`not-here`), wenn hier kein Händler ihn verkauft (`no-seller`), bei einem Händler, dessen `only` der Käufer nicht erfüllt (`not-to-you`), bei einem Geldbeutel, der den Preis nicht decken kann (`cannot-afford`), oder bei einer Tasche, die den Gegenstand nicht tragen kann.

**Was der Game Master sieht.** Solange eine Szene an einem Ort mit Markt spielt, nennt ein MARKET-Block den Ort und seine Größe, die Preisstufen und jeden Händler dort mit einem Dutzend seiner Waren zur Standardstufe, die günstigsten zuerst; alles, was sie verkaufen, kann per Name gekauft werden. Seine Anweisungen bringen dem Game Master `[place:]` und `buy` bei, und nur ein Regelsatz mit Markt hat sie.

#### Ein Markttag

Ein Markttag (Capability API 1.69) lässt die Regale eines Ortes wechseln und leer werden und erlaubt der Gruppe, Gegenstände zurückzuverkaufen. Der Markttag aus Gravewatch, leicht gekürzt:

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

- **Der Tag.** Ein Tag an einem Ort dauert, bis der Game Master einen anderen Ort nennt. Einen neuen Tag am selben Ort beginnt er, indem er ihn mit `day="new"` erneut nennt (`[place: name="Millbrook" size="market town" day="new"]`). Alles Folgende wird je Tag aus den Antworten des Game Master berechnet, die der Spieler sieht. Ein Swipe, eine neu generierte Antwort oder eine Verzweigung braucht also nichts Gespeichertes, und ein Tag würfelt immer dasselbe Regal.
- **`limited`** an einer `sold`-Regel oder am eigenen `sold` eines Gegenstands: wie viele ein Ort an einem Tag vorrätig hat, eine Zahl oder Würfel, die für den Tag gewürfelt werden. Eine Regel darf ein Limit ohne Ort nennen (überall verkauft, einige wenige auf einmal); der Ort eines Gegenstands stammt aus der ersten Regel, die einen Ort nennt, und sein Limit aus der ersten, die ein Limit nennt. Sind sie verkauft, wird ein Kauf bis zu einem neuen Tag abgelehnt (`sold-out`).
- **`stock`** an einem Händler: eine deiner Beutetabellen, einmal am Tag gewürfelt für das, was dieser Händler zusätzlich zu seinem Sortiment vorrätig hat. Was sie ergibt (nur Gegenstände, nie Münzen), verkauft allein dieser Händler, so viele wie gewürfelt, unabhängig von `sells` und `sold`; das `only` eines Händlers gilt weiterhin.
- **`buyBack`**: der Anteil (über 0, höchstens 1) am Standardpreis eines Gegenstands, den ein Händler dafür zahlt. Ein Händler hier, der diese Art von Ware verkauft, kauft sie an (ohne Händler jeder). Ohne `buyBack` kauft niemand etwas zurück.
- **`purse`** an einer Ortsgröße: das Höchste, was die Händler dort an einem Tag zusammen auszahlen, in einer deiner Münzen. Verkäufe, die in der Familie dieser Münze bezahlt werden, werden darauf angerechnet; ohne Kasse kaufen die Händler unbegrenzt an.
- **Verkaufen.** Der Game Master verkauft mit `[inventory: action="sell" item="Silver coffin nail" count="1" seller="smith" who="Ada"]`. Die Engine berechnet den Preis, nimmt den Gegenstand aus der Tasche des Verkäufers (ohne `who=` aus der des Spielers; nie einen gebundenen verfluchten Gegenstand) und zahlt die Münzen ein, die größten zuerst, oder lehnt ab und ändert nichts: kein Ort genannt, hier kauft niemand ihn an (`not-bought`), kein Preis (`not-for-sale`), die heutige Kasse ist erschöpft (`purse-spent`), keiner vorhanden oder Münzen, die die Tasche nicht tragen kann. Die Antwort nennt den Erlös (`price="4 shillings"`), das Journal führt den Gegenstand als verkauft, und eine Benachrichtigung zeigt es an.
- Der MARKET-Block ergänzt, was jeder Händler heute vorrätig hat, wie viele von einem begrenzten Gegenstand übrig sind, was die Händler hier zahlen und wie viel von der heutigen Kasse übrig ist; seine Anweisungen bringen dem Game Master `sell` und `day="new"` bei.

<a id="using-items-outside-a-fight"></a>

### Gegenstände außerhalb eines Kampfes nutzen

Dasselbe `use` funktioniert auch außerhalb eines Kampfes (Capability API 1.60 für `restore`; der Rest braucht nichts Neues). Die Schaltfläche **Use** im Inventar bei einem deiner Gegenstände mit `use` und das `[inventory: action="use" item="Poultice" who="Juno"]` des Game Master bewirken beide mit den Würfeln der Engine, was der Gegenstand bei dem tut, der ihn mitführt:

- Eine Heilung gibt Gesundheit zurück: die Menge bei einem Gesundheitspool oder eine entfernte Markierung bei einer Wundleiste, wie eine Heilung im Kampf. `temporary`-Punkte kommen auf einen Gesundheitspool, `restore` gibt seinen Pool zurück, und jeder Zustand, den die Nutzung mit `applies` auferlegt, wird gesetzt, bis etwas ihn entfernt.
- Eine Nutzung, die schadet oder auf die andere Seite zielt (`kind` `attack` oder `debuff` oder `targets` `enemy`), trifft niemanden: Außerhalb eines Kampfes steht niemand auf dem Brett, den man treffen könnte. Der Gegenstand wird trotzdem genutzt, und der Game Master erfährt, was er bewirkt, damit er es erzählen kann.
- Sie nimmt ein Stück vom Stapel oder verbraucht seine Ladungen genau wie ein Kampf. Ein Gegenstand ohne verbleibende Nutzung oder einer, der einen Trageplatz belegt oder gebunden wird und nicht getragen wird, wird abgelehnt und ändert nichts. Bogen und Tasche werden gemeinsam geschrieben.
- Ein `gate` wird zuerst gewürfelt, wie eine Probe: mit den eigenen Würfeln des Regelsatzes und der Zahl vom Bogen, dem Wundabzug des Nutzers und dem, was seine Zustände und getragenen oder mitgeführten Gegenstände an dieser Probe bewirken. Eine misslungene Probe verbraucht den Gegenstand für nichts, und die Zeile sagt das („Ada nutzt Page of the vigil litany: Ward-Probe 1 gegen 2, misslungen; der Gegenstand ist umsonst verbraucht. Noch 1 übrig.“).

Was geschehen ist, steht in einer Zeile: „Juno nutzt Poultice: heilt 4 (Grit 6/11). Noch 1 übrig.“ Die Schaltfläche **Use** schickt sie dem Game Master in einem `[item_used]`-Block nach „I use my Poultice.“, den der Chat als Badge anzeigt. Das eigene Tag des Game Master wird damit beantwortet, gemessen ab dem Beginn des Zuges wie seine anderen Tags, sodass eine neu generierte Antwort einen Gegenstand nie zweimal nutzt. Ein Gegenstand ohne `use` oder in einem Spiel ohne Regelsatz wird weiterhin einfach genannt: „I use my rope.“

<a id="battles-lending-the-sheet-to-marinaras-combat"></a>

## Kämpfe: Den Bogen Marinaras Kampfsystem bereitstellen

Standardmäßig weiß ein Kampf nichts vom Bogen. Er erstellt seine Kämpfer wie bisher. Ein Charakter kann einen Kampf daher verlassen, ohne dass sich die Trefferpunkte auf seinem Bogen verändert haben.

Ein optionaler `battle`-Block ändert das in genau einer Hinsicht: Er stellt dem Kampf die Zahlen des Bogens bereit und schreibt das Kampfergebnis zurück. **Dadurch folgt der Kampf nicht deinen Regeln.** Würfelberechnung, Treffer und Schadenshöhe bestimmt weiterhin Marinara. Deshalb wird Gesundheit als Anteil des Maximums übertragen, nicht als deine eigene Zahl. Ein auf dem Bogen halb gesunder Charakter beginnt den Kampf mit der Hälfte der von Marinara erzeugten Gesundheitsleiste. Dein Gesundheitspool von 9 Punkten wird nie direkt in einen Kampf übernommen, in dem ein Treffer 12 Schaden verursacht.

```json
"battle": {
  "health": { "pool": "grit" },
  "energy": { "pool": "luck" },
  "skills": [{ "list": "knacks" }]
}
```

- `health`: Erforderlich. Der laufende Pool für die Trefferpunkte des Charakters im Kampf. Er muss zu `sheet.live.pools` gehören und darf keine Liste mit Pool-Zeilen sein.
- `energy`: Optional. Ein vom Kampf verbrauchbarer laufender Pool, der zur MP-Leiste wird. Er muss sich von `health` unterscheiden, weil der Kampf keine Trefferpunkte als Energie verbrauchen kann.
- `slots`: Optional. Laufende Pools, aus denen ein Kampf jeweils einen Platz verbraucht, mit einer `level`-Angabe von 1 bis 9: `[{ "pool": "slots_1", "level": 1 }]`. Jede Stufe und jeder Pool dürfen nur einmal vorkommen.
- `skills`: Optional, höchstens acht. Die Bogenlisten, deren Zeilen zu Kampffertigkeiten werden. Es zählen nur Zeilen aus einem deiner Kataloge und nur dann, wenn ihr ursprünglicher Eintrag einen `mechanics`-Block hat. Eine von Hand eingegebene Zeile beschreibt keine Wirkung in Zahlen. `onlyWhen` nennt eine gesetzte boolesche Spalte, etwa für einen vorbereiteten Zauber. `alwaysWhen` nennt eine Spalte samt Wert, die eine Zeile trotzdem zulassen, etwa Zauber, die ohne Vorbereitung gewirkt werden. Das ist eine Ausnahme zu `onlyWhen` und wird ohne diese danebenstehende Angabe abgelehnt.

### Was in den Kampf hinein- und aus ihm herausgetragen wird

**Hinein**, für jedes Gruppenmitglied mit im Spiel vorhandenem Bogen: Der Anteil des Gesundheitspools an seinem Maximum bestimmt die Ausgangsposition auf Marinaras Gesundheitsleiste. Der Energiepool wird zu MP, jeder Platzpool zu den Plätzen seiner Stufe und die markierten Zeilen zu Fertigkeiten. Maximale Trefferpunkte, Angriff, Verteidigung, Geschwindigkeit und Stufe bleiben Marinaras eigene Zahlen. Ein Charakter mit null im Gesundheitspool beginnt am Boden, weil der Bogen das vorgibt. Ein Charakter über null beginnt nie mit weniger als einem Trefferpunkt, damit ein kleiner Anteil niemanden durch Rundung außer Gefecht setzt.

**Heraus**, sobald der Kampf endet: Der verbliebene Anteil der Gesundheitsleiste wird auf die eigene Skala des Gesundheitspools zurückgerechnet. Die Differenz zum Kampfbeginn wird als Schaden oder Heilung angewendet. Energie und Plätze sind Anzahlen statt Anteile und werden unverändert zurückgeschrieben. Alles folgt denselben Regeln wie die Schaltflächen des Bogens. Eine vom Bogen abgelehnte Änderung wird übersprungen und gemeldet, nicht erzwungen. Hat ein Kampf die Trefferpunkte eines Kämpfers nicht verändert, schreibt er überhaupt keine Gesundheitsänderung. Die beiden Umrechnungen können deshalb nicht von selbst einen Bogen verändern.

**In keiner Richtung**: Angriffswürfe, Rettungswürfe, Konzentration und die Zusatzwirkung höherer Kosten. Diese Angaben stehen im `mechanics`-Block für ein echtes Kampfsystem, das sie eines Tages liest. Diese Verbindung wendet sie nicht an; ein Regelsatz sollte das auch nicht behaupten.

Ein abgebrochener Kampf schreibt nichts zurück. Löschst du die Nachricht, in der er begonnen hat, oder erreicht der Kampf sein Ende nicht, bleibt der Bogen genau wie zuvor: Der Kampf hat nicht stattgefunden.

### Wie aus einem Eintrag eine Fertigkeit wird

Der `mechanics`-Block eines Katalogeintrags wird so gelesen:

- `kind` wird zum Typ der Fertigkeit. `utility`-Einträge und alles mit `reaction` entfallen, weil Marinaras Kampfsystem dafür keinen Platz hat.
- `amount` bestimmt die Stärke als Multiplikator des eigenen Angriffs des Kämpfers, nicht als Schadenszahl. Größere Würfel treffen niemals schwächer. Der Multiplikator bleibt im Bereich, den bereits generierte Fertigkeiten verwenden.
- `range` und `area.size` werden durch `units.distance.perCell` des Katalogs geteilt, um Rasterzellen zu erhalten. Sie werden nie auf null abgerundet. Eine Explosion verwendet ihren Radius, ein Kegel dessen Hälfte, eine Linie eine Zelle. Flächeneffekte treffen alle abgedeckten Gegner und beachten `friendlyFire`.
- `damageType` wird zum Element der Fertigkeit. `targets` wird nicht übertragen: Marinaras Kampfsystem entscheidet anhand des Fertigkeitstyps, auf wen Heilung, Verstärkung oder Angriff zielen dürfen.
- `cost` auf dem Energiepool wird zu MP-Kosten; mehrere Energiekosten werden addiert. `cost` von genau einem Zauberplatz verbraucht einen Platz dieser Stufe. Marinaras Kampfsystem berechnet eine Energiemenge oder einen Platz, nie beides. Einträge mit Kosten von zwei Plätzen, Plätzen zweier Stufen oder einem Platz plus Energie werden deshalb im Kampf ausgelassen. Dasselbe gilt für Kosten auf anderen Pools, etwa Trefferpunkten oder Klassenressourcen, weil die Engine sie sonst kostenlos gewähren würde.
- `buff` oder `debuff` wird zu Marinaras eigener Verstärkung oder Schwächung. Andere im Eintragstext versprochene Wirkungen, etwa das Entfernen eines Zustands auf dem Bogen, werden im Kampf nicht angewendet. Lass `mechanics` bei Einträgen weg, deren Wirkung nur außerhalb eines Kampfes sinnvoll ist.

`coverage.combat` bleibt davon getrennt und behält seine Bedeutung: Setze es nur, wenn Kämpfe tatsächlich deinen Systemregeln folgen.

<a id="combat-a-fight-your-own-rules-resolve"></a>

## Kampfregeln: Ein Kampf, den deine eigenen Regeln auflösen

Der obige `battle`-Block stellt einem Kampf die Zahlen des Bogens bereit, während Marinara weiter rechnet. Der optionale `combat`-Block beschreibt dagegen, wie deine Regeln einen Kampf AUFLÖSEN. Er parametrisiert eine von der Engine bereitgestellte Kampfart, genauso wie `resolution` eine Probenart parametrisiert. Jeder Name darin stammt von dir. Es gibt zwei Kampfarten: `attack-vs-defense`, bei der Würfel addiert und mit einer Zahl verglichen werden, und `dice-pool`, bei der ein Kampf die eigenen Pools deines `dice-pool`-Regelsatzes wirft und Erfolge zählt (siehe unten „Ein Kampf mit Würfelpools“).

**Ein Spiel mit einem Regelsatz, der `combat` deklariert, kämpft nach deinem Block.** Die Werte der Gruppe kommen von ihren eigenen Bögen, die Gegner aus deinem Bestiarium oder deiner Bedrohungsskala. Jeder Zug wird mit deinen Würfeln aufgelöst. Alles, was ein Charakter verbraucht oder verliert, wird unmittelbar auf seinen Bogen zurückgeschrieben; das Schließen des Tabs mitten im Kampf verliert also nichts. Der Kampfbildschirm spielt das in deinen Begriffen aus: deine Angriffe und Fähigkeiten im Menü, deine Budgets, deine Zustände und ein Protokoll mit der tatsächlichen Rechnung. Was noch fehlt, steht unter "Noch nicht umgesetzt".

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

Das ist der vollständige Ember-Roads-Block, nach dem ein Spiel mit Ember Roads kämpft. Der 5e-Entwurf verwendet dieselben Schlüssel für ein d20-System:

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

### Alle Schlüssel

- `kind`: `"attack-vs-defense"` (eine Seite würfelt gegen die Verteidigung der anderen; ein Treffer verursacht Schaden) oder `"dice-pool"` (siehe „Ein Kampf mit Würfelpools“). Jeder folgende Schlüssel bedeutet bei beiden dasselbe und wird so gelesen, wie die jeweilige Art Zahlen liest.
- `health`: Erforderlich. Was der Kampf abzieht. Entweder `{ "pool": "grit" }`, ein laufender Pool, den er verringert und dessen etwaiger temporärer Puffer zuerst Schaden aufnimmt, oder `{ "track": "harm" }`, eine Wundleiste, die er MARKIERT. Eine Leiste braucht daneben `damageKinds` und gewährt keine temporären Punkte.
- `defense`: Erforderlicher Wertverweis. Ein vom Spieler eingegebenes Feld oder ein von dir berechneter abgeleiteter Wert.
- `initiative`: Erforderlich. Die zu Beginn einmal gewürfelten Würfel und ein optionaler Modifikatorverweis. Bei Gleichstand gewinnt der höhere Modifikator, danach die Reihenfolge beim Aufstellen des Kampfes. `"each": "round"` würfelt die Initiative aller zu Beginn jeder neuen Runde erneut, mit dem Modifikator, wie er dann steht (ein Modifikator, der eine Wundleiste liest, ist bei Verwundung also langsamer), und die Runde beginnt bei dem, der in der neuen Reihenfolge vorn liegt. Capability API 1.47. Ein `dice-pool`-Kampf kann die Initiative stattdessen als Pool werfen: `pool` (ein Wertverweis) gibt die Zahl der Würfel an, und deren Erfolge plus `plus` ergeben die Zahl, ohne `dice` oder `modifier` daneben. `resource`, das `pool` braucht, behält diese Zahl und lässt Angriffe sie verschieben (siehe unten „Initiative, die Angriffe verschieben“). Capability API 1.48.
- `attackRoll`: Erforderlich bei `attack-vs-defense` und abgelehnt bei `dice-pool`, das die eigenen Pools deiner Auflösung wirft. Die Würfel; ob zweimal gewürfelt und ein Wurf behalten wird (`advantage`); die Wirkung der äußersten Seiten eines einzelnen Würfels (`naturals.max`: `critical`, `hit` oder `none`; `naturals.min`: `miss` oder `none`); und die Wirkung eines kritischen Treffers auf den Schaden (`critical`: `double-dice` würfelt die Schadenswürfel erneut, `max-dice` addiert ihre höchsten Augenzahlen einmal, `none` ist ein gewöhnlicher Treffer). Besondere Augenzahlen setzen wie bei Proben einen einzelnen Würfel voraus. Rettungswürfe im Kampf verwenden dieselben Würfel.
- `economy`: Erforderlich. `budgets` beschreibt, was ein Zug enthalten darf: ID, Beschriftung, `per` (`turn` füllt zu Beginn des eigenen Zuges auf, `round` zu Beginn einer neuen Runde) und `count`. Das ZUERST deklarierte Budget ist das Hauptbudget; es bezahlt Standardaktionen. `movement` ist ein optionaler Wertverweis für die Bewegungsweite eines Zuges in deiner eigenen Entfernungseinheit. Er wird bei Kämpfen auf einem Brett gelesen; siehe Positionen.
- `attacks`: Optional. Bogenlisten, deren Zeilen Waffen sind. `name` nennt die Textspalte für den Zeilennamen, `damage.dice` die Würfelspalte. `toHit.ability`, `toHit.proficiency`, `toHit.bonus`, `damage.ability`, `damage.bonus` und `damage.type` nennen jeweils eine Spalte derselben Liste. Eine `ability`-Spalte ist ein `enum` mit einer deiner Attributs-IDs; andere Werte addieren nichts. Eine `proficiency`-Spalte ist ein `boolean`; ist sie gesetzt, wird dein Übungsbonus addiert. Eine Zeile ohne lesbare Würfel ist kein Angriff. Ein Seil in derselben Liste bleibt also einfach ein Seil. `toHit.skill` nennt eine `enum`-Spalte mit einer deiner Fertigkeits-IDs: Die Zeile addiert, was eine Probe dieser Fertigkeit addiert, und nennt die Zeile ein eigenes `ability`, tritt dieses an die Stelle des Attributs der Fertigkeit, genau wie `with=` es bei einer Probe austauscht (Capability API 1.47). `strikes` ist ein optionaler Wertverweis, der angibt, wie viele Angriffe EIN Verbrauch des Listenbudgets kauft. Wählt der Charakter eine Zeile, während er keine Angriffe in Reserve hat, wird das Budget verbraucht und der Rest vorgemerkt. Solange Angriffe vorgemerkt sind, kostet jede Zeile mit `strikes` überhaupt kein Budget. Andere Waffen, andere Ziele und Bewegung dazwischen ergeben sich so von selbst aus dem Menü. `strikesCappedBy` nennt eine boolesche Spalte, die IHRE EIGENE Zeile auf einen einzelnen Angriff begrenzt, egal wie viele die Liste kauft. Das ist für Waffen gedacht, die pro Zug nur einmal schießen, unabhängig von der Angriffsanzahl ihres Trägers; dafür steht die Loading-Eigenschaft von SRD 5.1. Bei einer Liste, die ohnehin nur einen Angriff pro Budgetverbrauch kauft, wäre das bedeutungslos und wird abgelehnt. Die vorgemerkten Angriffe gehören dem KÄMPFER, nicht einer Liste. Deklarieren zwei Waffenlisten eines Charakters beide `strikes`, greifen ihre Zeilen auf denselben Vorrat zu. Er wird am Ende des Zuges gelöscht, der ihn gekauft hat. Ohne Angabe kauft die Liste einen Angriff pro Budgetverbrauch, wie alle Kämpfe vor dieser Erweiterung.

  ```json
  {
    "list": "attacks",
    "budget": "action",
    "name": "name",
    "strikes": { "field": "attacks_per_action" },
    "damage": { "dice": { "column": "damage" } }
  }
  ```

- `abilities`: Optional. Bogenlisten, deren katalogmarkierte Zeilen Fähigkeiten sind, mit denselben Filtern `onlyWhen` und `alwaysWhen` wie `battle.skills`. Die Wirkung bestimmt `mechanics` des jeweiligen Eintrags. Der Block legt das standardmäßig verbrauchte `budget`, den addierten `toHit` eines Angriffswurfs und die `saveDifficulty` für Rettungswürfe gegen den Eintrag fest. Ein Eintrag mit einem eigenen Rettungswurf oder einem zum Beenden eines angewendeten Zustands wird abgelehnt, wenn seine Zielliste keine `saveDifficulty` hat: Ein Rettungswurf gegen nichts wäre immer erfolgreich.
- `standard`: Optional, aus der geschlossenen Liste `dash`, `disengage`, `dodge`, `help`, `hide`, `ready`. `dodge` (Angriffe gegen den Ausweichenden werden zweimal gewürfelt, der schlechtere Wurf zählt) und `help` (der nächste Angriff des unterstützten Verbündeten wird zweimal gewürfelt, der bessere zählt) werden immer aufgelöst. `dash` (noch einmal dieselbe Bewegungsweite) und `disengage` (in diesem Zug greift dich niemand für das Weggehen an) werden auf einem Brett aufgelöst und ohne Brett nur protokolliert. `hide` und `ready` werden angenommen, bewirken aber noch nichts.
- `standardEffects`: Optional, für den Teil einer Standardaktion, den ihr Kennzeichen nicht ausdrückt. Derzeit gibt es das nur für `dodge`: `{ "dodge": { "saves": ["dex_save"] } }` nennt die Rettungswürfe, die ein ausweichender Charakter für die Dauer des Ausweichens zweimal würfelt, wobei der bessere zählt. Nenne nur auf dem Bogen deklarierte Rettungswürfe und nur dann, wenn `standard` auch `dodge` enthält. Ohne diese Angabe bleibt Ausweichen wie bisher: schwerer zu treffen und sonst nichts.
- `conditions`: Optional. Bildet DEINE Zustands-IDs auf ihre Wirkungen ab. Bogen und Kampf verwenden dadurch dieselben Zustände, und ein vergifteter Charakter bleibt danach vergiftet. Die Wirkungsliste ist geschlossen: `own-attacks-advantage`, `own-attacks-disadvantage`, `attacks-against-advantage`, `attacks-against-disadvantage`, `attacks-against-adjacent-advantage`, `attacks-against-far-disadvantage`, `attacks-from-adjacent-critical`, `cannot-act`, `cannot-react`, `speed-zero`, `half-move-to-stand`, `ends-on-damage`, `own-saves-advantage`, `own-saves-disadvantage`, `own-checks-advantage`, `own-checks-disadvantage`, `resist-all`, `cannot-target-source` und `cannot-approach-source`. `failsSaves` nennt Rettungswürfe, die der Zustand ohne Wurf scheitern lässt. Die sechs Wirkungen mit Entfernungs- oder Bewegungsbezug (`attacks-against-adjacent-advantage`, `attacks-against-far-disadvantage`, `attacks-from-adjacent-critical`, `speed-zero`, `half-move-to-stand`, `cannot-approach-source`) werden auf einem Brett gelesen und sagen ohne Brett nichts aus; siehe Positionen. `cannot-react` hält den betroffenen Charakter aus dem durch Bewegung geöffneten Fenster heraus; er wird nie gefragt. Vier weitere Schlüssel stehen neben den Wirkungen:
  - `modifiers`: Die Zahlen, die der Zustand verändert, solange er besteht. Siehe unten „Zahlen, die ein Zustand verändert“.
  - `saves`: Welche deiner Rettungswürfe die beiden Rettungswurfwirkungen und jeder Modifikator für Rettungswürfe betreffen. Ohne Angabe gelten sie für alle. Eine Angabe, neben der keines von beidem steht, wird abgelehnt.
  - `skills`: Welche deiner Fertigkeiten die beiden Probenwirkungen und jeder Modifikator für Proben betreffen, der keine eigenen Fertigkeiten nennt. Ohne Angabe gelten sie für alle. Eine Angabe, neben der keines von beidem steht, wird abgelehnt. Die vergleichenden Proben eines Kampfes würfeln die eigenen Probenwerte des Kampfes, keine Fertigkeiten; etwas, das auf Fertigkeiten eingegrenzt ist, erreicht sie also nie. Eine Probe außerhalb eines Kampfes mit einer dieser Fertigkeiten liest es (siehe unten „Proben außerhalb eines Kampfes“). Capability API 1.53.
  - `whileSourceInSight`: Was nur gilt, solange der Verursacher im Sichtfeld des Betroffenen ist. `true` macht den gesamten Zustand davon abhängig. Eine Liste eigener Wirkungen begrenzt nur diese und lässt den Rest bestehen. Das braucht etwa eine Furchtwirkung, die weiteres Annähern unabhängig davon verhindert, ob die Quelle sichtbar ist. Eine nicht im Zustand enthaltene Wirkung wird abgelehnt. Ohne Brett kann keine Sichtlinie unterbrochen werden; dann gilt alles in beiden Fällen.
  - `endsWhenSourceDown`: Der Zustand endet, sobald sein Verursacher zu Boden geht.

  `own-saves-advantage` und sein Gegenstück würfeln den Rettungswurf wie einen Angriff zweimal und behalten einen Wurf; sie heben sich gegenseitig auf. `own-checks-advantage` und sein Gegenstück tun dasselbe mit der Seite des Betroffenen in einer vergleichenden Probe, wo ein Kampf seine Attributsproben ablegt, und mit seinen Proben außerhalb eines Kampfes. `resist-all` halbiert jede Schadensart zusätzlich zu den eigenen Resistenzen des Ziels und hebt sich auf dieselbe Weise mit einer Verwundbarkeit auf. `cannot-target-source` verhindert, dass der Betroffene irgendetwas auf den Verursacher richtet. `cannot-approach-source` verhindert, dass er näher an diese Person herangeht als seine aktuelle Zelle, einschließlich des gesamten Weges. Ein Umweg zu einer ebenso weit entfernten Zelle wird weiterhin angeboten; ein Weg, der zwischendurch näher vorbeiführt und auf der anderen Seite endet, nicht.

  ```json
  { "condition": "restrained", "effects": ["own-saves-disadvantage"], "saves": ["dex_save"] }
  ```

  **Zahlen, die ein Zustand verändert.** Jeder Eintrag in `modifiers` nennt, was er verändert (`to`), und um wie viel:
  - `defense`: die Zahl, die ein Angriff gegen den Betroffenen erreichen muss. Nur eine feste Zahl.
  - `attacks`, `saves`, `checks`: die eigenen Angriffswürfe, Rettungswürfe und Proben des Betroffenen in vergleichenden Proben. Eine feste Zahl oder `dice`, die bei jedem dieser Würfe gewürfelt werden (`"1d4"`); `minus: true` zieht die Würfel ab, statt sie zu addieren.
  - `speed`: wie weit der Betroffene geht, in deiner eigenen Entfernungseinheit (`flat`), oder `times` 0,5 oder 2 für die Hälfte oder das Doppelte, angewendet nach allen festen Änderungen. Wird wie `speed-zero` nur auf einem Brett gelesen.

  Ein Modifikator für `checks` darf eigene `skills` nennen und einer für `saves` eigene `saves`; sie grenzen ihn dann anstelle der Angaben des Zustands ein. Beide dürfen mit `mode: "advantage"` oder `"disadvantage"` zweimal würfeln lassen. Das zählt zusammen mit den gleichnamigen Wirkungen und hebt sich wie diese auf, mit oder ohne Zahl daneben; ein Zustand kann also Sneak erschweren und zugleich Climb erleichtern. Diese drei Schlüssel brauchen Capability API 1.53.

  ```json
  { "condition": "muddy", "modifiers": [{ "to": "checks", "skills": ["sneak"], "mode": "disadvantage" }] }
  ```

  Eine feste Zahl trägt ihr eigenes Vorzeichen, und 0 wird abgelehnt. Jeder Modifikator erscheint im Protokoll neben dem Wurf, den er verändert hat, mit dem Namen des Zustands („12 + 5 + 3 (Blessed) = 20“), und Trefferchance und Gewinnchance im Menü berücksichtigen ihn. Verschiedene Zustände addieren sich; derselbe Zustand zweimal bleibt ein einziger Zustand.

  ```json
  "conditions": [
    { "condition": "blessed", "modifiers": [{ "to": "attacks", "dice": "1d4" }, { "to": "saves", "dice": "1d4" }] },
    { "condition": "shielded", "modifiers": [{ "to": "defense", "flat": 5 }] },
    { "condition": "slowed", "modifiers": [{ "to": "speed", "times": 0.5 }, { "to": "defense", "flat": -2 }] }
  ]
  ```

- `levels`: Optional. Stufen einer laufenden Leiste, die als Zustände zählen, solange die Leiste hoch genug steht. So drückt man einen Zustand aus, der sich in Stufen verschlimmert, etwa Erschöpfung. Jeder Eintrag nennt eine einfache `track` (keine Wundleiste), die Stufe `at`, ab der er beginnt, und was er bewirkt, mit denselben `effects`, `modifiers`, `failsSaves`, `saves` und `skills` wie ein Zustand. Jede von der Leiste erreichte Stufe zählt; sie addieren sich also, während die Leiste steigt. Eine Stufe hat keine Quelle und endet nur, wenn das Gelesene sinkt. `half-move-to-stand`, `ends-on-damage`, `cannot-target-source` und `cannot-approach-source` werden daher bei einer Stufe abgelehnt. Nur ein Kämpfer mit Bogen hat Leisten. Das Protokoll nennt eine Stufe nach ihrer Leiste und Zahl („- 1 (Heat 3)“).

  Anstelle von `track` kann eine Stufe einen `derived`-Wert lesen (Capability API 1.54), der mit Live-Zustand und Gegenständen des Charakters berechnet wird, sodass niemand etwas ankreuzen muss: Ember Roads verlangsamt jeden, der 10 Bulk oder mehr trägt.

  ```json
  { "derived": "bulk_carried", "at": 10, "modifiers": [{ "to": "speed", "flat": -2 }, { "to": "checks", "skills": ["sneak"], "flat": -1 }] }
  ```

  Eine Schwelle, die sich von Charakter zu Charakter unterscheidet („mehr als das Fünffache von Strength“), ist ein abgeleiteter Wert, der sie abzieht: `scale` mit -1 ergibt den abzuziehenden Teil, und eine `sum` aus beidem ist das, was die Stufe liest.

  ```json
  "levels": [
    { "track": "exhaustion", "at": 1, "effects": ["own-checks-disadvantage"] },
    { "track": "exhaustion", "at": 2, "modifiers": [{ "to": "speed", "times": 0.5 }] },
    { "track": "exhaustion", "at": 3, "effects": ["own-attacks-disadvantage", "own-saves-disadvantage"] }
  ]
  ```

  Modifikatoren, Stufen und die Probenwirkungen brauchen Capability API 1.45.

- `checks` und `contests`: Optional. Was eine vergleichende Probe liest, und die vergleichenden Proben, die jeder im Kampf beginnen darf: Packen, Stoßen, Losreißen. Siehe unten „Vergleichende Proben“. Capability API 1.43.
- `concentration`: Optional. Das laufende `text`-Feld für die aufrechterhaltene Wirkung, der durch Schaden erzwungene `save`, die Untergrenze `floor` der Schwierigkeit und `fromDamage`, der Anteil des erlittenen Schadens, der sie bei einem höheren Wert bestimmt. Der Beginn einer zweiten Konzentrationsfähigkeit beendet die erste. Ein misslungener Rettungswurf beendet sie und entfernt die von ihr aufrechterhaltenen Zustände.
- `dying`: Optional, `kind: "saves"`. Die beiden Leisten für die Würfe – die nötige Anzahl entspricht jeweils ihrem Maximum –, die `dice`, `succeedAt`, die Wirkungen der äußersten Augenzahlen (`naturals.max`: `revive-1` oder `success`; `naturals.min`: `one-failure` oder `two-failures`), die Kosten von Schaden am Boden (`damageWhileDown`, `criticalWhileDown`) und die `condition` eines niedergestreckten Charakters. Ohne diesen Block liegt ein Charakter bei null einfach am Boden und wird durch Heilung wieder aufgerichtet.
- `damageTypes`: Optional. Die Schadenstypen deines Systems, ohne Beachtung der Groß- und Kleinschreibung abgeglichen.
- `damageKinds`: Erforderlich, wenn `health` eine Wundleiste nennt, und bei einem Pool abgelehnt. Nur eine Markierung trägt eine Art; ein Punktepool hat keinen Speicherplatz dafür. Siehe Wundleisten oben. Der Block bestimmt, welche der `kinds` der Leiste ein Treffer markiert und wie viele Kästchen er ankreuzt. `default` gilt für alles nicht Zugeordnete, einschließlich Treffern ganz ohne Typ. `byType` bildet deine `damageTypes` auf Arten ab. Die Schlüssel werden wie die Typen ohne Beachtung der Groß- und Kleinschreibung abgeglichen; `"Fire"` und `"fire"` sind daher ein Schlüssel, und die Angabe beider wird abgelehnt. `marks` hat keinen Standard, weil die beiden Antworten gegensätzlich sind: `"per-point"`, wenn der Schadenswurf Gesundheitsstufen zählt – drei Schaden markieren drei Kästchen, und ein Punkt Schadensminderung lohnt sich –, oder `"per-blow"`, wenn nur zählt, ob ein Treffer ankommt – dann markiert er unabhängig von seiner Stärke ein Kästchen. Auch ein Treffer mit mehreren Schadensbestandteilen markiert dabei nur ein Kästchen und verwendet die schwerste angekommene Art. Gib an, welches Modell dein System nutzt. `{ "default": "bashing", "byType": { "fire": "aggravated" }, "marks": "per-point" }`.
- `pool`: Erforderlich bei `dice-pool`, abgelehnt bei `attack-vs-defense`. Siehe „Ein Kampf mit Würfelpools“.
- `spendLimits`: Optional. Wie viel eines laufenden Pools ein Kämpfer im Kampf pro `turn` oder pro `round` ausgeben darf, gleich welcher Kampfart: `[{ "pool": "blood", "max": { "derived": "blood_per_turn" }, "per": "turn" }]`. `max` wird einmal zu Kampfbeginn gelesen. Kosten über das hinaus, was vom Limit übrig ist, sind nicht bezahlbar und fehlen daher im Menü, egal wie viel im Pool ist. Ein Limit pro Zug beginnt zu Beginn des eigenen Zuges seines Besitzers neu, eines pro Runde, wenn eine Runde beginnt. Ein mit direkten Zahlen geschriebener Gegner bezahlt nichts von einem Bogen, ein Limit bindet ihn also nie; ein als Bogen geschriebener Gegner bezahlt aus seinen eigenen Pools und unterliegt dem Limit wie jeder andere. Capability API 1.47.
- `threat`: Optional und für ein Bestiarium erforderlich. `tiers` ist die Skala für die Gegnerauswahl: ID, Beschriftung, `health`-Bereich, `defense`, `toHit`, `damagePerRound`-Bereich und `saveDifficulty`. Jede ausgelieferte Kreatur nennt eine dieser Stufen. Ein nicht vorab geschriebener Gegner wird auf die vom Game Master gewünschte Stufe begrenzt, damit nichts außerhalb deiner Skala landet. Der Bereich `damagePerRound` beschreibt den Schaden einer Kreatur gegen EIN Ziel pro Runde, einschließlich ihrer gesamten Angriffsfolge.

### Ein Kampf mit Würfelpools: `dice-pool`

Ein `dice-pool`-Regelsatz zählt Erfolge, statt Würfel zu addieren, und seine Kämpfe können das auch. `"kind": "dice-pool"` setzt voraus, dass `resolution.kind` den Wert `"dice-pool"` hat, und liest den Bogen so, wie es deine Pool-Proben bereits tun: **Jede Zahl, die ein Wurf addiert, ist eine Anzahl Würfel, und jede Zahl, die er erreichen muss, ist eine Anzahl Erfolge.** Umbenannt wird also nichts:

- Ein Trefferwert (der einer Angriffszeile, das `toHit` einer Fähigkeitsliste, der einer Kreaturenaktion) ist der Pool, den ein Angriff wirft. Der Würfel, der Zielwert, den ein Würfel erreichen muss, und die Augenzahlen, bei denen verdoppelt, explodiert, gestrichen und gepatzt wird, stammen aus deiner `resolution`. Die Wundleiste, die dein `resolution.penaltyFrom` nennt, zieht ihren Abzug vom Pool ab, wie bei einer Probe.
- `defense` ist die Zahl der Erfolge, die ein Angriff braucht, und nie weniger als einer. Der Modifikator eines Zustands für `defense` und `cover.bonus` addieren sich dazu.
- Jeder Erfolg über die benötigten hinaus fügt einen Schadenswürfel hinzu. Ein Patzer verfehlt, egal was er gezählt hat. Kritische Treffer gibt es nicht: Ein starker Treffer ist seine zusätzlichen Würfel wert.
- Die Zahl eines Rettungswurfs ist sein Pool und seine Schwierigkeit die Zahl der nötigen Erfolge. Eine vergleichende Probe wirft die Probe jeder Seite als Pool, und die Seite mit mehr Erfolgen gewinnt.
- Der Modifikator eines Zustands für Angriffe, Rettungswürfe oder Proben besteht aus hinzugefügten oder abgezogenen Würfeln; er ist also eine `flat`-Zahl, und ein gewürfelter Modifikator (`dice`) wird abgelehnt.
- Schaden sind Würfel deines Würfeltyps: Das `dice` einer Menge (einer Kreatur, eines Eintrags) gibt an, wie viele, und muss von deinem Würfeltyp sein, und ihr `flat`-Teil sind automatische Erfolge, die nie geworfen werden. Bei der Schadenswürfelspalte einer Angriffszeile wird die Anzahl gelesen, `damage.ability` addiert den Wert dieses Attributs als Würfel, und `damage.bonus` addiert automatische Erfolge. `perCostStep` und `scales` fügen wie bisher Würfel hinzu. Heilung und temporäre Punkte sind Mengen und werden wie bisher addiert.
- Initiative ist eine Summe (`initiative.dice` und `modifier`) oder ein Pool, wenn du einen verlangst (`initiative.pool`, siehe unten), und eine Sterberegel behält ihre eigenen Würfel.

Was ein Pool-Kampf darüber hinaus würfelt, steht in seinem `pool`-Block:

```json
"pool": {
  "advantage": true,
  "damageTarget": 6,
  "soak": { "roll": true, "byKind": { "knock": { "abilityMod": "sinew" } } }
}
```

- `advantage`: ob ein Wurf, der sich in eine Richtung neigt, zweimal geworfen wird und der mit mehr Erfolgen zählt (ein Patzer zählt weniger als jeder Wurf ohne Patzer). Ohne diese Angabe wirft ein Kampf einmal, egal was ein Zustand sagt.
- `damageTarget`: der Zielwert je Würfel, gegen den Schadens- und Schadensminderungswürfel geworfen werden. Ohne Angabe der Standardzielwert deiner Auflösung. Diese Würfel zählen jede Augenzahl ab diesem Wert einmal und sonst nichts: Bei ihnen wird nichts verdoppelt, nichts explodiert, nichts gestrichen und nichts gepatzt.
- `soak`: was ein Ziel vom Schaden eines Treffers abzieht, nach Schadensart. `all` ist ein Wertverweis für jede Art und `byKind` einer pro Art deiner Gesundheitsleiste, der für seine Art Vorrang vor `all` hat; eine Art, die von keinem der beiden erfasst wird, wird nicht absorbiert. Mit `"roll": true` wirft das Ziel so viele Würfel gegen den Schadenszielwert, und jeder Erfolg zieht einen Punkt ab. Mit `"roll": false` wird die Zahl von den Schadenswürfeln abgezogen, bevor sie geworfen werden; sie berührt also nie automatische Erfolge, die eine geworfene Schadensminderung wie jeden anderen Erfolg abzieht. Nichts sinkt unter null, und für einen Schlag, der nichts gezählt hat, wird keine Schadensminderung geworfen. Eine Kreatur gibt ihre eigenen Zahlen als `soak: { "all": 1, "byKind": { "knock": 3 } }` an; dafür muss das `soak` des Blocks festlegen, wie Schadensminderung genommen wird. Resistenz, Verwundbarkeit, Immunität und `resist-all` gelten nach der Schadensminderung für den Rest.
- `hardness`: ein Wertverweis für die Härte eines Kämpfers, wo Initiative eine Zahl ist, die Angriffe verschieben. Siehe [Rüstung und getragene Wirkungen im Kampf](#armor-and-worn-effects-in-a-fight).

Der Schaden für jedes Ziel wird für dieses Ziel geworfen, denn was ein Treffer über die nötigen Erfolge hinaus verdient hat und was das Ziel absorbiert, gehört zu ihm. Der zweite Bestandteil eines Schlags und ein Zusatzeffekt sind jeweils ein eigener Schadenspool ihrer eigenen Art, ohne zusätzliche Würfel aus dem Treffer. Das Menü sagt einen Pool exakt voraus: die Chance, die nötigen Erfolge zu erreichen, und was die Schadenswürfel nach der Schadensminderung des ersten Ziels wert sind.

Gravewatch ist so geschrieben: Seine Wächter werfen einen Grundwert und eine Tätigkeit, absorbieren Knock-Schaden mit Sinew und Tear-Schaden überhaupt nicht, dürfen pro Zug einen Punkt Resolve ausgeben und würfeln die Initiative jede Runde neu.

**Ansagen in umgekehrter Reihenfolge ist nicht umgesetzt.** Manche Systeme lassen alle ihre Aktion ansagen, bevor jemand handelt, die Langsamsten zuerst. In einem Kampf, in dem jeder Kämpfer eine Aktion wählt, sobald er an der Reihe ist, ändert eine vorherige Ansage nichts, was eine Regel liest; ein Schlüssel hätte also nichts auszudrücken.

### Initiative, die Angriffe verschieben

Manche Pool-Systeme führen die Initiative den ganzen Kampf über als Zahl und lassen Angriffe sie verschieben: Eine Angriffsart nimmt sie dem Ziel weg, eine andere gibt die eigene des Angreifers als Schaden aus, und wer auf eine festgelegte Linie fällt, ist zusammengebrochen. Ein `dice-pool`-Kampf kann das ausdrücken (Capability API 1.48):

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

- **Die Eröffnung.** Jeder wirft zu Kampfbeginn `pool` Würfel, und die Erfolge plus `plus` sind seine Zahl. `resource` braucht daher `pool`: Addierte Würfel ergeben eine Reihenfolge, keine Anzahl Würfel. Das `initiativeModifier` einer Kreatur ist ihr Pool, weil jede Zahl, die ein Pool-Kampf addiert, aus Würfeln besteht. Die Zahl bleibt erhalten: Sie wird nie neu geworfen (`each` wird neben `resource` abgelehnt), und zu Beginn jeder Runde wird die Reihenfolge nach den aktuellen Zahlen sortiert.
- **Stile.** Jeder Angriff (eine Angriffszeile, eine Fähigkeit oder ein Katalogeintrag, der zum Treffen würfelt und Schaden verursacht, eine Kreaturenaktion, die das tut, und eine aus anderen Aktionen zusammengesetzte Aktion) wird einmal pro Stil angeboten, und ein Spieler wählt den Stil vor dem Ziel. Bis zu vier Stile; jeder nimmt oder gibt aus, und mindestens einer nimmt. Eine Wahl ohne Stil erfolgt im ersten; eine Wahl in einem Stil, in dem der Angriff nicht angeboten wird, wird abgelehnt. Ein Angriff behält den Stil, in dem er ausgeführt wurde, bis er vorbei ist, auch wenn eine Antwort die Zahl seines Urhebers ändert, bevor er trifft.
- **Ein nehmender Stil.** Bei einem Treffer wird der Schaden wie üblich geworfen (die zusätzlichen Würfel aus dem Treffer, die Würfel der Waffe, Schadensminderung nach Art, automatische Erfolge), und was er zählt, wird von der Zahl des Ziels statt von seiner Gesundheit abgezogen. Der Angreifer gewinnt alles davon plus `gain`.
- **Ein ausgebender Stil.** Wird nur angeboten, solange die Zahl des Angreifers über der Zusammenbruchslinie liegt (0 ohne `crash`), und nie für eine aus anderen Aktionen zusammengesetzte Aktion, weil eine Zahl für einen einzelnen Schlag ausgegeben wird. Bei einem Treffer ist der Schaden die Zahl des Angreifers zum Zeitpunkt des Angriffs, als Würfel gegen den Schadenszielwert, und sonst nichts: keine Waffenwürfel, keine zusätzlichen Würfel aus dem Treffer, keine automatischen Erfolge, keine Schadensminderung. Er markiert Gesundheit mit der eigenen Schadensart des Angriffs. Ist der Angriff vorbei, kehrt eine Zahl, die irgendwo getroffen hat, zu `base` zurück, und eine, die nirgends getroffen hat, verliert, was `onMiss` bei der Zahl vorgibt, mit der sie eingesetzt wurde (eine Stufentabelle: aufsteigende Paare `[at least, lose]`; unter der ersten Stufe nichts).
- **Zusammenbruch.** Eine Zahl, die auf `crash.at` oder darunter fällt, ist zusammengebrochen: Die `condition` (einer deiner eigenen Zustände, optional) wird dem Betroffenen auferlegt, und wer ihn dorthin gebracht hat, gewinnt `bonus`. Ein Fehlschlag, der seinen eigenen Urheber zusammenbrechen lässt, gibt niemandem einen Bonus. Ein zusammengebrochener Kämpfer kann nicht ausgeben. Der Zusammenbruch endet samt Zustand, wenn seine Zahl wieder über die Linie steigt, nach `recoverAfter` seiner eigenen Züge (seine Zahl kehrt zu Beginn dieses Zuges zu `base` zurück) und wenn der Kampf endet, wie auch immer er endet. Wer mit seinem Eröffnungswurf auf oder unter der Linie liegt, beginnt den Kampf zusammengebrochen. `base` muss über `crash.at` liegen, sonst würde die Rückkehr dorthin den Kämpfer erneut zusammenbrechen lassen.

Das Menü zeigt für jeden Stil eine eigene Vorhersage: die Trefferchance und was ein nehmender Stil nehmen oder welchen Schaden ein ausgebender verursachen würde. Die Statusanzeige zeigt die Zahl aller Beteiligten, und das Protokoll nennt jede Änderung samt Grund: „Ada gewinnt 5 Initiative für den Zusammenbruch von Rats und steht bei 14.“ Die Auswahl der Engine wägt einen Schlag im nehmenden Stil einen Zug voraus ab: erst nehmen und dann ausgeben gegen jetzt ausgeben und danach erneut von der Basis aus.

<a id="what-a-fight-reads-from-mechanics"></a>

### Was ein Kampf aus `mechanics` liest

`kind` entscheidet, ob `amount` Schaden oder Heilung bedeutet. Alles mit `reaction` fehlt im Menü, ebenso `utility`, sofern der Eintrag nicht verändert, was der Zug selbst enthalten darf; siehe unten. `attackRoll` würfelt mit dem `toHit` der Liste gegen die Verteidigung des Ziels; `autoHit` überspringt das vollständig. `save` würfelt den eigenen Rettungswurf des Ziels gegen die `saveDifficulty` der Liste. `onSuccess` legt fest, ob bei Erfolg die Hälfte oder nichts übrig bleibt. `targetCount` bestimmt die Zahl der Ziele. Eine Fähigkeit ohne Angriffswurf – etwa eine Fläche mit Rettungswürfen für alle oder ein automatischer Treffer – würfelt ihre Würfel EINMAL für alle Ziele. Eine Fähigkeit mit Angriffswurf je Ziel würfelt ihren Schaden bei jedem Treffer neu. `applies` setzt Zustände auf die betroffenen Ziele. Jeder hat eine `duration` von `instant` (ohne eigene Zeitbegrenzung; er bleibt, bis etwas ihn entfernt), `until-save` (braucht daneben `saveEnds`) oder `{ "rounds": n }`. Optional nennt `saveEnds` den Rettungswurf und dessen Wiederholung bei `turn-end` oder `turn-start`. Runden zählen die eigenen Züge des Betroffenen jeweils an deren Ende herunter; `{ "rounds": 1, "at": "turn-start" }` zählt sie jeweils an deren Beginn, und so drückt man „bis zum Beginn deines nächsten Zuges“ aus. `endsAfter` entfernt den Zustand nach dem ersten `own-attack` (dem nächsten Angriffswurf des Betroffenen), `attacked` (dem nächsten Angriffswurf gegen den Betroffenen) oder `own-save` (dem nächsten Rettungswurf des Betroffenen), egal was seine Uhr sagt; `{ "duration": { "rounds": 1 }, "endsAfter": "own-attack" }` bedeutet also „bei seinem nächsten Angriff vor dem Ende seines nächsten Zuges“. Ein Zustand, der einen Angriff lang hält, hält den ganzen Angriff einschließlich seines Schadens, und ein frischer, den derselbe Schlag auferlegt, bleibt bestehen. Beides braucht Capability API 1.45. `temporary` gewährt temporäre Punkte auf dem Gesundheitspool. Sie addieren sich nie; der größere Puffer bleibt. `scales` erhöht die Menge um die zusätzlichen WÜRFEL, die seine Tabelle für den gelesenen Wert vorgibt. `cost` wird über den eigenen `use`-Befehl des Bogens bezahlt. `budget` überschreibt den verbrauchten Teil des Aktionsbudgets.

`plus` enthält neben `amount` bis zu drei WEITERE Mengen desselben Treffers. Jede wird getrennt gewürfelt und typisiert, etwa "und 2d6 Feuer". Ein Bestandteil lautet `{ "dice": "2d6", "flat": 1, "type": "fire" }` und kann einen eigenen `save` tragen: `{ "save": "con_save", "difficulty": 13, "onSuccess": "none" | "half" }`. Diesen würfelt das ZIEL unabhängig von den bisherigen Anforderungen der Aktion. Bei Erfolg lässt `none` nichts von diesem Bestandteil übrig und `half` die Hälfte; der Rest des Treffers bleibt in beiden Fällen unberührt. Ohne `difficulty` wird die Zahl des Rettungswurfs der Aktion verwendet, danach die `saveDifficulty` der Liste. Ein kritischer Treffer verdoppelt die Würfel jedes Bestandteils nach derselben Regel wie die der ersten Menge. Ein Bestandteil ohne `type` verwendet die Schadensart des Treffers. Der gesamte Treffer bleibt EINE Konzentrationsprüfung mit dem summierten Schaden und eine Prüfung auf Niedergeschlagenwerden. Ein Bestandteil braucht ein `amount`, zu dem er gehört; `heal` trägt keinen.

```json
{
  "kind": "attack",
  "attackRoll": true,
  "amount": { "dice": "1d8" },
  "damageType": "piercing",
  "plus": [{ "dice": "2d6", "type": "fire" }]
}
```

Drei Schlüssel beschreiben die Wirkung eines Eintrags auf das Aktionsbudget des Zuges. Ein `utility`-Eintrag mit mindestens einem davon wird angeboten statt verworfen:

- `free`: Der Eintrag kostet überhaupt kein Budget. Die unter `cost` genannten Ressourcen werden trotzdem bezahlt. Gleichzeitig darf kein `budget` angegeben sein.
- `gives`: Bis zu vier Angaben wie `[{ "budget": "action", "count": 1 }]`. Die Nutzung erhöht diese Budgets sofort. Der neue Bestand ist auf das normale Zugbudget plus Zugabe begrenzt, damit sich nichts für spätere Züge ansparen lässt.
- `standard`: `{ "actions": ["dash", "disengage", "hide"], "budget": "bonus" }`. Der Besitzer darf diese Standardaktionen mit DIESEM Budget bezahlen. Sie werden neben den gewöhnlichen Aktionen als `standard:<id>@<budget>` angeboten. Besteht der Eintrag nur aus dieser Erlaubnis, bleibt er selbst dem Menü fern, denn eine Erlaubnis ist keine auswählbare Aktion.

Ein Eintrag mit dem neuen `kind: "rider"` ist PASSIV: Niemand wählt ihn aus, er steht nie im Menü und fügt dem ersten passenden Treffer eines Zeitraums automatisch einen weiteren Schadensbestandteil hinzu. Er trägt `rider` und keine anderen aktiv auszuführenden Angaben:

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

`sources` nennt die Angriffslisten, von denen der Effekt ausgeht, und `requires` eine als wahr gewertete Spalte ihrer Zeilen. So kann ein Zusatzeffekt seine Einschränkung auf bestimmte Waffen beschreiben, ohne dass die Engine wissen muss, was eine Waffe ist. Fehlen beide Angaben, gilt jeder Treffer des Besitzers. Eine aus der Tasche gehaltene Waffe ist keine Zeile; `weapons` (Capability API 1.70) nennt daher zusätzlich die Gegenstands-Tags, bei denen der Effekt auslöst: `"weapons": { "tags": ["finesse",
"ranged"] }` löst bei einem Treffer mit einer gehaltenen Waffe aus, die eines dieser Tags trägt, einschließlich ihres Schlags mit der Nebenhand. Wer nur `weapons` nennt, löst bei keiner Zeile aus, und wer nur Zeilen nennt, bei keiner gehaltenen Waffe. Bei `when` reicht EINE Bedingung: `advantage` ist der letztlich angewendete Vorteil des Angriffswurfs. `ally-adjacent` ist ein aufrecht stehender, handlungsfähiger Verbündeter des Angreifers, auf einem Brett höchstens eine Zelle vom Ziel entfernt und ohne Brett an beliebiger Position. `oncePer` ist `turn` (frisch zu Beginn jedes Zuges, damit auch ein Angriff während eines fremden Zuges einen Zusatzeffekt tragen kann) oder `round`. `amount` wächst mit den eigenen `scales` des Eintrags. `type` ist die Schadensart und verwendet standardmäßig die des Treffers.

<a id="creatures-a-bestiary-a-fight-reads"></a>

### Kreaturen: Ein Bestiarium für Kämpfe

Ein Katalog mit `"holds": "creatures"` enthält Gegner statt Bogenzeilen. Er befüllt keine Liste, wird nie in der Auswahl des Bogeneditors angeboten und verwendet für jede Zahl die bereits von deinem `combat`-Block deklarierten Schlüssel. Er braucht einen `combat`-Block und eine `threat`-Skala, weil jede Kreatur einer deiner eigenen Stufen zugeordnet wird.

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

Die folgenden Zahlen sind die direkte Schreibweise für eine Kreatur. Eine in den eigenen Begriffen deines Regelsatzes als `sheet` geschriebene Kreatur liest `health`, `defense`, `initiativeModifier`, `speed`, `abilities`, `saves`, `checks`, `soak` und `hardness` stattdessen aus diesem Bogen; siehe unten "Eine Kreatur in den eigenen Begriffen deines Regelsatzes".

- `health`: Eine Zahl oder `{ "dice": "3d6", "flat": 2 }`, einmal beim Erstellen des Kampfes gewürfelt. Eine Vorschau verwendet den Durchschnitt, damit das Menü keinen ungewürfelten Wert verspricht.
- `defense`, `initiativeModifier`, `speed`: Der Zielwert eines Angriffs, der Initiativmodifikator (oder, wo Initiative als Pool geworfen wird, die Zahl der Würfel, die sie wirft) und die Bewegungsweite pro Zug in deiner eigenen Entfernungseinheit.
- `abilities` und `saves`: Zuordnungen anhand der Attributs- und Rettungswurf-IDs deines Bogens. Ein nicht genannter Rettungswurf wird als null gelesen.
- `checks`: Was sie in einer vergleichenden Probe addiert, mit den IDs von `combat.checks` als Schlüssel. Ein nicht genannter Wert wird als null gelesen. Capability API 1.43.
- `soak`: Nur in einem `dice-pool`-Kampf; was sie absorbiert: `all` für jeden Schaden, `byKind` für eine Art deiner Gesundheitsleiste. Capability API 1.47.
- `hardness`: Wo Initiative eine Zahl ist, die Angriffe verschieben, bewirkt ein Schlag im ausgebenden Stil, dessen Würfel darunter liegen, nichts. Capability API 1.56.
- `resist`, `vulnerable`, `immune`: Schadenstypen, ohne Beachtung der Groß- und Kleinschreibung abgeglichen und bei vorhandener Deklaration gegen `combat.damageTypes` geprüft. Ein `resist`- oder `immune`-Eintrag kann außerdem angeben, was hindurchkommt, `{ "type": "tearing", "except": ["silver"] }`: Der Schlag einer Gegenstandswaffe, die eines dieser Gegenstands-Tags trägt, wird voll erlitten (Capability API 1.55, siehe [Waffen im Kampf](#weapons-in-a-fight)). `conditionImmunities` nennt deine eigenen Zustände.
- `tier`: Die zugehörige Stufe von `combat.threat`.
- `loot` (Capability API 1.63): Die ID der Beutetabelle, die ein gewonnener Kampf für sie würfelt (siehe oben „Beute“).
- `traits`: Kurze Paare aus Name und Text für den Game Master. Sie werden nie regeltechnisch aufgelöst. Alles mit Zahlen gehört deshalb in eine Aktion.
- `signaturePoints`: Punkte, die zu Beginn des eigenen Zuges zurückkehren und für `signature`-Aktionen ausgegeben werden.
- `riders`: Bis zu vier Zusatzeffekte wie `rider` an einem Katalogeintrag, hier direkt im Block. Jeder hat die Form `{ "id": "pack", "name": "Pack", "on": "hit", "oncePer": "turn" | "round", "amount": { "dice": "1d6" } }`, mit optionalem `type` und optionalen `actions` für die eigenen Aktionen dieses Blocks, bei denen er auslöst. Ein Block-Zusatzeffekt liest keine Bogenliste; deshalb hat er die Schlüssel `sources` und `requires` nicht. Eine als Bogen geschriebene Kreatur erhält wie ein Charakter die Zusatzeffekte ihrer Listen.
- `actions`: Bis zu zwölf, jeweils mit eigener `id`. Eine Aktion enthält die Angaben eines handgeschriebenen Werteblocks (`toHit`, `autoHit`, `damage`, `save`, `applies`, `targetCount`, `reach`, `range`, `area`) sowie vier nur für Kreaturen vorgesehene Angaben. `reach` ist die Nahkampfreichweite, `range` die Wurf- oder Schussreichweite und `area` die betroffene Form, alles in deiner eigenen Entfernungseinheit. `range` darf eine einfache Zahl sein oder `{ "normal": 30, "long": 120 }`, wenn größere Entfernung mit einem Abzug erreichbar bleibt. `area` lautet `{ "shape": "burst" | "cone" | "line", "size": n, "friendlyFire": false }`; siehe Positionen:
  - `uses`: `{ "per": "encounter" | "day", "count": n }`. Sind die Nutzungen aufgebraucht, verschwindet die Aktion aus dem Menü.
  - `recharge`: `{ "dice": { "count": 1, "sides": 6 }, "from": 5 }`. Die Aktion beginnt verfügbar, wird bei Nutzung verbraucht und würfelt zu Beginn des eigenen Zuges der Kreatur. `from` oder höher stellt sie wieder her. Das Protokoll zeigt die Würfel in beiden Fällen.
  - `sequence`: Andere Aktionen desselben Blocks in Reihenfolge, jede mit eigenem Ziel. **So wird eine Kreatur geschrieben, die mit einer Aktion zweimal angreift.** Ein Budget bezahlt die gesamte Folge. Eine Folge hat keine eigenen Wirkungen und darf nie eine andere Folge nennen.
  - `signature`: `{ "cost": n }`, bezahlt mit den eigenen Punkten der Kreatur statt mit einem Budget und nur, während jemand anderes handelt. Der Kampf bietet sie im Fenster zwischen zwei Zügen an; siehe Fenster.
  - `reaction`: Dasselbe Objekt wie `mechanics.reaction` eines Katalogeintrags (siehe Fenster): Die Aktion steht in keinem Zugmenü und wird stattdessen in dem Moment angeboten, den sie nennt. Eine Reaktion ist nicht zugleich eine Signaturaktion, und keine Folge darf eine enthalten.
  - `self: true`: Sie wirkt auf die Kreatur selbst statt auf jemand anderen und nimmt daher weder `targetCount` noch `area`. Beides braucht Capability API 1.46. Eine Parade (Parry) ist beides, mit einem Zustand `parrying`, der 2 zur Verteidigung addiert:

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
- Ein Rettungswurf braucht eine Schwierigkeit an der Aktion selbst: `save.difficulty` für einen durch die Aktion erzwungenen Rettungswurf oder `saveDifficulty` für einen durch Rettungswurf endenden Zustand, wenn die Aktion keinen eigenen Rettungswurf hat. Auch bei einer Kreatur mit Bogen wird eine Blockaktion in direkten Zahlen geschrieben; die Schwierigkeit steht daher an der Aktion. Der eigene Rettungswurf eines Schadensbestandteils darf seine `difficulty` auslassen und auf dieselbe Zahl zurückgreifen.
- `damage.plus` ist dieselbe Liste von Schadensbestandteilen wie `plus` an einem Katalogeintrag und wird genauso gelesen: `"damage": { "dice": "1d6", "flat": 2, "type": "piercing", "plus": [{ "dice": "1d4", "type": "fire" }] }` beschreibt einen Biss mit eigenem Feuerschaden, auf den Resistenzen und Verdopplung getrennt angewendet werden.

Das Bestiarium des 5e-Entwurfs enthält fünf handgeschriebene Kreaturen in `docs/development/ruleset-5e-2014.example.json`. Sie decken eine Aktionsfolge, Wiederaufladung, einen Rettungswurf mit Zustand, Resistenzen und Immunitäten, begrenzte Nutzungen, Signaturpunkte und eine als Bogen geschriebene Kreatur ab.

#### Eine Kreatur in den eigenen Begriffen deines Regelsatzes

Eine Kreatur muss nicht mit direkten Zahlen geschrieben werden. Gib ihr stattdessen ein `sheet` mit genau derselben Form wie ein Charakterbogen. Der Kampf baut sie dann wie ein Gruppenmitglied: Gesundheit, Verteidigung, Rettungswürfe, Initiative, Geschwindigkeit sowie sämtliche Angriffe und Fähigkeiten ihrer Listen ergeben sich aus deinen Bogenformeln. So sagt ein Regelsatz, dass seine Gegner dieselben Attribute, Fertigkeiten und Listen wie seine Charaktere haben, unabhängig von deren Ausgestaltung. Der Toll Warden aus Ember Roads:

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

- **Jeder Teil ist optional**: `abilities`, `skills`, `saves`, `bonuses`, `fields` und `lists`, mit den vom Bogen deklarierten IDs als Schlüssel. Ausgelassene Angaben verwenden die eigenen Bogenstandards, genau wie ein leerer Charakter. Grit des Wächters ist 9, weil dein `grit_max` 4, seine Toughness und sein Brawn addiert. Guard ist aus einem ähnlichen Grund 7.
- **Jede Zahl hat genau eine Quelle.** Eine Kreatur mit Bogen gibt nicht zusätzlich `health`, `defense`, `initiativeModifier`, `speed`, `abilities`, `saves` oder `checks` an. Tut sie es doch, lehnt die Engine die Datei ab. Sie darf ohne eigene `actions` auskommen, weil ihre Listen ihr Handeln beschreiben. Eine Kreatur ohne Bogen gibt weiterhin die ersten drei Werte und mindestens eine Aktion an.
- **Die Angaben werden als verfasste Daten geprüft.** Jede ID muss im Bogen deklariert sein. Eine Fertigkeit oder ein Rettungswurf verwendet eine dafür angebotene Ausbildungsstufe. Ein Feld, Wert, Bonus oder eine Spalte enthält genau den deklarierten Typ – eine ganze Zahl im erlaubten Bereich, einen angebotenen Auswahlwert und so weiter. Eine Liste enthält höchstens die erlaubte Zeilenzahl. Es gibt keinen `live`-Teil, denn den Verbrauch einer Kreatur verwaltet der Kampf.
- **Eine Zeile kann aus einem Katalog stammen.** `_catalog: "<catalog>/<entry>"` nennt wie auf einem Charakterbogen den gewählten Eintrag. Von dort liest der Kampf Kosten und Wirkung der Zeile. Der Katalog muss diese Liste befüllen. Bei einem direkt eingebetteten Katalog muss der Eintrag vorhanden sein. Bei einer separaten Katalogdatei gewährt eine Zeile mit nicht vorhandenem Eintrag der Kreatur schlicht nichts. Die von Bestiariumbögen genannten Kataloge werden zusammen mit dem Bestiarium für den Kampf geladen.
- **Die Angaben neben dem Bogen gelten weiter**: `tier`, `traits`, `actions`, `signaturePoints`, `riders`, `resist`, `vulnerable`, `immune` und `conditionImmunities`.
- **Sie bezahlt aus ihren eigenen Pools.** Diese starten voll und bezahlen, was ihre Listen anbieten. Größere Zahlungsmöglichkeiten, etwa ein Zauber aus einem höheren Platz, werden genauso wie bei Gruppenmitgliedern angeboten – unabhängig davon, ob die Engine oder der Game Master entscheidet. Hold the Line kostet den Wächter Luck.
- **Bei einer Wundleiste ist die Leiste ihre Gesundheit.** Ein Treffer markiert die eigene Leiste der Kreatur nach deinen `damageKinds`, nachdem `resist`, `vulnerable` und `immune` angewendet wurden. Eine gegen eine Schadensart immune Kreatur erhält dadurch keine Markierung.
- **Sie bleibt ein Gegner.** Bei null scheidet sie aus, statt im Sterben zu liegen, und macht nie Todesrettungswürfe. Der Bildschirm zeigt die üblichen Gegnerangaben und nichts von ihrem Bogen. Verbrauch wird nirgendwo zurückgeschrieben, auch nicht bei einem namensgleichen Charakter.
- **Ein Bogen, der insgesamt keine Gesundheit ergibt**, wird aus dem Kampf ausgelassen und im Eröffnungsprotokoll begründet, statt als unverwundbares Wesen aufzutreten.
- Eine Ebene, die einen Wert aus einem deiner Enum-Felder entfernt, entfernt niemals eine Kreatur, die ihn verwendet. Solange die Ebene aktiv ist, liest dieses Feld für die Kreatur wie für einen Charakter seinen Standardwert. Die Kreatur wird deswegen nicht abgelehnt.
- Auch der Game Master kann eine solche Kreatur erfinden. Sie wird auf ihre Stufe begrenzt; siehe "Nicht vorab geschriebene Gegner".
- Ein Paket mit einer solchen Kreatur deklariert Capability API 1.34.

Der Toll Sergeant des 5e-Entwurfs setzt dasselbe auf einem d20-Bogen um: Rüstungsklasse, Trefferpunkte, Rettungswürfe und zwei Angriffe pro Aktion stammen alle aus seinen eigenen Feldern und seiner Angriffsliste.

#### Nicht vorab geschriebene Gegner

Erfindet der Game Master einen Gegner, begrenzt die Engine den Vorschlag vor jedem Wurf auf deine `threat`-Skala: Gesundheit auf den Stufenbereich, Verteidigung, Trefferbonus und Rettungswurfschwierigkeiten auf höchstens zwei über den jeweiligen Stufenwert. Der Schaden wird so weit verringert, bis die beste Runde der Kreatur – ihre stärkste Folge oder Einzelaktion gegen ein Ziel – innerhalb von `damagePerRound` der Stufe liegt. Zuerst sinkt die Würfelzahl, dann der feste Anteil, dann wird ein Angriff aus einer Folge entfernt und erst danach die Würfelgröße reduziert. Nichts wird auf null reduziert. Im Regelsatz unbekannte Namen werden entfernt: unbekannte Schadenstypen, Zustände und Rettungswürfe sowie alles nach den ersten sechs Aktionen. Eine nicht deklarierte Stufe fällt auf die unterste deiner Skala zurück. Jede Änderung wird als einfacher Satz zurückgegeben, damit das Protokoll sie erklären kann.

Auch eine erfundene Kreatur darf wie eine Bestiariumkreatur als `sheet` geschrieben sein. So erhält ein erfundener Magier Plätze und Zauber. Der Game Master sieht die IDs deines Bogens samt erlaubten Inhalten, die vom Kampf gelesenen Listen und die dafür in deinen Katalogen angebotenen Namen. Ein Zauber wird dadurch benannt statt beschrieben: Eine Zeile, die einen Katalogeintrag ohne Beachtung der Groß- und Kleinschreibung nennt, wird zu diesem Eintrag. Die eigenen Werte des Game Master, etwa ob der Zauber vorbereitet ist, werden darübergelegt. Der Bogen wird nachsichtig gelesen, weil er von einem Modell stammt. Unbekannte Namen entfallen, Werte werden an ihr Feld oder ihre Spalte angepasst, und neben dem Bogen geschriebene Zahlen werden nicht verwendet.

Eine erfundene Kreatur, die kein Boss ist, bleibt auf das vom Regelsatz Erlaubte beschränkt. Ein Katalogfilter mit `startFrom` nennt das Bogenfeld, nach dem seine Einträge geordnet sind – etwa `class` für die Zauberliste des 5e-Pakets. Eine erfundene Kreatur behält nur Einträge, deren Filter zu ihrem eigenen Feldwert passt, mit demselben Abgleich wie beim Öffnen der Auswahl. Ein Sorcerer erhält also niemals die gesamte Zauberliste. Ohne Klassenangabe erhält eine Kreatur nichts aus einem nach Klassen geordneten Katalog. Offengelassene Entscheidungen werden anschließend ohne erneute Anfrage an den Game Master ergänzt. Für jede Liste, deren Zeilen erst nach Auswahl zählen (`onlyWhen` an einer Kampffähigkeitsquelle), werden aus den erlaubten und aus eigenen Pools bezahlbaren Einträgen einige je vorhandenen Pool ausgewählt, ebenso für beliebig oft nutzbare Fähigkeiten. Kompetentere Kreaturen erhalten mehr Einträge. Die Auswahl berücksichtigt dasselbe Temperament und dieselbe Kompetenz wie ihr Kampfverhalten: schützende oder unterstützende Kreaturen bevorzugen Hilfen für ihre Seite, rücksichtslose Schaden, methodische oder geduldige die Behinderung von Gegnern. Mit steigender Kompetenz werden Reaktionen, Gegenzauber und andere Eingriffe in den Zug wahrscheinlicher. Die Auswahl verwendet den Seed des Kampfes; derselbe Kampf wird deshalb immer gleich ergänzt. Eine vom Game Master genannte Zeile aus einer solchen Liste gilt als ausgewählt.

Einen Boss darf der Game Master als mögliche Ausnahme vollständig selbst schreiben. Bei ihm wird nichts entfernt und nichts ergänzt.

Danach werden beide auf ihre Stufe begrenzt:

- Gesundheit wird über das einzelne Feld, aus dem sie gelesen wird, in den Stufenbereich gebracht. Das Pool-Maximum ist dieses Feld oder eine `sum` mit genau einem Feld darin, etwa das Trefferpunktemaximum in 5e oder Toughness in Ember Roads. Eine Gesundheitsformel ohne ein solches einzelnes Feld bleibt unverändert; das Protokoll erklärt das. Die Länge einer Wundleiste stammt von dir und wird nie verändert.
- Nach dem Aufbau der Kreatur werden Verteidigung, Trefferbonus und Rettungswurfschwierigkeiten auf zwei über dem Stufenwert begrenzt. Der Schaden sinkt, bis ihre beste Runde innerhalb von `damagePerRound` liegt, unter Berücksichtigung der höchsten bezahlbaren Kosten. Zuerst wird die Zusatzwirkung höherer Zahlungen reduziert, dann die Würfel, der feste Anteil, ein Angriff und erst zuletzt die Würfelgröße.

Dein eigenes Bestiarium wird niemals begrenzt. Seine Daten stammen von dir; die Engine übernimmt sie so, wie du sie geschrieben hast.

### Positionen: Ein Kampf auf einem Brett

Ein Kampf bleibt in der Vorstellung, bis dein Block den Wert einer Brettzelle angibt. Mit `distance` kann er auf einem Raster stattfinden. Dann bekommen Bewegung, Nah- und Fernkampfreichweiten, Flächen, Sichtlinien, Deckung und Angriffe auf Weggehende eine Bedeutung. Jede Zahl stammt von dir; die Engine stellt nur das Brett bereit.

```json
"distance": { "label": "ft", "perCell": 5 },
"ranged": { "long": "disadvantage", "adjacentFoe": "disadvantage" },
"cover": { "bonus": 2 },
"opportunity": { "budget": "reaction" }
```

Ember Roads deklariert davon eine einzige Zeile und sonst nichts. Genau darum geht es: Der Rest ist nicht erforderlich.

```json
"distance": { "label": "paces", "perCell": 2 }
```

**Die Zelle.** `distance.perCell` gibt an, wie viel DEINER Einheit eine Zelle entspricht. `label` benennt diese Einheit. Alle Entfernungen des Blocks verwenden sie: `economy.movement`, `speed` einer Kreatur, `reach` und `range` einer Waffe sowie `reach` und `range` einer Kreaturenaktion. Ein Katalog mit eigenen `units.distance` rechnet seine `mechanics.range` und `area.size` mit seinem eigenen `perCell` um; ohne eigene Angabe verwendet er diese. Eine positive Entfernung wird auf die nächste Zelle gerundet, niemals auf null. Alles mit einer positiven Zahl reicht daher mindestens eine Zelle weit. Null ist keine kurze Entfernung, sondern behält ihre eigene Bedeutung: `mechanics.range` gleich 0 bedeutet selbst oder Berührung; eine Berührung eines anderen reicht bis in die Nachbarzelle. Eine Waffenzeile mit 0 in ihrer `reach`- oder `range`-Spalte hat dagegen keine entsprechende Entfernung.

**Ob ein Kampf auf einem Brett stattfindet.** Zwei Voraussetzungen müssen erfüllt sein: Dein Block deklariert `distance`, und das Spiel ist auf den Kampfstil **Tactical** eingestellt. Mit **Classic** oder einem Regelsatz ohne `distance` bleibt der Kampf wie bisher in der Vorstellung: Jeder kann jeden anvisieren, und nichts von den folgenden Angaben wird gelesen.

**Was Spieler sehen.** Das Brett wird mit dem Gelände des taktischen Stils gezeichnet. Jedes Feld ist eine Schaltfläche, per Zeiger oder Pfeiltasten erreichbar, und nennt seine Art, seine Belegung und seine Bedeutung für die angefangene Auswahl. Bei Bewegung leuchten die vom Menü angebotenen Felder auf, jeweils mit Kosten IN DEINER EINHEIT. Der Weg wird eingezeichnet. Felder, deren Weg einen Angriff provoziert, erscheinen bernsteinfarben; unter dem Brett werden die Angreifer genannt. Eine Option mit Zielauswahl hebt erlaubte Ziele gleichzeitig auf dem Brett und in der Liste hervor. Eine Option mit `area` zielt auf ein Feld. Das Feld unter dem Zeiger zeigt die Betroffenen, einschließlich Verbündeter. Die verbleibende Bewegungsweite steht neben deinen Budgets, wieder in deiner Einheit. Der Bildschirm misst nichts davon selbst: Jedes Feld, jede Kostenangabe, jeder Weg, jedes Ziel und jeder Zielpunkt kommen vom Server.

**Bewegung.** Die Zugreichweite ist `economy.movement` eines Gruppenmitglieds oder `speed` der Kreatur, geteilt durch `perCell` und ABGERUNDET, mindestens aber eine Zelle, solange Bewegung überhaupt möglich ist. Sie füllt sich zu Beginn des eigenen Zuges auf und darf vor, zwischen und nach Aktionen verbraucht werden: gehen, angreifen, weitergehen. Ein Feld kostet beim Betreten einen Punkt, schwieriges Gelände mehr. Es gibt acht Richtungen mit gleichen Kosten, wie auf den dafür vorgesehenen Tischrollenspielrastern. Verbündete dürfen passiert werden, doch auf niemandem darf die Bewegung enden. Gegner gelten als Wände. Feste Hindernisse dürfen weder betreten noch diagonal zwischen zwei solchen Zellen umgangen werden.

**Nah- und Fernkampfreichweite.** Eine Waffenzeile liest sie aus `combat.attacks[].reach` und `.range`, jeweils eine Spalte derselben Liste oder dieselbe Zahl für jede Zeile:

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

Liest eine Spalte für eine Zeile 0, hat die Zeile keine entsprechende Reichweite. So können ein gewöhnliches Schwert und eine Wurfaxt in derselben Liste stehen. Eine Zeile ganz ohne Nahkampfreichweite reicht eine Zelle weit. Eine Kreaturenaktion verwendet ihre eigene `reach` oder `range`, eine Katalogfähigkeit ihre `mechanics.range`. Dabei bedeutet 0 selbst oder Berührung, bei einem anderen Ziel also eine Zelle.

Eine Zeile mit BEIDEN Angaben ist eine Wurfwaffe: innerhalb ihrer Nahkampfreichweite ein Schlag, außerhalb ein Wurf. Die folgenden Fernkampfregeln betreffen sie also nicht im Nahkampf. Sie kann auch einen Weggehenden treffen, anders als ein Bogen.

Eine Kreaturenaktion darf ihre `area` in deiner Einheit angeben: `{ "shape": "cone",
"size": 15 }`, mit `"friendlyFire": false`, um die eigene Seite zu verschonen. So wird eine Atemwaffe auf dem Brett zu einem echten Kegel statt zu einer Zielanzahl. Eine Aktionsfolge hat keine eigene Form; ihre einzelnen Aktionen tragen ihre Formen. Ein Kampf ohne Brett ignoriert die Form und verwendet `targetCount`. Ein Kreatureneintrag kann deshalb beides enthalten und in beiden Darstellungen zutreffen.

**Wie weit eine Form geschickt werden darf.** Das legt `range` fest: Eine hundert Fuß weit geworfene Kugel hat eine solche Angabe. Ohne Reichweite explodiert eine Kreisfläche dort, wo sie abgelegt wird, auf der eigenen Zelle des Handelnden. Ein Kegel oder eine Linie darf innerhalb der eigenen Länge anvisiert werden, weil das Zielfeld hier nur die Richtung angibt. Das gilt für `mechanics.area` eines Katalogeintrags genauso wie für Kreaturen.

`ranged` legt den Nachteil eines Fernangriffs jenseits seiner gewöhnlichen Entfernung `normal` oder bei einem Gegner in der Nachbarzelle fest. Beide Angaben sind `"disadvantage"` oder `"normal"`. Ohne den Block kostet beides nichts. Ein Nahkampfschlag ist kein Fernangriff und wird von keiner Regel betroffen; dasselbe gilt für eine Wurfwaffe innerhalb ihrer Nahkampfreichweite.

**Flächen.** `mechanics.area` eines Eintrags wird zu einer echten Form auf dem Brett, die auf eine Zelle statt auf eine Person zielt. `targetCount` sagt darüber nichts aus: Die Form bestimmt die Zahl der Betroffenen. Alle in den erfassten Zellen werden getroffen, Freund wie Feind, außer der Eintrag setzt `"friendlyFire": false`.

```
burst, size 2, aimed at X        cone, size 3, aimed right      line, size 3, aimed right
. . . . .                        . . . .                        . . . .
. # # # .                        . . # .                        A # # #
. # X # .                        A # # #                        . . . .
. # # # .                        . . # .
. . . . .                        . . . .
```

Eine Kreisfläche umfasst jede Zelle innerhalb ihrer Größe um den Zielpunkt. Ein Kegel verläuft vom Handelnden zum Zielfeld und ist bei jedem Schritt so breit wie weit entfernt. Eine Linie verläuft ebenso, aber eine Zelle breit. Alle drei enden an festen Hindernissen.

**Sichtlinie und Deckung.** Zwischen beiden wird eine gerade Linie von Zellen betrachtet. Jedes feste Hindernis darauf blockiert einen Fernangriff und die weitere Ausbreitung einer Fläche; das Ziel erscheint dann gar nicht im Menü. Als Deckung zählendes Gelände addiert `cover.bonus` auf die Verteidigung gegen den Angriff, und das Protokoll nennt den Bonus. Es gibt weder Dreiviertel- noch vollständige Deckung und keine Höhenunterschiede.

**Angriffe auf Weggehende.** Mit `opportunity.budget` gilt: Verlässt ein Kämpfer die Reichweite eines aufrecht stehenden, handlungsfähigen Gegners, der dieses Budget und einen Nahkampfangriff hat, STOPPT die Bewegung an dieser Stelle. Der Gegner wird gefragt, ob er angreifen möchte. Der Angriff verbraucht das Budget und wird genau wie derselbe Angriff im eigenen Zug aufgelöst. Verstreichenlassen kostet nichts. Anschließend wird die Bewegung dort fortgesetzt, wo sie angehalten wurde, und bezahlt jede tatsächlich betretene Zelle. Streckt der Angriff den Gehenden nieder, endet die Bewegung an seinem Sturzort. Jeder Gegner bekommt für eine gesamte Bewegung eine Gelegenheit, unabhängig davon, wie oft der Weg seine Reichweite verlässt. `disengage` verhindert das für den Rest des Zuges. Ohne `opportunity` kennt der Regelsatz diesen Ablauf überhaupt nicht.

Die Nachfrage ist ein FENSTER, das den gesamten Kampf anhält. Nichts anderes bewegt sich, bis alle Befragten geantwortet haben. Das Fenster eines Gruppenmitglieds beantwortet der Spieler mit dem Angriff oder der danebenstehenden Option **Pass** (passen). Für alle anderen antwortet ihre steuernde Instanz, bei einem Boss des Game Master dessen eigene Entscheidung. Siehe Fenster unten.

**Was ein Gegner auf dem Brett tut.** Ein nicht fremdgesteuerter Gegner bewertet jede erreichbare Zelle zusammen mit jeder von dort möglichen Option. Für jeden provozierten Angriff zieht er Wert ab. Kann er seine beste Aktion bereits vom aktuellen Standort ausführen, bleibt er vorzugsweise stehen. Ist nichts in Reichweite, nähert er sich und sprintet zuerst, sofern deine `standard`-Liste `dash` enthält.

**Mögliche Ablehnungen.** `out-of-reach` (außerhalb der Reichweite), `no-line-of-sight` (ein festes Hindernis steht im Weg), `unreachable` (ein Feld ist nicht bezahlbar oder kein erlaubtes Bewegungsende) und `bad-cell` (ein unerlaubter Zielpunkt für eine Fläche).

### Was der Server im Kampf mit deinem Block macht

Ein Spiel mit einem Regelsatz, der `combat` deklariert, erhält einen entsprechend aufgelösten Kampf im selben gespeicherten Kampfdatensatz wie bisher:

- **Wer teilnimmt.** Der Game Master nennt die Beteiligten. Die Engine liest die Werte jedes Gruppenmitglieds vom eigenen Bogen. Ein Mitglied ohne Bogen für deinen Regelsatz wird namentlich abgelehnt, statt nicht von dir verfasste Zahlen zu erhalten.
- **Woher die Gegnerwerte stammen**, in dieser Reihenfolge: die vom Game Master benannte Bestiariumkreatur; eine Kreatur, deren Beschriftung dem Gegnernamen entspricht; ein vom Game Master für diesen Kampf vorgeschlagener und auf deine Bedrohungsskala begrenzter Werteblock; zuletzt eine einfache Kreatur aus den Zahlen der Stufe. Jeder Rückgriff und jede Begrenzung wird in einfachen Worten protokolliert. Ohne Bestiariumeintrag, Vorschlag und Bedrohungsskala lehnt ein Regelsatz den Kampf ab, statt Werte zu erfinden.
- **Deine Bögen sind der Datensatz.** Gesundheit, Pools, Zustände, Konzentration und Zähler deiner Sterberegel werden nach jeder angenommenen Aktion über die Bogenregeln geschrieben. Ein Neuladen mitten im Kampf zeigt dadurch exakt den hinterlassenen Zustand. Es gibt keine abschließende Kampfabrechnung, die davon abweichen könnte.
- **Dein Menü allein bestimmt die Erlaubnis.** Jeder Handelnde, Spieler wie Gegner, wählt eine ID aus demselben von deinem Block erzeugten Menü. Ein von der Engine gesteuerter Gegner verwendet ihre Taktik. Ein vom Game Master gesteuerter Gegner soll eine ID aus genau diesem Menü wählen, sieht deine Zahlen und erfährt nie vorab das Würfelergebnis.
- **Deine Würfel.** Ein Kampf führt seinen eigenen Seed und Positionszähler. Nach dem Laden von der Festplatte setzt er damit dieselbe Würfelfolge fort.

### Auf dem Bildschirm

Der Kampf läuft auf dem Kampfbildschirm in deinen Begriffen. Das Menü enthält deine Angriffe, Fähigkeiten, vergleichenden Proben und aufgeführten Standardaktionen, jeweils mit Kosten aus deinen Budgets und Pools. Angezeigt werden Zugreihenfolge, Runde, alle benannten Zustände mit verbleibenden Runden, temporäre Punkte, Konzentration und die beiden Zähler deiner Sterberegel. Das Protokoll zeigt die echte Rechnung in deinen Begriffen, etwa: "Juno greift Rust jackal mit Road axe an: 8 (5 + 3) + 3 = 11 gegen Guard 6, ein Treffer." Jede angenommene Aktion wird sofort auf den Bogen geschrieben. Neuladen mitten im Kampf ist daher exakt, und der Game Master wird anschließend angewiesen, diese Zahlen nicht erneut zu verändern.

Ein Kampf mit Positionen erscheint auf dem Brett statt auf der Porträtbühne. Siehe Positionen für die Bedienung. Jede Entfernung auf dem Brett, im Menü und im Protokoll verwendet DEINE Einheit, etwa: "Juno bewegt sich für 6 Schritte auf 4, 6 und hat noch 2 Schritte übrig."

### Vergleichende Proben: Packen, Stoßen, Losreißen

Manche Züge sind kein Angriff gegen eine Verteidigung, sondern eine vergleichende Probe: Beide Seiten würfeln, und wer besser abschneidet, setzt sich durch. Jemanden packen, ihn umstoßen oder wegstoßen und sich losreißen haben alle dieselbe Form; ein Regelsatz beschreibt daher jede davon als Daten:

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

- **`checks`** sind die Zahlen, die eine vergleichende Probe liest, jeweils ein Wert vom Bogen, der einmal zu Kampfbeginn gelesen wird, wie eine Verteidigung oder ein Rettungswurf. Eine mit direkten Zahlen geschriebene Kreatur gibt ihre eigenen an (`"checks": { "brawn": 2 }`); eine mit Bogen liest sie daraus. Bis zu zwölf.
- **Der Wurf.** Beide Seiten werfen deine `attackRoll.dice` und addieren den besten der Werte, die sie hier verwenden dürfen (`attacker.checks`, `defender.checks`). Die höhere Summe gewinnt; bei Gleichstand gewinnt der Verteidiger, sofern `ties` nicht `"attacker"` angibt. Das Protokoll nennt beide Seiten: „Juno versucht Grab bei Ash-hound: 12 (6 + 6) + 3 = 15 mit Brawn gegen 2 (1 + 1) + 2 = 4 mit Brawn und gewinnt.“
- **Gewinnen** bewirkt, was `onWin` angibt, mindestens eine Sache:
  - `applies` erlegt dem Verlierer Zustände auf, mit dem Gewinner als Quelle; ein Zustand, den du mit `endsWhenSourceDown` markierst, endet also, wenn der Festhaltende zu Boden geht. `rounds` gibt einem Zustand eine Uhr; ohne diese Angabe hält er, bis etwas ihn beendet.
  - `ends` entfernt Zustände vom Handelnden oder vom Ziel.
  - `push` bewegt den Verlierer so weit in deiner Entfernungseinheit geradewegs vom Gewinner weg und hält vor allem Festen, vor jedem Stehenden, am Rand des Bretts und an einer Ecke, die zu eng zum Hindurchzwängen ist. Die Bewegung ist erzwungen: Sie verbraucht nichts von seiner Bewegung, und niemand greift ihn dabei an. Ein Kampf ohne Brett bewegt niemanden.
    Verlieren bewirkt nichts; genau das ist ein misslungenes Packen.
- **`from: { "holding": "held" }`** richtet eine vergleichende Probe nur auf den, der dem Handelnden diesen Zustand auferlegt hat, und bringt sie nur ins Menü, solange er besteht: So drückt man Losreißen aus.
- **`reach`**, in deiner Entfernungseinheit, gibt an, wie weit sie reicht; ohne Angabe bis in die Nachbarzelle. `reach` und `push` brauchen `combat.distance`.
- **`strike: true`** lässt sie die Stelle eines Angriffs einnehmen, wenn eine Aktion mehrere kauft, so wie ein Angriff: Als Erstes gewählt verbraucht sie das Budget und lässt die übrigen Angriffe vorgemerkt; mit vorgemerkten Angriffen gewählt kostet sie einen davon. Die 5e-Referenz packt und stößt auf diese Weise.
- **Im Menü** hat eine vergleichende Probe eine eigene Gruppe und nennt ihre Gewinnchance. Ein von der Engine gespielter Gegner wägt sie wie alles andere ab, aber zurückhaltend: Packen oder Stoßen bereitet etwas vor, und Losreißen ist am meisten wert, jedoch nur, solange man festgehalten wird. Ein Game Master, der einen Gegner spielt, wählt sie aus demselben Menü.
- Bei einem vom Game Master erfundenen Gegner werden die Probenwerte auf seine Stufe begrenzt, wie seine Trefferchance.

Vergleichende Proben brauchen bei einem als Paket ausgelieferten Regelsatz Capability API 1.43.

<a id="windows-holding-the-fight-open"></a>

### Fenster: Den Kampf für eine Entscheidung anhalten

Manche Momente gehören einer anderen Person als der gerade handelnden. Die Engine hält den Kampf für sie an, statt für sie zu entscheiden. Diese Pause ist ein Fenster.

Sechs Situationen öffnen eines. Zwei davon ergeben sich aus deinen vorhandenen Deklarationen:

- **Jemand löst sich aus dem Nahkampf.** Verlässt eine Bewegung die Reichweite eines angreifenden Gegners, stoppt sie bei diesem Schritt und fragt ihn. Siehe oben "Angriffe auf Weggehende".
- **Zwischen zwei Zügen.** Endet ein Zug, wird vor dem nächsten jeder Gegner mit `signaturePoints` gefragt, der eine eigene `signature`-Aktion bezahlen kann. Nur in diesem Moment werden solche Aktionen gekauft. Sie erscheinen auf keinem Zugmenü, auch nicht dem eigenen.
- **Jemand nutzt etwas.** Vor der Auflösung wird jeder Beteiligte der ANDEREN Seite gefragt, der einen auf diesen Moment wartenden Eintrag besitzt, egal auf wen die Nutzung zielt, sofern dieser Eintrag den Nutzenden erreicht. Eine Standardaktion und eine vergleichende Probe öffnen keinen solchen Moment.
- **Etwas wird auf jemanden gerichtet.** Vor der Auflösung wird jeder anvisierte Beteiligte der ANDEREN Seite gefragt, der einen auf diesen Moment wartenden Eintrag besitzt. Heilung durch einen Freund ist keine zu beantwortende Bedrohung; eine befreundete Aktion öffnet daher kein Fenster.
- **Ein Angriffswurf trifft jemanden.** Vor dem Schaden wird der Getroffene gefragt, wenn er einen auf diesen Moment wartenden Eintrag besitzt. Was er einsetzt, zählt für diesen Angriff; siehe unten „Der Moment nach einem Treffer“.
- **Etwas hat jemanden verletzt.** Nach der Auflösung wird jeder Geschädigte gefragt, der einen auf DIESEN Moment wartenden Eintrag besitzt, unabhängig vom Verursacher. Verletzt zu sein betrifft dich selbst. Ein auf den Verursacher zurückgerichteter Eintrag darf trotzdem nicht auf einen Freund zielen.

Die letzten vier Momente fordert ein Katalogeintrag oder eine eigene Aktion einer Kreatur an, indem er oder sie benennt, worauf gewartet wird. Öffnet eine Aktion sowohl `used` als auch `aimed`, wird zuerst zur Nutzung gefragt; bricht niemand sie ab, werden als Nächstes die Anvisierten gefragt, und aufgelöst wird sie, sobald beide Fragen beantwortet sind. Zum Getroffenwerden wird jedes Mal gefragt, wenn einer ihrer Würfe trifft, zum Verletztwerden erst, wenn die ganze Aktion vorbei ist.

Was ein Fenster unabhängig vom Auslöser tut:

- **Solange es offen ist, geschieht nichts anderes.** Weder handelt die Person am Zug weiter, noch endet ihr Zug, noch öffnet sich ein weiteres Fenster. Der Kampf wartet.
- **Es fragt nacheinander** in Zugreihenfolge, jeden genau einmal. Passen ist immer möglich und kostenlos. Wer befragt würde, aber keine ausführbare Option hat, wird übersprungen.
- **Es setzt exakt am angehaltenen Punkt fort.** Eine Bewegung durchläuft die noch ausstehenden Zellen und bezahlt jede tatsächlich betretene.
- **Die steuernde Instanz antwortet.** Das Fenster deines Gruppenmitglieds gehört dir, mit der Option und **Pass** daneben im Menü. Für einen Gegner antwortet seine steuernde Instanz. Ein Boss des Game Master wird über den Game Master gefragt, mit Verstreichenlassen als möglicher Antwort.
- **Es wird mit dem Kampf gespeichert.** Nach dem Schließen mitten in einer Bewegung bleiben beim Fortsetzen dieselben Personen zu befragen und dieselben Zellen zu durchlaufen.

Für die ersten beiden Fenster deklarierst du nichts zusätzlich: Ein Regelsatz mit `opportunity.budget` erhält das eine, ein Bestiarium mit `signaturePoints` das andere. Ohne beide Angaben erscheinen sie nie.

**Den Moment benennen, auf den ein Eintrag wartet.** Schreibe `mechanics.reaction` als Objekt statt als `true`:

```json
"reaction": { "on": "aimed", "at": "source", "cancels": true }
```

- `on` ist `used`, `aimed`, `hit` oder `harmed` und bringt den Eintrag auf das Menü dieses Fensters. Nur diese vier Momente beobachtet die Engine. Ein Eintrag mit bloßem `"reaction": true` sagt nur, dass er nicht während eines Zuges gewählt wird. Das reicht für kein Angebot; er bleibt in keinem Menü.
- `at` ist `source` (Standard) oder `chosen`. `source` richtet die Aktion auf den Verursacher des Moments und setzt das Ziel selbst, ohne Auswahlfrage. `chosen` behält die eigenen Ziele des Eintrags bei und fragt nach.
- `cancels` verhindert vollständig, was das Fenster angehalten hat. Nur ein `used`- oder `aimed`-Eintrag darf es angeben: Ein bereits eingetretener Moment lässt sich nicht absagen, und ein Treffer ist bereits gewürfelt.
- `against` beschränkt, worauf der Eintrag antwortet, auf Dinge, die aus bestimmten Katalogen genutzt werden: `"against": { "catalogs": ["spells"] }` antwortet auf einen Zauber und auf nichts anderes. Eine Aktion ohne Eintrag dahinter (eine Waffe auf einer Liste, die eigene Aktion einer Kreatur) stammt aus keinem Katalog und öffnet den Moment daher nie für einen Eintrag, der Kataloge nennt. Lässt du `against` weg, antwortet der Eintrag auf alles.

Gib auch ein `budget` an, sonst wird das Standardbudget der Liste verbraucht. Eine Reaktion verwendet fast immer ein eigenes Budget. Das verhindert mehrere davon in einem Zug.

**Die Kosten werden vor der ersten Nachfrage bezahlt.** Eine abgebrochene Aktion wird an ihrer Wirkung gehindert, nicht am Kauf: Budget und Pool-Ressourcen sind bereits verbraucht. Falls dein System sie erstattet, lässt sich das noch nicht ausdrücken.

Ein Konter wird so geschrieben und antwortet auf einem Brett nur jemandem innerhalb seiner eigenen `range`:

```json
"mechanics": {
  "kind": "utility",
  "range": 8,
  "free": true,
  "cost": [{ "pool": "luck", "amount": 1 }],
  "reaction": { "on": "used", "cancels": true, "against": { "catalogs": ["knacks"] } }
}
```

**Der Moment nach einem Treffer.** Mit einem Eintrag für `hit` drückt man Shield, eine Parade und Uncanny Dodge aus. Der Angriff wird nach seinem Wurf angehalten, und der Getroffene wird gefragt, bevor Schaden verursacht wird. Was er einsetzt, zählt für diesen Angriff: Sein Wurf wird nicht wiederholt, aber erneut geprüft, und zwar gegen die Verteidigung des Getroffenen, wie sie jetzt steht. Ein Zustand, der zur Verteidigung addiert, kann den Treffer also in einen Fehlschlag verwandeln, und eine besondere Augenzahl, die immer trifft, trifft weiterhin. Ein Zustand, der einen Angriff lang hält (`endsAfter: "attacked"`) und als Antwort auferlegt wird, deckt diesen Angriff samt seinem Schaden ab und wird durch ihn verbraucht. So drückt man „halbiere den Schaden dieses Angriffs“ aus: `resist-all` für einen Angriff.

```json
"mechanics": {
  "kind": "buff",
  "targets": "self",
  "budget": "reaction",
  "reaction": { "on": "hit" },
  "applies": [{ "condition": "shielded", "duration": { "rounds": 1, "at": "turn-start" } }]
}
```

Der Angriff setzt genau dort fort, wo er angehalten wurde: mit seinen übrigen Zielen und, bei einer aus anderen Aktionen zusammengesetzten Aktion einer Kreatur, mit ihren übrigen Teilen. Jeder Wurf, der jemanden mit einer solchen Antwort trifft, wird an seiner Stelle angehalten, und ein Kampf, der bei offenem Fenster gespeichert wird, kommt genauso zurück. Ein innerhalb eines Fensters ausgeführter Angriff (ein Angriff auf jemanden, der sich entfernt, eine Signaturaktion, eine Antwort) wird nie angehalten, weil nichts, was innerhalb eines Fensters geöffnet wird, ein weiteres öffnet. Das Protokoll zeigt beim Öffnen des Fensters den Wurf und wogegen er gewürfelt wurde und, wenn die Antwort die Verteidigung verändert hat, ob er jetzt verfehlt. Von der Engine gespielte Gegner setzen eine Abwehr nur ein, wenn sie den Treffer in einen Fehlschlag verwandelt.

Ein Paket, das einen solchen Moment nennt, braucht Capability API 1.33, eines mit `used` oder `against` braucht 1.44 und eines, das auf `hit` wartet, 1.46.

### Noch nicht umgesetzt

Die Grenzen werden ausdrücklich genannt, damit ein Regelsatz nichts behauptet, was die Engine nicht tut:

- **Jenseits des einfachen Bretts**: keine Dreiviertel- oder vollständige Deckung, keine Höhenunterschiede, kein Überfliegen von Hindernissen, kein Hindurchzwängen, keine Reittiere, kein Verstecken oder Überraschen, und nichts bewegt jemanden außer seiner eigenen Bewegung und dem Wegstoßen durch eine vergleichende Probe. Niemand zieht eine Kreatur mit sich, die er festhält.
- **Eine vergleichende Probe ist einfach gehalten.** Sie kennt keine Größenbeschränkungen und öffnet kein Fenster (niemand darf darauf antworten).
- **Ein Eintrag kann nur auf vier Momente warten**, `used`, `aimed`, `hit` und `harmed`; siehe Fenster oben. Diese Momente beobachtet die Engine für einen Eintrag. Die beiden anderen Fenster – Weggehen und die Pause zwischen Zügen – öffnet der Kampf selbst; Einträge können sie nicht anfordern. Es gibt keinen Moment für einen Rettungswurf, einen Tod, den Zugbeginn oder einen Sturz.
- **Ein Konter stoppt, worauf er antwortet, vollständig.** Er kann einen Eintrag eines Katalogs nicht von einem anderen unterscheiden, und es gibt keine Probe, um etwas Größeres als ihn selbst aufzuhalten.
- **Keine Verkettung.** Der Kampf hält ein einzelnes Fenster statt eines Stapels. Innerhalb eines Fensters öffnet deshalb nichts ein weiteres: Ein Konter kann nicht selbst gekontert werden, der Schaden einer Reaktion löst keinen weiteren Moment aus, und ein innerhalb eines Fensters ausgeführter Angriff wird bei einem Treffer nicht angehalten.
- **Nur ein Angriffswurf ist ein Treffer.** Was ohne Angriffswurf trifft (Geschosse, die einfach treffen, eine Fläche, gegen die alle einen Rettungswurf ablegen), öffnet keinen `hit`-Moment, und eine Antwort kann einen Hieb nicht von einem Schuss unterscheiden; eine Parade, die die Regeln auf Schwerter beschränken, pariert also auch Pfeile.
- **Schaden wird durch Halbieren verändert, nicht um einen Betrag.** Eine Antwort kann den Schaden eines Angriffs halbieren (`resist-all` für diesen Angriff); sie kann keine gewürfelte Zahl davon abziehen.
- **Es gibt keine Erstattung.** Die Kosten einer abgebrochenen Aktion bleiben verbraucht.
- **Was ein Zustand verändert, ist eine geschlossene Liste.** Verteidigung, Angriffswürfe, Rettungswürfe, Proben in vergleichenden Proben und Geschwindigkeit (und damit auch, was eine Stufe und ein Gegenstand im Kampf verändern), sonst nichts: kein Bonus auf den Schaden, den der Betroffene verursacht, und keine Stufe, die die Höchstwerte eines Charakters senkt oder ihn aus dem Kampf nimmt.
- **Eine mit direkten Zahlen geschriebene Kreatur hat keine Wundleiste.** Verwendet der Regelsatz Gesundheit als Leiste, verliert eine solche Kreatur weiterhin Punkte. Mit einem `sheet` markieren Treffer dagegen Kästchen, nachdem ihre eigenen `resist`, `vulnerable` und `immune` angewendet wurden.
- **Ein Zusatzeffekt löst selbstständig aus.** `on` hat nur den Wert `hit`. Daher verbraucht der erste passende Treffer des Zeitraums den Effekt. Es gibt keinen Moment, in dem du nach seiner Nutzung gefragt wirst.
- **Eine Zahl, die Angriffe verschieben, ist einfach gehalten.** Ein Schlag im ausgebenden Stil wirft die Zahl und sonst nichts; keine Waffe verändert sie, auch nicht ihr `floor` (ein robustes Ziel absorbiert einen Schlag im nehmenden Stil, und seine Härte stoppt einen im ausgebenden Stil, doch sonst verkleinert nichts den einen oder den anderen). Ein Zusammenbruch endet nach einer festen Zahl von Zügen, egal wie tief er war, und ein in einem Fenster ausgeführter Angriff (ein Angriff auf jemanden, der sich losreißt, eine Reaktion, eine Signaturaktion) erfolgt im ersten Stil.
- **Ein erfundener Gegner absorbiert nichts und hat keine Härte.** Eine Kreatur, die der Game Master für einen Kampf erfindet, wird auf deine Bedrohungsskala begrenzt, und diese sagt nichts über Schadensminderung oder Härte aus; die Kreatur hat daher keines von beidem.

## Ebenen: Varianten deines eigenen Regelsatzes

Eine Ebene ist eine benannte Variante deines Regelsatzes, die der Spieler bei der Spielerstellung einschaltet: seltene Magie, ein harter Winter oder ein härterer Schwierigkeitsgrad. Ebenen stehen in einem optionalen `layers`-Array in der Regelsatzdatei. Sie reisen dadurch mit ihr und können einem Spiel, das sie verwendet hat, nie fehlen. Der Assistent zeigt sie als Schalter unter deinem Regelsatz. Die Auswahl gilt wie der Regelsatz selbst für die gesamte Lebensdauer dieses Spiels.

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

**Was eine Ebene kann.** Die Liste ist geschlossen. Jede Wirkung schränkt entweder etwas ein oder fügt Text hinzu:

- `gm.guidance` wird an `gm.checkGuidance` angehängt, nach deinem eigenen Text und nach allen früheren Ebenen. `gm.worldGuidance` wird auf dieselbe Weise an `gm.worldGuidance` angehängt.
- `fields` entfernt Werte aus einem **Enum**-Feld. `removeValues` nennt bereits vorhandene Feldwerte. Mindestens einer muss übrig bleiben. Wird auch `default` des Feldes entfernt, nennt die Ebene einen verbleibenden Wert als neuen `default`.
- `difficultyLadder` ersetzt deine Schwierigkeitsskala durch eine andere in der Form deiner eigenen Auflösungsart: `{label, dc}` für `dice-sum` und `{label, successes, target?}` für `dice-pool`. Es gelten genau dieselben Prüfungen wie für deine eigene Skala. Deklarieren mehrere aktive Ebenen eine, gewinnt die letzte.
- `catalogs` blendet Einträge aus der Auswahl des Bogeneditors aus. Jede Regel nennt einen der deklarierten `filters` dieses Katalogs und genau einen Vergleich: `above` oder `below` für einen `number`-Filter, `equals` oder `notIn` für `text` oder `tags`. Ein Eintrag, der den Filter überhaupt nicht setzt, wird niemals ausgeblendet.
- `currencies` nimmt Münzen heraus: `removeUnits` nennt einzelne Münzen und `removeFamilies` ganze Familien. Die kleinste Münze einer Familie fällt nur mit ihrer ganzen Familie weg, damit jede verbleibende Familie weiterhin bezahlen und Wechselgeld geben kann. Solange die Ebene aktiv ist, verdient, bezahlt, wählt oder erbeutet niemand eine herausgenommene Münze, und die Geldzeile lässt sie weg. Ein Preis, der in einer solchen Münze angegeben ist, wird zum selben Wert in der größten verbleibenden Münze genannt, die ihn genau bezahlt: Die Ebene „The long night“ in Gravewatch nimmt die crown heraus, und eine Wachpistole für 3 crowns kostet dann 15 shillings. Ein Gegenstand, dessen ganze Familie fehlt, hat keinen Preis.

**Was eine Ebene nicht kann.** Sie kann weder Enum-Werte, Felder, Fertigkeiten, Pools oder Rasten hinzufügen noch die Auflösungsart ändern, laufenden Zustand oder Kampfzahlen verändern oder einen Modellaufruf ergänzen. Ein von einer Ebene _hinzugefügter_ Wert wäre allen anderen Lesern des Bogens unbekannt. Deshalb können Werte nur entfallen. Alles außerhalb dieser Liste ist eine Änderung des Regelsatzes selbst oder ein zweiter Regelsatz.

**Konflikte.** `conflicts` nennt Ebenen, die nicht gleichzeitig aktiv sein dürfen. Die Angabe auf einer Seite des Paares reicht. Der Assistent deaktiviert den anderen Schalter. Enthält eine gespeicherte Auswahl dennoch beide, entfällt die **später** deklarierte. Dieselben zwei Auswahlen ergeben dadurch immer dieselben Regeln.

**Ein Bogen mit einem entfernten Wert behält ihn.** Kein Charakter wird umgeschrieben. Der Editor bietet den Wert lediglich nicht mehr an; bei einem Charakter, der ihn schon hatte, zeigt er ihn weiterhin an. Wird die Ebene in einem neuen Spiel ausgeschaltet, steht der Wert wieder zur Auswahl. Dasselbe gilt für einen ausgeblendeten Katalogeintrag: Er fehlt in der Auswahl, doch bereits ausgewählte Zeilen bleiben auf dem Bogen. Münzen, die eine Ebene herausnimmt und die jemand bereits mitführt, bleiben in seiner Tasche.

**Grenzen.** 12 Ebenen pro Regelsatz und 4000 Zeichen Anleitungstext je Ebene, beide Zeichenfolgen zusammengezählt. Ein als Paket ausgelieferter Regelsatz mit `layers` oder einem grundlegenden `gm.worldGuidance` braucht Capability API 1.25, und einer, dessen Ebenen Münzen herausnehmen, braucht 1.64. Ein importierter Regelsatz wird von der lesenden Engine geprüft und braucht keine solche Angabe.

**Von anderen geschriebene Ebenen**, etwa eine separat ausgelieferte Ebene für seltene Magie zu einem fremden Regelsatz, sind für später vorgesehen. Derzeit wird eine Ebene im zugehörigen Regelsatz ausgeliefert.

<a id="trying-your-ruleset"></a>

## Deinen Regelsatz ausprobieren

Community-Regelsätze verwenden denselben Schalter wie importierte Agenten. Öffne **Settings** > **Advanced** > **Danger Zone** (Einstellungen > Erweitert > Gefahrenbereich) und aktiviere **Allow custom Agent imports** (eigene Agentenimporte erlauben). Der Import braucht außerdem Zugriff über localhost oder eingerichteten **Admin Access** (Administratorzugriff).

1. Öffne das Panel **Agents** und wähle **Import agents** (Agenten importieren), das Download-Symbol in der oberen Schaltflächenreihe.
2. Wähle **Game Mode ruleset** (Regelsatz für Game Mode) und deine JSON-Datei.
3. Lies die Prüfung. Sie zeigt Name, Version, Lizenz, Umfang des Regelsatzes und den Text für den Game Master. Wähle **Import** (importieren).

Dein Regelsatz erscheint im Bereich **Rules** des Panels und in der Auswahl **Rules** des Assistenten für neue Spiele. Ein aus einer Datei importierter Regelsatz wird als `local/<your id>` abgelegt. Er kann dadurch nie mit einem offiziellen oder fremden Regelsatz verwechselt werden.

### Einen bereits importierten Regelsatz ändern

Eine importierte Version wird niemals überschrieben. Änderst du die Datei und importierst sie mit derselben `version` erneut, wird der Import abgelehnt und fordert eine höhere Nummer. Das ist beabsichtigt: Ein Spiel ist an genau die Version gebunden, mit der es erstellt wurde. Eine laufende Kampagne wacht deshalb nie mit anderer Mathematik auf.

Beim Entwerfen lautet die Folge daher: bearbeiten, `version` erhöhen, importieren, neues Spiel beginnen. Alte Versionen bleiben neben der neuen installiert, bis du den Regelsatz aus dem Bereich **Rules** entfernst. Verwendet ein Spiel einen entfernten Regelsatz noch, meldet es ihn als fehlend, bis du ihn erneut importierst.

Änderst du die Form des Bogens durch Hinzufügen, Entfernen oder Umbenennen von Einträgen, erhöhe auch `sheet.version`. Vorhandene Bögen werden tolerant gelesen: Dem neuen Bogen unbekannte Werte bleiben erhalten, fehlende Werte verwenden ihre Standards.

## Deinen Regelsatz teilen

**Als Datei.** Schick die JSON-Datei an einen Freund. Er importiert sie genauso wie du.

**Aus einem GitHub-Repository.** Verwahrst du deine Arbeit in einem öffentlichen GitHub-Repository, lege jeden Regelsatz als eigene Datei in einen Ordner `rulesets` auf dessen oberster Ebene:

```text
your-repository/
  agents.json        (optional, only if you also share agents)
  rulesets/
    ember-roads.json
    another-system.json
```

Nutzer fügen dein Repository einmal über die Liste eigener Agenten-Repositories hinzu, prüfen seinen Inhalt und können es später synchronisieren, um neue Versionen zu erhalten. Diese Repository-Liste ist eine erweiterte Funktion, die der Serverbetreiber mit `ENABLE_CUSTOM_AGENT_REPOS=true` einschalten muss. Regelsätze aus einem Repository werden unter dem Namen seines Besitzers abgelegt, etwa `alice/ember-roads`. So können zwei Autoren jeweils einen Regelsatz namens `v20` veröffentlichen, ohne einander zu überschreiben.

Es gelten zwei Grenzen. Ein Repository darf höchstens 32 JSON-Dateien direkt in `rulesets` enthalten; mehr führen zur Ablehnung. Ein Konto namens `local` kann keine Regelsätze veröffentlichen, denn `local/` ist für Dateiimporte reserviert.

**Im offiziellen Katalog.** Ein verbreitetes System mit geklärten Lizenzrechten kann über **Download Agents** (Agenten herunterladen) allen angeboten werden. Dafür öffnest du einen Pull Request im Repository [Marinara-Agents](https://github.com/Pasta-Devs/Marinara-Agents). Das dortige Paket `ruleset-5e-2014` zeigt die Verzeichnisstruktur.

## Lizenzierung

Veröffentliche nur Regeltexte, die du teilen darfst. Viele Systeme bieten ein Referenzdokument unter einer offenen Lizenz an; daraus darfst du kopieren. Trage unter `license` die Lizenz-ID und den verlangten Urheberhinweis ein. Kopiere keine Texte aus nicht offen lizenzierten Regelbüchern. Ein Regelsatz braucht vor allem Namen und Zahlen. Den Text für den Game Master solltest du selbst formulieren.

## Fehlerbehebung

- **Der Import meldet einen unbekannten Namen.** Etwas in der Datei verweist auf eine nicht deklarierte ID, etwa eine Fertigkeit auf ein entferntes Attribut. Die Meldung nennt den Pfad zur betroffenen Zeile.
- **Der Import meldet eine bereits installierte Version mit anderem Inhalt.** Erhöhe `version` und importiere erneut.
- **Mein Regelsatz fehlt im Einrichtungsassistenten.** Prüfe, ob **Allow custom Agent imports** eingeschaltet ist. Solange der Schalter aus ist, werden importierte Regelsätze für neue Spiele ausgeblendet. Bereits damit erstellte Spiele funktionieren weiter.
- **Ein Spiel meldet seinen Regelsatz als fehlend.** Die genaue Version, mit der es erstellt wurde, ist nicht installiert. Importiere diese Dateiversion erneut.
