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
