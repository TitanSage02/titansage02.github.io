---
layout: ../../layouts/BlogPost.astro
title: "PairQR : Plateforme P2P de partage chiffré par QR Code"
description: "Système de partage de fichiers et messages peer-to-peer avec chiffrement end-to-end, appairage QR et WebRTC pour la communication temps réel."
date: "2025-04-20"
category: "Web & Sécurité"
tags: ["WebRTC", "P2P", "Encryption", "QR Code", "React", "Node.js", "Docker"]
author: "Espérance AYIWAHOUN"
---

## L'obsession de la confidentialité à l'ère du numérique

Dans un monde où nos données personnelles sont constamment collectées, stockées et monétisées par les géants du web, j'ai voulu créer quelque chose de différent : un moyen de partager des informations **vraiment privées**.

L'idée de **PairQR** m'est venue en observant les pratiques de partage de fichiers actuelles. Combien de fois envoyons-nous des documents sensibles par email, les stockons-nous sur des clouds publics, ou les partageons-nous via des plateformes dont nous ne maîtrisons pas la sécurité ?

PairQR propose une alternative radicale : **zéro stockage serveur**, **chiffrement end-to-end**, et connexion directe via QR code. Vos données ne transitent que entre vous et votre destinataire.

**Code source :** [GitHub - PairQR](https://github.com/votre-username/PairQR)

---

## Les principes qui guident PairQR

**End-to-end encryption** : Chiffrement AES-256 côté client  
**Appairage QR code** : Connexion instantanée par scan  
**WebRTC P2P** : Transit direct, pas de stockage serveur  
**Sessions éphémères** : Aucune persistance de données  
**STUN/TURN (coturn)** : Traversée NAT pour tous les réseaux

---

## Architecture technique

```
┌─────────────┐        ┌─────────────┐
│  Client A   │        │  Client B   │
│  (React)    │        │  (React)    │
└──────┬──────┘        └──────┬──────┘
       │                      │
       │  WebSocket (Signal)  │
       └─────────┬────────────┘
                 │
                 v
         ┌──────────────┐
         │ Server API   │
         │ (Express)    │
         └──────────────┘
                 │
        ┌────────┴────────┐
        │                 │
   ┌────v────┐      ┌────v────┐
   │ Redis   │      │ coturn  │
   │ (Cache) │      │ (STUN/  │
   └─────────┘      │  TURN)  │
                    └─────────┘
```

### Stack

- **Frontend** : React 18, TypeScript, Vite, TailwindCSS
- **Backend** : Node.js, Express, WebSocket
- **Database** : Redis (sessions), PostgreSQL
- **P2P** : WebRTC + coturn (NAT traversal)
- **Deployment** : Docker Compose, Caddy (TLS)

---

## 🔐 Sécurité

```typescript
// Chiffrement AES-256 côté client
import CryptoJS from 'crypto-js';

const encryptData = (data: string, key: string) => {
  return CryptoJS.AES.encrypt(data, key).toString();
};

const decryptData = (encrypted: string, key: string) => {
  const bytes = CryptoJS.AES.decrypt(encrypted, key);
  return bytes.toString(CryptoJS.enc.Utf8);
};
```

**Garantie** : Le serveur ne peut pas lire le contenu.

---

## Déploiement Docker

```yaml
services:
  caddy:
    image: caddy:latest
    ports:
      - "80:80"
      - "443:443"
  
  api:
    build: ./server
    environment:
      - JWT_SECRET=${JWT_SECRET}
      - CORS_ORIGIN=${CORS_ORIGIN}
  
  redis:
    image: redis:alpine
  
  postgres:
    image: postgres:15
  
  coturn:
    image: coturn/coturn:latest
    ports:
      - "3478:3478/udp"
```

**PairQR incarne ma vision d'un internet respectueux de la vie privée, où partager ne rime plus avec surveillance.**