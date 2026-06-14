# 🧰 Toolbox Project Summary

## 📖 Overview

Toolbox è un'app CLI avanzata per sviluppatori che centralizza operazioni Git, gestione progetti, notifiche e automazioni di sistema su macOS. Include anche una mini app nativa per le notifiche (Toolbox Notifier), integrata nella menubar.

---

## 🧩 Architettura

**Struttura principale del progetto:**

```
home-scripts/
├── src/toolbox
│   ├── core/
│   │   ├── db.py
│   │   ├── notify.py
│   ├── ops/
│   │   ├── branches.py
│   │   ├── git_verify.py
│   │   ├── pull.py
│   │   ├── switch.py
│   │   └── scan.py
│   ├── cli_branches.py
│   ├── cli_git.py
│   ├── cli_projects.py
│   ├── cli_settings.py
│   └── cli.py
└── mac_app/
    ├── notifier_app.py
    ├── setup.py
    └── Info.plist
```

**Componenti principali:**

* `toolbox.core` → gestione DB (SQLite), notifiche e configurazioni.
* `toolbox.ops` → logica operativa (Git, branch, pull, switch, scan).
* `toolbox.cli_*` → comandi Click CLI modulari.
* `mac_app` → app nativa macOS con PyObjC e py2app.

---

## 🔔 Toolbox Notifier (mac_app)

App standalone macOS sviluppata in Python con `pyobjc` + `py2app`.

**Funzioni:**

* Invia notifiche native via `NSUserNotification` (macOS).
* Accetta parametri CLI (`--title`, `--message`, `--subtitle`).
* Build nativa `.app` generata con py2app.

**Build setup (`setup.py`):**

* Esclusione di `tkinter`, `Tcl`, `tk8.6` per evitare errori in codesign.
* Icona ufficiale Toolbox.icns integrata.
* Bundle name: *Toolbox Notifier.app*
* Firma automatica disattivata (signing manuale opzionale).

**Run manuale:**

```bash
python mac_app/notifier_app.py --title "Toolbox" --message "Build OK ✅" --subtitle "Bundle completo"
```

**Build:**

```bash
python mac_app/setup.py py2app
```

---

## ⚙️ CLI principale

### Moduli implementati

* **projects** → scansione repo, gestione DB, notifiche.
* **branches** → selezione branch remoti con `questionary`.
* **git** → verify, pull e switch automatizzati.
* **settings** → gestione configurazioni utente.

### Esempi CLI

```bash
# Scansione e registrazione progetti
toolbox projects scan --notify

# Verifica stato remoti e dirty
toolbox git verify --all --auto-ignore

# Pull di tutti i branch configurati
toolbox git pull --all --restore-dev

# Switch automatico a dev-branch
toolbox git switch --all --use-dev
```

---

## 🧠 Funzionalità chiave

* `auto-ignore` → marca repo con remote non raggiungibile.
* `--restore-dev` → dopo il pull torna al branch di sviluppo.
* `--all / --single` → supporto sia batch che interattivo.
* Notifiche asincrone tramite app nativa (`notifier_app.py`).

---

## 🧱 Tooling & Lint

* **Black**, **Ruff**, **Mypy** integrati in pre-commit.
* Configurazioni centralizzate in `pyproject.toml`.
* `pyobjc` installato solo su macOS (`platform_system == "Darwin"`).

---

## 🚀 Roadmap Feature future

| Area    | Feature                                            | Stato                |
| ------- | -------------------------------------------------- | -------------------- |
| UX      | Menubar app per interazione Toolbox                | 🟡 In pianificazione |
| Notify  | Rimozione icona secondaria / sinergia bundle       | ✅ Risolta            |
| Git     | Gestione merge safety & skip dirty                 | ✅ Implementata       |
| Build   | .pkg installer con symlink /usr/local/bin/toolbox  | 🔜                   |
| Release | Automazione release GH Actions + Changelog         | 🔜                   |
| Clean   | Aggiunta comando `toolbox clean` per repo multipli | 🟡                   |

---

## 🧩 Concept UX futura

* Icona nella menubar con badge attività Git.
* Pannello tipo *MacUpdater* con stato repo (branch, update, dirty, ecc.).
* Accesso rapido a CLI, configurazioni, notifiche.

---

## 💡 Considerazioni

Toolbox è diventato un framework CLI/macOS modulare e maturo, integrabile in ambienti di sviluppo professionali. È già operativo, stabile, e può essere esteso con componenti GUI o distribuito come pacchetto `.pkg` unico contenente CLI e Notifier.

> L’obiettivo futuro è una UX completa e un’integrazione trasparente tra CLI e App, mantenendo performance, modularità e standard qualitativi elevati.
