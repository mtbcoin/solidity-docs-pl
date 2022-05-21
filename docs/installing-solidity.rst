.. index:: ! installing

.. _installing-solidity:

###############################
Instalacja kompilatora Solidity
###############################

Wersje
======

Wersje Solidity nadawane są według zasad opisanych w `Semantic Versioning <https://semver.org/lang/pl/>`_.
Dodatkowo, wydania PATCH z wersją major 0 (np. 0.x.y) nie będą zawierać przełomowych zmian.
To znaczy, że kod, który kompiluje się w wersji 0.x.y, powinien się kompilować w wersji 0.x.z, gdzie z > y.

Oprócz wydań, dostarczamy **nocne robocze kompilacje**, aby umożliwić deweloperom przetestowanie nadchodzących
funkcjonalności i zebrać wczesne informacje zwrotne. Jednak choć nocne kompilacje są zazwyczaj bardzo stabilne,
to zawierają najnowszy kod z gałęzi roboczej i nie gwarantujemy, że zawsze będzie działał. Mimo naszych starań
mogą zawierać nieudokumentowane i/lub problematyczne zmiany, które nie trafią do aktualnego wydania.
Nie są przeznaczone do użytku produkcyjnego.

Kiedy wdrażasz kontrakt, powinieneś używać najnowszego wydania Solidity, ponieważ przełomowe zmiany, jak również
nowe funkcjonalności i poprawki błędów, są wprowadzane systematycznie. Aktualnie nadajemy numery wersji 0.x, aby
`podkreślić gwałtowne tempo rozwoju projektu <https://semver.org/lang/pl/#spec-item-4>`_.

Remix
=====

*Polecamy Remix dla małych kontraktów i do szybkiej nauki Solidity.*

`Uruchom Remix online <https://remix.ethereum.org/>`_, nie potrzebujesz niczego instalować.
Jeśli chcesz go używać bez dostępu do Internetu, wejdź na
https://github.com/ethereum/remix-live/tree/gh-pages i pobierz plik ``.zip`` tak jak
wyjaśniono na tej stronie. Remix to także wygodna opcja do testowania nocnych kompilacji
bez instalowania wielu wersji Solidity.

Kolejne akapity na tej stronie opisują instalację kompilatora Solidity obsługiwanego z wiersza poleceń
na swoim komputerze. Wybierz ten kompilator, jeśli pracujesz nad większym kontraktem lub potrzebujesz
więcej opcji kompilacji.

.. _solcjs:

npm / Node.js
=============

Użycie ``npm`` jest wygodnym i przenośnym sposobem na instalację ``solcjs``, kompilatora Solidity.
Program `solcjs` ma mniej możliwości niż opisany w dalszej części w pełni funkcjonalny kompilator.
Dokumentacja :ref:`commandline-compiler` zakłada, że korzystasz z pełnego kompilatora ``solc``.
Sposób użycia ``solcjs`` opisano w jego `repozytorium <https://github.com/ethereum/solc-js>`_.

Uwaga: Projekt solc-js stworzono na bazie napisanego w C++ `solc` przy pomocy Emscripten, co znaczy, że oba wykorzystują
ten sam kod źródłowy kompilatora. `solc-js` można używać bezpośrednio w projektach JavaScript (takich jak Remix).
Po szczegółowe instrukcje zajrzyj do repozytorium solc-js.

.. code-block:: bash

    npm install -g solc

.. note::

    Program wykonywalny w wierszu poleceń nazywa się ``solcjs``.

    Opcje wiersza poleceń programu ``solcjs`` nie są kompatybilne z ``solc`` i narzędzia (takie jak ``geth``)
    oczekujące zachowania ``solc`` nie będą działać z ``solcjs``.

Docker
======

Obrazy dockerowe kompilatora Solidity są dostępne pod nazwą ``solc`` w organizacji ``ethereum``.
Użyj tagu ``stable``, aby pobrać najnowsze stabilne wydanie lub ``nightly``, aby pobrać wydanie
z gałęzi roboczej zawierające potencjalnie niestabilne zmiany.

Obraz dockerowy uruchamia program wykonywalny kompilatora, więc możesz mu przekazać wszystkie argumenty kompilatora.
Na przykład poniższe polecenie ściąga wersję stabilną obrazu ``solc`` (jeśli jeszcze go nie masz)
i uruchamia go w nowym kontenerze, przekazując argument ``--help``.

