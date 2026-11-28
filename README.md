# NOVALAINE : Customer 360 et segmentation RFM

Projet de pairagogie n°1 du module Data Processing (M2 Data & IA, INSEEC 2026-2027), groupe 3 : Nour El Manar KHEZZAR, Amira HAMDI, Pierre Quentin NGANDJUI TIAKO et Hayat AKBI.

Le pipeline transforme les quatre exports bruts de NOVALAINE (clients, commandes, lignes de commande, produits) en une vue client unique, puis en segments RFM et en un jeu de données pour prédire le rachat à 6 mois. Soutenance le 10 décembre 2026.

## Contenu du dépôt

```
Projet_NOVALAINE/
├── NOVALAINE_pipeline.ipynb   notebook principal, une section par phase du brief
├── requirements.txt           bibliothèques Python à installer
├── sources_de_donnees/        les 4 CSV bruts (ne jamais les modifier)
├── sorties/
│   ├── donnees/               tables livrables (clients_360.csv, dataset_rachat_6m.csv…)
│   ├── tableaux/              tableaux d'audit, de réconciliation, de contrôles
│   └── figures/               graphes .png pour le rapport et la soutenance
├── docs/                      brief, fiche de cadrage, revues de code
└── Archives/                  anciennes versions de notebooks
```

## Installer le projet sur sa machine

Comptez une quinzaine de minutes la première fois. Les commandes sont données pour Windows (PowerShell), avec l'équivalent macOS juste en dessous quand il diffère.

### 1. Prérequis

- Python 3.11 ou plus récent. Vérifiez avec `py --version` sous Windows ou `python3 --version` sous macOS. Sinon, installez-le depuis python.org en cochant « Add python.exe to PATH ».
- GitHub Desktop (le plus simple) ou Git.
- VS Code avec les extensions Python et Jupyter de Microsoft. Jupyter dans le navigateur marche aussi.
- Un compte GitHub ajouté comme collaborateur du dépôt par Amira, sinon vous ne pourrez pas pousser vos changements.

### 2. Cloner le dépôt

Dans GitHub Desktop : File, puis Clone repository, onglet URL, et collez `https://github.com/Amirahamdi-201/Projet_NOVALAINE.git`.

En ligne de commande :

```
git clone https://github.com/Amirahamdi-201/Projet_NOVALAINE.git
```

Évitez si possible un dossier synchronisé par OneDrive, Dropbox ou iCloud. La synchronisation peut verrouiller les fichiers de git et créer des copies en conflit. `C:\Users\<vous>\Projets\` convient très bien.

### 3. Créer l'environnement virtuel

Ouvrez un terminal dans le dossier `Projet_NOVALAINE` (dans VS Code : Terminal, puis New Terminal), puis :

```
py -m venv .venv
.\.venv\Scripts\Activate.ps1
```

Sous macOS :

```
python3 -m venv .venv
source .venv/bin/activate
```

Le nom de l'environnement, `(.venv)`, doit apparaître au début de la ligne du terminal. Le dossier `.venv/` est exclu de git par le `.gitignore` : chacun a le sien.

Si PowerShell refuse d'exécuter `Activate.ps1`, lancez une fois `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`, répondez « O », puis recommencez.

### 4. Installer les bibliothèques

Environnement activé :

```
python -m pip install --upgrade pip
pip install -r requirements.txt
```

Pour vérifier :

```
python -c "import pandas, numpy, matplotlib, seaborn, sklearn; print('pandas', pandas.__version__)"
```

La version de pandas affichée doit commencer par 3. Tout le groupe doit travailler avec la même version majeure, sinon le même code peut donner des résultats différents d'une machine à l'autre.

Avec Anaconda, l'équivalent est :

```
conda create -n novalaine python=3.12
conda activate novalaine
pip install -r requirements.txt
```

### 5. Ouvrir le notebook et choisir le noyau

Ouvrez `NOVALAINE_pipeline.ipynb` dans VS Code. En haut à droite, cliquez sur « Select Kernel », puis sur l'environnement `.venv` (ou `novalaine` avec conda). Lancez ensuite « Restart », puis « Run All ». La cellule de chargement doit afficher :

```
Chargement   clients      4160 ->    4160
Chargement   commandes    9583 ->    9583
Chargement   lignes      16868 ->   16868
Chargement   produits      120 ->     120
```

Si c'est le cas, l'installation est terminée.

## Règles de travail du groupe

- Les CSV de `sources_de_donnees/` ne sont jamais modifiés, ni à la main ni par le code. Toute sortie va dans `sorties/`, via les variables `DOSSIER_DONNEES`, `DOSSIER_TABLEAUX` et `DOSSIER_FIGURES` de la cellule de paramètres.
- Les chemins restent relatifs à la racine du dépôt. Aucun chemin du type `C:\Users\...`, et jamais `today()` : la date de référence est le 30/09/2026.
- On préfixe les fichiers de sortie par leur phase : `p1_audit.csv`, `p6_ca_par_segment.png`.
- Avant de commencer à travailler, on récupère les derniers changements (Fetch puis Pull dans GitHub Desktop).
- Une branche par tâche, puis une pull request vers `main`. On ne pousse pas directement sur `main`.
- Un notebook se fusionne mal quand deux personnes le modifient en même temps. On se prévient dans le groupe avant de toucher `NOVALAINE_pipeline.ipynb`, et on fait « Clear All Outputs » avant de commiter, ce qui réduit beaucoup les conflits.

## Problèmes fréquents

`py` ou `python` n'est pas reconnu : Python n'est pas dans le PATH. Réinstallez-le en cochant l'option, ou utilisez le chemin complet de `python.exe`.

GitHub Desktop affiche les 5 CSV comme modifiés alors que personne n'y a touché : ce sont des différences de fins de ligne entre Windows et macOS. Ne les commitez pas. Pour régler le problème une fois pour toutes, lancez `git config core.autocrlf true` sous Windows, ou `git config core.autocrlf input` sous macOS.

`ModuleNotFoundError` dans le notebook : le noyau sélectionné n'est pas celui de `.venv`. Revenez à l'étape 5.

`FileNotFoundError` sur un CSV : le notebook ne tourne pas depuis la racine du dépôt, souvent parce qu'il a été copié ou déplacé dans un sous-dossier. Il doit rester à côté de `sources_de_donnees/`. Ouvrez aussi le dossier `Projet_NOVALAINE` entier dans VS Code (File, puis Open Folder), pas seulement le fichier.
