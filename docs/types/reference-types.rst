.. index:: ! type;reference, ! reference type, storage, memory, location, array, struct

.. _reference-types:

Typy referencyjne
=================

Wartości typów referencyjnych można modyfikować przez wiele różnych nazw.
Skontrastuj to z typami wartościowymi, gdzie uzyskujesz niezależną kopię,
gdziekolwiek ich używasz. Dlatego należy ostrożniej obchodzić się
z referencjami niż z typami wartościowymi. Aktualnie do typów
referencyjnych zaliczamy struktury, tablice i mapy.

Jeśli używasz referencji, zawsze powinieneś jawnie określić, w jakiej
przestrzeni danych informacje powinny być przechowywane: 
``memory`` (istnieje tylko przez czas wywołania zewnętrznej funkcji),
``storage`` (zawiera zmienne stanu, istnieje do końca życia kontraktu)
lub ``calldata`` (specjalna przestrzeń zawierająca argumenty funcji).

Przypisanie lub konwersja typu, która zmienia lokalizację danych, zawsze powoduje stworzenie kopii, podczas gdy przypisania w obrębie tej samej lokaliacji tworzą kopię jedynie w określonych przypadkach dla typów magazynowych.

.. _data-location:

Lokalizacja danych
------------------

Każdy typ referencyjny zawiera dodatkową adnotację - "lokalizację danych" - gdzie są one
przechowywane. Wyróżniamy trzy lokalizacje danych: ``memory``, ``storage`` i ``calldata``. Calldata to niemodyfikowalna, nietrwała przestrzeń, gdzie są przechowywane argumenty funkcji. Zachowuje się zazwyczaj tak jak memory.

.. note::
    Jeśli możesz, to używaj ``calldata`` jako lokalizacji danych, ponieważ unikniesz kopiowania
	i dodatkowo masz pewność, że dane nie zostaną zmodyfikowane. Tablice i struktury w lokalizacji
	``calldata`` również mogą być zwracane przez funkcje, ale nie da się przydzielać tych typów.

.. note::
	Do wersji 0.6.9 argumenty o typach referencyjnych mogły być przechowywane jedynie
	w ``calldata`` w funkcjach zewnętrznych, ``memory`` w funkcjach publicznych i w 
	``memory`` lub ``storage`` w funkcjach wewnętrznych i prywatnych.
	Teraz ``memory`` i ``calldata`` są dopuszczalne we wszystkich funkcjach
	niezależnie od ich widoczności.

.. note::
	Do wersji 0.5.0 można było pominąć lokalizację danych. Była ustawiana domyślnie
	w zależności od typu zmiennej, typu funkcji, itd., ale wszystkie typy złożone
	obecnie muszą mieć jawnie określoną lokalizację danych.

.. _data-location-assignment:

Lokalizacja danych i przypisania
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Miejsce przechowywania danych nie zależy wyłącznie od ich trwałości, ale także od semantyki przypisań:

* Przypisania między ``storage`` i ``memory`` (lub z ``calldata``)
  zawsze tworzą niezależną kopię.
* Przypisania z ``memory`` do ``memory`` tylko tworzą referencje. To znaczy, że
  zmiany w jednej zmiennej memory są widoczne we wszystkich innych zmiennych memory, któe
  odnoszą się do tych samych wartości.
* Przypisania z ``storage`` do **local** storage także tworzą referencję.
* Wszystkie inne przypisania z ``storage`` zawsze tworzą kopię. Przykłady
  to przypisania do zmiennych stanu lub do pól zmiennych lokalnych typów
  strukturalnych z modyfikatorem storage, nawet jeśli zmienna lokalna
  sama jest po prostu referencją.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.5.0 <0.9.0;

    contract C {
        // Lokalizacją x jest magazyn.
        // To jedyne miejsce, gdzie lokalizację
        // danych można pominąć.
        uint[] x;

        // Lokalizacją memoryArray jest pamięć.
        function f(uint[] memory memoryArray) public {
            x = memoryArray; // działa, kopiuje całą tablicę do magazynu
            uint[] storage y = x; // działa, przypisuje wskaźnik, lokalizacją y jest magazyn
            y[7]; // ok, zwraca ósmy element
            y.pop(); // ok, modyfikuje x poprzez y
            delete x; // ok, czyści tablicę, modyfikuje także y
            // Poniższy kod nie zadziała; musielibyśmy stworzyć nową tymczasową /
            // nienazwaną tablicę w magazynie, ale magazyn jest "statyczne" alokowany:
            // y = memoryArray;
            // To też nie zadziała, ponieważ wskaźnik zostałby "zresetowany", ale nie istnieje
            // sensowna lokalizacja, do której mógłby się odnosić.
            // delete y;
            g(x); // wywołuje g i przekazuje referencję do x
            h(x); // wywołuje h i tworzy niezależną, tymczasową kopię w pamięci
        }

        function g(uint[] storage) internal pure {}
        function h(uint[] memory) public pure {}
    }