.. code-block:: bash

    docker run ethereum/solc:stable --help

Możesz także sprecyzować numer wersji w tagu, np. 0.5.4.

.. code-block:: bash

    docker run ethereum/solc:0.5.4 --help

Aby wykorzystać obraz dockerowy do kompilacji plików na maszynie hosta,
zamontuj lokalny folder dla wejścia i wyjścia oraz określ kontrakt do kompilacji. Na przykład:

.. code-block:: bash

    docker run -v /local/path:/sources ethereum/solc:stable -o /sources/output --abi --bin /sources/Contract.sol

Możesz też użyć standardowego interfejsu JSON (który jest zalecany w przypadku użycia kompilatora z narzędziami).
Kiedy używasz tego interfejsu, nie trzeba montować żadnych folderów pod warunkiem, że JSON nie odwołuje się do
żadnych zewnętrznych plików, które miałyby być
:ref:`załadowane przez odwołanie import <initial-vfs-content-standard-json-with-import-callback>`).

.. code-block:: bash

    docker run ethereum/solc:stable --standard-json < input.json > output.json

Pakiety Linuksa
===============

Pakiety binarne Solidity są dostępne pod adresem
`solidity/releases <https://github.com/ethereum/solidity/releases>`_.

Mamy też pakiety PPA dla Ubuntu. Możesz pobrać ostanie stabilne wydanie za pomocą poleceń:

.. code-block:: bash

    sudo add-apt-repository ppa:ethereum/ethereum
    sudo apt-get update
    sudo apt-get install solc

Nocną kompilację można zainstalować za pomocą poniższych komend:

.. code-block:: bash

    sudo add-apt-repository ppa:ethereum/ethereum
    sudo add-apt-repository ppa:ethereum/ethereum-dev
    sudo apt-get update
    sudo apt-get install solc

Co więcej, niektóre dystrybucje Linuksa dostarczają swoje pakiety. Nie utrzymujemy ich bezpośrednio,
ale zazwyczaj społeczność dba o to, żeby były aktualne.

Na przykład pakiety Arch Linux dla najnowszej wersji roboczej:

.. code-block:: bash

    pacman -S solidity

Istnieją także `pakiety snao <https://snapcraft.io/solc>`_, lecz, **nie są obecnie utrzymywane**.
Da się je zainstalować we wszystkich `obsługiwanych dystrybucjach Linuksa <https://snapcraft.io/docs/core/install>`_.
Aby zainstalować najnowszą stabilną wersję solc:

.. code-block:: bash

    sudo snap install solc

Jeśli chcesz pomóc testować najnowszą wersję roboczą Solidity z ostatnimi zmianami, wpisz polecenie:

.. code-block:: bash

    sudo snap install solc --edge

.. note::

    Snap ``solc`` ściśle izoluje się od systemu. To najbezpieczniejszy tryb dla pakietów snap, ale zawiera ograniczenia,
    np. możesz operować na plikach wyłącznie w katalogach ``/home`` i ``/media``.
    Aby dowiedzieć się więcej, odwiedź `Demystifying Snap Confinement <https://snapcraft.io/blog/demystifying-snap-confinement>`_.

Pakiety macOS
=============

Rozpowszechniamy kompilator Solidity przez Homebrew do zbudowania ze źródeł. Aktualnie nie są dostępne
wstępnie zbudowane wersje (tzw. pre-built bottles).

.. code-block:: bash

    brew update
    brew upgrade
    brew tap ethereum/ethereum
    brew install solidity

Aby zainstalować najnowszą wersję 0.4.x / 0.5.x, wpisz ``brew install solidity@4``
lub odpowiednio ``brew install solidity@5``.

Jeśli potrzebujesz konkretnej wersji Solidity, możesz ją zainstalować bezpośrednio z GitHuba za pomocą Homebrew.

Zobacz
`commity solidity.rb na GitHubie <https://github.com/ethereum/homebrew-ethereum/commits/master/solidity.rb>`_.

Skopiuj hash commitu wersji, której potrzebujesz i wypróbuj ją na swoim komputerze.

.. code-block:: bash

    git clone https://github.com/ethereum/homebrew-ethereum.git
    cd homebrew-ethereum
    git checkout <your-hash-goes-here>

