# 1 Podstawowe zapytania
## 1.1 Podstawowa składnia
Bazowa składnia:
- `{}` - wyszukane są wszystkie tracy które spełniają warunki
- `{.service.name = "Ordering.API"}`- filtracja po propercie spanu
- `{duration > 10ms}` co jest równoważne `{ trace:duration > 10ms }`. Pełna lista jest [tu](https://grafana.com/docs/tempo/latest/traceql/#intrinsic-fields) 
- `{ duration > 10ms && .http.method = "GET" && .http.status_code = 200 }` - warunki można też łączyć. Gdzie atrybuty zaczynające się od `.` to ukryte odwołamie do `span.`


Co robi zapytanie:

```
{ .service.name = "BasketAPI" && (duration > 10ms || .http.status_code >= 500 ) }
```
I czemu zwraca takie wyniki?

## 1.2 Typy danych
Są pobierane z [typów Open Telemetry](https://github.com/open-telemetry/opentelemetry-proto/blob/main/opentelemetry/proto/common/v1/common.proto#L31), czyli mamy:
- **String:** efficient case-sensitive matching and regular expressions
- **Numeric:** your basic comparisons: =, !=, >, >=, etc.
- **Duration:** User-friendly representations like “10**ms**” or “15.5**s**,” and comparisons: =, <, <=, etc.
- **Bool:** Booleans can be compared against the built-in keywords **true** and **false**
- **Status:** span status can be compared against the built-in keywords **ok**, **error**, and **unset**

> Uwaga.
> Ponieważ Tempo nie ma schematu danych i nie ma rzutowania typów to:
- `{ .userID = 1000 }` - zwróci wyniki tylko dla intów
- `{ .userID = "1000" }` - zwróci wyniki tylko dla stringów

## 1.3 [Operatory](https://grafana.com/docs/tempo/latest/traceql/#comparison-operators)
Dostępne:
- `=` (equality)
- `!=` (inequality)
- `>` (greater than)
- `>=` (greater than or equal to)
- `<` (less than)
- `<=` (less than or equal to)
- `=~` (regular expression)
- `!~` (negated regular expression)

## 1.4 Scopy
W TraceQL mamy dwa ważne obiekty:
- `resource` - to na czym było wykonywana operacja (baza danych, serwer, etc.)
- `span` - właściwości samej operacji.

Wyszukiwanie po nich może odbywać się następująco
- `span.xxx.yyy` -  konkretny span
- `resource.xxx.yyy` - właściwości resourse
- `xxx.yyy` - przeszuka wszystko
  
Wiedząc, że Tempo [działa na kolumnach](https://grafana.com/blog/2024/01/22/accelerate-traceql-queries-at-scale-with-dedicated-attribute-columns-in-grafana-tempo/) to która operacja będzie wydajniejsza:
- `{ resource.service.name = "api" && span.http.url = "/method" }`
- `{ .service.name = "api" && .http.url = "/method" }`
?

## 1.5 Zapytania między wiele spanów

Możliwe jest określenie następujących relacji:
- `{ .service.name = "BasketAPI" } && { .service.name="Web.Status" }` - znajdzie tracy które zawierają spany z `BasketAPI` i `Web.Status`. Bez określania ich relacji między nimi.
- `{ .service.name = "BasketAPI" } || { .service.name="Web.Status" }` - znajdzie tracy które zawierają spany z `BasketAPI` lub `Web.Status`. Bez określania ich relacji między nimi.
- `{ .service.name="Web.Status" } >> { .service.name = "Identity.API" }` -  `Identity.API` jest dzieckiem `Web.Status`, ale **niekoniecznie** bezpośrednim.
- `{ .service.name="Web.Bff" }  ~ { .service.name="Ordering.API" }` - `Web.Bff` i `Ordering.API` mają tego samego bezpośredniego rodzica

## 1.6 Pipeliny

Co robi kwerenda?

```
{ .service.name = "Web.Bff" } | count() > 5 | avg(duration) > 10ms
```

I mamy też różne [funkcje agregujące](https://grafana.com/docs/tempo/latest/traceql/#aggregators)

# Zadania

## 1.1 Znajdź tracy które zawierały błąd HTTP

Znajdź tracy które gdziekolwiek zawierają span z zapytaniem HTTP które zakończyło się kodem 50x

## 1.2 Znajdź spany które zawierają treść zapytań do bazy danych MS SQL Server

## 1.3 Znajdź tracy które zawierają więcej niż jedno odwołanie do MS SQL Server



