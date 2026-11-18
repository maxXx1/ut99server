# Bitcore.cz | UT99 Warmup Server – Dokumentace konfigurace

Tento repozitář obsahuje zálohu a dokumentaci konfiguračních souborů pro dedikovaný **Unreal Tournament (UT99, OldUnreal v469)** server provozovaný přes **LinuxGSM (LGSM)** v LXC kontejneru na Proxmoxu.

---

## 1. Přehled souborů

| Soubor | Účel |
| :--- | :--- |
| [ut99server.ini](file:///e:/Bitcore/ut99server/serverfiles/System/ut99server.ini) | **Hlavní konfigurační soubor serveru**, se kterým LGSM spouští `ucc-bin`. |
| [UnrealTournament.ini](file:///e:/Bitcore/ut99server/serverfiles/System/UnrealTournament.ini) | Sekundární / záložní `.ini` sjednocený na stejné moderní parametry (bez MVE2i). |
| [MVE_Config.ini](file:///e:/Bitcore/ut99server/serverfiles/System/MVE_Config.ini) | Čistý výchozí předpis pro **Map Vote Extended (MVE2i)**. |
| [MVe_Config.backup.ini](file:///e:/Bitcore/ut99server/serverfiles/System/MVe_Config.backup.ini) | Záloha ostrého `MVE_Config.ini` ze serveru (doplněná samotným MVE2i o výchozí parametry a vygenerovaný `ServerCodeName`). |

> **Poznámka ke spouštěnému `.ini`:** LGSM spouští UT99 binárku s parametrem `ini=/home/ut99server/serverfiles/System/ut99server.ini`. Na ostrém serveru je aktivní právě `ut99server.ini` a `MVE_Config.ini`.

---

## 2. Klíčové nastavení serveru (`ut99server.ini`)

### Základní identifikace a porty
* **Název serveru:** `Bitcore.cz | Warmup server` (`ShortName=bitcore`)
* **Herní port:** `7777` (UDP)
* **LAN Beacon port:** `8777`
* **WebAdmin:** `http://<IP_SERVERU>:8076/ServerAdmin` (`ListenPort=8076`, `bEnabled=True`)
  * Výchozí uživatel: `admin`
  * Heslo (`AdminPassword` v `[Engine.GameInfo]` a `[UTServerAdmin.UTServerAdmin]`): v repozitáři je zástupná hodnota `ChangeMe_BitcoreAdmin!` – **na produkčním serveru musí být vždy silné vlastní heslo**.

### Síť a výkon (OldUnreal v469)
* **`NetServerMaxTickRate=60` / `LanServerMaxTickRate=60`** – moderní plynulý tickrate pro veřejný Deathmatch/CTF.
* **`MaxClientRate=25000` / `MaxInternetClientRate=25000`** – plná propustnost pro dnešní připojení.
* **`AllowPlayerPortUnreach=False`** – zabraňuje nechtěnému vyhození hráče při drobném ICMP výpadku.
* **`MaxFileDownloadRate=-1`** – neomezená rychlost přímého stahování balíčků ze serveru (FastDL je nastaveno na `http://uz.ut-files.com/`).
* **`CacheSizeMegs=64`**, **`UseSound=False`**, **`MD5PostJoinCheck=True`**, **`MinClientVersion=436`**.

### Hratelnost a ochrana
* **Kapacita:** `MaxPlayers=10`, `MaxSpectators=4`.
* **Boti:** `MinPlayers=2`, `InitialBots=0`, `Difficulty=2`, `bNoviceMode=False` (server udržuje minimálně 2 hráče/boty a po připojení živých hráčů boty plynule uvolňuje).
* **FOV a míření:** `bAllowFOV=True` (povoleno v rozsahu `80`–`130`), `AutoAim=1.000000` (autoaim vypnutý).
* **Opravy v469:** `bSimulateKickers=True`, `bFixMultiWeaponBug=True`, `bFixFeignDeathZoomBug=True`.
* **Ochrana proti brute-force loginu:** `MaxLoginAttempts=5`, `LoginDelaySeconds=1.0`, `ActionToTake=DO_KickBanPlayer`.
* **Statistiky:** `bWorldLog=False` (vypnutý neexistující `ngWorldStats`, nezpůsobuje záseky na konci mapy).

### Master servery (`[Engine.GameEngine]`)
* `master.oldunreal.com:27900`
* `master2.oldunreal.com:27900`
* `master.333networks.com:27900`
* `master.errorist.eu:27900`
* `master.openspy.net:27900`

---

## 3. Map Vote Extended (`MVE2i`)

Na serveru je nainstalován **Map Vote Extended (MVE2i)** (`MVES.u`, `MVES.int`, `MVE2i.u` v `~/serverfiles/System/`).

### Zapojení v `ut99server.ini`
V sekci `[Engine.GameEngine]` běží MVE jako `ServerActor` (nikoli přes `?mutator=` v LGSM parametrech):
```ini
ServerActors=MVES.MapVote
ServerPackages=MVE2i
```
V `MVE_Config.ini` je nastaveno **`bOverrideServerPackages=False`**, aby si MVE nepřepisovalo seznam `ServerPackages` definovaný v `ut99server.ini`.

### Nakonfigurované herní režimy (`CustomGame[0..3]`)

| Slot | Mód | Pravidla | GameClass | Mutátor | Limity | Translocator |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `CustomGame[0]` | **DeathMatch** | Classic | `Botpack.DeathMatchPlus` | – | FragLimit 30, TimeLimit 15 | Ne |
| `CustomGame[1]` | **DeathMatch** | InstaGib | `Botpack.DeathMatchPlus` | `Botpack.InstaGibDM` | FragLimit 40, TimeLimit 15 | Ne |
| `CustomGame[2]` | **Capture the Flag** | Classic | `Botpack.CTFGame` | – | GoalTeamScore 5, TimeLimit 20 | Ano |
| `CustomGame[3]` | **Capture the Flag** | InstaGib | `Botpack.CTFGame` | `Botpack.InstaGibDM` | GoalTeamScore 7, TimeLimit 20 | Ne |

### Filtry map (`MapFilters`)
Aktuálně jsou povoleny pouze dvě ikonické mapy:
```ini
MapFilters[0]=dm DM-Deck16][
MapFilters[1]=ctf CTF-Face
```
> **Jak v budoucnu povolit další mapy:**
> V `~/serverfiles/System/MVE_Config.ini` změň filtry například na `MapFilters[0]=dm DM-*` a `MapFilters[1]=ctf CTF-*` (nebo přidej další `MapFilters[2]=...`), případně povol náhodnou mapu u módů pomocí `bHasRandom=True`.

### Další důležité volby v ostré záloze (`MVe_Config.backup.ini`)
* `VoteTimeLimit=45` – čas na hlasování v sekundách.
* `ScoreBoardDelay=5` – zpoždění před automatickým otevřením hlasování po konci mapy (`bAutoOpen=True`).
* `MidGameVotePercent=51` – nadpoloviční většina hráčů může odhlasovat změnu mapy i v průběhu zápasu.
* `bKickVote=True` (`KickPercent=51`) – povoleno hlasování o vyhození hráče.
* `bSpecsAllowed=False` – diváci (spectatoři) nemohou hlasovat.
* `ServerIdleAfterMinutes=60` – po 60 minutách nečinnosti se server přepne (momentálně `bSwitchToRandomMapOnIdle=True`, `bSwitchToDefaultMapOnIdle=False`; pokud se má vždy vracet na `DM-Deck16][` Classic, nastav `bSwitchToRandomMapOnIdle=False` a `bSwitchToDefaultMapOnIdle=True`).

---

## 4. Ovládání ve hře a správa v LXC (LGSM)

### Příkazy pro hráče a adminy ve hře (konzole `~` nebo `;`)
* **Otevření MapVote menu kdykoliv během hry:**
  ```text
  mutate bdbmapvote votemenu
  ```
* **Nabindování na klávesu (např. `F8`):**
  ```text
  set input f8 mutate bdbmapvote votemenu
  ```
* **Přihlášení admina a znovunačtení `MVE_Config.ini` bez restartu serveru:**
  ```text
  adminlogin <heslo>
  mutate bdbmapvote reload
  ```

### Příkazy na serveru (LXC pod uživatelem `ut99server`)
```bash
# Kontrola stavu a parametrů spuštění
./ut99server details

# Start / stop / restart serveru
./ut99server start
./ut99server stop
./ut99server restart

# Živá konzole serveru (odpojení pomocí Ctrl+B, pak D – NE Ctrl+C!)
./ut99server console

# Kontrola logu (MVE inicializace, načtené mapy, chyby)
tail -n 60 ~/log/console/ut99server-console.log
```

### Jak správně upravovat `MVE_Config.ini`
1. Pokud upravuješ `~/serverfiles/System/MVE_Config.ini` při vypnutém serveru, nastav v něm **`bReloadOnNextRun=True`**, aby si MVE při dalším startu znovu naskenovalo mapy a pravidla (po startu se samo vrátí na `False`).
2. Pokud upravuješ `MVE_Config.ini` za běhu serveru, po uložení souboru spusť ve hře jako admin `mutate bdbmapvote reload`.

