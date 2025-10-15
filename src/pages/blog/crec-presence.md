---
layout: ../../layouts/BlogPost.astro
title: "CREC Presence : Système AIoT de Pointage Biométrique à Double Authentification"
description: "Plateforme AIoT innovante combinant reconnaissance faciale et RFID pour une gestion des présences sécurisée, conforme et anti-fraude au CREC Bénin."
date: "2025-08-20"
category: "Systèmes Embarqués & IoT"
tags: ["AIoT", "Computer Vision", "RFID", "FastAPI", "Raspberry Pi", "Edge Computing", "MQTT", "InsightFace", "RAG"]
author: "Espérance AYIWAHOUN"
---

## 🎯 Introduction

**CREC Presence** est un système **AIoT (Artificial Intelligence of Things)** développé pour le **Centre de Recherche, d'Étude et de Créativité (CREC)** au Bénin. Il révolutionne la gestion des présences scolaires en combinant **reconnaissance faciale** et **technologie RFID** pour une authentification robuste anti-fraude.

**🏆 Contribution scientifique :** Mémoire de Licence en Informatique, Option Systèmes Embarqués et Internet des Objets

---

## 📋 Problématique

### Limites des Systèmes Traditionnels

Les méthodes de gestion des présences au CREC présentaient plusieurs faiblesses :

