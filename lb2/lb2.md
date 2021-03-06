# M300-LB2 Dokumentation

---

# Inhaltsverzeichnis

- [Einführung](#einführung)
- [Grafische Übersicht des Services](#grafische)
- [Vagrantfile](#vagrantfile)
- [Bootstrap.sh](#bootstrap)
- [Service Anwendung & testen](#anwendung)
- [Quellenangaben](#quellenangaben)

---

# Einführung

Wir haben im Modul 300 den Auftrag bekommen, ein eigenes Projekt mit Vagrant zu gestallten.
Mein Projekt ist, dass man ein MySQL automatisch einrichten lassen kann. Damit es angenehmer ist die MySQL-Datenbank und ihre User zu verwalten und zu konfigurieren habe ich mich entschieden noch einen GUI mit einzubinden. Zu diesem Zweck verwenden ich das webbasierte phpmyadmin.

---
<a name="grafische"></a>
# Grafische Übersicht des Services

![Grafische-Übersicht](Bilder/grafischeuebersicht.jpg)

---


# Vagrantfile

Hier unten finden Sie noch mein Vagrantfile:

    Vagrant.configure("2") do |config|
    
      # General Vagrant VM configuration
      config.vm.box = "ubuntu/bionic64"

      config.vm.define "db-server" do |db|
        db.vm.network :forwarded_port, guest: 3306, host: 3306
        db.vm.network :forwarded_port, guest: 80, host: 3306
        db.vm.provision "shell", path: "bootstrap.sh"
      end

      # Adjustment for VM
      config.vm.provider :virtualbox do |vb|
        vb.customize [
            'modifyvm', :id,
            '--natdnshostresolver1', 'on',
            '--memory', '1024',
            '--cpus', '4'
        ] 
      end
    end

| Code| Beschreibung|
| --------------| -----------------|
| Vagrant.configure("2") do config | Diese Zeile im Code beschreibt die API Ausführung, hierbei die Menge 2, vom Vagrantfile. In diesem Block beschreibe ich die VM Konfigurationen die ich vornehmen werde.  |
| config.vm.box | Hier habe ich mich für ein Betirebssystem entscheiden, welches ich auf der VM laufen haben will.  |
| db.vm.network | Da definiere ich den Port auf welchen darauf folgend die VM zugreift. In diesem Zusammenhang wäre es für MySQL Port 3306 und für die Webapplikation phpmyadmin Port 80.  |
| db.vm.provision | In diesem Schritt erlaube ich die Variation von einem Shell Skript, in meinem Fall das bootstrap.sh file, nachdem das Guest OS gebootet hat.
| config.vm.provider :virtualbox do vb |  Hier definiere ich den Provider der VM, hierbei Virtualbox. Außerdem habe ich noch Anpassungen gemacht, z.b mehr RAM und CPUs.  |

---

<a name="bootstrap"></a>
## Bootstrap.sh

 
    DBHOST=localhost
    DBNAME=m300
    DBUSER=mzgraggen
    DBROOT=adminmz
    DBPASSWD=dinitante

    apt-get update

    debconf-set-selections <<< "mysql-server mysql-server/root_password password $DBPASSWD"
    debconf-set-selections <<< "mysql-server mysql-server/root_password_again password $DBPASSWD"
    debconf-set-selections <<< "phpmyadmin phpmyadmin/dbconfig-install boolean true"
    debconf-set-selections <<< "phpmyadmin phpmyadmin/app-password-confirm password $DBPASSWD"
    debconf-set-selections <<< "phpmyadmin phpmyadmin/mysql/admin-pass password $DBPASSWD"
    debconf-set-selections <<< "phpmyadmin phpmyadmin/mysql/app-pass password $DBPASSWD"
    debconf-set-selections <<< "phpmyadmin phpmyadmin/reconfigure-webserver multiselect none"

    apt-get -y install mysql-server phpmyadmin

    mysql -uroot -p$DBPASSWD -e "CREATE DATABASE $DBNAME"
    mysql -uroot -p$DBPASSWD -e "grant all privileges on $DBNAME.* to '$DBUSER'@'%' identified by '$DBPASSWD'"
    mysql -uroot -p$DBPASSWD -e "grant all privileges on $DBNAME.* to '$DBROOT'@'%' identified by '$DBPASSWD'"

    cd /vagrant

    sudo sed -i "s/.*bind-address.*/bind-address = 0.0.0.0/" /etc/mysql/mysql.conf.d/mysqld.cnf

    sudo service mysql restart

    service apache2 restart

| Code| Beschreibung|
| --------------| -----------------|
| DBROOT=adminmz | Als aller erstes habe ich die Variabeln definiert, welche ich in einem späteren Verlauf im Skript verwendet habe. Diese Variabel wäre nun für den Root User.  |
| debconf-set-selections ... | Die gleichen Angaben müsste man auch angeben, wenn man den Service manuell installiern würde. Dies wird hier aber automatisch übernommen. |
| mysql -uroot -p$DBROOTPASSWD -e ... | Hier mache ich die Konfigurationen auf der Datenbank bzw. ich erstelle die Datenbanken. |
| apt-get -y install mysql-server phpmyadmin | In diesem Schritt wird der mysql Service bzw. mysql-server und phpmyadmin installiert. |
| sudo sed -i "s/.*bind-address.*/bind-address = 0.0.0.0/" /etc/mysql/mysql.conf.d/mysqld.cnf | Die mysql konfig wird nun ergänzt, so dass man auf die Datenbank per Remote zugreifen kann. |

---
<a name="anwendung"></a>
# Service Anwendung & testen

Damit wir den Service starten können, muss man einige Schritte beachten. Diese werden hier unten beschrieben.

1. Folgenden Befehl auf Git Bash ausführen: `vagrant up`
2. Sobald die VM aufgesetzt wurde folgenden Link aufrufen: http://localhost:3306/phpmyadmin
3. Sich mit folgenden Anmeldedaten anmelden:
   - Benutzername: adminmz oder mzgraggen
   - Passwort:     dinitante

![Anmeldefenster](Bilder/Login.jpg)

4. Nun sieht man auf der Seite, dass es die Datenbank m300 erfolgreich erstellt hat.

![Database](Bilder/phpmyadmin.PNG)

5. Auf dem unten aufgelisteten Bild, sieht man noch, welcher User zur Zeit eingeloggt ist und noch einige andere nützlichen informationen zu der Datenbank.

![Database_info](Bilder/database_info.PNG)

# Quellenangaben

Da ich noch nie so ein Projekt gemacht habe, habe ich mir im Internet eine Anleitung rausgesucht. Diese findet man unter diesem Link [HIER](https://www.yourtechy.com/technology/mysql-server-vagrant-virtualbox/).
