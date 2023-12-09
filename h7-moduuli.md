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

Tehdään seuraavaksi scripti, joka hakee useamman kaupungin säätiedot ja tallentaa ne yksittäisiin tiedostoihin.

