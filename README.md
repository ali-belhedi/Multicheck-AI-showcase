
**Plateforme d'audit de conformité ISO 27001:2022 propulsée par l'IA**

MultiCheck-AI est une application web qui automatise l'audit de conformité ISO 27001:2022 grâce à un pipeline RAG (Retrieval-Augmented Generation) hybride, combinant analyse sémantique, recherche BM25 et reclassement par cross-encoder.

---

## Fonctionnalités principales

- **Workflow en 11 phases** : de la création de la mission jusqu'au rapport final
- **Mapping IA objectifs → clauses ISO** : Groq llama-3.3-70b suggère les contrôles pertinents selon les objectifs personnalisés
- **Analyse documentaire RAG hybride** : pgvector + BM25 + cross-encoder + LLM
- **Génération de checklists** : 5 questions expertes par contrôle, ancrées dans les documents uploadés
- **Score de conformité** : méthode pondérée multi-dimensions (CMMI + ISO 27006)
- **Fiches de constats automatiques** : NC majeure/mineure/observation avec matrice de risque ISO 27005
- **Human-in-the-loop** : validation auditeur à chaque étape critique
- **Fallback LLM** : Groq Cloud → Ollama local (offline, confidentialité)

---

## Stack technique

| Couche | Technologie |
|---|---|
| Frontend | React.js 19, Vite 8, Tailwind CSS 3 |
| Backend | FastAPI 0.129, Python 3.11+ |
| Base de données | PostgreSQL 15 + pgvector 0.4.2 |
| ORM / Migrations | SQLAlchemy 2.0, Alembic 1.18 |
| Embedding | intfloat/multilingual-e5-base (768 dims) |
| Reranking | cross-encoder/mmarco-mMiniLMv2-L12-H384-v1 |
| Orchestration RAG | LangChain 1.2, LangGraph 1.0 |
| LLM principal | Groq Cloud (llama-3.3-70b-versatile) |
| LLM fallback | Ollama local (qwen2.5:3b) |
| Authentification | JWT + MFA (TOTP / Email) |

---

## Prérequis

- Python 3.11+
- Node.js 18+
- PostgreSQL 15+ avec extension pgvector
- Ollama (optionnel, pour le mode offline)
- Clé API Groq (gratuite sur [console.groq.com](https://console.groq.com))

---

## Installation

### 1. Cloner le dépôt

```bash
git clone https://github.com/<votre-username>/MultiCheck-AI.git
cd MultiCheck-AI
```

### 2. Backend

```bash
cd backend
python -m venv ../venv
../venv/Scripts/activate      # Windows
# source ../venv/bin/activate  # Linux/Mac

pip install -r requirements.txt
```

Créer le fichier `backend/.env` :

```env
DATABASE_URL=postgresql://user:password@localhost:5432/multicheck
GROQ_API_KEY=your_groq_api_key
GROQ_MODEL=llama-3.3-70b-versatile
GROQ_MODEL_FAST=llama-3.1-8b-instant
OLLAMA_BASE_URL=http://127.0.0.1:11434
OLLAMA_MODEL=qwen2.5:3b
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your_email@gmail.com
SMTP_PASSWORD=your_app_password
```

Initialiser la base de données :

```bash
# Créer la base PostgreSQL
psql -U postgres -c "CREATE DATABASE multicheck;"
psql -U postgres -c "CREATE EXTENSION IF NOT EXISTS vector;" -d multicheck

# Appliquer les migrations
alembic upgrade head
```

Lancer le backend :

```bash
cd backend
uvicorn app.main:app --reload --port 8000
```

### 3. Frontend

```bash
cd frontend
npm install
npm run dev
```

L'application est accessible sur `http://localhost:5173`.

---

## Architecture RAG

```
Documents uploadés
       ↓
  Extraction texte (PyMuPDF, python-docx, Tesseract OCR...)
       ↓
  Chunking (1800 chars, overlap 250, par paragraphe)
       ↓
  Embedding : intfloat/multilingual-e5-base (768 dims)
       ↓
  Stockage : PostgreSQL + pgvector

Analyse d'un contrôle ISO :
  Query → embed_query → cosinus pgvector (top 15)
                     + BM25 (top 15)
                     → RRF Fusion (top 20)
                     → Cross-encoder reranking (top 3-8)
                     → Prompt LLM → Groq / Ollama
```

---

## Workflow des 11 phases

| Phase | Description |
|---|---|
| 1 | Création de la mission et sélection du périmètre |
| 2 | Contextualisation (organisation, secteur, objectifs) |
| 3 | Alignement ISO : mapping objectifs → clauses |
| 4 | Upload et vectorisation des documents |
| 5 | Génération des checklists par IA |
| 6 | Analyse IA des réponses de checklist |
| 7 | Tableau de bord de conformité |
| 8 | Révision auditeur (Human-in-the-loop) |
| 9 | Génération des fiches de constats |
| 10 | Score de vulnérabilité |
| 11 | Rapport final |

---

## Référentiels normatifs

- **ISO/IEC 27001:2022** — Système de management de la sécurité de l'information
- **ISO/IEC 27005:2022** — Gestion du risque (matrice impact × probabilité)
- **ISO/IEC 27006:2015** — Classification NC majeure / mineure / observation
- **ISO 19011:2018** — Structure des fiches de constat
- **CMMI** — Niveaux de maturité 0→4

---

## Développé dans le cadre de

Projet de Fin d'Études — Master Professionnel Audit et Sécurité Informatique  
Université de Sfax, Faculté FSEGS  
En collaboration avec **Hat Web Security**

---

## Licence

Ce projet est développé à des fins académiques.
