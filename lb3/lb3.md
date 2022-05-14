<div id="top"></div>

# LB3 - Gitlab-Installation automatisieren
Bitte nur Files im lb3/ Ordner im Branch main beachten!

## Inhaltsverzeichnis

Inhaltsverzeichnis für lb3.md von Ricardo Frei.

- [Einleitung](#Einleitung)
- [Service](#Service)
  - [Übersicht](#Übersicht)
- [Code](#Code)
  - [docker-compose](#docker-compose-file)
- [Codebeschreibung](#code-beschreibung)
- [Testing](#testing)
	- [Starten](#Starten)
- [Quellenverzeichnis](#Quellenverzeichnis)

## Einleitung
Unser Ziel ist es, auf Basis von Docker-Compose, einen Serverdienst zu automatisieren. 

Dabei waren wir in der Umsetzung frei, man konnte sich mit einem Klassenkameraden zusammenschliessen oder auf eigene Faust probieren. Ich habe mich dazu entschlossen, mit meinen Kollegen Sven Imhasly und Marks Zgraggen, gemeinsam zu arbeiten. Unter dem Moto, gemeinsam ist man stärker.

Den Code, welchen ich erstellt habe, ist in meinem Github sauber abgelegt und mit verschiedenen nachvollziehbaren commits versehen.

## Service
Die Services, welche ich automatisiere, sind Gitlab, Watchtower und Bytemark. Ich bin zusammen mit meinem Klassenkameraden, Sven Imhasly, darauf gekommen.

Wir wollen, dass beim Starten mit vagrant up im Hintergrund ein Gitlab-Server installiert wird und unter http://localhost:8080 erreichbar ist. 

Watchtower ist auf unserem Server zuständig dafür, dass regelmässig geprüft wird, dass die Images up-to-date sind. 

Der Mail-Container erlaubt es Gitlab direkt Mails zu versenden. Er erstellt ein SMTP-Host welcher unter dem Hostname: mail erreichbar ist. 

### Übersicht
![Übersicht Service](https://github.com/ricardofrei/M300_Services/blob/main/lb3/U%CC%88bersicht-Services_M300.png)

<p align="right">(<a href="#top">Zum Start</a>)</p>

## Code
### docker-compose-file
```yaml
version: '3'

networks:
  frontend:

volumes:
  vol-gitlab-config:
  vol-gitlab-logs:
  vol-gitlab-data:
```
  - Docker-Compose Version 
  - Netzwerk
  - Volumes 

```yaml
services:
  watchtower:
    image: v2tec/watchtower:latest
    command: --cleanup --schedule "0 0 0 * * *"
    restart: always
    networks:
      - frontend
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock

  gitlab:
    image: gitlab/gitlab-ce:latest
    restart: always
    hostname: "localhost"
    ports:
      - "8082:22"
      - "80:80"
    networks:
      - frontend
    volumes:
      - vol-gitlab-config:/etc/gitlab
      - vol-gitlab-logs:/var/log/gitlab
      - vol-gitlab-data:/var/opt/gitlab
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://localhost'
        letsencrypt['enable'] = false
        gitlab_rails['gitlab_email_enabled'] = true
        gitlab_rails['gitlab_email_display_name'] = 'GitLab'
        gitlab_rails['smtp_enable'] = true
        gitlab_rails['smtp_address'] = "mail"
        gitlab_rails['smtp_port'] = 25
        gitlab_rails['smtp_tls'] = false
        gitlab_rails['backup_keep_time'] = 604800

  mail:
    image: bytemark/smtp
    restart: always
    networks:
      - frontend
  
  gitlab-runner:
    image: gitlab/gitlab-runner:alpine
    restart: always
    depends_on:
    - gitlab
    volumes:
    - ./config/gitlab-runner:/etc/gitlab-runner
    - /var/run/docker.sock:/var/run/docker.sock
```
  - Services: 
    - Watchtower
    - Gitlab
    - mail
    - gitlab-runner



## Code-Beschreibung

| Code| Beschreibung|
| --------------| -----------------|
| version: '3'  | Version von Docker-Compose |
| networks:
  frontend: | Ich definiere hier ein Netzwerk `frontend`, damit sich alles Container erreichen können. |
| volumes:
  vol-gitlab-config:
  vol-gitlab-logs:
  vol-gitlab-data: | Ich erstelle hier 3 Volumes, um sie später im Container einzubinden. In diesen Volumes könnnen beispielsweise Configs liegen. |
| services: | Initialisierung der zu installierenden Services. |
| image: | Angabe zum Image. Vorzugsweise von [Dockerhub](https://hub.docker.com/) |
| restart: | Wir haben `always` gesetzt, damit es immer neustartet, sobald der Container einen exit-code hat. |
| hostname: | Hostname für Container. |
| ports: | Vagrant-VM:Docker-CT. Also haben wir Port 80 der Vagrant-VM auf den Container gebridged. |
| networks: | Angabe in welchem Netzwerk sich der Container befindet. |
| volumes: | Welche Volumes werden unter welchem Pfad im Container gemappt. |
| environment: | Konfig für Container. |
| GITLAB_OMNIBUS_CONFIG: | Gitlab-interne Konfigurationen. |

<p align="right">(<a href="#top">Zum Start</a>)</p>

## Testing
> Die Installation mit `docker-compose up` wurde auf meinem Gerät, einem MAC mit MacOS BigSur V.11.4, auf meinem Home-PC Windows 21h2 und auf dem Notebook von Herrn Imhasly, Windows 1909, erfolgreich getestet. 

### Starten
1. Herunterladen der Dateien und in dem Verzeichnis, welchem das `"docker-compose.yaml"` file liegt und Punkt 2. ausführen.
2. `docker-compose up`
3. Sobald die Container aufgestartet sind, respektive das CMD Fenster mit der Installation fertig ist, auf die Website: [http://localhost:8080](http://localhost:8080) verbinden.
4. Sie sollten nun auf dem Web-Interface von Gitlab sein. 
5. Im Hintergrund laufen nun auch Watchtower, SMTP und Gitlab-Runner.

<p align="right">(<a href="#top">Zum Start</a>)</p>

## Quellenverzeichnis

- [Markdownsystax](https://github.com/othneildrew/Best-README-Template/blob/master/README.md) 
- [Linux-Knowledge](https://wiki.ubuntuusers.de)
- [Install-Guide](https://github.com/BytemarkHosting/configs-gitlab-docker/blob)
- [Inspiration](https://github.com/containrrr/watchtower/blob/main/docker-compose.yml)

<p align="right">(<a href="#top">Zum Start</a>)</p>
