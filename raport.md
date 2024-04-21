# Etap 1

## Opis danych w OSM

Dane dotyczące przystanków autobusowych zapisane są w plikach `*.osm` w postaci znaczników `node` z tagami: `<tag k="bus" v="yes"/>`, `<tag k="public_transport" v="platform"/>` oraz `<tag k="highway" v="bus_stop"/>`. Przykładowy fragment pliku `*.osm`:

```xml
<node id="1701717675" visible="true" version="11" changeset="108577455" timestamp="2021-07-25T19:02:54Z" user="duran_defrost" uid="13478299" lat="51.1178972" lon="17.0310065">
  <tag k="bench" v="yes"/>
  <tag k="bin" v="yes"/>
  <tag k="bus" v="yes"/>
  <tag k="departures_board" v="realtime"/>
  <tag k="highway" v="bus_stop"/>
  <tag k="name" v="Pomorska"/>
  <tag k="network" v="Wrocław"/>
  <tag k="operator" v="ZDiUM Wrocław"/>
  <tag k="public_transport" v="platform"/>
  <tag k="ref" v="10655"/>
  <tag k="shelter" v="yes"/>
  <tag k="tactile_paving" v="no"/>
  <tag k="tram" v="yes"/>
 </node>
```

oraz

```xml
 <node id="302864607" visible="true" version="11" changeset="77276819" timestamp="2019-11-19T13:42:00Z" user="maro21" uid="3476229" lat="51.1136078" lon="17.0322370">
  <tag k="name" v="Uniwersytet Wrocławski"/>
  <tag k="old_name" v="Uniwersytet"/>
  <tag k="public_transport" v="stop_position"/>
  <tag k="railway" v="tram_stop"/>
  <tag k="ref" v="10015"/>
  <tag k="tram" v="yes"/>
 </node>
```

a w przypadku nieużywanych

```xml
 <node id="304085780" visible="true" version="10" changeset="81903508" timestamp="2020-03-07T16:24:48Z" user="Paweł Bogubowicz" uid="7708037" lat="51.1172015" lon="17.0317912">
  <tag k="disused:public_transport" v="stop_position"/>
  <tag k="disused:railway" v="tram_stop"/>
  <tag k="name" v="Pomorska"/>
  <tag k="note" v="Przystanek nieczynny ze względu remont mostów Pomorskich"/>
  <tag k="ref" v="10602"/>
  <tag k="tram" v="yes"/>
 </node>
```

Opis w dokumentacji OSM informuje nas, że

### Buses and trams

Bus stops along the route should be tagged with `highway=bus_stop`
or using the newer `public_transport=platform`,
which should be positioned to the side of the carriageway where passengers wait.

A tram running within the main carriageway should be tagged with `railway=tram` on the same way as the road and the road itself should be tagged with `embedded_rails=tram`.
If the tram runs into a separate right-of-way to the side of the carriageway or within the central reservation then create a separate way also tagged using `railway=tram`.

Jeżeli chodzi o informacje o buspasach i torach to czytając informacje zawarte na stronach: <https://wiki.openstreetmap.org/wiki/Tag:railway%3Dtram> oraz <https://wiki.openstreetmap.org/wiki/Bus_lanes> wiemy, że buspasy są oznaczane tagiem np.`busway=lane` oraz `lanes:bus=1` lub `bus:lanes=yes|yes|designated` a tory tramwajowe tagiem `railway=tram`.
Jednakże dane te nie są dostępne w plikach `*.osm` z którymi pracujemy.
W plikach z Krakowa znaleziono jedynie informacje o przejsciach dla pieszych oznaczone tagami `railway=tram_crossing`

Poprawne wczytanie tych informacji będzie wymagało edycji sposobu wczytania danych z plików w pliku `PatchesGraphReaderWriterImpl.java` oraz modyfikacje plików `WayData.java`, `JunctionData.java` lub dodanie własnej klasy obsługującej przystanki.
Informacja o przystankach jest zawarta w `edges.csv` w polu tagi.

## Analiza implementacji w SUMO

* Tory tramwajowe są normalnymi krawędziami, po których umożliwiono ruch pojazdów typu tramwaj. Buspasy można zrealizować w podobny sposób.
* Autobusy aby się zatrzymać muszą mieć przystanek. Musi on być postawiony na konkretnym pasie. Pojazdy mogą zmieniać pasy z różnych powodów: żeby skręcić, żeby ominąć przeszkodę, żeby zatrzymać się na przystanku
* Przystanki mają dodatkowo pojemność, która zawiera liczbę pasażerów, którzy czekają na autobus.Może ona wpłynąć na "zakorkowanie" chodnika.
* Jest możliwość importowania danych z OSM
* Rozkład jazdy działa w taki sposób, że w dodatkowym pliku definiujemy listy przystanków, godziny odjazdów oraz minimalny czas postoju na przystanku:

