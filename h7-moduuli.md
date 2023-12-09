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

Okei... Tähän kohtaan sellainen juonenkäännös että lopputulos ei ollut aivan toivottu :D joten käydään asia suht nopeasti tässä läpi kelattuna, koska vaihdoin tuon säädatan pois kokonaan.

Katsoin omaa h5 - CSI Kerava raporttia, sillä tulin tekemään uuden komennon Saltilla samalla tavalla kuin tuossa tehtävässä. Eli 

Tässä kohtaa Vagrant päälle Windowsin terminalissa komennolla `$ vagrant up`. Sen jälkeen yhteys tmasteriin komennolla `$ vagrant ssh tmaster` . Loin uuden kansion /srv/salt/ hakemistoon komennolla `$ mkdir /srv/salt/weather`.

Loin tänne uuden scriptin, jonka oli tarkoitus tallentaa Helsingin säädataa. Sisältö oli seuraavanlainen:

```
#!/bin/bash

log_dir="/usr/local/weather/finland"

helsinki_log="$log_dir/helsinki.txt"

{ echo $(date); curl -s wttr.in/helsinki | head -n 7; } >> "$helsinki_log"
```

Eli samaa kuin käsin testailussakin. Curlilla haetaan Helsingin säätiedot, ja tallennetaan ne helsinki.txt -nimiseen tiedostoon osoitteessa /usr/local/weather. Loin vielä uuden, yllä mainutun hakemiston orjalle komennolla `$ sudo salt 't001' cmd.run "mkdir /usr/local/weather/finland"`.

![image](https://github.com/hautadata/palvelintenhallinta-jh-moduuli/assets/148875340/c9687b41-a287-4f06-a011-27dbd441f0b2)
>Ylä: Scripti.

---

Sitten tarvittiin vielä init.sls -tiedosto, joka kertoo orjalle että missä komento sijaitsee. Se oli tämän tyylinen:

```

/usr/local/bin/weatherfi:
  file.managed:
    - source: salt://weather/finland
    - mode: "0755"
```

![image](https://github.com/hautadata/palvelintenhallinta-jh-moduuli/assets/148875340/2b3bf16d-415f-4da9-be58-ec3d0d19079d)
>Yllä: init.sls

---

Lähdin sitten ajamaan state.applyta orjalle komennolla `$sudo salt 't001' state.apply weather`. Tässä ei ongelmaa:

![image](https://github.com/hautadata/palvelintenhallinta-jh-moduuli/assets/148875340/52d59a44-de46-4fab-91ca-d576f5a7e33e)
>Yllä: Onnistunut state.apply

---

Katsoin komennolla `$ sudo salt 't001' cmd.run "ls /usr/local/bin"` , ja sieltä löytyi weatherfi -niminen scripti. Voimme siis ajaa komennon tällä nimellä. Ajoinkin seuraavaksi komennon `$ sudo salt 't001' cmd.run "weatherfi"` . Summary näytti jälleen hyvältä, mutta tässä kohtaa menin katsomaan lopputulosta itse luotuun tiedostoon ajamalla komennon `$ sudo salt 't001' cmd.run "cat /usr/local/weather/finland/helsinki.txt`.

No perhana, ei se näytäkään ihan yhtä nätiltä miltä se näytti aiemmin eri koneella testatessa... :D 

![image](https://github.com/hautadata/palvelintenhallinta-jh-moduuli/assets/148875340/fed773c9-bd1a-44e5-8e3f-3419107170e7)
>Yllä: Vähän erinäköistä kuin alussa testatessa...


Ajattelin että ei tuota oikein jaksa katsella, joten päätin yrittää toista dataa. Haetaan tällä kertaa METAR-tietoja, sillä niitä olen itse asiassa aiemmin yrittänyt hakea Linuxin komentoriviltä ja onnistunut siinä.

Eli tässä kohtaa unohdettiin tämä. Veikkaan että tuo näyttää sen takia erilaiselta, kun käytän Vagrantia Windowsilla. Windowsin komentorivi ja Linux eivät varmaan ihan samoissa mittasuhteissa ole, ja siksi tuo teksti "wrappaantyy" eri tavalla tässä. Mutta ei se mitään.

## Homma uusiksi, METAR






## Lähteet

Awati, R. Techtarget. 2/2023. What is crontab?. Luettavissa: https://www.techtarget.com/searchdatacenter/definition/crontab. Luettu: 9.12.2023.




