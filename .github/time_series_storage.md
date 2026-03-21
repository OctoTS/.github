<div align="right">
  <a href="time_series_storage_en.md">🇬🇧 English</a> | 🇵🇱 Polski
</div>

# Time Series Storage - Research

## Wprowadzenie

Szeregi czasowe (time series) to dane zapisywane w czasie, np. metryki CI/CD, wydajność, monitoring.

Celem dokumentu jest przegląd sposobów ich przechowywania w kontekście projektu OctoTS, ale nie tylko.

## 1. Plain text

#### Opis
Plain text to najprostszy możliwy sposób przechowywania danych szeregów czasowych, w którym każda linia reprezentuje pojedynczy pomiar. Brak narzuconej ścisłej struktury, sposób zapisu zależy od implementacji.

#### Zalety
- Maksymalna prostota.
- Uniwersalność – każdy system operacyjny i język programowania potrafi czytać i zapisywać pliki tekstowe.
- Minimalny narzut - brak znaczników, cudzysłowów czy separatorów, plik może być tak mały jak tylko konieczne

#### Wady
- Brak informacji o strukturze - plik sam w sobie nie mówi, co reprezentują liczby
- Wymaga własnych konwencji formatowania
- Brak walidacji – łatwo o błąd, który nie zostanie wykryty.

#### Źródła
- https://en.wikipedia.org/wiki/Plain_text

---

## 2. Tekstowe formaty plików

### 2.1 CSV (Comma-Separated Values)

#### Opis
CSV to najprostszy tekstowy format wymiany danych, w którym każda linia reprezentuje jeden rekord, a pola oddzielone są przecinkami (lub innym separatorem, np. średnikiem).

#### Przykład
```csv
timestamp,value1,value2
2026-03-01 14:30:00,120,22.4
2026-03-02 14:30:00,133,13.2
```

#### Zalety
- Uniwersalność - odczytywany przez praktycznie każde narzędzie analityczne, arkusze kalkulacyjne, skrypty w Pythonie, R, itp.
- Czytelność - można go otworzyć w edytorze tekstu i szybko sprawdzić dane.
- Prostota generowania i parsowania.

#### Wady
- Brak wydajnej kompresji - pliki szybko rosną, zwłaszcza przy wysokiej częstotliwości próbkowania.
- Problemy z danymi zawierającymi separator (np. przecinki w tekstach)
- Nie wspiera metadanych (np. jednostek, opisów serii) - trzeba je przechowywać osobno.

#### Źródła
- https://en.wikipedia.org/wiki/Comma-separated_values

---

### 2.2 JSON (JavaScript Object Notation)

#### Opis
JSON to format tekstowy do reprezentowania danych strukturalnych. Jest powszechnie używany w API oraz aplikacjach webowych.

#### Przykład
```json
[
  {
    "timestamp": "2026-03-01 14:30:00",
    "value1": 120,
    "value2": 22.4,
    "tags": {
      "host": "server01",
      "region": "eu"
    }
  },
  {
    "timestamp": "2026-03-02 14:30:00",
    "value1": 133,
    "value2": 13.2,
    "tags": {
      "host": "server01",
      "region": "eu"
    }
  }
]
```

#### Zalety
- Czytelność i uniwersalność - format tekstowy łatwy do odczytania przez człowieka, a jednocześnie maszynowo parsowalny w praktycznie każdym języku programowania.
- Elastyczność schematu - można dodawać nowe pola (np. dodatkowe metadane, tagi) bez konieczności migracji istniejących danych.
- Wsparcie dla struktur zagnieżdżonych - umożliwia przechowywanie złożonych metadanych bezpośrednio przy pomiarach (np. tagi, lokalizacje, informacje o czujniku).

