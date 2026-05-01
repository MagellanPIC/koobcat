# Architectoon · Vote pour l'expo

App de vote en ligne pour aider à sélectionner les œuvres d'une exposition.
Galerie virtuelle façon mur blanc IKEA — chaque œuvre dans un cadre noir fin,
plaque numérotée, pastille jaune curateur quand sélectionnée — où les visiteurs
choisissent entre 5 et 25 œuvres parmi 69.

![Aperçu](https://img.shields.io/badge/zero%20build-static%20site-EDE6D6?style=flat-square)
![Pas de backend requis](https://img.shields.io/badge/backend-optionnel-B8935A?style=flat-square)

---

## 📁 Structure

```
architectoon-vote/
├── index.html      ← App de vote (à partager avec les votants)
├── admin.html      ← Outil d'agrégation des résultats (privé)
└── README.md       ← Ce fichier
```

C'est tout. Pas de build, pas de dépendances, pas de framework.

---

## ⚡ Configuration rapide (3 minutes)

Ouvre `index.html` et trouve la section `CONFIG` au début du `<script>` :

```javascript
const CONFIG = {
  ADMIN_EMAIL: 'votre-email@example.com',  // ← TON email
  IMAGE_BASE: 'https://www.interlope.biz/koobcat/',
  MIN_VOTES: 5,
  MAX_VOTES: 25,
  DEFAULT_CATEGORY: 'architectoon',
  BACKEND_URL: ''                          // ← optionnel, voir plus bas
};
```

**Remplace `votre-email@example.com`** par ton vrai email — c'est là que les
votes arriveront.

C'est tout. L'app fonctionne.

---

## 🚀 Déploiement sur GitHub Pages

1. **Crée un repo** sur GitHub : `architectoon-vote` (ou ce que tu veux)
2. **Pousse les fichiers** :
   ```bash
   git init
   git add .
   git commit -m "Vote app pour l'expo"
   git branch -M main
   git remote add origin https://github.com/TON-USER/architectoon-vote.git
   git push -u origin main
   ```
3. **Active GitHub Pages** :
   - Settings → Pages
   - Source : **Deploy from a branch**
   - Branch : **main** / Folder : **/ (root)**
   - Save
4. **Attends 30 secondes**, ton app sera en ligne à :
   `https://TON-USER.github.io/architectoon-vote/`

Partage ce lien avec tes amis / public pour le vote.
La page admin sera à `https://TON-USER.github.io/architectoon-vote/admin.html`
(garde-la pour toi).

---

## 🗳 Comment ça marche

### Côté votant (`index.html`)

1. Arrive sur la galerie, voit les 3 collections (Architectoon, Odditorium, Snapshoots)
2. Clique sur une œuvre pour la sélectionner (cadre devient or)
3. Compteur en bas : `0/25` → `5/25` (min atteint) → `25/25`
4. Bouton "Soumettre" actif quand entre 5 et 25
5. Modal demande nom + email
6. À la soumission : un brouillon d'email s'ouvre, pré-rempli avec tous ses choix.
   Le votant clique "Envoyer" dans son client mail. Tu reçois l'email.
7. Alternative : il peut copier un "token" (texte encodé) à t'envoyer autrement
   (Messenger, WhatsApp, etc.)

Les sélections en cours sont sauvegardées localement — un votant peut fermer
l'onglet et revenir, ses choix sont gardés.

### Côté admin (`admin.html`)

1. Ouvre `admin.html` (sur ton GitHub Pages, ou en local)
2. Copie-colle tous les tokens reçus (un par ligne, ils apparaissent à la fin
   de chaque email reçu)
3. Clique "Analyser"
4. Tu vois :
   - Le nombre de voteurs
   - La liste des voteurs (nom, email, date)
   - Le classement complet des 69 œuvres par nombre de votes
   - Le top 25 mis en évidence en or
   - Répartition par collection
5. Bouton "Export CSV" pour avoir un tableur

Les tokens sont sauvegardés en local pour que tu puisses revenir et en ajouter
au fur et à mesure.

---

## 📧 Format des emails reçus

Chaque vote arrive comme ça dans ta boîte :

```
Sujet: Vote expo · Alice Dupont

Vote pour la sélection des œuvres exposées

Voteur : Alice Dupont
Email  : alice@example.com
Date   : 15/05/2026 14:32
Total  : 22 œuvres sélectionnées

--- SÉLECTIONS ---
· dessins/Architectoon.1.png
· dessins/Architectoon.18.png
· dessins/Sketches.108.png
... (etc)

--- TOKEN (pour aggregation auto) ---
eyJ2b3RlciI6eyJuYW1lIjoiQWxpY2UgRHVwb250IiwiZW1haWwiOiJhbGlj...
```

Le **token** est ce que tu copies dans `admin.html` pour l'agrégation
automatique. Tu peux aussi compter les votes manuellement à partir des
sélections en clair si tu préfères.

---

## 🔧 Upgrade optionnel : backend automatique

Si tu attends beaucoup de votes et que la collecte par email est trop lourde,
tu peux ajouter un backend gratuit Google Apps Script qui écrit chaque vote
directement dans une Google Sheet.

### Setup (5 minutes)

1. Va sur [script.google.com](https://script.google.com) → Nouveau projet
2. Colle ce code :

```javascript
function doPost(e) {
  try {
    const vote = JSON.parse(e.postData.contents);
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();

    // Header row si la sheet est vide
    if (sheet.getLastRow() === 0) {
      sheet.appendRow(['Date', 'Nom', 'Email', 'Nombre', 'Sélections']);
    }

    sheet.appendRow([
      new Date(vote.timestamp),
      vote.voter.name,
      vote.voter.email,
      vote.selections.length,
      vote.selections.join(', ')
    ]);

    return ContentService.createTextOutput(JSON.stringify({ ok: true }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService.createTextOutput(JSON.stringify({ ok: false, error: err.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

3. Crée une Google Sheet vide nommée "Votes Architectoon"
4. Dans la Sheet : Extensions → Apps Script → colle le même code
5. Déployer → Nouveau déploiement → type "Application web"
   - Exécuter en tant que : moi
   - Qui a accès : tout le monde
6. Copie l'URL de déploiement (genre `https://script.google.com/macros/s/AKfy.../exec`)
7. Dans `index.html`, remplace `BACKEND_URL: ''` par cette URL

À partir de là, chaque vote va directement dans ta Sheet en plus de l'email.
Tu peux faire des `=COUNTIF` directement pour compter.

---

## 🎨 Personnalisation

### Changer les couleurs

Dans `index.html`, en haut du `<style>`, les variables CSS :

```css
:root {
  --wall: #EDE6D6;       /* couleur du mur */
  --frame: #1A1410;      /* couleur des cadres */
  --gold: #B8935A;       /* couleur "sélectionné" */
  --ink: #221F1B;        /* texte principal */
  /* ... */
}
```

### Ajouter / retirer des œuvres

Modifie le tableau `ARTWORKS` dans `index.html` ET dans `admin.html`
(les deux doivent rester synchronisés).

### Changer min/max

Dans `CONFIG` : `MIN_VOTES` et `MAX_VOTES`.

### Catégorie par défaut

Dans `CONFIG.DEFAULT_CATEGORY` : `'architectoon'`, `'odditorium'`, ou `'snapshoots'`.

---

## 🧪 Tester en local

```bash
# Avec Python (déjà installé partout)
python3 -m http.server 8000

# Ou avec Node si tu l'as
npx serve

# Puis ouvre http://localhost:8000
```

L'app fonctionne aussi en ouvrant directement `index.html` dans le navigateur,
mais certains navigateurs bloquent `mailto:` en local.

---

## 📝 Notes

- **Pas de prévention contre les votes multiples par la même personne** — un
  motivé peut voter 10 fois avec 10 emails différents. Pour une expo entre
  amis, l'honneur civique suffit. Si tu veux verrouiller, l'upgrade backend
  permet d'ajouter une vérif par email.
- **Les images viennent de interlope.biz** (pas de copie locale dans le repo).
  Si tu déplaces le site, change `IMAGE_BASE` dans `CONFIG`.
- **Le token de vote est lisible** — c'est juste du base64. Si tu veux que
  les votes soient secrets, l'upgrade backend règle ça aussi.

---

## 📜 Licence

Fais-en ce que tu veux. Bonne expo&thinsp;!
