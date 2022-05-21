#########################################
Wprowadzenie do inteligentnych kontraktów
#########################################

.. _simple-smart-contract:

****************************
Prosty inteligentny kontrakt
****************************

Zacznijmy od prostego przykładu, który ustawia wartość zmiennej i ujawnia ją innym kontraktom.
Nie musisz teraz wszystkiego rozumieć. Zajmiemy się szczegółami później.

Przykład magazynu
=================

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.16 <0.9.0;

    contract SimpleStorage {
        uint storedData;

        function set(uint x) public {
            storedData = x;
        }

        function get() public view returns (uint) {
            return storedData;
        }
    }

Pierwsza linia mówi, że kod źródłowy jest na licencji GPL w wersji 3.0.
Czytelne dla maszyn oznaczenia licencji są istotne w sytuacji, gdy kod
źródłowy domyślnie jest publikowany.

Następna linia określa, że kod źródłowy jest napisany dla Solidity w wersji 0.4.16
lub nowszej, ale starszej niż 0.9.0. To ma zapewnić, że kontrakt nie skompiluje się
w nowej (przełomowej) wersji kompilatora, gdzie mógłby zachowywać się inaczej.

Kontrakt w rozumieniu Solidity to zbiór kodu (jego *funkcji*) i danych (*stanu*),
które znajdują się pod konkretnym adresem w łańcuchu bloków Ethereum.
Linia ``uint storedData;`` deklaruje zmienną stanu zwaną ``storedData`` typu ``uint``
(*u*\nsigned *int*\eger), czyli liczbę całkowitą bez znaku o wielkości *256* bitów.
Możesz o niej myśleć jak o pojedynczym fragmencie bazy danych, który możesz odpytać
lub zmodyfikować, wywołując funkcje w kodzie, które zarządzają bazą danych.
W tym przykładzie kontakt definiuje funkcje ``set`` i ``get``, które można
użyć do modyfikacji lub pobrania wartości zmiennej.

Aby uzyskać dostęp do pola (zmiennej stanu) bieżącego kontaktu, nie dodajemy prefiksu ``this.``,
lecz odwołujemy się do niej bezpośrednio po nazwie.
W przeciwieństwie do wielu innych języków, pominięcie go nie jest kwestią stylu,
gdyż całkowicie zmienia sposób dostępu do pola. Dowiesz się więcej później.

Ten kontrakt nie robi nic więcej poza (z powodu infrastruktury zbudowanej przez Ethereum)
pozwoleniem każdemu na zapisanie pojedynczej liczby, która jest dostępna dla każdego na
świecie i nie ma (skutecznego) sposobu, aby uniemożliwić opublikowanie tej liczby.
Każdy może ponownie wywołać ``set`` z inną wartością i nadpisać twoją liczbę, ale
poprzednia liczba wciąż jest przechowywana w historii łańcucha bloków. Później
zobaczysz, jak wprowadzić kontrolę dostępu, abyś tylko ty mógł zmienić liczbę.

.. warning::
    Zachowaj ostrożność, kiedy używasz tekstu Unicode, ponieważ podobnie wyglądające (a nawet identyczne)
    znaki mają różne kody i są kodowane jako inne tablice bajtów.

.. note::
   Wszystkie identyfikatory (nazwy kontraktów, funkcji i zmiennych) muszą składać się tylko ze znaków ASCII.
   Dopuszcza się zapis danych zakodowanych w UTF-8 w zmiennych typu string.

.. index:: ! subcurrency

Przykład podwaluty
==================

