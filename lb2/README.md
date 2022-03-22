# LB2 - Gitlab-Installation automatisieren

## Inhaltsverzeichnis

Inhaltsverzeichnis für README.md von Ricardo Frei.

- [Einleitung](#Einleitung)
- [Service](#Service)
  - [Übersicht](#Übersicht)
- [Code](#Code)
	- [Starten](#Start)
- [Testing](#Testing)
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

### Start
1. Herunterladen der Dateien und in dem Verzeichnis, welchem das `"Vagranfile"` liegt Punkt 2. ausführen.
2. `vagrant up`
3. Auf die Website: [http://localhost:8080](http://localhost:8080) verbinden.
4. Passwort neu setzten. (Muss 8 Zeichen lang sein!)
5. Anmelden mit Username. `root` und zuvor gesetztem Passwort. 
6. Feel-Free Projekte usw. zu erstellen. 

## Code

<pre><code>
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|      <b>#Config-Version festlegen. Wichtig für Rückwärts Kompatibilität.</b>
  config.vm.box = "ubuntu/bionic64"                         <b>#Angeben, welche Box verwendet wird.</b>
  config.vm.hostname = "ricardo.git"                        <b>#In-VM, Konfigurationen.</b>

  if Vagrant.has_plugin?("vagrant-vbguest")                 <b>#VirtualBox Guest Additions deaktivieren.</b>
    config.vbguest.auto_update = false
  end

  config.vm.network "forwarded_port", guest: 80, host: 8080 <b>#Portforwarding von Localhost Port 8080 auf VM Port 80</b>
  config.vm.network "forwarded_port", guest: 22, host: 8022 <b>#Portforwarding von Localhost Port 8022 auf VM Port 22</b>

  config.vm.network "private_network", ip: "192.168.56.45"  <b>#IP-Adresse konfigurieren</b>
  config.vm.synced_folder ".", "/vagrant", type: "rsync", rsync__exclude: [".git/"]         <b>#Das aktuelle Verzeichnis ins /vafrant auf der Linux Maschine. Mit Ausnahme von .git/</b>

  config.vm.provider "virtualbox" do |vb|                   <b>#Virtualbox Specs anpassen.</b>
    vb.name = "ricardo.local"                               <b>#Name der VirtualBox</b>
    vb.memory = "4096"                                      <b>#RAM angeben.</b>
    vb.cpus = "2" #Cpus festlegen
  end

  config.vm.provision "shell", inline: <<-SHELL
    sudo apt-get update                                     <b>#Update/neue Pakete herunterladen. Damit später mit install ausgeführt werden kann.</b>
    sudo apt-get install -y curl openssh-server ca-certificates       <b>#-y für keine Userinteraktion. Openssh-Server für SSH Connection. Ca-Certificates ist ein Test, für Zertifikate für Gitlab.</b>

    debconf-set-selections <<< "postfix postfix/mailname string $HOSTNAME"                    <b>Mit debconf-set-selections kann man allfällige Fragen, die im Wizard auftauchen beantworten</b>
    debconf-set-selections <<< "postfix postfix/main_mailer_type string 'Internet Site'"
    DEBIAN_FRONTEND=noninteractive sudo apt-get install -y postfix          <b>#noninteractive: Defaultvalue für jede Frage. DEBIAN_FRONTEND Ugebungsvariabel.</b>
    
    if [ ! -e /vagrant/ubuntu-bionic-gitlab-ce_12.9.3-ce.0_amd64.deb ]; then          <b>#Wenn die Bedingung nicht erfüllt ist, das Paket von https://packages.gitlab.com herunterladen.</b>
        wget --content-disposition -O /vagrant/ubuntu-bionic-gitlab-ce_12.9.3-ce.0_amd64.deb https://packages.gitlab.com/gitlab/gitlab-ce/packages/ubuntu/bionic/gitlab-ce_12.9.3-ce.0_amd64.deb/download.deb #Er schreibt den Inhalt von https://packages.gitlab.com/gitlab/gitlab-ce/packages/ubuntu/bionic/gitlab-ce_12.9.3-ce.0_amd64.deb/download.deb in die lokale Datei /vagrant/ubuntu-bionic-gitlab-ce_12.9.3-ce.0_amd64.deb.
    fi

    sudo dpkg -i /vagrant/ubuntu-bionic-gitlab-ce_12.9.3-ce.0_amd64.deb         <b>#installieren des .deb Paketes.</b>

    sudo gitlab-ctl reconfigure         <b>#Veränderungen in der /etc/gitlab/gitlab.rb schreiben.</b>

  SHELL
end
</code></pre>

### Testing
> Die Installation mit `vagrant up` wurde auf meinem Gerät, einem MAC mit MacOS BigSur V.11.4 und auf dem Notebook von Herrn Imhasly, Windows 10, erfolgreich getestet. 



## Quellenverzeichnis