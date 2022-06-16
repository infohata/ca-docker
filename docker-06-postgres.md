# Postgres duomenų bazės konteineris kompozicijoje [WIP]

Kad naudoti postgres duomenų bazę, mums reikės į python priklausomybes įtraukti `psycopg2` paketą. Rekomenduojame naudoti binary versiją (sukompiliuotą), nes pagrindiniai bibliotekos komponentai yra parašyti su C/C++.
``` bash
pip install psycopg2-binary
```
Nepamirškite įtraukti į `requirements.txt`.
``` bash
pip freeze >requirements.txt
```

Savo Django projekto `settings.py` faile taip pat pakeičiame duomenų bazės konfigūraciją:

``` python
DATABASES = {
    'default': {
        "ENGINE": "django.db.backends.postgresql",
        "HOST": "postgres",
        "NAME": "project",
        "USER": "project",
        "PASSWORD": "nesakysiu",
        "PORT": 5432
    }
}
```

Dabar į projekto pagrindinį katalogą, šalia `manage.py`, sukuriame `wait_for_postgres.py` failą. Šio failo tikslas bus užlaidyti visus su Django projektu susijusius veiksmus, iki kol bus sukurtas ir sėkmingai paleistas duomenų bazės konteineris.

``` python
import os
import logging
from time import time, sleep
import psycopg2

# tikrinimo dažnumas, pagal nutylėjimą laukiame 30 sekundžių, ir nepavykus po sekundės laukiame iš naujo.
check_timeout = os.getenv("POSTGRES_CHECK_TIMEOUT", 30)
check_interval = os.getenv("POSTGRES_CHECK_INTERVAL", 1)

# duomenų bazės konfigūracija - pagal nutylėjimą turėtų sutapti su Django nustatymais.
config = {
    "dbname": os.getenv("POSTGRES_DB", "project"),
    "user": os.getenv("POSTGRES_USER", "project"),
    "password": os.getenv("POSTGRES_PASSWORD", "nesakysiu"),
    "host": os.getenv("POSTGRES_HOST", "postgres"),
    "port": os.getenv("POSTGRES_PORT", "5432")
}

# sukonfigūruojame logerį
logger = logging.getLogger()
logger.setLevel(logging.INFO)
logger.addHandler(logging.StreamHandler())
logger.info(
    f"DB config {config['dbname']} {config['user']} {config['host']} ...")

# įsimename dabartinį laiką
start_time = time()

# prisijungimo į duomenų bazę tikrinimo funkcija
def pg_isready(host, user, password, dbname, port):
    while time() - start_time < check_timeout:
        try:
            conn = psycopg2.connect(**vars())
            logger.info("Postgres is ready! ✨ 💅")
            conn.close()
            return True
        except psycopg2.OperationalError:
            logger.info(
                f"Postgres isn't ready. Waiting for {check_interval} sec...")
            sleep(check_interval)

    logger.error(
        f"We could not connect to Postgres within {check_timeout} seconds.")
    return False

pg_isready(**config)

```

Ir galiausiai papildome patį `docker-compose.yml` failą:

``` yml
## Prie services: dev: nuo galo pridedame šį fragmentą, perrašydami command:
    # priklausomybės - kurie konteineriai turėtu būti paleisti, paleidžiant šį konteinerį.
    depends_on:
      - db
    # pervadiname db konteinerio host lokaliame projekto tinkle. Nepamirškite duomenų bazės konfigūracijos faile nurodyti `host=postgres` vietoj `host=localhost`
    links:
      - db:postgres
    # vykdome komandas startuojant konteineriui, o ne konstruojant to konteinerio paveiksliuką (image). Rekomenduojame šias komandas iš Dockerfile pašalinti arba užkomentuoti.
    command: >
      bash -c "python wait_for_postgres.py &&
               python manage.py migrate &&
               python manage.py collectstatic --noinput &&
               gunicorn project.wsgi --bind 0.0.0.0:8000"
  # sukuriame naują duomenų bazės servisą
  db:
    # naudosime standartinį postgres image
    image: postgres
    # konteinerio pavadinimas
    container_name: project.db
    restart: always
    ports:
      - 5432:5432
    volumes:
      - ./dbdata:/var/lib/postgresql/data
    # nurodžius environment'e duomenų bazės parametrus, nauajas postgres konteineris šiais parametrais sukurs tuščią duomenų bazę. Produkcinėje aplinkoje siūlytume nenurodyti jautrių duomenų šiame faile, o naudoti tarpinį .env failą, įdėtą į .gitignore.
    environment:
      POSTGRES_DB: project
      POSTGRES_USER: project
      POSTGRES_PASSWORD: nesakysiu
      POSTGRES_PORT: 5432
```
