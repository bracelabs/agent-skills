# agent-skills

[English](README.md) | **日本語**

[`gh skill`](https://cli.github.com/manual/gh_skill)でインストールできるagent skill集。

| スキル | 目的 |
| --- | --- |
| `agmsg-team` | [agmsg](https://github.com/fujibee/agmsg)と`claude` / `codex`両方のCLIが入った環境で、セッションで選択したモデルをTask Ownerにしてピアチームを組み、実装・調査・QA・レビュー・相談を役割分担して進める。Claude Code / Codexのどちらからでも起動できる。 |
| `project-scaffold` | Org Standardを作成し、各プロジェクトに適用する。Built-in Starterを使う方法、既存プロジェクトやドキュメントからBootstrapする方法、既存のOrg Standardを使う方法に対応する。 |
| `project-scaffold-audit` | プロジェクトとOrg Standardの差分を確認し、`Local` / `Promote` / `Remove-Migrate` / `Needs-decision`に分類する。内容を確認・調整したうえで、承認された変更をOrg Standardに反映する。 |
| `project-scaffold-maintain` | このリポジトリのメンテナー向け。`project-scaffold-audit`で`Global Promote`とされた改善候補を確認し、Built-in Starterに取り込むためのPRを作成する。 |

## インストール

```sh
gh skill install bracelabs/agent-skills <skill-name>
```

## project-scaffoldの考え方と使い方

AIエージェント向けの初回セットアップ資料は、英語の
[project-scaffold setup guide](docs/project-scaffold-setup.md)を参照してください。

### 3つのレイヤー

- **Built-in Starter** — スキルに同梱の汎用テンプレート。
- **Org Standard** — チームや組織で共通して使う、プロジェクト構成やドキュメント運用の標準。本体は
  `$PROJECT_SCAFFOLD_HOME`（既定`~/.config/agent-skills/project-scaffold/`）配下の`scaffold/`に置かれる。Git管理すればチームや組織全体に配布・共有可能。
- **Project** — Org Standardを適用した個々のリポジトリ。

### project-scaffold — Org Standardを作って適用する

初回は、Org Standardをどう用意するか3つから選ぶ:

1. **Built-in Starterから作る** — 同梱テンプレートをベースにする。
2. **既存資産からBootstrapする** — 参照プロジェクト・AGENTS.md・開発ガイドラインなどを一度だけ読み取って作る（`project-scaffold-audit`が必要）。作成後に、参照プロジェクトで複数回auditを実行することを推奨。
3. **既存のOrg Standardを使う** — すでにあるもの（ローカルパスまたはGitリモート）を指定する。

- 1・2の場合は、作成後にGitの初期化とリモートリポジトリ作成まで案内する（不要なら省略可）。
- 用意できたら各プロジェクトに適用する。適用は必ず「変更計画を提示 → 承認 → 最小変更」の順。

### project-scaffold-audit — Org Standardを育てる

初回に作るOrg Standardは内容が薄く、複数回auditを繰り返すことで内容が充実していく。Standardに足したい・直したい点がたまってきたら`project-scaffold-audit`を実行する。

1回のaudit:

1. プロジェクトとOrg Standardの差分を洗い出す。
2. 各差分を分類する:
   - `Local` — このプロジェクト固有。そのまま残す。
   - `Promote` — Org Standardに取り込む候補。
   - `Remove-Migrate` — 古い・重複・標準と矛盾。整理する候補。
   - `Needs-decision` — 判断材料が足りない。
3. 内容を確認・調整し、承認したものだけOrg Standardに反映する
   （Git管理ならPR、していなければ直接編集）。

汎用性が高くBuilt-in Starterにも入れるべきものは`Global Promote`としてフラグするだけに留める。取り込みは`project-scaffold-maintain`（メンテナー向け）。

## このリポジトリの保守

- `gh skill publish --dry-run`が通ること。リリースは`gh skill publish --tag vX.Y.Z`（GitHubリリースを作成し、利用者はタグからインストールする）。
- `skills/project-scaffold/starter/`は手編集しない — `project-scaffold-maintain`の役割。
- ローカル反復は`skills/<name>`を`~/.claude/skills/`と`~/.codex/skills/`にシンボリックリンクする。同じ名前のシンボリックリンクと`gh skill install`/`update`は併用しない（片方を消してからもう片方を使う）。
