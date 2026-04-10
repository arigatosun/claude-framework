# アリガトサン開発フレームワーク

Claude Code / Codex CLI を使った開発のためのチーム共通フレームワーク。

## 特徴

- **TDD（テスト駆動開発）**: テスト設計を先に書いてから実装する
- **設計と実装の分離**: 曖昧な要件を全て解消してから実装に入る
- **既存プロジェクト自動導入**: 技術スタック自動検出 + アーキテクチャ可視化 + 競合自動解消
- **セッション継続**: 翌日のセッションでも前回の続きから再開できる
- **やらかしログ**: チームのミスを記録して再発を防ぐ
- **安全設定**: 危険な操作を構造的にブロック

## 前提条件

- Claude Code (CLI / Desktop / Web)
- visual-explainer プラグイン（アーキテクチャ可視化に使用）
  ```
  /plugin marketplace add nicobailon/visual-explainer
  /plugin install visual-explainer
  /reload-plugins
  ```

## クイックスタート

### 既存プロジェクトへの導入（推奨）

```bash
# 1. 対象プロジェクトでClaude Codeを起動
cd /path/to/your-project

# 2. 以下を入力するだけ
# 「このプロジェクトにフレームワークを導入して」
```

技術スタック自動検出 → アーキテクチャHTML生成 → 既存設定とのマージ → commit まで自動実行される。

### 新規プロジェクトへの導入

```bash
# 1. このリポジトリをclone
gh repo clone arigatosun/claude-framework ~/claude-framework

# 2. 新規プロジェクトにコピー
cp -r ~/claude-framework/CLAUDE.md ./your-project/
cp -r ~/claude-framework/.claude ./your-project/
cp -r ~/claude-framework/docs ./your-project/
cp ~/claude-framework/.gitignore ./your-project/

# 3. CLAUDE.md の環境情報をプロジェクトに合わせて編集
```

## 詳細ガイド

[docs/framework-guide.md](docs/framework-guide.md) を参照。

## ファイル構成

```
├── CLAUDE.md                    # Claude Codeの初期設定
├── .claude/
│   ├── settings.json            # パーミッション（安全柵）
│   ├── rules/                   # 詳細ルール集
│   ├── skills/                  # 定型作業の自動化
│   └── agents/                  # 独立専門エージェント
└── docs/
    ├── test-spec.md             # テスト仕様書テンプレート
    ├── progress.md              # 作業進捗
    ├── design-notes/            # 設計メモ
    └── framework-guide.md       # ガイドブック
```

## コントリビューション

改善提案はIssue or PRで。チーム全員で育てていくフレームワークです。

## ライセンス

MIT
