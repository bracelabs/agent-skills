# agent-skills

[English](README.md) | **日本語**

[`gh skill`](https://cli.github.com/manual/gh_skill) でインストールできる agent skill 集。

| スキル | 目的 |
| --- | --- |
| `agmsg-team` | [agmsg](https://github.com/fujibee/agmsg) と `claude` / `codex` 両方の CLI が入った環境で、セッションで選択したモデルを Task Owner にしてピアチームを組み、実装・調査・QA・レビュー・相談を役割分担して進める。Claude Code / Codex のどちらからでも起動できる。 |
| `project-scaffold` | Org Scaffold を作成・更新し、各プロジェクトに適用する。Built-in Starter を使う方法、既存プロジェクトやドキュメントから Bootstrap する方法、既存の Org Scaffold を使う方法に対応する。 |
| `project-scaffold-audit` | プロジェクトと Org Scaffold の差分を確認し、`Local` / `Promote` / `Remove-Migrate` / `Needs-decision` に分類する。内容を確認・調整したうえで、承認された変更を Org Scaffold に反映する。 |
| `project-scaffold-maintain` | このリポジトリのメンテナー向け。audit で `Global Promote` とされた改善候補を確認し、Built-in Starter に取り込むための PR を作成する。 |

## インストール

```sh
gh skill install bracelabs/agent-skills <skill-name>
```

## project-scaffold の考え方と使い方

AI エージェント向けの初回セットアップ資料は、英語の
[project-scaffold setup guide](docs/project-scaffold-setup.md) を参照してください。

### 3 つのレイヤー

- **Built-in Starter** — スキルに同梱の汎用テンプレート。
- **Org Scaffold** — チームや組織で共通して使う、プロジェクト構成やドキュメント運用の標準。`$PROJECT_SCAFFOLD_HOME`
  （既定 `~/.config/agent-skills/project-scaffold/`）に置かれる。git管理すればチームや組織全体に配布・共有可能。
- **Project** — Org Scaffold を適用した個々のリポジトリ。

### project-scaffold — Org Scaffold を作って適用する

初回は、Org Scaffold をどう用意するか3つから選ぶ:

1. **Built-in Starter から作る** — 同梱テンプレートをベースにする。
2. **既存資産から Bootstrap する** — お手本プロジェクト・AGENTS.md・開発ガイドラインなどを一度だけ読み取って作る（`project-scaffold-audit`が必要）。
3. **既存の Org Scaffold を使う** — すでにある Scaffold（ローカルパスまたはgitリポジトリ）を指定する。

- 1・2の場合は、作成後にgit初期化とリモートリポジトリ作成まで案内する（不要なら省略可）。
- 用意できたら各プロジェクトに適用する。適用は必ず「変更計画を提示 → 承認 → 最小変更」の順。

### project-scaffold-audit — Org Scaffold を育てる

プロジェクトを進める中で標準に足したい・直したい点が出てきたら audit する:

1. プロジェクトと Org Scaffold の差分を洗い出す。
2. 各差分を分類する:
   - `Local` — このプロジェクト固有。そのまま残す。
   - `Promote` — Org Scaffold に取り込む候補。
   - `Remove-Migrate` — 古い・重複・標準と矛盾。整理する候補。
   - `Needs-decision` — 判断材料が足りない。
3. 内容を確認・調整し、承認したものだけ Org Scaffold に反映する
   （git 管理なら PR、していなければ直接編集）。

汎用性が高く Built-in Starter にも入れるべきものは `Global Promote` としてフラグするだけに留める。取り込みは `project-scaffold-maintain`（メンテナー向け）。

## このリポジトリの保守

- `gh skill publish --dry-run` が通ること。リリースは `gh skill publish --tag vX.Y.Z` （GitHubリリースを作成し、利用者はタグからinstallする）。
- `skills/project-scaffold/starter/` は手編集しない — `project-scaffold-maintain` の役割。
- ローカル反復は `skills/<name>` を `~/.claude/skills/` と `~/.codex/skills/` にシンボリックリンクする。同じ名前のsymリンクと `gh skill install`/`update` は併用しない（片方を消してからもう片方を使う）。
