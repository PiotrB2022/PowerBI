# Financial Report: mapa modelu semantycznego

Projekt PBIP z modelem finansowym (księga główna 2018-2020, 7 krajów). Stan opisu: 2026-11-18.

## Jak korzystać z tego pliku

- Do orientacji w modelu używaj tego pliku. Nie odpytuj modelu i nie czytaj całego TMDL, jeśli odpowiedź jest tutaj.
- Źródłem prawdy jest TMDL w `Financial Report.SemanticModel/definition/`. Przed zmianą istniejącego obiektu odczytaj tylko ten obiekt.
- **Każda zmiana modelu wymaga aktualizacji tego pliku w tym samym zadaniu**, razem z datą stanu opisu.
- Jeśli opis różni się od TMDL, obowiązuje TMDL. Popraw wtedy ten plik.
- Do edycji używaj MCP `powerbi-modeling-local`, gdy jest połączony z modelem. W przeciwnym razie edytuj TMDL.

## Źródło danych

- Tryb Import. Wszystkie tabele czytają skoroszyt wskazany parametrem M `Source File Path`.
- Plik: `C:\Users\barej\Desktop\Claude Space\Power BI\Finacial_Data.xlsx`. Literówka `Finacial` jest w prawdziwej nazwie pliku.
- Po otwarciu projektu bez zapisanej pamięci podręcznej tabele mogą być puste. Trzeba wtedy odświeżyć dane.

| Tabela modelu | Tabela Excel | Wiersze | Ziarno |
|---|---|---:|---|
| `General Ledger` | `tbl_GL` | 27 909 | jedno księgowanie |
| `Account` | `tbl_ChartofAccounts` | 54 | jedno konto |
| `Territory` | `tbl_territory` | 7 | jeden kraj |
| `Date` | `tbl_Calendar` | 1 096 | jeden dzień, 2018-01-01 do 2020-12-31 |
| `Cash Flow Statement` | `tbl_CF_St` | 66 | wiersz układu i konto |
| `Equity Statement` | `tbl_SoCE_St` | 13 | ruch i składnik kapitału |

Tabele kalkulowane (DAX `DATATABLE`, bez źródła w Excelu) opisują układ sprawozdań w raporcie:

| Tabela modelu | Wiersze | Ziarno |
|---|---:|---|
| `Income Statement Line` | 12 | wiersz rachunku wyników |
| `Balance Sheet Line` | 10 | wiersz bilansu |
| `Profit Bridge Step` | 8 | krok mostu zysku netto |

Import `General Ledger` pomija wiersz sumy z arkusza (kwota bez daty) oraz kolumnę `Index`, która nie jest unikalna.

## Tabele i kolumny

`(u)` oznacza kolumnę ukrytą, `(PK)` klucz główny.

- **General Ledger:** `Date` (u), `Territory Key` (u), `Account Key` (u), `Transaction Type`, `Amount` (u)
- **Account:** `Account Key` (u, PK), `Report`, `Class`, `Subclass`, `Subclass Detail`, `Account`, `Subaccount`
- **Territory:** `Territory Key` (u, PK), `Country`, `Region`
- **Date:** `Date` (PK), `Year`, `Quarter`, `Month`, `Month Number` (u), `Year Month`, `Year Month Number` (u), `Day of Week`, `Day of Week Number` (u)
- **Cash Flow Statement:** `Cash Flow Section`, `Cash Flow Line`, `Cash Flow Account`, `Cash Flow Subaccount`, `Value Type` (u), `Account Key` (u), `Section Order` (u), `Line Order` (u)
- **Equity Statement:** `Equity Movement`, `Equity Component`, `Balance Type` (u), `Account Key` (u), `Movement Order` (u), `Component Order` (u)
- **Income Statement Line:** `Line`, `Line Order` (u), `Is Subtotal` (u)
- **Balance Sheet Line:** `Line`, `Line Order` (u), `Is Subtotal` (u)
- **Profit Bridge Step:** `Step`, `Step Order` (u)

Sortowanie: `Month` po `Month Number`, `Year Month` po `Year Month Number`, `Day of Week` po `Day of Week Number`, `Cash Flow Section` po `Section Order`, `Cash Flow Line` po `Line Order`, `Equity Movement` po `Movement Order`, `Equity Component` po `Component Order`, `Line` po `Line Order` (w `Income Statement Line` i `Balance Sheet Line`), `Step` po `Step Order`.

Hierarchie:
- `Calendar Hierarchy`: Year > Quarter > Month > Date
- `Account Hierarchy`: Report > Class > Subclass > Subclass Detail > Account > Subaccount
- `Territory Hierarchy`: Region > Country

## Relacje

Wszystkie są typu wiele-do-jednego, z filtrowaniem w jednym kierunku:

- `General Ledger[Date]` > `Date[Date]`
- `General Ledger[Account Key]` > `Account[Account Key]`
- `General Ledger[Territory Key]` > `Territory[Territory Key]`

