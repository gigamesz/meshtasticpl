# Budowa noda Meshtastic - teoria

> Nie ma nic bardziej praktycznego niż dobra teoria.<br>
> &mdash; <cite>Ludwig Boltzmann</cite>

Ten artykuł ma na celu wyjaśnić wszystkie aspekty budowy węzła DIY.
Nie jest to gotowa instrukcja/poradnik jak zbudować node, ale raczej źródło wiedzy dla osób, które chciałyby stworzyć swój własny projekt lub chciałyby zrozumieć z jakich elementów składają się gotowe rozwiązania i czym rożnią się między sobą/na jakie parametry należy zwracać uwagę.
Jeśli szukasz gotowego projektu/poradnika/instrukcji/listy części do zamówienia, poszukaj w innych działach.

## Dlaczego DIY?

Zalety:
- niższa cena
- dostosowanie konstrukcji do własnych potrzeb (np. kształt, miejsce montażu złącza antenowego, diody LED lub ich brak, dodatkowe przyciski itp.)
- możliwość użycia komponentów w kombinacji niedostępnej w żadnej gotowej konstrukcji
- poprawienie błędów producenta (np. użycie bardziej wydajnego układu ładującego)
- możliwość osiągnięcia parametrów niedostępnych w gotowcach (np. większa moc nadawania)

Potencjalne wady:
- zazwyczaj większy rozmiar płytki (w warunkach nieprzemysłowych ciężko o miniaturyzację na poziomie masowo produkowanych płytek)
- mniejsza jakość/odporność mechaniczna (własnoręczne lutowanie, błędy montażu itp.)
- dłuższy czas produkcji w porównaniu do zamówienia gotowca
- potrzebny sprzęt
- potrzebna wiedza + umiejętności (np. lutowanie)
- brak gotowych akcesoriów typu obudowa (sklep lub szablony do druku 3D)

