---
name: wycena-elewacji
description: Użyj gdy user wycenia ELEWACJĘ / ocieplenie / termomodernizację / ETICS — zbiera wymiary ściana po ścianie, przelicza je normami zmierzonymi na realizacjach i składa pełną wycenę z komponentów cennika. Trigger- "wyceń elewację", "ile za ocieplenie", "kosztorys termomodernizacji", "policz elewację".
---

# Wycena elewacji ETICS

Prowadzisz wycenę ocieplenia budynku. Ogólny przepływ wyceny opisuje skill
`wycena` — tutaj jest to, co specyficzne dla elewacji: **jak zbierać wymiary**
i **jak zamienić je na pozycje**, żeby nie policzyć niczego dwa razy.

## Zbieraj ściana po ścianie, nie działka po działce

Wykonawca chodzi dookoła budynku i spisuje wszystko o jednej ścianie, zanim
przejdzie do następnej. Trzymaj się tego. Formularz zorganizowany „po rodzajach
prac" wymusza skakanie i dlatego nie był używany.

**Najpierw raz dla całego budynku:**
- nowy budynek czy modernizacja (włącza mycie, skucie, demontaże)
- materiał izolacji: styropian biały / grafitowy / wełna
- domyślna grubość izolacji w cm
- liczba kondygnacji (wpływa na rusztowania)