Poniższy kontakt implementuje kryptowalutę w najprostszej postaci. Pozwala on tworzyć nowe monety tylko jego twórcy
(różne programy emisji są możliwe). Dowolna osoba może wysłać monety do kogokolwiek bez potrzeby rejestracji za pomocą
nazwy użytkownika i hasła. Wszystko, czego potrzeba, to para kluczy Ethereum.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.4;

    contract Coin {
        // Słowo kluczowe "public" czyni zmienną
        // dostępną dla innych kontraktów
        address public minter;
        mapping (address => uint) public balances;

        // Zdarzenia umożliwiają klientom reakcję na konkretne
        // zmiany w kontrakcie, jakie zadeklarujesz
        event Sent(address from, address to, uint amount);

        // Konstruktor jest uruchamiany tylko wtedy,
        // kiedy kontrakt jest tworzony
        constructor() {
            minter = msg.sender;
        }

        // Wysyła określoną ilość nowo utworzonych monet na podany adres
        // Może być wywołana tylko przez twórcę kontraktu
        function mint(address receiver, uint amount) public {
            require(msg.sender == minter);
            balances[receiver] += amount;
        }

        // Błędy pozwalają przekazać szczegóły, dlaczego operacja
        // nie powiodła się. Są zwracane do funkcji wywołującej.
        error InsufficientBalance(uint requested, uint available);

        // Wysyła określoną ilość istniejących monet
        // od dowolnej osoby pod wskazany adres
        function send(address receiver, uint amount) public {
            if (amount > balances[msg.sender])
                revert InsufficientBalance({
                    requested: amount,
                    available: balances[msg.sender]
                });

            balances[msg.sender] -= amount;
            balances[receiver] += amount;
            emit Sent(msg.sender, receiver, amount);
        }
    }

Ten kontrakt wprowadza kilka nowych założeń. Przejdźmy przez nie po kolei.

Linia ``address public minter;`` deklaruje zmienną stanu typu :ref:`address<address>`.
Typ ``address`` zawiera dane o długości 160 bitów i nie pozwala na operacje arytmetyczne.
Jest odpowiedni do przechowywania adresów kontraktów lub sumy kontrolnej klucza publicznego
z pary kluczy należącej do :ref:`external accounts<accounts>`.

Słowo kluczowe ``public`` automatycznie generuje funkcję umożliwiającą odczyt aktualnej wartości zmiennej stanu
spoza kontraktu. Bez tego słowa inne kontrakty nie mogą uzyskać dostępu do zmiennej.
Kod tej funkcji generowany przez kompilator wygląda jak poniżej
(na razie zignoruj ``external`` i ``view``):

.. code-block:: solidity

    function minter() external view returns (address) { return minter; }

Mógłbyś sam napisać taką funkcję jak powyżej, ale miałbyś funkcję i zmienną stanu o takiej samej nazwie.
Nie potrzebujesz tego robić, ponieważ kompilator wykona to za ciebie.

.. index:: mapping

Następna linia ``mapping (address => uint) public balances;`` też tworzy publiczną zmienną stanu, ale to jest bardziej
złożony typ danych. Typ :ref:`mapping <mapping-types>` przypisuje adresy do :ref:`unsigned integers <integers>`.

Mapowania można postrzegać jako `tablice asocjacyjne <https://pl.wikipedia.org/wiki/Tablica_mieszająca>`_ które są
wirtualnie inicjowane w taki sposób, że wszystkie możliwe klucze istnieją od początku i mają przypisaną wartość,
jaka w reprezentacji bajtowej składa się z samych zer. Jednak nie da się pobrać listy wszystkich kluczy mapy ani
listy jej wartości. Zapisuj, co dodajesz do mapy lub używaj jej tam, gdzie to nie jest potrzebne. Albo jeszcze
lepiej, trzymaj listę lub użyj bardziej odpowiedniego typu danych.

:ref:`Funkcja odczytu<getter-functions>` utworzona przez dodanie słowa ``public``
jest bardziej złożona w przypadku map. Wygląda jak poniżej:

.. code-block:: solidity

    function balances(address account) external view returns (uint) {
        return balances[account];
    }

Możesz użyć tej funkcji do odczytu salda pojedynczego konta.

.. index:: event

