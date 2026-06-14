# 🤖 AI-Core (AI Secretary) — Infrastructure & Setup Summary

## 📖 Overview

**AI-Core** è il cuore infrastrutturale del progetto *AI Secretary*: una piattaforma AI modulare che funge da orchestratore per modelli locali (Ollama, Whisper, ecc.), servizi dockerizzati e gestione di rete interna. È progettato per operare in un **home-lab** privato, ma con architettura scalabile per ambienti cloud.

---

## 🧩 Obiettivi principali

1. Centralizzare i servizi AI (LLM, TTS, STT, orchestrazione vocale).
2. Garantire alta affidabilità e backup multilivello.
3. Offrire DNS e routing locale per tutto l’ecosistema (CoreDNS + Traefik).
4. Fornire una base di sviluppo condivisa accessibile via VPN (Tailscale + Headscale).

---

## 🧱 Infrastruttura attuale

### Hardware

* **Server principale:** HP ProDesk 600 G3 DM

  * CPU: Intel i5-7500T (4 core @ 3.3 GHz)
  * RAM: 32 GB
  * Storage:

    * SSD 1 TB (sistema + docker)
    * RAID 5 (5x2TB HDD ≈ 5.5 TB effettivi)
    * HDD esterno 2.7 TB per backup Borg
* **OS:** Ubuntu 24.04.3 LTS
* **Docker:** installato e funzionante
* **Servizi residui:** Jellyfin + MySpeed mantenuti

### Rete & DNS

```
[Internet]
   │
[Iliad Router 192.168.1.1]
   ├───> [QNAP Switch 2.5 Gb] → ai-core (192.168.1.46)
   ├───> Pro 16 / PC i7 / NAS (LAN 192.168.1.x)
   └───> [TP-Link WiFi 7] → MacBook Air (Wi-Fi 192.168.0.x)
```

* Iliad gestisce DHCP.
* TP-Link fa DNS passthrough.
* ai-core ha IP statico `192.168.1.46`.
* VPN privata tramite Headscale/Tailscale (dominio `home-lab.internal`).

---

## ⚙️ Servizi pianificati

| Stack   | Servizio                | Note                                             |
| ------- | ----------------------- | ------------------------------------------------ |
| Core    | **CoreDNS**             | Risoluzione DNS locale *.home-lab → 192.168.1.46 |
| Core    | **Traefik + step-ca**   | Reverse proxy + TLS interno                      |
| AI      | **Ollama**              | LLM locale (modelli Qwen, Mistral, ecc.)         |
| AI      | **Whisper.cpp**         | Speech-to-text locale                            |
| AI      | **Coqui / Piper TTS**   | Sintesi vocale locale                            |
| DB      | **PostgreSQL 17**       | Database primario (data su RAID, WAL su SSD)     |
| Backup  | **BorgBackup + Cubbit** | Backup RAID + offsite versionato                 |
| Utility | **Homepage dashboard**  | Accesso rapido a tutte le UI interne             |

---

## 🔒 Sicurezza e rete

* Tutte le porte chiuse tranne SSH (22) — accesso solo da IP autorizzati.
* UFW e fail2ban configurati.
* DNS primario impostato su `192.168.1.46` (fallback: 1.1.1.1).
* CoreDNS in modalità host per servire sia LAN (192.168.1.x) che WiFi (192.168.0.x).

---

## 🧠 Backup Strategy

**Tripla sicurezza:**

1. RAID 5 per resilienza hardware.
2. BorgBackup giornaliero → `/mnt/backup/borg`.
3. Upload automatico su **Cubbit** per versionamento offsite.

### Borg Script (`/usr/local/sbin/backup_nas.sh`)

```bash
borg create --stats --compression zstd "$REPO::$HOSTNAME-$DATE" "$SOURCE"
borg prune -v --list "$REPO" --keep-daily=5 --keep-weekly=2 --keep-monthly=1
echo "Backup completato su $HOSTNAME $DATE" | mail -s "[BACKUP] OK $HOSTNAME" "$MAILTO"
```

---

## 🧩 Rete & DNS (CoreDNS)

**Zona:** `home-lab`

```
*.home-lab → 192.168.1.46
@ → 192.168.1.46
```

**Fallback:** `1.1.1.1`, `8.8.8.8`
**Verifica:**

```bash
dig ai-core.home-lab @192.168.1.46 +short  # deve restituire 192.168.1.46
```

---

## 🔧 Pianificazione installazione

1. **BIOS Fix:** risolvere blocco boot con box USB RAID.
2. **Zsh & dotfiles:** configurazione personalizzata Tony.
3. **CoreDNS Setup:** `/opt/ai-core/services/coredns/`.
4. **DNS Router Iliad:** primario → 192.168.1.46, secondario → 1.1.1.1.
5. **Traefik + step-ca:** generazione automatica certificati validi in rete privata.
6. **PostgreSQL 17:** configurazione dual-storage (RAID + SSD WAL).
7. **Borg + Cubbit:** verifica backup integrato.
8. **Homepage Dashboard:** UI unica per tutti i container.

---

## 🧭 Roadmap generale

| Fase             | Attività                              | Stato       |
| ---------------- | ------------------------------------- | ----------- |
| 🔹 CoreDNS       | DNS locale + propagazione Wi-Fi/LAN   | 🟡 In corso |
| 🔹 Traefik       | Reverse proxy + TLS interno           | 🔜          |
| 🔹 PostgreSQL 17 | Migrazione e configurazione dual-disk | 🔜          |
| 🔹 Backup        | Borg + Cubbit integrazione automatica | 🔜          |
| 🔹 AI Stack      | Ollama, Whisper, TTS locali           | 🔜          |
| 🔹 Dashboard     | Homepage centralizzata                | 🔜          |

---

## 💡 Considerazioni finali

AI-Core diventa il **centro neurale** dell’intero ecosistema: un server intelligente, autonomo e auto-ripristinante, con stack completamente locale e open source. L’obiettivo è mantenere la massima sicurezza e privacy senza sacrificare potenza o flessibilità.

> Prossimo step immediato: deploy di CoreDNS e validazione routing `.home-lab` da entrambe le subnet.
