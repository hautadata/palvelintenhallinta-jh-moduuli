# H7 – Moduuli

## Johdanto

Tässä raportissa käydään läpi tekemääni Palvelinten Hallinta -kurssin miniprojektia. Sain alkuun sellaisen idean, että teen herra-orja-arkkitehtuurin, jossa orjakoneet hakisivat verkosta jotain jokaiselle erikseen määriteltyä dataa, tallentaisivat sitä, ja lopulta lähettäisivät koosteen näistä herrakoneelle.

Päätin alkaa keräämään säätietoja. Tavoitteena on orjakoneilla ajaa automaattisesti 30 minuutin välein komento, joka printtaa useamman kaupungin sen hetkisen sään, ja lisää mukaan vielä aikaleiman. Tiedot tallennetaan yhteen jokaiselle kaupungille erilliseen ”loki”-tiedostoon, joka päivittyy aina kun komento ajetaan automaattisesti uudelleen. Tämän jälkeen automatisoidaan myös se, että orjakoneet lähettävät säätietolokin herrakoneelle tietyn väliajoin.

## Ensin käsin…

Kuten kurssilla on jo varmasti opittu, lähdetään ensin kokeilemaan käsin, ja vasta sen jälkeen automatisoimaan. Tätä varten hyödynnän aiemmissakin tehtävissä käyttämääni, kurssin alussa luotua Debian 12-virtuaalikonetta VirtualBoxissa.

Luon aluksi uuden hakemiston kokeilua varten komennolla `$ mkdir weather` . Tämän jälkeen lähden kokeilemaan komentoa `$ { echo $(date); | curl -s wttr.in/helsinki; }`

![image](https://github.com/hautadata/palvelintenhallinta-jh-moduuli/assets/148875340/5192e102-4632-4b4b-81a7-d037fdbbdca9)
> Yllä: Komennon lopputulos.

---

Komennossa on kaksi osaa. Ensimmäinen osa eli echo $(date) printtaa nykyisen aikaleiman. Toinen osa eli curl -s wttr.in/helsinki käyttää curl (copy url)-syntaksia, jolla se hakee ja printtaa wttr.in -sivustolta haluamani kaupungin säätiedon. Kyseessä on GitHub-käyttäjä Chubinin luoma komentokonsoleille tarkoitettu sääpalvelu. En tiedä mistä palvelusta säätiedot haetaan varsinkin Suomesta, mutta luulen etteivät ne kuitenkaan ole täysin tarkkoja. Mutta se ei olekkaan tämän moduulin idea :) 

Muutetaan vielä vähän komentoa, lisätään curlin perään | head -n 7, joka printtaa vain ensimmäiset 7 riviä. Kuten yllä olevasta kuvasta näkyy, alempana näkyy hieman laajemmin säätietoja ja ne vievät jo enemmän tilaa näytöltä. Tuon "weather report" tekstin alla oleva pieni merkkitaide ja lämpötilat riittävät meille.

Tehdään seuraavaksi scripti, joka hakee Helsingin, Tampereen sekä Oulun säätiedot ja tallentaa ne yksittäisiin tiedostoihin. Avataan editori komennolla `$ sudo nano weather.sh`.

Scripti menee näin:

```
#!/bin/bash

log_dir="home/joonas/weather"

helsinki_log="$log_dir/helsinki.txt"

tampere_log="$log_dir/tampere.txt"

oulu_log="$log_dir/oulu.txt"

{ echo $(date); curl -s wttr.in/helsinki | head -n 7; } >> "$helsinki_log"

{ echo $(date); curl -s wttr.in/tampere | head -n 7; } >> "$tampere_log"

{ echo $(date); curl -s wttr.in/oulu | head -n 7; } >> "$oulu_log"
```

Scriptissä siis on määritelty haluamani hakemisto (log_dir), johon haluamani tiedostot (helsinki/tampere/oulu_log) tallentuvat, ja sisältävät ylempänä käydyn komennon jokaisesta kaupungista erikseen.

Testataan lopputulosta. Scriptilele ajo-oikeus komennolla `$ chmod +x weather.sh`. Sitten `$ ./weather.sh`.

Lopputulos näyttää hyvältä. Haluamaani hakemistoon on tallentunut 3 uutta tiedostoa, ja cat-komennolla voimme tarkistaa niiden sisällön. Näyttää juuri oikealta!

![image](https://github.com/hautadata/palvelintenhallinta-jh-moduuli/assets/148875340/58cc8fc1-1f2b-4fba-a04f-f48adf90edb3)
>Yllä: Scripti toimii.

---

Sitten halutaan vielä automatiikkaa. Haluan, että scripti ajetaan automaattisesti 30 minuutin välein. Tätä varten on crontab-komento. Se on Unix-komento, joka luo listan komennoista, jotka käyttöjärjestelmä ajaa automaattisesti määritellyn aikataulun mukaisesti. (Awati, 2023)

Avaan crontab-tiedoston komennolla `$ crontab -e`. Lisään sinne syntaksin `*/30 * * * * /home/joonas/weather/weather.sh` . Tähdet kertovat sen, kuinka monesti komentoa ajetaan. Ensimmäinen tähti on minuutit, seuraava tunnit, kuukauden päivät, kuukausi, ja viimeinen vielä viikon päivät. Loppuosa on itse ajettava komento. Haluan että komento ajetaan aina puolen tunnin välein, joten vaihdan minuuttien kohdalla 30.

![image](https://github.com/hautadata/palvelintenhallinta-jh-moduuli/assets/148875340/5bf6df41-ef0b-428c-9820-7e7e16bdd124)
>Yllä: Crontab-tiedoston muokkausta.

---

Sitten odotellaan hetki jotta näemme toimiiko palvelu ollenkaan. Tunti eteenpäin, ja voimme tarkistaa tilanteen katsomalla lokitiedostoamme. `$ cat helsinki.txt` , ja näemme että sinne on tallentunut 2 uutta printtiä. Aikaleimasta näemme, että muutokset ovat tapahtuneet klo 4 ja 4:30 aikaan (iltapäivällä).

![image](https://github.com/hautadata/palvelintenhallinta-jh-moduuli/assets/148875340/4a1678d7-2f68-48d1-9a00-da025a6a42b8)
>Yllä: Automaattisesti ajetut lisäykset.

Sitten automatisoinnin kimppuun?

## ... Vasta sitten automaattisesti







## Lähteet

Awati, R. Techtarget. 2/2023. What is crontab?. Luettavissa: https://www.techtarget.com/searchdatacenter/definition/crontab. Luettu: 9.12.2023.