Linia ``event Sent(address from, address to, uint amount);`` deklaruje :ref:`"zdarzenie" <events>`, które jest
emitowane w ostatniej linii funkcji ``send``. Oprogramowanie klienckie Ethereum takie jak aplikacje webowe może
nasłuchiwać tych zdarzeń emitowanych na łańcuchu bloków bez dużych kosztów. Od razu po emisji zdarzenia wywoływana
jest funkcja nasłuchująca z argumentami ``from``, ``to`` i ``amount``, które pozwalają śledzić transakcje.

Aby nasłuchiwać to zdarzenie, możesz wykorzystać poniższy kod JavaScript, który używa
`web3.js <https://github.com/ethereum/web3.js/>`_ do tworzenia obiektu ``Coin`` kontraktu
i interfejs użytkownika wywołuje automatycznie wygenerowaną funkcję ``balances`` z powyższego przykładu::

    Coin.Sent().watch({}, '', function(error, result) {
        if (!error) {
            console.log("Transfer monet: " + result.args.amount +
                " monety zostały wysłane z " + result.args.from +
                " do " + result.args.to + ".");
            console.log("Saldo obecnie:\n" +
                "Nadawca: " + Coin.balances.call(result.args.from) +
                "Odbiorca: " + Coin.balances.call(result.args.to));
        }
    })

.. index:: coin

:ref:`Konstruktor<constructor>` to specjalna funkcja, która jest wykonywana podczas tworzenia kontraktu
i nie może zostać wywołana później. W tym przypadku na stałe przechowuje adres osoby tworzącej kontrakt.
Zmienna ``msg`` (łącznie z ``tx`` i ``block``) jest :ref:`specjalną zmienną globalną <special-variables-functions>`
która zawiera pola pozwalające na dostęp do łańcucha bloków. ``msg.sender`` zawsze zawiera adres, z jakiego przyszło
wywołanie obecnej (zewnętrznej) funkcji.

Funkcje tworzące kontrakt, które użytkownicy i inne kontrakty mogą wywołać, to ``mint`` i ``send``.

Funkcja ``mint`` wysyła określoną ilość nowo utworzonych monet pod inny adres. Konstrukcja :ref:`require
<assert-and-require>` spowoduje wycofanie wszystkich zmian, jeśli podane warunki nie zostaną spełnione.
W tym przykładzie ``require(msg.sender == minter);`` zapewnia, że tylko twórca kontraktu może wywołać
``mint``. Zasadniczo twórca może wygenerować tyle tokenów, ile chce, ale w pewnym momencie doprowadzi
do zjawiska zwanego przekroczeniem (overflow). Ze względu na domyślną :ref:`arytmetykę ze sprawdzaniem <unchecked>`
transakcja zostanie wycofana, jeśli wyrażenie ``balances[receiver] += amount;`` spowoduje przekroczenie,
tj. kiedy ``balances[receiver] + amount`` przekroczy maksymalną wartość, jaką może przechować typ danych
``uint`` (``2**256 - 1``). To dotyczy też wyrażenia ``balances[receiver] += amount;`` w funkcji ``send``.

:ref:`Błędy <errors>` pozwalają dostarczyć więcej informacji o tym, dlaczego warunek nie został spełniony
lub dlaczego operacja nie powiodła się. Używa się ich razem z :ref:`instrukcją revert <revert-statement>`.
Instrukcja ``revert`` bezwarunkowo przerywa i wycofuje wszystkie zmiany podobnie jak funkcja ``require``,
ale także pozwala podać nazwę błędu i dodatkowe informacje, które zostaną dostarczone do funkcji wywołującej
(i ostatecznie do aplikacji użytkownika lub przeglądarki bloków), więc można łatwiej przeanalizować problem
i zareagować na niego.

