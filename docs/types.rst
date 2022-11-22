.. index:: type

.. _types:

****
Typy
****

Solidity to język statycznie typowany, co znaczy, że należy określić typ
każdej zmiennej (zarówno zmiennych stanu, jak i lokalnych).
Istnieje kilka podstawowych typów, które można łączyć, aby utworzyć typ złożony.

Dodatkowo typy mogą oddziaływać ze sobą w wyrażeniach zawierających operatory.
Aby poznać różne operatory, przeczytaj :ref:`order`.

Pojęcia "undefined" oraz "null" nie istnieją w Solidity. Nowo zadeklarowane
zmienne zawsze mają :ref:`domyślne wartości<default-value>` w zależności
od ich typu. Aby obsłużyć nieoczekiwane wartości, powinieneś użyć 
:ref:`funkcji revert<assert-and-require>`, aby wycofać całą transakcję
lub zwrócić krotkę z drugą wartością typu ``bool`` oznaczającą sukces.

.. include:: types/value-types.rst

.. include:: types/reference-types.rst

.. include:: types/mapping-types.rst

.. include:: types/operators.rst

.. include:: types/conversion.rst
