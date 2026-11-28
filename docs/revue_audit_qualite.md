# Revue de code croisée : notebook d'audit qualité

Notebook relu : `Notebook_Auditrice_Qualite_NOVALAINE.ipynb` (Hayat AKBI, auditrice qualité)
Relecteur : Pierre Quentin NGANDJUI TIAKO
Date : 28/11/2026
Grille utilisée : celle du brief (lisibilité, robustesse, hypothèses explicites, reproductibilité)

Salut Hayat,

J'ai relu ton notebook et j'ai recalculé chaque indicateur à part, directement sur les quatre CSV du dépôt. Bonne nouvelle d'abord : tous tes chiffres sont justes. Je retrouve les mêmes volumes (4 160 clients, 9 583 commandes, 16 868 lignes, 120 produits), les 86 commandes et 81 lignes en double exact, les 47 commandes sans client, les 116 à client inconnu, les 102 lignes à produit inconnu et les 279 écarts de réconciliation. La structure est claire, les données brutes restent intactes et la date de référence est bien fixée au 30/09/2026.

Mes remarques portent surtout sur la façon de lire certains chiffres et sur quelques contrôles qui manquent. Je les ai classées de la plus importante à la moins importante.

## 1. Reproductibilité

Le notebook ne trouve pas les données dans le dépôt. La fonction `trouver_dossier_donnees()` cherche dans `donnees/`, `.` et `/mnt/data`, alors que les CSV sont dans `sources_de_donnees/`. Il plante donc dès la première cellule. Il suffit d'ajouter `Path("sources_de_donnees")` en tête de la liste des candidats. On pourrait aussi retirer `/mnt/data`, qui ne sert que dans l'environnement de ChatGPT.

Pour les sorties, on s'est mis d'accord sur une arborescence commune, déjà créée dans le dépôt et définie dans la cellule de paramètres du notebook `NOVALAINE_pipeline.ipynb` :

| Dossier | Contenu | Variable |
|---|---|---|
| `sorties/donnees/` | tables livrables (`clients_360.csv`, `dataset_rachat_6m.csv`, `table_correspondance_clients.csv`) | `DOSSIER_DONNEES` |
| `sorties/tableaux/` | tableaux en .csv : audit, réconciliation du CA, contrôles, volumétries | `DOSSIER_TABLEAUX` |
| `sorties/figures/` | graphes en .png pour le rapport et la soutenance | `DOSSIER_FIGURES` |

Tes huit exports actuels (`01_volumetrie.csv` à `08_controles_automatiques.csv`) vont donc dans `sorties/tableaux/` et non plus dans `sorties_audit/`. On préfixe chaque fichier par sa phase, par exemple `p1_volumetrie.csv` ou `p1_audit.csv`, pour savoir d'où vient chaque tableau. Les graphes s'enregistrent avec `plt.savefig(DOSSIER_FIGURES / "p1_valeurs_manquantes.png", dpi=150, bbox_inches="tight")`. Tous les chemins restent relatifs à la racine du dépôt.

Les chemins `/mnt/data` et `/home/oai` dans les sorties montrent que le notebook a été exécuté dans ChatGPT. Ce n'est pas un problème, mais il faut le noter dans le journal IA avec les prompts utilisés.

## 2. Montants multipliés par 100 (gravité forte)

18 commandes ont un montant exactement 100 fois supérieur au total recalculé à partir des lignes. Par exemple, 32 160 € au lieu de 321,60 €. Ensemble, elles déclarent 312 227 € pour 3 122 € réels. Elles apparaissent dans tes 279 écarts mais ne sont pas isolées.

Conséquence : sur les commandes livrées, elles ajoutent 238 311 € au « CA net provisoire » de 1 573 172 €, soit environ 15 % de trop. C'est le point que le jury vérifiera en premier, puisque la réconciliation du CA fait partie du critère noté sur 7.

Contrôle proposé : signaler les commandes où `montant_ttc / montant_theorique` vaut 100 (à 0,5 près).

## 3. Dates de naissance futures (gravité moyenne)

Sur les 102 dates futures du tableau de validité, 91 sont des erreurs de lecture. 189 dates sont au format `jj-mm-aa`, et `pd.to_datetime(format="mixed")` lit `21-07-60` comme 2060 au lieu de 1960. Il n'y a en réalité que 11 dates futures, toutes au 12/05/2031.

À l'inverse, 39 dates impossibles ne sont pas signalées : 24 au 01/01/1900 et 15 au 03/03/1850. Ce sont des valeurs bouchons, à traiter comme des valeurs manquantes.

