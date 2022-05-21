********************
Kanał mikropłatności
********************

W tym rozdziale dowiesz się, jak stworzyć przykładową implementację
kanału płatności. Używa ona podpisów kryptograficznych, aby uczynić
powtarzalne przelewy etherów między tymi samymi stronami bezpiecznymi,
natychmiastowymi i bez prowizji. Musisz zrozumieć, jak podpisywać
i weryfikować sygnatury oraz konfigurować kanały płatności.

Tworzenie i weryfikacja podpisów
================================

Wyobraź sobie, że Alicja chce wysłać trochę etherów Robertowi, tzn.
Alicja jest nadawcą, a Robert odbiorcą.

Alicja potrzebuje tylko wysyłać podpisane kryptograficznie wiadomości
poza łańcuchem bloków (np. pocztą e-mail) do Roberta. Jest to podobne
do wystawiania czeków.

Alicja i Robert używają podpisów do autoryzacji transakcji. Taką możliwość
dają inteligentne kontrakty w sieci Ethereum. Alicja stworzy prosty inteligentny
kontrakt pozwalający jej wysyłać ethery, ale zamiast wywoływać funkcję samodzielnie
w celu rozpoczęcia płatności, zleci to Robertowi i to on zapłaci prowizję od transakcji.

Kontrakt będzie działał w następujący sposób:

    1. Alicja wdraża kontrakt ``ReceiverPays``, dołączając wystarczającą ilość etherów,
	   aby pokryć płatności, jakie będą dokonywane.
	2. Alicja autoryzuje płatność, podpisując wiadomość jej kluczem prywatnym.
	3. Alicja wysyła podpisaną kryptograficznie wiadomość do Roberta. Nie ma
	   potrzeby trzymać wiadomości w tajemnicy (wyjaśnione później), a sposób
	   jej wysyłki nie ma znaczenia.
	4. Robert ubiega się o wypłatę, pokazując kontraktowi podpisaną wiadomość.
	   Kontrakt weryfikuje autentyczność wiadomości i uwalnia środki.

Tworzenie podpisu
-----------------

Alicja nie musi mieć dostępu do sieci Ethereum, aby podpisać
transakcję. To się dzieje całkowicie offline. W tym poradniku
podpiszemy wiadomości w przeglądarce za pomocą biblioteki
`web3.js <https://github.com/ethereum/web3.js>`_
i portfela `MetaMask <https://metamask.io>`_,
używając sposobu opisanego w 
`EIP-712 <https://github.com/ethereum/EIPs/pull/712>`_,
ponieważ oferuje on wiele korzyści związanych z bezpieczeństwem.

.. code-block:: javascript

    /// Wygenerowanie funkcji skrótu na początku ułatwia sprawę
    var hash = web3.utils.sha3("wiadomość do podpisu");
    web3.eth.personal.sign(hash, web3.eth.defaultAccount, function () { console.log("Podpisano"); });

.. note::
  Funkcja ``web3.eth.personal.sign`` dopisuje długość wiadomości
  na początku podpisanych informacji. Ponieważ najpierw generujemy
  funkję skrótu, wiadomość zawsze będzie zawierać 32 bajty, a więc
  prefiks będzie zawsze taki sam.

Co podpisać
-----------

Dla kontraktu, który realizuje płatności, podpisana wiadomość musi zawierać:

    1. Adres odbiorcy.
	2. Kwotę do przesłania.
	3. Ochronę przed atakami powtórzeniowymi.
	
Atak powtórzeniowy polega na tym, że podpisaną wiadomość wykorzystuje się
ponownie w celu autoryzacji innej czynności. Aby zapobiec tego typu atakom,
używamy tego samego sposobu, co w transakcjach Ethereum, tzw. nonce, która
jest liczbą transakcji wysłanych przez dane konto. Inteligentny kontakt
sprawdza, czy tego samego nonce użyto wielokrotnie.

Inny typ ataku powtórzeniowego może wystąpić, kiedy właściciel wdroży
inteligentny kontrakt ``ReceiverPays``, dokona kilka płatności,
a następnie zniszczy kontrakt. Później decyduje się wdrożyć go
ponownie, ale nowy kontrakt nie zna poprzednio użytych liczb nonce,
więc atakujący może ich użyć ponownie.

