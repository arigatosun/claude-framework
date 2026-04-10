# アリガトサン開発フレームワーク ガイドブック

## このフレームワークとは

Claude Code / Codex CLI を使った開発で、チーム全員が同じ品質・同じワークフローで開発できるようにするための共通設定テンプレート。

**核心思想**: テスト駆動開発（TDD） + 設計と実装の分離

---

## 前提条件

### visual-explainer プラグイン（必須）

アーキテクチャ図の自動生成に使用する。フレームワーク導入前にインストールしておくこと。

```
/plugin marketplace add nicobailon/visual-explainer
/plugin install visual-explainer
/reload-plugins
```

> 未インストールの場合、onboard-projectスキル実行時にインストール案内が表示される。

### GitHub CLI（推奨）

リポジトリ作成・PR操作に使用。

```bash
# インストール確認
gh --version
```

---

## セットアップ

### 方法A: 既存プロジェクトへの自動導入（推奨）

既存プロジェクトに導入する場合は、**onboard-projectスキル**で全自動化できる。

```bash
# 1. 対象プロジェクトのディレクトリに移動
cd /path/to/your-project

# 2. Claude Codeで以下を実行
# 「このプロジェクトにフレームワークを導入して」
```

#### 自動導入の流れ（PHASE 0〜4）

**PHASE 0: 前提条件チェック**
- visual-explainerプラグインの導入状態を確認
- 未導入の場合 → インストール手順を案内し、手動実行を依頼
- ユーザーが「スキップ」を選択 → PHASE 2でテキスト版フォールバックに切替

**PHASE 1: 技術スタック自動検出**
- package.json, tsconfig.json, Dockerfile, vercel.json 等をスキャン
- Runtime / Framework / DB / デプロイ先 / テストツール / CSS / 認証 / CI/CD を自動特定
- 検出結果をユーザーに提示 → 確認・修正の機会あり

**PHASE 2: アーキテクチャ可視化**
- `/visual-explainer:project-recap` → プロジェクト現状スナップショットHTML
- `/visual-explainer:generate-web-diagram` → システム構成図HTML
- 生成物は `docs/architecture/` に保存
- （visual-explainer未導入時は `docs/architecture/overview.md` にテキスト版を生成）

**PHASE 3: 既存設定との競合解消**
- 既存の CLAUDE.md, .claude/, docs/ を検出
- CLAUDE.md → プロジェクト固有ルールを保持しつつフレームワーク構造に統合
- settings.json → denyは和集合（安全側）、allowは既存を尊重してマージ
- rules/ → 重複はフレームワーク版に統合、プロジェクト固有はそのまま保持
- docs/ → 既存ファイルは一切削除しない。フレームワーク分を追加
- マージ結果をユーザーに提示 → 確認・修正の機会あり

**PHASE 4: ファイル配置と最終設定**
- CLAUDE.md の環境情報をPHASE 1の検出結果で自動記入
- settings.json のallowリストを技術スタックに最適化（pnpm検出→pnpmコマンド許可等）
- 全ファイル配置 → ユーザー最終確認 → commit

各フェーズでユーザー確認を挟むため、意図しない変更は発生しない。

### 方法B: 新規プロジェクトへの手動導入

```bash
# 1. フレームワークリポジトリをclone（初回のみ）
gh repo clone arigatosun/claude-framework ~/claude-framework

# 2. 新規プロジェクトにテンプレートをコピー
cp -r ~/claude-framework/CLAUDE.md ./
cp -r ~/claude-framework/.claude ./
cp -r ~/claude-framework/docs ./
cp ~/claude-framework/.gitignore ./

# 3. CLAUDE.md の環境情報をプロジェクトに合わせて編集
```

```markdown
## 環境情報
- OS: Windows 11
- Runtime: Node.js 20 / pnpm
- Framework: Next.js 14 (App Router)
- DB: Supabase (PostgreSQL)
- デプロイ先: Vercel
```

### 個人設定（任意・共通）

```bash
# CLAUDE.local.md を作成（git管理外）
cat > CLAUDE.local.md << 'EOF'
# 個人設定
- mainブランチへの直接push許可（単独作業時）
- コミットメッセージの補足は英語でもOK
EOF
```

### 初回commit

```bash
git add CLAUDE.md .claude/ docs/ .gitignore
git commit -m "chore: アリガトサン開発フレームワーク導入"
```

---

## 開発ワークフロー（STEP 1〜5）

### 全体の流れ

```
STEP 1: 設計 → STEP 2: テスト設計 → STEP 3: 実装 → STEP 4: 検証・commit
                 ↑                                          |
                 └──────── STEP 5: 修正・拡張時はここに戻る ──┘
```

### STEP 1: 設計フェーズ

Claude Codeに「{機能名}を実装して」と伝えると、**いきなりコードを書かない**。

