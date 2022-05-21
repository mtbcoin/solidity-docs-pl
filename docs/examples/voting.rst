.. index:: voting, ballot

.. _voting:

**********
Głosowanie
**********

Poniższy kontrakt jest dość złożony, ale pokazuje dużo możliwości Solidity. Implementuje kontrakt głosowania.
Oczywiście, głównym problemem przeprowadzania elektronicznych wyborów jest to, jak udzielić prawa głosu właściwym
osobom i jak zapobiec manipulacjom. Nie rozwiążemy tutaj wszystkich problemów, ale przynajmniej pokażemy, jak
stworzyć głosowanie przez pełnomocnika w taki sposób, że liczenie głosów jest automatyczne i całkowicie przejrzyste.

Pomysł polega na tym, aby utworzyć jeden kontrakt dla każdej karty do głosowania i określić krótkie nazwy dla
poszczególnych opcji. Wtedy twórca kontraktu, przewodniczący komisji wyborczej, udzieli głosu każdemu adresowi osobno.

Posiadacze tych adresów mogą wybrać, czy chcą głosować samodzielnie, czy przekazać głos innej osobie, której ufają.

Kiedy upłynie czas na oddanie głosów, funkcja ``winningProposal()`` zwróci kandydata z największą liczbą głosów.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;
    /// @title Głosowanie przez pełnomocnika.
    contract Ballot {
        // Tworzy nowy złożony typ danych, który zostanie użyty
        // później. Będzie opisywał pojedynczego wyborcę.
        struct Voter {
            uint weight; // waga zgromadzona przez delegację
            bool voted;  // jeśli true, ta osoba już głosowała
            address delegate; // osoba, której przekazano głos
            uint vote;   // numer kandydata (pozycja na liście)
        }

        // To jest typ danych dla pojedynczego kandydata.
        struct Proposal {
            bytes32 name;   // krótka nazwa (do 32 bajtów)
            uint voteCount; // liczba zgromadzonych głosów
        }

        address public chairperson;

        // Deklaruje zmienną stanu, która przypisuje strukturę
        // `Voter` dla każdego możliwego adresu.
        mapping(address => Voter) public voters;

        // Dynamicznie rozszerzalna tablica struktur `Proposal`.
        Proposal[] public proposals;

        /// Tworzy nową kartę do głosowania z listą kandydatów `proposalNames`.
        constructor(bytes32[] memory proposalNames) {
            chairperson = msg.sender;
            voters[chairperson].weight = 1;

            // Dla każdej nazwy kandydata stwórz nowy obiekt
            // i dodaj go na koniec tablicy.
            for (uint i = 0; i < proposalNames.length; i++) {
                // `Proposal({...})` tworzy tymczasowy obiekt
                // Proposal, a `proposals.push(...)` dodaje go
                // na koniec tablicy `proposals`.
                proposals.push(Proposal({
                    name: proposalNames[i],
                    voteCount: 0
                }));
            }
        }

        // Udziela `voter` prawa głosu dla tej karty wyborczej.
        // Może ją wywołać tylko `chairperson`.
        function giveRightToVote(address voter) external {
            // Jeśli pierwszy argument `require` wyniesie
            // `false`, to wykonywanie kodu zakończy się
            // i wszystkie zmiany stanu zostaną wycofane,
            // a salda etherów przywrócone.
            // W starych wersjach EVM to zużywało całe paliwo,
            // ale obecnie tak się nie dzieje.
            // Jest dobrą praktyką używać `require` do sprawdzenia,
            // czy funkcje są wywoływane prawidłowo.
            // Jako drugi argument możesz przekazać wyjaśnienie,
            // co poszło nie tak.
            require(
                msg.sender == chairperson,
                "Tylko szef komisji wyborczej może udzielić głosu."
            );
            require(
                !voters[voter].voted,
                "Wyborca już głosował."
            );
            require(voters[voter].weight == 0);
            voters[voter].weight = 1;
        }

        /// Przekazuje głos innej osobie z adresem `to`.
        function delegate(address to) external {
            // przypisuje referencję
            Voter storage sender = voters[msg.sender];
            require(!sender.voted, "Już głosowałeś.");

            require(to != msg.sender, "Przekazanie głosu samemu sobie jest zabronione.");

            // Przekaż głos tak daleko, dokąd `to` oddał swój głos.
            // Ogólnie takie pętle są bardzo niebezpieczne,
            // ponieważ jeśli wykonują się zbyt długo,
            // potrzebują więcej paliwa niż jest dostępnego w bloku.
            // W tym przypadku głos nie zostanie przekazany,
            // ale w innych sytuacjach takie pętle mogą
            // całkowicie zawiesić kontrakt.
            while (voters[to].delegate != address(0)) {
                to = voters[to].delegate;

                // Znaleźliśmy pętlę w delegacji, niedopuszczalne.
                require(to != msg.sender, "Znaleziono pętlę w delegacji.");
            }

            // Ponieważ `sender` jest referencją, zostanie
            // zmodyfikowany `voters[msg.sender].voted`
            Voter storage delegate_ = voters[to];

            // Wyborcy nie mogą przekazać głosu osobom, które nie mogą głosować.
            require(delegate_.weight >= 1);
            sender.voted = true;
            sender.delegate = to;
            if (delegate_.voted) {
                // Jeśli delegat już głosował, bezpośrednio
                // dodaj do liczby głosów kandydata
                proposals[delegate_.vote].voteCount += sender.weight;
            } else {
                // Jeśli delegat jeszcze nie głosował,
                // zwiększ jego wagę.
                delegate_.weight += sender.weight;
            }
        }

        /// Oddaj swój głos (wliczając głosy przekazane tobie)
        /// na kandydata `proposals[proposal].name`.
        function vote(uint proposal) external {
            Voter storage sender = voters[msg.sender];
            require(sender.weight != 0, "Nie masz prawa głosu");
            require(!sender.voted, "Już głosowałeś.");
            sender.voted = true;
            sender.vote = proposal;

            // Jeśli `proposal` jest poza zakresem tablicy, zostanie
            // rzucony wyjątek, a wszystkie zmiany zostaną wycofane.
            proposals[proposal].voteCount += sender.weight;
        }

        /// @dev Zwraca zwycięskiego kandydata, biorąc pod uwagę wszystkie wcześniejsze głosy.
        function winningProposal() public view
                returns (uint winningProposal_)
        {
            uint winningVoteCount = 0;
            for (uint p = 0; p < proposals.length; p++) {
                if (proposals[p].voteCount > winningVoteCount) {
                    winningVoteCount = proposals[p].voteCount;
                    winningProposal_ = p;
                }
            }
        }

        // Wywołuje funkcję winningProposal(), aby pobrać numer
        // zwycięskiego kandydata zawartego w tablicy kandydatów
        // i zwraca jego nazwę
        function winnerName() external view
                returns (bytes32 winnerName_)
        {
            winnerName_ = proposals[winningProposal()].name;
        }
    }


Możliwe udoskonalenia
=====================

Aktualnie potrzebujemy wielu transakcji, aby nadać prawo głosu
wszystkim wyborcom. Możesz pomyśleć nad lepszym sposobem?
