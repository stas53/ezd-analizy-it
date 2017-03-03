Nienaruszalność danych w systemie EZD
=====================================

Wstęp
-----

Problemem prosto zaprojektowanych baz danych (zarówno cyfrowych, jak i analogowych) jest ich niewiarygodność jako źródła danych historycznych. Administrator o niecnych zamiarach może bazę danych dowolnie edytować (dokonując m.in. antydatowania), stąd przedstawienie osobie postronnej jej zawartości jako potwierdzenia czynności przeszłych nie stanowi samo w sobie wystarczającego dowodu. Przykładowo, administrator prostego systemu EZD mógłby:

* skasować istniejący dokument z akt sprawy;
* antydatować nieistniejący wcześniej dokument;
* podmienić treść istniejącego dokumentu na inną.

Zapisywanie w bazie danych historii zmian nie stanowi samo w sobie żadnego zabezpieczenia przed tym procederem, jako że administrator może wraz ze sfałszowaniem danych właściwych sfałszować również historię zmian ich dotyczącą.

Łańcuch sum kontrolnych
-----------------------

Narzędziem pozwalającym zyskać niemal stuprocentową pewność co do nienaruszalności określonego dokumentu jest `funkcja skrótu`_, pełniąca rolę `sumy kontrolnej`_. Jeśli w aktach danej sprawy każdy dokument zawiera w swojej treści możliwą do odczytania funkcję skrótu poprzedzającego go dokumentu, to do potwierdzenia treści wszystkich dokumentów z tej sprawy wystarczy znajomość funkcji skrótu ostatniego dokumentu.

Wymóg publikacji
----------------

Weryfikacja istnienia dokumentu w danym punkcie czasu za pomocą funkcji skrótu wymaga, aby wartość tej funkcji dla danego dokumentu była znana w tym punkcie.
 Dokument `„Keyless Signatures’ Infrastructure: How to Build Global Distributed Hash-Trees”`_ (suma kontrolna SHA256: ``422b8a0be599660503ae289977b2bda174c81e6fab486db5ef03934c1d1cf151``) opisuje, jak dzięki agregacji można spełnić ten wymóg publikując tylko pojedyncze wartości dla praktycznie nieograniczonych zbiorów różnych danych. Publikacja powinna nastąpić w szeroko dostępnym medium, w którym podmiana danych jest bądź to niemożliwa, bądź łatwa do wykrycia. Można wśród nich wymienić:

* gazetę;
* publicznie dostępne repozytorium Git_;
* łańcuch bloków określonej kryptowaluty (z wykorzystaniem kodu `OP_RETURN`_).

Sposoby zapewnienia krótkoterminowej nienaruszalności
-----------------------------------------------------

Podpisywanie wybranych obiektów i znakowanie ich czasem
+++++++++++++++++++++++++++++++++++++++++++++++++++++++

**Rozwiązanie proponowane dla EZD RP.**

Obiekty, których utworzenie/edycja są uznawane za prawnie istotne, są podpisywane (w postaci zserializowanej), a do opakowania podpisu jest wykorzystywany kontener XAdES_ lub CAdES_ (do ustalenia). Serwer po otrzymaniu podpisanych obiektów dokonuje znakowania ich czasem za pomocą KSI. URI odzwierciedlające powiązania podpisywanych obiektów są tworzone zgodnie z `RFC 6920`_, z obowiązkowo podanym parametrem ``ct`` oraz bez sekcji ``authority``.

Wykorzystanie repozytorium audycyjnego
++++++++++++++++++++++++++++++++++++++

Założenia
~~~~~~~~~

Odseparowanie:

* operacyjnej bazy danych, używanej w bieżącej działalności Urzędu,
* zbioru dokumentów, których racją istnienia jest umożliwienie - w razie takiej konieczności - udowodnienia, iż w pewnym momencie istniał konkretny stan danych.

Tą drugą kolekcję danych nazwiemy "Repozytorium audycyjne" (RA), w odróżnieniu od głównej bazy operacyjnej, nazywanej po prostu bazą danych (BD).

Jedna instalacja systemu EZD zawiera dokładnie jedną BD i jedno RA.

Baza danych
~~~~~~~~~~~

Budowa bazy danych nie jest przedmiotem tego dokumentu. Zakładamy jedynie, że wśród czynności Urzędu powodujących zmiany w BD, wyróżnimy takie, które są istotne
z prawnego punktu widzenia, np.

* pojawienie się nowego pliku, sprawy
* decyzja
* zmiana uprawnień pracownika
* itp.

Tego rodzaju "wydarzenia" będziemy chcieli odwzorować w Repozytorium Audycyjnym.

