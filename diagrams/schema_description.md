
# Projekt hurtowni danych — Notino

## Model danych
Schemat gwiazdy (Star Schema)

## Procesy biznesowe
1. Sprzedaż produktów kosmetycznych
2. Zwroty produktów

## Tabela dim_product

Klucz główny: product_key

Atrybuty:
- product_key — klucz zastępczy
- product_id — identyfikator biznesowy
- product_name — nazwa produktu
- brand — marka produktu
- category — kategoria
- subcategory — podkategoria
- is_active — status produktu

Relacja:
dim_product.product_key (1)
    -> fact_sales.product_key (N)

## Tabela dim_customer

Klucz główny: customer_key

Atrybuty:
- customer_key — klucz zastępczy
- customer_id — identyfikator biznesowy klienta
- country — kraj zamieszkania
- city — miasto
- age_group — grupa wiekowa
- loyalty_status — status programu lojalnościowego
- registration_date — data rejestracji

Relacja:
dim_customer.customer_key (1)
    -> fact_sales.customer_key (N)

Przykładowe grupy wiekowe:
18-24, 25-34, 35-44, 45-54, 55+

Przykładowe statusy lojalnościowe:
Standard, Silver, Gold

## Tabela dim_date

Klucz główny: date_key

Atrybuty:
- date_key — klucz zastępczy daty (format YYYYMMDD)
- full_date — pełna data
- day — dzień miesiąca
- month — numer miesiąca
- month_name — nazwa miesiąca
- quarter — numer kwartału
- year — rok
- day_of_week — numer dnia tygodnia (1 = poniedziałek)
- is_weekend — informacja, czy dzień przypada w weekend

Relacje:
dim_date.date_key (1)
    -> fact_sales.date_key (N)

dim_date.date_key (1)
    -> fact_returns.return_date_key (N)

Zakres dat:
2024-01-01 — 2026-10-31

## Tabela dim_location

Klucz główny: location_key

Atrybuty:
- location_key — klucz zastępczy lokalizacji
- store_id — identyfikator punktu sprzedaży
- store_name — nazwa sklepu lub rynku online
- country — kraj sprzedaży
- city — miasto
- region — region
- store_type — typ lokalizacji (stationary / online)

Relacja:
dim_location.location_key (1)
    -> fact_sales.location_key (N)

Zakres geograficzny:
- Polska
- Czechy
- Słowacja
- Węgry

Założenia:
- Sprzedaż stacjonarna jest przypisana do konkretnego sklepu.
- Sprzedaż internetowa jest przypisana do rynku online.
- Dla rynku online pola store_id, city i region
  mogą przyjmować wartość NULL.
- Wszystkie lokalizacje są danymi syntetycznymi.

## Tabela dim_channel

Klucz główny: channel_key

Atrybuty:
- channel_key — klucz zastępczy kanału sprzedaży
- channel_name — nazwa kanału sprzedaży

Przykładowe wartości:
1 — Online
2 — Stationary

Relacja:
dim_channel.channel_key (1)
    -> fact_sales.channel_key (N)

Założenia:
- Każda pozycja sprzedaży jest przypisana
  do jednego kanału sprzedaży.
- Kanał Online oznacza zakup internetowy.
- Kanał Stationary oznacza zakup stacjonarny.
- Lokalizacja sprzedaży jest przechowywana
  osobno w tabeli dim_location.

## Tabela dim_payment

Klucz główny: payment_key

Atrybuty:
- payment_key — klucz zastępczy metody płatności
- payment_method — nazwa metody płatności
- payment_type — rodzaj płatności

Przykładowe metody płatności:
1 — Karta płatnicza — Bezgotówkowa
2 — BLIK — Bezgotówkowa
3 — Przelew online — Bezgotówkowa
4 — Gotówka — Gotówkowa
5 — Płatność przy odbiorze — Przy odbiorze

Relacja:
dim_payment.payment_key (1)
    -> fact_sales.payment_key (N)

Założenia:
- Każde zamówienie ma jedną metodę płatności.
- Wszystkie pozycje jednego zamówienia mają
  tę samą wartość payment_key.
- Dostępność metod płatności zależy od kraju
  i kanału sprzedaży.
- Metody płatności są przypisywane podczas
  generowania danych syntetycznych.

## Tabela dim_promotion

Klucz główny: promotion_key

Atrybuty:
- promotion_key — klucz zastępczy promocji
- promotion_name — nazwa promocji
- promotion_type — rodzaj promocji
- discount_rate — wysokość rabatu w procentach
- start_date — data rozpoczęcia promocji
- end_date — data zakończenia promocji