#### Wady
- Większy rozmiar plików niż CSV - nazwy pól powtarzają się w każdym rekordzie, więcej interpunkcji
- Mniej czytelne diffy w Git - ze względu na formatowanie i zagnieżdżenia, drobne zmiany mogą powodować duże różnice w diffie, trudniej śledzić historię w systemach kontroli wersji.
- Trudniejsze dopisywanie (append) - dopisanie nowego rekordu wymaga przeparsowania całego pliku, zamknięcia nawiasu i dodania przecinka przed nowym elementem.

#### Źródła
- https://www.json.org/json-en.html
- https://en.wikipedia.org/wiki/JSON

---

### 2.3 JSON Lines (JSONL / NDJSON)

#### Opis
JSON Lines (JSONL), znany również jako NDJSON (Newline Delimited JSON), to format tekstowy, w którym każdy wiersz pliku jest osobnym obiektem JSON. Format jest zaprojektowany do przetwarzania strumieniowego oraz łatwego dopisywania danych (append-only). Jest często używany w logach, systemach big data oraz pipeline'ach danych.

#### Przykład
```json
{"timestamp": "2026-03-01 14:30:00", "value1": 120, "value2": 22.4, "tags": {"host": "server01", "region": "eu"}}
{"timestamp": "2026-03-02 14:30:00", "value1": 133, "value2": 23.2, "tags": {"host": "server01", "region": "eu"}}
```

#### Zalety
- Wszystkie zalety JSON.
- Efektywny append - wystarczy dopisać linię.
- Czytelniejsze diffy - zmiany widoczne jako zmiana jednej linii, nie całego bloku.

#### Wady
- Nadal większy rozmiar niż CSV
- Brak walidacji spójności na poziomie pliku - każda linia jest interpretowana niezależnie, możliwe niespójności schematu między rekordami.

#### Źródła
- https://jsonlines.org/

---

### 2.4 XML (eXtensible Markup Language)

#### Opis
XML to tekstowy format danych służący do przechowywania i przesyłania danych strukturalnych. Umożliwia definiowanie własnych znaczników (tagów), co pozwala na tworzenie złożonych struktur danych.

#### Przykład
```xml
<timeseries>
  <point>
    <time>2026-03-01 14:30:00</time>
    <value1>120</value1>
    <value2>22.4</value2>
  </point>
  <point>
    <time>2026-03-02 14:30:00</time>
    <value1>133</value1>
    <value2>13.2</value2>
  </point>
</timeseries>
```

#### Zalety
- Czytelność.
- Hierarchiczność - naturalna reprezentacja złożonych relacji (np. czujnik → seria → obserwacja).
- Bogactwo metadanych - możliwość dołączania dowolnych informacji opisowych (jednostki, lokalizacje, uwagi) bezpośrednio w strukturze danych.
- Walidowalność - możliwość zdefiniowania schematu (DTD, XML Schema, RelaxNG) i automatycznego sprawdzania poprawności danych.

#### Wady
- Duży narzut danych - znaczniki otwierające i zamykające wielokrotnie powtarzają nazwy pól, jeszcze większe rozmiary plików niż CSV czy JSON
- Mniej czytelne diffy.
- Trudniejszy append.
- Trudność w edycji ręcznej - ręczna edycja dużych plików jest uciążliwa, łatwo popełnić błędy składniowe.

#### Źródła
- https://en.wikipedia.org/wiki/XML

---

### 2.5 TimeSeries.JSON

#### Opis
TimeSeries.io to projekt open source udostępniony przez firmę Geosoft, który definiuje sposób przechowywania i przesyłania danych szeregów czasowych. Bazuje na formacie TimeSeries.JSON, rozszerzeniu JSON zaprojektowanym specjalnie dla danych czasowych.
Plik ma strukturę tablicy obiektów, każdy z nich składa się z trzech części:
- header - metadane opisujące cały zbiór.
- signals - tablica definicji sygnałów.
- data - tablica z właściwymi danymi pomiarowymi.

