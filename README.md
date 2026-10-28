## Paper-Konfiguration aus dem Repo

`config/config/paper-world-defaults.yml` wird beim Deployment durch `repo-sync`
nach `/opt/minecraft-server/config/config/paper-world-defaults.yml` kopiert.
Beim Minecraft-Start übernimmt das Image sie nach `/data/config/paper-world-defaults.yml`.
Der doppelte `config/`-Pfad im Repo ist beabsichtigt: `COPY_CONFIG_DEST: /data`
bildet den Inhalt des äußeren `config/`-Ordners auf das Serververzeichnis ab.

Repo-Werte überschreiben dabei die bestehende Datei. Einstellungen in den
weltbezogenen `paper-world.yml`-Dateien haben weiterhin Vorrang vor diesen Defaults.

## Plugins aus dem Repo

Plugin-JARs und ihre Konfigurationen liegen gemeinsam unter `config/plugins/`.
Beispielsweise wird `config/plugins/HeadHunter-1.2.0.jar` durch `repo-sync` nach
`/opt/minecraft-server/config/plugins/HeadHunter-1.2.0.jar` kopiert und beim
Minecraft-Start nach `/data/plugins/HeadHunter-1.2.0.jar` übernommen.
Geyser-Erweiterungen liegen unter `config/plugins/Geyser-Spigot/extensions/`.
Ein separater Repo-Ordner `plugins/` und dessen Bind-Mounts werden nicht benötigt.

Nach dem ersten Deployment dieser Umstellung kann die alte Quellkopie auf dem Pi
entfernt werden. Der folgende Befehl prüft zuerst, dass die neue Quellkopie existiert:

```bash
sudo test -s /opt/minecraft-server/config/plugins/HeadHunter-1.2.0.jar &&
  sudo rm -- /opt/minecraft-server/plugins/HeadHunter-1.2.0.jar
```

Die aktive Datei unter `/home/piadmin/minecraft/data/plugins/` bleibt erhalten.
`repo-sync` kopiert Dateien, entfernt aber keine aus dem Repo gelöschten Dateien
auf dem Host. Bei einem späteren Wechsel des JAR-Dateinamens müssen veraltete
Versionen auch aus den Quell- und Laufzeitordnern auf dem Host entfernt werden.

## EssentialsX

Das EssentialsX-Hauptplugin wird über `PLUGINS` in `compose.yaml` als offizieller
Release `2.22.0` heruntergeladen. Die Version ist festgelegt und wird durch Ändern
der Download-URL aktualisiert. Weitere EssentialsX-Module sind nicht enthalten.

Modrinth weist aktuell keine zu Minecraft `26.3` passende EssentialsX-Version aus.
Der Direktdownload umgeht nur diesen Downloadfilter; er bestätigt keine
Laufzeitkompatibilität. Nach dem Deployment die Container-Logs auf EssentialsX-
Ladefehler prüfen und als Operator `/version Essentials` ausführen.
Die generierte Konfiguration liegt auf dem Pi unter
`/home/piadmin/minecraft/data/plugins/Essentials/config.yml`.