Claude Codeが YES/NO・選択式で質問してくる:

```
認証方式を選択してください:
  A) メール+パスワードのみ
  B) メール+パスワード + OAuth（Google）
  C) その他
  → ?
```

全ての質問に回答すると、設計メモが `docs/design-notes/{機能名}.md` に保存される。

**ルール**: 曖昧な状態では絶対に実装に進まない。質問が多くても全て回答する。

### STEP 2: テスト設計ファースト

設計が確定したら、**実装より先に**テストケースを作る。

Claude Codeが提案してくる:

```
以下のテストケースを docs/test-spec.md に追加します:
  TC-AUTH-001 [P0] メール+パスワードで登録成功
  TC-AUTH-002 [P0] 重複メールで登録失敗
  追加してよいですか？ → Y / N
```

承認すると test-spec.md に追記され、テスト設計だけが先にcommitされる。

### STEP 3: 実装

テスト仕様書の「確認ポイント」を通すことをゴールにコードを書く。

- Phase単位（4-5ステップ）で分割して進行
- 各Phase完了後にテスト実行

### STEP 4: 検証・commit

- 全テストPASS → commit
- `docs/progress.md` が自動更新される

### STEP 5: 修正・拡張時

後日、既存機能を修正・拡張する場合:

1. Claude Codeが既存のテストケースへの影響を分析
2. テスト設計を先に更新
3. 実装

---

## ファイル構成と役割

```
project-root/
├── CLAUDE.md                          # Claude Codeの初期設定（環境情報・ワークフロー）
├── CLAUDE.local.md                    # 個人設定（git管理外）
├── .claude/
│   ├── settings.json                  # パーミッション設定（安全柵）
│   ├── rules/
│   │   ├── tdd-workflow.md            # TDDの手順ルール
│   │   ├── design-phase.md            # 設計フェーズの質問ルール
│   │   ├── session-continuity.md      # セッション跨ぎの継続ルール
│   │   ├── coding-standards.md        # コーディング規約
│   │   ├── git-workflow.md            # Git運用ルール
│   │   └── mistakes.md               # やらかしログ
│   ├── skills/
│   │   ├── onboard-project/SKILL.md   # 既存プロジェクト自動導入スキル
│   │   ├── test-design/SKILL.md       # テスト設計スキル
│   │   └── phase-commit/SKILL.md      # Phase完了時のcommitスキル
│   └── agents/
│       ├── reviewer.md                # コードレビューエージェント
│       └── security-check.md          # セキュリティチェックエージェント
├── docs/
│   ├── test-spec.md                   # テスト仕様書（一元管理）
│   ├── progress.md                    # 作業進捗（セッション跨ぎ用）
│   ├── architecture/                  # アーキテクチャ図（HTML）
│   ├── design-notes/                  # 設計メモ
│   └── framework-guide.md            # このファイル
└── .gitignore
```

### 各ファイルの役割まとめ

| ファイル | 役割 | 編集頻度 |
|---------|------|---------|
| CLAUDE.md | Claude Codeの初期OS。環境情報・基本ルール | プロジェクト初期に1回 |
| settings.json | 実行許可・禁止の境界線 | ほぼ変更なし |
| rules/ | 詳細なルール集 | mistakes.mdは随時追記 |
| skills/ | 定型作業の自動化 | 必要に応じて追加 |
| agents/ | 独立した専門エージェント | 必要に応じて追加 |
| test-spec.md | テスト仕様書 | 実装のたびに更新（TDDの核） |
| progress.md | 作業引継ぎ | 毎Phase自動更新 |
| architecture/ | アーキテクチャ図（HTML/md） | 導入時に生成、構成変更時に更新 |

---

## スキル・エージェントの使い方

### スキル（定型作業の自動化）

| スキル | トリガー例 | 内容 |
|--------|----------|------|
| **onboard-project** | 「フレームワーク導入して」「セットアップ」 | 既存プロジェクトへの自動導入（技術スタック検出→アーキテクチャ可視化→競合解消→配置） |
| **test-design** | 「テスト書いて」「テストケース追加」 | docs/test-spec.md にテストケースを追加・更新 |
| **phase-commit** | 「Phase完了」「コミットして」 | テスト実行→progress.md更新→test-spec.md更新→commit |

スキルはClaude Codeが自動的に認識するため、トリガーとなるキーワードを含む指示を出すだけでよい。

### エージェント（独立した専門家）

| エージェント | 起動方法 | 内容 |
|------------|---------|------|
| **reviewer** | 「レビューして」「コードチェック」 | 正確性・セキュリティ・保守性・テスト整合性の観点でコードレビュー。問題なければ「LGTM」 |
| **security-check** | 「セキュリティチェック」「脆弱性スキャン」 | OWASP Top 10基準で脆弱性を検出。実際に悪用可能な問題のみ報告 |

