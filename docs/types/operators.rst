.. index:: ! operator

Operatory
=========

Arytmetycznych i bitowych operatorów można użyć nawet wtedy, gdy oba operandy mają odmienne typy.
Na przykład można policzyć ``y = x + z``, gdzie ``x`` jest typu ``uint8``, zaś ``z`` ma typ 
``int32``. W takich sytuacjach do ustalenia typu, w jakim wykonywane jest działanie (to jest ważne w przypadku przepełnienia) i jaki typ jest zwracany, stosuje się poniższy mechanizm:

1. Jeśli typ operandu po prawej stronie można niejawnie skonwertować do typu operandu po lewej stronie, to użyj typu lewego operandu.
2. Jeśli typ operandu po lewej stronie można niejawnie skonwertować do typu operandu po prawej stronie, to użyj typu prawego operandu.
3. Inaczej działanie jest niedozwolone.

Jeśli jeden z operandów jest :ref:`literałem liczbowym <rational_literals>`, to jest najpierw konwertowany do "typu mobilnego", który jest najmniejszym typem mogącym pomieścić wartość 
(typy bez znaku tej samej długości uznaje się za "mniejsze" od typów ze znakiem).
Jeśli oba są literałami liczbowymi, to działanie jest wykonywane z nieograniczoną precyzją.

Typ wynikowy operatora jest taki sam jak typ, na jakim wykonywane jest działanie,
oprócz operatorów porównania, które zawsze zwracają typ ``bool``.

Operatory ``**`` (potęgowanie), ``<<``  i ``>>`` używają typu lewego operandu
zarówno dla działania, jak i dla wyniku.

Potrójny Operator
-----------------
Potrójny operator występuje w wyrażeniach typu ``<wyrażenie> ? <wyrażenieGdyPrawda> : <wyrażenieGdyFałsz>``.
Zwraca jedno z dwóch wyrażeń w zależności od wyniku głównego ``wyrażenia<expression>``.
Jeśli ``wyrażenie<expression>`` osiągnie wartość ``true``, wtedy ``<wyrażenieGdyPrawda>`` zostanie wykonane, w przeciwnym przypadku``<wyrażenieGdyFałsz>``.

Wynik potrójnego operatora nie ma typu wymiernego, nawet jeśli wszystkie jego operandy są literałami
liczb wymiernych.
Typ wynikowy jest wyznaczany na podstawie typów obu operandów w taki sam sposób jak powyżej, konwertując je najpierw do typów mobilnych, jeśli jest to wymagane.

W konsekwencji ``255 + (true ? 1 : 0)`` spowoduje wycofanie zmian z powodu przekroczenia zakresu, ponieważ ``(true ? 1 : 0)`` jest typu ``uint8``, co wymusza wykonanie dodawania w typie ``uint8``, a liczba 256 przekracza dopuszczalny zakres dla tego typu.

Inny przykład: wyrażenie typu ``1.5 + 1.5`` jest poprawne, ale ``1.5 + (true ? 1.5 : 2.5)`` już nie, ponieważ pierwsze wyrażenie jest wyrażeniem wymiernym o dowolnej precyzji i bierzemy pod uwagę tylko ostateczny wynik. Drugie wyrażenie pociąga konwersję ułamkowej liczby wymiernej do całkowitej. Jest to obecnie niedopuszczalne.

.. index:: assignment, lvalue, ! compound operators

Operatory złożone i inkrementacji/dekrementacji
-----------------------------------------------

Jeśli ``a`` jest LValue (tzn. zmienną lub czymś, do czego można przypisywać),
można stosować poniższe operatory skrótowe:

``a += e`` odpowiada ``a = a + e``. Operatory ``-=``, ``*=``, ``/=``, ``%=``,
``|=``, ``&=``, ``^=``, ``<<=`` i ``>>=`` działają na tej samej zasadzie. ``a++`` i ``a--`` to odpowiedniki ``a += 1`` / ``a -= 1`` ale w samym wyrażeniu ``a`` wciąż zawiera poprzednią wartość. ``--a`` i ``++a`` zwrócą taką samą wartość do ``a``, ale w wyrażeniu ``a`` zawiera już nową wartość.

.. index:: !delete

.. _delete:

delete
------

``delete a`` przypisuje domyślną wartość do ``a``. Jeśli ``a`` jest liczbą całkowitą,
to jest wykonywana jest operacja ``a = 0``. Jeśli ``a`` jest tablicą, to zostanie do
niej przypisana tablica o zerowej długości lub statyczna tablicę o tej samej długości,
ale z elementami zawierającymi ich domyślne wartości. ``delete a[x]`` usuwa element na
pozycji ``x`` i pozostawia inne elementy nienaruszone ani nie zmienia długości tablicy.
To znaczy, że pozostawia lukę w tablicy. Jeśli zamierzasz usuwać elementy z tablicy, to
prawdopodobnie lepszym wyborem są :ref:`mapy <mapping-types>`.

W przypadku struktur resetuje wszystkie pola. Czyli wartość ``a`` po ``delete a``
jest taka, jakby ``a`` została zadeklarowana bez przypisania, z poniższym zastrzeżeniem:

``delete`` nie ma zastosowania do map (ponieważ klucze map są generalnie nieznane).
Tak więc jeśli usuniesz strukturę, to zostaną rekursywnie zresetowane wszystkie pola
oprócz map. Można jednak usuwać pojedyncze klucze: jeśli ``a`` jest mapą, to
``delete a[x]`` usunie wartość przypisaną do klucza ``x``.

Należy zaznaczyć że ``delete a`` naprawdę zachowuje się tak jak przypisanie do ``a``,
tzn. zapisuje nowy obiekt do ``a``. Widać to w sytuacji, gdy ``a`` jest referencją:
zresetuje wyłącznie ``a``, a nie wartość, do której się odnosiła.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

    contract DeleteExample {
        uint data;
        uint[] dataArray;

        function f() public {
            uint x = data;
            delete x; // ustawia x na 0, nie zmienia data
            delete data; // ustawia data na 0, nie zmienia x
            uint[] storage y = dataArray;
            delete dataArray; // zmienia dataArray.length na zero, ale ponieważ uint[] to typ złożony,
            // y także jest zmieniany, ponieważ jest referencją do obiektu magazynowego
            // Ale: "delete y" jest niepoprawne, ponieważ przypisywać do zmiennych lokalnych
            // odnoszących się do obiektów magazynu można tylko z poziomu istniejących obiektów magazynu
            assert(y.length == 0);
        }
    }
