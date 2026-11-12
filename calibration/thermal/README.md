# Calibrations thermiques expérimentales

Déposer ici les fichiers originaux issus des essais en enceinte climatique, classés par génération matérielle :

```text
calibration/
└── thermal/
    ├── P508_generation_1/
    │   └── RECORDS.csv
    └── P606_generation_2/
        └── RECORDS.csv
```

Ne pas réenregistrer les fichiers dans un tableur : conserver les octets d'origine. Le chemin relatif et les noms de
colonnes doivent être déclarés dans `lab/lab.json`, sous `thermal_calibrations`.

Pour chaque fichier, documenter avant activation :
- la génération matérielle exacte et les capteurs utilisés ;
- quelle colonne représente la température externe, la température interne et la pression ;
- les unités, le fuseau horaire et le pas d'échantillonnage ;
- la pression de référence appliquée pendant l'essai et les conditions de l'expérience.

Le pipeline calcule l'empreinte SHA-256 du fichier et l'enregistre dans le manifeste de chaque point traité. Ne pas
remplacer silencieusement un fichier déjà utilisé : conserver une version distincte et mettre à jour explicitement
la référence dans le laboratoire.

Ce dossier ne contient volontairement aucun fichier de calibration réel tant que sa provenance et ses colonnes
n'ont pas été confirmées.


## Générer automatiquement la déclaration dans lab.json

Il n'est pas nécessaire de modifier le JSON à la main. Depuis le dépôt `MOLO-data-process` (branche
`CGS-MOLO`), avec l'environnement Pixi installé, lancer :

```bash
pixi run python -m raw_to_processed thermal-configure /chemin/vers/MOLO-data-light
```

L'assistant demande successivement :
1. un identifiant de génération matérielle ;
2. le chemin relatif du fichier de calibration dans le dépôt de données ;
3. le format (`records` pour le `RECORDS.csv` brut, ou `table` pour un CSV/Excel déjà normalisé) ;
4. les colonnes représentant la température externe, la température interne et la pression (pour `records`) ;
5. le nom de la série de température externe utilisée sur le terrain (par défaut `TempBed`) ;
6. le pas d'échantillonnage, si le fichier ne permet pas de le déduire de sa colonne `Date` ;
7. les Sampling Points qui utilisent cette génération.

Avant d'écrire, l'assistant lit le fichier avec le même lecteur que le pipeline et tente réellement d'ajuster les deux modèles.
Il affiche les paramètres obtenus et un aperçu du JSON. **Le fichier `lab/lab.json` n'est modifié qu'après confirmation explicite.**
En cas d'annulation ou d'erreur de lecture/ajustement, aucune modification n'est enregistrée.

Après la déclaration, lancer le pipeline normalement, par exemple :

```bash
pixi run python -m raw_to_processed run /chemin/vers/MOLO-data-light Point034
```

L'assistant ne valide pas à lui seul la pertinence scientifique de l'expérience : la génération matérielle, les rôles des
capteurs, les unités et les conditions de référence doivent toujours être confirmés par la personne responsable de la calibration.
