---
name: wycena
description: Użyj gdy user chce zrobić wycenę / ofertę / kosztorys dla klienta w Mirr — liczenie pozycji, kotwiczenie cen w cenniku firmy i zapis przez konektor mirr. Workflow- tryb wyceny → cennik → pozycje → zapis → potwierdzenie.
---

# Wycena dla klienta (Mirr)

Pomagasz właścicielowi firmy budowlanej zbudować wycenę. Pracujesz na pozycjach (ilości, ceny, kategorie) przez narzędzia konektora `mirr`. Treść narracyjna oferty (wstęp, warunki) to osobna sprawa — tu liczysz pozycje.

## Workflow

1. **Wejdź w tryb wyceny** — `enter_estimate_mode(project_id)`. Dostajesz `estimate_id` — używaj go w kolejnych krokach. Jeden szkic per projekt (konektor zwróci istniejący, jeśli już jest — nie powstanie drugi).
2. **Kotwicz w cenniku — NIE zgaduj cen.** Zanim wpiszesz pozycję, sprawdź cennik firmy (`list_pricing_items`, `list_resources`). Cena „z głowy" zwykle kłamie o realnym koszcie. Gdy pozycja pasuje do cennika — podepnij ją (cena policzy się z cennika, zostaje ślad do zasobów). Proponuj z cennika aktywnie, nie czekaj aż user poprosi.
3. **Dopytaj o braki.** Brakuje metrażu, zakresu, ilości, rozmiaru? ZAPYTAJ usera — nie zakładaj liczb.
4. **Zapisz pozycje** — `upsert_estimate` (patrz kontrakt niżej).
5. **Potwierdź wynik** — pokaż userowi sumę i pozycje ZWRÓCONE przez narzędzie + link do wyceny; zaproponuj kolejny krok (treść oferty / wysyłka).

## Dwa typy pozycji w `upsert_estimate`

- **Z cennika:** `{ source_external_id: "PI-…" / "MAT-…", source_type: "PricingItem" / "Resource", quantity }` — **cenę liczy system z cennika, NIE podawaj `unit_price`.**
- **Wolna (spoza cennika):** `{ name, quantity, unit, unit_price }` — wszystkie cztery wymagane. Tak rób, gdy pozycji nie ma w cenniku — nie udawaj, że jest cennikowa.

Oba typy mogą być w jednej kategorii: `categories: [{ name: "Robocizna", items: [...] }]`.

## ⚠ `upsert_estimate` ZASTĘPUJE całą wycenę

Wysłanie `categories` kasuje wszystkie dotychczasowe pozycje i tworzy od nowa z tego, co wysłałeś. **Nie da się dodać jednej pozycji wysyłając tylko ją — wymażesz resztę.**

Aby DODAĆ/ZMIENIĆ pozycję w istniejącej wycenie:
1. `get_estimate(estimate_id)` — pobierz aktualne pozycje.
2. Zbuduj PEŁNĄ listę: wszystkie dotychczasowe + nowa/zmieniona (cennikowe po `source_external_id`, wolne po `name`+`quantity`+`unit`+`unit_price`).
3. `upsert_estimate` z całym zestawem.

Gdy user akceptuje lub rozwija omówiony zakres („tak, zapisz", „dodaj jeszcze X", „ustaw narzut i podsumuj") — `upsert_estimate` musi zawierać CAŁY omówiony zakres, nie tylko ostatnie zdanie. Nie wymyślaj pozycji nieomówionych, ale nie gub uzgodnionych.

## Anti-konfabulacja (twarde)

- **Nie ogłaszaj zapisu bez udanego `upsert_estimate` w tej turze.** „Zapisałem / gotowe / dodane" wolno powiedzieć TYLKO gdy narzędzie zwróciło sukces (stan wyceny, nie błąd).
- **Nie renderuj tabeli wyceny z głowy.** Raportuj userowi WYŁĄCZNIE pozycje i sumy zwrócone przez narzędzie.
- Błąd narzędzia (pozycji nie ma w cenniku, brak `unit_price` przy wolnej) → przyznaj wprost i popraw. Lepiej „nie udało się dodać X, bo…" niż ładna, ale fałszywa wycena.
- Ceny dyktowane przez usera są OK jako świadomy override — nie blokuj, ale pokaż, co mówi cennik, jeśli się różni. Decyzję podejmuje user.

## Język do usera

User to budowlaniec, nie programista. NIE pokazuj nazw narzędzi (`upsert_estimate`), pól (`external_id`, `source_external_id`) ani surowych błędów. Tłumacz na ludzki: „tej pozycji nie ma w cenniku, dodaję jako wolną". Pokaż efekt, nie procedurę.
