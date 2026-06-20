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

**ReproRun** ist ein portabler **KI-Agenten-Skill** — nutzbar mit jedem kompatiblen KI-Coding-Assistenten — der den mühsamen Weg von *„ein Paper behauptet X“* zu *„X läuft tatsächlich auf meinem Rechner“* automatisiert. Papers liefern wunderschöne Zahlen; ihre Reproduktion scheitert meist an kaputtem Code, nicht baubaren Umgebungen und Versionsdrift. ReproRun bewältigt die gesamte Pipeline von Anfang bis Ende:

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

Von Anfang bis Ende an 4 Papers aus verschiedenen Domänen und Sprachen validiert:

| Paper | Domäne | Sprache | Smoke-Test |
|-------|--------|----------|:--:|
| scVelo (Nat Biotech 2020) | Einzelzelle | Python | ✓ |
| ScType (Nat Comms 2022) | Einzelzelle | R | ✓ |
| Annotatability (Nat Comp Sci 2024) | Einzelzelle | Python | ✓ |
| Robust Stitching (ICML 2023) | Bild-ML | Python | ✓ |

**Smoke-Test-Erfolgsquote 4/4 · P1-Blocker 0 · 18 verifizierte Verbesserungen**

---

## 📦 Erste Schritte

ReproRun ist ein Agenten-Skill, der mit jedem kompatiblen KI-Coding-Assistenten funktioniert. So verwendest du es:

1. Verwende einen KI-Coding-Agenten, der Skills laden kann.
2. Lege den Ordner `paper-reproduction/` dort ab, wo dein Agent Skills lädt.
3. Frage in einer Sitzung einfach danach — z. B. *„reproduce scVelo“* oder *„reproduce Table 2 from this paper.“* Der Skill wird automatisch ausgelöst.

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