Alicja może ochronić się prze tym atakiem, dołączając adres kontraktu
w wiadomości i tylko wiadomości zawierające adres kontraktu będą akceptowane.
Możesz znaleźć przykład w dwóch pierwszych liniach funkcji ``claimPayment()``
pełnego kontraktu pod koniec rozdziału.

Pakowanie argumentów
--------------------

Kiedy wiemy już, jakie informacje dołączyć do podpisanej wiadomości,
jesteśmy gotowi, aby połączyć je ze sobą, policzyć funkcję skrótu
i ją podpisać. Aby uprościć sprawę, dokonamy konkatenacji danych.
Bibliotka `ethereumjs-abi <https://github.com/ethereumjs/ethereumjs-abi>`_
dostarcza funkcję nazwaną ``soliditySHA3`` która imituje zachowanie funkcji
``keccak256``z Solidity przekazaną jako argumenty, zakodowaną przez ``abi.encodePacked``.
Poniżej znajduje się przykład funkcji w JavaScript, która tworzy podpis dla przykładu ``ReceiverPays``:

.. code-block:: javascript

    // recipient to adres, któremu wysyłamy płatności.
    // amount, w jednostce wei, określa, ile etherów chcemy wysłać.
    // nonce to dowolna unikalna liczba, aby zapobiec atakom powtórzeniowym.
    // contractAddress podajemy, aby zapobiec atakom powtórzeniowym między kontraktami.
    function signPayment(recipient, amount, nonce, contractAddress, callback) {
        var hash = "0x" + abi.soliditySHA3(
            ["address", "uint256", "uint256", "address"],
            [recipient, amount, nonce, contractAddress]
        ).toString("hex");

        web3.eth.personal.sign(hash, web3.eth.defaultAccount, callback);
    }

Odzyskiwanie sygnatariusza w Solidity
-------------------------------------

Zasaniczo podpisy ECDSA składają się z dwóch parametrów,
``r`` oraz ``s``. Podpisy w Ethereum zawierają trzeci
parametr zwany ``v``, który można użyć do sprawdzenia,
którego konta kluczem publicznym podpisano wiadomość,
a także kto jest nadawcą transakcji. Solidity zawiera
wbudowaną funkcję :ref:`ecrecover <mathematical-and-cryptographic-functions>`
która przyjmuuje wiadomość wraz z parametrami ``r``, ``s`` oraz ``v``
i zwraca adres, któego użyto do podpisania wiadomości.

Wydobycie parametrów podpisu
----------------------------

Podpisy generowane przez web3.js są konkatenacją ``r``,``s`` i ``v``,
więc pierwszy krok to rozdzielenie tych parametrów. Możesz to uczynić
po stronie klienta, ale wykonanie tego w kontrakcie oznacza konieczność
wysłania tylko jednego parametru podpisu zamiast trzech.
Rozdzielanie tablicy bajtów na składniki to jeden wielki
bałagan, dlatego w funkcji ``splitSignature`` użyjemy :doc:`wstawki assembly <assembly>`,
aby wykonała to za nas (trzecia funkcja w pełnym kontrakcie pod koniec rozdziału).

Liczenie funkcji skrótu wiadomości
----------------------------------

Inteligentny kontrakt musi wiedzieć, jakie dokładnie parametry podpisano,
ponieważ potrzebuje odtworzyć wiadomość z parametrów w celu weryfikacji podpisu.
Funkcje ``prefixed`` i ``recoverSigner`` czynią to w funkcji ``claimPayment``.

Pełny kontrakt
--------------

.. code-block:: solidity
    :force:

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;
    contract ReceiverPays {
        address owner = msg.sender;

        mapping(uint256 => bool) usedNonces;

        constructor() payable {}

        function claimPayment(uint256 amount, uint256 nonce, bytes memory signature) external {
            require(!usedNonces[nonce]);
            usedNonces[nonce] = true;

            // odtwarza wiadomość podpisaną przez klienta
            bytes32 message = prefixed(keccak256(abi.encodePacked(msg.sender, amount, nonce, this)));

            require(recoverSigner(message, signature) == owner);

            payable(msg.sender).transfer(amount);
        }

        /// niszczy kontrakt i zwraca pozostałe środki.
        function shutdown() external {
            require(msg.sender == owner);
            selfdestruct(payable(msg.sender));
        }

        /// metody dotyczące podpisu
        function splitSignature(bytes memory sig)
            internal
            pure
            returns (uint8 v, bytes32 r, bytes32 s)
        {
            require(sig.length == 65);

            assembly {
                // first 32 bytes, after the length prefix.
                r := mload(add(sig, 32))
                // second 32 bytes.
                s := mload(add(sig, 64))
                // final byte (first byte of the next 32 bytes).
                v := byte(0, mload(add(sig, 96)))
            }

            return (v, r, s);
        }

        function recoverSigner(bytes32 message, bytes memory sig)
            internal
            pure
            returns (address)
        {
            (uint8 v, bytes32 r, bytes32 s) = splitSignature(sig);

            return ecrecover(message, v, r, s);
        }

        /// tworzy hash z prefiksem, naśladując zachowanie eth_sign.
        function prefixed(bytes32 hash) internal pure returns (bytes32) {
            return keccak256(abi.encodePacked("\x19Ethereum Signed Message:\n32", hash));
        }
    }