.. index:: ! array

.. _arrays:

Tablice
-------

Tablice mogą mieć stały (znany podczas kompilacji) lub dynamiczny rozmiar.

Typ tablicy o stałym rozmiarze ``k`` i typie elementów ``T`` zapisuje się jako ``T[k]``,
natomiast tablicy o dynamicznym rozmiarze jako ``T[]``.

Na przykład tablicę pięciu dynamicznych tablic typu ``int`` zapisuje się jako
``uint[][5]``. Notacja jest odwrotna w odróżnieniu od wielu innych języków.
W Solidity ``X[3]`` to zawsze tablica zawierająca 3 elementy typu ``X``,
nawet jeśli ``X`` jest tablicą. Tak nie jest w innych językach takich jak C.

Indeksy zaczynają się od zera, a dostęp jest w przeciwnym kierunku do deklaracji.

Na przykład jeśli istnieje zmienna ``uint[][5] memory x``, dostęp do siódmego
elementu ``uint`` w trzeciej dynamicznej tablicy uzyskasz za pomocą ``x[2][6]``,
natomiast do trzeciej dynamicznej tablicy za pomocą ``x[2]``. Jeśli mamy tablicę
``T[5] a`` typu ``T``, który także może być tablicą, wtedy ``a[2]`` zawsze ma typ ``T``.

Elementy tablic mogą być dowolnego typu, także mapami lub strukturami.
Obowiązują ogólne ograniczenia dla typów, w tym że mapy można przechowywać
wyłącznie w lokalizacji ``storage``, a publicznie widoczne funkcje muszą
mieć parametry, które są :ref:`typami ABI <ABI>`.

Można oznaczyć tablice zmiennych stanu jako ``public``, aby Solidity stworzył dla nich :ref:`gettery <visibility-and-getters>`.
Wymaganym parametrem dla getterów jest liczbowy indeks.

Próba dostępu do elementu tablicy spoza zakresu powoduje błąd typu failing assertion. Metody ``.push()`` i ``.push(value)`` służą do dodawania nowych elementów na koniec tablicy, gdzie ``.push()`` dodaje element zainicjowany zerami i zwraca referencję do niego.

.. index:: ! string, ! bytes

.. _strings:

.. _bytes:

``bytes`` i ``string`` jako tablice
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Zmienne typu ``bytes`` i ``string`` są specjalnymi tablicami. Typ ``bytes`` jest podobny do ``bytes1[]``,
ale ściśle upakowany w calldata i pamięci. ``string`` to odpowiednik ``bytes``, ale nie pozwala na dostęp poprzez indeks ani odczyt długości.

Solidity nie zawiera funkcji do operowania na ciągach tekstowych, ale istnieją biblioteki
stworzone przez osoby trzecie. Możesz też porównywać dwa ciągi tekstowe, porównując ich
skróty keccak256 ``keccak256(abi.encodePacked(s1)) == keccak256(abi.encodePacked(s2))`` 
i łączyć oba ciągi tekstowe za pomocą ``string.concat(s1, s2)``.

Powinieneś używać ``bytes`` zamiast ``bytes1[]``, ponieważ są tańsze,
gdyż ``bytes1[]`` w ``memory`` dodaje 31 bajtów wypełniających między elementami. Zwróć uwagę, że w ``storage`` nie ma wypełnienia ze względu na ścisłe upakowanie danych, patrz :ref:`bajty i ciągi tekstowe <bytes-and-string>`. Zasadniczo używaj ``bytes`` dla surowych danych o dowolnej długości oraz ``string`` dla ciągów tekstowych (UTF-8) o dowolnej długości. Jeśli możesz ograniczyć długość do określonej liczby bajtów, zawsze stosuj typy od ``bytes1`` do ``bytes32`` ponieważ są one dużo tańsze.

.. note::
    Jeśli chcesz uzyskać reprezentację bajtową ciągu tekstowego ``s``,
	wywołaj ``bytes(s).length`` / ``bytes(s)[7] = 'x';``. Miej na uwadze,
	że operujesz niskopoziomowo na bajtach reprezentacji UTF-8, a nie na
	pojedynczych znakach.

