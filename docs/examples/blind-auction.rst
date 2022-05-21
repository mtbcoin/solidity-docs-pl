.. index:: auction;blind, auction;open, blind auction, open auction

***************
Aukcje w ciemno
***************

W tym rozdziale pokażemy, jak łatwo stworzyć kontrakt do przeprowadzania
aukcji w ciemno w sieci Ethereum. Zaczniemy od aukcji otwartych, gdzie
każdy może zobaczyć złożone oferty, a potem rozszerzymy ten kontrakt
o aukcje w ciemno, gdzie nie da się zobaczyć aktualnej oferty do momentu
zakończenia licytacji.

.. _simple_auction:

Prosta aukcja otwarta
=====================

Ogólna idea kontraktu polega na tym, że każdy może złożyć swoją ofertę w trakcie
trwania licytacji. Oferty już zawierają pieniądze / ethery na powiązanie oferentów
z ich ofertami. Jeśli najwyższa oferta zostanie przebita, poprzedni oferent otrzyma
zwrot pieniędzy. Po zakończeniu licytacji beneficjent musi ręcznie wywołać kontrakt,
aby otrzymać swoje pieniądze - kontrakty nie mogą same siebie wywoływać.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.4;
    contract SimpleAuction {
        // Parametry aukcji. Czas to albo
        // uniksowy znacznik czasu (liczba sekund od 1970-01-01),
        // albo okres czas w sekundach.
        address payable public beneficiary;
        uint public auctionEndTime;

        // Bieżący stan aukcji.
        address public highestBidder;
        uint public highestBid;

        // Dozwolone zwroty poprzednich ofert.
        mapping(address => uint) pendingReturns;

        // Ustaw na true po zakończeniu licytacji, aby nie pozwolić na żadne zmiany.
        // Domyślnie inicjowany do `false`.
        bool ended;

        // Zdarzenia wywoływane podczas zmian.
        event HighestBidIncreased(address bidder, uint amount);
        event AuctionEnded(address winner, uint amount);

        // Błędy opisujące niepowodzenia.

        // Komentarze z trzeba ukośnikami to tzw. komentarze natspec.
        // Pokazujemy je, kiedy pytamy użytkownika o potwierdzenie
        // transakcji lub gdy wyświetlamy błąd.

        /// Aukcja już się zakończyła.
        error AuctionAlreadyEnded();
        /// Jest już wyższa lub taka sama oferta.
        error BidNotHighEnough(uint highestBid);
        /// Aukcja jeszcze się nie rozpoczęła.
        error AuctionNotYetEnded();
        /// Funkcja auctionEnd została już wywołana.
        error AuctionEndAlreadyCalled();

        /// Stwórz prostą aukcję z czasem trwania `biddingTime` sekund
        /// w imieniu adresu beneficjenta `beneficiaryAddress`.
        constructor(
            uint biddingTime,
            address payable beneficiaryAddress
        ) {
            beneficiary = beneficiaryAddress;
            auctionEndTime = block.timestamp + biddingTime;
        }

        /// Licytuj w aukcji kwotę wysłaną w transakcji.
        /// Kwota zostanie zwrócona, jeśli nie wygrasz licytacji.
        function bid() external payable {
            // Nie potrzeba żadnych argumentów, ponieważ
            // potrzebne informacje są zawarte w transakcji.
            // Słowo kluczowe payable jest wymagane, aby
            // funkcja mogła przyjmować ethery.

            // Wycofaj zmiany, jeśli licytacja się zakończyła.
            if (block.timestamp > auctionEndTime)
                revert AuctionAlreadyEnded();

            // Jeśli oferta nie została przebita,
            // zwróć pieniądze (polecenie revert
            // wycofa wszystkie zmiany dokonane
            // wewnątrz funkcji łącznie
            // z otrzymaniem pieniędzy.
            if (msg.value <= highestBid)
                revert BidNotHighEnough(highestBid);

            if (highestBid != 0) {
                // Gdybyśmy zwrócili pieniądze, po prostu wywołując
                // highestBidder.send(highestBid) to stworzylibyśmy
                // lukę w bezpieczeństwie, ponieważ mógłby zostać
                // wywołany niezaufany kontrakt. Zawsze lepiej, aby
                // odbiorcy wycofywali pieniądze samodzielnie.
                pendingReturns[highestBidder] += highestBid;
            }
            highestBidder = msg.sender;
            highestBid = msg.value;
            emit HighestBidIncreased(msg.sender, msg.value);
        }

        /// Wycofaj ofertę, która została przebita.
        function withdraw() external returns (bool) {
            uint amount = pendingReturns[msg.sender];
            if (amount > 0) {
                // Jest ważne, aby ustawić zero, ponieważ odbiorca może
                // wywołać tę funkcję ponownie zanim `send` zakończy się.
                pendingReturns[msg.sender] = 0;

                // msg.sender nie jest typu `address payable` dlatego trzeba
                // go jawnie skonwertować za pomocą `payable(msg.sender)`
                // aby wywołać metodę `send()`.
                if (!payable(msg.sender).send(amount)) {
                    // Nie ma potrzeby wywoływać tutaj throw, po prostu zresetuj należną kwotę
                    pendingReturns[msg.sender] = amount;
                    return false;
                }
            }
            return true;
        }

        /// Zakończ licytację i wyślij najwyższą ofertę do beneficjenta.
        function auctionEnd() external {
            // Zaleca się podzielić funkcje komunikujące się z innymi kontraktami
            // (np. wywołujące ich funkcje lub wysyłające ethery) na 3 fazy:
            // 1. sprawdzenie warunków
            // 2. wykonanie akcji (potencjalnie zmiana warunków)
            // 3. interakcja z innymi kontraktami
            // Jeśli powyższe fazy zostaną wymieszane, inny kontrakt mógłby
            // wywołać zwrotnie bieżący kontrakt i zmodyfikować jego stan
            // lub doprowadzić do wielokrotnego wykonania tych samych
            // czynności (np. wypłacenia etherów). Jeśli funkcje wywoływane
            // wewnętrznie zawierają interakcję z zewnętrznymi kontraktami,
            // należy też traktować jako interakcję z zewnętrznymi kontraktami.

            // 1. Warunki
            if (block.timestamp < auctionEndTime)
                revert AuctionNotYetEnded();
            if (ended)
                revert AuctionEndAlreadyCalled();

            // 2. Efekty
            ended = true;
            emit AuctionEnded(highestBidder, highestBid);

            // 3. Interakcja
            beneficiary.transfer(highestBid);
        }
    }

