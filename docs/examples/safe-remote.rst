.. index:: purchase, remote purchase, escrow

************************
Bezpieczne zakupy online
************************

Kupowanie towarów online obecnie angażuje kilka stron, które muszą sobie wzajemnie ufać.
Najprostsza konfiguracja obejmuje sprzedającego i kupującego. Kupujący chciałby otrzymać
towar od sprzedającego, zaś sprzedający oczekuje w zamian pieniędzy (lub równowartość).
Największy problem stanowi tutaj wysyłka: nie istnieje sposób, aby dowieść, że kupujący
otrzymał towar.

Problem da się rozwiązać na wiele sposobów, ale wszystkie zawodzą w taki czy inny sposób.
W poniższym przykładzie obie strony muszą wysłać podwójną wartość towaru do kontraktu jako
zabezpieczenie. Kiedy tylko to uczynią, pieniądze zostaną zamrożone w kontrakcie do chwili,
aż kupujący potwierdzi, że otrzymał towar. Potem kupującemu zwraca się kwotę (połowę całego
depozytu), a sprzedający otrzymuje trzykrotność kwoty (depozyt i wartość towaru). Pomysł
opiera się na założeniu, że obu stronom zależy na rozwiązaniu sporu albo ich pieniądze
zostaną zamrożone na amen.

Ten kontrakt oczywiście nie rozwiązuje problemu, ale pokazuje, w jaki sposób
zaimplementować maszynę stanu.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.4;
    contract Purchase {
        uint public value;
        address payable public seller;
        address payable public buyer;

        enum State { Created, Locked, Release, Inactive }
        // Zmienna stanu zawiera domyślnie pierwszy element wyliczenia, `State.created`
        State public state;

        modifier condition(bool condition_) {
            require(condition_);
            _;
        }

        /// Tylko kupujący może wywołać tę funkcję.
        error OnlyBuyer();
        /// Tylko sprzedający może wywołać tę funkcję.
        error OnlySeller();
        /// Funkcja nie może zostać wywołana w aktualnym stanie.
        error InvalidState();
        /// Podana kwota musi być parzysta.
        error ValueNotEven();

        modifier onlyBuyer() {
            if (msg.sender != buyer)
                revert OnlyBuyer();
            _;
        }

        modifier onlySeller() {
            if (msg.sender != seller)
                revert OnlySeller();
            _;
        }

        modifier inState(State state_) {
            if (state != state_)
                revert InvalidState();
            _;
        }

        event Aborted();
        event PurchaseConfirmed();
        event ItemReceived();
        event SellerRefunded();

        // Upewnia się, że `msg.value` jest liczbą parzystą.
        // Wynik dzielenia zostanie ucięty do liczby całkowitej,
		// jeśli to będzie liczba nieparzysta. Sprawdzamy za
        // pomocą mnożenia, czy to nie jest liczba nieparzysta.
        constructor() payable {
            seller = payable(msg.sender);
            value = msg.value / 2;
            if ((2 * value) != msg.value)
                revert ValueNotEven();
        }

        /// Przerywa zakup i zwraca ethery.
        /// Może wywołać tylko sprzedający,
		/// zanim kontrakt zostanie zablokowany.
        function abort()
            external
            onlySeller
            inState(State.Created)
        {
            emit Aborted();
            state = State.Inactive;
            // Wywołujemy transfer bezpośrednio.
            // Jest to bezpieczne, ponieważ to jest ostatnie
			// wywołanie w tej funkcji i zmieniliśmy już stan.
            seller.transfer(address(this).balance);
        }

        /// Kupujący potwierdza zakup.
        /// Transakcja musi zawierać `2 * value` etherów.
        /// Ethery zostaną zamrożone do momentu wywołania confirmReceived.
        function confirmPurchase()
            external
            inState(State.Created)
            condition(msg.value == (2 * value))
            payable
        {
            emit PurchaseConfirmed();
            buyer = payable(msg.sender);
            state = State.Locked;
        }

        /// Kupujący potwierdza, że otrzymał towar.
        /// Zamrożone środki zostaną uwolnione.
        function confirmReceived()
            external
            onlyBuyer
            inState(State.Locked)
        {
            emit ItemReceived();
            // Istotne, aby najpierw zmienić stan, ponieważ inaczej
			// kontrakty wywołane za pomocą `send` poniżej mogłyby
            // ponownie wykonać ten fragment kodu.
            state = State.Release;

            buyer.transfer(value);
        }

        /// Ta funkcja zwraca środki sprzedającemu, tzn.
        /// uwalnia zamrożone fundusze sprzedającego.
        function refundSeller()
            external
            onlySeller
            inState(State.Release)
        {
            emit SellerRefunded();
            // Istotne, aby najpierw zmienić stan, ponieważ inaczej
			// kontrakty wywołane za pomocą `send` poniżej mogłyby
            // ponownie wykonać ten fragment kodu.
            state = State.Inactive;

            seller.transfer(3 * value);
        }
    }
