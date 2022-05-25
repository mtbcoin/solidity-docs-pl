*******************************
Układ pliku źródłowego Solidity
*******************************

Pliki źródłowe mogą zawierać dowolną ilość 
:ref:`definicji kontraktów<contract_structure>`, import_ , dyrektyw
:ref:`pragma<pragma>` i :ref:`using for<using-for>` oraz definicji
:ref:`struktur<structs>`, :ref:`wyliczeń<enums>`, :ref:`funkcji<functions>`, :ref:`błędów<errors>` i :ref:`stałych<constants>`.

.. index:: ! license, spdx

Identyfikator licencji SPDX
===========================

Zaufanie do inteligentnych kontraktów wzrasta, jeśli ich kod źródłowy
jest publiczne dostępny. Ponieważ udostępnianie kodu źródłowego zawsze
powoduje problemy prawne dotyczące praw autorskich, kompilator Solidity
zachęca do używania czytelnych dla maszyn `identyfikatorów licencji SPDX <https://spdx.org>`_.
Każdy plik źródłowy powinien zaczynać się od komentarza określającego jego licencję:

``// SPDX-License-Identifier: MIT``

Kompilator nie sprawdza, czy licencja znajduje się na
`liście dopuszczalnych licencji SPDX <https://spdx.org/licenses/>`_, ale
dołącza podany ciąg tekstowy do :ref:`metadanych kodu bajtowego <metadata>`.

Jeśli nie chcesz określać licencji lub jeśli kod źródłowy nie jest open-source,
to wpisz specjalną wartość ``UNLICENSED``. Pamiętaj, że ``UNLICENSED`` 
(nie występuje na liście licencji SPDX) to co innego niż ``UNLICENSE`` 
(przyznaje każdemu prawa do wszystkiego).
Solidity przestrzega `zaleceń npm <https://docs.npmjs.com/cli/v7/configuring-npm/package-json#license>`_.

Dodanie tego komentarza oczywiście nie zwalnia ciebie z innych
obowiązków związanych z licencjonowaniem, takich jak konieczność
wstawienia nagłówka konkretnej licencji w każdym pliku źródłowym
lub podania posiadacza praw autorskich.

Ten komentarz jest rozpoznawany przez kompilator w dowolnym miejscu
w pliku, ale zaleca się umieścić go na samym początku.

Więcej informacji, jak używać identyfikatorów licencji SPDX,
możesz znaleźć na `stronie SPDX <https://spdx.org/ids-how>`_.


.. index:: ! pragma

.. _pragma:

Dyrektywy pomocnicze pragma
===========================

Słowo kluczowe ``pragma`` służy do włączania pewnych funkcji kompilatora
lub sprawdzeń. Dyrektywy pragma zawsze dotyczą bieżącego pliku źródłowego,
więc musisz je dodać do wszystkich plików, jeśli chcesz je zastosować do
całego projektu. Jeśli :ref:`zaimportujesz<import>` inny plik, dyrektywy
z tego pliku *nie* zostaną automatycznie zastosowane do importującego go pliku.

.. index:: ! pragma, version

.. _version_pragma:

Dyrektywy wersji
----------------

Pliki źródłowe mogą (i powinny) zawierać adnotację z numerem wersji Solidity,
aby zapobiec kompilacji przez przyszłe wersje kompilatora, które mogłyby
wprowadzić niekompatybilne zmiany. Staramy się ograniczać takie zmiany do
niezbędnego minimum i wprowadzać je w taki sposób, że zmiany w semantyce
wymagają zmian w składni, ale nie zawsze jest to możliwe. Z tego powodu
zalecamy przeczytać listę zmian przynajmniej dla wydań, które zawierają
znaczące zmiany. Takie wydania zawsze mają wersje ``0.x.0`` lub ``x.0.0``.

Oznaczenie wersji wygląda następująco: ``pragma solidity ^0.5.2;``

Plik źródłowy z powyższą linią nie skompiluje się w starszych wersjach
kompilatora niż 0.5.2, ale także w werjach zaczynających się od 0.6.0 
(drugi warunek dodano symbolem ``^``). Ponieważ nie będzie przełomowych
zmian do wersji ``0.6.0``, możesz być pewny, że twój kod się skompiluje
i nie zmieni zachowania. Nie wymuszono dokładnej wersji kompilatora,
więc wersje z poprawionymi błędami są wciąż dopuszczalne.

Można określić bardziej złożone reguły dla wersji kompilatora zgodnie ze
składnią używaną przez `npm <https://docs.npmjs.com/cli/v6/using-npm/semver>`_.

.. note::
  Dodanie dyrektywy pragma z numerem wersji *nie zmienia* wersji kompilatora,
  **nie** włącza ani **nie** wyłącza funkcji kompilatora. Po prostu nakazuje
  kompilatori sprawdzić, czy jego numer wersji zgadza się z numerem wymaganym
  przez dyrektywę pragma. Jeśli się nie zgadza, kompilator zgłasza błąd.

