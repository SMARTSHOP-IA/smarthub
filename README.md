# smarthub

Portail des outils Smartshop, publié sur GitHub Pages : **https://smarthub.smartshop.fr**

`index.html` : un lien vers chaque outil (Sentinelle, Analyse clients, Smartform, BAT, Digital, Facturation,
Designer, Willow, Monday, Sellsy). Elle ne contient aucune donnée ; chaque outil garde sa propre connexion
Google @smartshop.fr.

Pour ajouter ou modifier un outil : éditer la carte correspondante dans `index.html`.

## Digital (`digital/index.html`)

Suivi des projets de développement : https://smarthub.smartshop.fr/digital/

Projets, statut, priorité, avancement, échéance, tags (Smartshop, Visibag, Azzurément Immo, Terra Group),
notes et documents. Connexion Google @smartshop.fr, comme le portail.

Données dans Firebase (projet `smartshop-catalogues`) :
- Firestore : collection `digital_projets`, avec les sous-collections `notes` et `documents` ;
- Storage : dossier `digital/<id du projet>/` pour les fichiers joints (25 Mo max).

### Règles d'accès à ajouter dans la console Firebase

Firestore (Firestore Database > Règles), à l'intérieur de `match /databases/{database}/documents { … }` :

```
match /digital_projets/{document=**} {
  allow read, write: if request.auth != null
    && request.auth.token.email_verified == true
    && request.auth.token.email.matches('.*@smartshop[.]fr$');
}
```

Storage (Storage > Règles), à l'intérieur de `match /b/{bucket}/o { … }` :

```
match /digital/{allPaths=**} {
  allow read, delete: if request.auth != null
    && request.auth.token.email_verified == true
    && request.auth.token.email.matches('.*@smartshop[.]fr$');
  allow create, update: if request.auth != null
    && request.auth.token.email_verified == true
    && request.auth.token.email.matches('.*@smartshop[.]fr$')
    && request.resource.size < 25 * 1024 * 1024;
}
```

Tant que ces règles ne sont pas publiées, la page affiche « Accès Firebase à ouvrir ».
À la première ouverture, le bouton « Importer les 11 projets de départ » remplit la liste.