Zainstaluj ją poleceniem ``brew``:

.. code-block:: bash

    brew unlink solidity
    # eg. Install 0.4.8
    brew install solidity.rb

Statyczne binarki
=================

W folderze `solc-bin`_ utrzymujemy repozytorium zawierające statyczne kompilacjami poprzednich i aktualnych
wersji kompilatora dla wszystkich obsługiwanych platform. Tam znajdziesz też nocne kompilacje.

To repozytorium to nie tylko szybki i łatwy sposób dla użytkowników końcowych, aby pobrać gotowe do użycia
pliki binarne, ale także zapewnia udogodnienia dla zewnętrznych narzędzi:

- Zawartość jest powielana do https://binaries.soliditylang.org skąd może zostać łatwo pobrana przez HTTPS
  bez uwierzytelnienia, ograniczeń transferu lub potrzeby użycia Gita.
- Zawartość jest serwowana z poprawnymi nagłówkami `Content-Type` i mniej restrykcyjną polityką CORS, co
  umożliwia narzędziom działającym w przeglądarce na bezpośrednie pobieranie.
- Pliki binarne nie wymagają instalacji ani rozpakowania (za wyjątkiem starszych wersji dla systemu Windows
  z dołączonymi niezbędnymi bibliotekami DLL)
- Dążymy do wysokiego poziomu wstecznej zgodności. Pliki, po ich dodaniu, nie są usuwane lub przenoszone
  bez stworzenia skrótu/przekierowania w starej lokalizacji. Nie są też nigdy modyfikowane w tym miejscu
  i ich suma kontrolna nie powinna ulec zmianie. Jedynym wyjątkiem byłyby uszkodzone lub nieużywalne
  pliki, jeśli mogłyby spowodować więcej szkód niż dobrego, gdyby je pozostawić.
- Pliki są serwowane zarówno przez HTTP i HTTPS. Dopóki będziesz mógł pobrać listę plików w bezpieczny sposób
  (przez git, HTTPS, IPFS lub masz ją lokalnie w pamięci podręcznej) i zweryfikować sumy kontrolne binarek po
  ich pobraniu, nie musisz ich pobierać przez HTTPS.

Te same pliki binarne są w większości przypadków dostępne na `stronie wydania Solidity w GitHubie`_. Różnica jest taka,
że zasadniczo nie aktualizujemy stron starych wydań. To znaczy, że nie zmieniamy ich nazwy, jeśli zmieni się konwencja
nazewnicza i nie dodajemy kompilacji dla platform, które nie były obsługiwane w momencie wydania danej wersji.
Wyjątek stanowi ``solc-bin``.

Repozytorium ``solc-bin`` zawiera kilka głównych folderów. Każdy z nich dotyczy konkretnej platformy i zawiera plik
``list.json`` z listą dostępnych plików binarnych. Na przykład w ``emscripten-wasm32/list.json`` znajdziesz następujące
informacje o wersji 0.7.4:

.. code-block:: json

    {
      "path": "solc-emscripten-wasm32-v0.7.4+commit.3f05b770.js",
      "version": "0.7.4",
      "build": "commit.3f05b770",
      "longVersion": "0.7.4+commit.3f05b770",
      "keccak256": "0x300330ecd127756b824aa13e843cb1f43c473cb22eaf3750d5fb9c99279af8c3",
      "sha256": "0x2b55ed5fec4d9625b6c7b3ab1abd2b7fb7dd2a9c68543bf0323db2c7e2d55af2",
      "urls": [
        "bzzr://16c5f09109c793db99fe35f037c6092b061bd39260ee7a677c8a97f18c955ab1",
        "dweb:/ipfs/QmTLs5MuLEWXQkths41HiACoXDiH8zxyqBHGFDRSzVE5CS"
      ]
    }

To znaczy, że:

- Możesz znaleźć ten plik binarny w tym samym katalogu pod nazwą
  `solc-emscripten-wasm32-v0.7.4+commit.3f05b770.js <https://github.com/ethereum/solc-bin/blob/gh-pages/emscripten-wasm32/solc-emscripten-wasm32-v0.7.4+commit.3f05b770.js>`_.
  Pamiętaj, że ten plik może być dowiązaniem symbolicznym i będziesz musiał go rozwiązać samodzielnie, jeśli nie
  pobierasz go za pomocą Gita lub jeśli twój system nie obsługuje dowiązań symbolicznych.