.. index:: ! bytes-concat, ! string-concat

.. _bytes-concat:
.. _string-concat:

Funkcje ``bytes.concat`` i ``string.concat``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Możesz łączyć dowolną ilość wartości ``string`` za pomocą ``string.concat``.
Ta funkcja zwraca pojedynczą tablicę ``string memory`` zawierającą zawartość argumentów bez wypełnienia.
Jeśli chcesz podać parametry innego typu, które nie są niejawnie konwertowane do ``string``, musisz najpierw je ręcznie skonwertować do ``string``.

Analogicznie funkcja ``bytes.concat`` może złączyć dowolną liczbę wartości ``bytes`` lub ``bytes1 ... bytes32``.
Zwraca ona pojedynczą tablicę ``bytes memory`` zawierającą zawartość argumentów bez wypełnienia.
Jeśli chcesz podać parametry typu string lub innych typów, które nie są niejawnie konwertowane do``bytes``, musisz najpier jawnie je skonwertować do ``bytes`` lub ``bytes1``/.../``bytes32``.


.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.12;

    contract C {
        string s = "Storage";
        function f(bytes calldata bc, string memory sm, bytes16 b) public view {
            string memory concatString = string.concat(s, string(bc), "Literal", sm);
            assert((bytes(s).length + bc.length + 7 + bytes(sm).length) == bytes(concatString).length);

            bytes memory concatBytes = bytes.concat(bytes(s), bc, bc[:2], "Literal", bytes(sm), b);
            assert((bytes(s).length + bc.length + 2 + 7 + bytes(sm).length + b.length) == concatBytes.length);
        }
    }

Jeśli wywołasz ``string.concat`` lub ``bytes.concat`` bez argumentów, zwrócą pustą tablicę.

.. index:: ! array;allocating, new

Przydzielanie tablic w pamięci
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Tablice w pamięci o dynamicznej długości można tworzyć za pomocą operatora ``new``.
W odróżnieniu od tablic w magazynie, **nie** da się zmienić rozmiaru tablic w pamięci (np.
metoda ``.push`` jest niedostępna).
Musisz albo z góry policzyć rozmiar, albo utworzyć nową tablicę w pamięci i skopiować każdy element.

Tak jak wszystkie zmienne w Solidity, elementy nowo utworzonych tablic podczas alokacji otrzymują :ref:`wartość domyślną<default-value>`.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.16 <0.9.0;

    contract C {
        function f(uint len) public pure {
            uint[] memory a = new uint[](7);
            bytes memory b = new bytes(len);
            assert(a.length == 7);
            assert(b.length == len);
            a[6] = 8;
        }
    }

.. index:: ! array;literals, ! inline;arrays

Literały tablicowe
^^^^^^^^^^^^^^^^^^

Literały tablicowe to oddzielone przecinkami listy jedneg lub wielu wyrażeń,
zamknięte w nawiasach kwadratowych (``[...]``). Na przykład ``[1, a, f(3)]``. 
Typ literału tablicowego jest wyznaczany w następujący sposób:

To jest zawsze tablica w pamięci o stałym rozmiarze, której długość
określa liczba wyrażeń.

Typem bazowym tablicy jest typ pierwszego wyrażenia na liście. Pozostałe
wyrażenia mogą być do niego niejawnie skonwertowane. Jeśli się nie da, to
wystąpi błąd typów.

Nie wystarczy, że istnieje typ, do którego da się skonwertować wszystkie elementy. Jeden z elementów musi być tego konkretnego typu.

W poniższym przykładzie ``[1, 2, 3]`` jest typu
``uint8[3] memory``, ponieważ typ każdego elementu listy to ``uint8``.
Jeśli chcesz uzyskać typ ``uint[3] memory``, musisz najpierw skonwertować
pierwszy element do ``uint``.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.16 <0.9.0;

    contract C {
        function f() public pure {
            g([uint(1), 2, 3]);
        }
        function g(uint[3] memory) public pure {
            // ...
        }
    }

Literał tablicowy ``[1, -1]`` jest niepoprawny, ponieważ typ pierwszego elementu to ``uint8``,
zaś drugiego to ``int8`` i nie mogą być niejawnie skonwertowane żadnego z nich. Musisz jawne dokonać konwersji na przykład tak: ``[int8(1), -1]``.

