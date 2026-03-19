# SelfPHP

SelfPHP ist ein Bash-basiertes Hilfsprojekt, das eine lokale PHP-Installation mit vielen Erweiterungen automatisiert aufsetzt. Der Schwerpunkt liegt auf einem manuellen Build aus dem PHP-Quellcode und dem anschließenden Einrichten von PHP-FPM, `php.ini` sowie zusätzlicher Extensions wie APCu, Imagick, MongoDB und Redis.

Das Projekt ist klar auf Linux-Systeme ausgelegt. Obwohl das Repository in dieser Workspace auf Windows liegt, sind die enthaltenen Skripte für eine Linux-Umgebung mit Root-Rechten gedacht.

## Zweck

Das Projekt soll eine breite PHP-Umgebung schnell bereitstellen, entweder:

- über Paketinstallation mit `apt`
- oder über einen Build direkt aus `php-src`

Im manuellen Modus werden zusätzlich Konfigurationsdateien für PHP-FPM vorbereitet und externe PECL-nahe Erweiterungen aus Git-Repositories gebaut.

## Einstieg

Der Haupteinstiegspunkt ist die Datei `index` im Projektverzeichnis.

Beispiel:

```bash
cd SelfPHP
bash ./index
```

Wenn keine Datei `version` existiert, fragt das Skript interaktiv nach:

- Modus
- PHP-Version

Die eingegebene Version wird anschließend in der Datei `version` gespeichert.

## Betriebsmodi

### `apt`

Im Modus `apt` installiert das Skript eine große Menge an PHP-Paketen direkt über die Distribution.

Eigenschaften:

- Standardversion ist `8.4`, falls keine Version gesetzt wurde
- installiert viele Module über `apt install php<VERSION>-...`
- eignet sich eher für schnelle Paketbereitstellung als für einen individuellen Source-Build

### `manual`

Im Modus `manual` wird PHP aus den Quellen gebaut und anschließend systemweit eingerichtet.

Der Ablauf ist grob:

1. Hilfsskripte `cru` und `msd` werden bei Bedarf über ein externes Setup-Skript nachgezogen.
2. Der Projektpfad wird in `/root/.bashrc` als `scriptPath` hinterlegt.
3. Als Zielversion wird `PHP-<version>` oder alternativ `master` verwendet.
4. Im vorgesehenen Build-Verzeichnis unter `/usr/local/php` wird der PHP-Quellbaum vorbereitet.
5. Falls nötig wird `buildconf` ausgeführt.
6. Das Projekt analysiert verfügbare `configure`-Optionen und erzeugt daraus Build-Parameter.
7. PHP wird mit FPM-Unterstützung gebaut und installiert.
8. Binaries werden von `/usr/local/bin` und `/usr/local/sbin` nach `/usr/bin` und `/usr/sbin` verschoben.
9. PHP-FPM-Konfigurationen und `php.ini` werden nach `/etc` kopiert und angepasst.
10. Zusätzliche externe Erweiterungen werden gebaut und installiert.
11. Alle gefundenen Erweiterungen im PHP-Extension-Verzeichnis werden in `/etc/php.ini` aktiviert.

## Projektstruktur

### `index`

Hauptskript zur Steuerung der Installation.

### `scripts/gen-rep`

Analysiert die Ausgabe von `./configure --help` im PHP-Quellbaum und erzeugt eine Report-Datei mit Arrays wie:

- `EXTENSIONS_WITH_WITH`
- `EXTENSIONS_WITH_ENABLE`
- `EXTENSIONS_NOT_CONFIGURABLE`

Diese Daten bilden die Grundlage für den späteren Configure-Aufruf.

### `scripts/build`

Lädt die im Report erkannten Optionen und baut daraus den eigentlichen `configure`-Aufruf für PHP. Zusätzlich werden aktuell noch feste Optionen für MariaDB/PDO gesetzt:

- `--with-pdo-mysql=/usr/local/mariadb`
- `--enable-pdo`
- `--enable-mysqlnd`
- `--enable-fpm`

### `externals/main`

Führt nacheinander alle Skripte im Ordner `externals/` aus, außer sich selbst.

### `externals/apcu`

Klonen, bauen und installieren von APCu.

### `externals/imagick`

Klonen, bauen und installieren von Imagick.

### `externals/mongo`

Baut den MongoDB-PHP-Treiber aus dem offiziellen Repository.

### `externals/redis`

Klonen, bauen und installieren von phpredis.

## Voraussetzungen

Für den manuellen Modus werden mindestens folgende Werkzeuge beziehungsweise Rahmenbedingungen vorausgesetzt:

- Linux-System
- Root-Rechte
- `bash`
- `git`
- Compiler- und Build-Toolchain
- `make`
- `cmake` für den MongoDB-Treiber
- `phpize`
- Zugriff auf das PHP-Quellrepository
- Schreibrechte auf `/usr/local`, `/usr/bin`, `/usr/sbin`, `/etc`, `/run` und `/var/log`

Zusätzlich werden systemnahe Werkzeuge wie `sed`, `grep`, `find` und `head` verwendet.

## Typischer Ablauf

```bash
cd SelfPHP
bash ./index
```

Danach interaktiv zum Beispiel:

```text
Mode: manual
PHP version: 8.4
```

## Ausgabe und Seiteneffekte

Das Projekt verändert Systemzustand recht umfassend:

- installiert oder baut PHP systemweit
- kopiert PHP-FPM-Dateien nach `/etc`
- legt Laufzeitpfade unter `/run/php` an
- verändert `/etc/php.ini`
- verändert `/etc/php-fpm.conf`
- verändert `/etc/php-fpm.d/www.conf`
- legt einen Benutzer `php` an, falls dieser nicht existiert
- startet beziehungsweise registriert `php-fpm` über `msd`

Das Projekt ist daher eher als Admin-/Provisioning-Skript zu verstehen als als isolierter lokaler Dev-Installer.

## Bekannte Grenzen

- Die Skripte sind stark distributions- und pfadabhängig.
- Der manuelle Modus erwartet eine Linux-Root-Umgebung und ist nicht portabel auf Windows.
- Externe Hilfstools wie `cru` und `msd` stammen nicht aus diesem Repository.
- Der Build ist nicht reproduzierbar im Sinne eines vollständig gepinnten Dependency-Sets.
- Der Configure-Aufruf setzt Pfade wie `/usr/local/mariadb` voraus.
- Das Projekt enthält aktuell keine Fehlerbehandlung für jeden Einzelschritt.

## Hinweise zur Nutzung

- Vor produktivem Einsatz sollten die Zielpfade und Service-Namen an die eigene Distribution angepasst werden.
- Für Tests empfiehlt sich zuerst ein isoliertes System oder Container.
- Wenn nur eine Standard-PHP-Installation benötigt wird, ist der `apt`-Modus deutlich risikoärmer als der manuelle Source-Build.

## Weiterentwicklung

Sinnvolle nächste Verbesserungen wären:

- klarere Trennung zwischen Download, Build und System-Setup
- konsistente Fehlerbehandlung mit `set -euo pipefail`
- explizite Paketliste für Build-Abhängigkeiten
- Versions-Pinning für externe Erweiterungen
- Dry-Run- oder Non-Interactive-Modus
