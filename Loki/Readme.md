# Budowa składni Loki
![alt text](image.png)

# Analizator składni Loki
Grafana wydała [onlinowy analizator składni Loki](https://grafana.com/docs/loki/latest/query/analyzer/) który krok po kroku pokazuje jak przebiega analiza.

# Podstawowe elementy składni LogQL (Loki Query Language):

## Labele
Służą do filtrowania logów. Są to pary klucz-wartość przypisane do każdego wpisu logu.

> ❗Poniższe przykłady są na **generycznych wartościach**. Aby zadziałały w instancji loki będzięcie musieli zmienić je na stosowne. 
> Nie ma kopiuj, przeklej :)

```logql
{app="my-app"}
```

### Operatory na labelach
- `=` - Dokładne matchowanie logów z labelem

    ```logql
    {app="my-app"}
    ```
- Matchowanie wielu labeli. Poniższe zmatchuje logi z **obydwoma** labelami
    ```
    {app="my-app",name="mysql-backup"}
    ```
- `!=` - Dokładne matchowanie logów **bez** labela

    ```logql
    {app !="my-app"}
    ```

- `=~` Matchowanie logów z labeli przy użyciu [regex](https://github.com/google/re2/wiki/Syntax)

    ```logql
    {app =~".*mysql.*"}
    ```
- `!~` Matchowanie logów **bez** labeli przy użyciem [regex](https://github.com/google/re2/wiki/Syntax)

    ```logql
    {app !~".*mysql.*"}
    ```

## Filtracja
Przetwarzania wybranych logów

### Filtracja tekstu
- `|=` - Wyszukuje logi zawierające określony tekst.

    ```
    {app="web-app"} |= "error"
    ```

- `!=` - Wyszukuje logi **NIE** zawierające określony tekst.

    ```
    {app="web-app"} != "error"
    ```

- `|~` - Wyszukuje logi pasujące do wyrażenia regularnego.

    ```
    {app="web-app"} |~ "er.*"
    ```

- `!~` - Wyszukuje logi **NIE** pasujące do wyrażenia regularnego.

    ```
    {app="web-app"} !~ "er.*"
    ```

### Parsed fields

Loki automatyczne parsuje logi i wyciąga z nich wartości. Są wyświetlane w UI w poniższy sposób:

![alt text](parsed_fields.png)

Pola `app` i `namespace` są labelami i można do nich odwoływać się taką składnią:

```
{namespace="default",app="webspa"} 
```

Pole `level` nie jest labelem i użycie go tak samo jak labela **nie** zadziała:

```
{namespace="default",app="webspa", level="error"} 
```

Natomiast zadziała taka składnia:

```
{namespace="default",app="webspa"} | level="error"
```


# Ćwiczenia

## 1. Wybór strumienia
### 1.1 Wybór po namespace
**Cel**: Wyszukaj logi z aplikacji znajdujacych się w Kubernetes namespace `default`.

### 1.2 Podstawowe zapytanie
**Cel**: Wyszukaj logi z aplikacji `webspa` znajdujacych się w Kubernetes namespace `default`.

### 1.3 Regex
**Cel**: Wyszkukaj logi ze wszystkich aplikacji których nazwa kończy się na `api` w namespace `default`


## 2 Wyszukiwanie w tekście
### 2.1 Wyszukiwanie w tekście
**Cel**: Wyszukaj logi z aplikacji `webspa` znajdujacych się w Kubernetes namespace `default` które mają w treści frazę `fail`.

### 2.2 Przeszukiwanie pól

**Cel**: Wyszukaj logi aplikacji znajdujacych się w Kubernetes namespace `default` które mają w treści frazę `fail`, ale **nie** mają statusu `error` dla pola `level`.

### 2.3 Wielokrotne wyszukiwanie

**Cel**: Wyszukaj logi aplikacji znajdujacych się w Kubernetes namespace `default` które mają problemy z połączeniem się po http do serwisu `Identity`

## 3. Agregacje

### Funkcje
Mamy 4 podstawowe funkcje agregujace:
- `rate(log-range)` - liczba wpisów na sekundę
- `count_over_time(log-range)` - zliczanie wystąpień.
- `bytes_rate(log-range)` - liczba bitów na sekundę
- `bytes_over_time(log-range)`-  zliczanie liczby bitów.
- `absent_over_time(log-range)`-  1 jak są wpisy spełniające zapytanie, 0 jak nie ma.

### Czas

Funcje te uruchamiamy w formacie:
```
FUNKCJA_AGREGUJĄCA( KWERENDA [ROZMIAR_WEKTORA_CZASU])
```

Gdzie:
- FUNKCJA_AGREGUJĄCA - wcześniej wymienione funkcje agregujące
- KWERENDA - kwerenda wybierajaca logi jak w paragrafie 1
- ROZMIAR_WEKTORA_CZASU - ile cofamy sie w czasie dla każdej sekundy pomiaru

#### Zrozumieć czas

> ❗Poniższe ćwiczenia wykonuj na grafie w trybie Points:

![alt text](grafana_points.png)

##### 3.1 Czas w funkcjach sumujących

Żeby zrozumieć porównaj działanie 2 kwerend:

Jednej z wektorem 1s
```
bytes_over_time({namespace="default", app="webmvc"} |= "http" [1s])
```
Drugiej z wektorem 1h
```
bytes_over_time({namespace="default", app="webmvc"} |= "http" [1h])
```

Zwróć uwagę na skalę Y wykresu.

##### 3.1 Czas w funkcjach częstotliwości

Teraz porównaj działanie dla funkcji częstotliwości porównując wyniki dla kwerend które mierzą liczbę wystąpień dla każdej sek:

Z zakresem 1s
```
bytes_rate({namespace="default", app="webmvc"} |= "http" [1s])
```

I z zakresem 1h

```
bytes_rate({namespace="default", app="webmvc"} |= "http" [1h])
```

Tym razem zwróć uwagę na to jak gęsto są rozmieszczone punkty pomiarowe na osi X.

### 3.2 Ile aplikacje produkują logów
**Cel**: Porównaj ile linii logów produkuje każdy z serwisów w przestrzeni `otel`

**Tips and tricks:**
- Pamiętaj, że samo użycie `count_over_time` grupuje po strumieniu a nie aplikacji.
- Grupowanie strumieni można osiągnąć przez użycie funkcji `sum` w poniższy sposób:

```
sum ( AGREGACJA ) by (LABEL_GRUPUJACY)
```


## 4. Parsowanie i przetwarzanie

Jeżeli logi są w jednym ze znanych formatów to mogą zostać sparsowane w celu odwoływania się do ich pól.
Funkcje:
- json 
- logfmt
- pattern

### `line_format`
Po przeparsowaniu logów można użyć ich w funkcji `line_format` korzystając z wywołania np: `line_format "{{.NAZWA_PROPERTY}}-jakiś tekst"`.
Gdzie:
- `{{}}` - zaznaczenie wystąpienia zmiennej
- `.NAZWA_PROPERTY` - ścieżka do property

### 4.3 Pattern

**Cel**: Z namespace `otel` dla aplikacji `otel-demo-imageprovider` pobierz jakie adresy IP znajdują się w logach.

Przydatne linki:
- [Opis jak działa składnia](https://grafana.com/blog/2021/08/09/new-in-loki-2.3-logql-pattern-parser-makes-it-easier-to-extract-data-from-unstructured-logs/) 


### 4.1 json
**Cel**: Z namespace `otel` dla aplikacji `otel-demo-imageprovider` pobierz jaki jest czas trwania operacji z logów typu:

```
{
  "message": "Successful to write message. offset: 0, duration: 10.132176ms",
  "severity": "info",
  "timestamp": "2024-06-16T12:39:50.701857639Z"
}
```

**Tips and tricks:**
- Aplikacja `otel-demo-imageprovider` produkuje logi w różnym formacie
- Funckja `json` - parsuje logi w json do formatu klucz-wartość
- Funkcja `keep` - pozwala zostawić tylko określone warości klucz-wartość do dalszego procesowania
- Funkcja `regexp` - pozwala skorzystać z regexp 
  - Strona [regex101.com](https://regex101.com/) ułatwia pisanie regexów
  - Klucz w regexp wygląda następująco: `(?P<NAZWA>REGEX)`
  - W trybie Table lepiej widać sparsowane property i ewentuane błędy
  - ![alt text](fields.png)

### 4.2 Porównywanie
**Cel**: Wyświetl tylko logi z aplikacji `otel-demo-paymentservice` których wartość pola `.amount.units.low` jest większa niż 2000

**Tips and tricks:**
- Loki automatycznie zajmuje się rzutowaniem
- Jak nie znasz składni to pamiętaj, ze UI ma podpowiedzi i możliwość "wyklikania" wyszukiwania poprzez zaznacznie interesujących pól na UI.
![alt text](add_filtering.png) 

### 4.3 Agregacja po własnych polach
**Cel:** Oblicz jaka jest suma wartości `amount.units.low` w aplikacji `otel-demo-paymentservice` względem waluty (pole `amount_currencyCode`).

**Tips and tricks:**
- Wykorzystaj zapytanie w poprzedniego ćwiczenia jako punkt startowy.
- Agregacja po customowej propercie może być zrealizowana przy użyciu funkcji `sum_over_time`
  - Aby określić po której propercie checmy agregować musi ona być używa w wyrażeniu `unwrap` w formie `unwrap PROPERTA`
- Loki umożliwia operacje agregacji na liczbach przez zestaw [wbudowanych funkcji](https://grafana.com/docs/loki/latest/query/metric_queries/#unwrapped-range-aggregations)


### 4.4 Błędy są względne względne
**Cel**: Na jednym wykresie umieść dane:
1. Stwórz wykres pokazujący procent logów z błędami do wszystkich logów dla top 3 aplikacji w namespace `otel`
2. Nałóż na to ten sam wykres co poprzednio, ale przesunięty o 24h

Dla wszystkich wykresów legenda powinna pokazywa nazwę aplikacji. Z tym, że dla aplikacji z offset powinna mieć postfix `-offset`

**Tips and tricks:**
- Funkcja `topk`
- Loki ma automatyczną konwersję do floatów
- Jeden wykres może posiadać wiele kwerend:
![alt text](add_query.png) 


### 4.5 Ile jakich logów
**Cel**:
Stwórz W dashboard `Otel` znajduje się wykres `SERWIS Log entries by Severity`.
Uzupełnij go tak, żeby pokazywał w formie tabeli informacje ile logów jakiego typu serwis produkuje. Czyli coś takiego (bez dodatkowych kwerend):

| Log level | Total |
|---        |---    |
|Error      |30     |
|Info       |351    |


Tips and tricks:
- **Zrób własną kopię tego dashboarda**
- Zwróć uwagę co jest w sekcjach na UI:
  - `Options`
  - `Transform data`
- Użycie zmiennej w kwerendzie odbywa się przez składnię: `${NAZWA_ZMIENNEJ}`
- Wbudowana w Grafanę zmienna odnosząca się do właśnie wybranego okresu czasowego nazywa się `$__range`
