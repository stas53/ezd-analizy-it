Zasady konstrukcji API – do wewnętrznego użycia w EZD oraz dla zewnętrznych aplikacji
=====================================================================================

Rekomendacje.
>>>>>>>>>>>>>






Spis rzeczy
'''''''''''''

 - API dla Java
 - API dla REST
 - API dla JavaScript







API dla Java
------------

**Projektuj przewidując rozwój**

Jeśli API jest cokolwiek warte – będzie ewoluować.   Z drugiej strony, jeśli coś w API już jest, to zapewne będzie musiało zostać.

2 postulaty wstecznej zgodności:
 - skompilowany kod nadal działa (zgodność binarna)
 - istniejący kod źródłowy można nadal skompilować (zgodność źródłowa).

Nie da się przewidzieć, w jaki sposób użytkownicy skorzystają z API.
Rozważając zmianę – należy rozumować konserwatywnie: tylko jeśli można stwierdzić z prawdopodobieństwem graniczącym z pewnością, że zmiana nie spowoduje załamania jakiejkolwiek istniejącej aplikacji – można zmianę wprowadzić.

Jeśli potrzebne jest wprowadzenie większych zmian – należy zbudować nowe API, z inną nazwą.

Dobrym pomysłem jest wprowadzenie na początek jednej lub kilku wersji o numerach 0.xx przed wersją 1.0. Użytkownicy rozumieją, że takie wersje mogą być zmieniane.

**API – cele projektowe**

Musi być:
 - absolutnie poprawny
 - łatwy do użycia
 - łatwy do nauczenia
 - wystarczająco szybki
 - wystarczająco mały.

**Minimalizm**

 Znacznie łatwiej dodawać nowe elementy niż je usuwać.

 Im więcej jest elementów w API, tym trudniej jest się nauczyć.

 Im większy API, tym więcej może być błędów.

 Nadmiarowe metody niepotrzebnie zajmują miejsce.

**Ostrożnie z pakietami**

   Całe API powinno być w jednym pakiecie.

**Różne wskazówki**

 Dobrze jest użyć klas niezmiennych.

 Jedyne widoczne pola powinny być ‘static’ i ‘final’.

 Unikaj ekscentryczności: pisz tak jak zwykle inni piszą. Jest wiele zwyczajów dotyczących nazw, getterów & setterów, klas obsługujących wyjątki etc.

 Nie implementuj Cloneable: tworzenie kopii obiektu jest zwykle mniej przydatne niż może się wydawać.

 Wyjątki (Exceptions) zwykle powinny nie być przesłaniane.

 Przewidź stosowanie dziedziczenia - lub nie pozwalaj na to.



API dla REST
------------


**Używaj czasowników, nie rzeczowników**

+-----------+---------------+--------------+--------------+-----------+	                        
|           | GET           | POST         | PUT          | DELETE    |
|           | pobieranie    | tworzenie    | aktualizacja | usuwanie  |
+-----------+---------------+--------------+--------------+-----------+
| /cars     | Zwraca listę  | Tworzy       | Masowa       | Usuwa     |
|           | samochodów    | nowy         | aktualizacja | wszystkie |
+-----------+---------------+--------------+--------------+-----------+	                        
| /cars/711 | Zwraca        | Metoda       | Aktualizuje  | Usuwa     |
|           | wskazany      | niedozwolona | wskazany     | wskazany  |
|           |               | ( 405 )      |              |           |
+-----------+---------------+--------------+--------------+-----------+	                        


**Metoda GET i parametry zapytania nie powinny modyfikować stanu**

  Nie używaj GET dla poleceń zmieniających stan.

**Używaj liczby mnogiej**

  Nie mieszaj rzeczowników w liczbie pojedynczej i mnogiej.

  Niech będzie prosto. Używaj tylko liczby mnogiej.

**Używaj hierarchii dla wskazania relacji między obiektami**

 ::

   GET /cars/711/drivers/               Lista kierowców samochodu 711
   GET /cars/711/drivers/4             Kierowca nr 4 samochodu 711

**Używaj nagłówków HTTP**

   Zarówno klient jak i serwer powinny znać format używany w komunikacji. Format powinien być wyspecyfikowany w nagłówku HTTP.

   *Content-Type*    określa format żądania

   *Accept*              określa listę akceptowalnych formatów odpowiedzi.

