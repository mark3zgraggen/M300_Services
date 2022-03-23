# M300-LB2 Dokumentation

---

# Inhaltsverzeichnis

- [Einführung](#einführung)
- [Grafische Übersicht des Services](#grafische)
- [Quellenangaben](#quellenangaben)
- [Vagrantfile](#vagrantfile)
- [Bootstrap.sh](#bootstrap)
- [Service Anwendung](#anwendung)
- [Service testen](#testen)

---

# Einführung

Wir haben im Modul 300 den Auftrag bekommen, ein eigenes Projekt mit Vagrant zu gestallten.
Mein Projekt ist, dass man ein MySQL automatisch einrichten lassen kann.

---
<a name="grafische"></a>
# Grafische Übersicht des Services

![Grafische-Übersicht](Bilder/grafischeuebersicht.jpg)

---

# Quellenangaben

Da ich noch nie so ein Projekt gemacht habe, habe ich mir im Internet eine Anleitung rausgesucht. Diese findet man unter diesem Link [HIER](https://www.yourtechy.com/technology/mysql-server-vagrant-virtualbox/).
Da ich nicht alles eins zu eins übernehmen wollte, habe ich einige Anpassungen am Skript vorgenommen.

Dies möchte ich Ihnen aber weiter unten genauer erklären.

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