Aukcja w ciemno
==============

Teraz rozszerzymy kod kontraktu o aukcje w ciemno. Zaletą takich aukcji jest to,
że nie ma presji czasu pod koniec licytacji. Tworzenie aukcji w ciemno na przejrzystej
platformie wydaje się nielogiczne, ale z pomocą przychodzi kryptografia.

Podczas **trwania licytacji** oferent wysyła swoją ofertę w postaci hashu.
Ponieważ znalezienie dwóch (odpowiednio długich) wartości, dla których funkcje
skrótu są identyczne, uznaje się za niemożliwe, taki sposób składania ofert można
uznać za bezpieczny. Po zakończeniu licytacji oferenci muszą ujawnić swoje offerty:
wysyłają swoje kwoty niezakodowane, a kontrakt weryfikuje, czy przesłane w trakcie
trwania licytacji funkcje skrótu zgadzają się z faktycznymi.

Kolejnym wyzwaniem jest utworzenie aukcji jednocześnie **wiążącej i w ciemno**.
Jedynym sposobem, aby zapobiec sytuacji, że oferent nie wyśle pieniędzy po wygraniu
aukcji, jest zobowiązanie do wysłania ich wraz z ofertą. Ale ponieważ w Ethereum nie
da się ukryć przesłanej kwoty, każdy może ją zobaczyć.

Poniższy kontrakt rozwiązuje ten problem, akceptując dowolną kwotę, która
jest większa od najwyższej oferty. Ponieważ da się to sprawdzić dopiero po
ujawnieniu ofert, niektóre oferty mogą zostać **odrzucone** i to jest celowe.
Istnieje jawna flaga do składania fałszywych ofert z dużą kwotą. Oferenci mogą
utrudnić rywalizację, składając wiele drogich lub tanich fałszywych ofert.