#### Przykład
```json
[
  {
    "header": {
      "TimeSeries.JSON": "1.0",
      "name": "New York City Marathon",
      "description": "R. Chepngetich",
      "source": "Garmin Fenix 8",
      "organization": "Runners World",
      "license": "Creative Commons BY-NC",
      "location": [40.785091, -73.968285],
      "timeStart": "2024-12-02T00:00:00.000Z",
      "timeEnd": "2024-12-02T02:09:56.000Z",
      "timeStep": null
    },
    "signals": [
      {
        "name": "time",
        "description": null,
        "quantity": "time",
        "unit": "s",
        "valueType": "datetime",
        "dimensions": 1
      },
      {
        "name": "latitude",
        "description": null,
        "quantity": "angle",
        "unit": "degA",
        "valueType": "float",
        "dimensions": 1
      },
      {
        "name": "longitude",
        "description": null,
        "quantity": "angle",
        "unit": "degA",
        "valueType": "float",
        "dimensions": 1
      },
      {
        "name": "elevation",
        "description": null,
        "quantity": "length",
        "unit": "m",
        "valueType": "float",
        "dimensions": 1
      }
    ],
    "data": [
      ["2024-12-02T00:00:00.000Z", 40.60305, -74.0556192, 27],
      ["2024-12-02T00:03:07.000Z", 40.60490, -74.0499759, -2],
      ["2024-12-02T00:07:04.000Z", 40.60721, -74.0428090, -2],
      ["2024-12-02T00:11:18.000Z", 40.60965, -74.0351057,  4],
      ["2024-12-02T00:12:44.000Z", 40.61053, -74.0325308, 16],
      ["2024-12-02T00:13:51.000Z", 40.61162, -74.0308571, 11],
      ["2024-12-02T00:14:53.000Z", 40.61294, -74.0298057, 18],
      ["2024-12-02T00:16:15.000Z", 40.61452, -74.0280890, 16],
      ["2024-12-02T00:17:43.000Z", 40.61627, -74.0263295, 12],
      ["2024-12-02T00:19:22.000Z", 40.61771, -74.0289688, 24]
    ]
 }
]
```

#### Zalety
- Czytelność.
- Elastyczność – obsługa sygnałów jednowymiarowych i wielowymiarowych, różnych typów danych, metadanych, jednostek i wartości brakujących.
- Ekosystem narzędzi – gotowe biblioteki w kilku popularnych językach, integracja z Excelem i Matlabem.

#### Wady
- Wady standardowego JSON
- Niszowość.
- Niepotrzebna złożoność dla prostych przypadków.

#### Źródła
- https://geosoft.no/
- https://github.com/geosoft-as/timeseries-io

---

### 2.6 MRTG Log Format (Multi Router Traffic Grapher)

#### Opis
MRTG to jedno z najstarszych narzędzi do monitorowania ruchu sieciowego, które zapisuje dane szeregów czasowych w prostych plikach tekstowych (logach) oraz generuje na ich podstawie wykresy HTML. Został zaprojektowany, aby oszczędzać miejsce poprzez stosowanie mechanizmu stopniowego zmniejszania rozdzielczości danych historycznych. Plik ma stałą strukturę: timestamp, ruch przychodzący, ruch wychodzący.

#### Przykład
```
1710681600 125000 98000
1710678000 118000 92000
1710674400 130000 100000
1710670800 110000 87000
1710667200 105000 86000
1710663600 99000 80000
```

#### Zalety
- Bardzo prosty, tekstowy format.
- Automatyczne zagęszczanie starszych danych (downsampling), co oszczędza miejsce.

#### Wady
- Specyficzny tylko dla metryk typu licznik.
- Brak elastyczności - stała liczba kolumn.
- Format trudny do integracji z nowoczesnymi narzędziami.

#### Źródła
- https://oss.oetiker.ch/mrtg/
- https://en.wikipedia.org/wiki/Multi_Router_Traffic_Grapher

---

### 2.7 .ts (sktime)

