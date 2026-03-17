# # Météo Express Pro 🚀

![Vue.js](https://img.shields.io/badge/Vue.js-35495E?style=for-the-badge&logo=vuedotjs&logoColor=4FC08D)
![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white)

Une application de tableau de bord météorologique moderne et réactive, développée dans le cadre d'un projet professionnel. L'application fournit des données météorologiques en temps réel et des prévisions sur 5 jours avec une interface utilisateur soignée en Glassmorphism.

## 🌟 Fonctionnalités

* **📍 Géolocalisation automatique :** Détection de la position de l'utilisateur pour afficher la météo locale instantanément.
* **🔍 Recherche par ville :** Possibilité de rechercher la météo de n'importe quelle ville dans le monde.
* **🎨 Interface Dynamique :** L'arrière-plan et le thème s'adaptent visuellement aux conditions météo (Ensoleillé, Pluvieux, Nuageux, Neige).
* **📊 Données Détaillées :**
    * Température actuelle et ressenti.
    * Humidité, Vitesse et Direction du vent.
    * Pression atmosphérique et Visibilité.
    * Heures de Lever et Coucher du soleil.
* **📅 Prévisions sur 5 jours :** Affichage intelligent des tendances futures.
* **💎 Design Glassmorphism :** Interface moderne avec effets de transparence et de flou.

## 🛠️ Technologies Utilisées

* **Vue.js 3 (Composition API) :** Pour la logique réactive et la gestion d'état, utilisé via CDN pour une légèreté maximale (sans étape de build complexe).
* **OpenWeatherMap API :** Fournisseur de données météorologiques en temps réel et prévisionnelles.
* **CSS3 natif :** Design responsive, animations (Keyframes) et mise en page Grid/Flexbox.
* **RemixIcon :** Bibliothèque d'icônes vectorielles.
* **Google Fonts :** Typographie "Poppins".

## 🚀 Installation et Utilisation

Ce projet a été conçu pour être **ultra-simple à lancer**, sans nécessiter l'installation de Node.js ou de gestionnaires de paquets (npm).

### Prérequis
* Un navigateur web moderne (Chrome, Firefox, Edge).
* Une clé API (gratuite) de chez [OpenWeatherMap](https://openweathermap.org/).

### Étapes
1.  **Cloner le projet** (ou télécharger le fichier ZIP) :
    ```bash
    git clone [https://github.com/VOTRE_NOM_UTILISATEUR/meteo-express-pro.git](https://github.com/VOTRE_NOM_UTILISATEUR/meteo-express-pro.git)
    ```
2.  **Configuration de la clé API :**
    * Ouvrez le fichier `index.html` dans un éditeur de code.
    * Cherchez la ligne : `const API_KEY = 'TA_CLE_API_ICI'`
    * Remplacez `'TA_CLE_API_ICI'` par votre propre clé API OpenWeatherMap.
3.  **Lancer l'application :**
    * Double-cliquez simplement sur le fichier `index.html`.
    * L'application s'ouvrira directement dans votre navigateur.

## 📸 Aperçu

*(Tu peux ajouter ici les captures d'écran que tu m'as montrées. Glisse les images dans ton dépôt GitHub et mets les liens ici)*

## 📝 Contexte du projet

Ce projet a été réalisé dans le cadre d'une **Situation Professionnelle (Avril 2026)**.
L'objectif était de démontrer la capacité à :
* Consommer une API REST externe (Fetch/Async/Await).
* Manipuler le DOM via un framework JavaScript (Vue.js).
* Gérer les erreurs et les états de chargement.
* Produire une interface utilisateur ergonomique et adaptative.

---
*Développé par Djalo Amadou
