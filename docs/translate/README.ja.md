<div align="center">

# Spec-Driven Document-First Development Framework

**仕様駆動 · ドキュメントファースト · AI支援**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Compatible-blueviolet)](https://claude.ai/claude-code)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)

[English](./README.en.md) · [中文](../../README.md) · 日本語

---

*仕様駆動・ドキュメントファーストの要件管理フレームワーク（マルチプロダクト対応）*

</div>

## ✨ なぜこのフレームワーク？

| 従来のアプローチ | Spec-Driven |
|----------------|-------------|
| 最初にすべてのドキュメントを作成 | 変更に応じて段階的に拡充 |
| ドキュメントとコードが乖離 | ドキュメントが唯一の信頼源 |
| 手動で整合性を確認 | AIが検証と更新を支援 |
| 要件変更の追跡が困難 | CR-IDで全工程を追跡 |

## 🎯 コアコンセプト

- **📝 変更駆動** - 一度にすべてを書く必要なし、変更ごとに段階的に拡充
- **🤖 AI支援** - 自然言語で入力、AIが統一フォーマットに整形
- **📚 ドキュメントが真実** - 確定したドキュメントが開発・テストの唯一の基準
- **🔍 粗から細へ** - まずモジュール概要、必要に応じて機能を詳細化

## 💪 フレームワークの強み

### インテリジェントな依存関係管理
- **双方向追跡**: deps（依存先）+ affects（依存元）を自動管理
- **スコープ拡張チェック**: confirm時に漏れた依存変更を自動検出
- **グローバル依存グラフ**: `_deps.yaml`で迅速な影響分析が可能

### Context Loading最適化
- **階層型ロード戦略**: Level 0-3のオンデマンドロードでトークン使用量を最適化
- **常時ロード**: project.yaml, glossary.yaml, overview.md
- **オンデマンド**: 関連featureのfrontmatterとコンテンツのみロード

### セーフティガード設計
- **RCプレビュー機構**: マージ前にプレビューを生成、人間が確認後に正式化
- **暗黙の状態ロールバック**: confirmedなCRを修正時に自動警告とロールバック
- **依存スコープ保護**: 影響を受けるドキュメントの漏れを防止

### 並行作業フレンドリー
- gitブランチで並行作業を実現、各ブランチに独立したRC
- マージ時はrebaseを使用、ルールが明確
- 並行作業のボトルネックなし

## 🚀 クイックスタート

```bash
# 1. プロダクトを初期化
/dd-init my-product

# 2. プロダクト概要を記述
/dd-update "ユーザー管理、注文、決済の3つのモジュールがあります..."

# 3. 変更を確定
/dd-confirm CR-001

# 4. (オプション) 仕様を生成
/dd-spec-dev CR-001
/dd-spec-test CR-001

# 5. 完了としてマーク
/dd-done CR-001
```

## 📂 ディレクトリ構造

```
spec/
├── CLAUDE.md                     # AI動作ガイド
├── README.md                     # このファイル
│
└── products/
    └── {product}/
        ├── project.yaml          # プロダクト設定 (next_cr_id含む)
        ├── glossary.yaml         # 用語集 (人間が管理)
        ├── overview.md           # プロダクト概要
        │
        ├── features/             # 機能ドキュメント
        │   ├── {feature}.md      # ビジネス要件 (正式版)
        │   ├── {feature}.rc-{id}.md    # ビジネス要件 (CRプレビュー)
        │   ├── {feature}.tech.md       # 技術的合意事項 (正式版)
        │   └── {feature}.tech.rc-{id}.md # 技術的合意事項 (CRプレビュー)
        │
        ├── changes/              # 変更記録
        │   ├── _index.yaml       # 変更インデックス
        │   ├── CR-{id}.md        # 進行中の変更
        │   ├── archive/          # 完了した変更
        │   └── dropped/          # 破棄された変更
        │
        └── specs/                # 仕様ファイル
            ├── _index.yaml       # 仕様インデックス
            ├── CR-{id}.dev.md    # 開発仕様
            ├── CR-{id}.test.md   # テスト仕様
            ├── archive/          # 完了した仕様
            └── dropped/          # 破棄された仕様
```

## 🛠️ Skills

> **dd** = **D**ocument-**D**riven、Spec-Driven **D**ocument-First の「D」でもあります。
> すべてのスキルは `/dd-` プレフィックスを使用し、「ドキュメント駆動」の理念を体現しています。

### コアSkills

| Skill | 用途 | 説明 |
|-------|------|------|
| `/dd-init` | プロダクト初期化 | 完全なディレクトリ構造を作成 |
| `/dd-status` | ステータス確認 | プロダクト/変更/RC/仕様の統計 |
| `/dd-update` | 変更の作成/修正 | 自然言語入力、confirmedからロールバック可能 |
| `/dd-confirm` | 変更を確定 | RCプレビュードキュメントを生成 |
| `/dd-done` | 完了としてマーク | RCを正式版にマージ、アーカイブ |
| `/dd-drop` | 変更を破棄 | RCと仕様を削除、droppedに移動 |

### 補助Skills

| Skill | 用途 | 説明 |
|-------|------|------|
| `/dd-check` | 総合チェック | コンソール出力のみ、ブロックなし |
| `/dd-rebase` | ブランチ競合処理 | 意図に基づいて変更を再適用 |
| `/dd-spec-dev` | 開発仕様を生成 | confirmedステータスが必要 |
| `/dd-spec-test` | テスト仕様を生成 | confirmedが必要、--initをサポート |

## 🔄 ワークフロー詳細

### 状態遷移

```
draft → confirmed → done (アーカイブ)
  │         │
  └────┬────┘
       ↓
    dropped
```

### 標準フロー

| ステップ | コマンド | 出力 | 説明 |
|---------|---------|------|------|
| 1 | `/dd-update "説明"` | CR-{id}.md | 変更記録を作成、影響範囲を分析 |
| 2 | 人間がレビュー | - | 複数回の `/dd-update CR-{id}` で調整可能 |
| 3 | `/dd-confirm CR-{id}` | *.rc-{id}.md | RCプレビュードキュメントを生成 |
| 4 | `/dd-spec-dev\|test` | specs/*.md | オプション：開発/テスト仕様を生成 |
| 5 | `/dd-done CR-{id}` | 正式版 | RCをマージ、CRと仕様をアーカイブ |

### 依存変更フロー

```
/dd-update   →  依存変更を分析、CRに記録
                ↓
/dd-confirm  →  スコープ外依存をチェック
                ├─ 漏れあり → CRを自動拡張、レビュー待ちで終了
                └─ 完全 → RCを生成、双方向依存を更新
                ↓
/dd-done     →  RCをマージ、_deps.yamlを再構築
```

### 特殊モード

| モード | コマンド | 用途 |
|--------|---------|------|
| コールドスタート | `/dd-update "説明" --bootstrap` | feature.mdを直接作成、CRをスキップ |
| 実装済み | `/dd-update "説明" --implemented` | CRフローを通るがdev specなし |
| 状態ロールバック | `/dd-update CR-{id}` (confirmed) | 警告後RC/specを削除、draftに戻る |

### ステータス説明

| ステータス | ドキュメント変化 |
|-----------|-----------------|
| draft | CRのみ、ドキュメント変更なし |
| confirmed | `.rc-{id}.md` プレビューを生成 |
| done | RCを削除、正式版にマージ、アーカイブ |
| dropped | RC/specを削除、dropped/に移動 |

## 📖 詳細ドキュメント

- **[CLAUDE.md](../../CLAUDE.md)** - AI動作ガイド、ドキュメントフォーマット、ワークフロー詳細

## 🤔 設計思想

「最も価値のあるコンテキストをいかに構築するか」を軸に設計されており、すべてのトークンの価値を最大化することを目指しています。多くのいわゆるSpec-Driven Developmentはアンチパターンです—大量のドキュメントをLLMに投げ込み、過剰な「ルール」がかえってモデルの注意力と遵守能力を低下させます。適切なバランスを見つけられないと、過剰設計の罠に陥りやすくなります。

Spec-Driven Developmentを真に活用するには、**モジュール化と段階的アプローチが必須**です。要件をモジュールと計画に分割し、各ステップごとにSpec-Drivenを適用してください。

## 📐 設計原則

> 詳細は [TODO.md](../../TODO.md) を参照

- **MVPマインドセット**: まずシンプルに動かす、必要に応じて拡張
- **引き算してから足し算**: 複雑さを先取りしない
- **gitを活用**: ブランチで並行作業、revertでロールバック—車輪の再発明をしない
- **ツールであって強制ではない**: `/dd-check`はツール、ブロッキングゲートではない

---

<div align="center">

**このプロジェクトが役に立ったら、⭐ Starをお願いします！**

</div>
