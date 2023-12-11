# H7 – Moduuli

## Johdanto

Tässä raportissa käydään läpi tekemääni Palvelinten Hallinta -kurssin miniprojektia. Sain alkuun sellaisen idean, että teen herra-orja-arkkitehtuurin, jossa orjakoneet hakisivat verkosta jotain jokaiselle erikseen määriteltyä dataa, tallentaisivat sitä, ja lopulta lähettäisivät koosteen näistä herrakoneelle.

Tein tehtävää itsenäisesti kotonani lauantaina 9.12. Viimeistelin itse raportin maanantaina 11.12. Tein tehtävät HP pavilion kannettavallani, jossa on Intel i5-9300H prosessori, 16gt RAM, 512gt SSD ja Windows 10 home-käyttöjärjestelmä. Tehtävissä on käytetty apuna Tero Karvisen materiaaleja ja linkkejä sivulta _"Infra as code"_. (Karvinen, 2023)

Päätin alkaa keräämään säätietoja. Tavoitteena on orjakoneilla ajaa automaattisesti 30 minuutin välein komento, joka printtaa useamman kaupungin sen hetkisen sään, ja lisää mukaan vielä aikaleiman. Tiedot tallennetaan yhteen jokaiselle kaupungille erilliseen ”loki”-tiedostoon, joka päivittyy aina kun komento ajetaan automaattisesti uudelleen. Tämän jälkeen automatisoidaan myös se, että orjakoneet lähettävät säätietolokin herrakoneelle tietyn väliajoin.

Tai näin ainakin luulin aluksi alkavani tekemään... Matkan varrella huomasin että Saltissa printti ei ihan vastannut sitä, mitä se oli testatessa.

Lopulta vaihdoin hieman ideaa, ja suunnitelma vaihtuikin siihen, että 2 orjakonetta printtaavat minulle ilmailussa käytettäviä METAR-säätiedotteita. Tästä lisää alempana!

Tätä moduulia varten loin erillisen repositoryn. Kurssin muut tehtävät löytyvät toisesta repostani, osoitteesta https://github.com/hautadata/palvelintenhallinta-jh

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

Ajattelin että ei tuota oikein jaksa katsella, joten turha yrittää tuota edes puskea herralle, vaan päätin yrittää toista dataa. Haetaan tällä kertaa METAR-tietoja, sillä niitä olen itse asiassa aiemmin yrittänyt hakea Linuxin komentoriviltä ja onnistunut siinä.

Eli tässä kohtaa unohdettiin tämä. Veikkaan että tuo näyttää sen takia erilaiselta, kun käytän Vagrantia Windowsilla. Windowsin komentorivi ja Linux eivät varmaan ihan samoissa mittasuhteissa ole, ja siksi tuo teksti "wrappaantyy" eri tavalla tässä. Mutta ei se mitään.

## Homma uusiksi, METAR -johdanto

Ilmailusta kiinnostuneena sain tämän jälkeen idean kerätä lentoasemilla raportoitavia METAR-säähavaintotiedotteita. Niitä käytetään kansainvälisessä siviili- ja liikenneilmailussa ilmoittamaan ajankohtaista säädiagnostiikkaa, joiden perusteella mm. tehdään lentosuunnitelmia ja tiedotetaan lentäjiä lähtö- sekä kohdekenttien sääoloista.

Linuxin komentoriviltä saa suoraan ladattua METAR-palvelun komennolla `$ sudo apt install metar`. Tämän jälkeen voit tarkistaa minkä tahansa kaupungin METAR-tiedotteen syöttämällä metar -d sekä lentoaseman ICAO-koodin. (Kansainvälisen siviili-ilmailujärjestön jokaiselle maailman lentoasemalle myöntämä 4-kirjaiminen tunnistekoodi)

Alhaalla kuvassa näkyy Debian 12-komentorivillä ajettu komento `$ metar -d EFHK`, joka kertoo Helsinki-Vantaan lentoaseman viimeisimmän METAR-tiedotteen. 