```xml
    <vType id="bus" vClass="bus"/>
    <vType id="tram" vClass="tram"/>

    <route id="busRoute" edges="-E1 -E0 -E3 -E2" color="yellow" repeat="10" cycleTime="140">
        <stop busStop="A_bus" duration="20.00" until="30.00"/>
        <stop busStop="B_bus" duration="20.00" until="90.00"/>
    </route>

    <vehicle id="bus" type="bus" depart="0.00" line="42" route="busRoute"/>

    <flow id="tram1" type="tram" begin="0.00" end="3600.00" period="300.00" line="23">
        <route edges="-E4" color="cyan"/>
        <stop busStop="A_tram1" duration="20.00" until="30.00"/>
        <stop busStop="B_tram1" duration="20.00" until="45.00"/>
    </flow>
```

## Analiza implementacji w MATSim

* Ścieżki między przystankami są tworzone na podstawie danych z OSM. Algorytm uwzględnia różne przystanki dla różnych kierunków jazdy.
* Korzysta z danych GTFS, HAFAS, OSM.
* Przystanki są podzielone na parent stop i stop location. Parent stop to ogólnie miejsce, gdzie zatrzymują się pojazdy, a stop location to konkretne miejsce na pasie ruchu. Przykładowo mamy Plac Inwalidów jako parent stop, a przystanek Plac Inwalidów 1 i Plac Inwalidów 2 jako stop location.
Mapping: <https://ethz.ch/content/dam/ethz/special-interest/baug/ivt/ivt-dam/publications/students/501-600/sa530.pdf>
* Public transport: <https://www.mos.ed.tum.de/fileadmin/w00ccp/tb/theses/Mehlsta__ubler_2019.pdf>
* Trasy są tworzone na podstawie sekwencji przystanków. Następnie kolejne pary przystanków są łączone w trasy za pomocą wariacji algorytmu Dijkstry.

## Analiza implementacji w SMARTS

Inicjalizacja pojazdów komunikacji miejskiej następuje w pliku `TrafficNetwork.java`. Znajdujemy tam informację, że cel autobusów i tramwajów jest wybierany losowo.

```java
void createOneInternalPublicVehicle(final VehicleType type,
   final double timeNow) {
  VehicleType transport = null;
  String randomRef = null;
  ArrayList<Edge> startEdgesOfRandomRoute = null;
  ArrayList<Edge> endEdgesOfRandomRoute = null;
  if ((type == VehicleType.TRAM)
    && (internalTramRefInSdWindow.size() > 0)) {
   transport = VehicleType.TRAM;
   randomRef = internalTramRefInSdWindow.get(random
     .nextInt(internalTramRefInSdWindow.size()));
   startEdgesOfRandomRoute = internalTramStartEdgesInSourceWindow
     .get(randomRef);
   endEdgesOfRandomRoute = internalTramEndEdgesInDestinationWindow
     .get(randomRef);
  } else if ((type == VehicleType.BUS)
    && (internalBusRefInSourceDestinationWindow.size() > 0)) {
   transport = VehicleType.BUS;
   randomRef = internalBusRefInSourceDestinationWindow.get(random
     .nextInt(internalBusRefInSourceDestinationWindow.size()));
   startEdgesOfRandomRoute = internalBusStartEdgesInSourceWindow
     .get(randomRef);
   endEdgesOfRandomRoute = internalBusEndEdgesInDestinationWindow
     .get(randomRef);
  }

  if ((startEdgesOfRandomRoute == null)
    || (endEdgesOfRandomRoute == null)) {
   return;
  }

  ArrayList<RouteLeg> route = ReferenceBasedSearch.createRoute(transport,
    randomRef, startEdgesOfRandomRoute, endEdgesOfRandomRoute);
  for (int numTry = 0; numTry < 10; numTry++) {
   if (route == null) {
    route = ReferenceBasedSearch.createRoute(transport, randomRef,
      startEdgesOfRandomRoute, endEdgesOfRandomRoute);
   } else {
    break;
   }
  }
  if (route != null) {
   addNewVehicle(type, false, false, route, internalVehiclePrefix,
     timeNow, "", getRandomDriverProfile());
  }
 }

```

Implementacja przystanku tramwajowego odbywa się w pliku `VehicleUtils.java`, gdzie w przypadku zatrzymania tramwaju na drodze, wszystkie inne pojazdy są hamowane.

