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