Dyrektywa ABI Coder
-------------------

Za pomocą dyrektywy ``pragma abicoder v1`` lub ``pragma abicoder v2`` możesz
przełączać się między dwoma implementacjami kodera i dekodera ABI.

Nowy koder ABI (v2) potrafi zakodować i odkodować dowolnie zagnieżdżone tablice
i struktury. Może produkować mniej optymalny kod i nie przeprowadzono na nim
tyle testów, co na poprzednim koderze, ale od wersji Solidity 0.6.0 nie jest
już uznawany za eksperymentalny. Jeśli chcesz go używać, to musisz go jawnie
włączyć za pomocą dyrektywy ``pragma abicoder v2;``. Od wersji Solidity 0.8.0
będzie domyślnie włączony, ale istnieje możliwość przywrócenia starego kodera
za pomocą``pragma abicoder v1;``.

Zestaw typów obsługiwanych przez nowy koder to ścisły nadzbiór zestawu typów
obsługiwanych przez stary koder. Kontrakty, które używają nowego kodera, mogą
bez ograniczeń komunikować się z kontraktami, które używają starego kodera.
Komunikacja na odwrót jest możliwa pod warunkiem, że kontrakt używający starego
kodera nie próbuje tworzyć wywołań wymagających dekodowania typów obsługiwanych
wyłącznie przez nowy koder. Kompilator to wykryje i wyświetli błąd. Włączenie 
``abicoder v2`` w twoim kontrakcie wystarczy, aby pozbyć się błędu.

.. note::
  Ta dyrektywa zostanie zastosowana do całego kodu zdefiniowanego w pliku,
  w którym zostanie umieszczona, niezależnie od tego, w którym miejscu ten
  kod się kończy. To znaczy, że kontrakt, który jest kompilowany z ABI v1,
  może wciąż zawierać kod używający nowego kodera, dziedzicząc go z innego
  kontraktu. To jest dopuszczalne, jeśli nowe typy używane są tylko
  wewnętrznie, a nie w sygnaturach zewnętrznych funkcji.
  
.. note::
  Do wersji Solidity 0.7.4 można było włączyć koder ABI v2 za pomocą 
  ``pragma experimental ABIEncoderV2``, ale nie dało się jawnie
  wybrać kodera v1, ponieważ był on włączony domyślnie.

.. index:: ! pragma, experimental

.. _experimental_pragma:

Experimental Pragma
-------------------

Drugą dyrektywą jest experimental pragma. Służy do włączenia funkcji
kompilatora lub języka, które nie są jeszcze domyślnie włączone.
Aktualnie obsługiwane są poniższe eksperymenalne opcje:


ABIEncoderV2
~~~~~~~~~~~~

Ponieważ koder ABI v2 nie jest już uznawany za eksperymentalny,
można go włączyć za pomocą ``pragma abicoder v2`` (patrz wyżej)
od Solidity 0.7.4.

.. _smt_checker:

SMTChecker
~~~~~~~~~~

Ten komponent musi zostać włączony podczas budowania kompilatora Solidity,
dlatego nie jest on dostępny we wszystkich plikach binarnych Solidity.
:ref:`Instrukcja kompilacji<smt_solvers_build>` objaśnia, jak aktywować tę opcję.
Jest ona włączona w paczkach PPA dla Ubuntu w większości wersji,
ale nie w obrazach Dockera, kompilacjach dla Windowsa ani w statycznie
budowanych binarkach dla Linuksa. Możesz ją włączyć w solc-js za pomocą
`smtCallback <https://github.com/ethereum/solc-js#example-usage-with-smtsolver-callback>`_ 
jeśli masz lokalnie zainstalowany solver SMT i uruchomisz solc-js z node (nie z przeglądarki).

Jeśli użyjesz ``pragma experimental SMTChecker;``, wtedy zobaczysz dodatkowe
:ref:`ostrzeżenia dotyczące bezpieczeństwa<formal_verification>` które są
uzyskiwane poprzez odpytanie solvera SMT.
Ten komponent wciąż nie obsługuje wszystkich funkcji języka Solidity i na pewno
wyświetla dużo ostrzeżeń. Jeśli zgłasza nieobsługiwane funkcje, to analiza może
być niepełna.

.. index:: source file, ! import, module, source unit

.. _import:

Importowanie innych plików źródłowych
=====================================

Składnia i semantyka
--------------------

Solidity obsługuje instrukcje import, aby umożliwić podział kodu
na moduły w taki sam sposób jak w JavaScripcie (od ES6).
Nie obsługuje jednak `default export <https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/export#Description>`_.

Na poziomie globalnym możesz użyć instrukcji import w następujący sposób:

.. code-block:: solidity

    import "filename";