`Cash Flow Statement` i `Equity Statement` **celowo nie mają relacji**. Miary przenoszą klucz konta przez `TREATAS`. Nie dodawaj tu relacji.

`Income Statement Line`, `Balance Sheet Line` i `Profit Bridge Step` też **celowo nie mają relacji**. Ich miary wybierają wiersz przez `SWITCH` po kolumnie kolejności.

## Miary (106, w tym 29 ukrytych)

Formaty: kwoty `#,##0`, procenty `0.00%`, wskaźniki `#,##0.00`. Bez symbolu waluty, bo źródło jej nie podaje.

**General Ledger, folder Ledger**
- `Ledger Amount`: suma `Amount`, miara bazowa
- `# Ledger Entries`: liczba księgowań

**General Ledger, folder Profit and Loss**
- `Net Sales`: Subclass = Sales
- `Cost of Sales`: Subclass = Cost of Sales
- `Gross Profit`: Class = Trading account
- `Operating Expenses`: Subclass = Operating Expenses
- `Depreciation and Amortization`: Subclass = Depreciation & Amortization
- `Operating Profit`: Class = Trading account lub Operating account
- `EBITDA`: `Operating Profit` - `Depreciation and Amortization`
- `Non-Operating Income`: Class = Non-operating
- `Interest Expense`: Subclass = Interest Expense
- `Tax Expense`: Subclass = Taxation
- `Net Profit`: Report = Profit and Loss
- `Profit Before Tax`: `Net Profit` - `Tax Expense`

**General Ledger, folder Profit and Loss\Margins** (udział w `Net Sales`)
- `Gross Margin (%)`, `EBITDA Margin (%)`, `Operating Margin (%)`, `Net Margin (%)`

**General Ledger, folder Time Intelligence**
- `Net Sales (ly)`, `Net Sales (ytd)`, `Net Sales Growth (%)`
- `Net Profit (ly)`, `Net Profit (ytd)`, `Net Profit Growth (%)`
- `EBITDA (ly)`, `EBITDA Growth (%)`
- `Operating Expenses (ly)`, `Operating Expenses Growth (%)`, `Operating Expenses Increase`: wzrost liczony od wielkości kosztu, więc wartość dodatnia oznacza wyższe koszty
- `Net Margin (ly)`, `Net Margin Change (pp)`: zmiana w punktach procentowych
- `Cash Balance (ly)`, `Cash Balance Growth (%)`
- `Total Assets (ly)`, `Total Assets Growth (%)`, `Total Liabilities (ly)`, `Total Liabilities Growth (%)`, `Total Equity (ly)`, `Total Equity Growth (%)`
- Wzrost liczony jest jako (bieżący - poprzedni) / ABS(poprzedni). Dla 2018 wynik jest pusty.

**General Ledger, folder Report Labels** (wszystkie ukryte, tylko do raportu)
- Pomocnicze: `Comparison Label`, `Scope Label`, `Period Label`
- Tytuły stron: `Overview Title`, `Profit and Loss Title`, `Countries Title`, `Balance Sheet Title`, `Cash Flow Title` oraz odpowiadające im `... Subtitle`
- Teksty zmian na kartach KPI (`... Change Label`): `Net Sales`, `EBITDA`, `Net Profit`, `Net Margin`, `Cash Balance`, `Total Assets`, `Total Liabilities`, `Total Equity`, `Operating Cash Flow`, `Investing Cash Flow`, `Financing Cash Flow`, `Net Cash Flow`
- Kolory: `Net Profit Color`, `Operating Expenses Increase Color`

**General Ledger, folder Balance Sheet** (saldo narastająco do ostatniej daty w kontekście)
- `Closing Balance`: miara bazowa sald
- `Total Assets`: Class = Assets
- `Current Assets`: Subclass Detail = Current Assets
- `Cash Balance`: Account = Cash & Cash Equivalents
- `Total Liabilities`: Subclass = Liabilities
- `Current Liabilities`: Subclass Detail = Current Liabilities
- `Total Equity`: Subclass = Owners Equity
- `Total Liabilities and Equity`: zobowiązania + kapitał
- `Balance Check`: aktywa - (zobowiązania + kapitał)

**General Ledger, folder Balance Sheet\Ratios**
- `Working Capital`, `Current Ratio`, `Liabilities to Equity Ratio`

**Cash Flow Statement**
- `Cash Flow Amount`: miara główna sprawozdania
- `Operating Cash Flow`, `Investing Cash Flow`, `Financing Cash Flow`, `Net Cash Flow`
- `Working Capital Changes`: wiersz `Working Capital Changes` ze sprawozdania
- Folder Time Intelligence: `Cash Flow Amount (ly)`, `Operating Cash Flow (ly)`, `Operating Cash Flow Growth (%)`, `Investing Cash Flow (ly)`, `Investing Outflow Growth (%)`, `Financing Cash Flow (ly)`, `Financing Cash Flow Growth (%)`, `Net Cash Flow (ly)`, `Net Cash Flow Change`
- `Cash at Start of Period`, `Cash at End of Period`

