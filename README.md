# MOLONARI-DATA

Dépôt de données du projet MOLONARI. Il sépare les métadonnées du laboratoire, les mesures brutes conservées sans modification et les données traitées produites par `molonari-pipeline`.

## État de la migration

Ce dépôt contient les premiers jeux importés depuis les archives historiques : Point034 et Point048. Les fichiers bruts de Point048 sont copiés sans nettoyage ni réécriture. Les sorties `processed_data` restent produites exclusivement par `molonari-pipeline`.

## Organisation

```text
MOLONARI-DATA/
├── lab/
│   └── lab.json
├── sampling_points/
│   └── Point034/
│       └── metadata.json
├── raw_data/
│   └── Point034/
│       └── v1_hobo/
├── processed_data/
│   └── Point034/
└── schemas/
    ├── lab.schema.json
    └── sampling_point.schema.json
```

### Rôle des répertoires

- `lab/lab.json` : paramètres partagés du laboratoire, calibrations et association entre appareils et Sampling Points.
- `sampling_points/<ID>/metadata.json` : description physique et historique du point (shaft, capteurs, profondeurs, dates de référence).
- `raw_data/<ID>/` : fichiers d'origine. Ils sont immuables après import ; `ingest.json` enregistre leurs empreintes.
- `processed_data/<ID>/` : sorties calculées par le pipeline (`data/measures.csv` et `manifest.json`).

Le code du pipeline actuel lit `lab/lab.json`, `raw_data/` et écrit `processed_data/`. Les métadonnées détaillées de `sampling_points/` constituent la couche descriptive du dépôt ; elles ne remplacent pas encore les champs du `lab.json` dont le pipeline a besoin.

## Point034 : provenance et paramètres

La fiche `sampling_points/Point034/metadata.json` reprend les champs connus dans l'ancien fichier `SamplingPoints/Point034/Point034_info.csv` et le fichier de shaft `Laboratory/shafts/S007.csv`.

La calibration `P508_T001` reprend :
- `intercept`, `dudh`, `dudt` du fichier historique `Laboratory/pressure_sensors/P508.csv` ;
- `beta = 3834 K`, `v_ref = 3.3 V` et `t0_k = 298.15 K` tels qu'attendus par le modèle de thermistance du pipeline.

Le fichier HOBO indique `GMT+01:00`. Le paramètre `utc_offset_hours = 1.0` fait convertir les horodatages en UTC naïf dans le pipeline. Le format de date des exports Point034 est `%m/%d/%y %I:%M:%S %p`.

**À vérifier lors du premier traitement :** comparer les mesures traitées avec un résultat de référence Molonaviz/pyheatmy. Les valeurs historiques sont reprises pour amorcer la migration, pas pour certifier à elles seules le résultat scientifique.

## Import des fichiers HOBO Point034

Depuis une copie locale de ce dépôt, placer les deux fichiers historiques dans un dossier temporaire, en conservant leurs octets et en utilisant les noms attendus par le pipeline :

```text
P_measures.csv
T_measures.csv
```

Les fichiers sources historiques se trouvent dans le dépôt `flipoyo/MOLONARI-data` :
- `SamplingPoints/Point034/point034_P_measures.csv`
- `SamplingPoints/Point034/point034_T_measures.csv`

Copier/renommer les fichiers (ne pas les ouvrir puis les réenregistrer dans un tableur). Puis, depuis le dépôt `molonari-pipeline`, lancer :

```bash
python -m raw_to_processed import-v1 /chemin/vers/MOLONARI-DATA Point034 /chemin/vers/dossier_hobo
python -m raw_to_processed run /chemin/vers/MOLONARI-DATA Point034
python -m raw_to_processed verify /chemin/vers/MOLONARI-DATA
```

Le premier import crée `raw_data/Point034/v1_hobo/` et son `ingest.json`. Le traitement produit `processed_data/Point034/data/measures.csv` et `manifest.json`.

> Le pipeline actuel attend `P_measures.csv` et `T_measures.csv` exactement. Les noms historiques `point034_P_measures.csv` et `point034_T_measures.csv` doivent donc être renommés au moment de la copie, sans modifier leur contenu.

## Point048 : CSV historiques de 2018

Les fichiers bruts proviennent de `flipoyo/MOLONARI1D/dataAnalysis/raw_data/point48_14_04_18/` :
- `raw_data/Point048/legacy_csv/pressure.csv` : tension différentielle déjà exportée en volts et température du flux en °C ;
- `raw_data/Point048/legacy_csv/temperature.csv` : quatre voies de température déjà exprimées en °C ;
- `raw_data/Point048/legacy_metadata/` : fiche historique et géométrie originales ;
- `raw_data/laboratory/pressure_sensors/P606.csv` : calibration de pression d'origine.

Chaque CSV contient 19 200 lignes, avec des horodatages identiques entre les deux séries, du 14 avril au 31 octobre 2018. Le lecteur dédié normalise les noms de colonnes vers l'état intermédiaire du pipeline ; il ne reconvertit pas les températures. Le calcul de pression réutilise les coefficients P606 et le même opérateur que les autres sources.

Le fuseau horaire des horodatages n'est pas documenté dans l'archive. Le paramètre `utc_offset_hours: null` conserve donc les dates locales naïves sans leur appliquer de décalage ni les présenter comme des dates UTC certifiées. La géométrie indique les profondeurs de température 10, 20, 30 et 40 cm, mais ne renseigne pas la profondeur du lit ni `delta_h`.

### Lancer la conversion Point048

Depuis le dépôt `molonari-pipeline`, avec Python/pixi installé :

```bash
python -m raw_to_processed register-legacy-csv /chemin/vers/MOLONARI-DATA Point048
python -m raw_to_processed run /chemin/vers/MOLONARI-DATA Point048 --source legacy_csv
python -m raw_to_processed verify /chemin/vers/MOLONARI-DATA Point048
```

La première commande valide les deux CSV et crée `raw_data/Point048/ingest.json` (empreintes de tous les fichiers bruts du point). La deuxième écrit `processed_data/Point048/data/measures.csv` et `manifest.json`. La troisième vérifie que les sources n'ont pas été altérées.

## Principes de gestion

1. Ne jamais modifier les fichiers bruts après leur import.
2. Ne jamais inventer une calibration ou une métadonnée manquante : laisser le champ absent ou documenter explicitement l'incertitude.
3. Toute transformation doit être reproductible à partir des données brutes, de `lab.json` et d'une version identifiée du code.
4. Ne pas versionner de secrets, identifiants MQTT privés ou mots de passe.
5. Avant de stocker de gros volumes dans Git, décider si les données doivent rester dans Git ou être placées dans un stockage externe adressé par empreinte.

## Dépendance avec les autres dépôts

- `molonari-pipeline` : lit ce dépôt comme `DATA_ROOT`, convertit les données et écrit les sorties.
- `MOLONARI1D` : référence historique pour le modèle conceptuel des Sampling Points et les anciens traitements.
- `ComplexGitSync` : organise les dépôts et calcule les empreintes de l'Operating Space. Il ne remplace ni les empreintes d'ingestion ni les manifestes métier du pipeline.
