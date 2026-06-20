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

**ReproRun** é um **skill de agente de IA** portátil — utilizável por qualquer assistente de programação com IA compatível — que automatiza o caminho doloroso de *"um artigo afirma X"* até *"X realmente roda na minha máquina."* Os artigos trazem números bonitos; reproduzi-los geralmente morre em código quebrado, ambientes que não compilam e divergência de versões. O ReproRun cuida de todo o pipeline de ponta a ponta:

**ler o artigo → encontrar código e dados → construir o ambiente → teste de fumaça → execução completa → comparar os números medidos com o que o artigo afirma.**

---

## ✨ Feito para se adaptar — em todas as plataformas

O ReproRun foi projetado para **se adaptar a tudo o que lhe é entregue**, em vez de assumir uma única configuração fixa:

- **Multi-SO** — roda em Windows, macOS e Linux
- **Multilinguagem** — pipeline completo para Python; caminho mínimo para R / MATLAB / Julia
- **Multidomínio** — biologia de célula única, ML de imagens e muito mais, sem reconfiguração por domínio
- **Multimodo** — *modo ferramenta* (`pip install` + escrever um script de chamada) ou *modo experimento* (clonar o repositório e rodar seus scripts) — selecionado automaticamente para cada artigo

---

## 🚀 O que ele faz

- **Encontra um artigo apenas pelo título** — busca automática na web pelo PDF e pelo repositório de código
- **Diagnostica automaticamente a deterioração do ambiente** — conflitos de ABI do numpy, mudanças de API do torchvision, métodos depreciados do pandas… detectados e corrigidos automaticamente
- **Sem adivinhar parâmetros** — inspeciona as assinaturas das funções após a instalação
- **Laço de autorreparo de dependências em 5 rodadas** — classifica o erro → correção direcionada → reverifica, em até 5 rodadas
- **Isolamento de artigos** — cada artigo recebe seu próprio namespace de saída

---

## 🏗️ Arquitetura

Um orquestrador (`SKILL.md`) comanda 6 agentes especializados:

| Agente | Função |
|-------|------|
| A · paper-reader | Extrair as afirmações numéricas a serem reproduzidas |
| B · resource-finder | Localizar o repositório de código e os conjuntos de dados |
| C · environment-builder | Construir e reparar o ambiente de execução (o mais complexo) |
| D · smoke-tester | Teste de fumaça rápido — confirmar que roda |
| E · full-runner | Execução completa de reprodução |
| F · result-comparator | Comparar medido vs. afirmado, item por item |

---

## ✅ Validação

Validado de ponta a ponta em 4 artigos de diferentes domínios e linguagens:

| Artigo | Domínio | Linguagem | Teste de Fumaça |
|-------|--------|----------|:--:|
| scVelo (Nat Biotech 2020) | célula única | Python | ✓ |
| ScType (Nat Comms 2022) | célula única | R | ✓ |
| Annotatability (Nat Comp Sci 2024) | célula única | Python | ✓ |
| Robust Stitching (ICML 2023) | ML de imagens | Python | ✓ |

**Taxa de aprovação no teste de fumaça 4/4 · Bloqueadores P1 0 · 18 melhorias verificadas**

---

## 📦 Primeiros passos

O ReproRun é um skill de agente que funciona com qualquer assistente de programação com IA compatível. Para usá-lo:

1. Use um agente de programação com IA capaz de carregar skills.
2. Coloque a pasta `paper-reproduction/` onde o seu agente carrega skills.
3. Em uma sessão, basta pedir — por exemplo, *"reproduza o scVelo"* ou *"reproduza a Tabela 2 deste artigo."* O skill é acionado automaticamente.

---

## 👥 Equipe

| Função | Membro |
|------|--------|
| Arquiteto-chefe | [@WUBING2023](https://github.com/WUBING2023) |
| Engenheiro de Desenvolvimento | [@TXZ-star](https://github.com/TXZ-star) |
| Engenheiro de Testes | [@qaqcrane](https://github.com/qaqcrane) |
| Operações | [@wanzi5872-oss](https://github.com/wanzi5872-oss) |

---

## 📄 Licença

**Apenas para uso não comercial.** Você é livre para **usar** e **modificar** o ReproRun para fins não comerciais (pesquisa, estudo, projetos pessoais). **O uso comercial não é permitido** sem permissão prévia.

Licença: **[PolyForm Noncommercial License 1.0.0](../LICENSE)** · [Ver licença em chinês →](../LICENSE.zh-CN.md)

---

## 📌 Status

**v1.0.0** · estável (modo de manutenção)
