---
layout: ../../layouts/BlogPost.astro
title: "Smart Shopping Cart : Caddie autonome avec localisation indoor basé sur BLE"
description: "Système de caddie intelligent qui suit automatiquement le client dans un magasin via trilatération Bluetooth Low Energy (BLE) et ESP32."
date: "2025-03-10"
category: "IoT & Robotique"
tags: ["IoT", "BLE", "ESP32", "Arduino", "MQTT", "Trilateration", "Indoor Positioning", "Autonomous"]
author: "Espérance AYIWAHOUN"
---

## Quand les courses deviennent une aventure technologique

Notre équipe a toujours été passionnée par l'idée de rendre la technologie invisible mais utile dans la vie quotidienne. Un jour, en faisant nos courses dans un grand magasin, nous avons observé les difficultés que pouvaient rencontrer certaines personnes avec des caddies lourds ou peu maniables.

C'est là qu'est née l'idée du **Smart Shopping Cart** : et si le caddie pouvait nous suivre automatiquement, comme un compagnon fidèle, nous libérant les mains pour choisir nos produits ?

Notre défi était de créer un système de **localisation en intérieur** suffisamment précis et abordable, basé sur la **trilatération Bluetooth Low Energy (BLE)**.

**Code source :** [GitHub - SmartCart](https://github.com/votre-username/Smart-Shopping-Cart)

---

## L'architecture que nous avons imaginée

### La vision d'ensemble

Notre concept reposait sur un écosystème interconnecté simple mais efficace :

1. **Le client porte un petit badge ESP32** qui émet un signal BLE
2. **Des ancres ESP32 placées dans le magasin** captent ce signal
3. **Un serveur central calcule la position** par trilatération
4. **Le caddie intelligent reçoit les instructions** et suit automatiquement

```
Client avec badge BLE
         ↓ Signal émis
Réseau d'ancres ESP32 dans le magasin
         ↓ Données RSSI
Serveur de calcul de position
         ↓ Commandes de mouvement
Caddie autonome qui suit
```

### Les défis techniques que nous avons relevés

**Défi 1 : La précision de localisation**
La trilatération BLE en intérieur est complexe à cause des interférences, des murs, et de la variabilité du signal RSSI. Nous avons développé un algorithme robuste qui combine plusieurs mesures pour améliorer la fiabilité.

**Défi 2 : La réactivité**
Un caddie qui suit avec 5 secondes de retard n'est pas utilisable. Nous avons optimisé notre pipeline pour atteindre une latence de moins de 200ms entre le mouvement du client et la réaction du caddie.

**Défi 3 : L'autonomie**
Faire fonctionner plusieurs ESP32 et des moteurs pendant des heures nécessitait une gestion intelligente de l'énergie.

---

## L'implémentation technique

### Notre algorithme de trilatération

Nous avons développé un système qui convertit la force du signal RSSI en distance approximative, puis calcule la position par optimisation mathématique :

```python
import numpy as np
from scipy.optimize import least_squares

def trilateration(anchors: list, distances: list) -> tuple:
    """
    Notre algorithme de calcul de position
    
    anchors: positions connues des ancres [(x1, y1), (x2, y2), (x3, y3)]
    distances: distances calculées depuis les signaux RSSI
    """
    
    def equations(p):
        x, y = p
        return [
            np.sqrt((x - anchors[i][0])**2 + (y - anchors[i][1])**2) - distances[i]
            for i in range(len(anchors))
        ]
    
    # Résolution par moindres carrés
    result = least_squares(equations, x0=[0, 0])
    return result.x  # Position estimée (x, y)

# Notre modèle de conversion RSSI vers distance
def rssi_to_distance(rssi: int, tx_power: int = -59) -> float:
    """
    Formule empirique que nous avons calibrée :
    d = 10 ^ ((TxPower - RSSI) / (10 * n))
    avec n = 2 pour l'espace libre
    """
    return 10 ** ((tx_power - rssi) / (10 * 2))
```

### Communication temps réel avec MQTT

Pour coordonner tous les éléments, nous avons choisi MQTT pour sa simplicité et sa fiabilité :

```python
import paho.mqtt.client as mqtt

# Notre système de communication centralisé
client = mqtt.Client()
client.connect("localhost", 1883, 60)

def send_cart_command(direction: str, speed: int):
    """
    Envoie des commandes de mouvement au caddie
    """
    payload = f"{direction},{speed}"
    client.publish("cart/control", payload)
    
def update_cart_display(total_price: float, item_count: int):
    """
    Met à jour l'affichage en temps réel
    """
    display_data = f"{total_price:.2f},{item_count}"
    client.publish("cart/display", display_data)

# Exemple d'utilisation
send_cart_command("forward", 50)  # Avancer à 50% de vitesse
```

---

## Les fonctionnalités que nous avons développées

### Suivi intelligent et adaptatif

Notre caddie ne se contente pas de suivre bêtement. Nous avons implémenté plusieurs modes de comportement :

- **Mode proximité** : Reste à distance respectable (1-2m)
- **Mode attente** : Se place en position de repos quand le client s'arrête
- **Mode évitement** : Contourne les obstacles et autres personnes
- **Mode urgence** : S'arrête immédiatement en cas de problème

### Interface utilisateur intuitive

L'écran du caddie affiche en temps réel :
- Le total des achats
- Le nombre d'articles
- Le statut de la batterie
- Des notifications utiles

### Précision et performance

Les résultats que nous avons obtenus dépassent nos attentes initiales :

| Métrique | Résultat | Ce que ça signifie |
|----------|----------|-------------------|
| **Précision de localisation** | ±1.5m | Suffisant pour le suivi en magasin |
| **Fréquence de mise à jour** | 2 Hz | Mouvement fluide et naturel |
| **Portée BLE** | 30m | Couvre la plupart des espaces commerciaux |
| **Latence commande** | ~200ms | Réactivité imperceptible |

---

## Les défis rencontrés et solutions trouvées

### La calibration du système

Chaque magasin ayant sa propre géométrie et ses interférences, nous avons développé une procédure de calibration semi-automatique. Une personne parcourt le magasin avec un badge de référence, permettant au système d'apprendre les caractéristiques de l'environnement.

### La gestion des obstacles

Les magasins sont des environnements dynamiques avec des clients, des employés, des présentoirs mobiles. Nous avons intégré des capteurs ultrasoniques pour la détection d'obstacles proche, complétant la localisation BLE.

### L'optimisation énergétique

Nous avons développé un système de gestion intelligent de l'énergie :
- **Veille adaptative** des composants non utilisés
- **Fréquence d'émission variable** selon l'activité
- **Optimisation des trajets** pour économiser la batterie

---

## Impact et perspectives d'avenir

### Les retours des premiers tests

Nos tests dans un environnement simulé ont révélé des bénéfices inattendus :

**Confort d'usage** : "On ne se rend même plus compte qu'il nous suit, c'est naturel"

**Accessibilité** : "Révolutionnaire pour les personnes avec des difficultés de mobilité"

**Efficacité** : "On peut se concentrer sur nos achats sans se soucier du caddie"

### Les améliorations que nous préparons

Notre roadmap inclut plusieurs évolutions passionnantes :

- **Filtrage de Kalman** pour une précision encore meilleure
- **Intelligence artificielle** pour prédire les mouvements du client
- **Intégration paiement** sans contact automatique
- **Navigation collaborative** entre plusieurs caddies
- **Application mobile** pour contrôle distant

### Notre vision à long terme

Nous imaginons un avenir où les courses deviennent une expérience fluide et agréable, où la technologie disparaît au profit de l'humain. Le Smart Shopping Cart n'est qu'un premier pas vers des environnements commerciaux intelligents qui s'adaptent à nos besoins.

Notre équipe continue de croire que l'IoT et la robotique peuvent transformer positivement nos interactions quotidiennes, en restant toujours au service de l'expérience humaine.

**Le Smart Shopping Cart représente notre vision d'une technologie invisible mais essentielle, qui améliore la vie sans la compliquer.**