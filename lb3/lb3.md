<div id="top"></div>

# LB3 - Gitlab-Installation automatisieren
Bitte nur Files im lb3/ Ordner im Branch main beachten!

## Inhaltsverzeichnis

Inhaltsverzeichnis für lb3.md von Ricardo Frei.

- [Einleitung](#Einleitung)
- [Service](#Service)
  - [Übersicht](#Übersicht)
- [Code](#Code)
  - [Vagrantconfig](#docker-compose-file)
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
![Übersicht Service](https://github.com/ricardofrei/M300_Services/blob/main/U%CC%88bersicht-Service_M300.png)

<p align="right">(<a href="#top">Zum Start</a>)</p>

## Code
### docker-compose-file

## Code-Beschreibung

| Code| Beschreibung|
| --------------| -----------------|
| Vagrant.configure("2") do  | Diese Zeile im Code beschreibt die API Version. |
| config.vm.box | Welches Betriebssystem wird installiert. |
| db.vm.network | Ich habe hier das Portforwarding eingerichtet und IP's gesetzt. |
| config.vm.synced_folder | Wird verwendet um Ordner in die VM zu sharen. In meinem Fall: Das aktuelle Verzeichnis ins /vagrant auf der Linux Maschine. Mit Ausnahme von .git/ |
| config.vm.provider :virtualbox do | Angabe zu Hypervisor und zugewiesenen Ressourcen. |
| db.vm.provision | Hier wird die Ausführung eines Shell-Skripts erlaubt. |
| sudo apt-get update | Hier werden die aktuellsten Pakete für geholt. |
| sudo apt-get install -y curl openssh-server ca-certificates | Die nötigen tools downloaden. -y für keine Userinteraktion. |
| debconf-set-selections | Hierbei können wir Auswahlen für im späteren GUI/Installationsfenster mitgeben. |
| DEBIAN_FRONTEND=noninteractive | Alle default Parameter aktzeptieren und mit der Installation fortfahren. |
| if [ ! -e /vagrant/ubuntu-bionic-gitlab-ce_12.9.3-ce.0_amd64.deb ]; then | Überprüfen, ob unser Gitlab-Installationsfile bereits auf dem Gerät vorhanden ist. |
| wget --content-disposition -O /vagrant/ubuntu-bionic-gitlab-ce_12.9.3-ce.0_amd64.deb https://packages.gitlab.com/gitlab/gitlab-ce/packages/ubuntu/bionic/gitlab-ce_12.9.3-ce.0_amd64.deb/download.deb | Man schreibt den Inhalt von https://packages.gitlab.com/gitlab/gitlab-ce/packages/ubuntu/bionic/gitlab-ce_12.9.3-ce.0_amd64.deb/download.deb in die lokale Datei /vagrant/ubuntu-bionic-gitlab-ce_12.9.3-ce.0_amd64.deb. |
| sudo dpkg -i /vagrant/ubuntu-bionic-gitlab-ce_12.9.3-ce.0_amd64.deb | Installation des .deb Files|
| sudo gitlab-ctl reconfigure | Vorgenommene Änderungen ins  /etc/gitlab/gitlab.rb schreiben |

<p align="right">(<a href="#top">Zum Start</a>)</p>

## Testing
> Die Installation mit `vagrant up` wurde auf meinem Gerät, einem MAC mit MacOS BigSur V.11.4 und auf dem Notebook von Herrn Imhasly, Windows 10, erfolgreich getestet. 

### Starten


<p align="right">(<a href="#top">Zum Start</a>)</p>

## Quellenverzeichnis

- [Markdownsystax](https://github.com/othneildrew/Best-README-Template/blob/master/README.md) 
- [Linux-Knowledge](https://wiki.ubuntuusers.de)
- [Postfixinstallation](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-postfix-on-ubuntu-20-04-de)
- [Vagrantsyntax](https://www.vagrantup.com/docs) 
- [Installationsguide](https://github.com/grafxflow/gitlab-ce-vagrant-ubuntu-18.04) 

<p align="right">(<a href="#top">Zum Start</a>)</p>
