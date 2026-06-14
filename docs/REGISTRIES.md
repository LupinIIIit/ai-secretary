# REGISTRIES.md

> Catalogo sorgenti immagini + policy di pin per **AI-Secretary (AI-Core)** e servizi App.
> Obiettivo: evitare `latest`, sorprese da major upgrade, immagini abbandonate e lock-in su un solo registry.

---

## 1) Ordine di preferenza registry (vincolante)

1. **GHCR** → `ghcr.io`
2. **Docker Hub (official/library/maintained)** → `docker.io`
3. **Quay** → `quay.io`
4. **Privato** → `registry.server-home.ovh` (login richiesto)

> Evitare immagini **Bitnami** salvo necessità specifiche (opzioni invasive).

---

## 2) Regole di pin (vincolanti)

* **Mai `latest`**. Pin esplicito a **major.minor.patch** oppure **release tag** documentato.
* Prima di proporre un tag:

  1. Controlla *tutti* i registry possibili nell’ordine sopra.
  2. Verifica `docker pull` (immagine realmente esistente).
  3. Smoke test: `--version` o health endpoint.
  4. Leggi note di rilascio / breaking changes.
* In produzione critica aggiungere **digest** `@sha256:…`.

*Comandi utili*

```bash
skopeo list-tags docker://ghcr.io/<org>/<img>
crane ls ghcr.io/<org>/<img> | tail -n 20
skopeo inspect docker://ghcr.io/<org>/<img>:<tag>
docker run --rm <image>:<tag> --version || true
curl -sI http://host:port/health || true
```

---

## 3) Immagini — AI-Core (verificate 2025-11)

| Servizio                                 | Registry preferito                | Ultima versione stabile        | Alternative                                | Note                                                   |
| ---------------------------------------- | --------------------------------- | ------------------------------ | ------------------------------------------ | ------------------------------------------------------ |
| **Ollama (runtime LLM)**                 | `docker.io/ollama/ollama`         | `v0.12.10`                     | —                                          | GHCR non disponibile; usare Docker Hub.                |
| **Open WebUI (chat web)**                | `ghcr.io/open-webui/open-webui`   | `v0.3.17`                      | Mirror non ufficiali su Docker Hub         | Ufficiale GHCR.                                        |
| **Whisper STT (Wyoming Faster-Whisper)** | `ghcr.io/rhasspy/wyoming-whisper` | `v1.6.0`                       | `docker.io/rhasspy/wyoming-whisper:v1.6.0` | GHCR preferito.                                        |
| **Piper TTS**                            | `ghcr.io/rhasspy/piper`           | `v1.2.1`                       | `docker.io/rhasspy/piper:v1.2.1`           | GHCR di solito più fresco.                             |
| **MinIO (S3 – AI)**                      | `ghcr.io/coollabsio/minio`        | `RELEASE.2025-10-15T17-29-55Z` | —                                          | MinIO ha cessato pubblicazione regolare su Docker Hub. |
| **Redis (AI)**                           | `docker.io/library/redis`         | `8.2.3-alpine3.22`             | —                                          | Volatile (no AOF).                                     |
| **PostgreSQL (AI)**                      | `docker.io/library/postgres`      | `18-alpine3.22`                | —                                          | LTS attuale.                                           |
| **CoreDNS**                              | `ghcr.io/coredns/coredns`         | `1.13.1`                       | `docker.io/coredns/coredns:1.13.1`         | Ultima serie 1.13.x.                                   |
| **Traefik**                              | `docker.io/library/traefik`       | `v3.5.4`                       | `ghcr.io/traefik/traefik:v3.5.4`           | Serie supportata 3.5.x.                                |

---

## 4) Immagini — App / Dev (verificate 2025-11)

| Servizio             | Registry preferito                         | Ultima versione stabile | Alternative                         | Note                                          |
| -------------------- | ------------------------------------------ | ----------------------- | ----------------------------------- | --------------------------------------------- |
| **Gitea**            | `docker.gitea.com/gitea`                   | `1.25.1`                | `ghcr.io/go-gitea/gitea:1.25.1`     | Preferire registry proprietario.              |
| **Gitea Act Runner** | `docker.gitea.com/act_runner`              | `0.3.4`                 | `ghcr.io/go-gitea/act_runner:0.3.4` | —                                             |
| **Vikunja**          | `ghcr.io/go-vikunja/vikunja`               | `0.24.1`                | `docker.io/vikunja/vikunja:0.24.1`  | GHCR preferito.                               |
| **Docmost**          | `ghcr.io/docmost/docmost`                  | `0.12.0`                | `docker.io/docmost/docmost:0.12.0`  | GHCR più fresco.                              |
| **Homepage**         | `ghcr.io/gethomepage/homepage`             | `v1.6.1`                | —                                   | Pull verificato.                              |
| **PocketBase**       | `registry.server-home.ovh/<ns>/pocketbase` | `TBD (build privata)`   | —                                   | Useremo build privata (GHCR non disponibile). |
| **Redis (App)**      | `docker.io/library/redis`                  | `8.2.3-alpine3.22`      | —                                   | Con AOF on.                                   |
| **PostgreSQL (App)** | `docker.io/library/postgres`               | `18-alpine3.22`         | —                                   | Official library.                             |
| **RedisInsight**     | `docker.io/redis/redisinsight`             | `2.66.1`                | —                                   | Repo ufficiale Docker.                        |
| **Dozzle**           | `ghcr.io/amir20/dozzle`                    | `v6.5.3`                | `docker.io/amir20/dozzle:v6.5.3`    | Leggero log viewer.                           |

---

## 5) Strategie di pin consigliate

* **Traefik** → `v3.6.1`
* **CoreDNS** → `1.13.1`
* **PostgreSQL** → `17.2`
* **Redis** → `7.4.1` / `7.4.1-alpine`
* **MinIO** → `RELEASE.2025-10-15T17-29-55Z`
* **Open WebUI** → `v0.3.17`
* **Ollama** → `v0.5.4`
* **Whisper / Piper** → `v1.6.0` / `v1.2.1`
* **Homepage** → `v0.9.12`
* **Gitea** → `1.25.1`

---

## 6) Procedura aggiornamenti immagini

1. Branch: `chore/image-pins-YYYYMMDD`.
2. Per ogni immagine: ricerca registry, `docker pull`, smoke test, changelog.
3. Aggiorna compose/k8s + questo file (se cambia registry/strategia), **no latest**.
4. Deploy uno stack alla volta (AI poi App, o viceversa).
5. Aggiorna `CHANGELOG.md`.

---

## 7) Note “compatibilità major”

* Esempio: **Postgres 15 → 17** con `latest`: il container **non parte** con data dir incompatibile; serve `pg_dump/restore` o `pg_upgrade` e pin controllato.
* Stessa attenzione per Traefik, CoreDNS, Gitea, Redis e PocketBase.
