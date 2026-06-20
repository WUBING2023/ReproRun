<p align="center">
  <a href="../README.md">English</a> ·
  <a href="README.zh-CN.md">简体中文</a> ·
  <a href="README.es.md">Español</a> ·
  <a href="README.fr.md">Français</a> ·
  <a href="README.de.md">Deutsch</a> ·
  <a href="README.ja.md">日本語</a> ·
  <a href="README.ko.md">한국어</a> ·
  <b>Português</b> ·
  <a href="README.ru.md">Русский</a>
</p>

# ReproRun

> Dê a ele um artigo. Ele te diz se os resultados realmente se reproduzem.

**ReproRun** é uma **skill de agente de IA** portátil — utilizável por qualquer assistente de programação de IA compatível — que automatiza o caminho doloroso de *"um artigo afirma X"* até *"X realmente roda na minha máquina."* Artigos entregam números bonitos; reproduzi-los geralmente morre em código quebrado, ambientes que não compilam e desvio de versões. O ReproRun cuida de todo o pipeline de ponta a ponta:

**ler o artigo → encontrar código e dados → construir o ambiente → teste de fumaça → execução completa → comparar os números medidos com as afirmações do artigo.**

---

## ✨ Feito para se adaptar — em todas as plataformas

O ReproRun foi projetado para **se adaptar ao que quer que lhe seja entregue**, em vez de assumir uma configuração fixa:

- **Multiplataforma (Cross-OS)** — roda em Windows, macOS e Linux
- **Multilinguagem** — pipeline completo para Python; caminho mínimo para R / MATLAB / Julia
- **Multidomínio** — biologia de célula única, ML de imagem e muito mais, sem reconfiguração por domínio
- **Multimodo** — *modo ferramenta* (`pip install` + escrever um script de chamada) ou *modo experimento* (clonar o repositório e rodar seus scripts) — selecionado automaticamente para cada artigo

---

## 🚀 O que ele faz

- **Encontra um artigo apenas pelo título** — busca automática na web pelo PDF e pelo repositório de código
- **Diagnostica automaticamente a degradação do ambiente** — conflitos de ABI do numpy, mudanças na API do torchvision, métodos depreciados do pandas… detectados e corrigidos automaticamente
- **Sem adivinhar parâmetros** — inspeciona as assinaturas das funções após a instalação
- **Loop de autorreparo de dependências em 5 rodadas** — classificar erro → correção direcionada → reverificar, até 5 rodadas
- **Isolamento de artigos** — cada artigo recebe seu próprio namespace de saída

---

## 🏗️ Arquitetura

Um orquestrador (`SKILL.md`) coordena 6 agentes especializados:

| Agente | Papel |
|-------|------|
| A · paper-reader | Extrair as afirmações numéricas a serem reproduzidas |
| B · resource-finder | Localizar o repositório de código e os conjuntos de dados |
| C · environment-builder | Construir e reparar o ambiente de execução (o mais complexo) |
| D · smoke-tester | Teste de fumaça rápido — confirmar que roda |
| E · full-runner | Execução completa de reprodução |
| F · result-comparator | Comparar medido vs. afirmado, item por item |

---

## ✅ Validação

O ReproRun foi executado de ponta a ponta em artigos reais de diversos domínios. Ele não apenas carimba "sucesso" — para cada artigo ele retorna um **veredito honesto**: números reproduzidos, pipeline reproduzido, ou *não é possível reproduzir como está* — sempre com a causa raiz.

