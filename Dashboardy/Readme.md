# Opis przypadku

Mamy sklep internetowy (byłoby forum, ale z forów już nikt nie korzysta... ;) )

Sklep oferuje takie funkcjonalności jak:
- Wyszukiwanie produktów
- Dodawanie do koszyka
- Zakładanie konta
- Tworzenie list zakupowych
- Wysyłkę za pomocą paczkomatu
- Płatność:
  - Kartą korzystając z bramki płatności
  - Blikiem korzystając z API BLIK
  - Szybkim przelewem bankowym korzystając z usługi zewnętrznej

Składa się z takich serwisów jak:
- Baza danych SQL
- Nginx jako api gateway i cache plików HTTP
- Serwis zajmujący się dostarczaniem plików z backendem na blobie w chmurze
- Wyszukiwanie w Elastic Search
- Frontend w JavaScript
- Backend serwujący REST dla frontendu
- System zarządznia dla pracowników
- Serwis obsługujący taski w tle:
  - Wysłka maili
  - Raporty dzienne dla biznesu
  - Analizę potencjalnych wyłudzeń zanim paczki zostaną wysłane. Korzystamy z tego, że paczki wysyłamy kolejnego dnia więc w nocy uruchamiana jest SQL która robi sprawdzenia czy zamówienia nie są wyłudzeniami itp. 
  - Pobieranie statusów paczek z in Post
  - etc.
- RabbitMQ do komunikacji asynchronicznej

Sklep stoi w 3 wersjach, bo jest on postawiony jako white-label (czyli ze zmienionym wyglądem) dla 4 innych firm. Z nimi mamy podpisaną umowę SLA w brzmieniu:

```
95% zapytań będzie realizowanych pomyślnie z czasem odpowiedzi poniżej 200ms
```

# Zadanie 1 - Metryki biznesowe
Zaprojektuj metryki biznesowe. Czyli metryki które:
- Stosują **osoby z biznesu** albo zrozumie bez znacznego tłumaczenia (czyli wytłumaczycie to w 5 minut 9 letniemu dziecku).
- Mapują się na operacje biznesowe które wykonujemy w interakcji z naszymi partnerami.
- Świadczą o tym ile wartości biznesowej dostarcza system (czy potensjalnie zarabiamy dużo, czy mało pieniędzy)
- Czy nie grożą nam jakieś ryzyka biznesowe

# Zadanie 2 - Podstawowe metryki techniczne
Zaprojektuj metryki techniczne. Czyli takie które:
- Zrozumie **osoba techniczna, ale nie developer** (administrator, Ops, pracownik 2 lini wsparcia) 
> Pierwsza linia wsparcia to jest call center który odbiera telefon jak chcemy zmienić zamówienie, albo coś jest nie tak z zamówieniem. 
- Które świadczą o problemie, albo przyszłym problemie systemu

# Zadanie 3 - Zaprojektuj dashboard techniczny
Dashboard ma odpowiadać na pytanie: "Co się **zmieniło** że nie działa". To jest dashboard który otwiera się jak widać, że system nie działa albo działa źle.

Kilka uwag:
- Użyj wcześniej zdefiniowanych metryk i zastanów się jak je wykorzystać
- Zastanów się jak możesz pokazać co się **zmieniło**
- Pamiętaj o [annotacjach](https://grafana.com/docs/grafana/latest/dashboards/build-dashboards/annotate-visualizations/)

# Zadanie 4 - Alerty
Zaproponuj alerty i ich poziomy dla systemu.

**Uwagi:**
- Domyślnie są 3 poziomy alertu:
  - Slack
  - Mail
  - Telefon
- Określ odbiorcę alertu
- Pamiętaj, że alerty nie muszą być tylko o działaniu systemu, ale także o kwestiach operacyjnych wokoło niego. Np: czy wykonał się skrypt optymalizujący indeksy na bazie?
