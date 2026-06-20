<p align="center">
  <a href="../README.md">English</a> ·
  <a href="README.zh-CN.md">简体中文</a> ·
  <a href="README.es.md">Español</a> ·
  <a href="README.fr.md">Français</a> ·
  <a href="README.de.md">Deutsch</a> ·
  <b>日本語</b> ·
  <a href="README.ko.md">한국어</a> ·
  <a href="README.pt-BR.md">Português</a> ·
  <a href="README.ru.md">Русский</a>
</p>

# ReproRun

> 論文を渡せば、その結果が本当に再現できるかどうかを教えてくれます。

**ReproRun** は、*「ある論文が X を主張している」* から *「X が実際に自分のマシンで動く」* までの苦痛に満ちた道のりを自動化する、[Claude Code](https://claude.com/claude-code) のカスタムスキルです。論文は美しい数値を提示しますが、その再現はたいてい壊れたコード、ビルドできない環境、バージョンのずれによって頓挫します。ReproRun はパイプライン全体をエンドツーエンドで処理します。

**論文を読む → コードとデータを探す → 環境を構築する → スモークテスト → フル実行 → 測定した数値を論文の主張と比較する。**

---

## ✨ あらゆるプラットフォームに適応する設計

ReproRun は、ひとつの固定されたセットアップを前提とするのではなく、**渡されたものが何であれそれに自ら適応する** ように設計されています。

- **クロス OS** — Windows、macOS、Linux で動作
- **クロス言語** — Python は完全なパイプライン、R / MATLAB / Julia は最小限の経路
- **クロスドメイン** — シングルセル生物学、画像 ML など、ドメインごとの再配線は不要
- **クロスモード** — *ツールモード*（`pip install` + 呼び出しスクリプトの作成）または *実験モード*（リポジトリをクローンしてそのスクリプトを実行）— 論文ごとに自動選択

---

## 🚀 できること

- **タイトルだけから論文を見つける** — PDF とコードリポジトリを自動ウェブ検索
- **環境の経年劣化を自動診断** — numpy の ABI 衝突、torchvision の API 変更、非推奨になった pandas メソッドなど… を自動的に検出して修正
- **パラメータの当て推量なし** — インストール後に関数のシグネチャを検査
- **5 ラウンドの依存関係自己修復ループ** — エラーを分類 → 的を絞った修正 → 再検証、最大 5 ラウンド
- **論文の分離** — 各論文に専用の出力名前空間を割り当て

---

## 🏗️ アーキテクチャ

ひとつのオーケストレーター（`SKILL.md`）が 6 つの専門エージェントを駆動します。

| エージェント | 役割 |
|-------|------|
| A · paper-reader | 再現すべき数値的主張を抽出する |
| B · resource-finder | コードリポジトリとデータセットを特定する |
| C · environment-builder | ランタイムを構築・修復する（最も複雑） |
| D · smoke-tester | クイックスモークテスト — 動作を確認する |
| E · full-runner | フル再現実行 |
| F · result-comparator | 測定値と主張値を項目ごとに比較する |

---

## ✅ 検証

異なるドメインと言語にまたがる 4 本の論文でエンドツーエンドに検証済みです。

| 論文 | ドメイン | 言語 | スモークテスト |
|-------|--------|----------|:--:|
| scVelo (Nat Biotech 2020) | シングルセル | Python | ✓ |
| ScType (Nat Comms 2022) | シングルセル | R | ✓ |
| Annotatability (Nat Comp Sci 2024) | シングルセル | Python | ✓ |
| Robust Stitching (ICML 2023) | 画像 ML | Python | ✓ |

**スモークテスト合格率 4/4 · P1 ブロッカー 0 · 18 件の検証済み改善**

---

## 📦 はじめに

ReproRun は Claude Code のスキルです。使い方は次のとおりです。

1. [Claude Code](https://claude.com/claude-code) をインストールします。
2. `paper-reproduction/` フォルダを、Claude Code がスキルを読み込める場所に配置します。
3. Claude Code のセッションで、ただ頼むだけです — 例えば *「reproduce scVelo」* や *「この論文の Table 2 を再現して」* など。スキルは自動的に起動します。

---

## 👥 チーム

| 役割 | メンバー |
|------|--------|
| チーフアーキテクト | [@WUBING2023](https://github.com/WUBING2023) |
| 開発エンジニア | [@TXZ-star](https://github.com/TXZ-star) |
| テストエンジニア | [@qaqcrane](https://github.com/qaqcrane) |
| オペレーション | [@wanzi5872-oss](https://github.com/wanzi5872-oss) |

---

## 📄 ライセンス

**非商用利用のみ。** ReproRun を非商用目的（研究、学習、個人プロジェクト）のために **使用** および **改変** することは自由です。**商用利用は** 事前の許可なしには **認められません**。

ライセンス: **[PolyForm Noncommercial License 1.0.0](../LICENSE)** · [中国語のライセンスを見る →](../LICENSE.zh-CN.md)

---

## 📌 ステータス

**v1.0.0** · 安定版（メンテナンスモード）· Claude Code 上で保守

