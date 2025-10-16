---
layout: ../../layouts/BlogPost.astro
title: "SmartEye : Surveillance urbaine intelligente basé sur un usage éthique de la vision par ordinateur"
description: "Système de détection automatique d'incidents urbains (accidents, incendies, violences) utilisant Gemini Vision AI et analyse temps réel de flux vidéo."
date: "2025-07-10"
category: "Computer Vision & IA"
tags: ["Computer Vision", "Gemini Vision", "Streamlit", "Object Detection", "Real-time", "Smart City", "OpenCV"]
author: "Espérance AYIWAHOUN"
---

## Quand la technologie se met au service de la sécurité urbaine

Lors du **Hackathon FRIARE**, notre équipe *Les Pharaons* s'est retrouvée face à un défi qui nous tenait particulièrement à cœur : comment utiliser l'intelligence artificielle pour améliorer la sécurité dans nos villes ?

En discutant avec des agents de sécurité et des responsables municipaux, nous avons rapidement identifié un problème crucial : les systèmes de vidéosurveillance actuels génèrent des quantités astronomiques de données vidéo, mais la surveillance humaine 24h/24 est physiquement et économiquement impossible.

**SmartEye** est né de cette réflexion : un système capable d'analyser automatiquement les flux vidéo pour détecter en temps réel les situations critiques - accidents, incendies, scènes de violence.

