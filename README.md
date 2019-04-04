Docker Workshop - eine Einführung
=========

```
git clone github.com/fbsb/docker-workshop.git
cd docker-workshop
```

Erste Schritte
---------------

https://docs.docker.com/install

Alternativ: Virtualbox + Vagrant
---------------

https://www.vagrantup.com/downloads.html

https://www.virtualbox.org/wiki/Downloads

> Für bestimmte Linux Distributionen werden `Vagrant` und `VirtualBox` auch als Paket angeboten.

```
vagrant up
vagrant ssh
```

Innerhalb der Vagrant VM:

```
cd /vagrant
```

Übungen:
--------

### 1. Basics

Docker image vom DockerHub herunterladen:

```
docker pull hello-world
```

Docker Container ausführen:

```
docker run hello-world
```

### 2. Web-App als Container verpacken

Ziel dieser Übung ist die containerisierung einer simplen Web-App.


```
cd 2048
```

Um die App in einem Container zu verpacken, müssen wir die Dockerfile anpassen.

Da hier von einer Web-App die Rede ist, brauchen wir einen Webserver. Hierzu installieren wir `nginx`.

Folgende Zeilen in der Dockerfile müssen hinzugefügt werden.

```
RUN apt-get update
RUN apt-get install -y nginx
```

Als nächstes kopieren wir die App in dem Document Root des Webservers. Default Document Root für `nginx` ist `/var/www/html`

```
COPY app /var/www/html/
```

Damit der Web Server auch erreichbar ist, müssen wir den Port freigeben. Dies ist möglich mit den `EXPOSE` befehl.

> Der `EXPOSE` Befehl ist eigentlich nur ein Hinweis für Nutzer dieser Anwendung, dass der Container auf diesen Port erreichbar ist.

```
EXPOSE 80
```

Als letztes definieren wir welches Programm laufen soll, wenn wir den Container starten.

```
CMD ["nginx", "-g", "daemon off;"]
```

Nachdem wir die Schritte, die für unsere Anwendung nötig sind, in der Dockerfile definiert haben, können wir das Container Image bauen.

Dafür nutzen wir den `docker build` Befehl.

```
docker build -t 2048 .
```

Da nun das Image gebaut ist, können wir die App starten.

```
docker run --rm 2048
```

Port freigeben für `nginx`

```
docker run --rm -p 80:80 2048
```

Zurückkehren zum Workshop.

```
cd ..
```

### 3. Multi-container voting app

```
cd multi-container
```

![Architecture diagram](multi-container/architecture.png)

`docker-compose.yml` im Editor öffnen.

```
code docker-compose.yml
```

Komponenten definieren

```
services:
  vote:
    build: ./vote

  results:
    build: ./result

  worker:
    build:
      context: ./worker

  redis:
    image: redis:alpine

  db:
    image: postgres:9.4
```

Komponenten starten

```
docker-compose up
```

Alle komponenten werden richtig gestartet, allerdings wir können die Anwendung nicht erreichen. Dafür müssen wir Ports für unseren 2 Frontends freigeben.

```
  vote:
    build: ./vote
    ports:
      - 5000:80

  results:
    build: ./result
    ports:
      - 5001:80
```

Komponenten durch private Netze trennen.


```
services:
  vote:
    networks:
      - front
      - back

  result:
    networks:
      - front
      - back

  worker:
    networks:
      - back

  redis:
    networks:
      - back

  db:
    networks:
      - back

networks:
  front:
  back:
```

Damit die DB-Daten nicht weg sind, wenn wir die Container löschen, müssen wir der DB Komponente einen Volume anhängen, wo die Daten persistent gespeichert werden.

```
services:
  db:
    volumes:
      - "db-data:/var/lib/postgresql/data"

volumes:
  db-data:
```

Damit die Komponenten in der Reihenfolge der Abhängigkeiten gestartet werden, müssen wir diese definieren.

```
services:
  vote:
    depends_on:
      - redis
  result:
    depends_on:
      - db
  worker:
    depends_on:
      - redis
      - db

```

Während der Entwicklung ist es üblich den Quellcode sehr oft zu ändern, allerdings erfordert jede Änderung ein Neubau der Container. Um dies zu verhindern, kann der Quelltext auch als Volume eingebunden werden, sodass eine Änderung sofort in den Container sichtbar ist.

```
services:
  vote:
    volumes:
      - ./vote:/app

  result:
    volumes:
      - ./result:/app
```

Zurückkehren zum Workshop.

```
cd ..
```

### 4. Builder pattern

```
cd builder-pattern
```

Zunächst bauen wir das Image.

```
docker build -t webapp .
```

Und dann warten wir...

Sobald das Image gebaut wurde führen wir diese aus.

```
docker run --rm -it -p 80:8080 webapp
```

… und machen einige Requests mit curl

```
curl localhost
curl localhost/test
```

Die Anwendung ist sehr simpel. Sie gibt den angefragten Pfad zurück.

Wir schauen uns die Größe des Images.

```
docker image ls webapp
```

… und sehen, dass das Image knapp 800MB groß ist. Das bedeutet, dass wir beim Deployen erstmal 800MB herunterladen müssen. Das verlangsamt das ganze und nutzt unnötig Speicher.

Wir können das besser machen.

Als erstes können wir eine `alpine`-Variante benutzen die deutlich kleiner ist.

```
FROM golang:1.12.1-alpine
```
… und bauen das Image neu

```
docker build -t webapp .
```

Wir prüfen die Größe erneut.

```
docker image ls webapp
```

… und merken eine Reduzierung auf ca. 400MB.

Wir können aber noch mehr tun.

Wir teilen das Dockerfile in 2 Teile.

```
FROM golang:1.12.1-alpine AS builder
```

und

```
FROM scratch

COPY --from=builder /go/bin/webapp /bin/webapp

EXPOSE 8080

CMD ["/bin/webapp"]
```

Wir bauen das Image und prüfen die endgültige Größe.

```
docker build -t webapp .
docker image ls webapp
```

Das finale Image hat knapp 8MB oder 1% von der Ursprungsgröße.