Repozytorium audycyjne
~~~~~~~~~~~~~~~~~~~~~~

Otrzymuje z BD informacje (wszystkie nazywane tu "dokumentami").

Dla każdego dokumentu obliczana jest suma kontrolna ("hasz") -- identyfikator, np. 32 bajtowy, który jest unikalnym identyfikatorem tego dokumentu; zakładamy że prawdopodobieństwo
iż dwa różne dokumenty będą miały tę samą wartość hasza jest pomijalnie małe.

Repozytorium przechowuje też własny hasz, który może być interpretowany jako "bieżący stan repozytorium". Gdy zostanie dostarczony nowy dokument, obliczany jest nowy stan repozytorium,
który jest wartością hasza obliczoną dla połączenia hasza stanu repozytorium z haszem nowodostarczonego dokumentu. Ilustruje to rysunek.

.. image:: images/repozytorium_i_baza_danych.png

Repozytorium audycyjne nie odwzorowuje wprost informacji o strukturze wiedzy w BD. W szczególności, związki pomiędzy pismami a sprawami, związki między różnymi wersjami tego samego pisma,
urzędowe znaki nadane pismom i sprawom, decyzje w sprawach itd. są pamiętane w postaci pojedynczych dokumentów, pamiętających pojedyncze opisy lub decyzje, razem ze specyficznymi dla nich
haszami. Połączenie z haszami stanu (,,Migawka Urzędu'') tworzy nowe hasze stanu. Okresowe utwalenie niektórych haszy stanu (np. publikacje) umożliwia wykazanie poprawności ciągu dokumentów.

Wykorzystanie systemu kontroli wersji
+++++++++++++++++++++++++++++++++++++

Proponowane rozwiązanie przypomina zasadę, na jakiej działa system kontroli wersji Git_. Jest on szeroko wykorzystywany do zapisywania treści różnych wersji programów w projektach
informatycznych. Używając Gita, można by wprost zapisywać w repozytorium dokumenty / sprawy, przy następujących upraszczających założeniach:

