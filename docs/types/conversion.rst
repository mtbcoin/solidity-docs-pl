.. index:: ! type;conversion, ! cast

.. _types-conversion-elementary-types:

Konwersje między typami podstawowymi
====================================

Konwersje niejawne
------------------

Kompilator w niektórych przypadkach dokonuje niejawnej konwersji typów - podczas
przypisań, przekazywania argumentów do funkcji i stosowania operatorów.
Zasadniczo niejawna konwersja między typami podstawowymi jest możliwa,
jeśli ma ona sens znaczeniowo i żadne informacje nie zostaną utracone.

Na przykład ``uint8`` da się skonwertować do ``uint16``, zaś ``int128``
do ``int256``, ale ``int8`` nie da się skonwertować do ``uint256``,
ponieważ ``uint256`` nie może przechowywać wartości takich jak ``-1``.

Jeśli stosujemy operator do odmiennych typów, kompilator próbuje niejawnie
skonwertować jeden z operandów do typu drugiego operandu (to samo dotyczy przypisań).
To znaczy, że operacje są zawsze wykonywane w typie jednego z tych operandów.

Aby dowiedzieć się więcej, jakie niejawne konwersje są możliwe,
przeczytaj rozdziały o poszczególnych typach danych.

W poniższym przykładzie ``y`` i ``z``, operandy dodawania, mają różne typy,
ale ``uint8`` można niejawnie skonwertować do ``uint16``. Natomiast nie da
się dokonać konwersji na odwrót. Dlatego przed wykonaniem dodawania typ ``y``
jest konwertowany do typu ``z``. Typ wynikowy wyrażenia ``y + z`` to ``uint16``.
Ponieważ wynik operacji przypisujemy do zmiennej typu ``uint32``,
następuje kolejna niejawna konwersja typów.

.. code-block:: solidity

    uint8 y;
    uint16 z;
    uint32 x = y + z;


Konwersje jawne
---------------

Jeśli kompilator nie pozwala na niejawną konwersję typów, ale jesteś pewny,
że konwersja zadziała, możesz dokonać jawnej konwersji. Może ona zaowocować
błędnym zachowaniem kontraktu. Pozwala też obejść niektóre mechanizmy
bezpieczeństwa kompilatora, więc pamiętaj, aby przetestować, czy
wynik jest zgodny z twoimi oczekiwaniami!

Spójrz na poniższy przykład, który konwertuje negatywny ``int`` do ``uint``:

.. code-block:: solidity

    int  y = -3;
    uint x = uint(y);

Pod koniec tego fragmentu kodu ``x`` będzie miał wartość ``0xfffff..fd`` (64 znaki hex), 
co daje -3 w kodzie uzupełnień do dwóch liczby 256-bitowej.

Jeśli liczba całkowita jest jawnie konwertowana do mniejszego typu,
najbardziej znaczące bity są obcinane:

.. code-block:: solidity

    uint32 a = 0x12345678;
    uint16 b = uint16(a); // b will be 0x5678 now

Jeśli liczba całkowita jest jawnie konwertowana do większego typu, to po lewej
stronie są dodawane zera (tzn. po stronie najbardziej znaczących bitów).
Wynik konwersji będzie równy oryginalnej liczbie całkowitej:

.. code-block:: solidity

    uint16 a = 0x1234;
    uint32 b = uint32(a); // b będzie wynosić 0x00001234
    assert(a == b);

Typy bajtowe o stałym rozmiarze zachowują się inaczej podczas konwersji.
Można o nich myśleć jak o ciągach pojedynczych bajtów i konwersja do
mniejszego typu spowoduje obcięcie ciągu:

.. code-block:: solidity

    bytes2 a = 0x1234;
    bytes1 b = bytes1(a); // b będzie zawierać 0x12

Jeśli typ bajtowy o stałym rozmiarze jest jawnie konwertowany do większego typu,
wtedy jest dopełniany zerami po prawej stronie. Jeśli odczytamy bajt o określonym
indeksie, uzyskamy taką samą wartość jak przed konersją (jeśli indeks wciąż jest w zakresie).

.. code-block:: solidity

    bytes2 a = 0x1234;
    bytes4 b = bytes4(a); // b będzie zawierać 0x12340000
    assert(a[0] == b[0]);
    assert(a[1] == b[1]);

