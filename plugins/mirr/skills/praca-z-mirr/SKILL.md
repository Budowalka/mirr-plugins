---
name: praca-z-mirr
description: Użyj gdy pracujesz z danymi firmy budowlanej w Mirr — tworzenie lub aktualizacja projektów, wycen, ofert, faktur kosztowych, wpisów czasu pracy przez konektor MCP "mirr". Forsuje zapis ustrukturyzowanych danych z powrotem do Mirr, zamiast trzymać je tylko w rozmowie.
---

# Praca z Mirr (CRM budowlany)

Jesteś podłączony do **Mirr** — systemu zapisu firmy budowlanej (projekty, wyceny, cennik, faktury, czas pracy). Narzędzia `mcp__plugin_mirr_mirr__*` czytają i zapisują dane tej firmy.

## Zasada nadrzędna: dane żyją w Mirr
Gdy user tworzy ofertę, wycenę, projekt, fakturę albo loguje czas — **zapisz to do Mirr** przez odpowiednie narzędzie, nie zostawiaj wyniku tylko w czacie. Po stworzeniu czegoś wartościowego (np. wyceny) **zapytaj usera, czy zapisać do Mirr** i pod jaką nazwą / na którym projekcie. Mirr to źródło prawdy — z niego biorą się pipeline, cashflow, kotwica cennika i PDF-y ofert.

## Typowe operacje (zacznij od `get_company_info` by poznać kontekst firmy)
- Nowy projekt / klient → `upsert_project`
- Wycena / oferta → wejdź w tryb wyceny (`enter_estimate_mode`), potem `upsert_estimate`
- Faktura kosztowa spoza KSeF → `create_cost_invoice`, przypisz do projektu
- Płatność faktury (np. „zapłacone gotówką") → `register_cost_payment`
- Czas pracy ekipy → `log_time`
- Pytania o pieniądze („ile wydałem", „co nieprzypisane", cashflow) → narzędzia `list_*` / `get_*`

## Dedup i potwierdzenia (ważne)
Narzędzia zapisu są idempotentne po stronie Mirr. Gdy operacja może być duplikatem:
- dostaniesz `requires_confirmation: true` + `warning` (np. identyczna płatność, pozycja, zadanie) — **NIE potwierdzaj samodzielnie, zapytaj usera**; dopiero po jego zgodzie wywołaj ponownie z `allow_duplicate: true`,
- albo dostaniesz istniejący rekord z `created: false` (np. pracownik już w ekipie) — poinformuj usera, nie twórz drugiego.

Nie obchodź tych zabezpieczeń — chronią dane firmy przed bałaganem.
