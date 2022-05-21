Solidity
========

Solidity jest obiektowym językiem wysokiego poziomu do implementacji inteligentnych umów.
Inteligentne kontrakty to programy, które zarządzają zachowaniem kont w stanie Ethereum.

Solidity to język wykorzystujący nawiasy klamrowe, zaprojektowany na potrzeby maszyny wirtualnej Ethereum (EVM).
Czerpie inspiracje z języków C++, Python i JavaScript. Więcej szczegółów o tym, jakimi językami jest inspirowany
Solidity, możesz znaleźć w sekcji :doc:`inspiracje innymi językami <language-influences>`.

Solidity jest statycznie typowany. Obsługuje między innymi dziedziczenie, biblioteki
i złożone typy definiowane przez użytkownika.

W Solidity możesz tworzyć inteligentne kontrakty dla takich zastosowań jak głosowanie, publiczne zbiórki pieniędzy,
przetargi i portfele wielosygnaturowe.

Kiedy wdrażasz kontrakty, powinieneś używać najnowszej wersji Solidity.
Oprócz wyjątkowych sytuacji tylko najnowsza wersja otrzymuje
`poprawki bezpieczeństwa <https://github.com/ethereum/solidity/security/policy#supported-versions>`_.
Ponadto regularnie wprowadzane są przełomowe zmiany, jak i nowe funkcje.
Obecnie używamy numerów wersji 0.y.z, `aby oznaczyć szybkie tempo zmian <https://semver.org/#spec-item-4>`_.

.. warning::

  Ostatnio wyszła wersja Solidity 0.8.x, która wprowadza wiele przełomowych
  zmian. Zapoznaj się z :doc:`ich pełną listą <080-breaking-changes>`.

Pomysły na ulepszenie Solidity lub tej dokumentacji są zawsze mile widziane.
Przeczytaj nasz :doc:`poradnik dla tłumaczy <contributing>`

.. Hint::

  Możesz pobrać dokumentację w formacie PDF, HTML lub Epub, klikając na pływające menu
  wersji w lewym dolnym rogu i wybierając preferowany format do pobrania.


Wprowadzenie
------------

**1. Zrozum podstawy Inteligentnych Kontraktów**

Jeśli nie znasz koncepcji inteligentnych kontaktów, zalecamy, abyś zaczął od zgłębienia
sekcji "Wprowadzenie do Inteligentnych Kontaktów", która obejmuje:

* :ref:`Prosty przykładowy inteligentny kontrakt <simple-smart-contract>` napisany w Solidity.
* :ref:`Podstawy Blockchain <blockchain-basics>`.
* :ref:`Maszyna wirtualna Ethereum <the-ethereum-virtual-machine>`.

**2. Poznaj Solidity**

Kiedy oswoisz się z podstawami, zalecamy, abyś przeczytał sekcje :doc:`"Solidity na przykładach" <solidity-by-example>`
i „Opis języka”, aby zrozumieć jego główne koncepcje.

**3. Zainstaluj kompilator Solidity**

Istnieje wiele sposobów instalacji kompilatora Solidity.
Wybierz preferowaną opcję i postępuj zgodnie ze wskazówkami na :ref:`stronie instalacji <installing-solidity`.

.. hint::

  Możesz uruchamiać przykładowe kody bezpośrednio w przeglądarce w środowisku `Remix IDE <https://remix.ethereum.org>`_.
  Remix działa w przeglądarce internetowej i pozwala na pisanie, wdrażanie i administrowanie inteligentnymi
  kontraktami bez potrzeby instalowania Solidity na lokalnym komputerze.

.. warning::

    Ponieważ oprogramowanie piszą ludzie, może zawierać błędy. Powinieneś przestrzegać dobrych praktyk
    podczas tworzenia swoich inteligentnych kontraktów. Obejmuje to przegląd kodu, testowanie, audyty
    i dowody poprawności. Użytkownicy inteligentnych kontraktów często są bardziej odważni od ich twórców,
    a łańcuchy bloków i inteligentne kontrakty mają swoje specyficzne problemy, na jakie trzeba uważać,
    więc zanim zaczniesz pisać kod, przeczytaj rozdział :ref:`security_considerations`.

**4. Dowiedz się więcej**

Jeśli chcesz dowiedzieć się więcej o budowaniu zdecentralizowanych aplikacji w Ethereum,
na stronie `Ethereum Developer Resources <https://ethereum.org/en/developers/>`_ znajdziesz
dokumentację ogólną Ethereum oraz duży zbiór przewodników, narzędzi i frameworków.

Jeśli masz jakieś pytania, możesz szukać odpowiedzi lub zapytać na
`Ethereum StackExchange <https://ethereum.stackexchange.com/>`_ lub
naszym `kanale Gitter <https://gitter.im/ethereum/solidity/>`_.

.. _translations:

Tłumaczenia
-----------

Społeczność pomaga w tłumaczeniu tego dokumentu na inne języki.
Zwróć uwagę, że niektóre są nieukończone lub nieaktualne.
Wersja angielska stanowi wzorzec.

Możesz przełączać się między językami, klikając pływające menu w lewym dolnym rogu i wybierając odpowiedni język.

* `Francuski <https://docs.soliditylang.org/fr/latest/>`_
* `Indonezyjski <https://github.com/solidity-docs/id-indonesian>`_
* `Perski <https://github.com/solidity-docs/fa-persian>`_
* `Japoński <https://github.com/solidity-docs/ja-japanese>`_
* `Koreański <https://github.com/solidity-docs/ko-korean>`_
* `Chiński <https://github.com/solidity-docs/zh-cn-chinese/>`_

.. note::

  Ostatnio utworzyliśmy nową organizację na GitHubie i ustaliliśmy przepływ pracy, aby usprawnić wysiłki społeczności.
  Przeczytaj `poradnik tłumacza <https://github.com/solidity-docs/translation-guide>`_
  aby uzyskać informacje, jak zacząć tłumaczyć na nowy język lub udzielać się w istniejących tłumaczeniach.

Tematy
========

:ref:`Keyword Index <genindex>`, :ref:`Search Page <search>`

.. toctree::
   :maxdepth: 2
   :caption: Podstawy

   introduction-to-smart-contracts.rst
   installing-solidity.rst
   solidity-by-example.rst

.. toctree::
   :maxdepth: 2
   :caption: Opis języka

   layout-of-source-files.rst
   structure-of-a-contract.rst
   types.rst
   units-and-global-variables.rst
   control-structures.rst
   contracts.rst
   assembly.rst
   cheatsheet.rst
   grammar.rst

.. toctree::
   :maxdepth: 2
   :caption: Kompilator

   using-the-compiler.rst
   analysing-compilation-output.rst
   ir-breaking-changes.rst

.. toctree::
   :maxdepth: 2
   :caption: Mechanizmy wewnętrzne

   internals/layout_in_storage.rst
   internals/layout_in_memory.rst
   internals/layout_in_calldata.rst
   internals/variable_cleanup.rst
   internals/source_mappings.rst
   internals/optimizer.rst
   metadata.rst
   abi-spec.rst

.. toctree::
   :maxdepth: 2
   :caption: Materiały dodatkowe

   050-breaking-changes.rst
   060-breaking-changes.rst
   070-breaking-changes.rst
   080-breaking-changes.rst
   natspec-format.rst
   security-considerations.rst
   smtchecker.rst
   resources.rst
   path-resolution.rst
   yul.rst
   style-guide.rst
   common-patterns.rst
   bugs.rst
   contributing.rst
   brand-guide.rst
   language-influences.rst
