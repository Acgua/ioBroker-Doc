# Voll verschlüsseltes Proxmox-System (LUKS, Debian)

Die folgende Anleitung erklärt, wie man auf einem Barebone (Intel NUC, Zotac, etc.) ein per LUKS voll verschlüsseltes Debian installiert und darüber dann Proxmox installiert. Beim Neustart des Barebone kann man sich dann von Windows aus über SSH verbinden und das Verschlüsselungspasswort eingeben.

### Warum verschlüsselt? 
Bei Diebstahl des Barebones oder etwa bei defekter Festplatte kommt keiner ohne Passphrase an die Daten.

## Voraussetzungen für diese Anleitung

 1. Barebone-Rechner (Intel NUC, Zotac, etc.) mit **einer** Festplatte. Weitere angeschlossene Laufwerke werden in dieser Anleitung nicht berücksichtigt
 1. Grundinstallation auf Barebone erfolgt mit angeschlossenem Monitor, Tastatur, Maus, später braucht man das natürlich nicht mehr ("headless")
 1. Windows-PC (Windows 10) zur Einrichtung/Steuerung wird verwendet. Putty ist installiert.
 1. USB-Stick für das Debian-Image zur Installation auf dem Barebone
 1. Erweiterte Kenntnisse zu Linux und Windows

## Vorbereitung

1. Windows: Debian-Linux (11.5, Bullseye) herunterladen: https://www.debian.org/CD/http-ftp/#stable -> CD -> "amd64" -> runterscrollen -> Datei: `debian-11.5.0-amd64-netinst.iso`
2. Windows: [Etcher (Portable)](https://www.balena.io/etcher/) installieren und ausführen, und Image auf USB-Stick schreiben.
3. Bildschirm, Tastatur/Maus, Netzwerkkabel und USB-Stick an Barebone anschließen, starten, Boot-Menü aufrufen und USB-Stick auswählen

## Debian installieren
 1. Graphical install, Sprache Deutsch etc.
 1. Netzwerk einrichten: Entsprechende Netzwerkbuchse wählen
 1. Rechnername: z.B. "prx"
 1. Domainname: z.B. "local"
 1. Kein Root-Passwort vergeben
 1. Neuer Benutzer: (Benutzername eingeben)
 1. Passwort: (PW eingeben)
 1. Festplatte partitionieren: "Geführt - gesamte Platte mit verschlüsseltem LVM"
 1. Alle Dateien auf eine Partition, mit "Ja" bestätigen
 1. Nun startet das überschreiben mit zufälligen Daten, dauert ewig, aber sollte man machen. Kann man mit "Abbrechen" auch abbrechen und die Installation dennoch weiter fortführen.
 1. Passphrase für die Verschlüsselung vergeben. Bitte sehr gutes Passwort verwenden.
 1. Größe: "max" eingeben oder so belassen
 1. "Partitionierung beenden und Änderungen übernehmen" bestätigen
 1. Deutsch als Spiegelserver, kein Proxy
 1. Statistiken: wie gewünscht ja oder nein
 1. Software: nur SSH Server und Standard-Systemwerkzeuge, **kein Desktop**
 1. Fertig. Neu starten, nun Verschlüsselungspasswort eingeben.
 1. Einloggen mit zuvor vergebenem Account und mit `ip addr` oder `ip route` die IP-Adresse ausgeben
 1. Zu Windows wechseln, Putty öffnen, die IP eingeben, verbinden und einloggen.

## Debian Grundeinrichtung

### IP-Adresse
Ich belasse die Einstellungen (DHCP) und weise die vergebene IP im Router fest zu (z.B. in Fritzbox: Heimnetz > Netzwerk > (Barebone) > Option: "Diesem Netzwerkgerät immer die gleiche IPv4-Adresse zuweisen.")

### Fehlende Firmware

## Zugang via SSH um Verschlüsselungspasswort etwa von Windows aus einzugeben
1. `sudo apt install dropbear-initramfs` Folgende Ausgabe fixen wir später: "WARNING: Invalid authorized_keys file, SSH login to initramfs won't work!"
1. Windows-Konsole: `ssh-keygen -t rsa -f c:\Users\<user>\Downloads\unlock_luks` - dabei ggf. Pfad und Dateiname entsprechend anpassen. Passwort vergeben
1. `sudo nano /etc/dropbear-initramfs/authorized_keys` und Dateiinhalt von `unlock_luks.pub` einfügen, speichern.
1. Change File Permissions (lt. Anleitung, aber wirklich notwendig? - ich habe es jedenfalls gemacht): `sudo chmod 600 /etc/dropbear-initramfs/authorized_keys`
1. `sudo nano /etc/dropbear-initramfs/config` -> Zeile `#DROPBEAR_OPTIONS` ändern in: `DROPBEAR_OPTIONS="-I 300 -j -k -p 2222 -s"`
 1. `ip route`, gibt z.B. in erster Zeile "default via 180.0.0.1 dev enp1s0" aus, hier entspricht DEVICE=enp1s0 und 180.0.0.1 dem Gateway
 1. `sudo nano /etc/initramfs-tools/initramfs.conf`
 1. Für DHCP folgende zwei Zeilen hinzufügen, dabei 2x "enp1s0" ggf. ersetzen durch Ausgabe von ip route:<br>`DEVICE=enp1s0`<br>`IP=:::::enp1s0:dhcp`
 1. `sudo update-initramfs -u` - aktualisieren
 1. Server neu starten - `sudo reboot`
 1. **Windows-Konsole**: `ssh -i c:\Users\<user>\Downloads\unlock_luks -p 2222 -o "HostKeyAlgorithms ssh-rsa" root@180.0.0.22` - IP entsprechend anpassen
 1. Are you sure you want to continue connecting -> yes
 1. Passwort für SSH-Schlüssel eingeben
 1. `cryptroot-unlock` eingeben
 1. Verschlüsselungspasswort eingeben
 1. Wenn erfolgreich, erscheint: "cryptsetup: sda3_crypt set up successfully"

(per: https://www.dwarmstrong.org/remote-unlock-dropbear/)

## Proxmox installieren

Vorgehensweise gemäß: https://pve.proxmox.com/wiki/Install_Proxmox_VE_on_Debian_11_Bullseye

### Besonderheiten:
 1. Zum Einloggen in Proxmox (https://IP-Adresse:8006/) dann doch ein Root-Passwort vergeben, da man ansonsten sich nicht in Proxmox einloggen konnte (dies gilt es noch zu optimieren):
   <br> - `sudo passwd root`, 2x das Passwort für root eingeben.
   <br> - Proxmox-Webinterface: Benutzername "root" und entsprechendes Passwort eingeben, bei Domäne "Linux PAM standard authentication" auswählen

 1. Proxmox-Webinterface: Beim Anlegen einer Linux Bridge unter Rechenzentrum > prx > System > Netzwerk:
<br>Name: `vmbr0`
<br>IPv4/CIDR: `180.0.0.22/24` (IP gemäß `ip addr`, dahinter `/24`)
<br>Gateway: `180.0.0.1` (gemäß `ip route`)
<br>Bridge ports: `enp1s0` (Name der aktiven Netzwerkkarte)
<br><br>(nach dem Speichern oben auf "Konfiguration anwenden" klicken)
