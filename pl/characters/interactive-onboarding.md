# Interaktywny onboarding

Ten przewodnik wyjaśnia, czym jest **interaktywny onboarding** – funkcja karty postaci, dzięki której gracz na początku czatu Roleplay tworzy swoją personę, odpowiadając na pytania autora karty. Opisuje, jak autor konfiguruje ją w edytorze **Character Editor** (edytor postaci) i co gracz widzi w kreatorze nowego czatu.

## Do czego służy

Persona to postać, która reprezentuje cię w czacie. Zobacz [Persony użytkownika](personas.md). Zwykle personę piszesz samodzielnie przed rozpoczęciem gry. Przy interaktywnym onboardingu karta postaci dostarcza zamiast tego szablon persony. Szablon to pięć pól persony – **Description** (opis), **Personality** (osobowość), **Backstory** (historia postaci), **Appearance** (wygląd) i **Scenario** (scenariusz) – z lukami do wypełnienia przez gracza oraz pytania, które te luki wypełniają.

Kiedy gracz zaczyna czat Roleplay z tą kartą, może wybrać **Create your persona** (utworzenie własnej persony), odpowiedzieć na pytania i dostać nową, zwykłą personę. Odpowiedzi zostają wpisane do tekstu jeden raz. Wynikiem jest zwykła persona, którą gracz może edytować, zachować albo usunąć jak każdą inną. Sama karta się nie zmienia.

Onboarding korzysta z istniejącej składni makr. Nie uruchamia żadnych skryptów.

## Konfiguracja po stronie autora

1. Otwórz postać w edytorze **Character Editor**. Zobacz [Tworzenie i edycja postaci](creating-and-editing-characters.md).
2. Otwórz zakładkę **Onboarding** obok zakładki **Card**.
3. Włącz przełącznik **Interactive onboarding** (interaktywny onboarding). Pod przełącznikiem pojawia się krótki poradnik **How onboarding works** (jak działa onboarding).

Wyłączenie przełącznika ukrywa pola onboardingu, ale zachowuje ich treść.

### Dodawanie pytań

W sekcji **Onboarding Questions** (pytania onboardingu) kliknij **Add Question** (dodanie pytania). Każde pytanie działa jak zmienna presetu. Zobacz [Edytor presetów i menedżer promptów](../prompts/presets.md). Karta może mieć do 100 pytań.

- **Variable Name** (nazwa zmiennej) to nazwa, której używasz w polach, na przykład `faction`. Wybierz dowolną nazwę z liter, cyfr i podkreśleń, która zaczyna się od litery albo podkreślenia. Wielkość liter w nazwach ma znaczenie: `Gender` i `gender` to dwie różne nazwy.
- **Question (shown to user)** (pytanie widoczne dla użytkownika) to tekst, który czyta gracz. Może korzystać z wcześniejszych odpowiedzi, na przykład `What does {{player}} look like?`. Tutaj `{{char}}` to nazwa karty, a `{{user}}` to imię gracza.
- **Options** (opcje) zamieniają pytanie w wybór. Domyślnie zaznaczona jest pierwsza opcja. Bez opcji gracz wpisuje dowolną odpowiedź tekstową.
- **Value** (wartość) to tekst, który trafia do persony. Wielkie i małe litery zostają takie, jak je wpiszesz; makra w tym polu wypełniają się tak samo jak w pozostałych polach. Zostaw je puste, żeby użyć etykiety opcji. Jeśli odpowiedź trafia do środka zdania, wpisz wartość małymi literami, na przykład `ranger`.
- **Help text for players** (tekst pomocy dla graczy) to opcjonalny tekst wyświetlany pod opcją.
- **Allow own answer** (zezwolenie na własną odpowiedź) dodaje obok twoich opcji wybór **Write your own** (własna odpowiedź).
- **Multi-Select** (wybór wielokrotny) pozwala graczowi wybrać kilka opcji. Łączy je separator.

