# 🎼 Trombinoscope de l'orchestre

Application web légère et autonome pour gérer et partager le trombinoscope d'un orchestre : fiches des musiciens avec photos, filtres par nationalité, famille d'instrument, instrument et concerts, suivi des anniversaires, et synchronisation commune pour que tout le monde voie la même liste.

- **Aucune installation** : un seul fichier `index.html`
- **Coût : 0 €** (GitHub Pages + Cloudflare Workers/KV, plans gratuits)
- **Données protégées** : accès par mot de passe, lecture et écriture séparées

---

## ✨ Fonctionnalités

- **Fiches musiciens** : prénom, nom, rôle, nationalité, date de naissance, famille d'instrument (Cordes, Bois, Cuivres, Percussions, Claviers, Chant, Direction, Autre), instrument, photo, notes
- **Concerts** : création de concerts, inscription des musiciens, filtre par concert avec effectifs
- **Filtres combinables** : recherche texte, nationalité, instrument, famille, concert + tri par nom, prénom, pays ou instrument
- **🎂 Anniversaires** : panneau des prochains anniversaires classés par date, avec âge
- **Deux niveaux d'accès** :
  - *Lecture* — consulter la liste, filtrer, imprimer, exporter en CSV
  - *Écriture (bureau)* — ajouter/modifier/supprimer fiches et concerts, importer/exporter en JSON, tout effacer
  - En mode lecture, l'interface n'affiche aucune fonction de modification
- **Import/Export** :
  - Export CSV (liste filtrée, ouvrable dans Excel)
  - Export/Import JSON (sauvegarde complète avec photos **et identifiants**)
  - L'import fusionne intelligemment : mise à jour par `id`, ou par prénom + nom à défaut ; les photos existantes ne sont jamais effacées par accident

---

## 🏗️ Architecture

```
┌─────────────────┐        ┌──────────────────────────┐
│  Navigateur     │  HTTP  │  Cloudflare Worker       │
│  index.html     │ ─────► │  + KV (base trombinoscope)│
│  (GitHub Pages) │        │  Lecture : READ_TOKEN    │
└─────────────────┘        │  Écriture : AUTH_TOKEN   │
                           └──────────────────────────┘
```

- `index.html` hébergé sur **GitHub Pages** (ou Netlify) : l'interface, servie à tous
- **Cloudflare Worker + KV** : stocke la liste complète (membres, photos, concerts) et la fournit à tous les visiteurs
- Chaque modification est poussée vers le Worker ; chaque visiteur voit la liste commune

## 👤 Rôles

| Rôle | Mot de passe (Secret Cloudflare) | Droits |
|---|---|---|
| Membre | `READ_TOKEN` | Consulter, filtrer, imprimer, CSV |
| Bureau / admin | `AUTH_TOKEN` | Tout, y compris modifications, import/export, suppression |

---

## 🚀 Installation

### 1. Le serveur de données (Cloudflare) — ~10 min, gratuit

1. Créez un compte sur <https://dash.cloudflare.com> (sans carte bancaire)
2. **Workers & Pages** → *Create* → nommez-le `trombinoscope` → *Deploy*
3. **Storage & Databases** → **KV** → *Create a namespace* → nommez-le `TROMBI`
4. Dans le worker `trombinoscope` → **Settings** → **Bindings** → *Add* → *KV Namespace* :
   - Variable name : `TROMBI`
   - Namespace : `TROMBI`
5. **Settings** → **Variables & Secrets** → *Add* (type **Secret**), deux entrées :
   - `READ_TOKEN` = mot de passe des membres (lecture)
   - `AUTH_TOKEN` = mot de passe du bureau (écriture)
6. Onglet **Edit code** → remplacez tout le contenu par le fichier `worker.js` de ce projet → **Deploy**
7. Notez l'adresse du worker : `https://trombinoscope.<votre-compte>.workers.dev`

> Test : ouvrir `https://.../data` doit afficher `unauthorized`.

> ⚠️ Le fichier `index.html` ci-joint appelle `https://trombinoscope.VOTRE-COMPTE.workers.dev`. **Modifiez cette ligne** (`SERVER_URL`, en tête du `<script>`) avant de publier.

### 2. Le site (GitHub Pages) — ~5 min, gratuit

1. Créez un dépôt **public** sur <https://github.com>
2. Déposez-y `index.html` et `README.md` (ce fichier)
3. **Settings** → **Pages** → Source : `main` / `(root)` → Save
4. Patientez 1–2 min : l'URL s'affiche en haut de la page Pages :
   `https://<votre-nom>.github.io/<nom-du-depot>/`

