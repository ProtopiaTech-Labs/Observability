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

> Uwaga poniższe kwerendy odpalamy w [UI Prometeusza](http://prometheus.workshop.indexoutofrange.com/)

Wartość za ostatnie 30 minut
```
http_client_request_duration_seconds_sum{app="webmvc", error_type="403"}[30m]
```

Wartości za ostatnie 30 minut z rozdzielczością 15 minut (przescrolij wyniki najbardziej w prawo):
```
http_client_request_duration_seconds_sum{app="webmvc", error_type="403"}[30m:15m]
```

### 1.6 Zrozumieć `rate` i `increment`

#### 1.6.1 rate i increment vs oryginalne wartości

❗❗❗ Zrozumienie `rate` i `increment` **jest kluczowe do zrozumienia poprawnego odpytywania się o metryki w Prometheus**. Poświęć czas na przeczytanie tych artykułów. 
❗❗❗

Uruchom na jednym wykresie (w razie potrzeby znormalizuj mnożąc wartości przes odpowiednie wartości tak aby wydoczne były zmiany w wykresach) kwerendy:

> ⚠️Prometheus zwraca wektory(macierze). Można na nich dokonywać standardowe operacja macierzowe (jak mnożenie przez stałą).
> Opis możliwości mnożenia wektorów jest w [tej dokumentacji](https://prometheus.io/docs/prometheus/latest/querying/operators/).

**Oryginalne wartości:**
```
http_client_request_duration_seconds_sum{app="webmvc", error_type="403"}
```

**Increase:**
```
increase(http_client_request_duration_seconds_sum{app="webmvc", error_type="403"}[2m])
```

**Rate:**
```
rate(http_client_request_duration_seconds_sum{app="webmvc", error_type="403"}[2m])
```

**🎯⚠️To-do, tips and tricks:**
- Do analizy jak działają powyższe przydaje się zmiana formatu na `Table` dostępny w `Options` dla kwerendy.
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

**🎯Pytanie:**
- Czemu jedna ma a druga nie ma wyników?

**⚠️Tips and tricks:**
- Przeczytaj [artykuł o wyborze interval](https://www.robustperception.io/what-range-should-i-use-with-rate/)
- Oraz przeczytaj artykuł o wprowadzeniu [`$__rate_interval`](https://grafana.com/blog/2020/09/28/new-in-grafana-7.2-__rate_interval-for-prometheus-rate-queries-that-just-work/) przez Grafanę.

### 1.7 Historgram

Uruchom zapytanie:

```
http_client_duration_milliseconds_bucket{ http_status_code="200",net_peer_name="otel-demo-emailservice" }
```

**🎯⚠️To-do, tips and tricks:**
- Czym się od siebie różnią buckety? (Legenda pod grafem)
- Jak układają się wartości?
  - Czy wartość dla `le 100` zawiera wartości dla `le 10`?

## 2. Zadania

### 2.1 Ile jest GB wykorzystanych na nodach Kubernetes

**🎯Cel:**

Stwórz wykres pokazujący ile jest zajętej pamięci w GB na każdym **nodzie** (❗nie podzie❗) w klastrze Kubernetes.

**⚠️To-do i Tips and tricks:**
- Metryki są w `node_memory_....`.
- Wyniki powinny być w GB.
- Legendą powinna być nazwa noda Kubernetes.


### 2.2 Najbardziej pamięciożerne kontenery
**🎯Cel:**

Stwórz wykres pokazujące najbardziej pamięciożerne kontenery (**nie pody**).

**⚠️To-do i Tips and tricks:**
- Kontenery a nie pody :)
- Zobacz metryki `container_memory_.....`

### 2.3 Problem z SLO
**🎯Cel:**

Znajdź 3 serwisy które mają największe problemy ze spełnieniem SLO zadeklarowanego jako:

```90% zapytań kończy się poniżej 10ms```

**⚠️To-do i Tips and tricks:**
- Do mierzenia czasu odpowiedzi możesz użyć `duration_milliseconds_bucket`
- Zestawienie funkcji Prometeusza jest [tu](https://prometheus.io/docs/prometheus/latest/querying/operators/). Może przydać się funkcja `bottomk`.
- Pamiętaj, że SLO liczy się **dla serwisu**, a **nie dla instancji**.
- Jak policzyć które serwisy mają największy problem z tym SLO?
  - Zobacz jakie są buckety dla `duration_milliseconds_bucket`. Te wartości to czasy odpowiedzi.
  - Dzieląc ilość requestów z czasem <=10ms przez całkowitą ilość requestów dla serwisu otrzymasz SLO dla serwisu

### 2.4 Czas odpowiedzi
**🎯Cel:**
Policz jaki musi być czas odpowiedzi serwisu `frontend-web` żeby wliczał się w 0.9 percentyl

**⚠️To-do i Tips and tricks:**
- Wykorzystaj zapytanie z poprzedniego zadania jako start.
- Funkcja licząca kwantyle to `histogram_quantile`
- Dobra interaktywna wizualizacja [percentyli jest tu](https://www.desmos.com/calculator/zxtwpjsjen)