<p align="center">
  <a href="../README.md">English</a> ·
  <a href="README.zh-CN.md">简体中文</a> ·
  <a href="README.es.md">Español</a> ·
  <b>Français</b> ·
  <a href="README.de.md">Deutsch</a> ·
  <a href="README.ja.md">日本語</a> ·
  <a href="README.ko.md">한국어</a> ·
  <a href="README.pt-BR.md">Português</a> ·
  <a href="README.ru.md">Русский</a>
</p>

# ReproRun

> Donnez-lui un article. Il vous dit si les résultats se reproduisent réellement.

**ReproRun** est une **compétence d'agent IA** portable — utilisable par n'importe quel assistant de codage IA compatible — qui automatise le parcours pénible entre *« un article affirme X »* et *« X tourne réellement sur ma machine »*. Les articles livrent de beaux chiffres ; leur reproduction meurt généralement dans du code cassé, des environnements impossibles à construire et des dérives de versions. ReproRun prend en charge l'ensemble du pipeline de bout en bout :

**lire l'article → trouver le code et les données → construire l'environnement → test de fumée → exécution complète → comparer les chiffres mesurés aux affirmations de l'article.**

---

## ✨ Conçu pour s'adapter — sur toutes les plateformes

ReproRun est conçu pour **s'adapter à tout ce qu'on lui confie**, au lieu de présupposer une configuration figée :

