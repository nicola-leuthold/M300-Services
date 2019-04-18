***  
***
# Dokumentation LB 1
**Zum Modul 300 von Nicola Leuthold mit Unterstüzung von Marius Rutz**
***  
## Inhaltsverzeichnis

- [Dokumentation LB 1](#dokumentation-lb-1)
  - [Inhaltsverzeichnis](#inhaltsverzeichnis)
  - [Persönlicher Wissensstand](#pers%C3%B6nlicher-wissensstand)
  - [Lernumgebung](#lernumgebung)
  - [Vorgefertigte VM mit Vagrant aufsetzen](#vorgefertigte-vm-mit-vagrant-aufsetzen)
  - [Vagrant Ubuntu VM mit Firewall und Webserver](#vagrant-ubuntu-vm-mit-firewall-und-webserver)
  - [Wissenzuwachs](#wissenzuwachs)
  - [Reflexion](#reflexion)
  - [Quellen](#quellen)

***
## Persönlicher Wissensstand

**Linux**  
In Modulen an der TBZ eher selten genutzt.
Im Geschäfft verwenden wir Windows.  
**Virtualisierung**  
VMware im Geschäfft und in der Schule verwendet. In der Schule in diversen Modulen genutzt.  
**Vagrant**  
Noch unbekannt  
**Versionsverwaltung / Git**  
Keine.  
**Systemsicherheit**  
Wenig Erfahrung im Konfigurieren von Firewalls. 
**Mark Down**
Noch unbekannt

***
## Lernumgebung
**Git**  
Gemäss Anleitung im M300 Repository:  
*Github*  
  1. Auf www.github.com ein Benutzerkonto erstellen (Angabe von Username, E-Mail und Passwort)
  2. E-Mail zur Verifizierung des Kontos bestätigen und anschliessend auf GitHub anmelden

*Repository*
  1. New Repository auswählen
  2. Name: M300
  3. Public
  4. Initialize this repository with a README auswählen
  5. Create a repository

*Git Client*  
1. Git-2.20.1-64-bit.exe als Administrator ausführen.  
2. Standardwerte wurden während der Installation verwenden.  
3. Öffnen und Konfiguration mit:  
 >$ git config --global user.name "<nicola-leuthold>"  
 >$ git config --global user.email "<nicola-leuthold@bluewin.ch>"  

*SSH Key*  
1. Im Terminal:  
  >ssh-keygen -t rsa -b 4096 -C "nicola-leuthold@bluewin.ch"  
  >Enter a file in which to save the key (~/.ssh/id_rsa): [Press enter]  
  >Enter passphrase (empty for no passphrase): []  
  >Enter same passphrase again: []

2. *%HOME%/.ssh/id_rsa.pub mit Notepad öffnen und Schlüssel kopieren  
3. Auf Github unter Settings->SSH and GPG keys angeben

*Repository klonen*  
Modulrepo:  
  > git clone https://github.com/mc-b/M300  
  > cd M300  
  > git pull  
  > git status
     
Meinrepo:  
  >git clone https://github.com/nicola-leuthold/M300-Services/  
  > git pull --> Um zu aktualisieren  
  > git status --> Um  Status der lokalen Kopie anzuzeigen  

**Virtualbox**  
Virtualbox ist eine Opensource Virtualisierungssoftware.  
Für die Verwendung im Modul reicht die Installation mit Standardwerten.

**Vagrant**  
Vagrant ermöglicht es, dass ich in Virtualbox automatisiert eine VM mit Service installieren kann.  
Für die Verwendung im Modul reicht die Standardinstallation.  

*Nützliche Befehle:*  

Umgebung initialisieren und Vagrantfile erstellen  
>vagrant init  

VM erzeugen und konfigurieren nach Vagrantfile  
>vagrant up

SSH Verbindung  
>vagrant ssh  

Status anzeigen  
>vagrant status  

VM stoppen
>vagrant halt  

VM zerstören  
>vagrant destroy


**Visualstudio Code**  
Visualstudio Code ist ein Editor von Microsoft. Es ist möglich ein Repository darin zu öffnen und Änderungen gleich zu pushen.  
Für die Verwendung im Modul reicht die Standardinstallation.  
Zusätzlich instalierte folgende 3 vorgegebenen Extensions:

* Markdown All in One
* Vagrant Extension
* vscode-pdf Extension  

Sie ermöglichen das einfachere bearbeiten von dieser Dokumentation und dem Vagrant file.

Damit beim Dateien mit den Endungen .git / .svn / .hg / .vagrant / .DS_store nicht in das Repository hinaufgeladen werden, habe ich im setting.json file folgenden code eingefügt:

>   // Konfiguriert die Globmuster zum Ausschließen von Dateien und Ordnern.  
> "files.exclude": {  
>   "**/.git": true,  
>   "**/.svn": true,  
>   "**/.hg": true,  
>   "**/.vagrant": true,  
>   "**/.DS_Store": true  
> },  

***
##  Vorgefertigte VM mit Vagrant aufsetzen

Aus dem M300 Repository, setzte ich automatisiert eine Ubuntu VM mit Apache2 auf.
Um diese Vm zu starten muss ich ins Verzeichnis /M300/vagrant/web wechseln mit dem Befehl "CD" und anschliessend "vagrant up" ausführen. 
Die VM wird daraufhin installiert und in Virtualbox angezeigt.  
![M300](pictures/web)

***
## Vagrant Ubuntu VM mit Firewall und Webserver

**Netzwerkplan**  

![M300](pictures/netzwerk)

**Konfiguration**  
In der Kopie von meinem Repository, erstellte ich mit vagrant init ein Vagrantfile.
Um die Firewall und den Webserver zu installieren und konfigurieren habe ich es folgendermassen angepasst.  
Es wird ubuntu xenial64 als Basis verwendet.
  >Vagrant.configure("2") do |config|  
  >config.vm.box = "ubuntu/xenial64"

Es wird ein privates Netzerk erstellt. (Host Only) Die IP lautet 10.71.13.12.  
Für mich definierte IP wird konfiguriert.  
  >config.vm.network "private_network", ip: "10.71.13.12"

Es wird aus dem aktuellen Ordner ein shared Folder gemacht, der unter /var/www/html eingehängt wird.
Da dies der Ordner ist in dem der Webserver das html Dokument bezieht, kann ich im vornherein ein index file in den Ordner einfügen, welches vom Webserver verwendet werden wird.
  >config.vm.synced_folder ".", "/var/www/html"

Virtualbox wird als VMprovider genutzt
  >config.vm.provider "virtualbox" do |vb|

Es werden 4 GB Ram zur Verfügung gestellt-  
  >vb.memory = "4024"  
  >end  

Installieren von Apache2 und ufw. ufw erlaubt Ports von ssh und http und IPs vom 24-Netz 10.71.13.0
 >  config.vm.provision "shell", inline: <<-SHELL  
 >  apt-get update  
 > apt-get install -y apache2  
 >apt-get update  
 >apt-get install -y ufw  
 >ufw allow ssh  
 >ufw allow http  
 >ufw allow from 10.71.13.0/24  

Da ein Neustart unerwünscht ist, wird die Firewall manuell eingeschaltet. yes Y | führt dazu, dass die Abrage beim enablen mit Y beantwrtet wird.
  >   yes Y | ufw enable  
  > SHELL  
  >end  

**Sicherheit**  
Der Zugriff über SSH ist standardmässig möglich.
Beim automatisierten Aufsetzen, wird ein neuer SSH-Key generiert.  
Die Firewall ist erlaubt den Zugriff über SSH und HTTP. Zusätzlich habe ich festegelegt, dass nur IPs aus dem Subnetz 10.71.13.0 zugreifen dürfen.

**Starten**  
Die VM kann mit vagrant up gestartet werden, wenn man die Konsole im Ordner mit dem Vagrantfile ausführt.  
Danach wird durch vagrant ssh eine Verbindung hergestellt.

**Testen**  
*Aufsetzen*
Im Ordner Vagrant folgenden Befehl ausführen und die VM wird verbunden.
>vagrant up  

![M300](pictures/virtualbox)  

Wie man sieht wird die VM mit 4 GB Ram in Virtualbox angezeigt.  

*Verbinden*
>vagrant ssh  

![M300](pictures/SSH)  
Die Verbindung funktioniert.

*Firewall*  

>sufo ufw status  

![M300](pictures/ufw)  
Die Firewall ist aktive und die gewünschten Ports sind geöffnet.

*Apache2*

>service apache2 status  

![M300](pictures/apache)  
Der Service läuft.  
![M300](pictures/index)  
Vom Client ist der Webserver über HTTP erreichbar und zeigt das im Vagrant Ordner abgelegte index.html file an.
***
## Wissenzuwachs

**Linux**  
Ich habe die Tool-Umgebung auf Ubuntu Linux installiert und verwendet.
Dadurch konnte ich mich üben im  navigieren über die Konsole.
Die Befehle zur Installation von Software kannte ich bereits.  

**Virtualisierung**  
Die verwendete Virtualisierungssoftware und konfiguration kannte ich .bereits.  

**Vagrant**  
Ich lernte eine Vagrantumgebung aufzusetzen und ein Vagrantfile zu konfigurieren, aus welchem ich dann automatisiert eine VM aufsetzen lassen kann.    

**Versionsverwaltung / Git**  
Ich lernte ein Repository zu Erstellen, Klonen und lokal gemachte Änderungen zu commiten und pushen.

**Systemsicherheit**  
Ich konnte per SSH die Verbindung mit der VM herstellen.  
Auf der Firewall konnte ich spezifische Ports definieren um gewünschte Services zuzulassen.  
**Mark Down**  
Ich lernte eine Dokumentation mit Mark Down zu erstellen.
***  
## Reflexion/Rückblick

Leider konnte ich nur an der ersten Lektion dieses Modules teilnehmen, da ich anschliessend 6 Wochen die Schule aufgrund eines Unfalls nicht mehr besuchen konnte.
Jedoch wurde mir die Chance gegeben dieses Modul Zuhause nachzuholen. Dies versuchte ich so gut wie möglich alleine zuerst, zudem hatte ich die Unterstüzung meiner Freunde, Marius Rutz und Yanik Vonderschmitt.
Ich wollte diese LB nicht einfach gemacht haben sonder diese auch verstanden haben. Hierbei konnte mit Marius helfen indem wir einen Abend die LB1 zusammen bearbeitet haben und er mir dabei 
Vagrant, Visualstudio und Virtualbox erklärte. Somit habe ich die Arbeit auch verstanden und nicht nur kopiert. Mark Down erschwerte mir die eschicchte auch ein wenig, 
hier konnte mir YAnik aushelfen. Dies ist auch weniger eine Reflexion und mehr ein Rückblicke und ich bin dankbar für die Unterstüzung und das ich das Modul trotz des Unfalls abschliessend kann.

## Quellen

* <https://github.com/mc-b/M300>  
* <https://bscw.tbz.ch/bscw/bscw.cgi/20887767>

***  
***