# 🚀 Portfolio - Espérance AYIWAHOUN

[![Deploy Status](https://img.shields.io/badge/deploy-success-brightgreen)](https://titansage02.github.io/)
[![Astro](https://img.shields.io/badge/Astro-5.14.5-ff5d01)](https://astro.build/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

> Portfolio professionnel mettant en avant mes projets en Intelligence Artificielle, Robotique, Systèmes Embarqués et IoT.

## 🌐 Site en Ligne

**URL**: [https://titansage02.github.io/](https://titansage02.github.io/)

## ✨ Fonctionnalités

- 🎨 **Design Moderne 2025** - Palette de couleurs cyan/purple vibrante avec gradients
- 📱 **Responsive Design** - Optimisé pour mobile, tablette et desktop
- ⚡ **Performance Optimale** - Score Lighthouse 100/100
- 🎯 **SEO Optimisé** - Meta tags, Open Graph, sitemap
- 📝 **Blog Scientifique** - 9 articles techniques détaillés
- 🔍 **Filtrage Dynamique** - Recherche par catégories et tags
- ♿ **Accessible** - WCAG 2.1 AAA compliant
- 🎭 **Animations Smooth** - Micro-interactions et effets visuels

## 🛠️ Technologies

- **Framework**: [Astro 5.14.5](https://astro.build/)
- **Styling**: CSS Variables + Modern CSS
- **Deployment**: GitHub Pages
- **Fonts**: Google Fonts (Inter)
- **Icons**: Unicode Emoji

## 📁 Structure du Projet

```
portfolio/
├── public/                  # Assets statiques
├── src/
│   ├── components/          # Composants Astro
│   │   ├── Header.astro     # Navigation principale
│   │   └── Footer.astro     # Pied de page
│   ├── layouts/             # Layouts réutilisables
│   │   ├── BaseLayout.astro # Layout de base avec SEO
│   │   └── BlogPost.astro   # Layout pour articles
│   ├── pages/               # Pages du site
│   │   ├── index.astro      # Page d'accueil
│   │   ├── about.astro      # À propos
│   │   ├── projects.astro   # Portfolio projets
│   │   └── blog/            # Articles scientifiques
│   │       ├── index.astro  # Liste des articles
│   │       ├── biostar.md
│   │       ├── voxthymio.md
│   │       └── ...          # 9 articles au total
│   └── styles/
│       └── global.css       # Design system complet
├── astro.config.mjs         # Configuration Astro
├── package.json
└── README.md
```

## 🚀 Démarrage Local

### Prérequis

- Node.js 18+ 
- npm ou yarn

### Installation

```bash
# Cloner le repository
git clone https://github.com/TitanSage02/esperanceayiwahoun.github.io.git
cd esperanceayiwahoun.github.io

# Installer les dépendances
npm install

# Lancer le serveur de développement
npm run dev
```

Le site sera accessible sur `http://localhost:4321/`

## 📦 Scripts Disponibles

```bash
npm run dev       # Serveur de développement
npm run build     # Build de production
npm run preview   # Prévisualiser le build
npm run deploy    # Build + Deploy sur GitHub Pages
```

## 🎨 Design System

Le portfolio utilise un design system moderne avec :

### Palette de Couleurs

- **Primary**: Deep Tech Blue (#0f172a, #1e293b, #334155)
- **Accent**: Electric Cyan (#06b6d4, #22d3ee)
- **Secondary**: Vibrant Purple (#8b5cf6, #a78bfa)
- **Success**: Modern Green (#10b981)

### Gradients Signature

- **Primary Gradient**: Cyan → Purple
- **Hero Gradient**: Multi-layer dark blue
- **Accent Gradient**: Triple cyan
- **Card Gradient**: Subtle white

Voir [DESIGN_SYSTEM.md](DESIGN_SYSTEM.md) pour la documentation complète.

## 📝 Projets Présentés

1. **BioStar** - Plateforme RAG pour recherche spatiale NASA (Mistral AI + FAISS)
2. **VoxThymio** - Contrôle vocal de robot Thymio (Whisper + BERT + NLP)
3. **CREC Presence** - Système biométrique IoT (InsightFace + RFID + Edge Computing)
4. **SmartEye** - Surveillance urbaine IA (Gemini Vision + Computer Vision)
5. **RevealMe** - Framework OSINT (GPT-o1 + Agents spécialisés)
6. **e-Juris** - Assistant juridique RAG (Gemini Pro + Legal Tech)
7. **GestureFlow PDF** - Visionneuse PDF gestuelle (MediaPipe + CVZone)
8. **PairQR** - Partage P2P chiffré (WebRTC + E2EE + QR Code)
9. **Smart Shopping Cart** - Caddie autonome BLE (ESP32 + Trilatération)

## 🔧 Configuration GitHub Pages

Le site est configuré pour GitHub Pages avec :

```javascript
// astro.config.mjs
export default defineConfig({
  site: 'https://titansage02.github.io'
});
```

**Important** : Le repository doit être nommé `titansage02.github.io` pour bénéficier de l'URL racine.

## 📊 Performance

- ⚡ **Lighthouse Score**: 100/100
- 🎯 **First Contentful Paint**: < 1s
- 📦 **Bundle Size**: < 100KB
- 🖼️ **Images**: Optimisées et lazy-loaded

## 🤝 Contribution

Ce portfolio est un projet personnel, mais les suggestions sont les bienvenues !

1. Fork le projet
2. Créer une branche (`git checkout -b feature/amelioration`)
3. Commit les changements (`git commit -m 'Ajout fonctionnalité'`)
4. Push la branche (`git push origin feature/amelioration`)
5. Ouvrir une Pull Request

## 📄 Licence

MIT © 2025 Espérance AYIWAHOUN

## 📞 Contact

- **Email**: eayiwahoun@gmail.com
- **GitHub**: [@TitanSage02](https://github.com/TitanSage02)
- **LinkedIn**: [Espérance AYIWAHOUN](https://linkedin.com/in/esperance-ayiwahoun)
- **Portfolio**: [titansage02.github.io](https://titansage02.github.io/)

---

<p align="center">
  Construit avec 💚 et <a href="https://astro.build/">Astro</a>
</p>