Funkcję ``send`` może wywołać każdy (kto już ma trochę monet), aby przesłać monety do kogoś innego. Jeśli nadawca
nie posiada wystarczającej ilości monet do wysłania, to warunek ``if`` zwróci true. W efekcie instrukcja ``revert``
przerwie operację i przekaże nadawcy szczegóły błędu za pomocą błędu ``InsufficientBalance``.

.. note::
    Jeśli wyślesz monety na jakiś adres, używając tego kontraktu, nie zobaczysz nic
    kiedy wyszukasz ten adres w eksploratorze bloków, ponieważ informacja o wysyłce
    monet i zmianie sald kont są przechowywane tylko w pamięci danych tego konkretnego
    kontraktu. Za pomocą zdarzeń możesz stworzyć "eksplorator bloków", który śledzi
    transakcje i salda twojej nowej kryptowaluty, ale musisz sprawdzać adres kontraktu,
    a nie posiadaczy monet.

.. _blockchain-basics:

*******************
Podstawy Blockchain
*******************

Koncepcja łańcucha bloków nie jest trudna do rozumienia dla programistów. Powód jest taki, że
większość skomplikowanych rzeczy (kopanie, `liczenie funkcji skrótu <https://pl.wikipedia.org/wiki/Funkcja_skrótu>`_,
`kryptografia krzywych eliptycznych <https://pl.wikipedia.org/wiki/Kryptografia_krzywych_eliptycznych>`_,
`sieci peer-to-peer <https://pl.wikipedia.org/wiki/Peer-to-peer>`_, itd.)
są tutaj tylko, aby dostarczyć pewien zbiór funkcji i cech dla platformy.
Kiedy zaakceptujesz je takimi, jakie są, nie musisz przejmować się tym,
jak działają pod spodem. Czy musisz wiedzieć, jak AWS Amazonu działa od wewnątrz, aby go używać?

.. index:: transaction

Transakcje
==========

Łańcuch bloków jest globalnie udostępnioną transakcyjną bazą danych.
To znaczy, że każdy może odczytywać rekordy z tej bazy, po prostu uczestnicząc w sieci.
Jeśli chcesz coś w niej zmienić, musisz stworzyć tak zwaną transakcję,
która musi zostać zaakceptowana przez wszystkich innych.
Słowo transakcja oznacza, że zmiana, jaką chcesz dokonać (załóżmy, że chcesz zmienić
dwie wartości jednocześnie), musi zostać całkowicie zastosowana albo całkowicie odrzucona.
Co więcej, podczas gdy twoja transakcja jest wprowadzana do bazy danych,
żadna inna transakcja nie może dokonać zmian.

Na przykład wyobraź sobie tabelę, która wyświetla salda wszystkich kont
w elektronicznej walucie. Jeśli zlecony jest przelew z jednego konta na drugie,
transakcyjna natura bazy danych zapewnia, że jeśli kwota zostanie odjęta z jednego
konta, to zawsze zostanie dodana do drugiego konta. Jeśli z jakiegoś powodu nie jest
możliwe dodanie kwoty do konta odbiorcy, to saldo konta nadawcy także nie ulegnie zmianie.

Co więcej, transakcja jest zawsze podpisana kryptograficznie przez nadawcę (twórcę).
To umożliwia kontrolę, kto może dokonywać modyfikacji w bazie danych. Na przykładzie
elektronicznej waluty, prosty test zapewnia, że tylko osoba posiadająca klucze do
konta może przelać z niego pieniądze.

.. index:: ! block

Bloki
=====

Dużą przeszkodą do pokonania jest (w terminologii Bitcoin) atak zwany "double-spend".
Co się dzieje, jeśli w sieci istnieją dwie transakcje, które chcą opróżnić konto?
Tylko jedna transakcja może być prawidłowa, zwykle ta, która pierwsza zostanie zaakceptowana.
Problem w tym, że "pierwsza" nie jest pojęciem obiektywnym w sieci peer-to-peer.