Ponieważ nie można skonwertować między sobą tablic w pamięci o stałej wielkości, które mają różne typy (nawet jeśli można skonwertować typy bazowe), zawsze musisz jawnie określić wspólny typ bazowy, jeśli chcesz używać dwuwymiarowych literałów tablic:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.16 <0.9.0;

    contract C {
        function f() public pure returns (uint24[2][4] memory) {
            uint24[2][4] memory x = [[uint24(0x1), 1], [0xffffff, 2], [uint24(0xff), 3], [uint24(0xffff), 4]];
            // Poniższy kod nie zadziała, ponieważ typ niektórych wewnętrznych tablic jest nieprawidłowy.
            // uint[2][4] memory x = [[0x1, 1], [0xffffff, 2], [0xff, 3], [0xffff, 4]];
            return x;
        }
    }

Tablic w pamięci o stałym rozmiarze nie można przypisać do dynamicznie
rozszerzalnych tablic w pamięci, np. poniższe jest niemożliwe:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

    // To się nie skompiluje.
    contract C {
        function f() public {
            // Kolejna linia powoduje błąd typu, ponieważ uint[3] memory
            // nie można skonwertować do uint[] memory.
            uint[] memory x = [uint(1), 3, 4];
        }
    }

Planowane jest usunięcie tego ograniczenia w przyszłości, ale tworzy to
pewne kompilacje przez sposób, w jaki tablice są przekazywane w ABI.

Aby zainicjować dynamicznie rozszerzalną tablicę, musisz przypisać wartości
poszczególnym elementom:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.16 <0.9.0;

    contract C {
        function f() public pure {
            uint[] memory x = new uint[](3);
            x[0] = 1;
            x[1] = 3;
            x[2] = 4;
        }
    }

.. index:: ! array;length, length, push, pop, !array;push, !array;pop

.. _array-members:

Własności tablic
^^^^^^^^^^^^^^^^

**length**:
    Tablice mają własność ``length``, która zawiera liczbę ich elementów.
	Rozmiar tablicy w pamięci jest stały (ale dynamiczny, tzn. zależy to od
	parametrów środowiska) po ich utworzeniu.
**push()**:
     Dynamiczne tablice w magazynie oraz ``bytes`` (ale nie ``string``) mają metodę
	 zwaną ``push()``, która dodaje element zainicjowany zerami na koniec tablicy.
	 Zwraca referencję do tego elementu, więc można jej użyć w ten sposób:
     ``x.push().t = 2`` lub ``x.push() = b``.
**push(x)**:
     Dynamiczne tablice w magazynie oraz ``bytes`` (ale nie ``string``) mają metodę
	 zwaną ``push(x)``, która dodaje przekazany element jako argument na koniec tablicy.
     Funkcja nic nie zwraca.
**pop()**:
     Dynamiczne tablice w magazynie oraz ``bytes`` (ale nie ``string``) mają metodę
	 zwaną ``pop()``, która usuwa element z końca tablicy. Wywołuje ona też niejawnie 
	 :ref:`delete<delete>` na usuwanym elemencie.

.. note::
    Zwiększenie długości tablicy w magazynie poprzez wywołanie ``push()``
    kosztuje tyle samo paliwa, ponieważ magazyn jest zainicjowany
	zerami, natomiast koszt zmniejszenia długości za pomocą ``pop()`` 
    zależy od rozmiaru usuwanego elementu.
	Jeśli ten element jest tablicą, może to być kosztowne, ponieważ
	koszt uwzględnia czyszczenie usuwanego elementu podobnie jakby
    wywołać na nim :ref:`delete<delete>`.

.. note::
    Aby zanieżdżać tablice w zewnętrznych (zamiast publicznych) funkcjach,
	musisz aktywować koder ABI v2.