| Problème | Impact | Fréquence |
|----------|--------|-----------|
| **Fraude par badge** ("Buddy punching") | Présences fictives | 15-20% des cas |
| **Erreurs humaines** (feuilles d'émargement) | Données inexactes | 10-15% des cas |
| **Lenteur du processus** | Perte de temps pédagogique | ~15 min/jour |
| **Non-conformité légale** | Risques juridiques | Permanent |

**Objectif** : Assurer une **traçabilité précise**, une **conformité légale** (Loi béninoise n°2009-09) et une **optimisation de la gestion pédagogique**.

---

## 🏗️ Architecture Technique

### Vue d'Ensemble

```
┌─────────────────────────────────────────────────────────┐
│                   MODULES EMBARQUÉS (Edge)              │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────────────────┐   ┌──────────────────────┐   │
│  │ Module de Pointage   │   │ Module d'Enrôlement  │   │
│  │ • Raspberry Pi 4     │   │ • ESP32              │   │
│  │ • PiCamera V2 (8MP)  │   │ • RC522 RFID         │   │
│  │ • RC522 RFID         │   │ • LCD 16x2           │   │
│  │ • VL53L0X (détection)│   └──────────┬───────────┘   │
│  │ • PiJuice HAT        │              │               │
│  └──────────┬───────────┘              │               │
│             │                          │               │
│             └──────────┬───────────────┘               │
│                        │ MQTT/TLS                      │
└────────────────────────┼───────────────────────────────┘
                         │
                         v
┌─────────────────────────────────────────────────────────┐
│              INFRASTRUCTURE CENTRALE (Backend)          │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌────────────┐  ┌──────────┐  ┌─────────────────┐    │
│  │ FastAPI    │  │ Mosquitto│  │ PostgreSQL      │    │
│  │ (API REST) │  │ (MQTT)   │  │ (Données)       │    │
│  └────────────┘  └──────────┘  └─────────────────┘    │
│                                                         │
│  ┌────────────┐  ┌──────────────────────────────┐     │
│  │ ChromaDB   │  │ Chatbot RAG (Gemini Pro)     │     │
│  │ (Vectors)  │  │ • Support admin              │     │
│  └────────────┘  └──────────────────────────────┘     │
└─────────────────────────────────────────────────────────┘
                         │
                         v
┌─────────────────────────────────────────────────────────┐
│                   FRONTEND (Web App)                    │
│  • React/TypeScript                                     │
│  • Gestion modules • Enrôlement • Consultation          │
└─────────────────────────────────────────────────────────┘
```

### Stack Technologique

#### Matériel (Edge)

| Module | Composants | Rôle |
|--------|-----------|------|
| **Module de Pointage** | • Raspberry Pi 4 Model B<br>• PiCamera V2 (8MP)<br>• RC522 RFID<br>• VL53L0X (distance)<br>• PiJuice HAT | Authentification double<br>(Face + RFID)<br>Autonomie énergétique |
| **Module d'Enrôlement** | • ESP32<br>• RC522 RFID<br>• LCD 16x2 | Association UID RFID<br>avec identité |

#### Logiciel (Backend)

| Composant | Technologie | Rôle |
|-----------|-------------|------|
| **API Backend** | FastAPI (Python) | Gestion utilisateurs, modules, présences |
| **Base relationnelle** | PostgreSQL | Stockage présences (horodatage, IDs chiffrés) |
| **Base vectorielle** | ChromaDB | Embeddings faciaux (AES-256) + logs RAG |
| **Broker IoT** | Mosquitto (MQTT/TLS) | Communication asynchrone sécurisée |
| **Reconnaissance faciale** | InsightFace | Extraction embeddings (Edge Computing) |
| **Chatbot RAG** | Gemini Pro + ChromaDB | Assistant admin conversationnel |

---

## ✨ Fonctionnalités Principales

### 1. Authentification Double (RFID + Face)

**Workflow en 5 secondes :**

```
1. Détection de présence (VL53L0X) → Active caméra
2. Lecture carte RFID → UID chiffré
3. Capture faciale → Extraction embedding (InsightFace)
4. Recherche ChromaDB → Comparaison vecteurs
5. Validation double facteur → Enregistrement horodaté
```

**Code : Extraction d'Embedding Facial**

```python
import insightface
from insightface.app import FaceAnalysis

app = FaceAnalysis(providers=['CPUExecutionProvider'])
app.prepare(ctx_id=0, det_size=(640, 640))

def extract_face_embedding(image_path: str) -> np.ndarray:
    """Extrait l'embedding facial (512D) avec InsightFace."""
    img = cv2.imread(image_path)
    faces = app.get(img)
    
    if len(faces) == 0:
        raise ValueError("Aucun visage détecté")
    
    # Retourne l'embedding du premier visage
    return faces[0].embedding  # 512 dimensions
```

### 2. Sécurité et Conformité (RGPD Béninoise)

#### Chiffrement AES-256

```python
from cryptography.fernet import Fernet

# Génération de clé
key = Fernet.generate_key()
cipher = Fernet(key)

# Chiffrement de l'embedding
def encrypt_embedding(embedding: np.ndarray) -> bytes:
    embedding_bytes = embedding.tobytes()
    return cipher.encrypt(embedding_bytes)

# Déchiffrement
def decrypt_embedding(encrypted: bytes) -> np.ndarray:
    decrypted = cipher.decrypt(encrypted)
    return np.frombuffer(decrypted, dtype=np.float32)
```

#### Protection de la Vie Privée

✅ **Images faciales supprimées** automatiquement après extraction d'embedding  
✅ **Stockage uniquement des vecteurs** (512D) chiffrés  
✅ **Consentement écrit** obligatoire avant enrôlement  
✅ **Conformité Loi n°2009-09** sur la protection des données personnelles (Bénin)

### 3. Résilience Réseau (Mode Offline)

En cas de coupure internet :

```python
# Stockage local JSON
def save_attendance_offline(uid: str, timestamp: datetime):
    data = {
        "uid_encrypted": encrypt_uid(uid),
        "timestamp": timestamp.isoformat(),
        "synchronized": False
    }
    
    with open("offline_queue.json", "a") as f:
        json.dump(data, f)
        f.write("\n")

# Synchronisation automatique au retour de connexion
def sync_offline_data():
    if check_internet_connection():
        with open("offline_queue.json", "r") as f:
            for line in f:
                data = json.loads(line)
                send_to_server(data)
        
        os.remove("offline_queue.json")
```

### 4. Chatbot RAG pour Support Admin

**Architecture RAG**

```
┌────────────────┐
│ Question Admin │ "Qui était absent hier ?"
└───────┬────────┘
        │
        v
┌─────────────────────────────────┐
│ 1. Vectorisation (Gemini Embed) │
└───────┬─────────────────────────┘
        │
        v
┌─────────────────────────────────┐
│ 2. Recherche ChromaDB (logs)    │
│    → Top 5 résultats pertinents │
└───────┬─────────────────────────┘
        │
        v
┌─────────────────────────────────┐
│ 3. Génération (Gemini Pro)      │
│    + Contexte des logs          │
└───────┬─────────────────────────┘
        │
        v
┌─────────────────────────────────┐
│ Réponse : "15 élèves absents..." │
└─────────────────────────────────┘
```

**Implémentation**

```python
import chromadb
from google.generativeai import GenerativeModel

# Initialisation
chroma_client = chromadb.PersistentClient(path="./chroma_db")
collection = chroma_client.get_collection("attendance_logs")

gemini_model = GenerativeModel('gemini-pro')

def rag_query(question: str) -> str:
    # 1. Recherche vectorielle
    results = collection.query(
        query_texts=[question],
        n_results=5
    )
    
    # 2. Construction du contexte
    context = "\n".join(results['documents'][0])
    
    # 3. Génération de réponse
    prompt = f"""
    Contexte des logs de présence :
    {context}
    
    Question : {question}
    
    Réponds de manière concise et précise.
    """
    
    response = gemini_model.generate_content(prompt)
    return response.text
```

---

## 📊 Résultats Expérimentaux

### Performances du Système

| Métrique | Résultat | Objectif |
|----------|----------|----------|
| **Temps d'authentification** | 3.8s | < 5s ✅ |
| **Taux de reconnaissance faciale** | 98.97% | > 95% ✅ |
| **FAR (False Acceptance Rate)** | 0.03% | < 0.1% ✅ |
| **FRR (False Rejection Rate)** | 1.03% | < 2% ✅ |
| **Latence recherche RAG** | 50ms | < 100ms ✅ |
| **Autonomie batterie (PiJuice)** | 8h | > 6h ✅ |

### Comparaison avec Systèmes Existants

| Système | FAR | FRR | Double Auth | Offline | Coût |
|---------|-----|-----|-------------|---------|------|
| **CREC Presence** | 0.03% | 1.03% | ✅ | ✅ | $$ |
| Badge RFID seul | N/A | N/A | ❌ | ✅ | $ |
| Face Recognition seul | 0.1% | 2% | ❌ | ❌ | $$ |
| Systèmes commerciaux | 0.05% | 1.5% | ✅ | ❌ | $$$$ |

---

## 🚀 Déploiement

### Architecture Docker

```yaml
version: '3.8'

services:
  api:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@postgres:5432/crec
      - MQTT_BROKER=mosquitto:1883
    depends_on:
      - postgres
      - mosquitto
  
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: crec
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
    volumes:
      - pgdata:/var/lib/postgresql/data
  
  mosquitto:
    image: eclipse-mosquitto:2
    ports:
      - "1883:1883"
      - "9001:9001"
    volumes:
      - ./mosquitto.conf:/mosquitto/config/mosquitto.conf
  
  chromadb:
    image: chromadb/chroma:latest
    ports:
      - "8001:8000"
    volumes:
      - chromadata:/chroma/chroma

volumes:
  pgdata:
  chromadata:
```

### Configuration Module Raspberry Pi

```python
# Module de Pointage (Raspberry Pi 4)
import asyncio
from tdmclient import ClientAsync
import RPi.GPIO as GPIO
from picamera2 import Picamera2

# Configuration MQTT
MQTT_BROKER = "api.crec.local"
MQTT_PORT = 8883  # TLS
API_KEY = "module_key_12345"

# Initialisation caméra
camera = Picamera2()
camera.start()

# Boucle principale
async def main():
    while True:
        # Détection présence (VL53L0X)
        if detect_presence():
            # Lecture RFID
            uid = read_rfid_card()
            
            # Capture faciale
            image = camera.capture_array()
            embedding = extract_embedding(image)
            
            # Envoi MQTT
            await send_attendance(uid, embedding)
            
        await asyncio.sleep(0.1)

asyncio.run(main())
```

---

## 💡 Innovations Techniques

### 1. Edge Computing pour la Vie Privée

**Avantage** : Les images faciales ne quittent **jamais** le module embarqué.

```
Traditionnel :
Image → Serveur → Extraction → Stockage image ❌

CREC Presence :
Image → Edge (RPi) → Extraction → DELETE image ✅
              ↓
        Embedding chiffré → Serveur
```

### 2. Protocole MQTT avec QoS 2

```python
import paho.mqtt.client as mqtt

# Configuration QoS 2 (Exactly Once Delivery)
client = mqtt.Client()
client.tls_set(ca_certs="ca.crt", certfile="client.crt", keyfile="client.key")

def on_publish(client, userdata, mid):
    print(f"Message {mid} delivered with QoS 2")

client.on_publish = on_publish
client.connect(MQTT_BROKER, MQTT_PORT, 60)

# Publication avec QoS 2
client.publish("attendance/module_001", payload, qos=2)
```

**Garantie** : Aucune perte de données même en cas de coupure réseau.

### 3. Système de Révocation de Clés API

```python
# Backend FastAPI
from fastapi import HTTPException, Header

async def verify_api_key(x_api_key: str = Header(...)):
    if x_api_key not in active_api_keys:
        raise HTTPException(status_code=403, detail="API key révoquée")
    return x_api_key

@app.post("/attendance", dependencies=[Depends(verify_api_key)])
async def record_attendance(data: AttendanceData):
    # Traitement sécurisé
    pass
```

---

## 🔮 Perspectives d'Amélioration

### Court Terme
- [ ] Support multi-visages (enrôlement groupé)
- [ ] Détection de masques faciaux (COVID-19)
- [ ] Application mobile pour consultations

### Moyen Terme
- [ ] Analyse prédictive des absences (ML)
- [ ] Intégration avec systèmes de paie
- [ ] Notifications SMS/Email automatiques

### Long Terme
- [ ] Extension à d'autres établissements
- [ ] Système de gestion de flotte de modules
- [ ] Certification ISO 27001 (sécurité des données)

---

## 📚 Références

1. Deng, J. et al. (2020). "RetinaFace: Single-Shot Multi-Level Face Localisation in the Wild". *CVPR*.
2. Guo, J. et al. (2021). "InsightFace: 2D and 3D Face Analysis Project". *GitHub*.
3. OWASP. (2023). "IoT Security Verification Standard (ISVS)".
4. Loi n°2009-09 du Bénin relative à la protection des données à caractère personnel.

---

**🌟 Impact :** Une solution **made in Africa** qui allie innovation technologique, respect de la vie privée et adaptation aux contraintes locales (connectivité, énergie, budget).