Krótka odpowiedź - nie musisz się tym przejmować. Globalnie zaakceptowana kolejność transakcji
zostanie wybrana za ciebie, rozwiązując konflikt. Wszystkie transakcje zostaną umieszczone w "bloku"
i wtedy zostaną wykonane i rozesłane do wszystkich uczestniczących węzłów. Jeśli dwie transakcje są
ze sobą sprzeczne, to druga z nich zostanie odrzucona i nie wejdzie do bloku.

Te bloki tworzą liniowy ciąg w czasie i stąd wywodzi się termin "łańcuch bloków".
Bloki są dodawane do łańcucha przeważnie w regularnych odstępach czasu
- dla Ethereum co około 17 sekund.

W ramach "mechanizmu wyboru kolejności" (zwanego "kopaniem") od czasu do czasu bloki mogą być wycofywane,
ale tylko na końcu łańcucha. Im więcej bloków zostanie dodanych na koniec konkretnego bloku, tym mniejsza
szansa, że ten blok zostanie wycofany. Tak więc twoje transakcje mogą zostać wycofane, a nawet usunięte z
łańcucha bloków, ale im dłużej czekasz, tym mniejsze prawdopodobieństwo, że tak się stanie.

.. note::
    Nie ma gwarancji, że transakcje zostaną dołączone do kolejnego lub któregoś kolejnego bloku,
    ponieważ to nie zależy od ich nadawcy, ale od górników, którzy decydują, do którego bloku dana transakcja zostanie dołączona.

    Jeśli chcesz zaplanować przyszłe wywołania twojego kontraktu, możesz użyć
    narzędzia do automatyzacji inteligentnych kontraktów lub usług "wyroczni".

.. _the-ethereum-virtual-machine:

.. index:: !evm, ! ethereum virtual machine

**************************
Maszyna Wirtualna Ethereum
**************************

Podsumowanie
============

Maszyna Wirtualna Ethereum (Ethereum Virtual Machine, EVM) to środowisko wykonawcze dla inteligentnych kontraktów
w Ethereum. Działa w piaskownicy i jest całkowicie odizolowana, co znaczy, że uruchamiany w niej kod nie ma dostępu
do sieci, systemu plików ani innych procesów. Inteligentne kontrakty mają ograniczony dostęp do innych
inteligentnych kontraktów.

.. index:: ! account, address, storage, balance

.. _accounts:

Konta
=====

Wyróżniamy 2 typy kont w Ethereum, które dzielą tę samą przestrzeń adresową:
**konta zewnętrzne** kontrolowane przez pary kluczy publiczny-prywatny (np. przez ludzi)
oraz **konta kontraktów** zarządzane przez kod przechowywany razem z kontem.

Adres zewnętrznego konta wyznacza się na podstawie klucza publicznego,
natomiast adres kontraktu jest generowany podczas jego tworzenia
(na podstawie adresu twórcy i ilości transakcji wysłanych z tego adresu, tzw. "nonce").

Niezależnie od tego, czy konto przechowuje kod, oba typy są traktowane tak samo przez EVM.

Każde konto ma trwały magazyn typu klucz-wartość przypisujący 256-bitowe słowa do 256-bitowych słów, zwany **storage**.

Co więcej, każde konto zawiera **saldo** w etherach (dokładnie w "wei", gdzie ``1 ether`` to ``10**18 wei``),
które może być modyfikowane przez wysyłanie transakcji zawierających ethery.

.. index:: ! transaction

Transakcje
==========

Transakcja to wiadomość wysyłana z jednego konta do drugiego (które może być tym samym kontem lub pustym, patrz niżej).
Zawiera dane binarne (tzw. "ładunek") i ethery.

Jeśli konto docelowe zawiera kod, to jest on wykonywany, a ładunek jest przekazywany jako dane wejściowe.

