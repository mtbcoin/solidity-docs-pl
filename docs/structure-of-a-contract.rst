.. index:: contract, state variable, function, event, struct, enum, function;modifier

.. _contract_structure:

*******************
Struktura kontraktu
*******************

Kontrakty w Solidity są podobne do klas w językach obiektowych.
Każdy kontrakt może zawierać deklaracje :ref:`structure-state-variables`, :ref:`structure-functions`,
:ref:`structure-function-modifiers`, :ref:`structure-events`, :ref:`structure-errors`, :ref:`structure-struct-types` i :ref:`structure-enum-types`.
Co więcej, kontrakty mogą dziedziczyć inne kontrakty.

Są także specjalne typy kontraktów zwane :ref:`bibliotekami<libraries>` i :ref:`interfejsami<interfaces>`.

Więcej szczegółów znajdziesz w rozdziale :ref:`kontraktach<contracts>`.

.. _structure-state-variables:

Zmienne stanu
=============

Zmienne stanu to zmienne, których wartości są na stałe przechowywane w pamięci kontraktu.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

    contract SimpleStorage {
        uint storedData; // Zmienna stanu
        // ...
    }

Przeczytaj rozdział :ref:`types`, aby dowiedzieć się o typach zmiennych stanu,
:ref:`widoczności i getterach<visibility-and-getters>`.

.. _structure-functions:

Funkcje
=======

Funkcje to wykonywalne jednostki kodu. Są zazwyczaj umieszczane wewnątrz
kontraktu, ale można je także zdefiniować poza kontraktem.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.1 <0.9.0;

    contract SimpleAuction {
        function bid() public payable { // Funkcja
            // ...
        }
    }

    // Helper function defined outside of a contract
    function helper(uint x) pure returns (uint) {
        return x * 2;
    }

:ref:`Wywołania funkcji<function-calls>` mogą być wewnętrzne lub zewnętrzne i mają
różne poziomy :ref:`widoczności<visibility-and-getters>` dla innych kontraktów.
:ref:`Funkcje<functions>` akceptują :ref:`parametry and zmienne wynikowe<function-parameters-return-variables>` do przekazywania parametrów
i wartości między nimi.

.. _structure-function-modifiers:

Modyfikatory funkcji
====================

Modyfikatorów funkcji używa się do zmiany semantyki funkcji w sposób deklaratywny
(zobacz :ref:`modyfikatory<modifiers>` w rozdziale o kontraktach).

Przeładowanie, czyli utworzenie tego samego modyfikatora z innymi+ parametrami,
jest niemożliwe.

Podobnie jak funkcje, modyfikatory można :ref:`nadpisać <modifier-overriding>`.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.22 <0.9.0;

    contract Purchase {
        address public seller;

        modifier onlySeller() { // Modyfikator
            require(
                msg.sender == seller,
                "Only seller can call this."
            );
            _;
        }

        function abort() public view onlySeller { // Użycie modyfikatora
            // ...
        }
    }

.. _structure-events:

Zdarzenia
=========

Zdarzenia to interfejsy udogodniające z funkcjonalnością logowania EVM.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.21 <0.9.0;

    contract SimpleAuction {
        event HighestBidIncreased(address bidder, uint amount); // Zdarzenie

        function bid() public payable {
            // ...
            emit HighestBidIncreased(msg.sender, msg.value); // Wyzwolenie zdarzenia
        }
    }

Zobacz :ref:`events` w sekcji o kontraktach, aby dowiedzieć się, jak deklarować
zdarzenia i jak je wykorzystać wewnątrz zdecentralizowanej aplikacji.

.. _structure-errors:

Błędy
=====

Błędy pozwalają zdefiniować opisowe nazwy i informacje dla niepowodzeń.
Można ich używać w :ref:`instrukcjach revert <revert-statement>`.
W odróżnieniu od opisów tekstowych, błędy są dużo tańsze i pozwalają
zakodować dodatkowe informacje. Możesz użyć NatSpec, aby opisać błąd użytkownikowi.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.4;

    /// Za mało środków, aby wykonać przelew. Zażądano `requested`,
    /// ale tylko `available` jest dostępnych.
    error NotEnoughFunds(uint requested, uint available);

    contract Token {
        mapping(address => uint) balances;
        function transfer(address to, uint amount) public {
            uint balance = balances[msg.sender];
            if (balance < amount)
                revert NotEnoughFunds(amount, balance);
            balances[msg.sender] -= amount;
            balances[to] += amount;
            // ...
        }
    }

Zobacz :ref:`błędy<errors>` w rozdziale o kontraktach, aby dowiedzieć się więcej.

.. _structure-struct-types:

Struktury
=========

Struktury to typy zdefiniowane przez użytkownika, które grupują różne
zmienne (zobacz :ref:`struktury<structs>` w rozdziale o typach).

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

    contract Ballot {
        struct Voter { // Struktura
            uint weight;
            bool voted;
            address delegate;
            uint vote;
        }
    }

.. _structure-enum-types:

Wyliczenia
==========

Wyliczenia to typy zdefiniowane przez użytkownika, które zawierają skońcony zbiór
'stałych wartości' (zobacz :ref:`wyliczenia<enums>` w rozdziale o typach).

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

    contract Purchase {
        enum State { Created, Locked, Inactive } // Wyliczenie
    }