**Używaj HATEOAS (Hypermedia as the Engine of Application State)**

 Klient kontaktuje się z aplikacją wyłącznie przez interfejs dostarczany przez serwer aplikacyjny.
 Nie potrzebuje żadnej wstępnej wiedzy o tym, jak komunikować się, poza generalną wiedzą o hipermediach:

 - rekordy są adresowane przez URL
 - można w ten sposób otrzymać wszystkie rekordy
 - ścieżka do korzenia udostępnia index API.

 W odróżnieniu, niektóre aplikacje o architekturze SOA (service-oriented architecture) komunikują się sztywnym interfejsem albo
 językiem opisu interfejsu (IDL).

 Zasada HATEOAS rozdziela klienta i serwer w sposób, który pozwala im ewoluować niezależnie.

**Umożliwiaj filtrowanie, sortowanie, wybór pól, stronicowanie przy pobieraniu kolekcji**

  Elastyczność filtrowania rozluźnia związek API z modelem.


Przykłady

::

   GET /cars?color=red            Lista czerwonych samochodów

   GET /cars?seats<=2            Lista samochodów o max 2 siedzeniach

   GET /cars?sort=-manufacturer,+model

   GET /cars?fields=manufacturer,model,id,color

   GET /cars?offset=10&limit=5

**Udzielaj kompletnych odpowiedzi**

 Odpowiedzi powinny zawierać wszystkie dane potrzebne do utworzenia kompletnego obiektu modelu.

::

   // GET /users/12
   {
     user: {
       id: 12,                       // powtórzone z pytania, ale ułatwia pracę klienta
       name: 'Bob Barker'
     }
   }

   // GET /users/12
   {
     user: {
       id: 12,
       name: 'Bob Barker',
       todo_list_ids: [ 4, 5 ]    // struktury będą w osobnych rekordach; dyskusyjne
     }
     todo_lists: [
       { id: 4, name: 'shopping' },
       { id: 5, name: 'work' }
     ]
   }

**Używaj warstwy prezentacji**

 Separuj dane API od warstwy danych aplikacji:

 - pozwala obliczać wtórne wartości, które mogą być potrzebne po stronie klienckiej, np. obliczenie płci na podstawie numeru PESEL
 - tworzy to dobrą warstwę do testów
 - chroni kod kliencki od zmian po stronie serwera.

**Autoryzacja**

 Używaj Tokenów.

 Ogranicz użycie sesji i cookies.


**Wersjonuj swoje API**

   /blog/api/v1

**Obsługuj błędy zwracane jako status HTTP**

 Przynajmniej te:

 -  200 – OK
 -  201 – OK                             Utworzono nowy obiekt
 -  204 – OK                             Udało się usunąć
 -  304 – Not Modified               Klient nie może używać buforowanych danych
 -  400 – Bad Request               Niepoprawne żądanie
 -  401 – Unauthorized               Niepoprawna autentykacja
 -  403 – Forbidden                   Niedozwolone żądanie lub brak dostępu do zasobów
 -  404 – Not found                    Nie odnaleziono zasobu URI
 -  422 – Unprocessable Entity   Powinno być użyte, jeśli serwer nie może obsłużyć wskazanego obiektu (np. Brak wymaganych pól)
 -  500 – Internal Server Error  Twórcy API powinni unikać tego błędu. Jeśli wystąpi błąd w aplikacji – powinien być zapisywany do logu; należy starać się zwrócić przyjazny tekst komunikatu.





API dla JavaScript
------------------

**Używaj HTML5**

**Dwie składnie wywołań**

 Dostarcz dwie różne składnie wywołań funkcji:

 - proceduralną
 - obiektową.

**Nazwy**

   Powinny rozpoczynać się i kończyć znakami “a-z”.

   Powinny zawierać jedynie znaki “a-z”, 0-9” i znak łącznika “-”








Wybrane dokumenty w Internecie
------------------------------

http://www.artima.com/weblogs/viewpost.jsp?thread=142428

http://jsonapi.org/recommendations/

https://madhatted.com/2013/3/19/suggested-rest-api-practices

http://blog.mwaysolutions.com/2014/06/05/10-best-practices-for-better-restful-api/

https://developers.google.com/youtube/iframe_api_reference