#### Opis
Format .ts został stworzony specjalnie na potrzeby biblioteki sktime - narzędzia do uczenia maszynowego dla danych szeregów czasowych w Pythonie.
Plik .ts składa się z nagłówka i sekcji z danymi. Nagłówek zawiera następujące metadane: nazwa zbioru danych, czy dane zawierają jawne znaczniki czasu, czy dane są jednowymiarowe, czy dane zawierają etykiety klas (z listą wartości jeżeli true).

#### Przykład
```
@problemName MultivariateExample
@timeStamps false
@univariate false
@classLabel true 0 1
@data
2,3,2,4:4,3,2,2:0
13,12,32,12:22,23,12,32:1
4,?,5,4:3,2,3,2:0
```

#### Zalety
- Wspiera złożone przypadki (multivariate, różne długości).
- Integracja z ekosystemem Python - bezpośrednie wczytanie do struktur sktime i dalsze przetwarzanie

#### Wady
- Niszowość - używany głównie w środowisku sktime i pokrewnych projektach badawczych. Poza tym kontekstem jest rzadko spotykany.
- Duży rozmiar plików.
- Trudniejszy append.

#### Źródła
- https://www.sktime.net/en/v0.24.2/examples/data/loading_data.html#Representing-data-with-.ts-files

---

### 2.8 TSD/DAT (Innovyze Time Series Data Format)

#### Opis
Format TSD/DAT został opracowany przez firmę Innovyze do przechowywania dużych ilości danych szeregów czasowych pochodzących z różnych źródeł, szczególnie w zastosowaniach związanych z gospodarką wodną i monitoringiem sieci wodociągowych.
Składa się z dwóch typów plików:
- plik TSD - zawiera metadane i definicje punktów pomiarowych
- pliki DAT - zawierają właściwe dane czasowe dla danego dnia (nazwa pliku w formacie YYYY-MM-DD.dat)

#### Przykład
TSD:
```
[TSD_VERSION=3.0]
[SYSTEM_TYPE=Radcom logger]
FO120716,FO12 STATION FLOW,Flow,m3/h,USED,0,500
FO120717,FO12 BOREHOLE FLOW,Flow,m3/h,USED,0,500
```

DAT:
```
_00:00
FO120716,238.0952, 1
FO120717,102.3199, 1
_00:21
FO120716,236.0195, 1
FO120717,102.3199, 2
FO120718,3.2451, 1
FO120719,2.2073, 1
```

#### Zalety
- Separacja metadanych i danych - zmiana opisu kanału, jednostek czy progów ważności nie wymaga modyfikacji ogromnych plików z pomiarami.
- Efektywne przechowywanie - dane są grupowane w pliki dzienne, co ułatwia archiwizację, przeszukiwanie i ewentualne usuwanie starych danych.
- Oszczędność miejsca - klucz kanału (8 znaków) jest krótki w porównaniu do powtarzania pełnych nazw w każdym rekordzie (jak w JSON).
- Możliwość walidacji danych (min/max, flagi)

#### Wady
- Format przystosowany konkretnie pod potrzeby firmy, nie nadaje się do szerszego wykorzystania
- Dwie powiązane struktury - konieczność utrzymywania spójności między plikami TSD i DAT (klucze muszą być zgodne), zarządzanie strukturą katalogów

#### Źródła
- https://help2.innovyze.com/infoworkswspro/Content/HTML/WS/r_time_series_data_file_format.htm
- https://help2.innovyze.com/infoworkswspro/Content/HTML/WS/a_time_series_data_files.htm

---

## 3. Formaty binarne

Formaty binarne są zaprojektowane z myślą o wysokiej wydajności i oszczędności miejsca. Są najlepszym wyborem dla dużych systemów produkcyjnych i analitycznych, jednak nie nadają się do projektów opartych o Git, gdzie kluczowa jest czytelność i możliwość śledzenia zmian.

---

### 3.1 HDF5 (Hierarchical Data Format version 5)