Jeśli konto docelowe nie jest podane (transakcja nie zawiera odbiorcy lub odbiorca ma wartość ``null``),
to taka transakcja tworzy **nowy kontrakt**.

Jak wcześniej wspomniano, adres takiego kontraktu nie jest zerowy, lecz wywodzi się z adresu nadawcy
i liczby wysłanych transakcji (tzw. "nonce"). Ładunek jest uznawany za kod bajtowy EVM i uruchamiany.
Dane wyjściowe są przechowywane na stałe jako kod kontraktu. To znaczy, że aby stworzyć nowy kontrakt,
nie wysyła się faktycznego kodu kontraktu, lecz kod, który zwraca ten kod po uruchomieniu.

.. note::
  W trakcie tworzenia kontraktu jego kod jest nadal pusty.
  Dlatego nie powinieneś wywoływać kontraktu do momentu, aż
  zakończy się wykonywanie kodu konstruktora.

.. index:: ! gas, ! gas price

Paliwo
======

Po utworzeniu, każda transakcja jest obciążana pewną ilością **paliwa**, którego
celem jest ograniczenie nakładu pracy potrzebnej do wykonania transakcji i aby
od razu zapłacić za wykonanie tej transakcji. W trakcie gdy EVM przetwarza
transakcję, paliwo jest stopniowo wyczerpywane według określonych reguł.

**Cena paliwa** jest wartością ustalaną przez twórcę transakcji, który musi z góry
zapłacić ``cena_paliwa * paliwo`` ze swojego konta. Jeśli trochę paliwa zostanie po
wykonaniu transakcji, jest ono w ten sam sposób zwracane do nadawcy.

Jeśli paliwo się zużyje w dowolnej chwili (tzn. będzie ujemne), to wyzwalany jest wyjątek
brak-paliwa, który wycofuje wszystkie zmiany stanu w bieżącej ramce wywołania funkcji.

.. index:: ! storage, ! memory, ! stack

Magazyn, pamięć i stos
======================

Maszyna Wirtualna Ethereum zawiera 3 przestrzenie do przechowywania danych:
magazyn, pamięć i stos, które objaśniono w kolejnych akapitach.

Każde konto ma przestrzeń danych zwaną **magazynem**, który pozostaje stały między wywoływaniami
funkcji i transakcjami.
Magazyn to słownik klucz-wartość, który przypisuje 256-bitowe słowa do 256-bitowych słów.
Nie da się wyliczyć wszystkich kluczy z poziomu kontraktu, ponieważ koszt odczytu danych
jest stosunkowo duży, nawet większy od inicjacji i modyfikacji. Z powodu tego kosztu
powinieneś ograniczyć ilość danych do przechowywania w stałym magazynie do takich,
które kontrakt potrzebuje do wykonania.
Kontrakt nie może ani odczytać, ani zapisać danych do nieswojego magazynu.

Drugą przestrzenią danych jest **pamięć**, czyszczona i tworzona na nowo dla każdej wiadomości.
Pamięć jest liniowa i adresowana na poziomie bajtów. Ograniczono możliwość odczytu do 256 bitów.
Można zapisać albo 8 bitów, albo 256 bitów. Pamięć rozszerza się co słowo (256 bitów) podczas
dostępu (zapisu lub odczytu) do nietkniętych wcześniej słów (tzn. dowolnego przesunięcia słowa).
Rozszerzanie pamięci zużywa paliwo. Tym więcej pamięć kosztuje, im bardziej rośnie (kwadratowo).

EVM wszystkie operacje prowadzi na **stosie**, a nie na rejestrach. Stos może pomieścić maksymalnie
1024 elementy - słowa o długości 256 bitów. Dostęp do stosu jest ograniczony do jego wierzchołka w
następujący sposób:
Można skopiować 1 z najwyższych 16 elementów na wierzch stosu lub zamienić najwyższy element
z jednym z 16 elementów poniżej niego.
Wszystkie inne operacje zdejmują z wierzchu 2 elementy (lub 1, lub więcej, w zależności od operacji)
i kładą na wierzch wynik.
Oczywiście można przenosić elementy stosu do magazynu lub do pamięci,
aby dostać się głębiej do stosu,
ale nie da się bezpośrednio odczytać dowolnego elementu głębiej w stosie
bez wcześniejszego usunięcia elementów z wierzchołka stosu.

