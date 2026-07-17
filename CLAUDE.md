# CLAUDE.md

Wskazówki dla Claude Code przy pracy nad tym repozytorium.

## Czym jest to repo

Publiczny **marketplace pluginów Claude Code** Budowalki. Nie ma kodu aplikacyjnego, builda, testów ani lintu — całość to pliki konfiguracyjne (`.json`) i skille (`SKILL.md`). „Wdrożenie" = commit + push; klienci instalują plugin przez ten marketplace.

Serwer MCP (`mirr`) i jego narzędzia (`upsert_estimate`, `enter_estimate_mode`, `get_company_info`, …) **NIE żyją w tym repo** — to zdalny serwer HTTP pod `https://app.smova.se/mcp` (aplikacja Mirr). Tutaj jest tylko konektor (deklaracja `.mcp.json`) i skille, które uczą Claude, jak z tych narzędzi korzystać.

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

- **Repo jest PUBLICZNE. NIGDY nie commituj tokenów ani sekretów.** `.mcp.json` używa `${MIRR_TOKEN}` — token żyje w **środowisku klienta**, nie w repo. Token = klucz API per firma generowany w Mirr; każdy klient ma własny.
- **Pliki pluginu nie dają dostępu do żadnych danych** — bramką jest wyłącznie token. Kill-switch żyje po stronie Mirr: dezaktywacja klucza API odcina dostęp **natychmiast**, niezależnie od tego, jaką wersję pluginu klient ma zainstalowaną.

## Dev-loop (jak pracować nad pluginem)

1. Edytuj skille / `.mcp.json` w tym repo.
2. **Walidacja:** `claude plugin validate ./plugins/mirr`.
3. **Test lokalny (zmiany widać OD RAZU, bez push):** `MIRR_TOKEN=… claude --plugin-dir <repo>/plugins/mirr`.
   - ⚠ na czas dev `claude plugin disable mirr@mirr-plugins`, żeby zainstalowana z marketplace wersja nie dublowała konektora z `--plugin-dir`.
4. **Wdrożenie:** commit + push → u klienta `claude plugin marketplace update mirr-plugins`. **Bump `version` w `plugin.json`** przy każdej zmianie — bez bumpa klient utknie na cache.

Instalacja pełnym torem klienta: patrz `README.md`.

## Podział odpowiedzialności: serwer vs skille

- Co narzędzie robi + bezpieczeństwo (dedup, walidacje, idempotencja) → **serwer Mirr**. Opisy narzędzi (`description:`) są serwowane z serwera — zmiana propaguje się **instant u wszystkich klientów**, bez ruszania tego repo.
- Procedura „jak orkiestrować narzędzia w workflow" → **skille tutaj**. Nie duplikuj kontraktu narzędzi w skillu — rozjedzie się z serwerem.

## Wzór skilla — gruby na procedurze, cienki na bezpieczeństwie

- Skill = **workflow proceduralny** (kolejność kroków, kiedy dopytać usera, kiedy kotwiczyć w cenniku). To jego właściwa, pełna treść.
- „Cienki" znaczy: **bez logiki bezpieczeństwa** (dedup/walidacje są w serwerze) — NIE „bez procedury".
- **Osobny skill per scenariusz** (`wycena`, `przypisz-fakture`, `raport-cashflow`) — lepsze triggerowanie po `description`, nie jeden monolit.
- Wzór do naśladowania: `skills/wycena/SKILL.md`.
- W `allowed-tools` skilla (jeśli go ograniczasz) używaj pełnych nazw narzędzi z pluginu: `mcp__plugin_mirr_mirr__*` — nie krótkich nazw z serwera in-process.
