# RH Paie Maroc v5.0.0 — Compta Management Paie RH

Système intégré **RH · Paie · Comptabilité CGNC partie double · EDI Maroc · TVA · Liasse fiscale · IS · CM**.

> Repo construit par application du `TODO_PROFESSIONAL.md` (v4.10.1 → v5.0.0) puis des phases d'amélioration 01, 02, 03.

## Stack

- **Python 3.12** · **FastAPI** · **SQLite** (WAL) · **Jinja2** · **ReportLab** · **PyQt5** (desktop optionnel)
- **Pydantic v2** · **python-dotenv** · **slowapi** · **secure** · **bleach** · **cryptography** · **loguru** · **cachetools** · **lxml**

## Phases appliquées

| Phase | Scope | Fichiers | Lignes |
|-------|-------|----------|--------|
| **v5.0.0 baseline** | Sécurité (CSRF, rate limiting, secure headers), architecture (config/, core/, schemas/, routes/, utils/), migrations 001-008, tests, Docker, CI/CD, docs | ~280 | ~30 000 |
| **Phase 01** | Noyau comptable partie double CGNC + `Money` (Decimal) | 22 | ~4 500 |
| **Phase 02** | TVA marocaine (Loi 30-85) — taux, prorata, autoliquidation, T1/T2/T3, XML CMC | 16 | ~4 800 |
| **Phase 03** | Liasse fiscale (forms 1XX/2XX/3XX), IS post-LF 2024, CM Art. 33, reports déficitaires, SIMPL-IS XML | 16 | ~4 400 |

Voir `PHASE01_REPORT.md`, `PHASE02_REPORT.md`, `PHASE03_REPORT.md` pour le détail.

## Structure

```
rh_system/
├── config/              # Settings Pydantic (P0-02)
├── core/                # money.py, exceptions, security, migrator
├── accounting/          # Phase 01: double-entry kernel
│   ├── accounts.py      # Plan comptable CGNC hiérarchique (94 comptes)
│   ├── ledger.py        # Posting / Transaction / Ledger.post()
│   ├── journal.py       # 6 journaux VE/AC/BQ/CA/OD/PA
│   ├── periods.py       # Périodes comptables open/close
│   ├── analytical.py    # Comptes divisionnaires multi-axes
│   ├── reconciliation.py # Lettrage 411/401
│   ├── ledger_service.py # AccountingService facade
│   ├── tva/             # Phase 02: TVA marocaine
│   │   ├── rates.py     # 6 taux (20/14/10/7/0/0%)
│   │   ├── engine.py    # Prorata, autoliquidation, calcul TVA due
│   │   └── declarations.py # T1/T2/T3 + XML CMC DGI
│   └── liasse_fiscale/  # Phase 03: Liasse + IS + CM
│       ├── forms.py     # Forms 1XX/2XX/3XX (51 lines bootstrap)
│       ├── engine.py    # LiasseEngine + tableau passage RC→RF
│       ├── is_engine.py # IS post-LF 2024 (10%/20%/31%) + déficits 4 ans
│       ├── cm_engine.py # Cotisation Minimum Art. 33 CGI
│       └── declarations.py # SIMPL-IS XML + workflow paiement
├── routes/              # APIRouters /api/v1/*
├── schemas/             # Pydantic I/O
├── utils/               # logger, cache, csrf
├── migrations/          # SQL 001-011
├── tests/               # pytest
├── scripts/             # backup_db, seed_demo, run_migrations, generate_secret
├── docs/                # DEPLOIEMENT, GUIDE_UTILISATEUR, GUIDE_DEVELOPPEUR, API_REFERENCE, RGPD_09-08, VALIDATION_EDI_XSD, MIGRATION_V5
├── web/templates/       # Jinja2 (compta v2, tva, liasse)
├── Dockerfile
├── docker-compose.yml
├── pyproject.toml
└── Makefile
```

## Démarrage rapide

```bash
# 1. Installer
pip install -e ".[dev]"

# 2. Configurer .env
cp .env.example .env
python scripts/generate_secret.py >> .env

# 3. Migrations
python scripts/run_migrations.py

# 4. Lancer
make dev  # ou: uvicorn web_app:app --reload --host 0.0.0.0 --port 8000
```

## Conformité

- **CGNC** — Code Général de Normalisation Comptable marocain
- **Loi 30-85** — TVA marocaine
- **Loi 24-22 (CGI)** — Impôt sur les Sociétés (post-LF 2024)
- **Loi 09-08** — Protection des données personnelles (CNDP)
- **Code du Travail** — Loi 65-99

## Licence

Voir `LICENCE` (à définir).
