# Docker BuildKit

Standartinis python konteinerio paveiksliukas (`image`) neveikia jo teisingai nesukonfigūravus, todėl mes turėsime naudoti BuildKit, kurio pagalba įkelsime savo programą į docker konteinerį, prieš jį paleisdami.

Docker BuildKit yra pagal nutylėjimą įjungtas visose Docker Desktop distribiucijose. Docker Server distribiucijose mums jį reikia įjungti. Tai galima padaryti įtraukiant arba papildant `"features":{"buildkit":true}` faile `/etc/docker/daemon.json`. Pastaba: kad redaguoti failus `/etc/` kataloge reikia root teisių arba `sudo`.

Išsirinkite bent vieną projektą, kurį naudosime. Rekomenduojame Django arba Flask projektą, kurį galima naudoti kaip serverį.

### Dockerfile

Atsidarykite projektą, jame sukurkite failą `Dockerfile` (tiesiog tokį, be extension).

Pirmoje eilutėje visada įrašome nuorodą į tai, kokio interpretatoriaus reikės failui įvykdyti:

```
# syntax=docker/dockerfile:1
```

Nurodome, kokį bazinį paveiksliuką (`image`) naudosime.

```
FROM python:slim-buster
```

šis paveiksliukas jau turi viską, ko reikia mūsų pirmai Flask arba Django aplikacijai paleisti.

Nustatoome kokiame kataloge leisime savo python projektą:

```
WORKDIR /app
```

Instrukcija nukopijuoti į konteinerį projekto failus:

```
COPY . .
```

Instrukcija paleisti vietinį konteinerio pip ir suinstaliuojame priklausomybes

```
RUN pip3 install -r requirements.txt
```

Ir paleidžiame Django (arba Flask savo nuožiūra) testinį serverį

```
WORKDIR /app/proejct
CMD ["python3", "manage.py", "runserver", "0.0.0.0:8000"]
```

Dabar galime surinkti savo konteinerį, `cmd` komandinėje eilutėje paleisdami:
```
docker build --tag django-app-project .
```

Dabar galime ir paleisti savo konteinerį - nepamirškime sukurti port forwarding'o (panašiai į NAT) savo konteineriui. Dabar pavyzdžiui nukreipsime iškart į `port 80`.
```
docker run -d -p 8000:80 --name django-project django-project
```
Galite atsidaryti savo projektą dabar naršyklėje.
Ir nuo dabar veiks docker start/stop komandos konteineriui django-project.

---
## Užduotys

* Sukurkite konteinerį savo Django projektui, naudodami ne runserver, o `uwsgi`. Nepamirškite `collectstatic`, taip pat įtraukite `uwsgi` į `requirements.txt`.
* Perkonfigūruokite savo projekto konteinerį, kad `uwsgi` būtų paleidžiamas per `nginx` ar `apache` proxy. (reikia būti pabaigus Flask/Django deployment kursų dalį)
* Sukonfigūruokite Django projektą naudoti PostgresSQL duomenų bazę iš atskiro duomenų bazės konteinerio `postgres`. Tam reikės pakoreguoti projekto `settings.py`, taip pat į `requirements.txt` įtraukite `psycopg2-binary`.

### Papildoma informacija Anglų kalba
[Getting Started with Docker](https://docs.docker.com/get-started/)

