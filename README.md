# Parc PC UDAF 47 — page GitHub Pages

Ce dossier n'est **pas** déployé sur le serveur osTicket. C'est le
contenu d'un **dépôt GitHub dédié** (ex. `Udaf47/parc-pc`), publié avec
GitHub Pages. Après saisie du mot de passe, la page liste tous les PC à
la suite ; un clic sur un PC déplie ses champs :

- **PC mis à jour** (case à cocher, datée) : les PC déjà faits apparaissent **en vert**,
- le modèle (liste : Latitude 3410 par défaut, Latitude 3420, Latitude 3520, Pro 16, PC fixe),
- le site (liste : Agen par défaut, Yakari, Badd, Kirikou, Marmande, Trenque, Fumel, Montanou),
- la version de Windows (Windows 10 ou Windows 11),
- batterie changée (oui/non, masqué pour un PC fixe),
- la personne à qui le PC est attribué,
- un commentaire libre.

Recherche, filtres (à faire / déjà mis à jour, modèle, site, Windows,
batterie), compteurs en haut de page, bouton « Actualiser ».

## Sécurité : la liste est chiffrée

Le dépôt et le site GitHub Pages sont **publics**. La seule protection
est le chiffrement de `pcs.json` :

- `pcs.json` ne contient qu'un bloc illisible
  (`{"v":1,"kdf":"PBKDF2-SHA256","iter":600000,"salt":…,"iv":…,"data":…}`) ;
  chiffrement AES-256-GCM, clé dérivée du mot de passe (PBKDF2-SHA256,
  600 000 itérations). Sans le mot de passe, ni les numéros, ni les
  noms, ni les commentaires ne sont lisibles — ni sur le site, ni dans
  le dépôt, ni dans son historique.
- Le mot de passe n'est écrit nulle part (ni dans la page, ni dans le
  dépôt). Chaque appareil retient seulement la **clé** après la première
  connexion ; « 🔒 Verrouiller » l'efface (à faire sur un poste partagé).
- **Le mot de passe doit être long** (12 caractères minimum, imposé) :
  quiconque récupère le fichier peut essayer des mots de passe sur sa
  propre machine. Une phrase du type `Parc-Udaf47-Agen-2026!` convient.
- Mot de passe perdu = liste illisible : il faut la régénérer depuis
  osTicket avec `--no-merge` (les saisies faites sur la page sont
  perdues). Le noter dans le coffre à mots de passe du service.
- Les noms restent réduits à « Prénom N. » et les notes internes
  d'osTicket ne sont pas exportées (sauf `--with-notes`), par précaution.

## Fichiers

| Fichier | Rôle |
|---|---|
| `index.html` | La page (aucune dépendance externe) |
| `pcs.json` | **Non fourni ici** : généré chiffré par `php manage.php parc-pages export`, voir ci-dessous |

## Mise en place (une seule fois)

1. **Créer un nouveau dépôt** sur github.com, ex. `Udaf47/parc-pc`
   (public, il ne contiendra que des données chiffrées). Ne pas réutiliser
   `Udaf47/inventaire` : son historique contient l'ancienne liste en clair.
2. **Générer `pcs.json` chiffré**, sur le serveur osTicket, dans le
   dossier de la version déployée. Pour reprendre l'ancienne liste en
   clair (et ce qui a pu y être saisi), la télécharger d'abord depuis
   `https://github.com/Udaf47/inventaire/blob/main/pcs.json` (bouton
   « Download raw file ») puis :
   ```
   php manage.php parc-pages export --in=C:\chemin\ancien-pcs.json --out=C:\chemin\pcs.json
   ```
   Le mot de passe de la liste est demandé (saisie visible : s'assurer
   que personne ne regarde l'écran), ou lu dans la variable
   d'environnement `PARC_PAGES_PASSWORD`. Sans ancienne liste, omettre
   `--in`. Supprimer ensuite `ancien-pcs.json` du serveur.
3. **Déposer `index.html` et le `pcs.json` chiffré** dans le nouveau
   dépôt (« Add file » → « Upload files »). Vérifier en ouvrant
   `pcs.json` sur github.com : il doit commencer par `{"v":1,`.
4. **Activer GitHub Pages** : Settings → Pages → Source « Deploy from a
   branch » → branche `main`, dossier `/ (root)` → Save. Adresse :
   `https://udaf47.github.io/parc-pc/`.
5. **Jeton d'écriture** : github.com → photo de profil → Settings →
   Developer settings → Personal access tokens → Fine-grained tokens →
   le jeton existant → Edit → Repository access : ajouter le nouveau
   dépôt → Repository permissions : **Contents : Read and write** →
   Update.
6. Sur la page : mot de passe de la liste, puis **⚙ Connexion GitHub** →
   propriétaire `Udaf47`, dépôt `parc-pc`, jeton → Valider.
7. **Nettoyer l'ancien dépôt `Udaf47/inventaire`** : y supprimer
   `index.html` et `pcs.json`, désactiver son GitHub Pages, puis le
   passer en **Private** (Settings → Danger Zone → Change visibility) :
   c'est la seule façon de masquer l'historique qui contient la liste en
   clair. La synchronisation des Issues de maintenance continue de
   fonctionner (elle passe par le jeton).

## Utilisation

- **Sans jeton GitHub** : consultation seulement (après le mot de passe).
- **Avec jeton** : déplier un PC, modifier (les PC modifiés sont
  encadrés en orange), puis **Enregistrer** et attendre le message vert
  « ✓ Enregistré ». Tant que ce n'est pas fait, rien n'est sauvegardé.
  Chaque enregistrement crée un commit (chiffré) sur `pcs.json`. Sur les
  autres appareils : « ↻ Actualiser ».
- Si quelqu'un d'autre a enregistré entre-temps, la page reprend sa
  version et n'y réapplique que vos propres modifications.
- « + Ajouter un PC » pour un PC absent de l'inventaire osTicket. Il n'y
  a pas de bouton pour retirer un PC : régénérer la liste depuis
  osTicket si nécessaire.
- **Changer le mot de passe** : ⚙ Connexion GitHub → « Changer le mot de
  passe de la liste » → Enregistrer. Tous les autres appareils devront
  saisir le nouveau mot de passe.

## Remettre à jour depuis osTicket

Télécharger le `pcs.json` actuel du dépôt, puis sur le serveur :

```
php manage.php parc-pages export --out=C:\chemin\pcs.json
```

Le fichier est déchiffré avec le mot de passe, **fusionné** puis
rechiffré (même sel : les appareils déjà connectés le restent) :

- nouveaux PC de l'inventaire ajoutés, numéro/modèle/état mis à jour
  (le site et « PC mis à jour », absents de l'inventaire, sont toujours
  ceux de la page) ;
- pour chaque PC, si la page a été modifiée **plus récemment** que
  l'inventaire, les valeurs de la page (modèle, site, Windows, batterie,
  personne, commentaire) sont conservées ; sinon ce sont celles
  d'osTicket ;
- les commentaires saisis sur la page sont conservés ;
- les PC ajoutés uniquement sur la page restent dans la liste.

Redéposer ensuite le fichier dans le dépôt. `--no-merge` repart de zéro
avec un nouveau mot de passe (perd tout ce qui a été saisi sur la page).

**Les modifications faites sur la page ne remontent pas dans
l'inventaire osTicket** : la page sert de vue de terrain, l'inventaire
du SCP reste la référence.