| Artigo | Domínio | Resultado |
|-------|--------|--------|
| **UMAP** (McInnes 2018, JOSS) | redução de dimensionalidade | ✅ **Números reproduzidos** — 11/14 métricas de acurácia k-NN coincidem dentro de ±0,01; MNIST e Fashion-MNIST confirmados em 3 casas decimais |
| **scVelo** (Bergen 2020, Nat Biotech) | célula única | ✅ **Pipeline reproduzido** — detectou um bug de ABI do numpy 2.x que causava 100% de NaN, corrigido fazendo downgrade para 1.26.4 |
| **Robust Stitching** (Ruiz 2023, ICML) | ML de imagem | ✅ **Pipeline reproduzido** — reparou 5 quebras por degradação (API do torchvision, `append` do pandas, dependências faltantes) |
| **Annotatability** (Nitzan 2024, Nat Comp Sci) | célula única | ✅ **Pipeline reproduzido** — 6 rodadas de depuração de API revelaram uma dependência `pooch` faltante |
| **ScType** (Ianevski 2022, Nat Comms) | célula única (R) | ✅ **Pipeline reproduzido** — caminho de R validado após um downgrade de versão |
| **Cropformer** (Wang 2025, Plant Communications) | genômica de cultivos | ⚠️ **Parcial** — código, modelo e treinamento verificados, mas o repositório entrega apenas 10 amostras de demonstração, então o PCC=0,92 do artigo não pode ser reproduzido como está |

**6 artigos · 1 reprodução com métricas completas · 4 reproduções de pipeline · 1 parcial honesta · 18 melhorias verificadas na skill**

> **Honesto por concepção.** O UMAP chegou a 78,6% de coincidência de métricas — logo *abaixo* da nossa barra de 80% de "reprodução limpa" — e o ReproRun reporta isso como uma contradição de dados em vez de arredondar para cima. Para o Cropformer, o framework roda de ponta a ponta, mas os números publicados precisam de dados reais de genoma de cultivos que o repositório nunca entrega — então é marcado como Parcial, não como Aprovado.

<details>
<summary><b>Estudo de caso — Cropformer (⚠️ Reprodução parcial)</b></summary>

**Verificado ✅**
- Repositório encontrado e clonado (`jiekesen/Cropformer`; a URL do artigo estava errada)
- Ambiente construído — Python 3.10 + PyTorch 2.5.1 + CUDA 12.1
- Arquitetura do modelo — Conv1d + autoatenção de 8 cabeças (2,6M parâmetros)
- Inferência em GPU na RTX 4090; pesos pré-treinados carregam e rodam
- Loop de treinamento converge — perda 89.540 → 23.771

**Não foi possível reproduzir ❌**
- Métricas do artigo (PCC=0,92, …) — o repositório entrega apenas 10 amostras de demonstração aleatórias, não dados reais de cultivos
- Tarefa de classificação — `model_class.py` está sem funções essenciais
- Validação cruzada aninhada, seleção de características com MIC, codificação SNP de 0–9 — não implementadas no repositório

**Causa raiz:** o repositório público é apenas de demonstração; o pipeline completo (seleção com MIC, CV aninhada, ajuste com Optuna) e os conjuntos de dados reais não estão incluídos. Uma reprodução fiel precisaria dos dados reais de genoma de cultivos → processamento com PLINK → reimplementação do pipeline descrito (~1–2 semanas de trabalho de dados e computação).
</details>

---

## 📦 Como começar

O ReproRun é uma skill de agente que funciona com qualquer assistente de programação de IA compatível. Para usá-lo:

1. Use um agente de programação de IA capaz de carregar skills.
2. Coloque a pasta `paper-reproduction/` onde o seu agente carrega skills.
3. Em uma sessão, basta pedir — por exemplo, *"reproduza o scVelo"* ou *"reproduza a Tabela 2 deste artigo."* A skill é acionada automaticamente.

---

## 👥 Equipe

| Papel | Membro |
|------|--------|
| Arquiteto-Chefe | [@WUBING2023](https://github.com/WUBING2023) |
| Engenheiro de Desenvolvimento | [@TXZ-star](https://github.com/TXZ-star) |
| Engenheiro de Testes | [@qaqcrane](https://github.com/qaqcrane) |
| Operações | [@wanzi5872-oss](https://github.com/wanzi5872-oss) |

---

## 📄 Licença

**Uso não comercial apenas.** Você é livre para **usar** e **modificar** o ReproRun para fins não comerciais (pesquisa, estudo, projetos pessoais). **O uso comercial não é permitido** sem permissão prévia.

Licença: **[PolyForm Noncommercial License 1.0.0](../LICENSE)** · [Ver licença em chinês →](../LICENSE.zh-CN.md)

---

## 📌 Status

**v1.0.0** · estável (modo de manutenção)
