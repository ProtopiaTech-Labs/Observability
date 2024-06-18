# 1. Podstawowa składnia
## 1.1 Poberanie bucketu

```
http_client_request_duration_seconds_sum
```

## 1.2 Zawężenie metryk po labelu

```
http_client_request_duration_seconds_sum{app="webmvc"}
```

## 1.3 Zawężenie metryk po labelach

Przez bezpośrednie porównanie:
```
http_client_request_duration_seconds_sum{app="webmvc", http_response_status_code="200"}
```

Negację:
```
http_client_request_duration_seconds_sum{app="webmvc", http_response_status_code!="200"}
```

Regex:
```
http_client_request_duration_seconds_sum{app="webmvc", http_response_status_code=~"20.*"}
```

## 1.4 Same labele

Podanie bucketa nie jest konieczne. Można np tak:

```
{ app="webmvc" }
```

Z dalszym zawężaniem:

```
{ app="webmvc", http_status_code=~"20.*" }
```

Możemy też odwołać się do bucketa przez składnię labeli:

```
{__name__="http_client_request_duration_seconds_sum"}
```

Dzięki temu możemy użyć regexów w wyszukiwaniu bucketów:

```
{__name__=~"http_client_request_duration_seconds_.*"}
```

### 1.5 Czas i dokładność

> Uwaga poniższe kwerendy odpalamy w [UI Prometeusza]()

Wartość za ostatnie 30 minut
```
http_client_request_duration_seconds_sum{app="webmvc", error_type="403"}[30m]
```

Wartości za ostatnie 30 minut z rozdzielczością 15 minut:
```
http_client_request_duration_seconds_sum{app="webmvc", error_type="403"}[30m:15m]
```

### 1.6 `rate` i `increment`

#### 1.6.1 rate i increment vs oryginalne wartości

Uruchom na jednym wykresie (w razie potrzeby znormalizuj mnożąc przes odpowiednie wartości) kwerendy:

**Oryginalne wartości:**
```
http_client_request_duration_seconds_sum{app="webmvc", error_type="403"}
```

**Increase:**
```
increase (http_client_request_duration_seconds_sum{app="webmvc", error_type="403"}[2m])
```

**Rate:**
```
rate(http_client_request_duration_seconds_sum{app="webmvc", error_type="403"}[2m])
```

To-do, tips and tricks:
- Do analizy jak działają powyższe przydaje się zmiana formatu na `Table` dostępny w Options dla kwerendy.
- Zobacz co się stanie z wartościami jak zmienisz `Min step` na różne wartości
- Bardzo dobre tłumaczenie działania rate jest pod tym wpisem na [StackOverflow](https://stackoverflow.com/questions/54494394/do-i-understand-prometheuss-rate-vs-increase-functions-correctly)

#### 1.6.2 Interval

Uruchom kwerendy:

```
rate(http_client_request_duration_seconds_sum{app="webmvc", error_type="403"} [2m])
```

i

```
rate(http_client_request_duration_seconds_sum{app="webmvc", error_type="403"} [5s])
```

Czemu jedna ma a druga nie ma wyników?

Tips and tricks:
- [artykuł o wyborze interval](https://www.robustperception.io/what-range-should-i-use-with-rate/)
- Wprowadzenie [`$__rate_interval`](https://grafana.com/blog/2020/09/28/new-in-grafana-7.2-__rate_interval-for-prometheus-rate-queries-that-just-work/) przez Grafanę

### 1.7 Historgram

Uruchom zapytanie:

```
http_client_duration_milliseconds_bucket{ http_status_code="200",net_peer_name="otel-demo-emailservice" }
```

I zaobserwuj:
- Czym się od siebie różnią buckety?
- Jak układają się wartości?


## 2. Zadania

### 2.1 Ile jest GB wykorzystanych na nodach Kubernetes

Stwórz wykres pokazujący ile jest zajętej pamięci na każdym nodzie w klastrze Kubernetes.

**To-do i Tips and tricks:**
- Metryki są w `node_memory_....`.
- Wyniki powinny być w GB.
- Legenda to nazwa noda Kubernetes.


### 2.2 Najbardziej pamięciożerne kontenery
Stwórz wykres pokazujące najbardziej pamięciożerne kontenery (**nie pody**).

**To-do i Tips and tricks:**
- Kontenery a nie pody :)

### 2.3 
Znajdź 3 serwisy które mają największe problemy ze spełnieniem SLO zadeklarowanego jako:

90% zapytań kończy się poniżej 10ms

**To-do i Tips and tricks:**
- Do mierzenia czasu odpowiedzi możesz użyć `duration_milliseconds_bucket`
- Zestawienie funkcji Prometeusza jest [tu]([bottomk](https://prometheus.io/docs/prometheus/latest/querying/operators/))
- Pamiętaj, że SLO liczy się dla serwisu, a nie dla instancji.

### 2.4 Czas odpowiedzi

Policz jaki musi być czas odpowiedzi serwisu `frontend-web` żeby wliczał się w 0.9 percentyl

**To-do i Tips and tricks:**
- Wykorzystaj zapytanie z poprzedniego zadania jako start.
- Funkcja licząca kwantyle to `histogram_quantile`
- Dobra interaktywna wizualizacja [percentyli](https://www.desmos.com/calculator/zxtwpjsjen)