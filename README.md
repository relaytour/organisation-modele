# Organisation Relaytour : gabarit

Ce dépôt contient tout ce qui appartient à **votre organisation** dans Relaytour, et rien du logiciel : vos périmètres, vos fiches méthode, vos tâches types et votre configuration. Le logiciel vit dans le dépôt `relaytour/relaytour` et vous n'avez jamais à le copier ni à le forker.

Pour démarrer : créez votre dépôt depuis ce gabarit (« Use this template » sur GitHub), remplacez le contenu d'exemple par le vôtre, puis rattachez le dossier à votre installation de Relaytour selon l'une des trois façons ci-dessous.

## Structure

```
contenu/
  organisation.yaml             nom, sigle, adresses de rôle, contacts, images, thème
  medias/                       logo, favicon et icône d'application (facultatif)
  perimetres.yaml               sports et pôles, avec leur description
  modeles/fiche.md              gabarit commun d'une fiche
  fiches/communes/<slug>.md     fiches communes à tous les périmètres
  fiches/<perimetre>/<slug>.md  fiches d'un périmètre
  taches/<perimetre>.yaml       tâches types d'un périmètre, dont les tâches partagées
configuration/
  .env.organisation.example     expéditeur des mails et valeurs d'amorçage à recopier dans le .env du serveur
.github/workflows/valider.yml   validation du contenu à chaque changement
```

Les règles d'écriture (une fiche, une tâche type, aucune coordonnée personnelle) sont décrites dans le dépôt de Relaytour, dossier `content/exemple`.

## Périmètres

```yaml
# contenu/perimetres.yaml
perimetres:
  - slug: football # identifiant stable, jamais modifié
    nom: Football
    description: Le tournoi à 7 contre 7 réunit les équipes du samedi sur deux terrains. # facultatif
    groupe: sport # sport ou pole en disposition plate
    couleur: '#8FC9B7' # facultatif
    ordre: 1
    effectif: 2 # facultatif
```

`description` présente le périmètre en une ou deux phrases, 400 caractères au plus. La page « Tous les périmètres », la page du périmètre et les mails d'équipe l'affichent.

`effectif` indique le nombre de référentes et de référents souhaité pour chaque période. L'import crée l'effectif s'il manque et ne le remplace jamais : les admins le modifient ensuite dans la page « Équipe ».

Un contenu plus ancien porte `type: SPORT` ou `type: POLE` à la place du groupe. L'import le lit comme le groupe `sport` ou `pole`.

## Tâches partagées

Une tâche type se décline dans d'autres périmètres de la même activité. Dans ce gabarit, le pôle Bénévoles déclare une tâche que chaque sport reçoit à son tour (`contenu/taches/benevoles.yaml`).

```yaml
taches:
  - modele: recueillir-les-besoins-en-benevoles
    titre: Recueillir les besoins en bénévoles de chaque sport
    echeance: J-150
    declinaison:
      groupe: sport # tous les périmètres de ce groupe
      # perimetres: [football, volley]   ou une liste de périmètres, à la place du groupe
      titre: Transmettre les besoins en bénévoles au pôle Bénévoles # facultatif
      echeance: J-160 # facultatif
```

- La tâche du fichier est la tâche partagée. Elle reste dans son périmètre, avec son statut.
- Chaque périmètre cible reçoit une déclinaison : une tâche à part entière, avec son statut, ses personnes assignées et son échéance.
- Le périmètre d'origine lit l'état de chaque déclinaison sur sa tâche partagée.
- Un périmètre cible ne déclare pas lui-même le `modele` qu'il reçoit.
- Une déclinaison ne cite qu'une fiche commune.

Dans l'espace organisateur, une personne qui écrit dans un périmètre propose aussi une déclinaison à un autre périmètre, qui l'accepte ou la refuse. Les tâches partagées exigent Relaytour 0.14.0.

## Plusieurs activités

Ce gabarit décrit une seule activité, en disposition plate : les périmètres, les fiches et les tâches types sont à la racine de `contenu/`. Cette activité reprend le slug et le nom de votre organisation.

Votre organisation mène peut-être plusieurs activités : un événement, une section qui vit à la saison, un conseil d'administration. Dans ce cas, rangez chacune dans son dossier, avec un fichier `activite.yaml` (ADR 0008 de Relaytour) :

```
contenu/
  organisation.yaml
  modeles/fiche.md
  activites/<activite>/activite.yaml     nom, nature, groupes de périmètres, phases, formulaire, identité propre
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
phases: # facultatif : quatre phases par défaut
  - cle: rentree
    libelle: Rentrée
    jusquA: J+30 # dernier jour de la phase, compté depuis le premier jour de la période
  - cle: saison
    libelle: Saison # la dernière phase ne porte pas de borne
souhaitsOuverts: true # facultatif : tous les membres découvrent les périmètres et formulent leurs souhaits
formulaire: # facultatif : formulaire public pour rejoindre l'équipe, ouvert par un admin dans l'application
  introduction: La section cherche des personnes pour encadrer les séances.
  question: Votre commune # libellé d'une question propre à l'organisation
  paliers: # paliers de disponibilité proposés, 8 au plus
    - Une séance par mois
    - Une séance par semaine
# Identité propre, facultative (ADR 0009) : chaque champ absent reprend celui de l'organisation.
contactRecrutement: natation@exemple.org # une adresse de rôle de l'organisation
pageEquipe: https://exemple.org/natation
logo: { png: medias/logo.png } # chemin relatif au dossier de l'activité
theme: # couleurs et fond seulement ; les polices restent celles de l'organisation
  couleurs: { primaire: '#1E4F7A' }
```

