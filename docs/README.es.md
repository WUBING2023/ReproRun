<p align="center">
  <a href="../README.md">English</a> ·
  <a href="README.zh-CN.md">简体中文</a> ·
  <b>Español</b> ·
  <a href="README.fr.md">Français</a> ·
  <a href="README.de.md">Deutsch</a> ·
  <a href="README.ja.md">日本語</a> ·
  <a href="README.ko.md">한국어</a> ·
  <a href="README.pt-BR.md">Português</a> ·
  <a href="README.ru.md">Русский</a>
</p>

# ReproRun

> Dale un artículo. Te dirá si los resultados realmente se reproducen.

**ReproRun** es una skill personalizada de [Claude Code](https://claude.com/claude-code) que automatiza el doloroso camino desde *"un artículo afirma X"* hasta *"X realmente funciona en mi máquina."* Los artículos presentan cifras hermosas; reproducirlas normalmente muere en código roto, entornos que no se pueden construir y deriva de versiones. ReproRun gestiona toda la canalización de principio a fin:

**leer el artículo → encontrar código y datos → construir el entorno → prueba de humo → ejecución completa → comparar las cifras medidas con las afirmaciones del artículo.**

---

## ✨ Diseñado para adaptarse — en todas las plataformas

ReproRun está diseñado para **adaptarse a lo que se le entregue**, en lugar de asumir una única configuración fija:

- **Multiplataforma (SO)** — funciona en Windows, macOS y Linux
- **Multilenguaje** — canalización completa para Python; ruta mínima para R / MATLAB / Julia
- **Multidominio** — biología de células individuales, ML de imágenes y más, sin reconfiguración por dominio
- **Multimodo** — *modo herramienta* (`pip install` + escribir un script de llamada) o *modo experimento* (clonar el repositorio y ejecutar sus scripts) — seleccionado automáticamente para cada artículo

---

## 🚀 Qué hace

- **Encuentra un artículo solo con su título** — búsqueda web automática del PDF y el repositorio de código
- **Diagnóstico automático del deterioro del entorno** — conflictos de ABI de numpy, cambios en la API de torchvision, métodos obsoletos de pandas… detectados y corregidos automáticamente
- **Sin adivinar parámetros** — inspecciona las firmas de las funciones tras la instalación
- **Bucle de autorreparación de dependencias de 5 rondas** — clasificar el error → corrección dirigida → volver a verificar, hasta 5 rondas
- **Aislamiento de artículos** — cada artículo obtiene su propio espacio de nombres de salida

---

## 🏗️ Arquitectura

Un orquestador (`SKILL.md`) dirige 6 agentes especializados:

| Agente | Función |
|-------|------|
| A · paper-reader | Extraer las afirmaciones numéricas a reproducir |
| B · resource-finder | Localizar el repositorio de código y los conjuntos de datos |
| C · environment-builder | Construir y reparar el entorno de ejecución (el más complejo) |
| D · smoke-tester | Prueba de humo rápida — confirmar que funciona |
| E · full-runner | Ejecución completa de reproducción |
| F · result-comparator | Comparar lo medido con lo afirmado, elemento por elemento |

---

## ✅ Validación

Validado de principio a fin en 4 artículos de distintos dominios y lenguajes:

| Artículo | Dominio | Lenguaje | Prueba de humo |
|-------|--------|----------|:--:|
| scVelo (Nat Biotech 2020) | célula individual | Python | ✓ |
| ScType (Nat Comms 2022) | célula individual | R | ✓ |
| Annotatability (Nat Comp Sci 2024) | célula individual | Python | ✓ |
| Robust Stitching (ICML 2023) | ML de imágenes | Python | ✓ |

**Tasa de aprobación de prueba de humo 4/4 · Bloqueadores P1 0 · 18 mejoras verificadas**

---

## 📦 Primeros pasos

ReproRun es una skill de Claude Code. Para usarla:

1. Instala [Claude Code](https://claude.com/claude-code).
2. Coloca la carpeta `paper-reproduction/` donde Claude Code pueda cargar skills.
3. En una sesión de Claude Code, simplemente pídelo — por ejemplo, *"reproduce scVelo"* o *"reproduce la Tabla 2 de este artículo."* La skill se activa automáticamente.

---

## 👥 Equipo

| Función | Miembro |
|------|--------|
| Arquitecto jefe | [@WUBING2023](https://github.com/WUBING2023) |
| Ingeniero de desarrollo | [@TXZ-star](https://github.com/TXZ-star) |
| Ingeniero de pruebas | [@qaqcrane](https://github.com/qaqcrane) |
| Operaciones | [@wanzi5872-oss](https://github.com/wanzi5872-oss) |

---

## 📄 Licencia

**Solo uso no comercial.** Eres libre de **usar** y **modificar** ReproRun con fines no comerciales (investigación, estudio, proyectos personales). **No se permite el uso comercial** sin permiso previo.

Licencia: **[PolyForm Noncommercial License 1.0.0](../LICENSE)** · [Ver licencia en chino →](../LICENSE.zh-CN.md)

---

## 📌 Estado

**v1.0.0** · estable (modo de mantenimiento) · mantenido en Claude Code
