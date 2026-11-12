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

Données dans Firebase (projet `smartshop-catalogues`) : Firestore, collection `digital_projets`, avec les
sous-collections `notes` et `documents`. Les documents sont des liens (Google Drive, GitHub…) : Firebase Storage
n'est pas activé sur le projet (il demande le forfait Blaze). Le code d'envoi de fichiers est prêt et se réactive
en rechargeant `firebase-storage-compat.js` si Storage est activé un jour (dossier `digital/<id du projet>/`).

### Règle d'accès (publiée le 12/11/2026)

Dans Firestore > Règles, avant les deux dernières accolades fermantes :

```
match /digital_projets/{document=**} {
  allow read, write: if request.auth != null && request.auth.token.email_verified == true
                     && request.auth.token.email.lower().matches('.*@smartshop[.]fr$');
}
```

Sans cette règle, la page affiche « Accès Firebase à ouvrir ».
À la première ouverture, le bouton « Importer les 11 projets de départ » remplit la liste.