Chaque périmètre déclare alors son groupe (`groupe: equipe`) à la place de son type. Les deux dispositions ne se mélangent pas : déplacez `perimetres.yaml`, `fiches/` et `taches/` dans le dossier de votre première activité. Le dossier `content/exemple` du dépôt de Relaytour suit cette disposition.

### Phases

Une activité découpe sa période en phases. L'espace organisateur regroupe les tâches d'un périmètre et du rétroplanning par phase.

- Une phase porte une clé, un libellé et une borne `jusquA`, de la forme `J-<jours>` ou `J+<jours>`. La borne désigne le dernier jour de la phase.
- Les bornes se suivent dans l'ordre croissant. La dernière phase ne porte pas de borne.
- Une tâche ne déclare pas sa phase : elle se range par son échéance.
- Une activité déclare 12 phases au plus. Sans la clé `phases`, elle garde quatre phases : Lancement (jusqu'à J-120), Préparation (jusqu'à J-30), Derniers réglages (jusqu'à J-1), Déroulement et bilan.

La disposition plate de ce gabarit ne décrit pas de phases : votre activité garde les quatre phases par défaut. Les admins de l'activité les modifient dans l'espace organisateur. Pour les écrire dans ce dépôt, passez le dossier en disposition `activites/`.

Le même principe vaut pour `souhaitsOuverts` et `formulaire` : ils se règlent dans l'espace organisateur, et ils ne s'écrivent ici qu'en disposition `activites/`.

### Versions requises

| Fonction | Version de Relaytour |
|---|---|
| Disposition `activites/`, groupes, identité des activités, images, `adressesRoleAutorisees` | 0.4.0 |
| `description` d'un périmètre, `souhaitsOuverts` | 0.8.0 |
| `formulaire` | 0.9.0 |
| `contactSupport` | 0.11.0 |
| `iconeApplication` | 0.13.0 |
| `phases`, `declinaison` d'une tâche type | 0.14.0 |

Faites suivre la variable `IMAGE` de `valider.yml` avant d'utiliser ces champs.

## Identité, images et adresses de rôle

`contenu/organisation.yaml` porte l'identité de l'organisation. Les admins peuvent aussi la modifier dans l'espace organisateur ; l'export la réécrit ici.

```yaml
domainesCourrielAutorises: [exemple.org] # domaines des adresses de rôle, jamais une messagerie grand public
adressesRoleAutorisees: [bureau.association@messagerie.example] # facultatif : boîtes partagées, une par une
contactSupport: support@exemple.org # facultatif : adresse ouverte par le bouton « Support »
logo: # facultatif : le PNG sert partout, y compris dans les mails
  png: medias/logo.png # chemin relatif à ce fichier, 512 Ko au plus
  svg: medias/logo.svg # facultatif, 128 Ko au plus, sans script ni ressource externe
favicon: medias/favicon.png # facultatif
iconeApplication: medias/icone-application.png # facultatif : PNG carré de 512 pixels de côté
```

`iconeApplication` donne son icône à l'application installée sur un téléphone. Sans elle, l'application porte l'icône de Relaytour.

Une adresse de rôle appartient à l'organisation, pas à une personne. Relaytour l'accepte dans les fiches et dans l'export sans la signaler comme donnée personnelle. Ces réglages ne limitent pas les invitations. Une association dont la boîte partagée est hébergée chez une messagerie grand public la déclare dans `adressesRoleAutorisees`. N'y déclarez jamais l'adresse d'une personne.

## Version de Relaytour

Ce dépôt suit une version précise de Relaytour, indiquée dans `.github/workflows/valider.yml` (variable `IMAGE`). Le gabarit pointe sur le tag `main` pour fonctionner sans réglage. Une fois votre installation en place, remplacez `main` par le numéro de la version installée (par exemple `0.14.0`), et mettez-le à jour à chaque mise à jour de votre installation.

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
yarn workspace @relaytour/server orga:exporter        # écrit ici tout le contenu que l'application porte
```

Une installation qui porte plusieurs organisations exige `--organisation <slug>` sur chaque commande de cette page : création d'édition, import et export, sur un poste comme sur un serveur.

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

L'image est publique : le workflow la lit avec le jeton du dépôt, sans secret à créer.

## Ce qui n'entre jamais ici

- Aucune adresse mail ni aucun numéro personnel : la validation refuse le fichier. Les contacts s'écrivent sous forme de rôles. Seules les adresses de rôle sont admises : les boîtes partagées des domaines listés dans `domainesCourrielAutorises`, et les adresses déclarées dans `adressesRoleAutorisees` (`contenu/organisation.yaml`).
- Aucun secret (mot de passe SMTP, clé de session) : ils vont dans le `.env` du serveur, jamais dans un dépôt.
- Aucun export de base ni aucune liste de personnes.

## Licence

Ce gabarit est placé dans le domaine public (CC0 1.0, voir `LICENSE`). Vous pouvez le copier, le modifier et le redistribuer sans condition. Le contenu que votre organisation y ajoute reste le sien : remplacez ou retirez ce fichier `LICENSE` si vous voulez fixer d'autres conditions.
