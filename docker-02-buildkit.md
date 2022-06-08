# Docker BuildKit

Standartinis python konteinerio paveiksliukas (`image`) neveikia jo teisingai nesukonfigūravus, todėl mes turėsime naudoti BuildKit, kurio pagalba įkelsime savo programą į docker konteinerį, prieš jį paleisdami.

Docker BuildKit yra pagal nutylėjimą įjungtas visose Docker Desktop distribiucijose. Docker Server distribiucijose mums jį reikia įjungti. Tai galima padaryti įtraukiant arba papildant `"features":{"buildkit":true}` faile `/etc/docker/daemon.json`. Pastaba: kad redaguoti failus `/etc/` kataloge reikia root teisių arba `sudo`.

Išsirinkite bent vieną projektą, kurį naudosime. Rekomenduojame Django arba Flask projektą, kurį galima naudoti kaip serverį.

### Dockerfile

Atsidarykite projektą, jame sukurkite failą `Dockerfile` (tiesiog tokį, be extension).

Pirmoje eilutėje visada įrašome nuorodą į tai, kokio interpretatoriaus reikės failui įvykdyti:

``` Dockerfile
# syntax=docker/dockerfile:1
```
Visual Code šitoje vietoje gali žymėti klaidą - nekreipkite dėmesio. Po sekančio skomandos susitvarkys.

Nurodome, kokį bazinį paveiksliuką (`image`) naudosime.

``` Dockerfile
FROM python:slim-buster
```

šis paveiksliukas jau turi viską, ko reikia mūsų pirmai Flask arba Django aplikacijai paleisti.

Nustatoome kokiame kataloge leisime savo python projektą:

``` Dockerfile
WORKDIR /app
```

Instrukcija nukopijuoti į konteinerį projekto failus, kur projekto katalogo pavadinimas šiuo atveju yra `project`. Pakeiskite `project` į savo projekto pavadinimą:

``` Dockerfile
COPY ./project .
COPY ./requirements.txt .
```

Instrukcija paleisti vietinį konteinerio pip ir suinstaliuojame priklausomybes

``` Dockerfile
RUN pip3 install -r requirements.txt
```

Ir paleidžiame Django (arba Flask savo nuožiūra) testinį serverį

``` Dockerfile
WORKDIR /app/project
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

Dabar galime surinkti savo konteinerį, `cmd` komandinėje eilutėje paleisdami:
``` bash
docker build --tag django-app-project .
```

Dabar galime ir paleisti savo konteinerį - nepamirškime sukurti port forwarding'o (panašiai į NAT) savo konteineriui. Dabar pavyzdžiui nukreipsime iškart į `port 80`.
``` bash
docker run -d -p 80:8000 --name django-project django-app-project
```
Galite atsidaryti savo projektą dabar naršyklėje.
Ir nuo dabar veiks docker start/stop komandos konteineriui django-project.

---
## gunicorn su Django

Kad paleisti Django serverį produkcijai, tinkamiau yra naudoti ne pačio Django `runserver`, bet `gunicorn` web serverį. Tam reikia:

- `pip install gunicorn`, taip pat nepamirškite atnaujinti priklausomybių failo (requirements.txt)
- projekto `settings.py` turi būti sutvarkyti pilnai `STATIC` ir `MEDIA` parametrai, ir teisingai nurodyti `STATIC_ROOT` ir `MEDIA_ROOT`.
- `Dockerfile` po `WORKDIR /app/project` pridedame eilutes, jeigu dar nepasidarėte, migracijoms ir statikai surinkti, ir pakeičiam CMD eilutę kad leistų `gunicorn` vietoj `runserver`:

``` Dockerfile
RUN python manage.py collectstatic --noinput
RUN python manage.py migrate
CMD ["gunicorn", "-b", "0.0.0.0:8000", "project.wsgi"]
```

---
## Užduotys

1. Sukurkite konteinerį savo kitam Django projektui, naudodami ne runserver, o `gunicorn` arba `uwsgi`. Nepamirškite `collectstatic` ir įtraukti `gunicorn` ar `uwsgi` į `requirements.txt`.
---
### Papildoma informacija Anglų kalba
[Getting Started with Docker](https://docs.docker.com/get-started/)