Część ``filename`` nazywana jest *ścieżką importu*.
Ta instrukcja importuje wszystkie globalne symbole z "filename" (wraz z zaimportowanymi tam symbolami)
do bieżącej przestrzeni symboli globalnych (inaczej niż w ES6 ale kompatybilnie wstecznie z Solidity).
Nie zaleca się importować całego modułu, ponieważ zaśmiecasz przestrzeń nazw w nieprzewidywalny sposób.
Jeśli dodasz nowe elementy do "filename" na najwyższym poziomie, pojawią się one automatycznie we
wszystkich plikach, które importują cały "filename". Lepiej importować konkretne symbole.

Poniższy przykład tworzy nowy globalny symbol ``symbolName``, który zawiera
wszystkie globalne symbole z ``"filename"``:

.. code-block:: solidity

    import * as symbolName from "filename";

w efekcie wszystkie globalne symbole są dostępne poprzez ``symbolName.symbol``.

Inny wariant tej składni, który nie jest częścią ES6, ale potencjalnie użyteczny, to:

.. code-block:: solidity

  import "filename" as symbolName;

co jest odpowiednikiem ``import * as symbolName from "filename";``.

Jeśli wystąpi kolizja nazw, możesz zaimportować symbole pod inną nazwą. 
Poniższy kod tworzy nowe globalne symbole  ``alias`` i ``symbol2``,
które są referencją do ``symbol1`` i ``symbol2`` z ``"filename"``.

.. code-block:: solidity

    import {symbol1 as alias, symbol2} from "filename";

.. index:: virtual filesystem, source unit name, import; path, filesystem path, import callback, Remix IDE

Ścieżki importu
---------------

Aby móc obsługiwać deterministyczne kompilacje na wszystkich platformach, kompilator Solidity musi
ukryć szczegóły systemu plików, w którym są przechowywane pliki źródłowe.
Dlatego ścieżki importu nie odwołują się bezpośrednio do plików.
Zamiast tego kompilator tworzy wewnętrzą bazę danych (*wirtualny system plików* lub skrótowo *VFS*),
gdzie każda jednostka źródłowa otrzymuje unikalną *nazwę jednostki źródłowej*, która jest
nieprzezroczystym i nieustrukturyzowanym identyfikatorem.
Ścieżka importu określona w instrukcji import jest tłumaczona na nazwę jednostki źródłowej i używana
do znalezienia odpowiadającej jednostki źródłowej w tej bazie danych.

Za pomocą :ref:`standardowego API JSON <compiler-api>` możesz bezpośrednio przekazać nazwy i zawartość wszystkich plików źródłowych na wejście kompilatora.
W tym przypadku nazwy jednostek źródłowych dowolne. Jednak jeśli chcesz, aby kompilator automatycznie
znalazł i załadował kod źródłowy do VFS, nazwy twoich jednostek źródłowych powinny być zorganizowane
w taki spsób, aby instrukcja :ref:`import callback <import-callback>` mogła je odnaleźć.
Kiedy używamy kompilatora z wiersza poleceń, domyślny import callback obsługuje tylko wczytywanie kodu źródłowego z lokalnego systemu plików hosta, co oznacza, że nazwy twoich jednostek źródłowych muszą być ścieżkami. Niektóre środowiska dostarczają bardziej uniwersalne callbacki.
Na przykład `Remix IDE <https://remix.ethereum.org/>`_ pozwala `importować pliki z HTTP, IPFS i Swarm URLs lub odwołać się bezpośrednio do paczek z rejestru NPM
<https://remix-ide.readthedocs.io/en/latest/import.html>`_.

Pełny opis wirtualnego systemu plików i logikę rozwiązywania ścieżki znajdziesz w rozdziale :ref:`Rozwiązywanie ścieżek <path-resolution>`.

.. index:: ! comment, natspec

Komentarze
==========

Jednolinijkowe komentarze (``//``) i wielolinijkowe komentarze (``/*...*/``) są obsługiwane.

.. code-block:: solidity

    // To jest jednolinijkowy komentarz.

    /*
    To jest
    wielolinijkowy komentarz.
    */

.. note::
  Jednolinijkowy komentarz kończy się dowolnym znakiem końca linii zgodnym z Unicode
  (LF, VF, FF, CR, NEL, LS or PS) w kodowaniu UTF-8. Terminator jest wciąż częścią
  kodu źródłowego po komentarzu, więc jeśli nie jest symbolem ASCII (NEL, LS i PS),
  doprowadzi do błędu parsowania.

Dodatkowo istnieje inny typ komentarza zwany komentarzem NatSpec, który
opisano w rozdziale :ref:`style guide<style_guide_natspec>`. Zaczynają
się od potrójnego ukośnika (``///``) lub bloku z podwójną gwiazdką (``/** ... */``)
i powinny znajdować się bezpośrednio nad deklaracjami funkcji lub instrukcjami.
