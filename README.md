# 🚀 Revise

Revise est une application web de quiz éducatif développée avec Vue 3 et Vite. Elle permet de réviser des notions scientifiques à travers des questions à choix multiples, organisées par matières et chapitres.

## ✨ Fonctionnalités principales

- 🎯 Sélection d'une matière et d'un chapitre, ou mode aléatoire
- ❓ Affichage d'une question à la fois avec plusieurs réponses possibles
- ✅ Indication immédiate de la bonne ou mauvaise réponse
- 📊 Barre de progression dynamique
- 🔄 Boutons de navigation adaptés (recommencer, retour à l'accueil, etc.)
- 📱 Interface responsive adaptée au mobile et au desktop

## 🛠️ Technologies utilisées

- [Vue 3](https://vuejs.org/) (Composition API)
- [Vite](https://vitejs.dev/) pour le développement rapide
- TypeScript pour la robustesse du code
- 🎨 CSS personnalisé pour le style

## ⚡ Installation et lancement

1. Installez les dépendances :
   ```sh
   npm install
   ```
2. Lancez le serveur de développement :
   ```sh
   npm run dev
   ```
3. Accédez à l'application sur [http://localhost:5173](http://localhost:5173) (ou le port indiqué)

## 🗂️ Structure du projet

- `src/views/HomeView.vue` : composant principal du quiz
- `src/assets/main.css` : styles globaux
- `example.json` : exemple de structure de données pour les matières, chapitres et questions

## 🧩 Personnalisation

Vous pouvez modifier ou ajouter des matières, chapitres et questions dans le fichier `example.json` pour adapter le quiz à vos besoins.

## 🚢 Déploiement

Pour générer une version de production:
```sh
npm run build
```

---

Projet réalisé avec Vue 3, Vite et TypeScript. 🦉