Gotowe płytki też mają problemy, np. [tutaj](https://www.youtube.com/watch?v=FcQzAxWBN7A) w teście Heltec T114 opisano mało wydajny układ ładowania z panelu słonecznego oraz zawieszanie się po rozładowaniu baterii bez wbudowanego układu ochrony przed głębokim rozładowaniem.

## Podstawy projektu

Projektując node Meshtastic musimy zdecydować jakich komponentów użyjemy w każdej kategorii:

Obowiązkowe:
- radio LoRa
- mikrokontroler
- antena

Opcjonalne:
- akumulator
- układ zasilania z akumulatora
- układ ładowania akumulatora
- panel fotowoltaiczny
- moduły "peryferyjne" typu GPS, czujniki, brzęczyk, diody LED, wyświetlacz, przyciski (klawiatura) itp.

Przy czym "jedynym sensownym" nodem jest taki, który jest autonomiczny, to znaczy działa na energii pobranej z ogniwa słonecznego, a na wypadek braku światła słonecznego posiada baterię, która jest w stanie podtrzymać działanie noda aż do powrotu zasilania z panelu.
Czyli pojemność baterii powinna być na tyle duża, żeby (uwzględniając pobór prądu przez wszystkie komponenty urzadzenia) wystarczyć na najdłużej trwające "ciemne" okresy na naszej szerokości geograficznej (przewidując również czynniki losowe takie jak zasłonięcie panelu przez pokrywę śnieżną aż do powrotu dodatniej temperatury, w której śnieg się roztopi).
Zatem panel solarny i bateria powinny być traktowane jako komponenty obowiązkowe w praktycznie każdym wypadku poza konstrukcjami testowymi/prototypowymi.
Nawet w wypadku węzła mobilnego powinna być przewidziana możliwość ładowania (przenośnym) panelem słonecznym - czy to podłączając go bezpośrednio do jakiegoś gniazda w obudowie czy też ładując (wyciągalne) baterie w jakiegoś rodzaju osobnej ładowarce słonecznej.

## Radio (modem) LoRa

Jedynym producentem układów radiowych LoRa (wyłączność licencji) jest firma Semtech, zatem wybór jest tutaj ograniczony.

### Częstotliwość

W Polsce (i Unii Europejskiej) są do wyboru warianty 433 i 868MHz.
Wersja 868MHz jest uznawana za lepiej dostosowaną do środowisk miejskich, dlatego na obszarach mocniej zabudowanych większość osób używa właśnie takich węzłów. 433 może mieć lepszy zasięg na terenach otwartych.

### Wsparcie Meshtastic dla układów LoRa

Meshtastic do obsługi układów LoRa używa biblioteki [RadioLib](https://jgromes.github.io/RadioLib/). Można założyć, że wszystkie układy na liście kompatybilności RadioLib są lub w niedługim czasie będą obsługiwane przez firmware Meshtastic.

Należy jednak wziąć pod uwagę pewne problemy, które w przeszłości uniemożliwiały prawidłowe działanie niektorych układów mimo wsparcia przez RadioLib.
Dobrym przykładem jest układ LR1110, który jest co prawda wspierany przez RadioLib i Meshtastic, ale w związku z pewnym bugiem [nie jest w stanie odbierać pakietów z węzłów opartych o układy SX127x]. 
Problemem jest niestandardowy identyfikator protokołu Meshtastic. Producent (Semtech) testował swoje urządzenia ze standardowym numerem protokołu i wszystkie testy przebiegły pomyślnie, niestety gdy Meshtastic próbuje użyć swojego identyfikatora, trafiamy na błąd i transmisja nie jest możliwa. Dlatego na stronie Meshtastic przy urządzeniach z LR1110 jest (stan na listopad 2024 r.) informacja o problemach:

```
Currently, LR1110 radios are unable to receive Meshtastic packets from the older SX127x radios. Semtech is aware of this issue, and we're hopeful they'll provide a solution soon.
```
Przed użyciem 

### Płytki LoRa

Ponieważ układy Semtecha są raczej niemożliwe do zamontowania metodami domowymi, zapewne będziesz chciał użyć gotowej płytki, gdzie chip z radiem jest już zainstalowany, a sama płytka posiada wyprowadzenia do lutowania, gniazdo antenowe itp. Taka płytka nazywa się "development board" albo "devkit".

Przykłady dostepnych producentów/marek: EBYTE, AI-Thinker, Seeed Studio, G-NiceRF.

EBYTE E22-900M* to SX1262 (900M22S to podstawowy 22dBi, 900M30S to ten ze wzmacniaczem 30dBi)

https://github.com/NanoVHF/Meshtastic-DIY/blob/main/Docs/LoRa%20modules%20V2%20en.pdf
https://www.nicerf.com/sx1262/

TCXO?

## Mikrokontroler

Obecnie w Meshtasticu liczą się dwie platformy MCU (mikrokontrolerów): ESP32 i nRF52.
Inne występujące to ARM, RP2040 (RaspberryPi), STM32.
Ze względu na drastycznie większą wydajność energetyczną nRF52 jest polecany do wszystkich zastosowań (przypominam, że artykuł zakłada budowę węzła zasilanego energią słoneczną).
Urządzenia z nRF52 pobierają nawet [6 razy mniej prądu](https://www.youtube.com/watch?v=EEXxiD1CSDM) w bezczynności niż te z ESP32.
Zaletami ESP32 są niższa cena i obsługa WiFi, dzięki czemu możemy uzyskać łączność IP (zdalne zarządzanie przez internet, integracja z serwerem MQTT). 

## Płytki z MCU

Rekomendowane: nRF supermini z https://github.com/joric/nrfmicro/wiki

## Płytki combo LoRa + MCU

Takie płytki 2w1 pozwalają zmniejszyć rozmiar całkowity węzła i upraszczają jego budowę, są jednak droższe niż płytki kupione osobno i własnoręcznie połączone.

Przykładowe produkty:

RAK 4630: nRF52 + SX1262 https://store.rakwireless.com/products/rak4630-nrf52840-sx1262-lora-bluetooth-module-for-lorawan?variant=43557762302150

Hetec HT-CT62: ESP32 + SX1262 https://heltec.org/project/ht-ct62/

AI-Thinker RA-XX: 

## Akumulator

12V lub 18650 (3.6V)

Odporność na temperaturę - możliwość ładowania na mrozie, brak samoczynnego rozładowania na mrozie/w upale.


1S Li-Ion and Li-Pol (standard, widely available batteries commonly used by Meshtastic enthusiasts),
1S LiFePo4 (popular in solar installations due to their long lifespan and capacity over 100 Ah), or
1S and 2S Lithium Titanate Oxide (LTO) (batteries that can be discharged and charged even in winter temperatures below −10 °C).

Oznaczenie *S określa ile ogniw jest połączonych szeregowo (ang. "serial", mnoży napięcie) a *P ile ogniw jest połączonych równolegle (ang. "parallel", napięcie jest stałe).

## Układ zasilania z akumulatora

Dostosowanie napięcia ładowania na styku akumulator-elektronika.
Zabezpieczenie przed głębokim rozładowaniem (ang. undervoltage protection) - powinien odciąć zasilanie gdy napięcie akumulatora spadnie poniżej poziomu krytycznego.

TPS63000 - regulator napięcia ustala 3.3V (np. dla akumulatora 3.6V podwyższa napięcie gdy akumulator jest rozładowany i obniża gdy jest naładowany).

TPS61023 bost converter

## Układ ładowania akumulatora

LiPo: MCP73831, gotowy układ: https://www.adafruit.com/product/2124
12V: 


CN3791, CN3795 firmy [Consonance](http://www.consonance-elec.com/en/products_32/).

## Panel słoneczny


# Linki

https://meshtastic.discourse.group/t/new-1w-diy-variant-xiao-nrf52840-ebyte-e22-900m30s/7904/8
https://github.com/ndoo/ikoka-nano-meshtastic-device



## (brudnopis) Parametry połączeniowe

Modem LoRa <-> MCU: interfejs SPI albo UART, wybrać SPI

Akumulator <-> urządzenie: napięcie, zabezpieczenie przed rozładowaniem

Panel słoneczny <-> akumulator: napięcie, prad ładowania, zabezpieczenie przed przeładowaniem