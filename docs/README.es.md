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

> Dale un artículo. Te dice si los resultados realmente se reproducen.

**ReproRun** es una **skill de agente de IA** portátil — utilizable por cualquier asistente de codificación de IA compatible — que automatiza el doloroso camino desde *"un artículo afirma X"* hasta *"X realmente funciona en mi máquina."* Los artículos presentan números hermosos; reproducirlos suele morir en código roto, entornos imposibles de construir y desviaciones de versiones. ReproRun se encarga de toda la cadena de principio a fin:

**leer el artículo → encontrar código y datos → construir el entorno → prueba de humo → ejecución completa → comparar los números medidos con las afirmaciones del artículo.**

---

## ✨ Diseñado para adaptarse — en cualquier plataforma

ReproRun está diseñado para **adaptarse a lo que sea que se le entregue**, en lugar de asumir una única configuración fija:

- **Multi-SO** — funciona en Windows, macOS y Linux
- **Multi-lenguaje** — cadena completa para Python; ruta mínima para R / MATLAB / Julia
- **Multi-dominio** — biología de célula única, ML de imágenes y más, sin reconfiguración por dominio
- **Multi-modo** — *modo herramienta* (`pip install` + escribir un script de llamada) o *modo experimento* (clonar el repositorio y ejecutar sus scripts) — seleccionado automáticamente por artículo

---

## 🚀 Qué hace

- **Encuentra un artículo solo con su título** — búsqueda web automática del PDF y el repositorio de código
- **Diagnostica automáticamente la degradación del entorno** — choques de ABI de numpy, cambios en la API de torchvision, métodos obsoletos de pandas… detectados y corregidos automáticamente
- **Sin adivinar parámetros** — inspecciona las firmas de las funciones tras la instalación
- **Bucle de autorreparación de dependencias de 5 rondas** — clasificar el error → corrección dirigida → reverificar, hasta 5 rondas
- **Aislamiento de artículos** — cada artículo obtiene su propio espacio de nombres de salida

---

## 🏗️ Arquitectura

Un orquestador (`SKILL.md`) dirige 6 agentes especializados:

| Agente | Rol |
|-------|------|
| A · paper-reader | Extraer las afirmaciones numéricas a reproducir |
| B · resource-finder | Localizar el repositorio de código y los conjuntos de datos |
| C · environment-builder | Construir y reparar el entorno de ejecución (el más complejo) |
| D · smoke-tester | Prueba de humo rápida — confirmar que funciona |
| E · full-runner | Ejecución completa de reproducción |
| F · result-comparator | Comparar lo medido frente a lo afirmado, elemento por elemento |

---

## ✅ Validación

ReproRun se ha ejecutado de principio a fin en artículos reales de distintos dominios. No se limita a sellar un "éxito" — para cada artículo devuelve un **veredicto honesto**: números reproducidos, cadena reproducida, o *no se puede reproducir tal cual* — siempre con la causa raíz.

| Artículo | Dominio | Resultado |
|-------|--------|--------|
| **UMAP** (McInnes 2018, JOSS) | reducción de dimensionalidad | ✅ **Números reproducidos** — 11/14 métricas de exactitud de k-NN coinciden dentro de ±0.01; MNIST y Fashion-MNIST confirmados con 3 decimales |
| **scVelo** (Bergen 2020, Nat Biotech) | célula única | ✅ **Cadena reproducida** — detectó un error de ABI de numpy 2.x que causaba 100% de NaN, corregido bajando a 1.26.4 |
| **Robust Stitching** (Ruiz 2023, ICML) | ML de imágenes | ✅ **Cadena reproducida** — reparó 5 roturas por degradación (API de torchvision, `append` de pandas, dependencias faltantes) |
| **Annotatability** (Nitzan 2024, Nat Comp Sci) | célula única | ✅ **Cadena reproducida** — 6 rondas de depuración de API revelaron una dependencia `pooch` faltante |
| **ScType** (Ianevski 2022, Nat Comms) | célula única (R) | ✅ **Cadena reproducida** — ruta de R validada tras bajar de versión |
| **Cropformer** (Wang 2025, Plant Communications) | genómica de cultivos | ⚠️ **Parcial** — código, modelo y entrenamiento verificados, pero el repositorio solo incluye 10 muestras de demostración, por lo que el PCC=0.92 del artículo no se puede reproducir tal cual |

**6 artículos · 1 reproducción de métricas completas · 4 reproducciones de cadena · 1 parcial honesta · 18 mejoras de skill verificadas**

> **Honesto por diseño.** UMAP alcanzó un 78.6% de coincidencia de métricas — justo *por debajo* de nuestro umbral del 80% de "reproducción limpia" — y ReproRun lo reporta como una contradicción de datos en lugar de redondear al alza. Para Cropformer, el marco se ejecuta de principio a fin, pero los números publicados necesitan datos reales de genoma de cultivos que el repositorio nunca incluye — por eso se marca como Parcial, no como Aprobado.

<details>
<summary><b>Estudio de caso — Cropformer (⚠️ Reproducción parcial)</b></summary>

**Verificado ✅**
- Repositorio encontrado y clonado (`jiekesen/Cropformer`; la URL del artículo era incorrecta)
- Entorno construido — Python 3.10 + PyTorch 2.5.1 + CUDA 12.1
- Arquitectura del modelo — Conv1d + autoatención de 8 cabezas (2.6M parámetros)
- Inferencia en GPU en RTX 4090; los pesos preentrenados cargan y se ejecutan
- El bucle de entrenamiento converge — pérdida 89,540 → 23,771

**No se pudo reproducir ❌**
- Métricas del artículo (PCC=0.92, …) — el repositorio solo incluye 10 muestras de demostración aleatorias, no datos reales de cultivos
- Tarea de clasificación — a `model_class.py` le faltan funciones clave
- Validación cruzada anidada, selección de características MIC, codificación SNP 0–9 — no implementadas en el repositorio

**Causa raíz:** el repositorio público es solo de demostración; la cadena completa (selección MIC, CV anidada, ajuste con Optuna) y los conjuntos de datos reales no están incluidos. Una reproducción fiel necesitaría los datos reales de genoma de cultivos → procesamiento con PLINK → reimplementar la cadena descrita (~1–2 semanas de trabajo de datos y cómputo).
</details>

---

## 📦 Primeros pasos

ReproRun es una skill de agente que funciona con cualquier asistente de codificación de IA compatible. Para usarla:

1. Usa un agente de codificación de IA que pueda cargar skills.
2. Coloca la carpeta `paper-reproduction/` donde tu agente carga las skills.
3. En una sesión, simplemente pídelo — p. ej. *"reproduce scVelo"* o *"reproduce la Tabla 2 de este artículo."* La skill se activa automáticamente.

---

## 👥 Equipo

| Rol | Miembro |
|------|--------|
| Arquitecto Jefe | [@WUBING2023](https://github.com/WUBING2023) |
| Ingeniero de Desarrollo | [@TXZ-star](https://github.com/TXZ-star) |
| Ingeniero de Pruebas | [@qaqcrane](https://github.com/qaqcrane) |
| Operaciones | [@wanzi5872-oss](https://github.com/wanzi5872-oss) |

---

## 📄 Licencia

**Solo uso no comercial.** Eres libre de **usar** y **modificar** ReproRun con fines no comerciales (investigación, estudio, proyectos personales). **El uso comercial no está permitido** sin permiso previo.

Licencia: **[PolyForm Noncommercial License 1.0.0](../LICENSE)** · [Ver licencia en chino →](../LICENSE.zh-CN.md)

---

## 📌 Estado

**v1.0.0** · estable (modo de mantenimiento)