.. index:: ! instruction

Zestaw instrukcji
=================

Zestaw instrukcji EVM jest jak najmniejszy, aby uniknąć niepoprawnych lub niespójnych
implementacji, które doprowadziłyby do problemów z osiąganiem konsensusu.
Wszystkie instrukcje operują na podstawowym typie danych, 256-bitowych słowach
lub na fragmentach pamięci (lub innych tablicach bajtowych).
Zwykłe działania arytmetyczne, bitowe, logiczne i porównania są dostępne.
Skoki warunkowe i bezwarunkowe są możliwe. Co więcej, kontrakty mają dostęp
do stosownych właściwości bieżącego bloku, takich jak jego numer i znacznik czasu.

Po kompletny zestaw instrukcji zajrzyj do :ref:`listy kodów operacji <opcodes>`.

.. index:: ! message call, function;call

Wywołania wiadomości
====================

Za pomocą wywołań wiadomości kontrakty mogą wywoływać inne kontrakty i wysyłać ethery do kont niebędących kontraktem.
Wywołania wiadomości przypominają transakcje w tym sensie, że zawierają źródło, cel, dane wejściowe, ethery, paliwo
i dane powrotne. Właściwie każda transakcja składa się z wywołania wiadomości najwyższego poziomu, które z kolei
może tworzyć kolejne wywołania wiadomości.

Kontrakt może decydować, ile pozostałego **paliwa** powinien wysłać z wewnętrznym wywołaniem wiadomości, a ile chce
pozostawić. Jeśli w wewnętrznym wywołaniu wystąpi wyjątek brak-paliwa (lub jakikolwiek inny wyjątek), zostanie to
zasygnalizowane odłożeniem wartości błędu na stosie. W takiej sytuacji tylko paliwo wysłane razem z wywołaniem jest
zużywane. W Solidity kontrakt wywołujący domyślnie rzuca wyjątek ręcznie w takich przypadkach, więc wyjątki są
propagowane w górę na stosie.

Jak już wspomniano, wywoływany kontrakt (który może być taki sam jak kontrakt wywołujący)
dostanie świeżo wyczyszczoną pamięć i ma dostęp do danych wejściowych - które zostaną
przekazane w oddzielnej przestrzeni zwanej **calldata**.
Kiedy zakończy działanie, może zwrócić dane, które zostaną zapisane do pamięci w miejscu
wydzielonym wcześniej przez nadawcę. Wszystkie te wywołania są w pełni synchroniczne.

Wywołania są **ograniczone** do głębokości 1024, co znaczy, że dla bardziej skomplikowanych
operacji preferowane są pętle zamiast rekurencyjnych wywołań. Co więcej, tylko 63/64 części
paliwa można przekazać dalej w wywołaniu wiadomości, co w praktyce ogranicza głębokość
wywołań do mniej niż 1000.

.. index:: delegatecall, callcode, library

Delegatecall / Callcode i biblioteki
====================================

Istnieje specjalna odmiana wywołań wiadomości, zwana **delegatecall**,
która różni się tylko tym, że kod pod adresem docelowym jest wykonywany
w kontekście kontraktu wywołującego i ``msg.sender`` oraz ``msg.value`` nie zmieniają swoich wartości.

To znaczy, że kontrakt może dynamicznie załadować kod z innego adresu w trakcie wykonywania.
Magazyn, bieżący adres i saldo wciąż dotyczą kontraktu wywołującego.
Tylko kod brany jest z wywoływanego adresu.