**Equity Statement**
- `Equity Statement Amount`: miara główna sprawozdania

**Income Statement Line**
- `Income Statement Amount`, `Income Statement Amount (ly)`, `Income Statement Share of Sales (%)`
- `Income Statement Change (%)`: dla wierszy kosztowych porównuje wielkość kosztu
- `Income Statement Row Color` (ukryta): tło wierszy z sumami

**Balance Sheet Line**
- `Balance Sheet Amount`, `Balance Sheet Amount (ly)`, `Balance Sheet Change (%)`
- `Balance Sheet Row Color` (ukryta): tło wierszy z sumami

**Profit Bridge Step**
- `Profit Bridge Amount`: zysk netto roku poprzedniego i zmiany grup rachunku wyników. Suma kroków równa się `Net Profit` wybranego okresu.

## Reguły, które łatwo pomylić

**Znaki w księdze**
- Przychody są dodatnie, koszty ujemne. `Cost of Sales`, `Operating Expenses` i `Tax Expense` zwracają wartości ujemne, więc w wyniku się je dodaje.
- Wzrost aktywów, zobowiązań i kapitału jest dodatni.
- `Ledger Amount` ma sens tylko w ramach jednego raportu. Suma całej księgi (24 640 002) nie ma znaczenia biznesowego.
- Konto 1010 (Report = Adjusting) jest drugą stroną przeniesienia zysku do `Retained Earnings` i nie należy do rachunku zysków i strat.

**Kody `Value Type` i `Balance Type`**

| Kod | Znaczenie |
|---|---|
| `All_FTP`, `FTP` | ruch w wybranym okresie |
| `All_FTP_CS` | ruch w okresie ze zmienionym znakiem |
| `All_FTP_Positive`, `FTP_positive` | tylko księgowania dodatnie |
| `All_FTP_Positive_CS` | tylko dodatnie, ze zmienionym znakiem |
| `All_FTP_Negative`, `FTP_negative` | tylko księgowania ujemne |
| `All_FTP_Negative_CS` | tylko ujemne, ze zmienionym znakiem |
| `Opening_balance` | saldo przed pierwszym dniem okresu |
| `Closing_balance` | saldo na ostatni dzień okresu |
| `ToBeDefined` | brak reguły, wynik pusty |

**Sumy w sprawozdaniach:** gdy w kontekście jest kilka sekcji lub ruchów, `Cash Flow Amount` i `Equity Statement Amount` pomijają saldo końcowe. Suma końcowa równa się wtedy saldu zamknięcia.

**Dokładne teksty w filtrach:** sekcja finansowa nazywa się `Cashflows from Financing Activities` (pisownia łączna), a operacyjna `Cash flows from Operating Activities`.

**`Balance Check`** wynosi 0 na koniec każdego miesiąca. Zysk przenoszony jest do kapitału raz w miesiącu, więc nie używaj tej miary na poziomie dnia.

## Wartości kontrolne

Zweryfikowane 2026-11-18 względem niezależnych sum z pliku Excel. Użyj ich do testów regresji po zmianach.

| Miara | 2018 | 2019 | 2020 |
|---|---:|---:|---:|
| `Net Sales` | 3 575 428 | 5 697 845 | 7 835 369 |
| `Net Profit` | 623 856 | 1 303 147 | 1 289 945 |
| `Operating Cash Flow` | 762 428 | 1 387 366 | 821 928 |
| `Investing Cash Flow` | -1 483 404 | -2 202 193 | -3 052 367 |
| `Financing Cash Flow` | 2 828 667 | 3 818 150 | 1 630 157 |
| `Cash Balance` | 2 107 691 | 5 111 014 | 4 510 732 |
| `Total Assets` | 3 875 802 | 9 238 096 | 12 320 001 |
| `Total Liabilities` | 844 946 | 1 427 893 | 1 971 645 |
| `Total Equity` | 3 030 856 | 7 810 203 | 10 348 356 |

Cały okres: `Net Sales` 17 108 642, `Gross Profit` 11 693 152, `EBITDA` 5 392 555, `Operating Profit` 3 738 655, `Profit Before Tax` 3 831 715, `Net Profit` 3 216 948.

## Znane problemy w danych źródłowych

- **Bad Debts:** 462 księgowania trafiają na konto Commissions (260) i zwiększają gotówkę w kasie o 126 141. Prawdopodobny błąd w źródle, pozostawiony bez zmian.
- **Changes in accounting policy:** wiersz w `Equity Statement` ma kod `ToBeDefined`, więc jest pusty.
- **Konta bez księgowań:** 16 kont z planu nie ma zapisów w księdze: 40, 41, 42, 50, 70, 80, 130, 140, 150, 170, 190, 250, 270, 360, 370, 390.
