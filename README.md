# Module 2 — Barre de recherche (Railway)

Page autonome qui affiche une barre de recherche par mot-clé sur les leçons
ThriveCart Learn. Recherche **floue, instantanée, côté navigateur** (lib
MiniSearch via CDN). Aucune IA, aucun coût, aucun backend.

À intégrer en iframe **en haut des pages de leçons**.

## Comment ça marche

1. La page charge `lessons.json` (l'index de toutes les leçons).
2. L'utilisateur tape un mot-clé → MiniSearch filtre et classe en temps réel.
3. La page affiche **uniquement la liste des hyperliens** des leçons correspondantes,
   chacune en lien cliquable qui s'ouvre dans un **nouvel onglet**.

Les champs indexés : `title` (boost ×3), `module` (boost ×2), `content`.
Recherche en mode `prefix: true` + `fuzzy: 0.2` → tolère les fautes de frappe
et matche dès le début d'un mot.

## L'index des leçons : `lessons.json`

Format attendu :

```json
[
  {
    "module": "Nom du module",
    "title": "Titre exact de la leçon",
    "url": "https://<ton-compte>.thrivecart.com/<formation>/lesson/<id>",
    "content": "Texte de la leçon + transcript de la vidéo (pour la v2)"
  },
  ...
]
```

Le fichier livré actuellement contient **3 leçons d'exemple** (à remplacer).
Cet index sera produit automatiquement par le crawl ThriveCart (étape 3 du
projet, via Claude in Chrome).

Plus le champ `content` est riche (transcripts inclus), meilleure est la
recherche. La v2 — passage des `.mp4` dans AssemblyAI — alimentera ce champ.

## Déploiement Railway

1. Pousser ce dossier sur un repo GitHub (ou Railway CLI).
2. railway.app → **New Project → Deploy from GitHub repo** → ce repo.
3. Railway lance `npm start` → la page est servie sur le port public.
4. Settings → **Networking → Generate Domain** → tu obtiens une URL publique
   `https://<nom>.up.railway.app`.

Mise à jour de l'index plus tard : tu remplaces `lessons.json` et tu re-déploies.
Aucune autre modif nécessaire.

## Intégration dans une leçon ThriveCart

Dans le bloc HTML, **en haut** de la leçon :

```html
<iframe
  src="https://<nom-search>.up.railway.app/"
  allowtransparency="true"
  style="display:block; width:100%; height:480px;
         border:0; background:transparent;"
  title="Recherche dans la formation">
</iframe>
```

- `height:480px` laisse la place à la barre + ~10 résultats. À ajuster.
- Pour le placer en bandeau fixe collé en haut du site (suit le scroll) :

```html
<iframe
  src="https://<nom-search>.up.railway.app/"
  allowtransparency="true"
  style="position:fixed; top:0; left:0; right:0;
         width:100%; height:80px;
         border:0; background:transparent; z-index:99998;"
  title="Recherche dans la formation">
</iframe>
```

⚠️ Avec `height:80px` les résultats sont coupés. Le mode bandeau fixe demande
une logique d'agrandissement à l'ouverture (à voir plus tard avec postMessage).
Pour démarrer, l'intégration en bloc inline (premier exemple, height:480px)
est la plus simple et donne le meilleur résultat.

## Limite à connaître

L'iframe occupe une zone fixe ; la zone transparente y intercepte les clics
(comme pour le chat). Placer l'iframe en zone vide en haut de la leçon, où
ThriveCart n'a pas de boutons.