1. Podstawowym, niepodzielnym obiektem jest dokument.
2. Sprawy to ciągi dokumentów. Dekretacje, akceptacje, informacje o udzieleniu dostępu do dokumentów itp. traktujemy w tym ujęciu bądź jako część dokumentów nich dotyczących, bądź jako oddzielne dokumenty.
3. Sprawom nadaje się znaki [#skladowe-znaku-sprawy]_.
4. Każdy dokument musi docelowo znaleźć się w aktach jakiejś sprawy lub w jakimś rejestrze.
5. Metadane i inne dane ułatwiające pracę z dokumentami (wersje robocze wraz z komentarzami do nich etc.) są przechowywane i indeksowane oddzielnie, jako niemające znaczenia prawnego.
6. Na system EZD jednego urzędu przypada jedno repozytorium Git.

Dokument wyabstrahowany z kontekstu sprawy bądź rejestru możemy w repozytorium Git odwzorować jako kroplę (ang. „blob”). Sprawy możemy odzworowywać jako drzewa (ang. „trees”), które w repozytorium Git oznaczają zbiory kropel oraz poddrzew (z których to poddrzew możemy korzystać w przypadku wydzielenia sprawy). Znak sprawy mógłby być zawarty w łańcuchu nazw plików prowadzących do sprawy z głównego drzewa.

Przyjmijmy, że chcemy, aby liczba operacji haszujących potrzebnych do zweryfikowania integralności danej sprawy nie zależała w istotnym stopniu od ogólnej aktywności w repozytorium między wprowadzaniem poszczególnych dokumentów. Możemy to osiągnąć poprzez wykonywanie operacji na danej sprawie na oddzielnej gałęzi (ang. „branch”). Gałąź ta byłaby regularnie włączana do głównej gałęzi.

Istnienie wkładów łączących (ang. „merge commits”) odkładanych na głównej gałęzi byłoby regularnie potwierdzane w zewnętrznej usłudze (vide „Wymóg publikacji”). Po potwierdzeniu istnienia wkładu byłby on oznaczany etykietą z adnotacją (ang. „annotated tag”), przy czym w treści adnotacji byłyby zawarte informacje potrzebne do weryfikacji poprawności potwierdzenia. Do weryfikacji istnienia określonego stanu sprawy w określonym punkcie czasu przez obywatela wystarczyłyby zatem:

* Pełne dane gałęzi odpowiadającej danemu stanowi.
* Znajomość surowych treści ciągu wkładów [#surowa-tresc-wkladu]_ włączających czubek tej gałęzi do głównej gałęzi.
* Znajomość danych pozwalających na weryfikację odnośnego wkładu z głównej gałęzi w zewnętrznej usłudze.

Jeżeliby gałąź sprawy zawierała tylko dane jej dotyczące [#numeracja-spraw]_, to obywatel mógłby dokonać takiej weryfikacji bez dostępu do danych innych spraw.
Rejestry przesyłek wpływających i wychodzących (oraz ewentualne inne rejestry dokumentów) można by odwzorowywać jako drzewa, w podobny sposób jak sprawy.

*Uwaga: nie jest naszym zamiarem realizacja repozytorium poprzez wykorzystanie Gita. Chcieliśmy tylko pokazać, że można by stworzyć odporne na manipulacje repozytorium danych EZD
wykorzystujące sumy kontrolne podobnie, jak to robi Git. Rzeczywista implementacja mogłaby wymagać rozwiązania dedykowanego i uwzględniać bardziej skomplikowane mechanizmy i struktury danych.
Nie analizowaliśmy też wydajności takiego systemu; niewykluczone, że w specyfice systemu EZD lepiej sprawdziłyby się inne systemy kontroli wersyj, np. Mercurial_.
Metadane, indeksowanie i funkcjonalności dodatkowe musiałyby być wdrażane poza repozytorium, z wykorzystaniem dodatkowej bazy danych.*

Zapewnienie spójności całej bazy danych
+++++++++++++++++++++++++++++++++++++++

Innym podejściem do zapewnienia nienaruszalności danych jest zabezpieczenie pod tym kątem całej bazy. „Gratisowo” otrzymujemy taki rezultat w przypadku korzystania z nowoczesnego, rozproszonego systemu kontroli wersyj jako bazy danych, choć weryfikowalność danych przez obywatela mogłaby wymagać pewnych wyszukanych zabiegów opisanych wcześniej.

`Firma Guardtime ogłosiła integrację ichniego systemu KSI z bazą danych Oracle`_, jednak komunikat prasowy nie obfituje w szczegóły techniczne. Można rozważać tworzenie ogólnych mechanizmów zabezpieczania integralności baz SQL czy też baz bezschematowych, aczkolwiek należy dostrzec potencjalne problemy z tym związane:

1. Zapewnienie nienaruszalności danych to pożądana funkcjonalność, która jednak nie jest dostępna w większości baz danych. Przypuszczalnie wdrożanie jej nie jest łatwe.
2. Zapewnienie nienaruszalności całej bazy danych może prowadzić do nadmiarowości, tj. do ochrony danych, które takiej ochrony nie potrzebują, i tym samym do zbyt dużego wykorzystania przestrzeni dyskowej.
3. Przywiązanie do konkretnej technologii bazodanowej mogłoby utrudnić wprowadzenie standardu EZD określającego format eksportu i importu oraz sposób weryfikacji danych systemu kancelaryjnego. Wprowadzenie takiego standardu (podobnie jak w świecie kryptowalut istnieje standard określający format transakcji i bloków) mogłoby pozwolić na przeprowadzanie eksportu i importu między różnymi systemami EZD z zachowaniem weryfikowalności danych.

Rzetelna ocena problemów i szans związanych z zapewnieniem nienaruszalności całej bazy danych wymagałaby oddzielnej analizy.

Słabe strony
------------

Możliwość tworzenia wersyj równoległych
+++++++++++++++++++++++++++++++++++++++

System KSI pozwala na udowodnienie, że określony stan bazy danych istniał w konkretnym czasie, ale nie pozwala sam w sobie na udowodnienie, że był on „obowiązujący”. Administrator o złych intencjach mógłby tworzyć równoległe wersje tej samej bazy danych i wysyłać do potwierdzenia za pomocą KSI wszystkie (jako że wysyłane są tylko wartości funkcji skrótu, to taki konflikt nie zostałby wykryty). Możliwości takie można zniwelować poprzez:

1. Publikację wartości funkcji skrótu odzwierciedlającej stan bazy danych z pominięciem systemu KSI, w medium pozwalającym na przypisanie tej wartości do określonego urzędu (np. ogłoszenie w gazecie, repozytorium Git będące we władaniu urzędu, transakcja kryptowalutowa wysłana z wykorzystaniem kluczy będących we władaniu urzędu).
2. Odwoływanie się do łańcucha sum kontrolnych w podpisach elektronicznych. Dzięki temu podpis uczciwego człowieka składany na dokumencie w danej sprawie poświadczałby również historię tej sprawy i uniemożliwiał jej zmianę do tego punktu.
3. Publikację cząstkowych sum kontrolnych w inny sposób (np. publikacja sumy konrolnej sprawy w powiadomieniach emailowych wysyłanych obywatelowi).

Długoterminowa archiwizacja podpisów
------------------------------------

Tradycyjna metoda konserwacji podpisu polega na opatrywaniu go pieczęcią elektroniczną poświadczającą, że podpis został utworzony przed określonym punktem czasowym (w którym tylko właściwa osoba dysponowała danymi pozwalającymi na złożenie tego podpisu). Pieczęć korzysta przy tym z tej samej technologii, co sam podpis. Zwiększenie mocy obliczeniowej komputerów niesie ze sobą ryzyko złamania zarówno jednego, jak i drugiego. Przyjmuje się, że wejście w życie komputerów kwantowych będzie oznaczało koniec współcześnie popularnych metod podpisywania (por. dokument “`CNSA Suite and Quantum Computing FAQ`_”). Za pomocą komputerów kwantowych będzie można, mając dostęp do publicznych certyfikatów, podrobić zarówno podpis pod dokumentem, jak i pieczęć znakującą ten podpis czasem.

Konserwacja podpisów w oparciu o Keyless Signatures’ Infrastructure bazuje tymczasem nie na tej samej technologii, co podpis (kryptografia klucza publicznego), tylko na funkcjach skrótu, które są uznawane za odporne na zagrożenia związane z informatyką kwantową. Co za tym idzie, o ile ktoś byłby w stanie w przyszłości podrobić podpis stosowany współcześnie, to niekoniecznie będzie w stanie udowodnić, że podpis ten został współcześnie stworzony, co jest kluczowe przy jego weryfikacji (certyfikaty wykorzystywane do tworzenia podpisów mają swoje okresy ważności).

Dodatkową zaletą KSI jest umniejszenie roli zaufanej trzeciej strony, co powinno pozwolić na:

1. Ogólnie większy poziom zaufania do usługi znakowania czasem.
2. Niższe koszty organizacyjne związane z zapewnieniem procedur bezpieczeństwa — które to procedury nie będą potrzebne w takim stopniu, jak obecnie, jako że nie będą istniały informacje tajne wymagające ochrony (certyfikaty pieczęci).

.. _CNSA Suite and Quantum Computing FAQ: https://www.iad.gov/iad/library/ia-guidance/ia-solutions-for-classified/algorithm-guidance/cnsa-suite-and-quantum-computing-faq.cfm
.. _funkcja skrótu: https://pl.wikipedia.org/wiki/Funkcja_skr%C3%B3tu
.. _sumy kontrolnej: https://pl.wikipedia.org/wiki/Suma_kontrolna
.. _`„Keyless Signatures’ Infrastructure: How to Build Global Distributed Hash-Trees”`: https://eprint.iacr.org/2013/834.pdf
.. _CAdES: https://tools.ietf.org/html/rfc5126
.. _XAdES: https://www.w3.org/TR/XAdES/
.. _Git: https://git-scm.com/
.. _RFC 3161: https://www.ietf.org/rfc/rfc3161.txt
.. _RFC 6920: https://tools.ietf.org/html/rfc6920
.. _Mercurial: https://www.mercurial-scm.org/
.. _`Firma Guardtime ogłosiła integrację ichniego systemu KSI z bazą danych Oracle`: https://guardtime.com/blog/guardtime-announces-ksi-blockchain-integration-for-oracle-11g-12c
.. _OP_RETURN: https://en.bitcoin.it/wiki/OP_RETURN

.. [#skladowe-znaku-sprawy]
   Zgodnie z instrukcją kancelaryjną, znak sprawy zawiera następujące elementy:

   1. oznaczenie komórki organizacyjnej;
   2. symbol klasyfikacyjny z wykazu akt;
   3. kolejny numer sprawy, wynikający ze spisu spraw;
   4. cztery cyfry roku kalendarzowego, w którym sprawa się rozpoczęła.

.. [#surowa-tresc-wkladu]
   W repozytorium Git można ją uzyskać wykonująć komendę ``git cat-file -p <ID_WKŁADU>``.

.. [#numeracja-spraw]
   Zauważmy, że w tej sytuacji numer sprawy, stanowiący informację zawartą w gałęzi sprawy, musiałby być generowany z wykorzystaniem informacji nieznajdujących się na tej gałęzi (numerów innych spraw).

   Jednym ze sposobów obejścia problemu potencjalnych konfliktów z tym związanych jest wydzielenie gałęzi zawierających sprawy z określonych zestawów komórek organizacyjnych, lat kalendarzowych i symboli klasyfikacyjnych. Sprawy byłyby następnie zakładane za pomocą sekwencyjnej (per taki zestaw) usługi, która tworzyłaby gałąź sprawy i włączałaby ją do gałęzi zestawu (synchronizowanej oczywiście do głównej gałęzi systemu EZD).