- Kopia pliku znajduje się także pod adresem https://binaries.soliditylang.org/emscripten-wasm32/solc-emscripten-wasm32-v0.7.4+commit.3f05b770.js.
  W tym przypadku git nie jest potrzebny, a dowiązania symboliczne rozwiązywane są przez serwer, który zwraca
  albo kopię pliku, albo nagłówek HTTP z przekierowaniem.
- Plik dostępny jest także w IPFS po adresem `QmTLs5MuLEWXQkths41HiACoXDiH8zxyqBHGFDRSzVE5CS`_.
- Plik w przyszłości może być dostępny w Swarm pod adresem `16c5f09109c793db99fe35f037c6092b061bd39260ee7a677c8a97f18c955ab1`_.
- Możesz zweryfikować prawdziwość pliku, porównując jego sumę kontrolną keccak256 z
  ``0x300330ecd127756b824aa13e843cb1f43c473cb22eaf3750d5fb9c99279af8c3``. Sumę kontrolną możesz wygenerować z wiersza
  poleceń narzędziem ``keccak256sum`` dostarczanym przez `sha3sum`_ lub za pomocą `funkcji keccak256() z  ethereumjs-util`_ w JavaScripcie.
- Możesz też zweryfikować prawdziwość pliku, porównując jego sumę kontrolną sha256 z
  ``0x2b55ed5fec4d9625b6c7b3ab1abd2b7fb7dd2a9c68543bf0323db2c7e2d55af2``.

.. warning::

   Ponieważ kładziemy duży nacisk na zachowanie kompatybilności wstecznej, repozytorium  może zawierać
   trochę starych rzeczy, ale nie powinieneś ich używać do tworzenia nowych narzędzi:

   - Korzystaj z ``emscripten-wasm32/`` (z rezerwą do ``emscripten-asmjs/``) zamiast ``bin/`` jeśli zależy ci na
     najlepszej wydajności. Do wersji 0.6.1 dostarczaliśmy tylko pliki binarne asm.js.
     Od wersji przerzuciliśmy się na bardziej wydajne `kompilacje WebAssembly`_.
     Przebudowaliśmy starsze wersje na WASM, ale oryginalne asm.js wciąż znajdują się w ``bin/``.
     Nowe musiały zostać przeniesione do osobnego katalogu, aby zapobiec kolizjom.
   - Korzystaj z katalogów ``emscripten-asmjs/`` i ``emscripten-wasm32/`` zamiast ``bin/`` i ``wasm/``
     jeśli chcesz być pewny, że pobierasz pliki binarne wasm lub asm.js.
   - Korzystaj z ``list.json`` zamiast ``list.js`` i ``list.txt``. Format JSON zawiera wszystkie dotychczasowe
     informacje i jeszcze więcej.
   - Korzystaj z https://binaries.soliditylang.org zamiast https://solc-bin.ethereum.org. Aby uprościć sprawę,
     przenieśliśmy prawie wszystko, co jest związane z kompilatorem, do nowej domeny ``soliditylang.org``
     i dotyczy to także ``solc-bin``. Choć rekomendujemy nową domenę, wciąż w pełni utrzymujemy starą
     i zapewniamy, że będzie wskazywać to samo miejsce.

.. warning::

    Pliki binarne są także dostępne pod adresem https://ethereum.github.io/solc-bin/ ale ta strona nie
    jest już aktualizowana po wydaniu wersji 0.7.2 i nie otrzyma żadnych nowych wydań ani nocnych kompilacji
    dla jakiejkolwiek platformy. Nie zawiera też nowej struktury folderów, w tym kompilacji non-emscripten.

    Jeśli korzystasz z niej, przełącz się na https://binaries.soliditylang.org.
    To nam pozwala dokonywać zmian w hostingu w przezroczysty sposób i uniknąć zakłóceń w działaniu.
    W odróżnieniu od domeny ``ethereum.github.io``, nad którą nie mamy żadnej kontroli, gwarantujemy, że
    domena ``binaries.soliditylang.org`` będzie działać i zachowa taką samą strukturę URL przez długi czas.

