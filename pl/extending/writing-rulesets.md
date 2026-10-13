# Pisanie zestawów zasad Game Mode

Zestaw zasad mówi trybowi Game Mode, jak działa system stołowy: jakie kości rzuca test, co znajduje się w arkuszu postaci, jakie zasoby są wydawane i co odnawia odpoczynek. Ten przewodnik jest dla osób chcących napisać i udostępnić własny zestaw. Jeśli chcesz grać z zestawem kogoś innego, zacznij od [Wyboru zasad](../game/getting-started.md#choosing-rules).

Zestaw zasad jest jednym plikiem JSON. Zawiera dane, nie kod. Nic się w nim nie wykonuje, więc import nie może niczego zrobić z twoim komputerem. Przed importem cudzego pliku dokładnie przeczytaj tekst Game Master: trafia do modelu w każdej grze używającej zestawu.

## Najpierw przeczytaj: co zestaw zasad może, a czego nie może

Zestaw może jedynie uzupełniać mechanikę, którą Engine już zna. Obecnie Engine zna dwa sposoby rozstrzygania testu; plik wybiera jeden przez `resolution.kind`:

- **`dice-sum`**: rzuć kośćmi, dodaj liczby z arkusza i osiągnij lub przekrocz trudność. Obejmuje systemy d20, systemy 2d6 plus cecha i wiele innych.
- **`dice-pool`**: rzuć liczbą kości wynikającą z postaci i policz te, które osiągnęły próg. Obejmuje systemy, w których wartość oznacza garść kości zamiast premii.

Oba dokładnie opisuje sekcja [Rodzaje rozstrzygania](#resolution-kinds).

Mechaniki niepasującej do żadnej z tych postaci nie da się zapisać w pliku zestawu. Przykłady to branie najwyższej kości puli, testy procentowe z wynikiem poniżej progu, kości z symbolami i przeciwstawne pule. Każda wymaga nowego rodzaju rozstrzygania w Engine, czyli wkładu w kod z testami, nie pliku JSON. Jeśli twój system go potrzebuje, zgłoś prośbę o funkcję w repozytorium Engine i opisz mechanikę kilkoma w pełni rozpisanymi rzutami. Te przykłady staną się testami.

Game Mode może rozstrzygać walkę według własnych zasad Marinara albo twojego zestawu. Opcjonalny blok `battle` przekazuje walce Marinara liczby z arkuszy postaci: zobacz [Bitwy](#battles-lending-the-sheet-to-marinaras-combat). Opcjonalny `combat` zamiast tego definiuje rozstrzyganie walki według zestawu: zobacz [Walkę](#combat-a-fight-your-own-rules-resolve). Engine już stosuje te zasady. Ustawienie **Combat Preference** (preferowany rodzaj walki) gry wybiera prezentację Classic lub, gdy zestaw definiuje odległość, pole bitwy Tactical.

## Szybki start

1. Skopiuj przykład pasujący do rzutów twojego systemu. [`ember-roads.json`](https://github.com/Pasta-Devs/Marinara-Engine/blob/staging/docs/examples/rulesets/ember-roads.json) to niewielki system 2d6 z trzema cechami, pokazujący, że format nie zakłada d20 ani sześciu atrybutów. [`gravewatch.json`](https://github.com/Pasta-Devs/Marinara-Engine/blob/staging/docs/examples/rulesets/gravewatch.json) to mała pula kości dziesięciościennych z trzema cechami i sześcioma umiejętnościami zawodowymi. Pełnowymiarowy przykład 5e (SRD 5.1) znajdziesz w [`ruleset-5e-2014.example.json`](https://github.com/Pasta-Devs/Marinara-Engine/blob/staging/docs/development/ruleset-5e-2014.example.json).
2. Zmień `id` na własne. Identyfikator zawiera małe litery, cyfry i pojedyncze łączniki, np. `ember-roads`.
3. Edytuj arkusz, odpoczynki i tekst Game Master.
4. Zaimportuj plik (zobacz [Wypróbowanie zestawu](#trying-your-ruleset)). Import sprawdza cały plik i wskazuje błędy wiersz po wierszu, zanim cokolwiek zapisze.
5. Utwórz nową grę, wybierz zestaw w **Rules** (zasady) i wykonaj kilka testów.

Aby uzyskać pomoc podczas pisania, wskaż edytorowi JSON Schema, dodając tę pierwszą linię wewnątrz zewnętrznych klamer pliku:

```json
"$schema": "https://raw.githubusercontent.com/Pasta-Devs/Marinara-Engine/staging/docs/extending/ruleset.schema.json",
```

Schemat podczas pisania wyłapuje błędne nazwy kluczy i typy. Nie sprawdza, czy nazwy w pliku wskazują istniejące rzeczy, np. czy umiejętność podaje istniejący atrybut. Robi to import.

Do dowolnego obiektu pliku możesz dodać linię `"$comment": "..."` z własną notatką. Engine ją ignoruje.

## Części pliku

| Klucz | Zawartość |
| --- | --- |
| `schemaVersion` | Zawsze `1`. |
| `id`, `version` | Nazwa zestawu dla Engine i liczba całkowita zwiększana przy każdej publikacji zmian. |
| `name` | To, co gracze widzą w kreatorze konfiguracji. |
| `edition` | Opcjonalne. Jedna linia określająca wydanie lub szkic. |
| `license` | Opcjonalne. Identyfikator SPDX i wymagany przez źródło tekst uznania autorstwa. |
| `coverage` | Zakres zestawu i jednoliniowy opis widoczny w kreatorze. |
| `resolution` | Sposób rzutu testu lub rzutu obronnego. |
| `sheet` | Cała zawartość arkusza postaci. |
| `rests` | Co odnawia i usuwa każdy rodzaj odpoczynku. |
| `gm` | Tekst dla modelu Game Master i wartości arkusza widoczne dla każdej postaci. |
| `catalogs` | Opcjonalne. Gotowe wpisy oferowane przez edytor arkusza, aby gracze nie przepisywali długich list. |
| `items` | Opcjonalne. Słowa, którymi opisuje się twoje przedmioty: kategorie, rzadkości, statystyki, miejsca, pieniądze. |
| `battle` | Opcjonalne. Dane, które bitwa może odczytać z arkusza i zapisać po zakończeniu. |
| `combat` | Opcjonalne. Sposób rozstrzygania walki według własnych zasad i to, co pokazuje ekran bitwy. |
| `layers` | Opcjonalne. Warianty zestawu włączane przez gracza przy tworzeniu gry. |

Plik może mieć do 256 KB. Tekst trafiający do promptu (nazwy, etykiety, tekst Game Master) nie może zawierać nowych linii, nawiasów kwadratowych ani podwójnych klamer.

Identyfikatory w arkuszu (atrybutów, umiejętności, pól, pul itd.) zaczynają się literą i zawierają małe litery, cyfry oraz podkreślenia, np. `grit_max`.

<a id="resolution-kinds"></a>

### Rodzaje rozstrzygania

`resolution.kind` wybiera sposób rzutu testu. Oba rodzaje czytają ten sam arkusz postaci i współdzielą trzy klucze, więc zmiana rodzaju nie zmienia części pliku poniżej `resolution`:

- `abilityModifier`: jak wynik w arkuszu staje się liczbą. `identity` oznacza, że wynik jest tą liczbą. `floorHalfMinusTen` to reguła 5e. `stepTable` pozwala podać własne progi jako `[[score, number], ...]`.
- `proficiencyTiers`: poziomy wyszkolenia umiejętności lub rzutu obronnego. Pierwszy dostaje niewymieniona umiejętność. Poziom dodaje `flat`, albo `multiplier` razy premię biegłości, albo oba. Jeśli system ma premię biegłości, wskaż jej źródło przez `"proficiency": { "bonus": { "derived": "proficiency_bonus" } }`.
- `proficiency`: opcjonalne, potrzebne tylko poziomowi używającemu mnożnika.

Znaczenie otrzymanej liczby zależy od rodzaju: `dice-sum` dodaje ją do rzutu, `dice-pool` rzuca tyloma kośćmi.

#### `dice-sum`: dodawanie kości

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

- `dice`: liczba kości i ich ścianek. Sumę porównuje się z trudnością.
- `advantage`: czy Game Master może poprosić o dwa rzuty i zachowanie jednego.
- `naturals`: wpływ najwyższej i najniższej ścianki pojedynczej kości na testy i rzuty obronne: `none`, `both`, `max-only` lub `min-only`. Pomiń dla samej arytmetyki. Wymaga pojedynczej kości, więc system 2d6 musi używać `none`.
- `difficultyLadder`: trudności, spośród których ma wybierać Game Master. `dc` to liczba, którą suma musi osiągnąć. Zamiast liczby Game Master może podać nazwę stopnia, np. `difficulty="Hard"`, a Engine odczyta liczbę z tego stopnia.

#### `dice-pool`: rzucanie kości i liczenie wyników

Liczba z arkusza to **wielkość puli**, a nie premia dodawana do niej. Cecha 3 i umiejętność zawodowa warta 2 dają pięć kości. To cała zasada: bez nowego słownika arkusza, bez nowego edytora; system, którego wartości są garściami kości, zapisuje się tymi samymi `abilities`, `skills` i `proficiencyTiers` co każdy inny.

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

- `die`: liczba ścianek jednej kości puli, od 2 do 100.
- `pool`: zakres, do którego ogranicza się liczbę z arkusza przed eksplozjami. `min` równe 0 pozwala pustej puli przegrać bez rzutu, a `max` może wynosić najwyżej 100. `abilityPlusAbility: true` pozwala testowi atrybutu wskazać przez `with=` drugi atrybut i rzucić kośćmi za oba, co służy systemom rzucającym dwoma atrybutami naraz; bez tego `with=` nic nie znaczy w teście atrybutu.
- `target`: wynik kości wymagany do jej zaliczenia. Ustaw `min` poniżej `max`, aby Game Master mógł zmieniać go dla testu przez `threshold=`; równe trzy wartości ustalają stały próg.
- `double`: opcjonalne. Wynik co najmniej `from` liczy się podwójnie.
- `explode`: opcjonalne. Wynik co najmniej `from` daje dodatkową kość, która także może eksplodować. Dodatkowych kości może być najwyżej `pool.max` ponad samą pulę, więc jeden test rzuca najwyżej dwukrotnością `pool.max`, a niskie `from` nie pozwala rzucać bez końca.
- Obie reguły mogą mieć `min`: najniższy wynik, do którego test może obniżyć regułę przez `explode="N"` lub `double="N"` w znaczniku albo przez wpis użyty przez postać. Z `min` można pominąć `from`; reguła działa wtedy tylko w teście, który o nią prosi. Specjalizacja, dzięki której dziesiątki dają dodatkową kość w jednej umiejętności, to `"explode": { "min": 10 }`. `from` nie może być niższe niż `min`.
- `cancel`: opcjonalne. Wynik nie większy niż `upTo` odejmuje jeden sukces. Liczba sukcesów nie spada poniżej zera.
- `botch`: opcjonalne. Jeśli **żadna** kość nie dała sukcesu, a pojawił się wynik nie większy niż `upTo`, test jest krytyczną porażką. Pula, której jedyny sukces anulowano, ponosi zwykłą, nie krytyczną porażkę. `"rule": "halfOrMore"` odczytuje to inaczej: wyniki nie większe niż `upTo` na co najmniej połowie kości rzuconych na początku (bez eksplozji) oznaczają krytyczną porażkę, gdy żadna kość nie dała sukcesu, a w przeciwnym razie zostawiają wynik bez zmian i oznaczają go `complication="true"`: obok wyniku coś idzie nie tak. Wartość domyślna to `"noSuccesses"`.
- `exceptional`: opcjonalne. Tyle sukcesów netto lub więcej w udanym teście oznacza krytyczny sukces.
- `situationalDice`: opcjonalne. Zakres kości dodawanych lub odejmowanych przez Game Master dla jednego testu przez `bonus=`, np. za wyczyny, rany lub słabe światło.
- `reroll`: opcjonalne. Ponowne rzuty przyznawane przez system bez żadnej opłaty, do sześciu, każdy z `id`, które Game Master podaje w teście przez `reroll=`. Wyniki nie większe niż `upTo` rzuca się ponownie: `"once"` oznacza, że nowy wynik zostaje, jakikolwiek jest, a `"until"`, że rzuca się nimi, aż wypadnie wyższy wynik. `upTo` wynosi od 1 do liczby o jeden mniejszej niż liczba ścianek kości, ponieważ ponowny rzut każdego wyniku nigdy by się nie skończył, a `until` i tak zatrzymuje się po stu ponownych rzutach w jednym teście. Żaden nie jest wykonywany, jeśli test go nie wskazuje. Jeden test wykonuje jeden ponowny rzut: jeśli wydatek lub wpis użyty przez postać także go kupił, wykonuje się ten, który obejmuje więcej wyników, a przy równym zasięgu `until` ma pierwszeństwo przed `once`.
- `difficultyLadder`: `successes` określa liczbę wymaganych sukcesów. Stopień może też wskazywać `target`, ale tylko gdy próg jest regulowany i w jego zakresie. Z tego progu korzysta test na danym stopniu: gdy Game Master wskazuje stopień przez `difficulty=` albo zapisuje `dc=` wymagane przez dokładnie jeden stopień. Gdy kilka stopni wymaga tej samej liczby sukcesów, samo `dc=` nie określa, o który chodzi, więc próg pozostaje domyślny; podaj nazwę stopnia.

Wyniki `cancel` i `botch` muszą być niższe niż najniższy próg, a każdy wynik wskazany przez te reguły musi istnieć na kości. Regułę, która nigdy nie zadziała, odrzuca import, zamiast wykrywać problem podczas gry.

Zestaw z pulą dostarczany jako pakiet wymaga Capability API 1.24; `min` w `explode` lub `double`, `pool.abilityPlusAbility` i `botch.rule` wymagają 1.37, a `reroll` – 1.38. Importowany zestaw społecznościowy sprawdza czytający go Engine, więc dodatkowa deklaracja nie jest potrzebna.

#### Co Game Master może zapisać w teście puli

```
[skill_check: skill="Ward" dc="2" who="Bram the Quiet" threshold="8" bonus="-2" with="Sinew"]
[skill_check: skill="Ward" difficulty="Grim" explode="9"]
[skill_check: skill="Nerve" dc="1" with="Warmth"]
[skill_check: skill="Ward" dc="2" reroll="careful"]
```

- `dc` to liczba wymaganych **sukcesów**, nie próg pojedynczej kości. Może wynosić od 1 do maksimum możliwego do zliczenia: maksimum puli, podwojonego przy eksplozjach i ponownie podwojonego przy podwójnych sukcesach.
- `threshold=` zmienia próg pojedynczej kości i jest dostępne tylko, gdy `target.min` jest mniejsze od `target.max`.
- `bonus=` dodaje lub odejmuje kości i jest dostępne tylko przy zadeklarowanym `situationalDice`.
- `with=` wykonuje test umiejętności lub rzut obronny z innym atrybutem. Działa w obu rodzajach, więc zestaw 5e uzyskuje "Strength (Intimidation)" tym samym atrybutem. W teście atrybutu dodaje zamiast tego drugi atrybut, jeśli włączono `pool.abilityPlusAbility`.
- `difficulty=` wskazuje stopień drabiny zamiast `dc=`, w obu rodzajach: jego sukcesy (lub trudność przy sumie) i, przy puli, jego próg. `dc=` zapisane obok nadal decyduje o liczbie, a `threshold=` o progu. Nazwa, której nie ma żaden stopień albo którą dzielą dwa stopnie, jest ignorowana.
- `explode=` i `double=` przesuwają wyniki tych reguł w danym teście i są dostępne tylko wtedy, gdy reguła ma `min`.
- `reroll=` wskazuje jeden z twoich wpisów `reroll` dla danego testu i jest dostępne tylko wtedy, gdy jakieś zadeklarowano. Nazwa, której nie odpowiada żaden z nich, jest ignorowana.

Każde pole respektuje deklarację pliku: wartość poza zakresem jest przycinana do bliższego końca, a nieoferowany atrybut jest ignorowany zamiast odrzucać test. Zapis pokazuje rzeczywiście użyte wartości: trudność i próg jako liczby, dodatkowe kości po ograniczeniu, `with=` tylko wtedy, gdy atrybut faktycznie podmieniono lub dodano, wynik reguły tylko wtedy, gdy test go przesunął, `reroll=` tylko wtedy, gdy wskazany ponowny rzut był tym wykonanym i coś przerzucił, oraz `complication="true"`, gdy reguła krytycznej porażki `halfOrMore` dała komplikację obok wyniku. Engine zawsze sam rzuca kośćmi. Zastępuje wynik puli napisany przez model, ignoruje `mode="advantage"`, bo ten rodzaj nie ma ułatwienia, i nie używa kości rzuconej przez gracza przed turą.

#### Co jest poza zakresem i dlaczego

Każda z tych mechanik wymaga własnego rodzaju rozstrzygania, bo żadnej nie da się wyrazić liczeniem kości osiągających próg:

- **Najwyższa kość** (jak w Blades in the Dark) wymaga stopnia częściowego sukcesu, którego wynik testu nie ma.
- **Pule postawy porównywane z cechą** (jak w Lasers and Feelings) ustalają "powyżej czy poniżej" osobno dla testu, co jest innym porównaniem.
- **Kości z symbolami** (jak w Genesys) w ogóle nie dają liczb.
- **Pule przeciwstawne** rozstrzygają dwie postacie naraz; test ma jedną rzucającą postać.
- **Rzuty poniżej progu i otwarte testy procentowe** porównują w drugą stronę.
- **Sumowane pule z dziką kością** (jak w OpenD6) sumują kości, traktując jedną szczególnie.

Obie rzeczy wcześniej pomijane przez ten rodzaj są już modelowane; poniższa sekcja Wydawanie zasobów na zmianę rzutu wyjaśnia jak. Własna reguła systemu, "wydaj punkt za sukces" albo "wydaj punkt, aby ponownie rzucić nieudanymi kośćmi", to `resolution.spend`. Ponowny rzut przyznawany przez system za darmo, gdy wymaga tego sytuacja, to opisane wyżej `resolution.reroll`. Ponowny rzut należący do czegoś wybranego przez postać zapisuje się jako `mechanics.check` wpisu katalogu i opłaca kosztem tego wpisu.

### Wydawanie zasobów na zmianę rzutu

Niektóre systemy pozwalają graczowi zapłacić za nadchodzący rzut, np. punkt woli za automatyczny sukces. `resolution.spend` opisuje to jako stałą regułę systemu, nie rzecz kupioną przez postać:

```json
"spend": [{ "pool": "resolve", "amount": 1, "successes": 1, "perCheck": 2 }]
```

- `pool` to jedna z `live.pools`. Nie może początkowo być pusta, bo na początku gry nie byłoby czego wydawać.
- `amount` to koszt JEDNEGO zakupu. `successes`, `dice` i `reroll` określają, co daje; zakup musi dawać przynajmniej jedno z nich. Sukcesy dodaje się po zliczeniu kości i anulowaniu, bo nie pochodzą z rzutów. Kości rzuca się razem z pulą, w jej własnym zakresie. `reroll` ma postać `{ "upTo": 6, "mode": "once" }`, odczytywaną tak jak `resolution.reroll`, i kupuje się go raz, niezależnie od liczby zakupów w teście: kość rzucona ponownie dwa razy to nadal jeden ponowny rzut.
- `perCheck` ogranicza liczbę zakupów jednego testu, więc maksymalna wartość zakupu to `amount * perCheck` punktów. Limit zapobiega kupowaniu pewnego zwycięstwa za pełną pulę. To liczba od 1 do 10, wartość z arkusza w tej samej postaci, w jakiej odczytuje ją wartość pochodna (`{ "abilityScore": "nerve" }`, `{ "derived": "focus" }`), albo `"pool"`: tyle, ile kości ma test, czyli wartość arkusza dla niego przed ranami, dodatkowymi kośćmi i modyfikatorami. Wartość z arkusza jest odczytywana dla rzucającego, zaokrąglana w dół i utrzymywana w zakresie od 0 do 100, a postać, której limit wynosi 0, niczego nie kupuje i za nic nie płaci.
- Tylko `dice-pool` może mieć tę regułę: sumowany rzut nie ma sukcesów ani puli, do której można dodać kości. `dice-sum` deklarujące `spend` jest odrzucane podczas importu.
- Najwyżej cztery wpisy, a dwa nie mogą nazywać tej samej puli, bo test nie mógłby wskazać, o który chodzi.
- Wydatek kupujący `reroll`, `perCheck` niebędące liczbą i więcej niż dwa wpisy wymagają Capability API 1.38 w zestawie dostarczanym jako pakiet.

**Wpisuje się to w sam test.** Game Master pisze `[skill_check: skill="Nerve" dc="2" spend="resolve:1"]`, nie osobną komendę `[sheet:]`, ponieważ kości rzuca się przed wykonaniem komend arkusza i później nie byłoby już czego zmieniać. Jedno rozstrzygnięcie rzuca kośćmi i płaci za to, co je zmieniło.

### Talizman zmieniający rzut

`resolution.spend` jest regułą systemu. Wpis rzeczywiście WYBRANY przez postać także może zmienić test, przez `mechanics.check` we wpisie katalogu:

```json
"mechanics": {
  "kind": "utility",
  "cost": [{ "pool": "blood", "amount": 1 }],
  "perCostStep": { "flat": 1 },
  "check": { "reroll": { "upTo": 1, "mode": "once" }, "successes": 1 }
}
```

- `reroll` ponownie rzuca kości z wynikiem nie większym niż `upTo`. `once` podmienia każdą raz i nowy wynik zostaje; `until` kontynuuje. `upTo` musi być niższe od najwyższej ścianki, inaczej cała pula byłaby rzucana bez końca; Engine ogranicza liczbę ponownych rzutów testu niezależnie od pliku.
- `dice` dodaje kości przed rzutem, `successes` dodaje sukcesy po ich zliczeniu, a `threshold` ustala próg pojedynczej kości tego rzutu w zakresie dozwolonym przez `target`.
- `explode` i `double` przesuwają wyniki tych reguł dla tego jednego rzutu, tylko gdy reguła ma `min`, i w jego granicach: `"check": { "explode": 8 }` to wpis, dzięki któremu ósemki dają dodatkową kość. Mają pierwszeństwo przed tym, co Game Master zapisał w znaczniku, tak jak `threshold`. Capability API 1.37.
- Wymagane jest co najmniej jedno z tych pól; inaczej wpis niczego nie określa i jest odrzucany.
- Obsługuje to wyłącznie `dice-pool`, więc `dice-sum` z `mechanics.check` jest odrzucane przy imporcie.

**Koszt określa własne `cost` wpisu**, płacone dokładnie tym samym mechanizmem co użycie innych rzeczy: pula i jedno użycie każdego licznika zapisanego przez ten wpis. `perCostStep` określa SKALOWANIE; wpis z tym polem kupuje się tyle razy, ile zapłacono cenę, a bez niego jeden raz, niezależnie od zaoferowanej kwoty.

**Game Master nazywa go w teście:** `[skill_check: skill="Brawl" dc="3" use="Potence" spend="blood:3"]`. Nie w osobnej komendzie arkusza, z tego samego powodu co wyżej: kości rzuca się przed wszelkimi zapisami stanu.

**Wszystko albo nic.** Jeśli pula nie wystarcza, zakup nie następuje i nic nie jest odejmowane: rzut pozostaje dokładnie taki jak bez zakupu. Punkty niebędące całkowitą wielokrotnością zakupu także nic nie kupują. Żądanie przekraczające `perCheck` jest przycinane, nie odrzucane, i płaci się tylko za limit. Wszystko oblicza Engine; Game Master podaje zadeklarowany przez gracza wydatek i nigdy nie dotyka kości. Zapis pokazuje rzeczywistą opłatę, zastosowany wpis, sukcesy uzyskane bez rzutu i liczbę ponownie rzuconych kości. Talizman niewybrany przez postać lub taki, którego katalogu Engine nie może odczytać, nie robi nic zamiast działać na zaufanie.

### Arkusz

- `sections` grupuje rzeczy w edytorze. Pola, wartości pochodne i listy wskazują przez `section` sekcję, w której się znajdują; mogą to robić także atrybuty, umiejętności i rzuty obronne. Edytor, arkusz w grze i blok arkusza dla Game Master pokazują je wtedy pod tym nagłówkiem (arkusz, który nie wskazuje dla nich sekcji, pokazuje je dokładnie tak jak wcześniej). Sekcja może też mieć regułę `untrained` (niżej).
- `abilities` to podstawowe cechy. `skills` i `saves` mogą wskazywać atrybut używany przy rzucie, `cap` i regułę `untrained` (niżej).
- `fields` to pojedyncze wartości. Typy: `number`, `text`, `longtext`, `boolean`, `enum` (stała lista wyborów) i `dice` (tekst taki jak `1d8`).
- Wartości `derived` oblicza się z innych wartości i nie można ich nadpisać ręcznie. Operacje to `sum`, `min`, `max`, `scale` (mnożenie i zaokrąglanie), `stepTable` (wyszukanie w progach, jak poziom wyznaczający premię biegłości) oraz `enumTable` (liczba dla każdej wartości pola wyliczeniowego lub stanu bieżącego; zobacz niżej Stany bieżące).
- `lists` to tabele z własnymi kolumnami, np. ekwipunek, zaklęcia lub zdolności. Lista z `pools` zmienia każdy wiersz w zasób z własnym maksimum, przydatny dla zdolności klasy o ograniczonej liczbie użyć.
- `live` to rzeczy zmieniające się podczas gry: `pools` (punkty życia, komórki zaklęć, Grit), `tracks` (liczba na skali, np. wyczerpanie, lub tor ran z zaznaczanymi polami), `text` (krótkie notatki, np. przedmiot koncentracji), `conditions` i `states` (jedna wartość z zamkniętego zbioru, np. forma lub postawa).

Wszystko, co odczytuje liczbę, nazywa ją odwołaniem do wartości: obiektem o dokładnie jednym kluczu `const`, `field`, `derived`, `abilityScore`, `abilityMod`, `abilityModFromField`, `skillMod`, `saveMod`, `listSum`, `livePool`, `liveTrack` lub `itemStat`. Przykład puli o maksimum będącym wartością pochodną: `"max": { "derived": "grit_max" }`.

- `listSum` sumuje jedną kolumnę liczbową listy: `{ "listSum": { "list": "gear", "column": "bulk", "onlyWhen": "packed" } }`. `onlyWhen` jest opcjonalne i wskazuje kolumnę logiczną; liczą się tylko wiersze, w których jest włączona. Pusta komórka liczy się jako wartość domyślna kolumny, a lista ukryta przez `hideWhen` nic nie dodaje. Tak Ember Roads oblicza Burden z ekwipunku spakowanego przez postać, więc wartość nigdy nie rozjedzie się z listą, jak mogłoby się stać z ręcznie wpisaną liczbą. Skalowana kolumna katalogu nie może odczytywać sumy listy, nawet przez wartość pochodną: lista może zawierać skalowane komórki, własne lub odczytujące ją z powrotem, i przeliczanie nigdy by się nie ustabilizowało.
- `livePool` odczytuje, ile zostało w jednej z twoich `live.pools` (pula wiersza listy do nich nie należy). `liveTrack` odczytuje jeden z twoich `live.tracks`, z opcjonalnym `read`: `"value"`, czyli bieżąca pozycja (domyślnie); `"filled"`, czyli odległość powyżej `min`; `"remaining"`, czyli odległość poniżej `max`; albo, tylko na torze ran, `"penalty"`, czyli obowiązująca kara. Ukryta pula lub tor dają 0. "Harm still to take" z Gravewatch to `{ "liveTrack": "harm", "read": "remaining" }`.
- Odczyt bieżący bierze bieżący stan z chwili rzutu testu, rozpoczęcia walki lub zapisu bloku arkusza dla Game Master. Gdy bieżącego stanu jeszcze nie ma (edytor arkusza, przegląd importu), odczytuje stan, od którego zaczyna się gra: pulę pełną lub pustą zgodnie z jej `start`, tor na wartości `default` i czysty tor ran.
- Nic, co oblicza się, zanim powstanie bieżący stan, nie może go odczytywać: `max` puli lub toru, premia biegłości ani skalowana kolumna lub skalowanie katalogu. Dotyczy to także odczytu przez wartość pochodną i przez umiejętność ograniczoną wartością bieżącą, a import podaje wartość, która to robi.
- Te trzy wymagają Capability API 1.39 w zestawie dostarczanym jako pakiet.
- `itemStat` odczytuje przedmioty posiadane przez postać, np. ochronę noszonego pancerza, i podlega tej samej regule co odczyt bieżący. Zobacz [Przedmioty w arkuszu](#items-on-the-sheet). Capability API 1.52.

**Co robi test bez wyszkolenia.** Umiejętność lub rzut obronny, w którym postać nie ma wyszkolenia (jej poziom wyszkolenia jest pierwszy), rzuca się normalnie, chyba że zestaw zasad stanowi inaczej przez `untrained` na umiejętności lub rzucie obronnym albo na sekcji, w której się znajdują (pierwszeństwo ma własna reguła umiejętności lub rzutu obronnego):

```json
"sections": [{ "id": "labour", "label": "Labour", "untrained": { "by": -1 } }],
"skills": [
  { "id": "dig", "label": "Dig", "ability": "sinew", "section": "labour", "untrained": "refuse" },
  { "id": "listen", "label": "Listen", "ability": "nerve", "untrained": "harder" }
]
```

- `"normal"`: normalnie (domyślnie).
- `{ "by": -3 }`: dodawane do liczby testu, od -20 do 20: kości w puli, stała wartość przy sumie. Jest częścią liczby pokazywanej w arkuszu i używanej w każdym teście, przed jakimkolwiek `cap`.
- `"harder"`: o jeden stopień trudniej. W puli próg pojedynczej kości rośnie o jeden (i pozostaje w zakresie progu), więc wymaga zestawu z pulą, którego próg może się zmieniać; gdzie indziej jest odrzucane przy imporcie, bo nigdy nie zmieniłoby rzutu.
- `"refuse"`: test w ogóle nie jest rzucany. Engine odsyła żądanie Game Master z `reason="untrained"`, co je rozstrzyga: nic później go nie rzuca, Game Master dowiaduje się, że postać nie mogła tego spróbować, a narracja to mówi. Endpoint testów umiejętności odpowiada na taki test kodem 400 (`skill_check_untrained`), a oparte na nim rozgałęzienie na sukces i porażkę nie zachowuje żadnej z połówek. Edytor arkusza nie pokazuje dla niego liczby.
- Postać, dla której nikt nie ma arkusza, nie ma wyszkolenia do odczytania, więc nie dotyczy jej żadna reguła testów bez wyszkolenia.
- Przypomnienie dla Game Master wymienia reguły testów bez wyszkolenia według sekcji lub umiejętności, więc prosi o testy, które postać może wykonać.

Sekcje na atrybutach, umiejętnościach i rzutach obronnych oraz `untrained` w dowolnym miejscu wymagają Capability API 1.41 w zestawie dostarczanym jako pakiet.

Umiejętność lub rzut obronny może mieć `cap`, odwołanie do wartości będące najwyższą wartością, jaką może osiągnąć test: wartość, której nie da się przerosnąć, albo tor, który ją ogranicza. Gravewatch ogranicza Soothe do Resolve, które zostało strażnikowi: `"cap": { "livePool": "resolve" }`. Ograniczona liczba jest tą pokazywaną w arkuszu i używaną w każdym teście, a podmianę przez `with=` oblicza się na liczbie sprzed ograniczenia i potem ogranicza ponownie. Ograniczenie nie może odczytywać modyfikatora umiejętności ani rzutu obronnego; nie może tego również odczytywana przez nie wartość pochodna ani żadna wartość pochodna zadeklarowana nad nią, więc arkusz nadal oblicza się raz, od góry do dołu. Capability API 1.39.

`hideWhen` ukrywa pole, wartość pochodną, listę, pulę lub zwykły tor na podstawie wartości innego pola, na dokładnie jeden z trzech sposobów: `equals` jedna wartość, `notEquals` jedna wartość (ukryte, gdy pole zawiera cokolwiek innego; tak wyraża się "tylko dla tego rodzaju postaci") albo `in` lista wartości. Każda wskazana wartość musi być możliwa w tym polu, inaczej reguła nigdy by nie pasowała, i jest sprawdzana pojedynczo; dla `notEquals` i `in` obejmuje to zakres pola liczbowego i długość pola tekstowego. Reguła odczytuje wartość pokazywaną przez edytor arkusza: nieustawione pole ma wartość, od której zaczyna pusty arkusz, a wartość wyliczeniowa, której zestaw już nie oferuje, jest wartością domyślną pola. Plik 5e używa `equals`, aby ukryć komórki zaklęć u postaci nierzucających zaklęć; Gravewatch usuwa olej do latarni z arkusza każdego strażnika, który nie pełni nocnej warty, przez `"hideWhen": { "field": "watch", "notEquals": "night" }`. Warstwa nie może usunąć wartości, z którą porównuje któraś z tych reguł. Toru ran nie można ukryć, bo rzuty i walki odczytują go niezależnie od tego, co pokazuje arkusz. `notEquals` i `in` wymagają Capability API 1.39.

`max` zwykłego toru może być odwołaniem do wartości zamiast liczby, więc jego długość wyznacza wartość posiadana przez postać: `"max": { "field": "willpower_rating" }`. Jego `min` i `default` pozostają liczbami. Tor ran z nazwanymi poziomami ma `max` równe liczbie poziomów; tor z numerowanymi polami jest tak długi jak jego własne `max` (zobacz niżej Numerowane pola). `alwaysShow: true` wypisuje tor w bloku arkusza dla Game Master nawet przy wartości domyślnej, dla wartości ważnej w każdej turze; bez tego tor na wartości domyślnej jest pomijany, bo trzy rzuty przeciw śmierci na zerze niczego nie mówią. Te trzy wymagają Capability API 1.37 w zestawie dostarczanym jako pakiet.

### Stany bieżące: forma, postawa, światło

Pole wyliczeniowe wybiera się przy tworzeniu postaci i ten wybór zostaje, a stan jest tylko włączony albo wyłączony. **Stan bieżący** to jedna wartość z zamkniętego zbioru, która zmienia się podczas gry: forma zmiennokształtnego, postawa w walce, jasność latarni.

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

- `values` to od dwóch do czterdziestu wartości, każda pokazywana jako jej wpis w `valueLabels` albo w niezmienionej postaci. Podlegają regule każdej etykiety (jedna linia, bez nawiasów kwadratowych i klamer makr) i nigdy nie zawierają cudzysłowu, bo komenda arkusza ujmuje je w cudzysłów. Najwyżej dwanaście stanów bieżących.
- `default` to wartość, od której zaczyna każdy arkusz; po pominięciu – pierwsza wartość. Zapisywany jest tylko stan bieżący zmieniony z wartości domyślnej, a zapisana wartość, której stan już nie oferuje, jest odczytywana jako domyślna.
- `hideWhen` usuwa stan bieżący z arkusza na podstawie wartości pola, tak jak pulę: nie jest pokazywany, nie ustawia go komenda, a tabela (niżej) odczytuje go jako brak wartości.
- Game Master ustawia go przez `[sheet: who="Name" op="state" state="Stance" value="Reckless"]`, wskazując stan przez identyfikator lub etykietę, a wartość przez nią samą lub jej etykietę. Stan lub wartość, których arkusz nie ma, są odrzucane (`unknown-state`, `unknown-value`). Engine uczy tej komendy i wymienia wszystkie wartości tylko wtedy, gdy zestaw ma stan bieżący, a każdy blok arkusza podaje wartość każdego stanu bieżącego, także domyślną, bo "w ludzkiej postaci" to fakt potrzebny narracji w każdej turze.
- Ekran arkusza w grze pokazuje selektor dla każdego stanu bieżącego.
- Odpoczynek przywraca go krokiem odnowienia, który go wskazuje: `{ "state": "stance", "to": "default" }` albo `"to"` jedna z jego wartości.

**Liczby, które za tym idą.** Wartość pochodna z `"op": "enumTable"` daje liczbę dla każdej wartości pola wyliczeniowego lub stanu bieżącego:

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

- `from` wskazuje dokładnie jedno z: `field` (pole wyliczeniowe) lub `liveState`. Każdy wiersz `table` to wartość możliwa w tym źródle, od jednej do czterdziestu. Wartość pominięta w tabeli i stan ukryty przez arkusz dają `default` (0 po pominięciu).
- Pole jest ustalane przy tworzeniu postaci, więc tabela oparta na nim może zasilać maksimum: Gravewatch daje strażnikowi na porannej warcie jeden dodatkowy punkt Resolve. Stan bieżący zmienia się podczas gry, więc tabela oparta na nim jest odczytem bieżącym, jak `liveTrack`: nic, co oblicza się, zanim powstanie bieżący stan, nie może jej odczytywać, bezpośrednio ani przez inną wartość pochodną. Dlatego klucz to `liveState`, a nie `state`: każdy odczyt bieżącego stanu ma w nazwie `live`, dzięki czemu zarówno ty, jak i import rozpoznajecie, czego maksimum nie może używać.
- Aby dodać tę liczbę do kości, wskaż ją w `resolution.adjust` (zobacz Modyfikatory z arkusza). Ember Roads dodaje swoją postawę (Stance) do każdego rzutu Brawn; Gravewatch odejmuje karę za zasłoniętą lub zgaszoną latarnię od każdego rzutu Nerve.
- Warstwa może usunąć z pola wartość, dla której tabela ma wiersz. Ten wiersz po prostu nie jest odczytywany, gdy warstwa jest włączona.

Stany bieżące, `enumTable` i krok odpoczynku wskazujący `state` wymagają Capability API 1.42 w zestawie dostarczanym jako pakiet.

### Tory ran: zdrowie jako tor zamiast liczby

Wiele systemów wcale nie liczy punktów życia. Mają kolumnę pól, każde gorsze od poprzedniego, i zaznacza się jedno po zranieniu. Dodaj do wpisu `live.tracks` `kinds` oraz nazwane `levels` albo numerowane `boxes` (niżej), aby zmienić liczbę na skali w taki tor:

**Jakiej formy potrzebuje twój system?** Pula zapisuje, ILE obrażeń zadano; tor zapisuje, ile ORAZ jakiego rodzaju była każda ich część. Jeśli twój system rozróżnia rany obuchowe, śmiertelne i spotęgowane, a rodzaj pozostaje ważny po ciosie, ponieważ spotęgowane goją się wolniej, nie mogą być pochłaniane albo ostatecznie zabijają, ten rodzaj musi gdzieś przetrwać po rzucie, a przechowuje go tylko znacznik. Pula punktów tego nie potrafi: po odjęciu obrażeń jest tylko mniejszą liczbą, a karta nie pamięta rodzaju poszczególnych punktów. Dlatego `combat.damageKinds` jest odrzucane w zestawie zasad ze zdrowiem jako pulą, zamiast być po cichu ignorowane. Pula nadal może mieć `damageTypes`, a przeciwnik może być na nie odporny lub niewrażliwy, ponieważ to kwestia wielkości otrzymanego ciosu, a nie rodzaju późniejszej rany.

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

- `levels` to 1–16 poziomów, od najlepszego do najgorszego. Każdy ma `label` i całkowity `penalty` nie większy niż 0. Duża ujemna liczba oznacza w takich systemach wyłączenie z działania, więc `-99` jest poprawne.
- `kinds` to 1–6 rodzajów obrażeń przyjmowanych przez tor, każdy z `id`, krótkim `label` pola i `severity`. Stopnie ciężkości muszą być różne; liczby znaczą tylko kolejność, więc odstępy są dowolne.
- Oba występują razem. `kinds` bez `levels` lub `boxes` jest odrzucane, bo nie ma czego zaznaczać, a `levels` lub `boxes` bez `kinds` – bo znacznik musi mieć rodzaj. Tor ma `levels` albo `boxes`, nigdy oba.
- **Rozróżniaj te słowa.** `kinds` to rodzaje dozwolone przez zestaw. ZNACZNIK to jeden taki rodzaj umieszczony na torze podczas gry. Definicja przechowuje rodzaje; arkusz postaci znaczniki.
- Tor ran zaczyna bez znaczników, więc jego `min` to 0. Tor z nazwanymi poziomami jest tak długi jak lista poziomów, więc jego `max` to `levels.length`; inny zapis jest odrzucany, nie poprawiany po cichu, więc plik nigdy nie przechowuje dwóch sprzecznych długości. Tor z polami jest tak długi jak jego własne `max`, które może być wartością obliczaną przez arkusz (zobacz niżej Numerowane pola).

**Dokładne reguły**, bo nieprecyzyjne odczytanie daje niepoprawny tor:

- Znaczniki są posortowane, **najcięższe najpierw**. Tor siedmiu poziomów mieści najwyżej siedem znaczników.
- Znacznik **wstawia się według ciężkości** pomiędzy obecne, nigdy na końcu. Zajmuje najwyższy poziom należny ciężkości i przesuwa lżejsze w dół.
- Działa kara z **najniższego zaznaczonego poziomu**, nie suma zaznaczonych. Trzy znaczniki powyższego toru dają `-3`, nie `0 + -1 + -3`.
- `amount` to liczba znaczników jednego rodzaju **stosowanych pojedynczo**; tor zapełniający się w trakcie podlega tej samej regule co już pełny.
- Zaznaczanie **pełnego** toru **podnosi najlżejszy znacznik o jeden stopień** zamiast dodawać nowy. To jeden stopień własnej skali rodzajów, niezależnie od rodzaju dodawanego znacznika.
- Znacznik przekraczający najwyższą ciężkość pozostaje na maksimum, a niemieszczący się jest liczony jako **nadmiar**. Nadmiar jest zapisywany, więc przeładowanie nie zapomina otrzymanych obrażeń.
- **Leczenie to ta sama komenda z ujemną liczbą.** Usuwa nadmiar przed jakimkolwiek znacznikiem. Leczenie wskazujące jeden z rodzajów toru usuwa tylko znaczniki tego rodzaju, więc "obuchowe się goją, śmiertelne zostają" to `kind="bashing"` z ujemną liczbą; leczenie niewskazujące żadnego rodzaju toru usuwa najpierw najlżejsze znaczniki.

**Zaznaczanie podczas gry.** Game Master pisze `[sheet: op="damage" track="harm" kind="knock" amount="1"]` i leczy przez ujemne `amount`. Na torze zaznaczanym według pola dodaje `box="3"`, czyli pole, w które trafia cios. Wariant `damage` wskazujący `pool=` nie zmienia się. Zwykła komenda `track` jest odrzucana dla toru ran: sama liczba nie określa nowych znaczników. Gracz może też ręcznie zaznaczać i czyścić pola arkusza, zgodnie z założeniami tych systemów.

**Walka także może zaznaczać tor.** Skieruj `combat.health` na tor zamiast puli. Trafiony cios zaznacza tyle pól, ile mówi `combat.damageKinds.marks`, rodzajem przypisanym przez ten blok typowi obrażeń. Postać z pełnym torem pada, co odczytuje reguła umierania. Na torze zaznaczanym według pola obrażenia `per-point` wskazują pole, w które trafiają, a `per-blow` celują w pierwsze, a cios, którego nie może przyjąć żadne pole, powala postać, bo tak w systemach z takim torem wygląda wyeliminowanie z walki. Leczenie usuwa jeden znacznik, najlżejszy, dowolnego rodzaju. Punkty tymczasowe są odrzucane, bo tor nie ma dla nich bufora. Engine odczytuje tor jako liczbę POZOSTAŁYCH poziomów, więc reszta walki, upadek, przywrócenie, dziennik i podsumowanie nie zmieniają się.

**Odpoczynek może leczyć tor ran.** Krok odnowienia z `"to"` czyści go do podanej liczby znaczników, wraz z nadmiarem; krok z UJEMNYM `"by"` usuwa tyle znaczników, zaczynając od nadmiaru, więc `"by": { "const": -2 }` usuwa dwa. Krok, który wskazuje także `kind`, usuwa tylko znaczniki tego rodzaju: "Catch your breath" z Gravewatch to `{ "track": "harm", "kind": "knock", "to": "min" }`, co leczy stłuczenia i zostawia rozdarcia. Krok, który DODAŁBY znaczniki, nie robi nic, bo odpoczynek nie zadaje obrażeń; dotyczy to także dodatniego `"by"`: jest pomijane bez słowa, więc pamiętaj o znaku minus.

#### Numerowane pola zamiast nazwanych poziomów

Gdy tor jest tak długi, jak wytrzymała jest postać, nadaj mu `boxes` zamiast `levels`. Ma tyle pól, ile wynosi jego własne `max`, które może być wartością obliczaną przez arkusz (nigdy odczytującą bieżący stan), najwyżej 64:

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

- Obowiązującą karę odczytuje się z `table` (tabeli progów, jak w wartości pochodnej, z karami nie większymi niż 0) dla liczby pól `filled` lub `remaining`, jakakolwiek jest, więc tabela może coś kosztować, nawet gdy nic nie jest zaznaczone. Kara nie zależy od tego, które pola są zaznaczone.
- Pola są numerowane: arkusz pokazuje "Box 3", a blok arkusza dla Game Master podaje, ile pól zaznaczono z ilu, zamiast nazywać poziom.

#### Zaznaczanie według pola i tory odrzucające znaczniki

- `"fill": "indexed"` umieszcza znacznik w polu wskazanym przez komendę przez `box=` (w pierwszym polu, gdy żadnego nie wskazuje) albo w najbliższym wolnym polu powyżej i nigdy nie przesuwa znacznika, który już stoi. Leczenie usuwa najwyżej położony spośród najlżejszych znaczników. Ekran arkusza pozwala graczowi kliknąć dowolne czyste pole.
- `"onFull": "refuse"` odrzuca znacznik, dla którego nie ma wolnego pola, zamiast podnosić najlżejszy znacznik. Odrzucana jest cała komenda, więc trzy znaczniki przy dwóch wolnych polach nie trafiają nigdzie. Tor zaznaczany według pola zawsze odrzuca; zaznacz to, dodając obok `"onFull": "refuse"`.

#### Poziomy dodawane przez listę

`"extra": { "list": "scars", "countColumn": "levels", "penaltyColumn": "penalty" }` na torze z nazwanymi poziomami pozwala liście w arkuszu wydłużać go dla każdej postaci osobno. Każdy wiersz, po kolei, dodaje swoją liczbę poziomów (najwyżej 16) ze swoją karą, za ostatnim poziomem o karze co najmniej tak dobrej; poziomy te noszą nazwę tamtego poziomu, gdy kary są równe, a w przeciwnym razie nazwę od kary. Tak działają blizny w Gravewatch. Lista ukryta przez arkusz nic nie dodaje, a gdy usunięto wiersz, którego poziomy są zaznaczone, znaczniki, które już się nie mieszczą, zostają zachowane jako nadmiar: edycja arkusza nie cofa obrażeń.

`boxes`, `fill`, `onFull`, `extra` i `kind` w kroku odpoczynku wymagają Capability API 1.40 w zestawie dostarczanym jako pakiet.

### Kara do rzutów

`resolution.penaltyFrom` wskazuje tor ran, którego kara dotyczy każdego testu zestawu. Jest deklarowana, nie domyślna: zestaw bez niej rzuca dokładnie tak jak przed wprowadzeniem torów ran.

DZIAŁANIE kary zależy od rodzaju rozstrzygania, tak samo jak znaczenie liczby z arkusza:

- W `dice-pool` **odejmuje kości z puli**, nie poniżej własnego `pool.min`. Przy `pool.min` równym 1 nawet postać na ostatnim poziomie rzuca jedną kością; przy `pool.min` równym 0 nie rzuca żadną i przegrywa bez rzutu.
- W `dice-sum` jest **stałym modyfikatorem rzutu**, włączanym do tej samej liczby co atrybut i wyszkolenie.

Wskazany tor musi być torem ran. Zwykły tor nie ma kary i wskazanie go jest odrzucane podczas importu. Wynik podaje zastosowaną karę, aby gracz rozumiał mniejszą liczbę kości, a blok arkusza dla Game Master pokazuje poziom i jego koszt.

### Modyfikatory z arkusza

Niektóre liczby w arkuszu towarzyszą każdemu rzutowi, którego dotyczą: ciężki plecak przy każdej wspinaczce, kara za pancerz przy każdym skradaniu, błogosławieństwo przy wszystkim. `resolution.adjust` to określa, w obu rodzajach:

```json
"adjust": [{ "value": { "derived": "burdened" }, "abilities": ["brawn"] }]
```

- `value` to wartość z arkusza, w tej samej postaci, w jakiej odczytuje ją wartość pochodna, łącznie z bieżącym stanem. Liczba ujemna odejmuje. Ember Roads sumuje objętość ekwipunku spakowanego przez postać w Burden, a tabela progów zamienia każde trzy jednostki na jeden punkt odejmowany od każdego rzutu Brawn. Tabela oparta na stanie bieżącym to sposób, w jaki forma lub postawa trafia do kości (zobacz Stany bieżące).
- `abilities` jest opcjonalne. Z nim modyfikator dotyczy tylko testu rzucanego jednym z tych atrybutów: samego atrybutu, umiejętności lub rzutu obronnego, które go używają, umiejętności rzucanej nim przez `with=` albo pary atrybutów, która go zawiera. Bez niego dotyczy każdego testu.
- Stosuje się go tam, gdzie karę za rany: jako kości w puli, z tym samym ograniczeniem `pool.min`, i jako stałą liczbę przy sumie, wewnątrz modyfikatora sumowanego przez zapis. Najwyżej osiem wpisów, sumowanych dla każdego testu.
- Walka według zestawu zasad dodaje go do każdego rzutu budowanego z arkusza, w chwili rozpoczęcia walki: atak rzucany atrybutem lub umiejętnością, rzut obronny, test przeciwstawny i inicjatywa odczytywane z atrybutu, umiejętności lub rzutu obronnego otrzymują wpisy dla tego atrybutu, a wszystko inne otrzymuje tylko wpisy dotyczące każdego rzutu.
- Postać, dla której nikt nie ma arkusza, nic z tego nie otrzymuje, tak samo jak nie otrzymuje modyfikatora.

Wynik podaje, co dodał arkusz (`adjust="-2"` w zapisie i wiersz na karcie rzutu), aby gracz widział, dlaczego rzut wypadł tak, a nie inaczej. Wymaga Capability API 1.38 w zestawie dostarczanym jako pakiet.

### Odpoczynki

Odpoczynek jest listą kroków odnowienia i rzeczy do wyczyszczenia. Każdy wskazuje jeden cel (`pool`, `poolGroup`, `listPools`, `track` lub `state`) i ustawia go (`"to": "max"`, `"to": "min"` albo liczba) lub zmienia (`"by": { "const": 1 }` albo `"by": { "fractionOfMax": 0.5 }`). Krok wskazujący tor ran może go tylko leczyć, więc jego `by` jest ujemne; zobacz wyżej. Krok wskazujący stan bieżący tylko go ustawia, na `"default"` albo jedną z jego wartości.

### Tekst Game Master

- `checkGuidance` zastępuje wbudowany akapit mówiący Game Master, jak prosić o test. Podaj system i okoliczności rzutu. Game Master wskazuje tylko umiejętność i trudność. Engine rzuca i oblicza wynik z arkusza, więc nie proś modelu o matematykę.
- `sheetGuidance` wprowadza arkusze postaci do promptu. Wyjaśnij, które zasoby są ważne i kiedy je wydawać.
- `worldGuidance` jest opcjonalne i czytane raz, przy generowaniu świata, aby wymyślone realia pasowały do zasad: brak prochu, rzadka magia, chodzący zmarli. Nigdy nie trafia do tury.
- `sheetSummary` wybiera pola, wartości pochodne i wiersze list widoczne dla Game Master dla każdej postaci. Engine zawsze pokazuje modyfikatory atrybutów, wyszkolone umiejętności i rzuty obronne oraz bieżące wartości. Resztę skróć, bo wysyłana jest w każdej turze. `nameColumn` listy w podsumowaniu może być kolumną tekstową lub wyliczeniową (kolumna wyliczeniowa pokazuje etykietę wartości), a `columns` wskazuje do trzech kolejnych kolumn wiersza wypisywanych po nazwie, więc blok brzmi `Gear: Crowbar 1d6` zamiast samych nazw. Kolumna logiczna wypisuje swoją etykietę, gdy jest włączona. Oba wymagają Capability API 1.37 w zestawie dostarczanym jako pakiet.
- Nie musisz uczyć komendy arkusza w `sheetGuidance`. Engine sam uczy każdej komendy, z nazwami zadeklarowanymi w pliku, a gdy zestaw ma tor ran, dodaje komendę `op="damage"` do jego zaznaczania i wymienia jego poziomy oraz rodzaje obrażeń. Gdy zestaw ma stany bieżące, dodaje komendę `op="state"` i wymienia wartości, które każdy może przyjąć.

## Katalogi: gotowe wpisy list arkusza

Wpisywanie zaklęć, ekwipunku lub strony zdolności klasy wiersz po wierszu jest uciążliwe. Katalog to nazwany zbiór gotowych wpisów dostarczany z zestawem. Edytor proponuje je w selektorze każdej zasilanej listy; wybór wypełnia wiersz.

Katalogi są opcjonalne. Zestaw może mieć ich do dwunastu, a Engine nie zna ich tematyki: wszystkie ID, kolumny, filtry i słowa pochodzą z pliku.

### Nagłówek

Nagłówek trafia do `catalogs` na najwyższym poziomie pliku, obok `gm`.

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

- `id` i `label`: ID przestrzega reguł identyfikatorów arkusza; etykieta nazywa selektor.
- `holds`: `"rows"` (domyślnie i we wszystkich katalogach sprzed tego wydania), `"creatures"` lub `"items"`. Katalog stworzeń jest bestiariuszem czytanym przez walkę, a katalog przedmiotów wymienia rzeczy niesione przez drużynę. Żaden z nich nic nie zapisuje w arkuszu, nie deklaruje `feeds` i nie pojawia się w selektorze. Zobacz niżej [Stworzenia](#creatures-a-bestiary-a-fight-reads) i [Przedmioty](#items-what-a-party-carries).
- `feeds`: od jednej do ośmiu list arkusza, do których mogą pisać wpisy katalogu. Wymagane dla katalogu wierszy, odrzucane dla katalogu stworzeń lub przedmiotów. Wpis nigdy nie zapisze do niewymienionej listy ani wartości niedozwolonej przez jej kolumny.
- `filters`: opcjonalne, do ośmiu. Pola zawężania listy w selektorze: `number`, wartość `text` lub `tags` (kilka słów). `startFrom` wskazuje pole arkusza ustawiane przy otwarciu, więc postać z Calling równym Tinker widzi najpierw wpisy Tinker.
- `units`: opcjonalne. Znaczenie zasięgu lub wielkości obszaru w bloku `mechanics` wpisu w twoim systemie.

### Wpis

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

- `id`: małe litery, cyfry i pojedyncze łączniki; unikalne w katalogu.
- `label` i `summary`: to, co pokazuje selektor. Podsumowanie jest opcjonalne, jednoliniowe, do 300 znaków.
- `filters`: wartości filtrów nagłówka. Filtr `number` przyjmuje liczbę, `text` jeden tekst, a `tags` listę tekstów.
- `rows`: zapis po wybraniu wpisu, od jednego do sześciu wierszy. `list` należy do `feeds` katalogu, a kluczami `values` są ID kolumn tej listy.
- `creature`: przeciwnik zamiast wierszy, w katalogu, którego `holds` wskazuje stworzenia. Wpis ma dokładnie jedno z `rows` lub `creature`; stworzenie nie ma `mechanics`, bo opisuje działanie we własnych akcjach.

Każda wartość jest sprawdzana według kolumn docelowej listy, więc błędna nazwa kolumny lub liczba poza zakresem jest zgłaszana z wpisem źródłowym. Wpisy wewnątrz pliku zestawu sprawdza się przy wczytywaniu, czyli podczas importu pliku. Osobny plik katalogu pakietu sprawdza się przy pierwszym żądaniu selektora; błędny plik pokazuje tam przyczyny zamiast wpisów.

### Jeden wpis, kilka list

Zdolność z ograniczonymi użyciami to dwa wiersze arkusza: sama zdolność i jej licznik. Nadal wybiera się ją raz.

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

### Wartości aktualizowane przez zestaw

Po wyborze liczby wiersza należą do gracza. Przydatny wyjątek to maksimum podążające za postacią, np. użycia równe atrybutowi lub zasób klasy rosnący z poziomem. Wiersz może nazwać do czterech własnych kolumn liczbowych w mapie `scaled`, a edytor utrzymuje ich poprawność.

```json
{
  "list": "tricks",
  "values": { "name": "Last Ember", "uses": 1, "recharge": "camp" },
  "scaled": { "uses": { "from": { "abilityScore": "heart" } } }
}
```

- Klucz jest jedną z kolumn `number` listy.
- `from` to zwykłe odwołanie do wartości, z tego samego zamkniętego słownika co wszędzie. Bardziej złożone obliczenie jest wartością `derived` deklarowaną przez arkusz, do której odwołuje się `from` (`"from": { "derived": "lay_on_hands_max" }`). Nie dochodzi nowa arytmetyka.
- `table` jest opcjonalne. Pozwala wyszukać wartość odwołania w tabeli progów, np. uzyskać liczbę z poziomu: `"scaled": { "max": { "from": { "field": "level" }, "table": [[1, 2], [3, 3], [6, 4]] } }`.
- `values` nadal musi zawierać zwykłą liczbę kolumny; jej brak odrzuca wiersz. To wartość przed poznaniem arkusza i zachowywana przez arkusz bez wskazanego odwołania.
- Wiersz z `scaled` musi być jedynym wierszem wpisu dla tej listy, aby oznaczony wiersz arkusza zawsze pasował do jednej specyfikacji.

Wartość oblicza się przy edycji arkusza, nigdy przy odczycie, więc zapisany wiersz zawsze zawiera podaną liczbę. Wynik dopasowuje się do kolumny: przycina do `min` i `max` oraz zaokrągla w dół dla liczb całkowitych. W przykładzie Heart 3 daje trzy użycia, a Heart 0 lub mniej – zero. Wiersz pozostaje w arkuszu z 0 użyć; licznik o maksimum 0 nie jest pulą, więc w grze nie ma czego wydać.

Skalowane kolumny w pakiecie wymagają Capability API 1.23. Importowany zestaw społecznościowy sprawdza czytający go Engine, więc nie potrzebuje dodatkowej deklaracji.

### Wybrane wiersze są kopiami

Każdy wybrany wiersz jest kopiowany do arkusza z dodatkowym kluczem `_catalog`, zawierającym `<catalog id>/<entry id>`. ID kolumn zawsze zaczynają się literą, więc ten klucz nie może być twoją kolumną.

Kopia należy do postaci. Gracz może później edytować wszystko, arkusz działa bez zainstalowanego zestawu, a publikacja nowej wersji nigdy nie przepisuje cudzej postaci. Znacznik pozwala selektorowi pokazać posiadane wpisy i jest czytany przez opisane niżej odświeżanie.

### Odświeżanie z zestawu zasad

Ponieważ wybrany wiersz zachowuje znacznik, edytor karty może powiadomić gracza, że twój nowszy tekst różni się od zawartości wiersza. Krótka informacja pod listą podaje liczbę wierszy z nowszym tekstem, a przycisk **Review** (przejrzyj) pokazuje je, zestawiając zawartość karty z treścią zestawu zasad i udostępniając pole wyboru przy każdym wierszu. Zapis następuje dopiero po kliknięciu **Update selected** (aktualizuj wybrane); obejmuje wyłącznie różniące się kolumny w zaznaczonych wierszach. Reszta wiersza, w tym znacznik, pozostaje bez zmian.

Zakres porównania jest celowo wąski:

- Istniejące wartości są porównywane tylko w kolumnach `text`, `longtext`, `dice` i `enum`. Wartości `number` i `boolean` należą do gracza i są zachowywane, również 0 i false. Kolumnę, której wiersz jeszcze nie zawiera, można zaproponować z wartością odpowiedniego typu, także liczbą lub przełącznikiem. Kolumny skalowane są wyłączone, ponieważ już podążają za kartą.
- Tylko kolumny ustawiane przez twój wpis. Kolumna pominięta we wpisie nigdy nie jest zmieniana, niezależnie od zawartości karty.
- Wartość, której sama kolumna by nie przyjęła, np. nieoferowana już wartość `enum` albo tekst przekraczający `maxLength`, jest pomijana zamiast zapisywana.
- Wiersz jest dopasowywany do źródłowego wiersza wpisu według pozycji wśród wierszy tej listy z tym samym znacznikiem, o ile karta nadal ma ich tyle, ile zapisuje wpis. W przeciwnym razie dopasowanie działa tylko wtedy, gdy wpis zapisuje pojedynczy wiersz w tej liście. Jeżeli gracz usunął jeden z dwóch wierszy wpisu, wpis pozostaje bez zmian, bez zgadywania.
- Wiersz, którego wpisu nie ma już w katalogu, pozostaje bez zmian i bez powiadomienia.

Zmiana sformułowania lub nazwy wpisu może więc dotrzeć do postaci, które już go wybrały, jeśli zaakceptują aktualizację. Zmiana znaczenia liczby nie może i nie będzie tak działać: gdy wiersz należy do gracza, ta kolumna również należy do niego.

### `mechanics`: liczbowe działanie wpisu

Wpis może zawierać opcjonalny blok `mechanics`, który opisuje jego działanie liczbowo: `kind` (`attack`, `heal`, `buff`, `debuff`, `utility`, `rider`), `range`, `area`, `targets`, `targetCount`, `friendlyFire`, `amount` (kości, np. `2d6`, lub stała liczba), `damageType`, `attackRoll`, `autoHit`, `save` (jeden z rzutów obronnych karty i skutek sukcesu), `applies` (nakładane stany), `temporary` (punkty tymczasowe puli zdrowia), `scales` (wielkość rosnąca wraz z kartą), `cost` (pula zużywana przy użyciu), `perCostStep`, `budget` (zużywana część ekonomii akcji), `concentration`, `reaction`, `plus`, `free`, `gives`, `standard`, `rider` i `check`.

Selektor pokazuje ten blok w jednym wierszu. To, kto odczytuje resztę, zależy od bloku włączonego przez zestaw zasad:

- Z [blokiem `combat`](#combat-a-fight-your-own-rules-resolve) walka odczytuje efekty bojowe. `range`, `area` i `friendlyFire` działają na polu bitwy z pozycjami; `reaction` oznacza wpis odpowiadający na coś, a nie wykonywany w turze, i wskazuje chwilę, na którą czeka (zobacz [Okna](#windows-holding-the-fight-open)); wpis oznaczony zwykłym `true` nie wskazuje żadnej i nie ma go w żadnym menu. `check` dotyczy testów umiejętności opisanych wyżej.
- Tylko z [blokiem `battle`](#battles-lending-the-sheet-to-marinaras-combat) bitwa odczytuje `kind`, `range`, `area`, `friendlyFire`, `amount`, `damageType` i `cost`, ponieważ własny system walki Marinara ma zastosowanie właśnie dla tych części.

Słownik jest zamknięty: klucz lub wartość spoza powyższej listy jest odrzucana, zamiast być po cichu ignorowana.

`cost` określa też koszt płacony poza bitwą przez polecenie `use` Mistrza Gry, opisane w następnej sekcji.

### Polecenie `use`: Mistrz Gry płaci zdefiniowaną przez ciebie cenę

Podczas narracji Mistrz Gry aktualizuje każdą kartę poleceniami `[sheet: ...]`: `spend`, `restore` (`heal` oznacza to samo), `damage`, `temp`, `track`, `condition`, `note` i `rest`. Zestaw zasad dostarczający katalogi otrzymuje jeszcze jedno:

```
[sheet: who="Mira" op="use" name="Fireball"]
[sheet: who="Mira" op="use" name="Fireball" pool="3rd-level slots"]
```

`op="cast"` oznacza to samo co `op="use"`, a `spell=` to samo co `name=`, więc sformułowanie wybrane przez Mistrza Gry działa, choć format nie musi znać słowa "zaklęcie".

Nazwa jest dopasowywana bez uwzględniania wielkości liter do wierszy karty tej postaci pochodzących z jednego z twoich katalogów. Wiersz odpowiada na nazwę pokazaną Mistrzowi Gry (kolumna nazwy `sheetSummary` tej listy, następnie jej `pools.nameColumn`, następnie pierwsza kolumna tekstowa) oraz na `label` źródłowego wpisu, więc gracz nadal ma do niego dostęp po zmianie nazwy wiersza. Nazwa bez dopasowania i nazwa pasująca do dwóch różnych wpisów są odrzucane.

Co zużywa:

- Każdy składnik `mechanics.cost` wpisu. Składnik wskazujący pulę bieżącą pobiera z niej koszt; składnik wskazujący GRUPĘ pul pobiera go z pierwszej puli tej grupy, w kolejności deklaracji, która może go pokryć. Nie przechodzi automatycznie do wyższej puli, ponieważ grupa nie zawsze jest drabiną.
- Dodatkowo jeden punkt z każdej puli w wierszu listy zapisanej przez ten sam wpis, np. licznika użyć zdolności. To drugi wiersz powyższego wpisu `Last Ember`. Licznik o maksimum 0 nie ma użyć do wydania, więc polecenie jest odrzucane, zamiast wykonywać się za darmo.

`pool=` oznacza użycie na wyższym poziomie: ta sama pojedyncza cena, płacona z innej puli tej samej grupy. Jest akceptowane tylko wtedy, gdy koszt ma dokładnie jeden składnik, a wskazana pula należy do grupy tego składnika. Wszystko inne jest odrzucane zamiast reinterpretowane.

Wszystko albo nic. Jeśli którejkolwiek części nie można opłacić, całe polecenie zostaje odrzucone, nic się nie zmienia, a gracz otrzymuje informację. Wpis bez jakiegokolwiek kosztu, np. sztuczka lub pasywna zdolność, jest akceptowany i niczego nie zmienia.

### Wewnątrz definicji lub we własnym pliku

Mały katalog umieszcza się bezpośrednio w `ruleset.json`, w `entries` nagłówka. Długi katalog ma własny plik wskazany w nagłówku przez `asset`. Katalog ma dokładnie jedną z tych dwóch form.

```json
{ "id": "knacks", "label": "Knacks", "feeds": ["knacks"], "asset": "catalogs/knacks.json" }
```

Ścieżka ma zawsze postać `catalogs/<the catalog's id>.json`. Sam plik wygląda tak:

```json
{ "schemaVersion": 1, "catalog": "knacks", "entries": [] }
```

Osobne pliki katalogów służą pakietom publikowanym przez oficjalny katalog: pakiet wymienia plik w `contributions.assets.paths` obok `ruleset.json` i wymaga Capability API 1.21. Katalog stworzeń, wewnętrzny lub osobny, wymaga Capability API 1.27, a katalog przedmiotów – 1.49. **Zestaw zasad importowany jako pojedynczy plik lub udostępniany przez repozytorium GitHub zawiera katalogi wewnątrz definicji**, więc muszą mieścić się w limicie 256 KB całego pliku zestawu zasad. To wystarcza na kilkaset krótkich wpisów.

Limity to 12 katalogów na zestaw zasad, 2000 wpisów na katalog w obu formach i 1 MB na osobny plik katalogu.

<a id="items-what-a-party-carries"></a>

## Przedmioty: co niesie drużyna

Pancerz, broń, mikstury, wyposażenie, amunicja i pieniądze to przedmioty. Opcjonalny blok `items` deklaruje słowa, którymi opisuje się każdy przedmiot twojego zestawu zasad, a katalog z `holds: "items"` wymienia same przedmioty. Oba wymagają Capability API 1.49; `rarityCaps` i `propose` bloku, które dotyczą przedmiotów wymyślanych przez Mistrza Gry, wymagają 1.51, wartość odczytująca przedmioty posiadane przez postać (`itemStat`) – 1.52, działanie przedmiotu na testy, gdy jest założony lub niesiony, wraz z limitami `bonus` bloku – 1.53, wymagania przedmiotu wobec noszącego i zmieniane przez niego atrybuty – 1.54, `attack` broni – 1.55, działanie przedmiotu w walce – 1.56, to, czym broń strzela i ile ma załadowane – 1.57, tryby broni, atak drugą ręką, minimum obrażeń i stany przy trafieniu – 1.58, działanie użycia przedmiotu w walce wraz z jego ładunkami – 1.59, użycie przywracające pulę – 1.60, ładunki odzyskiwane przy odpoczynku lub przedmiot niszczący się po opróżnieniu – 1.61, użycie wymagające najpierw testu (`gate`) – 1.62, tabele łupów i łupy stworzenia – 1.63, pozycja łupów dająca monety lub warstwa usuwająca monety – 1.64, targ, miejscowość `sold` przedmiotu i `service` – 1.65, dzień targowy (sakiewka, `buyBack`, `stock` sprzedawcy, liczba `limited` albo `sold` bez wskazanej miejscowości) – 1.69, a efekt dodatkowy działający przy trzymanej broni (`weapons`) – 1.70.

### Blok przedmiotów

Blok trafia do `items` na najwyższym poziomie pliku. Oto blok Ember Roads, nieco skrócony:

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

- `categories`: od jednej do 24. Każdy przedmiot ma dokładnie jedną, np. pancerz, broń lub mikstura.
- `rarities`: opcjonalne, do 12. Wymień je od najpospolitszej do najrzadszej.
- `tags`: opcjonalne, do 48. Właściwości, które może mieć przedmiot, np. miotany lub posrebrzany.
- `stats`: opcjonalne, do 24. Liczby i słowa opisujące przedmiot, deklarowane dokładnie jak kolumna listy: `number`, `text`, `boolean`, `enum` lub `dice`, z zakresem, wartościami i wartością domyślną tam, gdzie typ je ma. `promptVisible` (domyślnie `true`) określa, czy Mistrz Gry widzi statystykę obok przedmiotu. Ustaw `false` dla czegoś, co powinien widzieć tylko gracz, np. miejsca, w którym ukryto przedmiot.
- `slots`: opcjonalne, do 12. Miejsca, w których przedmiot się nosi lub trzyma, oraz ile takich miejsc ma postać, od 1 do 20.
- `binding`: opcjonalne. Dostrojenie, inwestytura lub cokolwiek innego, co ogranicza liczbę przedmiotów, które jedna postać może mieć jednocześnie powiązane. `label` to nazwa tego mechanizmu, a `max` to wartość odczytywana z arkusza postaci, np. `{ "abilityScore": "nerve" }` lub wartość pochodna. Nie może odczytywać bieżącego stanu, bo liczba przedmiotów, które postać może powiązać, nie zmienia się z każdym ciosem.
- `carry`: opcjonalne. `stat` wskazuje statystykę liczbową będącą wagą przedmiotu, której `min` wynosi 0 lub więcej. `encumberedAbove` to ilość, jaką postać może nieść, zanim stanie się przeciążona, a `limit` (opcjonalne) to najwięcej, ile w ogóle może unieść. Obie wartości odczytuje się z arkusza tak jak `binding.max`. Bez `carry` waga nic nie znaczy i nikt nigdy nie jest przeciążony.
- `currencies`: opcjonalne, do sześciu rodzin, każda z od jednej do dziesięciu monet.
  - Monety jednej rodziny wymienia się na siebie według `value`, liczonego w najmniejszej monecie rodziny. Najmniejsza moneta jest więc warta 1, a żadne dwie monety rodziny nie mają tej samej wartości.
  - Dwóch rodzin nigdy nie wymienia się na siebie. Moneta innego kraju albo przysługi w danym świecie to osobna rodzina.
  - Identyfikator monety jest unikalny we wszystkich rodzinach, bo koszt przedmiotu wskazuje samą monetę.
  - `perWeight` (opcjonalne i tylko obok `carry`) to liczba monet rodziny ważących jedną jednostkę statystyki udźwigu.
  - Monety to pieniądze drużyny, noszone w torbach jak każdy przedmiot (zobacz niżej Pieniądze).
- `rarityCaps`: opcjonalne, najwyżej jeden na rzadkość. Najwięcej, ile może dać przedmiot wymyślony przez Mistrza Gry przy tej rzadkości: największa wartość każdej statystyki liczbowej wskazanej w `stats`, w jej własnym zakresie i jako liczba całkowita dla statystyki przyjmującej liczby całkowite, oraz `bonus` (Capability API 1.53), największa stała premia, jaką może dodać jeden z jego modyfikatorów działających, gdy przedmiot jest założony lub niesiony. Przy rzadkości z `bonus` premia w kościach jest pomijana, bo kości nie da się ograniczyć do liczby; kara nigdy nie jest ograniczana. Wymyślony przedmiot jest do tego dopasowywany; przedmioty wymienione w twoich katalogach należą do ciebie i nigdy nie są ograniczane.
- `propose`: domyślnie `true`. `false` zabrania Mistrzowi Gry wymyślania przedmiotów twojego zestawu zasad.
- `native`: domyślnie `true`. `false` wyłącza własne, nietypowane przedmioty trybu Game Mode w grach z twoim zestawem zasad (zobacz **Co odczytuje przedmioty**). Mistrz Gry nadal może wymyślać przedmioty, opisane słowami twojego zestawu.
- `freeform`: czym staje się przedmiot wpisany przez gracza. `"plain"` (domyślnie) zachowuje go jako przedmiot bez zasad, jak dotąd. `"refuse"` dopuszcza tylko przedmioty twojego zestawu zasad.
- `lootTables` (Capability API 1.63): opcjonalne, do 24. Co zostawiają stworzenia z wygranej walki i co daje `[loot:]` Mistrza Gry (zobacz niżej Łupy).

### Przedmiot

Katalog przedmiotów nie deklaruje `feeds`:

```json
{ "id": "outfitter", "label": "Outfitter", "holds": "items", "entries": [] }
```

Każdy wpis zawiera `item` zamiast `rows` lub `creature`:

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

- `id`, `label`, `summary` i `filters` działają jak w każdym wpisie. Etykieta jest nazwą przedmiotu.
- `category` jest wymagane. `rarity` i `tags` są opcjonalne, a wszystkie trzy wskazują identyfikatory z bloku. Przedmiot może mieć rzadkość tylko wtedy, gdy blok jakieś deklaruje.
- `stats`: wartości statystyk bloku. Każda jest wartością dopuszczalną dla swojej statystyki.
- `slots`: ile miejsc każdego rodzaju zajmuje przedmiot, nigdy więcej, niż ma postać.
- `stack`: najwięcej, ile mieści jeden stos, od 1 do 999 999. Bez tego stos mieści tyle, co każdy stos w trybie Game Mode. Przedmiot z `charges` występuje w stosie pojedynczo.
- `cost`: całkowita liczba `amount` jednej monety, wskazanej przez identyfikator `unit`. Pokazuje go selektor, a Mistrz Gry widzi go obok przedmiotu (zobacz niżej Pieniądze).
- `sold` (Capability API 1.65): `{ "place": "city" }`, najmniejsza z miejscowości twojego targu, w której przedmiot jest sprzedawany, z pierwszeństwem przed każdą regułą mówiącą co innego; od 1.69 może zamiast tego lub dodatkowo określać, ile sztuk miejscowość ma do sprzedania w ciągu dnia (`"limited": 2` lub `"limited": "1d3"`) (zobacz niżej Targi).
- `service` (Capability API 1.65): `true` dla noclegu, przejazdu, błogosławieństwa: kupowanych jak przedmiot i nigdy nienoszonych. Zakup usługi tylko płaci; nic innego nie umieszcza jej w torbie (dodanie usługi jest odrzucane jako `service`), a selektor pomija usługi. Usługa ma `cost` i nic, co dotyczy noszenia lub używania (`slots`, `stack`, `binds`, `worn`, `carried`, `requires`, `attack`, `use`, `charges`).
- `binds`: przedmiot trzeba powiązać, zanim zacznie działać jako założony. `restriction` (opcjonalne) opisuje słowami, kto może go powiązać, a `cursed: true` oznacza przedmiot, który nie puszcza. Tylko zestaw zasad z `binding` może mieć przedmioty wymagające powiązania.
- `worn` i `carried` (opcjonalne, Capability API 1.53): działanie przedmiotu na testy i rzuty obronne posiadacza, gdy jest założony i gdy jest tylko niesiony, (1.54) atrybuty, które ustawia lub podnosi, oraz (1.56) jego działanie w walce. Zobacz [Testy poza walką](#checks-outside-a-fight) i [Pancerz i efekty założonych przedmiotów w walce](#armor-and-worn-effects-in-a-fight).
- `requires` (opcjonalne, Capability API 1.54): czego przedmiot wymaga od noszącego i co obowiązuje, dopóki ten nie spełnia wymagań. Zobacz [Testy poza walką](#checks-outside-a-fight).
- `attack` (opcjonalne, Capability API 1.55): działanie przedmiotu jako broni w walce, gdy jest założony, (1.57) czym strzela i ile ma załadowane oraz (1.58) jego inne tryby, atak drugą ręką, minimum obrażeń i stany nakładane przy trafieniu. Zobacz [Broń w walce](#weapons-in-a-fight), [Amunicja i przeładowanie](#ammunition-and-reloading) oraz [Tryby, druga broń, minimum obrażeń i stany przy trafieniu](#modes-a-second-weapon-a-floor-and-conditions-on-a-hit).
- `use` i `charges` (opcjonalne, Capability API 1.59): działanie użycia przedmiotu w walce i poza nią, ładunki, które to użycie wydaje, oraz (1.60) przywracana przez niego pula. Zobacz [Używanie przedmiotów w walce](#using-items-in-a-fight) i [Używanie przedmiotów poza walką](#using-items-outside-a-fight).

Przedmiot nie ma `mechanics`: jego działanie zapisuje się w bloku `item`.

### Co odczytuje przedmioty

Wszystko powyższe jest sprawdzane przy imporcie zestawu zasad, a twoje katalogi przedmiotów są dostarczane razem z nim. W grze odczytuje je ekwipunek trybu Game Mode:

- **Twoje przedmioty są w torbie.** Stos może być jednym z twoich przedmiotów i pozostaje nim, jakkolwiek nazwie go gracz. Przycisk **From the ruleset** (z zestawu zasad) w ekwipunku otwiera selektor wszystkich przedmiotów z twoich katalogów, z wyszukiwaniem i filtrami katalogu, a warstwa ukrywająca wpis ukrywa go także tam.
- **Nazwy odnajdują twoje przedmioty.** Nazwa wpisana przez gracza lub zapisana przez Mistrza Gry w `[inventory: action="add"]`, będąca `label` jednego z twoich przedmiotów, niezależnie od wielkości liter, dodaje ten przedmiot. Gdy dwa przedmioty mają tę samą etykietę, nazwa odnajduje ten, który twoje katalogi wymieniają jako pierwszy.
- **`stack`** jest przestrzegane: dodanie, ustawienie, połączenie lub przekazanie ponad ten limit wypełnia stos i zaczyna nowy.
- **`freeform: "refuse"`** zostawia graczowi tylko twoje przedmioty: selektor i nazwy, które są twoimi przedmiotami.
- **`native: false`** zostawia Mistrzowi Gry tylko twoje przedmioty i te, które sam wymyśli: nazwa, która nie jest żadnym z nich, jest odrzucana (`not-ruleset-item`), choć nadal można dodać więcej czegoś, co już jest w posiadaniu, o czym mówią jego instrukcje. Walka nie pyta już modelu, co robią przedmioty z ekwipunku, więc walczą tylko twoje przedmioty: w walce według zestawu zasad trzymana broń i użycia twoich przedmiotów (zobacz niżej Broń w walce), a w bitwie Classic lub Tactical twoje przedmioty z `use` (zobacz niżej Przedmioty w bitwach Classic i Tactical). To, co wpisuje gracz, nadal podlega `freeform`, a rzeczy niesione przez drużynę w obu przypadkach wracają w nowej sesji.
- **Czym jest przedmiot**, widać na wybranym stosie: jego kategorię, rzadkość i tagi według etykiet, dawane statystyki, opis i pojemność jednego stosu. Selektor pokazuje też jego `cost`. Mistrz Gry widzi każdy twój przedmiot z ekwipunku z kategorią, rzadkością, tagami i statystykami pozostawionymi jako `promptVisible`, np. `Hand axe [Weapon, Common, Thrown; Damage 1d6, Reach close]`.

- **`slots`**: przedmiot zajmujący miejsca może założyć ten, kto go niesie, jeśli ma te miejsca wolne, a jedna sztuka z większego stosu trafia do osobnego stosu, aby można ją było założyć. Ekwipunek pokazuje zajęte miejsca każdej postaci.
- **`binding`**: przedmiot z `binds` można powiązać, do `binding.max` odczytanego z arkusza jego posiadacza (postać bez arkusza odczytuje pusty arkusz). Przedmiot `cursed` po powiązaniu pozostaje powiązany: gracz nie może zerwać powiązania, zdjąć go, oddać ani usunąć, a klątwę może zakończyć tylko Mistrz Gry.
- **`carry`**: przedmiot waży tyle, ile wynosi jego `carry.stat` (przedmiot bez tej statystyki nic nie waży), a obciążenie postaci to waga jej torby, porównywana z `encumberedAbove` i `limit` odczytanymi z jej arkusza. Przedmiot dodany do wspólnego widoku ekwipunku, przez gracza lub przez Mistrza Gry bez `who=`, trafia do tego, kto może go nieść bez przeciążenia (najpierw gracz, potem drużyna po kolei), a gdy nikt nie może wziąć całości, jest dzielony według wolnego miejsca każdego. Nic nie przekracza niczyjego `limit`: to, czego nikt nie uniesie, zostaje na miejscu, a Mistrz Gry jest o tym informowany. Nadaj statystyce wagi `"integer": false` dla wag takich jak ćwierć funta.
- **Mistrz Gry wymyśla przedmioty twoimi słowami.** Jeśli nie ustawisz `propose: false`, jego `[inventory: action="add"]` może opisać nowy przedmiot: `like=` jeden z twoich przedmiotów jako punkt wyjścia, a następnie dowolne z `category=`, `rarity=`, `tags=`, `stats=` (pary `id=value`), `slots=` (`id=count`), `binds=` (`yes`, `cursed` lub `no`), `worn=` i `carried=` (zmiany rozdzielone `;`, każda to `+N`, `-N`, `advantage`, `disadvantage` lub `fails` dla rzutów obronnych, na umiejętnościach lub rzutach obronnych wskazanych nazwą albo na `checks` lub `saves` dla wszystkich, `+N` lub `-N` na nazwie atrybutu, aby go podnieść lub obniżyć, a tam, gdzie masz walki, `+N`, `-N`, ułatwienie lub utrudnienie na atakach oraz `+N` lub `-N` na obronie lub twoim słowie na nią: `worn="+1 Sneak; disadvantage on Sway checks"`, `carried="+1 Brawn"`, `worn="+1 Guard"` albo `none`) i `summary=`, każdą część przez identyfikator lub etykietę. Statystyka przedmiotu, którą twoja obrona już liczy (Guard w Ember Roads sumuje `guard` założonych rzeczy), to sposób, w jaki pancerz ją podnosi, więc zmiana obrony przy założeniu obok takiej statystyki jest pomijana, a odpowiedź o tym informuje: mały model pisze `guard=1` i `+1 Guard` dla jednego +1. Engine odczytuje też to, co mały model zwykle pisze zamiast tego: liczbę po nazwie (`Brawn +1`), "bonus" po atakach lub obronie (`+1 attack roll bonus`), `none` dla pustej listy tagów, statystyk lub miejsc oraz `worn=`, `carried=` lub `summary=` wewnątrz `stats=`, gdy nie masz statystyki o takiej nazwie. Engine zachowuje tylko to, co jest w twoim bloku: nieznana kategoria, tag, statystyka, miejsce, umiejętność lub rzut obronny są pomijane, rzadkość, której nie masz, staje się twoją najniższą, liczba jest dopasowywana do zakresu statystyki, a potem do `rarityCaps` swojej rzadkości, premia przy założeniu lub niesieniu – do `bonus` swojej rzadkości (także część przejęta przez `like`), a nazwa będąca jednym z twoich przedmiotów oznacza po prostu ten przedmiot. Odpowiedź mówi Mistrzowi Gry, co zmieniono (nigdy w odniesieniu do statystyki, której mu nie pokazujesz), a szczegóły przedmiotu pokazują graczowi każdą zmianę. Gra przechowuje przedmiot, więc ta sama nazwa oznacza go do końca gry, a nowa sesja zachowuje go, dopóki ktoś go jeszcze ma.
- Mistrz Gry może zakładać i zdejmować twoje przedmioty (`equip` i `unequip`) swoją komendą ekwipunku, gdy masz `slots`, oraz wiązać je i zrywać powiązanie (`bind` i `unbind`), gdy masz `binding`: dowiaduje się tylko o tych, które ma twój zestaw zasad. Widzi obciążenie każdej postaci, powiązane przedmioty i miejsca oraz to, co jest założone lub powiązane.

- **Działanie przedmiotu założonego lub niesionego** widać na wybranym stosie i w selektorze ("While worn: -1 on checks (Sneak)"), a Mistrz Gry widzi je obok przedmiotu (`worn: -1 on checks (Sneak)`). Testy poza walką je uwzględniają (zobacz niżej Testy poza walką).
- **Działanie broni** widać tak samo ("Attack (Action): Brawn to hit, 1d6 + Brawn cut damage"), a walka oferuje broń, gdy jest trzymana (zobacz niżej Broń w walce).
- **Działanie użycia przedmiotu** widać tak samo ("Use (Action): heals 1d4 + 1, range 0 paces, used up"), wraz z pozostałymi ładunkami. Oferuje je walka, a także przycisk **Use** (użyj) w ekwipunku i `[inventory: action="use"]` Mistrza Gry (zobacz niżej Używanie przedmiotów w walce i Używanie przedmiotów poza walką).

W bitwie Classic lub Tactical każdy twój przedmiot z `use` robi to, co tam zapisano, a przedmiot bez niego nie jest oferowany (zobacz niżej Przedmioty w bitwach Classic i Tactical). Niezależnie od `native` arkusz może odczytywać twoje przedmioty (niżej), trzymana broń jest atakiem w walce według zestawu zasad i strzela tym, co pobiera z torby, działanie przedmiotu założonego lub niesionego liczy się w takiej walce (zobacz niżej Pancerz i efekty założonych przedmiotów w walce), przedmiot z `use` jest używany według własnych zasad, w walce i poza nią (zobacz niżej Używanie przedmiotów w walce i Używanie przedmiotów poza walką), a twoje `currencies` są pieniędzmi drużyny (zobacz niżej Pieniądze).

<a id="items-on-the-sheet"></a>

### Przedmioty w arkuszu

Odwołanie do wartości może odczytywać przedmioty posiadane przez postać przez `itemStat` (Capability API 1.52). Ember Roads dodaje ochronę pancerza noszonego przez podróżnika do jego Guard, którą musi pokonać cios:

```json
{
  "id": "guard",
  "label": "Guard",
  "op": "sum",
  "of": [{ "const": 6 }, { "abilityMod": "wits" }, { "itemStat": { "stat": "guard", "from": "worn", "pick": "sum" } }]
}
```

- `from`: `"worn"` to przedmioty na postaci: przedmiot zajmujący miejsca, gdy jest założony, przedmiot z `binds`, gdy jest powiązany, a przedmiot z jednym i drugim, gdy spełnia oba warunki. Przedmiot bez żadnego z nich nigdy nie jest założony. `"carried"` to reszta tego, co postać ma, a `"all"` to jedno i drugie.
- `pick`: `"sum"` dodaje wartość każdego przedmiotu pomnożoną przez liczbę sztuk, `"max"` i `"min"` odczytują najwyższą lub najniższą pojedynczą wartość, a `"count"` liczy przedmioty według liczby sztuk.
- `stat` wskazuje jedną ze statystyk twoich przedmiotów. `sum`, `max` i `min` wymagają statystyki `number`. `count` może ją pominąć, aby policzyć każdy przedmiot, albo wskazać dowolną statystykę, aby liczyć tylko przedmioty, które ją dają.
- `slot`, `category` i `tag` (każde opcjonalne) zostawiają tylko przedmioty zajmujące to miejsce, należące do tej kategorii lub mające ten tag.
- `default` (opcjonalne, 0 po pominięciu) to wartość odczytywana, gdy nie wybrano żadnego przedmiotu albo żaden nie daje tej statystyki.

Przedmioty zmieniają się podczas gry, więc `itemStat` podlega tej samej regule co odczyt bieżący: `max` puli lub toru, premia biegłości, `binding.max`, liczby `carry` oraz skalowana kolumna lub skalowanie katalogu nie mogą go odczytywać, nawet przez wartość pochodną. Wszędzie indziej, gdzie odczytuje się liczbę, jest to możliwe: w wartości pochodnej, `adjust` testu, `cap` umiejętności, obronie, pochłanianiu lub inicjatywie w walce.

Przedmioty postaci to te w jej własnej torbie. Karta gracza (ta nazwana tak jak persona czatu, a w przeciwnym razie pierwsza) odczytuje torbę gracza, a każda inna karta – własną. Liczą się tylko przedmioty twojego zestawu zasad, bo zwykły przedmiot nie ma statystyk do odczytania. Arkusz na ekranie, test, blok arkusza dla Mistrza Gry i rozpoczynająca się walka odczytują to, co każda postać ma w danej chwili. Poza grą, w edytorze arkusza lub przeglądzie importu, nikt nic nie ma, a `itemStat` odczytuje swoje `default`.


<a id="checks-outside-a-fight"></a>

### Testy poza walką

Test lub rzut obronny, o który Mistrz Gry prosi poza walką, odczytuje więcej niż arkusz: aktywne stany postaci, osiągnięte przez nią poziomy, działanie jej przedmiotów założonych lub niesionych oraz koszt niespełnionego wymagania założonego przedmiotu. To ten sam słownik, którego używają stany w walce (Capability API 1.53 dla kluczy przedmiotów i zawężania).

Przedmiot określa swoje działanie przez `worn`, obowiązujące, gdy jest założony (i powiązany, jeśli wymaga powiązania), oraz `carried`, obowiązujące, gdy jest tylko niesiony. Skórzany płaszcz z Ember Roads skrzypi przy skradaniu, a kamień drogowy sprawia, że ludzie z karawan ci ufają:

```json
"worn": { "modifiers": [{ "to": "checks", "skills": ["sneak"], "flat": -1 }] }
```

```json
"carried": { "modifiers": [{ "to": "checks", "skills": ["sway"], "flat": 1 }] }
```

Każde przyjmuje te części stanu, które odczytuje test:

- `effects`: `own-checks-advantage`, `own-checks-disadvantage`, `own-saves-advantage` i `own-saves-disadvantage`.
- `modifiers`: zmiany `to` testów lub rzutów obronnych, z `flat`, `dice` (i `minus`), `mode` oraz własnymi `skills` lub `saves`, dokładnie jak w stanie.
- `failsSaves`: rzuty obronne automatycznie nieudane dla posiadacza, bez rzutu.
- `skills` i `saves`: których umiejętności lub rzutów obronnych dotyczą efekty i modyfikatory niewskazujące własnych.
- `abilities` (Capability API 1.54): zmieniane atrybuty, według identyfikatora, każdy jako `{ "set": N }` (co najmniej N: wyższa wartość zostaje) lub `{ "add": N }` (liczba ujemna obniża). Działają przed obliczeniem arkusza, więc wszystko, co odczytuje atrybut, odczytuje zmienioną wartość: arkusz na ekranie, wartości pochodne, testy, blok arkusza dla Mistrza Gry i rozpoczynająca się walka. Najpierw stosuje się dodawanie, potem najwyższe `set`, a własne `min` i `max` atrybutu nadal obowiązują. Maksimum puli lub toru i premię biegłości oblicza się z samego arkusza, więc przedmiot ich nie zmienia.

Rękawice z wołowej skóry z Ember Roads dają słabemu uściskowi siłę poganiacza bydła:

```json
"worn": { "abilities": { "brawn": { "set": 2 } } }
```

Działanie przedmiotu w walce (obronę, ataki, szybkość, inne efekty walki oraz obrażenia i stany, przed którymi chroni) opisuje [Pancerz i efekty założonych przedmiotów w walce](#armor-and-worn-effects-in-a-fight); test poza walką odczytuje tylko powyższe części. Jeden przedmiot działa raz, niezależnie od tego, ile jego stosów ktoś ma.

Przedmiot może też czegoś wymagać od noszącego przez `requires`: wartości z jego arkusza (`value`, dowolne odwołanie do wartości, odczytywane jak wszystko podczas gry), jej minimum (`atLeast`) i tego, co obowiązuje, dopóki wymaganie nie jest spełnione (`otherwise`, w tym samym słowniku, z tym że nie może zmieniać atrybutu, bo wymaganie może go odczytywać). Najwyżej cztery. Łopata grabarska z Gravewatch to ciężka praca dla słabego strażnika:

```json
"requires": [
  {
    "value": { "abilityScore": "sinew" },
    "atLeast": 3,
    "otherwise": { "modifiers": [{ "to": "checks", "skills": ["dig"], "flat": -1 }] }
  }
]
```

Wymaganie odczytuje atrybut po zmianach wprowadzonych przez przedmioty noszącego, więc rękawice ustawiające Sinew na 3 spełniają wymaganie łopaty.

W teście:

- Każdy modyfikator, który go dotyczy, dodaje swoją liczbę, a jego kości są rzucane: stała liczba w teście sumowanym, kości w puli. Test atrybutu i test, którego zestaw zasad nie potrafi nazwać, odczytują tylko to, co nie jest do niczego zawężone, a rzut obronny odczytuje tylko to, co dotyczy rzutów obronnych.
- Ułatwienia i utrudnienia ze wszystkich źródeł, w tym `mode=` Mistrza Gry, znoszą się: przy choćby jednym z każdego test rzuca się raz. Przechyla się tylko zestaw zasad, który w ogóle rzuca dwa razy (`resolution.advantage`).
- Rzut obronny automatycznie nieudany z powodu stanu lub przedmiotu jest nieudany bez rzutu i nic się na niego nie wydaje.
- Zapis podaje, co go zmieniło: `effects="-1"` dla liczby, `from="Leather coat"` dla źródła zmiany i `automatic="true"` dla rzutu obronnego nieudanego bez rzutu. Karta rzutu pokazuje to samo, a Mistrz Gry odczytuje to w następnej turze. Wie, że Engine stosuje te zmiany, więc nie dodaje ich ponownie.

Gravewatch pokazuje to samo w puli: powiązany Dawn bell (dzwonek świtu) dodaje kość do Ward, a Rattled odejmuje jedną od Soothe i Barter.

<a id="weapons-in-a-fight"></a>

### Broń w walce

Przedmiot może być bronią (Capability API 1.55). Jego `attack` ma kształt [wierszy ataków](#combat-a-fight-your-own-rules-resolve) bloku walki, z wartościami zamiast kolumn, a [walka według zestawu zasad](#combat-a-fight-your-own-rules-resolve) oferuje go jako atak, gdy przedmiot jest założony: trzymany w rękach, których wymaga, i powiązany, jeśli wymaga powiązania. Broń schowana lub tylko niesiona niczego nie oferuje, a wiersze ataków i własne akcje stworzeń działają dokładnie jak wcześniej. Toporek z Ember Roads:

```json
"attack": {
  "budget": "act",
  "toHit": { "abilities": { "stat": "swing" } },
  "damage": { "dice": { "stat": "damage" }, "abilities": { "stat": "swing" }, "type": "cut" },
  "reach": 2,
  "range": { "normal": 10, "long": 20 }
}
```

- `budget`: zużywany budżet, jeden z `combat.economy.budgets`.
- `toHit`: co dodaje do trafienia. `abilities` to jeden lub więcej identyfikatorów atrybutów, a liczy się najlepszy z nich (broń finezyjna wymienia dwa). `skill` dodaje to, co dodaje test tej umiejętności, z atrybutem ataku zamiast własnego atrybutu umiejętności, tak jak w wierszu ataku. `proficiency` to wartość z arkusza posiadacza: gdy jest większa od 0, dodaje się premię biegłości zestawu zasad (`{ "const": 1 }`, aby zawsze). `bonus` to liczba. `target` to własny próg pojedynczej kości dla ataku tą bronią w walce pulami (łopata z Gravewatch liczy każdą kość od 6 zamiast od 7 jak pula) i może go podać tylko zestaw `dice-pool`, którego `target` może się zmieniać; w walce sumowanej to samo wyraża `bonus`.
- `damage`: zadawane `dice`, dodany najlepszy z jej `abilities` obrażeń, `bonus` i `type`. Walka sumowana wymaga `dice`; w walce pulami trafienie zadaje swoje sukcesy, a `dice` dodaje tyle kolejnych kości puli.
- `reach` i `range` (`normal` oraz `long` dla tego, dokąd broń jeszcze sięga dalej) są w jednostce `combat.distance` i wymagają jej. Broń z obydwoma jest rzucana, tak jak wiersz ataku: z bliska zamach, dalej rzut. Bez żadnego sięga jednego pola.
- `versatile`: `dice` zadawane zamiast tego, gdy w każdym zajmowanym przez nią miejscu wśród rzeczy założonych przez posiadacza jest drugie tyle wolnego miejsca: włócznia w jednej ręce przy wolnej drugiej.
- `strikes`: ile uderzeń kupuje jedno zużycie, odczytywane z arkusza posiadacza, tak jak `strikes` wiersza ataku. Broń bez tego daje jedno uderzenie za zużycie, niezależnie od liczby ataków jej właściciela.

Każda z tych liczb lub słów może zamiast tego odczytywać własną statystykę przedmiotu: `{ "stat": "damage" }` to `damage` przedmiotu, a statystyka, której przedmiot nie daje, działa tak, jakby wartości nie zapisano. `abilities` i `skill` odczytują statystykę wyliczeniową, której słowa są identyfikatorami atrybutów lub umiejętności, a `type` – statystykę tekstową lub wyliczeniową. Przedmiot wymyślony przez Mistrza Gry przez `like=` na wzór broni też jest więc bronią, walczącą własnymi statystykami, dopasowanymi do `rarityCaps` jak każda statystyka (wartość zapisana wprost jest kopiowana bez zmian). Przedmiot wymyślony bez miejsca i powiązania nigdy nie jest założony, więc powstaje bez ataku. Mały model zwykle opisuje broń i pomija `like=`, więc broń wymyślona w kategorii twoich broni bez punktu wyjścia walczy jak ta twoja broń z tej kategorii, która jest do niej najbardziej podobna z nazwy (słowo zawarte w którejkolwiek z nazw, więc "crossbow" przypomina "bow"), albo jak pierwsza z nich, przejmując każdą odczytywaną przez atak statystykę, której propozycja nie podała, a odpowiedź o tym informuje ("It fights as Hand axe does."). Formularz propozycji Mistrza Gry również mówi mu, że broń utworzona na wzór innej walczy jak ona.

Tagi broni są przenoszone przez jej ciosy. Wpis `resist` lub `immune` stworzenia może określać, co się przez niego przebija: `{ "type": "tearing", "except": ["silver"] }` jest przyjmowane w całości od broni z tagiem `silver`, a od wszystkiego innego – z odpornością. Tak zapisano upiora grobowego z Gravewatch, a srebrny gwóźdź trumienny z tego zestawu przebija się przez jego odporność.

Szczegóły broni i wiersz dla Mistrza Gry podają jej działanie: `attack (Action): Brawn to hit, 1d6 + Brawn cut, reach 2 paces, range 10 to 20 paces`, z `1d8 with a hand free` dla oszczepu na dziki z Ember Roads i `at 6` obok łopaty z Gravewatch.

<a id="ammunition-and-reloading"></a>

#### Amunicja i przeładowanie

Broń może czymś strzelać i może mieć własną liczbę załadowanych pocisków (Capability API 1.57). Łuk myśliwski z Ember Roads strzela strzałami, a pistolet strażniczy z Gravewatch mieści jedną kulę, ładowaną z kul i prochu strażnika:

```json
"ammo": { "tag": "arrow", "recover": 0.5 }
```

```json
"ammo": { "tag": "shot" },
"clip": { "max": 1, "reload": "act" }
```

- `ammo`: czym strzela broń, według jednego z `tags` twoich przedmiotów. Każdy atak zabiera `perAttack` (1, gdy nic nie podano) przedmiotów z tym tagiem niesionych przez posiadacza, założonych lub nie, z pierwszego takiego stosu w jego torbie, potem z następnego. Broń jest oferowana tylko wtedy, gdy ma ich dość, a menu walki podaje, ile zostało. Po walce wygranej przez drużynę `recover` (od 0 do 1) tego, co każdy wystrzelił, wraca do stosu, z którego pochodziło: udział jest sumowany przez całą walkę i zaokrąglany w dół raz dla każdego stosu, więc połowa z pięciu strzał wystrzelonych z jednego kołczanu to dwie. Przegrana walka lub ucieczka nic nie zwraca.
- `clip`: liczba załadowanych pocisków przechowywana w samej broni. `max` to jej pojemność, zapisana wprost lub odczytana ze statystyki liczbowej przedmiotu, a `reload` to budżet zużywany przy przeładowaniu. Atak zużywa to, co załadowane, zamiast brać z torby, a broń jest oferowana tylko wtedy, gdy ma załadowane dość na jeden atak. Opcja **Reload** (przeładowanie) w menu walki zużywa `reload` i ładuje broń: z torby, gdy broń ma `ammo` (tyle, ile torba jeszcze ma, jeśli to mniej), a do pełna, gdy go nie ma. Liczba jest przechowywana na stosie broni w ekwipunku, więc pistolet opróżniony w jednej walce jest pusty także w następnej. Broń, z której nikt jeszcze nie strzelał, jest załadowana; tak samo broń, którą gracz dołoży do stosu kilku takich samych, bo liczba załadowanych pocisków należy do jednej broni. Pocisków z magazynka nie zbiera się po walce, więc `recover` dotyczy broni bez magazynka.

To, co walka wystrzeliwuje, ładuje i zbiera, jest zapisywane w ekwipunku przy każdym kroku, tak jak zdrowie drużyny, a dziennik przygody (Adventure Journal) podaje, co zużyto. Krok, którego stos zniknął lub zawiera mniej, niż zakładała walka, jest odrzucany, tak jak zużyty przedmiot, którego walka nie może znaleźć. Dziennik walki podaje każdy strzał, przeładowanie i zebranie ("Hunting bow: 2 left to shoot.", "Ada loads 1 into Watch pistol: 1 of 1 loaded."), a szczegóły przedmiotu i wiersz dla Mistrza Gry podają, czym broń strzela i ile mieści: `ammunition Arrow (1 an attack, 50% picked up after a won fight)` i `holds 1, reload (Act)`.

<a id="modes-a-second-weapon-a-floor-and-conditions-on-a-hit"></a>

#### Tryby, druga broń, minimum obrażeń i stany przy trafieniu

Broń może mieć inne sposoby wykonania ataku, partnerkę w drugiej ręce, minimalne zadawane obrażenia i stany nakładane na trafiony cel (Capability API 1.58). Łuk myśliwski z Ember Roads może wypuścić salwę, a srebrny gwóźdź trumienny z Gravewatch można trzymać w każdej ręce i naznacza on to, w co wbije się głęboko:

```json
"modes": [{ "id": "volley", "label": "Volley", "ammo": 2, "toHit": -2, "targets": 2 }]
```

```json
"offHand": true,
"onHit": [{ "condition": "marked", "atLeast": 2, "rounds": 2 }]
```

- `modes`: do sześciu innych sposobów wykonania ataku, każdy z `id`, `label` i tym, co zmienia. `ammo` to liczba pocisków wystrzeliwanych przez jeden atak w tym trybie, więc broń musi mieć `ammo` lub `clip`, z których strzela. `toHit` to dodatek do trafienia: kości w walce pulami i liczba w walce sumowanej. `target` przesuwa próg pojedynczej kości w walce pulami o tyle (od własnego progu broni, a w przeciwnym razie od progu puli; `+1` oznacza trudniej), jeśli próg puli może się zmieniać. `targets` to liczba celów, w które można wycelować, każdy z własnym rzutem ataku. Po wybraniu ataku menu walki pyta o sposób (po stylu inicjatywy, jeśli są style), podając przy każdym spodziewany efekt, i oferuje tylko tryby, na które posiadacz ma pociski. Członek drużyny prowadzony przez Engine rozważa każdy tryb wymierzony w jeden cel, a tryb na kilka celów zostawia graczowi. W oknie atak jest wykonywany w zwykłej postaci.
- `offHand`: broń do drugiej ręki. Blok walki wskazuje budżet zużywany przez drugi atak i to, czy jego obrażenia zachowują dodatni atrybut:

  ```json
  "offHand": { "budget": "quick", "ability": "full" }
  ```

  Gdy posiadacz zaatakował w tej turze jedną bronią oznaczoną `offHand`, każda inna taka założona broń oferuje drugi atak za ten budżet, nazwany w menu "Silver coffin nail, off hand". `ability` to `full` (domyślnie) lub `penalty-only`, które dodaje atrybut obrażeń tylko wtedy, gdy coś odejmuje, jak walka dwiema broniami w 5e. Atak drugą ręką to jeden cios, niezależnie od liczby uderzeń kupionych przez główny atak, i nigdy nie jest uderzeniem w odchodzącego. Broń oznaczona `offHand` wymaga `offHand` w bloku walki.
- `floor`: najmniej, ile zadaje trafienie, zapisane wprost lub odczytane ze statystyki liczbowej przedmiotu. Obrażenia w walce pulami po pochłonięciu albo obrażenia w walce sumowanej są do niego podnoszone, zanim odporność zmniejszy je o połowę, a dziennik to odnotowuje. Cios wydający rzuca liczbą swojego wykonawcy i nigdy nie jest podnoszony.
- `onHit`: do czterech twoich stanów, które cel otrzymuje, gdy zadane przez cios obrażenia osiągnęły `atLeast`, na `rounds` jego własnych tur lub, bez tego, dopóki coś ich nie zdejmie. Obrażenia to to, co do niego dotarło po pochłonięciu i odpornościach; cios zabierający odbiera inicjatywę, a nie zdrowie, więc nie nakłada żadnego stanu.

Szczegóły broni i wiersz dla Mistrza Gry podają to wszystko: `modes Volley (2 shots, -2 to hit, up to 2 targets)`, `off hand (Quick)`, `at least 1 on a hit before resistance` i `Marked for 2 rounds when a hit deals 2 or more`.

<a id="armor-and-worn-effects-in-a-fight"></a>

### Pancerz i efekty założonych przedmiotów w walce

Działanie przedmiotu założonego lub niesionego może też zmieniać walkę (Capability API 1.56) dla jego posiadacza, obok jego stanów i poziomów. Każde przyjmuje te części stanu, które odczytuje walka:

- `effects`: dowolne efekty stanu oprócz czterech, których nie może mieć poziom (`half-move-to-stand`, `ends-on-damage`, `cannot-target-source`, `cannot-approach-source`), ponieważ nikt nie nałożył przedmiotu na posiadacza i przedmiot nigdy nie kończy działania sam: `own-attacks-advantage`, `attacks-against-disadvantage`, `resist-all`, `speed-zero` i pozostałe.
- `modifiers`: zmiany `to` obrony, ataków i szybkości, a także testów i rzutów obronnych, dokładnie jak w stanie; w walce `dice-pool` zmiana ataków to stała liczba kości, tak jak w stanie.
- `resist`, `vulnerable` i `immune`: rodzaje obrażeń, z których posiadacz otrzymuje połowę, podwójną wartość lub nic, tak jak u stworzenia (i sprawdzane względem `combat.damageTypes`, jeśli jakieś deklarujesz). `conditionImmunities`: twoje własne stany, które nigdy nie są nakładane na posiadacza.

Przeklęty pierścień wdowy z Gravewatch kosztuje jedną kość przy każdym ataku, powiązany Dawn bell chroni posiadacza przed stanem Rattled, a niesiony kamień drogowy z Ember Roads chroni przed żarem:

```json
"worn": { "modifiers": [{ "to": "attacks", "flat": -1 }] }
```

```json
"worn": { "modifiers": [{ "to": "checks", "skills": ["ward"], "flat": 1 }], "conditionImmunities": ["rattled"] }
```

```json
"carried": { "modifiers": [{ "to": "checks", "skills": ["sway"], "flat": 1 }], "resist": ["burn"] }
```

Walka odczytuje każdy przedmiot raz, pod nazwą stosu ("Widow's ring" obok rzutu w dzienniku i przy obronie), tak jak test; `otherwise` niespełnionego wymagania liczy się także w walce, więc pancerz zbyt ciężki dla noszącego może kosztować szybkość. Szczegóły przedmiotu i wiersz dla Mistrza Gry podają to wszystko ("While worn: -1 on attacks", "worn: +1 on checks (Ward), immune to Rattled"). Pancerz, który po prostu dodaje do obrony lub pochłaniania, to statystyka przedmiotu odczytywana przez `itemStat` (zobacz Przedmioty w arkuszu), tak jak Guard w Ember Roads odczytuje założony płaszcz.

**Twardość.** Gdy inicjatywa jest liczbą przesuwaną atakami, `combat.pool.hardness` to twardość walczącego, odczytywana z jego arkusza na początku walki (twardość jego pancerza, przez `itemStat`), a stworzenie podaje własne `hardness`. Cios wydający, którego kości są poniżej tej wartości, trafia i nic nie robi: liczba wykonawcy i tak wraca do wartości bazowej, dziennik to odnotowuje ("Ada's Grave spade lands on the warden with 5 dice, below a hardness of 6, and does nothing."), a menu nie prognozuje dla niego obrażeń. Może ją mieć tylko zestaw zasad ze stylem wydającym, a stworzenie zapisane jako arkusz bierze ją z arkusza.

<a id="using-items-in-a-fight"></a>

### Używanie przedmiotów w walce

Przedmiotu można użyć w walce (Capability API 1.59): okład przyłożony do rany, wypity tonik, dzwonek, w który się dzwoni. Jego `use` określa działanie użycia słowami `mechanics` wpisu katalogu (zobacz [Co walka odczytuje z `mechanics`](#what-a-fight-reads-from-mechanics)): `kind` (`heal`, `attack`, `buff` lub `debuff`), `amount`, `damageType`, `plus`, `attackRoll`, `save`, `applies`, `temporary`, `range`, `area`, `targets`, `targetCount` i `friendlyFire`, ze zużywanym `budget` albo `free`. Okład z Ember Roads oraz rozgrzewający tonik i dzwonek świtu z Gravewatch:

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

- Użycie jest w menu walki w grupie **Items** (przedmioty), pod nazwą przedmiotu ("Poultice"), niezależnie od nazwy stosu. Przedmiotu zajmującego miejsca lub wymagającego powiązania używa się, gdy jest założony (i powiązany, jeśli tego wymaga), jak dzwonka; każdego innego – gdy jest niesiony.
- Zużywa budżet albo jest `free` i płaci samym sobą: `cost`, `perCostStep`, `check`, `concentration`, `reaction`, `scales`, `gives`, `standard` i `rider` są dla wpisu katalogu w arkuszu, a w użyciu są odrzucane.
- `toHit` w użyciu z `attackRoll` to dodatek do trafienia, wyrażany tak jak `toHit` broni (zobacz Broń w walce): atrybuty, umiejętność, premia oraz, w walce pulami, próg pojedynczej kości. Bez tego rzut niczego nie dodaje.
- `saveDifficulty`: liczba, przeciw której wykonuje się wymagany rzut obronny (własny, kończący nakładany stan lub składnika bez `difficulty`), zapisana wprost lub odczytana ze statystyki liczbowej przedmiotu. Wpis w arkuszu odczytuje tę liczbę ze źródła swojego katalogu; przedmiot go nie ma, więc podaje własną.
- `consumes: true` odejmuje jedną sztukę ze stosu przy każdym użyciu, a menu podaje, ile zostało. Ostatnia użyta sztuka zabiera ze sobą stos.
- `restore` (Capability API 1.60) przywraca część puli każdego arkusza, na który trafia, tak jak worek krwi przywraca krew: `{ "pool": "resolve", "amount": { "flat": 1 } }`, z `dice` i `flat` jak w `amount`. Wskazuje jedną z twoich bieżących pul, nigdy pulę zdrowia (zdrowie wraca przez leczenie), i występuje w użyciu pomocnym: `heal` lub `buff`. Stworzenie nie ma pul, więc nic nie otrzymuje. Dziennik to odnotowuje ("Ada gets back 1 Resolve, and is on 3 of 4."), a członek drużyny prowadzony przez Engine nie rusza pełnej puli. Rozgrzewający tonik z Gravewatch przywraca punkt Resolve obok leczenia.
- `charges` zużywa tyle własnych `charges` przedmiotu, których `max` to pojemność, od 1 do 100, zapisana wprost lub odczytana ze statystyki liczbowej przedmiotu (przedmiot, którego ta statystyka wynosi mniej niż 1, jest odrzucany, a powyżej 100 mieści 100). Liczba jest przechowywana na stosie przedmiotu w ekwipunku, więc dzwonek użyty dwa razy w jednej walce ma w następnej jeden ładunek, a dzwonek, którego nikt nie użył, jest pełny. Ładunki należą do jednego przedmiotu, więc przedmiot, który je ma, ma `stack` równe 1. Użycie albo zużywa przedmiot, albo wydaje ładunki, nigdy oba naraz, a ładunki przedmiotu zawsze wydaje jego użycie. Dochodzą dwa klucze `charges` (Capability API 1.61), takie jak w dzwonku świtu z Gravewatch:

  ```json
  "charges": { "max": 3, "recharge": { "rests": ["vigil"], "amount": "max" }, "breaksOn": { "die": 20, "atMost": 1 } }
  ```

  - `recharge`: które z twoich `rests` przywracają ładunki i ile: `"max"` napełnia przedmiot, a ilość (`{ "dice": "1d6", "flat": 1 }`) przywraca tyle, nigdy ponad `max`. Odpoczynek przywraca ładunki każdego niesionego przez postać przedmiotu, który go wskazuje, niezależnie od tego, czy uruchamia go przycisk odpoczynku w arkuszu, czy `[sheet: op="rest"]` Mistrza Gry; arkusz i torba są zapisywane razem, a ponownie wygenerowana odpowiedź odnawia ładunki tylko raz. Arkusz podaje, co wróciło po odpoczynku ("Dawn bell 3/3 charges"), a Mistrz Gry widzi pozostałe ładunki każdego przedmiotu obok niego w ekwipunku ("2 of 3 charges left").
  - `breaksOn`: gdy użycie wydaje ostatni ładunek, Engine rzuca kością d`die`, a przy wyniku nie większym niż `atMost` przedmiot się niszczy i znika z torby, w walce i poza nią. Dziennik walki to odnotowuje ("Dawn bell breaks (a 1 on its die)."), raport przycisku Use również ("Its last charge spent, it breaks."), a dziennik przygody liczy go jako utracony.
- `gate` (Capability API 1.62) to test, który użytkownik musi zdać, zanim przedmiot zadziała, tak jak zwój z zaklęciem powyżej poziomu czytającego wymaga testu. `check` wskazuje, czym się rzuca: `{ "skill": "ward" }`, `{ "ability": "wits" }` albo wartość z arkusza (`{ "value": { "derived": "spell_mod" } }`, dla liczby różnej u każdej postaci, jak modyfikator rzucania zaklęć u czarującego w 5e). `difficulty` wynosi od 1 do 100, zapisane wprost lub odczytane ze statystyki liczbowej przedmiotu (przedmiot, który jej nie daje, nie pojawia się w menu, a jego przycisk Use mówi to wprost). `unless` pomija test, gdy wartość w arkuszu użytkownika wynosi co najmniej `atLeast`. Nieudany test zużywa przedmiot (lub wydaje jego ładunki) na nic. W walce test jest rzucany w chwili wydania przedmiotu, kośćmi samej walki (pulą w walce pulami, gdzie `difficulty` to wymagane sukcesy) i z wartością arkusza odczytaną na początku walki, a liczy się to, co stany i założone przedmioty użytkownika dodają do tego testu, w tym modyfikatory samej umiejętności. Dziennik to odnotowuje ("Ada rolls Ward to use Page of the vigil litany: 2 successes from 2 dice at 7 or more (8, 9), needing 2 successes, a success."). Ma go stronica litanii czuwania z Gravewatch:

  ```json
  "gate": { "check": { "skill": "ward" }, "difficulty": 2, "unless": { "value": { "abilityScore": "nerve" }, "atLeast": 3 } }
  ```

To, co walka zużywa, i pozostawione ładunki są zapisywane w ekwipunku przy każdym kroku, tak jak strzały broni, a dziennik przygody podaje, co zużyto. Dziennik walki podaje każde użycie ("Poultice: 1 of 2 left.", "Dawn bell: 2 of 3 left."). Członek drużyny prowadzony przez Engine leczy tylko kogoś rannego, a nikt nie używa przedmiotu do uderzenia w odchodzącego. Szczegóły przedmiotu i wiersz dla Mistrza Gry podają działanie użycia: "Use (Action): heals 1d4 + 1, range 0 paces, used up" i `use (Act): Steel 7 save negates it, Rattled, 1 of 3 charges`, a szczegóły pokazują pozostałe ładunki. Przedmiot wymyślony przez Mistrza Gry przez `like=` na wzór jednego z twoich jest używany tak jak tamten, łącznie z ładunkami.

### Przedmioty w bitwach Classic i Tactical

Zestaw zasad, który nie rozstrzyga własnych walk, zostawia je bitwom Classic i Tactical trybu Game Mode, w których na początku walki model jest pytany, co robią przedmioty z ekwipunku. Działanie twoich przedmiotów nie jest zgadywane: każdy z `use` robi to, co tam zapisano, na liczbach samego Engine, a Mistrza Gry pyta się tylko o przedmioty, które nie są twoje (i o żadne przy `native: false`).

- `heal` leczy, a `attack` rani o część maksymalnego zdrowia celu wyznaczoną ze średniej `amount`: średnia 7, czyli `1d8+3` podstawowej broni, to nieco ponad jedna piąta, nigdy mniej niż jedna dwudziesta i najwyżej całość. W tej skali pomost walki odczytuje liczby wpisu katalogu. `damageType` jest żywiołem.
- Pierwszy stan w `applies` jest nakładany jako status pod własną nazwą, na swoje `rounds` (2, jeśli ich nie podano), a `buff` lub `debuff` to sam ten status, który podnosi lub obniża obronę jak własne statusy Engine.
- `targets` określa, na kim się go używa. Bez tego leczenie lub wzmocnienie trafia do sojusznika, a atak lub osłabienie – do wroga.
- `consumes: true` odejmuje jedną sztukę ze stosu, a użycie bez tego zostawia przedmiot na miejscu.
- Rzut na trafienie, rzut obronny, obszar, punkty tymczasowe i przywracana pula nie mają miejsca w tych bitwach, tak jak `mechanics` wpisu katalogu: opis przedmiotu w menu nadal je podaje.
- Przedmiot zajmujący miejsca jest używany tylko wtedy, gdy jest założony, a wymagający powiązania – tylko gdy jest powiązany, tak jak w walce według zestawu zasad. Przedmiot zużywany jest zabierany z rzeczy założonych, nigdy z zapasowej sztuki w torbie.
- Przedmiot, którego użycie wydaje `charges`, jest oferowany, dopóki zostało jakieś użycie, a menu Items liczy jego użycia. Każde użycie wydaje je ze stosu przedmiotu, najpierw z własnego stosu gracza, a ostatni wydany ładunek rzuca jego `breaksOn`: zniszczony przedmiot znika z torby, a dziennik przygody liczy go jako utracony.
- Test przedmiotu, który go najpierw wymaga (`gate`), jest rzucany na serwerze, gdy przedmiotu używa członek drużyny, z arkuszem tego członka i tym, co ma na sobie (towarzysz bez własnej karty rzuca na pustym arkuszu, a tylko własna jednostka gracza korzysta zastępczo z karty gracza), tak jak rzuca go przycisk Use, i pomijany, gdy spełnione jest `unless`. Po porażce przedmiot (lub jego ładunki) zostaje wydany i nic nie robi, a dziennik to odnotowuje: "Ada rolls Ward to use Page of the vigil litany: 0 against 2, failed, and it is used up for nothing."
- Przedmiot bez `use` nie jest oferowany w tych bitwach, podobnie jak leczenie bez `amount`.

Engine sam oblicza działanie każdego przedmiotu, więc to, co wysyła ekran, nigdy nie decyduje o działaniu twojego przedmiotu.

### Łupy

Wygrana walka daje łupy w każdej grze w trybie Game Mode, jednorazowo, do toreb drużyny. Bez zestawu zasad pochodzą z własnych tabel trybu Game Mode (więcej i rzadszych w trudniejszej grze); tak samo jest w zestawie, który nie deklaruje tabel łupów i zostawia włączone `native`. Zestaw deklarujący tabele łupów daje zamiast tego własne przedmioty, a zestaw z `native: false` i bez tabel nie daje nic.

Wymienia je `lootTables` w bloku `items` (Capability API 1.63), tak jak dary grobowe z Gravewatch:

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

- `rolls` to liczba losowań z tabeli: liczba od 0 do 20 lub kości (`"1d2"`), domyślnie 1.
- Każde losowanie wybiera jedną pozycję według `weight` (domyślnie 1) względem pozostałych. Pozycja wskazuje jeden z twoich przedmiotów jako `<catalog>/<entry>`, `filter` według `rarity`, `category` i `tag`, który losuje równo spośród wszystkich pasujących przedmiotów, albo (Capability API 1.64) `coins`, jedną z twoich monet według identyfikatora. `count` to liczba sztuk, liczba lub kości, domyślnie 1. Przedmiot lub moneta usunięte przez warstwę nic nie dają; tak samo filtr, który niczego nie znajduje.
- Stworzenie z bestiariusza wskazuje swoją tabelę przez `loot` (zobacz niżej Stworzenia). Wygrana walka według zestawu zasad losuje z tabeli każdego pokonanego stworzenia, a walka, której Engine nie rozstrzyga według twoich zasad, nie ma stworzeń z bestiariusza, więc nic nie daje.
- Łupy trafiają do wspólnego widoku tak jak dodanie przedmiotu, najpierw do torby gracza, a to, czego nikt nie uniesie, zostaje na miejscu. Podsumowanie otrzymywane przez Mistrza Gry podaje, co wypadło i że jest już w torbach; dziennik przygody to odnotowuje, a powiadomienie to pokazuje.
- Mistrz Gry losuje z tabeli w opowieści przez `[loot: table="grave_goods" who="Ada"]` (`who=` dla torby jednej postaci), mierzone od początku tury jak jego znaczniki ekwipunku, więc ponownie wygenerowana odpowiedź daje łupy raz. Każdy wypadający przedmiot otrzymuje odpowiedź jak przy dodaniu, a instrukcje tej komendy wymieniają twoje tabele.

### Pieniądze

Twoje `currencies` to pieniądze drużyny. Moneta jest przedmiotem w torbach, stosem każdej monety jak każdy inny, nazwanym jej `label`:

- Waży jedną jednostkę twojej statystyki udźwigu na każde `perWeight` monet swojej rodziny, liczy się do obciążenia posiadacza i przy otrzymaniu jest rozmieszczana według reguły noszenia. Rodzina bez `perWeight` nic nie waży.
- Gracz dodaje monety z listy **Coins** (monety) w selektorze, a dzieli, przekazuje i łączy je jak każdy stos.
- Wiersz nad stosami ekwipunku pokazuje widoczne monety każdej rodziny i ich wartość w najmniejszej monecie rodziny. Mistrz Gry widzi tę wartość obok każdej torby (`Coin worth 432 pennies` albo `no coins` dla pustej sakiewki, przy czym torba gracza jest wymieniana nawet wtedy, gdy nic nie zawiera), a `cost` przedmiotu obok przedmiotu (`costs 3 shillings`).
- Mistrz Gry płaci przez `[inventory: action="pay" amount="3 shillings" who="Ada"]` i otrzymuje zapłatę przez `action="earn"`. `amount` to liczba i moneta, przez identyfikator lub etykietę, w liczbie pojedynczej lub mnogiej, bez względu na wielkość liter (`1 penny`, `12 pennies`). Płatność pochodzi z jednej torby (gracza, bez `who=`) i tylko z rodziny danej monety: najpierw idą największe pasujące monety, potem rozmieniana jest najmniejsza pozostała moneta pokrywająca resztę należności, a reszta wraca w mniejszych monetach rodziny, od największych. Odpowiedź podaje, co zapłacono i co wróciło: 3 szylingi z sakiewki z jedną koroną to `Paid with crowns ×1; shillings ×2 back.` Płatność, której torba nie pokryje, jest odrzucana (`cannot-afford`), tak samo moneta, której nie masz (`unknown-coin`), i każda płatność w zestawie bez walut (`no-currencies`). Zarobek to dodanie tej monety do torby `who` albo rozdzielenie jej według reguły noszenia. Dziennik przygody wymienia wydatki i zarobki, a powiadomienie je pokazuje. Instrukcje tych komend wymieniają twoje monety i dostaje je tylko zestaw zasad, który jakieś ma.
- Zakup to płatność, a potem dodanie przedmiotu albo, w zestawie z targiem, `buy` (niżej). Nie ma ekranu sklepu.

### Targi

`market` w bloku `items` (Capability API 1.65) określa, co sprzedaje miejscowość i za jaką cenę, zamiast tego, by każdy sklep sprzedawał wszystko po cenie katalogowej. Oto targ z Gravewatch, nieco skrócony:

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

- `prices`: od jednego do ośmiu poziomów, każdy będący mnożnikiem (`times`, powyżej 0) `cost` przedmiotu, a jeden z nich jest `default`. Cena to koszt w najmniejszej monecie rodziny pomnożony przez poziom i zaokrąglony w tej monecie (nigdy poniżej jednej monety dla czegoś, co cokolwiek kosztuje), podawany w monecie, w której zapisano koszt, jeśli ta moneta pokrywa go dokładnie, a w przeciwnym razie w największej monecie pozostawionej przez twoje warstwy, która to potrafi: przy `dear` płaszcz kosztujący 8 szylingów kosztuje 12 szylingów, a mikstura z 5e za 50 gp kosztuje 50 sztuk złota, nigdy 5 sztuk platyny. Targowanie się lub humor sprzedawcy zmieniają poziom, nigdy liczbę.
- `places`: od jednej do dwunastu wielkości miejscowości, od najmniejszej, twoimi słowami.
- `sold`: reguły określające najmniejszą miejscowość sprzedającą to, co wybiera filtr. Filtr wskazuje `rarity`, `category` i `tag`, dowolne z nich, i pasuje do przedmiotu, który ma wszystkie wskazane; decyduje pierwsza reguła pasująca do przedmiotu. Własne `sold` przedmiotu ma pierwszeństwo przed regułami, a przedmiot, którego nic nie wskazuje, jest sprzedawany wszędzie.
- `sellers`: rodzaje sprzedawców, każdy z tym, co sprzedaje (`sells`: filtry, z których każdy może wybrać przedmiot), najmniejszą miejscowością (`place`), w której występuje (bez tego – wszędzie), i opcjonalnie tym, komu sprzedaje wyłącznie (`only`): wartość z arkusza kupującego, uwzględniająca jego bieżący stan i niesione rzeczy, wynosząca co najmniej `atLeast`, przekazywana Mistrzowi Gry jako `label`. Bez `sellers` każdy w danej miejscowości sprzedaje wszystko, co ta miejscowość sprzedaje.
- Przedmiot bez `cost` nie jest na sprzedaż, podobnie jak przedmiot ukryty przez warstwę lub taki, którego monety warstwa usunęła.

**Gdzie jest drużyna.** Mistrz Gry podaje miejscowość sceny i jej wielkość przez `[place: name="Millbrook" size="market town"]`, wielkość przez identyfikator lub etykietę, a pomija wielkość dla miejsca bez targu (droga, dzicz). Engine odpowiada na każdy znacznik na miejscu, odrzucając wielkość, której nie masz, a ostatnia miejscowość, na którą odpowiedział we własnych odpowiedziach widocznych dla gracza, licząc od najnowszego początku rozmowy, obowiązuje, dopóki nie padnie inna. Znacznik miejscowości wpisany przez gracza nigdy się nie liczy. Ponownie wygenerowana odpowiedź odczytuje tylko to, co było przed zastępowaną wypowiedzią.

**Kupowanie.** Mistrz Gry kupuje przez `[inventory: action="buy" item="Hand axe" count="1" level="dear" seller="smith" who="Ada"]`, przy czym `level` (bez niego domyślny) i `seller` (bez niego dowolny tutejszy sprzedawca, który sprzedaje przedmiot) są opcjonalne. Engine wycenia przedmiot w miejscowości obowiązującej na końcu odpowiedzi, pobiera cenę z sakiewki kupującego (gracza, bez `who=`) tak jak płatność, z resztą, i wkłada przedmiot do jego torby albo tylko płaci za usługę. Albo dzieje się jedno i drugie, albo nic. Odpowiedź podaje koszt (`price="12 shillings"`) oraz to, co zapłacono i wydano jako resztę, a powiadomienie to pokazuje. Zakup jest odrzucany i niczego nie zmienia, gdy nie podano miejscowości (`no-place`), gdy nie masz takiego poziomu, przedmiotu lub sprzedawcy (`unknown-level`, `unknown-item`), gdy brak ceny (`not-for-sale`), gdy miejscowość jest za mała na ten przedmiot (`not-here`), gdy nikt tutaj go nie sprzedaje (`no-seller`), gdy kupujący nie spełnia `only` sprzedawcy (`not-to-you`), gdy sakiewka nie wystarcza (`cannot-afford`) albo gdy torba go nie uniesie.

**Co widzi Mistrz Gry.** Gdy scena rozgrywa się w miejscowości z targiem, blok MARKET podaje miejscowość i jej wielkość, poziomy cen oraz każdego tamtejszego sprzedawcę z tuzinem sprzedawanych rzeczy w domyślnej cenie, od najtańszych; wszystko, co sprzedaje, można kupić po nazwie. Jego instrukcje uczą `[place:]` i `buy`, a ma je tylko zestaw zasad z targiem.

#### Dzień targowy

Dzień targowy (Capability API 1.69) sprawia, że towar w miejscowości zmienia się i wyczerpuje, a drużyna może odsprzedawać rzeczy. Oto dzień targowy z Gravewatch, nieco skrócony:

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

- **Dzień.** Dzień w miejscowości trwa, dopóki Mistrz Gry nie wskaże innej miejscowości; nowy dzień w tej samej miejscowości zaczyna, podając ją ponownie z `day="new"` (`[place: name="Millbrook" size="market town" day="new"]`). Wszystko poniżej oblicza się dla każdego dnia z odpowiedzi Mistrza Gry widocznych dla gracza, więc swipe, ponownie wygenerowana odpowiedź ani gałąź nie wymagają niczego zapisanego, a dany dzień zawsze losuje ten sam towar.
- **`limited`** w regule `sold` lub własnym `sold` przedmiotu: ile sztuk miejscowość ma do sprzedania w ciągu dnia, liczba lub kości rzucane na dany dzień. Reguła może podać limit bez miejscowości (sprzedawane wszędzie, po kilka sztuk); miejscowość przedmiotu pochodzi z pierwszej reguły wskazującej miejscowość, a limit z pierwszej wskazującej limit. Gdy zostaną wykupione, zakup jest odrzucany (`sold-out`) aż do nowego dnia.
- **`stock`** u sprzedawcy: jedna z twoich tabel łupów, losowana raz dziennie, określająca, co sprzedawca ma w zapasie oprócz tego, co zwykle sprzedaje. To, co wylosuje (tylko przedmioty, nigdy monety), sprzedaje wyłącznie ten sprzedawca, w wylosowanej liczbie, niezależnie od `sells` i `sold`; `only` sprzedawcy nadal obowiązuje.
- **`buyBack`**: część (powyżej 0, najwyżej 1) domyślnej ceny przedmiotu, którą płaci za niego sprzedawca. Kupuje go tutejszy sprzedawca handlujący takimi rzeczami (bez sprzedawców – ktokolwiek). Bez `buyBack` nikt niczego nie odkupuje.
- **`purse`** przy wielkości miejscowości: najwięcej, ile jej sprzedawcy łącznie wypłacają w ciągu dnia, w jednej z twoich monet. Liczą się do tego sprzedaże opłacane w rodzinie tej monety; bez sakiewki sprzedawcy kupują bez limitu.
- **Sprzedawanie.** Mistrz Gry sprzedaje przez `[inventory: action="sell" item="Silver coffin nail" count="1" seller="smith" who="Ada"]`. Engine wycenia przedmiot, wyjmuje go z torby sprzedającego (gracza, bez `who=`; nigdy powiązanego przeklętego przedmiotu) i wpłaca monety, od największych, albo odmawia i niczego nie zmienia, gdy nie podano miejscowości, nikt tutaj tego nie kupuje (`not-bought`), brak ceny (`not-for-sale`), dzisiejsza sakiewka jest wyczerpana (`purse-spent`), nikt nie ma przedmiotu albo torba nie uniesie monet. Odpowiedź podaje uzyskaną kwotę (`price="4 shillings"`), dziennik przygody odnotowuje sprzedaż, a powiadomienie ją pokazuje.
- Blok MARKET dodaje, co każdy sprzedawca ma dziś w zapasie, ile zostało przedmiotów o ograniczonej liczbie oraz ile płacą tutejsi sprzedawcy i ile zostało z dzisiejszej sakiewki; jego instrukcje uczą `sell` i `day="new"`.

<a id="using-items-outside-a-fight"></a>

### Używanie przedmiotów poza walką

To samo `use` działa poza walką (Capability API 1.60 dla `restore`; reszta nie wymaga niczego nowego). Przycisk **Use** w ekwipunku, na twoim przedmiocie z `use`, oraz `[inventory: action="use" item="Poultice" who="Juno"]` Mistrza Gry wykonują działanie przedmiotu na tym, kto go niesie, kośćmi Engine:

- Leczenie przywraca zdrowie: ilość w puli zdrowia albo jeden usunięty znacznik na torze ran, tak jak leczenie w walce. Punkty `temporary` trafiają do puli zdrowia, `restore` przywraca swoją pulę, a każdy stan z `applies` zostaje nałożony, dopóki coś go nie zdejmie.
- Użycie raniące lub wymierzone w drugą stronę (`kind` `attack` lub `debuff` albo `targets` `enemy`) nikogo nie trafia: poza walką na planszy nie ma nikogo do trafienia. Przedmiot i tak zostaje użyty, a Mistrz Gry dowiaduje się, co robi, aby to opisać.
- Odejmuje jedną sztukę ze stosu lub wydaje ładunki dokładnie tak jak walka, a przedmiot bez pozostałych ładunków albo zajmujący miejsce lub wymagający powiązania i niezałożony jest odrzucany i niczego nie zmienia. Arkusz i torba są zapisywane razem.
- Najpierw rzuca się `gate`, tak jak test: kośćmi zestawu zasad, z wartością z arkusza, karą za rany użytkownika i działaniem jego stanów oraz przedmiotów założonych lub niesionych na ten test. Nieudany zużywa przedmiot na nic, a wiersz to odnotowuje ("Ada uses Page of the vigil litany: Ward check 1 against 2, failed; it is used up for nothing. 1 left.").

To, co się stało, podaje jeden wiersz: "Juno uses Poultice: heals 4 (Grit 6/11). 1 left." Przycisk Use wysyła go Mistrzowi Gry w bloku `[item_used]` po "I use my Poultice.", który czat pokazuje jako plakietkę; na własny znacznik Mistrza Gry odpowiada się tym wierszem, mierzonym od początku tury jak jego inne znaczniki, więc ponownie wygenerowana odpowiedź nigdy nie użyje przedmiotu dwa razy. Przedmiot bez `use` albo w grze bez zestawu zasad jest nadal po prostu opisywany: "I use my rope."

<a id="battles-lending-the-sheet-to-marinaras-combat"></a>

## Bitwy: udostępnienie karty systemowi walki Marinara

Domyślnie bitwa nic nie wie o karcie. Tworzy walczących tak jak dotychczas, a postać może wyjść z walki z niezmienionymi punktami wytrzymałości na karcie.

Opcjonalny blok `battle` zmienia to tylko w jednym kierunku: udostępnia walce liczby z karty i zapisuje jej wynik z powrotem. **Nie sprawia, że walka stosuje twoje zasady.** Matematyka kości nadal należy do Marinara, podobnie jak ustalanie, kto kogo trafia i za ile. Dlatego zdrowie jest przenoszone jako ułamek maksimum, a nie twoja własna liczba: postać z połową zdrowia na karcie zaczyna walkę z połową paska zdrowia utworzonego dla niej przez Marinara. Twoja 9-punktowa pula zdrowia nigdy nie trafia bezpośrednio do walki, w której jeden cios zadaje 12.

```json
"battle": {
  "health": { "pool": "grit" },
  "energy": { "pool": "luck" },
  "skills": [{ "list": "knacks" }]
}
```

- `health`: wymagane. Bieżąca pula stanowiąca punkty wytrzymałości postaci w walce. Musi należeć do pul w `sheet.live.pools`, a nie być listą, której wiersze są pulami.
- `energy`: opcjonalne. Bieżąca pula zużywana przez walkę, stająca się paskiem MP. Musi być inna niż `health`, ponieważ walka nie może zużywać punktów wytrzymałości jako paliwa.
- `slots`: opcjonalne. Bieżące pule zużywane przez walkę po jednym punkcie, każda z `level` od 1 do 9: `[{ "pool": "slots_1", "level": 1 }]`. Każdy poziom i każda pula mogą wystąpić tylko raz.
- `skills`: opcjonalne, do ośmiu. Listy karty, których wiersze stają się umiejętnościami bojowymi postaci. Liczą się tylko wiersze z twoich katalogów i tylko wtedy, gdy ich wpis źródłowy ma blok `mechanics`: wiersz wpisany ręcznie nie opisuje działania liczbowego. `onlyWhen` wskazuje kolumnę logiczną, która musi być włączona w wierszu, np. przygotowane zaklęcie. `alwaysWhen` wskazuje kolumnę i wartość przepuszczającą wiersz mimo tego warunku, np. zaklęcia rzucane bez przygotowania. Jest wyjątkiem od `onlyWhen`, więc bez niego jest odrzucane.

### Co trafia do walki i co wraca na kartę

**Do walki**, dla każdego członka drużyny, którego kartę ma gra: udział puli zdrowia w jej maksimum ustala początkowy stan własnego paska zdrowia Marinara, pula energii staje się MP, każda pula komórek staje się komórkami danego poziomu, a oznaczone wiersze stają się umiejętnościami. Maksymalne punkty wytrzymałości, atak, obrona, szybkość i poziom pozostają własnymi liczbami Marinara. Postać z zerem w puli zdrowia zaczyna walkę powalona, ponieważ tak mówi karta, a postać z wartością dodatnią nigdy nie zaczyna poniżej jednego punktu wytrzymałości, więc zaokrąglenie małego ułamka nie może nikogo znokautować.

**Na kartę**, po zakończeniu walki: końcowy udział zdrowia walczącego jest przeliczany na skalę jego puli zdrowia, a różnica wobec początku walki jest stosowana jako obrażenia lub leczenie. Energia i komórki są licznikami, nie ułamkami, więc zapisuje się je bez przeliczania. Wszystko podlega tym samym zasadom co przyciski karty; zmiana odrzucona przez kartę jest pomijana i zgłaszana, zamiast wymuszana. Walka, która nie zmieniła punktów wytrzymałości walczącego, nie zapisuje żadnej zmiany zdrowia, więc same dwa przeliczenia nie mogą zmienić karty.

**W żadną stronę**: rzuty ataku, rzuty obronne, koncentracja i dodatkowe efekty wyższego kosztu. Są w bloku `mechanics`, by kiedyś odczytał je właściwy system walki; ten pomost ich nie stosuje i zestaw zasad nie powinien twierdzić inaczej.

Porzucona bitwa niczego nie zapisuje. Jeżeli usuniesz wiadomość rozpoczynającą walkę lub walka nie dojdzie do końca, karta pozostaje dokładnie taka jak wcześniej: walka nie miała miejsca.

### Jak wpis staje się umiejętnością

Blok `mechanics` wpisu katalogu jest odczytywany następująco:

- `kind` staje się typem umiejętności. Wpisy `utility` i wszystko oznaczone jako `reaction` są pomijane, ponieważ system walki Marinara nie ma dla nich miejsca.
- `amount` ustala siłę efektu jako mnożnik własnego ataku walczącego, a nie liczbę obrażeń. Większe kości nigdy nie dają słabszego efektu, a mnożnik pozostaje w zakresie już używanym przez generowane umiejętności.
- `range` i `area.size` są dzielone przez `units.distance.perCell` katalogu, by uzyskać pola siatki, i nigdy nie są zaokrąglane do zera. Wybuch staje się swoim promieniem, stożek połową tej wartości, a linia jednym polem. Wszystko z obszarem obejmuje każdego znajdującego się w nim wroga; `friendlyFire` jest respektowane.
- `damageType` staje się żywiołem umiejętności. `targets` nie jest przenoszone: system walki Marinara ustala dozwolone cele leczenia, wzmocnienia lub ataku na podstawie typu umiejętności.
- `cost` puli energii staje się kosztem MP, a kilka kosztów energii jest sumowanych. `cost` dokładnie jednej komórki zużywa jedną komórkę tego poziomu. System walki Marinara pobiera jedną ilość energii albo jedną komórkę, nigdy oba naraz, więc wpis kosztujący dwie komórki, komórki dwóch poziomów lub komórkę i energię nie trafia do walki. Tak samo koszt z dowolnej innej puli, np. punktów wytrzymałości lub zasobu klasowego, ponieważ inaczej Engine udostępniałby go za darmo.
- `buff` lub `debuff` staje się własnym wzmocnieniem lub osłabieniem Marinara. Inne obietnice tekstu wpisu, np. usunięcie stanu z karty, nie są wykonywane w walce. Nie dodawaj `mechanics` do wpisu, którego efekt ma sens wyłącznie poza bitwą.

`coverage.combat` jest osobne i nadal oznacza to samo: ustawiaj je tylko wtedy, gdy bitwy rzeczywiście stosują zasady twojego systemu.

<a id="combat-a-fight-your-own-rules-resolve"></a>

## Walka: starcie rozstrzygane przez twoje własne zasady

Powyższy blok `battle` udostępnia walce liczby z karty, pozostawiając matematykę Marinara. Opcjonalny blok `combat` służy do czegoś innego: określa, jak twoje zasady ROZSTRZYGAJĄ walkę. Parametryzuje należący do Engine rodzaj walki, tak jak `resolution` parametryzuje rodzaj testu; wszystkie nazwy w nim należą do ciebie. Istnieją dwa rodzaje: `attack-vs-defense`, w którym kości są sumowane i porównywane z liczbą, oraz `dice-pool`, w którym walka rzuca własnymi pulami twojego zestawu `dice-pool` i liczy sukcesy (zobacz niżej Walka rzucana pulami).

**Gra, której zestaw zasad deklaruje `combat`, prowadzi walki według twojego bloku.** Liczby drużyny są odczytywane z jej własnych kart, przeciwnicy pochodzą z twojego bestiariusza lub skali zagrożenia, każdą turę rozstrzygają twoje kości, a wszystkie wydatki i straty postaci są natychmiast zapisywane na jej karcie, więc zamknięcie zakładki w trakcie walki niczego nie gubi. Ekran bitwy używa twoich słów: twoje ataki i zdolności w menu, twoje budżety, twoje stany i dziennik z rzeczywistymi obliczeniami. To, czego jeszcze nie robi, opisano w sekcji "Jeszcze niedostępne".

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

To cały blok Ember Roads, według którego walczy gra używająca Ember Roads. Szkic 5e używa tych samych kluczy dla systemu d20:

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

### Wszystkie klucze

- `kind`: `"attack-vs-defense"` (jedna strona rzuca kośćmi przeciw obronie drugiej; trafienie zadaje obrażenia) lub `"dice-pool"` (zobacz Walka rzucana pulami). Każdy z poniższych kluczy znaczy to samo w obu rodzajach i jest odczytywany tak, jak dany rodzaj odczytuje liczby.
- `health`: wymagane. To, co zabiera walka. Albo `{ "pool": "grit" }`, bieżąca pula pomniejszana liczbowo, której bufor tymczasowy, jeśli istnieje, pochłania obrażenia jako pierwszy; albo `{ "track": "harm" }`, tor ran, który jest ZAZNACZANY. Tor wymaga obok `damageKinds` i nie przyznaje punktów tymczasowych.
- `defense`: wymagane, odwołanie do wartości. Pole wypełniane przez gracza albo obliczana przez ciebie wartość pochodna.
- `initiative`: wymagane. Kości rzucane raz na początku i opcjonalne odwołanie do modyfikatora. Remis wygrywa wyższy modyfikator, następnie kolejność tworzenia uczestników walki. `"each": "round"` ponownie rzuca inicjatywą wszystkich na początku każdej nowej rundy, z modyfikatorem w jego ówczesnej wartości (więc modyfikator odczytujący tor ran spowalnia po zranieniu), a runda zaczyna się od tego, kto jest pierwszy w nowej kolejności. Capability API 1.47. Walka `dice-pool` może zamiast tego rzucać inicjatywą jako pulą: `pool` (odwołanie do wartości) to liczba kości, a ich sukcesy plus `plus` dają wartość, bez `dice` ani `modifier` obok. `resource`, które wymaga `pool`, zachowuje tę wartość i pozwala atakom ją przesuwać (zobacz niżej "Inicjatywa przesuwana atakami"). Capability API 1.48.
- `attackRoll`: wymagane przy `attack-vs-defense` i odrzucane przy `dice-pool`, który rzuca własnymi pulami rozstrzygania. Kości, informacja, czy system rzuca dwa razy i zachowuje jeden wynik (`advantage`), działanie skrajnych ścian pojedynczej kości (`naturals.max`: `critical`, `hit` lub `none`; `naturals.min`: `miss` lub `none`) oraz wpływ trafienia krytycznego na obrażenia (`critical`: `double-dice` ponownie rzuca kośćmi obrażeń, `max-dice` dodaje raz ich najwyższe wyniki, `none` oznacza zwykłe trafienie). Szczególne wyniki naturalne wymagają jednej kości, dokładnie jak przy testach. Rzuty obronne w walce używają tych samych kości.
- `economy`: wymagane. `budgets` określa dostępne elementy tury: identyfikator, etykietę, `per` (`turn` uzupełnia na początku własnej tury posiadacza, `round` na początku nowej rundy) i `count`. PIERWSZY zadeklarowany budżet jest główny i zużywa go akcja standardowa. `movement` to opcjonalne odwołanie do wartości: odległość ruchu w jednej turze, we własnej jednostce odległości. Odczytuje ją walka na planszy (zob. Pozycje).
- `attacks`: opcjonalne. Listy karty, których wiersze są bronią. `name` to kolumna tekstowa nadająca wierszowi nazwę, `damage.dice` to kolumna kości, a każde z `toHit.ability`, `toHit.proficiency`, `toHit.bonus`, `damage.ability`, `damage.bonus` i `damage.type` wskazuje kolumnę tej samej listy. Kolumna `ability` to `enum` zawierające identyfikator jednej z twoich cech; inna wartość niczego nie dodaje. Kolumna `proficiency` to `boolean`; gdy jest włączona, dodaje premię z biegłości. Wiersz bez czytelnych kości nie jest atakiem, więc lina na tej samej liście pozostaje liną. `toHit.skill` wskazuje kolumnę `enum` zawierającą identyfikator jednej z twoich umiejętności: wiersz dodaje to, co dodaje test tej umiejętności, z własnym `ability` wiersza zamiast atrybutu umiejętności, jeśli go wskazuje, dokładnie tak jak podmienia go `with=` testu (Capability API 1.47). `strikes` to opcjonalne odwołanie do wartości określające, ile uderzeń kupuje JEDNO zużycie budżetu listy: wybranie wiersza bez uderzeń w zapasie zużywa budżet i daje pozostałe uderzenia, a dopóki zapas istnieje, każdy wiersz deklarujący `strikes` nie kosztuje budżetu. Dzięki temu menu samo pozwala zmienić broń, cel i przejść między nimi. `strikesCappedBy` wskazuje kolumnę logiczną ograniczającą JEJ WŁASNY wiersz do jednego uderzenia, niezależnie od liczby kupowanej przez listę, dla broni strzelającej raz na turę bez względu na liczbę ataków właściciela: powstała dla właściwości Loading z SRD 5.1. Na liście kupującej i tak jedno uderzenie jest bez znaczenia i jest odrzucana. Zapas uderzeń należy do WALCZĄCEGO, nie do jednej listy: postać z dwiema listami broni deklarującymi `strikes` korzysta z tego samego zapasu niezależnie od użytego wiersza. Jest on zerowany na końcu tury, w której go kupiono. Lista bez deklaracji kupuje jedno uderzenie za zużycie budżetu, jak każda walka przed wprowadzeniem tej funkcji.

  ```json
  {
    "list": "attacks",
    "budget": "action",
    "name": "name",
    "strikes": { "field": "attacks_per_action" },
    "damage": { "dice": { "column": "damage" } }
  }
  ```

- `abilities`: opcjonalne. Listy karty, których wiersze oznaczone katalogiem są zdolnościami, filtrowane przez `onlyWhen` i `alwaysWhen` tak samo jak `battle.skills`. Działanie każdej określa `mechanics` jej wpisu; blok wskazuje domyślny `budget`, dodawane `toHit` dla wpisu z rzutem na trafienie oraz `saveDifficulty`, przeciwko któremu wykonuje się rzut obronny wpisu. Wpis wymagający rzutu obronnego, własnego lub kończącego nakładany stan, jest odrzucany, jeśli lista docelowa nie ma `saveDifficulty`: rzut przeciwko niczemu zawsze byłby udany.
- `standard`: opcjonalne, z zamkniętej listy `dash`, `disengage`, `dodge`, `help`, `hide`, `ready`. `dodge` (ataki przeciw unikającemu rzuca się dwa razy i zachowuje gorszy wynik) i `help` (następny atak wspieranego sojusznika rzuca się dwa razy i zachowuje lepszy wynik) są zawsze rozstrzygane. `dash` (ponowny przydział tego samego ruchu) i `disengage` (nikt nie uderza cię za odejście w tej turze) są rozstrzygane na planszy, a poza nią odnotowywane. `hide` i `ready` są akceptowane, ale jeszcze nic nie robią.
- `standardEffects`: opcjonalne, dla części akcji standardowej niewyrażonej jej flagą. Obecnie tylko `dodge` ma taki efekt: `{ "dodge": { "saves": ["dex_save"] } }` określa, które twoje rzuty obronne unikający wykonuje dwa razy z zachowaniem lepszego wyniku przez czas trwania uniku. Wskazuj tylko rzuty zadeklarowane na karcie i tylko gdy lista `standard` zawiera `dodge`. Pominięcie zachowuje dotychczasowe działanie uniku: trudniej trafić i nic więcej.
- `conditions`: opcjonalne. Mapuje TWOJE identyfikatory stanów na ich działanie, dzięki czemu stany karty i walki są jednym zapisem, a zatruta postać pozostaje zatruta po walce. Efekty tworzą zamkniętą listę: `own-attacks-advantage`, `own-attacks-disadvantage`, `attacks-against-advantage`, `attacks-against-disadvantage`, `attacks-against-adjacent-advantage`, `attacks-against-far-disadvantage`, `attacks-from-adjacent-critical`, `cannot-act`, `cannot-react`, `speed-zero`, `half-move-to-stand`, `ends-on-damage`, `own-saves-advantage`, `own-saves-disadvantage`, `own-checks-advantage`, `own-checks-disadvantage`, `resist-all`, `cannot-target-source` i `cannot-approach-source`. `failsSaves` wskazuje rzuty obronne automatycznie nieudane w tym stanie. Sześć efektów wymagających odległości lub ruchu (`attacks-against-adjacent-advantage`, `attacks-against-far-disadvantage`, `attacks-from-adjacent-critical`, `speed-zero`, `half-move-to-stand`, `cannot-approach-source`) odczytuje walka na planszy; poza nią nie działają (zob. Pozycje). `cannot-react` wyklucza posiadacza z okna otwieranego przez ruch, więc nie jest pytany. Obok efektów są cztery dodatkowe klucze:
  - `modifiers`: liczby zmieniane przez stan, dopóki trwa. Zobacz niżej Liczby zmieniane przez stan.
  - `saves`: których rzutów obronnych dotyczą dwa efekty rzutów obronnych i każdy modyfikator rzutów obronnych. Pominięcie oznacza wszystkie; podanie bez żadnego z nich jest odrzucane.
  - `skills`: których umiejętności dotyczą dwa efekty testów i każdy modyfikator testów niewskazujący własnych umiejętności. Pominięcie oznacza wszystkie; podanie bez żadnego z nich jest odrzucane. Testy przeciwstawne w walce rzucają własne testy walki, a nie umiejętności, więc coś zawężonego do umiejętności nigdy do nich nie trafia; odczytuje to test jednej z tych umiejętności poza walką (zobacz niżej Testy poza walką). Capability API 1.53.
  - `whileSourceInSight`: co działa tylko wtedy, gdy osoba nakładająca stan znajduje się w polu widzenia posiadacza. `true` uzależnia cały stan; lista jego własnych efektów uzależnia tylko je, pozostawiając resztę. Tego wymaga strach zabraniający zbliżenia niezależnie od widoczności źródła. Wskazanie efektu nieobecnego w stanie jest odrzucane. Bez planszy nie ma linii widzenia do przerwania, więc wszystko działa w obu przypadkach.
  - `endsWhenSourceDown`: stan znika natychmiast po powaleniu osoby, która go nałożyła.

  `own-saves-advantage` i jego przeciwieństwo rzucają obronę dwa razy z zachowaniem jednego wyniku, dokładnie jak atak, i wzajemnie się znoszą. `own-checks-advantage` i jego przeciwieństwo robią to samo ze stroną posiadacza w teście przeciwstawnym, bo tam walka wykonuje testy atrybutów, oraz z jego testami poza walką. `resist-all` zmniejsza o połowę każdy rodzaj obrażeń, oprócz działania własnej ochrony celu, i tak samo znosi się z podatnością. `cannot-target-source` zabrania posiadaczowi kierowania czegokolwiek na osobę, która nałożyła stan, a `cannot-approach-source` zabrania zbliżenia się do niej bardziej niż z obecnego pola, także na trasie: obejście do równie odległego pola jest nadal oferowane, ale trasa przechodząca bliżej i wychodząca po drugiej stronie już nie.

  ```json
  { "condition": "restrained", "effects": ["own-saves-disadvantage"], "saves": ["dex_save"] }
  ```

  **Liczby zmieniane przez stan.** Każdy wpis w `modifiers` wskazuje, co zmienia (`to`) i o ile:
  - `defense`: liczba, którą musi osiągnąć atak przeciw posiadaczowi. Tylko stała liczba.
  - `attacks`, `saves`, `checks`: własne rzuty ataku, rzuty obronne i testy przeciwstawne posiadacza. Stała liczba albo `dice` rzucane przy każdym rzucie (`"1d4"`); `minus: true` odejmuje kości zamiast je dodawać.
  - `speed`: zasięg ruchu posiadacza we własnej jednostce odległości (`flat`) albo `times` 0.5 lub 2 dla połowy lub podwojenia, stosowane po wszystkich zmianach stałych. Odczytywane tylko na planszy, tak jak `speed-zero`.

  Modyfikator `checks` może wskazywać własne `skills`, a modyfikator `saves` własne `saves`, które zawężają go zamiast zawężenia stanu. Każdy może rzucać dwa razy przez `mode: "advantage"` lub `"disadvantage"`, liczone razem z efektami o tej samej nazwie i znoszące się tak jak one, z liczbą obok albo bez niej; jeden stan może więc jednocześnie utrudniać Sneak i ułatwiać Climb. Te trzy klucze wymagają Capability API 1.53.

  ```json
  { "condition": "muddy", "modifiers": [{ "to": "checks", "skills": ["sneak"], "mode": "disadvantage" }] }
  ```

  Stała liczba ma własny znak, a 0 jest odrzucane. Każdy modyfikator jest pokazywany w dzienniku obok zmienionego rzutu, z nazwą stanu ("12 + 5 + 3 (Blessed) = 20"), a szansa trafienia i szansa wygranej w menu go uwzględniają. Stany kumulują się ze sobą; ten sam stan nałożony dwa razy to nadal jeden stan.

  ```json
  "conditions": [
    { "condition": "blessed", "modifiers": [{ "to": "attacks", "dice": "1d4" }, { "to": "saves", "dice": "1d4" }] },
    { "condition": "shielded", "modifiers": [{ "to": "defense", "flat": 5 }] },
    { "condition": "slowed", "modifiers": [{ "to": "speed", "times": 0.5 }, { "to": "defense", "flat": -2 }] }
  ]
  ```

- `levels`: opcjonalne. Poziomy bieżącego toru liczone jako stany, gdy tor jest dostatecznie wysoko; tak wyraża się stan pogarszający się stopniowo, np. wyczerpanie. Każdy wpis wskazuje zwykły `track` (nie tor ran), poziom `at`, od którego działa, i jego działanie, z tymi samymi `effects`, `modifiers`, `failsSaves`, `saves` i `skills` co stan. Liczy się każdy osiągnięty przez tor poziom, więc sumują się w miarę wzrostu toru. Poziom nie ma źródła i kończy się tylko wtedy, gdy spada odczytywana przez niego wartość, więc `half-move-to-stand`, `ends-on-damage`, `cannot-target-source` i `cannot-approach-source` są w nim odrzucane. Tory ma tylko walczący z kartą. Dziennik nazywa poziom torem i numerem ("- 1 (Heat 3)").

  Zamiast `track` poziom może odczytywać wartość `derived` (Capability API 1.54), obliczaną z bieżącym stanem i przedmiotami postaci, więc nikt nie musi niczego zaznaczać: Ember Roads spowalnia każdego, kto niesie 10 lub więcej jednostek objętości.

  ```json
  { "derived": "bulk_carried", "at": 10, "modifiers": [{ "to": "speed", "flat": -2 }, { "to": "checks", "skills": ["sneak"], "flat": -1 }] }
  ```

  Próg różny dla każdej postaci ("więcej niż pięciokrotność Strength") to wartość pochodna, która go odejmuje: `scale` przez -1 daje część do odjęcia, a `sum` obu wartości jest tym, co odczytuje poziom.

  ```json
  "levels": [
    { "track": "exhaustion", "at": 1, "effects": ["own-checks-disadvantage"] },
    { "track": "exhaustion", "at": 2, "modifiers": [{ "to": "speed", "times": 0.5 }] },
    { "track": "exhaustion", "at": 3, "effects": ["own-attacks-disadvantage", "own-saves-disadvantage"] }
  ]
  ```

  Modyfikatory, poziomy i efekty testów wymagają Capability API 1.45.

- `checks` i `contests`: opcjonalne. Co odczytuje test przeciwstawny i jakie testy przeciwstawne może rozpocząć każdy uczestnik walki: chwytanie, pchnięcie, uwolnienie się. Zobacz niżej Testy przeciwstawne. Capability API 1.43.
- `concentration`: opcjonalne. Bieżące pole `text` zapisujące podtrzymywany efekt, `save` wymuszany obrażeniami, dolna granica trudności `floor` i `fromDamage`, czyli część otrzymanych obrażeń wyznaczająca trudność, gdy jest wyższa. Uruchomienie drugiej zdolności wymagającej koncentracji kończy pierwszą; nieudany rzut obronny kończy ją wraz z podtrzymywanymi stanami.
- `dying`: opcjonalne, `kind: "saves"`. Dwa tory liczące rzuty (wymaganą liczbę określa maksimum każdego toru), `dice`, `succeedAt`, działanie skrajnych wyników (`naturals.max`: `revive-1` lub `success`; `naturals.min`: `one-failure` lub `two-failures`), koszt obrażeń po powaleniu (`damageWhileDown`, `criticalWhileDown`) oraz `condition` powalonej postaci. Bez tego bloku postać z zerem jest po prostu powalona, a leczenie przywraca ją do walki.
- `damageTypes`: opcjonalne. Typy twojego systemu, dopasowywane bez uwzględniania wielkości liter.
- `damageKinds`: wymagane, gdy `health` wskazuje tor ran, a odrzucane, gdy wskazuje pulę, ponieważ tylko znacznik przechowuje rodzaj, a pula punktów nie ma na niego miejsca (zob. Tory ran powyżej). Określa, który z `kinds` toru zaznacza cios i ile pól wypełnia. `default` jest rodzajem dla wszystkiego bez mapowania, w tym ciosu bez typu, a `byType` mapuje twoje `damageTypes` na rodzaje; klucze, jak same typy, są dopasowywane bez wielkości liter, więc `"Fire"` i `"fire"` są jednym kluczem i podanie obu jest odrzucane. `marks` nie ma wartości domyślnej, ponieważ dwie odpowiedzi są przeciwne: `"per-point"`, gdy rzut obrażeń liczy poziomy zdrowia, więc cios za trzy zaznacza trzy pola i warto złagodzić go o jeden; oraz `"per-blow"`, gdy cios albo trafia, albo nie, więc zaznacza jedno pole niezależnie od siły. Cios z kilkoma składnikami obrażeń nadal zaznacza jedno pole, używając najcięższego rodzaju, który trafił. Wskaż, jak działa twój system. `{ "default": "bashing", "byType": { "fire": "aggravated" }, "marks": "per-point" }`.
- `pool`: wymagane przy `dice-pool`, odrzucane przy `attack-vs-defense`. Zobacz Walka rzucana pulami.
- `spendLimits`: opcjonalne. Ile z bieżącej puli jeden walczący może wydać na `turn` lub na `round` w walce dowolnego rodzaju: `[{ "pool": "blood", "max": { "derived": "blood_per_turn" }, "per": "turn" }]`. `max` jest odczytywane raz, na początku walki. Koszt przekraczający pozostałą część limitu jest niemożliwy do opłacenia, więc znika z menu, niezależnie od zawartości puli. Limit na turę odnawia się na początku własnej tury posiadacza, a limit na rundę na początku rundy. Przeciwnik zapisany zwykłymi liczbami za nic nie płaci z karty, więc limit nigdy go nie wiąże; przeciwnik zapisany jako karta płaci z własnych pul i podlega limitowi jak każdy inny. Capability API 1.47.
- `threat`: opcjonalne, wymagane przez bestiariusz. `tiers` to skala wyboru przeciwnika: identyfikator, etykieta, przedział `health`, `defense`, `toHit`, przedział `damagePerRound` i `saveDifficulty`. Każde dostarczane stworzenie wskazuje jeden z tych poziomów, a nienapisany wcześniej przeciwnik zostaje dopasowany do poziomu żądanego przez Mistrza Gry, więc nikt nie wypada poza skalę. Przedział `damagePerRound` oznacza działanie stworzenia na JEDEN cel w rundzie, łącznie z całą sekwencją.

### Walka rzucana pulami: `dice-pool`

Zestaw `dice-pool` liczy sukcesy zamiast sumować kości, a jego walki też mogą tak działać. `"kind": "dice-pool"` wymaga, aby `resolution.kind` miało wartość `"dice-pool"`, i odczytuje kartę tak, jak już robią to twoje testy puli: **każda liczba dodawana przez rzut to liczba kości, a każda liczba, którą rzut musi osiągnąć, to liczba sukcesów.** Nic więc nie zmienia nazwy:

- Liczba trafienia (wiersza ataku, `toHit` listy zdolności, akcji stworzenia) to pula rzucana przez atak. Kość, próg, który musi osiągnąć kość, oraz wyniki, przy których kości liczą się podwójnie, eksplodują, anulują sukcesy i dają krytyczną porażkę, pochodzą z twojego `resolution`. Tor ran wskazany w `resolution.penaltyFrom` odejmuje swoją karę od puli, tak jak w teście.
- `defense` to liczba sukcesów wymagana przez atak, nigdy mniejsza niż jeden. Dodaje się do niej modyfikator `defense` ze stanu oraz `cover.bonus`.
- Każdy sukces ponad wymagane dodaje jedną kość obrażeń. Krytyczna porażka chybia, niezależnie od zliczonych sukcesów. Nie ma trafień krytycznych: mocne trafienie jest warte swoich dodatkowych kości.
- Liczba rzutu obronnego to jego pula, a trudność to wymagane sukcesy. Test przeciwstawny rzuca test każdej strony jako pulę i wygrywa strona z większą liczbą sukcesów.
- Modyfikator ataków, rzutów obronnych lub testów ze stanu to dodawane lub odejmowane kości, więc jest liczbą `flat`: modyfikator rzucany (`dice`) jest odrzucany.
- Obrażenia to kości twojego rodzaju: `dice` ilości (stworzenia, wpisu) określa, ile ich jest, i musi używać twojej kości, a część `flat` to automatyczne sukcesy, nigdy nierzucane. Z kolumny kości obrażeń wiersza ataku odczytuje się liczbę kości, `damage.ability` dodaje wartość tego atrybutu jako kości, a `damage.bonus` dodaje automatyczne sukcesy. `perCostStep` i `scales` dodają kości, jak zawsze. Leczenie i punkty tymczasowe to ilości sumowane jak wcześniej.
- Inicjatywa jest sumą (`initiative.dice` i `modifier`) albo pulą, jeśli o nią poprosisz (`initiative.pool`, niżej), a reguła umierania zachowuje własne kości.

To, co walka pulami rzuca poza tym, określa jej blok `pool`:

```json
"pool": {
  "advantage": true,
  "damageTarget": 6,
  "soak": { "roll": true, "byKind": { "knock": { "abilityMod": "sinew" } } }
}
```

- `advantage`: czy rzut przechylony przez ułatwienie lub utrudnienie jest wykonywany dwa razy z zachowaniem tego z większą liczbą sukcesów (krytyczna porażka liczy się jako mniej niż każdy rzut bez krytycznej porażki). Bez tego walka rzuca raz, niezależnie od stanu.
- `damageTarget`: próg pojedynczej kości dla kości obrażeń i pochłaniania. Po pominięciu – domyślny próg twojego rozstrzygania. Te kości liczą raz każdy wynik równy progowi lub wyższy i nic poza tym: nic się na nich nie podwaja, nie eksploduje, nie anuluje ani nie daje krytycznej porażki.
- `soak`: co cel odejmuje od obrażeń trafienia, według rodzaju obrażeń. `all` to odwołanie do wartości dla dowolnego rodzaju, a `byKind` – po jednym dla każdego rodzaju twojego toru zdrowia, z pierwszeństwem przed `all` dla swojego rodzaju; rodzaj niewskazany przez żadne z nich nie jest pochłaniany. Przy `"roll": true` cel rzuca tyloma kośćmi przeciw progowi obrażeń, a każdy sukces odejmuje jeden. Przy `"roll": false` liczba jest odejmowana od kości obrażeń przed rzutem, więc nigdy nie dotyka automatycznych sukcesów, które pochłanianie rzucane odejmuje jak wszystkie inne. Nic nie spada poniżej zera i nic nie jest rzucane, by pochłonąć cios, który nic nie zliczył. Stworzenie podaje własne liczby jako `soak: { "all": 1, "byKind": { "knock": 3 } }`, co wymaga `soak` w bloku, określającego sposób pochłaniania. Odporność, podatność, niewrażliwość i `resist-all` działają po pochłonięciu, na to, co zostało.
- `hardness`: odwołanie do wartości twardości walczącego, gdy inicjatywa jest liczbą przesuwaną atakami. Zobacz [Pancerz i efekty założonych przedmiotów w walce](#armor-and-worn-effects-in-a-fight).

Obrażenia każdego celu są rzucane osobno dla niego, bo to, co trafienie uzyskało ponad wymagane sukcesy, i to, co cel pochłania, należą do niego. Drugi składnik ciosu i efekt dodatkowy to osobne pule obrażeń własnego rodzaju, bez dodatkowych kości z trafienia. Menu prognozuje pulę dokładnie: szansę osiągnięcia wymaganych sukcesów i wartość kości obrażeń po tym, co pochłonie pierwszy cel.

Tak zapisano Gravewatch: jego strażnicy rzucają cechą i umiejętnością zawodową, pochłaniają stłuczenia przez Sinew, a rozdarć wcale, mogą wydać jeden punkt Resolve na turę i rzucają inicjatywą ponownie w każdej rundzie.

**Deklarowanie w odwrotnej kolejności nie jest zaimplementowane.** W niektórych systemach wszyscy deklarują akcje, zanim ktokolwiek zacznie działać, od najwolniejszego. W walce, w której każdy walczący wybiera jedną akcję, gdy nadchodzi jego tura, wcześniejsza deklaracja nie zmienia niczego, co odczytuje jakakolwiek reguła, więc klucz nie miałby czego określać.

### Inicjatywa przesuwana atakami

Niektóre systemy pul utrzymują inicjatywę jako liczbę przez całą walkę i pozwalają atakom ją przesuwać: jeden sposób ataku odbiera ją celowi, inny wydaje inicjatywę atakującego jako obrażenia, a kto spadnie do ustalonej granicy, ulega załamaniu. Walka `dice-pool` może to wyrazić (Capability API 1.48):

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

- **Otwarcie.** Na początku walki każdy rzuca `pool` kośćmi, a sukcesy plus `plus` dają jego wartość, więc `resource` wymaga `pool`: zsumowane kości wyznaczają kolejność, a nie liczbę kości. `initiativeModifier` stworzenia to jego pula, bo każda liczba dodawana w walce pulami to kości. Wartość jest zachowywana: nigdy nie rzuca się nią ponownie (`each` obok `resource` jest odrzucane), a na początku każdej rundy kolejność jest sortowana według bieżących wartości.
- **Style.** Każdy atak (wiersz ataku, zdolność lub wpis katalogu z rzutem na trafienie zadający obrażenia, akcja stworzenia, która to robi, oraz akcja złożona z innych akcji) jest oferowany raz dla każdego stylu, a gracz wybiera styl przed celem. Najwyżej cztery, każdy albo zabiera, albo wydaje, i co najmniej jeden zabiera. Wybór niewskazujący stylu jest wykonywany w pierwszym; wybór stylu, w którym atak nie jest oferowany, jest odrzucany. Atak zachowuje styl, w którym go wykonano, do swojego końca, nawet gdy odpowiedź zmieni wartość wykonawcy, zanim cios dosięgnie celu.
- **Styl zabierający.** Przy trafieniu obrażenia rzuca się jak zwykle (dodatkowe kości z trafienia, kości broni, pochłanianie według rodzaju, automatyczne sukcesy), a zliczony wynik jest odejmowany od wartości celu zamiast od jego zdrowia. Atakujący zyskuje całość plus `gain`.
- **Styl wydający.** Oferowany tylko wtedy, gdy wartość atakującego jest powyżej granicy załamania (0, gdy nie podasz `crash`), i nigdy dla akcji złożonej z innych akcji, bo wartość wydaje się na jeden cios. Przy trafieniu obrażenia to wartość atakującego z chwili ataku, rzucana jako kości przeciw progowi obrażeń, i nic poza tym: bez kości broni, bez dodatkowych kości z trafienia, bez automatycznych sukcesów, bez pochłaniania. Zaznacza zdrowie własnym rodzajem obrażeń ataku. Po zakończeniu ataku wartość, która gdziekolwiek trafiła, wraca do `base`, a wartość, która nigdzie nie trafiła, traci tyle, ile określa `onMiss` dla wartości, z którą wykonano atak (tabela progów: pary `[at least, lose]`, rosnąco; poniżej pierwszego progu nic).
- **Załamanie.** Wartość, która spadnie do `crash.at` lub niżej, oznacza załamanie: na walczącego zostaje nałożony `condition` (jeden z twoich stanów, opcjonalny), a ten, kto go tam zepchnął, zyskuje `bonus`. Chybienie, które załamuje samego wykonawcę, nikomu nie daje premii. Załamany walczący nie może wydawać. Załamanie ustępuje, razem ze stanem, gdy jego wartość znów wzrośnie powyżej granicy, po `recoverAfter` jego własnych tur (jego wartość wraca do `base` na początku tej tury) oraz gdy walka się skończy, jakkolwiek się skończy. Ktoś, czyj rzut otwierający jest na granicy lub poniżej, zaczyna walkę załamany. `base` musi być powyżej `crash.at`, inaczej powrót do tej wartości ponownie by go załamał.

Menu pokazuje osobną prognozę każdego stylu: szansę trafienia oraz to, co zabrałby styl zabierający, lub obrażenia, które zadałby styl wydający. Panel statusu pokazuje wartości wszystkich, a dziennik podaje każdą zmianę i jej przyczynę: "Ada gains 5 initiative for crashing Rats, and is on 14." Mechanizm wyboru Engine ocenia cios zabierający z wyprzedzeniem o jedną turę: zabranie, a potem wydanie, wobec wydania teraz i ponownie od wartości bazowej.

<a id="what-a-fight-reads-from-mechanics"></a>

### Co walka odczytuje z `mechanics`

`kind` określa, czy `amount` to obrażenia czy leczenie; wszystko oznaczone `reaction` jest pomijane w menu, podobnie jak wpis `utility`, chyba że zmienia możliwości samej tury (zob. niżej). `attackRoll` wymusza rzut przeciw obronie celu z `toHit` listy; `autoHit` całkowicie go pomija. `save` wykonuje własny rzut obronny celu przeciw `saveDifficulty` listy, a `onSuccess` określa, czy sukces pozostawia połowę, czy nic. `targetCount` określa dozwoloną liczbę celów. Zdolność bez rzutu ataku (obszar, przeciw któremu wszyscy się bronią, lub coś trafiającego automatycznie) rzuca kośćmi RAZ dla wszystkich, a zdolność rzucająca na trafienie każdego celu rzuca obrażenia ponownie przy każdym trafieniu. `applies` nakłada stany na objęte cele, każdy z `duration`: `instant` (bez własnego zegara: trwa, dopóki coś go nie zdejmie), `until-save` (wymaga obok `saveEnds`) lub `{ "rounds": n }`, oraz opcjonalnym `saveEnds` wskazującym rzut obronny i powtarzanie przy `turn-end` lub `turn-start`. Rundy odliczają własne tury posiadacza w chwili zakończenia każdej z nich; `{ "rounds": 1, "at": "turn-start" }` odlicza je na początku każdej, co wyraża "do początku twojej następnej tury". `endsAfter` zdejmuje stan po pierwszym `own-attack` (następnym rzucie ataku posiadacza), `attacked` (następnym rzucie ataku wykonanym przeciw posiadaczowi) lub `own-save` (następnym rzucie obronnym posiadacza), niezależnie od jego zegara, więc `{ "duration": { "rounds": 1 }, "endsAfter": "own-attack" }` oznacza "przy jego następnym ataku przed końcem jego następnej tury". Stan trwający jeden atak trwa przez cały atak, łącznie z obrażeniami, a nowy stan nałożony przez ten sam cios zostaje. Oba wymagają Capability API 1.45. `temporary` przyznaje tymczasowe punkty puli zdrowia; nie kumulują się – pozostaje większy bufor. `scales` zwiększa wielkość o dodatkowe KOŚCI z tabeli dla odczytanej wartości. `cost` jest płacony przez własne polecenie `use` karty, a `budget` zastępuje domyślną część ekonomii akcji.

`plus` to do trzech DODATKOWYCH wielkości tego samego ciosu, obok `amount`, każda z osobnym rzutem i typem ("and 2d6 fire"). Składnik ma postać `{ "dice": "2d6", "flat": 1, "type": "fire" }` i może mieć własny `save`, `{ "save": "con_save", "difficulty": 13, "onSuccess": "none" | "half" }`, który CEL wykonuje niezależnie od pozostałych wymagań akcji: `none` usuwa cały ten składnik przy sukcesie, `half` pozostawia połowę, a reszta ciosu pozostaje niezmieniona. Bez `difficulty` używa liczby własnego rzutu obronnego akcji, a następnie `saveDifficulty` listy. Trafienie krytyczne podwaja kości każdego składnika według tej samej zasady co pierwszą wielkość; składnik bez `type` ma własny rodzaj obrażeń ciosu, a cały cios nadal oznacza JEDEN test koncentracji z sumą obrażeń i jeden test powalenia. Składnik wymaga bazowego `amount`; `heal` nie może mieć takich składników.

```json
{
  "kind": "attack",
  "attackRoll": true,
  "amount": { "dice": "1d8" },
  "damageType": "piercing",
  "plus": [{ "dice": "2d6", "type": "fire" }]
}
```

Trzy klucze określają wpływ wpisu na ekonomię samej tury; wpis `utility` deklarujący którykolwiek z nich jest oferowany zamiast pomijany:

- `free`: nie zużywa żadnego budżetu. Nadal płaci wskazany `cost` i nie może jednocześnie wskazywać `budget`.
- `gives`: `[{ "budget": "action", "count": 1 }]`, do czterech. W chwili użycia zwiększa te budżety, ograniczając ich wynik do pojemności tury powiększonej o przyznany dodatek, więc niczego nie można zachować na późniejszą turę.
- `standard`: `{ "actions": ["dash", "disengage", "hide"], "budget": "bonus" }`. Posiadacz może wykonywać te akcje standardowe za TEN budżet. Są oferowane obok zwykłych jako `standard:<id>@<budget>`, a sam wpis nie pojawia się w menu, gdy jest wyłącznie tym uprawnieniem, ponieważ uprawnienia się nie wykonuje.

Wpis nowego rodzaju `kind: "rider"` jest PASYWNY: nikt go nie wykonuje, nigdy nie ma go w menu i automatycznie dodaje kolejny składnik obrażeń do pierwszego kwalifikującego się trafienia w danym okresie. Zawiera `rider` i nic innego, co można wykonywać:

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

`sources` wskazuje listy ataków, z których pochodzi, a `requires` jedną kolumnę ich wierszy o wartości prawdziwej, więc dodatkowy efekt działający tylko z niektórymi broniami określa je bez znajomości pojęcia broni przez Engine; pominięcie obu oznacza każde trafienie posiadacza. Broń trzymana z torby nie jest wierszem, więc `weapons` (Capability API 1.70) wskazuje także tagi przedmiotów, przy których efekt działa: `"weapons": { "tags": ["finesse",
"ranged"] }` działa przy trafieniu trzymaną bronią z którymkolwiek z tych tagów, łącznie z uderzeniem drugą ręką. Wskazanie tylko `weapons` nie działa przy żadnym wierszu, a wskazanie tylko wierszy nie działa przy żadnej trzymanej broni. `when` oznacza DOWOLNY z warunków: `advantage` to ostateczna przewaga rzutu ataku, a `ally-adjacent` to stojący sojusznik atakującego zdolny do działania, oddalony od celu o najwyżej jedno pole na planszy lub znajdujący się gdziekolwiek bez planszy. `oncePer` to `turn` (odnawia się na początku każdej tury, więc uderzenie podczas cudzego działania również może go otrzymać) albo `round`. `amount` rośnie zgodnie z własnym `scales` wpisu, a `type` określa rodzaj obrażeń, domyślnie taki jak ciosu.

<a id="creatures-a-bestiary-a-fight-reads"></a>

### Stworzenia: bestiariusz odczytywany przez walkę

Katalog deklarujący `"holds": "creatures"` zawiera przeciwników zamiast wierszy karty. Nie zasila żadnej listy, selektor edytora karty nigdy go nie oferuje, a wszystkie liczby są zapisane w kluczach już zadeklarowanych przez blok `combat`. Wymaga bloku `combat` i skali `threat`, ponieważ stworzenie należy do jednego z twoich poziomów.

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

Poniższe liczby to prosty sposób zapisania stworzenia. Stworzenie zapisane we własnych pojęciach zestawu zasad, jako `sheet`, pobiera zamiast tego `health`, `defense`, `initiativeModifier`, `speed`, `abilities`, `saves`, `checks`, `soak` i `hardness` z tej karty (zob. "Stworzenie zapisane we własnych pojęciach zestawu zasad" poniżej).

- `health`: liczba lub `{ "dice": "3d6", "flat": 2 }` rzucane raz przy tworzeniu walki. Prognoza odczytuje średnią, więc menu nigdy nie obiecuje wyniku kości, której nikt nie rzucił.
- `defense`, `initiativeModifier`, `speed`: wartość, przeciw której wykonuje się atak, dodatek do inicjatywy (albo, gdy inicjatywą rzuca się jako pulą, liczba rzucanych kości) i odległość ruchu w turze we własnej jednostce odległości.
- `abilities` i `saves`: klucze odpowiadają identyfikatorom cech i rzutów obronnych zadeklarowanych na karcie. Niewskazany rzut obronny ma wartość zero.
- `checks`: co stworzenie dodaje w teście przeciwstawnym, z kluczami będącymi identyfikatorami `combat.checks`. Niewskazany test ma wartość zero. Capability API 1.43.
- `soak`: tylko w walce `dice-pool` – co stworzenie pochłania: `all` dla dowolnych obrażeń, `byKind` dla rodzaju twojego toru zdrowia. Capability API 1.47.
- `hardness`: gdy inicjatywa jest liczbą przesuwaną atakami, cios wydający, którego kości są poniżej tej wartości, nic nie robi. Capability API 1.56.
- `resist`, `vulnerable`, `immune`: typy obrażeń, dopasowywane bez wielkości liter i sprawdzane względem `combat.damageTypes`, jeśli jakieś deklarujesz. Wpis `resist` lub `immune` może też określać, co się przez niego przebija, `{ "type": "tearing", "except": ["silver"] }`: cios bronią-przedmiotem z jednym z tych tagów przedmiotów jest przyjmowany bez zmian (Capability API 1.55, zobacz [Broń w walce](#weapons-in-a-fight)). `conditionImmunities` wskazuje twoje własne stany.
- `tier`: szczebel `combat.threat`, do którego należy stworzenie.
- `loot` (Capability API 1.63): identyfikator tabeli łupów losowanej za nie po wygranej walce (zobacz wyżej Łupy).
- `traits`: krótkie pary nazwy i tekstu pokazywane Mistrzowi Gry. Nigdy nie są rozstrzygane, więc wszystko zawierające liczby należy do akcji.
- `signaturePoints`: punkty zwracane na początku własnej tury, wydawane na akcje `signature`.
- `riders`: do czterech, to samo co `rider` wpisu katalogu, zapisane w bloku. Każdy ma postać `{ "id": "pack", "name": "Pack", "on": "hit", "oncePer": "turn" | "round", "amount": { "dice": "1d6" } }`, z opcjonalnym `type` i opcjonalnym `actions` wskazującym własne akcje bloku, przy których działa. Dodatkowy efekt bloku nie odczytuje list karty, więc nie ma dwóch kluczy: `sources` i `requires`. Stworzenie zapisane jako karta otrzymuje efekty dodatkowe ze swoich list, dokładnie jak postać.
- `actions`: do dwunastu, każda z własnym `id`. Akcja zawiera to, co ręcznie napisany blok statystyk (`toHit`, `autoHit`, `damage`, `save`, `applies`, `targetCount`, `reach`, `range`, `area`), oraz cztery rzeczy dostępne tylko stworzeniom. `reach` określa zasięg uderzenia, `range` zasięg rzutu lub strzału, a `area` kształt obszaru działania, wszystko we własnej jednostce odległości; `range` może być zwykłą liczbą albo `{ "normal": 30, "long": 120 }`, gdy dalszy zasięg wiąże się z karą, a `area` ma postać `{ "shape": "burst" | "cone" | "line", "size": n, "friendlyFire": false }` (zob. Pozycje):
  - `uses`: `{ "per": "encounter" | "day", "count": n }`. Po wyczerpaniu użyć akcja znika z menu.
  - `recharge`: `{ "dice": { "count": 1, "sides": 6 }, "from": 5 }`. Na początku walki jest dostępna, użycie ją zużywa, a na początku własnej tury stworzenia wykonuje rzut: `from` lub więcej przywraca ją. Dziennik zapisuje kości w obu przypadkach.
  - `sequence`: inne akcje tego samego bloku, po kolei, każda z własnym celem. **Tak zapisuje się stworzenie uderzające dwa razy w jednej akcji.** Jeden budżet opłaca całą sekwencję. Sekwencja nie ma własnego efektu i nigdy nie może wskazywać kolejnej sekwencji.
  - `signature`: `{ "cost": n }`, opłacane własnymi punktami stworzenia zamiast budżetu i dostępne tylko podczas działania kogoś innego: walka oferuje je w oknie między turami (zob. Okna).
  - `reaction`: ten sam obiekt co `mechanics.reaction` wpisu katalogu (zob. Okna): akcji nie ma w menu żadnej tury, a zamiast tego jest oferowana w chwili, którą wskazuje. Reakcja nie jest jednocześnie akcją specjalną i żadna sekwencja nie może jej zawierać.
  - `self: true`: działa na samo stworzenie, a nie na kogoś innego, więc nie przyjmuje `targetCount` ani `area`. Oba wymagają Capability API 1.46. Parry jest jednym i drugim, ze stanem `parrying`, który dodaje 2 do obrony:

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
- Rzut obronny wymaga trudności na samej akcji: `save.difficulty` dla rzutu wymuszanego przez akcję lub `saveDifficulty` dla stanu kończonego rzutem obronnym, gdy akcja nie ma własnego rzutu. Akcja bloku jest zapisywana zwykłymi liczbami nawet u stworzenia z kartą, więc liczba należy do akcji. Własny rzut obronny składnika może pominąć `difficulty` i skorzystać z tej samej liczby.
- `damage.plus` to ta sama lista składników co `plus` wpisu katalogu, odczytywana identycznie: `"damage": { "dice": "1d6", "flat": 2, "type": "piercing", "plus": [{ "dice": "1d4", "type": "fire" }] }` to ugryzienie z osobną ilością obrażeń od żaru, osobno redukowaną przez odporność i osobno podwajaną.

Bestiariusz szkicu 5e zawiera pięć ręcznie napisanych stworzeń w `docs/development/ruleset-5e-2014.example.json`, obejmujących sekwencję, odnowienie, rzut obronny ze stanem, odporności i niewrażliwości, ograniczone użycia, punkty akcji specjalnych i jedno stworzenie zapisane jako karta.

#### Stworzenie zapisane we własnych pojęciach zestawu zasad

Nie musisz zapisywać stworzenia zwykłymi liczbami. Nadaj mu zamiast tego `sheet` o dokładnie takim kształcie jak karta postaci, a walka zbuduje je jak członka drużyny: zdrowie, obrona, rzuty obronne, inicjatywa, szybkość oraz każdy atak i zdolność z list wynikają z twoich własnych formuł karty. Tak zestaw zasad określa, że przeciwnicy mają te same cechy, umiejętności i listy co postacie, niezależnie od tego, czym są. Toll Warden z Ember Roads:

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

- **Każda część jest opcjonalna**: `abilities`, `skills`, `saves`, `bonuses`, `fields` i `lists`, z kluczami będącymi identyfikatorami zadeklarowanymi na karcie. Wszystko pominięte przyjmuje własną wartość domyślną karty, dokładnie jak na pustej karcie postaci. Grit strażnika wynosi 9, ponieważ `grit_max` dodaje 4, jego Toughness i Brawn; Guard wynosi 7 z podobnego powodu.
- **Każda liczba ma jedno źródło.** Stworzenie z kartą nie podaje jednocześnie `health`, `defense`, `initiativeModifier`, `speed`, `abilities`, `saves` ani `checks`; jeśli to zrobi, Engine odrzuca plik. Może nie mieć żadnych własnych `actions`, ponieważ jego działania wynikają z list. Stworzenie bez karty nadal podaje pierwsze trzy wartości i co najmniej jedną akcję.
- **Jest sprawdzane jak autorskie dane, którymi jest.** Każdy identyfikator musi być zadeklarowany na karcie, umiejętność lub rzut obronny musi należeć do oferowanych dla niego poziomów biegłości, pole, wynik, premia lub kolumna muszą zawierać deklarowany typ wartości (liczbę całkowitą w zakresie, jedną z dopuszczalnych wartości itd.), a lista nie może przekraczać limitu wierszy. Nie ma części `live`, ponieważ wydatki stworzenia przechowuje walka.
- **Wiersz może pochodzić z katalogu.** `_catalog: "<catalog>/<entry>"` wskazuje wpis, z którego wybrano wiersz, tak jak na karcie postaci; tam walka odczytuje koszt i działanie wiersza. Katalog musi zasilać tę listę. Jeśli jest zapisany wewnątrz definicji, musi zawierać wpis; jeśli ma osobny plik, wiersz wskazujący nieobecny w nim wpis po prostu niczego stworzeniu nie daje. Katalogi wskazywane przez karty bestiariusza są ładowane do walki razem z bestiariuszem.
- **Informacje wpisu obok karty nadal się liczą**: `tier`, `traits`, `actions`, `signaturePoints`, `riders`, `resist`, `vulnerable`, `immune` i `conditionImmunities`.
- **Płaci z własnych pul.** Zaczynają pełne, wydaje je na działania z list i otrzymuje droższe warianty płacenia (zaklęcie z wyższej komórki) dokładnie jak członek drużyny, niezależnie od tego, czy decyduje Engine czy Mistrz Gry. Hold the Line strażnika kosztuje go Luck.
- **Przy torze ran jego zdrowiem jest tor.** Cios zaznacza własny tor stworzenia według `damageKinds`, po uwzględnieniu jego `resist`, `vulnerable` i `immune`, więc stworzenie niewrażliwe na dany rodzaj obrażeń nie otrzymuje od niego znacznika.
- **Nadal jest przeciwnikiem.** Przy zerze wypada z walki zamiast umierać, nigdy nie wykonuje rzutów przeciw śmierci, ekran pokazuje to samo co zawsze dla przeciwnika i nic z jego karty, a jego wydatki nie są nigdzie zapisywane z powrotem, nawet gdy postać ma to samo imię.
- **Karta nie dająca żadnego zdrowia** jest pomijana w walce; dziennik początkowy podaje powód, zamiast wpuszczać kogoś, kogo nie można zranić.
- Warstwa usuwająca wartość z jednego z pól wyliczeniowych nigdy nie odbiera stworzenia, które jej używa: gdy warstwa jest aktywna, pole przyjmuje dla stworzenia wartość domyślną, dokładnie jak u postaci, i stworzenie nie jest z tego powodu odrzucane.
- Mistrz Gry też może takie wymyślić; zostaje ograniczone do swojego poziomu (zob. Przeciwnicy, których nikt nie napisał).
- Pakiet dostarczający takie stworzenie deklaruje Capability API 1.34.

Toll Sergeant ze szkicu 5e robi to samo na karcie d20: jego Klasa Pancerza, punkty wytrzymałości, rzuty obronne i dwa zamachy na akcję wynikają z własnych pól i listy ataków.

#### Przeciwnicy, których nikt nie napisał

Gdy Mistrz Gry wymyśla przeciwnika, Engine dopasowuje propozycję do skali `threat` przed jakimkolwiek rzutem: zdrowie do przedziału poziomu, obronę, premię trafienia i trudności rzutów obronnych do najwyżej dwóch powyżej wartości poziomu, a obrażenia zmniejsza, aż najlepsza runda stworzenia (najsilniejsza sekwencja lub pojedyncza akcja, mierzona wobec jednego celu) mieści się w `damagePerRound` poziomu. Najpierw zmniejsza liczbę kości, potem stały składnik, następnie uderzenie z sekwencji, a dopiero potem rozmiar kości; nigdy nie zmniejsza niczego do zera. Nieistniejące w zestawie zasad nazwy są usuwane: nieznane typy obrażeń, stany i rzuty obronne oraz wszystko poza pierwszymi sześcioma akcjami. Niezadeklarowany poziom wraca do najniższego szczebla skali. Każda zmiana wraca jako zwykłe zdanie, by dziennik mógł opisać działanie.

Wymyślone stworzenie też można zapisać jako `sheet`, tak samo jak stworzenie bestiariusza; tak wymyślony mag otrzymuje komórki i zaklęcia. Mistrz Gry widzi identyfikatory karty i dozwoloną zawartość każdego, listy odczytywane przez walkę oraz nazwy oferowane dla nich przez katalogi, więc wskazuje zaklęcie nazwą zamiast opisem: wiersz wskazujący wpis katalogu, niezależnie od wielkości liter, staje się tym wpisem, a własne wartości Mistrza Gry (np. przygotowanie zaklęcia) są nakładane na wierzch. Kartę odczytuje się tolerancyjnie, ponieważ napisał ją model: nieznana nazwa jest usuwana, wartość dopasowywana do pola lub kolumny, a liczby zapisane obok karty nie są używane.

Wymyślone stworzenie niebędące bossem jest ograniczone do tego, co otwiera przed nim zestaw zasad. Filtr katalogu z `startFrom` wskazuje pole karty porządkujące wpisy (lista zaklęć pakietu 5e według `class`), a stworzenie zachowuje tylko wpisy, których filtr pasuje do jego własnej wartości tego pola, według dopasowania otwierającego selektor. Czarownik nie ma więc całej listy zaklęć, a stworzenie bez klasy nie otrzymuje nic z katalogu uporządkowanego według klasy. Pozostawione wybory są następnie uzupełniane bez ponownego pytania Mistrza Gry: dla każdej listy, której wiersze liczą się dopiero po wybraniu (`onlyWhen` źródła zdolności bojowych), spośród dostępnych i możliwych do opłacenia z własnych pul wpisów uzupełnia się dla każdej puli niewielką liczbę wpisów (więcej dla bardziej kompetentnego stworzenia), podobnie jak działania dostępne dowolnie. Dobór zależy od temperamentu i kompetencji, tych samych co w walce: stworzenie opiekuńcze lub wspierające sięga po to, co wspomaga jego stronę, lekkomyślne po obrażenia, metodyczne lub cierpliwe po ograniczanie wroga; im większa kompetencja, tym większa szansa na reakcję, kontrę lub coś innego zmieniającego turę. Losowanie używa ziarna samej walki, więc ta sama walka zawsze uzupełnia tak samo. Wiersz wskazany przez Mistrza Gry z takiej listy liczy się jako wybrany.

Bossa Mistrz Gry opisuje w całości, jako dopuszczalny wyjątek: nic nie jest odbierane ani uzupełniane.

Następnie oba warianty są ograniczane do swojego poziomu:

- Zdrowie trafia do przedziału poziomu przez jedyne pole, z którego jest odczytywane: maksimum puli jest tym polem albo `sum` zawierającym dokładnie jedno pole (maksimum punktów wytrzymałości w 5e, Toughness w Ember Roads). Formuła zdrowia bez takiego pojedynczego pola pozostaje zapisana bez zmian, a dziennik o tym informuje. Długość toru ran należy do ciebie i nigdy się nie zmienia.
- Po utworzeniu stworzenia obrona, premia trafienia i trudności rzutów obronnych są ograniczane do wartości poziomu plus dwa, a obrażenia zmniejszane, aż najlepsza runda mieści się w `damagePerRound` poziomu, z uwzględnieniem największej możliwej do opłacenia ceny. Najpierw ustępuje to, co kupuje droższy wariant, potem kości, stały składnik, uderzenie i dopiero na końcu rozmiar kości.

Twój własny bestiariusz nigdy nie jest ograniczany. To napisane przez ciebie dane, więc Engine przyjmuje je w zapisanej postaci.

### Pozycje: walka na planszy

Walka jest teatrem wyobraźni, dopóki blok nie określi wartości jednego pola planszy. Zadeklaruj `distance`, by można było walczyć na siatce; wtedy ruch, zasięg, obszary, linia widzenia, osłona i uderzenia w odchodzących nabierają znaczenia. Każda z tych wartości jest twoją liczbą; Engine dostarcza wyłącznie planszę.

```json
"distance": { "label": "ft", "perCell": 5 },
"ranged": { "long": "disadvantage", "adjacentFoe": "disadvantage" },
"cover": { "bonus": 2 },
"opportunity": { "budget": "reaction" }
```

Ember Roads deklaruje jedną linię i nic więcej – właśnie o to chodzi: reszta nie jest wymagana.

```json
"distance": { "label": "paces", "perCell": 2 }
```

**Pole.** `distance.perCell` określa, ile TWOICH jednostek odpowiada polu, a `label` jest nazwą jednostki. Używa jej każda odległość w świecie bloku: `economy.movement`, `speed` stworzenia, `reach` i `range` broni oraz `reach` i `range` akcji stworzenia. Katalog z własnym `units.distance` przelicza własne `mechanics.range` i `area.size` swoim `perCell`; katalog bez niego używa tej definicji. Dodatnia odległość jest zaokrąglana do najbliższego pola i nigdy do zera, więc wszystko z podaną liczbą sięga co najmniej jednego. Zero nie jest krótką odległością, lecz zachowuje własne znaczenie: `mechanics.range` równe 0 oznacza siebie lub dotyk (a dotyk innej osoby sięga sąsiedniego pola), zaś 0 w kolumnie `reach` lub `range` broni oznacza brak tego rodzaju zasięgu w wierszu.

**Czy walka odbywa się na planszy.** Dwa warunki muszą być spełnione: blok deklaruje `distance`, a gra gracza używa stylu walki Tactical. W stylu Classic lub zestawie zasad bez `distance` walka pozostaje teatrem wyobraźni jak dotychczas: każdy może celować w każdego i nic poniżej nie jest odczytywane.

**Co widzi gracz.** Plansza jest rysowana z terenem stylu taktycznego. Każde pole jest przyciskiem dostępnym wskaźnikiem lub strzałkami i opisuje, czym jest, kto na nim stoi oraz co oznacza dla niego częściowo dokonywany wybór. Ruch podświetla pola oferowane przez menu, każde z kosztem W TWOJEJ JEDNOSTCE, rysuje trasę i oznacza bursztynowo każde pole, którego trasa wywołałaby uderzenie, podając napastników pod planszą. Opcja wymagająca celu jednocześnie podświetla dozwolone osoby na planszy i liście. Opcja z `area` jest kierowana na pole, a pole pod wskaźnikiem wskazuje objęte osoby, w tym sojuszników. Pozostały ruch jest pokazywany obok budżetów, ponownie w twojej jednostce. Ekran niczego z tego nie mierzy: każde pole, koszt, trasa, cel i punkt celowania przychodzą z serwera.

**Ruch.** Przydział na turę to `economy.movement` członka drużyny albo własne `speed` stworzenia, podzielone przez `perCell` i zaokrąglone W DÓŁ, zawsze co najmniej jedno pole, jeśli ruch jest w ogóle możliwy. Odnawia się na początku własnej tury posiadacza i można go wydawać przed akcjami, między nimi i po nich: ruch, uderzenie, kolejny ruch. Wejście na pole kosztuje jeden lub więcej na trudnym terenie. Osiem kierunków kosztuje tyle samo, ponieważ tak gra się na siatkach stołowych, dla których powstał ten system. Można przejść przez sojusznika, ale nie zatrzymać się na nikim; przeciwnik jest ścianą; nie można wejść w stałą przeszkodę ani ściąć narożnika między dwoma zablokowanymi polami.

**Zasięg wręcz i dystansowy.** Wiersz broni pobiera je z `combat.attacks[].reach` i `.range`, każde z kolumny tej samej listy lub tej samej liczby dla wszystkich wierszy:

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

Kolumna o wartości 0 oznacza brak danego rodzaju zasięgu w wierszu, dzięki czemu zwykły miecz może być na jednej liście z toporem do rzucania. Wiersz zupełnie bez zasięgu wręcz sięga jednego pola. Akcja stworzenia używa własnego `reach` lub `range`, a zdolność katalogu – `mechanics.range` (0 oznacza siebie lub dotyk, czyli jedno pole przy wskazaniu innej osoby).

Wiersz z OBU zasięgami jest bronią rzucaną: w zasięgu wręcz jest zamachem, dalej strzałem. Poniższe zasady strzału nie dotyczą więc broni trzymanej w dłoni, można też uderzyć nią przechodzącego przeciwnika, czego nie da się zrobić łukiem.

Akcja stworzenia może też zawierać swój `area`, we własnej jednostce: `{ "shape": "cone",
"size": 15 }`, z `"friendlyFire": false` chroniącym własną stronę. Dzięki temu broń oddechowa jest rzeczywistym stożkiem na planszy, a nie liczbą celów. Sekwencja nie ma własnego kształtu; mają go wskazywane akcje. Walka bez planszy ignoruje kształt i używa `targetCount`, więc wpis stworzenia może podać oba i pozostać zgodny z prawdą w obu trybach.

**Jak daleko można posłać obszar.** Określa to `range`: kula rzucana na sto stóp ma tę wartość. Bez zasięgu wybuch następuje tam, gdzie go umieszczono, na własnym polu wykonawcy, a stożek lub linię można skierować na dowolne pole w obrębie rysowanej długości, ponieważ pole określa tu tylko kierunek. Dotyczy to zarówno `mechanics.area` wpisu katalogu, jak i stworzenia.

`ranged` określa koszt strzału ponad zwykłą odległość `normal` lub przy przeciwniku na sąsiednim polu. Każdy wariant to `"disadvantage"` albo `"normal"`; pominięcie bloku usuwa oba koszty. Zamach nigdy nie jest strzałem, więc nie dotyczą go żadne z tych zasad, podobnie jak broni rzucanej użytej w zasięgu wręcz.

**Obszary.** `mechanics.area` wpisu staje się rzeczywistym kształtem na planszy, kierowanym na pole, a nie osobę; `targetCount` go nie dotyczy: to kształt określa liczbę objętych osób. Obejmuje wszystkich na polach, przyjaciół i wrogów, chyba że wpis podaje `"friendlyFire": false`.

```
burst, size 2, aimed at X        cone, size 3, aimed right      line, size 3, aimed right
. . . . .                        . . . .                        . . . .
. # # # .                        . . # .                        A # # #
. # X # .                        A # # #                        . . . .
. # # # .                        . . # .
. . . . .                        . . . .
```

Wybuch obejmuje każde pole w odległości jego rozmiaru od wskazanego pola. Stożek biegnie od wykonawcy w kierunku tego pola, na każdym kroku szeroki tak samo jak odległy. Linia biegnie tak samo, z szerokością jednego pola. Wszystkie trzy zatrzymują się na stałych przeszkodach.

**Linia widzenia i osłona.** Prosta linia pól między uczestnikami: każda stała przeszkoda blokuje strzał i rozchodzenie się obszaru za nią, a celu po prostu nie ma w menu. Teren stanowiący osłonę dodaje `cover.bonus` do obrony, przeciw której wykonuje się atak; dziennik to odnotowuje. Nie ma osłony trzech czwartych, pełnej osłony ani wysokości.

**Uderzenia w odchodzących.** Zadeklaruj `opportunity.budget`; gdy walczący wychodzi z zasięgu stojącego wroga, który może działać, ma ten budżet i coś do uderzenia wręcz, ruch ZATRZYMUJE SIĘ w bieżącym miejscu, a wróg otrzymuje pytanie o uderzenie. Przyjęcie zużywa budżet i rozstrzyga ten sam atak dokładnie jak we własnej turze; przepuszczenie nic nie kosztuje. Potem ruch wznawia się z miejsca zatrzymania, płacąc za każde rzeczywiście przebyte pole, a uderzenie powalające poruszającego się kończy ruch tam, gdzie upadł. Każdy ma jedną okazję na cały ruch, niezależnie od liczby wyjść trasy z tego samego zasięgu. `disengage` zapobiega temu przez resztę tury; zestaw zasad bez `opportunity` nie ma żadnej z tych mechanik.

Pytanie jest OKNEM i zatrzymuje całą walkę: nic nie rusza, dopóki wszyscy pytani nie odpowiedzą. Za członka drużyny odpowiada gracz, wybierając uderzenie lub sąsiednie Pass; za pozostałych odpowiada ich prowadzący, a za bossa Mistrza Gry – jego własna decyzja. Zob. Okna poniżej.

**Co przeciwnik robi z planszą.** Przeciwnik nieprowadzony przez nikogo ocenia każde osiągalne pole względem każdej dostępnej z niego opcji, odejmuje wartość za każde uderzenie wywołane ruchem i woli pozostać w miejscu, jeśli już może wykonać najlepszą akcję. Gdy nic nie jest w zasięgu, skraca dystans; najpierw sprintem, jeśli lista `standard` zawiera `dash`.

**Możliwe odmowy.** `out-of-reach` (poza zasięgiem), `no-line-of-sight` (stała przeszkoda na drodze), `unreachable` (pole, którego ruch nie może opłacić lub na którym nie może się zakończyć) i `bad-cell` (obszar skierowany w niedozwolone miejsce).

### Co walka robi z twoim blokiem na serwerze

Gra, której zestaw zasad deklaruje `combat`, otrzymuje walkę rozstrzyganą przez ten blok, w tym samym zapisanym obiekcie bitwy, którego Engine używał dotychczas:

- **Kto uczestniczy.** Mistrz Gry wskazuje walczących; Engine odczytuje liczby każdego członka drużyny z jego własnej karty. Członek bez karty dla twojego zestawu zasad jest odrzucany imiennie, zamiast otrzymać liczby, których nie ma w twoim zestawie.
- **Skąd pochodzą liczby przeciwnika**, w tej kolejności: stworzenie wskazane przez Mistrza Gry w bestiariuszu, następnie stworzenie o etykiecie pasującej do imienia przeciwnika, potem blok statystyk zaproponowany przez Mistrza Gry dla tej walki i ograniczony do skali zagrożenia, a na końcu zwykłe stworzenie zbudowane z liczb poziomu. Każdy wariant zastępczy i każde ograniczenie jest zapisywane zwykłymi słowami, by walka mogła wyjaśnić działanie. Zestaw zasad bez wpisu bestiariusza, propozycji i skali zagrożenia odrzuca walkę zamiast ją wymyślać.
- **Karty są zapisem stanu.** Zdrowie, pule, stany, koncentracja i liczniki zasad umierania są zapisywane przez własne reguły karty po każdej zaakceptowanej akcji, więc ponowne wczytanie w trakcie walki pokazuje dokładnie jej ostatni stan; nie ma końcowego rozliczenia bitwy, które mogłoby się z nim nie zgadzać.
- **Menu określa jedyne dozwolone działania.** Każdy działający, gracz lub przeciwnik, wybiera identyfikator z tego samego menu generowanego przez blok. Przeciwnik prowadzony przez Engine wybiera z niego według własnej taktyki Engine, a przeciwnik prowadzony przez Mistrza Gry otrzymuje prośbę o wybór jednego identyfikatora z tego samego menu, z twoimi liczbami i bez ujawniania wyników kości.
- **Twoje kości.** Walka przechowuje własne ziarno i kursor, więc po odczycie z dysku kontynuuje z kośćmi, którymi i tak by rzuciła.

### Na ekranie

Walka odbywa się na ekranie bitwy w twoich słowach. Menu zawiera twoje ataki, zdolności, testy przeciwstawne i wymienione akcje standardowe; każda wskazuje koszt z twoich budżetów i pul. Widać kolejność tur, rundę, każdy nazwany stan z pozostałymi rundami, punkty tymczasowe, koncentrację i dwa liczniki zasad umierania. Dziennik pokazuje rzeczywiste obliczenia w twoich pojęciach: "Juno attacks Rust jackal with Road axe: 8 (5 + 3) + 3 = 11 against Guard 6, a hit." Każda zaakceptowana akcja jest natychmiast zapisywana na karcie, więc ponowne wczytanie w trakcie walki jest dokładne, a Mistrz Gry otrzymuje później informację, by nie zmieniać tych liczb ponownie.

Walka z pozycjami jest rysowana na planszy zamiast na scenie z portretami; działania gracza opisuje sekcja Pozycje. Każda odległość na planszy, w menu i dzienniku jest podawana w TWOJEJ jednostce: "Juno moves to 4, 6 for 6 paces and has 2 paces left."

### Testy przeciwstawne: chwytanie, pchanie, uwalnianie się

Niektóre ruchy nie są atakiem przeciw obronie, lecz testem przeciwstawnym: obie strony rzucają, a ta, której pójdzie lepiej, dopina swego. Chwycenie kogoś, przewrócenie go lub odepchnięcie oraz uwolnienie się mają ten sam kształt, więc zestaw zasad zapisuje każde z nich jako dane:

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

- **`checks`** to liczby odczytywane przez test przeciwstawny, każda będąca wartością z karty, odczytywaną raz na początku walki, tak jak obrona lub rzut obronny. Stworzenie zapisane zwykłymi liczbami podaje własne (`"checks": { "brawn": 2 }`); stworzenie z kartą odczytuje je z niej. Najwyżej dwanaście.
- **Rzut.** Obie strony rzucają twoimi `attackRoll.dice` i dodają najlepszy z testów, których mogą tu użyć (`attacker.checks`, `defender.checks`). Wygrywa wyższa suma; remis wygrywa broniący się, chyba że `ties` ma wartość `"attacker"`. Dziennik podaje obie strony: "Juno tries Grab on Ash-hound: 12 (6 + 6) + 3 = 15 with Brawn against 2 (1 + 1) + 2 = 4 with Brawn, and wins."
- **Wygrana** robi to, co określa `onWin`, co najmniej jedną rzecz:
  - `applies` nakłada stany na przegranego, ze zwycięzcą jako ich źródłem, więc stan oznaczony `endsWhenSourceDown` kończy się, gdy trzymający padnie. `rounds` nadaje stanowi zegar; bez niego stan trwa, dopóki coś go nie zakończy.
  - `ends` zdejmuje stany z wykonawcy lub celu.
  - `push` przesuwa przegranego prosto od zwycięzcy, o tyle we własnej jednostce odległości, zatrzymując się przed czymkolwiek stałym, kimkolwiek stojącym, krawędzią planszy i narożnikiem zbyt ciasnym, by się przecisnąć. Jest wymuszony: nie zużywa ruchu przegranego i nikt w niego nie uderza. Walka bez planszy nikogo nie przesuwa. Przegrana nic nie robi; tak wygląda nieudany chwyt.
- **`from: { "holding": "held" }`** sprawia, że test przeciwstawny celuje tylko w tego, kto nałożył ten stan na wykonawcę, i pojawia się w menu tylko wtedy, gdy stan trwa: tak wyraża się uwolnienie.
- **`reach`**, we własnej jednostce odległości, to zasięg; po pominięciu – sąsiednie pole. `reach` i `push` wymagają `combat.distance`.
- **`strike: true`** pozwala testowi zastąpić jedno uderzenie, gdy akcja kupuje kilka, tak jak atak: wykonany jako pierwszy zużywa budżet i zostawia resztę uderzeń w zapasie; wykonany przy uderzeniach w zapasie kosztuje jedno z nich. Tak chwyta i pcha przykład referencyjny 5e.
- **W menu** test przeciwstawny ma własną grupę i podaje szansę wygranej. Przeciwnik prowadzony przez Engine rozważa go jak wszystko inne, ale umiarkowanie: chwyt lub pchnięcie coś przygotowują, a uwolnienie jest warte najwięcej, ale tylko w czasie trzymania. Mistrz Gry prowadzący przeciwnika wybiera z tego samego menu.
- Testy przeciwnika wymyślonego przez Mistrza Gry są ograniczane do jego poziomu, tak jak jego szansa trafienia.

Testy przeciwstawne wymagają Capability API 1.43 w zestawie dostarczanym jako pakiet.

<a id="windows-holding-the-fight-open"></a>

### Okna: wstrzymywanie walki

Niektóre chwile należą do kogoś innego niż bieżący wykonawca. Engine pozostawia dla niego walkę otwartą, zamiast decydować za niego; ta pauza jest oknem.

Otwiera je sześć sytuacji, z których dwie wynikają z już zadeklarowanych danych:

- **Ktoś odchodzi.** Ruch wychodzący z zasięgu wroga zdolnego do uderzenia zatrzymuje się na tym kroku i pyta wroga. Zob. Uderzenia w odchodzących powyżej.
- **Między turami.** Po zakończeniu tury, zanim zacznie się następna, każdy przeciwnik z `signaturePoints`, którego stać na własną akcję `signature`, jest pytany o jej zakup. Można je kupować tylko wtedy: akcji specjalnej nie ma w menu niczyjej tury, także własnej.
- **Ktoś czegoś używa.** Przed rozstrzygnięciem pytana jest każda osoba z DRUGIEJ strony mająca wpis oczekujący takiej chwili, niezależnie od tego, w kogo wycelowano, o ile ten wpis sięga używającego. Akcja standardowa i test przeciwstawny nie otwierają takiej chwili.
- **Coś zostaje w kogoś wycelowane.** Przed rozstrzygnięciem pytana jest każda osoba z DRUGIEJ strony, w którą wycelowano i która ma wpis oczekujący takiej chwili. Leczący sojusznik nie stanowi zagrożenia wymagającego odpowiedzi, więc jego akcja nie otwiera okna.
- **Rzut ataku kogoś trafia.** Przed obrażeniami pytany jest trafiony, jeśli ma wpis oczekujący takiej chwili. To, co wykona, liczy się dla tego ataku: zobacz niżej Chwila po trafieniu.
- **Coś kogoś zraniło.** Po rozstrzygnięciu pytana jest każda zraniona osoba mająca wpis oczekujący TEJ chwili, niezależnie od sprawcy. Otrzymanie obrażeń jest faktem dotyczącym ciebie; wpis kierowany z powrotem na sprawcę nadal nie może być skierowany na sojusznika.

Cztery ostatnie sytuacje wpis katalogu lub własna akcja stworzenia mogą zamówić, wskazując oczekiwaną chwilę. Gdy jedna akcja otwiera zarówno `used`, jak i `aimed`, najpierw pyta się o użycie; jeśli nikt go nie odwoła, następnie pyta się tych, w których wycelowano, a akcja rozstrzyga się po obu odpowiedziach. O trafienie pyta się przy każdym trafiającym rzucie, a o zranienie po zakończeniu całej akcji.

Działanie okna, niezależnie od przyczyny otwarcia:

- **Gdy jest otwarte, nic innego nie rusza.** Ani wykonawca bieżącej tury, ani jej zakończenie, ani inne okno. Walka czeka.
- **Pyta po jednej osobie**, w kolejności tur, każdą raz. Pasowanie zawsze jest odpowiedzią i nic nie kosztuje. Osoba bez dostępnego działania jest pomijana zamiast pytana.
- **Wznawia dokładnie od miejsca zatrzymania.** Ruch kończy się na pozostałych polach, płacąc za każde rzeczywiście przebyte.
- **Odpowiada ten, kto prowadzi.** Okno własnego członka drużyny należy do ciebie, z opcją i sąsiednim Pass w menu; za przeciwnika odpowiada prowadzący, a o bossa Mistrza Gry pytany jest Mistrz Gry, z możliwością przepuszczenia chwili jako jedną z odpowiedzi.
- **Jest zapisywane razem z walką.** Gra zamknięta w trakcie ruchu wraca z tymi samymi osobami do zapytania i polami do przejścia.

Dla pierwszych dwóch niczego dodatkowo nie deklarujesz: zestaw zasad z `opportunity.budget` otrzymuje jedno, bestiariusz z `signaturePoints` drugie, a zestaw bez obu nigdy ich nie widzi.

**Wskazanie chwili oczekiwanej przez wpis.** Zapisz `mechanics.reaction` jako obiekt zamiast `true`:

```json
"reaction": { "on": "aimed", "at": "source", "cancels": true }
```

- `on` to `used`, `aimed`, `hit` albo `harmed` i umieszcza wpis w menu tego okna. Tylko te cztery chwile Engine obserwuje. Wpis nadal podający `"reaction": true` mówi jedynie, że nie wykonuje się go w turze; to za mało, by gdziekolwiek go zaoferować, więc nie pojawia się w żadnym menu.
- `at` to `source` (domyślne) albo `chosen`. `source` kieruje wykonywaną akcję na sprawcę chwili i uzupełnia cel, więc nikt go nie wybiera; `chosen` zachowuje własne cele wpisu i pyta o wybór.
- `cancels` całkowicie zatrzymuje wydarzenie wstrzymane przez okno. Może je podać tylko wpis `used` lub `aimed`: nie można odwołać tego, co już się wydarzyło, a trafienie zostało już rzucone.
- `against` zawęża to, na co wpis odpowiada, do rzeczy użytych z określonych katalogów: `"against": { "catalogs": ["spells"] }` odpowiada na zaklęcie i na nic innego. Akcja bez wpisu za sobą (broń na liście, własna akcja stworzenia) nie pochodzi z żadnego katalogu, więc nigdy nie otwiera tej chwili dla wpisu wskazującego katalogi. Pomiń `against`, a wpis będzie odpowiadał na wszystko.

Nadaj też `budget`, inaczej zostanie wydany domyślny budżet listy. Reakcja niemal zawsze zużywa własny budżet, który uniemożliwia kilka reakcji w jednej turze.

**Koszt jest wydawany, zanim ktokolwiek zostanie zapytany.** Anulowana akcja nie dochodzi do skutku, ale została już kupiona: budżet i pule są już zużyte. Jeśli twój system je zwraca, nie potrafi jeszcze tego zadeklarować.

Kontrę zapisuje się tak; na planszy odpowiada ona tylko komuś w obrębie własnego `range`:

```json
"mechanics": {
  "kind": "utility",
  "range": 8,
  "free": true,
  "cost": [{ "pool": "luck", "amount": 1 }],
  "reaction": { "on": "used", "cancels": true, "against": { "catalogs": ["knacks"] } }
}
```

**Chwila po trafieniu.** Wpis z `hit` to sposób zapisania Shield, Parry i Uncanny Dodge. Atak zostaje wstrzymany po rzucie, a trafiony jest pytany, zanim zostaną zadane jakiekolwiek obrażenia. To, co wykona, liczy się dla tego ataku: jego rzut nie jest powtarzany, lecz ponownie sprawdzany względem obecnej obrony trafionego, więc stan dodający do obrony może zamienić trafienie w chybienie, a naturalny wynik, który zawsze trafia, nadal trafia. Stan trwający jeden atak (`endsAfter: "attacked"`) nałożony w odpowiedzi obejmuje ten atak, łącznie z obrażeniami, i zostaje przez niego zużyty; tak wyraża się "zmniejsz obrażenia tego ataku o połowę": `resist-all` na jeden atak.

```json
"mechanics": {
  "kind": "buff",
  "targets": "self",
  "budget": "reaction",
  "reaction": { "on": "hit" },
  "applies": [{ "condition": "shielded", "duration": { "rounds": 1, "at": "turn-start" } }]
}
```

Atak wznawia się dokładnie tam, gdzie go wstrzymano: z pozostałymi celami, a w przypadku akcji stworzenia złożonej z innych akcji – z pozostałymi częściami. Każdy rzut trafiający kogoś, kto ma taką odpowiedź, zostaje po kolei wstrzymany, a walka zapisana przy otwartym oknie wraca w tym samym stanie. Atak wykonany wewnątrz okna (uderzenie w odchodzącego, akcja specjalna, odpowiedź) nigdy nie jest wstrzymywany, bo nic otwartego wewnątrz okna nie otwiera kolejnego. Gdy okno się otwiera, dziennik pokazuje rzut i to, przeciw czemu go wykonano, a gdy odpowiedź zmieniła obronę – czy atak teraz chybia. Przeciwnicy prowadzeni przez Engine bronią się tylko wtedy, gdy zamienia to trafienie w chybienie.

Pakiet wskazujący chwilę wymaga Capability API 1.33, a używający `used` lub `against` – 1.44. Pakiet czekający na `hit` wymaga 1.46.

### Jeszcze niedostępne

Wprost, ponieważ zestaw zasad nie powinien przypisywać Engine nieistniejących możliwości:

- **Poza prostą planszą**: brak osłony trzech czwartych i pełnej osłony, wysokości, latania nad przeszkodami, przeciskania się, wierzchowców, ukrywania i zaskoczenia, a nic nikogo nie przesuwa poza jego własnym ruchem i pchnięciem z testu przeciwstawnego. Nikt nie ciągnie trzymanego stworzenia.
- **Test przeciwstawny jest prosty.** Nie ma ograniczeń rozmiaru i nie otwiera okna (nikt nie może na niego odpowiedzieć).
- **Wpis może oczekiwać tylko czterech chwil**, `used`, `aimed`, `hit` i `harmed` (zob. Okna powyżej). Te chwile Engine obserwuje w imieniu wpisu; dwa pozostałe okna, odejście i pauza między turami, otwiera sama walka, więc wpis nie może o nie prosić. Nie ma chwili rzutu obronnego, śmierci, początku tury ani upadku czegokolwiek.
- **Kontra zatrzymuje to, na co odpowiada, bezwarunkowo.** Nie odróżnia jednego wpisu katalogu od innego i nie ma testu, który pozwalałby zatrzymać coś potężniejszego od niej samej.
- **Bez łańcucha okien.** Walka przechowuje jedno okno zamiast stosu, więc nic wewnątrz okna nie otwiera kolejnego: kontry nie można skontrować, obrażenia reakcji nie otwierają następnej chwili, a atak wykonany wewnątrz okna nie jest wstrzymywany, gdy trafia.
- **Trafieniem jest tylko rzut ataku.** Coś, co trafia bez niego (pociski, które po prostu trafiają, obszar, przeciw któremu wszyscy wykonują rzut obronny), nie otwiera chwili `hit`, a odpowiedź nie odróżnia zamachu od strzału, więc akcja Parry, którą zasady ograniczają do mieczy, paruje też strzały.
- **Obrażenia zmienia się przez zmniejszenie o połowę, nie o określoną ilość.** Odpowiedź może zmniejszyć obrażenia jednego ataku o połowę (`resist-all` dla tego ataku); nie może odjąć od nich wyrzuconej liczby.
- **Nie ma zwrotów.** Koszt anulowanej akcji pozostaje wydany.
- **To, co zmienia stan, jest zamkniętą listą.** Obrona, rzuty ataku, rzuty obronne, testy przeciwstawne i szybkość (tak samo dla poziomu i działania przedmiotu w walce) i nic więcej: żadnej premii do obrażeń zadawanych przez posiadacza ani poziomu, który obniża maksimum postaci lub wyłącza ją z walki.
- **Stworzenie zapisane zwykłymi liczbami nie ma toru ran.** W zestawie zasad ze zdrowiem jako torem takie stworzenie nadal traci punkty; nadaj mu `sheet`, by ciosy zaznaczały pola po wcześniejszym uwzględnieniu jego `resist`, `vulnerable` i `immune`.
- **Efekt dodatkowy uruchamia się sam.** `on` ma jedną wartość, `hit`, więc otrzymuje go pierwsze kwalifikujące trafienie okresu; nie ma chwili, w której pojawia się pytanie, czy go zużyć.
- **Wartość przesuwana atakami jest prosta.** Cios wydający rzuca tą wartością i niczym więcej, więc żadna broń jej nie zmienia, łącznie z jej `floor` (wytrzymały cel pochłania cios zabierający, a jego twardość zatrzymuje cios wydający, ale nic innego nie zmniejsza żadnego z nich); załamanie ustępuje po stałej liczbie tur, niezależnie od jego głębokości; a atak wykonany w oknie (uderzenie w odchodzącego, reakcja, akcja specjalna) jest wykonywany w pierwszym stylu.
- **Wymyślony przeciwnik niczego nie pochłania i nie ma twardości.** Stworzenie wymyślone przez Mistrza Gry na jedną walkę jest dopasowywane do twojej skali zagrożenia, która nic nie mówi o pochłanianiu ani twardości, więc nie ma żadnego z nich.

## Warstwy: warianty twojego zestawu zasad

Warstwa to nazwany wariant zestawu zasad, włączany przez gracza przy tworzeniu gry: mało magii, ciężka zima, bardziej surowy poziom trudności. Warstwy znajdują się w pliku zestawu zasad, w opcjonalnej tablicy `layers`, więc podróżują z nim i nigdy nie znikają z gry, która ich używała. Kreator pokazuje je jako przełączniki pod zestawem zasad, a wybór pozostaje stały przez cały czas istnienia gry, dokładnie jak sam zestaw.

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

**Co może robić warstwa.** Lista jest zamknięta; każdy efekt coś zawęża albo dodaje tekst:

- `gm.guidance` jest dopisywane do `gm.checkGuidance`, po twoim tekście i tekście wcześniejszych warstw. `gm.worldGuidance` jest tak samo dopisywane do `gm.worldGuidance`.
- `fields` usuwa wartości z pola **enum**. `removeValues` wskazuje już istniejące wartości; co najmniej jedna musi pozostać, a jeśli `default` pola jest usuwane, warstwa wskazuje w zamian pozostające `default`.
- `difficultyLadder` zastępuje drabinę inną, w formacie twojego rodzaju rozstrzygania: `{label, dc}` dla `dice-sum` i `{label, successes, target?}` dla `dice-pool`. Podlega dokładnie tym samym kontrolom co własna drabina. Gdy kilka aktywnych warstw ją deklaruje, wygrywa ostatnia.
- `catalogs` ukrywa wpisy w selektorze edytora karty. Każda reguła wskazuje jeden z zadeklarowanych `filters` katalogu i dokładnie jedno porównanie: `above` albo `below` dla filtra `number`, `equals` albo `notIn` dla `text` lub `tags`. Wpis, który w ogóle nie ustawia tego filtra, nigdy nie jest ukrywany.
- `currencies` usuwa monety: `removeUnits` wskazuje pojedyncze monety, a `removeFamilies` całe rodziny. Najmniejsza moneta rodziny znika tylko razem z rodziną, więc każda pozostała rodzina nadal może płacić i wydawać resztę. Gdy warstwa jest włączona, nikt nie zarabia usuniętej monety, nie płaci nią, nie wybiera jej ani nie znajduje jej w łupach, a wiersz sakiewki ją pomija. Cena podana w takiej monecie jest wyrażana w największej pozostałej monecie, która pokrywa ją dokładnie, przy tej samej wartości: długa noc z Gravewatch usuwa koronę, więc pistolet strażniczy za 3 korony kosztuje 15 szylingów. Przedmiot, którego cała rodzina monet zniknęła, nie ma ceny.

**Czego warstwa nie może robić.** Nie może dodać wartości wyliczenia, pola, umiejętności, puli ani odpoczynku, zmienić rodzaju rozstrzygania, dotknąć bieżącego stanu lub liczb walki ani dodać wywołania modelu. Wartość _dodana_ przez warstwę byłaby nieznana wszystkim innym czytnikom karty, więc wartości mogą jedynie znikać. Wszystko poza tą listą wymaga zmiany samego zestawu zasad albo drugiego zestawu.

**Konflikty.** `conflicts` wskazuje warstwy, których nie można włączyć razem. Wystarczy zadeklarować jedną stronę pary. Kreator wyłącza drugi przełącznik; jeśli zapisany wybór mimo to zawiera obie warstwy, odrzuca zadeklarowaną **później**, więc te same dwa wybory zawsze dają te same zasady.

**Karta już zawierająca usuniętą wartość zachowuje ją.** Nic nie przepisuje postaci. Edytor przestaje jedynie oferować wartość, a postać, która już ją miała, nadal ją pokazuje. Wyłącz warstwę w nowej grze, a wartość znów będzie oferowana. Tak samo działa ukryty wpis katalogu: znika z selektora, a wcześniej wybrany wiersz pozostaje na karcie. Monety usunięte przez warstwę, które ktoś już niesie, zostają w jego torbie.

**Limity.** 12 warstw na zestaw zasad i 4000 znaków instrukcji na warstwę, łącznie dla obu tekstów. Pakiet zestawu zasad deklarujący `layers` lub bazowe `gm.worldGuidance` wymaga Capability API 1.25, a taki, którego warstwy usuwają monety – 1.64. Importowany zestaw zasad weryfikuje odczytujący go Engine, więc niczego takiego nie potrzebuje.

**Warstwy pisane przez inne osoby** (warstwa małej ilości magii dla cudzego zestawu zasad, dostarczana w osobnym pliku) są późniejszym rozszerzeniem. Obecnie warstwa jest dostarczana wewnątrz zestawu, do którego należy.

<a id="trying-your-ruleset"></a>

## Wypróbowanie zestawu zasad

Zestawy zasad społeczności używają tego samego przełącznika co importowani agenci. Otwórz **Settings** (ustawienia) > **Advanced** (zaawansowane) > **Danger Zone** (strefa niebezpieczna) i upewnij się, że **Allow custom Agent imports** (zezwalaj na import własnych agentów) jest włączone. Import wymaga też dostępu przez localhost lub skonfigurowanego **Admin Access** (dostępu administratora).

1. Otwórz panel **Agents** i wybierz przycisk **Import agents** (importuj agentów; ikona pobierania w rzędzie przycisków u góry panelu).
2. Wybierz **Game Mode ruleset** (zestaw zasad trybu Game) i swój plik JSON.
3. Przeczytaj podgląd. Pokazuje nazwę, wersję, licencję, zakres obsługi zestawu zasad i tekst Mistrza Gry. Wybierz **Import** (importuj).

Twój zestaw zasad pojawi się w sekcji **Rules** (zasady) panelu oraz w wyborze **Rules** kreatora nowych gier. Zestaw importowany z pliku jest zapisywany jako `local/<your id>`, więc nie można pomylić go z oficjalnym zestawem ani z cudzym.

### Zmiana już zaimportowanego zestawu zasad

Zaimportowana wersja nigdy nie jest nadpisywana. Jeśli zmienisz plik i zaimportujesz go ponownie z tym samym `version`, import zostanie odrzucony z prośbą o zwiększenie numeru. To celowe: gra jest związana z dokładną wersją, na której ją utworzono, więc trwająca kampania nigdy nie obudzi się z inną matematyką.

Cykl podczas tworzenia wygląda więc tak: edytuj, zwiększ `version`, importuj, rozpocznij nową grę. Stare wersje pozostają zainstalowane obok nowej, dopóki nie usuniesz zestawu z sekcji **Rules**. Usunięcie zestawu nadal używanego przez grę sprawia, że gra zgłasza jego brak aż do ponownego importu.

Jeśli zmienisz kształt karty (dodasz, usuniesz lub zmienisz nazwy elementów), zwiększ też `sheet.version`. Istniejące karty są odczytywane tolerancyjnie: wartości nieznane nowej karcie zostają zachowane, a brakujące otrzymują wartości domyślne.

## Udostępnianie zestawu zasad

**Jako plik.** Wyślij znajomemu plik JSON. Zaimportuje go tak samo jak ty.

**Z repozytorium GitHub.** Jeśli przechowujesz pracę w publicznym repozytorium GitHub, umieść każdy zestaw zasad w folderze `rulesets` w głównym katalogu repozytorium, po jednym pliku na zestaw:

```text
your-repository/
  agents.json        (optional, only if you also share agents)
  rulesets/
    ember-roads.json
    another-system.json
```

Użytkownik dodaje twoje repozytorium raz przez listę własnych repozytoriów agentów, przegląda zawartość i może później synchronizować nowe wersje. Lista własnych repozytoriów to zaawansowana funkcja włączana przez osobę prowadzącą serwer za pomocą `ENABLE_CUSTOM_AGENT_REPOS=true`. Zestawy z repozytorium są zapisywane pod nazwą właściciela repozytorium, np. `alice/ember-roads`, więc dwóch autorów może opublikować zestaw `v20` bez kolizji.

Obowiązują dwa limity. Repozytorium może zawierać najwyżej 32 pliki JSON bezpośrednio w `rulesets`; większa liczba powoduje odrzucenie. Konto o nazwie `local` nie może publikować zestawów, ponieważ `local/` jest zarezerwowane dla importów z pliku.

**W oficjalnym katalogu.** Popularny system z jasną licencją może być oferowany wszystkim przez **Download Agents** (pobierz agentów). Wymaga to pull requestu do repozytorium [Marinara-Agents](https://github.com/Pasta-Devs/Marinara-Agents). Układ można sprawdzić w tamtejszym pakiecie `ruleset-5e-2014`.

## Licencjonowanie

Publikuj tylko tekst zasad, który masz prawo udostępniać. Wiele systemów publikuje dokument referencyjny na otwartej licencji i właśnie z niego można kopiować. Umieść identyfikator licencji oraz wymagany przez nią tekst uznania autorstwa w `license`. Nie kopiuj tekstu z podręczników bez otwartej licencji. Zestaw zasad potrzebuje głównie nazw i liczb, a tekst Mistrza Gry powinien być napisany twoimi słowami.

## Rozwiązywanie problemów

- **Import mówi, że nazwa nie istnieje.** Coś w pliku wskazuje niezadeklarowany identyfikator, np. umiejętność wskazująca usuniętą cechę. Komunikat podaje ścieżkę do wiersza.
- **Import mówi, że wersja jest już zainstalowana z inną zawartością.** Zwiększ `version` i importuj ponownie.
- **Mojego zestawu zasad nie ma w kreatorze.** Sprawdź, czy **Allow custom Agent imports** jest włączone. Gdy jest wyłączone, importowane zestawy nie są dostępne dla nowych gier. Gry już ich używające nadal działają.
- **Gra mówi, że brakuje zestawu zasad.** Dokładna wersja, na której utworzono grę, nie jest zainstalowana. Zaimportuj ponownie tę wersję pliku.
