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

## 🔄 ワークフロー

```
┌──────────┐      ┌──────────┐
│ feature  │╌╌╌╌╌▶│  /done   │
└──────────┘      └────▲─────┘
     │                 │
     ▼                 │
┌──────────┐      ┌───────────┐      ┌─────────────────┐
│ /update  │◀────▶│ /confirm  │─────▶│ spec (dev/test) │
└────┬─────┘      └─────┬─────┘      └─────────────────┘
     │                  │
     └────────┬─────────┘
              ▼
        ┌──────────┐
        │  /drop   │
        └──────────┘
```

### ステータス説明

| ステータス | ドキュメント変化 |
|-----------|-----------------|
| update | CRのみ、ドキュメント変更なし |
| confirm | `.rc-{id}.md` プレビューを生成 |
| done | RCを削除、正式版にマージ |
| drop | RCを削除、ロールバック不要 |

## 📖 詳細ドキュメント

- **[CLAUDE.md](../../CLAUDE.md)** - AI動作ガイド、ドキュメントフォーマット、ワークフロー詳細

## 🤔 設計思想

「最も価値のあるコンテキストをいかに構築するか」を軸に設計されており、すべてのトークンの価値を最大化することを目指しています。多くのいわゆるSpec-Driven Developmentはアンチパターンです—大量のドキュメントをLLMに投げ込み、過剰な「ルール」がかえってモデルの注意力と遵守能力を低下させます。適切なバランスを見つけられないと、過剰設計の罠に陥りやすくなります。

Spec-Driven Developmentを真に活用するには、**モジュール化と段階的アプローチが必須**です。要件をモジュールと計画に分割し、各ステップごとにSpec-Drivenを適用してください。

---

<div align="center">

**このプロジェクトが役に立ったら、⭐ Starをお願いします！**

</div>
