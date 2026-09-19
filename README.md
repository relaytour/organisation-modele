# Organisation Relaytour : gabarit

Ce dépôt contient tout ce qui appartient à **votre organisation** dans Relaytour, et rien du logiciel : vos périmètres, vos fiches méthode, vos tâches types et votre configuration. Le logiciel vit dans le dépôt `relaytour/relaytour` et vous n'avez jamais à le copier ni à le forker.

Pour démarrer : créez votre dépôt depuis ce gabarit (« Use this template » sur GitHub), remplacez le contenu d'exemple par le vôtre, puis rattachez le dossier à votre installation de Relaytour selon l'une des trois façons ci-dessous.

## Structure

```
contenu/
  organisation.yaml             nom, sigle, domaines de mail, contact, thème
  perimetres.yaml               sports et pôles
  modeles/fiche.md              gabarit commun d'une fiche
  fiches/communes/<slug>.md     fiches communes à tous les périmètres
  fiches/<perimetre>/<slug>.md  fiches d'un périmètre
  taches/<perimetre>.yaml       tâches types d'un périmètre
configuration/
  .env.organisation.example     expéditeur des mails et valeurs d'amorçage à recopier dans le .env du serveur
.github/workflows/valider.yml   validation du contenu à chaque changement
```

Les règles d'écriture (une fiche, une tâche type, aucune coordonnée personnelle) sont décrites dans le dépôt de Relaytour, dossier `content/exemple`.

## Version de Relaytour

Ce dépôt suit une version précise de Relaytour, indiquée dans `.github/workflows/valider.yml` (variable `IMAGE`). Le gabarit pointe sur le tag `main` pour fonctionner sans réglage. Une fois votre installation en place, remplacez `main` par le tag de l'image installée (le SHA court du commit), et mettez-le à jour à chaque mise à jour de votre installation.

## Rattacher ce dossier à Relaytour

### 1. Sur un poste de travail

Pour rédiger, valider et importer depuis une installation locale de Relaytour :

```bash
# dans votre copie locale du dépôt relaytour/relaytour, fichier packages/server/.env
CONTENU_ORGA=/chemin/vers/ce/depot/contenu
# et COURRIEL_EXPEDITEUR de configuration/.env.organisation.example
```

```bash
yarn workspace @relaytour/server orga:valider
yarn workspace @relaytour/server edition:creer 2027 "Édition 2027" 2027-06-05 2027-06-06   # si l'édition n'existe pas encore
yarn workspace @relaytour/server orga:importer --edition 2027 --simulation
yarn workspace @relaytour/server orga:importer --edition 2027
yarn workspace @relaytour/server orga:exporter        # en fin d'édition : reverse les fiches modifiées ici
```

L'export écrit dans `contenu/fiches`. Relisez le diff, puis commitez dans ce dépôt.

### 2. Sur un serveur, avec l'image Docker

Clonez ce dépôt sur le serveur, par exemple dans `/srv/relaytour/contenu`, puis montez-le en volume au moment de l'import :

```bash
cd /srv/relaytour
docker compose exec server node dist/creer-edition.js 2027 "Édition 2027" 2027-06-05 2027-06-06   # si l'édition n'existe pas encore
docker compose run --rm -v /srv/relaytour/contenu/contenu:/contenu:ro server \
  node dist/orga-importer.js --dossier /contenu --edition 2027 --simulation
docker compose run --rm -v /srv/relaytour/contenu/contenu:/contenu:ro server \
  node dist/orga-importer.js --dossier /contenu --edition 2027
```

Les valeurs de `configuration/.env.organisation.example` se recopient dans le `.env` du serveur.

### 3. Dans la CI de ce dépôt

Le workflow `valider.yml` lance la validation dans l'image publiée de Relaytour, sans installer Node ni cloner le logiciel :

```bash
docker run --rm -v "$PWD/contenu:/contenu:ro" ghcr.io/relaytour/relaytour-server:main node dist/orga-valider.js /contenu
```

Tant que l'image n'est pas publique, le workflow a besoin d'un jeton en lecture seule sur les paquets, dans le secret `RELAYTOUR_IMAGE_TOKEN`.

## Ce qui n'entre jamais ici

- Aucune adresse mail ni aucun numéro personnel : la validation refuse le fichier. Les contacts s'écrivent sous forme de rôles. Seules les boîtes partagées des domaines listés dans `domainesCourrielAutorises` (`contenu/organisation.yaml`) sont admises.
- Aucun secret (mot de passe SMTP, clé de session) : ils vont dans le `.env` du serveur, jamais dans un dépôt.
- Aucun export de base ni aucune liste de personnes.

## Licence

Ce gabarit est placé dans le domaine public (CC0 1.0, voir `LICENSE`). Vous pouvez le copier, le modifier et le redistribuer sans condition. Le contenu que votre organisation y ajoute reste le sien : remplacez ou retirez ce fichier `LICENSE` si vous voulez fixer d'autres conditions.