Napisanie prostego kanału płatności
===================================

Alicja teraz buduje prostą, ale kompletną implementację kanału płatności.
Używa on podpisów kryptograficznych, aby uczynić powtarzalne przelewy
etherów bezpiecznymi, natychmiastowymi i bez opłat transakcyjnych.

Co to jest kanał płatności?
---------------------------

Kanały płatności umożliwiają uczestnikom zlecać powtarzalne przelewy
etherów bez transakcji. To znaczy, że możesz omijać opóźnienia i opłaty
transakcyjne. Będziemy odkrywać prosty jednokierunkowy kanał płatności
między dwoma stronami (Alicją i Robertem). Obejmuje trzy etapy:

	1. Alicja zasila inteligentny kontrakt etherami. To "otwiera" kanał płatności.
	2. Alicja podpisuje wiadomości, które określają, ile etherów chce wysłać odbiorcy. Ten krok trzeba powtórzyć przed każdą płatnością.
	3. Robert "zamyka" kanał płatności, wycofując swoją część etherow i odsyłając resztę nadawcy.

.. note::
  Tylko etapy 1 i 3 wymagają transakcji Ethereum. Krok 2 oznacza, że nadawca
  przesyła podpisaną kryptograficznie wiadomość do odbiorcy poza łańcuchem bloków
  (np. pocztą e-mail). To znaczy, że tylko dwie transakcje są potrzebne, aby
  obsłużyć dowolną liczbę przelewów.

Robert otrzymuje gwarancję, że otrzyma swoje środki, ponieważ inteligentny
kontrakt zabezpiecza ethery i honoruje prawidłowo podpisane wiadomości.
Wymusza także limit czasu, więc Alicja może odzyskać fundusze, jeśli
odbiorca odmówi zamknięcia kanału. To zależy od uczestników, jak długo
trzymać kanał otwarty. Dla krótkotrwałych transakcji, takich jak płatność
w kafejce internetowej za każdą minutę dostępu do Internetu, można otworzyć
kanał płatności na ograniczony okres czasu. Z drugiej strony, w przypadku
powtarzalnych płatności, takich jak wypłata pracownikowi stawki godzinowej,
może być otwarty przez kilka miesięcy lub lat.

Otwieranie kanału płatności
---------------------------

Aby otworzyć kanał płatności, Alicja wdraża inteligentny kontrakt,
dołączając ethery do zabezpieczenia i określając odbiorcę oraz limit
czasu, przez który kanał ma istnieć. Odpowiada za to funkcja
``SimplePaymentChannel`` w kontrakcie pod koniec rozdziału.

Dokonywanie płatności
-------------------

Alicja dokonuje płatności, wysyłając podpisane wiadomości do Roberta.
Ten krok odbywa się całkowicie poza siecią Ethereum.
Nadawca podpisuje wiadomości kryptograficznie i wysyła je bezpośrednio do odbiorcy.

Każda wiadomość zawiera następujące informacje:

    * Adres inteligentnego kontraktu, aby zapobiec atakom powtórzeniowym z innych kontraktów.
    * Całkowita ilość etherów, jaką dotychczas chcemy wysłać nadawcy.

Kanał płatności można zamknąć tylko raz, po serii przelewów.
Tylko jedna z wysłanych wiadomości spowoduje wypłatę środków.
Dlatego każda wiadomość określa łączną kwotę należnych etherów,
a nie kwotę pojedynczej mikropłatności. Odbiorca w celu wypłaty
pieniędzy wybierze wiadomość z najwyższą kwotą. Nie musimy już
określać nonce dla każdej wiadomości, ponieważ kontrakt uznaje
tylko pojedynczą wiadomość. Natomiast wciąż podaje się adres
kontraktu, aby zapobiec wykorzystaniu tej samej wiadomości w
innych kanałach mikropłatności.

