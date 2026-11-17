# Session-Uebergabe: BMW iDrive 7 Apps

Stand: 2026-11-17

## Ziel und Umgebung

- Eigene Apps fuer BMW MGU / iDrive 7 entwickeln und lokal emulieren.
- Entwicklung unter Windows mit VS Code, PowerShell, Python 3 und Node.js.
- Das BMW-SDK ist auf dem anderen Rechner bereits vorhanden. Es gehoert nicht ins Git-Repository.
- Kommunikation mit dem Benutzer auf Deutsch.

## Bauen und emulieren

Im integrierten VS-Code-Terminal, aus dem Repository-Verzeichnis:

```powershell
python kit\tools\build.py redline
node kit\tools\emu.js redline
```

Danach http://localhost:8077 im Browser oeffnen. Das Terminal bleibt offen;
Ctrl+C beendet den Emulator. Der Emulator baut beim Start selbst und baut
Aenderungen unter `apps\<id>` und `kit\runtime` beim Speichern neu.

- Python und Node.js reichen fuer normale App-Entwicklung; kein npm install
  und kein Docker erforderlich.
- Der Browser verwendet den bereits enthaltenen WebAssembly-/WebGL-Renderer.
- Demo drive oder manuelle Fahrzeugsignale lassen sich im Emulator einstellen.
- Der Emulator bildet nicht das gesamte BMW-System ab: insbesondere keine
  Antworten auf echte Diagnose-/UDS-Anfragen und keine vollstaendige HUD-/Cluster-Umgebung.
- Ohne BMW-SDK kann `python kit\tools\build.py redline --preview-only`
  eine Offline-Vorschau ohne installierbares Paket erstellen.

## Bereits behobene Windows-Probleme

Urspruenglicher Startfehler:

```text
emu: renderer socket: listen EACCES: permission denied ...\Temp\mgu-emu-<pid>.sock
```

Daraufhin wurden diese Dateien angepasst:

- [kit/tools/emu.js](kit/tools/emu.js):
  Windows verwendet `\\.\pipe\mgu-emu-<pid>` statt eines Unix-Socket-Dateipfads.
  macOS/Linux behalten den bisherigen Unix-Socket.
  Fuer den Build wird unter Windows `python`, sonst `python3` gestartet.
  Letzteres behebt den anschliessend beobachteten Microsoft-Store-Aliasfehler
  bei `python3`.
- [kit/runtime/renderd.js](kit/runtime/renderd.js):
  `RenderClient.available()` erkennt unter Windows Named-Pipe-Pfade,
  statt ausschliesslich `fs.statSync(...).isSocket()` zu verwenden.
- [docs/emulator.md](docs/emulator.md):
  Die plattformabhaengige lokale Verbindung ist dokumentiert.

Verifiziert auf Windows mit Node.js v24.19.0 und Python 3.12.0:

- Syntaxpruefung beider geaenderten JavaScript-Dateien erfolgreich.
- Emulator baut Redline erfolgreich und meldet `redline running`.
- Renderer verbindet sich erfolgreich mit `mgu-render/1`.
- Emulator-Seite antwortet mit HTTP 200.
- Eine visuelle Browserpruefung und macOS-/Linux-Tests wurden nicht durchgefuehrt.
- Die Testprozesse wurden wieder beendet.

Die Aenderungen wurden vom Assistenten nicht committet. Der Benutzer uebernimmt
Commit und Uebertragung auf den anderen Rechner.
Git-Status und Diff konnten in dieser Umgebung wegen einer
`unsafe repository`-Eigentuemerpruefung nicht zuverlaessig eingesehen werden;
die globale Git-Konfiguration wurde nicht veraendert.

## Naechster Schritt: eigene App

Es wurde noch keine eigene App angelegt und noch keine konkrete Funktion festgelegt.

Empfehlung:

1. [apps/gforce](apps/gforce) als relativ kleine Vorlage nach
   `apps\my_app` kopieren. Redline ist fuer den Einstieg deutlich umfangreicher.
2. In der kopierten `app.json` mindestens `id`, `title`, Beschreibung,
   Version und eine eindeutige vierstellige `loggingId` anpassen.
   Ordnername und `id` sollten uebereinstimmen.
3. `main.js` anpassen; Bildschirm- und Button-IDs muessen mit `app.json`
   uebereinstimmen. Ein uebernommenes `icon.png` bei Bedarf ersetzen.
4. Nur die tatsaechlich benoetigten Fahrzeugsignale in `app.json` deklarieren.
5. Mit `python kit\tools\build.py my_app` bauen und mit
   `node kit\tools\emu.js my_app` emulieren.

Das Kit generiert Paketstruktur, RHMI-Layout, Berechtigungen und Runner aus der
App-Beschreibung. Gemeinsame Laufzeitmodule werden als `require('./kit/mgu')`,
`require('./kit/gauges')` und `require('./kit/car')` eingebunden;
den Runtime-Ordner nicht in die Quell-App kopieren.

## Relevante Dokumentation und Beispiele

- [README.md](README.md), Abschnitt 6: Writing your own app,
  mit kleinem Boost-Beispiel fuer `app.json` und `main.js`.
  `apps\boost` existiert nicht als fertiger Ordner.
- [docs/emulator.md](docs/emulator.md): Emulator, Bedienung und Grenzen.
- [apps/gforce/app.json](apps/gforce/app.json) und
  [apps/gforce/main.js](apps/gforce/main.js): vorhandene kompakte Beispiel-App.
- [apps/redline/README.md](apps/redline/README.md): Redline und experimentelle Funktionen.

## Fortsetzung auf dem anderen Rechner

Repository inklusive dieser Datei und der Windows-Korrekturen aktualisieren.
Dann im neuen Chat:

> Lies session.md und setze unsere Arbeit an einer eigenen BMW-iDrive-7-App fort.
> Das SDK ist auf diesem Rechner bereits vorhanden.
