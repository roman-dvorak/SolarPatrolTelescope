# Rotace logů SPS (logrotate)

Konfigurace `arom-logrot.conf` rotuje logy Sluneční patroly v `/storage/log`.

## Co se rotuje

| Cesta | Obsah |
|---|---|
| `/storage/log/start.log` | log startovacího skriptu |
| `/storage/log/ros/*.log` | logy ROS nodů |
| `/storage/log/ros/latest/*.log` | logy aktuálního běhu `roslaunch` |

## Nastavení

- rotace jednou denně, logy se drží **7 dní** (`rotate 7`)
- soubor nad 50 MB se zrotuje při nejbližším běhu logrotate
- starší logy se komprimují (gzip), poslední rotovaný zůstává nekomprimovaný
- názvy rotovaných souborů obsahují datum, např. `rosout.log-20261029`
- `copytruncate`: log se zkopíruje a původní soubor se vynuluje, takže běžící nody není potřeba restartovat
- logrotate pracuje se soubory jako uživatel `odroid` (`su odroid odroid`)

## Instalace

Logrotate je na Ubuntu obvykle nainstalovaný. Pokud chybí:

```bash
sudo apt-get install logrotate
```

Konfiguraci nainstalujte jako kopii s vlastníkem root:

```bash
sudo install -o root -g root -m 644 \
    ~/arom_ws/src/SolarPatrolTelescope/tools/logrotate/arom-logrot.conf \
    /etc/logrotate.d/arom-logrot
```

> **Nepoužívejte symlink do repozitáře.** Logrotate ignoruje konfigurační soubory, které nepatří rootovi
> (`Ignoring ... because the file owner is wrong`). Git při změně souboru vytvoří nový soubor
> s vlastníkem `odroid` a rotace by potichu přestala fungovat.

Po každé změně `arom-logrot.conf` v repozitáři je potřeba příkaz `install` spustit znovu.

## Ověření

Suchý běh (nic nemění):

```bash
sudo logrotate -d /etc/logrotate.d/arom-logrot
```

Výstup musí obsahovat `Handling 1 logs` a seznam logů, nesmí obsahovat `error`.

Vynucená rotace (test):

```bash
sudo logrotate -f /etc/logrotate.d/arom-logrot
ls -l /storage/log/ros/
```

Logrotate spouští denně systémový cron (`/etc/cron.daily/logrotate`), nic dalšího není potřeba nastavovat.

## Požadavky na oprávnění

- Všechny logy v `/storage/log` musí patřit uživateli `odroid` (nebo pro něj být zapisovatelné).
  Jinak logrotate hlásí `error opening ...: Permission denied`. Oprava:

  ```bash
  sudo chown odroid:odroid /storage/log/ros/<soubor>.log
  ```

  Pokud se soubor s vlastníkem root objevuje opakovaně, některá část systému běží pod rootem.

- Do `start.log` musí startovací skript zapisovat v režimu připojování (`>>`), ne `>`.
  Jinak po vynulování vznikne soubor vyplněný nulami.

## Poznámky

- Adresáře jednotlivých běhů ROS (`/storage/log/ros/<uuid>/`) logrotate nemaže.
  Vznikají jen při restartu `roslaunch`, staré je potřeba občas smazat ručně.
- Stav rotace (kdy byl který log naposledy rotován) je v `/var/lib/logrotate/status`.
