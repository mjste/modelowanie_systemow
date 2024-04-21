## Etap 1

- analiza problemu
  - przeglad wsystkiego co nie jestt hiputs
    - sumo
    - matsim
    - missim
    - smarts
  - jak sie realizuje modelowanie komunikacji miejskiej
    - jak to jest zapisywane w osm
    - jak te dane wydobyc, czy to trudne
    - jak rozwiazac kwestie rozkladow jazdy - nice to have w miare realnie modelowac
  - 2 do 3 artykulow, moze byc analiza konkretnych rozwiazan sumo, matsim, smarts itd
- wizualizacja webowa ostatnio zmergowana, kontakt - jakaś Natalia
  - bedzie super uzyteczna do realizacji problemu
- model
  - wjazd do zatoczki ??
    - znikanie a potem wciskanie w ruch
    - aktualnie multilane nie jestt stabilny, nie zmergowany. Póki co analizujemy model jednopasmowy, ale może będzie trzeba to zmieniać w środku semestru
    - zalezy nam na realistycznym efekcie, nie wizualizacji
  - buspas
    - czy w osm jest?
    - jak to modelowac??
    - pierszenstwo busa - moze sie wryc
    - wazniejsze niz tramwaje
- ogólnie mozna sie zdziwic jak proste albo jak skomplikowane sa rozwiaznia
- tramwaje maja mniejszy priorytet

## Etap 2

- OSM
  - jak parsowane sa w hiputs?
  - na pewno sa wyrzucane dane o przystankach
  - buspasyyy
- uruchomic wizualizacje (apkę web)
- zbadanie danych dotyczacych rozkladow jazdy
- SPROBUJMY WYSWIETLIC PRZYSTANKI NA WIZUALIZACJI



## Etap 3
koniecznie przegląd papierów. Trzeba zrobić działające parsowanie OSMa do 

Zaprojektować
import danych
rozszerzenia modelu
przystanki, buspas, może ścieżki rowerowe.

parametr lane'a - co może wjechać na dany lane

może pojawić się multilane, trzeba to uwzględnić.


Zacząć od poczytania, nie stricte wymyślania. Najlepiej znaleźć kawałek tekstu, gdzie to wprost opisane

Musimy zaplanować co trzeba zrobić:
* parametry 


Duży branch multilane. Są światła, skrzyżowania multilane