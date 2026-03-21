<div align="right">
  <a href="README.md">🇬🇧 English</a> | 🇵🇱 Polski
</div>

# Witaj w OctoTS!


Tradycyjne bazy danych serii czasowych (takie jak InfluxDB czy kdb+) wymagają dedykowanej infrastruktury, płatnych licencji oraz stałego monitoringu. Dla mniejszych projektów, narzędzi wewnętrznych lub danych publicznych stosowanie tak złożonych systemów jest nieproporcjonalnym narzutem operacyjnym. Dodatkowo dane przechowywane w zewnętrznych chmurach stają się często „czarną skrzynką” - bez skomplikowanych logów audytowych trudno precyzyjnie śledzić, kto i kiedy zmodyfikował konkretny wpis.

Projekt OctoTS to rozwiązanie mające na celu opracowanie ustandaryzowanego protokołu oraz zestawu narzędzi do przechowywania serii czasowych w formie plików tekstowych bezpośrednio wewnątrz repozytorium Git. Głównym założeniem systemu jest wykorzystanie mechanizmów kontroli wersji jako natywnego magazynu danych z pełną historią zmian. Dzięki temu metryki techniczne są przechowywane razem z kodem źródłowym. Gwarantuje to ich spójność z historią rozwoju oprogramowania i pozwala całkowicie zrezygnować z zewnętrznych baz danych.
System opiera się na procesach ciągłej integracji GitHub Actions, które okresowo lub po określonych zdarzeniach uruchamiają skrypty pomiarowe. Wyniki są automatycznie dopisywane do plików tekstowych i zatwierdzane w repozytorium jako kolejne commity. 
Każdy punkt danych zostaje trwale powiązany z unikalnym identyfikatorem zmiany, co zapewnia pełną spójność historyczną całego zbioru informacji.

Dopełnieniem rozwiązania jest warstwa prezentacji danych stworzona w technologii React. Jest ona osadzona i udostępniana bezpośrednio w środowisku GitHub poprzez usługę GitHub Pages. Moduł ten odpowiada za pobieranie surowych informacji z plików w repozytorium i ich transformację w interaktywne wykresy oraz zestawienia. Dzięki temu użytkownik może w czasie rzeczywistym, bez opuszczania platformy GitHub, analizować zmiany parametrów oprogramowania, porównywać dane z różnych okresów oraz natychmiastowo wykrywać trendy lub regresje.