#### Opis
Zaawansowany, hierarchiczny format binarny umożliwiający przechowywanie ogromnych, złożonych zbiorów danych naukowych, w tym wielowymiarowych tablic z bogatymi metadanymi.

#### Cechy
- Bardzo wysoka wydajność.
- Bogate metadane i samoopisująca się struktura.
- Szeroko stosowany w naukach przyrodniczych, inżynierii i przemyśle.
- Wymaga specjalistycznych bibliotek.

#### Źródła
- https://www.hdfgroup.org/solutions/hdf5/
- https://en.wikipedia.org/wiki/Hierarchical_Data_Format

---

### 3.2 Apache Parquet

#### Opis
Kolumnowy, binarny format plików open-source, popularny w ekosystemie big data (Apache Spark, Hadoop) do wydajnego przechowywania danych, w tym szeregów czasowych, z naciskiem na kompresję i szybkie zapytania.

#### Cechy
- Bardzo dobra kompresja (Różne metody np. Snappy, Gzip).
- Kolumnowe przechowywanie - redukuje ilość odczytywanych danych przy zapytaniach agregujących.
- Wsparcie dla złożonych typów danych i zagnieżdżonych struktur.

#### Źródła
- https://parquet.apache.org/
- https://github.com/apache/parquet-format

---

### 3.3 Feather

#### Opis
Lekki, kolumnowy format binarny zaprojektowany z myślą o maksymalnej wydajności odczytu i zapisu oraz łatwej wymianie danych między różnymi językami analizy danych, zwłaszcza Python i R.

#### Cechy
- Maksymalna szybkość operacji I/O.
- Idealny do przechowywania tymczasowego i szybkiej wymiany danych.
- Brak wsparcia dla dużych systemów analitycznych.

#### Źródła
- https://arrow.apache.org/docs/python/feather.html
- https://github.com/wesm/feather

---

### 3.4 ORC (Optimized Row Columnar)

#### Opis
Kolumnowy format plików open-source, stworzony specjalnie dla ekosystemu Hadoop jako optymalizacja dla Apache Hive.

#### Cechy
- Najwyższa efektywność kompresji.
- Wysoka wydajność zapytań.
- Optymalizacja pod kątem Hive i ekosystemu Hadoop.

#### Źródła
- https://orc.apache.org/
- https://en.wikipedia.org/wiki/Apache_ORC

---

### 3.5 NetCDF (Network Common Data Form)

#### Opis
Format danych naukowych, szczególnie popularny w meteorologii, oceanografii i klimatologii do przechowywania wielowymiarowych danych zorientowanych na czas i przestrzeń.

#### Cechy
- Obsługa danych wielowymiarowych (np. czas × przestrzeń).
- Wbudowane metadane (opis zmiennych, jednostki).
- Bardzo wydajny dla dużych zbiorów danych.
- Bogaty ekosystem narzędzi (NCO, CDO, xarray) i integracja z językami programowania.

#### Źródła
- https://www.unidata.ucar.edu/software/netcdf/
- https://en.wikipedia.org/wiki/NetCDF

---

### 3.6 TsFile (Time Series File)

#### Opis
Format zaprojektowany i rozwijany przez Apache IoTDB, stworzony z myślą o specyficznych wymaganiach Przemysłowego Internetu Rzeczy (IIoT).

#### Cechy
- Zoptymalizowany pod dane czasowe.
- Bardzo wysoka wydajność zapisu i odczytu.
- Obsługa dużych wolumenów danych z czujników.
- Dobra kompresja redukująca koszty przechowywania.

#### Źródła
- https://tsfile.apache.org/
- https://github.com/apache/tsfile

---

### 3.7 TeaFile

#### Opis
Format plików płaskich do przechowywania dużych zbiorów danych czasowych zaprojektowany z myślą o prostocie i maksymalnej prędkości.

#### Cechy
- Bardzo szybki dostęp do danych.
- Proste API dostępne dla C#, C++, Python i R.
- Samoopisujący się nagłówek przechowujący schemat danych.

