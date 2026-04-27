# LinkedIn AI Content Automation - Workflow n8n

> Projet ESGI - M1 T2 - Usages IA Générative (2025-2026)
> Matière : IA dans les logiciels | Enseignant : DELANDHUY Matteo

## Objectif

Automatiser entièrement la création et la publication de posts LinkedIn quotidiens en exploitant plusieurs IA génératives. Le workflow génère chaque jour **2 posts** (un politique, un tech) avec des visuels personnalisés, en s'appuyant sur l'actualité en temps réel et l'analyse des performances passées.

Ce projet illustre un cas d'usage où **l'IA est au coeur de la technologie** : sans IA, le logiciel n'existerait tout simplement pas. Chaque étape clé (recherche d'actualités, rédaction, détection de doublons, génération d'images) repose sur un modèle d'IA distinct.

## Architecture du Workflow

```
⏰ Trigger (8h)
    │
    ▼
🔍 Perplexity ─── Recherche des actualités (politique + tech)
    │
    ▼
📊 Parser les actus ─── Extraction JSON, tri par potentiel viral
    │
    ▼
❓ Parse OK ? ──── Oui ──► 📊 Analyser top posts (Qdrant)
    │                              │
    Non                            ▼
    │                      🤖 Gemini ─── Génération de 2 posts LinkedIn
    ▼                              │
⚠️ Erreur                         ▼
                           ✂️ Parser les 2 posts
                                   │
                                   ▼
                           🔍 Check Qdrant ─── Détection de doublons (embeddings)
                                   │
                              ┌────┴────┐
                              │         │
                           Original   Doublon
                              │         │
                              │    🔄 Régénérer post (max 2 tentatives)
                              │         │
                              │    ❓ Régénéré OK ?
                              │    ┌────┴────┐
                              │    Oui      Non
                              │    │    ⚠️ Rejeté
                              ▼    ▼
                           🔀 Politique ou Tech ?
                           ┌────────┴────────┐
                           │                 │
                    📥 Template          📥 Template
                      Politique             Tech
                           │                 │
                    🔧 Préparer         🔧 Préparer
                    Gemini Edit         Gemini Edit
                           │                 │
                    🎨 Gemini Edit      🎨 Gemini Edit
                    (génération img)    (génération img)
                           │                 │
                    🖼️ Extraire         🖼️ Extraire
                      image               image
                           │                 │
                    ☁️ Cloudinary       ☁️ Cloudinary
                    (upload)            (upload)
                           │                 │
                    📤 Buffer           📤 Buffer
                    (MCP Client)        (MCP Client)
                    Publication         Publication
```

## Stack Technique

| Composant | Technologie | Rôle |
|-----------|------------|------|
| **Orchestrateur** | n8n | Workflow automation, scheduling, routing |
| **Recherche d'actualités** | Perplexity AI | Veille temps réel (politique + tech, dernières 48h) |
| **Rédaction des posts** | Google Gemini 2.5 Flash Lite | Génération de contenu LinkedIn viral |
| **Génération d'images** | Google Gemini 3.1 Flash Image Preview | Édition de templates visuels avec texte dynamique |
| **Base vectorielle** | Qdrant | Stockage d'embeddings, détection de doublons, analyse de performance |
| **Embeddings** | Gemini Embedding 001 | Vectorisation des posts pour comparaison sémantique |
| **Stockage d'images** | Cloudinary | Hébergement des visuels générés + banque de photos |
| **Publication** | Buffer (MCP Client) | Planification et publication sur LinkedIn |

## Fonctionnalités Détaillées

### 1. Veille automatique (Perplexity)
- Exécution quotidienne à 8h via un Schedule Trigger
- Recherche des 5 actualités politiques + 5 actualités tech les plus importantes des dernières 48h
- Chaque actualité est enrichie d'un score de potentiel viral (fort/moyen/faible)

### 2. Analyse des performances passées (Qdrant)
- Récupération de l'historique des posts depuis la base vectorielle Qdrant
- Calcul d'un score composite de viralité (engagement rate, réactions, commentaires, impressions)
- Extraction des patterns des top 5 et bottom 5 posts (hooks, longueur, type)
- Ces insights alimentent le prompt de Gemini pour améliorer la qualité des posts générés

### 3. Génération de contenu (Gemini)
- Prompt de ghostwriting avec profil auteur détaillé (ton, valeurs, style)
- Génération de 2 posts : un politique, un tech
- Format viral LinkedIn : hook percutant, corps structuré, CTA, hashtags
- Contraintes : 800-1500 caractères, tutoiement, ton cash et provocateur

### 4. Détection de doublons (Qdrant + Embeddings)
- Chaque post généré est vectorisé via Gemini Embedding
- Recherche de similarité dans Qdrant (seuil : 0.85)
- Si doublon détecté : régénération automatique (max 2 tentatives avec un angle différent)
- Si toujours doublon après régénération : rejet définitif du post

### 5. Génération d'images (Gemini Image)
- Téléchargement de templates visuels brandés depuis Cloudinary
- Pour le post politique : sélection aléatoire d'une photo depuis une banque Cloudinary, intégrée au template
- Pour le post tech : édition du template avec titre et sous-titre dynamiques
- Génération via Gemini 3.1 Flash Image Preview (multimodal)

### 6. Publication (Buffer via MCP)
- Upload des images générées sur Cloudinary
- Envoi des posts en draft sur Buffer via le protocole MCP (Model Context Protocol)
- Ajout automatique en queue avec tags de catégorisation (politique / tech)
- Publication automatique selon le calendrier Buffer

## Prérequis

- **n8n** (self-hosted ou cloud)
- **Qdrant** (base vectorielle, accessible sur `qdrant:6333`)
- Clés API :
  - Perplexity API
  - Google Gemini API
  - Cloudinary (API key + secret)
  - Buffer (Bearer token)

## Installation

1. Importer le fichier `workflow.json` dans n8n
2. Configurer les credentials :
   - Perplexity API
   - Google Gemini (PaLM API)
   - Buffer Bearer Auth
3. S'assurer que Qdrant est accessible avec la collection `linkedin_posts` créée
4. Configurer les templates d'images sur Cloudinary
5. Activer le workflow, il se déclenchera automatiquement chaque jour à 8h

## Livrables du Projet

| Livrable | Description | Date |
|----------|------------|------|
| Rendu final | Repo GitHub + Diapo de présentation | 20/04/2026 |
| Soutenance | Présentation PowerPoint (5 min) devant la promotion | 23/04/2025 |
