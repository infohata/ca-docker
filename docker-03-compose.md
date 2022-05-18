# Docker Compose

Docker compose yra įrankis, skirtas kompleksiškam docker konteinerių sprendimui orkestruoti - tiek konteinerių sukūrimui, tiek paleidimui ir valdymui. `docker-compose.yml` faile yra sukonfigūruojamas visas veikimo aplinkos ciklas - nuo konteinerių sukūrimo, iki paleidimo ir uždarymo. Konteinerių priklausomybės, tinklas, aplinkos savybės (`environment variables`) ir visa kita aprašomos čia.

---
## `docker-compose.yml` failas

Sukursime pavyzdinį failą elementariam Django projektui, kuriame jau yra `Dockerfile`, `requirements.txt`, sukonfigūruotas `uwsgi` ir duomenų bazė `postgresql`.

``` yml
version: "3.7"  # nurodome suderinamo docker-compose versiją
services:       # services yra konteinerių sąrašas
  dev:          # konfigūruojame konteinerį pavadinimu web
    # build komanda nudorome, kad iš kuriame kataloge (šiuo atveju iš esančio katalogo, .) esančio Dockerfile statyti konteinerį
    build: .
    # paveiksliuko (docker image) pavadinimas
    image: projektas:dev
    # konteinerio pavadinimas
    container_name: projektas.dev
    # parametras tty nurodo, ar konteineris gali naudotis linux subsistemos serijinės sąsajos TeleTYpewriter savybėmis, kurių iš esmės reikia komandinei eilutei funkcionuoti. Tuo pačiu atidarom ir stdin - komandinės eilutės įvedimo funkciją, kurią gali tekti prireikti naudoti pvz. su python input() funkcija.
    tty: true
    stdin_open: true
    # restart komanda nurodo sąlygas, kada perkrauti konteinerį jam išsijungus. Produkcinėje aplinkoje tai turėtų būti always. 
    restart: always
    # kokias komandas vykdyti paleidžiant konteinerį
    command: >
      bash -c "./projektas/manage.py migrate &&
               ./projektas/manage.py runserver 0.0.0.0:8000"
    # tinklo konfigūracija
    ports:
      - 8000:8000
    # volumes - disko sąsaja. Nerekomenduojame naudoti su Windows'ais.
    volumes:
      - .:/app
    # priklausomybės - kurie konteineriai turėtu būti paleisti, paleidžiant šį konteinerį.
    depends_on:
      - db
    # aplinkos savybės (environment variables)
    environment:
      PYTHONIOENCODING: UTF-8
      DJANGO_SETTINGS_MODULE: projektas.settings
  db:           # Duomenų bazės konteineris
    # naudosime standartinį postgres image
    image: postgres
    # konteinerio pavadinimas
    container_name: projektas.db
    restart: always
    ports:
      - 5432:5432
    volumes:
      - ./dbdata:/var/lib/postgresql/data
    # nurodžius environment'e duomenų bazės parametrus, nauajs postgres konteineris šiais kredencialais sukurs tuščią duomenų bazę. Produkcinėje aplinkoje siūlytume nenurodyti, arba pakeisti čia nustatytus.
    environment:
      POSTGRES_DB: projektas
      POSTGRES_USER: projektas
      POSTGRES_PASSWORD: nesakysiu
      POSTGRES_PORT: 5432
```

---
## Docker compose valdymo komandos
Kad paleisti savo docker'io kompoziciją, naudojame komandą `docker-compose up` su argumentu `-d`, kad paleisti kaip foninį servisą (daemon).
```
docker compose up -d
```

Lygiai taip pat galime naudoti subkomandas `down` ir `restart`, pvz.:
```
docker-compose restart
docker-compose down
```

---
## Užduotys

1. Sukonfigūruokite kešavimo servisą su redis, ir įtraukite redis konteinerį į savo kompiziciją
1. Sukonfigūruokite produkcinę aplinką su `uwsgi` per proxy su `nginx` arba `apache`, ir pridėkite prie savo kompozicijos `nginx` arba `apache` konteinerius. Nepamirškite apache arba nginx konfigūracinių failų nukopijuoti į jų konteinerius, į kelius kur jiems priklauso būti, arba bent jau sudėti atitinkamus "symbolic links".
1. parašykite python skriptą, kuris neleistų vykdyti migracijų ir taip pat paleisti serviso (užlaikytų run veiksmų eigą), jeigu duomenų bazė nepasiekiama iš pagrindinio konteinerio.
1. pridėkite komandą, kuri kiekvieno paleidimo metu pabandytų surinkti statikos failus su `python manage.py collectstatic -y`, ir sukompiliuotų vertimus `... compilemessages`, jeigu projekte yra daugiakalbystė.
1. parašykite python skriptą, kuris automatiškai administruotų `logger` kuriamų failų archyvą - kompresuotų `.log` failus kasdien, išvalydamas einamųjų failų turinį, ir trintų senesnius nei mėnesio `.log` failų archyvus.

---
### Papildoma informacija Anglų kalba
[Getting Started with Docker](https://docs.docker.com/get-started/)
