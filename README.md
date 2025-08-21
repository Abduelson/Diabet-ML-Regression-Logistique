# Prédiction du Diabète — README

> **But du dépôt** : Ce document fait le lien entre la présentation non technique et le notebook Jupyter. Il décrit l’objectif métier, le jeu de données, la méthodologie de modélisation et les résultats sans entrer dans les détails d’implémentation.

---

## 1) Aperçu

Le projet vise à **prédire la probabilité qu’un patient soit diabétique** à partir de mesures simples (ex. Stroke, CholCheck, MentHlth, 
Smoker, Sex, NoDocbcCost, AnyHealthcare, Fruits, Veggies). L’outil a pour vocation de **soutenir le dépistage précoce** en orientant plus rapidement les patients à risque vers des examens complémentaires.

**Valeur métier :**

* Prioriser les patients à risque (triage).
* Réduire le nombre de diagnostics tardifs et les coûts associés.
* Aider à la planification des campagnes de prévention.

---

## 2) Compréhension des entreprises et des données

### Parties prenantes

* **Professionnels de santé (médecins, infirmier·ère·s)** : outil d’aide au dépistage.
* **Gestionnaires d’établissements / santé publique** : allocation de ressources et prévention ciblée.
* **Patients** : orientation et sensibilisation.

### Jeu de données

* **Cible** : `Diabetes_binary` (0 = non diabétique, 1 = diabétique).
* **Variables disponibles (exemples)** : `HighBP`, `HighChol`, `BMI`, `PhysActivity`, `HvyAlcoholConsump`, `GenHlth`, `PhysHlth`, `DiffWalk`, `Age`, `Education`, `Income`, … (22 variables au total).
* **Nature des variables** :

  * Binaires (0/1) : habitudes/conditions (ex. `HighBP`).
  * Ordinales : `GenHlth` (santé perçue), `Education`, `Income`, `Age` (par classes).
  * Continues : `BMI`, `PhysHlth` (jours de mauvaise santé).

### Qualité & éthique

* **Valeurs extrêmes** plausibles (ex. IMC élevé) conservées pour ne pas perdre d’information clinique.
* **Aucun déséquilibre majeur** des classes observé (0/1 quasi équilibrés).
* L’outil **ne remplace pas** un diagnostic médical ; il sert d’aide à la décision.

---

## 3) Modèle (approche itérative)

### Modèle de base (interprétable)

* **Régression Logistique** (baseline) entraînée avec :

  * Normalisation des variables continues (`BMI`, `PhysHlth`).
  * Variables binaires/ordinales conservées telles quelles.
  * **Division stratifiée** train/test pour conserver les proportions de classes.

### Modèle amélioré (tuning)

* **Arbre de Décision** avec réglage d’hyperparamètres (ex. `max_depth`, `min_samples_split`) via validation croisée (score ciblant le **rappel** de la classe diabétique).

### Pourquoi deux familles de modèles ?

* **Régression Logistique** : baseline robuste et lisible, bon compromis global.
* **Arbre de Décision** : règles simples, souvent meilleur **rappel** après réglage — utile en dépistage.

---

## 4) Évaluation (métriques de classification)

> Les métriques sont présentées en langage métier.

* **Accuracy (exactitude)** : % de bonnes prédictions globales.
* **Précision (precision)** : parmi les patients prédits « diabétiques », part réellement diabétique (limiter les fausses alertes).
* **Rappel (recall)** : parmi les patients réellement diabétiques, part correctement détectée (**prioritaire en dépistage** pour éviter d’en rater).
* **F1-score** : compromis précision/rappel.
* **AUC-ROC** : capacité globale à distinguer « diabétique » vs « non diabétique » pour tous les seuils possibles.

### Résultats synthétiques (jeu de test)

* **Régression Logistique** :

  * Accuracy ≈ **0,74**, Rappel (classe 1) ≈ **0,76**, AUC ≈ **0,82**.
* **Arbre de Décision (optimisé)** :

  * Accuracy ≈ **0,71**, Rappel (classe 1) ≈ **0,84**, AUC ≈ **0,78**.

**Lecture métier :** L’arbre détecte davantage de patients diabétiques (**rappel plus élevé**), au prix d’un peu plus de fausses alertes (precision plus faible). La régression logistique offre de meilleures performances globales (AUC plus élevée) mais peut rater davantage de cas que l’arbre.

---

## 5) Conclusion (modèle final)

* **Objectif métier** : en dépistage, **ne pas rater** de patients diabétiques est crucial.
* **Choix retenu** : **Arbre de Décision optimisé (profondeur ≈ 3)** pour son **rappel élevé (\~0,84)** sur la classe diabétique, donc moins de faux négatifs.
* **Trade-off assumé** : davantage de fausses alertes acceptées, qui seront filtrées ensuite par un examen médical.

> Alternative envisagée : conserver la Régression Logistique mais **abaisser le seuil de décision** (ex. 0,40) pour augmenter le rappel. Cette piste fera l’objet des prochaines itérations.

---

## 6) Prochaines étapes

1. **Seuil de décision & coûts** : calibrer le seuil en fonction d’un **coût métier** (coût d’un faux négatif > faux positif en dépistage).
2. **Ensembles & modèles avancés** : Random Forest, Gradient Boosting, calibration des probabilités.
3. **Robustesse & équité** : tests par sous-population, détection de biais.
4. **Déploiement** : interface simple (tableur / formulaire) pour un usage clinique ; journalisation des prédictions.

---

## 7) Organisation du dépôt (suggestion)

```
├─ data/                     # jeux de données (version anonymisée / échantillon)
├─ notebooks/
│  └─ 01_diabetes_modeling.ipynb  # EDA, préparation, modèles, évaluation
├─ reports/
│  └─ presentation.pdf       # diaporama non technique
├─ src/                      # fonctions utilitaires (prétraitement, métriques)
└─ README.md                 # ce document
```

**Reproductibilité (hauts niveaux)** :

* Ouvrir le notebook principal dans `notebooks/` et exécuter séquentiellement.
* Le code utilise un **split stratifié** et une **validation croisée** ; aucune dépendance exotique.