#### Źródła
- https://github.com/discretelogics/TeaFiles.Net-Time-Series-Storage-in-Files

---

## 4. Bazy danych do przechowywania szeregów czasowych

Bazy danych do szeregów czasowych (TSDB) są zoptymalizowane pod przechowywanie danych uporządkowanych w czasie, umożliwiają szybkie zapytania, agregacje i skalowanie przy dużych wolumenach danych.

---

### 4.1 InfluxDB

#### Opis
Popularna TSDB typu open-source, zoptymalizowana pod dane metryczne i logi z IoT oraz monitoring systemów.

#### Cechy
- Kolumnowa architektura oparta na Arrow/Parquet.
- Obsługa zapytań w języku InfluxQL podobnym do SQL.
- Kompresja danych i retencja historyczna.

#### Źródła
- https://www.influxdata.com/
- https://docs.influxdata.com/influxdb/

---

### 4.2 TimescaleDB

#### Opis
Rozszerzenie PostgreSQL dla danych szeregów czasowych, które łączy relacyjną strukturę z wydajnością TSDB.

#### Cechy
- Pełna zgodność z PostgreSQL i standardowym SQL.
- Zaawansowane funkcje czasowe: hipertabele, ciągłe agregaty, kompresja kolumnowa.
- Hybrydowy silnik składowania (Hypercore) łączący wiersze i kolumny.
- Ograniczenia wydajnościowe wynikające z architektury PostgreSQL.

#### Źródła
- https://www.timescale.com/
- https://docs.timescale.com/

---

### 4.3 OpenTSDB

#### Opis
Rozproszona baza danych na Hadoop/HBase do przechowywania dużych ilości danych metrycznych w czasie rzeczywistym.

#### Cechy
- Skalowalność dzięki architekturze opartej na HBase.
- Wymaga utrzymywania klastra HBase – wysoka złożoność operacyjna.
- Własny, nie-SQL-owy język zapytań.
- Starsza generacja TSDB, rzadziej wybierana w nowych projektach.

#### Źródła
- http://opentsdb.net/
- https://github.com/OpenTSDB/opentsdb

---

### 4.4 Prometheus

#### Opis
TSDB typu open-source, popularna w monitoringu aplikacji i systemów, szczególnie w środowisku Kubernetes.

#### Cechy
- Model pull - serwer sam scrapuje metryki z exporterów.
- Wbudowany język zapytań PromQL.
- Ścisła integracja z Kubernetes i ekosystemem Cloud Native.
- Idealny do monitorowania, ale nie do długoterminowej archiwizacji.

#### Źródła
- https://prometheus.io/
- https://prometheus.io/docs/introduction/overview/

---

### 4.5 VictoriaMetrics

#### Opis
Lekka, wydajna TSDB zaprojektowana jako wydajne rozwiązanie monitoringowe, które może służyć zarówno jako długoterminowe przechowywanie dla Prometheusa, jak i jego samodzielny zamiennik.

#### Cechy
- Bardzo szybkie zapisy i odczyty.
- Skalowalność pionowa i horyzontalna.
- Szeroka kompatybilność protokołów.
- Niskie zużycie pamięci.

#### Źródła
- https://victoriametrics.com/
- https://github.com/VictoriaMetrics/VictoriaMetrics

---

### 4.6 Kdb+/q

#### Opis
Baza danych komercyjna do analizy danych wysokiej częstotliwości, popularna w finansach i tradingu.

#### Cechy
- Ekstremalna wydajność dla dużych wolumenów danych.
- Własny język zapytań q.
- Wbudowane funkcje agregacji i analizy statystycznej.
- Komercyjna licencja, niszowa w zastosowaniach naukowych.

#### Źródła
- https://kx.com/
- https://en.wikipedia.org/wiki/Kdb%2B

## 5. Dane binarne serializowane do tekstu

