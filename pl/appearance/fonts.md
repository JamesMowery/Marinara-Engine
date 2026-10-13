# Własne czcionki i Google Fonts

Z tego przewodnika dowiesz się, jak zmienić czcionkę używaną w całej aplikacji Marinara Engine. Do wyboru są trzy drogi: czcionka wbudowana, własne pliki czcionek albo czcionka pobrana z serwisu Google Fonts po nazwie.

## Wybór czcionki aplikacji

Ustawienie czcionki znajduje się w panelu **Settings** (Ustawienia), w zakładce **Appearance** (Wygląd), w sekcji **Text & Scale**.

1. Otwórz panel **Settings** i kliknij zakładkę **Appearance**.
2. Znajdź sekcję **Text & Scale**.
3. Rozwiń listę rozwijaną **Font**.
4. Wybierz czcionkę z listy.

Domyślnie ustawiona jest opcja **Default (Inter)**. Inter to czysty krój wybrany z myślą o czytaniu z ekranu. Dodane własne czcionki pojawiają się na tej samej liście rozwijanej **Font**, pod opcją domyślną.

Pola promptów w edytorze presetów i pole Example Dialogue na karcie postaci używają czcionki o stałej szerokości znaków, dopóki wybrana jest czcionka domyślna. Wybierz dowolną inną czcionkę, a te pola też będą jej używać. Pozostałe pola promptów, na przykład szablon promptu agenta, zostają przy czcionce o stałej szerokości znaków.

Widgety czatu mają osobną opcję **Font** w sekcji **Settings → Appearance → App → Chat widget style** (ustawienia, wygląd, aplikacja, styl widgetów czatu). Włącz w tym samym miejscu przełącznik **Apply preset font** (zastosowanie czcionki presetu), aby ta czcionka była używana także w wiadomościach, polach wpisywania i kontrolkach czatu. Gdy przełącznik jest wyłączony, zachowują one zwykłą czcionkę. Zobacz [Gotowe style okien czatu](custom-css-themes.md#ready-made-chat-window-styles).

Wybór czcionki synchronizuje się między urządzeniami. Po wybraniu czcionki przełącza się na nią każda przeglądarka i każde urządzenie połączone z tym samym serwerem Marinara. Jak działa ta synchronizacja, wyjaśnia przewodnik [Przegląd ustawień](../settings/settings-overview.md).

## Dodawanie własnych czcionek

Własną czcionkę dodaje się przez wrzucenie pliku czcionki do folderu na serwerze. To ta maszyna, na której działa aplikacja Marinara Engine.

1. Na maszynie serwera znajdź folder `data/fonts/` wewnątrz folderu danych aplikacji Marinara Engine.
2. Skopiuj do niego plik czcionki.
3. Wróć do panelu **Settings**, potem **Appearance**, potem **Text & Scale**.
4. Rozwiń listę rozwijaną **Font**. Czcionka jest już na liście.
5. Zaznacz ją.

Marinara odczytuje takie typy plików czcionek: `.ttf`, `.otf`, `.woff` oraz `.woff2`. Pliki z każdym innym rozszerzeniem są pomijane.

Nazwę wyświetlaną Marinara tworzy z nazwy pliku. Przykładowo plik `OpenSans-Bold.ttf` pokazuje się jako "Open Sans". Nazywaj więc pliki czytelnie, jeśli lista ma wyglądać porządnie.

Pliki czcionek z folderu `data/fonts/` leżą na serwerze. Może z nich korzystać każde urządzenie połączone z tym samym serwerem Marinara. Wybór czcionki synchronizuje się także między tymi urządzeniami, więc wszystkie pokazują tę samą czcionkę.

## Pobieranie z Google Fonts

Marinara potrafi pobrać czcionkę prosto z serwisu Google Fonts. Serwer musi mieć do tego dostęp do internetu.

1. Otwórz panel **Settings**, potem **Appearance**, potem **Text & Scale**.
2. Znajdź pole **Google Fonts**.
3. Wpisz dokładną nazwę czcionki, na przykład `Fira Code` albo `Lora`.
4. Kliknij przycisk **Add**.
5. Poczekaj na koniec pobierania. Nowa czcionka pojawia się wtedy na liście rozwijanej **Font**.

Wpisz nazwę dokładnie w takiej pisowni, jakiej używa serwis Google Fonts. Obok pola znajduje się link **Browse fonts at fonts.google.com**. Otwiera on stronę Google Fonts w nowej zakładce, więc nazwy da się tam sprawdzić.

Nazwa może zawierać wyłącznie litery, cyfry i spacje. Przy ponownym pobraniu tej samej czcionki Marinara zastępuje starą kopię, zamiast tworzyć duplikat.

Jeśli pobieranie się nie powiedzie, przeczytaj komunikat błędu. Kiedy Marinara nie może połączyć się z serwisem Google Fonts, prosi o sprawdzenie połączenia z internetem. Kiedy komunikat mówi, że czcionki nie znaleziono, przyczyny są dwie. Nazwa może nie pasować do żadnej czcionki w serwisie Google Fonts. Możliwe też, że czcionka nie ma odmiany zwykłej (400), czyli standardowego stylu bez pogrubienia. Sprawdź pisownię, a na stronie Google Fonts sprawdź, czy czcionka oferuje styl Regular.

## Przycisk Open Fonts Folder działa tylko lokalnie

Obok listy rozwijanej **Font** znajduje się przycisk **Open Fonts Folder** (otwarcie folderu czcionek). Otwiera on folder `data/fonts/` w eksploratorze plików na maszynie serwera.

Ten przycisk działa na serwerze, a nie na urządzeniu, na którym przeglądasz aplikację Marinara Engine. Jeśli aplikacja Marinara Engine działa na twoim własnym komputerze, folder po prostu się otworzy. Przy połączeniu z telefonu albo drugiego komputera przycisk nic przydatnego nie zrobi. W takim wypadku skopiuj pliki czcionek do folderu `data/fonts/` na serwerze samodzielnie.

## Powiązane przewodniki

- [Ustawienia wyglądu](appearance-settings.md)
