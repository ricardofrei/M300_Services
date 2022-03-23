<div id="top"></div>
# LB2 - Gitlab-Installation automatisieren

## Inhaltsverzeichnis

Inhaltsverzeichnis für README.md von Ricardo Frei.

- [Einleitung](#Einleitung)
- [Service](#Service)
  - [Übersicht](#Übersicht)
- [Code](#Code)
  - [Teil-1 - Vagrant (Ruby)](#teil-1)
  - [Teil-2 - Linux (bash)](#teil-2)
  - [Testing](#testing)
	  - [Starten](#Starten)
- [Quellenverzeichnis](#Quellenverzeichnis)

## Einleitung
Unser Ziel ist es, auf Basis von VirtualBox/Vagrant, einen Serverdienst zu automatisieren. 

Dabei waren wir in der Umsetzung frei, man konnte sich mit einem Klassenkameraden zusammenschliessen oder auf eigene Faust probieren. Ich habe mich dazu entschlossen, mit meinem Kollegen Sven Imhasly, gemeinsam zu arbeiten. Unter dem Moto, gemeinsam ist man stärker. 

Den Code, welchen ich erstellt habe, ist in meinem Github sauber abgelegt und mit verschiedenen nachvollziehbaren commits versehen. 

## Service
Den Service, welchen ich automatisiere, ist `Gitlab`. Ich bin zusammen mit meinem Klassenkameraden, Sven Imhasly, darauf gekommen.

Ich will, dass beim Starten mit Vagrant im Hintergrund ein Gitlab-Server installiert wird und unter [http://localhost:8080](http://localhost:8080) erreichbar ist. Dabei kann man sich mit dem Standarduser root anmelden können. 

Username | Password
---------|-----------
root     | 5iveL!fe --> beim Aufrufen von [http://localhost:8080](http://localhost:8080) wird man aufgefordert dieses Passwort zu ändern.  

Nach der Passwortänderung kommt man ins Gitlab-GUI und kann Projekte/Repositories erstellen. Man kann also nach der Installation das Gitlab sauber benutzen und damit arbeiten. 

### Übersicht
![Übersicht Service](https://github.com/ricardofrei/M300_Services/blob/main/U%CC%88bersicht-Service_M300.png)



<p align="right">(<a href="#top">Zum Start</a>)</p>

## Code
### Teil-1
```ruby
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|      #Config-Version festlegen. Wichtig für Rückwärts Kompatibilität.
  config.vm.box = "ubuntu/bionic64"                         #Angeben, welche Box verwendet wird.
  config.vm.hostname = "ricardo.git"                        #In-VM, Konfigurationen.

  if Vagrant.has_plugin?("vagrant-vbguest")                 #VirtualBox Guest Additions deaktivieren.
    config.vbguest.auto_update = false
  end

  config.vm.network "forwarded_port", guest: 80, host: 8080   #Portforwarding von Localhost Port 8080 auf VM Port 80
  config.vm.network "forwarded_port", guest: 22, host: 8022   #Portforwarding von Localhost Port 8022 auf VM Port 22

  config.vm.network "private_network", ip: "192.168.56.45"  <b>#IP-Adresse konfigurieren
  config.vm.synced_folder ".", "/vagrant", type: "rsync", rsync__exclude: [".git/"]         #Das aktuelle Verzeichnis ins /vafrant auf der Linux Maschine. Mit Ausnahme von .git/

  config.vm.provider "virtualbox" do |vb|                   #Virtualbox Specs anpassen.
    vb.name = "ricardo.local"                               #Name der VirtualBox
    vb.memory = "4096"                                      #RAM angeben.
    vb.cpus = "2" #Cpus festlegen
  end
``` 
### Teil-2
```shell
  config.vm.provision "shell", inline:  <<-SHELL
    sudo apt-get update                                     #Update/neue Pakete herunterladen. Damit später mit install ausgeführt werden kann.
    sudo apt-get install -y curl openssh-server ca-certificates       #-y für keine Userinteraktion. Openssh-Server für SSH Connection. Ca-Certificates ist ein Test, für Zertifikate für Gitlab.

    debconf-set-selections <<< "postfix postfix/mailname string $HOSTNAME"                    #Mit debconf-set-selections kann man allfällige Fragen, die im Wizard auftauchen beantworten
    debconf-set-selections <<< "postfix postfix/main_mailer_type string 'Internet Site'"
    DEBIAN_FRONTEND=noninteractive sudo apt-get install -y postfix         #noninteractive: Defaultvalue für jede Frage. DEBIAN_FRONTEND Ugebungsvariabel.
    
    if [ ! -e /vagrant/ubuntu-bionic-gitlab-ce_12.9.3-ce.0_amd64.deb ]; then         #Wenn die Bedingung nicht erfüllt ist, das Paket von https://packages.gitlab.com herunterladen.
        wget --content-disposition -O /vagrant/ubuntu-bionic-gitlab-ce_12.9.3-ce.0_amd64.deb https://packages.gitlab.com/gitlab/gitlab-ce/packages/ubuntu/bionic/gitlab-ce_12.9.3-ce.0_amd64.deb/download.deb #Er schreibt den Inhalt von https://packages.gitlab.com/gitlab/gitlab-ce/packages/ubuntu/bionic/gitlab-ce_12.9.3-ce.0_amd64.deb/download.deb in die lokale Datei /vagrant/ubuntu-bionic-gitlab-ce_12.9.3-ce.0_amd64.deb.
    fi

    sudo dpkg -i /vagrant/ubuntu-bionic-gitlab-ce_12.9.3-ce.0_amd64.deb         #installieren des .deb Paketes.

    sudo gitlab-ctl reconfigure         #Veränderungen in der /etc/gitlab/gitlab.rb schreiben.

  SHELL
end
```
<p align="right">(<a href="#top">Zum Start</a>)</p>

### Testing
> Die Installation mit `vagrant up` wurde auf meinem Gerät, einem MAC mit MacOS BigSur V.11.4 und auf dem Notebook von Herrn Imhasly, Windows 10, erfolgreich getestet. 

#### Starten

1. Herunterladen der Dateien und in dem Verzeichnis, welchem das `"Vagranfile"` liegt und Punkt 2. ausführen.
2. `vagrant up`
3. Sobald die VM aufgestartet ist, respektive das CMD Fenster mit der Installation fertig ist, auf die Website: [http://localhost:8080](http://localhost:8080) verbinden.
4. Passwort neu setzten. (Muss 8 Zeichen lang sein!)
5. Anmelden mit Username. `root` und zuvor gesetztem Passwort. 
6. Feel-Free Projekte usw. zu erstellen.


<p align="right">(<a href="#top">Zum Start</a>)</p>



## Quellenverzeichnis

- [Markdownsystax](https://github.com/othneildrew/Best-README-Template/blob/master/README.md) 
- [Linux-Knowledge](https://wiki.ubuntuusers.de)
- [Postfixinstallation](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-postfix-on-ubuntu-20-04-de)
- [Vagrantsyntax](https://www.vagrantup.com/docs) 
- [Installationsguide](https://github.com/grafxflow/gitlab-ce-vagrant-ubuntu-18.04) 

<p align="right">(<a href="#top">Zum Start</a>)</p>

## Git-Historie
<p>
  Zu Beginn des Moduls habe ich an meinem Code in einem anderen Ordner gearbeitet, als eigentlich vorgesehen war. Als ich die Anforderung gesehen habe, dass ein Ordner lb2 erstellt werden soll, habe ich das File mit dem Linux Befehl mv verschoben, anstelle von git mv. <br><br>
  Ausserdem habe ich mich im Verlauf des Moduls dazu entschieden, mit zwei branches zu arbeiten. Sprich ich code, commite und pushe alles in meinen `master` branch. Als ich sicher war, dass meine Änderungen gut sind und laufen, habe ich die Files dann mittels einem merge in meinen `main` branch gebracht.
</p>

<p align="right">(<a href="#top">Zum Start</a>)</p>