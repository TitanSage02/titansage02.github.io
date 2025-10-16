---
layout: ../../layouts/BlogPost.astro
title: "BioStar : Plateforme IA pour la Recherche en Biologie Spatiale"
description: "Système RAG intelligent utilisant Mistral AI et FAISS pour démocratiser l'accès aux publications scientifiques de la NASA sur la biologie spatiale."
date: "2025-10-01"
category: "Intelligence Artificielle"
tags: ["RAG", "NLP", "Mistral AI", "FAISS", "NASA", "Next.js", "FastAPI"]
author: "Espérance AYIWAHOUN"
---

## 🌌 Introduction

**BioStar** est une plateforme web intelligente développée dans le cadre du **NASA Space Apps Challenge 2025**. Elle utilise l'IA et le RAG (Retrieval Augmented Generation) pour rendre les recherches scientifiques de la NASA sur la biologie spatiale accessibles et interactives pour les chercheurs, étudiants et passionnés d'espace.

**🔗 Démo en ligne :** [biostarapp.vercel.app](https://biostarapp.vercel.app)  
**📦 Code source :** [GitHub](https://github.com/TitanSage02/BioStar-NASA-Space-Apps-Challenge-2025)

---

## 📋 Problématique

Les publications scientifiques de la NASA sur la biologie spatiale sont :
- **Volumineuses** et difficiles à naviguer (1k+ documents)
- **Techniques** avec un jargon scientifique complexe
- **Dispersées** sur plusieurs plateformes
- **Peu accessibles** aux non-experts

**Solution proposée** : Une interface conversationnelle alimentée par l'IA qui permet d'interroger instantanément l'ensemble du corpus scientifique en langage naturel.

---

## 🏗️ Architecture Technique

### Stack Technologique

#### Backend (API RAG)
- **Framework :** FastAPI (Python 3.9+)
- **Embeddings :** Sentence Transformers (`all-MiniLM-L6-v2`)
- **Recherche vectorielle :** FAISS (Facebook AI Similarity Search)
- **LLM :** Mistral AI API avec fallback intelligent
- **Traitement de texte :** PyPDF2, langdetect

#### Frontend
- **Framework :** Next.js 15 (App Router)
- **Language :** TypeScript
- **Styling :** TailwindCSS
- **UI Components :** Radix UI
- **Icônes :** Lucide React

#### Déploiement
- **Frontend :** Vercel
- **Backend :** Render
- **Stockage :** Embeddings pré-calculés en production

### Architecture RAG

```
┌─────────────┐
│   User      │
│   Query     │
└──────┬──────┘
       │
       v
┌─────────────────────────────────────┐
│  1. Vectorisation de la requête     │
│     (Mistral Embed - 1024D)         │
└──────┬──────────────────────────────┘
       │
       v
┌─────────────────────────────────────┐
│  2. Recherche de similarité         │
│     (FAISS - Distance cosinus)      │
└──────┬──────────────────────────────┘
       │
       v
┌─────────────────────────────────────┐
│  3. Extraction des passages (Top-K) │
│     + Calcul des scores             │
└──────┬──────────────────────────────┘
       │
       v
┌─────────────────────────────────────┐
│  4. Génération de réponse           │
│     (Mistral AI avec contexte)      │
└──────┬──────────────────────────────┘
       │
       v
┌─────────────────────────────────────┐
│     Réponse + Sources               │
└─────────────────────────────────────┘
```

---

## ✨ Fonctionnalités Principales

### 1. Recherche Sémantique Intelligente

- **Embedding multilingue** : Support anglais/français
- **Recherche vectorielle haute performance** : FAISS indexation
- **Top-K retrieval** : Extraction des passages les plus pertinents
- **Calcul de scores de pertinence** : Distance cosinus normalisée

**Exemple de requête :**
```
"Comment les plantes s'adaptent-elles à la microgravité ?"
```

### 2. Questions-Réponses Interactives

- **Génération contextuelle** : Réponses basées sur les documents scientifiques
- **Citations automatiques** : Références aux sources exactes
- **Explications détaillées** : Vulgarisation scientifique
- **Support multilingue** : Requêtes en anglais ou français

### 3. Quiz Auto-Générés

- **Génération dynamique** : QCM basés sur le contenu scientifique
- **Questions à choix multiples** : 4 options par question
- **Explications détaillées** : Justification de chaque réponse
- **Suivi de progression** : Statistiques de réussite

### 4. Découverte de Concepts

- **Extraction de mots-clés** : Identification des concepts principaux
- **Navigation thématique** : Exploration par sujets
- **Graphe de connaissances** : Relations entre concepts

---

## 🔧 Implémentation Technique

### Endpoint `/api/query` (Recherche RAG)

```python
@app.post("/api/query")
async def query_documents(request: QueryRequest):
    # 1. Vectorisation de la requête
    query_embedding = get_embedding(request.query)
    
    # 2. Recherche FAISS (Top 5)
    distances, indices = index.search(query_embedding, k=5)
    
    # 3. Extraction des passages pertinents
    contexts = [chunks[i] for i in indices[0]]
    
    # 4. Construction du prompt augmenté
    prompt = f"""
    Contexte : {' '.join(contexts)}
    
    Question : {request.query}
    
    Réponds de manière concise en te basant uniquement sur le contexte.
    """
    
    # 5. Génération avec Mistral AI
    response = mistral_client.chat(
        model="mistral-small-latest",
        messages=[{"role": "user", "content": prompt}]
    )
    
    return {
        "answer": response.choices[0].message.content,
        "sources": contexts,
        "confidence": calculate_confidence(distances)
    }
```

### Génération d'Embeddings

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('all-MiniLM-L6-v2')

def get_embedding(text: str) -> np.ndarray:
    """Génère un vecteur d'embedding 384D."""
    return model.encode([text])[0]
```

### Indexation FAISS

```python
import faiss

# Création de l'index
dimension = 384
index = faiss.IndexFlatIP(dimension)  # Inner Product

# Normalisation des vecteurs (pour distance cosinus)
faiss.normalize_L2(embeddings)

# Ajout des embeddings
index.add(embeddings)
```

---

## 📊 Résultats et Performance

### Métriques Techniques

| Métrique | Valeur | Commentaire |
|----------|--------|-------------|
| **Temps de recherche** | ~50ms | FAISS sur 10,000 documents |
| **Latence API** | ~1.5s | Génération Mistral AI incluse |
| **Taux de pertinence** | 87% | Évaluation manuelle sur 100 requêtes |
| **Support multilingue** | ✅ | Français + Anglais |

### Cas d'Usage Validés

✅ **Chercheurs** : Recherche rapide de publications pertinentes  
✅ **Étudiants** : Apprentissage interactif de la biologie spatiale  
✅ **Éducateurs** : Création de contenus pédagogiques avec quiz  
✅ **Data Scientists** : Exploration d'applications RAG

---

## 🚀 Installation et Déploiement

### Prérequis

- Node.js 18+
- Python 3.10+
- Clé API Mistral ([Obtenir gratuitement](https://console.mistral.ai/))

### Installation Locale

```bash
# 1. Cloner le repository
git clone https://github.com/TitanSage02/BioStar-NASA-Space-Apps-Challenge-2025.git
cd BioStar

# 2. Backend
cd backend
pip install -r requirements.txt
# Configurer .env avec MISTRAL_API_KEY
uvicorn main:app --reload --port 8000

# 3. Frontend (nouveau terminal)
cd ../frontend
pnpm install
pnpm dev
```

**Accès :** [http://localhost:3000](http://localhost:3000)

---

## 💡 Innovations et Contributions

### 1. Stratégie de Fallback Intelligente

En cas d'échec de Mistral AI, un système TF-IDF (scikit-learn) prend le relais :

```python
def fallback_search(query: str, documents: list):
    from sklearn.feature_extraction.text import TfidfVectorizer
    
    vectorizer = TfidfVectorizer()
    tfidf_matrix = vectorizer.fit_transform(documents)
    query_vec = vectorizer.transform([query])
    
    scores = cosine_similarity(query_vec, tfidf_matrix)
    return documents[scores.argmax()]
```

### 2. Interface Minimaliste (Inspired by Google Chrome)

- **Zero distraction** : Focus sur le contenu
- **Responsive design** : Mobile-first
- **Animations fluides** : Transitions CSS optimisées
- **Accessibilité** : WCAG 2.1 AA compliant

### 3. Optimisation des Coûts

- **Embeddings pré-calculés** : Pas de calcul en temps réel
- **Cache intelligent** : Réduction des appels API
- **Modèle gratuit** : Mistral AI free tier

---

## 🔮 Perspectives d'Amélioration

### Court Terme
- [ ] Support de plus de langues (Espagnol, Arabe)
- [ ] Graphes de connaissances interactifs
- [ ] Export PDF des réponses

### Moyen Terme
- [ ] Fine-tuning d'un modèle LLM dédié
- [ ] Système de recommandation d'articles
- [ ] API publique pour développeurs

### Long Terme
- [ ] Intégration de données en temps réel (NASA API)
- [ ] Support voix (Speech-to-Text)
- [ ] Application mobile native

---

## 📄 Licence et Remerciements

**Licence :** MIT  
**Équipe :** Team BioStar sous le leadership de Espérance AYIWAHOUN  
**Challenge :** NASA Space Apps Challenge 2025

**Remerciements :**
- **NASA** pour l'accès aux publications scientifiques
- **Mistral AI** pour les modèles LLM performants
- **Communauté Open Source** pour les outils utilisés

---

## 📚 Références

1. Lewis, P. et al. (2020). "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks". *NeurIPS*.
2. Johnson, J. et al. (2019). "Billion-scale similarity search with GPUs". *IEEE Transactions on Big Data*.
3. NASA GeneLab. "Space Biology Research Database". [genelab.nasa.gov](https://genelab.nasa.gov)

---

**🌟 Résultat :** Une plateforme qui démocratise l'accès aux connaissances scientifiques spatiales grâce à l'IA, tout en maintenant la rigueur académique et la traçabilité des sources.
