---
layout: ../../layouts/BlogPost.astro
title: "VoxThymio : Contrôle Vocal Intelligent du Robot Thymio avec IA"
description: "Système avancé de reconnaissance vocale et compréhension sémantique pour piloter des robots Thymio par la voix, utilisant Whisper, BERT et ChromaDB."
date: "2025-09-15"
category: "Robotique & IA"
tags: ["Robotique", "NLP", "Speech Recognition", "Whisper", "BERT", "ChromaDB", "Thymio", "Embeddings"]
author: "Espérance AYIWAHOUN"
---

## 🤖 Introduction

**VoxThymio** est un système avancé de contrôle vocal pour robots **Thymio** développé pour **AI4Innov**. Il utilise l'intelligence artificielle pour comprendre et exécuter des commandes en langage naturel, avec un système d'apprentissage dynamique qui enrichit automatiquement sa base de connaissances.

**📦 Code source :** [GitHub - VoxThymio](https://github.com/TitanSage02/Vox-Thymio)

---

## 📋 Problématique

### Limitations des Systèmes Existants

Les robots éducatifs comme Thymio sont traditionnellement programmés via :
- **Interfaces graphiques** (Aseba, Blockly) → Non intuitif pour débutants
- **Code Aseba** → Barrière technique pour enseignants
- **Télécommande infrarouge** → Interactions limitées

**Besoin identifié :** Un système de contrôle **vocal naturel** qui :
1. Comprend les **variations linguistiques** ("avance", "va tout droit", "va devant")
2. **Apprend dynamiquement** de nouvelles commandes
3. Fonctionne **hors ligne** (sans connexion internet)
4. Offre une **latence faible** (< 2 secondes)

---

## 🏗️ Architecture Technique

### Stack Technologique

| Composant | Technologie | Rôle |
|-----------|-------------|------|
| **Reconnaissance vocale** | OpenAI Whisper + SpeechRecognition | Conversion parole → texte |
| **Embeddings sémantiques** | SentenceTransformer (multilingual MiniLM) | Vectorisation des commandes |
| **Base vectorielle** | ChromaDB | Stockage et recherche de similarité |
| **Contrôle robot** | tdmclient (Protocol Aseba) | Communication Thymio |
| **Langage** | Python 3.8+ | Backend principal |

### Diagramme d'Architecture

```
┌──────────────────┐
│  Microphone      │
└────────┬─────────┘
         │
         v
┌──────────────────────────────┐
│  Speech Recognition          │
│  • Whisper (offline)         │
│  • Google API (fallback)     │
└────────┬─────────────────────┘
         │
         v
┌──────────────────────────────┐
│  Normalisation du texte      │
│  • Lowercase                 │
│  • Suppression accents       │
└────────┬─────────────────────┘
         │
         v
┌──────────────────────────────┐
│  Génération d'Embeddings     │
│  (SentenceTransformer)       │
└────────┬─────────────────────┘
         │
         v
┌──────────────────────────────┐
│  Recherche ChromaDB          │
│  • Similarité cosinus        │
│  • Top-1 résultat            │
└────────┬─────────────────────┘
         │
         v
    ┌───┴────┐
    │ Score  │
    │ > 0.5? │
    └───┬────┘
        │
    ┌───┴─────────────┐
    │                 │
   YES               NO
    │                 │
    v                 v
┌─────────┐    ┌─────────────┐
│ Exécute │    │ Score > 0.85│
│ Action  │    │ → Apprend   │
└─────────┘    └─────────────┘
```

---

## ✨ Fonctionnalités Principales

### 1. Reconnaissance Vocale Multimodale

**Whisper (Prioritaire)**
```python
import whisper

model = whisper.load_model("small")

def transcribe_whisper(audio_file):
    result = model.transcribe(audio_file, language="fr")
    return result["text"]
```

**Fallback : Google Speech Recognition**
```python
import speech_recognition as sr

recognizer = sr.Recognizer()

def transcribe_google(audio_data):
    return recognizer.recognize_google(audio_data, language="fr-FR")
```

### 2. Compréhension Sémantique

Utilisation d'embeddings multilingues pour comprendre les **variations de commandes** :

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('paraphrase-multilingual-MiniLM-L12-v2')

# Exemples de commandes similaires
commandes = [
    "avance",
    "va tout droit",
    "va devant",
    "avance un peu"
]

# Génération d'embeddings (384 dimensions)
embeddings = model.encode(commandes)
```

**Résultat :** Toutes ces commandes sont reconnues comme similaires (similarité > 0.85).

### 3. Base Vectorielle ChromaDB

```python
import chromadb

# Initialisation
client = chromadb.PersistentClient(path="./vector_db")
collection = client.get_or_create_collection("thymio_commands")

# Ajout de commandes
collection.add(
    documents=["avance", "tourne à droite"],
    embeddings=[embed_avance, embed_droite],
    metadatas=[
        {"code": "motor.left.target = 200\nmotor.right.target = 200"},
        {"code": "motor.left.target = 200\nmotor.right.target = -200"}
    ],
    ids=["cmd_001", "cmd_002"]
)

# Recherche par similarité
results = collection.query(
    query_embeddings=[embed_user_input],
    n_results=1
)
```

### 4. Apprentissage Dynamique

Le système apprend automatiquement si :
- Similarité > **0.85** (forte similarité)
- La commande **n'existe pas encore** dans la base

```python
def process_command(user_input: str):
    # Recherche
    results = collection.query(query_texts=[user_input], n_results=1)
    score = results['distances'][0][0]
    
    if score > 0.5:
        # Exécution
        execute_command(results['metadatas'][0]['code'])
        
        if score > 0.85 and not is_exact_match(user_input):
            # Apprentissage
            add_command_variant(user_input, results['metadatas'][0])
    else:
        print("Commande non reconnue")
```

---

## 🔧 Commandes Disponibles

### Commandes de Base

| Catégorie | Commandes | Code Aseba |
|-----------|-----------|------------|
| **Mouvement** | "avance", "va tout droit", "va devant" | `motor.left.target = 200`<br>`motor.right.target = 200` |
| **Rotation** | "tourne à droite", "va à droite" | `motor.left.target = 200`<br>`motor.right.target = -200` |
| **Rotation** | "tourne à gauche", "va à gauche" | `motor.left.target = -200`<br>`motor.right.target = 200` |
| **Arrêt** | "arrête", "stop", "arrête-toi" | `motor.left.target = 0`<br>`motor.right.target = 0` |
| **Demi-tour** | "fais demi-tour", "retourne-toi" | Rotation 180° |

### Ajout de Commandes Personnalisées

```python
from src.smart_voice_controller import SmartVoiceController

# Ajout d'une nouvelle commande
controller.add_new_command(
    command_id="dance",
    description="faire une danse",
    code="""
    motor.left.target = 200
    motor.right.target = -200
    timer.period[0] = 500
    """
)
```

---

## 📊 Résultats et Performance

### Métriques Techniques

| Métrique | Valeur | Détails |
|----------|--------|---------|
| **Latence de reconnaissance** | ~1.5s | Whisper (modèle small) |
| **Précision de transcription** | 94% | Dataset de 100 commandes FR |
| **Taux de reconnaissance** | 91% | Commandes avec variations |
| **Temps de recherche** | <50ms | ChromaDB sur 500 commandes |
| **Taux d'apprentissage** | 87% | Nouvelles variantes apprises |

### Comparaison des Modèles

| Modèle | WER | Latence | Offline |
|--------|-----|---------|---------|
| **Whisper Small** | 6% | 1.5s | ✅ |
| Google Cloud STT | 4% | 0.8s | ❌ |
| SpeechRecognition | 12% | 1.2s | ❌ |

**Choix** : Whisper pour l'équilibre performance/offline.

---

## 🚀 Installation et Utilisation

### Prérequis

- Python 3.8+
- Robot Thymio (USB/Bluetooth)
- Microphone fonctionnel
- (Optionnel) GPU CUDA pour Whisper accéléré

### Installation

```bash
# 1. Cloner le repository
git clone https://github.com/TitanSage02/Vox-Thymio.git
cd VoxThymio

# 2. Installer les dépendances
pip install -r requirements.txt

# 3. (Windows) Installer PyAudio
pip install pipwin
pipwin install pyaudio
```

### Utilisation

```python
from src.smart_voice_controller import SmartVoiceController
from src.controller.thymio_controller import ThymioController
import asyncio

async def main():
    # Connexion Thymio
    thymio = ThymioController()
    await thymio.connect()
    
    # Contrôleur vocal
    voice_controller = SmartVoiceController(thymio)
    
    # Reconnaissance vocale continue
    await voice_controller.voice_recognition()

asyncio.run(main())
```

### Configuration Avancée

```python
# Ajustement des seuils
voice_controller.update_thresholds(
    execution_threshold=0.6,   # Seuil d'exécution
    learning_threshold=0.85    # Seuil d'apprentissage
)

# Choix du modèle Whisper
voice_controller.speech_recognizer.model_size = "medium"  # tiny/small/medium/large
```

---

## 💡 Innovations Techniques

### 1. Recherche Vectorielle Optimisée

Utilisation de la **distance cosinus** pour comparer les commandes :

```python
def cosine_similarity(vec1, vec2):
    return np.dot(vec1, vec2) / (np.linalg.norm(vec1) * np.linalg.norm(vec2))
```

**Avantage :** Insensible à la longueur du texte.

### 2. Normalisation Intelligente

```python
import unicodedata

def normalize_text(text: str) -> str:
    # Minuscules
    text = text.lower()
    
    # Suppression des accents
    text = unicodedata.normalize('NFKD', text)
    text = text.encode('ascii', 'ignore').decode('utf-8')
    
    # Suppression ponctuation
    text = re.sub(r'[^\w\s]', '', text)
    
    return text.strip()
```

### 3. Système Anti-Conflit

Évite les doublons dans la base vectorielle :

```python
def is_duplicate(new_command: str, threshold=0.95):
    results = collection.query(query_texts=[new_command], n_results=1)
    return results['distances'][0][0] > threshold
```

---

## 🐛 Résolution de Problèmes

### Problème 1 : Aucun robot détecté

```bash
❌ Erreur : Aucun robot Thymio détecté
```

**Solution :**
1. Vérifier que le robot est allumé
2. Connecter via USB ou Bluetooth
3. Vérifier les drivers Aseba

### Problème 2 : Latence élevée

```bash
⚠️ Latence de reconnaissance > 3s
```

**Solutions :**
- Utiliser un modèle Whisper plus petit (`tiny`)
- Activer l'accélération GPU
- Réduire la qualité audio

### Problème 3 : Faible précision

```bash
⚠️ Commande mal reconnue
```

**Solutions :**
- Ajuster le seuil de bruit (`energy_threshold`)
- Parler plus clairement
- Ajouter des exemples de commandes

---

## 🔮 Perspectives d'Amélioration

### Court Terme
- [ ] Interface graphique (GUI) pour la gestion des commandes
- [ ] Support multi-robots (plusieurs Thymio simultanément)
- [ ] Retour vocal (Text-to-Speech) pour confirmer les actions

### Moyen Terme
- [ ] Fine-tuning du modèle Whisper sur corpus robotique
- [ ] Commandes complexes ("avance puis tourne à droite")
- [ ] Détection d'intentions (NLU avec BERT)

### Long Terme
- [ ] Apprentissage par renforcement (optimisation trajectoires)
- [ ] Intégration vision par ordinateur (commandes gestuelles)
- [ ] API REST pour contrôle distant

---

## 📚 Références

1. Radford, A. et al. (2022). "Robust Speech Recognition via Large-Scale Weak Supervision". *OpenAI*.
2. Reimers, N., & Gurevych, I. (2019). "Sentence-BERT: Sentence Embeddings using Siamese BERT-Networks". *EMNLP*.
3. Thymio Documentation. [thymio.org](https://www.thymio.org)
4. ChromaDB Documentation. [docs.trychroma.com](https://docs.trychroma.com)

---

**🌟 Résultat :** Un système de contrôle vocal intelligent qui rend la robotique éducative accessible à tous, tout en intégrant des technologies d'IA de pointe pour la compréhension du langage naturel.