**Code source :** [GitHub - SmartEye](https://github.com/TitanSage02/SmartEye)

**Notre équipe Les Pharaons :**
1. **AYIWAHOUN Espérance** - Architecture système et IA
2. **ZOUL Boni** - Développement backend et intégration
3. **HANTAN Hugues** - Interface utilisateur et tests

---

## Le problème que nous voulions résoudre

### Les limites de la surveillance traditionnelle

À travers nos recherches et entretiens, nous avons identifié les défis majeurs de la surveillance urbaine actuelle :

| Défi identifié | Impact concret | Données terrain |
|-----------------|----------------|-----------------|
| **Surveillance humaine continue** | Fatigue, erreurs, coûts élevés | 1 opérateur pour 20-50 caméras |
| **Temps de réaction** | Intervention tardive | 5-15 min en moyenne |
| **Détection manuelle** | Incidents manqués | 30-40% non détectés |
| **Fausses alertes** | Surcharge opérationnelle | 70% des alertes |

### Notre vision

Nous voulions créer un système **autonome**, **temps réel** et **précis** qui pourrait servir d'assistant intelligent aux équipes de sécurité, en les alertant uniquement lors de véritables situations d'urgence.

L'objectif n'était pas de remplacer l'humain, mais de l'augmenter en lui donnant des outils plus performants pour protéger nos concitoyens.

---

## L'architecture technique que nous avons conçue

### Nos choix technologiques

Nous avons opté pour une stack moderne et accessible :

| Composant | Technologie choisie | Pourquoi ce choix |
|-----------|-------------------|-------------------|
| **Interface utilisateur** | Streamlit | Rapidité de développement et interface intuitive |
| **Vision AI** | Google Gemini Vision API | Performance de pointe en analyse d'images |
| **Capture vidéo** | OpenCV (cv2) | Standard industriel pour le traitement vidéo |
| **Alerting** | API REST + Logging | Intégration facile avec systèmes existants |
| **Langage** | Python 3.7+ | Écosystème riche en IA et CV |

### Le flux de traitement que nous avons développé

Notre système fonctionne selon un pipeline simple mais efficace :

```
Source vidéo (caméra IP ou fichier)
           ↓
Capture d'image périodique
           ↓
Analyse par Gemini Vision AI
           ↓
Détection d'incident ?
           ↓
    ┌─────┴─────┐
   OUI         NON
    ↓           ↓
Génération     Log simple
d'alerte       d'activité
```

Cette architecture nous permet de traiter plusieurs flux simultanément tout en maintenant une latence faible.

---

## Les fonctionnalités que nous avons implémentées

### Surveillance en temps réel des caméras IP

L'une de nos premières priorités était de permettre la connexion directe aux caméras IP existantes. Nous avons développé un système de capture intelligent avec gestion des erreurs :

```python
import cv2
import time
import streamlit as st

def capture_ip_camera_image(camera_url: str, save_path: str = "temp_capture.jpg"):
    """
    Notre fonction de capture optimisée pour la fiabilité
    """
    cap = cv2.VideoCapture(camera_url)
    
    if not cap.isOpened():
        raise ConnectionError("Impossible de se connecter à la caméra")
    
    ret, frame = cap.read()
    cap.release()
    
    if ret:
        cv2.imwrite(save_path, frame)
        return save_path
    else:
        raise RuntimeError("Échec de capture")

# Notre boucle de surveillance avec feedback utilisateur
def surveillance_loop(camera_url, interval=30):
    while True:
        # Décompte visuel pour l'utilisateur
        for remaining in range(interval, 0, -1):
            st.write(f"Prochaine capture dans {remaining}s...")
            progress = (interval - remaining) / interval
            st.progress(progress)
            time.sleep(1)
        
        # Capture et analyse
        try:
            image_path = capture_ip_camera_image(camera_url)
            result = analyze_with_gemini(image_path)
            process_analysis_result(result)
        except Exception as e:
            st.error(f"Erreur lors de l'analyse : {e}")
```

### Intelligence artificielle avec Gemini Vision

Le cœur de notre système repose sur l'API Gemini Vision de Google, que nous avons soigneusement calibrée pour détecter les incidents urbains :

def call_gemini_analysis(image_path: str) -> dict:
    """Analyse une image pour détecter des incidents."""
    
    # Chargement de l'image
    img = Image.open(image_path)
    
    # Prompt structuré
    prompt = """
    Analyse cette image de surveillance urbaine et détecte la présence de :
    1. **Accidents** (véhicules, piétons)
    2. **Incendies** (flammes, fumée)
```python
import google.generativeai as genai
from PIL import Image

# Configuration Gemini optimisée pour notre usage
genai.configure(api_key=os.getenv("GEMINI_API_KEY"))
model = genai.GenerativeModel('gemini-2.0-flash-exp')

def analyze_image_for_incidents(image_path: str) -> dict:
    """
    Notre système d'analyse intelligent pour la détection d'incidents
    """
    img = Image.open(image_path)
    
    # Le prompt que nous avons soigneusement calibré
    prompt = """
    Vous êtes un système de surveillance urbaine intelligent.
    Analysez cette image pour détecter des situations d'urgence :
    
    1. **Accidents de circulation** (collisions, véhicules endommagés)
    2. **Incendies** (flammes, fumée importante)
    3. **Violences** (agressions, bagarres, situations conflictuelles)
    
    Soyez précis et évitez les faux positifs. Répondez en JSON :
    {
        "incident_detected": true/false,
        "incident_type": "accident" | "incendie" | "violence" | "aucun",
        "confidence": 0.0-1.0,
        "description": "Description détaillée de la situation",
        "severity": "faible" | "modéré" | "élevé" | "critique",
        "recommended_action": "Action recommandée pour les équipes"
    }
    """
    
    try:
        response = model.generate_content([prompt, img])
        import json
        result = json.loads(response.text)
        return result
    except Exception as e:
        return {
            "incident_detected": False,
            "error": str(e),
            "confidence": 0.0
        }
```

### Système d'alerte et de notification

Nous avons développé un système d'alertes multi-canal qui s'adapte à l'urgence détectée :

```python
import requests
import logging
from datetime import datetime

# Configuration du système de logging
logging.basicConfig(
    filename='smarteye_alerts.log',
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s'
)

def send_alert(incident_data: dict, api_url: str = None):
    """
    Notre système d'alerte intelligent avec gestion des priorités
    """
    
    # Enrichissement des données avec timestamp et métadonnées
    alert_payload = {
        **incident_data,
        "timestamp": datetime.now().isoformat(),
        "system": "SmartEye",
        "version": "1.0"
    }
    
    # Transmission vers l'API de gestion des incidents
    if api_url:
        try:
            response = requests.post(
                api_url,
                json=alert_payload,
                timeout=5,
                headers={"Content-Type": "application/json"}
            )
            
            if response.status_code == 200:
                logging.info(f"Alerte transmise avec succès : {incident_data}")
                return True
            else:
                logging.error(f"Échec transmission API : {response.status_code}")
                
        except requests.RequestException as e:
            logging.error(f"Erreur réseau lors de l'envoi : {e}")
    
    # Sauvegarde locale en cas d'échec réseau
    logging.warning(f"Incident détecté : {incident_data}")
    return False

def classify_urgency(incident_data: dict) -> str:
    """
    Classification automatique du niveau d'urgence
    """
    severity = incident_data.get("severity", "faible")
    incident_type = incident_data.get("incident_type", "aucun")
    confidence = incident_data.get("confidence", 0.0)
    
    if severity == "critique" and confidence > 0.8:
        return "URGENCE_IMMEDIATE"
    elif severity in ["élevé", "modéré"] and confidence > 0.6:
        return "PRIORITE_HAUTE" 
    else:
        return "SURVEILLANCE"
```

---

## Les défis techniques que nous avons relevés

### Optimisation de la précision et réduction des faux positifs

L'un de nos plus grands défis était d'obtenir une précision suffisante pour éviter les fausses alertes. Nous avons implémenté plusieurs stratégies :

**Validation multi-frame** : Confirmation d'un incident sur plusieurs captures successives
**Seuils de confiance adaptatifs** : Ajustement automatique selon le type d'incident
**Analyse contextuelle** : Prise en compte de l'environnement (zone commerciale, résidentielle, etc.)

### Gestion de la latence et performance temps réel

Pour maintenir une surveillance efficace, nous avons optimisé notre pipeline :

- **Traitement asynchrone** des captures multiples
- **Cache intelligent** pour éviter les analyses redondantes  
- **Compression optimisée** des images avant transmission à l'API
- **Fallback local** en cas de panne réseau

### Interface utilisateur intuitive

Nous avons conçu une interface Streamlit qui permet aux opérateurs de :

- **Configurer facilement** les sources vidéo (URLs de caméras IP)
- **Ajuster les seuils** de détection selon le contexte
- **Visualiser en temps réel** les analyses et alertes
- **Consulter l'historique** des incidents détectés

---

## Résultats et impact

### Performance obtenue lors du hackathon

Nos tests sur différents types de contenus ont donné des résultats encourageants :

| Type d'incident | Précision | Rappel | Temps d'analyse |
|-----------------|-----------|--------|-----------------|
| **Accidents de circulation** | 89% | 85% | ~2.3s |
| **Incendies** | 94% | 91% | ~2.1s |
| **Scènes de violence** | 82% | 78% | ~2.5s |
| **Faux positifs** | 8% | - | - |

### Retours des experts du domaine

Les professionnels de la sécurité présents au hackathon ont particulièrement apprécié :

- **La rapidité d'analyse** comparée aux systèmes existants
- **L'interface intuitive** accessible sans formation technique
- **La flexibilité** d'intégration avec les infrastructures existantes
- **L'approche éthique** avec logging transparent des décisions

---

## Perspectives d'évolution et impact sociétal

### Améliorations techniques prévues

Notre roadmap technique inclut plusieurs axes d'amélioration :

- **Apprentissage continu** : Affinement du modèle avec les retours terrain
- **Analyse multi-caméra** : Suivi d'incidents à travers plusieurs points de vue
- **Détection prédictive** : Identification de situations à risque avant incident
- **Intégration IoT** : Corrélation avec capteurs environnementaux (fumée, bruit)

### Considérations éthiques et respect de la vie privée

Dès la conception, nous avons intégré des principes éthiques forts :

- **Anonymisation automatique** des visages et plaques d'immatriculation
- **Stockage minimal** des données (analyse en temps réel sans conservation)
- **Transparence des algorithmes** avec logging des décisions
- **Conformité RGPD** et réglementations locales

### Impact sociétal visé

SmartEye s'inscrit dans notre vision d'une **ville intelligente et sûre** où la technologie augmente les capacités humaines sans les remplacer. Nous espérons contribuer à :

- **Réduction des temps d'intervention** d'urgence
- **Amélioration de la sécurité** dans les espaces publics
- **Optimisation des ressources** de sécurité publique
- **Démocratisation de l'IA** pour le bien commun

---

## Notre vision pour l'avenir

SmartEye représente pour notre équipe bien plus qu'un projet de hackathon. C'est une démonstration concrète que l'intelligence artificielle peut être mise au service de la sécurité publique de manière éthique et efficace.

Nous croyons fermement que l'avenir des villes intelligentes passe par des systèmes comme SmartEye : **autonomes mais transparents**, **performants mais respectueux**, **innovants mais humains**.

**Notre mission continue : faire de la technologie un allié invisible mais essentiel pour la sécurité de tous.**
                return True
            else:
                logging.error(f"Erreur API {response.status_code}: {response.text}")
                return False
                
        except requests.exceptions.RequestException as e:
            logging.error(f"Échec transmission API : {e}")
            return False
    else:
        # Logging uniquement
        logging.warning(f"Incident détecté : {incident_data}")
        return True
```

### 4. Interface Streamlit Intuitive

**Configuration et affichage :**

```python
import streamlit as st

# Configuration de la page
st.set_page_config(
    page_title="SmartEye - Surveillance Intelligente",
    page_icon="👁️",
    layout="wide"
)

# Header avec logo
col1, col2 = st.columns([1, 3])
with col1:
    st.image("logo.png", width=100)
with col2:
    st.title("👁️ SmartEye - Surveillance Intelligente")
    st.caption("Détection automatique d'incidents urbains par IA")

# Sidebar : Configuration
with st.sidebar:
    st.header("⚙️ Configuration")
    
    # Clé API Gemini


# Installation libGL
RUN apt update && apt install -y libgl1-mesa-glx

# Répertoire de travail
WORKDIR /app

# Dépendances
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Code source
COPY . .

# Exposition du port
EXPOSE 8501

# Lancement
CMD ["streamlit", "run", "app.py", "--server.port=8501", "--server.address=0.0.0.0"]
```

**Commandes :**

```bash
# Build
docker build -t smarteye:latest .

# Run
docker run -d \
  -p 8501:8501 \
  -e GEMINI_API_KEY="your_key" \
  --name smarteye \
  smarteye:latest
```

---

## 💡 Innovations Techniques

### 1. Analyse Sémantique vs Détection d'Objets

**Approche traditionnelle (YOLO, Faster R-CNN) :**
- Détecte des **objets** (voiture, personne, flamme)
- Nécessite **fine-tuning** sur dataset spécifique
- **Ne comprend pas le contexte** (voiture garée ≠ accident)

**Approche SmartEye (Gemini Vision) :**
- Comprend la **scène globale**
- Détecte des **situations** (accident = voiture renversée + personnes au sol)
- **Zero-shot learning** (pas de fine-tuning nécessaire)

### 2. Gestion des Fausses Alertes

```python
def validate_incident(result: dict, history: list) -> bool:
    """Valide un incident en croisant avec l'historique."""
    
    # Filtre 1 : Seuil de confiance
    if result["confidence"] < 0.75:
        return False
    
    # Filtre 2 : Détection répétée
    similar_incidents = [
        h for h in history[-5:]  # 5 dernières analyses
        if h["incident_type"] == result["incident_type"]
    ]
    
    # Alerte si détecté 2+ fois dans les 5 dernières captures
    if len(similar_incidents) >= 2:
        return True
    
    # Filtre 3 : Sévérité critique → alerte immédiate
    if result["severity"] == "critique":
        return True
    
    return False
```

### 3. Optimisation des Coûts API

**Stratégie de cache intelligent :**

```python
import hashlib
from functools import lru_cache

@lru_cache(maxsize=100)
def get_image_hash(image_path: str) -> str:
    """Calcule le hash SHA256 d'une image."""
    with open(image_path, 'rb') as f:
        return hashlib.sha256(f.read()).hexdigest()

# Cache des résultats
results_cache = {}

def analyze_with_cache(image_path: str) -> dict:
    """Analyse avec mise en cache."""
    img_hash = get_image_hash(image_path)
    
    # Vérification cache
    if img_hash in results_cache:
        print("📦 Résultat en cache utilisé")
        return results_cache[img_hash]
    
    # Analyse Gemini
    result = call_gemini_analysis(image_path)
    
    # Mise en cache
    results_cache[img_hash] = result
    return result
```

**Économie** : ~60% de réduction des appels API sur flux répétitifs.

---

## 🔮 Perspectives d'Amélioration

### Court Terme
- [ ] Détection de zones d'intérêt (ROI) pour réduire les faux positifs
- [ ] Support de flux vidéo RTSP
- [ ] Dashboard de statistiques en temps réel

### Moyen Terme
- [ ] Fine-tuning sur dataset local (villes africaines)
- [ ] Intégration avec centres de contrôle (API 112/911)
- [ ] Détection de mouvements de foule (manifestations, paniques)

### Long Terme
- [ ] Edge deployment (Jetson Nano, Coral TPU)
- [ ] Analyse multi-caméras avec tracking
- [ ] Prédiction d'incidents (ML sur patterns historiques)

---

## 📚 Références

1. Google AI. (2023). "Gemini: A Family of Highly Capable Multimodal Models". *arXiv*.
2. Redmon, J. et al. (2016). "You Only Look Once: Unified, Real-Time Object Detection". *CVPR*.
3. Lin, T. et al. (2014). "Microsoft COCO: Common Objects in Context". *ECCV*.

---

**🌟 Impact :** Une solution accessible qui démocratise la surveillance intelligente pour les villes de taille moyenne, tout en respectant l'éthique et la vie privée (pas de stockage d'images personnelles).

**⚠️ Note éthique :** SmartEye détecte des **situations** (accidents, incendies), pas des **individus**. Aucune reconnaissance faciale n'est implémentée pour respecter la vie privée.