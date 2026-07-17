# Mirr — konektor Claude Code

Plugin łączy [Claude Code](https://claude.com/claude-code) z **[Mirr](https://app.mirr.pl)** — systemem Budowalki dla firm budowlanych (projekty, wyceny, oferty, faktury kosztowe, czas pracy ekipy). Po instalacji pracujesz na danych swojej firmy prosto z czatu: „zrób wycenę remontu łazienki dla Kowalskiego", „zapisz fakturę za paliwo na budowę przy Polnej", „ile godzin ekipa zrobiła w tym tygodniu".

## Co dostajesz

- **Konektor do Twojego konta Mirr** — Claude czyta i zapisuje projekty, wyceny, faktury i czas pracy bezpośrednio w Mirr. Dane zostają w systemie, nie w rozmowie.
- **Skill `wycena`** — prowadzi wycenę krok po kroku: ceny brane z Twojego cennika (nie zgadywane), braki dopytywane, wynik zapisany w Mirr.
- **Skill `praca-z-mirr`** — pilnuje, żeby efekty pracy (oferty, faktury, wpisy czasu) trafiały do Mirr, a nie ginęły w czacie.

## Wymagania

- Konto w [Mirr](https://app.mirr.pl) — konektor jest częścią wdrożenia Budowalki.
- Klucz API do konektora — otrzymasz go podczas wdrożenia (kontakt: hej@budowalka.pl).
- Zainstalowany [Claude Code](https://claude.com/claude-code).

## Instalacja

### W aplikacji Claude na komputerze (najprościej — bez klucza)

1. W ustawieniach Claude wejdź w **Plugins** i kliknij **Add** → **Add marketplace**.
2. Wybierz **Add from a repository** i wpisz: `Budowalka/mirr-plugins`
3. Dodaj plugin **Mirr** plusem.
4. Claude poprosi o zalogowanie — podaj swój mail i hasło do Mirr i kliknij **Zezwól**. Gotowe.

Połączenie widzisz (i możesz unieważnić) w Mirr na stronie [Agenci](https://app.mirr.pl/agents).

### W Claude Code (terminal): niech Claude zrobi to za Ciebie

Masz już zainstalowany Claude Code? Przygotuj swój klucz API, otwórz Claude Code i wklej mu tę wiadomość:

> Zainstaluj konektor Mirr. Wykonaj po kolei: (1) `claude plugin marketplace add Budowalka/mirr-plugins`, (2) `claude plugin install mirr@mirr-plugins`, (3) zapytaj mnie o klucz API i dopisz do mojego pliku `~/.zshrc` linijkę `export MIRR_TOKEN="<mój-klucz>"`, (4) na końcu poproś, żebym zamknął i uruchomił Claude Code ponownie.

Claude wykona wszystkie kroki sam i poprosi Cię tylko o wklejenie klucza. Po ponownym uruchomieniu sprawdź połączenie: napisz w czacie „pokaż moje projekty w Mirr".

### Ręcznie, krok po kroku

Każdy krok to jedna komenda: wklej ją do terminala i naciśnij Enter.

1. Otwórz aplikację **Terminal**.

2. Dodaj katalog pluginów Budowalki:

   ```bash
   claude plugin marketplace add Budowalka/mirr-plugins
   ```

3. Zainstaluj plugin Mirr:

   ```bash
   claude plugin install mirr@mirr-plugins
   ```

4. Zapisz swój klucz API — **podmień `tutaj-twój-klucz` na klucz otrzymany od Budowalki** (cudzysłowy zostaw):

   ```bash
   echo 'export MIRR_TOKEN="tutaj-twój-klucz"' >> ~/.zshrc
   ```

5. Przeładuj ustawienia terminala, żeby klucz zaczął działać:

   ```bash
   source ~/.zshrc
   ```

6. Zamknij Claude Code i uruchom go ponownie — plugin ładuje się przy starcie.

7. Sprawdź połączenie — napisz w czacie: „pokaż moje projekty w Mirr". Jeśli Claude pokaże Twoje projekty, wszystko działa.

## Aktualizacja

```bash
claude plugin marketplace update mirr-plugins
```

## Bezpieczeństwo klucza

Klucz API daje dostęp do danych Twojej firmy w Mirr. Traktuj go jak hasło:

- nie wklejaj go do plików projektów, repozytoriów ani wiadomości,
- trzymaj go wyłącznie w zmiennej środowiskowej `MIRR_TOKEN` na swoim komputerze,
- podejrzewasz wyciek albo zgubiłeś komputer? Napisz na hej@budowalka.pl — unieważnimy klucz od ręki i wydamy nowy. Stary przestaje działać natychmiast.

Samo repozytorium nie zawiera żadnych danych ani sekretów — bez klucza plugin niczego nie odczyta.

## Dla kogo

Mirr budujemy dla małych firm budowlanych w Polsce. Jeśli chcesz pracować tak u siebie: [budowalka.pl](https://budowalka.pl).
