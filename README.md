M300 - LB3 - Kriterium 2
===================

Umgebung auf eigenem Notebook eingerichtet und funktionsfähig

### GitHub oder Gitlab-Account ist erstellt
***

1. Webseite öffnen: https://github.com/join?source=header-home
2. Username, Email Adrese und Passwort eingeben
3. Account Verifizieren und Create Accoutn klicken
4. Choose your plan -> Gratis oder bezahlt
5. Persönliche Informationen eingeben(Erfahrung, Nutzung, Berufung und Interesse)
6. Verifizierung von E-Mail Adresse
Mein GitHub-Account: https://github.com/ViV0rtex/

### Git-Client wurde verwendet
***

Siehe Installation

### Dokumentation ist als Mark-Down vorhanden
***

Siehe https://github.com/ViV0rtex/M300-Services/LB3/README.md

### Markdown-Editor ausgewählt und eingerichtet
Siehe Sublime Installation in K1->README.md

### Persönlicher Wissenstand im Bezug auf die wichtigsten Themen ist dokumentiert (Containerisierung / Docker, Microservices)
***

		Containerisierung:
			Das Ziel eines Containers ist das Isolieren einer Applikation/Dienst und dessen Abhängigkeiten in einer eigenständigen Einheit die überall laufen kann.
			Container Maschinen nutzen alle den Kernel des Hosts und können nach belieben von anderen Maschinen isoliert werden.
			Container vernichten die Abhängigkeit von physischer Hardware, und erlauben es die Rechenressourcen effizienter zu nutzen, bezogen auf Energie Konsum und Kosten Effektivität.
			Container belaufen auf  dem OS Level/Ring
		
			
		Microservices- aka Mircroservice Infrastruktur -Ist eine Software Architektur von einer Sammlung an Diensten, welche folgende Eigenschaften besitzen:
		-Einzeln
		Service Oriented Archtiecture(SOA)
		Software development Technique
    
M300 - LB3 - Kriterium 3
===================

### Bestehenden Docker-Container kombinieren
***
Dies kann man mit Docker Compose zu stande bringen, wie ich es hier zeigen werde.
Dazu benutze ich ein web Service der den Port 5000 exposed und darauf beläuft.(Standard Flask web server Port)
Ausserdem benutze ich redis um Docker compose zu testen.

Docker-Compose Aufbau:
  Dockerfile:
    FROM python:3.4-alpine
    ADD . /code
    WORKDIR /code
    RUN pip install -r requirements.txt
    CMD ["python", "app.py"]

  app.py:
    import time

    import redis
    from flask import Flask


    app = Flask(__name__)
    cache = redis.Redis(host='redis', port=6379)


    def get_hit_count():
        retries = 5
        while True:
            try:
                return cache.incr('hits')
            except redis.exceptions.ConnectionError as exc:
                if retries == 0:
                    raise exc
                retries -= 1
                time.sleep(0.5)


    @app.route('/')
    def hello():
        count = get_hit_count()
        return 'Hello World! I have been seen {} times.\n'.format(count)

    if __name__ == "__main__":
        app.run(host="0.0.0.0", debug=True)
        
  requirements.txt:
    flask
    redis
    
  docker-compose.yml:
    version: '3'
    services:
      web:
        build: .
        ports:
         - "5000:5000"
         volumes:
         - .:/c2
      redis:
        image: "redis:alpine"

Applikation bauen und laufen lassen:
  docker-compose up
Auf der lokalen Maschine zu http://localhost:5000 navigieren

Volume einrichten: 

$ docker run --network=host -it --name c2 -v /data --rm busybox

# Im Container
$ cd /data
$ mkdir t1
$ echo "Test" >t1/test.txt

# CTRL + P + CTRL + Q
$ docker inspect c2

# Nach Mounts suchen, z.B. 
        "Mounts":
        {
            "Type": "volume",
            "Name": "ea99634523a0aa3d6fbf7ee02c491029170d7105f9c5404760a71e046ad55c67",
            "Source": "/var/lib/docker/volumes/ea99634523a0aa3d6fbf7ee02c491029170d7105f9c5404760a71e046ad55c67/_data",
            "Destination": "/data",
            "Driver": "local",
            "Mode": "",
            "RW": true,
            "Propagation": ""
        }