Pytanie o nazwie `player` ustawia imię persony. Jeśli go nie dodasz, gracze najpierw są pytani o swoje imię.

Edytor ostrzega, gdy nazwa jest pusta, nieprawidłowa, użyta dwa razy albo zajęta przez wbudowane makro, a także gdy żadne pole nie korzysta z danego pytania.

### Pisanie pól persony

Wypełnij pięć pól jak we własnej personie, a w miejscu odpowiedzi wpisz `{{name}}`:

```
Name: {{player}}
Class: {{class}}
Role: {{player}} is a {{class}} sworn to {{faction}}.
```

Po kliknięciu w pole pod nim pojawia się lista **Available variables** (dostępne zmienne). Kliknij zmienną, żeby wstawić ją w miejscu kursora.

Użyj `{{#if}}`, żeby pokazać tekst tylko przy niektórych odpowiedziach:

```
{{#if knowsHer == yes}}Grew up next door to {{char}}. {{relationToAna}}{{/if}}
```

Pytanie, które występuje tylko wewnątrz `{{#if}}`, pada tylko wtedy, gdy ten warunek jest spełniony. W przykładzie `relationToAna` to pytanie uzupełniające, które pojawia się, gdy gracz odpowie **yes**. Porównania nie rozróżniają wielkich i małych liter.

`{{user}}` i `{{char}}` zostają w utworzonej personie jako makra, więc działają tak jak w każdej personie. Inne makra, takie jak `{{random}}`, `{{time}}` czy `{{getvar}}`, wypełniają się jeden raz przy tworzeniu persony albo zostają puste, jeśli nie mają wtedy wartości.

**Rendered preview** (podgląd wyniku) na dole pokazuje personę zbudowaną z przykładowych odpowiedzi i wypisuje kolejno zadawane pytania.

## Tworzenie persony przez gracza

1. Zacznij nowy czat Roleplay. W kroku **Persona & Characters** (persona i postacie) karty z onboardingiem mają na liście postaci małą ikonę osoby z plusem.
2. Dodaj postać. Na górze pola **Your Persona** (twoja persona) pojawia się wiersz **Create your persona** z nazwą karty, do której należy.
3. Wybierz ten wiersz i kliknij **Next** (dalej).
4. Okno **Create your persona** pokazuje pytania karty. Pytania uzupełniające pojawiają się, gdy tylko twoje odpowiedzi sprawią, że mają znaczenie.
5. Kliknij **Create persona** (utworzenie persony). Nowa persona zostaje zapisana i wybrana dla czatu, a kreator przechodzi dalej.

Jeśli wybierzesz inną personę albo opcję **None** (brak) z podpisem Stay anonymous, pytania nie padają. **Back** (powrót) w oknie pytań wraca do wyboru persony. Każda karta z onboardingiem w czacie ma własny wiersz **Create your persona**.

Onboarding jest dostępny w kreatorze nowego czatu Roleplay. Quick Start, konfiguracja czatu Conversation i rozpoczęcie czatu prosto z karty postaci jeszcze go nie oferują.

## Udostępnianie kart z onboardingiem

Onboarding jest zapisany razem z kartą. Eksport w formatach **Marinara Native** i **Compatible PNG Card** go zachowuje, więc gracz, który zaimportuje kartę, też dostaje pytania. Eksport w formacie **Compatible JSON** go pomija. Karta zaimportowana z ponad 100 pytaniami zachowuje pierwsze 100. Zobacz [Importowanie i eksportowanie kart postaci](import-export.md).

## Powiązane przewodniki

- [Tworzenie i edycja postaci](creating-and-editing-characters.md)
- [Persony użytkownika: tworzenie i edycja](personas.md)
- [Wybór persony w czacie](choosing-your-persona.md)
- [Makra promptów](../prompts/macros.md)
- [Importowanie i eksportowanie kart postaci](import-export.md)