Poniżej znajduje się zmodyfikowany kod Javascript z poprzedniego
rozdziału, który kryptograficznie podpisuje wiadomość:

.. code-block:: javascript

    function constructPaymentMessage(contractAddress, amount) {
        return abi.soliditySHA3(
            ["address", "uint256"],
            [contractAddress, amount]
        );
    }

    function signMessage(message, callback) {
        web3.eth.personal.sign(
            "0x" + message.toString("hex"),
            web3.eth.defaultAccount,
            callback
        );
    }

    // contractAddress zapobiega atakom powtórzeniowym między kontraktami.
    // amount, w jednostce wei, określa ilość etherów, jaką chcemy wysłać.

    function signPayment(contractAddress, amount, callback) {
        var message = constructPaymentMessage(contractAddress, amount);
        signMessage(message, callback);
    }


Zamykanie kanału płatności
--------------------------

Kiedy Robert będzie gotowy na przyjęcie środków, to zamknie płatności,
wywołując funkcję ``close`` w inteligentnym kontrakcie. Spowoduje ona
wypłatę odbiorcy należnych etherów i zwrot reszty do Alicji. Aby zamknąć
kanał, Robert musi dostarczyć wiadomość podpisaną przez Alicję.

Inteligentny kontrakt musi zweryfikować, czy wiadomość zawiera prawidłowy
podpis nadawcy. Proces weryfikacji jest taki sam, jakiego używa odbiorca.
Funkcje w Solidity ``isValidSignature`` i ``recoverSigner`` działają
tak samo jak ich odpowiedniki w JavaScripcie z poprzedniego rozdziału,
druga z nich pochodzi z kontraktu ``ReceiverPays``.

Tylko odbiorca kanału płatności może wywołać funkcję ``close``, do której
przekazuje ostatnią wiadomość, zawierającą największą należną łączną kwotę.
Jeśli umożliwilibyśmy nadawcy wywołanie tej funkcji, mógłby on przekazać
wiadomość z niższą kwotą i oszukać odbiorcę.

Funkcja weryfikuje, czy podpisana wiadomość jest zgodna z przekazanymi
parametrami. Jeśli wszystko się zgadza, odbiorca otrzymuje swoją część
etherów, zaś reszta zostanie zwrócona do nadawcy przez ``selfdestruct``.
Możesz zobaczyć funkcję ``close`` w pełnym kodzie kontraktu.

Upływ daty ważności kanału
--------------------------

Robert może zamknąć kanał płatności w dowolnym czasie, ale jeśli o tym
zapomni, to Alicja potrzebuje odzyskać zabezpieczone środki w inny sposób.
Kiedy upłynie czas *expiration* określony w trakcie wdrażania kontraktu,
Alicja może wywołać ``claimTimeout``, aby odzyskać swoje pieniądze.
Zobacz funkcję ``claimTimeout`` w pełnym kodzie kontraktu.

Kiedy ta funkcja zostanie wywołana, Robert nie może już otrzymać etherów,
dlatego ważne jest, aby Robert zamknął kanał przed upływem daty ważności.

Pełny kontrakt
--------------

