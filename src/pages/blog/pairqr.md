---
layout: ../../layouts/BlogPost.astro
title: "PairQR : Plateforme P2P de Partage Chiffré par QR Code"
description: "Système de partage de fichiers et messages peer-to-peer avec chiffrement end-to-end, appairage QR et WebRTC pour la communication temps réel."
date: "2025-04-20"
category: "Web & Sécurité"
tags: ["WebRTC", "P2P", "Encryption", "QR Code", "React", "Node.js", "Docker"]
author: "Espérance AYIWAHOUN"
---

## 🔒 Introduction

**PairQR** est une plateforme de partage **peer-to-peer** qui permet d'échanger des textes et fichiers de manière **sécurisée** via **QR code**. Les données sont chiffrées côté client et transitent directement entre utilisateurs via **WebRTC**.

**📦 Code source :** [GitHub - PairQR](https://github.com/votre-username/PairQR)

---

## ✨ Caractéristiques

✅ **End-to-end encryption** : Chiffrement AES-256 côté client  
✅ **Appairage QR code** : Connexion instantanée par scan  
✅ **WebRTC P2P** : Pas de stockage serveur  
✅ **Sessions éphémères** : Aucune persistance  
✅ **STUN/TURN (coturn)** : Traversée NAT

---

## 🏗️ Architecture

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

## 🚀 Déploiement Docker

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

---

**🌟 Use case :** Partage sécurisé de fichiers confidentiels sans cloud.