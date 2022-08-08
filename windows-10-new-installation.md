## Neuinstallation unter Windows 10
Stand: August 2022

## Vorwort
Für mich war die Installation unter Windows 10 eine ziemliche Herausforderung, weil es an offizieller Dokumentation mangelt und der offizielle [Windows-Installer](https://www.iobroker.net/#de/download) ziemlich veraltet ist (Version: 23.11.2019 - Stand August 2022).

Dies wird von viel Foren-Einträgen so auch bestätigt, meine Quellen waren u.a.:
* https://forum.iobroker.net/topic/33421/iobroker-unter-windows-installieren-ohne-installer
* https://forum.iobroker.net/topic/53050/erstinstallation-unter-windows
* https://forum.iobroker.net/topic/52340/installation-iobroker-unter-win10
* https://forum.iobroker.net/topic/51574/windows-installation-update - PDF-Anleitung

Ohne Installer hat es bei mir nicht funktioniert, und mit Installer nur mit node.js 14, mit node.js 16 nicht! Ebenso auch nicht mit der o.g. PDF-Anleitung.

Aber hier folgt eine Anleitung, die bei mir zum Erfolg führt. Wichtig ist dabei auch, diese Reihenfolge einzuhalten!
Falls es nicht funktioniert, ioBroker wieder deinstallieren über die Wondows-Installer-exe-Datei, also diese wieder ausführen und damit dann deinstallieren. Außerdem ggf. weitere zusätzlich zuvor durch ioBroker installierte Softwarepakete deinstallieren (Bonjour, Bonjour SDK, MS Visual C++ 2008, Python).

## 1. ioBroker Installation via Installer
1. Etwaige fremde Virenscanner deaktivieren 
2. Windows-Installer von [https://www.iobroker.net/#de/download](https://www.iobroker.net/#de/download) herunterladen
3. Windows-Installer starten und Installation durchführen

## 2. Node.js Update auf v14 (v14.20.0 bei mir), v16 funktionierte nicht!
1. In Windows 10: Start-Button anklicken, "ioBroker" eingeben und App (**ioBroker-Konsole**) öffnen:<br>![image](https://user-images.githubusercontent.com/95978245/183481279-a805790c-b8ff-401e-ae9d-4133371dbabc.png)
2. In der Konsole `iobroker stop` eingeben
3. Datei `nodevars.bat` vom ioBroker-Installationspfad unterhalb `nodejs` (z.B. unter `c:\Program Files\iobroker\<Gewählter Host-Name bei Installation>\nodejs\`) z.B. in das "Downloads"-Verzeichnis oder auf den Desktop kopieren.
4. Download von node.js als Archiv (zip), Hier der Link für node.js v14.20.0 (Windows x64): https://nodejs.org/download/release/v14.20.0/node-v14.20.0-win-x64.zip
5. Entpacken des Downloads und kopieren den gesamten Ordners über den vorhandenen nodejs-Ordner.
6. Im nodejs-Ordner `nodevars.bat` durch die zuvor gesicherte Datei ersetzen (darüber kopieren/ersetzen).
7. In der ioBroker-Konsole `iobroker start` eingeben
8. `node -v` eingeben -> sollte jetzt `v14.20.0` ausgeben.

## 3. Veralteten JS-Controller aktualisieren
1. In die ioBroker-Konsole gehen.
2. Zum Testen der installierten JS-Controller-Version `iobroker version js-controller` eingeben. Ausgabe wird wohl `2.1.0` sein (Stand August 2022).
3. `iobroker stop` eingeben
4. `npm i iobroker.js-controller@stable --ignore-scripts` eingeben zur Aktualisierung des JS-Controllers
5. Sobald durchlaufen, den Windows-Rechner neu starten (war bei mir notwendig).
6. Warten bis im Task-Manager (Str+Alt+Entf -> Task Manager), Tab "Dienste" der Eintrag "ioBroker(\<DEINE BEZEICHNUNG\>)" auf Status = "Wird ausgeführt" ist.
7. In der ioBroker-Konsole `iobroker update` eingeben: Hier sollte der Admin-Adapter noch eine alter Version zeigen, der JS-Controller aber aktuell.
8. Soweit alles gut, nun nächsten Schritt ausführen.
  
## 4. Veralteten Admin-Adapter aktualisieren
1. In ioBroker-Konsole `iobroker upgrade admin` eingeben, damit wird der Admin-Adapter aktualisiert. Hierbei ist ggf. eine Bestätigung mit ja/yes erforderlich in der Konsole.
2. `iobroker start`
3. Jetzt sollte über `http://localhost:8081/` oder `http://127.0.0.1:8081/` die Admin-Oberfläche erreichbar sein. Falls nicht: Rechner neu starten und erneut probieren. Sicherstellen, dass im Taskmanager "ioBroker(<DEINE BEZEICHNUNG>)" auf Status = "Wird ausgeführt" ist. Manuelles Starten ioBroker: ioBroker-Konsole öffnen und `iobroker start` eingeben.
  
## Fertig
Nun sollte also ioBroker unter Windows laufen.


