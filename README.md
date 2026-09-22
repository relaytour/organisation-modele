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

## Plusieurs activités

Ce gabarit décrit une seule activité, en disposition plate : les périmètres, les fiches et les tâches types sont à la racine de `contenu/`. Cette activité reprend le slug et le nom de votre organisation.

Votre organisation mène peut-être plusieurs activités : un événement, une section qui vit à la saison, un conseil d'administration. Dans ce cas, rangez chacune dans son dossier, avec un fichier `activite.yaml` (ADR 0008 de Relaytour) :

```
contenu/
  organisation.yaml
  modeles/fiche.md
  activites/<activite>/activite.yaml     nom, nature, groupes de périmètres, identité propre
  activites/<activite>/medias/           logo de l'activité (facultatif)
  activites/<activite>/perimetres.yaml
  activites/<activite>/fiches/…
  activites/<activite>/taches/…
```

```yaml
# contenu/activites/section-natation/activite.yaml
slug: section-natation # le nom du dossier
nom: Section natation
nature: SAISON # EVENEMENT (édition), SAISON (saison) ou MANDAT (mandat)
groupes: # facultatif : sport et pôle par défaut
  - cle: equipe
    libelle: Équipe
    libellePluriel: Équipes
  - cle: pole
    libelle: Pôle
    libellePluriel: Pôles
# Identité propre, facultative (ADR 0009) : chaque champ absent reprend celui de l'organisation.
contactRecrutement: natation@exemple.org # une adresse de rôle de l'organisation
pageEquipe: https://exemple.org/natation
logo: { png: medias/logo.png } # chemin relatif au dossier de l'activité
theme: # couleurs et fond seulement ; les polices restent celles de l'organisation
  couleurs: { primaire: '#1E4F7A' }
```

Chaque périmètre déclare alors son groupe (`groupe: equipe`) à la place de son type. Les deux dispositions ne se mélangent pas : déplacez `perimetres.yaml`, `fiches/` et `taches/` dans le dossier de votre première activité. Le dossier `content/exemple` du dépôt de Relaytour suit cette disposition.

La disposition `activites/` et les groupes exigent une version de Relaytour qui inclut l'ADR 0008. L'identité des activités, les images et `adressesRoleAutorisees` exigent l'ADR 0009. Faites suivre la variable `IMAGE` de `valider.yml` avant d'utiliser ces champs.

## Identité, images et adresses de rôle

`contenu/organisation.yaml` porte l'identité de l'organisation. Les admins peuvent aussi la modifier dans l'espace organisateur ; l'export la réécrit ici.

```yaml
domainesCourrielAutorises: [exemple.org] # domaines des adresses de rôle, jamais une messagerie grand public
adressesRoleAutorisees: [bureau.association@messagerie.example] # facultatif : boîtes partagées, une par une
logo: # facultatif : le PNG sert partout, y compris dans les mails
  png: medias/logo.png # chemin relatif à ce fichier, 512 Ko au plus
  svg: medias/logo.svg # facultatif, 128 Ko au plus, sans script ni ressource externe
favicon: medias/favicon.png # facultatif
```

Une adresse de rôle appartient à l'organisation, pas à une personne. Relaytour l'accepte dans les fiches et dans l'export sans la signaler comme donnée personnelle. Ces réglages ne limitent pas les invitations. Une association dont la boîte partagée est hébergée chez une messagerie grand public la déclare dans `adressesRoleAutorisees`. N'y déclarez jamais l'adresse d'une personne.

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
yarn workspace @relaytour/server orga:importer --edition 2027 --simulation   # --organisation <slug> si l'installation en porte plusieurs
yarn workspace @relaytour/server orga:importer --edition 2027
yarn workspace @relaytour/server orga:exporter        # écrit ici tout le contenu que l'application porte
```

L'application est la source de vérité du contenu (ADR 0009). L'export écrit l'identité, les activités, les périmètres, les fiches, les tâches types et les images. Un fichier dont le sens ne change pas reste intact, avec ses commentaires. Relisez le diff, puis commitez dans ce dépôt. Un import refuse d'écraser une modification faite dans l'application depuis le dernier export ; `--forcer` l'y autorise. Les admins peuvent aussi télécharger le contenu en archive depuis la page « Organisation ».

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

- Aucune adresse mail ni aucun numéro personnel : la validation refuse le fichier. Les contacts s'écrivent sous forme de rôles. Seules les adresses de rôle sont admises : les boîtes partagées des domaines listés dans `domainesCourrielAutorises`, et les adresses déclarées dans `adressesRoleAutorisees` (`contenu/organisation.yaml`).
- Aucun secret (mot de passe SMTP, clé de session) : ils vont dans le `.env` du serveur, jamais dans un dépôt.
- Aucun export de base ni aucune liste de personnes.

## Licence

Ce gabarit est placé dans le domaine public (CC0 1.0, voir `LICENSE`). Vous pouvez le copier, le modifier et le redistribuer sans condition. Le contenu que votre organisation y ajoute reste le sien : remplacez ou retirez ce fichier `LICENSE` si vous voulez fixer d'autres conditions.