Correction proposée : parser chaque format explicitement (`%Y-%m-%d`, `%d/%m/%Y`, `%d-%m-%y`). Pour le format à deux chiffres, retirer 100 ans quand l'année obtenue dépasse la date de référence. Puis ajouter un contrôle sur les naissances avant 1910.

## 4. Suppression des lignes en double exact (gravité moyenne)

Les 81 lignes strictement identiques appartiennent à 81 commandes différentes. Pour 74 d'entre elles, le montant de la commande ne colle que si on garde la ligne en double : ce sont donc de vrais articles achetés deux fois. Seules 3 commandes se réconcilient après suppression. La suppression crée à elle seule 73 des 279 écarts.

Le brief demande de supprimer les doublons exacts de commandes, pas de lignes. Je propose de garder les lignes et de documenter ce choix dans le journal de nettoyage, avec cette vérification à l'appui.

## 5. Montants négatifs et montants manquants

Les 28 montants négatifs ne sont pas des avoirs. Pour 27 d'entre eux, la valeur absolue est égale au montant recalculé, et 27 concernent des commandes livrées. C'est une inversion de signe, à corriger par une règle écrite.

La ligne « Montants de commande non numériques : 192 » compte en fait des valeurs vides. Le vrai problème de format est ailleurs : 375 montants sont écrits à la française (« 204,60 € »). Ta fonction `parse_montant` les convertit bien, mais l'audit ne les compte pas. Sur les 190 montants vides qui restent après dédoublonnage des commandes, 187 peuvent être reconstitués à partir des lignes.

## 6. Décomposition des 279 écarts

Pour la soutenance, voici d'où viennent les écarts :

| Cause | Commandes |
|---|---|
| Prix unitaire manquant sur au moins une ligne | 115 |
| Ligne en double supprimée (voir point 4) | 73 |
| Quantité nulle ou négative | 48 |
| Montant négatif | 27 |
| Montant multiplié par 100 | 18 |

Certaines commandes cumulent deux causes. Toutes les commandes en écart entrent dans au moins une catégorie, et il ne reste aucun écart inexpliqué.

## 7. Contrôles absents

Le brief demande la grille complète de la séance 1. Voici ce qui n'est pas encore mesuré :

- Prix à 0 € : 538 lignes. Tous les autres prix sont strictement égaux au prix catalogue, donc ceux-là peuvent être reconstitués. Ce sont eux qui expliquent les 112 commandes à 4,90 € (frais de port seuls).
- E-mails : 95 adresses invalides (« atgmail.com », adresse sans domaine), 212 vides. 76 adresses contiennent des majuscules et 72 des espaces en trop.
- Modalités à normaliser : `pays` (France écrit de cinq façons, 799 fiches hors forme standard), `canal_acquisition` (21 variantes pour 7 canaux), `opt_in_email` (11 modalités et 163 vides), `civilite` (8 variantes), `code_promo` (casse).
- Codes postaux : le zéro initial a disparu, par exemple 166 fiches à « 6000 » pour Nice.
- Cohérence temporelle : 43 commandes datées avant l'inscription du client.
- Intégrité dans l'autre sens : 94 commandes sans aucune ligne, 140 clients sans commande.

## 8. Les 8 identifiants de commande en double restants

Ce sont 4 commandes rattachées chacune à deux fiches de la même personne, par exemple Mathieu Moreau (C001898 et C004112) ou Emma Fournier (C003775 et C004071). Il ne faut pas les supprimer. Elles prouvent directement que ces fiches sont des doublons, et Amira pourra s'en servir pour le dédoublonnage.

## 9. Livrable de la phase 1

Le brief attend un tableau d'audit chiffré avec les colonnes anomalie, fichier, colonne, volume, pourcentage, gravité et traitement envisagé. Il manque encore, et la section 8 est restée « à compléter ». Tous les chiffres nécessaires sont dans cette revue.

Deux contrôles du bloc automatique (« volumétrie clients non nulle », « volumétrie commandes non nulle ») passeront toujours. On peut les remplacer par des contrôles plus utiles : montant ×100, naissance avant 1910, commande avant inscription.

## Ce qui est bien fait

- Les fichiers bruts ne sont jamais modifiés, tout se fait sur des copies.
- Les paramètres (date de référence, frais de port, tolérance) sont regroupés en tête.
- La prudence est la bonne : les cas douteux sont signalés au lieu d'être supprimés.
- La normalisation des statuts (accents et casse) est correcte : il ne reste bien que trois statuts.

Si tu veux, on regarde les corrections ensemble avant la séance de travail encadré.

Pierre Quentin
