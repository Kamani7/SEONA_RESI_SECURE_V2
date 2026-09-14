# SEONA RESI — V2 architecture + security baseline

## Architecture
- `index.html` — interface
- `assets/css/styles.css` — CSS
- `assets/js/firebase-config.js` — configuration Firebase publique
- `assets/js/app.js` — logique de l'application
- `assets/js/mobile-menu.js` — navigation mobile
- `firebase.json` — Firebase Hosting + HTTP security headers
- `firebase/firestore.rules` — règles Firestore renforcées
- `firebase/storage.rules` — règles Storage renforcées
- `index.legacy.backup.html` — sauvegarde de l'ancien fichier

## Audit du fichier fourni
Le fichier actuel contient une authentification propriétaire/gestionnaire/admin réalisée dans le navigateur : le hash admin est présent dans le JavaScript, les hashes des autres comptes sont récupérés depuis Firestore, et la session est conservée dans `localStorage`. La CNI est aussi enregistrée comme Data URL dans Firestore. Les opérations sensibles (validation, blocage, suppression, droits, solde/transferts) sont appelées directement depuis le navigateur.

Le fichier contient déjà une fonction d'échappement anti-XSS, ce qui est une bonne protection contre une partie des injections HTML.

## Important
Le refactoring ci-dessus sépare le code et ajoute des en-têtes de sécurité. En revanche, cela ne suffit pas à rendre l'authentification et la comptabilité réellement sécurisées.

Les règles Firebase fournies sont volontairement basées sur Firebase Authentication et des custom claims `admin`, `staff` et `owner`. Elles sont donc à appliquer après migration de l'authentification.

NE PAS utiliser en production des règles Firestore du type `allow read, write: if true`.

Pour une vraie version commerciale :
1. Firebase Authentication pour tous les comptes.
2. CNI dans Firebase Storage privé, pas dans Firestore.
3. Rôles via custom claims.
4. Calcul des commissions, soldes, retraits et validations côté serveur (Cloud Functions/Cloud Run).
5. Paiements confirmés par webhook côté serveur, jamais par une valeur envoyée par le navigateur.
6. Journal d'audit des opérations sensibles.
7. Limitation de taille/type des fichiers et validation serveur.

## Déploiement Firebase
```bash
npm install -g firebase-tools
firebase login
firebase use --add
firebase deploy --only hosting
firebase deploy --only firestore:rules
firebase deploy --only storage
```

Sélectionner le projet Firebase `seona-resi`.

## GitHub Pages
Le frontend statique peut être publié sur GitHub Pages, mais pour l'application commerciale, Firebase Hosting est recommandé. Ne jamais mettre dans GitHub une clé privée, un secret de paiement, un token serveur ou un credential d'administration.

## Avertissement
Cette V2 est une étape de refactoring et de durcissement. Elle ne remplace pas la migration vers Firebase Authentication + backend sécurisé pour les fonctions sensibles.