Przykładowe promocje:
1 — Brak promocji — 0%
2 — Black Friday — 25%
3 — Summer Sale — 15%
4 — Rabat lojalnościowy — 10%
5 — Kod rabatowy — 20%

Relacja:
dim_promotion.promotion_key (1)
    -> fact_sales.promotion_key (N)

Założenia:
- Każda pozycja zamówienia ma przypisaną
  jedną promocję lub wartość "Brak promocji".
- Jedna promocja może dotyczyć wielu transakcji.
- Rabaty są uwzględniane podczas obliczania
  wartości sprzedaży netto.
- Promocja może zostać przypisana wyłącznie
  w okresie jej obowiązywania.
- Promocje i rabaty są danymi syntetycznymi.

## Tabela fact_sales

Typ: tabela faktów
Proces biznesowy: sprzedaż produktów
Ziarno: jedna pozycja zamówienia

Klucz główny:
- sales_key — identyfikator pozycji w hurtowni

Identyfikatory biznesowe:
- order_id — identyfikator zamówienia
- order_item_id — identyfikator pozycji zamówienia

Klucze obce:
- date_key -> dim_date.date_key
- customer_key -> dim_customer.customer_key
- product_key -> dim_product.product_key
- location_key -> dim_location.location_key
- channel_key -> dim_channel.channel_key
- payment_key -> dim_payment.payment_key
- promotion_key -> dim_promotion.promotion_key

Miary:
- quantity INTEGER — liczba sztuk
- unit_price NUMERIC(12,2) — cena jednostkowa
- gross_amount NUMERIC(12,2) — wartość przed rabatem
- discount_amount NUMERIC(12,2) — kwota rabatu
- net_amount NUMERIC(12,2) — wartość po rabacie
- cost_amount NUMERIC(12,2) — koszt produktów
- margin_amount NUMERIC(12,2) — marża kwotowa

Założenia:
- Jeden rekord reprezentuje jedną pozycję zamówienia.
- Jedno zamówienie może zawierać wiele produktów.
- Wszystkie pozycje jednego zamówienia mają
  ten sam identyfikator order_id.
- Każda pozycja ma dodatnią ilość i cenę.
- Kwoty są przechowywane w PLN.
- Zakupy anonimowe są przypisane do rekordu
  "Nieznany klient" w dim_customer.
- Dla jednej pozycji obowiązuje maksymalnie
  jedna promocja.

Zasady obliczeń:
gross_amount = quantity * unit_price

discount_amount =
    gross_amount * discount_rate / 100

net_amount = gross_amount - discount_amount

cost_amount = quantity * unit_cost

margin_amount = net_amount - cost_amount

Uwaga:
unit_cost jest wartością źródłową wykorzystywaną
podczas generowania danych i ETL.
W hurtowni przechowujemy zagregowany koszt
pozycji sprzedaży jako cost_amount.

## Tabela fact_returns

Typ: tabela faktów
Proces biznesowy: zwroty produktów
Ziarno: jedno zdarzenie zwrotu dotyczące
konkretnej pozycji zamówienia

Klucz główny:
- return_key BIGINT — identyfikator zwrotu

Klucze obce:
- sales_key BIGINT -> fact_sales.sales_key
- return_date_key INTEGER -> dim_date.date_key

Atrybuty:
- return_quantity INTEGER — liczba zwróconych sztuk
- return_amount NUMERIC(12,2) — wartość zwrotu
- return_reason VARCHAR(100) — powód zwrotu

Przykładowe powody zwrotu:
- Rezygnacja z zakupu
- Uszkodzone opakowanie
- Niewłaściwy produkt
- Produkt niezgodny z opisem
- Inny powód

Założenia:
- Każdy zwrot musi odnosić się do istniejącej
  pozycji sprzedaży.
- Data zwrotu nie może być wcześniejsza
  od daty zakupu.
- Liczba zwróconych sztuk musi być dodatnia.
- Suma zwrotów dla jednej pozycji nie może
  przekroczyć pierwotnie sprzedanej ilości.
- Wartość zwrotu nie może przekroczyć
  wartości sprzedaży danej pozycji.
- Wszystkie kwoty są przechowywane w PLN.
- Brak zwrotu oznacza brak rekordu
  w tabeli fact_returns.

Relacje:
fact_sales.sales_key (1)
    -> fact_returns.sales_key (N)

dim_date.date_key (1)
    -> fact_returns.return_date_key (N)
