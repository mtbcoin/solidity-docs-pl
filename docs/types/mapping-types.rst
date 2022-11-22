.. index:: !mapping
.. _mapping-types:

Mapy
====

Mapy tworzy się za pomocą składni ``mapping(KeyType => ValueType)``, a zmienne typu mapa
deklaruje się przy użyciu ``mapping(KeyType => ValueType) VariableName``.
``KeyType`` może być dowolnym wbuowanym typem: ``bytes``, ``string``, kontraktem lub wyliczeniem enum. Typy zdefiniowane przez użytkownika lub typy złożone, takie jak mapy, struktury lub tablice, są niedozwolone.
``ValueType`` może być dowolnego typu, wliczając mapy, tablice i struktury.

Możesz myśleć o mapach jak o `tablicach mieszających <https://pl.wikipedia.org/wiki/Tablica_mieszaj%C4%85ca>`_,
które są wirtualnie inicjowane tak, że każdy możliwy klucz istnieje
i jest do niego przypisana :ref:`wartość domyślna <default-value>`,
w reprezentacji bajtowej złożona z samych zer.
Dane klucza nie są przechowywane w mapie, a jedynie jego skrót 
``keccak256`` jest używany do wyszukania wartości.

Dlatego mapy nie mają określonej długości i nie istnieje pojęcie dodawania klucza
ani wartości. Z tego powodu nie można ich wyczyścić bez podania dodatkowych informacji
dotyczących przypisanych kluczy (zobacz :ref:`czyszczenie map<clearing-mappings>`).

Mapy mogą zawierać jedynie położenie danych w ``magazynie<storage>``i tym samym
można ich użyć w zmiennych stanu jako typów referencji do magazynu, w funkcjach
oraz jako parametrów funkcji bibliotecznych.
Nie można ich używać jako argumentów ani parametrów zwracanych
w funkcjach kontraktów, które są widoczne publicznie.
Te ograniczenia dotyczą także tablic i struktur zawierających mapy.

Możesz oznaczyć zmienne stanu typu mapa jako ``public``. Wtedy Solidity automatycznie utworzy
:ref:`getter <visibility-and-getters>`. Parametrem gettera będzie ``KeyType``.
Jeśli ``ValueType`` jest typem wartości lub strukturą, getter zwraca ``ValueType``.
Jeśli ``ValueType`` jest tablicą lub mapą, getter ma jeden parametr dla każdego ``KeyType``, rekurencyjnie.

W poniższym przykładzie kontrakt ``MappingExample`` definiuje publiczną mapę ``balances``
z typem kluczy ``address`` i typem wartości ``uint``, przypisującą adresy Ethereum do
liczby całkowitej bez znaku. Ponieważ ``uint`` jest typem wartości, getter zwraca wartość
odpowiadającą typowi, jaki możes zobaczyć w kontrkacie ``MappingUser`` który zwraca kwotę
pod określonym adresem.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

    contract MappingExample {
        mapping(address => uint) public balances;

        function update(uint newBalance) public {
            balances[msg.sender] = newBalance;
        }
    }

    contract MappingUser {
        function f() public returns (uint) {
            MappingExample m = new MappingExample();
            m.update(100);
            return m.balances(address(this));
        }
    }

Poniższy przykład jest uproszczoną wersją
`tokenu ERC20 <https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol>`_.
``_allowances`` to przykład, jak umieścić mapę wewnątrz innej mapy.
Zmienna ``_allowances`` przechowuje kwotę, jaką inne osoby mogą wypłacić z twojego konta.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.22 <0.9.0;

    contract MappingExample {

        mapping (address => uint256) private _balances;
        mapping (address => mapping (address => uint256)) private _allowances;

        event Transfer(address indexed from, address indexed to, uint256 value);
        event Approval(address indexed owner, address indexed spender, uint256 value);

        function allowance(address owner, address spender) public view returns (uint256) {
            return _allowances[owner][spender];
        }

        function transferFrom(address sender, address recipient, uint256 amount) public returns (bool) {
            require(_allowances[sender][msg.sender] >= amount, "ERC20: Allowance not high enough.");
            _allowances[sender][msg.sender] -= amount;
            _transfer(sender, recipient, amount);
            return true;
        }

        function approve(address spender, uint256 amount) public returns (bool) {
            require(spender != address(0), "ERC20: approve to the zero address");

            _allowances[msg.sender][spender] = amount;
            emit Approval(msg.sender, spender, amount);
            return true;
        }

        function _transfer(address sender, address recipient, uint256 amount) internal {
            require(sender != address(0), "ERC20: transfer from the zero address");
            require(recipient != address(0), "ERC20: transfer to the zero address");
            require(_balances[sender] >= amount, "ERC20: Not enough funds.");

            _balances[sender] -= amount;
            _balances[recipient] += amount;
            emit Transfer(sender, recipient, amount);
        }
    }


