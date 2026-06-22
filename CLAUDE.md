# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Czym jest to repo

Prywatny **marketplace pluginów Claude Code** dla Budowalki. Nie ma kodu aplikacyjnego, builda, testów ani lintu — całość to pliki konfiguracyjne (`.json`) i skille (`SKILL.md`). „Wdrożenie" = commit + push; klienci instalują plugin przez ten marketplace.

Serwer MCP (`mirr`) i jego narzędzia (`upsert_estimate`, `enter_estimate_mode`, `get_company_info`, …) **NIE żyją w tym repo** — to zdalny serwer HTTP pod `https://app.smova.se/mcp` (aplikacja Mirr / Smova). Tutaj jest tylko konektor (deklaracja `.mcp.json`) i skille, które uczą Claude jak z tych narzędzi korzystać.

## Struktura

```
.claude-plugin/marketplace.json   # rejestr marketplace → wskazuje na plugins/*
plugins/mirr/
  .claude-plugin/plugin.json      # manifest pluginu (name, version, author)
  .mcp.json                       # konektor MCP http → app.smova.se/mcp, Bearer ${MIRR_TOKEN}
  skills/<nazwa>/SKILL.md         # skille z frontmatterem name + description
```

Dodanie nowego skilla: utwórz `plugins/mirr/skills/<nazwa>/SKILL.md` z frontmatterem `name` + `description` (trigger). Nie ma osobnego rejestru skilli — są wykrywane z katalogu. Bumpuj `version` w `plugin.json` przy zmianach kontraktu.

## Klucze architektoniczne (czytaj przed zmianą skilli)

- **Mirr jest źródłem prawdy.** Skille forsują zapis ustrukturyzowanych danych z powrotem do Mirr przez narzędzia MCP, zamiast trzymać wynik tylko w czacie. Z Mirr biorą się pipeline, cashflow, kotwica cennika, PDF-y ofert.
- **`upsert_estimate` ZASTĘPUJE całą wycenę** — wysłanie `categories` kasuje dotychczasowe pozycje. Żeby dodać/zmienić jedną: `get_estimate` → zbuduj pełną listę → `upsert_estimate`. To najczęstsza pułapka; skill `wycena` opisuje ją szczegółowo.
- **Anti-konfabulacja jest twarda.** Nie ogłaszaj zapisu bez udanego wywołania narzędzia w tej turze; nie renderuj tabeli wyceny „z głowy" — raportuj wyłącznie to, co zwróciło narzędzie. Ceny kotwicz w cenniku (`list_pricing_items`, `list_resources`), nie zgaduj.
- **Dedup po stronie Mirr.** Narzędzia zapisu są idempotentne. `requires_confirmation: true` → zapytaj usera, dopiero potem `allow_duplicate: true`. `created: false` → rekord już istnieje, nie twórz drugiego. Nie obchodź tych zabezpieczeń.

## Język i odbiorca

Końcowy user skilla to **właściciel firmy budowlanej, nie programista**. W treści skierowanej do usera nie pokazuj nazw narzędzi (`upsert_estimate`), pól (`source_external_id`) ani surowych błędów — tłumacz na ludzki. Wszystkie skille i teksty po polsku z polskimi znakami.

## 🔒 Sekrety i bezpieczeństwo (czytaj zanim cokolwiek commitujesz)

- **NIGDY nie commituj tokenów ani sekretów.** `.mcp.json` używa `${MIRR_TOKEN}` — token jest w **env klienta**, nie w repo. Token = `ApiKey` per firma (scope `mcp`) generowany w Mirr; każdy klient ma własny.
- **Repo jest PRYWATNE** — dystrybucja przez GitHub collaborator (Read). Dodanie klienta = dodanie collaboratora.
- **Kill-switch danych** żyje po stronie Mirr: `ApiKey.active = false` odcina dostęp **natychmiast**, niezależnie od tego, jaką wersję pluginu klient ma w cache. Usunięcie collaboratora blokuje tylko przyszłe `marketplace update`, NIE odcina już zainstalowanej kopii — realny kill-switch to token.
- Token demo do własnego dogfoodu (NIE klient): `ApiKey` „claude-konektor-demo" na koncie `demo`.

## Dev-loop (jak pracować nad pluginem)

1. Edytuj skille / `.mcp.json` w tym repo.
2. **Walidacja:** `claude plugin validate ./plugins/mirr`.
3. **Test lokalny (zmiany widać OD RAZU, bez push):** `MIRR_TOKEN=… claude --plugin-dir ~/dev/mirr-plugins/plugins/mirr`.
   - ⚠ na czas dev `claude plugin disable mirr@mirr-plugins`, żeby zainstalowana z marketplace wersja nie dublowała konektora z `--plugin-dir`.
4. **Wdrożenie:** commit + push → u klienta `/plugin marketplace update mirr-plugins`. **Bump `version` w `plugin.json`** przy każdej zmianie — bez bumpa klient utknie na cache.

Instalacja jak klient (do testu pełnego toru): `claude plugin marketplace add Budowalka/mirr-plugins` → `claude plugin install mirr@mirr-plugins` (prywatne repo, przez `gh auth`).

## Powiązanie ze smova-3 (gdzie co edytować)

- Serwer MCP i narzędzia: **`~/dev/smova-3/lib/mcp/`** (`server.rb`, `tools/*.rb`). Opisy narzędzi (`description:`) to „mózg" — serwowane z Railsa, więc zmiana propaguje się **instant u wszystkich klientów po deployu**, bez ruszania tego repo.
- **Podział odpowiedzialności:** co narzędzie robi + bezpieczeństwo (dedup, walidacje, idempotencja) → **smova-3** (serwer). Procedura „jak orkiestrować narzędzia w workflow" → **skille tutaj**. Nie duplikuj kontraktu narzędzi w skillu — rozjedzie się z serwerem.
- Idempotencja zapisu (dedup) jest już wdrożona w smova-3 — spec/plany `docs/superpowers/{specs,plans}/2026-06-20-mcp-konektor-idempotencja-*`. Handoff całości: `smova-3/docs/superpowers/plans/CONTINUATION.md`, sekcja „🔌 Zewnętrzny dostęp do MCP".

## Wzór skilla — gruby na procedurze, cienki na bezpieczeństwie

- Skill = **workflow proceduralny** (kolejność kroków, kiedy dopytać usera, kiedy kotwiczyć w cenniku). To jego właściwa, pełna treść.
- „Cienki" znaczy: **bez logiki bezpieczeństwa** (dedup/walidacje są w serwerze) — NIE „bez procedury".
- **Osobny skill per scenariusz** (`wycena`, `przypisz-fakture`, `raport-cashflow`) — lepsze triggerowanie po `description`, nie jeden monolit.
- Wzór do naśladowania: `skills/wycena/SKILL.md` (esencja z `smova-3/skills/shared/estimating/wycena-z-komponentow`).
- W `allowed-tools` skilla (jeśli go ograniczasz) używaj pełnych nazw narzędzi z pluginu: `mcp__plugin_mirr_mirr__*` — nie krótkich nazw z serwera in-process.
