# Modele manuel ou modele AutoML ? (TP05)

Mesures du 26/10/2026, poste local (Python 3.11, FLAML 2.7.0, MLflow 3.17.2).
Les deux modeles sont servis par `mlflow models serve` (alias `production`), 200 appels mesures apres 10 appels d'echauffement, chaque mesure lancee deux fois.

| Critere                    | Manuel (rf 500/16)                         | AutoML (FLAML 90 s)                                  |
|----------------------------|--------------------------------------------|------------------------------------------------------|
| Famille de modele          | foret aleatoire (RandomForestClassifier)   | xgboost (XGBClassifier, 322 arbres, max_leaves 4)    |
| F1 sur le jeu de test      | 0.8669                                     | 0.8561                                               |
| Temps de recherche machine | 1.1 s (un entrainement ; plusieurs runs au TP2) | 90.1 s (`duree_recherche_s`)                    |
| Temps humain (estimation)  | ~1 a 2 h (TP2 : runs, comparaison MLflow)  | ~10 min (ecrire et lancer le script)                 |
| Latence mediane (ms)       | 48.7 / 48.5                                | 8.1 / 8.8                                            |
| Latence p95 (ms)           | 85.0 / 67.6                                | 11.3 / 10.3                                          |
| Taille du fichier (Mo)     | 13.37                                      | 0.28                                                 |
| Explicabilite              | importance des variables (montant 0.48, anciennete 0.24, freq_24h 0.19, nuit 0.09) ; 500 arbres de profondeur 16, illisibles un par un | importance des variables (nuit 0.31, montant 0.29, freq_24h 0.23, anciennete 0.16) ; arbres tres courts (4 feuilles) ; reglages et journal de recherche traces dans MLflow |

## Decision (5 lignes) : quel modele mettre en production, et pourquoi ?

1. Nous mettons en production le modele AutoML (`fraude-automl`, version 1).
2. Il perd 0.011 de F1 (0.8561 contre 0.8669), un ecart faible sur 1 250 transactions de test.
3. En contrepartie, il repond environ 6 fois plus vite (mediane ~8 ms contre ~48 ms, p95 ~11 ms contre 68-85 ms), ce qui compte pour une decision prise pendant le paiement.
4. Il est 48 fois plus leger (0.28 Mo contre 13.37 Mo) : image Docker plus petite, demarrage plus rapide, deploiement mobile possible.
5. Le modele manuel reste disponible dans le registre (`fraude-manuel@production`) comme solution de repli ; les futures versions AutoML seront promues par l'alias `production`, avec retour arriere sans toucher au code du client.
