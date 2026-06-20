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

> Donnez-lui un article. Il vous dit si les résultats se reproduisent vraiment.

**ReproRun** est une compétence (skill) personnalisée pour [Claude Code](https://claude.com/claude-code) qui automatise le chemin pénible allant de *« un article affirme X »* à *« X tourne réellement sur ma machine. »* Les articles présentent de beaux chiffres ; les reproduire échoue généralement à cause de code cassé, d'environnements impossibles à construire et de dérives de versions. ReproRun gère l'ensemble du pipeline de bout en bout :

**lire l'article → trouver le code et les données → construire l'environnement → test de fumée → exécution complète → comparer les chiffres mesurés aux affirmations de l'article.**

---

## ✨ Conçu pour s'adapter — sur toutes les plateformes

ReproRun est conçu pour **s'adapter à tout ce qu'on lui confie**, au lieu de présupposer une configuration figée :

- **Multi-OS** — fonctionne sous Windows, macOS et Linux
- **Multi-langage** — pipeline complet pour Python ; chemin minimal pour R / MATLAB / Julia
- **Multi-domaine** — biologie unicellulaire, ML d'image, et plus encore, sans reconfiguration par domaine
- **Multi-mode** — *mode outil* (`pip install` + écriture d'un script appelant) ou *mode expérience* (cloner le dépôt et lancer ses scripts) — sélectionné automatiquement pour chaque article

---

## 🚀 Ce qu'il fait

- **Trouver un article à partir de son seul titre** — recherche web automatique du PDF et du dépôt de code
- **Diagnostiquer automatiquement la décrépitude de l'environnement** — conflits d'ABI numpy, changements d'API torchvision, méthodes pandas dépréciées… détectés et corrigés automatiquement
- **Aucune supposition sur les paramètres** — inspecte les signatures de fonctions après installation
- **Boucle d'auto-réparation des dépendances sur 5 tours** — classer l'erreur → correction ciblée → revérifier, jusqu'à 5 tours
- **Isolation des articles** — chaque article obtient son propre espace de noms de sortie

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

Validé de bout en bout sur 4 articles dans différents domaines et langages :

| Article | Domaine | Langage | Test de fumée |
|-------|--------|----------|:--:|
| scVelo (Nat Biotech 2020) | unicellulaire | Python | ✓ |
| ScType (Nat Comms 2022) | unicellulaire | R | ✓ |
| Annotatability (Nat Comp Sci 2024) | unicellulaire | Python | ✓ |
| Robust Stitching (ICML 2023) | ML d'image | Python | ✓ |

**Taux de réussite des tests de fumée 4/4 · Bloqueurs P1 0 · 18 améliorations vérifiées**

---

## 📦 Pour commencer

ReproRun est une compétence Claude Code. Pour l'utiliser :

1. Installez [Claude Code](https://claude.com/claude-code).
2. Placez le dossier `paper-reproduction/` là où Claude Code peut charger les compétences.
3. Dans une session Claude Code, il suffit de demander — par ex. *« reproduis scVelo »* ou *« reproduis le Tableau 2 de cet article. »* La compétence se déclenche automatiquement.

---

## 👥 Équipe

| Rôle | Membre |
|------|--------|
| Architecte en chef | [@WUBING2023](https://github.com/WUBING2023) |
| Ingénieur de développement | [@TXZ-star](https://github.com/TXZ-star) |
| Ingénieur de test | [@qaqcrane](https://github.com/qaqcrane) |
| Opérations | [@wanzi5872-oss](https://github.com/wanzi5872-oss) |

---

## 📄 Licence

**Usage non commercial uniquement.** Vous êtes libre d'**utiliser** et de **modifier** ReproRun à des fins non commerciales (recherche, étude, projets personnels). **L'usage commercial n'est pas autorisé** sans permission préalable.

Licence : **[PolyForm Noncommercial License 1.0.0](../LICENSE)** · [Voir la licence en chinois →](../LICENSE.zh-CN.md)

---

## 📌 Statut

**v1.0.0** · stable (mode maintenance) · maintenu sur Claude Code
