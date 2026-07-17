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

1. W terminalu dodaj marketplace i zainstaluj plugin:

   ```bash
   claude plugin marketplace add Budowalka/mirr-plugins
   claude plugin install mirr@mirr-plugins
   ```

2. Ustaw klucz API w środowisku (macOS, terminal):

   ```bash
   echo 'export MIRR_TOKEN="tutaj-twój-klucz"' >> ~/.zshrc
   source ~/.zshrc
   ```

3. Uruchom Claude Code na nowo i sprawdź połączenie — napisz w czacie: „pokaż moje projekty w Mirr".

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
