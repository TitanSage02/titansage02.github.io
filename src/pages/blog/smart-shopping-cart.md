---
layout: ../../layouts/BlogPost.astro
title: "Smart Shopping Cart : Caddie Autonome avec Localisation Indoor BLE"
description: "Système de caddie intelligent qui suit automatiquement le client dans un magasin via trilatération Bluetooth Low Energy (BLE) et ESP32."
date: "2025-03-10"
category: "IoT & Robotique"
tags: ["IoT", "BLE", "ESP32", "Arduino", "MQTT", "Trilateration", "Indoor Positioning", "Autonomous"]
author: "Espérance AYIWAHOUN"
---

## 🛒 Introduction

**Smart Shopping Cart** est un projet de caddie intelligent qui suit automatiquement le client dans une boutique grâce à un système de **localisation en intérieur** basé sur la **trilatération Bluetooth Low Energy (BLE)**.

**📦 Code source :** [GitHub - SmartCart](https://github.com/votre-username/Smart-Shopping-Cart)

---

## 🏗️ Architecture Système

```
┌─────────────┐
│  Tag BLE    │ (Client porte un ESP32)
│  (ESP32)    │
└──────┬──────┘
       │ Signal BLE
       │
┌──────▼──────────────────────────────┐
│  Ancres ESP32 (BLE Scanners)        │
│  • Anchor 1 : (x1, y1)              │
│  • Anchor 2 : (x2, y2)              │
│  • Anchor 3 : (x3, y3)              │
└──────┬──────────────────────────────┘
       │ RSSI → MQTT
       │
┌──────▼──────────────────────────────┐
│  Serveur Python                     │
│  • Calcul trilatération             │
│  • Estimation position (x, y)       │
└──────┬──────────────────────────────┘
       │ Commandes → MQTT
       │
┌──────▼──────────────────────────────┐
│  Caddie Autonome                    │
│  • ESP32 (réception)                │
│  • Arduino + L298N (moteurs)        │
│  • Écran (prix des articles)        │
└─────────────────────────────────────┘
```

---

## ✨ Fonctionnalités

✅ **Localisation indoor BLE** : Trilatération RSSI  
✅ **Suivi autonome** : Le caddie suit le client  
✅ **Affichage temps réel** : Prix des produits scannés  
✅ **Architecture distribuée** : MQTT pour communication  
✅ **Évitement d'obstacles** (futur)

---

## 🔧 Implémentation

### Calcul de Trilatération

```python
import numpy as np
from scipy.optimize import least_squares

def trilateration(anchors: list, distances: list) -> tuple:
    """
    Calcule la position (x, y) du tag BLE.
    
    anchors: [(x1, y1), (x2, y2), (x3, y3)]
    distances: [d1, d2, d3] calculés depuis RSSI
    """
    
    def equations(p):
        x, y = p
        return [
            np.sqrt((x - anchors[i][0])**2 + (y - anchors[i][1])**2) - distances[i]
            for i in range(len(anchors))
        ]
    
    # Résolution par moindres carrés
    result = least_squares(equations, x0=[0, 0])
    return result.x  # (x, y)

# Conversion RSSI → Distance (modèle empirique)
def rssi_to_distance(rssi: int, tx_power: int = -59) -> float:
    """
    Formule : d = 10 ^ ((TxPower - RSSI) / (10 * n))
    n = 2 (free space)
    """
    return 10 ** ((tx_power - rssi) / (10 * 2))
```

### Communication MQTT

```python
import paho.mqtt.client as mqtt

client = mqtt.Client()
client.connect("localhost", 1883, 60)

# Publication des commandes au caddie
def send_cart_command(direction: str, speed: int):
    payload = f"{direction},{speed}"
    client.publish("cart/control", payload)

# Exemple : avancer à 50%
send_cart_command("forward", 50)
```

---

## 📊 Performance

| Métrique | Valeur |
|----------|--------|
| **Précision de localisation** | ±1.5m |
| **Fréquence de mise à jour** | 2 Hz |
| **Portée BLE** | 30m |
| **Latence commande** | ~200ms |

---

## 🔮 Améliorations Futures

- [ ] Filtrage de Kalman pour précision
- [ ] Capteurs ultrason (évitement obstacles)
- [ ] Paiement automatique intégré

---

**🌟 Innovation :** Expérience shopping sans friction avec IoT et robotique.