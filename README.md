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