.. _IPFS: https://ipfs.io
.. _Swarm: https://swarm-gateways.net/bzz:/swarm.eth
.. _solc-bin: https://github.com/ethereum/solc-bin/
.. _Solidity release page on github: https://github.com/ethereum/solidity/releases
.. _sha3sum: https://github.com/maandree/sha3sum
.. _keccak256() function from ethereumjs-util: https://github.com/ethereumjs/ethereumjs-util/blob/master/docs/modules/_hash_.md#const-keccak256
.. _WebAssembly builds: https://emscripten.org/docs/compiling/WebAssembly.html
.. _QmTLs5MuLEWXQkths41HiACoXDiH8zxyqBHGFDRSzVE5CS: https://gateway.ipfs.io/ipfs/QmTLs5MuLEWXQkths41HiACoXDiH8zxyqBHGFDRSzVE5CS
.. _16c5f09109c793db99fe35f037c6092b061bd39260ee7a677c8a97f18c955ab1: https://swarm-gateways.net/bzz:/16c5f09109c793db99fe35f037c6092b061bd39260ee7a677c8a97f18c955ab1/

.. _building-from-source:

Budowanie ze źródeł
===================

Wymagania - wszystkie systemy operacyjne
----------------------------------------

Poniższa tabela zawiera wymagania dla wszystkich kompilacji Solidity:

+-----------------------------------+-------------------------------------------------------+
| Programy                          | Notatki                                               |
+===================================+=======================================================+
| `CMake`_ (wersja 3.13+)           | Międzyplatformowy generator plików kompilacji.        |
+-----------------------------------+-------------------------------------------------------+
| `Boost`_ (wersja 1.77+ dla        | Biblioteki C++.                                       |
| Windowsa, 1.65+ dla innych)       |                                                       |
+-----------------------------------+-------------------------------------------------------+
| `Git`_                            | Narzędzie konsolowe do pobierania kodu źródłowego.    |
+-----------------------------------+-------------------------------------------------------+
| `z3`_ (wersja 4.8+, opcjonalnie)  | Do użytku z narzędziem sprawdzającym SMT.             |
+-----------------------------------+-------------------------------------------------------+
| `cvc4`_ (opcjonalnie)             | Do użytku z narzędziem sprawdzającym SMT.             |
+-----------------------------------+-------------------------------------------------------+

.. _cvc4: https://cvc4.cs.stanford.edu/web/
.. _Git: https://git-scm.com/download
.. _Boost: https://www.boost.org
.. _CMake: https://cmake.org/download/
.. _z3: https://github.com/Z3Prover/z3

.. note::
    Solidity do wersji 0.5.10 może poprawnie nie zlinkować się z Boost w wersjach 1.70+.
    Jednym z obejść jest zmiana nazwy ``<ścieżka instalacji Boost>/lib/cmake/Boost-1.70.0``
    przed uruchomieniem polecenia cmake w celu konfiguracji Solidity.

    Od wersji 0.5.10 linkowanie Boost 1.70+ powinno działać bez ręcznej interwencji.

.. note::
    Domyślne ustawienia kompilacji wymagają konkretnej wersji Z3 (najnowszej z momentu, w którym kod
    został zaktualizowany). Zmiany wprowadzane pomiędzy wydaniami Z3 często skutkują trochę innymi
    wynikami (ale wciąż poprawnymi). Nasze testy SMT nie uwzględniają tych różnic i prawdopodobnie
    nie przejdą w innej wersji niż ta, dla której zostały napisane. To nie znaczy, że kompilacja
    używająca innej wersji jest wadliwa. Jeśli przekażesz argument ``-DSTRICT_Z3_VERSION=OFF`` do CMake,
    możesz budować z dowolną wersją, która spełnia wymagania z tabeli powyżej. Jednak jeśli to zrobisz,
    to pamiętaj o dodaniu opcji ``--no-smt`` do ``scripts/tests.sh``, aby nie uruchamiać testów SMT.

Minimalna wersja kompilatora
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Można użyć poniższych kompilatorów C++ do zbudowania bazy kodu Solidity:

- `GCC <https://gcc.gnu.org>`_, wersja 8+
- `Clang <https://clang.llvm.org/>`_, wersja 7+
- `MSVC <https://visualstudio.microsoft.com/vs/>`_, wersja 2019+

Wymagania - macOS
-----------------

