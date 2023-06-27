# Diegimas su atskiru nginx konteineriu

Dabar savo projekte sukuriame `nginx` katalogą, jame sukuriame  `Dockerfile`, kuriame nurodysime instrukcijas nuosavo `nginx` paveiksliuko sukūrimui, į kurį sudėsime savo projekto `.conf` konfigūraciją.

``` Dockerfile
# syntax=docker/dockerfile:1
FROM nginx:latest
COPY ./project.conf /etc/nginx/conf.d/default.conf
```

Norėdami teisingai įdiegti projektą su nginx, rekomenduojame naudoti oficialų nginx konteinerio paveiksliuką. Tam rekomenduojame "surinkti" savo konfigūracijos nginx paveiksliuką, į jį perduodant savo projekto .conf failą. Jeigu serveryje leisite tik vieną projektą - tiesiog perrašysime savo konfigūracijos failu `/etc/nginx/conf.d/default.conf` failą. Čia pateikiamas pavyzdinis projekto `project.conf` failas, kurį galite prie projekto pasidėti `nginx` kataloge.

``` sh
# aprašome savo projekto backend'o upstream, kurį aptarnaus projekto gunicorn. Čia host turi sutapti su vėliau konfigūruojamu docker-compose python konteinerio sufiksu, kuris šio kurso atveju nustatytas kaip `dev`:
upstream project-backend {
    server project:8000;
}

server {
    # nustatome domain name, kuriuo bus galima kreiptis į serverį. Django settings ALLOWED_HOSTS sąraše turi būti įtrauktas šis domenas.
    server_name project.local;

    # nurodome kur padėsime Django static katalogą
    location /static/ {
	alias /app/static/;
    }

    # nurodome kur padėsime Django media katalogą
    location /media/ {
        alias /app/media/;
    }

    # leidžiame per URL siųstis failus, jeigu jie randami pagal URI.
    location / {
        try_files $uri @proxy_to_wsgi;
    }

    # perrašome URI pagal Django/gunicorn taisykles, kurios tiesiog aklai nukopijuotos nuo Django rekomendacijų.
    location @proxy_to_wsgi {
        uwsgi_param  QUERY_STRING       $query_string;
        uwsgi_param  REQUEST_METHOD     $request_method;
        uwsgi_param  CONTENT_TYPE       $content_type;
        uwsgi_param  CONTENT_LENGTH     $content_length;

        uwsgi_param  REQUEST_URI        $request_uri;
        uwsgi_param  PATH_INFO          $document_uri;
        uwsgi_param  DOCUMENT_ROOT      $document_root;
        uwsgi_param  SERVER_PROTOCOL    $server_protocol;
        uwsgi_param  REQUEST_SCHEME     $scheme;
        uwsgi_param  HTTPS              $https if_not_empty;

        uwsgi_param  REMOTE_ADDR        $remote_addr;
        uwsgi_param  REMOTE_PORT        $remote_port;
        uwsgi_param  SERVER_PORT        $server_port;
        uwsgi_param  SERVER_NAME        $server_name;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Host $server_name;

        # šioje vietoje nurodome kur yra mūsų gunicorn servisas - turi sutapti su upstream
        proxy_pass http://project:8000;
    }

    # laukiame užklausų 80 portu (http). Jeigu naudotume SSL/TLS sertifikatą, jį sukonfigūruotume 443 porte su `ssl` sufiksu ir nurodytume kur ieškoti sertifikatų - pavyzdys žemiau. 
    listen 80;
    # listen 443 ssl; # managed by Certbot
    # ssl_certificate /etc/letsencrypt/live/project.local/fullchain.pem; # managed by Certbot
    # ssl_certificate_key /etc/letsencrypt/live/project.local/privkey.pem; # managed by Certbot
    # include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    # ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}

# server {
#     if ($host = miracle.endless.pro) {
#         return 301 https://$host$request_uri;
#     } # managed by Certbot

#     server_name project.local;
#     listen 80;
#     return 404; # managed by Certbot
# }
```

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
    image: project:dev
    # konteinerio pavadinimas
    container_name: project.dev
    # parametras tty nurodo, ar konteineris gali naudotis linux subsistemos serijinės sąsajos TeleTYpewriter savybėmis, kurių iš esmės reikia komandinei eilutei funkcionuoti. Tuo pačiu atidarom ir stdin - komandinės eilutės įvedimo funkciją, kurią gali tekti prireikti naudoti pvz. su python input() funkcija.
    tty: true
    stdin_open: true
    # restart komanda nurodo sąlygas, kada perkrauti konteinerį jam išsijungus. Produkcinėje aplinkoje tai turėtų būti always. 
    restart: always
    # tinklo konfigūracija
    ports:
      - 8000:8000
    # volumes - disko sąsaja, kur konteineris sinchronizuos savo failus su realiais diske esančiais failais. Šių failų nereikės kopijuoti su cp. Taip pat panašiai sinchronizuosime ir `static` bei `media` katalogus su nginx konteineriu.
    volumes:
      - ./project:/app
    # vykdome komandas startuojant konteineriui, o ne konstruojant to konteinerio paveiksliuką (image). Rekomenduojame šias komandas iš Dockerfile pašalinti arba užkomentuoti.
    command: >
    bash -c "python manage.py migrate &&
             python manage.py collectstatic --noinput &&
             gunicorn library.wsgi --bind 0.0.0.0:8000"
  nginx:           # Nginx konteineris
    # surenkame nginx konteinerį su mūsų projekto .conf
    build: ./nginx/
    # savo surinkto nginx image paveiksliuko pavadinimas ir tagas
    image: nginx:project
    # konteinerio pavadinimas
    container_name: project.nginx
    # jeigu konteineris pakibtų, jį visada perkrausime
    restart: always
    # kokius portus atidaryti - jeigu reikia, įtraukite ir SSL
    ports:
      - 80:80
      # - 443:443
    # nurodome tinklo sąsają su pagrindiniu projekto konteineriu `dev`.
    links:
      - dev:project
    # sinchronizuojame failus tarp konteinerio ir projekto `static` ir `media` katalogų. Šiuo atveju, netgi padarius `python manage.py collectstatic` iš projekto `dev` konteinerio, `nginx` konteineryje atitinkami failai taip pat atsinaujins.
    volumes:
      - ./project/media:/app/media
      - ./project/static:/app/static
