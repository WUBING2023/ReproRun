<p align="center">
  <a href="../README.md">English</a> ·
  <a href="README.zh-CN.md">简体中文</a> ·
  <a href="README.es.md">Español</a> ·
  <a href="README.fr.md">Français</a> ·
  <b>Deutsch</b> ·
  <a href="README.ja.md">日本語</a> ·
  <a href="README.ko.md">한국어</a> ·
  <a href="README.pt-BR.md">Português</a> ·
  <a href="README.ru.md">Русский</a>
</p>

# ReproRun

> Gib ihm ein Paper. Es sagt dir, ob sich die Ergebnisse tatsächlich reproduzieren lassen.

**ReproRun** ist eine portable **AI-agent skill** — nutzbar von jedem kompatiblen KI-Coding-Assistenten — die den mühsamen Weg von *„ein Paper behauptet X"* zu *„X läuft tatsächlich auf meinem Rechner"* automatisiert. Papers liefern wunderschöne Zahlen; ihre Reproduktion scheitert meist an kaputtem Code, nicht baubaren Umgebungen und Versionsdrift. ReproRun bewältigt die gesamte Pipeline von Anfang bis Ende:

**Paper lesen → Code & Daten finden → Umgebung bauen → Smoke-Test → vollständiger Lauf → gemessene Zahlen mit den Behauptungen des Papers vergleichen.**

---

## ✨ Gebaut zur Anpassung — auf jeder Plattform

ReproRun ist darauf ausgelegt, sich **an alles anzupassen, was ihm übergeben wird**, anstatt von einem festen Setup auszugehen:

- **Plattformübergreifend** — läuft unter Windows, macOS und Linux
- **Sprachübergreifend** — vollständige Pipeline für Python; minimaler Pfad für R / MATLAB / Julia
- **Domänenübergreifend** — Einzelzellbiologie, Bild-ML und mehr, ohne domänenspezifische Umverdrahtung
- **Modusübergreifend** — *Tool-Modus* (`pip install` + ein aufrufendes Skript schreiben) oder *Experiment-Modus* (das Repo klonen & seine Skripte ausführen) — automatisch pro Paper ausgewählt

---

## 🚀 Was es tut

- **Ein Paper allein anhand seines Titels finden** — automatische Websuche nach PDF & Code-Repo
- **Umgebungs-Bit-Rot automatisch diagnostizieren** — numpy-ABI-Konflikte, torchvision-API-Änderungen, veraltete pandas-Methoden… automatisch erkannt und behoben
- **Kein Raten von Parametern** — inspiziert Funktionssignaturen nach der Installation
- **Selbstheilende Abhängigkeitsschleife über 5 Runden** — Fehler klassifizieren → gezielte Behebung → erneut prüfen, bis zu 5 Runden
- **Paper-Isolation** — jedes Paper erhält seinen eigenen Ausgabe-Namensraum

---

## 🏗️ Architektur

Ein Orchestrator (`SKILL.md`) steuert 6 spezialisierte Agenten:

| Agent | Rolle |
|-------|------|
| A · paper-reader | Extrahiert die zu reproduzierenden numerischen Behauptungen |
| B · resource-finder | Lokalisiert Code-Repo & Datensätze |
| C · environment-builder | Baut & repariert die Laufzeitumgebung (am komplexesten) |
| D · smoke-tester | Schneller Smoke-Test — bestätigt, dass es läuft |
| E · full-runner | Vollständiger Reproduktionslauf |
| F · result-comparator | Vergleicht gemessen vs. behauptet, Punkt für Punkt |

---

## ✅ Validierung

ReproRun wurde an echten Papers aus verschiedenen Domänen von Anfang bis Ende ausgeführt. Es stempelt nicht einfach „Erfolg" ab — für jedes Paper liefert es ein **ehrliches Urteil**: Zahlen reproduziert, Pipeline reproduziert oder *im Ist-Zustand nicht reproduzierbar* — stets mit der zugrunde liegenden Ursache.

