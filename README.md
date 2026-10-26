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