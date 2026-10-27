## Server vorbereiten
- Ein Ordner /opt/minecraft-server/ muss vorhanden sein
- Docker Secrets erstellen
```bash
# Verzeichnis erstellen
sudo install -d -m 700 -o root -g root /opt/minecraft-server/secrets

# Sicheres zufälliges Passwort generieren (64 Hex-Zeichen)
sudo sh -c 'umask 077; openssl rand -hex 32 > /opt/minecraft-server/secrets/rcon_password'

# Dateiberechtigungen sicherstellen
sudo chmod 600 /opt/minecraft-server/secrets/rcon_password
sudo chown root:root /opt/minecraft-server/secrets/rcon_password

# Berechtigungen kontrollieren
sudo ls -l /opt/minecraft-server/secrets/rcon_password
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

## Plugins installieren
Die Plugins liegen weiterhin auf dem Raspberry Pi unter:

/home/piadmin/minecraft/data/plugins

Am einfachsten überträgst du neue .jar-Dateien mit SFTP, beispielsweise über WinSCP oder den scp-Befehl auf Windows:
```bash
scp -i "$HOME\.ssh\id_ed25519_raspi" `
  "C:\Pfad\zum\plugin.jar" `
  piadmin@home-pi.local:/home/piadmin/minecraft/data/plugins/
```

## Timing
- you can set a specific date to aktivate the nether and end [WorldAccessBlocker config](./config/plugins/WorldAccessBlocker/config.yml)
- you can set the Date when the Boarder gets expandet to 20000 blocks [compose.yaml](./compose.yaml)

## pre-chunking
- Fortschritt abfargen: `/chunky progress`

## Teams verwalten
```
/team create Rot
/team create Blau
/team create Gruen
/team join Rot
/teamadmin
```