# Datei ausgeben (auf Host)
$ sudo cat /var/lib/docker/volumes/ea99634523a0aa3d6fbf7ee02c491029170d7105f9c5404760a71e046ad55c67/_data/t1/test.txt

docker volumes create

  VOLUME /var/lib/mysql # Dockerfile Referenz 

Datenverzeichnis /var/lib/mysql vom Container auf dem Host einhängen (mount):

    $ docker run -d -p 3306:3306  -v ~/data/mysql:/var/lib/mysql --name mysql --rm mysql
    
    # Datenverzeichnis
    $ ls -l ~/data/mysql
Einzelne Datei auf dem Host einhängen:

    $ docker run --rm -it -v ~/.bash_history:/root/.bash_history ubuntu /bin/bash

### Kennt die Docker spezifischen Befehle
***

| Befehl                | Erklärung                                                                   |
|-----------------------|-----------------------------------------------------------------------------|
| docker dockerd        | Startet den Docker-Daemon													  |
| docker info           | Zeigt Systeminformationen an  										      |
| docker inspect        | Zeigt Infromationen von einem Container oder Image an                       |
| docker build          | Erstellt ein Image von einem Dockerfile                                     |
| docker commit         | Erstellt ein Image von den Änderung eines Containers                        |
| docker history        | Zeigt die Vorgeschichte/Verlauf von einem Image an                          |
|docker run             | Starten neuer Container (-d detach läuft im Hintergrund, -it interactive tty|
|docker service create  | Create a new services                                                       |
|docker ps              | List Containers                                                             |
|docker version         | Versions info anzeigen                                                      |
|docker restart         | Neustart                                                                    |
|docker create          | Neuen contrianer erstellen                                                  |
|docker search          | Dockerhub nach Images suchen                                                |
|docker pull            | Lädt Image aus einer Repository herunter                                  |
|-----------------------|-----------------------------------------------------------------------------|


### Funktionsweise getestet inkl. Dokumentation der Testfälle
***

| Testfall                            | Erwartetes Resultat                   | Resultat                              |
|-------------------------------------|---------------------------------------|---------------------------------------|
| Webserer erreichbar unter Localhost | Hello World! I have been seen x times | Hello World! I have been seen 2 times |


M300 - LB3 - Kriterium 4
===================

### Service-Überwachung ist implementiert
***
    Mit diesem Befehl werden auch gleich Benachrichtigungen erstellt/gesendet
    $ docker run -d --name cadvisor -v /:/rootfs:ro -v /var/run:/var/run:rw -v /sys:/sys:ro -v /var/lib/docker/:/var/lib/docker:ro -p 8080:8080 google/cadvisor:latest

### Mind. 3 Aspekte der Container-Absicherung sind berücksichtigt
***
1. Container laufen auf in einer V M oder auf einem dedizierten Host, um zu vermeiden, dass andere benutzer oder Services angegriffen werden können(Schadensverringerung)
2. Images werden mit einem low priviledge User erstellt und nicht als root
3. Pattern Based IPS einsetzen


### Reflexion
***
Markdown gefällt mir nicht und ist für mich unnützlich, weil ich es viel einfacher und schneller mit Microsot Word machen könnte. Jedoch denke ich es ist sehr nützlich für Softwareentwickler oder andere Systemtechniker die mehr mit Applikationen und Entwicklung zu tun haben.
Ich musste sehr viel recherchieren um überhaupt mit der Arbeit zu recht zu kommen, jedoch hat sich das recherchieren gelohnt und ich konnte auf diesem Wissen aufbauen und mehr und mehr über Docker erfahren.
Docker finde ich extrem nützlich, und werde es wahrscheinlich weiterhin verwenden. Es gibt unmengen an Möglichkeiten mit Docker welche sehr nützlich sein könnten und interessant wären zum testen.