### 3. Partage

- Envoyez le **lien du site** aux membres, avec le **mot de passe de lecture** (`READ_TOKEN`)
- Le **mot de passe d'écriture** (`AUTH_TOKEN`) est réservé au bureau
- Chacun clique sur **🔑 Connexion**, saisit son mot de passe (masqué, mémorisé dans son navigateur) : l'application détecte automatiquement les droits

---

## 🔧 Configuration de `index.html`

En tête du `<script>`, un bloc à modifier **une seule fois** avant publication :

```js
var SERVER_URL = "https://trombinoscope.VOTRE-COMPTE.workers.dev";
```

C'est la **seule** valeur à renseigner. **Aucun mot de passe ne doit figurer dans le fichier** : le dépôt GitHub étant public, quiconque afficherait le code source les verrait. Les mots de passe sont saisis par chaque utilisateur dans le menu 🔑 Connexion et mémorisés dans son navigateur.

## 🔄 Utilisation courante

| Action | Comment |
|---|---|
| Ajouter un musicien | **+ Ajouter un membre** (bureau) — prénom, nom, rôle, nationalité, date de naissance, famille, instrument, concerts cochés |
| Modifier une fiche | Cliquer sur la carte — les champs sont modifiables directement |
| Ajouter/changer une photo | Dans la fiche, **📷 Ajouter/Changer la photo** (max ~3 Mo par photo) |
| Créer un concert | Menu **Concerts** → **+ Nouveau concert** |
| Supprimer un concert | Choisir le concert dans la liste → **🗑️ Supprimer** (bureau) — le retire aussi des fiches |
| Voir les anniversaires | Panneau **🎂 Prochains anniversaires**, trié par date |
| Sauvegarder la base | **⬇️ Export** (bureau) → fichier JSON complet |
| Restaurer / mettre à jour en masse | **⬆️ Import** (bureau) → fusion par `id` ou prénom + nom |
| Repartir de zéro | Sauvegarde d'abord (**⬇️ Export**), puis **🗑️ Tout effacer** |

## 📥 Format d'import JSON

```json
{
  "members": [
    {
      "id": "a1b2c3d4",
      "firstName": "Adèline",
      "lastName": "Robinson",
      "nationality": "FR",
      "birthdate": "1992-03-14",
      "role": "Saxophoniste",
      "instrument": "Saxophone",
      "category": "Bois",
      "concerts": ["Bersenbrück 2025"],
      "notes": ""
    }
  ],
  "concerts": ["Bersenbrück 2025"]
}
```

- `id` **fortement recommandé** : garantit la mise à jour de la bonne fiche d'un import à l'autre (générez une colonne id stable dans votre tableur)
- Sans `id`, la fusion se fait par prénom + nom identiques
- Clés tolérantes : `firstname`/`prénom`, `lastname`/`nom`, `birthday`/`dateNaissance`… sont acceptées
- `photo` peut contenir une image encodée (data URL) ; à l'import, une photo existante n'est jamais écrasée par une valeur vide

## 🔐 Sécurité

- Sans mot de passe, le Worker refuse tout accès (`unauthorized`) : aucune donnée n'est exposée publiquement
- Les mots de passe sont stockés en **Secrets** Cloudflare, jamais dans le code du site
- Pour **révoquer un accès** : changer les Secrets dans Cloudflare et communiquer les nouveaux mots de passe
- Charge maximale de la base KV : 25 Mo — largement suffisant pour ~100 musiciens avec des photos de taille raisonnable
- Limites connues : pas d'historique ni corbeille (d'où l'export de sauvegarde) ; en cas de modifications simultanées, la dernière écriture gagne (penser à **🔄 Actualiser** avant une grosse session de saisie)

## ❓ FAQ

**Le site affiche « Hors ligne — site non configuré »** : la ligne `SERVER_URL` n'a pas été modifiée avant publication, ou le panneau 🔑 Connexion a une URL erronée.

**« Mot de passe incorrect »** : le mot de passe saisi ne correspond ni à `READ_TOKEN` ni à `AUTH_TOKEN`.

**« Lecture seule » s'affiche alors que j'ai le mot de passe du bureau** : vous avez saisi le mot de passe de lecture ; rouvrez 🔑 Connexion et saisissez celui d'écriture.

**Un membre voit encore d'anciennes données** : cliquez sur **🔄 Actualiser** (le rafraîchissement automatique a lieu toutes les 2 minutes).

---

*Application conçue pour un orchestre amateur — adaptez librement les familles d'instruments, les couleurs et les intitulés à votre ensemble.*