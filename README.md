Ce dépôt contient le pipeline complet de traitement du signal, d'ingénierie des caractéristiques (*features*), d'entraînement et d'évaluation comparative de modèles de Machine Learning pour la détection binaire des états d'hypovigilance/somnolence (sur l'échelle de Karolinska - KSS).

Le projet s'appuie sur une acquisition multimodale combinant la spectroscopie dans le proche infrarouge fonctionnelle (**fNIRS** frontale), l'électrocardiographie (**ECG**) et la fréquence respiratoire (**RSP**).

---

## 📋 Architecture & Pipeline des Notebooks

Les 5 notebooks structurent le flux de données dans l'ordre chronologique suivant :
Raw Data (Oxysoft / SensorHub)
│
▼
[`Prétraitement & Visualisation des données.ipynb`] ──► Nettoyage (TDDR, Bandpass, SOS filters) & Contrôle qualité
│
▼
[`Création des Datasets.ipynb`] ──► Fenêtrage glissant multirésolution, extraction TSFEL/NeuroKit2
│
├────────────────────────────────────────┬────────────────────────────────────────┐
▼ ▼ ▼
[`Benchmark_ML_methods.ipynb`] [`1 MODEL PER PARTICIPANT - leave a kss out - KSS 4 FOLDS.ipynb`] [`visualisation résultats ML.ipynb`]
(GroupKFold, Normalisation Z-score/ (Modèles personnalisés OOF, (Graphiques comparatifs,
Calibration, Optimisation seuil MCC) Leave-one-KSS-out 4-folds) Top-Biomarqueurs, Δ AP)

### Description détaillée des notebooks

1. **`Prétraitement & Visualisation des données.ipynb` (Prétraitement & Visualisation brute)** :
   * Synchronisation temporelle multi-sources (*SensorHub* et *Oxysoft*).
   * Filtrage fNIRS : correction des artefacts de mouvement par TDDR (*Temporal Derivative Distribution Repair*) et filtrage passe-bande $[0.01 - 0.5\text{ Hz}]$.
   * Filtrage ECG/RSP : filtres numériques IIR (structure SOS) et extraction de signaux d'onde de pouls et de respiration.
   * Analyse préliminaire de corrélation inter-canaux et variations hémodynamiques par phase.

2. **`Creation des Datasets.ipynb` (Fenêtrage & Feature Extraction)** :
   * Fenêtrage glissant multi-durées (Courte: 20s/60s, Moyenne: 30s/120s, Longue: 60s/180s).
   * Extraction massive de caractéristiques via `TSFEL` (fNIRS) et `NeuroKit2` (ECG, HRV temporel/fréquentiel/non-linéaire, RSP, RSA).
   * Gestion de la qualité du signal (seuil de couverture minimal et détection des sauts temporels).
   * Génération de la base de données de calibration ($1\text{ min}$ repos $+ 5\text{ min}$ PVT initiale).

3. **`Benchmark_ML_methods.ipynb` (Benchmark Global & Modèles de Population)** :
   * Validation croisée `GroupKFold` par participant ($N=5$) pour éliminer le surapprentissage inter-sujet (*data leakage*).
   * Comparaison des normalisations Z-score (*par participant* vs *sur baseline de calibration initiale*).
   * Réduction dimensionnelle supervisée/non-supervisée fNIRS (décorrélation greedy par information mutuelle, agrégations globale/hémisphérique).
   * Boucle imbriquée (*Nested CV*) pour l'optimisation non biaisée du seuil de décision clinique (MCC, $F_2$, Youden, etc.).

4. **`1 MODEL PER PARTICIPANT - leave a kss out - KSS 4 FOLDS.ipynb` (Benchmark Modèles Personnalisés)** :
   * Validation croisée intra-sujet de type *Leave-One-KSS-Out* (4 folds).
   * Évaluation de la transférabilité des modèles spécifiques à un individu entre différents épisodes temporels.

5. **`visualisation résultats ML.ipynb` (Synthèse & Biomarqueurs Robustes)** :
   * Comparaison comparative de l'apport des modalités ($\Delta\text{AP}$ vs fNIRS seule).
   * Évaluation de la robustesse face à la taille de fenêtre temporelle.
   * Extraction et identification des biomarqueurs physiologiques les plus stables et discriminants (sélectionnés sur 5/5 folds).

---

## ⚙️ Définition du Problème & Labellisation

* **Échelle KSS (*Karolinska Sleepiness Scale*)** :
  * **Classe 0 (Alerte / Vigilant)** : $KSS \le 5$
  * **Classe 1 (Hypovigilance / Somnolent)** : $KSS \ge 7$
  * *Note : Les états transitoires ($KSS = 6$) sont exclus pour garantir une frontière de décision nette.*

---

## 🛠️ Modèles Implémentés

* **Modèles linéaires & SVM** : Régression Logistique (L2 & ElasticNet), Linear SVC, SVM à noyau RBF.
* **Ensembles d'arbres & Boosting** : Random Forest, Extra Trees, HistGradientBoosting, XGBoost, CatBoost.

---

## 📦 Installation & Dépendances

```bash
git clone .....
cd Hypovigilance_Detection_ML_Neurostim_ULaval
pip install -r requirements.txt