.. note::
    W wersjach EVM wcześniejszych od Byzantium nie był możliwy dostęp
	do dynamicznych tablic zwracanych z wywołań funkcji. Jeśli wywołujesz
	funkcje, które zwracają dynamiczne tablice, upewnij się, że tryb
	Byzantium w EVM jest włączony.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.6.0 <0.9.0;

    contract ArrayContract {
        uint[2**20] aLotOfIntegers;
		// Zauważ, że to nie jest para dynamicznych tablic, lecz
		// dynamiczna tablica par (tablic o stałym rozmiarze równym 2).
		// Dlatego T[] zawsze jest dynamiczną tablicą T, nawet jeśli T
		// jest sam w sobie tablicą.
		// Lokalizacją wszystkich zmiennych stanu jest magazyn.
        bool[2][] pairsOfFlags;

        // newPairs jest przechowywany w pamięci - to jedyna opcja
        // dla argumentów publicznych funkcji w kontrakcie
        function setAllFlagPairs(bool[2][] memory newPairs) public {
            // przypisanie do tablicy w magazynie tworzy kopię ``newPairs``
            // i zamienia całą tablicę ``pairsOfFlags``.
            pairsOfFlags = newPairs;
        }

        struct StructType {
            uint[] contents;
            uint moreInfo;
        }
        StructType s;

        function f(uint[] memory c) public {
            // przechowuje referencję do ``s`` w ``g``
            StructType storage g = s;
            // modyfikuje także ``s.moreInfo``.
            g.moreInfo = 2;
            // przypisuje kopię, ponieważ ``g.contents``
            // nie jest zmienną lokalną, lecz polem
            // zmiennej lokalnej.
            g.contents = c;
        }

        function setFlagPair(uint index, bool flagA, bool flagB) public {
            // odwołanie do nieistniejącego indeksu powoduje rzucenie wyjątku
            pairsOfFlags[index][0] = flagA;
            pairsOfFlags[index][1] = flagB;
        }

        function changeFlagArraySize(uint newSize) public {
            // użycie push i pop to jedyny sposób na zmianę długości tablicy
            if (newSize < pairsOfFlags.length) {
                while (pairsOfFlags.length > newSize)
                    pairsOfFlags.pop();
            } else if (newSize > pairsOfFlags.length) {
                while (pairsOfFlags.length < newSize)
                    pairsOfFlags.push();
            }
        }

        function clear() public {
            // całkowicie czyszczą tablice
            delete pairsOfFlags;
            delete aLotOfIntegers;
            // taki sam efekt jak powyżej
            pairsOfFlags = new bool[2][](0);
        }

        bytes byteData;

        function byteArrays(bytes memory data) public {
            // tablice bajtów ("bytes") są przechowywane bez wypełnienia,
            // ale można je traktować tak samo jak "uint8[]"
            byteData = data;
            for (uint i = 0; i < 7; i++)
                byteData.push();
            byteData[3] = 0x08;
            delete byteData[2];
        }

        function addFlag(bool[2] memory flag) public returns (uint) {
            pairsOfFlags.push(flag);
            return pairsOfFlags.length;
        }

        function createMemoryArray(uint size) public pure returns (bytes memory) {
            // Dynamiczne tablice w pamięci tworzy się słowem kluczowym `new`:
            uint[2][] memory arrayOfPairs = new uint[2][](size);

            // Tablice inline mają zawsze stały rozmiar i jeśli podajesz tylko literał,
            // to musisz określić przynajmniej 1 typ.
            arrayOfPairs[0] = [uint(1), 2];

            // Tworzy dynamiczną tablicę bajtów:
            bytes memory b = new bytes(200);
            for (uint i = 0; i < b.length; i++)
                b[i] = bytes1(uint8(i));
            return b;
        }
    }

.. index:: ! array;slice

.. _array-slices:

Wycinki tablic
--------------

Wycinki tablic to widoki (perspektywy) umożliwiające na dostęp
do fragmentu tablicy. Oznacza się je jako ``x[start:end]``,
gdzie ``start`` i ``end`` to wyrażenia typu uint256 (lub
niejawnie konwertowalne do niego). Pierwszy element wycinka
to ``x[start]``, a ostatni element to ``x[end - 1]``.

Jeśli ``start`` jest większy od ``end`` lub ``end`` jest większy
od długości tablicy, to rzucany jest wyjątek.

Zarówno  ``start`` jak i ``end`` są opcjonalne: ``start`` domyślnie
przyjmuje ``0``, natomiast ``end`` jest równy długości tablicy.

Wycinki tablic nie mają żadnych własności. Są niejawnie
konwertowalne do tablic takiego samego typu, co oryginalna
tablica i można odwoływać się do ich elementów po indeksie.

Nie istnieje typ danych dla wycinków tablic. To znaczy, że
nie da się zadeklarować zmiennej typu wycinek tablicy.
Istnieją one tylko w wyrażeniach pośrednich.

.. note::
    Dotychczas wycinki tablic zaimplementowano tylko dla tablic calldata.

