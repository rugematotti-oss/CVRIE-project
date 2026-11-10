# CVRIE — CVRIE V1

Projet Epitech (spécialisation Data/IA) : construction d'une IA médicale (CVRIE V1) capable d'analyser des témoignages patients non labellisés et de classifier des images médicales.

## Description

CVRIE est un projet en deux volets de Machine Learning :

* **Supervised Learning** : classification d'un dataset d'images médicales de scanners CT, pour prédire si un scanner a été réalisé avec ou sans injection de produit de contraste.
* **Unsupervised Learning** : clustering d'un dataset de témoignages patients non labellisés (fourni par l'école).

L'objectif n'est pas seulement d'obtenir un bon résultat final, mais de documenter et de justifier toute la démarche : hypothèses, expériences, résultats (bons ou mauvais), analyses, corrections. Un résultat médiocre bien analysé compte autant qu'un bon résultat.

---

## Équipe
Mohamed Naim & Rugema Nyiringabo Epiphane

---

## Objectifs

* Choisir, analyser et pré-traiter un dataset d'images médicales adapté à une classification
* Entraîner et comparer plusieurs modèles supervisés, justifier l'approche retenue
* Pré-traiter et regrouper les témoignages patients en clusters cohérents avec la réalité clinique
* Documenter une véritable démarche expérimentale (hypothèse → expérience → résultat → analyse)
* Être capable de défendre chaque choix technique, pas seulement présenter du code

---

## Structure du projet

```bash
CVRIE/
├── .gitignore
├── README.md
├── requirements.txt
├── supervised/
│ ├── notebook.ipynb
│ └── data/
├── unsupervised/
│ ├── notebook.ipynb
│ └── data/
│ └── Student_Dataset.csv
├── docs/
│ ├── kickoff-analysis.md # analyse de cadrage du sujet
│ └── dataset-candidates.md # comparatif des datasets supervised candidats
└── bonus/
```

---

## Datasets

### Unsupervised (fourni)

`unsupervised/data/Student_Dataset.csv` — témoignages de patients non labellisés, 1011 lignes. Une colonne couleur cachée (`0x000000`) sera révélée en soutenance pour vérifier la cohérence visuelle des clusters. **Interdiction d'ajouter ses propres labels pour entraîner le modèle.**

### Supervised (choisi)

Dataset de scanners CT avec des colonnes cliniques réelles, dont l'âge du patient et la variable cible `Contrast` (présence ou non de produit de contraste). L'objectif est de prédire si un scanner a été réalisé avec injection de contraste à partir de features extraites des images et de métadonnées cliniques. Ce choix permet une validation croisée avec des connaissances médicales réelles, notamment le lien entre âge, fonction rénale, et contre-indication au contraste.

---

## Installation

### 1. Cloner le repository

```bash
git clone le repos
cd projet/
```

### 2. Créer un environnement virtuel

```bash
python3 -m venv venv
source venv/bin/activate
```

### 3. Installer les dépendances

```bash
pip install -r requirements.txt
```

### 4. Lancer JupyterLab

```bash
jupyter lab
```

---

## Structure des notebooks

Nos deux notebooks (`supervised/notebook.ipynb` et `unsupervised/notebook.ipynb`) suivent la même trame, adaptée à leur contexte (classification pour l'un, clustering pour l'autre) :

Introduction → Objectif → Compréhension du dataset → Exploration/EDA → Problèmes identifiés → Preprocessing → Baseline → Première expérience → Résultats → Analyse → Nouvelle hypothèse → Expérience suivante → Comparaison → Analyse des erreurs → Validation → Limites → Conclusion → Insights

---

## Workflow Git

* `main` : branche finale, ne reçoit que du travail relu et vérifié
* Une branche personnelle par membre pour le travail quotidien
* Commits petits et réguliers avec préfixes conventionnels : `feat`, `fix`, `docs`, `chore`, `refactor`, `test`
* Merge vers `main` uniquement après relecture croisée

---

## Livrables attendus

* [x] Un notebook par partie (supervised + unsupervised)
* [x] Code Python produisant le clustering (partie non supervisée)
* [ ] Dataset de la partie supervisée livré avec le code
* [x] Code Python entraînant un modèle et classant les données (partie supervisée)
* [x] Section preprocessing claire dans chaque notebook
* [x] Section comparaison de modèles claire dans chaque notebook
* [x] Section conclusions/insights claire dans chaque notebook
* [ ] Explication claire de la loss function dans chaque notebook
* [x] Graphiques soignés et expliqués
* [ ] Fichiers bonus dans un dossier `bonus/` séparé

---

## État d'avancement

* [x] Cadrage du sujet et des compétences
* [x] Structure du repo, `.gitignore`, `requirements.txt`
* [x] Choix du dataset supervised
* [x] Répartition définitive des rôles (qui pilote quel notebook)
* [x] Exploration et preprocessing des deux datasets
* [x] Entraînement et comparaison des modèles
* [x] Validation critique des résultats
* [ ] Préparation de la soutenance