![image](https://github.com/hautadata/palvelintenhallinta-jh-moduuli/assets/148875340/6293e9fa-c1ea-49e5-b373-e8e89264da38)
>Yllä: METAR-tiedote

Punaisella alleviivattu pätkä on standardoitu kirjoitusmuoto, jota mm. lentäjät lukevat. Näyttää melkoiselta numero- ja kirjainhelvetiltä, mutta tuosta pystyy lukemaan mm. tuulen ja tuulenpuuskien suunnan sekä nopeuden, näkyvyyden, lämpötilan sekä ilmanpaineen. Ensin vain pitää opetella lukemaan tuota koodinpätkää.

Lisäämällä komennon keskelle -d (decode metar), saamme printtiin puretun ja kieltämättä helpommin luettavan koosteen säätiedoista.

## METAR Saltissa

Aiemmin yritettiin jo Vagrantilla ja Saltilla saada tiedostot ja herralta orjalle -komennot toimimaan, joten ei tässä muuta kuin uusiksi, mutta eri tavalla.

Lähdetään tekemään vaikka niin, että orja numero 1 tallentaa jatkuvasti Suomen lentokenttien METAR-tietoja, ja pistetään toinen orja keräämään tietoa Euroopan suurimmilta kentiltä.

METAR on siinä mielessä helpompi, koska sen tosiaan saa suoraan komentoriviltä. Ei tarvita curleja.

Lähdin luomaan tätä varten uuden kansion tmasterilla /srv/saltiin komennolla `$ mkdir /srv/salt/metar`. Tänne on tarkoitus tulla scripti, sekä init.sls-tiedosto. Loin scriptitiedoston komennolla `$ sudo nano finlandmetar` . Tänne kirjoitin seuraavanlaisen scriptin:

```
#!/bin/bash

data_dir="/usr/local/metar"

data_file="$data_dir/metarFinland.txt"

{
echo $(date);
metar EFET; metar EFHA; metar EFHK;
metar EFIV; metar EFJO; metar EFJY;
metar EFKE; metar EFKI; metar EFKK; 
metar EFKS; metar EFKT; metar EFKU; 
metar EFLP; metar EFMA; metar EFMI; 
metar EFOU; metar EFPO; metar EFRO; 
metar EFSA; metar EFSI; metar EFTP; 
metar EFTU; metar EFUT; metar EFVA;
} >> "$data_file"
```
Scripti luo /usr/local/metar -hakemistoon uuden tiedoston nimeltä "metarFinland.txt". Tämä tiedosto sisältää METAR-tiedot jokaiselta Suomen kaupallisessa- ja sotilaskäytössä olevalta kentältä. En lähde koodeja avaamaan, mutta esim. EFJY = Jyväskylä, EFTU = Turku jne. Kenttien koodit on kopioitu Ilmatieteenlaitoksen ilmailusään sivuilta, josta myös löytää kyseiset METAR-tiedotteet. (Ilmatieteenlaitos)


![image](https://github.com/hautadata/palvelintenhallinta-jh-moduuli/assets/148875340/11005d90-8bba-4b38-b33a-3967e0116b4a)
>Yllä: Metarin scriptitiedosto.
>

---

Tämän jälkeen tarvittiin vielä init.sls-tiedosto, joka kertoo orjakoneelle missä scripti sijaitsee ja minkä se kopioi orjalle. Komennolla `$ sudoedit init.sls` editori auki ja sinne seuraava sisältö:

```
/usr/local/bin/metarfinland:
  file.managed:
    - source: salt://metar/finlandmetar
    - mode: "0755"
```

![image](https://github.com/hautadata/palvelintenhallinta-jh-moduuli/assets/148875340/d2d8b48c-15ee-42e9-8b8c-9936a8c53aa1)
>Yllä: init.sls tiedosto.

---

Sittenhän voidaan lähteä ajamaan komentoja orjalle. Tässä kohtaa ajetaan vain orjalle numero 1, eli ei käytetä salt-funktiossa tähteä *. Ajan komennon `$ sudo salt 't001' state.apply metar` , ja saan heti lupaavan summaryn. Uusi tiedosto on luotu, ja succeeded: 1 (changed= 1). 

![image](https://github.com/hautadata/palvelintenhallinta-jh-moduuli/assets/148875340/4318cacf-ae29-4a52-99b4-26c7513eead8)
>Yllä: Orjalle heinää.

---

Luon vielä itse scriptissä olevan hakemiston orjakoneelle komennolla `$ sudo salt 't001' cmd.run "mkdir /usr/local/metar"` , jotta siitä ei tule erroreita komentoja ajettaessa. Tämän jälkeen ajetaan itse komento orjalla Saltilla, eli syötetään komento `$ sudo salt 't001' cmd.run "metarfinland"` . Tästä ei tullut mitään palautetta, mutta voimme cat-komennolla tarkistaa onko homma toiminut. Eli komento on `$ sudo salt 't001' cmd.run "cat /usr/local/metar/metarHelsinki.txt"`.

Jes! Hommahan toimii mainiosti. Tiedostossa on haluamamme aikaleima, sekä kaikkien kenttien METAR-tiedot!

![image](https://github.com/hautadata/palvelintenhallinta-jh-moduuli/assets/148875340/1a6b0cce-23fa-4750-90aa-3eb19c7b0dfb)
>Yllä: metarHelsinki.txt sisältö päivittynyt onnistuneesti!

---

## Automaattiset päivittelyt crontabilla.

Sitten halutaan että tiedostoa päivitetään automaattisesti tunnin välein. METAR-tiedotteet päivitetään myös vain tunnin välein, ellei säässä ole jotain merkittäviä muutoksia lyhyemmän ajan sisällä. Käytin ensimmäisessä vaiheessani jo crontabia, joten otetaan se käyttöön nytkin. Tästä tulikin hieman hankaluuksia, kun koitin muokata crontabia Saltin kautta. Yritin komentoa `$ sudo salt 't001' cmd.run "echo "0 * * * * metarfinland" | crontab -e/-l"`, useampaan kertaan. En kuitenkaan saanut echolle lisättyä haluamaani syntaksia tiedostoon. Tämä varmaankin johtuu siitä, että crontab-komento avaa aina editorin, ja niit
ei oikeen Saltin kautta voi käyttää. 

![image](https://github.com/hautadata/palvelintenhallinta-jh-moduuli/assets/148875340/81efa7b0-dbb2-45f2-9ffd-a6b3e99995e5)
>Yllä: crontab-lisäys ei toimi.

---

Päätin käydä tekemässä muutokset paikan päällä. Poistuin tmasterin yhteydeltä komennolla `$ exit` ja otin ssh-yhteyden t001-orjaan komennolla `$ vagrant ssh t001`.

Avasin crontab-tiedoston komennolla `$ sudo crontab -e` , ja lisäsin sinne hieman aiemmasta poikkeavan syntaksin `*/60 * * * * /usr/local/bin/metarfinland`. Tämä siis tarkoittaa että jokaisella tunnin 60. minuutilla järjestelmä ajaa automaattisesti metarfinland-scriptin.

![image](https://github.com/hautadata/palvelintenhallinta-jh-moduuli/assets/148875340/8f1248cb-5f43-482d-9e98-34e1c3ab98ae)
>Yllä: crontabilla automatisointi.

---

Ei auta kuin odotella, kun kello ei vielä ole tasatuntia. Tällä välin palaan tmasterille, ja teen sinne myös crontab-muutoksen, joka puskee tiedoston orjalta masterille automaattisesti myös kerran tunnissa heti metar-tiedoston päivityksen jälkeen. Avaan crontabin komennolla `$ crontab -e` , ja lisään sinne seuraavan syntaksin `*/01 * * * * sudo salt 't001' cp.push /usr/local/metar/metarHelsinki.txt`

Tämä syntaksi ajaa jäljempänä olevan komennon jokaisella tunnin ensimmäisellä minuutilla. cp.push taas puskee tiedoston orjalta herralle sen oletus minioncache-sijaintiin. (Salt Project s.a.)

![image](https://github.com/hautadata/palvelintenhallinta-jh-moduuli/assets/148875340/50eeb807-f740-47e4-b357-feba5c06d6b3)
>Yllä: Automaattisen puskun crontab-syntaksi.

---

Salt Projectin cp.push -kohdassa sanotaan, että toiminto on oletuksena pois käytöstä. Jos sen haluaa käyttöön, tulee configuraatiotiedoston file_recv päivittää arvoon "true". Nopealla haulla löysin config-filen sijainnin, joka sijaitsee herralla kohteessa /etc/salt/master. (Salt Project, s.a.)

Siirrytään kohteeseen komennolla `$ cd /etc/salt` . Siellä `$ ls` ja nähdään että master-niminen tiedosto on tosiaan olemassa. Komennolla `$ sudo nano master` se auki, ja nyt on muuten paljon rivejä. Tässä ei kannata scrollailla ja yrittää jos se pikku präntti iskisi silmään, vaan kätetään nanon "Where is"-toimintoa näppäinyhdistelmällä ctrl + W. Siihen kirjoitan "file_recv", ja löydän haluamani rivin heti!

Siinä kohdassa lukee oletuksena #file_recv: False , joten muokataan sitä ottamalla risuaita pois ja pistämällä Falsen tilalle True. Nyt meillä on file_recv siis käytössä herralla, ja tiedostojen pusku orjalta pitäisi onnistua!

![image](https://github.com/hautadata/palvelintenhallinta-jh-moduuli/assets/148875340/b59a9021-15c6-48de-a8f1-0760a93ecb2c)
>Yllä: file_recv: True

---

Pikakelaus seuraavaan tasatuntiin, ja pääsemme tarkastamaan miten toimii! Aiemmin mainitsemani minioncachen polku on herralla /var/cache/salt/minion/t001/files/ , joka sisältää orjan t001 tiedostot, jota sieltä pusketaan herralle. Tässä kohtaa näköjään itse orjan tiedostopolkukin on päässyt mukaan, eikä pelkkä tiedosto. Mennään siis aikas syvälle, ja orjalta puskettu tiedosto löytyy nyt herralta osoitteesta `/var/cache/salt/minion/t001/files/usr/local/metar/metarHelsinki.txt`

Suoritetaan cat-komento sille, ja nähdään että homma toimii erittäin hyvin! Meillä on tasatunnila 19:00:01 UTC (Suomen aika +2 tuntia) aikaleima, ja sen alla kenttien METAR-tiedotteet. Ne ovat päivittyneet n. 10 minuuttia aikaisemmin, UTC-ajalla 1850. Jes!

![image](https://github.com/hautadata/palvelintenhallinta-jh-moduuli/assets/148875340/4cde78a2-9028-442f-a0c6-17c5f216f139)
>Yllä: metarHelsinki.txt puskettu ja päivitetty automaattisesti orjalta herralle. Huom. history tail -n 2 (komentohistoria, viimeisimmät 2 komentoa) -komento, jolla vain vakuutan että ajoin juuri cat-komennon, koska alkua ei tuossa printtien seassa näy.

---

Huhhuh, hyvin näyttää toimivan. Käydään seuraavaksi toisen orjan kimppuun, ja pistetään se tallentamaan Euroopan 10:n suurimman kentän METAR-tiedotteet meille.

## METAR - Orja nro 2.

Sitten lähdetään tekemään sama setti toiselle orjalle, hieman eri twistillä. En tässä raportoi ihan yhtä yksityiskohtaisesti, koska teen käytännössä vain uudelleen kaiken mitä tuossa ylempänäkin. Mutta katsotaan pääasiat läpi!

Loin herralla uuden kansion aiemmassa käytetyn kansion sisään. Komento `$ mkdir /srv/salt/metar/metareurope`. Luon sinne scriptitiedoston komennolla `$ sudo nano metareurope` . Sisältö sama kuin aiemmassakin, mutta kenttien koodit muuttuneet. Muutin ne Euroopan 10:n suurimman kentän mukaisiksi, ja listalta löytyy mm. Lontoo Heathrow, Amsterdam, Istanbul ja Pariisi Charles De Gaulle. (Wikipedia, 11/2023)

```
#!/bin/bash

data_dir="/usr/local/metar"

data_file="$data_dir/metarEurope.txt"

{
echo $(date);
metar EGLL; metar LFPG; metar EHAM;
metar LTFM; metar LEMD; metar EDDF;
metar LEBL; metar EGKK; metar LIRF; 
metar LFPO;
} >> "$data_file"
```

![image](https://github.com/hautadata/palvelintenhallinta-jh-moduuli/assets/148875340/a2acc420-c29a-4887-8a90-3705356834cf)
>Yllä: Toisen orjan scriptitiedosto.

---

Sitten inits.sls komennolla `$ sudoedit init.sls` . Sen sisältö sama kuin ykkösorjalla, mutta eri tiedostopoluilla.

```
/usr/local/bin/metar:
  file.managed:
    - source: salt://metar/metareurope/metareurope
    - mode: "0755"
```

![image](https://github.com/hautadata/palvelintenhallinta-jh-moduuli/assets/148875340/7bef034d-b703-4dd1-931a-02e7fe4a96ff)
>Yllä: Toisen orjan init.sls

---

Sitten ei muuta kuin komentoa kehiin. Ajetaan tila toiselle orjalle komennolla `$ sudo salt 't002' state.apply metar/metareurope`. Sieltä tuleekin onnistunut lopputulos, succeeded: 1 (changed= 1).

![image](https://github.com/hautadata/palvelintenhallinta-jh-moduuli/assets/148875340/5a228f4a-5554-499d-b2a0-76bedb82a423)
>Yllä: state.apply onnistunut.

---

Sitten ajetaan jälleen komento `$ sudo salt 't001' cmd.run "metar"` . Sitten pitää tehdä crontab-päivitys, joten siirrytään t002-orjalle exitillä ja `$ vagrant ssh t002`. Siellä `$ sudo crotnab -e` ja lisätään syntaksi `*/60 * * * * /usr/local/bin/metar`. Eli metar-scripti ajetaan automaattisesti tasatunnein.

Siirrytään takaisin herralle, ja tehdään myös sinne crontab-muutos, jotta toisenkin orjan tiedostot pusketaan automaattisesti herralle. tmasterilla `$ crontab -e` ja lisätään sinne `*/01 * * * * sudo salt 't002' cp.push /usr/local/metar/metarEurope.txt`. 

Sitten odotellaan tasatuntia. Ja kun se tuli, käydään katsomassa osoite `/var/cache/salt/minion/t002/files/usr/local/metar/`. Eli tällä kertaa t002-koneen lähettämät tiedostot. Siellä komento `$ cat metarEurope.txt` , ja näemme että tälläkin orjalla homma toimii moitteettomasti. Aikaleima tasatunnilta, alla halutut METAR-tiedotteet. Ja mikä parasta, tiedosto päivittynyt ja tullut herralle automaattisesti. Nyt on huojentunut olo!

![image](https://github.com/hautadata/palvelintenhallinta-jh-moduuli/assets/148875340/87b84f3f-b9f0-4295-ba52-4925557cca1e)
>Yllä: t002-orjan puskema metar-tiedosto. Toimii!

---

Huhhuh. Käyn vielä katsomassa parin tunnin päästä tuleeko muutoksia ja tiedostoja tosiaan vieläkin automaattisesti, ja siltä se tosiaan näyttää. Kävin katsomassa t001-orjan lähettämää dataa, ja allaolevasta kuvasta näkyy että 22:00 UTC on tullut viimeinen, ja koneeni kello on 0:26, eli kyllä viimeinen muutos on tullut viimeisen tasatunnin aikana. Ja tiedostoa scrollaamalla näen, että muutoksia on tullut näitä ennenkin! Pidetään vagrant päällä, ja katsotaan kuinka iso tiedosto saadaan kasaan :D

![image](https://github.com/hautadata/palvelintenhallinta-jh-moduuli/assets/148875340/f36f6c91-a460-41b0-ad5d-c3a2ccc3556b)
>Yllä: Hyvin toimii


## Itsearviointi

Olen melko tyytyväinen työni lopputulokseen. Se ei ole vaativimmasta päästä, mutta lähtötasooni (nolla) nähden siinä oli itselleni henkilökohtaisesti sopivaa haastetta. Moduulin aikana tuli opittua uusia asioita, kuten cronin käyttöä, cp.pushia ja master configuration tiedoston muokkausta. Kaiken kaikkiaan moduulia oli hauska työstää, vaikkakin se oli hieman aikaa vaativa. Se oli kuitenkin opettavainen kokemus, niin kuin koko kurssikin tähän mennessä! 

Kiitokset Terolle mahtavasta kurssista, ja kiitos sinulle jos jaksoit lukea tänne asti! Hyvää joulun odotusta.


## Lähteet

Awati, R. Techtarget. 2/2023. What is crontab?. Luettavissa: https://www.techtarget.com/searchdatacenter/definition/crontab. Luettu: 9.12.2023.

Hautadata. 2023. H5 - CSI Kerava. Luettavissa: https://github.com/hautadata/palvelintenhallinta-jh/blob/main/h5-csikervo.md.

Ilmatieteenlaitos. Ilmailusää. Luettavissa: https://ilmailusaa.fi/index.html#flash_checkbox=checked#id=radar#map=southern-finland#level=null#top=0. Luettu: 9.12.2023.

Karvinen, T. 13.10.2023. Infra as Code 2023. Luettavissa: https://terokarvinen.com/2023/configuration-management-2023-autumn/. 

Salt Project. s.a. CONFIGURING THE SALT MASTER. Luettavissa: https://docs.saltproject.io/en/latest/ref/configuration/master.html. Luettu: 9.12.2023.

Salt Project. s.a. SALT.MODULES.CP. Luettavissa: https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.cp.html. Luettu: 9.12.2023.

Wikipedia. 12.11.2023. List of the busiest airports in Europe. Luettavissa: https://en.wikipedia.org/wiki/List_of_the_busiest_airports_in_Europe. Luettu: 9.12.2023.