```java
/**
  * Find impeding object related to tram stops. A tram stop is located at the
  * end of an edge. The edge has a count-down timer, which is triggered when
  * a tram approaches the tram stop. If the count-down is in process, the
  * tram cannot pass the tram stop.
  * 
  * Tram stop also affects other vehicles that move in parallel lanes/edges
  * besides tram track. Note that other vehicles must stop behind tram, as
  * required by road rule.
  * 
  * In OpenStreetMap data, tram tracks are separated from other roads. Hence
  * there are many roads parallel to tram tracks. The tram edges that are
  * parallel to an edge are identified during pre-processing.
  */
 void updateImpedingObject_Tram(final Vehicle vehicle, final double examinedDist, final Edge targetEdge,
   final Vehicle slowdownObj) {

  if ((vehicle.type == VehicleType.TRAM) && (targetEdge.timeTramStopping > 0)) {
   slowdownObj.speed = 0;
   slowdownObj.headPosition = (examinedDist + targetEdge.length + vehicle.driverProfile.IDM_s0) - 0.00001;
   slowdownObj.type = VehicleType.VIRTUAL_STATIC;
   slowdownObj.length = 0;
   return;
  }

  if ((vehicle.type != VehicleType.TRAM) && Settings.isAllowTramRule) {

   if (targetEdge.timeTramStopping > 0) {
    slowdownObj.speed = 0;
    slowdownObj.headPosition = (((examinedDist + targetEdge.length) - VehicleType.TRAM.length)
      + vehicle.driverProfile.IDM_s0) - 0.00001;
    slowdownObj.type = VehicleType.VIRTUAL_STATIC;
    slowdownObj.length = 0;
    return;
   }

   else {
    final Edge parallelTramEdgeWithTramStop = targetEdge.parallelTramEdgeWithTramStop;
    if ((parallelTramEdgeWithTramStop != null) && (parallelTramEdgeWithTramStop.timeTramStopping > 0)) {
     slowdownObj.speed = 0;
     slowdownObj.headPosition = (((examinedDist + targetEdge.distToTramStop) - VehicleType.TRAM.length)
       + vehicle.driverProfile.IDM_s0) - 0.00001;
     slowdownObj.type = VehicleType.VIRTUAL_STATIC;
     slowdownObj.length = 0;
     return;
    }
   }
  }

 }
```

Istotna informacja znajduje się w komentarzu opisującym powyższą funkcję, ponieważ jest to sposób modelowania przystanku, który można zastosować również w przypadku przystanku autobusowego.
Postoje autobusu różnią się jednak tym, że czasami autobusy zatrzymują się na drodze, a czasami zjeżdżają na przystanek. W przypadku zatrzymania na drodze, wszystkie inne pojazdy są hamowane, a w przypadku zjechania na przystanek, inne pojazdy mogą kontynuować jazdę i muszą zatrzymać się dopiero przy wyjeżdżaniu autobusu z zatoczki.
Na podstawie powyższego kodu można zaproponować rozwiązanie, gdzie w zależności od konfiguracji przystanku pozostałe pojazdy zatrzymują się na cały czas trwania postoju autobusu lub są hamowane tylko raz pod koniec czasu postoju.

* analiza problemu
  * przeglad wsystkiego co nie jestt hiputs
    * sumo
    * matsim
    * missim
    * smarts
  * jak sie realizuje modelowanie komunikacji miejskiej
    * jak to jest zapisywane w osm
    * jak te dane wydobyc, czy to trudne
    * jak rozwiazac kwestie rozkladow jazdy - nice to have w miare realnie modelowac
  * 2 do 3 artykulow, moze byc analiza konkretnych rozwiazan sumo, matsim, smarts itd
* wizualizacja webowa ostatnio zmergowana, kontakt - jakaś Natalia
  * bedzie super uzyteczna do realizacji problemu
* model
  * wjazd do zatoczki ??
    * znikanie a potem wciskanie w ruch
    * aktualnie multilane nie jestt stabilny, nie zmergowany. Póki co analizujemy model jednopasmowy, ale może będzie trzeba to zmieniać w środku semestru
    * zalezy nam na realistycznym efekcie, nie wizualizacji
  * buspas
    * czy w osm jest?
    * jak to modelowac??
    * pierszenstwo busa - moze sie wryc
    * wazniejsze niz tramwaje
* ogólnie mozna sie zdziwic jak proste albo jak skomplikowane sa rozwiaznia
* tramwaje maja mniejszy priorytet

## Etap 2

* OSM
  * jak parsowane sa w hiputs?
  * na pewno sa wyrzucane dane o przystankach
  * buspasyyy
* uruchomic wizualizacje (apkę web)
* zbadanie danych dotyczacych rozkladow jazdy
* SPROBUJMY WYSWIETLIC PRZYSTANKI NA WIZUALIZACJI

# ...i wszystko to zapisać w jakimś dokumencie na GitHubie przed 08.04