Quelle: [EssentialsX 2.22.0 und unterstützte Serverversionen](https://github.com/EssentialsX/Essentials/releases/tag/2.22.0).

## BlueMap

BlueMap wird über `MODRINTH_PROJECTS` in Compose für die konfigurierte
Minecraft-/Paper-Version heruntergeladen. Der integrierte Webserver ist über
TCP-Port `8100` veröffentlicht. Nach Commit, Push und vollständigem Redeployment
ist die Karte unter `http://<IP-des-Pi>:8100/` erreichbar (HTTP, nicht HTTPS).
Im Heimnetz ist keine Router-Portweiterleitung erforderlich. Über Tailscale kann
die Tailscale-IP des Pi verwendet werden, sofern die Zugriffsregeln dies erlauben.
Für öffentlichen Zugriff wären eine passende Freigabe oder ein Reverse-Proxy
mit HTTPS nötig. Die Minecraft-Whitelist schützt die Webkarte nicht: BlueMap hat
keine eigene Anmeldung, und Karten sowie ggf. Spielerpositionen sind für alle
sichtbar, die den Web-Port erreichen können.

Die Repo-Konfiguration liegt unter `config/plugins/BlueMap/` und wird beim Start
nach `/data/plugins/BlueMap/` kopiert:

- `core.conf`: `accept-download: true` erlaubt den notwendigen Download der
  Minecraft-Client-Ressourcen von Mojang (Java-Edition-Lizenz und EULA vorausgesetzt).
  `render-thread-count: 1` begrenzt die Renderarbeit auf einen Thread.
- `plugin.conf`: `player-render-limit: 1` pausiert das Rendern, sobald mindestens
  ein Spieler online ist. Die bereits gerenderte Karte bleibt erreichbar.
  Mit `-1` wird auch während des Spielens gerendert.
- `webserver.conf`: aktiviert den Webserver auf Port `8100` im Container.

Weitere Einstellungen sind für den Grundbetrieb nicht erforderlich. BlueMap
erstellt beim ersten Start die übrigen Standardkonfigurationen und Karten für
die gefundenen Welten. Es liest gespeicherte Welt-Chunks und erzeugt daraus
3D-Kartendaten für den Browser; Spieler brauchen keinen zusätzlichen Mod.
Änderungen erscheinen nach dem Speichern der Welt und dem nächsten Renderlauf.
Das erste Rendern großer, etwa mit Chunky vorgenerierter Welten kann auf dem Pi
lange dauern und viel Speicherplatz belegen. Chunky und BlueMap möglichst nicht
gleichzeitig große Flächen bearbeiten lassen; bei Bedarf Kartenflächen begrenzen.

Status und Wartung als Operator:

```text
/bluemap
/bluemap maps
/bluemap stop
/bluemap start
/bluemap reload
```

`stop` und `start` pausieren bzw. erlauben das Rendern; das Spielerlimit gilt
weiterhin. Die Kartendaten bleiben unter `/home/piadmin/minecraft/data/bluemap/`
auf dem Pi persistent. Die automatisch erstellten Kartenkonfigurationen liegen
unter `/home/piadmin/minecraft/data/plugins/BlueMap/maps/`.
Änderungen an den drei vom Repo verwalteten Dateien im Repo vornehmen und neu
deployen, da lokale Änderungen an diesen Dateien beim Start überschrieben werden.
Bei BlueMap-Updates die Release-Hinweise beachten: Manche Updates erfordern das
Erneuern der Webapp-Dateien zusätzlich zum automatischen JAR-Update.

Dokumentation: [Installation](https://bluemap.bluecolored.de/wiki/getting-started/Installation.html),
[Konfiguration und Performance](https://bluemap.bluecolored.de/wiki/getting-started/Configuration.html),
[Befehle](https://bluemap.bluecolored.de/wiki/getting-started/Commands.html).

## Simple Voice Chat

`simple-voice-chat` wird über `MODRINTH_PROJECTS` für Paper installiert.
Compose veröffentlicht den separaten Voice-Port `24454/udp`.
`config/plugins/voicechat/voicechat-server.properties` wird beim Start nach
`/data/plugins/voicechat/voicechat-server.properties` kopiert. Konfiguriert sind
48 Blöcke Sprachreichweite, 24 Blöcke Flüstern, Gruppen und optionaler Voicechat.
Gruppen erlauben Gespräche unabhängig von der Entfernung.

Nach Commit, Push und vollständigem Redeployment:

1. Java-Spieler installieren Simple Voice Chat für ihre Minecraft-Version und
   ihren Client-Modloader (z. B. Fabric). Möglichst dieselbe Voicechat-Version wie
   auf dem Server verwenden. Im Spiel mit `V` Mikrofon, Ausgabe und Sprechtaste
   konfigurieren. Das Serverplugin allein reicht nicht für Sprachübertragung.
2. Bei direktem Internetzugriff UDP-Port `24454` im Router zum Pi weiterleiten und
   in einer vorhandenen Firewall erlauben. Im LAN oder über Tailscale ist keine
   Router-Weiterleitung nötig; UDP muss zwischen Spielern und Pi erreichbar sein.
   Ein reiner Minecraft-TCP-Tunnel transportiert den Voicechat nicht. Bei einem
   separaten UDP-Tunnel dessen öffentliche Adresse und Port als `voice_host`
   eintragen, ansonsten den Wert leer lassen.
3. Als Operator mit `/voicechat test <Spielername>` die Verbindung eines Spielers
   mit installiertem Client-Mod prüfen. Gewöhnliche TCP-Porttester helfen hier nicht.

Bedrock-/Geyser-Spieler können Simple Voice Chat nicht direkt nutzen. Durch
`force_voice_chat=false` können sie und Java-Spieler ohne Mod weiterhin mitspielen.
Änderungen an der Properties-Datei im Repo vornehmen und neu deployen.

Dokumentation: [Installation](https://modrepo.de/minecraft/voicechat/wiki/installation),
[Server-Konfiguration](https://modrepo.de/minecraft/voicechat/wiki/server_config),
[Verbindungstest](https://modrepo.de/minecraft/voicechat/wiki/server_setup)
und [Bedrock-/Client-Hinweise](https://modrepo.de/minecraft/voicechat/faq).

## Änderungen mit Portainer deployen

Nach Änderungen die betroffenen Dateien committen und pushen. Ein manueller
Deploy-Zähler in `compose.yaml` ist nicht erforderlich.

In Portainer den Stack öffnen und **Pull and redeploy** mit
**Re-pull image and redeploy** verwenden. Das vollständige Redeployment startet
`repo-sync` erneut und erstellt den Minecraft-Container neu. Minecraft wartet auf
den erfolgreichen Sync und übernimmt beim Start die Konfigurationen, Plugins,
Geyser-Erweiterungen und Ressourcenpakete aus den eingebundenen Quellordnern.
Die Option kann auch neuere Container-Images für die konfigurierten Tags laden.

Für automatische GitOps-Deployments **Force redeployment** aktivieren; ein
Webhook eignet sich, um das Deployment gezielt nach einem Push auszulösen.
Bei Polling erzwingt diese Option ein Deployment in jedem Intervall, auch ohne
Git-Änderungen, und kann damit unnötige Serverneustarts verursachen.

Ein einfacher Container-Neustart oder ein normales Compose-Update ohne
Neuerstellung reicht für reine Repo-Dateiänderungen nicht zuverlässig aus.
`repo-sync` bleibt ein einmaliger Vorbereitungsschritt; es gibt keinen laufenden
Git-Abgleich. Der Sync verwendet den in `REPO_BRANCH` angegebenen Branch (`main`).

Portainer-Dokumentation:
[vollständiges Redeployment](https://docs.portainer.io/faqs/troubleshooting/stacks-deployments-and-updates/empty-relative-bind-mounts)
und [GitOps-Optionen](https://docs.portainer.io/user/docker/stacks/add).

## Server vorbereiten
- Ein Ordner /opt/minecraft-server/ muss vorhanden sein
- Docker Secrets erstellen
```bash
sudo bash -euc '
  # Für das jeweilige Secret anpassen
  secret_file="/opt/minecraft-server/secrets/rcon_password"
  secret_gid="1000"

  # Secret-Verzeichnis vorbereiten
  install -d -m 700 -o root -g root -- "$(dirname -- "$secret_file")"

  # Neue Dateien zunächst nur für root zugänglich machen
  umask 077

  # Verhindert versehentliches Überschreiben vorhandener Secrets
  set -o noclobber

  # Zufälliges Passwort erzeugen
  openssl rand -hex 32 > "$secret_file"

  # Leserechte für die Container-Gruppe vergeben
  chown "root:$secret_gid" -- "$secret_file"
  chmod 640 -- "$secret_file"

  # Rechte kontrollieren
  stat -c "%A %a %u:%g %n" -- "$secret_file"
'
```

## Minecraft-Befehle ausführen
Unter Containers → minecraft → Console kannst du eine Shell wie /bin/sh öffnen und darin beispielsweise ausführen:
```bash
rcon-cli --host 127.0.0.1 "list"
```
```bash
rcon-cli --host 127.0.0.1 \
  "whitelist add WeitererSpieler"
```

## Timing
- you can set a specific date to aktivate the nether and end [WorldAccessBlocker config](./config/plugins/WorldAccessBlocker/config.yml)
- you can set the Date when the Boarder gets expandet to 20000 blocks [compose.yaml](./compose.yaml)

## pre-chunking
- add into compose:
´´´yaml
  enviorment:
    RCON_CMDS_STARTUP: |-
      bluemap stop
      chunky world world
      chunky shape square
      chunky spawn
      chunky radius 10000
      chunky start
´´´
- Fortschritt abfargen: `/chunky progress`

## Teams verwalten
```
/team create Rot
/team create Blau
/team create Gruen
/team join Rot
/teamadmin
```

## Crossplay 
- für Badrock Signaling und Gameplay braucht es Protfreigaben oder Tunneling da Playstation nicht einfach Talescale verwenden kann: `19132/tcp`, `19132/udp`
- Bedrock spieler müssen erst einmal gejoint sein bevor sie gewhitelisted werden können, also:
```
/whitelist off
/fwhitelist add XboxGamertag
/whitelist on
```