.. code-block:: solidity
    :force:

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;
    contract SimplePaymentChannel {
        address payable public sender;      // Konto, z którego wysyłamy płatności.
        address payable public recipient;   // Konto, które otrzymuje płatności.
        uint256 public expiration;  // Data ważności (jeśli odbiorca nie zamknie kanału).

        constructor (address payable recipientAddress, uint256 duration)
            payable
        {
            sender = payable(msg.sender);
            recipient = recipientAddress;
            expiration = block.timestamp + duration;
        }

        /// Odbiorca może zamknąć kanał w dowolnej chwili, przekazując podpisaną
		/// wiadomość z kwotą od nadawcy. Odbiorca otrzyma żądaną sumę, a reszta
		/// zostanie zwrócona do nadawcy.
        function close(uint256 amount, bytes memory signature) external {
            require(msg.sender == recipient);
            require(isValidSignature(amount, signature));

            recipient.transfer(amount);
            selfdestruct(sender);
        }

        /// nadawca może przedłużyć datę ważnośi w każdej chwili
        function extend(uint256 newExpiration) external {
            require(msg.sender == sender);
            require(newExpiration > expiration);

            expiration = newExpiration;
        }

        /// jeśli data ważności upłynie, a odbiorca nie zamknie kanału,
        /// wtedy ethery wracają do nadawcy.
        function claimTimeout() external {
            require(block.timestamp >= expiration);
            selfdestruct(sender);
        }

        function isValidSignature(uint256 amount, bytes memory signature)
            internal
            view
            returns (bool)
        {
            bytes32 message = prefixed(keccak256(abi.encodePacked(this, amount)));

            // sprawdź, czy podpis pochodzi od płatnika
            return recoverSigner(message, signature) == sender;
        }

        /// Wszystkie funkcje poniżej pochodzą z rozdziału
        /// 'tworzenie i weryfikacja podpisów'.

        function splitSignature(bytes memory sig)
            internal
            pure
            returns (uint8 v, bytes32 r, bytes32 s)
        {
            require(sig.length == 65);

            assembly {
                // pierwsze 32 bajtów, po prefiksie - długości
                r := mload(add(sig, 32))
                // kolejne 32 bajty
                s := mload(add(sig, 64))
                // ostatni bajt (pierwszy bajt z kolejnych 32 bajtów)
                v := byte(0, mload(add(sig, 96)))
            }

            return (v, r, s);
        }

        function recoverSigner(bytes32 message, bytes memory sig)
            internal
            pure
            returns (address)
        {
            (uint8 v, bytes32 r, bytes32 s) = splitSignature(sig);

            return ecrecover(message, v, r, s);
        }

        /// zwraca sumę kontrolną poprzedzoną prefiksem, aby naśladować zachowanie eth_sign.
        function prefixed(bytes32 hash) internal pure returns (bytes32) {
            return keccak256(abi.encodePacked("\x19Ethereum Signed Message:\n32", hash));
        }
    }


.. note::
  Funkcja ``splitSignature`` nie przeprowadza pełnej kontroli bezpieczeństwa.
  Rzeczywista implementacja powinna używać bardziej rygorystycznie przetestowanej biblioteki,
  takiej jak `openzeppelin <https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol>`_.

Weryfikacja płatności
---------------------

Wiadomości w kanale płatności nie powodują wypłaty środków od razu.
Odbiorca wypłaca środki, kiedy nadejdzie czas zamknąć kanał. Ważne,
aby odbiorca we własnym zakresie weryfikował każdą wiadomość.
Inaczej nie ma gwarancji, że będzie mógł on wybrać pieniądze.

Odbiorca powinien weryfikować każdą wiadomość następująco:

    1. Sprawdź, czy adres kontraktu w wiadomości zgadza się z adresem kanału płatności.
    2. Sprawdź, czy nowa suma etherów jest oczekiwaną kwotą.
    3. Sprawdź, czy nowa suma nie przekracza ilości zabezpieczonych etherów.
    4. Sprawdź, czy podpis jest poprawny i pochodzi od nadawcy kanału płatności.

Użyjemy biblioteki `ethereumjs-util <https://github.com/ethereumjs/ethereumjs-util>`_
do weryfikacji. Ostatni krok można wykonać na wiele sposobów, a my użyjemy JavaScriptu.
Poniższy kod zapożyczono z funkcji ``constructPaymentMessage`` z **kodu JavaScript** powyżej:

.. code-block:: javascript

    // naśladuje zachowanie prefiksowania metody eth_sign z JSON-RPC.
    function prefixed(hash) {
        return ethereumjs.ABI.soliditySHA3(
            ["string", "bytes32"],
            ["\x19Ethereum Signed Message:\n32", hash]
        );
    }

    function recoverSigner(message, signature) {
        var split = ethereumjs.Util.fromRpcSig(signature);
        var publicKey = ethereumjs.Util.ecrecover(message, split.v, split.r, split.s);
        var signer = ethereumjs.Util.pubToAddress(publicKey).toString("hex");
        return signer;
    }

    function isValidSignature(contractAddress, amount, signature, expectedSigner) {
        var message = prefixed(constructPaymentMessage(contractAddress, amount));
        var signer = recoverSigner(message, signature);
        return signer.toLowerCase() ==
            ethereumjs.Util.stripHexPrefix(expectedSigner).toLowerCase();
    }