**Potem dla każdej ściany / szczytu / sufitu / słupa:**
- nazwa („ściana frontowa") i wymiary
- grubość izolacji **tylko jeśli inna niż domyślna**
- otwory: szerokość × wysokość, typ parapetu, czy jest listwa przyokienna
- wykończenia jako **lista z powierzchniami** — jedna ściana może mieć tynk na
  40 m² i deskę odciskaną na 12 m²

**Na koniec:** dodatki wymiarowe (LED, bonie, rury spustowe, kratki, skrzynki),
prace towarzyszące i narzut.

## O co NIE pytać

Te rzeczy wynikają z norm albo nie zmieniają ceny. Pytanie o nie zniechęca
usera i nic nie wnosi:

| Nie pytaj | Bo |
|---|---|
| Długość krawędzi, obwód siatkowania, liczba kołków | wynikają z powierzchni przez normy |
| Lambda izolacji, podłoże pod kołki, typ trzpienia, długość kołka | parametry doboru, nie ceny; długość kołka wynika z grubości |
| Liczba transportów, kontenerów, worków, folii | skalują się z powierzchnią |
| Kolor RAL parapetu, boczków, listew | **zapisz, ale nie licz** — patrz niżej |

**Kolory zapisuj zawsze**, gdy user je poda: idą do nazwy pozycji, więc trafiają
na ofertę i do zamówienia materiału. Nie wpływają na koszt — z jednym wyjątkiem:
**baza jasna kontra ciemna tynku** to różnica 4,61 vs 6,43 zł/kg, więc o bazę
zapytaj.

## Geometria

- Powierzchnia netto = brutto − otwory. Ocieplasz ścianę bez okien.
- Ściana szczytowa to **trapez**: (wysokość do okapu + wysokość w kalenicy) / 2
  × szerokość. Nie pytaj o pole, pytaj o dwie wysokości.
- Listwa przyokienna idzie po **trzech bokach** otworu: szerokość + 2 × wysokość.
- Ościeże (glif) = obwód listwy × 0,25 m szerokości ościeża.
- Parapet = szerokość otworu + 0,10 m.
- Obwód cokołu, gdy user go nie zna = suma szerokości ścian.

## Normy zmierzone na realizacjach

Z 12 budów elewacyjnych (4 948 m²) firmy On the wall. Liczba budów mówi, ile
waży norma w sporze — podawaj ją, gdy user kwestionuje pozycję.

| Norma | Wartość | Budów |
|---|---|---|
| Izolacja | 1,55 zł/cm/m² (EPS), 4,77 (wełna) | 12 |
| Zaprawa klejowa | 9,39 kg/m² | 10 |
| Kołki | 3,95 szt/m² | 12 |
| Siatka | 1,76 m²/m² | 9 |
| Tynk baranek | 2,27 kg/m² | 10 |
| Robocizna | 1,28 rbh/m² × 42,78 zł/h | 4 |

Stawka robocizny **obejmuje podwykonawców** — 47,6% wykonawstwa jest kupowane
na zewnątrz, a podwykonawca nie wpisuje godzin do ewidencji. Sama płaca własna
(22,40 zł/h) zaniżyłaby koszt o połowę.

Narzut, który odtwarza realnie fakturowaną cenę: **67%**.

## ⚠ Pułapka: ilość materiału to NIE ilość komponentu

To najczęstszy błąd i zawyża wycenę o kilkadziesiąt procent.

Komponenty w cenniku są wycenione **za m² elewacji** i mają normy już w swoich
składnikach. Do `upsert_estimate` wysyłaj **powierzchnię elewacji**, nie ilość
materiału.

```
✅ Siatkowanie pojedyncze     → 200 m² (powierzchnia netto ściany)
❌ Siatkowanie pojedyncze     → 352 m² (200 × 1,76 normy siatki)
```

Drugi wariant policzy normę dwa razy: 352 × 28,84 zł = 10 152 zł zamiast 5 768.

Rozbicie materiałowe (ile kleju, ile kołków) podawaj userowi **jako
uzasadnienie i podstawę zamówienia**, ale nie jako pozycje wyceny.

## Mapowanie na komponenty

Etapy ETICS rozliczane za m² elewacji netto:

| Zakres | Komponent w cenniku |
|---|---|
| Rusztowania, listwa startowa, gruntowanie, styropian, kołkowanie | `Ocieplenie EPS biały/grafitowy` |
| Siatka, klej 2×, narożniki, obróbka okien | `Siatkowanie pojedyncze` |
| Szlifowanie i podkład pod tynk | `Szlifowanie + grunt pod tynk` |
| Tynk / mozaika / efekt dekoracyjny | wg wybranego wykończenia, **własna powierzchnia** |

Pozycje o własnych jednostkach idą 1:1: parapety (mb), listwa startowa (mb),
LED (mb), bonie (mb), kratki i drzwiczki (szt), kontenery (szt), toi-toi,
transport, rusztowania i prace przygotowawcze (m²).

**Szukaj komponentu po NAZWIE, nie po slugu.** `list_pricing_items` zwraca
`external_id` w formacie `PI-XXXXXXXX` — jest generowany i nie da się go
przewidzieć. Slug nie jest wystawiany. Pobierz cennik, dopasuj po nazwie,
dopiero potem wstaw `source_external_id`.

Nie ma komponentu? Wstaw pozycję wolną (`name` + `quantity` + `unit` +
`unit_price`) i **powiedz userowi, czego brakuje w cenniku** — to sygnał do
uzupełnienia, nie rzecz do przemilczenia.

## Wycena musi wisieć na projekcie

`upsert_estimate` bez projektu zwróci „Validation: Project musi istnieć".
Gdy wyceniasz dla nowego klienta, najpierw `upsert_project` (imię, nazwisko,
adres, `status: new_lead`), potem wycena z `project_external_id`.

## Kontrola przed zapisem

Zanim wywołasz `upsert_estimate`, sprawdź i **zgłoś userowi**, jeśli:

1. **Suma wykończeń nie zgadza się z powierzchnią netto** ściany — brakująca
   powierzchnia zostanie bez tynku, nadmiar policzy się podwójnie.
2. **Ponad 25% kosztu pochodzi z pozycji szacowanych**, nie zmierzonych
   (efekty dekoracyjne, prace przygotowawcze) — powiedz to wprost, zamiast
   podawać wycenę jako pewną.
3. **Koszt wypada poza 95–135 zł/m²** przy typowym ETICS — to sygnał błędu
   w wymiarach albo w jednostkach, nie wyjątkowej budowy.

## Język do usera

Budowlaniec, nie programista. Mów „tej pozycji nie ma w cenniku, dodaję jako
wolną", nie „source_external_id nie rozwiązał się". Podawaj liczbę budów przy
normie, gdy user pyta skąd wartość. Nie pokazuj nazw narzędzi ani pól.