Technika ta jest potężnym narzędziem zwłaszcza w architekturach IoT (np. w sieciach LoRaWAN, Sigfox) oraz wszędzie tam, gdzie zoptymalizowane czasowo i przestrzennie struktury binarne muszą zostać „przepchnięte” przez API akceptujące wyłącznie tekst (jak klasyczne endpointy REST JSON).

---

### 5.1 Protocol Buffers (Protobuf)

#### Opis
Opracowany przez Google format binarnej serializacji danych, szeroko stosowany w komunikacji między mikrousługami (np. gRPC) oraz w systemach IoT. Wymaga zdefiniowania ścisłego schematu danych w pliku `.proto`, na podstawie którego generowany jest kod do odczytu i zapisu dla wybranego języka programowania.

#### Przykład
Ponieważ dane są binarne, sam przesyłany ciąg bajtów jest nieczytelny (np. jako `08 a0 8d` ...). Podstawą jest jednak plik schematu definiujący strukturę.
Plik `sensor.proto`:

```proto
syntax = "proto3";

message DataPoint {
  int64 timestamp = 1;
  float value = 2;
  string device_id = 3;
}
```

#### Zalety
- Ścisłe typowanie: Struktura jest znana z góry i walidowana, co zapobiega wielu błędom w aplikacjach rozproszonych.
- Wysoka kompresja przestrzenna: Znacznie mniejszy rozmiar wiadomości w porównaniu do JSON czy XML, ponieważ nazwy kluczy (np. "timestamp") nie są przesyłane – zastępują je krótkie, liczbowe identyfikatory zdefiniowane w schemacie (np. 1, 2).
- Kompatybilność: Umożliwia dodawanie nowych pól do schematu bez psucia starszych klientów (wsteczna i przyszła kompatybilność).

#### Wady
- Brak czytelności: Bez pliku .proto dane są całkowicie nieczytelne.
- Narzut wdrożeniowy: Wymaga kroku kompilacji schematów (narzędzie protoc), co utrudnia szybkie prototypowanie.
- Brak elastyczności ad-hoc: Trudno używać go w systemach, w których struktura danych często i dynamicznie się zmienia.

#### Źródła
- https://protobuf.dev/
- https://en.wikipedia.org/wiki/Protocol_Buffers
- https://github.com/protocolbuffers/protobuf

---

### 5.2 MessagePack

#### Opis
MessagePack określa sam siebie jako "wydajny jak struktury binarne, wygodny jak JSON". Jest to format serializacji, który pozwala wymieniać dane strukturalne pomiędzy różnymi językami, nie wymagając przy tym definiowania wcześniejszych schematów.

#### Przykład
MessagePack mapuje struktury zbieżne z JSON w skompresowany ciąg bajtów.
Odpowiednik JSON: `{"temp": 22.5}`
W zapisie szesnastkowym (Hex) MessagePack wygeneruje np.:
`81 a4 74 65 6d 70 cb 40 36 80 00 00 00 00 00`
Gdzie `81` oznacza mapę z 1 elementem, `a4` ciąg znaków o długości 4, dalej kody ASCII dla "temp", a `cb` oznacza 64-bitowy float, po którym następuje wartość `22.5.`

#### Zalety
- Brak wymogu schematu (Schemaless): Działa na takiej samej zasadzie jak JSON, pozwalając na wstawianie dowolnych kluczy "w locie".
- Ekstremalnie prosta migracja: W wielu językach (np. Python, Node.js) przejście z formatu JSON wymaga jedynie podmiany biblioteki i zmiany funkcji pakującej (np. json.dumps() na msgpack.packb()).
- Natywne typy binarne: Wsparcie dla wstawiania surowych tablic bajtów bez konieczności kodowania ich do Base64 (co jest zmorą w JSON).

#### Wady
- Większy rozmiar niż Protobuf: Ze względu na przesyłanie nazw kluczy wraz z danymi w każdym rekordzie, paczki są większe niż w formatach opartych na schemacie.