| Paper | Domäne | Ergebnis |
|-------|--------|--------|
| **UMAP** (McInnes 2018, JOSS) | Dimensionsreduktion | ✅ **Zahlen reproduziert** — 11/14 k-NN-Genauigkeitsmetriken stimmen innerhalb von ±0,01 überein; MNIST & Fashion-MNIST auf 3 Dezimalstellen bestätigt |
| **scVelo** (Bergen 2020, Nat Biotech) | Einzelzelle | ✅ **Pipeline reproduziert** — entdeckte einen numpy-2.x-ABI-Bug, der zu 100 % NaN führte, behoben durch Downgrade auf 1.26.4 |
| **Robust Stitching** (Ruiz 2023, ICML) | Bild-ML | ✅ **Pipeline reproduziert** — 5 Bit-Rot-Defekte repariert (torchvision-API, pandas `append`, fehlende Abhängigkeiten) |
| **Annotatability** (Nitzan 2024, Nat Comp Sci) | Einzelzelle | ✅ **Pipeline reproduziert** — 6 API-Debug-Runden brachten eine fehlende `pooch`-Abhängigkeit zum Vorschein |
| **ScType** (Ianevski 2022, Nat Comms) | Einzelzelle (R) | ✅ **Pipeline reproduziert** — R-Pfad nach einem Versions-Downgrade validiert |
| **Cropformer** (Wang 2025, Plant Communications) | Crop-Genomik | ⚠️ **Teilweise** — Code, Modell & Training verifiziert, aber das Repo liefert nur 10 Demo-Samples, sodass der PCC=0,92 des Papers im Ist-Zustand nicht reproduzierbar ist |

**6 Papers · 1 vollständige Metrik-Reproduktion · 4 Pipeline-Reproduktionen · 1 ehrliche Teilreproduktion · 18 verifizierte Skill-Verbesserungen**

> **Ehrlich by design.** UMAP landete bei 78,6 % Metrik-Übereinstimmung — knapp *unter* unserer 80-%-Schwelle für eine „saubere Reproduktion" — und ReproRun meldet dies als Datenwiderspruch, anstatt aufzurunden. Bei Cropformer läuft das Framework von Anfang bis Ende, aber die veröffentlichten Zahlen erfordern echte Crop-Genom-Daten, die das Repo nie liefert — daher wird es als Teilweise gekennzeichnet, nicht als Bestanden.

<details>
<summary><b>Fallstudie — Cropformer (⚠️ Teilreproduktion)</b></summary>

**Verifiziert ✅**
- Repo gefunden & geklont (`jiekesen/Cropformer`; die URL des Papers war falsch)
- Umgebung gebaut — Python 3.10 + PyTorch 2.5.1 + CUDA 12.1
- Modellarchitektur — Conv1d + 8-Head-Self-Attention (2,6 Mio. Parameter)
- GPU-Inferenz auf RTX 4090; vortrainierte Gewichte laden & laufen
- Trainingsschleife konvergiert — Loss 89.540 → 23.771

**Nicht reproduzierbar ❌**
- Paper-Metriken (PCC=0,92, …) — das Repo liefert nur 10 zufällige Demo-Samples, keine echten Crop-Daten
- Klassifikationsaufgabe — `model_class.py` fehlen Schlüsselfunktionen
- Verschachtelte Kreuzvalidierung, MIC-Feature-Selektion, 0–9-SNP-Kodierung — im Repo nicht implementiert

**Ursache:** Das öffentliche Repo ist reine Demo; die vollständige Pipeline (MIC-Selektion, verschachtelte CV, Optuna-Tuning) und die echten Datensätze sind nicht enthalten. Eine getreue Reproduktion bräuchte die echten Crop-Genom-Daten → PLINK-Verarbeitung → Neuimplementierung der beschriebenen Pipeline (~1–2 Wochen Daten- + Rechenarbeit).
</details>

---

## 📦 Erste Schritte

ReproRun ist ein Agenten-Skill, der mit jedem kompatiblen KI-Coding-Assistenten funktioniert. So verwendest du es:

1. Verwende einen KI-Coding-Agenten, der Skills laden kann.
2. Lege den Ordner `paper-reproduction/` dort ab, wo dein Agent Skills lädt.
3. Frage in einer Sitzung einfach danach — z. B. *„reproduce scVelo"* oder *„reproduce Table 2 from this paper."* Der Skill wird automatisch ausgelöst.

---

## 👥 Team

| Rolle | Mitglied |
|------|--------|
| Chefarchitekt | [@WUBING2023](https://github.com/WUBING2023) |
| Entwicklungsingenieur | [@TXZ-star](https://github.com/TXZ-star) |
| Testingenieur | [@qaqcrane](https://github.com/qaqcrane) |
| Betrieb | [@wanzi5872-oss](https://github.com/wanzi5872-oss) |

---

## 📄 Lizenz

**Nur für nicht-kommerzielle Nutzung.** Du darfst ReproRun für nicht-kommerzielle Zwecke (Forschung, Studium, persönliche Projekte) frei **verwenden** und **modifizieren**. **Kommerzielle Nutzung ist ohne vorherige Genehmigung nicht gestattet.**

Lizenz: **[PolyForm Noncommercial License 1.0.0](../LICENSE)** · [Lizenz auf Chinesisch ansehen →](../LICENSE.zh-CN.md)

---

## 📌 Status

**v1.0.0** · stabil (Wartungsmodus)