W przypadku macOS upewnij się, czy masz zainstalowaną
`najnowszą wersję Xcode <https://developer.apple.com/xcode/download/>`_.
Zawiera ona `kompilator C++ Clang <https://en.wikipedia.org/wiki/Clang>`_, środowisko
`Xcode IDE <https://en.wikipedia.org/wiki/Xcode>`_ i inne narzędzia deweloperskie Apple,
które są potrzebne do zbudowania aplikacji C++ dla OS X.
Jeśli instalujesz Xcode po raz pierwszy lub instalujesz nowszą wersję, musisz
zaakceptować warunki licencji zanim będziesz mógł użyć narzędzi konsolowych:

.. code-block:: bash

    sudo xcodebuild -license accept

Nasz skrypt budujący dla OS X wykorzystuje menedżer pakietów `Homebrew <https://brew.sh>`_
do instalacji zewnętrznych zależności.
Tutaj znajdziesz, jak `odinstalować Homebrew
<https://docs.brew.sh/FAQ#how-do-i-uninstall-homebrew>`_,
jeśli zechcesz kiedykolwiek zacząć od początku.

Wymagania - Windows
-------------------

Musisz zainstalować poniższe zależności, aby zbudować Solidity dla Windowsa:

+-------------------------------------+-------------------------------------------------------+
| Oprogramowanie                      | Opis                                                  |
+=====================================+=======================================================+
| `Visual Studio 2019 Build Tools`_   | kompilator C++                                        |
+-------------------------------------+-------------------------------------------------------+
| `Visual Studio 2019`_ (opcjonalnie) | kompilator C++ i środowisko deweloperskie             |
+-------------------------------------+-------------------------------------------------------+
| `Boost`_ (version 1.77+)            | biblioteki C++                                        |
+-------------------------------------+-------------------------------------------------------+

Jeśli masz już jedno środowisko IDE i potrzebujesz wyłącznie kompilatora oraz bibliotek,
możesz zainstalować tylko Visual Studio 2019 Build Tools.

Visual Studio 2019 zawiera IDE wraz z kompilatorem i potrzebnymi bibliotekami.
Tak więc jeśli nie masz żadnego IDE i zamierzasz rozwijać Solidity, to Visual Studio 2019
może być dobrym wyborem dla ciebie, aby wszystko łatwo skonfigurować.

Oto lista komponentów, jakie należy zainstalować wraz z Visual Studio 2019 Build Tools lub Visual Studio 2019:

* Visual Studio C++ core features
* VC++ 2019 v141 toolset (x86,x64)
* Windows Universal CRT SDK
* Windows 8.1 SDK
* C++/CLI support

.. _Visual Studio 2019: https://www.visualstudio.com/vs/
.. _Visual Studio 2019 Build Tools: https://www.visualstudio.com/downloads/#build-tools-for-visual-studio-2019

Mamy skrypt pomocniczy, którego możesz użyć do instalacji wszystkich zewnętrznych zależności:

.. code-block:: bat

    scripts\install_deps.ps1

Zainstaluje on ``boost`` oraz ``cmake`` w katalogu ``deps``.

Sklonuj repozytorium
--------------------

Aby sklonować kod źródłowy, wykonaj następujące polecenia:

.. code-block:: bash

    git clone --recursive https://github.com/ethereum/solidity.git
    cd solidity

Jeśli chcesz pomóc w rozwoju Solidity,
powinieneś sforkować projekt i dodać swoją osobistą kopię jako drugie zewnętrzne repozytorium:

.. code-block:: bash

    git remote add personal git@github.com:[username]/solidity.git

.. note::
    W ten sposób zbudujesz wstępną wersję kompilatora, co poskutkuje tym, że wygenerowany przez niego
    kod bajtowy będzie zawierał flagę. Jeśli chcesz ponownie zbudować wydaną już wersję kompilatora
    Solidity, to pobierz archiwum z kodem źródłowym ze strony:

    https://github.com/ethereum/solidity/releases/download/v0.X.Y/solidity_0.X.Y.tar.gz

    (nie ze strony "Source code" na GitHubie).

Budowanie z wiersza poleceń
---------------------------

**Zainstaluj zewnętrzne zależności (patrz wyżej) zanim przejdziesz dalej.**

Projekt Solidity używa CMake do konfiguracji procesu budowania.
Możesz chcieć zainstalować `ccache`_ aby przyspieszyć ponowną budowę.
CMake wykryje go automatycznie.
Budowanie Solidity wygląda podobnie jak pod Linuksem, macOS i wielu innych systemach uniksowych:

