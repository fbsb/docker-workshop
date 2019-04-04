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
