# Calibrations instrumentales

Les fichiers de calibration instrumentale importés par le pipeline sont archivés dans ce dossier, sous
`instrumental/<ID>.csv`. Ils sont copiés tels quels depuis le fichier source.

Le pipeline enregistre également les paramètres numériques utilisés dans `lab/lab.json`, sous
`calibrations/<ID>`, ainsi que le SHA-256 du CSV source dans `calibration_sources/<ID>`.

Format attendu : lignes `Intercept`, `dU/dH` et `dU/dT`, chacune suivie de sa valeur dans la deuxième colonne.
Les séparateurs virgule et point-virgule sont acceptés.

Cette calibration instrumentale ne doit pas être confondue avec les tables de calibration thermique,
archivées séparément dans `calibration/thermal/`.