.. _ccache: https://ccache.dev/

.. code-block:: bash

    mkdir build
    cd build
    cmake .. && make

lub nawet łatwiej - pod Linuksem i macOS, możesz wywołać:

.. code-block:: bash

    #note: ten skrypt zainstaluje solc oraz soltest w usr/local/bin
    ./scripts/build.sh

.. warning::

    kompilacje dla BSD powinny działać, ale nie są przetestowane przez zespół Solidity.

A pod Windowsem:

.. code-block:: bash

    mkdir build
    cd build
    cmake -G "Visual Studio 16 2019" ..

Jeśłi chcesz użyć wersji Boost zainstalowanej przez ``scripts\install_deps.ps1``, musisz dodatkowo przekazać argumenty
``-DBoost_DIR="deps\boost\lib\cmake\Boost-*"`` i ``-DCMAKE_MSVC_RUNTIME_LIBRARY=MultiThreaded`` do wywołania ``cmake``.

W efekcie powinien zostać utworzony plik **solidity.sln** w tym folderze.
Klikając podwójnie powinieneś uruchomić Visual Studio. Zalecamy budować konfigurację **Release**,
ale wszystkie inne też działają.

Ewentualnie możesz dokonać kompilacji pod Windowsa z linii poleceń, np.:

.. code-block:: bash

    cmake --build . --config Release

Opcje CMake
===========

Jeśli chcesz wiedzieć, jakie opcje udostępnia CMake, wywołaj ``cmake .. -LH``.

.. _smt_solvers_build:

SMT Solvers
-----------
Solidity można zbudować z wykorzystaniem solverów SMT. Jeśli zostaną one wykryte w systemie,
to zostaną dołączone do kompilacji. Każdy solver można wyłączyć za pomocą opcji `cmake`.

*Note: W niektórych przypadkach można w ten sposób obejść błędy kompilacji.*

Ponieważ są domyślnie włączone, będąc w folderze kompilacji, możesz je wyłączyć jak w przykładzie poniżej.

.. code-block:: bash

    # wyłącza tylko Z3 SMT Solver.
    cmake .. -DUSE_Z3=OFF

    # wyłącza tylko CVC4 SMT Solver.
    cmake .. -DUSE_CVC4=OFF

    # wyłącza oba Z3 i CVC4
    cmake .. -DUSE_CVC4=OFF -DUSE_Z3=OFF

Oznaczenie wersji
=================

Ciąg tekstowy z oznaczeniem wersji Solidity składa się z 4 części:

- numeru wersji
- znacznika pre-release, zazwyczaj ``develop.YYYY.MM.DD`` lub ``nightly.YYYY.MM.DD``
- commitu w formacie ``commit.GITHASH``
- platformy, która ma dowolną liczbę składowych, zawierających szczegóły o platformie i kompilatorze

Jeśli są lokalne zmiany, commit będzie zawierał końcówkę ``.mod``.

Te części są połączone, tak jak wymaga tego SemVer, gdzie znacznik pre-release jest taki sam jak w przypadku SemVer,
a commit i platforma razem tworzą metadane kompilacji SemVer.

Przykład wydania końcowego: ``0.4.8+commit.60cc1668.Emscripten.clang``.

Przykład wydania przedpremierowego: ``0.4.9-nightly.2017.1.17+commit.6ecb4aa3.Emscripten.clang``

Ważna uwaga na temat wersjonowania
==================================

Po wydaniu wersji, poziom wersji "patch" jest podbijany, ponieważ zakładamy, że zmieni się tylko poziom "patch".
Kiedy zmiany są scalane, wersja powinna zostać podbita zgodnie z SemVer oraz istotnością zmian. Ostatecznie wydanie
otrzymuje taki sam numer wersji jak wydanie nocne, ale bez słowa ``prerelease``.

Przykład:

1. Wydana zostaje wersja 0.4.0.
2. Od tej pory nocne wydanie ma numer wersji 0.4.1.
3. Wprowadzono mało istotne zmiany --> numer wersji nie zmienia się.
4. Wprowadzono przełomową zmianę --> numer wersji jest podnoszony do 0.5.0.
5. Wydana zostaje wersja 0.5.0.

Takie zachowanie działa poprawnie z :ref:`dyrektywą version <version_pragma>`.