- Wolniejsze parsowanie: Dekoder musi odczytywać strukturę i klucze w locie, co narzuca pewien koszt procesora (choć wciąż znacznie niższy niż dla tekstu).

#### Źródła
- https://msgpack.org/
- https://github.com/msgpack/msgpack

---

### 5.3 CBOR (Concise Binary Object Representation)

#### Opis
Standaryzowany przez IETF (RFC 8949) format danych zaprojektowany specjalnie pod kątem urządzeń o ekstremalnie ograniczonych zasobach (np. mikrokontrolery zasilane bateryjnie w sieciach IoT). Działa podobnie do MessagePack, ale kładzie nacisk na standardy internetowe.

#### Przykład
Podobnie jak w MessagePack, struktury ad-hoc są zamieniane na zwięzłe bajty.
Dla odpowiednika JSON `{"t": 1710681600}`, zrzut Hex w CBOR będzie wyglądał mniej więcej tak:
`a1 61 74 1a 65 f6 d1 80`
(`a1` - mapa, `61` - string "t", `1a` - 32-bitowy integer, po którym następuje czas z epoki UNIX).

#### Zalety
- Standard Internetowy IETF: Niezależny od jednego dostawcy czy firmy, stabilny protokół do długoterminowych wdrożeń.
- Minimalistyczny dekoder: Narzut kodu (zajętość pamięci Flash/RAM) potrzebnego na mikrokontrolerze do obsługi CBOR jest na tyle mały, że nadaje się dla najmniejszych układów.
- Tagi semantyczne: Pozwala na natywne oznaczanie typów danych z wyższego poziomu (np. bezpośrednie oznaczenie bajtów jako "znacznik czasu daty i godziny" bez parsowania ze stringów).

#### Wady
- Brak strukturyzowanej kompresji kluczy: Podobnie jak JSON/MessagePack, powtarza nazwy kluczy w tablicach, jeśli nie zaprojektuje się struktury ręcznie (np. przesyłając samą tablicę wartości bez kluczy).
- Mniejsza popularność analityczna: Świetny do transportu z IoT, ale słabo wspierany w dużych systemach Big Data.

#### Źródła
- https://cbor.io/
- https://datatracker.ietf.org/doc/html/rfc8949

---

### 5.4 FlatBuffers

#### Opis
Rozwiązuje jeden z głównych problemów innych formatów: wymóg uprzedniego rozpakowania (deserializacji) całości danych do pamięci operacyjnej, aby móc cokolwiek z nich odczytać. Pozwala na bezpośredni dostęp do zserializowanych bajtów.

#### Przykład
Podobnie jak Protobuf, wymaga schematu, ale tym razem z rozszerzeniem `.fbs`.
```fbs
table DataPoint {
  timestamp: long;
  value: float;
}
root_type DataPoint;
```

#### Zalety
- Brak deserializacji (Zero-copy): Możesz odczytać konkretny punkt z szeregu czasowego bezpośrednio ze strumienia bajtów. Powoduje to drastyczny spadek zużycia pamięci RAM.
- Maksymalna wydajność odczytu: Znacznie szybszy czas dostępu do danych niż w przypadku Protobuf czy MessagePack.
- Ścisłe typowanie: Posiada schemat, co gwarantuje spójność danych strukturalnych.

#### Wady
- Większy rozmiar pliku binarnego: FlatBuffers wykorzystuje "padding" (wyrównywanie pamięci) oraz wskaźniki (offsets), aby umożliwić odczyt zero-copy. Sprawia to, że pliki wynikowe są większe niż w zwięzłym Protobuf.
- Trudniejsze w użyciu API: Pisanie kodu z użyciem FlatBuffers (tzw. budowanie bufora od dołu do góry) bywa mało intuicyjne dla programistów w porównaniu do zwykłego przypisywania wartości do obiektów.

#### Źródła
- https://flatbuffers.dev/
- https://github.com/google/flatbuffers

---