Wycinki tablic wykorzystuje się przy dekodowaniu danych przekazanych w argumentach
funkcji z wykorzystaniem interfejsu ABI:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.8.5 <0.9.0;
    contract Proxy {
        /// @dev Adres kontraktu klienta zarządzany przez pośrednika, np. przez ten kontrakt
        address client;

        constructor(address client_) {
            client = client_;
        }

        /// Przekazuje wywołanie do "setOwner(address)" która jest zaimplementowana
		/// przez klienta po dokonaniu podstawowej weryfikacji adresu na wejściu.
        function forward(bytes calldata payload) external {
            bytes4 sig = bytes4(payload[:4]);
            // Ze względu na przycinanie danych, bytes4(payload) daje taki sam wynik.
            // bytes4 sig = bytes4(payload);
            if (sig == bytes4(keccak256("setOwner(address)"))) {
                address owner = abi.decode(payload[4:], (address));
                require(owner != address(0), "Adres właściciela nie może być zerowy.");
            }
            (bool status,) = client.delegatecall(payload);
            require(status, "Przekazanie wywołania nie powiodło się.");
        }
    }



.. index:: ! struct, ! type;struct

.. _structs:

Struktury
--------

Solidity umożliwia definiowanie nowych typów w postaci struktur, co pokazano
w poniższym przykładzie:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.6.0 <0.9.0;

    // Tworzy nowy typ z dwoma polami.
	// Deklaracje utworzone poza kontraktem można
	// dzielić między wieloma kontraktami.
	// Tutaj nie jest to konieczne.
    struct Funder {
        address addr;
        uint amount;
    }

    contract CrowdFunding {
	    // Struktury można także definiować wewnątrz kontraktów. Są one
		// wtedy widoczne tylko dla kontraktu i pochodnych kontraktów.
        struct Campaign {
            address payable beneficiary;
            uint fundingGoal;
            uint numFunders;
            uint amount;
            mapping (uint => Funder) funders;
        }

        uint numCampaigns;
        mapping (uint => Campaign) campaigns;

        function newCampaign(address payable beneficiary, uint goal) public returns (uint campaignID) {
            campaignID = numCampaigns++; // campaignID jest zmienną zwracaną
            // Nie możemy napisać "campaigns[campaignID] = Campaign(beneficiary, goal, 0, 0)"
            // ponieważ wyrażenie po prawej stronie tworzy strukturę w pamięci "Campaign"
			// która zawiera mapę.
            Campaign storage c = campaigns[campaignID];
            c.beneficiary = beneficiary;
            c.fundingGoal = goal;
        }

        function contribute(uint campaignID) public payable {
            Campaign storage c = campaigns[campaignID];
			// Tworzy nową tymczasową strukturę w pamięci, wypełnioną podanymi wartośćiami
			// i kopiuje ją do magazynu.
			// Możesz też napisać Funder(msg.sender, msg.value) w celu inicjacji.
            c.funders[c.numFunders++] = Funder({addr: msg.sender, amount: msg.value});
            c.amount += msg.value;
        }

        function checkGoalReached(uint campaignID) public returns (bool reached) {
            Campaign storage c = campaigns[campaignID];
            if (c.amount < c.fundingGoal)
                return false;
            uint amount = c.amount;
            c.amount = 0;
            c.beneficiary.transfer(amount);
            return true;
        }
    }

Kontrakt nie dostarcza pełnej funkcjonalności zbiórki społecznościowej,
ale zawiera podstawowe zagadnienia potrzebne do zrozumienia struktur.
Typów strukturalnych można używać w mapach i tablicach, a także same
w sobie mogą zawierać mapy i tablice.

Struktura nie może zawierać pola swojego typu, chociaż sama w sobie
może być typem wartości w mapie i może zawierać dynamicznie rozszerzalną
tablicę elementów o takim samym typie. Takie ograniczenie jest potrzebne,
ponieważ rozmiar struktury musi być skończony.

Zobacz, w jaki sposób we wszystkich powyższych funkcjach typ strukturalny
jest przypisywany do zmiennej lokalnej z lokalizacją danych ``storage``.
Nie kopiujemy struktury, lecz przechowujemy referencję, więc przypisania
do pól tej zmiennej lokalnej faktycznie powodują modyfikację stanu.

Oczywiście możesz też bezpośrenio odwołać się do pól struktury bez
przypisywania jej do zmiennej lokalnej, tak jak w
``campaigns[campaignID].amount = 0``.

.. note::
    Do Solidity 0.7.0 struktury w pamięci zawierające pola typów tylko magazynowych (np. mapy)
	były dozwolone i przypisania typu``campaigns[campaignID] = Campaign(beneficiary, goal, 0, 0)``
    w powyższym przykładzie by działały i po cichu takie pola byłby pomijane.