エージェントは読み取り専用。ファイルを変更することはない。

### visual-explainer コマンド（アーキテクチャ可視化）

| コマンド | 用途 |
|---------|------|
| `/visual-explainer:generate-web-diagram` | アーキテクチャ図・フロー図・ER図をHTMLで生成 |
| `/visual-explainer:project-recap` | プロジェクトの現状スナップショットを生成 |
| `/visual-explainer:generate-visual-plan` | 機能実装のビジュアル計画書を生成 |
| `/visual-explainer:diff-review` | コード変更のビジュアルレビュー |
| `/visual-explainer:plan-review` | プランをコードと照合してリスク評価 |
| `/visual-explainer:fact-check` | ドキュメントの内容をコードと照合して検証 |

生成されたHTMLは `~/.agent/diagrams/` に保存され、ブラウザで自動的に開く。
プロジェクト管理用にコピーする場合は `docs/architecture/` に配置する。

---

## セッション跨ぎの仕組み

### 作業を中断するとき

特に何もしなくてよい。Claude Codeが `docs/progress.md` を自動更新している。

### 翌日セッションを再開するとき

Claude Codeを起動するだけでよい。自動で:

1. `docs/progress.md` を読み込む
2. 前回の作業状況を要約して報告する
3. 「続きから再開しますか？」と聞いてくる

```
前回の作業状況:
- 機能: ユーザー登録
- 進捗: Phase 2/5 完了
- 次にやること: Phase 3 バリデーション実装

続きから再開しますか？ → Y / N
```

---

## settings.json の安全設定

### 自動許可（allowリスト）
npm run, git status, git diff 等の安全なコマンド → 確認なしで実行

### 絶対禁止（denyリスト）
- `rm -rf` — ファイル全削除
- `curl | sh` / `wget | bash` — 外部スクリプト実行
- `.env` ファイルの読み取り — シークレット漏洩防止

### 都度確認（askリスト）
- `git push` — プッシュ前にテスト通過を確認
- `rm` — ファイル削除は慎重に
- `gh pr create` — PR作成前に内容確認

---

## やらかしログ（mistakes.md）の運用

チーム全員で育てる最重要ファイル。

### ミスが発生したら

1. `.claude/rules/mistakes.md` を開く
2. 以下のフォーマットで追記:
   ```
   ### Case N: {タイトル}
   状況: {何が起きたか}
   原因: {なぜ起きたか}
   → 対策: {具体的な再発防止策}
   ```
3. commit: `docs: やらかしログ追記 - {タイトル}`

### ルール
- 「気をつける」は対策として認めない。具体的な行動を書く
- 人を責めない。仕組みで防ぐ方法を書く

---

## よくあるQ&A

### Q: 既存プロジェクトに導入したらCLAUDE.mdが上書きされる？
A: されない。onboard-projectスキルは既存のCLAUDE.mdを読み込み、プロジェクト固有ルールを保持したままフレームワーク構造に統合する。各フェーズでユーザー確認があるので、意図しない変更は発生しない。

### Q: visual-explainerのインストールができない
A: onboard-projectスキル実行時に案内が表示される。インストールできない場合は「スキップ」と回答すれば、テキスト版のアーキテクチャ概要（`docs/architecture/overview.md`）が代わりに生成される。後からvisual-explainerを導入してHTMLに差し替えることも可能。

### Q: 自動検出された技術スタックが間違っている
A: PHASE 1の検出結果提示時に修正できる。「修正内容を入力」を選択して正しい情報を伝えれば、修正した内容でCLAUDE.mdが生成される。

### Q: Claude Codeが設計フェーズを飛ばして実装を始めた
A: CLAUDE.md の禁止事項に記載があるため、通常は発生しない。発生した場合は「設計フェーズから始めて」と指示する。

### Q: テスト仕様書が長くなりすぎた
A: カテゴリごとに別ファイルに分割可能。`docs/test-spec/auth.md`, `docs/test-spec/editor.md` のように分ける。

### Q: 個人的なルールを追加したい
A: `CLAUDE.local.md` に書く。このファイルはgit管理外なのでチームに影響しない。

### Q: 新しいスキルやエージェントを追加したい
A: `.claude/skills/{スキル名}/SKILL.md` または `.claude/agents/{エージェント名}.md` を作成してPRを出す。

### Q: mistakes.md に書くべきか迷う
A: 迷ったら書く。不要なら後で消せる。書かなかったミスは繰り返される。

### Q: アーキテクチャ図を更新したい
A: `/visual-explainer:generate-web-diagram` を実行して再生成する。生成されたHTMLを `docs/architecture/` にコピーすればよい。

---

## バージョン管理

このフレームワークは `arigatosun/claude-framework` リポジトリで管理。
改善提案はIssue or PRで。全員で育てていく。