```

Kad veiktų nurodytas domenas (http://project.local/) - reikia jį įsidėti į hosts failą. Windows'uose dažniausiai būna `C:\Windows\System32\drivers\etc\hosts`.
``` hosts
127.0.0.1 project.local
```

---
## Docker compose valdymo komandos

Surenkame kompoziciją:
``` bash
docker-compose build
```

Kad paleisti savo docker'io kompoziciją, naudojame komandą `docker-compose up` su argumentu `-d`, kad paleisti kaip foninį servisą (daemon). Kol neįsitikinote kad veikia,  atskiroje komandinėje eilutėje paleiskite docker-compose be `-d` parametro, kad matytumėte kas neveikia.
``` bash
docker-compose up -d
```

Lygiai taip pat galime naudoti subkomandas `down` ir `restart`, pvz.:
``` bash
docker-compose restart
docker-compose down
```

---
## Užduotys

1. Sukonfigūruokite kešavimo servisą su redis, ir įtraukite redis konteinerį į savo kompiziciją
1. Sukonfigūruokite produkcinę aplinką su `uwsgi` per proxy su `nginx` arba `apache`, ir pridėkite prie savo kompozicijos `nginx` arba `apache` konteinerius. Nepamirškite apache arba nginx konfigūracinių failų nukopijuoti į jų konteinerius, į kelius kur jiems priklauso būti, arba bent jau sudėti atitinkamus "symbolic links".
1. parašykite python skriptą, kuris neleistų vykdyti migracijų ir taip pat paleisti serviso (užlaikytų run veiksmų eigą), jeigu duomenų bazė nepasiekiama iš pagrindinio konteinerio.
1. pridėkite komandą, kuri kiekvieno paleidimo metu pabandytų surinkti statikos failus su `python manage.py collectstatic --noinput`, ir sukompiliuotų vertimus `... compilemessages`, jeigu projekte yra daugiakalbystė.
1. parašykite python skriptą, kuris automatiškai administruotų `logger` kuriamų failų archyvą - kompresuotų `.log` failus kasdien, išvalydamas einamųjų failų turinį, ir trintų senesnius nei mėnesio `.log` failų archyvus.

---
### Papildoma informacija Anglų kalba
[Getting Started with Docker](https://docs.docker.com/get-started/)