.. index:: !iterable mappings
.. _iterable-mappings:

Iterowalne mapy
---------------

Nie możesz iterować po mapach, tzn. nie możesz wyliczyć ich kluczy.
Natomiast można na ich bazie stworzyć osobną strukturę danych i po
niej iterować. Poniższy kod tworzy bibliotekę ``IterableMapping``.
Kontrakt ``User`` dodaje elementy do struktury, zaś funkcja ``sum`` 
iteruje po niej i sumuje wszystkie kwoty.

.. code-block:: solidity
    :force:

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.8;

    struct IndexValue { uint keyIndex; uint value; }
    struct KeyFlag { uint key; bool deleted; }

    struct itmap {
        mapping(uint => IndexValue) data;
        KeyFlag[] keys;
        uint size;
    }

    type Iterator is uint;

    library IterableMapping {
        function insert(itmap storage self, uint key, uint value) internal returns (bool replaced) {
            uint keyIndex = self.data[key].keyIndex;
            self.data[key].value = value;
            if (keyIndex > 0)
                return true;
            else {
                keyIndex = self.keys.length;
                self.keys.push();
                self.data[key].keyIndex = keyIndex + 1;
                self.keys[keyIndex].key = key;
                self.size++;
                return false;
            }
        }

        function remove(itmap storage self, uint key) internal returns (bool success) {
            uint keyIndex = self.data[key].keyIndex;
            if (keyIndex == 0)
                return false;
            delete self.data[key];
            self.keys[keyIndex - 1].deleted = true;
            self.size --;
        }

        function contains(itmap storage self, uint key) internal view returns (bool) {
            return self.data[key].keyIndex > 0;
        }

        function iterateStart(itmap storage self) internal view returns (Iterator) {
            return iteratorSkipDeleted(self, 0);
        }

        function iterateValid(itmap storage self, Iterator iterator) internal view returns (bool) {
            return Iterator.unwrap(iterator) < self.keys.length;
        }

        function iterateNext(itmap storage self, Iterator iterator) internal view returns (Iterator) {
            return iteratorSkipDeleted(self, Iterator.unwrap(iterator) + 1);
        }

        function iterateGet(itmap storage self, Iterator iterator) internal view returns (uint key, uint value) {
            uint keyIndex = Iterator.unwrap(iterator);
            key = self.keys[keyIndex].key;
            value = self.data[key].value;
        }

        function iteratorSkipDeleted(itmap storage self, uint keyIndex) private view returns (Iterator) {
            while (keyIndex < self.keys.length && self.keys[keyIndex].deleted)
                keyIndex++;
            return Iterator.wrap(keyIndex);
        }
    }

    // Jak tego użyć
    contract User {
        // Struktura przechowujaca nasze dane.
        itmap data;
        // Stosuje funkcje biblioteczne do tego typu danych.
        using IterableMapping for itmap;

        // Wstaw coś.
        function insert(uint k, uint v) public returns (uint size) {
            // Wywołuje IterableMapping.insert(data, k, v)
            data.insert(k, v);
            // Wciąż mamy dostęp do pól struktury, ale nie
            // powinniśmy odwoływać się do nich bez potrzeby.
            return data.size;
        }

        // Oblicza sumę wszystkich przechowywanych wartości.
        function sum() public view returns (uint s) {
            for (
                Iterator i = data.iterateStart();
                data.iterateValid(i);
                i = data.iterateNext(i)
            ) {
                (, uint value) = data.iterateGet(i);
                s += value;
            }
        }
    }
