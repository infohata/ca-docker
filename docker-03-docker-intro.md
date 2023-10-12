# DOCKER

## Parsisiuntimas:
https://docs.docker.com/get-docker/

Windows ir Mac siunčiamės Desktop programą, kurią instaliuojame. Linux'ui naudojame serverinę komandinės eilutės versiją. 

!!! Prieš instaliuojant Windows'uose, reikia susitvarkyti procesoriaus virtualizaciją ir Windows Subsistemą Linux'ui. !!!

---
## Docker Desktop varotojo sąsaja

Palengvina vizualiai valdyti docker'io container'ius, bet yra visiškai nereikalingas. Naudosime tik komandinę eilutę iš `cmd`.


## Docker komponentai

### Container (app)
Darbinė virtuali aplinka, sukurta iš Docker Image (paveiksliuko), volume(s) - sąsajų su realiais failais ar direktorijomis jūsų tikroje failų sistemoje, ir savo egzistencijai naudojanti jūsų kompiuterio resursus, kaip procesorių, RAM, disko vietą, vaizdo plokštę, tinklo infrastruktūrą, ir t.t.

### Image
Paveiksliukas. Tai yra virtualios mašinos/operacinė sistemos produkcinis paveiksliukas, naujausia jo versija. Paveiksliuką galime parsisiųsti iš repozitorijos ir paleisti kaip konteinerį.

### Volume
Disko vieta, kurioje docker'is saugo konteinerio pokyčius tarp image ir container esamos būsenos. Per volume taip pat gali būti susieti duomenys (failai arba net visos direktorijos su subdirektorijomis) tarp konteinerio ir jūsų failų sistemos.

---
## Docker komandos

Pagrindiniai veiksmai su Dockeriu atliekami per komandinę eilutę, naudojant `Docker Engine`. Vartotojo sąsaja Windows/Mac yra skirta tik patogiam konteinerių situacijos atvaizdavimui (kolkas).

SVARBU: Komandų argumentai visada nurodomi prieš konteinerio pavadinimą, ypač komandose `run`, `start` ir `exec`.

`docker help [komanda]` - svarbiausia komanda visame dockeryje - jos pagalba galėsite greitai gauti visų komandos parametrų aprašymą ir naudojimo taisykles.

### Trys viename `pull`, `start`, `exec`

`docker run --name [name] -it [image] [command]"` - taip pat galime perduoti papildomas komandas ar argumentus konteineriui. Pvz. `docker run --name "pytest" -it python:slim-buster bash` parsisiūs, instaliuos ir paleis naujausią `slim-buster` Linux distribucijos `python` `docker image` paveiksliuką ir sukurs standartinį konteinerį pavadinimu `pytest` iš to paveiksliuko. Naudojant parametrus `-it` galime paleisti konteinerį su interaktyvia komandine eilute.

### Image parsisiuntimas, peržiūra, išmetimas

`docker pull [image]` - atnaujins `docker image` į naujausią arba nurodytą versiją. Versiją galima nurodyti per dvitaškį po paveiksliuko pavadinimo, pvz. `docker pull python:latest`, arba `docker pull python:slim-buster`

`docker images` - išspausdins jūsų sistemoje egzistuojančių paveiksliukų sąrašą, su esminiais parametrais, įskaitant disko resursų sunaudojimą.

`docker rmi [image id]` - ištrina paveiksliuką ir visus su juo susijusius priklausomybių komponentus. Kad veiktų, paveiksliukas neturi būti naudojamas jokiuose aktyviuose konteineriuose

### Konteinerių peržiūra, paleidimas/stabdymas ir išmetimas

`docker ps` - išspausdina konteinerių sąrašą su esminiais parametrais, kaip paveiksliuko pavadinimas, paleista komanda, statusas. `-a` parametras leis matyti ir neaktyvius konteinerius.

`docker rm [container name/id]` ištrina konteinerį.

`docker start [container name/id]` - paleidžia konteinerį

`docker stop [container name/id]` - sustabdo konteinerį

`docker exec [container name/id] [command]` - įvykdo konteineryje komandą. pvz. `docker exec -it pytest bash`. Šiuo atveju `-it` nurodo, kad naudosime interaktyviai komandinę eilutę.

### Failų kopijavimas į/iš konteinerio

`docker cp [from] [to]` - kopijuoja failą iš vienos vietos į kitą. Gali kopijuoti failus tarp konteinerių, arba tarp jūsų failo sistemos ir konteinerio (į bet kurią pusę). Jeigu failas nurodomas konteineryje, konteinerio id/pavadinimą nurodome iki dvitaškio. pvz. `docker cp test.py pytest:/app/` nukopijuos `test.py` failą iš esamos direktorijos į konteinerio `pytest` direktoriją `/app`. Jeigu nurodome kelią be konteinerio pavadinimo (prieš dvitaškį), tai nurodytas kelias/failas yra imamas nuo esančios komandinės eilutės.

---
## Užduotys

Užduotims įvykdyti turi būti pabaigtas Linux pradmenų kursas, arba bent jau įsisavinta Linux pradmenų kurso teorija.

1. Sukurkite naują `python` konteinerį, nauju pavadinimu
   * paleiskite parsisiųstą konteinerį;
   * susiinstaliuokite jame su `apt` trūkstamus įrankius tokius kaip teksto redaktoriu (`nano` ar kitą);
   * sukurkite `python` konteineryje `/app` katalogą, jame sukurkite kelis `.py` failus ir paeksperimentuokite;
   
   Pastaba: jeigu norite sukurti `virtualenv`, reikalingus `virtualenv` python modulius rasite `apt` repozitorijose. Aktyvavimas `venv/bin/activate` - kiek kitaip nei Windows'uose.

1. Nukopijuokite į naują konteinerį savo Flask ar Django projektą, paleiskite jį, pakoreguokite konteinerio paleidimą, kad startuotų kaip `daemon` (kaip foninis servisas) su raktu `-d`, ir priskirkite IP portą su raktu `-p 5000:80`. Čia pirmas skaičius yra portas atidarytas konteineryje, o antras skaičius nurodo, į kurį portą nukreipti jūsų kompiuteryje.

1. Išbandykite nostalgiją - susiinstaliuokite MC. `apt install mc`;


---
### Papildoma informacija Anglų kalba
[Getting Started with Docker](https://docs.docker.com/get-started/)