.. code-block:: solidity
    :force:

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.4;
    contract BlindAuction {
        struct Bid {
            bytes32 blindedBid;
            uint deposit;
        }

        address payable public beneficiary;
        uint public biddingEnd;
        uint public revealEnd;
        bool public ended;

        mapping(address => Bid[]) public bids;

        address public highestBidder;
        uint public highestBid;

        // Dozwolone wypłaty poprzednich ofert
        mapping(address => uint) pendingReturns;

        event AuctionEnded(address winner, uint highestBid);

        // Błędy, które opisują niepowodzenia.

        /// Funkcja wywołania za wcześnie.
        /// Spróbuj ponownie o `time`.
        error TooEarly(uint time);
        /// Funkcja wywołana za późno.
        /// Nie można jej wywołać po `time`.
        error TooLate(uint time);
        /// Funkcja auctionEnd została już wywołana.
        error AuctionEndAlreadyCalled();

        // Modyfikatory są wygodnym sposobem walidacji danych wejściowych
        // do funkcji. `onlyBefore` zostanie użyty w `bid` poniżej:
        modifier onlyBefore(uint time) {
            if (block.timestamp >= time) revert TooLate(time);
            _;
        }
        modifier onlyAfter(uint time) {
            if (block.timestamp <= time) revert TooEarly(time);
            _;
        }

        constructor(
            uint biddingTime,
            uint revealTime,
            address payable beneficiaryAddress
        ) {
            beneficiary = beneficiaryAddress;
            biddingEnd = block.timestamp + biddingTime;
            revealEnd = biddingEnd + revealTime;
        }

        /// Składa ofertę w ciemno, gdzie `blindedBid` =
        /// keccak256(abi.encodePacked(value, fake, secret)).
        /// Wysłane ethery wrócą do oferenta tylko wtedy, gdy uda
        /// się odsłonić poprawnie ofertę po zakończeniu licytacji.
        /// Oferta jest prawidłowa, jeśli kwota przesłana z ofertą
		/// jest równa lub większa "value" i "fake" nie jest true.
        /// Ustawienie "fake" na true i wysłanie niedokładnej kwoty
        /// są sposobami na ukrycie prawdziwej oferty, ale przesłana
        /// kwota zalicza się do depozytu. Ten sam adres może złożyć
        /// wiele ofert.
        function bid(bytes32 blindedBid)
            external
            payable
            onlyBefore(biddingEnd)
        {
            bids[msg.sender].push(Bid({
                blindedBid: blindedBid,
                deposit: msg.value
            }));
        }

        /// Odsłania ofertę w ciemno. Otrzymasz zwrot wszystkich poprawnie
		/// zakodowanych fałszywych ofert poza najwyższą zwycięską ofertą.
        function reveal(
            uint[] calldata values,
            bool[] calldata fakes,
            bytes32[] calldata secrets
        )
            external
            onlyAfter(biddingEnd)
            onlyBefore(revealEnd)
        {
            uint length = bids[msg.sender].length;
            require(values.length == length);
            require(fakes.length == length);
            require(secrets.length == length);

            uint refund;
            for (uint i = 0; i < length; i++) {
                Bid storage bidToCheck = bids[msg.sender][i];
                (uint value, bool fake, bytes32 secret) =
                        (values[i], fakes[i], secrets[i]);
                if (bidToCheck.blindedBid != keccak256(abi.encodePacked(value, fake, secret))) {
                    // Bid was not actually revealed.
                    // Do not refund deposit.
                    continue;
                }
                refund += bidToCheck.deposit;
                if (!fake && bidToCheck.deposit >= value) {
                    if (placeBid(msg.sender, value))
                        refund -= value;
                }
                // Make it impossible for the sender to re-claim
                // the same deposit.
                bidToCheck.blindedBid = bytes32(0);
            }
            payable(msg.sender).transfer(refund);
        }

        /// Zwraca ofertę, która została przebita.
        function withdraw() external {
            uint amount = pendingReturns[msg.sender];
            if (amount > 0) {
                // Jest ważne, aby wyzerować to pole, ponieważ odbiorca może
                // wywołać tę funkcję ponownie, zanim `transfer` zakończy się
				   (przeczytaj uwagę powyżej o warunki -> efekty -> interakcja).
                pendingReturns[msg.sender] = 0;

                payable(msg.sender).transfer(amount);
            }
        }

        /// Zakańcza aukcję o wysyła kwotę z najwyższej oferty do beneficjenta.
        function auctionEnd()
            external
            onlyAfter(revealEnd)
        {
            if (ended) revert AuctionEndAlreadyCalled();
            emit AuctionEnded(highestBidder, highestBid);
            ended = true;
            beneficiary.transfer(highestBid);
        }

        // To jest "wewnętrzna" funkcja, co znaczy, że może być
		// wywołana tylko przez kontrakt (lub kontrakt pochodny).
        function placeBid(address bidder, uint value) internal
                returns (bool success)
        {
            if (value <= highestBid) {
                return false;
            }
            if (highestBidder != address(0)) {
                // Zwróć poprzednią najwyższą kwotę oferentowi.
                pendingReturns[highestBidder] += highestBid;
            }
            highestBid = value;
            highestBidder = bidder;
            return true;
        }
    }
