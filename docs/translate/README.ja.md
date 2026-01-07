<div align="center">

# Spec-Driven Document-First Development Framework

**仕様駆動 · ドキュメントファースト · AI支援**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Compatible-blueviolet)](https://claude.ai/claude-code)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)

[English](https://github.com/mkhrdev/cc-spec-driven/blob/main/README.md) · [中文](https://github.com/mkhrdev/cc-spec-driven/blob/main/docs/translate/README.zh.md) · [日本語](https://github.com/mkhrdev/cc-spec-driven/blob/main/docs/translate/README.ja.md)

</div>

仕様駆動・ドキュメントファーストの要件管理フレームワーク（マルチプロダクト対応）

**コアポジショニング**：要件ドキュメントの管理、変更の追跡、Specの出力を行い、下流ツール（Kiro、Cursor、OpenCode）が高品質なSpecに基づいてコードを生成し、E2Eテストを完了できるようにします。

---

## ✨ なぜこのフレームワーク

| 従来のアプローチ | Spec-Driven |
|----------------|-------------|
| 最初にすべてのドキュメントを作成 | 変更に応じて段階的に拡充 |
| ドキュメントとコードが乖離 | ドキュメントが唯一の信頼源 |
| 手動で整合性を確認 | AIが検証と更新を支援 |
| 要件変更の追跡が困難 | CR-IDで全工程を追跡 |

## 🎯 コアコンセプト

- **変更駆動** - 一度にすべてを書く必要なし、変更ごとに段階的に拡充
- **AI支援** - 自然言語で入力、AIが統一フォーマットに整形
- **ドキュメントが真実** - 確定したドキュメントが開発・テストの唯一の基準
- **粗から細へ** - まずモジュール概要、必要に応じて機能を詳細化

## フレームワークの強み

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
- gitブランチで並行作業を実現、各ブランチに独立したRC、並行ボトルネックなし
- rebase skillでマージ、自然言語でのマージルールが明確

---

## クイックスタート

```bash
# 1. プロダクトを初期化
/dd-init my-product

# 2. 変更を作成（自然言語で記述）
/dd-update "ユーザーログイン機能を追加、メールと電話番号に対応"

# 3. 確定してRCプレビューを生成
/dd-confirm CR-001

# 4. 正式ドキュメントにマージ
/dd-done CR-001
```

---

## コマンド概要

### コアコマンド

| コマンド | 用途 |
|---------|------|
| `/dd-init` | プロダクト初期化 |
| `/dd-update` | 変更の作成/修正 |
| `/dd-confirm` | 変更を確定、RCプレビューを生成 |
| `/dd-done` | RCを正式ドキュメントにマージ |
| `/dd-status` | ステータス確認 |
| `/dd-drop` | 変更を破棄 |

### 補助コマンド

| コマンド | 用途 |
|---------|------|
| `/dd-check` | 整合性チェック |
| `/dd-rebase` | ブランチ競合の処理 |
| `/dd-spec-dev` | 開発Specを生成 |
| `/dd-spec-test` | テストSpecを生成 |

> 完全なドキュメント：[CLAUDE.md - Skills](CLAUDE.md#skills)

---

## ディレクトリ構造

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
        │   └── {feature}.rc-{id}.md  # ビジネス要件 (CRプレビュー)
        │
        ├── changes/              # 変更記録
        │   ├── _index.yaml       # 変更インデックス
        │   ├── CR-{id}.md        # 進行中の変更
        │   ├── archive/          # 完了した変更
        │   └── dropped/          # 破棄された変更
        │
        └── specs/                # 仕様ファイル
            ├── CR-{id}.dev.md    # 開発仕様
            ├── CR-{id}.test.md   # テスト仕様
            └── archive/          # 完了した仕様
```

---

## ロードマップ

### Phase 1: 基盤整備
- [ ] `/dd-spec-test` でGherkin形式出力
- [ ] E2E統合ドキュメント（Maestro）

### Phase 2: VSCode拡張機能
- [ ] CRステータスパネル
- [ ] 依存グラフの可視化

### Phase 3: E2Eテストループ
- [ ] `/dd-spec-e2e` でMaestro YAML生成
- [ ] 完全なE2E統合サンプル

---

## 他ツールとの関係

本フレームワークはAI開発ツールチェーンの**上流**であり、代替ではありません：

| ツール | ポジショニング | 本フレームワークとの関係 |
|--------|---------------|------------------------|
| AWS Kiro | 単一プロジェクト開発支援 | 本フレームワークがSpec出力、Kiroがコード生成 |
| Cursor | AIプログラミングIDE | 本フレームワークがSpec出力、Cursorが実装 |
| GitHub Spec Kit | Specフォーマット標準 | 本フレームワークがSpecライフサイクルを管理 |

### 独自の価値

| 機能 | 本フレームワーク | 他ツール |
|------|----------------|----------|
| RCプレビュー機構 | ✅ | ❌ |
| 双方向依存追跡 | ✅ 自動 | 手動または無し |
| Context Loading階層化 | ✅ | ❌ |
| マルチプロダクト管理 | ✅ | 単一プロジェクト |
| CRライフサイクル | ✅ 完全追跡 | 無しまたは部分的 |

---

<div align="center">

**このプロジェクトが役に立ったら、Starをお願いします！**

</div>
