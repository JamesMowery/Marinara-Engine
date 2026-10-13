# Eigene Schriftarten und Google Fonts

In dieser Anleitung erfährst du, wie du die Schriftart wechselst, die Marinara Engine überall in der App verwendet. Zur Wahl stehen die mitgelieferte Schriftart, eigene Schriftdateien oder ein Download von Google Fonts über den Namen.

## Schriftart für die App wählen

Die Einstellung dafür steckt in **Settings** (Einstellungen), im Tab **Appearance** (Darstellung), im Abschnitt **Text & Scale** (Text und Skalierung).

1. Öffne **Settings** und klick auf den Tab **Appearance**.
2. Suche den Abschnitt **Text & Scale**.
3. Öffne das Dropdown-Menü **Font** (Schriftart).
4. Wähle eine Schriftart aus der Liste.

Standardmäßig ist **Default (Inter)** eingestellt. Inter ist eine klare Schriftart, ausgelegt aufs Lesen am Bildschirm. Eigene Schriftarten landen im selben **Font**-Dropdown-Menü, unterhalb der Standardoption.

Solange die Standardschriftart eingestellt ist, verwenden die Prompt-Felder im Preset-Editor und das Feld Example Dialogue eines Charakters eine Monospace-Schriftart. Wählst du irgendeine andere Schriftart, übernehmen sie diese ebenfalls. Andere Prompt-Felder, etwa die Prompt-Vorlage eines Agenten, bleiben bei der Monospace-Schriftart.

Chat-Widgets haben unter **Settings > Appearance > App > Chat widget style** (Einstellungen, Darstellung, App, Chat-Widget-Stil) eine eigene **Font**-Auswahl. Schaltest du dort **Apply preset font** (Preset-Schrift anwenden) ein, gilt diese Auswahl auch für Nachrichten, Eingabefelder und Chat-Bedienelemente. Lässt du den Schalter aus, behalten diese Bereiche ihre übliche Schriftart. Siehe [Fertige Stile für Chat-Fenster](custom-css-themes.md#ready-made-chat-window-styles).

Die gewählte Schriftart wird zwischen Geräten synchronisiert. Sobald du sie änderst, stellt jeder Browser und jedes Gerät am selben Marinara-Server ebenfalls um. Wie diese Synchronisierung funktioniert, erklärt die Anleitung [Übersicht der Einstellungen](../settings/settings-overview.md).

## Eigene Schriftarten hinzufügen

Eigene Schriftarten kommen als Datei in einen Ordner auf dem Server. Damit ist der Rechner gemeint, auf dem Marinara läuft.

1. Suche auf dem Server-Rechner den Ordner `data/fonts/` im Datenordner von Marinara.
2. Kopiere die Schriftdatei in diesen Ordner.
3. Geh zurück zu **Settings**, dann **Appearance**, dann **Text & Scale**.
4. Öffne das **Font**-Dropdown-Menü. Die Schriftart steht jetzt in der Liste.
5. Wähle sie aus.

Marinara liest diese Dateitypen: `.ttf`, `.otf`, `.woff` und `.woff2`. Dateien mit anderer Endung bleiben unberücksichtigt.

Den Anzeigenamen leitet Marinara aus dem Dateinamen ab. Aus `OpenSans-Bold.ttf` wird zum Beispiel „Open Sans“. Benenne die Dateien also sprechend, wenn du eine aufgeräumte Liste willst.

Schriftdateien im Ordner `data/fonts/` liegen auf dem Server. Jedes Gerät am selben Marinara-Server kann sie nutzen. Auch die Auswahl selbst synchronisiert sich über diese Geräte hinweg, sodass überall dieselbe Schriftart erscheint.

## Von Google Fonts herunterladen

Marinara holt eine Schriftart auf Wunsch direkt von Google Fonts. Dafür braucht der Server Internetzugang.

1. Öffne **Settings**, dann **Appearance**, dann **Text & Scale**.
2. Suche das Feld **Google Fonts**.
3. Tipp den exakten Namen der Schriftart ein, etwa `Fira Code` oder `Lora`.
4. Klick auf **Add** (Hinzufügen).
5. Warte, bis der Download durch ist. Danach taucht die neue Schriftart im **Font**-Dropdown-Menü auf.

Schreib den Namen genau so, wie Google Fonts ihn schreibt. Direkt neben dem Feld sitzt der Link **Browse fonts at fonts.google.com** (Schriftarten auf fonts.google.com durchsuchen). Er öffnet die Google-Fonts-Website in einem neuen Tab, damit du Namen nachschlagen kannst.

Erlaubt sind ausschließlich Buchstaben, Ziffern und Leerzeichen. Lädst du dieselbe Schriftart später erneut herunter, ersetzt Marinara die alte Kopie, statt ein Duplikat anzulegen.

Scheitert der Download, lies die Fehlermeldung. Erreicht Marinara Google Fonts nicht, folgt der Hinweis, die Internetverbindung zu prüfen. Heißt es dagegen, die Schriftart sei nicht gefunden worden, kommen zwei Ursachen infrage. Entweder passt der Name zu keiner Schriftart bei Google Fonts. Oder die Schriftart hat keinen regulären Schnitt (400), also den normalen, nicht fetten Stil. Prüfe die Schreibweise und schau auf der Google-Fonts-Website nach, ob es überhaupt einen Regular-Schnitt gibt.

## Open Fonts Folder wirkt nur lokal

Neben dem **Font**-Dropdown-Menü liegt die Schaltfläche **Open Fonts Folder** (Schriftarten-Ordner öffnen). Sie öffnet den Ordner `data/fonts/` im Dateimanager des Server-Rechners.

Die Schaltfläche wirkt auf dem Server, nicht auf dem Gerät, mit dem du Marinara gerade ansiehst. Läuft Marinara auf deinem eigenen Rechner, öffnet sich der Ordner direkt vor dir. Verbindest du dich vom Handy oder von einem zweiten Rechner aus, bringt die Schaltfläche dir nichts. Kopiere die Schriftdateien dann selbst in den Ordner `data/fonts/` auf dem Server.

## Verwandte Anleitungen

- [Darstellungs-Einstellungen](appearance-settings.md)
