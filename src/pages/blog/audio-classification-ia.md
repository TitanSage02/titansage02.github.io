---
layout: ../../layouts/BlogPost.astro
title: "Classification Audio IA : Détection de sons en temps réel avec TensorFlow.js"
description: "Application web de classification audio intelligente utilisant TensorFlow.js et Teachable Machine pour identifier des sons en temps réel directement dans le navigateur."
date: "2025-11-02"
category: "Intelligence Artificielle & Web"
tags: ["TensorFlow.js", "Machine Learning", "Audio Processing", "Teachable Machine", "Web AI", "Real-time", "JavaScript"]
author: "Espérance AYIWAHOUN"
---

## Quand l'IA s'invite dans votre navigateur

Au **Centre de Recherche et d'Expertise en Computation (CREC)**, nous explorons constamment les frontières de l'intelligence artificielle accessible. Notre dernière expérimentation nous a conduit vers un territoire fascinant : **l'analyse audio en temps réel, directement dans le navigateur web**.

L'idée peut sembler simple, mais elle représente un défi technique majeur : comment faire tourner un modèle d'intelligence artificielle performant sans serveur backend, sans installation complexe, uniquement avec les technologies web natives ?

**Classification Audio IA** est notre réponse à cette question - un système capable d'écouter, comprendre et classifier des sons instantanément, le tout fonctionnant à 100% côté client.