- **Multi-OS** — fonctionne sous Windows, macOS et Linux
- **Multi-langage** — pipeline complet pour Python ; chemin minimal pour R / MATLAB / Julia
- **Multi-domaine** — biologie unicellulaire, ML d'images, et plus encore, sans recâblage par domaine
- **Multi-mode** — *mode outil* (`pip install` + écriture d'un script appelant) ou *mode expérience* (cloner le dépôt et exécuter ses scripts) — sélectionné automatiquement pour chaque article

---

## 🚀 Ce qu'il fait

- **Trouver un article à partir de son seul titre** — recherche web automatique du PDF et du dépôt de code
- **Auto-diagnostic de la décrépitude de l'environnement** — conflits d'ABI numpy, changements d'API torchvision, méthodes pandas obsolètes… détectés et corrigés automatiquement
- **Pas de paramètres à deviner** — inspecte les signatures de fonctions après installation
- **Boucle d'auto-réparation des dépendances sur 5 tours** — classer l'erreur → correctif ciblé → re-vérifier, jusqu'à 5 tours
- **Isolation des articles** — chaque article reçoit son propre espace de noms de sortie

---

## 🏗️ Architecture

Un orchestrateur (`SKILL.md`) pilote 6 agents spécialisés :

| Agent | Rôle |
|-------|------|
| A · paper-reader | Extraire les affirmations numériques à reproduire |
| B · resource-finder | Localiser le dépôt de code et les jeux de données |
| C · environment-builder | Construire et réparer l'environnement d'exécution (le plus complexe) |
| D · smoke-tester | Test de fumée rapide — confirmer que ça tourne |
| E · full-runner | Exécution complète de la reproduction |
| F · result-comparator | Comparer mesuré vs. affirmé, point par point |

---

## ✅ Validation

ReproRun a été exécuté de bout en bout sur de vrais articles, dans plusieurs domaines. Il ne se contente pas d'apposer un tampon « succès » — pour chaque article il renvoie un **verdict honnête** : chiffres reproduits, pipeline reproduit, ou *impossible à reproduire en l'état* — toujours avec la cause racine.

| Article | Domaine | Résultat |
|-------|--------|--------|
| **UMAP** (McInnes 2018, JOSS) | réduction de dimension | ✅ **Chiffres reproduits** — 11/14 métriques de précision k-NN concordent à ±0,01 près ; MNIST et Fashion-MNIST confirmés à 3 décimales |
| **scVelo** (Bergen 2020, Nat Biotech) | unicellulaire | ✅ **Pipeline reproduit** — a détecté un bug d'ABI numpy 2.x causant 100 % de NaN, corrigé en rétrogradant vers 1.26.4 |
| **Robust Stitching** (Ruiz 2023, ICML) | ML d'images | ✅ **Pipeline reproduit** — a réparé 5 ruptures de décrépitude (API torchvision, `append` de pandas, dépendances manquantes) |
| **Annotatability** (Nitzan 2024, Nat Comp Sci) | unicellulaire | ✅ **Pipeline reproduit** — 6 tours de débogage d'API ont fait apparaître une dépendance `pooch` manquante |
| **ScType** (Ianevski 2022, Nat Comms) | unicellulaire (R) | ✅ **Pipeline reproduit** — chemin R validé après une rétrogradation de version |
| **Cropformer** (Wang 2025, Plant Communications) | génomique des cultures | ⚠️ **Partiel** — code, modèle et entraînement vérifiés, mais le dépôt ne livre que 10 échantillons de démonstration, donc le PCC=0,92 de l'article ne peut pas être reproduit en l'état |

**6 articles · 1 reproduction sur toutes les métriques · 4 reproductions de pipeline · 1 partiel honnête · 18 améliorations de la compétence vérifiées**

> **Honnête par conception.** UMAP a atteint 78,6 % de concordance des métriques — juste *en dessous* de notre barre des 80 % de « reproduction propre » — et ReproRun le signale comme une contradiction de données plutôt que d'arrondir vers le haut. Pour Cropformer, le cadre tourne de bout en bout, mais les chiffres publiés nécessitent de vraies données de génome de culture que le dépôt ne livre jamais — il est donc marqué Partiel, et non Réussi.

<details>
<summary><b>Étude de cas — Cropformer (⚠️ Reproduction partielle)</b></summary>

**Vérifié ✅**
- Dépôt trouvé et cloné (`jiekesen/Cropformer` ; l'URL de l'article était erronée)
- Environnement construit — Python 3.10 + PyTorch 2.5.1 + CUDA 12.1
- Architecture du modèle — Conv1d + auto-attention à 8 têtes (2,6 M de paramètres)
- Inférence GPU sur RTX 4090 ; les poids pré-entraînés se chargent et s'exécutent
- La boucle d'entraînement converge — perte 89 540 → 23 771

**Impossible à reproduire ❌**
- Métriques de l'article (PCC=0,92, …) — le dépôt ne livre que 10 échantillons de démonstration aléatoires, pas de vraies données de culture
- Tâche de classification — `model_class.py` n'a pas de fonctions clés
- Validation croisée imbriquée, sélection de caractéristiques MIC, encodage SNP 0–9 — non implémentés dans le dépôt

**Cause racine :** le dépôt public est uniquement une démonstration ; le pipeline complet (sélection MIC, validation croisée imbriquée, réglage Optuna) et les vrais jeux de données ne sont pas inclus. Une reproduction fidèle nécessiterait les vraies données de génome de culture → traitement PLINK → réimplémentation du pipeline décrit (environ 1–2 semaines de travail de données et de calcul).
</details>

---

## 📦 Pour commencer

ReproRun est une compétence d'agent qui fonctionne avec n'importe quel assistant de codage IA compatible. Pour l'utiliser :

1. Utilisez un agent de codage IA capable de charger des compétences.
2. Placez le dossier `paper-reproduction/` là où votre agent charge les compétences.
3. Au cours d'une session, il suffit de demander — par exemple *« reproduis scVelo »* ou *« reproduis le Tableau 2 de cet article »*. La compétence se déclenche automatiquement.

---

## 👥 Équipe

| Rôle | Membre |
|------|--------|
| Architecte en chef | [@WUBING2023](https://github.com/WUBING2023) |
| Ingénieur de développement | [@TXZ-star](https://github.com/TXZ-star) |
| Ingénieur de test | [@qaqcrane](https://github.com/qaqcrane) |
| Exploitation | [@wanzi5872-oss](https://github.com/wanzi5872-oss) |

---

## 📄 Licence

**Usage non commercial uniquement.** Vous êtes libre d'**utiliser** et de **modifier** ReproRun à des fins non commerciales (recherche, étude, projets personnels). **L'usage commercial n'est pas autorisé** sans permission préalable.

Licence : **[PolyForm Noncommercial License 1.0.0](../LICENSE)** · [Voir la licence en chinois →](../LICENSE.zh-CN.md)

---

## 📌 Statut

**v1.0.0** · stable (mode maintenance)