Ponieważ liczby całkowite i tablice bajtów o stałym rozmiarze zachowują się
odmiennie podczas skracania lub rozszerzania, jawne konwersje między nimi są
dozwolone tylko w sytuacji, kiedy oba mają ten sam rozmiar. Aby skonwertować
typy między różniącymi się długością liczbami całkowitymi a tablicami bajtów
o stałym rozmiarze, musisz dokonać pośrednich konwersji:

.. code-block:: solidity

    bytes2 a = 0x1234;
    uint32 b = uint16(a); // b wyniesie 0x00001234
    uint32 c = uint32(bytes4(a)); // c wyniesie 0x12340000
    uint8 d = uint8(uint16(a)); // d wyniesie 0x34
    uint8 e = uint8(bytes1(a)); // e wyniesie 0x12

Tablice ``bajtów<bytes>`` i ``bajtowe<bytes>`` wycinki calldata można konwertować jawnie do typów bajtowycch o stałej długości (``bytes1``/.../``bytes32``).
Jeśli tablica jest dłuższa od docelowego typu bajtowego o stałej długości, zostanie obcięta na końcu.
Jeśli tablica jest krótsza od typu docelowego, zostanie na końcu dopełniona zerami.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.5;

    contract C {
        bytes s = "abcdefgh";
        function f(bytes calldata c, bytes memory m) public view returns (bytes16, bytes3) {
            require(c.length == 16, "");
            bytes16 b = bytes16(m);  // jeśli m jest dłuższy od 16, zostanie obcięty
            b = bytes16(s);  // dopełniony po prawej, wynik to "abcdefgh\0\0\0\0\0\0\0\0"
            bytes3 b1 = bytes3(s); // obcięty, b1 to "abc"
            b = bytes16(c[:8]);  // też dopełniony zerami
            return (b, b1);
        }
    }

.. _types-conversion-literals:

Konwersje między literałami a typami podstawowymi
=================================================

Typy całkowite
--------------

Literały liczbowe dziesiętne i szestnastkowe mogą być niejawnie konwertowane do dowolnej
liczby całkowitej, która jest wystarczająco duża, aby je reprezentować bez obcinania:

.. code-block:: solidity

    uint8 a = 12; // dobrze
    uint32 b = 1234; // dobrze
    uint16 c = 0x123456; // źle, ponieważ wartość zostałaby obcięta do 0x3456

.. note::
    Do wersji 0.8.0 można było jawnie skonwertować dowolny literał liczbowy dziesiętny
	lub szestnastkowy skonwertowany do typu całkowitego. Od 0.8.0 takie jawne konwersje 
	są tak samo rygorystyczne jak niejawne konwersje, tzn. można ich dokonać tylko wtedy,
	gdy literał mieści się w zakresie wynikowym.

Tablice bajtów o stałej długości
------------------------------

Literałów liczb dziesiętnych nie można niejawnie konwertować do tablic bajtów o stałej długości.
Literały liczb szesnastkowych można w ten sposób konwertować, ale tylko wtedy, gdy ilość cyfr
szestnastkowych zgadza się z rozmiarem typu tablicy bajtów. Istnieje wyjątek, że zarówno literały
całkowite jak i szesnastkowe o wartości 0 można skonwertować do dowolnej tablicy bajtów o stałej długości:

.. code-block:: solidity

    bytes2 a = 54321; // niedozwolone
    bytes2 b = 0x12; // niedozwolone
    bytes2 c = 0x123; // niedozwolone
    bytes2 d = 0x1234; // ok
    bytes2 e = 0x0012; // ok
    bytes4 f = 0; // ok
    bytes4 g = 0x0; // ok

Literały tekstowe mogą być niejawnie konwertowane do tablic bajtów o stałej długości,
jeżeli liczba znaków odpowiada długości tablicy:

.. code-block:: solidity

    bytes2 a = hex"1234"; // ok
    bytes2 b = "xy"; // ok
    bytes2 c = hex"12"; // niedopuszczalne
    bytes2 d = hex"123"; // niedopuszczalne
    bytes2 e = "x"; // niedopuszczalne
    bytes2 f = "xyz"; // niedopuszczalne

Adresy
------

Jak opisano w rozdziale :ref:`literały adresowe<address_literals>`, szestnastkowe literały o odpowieniej długości, które przechodzą test sumy kontrolnej, są typu ``address``. Innych literałów nie można niejawnie skonwertować do typu ``address``.

Jawna konwersja ``bytes20`` lub dowolnej liczby całkowitej do ``address`` zwraca typ ``address payable``.

``address a`` można skonwertować do ``address payable`` funkcją ``payable(a)``.
