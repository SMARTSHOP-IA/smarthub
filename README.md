# smarthub

Portail des outils Smartshop, publié sur GitHub Pages : **https://smarthub.smartshop.fr**

`index.html` : un lien vers chaque outil (Sentinelle, Analyse clients, Smartform, BAT, Digital, Stock John Taylor, Facturation,
Designer, Willow, Monday, Sellsy). Elle ne contient aucune donnée ; chaque outil garde sa propre connexion
Google @smartshop.fr.

Pour ajouter ou modifier un outil : éditer la carte correspondante dans `index.html`.

## Digital (`digital/index.html`)

Suivi des projets de développement : https://smarthub.smartshop.fr/digital/

Projets, statut, priorité, avancement, échéance, tags (Smartshop, Visibag, Azzurément Immo, Terra Group),
notes et documents. Connexion Google @smartshop.fr, comme le portail.

Données dans Firebase (projet `smartshop-catalogues`) : Firestore, collection `digital_projets`, avec les
sous-collections `notes` et `documents`. Notifications (nouveau projet, changement de statut, document ajouté) :
`digital_projets/_journal/evenements`, et ce que chacun a déjà vu dans `digital_projets/_journal/lus/<email>`. Les documents sont des liens (Google Drive, GitHub…) : Firebase Storage
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

## Stock John Taylor (`stock-jt/index.html`)

Suivi des goodies en stock du réseau John Taylor : https://smarthub.smartshop.fr/stock-jt/

Achats du siège (ils créent le stock), commandes des agences (elles le diminuent), stock restant, rétrocession
par trimestre avec l'état des factures, base des agences et entités de facturation, contrôles automatiques
(commande non facturée, facture impayée, stock négatif, produit à identifier…). Les montants sont les lignes
de produits hors taxes, hors frais de port. Saisie manuelle des commandes hors site avec le numéro de devis Sellsy.

La page ne contient aucune donnée. Elles sont dans Firebase (projet `smartshop-catalogues`), Firestore,
`stock_jt/base/<collection>` : `articles`, `achats`, `commandes`, `entites`, `inventaires`, `versements`.
Les accès externes sont dans `stock_jt_acces/<e-mail>` et se gèrent depuis l'onglet Réglages.
L'historique se charge depuis Réglages > Importer (fichier produit par le dépôt privé `stock-john-taylor`).

Photos : le champ `image` d'un article contient soit l'adresse de la photo du produit sur le site WooCommerce
(fichier produit par `scripts/photos_woo.py` du dépôt `stock-john-taylor`, importé en mode « fusion »), soit
une photo ajoutée à la main depuis la fiche de l'article, réduite à 480 px et enregistrée dans la fiche.

Contrôle WooCommerce : le champ `stockWoo` d'un article (`suivi`, `quantite`, `etat`, `le`) est le relevé du
stock déclaré sur le site, produit par `scripts/stock_woo.py` et importé en mode « fusion ». Le catalogue le
compare au stock calculé par l'outil et l'onglet Contrôles signale les écarts.

Sauvegardes : chaque modification est notée dans `stock_jt/base/journal` (qui, quand, fiche avant et après).
Une copie complète des données est écrite dans `stock_jt/base/sauvegardes/<date>` (texte découpé dans la
sous-collection `morceaux`) dans les 30 secondes qui suivent la dernière modification, avant chaque import et
avant chaque restauration. Les 100 dernières copies automatiques sont gardées, les copies manuelles toujours.
Réglages > Sauvegardes automatiques permet de les télécharger ou de restaurer l'une d'elles.

`/stock-jt/?local=1` ouvre un mode essai sans connexion : les données restent dans le navigateur.

### Règles d'accès

Dans Firestore > Règles, avant les deux dernières accolades fermantes :

```
match /stock_jt/{document=**} {
  allow read, write: if request.auth != null && request.auth.token.email_verified == true
    && (request.auth.token.email.lower().matches('.*@smartshop[.]fr$')
        || exists(/databases/$(database)/documents/stock_jt_acces/$(request.auth.token.email.lower())));
}
match /stock_jt_acces/{email} {
  allow read: if request.auth != null && request.auth.token.email_verified == true
    && (request.auth.token.email.lower().matches('.*@smartshop[.]fr$') || request.auth.token.email.lower() == email);
  allow write: if request.auth != null && request.auth.token.email_verified == true
    && request.auth.token.email.lower().matches('.*@smartshop[.]fr$');
}
```

Sans ces règles, la page affiche « Accès Firebase à ouvrir ».

### Liens vers Sellsy

Les numéros de facture et de devis s'ouvrent dans Sellsy quand la fiche porte l'identifiant interne du document
(`sellsyFactureId`, `sellsyDevisId`). Ces identifiants viennent d'un fichier « liens Sellsy » produit à partir de
l'API Sellsy et importé en mode « fusion » dans Réglages. Sans identifiant, le numéro reste un simple texte.
