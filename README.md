# rclone-mount-home-hidden-cache

Script Bash per montare un remote `rclone` come filesystem locale tramite `rclone mount`, usando:

- una **directory locale visibile nella home** come mountpoint;
- una **directory nascosta nella home** come VFS cache;
- un `tmpfs` montato sulla directory cache, così la cache vive interamente in RAM.

Questa è la variante più pratica per uso desktop quotidiano, perché il mountpoint è immediatamente accessibile dall’utente, mentre la cache resta nascosta e ad alte prestazioni. `rclone mount` permette di scegliere liberamente sia il mountpoint locale sia la directory di cache tramite `--cache-dir`. [web:16]

## Caratteristiche

- Selezione interattiva del remote configurato in rclone.
- Calcolo dinamico della RAM disponibile tramite `MemAvailable`. [web:352]
- Creazione automatica di:
  - `~/NomeRemote` come cartella file locale;
  - `~/.rclone-cache-NomeRemote` come cache nascosta.
- Mount di un `tmpfs` direttamente sulla cartella cache nascosta.
- Apertura automatica del mount in Nemo.
- Cleanup con `Ctrl+C`:
  - chiusura Nemo;
  - unmount FUSE;
  - stop di rclone;
  - unmount del `tmpfs` della cache.

## Architettura

Esempio per un remote chiamato `Mega`:

```text
~/Mega                      <- mountpoint locale rclone
~/.rclone-cache-Mega        <- cache VFS nascosta, montata su tmpfs
```

Il mountpoint è una directory locale normale nella home. La cache è nascosta e separata, ma viene montata su `tmpfs`, quindi i dati temporanei della cache vivono comunque in RAM. `rclone` usa per default una cache sotto `~/.cache/rclone/`, quindi questa soluzione segue una convenzione naturale del tool ma con un path dedicato per remote. [web:22][web:28]

## Vantaggi

- Mountpoint comodo, leggibile e subito disponibile nella home.
- Cache nascosta, ordinata e separata.
- Prestazioni molto simili alla versione “tutto in tmpfs”, perché il fattore principale resta la VFS cache in RAM. [web:16]
- Migliore usabilità desktop.

## Svantaggi

- Architettura leggermente meno “monolitica” della versione con unico tmpfs.
- Il mountpoint resta visibile nella home anche se la cache è nascosta.

## Requisiti

Pacchetti richiesti su Debian/Ubuntu/Linux Mint:

```bash
sudo apt update
sudo apt install rclone nemo fuse3 fuse libfuse2 coreutils netcat-openbsd psmisc lsof
```

Per lo smontaggio serve `fusermount3` oppure `fusermount`. [web:16]

## Uso

```bash
chmod +x rclone-mount-home-hidden-cache.sh
./rclone-mount-home-hidden-cache.sh
```

Flusso:
1. scegli il remote;
2. scegli la percentuale di `MemAvailable` da usare per la cache in RAM;
3. lo script crea il mountpoint in home;
4. lo script crea la cache nascosta e ci monta sopra un `tmpfs`;
5. rclone monta il remote nella cartella della home;
6. Nemo si apre direttamente sulla cartella mountata;
7. `Ctrl+C` smonta tutto.

## Parametri rclone usati

La configurazione tipica è:

```bash
rclone mount REMOTE: ~/NomeRemote \
  --cache-dir ~/.rclone-cache-NomeRemote \
  --vfs-cache-mode full \
  --vfs-cache-max-size ... \
  --vfs-cache-min-free-space ... \
  --vfs-cache-max-age 1h \
  --dir-cache-time 10m \
  --buffer-size 16M \
  --vfs-read-chunk-size 32M \
  --vfs-read-chunk-size-limit 256M
```

La documentazione `rclone mount` raccomanda di usare la VFS cache per workload reali e permette di ottimizzare ulteriormente tramite `--vfs-read-chunk-size`, `--vfs-read-chunk-streams`, `--buffer-size` e `--dir-cache-time`. [web:16][web:346][web:347]

## Prestazioni

Questa è in genere la versione più equilibrata. A parità di dimensione cache e flag rclone, il mountpoint in home non penalizza sensibilmente le prestazioni, perché la parte che incide di più è la cache VFS in RAM. [web:16][web:346]  
Per questo motivo questa variante è spesso preferibile per uso quotidiano: offre quasi le stesse prestazioni pratiche della variante interamente sotto tmpfs, ma con ergonomia migliore. [web:16]

## Quando usarla

Usa questa versione se vuoi:
- prestazioni elevate;
- un mountpoint locale comodo e stabile;
- una cache nascosta in RAM;
- una soluzione desktop più pulita per l’uso giornaliero.

## Limiti

- Il mountpoint non è effimero come nella variante tutta in tmpfs, anche se resta solo una directory locale usata come punto di mount.
- Richiede comunque attenzione alla quantità di RAM allocata.

## Licenza

MIT.
