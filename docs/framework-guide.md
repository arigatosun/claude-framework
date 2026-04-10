# アリガトサン開発フレームワーク ガイドブック

## このフレームワークとは

Claude Code / Codex CLI を使った開発で、チーム全員が同じ品質・同じワークフローで開発できるようにするための共通設定テンプレート。

**核心思想**: テスト駆動開発（TDD） + 設計と実装の分離

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

自動で以下が実行される:

1. **技術スタック自動検出** — package.json, tsconfig.json, Dockerfile等をスキャンしてRuntime/Framework/DB/デプロイ先を特定
2. **アーキテクチャ可視化** — visual-explainerでシステム構成図・プロジェクト現状スナップショットをHTML生成 → `docs/architecture/` に保存
3. **既存設定との競合解消** — 既存のCLAUDE.md, .claude/, docs/ を検出し、内容をマージ（既存設定は削除せず保持）
4. **settings.json最適化** — 検出した技術スタックに応じてallow/askリストを自動調整（pnpm検出→pnpmコマンド許可等）
5. **commit** — 全ファイルを配置してcommit

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

---

## バージョン管理

このフレームワークは `arigatosun/claude-framework` リポジトリで管理。
改善提案はIssue or PRで。全員で育てていく。