**Démo en ligne :** [https://titansage02.github.io/tm-my-audio-model/](https://titansage02.github.io/tm-my-audio-model/)  
**Code source :** [GitHub - tm-my-audio-model](https://github.com/TitanSage02/tm-my-audio-model)

---

## Le défi : L'IA audio accessible à tous

### Les limites des solutions traditionnelles

Les systèmes de reconnaissance audio nécessitent généralement :

| Contrainte traditionnelle | Impact | Limite |
|---------------------------|--------|--------|
| **Serveur backend** | Infrastructure coûteuse | Hosting, maintenance |
| **Latence réseau** | Délai de réponse | 100-500ms minimum |
| **Installation logicielle** | Barrière à l'adoption | Processus complexe |
| **Données sensibles** | Problèmes de confidentialité | Audio envoyé au serveur |
| **Dépendance internet** | Fonctionnement limité | Connexion requise |

### Notre approche : Edge AI dans le navigateur

Nous avons opté pour une architecture **100% client-side** qui offre :

- ⚡ **Temps réel** : Détection instantanée sans latence réseau
- 🔒 **Confidentialité** : Les données audio ne quittent jamais votre appareil
- 🌐 **Universalité** : Fonctionne sur n'importe quel navigateur moderne
- 💰 **Coût zéro** : Aucune infrastructure serveur nécessaire
- 🚀 **Déploiement simple** : Hébergement statique via GitHub Pages

---

## Architecture technique

### Stack technologique

```
┌─────────────────────────────────────────────────────────┐
│                   NAVIGATEUR WEB                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────────────────┐   ┌──────────────────────┐   │
│  │   Capture Audio      │   │  TensorFlow.js       │   │
│  │   • Web Audio API    │──▶│  • Model Runtime     │   │
│  │   • getUserMedia()   │   │  • Inference Engine  │   │
│  │   • FFT Transform    │   │  • Audio Features    │   │
│  └──────────────────────┘   └──────────────────────┘   │
│            │                           │                │
│            ▼                           ▼                │
│  ┌──────────────────────┐   ┌──────────────────────┐   │
│  │  Prétraitement       │   │  Modèle IA           │   │
│  │  • Spectrogramme     │──▶│  • Teachable Machine │   │
│  │  • Normalisation     │   │  • 3 Classes         │   │
│  │  • Feature Extract   │   │  • Softmax Output    │   │
│  └──────────────────────┘   └──────────────────────┘   │
│                                       │                │
│                                       ▼                │
│  ┌──────────────────────────────────────────────────┐  │
│  │           Interface Utilisateur                  │  │
│  │  • Bootstrap 5 • Barres de progression animées  │  │
│  │  • Font Awesome • Design responsive             │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

### Composants clés

#### 1. **TensorFlow.js** - Le cerveau de l'application

TensorFlow.js nous permet d'exécuter des modèles de deep learning directement dans le navigateur grâce à WebGL.

```javascript
// Chargement du modèle entraîné
const URL = "./model/";
const modelURL = URL + "model.json";
const metadataURL = URL + "metadata.json";

model = await tmAudio.create(modelURL, metadataURL);
```

**Avantages** :
- Inférence GPU-accélérée via WebGL
- Conversion automatique depuis Teachable Machine
- API simple et intuitive

#### 2. **Teachable Machine** - L'entraînement simplifié

Google Teachable Machine nous a permis d'entraîner notre modèle sans écrire une seule ligne de code Python :

1. **Collecte de données** : Enregistrement d'échantillons audio pour chaque classe
2. **Entraînement** : Configuration automatique du réseau de neurones
3. **Export** : Téléchargement du modèle au format TensorFlow.js

**Classes détectées** :
- 🔊 **Bruit de fond** - Ambiance générale, silence relatif
- 💧 **Son de l'eau** - Écoulement, ruissellement
- 👏 **Clap de mains** - Applaudissements, tapements

#### 3. **Web Audio API** - La capture audio

```javascript
// Accès au microphone
const stream = await navigator.mediaDevices.getUserMedia({ audio: true });

// Création du contexte audio
const audioContext = new AudioContext();
const source = audioContext.createMediaStreamSource(stream);

// Analyse spectrale
const analyser = audioContext.createAnalyser();
analyser.fftSize = 2048;
```

L'API Web Audio nous donne accès à :
- **FFT (Fast Fourier Transform)** : Conversion temps → fréquence
- **Spectrogramme** : Représentation visuelle des fréquences
- **Feature extraction** : Caractéristiques audio pour l'IA

---

## Pipeline de traitement

### Étape 1 : Capture et prétraitement

```
Microphone → Signal audio brut (44.1kHz)
              ↓
         Fenêtrage (chunks de 23ms)
              ↓
         FFT Transform
              ↓
         Spectrogramme mel-scale
              ↓
         Normalisation [-1, 1]
```

### Étape 2 : Inférence du modèle

Le modèle est un réseau de neurones convolutionnel (CNN) optimisé pour l'audio :

```
Input Layer (spectrogramme)
     ↓
Conv1D Layers (extraction de features)
     ↓
MaxPooling (réduction dimensionnelle)
     ↓
Dense Layers (classification)
     ↓
Softmax (probabilités par classe)
```

### Étape 3 : Post-traitement et affichage

```javascript
// Prédiction temps réel
async function predict() {
    const predictions = await model.classify(recognizer.spectrogram);
    
    predictions.forEach(prediction => {
        const className = prediction.className;
        const probability = (prediction.probability * 100).toFixed(2);
        
        // Mise à jour UI
        updateProgressBar(className, probability);
    });
}
```

---

## Implémentation technique

### Structure du projet

```
tm-my-audio-model/
│
├── index.html              # Application principale
│
├── model/                  # Modèle Teachable Machine
│   ├── model.json         # Architecture (layers, config)
│   ├── metadata.json      # Labels et paramètres
│   └── weights.bin        # Poids entraînés (1.2 MB)
│
└── README.md              # Documentation
```

### Code principal

```javascript
// Initialisation du système
let model, recognizer, audioContext;

async function init() {
    // 1. Charger le modèle
    const modelURL = "./model/model.json";
    const metadataURL = "./model/metadata.json";
    
    model = await tmAudio.create(modelURL, metadataURL);
    
    // 2. Initialiser le recognizer
    recognizer = model.createAudioRecognizer();
    
    // 3. Démarrer la classification
    await recognizer.listen(result => {
        const predictions = result.scores;
        displayPredictions(predictions);
    }, {
        probabilityThreshold: 0.75,
        invokeCallbackOnNoiseAndUnknown: true,
        overlapFactor: 0.5
    });
}

// Affichage des résultats
function displayPredictions(predictions) {
    const labels = ["Bruit de fond", "Son de l'eau", "Clap de mains"];
    
    predictions.forEach((prob, index) => {
        const percentage = (prob * 100).toFixed(1);
        const progressBar = document.getElementById(`bar-${index}`);
        
        progressBar.style.width = `${percentage}%`;
        progressBar.textContent = `${percentage}%`;
        
        // Code couleur selon la confiance
        if (prob > 0.8) {
            progressBar.className = 'progress-bar bg-success';
        } else if (prob > 0.5) {
            progressBar.className = 'progress-bar bg-warning';
        } else {
            progressBar.className = 'progress-bar bg-secondary';
        }
    });
}
```

### Interface utilisateur

L'interface Bootstrap 5 offre une expérience utilisateur fluide :

```html
<!-- Contrôles -->
<div class="text-center mb-4">
    <button id="startBtn" class="btn btn-success btn-lg">
        <i class="fas fa-play"></i> Démarrer la détection
    </button>
    <button id="stopBtn" class="btn btn-danger btn-lg" disabled>
        <i class="fas fa-stop"></i> Arrêter
    </button>
</div>

<!-- Résultats en temps réel -->
<div class="predictions-container">
    <div class="prediction-item">
        <label>🔊 Bruit de fond</label>
        <div class="progress">
            <div id="bar-0" class="progress-bar" role="progressbar"></div>
        </div>
    </div>
    <!-- ... autres classes ... -->
</div>
```

---

## Performance et optimisations

### Métriques de performance

| Métrique | Valeur | Détails |
|----------|--------|---------|
| **Temps de chargement** | < 3s | Modèle + dépendances |
| **Latence d'inférence** | ~50ms | Par frame audio (23ms) |
| **Taille du modèle** | 1.2 MB | Weights + architecture |
| **Précision** | ~85-90% | Sur les 3 classes |
| **FPS** | ~20-30 | Prédictions/seconde |

### Optimisations implémentées

1. **Lazy loading** : Chargement du modèle uniquement au clic
2. **WebGL acceleration** : GPU pour les calculs matriciels
3. **Overlap factor** : 0.5 pour meilleure réactivité
4. **Threshold adaptatif** : 0.75 pour réduire les faux positifs

---

## Déploiement sur GitHub Pages

### Configuration

Le déploiement est automatisé via GitHub Pages :

```yaml
# Configuration GitHub Pages
source: main branch
folder: / (root)
url: https://titansage02.github.io/tm-my-audio-model/
```

### Avantages

- ✅ **Déploiement instantané** : Push → Live en < 1 minute
- ✅ **HTTPS natif** : Requis pour getUserMedia()
- ✅ **CDN global** : Faible latence mondiale
- ✅ **Coût zéro** : Hébergement gratuit illimité

### Contraintes techniques

⚠️ **Permissions microphone** : Requiert HTTPS (GitHub Pages ✓)  
⚠️ **Compatibilité navigateur** : Chrome, Firefox, Edge modernes  
⚠️ **Taille limite** : 100 MB (notre projet : ~1.5 MB)

---

## Défis rencontrés et solutions

### 1. **Gestion du bruit ambiant**

**Problème** : Détection instable en environnement bruyant

**Solution** :
```javascript
// Seuil de confiance adaptatif
const probabilityThreshold = 0.75;

// Moyenne glissante sur 5 frames
let predictionHistory = [];
function smoothPredictions(newPred) {
    predictionHistory.push(newPred);
    if (predictionHistory.length > 5) {
        predictionHistory.shift();
    }
    return averagePredictions(predictionHistory);
}
```

### 2. **Latence audio**

**Problème** : Délai entre son et détection

**Solution** :
- Réduction de la taille des chunks audio (23ms)
- Overlap factor à 0.5 pour détection continue
- Utilisation de Web Audio API native (low-level)

### 3. **Compatibilité navigateurs**

**Problème** : API différentes selon les navigateurs

**Solution** :
```javascript
// Polyfill pour getUserMedia
navigator.mediaDevices.getUserMedia = 
    navigator.mediaDevices.getUserMedia ||
    navigator.webkitGetUserMedia ||
    navigator.mozGetUserMedia;

// Vérification des capacités
if (!navigator.mediaDevices) {
    alert("Votre navigateur ne supporte pas l'accès au microphone");
}
```

---

## Résultats et impact

### Métriques de réussite

✅ **Application fonctionnelle** : Démo live en production  
✅ **Performance temps réel** : < 100ms de latence  
✅ **Accessibilité** : Aucune installation requise  
✅ **Privacy-first** : Traitement 100% local  

### Apprentissages clés

1. **Edge AI est viable** : Les navigateurs modernes sont suffisamment puissants
2. **Teachable Machine démocratise l'IA** : Pas besoin d'être expert ML
3. **UX est cruciale** : Feedback visuel clair = adoption utilisateur
4. **Web APIs sont matures** : Capacités audio/vidéo de niveau natif

---

## Ressources et références

### Technologies utilisées

| Technologie | Version | Documentation |
|-------------|---------|---------------|
| TensorFlow.js | 1.3.1 | [tensorflow.org/js](https://www.tensorflow.org/js) |
| Teachable Machine | - | [teachablemachine.withgoogle.com](https://teachablemachine.withgoogle.com/) |
| Bootstrap | 5.3.2 | [getbootstrap.com](https://getbootstrap.com/) |
| Font Awesome | 6.4.0 | [fontawesome.com](https://fontawesome.com/) |

### Pour aller plus loin

- 📚 [Web Audio API MDN](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)
- 📚 [TensorFlow.js Audio Recognition](https://www.tensorflow.org/js/tutorials/transfer/audio_recognizer)
- 📚 [Teachable Machine Guide](https://teachablemachine.withgoogle.com/train/audio)
- 📚 [GitHub Repository](https://github.com/TitanSage02/tm-my-audio-model)

---

## Conclusion

**Classification Audio IA** démontre qu'il est possible de créer des applications d'intelligence artificielle performantes et accessibles sans infrastructure complexe. En combinant TensorFlow.js, Teachable Machine et les Web APIs modernes, nous avons développé un système de détection audio temps réel qui fonctionne entièrement dans le navigateur.

Ce projet ouvre la voie à une nouvelle génération d'applications web intelligentes qui respectent la vie privée des utilisateurs tout en offrant des performances comparables aux solutions cloud.

### Essayez maintenant !

👉 **Démo live** : [https://titansage02.github.io/tm-my-audio-model/](https://titansage02.github.io/tm-my-audio-model/)  
💻 **Code source** : [GitHub - TitanSage02/tm-my-audio-model](https://github.com/TitanSage02/tm-my-audio-model)

---

**Développé avec ❤️ au CREC**  
*Centre de Recherche et d'Expertise en Computation*  
📅 02 Novembre 2025  
👤 **Espérance AYIWAHOUN**
