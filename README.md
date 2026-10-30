# Projet 2 — Détecter des attaques réseau avec le Machine Learning

**Nom / Prénom : GAYE Maimouna**

## Lancer le projet
```bash
pip install -r requirements.txt
python src/train.py      # compare les modèles et enregistre models/model.joblib
python src/predict.py    # petit test de la fonction predict
```

## Données et prétraitement
3 000 flux, 28 % d'attaques (classes déséquilibrées : on suit le F1 de la classe « attaque »).
Seule `duration` a des valeurs manquantes (90). Le `Pipeline` impute la médiane, met à l'échelle
les variables numériques (pour la régression logistique) et encode `protocol` en one-hot.
Les modèles utilisent `class_weight="balanced"`. Validation : 20 % des données (stratifié),
plus une validation croisée à 5 plis pour choisir de façon plus stable.

## Résultats

**Nom / Prénom :** GAYE Maimouna

Jeu de validation (600 flux, dont 167 attaques) :

| Modèle | F1 (attaque) | Rappel (attaque) | Précision (attaque) | F1 validation croisée (5 plis) |
|---|---|---|---|---|
| Régression logistique | 0,920 | 0,898 | 0,943 | 0,896 |
| Arbre de décision (profondeur 6) | 0,953 | 0,964 | 0,942 | 0,935 |
| **Forêt aléatoire (300 arbres)** | **0,982** | **0,976** | **0,988** | **0,960** |

**Modèle retenu et pourquoi :** la forêt aléatoire, qui a le meilleur F1 sur la validation
et en validation croisée. Elle capte mieux les relations non linéaires (par exemple un grand nombre de
connexions vers la même machine combiné à un `syn_rate` élevé) que la régression logistique, et elle est
plus stable qu'un arbre seul. Elle a été ré-entraînée sur les 3 000 flux et enregistrée dans `models/model.joblib`.

**Analyse des erreurs (faux négatifs / faux positifs) :**
- Sur la validation, la forêt a raté **4 attaques sur 167** (faux négatifs) et déclenché **2 fausses alertes** (faux positifs). Matrice : VN = 431, FP = 2, FN = 4, VP = 163.
- En validation croisée sur les 3 000 flux : 46 attaques ratées sur 833 (≈ 5,5 %) et 20 fausses alertes.
- Les faux négatifs sont des attaques discrètes : elles ressemblent à du trafic normal (peu de connexions vers la même machine, `syn_rate` faible) et visent surtout des ports courants (443, 80). Les faux positifs sont au contraire des flux normaux au comportement « bruyant » (beaucoup de connexions, `syn_rate` élevé).
- Un faux négatif est le pire cas en sécurité : une attaque réelle passe inaperçue, sans alerte ni blocage, et l'attaquant peut poursuivre son action. Un faux positif coûte seulement du temps d'analyse. On peut donc privilégier le **rappel** en abaissant le seuil de décision, au prix de quelques fausses alertes de plus.