To umożliwia tworzenie bibliotek w Solidity: kod wielokrotnego użytku, który można
wykorzystać np. do implementacji złożonych struktur danych.

.. index:: log

Dzienniki
=========

Można przechowywać dane w specjalnie indeksowanej strukturze danych,
która przypisuje wszystko do poziomu bloku. Taka funkcjonalność zwana
**dziennikiem** służy w Solidity do implementacji :ref:`zdarzeń <events>`.
Kontrakty nie mają dostępu do dziennika po jego utworzeniu, ale można się
do nich dostać spoza łańcucha bloków.
Ponieważ niektóre fragmenty dziennika są przechowywane w
`filtrach Blooma <https://en.wikipedia.org/wiki/Bloom_filter>`_,
można go przeszukiwać w optymalny i bezpieczny kryptograficznie sposób,
więc węzły sieci, które nie pobiorą całego łańcucha bloków (tzw. lekkie klienty),
wciąż mogą przeszukiwać dziennik.

.. index:: contract creation

Create
======

Kontrakty mogą nawet tworzyć inne kontrakty za pomocą specjalnego kodu operacji (nie wywołują
po prostu adresu zerowego jak w przypadku transakcji). **Wywołania create** różnią się od zwykłych
wywołań wiadomości tylko tym, że ładunek wejściowy jest wykonywany, a wynik zapisywany jako kod
i nadawca / twórca otrzymuje adres kontraktu na stosie.

.. index:: selfdestruct, self-destruct, deactivate

Deactivate i Self-destruct
==========================

Jedynym sposobem na usunięcie kodu z łańcucha bloku jest wykonanie operacji ``selfdestruct``.
Pozostałe ethery przechowywane pod danym adresem są wysyłane pod wskazany adres, a magazyn i kod
są usuwany ze stanu. Usuwanie kontaktów w teorii wydaje się dobrym pomysłem, ale jest potencjalnie
niebezpieczne, ponieważ jeśli ktoś wyśle ethery do usuniętego kontraktu, przepadną na zawsze.

.. warning::
    Nawet jeśli kontrakt zostanie usunięty przez ``selfdestruct``, wciąż jest częścią historii
    łańcucha bloków i prawdopodobnie zachowany przez większość węzłów Ethereum.
    Tak więc użycia ``selfdestruct`` nie można porównać do usuwania plików z dysku twardego.

.. note::
    Nawet jeśli kod kontraktu nie zawiera wywołania ``selfdestruct``, to wciąż
    może wykonać tę operację za pomocą ``delegatecall`` lub ``callcode``.

Jeśli chcesz dezaktywować swoje kontrakty, powinieneś w zamian je **wyłączyć**,
zmieniając wewnętrzny stan w ten sposób, żeby każda funkcja wycofywała zmiany.
Wtedy używanie kontraktu stanie się niemożliwe, a ethery wrócą do nadawcy.

.. index:: ! precompiled contracts, ! precompiles, ! contract;precompiled

.. _precompiledContracts:

Wstępnie skompilowane kontrakty
===============================

Istnieje mały zbiór adresów kontaktów, które są specjalne:
adresy od ``1`` do (wliczając) ``8`` zawierają "wstępnie skompilowane kontakty",
które mogą być wywoływane przez inne kontrakty, ale ich zachowanie (i zużycie paliwa)
nie jest zdefiniowane przez kod EVM przechowywany pod tym adresem (nie zawierają kodu),
ale są zaimplementowane bezpośrednio w środowisku wykonawczym EVM.

Różne łańcuchy kompatybilne z EVM mogą używać odmiennego zestawu wstępne skompilowanych kontraktów.
Może się też zdarzyć, że w przyszłości nowe wstępnie skompilowane kontrakty zostaną dodane do głównego
łańcucha Ethereum, ale możesz się spodziewać, że znajdą się w zakresie od ``1`` and ``0xffff`` (włącznie).
